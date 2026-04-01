# CI-CD-Gitlab

A minimal **Python / Flask** web application with a full **GitLab CI/CD** pipeline, Docker support, and step-by-step setup instructions for every major platform.

---

## Table of Contents

1. [Project structure](#project-structure)
2. [Clone the repository](#clone-the-repository)
   - [Push to a new GitLab repository](#push-to-a-new-gitlab-repository)
   - [Push to an existing GitLab repository](#push-to-an-existing-gitlab-repository)
3. [Install dependencies](#install-dependencies)
   - [Windows](#windows)
   - [Debian](#debian)
   - [Ubuntu](#ubuntu)
   - [macOS](#macos)
4. [Run locally (without Docker)](#run-locally-without-docker)
5. [Run with Docker](#run-with-docker)
6. [Run with Docker Compose](#run-with-docker-compose)
7. [CI/CD Pipeline](#cicd-pipeline)
8. [Environment variables](#environment-variables)
9. [Git workflow & triggering the pipeline](#git-workflow--triggering-the-pipeline)

---

## Project structure

```
.
├── app.py                 # Flask application
├── requirements.txt       # Production dependencies
├── requirements-dev.txt   # Development & test dependencies
├── Dockerfile             # Docker image definition
├── docker-compose.yml     # Docker Compose configuration
├── .gitlab-ci.yml         # GitLab CI/CD pipeline
└── tests/
    └── test_app.py        # Pytest unit tests
```

---

## Clone the repository

> Git must be installed on your system (see the platform-specific sections below).

```bash
git clone https://gitlab.com/<your-namespace>/CI-CD-Gitlab.git
cd CI-CD-Gitlab
```

---

### Push to a new GitLab repository

Use this when you want to host this project in a brand-new GitLab repository.

1. **Create an empty repository on GitLab** (no README, no `.gitignore`): **GitLab → New project → Create blank project**.

2. **Configure the remote and push:**

```bash
# Remove the original remote (if cloned from another source)
git remote remove origin

# Add your new GitLab repository as the remote
git remote add origin https://gitlab.com/<your-namespace>/<your-repo>.git

# Push all branches and tags
git push -u origin --all
git push -u origin --tags
```

---

### Push to an existing GitLab repository

Use this when you want to add this project's content on top of an existing GitLab repository.

```bash
# Add the existing GitLab repository as a remote (choose a name other than "origin" if already taken)
git remote add gitlab https://gitlab.com/<your-namespace>/<existing-repo>.git

# Fetch the remote history to avoid divergent-history conflicts
git fetch gitlab

# Push your current branch (adjust the branch name as needed)
git push gitlab main
```

> If the remote already has commits you don't have locally, rebase or merge first:
>
> ```bash
> git pull --rebase gitlab main
> git push gitlab main
> ```

---

## Install dependencies

### Windows

#### 1. Install Git

Download and run the installer from <https://git-scm.com/download/win>.  
During setup, choose **"Git from the command line and also from 3rd-party software"**.

#### 2. Install Python

Download Python 3.12+ from <https://www.python.org/downloads/windows/>.  
**Important:** check **"Add python.exe to PATH"** during installation.

Verify:

```powershell
python --version
pip --version
```

#### 3. Create a virtual environment & install packages

Open **PowerShell** or **Command Prompt** in the project folder:

```powershell
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements-dev.txt
```

#### 4. (Optional) Install Docker Desktop

Download from <https://www.docker.com/products/docker-desktop/> and follow the installer.  
Enable **WSL 2** backend when prompted.

---

### Debian

#### 1. Install Git & Python

```bash
sudo apt-get update
sudo apt-get install -y git python3 python3-pip python3-venv
```

Verify:

```bash
git --version
python3 --version
pip3 --version
```

#### 2. Create a virtual environment & install packages

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
```

#### 3. (Optional) Install Docker

```bash
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo usermod -aG docker $USER   # log out & back in to apply
```

---

### Ubuntu

#### 1. Install Git & Python

```bash
sudo apt-get update
sudo apt-get install -y git python3 python3-pip python3-venv
```

Verify:

```bash
git --version
python3 --version
pip3 --version
```

#### 2. Create a virtual environment & install packages

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
```

#### 3. (Optional) Install Docker

```bash
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo usermod -aG docker $USER   # log out & back in to apply
```

---

### macOS

#### 1. Install Homebrew (if not already installed)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 2. Install Git & Python

```bash
brew install git python@3.12
```

Verify:

```bash
git --version
python3 --version
pip3 --version
```

#### 3. Create a virtual environment & install packages

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
```

#### 4. (Optional) Install Docker Desktop

```bash
brew install --cask docker
```

Then open the **Docker** app from your Applications folder to start the daemon.

---

## Run locally (without Docker)

```bash
# Activate your virtual environment first (see above)
python app.py
```

The application will be available at <http://localhost:5000>.

| Endpoint   | Description            |
|------------|------------------------|
| `GET /`    | Returns a JSON greeting |
| `GET /health` | Health-check endpoint |

---

## Run with Docker

```bash
# Build the image
docker build -t ci-cd-gitlab .

# Run the container
docker run -p 5000:5000 ci-cd-gitlab
```

---

## Run with Docker Compose

```bash
# Build and start in the background
docker compose up -d --build

# View logs
docker compose logs -f

# Stop
docker compose down
```

---

## CI/CD Pipeline

The `.gitlab-ci.yml` file defines the following stages:

```
build ──► test ──► lint ──► deploy-staging ──► deploy-production
```

> **Runner type:** this pipeline is designed for a **Shell runner**.  
> Each Python job creates its own isolated virtual environment (`python3 -m venv .venv`) to avoid dependency on a system-wide `pip` and to prevent permission errors on system `site-packages`.  
> Docker jobs (`build-docker`, `deploy-*`) require Docker to be installed on the runner machine and the runner user to be in the `docker` group.

| Stage              | Job(s)             | Trigger           | Description |
|--------------------|--------------------|-------------------|-------------|
| **build**          | `build`            | every push        | Creates venv, installs production deps |
| **build**          | `build-docker`     | `main` / `develop`| Builds & pushes Docker image to GitLab Registry |
| **test**           | `test`             | every push        | Runs pytest with coverage report (isolated venv) |
| **lint**           | `lint`             | every push        | Checks code style with flake8 (isolated venv) |
| **deploy-staging** | `deploy-staging`   | `develop` branch  | Deploys automatically to staging |
| **deploy-production** | `deploy-production` | `main` branch  | **Manual** deploy to production |

### Runner prerequisites

| Requirement | Check |
|-------------|-------|
| Python 3 | `python3 --version` |
| venv module | `python3 -m venv --help` — if missing: `sudo apt-get install -y python3-venv` (Debian/Ubuntu); included by default on macOS and Windows with Python 3 |
| Docker | `docker --version` (required for `build-docker` and deploy jobs) |

### Required CI/CD variables

Configure these in **GitLab → Settings → CI/CD → Variables**:

| Variable | Description |
|----------|-------------|
| `CI_REGISTRY_USER` | GitLab Registry username (auto-provided by GitLab) |
| `CI_REGISTRY_PASSWORD` | GitLab Registry password / token (auto-provided) |
| `CI_REGISTRY` | Registry URL (auto-provided by GitLab) |

### Run tests locally

```bash
source .venv/bin/activate   # or .venv\Scripts\activate on Windows
pytest tests/ -v
```

### Check code style locally

```bash
flake8 app.py tests/ --max-line-length=120
```

---

## Environment variables

| Variable      | Default      | Description |
|---------------|--------------|-------------|
| `FLASK_ENV`   | `production` | Flask environment (`development`, `staging`, `production`) |
| `PORT`        | `5000`       | Port the app listens on |

---

## Git workflow & triggering the pipeline

For the full Git workflow — including branch creation, conventional commits, and how to trigger the pipeline with an allow-empty push — see **[GIT_WORKFLOW.md](GIT_WORKFLOW.md)**.
