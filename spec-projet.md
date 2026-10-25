# Spécification — Hyperfocale Projet

> **Esquisse.** Ce document décrit le **dossier de travail** Hyperfocale : un projet suivi à la source, de la première prise de contact à la livraison. Il se sépare de la spec web (`spec-hyperfocale.md`), qui décrit un contenu fait pour être publié. Rien n'y est encore normatif : les décisions à trancher sont en §D. Réflexion d'origine : [#36](https://github.com/izo/hyperfocale-spec/issues/36).

**Version** : 0.1-draft
**Statut** : esquisse — non normative
**Dernière révision** : 2026-10-25
**Socle** : `spec-hyperfocale.md` 2.12-draft. La 2.13-draft en cours ([#37](https://github.com/izo/hyperfocale-spec/issues/37)) règle trois points dont ce document dépend : `date` portée par la collection, `media/` facultatif sans fichier local, ordre alphabétique défini.

---

## Table des matières

0. Relation au socle
1. Le dossier projet
2. Frontmatter
3. Lots et sous-lots
4. Outils de gestion et agents
5. Export vers la spec web
6. Interopérabilité
7. Compatibilité avec les dossiers G.12
8. Fixtures
D. Décisions ouvertes
Changelog

---

## 0 — Relation au socle

La spec web impose à tout contenu des contraintes nées de la publication : slug utilisable en URL, `date` requise pour le tri, imbrication limitée à un niveau pour les routes, `draft` comme garde-fou. Le profil G.12 (v2.11) fait entrer un dossier de travail dans ces contraintes et les contourne une à une. Ce document les lève pour le dossier de travail, et garde le reste.

| | Socle commun | Spec web | Spec Projet |
|---|---|---|---|
| Unité | dossier + `index.md` (frontmatter + body) + `media/` | contenu publiable | dossier de travail |
| `media/` | plat ; classes image / document joint / original (§1.9 web) | galerie, documents joints | idem, plus des **dossiers de travail** à côté (§1.3) |
| Nom de dossier | — | slug obligatoire | **libre** : le nom du dossier fait l'identité |
| `title` | — | requis | optionnel, nom du dossier par défaut |
| `date` | — | requise par défaut | optionnelle (`project.start`, `project.due`…) |
| Profondeur | — | rangement libre, imbrication sur un niveau | libre |
| Champs inconnus | conservés (passthrough), body préservé | | |
| Passage de l'un à l'autre | | reçoit | **export** (§5), seul point de contact |

**Repris du socle sans changement** : `index.md` en YAML + Markdown, `media/` plat, classification des fichiers par extension en minuscules et fichiers système ignorés (2.13-draft), originaux (§1.9 web), passthrough des champs inconnus, aucune relation entre contenus.

*À rédiger : la liste exacte des sections de la spec web qui s'appliquent telles quelles, et de celles qui ne s'appliquent pas.*

---

## 1 — Le dossier projet

```
Projets/                        ← rangement, profondeur libre
└── clients/Acme/
    └── Refonte site/           ← projet
        ├── index.md
        ├── media/              ← partie montrable : images, devis, comptes rendus
        ├── Maquettes/          ← lot (§3)
        │   ├── index.md
        │   └── media/
        └── sources/            ← dossier de travail, sans index.md
            └── maquette.psd
```

### 1.1 — Discriminant

*Piste* : `type: project` dans `index.md` (décision D1). Il voyage avec le dossier quand on le déplace ; l'ingestion web le refuse déjà (`type-invalid`, §4.10 web), donc un dossier projet confié par erreur à un pipeline web bloque la publication.

### 1.2 — Nom du dossier

*Piste* : libre, contraint seulement par les règles de chemin du socle (§4.1 web : NFC, pas de caractère de contrôle). Le nom du dossier sert de titre quand `title` est absent ; l'export le convertit en slug (§5).

### 1.3 — Dossiers de travail

Un sous-dossier **sans** `index.md` dans un projet est un dossier de travail : il fait partie du projet et se déplace avec lui, un lecteur l'ignore (ni galerie, ni documents joints, ni couverture), et il ne sort jamais du projet — l'export ne le reprend pas.

*À rédiger : son statut pour un outil d'ingestion (`kind: other`, couche 4 web).*

---

## 2 — Frontmatter

### 2.1 — Core assoupli

| Champ | Dans un projet |
|---|---|
| `title` | Optionnel ; à défaut, le nom du dossier |
| `date` | Optionnelle ; un projet au stade `prospect` n'a pas encore de date |
| `description`, `cover`, `location`, `lang`, `tags` | Sens du socle |
| `draft` | Sans objet dans la spec Projet ; écrit `true` tant que la compatibilité G.12 l'exige (§7) |

### 2.2 — Bloc `project:`

Repris de G.12, tous les champs optionnels : `status`, `stage`, `client`, `contact`, `start`, `end`, `due`, `deliverables` (`name`, `date`), `tasks` (`title`, `done`, `due`).

*Pistes d'ajout* :

| Clé | Où | Rôle |
|---|---|---|
| `next` | projet | Une seule prochaine action, concrète |
| `owner` | projet | Responsable, en texte libre |
| `priority` | projet, tâche | Slug ASCII : `basse` · `normale` · `haute` |
| `assignee` | tâche | Personne en charge, en texte libre |
| `stage` | tâche | Étape libre, pour une vue en colonnes |
| `ref` | tâche, livrable | URL externe (issue, maquette) — seul lien admis |
| `done` | livrable | Livré ou non ; lève l'ambiguïté de `date` |

### 2.3 — Statuts

Repris de G.12 : `prospect` · `en-cours` · `livre` · `archive`. *Pistes d'ajout* : `en-pause` (une pause est un état normal, pas un échec) et `abandonne`. Aucun statut existant n'est renommé. Un statut absent reste « non renseigné » et ne se déduit jamais des dates.

### 2.4 — Écarté

Dépendances entre tâches ou projets (relations) ; avancement stocké (donnée dérivée des tâches) ; comptes, permissions, suivi du temps ; tâches en cases à cocher dans le body, que les outils préservent à l'octet.

---

## 3 — Lots et sous-lots

Un sous-dossier **avec** `index.md` est un lot. *Pistes* : un lot porte son propre bloc `project:` ; il n'hérite de rien de son parent ; le statut d'un projet ne se calcule jamais à partir de ses lots ; la profondeur est libre (décision D4).

---

## 4 — Outils de gestion et agents

Repris de G.12 : un outil qui réécrit `index.md` préserve les champs inconnus — à la racine, dans `project:` et dans chaque entrée de `deliverables` et `tasks` — et le body à l'octet ; il DEVRAIT préserver l'ordre des clés et les commentaires YAML.

*Pistes* :

- **Opérations normatives** : créer un projet, changer le statut, ajouter une tâche, la cocher, définir `next`, ajouter une entrée au journal, archiver, exporter. Chacune décrite par un `index.md` avant / après, pour que l'app macOS, l'app web et les outils d'un agent produisent le même fichier.
- **Consentement** : trois niveaux — lire, proposer, écrire. Un agent travaille en *proposer* par défaut ; aucun outil ne change un statut, ne supprime ni n'exporte sans action explicite de l'utilisateur.
- **Boîte de réception** : un dossier `inbox/` à la racine pour noter sans décider où ranger.

---

## 5 — Export vers la spec web

Repris de G.12 et généralisé. L'export crée un **nouveau** dossier, contenu valide de la spec web :

- slug tiré du nom du dossier projet ou choisi ;
- `media/` d'images seulement — ni originaux, ni documents de travail, ni dossiers de travail ;
- frontmatter repris au choix (`title`, `date`, `description`, `cover`, `location`, `lang`, `tags`, `iptc:`), **sans le bloc `project:`** ;
- profondeur aplatie : un lot s'exporte comme une série à part entière.

*Piste* : cible série **ou** portfolio (G.7 web) — `project.client` → `portfolio.client`, statut livré → `portfolio.status: delivered`.

---

## 6 — Interopérabilité

- Projet ↔ `schema.org/CreativeWork` (`project.status` → `creativeWorkStatus`), comme G.12.
- *Piste* : tâches ↔ iCalendar VTODO — `title` → `SUMMARY`, `due` → `DUE`, `done` → `STATUS:COMPLETED`, `priority` → `PRIORITY` — pour un export `.ics`.

---

## 7 — Compatibilité avec les dossiers G.12

*À rédiger* : un dossier G.12 existant (sans `type`, `draft: true`) reste lisible ; migration vers `type: project` ; G.12 devient dans la spec web un renvoi vers ce document.

---

## 8 — Fixtures

*À décider* : si le contrat d'export (§5) et les opérations (§4) deviennent normatifs, ils prennent des fixtures, dans `fixtures/projet/`, sur le modèle de `fixtures/ingestion/`.

---

## D — Décisions ouvertes

| # | Décision | Piste |
|---|---|---|
| D1 | Discriminant | `type: project` |
| D2 | Socle | `spec-hyperfocale.md` porte le socle, ce document s'y réfère ; extraction d'un socle autonome plus tard |
| D3 | Dépôt et versions | Fichier à part, version propre (`0.x-draft`), qui cite la version web dont il dépend ; développé sur la branche `spec/projet` |
| D4 | Profondeur des lots | Libre, l'export aplatit |
| D5 | Nom de dossier libre | Contraint seulement par §4.1 web ; slug produit à l'export |
| D6 | `title` optionnel | Nom du dossier par défaut |

---

## Changelog

### 0.1-draft — 2026-10-25

Esquisse initiale, tirée de la réflexion [#36](https://github.com/izo/hyperfocale-spec/issues/36) : découpage socle / web / projet, structure du dossier, bloc `project:` et pistes d'ajout, outils de gestion et agents, export, décisions ouvertes. Rien n'est normatif.
