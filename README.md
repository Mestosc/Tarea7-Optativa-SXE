# Tarea 7 Optativa

## Instalacion de redis

```yaml
redis:
    image: redis
    container_name: central-redis
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
    ports:
      - 6379:6379
    networks:
      - prestashop_network
    volumes:
      - redis_data:/data

```
Añadimos esta linea la usamos para desplegar un contenedor de redis en la misma red de prestashop 

He buscado en la tienda de modulos de prestashop y existe un modulo para usar redis como sistema de cache necesitamos usar un modulo de prestashop como [Redis Cache](https://addons.prestashop.com/es/rendimiento-sitio-web/50661-redis-cache.html)

<img width="1297" height="1650" alt="imagen" src="https://github.com/user-attachments/assets/8c576ebb-e9c1-433f-abd5-86de3e0fdde8" />


## Instalacion de dolibarr
```yaml
  dolibarr:
    image: dolibarr/dolibarr:latest
    container_name: dolibarr_container
    environment:
      DOLI_DB_HOST: ${DB_SERVER}
      DOLI_DB_USER: ${DB_USER}
      DOLI_DB_NAME: ${DOLI_DB_NAME}
      DOLI_DB_PASSWORD: ${DB_PASSWD}
      DOLI_ADMIN_LOGIN: ${DOLI_ADMIN_LOGIN}
      DOLI_ADMIN_PASSWORD: ${DOLI_ADMIN_PASSWORD}
      DOLI_INSTALL_AUTO: 1
      DOLI_COMPANY_NAME: "Intento de empresa"
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
    ports:
      - 9000:80
    networks:
      - prestashop_network
    volumes:
      - dolibarr_docs:/var/www/documents
      - dolibarr_custom:/var/www/html/custom
```
Aqui introduzco la parte del docker compose que hace referencia a dolibarr, en el caso de dolibarr tuve que crear la base de datos manualmente mediante phpMyAdmin porque sino daba error de que no encontraba la base de datos y no salia de ahi, despues de hacer eso y ejecutar dolibarr sin ninguno de los volumenes creados y sin nada almacenado, aparece lo siguiente

<img width="2736" height="1657" alt="imagen" src="https://github.com/user-attachments/assets/13c0adb9-a210-4298-81d0-7cc446a6a13d" />

<img width="2743" height="1593" alt="imagen" src="https://github.com/user-attachments/assets/dcdc8048-71a3-4e40-a22b-4bf0ea436120" />

# Archivo final
Aqui dejare los ``docker-compose.yml`` y ``.env`` de forma total
```yml
services:
  redis:
    image: redis
    container_name: central-redis
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
    ports:
      - 6379:6379
    networks:
      - prestashop_network
    volumes:
      - redis_data:/data
  dolibarr:
    image: dolibarr/dolibarr:latest
    container_name: dolibarr_container
    environment:
      DOLI_DB_HOST: ${DB_SERVER}
      DOLI_DB_USER: ${DB_USER}
      DOLI_DB_NAME: ${DOLI_DB_NAME}
      DOLI_DB_PASSWORD: ${DB_PASSWD}
      DOLI_ADMIN_LOGIN: ${DOLI_ADMIN_LOGIN}
      DOLI_ADMIN_PASSWORD: ${DOLI_ADMIN_PASSWORD}
      DOLI_INSTALL_AUTO: 1
      DOLI_COMPANY_NAME: "Intento de empresa"
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
    ports:
      - 9000:80
    networks:
      - prestashop_network
    volumes:
      - dolibarr_docs:/var/www/documents
      - dolibarr_custom:/var/www/html/custom

  mysql:
    container_name: prestashop-mysql
    image: mysql:5.7
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 10s      # cada cuánto ejecutar el chequeo
      timeout: 5s        # cuánto esperar por una respuesta
      retries: 5         # cuántos fallos seguidos antes de marcar como unhealthy
      start_period: 20s  # espera inicial antes de empezar los chequeos
    networks:
      - prestashop_network
    volumes:
      - dbdata:/var/lib/mysql
  prestashop:
    container_name: prestashop
    image: prestashop/prestashop:latest
    restart: unless-stopped
    depends_on:
      mysql:
       condition: service_healthy
      redis:
        condition: service_healthy
    ports:
      - 8080:80
    environment:
      PS_LANGUAGE: ${PS_LANGUAGE}
      PS_COUNTRY: ${PS_COUNTRY}
      DB_SERVER: ${DB_SERVER}
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWD: ${DB_PASSWD}
      PS_INSTALL_AUTO: ${PS_INSTALL_AUTO}
      PS_FOLDER_ADMIN: ${PS_FOLDER_ADMIN}
      PS_FOLDER_INSTALL: ${PS_FOLDER_INSTALL}
      ADMIN_MAIL: ${ADMIN_MAIL}
      ADMIN_PASSWD: ${ADMIN_PASSWD}
    networks:
      - prestashop_network
    volumes:
      - psdata:/var/www/html
  phpmyadmin:
    image: phpmyadmin
    restart: always
    ports:
      - 8090:80
    depends_on:
     mysql:
      condition: service_healthy
    environment:
      PMA_HOST: ${DB_SERVER}
      PMA_ARBITRARY: ${PMA_ARBITRARY}
      PMA_USER: ${DB_USER}
      PMA_PASSWORD: ${DB_PASSWD}
    networks:
      - prestashop_network
networks:
    prestashop_network:
volumes:
    dbdata:
    psdata:
    redis_data:
    dolibarr_docs:
    dolibarr_custom:
```
## Fichero .env
```sh
MYSQL_ROOT_PASSWORD=admin
MYSQL_DATABASE=prestashop
DB_SERVER=prestashop-mysql
DB_USER=root
DB_PASSWD=admin
PMA_ARBITRARY=1
PS_INSTALL_AUTO=1
PS_LANGUAGE=es
PS_COUNTRY=es
DB_NAME=prestashop
ADMIN_MAIL=admin@prestashop.local
ADMIN_PASSWD=admin123
PS_FOLDER_ADMIN=admin4577
PS_FOLDER_INSTALL=install4577
DOLI_ADMIN_LOGIN=admin
DOLI_ADMIN_PASSWORD=admin123
DOLI_DB_NAME=dolibarrbd
```

