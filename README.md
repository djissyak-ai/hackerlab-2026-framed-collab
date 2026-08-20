# HackerLab 2026 — Framed

Dépôt privé de collaboration consacré exclusivement à la mission **Framed** de HackerLab 2026, en particulier à Q4 — **Every Breath You Take**.

## Objectif

Reconstituer de manière reproductible l’activité de l’administrateur sur le portail BCT le 30 juillet 2026 et déterminer le compteur `N` demandé par l’énoncé. Aucun flag ne doit être soumis depuis ce dépôt. Un résultat peut être qualifié de candidat uniquement après comparaison des preuves et des méthodes indépendantes.

## À lire en premier

1. [`HackerLab_2026_—_Framed.md`](HackerLab_2026_—_Framed.md) contient le journal technique complet, les énoncés, les limites et l’historique des hypothèses.
2. [`STATUS.md`](STATUS.md) contient l’état courant des vérifications.
3. [`ARTIFACTS.md`](ARTIFACTS.md) indique quels fichiers sont réellement disponibles et comment les vérifier.
4. [`DECISIONS.md`](DECISIONS.md) conserve les désaccords et les raisons de rejet.
5. [`questions/q4-every-breath-you-take.md`](questions/q4-every-breath-you-take.md) contient la fiche de travail dédiée à Q4.

## Situation actuelle

La page HackerLab indiquait Framed à **3/4** : Q1 `Money Trees`, Q2 `The Beginning` et Q3 `Paranoid Android` étaient affichées comme résolues historiquement. Q4 restait ouverte. Le format affiché pour Q4 est :

```text
HLB2026{HH:MM:SS_HH:MM:SS_N}
```

Les timestamps `12:46:23` et `13:13:40` sont des valeurs héritées de l’analyse précédente, à revérifier. Le compteur `N` n’est pas établi. Les valeurs `34`, `35`, `36`, `37` et `38` sont des hypothèses ou des essais historiques, jamais des flags confirmés dans ce dépôt.

## Pourquoi ce dépôt existe

Les agents n’ont pas automatiquement accès au sandbox, aux fichiers locaux ni à la conversation précédente. GitHub sert donc de mémoire commune, d’archive de preuves, de journal de communication et d’espace de revue. Toute conclusion utile doit être écrite ici avec sa source, sa méthode, ses limites et son statut.

Les agents doivent utiliser des branches distinctes, des commits descriptifs, des Issues pour les questions ouvertes et des Pull Requests pour les modifications importantes. Un nombre isolé sans preuve n’est pas un résultat exploitable.

## Règles de sécurité et de soumission

Les artefacts doivent être analysés statiquement. Aucun binaire, script ou contenu actif provenant d’un artefact ne doit être exécuté. Les secrets, cookies, tokens et données personnelles ne doivent pas être ajoutés au dépôt. Le PCAP brut volumineux est conservé hors du dépôt Git normal ; son nom, sa taille, son hash et sa provenance doivent être documentés dans `ARTIFACTS.md`.

Aucun flag ne doit être soumis automatiquement. Le dépôt sert à converger vers une réponse documentée, pas à déclencher une action sur HackerLab.

## Méthode de contribution

Chaque rapport doit préciser la question, l’artefact utilisé, sa taille et son hash, la commande ou la méthode, les observations, l’interprétation, le statut du résultat et les limites. Les statuts autorisés sont **vérifié**, **reproduit**, **candidat**, **rejeté** et **indéterminé**.

Pour commencer : crée une branche `agent/<sujet>`, lis les rapports existants, ajoute une analyse reproductible dans `reports/` ou `evidence/`, mets à jour `STATUS.md` uniquement avec des faits sourcés, puis ouvre une Pull Request. Ne supprime pas les résultats contradictoires : documente-les dans `DECISIONS.md`.
