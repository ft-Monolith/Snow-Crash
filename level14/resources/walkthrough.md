# Level14 — GDB : bypass ptrace anti-debug + UID spoofing sur getflag

## Reconnaissance

Aucun binaire SUID flag14, aucun fichier dans le home, aucun process. Le seul vecteur : appeler `getflag` directement en se faisant passer pour `flag14`.

```sh
ltrace getflag
# ptrace(0, 0, 1, 0, 0) = -1
# puts("You should not reverse this")
# exit(1)
```

`getflag` se protège contre le debugging avec `ptrace` — si un debugger est attaché, `ptrace` retourne `-1` et le programme quitte immédiatement.

## Vulnérabilités

1. **Anti-debug bypassable** : le check `ptrace` se fait en lisant la valeur de retour dans `eax` — on peut la falsifier depuis GDB.
2. **UID check bypassable** : comme level13, `getflag` appelle `getuid()` et compare le résultat — on force le retour à l'UID de `flag14` (3014).

## Exploitation

```sh
gdb /bin/getflag
(gdb) break ptrace
(gdb) break getuid
(gdb) run
# arrêt sur ptrace
(gdb) finish
(gdb) set $eax=0       # ptrace retourne 0 → "pas de debugger"
(gdb) continue
# arrêt sur getuid
(gdb) return (int)3014  # getuid retourne l'UID de flag14
(gdb) continue
# Check flag.Here is your token : <TOKEN>
```

L'UID `3014` est celui de `flag14` dans `/etc/passwd`.

## Token

Voir `../flag`.
