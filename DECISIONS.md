# DECISIONS — Framed Q4

## Décision 1 — Ne pas confondre rejet de plateforme et preuve de méthode

Les essais historiques de plusieurs candidats ont été rejetés par la plateforme. Cela montre seulement que la chaîne soumise n’était pas acceptée à ce moment-là. Cela ne permet pas d’isoler la variable fautive : heures, format, définition de requête, compteur `N` ou plusieurs éléments à la fois.

## Décision 2 — Ne pas traiter les fichiers dérivés comme le PCAP brut

Les fichiers `framed_client_bursts.txt`, `framed_bct_short_streams_detail.txt`, `framed_bct_response_clusters.txt` et `framed_bct_tls_app_once.tsv` sont utiles pour comprendre l’historique, mais leur méthode de génération doit être vérifiée. Ils ne doivent pas être considérés comme une représentation complète du trafic tant qu’ils n’ont pas été comparés au PCAP brut.

## Décision 3 — Conserver les candidats historiques sans les promouvoir

Les valeurs `34`, `35`, `36`, `37` et `38` restent des candidats ou des résultats historiques. Aucune n’est validée dans ce dépôt. Les conserver est utile pour mesurer les divergences, mais aucune ne doit être présentée comme la réponse probable sans nouveau calcul.

## Décision 4 — Refaire le comptage avec une définition écrite

Avant de compter, chaque agent doit écrire ce que signifie exactement « requête » dans sa méthode : requête HTTP/2, groupe temporel de paquets client, transaction client/serveur, flux TCP, burst métier ou autre unité. Deux agents qui comptent des unités différentes peuvent obtenir des nombres différents sans que l’un ait nécessairement mal calculé.

## Décision 5 — Les timestamps doivent être revérifiés

`12:46:23` et `13:13:40` sont des valeurs héritées. Elles peuvent correspondre à la première et dernière activité retenue selon un filtre antérieur. Elles ne doivent pas être considérées comme définitivement correctes avant une extraction reproductible depuis le PCAP brut.

## Décision 6 — Aucun flag soumis

Le dépôt peut contenir des chaînes au format `HLB2026{...}` à titre de candidats, mais aucune soumission ne doit être déclenchée depuis le dépôt ou par un agent collaborateur.

## Questions ouvertes

1. Quel est l’énoncé exact de l’unité comptée par Q4 ?
2. Les heures demandées correspondent-elles aux premiers et derniers événements métier ou à toute activité applicative ?
3. Les keep-alives, tickets TLS, contrôles HTTP/2, retransmissions et fermetures doivent-ils être exclus ?
4. Le comptage doit-il se faire par requête, par transaction, par stream HTTP/2 ou par groupe temporel ?
5. Le PCAP brut et les fichiers dérivés décrivent-ils exactement la même capture ?
