# Cours Git — Guide pratique (Windows + Debian)

> Workflow visé : je code sur Windows avec GitHub Desktop, je déploie sur Debian avec `git pull`, et parfois je modifie directement sur le serveur.

---

## 1. Les concepts de base

| Terme | Signification |
|---|---|
| **Repository (repo)** | Le dossier du projet + tout son historique |
| **Commit** | Une "photo" de ton projet à un instant T, avec un message |
| **Branche** | Une ligne de travail parallèle (par défaut `main`) |
| **Remote** | Le dépôt distant (GitHub). Par défaut il s'appelle `origin` |
| **Push** | Envoyer mes commits locaux vers GitHub |
| **Pull** | Récupérer les commits de GitHub vers ma machine |
| **Clone** | Copier un repo GitHub pour la première fois |
| **Staging area** | Zone intermédiaire où l'on choisit ce qui ira dans le prochain commit |

### Les 3 zones de Git

```
Working directory  --git add-->  Staging area  --git commit-->  Repo local  --git push-->  GitHub
   (mes fichiers)               (index)                         (historique)                (remote)
```

---

## 2. Installation et configuration (Debian 13)

```bash
sudo apt update
sudo apt install git -y
git --version
```

Configuration initiale (une seule fois par machine) :

```bash
git config --global user.name "TonPseudo"
git config --global user.email "ton-email@exemple.com"
git config --global init.defaultBranch main
git config --global pull.rebase false     # pull = fetch + merge (comportement classique)
git config --global core.editor nano
```

Vérifier la config :

```bash
git config --list
```

> Utilise la même adresse e-mail que celle de ton compte GitHub pour que tes commits te soient attribués.

---

## 3. Connexion à GitHub depuis le serveur

### Option A : clé SSH (recommandé)

```bash
ssh-keygen -t ed25519 -C "debian-vps"
cat ~/.ssh/id_ed25519.pub
```

Copie la clé publique affichée, puis dans GitHub :
**Settings → SSH and GPG keys → New SSH key**.

Tester :

```bash
ssh -T git@github.com
```

Cloner ensuite avec l'URL SSH :

```bash
git clone git@github.com:TonUser/mon-bot.git
```

### Option B : Deploy key (pour un serveur qui ne fait que `pull`)

Dans le repo GitHub : **Settings → Deploy keys → Add deploy key**.
Par défaut elle est en **lecture seule**, ce qui est idéal pour un serveur de production : il peut récupérer le code mais pas pousser.
Si tu veux aussi `push` depuis le serveur, coche **Allow write access**.

### Option C : HTTPS avec token

```bash
git clone https://github.com/TonUser/mon-bot.git
```

GitHub n'accepte plus les mots de passe : il faut un **Personal Access Token** à saisir à la place du mot de passe. C'est moins pratique que SSH.

---

## 4. Les commandes essentielles

### Démarrer

```bash
git clone <url>              # copier un repo existant
git init                     # créer un nouveau repo dans le dossier courant
```

### Le cycle quotidien

```bash
git status                   # voir l'état (À FAIRE SOUVENT)
git add fichier.py           # ajouter un fichier au staging
git add .                    # ajouter tous les changements
git commit -m "Message clair"
git push                     # envoyer sur GitHub
git pull                     # récupérer depuis GitHub
```

### Consulter

```bash
git log                      # historique complet
git log --oneline            # historique compact
git log --oneline --graph --all   # avec les branches
git diff                     # changements non ajoutés au staging
git diff --staged            # changements prêts à être commit
git show <hash>              # détails d'un commit
```

### Annuler

```bash
git restore fichier.py            # annuler les modifs non commit d'un fichier
git restore --staged fichier.py   # retirer un fichier du staging (garde les modifs)
git commit --amend -m "Nouveau message"   # modifier le dernier commit (si pas encore push)
git revert <hash>                 # créer un commit qui annule un ancien commit (sûr)
```

> `git reset --hard` supprime définitivement tes modifications locales non commit. À utiliser en connaissance de cause.

---

## 5. Le workflow principal : Windows → GitHub → Debian

### Sur Windows (GitHub Desktop)

1. Modifier les fichiers dans ton éditeur.
2. Dans GitHub Desktop, cocher les fichiers à inclure.
3. Écrire un message de commit, puis **Commit to main**.
4. Cliquer sur **Push origin**.

### Sur Debian (déploiement)

```bash
cd ~/mon-bot
git pull
```

Puis, selon ton projet, redémarrer le service :

```bash
sudo systemctl restart mon-bot
# ou pour Docker :
docker compose up -d --build
```

### Script de déploiement rapide (optionnel)

Crée `deploy.sh` sur le serveur :

