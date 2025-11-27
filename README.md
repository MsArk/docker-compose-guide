# 🐳 Guía de Organización Perfecta para `docker-compose.yml`

Este documento define un **estándar completo, claro y mantenible** para
estructurar un archivo `docker-compose.yml` en cualquier proyecto. Sigue
este formato para obtener un stack ordenado, escalable y fácil de
depurar.

------------------------------------------------------------------------

## 📌 1. Estructura general recomendada

El orden top-level debe seguir esta secuencia:

    version: "3.9"        # opcional con Compose Spec, pero recomendado
    name: proyecto-x       # o usa -p durante ejecución

    # anchors opcionales para reutilizar bloques
    x-env: &default-env
      TZ: Europe/Madrid

    services:
      # servicios aquí

    networks:
      # redes

    volumes:
      # volúmenes

    secrets:
      # secretos

    configs:
      # configs (Swarm)

------------------------------------------------------------------------

## 📌 2. Orden recomendado de servicios

El archivo debe reflejar una lógica **de exterior → interior**:

1.  **Proxy / Entrada**
2.  **Bases de datos y almacenamiento**
3.  **Servicios core**
4.  **Herramientas auxiliares**

------------------------------------------------------------------------

## 📌 3. Estructura perfecta dentro de cada servicio

Cada servicio debe seguir siempre este orden:

    services:
      backend:
        container_name: backend
        image: my-org/backend:latest
        # build:
        #   context: .
        #   dockerfile: Dockerfile

        hostname: backend
        command: ["npm", "run", "start"]
        entrypoint: ["docker-entrypoint.sh"]

        depends_on:
          db:
            condition: service_healthy
          redis:
            condition: service_started

        env_file:
          - .env
        environment:
          <<: *default-env
          NODE_ENV: "production"
          DB_HOST: db

        secrets:
          - db_password

        configs:
          - app_config

        volumes:
          - backend-data:/var/www/data
          - ./config/backend.yml:/app/config.yml:ro

        ports:
          - "8080:3000"
        expose:
          - "3000"

        networks:
          - frontend
          - backend

        healthcheck:
          test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
          interval: 30s
          timeout: 5s
          retries: 5
          start_period: 10s

        restart: unless-stopped

        logging:
          driver: json-file
          options:
            max-size: "10m"
            max-file: "3"

        labels:
          traefik.enable: "true"
          traefik.http.routers.backend.rule: Host(`api.ejemplo.com`)
          traefik.http.services.backend.loadbalancer.server.port: "3000"

        extra_hosts:
          - "host.docker.internal:host-gateway"

------------------------------------------------------------------------

## 📌 4. Redes, volúmenes, secretos y configs

    networks:
      frontend:
        driver: bridge
      backend:
        driver: bridge
        internal: true

    volumes:
      backend-data:
      db-data:
      redis-data:

    secrets:
      db_password:
        file: ./secrets/db_password.txt

    configs:
      app_config:
        file: ./configs/app.yml

------------------------------------------------------------------------

## 📌 5. Checklist para un `docker-compose` profesional

-   Usar `image` o `build`, no ambos\
-   Definir `depends_on` con `condition: service_healthy`\
-   Mantener variables en `.env`\
-   Usar healthchecks\
-   Separar redes\
-   Evitar `ports:` en servicios internos\
-   Controlar logs\
-   Reinicios correctos

------------------------------------------------------------------------

## 📌 6. Por qué seguir este estándar

-   Lectura más clara\
-   Evita errores de arranque\
-   Minimiza exposición de variables\
-   Facilita migraciones\
-   Perfecto para microservicios
