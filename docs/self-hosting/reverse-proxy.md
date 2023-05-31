# Reverse proxy setup

Securing your bot with [SSL/TLS encryption](https://www.cloudflare.com/en-gb/learning/ssl/what-is-ssl/) is strongly recommended.

To do this, you need:

1. An FQDN (e.g. `example.com`)
2. To set up a web server as a reverse proxy

!!! tip
	If your bot is running on a [supported port](https://developers.cloudflare.com/fundamentals/get-started/reference/network-ports/),
	you can use Cloudflare's Proxy and free SSL/TLS instead.

If you already have a domain and know how to create an HTTPS proxy, you can safely skip this page.
If not, there are several options available:

|                   |    [Traefik](#traefik)     |   [Nginx](#nginx)    | [Caddy](#caddy-2-recommended) | [PebbleHost](#pebblehost) |
| :---------------- | :------------------------: | :------------------: | :---------------------------: | :-----------------------: |
| Difficulty        | Most difficult { .orange } | Moderate { .orange } |        Easy { .green }        |      Easy { .green }      |
| Bot installations |        Docker only         |         Any          |              Any              |      PebbleHost only      |

!!! warning ""
    Make sure you set the bot's `HTTP_TRUST_PROXY` environment variable to `#!yaml true`.

## Caddy 2 (recommended)


!!! info ""
    If you already have Caddy running, update your existing configuration and use `caddy reload` instead.

First, [install Caddy](https://caddyserver.com/docs/install), then open the `Caddyfile` and edit the domain.

```nginx title="Caddyfile"
{==tickets.example.com==} {
    reverse_proxy 127.0.0.1:8169
}
```

Now start Caddy:

```bash
sudo caddy start
```

## Nginx

### Community guides

<div class="grid cards" markdown>

-   :one: __Nginx installation__

    ---

    How to install Nginx on various Linux distributions.

    [:octicons-arrow-right-24: DigitalOcean Community](https://www.digitalocean.com/community/tutorial_collections/how-to-install-nginx)

-   :two: __Securing Nginx__

    ---

    How to secure Nginx with Let's Encrypt on various Linux distributions.

    [:octicons-arrow-right-24: DigitalOcean Community](https://www.digitalocean.com/community/tutorial_collections/how-to-secure-nginx-with-let-s-encrypt)

</div>

### Configuration

<div class="annotate" markdown>

This example will proxy traffic from `http://tickets.example.com` to your bot.
To secure the connection, refer to the guides linked above.

```nginx title="/etc/nginx/sites-available/tickets.example.com"
server {
    listen 80;
    listen [::]:80;(1)

    server_name {==tickets.example.com==};(2)

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_http_version 1.1;
        proxy_pass http://127.0.0.1:8169;(3)
    }
}
```

</div>

1. Remove this line if you don't have IPv6 networking.
2. Replace this with the FQDN that you set in your bot's `HTTP_EXTERNAL` environment variable.
3. Change the port to match your bot's `HTTP_PORT` environment variable.
   Also, change the IP address if the bot is running on a different server.



## Traefik

### Documentation

<div class="grid cards" markdown>

-   :simple-traefikproxy: __Traefik quick start__

    ---

    A quick start guide to adding Traefik to your existing `docker-compose.yml` file.

    [:octicons-arrow-right-24: Traefik Documentation](https://doc.traefik.io/traefik/getting-started/quick-start/)

-   :simple-traefikproxy: __Traefik configuration__

    ---

    How to use Traefik with Docker Compose.

    [:octicons-arrow-right-24: Traefik Documentation](https://doc.traefik.io/traefik/user-guides/docker-compose/basic-example/)

</div>

### Configuration

This example shows the labels you may need to add to the `bot` service in your `docker-compose.yml` file.
Refer to the documentation linked above for more information.

```yaml linenums="0" title="docker-compose.yml"
labels:
  - "traefik.enable=true"
  - "traefik.docker.network=traefik_network"
  - "traefik.http.routers.tickets.entrypoints=websecure"
  - "traefik.http.routers.tickets.rule=Host(`{==tickets.example.com==}`)"
  - "traefik.http.services.tickets.loadbalancer.server.port=8169"
```
## PebbleHost

<div class="grid cards" markdown>

-   :fontawesome-solid-life-ring:{ .lg .middle } __PebbleHost Reverse Proxy__

    ---

    How to set up a reverse proxy on PebbleHost.

    [:octicons-arrow-right-24: PebbleHost Knowledgebase](ttps://help.pebblehost.com/en/minecraft/how-to-setup-a-reverse-proxy)


</div>