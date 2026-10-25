# Spécification — Hyperfocus

> **En rédaction.** **Hyperfocus** est le format du **dossier de travail** de l'écosystème Hyperfocale : un projet suivi à la source, de la première prise de contact à la livraison. Il se sépare de la spec web (`spec-hyperfocale.md`), qui décrit un contenu fait pour être publié. Les sections 1 et 2 sont rédigées au futur contrat (DOIT / PEUT) mais rien n'est encore figé : tant que le statut est « en rédaction », les décisions sont réversibles (§D). Réflexion d'origine : [#36](https://github.com/izo/hyperfocale-spec/issues/36).

**Version** : 0.4-draft
**Statut** : en rédaction — §1 et §2 rédigés, le reste est une esquisse
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

| | Socle commun | Hyperfocale (spec web) | Hyperfocus |
|---|---|---|---|
| Unité | dossier + `index.md` (frontmatter + body) + `media/` | contenu publiable | dossier de travail |
| `media/` | plat ; classes image / document joint / original (§1.9 web) | galerie, documents joints | idem, plus des **dossiers de travail** à côté (§1.6) |
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

### 1.1 — Définition

Un **projet** est un dossier de travail : un dossier dont l'`index.md` déclare `type: project` (§1.2). Il porte ses fichiers, ses notes et ses informations de suivi. Il n'est pas fait pour être publié : il se publie par export (§5).

```
Projets/                        ← rangement, profondeur libre
└── clients/Acme/
    └── Refonte site/           ← projet
        ├── index.md            ← type: project
        ├── media/              ← partie montrable : images, devis, comptes rendus
        ├── Maquettes/          ← lot (§3) : index.md avec type: project
        │   ├── index.md
        │   └── media/
        └── sources/            ← dossier de travail (§1.5) : pas d'index.md projet
            └── maquette.psd
```

### 1.2 — Discriminant *(décision D1)*

| Règle | Description |
|-------|-------------|
| Déclaration | Un dossier est un projet si, et seulement si, son `index.md` déclare `type: project`. |
| Jamais deviné | Un outil NE DOIT PAS reconnaître un projet à la présence d'un bloc `project:`, à l'absence de `date` ou à la forme du dossier : seul `type` compte, comme pour une section de la spec web (§1.10 web). |
| Un fichier, un rôle | `type` n'admet qu'une valeur par fichier : un `index.md` de projet n'est ni une série ni une section. |
| Dossier voisin | Un `index.md` sans `type: project`, rencontré dans un projet, n'est pas un projet — par exemple l'export HTML d'un site rangé dans un dossier de travail. |
| Lecteur de la spec web | Une ingestion web refuse la valeur (`type-invalid`, §4.10 web) et bloque donc la publication d'un dossier projet confié par erreur à un pipeline web. Un adaptateur web qui ne connaît pas la valeur la traite comme un champ inconnu (passthrough) et lit le dossier comme une série ; c'est pourquoi un outil qui crée un projet écrit aussi `draft: true`, aussi longtemps que cette compatibilité compte (§7). |

### 1.3 — Nom du dossier *(décision D5)*

Le **nom du dossier** identifie le projet. Il est libre : la regex de slug de la spec web (§1.2 web) ne s'applique pas, puisqu'un projet n'a pas d'URL.

| Règle | Description |
|-------|-------------|
| Contraintes | Celles d'un segment de chemin du socle (§4.1 web) : forme NFC, aucun caractère de contrôle, aucun `/`. Un nom qui commence par `.` est un fichier système (§4.2 web) et n'est pas un projet. |
| Espaces, accents, majuscules | Admis et préservés. `Refonte site` est un nom valide. |
| Collision | Deux dossiers frères dont les noms sont égaux après NFC puis minuscule simple (§4.1 web) sont en collision : le projet ne s'ouvre pas de façon fiable sur Dropbox ou APFS, qui sont insensibles à la casse. |
| Identité | Le chemin du dossier relatif à la racine de l'espace de projets. Deux projets de même nom rangés à des endroits différents sont distincts. |
| Renommer | Renommer le dossier change l'identité du projet. C'est une opération de l'utilisateur, jamais d'un outil. |

### 1.4 — Titre *(décision D6)*

`title` est facultatif. Le **titre affiché** d'un projet est son `title` s'il est présent et non vide ; sinon le nom du dossier, tel quel — sans transformation de casse ni de ponctuation. Un outil qui écrit `title` NE DOIT PAS le dériver silencieusement du nom du dossier : un `title` absent reste absent.

### 1.5 — Contenu d'un projet

