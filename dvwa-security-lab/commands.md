# DVWA Setup Commands

## 1. Verify Docker

```bash
docker --version
```

## 2. Pull DVWA Image

```bash
docker pull vulnerables/web-dvwa
```

## 3. Run DVWA Container

```bash
docker run -d --name dvwa -p 8080:80 vulnerables/web-dvwa
```

## 4. Confirm Container Is Running

```bash
docker ps
```

## 5. Open DVWA

- URL: `http://localhost:8080`
- Click **Create / Reset Database**
- Login with:
  - Username: `admin`
  - Password: `password`

## 6. Bonus Setup (Nginx + HTTPS)

Create self-signed cert (OpenSSL via Docker):

```bash
docker run --rm -v "${PWD}/bonus/certs:/out" alpine:3.20 sh -c "apk add --no-cache openssl >/dev/null && openssl req -x509 -nodes -newkey rsa:2048 -keyout /out/dvwa.key -out /out/dvwa.crt -days 365 -subj '/CN=localhost'"
```

Start reverse-proxy stack:

```bash
docker compose -f bonus/docker-compose.yml up -d
docker compose -f bonus/docker-compose.yml ps
```

Verify:

```bash
curl -I http://localhost:8081/login.php
curl -k -I https://localhost:8443/login.php
```

Stop bonus stack:

```bash
docker compose -f bonus/docker-compose.yml down
```
