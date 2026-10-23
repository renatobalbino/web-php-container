# Web PHP Container (PHP-FPM + Nginx on Alpine Linux)

Production-ready, optimized Docker images for PHP applications (Laravel / Lumen / generic PHP), based on **Alpine Linux (`3.24.1`)** with **PHP compiled from official source code (`php.net`)**, **Nginx Mainline**, **Composer 2**, on-demand extension installation helpers (`install-php-extensions`), and process management via **s6-overlay**.

---

## 📦 Available Variants (Active)

| Variant | Directory | Image Tag | Description |
| :--- | :--- | :--- | :--- |
| **PHP 8.5** | `php8.5/` | `renatobalbino/php:8.5` | PHP 8.5.12 + Nginx + Redis + Essential extensions + `install-php-extensions` |
| **PHP 8.4** | `php8.4/` | `renatobalbino/php:8.4` | PHP 8.4.27 + Nginx + Redis + Essential extensions + `install-php-extensions` |
| **PHP 8.5 Octane** | `php8.5-octane/` | `renatobalbino/php:8.5-octane` | PHP 8.5 + Swoole for Laravel Octane |
| **PHP 8.5 Octane + OCI8** | `php8.5-octane-oci8/` | `renatobalbino/php:8.5-octane-oci8` | PHP 8.5 + Swoole + Oracle Instant Client (`oci8`) |

---

## 🧩 Default Extensions and Libraries (8.4 and 8.5)

- **Compiled and enabled PHP extensions:**
  `bcmath`, `ctype`, `curl`, `dom`, `exif`, `fileinfo`, `filter`, `ftp`, `gd` (with JPEG, WebP, and FreeType support), `hash`, `iconv`, `intl`, `json`, `libxml`, `mbstring`, `mysqli`, `mysqlnd`, `openssl`, `pcntl`, `pcre`, `PDO`, `pdo_mysql`, `pdo_sqlite`, `Phar`, `posix`, `redis` (PECL), `session`, `SimpleXML`, `sockets`, `sodium`, `SPL`, `sqlite3`, `tokenizer`, `xml`, `xmlreader`, `xmlwriter`, `Zend OPcache`, `zip`, `zlib`.
- **Additional system packages (Alpine):**
  `nginx` (official mainline), `curl`, `ca-certificates`, `xvfb`, `ttf-freefont`, `fontconfig`, `dbus`, `libxrender`, `icu-libs`, `libzip`.

---

## 🛠️ Extension Installation & Management Helpers

Images `8.4` and `8.5` provide the following utilities in `/usr/local/bin/`:
- **`install-php-extensions`** (and its alias **`docker-php-ext-install`**): Installs, compiles, and enables one or more extensions (via PECL, PHP core sources, or Oracle Instant Client) and cleans up all temporary build dependencies once finished.
- **`docker-php-ext-enable`**: Enables an already-compiled `.so` module by creating the corresponding `.ini` file in `/usr/local/etc/php/conf.d/`.

### Examples of extensions supported by `install-php-extensions`:
- **Oracle (`oci8` and `pdo_oci`):** Automatically downloads and configures Oracle Instant Client `21.17` in `/opt/oracle/instantclient` and compiles the extension:
  ```dockerfile
  RUN install-php-extensions oci8
  # or with PDO_OCI:
  RUN install-php-extensions oci8 pdo_oci
  ```
- **Other extensions (Core or PECL):**
  ```dockerfile
  RUN install-php-extensions soap xsl pgsql pdo_pgsql ldap gmp gettext bz2 calendar
  RUN install-php-extensions xdebug imagick swoole amqp apcu mongodb
  ```

---

## ⚙️ Environment Variables Configuration (`.env`)

The project uses two levels of `.env` files for easy maintenance and version creation:

1. **Root `.env`** — defines the Docker Hub repository and image name:
   ```env
   REPOSITORY=renatobalbino
   IMAGE=php
   ```
2. **Version-level `.env`** (e.g., `php8.5/.env`, `php8.4/.env`) — defines the specific image tag for that variant:
   ```env
   TAG=8.5
   ```

---

## 🧪 Running Automated Tests (Without pushing to Docker Hub)

The repository provides an automated test suite at `tests/test-images.sh` which:
1. Builds the image locally (or tests a pre-built image).
2. Prints a complete inventory of installed versions (Alpine, Kernel, PHP CLI, PHP-FPM, Nginx, Composer, OpenSSL, cURL, `apk` packages, and PHP extensions).
3. Validates the availability of helper scripts (`install-php-extensions`, `docker-php-ext-enable`), all required extensions, Alpine packages, and `php.ini` directives (`expose_php = Off`).
4. Starts a temporary container to verify Nginx + PHP-FPM HTTP responses (`GET /`, security headers, hidden `X-Powered-By`, `/ping`, and `/status`).

```bash
# Test all active primary variants (php8.4 and php8.5)
./tests/test-images.sh all

# Test a specific version family
./tests/test-images.sh 8.5
./tests/test-images.sh 8.4
```

---

## 🚀 Building and Publishing to Docker Hub

The root `./build.sh` script builds the targeted version, **automatically runs the test suite**, and pushes (`docker push`) the image to Docker Hub only if 100% of the tests pass. If any test fails, the push is aborted.

```bash
# Build + Test + Push PHP 8.5 (default)
./build.sh
# or
./build.sh 8.5

# Build + Test + Push PHP 8.4
./build.sh 8.4
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

### 2. Microservice requiring Oracle (`oci8`) or additional custom extensions:
```dockerfile
FROM renatobalbino/php:8.5

# Installs Oracle Instant Client + oci8 extension (and/or soap, xdebug, pgsql, etc.)
RUN install-php-extensions oci8

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
