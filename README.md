# Summary

This repository contains the solution to Aleyant challenge with HAProxy running in docker.

## Issues Found

- Wrong network configuration in docker compose
    - Fix: replace `wrongNet` with `alyantNet` in docker compose file, using `bridge` as network driver.

- Unable to pull lorem-ipsum-images
    - Reason: probably the image doens't exist on Docker Hub or I don't have permissions to pull it.
    - Fix: As it's purpose is to serve a static page, the following steps were made:
        - Replace the image with another one that offer an static webserver.
        - The choosen one was `python-alpine` as by default python comes with HTTP server module that is useful to serve static files.
        - Made the changes in docker compose file to make this happen on backends (volume, command, ports, etc)
        - Bonus, used yaml anchor to avoid code repetition
- HAProxy image failing to build
    - Fixes:
        - Base image tag misconfigured (doesn't exist on registry), also it was using a busybox one that doesn't support apt-get. It was changed to use `debian:12-slim` instead.
        - Copy of haproxy.cfg was failing due to wrong path. Moved file to right location.
        - Crontab file mising new line before end of file. Added it
        - added build section on docker compose file
- HAProxy binary not being installed
    - Fixes: line in Dockerfile was commented, so I uncommented it and bumped HAProxy version to the one available im Debian 12.
- Scripts without execution permission
    - Fix: chmod entry added to enable execution on both
- Wrong port on haproxy backend configuration
    - Fix: edit config file to adjust backend ports
- certbot using staging environmment
    - Fix: change variable to make certbot use ACME production environment
- renewal script not configured in crontab file
    - Fix: replace the dummy action with the cert-renewal-script.sh, sending output to a log file

## Pre-requisites to run the solution

- Linux machine (VM or baremetal) with ports 80 and 443 published and reachable on public internet
    - the machine can have a public IP address and firewall rules accepting connections
    - in case of a machine with no public address, you can use a tunnel service like cloudflare or tailscale (I've used cloudflare to develop)
- Docker and docker composed installed in the machine
- A domain name that you can manage
    - make sure you have the FQDN to be used poiting to the machine IP address or tunnel provider

## How to run the solution

Clone this repository and change to the created directory.

Edit the `docker-compose.yml`, and change the *CERTS* (line 7) and *EMAIL* (line 8) environment variables for LB container.

> Note: In the commited file i've used my personal email and a personal domain.

Run the solution with `docker-compose up --build`.

## Improvements in the solution

- Use of DNS01 challenge instead of HTTP01 for certificate issue and renewal
    - This allows to run certbot in a private network environment
- Reduce the number of moving parts replacing HAProxy + certbot + cron image with an image of a webserver like [Caddy](https://caddyserver.com/) or [Traefik](https://traefik.io/traefik/), that can handle issue and renewal of Let's Encrypt certificate out of the box.
