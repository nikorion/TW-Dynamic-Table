# TW-Dynamic-Table — contexte projet pour Claude

> **Avant toute tâche sur ce plugin, consulter d'abord le `CLAUDE.md` du workspace** (`../CLAUDE.md`) et ses `guides/` : outillage de dev commun (pnpm, `dev.cjs`/HMR, Ctrl+C, git push), pièges PowerShell/Windows, `publishFilter`, conventions modules JS, symlink. Ci-dessous : uniquement le spécifique à TW-Dynamic-Table.

## Ce que c'est
Plugin TiddlyWiki (`$:/plugins/nikorion/dyntable`) fournissant la macro `<<dyntable>>` : tableau HTML éditable, triable, paginable, généré depuis un filtre (une ligne par tiddler, une colonne par champ ou index). Aucun JS : que du wikitext + CSS.

**Origine : extrait de [Shiraz](https://github.com/kookma/TW-Shiraz)** (dossier `tables/` de ce plugin tiers, auteur Mohammad Rahmani, MIT), suite à une cartographie de dépendances (voir `guides/extraire-plugin.md` du workspace) qui a confirmé la fonctionnalité Dynamic Table (`dt-*`) séparable proprement du système `ct-*` (create-table CSV, resté dans Shiraz — porté depuis dans [[TW-Table]], le tableau statique).

**Ex-`TW-Table`** (2026-09-24, renommé pour libérer ce nom au profit du nouveau plugin `TW-Table` — tableau statique CSV, portage du `ct-*` de Shiraz) : macro renommée `table` → `dyntable` au même moment (sans `TW`, sans tiret, `dynamic` abrégé en `dyn` — cohérent avec le nouveau nom de dossier). Nom court `table` → `dyntable` du même mouvement (identité TW `$:/plugins/nikorion/table` → `$:/plugins/nikorion/dyntable`, changement de titre : tout wiki ayant installé l'ancien plugin doit le réinstaller sous le nouveau titre, pas de migration automatique).

## Cohabitation avec Shiraz
Installable dans le même wiki que Shiraz (2026-09-17, demande explicite — motivation : profiter des classes utilitaires CSS façon Bootstrap de Shiraz sur `class=<<dyntable ...>>`). Les 4 tags internes ont été renommés `$:/tags/Table/Procedure|HeaderTemplate|BodyTemplate|FooterTemplate` → `$:/tags/nk-Table/...` (namespace propre, plus de collision avec les tags identiques de Shiraz, importés en bloc par la procédure `dyntable` via `[all[shadows+tiddlers]tag[...]]`). Tout le reste était déjà séparé : titres de tiddlers, classes CSS (`tbldyn-*`), noms `\procedure`/`\function` globaux (`dyntable`, `table-lingo*`, `table-color-scheme` vs `table-dynamic`, `shiraz-lingo*`, `color-scheme`, `table-csv`… — vérifié contre la source Shiraz). Documenté dans le readme utilisateur (§ Styling the table), qui liste aussi les classes CSS utilitaires disponibles via Shiraz — ou, sans Shiraz, via [[TW-Tiny-Bootstrap]] (mêmes classes, s'auto-désactive si Shiraz est présent).

## Renommages faits lors du portage (namespace Shiraz → nikorion/dyntable)
| Shiraz | TW-Dynamic-Table |
|---|---|
| `$:/plugins/kookma/shiraz/tables/procs/` | `$:/plugins/nikorion/dyntable/procedures/` |
| `shiraz-lingo` / `shiraz-lingo-text` | `table-lingo` / `table-lingo-text` (mécanisme i18n identique) + `table-lingo-value` ajouté (voir `language/lingo.tid`) |
| `$:/state/dynamictables`, `$:/keepstate/dynamictables` | `$:/state/nikorion/dyntable`, `$:/keepstate/nikorion/dyntable` |
| `$:/temp/dynamictables/delete-all-records` | `$:/temp/nikorion/dyntable/delete-all-records` |
| `$:/config/shiraz/dynamictables/editor-type` | `$:/config/nikorion/dyntable/editor-type` (exposé dans `settings.tid`) |
| classes CSS `shiraz-dtable-*`, `shiraz-cell-centered`, `shiraz-default-cursore` | préfixe `tbldyn-*` |
| `color-scheme` (fonction, lisait `{$:/palette}get[color-scheme]`) | `table-color-scheme` dans `procedures/helper.tid` (tag `$:/tags/Global`, renommée : nom global trop générique), aucune dépendance à Shiraz |
| variables internes `coulmnFilter` / `persistantState` | corrigées en `columnFilter` / `persistentState` (casse les templates de colonne perso qui utiliseraient l'ancien nom) |
| `$:/tags/Table/Procedure\|HeaderTemplate\|BodyTemplate\|FooterTemplate` | `$:/tags/nk-Table/...` (2026-09-17, seul point de collision restant avec Shiraz — voir § Cohabitation) |
| macro `table-dynamic` (Shiraz) → `table` (2026-09-15) → `dyntable` (2026-09-24, renommage du plugin) | nom final : `dyntable` |

Dépendances optionnelles conservées telles quelles (dégradation gracieuse déjà en place dans le code source, testée via `is[missing]`) : [[TW-Trashbin]] (`dt-confirm-delete.tid`, `templates/body/tbl-delete.tid`), [[TW-Pikaday]] (`templates/body/due-date.tid`), [[TW-Math]] (procédure `show-number` dans `dt-show-edit-cell.tid` : cellules numériques et valeurs de `footer` via `<$math>`, réglages = ceux de TW-Math). Aucune dépendance à Bootstrap : l'attribut `data-bs-theme` posé sur le conteneur est cosmétique (n'a d'effet que si un CSS Bootstrap est chargé par ailleurs), la classe `class=<<class>>` du `<table>` est fournie par l'appelant — avec Shiraz installé (§ Cohabitation), ce peut être une de ses classes utilitaires façon Bootstrap (`w-100`, etc.).

## Structure
```
src/dyntable/
  procedures/
    dt-table-dynamic.tid    ← point d'entrée, \procedure dyntable(...) (fichier gardé sous son nom d'origine, continuité avec la source Shiraz)
    dt-helper.tid           ← baseState/persistentState + fonctions de clé d'état + column-label (en-tête traduit : libellé de l'ontologie kms, sinon Tables/Column/<nom>)
    dt-kms.tid              ← colonnes des champs de l'ontologie kms : tbl-kms-kind (garde), cellules/select/cases/pastilles, repli brut sans ontologie
    dt-maths.tid            ← count/average/median/sum/product/minall/maxall (pour footerRows)
    dt-pagination.tid       ← prev-button/next-button/limit-entries
    dt-show-edit-cell.tid   ← show-cell/edit-cell/show-cell-locked par défaut
    dt-toggle-edit-view.tid ← bouton bascule vue/édition
    dt-confirm-delete.tid   ← confirmation de suppression globale
    dt-warning-message.tid ← alerte types hétérogènes (tables construites sur indexes)
    helper.tid              ← \function table-color-scheme() (tag $:/tags/Global)
  segments/
    dt-thead.tid, dt-tbody.tid, dt-tfoot.tid  ← assemblage des lignes depuis les templates de colonne
  templates/
    header|body|footer/*.tid  ← un template par colonne, sélectionné via son champ tbl-column-list, sinon (corps)
                                 son tbl-column-filter (default.tid = repli universel ; title/type/tags/color/email/
                                 date (created, modified) génériques ; priority/status/due-date = "Task Manager" ;
                                 tbl-checkbox/clone/delete/expand/nodetype = colonnes d'action ; kms-vocab/
                                 kms-vocab-list/kms-list = champs de l'ontologie kms, choisis par leur kind)
  styles/
    dt-tables.css, dt-tables-var.tid (variables palette), dt-edit-tags.css, task-complete.tid
    table-variants.css     ← variantes `table-hover`/`thead-*`/`table-striped-*`/etc., portées depuis Shiraz `styles/tables.css` (bug copié-collé corrigé au passage : les sélecteurs `a`/`.tc-tiddlylink` de chaque `thead-*` référençaient tous `thead-primary`), aucune dépendance à Shiraz ni Tiny Bootstrap ; **copie partielle dans TW-Table** (`table-variants.css`, sans `tfoot-*`/`tbldyn-*`) : toute correction de variante à répercuter dans les deux
  language/
    lingo.tid                     ← table-lingo / table-lingo-text / table-lingo-value
    en-GB|fr-FR/tables.multids     ← chaînes Tables/* (en-têtes Column/*, Format/Date, pagination, suppression, priorité, statut, NodeType, infobulles…)
    en-GB|fr-FR/{readme,history,license}.tid, settings.multids
  default-config.multids   ← $:/config/nikorion/dyntable/editor-type: simple
  settings.tid             ← onglet ControlPanel : choix éditeur d'enregistrement (simple / main-editor)
  readme.tid / history.tid / licence.tid  ← sélecteurs de langue
  plugin.info

wiki/                      ← wiki TW de dev : Playground.tid (i18n via `detect-language-lingo`, chaînes sous `wiki/tiddlers/language/`) + tiddlers de démo (tags "Demo Task", "Demo Content" — porte les champs de l'ontologie kms —, "Demo Expense", "Demo Ticket", "Demo Student")
dist/                      ← généré par pnpm build, gitignored
docs/                      ← TW-Dynamic-Table-Wiki.html standalone (distribution)
```

## Spécificités dev
- Aucun module JS → pas de `pnpm lint`, `nodemon.json` ne surveille que `plugin.info`.
- HMR : tout est `.tid`/`.css`/`.multids`, poussé à chaud dans le navigateur déjà ouvert. Un changement de `plugin.info` reboote (nodemon).
- `pnpm build` → `dist/TW-Dynamic-Table-Plugin.json` + `docs/TW-Dynamic-Table-Wiki.html`.

## Points d'attention portage
- **`tbl-column-list` pilote la sélection de template**, pas le nom de fichier : ajouter une colonne = ajouter un tiddler taggé `$:/tags/nk-Table/{Header,Body,Footer}Template` avec ce champ, pas modifier `dt-thead`/`dt-tbody`/`dt-tfoot`. Repli (corps seulement, 2026-09-28) : `tbl-column-filter`, filtre évalué avec `currentColumn` (via `:filter[subfilter{!!tbl-column-filter}]` dans l'`emptyValue` du `$set` de `dt-tbody`) — un nom explicite gagne toujours (d'où `status.tid` prioritaire sur `kms-vocab`).
- **Valeurs de données traduites via `table-lingo-value`** (en-têtes `Tables/Column/<champ>`, `Tables/Status/<valeur>`, `Tables/NodeType/<type>`) et non `table-lingo-text` : repli final = valeur brute (champ ou statut perso de l'utilisateur), jamais la clé. Appel obligatoire via `[function[table-lingo-value],[préfixe],<valeur>]` (piège `\function` en opérateur direct, voir `../CLAUDE.md`). Valeurs stockées toujours canoniques (jamais traduites en base).
- **Pied de tableau : `footer` (lignes calculées, `calc[:décimales][@colonnes]`, rendues par `segments/dt-tfoot.tid`, rien de stocké) + `footerRows` (cellules manuelles, keepstate).** Libellés via `Tables/Footer/<Calc>` (`min`/`max` → `Minimum`/`Maximum`) ; 1re colonne = libellé, jamais calculée ; sans `@`, colonnes entièrement numériques (`column-values` dans `dt-maths.tid`). Fonctions appelées par `[function<fname>,<pn>]`.
- **Pas de clé `Tables/Column/tags`** (demande explicite) : l'en-tête `tags` reste le nom brut. Casse des en-têtes via `::first-letter` (pas `text-transform: capitalize`, faux en français).
- **Tout texte visible passe par le lingo** (y compris infobulles/`aria-label`, format de date `Tables/Format/Date`) : ajouter une chaîne = clé en-GB **et** fr-FR.
- **Colonnes de l'ontologie kms (`../TW-KMS-Ontology`) = miroir de l'éditeur TW-Base-Fields**, choisies par `kind` (templates `kms-*`, jamais par nom de champ : un champ ajouté à l'ontologie a sa colonne sans rien changer ici) : `vocab` = select dont la valeur vide (`blank-value`) remplace « Select… » (option `value=""`, champ supprimé si vide) ; `vocab-list` = cases `listField` ; `list` = `kms-pill` en vue, `bf-list-value-field` de TW-Base-Fields en édition si installé (avec `transclusion` unique par enregistrement+colonne, sinon toutes les lignes partagent la même saisie), texte sinon ; `value` = template `default`. En-tête = libellé de l'ontologie. **Garde obligatoire** avant tout appel `kms-*` : `tbl-kms-kind()` (`procedures/dt-kms.tid`) ou lecture directe de `kind` (les `tbl-column-filter`) — un `function[]` non résolu renvoie tout le wiki. `status.tid` garde son template (synchro du tag `Done`, surlignage de ligne) et retombe sur valeur brute/texte sans ontologie. **Quand l'API de l'ontologie ou `bf-list-value-field` change → aligner `dt-kms.tid`** (voir leurs CLAUDE.md).
- **`body/priority.tid` et `body/status.tid` sont "Task Manager"** : fonctionnels indépendamment de `dyntable` (colonnes optionnelles), mais font partie de la parité fonctionnelle avec Shiraz — ne pas les retirer sans le signaler dans le history.
- **Macro renommée `table-dynamic` → `table`** (2026-09-15) **→ `dyntable`** (2026-09-24, renommage du plugin en `TW-Dynamic-Table`, nom court `table` → `dyntable`) : seul le nom de la `\procedure` dans `dt-table-dynamic.tid` a changé à chaque fois, le nom de fichier/titre du tiddler est resté `dt-table-dynamic` par continuité avec la source Shiraz — ne pas renommer le fichier sans raison, ça casserait le suivi de provenance documenté ci-dessus.
