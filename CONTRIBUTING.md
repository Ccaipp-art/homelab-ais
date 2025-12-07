# CONTRIBUTING – Homelab AIS

Ce projet documente la construction d’un laboratoire professionnel orienté
“Administration Infrastructure Sécurisée”.

Les contributions suivent un workflow simple mais structuré, similaire à celui
utilisé en entreprise.

---

## 🔀 1. Branches

- `main` : branche stable et relisible.
- Branches de travail recommandées :
  - `doc-XX-*`   → documentation
  - `feat-*`     → nouvelles fonctionnalités
  - `fix-*`      → corrections
  - `chore-*`    → maintenance interne
  - `refactor-*` → réorganisation ou nettoyage

Exemples :
- `doc-05-bind9-configuration`
- `feat-06-nginx-install`
- `fix-dns-options-listen-ip`
- `chore-templates-github`

---

## 📝 2. Conventions de commit

Format :
<type>: <description courte>


Types :

- `doc:` documentation uniquement  
- `feat:` nouvelle fonctionnalité  
- `fix:` correction de bug  
- `chore:` maintenance interne  
- `refactor:` réorganisation sans fonctionnalité nouvelle  

Exemples :
doc: ajouter la configuration Bind9
feat: installer Nginx avec virtual host
fix: corriger la zone DNS reverse
chore: ajouter template de Pull Request
refactor: simplifier named.conf.options


---

## 🔧 3. Workflow recommandé

1. Créer une branche spécifique :

'''
git checkout -b doc-06-nginx-installation
'''


2. Travailler + commits propres :

'''
git add .
git commit -m "doc: rédiger la documentation Nginx"
'''


3. Pousser :

'''
git push -u origin doc-06-nginx-installation
'''


4. Ouvrir une Pull Request sur GitHub  
→ utiliser le template `.github/pull_request_template.md`

5. Merge uniquement quand :
- tests OK  
- documentation relue  
- PR validée  

---

## 🧰 4. Bonnes pratiques

- Toutes les IP dans les docs publiques doivent être **anonymisées**.  
- Les erreurs rencontrées doivent être documentées.  
- Toujours structurer les docs : Objectif → Prérequis → Actions → Tests → Erreurs.  
- Garder un lab cohérent entre Desktop (poste d'admin) et Serveur (infra).  

---

Fin du fichier.


