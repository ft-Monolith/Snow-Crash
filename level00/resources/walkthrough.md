# Level 00 — Caesar cipher

## Vulnérabilité

Fichier texte appartenant à `flag00`, lisible par tous, contenant un mot de passe encodé en chiffre de César.

## Recon

```sh
find / -user flag00 2>/dev/null
```

→ `/usr/sbin/john` : fichier de 14 caractères, owned `flag00`, permissions `---r--r--`

```sh
cat /usr/sbin/john
# cdiiddwpgswtgt
```

## Analyse

- Pas un hash : `john` retourne "No password hashes loaded", crackstation dit "Unrecognized hash format"
- Pas du clair : `su flag00` échoue
- Que des lettres minuscules → encodage simple → Caesar

Indice : le fichier s'appelle **john** (comme John the Ripper), hint que son contenu doit être "cracké".

## Exploit

Tester tous les décalages Caesar: (dcode.fr/chiffre-cesar)


Un des résultats est le mot de passe de `flag00`. Token obtenu via `getflag` → voir `../flag`.

## Commandes finales

```sh
su flag00        # mot de passe = résultat du décodage Caesar
getflag
```
