# grants-ui-admin

[![Security Rating](https://sonarcloud.io/api/project_badges/measure?project=DEFRA_grants-ui-admin&metric=security_rating)](https://sonarcloud.io/summary/new_code?id=DEFRA_grants-ui-admin)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=DEFRA_grants-ui-admin&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=DEFRA_grants-ui-admin)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=DEFRA_grants-ui-admin&metric=coverage)](https://sonarcloud.io/summary/new_code?id=DEFRA_grants-ui-admin)

Internal CDP frontend for Grants UI administration. The skeleton serves a landing
page at `/` and the CDP health check at `/health`.

- [CDP deployment](#cdp-deployment)
- [Requirements](#requirements)
  - [Node.js](#nodejs)
- [Server-side Caching](#server-side-caching)
- [Redis](#redis)
- [Local Development](#local-development)
  - [Setup](#setup)
  - [Development](#development)
  - [Production](#production)
  - [Npm scripts](#npm-scripts)
  - [Update dependencies](#update-dependencies)
  - [Formatting](#formatting)
    - [Windows prettier issue](#windows-prettier-issue)
- [Docker](#docker)
  - [Development image](#development-image)
  - [Production image](#production-image)
  - [Docker Compose](#docker-compose)
  - [Dependabot](#dependabot)
  - [SonarCloud](#sonarcloud)
- [Licence](#licence)
  - [About the licence](#about-the-licence)

## CDP deployment

The publish workflow builds and publishes a versioned image from `main`. Enable
auto-deployment for the eligible lower environments in the CDP Portal after the
first image is published. Deploy the same tested version through the Portal for
any remaining environments, including production.

In each environment, open `https://grants-ui-admin.<env>.cdp-int.defra.cloud/`
from a Defra device or VPN and confirm the page says the service is running.
Check that `/health` returns HTTP 200 and verify that the internal URL is
unreachable from an external network. Use the default CDP route without a vanity
URL.

## Requirements

### Node.js

Please install Node Version Manager [nvm](https://github.com/creationix/nvm)

To use the correct version of Node.js for this application, via nvm:

```bash
cd grants-ui-admin
nvm use
```

## Server-side Caching

We use Catbox for server-side caching. Production uses CatboxRedis by default. Other environments use CatboxMemory.
You can override the default behaviour by setting the `SESSION_CACHE_ENGINE` environment variable to either `redis` or
`memory`.

Please note: CatboxMemory (`memory`) is _not_ suitable for production use! The cache will not be shared between each
instance of the service and it will not persist between restarts.

## Redis

Redis is an in-memory key-value store. Every instance of a service has access to the same Redis key-value store similar
to how services might have a database (or MongoDB). All frontend services are given access to a namespaced prefixed that
matches the service name. e.g. `my-service` will have access to everything in Redis that is prefixed with `my-service`.

## Proxy

We are using forward-proxy which is set up by default. Services are automatically configured with the proxy environment variables when deployed.

Node.js 24 uses these variables to route outbound HTTP(S) requests through the proxy:

NODE_USE_ENV_PROXY=1
HTTPS_PROXY=...
NO_PROXY=...

No additional proxy configuration is required in the service.

## Local Development

### Setup

Install application dependencies:

```bash
npm install
```

### Git hooks

Install git hooks (optional)

```bash
npm run git:hooks
```

### Development

To run the application in `development` mode run:

```bash
npm run dev
```

### Production

To run in production mode locally without Redis, use the in-memory session cache:

```bash
SESSION_CACHE_ENGINE=memory AWS_EMF_ENVIRONMENT=Local npm start
```

### Npm scripts

All available Npm scripts can be seen in [package.json](./package.json)
To view them in your command line run:

```bash
npm run
```

### Update dependencies

To update dependencies use [npm-check-updates](https://github.com/raineorshine/npm-check-updates):

> The following script is a good start. Check out all the options on
> the [npm-check-updates](https://github.com/raineorshine/npm-check-updates)

```bash
ncu --interactive --format group
```

### Formatting

#### Windows prettier issue

If you are having issues with formatting of line breaks on Windows update your global git config by running:

```bash
git config --global core.autocrlf false
```

## Docker

### Development image

> [!TIP]
> For Apple Silicon users, you may need to add `--platform linux/amd64` to the `docker run` command to ensure
> compatibility fEx: `docker build --platform=linux/arm64 --no-cache --tag grants-ui-admin`

Build:

```bash
docker build --target development --no-cache --tag grants-ui-admin:development .
```

Run:

```bash
docker run -p 3000:3000 grants-ui-admin:development
```

### Production image

Build:

```bash
docker build --no-cache --tag grants-ui-admin .
```

Run:

```bash
docker run -p 3000:3000 grants-ui-admin
```

### Docker Compose

A local environment with:

- Floci (replacing Localstack) for AWS services (S3, SQS)
- Redis
- MongoDB
- This service.
- A commented out backend example.

```bash
docker compose up --build -d
```

### Dependabot

We have added an example dependabot configuration file to the repository. You can enable it by renaming
the [.github/example.dependabot.yml](.github/example.dependabot.yml) to `.github/dependabot.yml`

### SonarCloud

Instructions for setting up SonarCloud can be found in [sonar-project.properties](./sonar-project.properties).

## Licence

THIS INFORMATION IS LICENSED UNDER THE CONDITIONS OF THE OPEN GOVERNMENT LICENCE found at:

<http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3>

The following attribution statement MUST be cited in your products and applications when using this information.

> Contains public sector information licensed under the Open Government license v3

### About the licence

The Open Government Licence (OGL) was developed by the Controller of Her Majesty's Stationery Office (HMSO) to enable
information providers in the public sector to license the use and re-use of their information under a common open
licence.

It is designed to encourage use and re-use of information freely and flexibly, with only a few conditions.
