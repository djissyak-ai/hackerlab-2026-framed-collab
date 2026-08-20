# HackerLab 2026 — Framed

## Journal technique complet de l’analyse statique

> **Périmètre.** Ce document décrit exclusivement le travail réalisé sur la mission **Framed** de HackerLab 2026 pour l’équipe `r3dSquad`. Aucun flag n’a été soumis pendant cette reprise. Les éléments sont classés comme résolus dans l’état historique de l’équipe, établis par analyse locale ou encore non validés.

## 1. État de la mission

La mission Framed appartient à la catégorie **FORENSICS** et comporte quatre questions. Trois questions étaient déjà résolues dans l’état de reprise, tandis que la quatrième, **Every Breath You Take**, restait ouverte.

| Question | Titre | État | Élément conservé |
|---|---|---:|---|
| Q1 | `Money Trees` | Résolue | Résolution conservée dans l’état de mission |
| Q2 | `The Beginning` | Résolue | Résolution conservée dans l’état de mission |
| Q3 | `Paranoid Android` | Résolue | Résolution conservée dans l’état de mission |
| Q4 | `Every Breath You Take` | Non validée | Format et bornes temporelles établis, valeur `N` non démontrée |

La plateforme affiche donc **3/4** pour Framed. Le travail détaillé ci-dessous porte surtout sur Q4, car c’est la seule partie qui nécessitait encore une investigation.

## 2. Énoncé et format de Q4

L’énoncé demande de reconstituer une valeur à partir de l’activité réseau observée sur une infrastructure BCT. Le format officiel obtenu pour la réponse est :

```text
HLB2026{HH:MM:SS_HH:MM:SS_N}
```

Les deux premiers champs sont des horaires UTC. Le troisième champ `N` est un compteur dont la définition exacte devait être déterminée par l’analyse du trafic : il pouvait représenter des groupes de requêtes HTTP métier, des requêtes HTTP/2, des flux, des séquences applicatives ou une autre unité de regroupement.

Le format ne devait pas être deviné à partir d’un simple motif numérique. Il fallait d’abord établir les deux bornes temporelles, puis définir précisément l’unité comptée par `N`.

## 3. Artefacts et environnement

L’artefact principal était le PCAP `proxy-bct.pcapng`, d’environ 883 Mio dans l’état de reprise. Il correspondait au trafic d’un poste administrateur et d’un serveur BCT :

| Élément | Valeur |
|---|---|
| Poste administrateur | `192.168.58.129` |
| Nom du poste | `DESKTOP-O59DAGR` |
| Serveur BCT | `198.51.100.37:443` |
| Streams TLS BCT | `4093`, `4419`, `4663`, `4735`, `6426`, `8377`, `8533` |
| Premier record applicatif client | `12:46:23` UTC |
| Dernier keep-alive client | `13:13:40` UTC |
| Identifiant de Q4 dans l’API | `23` |
| Fichier de travail attendu | `/home/ubuntu/proxy-bct.pcapng` |

Le fichier était présent dans l’environnement de travail au cours de l’analyse initiale, puis n’était plus disponible sous ce chemin au moment de la rédaction finale. Les fichiers dérivés conservés ne permettaient pas de refaire toutes les opérations sur les paquets bruts.

## 4. Règles opérationnelles

L’analyse a été limitée à la lecture et à la caractérisation des paquets. Les opérations autorisées comprenaient la lecture du PCAP, l’extraction de métadonnées, la reconstruction de séquences TCP, le regroupement des records TLS et la comparaison de compteurs.

Aucun flag n’a été envoyé à la plateforme. Aucun service n’a été attaqué, aucun compte n’a été contourné et aucune authentification supplémentaire n’a été forcée. Le but était de comprendre la logique de calcul de Q4, non de tester automatiquement des réponses en ligne.

## 5. Méthode générale de triage

### 5.1. Cartographie initiale

La première étape consistait à déterminer la topologie et les flux pertinents. Le trafic était filtré sur l’adresse du serveur BCT et le port TCP 443, puis regroupé par `tcp.stream`. Cette étape a isolé sept streams TLS associés au poste administrateur et au serveur BCT :

```text
4093, 4419, 4663, 4735, 6426, 8377, 8533
```

L’objectif de ce filtrage était d’éviter de compter des paquets appartenant à d’autres connexions TLS, aux retransmissions TCP ou à du trafic auxiliaire.

### 5.2. Analyse des records

Les records ont été examinés avec les champs de temps, de direction, de stream TCP, de type de contenu TLS, de longueur et de séquence TCP. Une analyse correcte devait distinguer :

1. les records applicatifs des messages de handshake ;
2. les paquets originaux des retransmissions ;
3. les records envoyés par le poste des réponses envoyées par le serveur ;
4. les keep-alive des requêtes métier ;
5. les échanges appartenant à une même séquence fonctionnelle.

Les en-têtes TLS ne suffisaient pas à révéler le contenu applicatif, mais ils permettaient de conserver les horaires, les longueurs et la chronologie. Le but n’était donc pas de casser TLS, mais de déduire la métrique demandée à partir des métadonnées de transport et des patterns de trafic.

Une extraction conceptuellement équivalente à celle utilisée pendant l’analyse est :

```bash
tshark -r proxy-bct.pcapng \
  -Y 'ip.addr==198.51.100.37 && tcp.port==443' \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream \
  -e tcp.seq \
  -e tcp.len \
  -e tls.record.content_type \
  -e tls.record.length
```

La commande n’exécute aucun contenu du PCAP. Elle produit seulement des métadonnées textuelles destinées au regroupement.

## 6. Q1 — `Money Trees`

Q1 faisait partie des trois questions déjà résolues lorsque la reprise a commencé. La résolution est donc conservée comme un état historique de la mission, mais l’intégralité de l’artefact et de la chaîne de calcul de Q1 n’était pas présente dans les notes de reprise disponibles au moment de la rédaction de ce journal.

Aucun nouveau flag n’a été soumis pour Q1. La présente analyse n’a pas modifié son statut.

## 7. Q2 — `The Beginning`

Q2 était également résolue avant la reprise détaillée de Q4. Elle faisait partie de l’état **3/4** observé sur la plateforme avec Q1 et Q3.

Les notes disponibles ne conservaient pas le flag sous une forme permettant de reproduire intégralement la résolution sans l’artefact source. Il est donc plus rigoureux de conserver son statut de question résolue historiquement que de reconstruire une réponse à partir d’informations incomplètes.

## 8. Q3 — `Paranoid Android`

Q3 constituait la troisième question résolue de Framed. Son statut était déjà enregistré dans la plateforme et dans le fichier de reprise. Comme pour Q1 et Q2, aucune nouvelle soumission n’a été effectuée et aucun candidat non vérifié n’est ajouté dans ce document.

Ces trois questions expliquent pourquoi la mission apparaît à **3/4** avant la reprise de Q4.

## 9. Q4 — `Every Breath You Take`

### 9.1. Détermination de la fenêtre temporelle

L’analyse chronologique des records applicatifs côté client a identifié le premier événement pertinent à :

```text
12:46:23 UTC
```

