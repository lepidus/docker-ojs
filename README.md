# OJS (Open Journal Systems) - PKP - Container/Docker


| **IMPORTANT:** |
|:---------------------------------------------------------|
| **This repository is still beta, so it should be used with care in production settings - please provide feedback early and often about your experience.** <br />We are actively working to release a stable version soon. Keep tuned. |

Open Journal Systems (OJS) é um gerenciador e publicador de periódicos que foi desenvolvida pela [Public Knowledge Project](https://pkp.sfu.ca/) atráves dos seus esforços federais de financiamento para expandir e melhorar o acesso a pesquisa.   

Este repositório é um fork de um trabalho primeiramente feito pelo [Lucas Dietrich](https://github.com/lucasdiedrich/ojs) e [docker-ojs](http://github.com/pkp/docker-ojs).

## Construindo sua imagem local (development)

Primeiro você deverá instalar o [docker](https://docs.docker.com/get-docker/) e [docker-compose](https://docs.docker.com/compose/).

Para todas as versão são disponibilizados os seguintes arquivos: **docker-compose.yml** e **docker-compose-local.yml**.  
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

3. Entre no diretório `development/versions/<versão desejada>/php/`
   Dê uma olhada no arquivo **docker-compose-local.yml** procure pela tag **image** abaixo do **ojs**:
   ```dockerfile
   ojs:
    image: local/ojs:3_3_0-8
   ``` 
   Execute o seguinte comando `docker build -t local/ojs:<versão> .` substituindo **<versão>** pela versão desejada.
   ```
   docker build -t local/ojs:3_3_0-8 .
   ```

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
   - **Host**: `db` (which is the name of the container in the internal Docker network)
   - **Username**: `ojs`
   - **Password**: `ojs`
   - **Database name**: `ojs`
   - _Desmarque_ "Create new database"
   - _Desmarque_ "Beacon"

   O Diretório de uploads de acordo com o arquivo docker-compose "/var/www/files"

## Construindo sua imagem local

Each version folder also includes has a file called `docker-compose-local.yml`.

This compose won't ask dockerHub for the required images, it expects you build a docker image locally.

This is useful if you don't want external dependencies or you like to modify our official Dockerfiles to fit your specific needs.

To do this...

1. Go to your preferred version folder and and build the image as follows:
    ```bash
    $ docker build -t local/ojs:3_2_1-4 .
    ```

    If something goes wrong, double-check if you ran the former command with the right version number or in a folder without the local Dockerfile.

## Variáveis de ambiente
Nas configuração do arquivo docker-compose.yml é lido outro arquivo **.env** no qual contém algumas variáveis de ambiente como porta HTTP/MYSQL

## Volumes

Docker content is efimerous by design, but in some situations you would like
to keep some stuff **persistent** between docker falls (ie: database content,
upload files, plugin development...)

By default we include an structure of directories in the version folders
(see ./volumes), but they are empty and disabled by default in your compose.
To enable them, **you only need to uncomment the volume lines in
your docker-compose.yml** and fill the folders properly.

When you run the docker-compose it will mount the volumes with persistent
data and will let you share files from your host with the container.


| Host                                    | Container  | Volume                                | Description                    |
|:----------------------------------------|:----------:|:--------------------------------------|:-------------------------------|
| ./volumes/public                        | ojs        | /var/www/html/public                  | All public files               |
| ./volumes/private                       | ojs        | /var/www/files                        | All private files (uploads)    |
| ./volumes/config/db.charset.conf        | db         | /etc/mysql/conf.d/charset.cnf         | mariaDB config file            |
| ./volumes/config/ojs.config.inc.php     | ojs        | /var/www/html/config.inc.php          | OJS config file                |
| ./volumes/config/php.custom.ini         | ojs        | /usr/local/etc/php/conf.d/custom.ini  | PHP custom.init                |
| ./volumes/config/apache.htaccess        | ojs        | /var/www/html/.htaccess               | Apache2 htaccess               |
| ./volumes/logs/app                      | ojs        | /var/log/apache2                      | Apache2 Logs                   |
| ./volumes/logs/db                       | db         | /var/log/mysql                        | mariaDB Logs                   |
| ./volumes/db                            | db         | /var/lib/mysql                        | mariaDB database content       |
| ./volumes/migration                     | db         | /docker-entrypoint-initdb.d           | DB init folder (with SQLs)     |
| /etc/localtime                          | ojs        | /etc/localtime                        | Sync clock with the host one.  |
| TBD                                     | ojs        | /etc/ssl/apache2/server.pem           | SSL **crt** certificate        |
| TBD                                     | ojs        | /etc/ssl/apache2/server.key           | SSL **key** certificate        |

In this image we use "bind volumes" with relative paths because it will 
give you a clear view where your data is stored.

The down sides of those volumes is they can not be "named" and docker will 
store them with an absolute path (that it’s annoying to make stuff portable) 
but I prefer better control about where data is stored than leave it in docker hands.

This is just an image, so feel free to modify to fit your needs.

You can add your own volumes. For instance, make sense for a plugin developer
or a themer to create a volume with his/her work, to keep a persistent copy in
the host of the new plugin or theme.

An alternative way of working for developers is working with his/her own local
Dockerfile that will be build to pull the plugin for his/her own repository...
but this will be significantly slower than the volume method.

Last but not least, those storage folders need to exist with the right permissions
before you run your docker-compose or it will fail.

To be sure your volumes have the right permissions, you can run those commands:

   ```bash
   $ chown 100:101 ./volumes -R
   $ chown 999:999 ./volumes/db -R
   $ chown 999:999 ./volumes/logs/db -R
   ```

In other words... all the content inside volumes will be owned by apache2 user
and group (uid 100 and gid 101 inside the container), execpt for db and logs/db
folders that will be owned by mysql user and group (uid and gid 999).


## Built in scripts

The Dockerfile includes some scritps at "/usr/local/bin" to facilitate common opperations:

| Script               | Container  | Description                                                                                                   |
|:---------------------|:----------:|:--------------------------------------------------------------------------------------------------------------|
| ojs-run-scheduled    | ojs        | Runs "php tools/runScheduledTasks.php". Called by cron every hour.                                            |
| ojs-cli-install      | ojs        | Uses curl to call the ojs install using pre-defined variables.                                                |
| ojs-pre-start        | ojs        | Enforces some config variables and generates a self-signed cert based on ServerName.                          |
| ojs-upgrade          | ojs        | Runs "php tools/upgrade.php upgrade". (issue when config.inc.php is a volume)                                 |
| ojs-variable         | ojs        | Replaces the variable value in config.inc.php (ie: ojs-variable variable newValue)                            |
| ojs-migrate          | ojs        | Takes a dump.sql, public and private files from "migration" folder and builds and builds a docker site (beta) |

Some of those scripts are still beta, you be careful when you use them.

You can call the scripts outside the container as follows:

   ```bash
   $ docker exec -it ojs_app_journalname /usr/local/bin/ojs-variable session_check_ip Off
   ```
## Apache2

As said, right now the only avaliable stack is Apache2, so configuration files
and volumes are thought you will work over an apache.

If you want to know the fastest method to set your own config jump to the next
section **["Easy way to change config stuff"](#easy-way-to-change-config-stuff)**.

For those who like to understand what happens behind the courtains, you need to
know that during the image building we ask the Dockerfile to copy the content of
`./templates/webServers/apache/phpVersion/root` folder in the root
of your container.

It means, if you like to add your specific php settings you can do it
creating a PHP configuration in `./templates/webServers/php{5,7,73}/conf.d/0-ojs.ini`
and build you own image locally.

There are some optimized variables already, you can check them within each
version directory, e.g. `/root/etc/apache2/conf.d/ojs.conf` with your virtualhost
or `/etc/supervisor.conf`... or, now you know what Dockerfile will do, you
can add your own.

**Again:** Note that template's /root folder is copied into the Docker image
**at build time** so, if you change it, you must rebuild the image for changes
to take effect.

## Update the compose configurations and Dockerfiles

If you want to join the docker-ojs (aka. docker4ojs) team you will like to
contribute with new Dockerfiles or docker-composes. To do this, you will need
to clone this project and send us a PR with your contributions.

Before we follow, let us clarify something:
Versions of this project fit with OJS tag versions, BUT...
if you take a look to PKP's repositories, you will notice the tag naming
syntax changed at version 3.1.2-0. We moved from "ojs-3_1_2-0" to "3_1_2-0".

So, to avoid keeping in mind at what momment we changed the syntax, in this
project we only use the modern syntax, so you cant/must refer every version
with the same notation.

Said this, let's talk about how we generate all this versions:

The files for the different OJS versions and software stacks are generated based
on a number of template files, see directory `templates`.

You can re-generate all `Dockerfile`, `docker-compose(-local).yml`, configuration
files, etc. calling the `build.sh` script.

```bash
# generate a specific version
$ ./build.sh 3_1_2-4

# generate _all_ versions
$ ./build.sh
```
## License

GPL3 © [PKP](https://github.com/pkp)
