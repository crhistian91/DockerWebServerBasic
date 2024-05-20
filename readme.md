# Configuración de Docker Compose para Nginx, PHP-FPM, MySQL y phpMyAdmin

Este repositorio contiene una configuración de Docker Compose para ejecutar un servidor web con Nginx, PHP-FPM, MySQL y phpMyAdmin. La configuración está diseñada para soportar múltiples sitios web alojados en diferentes dominios.

## Servicios

- **Nginx**: Servidor web para servir páginas web y proxy inverso para PHP-FPM.
- **PHP-FPM**: Gestor de procesos FastCGI para PHP para manejar las solicitudes PHP.
- **MySQL**: Sistema de gestión de bases de datos relacional para almacenar datos de la aplicación.
- **phpMyAdmin**: Herramienta de administración web para MySQL.

## Requisitos Previos

- Docker instalado en tu sistema.
- Docker Compose instalado en tu sistema.
- Nombres de dominio válidos que apunten a la dirección IP de tu servidor.

## Configuración

### Docker Compose

El archivo `docker-compose.yml` configura los servicios y sus dependencias.

```yaml
version: '3.9'

services:
  loadbalancer:
    container_name: loadbalancer
    image: loadbalancer:${APP_VERSION}
    build:
      context: ./
      dockerfile: Dockerfile.nginx.prd
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./volumes/nginx/certs:/etc/nginx/certs
      - ./volumes/nginx/templates:/etc/nginx/templates
      - ./volumes/nginx/webapps:/var/www/html
      - ./volumes/nginx/logs:/var/log/nginx
    environment:
      - NGINX_ENVSUBST_OUTPUT_DIR=/etc/nginx/
    command: [nginx, '-g', 'daemon off;']
    depends_on:
      - php-fpm
      - mysql
    networks:
      - net

  php-fpm:
    image: php:8.0-fpm
    restart: always
    volumes:
      - ./volumes/nginx/webapps:/var/www/html
      - ./config/php:/usr/local/etc/php
    depends_on:
      - mysql
    networks:
      - net

  mysql:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: CRBqsplaz#2024$
    volumes:
      - ./volumes/mysql_data:/var/lib/mysql
    networks:
      - net

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    restart: always
    ports:
      - 8080:80
    environment:
      PMA_HOST: mysql
      MYSQL_ROOT_PASSWORD: CRBqsplaz#2024$
    depends_on:
      - mysql
    networks:
      - net

networks:
  net:
    driver: bridge
