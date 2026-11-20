# TP7 — Actions tierces et actions personnalisées

**Étudiante :** Achiban Aya — **Établissement :** ENSAM Casablanca  
**Dépôt :** `ashAya758jv/recettes-duo` — **Branche :** `feature/tp-actions`

Les réponses du TP6 sont conservées dans leur branche d'origine :
[REPONSES.md du TP6](https://github.com/ashAya758jv/recettes-duo/blob/feature/tp-env/REPONSES.md).
Ce fichier était absent de `main` au départ du TP7 ; aucun contenu antérieur n'a
été effacé et aucune ancienne branche n'a été modifiée.

## Q1 — Pourquoi checkout n'a-t-il pas nécessairement besoin de `with` ?

Les inputs de `actions/checkout` ont des valeurs par défaut : l'action récupère
le dépôt courant et la référence correspondant à l'événement du workflow.
`with` est utile pour changer ce comportement, par exemple pour choisir une
autre branche. Pour `setup-node`, `with: node-version: '20'` sert à demander
explicitement la version de Node.js imposée par le TP.

## Q2 — Pourquoi verify ne retrouve-t-il pas report.txt sans artefact ?

Sur les runners GitHub hébergés utilisés ici, chaque job dispose de sa propre
machine et de son propre espace de travail. `report.txt` est créé pendant `build`
et ne fait pas partie des fichiers du dépôt. `needs: build` fait attendre `verify`,
mais ne partage pas son disque. Il faut publier `build-report` avec
`upload-artifact`, puis le récupérer avec `download-artifact` pour pouvoir lire
le même fichier dans `verify`.

## Q6 — Pourquoi checkout doit-il précéder une action locale ?

`uses: ./.github/actions/build-info` et `uses: ./.github/actions/docker-hello`
cherchent leurs fichiers dans l'espace de travail du runner. Le checkout y
récupère le dépôt, notamment les fichiers `action.yml`, le Dockerfile et le
script. Sans lui, ces chemins locaux ne sont pas disponibles dans ce job.

## Q7 — Pourquoi préciser `shell: bash` dans une action composite ?

Chaque step `run` d'une action composite doit déclarer le shell qui exécutera
ses commandes. Le runner ne déduit pas ce choix des paramètres du workflow
appelant. `shell: bash` donne l'interpréteur utilisé par les deux scripts et
permet notamment d'écrire l'output dans `"$GITHUB_OUTPUT"`.

Les réponses Q3 à Q5 figurent dans [SECURITY-NOTES.md](SECURITY-NOTES.md).

## Fichiers et comportement attendus

- `.github/workflows/tp-actions.yml` : déclenchement manuel, jobs `build`,
  `verify` et `docker-hello`, permissions limitées à `contents: read`.
- `.github/actions/build-info/action.yml` : affiche `Building my-app v1.0.0`
  et renvoie `build-id=my-app-1.0.0`. La step appelante affiche cet identifiant
  et vérifie sa valeur.
- `.github/actions/docker-hello/` : conteneur `alpine:3.20`, script exécutable
  avec fins de ligne LF, input `who` et message `Hello, Aya` dans le workflow.
- `build-report` : contient `report.txt`, avec la version de Node.js et une date
  UTC. L'upload échoue si le fichier est absent ; `verify` télécharge cet artefact
  par son nom puis affiche le fichier avec `cat report.txt`.

Les inputs des actions et l'output composite passent par des variables
d'environnement avant leur utilisation dans les scripts Bash.

## Validation locale réalisée le 20 novembre 2026

- `actionlint v1.7.12` : workflow valide.
- Les trois fichiers YAML sont analysés ; chemins locaux, ordre du checkout,
  dépendance `verify → build` et correspondance des noms d'artefacts vérifiés.
- Scripts Bash exécutés : identifiant par défaut `my-app-1.0.0`, version
  personnalisée `2.3.4`, conservation des outputs déjà présents et traitement
  littéral d'un nom contenant `$(false)` vérifiés.
- Génération et lecture de `report.txt` vérifiées avec le Node.js local.
  L'installation de Node.js 20 par `setup-node` reste à valider sur le runner.
- Image Docker réellement construite ; `docker run --rm tp7-docker-hello:local Aya`
  affiche exactement `Hello, Aya`.
- Tous les fichiers déjà présents dans `main`, y compris les workflows du TP6
  et les recettes, sont conservés sans modification.

**Statut : validation GitHub Actions encore en attente.** Les contrôles locaux
ne prouvent pas encore le transfert réel de l'artefact entre deux jobs GitHub.
Aucune durée distante et aucune capture d'exécution réussie ne sont inventées.

## Lancement manuel sur GitHub

GitHub exige que le workflow soit présent sur la branche par défaut pour activer
`workflow_dispatch`. Au contrôle initial, `main` ne contenait pas `tp-actions.yml`.
Une copie de ce seul fichier est préparée pour publication sur `main`, soumise
à l'autorisation explicite de l'étudiante. Aucune fusion de branche n'est requise.

Après cette autorisation et la publication du seul fichier de workflow :

1. Ouvrir [Actions du dépôt](https://github.com/ashAya758jv/recettes-duo/actions).
2. Choisir **Third-Party and Custom Actions Lab**, puis **Run workflow**.
3. Sélectionner impérativement **feature/tp-actions** dans **Use workflow from**.
   Les actions locales sont présentes sur cette branche.
4. Lancer le workflow et attendre la réussite de `build`, `verify` et `docker-hello`.
5. Dans `build`, vérifier `Print build id` et `Upload build report`. Dans `verify`,
   vérifier `Download build report` puis le contenu de `Print build report`.
6. Dans `docker-hello`, vérifier le message `Hello, Aya`.
7. Dans le résumé, vérifier la présence de **build-report** dans **Artifacts**,
   le télécharger et contrôler son fichier `report.txt`.

## Comparaison des durées : à compléter avec une exécution réelle

Relever les horodatages `started_at` et `completed_at` des steps dans l'API des
jobs d'une même exécution réussie, ou les durées affichées dans les journaux :

| Élément | Statut de la mesure GitHub Actions |
| --- | --- |
| `Run build info action` | En attente d'une exécution réelle |
| `Run Docker hello action` | En attente d'une exécution réelle |
| Construction de l'image Docker | En attente ; peut apparaître dans la préparation du job |

La construction de l'image et le démarrage du conteneur ajoutent du travail
par rapport à une action composite. Il faut distinguer ce coût du seul temps
du message de salutation, et ne pas comparer les durées complètes des jobs :
`build` effectue aussi l'installation de Node.js et la publication de l'artefact.
L'API des steps a une précision à la seconde ; une durée affichée de zéro ne
signifie pas que l'action n'a effectué aucun travail.

## Capture réelle à fournir après la réussite

Suivre [les consignes de capture](screenshots/README.md). La capture sera ajoutée
à `screenshots/tp7-actions-success.png` puis publiée avec un commit dédié sur
`feature/tp-actions`. Une capture réelle n'a pas encore été fournie.

## Références

- [Lancer manuellement un workflow](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/manually-run-a-workflow)
- [Transférer des données entre jobs](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/store-and-share-data)
- [Créer une action composite](https://docs.github.com/en/actions/tutorials/create-actions/create-a-composite-action)
- [Créer une action Docker](https://docs.github.com/en/actions/tutorials/create-actions/create-a-docker-container-action)
