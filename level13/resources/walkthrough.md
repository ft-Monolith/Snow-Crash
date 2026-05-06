# Level13 — UID Spoofing via GDB (LD_PRELOAD bypass)

## Reconnaissance

```sh
ltrace ./level13
# getuid() = 2013
# getuid() = 2013
# printf("UID %d started us but we we expect 4242", 2013)
# exit(1)
```

Le binaire vérifie que l'UID courant est `4242` — aucun utilisateur ne l'a sur ce système.

## Tentative LD_PRELOAD

Créer une fausse `getuid()` via `LD_PRELOAD` ne fonctionne pas : Linux ignore `LD_PRELOAD` sur les binaires SUID pour éviter l'escalade de privilèges.

## Exploitation — GDB

GDB permet de forcer la valeur de retour de `getuid()` avant que `main` la lise.

```sh
gdb ./level13
(gdb) break getuid
(gdb) run
# s'arrête au premier appel getuid
(gdb) return 4242
(gdb) continue
# s'arrête au deuxième appel getuid
(gdb) return 4242
(gdb) continue
# your token is <TOKEN>
```

`return 4242` dans GDB force la fonction à retourner immédiatement avec la valeur `4242` sans l'exécuter — `main` croit que l'UID est `4242` et donne le token.

## Token

Voir `../flag`.
