# TW-Table

A [TiddlyWiki](https://tiddlywiki.com) plugin providing `<<table>>`: an editable, sortable, paginated HTML table rendered from a filter — one row per tiddler, one column per field or data-index.

Extracted from [Shiraz](https://github.com/kookma/TW-Shiraz)'s Dynamic Table feature (by Mohammad Rahmani) as an independent plugin, with no dependency on Shiraz itself. Do not install both at once — see the readme's *Compatibility* section.

```
<<table filter:"[tag[task]]" fields:"title status priority due-date"/>>
```

Full parameter list, column-template mechanism, and settings are documented in the plugin's own readme (`src/table/language/<lang>/readme.tid`), visible from the control panel's Plugins tab once installed.

## Development

```sh
pnpm install
pnpm dev     # dev wiki on http://localhost:8080, with HMR
pnpm build   # dist/TW-Table-Plugin.json + docs/TW-Table-Wiki.html
```

## License

MIT — see `LICENSE`.