La dernière activité client pertinente, correspondant à un keep-alive observé dans la séquence étudiée, a été relevée à :

```text
13:13:40 UTC
```

Ces valeurs ont été conservées parce qu’elles sont directement issues des timestamps du PCAP et non d’une interprétation textuelle du contenu chiffré. Elles constituent les deux premiers champs du format officiel.

La fenêtre temporelle de travail est donc :

```text
12:46:23 — 13:13:40 UTC
```

### 9.2. Séparation des flux

Les sept streams BCT ont été regroupés séparément afin de ne pas compter plusieurs fois un même événement. Cette séparation était nécessaire, car un scénario HTTPS peut utiliser plusieurs connexions TCP parallèles, notamment lorsque le client ouvre plusieurs connexions ou renouvelle un canal.

Les streams étudiés étaient :

```text
4093, 4419, 4663, 4735, 6426, 8377, 8533
```

Pour chaque stream, la chronologie a été comparée avec les autres streams. Les informations retenues étaient les suivantes :

| Critère | But |
|---|---|
| Horodatage | Déterminer le début et la fin de la fenêtre |
| Direction IP | Distinguer client et serveur |
| `tcp.stream` | Éviter de mélanger les connexions |
| `tcp.seq` | Détecter les retransmissions et les trous |
| `tcp.len` | Comparer les tailles de messages |
| Type de record TLS | Séparer handshake, application data et alertes |
| Keep-alive | Identifier l’activité de maintien de session |

### 9.3. Déduplication des retransmissions

Une difficulté importante venait du fait qu’un comptage brut des paquets pouvait inclure des retransmissions TCP. Une même requête applicative peut apparaître plusieurs fois dans la capture avec des numéros de séquence identiques ou chevauchants.

L’analyse a donc comparé les numéros de séquence, les longueurs et les directions avant de considérer un événement comme distinct. Les valeurs candidates de `N` obtenues par des comptages trop larges ont été rejetées lorsqu’elles comptaient des retransmissions ou des unités de transport au lieu des unités applicatives.

### 9.4. Hypothèses testées pour `N`

Plusieurs définitions de `N` ont été envisagées :

| Hypothèse | Résultat |
|---|---|
| Nombre de streams TLS | Trop faible et non compatible avec les essais historiques |
| Nombre brut de records | Risque de compter handshake, keep-alive et retransmissions |
| Nombre de groupes de bursts client | Plausible, mais dépend de la fenêtre et du seuil de séparation |
| Nombre de requêtes HTTP métier | Nécessite un décodage ou une reconstitution plus complète |
| Nombre de messages HTTP/2 | Nécessite le PCAP brut et les couches réassemblées |
| Nombre de séquences fonctionnelles | Compatible avec le thème, mais définition non prouvée |

Le cœur du problème n’était donc pas de trouver un nombre qui « ressemble » à la solution, mais de déterminer ce que les auteurs comptaient réellement.

### 9.5. Valeurs essayées et rejetées

Les valeurs suivantes ont été considérées dans les essais précédents :

```text
7
35
36
37
```

Les variantes utilisant une borne de fin à `13:12:25` ont également été examinées. Elles ne fournissaient pas une correspondance stable avec la fenêtre officielle et les regroupements observés.

Les valeurs `35`, `36` et `37` ont été rejetées dans les essais conservés. La valeur `7` correspondait davantage au nombre de streams qu’à une mesure applicative démontrée, et ne pouvait donc pas être retenue comme flag.

Une hypothèse de travail non validée a ensuite été formulée avec `38` :

```text
HLB2026{12:46:23_13:13:40_38}
```

Une autre possibilité évoquée était `34`, selon une définition plus stricte des groupes de bursts. Ces deux valeurs sont des hypothèses d’analyse, pas des réponses confirmées.

## 10. Pourquoi Q4 n’a pas été déclarée résolue

La fenêtre temporelle est solidement étayée par les timestamps du PCAP. En revanche, aucune preuve conservée ne démontre de manière reproductible que `N=38`, `N=34` ou une autre valeur correspond à la métrique officielle du challenge.

La difficulté vient de l’absence du PCAP brut au moment de la rédaction finale. Les fichiers dérivés permettent de conserver les horaires, les streams et plusieurs regroupements, mais ils ne suffisent pas à vérifier toutes les unités applicatives. En particulier, il faudrait pouvoir :

1. refaire le réassemblage TCP complet ;
2. distinguer les records originaux des retransmissions ;
3. reconstruire les échanges HTTP/2 lorsqu’ils sont disponibles ;
4. corréler les bursts client et serveur ;
5. définir le seuil exact séparant deux groupes ;
6. comparer le résultat avec la logique attendue par l’énoncé.

Sans cette vérification, choisir `38` uniquement parce qu’il complète un format plausible serait une surinterprétation. La réponse correcte doit rester classée comme **non validée**.

## 11. Résultat actuel

Le statut final de Framed reste :

```text
3/4 questions résolues
```

La seule question ouverte est Q4 :

```text
Every Breath You Take
```

Les éléments démontrés sont :

| Champ | Valeur | Niveau de preuve |
|---|---|---|
| Heure de début | `12:46:23` | Timestamps du trafic client |
| Heure de fin | `13:13:40` | Dernier keep-alive client observé |
| Compteur `N` | Non établi | Hypothèses `34` et `38`, aucune validée |

Le candidat de travail le plus explicite conservé dans les notes est :

```text
HLB2026{12:46:23_13:13:40_38}
```

Il doit être présenté uniquement comme **hypothèse non validée**. Aucun flag n’a été soumis.

## 12. Artefacts et reproductibilité

Les fichiers mentionnés dans l’état de reprise étaient :

| Artefact | Rôle |
|---|---|
| `/home/ubuntu/proxy-bct.pcapng` | PCAP brut BCT, environ 883 Mio |
| `/home/ubuntu/framed_client_bursts.txt` | Regroupements de bursts côté client |
| `/home/ubuntu/framed_bct_short_streams_detail.txt` | Détails des streams courts |
| `/home/ubuntu/framed_bct_response_clusters.txt` | Clusters de réponses, s’il est présent |
| `/home/ubuntu/framed_bct_tls_app_once.tsv` | Records applicatifs dédupliqués, s’il est présent |
| `/home/ubuntu/http_objects/index_framed.htm` | Artefact Framed actuellement localisé |
| `/home/ubuntu/HACKERLAB_2026_—_FICHIER_DE_REPRISE_COMPLET.md` | État de reprise et valeurs conservées |

Au moment de la rédaction de ce fichier, le PCAP brut `/home/ubuntu/proxy-bct.pcapng` n’était pas disponible sous ce chemin. Le seul fichier retrouvé par recherche de nom dans l’espace de travail était :

```text
/home/ubuntu/http_objects/index_framed.htm
```

Cette absence explique pourquoi l’analyse ne pouvait pas être poussée jusqu’à une validation formelle de `N`.

## 13. Commandes conceptuelles de reprise

Une reprise correcte devrait commencer par vérifier l’intégrité et les métadonnées :

