# traefik-docker-local
Use traefik in local and automatic discovery for docker projects.

# Usage
## Create a network
create a network that will be used by docker projects.
example : ``docker network create traefik``

## Modify your docker compose file
in your docker compose file :
add these labels to exposed service (like caddy, apache, mailer ...)
and the network : 

Replace {SERVICE-NAME} and {INTERNAL_PORT_OF_SERVICE} with the correct values.

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

then and add the network you created in the docker compose file :
``` yaml
networks:
  traefik:
    external: true
```
