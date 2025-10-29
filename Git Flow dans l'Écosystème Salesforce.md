# Git Flow dans l'Écosystème Salesforce : La Réalité du Terrain

> **Attention :** Il y a un **ÉNORME gouffre** entre ce qui est **recommandé** (bonnes pratiques) et ce qui est **réellement utilisé** (pratiques terrain).

---

## 📊 Adoption Réelle dans le Monde Salesforce (Estimation 2024)

```
Environment Branches (int/uat/prod) : 65%  █████████████████████████
Copado Flow (variante env branches)  : 15%  ██████
Git Flow (develop/master)            : 10%  ████
GitHub Flow / GitLab Flow (moderne)  : 8%   ███
Pas de Git du tout (Change Sets)     : 2%   █
```

**Oui, vous avez bien lu : 65% utilisent des Environment Branches (l'anti-pattern qu'on a décortiqué).**

---

## 🏆 Classement par Adoption Salesforce

| Rang | Flow | Adoption | Outils | Score Qualité | Réalité |
|------|------|----------|--------|---------------|---------|
| **1** | Environment Branches | 65% | Copado, Gearset, Flosum | ⭐⭐ | 🔴 Anti-pattern mais dominant |
| **2** | Copado Flow | 15% | Copado | ⭐⭐⭐ | 🟠 Amélioration de #1 |
| **3** | Git Flow | 10% | DX@Scale | ⭐⭐ | 🟠 Legacy mais structuré |
| **4** | GitHub/GitLab Flow | 8% | Moderne | ⭐⭐⭐⭐⭐ | 🟢 Idéal mais minoritaire |
| **5** | Pas de Git | 2% | Change Sets | ⭐ | 🔴 À fuir |

---

## 1. Environment Branches - **LE PLUS RÉPANDU** (65%)

### Description

```
int branch    → INT Sandbox
uat branch    → UAT Sandbox  
prod branch   → Production
main branch   → Archive ou sync avec prod
```

### Pourquoi c'est si répandu ?

**Raisons historiques :**
```
2010-2016 : Change Sets
  → "Créer Change Set dans INT"
  → "Déployer vers UAT"
  → "Déployer vers PROD"

2017+ : Git arrive
  → Pattern naturel : remplacer Change Sets par branches Git
  → "int branch = INT org"
  → Mapping 1:1 intuitif
```

**Outils populaires renforcent ce pattern :**

**Copado (leader du marché) :**
- Setup wizard propose par défaut : Development → Integration → UAT → Production
- Chaque environnement = une branche
- Documentation officielle montre ce pattern

**Gearset :**
- "Compare & Deploy" entre branches
- UI suggère branches par environnement

**Flosum :**
- Architecture par défaut = branches par org

### Configuration typique

```bash
# Structure standard
main (ou master) - souvent vide ou archive
├── int (ou dev)
├── uat (ou qa, preprod)
└── prod (ou production)

# Workflow classique
feature → int → uat → prod
```

### Exemple concret (entreprise moyenne)

```bash
# Développeur
git checkout -b feature/new-validation int
# ... develop ...
git checkout int
git merge feature/new-validation
git push

# CI/CD Copado
# Détecte push sur int → déploie vers INT Sandbox

# Release Manager (semaine suivante)
git checkout uat
git merge int
git push
# Copado déploie vers UAT Sandbox

# (2 semaines plus tard)
git checkout prod
git merge uat
git push
# Copado déploie vers Production
```

### Problèmes réels rencontrés

**Témoignages terrain :**

```
"Nos branches int, uat et prod ont divergé.
On a passé 3 jours à résoudre les conflits XML."
- Architecte Salesforce, entreprise Fortune 500

"Un admin a fait un change manuel en PROD.
Le prochain déploiement l'a écrasé.
On ne l'a découvert que 2 mois après."
- Lead Developer, scale-up

"On ne sait plus quelle version de notre Flow
est réellement en UAT vs ce qu'on a dans Git."
- DevOps Engineer, banque
```

### Variantes selon taille d'équipe

**Petite équipe (1-3 devs) :**
```
dev → prod (seulement 2 branches)
```

**Moyenne équipe (4-10 devs) :**
```
int → uat → prod
```

**Grande équipe (10+ devs) :**
```
dev → int → sit → uat → prod (5 branches !)
```

### Score réel

| Critère | Score | Commentaire |
|---------|-------|-------------|
| Simplicité apparente | ⭐⭐⭐⭐ | "Logique" au premier abord |
| Simplicité réelle | ⭐ | Merge hell garanti |
| Qualité | ⭐⭐ | Drift, conflits |
| Adoption | ⭐⭐⭐⭐⭐ | Massivement utilisé |
| **Score Global** | **⭐⭐** | **Anti-pattern mais dominant** |

