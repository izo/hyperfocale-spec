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
| `@regrets/hyperfocale` (v0.19.0) | https://github.com/izo/hyperfocale-astro-plugins | Plugin Astro 7 — schéma Zod, collections, presets, composants ; outils d'ingestion de la couche 4 (snapshot, diff, validation, garde, providers filesystem, Dropbox, WebDAV) |
| Exporter Lightroom | https://github.com/izo/hyperfocale-exporter-app | Export LR → format Hyperfocale avec métadonnées IPTC |
| Hyperfocale CMS | https://github.com/izo/hyperfocale-cms | Éditeur Swift (CLI, macOS, iOS) — ouvrir, éditer et publier une série sans écrire de YAML |
| hyperfocale-vault | https://github.com/izo/hyperfocale-vault | Gestion de projets photo macOS — producteur du profil Projet (G.12) |
| `mathieu-drouet.com` | https://github.com/izo/mdr-monorepo | Archive photo grandeur nature, premier pipeline d'ingestion (couche 4) |
| `laurenceguenoun.com` | https://github.com/izo/laurenceguenoun | Portfolio, plugin en couche data seule |
| `hyperfocale.app` | https://github.com/izo/hyperfocale.app | Site de l'écosystème, lui-même une archive Hyperfocale |

L'état de conformité de chacun est en **§0.5** de la spec.

> **Site vitrine.** `hyperfocale.app` présente l'écosystème. Le dépôt `izo/hyperforcale-site` (graphie sic) y a été repris le 2026-10-20 et est à archiver.

## Lire la spec

→ [`spec-hyperfocale.md`](./spec-hyperfocale.md)

La spec est organisée en quatre couches, précédées d'une spec générique :

| Couche | Contenu |
|--------|---------|
| **0 — Spec générique** | Description normative autosuffisante — lire ça pour implémenter un adaptateur |
| **1 — Format de contenu** | Filesystem, frontmatter, règles métier, types de données |
| **2 — Adaptateurs** | Astro, Next.js, Hugo, 11ty, Obsidian (+ plugin), CMS headless, Lightroom |
| **3 — Composants UI** | Vocabulaire partagé (SeriesCard, Gallery, Lightbox...) |
| **4 — Ingestion** | Contrat des outils qui publient un corpus édité dans Dropbox, iCloud Drive, WebDAV… : snapshot, diff, diagnostics, garde de publication. Ne change rien à ce qu'un lecteur doit faire |

La couche 4 s'accompagne de **fixtures de conformité** cross-language, dans [`fixtures/ingestion/`](./fixtures/ingestion/) : corpus d'entrée et résultats attendus que toute implémentation — TypeScript, Swift ou autre — doit reproduire à l'identique.

Au-delà de la série photo, l'**Annexe G** standardise **douze profils de contenu** réutilisant le même squelette : Série (`series`), Événement (`event`), Recette (`recipe`), Application (`app`), Livre (`book`), Lieu (`place`), Écran (`screen`), Portfolio (`portfolio`), Musique (`music`), Catalogue (`catalog`), Presse (`press`), Projet (`project`).

Le profil Projet décrit un **dossier de travail** géré dès la prise de vue : `media/` y porte aussi les **originaux** (RAW, sidecars `.xmp`), que les lecteurs ignorent et que la publication écarte. Un tel dossier ne se publie jamais tel quel, mais par export vers une série.

## Statut

`v2.11-draft` — spécification active, source de vérité canonique. Voir le **Changelog** en fin de `spec-hyperfocale.md` pour l'historique des révisions.
