# TP7 — Actions tierces et actions personnalisées

**Étudiante :** Achiban Aya — **Établissement :** ENSAM Casablanca  
**Dépôt :** `ashAya758jv/recettes-duo` — **Branche :** `feature/tp-actions`

Les réponses du TP6 sont conservées dans leur branche d'origine :
[REPONSES.md du TP6](https://github.com/ashAya758jv/recettes-duo/blob/feature/tp-env/REPONSES.md).
Ce fichier était absent de `main` au départ du TP7 ; aucune réponse antérieure
n'a été effacée et les fichiers des TP5 et TP6 sont inchangés.

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
- Génération et lecture de `report.txt` vérifiées avec le Node.js local, puis
  installation de Node.js 20 confirmée sur le runner lors de la validation distante.
- Image Docker réellement construite ; `docker run --rm tp7-docker-hello:local Aya`
  affiche exactement `Hello, Aya`.
- Tous les fichiers déjà présents dans `main`, y compris les workflows du TP6
  et les recettes, sont conservés sans modification.

## Validation GitHub Actions réelle

[Exécution réussie du 20 novembre 2026](https://github.com/ashAya758jv/recettes-duo/actions/runs/44807967504),
lancée manuellement sur `feature/tp-actions`, commit `cc47b5c` :

| Job | Résultat vérifié |
| --- | --- |
| [`build`](https://github.com/ashAya758jv/recettes-duo/actions/runs/44807967504/job/136536840036) | Réussi ; sortie `my-app-1.0.0`, Node.js `v20.20.2`, artefact publié |
| [`verify`](https://github.com/ashAya758jv/recettes-duo/actions/runs/44807967504/job/136536930350) | Réussi ; artefact téléchargé et `report.txt` affiché |
| [`docker-hello`](https://github.com/ashAya758jv/recettes-duo/actions/runs/44807967504/job/136536839400) | Réussi ; image construite et message `Hello, Aya` confirmé dans les journaux |

L'artefact [`build-report`](https://github.com/ashAya758jv/recettes-duo/actions/runs/44807967504/artifacts/14664655757)
est présent et non expiré au contrôle. Son archive ZIP de **186 octets** a été
réellement téléchargée ; elle contient uniquement `report.txt` :

```text
Node.js: v20.20.2 | Date (UTC): 2026-11-20T20:04:20Z
```

Ce contenu est identique à la ligne affichée dans les journaux de `verify`.
Le SHA-256 de l'archive téléchargée correspond au digest publié par GitHub :

```text
e61de6d1302f7d23ab39f7ffd30ca23f4935db67274f24edaa59df4d44833c65
```

Le code du workflow et des actions est celui de cette exécution ; le commit
de documentation suivant ajoute les résultats observés. La capture réelle
reste à fournir pour compléter les livrables du TP.

## Lancement manuel sur GitHub

GitHub exige que le workflow soit présent sur la branche par défaut pour activer
`workflow_dispatch`. Au contrôle initial, `main` ne contenait pas `tp-actions.yml`.
Après autorisation explicite de l'étudiante, **seul ce fichier** a été ajouté à
`main` par le [commit `174a3f8`](https://github.com/ashAya758jv/recettes-duo/commit/174a3f83ab4a8ed4eb313711f76f42d6d4f2a20e).
Ce commit a un seul parent et ne constitue pas une fusion. Les fichiers des
TP5 et TP6 ainsi que les anciennes branches de travail sont conservés.

Pour relancer le workflow :

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

## Comparaison des durées mesurées sur GitHub Actions

Mesures issues des champs `started_at` et `completed_at` des steps de
[l'API des jobs de cette exécution](https://api.github.com/repos/ashAya758jv/recettes-duo/actions/runs/44807967504/jobs) :

| Step | Début UTC | Fin UTC | Durée selon l'API |
| --- | --- | --- | --- |
| `Run build info action` | 20:04:17 | 20:04:17 | 0 s à la précision de l'API |
| `Run Docker hello action` | 20:04:17 | 20:04:19 | 2 s |

Les journaux confirment que, pour cette action locale, la construction de l'image
a lieu dans la step Docker : `docker build` démarre à `20:04:17.923` UTC, puis
`docker run` à `20:04:19.726` UTC, soit un intervalle d'environ **1,80 s** incluant
la construction et la transition vers l'exécution. Le message arrive à
`20:04:19.906` UTC.

Sur cette exécution, la step Docker prend davantage de temps que l'action
composite. Son coût comprend la construction de l'image et le démarrage du
conteneur. L'API des steps a une précision à la seconde : **0 s n'est pas une
mesure d'une durée nulle**. Ces deux actions effectuent des tâches différentes ;
ces observations ne constituent pas un benchmark général. Les durées complètes
des jobs ne sont pas comparées, car `build` installe aussi Node.js et publie
l'artefact.

## Capture réelle à fournir après la réussite

Suivre [les consignes de capture](screenshots/README.md). La capture sera ajoutée
à `screenshots/tp7-actions-success.png` puis publiée avec un commit dédié sur
`feature/tp-actions`. Une capture réelle n'a pas encore été fournie.

## Références

- [Lancer manuellement un workflow](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/manually-run-a-workflow)
- [Transférer des données entre jobs](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/store-and-share-data)
- [Créer une action composite](https://docs.github.com/en/actions/tutorials/create-actions/create-a-composite-action)
- [Créer une action Docker](https://docs.github.com/en/actions/tutorials/create-actions/create-a-docker-container-action)
