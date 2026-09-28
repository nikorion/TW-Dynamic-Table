# TW-Dynamic-Table

A [TiddlyWiki](https://tiddlywiki.com) plugin providing `<<dyntable>>`: an editable, sortable, paginated HTML table rendered from a filter — one row per tiddler, one column per field or data-index.

Extracted from [Shiraz](https://github.com/kookma/TW-Shiraz)'s Dynamic Table feature (by Mohammad Rahmani) as an independent plugin, with no dependency on Shiraz itself. Safe to install alongside Shiraz — see the readme's *Compatibility* section.

```
<<dyntable filter:"[tag[task]]" fields:"title status tags modified"/>>
```

Full parameter list, column-template mechanism, and settings are documented in the plugin's own readme (`src/dyntable/language/<lang>/readme.tid`), visible from the control panel's Plugins tab once installed.

## Development

```sh
pnpm install
pnpm dev     # dev wiki + hot reload; the URL (random free port) is printed on start
pnpm build   # dist/TW-Dynamic-Table-Plugin.json + docs/TW-Dynamic-Table-Wiki.html
```

## Extending from another plugin

Dynamic Table knows no field but the core's. Another plugin teaches it about its own fields by tagging tiddlers, which the table collects with `[all[shadows+tiddlers]tag[…]]` — nothing to change here, and nothing happens when that plugin is absent:

| Tag | Tiddler | Evaluated with |
|---|---|---|
| `$:/tags/nk-dyntable/BodyTemplate` + `nk-dyntable-column-filter` | a cell template, picked when the filter returns something (a `nk-dyntable-column-list` naming the column wins over it) | `currentColumn` |
| `$:/tags/nk-dyntable/RowClassFilter` | a filter whose results are all added to the row's classes (`nk-dyntable-row-success`, `nk-dyntable-row-muted` are styled here) | `currentRecord` |
| `$:/tags/nk-dyntable/ColumnLabelFilter` | a filter giving the column header; the first with any output wins, so return nothing — not an empty string — for a column you do not know | `currentTiddler` = `currentColumn` |
| `$:/tags/nk-dyntable/ColumnHintFilter` | a filter giving the column header's tooltip, same contract as the label filters (first with any output wins; nothing for a column you do not know); no tooltip when none answers | `currentTiddler` = `currentColumn` |
| `$:/tags/nk-dyntable/Procedure` | procedures and functions imported into every table (`\import`), for the templates to call | — |

A body template sees `currentRecord` (the row's tiddler), `currentColumn` (the field or index), `tempTableEdit` (`getindex[mode]` is `edit` in edit mode) and `tempTableSort` (`getindex[sortIndex]`: the sort column, locked in edit mode), and may use the `dyntable-lingo`/`dyntable-lingo-text` strings (`Tables/Select`, `Tables/Format/Date`…) and the cell classes (`nk-dyntable-cell-left`, `nk-dyntable-col-fixedsize`, `nk-dyntable-date`, `nk-dyntable-overdue`, `nk-dyntable-locked-cell`). These names are the contract: [TW-PKM-Fields](https://github.com/nikorion/TW-PKM-Fields) relies on them for the pkm suite's columns (`src/pkm-fields/dyntable/`), so a rename here breaks it.

## License

MIT — see `LICENSE`.
