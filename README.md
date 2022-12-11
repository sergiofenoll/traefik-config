# Traefik config

Add the following to your app's docker-compose file to get the routing working:

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

The config expects a self signed certificate as well, because it uses HTTPS. It's probably overkill since this is meant for local development, but I couldn't get Firefox to access the services via the unencrypted link, so I had to go for HTTPS.

Cert commands, taken from here: https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/

``` sh
# Create a CA key
openssl genrsa -des3 -out CA.key 4096

# Create a CA root certificate
openssl req -x509 -new -nodes -key CA.key -sha256 -days 365 -out CA.pem

# Adding the root certificate so my OS trusts it (Fedora)
cp CA.pem /etc/pki/ca-trust/source/anchors/
update-ca-trust

# Generate key for domain
openssl genrsa -out dev.sff.im.key 4096

# Generate CSR (certificate signing request)
openssl req -new -key dev.sff.im.key -out dev.sff.im.csr

# Create a x509 v3 certificate extension config
cat > dev.sff.im.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = dev.sff.im
EOF

# Create a certificate for our domain name
openssl x509 -req -in dev.sff.im.csr -CA CA.pem -CAkey CA.key -CAcreateserial -out dev.sff.im.crt -days 365 -sha256 -extfile dev.sff.im.ext
```

Repeat the domain specific steps for the wildcard domain too. Put the files in `./certs` and make sure the filenames match the definitions in `./config/dynamic/certs.yml`.
