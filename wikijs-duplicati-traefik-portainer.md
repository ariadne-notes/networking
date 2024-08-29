```
#
# This is the config for portainer, and the reverse proxy, traefik
#

#
# This is a VM (grove) that hosts portainer. These are services started by docker compose
#
# sudo docker comopose up -d
# sudo docker compose down
#
# the network user-bridge needs to be specified in advance
#
# My wiki host is wiki.<mydomain>.org
# My wiki backup host is wiki-backup.<mydomain>.org
#
# The A and AAAA records point to the IP of the VM.
#
#
# My external DNS is handled by cloudflare. I'm using dns-challenge for getting LetsEncrypt SSL certs.
#
#


ariadne@docker-host:~/docker/portainer-traefik$ cat docker-compose.yml 
version: '3.1'
services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce:latest
    command: -H unix:///var/run/docker.sock
    restart: always
      #    ports:
      #- 8000:8000
      #- 9443:9443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    networks:
      - user-bridge
    labels:
      - "traefik.enable=true"
      # using-the-fqdn
      - "traefik.http.routers.using-the-fqdn.rule=Host(`<docker-host>.<redacted>.org`)"
      - "traefik.http.routers.using-the-fqdn.entrypoints=websecure"
      - "traefik.http.routers.using-the-fqdn.service=using-the-fqdn"
      - "traefik.http.routers.using-the-fqdn.tls.certresolver=letsencrypt"
      - "traefik.http.services.using-the-fqdn.loadbalancer.server.port=9000"
  traefik:
    image: "traefik:v2.10"
    container_name: traefik
    restart: always
    command:
      # - "--log.level=DEBUG"
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      # create entry point "web"
      - "--entrypoints.web.address=:80"
      # create entry point "websecure"
      - "--entrypoints.websecure.address=:443"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
      # create cert resolver "letsencrypt"
      - "--certificatesresolvers.letsencrypt.acme.dnschallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.dnschallenge.provider=cloudflare"
      - "--certificatesresolvers.letsencrypt.acme.dnschallenge.resolvers=1.1.1.1:53,8.8.8.8:53"
      # - "--certificatesresolvers.letsencrypt.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory" # Staging CA Server
      - "--certificatesresolvers.letsencrypt.acme.caserver=https://acme-v02.api.letsencrypt.org/directory" # Production CA Server
      - "--certificatesresolvers.letsencrypt.acme.email=<redacted>"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    networks:
      - user-bridge
    environment:
      - "CF_DNS_API_TOKEN=<redacted>"
    volumes:
      - "./letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    labels:
      # create router "http-catchall"
      - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      # create middleware "middlewares"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.permanent=true"
volumes:
  portainer_data:

networks:
  user-bridge:
    external: true


#
# This is the config for the db, wiki, and duplicati backup services
#
ariadne@grove:~/docker/home-wiki$ cat docker-compose.yml 
version: "3.1"

services:
  db:
    image: postgres:15-alpine
    restart: no
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: <redacted>
      POSTGRES_USER: wikijs
    logging:
      driver: "none"
    volumes:
      - /mnt/wiki-drive:/var/lib/postgresql/data
    networks:
      - user-bridge

  wiki:
    image: ghcr.io/requarks/wiki:2
    restart: always
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
    ports:
      - "3000:3000"
    networks:
      - user-bridge
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.wiki.rule=Host(`wiki.<redacted>.org`)"
      - "traefik.http.routers.wiki.entrypoints=web,websecure"
      - "traefik.http.routers.wiki.tls.certresolver=letsencrypt"
      - "traefik.http.services.wiki.loadbalancer.server.port=3000"

  duplicati:
    image: duplicati/duplicati:latest
    restart: always
    ports:
      - "8200:8200"
    command: "/usr/bin/duplicati-server --webservice-port=8200 --webservice-interface=any --webservice-allowed-hostnames=*"
    volumes:
      - /mnt/wiki-drive:/wiki-drive:rw        # What we want to back up 
      - /opt/duplicati/data:/data:rw          # Config Storage on the host
    networks:
      - user-bridge
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.duplicati.rule=Host(`wiki-backup.<redacted>.org`)"
      - "traefik.http.routers.duplicati.entrypoints=web,websecure"
      - "traefik.http.routers.duplicati.tls.certresolver=letsencrypt"
      - "traefik.http.services.duplicati.loadbalancer.server.port=8200"

networks:
  user-bridge:
    external: true
```
