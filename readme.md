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

-6. Run docker-compose up. Whoami is provided in docker-compose.yml as a means to test everything worked. You can delete it if you have other containers set-up. I prefer to leave this traefik, along with its docker-compose.yml alone, and for my projects I use other docker-compose.yml and in those projects services I use "labels" and "networks" to connect to traefik. Here is an example docker-compose.yml for a website that connects to traefik
````
networks:
  yournetworkname:
    external: true

# just one service here, running a nextjs app for example
services:
  nextjs:
  # here would be a bunch of other labels like logging, build, ports, volumes, tty
  networks:
    - yournetworkname
  # and all this good stuff below is what connects our traefik container, in a seperate folder somewhere on the host, to this particular project, and applys middlewares   
  labels:
    - "traefik.enable=true"
    - "traefik.http.services.nextjs.loadbalancer.server.port=3000"
    - "traefik.http.routers.yourroutername1.rule=Host(`yourwebsite.com`)"
    - "traefik.http.routers.yourroutername1.entrypoints=websecure"
    - "traefik.http.routers.yourroutername1.tls.certresolver=myresolver"
    - "traefik.http.routers.yourroutername1.tls.domains[0].main=yourwebsite.com"
    - "traefik.http.routers.yourroutername1.tls.domains[0].sans=*.yourwebsite.com"
    - "traefik.http.middlewares.yourmidwarename1.ratelimit.average=30"
    - "traefik.http.middlewares.yourmidwarename1.ratelimit.burst=100"
    - "traefik.http.middlewares.yourmidwarename1.ratelimit.period=1"
    - "traefik.http.routers.yourroutername1.middlewares=yourmidwarename1@docker"
````

P.S.
The traefik-certs-dumper is not needed to make a traefik with ssl site - but is included here for generation of .pem files from acme.json to be used by other code in other containers. For example Go http server uses .pem files for TLS connection.
