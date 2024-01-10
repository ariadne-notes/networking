This is a reverse-proxy keycloak setup with persistent volumes. I set this up on accident trying to figure out directory services for my home lab.

# Docker Compose
```
version: '3'

services:
  keycloak-db:
    image: postgres
    volumes:
      - /opt/keycloak-postgresql-db:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: "<password>"
    networks:
      - user-bridge

  keycloak:
    image: jboss/keycloak
    environment:
      DB_VENDOR: POSTGRES
      DB_ADDR: keycloak-db
      DB_DATABASE: keycloak
      DB_USER: keycloak
      DB_PASSWORD: "<password>"
      KEYCLOAK_USER: admin
      KEYCLOAK_PASSWORD: "<another-password>"
      PROXY_ADDRESS_FORWARDING: true
      KEYCLOAK_FRONTEND_URL: "https://<host>.<domain>.org/auth"
    ports:
      - "6060:8080"
    depends_on:
      - keycloak-db
    networks:
      - user-bridge
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.keycloak.rule=Host(`<host>.<domain>.org`)"
      - "traefik.http.routers.keycloak.entrypoints=web,websecure"
      - "traefik.http.routers.keycloak.tls.certresolver=letsencrypt"
      - "traefik.http.services.keycloak.loadbalancer.server.port=8080"

volumes:
  keycloak-db:

networks:
  user-bridge:
    external: true
```
