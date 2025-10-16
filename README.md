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
    environment:
      DOLLI_DB_HOST: ${DB_SERVER}
      DOLLI_DB_USER: ${DB_USER}
      DOLLI_DB_NAME: "dolibarrDB"
      DOLLI_DB_PASSWORD: ${DB_PASSWD}
      DOLLI_ADMIN_LOGIN: ${DOLLI_ADMIN_LOGIN}
      DOLLI_ADMIN_PASSWORD: ${DOLLI_ADMIN_PASSWORD}
      DOLLI_INSTALL_AUTO: 1
      DOLLI_COMPANY_NAME: "Intento de empresa"
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
Aqui introduzco la parte del docker compose que hace referencia a dolibarr 
