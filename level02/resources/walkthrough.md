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

## Extraction du flux TCP

En Telnet, chaque touche est envoyée dans un paquet TCP séparé (`length 1`). On extrait le flux en hex avec `tshark` (équivalent CLI de Wireshark) :

```sh
tshark -r level02.pcap -q -z follow,tcp,raw,0
```

Décomposition de la commande :
- `-r level02.pcap` : lit depuis le pcap au lieu de capturer en live
- `-q` : silence le résumé paquet-par-paquet (sans ça, bruit énorme)
- `-z follow,tcp,raw,0` : produit le rapport "Follow TCP Stream"
  - `follow` : type de rapport = suivre un flux
  - `tcp` : protocole
  - `raw` : format hex brut (préserve les bytes non-imprimables comme `0x7f`)
  - `0` : numéro du stream (le premier — vu qu'il n'y en a qu'un ici)

Dans la sortie : lignes **non indentées** = client → serveur (les frappes), **indentées** = serveur → client.

## Reconstitution du password

Après l'envoi du prompt serveur `Password:` (`50 61 73 73 77 6f 72 64 3a 20`), le client envoie ses frappes octet par octet jusqu'au `0x0d` (Entrée). La séquence mélange trois types de bytes :

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
