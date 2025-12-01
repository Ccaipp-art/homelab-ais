🧾 SYNTHÈSE COMPLÈTE – Installation d’une VM Ubuntu Desktop pour poste de travail administrateur
# 1. 🎯 **Objectif global du projet**

Mettre en place un **poste de travail Ubuntu Desktop 22.04 LTS** dans une machine virtuelle VMware Workstation Player (Windows 11), configurable, stable et adaptée à un environnement d’apprentissage « Administrateur Infrastructure & Sécurité ».

Ce poste servira à :

- la gestion des autres VM (Ubuntu Server, Rocky Linux)
- les outils devops (VSCode, Git, Ansible…)
- les tests réseaux
- l’automatisation
- la documentation Notion

---

# 2. ⚙️ **Préparation de Windows 11**

### Objectif

Assurer la compatibilité maximale entre Windows, VMware et la virtualisation.

### Actions réalisées

- Installation de **VMware Workstation Player**.
- Activation des fonctionnalités Windows nécessaires :
    - Plateforme de l’hyperviseur Windows
    - Virtual Machine Platform
    - Sous-système Windows pour Linux (optionnel mais présent)

### Constats importants

- Windows 11 réactive Hyper-V automatiquement → **pas un problème** pour VMware.
- Vérification que la virtualisation CPU (Intel VT-x / AMD-V) est active.

### Résultat

Le PC est parfaitement configuré pour exécuter des VMs dans VMware Player.

---

# 3. 💾 **Téléchargement & choix de la version Ubuntu**

### Ce qui a été tenté

- Ubuntu Desktop 24.04 LTS → **échec**, l’installateur Subiquity plante systématiquement dans VMware.

### Analyse de l’erreur

- Bug connu du nouvel installateur Ubuntu 24.04 dans VMware → pas utilisable pour un poste Desktop stable.

### Solution choisie

✔️ Téléchargement de **Ubuntu 22.04.5 LTS Desktop**, version fiable, robuste et éprouvée.

### Résultat

Image ISO prête, adaptée à une installation sans erreur.

---

# 4. 🏗️ **Création de la machine virtuelle dans VMware**

### Configuration retenue

- OS : Ubuntu 64-bit
- RAM : **4 Go**
- CPU : **2 cœurs**
- Disque : **40 Go**, format en un seul fichier
- Réseau : **NAT**
- Affichage : 3D accélérée (2 Go VRAM)
- USB : 2.0 (plus stable)
- **Emplacement** : disque dur externe (pour isolation + portabilité)

### Résultat

VM créée proprement, prête pour installation.

---

# 5. 🐧 **Installation d’Ubuntu Desktop 22.04**

### Étapes suivies

- Choix langue / clavier en français
- Mode “Installation normale”
- Téléchargement des mises à jour : activé
- Installation des logiciels tiers : activé
- Partitionnement : “Effacer le disque et installer Ubuntu”
    
    → Sans risque, car disque virtuel
    
- Création du compte utilisateur (ex. `brice`)
- Installation réussie

### Problème important évité

- Aucun plantage contrairement à Ubuntu 24.04
- L’installation Ubiquity (ancien installeur) est stable dans VMware

### Résultat

Ubuntu Desktop installé, fonctionnel.

---

# 6. 🧹 **Post-installation du système**

### Mise à jour complète

Commande exécutée :

```
sudo apt update && sudo apt full-upgrade -y

```

### Problème rencontré

- APT verrouillé par `unattended-upgrades`
    
    → Cela s’est résolu automatiquement (comportement normal juste après installation).
    

### Résultat

Système entièrement patché et propre.

---

# 7. 🛠️ **Installation des outils essentiels (sysadmin & réseau)**

Outils installés :

- Réseau : `net-tools`, `iproute2`, `dnsutils`, `traceroute`, `nmap`, `iputils-ping`
- Téléchargement : `curl`, `wget`
- Débogage : `htop`
- Compression : `unzip`
- Infos système : `neofetch`
- Dépôts : `software-properties-common`
- Contrôle de version : `git`

