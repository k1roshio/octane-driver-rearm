# rearm/octane-driver

Laravel Octane driver for **[Rearm](https://github.com/k1roshio/rearm-php)** — a Rust application server for Laravel 12 / PHP 8.5+.

Packagist name: **`rearm/octane-driver`**  
GitHub (this repo): **[k1roshio/octane-driver-rearm](https://github.com/k1roshio/octane-driver-rearm)**  
Author: [k1roshio](https://github.com/k1roshio)

This package is the PHP side of Rearm:

1. Listens on a Unix socket (`REARM_SOCK`)
2. Boots Laravel once from `REARM_APP_ROOT`
3. Speaks Rearm’s length-prefixed MessagePack protocol with `rearm-server`
4. Uses Octane sandbox APIs between requests
5. Registers Artisan commands: `octane:rearm`, `rearm:start`, and extends `octane:start --server=rearm`

The **Rust binary** (`rearm-server`) lives in the [rearm-php](https://github.com/k1roshio/rearm-php) monorepo / releases — install it on `PATH` or set `REARM_BIN`.

## Requirements

- PHP **8.5+**
- Extensions: `pcntl`, `sockets` (and `ffi` if you use fork scale-up)
- [`laravel/octane`](https://github.com/laravel/octane) ^2
- Laravel 12
- `rearm-server` binary

## Install

```bash
composer require rearm/octane-driver
```

Until the package is on Packagist, you can require this GitHub repo directly:

```json
{
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/k1roshio/octane-driver-rearm"
    }
  ],
  "require": {
    "rearm/octane-driver": "dev-main"
  }
}
```

Publish optional config stub:

```bash
php artisan vendor:publish --tag=rearm-config
```

Prefer adding a `'rearm' => […]` section to your app’s `config/octane.php` (see `config/rearm.php` in this package for defaults). Set:

```php
'server' => env('OCTANE_SERVER', 'rearm'),
```

## Start

```bash
# needs rearm-server on PATH (or REARM_BIN=/path/to/rearm-server)
php artisan octane:start --server=rearm
# aliases:
php artisan rearm:start --workers=8
php artisan octane:rearm --host=0.0.0.0 --port=8080
```

CLI flags and `config('octane.rearm')` are mapped to `REARM_*` env vars for the Rust process.

## Configuration (env)

Common variables (full list → `config/rearm.php` / `rearm-server --help`):

| Variable | Description |
|----------|-------------|
| `REARM_BIN` | Path to `rearm-server` |
| `REARM_HTTP_ADDR` | Listen address (`0.0.0.0:8000`) |
| `REARM_HTTP3` | Opt-in HTTP/3 (`true`/`false`) |
| `REARM_WORKERS` / `REARM_WORKERS_MAX` | Elastic PHP pool |
| `REARM_LONG_PATHS` | Split long-pool allowlist |
| `REARM_FORK_SCALE` | Warm donor + `pcntl_fork` |
| `REARM_APP_ROOT` | Laravel root (defaults to `base_path()`) |
| `REARM_SOCK` | Per-worker socket (set by the server) |

## Worker entrypoint

Usually started by `rearm-server`. Manual:

```bash
REARM_SOCK=/tmp/rearm-worker-0.sock \
REARM_APP_ROOT=/path/to/laravel \
php vendor/bin/worker.php
```

Echo worker (no Laravel, for smoke tests):

```bash
REARM_SOCK=/tmp/rearm-echo.sock \
php vendor/rearm/octane-driver/bin/echo-worker.php
```

## Protocol (summary)

- Frame: `[u32 BE length][MessagePack body]`, max 64 MiB
- PHP **listens**; Rust **connects** and keeps the connection open
- Multipart files arrive as paths on disk (`files[]` metadata)

See package source / Rearm monorepo docs for the full map.

## Related

- Server monorepo: https://github.com/k1roshio/rearm-php
- Packagist submit: use this public repo URL once tagged (`v0.1.0`)

## License

MIT © [k1roshio](https://github.com/k1roshio) — see [`LICENSE`](LICENSE).
