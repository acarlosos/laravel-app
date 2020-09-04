## Como configurar Laravel, Nginx, e MySQL com Docker Compose
## Passo 1 — Fazendo download do Projeto e instalando dependências

Primeiramente, verifique se você está no seu diretório home e faça uma cópia da versão mais recente do Projeto para um diretório chamado laravel-app:

    $ cd ~
    $ git clone https://github.com/acarlosos/laravel-app laravel-app

Vá até o diretório laravel-app:

    $ cd ~/laravel-app

Em seguida, utilize a imagem do composer para montar os diretórios que você precisará para seu projeto:

    $ docker run --rm -v $(pwd):/app composer install

Como passo final, defina as permissões no diretório do projeto para que ele seja propriedade do seu usuário não root:

    sudo chown -R $USER:$USER ~/laravel-app

## Passo 2 - Modificando as configurações do ambiente e executando os contêineres

Como passo final, porém, vamos fazer uma cópia do arquivo .env.example que o Laravel inclui por padrão e nomear a copia .env, que é o arquivo que o Laravel espera para definir seu ambiente:

    $ cp .env.example .env

Você pode agora modificar o arquivo .env no contêiner app para incluir detalhes específicos sobre sua configuração.

    $ nano .env

Encontre o bloco que especifica o DB_CONNECTION e atualize-o para refletir as especificidades da sua configuração. Você modificará os seguintes campos:

O DB_HOST será seu contêiner de banco de dados db.
O DB_DATABASE será o banco de dados laravel.
O DB_USERNAME será o nome de usuário que você usará para o seu banco de dados. Neste caso, vamos usar laraveluser.
O DB_PASSWORD será a senha segura que você gostaria de usar para esta conta de usuário, vamos usar secret.

```/var/www/.env```
```
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laraveluser
DB_PASSWORD=secret
```

Salve suas alterações e saia do seu editor.

Com todos os seus serviços definidos no seu arquivo docker-compose, você precisa emitir um único comando para iniciar todos os contêineres, criar os volumes e configurar e conectar as redes:

    $ docker-compose up -d

Assim que o processo for concluído, utilize o comando a seguir para listar todos os contêineres em execução:
    $ docker ps

Você verá o seguinte resultado com detalhes sobre seus contêineres do app, webserver e db:

```Output```

```
CONTAINER ID        NAMES               IMAGE                             STATUS              PORTS
c31b7b3251e0        db                  mysql:5.7.22                      Up 2 seconds        0.0.0.0:3306->3306/tcp
ed5a69704580        app                 digitalocean.com/php              Up 2 seconds        9000/tcp
5ce4ee31d7c0        webserver           nginx:alpine                      Up 2 seconds
```

Usaremos agora o docker-compose exec para definir a chave do aplicativo para o aplicativo Laravel.
Este comando gerará uma chave e a copiará para seu arquivo .env, garantindo que as sessões do seu usuário e os dados criptografados permaneçam seguros:

    $ docker-compose exec app php artisan key:generate

Para colocar essas configurações em um arquivo de cache, que irá aumentar a velocidade de carregamento do seu aplicativo, execute:

    $ docker-compose exec app php artisan config:cache

Suas definições da configuração serão carregadas em /var/www/bootstrap/cache/config.php no contêiner.

Como passo final, visite http://localhost no navegador. 
Você verá a seguinte página inicial para seu aplicativo Laravel:

<img src="https://assets.digitalocean.com/articles/laravel_docker/laravel_home.png" >


## Passo 3 - Criando um usuário para o MySQL

Para criar um novo usuário, execute uma bash shell interativa no contêiner db com o docker-compose exec:

    $ docker-compose exec db bash

Dentro do contêiner, logue na conta administrativa root do MySQL:

    root@6efb373db53c:/# mysql -u root -p

Você será solicitado a inserir a senha para a conta root do MySQL ( secret ).

    mysql> show databases;

Você verá o banco de dados laravel listado no resultado:

```
Output
+--------------------+
| Database           |
+--------------------+
| information_schema |
| laravel            |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

Em seguida, crie a conta de usuário que terá permissão para acessar esse banco de dados.

    mysql> GRANT ALL ON laravel.* TO 'laraveluser'@'%' IDENTIFIED BY 'secret';

Reinicie os privilégios para notificar o servidor MySQL das alterações:

    mysql> FLUSH PRIVILEGES;

Saia do MySQL:

    mysql>exit;

Por fim, saia do contêiner:

    root@6efb373db53c:/# exit

## Passo 4 - Migrando dados e teste com o console Tinker

Primeiramente, teste a conexão com o MySQL executando o comando Laravel artisan migrate, que cria uma tabela migrations no banco de dados de dentro do contêiner:

    $ docker-compose exec app php artisan migrate

Este comando irá migrar as tabelas padrão do Laravel. O resultado que confirma a migração será como este:

```
Output

Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table
```

Assim que a migração for concluída, você pode fazer uma consulta para verificar se está devidamente conectado ao banco de dados usando o comando tinker:

    $ docker-compose exec app php artisan tinker

Teste a conexão do MySQL obtendo os dados que acabou de migrar:

    >>> \DB::table('migrations')->get();

Você verá um resultado que se parece com este:

```
Output
=> Illuminate\Support\Collection {#2856
     all: [
       {#2862
         +"id": 1,
         +"migration": "2014_10_12_000000_create_users_table",
         +"batch": 1,
       },
       {#2865
         +"id": 2,
         +"migration": "2014_10_12_100000_create_password_resets_table",
         +"batch": 1,
       },
     ],
   }
```


