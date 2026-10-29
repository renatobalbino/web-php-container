# Web PHP Container (PHP-FPM + Nginx on Alpine Linux)

[![Docker Image CI](https://github.com/renatobalbino/web-php-container/actions/workflows/docker-image.yml/badge.svg)](https://github.com/renatobalbino/web-php-container/actions/workflows/docker-image.yml)
![Build Status](https://img.shields.io/badge/docker%20build-passing-brightgreen?logo=docker)
![Test Suite](https://img.shields.io/badge/tests-passing-brightgreen?logo=checkmarx)
![Alpine Linux](https://img.shields.io/badge/Alpine_Linux-3.24.1-0D597F?logo=alpinelinux)
![Hardened](https://img.shields.io/badge/security-apk%20purged%20at%20runtime-blue?logo=linux)

Production-ready, hardened, and optimized Docker images for PHP applications (Laravel / Lumen / Octane / generic PHP), based on **Alpine Linux** with **PHP compiled from official source code (`php.net`)** on `8.4`/`8.5`, **Nginx Mainline**, **Composer 2**, on-demand extension & Octane installation helpers (`install-php-extensions`, `octane-install`), and process management via **s6-overlay**.

---

## 📦 Available Variants & Build/Test Status

> The **Build Status** and **Test Status** badges below are automatically updated whenever `./tests/test-images.sh`, `./build.sh`, or `./build-all.sh` is executed.

### Active Primary Variants (Custom-Compiled PHP + Helpers)

| Variant | Directory | Image Tag | Build Status | Test Status | Description |
| :--- | :--- | :--- | :---: | :---: | :--- |
| **PHP 8.5** | `php8.5/` | `renatobalbino/php:8.5` | ![Build](https://img.shields.io/badge/build-passing-brightgreen) | ![Tests](https://img.shields.io/badge/tests-55%20passed-brightgreen) | PHP 8.5.12 + Nginx + Redis + `install-php-extensions` + `octane-install` |
| **PHP 8.4** | `php8.4/` | `renatobalbino/php:8.4` | ![Build](https://img.shields.io/badge/build-passing-brightgreen) | ![Tests](https://img.shields.io/badge/tests-55%20passed-brightgreen) | PHP 8.4.27 + Nginx + Redis + `install-php-extensions` + `octane-install` |

### Legacy Variants (With `install-php-extensions` & Runtime Hardening)

| Variant | Directory | Image Tag | Build Status | Test Status | Description |
| :--- | :--- | :--- | :---: | :---: | :--- |
| **PHP 8.2** | `php8.2/` | `renatobalbino/php:8.2` | ![Build](https://img.shields.io/badge/build-passing-brightgreen) | ![Tests](https://img.shields.io/badge/tests-31%20passed-brightgreen) | PHP 8.2 + Nginx + Redis + Composer 2 + `install-php-extensions` + Runtime `apk` removal |
| **PHP 8.1** | `php8.1/` | `renatobalbino/php:8.1` | ![Build](https://img.shields.io/badge/build-passing-brightgreen) | ![Tests](https://img.shields.io/badge/tests-28%20passed-brightgreen) | PHP 8.1 + Nginx + Composer 2 + `install-php-extensions` + Runtime `apk` removal |
| **PHP 8.0** | `php8.0/` | `renatobalbino/php:8.0` | ![Build](https://img.shields.io/badge/build-passing-brightgreen) | ![Tests](https://img.shields.io/badge/tests-28%20passed-brightgreen) | PHP 8.0 + Nginx + Composer 2 + `install-php-extensions` + Runtime `apk` removal |
| **PHP 7.4** | `php7.4/` | `renatobalbino/php:7.4` | ![Build](https://img.shields.io/badge/build-passing-brightgreen) | ![Tests](https://img.shields.io/badge/tests-28%20passed-brightgreen) | PHP 7.4 + Nginx + Composer + `install-php-extensions` + Runtime `apk` removal |
| **PHP 7.3** | `php7.3/` | `renatobalbino/php:7.3` | ![Build](https://img.shields.io/badge/build-passing-brightgreen) | ![Tests](https://img.shields.io/badge/tests-28%20passed-brightgreen) | PHP 7.3 + Nginx + Composer + `install-php-extensions` + Runtime `apk` removal |

---

## 🧩 Default Extensions and Libraries (`8.4` and `8.5`)

- **Compiled and enabled PHP extensions:**
  `bcmath`, `ctype`, `curl`, `dom`, `exif`, `fileinfo`, `filter`, `ftp`, `gd` (with JPEG, WebP, and FreeType support), `hash`, `iconv`, `intl`, `json`, `libxml`, `mbstring`, `mysqli`, `mysqlnd`, `openssl`, `pcntl`, `pcre`, `PDO`, `pdo_mysql`, `pdo_sqlite`, `Phar`, `posix`, `redis` (PECL), `session`, `SimpleXML`, `sockets`, `sodium`, `SPL`, `sqlite3`, `tokenizer`, `xml`, `xmlreader`, `xmlwriter`, `Zend OPcache`, `zip`, `zlib`.
- **Additional system packages (Alpine):**
  `nginx` (official mainline), `curl`, `ca-certificates`, `xvfb`, `ttf-freefont`, `fontconfig`, `dbus`, `libxrender`, `icu-libs`, `libzip`.

---

## 🛠️ Extension, Octane & Security Hardening Helpers

All images (`7.3` through `8.5`) provide the following utilities in `/usr/local/bin/`:
- **`install-php-extensions`** (and its alias **`docker-php-ext-install`**): Available in **all versions (`7.3` through `8.5`)**. Installs, compiles, and enables one or more extensions (via PECL, official Alpine packages, PHP core sources, or Oracle Instant Client) and cleans up all temporary build dependencies and the Alpine `apk` package manager once finished.
- **`docker-php-ext-enable`**: Available in **all versions (`7.3` through `8.5`)**. Enables an already-compiled `.so` module by creating the corresponding `.ini` file in PHP's `conf.d` directory.
- **`octane-install`** (and its alias **`install-octane`**): Available in **`8.4` and `8.5`**. Installs and enables `swoole`, `pcntl`, and `posix`, configures the `s6-overlay` Octane service (`/etc/services.d/octane/run`), updates `/etc/nginx/nginx.conf` to reverse-proxy requests to Octane on port `8000`, and purges `apk` after compilation.
- **Runtime Hardening (`s6-overlay` `cont-init.d`):** All images (`7.3`, `7.4`, `8.0`, `8.1`, `8.2`, `8.4`, `8.5`) automatically purge `apk-tools` and remove `/sbin/apk`, `/usr/sbin/apk`, `/etc/apk`, `/lib/apk`, `/usr/share/apk`, and `/var/cache/apk` during container startup (`/etc/cont-init.d/00-remove-apk.sh`). This allows developers to run `install-php-extensions`, `octane-install`, or `apk add` during `docker build`, while ensuring running pods do not have the `apk` binary available.

### Examples of extensions supported by `install-php-extensions`:
- **Oracle (`oci8` and `pdo_oci`):** Automatically downloads and configures Oracle Instant Client in `/opt/oracle/instantclient` and compiles the version-matched `oci8` extension (`oci8-2.2.0` on PHP 7.x, `oci8-3.0.1` on PHP 8.0, `oci8-3.2.1` on PHP 8.1/8.2, and latest `oci8` on PHP 8.4/8.5):
  ```dockerfile
  RUN install-php-extensions oci8
  # or with PDO_OCI (PHP 8.4/8.5):
  RUN install-php-extensions oci8 pdo_oci
  ```
- **Other extensions (Core, Alpine, or PECL):**
  ```dockerfile
  RUN install-php-extensions soap xsl pgsql pdo_pgsql ldap gmp gettext bz2 calendar
  RUN install-php-extensions xdebug imagick swoole amqp apcu mongodb
  ```

---

## ⚙️ Environment Variables (`.env`) & GitHub Actions CI/CD

All versions (`7.3` through `8.5`) use two levels of `.env` configuration for standardized maintenance and tagging:

1. **Root `.env` (or GitHub Secrets in CI)** — defines the Docker Hub repository and image name:
   - **Local development:** Copy `.env.example` to `.env` in the repository root (ignored by git via `.gitignore`):
     ```bash
     cp .env.example .env
     ```
     ```env
     REPOSITORY=renatobalbino
     IMAGE=php
     ```
   - **GitHub Actions CI (`.github/workflows/docker-image.yml`):**
     The workflow uses a parallel build matrix across all 7 PHP versions (`7.3`, `7.4`, `8.0`, `8.1`, `8.2`, `8.4`, `8.5`), sets up Docker Buildx, generates `.env` from **GitHub Secrets**, runs the automated test suite, and publishes passing images to Docker Hub:
     - `secrets.REPOSITORY` (fallback: `secrets.DOCKERHUB_USERNAME`)
     - `secrets.IMAGE` (fallback: `php`)
     - `secrets.DOCKERHUB_USERNAME` & `secrets.DOCKERHUB_TOKEN` (for Docker Hub authentication)
     - Supports automatic triggers on `push` / `pull_request` to `master`, as well as manual execution (`workflow_dispatch`) where you can select `all` or a single PHP version from a dropdown menu.
2. **Version-level `.env`** (e.g., `php8.5/.env`, `php8.4/.env`, `php7.4/.env`) — defines the specific image tag for that variant:
   ```env
   TAG=8.5
   ```

---

## 🧪 Running Automated Tests (Without pushing to Docker Hub)

The repository provides an automated test suite at `tests/test-images.sh` which:
1. Builds the image locally (or tests a pre-built image).
2. Prints a complete inventory of installed versions (Alpine, Kernel, PHP CLI, PHP-FPM, Nginx, Composer, OpenSSL, cURL, `apk` packages, and PHP extensions).
3. Validates the availability of helper scripts (`install-php-extensions`, `docker-php-ext-enable`, `octane-install`), all required extensions, Alpine packages, and `php.ini` directives (`expose_php = Off`).
4. Starts a temporary container to verify Nginx + PHP-FPM HTTP responses (`GET /`, security headers, hidden `X-Powered-By`, `/ping`, `/status`), and confirms that `apk` has been completely removed inside the running pod.
5. Automatically updates the **Build Status** and **Test Status** badges in `readme.md`.

```bash
# Test all versions (7.3 through 8.5) and update readme.md status badges
./tests/test-images.sh all
# or
./build-all.sh --no-push

# Test only modern custom-compiled versions (php8.4, php8.5)
./tests/test-images.sh modern

# Test a specific version (8.5, 8.4, 8.2, 8.1, 8.0, 7.4, 7.3)
./tests/test-images.sh 8.5
./tests/test-images.sh 8.4
./tests/test-images.sh 8.1
./tests/test-images.sh 7.4
```

---

## 🚀 Building and Publishing to Docker Hub

The root `./build.sh` and `./build-all.sh` scripts build the targeted version(s) using the `.env` files, **automatically run the test suite** (updating the status flags in `readme.md`), and push (`docker push`) the image(s) to Docker Hub only if 100% of the tests pass. If any test fails, the push is aborted.

```bash
# Build + Test + Update readme.md + Push ALL versions (7.3 through 8.5)
./build-all.sh
# or
./build.sh all

# Build + Test + Update readme.md locally for ALL versions WITHOUT pushing to Docker Hub
./build-all.sh --no-push

# Build + Test + Push PHP 8.5 (default)
./build.sh
# or
./build.sh 8.5

# Build + Test + Push specific versions
./build.sh 8.4
./build.sh 8.2
./build.sh 8.1
./build.sh 8.0
./build.sh 7.4
./build.sh 7.3
```

---

## 💻 Usage in Microservices (Laravel / Lumen)

### 1. Standard microservice:
```dockerfile
FROM renatobalbino/php:8.5

WORKDIR /var/www/html
COPY . .

RUN composer install --no-dev --ignore-platform-reqs --optimize-autoloader
```

### 2. Microservice requiring Oracle (`oci8`) or additional custom extensions (works on 7.3 through 8.5):
```dockerfile
FROM renatobalbino/php:8.1

# Installs Oracle Instant Client + oci8 extension (and/or soap, xdebug, pgsql, etc.)
RUN install-php-extensions oci8

WORKDIR /var/www/html
COPY . .

RUN composer install --no-dev --ignore-platform-reqs --optimize-autoloader
```

### 3. Microservice with Laravel Octane (Swoole):
```dockerfile
FROM renatobalbino/php:8.5

# Installs Swoole + pcntl + posix, configures Nginx reverse proxy (port 8000) and s6-overlay Octane service
RUN octane-install

WORKDIR /var/www/html
COPY . .

RUN composer install --no-dev --ignore-platform-reqs --optimize-autoloader
```

### Running locally with volume mounting:
```bash
cd your_project/
docker run -d -p 8000:80 -v $(pwd):/var/www/html renatobalbino/php:8.5
```

---

## ➕ Creating a New Version in the Future

1. Duplicate the latest version directory (e.g., `cp -r php8.5 php8.6`).
2. Edit `php8.6/.env` to update the `TAG` variable (e.g., `TAG=8.6`).
3. Update the PHP release version in `php8.6/config/boostrap.sh` (`PHP_RELEASE`).
4. Run `./tests/test-images.sh php8.6` to validate locally or `./build.sh 8.6` to test and push.
