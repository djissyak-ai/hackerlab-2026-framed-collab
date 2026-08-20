# Q4 — Every Breath You Take

## Statut

Question encore ouverte historiquement. Aucun flag n’est confirmé dans ce dépôt et aucune soumission n’est autorisée.

## Format hérité

```text
HLB2026{HH:MM:SS_HH:MM:SS_N}
```

Le format doit être vérifié contre la page authentifiée. Les heures et `N` ne doivent pas être supposés corrects simultanément.

## Contexte réseau connu

L’analyse héritée identifie un poste administrateur `192.168.58.129` nommé `DESKTOP-O59DAGR`, communiquant avec le serveur BCT `198.51.100.37:443`. Sept streams TLS ont été retenus :

```text
4093, 4419, 4663, 4735, 6426, 8377, 8533
```

La capture brute associée est `proxy-bct.pcapng`. Elle doit être vérifiée par taille, hash et métadonnées dans l’environnement de l’agent qui y a accès.

## Valeurs historiques

| Élément | Valeur héritée | Statut correct |
|---|---|---|
| Première activité retenue | `12:46:23` | Observation à reproduire |
| Dernière activité retenue | `13:13:40` | Observation à reproduire |
| Nombre de groupes client | `50` | Résultat d’une méthode antérieure |
| Keep-alives identifiés | `12` | Classification à revérifier |
| Contrôles purs identifiés | `2` | Classification à revérifier |
| Bursts métier retenus dans une méthode | `36` | Candidat de méthode, pas réponse |

## Candidats historiques

Les candidats suivants ont été essayés dans l’historique et rejetés ou laissés incertains. Les rejets ne prouvent pas que seule la valeur `N` était incorrecte.

```text
HLB2026{12:46:23_13:13:40_7}
HLB2026{12:46:23_13:13:40_35}
HLB2026{12:46:23_13:13:40_36}
HLB2026{12:46:23_13:13:40_37}
HLB2026{12:46:23_13:13:40_38}
```

Certaines variantes ont utilisé une borne finale `13:12:25`. Elles doivent être documentées comme essais historiques, sans en déduire que la borne correcte est `13:12:25`.

## Méthodes déjà tentées

L’historique contient des regroupements de paquets client avec un seuil temporel d’environ 0,5 seconde, une séparation des paquets de 41 octets considérés comme keep-alives et une exclusion de deux groupes de contrôle purs. Cette méthode a produit notamment `36`, tandis que des traitements voisins ont produit `35`, `37` ou `38`.

Cette divergence est précisément la raison pour laquelle Q4 doit être refaite depuis le PCAP brut. Il faut identifier les messages ou frames HTTP/2, distinguer les contrôles TLS et HTTP/2 des transactions métier, gérer les retransmissions et expliquer les connexions fermées sans réponse.

## Protocole de recalcul

1. Vérifier le PCAP et consigner sa taille, son hash et sa provenance.
2. Extraire les sept streams avec leur direction, timestamp, longueur et numéro de frame.
3. Distinguer les records TLS des frames HTTP/2 observables ; ne pas compter automatiquement chaque record TLS comme une requête.
4. Identifier les événements client et serveur correspondant à une même transaction.
5. Documenter séparément les keep-alives, tickets TLS, SETTINGS, WINDOW_UPDATE, ACK, retransmissions et fermetures.
6. Produire au moins deux comptages : un comptage par transaction et un comptage par autre unité explicitement définie.
7. Comparer les heures obtenues selon chaque définition.
8. Déposer les tableaux bruts dans `evidence/`, le raisonnement dans `reports/` et les désaccords dans `DECISIONS.md`.
9. Ne proposer un candidat qu’après une seconde vérification indépendante.
10. Ne soumettre aucun candidat.

## Fiche de résultat attendue

```markdown
### Analyse <identifiant>

Artefact :
Taille :
SHA-256 :
Outil et version :
Filtres appliqués :
Définition de « requête » :
Première activité observée :
Dernière activité observée :
Événements inclus :
Événements exclus :
N obtenu :
Candidat éventuel :
Niveau de preuve :
Limites :
Résultat reproduit par un second agent : oui/non
Soumission : aucune
```
