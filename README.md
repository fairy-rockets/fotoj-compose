# Pixelfed using docker-compose

[Pixelfed](https://pixelfed.org/) is a federated photo sharing platform.

We are runnning an instance, [fotoj.hexe.net](https://fotoj.hexe.net/) using this repository with docker.

## How to setup

Please preapare `.env` file in the repository root. Please see [the document](https://docs.pixelfed.org/) about it.

Then,

```sh
% make up
```

If succeed, `fotoj_web` exposes port 80 to accept HTTP request:

```
% docker-compose ps
    Name                  Command               State    Ports
----------------------------------------------------------------
fotoj_app      docker-php-entrypoint /var ...   Up      9000/tcp
fotoj_db       docker-entrypoint.sh --def ...   Up
fotoj_redis    docker-entrypoint.sh redis ...   Up
fotoj_web      /docker-entrypoint.sh ngin ...   Up      80/tcp
fotoj_worker   docker-php-entrypoint gosu ...   Up      9000/tcp
```

Then follow [the document](https://docs.pixelfed.org/) to have one-time setup tasks done.

Finally, you can setup reverse proxy to connect it!

```
# I'm using nginx-upstream-dynamic-servers module to resolve servername dynamically.
# https://github.com/GUI/nginx-upstream-dynamic-servers
upstream fotoj {
  server fotoj_web:80 resolve;
}
```

```
server {
  listen      443 ssl http2;
  listen [::]:443 ssl http2;
  server_name  fotoj.hexe.net;

  ssl_certificate /etc/letsencrypt/live/hexe.net/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/hexe.net/privkey.pem;

  client_max_body_size 512m;
  location / {
    expires off;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_redirect http:// https://;
    proxy_pass http://fotoj/;
  }
}
```

# LICENSE

MIT