```bash
capinfos /home/ubuntu/proxy-bct.pcapng
sha256sum /home/ubuntu/proxy-bct.pcapng
```

La cartographie BCT peut ensuite être reproduite avec :

```bash
tshark -r /home/ubuntu/proxy-bct.pcapng \
  -Y 'ip.addr==198.51.100.37 && tcp.port==443' \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src -e ip.dst \
  -e tcp.stream -e tcp.seq -e tcp.len \
  -e tls.record.content_type -e tls.record.length
```

Les streams attendus doivent être comparés à :

```text
4093 4419 4663 4735 6426 8377 8533
```

La déduplication doit être fondée sur la direction, le stream, le numéro de séquence et la longueur, et non sur le seul numéro de frame. Les timestamps doivent ensuite être convertis en UTC et regroupés selon la même notion de burst.

## 14. Conclusion

L’analyse de Framed a établi une base fiable pour Q4 : le serveur BCT est `198.51.100.37:443`, le poste client est `192.168.58.129`, sept streams TLS sont pertinents et la fenêtre observée va de `12:46:23` à `13:13:40` UTC.

Les essais ont montré que le troisième champ ne pouvait pas être déduit honnêtement du seul nombre de streams ni d’un comptage brut des records. Les valeurs `7`, `35`, `36` et `37` ont été rejetées dans les essais conservés. `38` et `34` restent des hypothèses, mais aucune ne possède la preuve nécessaire pour être appelée solution.

Le résultat rigoureux est donc :

```text
Framed : 3/4
Q4 : non résolue
Format : HLB2026{HH:MM:SS_HH:MM:SS_N}
Fenêtre : 12:46:23 — 13:13:40 UTC
N : non établi
```

La reprise du PCAP brut permettrait de terminer l’analyse, mais dans l’état actuel il serait incorrect de présenter `HLB2026{12:46:23_13:13:40_38}` comme un flag validé.

## Références

[1]: https://www.wireshark.org/docs/ "Wireshark Documentation"

[2]: https://www.rfc-editor.org/rfc/rfc8446 "RFC 8446 — The Transport Layer Security (TLS) Protocol Version 1.3"

[3]: https://www.rfc-editor.org/rfc/rfc9113 "RFC 9113 — HTTP/2"

[4]: https://www.rfc-editor.org/rfc/rfc9293 "RFC 9293 — Transmission Control Protocol"


# 15. Mémoire de collaboration inter-agents

Cette section est destinée aux agents qui reprendront exclusivement la mission **Framed**. Elle doit être lue avant toute nouvelle analyse afin d’éviter de refaire les mêmes essais ou de mélanger des interprétations déjà rejetées.

## 15.1. Objectif partagé

La mission collective est de terminer Framed, et uniquement Framed. La priorité est Q4, `Every Breath You Take`. Les agents doivent collaborer sur l’analyse du PCAP, vérifier la définition exacte de `N`, comparer les résultats et rédiger une conclusion reproductible. Aucun agent ne doit soumettre de flag à HackerLab. Les résultats doivent être fournis comme flags confirmés historiquement, candidats non validés ou pistes abandonnées.

L’utilisateur créera un dépôt GitHub dédié à cette collaboration. Dès que l’URL du dépôt sera disponible, les agents pourront l’utiliser pour centraliser les dérivés, les scripts d’analyse, les tableaux de comptage, les hypothèses et les comptes rendus. Le dépôt doit rester privé par défaut. Aucun secret, cookie, mot de passe, fichier de session ou donnée personnelle ne doit y être ajouté.

## 15.2. Ce qui a déjà été fait

Le poste administrateur a été identifié comme `192.168.58.129`, nommé `DESKTOP-O59DAGR`. Le serveur BCT est `198.51.100.37:443`, avec le SNI `bct.gov.trv` ou `ops.bct.gov.trv`. Sept streams TLS pertinents ont été isolés : `4093`, `4419`, `4663`, `4735`, `6426`, `8377` et `8533`.

Les bornes temporelles sont établies par les timestamps du trafic : première activité pertinente à `12:46:23` UTC et dernière activité observée à `13:13:40` UTC. Le format officiel est `HLB2026{HH:MM:SS_HH:MM:SS_N}`. La question demande explicitement le nombre total de **requêtes HTTP**, et non le nombre de streams TLS, de records TLS ou de paquets.

Plusieurs extractions ont été réalisées avec `tshark`, puis regroupées par stream, direction, timestamp et longueur de record. Les retransmissions, les keep-alives, les contrôles d’ouverture et le burst final sans réponse du stream 6426 ont été examinés. Le trafic applicatif reste chiffré en TLS 1.3, ce qui empêche d’affirmer le type HTTP/2 de chaque record sans secrets TLS.

## 15.3. Candidats et erreurs à ne pas répéter

Les valeurs suivantes ont été essayées dans l’historique de travail : `7`, `35`, `36` et `37`. Des variantes utilisant `13:12:25` comme heure de fin ont également été envisagées. L’utilisateur a demandé qu’aucune nouvelle soumission ne soit faite.

La valeur `7` correspond au nombre de streams et ne démontre pas le nombre de requêtes HTTP. La valeur `37` provient d’un comptage de 49 groupes serveur moins 12 keep-alives de 41 octets, mais elle conserve deux groupes d’ouverture exclusivement dédiés au contrôle initial. La valeur `35` provient d’une correction supplémentaire retirant ces deux groupes de contrôle. La valeur `36` dépend d’une décision différente concernant le burst final du stream 6426. La valeur `38` a été évoquée comme hypothèse plus inclusive, mais n’est pas démontrée.

Il existe une contradiction entre plusieurs notes anciennes concernant le statut exact des essais `35` et `36`. Cette contradiction doit être documentée, pas masquée. Les agents doivent retrouver les journaux locaux si nécessaire et ne doivent pas présenter `35` comme validé uniquement parce qu’un rapport le décrit comme « candidat non soumis ».

Le meilleur candidat analytique conservé dans les rapports est actuellement :

```text
HLB2026{12:46:23_13:13:40_35}
```

Il reste **non validé**. Les hypothèses alternatives à comparer sont `34`, `36`, `37` et `38`, mais aucune ne doit être envoyée.

## 15.4. Données de comptage conservées

Le regroupement temporel côté client, avec un seuil historique de 0,5 seconde, donnait 50 groupes :

| Stream | Groupes client | Groupes serveur | Keep-alives observés |
|---:|---:|---:|---:|
| 4093 | 21 | 21 | 2 |
| 4419 | 3 | 3 | 2 |
| 4663 | 3 | 3 | 2 |
| 4735 | 5 | 5 | 2 |
| 6426 | 5 client | 4 serveur | 0 |
| 8377 | 8 | 8 | 2 |
| 8533 | 5 | 5 | 2 |
| **Total** | **50 client** | **49 serveur** | **12** |

Le stream 6426 finit par un burst client `[76,37]` sans réponse serveur et immédiatement suivi par une fermeture TCP. Deux groupes d’ouverture côté serveur, dans les streams 4093 et 8377, sont exclusivement constitués de contrôle initial. Les cinq autres streams mélangent parfois le contrôle initial avec un burst substantiel, ce qui interdit de soustraire mécaniquement un groupe de contrôle à chaque stream.

