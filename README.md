# app-solid

Docker Compose stack for hosting a [Solid](https://solidproject.org/) pod server, built on
[`mashlib-solid-server`](../mashlib-solid-server) (Community Solid Server v7 with redpencil.io branding).

---

## Tutorials

### Spin up a local Solid pod server

```bash
git clone <this-repo>
cd app-solid
docker compose -f docker-compose.yml -f docker-compose.dev.yml up
```

The server starts at `http://localhost:88/`.

> **Dev port note:** `BASE_URL` and `PORT` must match in the dev setup. CSS verifies DPoP tokens by fetching the WebID document over HTTP. From inside the container `localhost` only resolves to the container's own loopback, so `localhost:PORT` must be where CSS actually listens. In production this is not an issue because `BASE_URL` uses a real domain name that routes through nginx-proxy.

### Register your first pod

1. Navigate to `http://localhost:88/idp/register/`
2. Fill in your name, email and password
3. Your new pod will be created with a profile card, inbox, settings and type indexes

---

## How-to guides

### Deploy on a public domain with HTTPS

Set `BASE_URL` and the jwilder nginx-proxy labels via environment variables before starting:

```bash
BASE_URL=https://solid.example.org/ \
VIRTUAL_HOST=solid.example.org \
LETSENCRYPT_HOST=solid.example.org \
LETSENCRYPT_EMAIL=ops@example.org \
docker compose up -d
```

The container listens on port 80 and expects a [jwilder/nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) instance on a shared `proxy` network to handle TLS termination.

### Customise the image (config, templates, branding)

See the [`mashlib-solid-server` README](../mashlib-solid-server/README.md) for all overridable paths and environment variables.

---

## Discussions

### Root templates vs pod templates

**Pod templates** (`templates/pod/`) are stamped out per user when they register.
They define the initial contents of each user's storage (profile card, inbox,
type indexes, ACLs).

**Root templates** (`templates/root/`) initialise the server's root container on
first boot. They declare the container as `pim:Storage` (required by the Solid
spec) and set a public-read root ACL.

### The `/apps` container

`templates/root/apps/` creates a shared container at the root of the server
(e.g. `https://solid.example.org/apps/`) intended for hosting web applications
accessible to all pod users (e.g. Penny, Solid Forge). By default it inherits
the root ACL (public read, no write). To grant a specific administrator write
access, add an `.acl` document to `/apps` via the DataBrowser after first boot,
or include a custom `apps/.acl.hbs` in your template override.

### Use SPARQL-backed storage

> **Note:** A CSS v7-compatible SPARQL config is not yet included. The triplestore
> service in `docker-compose.yml` is currently commented out. Contributions welcome.
