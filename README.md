# 🦙 Ollama Linux Installer

Script d'installation automatisé pour **Ollama** sur Linux avec détection intelligente de l'architecture, support GPU (NVIDIA/AMD) et configuration systemd complète.

## 📖 Qu'est-ce qu'Ollama ?

[Ollama](https://ollama.com) permet d'exécuter des modèles de langage (LLM) en local sur votre machine Linux. Ce script automatise entièrement son installation et sa configuration.

## ✨ Fonctionnalités

### Installation Automatique
- 🔍 Détection automatique de l'architecture (amd64, arm64)
- 📥 Téléchargement depuis ollama.com
- 🔧 Installation dans `/usr/local` ou `/usr`
- 🧹 Nettoyage des versions précédentes
- 🔗 Création de liens symboliques automatiques

### Support GPU
- 🎮 **NVIDIA** : Installation automatique des drivers CUDA
- 🔴 **AMD** : Installation automatique ROCm
- 🤖 **NVIDIA JetPack** : Support Jetson (JetPack 5 et 6)
- 🪟 **WSL2** : Détection GPU via nvidia-smi
- ⚡ Détection via `lspci` ou `lshw`

### Configuration Système
- 👤 Création utilisateur système `ollama`
- ⚙️ Configuration **systemd service** automatique
- 🎯 Ajout aux groupes `render` et `video` pour GPU
- 🚀 Démarrage automatique au boot
- 🌐 API disponible sur `127.0.0.1:11434`

### Distributions Supportées
- Ubuntu / Debian
- CentOS / RHEL
- Fedora
- Rocky Linux
- Amazon Linux
- Arch Linux
- WSL2 (Windows Subsystem for Linux)

## 🚀 Installation Rapide

### Méthode Simple (Recommandée)

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Installation depuis ce Dépôt

```bash
# Cloner le dépôt
git clone https://github.com/ledokter/ollama-installer.git
cd ollama-installer

# Rendre exécutable
chmod +x install-ollama.sh

# Installer
sudo ./install-ollama.sh
```

### Installer une Version Spécifique

```bash
export OLLAMA_VERSION="0.1.20"
curl -fsSL https://ollama.com/install.sh | sh
```

## 📋 Prérequis

### Outils Requis

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install curl pciutils lshw -y

# CentOS/RHEL/Rocky
sudo yum install curl pciutils lshw -y

# Fedora
sudo dnf install curl pciutils lshw -y

# Arch Linux
sudo pacman -S curl pciutils lshw
```

### Dépendances

- `curl` - téléchargement
- `awk`, `grep`, `sed` - parsing
- `sudo` - privilèges élevés
- `lspci` ou `lshw` - détection GPU (optionnel)

## 💻 Utilisation

### Commandes de Base

```bash
# Télécharger un modèle
ollama pull llama3

# Lancer une conversation
ollama run llama3

# Lister les modèles
ollama list

# Supprimer un modèle
ollama rm llama3
```

### Gestion du Service

```bash
# Vérifier le statut
sudo systemctl status ollama

# Démarrer
sudo systemctl start ollama

# Arrêter
sudo systemctl stop ollama

# Redémarrer
sudo systemctl restart ollama

# Voir les logs
sudo journalctl -u ollama -f
```

### API REST

L'API est accessible sur `http://127.0.0.1:11434` :

```bash
# Générer du texte
curl http://localhost:11434/api/generate -d '{
  "model": "llama3",
  "prompt": "Pourquoi le ciel est bleu ?"
}'

# Lister les modèles
curl http://localhost:11434/api/tags
```

## 🎮 Support GPU

### NVIDIA GPU

Le script installe automatiquement :
- Dépôts CUDA officiels NVIDIA
- Drivers NVIDIA (cuda-drivers ou nvidia-driver-latest)
- Kernel headers si nécessaire
- Modules kernel (nvidia, nvidia_uvm)

**Détection** : Vendor ID `10de` (NVIDIA)

**Distributions supportées** :
- Ubuntu, Debian
- CentOS/RHEL 7, 8, 9
- Rocky Linux
- Fedora (jusqu'à 39)
- Amazon Linux

### AMD GPU

Télécharge automatiquement la version ROCm optimisée.

**Détection** : Vendor ID `1002` (AMD)

### NVIDIA JetPack

Support natif pour :
- **JetPack 6** (R36) - Jetson Orin
- **JetPack 5** (R35) - Jetson Xavier/Nano

Détection via `/etc/nv_tegra_release`

### Mode CPU Uniquement

Si aucun GPU n'est détecté, Ollama fonctionne en mode CPU avec un avertissement.

## 🪟 Support WSL2

Le script détecte automatiquement WSL2 et adapte l'installation.

**Prérequis WSL2** :
- Systemd activé
- GPU passthrough NVIDIA configuré

**Activer systemd** :

Éditez `/etc/wsl.conf` :
```ini
[boot]
systemd=true
```

Redémarrez WSL :
```powershell
wsl --shutdown
```

**Note** : WSL1 n'est **pas supporté**.

## 📂 Architecture d'Installation

### Chemins Créés

```
/usr/local/ollama              # Binaire principal
/usr/local/bin/ollama          # Lien symbolique
/usr/share/ollama              # Home utilisateur ollama
/etc/systemd/system/ollama.service  # Service systemd
/etc/modules-load.d/nvidia.conf     # Modules NVIDIA (si GPU)
```

### Utilisateur et Groupes

```bash
# Utilisateur système
ollama:x:999:999::/usr/share/ollama:/bin/false

# Groupes
- ollama (groupe principal)
- render (accès GPU)
- video (accès GPU)
```

### Service Systemd

Fichier `/etc/systemd/system/ollama.service` :

```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

[Install]
WantedBy=default.target
```

## ⚙️ Configuration Avancée

### Variables d'Environnement

```bash
# Version spécifique
export OLLAMA_VERSION="0.1.20"
./install-ollama.sh

# Forcer mode CPU (pas de GPU)
export OLLAMA_SKIP_GPU=1
./install-ollama.sh
```

### Personnaliser le Service

Éditez le service :
```bash
sudo systemctl edit ollama
```

Ajoutez des variables :
```ini
[Service]
# Écouter sur toutes les interfaces
Environment="OLLAMA_HOST=0.0.0.0:11434"

# Utiliser plusieurs GPUs
Environment="OLLAMA_NUM_GPU=2"

# Augmenter la limite mémoire
Environment="OLLAMA_MAX_LOADED_MODELS=5"
```

Redémarrer :
```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

### Changer le Port

```bash
sudo systemctl edit ollama
```

```ini
[Service]
Environment="OLLAMA_HOST=127.0.0.1:11435"
```

```bash
sudo systemctl restart ollama
```

## 🔧 Désinstallation

```bash
# Arrêter et désactiver le service
sudo systemctl stop ollama
sudo systemctl disable ollama

# Supprimer les fichiers
sudo rm -rf /usr/local/ollama
sudo rm /usr/local/bin/ollama
sudo rm /etc/systemd/system/ollama.service

# Supprimer l'utilisateur
sudo userdel -r ollama

# Recharger systemd
sudo systemctl daemon-reload
```

## 🐛 Dépannage

### Le script demande sudo

```bash
# Exécuter avec sudo
sudo ./install-ollama.sh

# Ou en tant que root
su -
./install-ollama.sh
```

### Architecture non supportée

Le script supporte uniquement `x86_64`, `aarch64`, `arm64`.

Vérifiez votre architecture :
```bash
uname -m
```

### Le service ne démarre pas

```bash
# Vérifier les logs
sudo journalctl -u ollama -n 50 --no-pager

# Tester manuellement
/usr/local/bin/ollama serve

# Vérifier les permissions
ls -la /usr/local/ollama
```

### GPU non détecté

```bash
# Vérifier la détection
lspci | grep -i nvidia
lspci | grep -i amd

# Vérifier les drivers NVIDIA
nvidia-smi

# Réinstaller les tools de détection
sudo apt install pciutils lshw
```

### Erreur de téléchargement

```bash
# Tester la connectivité
curl -I https://ollama.com/download/ollama-linux-amd64.tgz

# Via proxy
export http_proxy=http://proxy:8080
export https_proxy=http://proxy:8080
./install-ollama.sh
```

### Port 11434 déjà utilisé

```bash
# Trouver le processus
sudo lsof -i :11434

# Changer le port d'Ollama
sudo systemctl edit ollama
# Ajouter : Environment="OLLAMA_HOST=127.0.0.1:11435"

sudo systemctl restart ollama
```

### Permissions GPU

```bash
# Vérifier l'appartenance aux groupes
groups $(whoami)

# Ajouter manuellement aux groupes
sudo usermod -aG render,video $(whoami)

# Déconnexion/reconnexion requise
```

## 📊 Exemple d'Installation Complète

```bash
$ sudo ./install-ollama.sh
>>> Installing ollama to /usr/local
>>> Downloading Linux amd64 bundle
████████████████████████████████████████ 100%
>>> Making ollama accessible in the PATH in /usr/local/bin
>>> Creating ollama user...
>>> Adding ollama user to render group...
>>> Adding ollama user to video group...
>>> Adding current user to ollama group...
>>> Creating ollama systemd service...
>>> Enabling and starting ollama service...
>>> NVIDIA GPU detected.
>>> Installing NVIDIA repository...
>>> Installing CUDA driver...
████████████████████████████████████████ 100%
>>> NVIDIA GPU ready.
>>> The Ollama API is now available at 127.0.0.1:11434.
>>> Install complete. Run "ollama" from the command line.

$ ollama --version
ollama version 0.1.20

$ ollama pull llama3
pulling manifest
pulling ff82381e2bea... 100% ▕████████████████▏ 4.7 GB
pulling 43070e2d4e53... 100% ▕████████████████▏ 11 KB
success

$ ollama run llama3
>>> Bonjour, comment ça va ?
Bonjour ! Je vais bien, merci ! Comment puis-je vous aider aujourd'hui ?
```

## 🔒 Sécurité

### Par Défaut

- ✅ API écoute sur `127.0.0.1` uniquement (localhost)
- ✅ Utilisateur dédié sans shell (`/bin/false`)
- ✅ Pas d'authentification nécessaire en local
- ✅ Permissions restrictives sur les fichiers

### Exposition Publique (⚠️ Attention)

**Pour exposer l'API sur le réseau** :

```bash
sudo systemctl edit ollama
```

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
```

```bash
sudo systemctl restart ollama
```

**⚠️ DANGER** : Aucune authentification par défaut !

**Recommandation** : Utilisez un reverse proxy avec auth :

```nginx
# nginx
location /ollama/ {
    auth_basic "Ollama API";
    auth_basic_user_file /etc/nginx/.htpasswd;
    proxy_pass http://127.0.0.1:11434/;
}
```

## 📚 Ressources

### Documentation Officielle

- [Ollama Documentation](https://github.com/ollama/ollama/tree/main/docs)
- [API Reference](https://github.com/ollama/ollama/blob/main/docs/api.md)
- [Model Library](https://ollama.com/library)
- [Modelfile Guide](https://github.com/ollama/ollama/blob/main/docs/modelfile.md)

### Guides GPU

- [NVIDIA CUDA Installation](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/)
- [AMD ROCm Installation](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html)
- [WSL2 GPU Support](https://learn.microsoft.com/en-us/windows/wsl/tutorials/gpu-compute)

### Communauté

- [Ollama GitHub](https://github.com/ollama/ollama)
- [Discussions](https://github.com/ollama/ollama/discussions)
- [Discord](https://discord.gg/ollama)

## 🤝 Contribution

Les contributions sont bienvenues !

1. Fork ce dépôt
2. Créez une branche : `git checkout -b feature/amelioration`
3. Testez sur plusieurs distributions
4. Committez : `git commit -m "Description"`
5. Push : `git push origin feature/amelioration`
6. Ouvrez une Pull Request

### Guidelines

- Testez sur Ubuntu, Debian, CentOS minimum
- Conservez la compatibilité POSIX shell
- Ajoutez des messages de status clairs
- Documentez les nouvelles fonctionnalités

## 📝 Changelog

### v1.0 (2026-02-03)
- 🎉 Documentation complète pour GitHub
- ✨ Support NVIDIA CUDA automatique
- ✨ Support AMD ROCm
- ✨ Support NVIDIA JetPack (5 et 6)
- ✨ Détection WSL2
- ✨ Configuration systemd complète
- ✨ Installation multi-distributions

## ⚖️ Licence

MIT License

```
Copyright (c) 2026 Ollama Team

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 🙏 Crédits

- **Ollama Team** - Script d'installation original
- **ledokter** - Documentation et packaging GitHub
- Communauté NVIDIA - Documentation CUDA
- Communauté AMD - Documentation ROCm

## 📬 Contact

**Documentation par** : [ledokter](https://github.com/ledokter)

**Script original** : [Ollama](https://ollama.com)

---

⭐ **Si ce guide vous aide, donnez une étoile au projet !**
```