| Élément | Rôle |
|---------|------|
| `index.md` | Frontmatter (§2) et notes libres. Obligatoire. |
| `media/` | Partie montrable : images, documents, originaux. Plat, facultatif s'il est vide (§1.2 web). Les classes de fichiers du socle s'appliquent (§1.9 web). |
| Lot | Sous-dossier dont l'`index.md` déclare `type: project` (§3). |
| Dossier de travail | Tout autre sous-dossier que `media/` : il appartient au projet (§1.6). |

Un dossier sans `index.md` projet est un **rangement** quand il contient un projet ou un lot à une profondeur quelconque, et un **dossier de travail** sinon. Le critère est mécanique : il ne dépend ni du nom du dossier ni de son contenu en fichiers.

### 1.6 — Dossiers de travail

Un dossier de travail (`sources/`, `admin/`, `rendus/`…) fait partie du projet : il se déplace avec lui, avec tout ce qu'il contient, sous-dossiers compris — la profondeur y est libre, contrairement à `media/`.

| Règle | Description |
|-------|-------------|
| Lecture | Un lecteur l'ignore : ni galerie, ni documents joints, ni couverture. Ses fichiers ne sont jamais des médias du projet. |
| Opaque | Son contenu n'est pas interprété. Un `index.md` qui s'y trouve sans `type: project` n'est pas lu comme un contenu. |
| Export | L'export (§5) ne le reprend jamais. |
| Préservation | Un outil de gestion NE DOIT PAS y modifier, y renommer ni y supprimer de fichier de lui-même. |

### 1.7 — Contrat minimum d'un lecteur Projet

Un outil qui lit des projets DOIT :

1. parser le frontmatter YAML de `index.md` et conserver les champs inconnus (passthrough) ;
2. reconnaître un projet à `type: project`, et à rien d'autre (§1.2) ;
3. calculer le titre affiché (§1.4) ;
4. ne jamais échouer sur un champ, un statut ou un fichier inconnu ;
5. traiter `media/` selon le socle — absent, il vaut un `media/` vide — et ignorer les dossiers de travail (§1.6) ;
6. ne jamais publier un projet tel quel (§5).

---

## 2 — Frontmatter

### 2.1 — Core assoupli

| Champ | Dans un projet |
|-------|----------------|
| `type` | **Requis**, valeur `project` (§1.2). |
| `title` | Facultatif ; à défaut, le nom du dossier (§1.4). |
| `date` | Facultative : un projet `prospect` n'a pas encore de date. Présente, elle DOIT être une date valide ; elle sert au tri (date décroissante) quand elle existe. |
| `description`, `location`, `lang`, `tags` | Sens du socle (§1.3 web). |
| `cover` | Une image de `media/` — jamais un original, jamais un fichier d'un dossier de travail (§1.9 web). |
| `draft` | Sans objet pour Hyperfocus, qui ne publie rien. Un outil qui crée un projet écrit `draft: true` explicitement, tant que des lecteurs web qui ignorent `type: project` peuvent croiser le dossier (§1.2, §7). |

### 2.2 — Bloc `project:`

Les données de gestion vivent **dans le frontmatter de `index.md`**, sous la clé unique `project:` — pas dans un fichier voisin. Un projet reste un dossier autonome : le déplacer, c'est déplacer toutes ses données.

Tous les champs sont optionnels. Un outil DOIT ignorer sans erreur ce qu'il ne supporte pas.

| Clé | Type | Description |
|-----|------|-------------|
| `status` | `string` | Cycle de vie (§2.3) |
| `stage` | `string` | Étape du workflow, **libre** (« tri », « retouche », « livraison »…). La spec ne fixe aucune liste ; la valeur n'est jamais validée. |
| `next` | `string` | La **prochaine action**, une seule, en texte libre (§2.2.1) |
| `client` | `string` | Commanditaire |
| `contact` | `string` | Contact chez le client, en texte libre (nom, e-mail, téléphone) |
| `start` | `date` | Début du projet (pour un projet photo, de la prise de vue — sens de G.12) |
| `end` | `date` | Fin de la production (pour un projet photo, de la prise de vue — sens de G.12) |
| `due` | `date` | Échéance de livraison |
| `deliverables` | `object[]` | Livrables, dans l'ordre du tableau : `name` (`string`, requis), `date` (`date`, livraison prévue ou effective) |
| `tasks` | `object[]` | Tâches, dans l'ordre du tableau : `title` (`string`, requis), `done` (`boolean`, défaut `false`), `due` (`date`) |

