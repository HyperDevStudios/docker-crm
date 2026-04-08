# Monica + host nginx reverse proxy (port 1234)

Use this setup when nginx already runs on your server and you only need Monica on an internal host port.

## 1) Compose file

Use `docker-compose.yml` from this folder and place an `.env` file next to it. Example minimum:

```env
APP_KEY=base64:replace-with-your-generated-key
APP_ENV=production
APP_URL=https://fluff-crm.jroering.com
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=monica
DB_USERNAME=monica
DB_PASSWORD=secret
```

> Tip: generate the `APP_KEY` with `echo -n 'base64:'; openssl rand -base64 32`.

## 2) Start containers

```bash
docker compose up -d
```

## 3) Publish Monica on host port 1234

The compose file in this folder already maps `1234:80`, so Monica is reachable locally on `http://127.0.0.1:1234`.

If you change compose settings later, apply them with:

```bash
docker compose up -d
```

## 4) nginx vhost on the host machine

Create a server block for `fluff-crm.jroering.com`:

```nginx
server {
  listen 80;
  server_name fluff-crm.jroering.com;

  location / {
    proxy_pass http://127.0.0.1:1234;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

Then reload nginx:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

## 5) TLS/HTTPS

For production, add TLS (for example with certbot) and keep `APP_URL=https://fluff-crm.jroering.com`.
