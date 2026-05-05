# Level09 — Décodage d'encodage positionnel

## Fichiers fournis

```sh
ls -la ~
# -rwsr-s---  1 flag09 level09 ... level09   (binaire SUID)
# -rw-------  1 flag09 flag09  ... token     (token encodé)
```

Le fichier `token` est illisible directement (owned `flag09`). Le binaire SUID permet de l'atteindre indirectement.

## Reconnaissance

```sh
./level09 test
# urww  (chaque char décalé par sa position)
```

Le binaire **encode** son argument : chaque caractère voit sa valeur ASCII augmentée de sa position (index).

```
't' (116) + 0 = 116 = 't'
'e' (101) + 1 = 102 = 'f'  (... wait)
```

Schéma : `char[i] → char[i] + i`

Le fichier `token` contient le password de `flag09` **encodé** de cette façon — avec des bytes non-imprimables car certaines valeurs dépassent 127.

## Exploitation

Rapatrier le token en local :

```sh
scp -P 4242 level09@<vm-ip>:~/token ./notes_perso/level09/token
chmod 644 ./notes_perso/level09/token
```

Décoder en soustrayant la position de chaque byte :

```python
python3 -c "
s = open('notes_perso/level09/token', 'rb').read()
print(''.join(chr(b - i) for i, b in enumerate(s) if b - i > 0))
"
# → password de flag09
```

Se connecter et récupérer le token :

```sh
su flag09   # password = résultat du décodage
getflag
```

## Vulnérabilité

L'encodage positionnel est **réversible** — ce n'est pas du chiffrement, juste une obfuscation. Connaître l'algorithme (trouvable via `strings` ou `ltrace` sur le binaire) suffit à décoder en clair.

## Token

Voir `../flag`.
