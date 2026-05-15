# hyperfocale-spec

Dépôt de **spécification uniquement** — pas de code, pas de build, pas de dépendances.

> ## Source de vérité canonique
>
> `spec-hyperfocale.md` est **la source de vérité unique** du format Hyperfocale. Toutes les implémentations (plugin Astro, site mathieu-drouet.com, exporter Lightroom, projet Recipes) suivent cette spec — elles ne la précèdent pas. Toute évolution du format passe par une modification ici, en premier.
>
> Version courante : **2.1-draft** (révisée le 2026-05-15). Voir le Changelog en fin de fichier.

## Contenu

- `spec-hyperfocale.md` — spec complète du format de contenu Hyperfocale
- `README.md` — présentation du projet

## Ce qu'est ce repo

Une spec de format de contenu photo universel (séries photo en Markdown + frontmatter YAML). Le format est portable entre Astro, Next.js, Hugo, 11ty, Obsidian et CMS headless.

## Implémentations de référence

- Plugin Astro : https://github.com/izo/hyperfocale-astro-plugins
- Exporter Lightroom : https://github.com/izo/hyperfocale-exporter-app

## Versioning de la spec

Format de version : `MAJOR.MINOR-status` (ex: `2.0-draft`, `2.1-rc`, `2.1`).
La version est dans le header de `spec-hyperfocale.md`.

## Règles de travail

- Ne pas créer d'autres fichiers que `spec-hyperfocale.md`, `README.md`, `CLAUDE.md`
- Ne pas générer de code d'implémentation dans ce repo
- Toute modification de la spec doit rester cohérente avec la section 0 (spec générique)
