# OJS (Open Journal Systems) - PKP - Container/Docker


| **IMPORTANT:**                                                                                                                                                                                                                        |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **This repository is still beta, so it should be used with care in production settings - please provide feedback early and often about your experience.** <br />We are actively working to release a stable version soon. Keep tuned. |

Open Journal Systems (OJS) é um gerenciador e publicador de periódicos que foi desenvolvida pela [Public Knowledge Project](https://pkp.sfu.ca/) atráves dos seus esforços federais de financiamento para expandir e melhorar o acesso à pesquisa.

Este repositório é um fork de um trabalho primeiramente feito pelo [Lucas Dietrich](https://github.com/lucasdiedrich/ojs) e [docker-ojs](http://github.com/pkp/docker-ojs).

## Construindo sua imagem local (development)

Primeiramente, é necessário instalar o [docker](https://docs.docker.com/get-docker/) e [docker-compose](https://docs.docker.com/compose/install/).

Para todas as versões são disponibilizados os seguintes arquivos: **docker-compose.yml** e **docker-compose-local.yml**.
 - O arquivo **docker-compose.yml** contém a imagem oficial da pkp para produção. (em alfa)
 - O **docker-compose-local.yml** possui os seguintes serviços:
   - Aplicação ([OJS](http://github.com/pkp/ojs))
   - Banco de dados ([MariaDB](https://hub.docker.com/_/mariadb))
   - Serviço de e-mail(SMTP) para testes ([Mailhog](https://github.com/mailhog/MailHog))

1. Faça o clone do repositório via SSH/HTTP na sua máquina:

   ```
   git clone git@gitlab.lepidus.com.br:softwares-pkp/docker-ojs.git
   ```

2. Execute o script **develop-build.sh**:
   ```
   ./develop-build.sh
   ```
   | **DICAS:** |
   :-----------------------------------------------------------------------------------|
   | O script faz uso do sudo e gera todas as versões do OJS ou uma versão de sua escolha. |
   Para utilizar a versão específica basta informa a versão como argumento:
   ```
   ./develop-build.sh 3_3_0-8
   ```

3. Entre no diretório `development/versions/<versão desejada>/alpine/apache/<versão desejada do php>/`
   Dê uma olhada no arquivo **docker-compose-local.yml** e procure pela tag **image** abaixo do **ojs**:
   ```dockerfile
   ojs:
    image: local/ojs:3_3_0-8
   ```
   Execute o seguinte comando `docker build -t local/ojs:<versão> .` substituindo **<versão>** pela versão desejada.
   ```
   docker build -t local/ojs:3_3_0-8 .
   ```
   | **OBS:** |
   :-----------------------------------------------------------------------------------|
   | Se algo der errado verifique novamente se você executou o comando anterior com o número de versão correto ou em uma pasta sem o Dockerfile local. |
4. Após a construção da imagem do passo anterior execute o seguinte comando:
   ```
   docker-compose -f docker-compose-local.yml up
   ```
   ou
   ```
   docker-compose -f docker-compose-local.yml up -d
   ```
   Caso não queria ver os logs dos containeres.

5. Acesse **http://127.0.0.1:8081** e continue o processo de instalação via web.

   Para configurar o banco de dados utilize as seguintes opções:

   - **Database driver**: `mysqli` (ou "mysql" se o php for menor que 7.3)
   - **Host**: `db`
   - **Username**: `ojs`
   - **Password**: `ojs`
   - **Database name**: `ojs`
   - _Desmarque_ "Create new database"
   - _Desmarque_ "Beacon"

   O Diretório de uploads está de acordo com o arquivo docker-compose "/var/www/files"
## Variáveis de ambiente
Na configuração do arquivo docker-compose.yml é lido outro arquivo **.env** no qual contém algumas variáveis de ambiente como porta HTTP/MYSQL.

| Nome            | Container   | Informação         |
|:---------------:|:---------:|:---------------------|
| MYSQL_ROOT_PASSWORD    | db      | Senha do usuário root do banco de dados|
| MYSQL_USER             | db      | Nome de usuário do banco de dados|
| MYSQL_PASSWORD         | db      | Senha do usuário do banco de dados|
| MYSQL_DATABASE         | db      | Nome do banco de dados|
| MYSQL_PORT             | db      | Porta utilizada pelo banco de dados|
| SMTP_MAILHOG_PORT      | mailhog | Porta SMTP utilizada pelo mailhog|
| SMTP_MAILHOG_HTTP_PORT | mailhog | Porta HTTP utilizada pelo mailhog|
| HTTP_PORT              | ojs     | Porta HTTP utilizada pelo OJS    |
| HTTPS_PORT             | ojs     | Porta HTTPs utilizada pelo OJS   |
| COMPOSE_PROJECT_NAME   | todos   | Define o nome do projeto         |
## Volumes

Por padrão é incluído uma estrutura de diretórios em cada versão dentro do diretório `versions`, veja o diretório `./volumes` que inicialmente encontra-se vazio.
Quando você executar o docker-compose os volumes serão montados de acordo com sua configuração do arquivo `docker-compose.yml` ou `docker-compose-local.yml`. Para habilitar determinado volume basta descomentar a linha nos arquivos citados.

Quando você executar o docker-compose, os volumes com os dados serão montados e permitirá que você compartilhe arquivos de seu host com o contêiner.

| Host                                    | Container  | Volume                                | Descrição                    |
|:----------------------------------------|:----------:|:--------------------------------------|:-------------------------------|
| ./volumes/public                        | ojs        | /var/www/html/public                  | Todos os arquivos públicos |
| ./volumes/private                       | ojs        | /var/www/files                        | Todos os arquivos privados (uploads) |
| ./volumes/config/db.charset.conf        | db         | /etc/mysql/conf.d/charset.cnf         | Arquivo de configuração do mariaDB |
| ./volumes/config/ojs.config.inc.php     | ojs        | /var/www/html/config.inc.php          | Arquivos de configuração do OJS |
| ./volumes/config/php.custom.ini         | ojs        | /usr/local/etc/php/conf.d/custom.ini  | PHP custom.init                |
| ./volumes/config/apache.htaccess        | ojs        | /var/www/html/.htaccess               | Apache2 htaccess               |
| ./volumes/logs/app                      | ojs        | /var/log/apache2                      | Apache2 Logs                   |
| ./volumes/logs/db                       | db         | /var/log/mysql                        | mariaDB Logs                   |
| ./volumes/db                            | db         | /var/lib/mysql                        | mariaDB database content       |
| ./volumes/migration                     | db         | /docker-entrypoint-initdb.d           | DB init folder (with SQLs)     |
| /etc/localtime                          | ojs        | /etc/localtime                        | Sincroniza a data/hora do container com a máquina host. |

Nesta imagem, usamos [bind volumes](https://docs.docker.com/storage/bind-mounts/) com caminhos relativos porque
oferecem uma visão clara de onde seus dados estão armazenados.

O lado negativo desses volumes é que eles não podem ser [nomeados](https://towardsdatascience.com/the-complete-guide-to-docker-volumes-1a06051d2cce), o docker irá armazená-los no caminho absoluto. Como esta é apenas uma imagem, sinta-se livre para modificá-la de acordo com suas necessidades.

Por último, mas não menos importante, essas pastas de armazenamento precisam existir com as permissões corretas
antes de executar o docker-compose ou ele falhará.

Para ter certeza de que seus volumes têm as permissões corretas, você pode executar estes comandos:
   ```bash
   $ chown 100:101 ./volumes -R
   $ chown 999:999 ./volumes/db -R
   $ chown 999:999 ./volumes/logs/db -R
   ```
Em outras palavras ... todo o conteúdo dentro dos volumes pertence ao usuário apache2 e grupo (uid 100 e gid 101 dentro do conteiner), exceto para db e logs / db, pastas que pertencerão ao usuário e grupo mysql (uid e gid 999).
## Built in scripts

O Dockerfile inclui alguns scripts em "/usr/local/bin" para facilitar operações comumente utilizadas:

| Script               | Container  | Descrição|
|:---------------------|:----------:|:---------|
| ojs-run-scheduled    | ojs        | Roda "php tools/runScheduledTasks.php". Chamada pelo cron a cada hora.                              |
| ojs-cli-install      | ojs        | Usa curl para instalar OJS através das variáveis definidas no arquivo `.env` (Não funciona).            |
| ojs-pre-start        | ojs        | Garante algumas variáveis de configuração e gera um certificado auto-assinado baseado no ServerName.|
| ojs-upgrade          | ojs        | Roda "php tools/upgrade.php upgrade" (problemas quando config.inc.php é um volume).                 |
| ojs-variable         | ojs        | Substitui o valor da variável em config.inc.php (exemplo: ojs-variable variable newValue)           |
| ojs-migrate          | ojs        | Pega um dump.sql, arquivos públicos e privados da pasta "migration" e constrói um site docker (beta)|


**Alguns desses scripts ainda estão na versão beta, seja cuidadoso em usá-los.**

Você pode chamar os scripts fora do contêiner seguindo os passos abaixo:

   ```bash
   $ docker exec -it <nome/id-do-container> /usr/local/bin/ojs-variable session_check_ip Off
   ```
## Licença

GPL3 © [PKP](https://github.com/pkp)
