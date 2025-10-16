# Tarea 7 Optativa

Instalacion de redis

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
