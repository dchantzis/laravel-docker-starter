# Laravel Docker Starter

Reusable local Laravel development infrastructure. **No Laravel source code is included or installed automatically.**

## Stack

- Ubuntu 24.04 + PHP 8.5 (packages from Ondřej Surý's Ubuntu PPA, newest available patch at build time)
- Nginx + PHP-FPM + Supervisor
- MySQL 8.4 LTS (persistent named volume)
- Redis 7 (persistent named volume)
- SQLite PHP extension + SQLite CLI (no SQLite container)
- Composer 2, Node.js 24, npm, Xdebug (disabled by default)

## Getting started

```bash
cd ~/Code/laravel-docker-starter
cp .env.example .env
./dev build
./dev start
./dev ps
./dev php -v
./dev php -m
```

The application is served at `http://localhost:8090` after Laravel is installed. Before installation, Nginx will return 404; this is expected.

## Laravel application (optional, later)

The `application/` directory is a bind mount at `/var/www`. Add an existing Laravel application there, or create one separately later. This starter does **not** run `composer create-project`, `composer install`, `npm install`, migrations, or key generation on startup.

For a Laravel application, configure **`application/.env`** independently of the root Docker `.env`:

```dotenv
DB_CONNECTION=mysql
DB_HOST=database
DB_PORT=3306
DB_DATABASE=laravel_starter
DB_USERNAME=laravel
DB_PASSWORD=laravel-local-password
REDIS_HOST=redis
REDIS_PORT=6379
```

Then, when ready:

```bash
./dev composer install
./dev artisan key:generate
./dev artisan migrate
./dev npm ci
./dev assets build
```

For a new Laravel app, ensure its `application/.env` exists before generating a key. The commands above are illustrative; use the actual app's lockfiles and setup instructions.

## Tests

`./dev test` sets `DB_CONNECTION=sqlite` and `DB_DATABASE=:memory:` for the test process. Laravel's own `phpunit.xml` / `.env.testing` may override or interact with these settings; explicitly configure tests and use `RefreshDatabase` where appropriate. `./dev test-mysql` runs the app's normal test configuration; use a **separate testing database** to prevent destructive tests touching development data. Do not run it against the development database.

SQLite tests do not fully reproduce MySQL locking, transaction and SQL-dialect behaviour.

## Commands

Run `./dev help` for all supported commands.

## Configuration / safety

- Root `.env` configures Docker; `application/.env` configures Laravel.
- Ports bind to `127.0.0.1` by default; adjust APP_PORT, DB_PORT and REDIS_PORT to avoid collisions.
- Compose uses an isolated project name and named volumes.
- `./dev stop` preserves data; `./dev purge` **deletes database and Redis volumes**.
- Xdebug defaults to off. Set `XDEBUG_MODE=debug` in root `.env` and recreate the app container to enable; use an IDE listener on port 9003.
- This is a **local-development-only** configuration; not hardened for production.
- PHP package patch levels depend on the package repository at image build time; rebuild to receive updates.
- Laravel is not pinned or installed; choose the latest stable compatible release when adding an application.

## Validation

```bash
bash -n dev docker-configuration/entrypoint/start-container docker-configuration/entrypoint/start-container-local
./dev config
./dev build
./dev start
./dev php -v
./dev php -r 'echo extension_loaded("pdo_sqlite") ? "SQLite OK\n" : "SQLite missing\n";'
./dev redis ping
```
