# Stratégies de fusion Git : Guide expert

Voici une explication détaillée des trois stratégies principales pour intégrer des branches dans Git :

## 1. **Merge Commit** (Fusion classique)

**Comment ça fonctionne :**
```
main:     A---B-------E
               \     /
feature:        C---D
```
Crée un commit de fusion (E) qui a deux parents, préservant tout l'historique.

**Commande :**
```bash
git checkout main
git merge feature
```

**✅ Avantages :**
- Historique complet et traçable
- Préserve le contexte de développement de la branche
- Non-destructif (historique original intact)
- Facilite les reverts de fonctionnalités entières

**❌ Inconvénients :**
- Historique complexe avec beaucoup de branches
- Graph difficile à lire avec de nombreuses fusions
- Commits de merge "pollution visuelle"

**Quand l'utiliser :** Pour des features importantes, travail collaboratif, ou quand la traçabilité est cruciale.

---

## 2. **Squash and Merge**

**Comment ça fonctionne :**
```
main:     A---B---E (contient C+D combinés)
               \
feature:        C---D (branche supprimée)
```
Tous les commits de la branche sont compactés en un seul commit propre.

**Commande :**
```bash
git checkout main
git merge --squash feature
git commit -m "Feature: description complète"
```

**✅ Avantages :**
- Historique linéaire et propre sur main
- Un commit = une feature (logique claire)
- Idéal pour les petites PR avec beaucoup de commits de travail
- Facilite la lecture du changelog

**❌ Inconvénients :**
- Perte de l'historique détaillé des commits intermédiaires
- Difficile de revert des parties spécifiques
- Perd l'information sur les auteurs individuels des commits

**Quand l'utiliser :** Pour des PR avec commits de "work in progress", fixups, ou quand un historique linéaire est prioritaire.

---

## 3. **Rebase and Merge**

**Comment ça fonctionne :**
```
Avant:
main:     A---B-------E
               \     /
feature:        C---D

Après:
main:     A---B---C'---D'---E
```
Les commits sont réécrits (C', D') et placés linéairement sur main.

**Commande :**
```bash
git checkout feature
git rebase main
# Résoudre les conflits si nécessaire
git checkout main
git merge feature  # Fast-forward
```

**✅ Avantages :**
- Historique complètement linéaire (pas de branches)
- Préserve les commits individuels
- Log très lisible (`git log --oneline`)
- Chaque commit est directement sur main

**❌ Inconvénients :**
- **Réécrit l'historique** (SHA modifiés)
- Peut créer des conflits multiples pendant le rebase
- Dangereux sur des branches partagées publiquement
- Perd la notion de "quand la feature a été intégrée"

**Quand l'utiliser :** Pour des branches personnelles, petites équipes, ou quand un historique ultra-propre est requis.

---

## 📊 Tableau comparatif

| Critère | Merge Commit | Squash | Rebase |
|---------|-------------|---------|---------|
| **Historique** | Complet | Simplifié | Linéaire |
| **Traçabilité** | ⭐⭐⭐ | ⭐ | ⭐⭐ |
| **Lisibilité** | ⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Sécurité** | ✅ Safe | ✅ Safe | ⚠️ Réécrit |
| **Bisect** | Complexe | Facile | Facile |

## 🎯 Recommandations selon le contexte

**Projets open-source :** Merge commit (traçabilité importante)  
**Équipes agiles/sprints :** Squash (un commit par feature/ticket)  
**Projets petite équipe :** Rebase (historique propre)  
**Hotfixes :** Merge commit ou Squash  
**Main/production :** Éviter le rebase direct

**Règle d'or :** Ne jamais rebase des branches publiques/partagées ! 

Besoin d'exemples pratiques ou de cas d'usage spécifiques ?

---

# Git Bisect : Trouvez le bug comme un pro 🔍

## 📖 Qu'est-ce que `git bisect` ?

**Bisect** est un outil Git qui utilise la **recherche dichotomique** (binary search) pour identifier le commit exact qui a introduit un bug. Au lieu de tester manuellement chaque commit, bisect divise l'historique en deux à chaque étape.

**Complexité :** O(log n) au lieu de O(n)
- 1000 commits → ~10 tests maximum
- 10000 commits → ~14 tests maximum

---

## 🎯 Concept de base

Imaginez que vous avez 100 commits et un bug :

```
v1.0 (✅ bon)  →  ... 98 commits ...  →  HEAD (❌ bug)
```

Au lieu de tester les 100 commits, bisect fait :
1. Test du commit 50 : bug présent ? → Chercher dans 1-50
2. Test du commit 25 : bug présent ? → Chercher dans 25-50
3. Test du commit 37 : bug absent ? → Chercher dans 37-50
4. etc.