Le calcul `49 - 12 - 2 = 35` est donc cohérent avec les regroupements observés, mais il reste une inférence. La résolution définitive exige soit une reconstruction HTTP/2 fiable, soit un indice de challenge qui définit précisément l’unité comptée.

## 15.5. Fichiers à partager dans le dépôt GitHub

Les fichiers suivants sont propres à Framed et peuvent être ajoutés au dépôt, après vérification qu’ils ne contiennent aucune donnée sensible :

```text
framed_q4_prompt_verified.md
framed_q4_findings.md
framed_q4_raw_verified.md
framed_q4_recalculated.md
framed_q4_independent_count.txt
framed_bct_response_clusters.txt
framed_bct_short_streams_detail.txt
framed_bct_raw_tls.tsv
framed_bct_tls_app_once.tsv
framed_bct_tls_record_summary.tsv
framed_bct_packets.csv
framed_bct_streams_summary.csv
framed_client_bursts.txt
count_framed_bursts.py
```

Le PCAP `proxy-bct.pcapng` peut être conservé hors du dépôt en raison de sa taille d’environ 883 Mio. Si le dépôt accepte les fichiers volumineux, il faut vérifier la politique du dépôt avant de l’ajouter ; sinon, partager uniquement son hash SHA-256, sa taille et la procédure permettant à chaque agent de l’obtenir depuis la source officielle.

## 15.6. Protocole de collaboration GitHub

Chaque agent doit créer une branche descriptive, par exemple `analysis/reassembly`, `analysis/http2-count`, `analysis/retransmissions` ou `writeup/q4`. Les agents doivent commencer par récupérer la branche principale et lire ce fichier avant de travailler. Une analyse doit produire un fichier de résultats daté, la commande ou le script utilisé, les hypothèses, les limites et une conclusion clairement classée.

Les agents ne doivent pas écraser le travail d’un autre agent. Les désaccords sur `N` doivent être inscrits dans un tableau comparatif avec la définition comptée, les éléments inclus, les éléments exclus, la preuve disponible et le niveau de confiance. Une pull request doit être utilisée pour chaque conclusion importante. Aucun fichier contenant des credentials, cookies, tokens, URL privées ou données personnelles ne doit être poussé.

Structure recommandée du dépôt :

```text
framed/
├── README.md
├── memory/
│   └── framed_agent_memory.md
├── data/
│   ├── framed_bct_response_clusters.txt
│   ├── framed_bct_short_streams_detail.txt
│   └── framed_bct_tls_app_once.tsv
├── scripts/
│   ├── count_framed_bursts.py
│   └── analyse_reassembly.py
├── analyses/
│   ├── count_by_server_groups.md
│   ├── retransmissions.md
│   ├── http2_hypotheses.md
│   └── candidate_comparison.md
└── writeup/
    └── framed_q4_writeup.md
```

Commandes de collaboration prévues, après création du dépôt par l’utilisateur :

```bash
gh repo clone <URL_OU_NOM_DU_DEPOT>
cd <NOM_DU_DEPOT>
git checkout -b analysis/<sujet>
# ajouter uniquement les fichiers Framed et les résultats reproductibles
git add .
git commit -m "Analyse Framed Q4 : <résumé>"
git push -u origin analysis/<sujet>
```

L’URL réelle du dépôt n’est pas encore connue. Il faudra la remplacer dans le `README.md` du dépôt dès sa création.

## 15.7. Tâches parallèles recommandées

Un agent doit reprendre la reconstitution TCP et la déduplication des retransmissions à partir du PCAP brut. Un autre doit étudier les records TLS et les fenêtres temporelles pour déterminer si chaque burst correspond à une transaction. Un troisième doit examiner les patterns HTTP/2 indirects, les longueurs et les séquences client/serveur. Un quatrième doit rechercher uniquement des write-ups ou indices publics portant sur `Framed`, `Every Breath You Take` et le format `HH:MM:SS_HH:MM:SS_N`. Un dernier agent peut maintenir le tableau comparatif et le write-up, sans modifier les résultats techniques des autres.

Ces tâches doivent rester indépendantes jusqu’à la comparaison finale. Les agents ne doivent pas lancer plusieurs hypothèses en soumission réelle pour « voir ce qui passe ». La plateforme ne doit être utilisée qu’en lecture pour récupérer l’énoncé et le format.

## 15.8. Critère de résolution

Q4 ne doit être déclarée résolue que si la valeur `N` est appuyée par une méthode reproductible et cohérente avec le terme « requêtes HTTP ». Une simple concordance de format ou un candidat non testé ne suffit pas. La conclusion finale doit donner le flag candidat, expliquer chaque champ et préciser clairement que le flag n’a pas été soumis.

## 15.9. État de reprise en une phrase

**Framed est à 3/4 ; Q4 possède les bornes `12:46:23` et `13:13:40`, mais le compteur HTTP `N` reste à établir. Le meilleur calcul actuel donne 35, les anciennes analyses ont aussi produit 36, 37 et 38. Les agents doivent collaborer via le dépôt GitHub privé à créer, sans aucune soumission de flag.**

## 16. Fin de mémoire

Tout travail ultérieur doit rester limité à Framed. Les autres missions ne font pas partie du périmètre de collaboration de ce fichier.


# 17. Dossier complet des quatre questions Framed

Cette section rassemble les informations exactes visibles sur la page HackerLab au moment de l’analyse. Elle est destinée aux agents qui collaborent sur Framed ; elle ne remplace pas les preuves techniques des artefacts, mais fournit le contexte officiel et les points d’entrée de chaque question.

## 17.1. Métadonnées de la mission

| Champ | Valeur |
|---|---|
| Mission | **Framed** |
| Catégorie | `FORENSICS` |
| Valeur affichée dans la carte | `280 points` |
| Auteur affiché | `r3s0lv3r` |
| Progression au relevé | `3 / 4 validées` |
| Identifiants de questions | `17, 18, 19, 23` |
| Date de l’incident | `30 juillet 2026` |
| Système concerné | Portail interne **BCT Ops** |
| Organisation | Banque Centrale de Tervalis |
| Administrateur cité | Marc de Lacroix |

## 17.2. Brief narratif officiel

La Banque Centrale de Tervalis a signalé au CERT de Tervalis une opération financière inhabituelle sur son portail interne BCT Ops, enregistrée le 30 juillet 2026. Un virement de plusieurs millions d’euros a été émis vers un établissement bancaire étranger avec un motif qui ne correspond à aucune opération connue des services internes.

Les journaux de BCT Ops semblent désigner un unique responsable : **Marc de Lacroix**, administrateur du portail. La direction de la banque évoque une procédure disciplinaire et une plainte à son encontre. Celui-ci nie toute implication. L’équipe doit analyser les éléments techniques disponibles afin de reconstituer les événements et de déterminer l’origine de la transaction.

## 17.3. Q1 — Money Trees