| Règle | Description |
|-------|-------------|
| Dates | `AAAA-MM-JJ`, comme `date` (§1.3 web). Aucun contrôle de cohérence n'est imposé entre `start`, `end` et `due` : un outil PEUT signaler `end` avant `start`, jamais le refuser. |
| Ordre | L'ordre des tableaux `deliverables` et `tasks` fait foi. Aucun tri n'est appliqué. |
| Entrée incomplète | Une entrée de `deliverables` sans `name`, ou de `tasks` sans `title`, DEVRAIT être signalée par un outil de gestion et ignorée à l'affichage — jamais un échec. |
| Valeur mal typée | Une valeur d'un type inattendu (`due: "bientôt"`, `done: "oui"`) vaut « non renseignée » pour l'affichage, le tri et les filtres ; elle est conservée telle quelle à la réécriture et un outil DEVRAIT la signaler. |
| Champs inconnus | Conservés partout : à la racine du frontmatter, dans `project:` et dans chaque entrée de `deliverables` et de `tasks` (passthrough). |
| Confidentialité | Le bloc `project:` (client, contact, échéances) est privé. Un projet n'est pas rendu hors de l'outil de gestion (§5), et aucun balisage public ne l'émet. |

#### 2.2.1 — La prochaine action *(décision D9)*

`project.next` désigne **une** action concrète, la prochaine : « Envoyer la sélection à Claire », pas « Livraison ».

| Règle | Description |
|-------|-------------|
| Une seule | `next` est une chaîne, jamais une liste. Plusieurs actions relèvent de `tasks`. |
| Indépendante | Elle ne se calcule pas depuis `tasks` : ce n'est pas la première tâche non faite, c'est ce que l'utilisateur décide de faire ensuite. Un outil PEUT en proposer une ; il ne l'écrit jamais de lui-même (§4). |
| Absente | Absente ou vide, il n'y a pas de prochaine action renseignée. Ce n'est pas une anomalie. |
| Effacée | Une action faite s'efface (ou se remplace) : `next` ne garde pas d'historique. Le journal du projet est le body. |

### 2.3 — Statuts

| Valeur | Sens |
|--------|------|
| `prospect` | Projet envisagé : premier contact, devis, date posée en option. Rien n'est confirmé. |
| `en-cours` | Projet confirmé, de la prise de vue à la livraison : tri, retouche, export. |
| `en-pause` | Projet confirmé, **suspendu** : aucune action n'est attendue avant sa reprise. Une pause est un état normal, pas un échec. |
| `livre` | Livrables remis. Le projet peut encore recevoir une retouche ou donner lieu à une publication. |
| `archive` | Projet clos, conservé pour mémoire. Plus aucune action attendue. |
| `abandonne` | Projet arrêté **avant sa livraison** et qui ne reprendra pas. Il se distingue d'`archive`, qui clôt un projet mené à terme. |

| Règle | Description |
|-------|-------------|
| Slugs ASCII | Ni accent ni majuscule, pour rester stables entre outils et comparables à l'octet. Le libellé affiché est libre (« En pause », « Abandonné »). |
| Pas de machine à états | La spec ne définit aucune transition : un projet peut passer de `en-pause` à `en-cours`, ou de `abandonne` à `prospect`. Les statuts décrivent un état, pas un parcours imposé. |
| Absent | `status` absent signifie *non renseigné*. Un outil NE DOIT PAS le déduire des dates, des tâches ni d'un autre champ. |
| Inconnu | Une valeur inconnue est conservée et ne fait jamais échouer un outil, qui l'affiche comme non renseignée. |
| Compatibilité | Les quatre valeurs de G.12 (`prospect`, `en-cours`, `livre`, `archive`) ne changent ni de forme ni de sens ; `en-pause` et `abandonne` sont additives. Un outil qui ne connaît que G.12 les traite en valeurs inconnues. |
| Hors publication | Le statut est indépendant de `draft` : un projet `livre` reste un brouillon tant qu'il n'a pas été exporté. |

*Pistes non retenues, en attente d'un besoin réel* : `owner` (responsable), `priority` (slug `basse` · `normale` · `haute`, projet et tâche), `assignee` et `stage` sur une tâche, `ref` (URL externe d'un ticket ou d'une maquette, seul lien admis), `done` sur un livrable (lève l'ambiguïté de `date`, « prévue ou effective »). Aucun n'est nécessaire à un projet mené seul.

### 2.4 — Écarté

Dépendances entre tâches ou projets (relations) ; avancement stocké (donnée dérivée des tâches) ; comptes, permissions, suivi du temps ; tâches en cases à cocher dans le body, que les outils préservent à l'octet.

---

## 3 — Lots et sous-lots

