# traefik-docker-local
Use traefik in local and automatic discovery for docker projects.

# Classic Usage
## 1 - Create a network
First create a network that will be used by docker projects.

just type for example : 
``` sh
docker network create traefik
```

## 2 - Start traefik
just type 
``` sh
docker compose up -d
```

then traefik web interface will be available at ``traefik.docker.localhost``

## 3 - Modify your docker compose file
in your docker compose file:
add these labels to exposed service (like caddy, apache, mailer ...)
and the network.

**Replace {SERVICE-NAME} {SERVICE-URL} and {INTERNAL_PORT_OF_SERVICE} with the correct values.**

``` yaml
services:
    webserver:
        image: my-server-image  
        labels:
            - traefik.enable=true
            - traefik.http.routers.{SERVICE-NAME}.rule=Host(`{SERVICE-URL}.docker.localhost`)
            - traefik.http.services.{SERVICE-NAME}.loadbalancer.server.port={INTERNAL_PORT_OF_SERVICE}
        networks:
            - default // you need to add this in order to keep the default network for your stack
            - traefik
```
you can have a service being accessible by multiple name :
``` yaml
services:
    webserver:
        image: my-server-image  
        labels:
            - traefik.enable=true
            - traefik.http.routers.{SERVICE-NAME}.rule=Host(`{SERVICE-URL}.docker.localhost`) || Host(`{SERVICE-URL-2}.docker.localhost`)
            - traefik.http.services.{SERVICE-NAME}.loadbalancer.server.port={INTERNAL_PORT_OF_SERVICE}
        networks:
            - default // you need to add this in order to keep the default network for your stack
            - traefik
```

then and add the network you created in the docker compose file :
``` yaml
networks:
    default: // you need to add this in order to keep the default network for your stack
    traefik:
        external: true
```

## 4 - Update vhost in your projects
Remember to update the vhost of your projects to use the {SERVICE-URL} you mentioned in your compose files.

## 5 - Access your project
Your project will be accessible via the {SERVICE-URL}.


# Serve other web server (e.g caddy, apache, nginx)
If you need to install another web server like caddy for example, just make it work on another port (for example 81 instead of 80 and 441 instead of 443)
then in your traefik-dynamic.yml update :
``` yaml
http:
    routers:
        test-router:
            rule: Host(`other-service.localhost`)  # Définir la règle de routage pour l'adresse other-service.localhost
            service: caddy-service  # Rediriger vers le service Caddy
            entryPoints:
                - web  # Traefik utilise l'entrée "web" qui écoute sur le port 80 (configuré dans traefik.yml)

    services:
        caddy-service:
            loadBalancer:
                servers:
                    -   url: "http://localhost:81"  # Caddy écoute sur localhost:81
```
and you need to update /etc/hosts as you would usually do without traefik.
