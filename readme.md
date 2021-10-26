# Setup for traefik, docker-compose, lets-encrypt and .pem file generation
So this is a repo I made for myself to quickly start up a new server with traefik and lets-encrypt ssl, along with generation of .pem certificate files from lets-encrypt.
The set-up for certificate retrieval is outlined in traefik.yml and is made to work with dns-challenge from cloudflare to get a wildcard certificate. Also there are instructions in dynamic_conf.yml to redirect all domains (main and subdomains) to https from http and www.

-0. To test if dns is set-up correctly. rename check-dns.yml to docker-compose.yml and, use that to run "docker-compose up"

-1. make a new docker network with `docker network create yournetworkname` use this network to connect your containers

-2. change email in traefik/traefik.yml

-3. set-up a subdomain for traefik dashboard (something like traefik.your-site.com) and write it in dynamic_conf.yml

-4. in traefik/dynamic_conf.yml set up a basic auth login and password hash (make a hash with some site that makes basic auth hashes)

-5. make a dns token in cloudflare (https://dash.cloudflare.com/profile/api-tokens) like below
````  
Zone Zone Read
Zone DNS  Edit
````
and write the token key in .env file

-6. Run docker-compose up. Whoami is provided in docker-compose.up as a means to test everything worked. You can delete it if you have other containers set-up.

P.S.
The traefik-certs-dumper is not needed to make a traefik with ssl site - but is included here for generation of .pem files from acme.json to be used by other code in other containers. For example Go http server uses .pem files for TLS connection.
