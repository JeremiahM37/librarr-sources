# librarr-sources

Indexer-endpoint registry for [Librarr](https://github.com/JeremiahM37/librarr).

The Librarr binary ships with **driver implementations** -- the protocols it can speak. This repository holds the **list of indexers** to query and the URLs / mirrors for each. The two are intentionally split so:

- The driver framework can stay generic and content-neutral.
- A single endpoint going down (or being removed for legal reasons) is a one-line edit here, not a code change.

## How Librarr consumes this file

On startup Librarr resolves the registry in this order:

1. `LIBRARR_SOURCES_PATH` -- local file (takes precedence)
2. `LIBRARR_SOURCES_URL`  -- HTTP(S) URL (this repo's raw `sources.json` is a reasonable default)
3. Embedded defaults baked into the binary (offline fallback)

To use this repo's registry, set:

```bash
LIBRARR_SOURCES_URL=https://raw.githubusercontent.com/JeremiahM37/librarr-sources/main/sources.json
```

Optionally, set `LIBRARR_SOURCES_REFRESH_MIN=N` to re-fetch every N minutes.

## Schema

`sources.json` is a single JSON object with these top-level keys:

| Key | Type | Used by driver |
|-----|------|----------------|
| `version` | int | (informational) |
| `annas` | `{domain}` | Anna's-Archive-style direct-download driver |
| `audiobookbay` | `{mirrors[], trackers[]}` | AudioBookBay-style scrape driver |
| `thepiratebay` | `{url}` | Torrent-API driver |
| `gutenberg` | `{url}` | Gutendex-style public-domain ebook API |
| `openlibrary` | `{search_url, cover_url}` | Open Library JSON API |
| `librivox` | `{url}` | Librivox-style public-domain audiobook feed |
| `standardebooks` | `{url}` | OPDS catalog (Standard Ebooks) |
| `mangadex` | `{api_url, uploads_url, web_url}` | MangaDex-compatible API |
| `nyaa` | `{url}` | Nyaa-style RSS feed |
| `webnovels` | `[{id, label, url, base_url}]` | Web-novel crawler (lncrawl) |
| `libgen_mirrors` | `[]string` | Libgen `ads.php`/`get.php` mirror list |
| `zlibrary_default` | string | Library-card-style API fallback URL |

Credentials, API keys, and per-deployment knobs (Prowlarr URL/key, Z-Library account, BookTracker account, Flibusta URL) are **not** stored here -- those live in Librarr's environment variables.

## Editing the registry

To **remove** an indexer (e.g. on a takedown notice or because it's offline): delete its entry and open a PR. The next `LIBRARR_SOURCES_REFRESH_MIN` tick (or restart) picks up the change.

To **update** an endpoint (domain rotation, new mirror, etc.): edit the value and open a PR.

To **add** a new indexer: the driver implementation must already exist in the Librarr binary. If it does, add a JSON entry in the matching driver shape. If it doesn't, open an issue on the [Librarr repository](https://github.com/JeremiahM37/librarr) describing the protocol.

PR checklist:

- [ ] `sources.json` is valid JSON (run `jq . sources.json`).
- [ ] No credentials, API keys, or per-user secrets included.
- [ ] One change per PR where practical.

## License

MIT
