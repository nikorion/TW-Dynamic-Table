# TW-Dynamic-Table

A [TiddlyWiki](https://tiddlywiki.com) plugin providing `<<dyntable>>`: an editable, sortable, paginated HTML table rendered from a filter — one row per tiddler, one column per field or data-index.

Extracted from [Shiraz](https://github.com/kookma/TW-Shiraz)'s Dynamic Table feature (by Mohammad Rahmani) as an independent plugin, with no dependency on Shiraz itself. Safe to install alongside Shiraz — see the readme's *Compatibility* section.

```
<<dyntable filter:"[tag[task]]" fields:"title status priority due-date"/>>
```

Full parameter list, column-template mechanism, and settings are documented in the plugin's own readme (`src/dyntable/language/<lang>/readme.tid`), visible from the control panel's Plugins tab once installed.

## Development

```sh
pnpm install
pnpm dev     # dev wiki on http://localhost:8080, with HMR
pnpm build   # dist/TW-Dynamic-Table-Plugin.json + docs/TW-Dynamic-Table-Wiki.html
```

## License

MIT — see `LICENSE`.
