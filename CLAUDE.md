# hyperfocale-spec

Dépôt de **spécification uniquement** — pas de code, pas de build, pas de dépendances.

> ## Source de vérité canonique
>
> `spec-hyperfocale.md` est **la source de vérité unique** du format Hyperfocale. Toutes les implémentations (plugin Astro, site mathieu-drouet.com, exporter Lightroom, projet Recipes) suivent cette spec — elles ne la précèdent pas. Toute évolution du format passe par une modification ici, en premier.
>
> Version courante : **2.10-draft** (révisée le 2026-09-29). Le header de `spec-hyperfocale.md` fait foi — cette ligne dérive (constatée 6 versions en retard le 2026-08-22) : la resynchroniser à chaque bump.

## Contenu

- `spec-hyperfocale.md` — spec complète du format de contenu Hyperfocale (couches 1 à 4)
- `README.md` — présentation du projet
- `fixtures/ingestion/` — fixtures de conformité de la couche 4 (corpus, snapshots, validations, diffs, gardes, identifiants, chemins, vecteurs de hash) ; leur format est dans `fixtures/ingestion/README.md`

## Ce qu'est ce repo

Une spec de format de contenu photo universel (séries photo en Markdown + frontmatter YAML). Le format est portable entre Astro, Next.js, Hugo, 11ty, Obsidian et CMS headless.

## Implémentations de référence

- Plugin Astro : https://github.com/izo/hyperfocale-astro-plugins
- Exporter Lightroom : https://github.com/izo/hyperfocale-exporter-app

## Versioning de la spec

Format de version : `MAJOR.MINOR-status` (ex: `2.0-draft`, `2.1-rc`, `2.1`).
La version est dans le header de `spec-hyperfocale.md`.

## Règles de travail

- Ne pas créer d'autres fichiers que `spec-hyperfocale.md`, `README.md`, `CLAUDE.md` — et `fixtures/`, qui porte les fixtures de conformité cross-language (décision epic #23 : la spec possède les fixtures que TypeScript et Swift doivent passer)
- Ne pas générer de code d'implémentation dans ce repo — y compris dans `fixtures/` : les valeurs attendues se calculent avec un script jetable hors dépôt, jamais commité
- Toute évolution du contrat de la couche 4 met à jour `fixtures/ingestion/` dans la même PR que la prose
- Toute modification de la spec doit rester cohérente avec la section 0 (spec générique)

## Workflow

- Mise à jour du CLAUDE.md : invoquer le skill `claude-md-management:revise-claude-md`
- Pattern de commit : `type(spec): message` + PR via `gh pr create`
- Avant toute modification : lire `spec-hyperfocale.md` en entier (~3 250 lignes — dépasse la limite de Read, lire par tranches de ≤950 lignes)
- À chaque release du plugin Astro : mettre à jour §0.5, le header (ligne plugin) et le README. Version réelle : `gh api -H "Accept: application/vnd.github.raw" repos/izo/hyperfocale-astro-plugins/contents/CHANGELOG.md` (CHANGELOG à la racine — le repo n'est pas un monorepo `packages/`)
- Bump de version uniquement sur changement normatif de la spec (format ou couche 4 ingestion) — une mise à jour §0.5 seule ne bumpe pas (commits `docs(0.5): ...` sans bump)
