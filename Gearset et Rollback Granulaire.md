# Gearset et Rollback Granulaire : Architecture Théorique

Excellente observation ! Vous avez identifié quelque chose de **fondamental** : Gearset résout un problème que Git seul ne peut pas résoudre nativement. Voici mon analyse de ce qu'ils font probablement.

## Le Problème Git "Vanille"

Avec Git classique, rollbacker un seul champ est impossible :

```
Commit abc123:
  - Ajoute Field "Status"
  - Ajoute Layout "AccountLayout"
  - Modifie RecordType
  
git revert abc123 ?
→ TOUT revient : les 3 artefacts
```

Pour isoler, il faudrait **un commit par artefact** :

```
Commit 1: Add Field "Status"
Commit 2: Add Layout "AccountLayout"
Commit 3: Modify RecordType

git revert 2
→ Seul le layout est revertir
```

Mais c'est **contre-nature** pour les équipes Salesforce et créerait une charge Git énorme.

---

## Hypothèse 1 : Metadata Registry (Plus Probable)

Gearset **ne fonctionne pas avec Git**, mais avec **l'état des métadonnées Salesforce**. Ils maintiendraient probablement une **couche de versioning métier** au-dessus :

### Architecture Hypothétique

```
Salesforce Orgs
       ↓
Gearset Metadata State Database
       ↓
Artifact Version Registry
       ├─ Account.Status: [v1, v2, v3, v4]
       ├─ Account.Layout: [v1, v2]
       └─ RecordType.NewRT: [v1]
       
Timestamped Snapshots (par artefact)
       ├─ 2025-10-01 10:30 → Status v2
       ├─ 2025-10-02 14:15 → Status v3
       ├─ 2025-10-05 09:00 → Status v4
```

### Rollback Unitaire

```
Request: "Rollback Status field to 2025-10-02"

Gearset:
  1. Lookup: Status v3 (état à cette date)
  2. Calcul diff: v4 → v3
  3. Generate: Deployment XML (seulement Account.Status inversé)
  4. Deploy: Juste ce changement
```

**Avantage** : Indépendant de comment les commits Git ont été structurés.

---

## Hypothèse 2 : Commits Hyper-Atomiques

Mais si Gearset utilise Git en arrière-plan, ils pourraient **forcer une structure stricte** :

```
Commit Policy:
├─ 1 Artefact = 1 Commit
├─ package.xml versionnée par artefact
└─ Metadata manifest (Custom JSON indexant les artefacts)
```

### Structure Possible

```
git log --oneline

a1b2c3d | Deployment#123: Add Field Account.Status
d4e5f6g | Deployment#124: Add Layout Account.NewLayout
h7i8j9k | Deployment#125: Modify Field Account.Status
```

Avec **métadonnées par commit** :

```json
// .git-metadata/a1b2c3d.json
{
  "deployment_id": "123",
  "artifacts": ["Account.Status"],
  "type": "create",
  "org": "production"
}

// .git-metadata/h7i8j9k.json
{
  "deployment_id": "125",
  "artifacts": ["Account.Status"],
  "type": "modify",
  "org": "production"
}
```

### Rollback Query

```bash
gearset rollback --artifact "Account.Status" --to "Deployment#124"

Gearset:
  1. Parcourt les commits avec artifacts = ["Account.Status"]
  2. Trouve: a1b2c3d (create) → h7i8j9k (modify)
  3. Calcule: revert h7i8j9k seulement
  4. Cherry-pick (ou revert) a1b2c3d
```

---

## Hypothèse 3 : Delta Structuré (Plus Sophistiqué)

Gearset pourrait parser les **métadonnées XML Salesforce** et créer des **deltas sémantiques** plutôt que textuels :

```xml
<!-- Métadonnées Salesforce (format natif) -->
<CustomField>
  <fullName>Account.Status__c</fullName>
  <label>Status</label>
  <type>Picklist</type>
  <valueSet>
    <value>
      <fullName>Active</fullName>
    </value>
    <value>
      <fullName>Inactive</fullName>
    </value>
  </valueSet>
</CustomField>
```

**Delta Structuré** (pas textuel) :

```json
{
  "entity": "CustomField",
  "identifier": "Account.Status__c",
  "changes": [
    {
      "date": "2025-10-01",
      "field": "valueSet.values",
      "added": ["Active", "Inactive"],
      "removed": []
    },
    {
      "date": "2025-10-05",
      "field": "label",
      "old_value": "Status",
      "new_value": "Account Status"
    }
  ]
}
```

**Rollback** = Requête de reconstruction d'état :

```
État cible: 2025-10-02
Account.Status__c revenir à label="Status", valueSet=["Active", "Inactive"]
```

---

## Ce Que Gearset Revendique Probablement

Regardons leur discours marketing (hypothétiquement) :

> "Rollback granulaire au niveau d'un champ"

Cela implique qu'ils trackent **au niveau sémantique** (métadonnées Salesforce), pas au niveau Git.

> "Déploiements ciblés"

Cela signifie qu'ils recalculent les deltas nécessaires pour atteindre un état cible.

---

## Avantage vs Git Pur

| Approche | Rollback Unitaire | Simplicité | Naturel |
|----------|-------------------|-----------|---------|
| **Git atomique** | ✅ Possible si 1 artefact = 1 commit | ❌ Complexe, beaucoup de commits | ❌ Non naturel |
| **Metadata Registry** | ✅ Natif, par artefact | ✅ Simple | ✅ Oui |
| **Delta Structuré** | ✅ Très précis | ⚠️ Moyen | ✅ Oui |

---

## Mon Verdict

**Gearset utilise probablement une combination** :

1. **Git en background** pour l'historique et la collaboration
2. **Metadata Registry** pour tracker l'état des artefacts à travers le temps
3. **Deployment Manifest** indexant les artefacts par déploiement
4. **Semantic Queries** pour les rollbacks granulaires

```
Git History
  ↓
Parse Deployments
  ↓
Build Artifact Index
  ↓
Rollback Query: Account.Status → State X
  ↓
Generate Inverse Deployment
  ↓
Deploy to Salesforce
```

**Terme plus juste que "atomique"** : **Artefacts versionné indépendamment** ou **"Metadata State Management"**.

Gearset traite les métadonnées Salesforce comme des **entités discrètes versionnées**, pas comme des blobs textuels Git. C'est ça qui change tout.

C'est une leçon : **Git est généraliste, mais pour des domaines spécialisés (Salesforce), vous avez souvent besoin d'une couche sémantique au-dessus**. 🎯
