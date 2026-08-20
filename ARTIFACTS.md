# ARTIFACTS — Framed

Ce fichier recense les artefacts réellement connus. Un chemin local n’implique pas que les autres agents y aient accès. Lorsqu’un artefact n’est pas dans le dépôt, il faut fournir une méthode de récupération contrôlée, une taille et un hash.

## Fichiers versionnés dans le dépôt

| Fichier | Rôle | Statut |
|---|---|---|
| `evidence/framed_client_bursts.txt` | Groupes de bursts client hérités | Copie versionnée, méthode à reproduire |
| `evidence/framed_bct_short_streams_detail.txt` | Détail des flux BCT courts | Copie versionnée, méthode à reproduire |
| `evidence/framed_bct_response_clusters.txt` | Clusters de réponses BCT | Copie versionnée, à comparer au PCAP |
| `evidence/framed_bct_tls_app_once.tsv` | Enregistrements applicatifs TLS dérivés | Copie versionnée, interprétation non définitive |
| `HackerLab_2026_—_Framed.md` | Mémoire maître de la mission | Document de contexte |

## PCAP brut

| Élément | Valeur actuelle |
|---|---|
| Nom | `proxy-bct.pcapng` |
| Chemin local connu | `/home/ubuntu/proxy-bct.pcapng` |
| Copie dans Git normal | Non, fichier trop volumineux pour la limite standard |
| Taille vérifiée | `926549460` octets |
| Hash SHA-256 | `06dbb94fba5d38c8aa453e7ffe3f59de42df3f5e9589b2156a11d9498b100683` |
| Provenance | PCAP brut de Framed Q4, selon l’historique de travail |
| Release GitHub privée | [framed-artifacts-v1](https://github.com/djissyak-ai/hackerlab-2026-framed-collab/releases/tag/framed-artifacts-v1) |
| URL directe de téléchargement | `https://github.com/djissyak-ai/hackerlab-2026-framed-collab/releases/download/framed-artifacts-v1/proxy-bct.pcapng` |
| Utilisation | Analyse statique uniquement |

Le PCAP a été retiré de l’historique Git avant la publication du dépôt parce qu’un fichier d’environ 883 Mio dépasse la limite GitHub de 100 Mio pour un fichier normal. Il peut être partagé séparément par Git LFS, une release privée ou un stockage de fichiers contrôlé. Toute copie doit être comparée par hash.

## Commandes de vérification proposées

```bash
ls -lh /chemin/vers/proxy-bct.pcapng
capinfos /chemin/vers/proxy-bct.pcapng
sha256sum /chemin/vers/proxy-bct.pcapng
```

Les commandes ci-dessus n’exécutent pas le contenu du PCAP ; elles lisent ses métadonnées et calculent son empreinte. Les résultats doivent être ajoutés dans une Pull Request ou dans `hashes/SHA256SUMS.txt`.

## Artefacts absents ou non confirmés

Aucun artefact supplémentaire ne doit être inventé. Si un agent possède une capture différente, un export Wireshark, une clé TLS ou un fichier de session, il doit ajouter son nom exact, son origine, sa taille, son hash et le niveau de confiance dans ce fichier avant d’en tirer une conclusion.