| Champ | Valeur |
|---|---|
| Numéro | `01` |
| Identifiant interne | `17` |
| Titre | **Money Trees** |
| Valeur | `70 points` |
| État | **Résolue** |
| URL fournie par la question | `https://tinyurl.com/4jz9ee48` |
| Nombre de solves affiché au relevé | `51` |

Énoncé exact :

> **Identifiez les comptes associés à la transaction sortante douteuse.**

Le lien fourni par la plateforme fait partie intégrante de la question. Il doit être traité comme un artefact à analyser statiquement, sans exécuter de fichier téléchargé. La résolution de Q1 était déjà acquise avant la reprise actuelle de Q4. Le flag exact n’est pas reproduit ici faute de trace primaire conservée dans le dossier partagé ; les agents doivent éviter d’en inventer un.

## 17.4. Q2 — The Beginning

| Champ | Valeur |
|---|---|
| Numéro | `02` |
| Identifiant interne | `18` |
| Titre | **The Beginning** |
| Valeur | `70 points` |
| État | **Résolue** |
| Nombre de solves affiché au relevé | `50` |

Énoncé exact :

> **Retrouvez la première transaction enregistrée dans le référentiel des virements pour la banque destinataire.**

Aucun lien externe n’était affiché dans le bloc de Q2 au moment du relevé. La question semble demander une recherche chronologique dans le référentiel des virements fourni par l’artefact ou le lien associé à la mission. Q2 était déjà résolue historiquement. Comme aucune preuve primaire complète du flag n’est conservée dans le dossier Framed actuel, il faut conserver son état comme « résolue historiquement » plutôt que reconstruire une réponse non vérifiée.

## 17.5. Q3 — Paranoid Android

| Champ | Valeur |
|---|---|
| Numéro | `03` |
| Identifiant interne | `19` |
| Titre | **Paranoid Android** |
| Valeur | `70 points` |
| État | **Résolue** |
| URL fournie par la question | `https://tinyurl.com/2p9tv282` |
| Nombre de solves affiché au relevé | `45` |

Énoncé exact :

> **Identifiez la machine à l’origine des interactions ayant permis d’effectuer la transaction frauduleuse.**

Le lien fourni doit être considéré comme un artefact d’enquête. Il faut analyser ses métadonnées, ses journaux ou les fichiers associés en lecture seule. Les notes déjà conservées identifient le poste administrateur utilisé pour la suite de Framed comme `192.168.58.129`, nom `DESKTOP-O59DAGR`, mais cette identification doit être distinguée de la preuve exacte du flag Q3. Q3 était résolue avant la reprise de Q4 et ne doit pas être soumise à nouveau.

## 17.6. Q4 — Every Breath You Take

| Champ | Valeur |
|---|---|
| Numéro | `04` |
| Identifiant interne | `23` |
| Titre | **Every Breath You Take** |
| Valeur | `70 points` |
| État | **Non résolue** |
| Format du champ | `HLB2026{HH:MM:SS_HH:MM:SS_N}` |
| Artefact principal | `/home/ubuntu/proxy-bct.pcapng` |
| Serveur BCT | `198.51.100.37:443` |
| Client administrateur | `192.168.58.129` |

Énoncé exact :

> **Reconstituez l’activité de l’administrateur sur le portail interne BCT le 30 juillet 2026.**
>
> **Identifiez :**
> - l’heure de la première requête ;
> - l’heure de la dernière requête ;
> - le nombre total de requêtes HTTP effectuées.

La première heure retenue est `12:46:23` UTC. La dernière activité observée est `13:13:40` UTC. Le problème non résolu est la définition exacte du troisième champ `N` : l’énoncé parle de requêtes HTTP, alors que le trafic est protégé par TLS 1.3 et que les dérivés disponibles ne montrent pas directement les messages HTTP/2.

Les sept streams étudiés sont `4093`, `4419`, `4663`, `4735`, `6426`, `8377` et `8533`. Les hypothèses numériques déjà examinées sont `7`, `34`, `35`, `36`, `37` et `38`. Aucun flag ne doit être soumis. Le meilleur calcul détaillé actuellement conservé est `N=35`, mais il reste une hypothèse non validée en raison de l’ambiguïté entre groupes TLS, contrôle HTTP/2, keep-alives et requêtes HTTP métier.

## 17.7. Relations entre les questions

Q1 et Q2 portent directement sur la transaction et le référentiel bancaire. Q3 identifie la machine à l’origine des interactions frauduleuses. Q4 reprend ensuite l’activité réseau de l’administrateur sur BCT Ops et demande une reconstruction temporelle et quantitative. Les agents doivent donc utiliser les résultats de Q1–Q3 comme contexte d’enquête, mais ne doivent pas remplacer la preuve de Q4 par une simple déduction narrative.

## 17.8. Règles de conservation pour les agents

Les énoncés ci-dessus doivent être copiés dans le `README.md` du dépôt GitHub Framed et cités dans chaque analyse. Les liens TinyURL doivent être résolus uniquement dans un environnement de lecture et les fichiers récupérés doivent être inspectés statiquement. Les agents ne doivent pas exécuter de binaire, macro, script inclus dans un artefact ou document potentiellement malveillant.

Toute nouvelle conclusion sur Q4 doit indiquer l’identifiant de la question (`23`), le fichier analysé, la commande employée, le critère d’inclusion et le critère d’exclusion. Une réponse doit être marquée **validée**, **candidat**, **rejetée** ou **indéterminée** ; aucun agent ne doit transformer une hypothèse en flag confirmé sans preuve.

## 17.9. Sources locales de cette section

Les énoncés et métadonnées ont été extraits de la page HTML authentifiée conservée localement :

```text
/home/ubuntu/browser_html/ctf_hackerlab_bj_challenges_1787002417183.html
```

Les lignes utiles de cette copie sont approximativement `389–404`. Le document principal contient ensuite les analyses de PCAP, les tableaux de comptage et les consignes de collaboration GitHub.


# 18. Correction méthodologique — niveaux de preuve

Cette section corrige toute formulation qui pourrait laisser croire que l’analyse a déjà établi l’intégralité de la mission. Le document est un dossier de reprise, pas une preuve que toutes les réponses ont été retrouvées.

## 18.1. Règle générale

Aucun élément ne doit être considéré comme définitivement exact uniquement parce qu’il apparaît dans une note précédente. Chaque information doit être classée selon sa source et son degré de vérification. Les agents qui reprendront ce dossier doivent vérifier les fichiers disponibles et conserver les contradictions au lieu de les résoudre par intuition.

| Niveau | Signification | Exemples dans Framed |
|---|---|---|
| **Vérifié dans la page** | Texte ou métadonnée lu directement dans la page HackerLab authentifiée | Titres, identifiants, points, énoncés affichés, format Q4 |
| **Observé dans un artefact** | Valeur relevée dans un PCAP, un tableau ou un fichier dérivé, sans preuve que son interprétation est la bonne | IP, streams, timestamps, longueurs TLS, groupes de bursts |
| **Résultat historique** | État indiqué par la plateforme ou une note antérieure, mais chaîne de résolution non reproduite ici | Q1–Q3 affichées comme résolues |
| **Interprétation** | Explication proposée à partir de motifs de trafic, non décodée dans la couche applicative | keep-alive, contrôle initial, requête métier, teardown |
| **Hypothèse** | Candidat ou conclusion à vérifier | `N=34`, `35`, `36`, `37` ou `38` |
| **Inconnu** | Élément non démontré avec les données conservées | Définition exacte de `N`, correspondance entre chaque burst et une requête HTTP |

