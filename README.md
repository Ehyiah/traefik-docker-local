# traefik-docker-local
Use traefik in local and automatic discovery for docker projects.

# Usage
## 1 - Create a network
Create a network that will be used by docker projects.

``` sh
    docker network create traefik
```

## 2 - Start traefik
``` sh
    docker compose up -d
```

then traefik web interface will be available at ``traefik.docker.localhost``

## 3 - Modify your projects docker compose file

in your docker compose file: 2 steps are needed
### 1. add these labels to exposed service (like caddy, apache, mailer ...)

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

### 2. and the networks.

then and add the network you created in the docker compose file :
``` yaml
networks:
    default: // you need to add this in order to keep the default network for your stack
    traefik:
        external: true
```

## 4. Update vhost in your projects
Remember to update the vhost of your projects to use the {SERVICE-URL} you mentioned in your compose files.

## 5. Access your project
Your project will be accessible via the {SERVICE-URL}



## 6. Optional Adding https:
Create certificate
- mkcert -cert-file certs/local-cert.pem -key-file certs/local-key.pem "docker.localhost" "*.docker.localhost" "domain.local" "*.domain.local"
- _Reminder: X.509 wildcards only go one level deep, so this won't match a.b.docker.localhost_

In your project update labels
``` yaml
    labels:
      - traefik.enable=true // should already be existing il you followed previous steps
      - traefik.http.routers.{SERVICE-NAME}.rule=Host(`project.docker.localhost`) // should already be existing il you followed previous steps
      - traefik.http.services.{SERVICE-NAME}.loadbalancer.server.port=80 // should already be existing il you followed previous steps
      // TLS related lines
      - traefik.http.routers.{SERVICE-NAME}-tls.rule=Host(`project.docker.localhost`) // add these lines on services you want to be https exposed
      - traefik.http.routers.{SERVICE-NAME}-tls.tls=true // add these lines on services you want to be https exposed
      - traefik.http.routers.{SERVICE-NAME}-tls.entrypoints=webSecure // add these lines on services you want to be https exposed
```



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
