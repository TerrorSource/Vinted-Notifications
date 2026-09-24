# Vinted-Notifications (September 2026 API fix)

A real-time notification system for Vinted listings that works across all Vinted country
domains. Get instant alerts when items matching your search criteria are posted.

This is a fork that keeps the bot working after **Vinted retired the legacy catalogue API in
September 2026**. The old `www.vinted.<tld>/api/v2/catalog/items` endpoint now returns `404`
for every request; the catalogue moved to a dedicated host, `api.vinted.<tld>/svc-catalogue/items`,
behind a bearer token that the `www` host still hands out on a plain request (so no account or
API key is needed for searching).

## Fork lineage & credits

- Original project: [Fuyucch1/Vinted-Notifications](https://github.com/Fuyucch1/Vinted-Notifications)
  (unmaintained since ~late 2025).
- September 2026 API fix: [3vilrabbit/Vinted-Notifications](https://github.com/3vilrabbit/Vinted-Notifications),
  which updated the catalogue endpoint, renamed/removed filters, item parsing, and the
  "new item" detection after Vinted dropped listing timestamps.
- This fork only adds a GitHub Actions workflow that builds and publishes a ready-to-pull
  container image; the application code is unchanged from 3vilrabbit.

## Image

A prebuilt image is published to GitHub Container Registry on every push to `main`:

```
ghcr.io/terrorsource/vinted-notifications:latest
```

If you fork this repo, the workflow publishes under your own username instead. Set the package
to **public** (Packages → the package → Package settings → Change visibility) so it can be
pulled without registry credentials.

## Deploy (Docker Compose / Portainer)

```yaml
version: "3"

services:
  vinted-notifications:
    container_name: vinted-notifications
    image: ghcr.io/terrorsource/vinted-notifications:latest
    pull_policy: always
    init: true
    restart: unless-stopped
    network_mode: bridge
    environment:
      - TZ=Europe/Amsterdam
      - PUID=1000
      - PGID=1000
    ports:
      - 8200:8000        # Web UI (change the host port to taste)
      # - 8280:8080      # RSS feed (optional; uncomment if you use RSS)
    volumes:
      - /share/CACHEDEV1_DATA/Docker/vinted-notifications/data:/app/data
      - /share/CACHEDEV1_DATA/Docker/vinted-notifications/logs:/app/logs
```

Adjust the volume host paths and ports to your setup. In Portainer, deploy via **Web editor**
(paste the compose) or **Repository** — either works, since the compose pulls a registry image
and doesn't build anything.

After starting, open the Web UI (e.g. `http://<host>:8200`) to configure Telegram, queries, and
other settings. Your database, queries and config live in the mounted `data` volume, so they
survive image updates.

> **Upgrading from the old broken image?** Back up your `data` folder first — the fixed code
> runs a migration for the dropped-timestamp change. Then just swap the `image:` line and
> redeploy; your existing database and queries are preserved.

## Features

- **Web UI** to manage everything
- **Multi-country support** across all Vinted domains
- **Real-time notifications** for new listings
- **Multiple search queries** monitored at once
- **Country filtering** by seller origin
- **RSS feed** and **Telegram** integration

## Usage

### Adding a query

Add a query as a full Vinted catalog URL (filters included), via the Web UI or Telegram:

```
/add_query https://www.vinted.nl/catalog?search_text=nike%20shoes&price_to=50&currency=EUR&brand_ids[]=53
```

### Telegram commands

`/add_query`, `/remove_query <n>`, `/remove_query all`, `/queries`, `/hello`,
`/create_allowlist`, `/delete_allowlist`, `/add_country XX`, `/remove_country XX`, `/allowlist`.

### Custom notification format

```
MESSAGE = '''\
🆕 Title: {title}
💶 Price: {price}
🛍️ Brand: {brand}
<a href='{image}'>&#8205;</a>
'''
```

## Notes

- The `svc-catalogue` endpoint works on the anonymous token the `www` host hands out, so a
  logged-in token is not required for searching. (If you separately run a token refresher for
  logged-in features, it does no harm — it just isn't needed for catalogue search anymore.)
- Because upstream is unmaintained, a fork like this is responsible for its own updates. If
  Vinted changes the API again, watch 3vilrabbit (or the wider community) for the next fix and
  sync it into your fork.

## License

[GNU Affero General Public License](LICENSE), inherited from the upstream project.

## Acknowledgements

- [Fuyucch1](https://github.com/Fuyucch1) for the original project.
- [3vilrabbit](https://github.com/3vilrabbit) for the September 2026 API fix.
- [@herissondev](https://github.com/herissondev) for pyVinted, a core dependency.
