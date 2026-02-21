# Como funciona o gerenciamento de usuários no PostgreSQL


<br><br>

## Definição conceitual

O PostgreSQL não é igual nem ao MySQL, nem ao SQL Server, nem ao Oracle — ele fica no meio do caminho, com conceitos próprios.  
**Conceito central**
No PostgreSQL existe UMA entidade só chamada ROLE.
>[!NOTE]
>Role pode ser usuário, grupo, ou ambos.



<br><br>

## Níveis de permissão no PostgreSQL  

### 1. Permissões globais, válidas para todos os bancos:  

- LOGIN → pode conectar
- SUPERUSER → root do PostgreSQL
- CREATEDB → criar bancos
- CREATEROLE → criar/gerenciar roles
- REPLICATION
- BYPASSRLS

Equivalente a:  

- SQL Server → login
- MySQL → global privileges
- Oracle → system privileges

### 2. Nível de banco (DATABASE) 

Controla acesso ao banco, não aos objetos:

- CONNECT
- TEMP

```postgresql
GRANT CONNECT ON DATABASE <database name> TO <role>;
```

Equivalente a:

- SQL Server → acesso ao database
- Oracle → CREATE SESSION
- MySQL → acesso ao schema  

### 3. Nível de schema

Controla visibilidade do namespace:

- USAGE → pode “ver” objetos
- CREATE → pode criar objetos no schema

```postgresql
GRANT USAGE ON SCHEMA <schema name> TO <role>;
```

### 4. Pontos IMPORTANTES do PostgreSQL e pegadinhas comuns

>[!NOTE]
>Aqui o PostgreSQL se aproxima do Oracle, mas sem sinônimos.

Comparação direta com outros bancos.  
|**Banco**|**Como funciona**|  
|:---|:---|
|PostgreSQL|Uma ROLE global + grants em database/schema/objeto|
|MySQL|Um usuário, permissões globais ou por schema|
|SQL Server|Login (instância) + User (database)|
|Oracle|User + grants diretos ou via roles + sinônimos|

>[!IMPORTANT]
>Não existe “usuário por banco”  

O usuário existe no cluster inteiro, mas:  

- só entra no banco se tiver CONNECT
- só vê objetos se tiver USAGE no schema

**ALTER DEFAULT PRIVILEGES**
Permissões não herdam automaticamente:

```postgresql
ALTER DEFAULT PRIVILEGES
```

```postgresql
GRANT SELECT ON TABLES TO <role>;
```

**Ownership importa**
O owner manda mais que o GRANT.

```postgresql
ALTER TABLE x OWNER TO y;
```

É parecido com Oracle.

**Não existem sinônimos**
Diferente do Oracle:

- você usa schema.objeto
- ou ajusta o search_path


<br><br>

## Como verificar os usuários da minha instancia ?

No PostgreSQL, **usuários = roles.** Para ver os usuários da instância (cluster), você tem algumas formas — do mais simples ao mais detalhado.

### Listar da forma mais simples (psql)

Conecte-se e pela linha de comando com uma role como superusuário (postgres), execute:

```postgresql
\du
```

Isso lista todas as roles e seus atributos (login, superuser, createdb etc.).  
Exemplo de saída típica:  
|Role name|Attributes|Member of|
|:---|:---|:---|
|postgres|Superuser, Create role, Create DB, Replication, Bypass RLS |{}|
|usuario|Superuser|{}|
|usrDbBackup||{acesso_completo}|

### Listar somente usuários que podem fazer login

```postgresql
SELECT 
  rolname
FROM pg_roles
  WHERE rolcanlogin = true
ORDER BY 
  rolname;
```

Quem têm LOGIN são usuários “de verdade”.

###  Listar usuários e privilégios globais (instância)

```postgresql
SELECT
  rolname
  ,rolcanlogin
  ,rolsuper
  ,rolcreatedb
  ,rolcreaterole
  ,rolreplication
  ,rolvaliduntil
FROM pg_roles
ORDER BY 
  rolname;
```

Isso é o “raio-X” de segurança da instância.

### Listar grupos (roles sem login)

```postgresql
SELECT 
  rolname
FROM pg_roles
  WHERE rolcanlogin = false
ORDER BY 
  rolname;
```

### Listar membros de grupos (roles herdadas)

```postgresql
SELECT
  r.rolname AS role
  ,m.rolname AS member
FROM pg_auth_members am
  JOIN pg_roles r ON r.oid = am.roleid
  JOIN pg_roles m ON m.oid = am.member
ORDER BY 
  r.rolname
  ,m.rolname;
```

###  Listar usuários conectados agora

```postgresql
SELECT 
  pid
  ,usename
  ,datname
  ,client_addr
  ,application_name
  ,state
FROM pg_stat_activity
ORDER BY 
  usename
  ,datname;
```

### Importante lembrar (Dicas finais)

>[!IMPORTANT]
>Não existe “usuário por banco” no PostgreSQL
>→ o usuário é da instância

O acesso real depende de:
- CONNECT no database
- USAGE no schema
- grants nos objetos


## Resumo rápido
|**O que você quer**|**Comando**|
|:--|:--|
|Ver todos os usuários|\du|
|Só quem faz login|pg_roles WHERE rolcanlogin|
|Ver superusers|pg_roles WHERE rolsuper|
|Ver conectados|pg_stat_activity|