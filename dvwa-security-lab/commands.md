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
