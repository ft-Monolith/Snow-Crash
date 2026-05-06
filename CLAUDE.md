# Snowcrash — Guide de collaboration

Projet de sécurité offensive de l'école 42, format CTF. ~14 niveaux à enchaîner sur une VM dédiée. Chaque niveau = un user `levelN` qui doit récupérer le password du user `flagN` pour passer au suivant.

## 1. Contexte projet

- **Rendu 42 (V3.2)** : 10 niveaux mandatory (`level00` → `level09`) + 5 bonus (`level10` → `level14`). Bonus comptent **uniquement si mandatory PARFAIT**.
- **Structure imposée par le sujet** : chaque `levelXX/` contient un fichier `flag` (le token de `getflag`) et un dossier `resources/` (tout ce qui prouve la résolution).
- **Le fichier `levelXX/flag` EST destiné à recevoir le token** (peut être vide, mais alors justifier en éval). C'est l'**unique exception** à la règle no-password.
- **Aucun autre password en clair** ailleurs (walkthrough, exploit hardcodé, commit message).
- **Aucun binaire dans le rendu** (règle 42). Les binaires rapatriés depuis la VM pour analyse vivent uniquement dans `notes_perso/` (gitignored).
- **Brute-force SSH interdit** par le sujet.
- Repo **à garder privé** (les flags committés = compromission du projet pour les autres si public).
- Connexion VM : `ssh level00@<vm-ip> -p 4242`, password initial `level00`.

## 2. Structure de rendu (imposée par le sujet)

```
.
├── CLAUDE.md
├── .gitignore
├── en.subject.pdf
├── level00/
│   ├── flag                    # token retourné par getflag (= password de level01)
│   └── resources/              # tout ce qui prouve / reproduit la résolution
│       ├── walkthrough.md      # démarche, vuln identifiée, commandes
│       ├── exploit.sh          # script reproductible (sans password hardcodé)
│       └── ...                 # sources, screenshots, notes complémentaires
├── level01/
│   ├── flag
│   └── resources/
...
├── level14/                    # dernier bonus
│   ├── flag
│   └── resources/
└── notes_perso/                # IGNORÉ par git — binaires VM, dumps, brouillons
    └── passwords.txt
```

