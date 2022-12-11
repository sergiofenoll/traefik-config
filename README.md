# Traefik config

Add the following to your docker-compose file to get the routing working:

``` yaml

networks:
  default:
  traefik:
    name: "traefik"
    external: true

# [...]

services:
  identifier:
    networks:
      - default
      - traefik
  
  triplestore:
    networks:
      - default
      - traefik
    labels: # this part is only necessary for images that don't expose a port or expose multiple ports
      - "traefik.http.services.triplestore-<<app dir goes here>>.loadblancer.server.port=8890"
```

When adding the port label, make sure to make the key unique. If you use e.g. `traefik.http.services.triplestore.loadblancer.server.port` for all your triplestore containers over all your stacks, it won't work because Traefik will see all your containers as one service.

Defining the port with the Traefik labels is only necessary if the image exposes no port or if the image exposes multiple ports. In that case Traefik will use the last exposed port, but for the virtuoso image that's port 1111, which is not the one we want.