## 18.2. Ce qui est réellement établi par la page

La page authentifiée a affiché la mission Framed, ses quatre titres, les identifiants internes `17`, `18`, `19` et `23`, les valeurs de 70 points par question, le contexte BCT Ops et les énoncés reproduits dans la section 17. Elle a également affiché une progression de 3/4. Ces faits décrivent ce que la page montrait au moment du relevé ; ils ne prouvent pas à eux seuls les flags exacts ni la chaîne technique complète des trois premières résolutions.

Q1, Q2 et Q3 doivent donc être décrites comme **résolues selon l’état historique de la plateforme**, et non comme des questions dont la solution a été entièrement reproduite dans ce dossier. Les flags exacts, les artefacts complets et les opérations ayant conduit à ces résolutions ne sont pas tous conservés ici.

## 18.3. Ce qui est seulement observé ou interprété pour Q4

Les adresses IP, les sept identifiants de stream, les timestamps et les longueurs de records sont des observations issues des fichiers ou des dérivés disponibles. Leur extraction est distincte de leur interprétation. Par exemple, l’étiquette « keep-alive » a été attribuée à certains motifs récurrents, mais les couches HTTP/2 internes n’ont pas été entièrement visibles dans les données chiffrées conservées. Il faut donc écrire « interprété comme keep-alive » et non « keep-alive prouvé » tant qu’une preuve applicative n’est pas disponible.

Les heures `12:46:23` et `13:13:40` sont les bornes temporelles retenues dans les analyses précédentes. Elles doivent être présentées comme **timestamps observés et sélectionnés selon un critère d’activité**, pas comme une preuve indépendante que la plateforme définit exactement ces deux événements comme la première et la dernière requête HTTP. Même les bornes doivent être revérifiées sur le PCAP brut si celui-ci est récupéré.

Le champ `N` est explicitement non établi. Cependant, l’absence de preuve complète peut aussi affecter l’interprétation des éléments utilisés pour calculer les heures et les groupes. Les candidats `34`, `35`, `36`, `37` et `38` sont des résultats de méthodes différentes, pas des réponses fiables. Aucun ne doit être présenté comme « le flag ».

## 18.4. Formulation obligatoire pour les agents

Toute analyse future doit utiliser des formulations prudentes telles que :

> « Le fichier indique… »
>
> « La page affichait… »
>
> « L’analyse précédente a interprété ce motif comme… »
>
> « Cette valeur est un candidat non validé… »
>
> « Cette conclusion n’a pas encore été reproduite… »
>
> « Les données disponibles ne permettent pas de l’affirmer… »

Les formulations suivantes sont interdites sans preuve primaire complète : « tout est trouvé », « le flag est certain », « la requête est prouvée », « le compte est définitivement identifié » ou « seule la valeur N manque ». Il est possible que plusieurs hypothèses ou extractions antérieures soient erronées.

## 18.5. Consigne de reprise

Le prochain agent doit repartir des sources et non des conclusions : vérifier l’énoncé dans la page, vérifier l’existence et le hash des artefacts, refaire l’extraction du PCAP, documenter la déduplication, comparer plusieurs méthodes et enregistrer les écarts dans le dépôt GitHub. Le dossier actuel sert à transmettre le travail déjà tenté et ses limites ; il ne constitue pas une validation indépendante.

La conclusion honnête au moment de la transmission est donc : **Framed comporte quatre questions ; Q1–Q3 sont affichées comme résolues historiquement ; Q4 reste non validée ; plusieurs observations réseau et hypothèses de comptage sont conservées, mais aucune réponse complète ne doit être considérée comme définitivement établie.**


# 19. Journal exact de l’analyse effectuée

Cette section décrit uniquement les opérations effectivement réalisées pendant la reprise de Framed. Elle ne transforme pas les informations héritées en découvertes personnelles et ne constitue pas une résolution supplémentaire.

## 19.1. Consultation de la plateforme

La page authentifiée `https://ctf.hackerlab.bj/challenges` a été ouverte en lecture seule. La mission **Framed** a été sélectionnée pour vérifier son état et identifier la question encore disponible. La carte indiquait **3/4**, avec Q1 `Money Trees`, Q2 `The Beginning` et Q3 `Paranoid Android` déjà résolues, et Q4 `Every Breath You Take` encore ouverte.

Le panneau de Q4 a été consulté afin de relever son titre, son identifiant interne, son énoncé et son format. Les informations utilisées dans le dossier sont :

| Élément | Valeur relevée |
|---|---|
| Question | `Every Breath You Take` |
| Identifiant interne | `23` |
| Format | `HLB2026{HH:MM:SS_HH:MM:SS_N}` |
| Machine administrateur | `192.168.58.129` / `DESKTOP-O59DAGR` |
| Serveur BCT | `198.51.100.37:443` |

Aucun champ de réponse n’a été rempli et aucun flag n’a été soumis.

## 19.2. Lecture des éléments de reprise

Le fichier `/home/ubuntu/upload/HACKERLAB_2026_—_FICHIER_DE_REPRISE_COMPLET.md` a été lu pour comprendre l’état antérieur de Framed et ne pas refaire inutilement des analyses déjà documentées. Cette lecture a fourni les noms des streams, les timestamps retenus, les hypothèses de compteur et les chemins des artefacts attendus.

Les valeurs `12:46:23`, `13:13:40`, `7`, `34`, `35`, `36`, `37` et `38` étaient déjà présentes dans ces notes. Elles ne doivent donc pas être décrites comme des valeurs que j’aurais découvertes moi-même pendant cette reprise. Le candidat `HLB2026{12:46:23_13:13:40_38}` est repris comme hypothèse héritée non validée, pas comme flag trouvé.

## 19.3. Recherche locale des artefacts

Les fichiers dont le nom contenait `framed` ont été recherchés dans `/home/ubuntu`. Cette recherche n’a retrouvé que :

```text
/home/ubuntu/http_objects/index_framed.htm
```

Le PCAP brut `/home/ubuntu/proxy-bct.pcapng`, les scripts de comptage et les tableaux dérivés mentionnés dans le document de reprise n’étaient pas disponibles sous leurs chemins attendus au moment de cette reprise. Je n’ai donc pas pu relancer une extraction `tshark`, refaire le réassemblage TCP ou recalculer indépendamment `N` à partir du PCAP brut.

## 19.4. Ce qui a été directement vérifié et ce qui ne l’a pas été

| Élément | Statut dans cette reprise |
|---|---|
| Présence de la mission Framed | Vérifiée sur la page HackerLab |
| Progression `3/4` | Vérifiée sur la page HackerLab |
| Titre et format de Q4 | Relevés sur la page et repris dans le fichier |
| Existence locale du PCAP brut | Non retrouvée lors de la recherche locale |
| Extraction de nouveaux paquets | Non effectuée, faute de PCAP disponible |
| Recalcul indépendant du compteur `N` | Non effectué |
| Flags exacts de Q1–Q3 | Non retrouvés dans les éléments consultés |
| Candidat Q4 | Présent dans les notes héritées, non validé par moi |
| Soumission à HackerLab | Aucune |