---

## 💻 Exemple pratique complet

### Scénario
Vous développez une application. La version `v1.0` (commit `abc123`) fonctionne parfaitement. Aujourd'hui (commit `xyz789`), un test unitaire échoue. Il y a 50 commits entre les deux.

### Étape 1 : Démarrer bisect

```bash
git bisect start
```

### Étape 2 : Marquer le commit actuel comme "mauvais"

```bash
git bisect bad
# ou
git bisect bad HEAD
# ou un commit spécifique
git bisect bad xyz789
```

### Étape 3 : Marquer un ancien commit "bon"

```bash
git bisect good v1.0
# ou
git bisect good abc123
```

**Résultat :**
```
Bisecting: 25 revisions left to test after this (roughly 5 steps)
[commit m45n67] Commit du milieu
```

Git vous place automatiquement sur le commit du milieu !

### Étape 4 : Tester et marquer

Testez votre application :

```bash
# Exécutez vos tests
npm test
# ou
python -m pytest
# ou testez manuellement
```

**Si le bug est présent :**
```bash
git bisect bad
```

**Si le bug est absent :**
```bash
git bisect good
```

Git vous déplace vers le prochain commit à tester :
```
Bisecting: 12 revisions left to test after this (roughly 4 steps)
[commit p89q01] Autre commit
```

### Étape 5 : Répéter jusqu'à trouver le coupable

Continuez à marquer `good` ou `bad` jusqu'à ce que Git trouve :

```
abc123def456 is the first bad commit
commit abc123def456
Author: John Doe <john@example.com>
Date:   Mon Oct 20 14:30:00 2025

    Fix: Optimize database query

:100644 100644 a1b2c3 d4e5f6 M    src/database.js
```

**🎉 Trouvé !** Le commit `abc123def456` a introduit le bug.

### Étape 6 : Terminer bisect

```bash
git bisect reset
```

Retour à votre branche initiale.

---

## 🤖 Bisect automatique (mode avancé)

Si vous avez un script de test automatique, bisect peut tout faire seul !

```bash
git bisect start HEAD v1.0
git bisect run npm test
```

**Git va :**
1. Exécuter `npm test` sur chaque commit
2. Marquer automatiquement `good` (exit code 0) ou `bad` (exit code non-0)
3. Trouver le commit fautif sans intervention

### Exemple avec un script personnalisé

```bash
# test-bug.sh
#!/bin/bash
npm install > /dev/null 2>&1
npm test | grep "should calculate total"
```

```bash
chmod +x test-bug.sh
git bisect start HEAD v1.0
git bisect run ./test-bug.sh
```

---

## 📝 Exemple complet de session

```bash
$ git bisect start
$ git bisect bad                    # Commit actuel est mauvais
$ git bisect good v2.0              # v2.0 était bon

Bisecting: 15 revisions left to test after this (roughly 4 steps)
[a1b2c3d] Add user authentication

$ npm test
✗ Test failed

$ git bisect bad

Bisecting: 7 revisions left to test after this (roughly 3 steps)
[e4f5g6h] Update dependencies

$ npm test
✓ All tests passed

$ git bisect good

Bisecting: 3 revisions left to test after this (roughly 2 steps)
[i7j8k9l] Refactor payment module

$ npm test
✗ Test failed

$ git bisect bad

Bisecting: 1 revision left to test after this (roughly 1 step)
[m0n1o2p] Fix typo in config

$ npm test
✓ All tests passed

$ git bisect good

i7j8k9l is the first bad commit
commit i7j8k9l
Author: Jane Smith <jane@example.com>
Date:   Wed Oct 22 10:15:00 2025

    Refactor payment module

$ git bisect reset
```

---

## 🛠️ Commandes utiles

| Commande | Description |
|----------|-------------|
| `git bisect start` | Démarre une session bisect |
| `git bisect bad [commit]` | Marque un commit comme mauvais |
| `git bisect good [commit]` | Marque un commit comme bon |
| `git bisect skip` | Saute un commit (ne compile pas, etc.) |
| `git bisect reset` | Termine et retourne à HEAD |
| `git bisect log` | Affiche l'historique de la session |
| `git bisect replay <file>` | Rejoue une session depuis un fichier |
| `git bisect visualize` | Affiche graphiquement les commits restants |
| `git bisect run <script>` | Automatise avec un script |

---

## 🎭 Cas d'usage réels

