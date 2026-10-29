# Spécification — Format de contenu Hyperfocale

> **Source de vérité canonique.** Ce document définit le format Hyperfocale — un standard de **séries photo pour le web**, portable entre SSG (Astro, Next.js, Hugo, 11ty...), vaults Obsidian, et CMS headless (Strapi, Sanity, Payload...). Le travail qui précède la publication — projets, originaux, sélection — relève d'**Hyperfocus** ([`spec-hyperfocus.md`](https://github.com/izo/hyperfocus/blob/main/spec-hyperfocus.md)), qui découle de cette spec. Toute évolution du format doit être proposée d'abord ici, dans ce dépôt.

**Version** : 2.17-draft
**Statut** : spécification active — source de vérité canonique
**Dernière révision** : 2026-10-30

### Implémentations de référence

| Implémentation | Dépôt | Rôle | Conformité |
|----------------|-------|------|------------|
| Plugin Astro `@regrets/hyperfocale` | https://github.com/izo/hyperfocale-astro-plugins | Adaptateur Astro (couche 2) et outils d'ingestion (couche 4) | ✅ Conforme v2.16 sur le contrat §2.0 — v0.25.0 publiée sur npm le 2026-10-25 (pagination sous `page/`, tri, sous-séries à travers rangement et section : [plugin#144](https://github.com/izo/hyperfocale-astro-plugins/pull/144)) ; preset `project` retiré |
| Site `mathieu-drouet.com` | https://github.com/izo/mdr-monorepo | Consommateur grandeur nature (Astro) et **premier pipeline d'ingestion** (couche 4, via le plugin) | ⚠️ Couche 4 conforme v2.11 (plugin 0.20.0, projection publiable) ; couche data en écart — loader sans passthrough, ordre du manifeste, couverture, `.tif`, documents joints ([mdr#307](https://github.com/izo/mdr-monorepo/issues/307), [#308](https://github.com/izo/mdr-monorepo/issues/308)) |
| Site `laurenceguenoun.com` | https://github.com/izo/laurenceguenoun | Consommateur en **couche data seule** (Astro) | ✅ Conforme v2.8, inchangé en v2.11 (`embeds[]` §1.11 depuis le 2026-08-12) — plugin 0.19.0, deux écarts assumés |
| Hyperfocale CMS | https://github.com/izo/hyperfocale-cms | Éditeur Swift (CLI, macOS, iOS) : lecture tolérante, écriture stricte | ⚠️ Vise v2.12 hors couche 4 ; écriture pour une source distante et dossier Dropbox / iCloud en place, canal GitHub retiré (24 oct.) — **retrait ni compilé ni testé**, jamais essayé sur un vrai `MDR Content/` ni sur appareil ([cms#77](https://github.com/izo/hyperfocale-cms/issues/77), [#78](https://github.com/izo/hyperfocale-cms/issues/78)) |
| Exporter Lightroom | https://github.com/izo/hyperfocale-exporter-app | Source de contenu : LR → format Hyperfocale | ⚠️ Conforme v2.1 — correctifs RAW (§1.9) et date (§2.7) fusionnés le 2026-10-20 ([exporter-app#98](https://github.com/izo/hyperfocale-exporter-app/pull/98)), slug pur fusionné le 2026-10-25 ([#102](https://github.com/izo/hyperfocale-exporter-app/pull/102)), tous **non compilés** ; date sans saisie manuelle ([#101](https://github.com/izo/hyperfocale-exporter-app/issues/101)) |
| hyperfocale-vault | https://github.com/izo/hyperfocale-vault | Gestion de projets photo macOS : producteur du dossier de travail (G.12, repris par Hyperfocus) et export vers une série | ✅ Vise v2.11 (ADR-034) — vérifié contre §0 et G.12, pas contre les fixtures ; **non conforme à Hyperfocus** (pas de `type: project`, quatre statuts sur sept, ni `next`, ni journal, ni `inbox/`) |
| Site `hyperfocale.app` | https://github.com/izo/hyperfocale.app | Site de l'écosystème, lui-même une archive Hyperfocale (Astro) | ✅ Refait le 2026-10-27 sur le plugin 0.25.0 (FR/EN) : présente Hyperfocale, Hyperfocus, le plugin et ses composants ; fiches lues par `seriesLoader` |

Le détail de conformité par implémentation est en **§0.5 — État des implémentations**.

---

## Table des matières