```bash
#!/bin/bash
set -e
cd ~/mon-bot
git pull
docker compose up -d --build
echo "Déploiement terminé"
```

```bash
chmod +x deploy.sh
./deploy.sh
```

---

## 6. Modifier directement sur le serveur (cas délicat)

C'est ici que les problèmes arrivent. Si tu modifies sur Debian **sans** pousser, puis que tu modifies aussi sur Windows, tu auras un conflit au prochain `git pull`.

### Bonne pratique : toujours commit + push depuis le serveur

```bash
cd ~/mon-bot
git status
git add .
git commit -m "Fix rapide directement sur le serveur"
git push
```

Ensuite, sur Windows, ouvre GitHub Desktop et clique sur **Fetch origin** puis **Pull origin** pour récupérer cette modification.

### Règle d'or

> Avant de commencer à modifier quelque part, fais **toujours un `git pull`** (ou **Fetch/Pull** dans GitHub Desktop) pour partir de la dernière version.

### Si `git pull` échoue à cause de modifs locales

Message typique :
```
error: Your local changes to the following files would be overwritten by merge
```

**Solution 1 : garder mes modifs** (commit puis pull)

```bash
git add .
git commit -m "Mes modifs locales"
git pull
```

**Solution 2 : mettre de côté temporairement** (stash)

```bash
git stash                # met tes modifs de côté
git pull                 # récupère la dernière version
git stash pop            # réapplique tes modifs
```

**Solution 3 : jeter mes modifs locales** (destructif)

```bash
git restore .            # annule toutes les modifs non commit
git pull
```

**Solution 4 : forcer le serveur à être identique à GitHub** (destructif)

```bash
git fetch origin
git reset --hard origin/main
```

> Très utile pour un serveur qui doit toujours refléter exactement le repo. Attention : tout ce qui n'est pas sur GitHub est perdu.

---

## 7. Résoudre un conflit de merge

Un conflit apparaît quand la même partie d'un fichier a été modifiée à deux endroits.

```
Auto-merging bot.py
CONFLICT (content): Merge conflict in bot.py
Automatic merge failed; fix conflicts and then commit the result.
```

Ouvre le fichier, tu verras :

```
<<<<<<< HEAD
version locale (celle du serveur)
=======
version venant de GitHub
>>>>>>> origin/main
```

**Étapes :**

1. Édite le fichier : garde la bonne version (ou un mélange) et **supprime les lignes** `<<<<<<<`, `=======`, `>>>>>>>`.
2. Ajoute et commit :

```bash
git add bot.py
git commit -m "Résolution du conflit"
git push
```

Pour abandonner le merge et revenir à l'état d'avant :

```bash
git merge --abort
```

GitHub Desktop propose aussi une aide pour résoudre les conflits, avec ouverture dans un éditeur.

---

## 8. Les branches

Utiles pour tester une nouveauté sans casser la version qui tourne.

```bash
git branch                    # lister les branches
git branch nouvelle-feature   # créer une branche
git switch nouvelle-feature   # changer de branche
git switch -c ma-feature      # créer ET basculer (raccourci)
git merge ma-feature          # fusionner ma-feature dans la branche courante
git branch -d ma-feature      # supprimer la branche (après merge)
git push -u origin ma-feature # pousser une nouvelle branche
```

**Exemple de workflow :**

```bash
git switch main
git pull
git switch -c test-nouvelle-commande
# ... je code, je commit ...
git push -u origin test-nouvelle-commande
# test OK → je merge sur main
git switch main
git merge test-nouvelle-commande
git push
```

Dans GitHub Desktop : **Current Branch → New Branch**, et **Branch → Merge into current branch**.

---

## 9. Le fichier `.gitignore` (très important)

Il liste ce que Git doit **ignorer** : secrets, fichiers temporaires, dépendances.

Exemple pour un bot :

```gitignore
# Secrets
.env
config.json
*.key
*.pem

# Python
__pycache__/
*.pyc
venv/
.venv/

# Node
node_modules/

# Logs et données
*.log
data/
*.db

# Système
.DS_Store
Thumbs.db
```

### Ne jamais commit de secrets

Token Discord/Telegram, mots de passe, clés API : **jamais dans le repo**. Utilise un fichier `.env` ignoré par Git, et fournis un `.env.example` sans valeurs :

```
DISCORD_TOKEN=
DB_PASSWORD=
```

### Si j'ai commit un secret par erreur

1. **Révoque immédiatement le token** (le régénérer côté Discord, API, etc.). Même si tu supprimes le fichier ensuite, il reste dans l'historique.
2. Retire le fichier du suivi Git sans le supprimer du disque :

