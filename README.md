# php84

[![Build Images](https://github.com/specsnl/php84/actions/workflows/main.yml/badge.svg)](https://github.com/specsnl/php84/actions/workflows/main.yml)

A PHP 8.4 (FPM, Apache and FrankenPHP) based Docker base image.

## Pulling the images

```
# FPM
docker pull ghcr.io/specsnl/php84:latest
docker pull ghcr.io/specsnl/php84/builder:latest
docker pull ghcr.io/specsnl/php84/builder_nodejs:latest

# Apache
docker pull ghcr.io/specsnl/php84/apache:latest
docker pull ghcr.io/specsnl/php84/apache/builder:latest
docker pull ghcr.io/specsnl/php84/apache/builder_nodejs:latest

# FrankenPHP
docker pull ghcr.io/specsnl/php84/frankenphp:latest
docker pull ghcr.io/specsnl/php84/frankenphp/builder:latest
docker pull ghcr.io/specsnl/php84/frankenphp/builder_nodejs:latest
```

The image name scheme: `ghcr.io/specsnl/php84[/{VARIANT}][/{TARGET}]:{VERSION}`

- **{VARIANT}**: omitted for FPM, otherwise `apache` or `frankenphp`
- **{TARGET}**: omitted for `runtime`, otherwise `builder` or `builder_nodejs` (also published as `node`)
- **{VERSION}**: `latest`, a release tag (i.e. `1.5.5`), `main` or `pr-<number>`

## Building the docker image(s)

There are multiple targets:

  - **runtime**: this is for *production*. It does not contain any development tools like Composer and Xdebug.
  - **builder**: this is for *development*. This is based on the runtime-target and it adds Composer, Xdebug etc.
  - **builder_nodejs**: this is for *development*. This is based on the builder-target and it adds NodeJS.

Building `runtime`-target:

```
docker build --tag ghcr.io/specsnl/php84:runtime-latest --file fpm/Dockerfile --target runtime .
```

Building `builder`-target:

```
docker build --tag ghcr.io/specsnl/php84:builder-latest --file fpm/Dockerfile --target builder .
```

Building `builder_nodejs`-target:

```
docker build --tag ghcr.io/specsnl/php84:builder_nodejs-latest --file fpm/Dockerfile --target builder_nodejs .
```

## Task commands

Available [Task](https://taskfile.dev/#/) commands:

```
* build:              Build all PHP Docker image targets of the FPM, Apache and FrankenPHP variants
* generate:           Generate all Dockerfiles from Dockerfile.tmpl
* lint:               Apply a Dockerfile linter to all Dockerfiles
* build:apache:       Build all PHP Docker image targets of the Apache variant
* build:fpm:          Build all PHP Docker image targets of the FPM variant
* build:frankenphp:   Build all PHP Docker image targets of the FrankenPHP variant
* install:hooks:      Install git hooks (pre-commit regenerates and stages Dockerfiles from template)
* lint:apache:        Apply a Dockerfile linter (https://github.com/hadolint/hadolint)
* lint:fpm:           Apply a Dockerfile linter (https://github.com/hadolint/hadolint)
* lint:frankenphp:    Apply a Dockerfile linter (https://github.com/hadolint/hadolint)
* remove:hooks:       Remove installed git hooks
* shell:apache:       Interactive shell
* shell:fpm:          Interactive shell
* shell:frankenphp:   Interactive shell
```
