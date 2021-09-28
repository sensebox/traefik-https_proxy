# traefik-https_proxy
Simple HTTPS Proxy with Traefik &amp; LetsEncypt

## Usage
To publish a service and make it available through `traefik` you have to attach [labels](https://doc.traefik.io/traefik/providers/docker/#routing-configuration) to your container or service.

Docker-Compose Example:

```yml
version: "3.9"

networks:
  internal: # Use this network to expose services within our internal network and not expose it to the outside world
    internal: true
  https_proxy_traefik: # This network is created by our proxy
    external: true

services:
  pgadmin:
    image: dpage/pgadmin4:5.7
    restart: unless-stopped
    environment:
      ...
    networks:
      - https_proxy_traefik
      - internal
    volumes:
      ...
    labels:
      - "traefik.enable=true" # Enable discovery through traefik
      - "traefik.docker.network=https_proxy_traefik" # Our specified network
      - "traefik.http.routers.pgadmin.rule=Host(`pgadmin.testing.opensensemap.org`)" # What domain we want to use
      - "traefik.http.routers.pgadmin.entrypoints=web-secure" # What traefik entrypoint we want to use (https) specified in traefik.toml
      - "traefik.http.routers.pgadmin.tls.certresolver=letsencrypt" # Our certificate resolver specified in traefik.toml
      - "traefik.http.services.pgadmin.loadbalancer.server.port=80" # Exposed port from our dpage/pgadmin image
```