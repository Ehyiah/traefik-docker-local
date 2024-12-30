# traefik-docker-local
Use traefik in local and automatic discovery for docker projects.

# Usage
## Create a network
First of all create a network that will be used by docker projects.

just type for example : 
``` sh
docker network create traefik
```

## Start traefik
just type 
``` sh
docker compose up -d
```

then traefik web interface should be available at ``traefik.docker.localhost``

## Modify your docker compose file
in your docker compose file :
add these labels to exposed service (like caddy, apache, mailer ...)
and the network : 

Replace {SERVICE-NAME} {SERVICE-URL} and {INTERNAL_PORT_OF_SERVICE} with the correct values.

``` yaml
services:
    webserver:
        image: my-server-image  
        labels:
            - traefik.enable=true
            - traefik.http.routers.{SERVICE-NAME}.rule=Host(`{SERVICE-URL}.docker.localhost`)
            - traefik.http.services.{SERVICE-NAME}.loadbalancer.server.port={INTERNAL_PORT_OF_SERVICE}
        networks:
            - traefik
```
you can have a service being accessible by multiple name :
``` yaml
services:
    webserver:
        image: my-server-image  
        labels:
            - traefik.enable=true
            - traefik.http.routers.{SERVICE-NAME}.rule=Host(`{SERVICE-URL}.docker.localhost`, `{SERVICE-URL-2}.docker.localhost`)
            - traefik.http.services.{SERVICE-NAME}.loadbalancer.server.port={INTERNAL_PORT_OF_SERVICE}
        networks:
            - traefik
```

then and add the network you created in the docker compose file :
``` yaml
networks:
  traefik:
    external: true
```

## Update vhost in your projects
Don't forget to update the vhost of your projects to use the {SERVICE-URL} you mentioned in your compose files.

## Access your project
Your project will be accessible via the {SERVICE-URL}.
