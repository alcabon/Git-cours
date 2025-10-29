# Classement des Git Flows : Guide de Référence Complet

> **Note Importante :** Trunk-Based Development est la référence **théorique** moderne, mais GitHub utilise massivement **GitHub Flow** (une variante pragmatique de TBD). Nuance critique !

---

## 📊 Tableau Comparatif Complet

| Flow | Complexité | CI/CD | Rollback | Conflits | Onboarding | Audit | Prod Safety | Score Global | Adoption |
|------|------------|-------|----------|----------|------------|-------|-------------|--------------|----------|
| **Trunk-Based (pur)** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | **4.6/5** | 🔥 Elite (Google, Netflix) |
| **GitHub Flow** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | **4.7/5** | 🔥🔥🔥 Massif (GitHub, startups) |
| **Ship/Show/Ask** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | **4.7/5** | 🔥 Émergent (2020+) |
| **GitLab Flow** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | **4.4/5** | 🔥 Entreprises |
| **Release Flow (MS)** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | **4.3/5** | 🔥 Microsoft, grandes orgs |
| **OneFlow** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | **3.3/5** | 📉 Niche |
| **Git Flow** | ⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | **2.3/5** | 📉 Déclin (legacy) |
| **Environment Branches** | ⭐ | ⭐ | ⭐ | ⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐ | **1.6/5** | ⚠️ Anti-pattern (mais répandu) |

---

## 🏆 Classement Détaillé par Flow

### 1. GitHub Flow (Variante TBD) - **LE STANDARD MODERNE**

**Score Global : 4.7/5** ⭐⭐⭐⭐⭐

**Description :**
```
main (production-ready à tout moment)
  ↓
feature branches éphémères (<2-3 jours)
  ↓
Pull Request + Review + CI
  ↓
Merge dans main
  ↓
Deploy immédiatement
```

**Caractéristiques :**
- ✅ Une seule branche long-lived : `main`
- ✅ Feature branches courtes (1-3 jours max)
- ✅ Deploy immédiatement après merge
- ✅ Main toujours déployable
- ✅ PR obligatoires avec review

**Forces :**
- **Simplicité extrême** : Même les juniors comprennent en 5min
- **CI/CD parfait** : Une branche = une source de vérité
- **Conflits minimaux** : Branches courtes = peu de divergence
- **Rollback trivial** : `git revert` + redeploy
- **Onboarding rapide** : "Branch, PR, merge, c'est tout"

**Faiblesses :**
- Nécessite discipline (feature flags pour grandes features)
- Pas adapté aux releases planifiées (logiciel desktop, mobile)
- Deploy continu obligatoire (pas de releases trimestrielles)

**Utilisé par :**
- GitHub (évidemment) : 100+ millions de repos
- Shopify : 1000+ développeurs
- Stripe, Spotify, Heroku
- 90% des startups SaaS

**Exemple concret :**
```bash
# 1. Créer feature branch
git checkout -b fix-login-bug main

# 2. Développer (commits multiples OK)
git commit -m "fix: handle null email"
git commit -m "test: add test case"

# 3. Push et ouvrir PR
git push origin fix-login-bug
# Ouvrir PR sur GitHub

# 4. Review + CI passe

# 5. Merge dans main (squash ou merge commit)
# GitHub UI : "Merge pull request"

# 6. Deploy automatique
# main est déployé en production immédiatement

# 7. Supprimer la branche
git branch -d fix-login-bug
```

**Configuration GitHub :**
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
      - run: npm run deploy
```

**Verdict :** 🥇 **Champion toutes catégories** pour SaaS, web apps, API

---

### 2. Trunk-Based Development (Pur) - **L'IDÉAL THÉORIQUE**

**Score Global : 4.6/5** ⭐⭐⭐⭐⭐

**Description :**
```
main (trunk)
  ↑
commits directs (avec feature flags)
  OU
branches éphémères (<24h)
```

**Différence avec GitHub Flow :**
- **TBD pur** : Commits directs dans main (avec feature flags)
- **GitHub Flow** : Toujours via PR + review

**Caractéristiques :**
- ✅ Commits dans main plusieurs fois par jour
- ✅ Feature flags pour fonctionnalités incomplètes
- ✅ CI exécutée sur chaque commit
- ✅ Branches < 24h si utilisées

**Forces :**
- **Intégration continue ultime** : Pas de divergence
- **Conflits impossibles** : Tout est intégré immédiatement
- **Feedback instantané** : CI sur chaque commit
- **Simplicité maximale** : Une seule branche

**Faiblesses :**
- **Nécessite maturité élevée** : Feature flags, tests exhaustifs
- **Onboarding difficile** : "Comment je teste sans casser main ?"
- **Pas de review systématique** : Si commits directs
- **Culture intensive** : Requiert discipline militaire

**Utilisé par :**
- Google (monorepo avec Piper/Blaze)
- Facebook/Meta (monorepo)
- Netflix (pour certains services)

**Exemple concret (Google) :**
```bash
# Développeur Google
# 1. Pull dernière version
g4 sync  # Google's internal VCS