### Résultat

Environnement système complet pour les futurs labs réseaux, Ansible, scripts…

---

# 8. 📝 **Installation propre de VSCode**

### Problème rencontré

- Ubuntu propose `code` uniquement via Snap → non adapté (lent, sandboxé).
- Ajout du dépôt Microsoft nécessaire.

### Solution

Ajout manuel de la clé GPG et du dépôt officiel :

```
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo install -o root -g root -m 644 microsoft.gpg /etc/apt/trusted.gpg.d/
echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" \
| sudo tee /etc/apt/sources.list.d/vscode.list
sudo apt update
sudo apt install -y code

```

### Résultat

VSCode **officiel**, performant, autonome, mis à jour automatiquement.

---

# 9. 🔐 **Configuration Git (identité + clé SSH)**

### Actions

- Définition du nom + email
- Génération d’une clé SSH moderne :

```
ssh-keygen -t ed25519 -C "email"

```

### Résultat

Git prêt pour GitHub / GitLab / Ansible.

---

# 10. 📁 **Création d’un workspace professionnel**

Arborescence créée :

```
~/workspace
│── scripts
│── ansible
└── tests

~/docs

```

### Objectif

Fournir une structure claire pour :

- scripts (bash, python)
- playbooks Ansible
- tests techniques
- documentation locale

### Résultat

Un environnement rangé, logique et adapté au métier.

---

# 11. 🔌 **Extensions VSCode essentielles**

Installées :

- YAML (Red Hat)
- Ansible (Red Hat)
- GitLens
- ShellCheck
- Docker
- Markdown All in One

### Résultat

VSCode devient un outil complet d’administration et de développement.

---

# 12. 💾 **Création d’un “snapshot manuel” de la VM**

### Contexte

VMware Player **ne propose pas** de snapshots → solution professionnelle :

➡️ **copie complète du dossier de la VM**.

### Actions

1. Éteindre correctement la VM
2. Fermer VMware Player
3. Supprimer les fichiers `.lck`
4. Copier le dossier de la VM vers :

```
Ubuntu-desktop_backup

```

### Résultat

Snapshot complet, portable, restaurable en 10 secondes.

---
Parfait, on va compléter ta synthèse avec tout ce qu’on a ajouté depuis 👍

Je repars de la synthèse précédente, et j’ajoute la **suite logique** comme si c’était la “Phase 2 : personnalisation du poste admin”.

Tu peux copier-coller tel quel dans Notion.

---

## PHASE 2 : Personnalisation & productivité du poste administrateur

---

## 13. 🔗 Connexion GitHub ↔ VM Ubuntu (SSH)

### 🎯 Objectif

Pouvoir cloner / pousser des dépôts GitHub depuis la VM **sans mot de passe**, de manière sécurisée.

### ⚙️ Actions

1. Génération d’une clé SSH `ed25519` (si pas déjà fait) :

   ```bash
   ssh-keygen -t ed25519 -C "email_github"
   ```

2. Affichage de la clé publique :

   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

3. Ajout de la clé dans GitHub :
   **GitHub → Settings → SSH and GPG Keys → New SSH key**

   * Title : `Ubuntu-Desktop`
   * Type : Authentication
   * Key : coller la clé publique

4. Test de la connexion :

   ```bash
   ssh -T git@github.com
   ```

### ✅ Résultat

* La VM est **reconnue par GitHub**.
* `git clone`, `git pull`, `git push` fonctionnent **sans mot de passe**.
* Préparation idéale pour stocker scripts, playbooks Ansible, labs, etc.

---

## 14. 🔐 Installation d’un gestionnaire de mots de passe (KeePassXC)

### 🎯 Objectif

Gérer proprement les mots de passe et secrets du lab (GitHub, comptes Linux, futures VMs, etc.).

### ⚙️ Actions

Installation de KeePassXC via PPA recommandé :

