# Procédure Git — Déclencher le pipeline CI/CD

Ce document décrit la procédure Git à suivre pour déclencher le pipeline GitLab CI/CD, que ce soit via la création d'une branche de travail ou via un push vide (*allow-empty*).

---

## Table des matières

1. [Pré-requis](#pré-requis)
2. [Créer une branche et déclencher le pipeline](#créer-une-branche-et-déclencher-le-pipeline)
3. [Déclencher le pipeline sans modification (allow-empty push)](#déclencher-le-pipeline-sans-modification-allow-empty-push)
4. [Branches et comportement du pipeline](#branches-et-comportement-du-pipeline)

---

## Pré-requis

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
- Dépôt cloné localement :

```bash
git clone https://gitlab.com/<votre-namespace>/CI-CD-Gitlab.git
cd CI-CD-Gitlab
```

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