### 1. Bug de régression
```bash
# Le test passait hier, échoue aujourd'hui
git bisect start
git bisect bad HEAD
git bisect good HEAD~10
git bisect run pytest tests/test_payment.py
```

### 2. Performance dégradée
```bash
# Script qui mesure le temps d'exécution
# benchmark.sh renvoie 0 si < 2s, 1 si > 2s
git bisect start HEAD v3.0
git bisect run ./benchmark.sh
```

### 3. Sauter des commits problématiques
```bash
git bisect start
git bisect bad
git bisect good v1.0
# Ce commit ne compile pas
git bisect skip
```

---

## ⚡ Pourquoi bisect est puissant avec différentes stratégies de merge

**Avec Squash :** 
- ✅ Très efficace (peu de commits à tester)
- ❌ Trouve seulement la feature complète, pas le commit précis

**Avec Rebase :**
- ✅ Historique linéaire = bisect plus rapide
- ✅ Trouve le commit exact dans la feature

**Avec Merge Commit :**
- ⚠️ Peut tester des commits de merge (moins utile)
- ✅ Mais trouve quand même le commit problématique

---

## 🎓 Conseil pro

Utilisez des **commits atomiques** (un changement = un commit) pour maximiser l'efficacité de bisect. Chaque commit devrait :
- Compiler sans erreur
- Passer les tests de base
- Représenter un changement logique unique

Besoin d'aide pour automatiser bisect dans votre projet.

----

# Exactement ! 🎯

Vous avez parfaitement compris. Voici une illustration concrète :

---

## 📊 Comparaison visuelle

### Avant le merge (dans la branche feature)

```
feature branch:
  C1: "Add user model"
  C2: "Fix typo in model"
  C3: "Add validation logic"
  C4: "Fix validation bug" ← Le bug est introduit ICI
  C5: "Add tests"
  C6: "Update documentation"
```

---

### Après SQUASH merge sur main

```
main:
  A---B---S
          ↑
          Un seul commit contenant C1+C2+C3+C4+C5+C6
          Message: "Feature: Add user validation"
```

**Les commits C1 à C6 n'existent plus dans l'historique de `main` !**

Quand vous faites `git bisect` :
```bash
git bisect start
git bisect bad HEAD
git bisect good B

# Git trouve:
S is the first bad commit  ← Tout le commit squashé
```

**Résultat :** Vous savez que le bug est dans "Feature: Add user validation", mais **impossible de savoir** que c'est précisément le commit C4 "Fix validation bug" qui est responsable.

---

### Après REBASE merge sur main

```
main:
  A---B---C1'---C2'---C3'---C4'---C5'---C6'
                              ↑
                              Le bug est ICI
```

Les commits existent individuellement (réécrits avec de nouveaux SHA).

Quand vous faites `git bisect` :
```bash
git bisect start
git bisect bad HEAD
git bisect good B

# Git teste:
# C3' → good
# C5' → bad
# C4' → bad ✓ TROUVÉ!

C4' is the first bad commit
commit a1b2c3d4e5f6
Author: Dev <dev@example.com>
Date:   Wed Oct 23 11:00:00 2025

    Fix validation bug  ← Commit PRÉCIS identifié
```

**Résultat :** Vous savez **exactement** quel changement a introduit le bug.

---

## 🔍 Exemple pratique

Imaginons 6 commits dans une branche feature :

```javascript
// Commit 1: Ajout de la fonction
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// Commit 2: Ajout de la TVA
function calculateTotal(items) {
  const subtotal = items.reduce((sum, item) => sum + item.price, 0);
  return subtotal * 1.2; // +20% TVA
}

// Commit 3: Ajout des réductions
function calculateTotal(items, discount = 0) {
  const subtotal = items.reduce((sum, item) => sum + item.price, 0);
  return (subtotal - discount) * 1.2;
}

// Commit 4: Bug introduit (ordre incorrect)
function calculateTotal(items, discount = 0) {
  const subtotal = items.reduce((sum, item) => sum + item.price, 0);
  return subtotal * 1.2 - discount; // ❌ BUG! La réduction est appliquée APRÈS la TVA
}

// Commit 5: Ajout des tests (qui ne détectent pas le bug)
// Commit 6: Documentation
```

### Avec Squash

```bash
$ git log --oneline main
a1b2c3d (HEAD -> main) Feature: Add invoice calculation
xyz789 Previous commit
```

Bisect trouve uniquement `a1b2c3d`, qui contient **tout** le code des 6 commits mélangés.

**Investigation nécessaire :**
- Lire tout le diff du commit squashé (potentiellement des centaines de lignes)
- Analyser manuellement pour trouver la ligne problématique
- Temps perdu ⏱️

