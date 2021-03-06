# HTTPS con Docker y Docker-compose

Instalando una aplicación web y añadiendo un certificado HTTPS con Docker.

## Preparando la máquina

Necesitamos una máquina con al menos 2GB de RAM y con los puertos HTTP(80), HTTPS(443), MySQL(3306) y el 8080 abiertos.

## Preparando nuestro dominio

Creamos el nombre de dominio, vamos a la página [Freenom](https://www.freenom.com/es/index.html?lang=es), nos registramos e iniciamos sesión, cuando estemos logueados vamos a Servicios>Registrar nuevo dominio. Se nos abrirá una nueva página en la que ingresaremos el nombre de dominio que queremos crear y pulsaremos el botón para comprobar la disponibilidad.

Si está libre, podremos elegir una lista de dominios, cuando seleccionemos el que queremos, en la parte superior saldrá que tenemos un dominio añadido junto con un botón llamado checkout, le damos y pasamos a la pantalla de "compra". Sobre el tiempo que estará el dominio disponible, seleccionamos 1 mes ya que esto es una prueba. 

Le damos a continuar, en la pantalla de confirmación aceptamos los términos y condiciones de uso, y le damos a completar encargo.

Cuando termine este proceso, volvemos a la ventana de servicios y vamos a Mis dominios, dentro veremos que ya podemos utilizar el dominio que hemos reservado antes, vamos a sus opciones (botón Manage Domain), una vez dentro vamos a la sección Manage Freenom DNS.

En la nueva página que se nos abre, deberemos de crear 2 registros DNS de tipo A que apunten a nuestra máquina de Amazon, en mi caso los registros han quedado de la siguiente manera.

![Imagen de demostracion 1](/capturas/captura1.png)

## Trabajando con Docker y Docke-compose

Actualizamos los repositorios del sistema e instalamos docker y docker-compose y lo configuramos igual que hemos hecho en prácticas anteriores.

Cuando hayamos hecho este paso crearemos el archivo YML, este archivo consta de lo siguiente:

```
version: '3.4'

services:
  mysql:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    ports: 
      - 3306:3306
    environment: 
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    volumes: 
      - mysql_data:/var/lib/mysql
    networks: 
      - backend-network
    restart: always
  
  phpmyadmin:
    image: phpmyadmin
    ports:
      - 8080:80
    environment: 
      - PMA_ARBITRARY=1
    networks: 
      - backend-network
      - frontend-network
    restart: always
    depends_on: 
      - mysql

  prestashop:
    image: prestashop/prestashop
    environment: 
      - DB_SERVER=mysql
    volumes:
      - prestashop_data:/var/www/html
    networks: 
      - backend-network
      - frontend-network
    restart: always
    depends_on: 
      - mysql

  https-portal:
    image: steveltn/https-portal:1
    ports:
      - 80:80
      - 443:443
    restart: always
    environment:
      DOMAINS: 'rcapsecuredocker.tk -> http://prestashop:80'
      #STAGE 'production' # Utilizar cuando el sitio esté listo
      STAGE: 'staging' # Utilizar para pruebas
      # FORCE_RENEW: 'true'
    networks:
      - frontend-network

volumes:
  mysql_data:
  prestashop_data:

networks: 
  backend-network:
  frontend-network:
```

El archivo .yml estará disponible en el repositorio.

Es importante que hasta que no tengamos claro que no tengamos que nuestro contenedor está listo para lanzarlo de forma profesional a Internet tenemos que utilizar el STAGE de pruebas denominado "staging", cuando tengamos claro que nuestra infraestructura está preparada, utilizaremos el stage "production" podemos incluir en el yml varias lineas de STAGE comentadas de cada tipo, así solo tendremos que descomentar la que nos interese utilizar..

El archivo .env con las variables correspondientes estará disponible en el repositorio.

Antes de poner en funcionamiento todos los contenedores, tenemos que asegurarnos de que la linea DOMAINS del servicio HTTPS tiene nuestro dominio puesto.

Teniendo todo listo lanzamos nuestra infraestructura con el comando docker-compose up -d.

Si entramos en el dominio podremos ver que tenemos nuestra aplicación web bajo una conexión SSL.

![Imagen de demostracion 2](/capturas/captura2.png)

Cuando hayamos instalado nuestra aplicación web (en este caso Prestashop) cabe la posibilidad de que tengamos que activar el protocolo SSL desde el panel de administración, si no nos deja, podemos hacerlo modificando la base de datos. Vamos al PhpMyAdmin y ejecutamos las siguientes instrucciones en el apartado SQL o modificando los valores de forma gráfica.

```
UPDATE ps_configuration SET value=1 WHERE name="PS_SSL_ENABLED";

UPDATE ps_configuration SET value=1 WHERE name="PS_SSL_ENABLED_EVERYWHERE";
```

Con estos cambios hechos si entramos a nuestro sitio web Prestashop podremos observar que ya es un sitio completamente seguro.

**En cada aplicación web este proceso puede ser distinto.**

## Enlace a mi sitio prestashop

- **[rcapsecuredocker.tk](https://rcapsecuredocker.tk)**