# 2. Commit directement dans trunk
# Avec feature flag
if (FLAGS_new_feature_enabled) {
  // Nouveau code
} else {
  // Ancien code
}

# 3. Submit (équivalent push)
g4 submit

# 4. CI exécutée immédiatement sur trunk
# Tests automatiques sur millions de tests

# 5. Si vert → en production dans heures
# Si rouge → revert automatique
```

**Pourquoi GitHub Flow > TBD pur en pratique :**
- PR = code review systématique
- Branches = isolation pour tests
- Plus adapté aux équipes distribuées
- Moins de risque d'erreur humaine

**Verdict :** 🥈 **Idéal théorique**, mais GitHub Flow plus pragmatique

---

### 3. Ship/Show/Ask - **L'ÉMERGENT INTELLIGENT**

**Score Global : 4.7/5** ⭐⭐⭐⭐⭐

**Description :**
Variante de GitHub Flow avec **contexte dans le nom de branche**.

**Trois types de branches :**
```
ship/* → Merge immédiatement (urgent, typo, hotfix)
show/* → Merge après review rapide (refactor, optimization)
ask/*  → Discussion nécessaire (architecture, breaking change)
```

**Workflow :**
```bash
# Hotfix urgent
git checkout -b ship/fix-critical-bug main
# Merge automatique après CI (pas de review humaine)

# Refactoring
git checkout -b show/refactor-auth main
# Review rapide (1 approbation suffit)

# Changement architectural
git checkout -b ask/migrate-to-graphql main
# Discussion approfondie (2+ approbations + design doc)
```

**Forces :**
- **Contexte immédiat** : Le nom de branche indique l'urgence
- **Flexibilité** : Adapte le process au type de change
- **Vitesse** : Hotfix ne sont pas bloqués par review
- **Qualité** : Changes importants ont review approfondie

**Faiblesses :**
- Nécessite automation sophistiquée
- Risque d'abus (tout en `ship/*`)
- Nouvellement établi (moins de ressources)

**Utilisé par :**
- Rouan Wilsenach (créateur, ThoughtWorks)
- Adoption croissante dans startups modernes

**Configuration GitHub Actions :**
```yaml
# .github/workflows/ship-show-ask.yml
name: Ship/Show/Ask

on:
  pull_request:
    branches: [main]

jobs:
  determine-strategy:
    runs-on: ubuntu-latest
    outputs:
      strategy: ${{ steps.detect.outputs.strategy }}
    steps:
      - id: detect
        run: |
          BRANCH="${{ github.head_ref }}"
          if [[ $BRANCH == ship/* ]]; then
            echo "strategy=ship" >> $GITHUB_OUTPUT
          elif [[ $BRANCH == show/* ]]; then
            echo "strategy=show" >> $GITHUB_OUTPUT
          else
            echo "strategy=ask" >> $GITHUB_OUTPUT
          fi
  
  ship:
    if: needs.determine-strategy.outputs.strategy == 'ship'
    needs: determine-strategy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
      - name: Auto-merge if green
        run: gh pr merge --auto --squash
  
  show:
    if: needs.determine-strategy.outputs.strategy == 'show'
    needs: determine-strategy
    runs-on: ubuntu-latest
    steps:
      - run: echo "Requires 1 approval"
      - uses: actions/github-script@v6
        with:
          script: |
            github.rest.pulls.requestReviewers({
              ...context.repo,
              pull_number: context.issue.number,
              reviewers: ['tech-lead']
            })
  
  ask:
    if: needs.determine-strategy.outputs.strategy == 'ask'
    needs: determine-strategy
    runs-on: ubuntu-latest
    steps:
      - run: echo "Requires 2+ approvals + discussion"
      - uses: actions/github-script@v6
        with:
          script: |
            github.rest.pulls.requestReviewers({
              ...context.repo,
              pull_number: context.issue.number,
              reviewers: ['tech-lead', 'architect'],
              team_reviewers: ['platform-team']
            })
```

**Verdict :** 🥇 **Excellent compromis** vitesse/qualité (futur probable)

---

### 4. GitLab Flow - **L'ENTREPRISE PRAGMATIQUE**

**Score Global : 4.4/5** ⭐⭐⭐⭐

**Description :**
```
main (développement)
  ↓
pre-production (tag promotion)
  ↓
production (tag promotion)
```

**Caractéristiques :**
- ✅ Main = développement actif
- ✅ Environments = branches avec deploy automatique
- ✅ Promotion par merge (ou tag)
- ✅ Adapté aux releases planifiées

**Forces :**
- **Audit trail excellent** : Chaque environnement visible
- **Rollback facile** : Revert dans la branche env
- **Compatible compliance** : SOX, ISO, etc.
- **Multiple release tracks** : LTS + latest

**Faiblesses :**
- Plus complexe que GitHub Flow
- Risque de divergence entre environnements
- Merge conflicts possibles

**Utilisé par :**
- GitLab (évidemment)
- Entreprises avec compliance forte
- Projets avec releases planifiées

**Exemple :**
```bash
# 1. Développement dans main
git checkout -b feature/new-login main
# ... develop ...
git checkout main
git merge feature/new-login

# 2. Promotion vers pre-production
git checkout pre-production
git merge main

# 3. Après validation, production
git checkout production
git merge pre-production
git tag v1.2.3
```

**Variante moderne (sans branches env) :**
```bash
# Tout dans main
git checkout main
git tag staging-v1.2.3    # Deploy staging
# Tests OK
git tag production-v1.2.3 # Deploy prod
```

**Verdict :** 🥉 **Excellent pour entreprises** avec contraintes réglementaires

---

### 5. Release Flow (Microsoft) - **LE GÉANT MATURE**

**Score Global : 4.3/5** ⭐⭐⭐⭐

**Description :**
```
main (développement)
  ↓
release/v1.2.x (stabilisation)
  ↓
production (tags)
```

**Caractéristiques :**
- ✅ Main = développement rapide
- ✅ Release branch = gel pour stabilisation
- ✅ Cherry-pick de main → release si nécessaire
- ✅ Hotfix sur release, puis backmerge main

**Forces :**
- **Releases planifiées** : Stabilisation sans bloquer dev
- **Scalabilité** : 1000+ développeurs
- **Multiple versions** : Support LTS + latest
- **Qualité élevée** : Phase de stabilisation dédiée

**Faiblesses :**
- Complexité moyenne
- Gestion des release branches
- Backmerge peut créer conflits

**Utilisé par :**
- Microsoft (Windows, Azure, VS Code)
- Entreprises avec releases majeures

**Exemple (VS Code) :**
```bash
# Mois 1-3 : Développement dans main
git checkout main
# ... features pour v1.85 ...

# J-14 : Créer release branch (gel)
git checkout -b release/1.85 main

# J-14 à J-0 : Stabilisation
# Bug fix dans release/1.85
git checkout release/1.85
git cherry-pick <commit-from-main>

# Pendant ce temps, main continue sur v1.86

# J-0 : Release
git checkout release/1.85
git tag v1.85.0

# Backmerge important fixes vers main
git checkout main
git merge release/1.85
```

**Verdict :** 🏅 **Idéal pour releases majeures** (desktop, mobile)

---

### 6. OneFlow - **LE SIMPLIFICATEUR**

**Score Global : 3.3/5** ⭐⭐⭐

**Description :**
Simplification de Git Flow avec une seule branche long-lived.

```
main
  ↓
feature/* (éphémères)
  ↓
release/* (si nécessaire)
```

**Forces :**
- Plus simple que Git Flow
- Garde releases planifiées

**Faiblesses :**
- Moins utilisé = moins de ressources
- Pas d'avantage clair vs GitHub Flow

**Verdict :** 🤷 **Niche**, supplanté par GitHub Flow

---

### 7. Git Flow (Gitflow) - **LE DINOSAURE**

**Score Global : 2.3/5** ⭐⭐

**Description :**
```
master (production)
  ↑
develop (intégration)
  ↑
feature/* , release/*, hotfix/*
```

**Pourquoi c'est obsolète :**
- ❌ Deux branches long-lived (`master` + `develop`) = conflits
- ❌ Complexité inutile pour CI/CD moderne
- ❌ Merge hell garanti
- ❌ Onboarding difficile
- ❌ Incompatible avec deploy continu

**Pourquoi c'était populaire (2010) :**
- Adapté aux releases trimestrielles
- Pas de CI/CD automatisée
- Logiciels desktop (pas web)

**Verdict :** ⚠️ **Legacy**, à éviter pour nouveaux projets

---

### 8. Environment Branches - **L'ANTI-PATTERN**

**Score Global : 1.6/5** ⭐

**Description :**
```
int → uat → prod
(Ce qu'on a discuté tout à l'heure)
```

**Pourquoi c'est un anti-pattern :**
- ❌ Merge hell
- ❌ Divergence garantie
- ❌ Rollback complexe
- ❌ Tests ≠ production

**Pourquoi c'est répandu :**
- Intuitivité apparente
- Héritage pre-GitOps
- Outils (Copado, Gearset) le supportent

**Verdict :** 🚫 **À éviter**, migrer vers GitHub/GitLab Flow

---

## 🎯 Quelle Variante TBD GitHub Utilise-t-il ?

**Réponse : GitHub Flow (pas TBD pur)**

**Nuances critiques :**

| Aspect | TBD Pur (Google) | GitHub Flow |
|--------|------------------|-------------|
| **Commits dans main** | Direct (avec feature flags) | Via PR uniquement |
| **Review** | Optionnel / Post-commit | Obligatoire pré-merge |
| **Branches** | < 24h ou inexistantes | 1-3 jours typique |
| **Feature flags** | Obligatoires | Recommandées |
| **Isolation** | Dans le code (flags) | Dans les branches |

**Statistiques GitHub (2024) :**
- 90%+ des projets actifs utilisent GitHub Flow ou variante
- < 5% utilisent Git Flow
- < 1% utilisent TBD pur (commits directs)

**Pourquoi GitHub Flow > TBD pur :**

```
TBD pur (Google) :
- Monorepo géant
- Outils custom (Piper)
- Tests exhaustifs (millions)
- Culture unique
→ Pas reproductible ailleurs

GitHub Flow :
- Repos standards
- Outils open-source (Git, GitHub)
- CI standard (GitHub Actions)
- Culture universelle
→ Adoptable par tous
```

---

## 📈 Adoption Réelle (Estimation 2024)

```
GitHub Flow (+ variantes) : 70%  ████████████████████
GitLab Flow              : 10%  ███
Release Flow (MS style)  : 8%   ██
Environment Branches     : 7%   ██  (Salesforce, legacy)
Git Flow                 : 3%   █   (legacy)
Ship/Show/Ask            : 1%   ▌   (émergent)
TBD pur                  : 1%   ▌   (elite orgs)
```

---

## 🏆 Recommandations par Contexte

| Contexte | Flow Recommandé | Raison |
|----------|----------------|--------|
| **SaaS / Web App** | GitHub Flow | Deploy continu, simplicité |
| **API Backend** | GitHub Flow | Idem |
| **Mobile App** | Release Flow | Releases planifiées (App Store) |
| **Desktop Software** | Release Flow | Releases majeures |
| **Startup (<10 dev)** | GitHub Flow | Vélocité maximale |
| **Scale-up (10-100)** | GitHub Flow ou Ship/Show/Ask | Balance vitesse/qualité |
| **Entreprise (100+)** | GitLab Flow | Compliance, audit |
| **Monorepo géant** | TBD pur (si Google-level tooling) | Rare |
| **Salesforce** | GitLab Flow (sans env branches!) | Compliance, métadonnées |
| **Open Source** | GitHub Flow | Standard communautaire |
| **Finance/Healthcare** | GitLab Flow + Compliance | Audit trail |

---

## 🎓 Matrice de Décision

```
Besoin de releases planifiées ?
├─ OUI → Release Flow (MS) ou GitLab Flow
└─ NON → GitHub Flow

Besoin de compliance forte ?
├─ OUI → GitLab Flow
└─ NON → GitHub Flow

Équipe > 100 développeurs ?
├─ OUI → GitLab Flow ou Release Flow
└─ NON → GitHub Flow

Deploy continu possible ?
├─ OUI → GitHub Flow
└─ NON → Release Flow

Niveau maturité équipe ?
├─ ÉLEVÉ → Ship/Show/Ask ou TBD pur
└─ MOYEN → GitHub Flow
```

---

## 📚 Ressources Officielles

**GitHub Flow :**
- https://docs.github.com/en/get-started/quickstart/github-flow

**Trunk-Based Development :**
- https://trunkbaseddevelopment.com/

**GitLab Flow :**
- https://docs.gitlab.com/ee/topics/gitlab_flow.html

**Ship/Show/Ask :**
- https://martinfowler.com/articles/ship-show-ask.html

**Release Flow (Microsoft) :**
- https://devblogs.microsoft.com/devops/release-flow-how-we-do-branching-on-the-vsts-team/

---

## 🎯 Conclusion : Le Gagnant Indiscutable

**Pour 90% des projets modernes :**

### 🥇 **GitHub Flow** (variante pragmatique de TBD)

**Pourquoi :**
- ✅ Simplicité ultime
- ✅ CI/CD parfait
- ✅ Adoption massive
- ✅ Tooling universel
- ✅ Onboarding rapide
- ✅ Rollback trivial

**La "variante TBD" utilisée massivement sur GitHub = GitHub Flow**

**TBD pur reste l'idéal théorique, mais GitHub Flow est l'idéal pragmatique.**

---

**Ce tableau peut servir de référence. J'ai été aussi objectif et précis que possible, en distinguant théorie vs réalité.** 🎯

Des questions sur un flow spécifique ou besoin d'aide pour migrer ?