Un sous-dossier dont l'`index.md` déclare `type: project` est un lot (§1.5, décision D7) ; un `index.md` sans cette déclaration n'en fait pas un lot. *Pistes* : un lot porte son propre bloc `project:` ; il n'hérite de rien de son parent ; le statut d'un projet ne se calcule jamais à partir de ses lots ; la profondeur est libre (décision D4).

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
| D1 | Discriminant | **Retenue (0.2)** : `type: project`, seul discriminant (§1.2) |
| D2 | Socle | `spec-hyperfocale.md` porte le socle, ce document s'y réfère ; extraction d'un socle autonome plus tard |
| D3 | Dépôt et versions | Fichier à part, version propre (`0.x-draft`), qui cite la version web dont il dépend ; développé sur la branche `spec/projet` |
| D4 | Profondeur des lots | Libre, l'export aplatit |
| D5 | Nom de dossier libre | **Retenue (0.2)** : contraint seulement par §4.1 web ; slug produit à l'export (§1.3) |
| D6 | `title` optionnel | **Retenue (0.2)** : nom du dossier par défaut, jamais écrit en retour (§1.4) |
| D7 | Un lot déclare-t-il `type: project` ? | **Retenue (0.2)** : oui — c'est ce qui distingue un lot d'un `index.md` quelconque dans un dossier de travail (§1.5). À confirmer à l'usage |
| D8 | Dossiers de travail et ingestion web | Ouverte : la spec web (§4.3, §4.11) les classerait `other` et les publierait. Un projet n'étant jamais ingéré (§1.7), la question ne se pose que par erreur ; la garde pourrait les exclure comme les originaux |
| D9 | `next`, `en-pause`, `abandonne` | **Retenues (0.3)** : la prochaine action unique (§2.2.1) et deux statuts additifs (§2.3). Les autres pistes d'ajout attendent un besoin réel |
| D10 | Périmètre des composants « projets » et « photos » d'Hyperfocale à porter dans Hyperfocus | Ouverte : G.12 seul, ou aussi les originaux (§1.9 web) et l'exporter Lightroom (§2.7 web) ? Voir le message de cadrage dans #36 |

---

## Changelog

### 0.4-draft — 2026-10-25

**Renommage** : la spec s'appelle **Hyperfocus**, pour la distinguer d'**Hyperfocale**, la spec web. `spec-projet.md` devient `spec-hyperfocus.md`. Hyperfocale décrit un contenu fait pour être publié ; Hyperfocus décrit le travail qui le précède — projets et photos, à la source.

**Intention enregistrée** : les composants « projets » et « photos » d'Hyperfocale doivent être implémentés dans Hyperfocus. Leur périmètre exact est la décision D10, ouverte.

### 0.3-draft — 2026-10-25

**Rédigé** : §2.2 (bloc `project:`, règles de dates, ordre, entrées incomplètes, valeurs mal typées, confidentialité), §2.2.1 (prochaine action) et §2.3 (statuts).

**Décisions retenues** :
- **D9** — `project.next`, la prochaine action : une seule, en texte libre, jamais calculée depuis `tasks` et jamais écrite par un outil de lui-même.
- **`en-pause`** : projet confirmé et suspendu, état normal. **`abandonne`** : arrêté avant livraison, distinct d'`archive`. Les quatre statuts de G.12 sont inchangés ; aucune transition n'est définie.
- **Une valeur mal typée vaut « non renseignée »** pour l'affichage et les filtres, est conservée à la réécriture et est signalée — comme une valeur de statut inconnue.

**Écarté pour l'instant** : `owner`, `priority`, `assignee`, `stage` de tâche, `ref`, `done` de livrable — aucun besoin pour un projet mené seul.

### 0.2-draft — 2026-10-25

**Rédigé** : §1 (définition, discriminant, nom du dossier, titre, contenu, dossiers de travail, contrat minimum d'un lecteur) et §2.1 (core assoupli).

**Décisions retenues** :
- **D1** — `type: project` est le seul discriminant d'un projet ; il n'est jamais deviné.
- **D5** — le nom du dossier est libre, contraint seulement par les règles de chemin du socle ; il fait l'identité du projet.
- **D6** — `title` est facultatif, le nom du dossier tient lieu de titre affiché, et aucun outil ne l'écrit en retour.
- **D7** — un lot déclare `type: project`, ce qui distingue lot, rangement et dossier de travail par un critère mécanique.

**Ouvert** : D8 (dossiers de travail face à l'ingestion web).

### 0.1-draft — 2026-10-25

Esquisse initiale, tirée de la réflexion [#36](https://github.com/izo/hyperfocale-spec/issues/36) : découpage socle / web / projet, structure du dossier, bloc `project:` et pistes d'ajout, outils de gestion et agents, export, décisions ouvertes. Rien n'est normatif.
