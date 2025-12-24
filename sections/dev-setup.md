# Dev Setup

## Overview

Standard local development setup for all GRS Laravel applications.

All GRS development repositories **must** support being run and edited inside Docker.

---

## Docker Requirements

- Development must be possible entirely inside containers
- Source code should be bind-mounted so it can be edited from the host
- The default `docker-compose up` should start a working dev environment

---

## Baseline Docker Configuration

Use the following baseline configuration for Laravel apps. Projects may extend this as needed, but should not diverge without a clear reason.

### .dockerignore

```gitignore
vendor
node_modules
.git
.gitignore
npm-debug.log
yarn-error.log
storage/logs
storage/framework/cache
storage/framework/sessions
storage/framework/views
.idea
.vscode
```

### docker-compose.yml

```yaml
version: "3.9"

services:
  app:
    build: .
    container_name: <myappname>
    working_dir: /var/www/html
    volumes:
      - ./:/var/www/html
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      - redis
    command: php artisan serve --host=0.0.0.0 --port=8000

  redis:
    image: redis:7-alpine
    container_name: <myappname>_redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: ["redis-server", "--appendonly", "yes"]

volumes:
  redis-data:
```

### Dockerfile

```dockerfile
FROM php:8.4-cli

# Install system dependencies and PHP extensions required by Laravel
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    git \
    unzip \
    libzip-dev \
    libpq-dev \
    libsqlite3-dev \
    sqlite3 \
    libicu-dev \
    && docker-php-ext-install \
    pdo \
    pdo_mysql \
    pdo_sqlite \
    bcmath \
    intl \
    pcntl \
    zip \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && rm -rf /var/lib/apt/lists/*

# Install Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

# Default command: run Laravel development server
CMD ["php", "artisan", "serve", "--host=0.0.0.0", "--port=8000"]
```