```bash
sudo add-apt-repository ppa:phoerious/keepassxc
sudo apt update
sudo apt install keepassxc -y
```

Organisation :

* Création d’une base `Passwords.kdbx` (ex : dans `~/docs/Passwords.kdbx`)
* Classement par catégories :

  * Comptes Linux
  * GitHub / GitLab
  * Services du lab (DNS, web, etc.)

### ✅ Résultat

* Tous les mots de passe du lab sont **centralisés, chiffrés, sauvegardables**.
* Tu peux documenter les accès sans les mettre en clair dans Notion.

---

## 15. 🐚 Passage à Zsh + Oh My Zsh

### 🎯 Objectif

Remplacer `bash` par un shell plus confortable, lisible et moderne pour l’administration au quotidien.

### ⚙️ Actions

1. Installation de Zsh :

   ```bash
   sudo apt install zsh -y
   ```

2. Installation d’Oh My Zsh :

   ```bash
   sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
   ```

3. Zsh défini comme shell par défaut.

4. Plugins installés :

   ```bash
   git clone https://github.com/zsh-users/zsh-autosuggestions \
     ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions

   git clone https://github.com/zsh-users/zsh-syntax-highlighting.git \
     ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
   ```

5. Fichier `~/.zshrc` configuré avec :

   ```bash
   export ZSH="$HOME/.oh-my-zsh"
   ZSH_THEME="bira"

   plugins=(
     git
     zsh-autosuggestions
     zsh-syntax-highlighting
   )

   source $ZSH/oh-my-zsh.sh
   ```

   * options & alias utiles (ls coloré, git raccourcis, etc.).

### ✅ Résultat

* Shell plus confortable, avec complétion, couleurs et highlighting.
* Base idéale pour l’administration, les scripts et les futures commandes Ansible.

---

## 16. ⏱️🕒 Personnalisation avancée du prompt Zsh

### 🎯 Objectif

Avoir un prompt **orienté admin** qui donne les infos vraiment utiles :

* durée de la dernière commande
* statut Git (propre / modifié)
* heure
* IP locale de la VM
* séparation visuelle entre les commandes

### ⚙️ Actions principales dans `~/.zshrc`

#### 16.1. Chronométrage par commande

```bash
function preexec() {
  TIMER=$SECONDS
}

function precmd() {
  # 1) Chrono
  if [[ -n "$TIMER" ]]; then
    local elapsed=$(( SECONDS - TIMER ))
    if (( elapsed > 1 )); then
      print -P "%F{green}⏱  Temps: ${elapsed}s%f"
    fi
  fi

  # 2) Indicateur Git
  if git rev-parse --is-inside-work-tree &>/dev/null; then
    if [[ -n $(git status --porcelain 2>/dev/null) ]]; then
      GIT_PROMPT='%F{yellow}±%f '
    else
      GIT_PROMPT='%F{green}✓%f '
    fi
  else
    GIT_PROMPT=''
  fi

  # 3) Séparateur visuel
  print -P "%F{240}────────────────────────────────────%f"
}
```

#### 16.2. Heure + IP locale à droite du prompt

```bash
LOCAL_IP=$(ip -4 addr show scope global | awk '/inet /{print $2}' | cut -d/ -f1 | head -n1)
RPROMPT='%F{cyan}%*%f %F{yellow}'"$LOCAL_IP"'%f'
```

#### 16.3. Intégration de l’indicateur Git dans le prompt

```bash
PROMPT='${GIT_PROMPT}'"$PROMPT"
```

### 🔍 Comportement obtenu

* Pour une commande longue (`sleep 5`) :

  ```text
  ⏱  Temps: 5s                      17:15:32 192.168.38.129
  ✓ theo@ubuntu-desktop-lab ~ $
  ```

* Dans un repo Git :

  * `✓` vert si le dépôt est propre
  * `±` jaune si des modifications sont présentes

* Entre chaque commande :

  ```text
  ────────────────────────────────────
  ```

