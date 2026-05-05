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

## Extraction du password

En Telnet, chaque touche est envoyée dans un paquet TCP séparé (`length 1`).
On analyse le flux avec Wireshark → clic droit sur un paquet → **Follow > TCP Stream** → **Hex Dump**.

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
