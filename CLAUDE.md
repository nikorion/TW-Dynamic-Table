# TW-Table — contexte projet pour Claude

> **Avant toute tâche sur ce plugin, consulter d'abord le `CLAUDE.md` du workspace** (`../CLAUDE.md`) et ses `guides/` : outillage de dev commun (pnpm, `dev.cjs`/HMR, Ctrl+C, git push), pièges PowerShell/Windows, `publishFilter`, conventions modules JS, symlink. Ci-dessous : uniquement le spécifique à TW-Table.

## Ce que c'est
Plugin TiddlyWiki (`$:/plugins/nikorion/table`) fournissant la macro `<<table>>` : tableau HTML éditable, triable, paginable, généré depuis un filtre (une ligne par tiddler, une colonne par champ ou index). Aucun JS : que du wikitext + CSS.

**Origine : extrait de [Shiraz](https://github.com/kookma/TW-Shiraz)** (dossier `tables/` de ce plugin tiers, auteur Mohammad Rahmani, MIT), suite à une cartographie de dépendances (voir `guides/extraire-plugin.md` du workspace) qui a confirmé la fonctionnalité Dynamic Table (`dt-*`) séparable proprement du système `ct-*` (create-table CSV, resté dans Shiraz, non porté ici).

## Incompatibilité avec Shiraz
Ce plugin définit les mêmes tags `$:/tags/Table/Procedure|HeaderTemplate|BodyTemplate|FooterTemplate` que Shiraz (importés en bloc par la procédure `table` via `[all[shadows+tiddlers]tag[...]]`) — **ne jamais installer les deux dans le même wiki** (double définition, comportement indéterminé), même si la macro principale porte un nom différent (`<<table>>` ici, `<<table-dynamic>>` chez Shiraz). Documenté dans le readme utilisateur.

## Renommages faits lors du portage (namespace Shiraz → nikorion/table)
| Shiraz | TW-Table |
|---|---|
| `$:/plugins/kookma/shiraz/tables/procs/` | `$:/plugins/nikorion/table/procedures/` |
| `shiraz-lingo` / `shiraz-lingo-text` | `table-lingo` / `table-lingo-text` (mécanisme i18n identique) + `table-lingo-value` ajouté (voir `language/lingo.tid`) |
| `$:/state/dynamictables`, `$:/keepstate/dynamictables` | `$:/state/nikorion/table`, `$:/keepstate/nikorion/table` |
| `$:/temp/dynamictables/delete-all-records` | `$:/temp/nikorion/table/delete-all-records` |
| `$:/config/shiraz/dynamictables/editor-type` | `$:/config/nikorion/table/editor-type` (exposé dans `settings.tid`) |
| classes CSS `shiraz-dtable-*`, `shiraz-cell-centered`, `shiraz-default-cursore` | préfixe `tbldyn-*` |
| `color-scheme` (fonction, lisait `{$:/palette}get[color-scheme]`) | `table-color-scheme` dans `procedures/helper.tid` (tag `$:/tags/Global`, renommée : nom global trop générique), aucune dépendance à Shiraz |
| variables internes `coulmnFilter` / `persistantState` | corrigées en `columnFilter` / `persistentState` (casse les templates de colonne perso qui utiliseraient l'ancien nom) |

Dépendances optionnelles conservées telles quelles (dégradation gracieuse déjà en place dans le code source, testée via `is[missing]`) : [[TW-Trashbin]] (`dt-confirm-delete.tid`, `templates/body/tbl-delete.tid`), [[TW-Pikaday]] (`templates/body/due-date.tid`). Aucune dépendance à Bootstrap : l'attribut `data-bs-theme` posé sur le conteneur est cosmétique (n'a d'effet que si un CSS Bootstrap est chargé par ailleurs), la classe `class=<<class>>` du `<table>` est fournie par l'appelant.

## Structure
```
src/table/
  procedures/
    dt-table-dynamic.tid    ← point d'entrée, \procedure table(...) (fichier gardé sous son nom d'origine)
    dt-helper.tid           ← baseState/persistentState + fonctions de clé d'état + column-label (en-tête traduit)
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
    header|body|footer/*.tid  ← un template par colonne, sélectionné via son champ tbl-column-list
                                 (default.tid = repli universel ; title/type/tags/color/email/date (created, modified)
                                 génériques ; priority/status/due-date = "Task Manager" ; tbl-checkbox/clone/
                                 delete/expand/nodetype = colonnes d'action)
  styles/
    dt-tables.css, dt-tables-var.tid (variables palette), dt-edit-tags.css, task-complete.tid
  language/
    lingo.tid                     ← table-lingo / table-lingo-text / table-lingo-value
    en-GB|fr-FR/tables.multids     ← chaînes Tables/* (en-têtes Column/*, Format/Date, pagination, suppression, priorité, statut, NodeType, infobulles…)
    en-GB|fr-FR/playground.multids ← prose du Playground du wiki de dev (Playground/*)
    en-GB|fr-FR/{readme,history,license}.tid, settings.multids
  default-config.multids   ← $:/config/nikorion/table/editor-type: simple
  settings.tid             ← onglet ControlPanel : choix éditeur d'enregistrement (simple / main-editor)
  readme.tid / history.tid / licence.tid  ← sélecteurs de langue
  plugin.info

wiki/                      ← wiki TW de dev : Playground.tid (i18n) + 3 tiddlers "Demo Task" + pied pré-rempli $:/keepstate/nikorion/table/Playground/footer/footer
dist/                      ← généré par pnpm build, gitignored
docs/                      ← TW-Table-Wiki.html standalone (distribution)
```

## Spécificités dev
- Aucun module JS → pas de `pnpm lint`, `nodemon.json` ne surveille que `plugin.info`.
- HMR : tout est `.tid`/`.css`/`.multids`, poussé à chaud dans le navigateur déjà ouvert. Un changement de `plugin.info` reboote (nodemon).
- `pnpm build` → `dist/TW-Table-Plugin.json` + `docs/TW-Table-Wiki.html`.

## Points d'attention portage
- **`tbl-column-list` pilote la sélection de template**, pas le nom de fichier : ajouter une colonne = ajouter un tiddler taggé `$:/tags/Table/{Header,Body,Footer}Template` avec ce champ, pas modifier `dt-thead`/`dt-tbody`/`dt-tfoot`.
- **Valeurs de données traduites via `table-lingo-value`** (en-têtes `Tables/Column/<champ>`, `Tables/Status/<valeur>`, `Tables/NodeType/<type>`) et non `table-lingo-text` : repli final = valeur brute (champ ou statut perso de l'utilisateur), jamais la clé. Appel obligatoire via `[function[table-lingo-value],[préfixe],<valeur>]` (piège `\function` en opérateur direct, voir `../CLAUDE.md`). Valeurs stockées toujours canoniques (jamais traduites en base).
- **Pas de clé `Tables/Column/tags`** (demande explicite) : l'en-tête `tags` reste le nom brut. Casse des en-têtes via `::first-letter` (pas `text-transform: capitalize`, faux en français).
- **Tout texte visible passe par le lingo** (y compris infobulles/`aria-label`, format de date `Tables/Format/Date`) : ajouter une chaîne = clé en-GB **et** fr-FR.
- **`body/priority.tid` et `body/status.tid` sont "Task Manager"** : fonctionnels indépendamment de `table` (colonnes optionnelles), mais font partie de la parité fonctionnelle avec Shiraz — ne pas les retirer sans le signaler dans le history.
- **Macro renommée `table-dynamic` → `table`** (2026-09-15, demande explicite) : seul le nom de la `\procedure` dans `dt-table-dynamic.tid` a changé, le nom de fichier/titre du tiddler est resté `dt-table-dynamic` par continuité avec la source Shiraz — ne pas renommer le fichier sans raison, ça casserait le suivi de provenance documenté ci-dessus.