Conventions :
- `levelXX/flag` = **uniquement** le token brut retourné par `getflag` (rien d'autre, pas de prose).
- `walkthrough.md` = démarche complète. Référence le password obtenu comme `<TOKEN>` ou pointe vers `../flag`, ne le réécris pas.
- `exploit.sh` = script reproductible. Lit les credentials depuis env / argv / stdin, jamais en dur.
- Tout artefact non commitable (binaires rapatriés, dumps lourds, brouillons) → `notes_perso/`.

## 3. Méthodologie par niveau (checklist)

Reconnaissance systématique avant tout :
```sh
id                                          # mes UID/GID
ls -la ~ /tmp /var/tmp                      # fichiers visibles
getfacl .                                   # ACLs cachées
cat /etc/passwd | grep flagN                # hash en clair ? (vieux systèmes sans shadow)
find / -user flagN 2>/dev/null              # fichiers du flag
find / -perm -u=s -type f 2>/dev/null       # binaires SUID
find / -group levelN 2>/dev/null            # ressources accessibles
ps -ef | grep flagN                         # process tournant en flag
ss -tlnp ; ss -ulnp                         # services réseau locaux
crontab -l ; cat /etc/crontab               # tâches planifiées
```

Sur un binaire suspect :
```sh
file ./bin ; ldd ./bin ; checksec ./bin
strings -n 6 ./bin | less
ltrace ./bin ; strace -f ./bin
gdb -q ./bin                                # disas main, info func, etc.
```

Sur un service web (souvent port 4747 / 80) : `curl -i`, view-source, params GET/POST, cookies, robots.txt, headers Server.

## 4. Boîte à outils (one-liners réutilisables)

```sh
# Recherche SUID owned par flagN
find / -perm -4000 -user flagN 2>/dev/null

# Dump trafic local
sudo tcpdump -i lo -A -s0 -w /tmp/dump.pcap

# Décodages rapides
echo "..." | base64 -d
echo "..." | tr 'A-Za-z' 'N-ZA-Mn-za-m'      # ROT13
xxd ./fichier | less

# Crack
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
hydra -l user -P wordlist.txt ssh://target

# Stégano
file image.jpg ; exiftool image.jpg
binwalk -e image.jpg
steghide extract -sf image.jpg

# Connexion rapide au niveau suivant (lit depuis notes_perso, jamais hardcodé)
sshpass -p "$(cat notes_perso/passwords.txt | grep ^levelXX | cut -d= -f2)" ssh levelXX@<vm-ip>
```

## 5. Règles pour Claude (toi)

- Le **seul endroit** où un password/token peut apparaître dans le repo est `levelXX/flag` (exigence du sujet). Partout ailleurs (CLAUDE.md, walkthrough.md, exploit.sh, commit message) → **JAMAIS de password en clair**. Référence-le comme `<TOKEN>` ou pointe vers `../flag`.
- Si je te montre la sortie d'un binaire ou un dump, **aide-moi à raisonner sur la vuln** — n'essaie pas de deviner le flag à ma place.
- Privilégie expliquer la **technique** (format string, BoF, race condition, ret2libc, command injection, path traversal, etc.) plutôt que livrer la réponse. Je suis là pour apprendre.
- Quand je bloque, propose **2-3 pistes ordonnées par probabilité** avant de creuser une seule.
- Pour les writeups : tu peux les rédiger, mais relis-toi pour vérifier qu'aucun password ne traîne. Référence-les comme `<PASSWORD_LEVELN+1>`.
- Ne jamais suggérer `git add .` ou `git add -A` sur ce repo — toujours add fichier par fichier (risque de leak via fichiers oubliés).
- Si tu vois un fichier qui ressemble à un dump ou un password, **alerte-moi avant de le lire en entier** dans la conversation.
- Si je veux ajouter au repo un fichier qui ressemble à un binaire ELF (sans extension, taille > quelques Ko, magic bytes `\x7fELF`), **bloque-moi** et propose de le déplacer dans `notes_perso/` — règle 42 : aucun binaire dans le rendu.

## 6. Glossaire rapide

- **flag** : le password du user `flagN`, à récupérer pour valider un niveau.
- **SUID** : binaire qui s'exécute avec l'UID de son owner (souvent `flagN`) → vecteur d'escalade clé.
- **GTFOBins** : ressource pour exploiter les binaires SUID légitimes.
- **ASLR** : randomisation des adresses mémoire (`/proc/sys/kernel/randomize_va_space`).
- **NX / DEP** : pile non-exécutable.
- **Canary** : valeur sentinelle anti-buffer-overflow.
- **PIE** : binaire à base aléatoire.
- **ret2libc** : technique de ROP qui saute dans `system()` de la libc pour bypass NX.
- **Format string** : `printf(user_input)` → lecture/écriture mémoire arbitraire via `%x`, `%n`.
- **Race condition / TOCTOU** : exploiter la fenêtre entre check et use d'un fichier.
- **Path injection** : manipuler `$PATH` ou `$IFS` pour qu'un script SUID exécute mon binaire.

## 7. Workflow VM ↔ machine locale

La VM est minimaliste (peu d'outils, éditeurs basiques) et `/tmp` / `/var/tmp` sont resettés. Ne jamais bosser directement dessus.

**Cycle standard** :
1. SSH dans la VM uniquement pour la **recon** et l'**exécution finale** de l'exploit.
2. Récupérer les binaires/sources à analyser **chez soi** avec `scp`.
3. Reverse / dev de l'exploit en local dans `notes_perso/levelXX/` (Ghidra, gdb, IDE).
4. Renvoyer l'exploit prêt sur la VM avec `scp`, l'exécuter via SSH, capturer le token.
5. Rédiger walkthrough propre dans `levelXX/` (sans password).

**Commandes scp clés** :
```sh
# Download : VM → local
scp levelXX@<vm-ip>:/chemin/sur/vm ./notes_perso/levelXX/

# Upload : local → VM (vers /tmp car writable)
scp ./exploit.sh levelXX@<vm-ip>:/tmp/

# Récursif (dossier entier)
scp -r levelXX@<vm-ip>:/home/levelXX/sources ./notes_perso/levelXX/

# Port SSH non-standard
scp -P <port> ...
```

**Setup recommandé `~/.ssh/config`** pour aliaser chaque niveau :
```
Host snowXX
    HostName <vm-ip>
    User levelXX
```
→ `ssh snow03` et `scp ./e.sh snow03:/tmp/` au lieu de retaper user@ip à chaque fois.

## 8. Progression

**Mandatory** (10) :

| Niveau   | Statut | Technique principale | Date |
|----------|--------|----------------------|------|
| level00  | ✅     |                      |      |
| level01  | ✅     |                      |      |
| level02  | ✅     | Analyse PCAP / Telnet en clair | 2026-05-05 |
| level03  | ✅     | SUID + Path Injection | 2026-05-05 |
| level04  | ✅     | Command Injection (CGI Perl) | 2026-05-05 |
| level05  | ✅     | Cron Job + écriture dossier surveillé | 2026-05-05 |
| level06  | ✅     | PHP preg_replace /e (Code Injection) | 2026-05-05 |
| level07  | ✅     | SUID + Command Injection via env var | 2026-05-05 |
| level08  | ✅     | Bypass filtre strstr par lien symbolique | 2026-05-05 |
| level09  | ✅     | Décodage encodage positionnel (index+ASCII) | 2026-05-05 |

**Bonus** (5, comptent uniquement si mandatory PARFAIT) :

| Niveau   | Statut | Technique principale | Date |
|----------|--------|----------------------|------|
| level10  | ✅     | Race condition TOCTOU (access vs open) | 2026-05-05 |
| level11  | ✅     | Command Injection via Lua io.popen | 2026-05-06 |
| level12  | ✅     | Command Injection CGI Perl + glob uppercase bypass | 2026-05-06 |
| level13  | ✅     | UID Spoofing via GDB (bypass LD_PRELOAD SUID) | 2026-05-06 |
| level14  | ✅     | GDB bypass ptrace anti-debug + UID spoofing | 2026-05-06 |

*(pas de détails de vuln ici — garde-les dans `notes_perso/levelXX/` puis transfère le walkthrough propre dans `levelXX/resources/walkthrough.md`)*
