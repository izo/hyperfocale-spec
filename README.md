# hyperfocale-spec

**Source de vérité canonique** du format Hyperfocale — standard ouvert pour la gestion de séries photo, portable entre tous les outils.

> Toute évolution du format passe d'abord par une PR contre ce dépôt. Les implémentations (plugin Astro, site mathieu-drouet.com, exporter Lightroom, projet Recipes) suivent cette spec — elles ne la précèdent pas.

## Qu'est-ce que le format Hyperfocale ?

Un **contenu Hyperfocale** est une série photo : un dossier autonome contenant des images et un fichier Markdown avec frontmatter.

```
bretagne-2024/
├── index.md
└── media/
    ├── 01.jpg
    ├── 02.jpg
    └── ...
```

Deux champs suffisent pour être valide :

```yaml
---
title: "Bretagne 2024"
date: 2024-06-15
---
```

Le reste (métadonnées IPTC, GPS, couverture, mots-clés...) est optionnel et standardisé.

## Pourquoi ce format ?

- **Portable** — le même dossier fonctionne dans Astro, Next.js, Hugo, Obsidian, ou un CMS headless
- **Markdown-first** — lisible et éditable dans n'importe quel éditeur
- **Filesystem = API** — pas de base de données, pas de config à maintenir
- **IPTC-compatible** — les métadonnées photo suivent le standard professionnel de l'industrie

## Écosystème

| Outil | Dépôt | Rôle |
|-------|-------|------|
| Plugin Astro | https://github.com/izo/hyperfocale-astro-plugins | Intégration Astro 6 (routes, composants, helpers) |
| Exporter Lightroom | https://github.com/izo/hyperfocale-exporter-app | Export LR → format Hyperfocale avec métadonnées IPTC |

## Lire la spec

→ [`spec-hyperfocale.md`](./spec-hyperfocale.md)

La spec est organisée en trois couches :

| Couche | Contenu |
|--------|---------|
| **0 — Spec générique** | Description normative autosuffisante — lire ça pour implémenter un adaptateur |
| **1 — Format de contenu** | Filesystem, frontmatter, règles métier, types de données |
| **2 — Adaptateurs** | Astro, Next.js, Hugo, 11ty, Obsidian (+ plugin), CMS headless, Lightroom |
| **3 — Composants UI** | Vocabulaire partagé (SeriesCard, Gallery, Lightbox...) |

Au-delà de la série photo, l'**Annexe G** standardise des **profils de contenu** réutilisant le même squelette : Événement (`event`), Recette (`recipe`), Application (`app`).

## Statut

`v2.3-draft` — spécification active, source de vérité canonique. Voir le **Changelog** en fin de `spec-hyperfocale.md` pour l'historique des révisions.
