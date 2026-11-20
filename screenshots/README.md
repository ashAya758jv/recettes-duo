# Capture réelle du TP7

L'[exécution réelle du TP7](https://github.com/ashAya758jv/recettes-duo/actions/runs/44807967504)
a réussi sur `feature/tp-actions` : les trois jobs sont verts et l'artefact
`build-report` a été téléchargé et vérifié. Ce dossier attend encore la capture.

1. Ouvrir le [résumé de l'exécution réussie](https://github.com/ashAya758jv/recettes-duo/actions/runs/44807967504)
   **Third-Party and Custom Actions Lab**, en étant connecté à GitHub.
2. Vérifier le statut **Success**, la branche **feature/tp-actions**, les jobs
   réussis **build**, **verify** et **docker-hello**, puis l'artefact **build-report**.
3. Ajuster le zoom du navigateur pour rendre visibles les jobs et **Artifacts**.
4. Sous Windows, utiliser **Win + Maj + S** et enregistrer l'image réelle.
5. Fournir cette image dans la conversation pour son ajout sous
   `screenshots/tp7-actions-success.png`, puis sa publication avec un commit dédié.

Pour une capture complémentaire du transfert de fichier, ouvrir le job
[`verify`](https://github.com/ashAya758jv/recettes-duo/actions/runs/44807967504/job/136536930350),
déplier **Print build report** et capturer la ligne :

```text
Node.js: v20.20.2 | Date (UTC): 2026-11-20T20:04:20Z
```

Pour récupérer le fichier, revenir au résumé et cliquer sur **build-report**
dans **Artifacts** ; décompresser l'archive puis ouvrir `report.txt`.

Le commit de cette capture et sa publication se feront uniquement sur
`feature/tp-actions`. Aucune capture simulée ne remplace ce livrable.
