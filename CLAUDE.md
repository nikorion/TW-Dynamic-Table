# TW-Dynamic-Table — contexte projet pour Claude

> **Avant toute tâche sur ce plugin, consulter d'abord le `CLAUDE.md` du workspace** (`../CLAUDE.md`) et ses `guides/` : outillage de dev commun (pnpm, `dev.cjs`/HMR, Ctrl+C, git push), pièges PowerShell/Windows, `publishFilter`, conventions modules JS, symlink. Ci-dessous : uniquement le spécifique à TW-Dynamic-Table.

## Ce que c'est
Plugin TiddlyWiki (`$:/plugins/nikorion/dyntable`) fournissant la macro `<<dyntable>>` : tableau HTML éditable, triable, paginable, généré depuis un filtre (une ligne par tiddler, une colonne par champ ou index). Aucun JS : que du wikitext + CSS.

**Origine : extrait de [Shiraz](https://github.com/kookma/TW-Shiraz)** (dossier `tables/` de ce plugin tiers, auteur Mohammad Rahmani, MIT), suite à une cartographie de dépendances (voir `guides/extraire-plugin.md` du workspace) qui a confirmé la fonctionnalité Dynamic Table (`dt-*`) séparable proprement du système `ct-*` (create-table CSV, resté dans Shiraz — porté depuis dans [[TW-Table]], le tableau statique).

**Ex-`TW-Table`** (2026-09-24, renommé pour libérer ce nom au profit du nouveau plugin `TW-Table` — tableau statique CSV, portage du `ct-*` de Shiraz) : macro renommée `table` → `dyntable` au même moment (sans `TW`, sans tiret, `dynamic` abrégé en `dyn` — cohérent avec le nouveau nom de dossier). Nom court `table` → `dyntable` du même mouvement (identité TW `$:/plugins/nikorion/table` → `$:/plugins/nikorion/dyntable`, changement de titre : tout wiki ayant installé l'ancien plugin doit le réinstaller sous le nouveau titre, pas de migration automatique).

## Cohabitation avec Shiraz
Installable dans le même wiki que Shiraz (2026-09-17, demande explicite — motivation : profiter des classes utilitaires CSS façon Bootstrap de Shiraz sur `class=<<dyntable ...>>`). Les 4 tags internes ont été renommés `$:/tags/Table/Procedure|HeaderTemplate|BodyTemplate|FooterTemplate` → `$:/tags/nk-Table/...` puis `$:/tags/nk-dyntable/...` (2026-09-29) (namespace propre, plus de collision avec les tags identiques de Shiraz, importés en bloc par la procédure `dyntable` via `[all[shadows+tiddlers]tag[...]]`). Tout le reste était déjà séparé : titres de tiddlers, classes CSS (`nk-dyntable-*`), noms `\procedure`/`\function` globaux (`dyntable`, `dyntable-lingo*`, `nk-dyntable-color-scheme` vs `table-dynamic`, `shiraz-lingo*`, `color-scheme`, `table-csv`… — vérifié contre la source Shiraz). Documenté dans le readme utilisateur (§ Styling the table), qui liste aussi les classes CSS utilitaires disponibles via Shiraz — ou, sans Shiraz, via [[TW-Tiny-Bootstrap]] (mêmes classes, s'auto-désactive si Shiraz est présent).

## Renommages faits lors du portage (namespace Shiraz → nikorion/dyntable)
| Shiraz | TW-Dynamic-Table |
|---|---|
| `$:/plugins/kookma/shiraz/tables/procs/` | `$:/plugins/nikorion/dyntable/procedures/` |
| `shiraz-lingo` / `shiraz-lingo-text` | `dyntable-lingo` / `dyntable-lingo-text` (mécanisme i18n identique) + `dyntable-lingo-value` ajouté (voir `language/lingo.tid`) |
| `$:/state/dynamictables`, `$:/keepstate/dynamictables` | `$:/state/nikorion/dyntable`, `$:/keepstate/nikorion/dyntable` |
| `$:/temp/dynamictables/delete-all-records` | `$:/temp/nikorion/dyntable/delete-all-records` |
| `$:/config/shiraz/dynamictables/editor-type` | `$:/config/nikorion/dyntable/editor-type` (exposé dans `settings.tid`) |
| classes CSS `shiraz-dtable-*`, `shiraz-cell-centered`, `shiraz-default-cursore` | préfixe `nk-dyntable-*` (ex-`tbldyn-*`) |
| préfixes `dt-` (titres), `tbl-` (colonnes d'action `tbl-checkbox`…, champs `tbl-column-list`/`tbl-column-filter`, classes `tbl-row-*`, popups), variable `sv-exclude-tags` | `nk-dyntable-` partout (2026-09-29, convention `nk-` du workspace), **sauf les noms que l'utilisateur tape dans un appel, gardés courts** : colonnes d'action `nk-checkbox`/`nk-clone`/`nk-delete`/`nk-expand`/`nk-nodetype`/`nk-linktype` (`nk-` = fournie par le plugin, pas un champ : un nom nu masquerait un vrai champ homonyme) et variable `nk-exclude-tags` ; `dt-table-dynamic` → `procedures/nk-dyntable` |
| `color-scheme` (fonction, lisait `{$:/palette}get[color-scheme]`) | `nk-dyntable-color-scheme` (ex-`table-color-scheme`) dans `procedures/helper.tid` (tag `$:/tags/Global`, renommée : nom global trop générique), aucune dépendance à Shiraz |
| variables internes `coulmnFilter` / `persistantState` | corrigées en `columnFilter` / `persistentState` (casse les templates de colonne perso qui utiliseraient l'ancien nom) |
| `$:/tags/Table/Procedure\|HeaderTemplate\|BodyTemplate\|FooterTemplate` | `$:/tags/nk-Table/...` (2026-09-17, seul point de collision restant avec Shiraz — voir § Cohabitation), puis `$:/tags/nk-dyntable/...` (2026-09-29) |
| macro `table-dynamic` (Shiraz) → `table` (2026-09-15) → `dyntable` (2026-09-24, renommage du plugin) | nom final : `dyntable` |

Dépendances optionnelles conservées telles quelles (dégradation gracieuse déjà en place dans le code source, testée via `is[missing]`) : [[TW-Trashbin]] (`templates/body/nk-delete.tid`), [[TW-Math]] (procédure `show-number` dans `nk-dyntable-show-edit-cell.tid` : cellules numériques et valeurs de `footer` via `<$math>`, réglages = ceux de TW-Math). Aucune dépendance à Bootstrap : l'attribut `data-bs-theme` posé sur le conteneur est cosmétique (n'a d'effet que si un CSS Bootstrap est chargé par ailleurs), la classe `class=<<class>>` du `<table>` est fournie par l'appelant — avec Shiraz installé (§ Cohabitation), ce peut être une de ses classes utilitaires façon Bootstrap (`w-100`, etc.).

## Structure
```
src/dyntable/
  procedures/
    nk-dyntable.tid         ← point d'entrée, \procedure dyntable(...) (ex-`dt-table-dynamic` de Shiraz)
    nk-dyntable-helper.tid           ← baseState/persistentState + fonctions de clé d'état + column-label (en-tête traduit : 1er `ColumnLabelFilter` qui répond, sinon Tables/Column/<nom>)
    nk-dyntable-maths.tid            ← count/average/median/sum/product/minall/maxall (pour footerRows)
    nk-dyntable-pagination.tid       ← prev-button/next-button/limit-entries
    nk-dyntable-show-edit-cell.tid   ← show-cell/edit-cell/show-cell-locked par défaut
    nk-dyntable-toggle-edit-view.tid ← bouton bascule vue/édition
    nk-dyntable-warning-message.tid ← alerte types hétérogènes (tables construites sur indexes)
    helper.tid              ← \function nk-dyntable-color-scheme() (tag $:/tags/Global)
  segments/
    nk-dyntable-thead.tid, nk-dyntable-tbody.tid, nk-dyntable-tfoot.tid  ← assemblage des lignes depuis les templates de colonne
  templates/
    header|body|footer/*.tid  ← un template par colonne, sélectionné via son champ nk-dyntable-column-list, sinon (corps)
                                 son nk-dyntable-column-filter (default.tid = repli universel ; title/type/tags/color/email/
                                 date (created, modified) génériques ; nk-size = taille du texte en octets UTF-8, calculée en wikitext
                                 (encodeuricomponent, %XX → 1 car.), triée via nk-dyntable-sort-filter/-sort-type du
                                 template de corps (segments/nk-dyntable-tbody) ; le pied calcule sur la même clé
                                 (nk-dyntable-column-all, procedures/nk-dyntable-maths) et formate via nk-dyntable-value-format ; nk-checkbox/clone/delete/expand/nodetype =
                                 colonnes d'action)
  rows/done.tid             ← filtre de classe de ligne ($:/tags/nk-dyntable/RowClassFilter) : tag Done → nk-dyntable-row-success
  styles/
    nk-dyntable.css, nk-dyntable-var.tid (variables palette), nk-dyntable-edit-tags.css, row-tones.tid (nk-dyntable-row-success/-danger)
    table-variants.css     ← variantes `table-hover`/`thead-*`/`table-striped-*`/etc., portées depuis Shiraz `styles/tables.css` (bug copié-collé corrigé au passage : les sélecteurs `a`/`.tc-tiddlylink` de chaque `thead-*` référençaient tous `thead-primary`), aucune dépendance à Shiraz ni Tiny Bootstrap ; **copie partielle dans TW-Table** (`table-variants.css`, sans `tfoot-*`/`nk-dyntable-*`) : toute correction de variante à répercuter dans les deux
  language/
    lingo.tid                     ← dyntable-lingo / dyntable-lingo-text / dyntable-lingo-value
    en-GB|fr-FR/tables.multids     ← chaînes Tables/* (en-têtes Column/*, Format/Date, pagination, suppression, priorité, statut, NodeType, infobulles…)
    en-GB|fr-FR/{readme,history,license}.tid, settings.multids
  default-config.multids   ← $:/config/nikorion/dyntable/editor-type: simple
  settings.tid             ← onglet ControlPanel : choix éditeur d'enregistrement (simple / main-editor)
  readme.tid / history.tid / licence.tid  ← sélecteurs de langue
  plugin.info

wiki/                      ← wiki TW de dev, **sans la suite pkm** (Dynamic Table seul, avec ses compagnons) : Playground.tid (i18n via `detect-language-lingo`, chaînes sous `wiki/tiddlers/language/`) + tiddlers de démo (tags "Demo Task", "Demo Expense", "Demo Ticket", "Demo Student")
dist/                      ← généré par pnpm build, gitignored
docs/                      ← TW-Dynamic-Table-Wiki.html standalone (distribution)
```

## Spécificités dev
- Aucun module JS → pas de `pnpm lint`, `nodemon.json` ne surveille que `plugin.info`.
- HMR : tout est `.tid`/`.css`/`.multids`, poussé à chaud dans le navigateur déjà ouvert. Un changement de `plugin.info` reboote (nodemon).
- `pnpm build` → `dist/TW-Dynamic-Table-Plugin.json` + `docs/TW-Dynamic-Table-Wiki.html`.

## Points d'attention portage
- **`nk-dyntable-column-list` pilote la sélection de template**, pas le nom de fichier : ajouter une colonne = ajouter un tiddler taggé `$:/tags/nk-dyntable/{Header,Body,Footer}Template` avec ce champ, pas modifier `nk-dyntable-thead`/`nk-dyntable-tbody`/`nk-dyntable-tfoot`. Repli (corps seulement, 2026-09-28) : `nk-dyntable-column-filter`, filtre évalué avec `currentColumn` (via `:filter[subfilter{!!nk-dyntable-column-filter}]` dans l'`emptyValue` du `$set` de `nk-dyntable-tbody`) — un nom explicite gagne toujours.
- **Aucune connaissance de la suite pkm ni des tâches, volontairement** (2026-09-28, plugin compagnon autonome — `../CLAUDE.md` § Suite pkm) : les colonnes des champs du schéma vivent dans TW-PKM-Fields (`src/pkm-fields/dyntable/`), branchées par les points d'extension (`nk-dyntable-column-filter`, `$:/tags/nk-dyntable/RowClassFilter`, `…/ColumnLabelFilter`, `…/Procedure`). Ne jamais réintroduire ici un nom de champ pkm, un appel `pkm-*` ni un template « Task Manager » (`priority`/`status`/`due-date`, retirés le 2026-09-28). **Avant de renommer une variable, une procédure, une chaîne `Tables/*` ou une classe CSS qu'un template reçoit → lire `README.md` § Extending from another plugin** (le contrat) et répercuter dans `../TW-PKM-Fields/src/pkm-fields/dyntable/`, même tâche.
- **Valeurs de données traduites via `dyntable-lingo-value`** (en-têtes `Tables/Column/<champ>`, `Tables/Status/<valeur>`, `Tables/NodeType/<type>`) et non `dyntable-lingo-text` : repli final = valeur brute (champ ou statut perso de l'utilisateur), jamais la clé. Appel obligatoire via `[function[dyntable-lingo-value],[préfixe],<valeur>]` (piège `\function` en opérateur direct, voir `../CLAUDE.md`). Valeurs stockées toujours canoniques (jamais traduites en base).
- **Pied de tableau : `footer` (lignes calculées, `calc[:décimales][@colonnes]`, rendues par `segments/nk-dyntable-tfoot.tid`, rien de stocké) + `footerRows` (cellules manuelles, keepstate).** Libellés via `Tables/Footer/<Calc>` (`min`/`max` → `Minimum`/`Maximum`) ; 1re colonne = libellé, jamais calculée ; sans `@`, colonnes entièrement numériques (`column-values` dans `nk-dyntable-maths.tid`) et non écartées par un `$:/tags/nk-dyntable/ColumnNoCalcFilter` (`nk-dyntable-no-calc` ; pkm-fields y écarte les `kind` vocab/vocab-list, ex. `priority`) — un `@colonne` explicite reste calculé. Fonctions appelées par `[function<fname>,<pn>]`.
- **Pas de clé `Tables/Column/tags`** (demande explicite) : l'en-tête `tags` reste le nom brut. Casse des en-têtes via `::first-letter` (pas `text-transform: capitalize`, faux en français).
- **Tout texte visible passe par le lingo** (y compris infobulles/`aria-label`, format de date `Tables/Format/Date`) : ajouter une chaîne = clé en-GB **et** fr-FR.
- **Classes de ligne = tous les résultats des `RowClassFilter`**, pas le premier (≠ `TiddlerColourFilter` du core) ; **libellé de colonne = `:cascade`** sur les `ColumnLabelFilter` : un filtre qui renvoie une chaîne vide (ex. un `:map` dont le filtre ne renvoie rien) arrête la cascade — il doit ne rien renvoyer.
- **Macro renommée `table-dynamic` → `table`** (2026-09-15) **→ `dyntable`** (2026-09-24, renommage du plugin en `TW-Dynamic-Table`, nom court `table` → `dyntable`) ; fichier du point d'entrée renommé `dt-table-dynamic` → `nk-dyntable` le 2026-09-29 (retrait des préfixes Shiraz, demande utilisateur) : provenance tracée par le tableau ci-dessus, pas par les noms de fichiers.