## 19.5. Réponse à la question « qu’est-ce qui a été trouvé ? »

Pour Framed, je n’ai pas retrouvé de nouveau flag exact pendant cette reprise. Q1, Q2 et Q3 étaient déjà affichées comme résolues, mais leurs flags primaires n’étaient pas présents dans les fichiers consultés. Pour Q4, j’ai confirmé le contexte de la question et constaté que le candidat présent dans les notes était `HLB2026{12:46:23_13:13:40_38}` ; je n’ai pas confirmé que ce candidat était correct.

Le résultat honnête de mon travail est donc limité à la vérification de l’état de la mission, à la lecture de l’énoncé et à l’inventaire des artefacts disponibles. La résolution de Q4 n’a pas été effectuée pendant cette reprise.

## 19.6. Limite explicite

Les sections précédentes du document contiennent également des informations historiques issues du fichier de reprise. Elles doivent être lues comme un contexte transmis, et non comme la liste exclusive des opérations exécutées dans cette session. La présente section est la référence pour distinguer les actions effectivement réalisées ici des résultats hérités.


## 20. Mise à jour de vérification du fichier brut

Une vérification ultérieure de l’espace de travail a retrouvé le PCAP brut à l’emplacement suivant :

```text
/home/ubuntu/proxy-bct.pcapng
```

La mention précédente indiquant que le fichier n’était pas retrouvé doit donc être comprise comme une **observation limitée à une recherche antérieure ou à un état intermédiaire du sandbox**, et non comme une absence actuelle du fichier. Cette découverte ne constitue pas encore une résolution de Q4 : aucune nouvelle extraction, aucun recalcul indépendant de `N` et aucune validation de flag n’ont été effectués dans le cadre de cette mise à jour.

Les agents peuvent maintenant vérifier la taille, le hash et les métadonnées du PCAP avant toute analyse. Ils doivent comparer leurs résultats avec les tableaux hérités, documenter toute divergence et conserver l’incertitude sur les deux bornes temporelles ainsi que sur `N` jusqu’à preuve reproductible.

## 20.1. Première action recommandée pour la collaboration

La première branche de travail doit être consacrée à la vérification de l’artefact, sans soumission :

```bash
ls -lh /home/ubuntu/proxy-bct.pcapng
capinfos /home/ubuntu/proxy-bct.pcapng
sha256sum /home/ubuntu/proxy-bct.pcapng
```

Cette étape est une recommandation de reprise, pas une opération exécutée et pas une conclusion sur le flag. Le dépôt GitHub doit enregistrer la taille, le hash et la date de vérification, sans nécessairement versionner le PCAP de 883 Mio si sa taille dépasse les limites du dépôt.


# 21. Protocole de collaboration inter-agents via GitHub

## 21.1. Pourquoi GitHub est nécessaire

Les agents qui reprendront cette mission ne possèdent pas automatiquement les fichiers du sandbox, l’historique de cette conversation ni les résultats des analyses précédentes. Un agent peut donc voir un résumé sans avoir la preuve correspondante, ou refaire une analyse déjà tentée avec une interprétation différente. Le dépôt GitHub doit résoudre ce problème en devenant la **source commune de vérité de Framed**.

GitHub ne sert pas seulement à stocker le fichier Markdown. Il sert simultanément de mémoire persistante, d’archive des preuves, de journal de communication, d’espace de comparaison des résultats et de mécanisme de revue entre agents. Chaque agent doit pouvoir lire ce que les autres ont fait, ajouter une analyse reproductible, signaler une erreur et proposer une correction sans effacer l’historique.

> **Règle centrale : une conclusion non écrite dans le dépôt, avec sa preuve ou sa limite, n’est pas une conclusion disponible pour les autres agents.**

## 21.2. Ce que chaque agent doit recevoir

Avant de commencer une analyse, l’agent doit cloner ou ouvrir le dépôt et lire au minimum `README.md`, le présent dossier Framed, le journal des décisions et l’inventaire des artefacts. Il ne doit pas supposer que le chemin `/home/ubuntu/...` existe dans son environnement. Les chemins locaux sont des références de provenance, pas des garanties d’accès.

| Élément | Rôle dans le dépôt |
|---|---|
| `README.md` | Résumé opérationnel, objectif Q4 et règles de sécurité |
| `HackerLab_2026_—_Framed(4).md` | Mémoire principale et historique complet |
| `STATUS.md` | État courant des hypothèses, avec date et agent |
| `DECISIONS.md` | Décisions, désaccords et raisons de rejet |
| `ARTIFACTS.md` | Inventaire des fichiers, tailles, hashes et provenance |
| `evidence/` | Extraits, tableaux, métadonnées et résultats vérifiables |
| `scripts/` | Scripts d’analyse reproductibles, sans exécution d’artefacts suspects |
| `reports/` | Rapports détaillés par agent ou par méthode |
| `questions/` | Notes propres à Q1, Q2, Q3 et Q4 |
| `candidates/` | Candidats de flag, toujours marqués non validés |
| `chat/` ou Issues GitHub | Discussions et questions entre agents |

## 21.3. Organisation recommandée du dépôt

La structure minimale recommandée est :

```text
framed-ctf/
├── README.md
├── HackerLab_2026_—_Framed(4).md
├── STATUS.md
├── DECISIONS.md
├── ARTIFACTS.md
├── questions/
│   ├── q1-money-trees.md
│   ├── q2-the-beginning.md
│   ├── q3-paranoid-android.md
│   └── q4-every-breath-you-take.md
├── evidence/
│   ├── metadata/
│   ├── tshark/
│   ├── tcp-reassembly/
│   └── screenshots-or-page-copies/
├── reports/
├── scripts/
├── candidates/
│   └── q4-candidates.md
└── hashes/
    └── SHA256SUMS.txt
```

Le PCAP brut de grande taille ne doit pas être ajouté au dépôt normal sans vérifier la taille et les limites du dépôt. Il peut être partagé par Git LFS, une release privée ou un lien officiel contrôlé. Dans tous les cas, le dépôt doit contenir son nom, sa taille, son hash SHA-256, sa provenance et la procédure de récupération. Un hash ne remplace pas le fichier, mais il permet aux agents de savoir s’ils travaillent sur le même artefact.

## 21.4. Rôles des agents

Les agents peuvent travailler en parallèle, mais chaque rôle doit produire un résultat vérifiable. Les rôles ne sont pas des personnes fixes ; plusieurs agents peuvent reprendre le même rôle pour faire une vérification indépendante.