0. [[#0 — Spec générique : qu'est-ce qu'un contenu Hyperfocale ?|Spec générique — qu'est-ce qu'un contenu Hyperfocale ?]]
0.5. [[#0.5 — État des implémentations de référence|État des implémentations de référence]]
1. [[#Philosophie]]
2. [[#Architecture en couches]]
3. [[#Couche 1 — Format de contenu]]
4. [[#Couche 2 — Adaptateurs plateforme]]
   - 2.1 Astro · 2.2 Next.js · 2.3 Hugo · 2.4 11ty · 2.5 Obsidian · 2.6 CMS headless · 2.7 Exporter Lightroom
5. [[#Couche 3 — Composants UI]]
6. [[#Couche 4 — Ingestion : sources, snapshots et publication]]
   - 4.0 Trois contrats · 4.1 Chemins · 4.2 Exclusions · 4.3 Classification · 4.4 Hash · 4.5 ContentSnapshot · 4.6 Identifiant · 4.7 ContentChangeSet · 4.8 ProviderCapabilities · 4.9 PublicationState · 4.10 Diagnostics · 4.11 Garde · 4.12 Fixtures · 4.13 Exemples
7. [[#Annexes]]
8. [[#Changelog]]

---

## 0 — Spec générique : qu'est-ce qu'un contenu Hyperfocale ?

Cette section est une description normative et autosuffisante du format. Elle peut être lue indépendamment du reste de la spec. C'est le document de référence pour implémenter un adaptateur de zéro.

### Définition

Un **contenu Hyperfocale** est une **série photo** : un ensemble cohérent de photographies, accompagné de métadonnées et d'un texte libre, stocké comme une unité autonome dans le système de fichiers.

### Anatomie

Un contenu Hyperfocale est toujours un **dossier** avec cette structure stricte :

```
<slug>/
├── index.md
└── media/
    ├── 01.jpg
    ├── 02.jpg
    └── ...
```

| Élément | Rôle | Contrainte |
|---------|------|-----------|
| `<slug>/` | Identifiant de la série | `^[a-z0-9]+(-[a-z0-9]+)*$` |
| `index.md` | Toutes les métadonnées + texte | Obligatoire |
| `media/` | Médias de la série (images, documents joints, originaux) | Plat (pas de sous-dossiers). Présent dès que la série a des fichiers locaux ; PEUT être absent sinon (§1.2) |

### Le fichier `index.md`

Il contient deux parties : un **frontmatter YAML** et un **body Markdown**.

```
---
[frontmatter YAML]
---

[body Markdown — texte libre, affiché avant la galerie]
```

#### Frontmatter minimal valide

```yaml
---
title: "Titre de la série"
date: 2024-06-15
---
```

Deux champs sont **obligatoires** : `title` (string) et `date` (ISO 8601). `title` l'est toujours. `date` l'est par défaut : une **collection** entière PEUT la rendre optionnelle en se déclarant `dateRequired: false` (§1.3, *Date requise ou optionnelle*). Une page d'index de section ne porte pas de `date` (§1.10).

#### Frontmatter complet

```yaml
---
# ── Core (standardisé) ──────────────────────────
title: "Bretagne 2024"
date: 2024-06-15
description: "Côtes sauvages du Finistère"
cover: "./media/01.jpg"     # fallback : première image alphabétique
location: "Finistère, France"
draft: false                # true = absent de la production (§1.6)
lang: "fr"                  # code ISO 639-1

# ── Extension IPTC (optionnelle) ─────────────────
iptc:
  creator: "Mathieu Drouet"
  copyright: "© 2024 Mathieu Drouet"
  keywords: [paysage, bretagne, mer]
  camera: "Fujifilm X-T5"
  lens: "XF 16-55mm f/2.8"
  city: "Brest"
  country: "France"
  country_code: "FR"
  gps: { lat: 48.39, lng: -4.49 }
---
```

### Règles invariantes

Ces règles s'appliquent quel que soit l'adaptateur ou la plateforme :

| Règle | Description |
|-------|-------------|
| **Corps avant galerie** | Le body Markdown s'affiche avant les images |
| **Tri des images** | Alphabétique par nom de fichier, au sens strict : ordre des octets UTF-8 du nom en NFC, casse comprise (§1.6). `10.jpg` précède `9.jpg`, `B.jpg` précède `a.jpg` — d'où le nommage `01.jpg`, `02.jpg`... |
| **Couverture** | `cover` du frontmatter, sinon première image alphabétique |
| **Tri des séries** | Date décroissante dans les listings ; à date égale, titre croissant puis chemin ; les séries sans date après les datées (§1.6) |
| **Brouillons** | `draft: true` → ni page, ni listing, ni flux en production (§1.6) ; ce n'est pas une protection d'accès aux fichiers |
| **Formats images** | `.jpg`, `.jpeg`, `.png`, `.webp`, `.avif`, `.tif`, `.tiff` — seuls formats alimentant la galerie. L'extension se compare sans tenir compte de la casse : `DSCF0001.JPG` est une image |
| **Fichiers système** | Un nom qui commence par `.` (`.DS_Store`, `._01.jpg`), `Thumbs.db`, `desktop.ini` et `Icon\r` (§4.2) ne sont jamais des médias : ni image, ni original, ni document joint |
| **Originaux** | RAW et sidecars (`.raf`, `.cr3`, `.nef`, `.dng`, `.xmp`… — liste fermée, §1.9) sont des originaux : ignorés par les lecteurs — ni galerie, ni documents joints, ni couverture |
| **Documents joints** | Tout autre fichier de `media/`, ni image ni original, est un document joint (§1.9), listé après la galerie |
| **Pas de récursion** | `media/` est plat, pas de sous-dossiers |
| **Passthrough** | Les champs inconnus du frontmatter ne provoquent pas d'erreur |
| **Contenu exécutable** | Pas de `index.mdx` : seul `index.md` porte une série. Le HTML rendu depuis le body est assaini (§1.12) |
| **URLs de contenu** | Un champ d'URL n'admet que `http:`, `https:`, un chemin absolu au site ou un chemin relatif : `javascript:`, `data:`, `file:` et tout autre schéma sont refusés (§1.12) |

### Ce que n'est PAS le format

- Une base de données — pas de relations entre séries, pas de joins
- Un CMS — pas d'interface d'administration
- Un format propriétaire — du Markdown standard lisible par n'importe quel outil
- Un outil de gestion de projet — le suivi d'un travail en cours (statuts, tâches, échéances, originaux) est le périmètre d'**Hyperfocus**, pas de cette spec

### Contrat minimum d'un lecteur Hyperfocale

Toute implémentation (adaptateur, script, outil) qui lit du contenu Hyperfocale **doit** :

1. Parser le frontmatter YAML de `index.md`
2. Exiger `title`, et `date` sauf dans une collection déclarée `dateRequired: false` (§1.3) ; ignorer les champs inconnus
3. Scanner `media/` — absent, il vaut un `media/` vide — en ignorant les fichiers système (§4.2), et trier les fichiers alphabétiquement au sens de §1.6 — les images alimentent la galerie, les originaux sont ignorés, les autres fichiers sont des documents joints (§1.9)
4. Utiliser `cover` ou la première image comme couverture (ni les documents joints ni les originaux ne sont jamais candidats)
5. Exclure les entrées `draft: true` des listings publics
6. Rendre le body Markdown, en HTML assaini (§1.12)
7. Ne jamais échouer sur un fichier de type inconnu dans `media/`
8. Ne pas traiter comme une série un `index.md` déclarant `type: section` — c'est un rangement, pas un contenu (§1.10)
9. Ne rendre en lien ou en ressource (`href`, `src`) qu'une URL de forme admise : le contenu d'un corpus partagé n'est pas fiable (§1.12)

---

## 0.5 — État des implémentations de référence

Section informative — un audit de conformité des implémentations connues, mis à jour à chaque révision majeure de la spec.

### Plugin Astro `@regrets/hyperfocale` (v0.25.0, publiée sur npm le 2026-10-25)

**Conformité** : ✅ Conforme sur le contrat d'adaptateur (§2.0), obligations v2.6 comprises, aligné sur les profils de l'Annexe G depuis la v0.12.0, et **§1.11 (contenus embarqués) implémentée depuis la v0.17.0** — plus aucune obligation ouverte. **La couche 4 (ingestion) est implémentée depuis la v0.19.0** : les 109 fixtures cross-language de la spec (`main@18f48de`) passent toutes. **Les deux obligations rouvertes par la v2.11 sont refermées en v0.20.0** (fusionnée le 2026-10-20, publiée sur npm le 2026-10-21) : les originaux (§1.9) et leur exclusion par la garde de publication (§4.11). Un site resté sur la 0.19.0 ne doit pas recevoir de dossier de travail (G.12). **Les versions 0.21.0 à 0.24.0** ne touchent pas le contrat §2.0 : loader Content Layer hors de `src/content/` (0.21.0, 2.12), options du loader (0.22.0), i18n et jetons des composants (0.23.0), feuilles émises à la demande (0.24.0). La 2.13 et la 2.15 avaient ouvert trois écarts hors du contrat §2.0 — pagination, tri, sous-séries (§1.6, §1.8) —, relevés sur `main@f8d1a52` et **refermés en v0.25.0** ([plugin#144](https://github.com/izo/hyperfocale-astro-plugins/pull/144), fusionnée le 2026-10-25 ; 851 tests unitaires et composants et 76 e2e verts). **v0.25.0 retire aussi le preset `project`** : changement cassant pour un site qui l'utilisait.

> Le paquet s'appelait `@izo/hyperfocale` jusqu'au 2026-08-04. Le scope `@izo` ne correspondait à aucun compte npm et aucune version n'avait jamais été publiée sous ce nom : le renommage n'a rien cassé.

| Obligation | Statut | Note |
|------------|--------|------|
| Slug regex | ✅ | Enforced par Astro |
| `title` + `date` requis | ✅ | `dateRequired` configurable via preset, comme §1.3 et §2.0.1 le prévoient (défaut `true`) |
| `description`, `cover`, `location` | ✅ | Tous optionnels, présents |
| `draft` respecté | ✅ | |
| `lang` lu | ✅ | Ajouté au schéma Zod (v0.3.0) |
| Bloc `iptc.*` | ✅ | `z.looseObject()` pour `iptc.custom.*` — champs inconnus transmis (v0.4.0) |
| Mode distant (`images[]`) | ✅ | `getSeriesImages()` détecte `images[]` ; SeriesGallery/Lightbox gèrent les URLs distantes (v0.3.0) |
| Documents joints (§1.9) | ✅ | Implémenté en **v0.7.0** : `classifyAttachment()`, `getSeriesAttachments()`, bloc `attachments:`, `files[]` en mode distant, `<SeriesAttachments>` rendu après la galerie (invariant §1.9) |
| Manifeste d'images (§1.5.1) | ✅ | Implémenté en **v0.10.0** : priorité `images:` > `images.json` > `media/`, formes courte et longue, résolution des trois formes d'URL, clé `files`. Le manifeste est lu en `?raw` puis parsé dans un `try` — un import JSON ferait échouer le bundler au parsing, là où §1.5.1 impose un repli sur `media/` sans échec de build |
| Page d'index de section (§1.10) | ✅ | Implémentée en **v0.9.0** : champ `type`, exclusion des listings, `date` non requise pour une section. `isSection()` ne teste que `type` — jamais l'absence de date (« discriminant explicite »). Helpers `isSection()` / `getSections()` exposés ; la route de section reste au site consommateur (§1.10 la donne en PEUT) |
| Séries imbriquées (§1.8) | ✅ | Implémentées en **v0.12.0** : `getSubSeries()` et line-up rendu sur la page du conteneur, tri par `lineup_order` puis date décroissante. **Depuis la v0.25.0**, le parent d'une série est son plus proche ancêtre *qui est une série* (§1.8, précisé en 2.16) : un dossier de rangement intercalé (`conteneur/lots/sous-serie/`) ou une page de section (§1.10) ne l'écarte plus. Avant, seules les entrées situées exactement un segment plus bas étaient retenues |
| Profils de contenu (Annexe G) | ✅ | Les **11 profils** de l'annexe v2.10 sont couverts depuis la **v0.14.0**, le profil `project` (G.12) depuis la **v0.20.0** ; leurs prefix restent localisés en français, ce que §2.0.1 autorise |
| Originaux (§1.9, v2.11) | ✅ | **v0.20.0** : `isOriginal()`, exclus de la galerie, des documents joints, de `files[]` et de la couverture ; l'exclusion se fait dès le glob, sans quoi Vite émettrait le fichier au build. Le motif du glob est insensible à la casse (une classe par lettre, `[rR][aA][fF]`) : une graphie mixte comme `.Raf` est exclue aussi ([#125](https://github.com/izo/hyperfocale-astro-plugins/issues/125)) |
| Garde : originaux (§4.11, v2.11) | ✅ | **v0.20.0** : `publishable(snapshot)`, `guard-originals-excluded` (`count`, `bytes`), `guard-oversize` sur la projection, règle 12 de §4.10. Fixtures rejouées sur `main@51cbf73` (207 fichiers) |
| Ingestion (couche 4) | ✅ | Implémentée en **v0.19.0**, en quatre sous-chemins **additifs** : `/ingest` (contrat, agnostique du runtime), `/ingest/fs`, `/ingest/dropbox`, `/ingest/webdav`, plus les commandes `validate`, `snapshot` et `diff` du CLI. Les 109 fixtures de `fixtures/ingestion/` (`main@18f48de`) sont copiées à une ref épinglée et rejouées en totalité. L'entrée racine n'atteint aucun code d'ingestion : rien ne change pour un site qui ne l'importe pas |
| Contenus embarqués (§1.11) | ✅ | Implémentés en **v0.17.0** : champ `embeds`, `getSeriesEmbeds()` (résolution dans l'ordre du tableau, `playable` = plateforme reconnue + `id` présent), `<SeriesEmbeds>` rendu **en façade** (le poster s'affiche, l'iframe n'arrive qu'au clic ; sans JavaScript la façade reste un lien fonctionnel), liste de plateformes ouverte (valeur inconnue → repli en lien), posters exclus du scan de galerie. Vocabulaires (`EMBED_PLATFORMS`, `ATTACHMENT_KINDS`) exportés à la racine en v0.17.1 |
| URLs (§1.6, v2.13) | ✅ | Routes injectées sur l'`id` du loader, c'est-à-dire sur le chemin (`${prefix}/[...slug]/`) : règle *Identité* respectée. **Depuis la v0.25.0**, la page *n* d'une galerie est servie à `${prefix}/[...slug]/page/[n]/` (le dossier `page` est réservé), ce qui supprime la collision entre la page 2 d'une galerie et une sous-série `2` ; l'ancienne forme `${prefix}/[...slug]/[n]/` redirige en 301 sauf collision avec une série réelle. **Les URLs des pages 2 et suivantes ont changé** |
| Passthrough racine (champs inconnus) | ✅ | `z.looseObject()` racine — les extensions site-spécifiques ne sont jamais rejetées (v0.4.0) |
| Tri date desc | ✅ | **Depuis la v0.25.0**, `getSeriesList()`, `getSubSeries()` et `querySeries()` comparent la date calendaire seule, départagent à égalité par titre puis chemin et placent les séries sans date après les datées (§1.6, 2.15) ; images et documents joints se trient dans l'ordre canonique de §4.1. Les brouillons sont absents des listings et des routes en production. Limite : le jour est lu sur la `Date` déjà coercée par le schéma, donc en UTC — une date écrite avec heure et fuseau peut tomber la veille ou le lendemain, sans effet sur `AAAA-MM-JJ` |

**Changements depuis v0.4.0** :
- **v0.7.0** — documents joints (§1.9) ; slot de layout du site consommateur (options `layout`, `injectRoutes`) ; `galleryLayout: 'grid' | 'column'` ; images locales ordonnées portant leur `alt`.
- **v0.8.0** — presets de domaine (option `preset`) ; option `listRoute` (désactive l'injection de la route d'index) ; correctif de packaging : `<SeriesFilter>`, `<SeriesMap>` et `<SeriesMasonry>` étaient livrés mais absents du champ `exports`, donc non importables.
- **v0.9.0** — page d'index de section (§1.10) ; renommage du paquet en `@regrets/hyperfocale`. Correctif structurel associé : le module virtuel `virtual:hyperfocale/collection` redéclarait le schéma en dur et avait divergé de la source sur §1.9. Il délègue désormais au schéma exporté — sans quoi le correctif §1.10 n'aurait atteint aucun site installé par `hyperfocale init`.
- **v0.10.0** — manifeste d'images externalisé (§1.5.1).
- **v0.11.0** — `pageSize` exposé dans le résultat de pagination (§3.2).
- **v0.12.0** — séries imbriquées (§1.8) : `getSubSeries()`, line-up, champ `lineup_order` ; preset canonique renommé `series` (`photo` conservé en alias déprécié, retiré en 1.0) ; option `imageOptimization` et `srcset` omis en développement, où les endpoints d'optimisation d'un hébergeur n'existent pas.
- **v0.13.0** — `images[]` valide enfin les trois formes que `getSeriesImages()` traitait déjà : une entrée `{ file: '01.jpg' }` était rejetée par Zod avant d'atteindre le helper qui savait la lire.
- **v0.14.0** — les cinq profils manquants de l'Annexe G (`event`, `app`, `book`, `place`, `screen`) ; `music` aligné sur G.8, qui donne la date de sortie optionnelle ; `published` déprécié au profit de `draft`.
- **v0.15.0** — `theme: 'none'`, qui n'injecte aucune feuille. Remonté par `laurenceguenoun.com`, premier site à monter le plugin en **couche data seule** — schéma et helpers, pages entièrement maison : il embarquait les 30 custom properties du thème sur toutes ses pages sans qu'une règle les lise. Cet usage-là n'était pas prévu ; il est désormais un cas pris en charge.
- **v0.16.0** — `theme: 'light'` / `'dark'` agissent enfin sur l'apparence (#FE-012) : l'attribut `data-hf-theme`, sur lequel `base.css` articulait déjà ses trois blocs, est posé par un script inline en `head-inline` — avant le premier paint. Referme le point *Connu* de la v0.15.0.
- **v0.17.0** — contenus embarqués (§1.11) : champ `embeds`, `getSeriesEmbeds()`, `<SeriesEmbeds>` en façade, posters exclus du scan de galerie. La construction de l'URL de lecture vit dans le composant, pas dans le schéma — §1.11 ne fige aucun gabarit d'iframe.
- **v0.17.1** — `ATTACHMENT_KINDS`, `EMBED_PLATFORMS` et leurs types exportés par l'entrée racine ; ils n'étaient atteignables que via `/helpers`, sous-chemin qui importe `astro:content` et n'est pas chargeable hors runtime Astro.
- **v0.18.0** — helpers multi-collections : la plupart des helpers acceptent un nom de collection en argument (`collectionName` pour `querySeries`), `getAllSeries()` expose la collection brute, et le cache — jusqu'ici scalaire, il servait une collection pour une autre sur un site bilingue — est indexé par collection. Besoin remonté par `mathieu-drouet.com` (une collection par locale).
- **v0.18.1** et **v0.18.2** — correctifs de sécurité, sans changement d'API : les données de contenu injectées dans un `<script>` (JSON-LD, lightbox) ne peuvent plus le refermer (`<` échappé en `\u003c`) ; `embeds[].id` ne peut plus détourner l'URL de lecture d'un embed.
- **v0.19.0** — couche 4 (ingestion) : snapshot, diff, validation (§4.10), garde de publication fail-closed côté base (§4.11), providers filesystem, Dropbox et WebDAV, états de publication, CLI `validate` / `snapshot` / `diff`. Le frontmatter y est lu en schéma YAML 1.2 *core*, comme §4.10 le demande.
- **v0.20.0** (publiée le 2026-10-21) — spec 2.11 : originaux (§1.9), projection publiable et `guard-originals-excluded` (§4.11), preset `project` (G.12). Corrige au passage `.tif` et les extensions d'image en majuscules, classées en documents joints. `ORIGINAL_EXTENSIONS` et `PROJECT_STATUSES` exportés à la racine.
- **v0.21.0** (2026-10-24) — spec 2.12 : `seriesLoader` (`/loader`) et résolveur de médias (`/media`) hors de `src/content/` ; ordre éditorial d'une source distante dans `images:` du frontmatter (§1.5.1). Besoin remonté par `mathieu-drouet.com`.
- **v0.22.0** (2026-10-24) — option `prepare` du `seriesLoader` (replis du contrat appliqués avant `parseData`), `sections`.
- **v0.23.0** (2026-10-24) — i18n des 9 composants (`locale`, `labels`, dictionnaires `fr`/`en`), jetons `--hf-*` sans couleur en dur, options par instance.
- **v0.24.0** (2026-10-25) — `SeriesAttachments` : variante `inline`, attributs par document, nom accessible, feuille émise à la demande (`componentStyles`) et importable en cascade layer.
- **v0.25.0** (2026-10-25) — alignement sur la 2.15 : pagination sous `page/` avec redirection de l'ancienne forme, tri par date calendaire, ordre canonique NFC, sous-séries à travers rangement et section ; **retrait du preset `project`**.
- Socle : **Astro 7.2.0**, TypeScript 7, Zod 4.

**Presets vs Annexe G — écart refermé en v0.12.0** :

Le plugin expose un preset par profil de l'annexe, sous le nom de l'annexe : `series`, `portfolio`, `music`, `catalog`, `press`, `recipe` depuis la v0.12.0, `event`, `app`, `book`, `place`, `screen` depuis la v0.14.0, `project` depuis la v0.20.0. Quatre d'entre eux — `portfolio`, `music`, `catalog`, `press` — y ont été standardisés par la v2.7, à partir de cette implémentation ; le sixième, `series`, s'appelait `photo` jusqu'à la v0.12.0.

| Preset plugin | Prefix plugin | Profil Annexe G | État |
|---------------|---------------|-----------------|------|
| `series` | `/series` | `series` → `/series` | ✅ nom conforme depuis la v0.12.0 ; `photo` reste un alias déprécié, retiré en 1.0 |
| `recipe` | `/recettes` | `recipe` → `/recipes` | ✅ nom conforme ; prefix localisé |
| `portfolio` | `/projets` | `portfolio` → `/portfolio` | ✅ nom conforme ; prefix localisé |
| `music` | `/discographie` | `music` → `/music` | ✅ nom conforme ; prefix localisé |
| `catalog` | `/catalogue` | `catalog` → `/catalog` | ✅ nom conforme ; prefix localisé |
| `press` | `/presse` | `press` → `/press` | ✅ nom conforme ; prefix localisé |
| `event` | `/evenements` | `event` → `/events` | ✅ v0.14.0 ; prefix localisé |
| `app` | `/applications` | `app` → `/apps` | ✅ v0.14.0 ; prefix localisé |
| `book` | `/livres` | `book` → `/books` | ✅ v0.14.0 ; prefix localisé |
| `place` | `/lieux` | `place` → `/places` | ✅ v0.14.0 ; prefix localisé |
| `screen` | `/ecrans` | `screen` → `/screens` | ✅ v0.14.0 ; prefix localisé |
| ~~`project`~~ | ~~`/suivi`~~ | ~~`project` → `/projects`~~ | ⛔ **retiré en v0.25.0** (décision du propriétaire, 2026-10-25). Il avait été ajouté en v0.20.0 (collection `work`). G.12 est passé dans Hyperfocus en 2.14 ; le plugin ne prend pas Hyperfocus en charge (D18 d'Hyperfocus, §7.4) et son schéma refuse `type: project`. Le bloc `project:` du schéma et `PROJECT_STATUSES` restent exportés, sans rendu |

Les prefix divergents ne sont pas des non-conformités — la colonne de l'Annexe G est un **prefix recommandé**, et un preset PEUT fixer le sien (§2.0.1) ; le plugin les a simplement localisés en français. Le squelette, le slug regex et les champs core sont préservés, conformément aux interdits de §2.0.1.

Les dix profils de l'annexe sont couverts depuis la v0.14.0. Depuis la 2.14, le profil `project` n'est plus dans l'annexe : G.12 est passé dans Hyperfocus. Le preset correspondant (v0.20.0) est **retiré en v0.25.0** ; le plugin ne prend pas Hyperfocus en charge (D18 d'Hyperfocus, §7.4). Le nom de collection d'un preset n'est pas normé par l'annexe : `work` pour `project` n'est pas un écart.

**Extensions au-delà du contrat** :
- `featured: boolean` (boost ranking) — pattern utile, officialisé en §1.3 v2.1
- `tags: string[]` — pattern utile, officialisé en §1.3 v2.1
- `published: boolean` — redondant avec `draft` ; arbitré en v0.14.0 au profit de `draft`, seul champ standardisé par §1.3. Déprécié, retiré en 1.0
- Module virtuel Vite `virtual:hyperfocale/collection` — pattern d'implémentation Astro

### Site `mathieu-drouet.com` (dépôt `izo/mdr-monorepo`)

**Conformité** : ⚠️ Couche 4 conforme v2.11 ; couche data **largement refermée depuis le passage au plugin** (`seriesLoader` ≥ 0.21, `resolveSeriesMedia`, plugin ^0.24.0 au 2026-10-25) : il reste le tri (§1.6, 2.15) et le `.tif` assumé. Relevé le 2026-10-25 sur `mdr-monorepo@3fd9008`, par lecture du code ([mdr-monorepo#331](https://github.com/izo/mdr-monorepo/issues/331)) ; rien exécuté. Anciens écarts : [mdr#307](https://github.com/izo/mdr-monorepo/issues/307), [#308](https://github.com/izo/mdr-monorepo/issues/308).

> Le site vivait dans `izo/mathieu-drouet.com` jusqu'au 2026-08-22. Ce dépôt est archivé ; contenu et site sont dans `izo/mdr-monorepo`, organisé autour du contenu (`content/`) et consommé par le site Astro, les apps iOS/tvOS et Android, et un serveur MCP. Migration v2.1 terminée le 2026-05-19 (`docs/migration-spec-v2.1.md`).

| Obligation | Statut | Note |
|------------|--------|------|
| Slug regex | ✅ | |
| `title` + `date` requis | ✅ | `baseSeriesSchema({ dateRequired: true })` appliqué par `seriesLoader` (`parseData`) ; `series_id` et `description`, que le corpus n'a pas toujours, sont posés par `prepare` (replis signalés au build) |
| Passthrough (champs inconnus) | ✅ | Le loader maison et sa liste blanche de 13 champs sont remplacés par `seriesLoader` ; le schéma du socle est `looseObject` à la racine, et `serializeSeries` sélectionne explicitement les champs publiés |
| Dossier `media/` | ✅ | Médias hors dépôt (disque local + bucket R2 privé), servis par le Worker |
| Manifeste d'images (§1.5.1) | ✅ | Séries : `resolveSeriesMedia` du plugin (`images:` > `images.json` > `media/`, ordre du tableau, formes courte et longue). Reste hors spec et assumé : `listMedia` (boutique, portfolio) range encore la couverture en tête puis trie par `localeCompare` |
| Couverture par défaut (§1.6) | ✅ | Première entrée rendue par le résolveur du plugin (séries) |
| Pas de récursion (§1.8) | ✅ | Le repli sur `media/` passe par `resolveSeriesMedia` (`withFileTypes`, fichiers seulement) |
| Formats images (`.tif`, `.tiff`) | ⚠️ | Écart **assumé** : le TIFF reste une image pour l'ingestion (R2) mais est écarté des galeries (`isGalleryImagePath`) ; la garde l'annonce par `x-mdr-tiff-not-displayed` (warning) pour que le contributeur exporte en JPEG |
| Documents joints (§1.9), mode distant (§1.5) | ⚠️ | `SeriesAttachments` du plugin 0.24 ne sert qu'au ZIP de la série (`download: true`, badge « ↓ ZIP ») ; le rendu des autres documents de `media/` n'a pas été relevé, et le mode distant (`files[]`) n'est pas exercé |
| Page d'index de section (§1.10) | ✅ | 16 `index.md` en `type: section` |
| `draft` respecté | ✅ | |
| Bloc `iptc.*` | ✅ | 548 `index*.md` le portent ; le site en retire les métadonnées des séries privées (ADR-012) |
| Contenus embarqués (§1.11) | — | Non exercés : aucune série ne porte `embeds:` |
| Ingestion (couche 4) | ✅ | Runner local (agent launchd) sur `@regrets/hyperfocale/ingest`, `/ingest/dropbox` (source en vigueur : l'API Dropbox) et `/ingest/fs` (plugin `^0.20.0`) : snapshot, diff, validation, garde. Dropbox fait foi (`MDR Content/`), aucune primitive générique dans le site. Garde site-spécifique `x-mdr-private-exposed` (§4.10, code `x-*`) |
| Originaux (§1.9, v2.11) | ✅ | Non exercés — le corpus n'en contient pas —, mais protégés depuis le 2026-10-20 ([mdr-monorepo#246](https://github.com/izo/mdr-monorepo/pull/246)) : diff, garde et snapshot publié passent par `publishable()` du plugin 0.20.0 |
| Tri date desc | ⚠️ | `sortSeriesByDate` (`src/lib/utils.ts`) trie sur `getTime()` **sans départage** : la 2.15 demande la date calendaire seule, puis titre croissant, puis chemin (§1.6). Le plugin 0.25.0 expose `compareSeriesOrder` (`src/order.ts`, non exporté publiquement) ; à aligner côté site |

**Extensions strictement site-spécifiques (hors spec)** :
- `private` + `password_hash` — séries protégées par mot de passe (connues de la garde §4.11 comme extension de site)
- `artist`, `bio_source`, `genres` — pipeline de génération de bios via LLM
- Routage par collection (`series`, `series_fr`, `projects_*`, `store_*`) — pattern i18n documenté en Annexe F stratégie 3. Conventions de nommage des locales divergentes selon le type de contenu (archives : `index.md` = FR ; pages : `{slug}.md` = EN)
- Champs e-commerce (`price`, `paage_url`, `source_url`, `format`, `edition`, `available`) — strictement hors spec

### Site `laurenceguenoun.com`

**Conformité** : ✅ Conforme v2.8 (mise en conformité du 2026-08-23), inchangée en v2.11 — ni ingestion, ni originaux, ni dossier de travail. Plugin **0.19.0** (2026-10-20, HTML identique à l'octet) en **couche data seule** : `baseSeriesSchema` et helpers, `injectRoutes: false`, `theme: 'none'`, pages et design entièrement maison.

| Obligation | Statut | Note |
|------------|--------|------|
| Structure §1.2 | ✅ | `galleries/<slug>/index.md` + `media/` depuis le 2026-08-23 (193 médias) |
| `title` + `date` | ✅ | `date` posée sur les 8 séries — la même pour toutes : galeries thématiques sans chronologie. Le site trie par `order` ; tout consommateur de ce contenu doit faire de même, le tri par défaut de §1.6 y est indéterminé |
| `type` | ✅ | `series` partout ; la nature d'une galerie se lit à la présence de `images[]` ou `embeds[]` |
| Contenus embarqués (§1.11) | ✅ | 11 embeds Vimeo à la forme §1.11 depuis le 2026-08-12 — l'ancien bloc `videos[]` a servi de prototype à §1.11 |
| Bloc `iptc.*`, `tags`, `location` | ✅ | Renseignés depuis le 2026-08-23 |
| `draft` respecté | ⚠️ | Les champs à défaut du plugin existent dans `entry.data` mais le routage du site les ignore : `draft: true` ne retire pas une galerie. Relevé le 2026-10-25 : `getStaticPaths` de `[slug].astro` ne filtre rien (documenté dans le `CLAUDE.md` du site) ; sans effet tant qu'aucune galerie n'est en brouillon |

**Écarts assumés** : `images[].src` et `embeds[].poster` sont surchargés en `image()` d'Astro (validation au build, WebP responsive) ; `getSeriesEmbeds()` et `<SeriesEmbeds>` ne sont donc pas utilisés.

### Hyperfocale CMS (Swift — CLI, macOS, iOS)

**Conformité** : ⚠️ Vise **v2.12-draft** hors couche 4 (`Hyperfocale.specVersion`, depuis le 2026-10-24). Relevé le 2026-10-25 sur `main@ac59e39`, par lecture du code et de son `CLAUDE.md` : rien n'a été compilé ni exécuté.

- Noyau `HyperfocaleCore` sans accès au système de fichiers : **lecture tolérante, écriture stricte** (champs inconnus, ordre des clés et style YAML préservés ; n'émet que du Hyperfocale conforme).
- Couvre le core, `iptc.*`, documents joints, manifeste `images.json`, `embeds` (§1.11), profils de l'Annexe G (date optionnelle selon le profil), validation (`hyperfocale check`).
- **Non implémentée** : la couche 4. Décision du 2026-09-29 : le CMS écrit dans Dropbox, la synchro du site publie. Les apps et le CLI n'ouvrent plus qu'un **dossier** (local, Dropbox ou iCloud Drive) ; un dossier Dropbox ou iCloud est reconnu au chemin comme une source distante (`CloudWorkspace`, première tranche de [cms#77](https://github.com/izo/hyperfocale-cms/issues/77)). Pas d'accès à l'API Dropbox ni de File Provider complet.
- **Écriture pour une cible distante** : refermée le 2026-10-24 ([cms#93](https://github.com/izo/hyperfocale-cms/pull/93), spec 2.12) — jamais d'`images.json` dans une source distante, l'ordre non alphabétique va dans `images:` ; `series_id` toujours écrit ; `draft: true` par défaut. Validation alignée sur §4.10 et 31 fixtures de la spec rejouées ([cms#92](https://github.com/izo/hyperfocale-cms/pull/92)). Les contraintes du site (`MDR Content/` : exclusions, média de plus de 40 Mio écarté, TIFF signalé) sont codées en dur.
- **Originaux (§1.9)** : portés dans `HyperfocaleCore` ([cms#86](https://github.com/izo/hyperfocale-cms/pull/86), [#91](https://github.com/izo/hyperfocale-cms/pull/91)) ; `swift test` rendait 1 315 tests verts le 2026-10-24, avant le retrait du code GitHub. **Profil `project` (G.12)** : repris par Hyperfocus depuis la 2.14, le CMS ne l'a pas relevé.
- **Écarts relevés à la lecture le 2026-10-25** (§4.2, §1.8) : le magasin **liste** les fichiers cachés et l'inventaire de `media/` (`MediaResolver.inventory`) ne filtre aucun fichier système — un `.DS_Store` ressort comme pièce jointe, et `Thumbs.db`, `desktop.ini` et `Icon\r` ne sont nulle part nommés. Seuls les dossiers cachés et le calcul des collisions écartent les chemins à point. Correctif fusionné le 2026-10-25 ([cms#106](https://github.com/izo/hyperfocale-cms/pull/106), `SystemFile.isSystem` appliqué à l'inventaire de `media/`) — **non compilé ni testé**, à vérifier par `swift test --filter SystemFileTests`. `ContentDiscovery` ne traite jamais une section comme un conteneur (`ContentDiscovery.swift:124-129`) ; ce que la 2.16 de §1.8 dit aussi. Non relevé sur une série rangée sous une section dans un conteneur.
- **Canal GitHub retiré** ([cms#78](https://github.com/izo/hyperfocale-cms/issues/78), étapes 1 à 5, fusionné le 2026-10-24) : plus de connexion, de jeton, de `publish` ni de `status`. ⚠️ Le retrait a été fait par grep et relecture : **ni `swift build`, ni `swift test`, ni `xcodebuild` n'ont été lancés depuis**, et rien n'a jamais tourné sur un vrai `MDR Content/` ni sur appareil (de l'aveu du `CLAUDE.md` du dépôt).

### hyperfocale-vault (macOS)

**Conformité** : ✅ Vise **v2.11-draft** — c'est le besoin qui a fondé les originaux (§1.9) et le profil Projet (G.12) ; ADR-034 du dépôt.

- **Producteur G.12** : un projet est un dossier `<slug>/index.md` + `media/` portant les originaux ; `draft: true` toujours écrit, aucun `type`, bloc `project:` et `iptc:`. Le fichier fait foi, les champs inconnus et le body sont préservés.
- **Export vers une série** (« Exporter en série web… ») : nouveau dossier `<slug>/`, `media/` d'images rendues seulement, sans bloc `project:` — la voie de publication que la v2.11 prescrit.
- **Écart** : vérifié contre les règles de §0 et G.12, pas contre les fixtures de `fixtures/ingestion/`, absentes du dépôt.
- **Hyperfocus (2.14)** : G.12 est repris par Hyperfocus ([`spec-hyperfocus.md`](https://github.com/izo/hyperfocus/blob/main/spec-hyperfocus.md)). Relevé le 2026-10-25 sur `main@41ff8f5`, par lecture du code : le vault produit un dossier **sans** `type: project` (Hyperfocus en fait l'unique discriminant) ; son `ProjectStatus` connaît `prospect`, `en-cours`, `livre`, `archive` et un cas `other(String)`, contre sept statuts (`en-pause` et `abandonne` manquent) ; aucune trace des opérations `set-next`, `add-journal-entry`, `capture` / `file-note` (`inbox/`) ni `adopt-project` n'a été trouvée à la recherche (`inbox`, `adopt`, `journal`, `en-pause`, `abandonne` : seuls des journaux d'audit sans rapport). Aucune des fixtures d'Hyperfocus n'a été rejouée. Le vault est donc un producteur G.12, pas encore un outil Hyperfocus. **Décision du propriétaire (2026-10-25)** : il devient `hyperfocale-photos`, utilise la spec Hyperfocus et se spécialise dans les projets photographiques ([vault#189](https://github.com/izo/hyperfocale-vault/issues/189)) ; le dépôt n'est pas encore renommé.

### Site `hyperfocale.app`

**Conformité** : ✅ Site de l'écosystème, lui-même une archive Hyperfocale, refait de zéro le 2026-10-27 ([hyperfocale.app#14](https://github.com/izo/hyperfocale.app/pull/14)) sur le plugin `@regrets/hyperfocale` **0.25.0** : Astro 7.4, Tailwind v4, bilingue FR/EN. Les quatorze fiches projets sont des contenus du profil Application (G.3) rangés dans `src/content/projects/<slug>/`, `index.md` (FR) et `index.en.md` (EN), lus par `seriesLoader` avec repli de locale (Annexe F, stratégies 2 et 3). Deux séries photo de démonstration (`src/content/demo/`, `seriesSchema`) font tourner `SeriesList`, `SeriesMasonry` et `SeriesGallery` sur la page du plugin. Pages alignées sur la 2.16 : onze profils, G.12 renvoyé à Hyperfocus, qui a sa page.

Deux écarts du plugin y ont été relevés et corrigés en amont (0.25.1, [plugin#148](https://github.com/izo/hyperfocale-astro-plugins/issues/148), [#149](https://github.com/izo/hyperfocale-astro-plugins/issues/149)) : le barrel `/components` n'exportait aucun composant, et `virtual:hyperfocale/collection` n'avait pas de déclaration de type. Le site garde ses contournements jusqu'à la publication de la 0.25.1 sur npm. Déploiement : Cloudflare Workers, assets statiques ([hyperfocale.app#16](https://github.com/izo/hyperfocale.app/pull/16)).

> Le dépôt `izo/hyperforcale-site` (graphie sic, contenu vitrine sans code) a été repris dans `hyperfocale.app` le 2026-10-20 et est à archiver.

### Exporter Lightroom (SwiftUI, v0.1.0)

**Conformité** : ⚠️ Conforme v2.1 sur le format ; les écarts originaux (RAW) et date sont corrigés dans le code depuis la fusion de [exporter-app#98](https://github.com/izo/hyperfocale-exporter-app/pull/98) (2026-10-20, `main@25344d5`), **non compilés ni testés** ; le slug legacy est encore produit — **décision du 2026-10-25 : slug pur du titre**, implémentée et fusionnée le jour même ([exporter#102](https://github.com/izo/hyperfocale-exporter-app/pull/102), `generateSlug(title:)`), **non compilée ni testée** ; sans suffixe automatique de collision (l'export écrase un dossier existant), ni garantie de la regex pour un titre non latin. Relevé le 2026-10-25 par lecture du code. La version Tauri est abandonnée depuis le 2026-10-20 ([exporter-app#100](https://github.com/izo/hyperfocale-exporter-app/issues/100)) : son backend Rust n'a jamais été versionné.

| Obligation | Statut | Note |
|------------|--------|------|
| Dossier `media/` | ✅ | |
| Slug regex | ✅ | Slug pur du titre depuis [exporter#102](https://github.com/izo/hyperfocale-exporter-app/pull/102) (fusionnée le 2026-10-25, **non compilée**) ; le format `hyperfocale-{year}-{slug}-{seq}` n'est plus produit ([exporter#45](https://github.com/izo/hyperfocale-exporter-app/issues/45)). Reste non traité : titre non latin (lettres hors ASCII acceptées par `slugify`), titre sans lettre (slug vide) |
| `date` (§2.7) | ✅ | `seriesDate` prend le jour de capture le plus récent (`days.max()`), repli sur le jour d'export ; correctif de [exporter-app#98](https://github.com/izo/hyperfocale-exporter-app/pull/98), **non compilé** ([#44](https://github.com/izo/hyperfocale-exporter-app/issues/44)). Sans aucune date de capture le repli est silencieux : la saisie manuelle que §2.7 prévoit n'existe pas dans le formulaire |
| Frontmatter core | ✅ | |
| Bloc `iptc.*` | ✅ | Mapping complet depuis le catalogue Lightroom |
| Dual naming mode | ✅ | `original` ou `sequential` (01.jpg, 02.jpg...) — officialisé en §2.7 v2.1 |
| Tri images | ⚠️ | Ordre de la collection Lightroom (`positionInCollection`, puis `captureTime`), pas alphabétique — divergence intentionnelle préservant l'ordre éditorial |
| Bloc `translations:` | ✅ | Pattern officialisé en §2.7 v2.1 |
| Originaux (§1.9, v2.11) | ✅ | Un master RAW (liste fermée de §1.9, `ExportService.rawExtensions`, insensible à la casse) est converti en JPEG pleine définition, nom de base conservé, `cover` comprise ([exporter-app#98](https://github.com/izo/hyperfocale-exporter-app/pull/98), [#96](https://github.com/izo/hyperfocale-exporter-app/issues/96)). **Non éprouvé sur Mac** ([#99](https://github.com/izo/hyperfocale-exporter-app/issues/99)) |
| Destination projet (Hyperfocus §10) | — | L'exporter n'écrit que des séries (§2.7). Décision du 2026-10-25 : la destination projet n'est pas le sujet de l'exporter ; Hyperfocus §10 est à réconcilier avec la spécialisation du vault en `hyperfocale-photos` ([vault#189](https://github.com/izo/hyperfocale-vault/issues/189)) |

---

## Philosophie

### Principes fondateurs

1. **Markdown-first** — Le contenu est du Markdown standard avec frontmatter YAML. Lisible par un humain, éditable dans n'importe quel éditeur.
2. **Filesystem = API** — La structure de dossiers EST la base de données. Pas de base externe requise, pas de config à maintenir.
3. **Core petit, extensions standardisées** — Le frontmatter obligatoire tient sur 2 champs. Le reste suit le vocabulaire IPTC pour l'interopérabilité.
4. **Portabilité totale** — Un même dossier de contenu fonctionne dans Astro, Next.js, Hugo, Obsidian, ou n'importe quel outil qui lit du Markdown.
5. **La série est l'atome** — L'unité de contenu est la série photo, pas l'image individuelle. Une série = un dossier autonome.

### Ce que cette spec définit

- Le **format de contenu** (fichiers, frontmatter, conventions)
- Les **règles métier** (tri, couverture, pagination)
- Le **contrat d'adaptateur** (ce que chaque plateforme doit implémenter)
- Le **vocabulaire d'extension** (champs IPTC supportés)
- Le **contrat d'ingestion** (couche 4) : comment un corpus édité dans une source — locale, synchronisée ou distante — devient un snapshot validé et publié

### Ce que cette spec NE définit PAS

- L'implémentation des adaptateurs (chacun a sa propre spec)
- Le design visuel (libre, seul le contrat de données est spécifié)
- Le travail à la source — projets, originaux, sélection, chaîne de développement : c'est le périmètre d'**Hyperfocus**, qui découle de cette spec et n'en est pas un profil
- Le workflow d'édition (chaque outil a le sien) — la couche 4 fixe seulement ce qui passe d'une source éditoriale à un snapshot publié, et §1.5.1 ce qu'un éditeur ne doit pas écrire dans une source distante ; ni l'une ni l'autre ne prescrit la façon d'éditer
- L'infrastructure de production (hébergeur, CDN, stockage des médias) — choix du consommateur

---

## Architecture en couches

```
┌─────────────────────────────────────────────┐
│  Couche 1 : FORMAT DE CONTENU               │
│  Frontmatter · Filesystem · Règles métier   │
│  → Universel, aucune dépendance             │
├─────────────────────────────────────────────┤
│  Couche 2 : ADAPTATEURS PLATEFORME          │
│  Astro · Next.js · Hugo · 11ty · Obsidian   │
│  · CMS headless (Strapi, Sanity, Payload)   │
│  → Un par plateforme, implémente le contrat │
├─────────────────────────────────────────────┤
│  Couche 3 : COMPOSANTS UI (optionnel)       │
│  SeriesCard · Gallery · Lightbox · Map      │
│  → Par framework (React, Astro, Web Comp.)  │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│  Couche 4 : INGESTION (outils)              │
│  Source → Snapshot → Diff → Publication     │
│  → En amont : alimente le filesystem lu     │
│    par les couches 1 à 3                    │
└─────────────────────────────────────────────┘
```

La couche 1 est normative. La couche 2 l'est par son **contrat d'adaptateur** (§2.0), que tout adaptateur qui se dit conforme remplit ; ses descriptions par plateforme (§2.1 à §2.7) sont des **exemples**. La couche 3 est une recommandation. La couche 4 est normative **pour les outils d'ingestion** — ce qui transforme un état de source éditoriale en snapshot publié — et ne change rien à ce qu'un lecteur doit faire : un adaptateur lit toujours un corpus matérialisé sur un filesystem.

---

## Couche 1 — Format de contenu

### 1.1 — La série

Une **série** est l'unité de contenu fondamentale. Elle regroupe un ensemble de photographies autour d'un sujet, d'un lieu ou d'un moment cohérent.

Chaque série est **autonome** : toutes ses données (métadonnées + médias) vivent dans un seul dossier. Déplacer le dossier = déplacer la série.

### 1.2 — Structure filesystem

```
<content-root>/series/<slug>/
├── index.md          ← métadonnées + texte libre (frontmatter YAML + body Markdown)
└── media/            ← images de la série (pas de sous-dossiers)
    ├── 01.jpg
    ├── 02.jpg
    └── ...
```

#### Règles

| Élément | Contrainte |
|---------|-----------|
| `<content-root>` | Libre selon la plateforme. Exemples : `src/content/` (Astro), `content/` (Next.js), racine du vault (Obsidian). |
| `<slug>` | Identifiant unique. Minuscules, chiffres, tirets uniquement. Regex : `^[a-z0-9]+(-[a-z0-9]+)*$`. Utilisé dans les URLs. |
| `index.md` | Obligatoire. Contient le frontmatter YAML et le body Markdown. Encodage UTF-8. **`index.mdx` est interdit** *(v2.17)* : seul `index.md` porte une série, et un corpus qui contient un `index.mdx` est invalide (§1.12, `index-mdx-forbidden`, §4.10). Les variantes de langue `index.<lang>.md` relèvent de l'Annexe F. |
| `media/` | Plat — pas de sous-dossiers. Porte les fichiers locaux de la série ; PEUT être absent quand il n'y en a aucun (voir *Absence de `media/`*). |
| Images | Formats alimentant la galerie : `.jpg`, `.jpeg`, `.png`, `.webp`, `.avif`, `.tif`, `.tiff`. Pas de récursion dans `media/`. |
| Originaux | RAW et sidecars de développement (liste fermée, §1.9) : acceptés dans `media/`, ignorés par les lecteurs. |
| Documents joints | Tout autre type de fichier est accepté dans `media/` (PDF, vidéo, audio, archives…) et traité en document joint — voir §1.9. |
| Nommage des images | Libre, mais recommandé : `01.jpg`, `02.jpg`... (padding 2+ chiffres pour l'ordre). |

#### Absence de `media/` *(clarifié v2.13)*

`media/` porte les fichiers locaux d'une série. Une série qui n'en a aucun PEUT ne pas avoir de `media/` :

- images listées par `images:` (§1.5) ou par un manifeste `images.json` (§1.5.1) ;
- médias embarqués seulement (§1.11) ;
- conteneur sans galerie propre (§1.8) ;
- brouillon encore vide.

| Règle | Description |
|-------|-------------|
| Lecture | Un lecteur DOIT traiter un `media/` absent comme un `media/` vide : aucune image, aucun document joint, aucune erreur. |
| Écriture | Un outil qui crée une série PEUT créer un `media/` vide ; aucun lecteur ne DOIT dépendre de sa présence. |
| Raison | Ni Git ni un snapshot (§4.5) ne conservent un dossier vide : seuls des fichiers en sont des entrées. Un `media/` obligatoire mais vide disparaîtrait au premier commit ou à la première ingestion. |

Un corpus PEUT résider sur un filesystem distant ou synchronisé (Dropbox, iCloud Drive, WebDAV…) : sa sémantique reste celle d'un filesystem, indépendamment du transport — la couche 4 définit comment un tel corpus devient un snapshot publiable.

#### Profondeur de rangement *(clarifié v2.6)*

`<content-root>/series/<slug>/` est la forme **canonique**, pas une contrainte de profondeur. Un dossier de série PEUT être rangé à une profondeur arbitraire sous `<content-root>` :

```
<content-root>/archives/music/concerts/2010/<slug>/
├── index.md
└── media/
```

Les segments situés **au-dessus** du dossier de série sont des **sections de rangement** : des dossiers de classement, propres au routage du site, sans existence dans le format.

| Élément | Contrainte |
|---------|-----------|
| Section de rangement | Dossier **sans** `index.md` propre. Profondeur libre. N'est pas un contenu : pas de slug, pas de frontmatter, absente de tout listing. |
| `<slug>` | Le **dernier** segment du chemin. Suit la regex §1.2. Les segments de rangement ne font pas partie du slug. |
| Découverte | Un adaptateur DOIT découvrir les séries par **parcours récursif** de `<content-root>` — tout dossier portant un `index.md` est un contenu. Un listing du seul premier niveau est non conforme. |
| Identité | Le chemin relatif à `<content-root>` est la clé de routage. Le slug seul PEUT ne pas être unique dans le corpus (deux sections peuvent porter un `bretagne-2024/`) ; c'est le chemin qui l'est. |

> **Rangement ≠ imbrication.** La limite d'un seul niveau posée en §1.8 porte sur l'**imbrication** — une série à l'intérieur d'une série — et non sur la profondeur de rangement. Le critère est mécanique et unique : **un dossier parent qui porte un `index.md` est un conteneur** (§1.8, un seul niveau) ; **un dossier parent sans `index.md` est une section de rangement** (profondeur libre).

#### Variante : médias externes

Pour les CMS headless ou les CDN, les images peuvent être des URLs plutôt que des fichiers locaux — dans le frontmatter (§1.5) ou dans un manifeste annexe `images.json` (§1.5.1). Voir [[#1.5 — Mode distant]].

### 1.3 — Frontmatter

Le frontmatter est en YAML, délimité par `---`. Il se divise en deux niveaux : **core** (standardisé) et **extension** (vocabulaire IPTC).

#### Core (2 champs obligatoires, 3 optionnels)

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `title` | `string` | **oui** | Titre de la série |
| `date` | `date` (ISO 8601) | **oui**, sauf collection `dateRequired: false` | Date de la série, utilisée pour le tri. Format : `AAAA-MM-JJ`, éventuellement suivi d'une heure et d'un fuseau (§4.10, règle 8) ; **seule la date calendaire sert au tri**, l'heure est ignorée. |
| `description` | `string` | non | Description courte. Utilisée dans les cards, le SEO, les previews. |
| `cover` | `string` | non | Chemin relatif vers l'image de couverture (`./media/01.jpg`). Si absent : première image par ordre alphabétique. |
| `location` | `string` | non | Lieu associé à la série. Texte libre. |

#### Date requise ou optionnelle *(clarifié v2.13)*

`date` est requise par défaut. Une collection PEUT la rendre optionnelle pour **tous** ses contenus en se déclarant `dateRequired: false` : par son preset (§2.0.1) ou, pour un outil d'ingestion, par sa racine de validation (§4.10). Chaque profil de l'Annexe G indique la valeur de son preset.

| Règle | Description |
|-------|-------------|
| Propriété de la collection | Le caractère requis de `date` se déclare pour une collection, jamais dans le fichier : un contenu ne peut pas s'en dispenser lui-même. |
| Défaut | Sans déclaration, `dateRequired` vaut `true` : un lecteur qui ne connaît pas la collection exige `date`. |
| Série sans date | Dans une collection où `date` est requise, une série sans `date` est invalide (`date-missing`, §4.10). |
| Date présente | Dans une collection `dateRequired: false`, la `date` d'une série, si elle est présente, DOIT rester une date valide (`date-invalid`, §4.10). |
| Section | Une page d'index de section (§1.10) n'est jamais soumise à la règle, quelle que soit la collection : elle se distingue par `type: section`, pas par l'absence de `date`. |
| `title` | Toujours requis, dans toute collection. |

#### Champs de workflow et de nature

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `draft` | `boolean` | non | `true` = série masquée en production. Défaut : `false`. |
| `lang` | `string` | non | Code de langue : deux lettres ISO 639-1, éventuellement suivies d'une région (`fr`, `en`, `pt-BR`) — la forme que la couche 4 reconnaît dans `index.<lang>.md` (§4.10). Pour les sites multilingues. |
| `featured` | `boolean` | non | `true` = série mise en avant (boost dans les listings, sections "à la une"). Défaut : `false`. *Officialisé en v2.1 à partir du pattern observé dans plusieurs implémentations.* |
| `tags` | `string[]` | non | Tags éditoriaux libres. **Distincts** de `iptc.keywords` (qui suit le vocabulaire IPTC normalisé). Voir note ci-dessous. *Officialisé en v2.1.* |
| `type` | `string` | non | Nature du contenu. Défaut : `series`. Seule autre valeur normative : `section` — le fichier est alors une page d'index de section et non une série (§1.10). *Introduit en v2.6.* La valeur `project` est **réservée à Hyperfocus** : elle désigne un dossier de travail qui ne se publie pas. Un lecteur de cette spec ne la lit pas comme une série : l'ingestion la refuse (`type-invalid`, §4.10) et un adaptateur DEVRAIT la refuser explicitement plutôt que d'afficher le dossier. *Réservée en v2.14.* |
| `embeds` | `object[]` | non | Médias hébergés par une plateforme tierce et joués dans la page (Vimeo, YouTube, SoundCloud…). Distincts des documents joints §1.9, qui vivent dans `media/`. Voir §1.11. *Introduit en v2.8.* |

> **Relation `tags` ↔ `iptc.keywords`** : `tags` est un vocabulaire éditorial libre, géré par l'auteur (ex : `featured`, `portrait`, `intimite`). `iptc.keywords` suit le standard IPTC et peut être peuplé automatiquement depuis les métadonnées image (ex : depuis Lightroom). Les deux peuvent coexister. Les adaptateurs DEVRAIENT permettre la recherche par les deux.

#### Extension IPTC

Les champs d'extension suivent le vocabulaire [IPTC Photo Metadata Standard](https://iptc.org/standards/photo-metadata/). Ils sont regroupés sous la clé `iptc` pour éviter les collisions avec le core.

```yaml
iptc:
  # Créateur
  creator: "Mathieu Drouet"
  credit: "Mathieu Drouet / Hyperfocale"
  copyright: "© 2024 Mathieu Drouet. Tous droits réservés."

  # Mots-clés
  keywords:
    - paysage
    - bretagne
    - côte

  # Lieu structuré (complète `location` du core)
  city: "Brest"
  province: "Finistère"
  country: "France"
  country_code: "FR"

  # Technique
  camera: "Fujifilm X-T5"
  lens: "XF 16-55mm f/2.8"
  film: null          # pour l'argentique : "Kodak Portra 400"

  # Éditorial
  headline: "Côtes sauvages du Finistère"
  instructions: "Usage éditorial uniquement"
  source: "Hyperfocale"
```

##### Champs IPTC reconnus

Tous les champs ci-dessous sont optionnels. Un adaptateur DOIT les ignorer sans erreur s'il ne les supporte pas.

| Clé IPTC | Type | Description |
|-----------|------|-------------|
| `creator` | `string` | Nom du photographe / auteur |
| `credit` | `string` | Ligne de crédit (peut différer du creator) |
| `copyright` | `string` | Mention de droits d'auteur |
| `keywords` | `string[]` | Mots-clés / tags libres |
| `city` | `string` | Ville |
| `province` | `string` | Région / État / Province |
| `country` | `string` | Pays |
| `country_code` | `string` | Code pays ISO 3166-1 alpha-2 |
| `camera` | `string` | Boîtier utilisé |
| `lens` | `string` | Objectif utilisé |
| `film` | `string` | Pellicule (argentique) ou profil de simulation |
| `headline` | `string` | Titre court pour syndication (si différent de `title`) |
| `instructions` | `string` | Instructions d'utilisation |
| `source` | `string` | Source originale du contenu |
| `gps` | `object` | Coordonnées GPS : `{ lat: number, lng: number }` |

> **Extensibilité** : des champs hors de ce vocabulaire peuvent être ajoutés sous `iptc.custom.*` ou à la racine du frontmatter. Les adaptateurs DOIVENT les transmettre sans modification (passthrough).

### 1.4 — Exemple complet

```yaml
---
title: "Bretagne 2024"
date: 2024-06-15
description: "Côtes sauvages du Finistère"
cover: "./media/01.jpg"
location: "Finistère, France"
draft: false
lang: fr

iptc:
  creator: "Mathieu Drouet"
  copyright: "© 2024 Mathieu Drouet"
  keywords:
    - paysage
    - bretagne
    - mer
  camera: "Fujifilm X-T5"
  lens: "XF 16-55mm f/2.8"
  city: "Brest"
  province: "Finistère"
  country: "France"
  country_code: "FR"
  gps:
    lat: 48.3905
    lng: -4.4861
---

Texte libre affiché **avant** la galerie de photos.

Paragraphes, liens, emphases — tout le Markdown standard est supporté.
```

### 1.5 — Mode distant

Pour les CMS headless (Strapi, Sanity, Payload...) ou les CDN, les images ne sont pas locales. Le format supporte un champ `images` explicite dans le frontmatter :

```yaml
---
title: "Bretagne 2024"
date: 2024-06-15
description: "Côtes sauvages du Finistère"
cover: "https://cdn.example.com/series/bretagne-2024/01.jpg"

images:
  - url: "https://cdn.example.com/series/bretagne-2024/01.jpg"
    alt: "Phare de la Pointe Saint-Mathieu"
    width: 3000
    height: 2000
  - url: "https://cdn.example.com/series/bretagne-2024/02.jpg"
    alt: "Rochers à marée basse"
    width: 3000
    height: 2000
---
```

#### Règles du mode distant

- Si `images` est présent dans le frontmatter, il a priorité sur le dossier `media/`.
- Si `images` est absent, l'adaptateur scanne `media/` (mode local, comportement par défaut).
- Les deux modes sont mutuellement exclusifs par série. Ne pas mélanger.
- Chaque entrée `images` : `url` (requis), `alt` (optionnel), `width`/`height` (optionnels).

#### 1.5.1 — Manifeste d'images externalisé *(introduit v2.6)*

Le tableau `images:` du frontmatter suppose que la liste des images est **écrite à la main**. Dès qu'elle est **générée** — synchronisation vers un CDN, pipeline d'optimisation, export depuis un catalogue — l'inscrire dans le frontmatter mélange une donnée dérivée à une donnée éditoriale. Deux conséquences pratiques : chaque resynchronisation réécrit `index.md` et pollue son historique Git, et tout éditeur du fichier doit préserver à l'octet un tableau qu'il n'a pas produit.

Le format accepte donc une troisième forme : un **manifeste d'images** dans un fichier annexe `images.json`, à côté de `index.md`.

```
<slug>/
├── index.md
└── images.json
```

**Forme courte** — l'ordre du tableau porte l'ordre de la galerie :

```json
{ "images": [
  "/content/bretagne-2024/media/01.jpg",
  "/content/bretagne-2024/media/02.jpg"
] }
```

**Forme longue** — mêmes clés qu'une entrée `images:` du frontmatter :

```json
{ "images": [
  { "url": "https://cdn.example.com/bretagne-2024/01.jpg", "alt": "Phare de la Pointe Saint-Mathieu", "width": 3000, "height": 2000 }
] }
```

Un adaptateur DOIT accepter les deux formes : une entrée de type chaîne équivaut à `{ "url": <chaîne> }`.

##### Règles du manifeste

| Règle | Description |
|-------|-------------|
| Priorité | `images:` du frontmatter > `images.json` > scan de `media/`. |
| Exclusivité | Les trois modes sont mutuellement exclusifs **par série**. Une série qui porte un `images.json` ne DOIT pas porter aussi un tableau `images:` — le lint DOIT le signaler. |
| Ordre | L'ordre du tableau fait foi. Le tri alphabétique de §1.6 ne s'applique pas : le manifeste est une donnée ordonnée, pas un scan. |
| Résolution des URLs | Une entrée est soit une URL absolue (`https://…`), soit un chemin absolu au site (`/…`), soit un chemin relatif à `index.md` (`./media/01.jpg`). L'adaptateur DOIT supporter les trois. |
| Couverture | `cover` du frontmatter, sinon **première entrée du tableau** (et non la première par ordre alphabétique). |
| Documents joints | Une clé `files` optionnelle complète `images`, avec les mêmes entrées qu'en §1.9 mode distant. |
| Robustesse | JSON illisible, clé `images` absente ou non-tableau : l'adaptateur DOIT se rabattre sur `media/` et DEVRAIT signaler l'anomalie. Jamais d'échec de build. |
| `images.json` | N'est jamais un média ni un document joint — c'est un fichier de métadonnées, au même titre qu'`index.md`. Il reste une donnée dérivée quand le corpus vient d'une source distante : un outil d'ingestion le classe `derived` (§4.3) et PEUT le régénérer depuis le snapshot plutôt que le reprendre de la source. Il NE DOIT PAS en produire pour une série dont l'`index.md` porte `images:` — l'outil violerait lui-même la règle d'exclusivité (`images-conflict`, §4.10). *(v2.12)* |
| Ordre éditorial dans une source distante | Un éditeur qui écrit dans une source distante (§1.2, couche 4) NE DOIT PAS y écrire d'`images.json` : l'ingestion le classe `derived` et PEUT le régénérer, si bien que l'ordre posé par l'auteur serait perdu sans diagnostic. Un ordre de galerie qui n'est pas l'ordre alphabétique des noms s'écrit dans le frontmatter, champ `images:`, avec des entrées relatives à `index.md` (`./media/03.jpg`) ; à défaut, par le nommage des fichiers (§1.6). Un ordre alphabétique ne s'écrit pas : le scan de `media/` le donne. *(v2.12)* |
| Entrées locales de `images:` | La règle *Résolution des URLs* ci-dessus s'applique aussi aux entrées du tableau `images:` du frontmatter (§1.5) : une entrée relative (`./media/01.jpg`) désigne un fichier de `media/`, et un adaptateur DOIT la résoudre comme une entrée relative du manifeste. *(v2.12)* |

##### Pourquoi un fichier annexe plutôt que le frontmatter

| | `images:` (§1.5) | `images.json` (§1.5.1) |
|---|---|---|
| Liste écrite à la main | ✅ naturel | ⚠️ un fichier de plus |
| Liste générée par un outil | ⚠️ réécrit `index.md` à chaque sync | ✅ isole la donnée dérivée |
| Diff Git d'une modification éditoriale | bruité par la liste d'images | propre |
| Contrat d'un éditeur (CMS) | doit préserver un tableau qu'il n'a pas écrit | n'a pas à toucher au fichier |

Les deux formes restent normatives : une série dont la liste d'images est éditoriale a toute raison de la garder dans son frontmatter.

##### Compatibilité

Un adaptateur antérieur à v2.6 ignore le fichier et ne trouve aucune image (`media/` absent) : la série s'affiche sans galerie, sans erreur — c'est le comportement de §1.9 pour un fichier inconnu, appliqué à un dossier vide. Aucun contenu existant n'est cassé. La prise en charge du manifeste est requise pour la conformité v2.6.

### 1.6 — Règles métier

#### Images

| Règle | Description |
|-------|-------------|
| Scan | Mode local : les fichiers situés directement dans `media/` dont l'extension — partie du nom qui suit son dernier point, comparée en minuscules (§4.1) — est `jpg`, `jpeg`, `png`, `webp`, `avif`, `tif` ou `tiff`. Pas de récursion. Les fichiers système (§4.2 : nom commençant par `.`, `Thumbs.db`, `desktop.ini`, `Icon\r`) sont ignorés. Un glob littéral `media/*.{jpg,…}` ne suffit pas : il est sensible à la casse sur la plupart des systèmes de fichiers et manque `DSCF0001.JPG`. |
| Tri | Alphabétique par nom de fichier, défini comme l'ordre canonique de §4.1 : comparaison des octets UTF-8 du nom normalisé en NFC, casse préservée — ni collation locale, ni tri « naturel » des nombres. `10.jpg` précède `9.jpg`, `B.jpg` précède `a.jpg`, `DSCF0002.JPG` précède `dscf0001.jpg`. D'où le nommage recommandé `01.jpg`, `02.jpg`, à padding constant. |
| Couverture | `cover` du frontmatter. Fallback : première image par ordre alphabétique. |
| Alt text | Mode local : l'adaptateur PEUT extraire depuis EXIF/IPTC embarqué ou utiliser le nom de fichier. Mode distant : champ `alt` de chaque image. |

#### Affichage d'une série

| Règle | Description |
|-------|-------------|
| Body | Le contenu Markdown s'affiche **avant** la galerie. |
| Pagination | La galerie est paginée. Taille de page configurable par l'adaptateur (défaut recommandé : 12). |
| Lightbox | La visionneuse plein écran charge **toutes** les images de la série (pas seulement la page courante). |

#### Tri des séries

| Règle | Description |
|-------|-------------|
| Défaut | Date décroissante (les plus récentes en premier). Seule la date calendaire compte ; l'heure éventuelle est ignorée (§1.3). |
| À date égale | Titre croissant, puis chemin (§1.2) : deux comparaisons sur les octets UTF-8 du texte en NFC, comme l'ordre canonique de §4.1. Deux lecteurs rendent ainsi le même listing. |
| Sans date | Dans une collection `dateRequired: false` (§1.3), les séries sans `date` viennent **après** les séries datées, triées par titre puis chemin. |
| Draft | En production, une série `draft: true` n'a ni page de détail, ni entrée de listing, ni entrée de flux (Annexe E) ni de sitemap. En développement elle PEUT être visible. **`draft` masque une série ; il ne protège pas ses fichiers** : l'ingestion (couche 4) copie ceux d'un brouillon comme ceux de toute série. Pour restreindre l'accès, un site utilise un mécanisme à lui (`private`, §4.11). |

#### URLs (pour les plateformes web)

| Route | Description |
|-------|-------------|
| `/<prefix>/` | Liste de toutes les séries |
| `/<prefix>/<chemin>/` | Page d'une série (body + galerie) |
| `/<prefix>/<chemin>/page/<n>/` | Pages suivantes de la galerie, `<n>` ≥ 2 |

Le `<prefix>` est configurable par l'adaptateur (défaut recommandé : `series`). `<chemin>` est le chemin du dossier de la série relatif à la racine de la collection, segments de rangement compris (§1.2) ; il peut donc compter plusieurs segments. Pour la forme canonique `<content-root>/series/<slug>/`, il se réduit au slug.

| Règle | Description |
|-------|-------------|
| Identité *(v2.13)* | Une route se construit sur le chemin, jamais sur le slug seul, qui PEUT ne pas être unique (§1.2) : `concerts/bretagne-2024/` et `festivals/bretagne-2024/` ont deux routes distinctes. |
| Pagination *(v2.13)* | La page 1 est la route de la série elle-même ; `/page/1/` n'est pas une route (un adaptateur PEUT la rediriger). Le segment `page` sépare la pagination des sous-séries (§1.8), dont le slug peut être un nombre : `/<conteneur>/2/` est une sous-série, `/<conteneur>/page/2/` la page 2 de la galerie du conteneur. |
| Segment réservé *(v2.13)* | Un dossier nommé `page` situé directement dans un dossier de série est réservé : une série rangée sous `<série>/page/2/` aurait la route de la page 2 de `<série>`. Le lint DEVRAIT le signaler (Annexe A). |
| Ancienne forme | Jusqu'à la v2.12, la pagination s'écrivait `/<prefix>/<slug>/<page>/`. Un adaptateur qui l'a servie DEVRAIT rediriger une telle URL vers `/<prefix>/<chemin>/page/<n>/` quand aucune série n'occupe ce chemin. |

### 1.7 — Ajout d'une série

Pour n'importe quelle plateforme, la procédure est :

1. Créer le dossier `<content-root>/series/<slug>/`
2. Créer `index.md` avec au minimum `title` et `date`
3. Ajouter les photos dans `media/`
4. La série apparaît automatiquement (après build ou refresh selon la plateforme)

### 1.8 — Séries imbriquées (conteneur)

Une **série conteneur** est une série dont le dossier contient, en plus de son `index.md` et de son `media/`, d'autres dossiers de séries. Elle se comporte comme une série normale (frontmatter + body + galerie propre éventuelle), mais elle **regroupe** un ensemble de sous-séries liées sur un plan éditorial.

Cas d'usage typiques :

- **Festival → performances** : un festival regroupe les séries photo de chaque artiste / set qui y a joué.
- **Évènement multi-temps** : un mariage ou une exposition se décompose en plusieurs moments distincts, chacun méritant sa propre série.
- **Reportage chapitré** : un sujet long déroulé en plusieurs séries indépendantes mais liées.

> **Ne pas confondre avec le rangement (§1.2).** `archives/music/concerts/2010/<slug>/` est une série rangée à quatre segments de profondeur — pas une sous-série de quatrième niveau. Aucun des dossiers traversés ne porte d'`index.md` : ce sont des sections de routage propres au site, invisibles du format. L'imbrication commence quand un dossier de série **porteur d'un `index.md`** en contient un autre ; c'est cette imbrication-là qui est limitée à un niveau.

#### Structure filesystem

```
<content-root>/series/<slug-conteneur>/
├── index.md                  ← série conteneur (frontmatter + body)
├── media/                    ← optionnel : photos propres au conteneur (équipe, lieu vide, etc.)
├── <sous-slug-1>/
│   ├── index.md
│   └── media/
├── <sous-slug-2>/
│   ├── index.md
│   └── media/
└── ...
```

#### Règles

| Élément | Contrainte |
|---------|-----------|
| `index.md` du conteneur | Obligatoire. Mêmes règles que pour une série standard (`title` requis ; `date` selon la collection, §1.3). |
| `media/` du conteneur | **Optionnel**. Si absent, le conteneur n'a pas de galerie propre — il sert uniquement de point d'entrée vers ses sous-séries. |
| Sous-séries | Chacune est une série complète et autonome (au sens des §1.1–1.6). Une sous-série ne peut **pas** elle-même contenir des sous-séries (pas de récursion au-delà d'un niveau). |
| Ce qu'est un conteneur | Un dossier de série dont un **sous-dossier porte un `index.md`**. Un dossier sans `index.md` traversé pour atteindre une série est une section de rangement (§1.2), pas un conteneur — sa profondeur est libre. |
| Rangement dans un conteneur *(v2.15)* | Un dossier de rangement PEUT se trouver **entre** un conteneur et une sous-série (`<conteneur>/lots/<sous-slug>/`). La sous-série garde le conteneur pour parent : le parent d'une série est son plus proche dossier ancêtre **qui est une série** *(précisé en v2.16)* — un dossier de rangement n'en est pas un (il ne porte pas d'`index.md`), et une page de section (§1.10) non plus : elle range, elle ne contient pas, elle est donc transparente. `<conteneur>/rubrique/<sous-slug>/`, où `rubrique/index.md` est `type: section`, donne bien `<sous-slug>` pour sous-série de `<conteneur>`. Son chemin, donc sa route (§1.6), compte les segments de rangement. La limite d'un niveau (`nesting-too-deep`, §4.10) ne les compte pas. |
| `<slug-conteneur>` et `<sous-slug>` | Suivent les mêmes règles de slug que §1.2. Le slug d'une sous-série est local : il n'a pas besoin d'inclure le slug du conteneur. |
| Tri des sous-séries | Date décroissante par défaut (idem listing standard). L'adaptateur PEUT exposer un tri alternatif via un champ `lineup_order: number` dans le frontmatter des sous-séries. |

#### Cover du conteneur

Le champ `cover` du conteneur PEUT pointer vers une image d'une sous-série, en chemin relatif depuis l'index du conteneur :

```yaml
# series/printemps-bourges-2008/index.md
cover: "./the-wombats/media/01.jpg"
```

Cette résolution traverse les sous-dossiers — c'est une dérogation explicite à la règle "pas de récursion dans `media/`" (§1.6), justifiée par le besoin éditorial.

#### URLs

| Route | Description |
|-------|-------------|
| `/<prefix>/<chemin-conteneur>/` | Page du conteneur : body + galerie propre éventuelle + liste des sous-séries (line-up). |
| `/<prefix>/<chemin-conteneur>/<sous-slug>/` | Page d'une sous-série : comportement standard (body + galerie). |

L'adaptateur DOIT générer ces deux niveaux d'URL automatiquement à partir de la structure filesystem. `<chemin-conteneur>` suit la règle *Identité* de §1.6 ; la pagination de l'une ou l'autre page passe par le segment `page` (§1.6), si bien qu'une sous-série nommée `2` ne se confond jamais avec la page 2 du conteneur.

#### Listing global

Lorsque l'adaptateur génère le listing de toutes les séries (`/<prefix>/`), il PEUT au choix :

- **Aplatir** : exposer conteneurs et sous-séries au même niveau (toutes apparaissent dans la grille).
- **Hiérarchiser** : n'afficher que les conteneurs ; les sous-séries n'apparaissent qu'en navigant dans le conteneur.

Ce choix DOIT être explicite dans la config de l'adaptateur. Aplatir est le défaut recommandé pour préserver la rétro-compatibilité avec les implémentations qui n'ont pas la notion de conteneur.

#### Frontmatter de la sous-série — recommandations

Pour faciliter la navigation et le SEO, une sous-série SHOULD :

- Mentionner le conteneur dans son `title` (ex : `"The Wombats - Le Printemps de Bourges 2008"`) ;
- Hériter de tags pertinents du conteneur (lieu, édition) en plus de ses propres tags.

Ces conventions sont **éditoriales**, pas normatives — un adaptateur ne doit pas les imposer.

#### Compatibilité

Les adaptateurs qui ne supportent pas (encore) les séries imbriquées DOIVENT au minimum :

- Ne pas crasher sur la présence de sous-dossiers à côté de `media/`.
- Indexer le conteneur comme une série normale (et ignorer les sous-séries) — perte d'information acceptable en attendant l'implémentation.

### 1.9 — Documents joints (tous types de médias) *(introduit v2.5)*

Une série n'est pas limitée aux images : `media/` accepte **tous les types de documents** (PDF, vidéo, audio, archives, tracés GPX…). Tout fichier de `media/` qui n'est ni une image au sens de §1.2, ni un original (voir *Originaux* ci-dessous), est un **document joint** (*attachment*).

#### Classes de médias

La classification se fait par extension de fichier — partie du nom qui suit son dernier point (§4.1) —, comparée en minuscules : `01.JPG` est une `image`, `Notes.PDF` un `document`. Un adaptateur PEUT affiner (détection MIME) mais DOIT rester prévisible. Les fichiers système (§4.2) ne sont classés dans aucune classe : un lecteur les ignore.

| Classe | Extensions | Rendu recommandé |
|--------|-----------|------------------|
| `image` | `.jpg`, `.jpeg`, `.png`, `.webp`, `.avif`, `.tif`, `.tiff` | Galerie (comportement §1.6, inchangé) |
| `video` | `.mp4`, `.webm`, `.mov`, `.m4v` | Lecteur intégré (`<video>`) dans ou après la galerie |
| `audio` | `.mp3`, `.m4a`, `.ogg`, `.wav`, `.flac` | Lecteur intégré (`<audio>`) après la galerie |
| `document` | `.pdf`, `.epub`, `.txt`, `.md` (hors `index.md`) | Lien de téléchargement / visionneuse |
| `original` | RAW et sidecars — liste fermée, voir *Originaux* | **Aucun** : ni galerie, ni document joint |
| `file` | Tout le reste (`.zip`, `.gpx`, `.svg`…) | Lien de téléchargement |

`original` n'est pas une classe de document joint : c'est la troisième catégorie de fichiers de `media/`, à côté des images et des documents joints. Elle n'apparaît ni dans `attachments:`, ni dans `files:`, ni dans l'interface `Attachment` (§3.2).

#### Règles

| Règle | Description |
|-------|-------------|
| Galerie | Seule la classe `image` alimente la galerie et la lightbox. Les invariants §0 / §1.6 sont inchangés. |
| Pièces jointes | Les autres classes sont exposées en **liste de documents joints**, affichée **après** la galerie. Tri alphabétique par nom de fichier, au sens de §1.6. |
| Couverture | `cover` DOIT référencer une image. Les documents joints ne sont jamais candidats au fallback de couverture. |
| Lecteurs | Un adaptateur PEUT rendre `video` / `audio` en lecteurs intégrés ; à défaut, il les liste en pièces jointes. |
| Robustesse | Un lecteur NE DOIT jamais échouer sur un fichier de type inconnu dans `media/` : il le classe `file` et le liste en document joint. |
| `index.md` | N'est jamais un document joint (c'est le fichier de métadonnées de la série). |

#### Originaux *(introduit v2.11)*

Un dossier de travail — un projet photo géré à la source dès la prise de vue (Hyperfocus, [`spec-hyperfocus.md`](https://github.com/izo/hyperfocus/blob/main/spec-hyperfocus.md)) — porte dans `media/` les fichiers sortis du boîtier et ceux des logiciels de développement : les RAW, et les fichiers annexes (*sidecars*) qui décrivent leurs réglages. Ce sont des **originaux**. Ils ne sont pas faits pour être montrés : un RAW ne s'affiche pas dans un navigateur, un `.xmp` n'est lisible que par un logiciel, et ni l'un ni l'autre n'a vocation à sortir du dossier.

| Catégorie | Extensions (liste fermée, insensible à la casse) |
|-----------|--------------------------------------------------|
| RAW | `.raf` `.cr2` `.cr3` `.nef` `.nrw` `.arw` `.srf` `.sr2` `.orf` `.rw2` `.pef` `.dng` `.3fr` `.iiq` `.x3f` `.srw` `.rwl` `.erf` `.mos` `.mef` `.kdc` `.dcr` |
| Sidecars | `.xmp` `.pp3` `.dop` `.cos` |

| Règle | Description |
|-------|-------------|
| Ignorés | Un lecteur DOIT ignorer les originaux : ils n'alimentent ni la galerie, ni la lightbox, ni la liste des documents joints. |
| Couverture | Un original n'est jamais candidat au fallback de couverture. `cover` DOIT désigner une image : un `cover` vers un original est une erreur (`cover-not-image`, §4.10). |
| `attachments:` | Une entrée `attachments:` qui désigne un original ne désigne pas un document joint : le lint DEVRAIT la signaler (`attachment-not-found`, §4.10) et le rendu l'ignore. |
| Extension | La partie du nom qui suit son **dernier** point, comparée en minuscules (§4.1) : `DSCF0001.RAF` est un original, `DSCF0001.raf.jpg` une image, un fichier nommé `raf` un document joint. |
| Liste fermée | La liste est fermée et versionnée avec la spec, sans renvoi à une source externe : deux implémentations doivent classer un fichier de la même façon. Une extension RAW absente de la liste reste un document joint de classe `file` ; elle s'ajoute par PR contre cette spec. `.dng` est toujours un original, même issu d'une conversion. |
| À plat | Les originaux vivent **à plat** dans `media/`, à côté de leurs images. Un sous-dossier `media/originals/` casserait la règle « pas de récursion » (§1.2) ; il n'est pas une variante admise. |
| Ingestion | Un outil d'ingestion retire les originaux du snapshot publié et en rapporte le nombre et le poids (§4.11). |

##### Compatibilité

Un lecteur antérieur à v2.11 classe les originaux en documents joints de classe `file` et les **propose en téléchargement** après la galerie. Aucun contenu publié n'est cassé — une série publiée n'a aucune raison de porter un RAW — mais un dossier de travail lu par un tel lecteur laisse fuir ses originaux. La règle qui en découle ne dépend d'aucune version d'adaptateur : **un dossier de travail ne se publie jamais directement**, il se publie par export (Hyperfocus, §5). La garde de publication v2.11 (§4.11) protège en plus contre l'erreur, mais seulement quand l'outil d'ingestion la connaît. La prise en charge des originaux est requise pour la conformité v2.11 (§2.0).

#### Métadonnées des documents joints (optionnel)

Le nom de fichier sert de libellé par défaut. Le frontmatter PEUT enrichir les pièces jointes via un bloc `attachments:` :

```yaml
attachments:
  - file: "./media/dossier-de-presse.pdf"
    title: "Dossier de presse"
    description: "Version print, 12 pages"
  - file: "./media/interview.mp3"
    title: "Interview de l'artiste"
```

Les fichiers non listés dans `attachments:` restent des pièces jointes valides (libellé = nom de fichier). Une entrée `attachments:` dont le fichier est absent de `media/` DEVRAIT être signalée par le lint et ignorée au rendu.

#### Mode distant

En mode distant (§1.5), le champ `files` complète `images` pour les pièces jointes hébergées sur un CDN :

```yaml
files:
  - url: "https://cdn.example.com/series/bretagne-2024/dossier.pdf"
    title: "Dossier de presse"
    kind: document
    size: 2400000
```

`url` est requis ; `title`, `kind` (une classe du tableau ci-dessus) et `size` (octets) sont optionnels. Mêmes règles d'exclusivité que §1.5 : si `files` est présent, il a priorité sur les documents joints de `media/`.

#### Compatibilité

Les adaptateurs antérieurs à v2.5 globbent les extensions image et ignorent déjà de facto les autres fichiers : **aucun contenu existant n'est cassé**. Un adaptateur qui n'implémente pas encore les documents joints DOIT au minimum ne pas échouer sur leur présence ; l'exposition des pièces jointes est requise pour la conformité v2.5 (§2.0).

### 1.10 — Page d'index de section *(introduit v2.6)*

Un corpus un peu grand range ses contenus par **sections** : des dossiers de classement (`archives/music/`, `portfolio/`) qui regroupent des séries sans être eux-mêmes des séries. Ces sections ont besoin d'un titre et d'un texte de présentation — c'est ce qui s'affiche en tête de la page de listing.

Le format n'offrait aucune forme pour cela. Un `index.md` posé à cet emplacement était lu comme une série et échouait sur `date` manquante, alors qu'une section n'a pas de date : elle n'est pas un moment, c'est un rangement.

Une **page d'index de section** est un `index.md` qui déclare `type: section`. **Ce n'est pas une série** : pas de galerie, pas de `date`, pas de présence dans les listings de séries.

#### Structure filesystem

```
<content-root>/archives/music/
├── index.md            ← page d'index de section (type: section)
├── concerts/
│   └── <slug>/
│       ├── index.md    ← série
│       └── media/
└── festivals/
    └── <slug>/…
```

#### Frontmatter

```yaml
---
type: section
title: "Musique"
description: "Concerts, festivals et portraits d'artistes depuis 2005."
---

Texte libre affiché **avant** la liste des contenus de la section.
```

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `type` | `string` | **oui** | Valeur `section`. C'est ce champ, et lui seul, qui distingue une page d'index d'une série. |
| `title` | `string` | **oui** | Nom de la section. |
| `date` | — | **non** | Non requise. Une section n'est pas datée. Si présente, elle est ignorée (aucun tri ne s'appuie dessus). |
| `description`, `cover`, `lang`, `draft` | | non | Même sens qu'en §1.3. |

Un `index.md` placé à la racine de `<content-root>` doit lui aussi déclarer `type: section` *(v2.15)* : sans cela il est lu comme une série, le corpus entier devient son conteneur (§1.8), et tout conteneur qu'il porte déjà dépasse la limite d'un niveau (`nesting-too-deep`, §4.10).

Le champ `type` est **réservé** au niveau du core. Son absence vaut `type: series` — la valeur par défaut, et le comportement de tout le contenu antérieur à v2.6.

#### Règles

| Règle | Description |
|-------|-------------|
| Ce n'est pas un contenu de collection | Une page d'index de section est **exclue** des listings de séries, des flux de syndication (Annexe E) et du tri par date (§1.6). Un adaptateur PEUT lui générer une page de section, avec son body en tête et la liste de ses contenus en dessous. |
| Pas de galerie | La section ne porte pas de `media/`. Un `cover` PEUT être renseigné pour l'illustrer dans une navigation ; il pointe alors vers une image d'un contenu de la section, en chemin relatif — même dérogation qu'en §1.8. |
| Discriminant explicite | La distinction série / section se lit **uniquement** dans `type`. Un adaptateur ne DOIT jamais la deviner (par l'absence de `date`, par la présence de sous-dossiers, ou autrement) : dans une collection où `date` est requise (§1.3), une série sans `date` reste une série invalide. |
| Pas de contenu direct | Les contenus d'une section vivent dans ses sous-dossiers. Une section ne « contient » rien au sens de §1.8 : elle ne les regroupe pas éditorialement, elle les range. |
| Imbrication | Une section PEUT contenir d'autres sections. Contrairement à §1.8, la profondeur n'est pas limitée : ce sont des dossiers de classement, pas des contenus. |
| Robustesse | Un adaptateur qui n'implémente pas §1.10 DOIT **ignorer** un `index.md` portant `type: section` — sans erreur, et sans tenter de le valider comme série (validation qui échouerait sur `date`). |

#### Distinction avec la série conteneur (§1.8)

Les deux formes rassemblent des séries ; elles ne sont pas interchangeables.

| | Conteneur §1.8 | Page d'index de section §1.10 |
|---|---|---|
| Nature | Une **série** qui en regroupe d'autres | Un **rangement**, pas un contenu |
| Intention | Éditoriale (un festival, un reportage chapitré) | Structurelle (une rubrique du site) |
| `date` | Selon la collection (§1.3) | Sans objet |
| Galerie propre | Possible (`media/` optionnel) | Non |
| Dans les listings de séries | Oui | Non |
| Profondeur | Un seul niveau d'imbrication | Libre |
| Déclaration | `type` absent (ou `series`) | `type: section` |

#### Compatibilité

Le champ `type` est nouveau : aucun contenu antérieur ne le porte, donc tout contenu antérieur reste une série. Un adaptateur antérieur à v2.6 qui rencontre un `type: section` le traite comme un champ inconnu (passthrough §1.3) et échoue sur `date` — c'est précisément le comportement que cette section corrige, et la raison pour laquelle la prise en charge de §1.10 est requise pour la conformité v2.6 (§2.0).

### 1.11 — Contenus embarqués *(introduit v2.8)*

§1.9 couvre le média qu'on **possède** : un fichier posé dans `media/`. Il ne dit rien du média qu'on **héberge ailleurs** — une vidéo Vimeo ou YouTube, un morceau SoundCloud, un set Bandcamp. Or c'est le cas majoritaire dès qu'il y a de la vidéo : un documentaire de 74 minutes ne vit pas dans un dépôt Git.

Rien dans le format ne savait le décrire. Les trois formes existantes échouent chacune pour une raison différente :

| Forme | Pourquoi elle ne convient pas |
|-------|-------------------------------|
| `attachments:` (§1.9) | Ne référence que des fichiers de `media/`. Une URL Vimeo n'en est pas un. |
| `files:` (§1.9 mode distant) | Accepte une URL, mais n'a ni vignette, ni dimensions, ni identifiant de plateforme. Un adaptateur ne peut qu'en faire un lien. |
| `images:` (§1.5) | Une vidéo n'est pas une image et n'a rien à faire dans la galerie ni dans la lightbox. |

Un **contenu embarqué** (*embed*) est un média hébergé par une plateforme tierce, désigné par son URL et destiné à être joué **dans** la page.

#### Frontmatter

```yaml
embeds:
  - url: "https://vimeo.com/123831041"
    platform: vimeo
    id: "123831041"
    title: "O Jardim da Esperança"
    description: "Documentaire, 74 min"
    poster: "./media/o-jardim-poster.jpg"
    width: 1920
    height: 1080
```

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `url` | `string` | **oui** | URL canonique du média chez son hébergeur. Toujours suffisante pour faire un lien, ce qui garantit la dégradation (voir *Règles*). |
| `platform` | `string` | non | Identifiant d'hébergeur en minuscules. Vocabulaire reconnu : `vimeo` · `youtube` · `dailymotion` · `soundcloud` · `bandcamp` · `spotify`. Toute autre valeur est licite et traitée comme inconnue. |
| `id` | `string` | non | Identifiant du média **chez cet hébergeur** (`123831041`), pas l'URL. |
| `title` | `string` | non | Libellé. À défaut, l'adaptateur PEUT afficher l'URL. |
| `description` | `string` | non | Texte court accompagnant le média. |
| `poster` | `string` | non | Vignette. Même résolution d'URL qu'en §1.5.1 : URL absolue, chemin absolu au site, ou chemin relatif à `index.md`. |
| `width`, `height` | `number` | non | Dimensions natives, en pixels. Servent à réserver le ratio et éviter un décalage de mise en page au chargement. |

#### Règles

| Règle | Description |
|-------|-------------|
| Hors galerie | Un embed n'alimente **ni la galerie ni la lightbox** (§1.6). Il se rend dans sa propre zone, après le body. Un adaptateur PEUT l'intercaler dans la galerie s'il sait le faire proprement, mais ne DOIT jamais le compter comme une image. |
| **Le poster n'est pas une photo de la série** | Une image de `media/` référencée par un `poster` est **exclue** du scan de galerie (§1.6). Sans cette règle, une série de trois vidéos afficherait trois vignettes parasites dans sa galerie. C'est la seule exception au principe « toute image de `media/` alimente la galerie ». |
| Dégradation | `url` seule suffit à un rendu valide : un lien. Construire un lecteur exige `platform` **et** `id` ; à défaut l'adaptateur DOIT se rabattre sur le lien, sans erreur. |
| Construction de l'URL de lecture | Elle appartient à l'adaptateur, pas au format. La spec ne fige aucun gabarit d'iframe : les hébergeurs changent les leurs, une spec ne se réédite pas au même rythme. |
| Restriction de domaine | Beaucoup d'hébergeurs restreignent leurs lecteurs au domaine déclaré. Un embed qui ne se lance pas en local n'est pas nécessairement un défaut de contenu — c'est un point à vérifier en production, pas au build. |
| Couverture | `cover` DOIT rester une image (§1.9). Le `poster` d'un embed est un candidat légitime, à condition d'être désigné explicitement. |
| Série sans image | Une série PEUT ne porter que des embeds : ni `media/` d'images, ni `images:`. Elle s'affiche alors sans galerie, ce qui n'est pas une erreur. |
| Ordre | L'ordre du tableau fait foi. Aucun tri n'est appliqué. |
| Robustesse | Une entrée sans `url` DOIT être ignorée et signalée par le lint. Une `platform` inconnue ne DOIT jamais faire échouer un build. |

#### Frontière avec §1.9

Le critère est **où vit l'octet**, pas ce que le média représente.

| | Document joint §1.9 | Contenu embarqué §1.11 |
|---|---|---|
| Emplacement | Fichier dans `media/`, ou `files:` en mode distant | Chez un hébergeur tiers |
| Désignation | Nom de fichier, ou URL du CDN | URL canonique de la plateforme |
| Une vidéo `.mp4` locale | ✅ classe `video` | ✗ |
| Une vidéo Vimeo | ✗ | ✅ |
| Un `.mp4` sur son propre CDN | ✅ `files:` avec `kind: video` | ✗ — on sert le fichier, pas un lecteur tiers |
| Rendu | `<video>` / `<audio>` natif, ou lien | Lecteur de la plateforme, ou lien |

Un `.mp4` posé sur son propre CDN reste un document joint : on en sert l'octet et on le lit avec une balise native. L'embed commence là où c'est **le lecteur de quelqu'un d'autre** qui rend le média.

#### Compatibilité

`embeds` est un champ nouveau. Aucun contenu antérieur ne le porte, et un adaptateur antérieur à v2.8 le traite en champ inconnu (passthrough §1.3) : il ne rend rien, sans erreur. Aucun contenu existant n'est cassé.

L'exclusion des posters du scan de galerie ne s'applique qu'aux séries portant un `embeds:` — donc à aucun contenu antérieur. Un adaptateur qui ignore §1.11 et rencontre une série d'embeds affichera les posters comme des photos : dégradation visible mais non destructrice, et raison pour laquelle la prise en charge de §1.11 est requise pour la conformité v2.8 (§2.0).

### 1.12 — Modèle de confiance *(introduit v2.17)*

Le contenu d'un corpus partagé n'est pas fiable. Toute personne qui peut déposer un fichier dans le dossier source — contributeur, client, outil de synchronisation, dépôt Git — écrit à la fois des métadonnées (frontmatter), du texte (body), des noms de fichiers et des fichiers de `media/`. Un lecteur, un adaptateur, un outil d'ingestion ou un exporter DOIT traiter tout cela comme des **données** : jamais comme du code à exécuter, ni comme une adresse à suivre sans contrôle. Le format ne définit aucun rôle d'administrateur : ces garde-fous valent pour toute source partagée, et un projet qui édite seul son dossier les applique aussi, par prudence.

Cette section réunit les règles de la couche 1. Celles de la couche 4 sont en §4.0 (providers), §4.10 (diagnostics) et §4.11 (garde).

#### Contenu exécutable et rendu du body

| Règle | Description |
|-------|-------------|
| `index.mdx` interdit | Seul `index.md` (et ses variantes `index.<lang>.md`, Annexe F) porte une série. Un fichier MDX est du JavaScript exécuté au build : qui peut écrire dans le dossier source exécuterait du code dans la chaîne de publication. Un lecteur NE DOIT PAS compiler, évaluer ni importer un fichier `index.mdx` ou `index.<lang>.mdx` d'un corpus ; un outil d'ingestion le refuse (`index-mdx-forbidden`, §4.10, qui vaut pour toute racine de validation) ; un outil qui écrit (CMS, exporter) NE DOIT PAS en créer. La compatibilité MDX par plateforme, admise jusqu'à la v2.16, disparaît : un site MDX lit son contenu en Markdown. |
| Rendu du body assaini | Le HTML produit à partir du body — qu'il vienne du Markdown ou du HTML brut que le moteur laisse passer — DOIT être assaini : le rendu retire les éléments exécutables ou qui changent le document (`script`, `iframe`, `object`, `embed`, `style`, `link`, `meta`, `base`, `form`), tout attribut d'événement (`on…`), et tout attribut `href`, `src`, `srcset`, `action`, `formaction`, `poster` ou `xlink:href` dont la valeur n'a pas une forme admise (*Schémas d'URL*, ci-dessous). Un adaptateur PEUT à la place désactiver le HTML brut de son moteur Markdown, pourvu que les liens et images qui restent suivent la même liste blanche. |
| Données injectées | Toute donnée de contenu insérée dans une page — titre, description, JSON-LD, attribut, champ de profil — est échappée selon son contexte : une chaîne qui contient `</script>` ne ferme pas le script qui la porte. |

#### Schémas d'URL

| Règle | Description |
|-------|-------------|
| Champs concernés | `cover` ; `images[].url`, `files[].url`, `embeds[].url` et `embeds[].poster`, dans le frontmatter comme dans le manifeste `images.json` (§1.5.1) ; les champs d'URL des profils de l'Annexe G (`url`, `repository`, `archive_url`, `streaming.*`…) ; les liens et images du body Markdown. |
| Formes admises | (1) un chemin relatif à `index.md` (`./media/01.jpg`) ; (2) un chemin absolu au site (`/content/…`), avec un seul `/` initial ; (3) une URL de schéma `http:` ou `https:`, schéma comparé sans tenir compte de la casse. Pour `embeds[].url`, seule la forme (3) est admise : un embed désigne un hébergeur. |
| Refusé | Tout autre schéma — `javascript:`, `data:`, `file:`, `blob:`, `vbscript:`, `ftp:`… —, une URL à schéma implicite (`//hôte/…`), un chemin absolu qui contient `\`, une valeur qui contient un caractère de contrôle (U+0000–U+001F, U+007F) ou commence par une espace : les navigateurs ignorent ces caractères avant de lire le schéma (`java<tab>script:` est un `javascript:`). `mailto:` n'est admis que dans un champ qui désigne un contact et se rend en lien ; aucun champ du format n'en désigne un, et jamais une ressource (`src`). |
| Au rendu | Un lecteur DOIT refuser une valeur refusée avant de produire un `href` ou un `src` : il omet le lien ou la ressource, sans échec de build, et DEVRAIT le signaler. La valeur se lit telle qu'écrite : elle n'est jamais décodée ni normalisée avant le contrôle. |
| À l'ingestion | `url-scheme-forbidden` (§4.10, règle 15) pour les champs du core. Les champs des profils ne sont pas validés par la couche 4 (Annexe G) : le contrôle du rendu les couvre. |
| Identifiants d'embed | Un adaptateur ne construit l'URL de lecture d'un embed que si `platform` appartient au vocabulaire reconnu (§1.11) et si `id` vérifie `^[A-Za-z0-9_-]{1,128}$` ; sinon il se rabat sur le lien (`url`, déjà contrôlée). L'`id` ne se concatène jamais à une URL sans ce contrôle. |

#### Confidentialité

| Règle | Description |
|-------|-------------|
| `private` est un booléen | `private` n'est pas un champ du core : c'est une extension de site (§4.11). Un site qui l'emploie l'écrit `private: true`, un **booléen** YAML. Toute autre valeur — la chaîne `"true"`, `yes`, `1`, `oui`, une liste — est une **erreur**, jamais un contenu public : un site qui emploie `private` DOIT refuser de publier le contenu, ou le traiter comme privé, quand `private` est présent (non `null`) sans être un booléen. Un outil d'ingestion la signale (`private-type-invalid`, §4.10) et ne publie pas sans intervention. `yes` est une chaîne en YAML 1.2 et un booléen en YAML 1.1 : deux lecteurs divergeraient, d'où le refus plutôt que l'interprétation. |
| Clés quasi-homonymes | Une clé qui ne diffère de `private` que par la casse, des espaces, `_` ou `-` (`Private`, `PRIVATE`, `private_`) n'est pas `private` : elle ne protège rien. L'ingestion la signale (`private-key-suspect`, §4.10). |

---

## Couche 2 — Adaptateurs plateforme

Un adaptateur est une implémentation du format Hyperfocale pour une plateforme donnée. Chaque adaptateur DOIT respecter le contrat ci-dessous.

### 2.0 — Contrat d'adaptateur

Tout adaptateur Hyperfocale **DOIT** :

| Obligation | Description |
|-----------|-------------|
| Lire le frontmatter core | Parser `title`, `date`, `description`, `cover`, `location`, `draft`, `lang` |
| Exiger les champs requis | `title` toujours ; `date` sauf dans une collection déclarée `dateRequired: false` ; jamais pour une page d'index de section (§1.3, §1.10) |
| Ignorer les champs inconnus | Ne jamais échouer sur un champ frontmatter non reconnu |
| Transmettre les extensions | Rendre accessible `iptc.*` et tout champ supplémentaire aux templates/composants |
| Scanner `media/` | Mode local : lister les images de `media/` — extension comparée en minuscules, fichiers système (§4.2) ignorés — et les trier dans l'ordre alphabétique de §1.6 |
| Exposer les documents joints | Lister les documents joints de `media/` — ni images ni originaux — (tri alphabétique) et les rendre accessibles aux templates (§1.9) — *conformité v2.5* |
| Ignorer les originaux | Ne jamais placer un original (§1.9 — RAW, sidecars) dans la galerie, la lightbox, les documents joints ni le fallback de couverture — *conformité v2.11* |
| Supporter le mode distant | Si `images` est présent, l'utiliser à la place de `media/` ; si `files` est présent, l'utiliser pour les pièces jointes |
| Distinguer les sections | Ne pas valider comme série un `index.md` portant `type: section` ; l'exclure des listings de séries (§1.10) — *conformité v2.6* |
| Supporter le manifeste d'images | Si un `images.json` accompagne `index.md`, l'utiliser à la place de `media/` en respectant l'ordre du tableau (§1.5.1) — *conformité v2.6* |
| Exposer les contenus embarqués | Rendre `embeds` accessible aux templates, et **exclure les images désignées par un `poster` du scan de galerie** (§1.11) — *conformité v2.8* |
| Respecter `draft` | Exclure les drafts en production |
| Trier par date desc | Listing par défaut : date décroissante |
| Exposer le body | Rendre le Markdown du body en HTML **assaini** (§1.12) — *conformité v2.17* |
| Refuser le contenu exécutable | Ne jamais compiler, évaluer ni importer un fichier `index.mdx` d'un corpus (§1.12) — *conformité v2.17* |

Tout adaptateur **PEUT** :

| Extension | Description |
|-----------|-------------|
| Optimiser les images | Générer des formats modernes, srcset, dimensions |
| Ajouter des routes | Pages de tags, flux RSS, sitemap, API JSON |
| Enrichir les métadonnées | Extraire EXIF des images, géocoder, etc. |
| Proposer un thème | CSS, tokens, design system — hors périmètre de la spec |
| Exposer des presets de domaine | Voir §2.0.1 ci-dessous |

### 2.0.1 — Presets de domaine *(officialisé v2.1)*

Un adaptateur PEUT exposer un système de **presets de domaine** permettant de pré-configurer la collection selon l'usage (galerie photo classique, recettes, fiches produits, line-up de festival, etc.) sans dupliquer la configuration manuellement.

Exemple (plugin Astro `@regrets/hyperfocale`) :

```ts
hyperfocale({ preset: 'series' })   // séries photo, prefix /series, date requise
hyperfocale({ preset: 'recipe' })   // recettes, prefix /recipes, date optionnelle
hyperfocale({ preset: 'catalog' })  // fiches produit, prefix /catalog, date optionnelle
```

#### Contraintes des presets

Tout preset DOIT respecter le contrat d'adaptateur §2.0. Un preset PEUT modifier :
- Le nom de la collection (`collectionName`)
- Le préfixe d'URL (`prefix`)
- La conditionnalité de `date`, par `dateRequired: false` (recettes intemporelles…) — seul champ du core qu'un preset peut rendre optionnel. Ce n'est pas la suppression d'une obligation : le contrat (§0, §2.0) exige `date` sauf dans une collection ainsi déclarée (§1.3)
- Le vocabulaire d'extension affiché à l'utilisateur

Un preset NE DOIT PAS :
- Modifier le slug regex
- Supprimer les obligations du contrat d'adaptateur
- Rendre `title` optionnel
- Renommer les champs core (`title`, `date`, `description`, `cover`, etc.)

Dix presets non-photo sont standardisés en **Annexe G — Profils de contenu** (`event`, `recipe`, `app`, `book`, `place`, `screen`, `portfolio`, `music`, `catalog`, `press`), comme extensions hors noyau. Le profil Projet (`project`), qui y figurait jusqu'à la v2.13 sous G.12, relève d'**Hyperfocus**. L'ajout de nouveaux profils se discute par PR contre cette spec, en étendant cette annexe.

### 2.1 — Adaptateur Astro

**Dépôt** : https://github.com/izo/hyperfocale-astro-plugins
**Forme** : intégration Astro (package npm, privé ou public).

#### Content Collection

L'intégration enregistre la collection de son preset (par défaut `series`). L'utilisateur n'écrit pas le schéma à la main : le plugin l'exporte sous le nom `baseSeriesSchema`, qui se prolonge avec `.extend()`. Le bloc ci-dessous en donne une **forme simplifiée, à titre d'illustration** — le schéma réel est celui du plugin, et il couvre aussi `type`, `tags`, `embeds`, `attachments` et `images.json`.

```ts
// Schéma Zod simplifié (zod 4) — illustratif
const series = defineCollection({
  type: 'content',
  schema: ({ image }) =>
    z.looseObject({
      title: z.string(),
      date: z.coerce.date(),
      description: z.string().optional(),
      cover: image().optional(),
      location: z.string().optional(),
      draft: z.boolean().default(false),
      lang: z.string().optional(),
      iptc: z.looseObject({}).optional(),
      images: z.array(z.object({
        // http(s), chemin absolu au site ou relatif à index.md (§1.5.1, §1.12) :
        // z.url() seul accepterait javascript:, data: et file:
        url: z.string().refine(isAllowedUrl),
        alt: z.string().optional(),
        width: z.number().optional(),
        height: z.number().optional(),
      })).optional(),
    }),
});
```

#### Configuration

```ts
// astro.config.mjs
import hyperfocale from '@regrets/hyperfocale';

export default defineConfig({
  integrations: [
    hyperfocale({
      prefix: '/series',     // préfixe des routes
      pageSize: 12,          // images par page
      theme: 'auto',         // 'light' | 'dark' | 'auto'
    }),
  ],
});
```

#### Routes injectées

| Route | Page |
|-------|------|
| `/<prefix>/` | Liste des séries |
| `/<prefix>/[...chemin]/` | Détail série + galerie — `chemin` est un paramètre rest, qui peut compter plusieurs segments (§1.6) |
| `/<prefix>/[...chemin]/page/[n]/` | Pagination galerie, `n` ≥ 2 |

#### Composants exposés

```ts
import {
  SeriesCard,
  SeriesList,
  SeriesGallery,
  SeriesLightbox,
} from '@regrets/hyperfocale/components';
```

#### Helpers

```ts
import {
  getSeriesList,
  getSeriesBySlug,
  getSeriesImages,
  paginateImages,
} from '@regrets/hyperfocale/helpers';
```

### 2.2 — Adaptateur Next.js

**Forme** : package npm avec utilitaires + composants React.

#### Chargement du contenu

L'adaptateur fournit un loader compatible avec les bibliothèques de contenu Next.js (contentlayer, velite, ou loader custom) :

```ts
// next.config.ts ou lib/content.ts
import { createHyperfocaleLoader } from 'hyperfocale/next';

const loader = createHyperfocaleLoader({
  contentDir: './content/series',
  pageSize: 12,
});
```

#### Routes (App Router)

Structure de pages recommandée :

```
app/
├── series/
│   ├── page.tsx              ← liste des séries
│   └── [...chemin]/
│       └── page.tsx          ← détail série ; un chemin terminé par page/<n> est la page <n> de sa galerie
```

Un segment catch-all est forcément le dernier : la pagination se reconnaît donc dans la page elle-même, à ses deux derniers segments (`page`, `<n>`), selon §1.6.

#### Composants React

```tsx
import {
  SeriesCard,
  SeriesList,
  SeriesGallery,
  SeriesLightbox,
} from 'hyperfocale/react';
```

Les composants sont des Server Components par défaut, sauf `SeriesLightbox` (Client Component — interaction clavier/souris).

#### Optimisation d'images

L'adaptateur utilise `next/image` pour l'optimisation automatique. En mode local, les images sont servies depuis `public/` ou via un loader custom.

### 2.3 — Adaptateur Hugo

**Forme** : module Hugo ou thème.

#### Mapping

| Concept Hyperfocale | Équivalent Hugo |
|-------------------|-----------------|
| Série | Page Bundle (`content/series/<slug>/`) |
| `index.md` | `index.md` (leaf bundle) |
| `media/` | Page resources (images dans le bundle) |
| Frontmatter core | Front matter Hugo standard |
| `iptc.*` | Front matter custom (passthrough via `.Params.iptc`) |

#### Structure Hugo

```
content/series/<slug>/
├── index.md
└── media/
    ├── 01.jpg
    └── ...
```

Hugo reconnaît nativement les leaf bundles. Les images dans `media/` sont des Page Resources accessibles via `.Resources`.

#### Shortcodes

L'adaptateur fournit des shortcodes optionnels :

```
{{</* series-gallery */>}}
{{</* series-lightbox */>}}
```

### 2.4 — Adaptateur 11ty (Eleventy)

**Forme** : plugin Eleventy.

#### Mapping

| Concept Hyperfocale | Équivalent 11ty |
|-------------------|-----------------|
| Série | Fichier dans une collection |
| `media/` | Fichiers copiés via passthrough |
| Frontmatter core | Front matter standard (data cascade) |

#### Configuration

```js
// .eleventy.js
const hyperfocale = require('hyperfocale/eleventy');

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(hyperfocale, {
    prefix: '/series',
    pageSize: 12,
  });
};
```

### 2.5 — Adaptateur Obsidian

L'adaptateur Obsidian opère sur **trois niveaux** progressifs, du plus simple au plus riche.

#### Niveau 1 — Source d'édition

Le format Hyperfocale est nativement compatible Obsidian sans plugin. Un vault contenant des séries est directement éditable :

```
vault/
├── series/
│   ├── bretagne-2024/
│   │   ├── index.md      ← visible et éditable dans Obsidian
│   │   └── media/
│   │       ├── 01.jpg    ← preview inline via ![[01.jpg]]
│   │       └── 02.jpg
│   └── ...
```

**Compatibilité native** :
- Le frontmatter YAML est lu par Obsidian (propriétés)
- Les images dans `media/` sont prévisualisées via `![[media/01.jpg]]` ou `![](media/01.jpg)`
- Le body Markdown est rendu normalement
- Les champs `iptc.keywords` apparaissent dans les propriétés

**Limitation** : pas de galerie, pas de navigation inter-séries.

#### Niveau 2 — Consultation enrichie

Avec les plugins communautaires Dataview et/ou DB Folder, le vault devient navigable :

**Dataview — Liste des séries** (dans une note `Series.md`) :

````markdown
```dataview
TABLE date, location, iptc.camera AS "Camera"
FROM "series"
WHERE !draft
SORT date DESC
```
````

**Dataview — Par mot-clé** :

````markdown
```dataview
TABLE date, location
FROM "series"
WHERE contains(iptc.keywords, "bretagne")
SORT date DESC
```
````

**Tags Obsidian** : ajouter un champ `tags` en miroir de `iptc.keywords` pour activer la navigation par tags native d'Obsidian. Le plugin de niveau 3 maintient cette synchronisation automatiquement.

#### Niveau 3 — Spec plugin Obsidian dédié

Un plugin Obsidian qui comprend nativement le format Hyperfocale. Il s'installe via le gestionnaire de plugins communautaires d'Obsidian.

##### Périmètre

Le plugin opère uniquement sur les dossiers qui respectent le format Hyperfocale (présence d'un `index.md` portant `title`, §0) — ni `media/` (§1.2, *Absence de `media/`*) ni `date` (§1.3) n'en sont des conditions. Il ignore silencieusement les autres fichiers du vault.

##### Vues

| Vue | Déclencheur | Description |
|-----|-------------|-------------|
| **Galerie de série** | Ouverture d'un `index.md` de série | Remplace (ou complète) la vue Markdown par une grille d'images tirées de `media/` + le body rendu au-dessus |
| **Lightbox** | Clic sur une image de la galerie | Visionneuse plein écran, navigation ← → clavier et swipe, affichage optionnel des métadonnées IPTC |
| **Panneau Séries** | Commande ou icône barre latérale | Liste toutes les séries du vault (ou d'un dossier configuré) avec cover, titre, date, nombre d'images |
| **Carte** | Commande ou onglet dans le panneau Séries | Affiche les séries géolocalisées sur une carte (OpenStreetMap), uniquement celles ayant `iptc.gps` |

##### Commandes (Command Palette)

| ID | Libellé | Action |
|----|---------|--------|
| `hyperfocale:open-panel` | Ouvrir le panneau Séries | Affiche le panneau latéral |
| `hyperfocale:open-map` | Carte des séries | Ouvre la vue carte |
| `hyperfocale:new-series` | Nouvelle série | Crée un dossier `<slug>/index.md` + `media/` avec template |
| `hyperfocale:validate` | Valider la série courante | Vérifie la conformité du frontmatter, signale les erreurs |
| `hyperfocale:sync-tags` | Synchroniser les tags | Copie `iptc.keywords` → `tags` dans le frontmatter de toutes les séries |
| `hyperfocale:set-cover` | Définir comme couverture | Sur une image ouverte : inscrit son chemin dans `cover` du `index.md` parent |

##### Paramètres du plugin

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| `seriesRoot` | string | `series` | Dossier racine des séries dans le vault |
| `pageSize` | number | 12 | Images par page dans la galerie |
| `autoSyncTags` | boolean | false | Synchroniser `iptc.keywords` → `tags` à chaque modification |
| `showExifOnLightbox` | boolean | true | Afficher les métadonnées IPTC dans la lightbox |
| `mapProvider` | string | `osm` | Fournisseur de carte : `osm` (OpenStreetMap) ou `maptiler` |

##### Comportements

| Comportement | Description |
|-------------|-------------|
| **Détection automatique** | À l'ouverture d'un fichier `index.md` situé n'importe où sous `<seriesRoot>` (découverte récursive, §1.2), basculer en vue galerie |
| **Fallback** | Si `media/` est vide ou manquant, afficher le body Markdown normalement sans erreur |
| **Tri des images** | Identique à la spec core : alphabétique par nom de fichier |
| **Couverture** | `cover` du frontmatter ou première image alphabétique |
| **Drafts** | Les séries `draft: true` sont affichées normalement dans le vault (l'auteur les voit), mais marquées visuellement (badge ou opacité réduite) |
| **Nouveau template** | La commande `hyperfocale:new-series` génère un `index.md` avec tous les champs core commentés |

##### Interopérabilité plugins tiers

Le plugin expose une API JavaScript accessible aux autres plugins Obsidian (Templater, QuickAdd, Dataview) :

```ts
// Accessible via app.plugins.plugins['hyperfocale'].api
interface HyperfocaleAPI {
  /** Retourne toutes les séries du vault */
  getSeries(): Promise<Series[]>;
  /** Retourne une série par slug */
  getSeriesBySlug(slug: string): Promise<Series | null>;
  /** Retourne les images d'une série */
  getImages(slug: string): Promise<string[]>; // chemins relatifs
}
```

##### Non-périmètre du plugin

- Pas de sync avec Astro/Next.js (outil CLI séparé)
- Pas d'upload vers un CDN
- Pas de gestion des permissions / partage
- Pas d'édition des métadonnées EXIF des fichiers images (lecture seule)

### 2.6 — Adaptateur CMS headless

Pour Strapi, Sanity, Payload, ou tout CMS headless : l'adaptateur est un **bridge bidirectionnel** entre le format fichier et l'API du CMS.

#### Direction : fichier → CMS (import)

Un script CLI ou un webhook qui :
1. Lit les dossiers de séries
2. Parse le frontmatter
3. Upload les images vers le media library du CMS
4. Crée les entrées dans le content type correspondant

#### Direction : CMS → fichier (export)

Un script CLI ou un webhook qui :
1. Requête l'API du CMS
2. Génère les dossiers `<slug>/index.md` + télécharge les médias dans `media/`
3. Ou génère du frontmatter en [mode distant](#15--mode-distant) avec les URLs du CDN

#### Content Type générique

Le mapping vers un CMS suit ce schéma :

| Champ Hyperfocale | Content Type CMS |
|-------------------|-----------------|
| `title` | Champ texte (requis) |
| `date` | Champ date (requis) |
| `description` | Champ texte long |
| `cover` | Champ média (relation) |
| `location` | Champ texte |
| `draft` | Champ booléen |
| `iptc.*` | Composant / JSON field |
| `media/*` | Collection de médias (relation multiple) |
| Body Markdown | Champ rich text ou Markdown |

### 2.7 — Exporter Lightroom (source de contenu)

**Dépôt** : https://github.com/izo/hyperfocale-exporter-app

> **Hyperfocus (2.14).** L'exporter a deux destinations. Celle décrite ici — un dossier de **série** — reste le contrat de cette section : c'est un export vers Hyperfocale, et elle ne déménage pas. Sa seconde destination, qui dépose des images dans le `media/` d'un **projet**, relève d'Hyperfocus ([§10](https://github.com/izo/hyperfocus/blob/main/spec-hyperfocus.md)) ; aucun exporter n'est tenu de l'offrir pour rester conforme à cette spec.

L'exporter Lightroom est un **plugin d'export Adobe Lightroom Classic / Lightroom** qui génère du contenu au format Hyperfocale directement depuis le catalogue photo.

Il est traité comme un adaptateur "source" : il produit le format, il ne le consomme pas.

#### Rôle

```
Adobe Lightroom (catalogue)
    ↓  sélection de photos + export
Format Hyperfocale
    ├── <slug>/index.md   (frontmatter peuplé depuis les métadonnées LR)
    └── <slug>/media/     (fichiers exportés)
    ↓
Astro / Next.js / Obsidian / ...
```

#### Mapping Lightroom → frontmatter Hyperfocale

L'exporter lit les métadonnées du catalogue Lightroom et les inscrit dans le frontmatter. La correspondance est la suivante :

| Source Lightroom | Champ Hyperfocale | Notes |
|-----------------|-------------------|-------|
| Titre de la collection / album | `title` | Saisie manuelle si absent |
| Date de la photo la plus récente du lot | `date` | Ou saisie manuelle |
| Légende de la collection | `description` | |
| Première photo exportée | `cover` | Configurable |
| Lieu (texte libre) | `location` | Assemblage `City, Country` si absent |
| Creator | `iptc.creator` | Depuis les paramètres IPTC de LR |
| Copyright | `iptc.copyright` | Depuis les paramètres IPTC de LR |
| Mots-clés LR | `iptc.keywords` | Hiérarchie aplatie |
| Appareil (EXIF) | `iptc.camera` | Lu depuis les EXIF de la première photo |
| Objectif (EXIF) | `iptc.lens` | Lu depuis les EXIF de la première photo |
| Ville | `iptc.city` | Métadonnée IPTC LR |
| Province / État | `iptc.province` | Métadonnée IPTC LR |
| Pays | `iptc.country` | Métadonnée IPTC LR |
| Code pays | `iptc.country_code` | Métadonnée IPTC LR |
| GPS (EXIF) | `iptc.gps` | Centroïde si plusieurs positions |

#### Paramètres de l'exporter

| Paramètre | Description |
|-----------|-------------|
| **Dossier de destination** | Chemin vers `<content-root>/series/` du projet cible |
| **Slug** | Manuel ou généré depuis le titre (slugify) |
| **Format d'export image** | JPEG / WebP / AVIF — qualité et taille configurables |
| **Nommage des fichiers** | `01.jpg`, `02.jpg`... (padding configurable) ou nom original |
| **Mode draft** | Exporter avec `draft: true` pour révision avant publication |
| **Écraser** | Si la série existe déjà : écraser, fusionner, ou erreur |
| **Body** | Texte libre injecté dans le body Markdown |

#### Dual naming mode des images *(officialisé v2.1)*

L'exporter Lightroom supporte deux modes de nommage des fichiers exportés dans `media/`, au choix de l'utilisateur à l'export :

| Mode | Description | Tri |
|------|-------------|-----|
| `sequential` | Renomme en `01.jpg`, `02.jpg`... avec padding configurable (2 chiffres < 100 photos, 3 chiffres ≥ 100) | Ordre Lightroom (`customSortOrder, captureTime`) |
| `original` | Préserve le nom de fichier original du catalogue Lightroom | Ordre alphabétique (conforme à la règle générale §1.6) |

Les deux modes produisent un format valide. Le mode `sequential` est recommandé pour préserver l'ordre éditorial défini dans Lightroom.

#### Bloc `translations:` — i18n par série *(officialisé v2.1)*

Pour les séries multilingues sans recourir à des collections séparées (cf. Annexe F stratégie 3), l'exporter peut générer un bloc `translations:` dans le frontmatter :

```yaml
---
title: "Bretagne 2024"
date: 2024-06-15
description: "Côtes sauvages du Finistère"
lang: fr

translations:
  en:
    title: "Brittany 2024"
    description: "Wild coasts of Finistère"
  es:
    title: "Bretaña 2024"
    description: "Costas salvajes de Finistère"
  ja:
    title: "ブルターニュ 2024"
    description: "フィニステールの野生の海岸"
---
```

##### Règles du bloc `translations:`

| Règle | Description |
|-------|-------------|
| Structure | `translations.<lang>.<champ>` — `<lang>` est un code de langue, de la même forme que `lang` (§1.3 : `en`, `pt-BR`) |
| Champs traduisibles | `title`, `description`, `location` (les autres champs ne sont pas traduisibles) |
| Fallback | Si la locale demandée n'est pas dans `translations`, l'adaptateur DOIT renvoyer la version dans le champ racine (canonique selon `lang`) |
| `keywords` | Les `tags` éditoriaux ne sont **pas** dans `translations` (vocabulaire stable). Les `iptc.keywords` peuvent l'être si pertinent (sous `iptc.keywords` racine, plus rarement traduits) |

##### Implémentation des adaptateurs

Tout adaptateur **PEUT** lire `translations` — c'est une extension optionnelle. Un adaptateur qui ne l'implémente pas DOIT ignorer le bloc sans erreur (passthrough). La stratégie 3 de l'Annexe F (collections séparées par locale) est une alternative équivalente — choisir l'une OU l'autre par projet, pas les deux.

#### Fichiers générés

Pour une collection LR "Bretagne 2024" exportée vers `./content/series/` :

```
content/series/bretagne-2024/
├── index.md          ← généré automatiquement
└── media/
    ├── 01.jpg
    ├── 02.jpg
    └── ...
```

`index.md` généré :

```yaml
---
title: "Bretagne 2024"
date: 2024-06-15
description: ""
cover: "./media/01.jpg"
location: "Brest, France"
draft: false

iptc:
  creator: "Mathieu Drouet"
  copyright: "© 2024 Mathieu Drouet"
  keywords: [paysage, bretagne, mer, côte]
  camera: "Fujifilm X-T5"
  lens: "XF 16-55mm f/2.8"
  city: "Brest"
  province: "Finistère"
  country: "France"
  country_code: "FR"
  gps: { lat: 48.39, lng: -4.49 }
---
```

#### Workflow type

1. Dans Lightroom, sélectionner les photos d'une série
2. File → Export → Hyperfocale
3. Choisir le dossier destination et le slug
4. Ajuster les options (qualité, draft, body)
5. Exporter → les fichiers apparaissent dans `content/series/<slug>/`
6. Dans Astro/Next/Obsidian : la série est immédiatement disponible

---

## Couche 3 — Composants UI

La couche UI est **optionnelle et par framework**. Elle définit un vocabulaire de composants commun que chaque implémentation peut adapter.

### 3.1 — Vocabulaire de composants

| Composant | Rôle | Props minimales |
|-----------|------|-----------------|
| `SeriesCard` | Card d'aperçu : cover, titre, date, description | `series: Series` |
| `SeriesList` | Grille de cards | `series: Series[]`, `columns?: number` |
| `SeriesGallery` | Galerie d'images paginée | `images: Image[]`, `page: number`, `totalPages: number`, `baseUrl: string` |
| `SeriesLightbox` | Visionneuse plein écran | `images: Image[]` |
| `SeriesMap` | Carte des séries géolocalisées | `series: Series[]` (filtrées sur celles ayant `iptc.gps`) |
| `SeriesFilter` | Filtrage par keywords, date, lieu | `series: Series[]`, `filters: FilterConfig` |
| `SeriesAttachments` | Liste des documents joints (téléchargements, lecteurs audio/vidéo) | `attachments: Attachment[]` |
| `SeriesEmbeds` | Contenus embarqués (lecteurs de plateforme, ou liens) | `embeds: Embed[]` |

### 3.2 — Types de données partagés

Ces types sont la **lingua franca** entre les adaptateurs et les composants :

```ts
/** Série photo — objet retourné par les helpers */
interface Series {
  path: string;           // chemin relatif à la racine de la collection : identité et clé de routage (§1.2, §1.6)
  slug: string;           // dernier segment du chemin
  title: string;
  date?: Date;            // absente seulement dans une collection `dateRequired: false` (§1.3)
  description?: string;
  cover?: Image;          // absente d'une série sans image, ex. embeds seuls (§1.11)
  location?: string;
  draft: boolean;
  lang?: string;
  tags?: string[];
  featured?: boolean;
  body: string;           // Markdown brut ou HTML rendu, selon la plateforme
  iptc: IPTCMetadata;
  images: Image[];        // toutes les images de la série
  attachments: Attachment[]; // documents joints : ni images ni originaux (§1.9)
  embeds: Embed[];        // contenus embarqués (§1.11)
}

/** Image — mode local ou distant */
interface Image {
  src: string;            // chemin relatif (local) ou URL (distant)
  alt?: string;
  width?: number;
  height?: number;
}

/** Document joint — mode local ou distant (§1.9) */
interface Attachment {
  src: string;            // chemin relatif (local) ou URL (distant)
  kind: 'video' | 'audio' | 'document' | 'file';
  title?: string;         // libellé (frontmatter attachments: ou nom de fichier)
  description?: string;
  size?: number;          // octets, si connu
}

/** Contenu embarqué — hébergé par une plateforme tierce (§1.11) */
interface Embed {
  url: string;            // seul champ requis : suffit à faire un lien
  platform?: string;      // vocabulaire ouvert (vimeo, youtube, …)
  id?: string;            // identifiant chez l'hébergeur
  title?: string;
  description?: string;
  poster?: string;        // vignette ; exclue de la galerie (§1.11)
  width?: number;
  height?: number;
}

/** Métadonnées IPTC */
interface IPTCMetadata {
  creator?: string;
  credit?: string;
  copyright?: string;
  keywords?: string[];
  city?: string;
  province?: string;
  country?: string;
  country_code?: string;
  camera?: string;
  lens?: string;
  film?: string;
  headline?: string;
  instructions?: string;
  source?: string;
  gps?: { lat: number; lng: number };
  [key: string]: unknown; // extensible
}

/** Résultat de pagination */
interface PaginatedImages {
  items: Image[];
  currentPage: number;
  totalPages: number;
  pageSize: number;
}
```

### 3.3 — Interactions requises

| Interaction | Comportement attendu |
|-------------|---------------------|
| Clic sur image (galerie) | Ouvre la lightbox sur cette image |
| Navigation lightbox | ← → au clavier, swipe tactile, boutons |
| Fermeture lightbox | Esc, clic hors image, bouton fermer |
| Pagination | Navigation entre les pages de la galerie |
| Lazy loading | Les images hors viewport ne sont pas chargées |

---

## Couche 4 — Ingestion : sources, snapshots et publication

La couche 1 décrit un corpus au repos, sur un filesystem. Elle ne dit rien du chemin qui l'y amène quand il s'édite ailleurs — dans un dossier Dropbox, iCloud Drive, Google Drive ou WebDAV — et doit être publié sans que la production dépende de cet ailleurs. Sans contrat commun, chaque outil réinvente le listing, la comparaison et la suppression, et le premier listing incomplet devient une suppression massive en production.

La couche 4 fixe ce contrat : comment un état de source devient un **snapshot** complet, identifié et reproductible, comment deux snapshots se comparent, quels diagnostics bloquent une publication, et quel vocabulaire décrit l'état d'une publication.

> **Portée.** La couche 4 est normative **pour les outils d'ingestion** — ce qui lit une source éditoriale, en tire un snapshot, le compare, le valide et le publie. Elle ne change rien à ce qu'un lecteur doit faire (§0, §2.0) : un adaptateur lit toujours un corpus matérialisé sur un filesystem. Un projet qui édite et construit depuis le même dossier n'a besoin d'aucune de ses règles.

### 4.0 — Trois contrats, une frontière

| Contrat | Objet | Section | Qui l'implémente |
|---|---|---|---|
| **Format** | dossier de série, `index.md`, `media/`, frontmatter | §0, couche 1 | tout outil qui écrit ou lit du contenu |
| **Ingestion / snapshot** | état d'une source éditoriale → snapshot complet, validé, reproductible → publication | couche 4 | outils d'ingestion : plugin, CMS, pipeline d'un site |
| **Consommation** | lecture du snapshot publié et des artefacts qui en dérivent | couches 2 et 3 | adaptateurs, sites, applications |

La **source éditoriale** est l'endroit où des humains éditent le corpus : Dropbox, iCloud Drive, Google Drive, un partage WebDAV, un disque local, un checkout Git. Le **provider** est l'accès à cette source. Le **snapshot publié** est l'état complet, validé et reproductible que la production sert, indépendamment de la disponibilité de la source. La **révision source** (`sourceRevision`) identifie un état de la source ; la **révision publiée** (`publishedRevision`) identifie ce qui est en production. Les deux sont distinctes et ne se déduisent pas l'une de l'autre.

```
 source éditoriale (Dropbox · iCloud Drive · Google Drive · WebDAV · filesystem)
        │   provider : listing, lecture, changements
        ▼
 ContentSnapshot N+1  ── diff ──►  ContentChangeSet (contre le snapshot publié N)
        │   validation (§4.10) + garde (§4.11)
        ▼
 publication : matérialisation du corpus + transfert des objets, puis bascule
        ▼
 snapshot publié N+1  ──►  consommateurs (couches 2 et 3) — ne voient jamais le provider
```

Les choix d'infrastructure — dépôt Git pour les textes, stockage objet pour les médias, CDN, hébergeur — appartiennent au consommateur. La spec ne les standardise pas.

#### Invariants

| # | Invariant |
|---|---|
| 1 | **Provider opaque.** Un consommateur d'un snapshot publié ne connaît pas le provider qui l'a produit. Changer de provider ne modifie ni les URLs publiques, ni l'API d'un site, ni ses clients. |
| 2 | **Aucune lecture runtime.** Aucune requête de visiteur ou d'application ne contacte un provider, et aucune URL publiée ne pointe vers lui — lien temporaire compris. Le provider alimente un snapshot : il ne remplace ni le filesystem, ni le chargement de contenu d'un adaptateur. |
| 3 | **Incomplet ≠ suppression.** Un provider indisponible ou un listing incomplet ne s'interprète jamais comme une suppression. Seul un snapshot `complete: true` peut fonder la suppression d'un objet publié. |
| 4 | **N reste intact tant que N+1 n'est pas publié.** La production bascule d'un snapshot complet et validé à un autre. Elle ne se modifie jamais progressivement au fil des événements du provider ; un échec laisse N en place. |
| 5 | **Nettoyage après succès.** Les objets de N devenus inutiles ne sont supprimés qu'après la publication réussie de N+1, selon la politique de rétention du consommateur. |
| 6 | **Idempotence.** Un même état de source, hashé avec les mêmes algorithmes, donne le même snapshot et le même identifiant (§4.6) ; un événement dupliqué ne produit aucun changement. |

Un provider DOIT préserver les chemins relatifs, les octets des fichiers, les renommages et les suppressions. La sémantique d'un corpus distant est celle d'un filesystem (§1.2) : les règles de la couche 1 s'y appliquent sans exception.

### 4.1 — Chemins

Un chemin d'entrée désigne un fichier relativement à la racine du corpus.

| Règle | Description |
|---|---|
| Forme | POSIX, séparateur `/`. Ni `/` initial ni final, aucun segment vide, `.` ou `..`, aucun `\`, aucun caractère de contrôle U+0000–U+001F ni U+007F. Les autres points de code (espace, U+0085, emoji…) sont admis. |
| Normalisation | Unicode **NFC**. La casse est préservée. La normalisation ne corrige rien d'autre : un chemin à `/` initial reste invalide. |
| Collision | Deux chemins égaux après NFC puis **minuscule simple** d'Unicode (`Simple_Lowercase_Mapping` de `UnicodeData.txt`, appliquée point de code par point de code, sans contexte ni locale) sont en collision. Motif : Dropbox, APFS et HFS+ dans leur configuration par défaut sont insensibles à la casse — deux tels chemins ne coexistent pas à la source. |
| Ordre canonique | Tri par la séquence d'octets **UTF-8** du chemin. Ni UTF-16, ni collation locale. |
| Extension | La partie du nom de base qui suit son dernier point, comparée en minuscules. Un nom sans point, ou dont le seul point est initial, n'a pas d'extension. |

> **Minuscule simple, en pratique.** `toLowerCase()` (JavaScript) et `lowercased()` (Swift) appliquent la correspondance **complète** et contextuelle. Appliquées à chaque point de code isolément, elles donnent la correspondance simple, à une exception près : U+0130 `İ` doit donner U+0069 `i`, et non `i` suivi de U+0307. Il n'y a pas de règle du sigma final (`Σ` donne `σ` partout), et `ß` reste `ß`. `fixtures/ingestion/paths/collision.json` couvre ces cas.
>
> **Égalité et ordre, en pratique.** En Swift, `String` compare par équivalence canonique : `"é" == "e\u{301}"` est vrai, et `<` ne suit pas l'ordre des octets UTF-8. Comparer, trier et tester la forme NFC sur les vues `unicodeScalars` ou `utf8` (par exemple `Array(chemin.utf8)`), jamais avec `==` ou `<` sur `String` — sinon un chemin non NFC passe pour valide. En JavaScript, `===` compare bien les points de code, mais `<` et `localeCompare` ne suivent pas l'ordre canonique : trier sur les octets UTF-8.

### 4.2 — Exclusions

Certains fichiers n'entrent jamais dans un snapshot, et leur présence n'est jamais une erreur :

- tout chemin dont **un segment** commence par `.` — `.DS_Store`, `.git/`, `.dropbox`, `._01.jpg`, `.gitkeep` ;
- les noms de base `Thumbs.db`, `desktop.ini` et `Icon\r` (`Icon` suivi de U+000D), comparés exactement, casse comprise ;
- un consommateur PEUT ajouter ses propres exclusions (`_todo/`, `_drafts/`…). Elles modifient le contenu du snapshot : il DEVRAIT les documenter.

L'exclusion s'évalue **avant** la validité : un chemin exclu disparaît sans diagnostic, même s'il est invalide (`Icon\r` contient un caractère de contrôle). Un chemin non exclu et invalide produit `entry-path-invalid` (§4.10). Dans un snapshot, une entrée dont le chemin relève d'une exclusion est non conforme et produit elle aussi `entry-path-invalid`.

### 4.3 — Classification

Chaque entrée porte un `kind`, déterminé par son seul chemin, par la première règle qui s'applique :

| Ordre | Règle | `kind` |
|---|---|---|
| 1 | nom de base exactement `images.json` | `derived` — donnée dérivée, jamais éditoriale par défaut (§1.5.1) |
| 2 | dossier parent **immédiat** nommé exactement `media` | `media` |
| 3 | extension `md` ou `mdx` (casse ignorée) | `content` |
| 4 | tout le reste | `other` |

L'ordre compte : `media/images.json` est `derived`, `media/index.md` est `media`, `media/raw/01.tif` est `other` (son parent immédiat est `raw`), `Media/01.jpg` est `other`. Le `kind` d'une entrée DOIT être égal à la classification de son chemin ; un écart produit `entry-kind-mismatch`, et les règles de validation (§4.10) s'appuient toujours sur la classification recalculée.

Un fichier `index.mdx` reste `content` — le snapshot décrit tout ce que la source contient — mais n'est jamais un fichier index : il est refusé (`index-mdx-forbidden`, §4.10).

Un original (§1.9) reste une entrée `media` : le snapshot d'une source décrit tout ce qu'elle contient. C'est la publication qui l'écarte (§4.11, *Originaux et projection publiable*).

### 4.4 — Hash

Une entrée porte un objet `hashes` : `{ "<algorithme>": "<valeur>" }`.

| Algorithme | Valeur |
|---|---|
| `sha256` | SHA-256 des octets du fichier, en hexadécimal minuscule (64 caractères). |
| `dropbox` | *Content hash* Dropbox : découper le fichier en blocs de 4 194 304 octets (le dernier peut être plus court), calculer le SHA-256 de chaque bloc, concaténer les **digests binaires** (32 octets chacun), calculer le SHA-256 de cette concaténation ; hexadécimal minuscule. Un fichier vide n'a aucun bloc : son hash est le SHA-256 de la chaîne vide, `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`. |
| `x-<nom>` | Tout autre algorithme est préfixé `x-` (`x-etag`, `x-md5`) : valeur opaque, comparée à l'identique, comparable seulement à elle-même. Un ETag s'enregistre tel que le serveur le rend, guillemets compris. |

- Une entrée **matérialisée** DOIT porter au moins un hash. Une entrée `placeholder` (présente à la source mais non matérialisée) ou `conflict` (en conflit de version, §4.5) PEUT n'en porter aucun.
- Un outil qui lit les octets d'une entrée DOIT les vérifier contre son hash : le premier algorithme enregistré qu'elle porte, `sha256` puis `dropbox` (une valeur `x-*` est opaque et ne se vérifie pas). Un écart signe un fichier modifié entre le listing et la lecture : le snapshot n'est pas publiable, et un nouveau listing s'impose. La validation le signale par `entry-hash-mismatch` (§4.10) ; la garde en tire ses propres conséquences (§4.11).
- Pour comparer deux entrées, l'ordre de préférence est `sha256`, puis `dropbox`, puis les `x-*` par ordre alphabétique (octets UTF-8). On retient le **premier algorithme présent sur les deux entrées** : lui seul décide, même si un algorithme suivant diverge. Dans un snapshot, un nom d'algorithme ni enregistré ni préfixé `x-` rend l'entrée invalide (`entry-invalid`, §4.5).

Pour un fichier non vide d'au plus un bloc, `dropbox` est le SHA-256 du digest binaire SHA-256 du contenu : il diffère donc de `sha256`. Pour le fichier vide, les deux valeurs sont égales. Vecteurs multi-blocs, dont 5 000 000 octets nuls : `fixtures/ingestion/hashes/vectors.json`.

### 4.5 — ContentSnapshot v1

Un snapshot est la liste complète des fichiers d'un corpus à un instant donné, avec de quoi identifier leur contenu.

```json
{
  "format": "hyperfocale.snapshot",
  "version": 1,
  "id": "sha256:<hex>",
  "createdAt": "2026-09-29T10:00:00.000Z",
  "complete": true,
  "source": { "provider": "dropbox", "revision": "<opaque>", "root": "/MDR Content" },
  "entries": [
    { "path": "archives/music/foo/index.md", "kind": "content", "size": 1234,
      "hashes": { "sha256": "…", "dropbox": "…" },
      "identity": "id:a4ayc_80_OEAAAAAAAAAXw", "modifiedAt": "2026-09-28T08:00:00Z", "state": "materialized" }
  ]
}
```

| Champ | Requis | Règle |
|---|---|---|
| `format` | oui | `"hyperfocale.snapshot"`. Un document qui porte une autre valeur n'est pas un snapshot. |
| `version` | oui | `1`. Un lecteur DOIT refuser une version inconnue (`snapshot-version-unsupported`). |
| `id` | oui | Calculé (§4.6). Un `id` qui ne correspond pas aux entrées déclarées produit `snapshot-id-mismatch`. |
| `createdAt` | oui | ISO 8601 UTC. Informatif, hors `id`. |
| `complete` | oui | `true` seulement si le listing du provider a abouti sans erreur ni page manquante. Hors `id`. |
| `source` | non | Opaque ; `provider`, `revision` et `root` facultatifs. Hors `id`. Un consommateur d'un snapshot **publié** NE DOIT PAS en dépendre. |
| `entries` | oui | Triées dans l'ordre canonique (§4.1), chemins uniques, chacun valide et non exclu. |
| `entries[].path` | oui | Chemin (§4.1). |
| `entries[].kind` | oui | Classification du chemin (§4.3). Un écart produit `entry-kind-mismatch`. |
| `entries[].size` | oui | Taille en octets, entier ≥ 0. |
| `entries[].hashes` | oui* | *Sauf `state: "placeholder"` ou `"conflict"` (§4.4). |
| `entries[].identity` | non | Identifiant stable attribué par le provider, qui survit au renommage. Hors `id`. |
| `entries[].modifiedAt` | non | Informatif. Hors `id`. |
| `entries[].state` | non | `"materialized"` (défaut) ; `"placeholder"` : présente à la source, contenu non disponible localement (fichier iCloud non téléchargé, par exemple) ; `"conflict"` : le provider signale un conflit de version sur ce fichier (copie conflictuelle Dropbox, versions concurrentes iCloud). Une entrée `placeholder` ou `conflict` rend le snapshot impubliable. La détection d'un conflit incombe au provider, qui PEUT marquer l'entrée `conflict`. Hors `id`. |

Champs inconnus : un lecteur les ignore et les transmet (passthrough) ; un écrivain NE DOIT PAS en créer hors préfixe `x-`. Un lecteur ne suppose pas l'ordre des entrées : tout calcul (§4.6, §4.7) les trie d'abord.

**Rejet structurel.** Un lecteur vérifie la structure avant tout le reste, dans cet ordre :

1. le document est un objet JSON dont `format` vaut `"hyperfocale.snapshot"` — sinon ce n'est pas un snapshot : `snapshot-invalid` ;
2. `version` vaut `1` — sinon `snapshot-version-unsupported` ;
3. `id` et `createdAt` sont des chaînes, `complete` un booléen, `entries` un tableau — sinon `snapshot-invalid`.

Chacun de ces trois diagnostics est alors le seul produit : rien d'autre n'est vérifié. Une **entrée** est ensuite structurellement invalide (`entry-invalid`) si elle n'est pas un objet, si `path` n'est pas une chaîne (le diagnostic porte alors le chemin `""`), si `kind` n'est pas l'une des quatre valeurs de §4.3, si `size` n'est pas un entier ≥ 0, si `state` porte une valeur inconnue, si `hashes` n'est pas un objet, si une valeur `sha256` ou `dropbox` n'est pas faite de 64 chiffres hexadécimaux minuscules, si une valeur `x-*` n'est pas une chaîne, ou si un algorithme n'est ni enregistré ni préfixé `x-`. Une entrée invalide est écartée de tout le reste de la validation. L'identifiant (§4.6) et le diff (§4.7) portent sur des snapshots structurellement valides.

### 4.6 — Identifiant de snapshot

```
id = "sha256:" + hex(SHA-256(utf8(L)))
```

`L` est la concaténation, dans l'ordre canonique des entrées, d'une ligne par entrée :

```
<path>\t<kind>\t<size>\t<alg1>=<valeur1>,<alg2>=<valeur2>\n
```

- `\t` est U+0009, `\n` est U+000A ; `size` s'écrit en décimal, sans signe ni zéro initial ;
- les hashes sont triés par nom d'algorithme (octets UTF-8) et séparés par `,` ; une entrée sans hash donne une liste vide (`…\t<size>\t\n`) ;
- un snapshot sans entrée a pour `L` la chaîne vide : `sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`.

Seuls `path`, `kind`, `size` et `hashes` entrent dans l'identifiant. Conséquence voulue : un même état de source hashé avec les mêmes algorithmes donne le même `id`, quels que soient l'instant, la machine ou l'implémentation. Deux snapshots d'un même état calculés avec des algorithmes différents ont des `id` différents : un changement de provider se voit. Un pipeline DEVRAIT donc choisir un jeu d'algorithmes et s'y tenir d'une ingestion à l'autre — un snapshot Dropbox (`dropbox` seul) et le même état recalculé avec `sha256` ne partagent pas d'`id`.

### 4.7 — ContentChangeSet v1

Un changeset décrit ce qui sépare un snapshot `base` (le dernier publié, ou `null` pour une première publication) d'un snapshot `target`.

```json
{
  "format": "hyperfocale.changeset",
  "version": 1,
  "base": "sha256:… | null",
  "target": "sha256:…",
  "added":    [ <entrée> ],
  "modified": [ { "path": "…", "before": <entrée>, "after": <entrée> } ],
  "deleted":  [ <entrée> ],
  "moved":    [ { "from": "…", "to": "…", "before": <entrée>, "after": <entrée>, "modified": false } ],
  "diagnostics": [ <diagnostic> ]
}
```

Les entrées y figurent telles qu'en snapshot, champs informatifs compris. L'algorithme est déterministe :

0. Si `base` n'est pas `null` et que les deux snapshots **déclarent** le même `id`, le changeset est vide. La cohérence d'un `id` déclaré avec ses entrées relève de la validation (`snapshot-id-mismatch`, §4.10), pas du diff.
1. **Chemins présents des deux côtés.** L'entrée est `modified` si `kind` ou `size` diffère. Sinon on cherche le premier algorithme commun (§4.4) : s'il n'y en a pas, l'entrée est `modified` et reçoit le diagnostic `hash-incomparable` (warning) — prudence : jamais « inchangé » par défaut ; s'il y en a un, l'entrée est `modified` si les deux valeurs diffèrent.
2. Restent `A`, les chemins présents seulement dans `target`, et `D`, les chemins présents seulement dans `base`.
3. **Déplacements par identité.** Une `identity` non vide portée par exactement une entrée de `D` et exactement une entrée de `A` apparie ces deux entrées en `moved`. `modified` vaut `true` si `size` diffère ou si le premier algorithme commun donne deux valeurs différentes ; sans algorithme commun, `modified` vaut `true`. Comme à l'étape 1, `hash-incomparable` exige que `kind` **et** `size` soient égaux : il porte alors sur le chemin `to` ; un déplacement dont la taille ou le `kind` change est `modified: true` sans diagnostic. Hors de ce diagnostic, `kind` n'entre pas dans la comparaison : il dérive du chemin, qui change par définition.
4. **Déplacements par contenu**, parmi les entrées restantes. Une entrée `d` de `D` et une entrée `a` de `A` sont **candidates** l'une de l'autre si elles ont la même `size` et un premier algorithme commun de même valeur. Si `d` n'a que `a` pour candidate et `a` n'a que `d`, l'appariement est unique : `moved`, avec `modified: false`. Toute entrée qui a au moins une candidate sans appariement unique reçoit `move-ambiguous` (info) sur son chemin, et reste en `added` ou `deleted`.
5. Le reste de `A` est `added`, le reste de `D` est `deleted`.
6. **Tri** : `added`, `modified` et `deleted` par `path`, `moved` par `to`, dans l'ordre canonique ; `diagnostics` par `path` puis `code`, au plus un par couple `(code, path)`.
7. `diff(S, S)` est vide, sans diagnostic — l'étape 0 le garantit, y compris pour des entrées `placeholder` sans hash.

Une première publication (`base: null`) range toutes les entrées de `target` en `added`. Le changeset ne dit rien de la validité du contenu : c'est le rôle de la validation (§4.10) et de la garde (§4.11).

### 4.8 — ProviderCapabilities v1

Tous les providers ne savent pas tout faire. Un provider déclare ses capacités ; le pipeline s'y adapte.

```json
{ "localRead": true, "localWrite": false, "remoteRead": true, "remoteWrite": false,
  "incrementalChanges": true, "stableIdentity": true, "serverWebhook": true,
  "clientChangeObservation": false, "materializationAware": false,
  "hashAlgorithms": ["dropbox"] }
```

| Capacité | Sens |
|---|---|
| `localRead` / `localWrite` | lecture / écriture sur un filesystem local |
| `remoteRead` / `remoteWrite` | lecture / écriture par une API distante |
| `incrementalChanges` | changements depuis un curseur, sans listing complet |
| `stableIdentity` | `identity` qui survit au renommage (§4.5) |
| `serverWebhook` | notification de changement envoyée à un serveur |
| `clientChangeObservation` | changements observables seulement par une application cliente |
| `materializationAware` | le provider distingue un fichier présent d'un fichier matérialisé (`state: "placeholder"`) |
| `hashAlgorithms` | algorithmes que le provider fournit sans lire les octets (§4.4) |

Une capacité booléenne absente vaut `false` ; `hashAlgorithms` absent vaut `[]`. Le pipeline NE DOIT jamais supposer une capacité absente : sans webhook, il réconcilie périodiquement ; sans changements incrémentaux, il relit le listing complet ; sans identité stable, il n'infère les déplacements que par contenu (§4.7).

Valeurs de référence, indicatives :

| Provider | Capacités |
|---|---|
| Filesystem | `localRead`, `localWrite`, `hashAlgorithms: ["sha256", "dropbox"]` (calculés) |
| Dropbox | `remoteRead`, `remoteWrite`, `incrementalChanges`, `stableIdentity`, `serverWebhook`, `hashAlgorithms: ["dropbox"]` |
| WebDAV | `remoteRead`, `remoteWrite`, `hashAlgorithms: ["x-etag"]` |
| iCloud Drive (depuis un CMS) | `localRead`, `localWrite`, `clientChangeObservation`, `materializationAware`, `stableIdentity`, `hashAlgorithms: ["sha256"]` |

Google Drive relève du même contrat — identifiants stables, jeton de changements — et reste un provider cible au même titre que les autres.

### 4.9 — PublicationState

Vocabulaire commun de l'état d'une publication. Un CMS et un site PEUVENT afficher des libellés différents ; la sémantique DOIT rester celle-ci.

| État | Sens |
|---|---|
| `sourceSynced` | la source correspond au dernier snapshot publié |
| `sourceDirty` | la source a divergé du dernier snapshot publié |
| `snapshotPending` | un snapshot de la source est en construction (listing, lecture, hashes) |
| `validating` | le snapshot est construit ; diff, validation et garde en cours |
| `publishing` | le snapshot est validé ; transfert des objets et bascule en cours |
| `published` | le snapshot est la révision servie |
| `failed` | la dernière tentative a échoué ; la révision publiée précédente reste servie |
| `conflict` | le provider signale un conflit de version (fichier dupliqué en « copie en conflit », entrée `state: "conflict"`) ; la publication est bloquée |

```json
{ "format": "hyperfocale.publication", "version": 1, "state": "published",
  "sourceProvider": "dropbox", "sourceRevision": "<opaque>",
  "snapshot": "sha256:…", "publishedRevision": "<défini par le consommateur, ex. commit git>",
  "updatedAt": "2026-09-29T10:05:00.000Z", "error": { "code": "…", "message": "…" } }
```

`format`, `version` (`1`), `state` et `updatedAt` (ISO 8601 UTC) sont requis. `sourceProvider` et `sourceRevision` décrivent la source ; `snapshot` est l'`id` du snapshot concerné ; `publishedRevision` est défini par le consommateur (commit Git, identifiant de déploiement). `error` accompagne `failed` et `conflict` ; son `code` reprend un code de diagnostic (§4.10, §4.11) quand il y en a un.

Transitions — la liste est **exhaustive** : toute autre transition est interdite.

| Depuis | Vers | Quand |
|---|---|---|
| — | `sourceDirty` | état initial, avant toute publication |
| — | `sourceSynced` | amorçage depuis un snapshot déjà publié (pipeline mis en service sur une production existante) |
| `sourceDirty` | `snapshotPending` | une ingestion démarre |
| `snapshotPending` | `validating` | le snapshot est construit |
| `validating` | `publishing` | validation et garde sans erreur |
| `publishing` | `published` | la bascule a réussi |
| `published` | `sourceSynced` | la source n'a pas bougé depuis la publication |
| `sourceSynced`, `published`, `failed`, `conflict` | `sourceDirty` | la source diverge du dernier snapshot publié |
| `failed` | `snapshotPending` | nouvel essai, source inchangée (erreur transitoire du provider, par exemple) |
| tout état | `failed` | la tentative échoue — `validating` → `failed` compris ; la révision publiée précédente n'est jamais altérée |
| tout état | `conflict` | le provider signale un conflit ; aucune publication n'a lieu tant qu'il persiste |

### 4.10 — Diagnostics

```json
{ "code": "slug-invalid", "severity": "error", "path": "archives/Foo_Bar", "message": "…", "rule": "§1.2" }
```

`severity` vaut `error`, `warning` ou `info`. `message` est libre ; `rule`, facultatif, renvoie à la règle de la spec. Deux implémentations sont conformes si elles produisent les mêmes triplets **`code` + `severity` + `path`** — jamais le message. Une liste de diagnostics est triée par `path` (ordre canonique) puis par `code`, et ne contient jamais deux fois le même couple `(code, path)`. Un diagnostic qui porte sur le snapshot entier a pour `path` la chaîne vide. Un consommateur qui ajoute ses propres contrôles les code avec le préfixe `x-`. Un code PEUT définir des champs supplémentaires, normatifs pour lui seul : ils se comparent alors avec le triplet (`count` et `bytes` de `guard-originals-excluded`, §4.11).

| Code | Sévérité | Règle | `path` |
|---|---|---|---|
| `snapshot-invalid` | error | `format` ≠ `"hyperfocale.snapshot"`, ou champ requis absent ou mal typé (§4.5) | `""` |
| `snapshot-version-unsupported` | error | `version` ≠ 1 | `""` |
| `snapshot-id-mismatch` | error | `id` déclaré ≠ `id` recalculé (§4.6) à partir des entrées telles que déclarées | `""` |
| `snapshot-incomplete` | error | `complete: false` — interdit toute publication, donc toute suppression | `""` |
| `snapshot-empty` | error | aucune entrée `content` | `""` |
| `entry-invalid` | error | entrée structurellement invalide (§4.5) | l'entrée, ou `""` si `path` n'est pas une chaîne |
| `entry-path-invalid` | error | chemin non conforme à §4.1, non NFC, ou exclu (§4.2) | l'entrée |
| `entry-kind-mismatch` | error | `kind` déclaré ≠ classification du chemin (§4.3) | l'entrée |
| `entry-path-collision` | error | collision (§4.1) | le second chemin dans l'ordre canonique, et chacun des suivants |
| `entry-hash-missing` | error | entrée matérialisée sans hash (`hashes` absent ou vide) | l'entrée |
| `entry-not-materialized` | error | `state: "placeholder"` — publication impossible | l'entrée |
| `entry-hash-mismatch` | error | octets lus ≠ hash de l'entrée (§4.4) — fichier modifié entre le listing et la lecture | l'entrée |
| `entry-conflict` | error | `state: "conflict"` — le provider signale un conflit de version, publication impossible | l'entrée |
| `slug-invalid` | error | dossier porteur d'un fichier index dont le nom ne suit pas `^[a-z0-9]+(-[a-z0-9]+)*$` (§1.2) | le dossier |
| `media-nested` | error | dossier sous `media/` (§1.2) : entrée dont un segment ancêtre autre que le parent immédiat est `media` | le dossier imbriqué |
| `media-orphan` | warning | dossier `media` dont le dossier parent ne porte aucun fichier index | le dossier `media` |
| `index-default-missing` | warning | `index.<lang>.md` présent sans `index.md` | le dossier |
| `nesting-too-deep` | error | série sous une sous-série (§1.8) ; les sections `type: section` ne comptent pas | la série la plus profonde |
| `section-has-media` | warning | `type: section` et `media/` (§1.10) | le dossier |
| `frontmatter-missing` | error | fichier index sans bloc `---` initial refermé | le fichier |
| `frontmatter-invalid` | error | YAML illisible ou qui n'est pas un mapping | le fichier |
| `title-missing` | error | pas de `title` chaîne non vide | le fichier |
| `date-missing` | error | série sans `date` (sauf `type: section`, ou racine `dateRequired: false`) | le fichier |
| `date-invalid` | error | `date` qui n'est pas une date ISO 8601 valide | le fichier |
| `type-invalid` | error | `type` ∉ {`series`, `section`} | le fichier |
| `cover-not-found` | warning | `cover` relatif introuvable, dans le snapshot comme dans `images.json` | le fichier |
| `cover-not-image` | error | `cover` relatif qui désigne un fichier non image (§1.9) | le fichier |
| `images-conflict` | error | `images:` et `images.json` dans la même série (§1.5.1) | le fichier |
| `images-json-invalid` | warning | JSON illisible, racine qui n'est pas un objet, clé `images` absente ou non tableau (§1.5.1) | le `images.json` |
| `attachment-not-found` | warning | entrée `attachments[]` dont le `file` est absent de `media/` ou désigne un original (§1.9) | le fichier |
| `embed-url-missing` | error | entrée `embeds[]` sans `url` (§1.11) | le fichier |
| `private-type-invalid` | error | `private` présent (non `null`) et non booléen : `"true"`, `yes`, `1`… (§1.12) | le fichier |
| `private-key-suspect` | warning | clé qui ne diffère de `private` que par la casse, des espaces, `_` ou `-` (§1.12) | le fichier |
| `index-mdx-forbidden` | error | fichier `index.mdx` ou `index.<lang>.mdx` : MDX est du code exécutable (§1.12) | le fichier |
| `url-scheme-forbidden` | error | valeur d'un champ d'URL du core hors des formes admises : schéma autre que `http:`/`https:`, `//hôte`, caractère de contrôle (§1.12) | le fichier où la valeur est écrite |
| `hash-incomparable` | warning | diff : `kind` et `size` égaux, aucun algorithme commun (§4.7) | l'entrée (le `to` d'un déplacement) |
| `move-ambiguous` | info | diff : appariement non unique (§4.7) | l'entrée |

La sévérité est celle d'un **outil d'ingestion**, qui décide d'une publication automatique : elle peut être plus stricte que la règle de rendu d'un adaptateur. Un adaptateur continue d'ignorer une entrée `embeds` sans `url` (§1.11) ; l'ingestion, elle, refuse de publier sans intervention.

#### Fichiers index et racines

**Fichiers index** : entrées de `kind` `content` dont le nom de base est `index.md` ou `index.<lang>.md` (Annexe F, stratégie 2), avec `<lang>` = `[a-z]{2}(-[A-Z]{2})?`. `index.english.md` ou `index.EN.md` n'en sont pas : ils sont copiés, jamais validés. Chaque fichier index est validé indépendamment. Un **dossier porteur** est un dossier qui contient directement au moins un fichier index. *(v2.17)* `index.mdx` et `index.<lang>.mdx` ne sont **pas** des fichiers index : ils ne sont jamais lus, ne rendent pas leur dossier porteur, et produisent `index-mdx-forbidden` (§1.12) dans toute racine de validation, quel que soit leur état (`placeholder` et `conflict` compris : le diagnostic ne dépend que du chemin).

**Racines de validation** : la validation du contenu porte sur des racines configurées, `roots: [{ "path": "archives", "dateRequired": true }, …]` (`dateRequired` vaut `true` par défaut). Défaut : une racine unique `""` (tout le corpus), `dateRequired: true`. Un fichier appartient à la racine la plus longue qui le contient ; hors racines, les fichiers sont copiés, pas validés. Les diagnostics `snapshot-*` et `entry-*` portent sur le snapshot entier, indépendamment des racines.

#### Règles d'évaluation

1. **Structure et version.** Dans l'ordre de §4.5 : `format`, puis `version`, puis les champs requis. `snapshot-invalid` ou `snapshot-version-unsupported` est alors le seul diagnostic produit ; rien d'autre n'est vérifié.
2. **Identifiant.** Si toutes les entrées sont structurellement valides, l'`id` est recalculé (§4.6) à partir des entrées telles que déclarées — `kind` déclaré compris, entrées au chemin invalide comprises. S'il diffère de l'`id` déclaré : `snapshot-id-mismatch`. La validation continue. Si une entrée est `entry-invalid`, l'`id` n'est pas recalculable et la règle est sans objet.
3. **Entrées.** Les entrées structurellement invalides (`entry-invalid`), puis les entrées au chemin invalide (`entry-path-invalid`), sont écartées de tout le reste. Pour les entrées retenues, un `kind` déclaré différent de la classification du chemin produit `entry-kind-mismatch` ; l'entrée reste dans la validation avec la classification recalculée. Les collisions se cherchent parmi les entrées retenues, `snapshot-empty` aussi. Une entrée `placeholder` produit `entry-not-materialized`, une entrée `conflict` produit `entry-conflict` ; ni l'une ni l'autre n'est jamais lue : un tel fichier index ne produit aucun diagnostic de contenu.
4. **Lecture.** Seuls sont lus les fichiers index et les `images.json` des racines, à l'état `materialized`. Les octets lus sont vérifiés contre le hash de l'entrée (§4.4) ; un écart produit `entry-hash-mismatch`, et le fichier est traité comme non lu : aucun diagnostic de contenu, et un manifeste non lu ne sert pas à la recherche de couverture. Le texte est de l'UTF-8 ; un BOM initial est ignoré ; des octets qui ne forment pas de l'UTF-8 valide rendent le fichier illisible (`frontmatter-invalid`, `images-json-invalid`).
5. **Bloc de frontmatter.** La première ligne vaut exactement `---` ; le bloc se ferme à la ligne suivante qui vaut exactement `---` (fins de ligne `\n` ou `\r\n`). Sans ligne d'ouverture ou sans ligne de fermeture : `frontmatter-missing`.
6. **YAML.** Le bloc s'interprète selon le **schéma YAML 1.2 *core*** : une date non guillemetée y reste une chaîne, validée par la règle 8. Une clé dupliquée rend le YAML illisible. Un bloc vide, ou dont la racine n'est pas un mapping : `frontmatter-invalid`. Après `frontmatter-missing` ou `frontmatter-invalid`, aucun diagnostic de champ n'est produit pour ce fichier.
7. **Champs.** Une clé de valeur `null` vaut absente (`type`, `date`, `cover`, `images`, `private`). `title` doit être une chaîne d'au moins un caractère. `type`, s'il est présent, vaut `series` ou `section` ; toute autre valeur produit `type-invalid`, et le fichier est traité en série. `private`, s'il est présent, vaut un booléen YAML (`true`, `True`, `TRUE`, `false`, `False`, `FALSE`) ; toute autre valeur — chaîne, entier, liste — produit `private-type-invalid`, **y compris dans une page de section** et dans un `index.<lang>.md`. Une clé distincte de `private` dont la forme normalisée (casse ignorée, espaces, `_` et `-` retirés) est `private` produit `private-key-suspect`.
8. **Date.** Une date valide est une chaîne `AAAA-MM-JJ`, éventuellement suivie de `THH:MM`, de `:SS`, d'une fraction `.S…` et d'un fuseau `Z` ou `±HH:MM`, qui forme une date calendaire réelle (heures 00–23, minutes et secondes 00–59). `2024-02-30` produit `date-invalid`. Pour une section, `date` n'est pas vérifiée (§1.10).
9. **Références relatives.** Une référence est relative si elle n'est pas vide, ne commence pas par `/` et ne porte pas de schéma (`https:`, `data:`…). Elle se résout depuis le dossier du fichier index : les segments vides et `.` sont ignorés, `..` remonte d'un niveau ; une résolution qui sort de la racine du corpus n'aboutit pas.
10. **Couverture.** Seul un `cover` relatif est vérifié. `cover-not-image` dépend de la seule extension, que le fichier existe ou non. `cover-not-found` est produit si le chemin résolu n'est pas une entrée du snapshot **et** qu'aucune entrée du manifeste `images.json` de la série (chaîne, ou `url` d'un objet) ne lui correspond : une entrée relative correspond si elle se résout au même chemin ; une entrée absolue, si elle se termine par `/` suivi du chemin résolu (`/content/<chemin résolu>`, `https://cdn.example.com/<chemin résolu>`).
11. **Manifeste.** `images-conflict` : le frontmatter porte `images` et le dossier contient un `images.json`. `images-json-invalid` s'évalue pour tout `images.json` des racines.
12. **Documents joints et embeds.** Chaque entrée de `attachments` dont `file` manque, n'est pas relatif, ou ne se résout pas vers une entrée existante située directement dans `<dossier>/media/`, ou se résout vers un original (extension de la liste fermée de §1.9), produit `attachment-not-found`. Chaque entrée de `embeds` qui n'est pas un objet portant une `url` chaîne non vide produit `embed-url-missing`.
13. **Type d'un dossier porteur.** Il se lit dans `index.md`, sinon dans le premier `index.<lang>.md` de l'ordre canonique. Un fichier illisible, aux octets divergents (§4.4), `placeholder` ou `conflict` vaut `series` : la nature de section ne se devine jamais (§1.10).
14. **Structure.** `slug-invalid` porte sur le dernier segment de chaque dossier porteur, sauf le dossier qui est lui-même la racine de validation : il n'a pas de slug, et un `index.md` à la racine du corpus est licite. `nesting-too-deep` : une série qui compte au moins deux dossiers porteurs de type série parmi ses ancêtres situés dans sa racine (le dossier racine compris) — un diagnostic par série fautive. `media-nested` : pour chaque entrée d'une racine, le dossier situé juste sous le segment `media` ancêtre le moins profond, parent immédiat exclu. `media-orphan` : tout dossier nommé `media` dont le parent ne porte aucun fichier index. Pour ces deux règles, seuls comptent les segments situés sous la racine.
15. **Schémas d'URL.** `url-scheme-forbidden` : une valeur chaîne de `cover`, de `images[].url`, `files[].url`, `embeds[].url` ou `embeds[].poster` du frontmatter, ou d'une entrée de `images.json` (chaîne, ou `url` d'un objet de `images` ou de `files`), qui n'a pas une forme admise de §1.12. Une valeur absente, `null`, vide ou qui n'est pas une chaîne n'est pas concernée (une `url` d'embed absente relève d'`embed-url-missing`). Le contrôle porte sur la valeur telle qu'écrite et vaut aussi pour un `cover` que la règle 10 ne vérifie pas (non relatif) ; une valeur refusée ne subit aucun autre contrôle (pas de `cover-not-image` ni de `cover-not-found`). Un fichier à l'état `placeholder`, `conflict` ou aux octets divergents n'est pas lu (règles 3 et 4) : aucun diagnostic de champ.

### 4.11 — Garde de publication

La garde est un mécanisme générique ; ses seuils appartiennent au consommateur.

```
guardChangeSet(changeSet, base, target, { read, policy }) → diagnostics
```

`target` est le snapshot de la source ; `changeSet` est le diff de `base` (ou `null`) vers `publishable(target)` (§4.7), sa projection publiable — égale à `target` quand il ne porte aucun original (voir *Originaux et projection publiable*). `read(côté, chemin)` rend les octets du fichier `chemin` dans `base` (`côté = "base"`) ou dans `target` (`côté = "target"`) : la garde lit les fichiers index pour connaître la confidentialité des séries, et vérifie les octets lus contre le hash de l'entrée (§4.4). Elle n'émet pas `entry-hash-mismatch` — ce diagnostic appartient à la validation (§4.10) — mais traite un écart comme un fichier illisible (voir *Série privée*). `policy` porte les seuils, tous facultatifs : `maxDeletedSeries` (entier), `maxDeletedMediaRatio` (nombre entre 0 et 1), `maxMovedSeries` (entier), `maxFileBytes` (objet `{ "<kind>": entier }`). Les diagnostics suivent les conventions de §4.10.

| Code | Sévérité | Déclenchement | `path` |
|---|---|---|---|
| `guard-snapshot-incomplete` | error | `target` n'est pas `complete: true` | `""` |
| `guard-snapshot-empty` | error | `target` n'a aucune entrée `content` | `""` |
| `guard-mass-deletion` | error | séries supprimées > `maxDeletedSeries`, ou médias supprimés > `maxDeletedMediaRatio` × médias de `base` | `""` |
| `guard-mass-move` | warning | séries déplacées > `maxMovedSeries` | `""` |
| `guard-private-exposed` | error | une série privée dans `base`, présente dans `target`, n'y est plus privée | le dossier de la série dans `target` |
| `guard-oversize` | error | entrée de `publishable(target)` dont la taille dépasse `maxFileBytes[kind]` | l'entrée |
| `guard-originals-excluded` | info | `target` porte au moins une entrée originale, retirée du snapshot publié. Le diagnostic porte `count`, le nombre d'entrées originales, et `bytes`, la somme de leurs `size` (entiers) | `""` |

- **Série** : dossier porteur d'un fichier index (§4.10). Les pages d'index de section (`type: section`, §1.10) en sont, **par prudence** *(précisé en v2.15)* : une section supprimée ou déplacée compte comme une série supprimée ou déplacée, ce qui rend les seuils `maxDeletedSeries` et `maxMovedSeries` plus stricts, jamais plus lâches. La garde ne lit pas le `type` d'un dossier pour compter. **Série supprimée** : dossier porteur dans `base`, qui ne l'est plus dans `target`, et dont aucun fichier index n'est la source (`from`) d'un `moved`. **Série déplacée** : dossier distinct parmi les `from` des `moved` qui désignent un fichier index. **Médias supprimés** : entrées `media` de `deleted`, rapportées au nombre d'entrées `media` de `base`. Une série déplacée n'est jamais comptée comme supprimée.
- **Série privée** : au moins un de ses fichiers index, à l'état `materialized`, déclare `private: true` — un **booléen** YAML (schéma core : `true`, `True`, `TRUE`). La chaîne `"true"` ne rend pas une série privée : la validation la refuse (`private-type-invalid`, §4.10), et la garde ne la lit pas comme un marqueur — une série privée dans `base` dont le `private` devient `"true"` ou `yes` dans `target` y est **exposée** (`guard-private-exposed`). Le frontmatter se lit selon les règles 4 à 7 de §4.10. Un fichier index est **illisible** s'il n'est pas à l'état `materialized` (`placeholder`, `conflict`), si ses octets divergent de son hash, ou si son frontmatter ne se lit pas (UTF-8 invalide, bloc absent, YAML illisible ou qui n'est pas un mapping). La garde est **fail-closed côté `base`** : un fichier index illisible y rend la série privée par prudence. Côté `target`, un fichier illisible ne déclare rien : une série privée dans `base` dont l'homologue est illisible est donc exposée. La garde lit tous les fichiers index de chaque série de `base`, puis ceux de la série homologue dans `target` quand la série de `base` est privée.
- **Exposition** : une série privée dans `base` est présente dans `target` au même chemin si le dossier y porte encore un fichier index, sinon à la destination (dossier du `to`) du premier de ses fichiers index déplacés, dans l'ordre canonique. Si elle n'y est plus privée, `guard-private-exposed` porte sur ce dossier de `target`. La **suppression** d'une série privée n'est pas une exposition.
- `private` n'est pas un champ du format : c'est une extension de site (§0.5), la seule marque de confidentialité que connaissent les consommateurs actuels. Sa valeur est un booléen (§1.12) ; la validation (`private-type-invalid`, `private-key-suspect`) est le premier garde-fou, la garde ne comparant que `base` et `target`. La garde s'applique aux corpus qui l'emploient ; un corpus qui ne l'emploie pas ne la déclenche jamais.
- `guard-snapshot-incomplete`, `guard-snapshot-empty`, `guard-private-exposed` et `guard-originals-excluded` sont toujours évaluées, sans seuil ni désactivation possible. Les gardes à seuil sont inactives quand leur seuil est absent. Avec `base: null`, seules `guard-snapshot-incomplete`, `guard-snapshot-empty`, `guard-oversize` et `guard-originals-excluded` s'évaluent, et rien n'est lu.
- Le consommateur décide : un diagnostic `error`, de garde ou de validation, interdit la publication automatique.

#### Originaux et projection publiable *(introduit v2.11)*

Une **entrée originale** est une entrée dont le `kind` est `media` — classification recalculée de son chemin (§4.3) — et dont l'extension (§4.1) appartient à la liste fermée des originaux (§1.9). Une entrée `other` n'est jamais originale, même avec une extension RAW : la catégorie n'existe que dans `media/`.

Le snapshot publié ne contient jamais d'original :

```
publishable(snapshot) → snapshot
```

La **projection publiable** reprend le snapshot à l'identique — `format`, `version`, `createdAt`, `complete`, `source`, champs inconnus — sans ses entrées originales, avec un `id` recalculé (§4.6) sur les entrées restantes. Un snapshot sans original est sa propre projection, au même `id`.

- Ce qui se publie est `publishable(target)`. Le diff que reçoit la garde, et que la publication applique, va de `base` à `publishable(target)`. La validation (§4.10), elle, porte sur `target` tel quel : elle signale un `cover` ou une entrée `attachments:` qui désigne un original.
- `guard-oversize` s'évalue sur la projection : un RAW de 80 Mo ne bloque pas une publication dont il est exclu.
- `guard-originals-excluded` est de sévérité `info` : l'exclusion est le comportement attendu, le diagnostic en rapporte le poids — un dossier de travail confié par erreur à l'ingestion se voit à ses `bytes`.
- Un snapshot publié par un outil antérieur à v2.11 peut contenir des originaux. Le premier diff vers une projection les range en `deleted` ; ils comptent dans `maxDeletedMediaRatio`, et c'est voulu : leur retrait est une suppression réelle de la production.
- Cette exclusion est un filet, pas un mode de publication : un dossier de travail se publie par export (Hyperfocus, §5), entre autres parce que la garde ne retire pas le bloc `project:` de son `index.md`.

### 4.12 — Fixtures de conformité

`fixtures/ingestion/`, dans ce dépôt, contient les fixtures de la couche 4 : corpus d'entrée, snapshots, projections publiables, validations, diffs, gardes, identifiants, chemins et vecteurs de hash, avec leurs résultats attendus. Elles sont normatives au même titre que ce texte : une implémentation est conforme si elle les passe toutes, ce qui suppose de savoir calculer `sha256` **et** `dropbox`. Leur format et leurs règles de comparaison sont décrits dans `fixtures/ingestion/README.md`. Un consommateur les copie à une ref épinglée ; toute évolution du contrat met à jour les fixtures dans la même révision que la prose.

### 4.13 — Exemples de providers *(non normatif)*

Ces exemples montrent comment des providers réels se projettent sur le contrat. Aucun n'est normatif — Dropbox pas plus que les autres : ce que la spec exige tient dans §4.1 à §4.11.

#### Dropbox

| Contrat | Projection Dropbox |
|---|---|
| Listing complet | `files/list_folder` récursif, puis `files/list_folder/continue` jusqu'à `has_more: false`. `complete: true` seulement si toutes les pages ont abouti. |
| Changements | curseur sauvegardé → `files/list_folder/continue`. Une erreur `reset` (curseur expiré) impose un listing complet — jamais une interprétation « tout supprimé ». Une suppression de dossier vaut pour tout son contenu. |
| Hash | `content_hash` → `dropbox` (§4.4), sans télécharger le fichier. |
| Identité | `id` du fichier (`id:…`) → `identity` : survit aux renommages et déplacements. |
| Chemins | `path_display` n'est fiable qu'en son dernier segment : la casse des dossiers se reconstruit depuis leurs propres entrées, puis NFC. |
| Exclusions | `.dropbox` et `.dropbox.cache` tombent sous §4.2. |
| Conflit | la détection des conflits incombe au provider : une « copie en conflit » Dropbox est le signal qu'il traduit en entrée `state: "conflict"` (`entry-conflict`, §4.5), et la publication passe à l'état `conflict` (§4.9). |
| Webhook | simple **déclencheur** : la notification ne porte aucun contenu ; sa signature (`X-Dropbox-Signature`, HMAC-SHA256 du corps avec le secret de l'application) se vérifie, puis une ingestion démarre. Aucune lecture Dropbox au runtime. |

```json
{ "path": "archives/music/concerts/2010/the-wombats-paris-2010/media/01.jpg",
  "kind": "media", "size": 2481530,
  "hashes": { "dropbox": "…" },
  "identity": "id:a4ayc_80_OEAAAAAAAAAXw", "modifiedAt": "2026-09-28T08:00:00Z" }
```

L'identifiant d'un tel snapshot ne porte que l'algorithme `dropbox` ; recalculer `sha256` à la lecture des octets donne un autre `id` (voir §4.6).

#### WebDAV

| Contrat | Projection WebDAV |
|---|---|
| Listing | `PROPFIND` en `Depth: 1`, dossier par dossier (`Depth: infinity` est souvent désactivé). |
| Hash | `getetag` → `x-etag`, tel quel. Un ETag ne se compare qu'à un ETag : passer de Dropbox à WebDAV rend tout le corpus `hash-incomparable` au premier diff, par prudence. |
| Taille, date | `getcontentlength` → `size`, `getlastmodified` → `modifiedAt`. |
| Changements | aucun mécanisme standard : réconciliation périodique par listing complet. Sans identité stable, les déplacements ne s'infèrent que par contenu. |
| Lecture | `GET`. |

#### iCloud Drive (depuis un CMS)

iCloud Drive n'offre pas d'API serveur : les changements ne s'observent que depuis une application cliente (`clientChangeObservation`). Un fichier peut y être présent sans être téléchargé : il entre au snapshot en `state: "placeholder"`, sans hash, et bloque la publication (`entry-not-materialized`) jusqu'à sa matérialisation. Des versions concurrentes d'un même fichier sont signalées par le provider en `state: "conflict"` (`entry-conflict`). Les hashes (`sha256`) se calculent localement sur les octets. L'application construit le snapshot et le transmet au pipeline d'ingestion : une fois ingéré, il se publie exactement comme un snapshot Dropbox, et l'infrastructure de production ne lit jamais iCloud.

---

## Annexes

### A — Validation du format

Un outil CLI `hyperfocale-lint` est recommandé pour valider la conformité :

```bash
hyperfocale-lint ./content/series/
```

Vérifications :
- [ ] Chaque dossier série contient `index.md` ; aucun `index.mdx` (§1.12)
- [ ] Frontmatter contient `title`, et `date` sauf dans une collection `dateRequired: false` (§1.3) — une section ne requiert que `title` (§1.10)
- [ ] `type`, si présent, vaut `series` ou `section`
- [ ] Dans une collection où `date` est requise, un `index.md` sans `date` déclare explicitement `type: section` (une série sans date y reste une erreur)
- [ ] `date` est au format ISO 8601
- [ ] `slug` respecte le pattern `^[a-z0-9]+(-[a-z0-9]+)*$`
- [ ] Un `cover` relatif désigne un fichier existant : une image de `media/`, ou — pour un conteneur (§1.8) ou une section (§1.10) — une image d'un contenu qu'il regroupe ; une URL ou un chemin absolu au site n'est pas vérifié
- [ ] `media/` ne contient pas de sous-dossiers
- [ ] Chaque fichier de `media/` est classé (image, original ou document joint, §1.9)
- [ ] `cover` pointe vers une image (jamais un document joint ni un original)
- [ ] Chaque entrée `attachments:` du frontmatter référence un fichier existant de `media/` qui n'est pas un original
- [ ] `iptc.country_code` est un code ISO 3166-1 valide (si présent)
- [ ] `iptc.gps.lat` est entre -90 et 90, `lng` entre -180 et 180 (si présent)
- [ ] Pas de mélange mode local / mode distant / manifeste dans la même série
- [ ] `images.json` (si présent) est un JSON valide dont la clé `images` est un tableau (§1.5.1)
- [ ] Aucune série imbriquée au-delà d'un niveau — un conteneur §1.8 n'est jamais lui-même une sous-série (la profondeur de rangement §1.2, elle, n'est pas contrainte)
- [ ] Aucun dossier nommé `page` directement dans un dossier de série — segment réservé à la pagination (§1.6)
- [ ] Un `index.md` à la racine du corpus déclare `type: section` (§1.10)
- [ ] Chaque entrée `embeds:` porte une `url` (§1.11)
- [ ] Chaque champ d'URL (`cover`, `images[].url`, `files[].url`, `embeds[].url`, `embeds[].poster`) a une forme admise : chemin relatif, chemin absolu au site ou `http:`/`https:` — jamais `javascript:`, `data:` ni `file:` (§1.12) ; `embeds[].url` est en `http:`/`https:`
- [ ] Chaque `embeds[].poster` en chemin relatif pointe vers un fichier existant de `media/`
- [ ] Aucun `poster` d'embed n'est aussi listé dans `images:` ou `attachments:` — une image est une photo de la série ou la vignette d'un embed, pas les deux

Les outils d'ingestion (couche 4) expriment la plupart de ces vérifications dans un vocabulaire de diagnostics normalisé — code, sévérité, chemin — avec des règles d'évaluation exactes et des fixtures de conformité : voir §4.10 et `fixtures/ingestion/`. Un linter PEUT adopter ce vocabulaire.

### B — Migration depuis la spec v1 (Astro-only)

Pour migrer un projet utilisant la spec v1 (Astro plugin) :

1. **Frontmatter** : déplacer `camera` sous `iptc.camera` ; les anciens `tags` qui étaient des mots-clés IPTC passent sous `iptc.keywords`, ceux qui étaient éditoriaux restent dans `tags`, champ du core depuis la v2.1 (§1.3)
2. **Structure** : aucun changement (déjà compatible)
3. **Imports** : `hyperfocale/components` → `hyperfocale/astro/components`
4. **Config** : ajouter le préfixe du package (`hyperfocale/astro`)

### C — Correspondance IPTC ↔ EXIF

Pour les adaptateurs qui extraient des métadonnées EXIF des images :

| Champ Hyperfocale | IPTC standard | Champ EXIF |
|-------------------|---------------|------------|
| `iptc.creator` | Creator | Artist |
| `iptc.copyright` | Copyright Notice | Copyright |
| `iptc.keywords` | Keywords | XMP:Subject |
| `iptc.camera` | — | Model |
| `iptc.lens` | — | LensModel |
| `iptc.city` | City | — |
| `iptc.country` | Country | — |
| `iptc.gps` | — | GPSLatitude + GPSLongitude |

### D — Thème et CSS

Le format ne prescrit pas de design. Les adaptateurs qui fournissent un thème DEVRAIENT exposer des CSS custom properties pour la personnalisation :

```css
:root {
  --hf-color-bg: #ffffff;
  --hf-color-text: #111111;
  --hf-color-accent: #0066ff;
  --hf-font-sans: system-ui, sans-serif;
  --hf-gallery-gap: 0.5rem;
  --hf-card-radius: 4px;
  --hf-lightbox-bg: rgba(0, 0, 0, 0.95);
}
```

### E — Flux RSS / JSON Feed

Les adaptateurs web DEVRAIENT exposer un flux de syndication :

| Format | URL recommandée | Contenu |
|--------|----------------|---------|
| RSS 2.0 | `/<prefix>/feed.xml` | Dernières séries, cover en enclosure |
| JSON Feed | `/<prefix>/feed.json` | Dernières séries, format JSON Feed 1.1 |
| Sitemap | `/sitemap.xml` | Toutes les séries (excluant drafts) |

### F — Internationalisation

Pour les sites multilingues, quatre stratégies sont supportées. Chaque projet choisit **une** stratégie et s'y tient — pas de mélange dans un même projet.

**Stratégie 1 — Préfixe de langue dans le chemin** :
```
content/series/fr/bretagne-2024/index.md
content/series/en/brittany-2024/index.md
```

Slug peut différer entre locales (`bretagne-2024` ↔ `brittany-2024`). Sitemap et hreflang à gérer manuellement.

**Stratégie 2 — Fichier `index.<lang>.md` dans le même dossier** :
```
content/series/bretagne-2024/
├── index.md         (lang: fr — défaut)
├── index.en.md      (lang: en)
└── media/
```

Slug commun, médias partagés, traductions côte à côte. Idéal quand la photo est la même mais le texte change.

**Stratégie 3 — Collections séparées par locale** *(officialisée v2.1)* :
```
content/series/<slug>/         (collection canonique)
content/series_fr/<slug>/      (collection FR)
content/series_es/<slug>/      (collection ES)
content/store/<slug>/
content/store_fr/<slug>/
```

Chaque locale est une collection à part avec son propre loader. Permet des slugs traduits indépendants, des hierarchies de routage distinctes (`/archives/...` en EN, `/fr/archives/...` en FR), et un travail i18n en parallèle sans collision. Adapté aux gros corpus multilingues (cf. mathieu-drouet.com avec 5 locales).

**Stratégie 4 — Bloc `translations:` dans le frontmatter** :

Documentée en §2.7 (Dual naming mode et bloc `translations:`). Utile quand l'exporter Lightroom est la source de contenu et qu'on veut un seul fichier par série multilingue.

#### Quelle stratégie choisir ?

| Cas | Stratégie recommandée |
|-----|----------------------|
| Site monolingue avec quelques traductions ponctuelles | 2 |
| Site bilingue équilibré, contenu très synchronisé | 2 ou 4 |
| Site multilingue (3+ locales), routage différencié, gros corpus | 3 |
| Slugs distincts par locale, séparation forte des contenus | 1 |
| Pipeline LR → site direct avec traductions automatisées | 4 |

L'adaptateur DOIT documenter quelle(s) stratégie(s) il supporte. Un adaptateur PEUT en supporter plusieurs (le plugin Astro `@regrets/hyperfocale` supporte 2 et 3).

### G — Profils de contenu (presets standardisés) *(officialisé v2.3)*

La section 0 décrit un **squelette universel** : un dossier autonome = `index.md` (frontmatter core + body Markdown) + `media/`. Jusqu'à la v2.2, ce squelette n'avait qu'une seule déclinaison : la **série photo**. Cette annexe officialise l'idée qu'un même squelette peut porter **d'autres types de contenu**.

> **Extensions hors noyau *(2.14)*.** Hyperfocale est une spec photo pour le web : son noyau est la série photo et son bloc `iptc:`. Les profils de cette annexe sont des **extensions** : ils restent normatifs pour qui les implémente, mais aucun adaptateur n'a à les supporter pour être conforme au noyau (§2.0). Le travail à la source n'est pas un profil : il relève d'Hyperfocus.

Un **profil de contenu** est une déclinaison du squelette pour un domaine donné. Il n'invente rien au niveau structurel — il se contente de :

1. **Réutiliser le core tel quel** (`title`, `date`, `description`, `cover`, `location`, `draft`, `lang`, `tags`, `featured`, `type`).
2. **Ajouter un bloc d'extension namespacé** propre au domaine (exactement comme `iptc:` l'est pour la photo) : `event:`, `recipe:`, `app:`.
3. **Optionnellement, rendre `date` optionnelle** pour la collection du profil, par `dateRequired: false` dans son preset (§2.0.1, §1.3).

#### Règles communes à tous les profils

Ces règles garantissent qu'un profil reste un contenu Hyperfocale valide au sens de la §0.

| Règle | Description |
|-------|-------------|
| **Squelette inchangé** | `<slug>/index.md` + `media/` dès que le contenu a des fichiers locaux (§0, §1.2). Aucun profil ne modifie la structure filesystem ni le slug regex. |
| **Core préservé** | Les champs core ne sont jamais renommés. `title` reste obligatoire ; `date` peut devenir optionnelle pour toute la collection du profil (`dateRequired: false`, §1.3) — un relâchement que le contrat prévoit, pas la suppression d'une obligation (§2.0.1). |
| **Extension namespacée** | Tout vocabulaire de domaine vit sous **une** clé unique (`event`, `recipe`, `app`) pour ne jamais entrer en collision avec le core. |
| **Invariants §0 respectés** | Body avant galerie, tri alphabétique de `media/`, `cover`/fallback, `draft`, passthrough des champs inconnus s'appliquent identiquement. |
| **Rétro-compatibilité** | Un adaptateur qui ne connaît pas un profil DOIT lire le contenu comme un contenu Hyperfocale standard (core + media + body) et ignorer le bloc d'extension sans erreur (passthrough §1.3). Aucun profil ne casse un lecteur existant. |
| **Alignement externe** | Chaque profil DEVRAIT s'aligner sur un vocabulaire externe reconnu (schema.org), comme la photo s'aligne sur IPTC, pour l'interop et le balisage SEO. |
| **Ordre de tri propre** *(v2.15)* | Un profil PEUT définir un ordre canonique alternatif au tri par date, à documenter dans son preset (`screen.order`, G.6 ; les pistes de `music.tracks`, G.8). C'est une dérogation à §1.6, pas aux invariants de §0. |
| **Fichiers désignés** *(v2.15)* | Un champ d'extension qui désigne un fichier de `media/` (`catalog.datasheet`, `press.clipping`) s'écrit comme un nom de fichier ou comme `./media/<nom>` ; un adaptateur résout les deux. La couche 4 ne valide pas ces champs. |
| **Vocabulaires fermés** *(v2.15)* | Un profil nouveau écrit ses valeurs fermées en slugs ASCII (minuscules, sans accent, `_` ou `-`). Les vocabulaires déjà publiés — dont les valeurs accentuées de `recipe` — sont conservés : un lecteur les accepte tels quels. |
| **Sens de `media/`** | `media/` garde son rôle : un dossier plat de médias liés au contenu (affiches, photos du plat, captures d'écran, originaux d'un dossier de travail...). |

#### Vue d'ensemble

| Profil | Atome | Preset | Prefix recommandé | `date` | Bloc d'extension | Vocab externe |
|--------|-------|--------|-------------------|--------|------------------|---------------|
| Série (canonique) | série photo | `series` | `/series` | requise | `iptc:` | IPTC |
| Événement | un événement | `event` | `/events` | requise (= début) | `event:` | schema.org/Event |
| Recette | une recette | `recipe` | `/recipes` | optionnelle | `recipe:` | schema.org/Recipe |
| Application | une app | `app` | `/apps` | optionnelle (= sortie) | `app:` | schema.org/SoftwareApplication |
| Livre | un livre | `book` | `/books` | optionnelle | `book:` | schema.org/Book |
| Lieu | un lieu | `place` | `/places` | optionnelle | `place:` | schema.org/Place |
| Écran | un écran | `screen` | `/screens` | optionnelle | `screen:` | schema.org/HowToStep |
| Portfolio | un projet | `portfolio` | `/portfolio` | optionnelle (= livraison) | `portfolio:` | schema.org/CreativeWork |
| Musique | une sortie | `music` | `/music` | optionnelle (= sortie) | `music:` | schema.org/MusicAlbum |
| Catalogue | un produit | `catalog` | `/catalog` | optionnelle | `catalog:` | schema.org/Product |
| Presse | une parution | `press` | `/press` | requise (= parution) | `press:` | schema.org/NewsArticle |
| ~~Projet~~ | *passé dans Hyperfocus (G.12)* | — | — | — | — | — |

---

### G.1 — Profil Événement (`event`)

**Cas d'usage** : agenda, dates de concerts, expositions, conférences, vernissages, ateliers. L'atome est **un événement** : une occurrence datée, située, avec un lien d'inscription ou de billetterie.

#### Mapping du core

| Champ core | Sens dans le profil |
|------------|---------------------|
| `title` | Nom de l'événement |
| `date` | Date (jour) de **début** — sert au tri. La précision horaire vit dans `event.start`. |
| `description` | Accroche / résumé |
| `cover` | Affiche de l'événement (fallback : première image de `media/`) |
| `location` | Lieu en texte libre (le lieu structuré vit dans `event.*`) |
| `media/` | Affiche(s), plan d'accès, photos du lieu ou des éditions précédentes |

#### Bloc d'extension `event:`

Tous les champs sont optionnels. Un adaptateur DOIT les ignorer sans erreur s'il ne les supporte pas.

| Clé | Type | Description |
|-----|------|-------------|
| `start` | `datetime` (ISO 8601) | Début, avec heure et fuseau si pertinent (`2026-09-12T20:30:00+02:00`) |
| `end` | `datetime` (ISO 8601) | Fin |
| `timezone` | `string` | Fuseau IANA (`Europe/Paris`) si `start`/`end` sont sans offset |
| `status` | `string` | `scheduled` (défaut) · `rescheduled` · `postponed` · `cancelled` · `sold_out` |
| `attendance` | `string` | `offline` · `online` · `hybrid` |
| `venue` | `string` | Nom du lieu (« La Brat Cave ») |
| `address` | `string` | Adresse postale |
| `city` | `string` | Ville |
| `country` | `string` | Pays |
| `country_code` | `string` | Code ISO 3166-1 alpha-2 |
| `gps` | `object` | `{ lat, lng }` — réutilise la convention §1.3 |
| `organizer` | `string` | Organisateur |
| `performers` | `string[]` | Artistes / intervenants (utile pour le line-up, cf. §1.8) |
| `url` | `string` (URL) | Page d'inscription / billetterie |
| `price` | `string` | Tarif en texte libre (« 12 € » · « Gratuit » · « Prix libre ») |
| `currency` | `string` | Code ISO 4217 si `price` est numérique ailleurs |
| `recurrence` | `string` | Règle de récurrence façon iCal RRULE (`FREQ=WEEKLY;BYDAY=TH`) |

> **Synergie avec les séries imbriquées (§1.8).** Un festival est naturellement un **événement conteneur** : son `index.md` porte le profil `event`, et chaque sous-dossier est soit une sous-série photo, soit un sous-événement (un set d'artiste). `event.performers` côté conteneur et le `lineup_order` des sous-séries se complètent.

#### Exemple

```yaml
---
title: "Daimonion Fest #28"
date: 2026-09-12
description: "Nuit black/doom à Faches-Thumesnil"
cover: "./media/affiche.jpg"
location: "Faches-Thumesnil, France"
draft: false
lang: fr
tags: [concert, metal, lille]

event:
  start: 2026-09-12T19:00:00+02:00
  end: 2026-09-13T02:00:00+02:00
  timezone: Europe/Paris
  status: scheduled
  attendance: offline
  venue: "La Cave aux Poètes"
  address: "4 Rue de l'Église"
  city: "Faches-Thumesnil"
  country: "France"
  country_code: FR
  gps: { lat: 50.59, lng: 3.07 }
  organizer: "Daimonion Prod"
  performers: ["Cantique Lépreux", "Sortilegia", "Sühnopfer"]
  url: "https://billetterie.example.com/daimonion-28"
  price: "18 € prévente / 22 € sur place"
---

Programme de la soirée, conditions d'accès, plan. Le body Markdown
s'affiche **avant** la galerie (affiche + photos du lieu).
```

#### Preset & règles métier spécifiques

```ts
hyperfocale({ preset: 'event' })  // collection 'events', prefix /events, dateRequired: true
```

| Règle | Description |
|-------|-------------|
| Tri | Par défaut date décroissante (§1.6). Un adaptateur « agenda » PEUT exposer un tri **ascendant filtré sur les événements à venir** (`start >= now`). |
| Statut | Les adaptateurs DEVRAIENT signaler visuellement `cancelled` / `postponed` / `sold_out`. Un événement annulé reste publié (≠ `draft`). |
| Balisage | Un adaptateur web DEVRAIT émettre du JSON-LD `schema.org/Event` depuis `event.*`. |

---

### G.2 — Profil Recette (`recipe`)

**Cas d'usage** : carnet de recettes, blog culinaire. L'atome est **une recette**. C'est le profil le plus structuré : ingrédients et étapes méritent un balisage machine.

#### Mapping du core

| Champ core | Sens dans le profil |
|------------|---------------------|
| `title` | Nom de la recette |
| `date` | Date de publication — **optionnelle** (une recette est intemporelle). Preset `dateRequired: false`. |
| `description` | Présentation / histoire courte du plat |
| `cover` | Photo du plat fini |
| `media/` | Photo du plat fini + photos d'étapes |

#### Bloc d'extension `recipe:`

| Clé | Type | Description |
|-----|------|-------------|
| `servings` | `number` ou `string` | Nombre de portions (« 4 personnes ») |
| `yield` | `string` | Rendement alternatif (« 12 cookies », « 1 cake ») |
| `prep_time` | `duration` (ISO 8601) | Préparation (`PT20M`) |
| `cook_time` | `duration` (ISO 8601) | Cuisson (`PT45M`) |
| `rest_time` | `duration` (ISO 8601) | Repos / pousse |
| `total_time` | `duration` (ISO 8601) | Total (sinon dérivable) |
| `difficulty` | `string` | `facile` · `moyen` · `difficile` |
| `cuisine` | `string` | Type de cuisine (« italienne », « bretonne ») |
| `course` | `string` | `entrée` · `plat` · `dessert` · `boisson` · `accompagnement` |
| `diet` | `string[]` | `vegan` · `végétarien` · `sans-gluten` · `sans-lactose`... |
| `ingredients` | `list` | Liste structurée (voir ci-dessous) |
| `steps` | `string[]` | Étapes ordonnées (Markdown autorisé par étape) |
| `equipment` | `string[]` | Matériel requis |
| `nutrition` | `object` | `{ calories, protein, carbs, fat, ... }` par portion |
| `source` | `string` | Origine / inspiration de la recette |

##### Ingrédients structurés

`ingredients` est soit une liste plate, soit groupée par section. Chaque entrée : `item` (requis), `qty`, `unit`, `note` (optionnels).

```yaml
recipe:
  ingredients:
    - group: "Pâte"
      items:
        - { item: "Farine T55", qty: 250, unit: "g" }
        - { item: "Beurre doux", qty: 125, unit: "g", note: "froid, en dés" }
        - { item: "Eau", qty: 60, unit: "ml" }
    - group: "Garniture"
      items:
        - { item: "Pommes", qty: 4, unit: "pièces" }
        - { item: "Sucre", qty: 2, unit: "c. à soupe" }
```

> **Structuré vs body.** Le bloc `recipe.ingredients` / `recipe.steps` est la source **machine** (balisage, listes de courses, conversion de portions). Le **body Markdown** reste libre pour la narration. Un adaptateur PEUT générer l'affichage des ingrédients/étapes depuis le frontmatter, ou laisser l'auteur les écrire dans le body — les deux sont valides, mais le frontmatter structuré est recommandé pour l'interop.

#### Exemple

```yaml
---
title: "Tarte aux pommes"
description: "La tarte du dimanche, pâte brisée maison"
cover: "./media/01.jpg"
tags: [dessert, classique, pommes]

recipe:
  servings: 6
  prep_time: PT30M
  cook_time: PT40M
  total_time: PT1H10M
  difficulty: facile
  cuisine: française
  course: dessert
  diet: [végétarien]
  equipment: ["Moule à tarte 28 cm", "Rouleau à pâtisserie"]
  ingredients:
    - { item: "Pâte brisée", qty: 1, unit: "rouleau" }
    - { item: "Pommes", qty: 5, unit: "pièces" }
    - { item: "Sucre", qty: 50, unit: "g" }
    - { item: "Beurre", qty: 30, unit: "g" }
  steps:
    - "Préchauffer le four à 180 °C."
    - "Étaler la pâte dans le moule, piquer le fond."
    - "Disposer les pommes en rosace, saupoudrer de sucre."
    - "Parsemer de beurre, enfourner 40 min."
---

Une note perso sur la recette, son origine, les variantes possibles.
Le body s'affiche **avant** les photos d'étapes.
```

#### Preset & règles métier spécifiques

```ts
hyperfocale({ preset: 'recipe' })  // collection 'recipes', prefix /recipes, dateRequired: false
```

| Règle | Description |
|-------|-------------|
| Tri | `date` étant optionnelle, un adaptateur PEUT trier par `title` ou par `featured` si `date` est absente. |
| Conversion de portions | Un adaptateur PEUT recalculer `qty` au prorata de `servings` (extension UI, hors spec). |
| Balisage | Un adaptateur web DEVRAIT émettre du JSON-LD `schema.org/Recipe` (éligible aux rich results Google). |

> **Note historique.** Une variante recettes existait déjà hors source canonique (`Recipes-hyperfocale/docs/spec-hyperfocale-v2.md`, cf. Changelog 2.1). Ce profil G.2 en est la version officielle et alignée sur le squelette générique — la copie projet devrait converger vers ce vocabulaire.

---

### G.3 — Profil Application (`app`)

**Cas d'usage** : portfolio d'apps, page produit logiciel, showcase de projets. L'atome est **une application** (ou un produit logiciel). `media/` porte les captures d'écran et l'icône.

#### Mapping du core

| Champ core | Sens dans le profil |
|------------|---------------------|
| `title` | Nom de l'application |
| `date` | Date de sortie / lancement — **optionnelle** (preset `dateRequired: false`) |
| `description` | Pitch / tagline |
| `cover` | Hero (capture principale) ou icône (fallback : première image) |
| `media/` | Captures d'écran, icône, mockups |

#### Bloc d'extension `app:`

| Clé | Type | Description |
|-----|------|-------------|
| `platform` | `string[]` | `ios` · `android` · `web` · `macos` · `windows` · `linux` |
| `version` | `string` | Version courante (semver, « 1.4.0 ») |
| `status` | `string` | `live` · `beta` · `alpha` · `wip` · `sunset` |
| `category` | `string` | Catégorie (« Productivité », « Photo ») |
| `pricing` | `string` | `free` · `paid` · `freemium` · `subscription` |
| `price` | `string` | Prix en texte libre (« 4,99 € », « Gratuit ») |
| `url` | `string` (URL) | Site officiel / landing page |
| `store` | `object` | Liens stores : `{ app_store, play_store, web, ... }` |
| `repository` | `string` (URL) | Dépôt source (si open source) |
| `tech` | `string[]` | Stack technique (« Swift », « Astro », « Rust ») |
| `developer` | `string` | Éditeur / développeur |
| `license` | `string` | Licence (SPDX si open source : `MIT`, `GPL-3.0`) |

#### Exemple

```yaml
---
title: "Hyperfocale Exporter"
date: 2026-03-01
description: "Export Lightroom → format Hyperfocale, en un clic"
cover: "./media/hero.png"
featured: true
tags: [lightroom, photo, export]

app:
  platform: [macos]
  version: "0.1.0"
  status: beta
  category: "Photo"
  pricing: free
  url: "https://github.com/izo/hyperfocale-exporter-app"
  store:
    web: "https://github.com/izo/hyperfocale-exporter-app/releases"
  repository: "https://github.com/izo/hyperfocale-exporter-app"
  tech: [SwiftUI]
  developer: "Mathieu Drouet"
  license: MIT
---

Présentation de l'app, fonctionnalités clés, captures d'écran.
Le body s'affiche **avant** la galerie de screenshots.
```

#### Preset & règles métier spécifiques

```ts
hyperfocale({ preset: 'app' })  // collection 'apps', prefix /apps, dateRequired: false
```

| Règle | Description |
|-------|-------------|
| Tri | Date desc si présente, sinon `featured` puis `title`. |
| Statut | `sunset` DEVRAIT être signalé visuellement ; l'app reste publiée (≠ `draft`). |
| Balisage | Un adaptateur web DEVRAIT émettre du JSON-LD `schema.org/SoftwareApplication` depuis `app.*`. |

---

### G.4 — Profil Livre (`book`)

**Cas d'usage** : bibliothèque, fiches de lecture, catalogue d'éditions, suivi de lectures. L'atome est **un livre** (une œuvre ou une édition précise).

#### Mapping du core

| Champ core | Sens dans le profil |
|------------|---------------------|
| `title` | Titre du livre |
| `date` | Date de **lecture** ou d'ajout à la bibliothèque — **optionnelle** (preset `dateRequired: false`). La date de parution vit dans `book.published`. |
| `description` | Résumé / quatrième de couverture / avis |
| `cover` | Couverture du livre |
| `location` | Sans objet en général (peut servir au lieu d'achat / bibliothèque physique) |
| `media/` | Couverture, pages scannées, photos de l'exemplaire |

#### Bloc d'extension `book:`

| Clé | Type | Description |
|-----|------|-------------|
| `authors` | `string[]` | Auteur(s) |
| `translators` | `string[]` | Traducteur(s) |
| `isbn` | `string` | ISBN-13 (ou ISBN-10) |
| `publisher` | `string` | Éditeur |
| `published` | `date` (ISO 8601) | Date de parution de l'édition |
| `language` | `string` | Langue de l'ouvrage (ISO 639-1) |
| `original_language` | `string` | Langue d'origine (si traduction) |
| `pages` | `number` | Nombre de pages |
| `format` | `string` | `broché` · `relié` · `poche` · `ebook` · `audio` |
| `genre` | `string[]` | Genres littéraires |
| `series` | `string` | Série / cycle littéraire (ex : « Le Trône de fer ») |
| `series_index` | `number` | Position dans la série |
| `status` | `string` | `to_read` · `reading` · `read` · `abandoned` |
| `rating` | `number` | Note de lecture (échelle libre, ex : 0–5) |
| `read_date` | `date` (ISO 8601) | Date de fin de lecture |
| `url` | `string` (URL) | Lien éditeur / achat / fiche |

> **`book.series` ≠ série photo.** Le champ `series` est ici la **collection littéraire** d'un livre ; il est namespacé sous `book.` et n'a aucun rapport avec la série photo canonique. Aucune collision possible.

#### Exemple

```yaml
---
title: "Le Nom de la rose"
date: 2026-04-10
description: "Relecture annuelle. Toujours aussi dense."
cover: "./media/couverture.jpg"
tags: [roman, médiéval, polar]

book:
  authors: ["Umberto Eco"]
  translators: ["Jean-Noël Schifano"]
  isbn: "9782253033134"
  publisher: "Grasset"
  published: 1982-01-01
  language: fr
  original_language: it
  pages: 640
  format: poche
  genre: [roman, policier, historique]
  status: read
  rating: 5
  read_date: 2026-04-10
  url: "https://www.grasset.fr/livres/le-nom-de-la-rose"
---

Notes de lecture, citations marquantes, contexte. Le body s'affiche
**avant** la galerie (couverture, pages photographiées).
```

#### Preset & règles métier spécifiques

```ts
hyperfocale({ preset: 'book' })  // collection 'books', prefix /books, dateRequired: false
```

| Règle | Description |
|-------|-------------|
| Tri | `date` (lecture) desc si présente, sinon `book.published` desc, sinon `title`. |
| Statut | Un adaptateur « bibliothèque » PEUT filtrer/grouper par `book.status` (à lire / en cours / lu). |
| Balisage | Un adaptateur web DEVRAIT émettre du JSON-LD `schema.org/Book` depuis `book.*`. |

---

### G.5 — Profil Lieu (`place`)

**Cas d'usage** : guide d'adresses, carnet de lieux, points d'intérêt (POI), repérages photo, sélection de restaurants / musées / cafés. L'atome est **un lieu**.

#### Mapping du core

| Champ core | Sens dans le profil |
|------------|---------------------|
| `title` | Nom du lieu |
| `date` | Date de visite / découverte — **optionnelle** (preset `dateRequired: false`) |
| `description` | Présentation / avis |
| `cover` | Photo principale du lieu |
| `location` | Adresse en texte libre (le détail structuré vit dans `place.*`) |
| `media/` | Photos du lieu |

#### Bloc d'extension `place:`

| Clé | Type | Description |
|-----|------|-------------|
| `category` | `string` | Type de lieu (« restaurant », « musée », « café », « parc », « hôtel ») |
| `address` | `string` | Adresse postale |
| `city` | `string` | Ville |
| `province` | `string` | Région / État / Province |
| `country` | `string` | Pays |
| `country_code` | `string` | Code ISO 3166-1 alpha-2 |
| `gps` | `object` | `{ lat, lng }` — **réutilise la convention §1.3** |
| `url` | `string` (URL) | Site web |
| `phone` | `string` | Téléphone |
| `hours` | `string` | Horaires d'ouverture (texte libre ou OSM `opening_hours`) |
| `price_range` | `string` | Gamme de prix (`$` · `$$` · `$$$` · `$$$$`) |
| `rating` | `number` | Note personnelle (échelle libre) |
| `amenities` | `string[]` | Équipements / commodités (« terrasse », « wifi », « accessible PMR ») |
| `status` | `string` | `open` · `closed` (temporaire) · `permanently_closed` |

> **Synergie carte (§3.1).** `place.gps` réutilise exactement la convention de `iptc.gps`. Le composant `SeriesMap` (et la vue Carte du plugin Obsidian, §2.5) peut donc cartographier les contenus `place` sans adaptation, en filtrant sur la présence de `place.gps`.

#### Exemple

```yaml
---
title: "Café de la Presse"
date: 2026-05-22
description: "Bon petit déj, calme le matin, terrasse au sud."
cover: "./media/01.jpg"
location: "12 rue de la Monnaie, Lille"
tags: [café, lille, petit-déjeuner]

place:
  category: café
  address: "12 rue de la Monnaie"
  city: "Lille"
  country: "France"
  country_code: FR
  gps: { lat: 50.64, lng: 3.06 }
  url: "https://example.com/cafe-presse"
  phone: "+33 3 20 00 00 00"
  hours: "Mar-Dim 08:00-18:00"
  price_range: "$$"
  rating: 4
  amenities: ["terrasse", "wifi"]
  status: open
---

Avis détaillé, ce qu'il faut commander, le bon moment pour venir.
Le body s'affiche **avant** la galerie de photos du lieu.
```

#### Preset & règles métier spécifiques

```ts
hyperfocale({ preset: 'place' })  // collection 'places', prefix /places, dateRequired: false
```

| Règle | Description |
|-------|-------------|
| Tri | `date` (visite) desc si présente, sinon `title`. Un adaptateur PEUT trier/filtrer par `place.category`. |
| Carte | Réutiliser `SeriesMap` sur les lieux ayant `place.gps`. |
| Statut | `permanently_closed` DEVRAIT être signalé ; le lieu reste publié (≠ `draft`). |
| Balisage | Un adaptateur web DEVRAIT émettre du JSON-LD `schema.org/Place` (ou un sous-type `LocalBusiness`) depuis `place.*`. |

---

### G.6 — Profil Écran (`screen`)

**Cas d'usage** : expérience interactive narrative découpée en écrans/étapes — séquence de boot, environnements applicatifs, parcours onboarding, kiosque, démo produit pas-à-pas. L'atome est **un écran** (une étape de l'expérience). C'est le premier profil dont le contenu n'est pas une « collection d'objets » mais une **séquence ordonnée**.

> **Origine** : observé dans le projet `mdr-terminal-portfolio` (portfolio dual-persona simulant des OS rétro — boot BIOS → sélection persona → ascenseur → desktop). Le contenu de chaque écran y était dispersé et hardcodé ; le profil `screen` l'unifie au format Hyperfocale.

#### Mapping du core

| Champ core | Sens dans le profil |
|------------|---------------------|
| `title` | Nom de l'écran |
| `date` | **Optionnelle** — un écran est intemporel (preset `dateRequired: false`). La séquence est portée par `screen.order`, pas par la date. |
| `description` | Pitch court de l'écran |
| `cover` | Visuel / capture de l'écran (fallback : première image de `media/`) |
| `location` | Sans objet en général |
| `media/` | Captures d'écran, assets visuels de l'écran |

#### Bloc d'extension `screen:`

`order` et `kind` sont **requis** (squelette de séquence) ; tout le reste est optionnel.

| Clé | Type | Description |
|-----|------|-------------|
| `order` | `number` | **Requis.** Position dans la séquence globale |
| `kind` | `string` | **Requis.** Nature de l'écran. Vocabulaire suggéré : `boot` · `persona` · `app` · `terminal` · `step` · `easter-egg` (libre selon le domaine) |
| `stage_id` | `string` | Identifiant technique reliant l'écran au code (enum d'états, id d'app) |
| `persona` | `string` | Identité / rôle associé à l'écran, si l'expérience est multi-persona |
| `theme` | `string` | Thème visuel actif sur l'écran |
| `duration` | `duration` (ISO 8601) | Durée d'affichage des écrans temporisés (`PT2.5S`) |
| `skippable` | `boolean` | L'utilisateur peut-il sauter cet écran ? Défaut `false` |
| `next` | `string[]` | Chemins des écrans suivants possibles (graphe de navigation) |
| `prev` | `string` | Chemin de l'écran précédent |
| `llm_context` | `string` | Chemin vers un contexte LLM lu par un assistant conversationnel quand l'écran est actif (base de connaissance contextuelle) |

> **`next` et `prev` sont des indications de navigation, pas des jointures.** Ils désignent un écran par son chemin (§1.2) ; un chemin qui ne désigne rien est ignoré, et aucune lecture du contenu n'en dépend. Le principe de §0 (« pas de relations entre séries ») reste vrai.

> **`screen` est une séquence, pas une collection.** Contrairement aux profils G.1–G.5 (tri par date desc), l'ordre canonique d'un contenu `screen` est `screen.order` ascendant. Le champ `next`/`prev` permet d'exprimer un graphe de navigation non-linéaire (branches, easter-eggs) au-delà de l'ordre plat.

#### Exemple

```yaml
---
title: "BIOS Screen"
description: "Écran BIOS rétro-corporate"
draft: false
lang: en
tags: [boot, bios]

screen:
  order: 1
  kind: boot
  stage_id: BIOS
  theme: lumon
  duration: PT8S
  skippable: true
  next: ["02-os9-boot"]
  llm_context: /personas/01-mathieu-d/llm.txt
---

Texte de l'écran (affiché **avant** ses captures). Le body Markdown
décrit le contenu narratif de l'étape.
```

#### Preset & règles métier spécifiques

```ts
hyperfocale({ preset: 'screen' })  // collection 'screens', prefix /screens, dateRequired: false
```

| Règle | Description |
|-------|-------------|
| Tri | **`screen.order` ascendant** (dérogation au tri date desc §1.6, justifiée par la nature séquentielle). `date` absente n'empêche pas l'ordonnancement. |
| Navigation | Un adaptateur PEUT exposer la navigation `next`/`prev` comme un parcours guidé (boutons précédent/suivant, deep-linking par slug). |
| Balisage | Un adaptateur web DEVRAIT émettre du JSON-LD `schema.org/HowToStep` pour les écrans séquencés (`kind: boot`/`step`), ou `schema.org/WebPageElement` pour les écrans-pages. |
| Assistant LLM | `screen.llm_context` est un pointeur de contexte ; l'adaptateur PEUT le charger comme system prompt / base de connaissance d'un chat contextuel. Champ optionnel, ignoré sans erreur. |

---

### G.7 — Profil Portfolio (`portfolio`)

**Cas d'usage** : book de créatif, agence, studio, indépendant — présenter des réalisations livrées. L'atome est **un projet** : une réalisation, pour un commanditaire ou en propre, dont les visuels sont la trace.

> **Distinction avec la série canonique.** Une série documente un **corpus d'images** — les images *sont* le contenu. Un projet documente une **réalisation** dont les images sont la *preuve*. Un book de photographe reste donc une collection de séries ; un book de graphiste ou de développeur relève de `portfolio`.

#### Mapping du core

| Champ core | Sens dans le profil |
|------------|---------------------|
| `title` | Nom du projet |
| `date` | Date de livraison ou de mise en ligne — sert au tri. Optionnelle : un travail en cours ou un projet ancien non daté reste valide. |
| `description` | Accroche / pitch du projet |
| `cover` | Visuel principal (fallback : première image de `media/`) |
| `location` | Lieu de réalisation, si pertinent |
| `media/` | Visuels du projet : rendus, captures, photos de mise en situation, planches |

#### Bloc d'extension `portfolio:`

Tous les champs sont optionnels. Un adaptateur DOIT les ignorer sans erreur s'il ne les supporte pas.

| Clé | Type | Description |
|-----|------|-------------|
| `client` | `string` | Commanditaire |
| `role` | `string[]` | Rôles tenus (« direction artistique », « développement », « photographie ») |
| `discipline` | `string[]` | Champs d'intervention (« identité visuelle », « web », « édition », « motion ») |
| `team` | `string[]` | Collaborateurs, studio, agence |
| `year` | `number` | Année, quand `date` est absente et que seule l'année est connue |
| `status` | `string` | `delivered` (défaut) · `wip` · `concept` · `archived` |
| `url` | `string` (URL) | Projet en ligne |
| `repository` | `string` (URL) | Dépôt public, le cas échéant |
| `awards` | `string[]` | Distinctions |
| `tools` | `string[]` | Outils et technologies |
| `duration` | `string` | Durée en texte libre (« 6 semaines ») |
| `confidential` | `boolean` | Si `true`, l'adaptateur DEVRAIT masquer `client` au rendu |

#### Exemple

```yaml
---
title: "Identité visuelle — Librairie Ptyx"
date: 2026-03-14
description: "Refonte complète : logotype, signalétique, site"
cover: "./media/logotype.jpg"
draft: false
lang: fr
tags: [identite, edition]

portfolio:
  client: "Librairie Ptyx"
  role: ["direction artistique", "design graphique"]
  discipline: ["identité visuelle", "signalétique"]
  team: ["Studio Corbeau"]
  status: delivered
  url: "https://ptyx.example.com"
  tools: ["Illustrator", "Astro"]
  duration: "6 semaines"
---

Contexte, contraintes, parti pris. Le body s'affiche **avant** la galerie.
```

#### Preset & règles métier spécifiques

```ts
hyperfocale({ preset: 'portfolio' })  // collection 'portfolio', prefix /portfolio, dateRequired: false
```

| Règle | Description |
|-------|-------------|
| Tri | Date décroissante (§1.6). Les projets sans `date` se rangent après les projets datés, `portfolio.year` servant de clé secondaire. |
| Statut | Un adaptateur DEVRAIT distinguer visuellement `wip` et `concept` d'un projet `delivered`. Un projet `archived` reste publié (≠ `draft`). |
| Confidentialité | Si `confidential: true`, `portfolio.client` NE DOIT PAS être rendu ni émis dans le balisage. |
| Balisage | Un adaptateur web DEVRAIT émettre du JSON-LD `schema.org/CreativeWork` depuis `portfolio.*`. |

---

### G.8 — Profil Musique (`music`)

**Cas d'usage** : discographie d'artiste, catalogue de label, page de sortie. L'atome est **une sortie** : un album, un EP, un single ou une compilation.

#### Mapping du core

| Champ core | Sens dans le profil |
|------------|---------------------|
| `title` | Titre de la sortie |
| `date` | Date de sortie — sert au tri. Optionnelle : démos et sorties non datées restent valides. |
| `description` | Note d'intention, présentation |
| `cover` | Pochette (fallback : première image de `media/`) |
| `location` | Lieu d'enregistrement, si pertinent |
| `media/` | Pochette, verso, photos de studio, pages de livret |

#### Bloc d'extension `music:`

Tous les champs sont optionnels.

| Clé | Type | Description |
|-----|------|-------------|
| `artist` | `string` | Artiste principal |
| `album_type` | `string` | `album` · `ep` · `single` · `compilation` · `live` · `demo` |
| `label` | `string` | Label |
| `catalog_number` | `string` | Référence catalogue du label |
| `release_date` | `date` | Date de sortie, si `date` porte autre chose (réédition, enregistrement) |
| `formats` | `string[]` | `vinyl` · `cd` · `cassette` · `digital` |
| `genre` | `string[]` | Genres |
| `tracks` | `object[]` | `{ position, title, duration?, isrc? }` — voir la règle de tri |
| `duration` | `string` | Durée totale, ISO 8601 (`PT42M13S`) |
| `upc` | `string` | Code-barres |
| `credits` | `string[]` | Musiciens, production, mastering |
| `streaming` | `object` | URLs par plateforme (`{ bandcamp?, spotify?, apple? }`) |

> **Les pistes ne sont pas des contenus.** `music.tracks` est une liste de métadonnées, pas une arborescence : une sortie reste **un** dossier. Un coffret ou une intégrale qui justifie une page par disque relève du conteneur §1.8, chaque disque étant alors une sortie à part entière.

#### Exemple

```yaml
---
title: "Fragments"
date: 2026-05-02
description: "Second album, enregistré en deux sessions hivernales"
cover: "./media/pochette.jpg"
location: "Studio Nord, Lille"
lang: fr
tags: [ambient, drone]

music:
  artist: "Hélène Varn"
  album_type: album
  label: "Nord Records"
  catalog_number: "NR-042"
  formats: [vinyl, digital]
  genre: ["ambient", "drone"]
  duration: "PT42M13S"
  tracks:
    - { position: 1, title: "Seuil", duration: "PT7M04S" }
    - { position: 2, title: "Fragments", duration: "PT12M31S" }
  credits: ["Mastering : A. Rouvier"]
  streaming: { bandcamp: "https://helenevarn.bandcamp.com/album/fragments" }
---

Genèse du disque, intentions, matériel. Body avant galerie.
```

#### Preset & règles métier spécifiques

```ts
hyperfocale({ preset: 'music' })  // collection 'music', prefix /music, dateRequired: false
```

| Règle | Description |
|-------|-------------|
| Tri des sorties | Date décroissante (§1.6). `music.release_date` prime sur `date` s'il est présent. |
| Tri des pistes | `music.tracks` est rendu par `position` **ascendant**, jamais dans l'ordre du fichier ni alphabétique. |
| Balisage | Un adaptateur web DEVRAIT émettre du JSON-LD `schema.org/MusicAlbum`, chaque piste en `schema.org/MusicRecording`. |

---

### G.9 — Profil Catalogue (`catalog`)

**Cas d'usage** : catalogue de gamme, showroom, matériauthèque, fiches produit d'un fabricant. L'atome est **un produit**.

> **Hors périmètre.** Ce profil décrit un produit, pas une boutique : la spec ne définit ni panier, ni stock temps réel, ni paiement (§0 — « ce que cette spec NE définit PAS »). `availability` et `price` sont des métadonnées éditoriales, pas une source de vérité transactionnelle.

#### Mapping du core

| Champ core | Sens dans le profil |
|------------|---------------------|
| `title` | Nom du produit |
| `date` | Mise au catalogue — sert au tri. Optionnelle : un catalogue se range plus souvent par gamme que par date. |
| `description` | Accroche produit |
| `cover` | Visuel principal (fallback : première image de `media/`) |
| `location` | Lieu de fabrication, si revendiqué |
| `media/` | Photos produit, détails, plans, échantillons de matière, fiche technique (§1.9) |

#### Bloc d'extension `catalog:`

Tous les champs sont optionnels.

| Clé | Type | Description |
|-----|------|-------------|
| `sku` | `string` | Référence interne |
| `brand` | `string` | Marque |
| `category` | `string[]` | Catégories, du général au spécifique |
| `price` | `string` | Tarif en texte libre (« 240 € », « sur devis ») |
| `currency` | `string` | Code ISO 4217 |
| `availability` | `string` | `in_stock` · `out_of_stock` · `preorder` · `made_to_order` · `discontinued` |
| `materials` | `string[]` | Matériaux |
| `dimensions` | `object` | `{ width?, height?, depth?, unit }` |
| `weight` | `object` | `{ value, unit }` |
| `colors` | `string[]` | Coloris disponibles |
| `gtin` | `string` | Code-barres (EAN / UPC) |
| `url` | `string` (URL) | Fiche ou point de vente externe |
| `datasheet` | `string` | Nom d'un document joint de `media/` (§1.9) |

> **Synergie avec les documents joints (§1.9).** Une fiche technique PDF vit dans `media/` et se référence via `catalog.datasheet` ; elle est rendue par `<SeriesAttachments>` ou équivalent, après la galerie.

#### Exemple

```yaml
---
title: "Étagère Ligne — chêne massif"
description: "Étagère murale, trois plateaux, assemblage sans vis apparentes"
cover: "./media/etagere-face.jpg"
location: "Atelier de Roubaix"
lang: fr
tags: [mobilier, chene]

catalog:
  sku: "ETG-LIG-03"
  brand: "Atelier Ligne"
  category: ["mobilier", "rangement", "étagère"]
  price: "340 €"
  currency: EUR
  availability: made_to_order
  materials: ["chêne massif", "huile-cire"]
  dimensions: { width: 90, height: 32, depth: 22, unit: cm }
  weight: { value: 6.4, unit: kg }
  colors: ["naturel", "fumé"]
  datasheet: "fiche-technique.pdf"
---

Fabrication, finitions, entretien. Body avant galerie.
```

#### Preset & règles métier spécifiques

```ts
hyperfocale({ preset: 'catalog' })  // collection 'catalog', prefix /catalog, dateRequired: false
```

| Règle | Description |
|-------|-------------|
| Tri | Date décroissante par défaut (§1.6). Un adaptateur « catalogue » PEUT exposer un tri par `catalog.category` puis `title` alphabétique, plus pertinent pour une gamme. |
| Disponibilité | Un adaptateur DEVRAIT signaler `discontinued` et `out_of_stock`. Un produit arrêté reste publié (≠ `draft`) — le catalogue fait mémoire. |
| Prix | `catalog.price` est **informatif**. Un adaptateur NE DOIT PAS le présenter comme un prix transactionnel engageant. |
| Balisage | Un adaptateur web DEVRAIT émettre du JSON-LD `schema.org/Product`, `availability` mappé sur `schema.org/ItemAvailability`. |

---

### G.10 — Profil Presse (`press`)

**Cas d'usage** : revue de presse, retombées médias, communiqués. L'atome est **une parution** : un article publié dans un média, ou un communiqué émis.

#### Mapping du core

| Champ core | Sens dans le profil |
|------------|---------------------|
| `title` | Titre de la parution |
| `date` | Date de parution — **requise**, sert au tri. Une retombée sans date n'est pas exploitable. |
| `description` | Chapô ou résumé |
| `cover` | Une du média, capture de l'article (fallback : première image de `media/`) |
| `location` | Zone de diffusion, si pertinent |
| `media/` | Scans, captures d'écran, PDF de la parution (§1.9) |

#### Bloc d'extension `press:`

Tous les champs sont optionnels.

| Clé | Type | Description |
|-----|------|-------------|
| `publication` | `string` | Nom du média |
| `author` | `string` | Signature de l'article |
| `kind` | `string` | `article` · `interview` · `review` · `mention` · `broadcast` · `press_release` |
| `url` | `string` (URL) | Article en ligne |
| `archive_url` | `string` (URL) | Copie archivée, contre la disparition du lien |
| `issue` | `string` | Numéro ou édition |
| `page` | `string` | Pagination (« p. 34-36 ») |
| `language` | `string` | Langue de la parution, si différente de `lang` |
| `paywall` | `boolean` | L'article en ligne est-il payant |
| `excerpt` | `string` | Citation courte de la parution |
| `clipping` | `string` | Nom d'un document joint de `media/` (§1.9) |

> **Émis ou subi.** `kind: press_release` désigne un contenu **émis par soi** ; toutes les autres valeurs désignent une **retombée**, écrite par un tiers. Un adaptateur PEUT séparer les deux flux — ils ne se valent pas éditorialement.

#### Exemple

```yaml
---
title: "Le silence photographié"
date: 2026-04-18
description: "Portrait dans la rubrique Culture"
cover: "./media/une.jpg"
lang: fr
tags: [presse, portrait]

press:
  publication: "La Voix du Nord"
  author: "C. Delmas"
  kind: interview
  url: "https://lavoixdunord.example.com/le-silence-photographie"
  archive_url: "https://web.archive.org/web/2026/https://lavoixdunord.example.com/le-silence-photographie"
  issue: "n° 24 812"
  page: "p. 18"
  paywall: true
  excerpt: "Une écriture du vide qui refuse l'anecdote."
  clipping: "parution-2026-04-18.pdf"
---

Contexte de la parution. Body avant galerie.
```

#### Preset & règles métier spécifiques

```ts
hyperfocale({ preset: 'press' })  // collection 'press', prefix /press, dateRequired: true
```

| Règle | Description |
|-------|-------------|
| Tri | Date décroissante (§1.6) — la retombée la plus récente en premier. |
| Lien mort | Si `press.url` et `press.archive_url` sont tous deux présents, un adaptateur DEVRAIT exposer le second en secours. La presse en ligne disparaît. |
| Paywall | Un adaptateur DEVRAIT signaler `paywall: true` avant d'envoyer le lecteur sur un mur payant. |
| Citation | `press.excerpt` reste une **citation courte**, au sens du droit de citation. Un adaptateur NE DOIT PAS reproduire un article intégral dans le body sans droits. |
| Balisage | Un adaptateur web DEVRAIT émettre du JSON-LD `schema.org/NewsArticle` pour les retombées ; `press_release` relève de `schema.org/PressRelease`. |

---

### G.11 — Créer un nouveau profil

Pour proposer un profil supplémentaire (livre, lieu, produit e-commerce...), une PR contre cette annexe DOIT préciser :

1. **L'atome** et le cas d'usage.
2. **Le mapping du core** (sens de `title`/`date`/`cover`/`media`, `dateRequired` ou non).
3. **Le bloc d'extension namespacé** (clé unique, table de champs, tous optionnels — sauf les champs que le profil déclare requis explicitement, comme `screen.order` et `screen.kind`).
4. **Le vocabulaire externe** d'alignement (schema.org ou équivalent).
5. **Le preset d'adaptateur** (collection, prefix) — dans le respect des contraintes §2.0.1.
6. La démonstration que les **invariants §0** restent satisfaits.

Un profil décrit un contenu **publié** : un travail en cours, avec son suivi, relève d'Hyperfocus et non d'un profil.

Un profil ne DOIT jamais : renommer un champ core, modifier le slug regex, supprimer une obligation du contrat d'adaptateur (§2.0), ou exiger une récursion dans `media/`.

---

### G.12 — Profil Projet : passé dans Hyperfocus *(introduit v2.11, retiré en v2.14)*

Le profil Projet décrivait un dossier de travail — un projet photo suivi à la source, avec ses originaux, son client et ses échéances. Ce n'est pas un contenu publié : il relève d'**Hyperfocus** ([`spec-hyperfocus.md`](https://github.com/izo/hyperfocus/blob/main/spec-hyperfocus.md)), qui découle de cette spec.

| Ce que décrivait G.12 | Où le trouver |
|---|---|
| Le bloc `project:` et ses champs | Hyperfocus §2.2 |
| Les statuts | Hyperfocus §2.3 (les quatre de G.12, plus `en-pause` et `abandonne`) |
| Le nom du dossier, le titre, `date` | Hyperfocus §1.3, §1.4, §2.1 |
| La publication par export | Hyperfocus §5 |
| Les règles d'un outil de gestion | Hyperfocus §4 |
| Migrer un dossier G.12 | Hyperfocus §7 : ajouter `type: project` |

**Ce qui reste normatif ici**, parce que les lecteurs et la publication en ont besoin :

- les **originaux** (§1.9) : un lecteur les ignore, la liste fermée est dans cette spec ;
- la **projection publiable** et la garde (§4.11), qui retirent les originaux d'un snapshot — mais pas le contenu de l'`index.md` ;
- la règle qu'**un dossier de travail ne se publie jamais directement** : il se publie par export ;
- la valeur `type: project`, réservée à Hyperfocus (§1.3).

Un dossier G.12 existant reste lisible par Hyperfocus : c'est un projet auquel il manque `type: project`. Le corpus `fixtures/ingestion/corpora/originals/` en reste le témoin pour la couche 4 : un dossier de travail avec RAW, sidecars et JPEG, tel que la garde le traite.

---

## Changelog

### 2.17-draft — 2026-10-30

Durcissement issu de l'audit du 2026-10-29 (sécurité, coût imposé, fixtures) : la spec n'avait **aucun modèle de menace**, alors que sa raison d'être est de laisser des contributeurs non développeurs publier depuis un dossier partagé. Une seule version regroupe les correctifs de l'audit, détaillés ci-dessous au fil des issues.

#### Schémas d'URL et modèle de confiance (§0, §1.12, §2.1, §4.10, Annexe A) — SPEC-S-02, [#49](https://github.com/izo/hyperfocale-spec/issues/49)

- **§1.12 (nouveau)** : le contenu d'un corpus partagé n'est pas fiable ; les champs d'URL n'admettent que `http:`, `https:`, un chemin absolu au site ou un chemin relatif. `javascript:`, `data:`, `file:` et tout autre schéma sont refusés, ainsi que `//hôte` et les valeurs à caractère de contrôle. Contrat minimum d'un lecteur (§0, point 9) et invariants de §0 alignés.
- **§4.10** : nouveau diagnostic `url-scheme-forbidden` (error, règle 15) pour les champs d'URL du core, frontmatter et `images.json`.
- **§2.1** : l'exemple Zod n'utilise plus `z.url()`, qui accepte `javascript:`, `data:` et `file:` (mesuré avec Zod 4.6.7) et rejetait déjà les entrées relatives de §1.5.1.
- **Identifiants d'embed** : `id` borné à `^[A-Za-z0-9_-]{1,128}$` avant toute construction d'URL de lecture ; sinon repli sur le lien (la classe de défaut de plugin 0.18.2).

**Décisions normatives** : *liste blanche* plutôt qu'une liste noire de schémas dangereux (la liste noire oublie le suivant) ; `mailto:` refusé dans les champs du core, aucun n'en désigne un contact ; la valeur se lit telle qu'écrite, sans décodage, pour qu'aucun contournement par encodage ne passe le contrôle ; les champs des profils G restent hors de la validation de la couche 4 (règle commune de G) mais sous le contrôle du rendu.

**Fixtures** : corpus `invalid-url-scheme-forbidden` (deux fichiers fautifs, une série aux formes admises).

#### `index.mdx` interdit, rendu assaini (§0, §1.2, §1.12, §2.0, §4.3, §4.10, Annexe A) — SPEC-S-01, [#50](https://github.com/izo/hyperfocale-spec/issues/50)

- **`index.mdx` est interdit** : seul `index.md` (et `index.<lang>.md`) porte une série. Décision du propriétaire du 2026-10-29 ([Hyper-F#15](https://github.com/izo/Hyper-F/issues/15)), qui **remplace** la règle de la 2.15 (« `index.mdx` admis, `index.md` fait foi »). §1.2, §0 (invariants), §2.0 (contrat d'adaptateur) et l'Annexe A alignés.
- **§1.12** : un lecteur ne compile, n'évalue ni n'importe jamais un `.mdx` de corpus ; un outil qui écrit n'en crée pas ; le HTML rendu depuis le body est **assaini** (éléments exécutables, attributs `on…`, attributs d'URL hors liste blanche) ; les données injectées dans une page sont échappées selon leur contexte (la faille corrigée en 0.18.1 devient une règle).
- **§4.10** : nouveau diagnostic `index-mdx-forbidden` (error). `index.mdx` n'est plus un fichier index : il ne rend pas son dossier porteur, n'est jamais lu, et la règle 13 (type d'un dossier porteur) ne le consulte plus. §4.3 : il reste `content` (le snapshot décrit tout ce que la source contient).

**Décisions normatives** : *interdire* plutôt qu'exiger l'assainissement du MDX — un fichier MDX est du code, aucun filtre ne le rend inoffensif ; l'assainissement du Markdown s'ajoute quand même, car le HTML brut d'un body passe par le moteur de rendu ; *diagnostic d'erreur* plutôt qu'un fichier ignoré en silence, pour qu'un `index.mdx` déposé par erreur ne soit pas publié comme une page vide ; le diagnostic vaut dans toute racine de validation, `placeholder` et `conflict` compris (il ne dépend que du chemin).

**Fixtures** : corpus `invalid-index-mdx-forbidden` (`index.md` + `index.mdx`, `index.en.mdx`, `index.mdx` seul). Les validations existantes sont inchangées.

#### `private` strictement booléen (§1.12, §4.10, §4.11) — SPEC-S-04, [#51](https://github.com/izo/hyperfocale-spec/issues/51)

La confidentialité ne reposait que sur la frappe exacte d'un booléen dans un fichier édité à la main : `private: "true"`, `private: yes` ou une clé `Private` laissaient partir une série en public, sans diagnostic, notamment à la première publication (`base: null`, rien n'est lu).

- **§1.12 (Confidentialité)** : `private` est un booléen YAML ; toute autre valeur est une erreur, jamais un contenu public. Un site qui emploie `private` refuse de publier un contenu dont `private` n'est pas un booléen, ou le traite comme privé.
- **§4.10** : `private-type-invalid` (error) pour une valeur non booléenne, page de section et `index.<lang>.md` compris ; `private-key-suspect` (warning) pour une clé qui ne diffère de `private` que par la casse, des espaces, `_` ou `-`. `null` vaut absent, comme pour `type`, `date`, `cover`.
- **§4.11** : la garde ne lit pas un `private` non booléen comme un marqueur : une série privée dans `base` dont le `private` devient `"true"`, `yes`, `1` ou `Private` dans `target` est exposée. Le comportement de la garde ne change pas, la règle était déjà « booléen strict » ; elle est maintenant écrite et testée.

**Décisions normatives** : *erreur* plutôt qu'interprétation (`yes` est un booléen en YAML 1.1 et une chaîne en 1.2 : deux lecteurs divergeraient) ; *refus côté site* plutôt que « traiter comme public » ; clés quasi-homonymes en *warning*, car une clé `Private` n'est pas une erreur de format, seulement une protection qui n'en est pas une.

**Hors périmètre de cette version** (non décidé, à reprendre à part) : promouvoir `private` (ou `visibility`) dans le core avec un effet normatif sur le snapshot publié ; faire retirer `portfolio.client` du fichier publié par `confidential` (G.7), pas seulement du rendu.

**Fixtures** : corpus `invalid-private-type-invalid` et `invalid-private-key-suspect` ; garde `private-type-changed`.

### 2.16-draft — 2026-10-25

Précision de §1.8 ([#45](https://github.com/izo/hyperfocale-spec/issues/45)) : **le parent d'une série est son plus proche ancêtre qui est une série**, et non son plus proche ancêtre porteur d'un `index.md`. **Aucune fixture ne change** : la couche 4 comptait déjà ainsi (§4.10, `nesting-too-deep` : « les sections `type: section` ne comptent pas » ; règle 14 : dossiers porteurs *de type série*) ; c'est la couche 1 qui ne le disait pas.

**Décisions normatives**, choisie par le propriétaire parmi trois lectures, réversible :

- **Une page de section est transparente pour le parent** (§1.8). Alternatives écartées : *parent = la section* (lecture littérale de la 2.15) — une section deviendrait un conteneur de line-up, ce que §1.10 ne prévoit pas (« une section range, elle ne contient pas ») et que §4.10 contredit en ne la comptant pas dans la profondeur ; *interdire une série sous une section* — rejette des corpus valides (une rubrique intercalée dans un festival) et ajouterait une règle de validation, donc une fixture, pour un gain nul.

**Écarts consignés** (§0.5) : le plugin Astro implémente cette lecture depuis la 0.25.0 ; le CMS ne traite pas une section comme un conteneur et n'a pas été relevé sur ce cas.

### 2.15-draft — 2026-10-25

Suite de la revue de cohérence de la 2.12 ([#37](https://github.com/izo/hyperfocale-spec/issues/37)) : les points restants. **Aucune fixture ne change** et la couche 4 garde son comportement ; plusieurs règles de la couche 1 se précisent.

#### Listing, brouillons, identité (§0, §1.3, §1.6, §1.8, §1.10)

- **Tri** : seule la date calendaire compte (l'heure éventuelle est ignorée) ; à date égale, titre croissant puis chemin, sur les octets UTF-8 en NFC ; les séries sans date viennent après les datées (points 9).
- **`draft`** : en production, ni page, ni listing, ni flux, ni sitemap ; **ce n'est pas une protection d'accès aux fichiers** — l'ingestion copie ceux d'un brouillon (10).
- **Un rangement peut s'intercaler entre un conteneur et une sous-série** ; le parent d'une série est son plus proche ancêtre porteur d'un `index.md` ; `nesting-too-deep` ne compte pas les rangements (11).
- **Un `index.md` à la racine du corpus déclare `type: section`** (12). **`index.mdx`** est admis, `index.md` fait foi (13).
- **Codes de langue** : deux lettres, région possible (`pt-BR`), pour `lang` comme pour `translations` (15).

#### Annexes et profils (Annexes A, B, F, G ; §2.5)

- Annexe A : `cover` vérifié selon le cas (image de `media/`, ou d'un contenu regroupé) ; Annexe B : les anciens `tags` ne migrent pas tous sous `iptc.keywords` ; Annexe F : « quatre » stratégies (16 à 18).
- **Règles communes de G** : ordre de tri propre à un profil, fichiers désignés par un nom ou `./media/<nom>`, vocabulaires fermés en slugs ASCII pour les profils nouveaux ; G.11 : un profil peut déclarer des champs requis ; `type` rangé dans le core (4, 19, 20, 21).
- **G.6** : `next` et `prev` désignent un écran par son chemin et ne sont pas des jointures (4).
- §2.5 : le plugin Obsidian découvre les séries récursivement (24).

#### Architecture, périmètre, types (Architecture, Philosophie, §3, §2.1)

- La couche 2 est normative par son **contrat d'adaptateur** ; ses descriptions par plateforme sont des exemples ; la couche 3 est une recommandation (3).
- Périmètre : §1.5.1 est cité comme borne de l'éditeur (5).
- **Types partagés** : `Series.path`, `date` et `cover` optionnels, `tags`, `featured`, `embeds` ; interface `Embed` et composant `SeriesEmbeds` (14).
- §2.1 : noms de paquet réels (`@regrets/hyperfocale`), schéma présenté comme illustratif ; exemple G.3 sans Tauri (23).

#### Garde de publication (§4.11)

- Les sections comptent comme des séries dans les seuils, **par prudence** : le choix est maintenant écrit, le comportement ne change pas (22).

**Décisions normatives**, toutes **prises sur recommandation et réversibles** : l'égalité de date départagée par titre puis chemin ; l'absence de page d'un brouillon ; la sous-série à travers un rangement ; `index.md` qui fait foi sur `index.mdx` ; les codes de langue avec région.

**Écart consigné** : le plugin Astro ne retient pas, dans `getSubSeries()`, une série atteinte à travers un rangement (§0.5).

### 2.14-draft — 2026-10-25

#### Hyperfocale, spec photo pour le web ; le travail à la source passe dans Hyperfocus

**Modifications** :
- En-tête, §0 (*Ce que n'est PAS le format*) et Philosophie : Hyperfocale est une spec de **séries photo pour le web**. Le travail qui précède la publication relève d'**Hyperfocus** ([`spec-hyperfocus.md`](https://github.com/izo/hyperfocus/blob/main/spec-hyperfocus.md)), qui découle de cette spec.
- **G.12 — Profil Projet est retiré** de l'annexe et remplacé par un renvoi vers Hyperfocus (tableau de correspondance). §1.9, §2.0.1, §4.11, G.11 et la vue d'ensemble de l'Annexe G alignés ; §2.0.1 compte désormais **dix** presets non photo.
- **Annexe G** : les profils sont des **extensions hors noyau** — normatifs pour qui les implémente, jamais requis pour la conformité au noyau (§2.0).
- **§1.3** : la valeur `type: project` est **réservée à Hyperfocus**. L'ingestion la refuse déjà (`type-invalid`) ; un adaptateur DEVRAIT la refuser explicitement plutôt que d'afficher le dossier.
- §2.7 : note de renvoi — la chaîne Lightroom relève à terme d'Hyperfocus, la section reste le contrat de l'exporter d'ici là.
- §0.5 : hyperfocale-vault repris par Hyperfocus (non relevé) ; preset `project` du plugin Astro, qui refuse `type: project`.

**Décisions normatives** :
- **Ce qui reste dans Hyperfocale** : la série publiée et son `iptc:`, les images et la galerie, le classement des originaux (§1.9, un lecteur et la garde en ont besoin), la couche 4. La ligne passe **à l'export**.
- **Un dossier de travail ne se publie jamais directement** : règle conservée ici, sous le nom d'Hyperfocus.
- **`type: project` est refusé, pas ignoré.** Le comportement de l'ingestion et du plugin de référence devient la règle ; il évite qu'un dossier de travail s'affiche comme une série.
- **§2.7 n'est pas déménagée** : Hyperfocus ne définit pas encore comment la chaîne Lightroom alimente un projet. Déplacer le texte avant aurait retiré à l'exporter son contrat.

**Aucune fixture ne change.** `invalid-type-invalid` (valeur `galerie`) n'est pas concernée ; le corpus `originals` reste le témoin de la garde.

**Dépendances** : la 2.13 ([#38](https://github.com/izo/hyperfocale-spec/pull/38)) est fusionnée. Cette version renvoie à **Hyperfocus**, une spec distincte qui a son **propre dépôt**, [izo/hyperfocus](https://github.com/izo/hyperfocus) : les renvois nomment Hyperfocus et ses sections (`Hyperfocus §5`) et pointent vers lui. Aucune fusion simultanée n'est requise.

**Justification** : cadrage du 2026-10-25 ([#36](https://github.com/izo/hyperfocale-spec/issues/36)) — Hyperfocus reprend la gestion de projet et le côté photo en amont ; Hyperfocale reste la spec photo pour le web.

### 2.13-draft — 2026-10-24

Corrections de cohérence issues de la relecture intégrale de la 2.12-draft ([#37](https://github.com/izo/hyperfocale-spec/issues/37)).

#### Caractère requis de `date` (§0, §1.3, §2.0, §2.0.1, Annexes A et G)

**Ajouts** :
- §1.3 — *Date requise ou optionnelle* : `date` est requise par défaut ; une collection la rend optionnelle pour tous ses contenus en se déclarant `dateRequired: false`, par son preset (§2.0.1) ou par sa racine de validation (§4.10).
- §2.0 — obligation « Exiger les champs requis », jusqu'ici écrite au seul §0.

**Modifications** :
- §0 (frontmatter minimal, point 2 du contrat minimum), §1.8, §1.10, §2.0.1, Annexe A et règles communes de l'Annexe G alignés sur cette règle. §2.0.1 : un preset NE DOIT PAS rendre `title` optionnel.
- §0.5 — l'option `dateRequired` du plugin Astro n'est plus notée comme un écart.

**Décisions normatives** :
- **Le caractère requis de `date` appartient à la collection, jamais au fichier.** Un contenu ne peut pas s'en dispenser lui-même ; sans déclaration, `dateRequired` vaut `true`.
- **`dateRequired: false` n'est pas la suppression d'une obligation** : le contrat exige `date` sauf dans une collection ainsi déclarée. `date` est le seul champ du core qu'un preset peut rendre optionnel ; `title` ne l'est jamais.
- **Une section échappe à la règle par `type: section`**, dans toute collection. Le changelog 2.6 justifiait §1.10 par l'impossibilité de relâcher `date` par preset ; la raison qui tient est autre : un preset vaut pour une collection entière, alors qu'une section vit **parmi** des séries datées — il fallait un discriminant par fichier.
- **Aucune fixture ne change** : `date-missing` (§4.10) appliquait déjà cette règle (`type: section` ou racine `dateRequired: false`).

**Justification** : la 2.12 se contredisait. §0 et l'Annexe A exigeaient `date` de tout contenu qui n'est pas une section, huit profils de l'Annexe G la rendaient optionnelle par preset, et le changelog 2.6 affirmait qu'un preset ne pouvait pas le faire. Un lecteur qui suivait §0 à la lettre rejetait une recette valide.

#### `media/` facultatif sans fichier local (§0, §1.2, §2.5, Annexe G)

**Ajouts** :
- §1.2 — *Absence de `media/`* : une série sans fichier local PEUT ne pas avoir de `media/` ; un lecteur DOIT traiter un `media/` absent comme un `media/` vide.

**Modifications** :
- §0 (anatomie, point 3 du contrat minimum), §1.2, périmètre du plugin Obsidian (§2.5, qui exigeait aussi `date`) et règle commune *Squelette inchangé* de l'Annexe G alignés.

**Décisions normatives** :
- **`media/` porte les fichiers locaux d'une série ; sans fichier local, il est facultatif.** Cas couverts : `images:` ou manifeste `images.json`, embeds seuls, conteneur sans galerie propre, brouillon vide.
- **Absent vaut vide** : ni image, ni document joint, ni erreur. Un outil PEUT créer un `media/` vide à la création d'une série ; aucun lecteur ne DOIT en dépendre.
- **Aucune fixture ne change** : la couche 4 n'a jamais exigé `media/` — `manifest/series/concert-2019` n'en a pas et se valide sans diagnostic.

**Justification** : la 2.12 déclarait `media/` obligatoire (§0, §1.2) tout en décrivant des séries qui n'en ont pas : manifeste (§1.5.1 — 309 séries sur 332 de `mathieu-drouet.com` au 2026-07-26), conteneur sans galerie (§1.8), série d'embeds (§1.11). L'obligation était en outre intenable : ni Git ni un snapshot ne conservent un dossier vide, si bien que le `media/` vide d'un brouillon disparaissait au premier commit ou à la première ingestion.

#### Ordre alphabétique, casse des extensions et fichiers système (§0, §1.6, §1.9, §2.0)

**Modifications** :
- §1.6 — *Tri* : l'ordre « alphabétique » des fichiers de `media/` est l'ordre canonique de §4.1 (octets UTF-8 du nom en NFC, casse préservée). *Scan* : le glob littéral est remplacé par une règle — extension comparée en minuscules, fichiers système ignorés.
- §0 (règles invariantes, point 3 du contrat minimum), §1.9 (classification, tri des documents joints) et §2.0 (obligation *Scanner `media/`*) renvoient à cette définition. Nouvelle règle invariante *Fichiers système*.
- §1.9 — *Robustesse* : un fichier de type inconnu est classé `file` et listé en document joint, comme le dit le tableau des classes — et non « ignoré ».

**Décisions normatives** :
- **« Alphabétique » veut dire octets UTF-8, pas collation.** `10.jpg` précède `9.jpg`, `B.jpg` précède `a.jpg`. Le tri « naturel » des nombres et la collation locale sont écartés : ni l'un ni l'autre ne donne le même résultat en TypeScript et en Swift, et la couverture par défaut en dépend. L'ordre est celui que la couche 4 fixe déjà pour les chemins (§4.1) ; le nommage à padding constant (`01.jpg`) reste la façon d'imposer un ordre lisible.
- **L'extension se compare en minuscules partout**, scan de galerie compris : `DSCF0001.JPG` est une image. Un glob littéral sensible à la casse n'est pas conforme.
- **Les fichiers que la couche 4 exclut (§4.2) ne sont jamais des médias.** Sans cette règle, un `.DS_Store` ou un `._01.jpg` d'un volume macOS tombait dans « tout le reste » et devenait un document joint proposé au téléchargement.
- **Aucune fixture ne change** : la couche 4 ne trie pas la galerie et classait déjà par extension en minuscules ; l'ordre canonique y est déjà exercé (identifiants §4.6 dans `snapshot-id/`, collisions dans `paths/collision.json`).

**Justification** : la 2.12 triait « alphabétiquement » sans dire comment, et décrivait le scan par un glob `media/*.{jpg,…}` qui, pris à la lettre, manque `DSCF0002.JPG` — la couverture de l'exemple G.12. Le plugin Astro avait corrigé la casse de lui-même en v0.20.0 ; la spec, non. Deux lecteurs conformes pouvaient ainsi ordonner la même galerie différemment et choisir une couverture différente.

#### Routes construites sur le chemin, pagination sous `page/` (§1.6, §1.8, §2.1, §2.2, Annexe A)

**Modifications** :
- §1.6 — *URLs* : `/<prefix>/<chemin>/` pour une série, `/<prefix>/<chemin>/page/<n>/` (`<n>` ≥ 2) pour la pagination, et quatre règles : *Identité*, *Pagination*, *Segment réservé*, *Ancienne forme*.
- §1.8 (routes du conteneur), §2.1 (routes injectées), §2.2 (arborescence Next.js) et Annexe A (dossier `page` réservé) alignés.
- §0.5 — plugin Astro : routes déjà construites sur le chemin ; pagination à l'ancienne forme, écart ouvert.

**Décisions normatives** :
- **Une route se construit sur le chemin, jamais sur le slug seul.** §1.2 l'affirmait depuis la v2.6 (« le chemin relatif est la clé de routage ») ; §1.6 routait encore par `<slug>`. Pour la forme canonique `series/<slug>/`, rien ne change.
- **La pagination passe sous un segment `page`.** L'ancienne forme `/<slug>/<page>/` se confondait avec une sous-série dont le slug est un nombre — `2` est un slug valide. Écarté : interdire les slugs numériques, qui casse des contenus existants pour un problème de routage.
- **`page` est réservé directement dans un dossier de série**, seul cas où une série rangée pourrait encore prendre une route de pagination. Le lint le signale ; aucun diagnostic de couche 4 n'est ajouté.
- **L'ancienne forme se redirige** quand aucune série n'occupe le chemin, pour ne pas casser les liens publiés.
- **Aucune fixture ne change** : la couche 4 ne construit pas de route.

**Justification** : §1.6 routait par slug alors que §1.2 dit le slug non unique, et sa route de pagination entrait en collision avec les sous-séries de §1.8. Le cas est réel dans le plugin Astro (`main@f8d1a52`) : dès que la galerie d'un conteneur `festival` dépasse une page, sa page 2 et sa sous-série `2` produisent toutes deux `/series/festival/2/`.

### 2.12-draft — 2026-10-24

#### Ordre éditorial des images dans une source distante (§1.5.1)

**Ajouts** :
- §1.5.1 — trois règles du manifeste : un outil d'ingestion NE DOIT PAS produire d'`images.json` pour une série qui porte `images:` ; un éditeur NE DOIT PAS écrire d'`images.json` dans une source distante, un ordre éditorial s'écrit dans `images:` du frontmatter ou par le nommage ; les entrées relatives de `images:` se résolvent comme celles du manifeste.

**Décisions normatives** :
- **Dans une source distante, l'ordre éditorial vit dans `index.md`, jamais dans `images.json`.** Le manifeste y est une donnée dérivée (§4.3) que l'ingestion PEUT régénérer : un ordre qu'un éditeur y écrirait serait écrasé sans diagnostic. L'alternative — obliger l'ingestion à conserver un manifeste édité — est écartée : elle imposait de distinguer un manifeste éditorial d'un manifeste dérivé, donc de revoir la classification, le hash et les fixtures de la couche 4. L'alternative du nommage seul aussi : chaque réordonnancement renommerait les médias, donc changerait leurs clés de stockage.
- **Aucune fixture ne change.** La couche 4 ne fixe pas la génération du manifeste ; le seul diagnostic concerné, `images-conflict`, existe déjà et s'applique tel quel.

**Justification** : constaté le 2026-10-24 sur la chaîne Hyperfocale CMS → Dropbox → `mathieu-drouet.com` ([#33](https://github.com/izo/hyperfocale-spec/issues/33)). En nommage d'origine, le CMS écrivait un `images.json` pour porter un ordre non alphabétique ; l'ingestion du site l'ignorait comme donnée dérivée puis le régénérait en ordre alphabétique. La spec suggérait déjà la voie retenue (§1.5.1, *Pourquoi un fichier annexe* : « une série dont la liste d'images est éditoriale a toute raison de la garder dans son frontmatter ») sans la rendre obligatoire.

### 2.11-draft — 2026-10-15

#### Originaux dans `media/` (§1.9), garde de publication (§4.11) et profil Projet (G.12)

**Ajouts** :
- §1.9 — **Originaux** : troisième catégorie de fichiers de `media/`, à côté des images et des documents joints. Liste fermée, insensible à la casse : RAW `.raf .cr2 .cr3 .nef .nrw .arw .srf .sr2 .orf .rw2 .pef .dng .3fr .iiq .x3f .srw .rwl .erf .mos .mef .kdc .dcr`, sidecars `.xmp .pp3 .dop .cos`. Classe `original` ajoutée au tableau des classes de médias.
- §0 — règles invariantes (ligne « Originaux », ligne « Documents joints » restreinte) et contrat minimum du lecteur (points 3 et 4) ; §1.2, §1.9 *Mode distant*, §2.0 (obligation « Ignorer les originaux », conformité v2.11), §3.2 et Annexe A alignés : un fichier de `media/` qui n'est pas une image n'est plus d'office un document joint.
- §4.11 — **projection publiable** `publishable(snapshot)` : le snapshot sans ses entrées originales, `id` recalculé. Diagnostic `guard-originals-excluded` (info, toujours évalué) avec deux champs normatifs, `count` et `bytes`. `guard-oversize` s'évalue sur la projection. §4.10 — un code PEUT porter des champs supplémentaires ; règle 12 : une entrée `attachments:` vers un original produit `attachment-not-found`.
- `fixtures/ingestion/` — corpus `originals` (RAW, XMP, JPEG, devis), `snapshots/originals.json`, `validation/originals.json`, `validation/originals-references.json`, nouveau dossier `publishable/` (trois cas), `guard/originals.json`.
- **G.12 — Projet** (`project`, `/projects`, `schema.org/CreativeWork`) : bloc `project:` (`status`, `stage`, `client`, `contact`, `start`, `end`, `due`, `deliverables`, `tasks`), statuts `prospect` · `en-cours` · `livre` · `archive`, publication par export, règles d'un outil de gestion. Vue d'ensemble de l'annexe et §2.0.1 complétés.
- §0.5 — deux obligations rouvertes pour le plugin Astro (originaux, garde), profil `project` non couvert.

**Décisions normatives** :
- **Un original n'est ni une image ni un document joint.** Un lecteur l'ignore : ni galerie, ni pièces jointes, jamais candidat à `cover`. Le critère est l'extension — partie qui suit le dernier point, en minuscules —, comme pour les autres classes.
- **La liste est fermée et vit dans la spec**, sans renvoi à une source externe : deux implémentations doivent classer le même fichier de la même façon. Une extension RAW manquante reste un document joint `file` jusqu'à son ajout par PR.
- **Les originaux restent à plat dans `media/`.** L'alternative d'un sous-dossier `media/originals/` est écartée : elle casse la règle « pas de récursion ».
- **Le snapshot d'une source garde ses originaux ; le snapshot publié n'en a jamais.** La classification de §4.3 ne change pas — un original reste une entrée `media` —, c'est la projection qui l'écarte. La validation porte sur la source, le diff et la garde sur la projection. Un snapshot sans original est sa propre projection : aucune fixture existante ne change.
- **L'exclusion se voit** : `guard-originals-excluded` rapporte nombre et octets, en `info` — c'est le comportement attendu, pas une faute.
- **Rétro-compatibilité** : un lecteur antérieur à v2.11 classe les originaux en documents joints `file` et les propose en téléchargement. Aucun contenu publié n'est cassé, mais un dossier de travail ne doit donc jamais être publié directement : il se publie par **export** vers un nouveau `<slug>/` de type série, images seules, sans bloc `project:`. La garde v2.11 reste un filet, pas un mode de publication.
- **Les données de gestion vivent dans `index.md`, sous `project:`**, et non dans un fichier voisin (`project.yml`) : un projet reste un dossier autonome. Un outil de gestion qui réécrit `index.md` DOIT préserver les champs inconnus et le body.
- **`draft: true` est écrit, jamais supposé** : un lecteur qui ignore le profil applique le défaut du core, `false`.
- **`date` est requise** (date de prise de vue ou de début) ; **`stage` est libre** ; les statuts sont des slugs ASCII, et un statut absent ne se déduit pas des dates.
- **`rating_min` est écarté** de la proposition initiale : la sélection publiable se décide image par image — notes des sidecars, choix de l'export —, pas par un seuil inscrit dans le projet.
- **Pas de `type: project`** : `type` reste `series` | `section`, et le profil se déclare par preset, comme les autres.

**Justification** : l'application macOS [hyperfocale-vault](https://github.com/izo/hyperfocale-vault) veut faire du format la base de la gestion de ses projets photo, dès la prise de vue : un dossier par projet, un `index.md` qui porte toutes ses informations, et `media/` qui contient les originaux sortis du boîtier ([hyperfocale-vault#127](https://github.com/izo/hyperfocale-vault/issues/127)). Lu par un lecteur v2.10, un tel dossier proposait ses `.RAF` et ses `.xmp` en téléchargement, puisque tout fichier non image de `media/` était un document joint ; et aucun profil ne décrivait un travail en cours avec son client et ses échéances. Issue [#26](https://github.com/izo/hyperfocale-spec/issues/26).

### 2.10-draft — 2026-09-29

#### Couche 4 (§4.0 à §4.13) et fixtures d'ingestion

**Ajouts** :
- **Couche 4** — contrat des outils d'ingestion : chemins (§4.1), exclusions (§4.2), classification (§4.3), hash dont l'algorithme Dropbox (§4.4), `ContentSnapshot` v1 (§4.5) et son identifiant (§4.6), `ContentChangeSet` v1 et son algorithme (§4.7), `ProviderCapabilities` v1 (§4.8), `PublicationState` (§4.9), table des diagnostics et règles d'évaluation (§4.10), garde de publication (§4.11), fixtures (§4.12), exemples Dropbox, WebDAV et iCloud Drive non normatifs (§4.13).
- §4.0 — séparation des trois contrats (format, ingestion/snapshot, consommation) et six invariants : provider opaque, aucune lecture runtime, incomplet ≠ suppression, N intact tant que N+1 n'est pas publié, nettoyage après succès, idempotence.
- `fixtures/ingestion/` — premier contenu du dépôt hors prose : 26 corpus, leurs snapshots, 44 cas de validation, 15 diffs, 5 cas de garde, 7 identifiants, 5 jeux de chemins, vecteurs de hash. Normatives au même titre que le texte.
- Architecture en couches (quatre couches), table des matières, « Ce que cette spec définit / NE définit PAS ».
- §1.2 — un corpus PEUT résider sur un filesystem distant ou synchronisé ; §1.5.1 — `images.json` reste dérivé à l'ingestion ; Annexe A — renvoi vers §4.10.

**Décisions normatives** :
- **Égalité et ordre sur les octets** : une implémentation Swift compare et trie les chemins sur `unicodeScalars`/`utf8`, jamais sur `String`, qui compare par équivalence canonique. Toute implémentation conforme sait calculer `sha256` et `dropbox` ; un pipeline DEVRAIT garder le même jeu d'algorithmes d'une ingestion à l'autre (§4.6).
- **La couche 4 ne touche pas au format.** Elle est normative pour les outils d'ingestion et ne change rien à ce qu'un lecteur doit faire. Aucun champ n'est ajouté au frontmatter ; la couche 1 ne reçoit que deux phrases.
- **Incomplet ≠ suppression.** Seul un snapshot `complete: true` fonde une suppression ; la garde bloque tout snapshot incomplet ou vide, sans désactivation possible.
- **L'identifiant ne dépend que de `path`, `kind`, `size` et `hashes`** — ni de l'instant, ni de la source, ni de l'identité attribuée par le provider. Même état, mêmes algorithmes, même `id`.
- **Le diff est prudent.** Sans algorithme de hash commun, une entrée est `modified`, jamais « inchangée ». Les déplacements s'infèrent par identité, puis par contenu, et seulement sur appariement unique.
- **Minuscule simple pour les collisions, octets UTF-8 pour le tri** : les deux points où JavaScript et Swift divergent si l'on s'en remet à leurs bibliothèques (`toLowerCase()` applique le sigma final, les chaînes JavaScript se comparent en UTF-16).
- **Le frontmatter se lit sous le schéma YAML 1.2 core.** Une date non guillemetée reste une chaîne, validée par un motif ISO 8601 et par le calendrier. Sans cette règle, `2024-02-30` passait avec le schéma par défaut de js-yaml (débordement silencieux vers le 1ᵉʳ mars) et échouait avec d'autres parseurs.
- **Les transitions de `PublicationState` sont exhaustives** : état initial `sourceDirty` (ou `sourceSynced` pour un amorçage depuis un snapshot déjà publié), et `failed` → `snapshotPending` pour un nouvel essai sur une source inchangée ; toute transition absente de la table est interdite.
- **La garde est fail-closed côté `base`** : un fichier index de `base` illisible, aux octets divergents, `placeholder` ou en conflit rend la série privée par prudence ; côté `target`, un fichier illisible ne déclare rien. La garde vérifie les octets mais n'émet pas `entry-hash-mismatch`, qui appartient à la validation.
- **La garde connaît la confidentialité** : signature `guardChangeSet(changeSet, base, target, { read, policy })` ; une série est privée si au moins un de ses fichiers index déclare `private: true` (booléen YAML, la chaîne `"true"` ne compte pas) ; l'exposition suppose une série présente dans `target`, au même chemin ou déplacée — une série privée supprimée n'est pas exposée. `private` reste une extension de site.
- **Les octets lus se vérifient** : un outil DOIT comparer ce qu'il lit au hash de l'entrée (premier algorithme enregistré qu'elle porte). Un écart (`entry-hash-mismatch`) signe un fichier modifié entre le listing et la lecture : le snapshot n'est pas publiable, il faut un nouveau listing.
- **Un fichier en conflit bloque la publication** : `state` admet `"conflict"` (copie conflictuelle Dropbox, versions concurrentes iCloud) ; l'entrée n'est jamais lue et produit `entry-conflict`.
- **Un document mal formé est rejeté avant d'être interprété** : `snapshot-invalid` (format, champ requis absent ou mal typé) arrête la validation comme `snapshot-version-unsupported` ; `entry-invalid` écarte une entrée mal typée (taille, `kind`, `state`, hashes, algorithme inconnu) du reste de la validation.
- **`hash-incomparable` exige `kind` et `size` égaux**, y compris pour un déplacement par identité : un move dont la taille change est `modified: true` sans diagnostic. L'étape 0 du diff compare les `id` **déclarés** ; leur cohérence relève de `snapshot-id-mismatch`.
- **L'exclusion précède la validité** ; dans un snapshot, un chemin non NFC ou exclu est invalide.
- **Un snapshot se vérifie, il ne se croit pas sur parole** : un `id` qui ne correspond pas aux entrées (`snapshot-id-mismatch`) et un `kind` qui contredit le chemin (`entry-kind-mismatch`) sont des erreurs. La validation continue dans les deux cas, et s'appuie sur la classification recalculée.
- **`cover-not-found` consulte `images.json` en plus du snapshot**, par résolution d'un chemin relatif ou par suffixe d'une URL absolue — sans quoi un corpus dont les médias ne vivent que dans le manifeste produirait un avertissement par série.
- **La sévérité est celle de l'ingestion**, qui décide d'une publication automatique : une entrée `embeds` sans `url` reste ignorée au rendu (§1.11) mais bloque la publication (`embed-url-missing`, error).
- **Dropbox n'est jamais normatif.** Son algorithme de hash et ses identifiants sont des cas du contrat, au même titre que ceux de WebDAV, d'iCloud Drive ou de Google Drive.

**Justification** : le site `mathieu-drouet.com` doit laisser des contributeurs non développeurs gérer le corpus depuis un dossier synchronisé, puis publier sans intervention ([mdr-monorepo#129](https://github.com/izo/mdr-monorepo/issues/129)) ; le CMS Swift doit faire de même depuis iCloud Drive ([hyperfocale-cms#77](https://github.com/izo/hyperfocale-cms/issues/77)). Sans contrat commun, le plugin TypeScript et le CMS Swift auraient chacun inventé leur snapshot, leur diff et leurs règles de suppression — et le premier listing incomplet serait devenu une suppression massive en production. La couche 4 fige ce contrat avant toute implémentation profonde : c'est le Gate W0 de l'epic [#23](https://github.com/izo/hyperfocale-spec/issues/23), atteint quand TypeScript et Swift peuvent l'implémenter sans inventer de champ ni de sémantique (issue [#22](https://github.com/izo/hyperfocale-spec/issues/22)).

### 2.9-draft — 2026-08-22

#### Extension `.tif` reconnue comme image

**Ajout** :
- `.tif` rejoint les extensions d'image partout où la liste apparaît : contrat minimum (§0 — Règles invariantes), règles de structure (§1.2), glob de scan (§1.6 — `media/*.{jpg,jpeg,png,webp,avif,tif,tiff}`) et classe `image` des documents joints (§1.9).

**Décision normative** :
- `.tif` et `.tiff` désignent le même format (TIFF) et sont traités identiquement. La liste ne portait que la graphie longue : un fichier `.tif` — graphie par défaut de nombreux outils, dont l'export TIFF de Lightroom — tombait en classe `file` (§1.9) au lieu d'alimenter la galerie. Non cassant : aucun fichier valide ne change de classe, des fichiers jusqu'ici mal classés en gagnent une meilleure. (Issue [#8](https://github.com/izo/hyperfocale-spec/issues/8).)

#### §0.5 — plugin en v0.18.0, §1.11 refermée

- Le plugin Astro `@regrets/hyperfocale` implémente **§1.11 (contenus embarqués) depuis la v0.17.0** — la dernière obligation ouverte du contrat est refermée. La v0.17.1 exporte les vocabulaires à la racine, la v0.18.0 apporte les helpers multi-collections (besoin de `mathieu-drouet.com`, une collection par locale), la v0.16.0 avait rendu `theme: 'light'`/`'dark'` effectifs.
- `laurenceguenoun.com` reste ⚠️ : son bloc `videos[]` local est à migrer vers `embeds:` maintenant que le plugin le porte.

### 2.8-draft — 2026-08-12

#### §1.11 — Contenus embarqués

**Ajout** :
- **§1.11 — Contenus embarqués** : le champ `embeds`, pour les médias hébergés par une plateforme tierce (Vimeo, YouTube, SoundCloud…) et joués dans la page. `url` est le seul champ requis ; `platform`, `id`, `title`, `description`, `poster`, `width` et `height` sont optionnels.
- `embeds` rejoint la table des champs de workflow (§1.3), le contrat d'adaptateur (§2.0) et la liste de vérifications du lint (Annexe A).

**Décisions normatives** :
- **La frontière avec §1.9 est l'emplacement de l'octet, pas la nature du média.** Un `.mp4` dans `media/` — ou sur son propre CDN via `files:` — reste un document joint : on en sert le fichier et une balise native le lit. L'embed commence là où c'est le lecteur d'un tiers qui rend le média. Un même documentaire relève donc de §1.9 ou de §1.11 selon qui l'héberge.
- **Un poster n'est pas une photo de la série.** Une image de `media/` référencée par `embeds[].poster` est exclue du scan de galerie — seule exception au principe « toute image de `media/` alimente la galerie » (§1.6). Sans elle, une série de trois vidéos afficherait trois vignettes parasites. Le lint interdit aussi qu'une image soit à la fois poster et entrée d'`images:`.
- **`url` seule doit suffire.** Construire un lecteur exige `platform` et `id` ; à défaut l'adaptateur se rabat sur un lien. C'est ce qui rend le champ utilisable avec un hébergeur que la spec ne connaît pas.
- **La spec ne fige aucun gabarit d'iframe.** Les hébergeurs changent les leurs plus vite qu'une spec ne se réédite : la construction de l'URL de lecture appartient à l'adaptateur. Le vocabulaire de `platform` est une liste reconnue, pas une énumération fermée.
- **Une série peut n'avoir aucune image.** Ni `media/` d'images, ni `images:` — elle s'affiche sans galerie, ce n'est pas une erreur. Le format supposait jusqu'ici qu'une série était d'abord un corpus d'images.

**Justification** : le site `laurenceguenoun.com`, premier consommateur du plugin Astro à monter le format en couche data, porte **onze vidéos Vimeo réparties sur trois de ses huit séries** — des pages entières dont le média n'est pas une image. Aucune des trois formes existantes ne les décrit : `attachments:` ne référence que `media/`, `files:` n'a ni vignette ni dimensions, `images:` mettrait une vidéo dans la lightbox. Le site a donc dû étendre le schéma localement, avec un bloc `videos[]` propriétaire — exactement la divergence en silence que §2.0.1 cherche à éviter. Le cas n'a rien de particulier à ce site : dès qu'une série documente de la vidéo, l'octet est ailleurs.

### 2.7-draft — 2026-07-27

#### Quatre profils de contenu (Annexe G)

**Ajouts** :
- **G.7 — Portfolio** (`portfolio`, `/portfolio`, `schema.org/CreativeWork`) : l'atome est un projet livré, pas un corpus d'images.
- **G.8 — Musique** (`music`, `/music`, `schema.org/MusicAlbum`) : l'atome est une sortie (album, EP, single).
- **G.9 — Catalogue** (`catalog`, `/catalog`, `schema.org/Product`) : l'atome est un produit, sans dimension transactionnelle.
- **G.10 — Presse** (`press`, `/press`, `schema.org/NewsArticle`) : l'atome est une parution, émise ou subie.
- « Créer un nouveau profil » passe de G.7 à **G.11**.
- Vue d'ensemble de l'Annexe G complétée : les quatre nouveaux profils, **et `screen` (G.6) qui y manquait** depuis son ajout.

**Décisions normatives** :
- Ces quatre profils ne touchent ni au squelette, ni au slug regex, ni aux champs core : ils n'ajoutent qu'un bloc d'extension namespacé, conformément aux règles communes de l'Annexe G. Un lecteur qui les ignore lit le contenu comme une série standard.
- **Portfolio n'est pas la série canonique.** Une série documente un corpus d'images — les images *sont* le contenu ; un projet documente une réalisation dont les images sont la *trace*. Un book de photographe reste une collection de séries.
- **Les pistes d'une sortie ne sont pas des contenus** : `music.tracks` est une liste de métadonnées, pas une arborescence. Un coffret justifiant une page par disque relève du conteneur §1.8.
- **Le profil catalogue ne fait pas de commerce** : `price` et `availability` sont éditoriaux, jamais une source de vérité transactionnelle (§0 — ce que la spec ne définit pas).
- **Presse distingue l'émis du subi** : `kind: press_release` est produit par soi, toute autre valeur est une retombée tierce. `excerpt` reste une citation courte, au sens du droit de citation.
- Deux dérogations de tri sont explicitées : `music.tracks` se rend par `position` ascendant, et un adaptateur catalogue PEUT trier par catégorie plutôt que par date.

**Justification** : le plugin Astro `@izo/hyperfocale` a livré en v0.8.0 six presets de domaine dont quatre — `portfolio`, `music`, `catalog`, `press` — ne correspondaient à aucun profil standardisé, l'annexe n'en décrivant aucun équivalent. §2.0.1 prévoit explicitement cette voie (« L'ajout de nouveaux profils se discute par PR contre cette spec ») : plutôt que de laisser une implémentation de référence diverger en silence, les quatre profils sont décrits ici. L'écart qui subsistait alors — `photo` au lieu de `series` — portait sur un profil **déjà** standardisé et relevait donc d'une correction côté plugin, pas d'une évolution de la spec ; elle a été livrée en v0.12.0. Son préfixe francisé (`/recettes` là où l'annexe recommande `/recipes`) n'a jamais été un écart : la colonne `prefix` est une recommandation, et §2.0.1 autorise un preset à fixer le sien — voir §0.5.

### 2.6-draft — 2026-07-26

Clarifications issues d'une même mesure : le round-trip du 2026-07-26 sur les **332 séries** de `mathieu-drouet.com`, qui a confronté la spec à un corpus réel pour la première fois. Aucune n'est une rupture — chacune nomme une forme que le réel pratiquait déjà sans que le format sache la décrire.

---

#### Page d'index de section (§1.10)

Comblement d'une lacune du format : un dossier de **rangement** n'avait aucune façon de porter un titre et un texte de présentation.

**Ajouts** :
- §1.10 — Page d'index de section : `type: section`, frontmatter sans `date`, règles de rendu et d'exclusion des listings, tableau de distinction avec le conteneur §1.8.
- §1.3 — Champ core `type` (défaut `series`, valeur normative alternative `section`).
- §0 — Contrat minimum d'un lecteur : ne pas traiter comme une série un `index.md` déclarant `type: section`.
- §2.0 — Nouvelle obligation du contrat d'adaptateur (conformité v2.6).
- Annexe A — Vérifications lint correspondantes.

**Décisions normatives** :
- Une page d'index de section **n'est pas un contenu Hyperfocale** : elle est exclue des listings de séries, des flux et du tri par date. Les invariants §0 sur les séries sont donc inchangés.
- Le discriminant est **explicite et unique** : le champ `type`. Un adaptateur ne DOIT jamais deviner la nature d'un `index.md` par l'absence de `date` ou par la présence de sous-dossiers — une série sans `date` reste une série invalide.
- `type` absent vaut `series` : rétro-compatibilité totale, aucun contenu antérieur n'est requalifié.
- Contrairement au conteneur §1.8, une section PEUT en contenir une autre sans limite de profondeur — ce sont des dossiers de classement, pas des contenus.

**Justification** : mesuré sur le corpus de `mathieu-drouet.com` le 2026-07-26 — **5 fichiers** (`archives/corporate`, `archives/experiments`, `archives/fashion`, `archives/food-and-wine`, `archives/music`) portent `title` + `description` sans `date`. Ce sont les **seuls points bloquants** de tout le corpus, et ce ne sont pas des séries. Aucune forme du format ne les couvrait : `date` est obligatoire au §0, un conteneur §1.8 doit en avoir une, et aucun des six profils de l'Annexe G ne décrit un rangement. Le §2.0.1 interdisant à un preset de supprimer une obligation du contrat d'adaptateur, la lacune ne pouvait pas se contourner par preset.

---

#### Rangement et imbrication (§1.2, §1.8)

Clarification d'une ambiguïté de la §1.8 : la spec confondait **profondeur de rangement** et **imbrication de séries**.

**Ajouts** :
- §1.2 — « Profondeur de rangement » : notion de **section de rangement** (dossier sans `index.md`, profondeur libre), slug = dernier segment, découverte par parcours récursif, chemin relatif comme clé de routage.
- §1.8 — Encadré « Ne pas confondre avec le rangement » + ligne de règle définissant mécaniquement ce qu'est un conteneur.
- Annexe A — Vérification lint : imbrication limitée à un niveau, profondeur de rangement non contrainte.

**Décisions normatives** :
- La limite d'un seul niveau de la §1.8 porte sur l'**imbrication** (une série dans une série), **pas** sur la profondeur de rangement, qui est libre.
- Le discriminant est mécanique : un dossier parent porteur d'un `index.md` est un conteneur §1.8 ; sans `index.md`, c'est une section de rangement §1.2.
- La découverte des séries est un **parcours récursif** de `<content-root>`. Un adaptateur qui liste le seul premier niveau est non conforme.
- Le slug seul PEUT ne pas être unique dans un corpus ; c'est le chemin relatif à `<content-root>` qui identifie une série.

**Justification** : mesuré sur le corpus de `mathieu-drouet.com` le 2026-07-26 — **303 séries sur 332** sont rangées 2 à 4 segments au-dessus de leur slug (`archives/music/concerts/2010/<slug>/`), profondeur maximale 5. Lues à la lettre de la §1.8, ces séries violaient la limite d'un niveau ; elles n'imbriquent pourtant rien — aucun dossier traversé ne porte d'`index.md`. Le même corpus compte par ailleurs **11 vrais conteneurs §1.8** portant 41 sous-séries, tous conformes : les deux formes coexistent et méritaient d'être nommées séparément.
---

#### Manifeste d'images externalisé (§1.5.1)

Officialisation du **manifeste d'images externalisé** : la liste des images d'une série peut vivre dans un fichier annexe `images.json` plutôt que dans le frontmatter.

**Ajouts** :
- §1.5.1 — Manifeste d'images externalisé : formes courte et longue, ordre de priorité `images:` > `images.json` > `media/`, résolution des URLs, couverture, robustesse.
- §1.2 — La variante « médias externes » mentionne les deux formes.
- §2.0 — Nouvelle obligation du contrat d'adaptateur : prendre en charge le manifeste (conformité v2.6).
- Annexe A — Vérifications lint correspondantes (exclusivité des trois modes, validité du JSON).

**Décisions normatives** :
- Les trois modes (scan de `media/`, `images:` du frontmatter, `images.json`) sont **mutuellement exclusifs par série**, dans cet ordre de priorité inverse.
- L'ordre du manifeste **fait foi** — le tri alphabétique de §1.6 ne s'applique qu'au scan de `media/`. Le fallback de couverture devient « première entrée du tableau ».
- `images.json` n'est ni un média ni un document joint (§1.9) : c'est un fichier de métadonnées, comme `index.md`.
- Rétro-compatibilité : un adaptateur antérieur ignore le fichier et rend une série sans galerie, sans erreur. Aucun contenu existant n'est cassé.

**Justification** : mesuré sur le corpus de `mathieu-drouet.com` le 2026-07-26 — **309 séries sur 332** portent un `images.json` par série, aucune n'utilise le mode distant §1.5, et **aucune n'a de dossier `media/` local** (les images vivent sur Cloudflare R2, synchronisées par un manifeste). Le format n'offrait aucune forme normative pour une liste d'images **générée** : l'inscrire dans le frontmatter fait réécrire `index.md` à chaque synchronisation et impose à tout éditeur de préserver une donnée dérivée qu'il n'a pas produite.

### 2.5-draft — 2026-07-09

Extension de `media/` à **tous les types de documents** : une série peut embarquer des documents joints (PDF, vidéo, audio, archives…) aux côtés de ses images.

**Ajouts** :
- §1.9 — Documents joints : classes de médias (`image` / `video` / `audio` / `document` / `file`), règles de rendu (galerie inchangée, pièces jointes après la galerie), bloc frontmatter optionnel `attachments:`, champ `files[]` en mode distant.
- §2.0 — Nouvelle obligation du contrat d'adaptateur : exposer les documents joints (conformité v2.5).
- §3.1 — Composant `SeriesAttachments` ; §3.2 — `Series.attachments` et interface `Attachment`.
- Annexe A — Vérifications lint correspondantes (classification des fichiers, `cover` = image, cohérence du bloc `attachments:`).

**Décisions normatives** :
- La galerie et la lightbox restent réservées à la classe `image` ; `cover` DOIT être une image. Les invariants §0 / §1.6 sont inchangés.
- Rétro-compatibilité totale : les lecteurs existants globbent les extensions image et ignorent déjà les autres fichiers — aucun contenu ni adaptateur existant n'est cassé. L'exposition des pièces jointes devient une obligation à partir de la conformité v2.5.
- `index.md` n'est jamais un document joint.
- Contrat minimum (§0) : ajout de l'exigence « ne jamais échouer sur un fichier de type inconnu dans `media/` ».

**Justification** : le plugin SPIP `spip2astro` (export SPIP → format Hyperfocale) doit exporter **tous** les documents liés à un contenu SPIP — PDF, sons, vidéos, archives — et pas seulement les images. Plus largement, une série photo réelle s'accompagne souvent de pièces (dossier de presse, tracé GPX, enregistrement) qui n'avaient pas de place normative dans le format.

### 2.4-draft — 2026-06-11

Ajout du profil **Écran** (`screen`) — premier profil **séquentiel** (l'atome n'est plus une collection d'objets triée par date, mais une étape ordonnée par `screen.order`).

**Ajouts** :
- G.6 — Profil **Écran** (`screen`) : bloc `screen:` (`order`/`kind` requis, `stage_id`/`persona`/`theme`/`duration`/`skippable`/`next`/`prev`/`llm_context` optionnels), alignement schema.org/HowToStep (écrans séquencés) et schema.org/WebPageElement (écrans-pages).
- G.7 — « Créer un nouveau profil » (renommé depuis G.6).

**Décisions normatives** :
- Un profil PEUT définir un **ordre canonique alternatif** au tri date desc (§1.6) quand sa nature le justifie — `screen` trie par `screen.order` ascendant. C'est une dérogation explicite, à documenter dans le preset.
- Les invariants §0 restent satisfaits : core non renommé, extension namespacée sous `screen:`, passthrough (§1.3). Un lecteur qui ignore le profil lit un contenu Hyperfocale valide.
- `date` reste optionnelle (`dateRequired: false`), comme `recipe`/`app`/`book`/`place`.

**Justification** : le projet `mdr-terminal-portfolio` (portfolio dual-persona, séquence de boot rétro-OS) modélisait chaque écran de son expérience en contenu hardcodé et dispersé. Le profil `screen` les fait converger vers le squelette Hyperfocale, en généralisant le format aux expériences narratives multi-écrans (onboarding, kiosque, démo pas-à-pas) au-delà des collections d'objets.

### 2.3-draft — 2026-06-08

Officialisation des **profils de contenu** : le squelette universel (dossier + `index.md` + `media/`) n'est plus réservé à la série photo. Cinq profils non-photo sont standardisés comme presets de domaine, chacun avec son bloc d'extension namespacé (à l'image de `iptc:` pour la photo).

**Ajouts** :
- Annexe G — Profils de contenu (presets standardisés) : cadre générique + règles communes + vue d'ensemble.
- G.1 — Profil **Événement** (`event`) : bloc `event:`, alignement schema.org/Event, synergie avec les séries imbriquées (§1.8).
- G.2 — Profil **Recette** (`recipe`) : bloc `recipe:` avec ingrédients/étapes structurés, `dateRequired: false`, alignement schema.org/Recipe.
- G.3 — Profil **Application** (`app`) : bloc `app:`, alignement schema.org/SoftwareApplication.
- G.4 — Profil **Livre** (`book`) : bloc `book:`, alignement schema.org/Book.
- G.5 — Profil **Lieu** (`place`) : bloc `place:`, `gps` réutilisé (compatible `SeriesMap`), alignement schema.org/Place.
- G.6 — Procédure de création d'un nouveau profil.

**Décisions normatives** :
- Un profil réutilise le core **sans le renommer** et n'ajoute que sous une clé d'extension unique. Aucun profil ne casse la rétro-compatibilité : un lecteur qui l'ignore lit toujours un contenu Hyperfocale valide (passthrough §1.3).
- Les invariants §0 (body avant galerie, tri alphabétique de `media/`, cover/fallback, draft) s'appliquent identiquement à tous les profils.
- Seule la conditionnalité de `date` peut varier par preset (`dateRequired: false` pour `recipe` et `app`), dans le respect des contraintes §2.0.1.

**Justification** : plusieurs implémentations du dépôt (preset `recipe` du plugin Astro, variante recettes hors source canonique, agenda d'événements de mathieu-drouet.com) divergeaient faute de vocabulaire officiel. L'Annexe G les fait converger vers un cadre unique, dérivé de la spec générique.

### 2.2-draft — 2026-05-19

Officialisation des **séries imbriquées** (séries conteneur + sous-séries) après observation du pattern dans le site `mathieu-drouet.com` (festivals 2026 : `dragatypie-3-la-brat-cave-lille/` et `daimonion-fest-28-fev-2026-faches/` regroupant les performances d'artistes en sous-dossiers).

**Ajouts** :
- §1.8 — Séries imbriquées (conteneur) : structure filesystem, règles, cover du conteneur, URLs, listing global, recommandations de frontmatter, compatibilité.

**Décisions normatives** :
- Une seule profondeur d'imbrication autorisée (pas de récursion infinie).
- Le champ `cover` du conteneur PEUT traverser un sous-dossier vers une image d'une sous-série — dérogation explicite à la règle "pas de récursion dans `media/`" (§1.6).
- Le listing global PEUT aplatir ou hiérarchiser ; ce choix est de la responsabilité de l'adaptateur, à expliciter en config.

**Justification** : permettre de représenter proprement les évènements à line-up multiple (festivals, expositions collectives, reportages chapitrés) sans dupliquer le contexte du conteneur dans chaque sous-série.

### 2.1-draft — 2026-05-15

Première révision normative après audit de conformité des implémentations de référence (plugin Astro, site mathieu-drouet.com, exporter Lightroom SwiftUI + Tauri).

**Ajouts** :
- §0.5 — État des implémentations de référence (tableau de conformité)
- §1.3 — Champs `featured` et `tags` (officialisation de patterns observés)
- §1.3 — Clarification de la relation `tags` ↔ `iptc.keywords`
- §2.0.1 — Presets de domaine (officialisation du pattern Astro)
- §2.7 — Dual naming mode des images (`sequential` vs `original`)
- §2.7 — Bloc `translations:` pour i18n par série
- Annexe F — Stratégie 3 (collections séparées par locale)
- Annexe F — Stratégie 4 (bloc `translations:` dans frontmatter)
- Annexe F — Tableau d'aide au choix de stratégie i18n

**Décisions normatives** :
- Le nom canonique du dossier d'images reste `media/` (et non `images/`). Le site `mathieu-drouet.com` utilise `images/` par héritage historique — un plan de migration est documenté dans son propre dépôt (`docs/migration-spec-v2.1.md`).
- Le nom canonique du champ texte d'introduction reste `description` (et non `intro`). Même remarque que ci-dessus.
- Les implémentations qui divergent restent fonctionnelles, mais la cible de convergence est cette spec.

**Suppression des copies internes** :
- `hyperfocale-astro-plugins/spec-hyperfocale.md` (copie partielle, masquait des omissions) — remplacée par une note pointant vers cette spec.
- `Recipes-hyperfocale/spec-hyperfocale.md` (copie obsolète) — remplacée par une note pointant vers cette spec. La variante recettes vit dans `Recipes-hyperfocale/docs/spec-hyperfocale-v2.md` (extension projet, hors source canonique).

### 2.0-draft — antérieur à 2026-05-15

Première rédaction de la spec multi-plateforme (Astro, Next.js, Hugo, 11ty, Obsidian, CMS headless, Exporter Lightroom). Architecture en trois couches (format, adaptateurs, composants UI).
