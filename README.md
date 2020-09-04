## Como configurar Laravel, Nginx, e MySQL com Docker Compose
## Passo 1 — Fazendo download do Projeto e instalando dependências

Primeiramente, verifique se você está no seu diretório home e faça uma cópia da versão mais recente do Projeto para um diretório chamado laravel-app:

    $ cd ~
    $ git clone https://github.com/laravel/laravel.git laravel-app

Vá até o diretório laravel-app:

    $ cd ~/laravel-app

Em seguida, utilize a imagem do composer para montar os diretórios que você precisará para seu projeto:

    $ docker run --rm -v $(pwd):/app composer install

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
