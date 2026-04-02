# Procédure Git — Connexion, commandes essentielles & pipeline CI/CD

Ce document décrit la procédure complète pour se connecter à GitLab via SSH, maîtriser les commandes Git fondamentales (clone, fetch, push, pull…) et déclencher le pipeline GitLab CI/CD.

---

## Table des matières

1. [Configuration initiale de Git](#configuration-initiale-de-git)
2. [Générer et configurer une clé SSH](#générer-et-configurer-une-clé-ssh)
   - [Générer la paire de clés](#générer-la-paire-de-clés)
   - [Ajouter la clé à l'agent SSH](#ajouter-la-clé-à-lagent-ssh)
   - [Ajouter la clé publique à GitLab](#ajouter-la-clé-publique-à-gitlab)
   - [Tester la connexion SSH](#tester-la-connexion-ssh)
3. [Cloner le dépôt](#cloner-le-dépôt)
4. [Commandes Git essentielles](#commandes-git-essentielles)
   - [fetch](#fetch)
   - [pull](#pull)
   - [push](#push)
   - [status / log / diff](#status--log--diff)
5. [Pré-requis pipeline](#pré-requis-pipeline)
6. [Créer une branche et déclencher le pipeline](#créer-une-branche-et-déclencher-le-pipeline)
7. [Déclencher le pipeline sans modification (allow-empty push)](#déclencher-le-pipeline-sans-modification-allow-empty-push)
8. [Branches et comportement du pipeline](#branches-et-comportement-du-pipeline)

---

## Configuration initiale de Git

Avant toute chose, indiquez à Git votre identité. Ces informations apparaissent dans chaque commit.

```bash
# Définir votre nom (affiché dans les commits)
git config --global user.name "Prénom Nom"

# Définir votre adresse e-mail (doit correspondre à celle de votre compte GitLab)
git config --global user.email "vous@exemple.com"

# (Optionnel) Définir l'éditeur par défaut pour les messages de commit
git config --global core.editor "nano"   # ou "vim", "code --wait", etc.

# Vérifier la configuration
git config --list
```

---

## Générer et configurer une clé SSH

L'authentification SSH évite de saisir un mot de passe à chaque opération réseau (`push`, `fetch`, `pull`).

### Générer la paire de clés

```bash
# Générer une clé ED25519 (algorithme recommandé, plus moderne que RSA)
# Remplacez l'adresse e-mail par celle associée à votre compte GitLab
ssh-keygen -t ed25519 -C "vous@exemple.com"
```

> Le programme vous demande :
> - **Emplacement** : appuyez sur **Entrée** pour accepter `~/.ssh/id_ed25519` (par défaut)
> - **Passphrase** : choisissez une phrase secrète (recommandé) ou appuyez sur **Entrée** pour laisser vide

Si votre système ne supporte pas ED25519, utilisez RSA 4096 bits :

```bash
ssh-keygen -t rsa -b 4096 -C "vous@exemple.com"
```

### Ajouter la clé à l'agent SSH

L'agent SSH mémorise votre passphrase pour la durée de la session, évitant de la retaper à chaque commande.

```bash
# Démarrer l'agent SSH (si non démarré)
eval "$(ssh-agent -s)"

# Ajouter la clé privée à l'agent
ssh-add ~/.ssh/id_ed25519
```

> **Windows (PowerShell)** : l'agent SSH est géré par le service `ssh-agent` de Windows.
> ```powershell
> # Activer le service (une seule fois, en administrateur)
> Set-Service -Name ssh-agent -StartupType Automatic
> Start-Service ssh-agent
>
> # Ajouter la clé
> ssh-add $env:USERPROFILE\.ssh\id_ed25519
> ```

### Ajouter la clé publique à GitLab

1. Affichez et copiez votre clé publique :

```bash
# Afficher la clé publique (à copier intégralement)
cat ~/.ssh/id_ed25519.pub
```

2. Dans GitLab : **Profil (avatar) → Preferences → SSH Keys → Add new key**
3. Collez le contenu de la clé dans le champ **Key**, donnez-lui un **Title** (ex. `laptop-perso`) et cliquez sur **Add key**.

### Tester la connexion SSH

```bash
# GitLab.com
ssh -T git@gitlab.com

# Instance auto-hébergée (remplacez l'URL par la vôtre)
ssh -T git@gitlab.localdomain
```

Une réponse du type `Welcome to GitLab, @votre-pseudo!` confirme que la connexion fonctionne.

---

## Cloner le dépôt

Une fois la clé SSH configurée, clonez toujours via l'URL SSH (pas HTTPS) pour ne jamais saisir de mot de passe :

```bash
# Cloner via SSH (recommandé)
git clone git@gitlab.com:<votre-namespace>/CI-CD-Gitlab.git
cd CI-CD-Gitlab

# Cloner via HTTPS (si vous n'avez pas encore de clé SSH)
git clone https://gitlab.com/<votre-namespace>/CI-CD-Gitlab.git
cd CI-CD-Gitlab
```

---

## Commandes Git essentielles

### fetch

`git fetch` télécharge les nouveaux commits et branches depuis le dépôt distant **sans modifier** votre copie de travail locale.

```bash
# Télécharger toutes les mises à jour du remote "origin"
git fetch origin

# Télécharger uniquement la branche main
git fetch origin main

# Voir les branches distantes disponibles après le fetch
git branch -r
```

> Utile pour inspecter ce qui a changé avant d'intégrer les modifications.

### pull

`git pull` = `git fetch` + `git merge` (ou `--rebase`). Il télécharge **et** intègre les modifications distantes dans votre branche locale.

```bash
# Mettre à jour la branche courante depuis origin
git pull origin main

# Variante recommandée : utiliser rebase plutôt que merge pour un historique linéaire
git pull --rebase origin main
```

> En cas de conflits, Git vous demandera de les résoudre manuellement, puis :
> ```bash
> git add <fichier-en-conflit>
> git rebase --continue   # si --rebase
> # ou
> git commit              # si merge classique
> ```

### push

`git push` envoie vos commits locaux vers le dépôt distant.

```bash
# Pousser la branche courante vers origin (premier push : -u définit le tracking)
git push -u origin <nom-de-la-branche>

# Pousser après avoir déjà défini le tracking
git push

# Pousser toutes les branches locales
git push origin --all

# Pousser les tags
git push origin --tags

# Forcer le push (⚠️ écrase l'historique distant — à utiliser avec précaution)
git push --force-with-lease origin <nom-de-la-branche>
```

> `--force-with-lease` est plus sûr que `--force` : il vérifie que personne n'a poussé entre-temps.

### status / log / diff

```bash
# Voir l'état du répertoire de travail (fichiers modifiés, en attente de commit, etc.)
git status

# Voir les 10 derniers commits de manière condensée
git log --oneline -10

# Voir les différences entre le répertoire de travail et le dernier commit
git diff

# Voir les différences entre deux branches
git diff main feature/ma-branche
```

---

## Pré-requis pipeline

- Git installé sur votre machine (voir le [README](README.md) pour l'installation par plateforme)
- Python 3 disponible sur la machine runner :
  ```bash
  python3 --version
  python3 -m venv --help
  ```
  Si le module `venv` est absent (fréquent sur Debian/Ubuntu minimal) :
  ```bash
  sudo apt-get install -y python3-venv   # Debian / Ubuntu
  ```
  Sur macOS/Windows, `venv` est inclus avec Python 3 par défaut (voir [README](README.md)).
- Accès en écriture au dépôt GitLab
- Dépôt cloné localement (voir [Cloner le dépôt](#cloner-le-dépôt) ci-dessus)

---

## Créer une branche et déclencher le pipeline

Tout push vers le dépôt GitLab déclenche automatiquement le pipeline CI/CD. La procédure habituelle est la suivante :

### 1. Se placer sur la branche de base

```bash
# Depuis la branche principale
git checkout main

# Mettre à jour la branche locale
git pull origin main
```

### 2. Créer une nouvelle branche

```bash
# Syntaxe : git checkout -b <nom-de-la-branche>
git checkout -b feature/ma-nouvelle-fonctionnalite
```

> **Convention de nommage recommandée :**
> | Préfixe | Usage |
> |---------|-------|
> | `feature/` | Nouvelle fonctionnalité |
> | `fix/` | Correction de bug |
> | `hotfix/` | Correction urgente en production |
> | `chore/` | Maintenance, refactoring, docs |

### 3. Effectuer des modifications et committer

```bash
# Modifier des fichiers, puis :
git add .
git commit -m "feat: description de la modification"
```

### 4. Pousser la branche et déclencher le pipeline

```bash
git push origin feature/ma-nouvelle-fonctionnalite
```

> Le pipeline se déclenche automatiquement dès que GitLab reçoit le push.  
> Vous pouvez le suivre dans **GitLab → CI/CD → Pipelines**.

### 5. (Optionnel) Ouvrir une Merge Request

Après le push, GitLab affiche directement un lien pour créer une **Merge Request** vers `main` ou `develop`.

---

## Déclencher le pipeline sans modification (allow-empty push)

Il est parfois nécessaire de relancer le pipeline sans avoir apporté de modification au code (re-run après un problème d'infrastructure, test du pipeline, etc.).

### Méthode 1 — Commit vide (`--allow-empty`)

```bash
# S'assurer d'être sur la bonne branche
git checkout <nom-de-la-branche>

# Créer un commit vide
git commit --allow-empty -m "ci: trigger pipeline manually"

# Pousser pour déclencher le pipeline
git push origin <nom-de-la-branche>
```

### Méthode 2 — Depuis l'interface GitLab

1. Aller dans **GitLab → CI/CD → Pipelines**
2. Cliquer sur **"Run pipeline"**
3. Sélectionner la branche cible
4. Cliquer sur **"Run pipeline"**

> Cette méthode ne crée aucun commit et ne modifie pas l'historique Git.

---

## Branches et comportement du pipeline

| Branche | Stages déclenchés |
|---------|-------------------|
| Toutes les branches | `build` → `test` → `lint` |
| `develop` | + `build-docker` → `deploy-staging` (automatique) |
| `main` | + `build-docker` → `deploy-production` (**manuel**) |

> **Conseil :** Travaillez toujours sur une branche de fonctionnalité (`feature/`, `fix/`, etc.) et fusionnez vers `develop` pour valider en staging avant de merger vers `main` pour la production.
