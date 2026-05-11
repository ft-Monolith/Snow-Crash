# Level02 — Analyse de capture réseau (PCAP / Telnet)

## Fichier fourni

`~/level02.pcap` : capture tcpdump d'une session réseau, à rapatrier en local pour analyse.

```sh
scp -P 4242 level02@<vm-ip>:~/level02.pcap ./
file level02.pcap
# tcpdump capture file (little-endian) - version 2.4 (Ethernet)
```

## Reconnaissance

```sh
strings level02.pcap | head -50
```

Révèle une session **Telnet** avec les marqueurs clés :
- prompt `wwwbugs login:`
- prompt `Password:`
- réponse `Login incorrect`

Quelqu'un s'est connecté en Telnet et a tapé un password — Telnet transmet tout en clair, donc les frappes sont dans les paquets.

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

| Catégorie | Hex | Sens |
|-----------|-----|------|
| ASCII imprimable | `0x20`–`0x7e` | caractère ajouté au buffer |
| DEL / Backspace | `0x7f` | retire le **dernier** caractère du buffer |
| CR | `0x0d` | fin de saisie |

L'utilisateur a tapé, s'est trompé, a corrigé avec `Backspace`, et a retapé. Le password réel = la séquence finale **après application des backspaces**.

### Technique pour appliquer les backspaces

Le terminal interprète `\x7f` nativement. En piping les bytes bruts à `printf`, le terminal applique les effacements visuellement :

```sh
printf '\x41\x42\x7f\x43\n'   # affiche : AC
```

Reproduire la même opération sur la séquence d'octets observée dans la sortie `tshark` (les bytes envoyés par le client après le prompt `Password:`) donne le password de `flag02`. Le token résultant est stocké **uniquement** dans `../flag`.

## Exploitation

```sh
ssh level02@<vm-ip> -p 4242
su flag02         # password reconstitué (ne jamais hardcoder)
getflag           # → token à écrire dans level02/flag
```

## Vulnérabilité

**Telnet transmet tout en clair** — login, password, données applicatives. Une capture réseau passive (tcpdump local, port mirror, MITM Wi-Fi…) suffit à récupérer les credentials sans aucun déchiffrement. Aggravé ici par le mode "un octet par paquet" qui rend les frappes encore plus lisibles. Remplacé par **SSH** dans tout environnement moderne (chiffrement + intégrité + authentification serveur).
