# Cherry-Pick pour Aligner les Branches Multi-Environnement

Excellente question sur un cas d'usage réel ! Mais clarifions d'abord le flux, car **cherry-pick seul n'est qu'une partie de la solution**.

## Scenario Classique : Promotion Linéaire

```
dev  ──→ integ  ──→ preprod  ──→ prod
      (merge)      (cherry-pick ou merge)
```

### Cas 1 : Feature Abandonnée EN PREPROD

Une feature F1 a progressé jusqu'à preprod, mais elle est rejetée/instable et vous décidez de l'abandonner **avant prod**.

**Problème** : Comment l'enlever de preprod sans casser integ/dev ?

```
dev      : commit A ─→ F1 (commits) ─→ commit B
integ    : commit A ─→ F1 (commits) ─→ commit B
preprod  : commit A ─→ F1 (commits) ─→ commit B ─→ commit C (corrections)
```

**Solution avec Cherry-Pick** :

```bash
# 1. Sur preprod, annuler F1
git revert <commit-F1-start>...<commit-F1-end>
# ou sélectivement avec revert si F1 = plusieurs commits

# 2. Créer une branche "clean" preprod SANS F1
git checkout -b preprod-without-F1 commit-B
# Puis cherry-pick seulement les bons commits (pas F1)

# 3. Basculer vers preprod
git checkout preprod
git reset --hard preprod-without-F1
git push -f origin preprod  ⚠️ Risqué si partagé
```

---

## Scenario Meilleur : Cherry-Pick Sélectif

C'est l'**usage principal** de cherry-pick dans ce contexte :

### Architecture Idéale

```
dev         : F1, F2, F3, F4, F5 (tout)
integ       : sélection (F1, F2, F3)
preprod     : sélection (F1, F2)
prod        : ce qui est stable (F1, peut-être F2)
```

**Flux de promotion sélectif** :

```bash
# Dev a 5 features, on promeut seulement F1 et F2 à integ
git checkout integ
git cherry-pick <hash-F1> <hash-F2>
git push

# Integ a F1, F2, on promeut seulement F1 à preprod
git checkout preprod
git cherry-pick <hash-F1>
git push
```

### L'Alignement en Cas d'Abandon

Si vous décidez d'**abandonner F2 en preprod** :

```
Avant :
dev      : F1 ─→ F2 ─→ F3
integ    : F1 ─→ F2
preprod  : F1 ─→ F2 ─→ corrections

Après (abandon F2) :
dev      : F1 ─→ F2 ─→ F3  ← reste inchangé
integ    : F1 ─→ F2         ← reste inchangé
preprod  : F1 ─→ corrections ← F2 annulé
```

**Avec cherry-pick** :

```bash
# Sur preprod : annuler F2
git revert <hash-F2>

# Les autres branches (dev, integ) gardent F2
# → Les futures promotions à preprod peuvent sauter F2
```

---

## Pattern Recommandé : Gestion d'Artefacts

Pour un workflow robuste avec abandons potentiels :

### 1. Tracer Explicitement les Artefacts

```bash
# Chaque feature/artefact = branche feature
git checkout -b feature/F2-awesome-stuff dev

# Commits de travail
git commit -m "F2: awesome stuff part 1"
git commit -m "F2: awesome stuff part 2"

# Tagging des artefacts promotables
git tag -a artefact/F2-v1 -m "F2 ready for integ"
```

### 2. Promotion Sélective avec Cherry-Pick

```bash
# Integ accepte F2
git checkout integ
git cherry-pick artefact/F2-v1
git push

# Preprod refuse F2 (ou version ultérieure)
# → Pas de cherry-pick, F2 n'arrive jamais en preprod
```

### 3. Abandon Explicite

```bash
# Si F2 était déjà en preprod mais doit être retiré
git checkout preprod
git revert <commit-F2>  # Annule les changements de F2
git push

# Dev et integ ne sont pas affectées
# Futures promotions: cherry-pick les bons commits
```

---

## Cas Complexe : Réalignable Avec Cherry-Pick

Imaginons que preprod contient **F1 + F2 + corrections**, et vous devez **retirer F2** tout en gardant les corrections :

```bash
# Identifier les commits
# preprod: C1(F1) → C2(F2) → C3(F2-suite) → C4(corrections)

# Solution 1 : Revert sélectif
git revert C2..C3  # Annule F2 et sa suite

# Solution 2 : Rebase interactif (réécriture d'historique local)
git rebase -i <parent-de-C1>
# Marquer C2 et C3 comme "drop"
# Garder C1 et C4

# Solution 3 : Cherry-pick des bons commits
git checkout -b preprod-clean <hash-avant-F2>
git cherry-pick C1 C4  # Saute F2
git reset --hard preprod-clean && rm preprod-clean
```

---

## Avantages de Cherry-Pick dans ce Workflow

| Aspect | Avantage |
|--------|----------|
| **Sélectivité** | Promeut seulement les features validées |
| **Granularité** | Commits/artefacts individuels, pas toute la branche |
| **Non-linéarité** | Saute les features abandonnées |
| **Traçabilité** | Chaque promotion laisse des traces |
| **Flexibilité** | Peut réordonner les promotions |

---

## Règle Pratique pour Votre Workflow

```
┌─ Artefact abandonné en preprod ?
│
├─ OUI → git revert sur preprod
│        dev/integ gardent l'artefact (ok, pas utilisé)
│
└─ Prochaine promotion à preprod ?
   ← cherry-pick seulement les nouveaux artefacts validés
      (skips automatiquement les abandonnés)
```

**Mantra** :
- **Cherry-pick** = "Je sélectionne ce qui est bon"
- Combine avec **revert** pour nettoyer ce qui ne l'est pas

C'est la vraie puissance pour les workflows multi-environnement ! 🎯
