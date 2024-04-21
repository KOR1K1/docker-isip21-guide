## Вступление

### Иерархия проекта выглядит так:
- 📁 docker
    - 📁 nginx
        - 📁 conf.d
            - ⚙ default.conf
- 📁 php
    - ↪ Файлы сайта
- ↪ docker-compose.yml
- ↪ Dockerfile

### docker-compose.yml
```yml
version: '3'
services:
  php:
    build:
      context: .
      dockerfile: ./Dockerfile
    volumes:
      - "./php:/var/www/html"

  nginx:
    image: nginx:latest
    ports:
      - 80:80
    volumes:
      - "./php:/var/www/html"
      - "./docker/nginx/conf.d/:/etc/nginx/conf.d"

  mysql:
    image: mysql:8
    environment:
      - MYSQL_ROOT_PASSWORD=root
    volumes:
      - mysql-data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin
    restart: always
    ports:
      - 1337:80
    environment: 
      - PMA_ARBITRARY=1
volumes:
  mysql-data:
    external: true
```

### Dockerfile
```dockerfile
FROM php:7.4-fpm


RUN docker-php-ext-install mysqli \
    && docker-php-ext-enable mysqli
```

### default.conf
```
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        root           /var/www/html;
        fastcgi_pass   php:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

### content.php
```php
<div id="content">
<?php
/* Заменить данные о бд */
$nameDB = "phptest";
$nameSERVER = 'mysql';
$nameUSER = 'root';
$passUSER = 'root';
$db = mysqli_connect($nameSERVER, $nameUSER, $passUSER,$nameDB);
/*if (!$db) {
   die("Connection failed: " . mysqli_connect_error());
}
echo "Connected successfully";
mysqli_close($db);*/
mysqli_set_charset($db,'utf-8');
 
$c=$_GET['action'];

if ($c=="") $c=1;
$query = "SELECT page FROM content WHERE id=$c";
$result = mysqli_query($db,$query);
$myrow = mysqli_fetch_array($result);
echo $myrow[page];   
$query1 = "SELECT footer FROM content WHERE id=$c";
$result1 = mysqli_query($db,$query1);
$myrow1 = mysqli_fetch_array($result1);  
?>
</div>
```

### title.php
```php
<?php
/* Также заменить данные бд */
$nameDB = "phptest";
$nameSERVER = "mysql";
$nameUSER = "root";
$passUSER = 'root';
$db = mysqli_connect($nameSERVER, $nameUSER, $passUSER,$nameDB);
mysqli_set_charset($db,'utf-8');
$t=$_GET['action'];
if ($t=="") $t=1;
$query = "SELECT title FROM content WHERE id=$t";
$result = mysqli_query($db,$query);
$myrow = mysqli_fetch_array($result);
echo $myrow[title];     
?>
```

### Далее нужно создать Том для сохранения базы данных после перезапуска контейнера
```shell
sudo docker volume create mysql-data
```

### Далее можно запускать контейнер
```shell
sudo docker-compose up -d
```

> [!Note]
> Для отключения контейнера требуется ввести 
> ```shell
> sudo docker-compose down
> ```

## Настройка базы данных

### Переходим по адресу `localhost:1337` и попадаем в phpMyAdmin

![[Pasted image 20240421224147.png]]
#### Данные для входа:
##### Сервер: mysql
##### Пользователь: root
##### Пароль: root

### Далее создаем бд с названием phptest
![[Pasted image 20240421224316.png]]

### Переходим на вкладку Импорт
![[Pasted image 20240421224435.png]]
#### Выбираем файл 
##### phptest.sql из папки сайта
##### листаем вниз, жмём импорт

## Переход на сайт
### В адресной строке вписываем `localhost/index.php`

### Результат:
![[Pasted image 20240421224806.png]]

#### Дальше я хуй знает как это привести к норм виду