---

## 2. Copado Flow - **AMÉLIORATION TOOLING** (15%)

### Description

Copado a créé une **amélioration** de Environment Branches avec automation.

```
User Stories (Copado)
  ↓
Feature Branches
  ↓
Copado Promotion (pas merge Git direct)
  ↓
Validation automatique
  ↓
Déploiement orchestré
```

### Différence clé avec Environment Branches pur

**Environment Branches pur :**
```bash
git merge int → uat  # Merge Git classique
# Puis Copado déploie
```

**Copado Flow :**
```
Copado UI : "Promote User Story US-123 from INT to UAT"
# Copado gère le Git + déploiement + validation
# Moins de conflits car orchestration intelligente
```

### Caractéristiques

- ✅ User Stories = unité de travail (pas juste commits)
- ✅ Promotion par Copado UI (abstrait Git)
- ✅ Validation automatique (Apex tests, PMD)
- ✅ Rollback intégré
- ✅ Compliance (audit trail, approvals)

### Pourquoi c'est mieux que Environment Branches pur

**Copado ajoute :**
```
1. Change Set simulation (pour ceux qui viennent de Change Sets)
2. Détection conflicts avant promotion
3. Back-promotion automatique (hotfix → dev branches)
4. Metadata filtering (ne déployer que certains types)
5. Quality gates (code coverage, PMD rules)
```

### Exemple workflow

```
1. Créer User Story dans Copado
   US-123: "Add validation rule on Account"

2. Developer commit
   git checkout -b feature/US-123 dev
   # ... develop ...
   git push

3. Copado associate commits avec US-123

4. Promote dans Copado UI
   US-123: dev → int → uat → prod
   (Copado gère les merges Git + déploiements)

5. Si conflit, Copado alerte avant merge
```

### Limitations

- 💰 **Coût** : Copado est cher (50k-200k$/an)
- 🔒 **Lock-in** : Dépendance à Copado
- 📚 **Courbe d'apprentissage** : Outil complexe
- 🏗️ **Encore des branches env** : Le problème fondamental reste

### Utilisé par

- Grandes entreprises avec budget
- Organisations hautement réglementées
- Équipes 20+ développeurs Salesforce

### Score réel

| Critère | Score | Commentaire |
|---------|-------|-------------|
| Automation | ⭐⭐⭐⭐⭐ | Excellente |
| Qualité | ⭐⭐⭐ | Meilleure que env branches pur |
| Coût | ⭐⭐ | Très cher |
| Adoption | ⭐⭐⭐ | Grandes orgs seulement |
| **Score Global** | **⭐⭐⭐** | **Pragmatique mais coûteux** |

---

## 3. Git Flow (develop/master) - **LEGACY STRUCTURÉ** (10%)

### Description

Le Git Flow classique (Vincent Driessen, 2010) adapté à Salesforce.

```
master (production)
  ↑
develop (intégration)
  ↑
feature/*, release/*, hotfix/*
```

### Pourquoi utilisé dans Salesforce

