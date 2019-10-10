Stack de docker preparado para lanzar rápidamente un entorno de desarrollo para drupal 7/8

__Instalación__

1. Descárgate o clona el repositorio y copia tódo el contenido a la raíz de tu proyecto drupal

2. Edita el fichero .env si necesitas modificar puertos usuarios o contraseñas de los servicios

3. Edita docker-compose.yml si necesitas añadir o quitar servicios. Por defecto se lanzan Apache2 con php 7.2, Mysql 5.7, Redis 2.8 y Mailhog.

4. Por último, si quieres modificar los hosts donde se sirve drupal puedes editar el fichero docker/web/vhosts/docker.conf

5. Lanza docker compose: docker-compose up -d

Si no has modificado la config de los vhosts, verás tu entorno en http://drupal.localhost o https://drupal.localhost

__Funcionalidades__

- Apache2 + Php 7.1/7.2/7.3
- Adminer (http://localhost) para gestionar bbdd
- Revisión de logs visual mediante pimpmylog en http://logs.localhost

__Como usar adminer__

Adminer responde a la conexión por defecto en http://localhost. Para conectarnos a la BBDD usar estos datos:
*Server*: mysql
*Username*: docker
*Password*: docker


__CHANGELOG__

10-10-2019
  - Añadida definitivamente imagen de pimpmylog para ver los logs desde http://logs.localhost
  - Habilitado adminer en http://localhost para poder suprimir contenedor de phpmyadmin
  - Aumentada memory_limit a 512M y max_input_vars a 7000
  - Limpieza de ficheros sobrantes
