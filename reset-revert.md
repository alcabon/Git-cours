# `git reset` vs `git revert` : Deux Stratégies Opposées

Excellente distinction ! Ces deux commandes agissent de façons **radicalement différentes** sur l'historique, et le choix entre elles a des implications majeures.

## `git reset` : Réécriture d'Historique (Destruction)

### Ce qu'il fait

`git reset` **supprime des commits** de l'historique en déplaçant la branche vers un commit antérieur :

```
Avant :
main  ─→ abc123 ─→ def456 ─→ ghi789  (HEAD ici)

git reset --hard def456

Après :
main  ─→ abc123 ─→ def456  (HEAD ici)
                   ↑
           Les commits ghi789... 
           deviennent orphelins/inaccessibles
```

### Types de reset

```bash
git reset --soft HEAD~1
# Déplace HEAD, garde les changements en staging
# ✓ Utile pour corriger le dernier commit avant de le repousser

git reset --mixed HEAD~1  (défaut)
# Déplace HEAD et l'index, déstage les changements
# ✓ Utile pour "défaire" les changements en working directory

git reset --hard HEAD~1
# Déplace HEAD, index ET working directory
# ⚠️ DESTRUCTIF : perte des données locales non committées
```

### Problème majeur : Historique Partagé

```
Avant (shared repo) :
main  ─→ abc123 ─→ def456 ─→ ghi789

Vous : git reset --hard def456
main  ─→ abc123 ─→ def456

Collègue qui a ghi789 localement :
collision ! Il ne peut pas push sans conflit
```

**Règle d'or** : Ne JAMAIS `reset` sur des commits partagés/pushés → **brise la collaboration distribuée**.

---

## `git revert` : Annulation Par Nouveau Commit (Traçabilité)

### Ce qu'il fait

`git revert` **crée un nouveau commit** qui annule les changements d'un commit existant :

```
Avant :
main  ─→ abc123 ─→ def456 (changement A) ─→ ghi789

git revert def456

Après :
main  ─→ abc123 ─→ def456 ─→ ghi789 ─→ jkl123 (NOUVEAU)
                    (A)                  (-A, inverse A)
```

### Caractéristiques

- ✅ Crée un **nouveau commit** identifié par un hash unique
- ✅ L'historique reste **intact et traçable**
- ✅ Les autres développeurs voient clairement ce qui s'est passé
- ✅ **Peut être pushé** sans risque de conflit
- ✅ Parfait pour les branches partagées

```bash
git revert abc123        # Annule le commit abc123 par un nouveau commit
git log                  # Voit l'annulation + le commit original
```

---

## Comparaison Directe

| Aspect | `reset` | `revert` |
|--------|---------|----------|
| **Action** | Supprime commits | Crée commit d'annulation |
| **Historique** | Réécrit/détruit | Préserve |
| **Hash du commit annulé** | Disparaît | Reste visible |
| **Traçabilité** | ❌ Opaque | ✅ Transparente |
| **Collaboratif** | ❌ Dangereux | ✅ Sûr |
| **Quand l'utiliser** | Avant push seulement | Après push |

---

## Cas d'Usage Concrets

### Scenario 1 : Vous avez commis une erreur AVANT de pusher

```bash
# Vous avez fait 3 commits locaux, le dernier est mauvais
git reset --soft HEAD~1
# Modifiez les fichiers
git commit -m "Correction du dernier commit"
git push
```
✅ Propre, aucun problème.

### Scenario 2 : Vous avez pushé une erreur, collègues l'ont aussi

```bash
# Les commits abc123, def456 sont déjà pushés
# Vous réalisez que def456 contient un bug

git revert def456
# Crée jkl123 qui annule def456
git push
# Collègues font git pull
# Tout le monde voit l'annulation
```
✅ Collaboratif, traçable, sûr.

### Scenario 3 : MAUVAIS approche

```bash
# Commits pushés
git reset --hard HEAD~2
git push -f
```
❌ Collègues en rage. Conflit majeur. À NE PAS FAIRE.

---

## Règles Pratiques

```
┌─ Commit pushé ? ────── OUI ─────→ git revert ✅
│                         NON ─────→ git reset ✅
└─ Historique partagé ? ─ OUI ─────→ git revert ✅
                         NON ─────→ git reset ✅
```

**Mantra** :
- **`reset`** = "Je n'ai jamais fait ça" (local, avant partage)
- **`revert`** = "J'ai fait ça, mais je l'annule" (public, historique)

---

## Récupération Après Reset Destructif

Si vous avez fait `git reset --hard` par erreur :

```bash
git reflog                    # Affiche l'historique des mouvements de HEAD
# Vous voyez : jkl123 HEAD@{0}
git reset --hard jkl123      # Récupère le commit "perdu"
```

Git ne supprime vraiment rien avant la collecte des ordures (gc) → recovery possible pendant ~30 jours.

---

Résumé : **`reset` réécrit, `revert` ajoute**. Le choix détermine si vous cachez ou exposez l'historique.
