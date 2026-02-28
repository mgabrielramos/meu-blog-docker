# 🛰️ WordPress + Docker + Nginx + Cloudflare Tunnel (Homelab on Windows 11)

This project is a complete stack to run a self‑hosted WordPress blog **at home** on Windows 11 using:

- Docker + Docker Compose
- Nginx as a reverse proxy
- MySQL as the database
- Cloudflare Tunnel to expose the site publicly, even with a dynamic IP and without router access

Ideal for learning DevOps, networking, containers, and self‑hosting.

---

## 🔧 Stack Architecture

### 1. Database (`db` – MySQL 8)

- Container based on the official `mysql:8.0` image.
- Stores all WordPress data: posts, users, settings.
- Data is persisted in the Docker volume `db_data` so nothing is lost when containers restart.
- Credentials and passwords are defined via environment variables in the `.env` file (not versioned).

### 2. Application (`wordpress` – WordPress + PHP‑FPM)

- Container based on the official `wordpress:php8.3-fpm` image (WordPress running via PHP‑FPM).
- Connects to the MySQL database using variables defined in `docker-compose.yml` and `.env`.
- WordPress code and files (plugins, themes, uploads) live in the `wp_data` volume, ensuring persistence across container recreations.

### 3. Web Server (`nginx` – Nginx as reverse proxy)

- Lightweight container based on `nginx:alpine`.
- Serves static files (images, CSS, JS) and forwards PHP requests to the `wordpress` container (PHP‑FPM).
- Configuration in `nginx.conf`, including:
  - GZIP response compression.
  - Basic security headers (X-Frame-Options, X-Content-Type-Options, etc.).
  - Full page caching with `proxy_cache` to reduce PHP/MySQL load.
  - Protection of sensitive files like `wp-config.php`.

### 4. Secure Tunnel (`tunnel` – Cloudflare Tunnel / `cloudflared`)

- Container based on the official `cloudflare/cloudflared` image.
- Creates a secure tunnel between your machine and Cloudflare's edge using a `TUNNEL_TOKEN`.
- Allows you to expose the site on a public domain (e.g. `https://your-domain.com`) **without opening router ports** and even behind CGNAT.
- Traffic enters through Cloudflare, passes through the tunnel to Nginx, and then reaches WordPress.

---

## 🗂 File Structure

Main files in this repository:

- `docker-compose.yml`  
  Defines all services (db, wordpress, nginx, tunnel), volumes, and the internal Docker network.

- `nginx.conf`  
  Nginx configuration with:
  - Reverse proxy for PHP‑FPM
  - Page cache
  - GZIP
  - Security headers
  - WordPress‑specific rules

- `.gitignore`  
  Ensures that sensitive files and data are not pushed to GitHub, such as:
  - `.env` (passwords, tokens)
  - Local volumes (`db_data/`, `wp_data/`)
  - Logs, temporary files, editor configs, etc.

> Note: the `.env` file is **not** versioned for security reasons. It must be created locally.

---

## 🔐 `.env` File (example)

Create a file named `.env` at the project root with variables like:

```env
MYSQL_ROOT_PASSWORD=change_this_root_password_2026!
MYSQL_PASSWORD=wp_user_password_2026!
WORDPRESS_DB_PASSWORD=wp_user_password_2026!
TUNNEL_TOKEN=YOUR_REAL_CLOUDFLARE_TUNNEL_TOKEN
```

> Never commit this file. It contains database passwords and the Cloudflare Tunnel token.

---

## 🚀 How to Run the Stack

1. Clone the repository:

```bash
git clone https://github.com/YOUR_USER/YOUR_REPO.git
cd YOUR_REPO
```

2. Create the `.env` file with your own passwords/tokens.

3. Start the services:

```bash
docker compose up -d
```

4. Access WordPress through the domain configured in Cloudflare Tunnel (e.g. `https://your-domain.com`) and complete the standard installer.

---

## 🌐 Cloudflare / HTTPS Adjustments for WordPress

In `wp-config.php` you can add adjustments to:

- Use the real visitor IP coming from Cloudflare headers.
- Force HTTPS behind the tunnel.
- Set `WP_HOME` and `WP_SITEURL` to your public domain.

These tweaks help avoid mixed‑content issues and ensure the admin panel always uses HTTPS.

---

## 🎯 Project Goals

This repository was created to:

- Provide a practical lab for Docker, Nginx, WordPress, and Cloudflare Tunnel.
- Serve as a homelab base for self‑hosting a blog.
- Demonstrate a modern containerized web application architecture, focusing on:
  - Separation of concerns (app, database, proxy, tunnel)
  - Basic security
  - Reproducibility (bringing everything up with a single `docker compose up -d` command).

Feel free to fork, adapt, and evolve this stack (more aggressive caching, Redis, monitoring, CI/CD, etc.).
