<img src="http://www.nginx.com/wp-content/uploads/2018/08/NGINX-logo-rgb-large.png" align="center" width="250px" height="100px">

# Nginx & PHP & MYSQL & Docker

- [x] Docker Compose
- [x] DockerFile
- [x] Default.conf
- [x] PHP & Conexión a base de datos
- [x] Cliente MYSQL 
- [x] Levantar los contenedores 

## 1. Estructura de Docker Compose

```
services:
  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf
      - ./src:/var/www/html
    links:
      - php-fpm
 php-fpm:
   image: php:8-fpm
   volumes:
    - ./src:/var/www/html
   build:
    context: .
    dockerfile: DockerFile
 mysql:
  image: mysql:8
  ports:
    - "3306"
  environment:
    - MYSQL_ROOT_PASSWORD=passwd
    - MYSQL_PASSWORD=passwd
    - MYSQL_DATABASE=users
```

## 2. Estructura de Dockerfile

```
FROM php:8.fpm

# apt-get update && apt-get upgrade -y
RUN docker-php-ext-install mysqli && docker-php-ext-enable mysqli
```

## 3. Default.conf
```
server {
    index index.php index.html;
    server_name phpfpm.local;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/html;

    location ~ \.php$ {
    try_files $uri =404;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass php-fpm:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

## 4. PHP & Conexión a base de datos

:bulb: **Tip:** Creamos una carpeta o archivo php con el cual realizaremos la conexión a la base de datos. Hay que indicarle al docker compose la ruta de los archivos php.

- index.php

```
<?php
    $servername = "mysql";
    $username = "root";
    $password = "passwd";
    $dbname = "users";

    $conn = new mysqli($servername, $username, $password , $dbname);

    // If CONN failed -> throw error
    if($conn->connect_error){
        die("Connection Failed: " . $conn->connect_error);
    }
    echo "Connection Succesfully" . "<br>";


    $sql = "SELECT * FROM client";
    $result = $conn->query($sql);

    if ($result->num_rows > 0) {
    // output data of each row
    while($row = $result->fetch_assoc()) {
        echo "id: " . $row["id"]. " - Name: " . $row["username"]. " " . $row["email"]. "<br>";
    }
    } else {
    echo "0 results";
    }
    
    $conn->close();
?>

```

## 5. Cliente MYSQL

:bulb: **Tip:** Habria que utilizar un cliente de mysql para introducir datos y obtener el resultado en el servicio web. Tendremos que realizar la conexión a la base de datos y posteriormente hacer las queries para crear una tabla y añadir los campos y datos. Podemos utilizar un servicio de Docker con interfaz gráfica para hacerlo mas rápidamente, se llama "adminer".


## 6. Levantar los contenedores

📝: **Nota:** Antes de hacer nada, nos tenemos que asegurar de que tenemos Docker Desktop encendido y listo para levantar los contenedores.

`docker compose up -d --build`
