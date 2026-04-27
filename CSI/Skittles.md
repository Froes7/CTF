# **Skittle's Newspaper**
> **Categoria:** *Web Exploitation*
> 
> **Autor:** *AltDel*

### Introdução

Esta questão de [*CTF*](https://en.wikipedia.org/wiki/Capture_the_flag_(cybersecurity)) é um desafio sobre [*web exploitation*](https://devopedia.org/web-exploitation), temática do jogo [*Cyberpunk 2077*](https://pt.wikipedia.org/wiki/Cyberpunk_2077). Ela simula um ***blog* jornalístico** de um personagem chamado ***Skittle*** e possui notícias sobre crimes e incidentes na cidade de [*Night City*](https://cyberpunk.fandom.com/wiki/Night_City):

[![wp11550782-night-city-cyberpunk-wallpapers.jpg](https://i.postimg.cc/3xZb4sxV/wp11550782-night-city-cyberpunk-wallpapers.jpg)](https://postimg.cc/xXcPBFH3)

### Processo

A página inicial do blog possui um título, uma barra de pesquisa, e 5 notícias diferentes sobre **ataques cibernéticos e vazamento de dados** em várias corporações:

[![Skittlenews.png](https://i.postimg.cc/QdVW15X4/Skittlenews.png)](https://postimg.cc/v4FBd4xf)

Não há pistas sobre a *flag* na página inicial, porém, é possível checar **arquivos padrões** que frequentemente expõem **diretórios sensíveis** esquecidos por desenvolvedores, como por exemplo, no [*"Robots.txt"*](https://www.cloudflare.com/pt-br/learning/bots/what-is-robots-txt/) está escrito:

> "Disallow: **/admin.php**"
>
> Nota: também é possível encontrar um comentário no [código-fonte](https://pt.wikipedia.org/wiki/C%C3%B3digo-fonte) da página sobre este mesmo diretório.

O diretório encontrado nos redireciona a um outro diretório chamado de ***"/login.php"***, onde pede um **usuário** e uma **senha**. Provavelmente a *flag* está depois dessa página de *login*, porém, não temos as credenciais para acessá-la. No **código-fonte** dessa página é possível notar uma pista em forma de comentário sobre qual vulnerabilidade podemos utilizar:

> "anotação: trocar **sqlite** por **mysql** qdo eu conseguir pagar por um servidor melhor!"

### Vulnerabilidade

Agora que sabemos que o servidor utiliza [***sqlite***](https://codigofacil.com.br/o-que-e-sqlite/), para burlar o *login* e acessar a *flag* podemos utilizar uma **vulnerabilidade** conhecida como [***SQL Injection***](https://www.datacamp.com/pt/tutorial/sql-injection). [**SQL**](https://pt.wikipedia.org/wiki/SQL) é uma linguagem utilizada para gerenciar informações em um **banco de dados**, no caso dessa questão: **usuários e senhas**. Se o código utilizado para validar um *login* não estiver **sanitizado**, ele pode ser explorado com uma [string](https://en.wikipedia.org/wiki/String_(computer_science)) maliciosa, como por exemplo:

```
' OR 1=1 --
```

Funcionalidade:

```
' OR 1=1 --
___

' -> Caractere utilizado para fechar a string dentro do código.

OR -> Operador lógico.

1=1 -> Comparação que é sempre verdadeira.

-- -> Começo de um comentário em SQL, ou seja, ignora tudo após.
```

Basicamente, essa *string* muda a **lógica** do código para "campo **vazio** ou **verdadeiro**" e aceita o *login*, nos direcionando para a página de **administrador**, (*/admin.php*), onde está a *flag* do desafio:

[![2026-02-23-16-20.png](https://i.postimg.cc/dtCXRqFB/2026-02-23-16-20.png)](https://postimg.cc/BXqM4fXL)

> DUCK{SK1TTL3_N3W5_2077_SQL1NJ3CT10N_H45_B33N_H4CK3D}

### Mitigação

***SQL injection*** é uma vulnerabilidade comum dentro do mundo da **cybersegurança**, mas não deveria ser. A criação de **códigos não sanitizados** pode ser por diversos motivos: usuários como parte da consulta, privilégios desnecessários, caracteres e entradas que deveriam ser proibidas e até mensagens de erros detalhadas de mais.

Tudo isso pode desencadear em **prejuízos** enormes para empresas, por isso, há de ter equipes de programadores **treinados** e **profissionais de cybersegurança** para **mitigar** essa vulnerabilidade, seja por:

> **Consultas parametrizadas:** Tratar usuários apenas como valores dentro do código.
> 
> **Utilizar ORMs e Query Builders:** Ferramentas que ajudam o programador com a construção de consultas parametrizadas.
> 
> **Menor privilégio:** Contas com privilégio mínimo necessário.
> 
> **Validação de entrada:** Bloquear caracteres maliciosos e criar limites dentro da entrada do usuário. (Não como defesa primária).
>
> **Mensagens de erro genérica**: Não vazar informações importantes para atacantes.

Tudo isso para garantir a **segurança dos dados** e se defender contra **atacantes maliciosos**, caso contrário,veremos ainda mais incidentes e notícias no ***Skittle's Newspaper***.
