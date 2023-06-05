# traefik-docker-local
config for using traefik in local env and automatic discovery

# https
- for https generate keys with mkcert
- first generate keys with
  mkcert *.localhost *.docker.localhost 127.0.0.1 ::1
- install key in store with mkcert -install

## help
- if on WSL do install the same key on both instances.
- on windows the keys will be located here : C:\Users\<username>\AppData\Local\mkcert
you can create a directory on wsl side like this : /home/<username>/.local/share/mkcert
- install mkcert on windows with 
  https://chocolatey.org/install#individual
  choco install mkcert
- install mkcert on ubuntu with 
  wget https://github.com/FiloSottile/mkcert/releases/download/v1.4.4/mkcert-v1.4.4-linux-amd64
  mv mkcert-v1.4.4-linux-amd64 mkcert
  chmod +x mkcert
  cp mkcert /usr/local/bin/