| Rôle | Travail attendu | Livrable |
|---|---|---|
| **Archiviste** | Vérifier les fichiers, tailles, hashes et sources | `ARTIFACTS.md`, `hashes/SHA256SUMS.txt` |
| **Analyste réseau** | Refaire l’extraction et le réassemblage du PCAP | `reports/network-*.md`, fichiers dans `evidence/` |
| **Analyste TLS/HTTP2** | Déterminer les couches visibles et les limites du chiffrement | rapport séparé, jamais une supposition silencieuse |
| **Compteur indépendant** | Recalculer `N` avec une méthode différente | `reports/count-*.md` |
| **Auditeur** | Comparer les résultats, relever les incohérences | mise à jour de `DECISIONS.md` |
| **Coordinateur** | Fusionner uniquement les résultats justifiés | mise à jour de `STATUS.md` et du dossier maître |

Un agent ne doit pas modifier silencieusement l’analyse d’un autre agent. Toute correction substantielle doit être faite par commit identifiable ou Pull Request, avec une explication de la modification.

## 21.5. Convention de branches et de commits

Chaque agent doit créer une branche descriptive, par exemple :

```text
agent/tshark-inventory
agent/tcp-reassembly
agent/http2-count
agent/independent-n35-check
agent/audit-timestamps
```

Les commits doivent décrire l’action réalisée et non prétendre à une résolution prématurée. Des exemples acceptables sont :

```text
Add PCAP metadata and SHA-256 inventory
Record client burst extraction for stream 4093
Compare retransmission de-duplication methods
Document uncertainty around final HTTP request count
Reject count based only on TLS stream total
```

Un commit ne doit pas contenir un flag présenté comme validé si la plateforme ne l’a pas confirmé et si l’utilisateur n’a pas autorisé une soumission. Le dépôt est un espace d’analyse, pas un mécanisme de soumission automatique.

## 21.6. Format obligatoire pour chaque résultat

Chaque agent doit utiliser la fiche suivante dans son rapport ou sa Pull Request :

```markdown
## Objet
Question : Q4 — Every Breath You Take
Agent : <identifiant>
Date : <date UTC>
Branche : <branche>

## Source
Fichier(s) : <chemins ou URL>
Taille : <taille>
SHA-256 : <hash ou « non calculé »>

## Méthode
Décrire les filtres, commandes, seuils, règles de déduplication et unités comptées.

## Observations
Indiquer les valeurs réellement mesurées, avec les numéros de frames, streams ou lignes de sortie utiles.

## Interprétation
Séparer explicitement les observations des explications proposées.

## Résultat
Classer la conclusion comme : vérifiée, reproduite, candidate, rejetée ou indéterminée.

## Limites
Décrire ce qui pourrait rendre le résultat faux ou incomplet.

## Action proposée
Indiquer la prochaine vérification, sans soumettre de flag.
```

Cette fiche oblige les agents à communiquer par des preuves plutôt que par des affirmations générales. Une Pull Request qui ne contient qu’un nombre, sans méthode ni source, doit être renvoyée pour complément.

## 21.7. Communication et désaccords

Les Issues GitHub doivent être utilisées pour les questions ouvertes et les contradictions. Une Issue peut porter un titre tel que :

```text
Q4: Is 13:13:40 a request timestamp or only a keep-alive?
Q4: Reconcile 35 vs 36 after retransmission de-duplication
Q4: Identify whether HTTP/2 stream IDs are recoverable
Q4: Verify the seven TCP streams against the raw PCAP
```

Les commentaires doivent répondre avec une preuve, un fichier ou une reproduction. Les phrases « je pense que c’est 38 » ou « cela semble être 35 » ne suffisent pas. En cas de désaccord persistant, les deux méthodes doivent être conservées dans `DECISIONS.md` jusqu’à ce qu’un artefact tranche.

Le fichier `STATUS.md` doit contenir un tableau de synthèse de ce type :

| Élément | Valeur actuelle | Statut | Source | Prochaine vérification |
|---|---|---|---|---|
| Première heure | `12:46:23` | Observée, à revérifier | PCAP ou dérivé | Refaire l’extraction brute |
| Dernière heure | `13:13:40` | Observée, interprétation à confirmer | PCAP ou dérivé | Distinguer requête et keep-alive |
| `N=34` | Candidat | Non validé | Rapport de comptage | Comparer à HTTP/2 |
| `N=35` | Candidat | Non validé | Rapport de comptage | Refaire avec PCAP brut |
| `N=36` | Candidat rejeté historiquement | À vérifier seulement si méthode changée | Notes héritées | Ne pas soumettre |
| `N=37` | Candidat rejeté historiquement | À vérifier seulement si preuve contradictoire | Notes héritées | Ne pas soumettre |
| `N=38` | Candidat de travail | Non validé | Notes héritées | Refaire indépendamment |

## 21.8. Critère de convergence vers le bon flag

Les agents ne doivent pas choisir la valeur qui obtient le plus de votes. La convergence est atteinte seulement si au moins deux analyses indépendantes, partant du même artefact vérifié, obtiennent la même fenêtre et le même compteur, avec une définition identique de « requête HTTP ». Il faut ensuite expliquer pourquoi les keep-alives, les contrôles TLS, les retransmissions et les fermetures ne sont pas comptés comme requêtes métier.

Avant de marquer un candidat comme **réponse probable**, le dépôt doit contenir :

1. le hash du PCAP utilisé ;
2. l’extraction ou le script permettant de refaire le calcul ;
3. la liste des événements inclus ;
4. la liste des événements exclus et la raison de chaque exclusion ;
5. la définition textuelle de l’unité comptée ;
6. une seconde vérification indépendante ou une justification documentée de son absence ;
7. les divergences avec les candidats précédents ;
8. la mention explicite « aucun flag soumis ».

Même après cette convergence interne, le résultat doit rester présenté comme **flag candidat** tant que la plateforme ne l’a pas validé. Dans le cadre de cette collaboration, aucune soumission ne doit être effectuée automatiquement.

## 21.9. Sécurité du dépôt

Le dépôt doit être privé. Les secrets, mots de passe, cookies, tokens, données personnelles et fichiers de session ne doivent jamais être commités. Les artefacts suspects doivent être traités comme des données : analyse de hash, strings, métadonnées et structure statique uniquement. Aucun script contenu dans un artefact ne doit être exécuté pour « gagner du temps ».

Les agents peuvent exécuter des outils d’analyse locaux contrôlés comme `tshark`, `capinfos`, des scripts écrits spécialement pour lire des métadonnées ou des parseurs forensiques, mais ils doivent documenter la commande et éviter tout chargement automatique de contenu actif. Toute commande qui modifie le PCAP original doit être remplacée par une copie ou une sortie dérivée versionnée.

## 21.10. Résumé à copier dans le README

> Les agents ne partagent pas automatiquement le sandbox ni la conversation. Ce dépôt GitHub est donc la mémoire commune de Framed. Chaque agent doit lire les rapports existants, déposer ses preuves, documenter sa méthode, commenter les contradictions et proposer ses modifications par commit ou Pull Request. Aucun candidat ne devient un flag confirmé par simple intuition ou par vote. La mission collective est de refaire Q4 à partir du PCAP vérifié, de déterminer la définition exacte de `N`, de comparer les résultats indépendants et de conserver les limites jusqu’à preuve reproductible. Aucun flag ne doit être soumis.