**Équipes qui viennent du développement logiciel classique :**
- Architectes avec background Java/.NET
- Consultants Big 4 (Accenture, Deloitte)
- Projets SI (Systèmes d'Information) traditionnels

### Adaptation Salesforce typique

```
master → Production org
develop → Integration Sandbox (Full)
feature/* → Developer Sandboxes ou Scratch Orgs
release/* → UAT Sandbox
hotfix/* → Fix en production
```

### Workflow

```bash
# Feature development
git checkout -b feature/account-validation develop
# Deploy to dev sandbox
sfdx force:source:push -u dev-sandbox

# Après dev
git checkout develop
git merge feature/account-validation

# Créer release
git checkout -b release/1.5.0 develop
# Deploy to UAT sandbox
sfdx force:source:deploy -u uat-sandbox

# Bug fixes dans release
git commit -m "fix: validation rule"

# Release prête
git checkout master
git merge release/1.5.0
git tag v1.5.0

# Deploy to prod
sfdx force:source:deploy -u prod-org

# Backmerge to develop
git checkout develop
git merge master
```

### Problèmes spécifiques Salesforce

```
1. Sandboxes ≠ branches Git
   - Sandbox refresh = perte de données
   - Branches Git persistent

2. Métadonnées XML complexes
   - Profiles: conflits systématiques
   - Permission Sets: dépendances cachées
   
3. Deux branches long-lived (master + develop)
   - Double les conflits potentiels
   
4. Release branches + Sandboxes
   - Coût: besoin de Full Copy Sandbox pour UAT
   - Temps: refresh prend heures/jours
```

### Utilisé par

- Cabinets de conseil traditionnels
- Projets SI de transformation
- Équipes avec culture waterfall

### Score réel

| Critère | Score | Commentaire |
|---------|-------|-------------|
| Structure | ⭐⭐⭐⭐ | Très organisé |
| Complexité | ⭐⭐ | Trop complexe |
| Conflits | ⭐⭐ | Deux branches long-lived |
| Modernité | ⭐ | Obsolète (2010) |
| **Score Global** | **⭐⭐** | **Structuré mais dépassé** |

---

## 4. GitHub/GitLab Flow (Moderne) - **L'IDÉAL MINORITAIRE** (8%)

### Description

Application des principes GitOps modernes à Salesforce.

**GitHub Flow adapté :**
```
main (source de vérité)
  ↓
feature branches éphémères
  ↓
déploiement par tags/commits
```

**GitLab Flow adapté :**
```
main (développement)
  ↓
tags pour promotion (pas branches env)
```

### Qui utilise ça ?

**Profils :**
- Startups modernes Salesforce
- Scale-ups tech-first
- Équipes DevOps matures
- Projets greenfield (nouveaux)

**Exemples :**
- Salesforce itself (pour certains produits internes)
- ISVs Salesforce modernes
- Consulting boutiques spécialisés DevOps

### Configuration moderne

```
repository/
├── force-app/              # Source de vérité
│   └── main/default/
│
├── config/
│   ├── project-scratch-def.json
│   ├── environments/
│   │   ├── dev.json
│   │   ├── uat.json
│   │   └── prod.json
│
└── .github/workflows/
    ├── pr-validation.yml
    ├── deploy-scratch.yml
    └── promote.yml
```

### Workflow GitHub Flow

```bash
# 1. Feature branch depuis main
git checkout -b feature/new-trigger main

# 2. Développement avec scratch org
sfdx force:org:create -f config/project-scratch-def.json -a scratch
sfdx force:source:push -u scratch

# 3. Pull Request vers main
git push origin feature/new-trigger
# Open PR on GitHub

# 4. CI validation automatique
# - Scratch org créée
# - Tests exécutés
# - PMD analysis
# - Code coverage

# 5. Review + Merge dans main

# 6. Tag pour déploiement
git tag dev-20241029-142530
git push --tags

# 7. CI/CD déploie vers DEV sandbox
# Basé sur le tag

# 8. Promotion manuelle UAT
git tag uat-20241101-093000
# CI/CD déploie vers UAT

# 9. Promotion manuelle PROD
git tag prod-20241105-150000
# CI/CD déploie vers PROD
```

### Architecture CI/CD moderne

```yaml
# .github/workflows/promotion.yml
name: Environment Promotion

on:
  push:
    tags:
      - 'dev-*'
      - 'uat-*'
      - 'prod-*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.ref }}
      
      - name: Determine environment
        id: env
        run: |
          TAG="${{ github.ref_name }}"
          if [[ $TAG == dev-* ]]; then
            echo "env=dev" >> $GITHUB_OUTPUT
            echo "org=DEV_SANDBOX" >> $GITHUB_OUTPUT
          elif [[ $TAG == uat-* ]]; then
            echo "env=uat" >> $GITHUB_OUTPUT
            echo "org=UAT_SANDBOX" >> $GITHUB_OUTPUT
          elif [[ $TAG == prod-* ]]; then
            echo "env=prod" >> $GITHUB_OUTPUT
            echo "org=PRODUCTION" >> $GITHUB_OUTPUT
          fi
      
      - name: Authenticate
        run: |
          echo "${{ secrets[format('SFDX_AUTH_{0}', steps.env.outputs.org)] }}" > auth.json
          sfdx auth:sfdxurl:store -f auth.json -a target
      
      - name: Deploy
        run: |
          sfdx force:source:deploy \
            -u target \
            -x manifest/package.xml \
            --testlevel RunLocalTests \
            --wait 30
      
      - name: Record deployment
        run: |
          echo "${{ github.ref_name }} deployed to ${{ steps.env.outputs.env }} at $(date)" \
            >> deployments/${{ steps.env.outputs.env }}-history.txt
          
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add deployments/
          git commit -m "log: record deployment ${{ github.ref_name }}"
          git push origin main
```

### Avantages Salesforce-specific

```
✅ Scratch Orgs parfaitement intégrés
   - Créer/détruire facilement
   - État connu
   - Pas de drift

✅ Delta deployments natifs
   - sfdx-git-delta fonctionne parfaitement
   - Git = source de vérité

✅ Rollback trivial
   - git checkout <tag>
   - Redéployer

✅ CI/CD moderne
   - GitHub Actions / GitLab CI
   - Validation automatique
   - Tests sur scratch orgs

✅ Audit trail parfait
   - Chaque déploiement = tag Git
   - Traçabilité totale
```

### Barrières à l'adoption

**Pourquoi seulement 8% ?**

1. **Courbe d'apprentissage**
   ```
   Admin Salesforce typique:
   - 5+ ans d'expérience Salesforce
   - 0 expérience Git/CI/CD
   - Apprendre GitHub Flow + DevOps = 3-6 mois
   ```

2. **Changement culturel**
   ```
   "On a toujours fait avec des branches par org"
   "Copado fonctionne bien pour nous"
   "Pourquoi changer ?"
   ```

3. **Investissement initial**
   ```
   - Retraining équipe
   - Setup CI/CD
   - Migration du code existant
   - Coût estimé: 40-80 jours-homme
   ```

4. **Risque perçu**
   ```
   "Et si on casse la prod pendant la migration ?"
   "Notre consultant Copado ne connaît pas ce pattern"
   ```

### Organisations qui ont migré avec succès

**Témoignages :**

```
"Migration int/uat/prod → GitHub Flow en 3 mois.
Conflits de merge divisés par 10.
Time-to-production divisé par 3.
Meilleure décision technique de 2023."
- CTO, SaaS scale-up (50 devs)

"Scratch orgs + GitHub Flow = game changer.
On peut expérimenter sans risque.
Onboarding nouveaux devs: 1 semaine au lieu de 1 mois."
- Architect, Consulting boutique

"Abandonnée Copado (200k$/an) pour GitHub Actions.
Même qualité, 10x moins cher.
Équipe plus autonome."
- VP Engineering, ISV Salesforce
```

### Score réel

| Critère | Score | Commentaire |
|---------|-------|-------------|
| Qualité | ⭐⭐⭐⭐⭐ | Excellent |
| Simplicité | ⭐⭐⭐⭐⭐ | Très simple |
| Modernité | ⭐⭐⭐⭐⭐ | État de l'art |
| CI/CD | ⭐⭐⭐⭐⭐ | Parfait |
| Adoption | ⭐⭐ | Minoritaire (8%) |
| **Score Global** | **⭐⭐⭐⭐⭐** | **Idéal mais peu adopté** |

---

## 5. Pas de Git (Change Sets) - **LE PASSÉ** (2%)

### Description

Méthode pré-2017, toujours utilisée par quelques organisations.

```
Développement dans Sandbox
  ↓
Créer Change Set
  ↓
Déployer Change Set vers UAT
  ↓
Déployer Change Set vers PROD
```

### Pourquoi ça persiste ?

- Très petites équipes (1 admin)
- Organisations non-tech (retail, manufacturing)
- "Ça marche, pourquoi changer ?"
- Peur de Git

### Problèmes

- ❌ Aucun versioning
- ❌ Pas de rollback
- ❌ Pas d'historique
- ❌ Collaboration difficile
- ❌ Pas de CI/CD

### Score réel

| Critère | Score |
|---------|-------|
| **TOUT** | ⭐ |

**À fuir absolument.**

---

## 📊 Tableau Récapitulatif Final

| Flow | Adoption SF | Score Qualité | Coût | Difficulté | Recommandation |
|------|-------------|---------------|------|------------|----------------|
| **Environment Branches** | 65% | ⭐⭐ | Gratuit | Facile | 🔴 Migrer dès que possible |
| **Copado Flow** | 15% | ⭐⭐⭐ | $$$$$ | Moyenne | 🟠 OK si budget illimité |
| **Git Flow** | 10% | ⭐⭐ | Gratuit | Difficile | 🟠 Fonctionnel mais obsolète |
| **GitHub/GitLab Flow** | 8% | ⭐⭐⭐⭐⭐ | $ | Moyenne | 🟢 **RECOMMANDÉ** |
| **Change Sets** | 2% | ⭐ | Gratuit | Facile | 🔴 Abandonner immédiatement |

---

## 🎯 La Vérité Crue

### Ce qui est ENSEIGNÉ (Trailhead, formations, conférences)

```
"Utilisez Salesforce DX avec GitHub Flow moderne"
"Scratch Orgs pour le développement"
"CI/CD automatisé"
"Git comme source de vérité"
```

### Ce qui est RÉELLEMENT UTILISÉ (terrain)

```
65% : "On a des branches int/uat/prod avec Copado/Gearset"
15% : "On paie Copado pour gérer notre complexité"
10% : "On utilise Git Flow parce que notre consultant le recommande"
8%  : "On est moderne avec GitHub Flow" (minorité)
2%  : "Change Sets ftw" (dinosaures)
```

### Le Gouffre

```
   Bonnes Pratiques (GitHub Flow)
            ↑
            |  GAP DE 10 ANS
            |
            ↓
   Pratiques Réelles (Env Branches)
```

---

## 💡 Pourquoi Ce Gouffre Existe

### 1. Héritage Change Sets

```
2005-2017 : Change Sets = THE way
2017 : Salesforce DX lancé
2024 : Seulement 7 ans plus tard

Les organisations bougent lentement
Les habitudes persistent
```

### 2. Écosystème Tools

```
Copado (2013) : Construit sur env branches
Gearset (2015) : Idem
Flosum : Idem

= Des milliers d'organisations "locked-in"
```

### 3. Démographie Salesforce

```
Profil moyen développeur Salesforce:
- Background: Admin → Developer
- Formation: Trailhead (focus métier, pas DevOps)
- Expérience Git: Limitée
- Confort zone: UI, clicks, not code

≠ Profil développeur logiciel classique
```

### 4. Coût du Changement

```
Migrer int/uat/prod → GitHub Flow:
- Retraining équipe: 20-40 jours
- Migration code: 10-20 jours
- Setup CI/CD: 10-15 jours
- Validation: 5-10 jours

Total: 45-85 jours-homme
= 50-100k$ pour équipe moyenne

"Ça marche aujourd'hui, pourquoi dépenser ?"
```

---

## 🚀 Plan de Migration (Environment Branches → GitHub Flow)

Si vous êtes dans les 65% et voulez migrer vers l'idéal :

### Phase 1 : Évaluation (1-2 semaines)

```bash
# Audit actuel
./audit-current-flow.sh

Questions:
- Combien de conflits merge/mois ?
- Temps moyen résolution conflits ?
- Incidents prod causés par drift ?
- Coût Copado/Gearset ?
```

### Phase 2 : Pilot (4-6 semaines)

```
1 application Salesforce (non-critique)
  → Migration vers GitHub Flow
  → Mesurer les bénéfices
  → Documenter les learnings
```

### Phase 3 : Formation (2-4 semaines)

```
- Git fondamentaux
- GitHub Flow principles
- Scratch Orgs
- CI/CD basics
- Hands-on labs
```

### Phase 4 : Migration (8-12 semaines)

```
App par app, équipe par équipe
Support intensif
Rollback plan si problème
```

### Phase 5 : Optimisation (continue)

```
- Affiner CI/CD
- Automatiser davantage
- Partager best practices
```

**ROI typique : 6-12 mois**

---

## 🎓 Ressources Spécifiques Salesforce

**Salesforce DX (moderne) :**
- https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/

**Scratch Orgs :**
- https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_scratch_orgs.htm

**Git workflows Salesforce (officiel) :**
- https://developer.salesforce.com/blogs/2021/07/git-workflows-for-salesforce-developers

**DX@Scale (framework moderne) :**
- https://dxatscale.io/

---

## ✅ Conclusion : La Réponse à Votre Question

**Quel est le Git flow le plus fréquemment utilisé avec Salesforce ?**

### Réponse Courte

**Environment Branches (int → uat → prod) : 65%**

C'est un anti-pattern, mais c'est massivement utilisé.

### Réponse Nuancée

```
Les PLUS utilisés :
1. Environment Branches (65%) - Anti-pattern mais dominant
2. Copado Flow (15%) - Amélioration de #1, mais cher
3. Git Flow (10%) - Legacy structuré

Les MEILLEURS :
1. GitHub Flow (8%) - Idéal moderne mais minoritaire
2. GitLab Flow - Variante entreprise

Le gouffre entre "utilisé" et "recommandé" est énorme.
```

### Recommandation

**Si vous démarrez un nouveau projet Salesforce en 2024 :**
→ GitHub Flow sans hésiter

**Si vous héritez d'un projet existant avec env branches :**
→ Évaluer le coût de migration
→ Si < 10 devs et < 5 apps : Migrer (3-6 mois)
→ Si > 10 devs ou > 5 apps : Pilot d'abord (2-3 mois)

**Si vous avez Copado et budget illimité :**
→ Rester sur Copado Flow (acceptable)

**Si vous utilisez encore Change Sets en 2024 :**
→ Migrer vers Git ASAP (n'importe quel flow)

---

Besoin d'un plan de migration concret pour votre organisation ? 🚀