### Avec Rebase

```bash
$ git log --oneline main
f6e5d4c (HEAD -> main) Add documentation
c3b2a1d Add tests
9d8e7f6 Fix discount calculation ← LE BUG EST ICI
5b4c3a2 Add discount support
2a1b0c9 Add VAT calculation
1z9y8x7 Add calculateTotal function
xyz789 Previous commit
```

Bisect trouve précisément `9d8e7f6`.

**Investigation :**
- Le commit est identifié ✅
- Le diff est petit et ciblé
- Correction rapide 🚀

---

## 💡 Conséquence pratique

**Avec Squash, vous devrez :**

1. Identifier la feature problématique via bisect
2. **Retourner dans l'historique de la branche** (si elle existe encore)
   ```bash
   git log origin/feature/user-validation
   ```
3. Ou examiner le **gros diff** du commit squashé
   ```bash
   git show a1b2c3d
   ```
4. Débugger manuellement pour isoler le problème

**Avec Rebase, bisect fait tout le travail :**
```bash
git bisect run npm test
# → Commit précis trouvé automatiquement
```

---

## 📈 Tableau récapitulatif

| Aspect | Squash | Rebase | Merge Commit |
|--------|--------|--------|--------------|
| **Granularité** | Feature entière | Commit précis | Commit précis |
| **Historique** | 1 commit | N commits | N commits + merge |
| **Bisect précision** | ⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Débug après bisect** | Manuel | Automatique | Automatique |
| **Temps d'investigation** | Long | Court | Court |

---

## 🎯 Recommandation

**Si vous utilisez Squash :**
- Gardez les branches feature un moment après le merge
- Ajoutez des tags ou notes sur les commits squashés
- Utilisez des messages de commit très détaillés
- Exemple : 
  ```
  Feature: Add user validation
  
  - Added User model (commit hash from branch)
  - Fixed validation logic (commit hash)
  - Added comprehensive tests
  ```

**Si la capacité de débug est critique :**
- Préférez **Rebase** pour conserver la granularité
- Ou utilisez **Merge Commit** avec `--no-ff`

Votre compréhension est parfaite ! 👏

---

# Absolument ! Excellente analyse 🎯

Vous avez mis le doigt sur un point crucial du **Trunk-Based Development (TBD)** puriste.

---

## 🌳 Trunk-Based Development : Le contexte

### Principe fondamental
```
main (trunk):  A---B---C---D---E---F---G---H
                    ↑   ↑   ↑   ↑   ↑   ↑
                    Commits directs ou branches de quelques heures max
```

**Caractéristiques :**
- Les branches feature vivent **< 24h** (souvent < 2h)
- Merge/intégration **plusieurs fois par jour**
- Branches **supprimées immédiatement** après merge
- Pas de branches long-lived (sauf release branches)

---

## 🔍 Impact sur le bisect selon la stratégie

### Scénario : Bug détecté 1 semaine après l'intégration

```
Développeur crée une branch → 3 commits → merge → branch supprimée
                              (2 heures de vie)
```

### Avec SQUASH en TBD

```
main: A---B---S---D---E---F---G (bug détecté ici)
              ↑
         Commit squashé contenant 3 commits originaux
         Branch feature/login SUPPRIMÉE ✂️
```

**Bisect trouve :** Commit `S`

**Problème :**
- ❌ Les 3 commits originaux n'existent **nulle part**
- ❌ La branche n'existe plus
- ❌ Impossible de retrouver la granularité
- 😰 Vous devez débugger manuellement tout le diff du commit squashé

```bash
$ git log --all --grep="feature/login"  # rien
$ git branch -a | grep login             # rien
$ git show S                             # diff de 500 lignes...
```

### Avec REBASE en TBD

```
main: A---B---C1'---C2'---C3'---D---E---F---G
                     ↑
                  Commit précis du bug
         Branch feature/login SUPPRIMÉE ✂️ (mais pas grave!)
```

**Bisect trouve :** Commit `C2'`

**Avantage :**
- ✅ Les commits individuels **existent dans main**
- ✅ Granularité préservée même si la branche est supprimée
- ✅ Bisect pointe exactement sur le changement problématique
- 🚀 Investigation rapide et précise

---

## 💻 Exemple concret

### Timeline d'une feature en TBD puriste

**10h00** - Développeur crée `feature/fix-payment`
```bash
git checkout -b feature/fix-payment
```

**10h30** - Premier commit
```javascript
// Commit 1: Restructure payment validation
function validatePayment(amount) {
  return amount > 0;
}
```

