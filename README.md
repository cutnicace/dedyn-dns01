# Big Thanks to Pavel Král for providing the configuration for writing the dm-dns01 provider for Czech DNS provider Domain Master. This is an adaption of his program to work with the dedyn api.

# dedyn-dns01

This is [Traefik](https://traefik.io/) ACME exec provider for german dyndns provider [dedyn.io](https://www.desec.io/).

It can be used for [Let's Encrypt](https://letsencrypt.org/) DNS01 challenge 
automation for domains from desec.io . This provider is suitable for running under
dockerized Traefic (e.g. [traefik:latest](https://hub.docker.com/_/traefik/) based on scratch) that does not
contains shell or other unnecessary utilities.

## Building

If you are already running docker you can setup a functional go development environment (credit goes to this [tutorial](https://levelup.gitconnected.com/setup-simple-go-development-environment-with-docker-b8b9c0d4e0a8)) with:
`docker run --rm -it --name go-restful -v $PWD:/go/src/github.com/the-evengers/go-restful golang`

You may need to install dependencies first:

`go get github.com/docopt/docopt-go`

To obtain full static binary without single dependency run:

`CGO_ENABLED=0 GOOS=linux GOARCH=arm go build -a -ldflags="-s -w" -o dedyn_dns01`

Thanks to Go goodness, you may also choose different platform (e.g. mips, aarch64, etc.) based on your needs.
For example the above line is for an arm architecture since my traefik is running on an raspberry pi.

And optionally shrink the result:

`upx --ultra-brute dedyn_dns01`

## Example usage with docker-compose

Just provide following environment properties to access [dedyn_api](https://desec.io/api/v1/) 

* `EXEC_PATH` - Path to build dedyn_dns01, accessible in guest container. 
* `DEDYN_TOKEN` - your dedyn.io access token 
* `DEDYN_NAME`- your dedyn domain name


```
version: '3'

services:
  reverse-proxy:
    container_name: traefik
    image: traefik
    ports:
      - 80:80     # http
      - 443:443   # https      
    volumes:
      - traefik-tmp:/tmp
      - ./etc:/etc/traefik
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - EXEC_PATH=/etc/traefik/dedyn_dns01
      - DEDYN_TOKEN=d41d8cd98f00b204e9800998ecf8427e
      - DEDYN_NAME=example.dedyn.io
  whoami:
    image: containous/whoami 
    labels:
      - traefik.enable=true
      - "traefik.frontend.rule=Host:whoami.docker.localhost"

volumes:
   traefik-tmp:
      driver: local

```

In the example above the dedyn_dns01 executable must be put under ./etc/dedyn_dns01 relative to docker-compose.yaml
