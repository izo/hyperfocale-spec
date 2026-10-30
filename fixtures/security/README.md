# Fixtures de sécurité — récupération d'URL

Cas de conformité des règles de **§1.12, *Récupération d'URL au build ou en edge*** de `spec-hyperfocale.md`. Elles portent sur des décisions pures, sans réseau ni DNS : toute implémentation d'un optimiseur d'images, d'une extraction d'EXIF ou d'un téléchargement de contenu — TypeScript, Swift ou autre — DOIT les reproduire. Même convention que `fixtures/ingestion/` : fichiers UTF-8, comparés après parsing, octets exacts (`fixtures/.gitattributes`).

| Fichier | Contenu |
|---|---|
| `remote-hosts.json` | `cases[]` : `url`, `allowlist` (la liste blanche d'hôtes du site), `expected` (`allow` ou `deny`) et `why`. L'implémentation répond `allow` seulement si l'URL est de forme admise **et** que son hôte correspond à un motif de la liste ; une liste vide refuse tout. |
| `addresses.json` | `addresses[]` : `ip`, `refuse`. Adresses **résolues** que l'outil doit refuser à chaque connexion : privées, locales, réservées, dont les métadonnées cloud `169.254.169.254` et les adresses IPv4 portées par IPv6. |

Une URL se lit telle qu'écrite : un schéma comparé sans tenir compte de la casse, pas de décodage préalable. Un hôte littéral IP est refusé **dans toute écriture** (`2130706433`, `0x7f.1`, `0177.0.0.1`) : l'implémentation s'appuie sur un analyseur d'URL qui normalise ces formes (WHATWG `URL` en JavaScript, `URLComponents` puis une vérification d'adresse en Swift), ou les refuse avant analyse.

Ce que ces fixtures ne testent pas : la résolution DNS, la vérification de l'adresse à chaque redirection et les bornes de délai et de taille, qui dépendent du réseau. Elles se testent avec un serveur local de l'implémentation.
