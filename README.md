# SSL-enabled Postgres 17 image

This repository builds an SSL-enabled PostgreSQL 17 image with additional
extensions pre-installed. PostgreSQL 17 is used to match PGLite's version.

[![Deploy on
Railway](https://railway.app/button.svg)](https://railway.app/template/postgres)

### Benefits over the standard Railway Postgres template

This image provides several advantages over the default Railway Postgres template:

- **SSL enabled out of the box** - Secure connections without manual configuration
- **Pre-installed extensions** ready to use with `CREATE EXTENSION`:
  - **pg_uuidv7** - Generate UUIDv7 values (time-sortable UUIDs)
  - **pg_hashids** - Create short, unique, non-sequential IDs from numbers
  - **pgvector** - Vector similarity search for AI/ML embeddings
  - **pg_ivm** - Incremental View Maintenance for efficient materialized view updates
- **Automatic certificate renewal** - SSL certs are refreshed on restart if expiring soon

### PGLite Parity

This image is designed to maintain extension parity with
[PGLite](https://pglite.dev/extensions/), enabling you to use PGLite for fast
local development and testing while deploying to this production Postgres image
with confidence.

All extensions listed above are available in both PGLite and this image.

**Roadmap:** PostGIS support will be added once PGLite supports it.

### Why SSL?

The official Postgres image in Docker Hub does not come with SSL baked in.

Since this could pose a problem for applications or services attempting to
connect to Postgres services, we decided to roll our own Postgres image with SSL
enabled right out of the box.

### How does it work?

The Dockerfiles contained in this repository start with the official Postgres
image as base. Then the `init-ssl.sh` script is copied into the
`docker-entrypoint-initdb.d/` directory to be executed upon initialization.

### Certificate expiry

By default, the cert expiry is set to 820 days. You can control this by
configuring the `SSL_CERT_DAYS` environment variable as needed.

### Certificate renewal

When a redeploy or restart is done the certificates expiry is checked, if it has
expired or will expire in 30 days a new certificate is automatically generated.

### Available image tags

Images are automatically built weekly. Only PostgreSQL 17 is supported to
maintain parity with PGLite.

- **`:17`** - Always points to the latest 17.x minor version
- **`:17.x`** (e.g., `:17.6`) - Pins to a specific minor version for stability
- **`:latest`** - Points to PostgreSQL 17

Example usage:

```bash
# Auto-update to latest minor versions (recommended for development)
docker run ghcr.io/railwayapp-templates/postgres-ssl:17

# Pin to specific minor version (recommended for production)
docker run ghcr.io/railwayapp-templates/postgres-ssl:17.6
```

### A note about ports

By default, this image is hardcoded to listen on port `5432` regardless of what
is set in the `PGPORT` environment variable. We did this to allow connections
to the postgres service over the `RAILWAY_TCP_PROXY_PORT`. If you need to
change this behavior, feel free to build your own image without passing the
`--port` parameter to the `CMD` command in the Dockerfile.
