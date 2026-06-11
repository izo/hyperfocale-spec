# Spécification — Format de contenu Hyperfocale

> **Source de vérité canonique.** Ce document définit le format Hyperfocale — un standard de gestion de **séries photo** portable entre SSG (Astro, Next.js, Hugo, 11ty...), vaults Obsidian, et CMS headless (Strapi, Sanity, Payload...). Toute évolution du format doit être proposée d'abord ici, dans ce dépôt.

**Version** : 2.4-draft
**Statut** : spécification active — source de vérité canonique
**Dernière révision** : 2026-06-08

### Implémentations de référence

| Implémentation | Dépôt | Rôle | Conformité |
|----------------|-------|------|------------|
| Plugin Astro `@izo/hyperfocale` | https://github.com/izo/hyperfocale-astro-plugins | Adaptateur Astro (couche 2) | ✅ Conforme |
| Site `mathieu-drouet.com` | https://github.com/izo/mathieu-drouet.com | Consommateur grandeur nature (Astro) | ⚠️ Migration v2.1 prévue |
| Exporter Lightroom | https://github.com/izo/hyperfocale-exporter-app | Source de contenu : LR → format Hyperfocale | ✅ Conforme |

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
6. [[#Annexes]]
7. [[#Changelog]]

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
| `media/` | Images de la série | Obligatoire, plat (pas de sous-dossiers) |

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

Deux champs sont **obligatoires** : `title` (string) et `date` (ISO 8601).

#### Frontmatter complet

```yaml
---
# ── Core (standardisé) ──────────────────────────
title: "Bretagne 2024"
date: 2024-06-15
description: "Côtes sauvages du Finistère"
cover: "./media/01.jpg"     # fallback : première image alphabétique
location: "Finistère, France"
draft: false                # true = invisible en production
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
| **Tri des images** | Alphabétique par nom de fichier (d'où `01.jpg`, `02.jpg`...) |
| **Couverture** | `cover` du frontmatter, sinon première image alphabétique |
| **Tri des séries** | Date décroissante dans les listings |
| **Brouillons** | `draft: true` → exclu des listings en production |
| **Formats images** | `.jpg`, `.jpeg`, `.png`, `.webp`, `.avif`, `.tiff` |
| **Pas de récursion** | `media/` est plat, pas de sous-dossiers |
| **Passthrough** | Les champs inconnus du frontmatter ne provoquent pas d'erreur |

### Ce que n'est PAS le format

- Une base de données — pas de relations entre séries, pas de joins
- Un CMS — pas d'interface d'administration
- Un format propriétaire — du Markdown standard lisible par n'importe quel outil

### Contrat minimum d'un lecteur Hyperfocale

Toute implémentation (adaptateur, script, outil) qui lit du contenu Hyperfocale **doit** :

1. Parser le frontmatter YAML de `index.md`
2. Exiger `title` et `date`, ignorer les champs inconnus
3. Scanner `media/` et trier les images alphabétiquement
4. Utiliser `cover` ou la première image comme couverture
5. Exclure les entrées `draft: true` des listings publics
6. Rendre le body Markdown

---

## 0.5 — État des implémentations de référence

Section informative — un audit de conformité des implémentations connues, mis à jour à chaque révision majeure de la spec.

### Plugin Astro `@izo/hyperfocale` (v0.4.0)

**Conformité** : ✅ Conforme.

| Obligation | Statut | Note |
|------------|--------|------|
| Slug regex | ✅ | Enforced par Astro |
| `title` + `date` requis | ⚠️ | `dateRequired` est configurable via preset (par défaut conforme) |
| `description`, `cover`, `location` | ✅ | Tous optionnels, présents |
| `draft` respecté | ✅ | |
| `lang` lu | ✅ | Ajouté au schéma Zod (v0.3.0) |
| Bloc `iptc.*` | ✅ | `z.looseObject()` pour `iptc.custom.*` — champs inconnus transmis (v0.4.0) |
| Mode distant (`images[]`) | ✅ | `getSeriesImages()` détecte `images[]` ; SeriesGallery/Lightbox gèrent les URLs distantes (v0.3.0) |
| Passthrough racine (champs inconnus) | ✅ | `z.looseObject()` racine — les extensions site-spécifiques ne sont jamais rejetées (v0.4.0) |
| Tri date desc | ✅ | |

**Changements v0.4.0** :
- **Breaking** : peer dependency `zod` passe de `^3.0.0 || ^4.0.0` à `^4.0.0`. Zod 3 n'est plus supporté.
- Schéma modernisé API zod 4 : `z.looseObject()` (remplace `z.object().passthrough()`), `z.url()` (remplace `z.string().url()`).

**Extensions au-delà du contrat** :
- `featured: boolean` (boost ranking) — pattern utile, officialisé en §1.3 v2.1
- `tags: string[]` — pattern utile, officialisé en §1.3 v2.1
- `published: boolean` — redondant avec `draft`, à arbitrer
- Presets de domaine (`series`, `recipe`, `brands`, `products`, etc.) — officialisé en §2.0.1 v2.1
- Module virtuel Vite `virtual:hyperfocale/collection` — pattern d'implémentation Astro 6

### Site `mathieu-drouet.com`

**Conformité** : ✅ Conforme v2.1 (migration terminée 2026-05-19).

| Obligation | Statut | Note |
|------------|--------|------|
| Slug regex | ✅ | |
| `title` + `date` requis | ✅ | |
| Dossier `media/` | ✅ | Migré depuis `images/` — 304 dossiers (SPEC-001) |
| Champ `description` | ✅ | Migré depuis `intro` — 683 fichiers (SPEC-002) |
| `draft` respecté | ✅ | |
| Bloc `iptc.*` | ❌ | Métadonnées IPTC ignorées (perdues à l'ingestion) |
| Tri date desc | ✅ | |

**Migration v2.1** terminée sur branche `migration/spec-v2.1` (PR à merger). Historique dans `mathieu-drouet.com/docs/migration-spec-v2.1.md`.

**Extensions strictement site-spécifiques (hors spec)** :
- `private` + `password_hash` — séries protégées par mot de passe
- `featured` — officialisé en spec v2.1
- `tags` — officialisé en spec v2.1
- `artist`, `bio_source`, `genres` — pipeline de génération de bios via LLM
- Routage par collection (`series`, `series_fr`, `projects_*`, `store_*`) — pattern i18n documenté en Annexe F stratégie 3
- Champs e-commerce (`price`, `paage_url`, `source_url`, `format`, `edition`, `available`) — strictement hors spec

### Exporter Lightroom (SwiftUI + Tauri, v0.1.0)

**Conformité** : ✅ Conforme (~95 %).

| Obligation | Statut | Note |
|------------|--------|------|
| Dossier `media/` | ✅ | Les deux impls |
| Slug regex | ⚠️ | Slug pur côté output ; le format legacy `hyperfocale-{year}-{slug}-{seq}` est l'ID de session, pas le slug de dossier |
| Frontmatter core | ✅ | |
| Bloc `iptc.*` | ✅ | Mapping complet depuis le catalogue Lightroom |
| Dual naming mode | ✅ | `original` ou `sequential` (01.jpg, 02.jpg...) — officialisé en §2.7 v2.1 |
| Tri images | ⚠️ | Tri Lightroom (`customSortOrder, captureTime`), pas alphabétique — divergence intentionnelle préservant l'ordre éditorial |
| Bloc `translations:` | ⚠️ | Implémenté côté SwiftUI uniquement — divergence Tauri ↔ SwiftUI à résoudre, pattern officialisé en §2.7 v2.1 |

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

### Ce que cette spec NE définit PAS

- L'implémentation des adaptateurs (chacun a sa propre spec)
- Le design visuel (libre, seul le contrat de données est spécifié)
- Le workflow d'édition (chaque outil a le sien)

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
```

La couche 1 est normative. Les couches 2 et 3 sont des recommandations.

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
| `index.md` | Obligatoire. Contient le frontmatter YAML et le body Markdown. Encodage UTF-8. |
| `media/` | Obligatoire (peut être vide pour une série en brouillon). Plat — pas de sous-dossiers. |
| Images | Formats acceptés : `.jpg`, `.jpeg`, `.png`, `.webp`, `.avif`, `.tiff`. Pas de récursion dans `media/`. |
| Nommage des images | Libre, mais recommandé : `01.jpg`, `02.jpg`... (padding 2+ chiffres pour l'ordre). |

#### Variante : médias externes

Pour les CMS headless ou les CDN, les images peuvent être des URLs dans le frontmatter plutôt que des fichiers locaux. Voir [[#1.5 — Mode distant]].

### 1.3 — Frontmatter

Le frontmatter est en YAML, délimité par `---`. Il se divise en deux niveaux : **core** (standardisé) et **extension** (vocabulaire IPTC).

#### Core (2 champs obligatoires, 3 optionnels)

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `title` | `string` | **oui** | Titre de la série |
| `date` | `date` (ISO 8601) | **oui** | Date de la série, utilisée pour le tri. Format : `YYYY-MM-DD` |
| `description` | `string` | non | Description courte. Utilisée dans les cards, le SEO, les previews. |
| `cover` | `string` | non | Chemin relatif vers l'image de couverture (`./media/01.jpg`). Si absent : première image par ordre alphabétique. |
| `location` | `string` | non | Lieu associé à la série. Texte libre. |

#### Champs de workflow

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `draft` | `boolean` | non | `true` = série masquée en production. Défaut : `false`. |
| `lang` | `string` | non | Code langue ISO 639-1 (`fr`, `en`...). Pour les sites multilingues. |
| `featured` | `boolean` | non | `true` = série mise en avant (boost dans les listings, sections "à la une"). Défaut : `false`. *Officialisé en v2.1 à partir du pattern observé dans plusieurs implémentations.* |
| `tags` | `string[]` | non | Tags éditoriaux libres. **Distincts** de `iptc.keywords` (qui suit le vocabulaire IPTC normalisé). Voir note ci-dessous. *Officialisé en v2.1.* |

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

### 1.6 — Règles métier

#### Images

| Règle | Description |
|-------|-------------|
| Scan | Mode local : glob `media/*.{jpg,jpeg,png,webp,avif,tiff}`. Pas de récursion. |
| Tri | Alphabétique par nom de fichier. D'où le nommage recommandé `01.jpg`, `02.jpg`. |
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
| Défaut | Date décroissante (les plus récentes en premier). |
| Draft | Les séries `draft: true` sont exclues des listings en production. |

#### URLs (pour les plateformes web)

| Route | Description |
|-------|-------------|
| `/<prefix>/` | Liste de toutes les séries |
| `/<prefix>/<slug>/` | Page d'une série (body + galerie) |
| `/<prefix>/<slug>/<page>/` | Pages suivantes (à partir de la page 2) |

Le `<prefix>` est configurable par l'adaptateur (défaut recommandé : `series`).

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
| `index.md` du conteneur | Obligatoire. Mêmes règles que pour une série standard (`title`, `date` requis). |
| `media/` du conteneur | **Optionnel**. Si absent, le conteneur n'a pas de galerie propre — il sert uniquement de point d'entrée vers ses sous-séries. |
| Sous-séries | Chacune est une série complète et autonome (au sens des §1.1–1.6). Une sous-série ne peut **pas** elle-même contenir des sous-séries (pas de récursion au-delà d'un niveau). |
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
| `/<prefix>/<slug-conteneur>/` | Page du conteneur : body + galerie propre éventuelle + liste des sous-séries (line-up). |
| `/<prefix>/<slug-conteneur>/<sous-slug>/` | Page d'une sous-série : comportement standard (body + galerie). |

L'adaptateur DOIT générer ces deux niveaux d'URL automatiquement à partir de la structure filesystem.

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

---

## Couche 2 — Adaptateurs plateforme

Un adaptateur est une implémentation du format Hyperfocale pour une plateforme donnée. Chaque adaptateur DOIT respecter le contrat ci-dessous.

### 2.0 — Contrat d'adaptateur

Tout adaptateur Hyperfocale **DOIT** :

| Obligation | Description |
|-----------|-------------|
| Lire le frontmatter core | Parser `title`, `date`, `description`, `cover`, `location`, `draft`, `lang` |
| Ignorer les champs inconnus | Ne jamais échouer sur un champ frontmatter non reconnu |
| Transmettre les extensions | Rendre accessible `iptc.*` et tout champ supplémentaire aux templates/composants |
| Scanner `media/` | Mode local : glob les images, trier alphabétiquement |
| Supporter le mode distant | Si `images` est présent, l'utiliser à la place de `media/` |
| Respecter `draft` | Exclure les drafts en production |
| Trier par date desc | Listing par défaut : date décroissante |
| Exposer le body | Rendre le Markdown du body en HTML |

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

Exemple (plugin Astro `@izo/hyperfocale`) :

```ts
hyperfocale({ preset: 'series' })   // séries photo, prefix /series, date requise
hyperfocale({ preset: 'recipe' })   // recettes, prefix /recipes, date optionnelle
hyperfocale({ preset: 'brands' })   // fiches marque, prefix /brands
```

#### Contraintes des presets

Tout preset DOIT respecter le contrat d'adaptateur §2.0. Un preset PEUT modifier :
- Le nom de la collection (`collectionName`)
- Le préfixe d'URL (`prefix`)
- La conditionnalité de certains champs (ex : `dateRequired: false` pour les recettes intemporelles)
- Le vocabulaire d'extension affiché à l'utilisateur

Un preset NE DOIT PAS :
- Modifier le slug regex
- Supprimer les obligations du contrat d'adaptateur
- Renommer les champs core (`title`, `date`, `description`, `cover`, etc.)

Cinq presets non-photo sont standardisés en **Annexe G — Profils de contenu** (`event`, `recipe`, `app`, `book`, `place`). L'ajout de nouveaux profils se discute par PR contre cette spec, en étendant cette annexe.

### 2.1 — Adaptateur Astro

**Dépôt** : https://github.com/izo/hyperfocale-astro-plugins
**Forme** : intégration Astro (package npm, privé ou public).

#### Content Collection

L'intégration enregistre la collection `series` via l'API Astro 6. L'utilisateur n'écrit pas le schéma manuellement.

```ts
// Schéma Zod interne (zod 4)
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
        url: z.url(),
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
import hyperfocale from 'hyperfocale/astro';

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
| `/<prefix>/[slug]/` | Détail série + galerie |
| `/<prefix>/[slug]/[page]/` | Pagination galerie |

#### Composants exposés

```ts
import {
  SeriesCard,
  SeriesList,
  SeriesGallery,
  SeriesLightbox,
} from 'hyperfocale/astro/components';
```

#### Helpers

```ts
import {
  getSeriesList,
  getSeriesBySlug,
  getSeriesImages,
  paginateImages,
} from 'hyperfocale/astro/helpers';
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
│   └── [slug]/
│       ├── page.tsx          ← détail série
│       └── [page]/
│           └── page.tsx      ← pagination
```

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

Le plugin opère uniquement sur les dossiers qui respectent le format Hyperfocale (présence de `index.md` avec `title` + `date` + dossier `media/`). Il ignore silencieusement les autres fichiers du vault.

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
| **Détection automatique** | À l'ouverture d'un fichier `index.md` dans `<seriesRoot>/*/`, basculer en vue galerie |
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
| Structure | `translations.<lang>.<champ>` — `<lang>` est un code ISO 639-1 |
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

### 3.2 — Types de données partagés

Ces types sont la **lingua franca** entre les adaptateurs et les composants :

```ts
/** Série photo — objet retourné par les helpers */
interface Series {
  slug: string;
  title: string;
  date: Date;
  description?: string;
  cover: Image;
  location?: string;
  draft: boolean;
  lang?: string;
  body: string;           // Markdown brut ou HTML rendu, selon la plateforme
  iptc: IPTCMetadata;
  images: Image[];        // toutes les images de la série
}

/** Image — mode local ou distant */
interface Image {
  src: string;            // chemin relatif (local) ou URL (distant)
  alt?: string;
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

## Annexes

### A — Validation du format

Un outil CLI `hyperfocale-lint` est recommandé pour valider la conformité :

```bash
hyperfocale-lint ./content/series/
```

Vérifications :
- [ ] Chaque dossier série contient `index.md`
- [ ] Frontmatter contient `title` et `date`
- [ ] `date` est au format ISO 8601
- [ ] `slug` respecte le pattern `^[a-z0-9]+(-[a-z0-9]+)*$`
- [ ] `cover` pointe vers un fichier existant dans `media/`
- [ ] `media/` ne contient pas de sous-dossiers
- [ ] Les images sont dans un format accepté
- [ ] `iptc.country_code` est un code ISO 3166-1 valide (si présent)
- [ ] `iptc.gps.lat` est entre -90 et 90, `lng` entre -180 et 180 (si présent)
- [ ] Pas de mélange mode local / mode distant dans la même série

### B — Migration depuis la spec v1 (Astro-only)

Pour migrer un projet utilisant la spec v1 (Astro plugin) :

1. **Frontmatter** : déplacer `camera` sous `iptc.camera`, `tags` sous `iptc.keywords`
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

Pour les sites multilingues, trois stratégies sont supportées. Chaque projet choisit **une** stratégie et s'y tient — pas de mélange dans un même projet.

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

L'adaptateur DOIT documenter quelle(s) stratégie(s) il supporte. Un adaptateur PEUT en supporter plusieurs (le plugin Astro `@izo/hyperfocale` supporte 2 et 3).

### G — Profils de contenu (presets standardisés) *(officialisé v2.3)*

La section 0 décrit un **squelette universel** : un dossier autonome = `index.md` (frontmatter core + body Markdown) + `media/`. Jusqu'à la v2.2, ce squelette n'avait qu'une seule déclinaison : la **série photo**. Cette annexe officialise l'idée qu'un même squelette peut porter **d'autres types de contenu**.

Un **profil de contenu** est une déclinaison du squelette pour un domaine donné. Il n'invente rien au niveau structurel — il se contente de :

1. **Réutiliser le core tel quel** (`title`, `date`, `description`, `cover`, `location`, `draft`, `lang`, `tags`, `featured`).
2. **Ajouter un bloc d'extension namespacé** propre au domaine (exactement comme `iptc:` l'est pour la photo) : `event:`, `recipe:`, `app:`.
3. **Optionnellement, relâcher la conditionnalité de certains champs** via un preset d'adaptateur (§2.0.1) — typiquement `dateRequired: false`.

#### Règles communes à tous les profils

Ces règles garantissent qu'un profil reste un contenu Hyperfocale valide au sens de la §0.

| Règle | Description |
|-------|-------------|
| **Squelette inchangé** | `<slug>/index.md` + `media/` (§0, §1.2). Aucun profil ne modifie la structure filesystem ni le slug regex. |
| **Core préservé** | Les champs core ne sont jamais renommés. `title` reste obligatoire ; `date` peut devenir optionnelle via preset (contrainte §2.0.1 : pas de suppression d'obligation du contrat). |
| **Extension namespacée** | Tout vocabulaire de domaine vit sous **une** clé unique (`event`, `recipe`, `app`) pour ne jamais entrer en collision avec le core. |
| **Invariants §0 respectés** | Body avant galerie, tri alphabétique de `media/`, `cover`/fallback, `draft`, passthrough des champs inconnus s'appliquent identiquement. |
| **Rétro-compatibilité** | Un adaptateur qui ne connaît pas un profil DOIT lire le contenu comme un contenu Hyperfocale standard (core + media + body) et ignorer le bloc d'extension sans erreur (passthrough §1.3). Aucun profil ne casse un lecteur existant. |
| **Alignement externe** | Chaque profil DEVRAIT s'aligner sur un vocabulaire externe reconnu (schema.org), comme la photo s'aligne sur IPTC, pour l'interop et le balisage SEO. |
| **Sens de `media/`** | `media/` garde son rôle : un dossier plat de médias liés au contenu (affiches, photos du plat, captures d'écran...). |

#### Vue d'ensemble

| Profil | Atome | Preset | Prefix recommandé | `date` | Bloc d'extension | Vocab externe |
|--------|-------|--------|-------------------|--------|------------------|---------------|
| Série (canonique) | série photo | `series` | `/series` | requise | `iptc:` | IPTC |
| Événement | un événement | `event` | `/events` | requise (= début) | `event:` | schema.org/Event |
| Recette | une recette | `recipe` | `/recipes` | optionnelle | `recipe:` | schema.org/Recipe |
| Application | une app | `app` | `/apps` | optionnelle (= sortie) | `app:` | schema.org/SoftwareApplication |
| Livre | un livre | `book` | `/books` | optionnelle | `book:` | schema.org/Book |
| Lieu | un lieu | `place` | `/places` | optionnelle | `place:` | schema.org/Place |

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
  platform: [macos, windows]
  version: "0.1.0"
  status: beta
  category: "Photo"
  pricing: free
  url: "https://github.com/izo/hyperfocale-exporter-app"
  store:
    web: "https://github.com/izo/hyperfocale-exporter-app/releases"
  repository: "https://github.com/izo/hyperfocale-exporter-app"
  tech: [SwiftUI, Tauri, Rust]
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
| `next` | `string[]` | Slugs des écrans suivants possibles (graphe de navigation) |
| `prev` | `string` | Slug de l'écran précédent |
| `llm_context` | `string` | Chemin vers un contexte LLM lu par un assistant conversationnel quand l'écran est actif (base de connaissance contextuelle) |

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

### G.7 — Créer un nouveau profil

Pour proposer un profil supplémentaire (livre, lieu, produit e-commerce...), une PR contre cette annexe DOIT préciser :

1. **L'atome** et le cas d'usage.
2. **Le mapping du core** (sens de `title`/`date`/`cover`/`media`, `dateRequired` ou non).
3. **Le bloc d'extension namespacé** (clé unique, table de champs, tous optionnels).
4. **Le vocabulaire externe** d'alignement (schema.org ou équivalent).
5. **Le preset d'adaptateur** (collection, prefix) — dans le respect des contraintes §2.0.1.
6. La démonstration que les **invariants §0** restent satisfaits.

Un profil ne DOIT jamais : renommer un champ core, modifier le slug regex, supprimer une obligation du contrat d'adaptateur (§2.0), ou exiger une récursion dans `media/`.

---

## Changelog

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
