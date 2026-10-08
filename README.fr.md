# TW-Dynamic-Table

[English](README.md) · **Français**

Un plugin [TiddlyWiki](https://tiddlywiki.com) qui fournit `<<dyntable>>` : un tableau HTML éditable, triable et paginé, construit à partir d'un filtre — une ligne par tiddler, une colonne par champ ou par index de données.

Extrait de la fonction Dynamic Table de [Shiraz](https://github.com/kookma/TW-Shiraz) (de Mohammad Rahmani) pour en faire un plugin indépendant, sans aucune dépendance à Shiraz. Il peut être installé à côté de Shiraz sans risque — voir la section *Compatibility* du readme.

```
<<dyntable filter:"[tag[task]]" fields:"title status tags modified"/>>
```

La liste complète des paramètres, le mécanisme des modèles de colonnes et les réglages sont documentés dans le readme du plugin lui-même (`src/dyntable/language/<lang>/readme.tid`), visible depuis l'onglet Plugins du panneau de contrôle une fois le plugin installé.

## Développement

```sh
pnpm install
pnpm dev     # wiki de dev + rechargement à chaud ; l'URL (port libre aléatoire) s'affiche au démarrage
pnpm build   # dist/TW-Dynamic-Table-Plugin.json + docs/ (wiki de démo, publié par la CI)
```

## Extension depuis un autre plugin

Dynamic Table ne connaît aucun champ hormis ceux du core. Un autre plugin lui fait connaître ses propres champs en taguant des tiddlers, que le tableau collecte avec `[all[shadows+tiddlers]tag[…]]` — rien à modifier ici, et rien ne se passe quand ce plugin est absent :

| Tag | Tiddler | Évalué avec |
|---|---|---|
| `$:/tags/nk-dyntable/BodyTemplate` + `nk-dyntable-column-filter` | un modèle de cellule, choisi quand le filtre renvoie quelque chose (un `nk-dyntable-column-list` qui nomme la colonne l'emporte sur lui) | `currentColumn` |
| `$:/tags/nk-dyntable/BodyTemplate` + `nk-dyntable-sort-filter` (+ `nk-dyntable-sort-type`) | la clé de tri d'une colonne calculée, utilisée quand le tableau est trié sur une colonne que nomme son `nk-dyntable-column-list` ; le type remplace le `sortType` du tableau ; `footer` calcule sur la même clé et affiche sommes/moyennes/extrêmes via la procédure nommée par un `nk-dyntable-value-format` facultatif (paramètre `value`) (exemple intégré : `templates/body/nk-size`) | l'enregistrement en entrée et `currentTiddler` |
| `$:/tags/nk-dyntable/RowClassFilter` | un filtre dont tous les résultats sont ajoutés aux classes de la ligne (`nk-dyntable-row-success` et `nk-dyntable-row-danger` sont stylées ici) | `currentRecord` |
| `$:/tags/nk-dyntable/ColumnLabelFilter` | un filtre qui donne l'en-tête de colonne ; le premier qui renvoie quelque chose l'emporte, donc ne rien renvoyer — pas une chaîne vide — pour une colonne inconnue | `currentTiddler` = `currentColumn` |
| `$:/tags/nk-dyntable/ColumnHintFilter` | un filtre qui donne l'infobulle de l'en-tête de colonne, même contrat que les filtres de libellé (le premier qui renvoie quelque chose l'emporte ; rien pour une colonne inconnue) ; pas d'infobulle si aucun ne répond | `currentTiddler` = `currentColumn` |
| `$:/tags/nk-dyntable/ColumnNoCalcFilter` | un filtre qui renvoie quelque chose pour une colonne que les lignes `footer` ne nommant aucune colonne (sans `@`) doivent ignorer — des codes d'allure numérique, pas des quantités ; une colonne nommée après `@` est tout de même calculée | `currentTiddler` = `currentColumn` |
| `$:/tags/nk-dyntable/Procedure` | procédures et fonctions importées dans chaque tableau (`\import`), à l'usage des modèles | — |

Un modèle de corps voit `currentRecord` (le tiddler de la ligne), `currentColumn` (le champ ou l'index), `tempTableEdit` (`getindex[mode]` vaut `edit` en mode édition) et `tempTableSort` (`getindex[sortIndex]` : la colonne de tri, verrouillée en mode édition), et peut utiliser les chaînes `dyntable-lingo`/`dyntable-lingo-text` (`Tables/Select`, `Tables/Format/Date`…) et les classes de cellule (`nk-dyntable-cell-left`, `nk-dyntable-col-fixedsize`, `nk-dyntable-date`, `nk-dyntable-overdue`, `nk-dyntable-locked-cell`). Ces noms constituent le contrat : [TW-PKM-Fields](https://github.com/nikorion/TW-PKM-Fields) s'appuie dessus pour les colonnes de la suite pkm (`src/pkm-fields/dyntable/`), donc un renommage ici le casse.

## Installation

**Démo en ligne** : [https://nikorion.github.io/TW-Dynamic-Table/](https://nikorion.github.io/TW-Dynamic-Table/) — pour essayer le plugin avant de l'installer.

**Depuis la bibliothèque de plugins nikorion** (TiddlyWiki propose ensuite chaque nouvelle version en mise à jour) :

1. Dans votre wiki, créer un tiddler tagué `$:/tags/PluginLibrary`, avec un champ `url` valant `https://nikorion.github.io/tw-dev/library/index.html` et une `caption` comme `nikorion`.
2. Ouvrir *Panneau de configuration → Plugins → Obtenir d'autres plugins*, choisir la bibliothèque nikorion et installer **Dynamic Table**.

**À la main** : télécharger [`TW-Dynamic-Table-Plugin.json`](https://nikorion.github.io/TW-Dynamic-Table/TW-Dynamic-Table-Plugin.json) et le glisser-déposer sur votre wiki.

Nécessite TiddlyWiki ≥ 5.3.5.

## Licence

MIT — voir `LICENSE`.