```bash
git rm --cached .env
echo ".env" >> .gitignore
git commit -m "Retrait du .env du suivi"
git push
```

> Un fichier déjà suivi par Git n'est pas ignoré par `.gitignore` tant qu'on ne l'a pas retiré avec `git rm --cached`.

---

## 10. Bien écrire ses commits

- Un commit = **une idée** (pas 15 changements mélangés).
- Message court, à l'impératif ou au passé, qui explique **quoi** et **pourquoi**.

| Mauvais | Bon |
|---|---|
| `maj` | `Ajout de la commande /ping` |
| `fix` | `Correction du crash si le salon n'existe pas` |
| `truc` | `Mise à jour du docker-compose (port 8080)` |

---

## 11. Stash : mettre de côté

```bash
git stash                 # sauvegarde les modifs et nettoie le dossier
git stash list            # voir les stash
git stash pop             # réapplique et supprime le dernier stash
git stash drop            # supprime le dernier stash sans l'appliquer
```

---

## 12. Gérer les remotes

```bash
git remote -v                                  # voir les remotes
git remote add origin git@github.com:User/repo.git
git remote set-url origin git@github.com:User/repo.git   # changer l'URL
```

Passer d'HTTPS à SSH sur un repo déjà cloné :

```bash
git remote set-url origin git@github.com:TonUser/mon-bot.git
```

---

## 13. Fetch vs Pull

```bash
git fetch      # télécharge les nouveautés SANS toucher à tes fichiers
git pull       # = git fetch + git merge
```

Pratique pour voir ce qui a changé avant de mettre à jour :

```bash
git fetch
git log HEAD..origin/main --oneline   # commits présents sur GitHub mais pas chez moi
git diff HEAD origin/main             # voir les différences
git status                            # indique si je suis en avance / en retard
```

---

## 14. Problèmes fréquents

| Problème | Cause probable | Solution |
|---|---|---|
| `Permission denied (publickey)` | Clé SSH non ajoutée à GitHub | Vérifier `ssh -T git@github.com` et la clé |
| `fatal: not a git repository` | Mauvais dossier | `cd` dans le repo |
| `rejected ... non-fast-forward` | GitHub a des commits que je n'ai pas | `git pull` puis `git push` |
| `Your local changes would be overwritten` | Modifs locales non commit | commit, stash ou restore (cf. section 6) |
| `detected dubious ownership` | Dossier appartenant à un autre utilisateur | `git config --global --add safe.directory /chemin/du/repo` |
| Le `.gitignore` ne marche pas | Fichier déjà suivi | `git rm --cached fichier` |
| Fin de ligne (CRLF/LF) qui change tout | Différence Windows/Linux | voir ci-dessous |

### Problème des fins de ligne Windows/Linux

Windows utilise CRLF, Linux LF. Pour éviter les différences parasites, ajoute un fichier `.gitattributes` à la racine du repo :

```
* text=auto eol=lf
```

Cela est particulièrement important pour les scripts `.sh` : un script avec des CRLF échoue sous Linux avec des erreurs étranges (`bad interpreter`).

---

## 15. Aide-mémoire (cheat sheet)

```bash
# Quotidien
git status                    # où j'en suis ?
git pull                      # récupérer
git add . && git commit -m "message"   # enregistrer
git push                      # envoyer

# Consulter
git log --oneline -10         # 10 derniers commits
git diff                      # ce qui a changé

# Annuler
git restore fichier           # annuler modifs d'un fichier
git stash / git stash pop     # mettre de côté / reprendre

# Branches
git switch -c nom             # créer + aller
git switch main               # retourner sur main
git merge nom                 # fusionner

# Forcer le serveur = GitHub (destructif)
git fetch origin && git reset --hard origin/main
```

---

## 16. Récap : les bonnes habitudes

1. **`git pull` avant de commencer** à travailler, où que ce soit.
2. **`git status`** régulièrement pour savoir où tu en es.
3. **Commits petits et fréquents**, avec des messages clairs.
4. **Un seul endroit de vérité : GitHub.** Si tu modifies sur le serveur, commit + push dans la foulée.
5. **Jamais de secrets dans le repo** : `.env` + `.gitignore`.
6. Utilise des **branches** pour tester sans casser ce qui tourne.
7. En cas de doute avant une commande destructive : `git status` et `git diff`.

---

## Pour aller plus loin

- Documentation officielle : https://git-scm.com/doc
- Livre gratuit *Pro Git* (en français) : https://git-scm.com/book/fr/v2
- Apprendre les branches de façon visuelle : https://learngitbranching.js.org/?locale=fr_FR
- GitHub Actions : automatiser le déploiement à chaque push (idée de projet pour ton alternance)
