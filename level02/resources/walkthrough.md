# Level02 — Analyse de capture réseau (PCAP / Telnet)

## Fichier fourni

`~/level02.pcap` : capture tcpdump d'une session réseau.

```sh
file level02.pcap
# tcpdump capture file (little-endian) - version 2.4 (Ethernet)
```

## Reconnaissance

`strings level02.pcap` révèle une session **Telnet** avec les marqueurs clés :
- prompt `wwwbugs login:`
- prompt `Password:`
- réponse `Login incorrect`

Quelqu'un s'est connecté et a tapé un password — il est dans les paquets.

## Étape 1 — Récupérer le fichier depuis la VM

Depuis ta **machine locale** (pas depuis la VM), on copie le `.pcap` avec `scp` :

```sh
scp -P 4242 level02@<vm-ip>:~/level02.pcap ./notes_perso/level02/level02.pcap
```

Le fichier est maintenant en local, prêt à être ouvert.

## Étape 2 — Lancer Wireshark

```sh
wireshark ./notes_perso/level02/level02.pcap
```

Si Wireshark n'est pas installé :

```sh
sudo apt install wireshark
```

Wireshark s'ouvre directement sur la liste des paquets de la capture.

## Étape 3 — Suivre le flux TCP (TCP Stream)

1. Clic droit sur n'importe quel paquet Telnet dans la liste
2. **Follow > TCP Stream**
3. En haut à droite, passer l'affichage en **Hex Dump**

On voit le dialogue complet : le serveur envoie les prompts (`login:`, `Password:`), le client répond paquet par paquet (une touche = un paquet TCP de `length 1`).

## Étape 4 — Décoder les frappes

Après le prompt `Password:`, les octets envoyés par le client sont :

| Octets | Signification |
|--------|---------------|
| `66 74 5f 77 61 6e 64 72` | frappe : `ft_wandr` |
| `7f 7f 7f` | 3× DEL (0x7f = backspace) → efface `r`, `d`, `n` |
| `4e 44 52 65 6c` | frappe : `NDRel` |
| `7f` | 1× DEL → efface `l` |
| `4c 30 4c` | frappe : `L0L` |
| `0d` | Entrée |

Reconstitution : `ft_wa` + `NDRe` + `L0L` = le password de `flag02` (voir `../flag`).

## Exploitation

```sh
su flag02
# Password : voir ../flag
getflag
```

## Vulnérabilité

**Telnet transmet tout en clair** — login, password, données. Une capture réseau passive suffit à récupérer les credentials. Remplacé par SSH dans tout environnement moderne.