**11h00** - Deuxième commit  
```javascript
// Commit 2: Add currency check
function validatePayment(amount, currency) {
  return amount > 0 && currency !== null;
}
```

**11h30** - Troisième commit (BUG introduit)
```javascript
// Commit 3: Add limit check
function validatePayment(amount, currency) {
  return amount > 0 && currency !== null && amount < 1000; // ❌ Should be <=
}
```

**12h00** - Merge sur main + **suppression de la branche**
```bash
git checkout main
git merge feature/fix-payment  # (squash ou rebase)
git branch -d feature/fix-payment
git push origin :feature/fix-payment  # Suppression remote
```

### 1 semaine plus tard : Bug détecté

#### Avec Squash
```bash
$ git bisect start HEAD main~50
$ git bisect run npm test

abc123 is the first bad commit
commit abc123
    Fix: Payment validation improvements
    
    - Restructured validation
    - Added currency support  
    - Added limit checks
    
    +150 lines, -80 lines
```

**Vous devez :**
1. Lire 150 lignes de diff
2. Comprendre 3 changements mélangés
3. Trouver manuellement que c'est `< 1000` au lieu de `<= 1000`
4. Temps d'investigation : **30-60 minutes** 😓

#### Avec Rebase
```bash
$ git bisect start HEAD main~50
$ git bisect run npm test

def456 is the first bad commit
commit def456
    Add limit check
    
    +1 line, -0 lines
    
diff --git a/payment.js b/payment.js
-  return amount > 0 && currency !== null;
+  return amount > 0 && currency !== null && amount < 1000;
```

**Vous savez immédiatement :**
1. Le commit précis ✅
2. La ligne exacte ✅  
3. Le changement isolé ✅
4. Temps d'investigation : **2 minutes** 🚀

---

## 🎯 Pourquoi Rebase est quasi-obligatoire en TBD puriste

| Critère | Sans Rebase | Avec Rebase |
|---------|-------------|-------------|
| **Branches supprimées** | ⚠️ Perte d'info | ✅ Info dans main |
| **Bisect efficace** | ❌ Feature-level | ✅ Commit-level |
| **Historique** | Opaque | Transparent |
| **Débug rapide** | Manuel, long | Automatique |
| **Code review post-mortem** | Difficile | Facile |

---

## 🏢 Pratiques réelles en TBD

### Option 1 : TBD puriste + Rebase (Google, Facebook)
```bash
# Commits atomiques directement sur main via rebase
git checkout -b quick-fix
git commit -m "Add validation"      # Commit atomique
git commit -m "Update tests"        # Commit atomique
git rebase main
git checkout main
git merge quick-fix --ff-only       # Fast-forward
git branch -d quick-fix
```

**Résultat :** Historique linéaire avec granularité

### Option 2 : TBD + Squash (compromis)
Certaines équipes utilisent quand même Squash en TBD car :
- ✅ Historique ultra-clean (1 commit = 1 ticket/story)
- ✅ Changelog automatique facile
- ❌ Mais acceptent la perte de granularité pour le bisect

**Mitigation :**
- Commits **très** petits (< 50 lignes)
- Messages de commit exhaustifs
- Feature flags pour isoler les changements
- Tests très complets avant merge

### Option 3 : TBD modéré + Merge Commit
```bash
# Garde un historique des branches même courtes
git merge feature/quick-fix --no-ff
```

Moins puriste, mais garde une trace de "quand" les features ont été intégrées.

---

## 📊 Recommandation selon le contexte

```
Si TBD puriste (branches < 24h, supprimées) :
  └─ Commits atomiques obligatoires
      └─ Rebase pour granularité bisect ⭐⭐⭐
      └─ Squash seulement si commits TRÈS petits

Si TBD modéré (branches < 3 jours) :
  └─ Rebase ou Squash selon préférence équipe
      └─ Garder branches 1-2 semaines pour référence

Si Feature branches (branches > 1 semaine) :
  └─ Merge commit ou Rebase
      └─ Branches archivées, pas supprimées
```

---

## 🎓 Conclusion de votre observation

Vous avez 100% raison :

**En Trunk-Based Development puriste :**
- Les branches sont éphémères et **disparaissent définitivement**
- **Rebase est la seule stratégie** qui préserve la traçabilité fine dans `main`
- Squash devient un handicap majeur pour le debugging et bisect
- C'est pourquoi Google, qui pratique TBD strict, utilise un historique linéaire avec commits atomiques

**L'équation est simple :**
```
TBD puriste + Squash = Perte définitive de granularité
TBD puriste + Rebase = Granularité préservée dans trunk
```

Excellente compréhension des implications architecturales ! 🎖️
