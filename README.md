![Pterodactyl Banner](https://cdn.pterodactyl.io/logos/new/pterodactyl_logo_transparent.png)

# How to Create Pterodactyl Panel on CodeSandbox

This guide shows you how to install and run the **Pterodactyl Panel** on CodeSandbox (or any environment supporting Docker).

---

## ⚙️ Step 0: Install Dependencies
Before starting, make sure Docker Compose is installed:

```bash
apt update
apt install docker-compose -y
```

---

## 📦 Step 1: Create Project Directories
```bash
mkdir pterodactyl
cd pterodactyl

mkdir panel
cd panel
```

---

## 📝 Step 2: Create docker-compose.yml
Open the `docker-compose.yml` file:
```bash
nano docker-compose.yml
```
Paste the following configuration (replace `CHANGE_ME` values with secure passwords):

```yaml
version: '3.8'

x-common:
  database:
    &db-environment
    MYSQL_PASSWORD: &db-password "CHANGE_ME"
    MYSQL_ROOT_PASSWORD: "CHANGE_ME_TOO"
  panel:
    &panel-environment
    APP_URL: "https://pterodactyl.example.com"
    APP_TIMEZONE: "UTC"
    APP_SERVICE_AUTHOR: "noreply@example.com"
    TRUSTED_PROXIES: "*"
  mail:
    &mail-environment
    MAIL_FROM: "noreply@example.com"
    MAIL_DRIVER: "smtp"
    MAIL_HOST: "mail"
    MAIL_PORT: "1025"
    MAIL_USERNAME: ""
    MAIL_PASSWORD: ""
    MAIL_ENCRYPTION: "true"

services:
  database:
    image: mariadb:10.5
    restart: always
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - "./data/database:/var/lib/mysql"
    environment:
      <<: *db-environment
      MYSQL_DATABASE: "panel"
      MYSQL_USER: "pterodactyl"

  cache:
    image: redis:alpine
    restart: always

  panel:
    image: ghcr.io/pterodactyl/panel:latest
    restart: always
    ports:
      - "8030:80"
      - "4433:443"
    links:
      - database
      - cache
    volumes:
      - "./data/var:/app/var"
      - "./data/nginx:/etc/nginx/http.d"
      - "./data/certs:/etc/letsencrypt"
      - "./data/logs:/app/storage/logs"
    environment:
      <<: [*panel-environment, *mail-environment]
      DB_PASSWORD: *db-password
      APP_ENV: "production"
      APP_ENVIRONMENT_ONLY: "false"
      CACHE_DRIVER: "redis"
      SESSION_DRIVER: "redis"
      QUEUE_DRIVER: "redis"
      REDIS_HOST: "cache"
      DB_HOST: "database"
      DB_PORT: "3306"

networks:
  default:
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

---

## 📂 Step 3: Create Data Directories
```bash
mkdir -p ./data/{database,var,nginx,certs,logs}
```

---

## ▶ Step 4: Start the Containers
```bash
docker-compose up -d
```

---

## 👤 Step 5: Create Your First User
```bash
docker-compose run --rm panel php artisan p:user:make
```

Follow the prompts to set up your admin account.

---

## ✅ Done!
Your **Pterodactyl Panel** is now running on CodeSandbox.  
Access it via the mapped port (8030) or configure a custom domain in your APP_URL.

---

**Credit:** HopingBoyz  
**Subscribe on YouTube:** [@HopingBoyzL](https://youtube.com/@hopingboyz)

![Pterodactyl Footer](https://cdn.pterodactyl.io/site-assets/carousel/screenshot-1.png)
