# TP7 — Sécurité des actions tierces

Achiban Aya — ENSAM Casablanca

## Vérification de `actions/setup-node`

Informations relevées le **20 novembre 2026** :

- Dépôt officiel : [actions/setup-node](https://github.com/actions/setup-node).
- Créateur `actions` : badge **Verified** présent sur la
  [fiche du Marketplace](https://github.com/marketplace/actions/setup-node-js-environment).
  Ce badge concerne l'identité du créateur.
- **5 028 étoiles**, valeur exacte du champ `stargazers_count` de
  [l'API GitHub](https://api.github.com/repos/actions/setup-node) lors du contrôle.
  Le nombre peut évoluer après cette date.
- Sources TypeScript : dossier [`src/`](https://github.com/actions/setup-node/tree/main/src).
  Le dossier `dist/` contient le JavaScript distribué et exécuté par l'action.

## Expérience : `checkout@main` puis `checkout@v4`

Le workflow local a été temporairement modifié pour utiliser
`actions/checkout@main`, contrôlé avec `actionlint v1.7.12`, puis rétabli à
`actions/checkout@v4` et contrôlé de nouveau. Le contenu initial a été restauré
à l'identique. Cette expérience est un contrôle local, sans exécution GitHub
Actions de la variante `@main`.

Références et métadonnées officielles observées le 20 novembre 2026 :

| Référence | Commit observé | Moteur déclaré dans `action.yml` |
| --- | --- | --- |
| `main` | `f548e57e544e1ff5a4c46bf1e1b8685f8e4a348a` | `node24` |
| `v4` | `11d5960a326750d5838078e36cf38b85af677262` | `node20` |

Sources : [référence main](https://api.github.com/repos/actions/checkout/git/ref/heads/main),
[référence v4](https://api.github.com/repos/actions/checkout/git/ref/tags/v4),
[métadonnées main](https://github.com/actions/checkout/blob/main/action.yml),
[métadonnées v4](https://github.com/actions/checkout/blob/v4/action.yml).
Le moteur interne de checkout est distinct de la version de Node.js installée
pour le projet par `setup-node`.

## Q3 — Pourquoi utiliser `@v4` plutôt que `@main` ?

`@main` suit directement une branche de développement : son contenu peut changer
et introduire une incompatibilité à la prochaine exécution. `@v4` sélectionne
la version majeure demandée par le TP, avec une interface plus prévisible.
Ce tag reste déplaçable : il ne garantit pas une exécution strictement identique.
En production, il est préférable d'épingler un **SHA de commit complet vérifié**
et de gérer explicitement ses mises à jour.

## Q4 — Le badge « créateur vérifié » garantit-il la sécurité ?

Non. GitHub a vérifié l'identité de l'organisation créatrice, pas l'absence de
failles dans chaque version de son code ou de ses dépendances. Une action d'un
créateur vérifié peut comporter une vulnérabilité ou être compromise.

## Q5 — Quelles sont les deux vérifications avant la production ?

1. **Vérifier le fournisseur et la maintenance** : identité du propriétaire,
   historique des versions, activité récente et traitement des alertes de
   sécurité. Les étoiles seules ne constituent pas une garantie.
2. **Examiner le code de la version exacte et ses accès** : `action.yml`, scripts,
   dépendances, permissions, secrets et communications réseau. Limiter les droits
   accordés et épingler le SHA complet du code contrôlé.

Référence : [recommandations de sécurité GitHub Actions](https://docs.github.com/en/actions/reference/security/secure-use#using-third-party-actions).
