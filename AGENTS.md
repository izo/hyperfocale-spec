# AGENTS.md

Consignes pour les agents de code qui travaillent dans ce dépôt. Miroir de `CLAUDE.md` : toute modification de l'un se reporte dans l'autre.

## hyperfocale-spec

Dépôt de **spécification uniquement** — pas de code, pas de build, pas de dépendances.

> ## Source de vérité canonique
>
> `spec-hyperfocale.md` est **la source de vérité unique** du format Hyperfocale. Toutes les implémentations (plugin Astro, site mathieu-drouet.com, exporter Lightroom, projet Recipes) suivent cette spec — elles ne la précèdent pas. Toute évolution du format passe par une modification ici, en premier.
>
> Version courante : **2.15-draft** (révisée le 2026-10-25). Le header de `spec-hyperfocale.md` fait foi — cette ligne dérive (constatée 6 versions en retard le 2026-08-22) : la resynchroniser à chaque bump.

## Contenu

- `spec-hyperfocale.md` — spec complète du format de contenu Hyperfocale (couches 1 à 4)
- `README.md` — présentation du projet (tableau Écosystème avec la version du plugin, ligne Statut avec la version de la spec)
- `CLAUDE.md` — miroir de ce fichier pour Claude Code : le tenir synchronisé (règles, ligne de version)
- `fixtures/ingestion/` — fixtures de conformité de la couche 4 (corpus, snapshots, validations, diffs, gardes, identifiants, chemins, vecteurs de hash) ; leur format est dans `fixtures/ingestion/README.md`

## Ce qu'est ce repo

Une spec de format de contenu photo universel (séries photo en Markdown + frontmatter YAML). Le format est portable entre Astro, Next.js, Hugo, 11ty, Obsidian et CMS headless.

## Implémentations de référence

- Plugin Astro : https://github.com/izo/hyperfocale-astro-plugins
- Exporter Lightroom : https://github.com/izo/hyperfocale-exporter-app
- CMS Swift : https://github.com/izo/hyperfocale-cms
- hyperfocale-vault (dossier de travail, repris par Hyperfocus) : https://github.com/izo/hyperfocale-vault
- Sites : https://github.com/izo/mdr-monorepo (`mathieu-drouet.com`), https://github.com/izo/laurenceguenoun, https://github.com/izo/hyperfocale.app

L'état de chacun est en §0.5 — à relever à chaque release du plugin **et** à chaque évolution d'une implémentation.

**Hyperfocus** (https://github.com/izo/hyperfocus, `spec-hyperfocus.md`) découle de cette spec : le dossier de travail en amont de la publication, ex-profil Projet G.12. Il reprend le socle sans le redéfinir — une évolution du socle (chemins, classement de `media/`, originaux) se fait ici d'abord ; ce qui relève du projet (`type: project`, bloc `project:`, export) se fait là-bas.

## Plan de `spec-hyperfocale.md`

Repérer les sections avec `grep -n '^## \|^### ' spec-hyperfocale.md` plutôt que par numéro de ligne.

- Header : version, date de révision, tableau des implémentations (résumé d'une ligne par implémentation)
- §0 spec générique (normative, autosuffisante) → §0.5 état détaillé des implémentations
- Couches 1 (format), 2 (adaptateurs), 3 (composants UI), 4 (ingestion — §4.12 décrit les fixtures)
- Annexes, dont l'Annexe G : profils G.1 à G.11, extensions hors noyau ; G.12 n'est plus qu'un renvoi vers Hyperfocus
- Changelog en fin de fichier, entrée la plus récente en tête ; chaque entrée : `### <version> — <date>`, puis `**Ajouts**`, `**Décisions normatives**` (alternatives écartées et pourquoi), et l'impact sur les fixtures

## Versioning de la spec

Format de version : `MAJOR.MINOR-status` (ex: `2.0-draft`, `2.1-rc`, `2.1`).
La version est dans le header de `spec-hyperfocale.md`.

Un bump touche, dans le même commit : header (`**Version**` et `**Dernière révision**`), nouvelle entrée en tête du Changelog, ligne de version de `CLAUDE.md` et `AGENTS.md`, ligne Statut du `README.md`.

## Règles de travail

- Ne pas créer d'autres fichiers que `spec-hyperfocale.md`, `README.md`, `CLAUDE.md`, `AGENTS.md` — et `fixtures/`, qui porte les fixtures de conformité cross-language (décision epic #23 : la spec possède les fixtures que TypeScript et Swift doivent passer)
- Ne pas générer de code d'implémentation dans ce repo — y compris dans `fixtures/` : les valeurs attendues se calculent avec un script jetable hors dépôt, jamais commité
- Toute évolution du contrat de la couche 4 met à jour `fixtures/ingestion/` dans la même PR que la prose
- Toute modification de la spec doit rester cohérente avec la section 0 (spec générique)

## Workflow

- Mise à jour de ce fichier : reporter la même modification dans `CLAUDE.md`
- Pattern de commit : `type(spec): message` + PR via `gh pr create`
- Avant toute modification : lire `spec-hyperfocale.md` en entier (~3 570 lignes — dépasse la limite de Read, lire par tranches de ≤950 lignes, soit 4 tranches)
- À chaque release du plugin Astro : mettre à jour §0.5, le header (ligne plugin) et le README. Version réelle : `gh api -H "Accept: application/vnd.github.raw" repos/izo/hyperfocale-astro-plugins/contents/CHANGELOG.md` (CHANGELOG à la racine — le repo n'est pas un monorepo `packages/`)
- Bump de version uniquement sur changement normatif de la spec (format ou couche 4 ingestion) — une mise à jour §0.5 seule ne bumpe pas (commits `docs(0.5): ...` sans bump)