### ✅ Résultat

* Prompt lisible et riche en infos sans être chargé.
* Aide à diagnostiquer : lenteur, contexte Git, IP utilisée, horodatage.
* Niveau de confort digne d’un environnement **DevOps / SRE**.

---

## 🎯 17 **SYNTHÈSE FINALE — Poste de travail Administrateur (VM Ubuntu Desktop)**

À l’issue de toute la phase d’installation, configuration et personnalisation, ton poste de travail Ubuntu est devenu un **environnement professionnel complet**, stable et prêt pour la gestion d’une infrastructure entière.

---

### 🧱 17.1 **Un poste d’administrateur opérationnel**

Tu disposes maintenant d’une VM :

✔️ **Ubuntu Desktop 22.04 LTS**
→ version stable, idéale en environnement VMware

✔️ **Totalement fonctionnelle dans VMware Workstation Player**
→ réseau stable, performance correcte, snapshot manuel disponible

✔️ **Écosystème complet d’outils sysadmin**
→ `htop`, `nmap`, `dig`, `net-tools`, `traceroute`, `neofetch`, etc.

✔️ **VSCode installé en version officielle (APT)**
→ avec extensions pro : YAML, Ansible, GitLens, Docker, Markdown…

✔️ **Git configuré + connexion SSH à GitHub**
→ possibilité de cloner/pousser sans mot de passe, workflow professionnel

✔️ **KeePassXC installé**
→ gestion sécurisée et centralisée des mots de passe du lab

✔️ **Workspace structuré proprement**

```
~/workspace/scripts
~/workspace/ansible
~/workspace/tests
~/docs
```

✔️ **Zsh + Oh My Zsh avec configuration avancée**
→ autosuggestions
→ syntax highlighting
→ thème propre
→ prompt enrichi (statut Git, chrono par commande, heure, IP locale, séparateur)
→ un terminal réellement “pro” et confortable

✔️ **Snapshot manuel (`Ubuntu-desktop_backup`)**
→ sauvegarde complète du système prêt à être restauré
→ sécurité pour expérimenter, casser, tester

---

### 🧭 17.2 Un environnement productif, propre et documenté

Ce poste est maintenant :

✨ **Reproductible** :
Toutes les étapes de création sont documentées, testées et compréhensibles même par un débutant.

✨ **Compréhensible** :
Chaque erreur rencontrée a une cause connue et une solution claire (APT lock, Subiquity, dépôts VSCode…).

✨ **Organisé** :
La structure du workspace et la configuration du terminal facilitent tous les futurs travaux.

✨ **Sécurisé** :

* Gestion des mots de passe via KeePassXC
* Connexion SSH propre
* Snapshots réguliers
* Zsh optimisé mais sans gadgets risqués

---

### 🚀 17.3 Entièrement prêt pour la suite du laboratoire

Ton environnement Ubuntu Desktop est maintenant conçu pour orchestrer **toute ton infrastructure de formation** :

➡️ **Création de la VM Ubuntu Server 24.04**
(DNS, firewall, durcissement, supervision, web…)

➡️ **Installation et gestion d'Ansible**
depuis ton poste Desktop (maître → serveurs cibles)

➡️ **Mise en place du client Rocky Linux**
pour tests RHEL-like entreprise

➡️ **Automatisation, scripting, monitoring, sécurité**
depuis un poste propre, stable et optimisé

➡️ **Documentation Notion centralisée**
avec captures, erreurs, commandes, snapshots

---

# 🏁 **Conclusion — Un vrai poste d’administrateur Linux**

Ton poste réunit désormais :

✔️ stabilité

✔️ outils professionnels

✔️ configuration shell avancée

✔️ gestion Git pro

✔️ organisation claire

✔️ sécurité

✔️ reproductibilité

✔️ documentation


C’est un **socle idéal** pour monter en compétences, pratiquer de vrais scénarios d’administration, et préparer les entretiens techniques.

---