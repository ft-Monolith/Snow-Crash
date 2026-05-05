# Level 01 — Hash DES-crypt dans /etc/passwd

## Vulnérabilité

Sur les vieux systèmes Unix sans `/etc/shadow`, les hashs de mots de passe sont stockés directement dans `/etc/passwd`, lisible par tous. Le hash est crackable hors ligne avec john.

## Recon

```sh
cat /etc/passwd | grep flag01
# flag01:42hDRfypTqqnw:3001:3001::/home/flag/flag01:/bin/bash
```

Le deuxième champ `42hDRfypTqqnw` est un hash DES-crypt (13 caractères, sel = 2 premiers chars).

## Exploit

Crack hors ligne avec john :

```sh
echo "flag01:42hDRfypTqqnw" > /tmp/level01.txt
john /tmp/level01.txt
john --show /tmp/level01.txt
# → abcdefg
```

```sh
su flag01   # mot de passe = résultat john
getflag
```

Token → voir `../flag`.
