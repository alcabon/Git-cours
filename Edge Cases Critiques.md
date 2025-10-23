# Excellent Catch : Les Edge Cases Réels

Vous avez raison ! J'aurais dû aborder ça **dès le début** parce que c'est là où **Gearset gère VRAIMENT la complexité**. Les cas simples (1 feature = 1 problem) sont trivials. Les vrais problèmes viennent des **dépendances entrecroisées**.

## Les Edge Cases Critiques

### Edge Case 1 : Chaîne de Dépendances

```
Commit 1: Feature A → Create Field_X
Commit 2: Feature B → Create Layout_Y (uses Field_X)
Commit 3: Feature C → Create Flow_Z (uses Layout_Y)
Commit 4: Feature D → Create Apex_W (calls Flow_Z)

Production problem détecté: Layout_Y (Feature B) est cassée

Gearset doit répondre:
  "Si je revert Layout_Y, je casse quoi ?"
  
  Dependency Graph:
    Field_X ← Layout_Y ← Flow_Z ← Apex_W
    
  If revert Feature B (Layout_Y):
    ├─ Flow_Z perd sa source (BROKEN)
    ├─ Apex_W appelle Flow_Z cassé (BROKEN)
    ├─ Utilisateurs voir erreur (IMPACT)
    └─ Question: faut-il revert Feature C & D aussi ?
```

### Edge Case 2 : Modifications Partagées

```
Commit 1: Feature A → Modify Field_X (add field "Status")
Commit 2: Feature B → Modify Field_X (add picklist values)
Commit 3: Feature C → Modify Field_X (change validation rule)

Problem: Field_X brisé

Gearset query:
  "Qui a modifié Field_X ?"
  → A, B, C
  
  "Quel commit l'a cassé ?"
  → Could be A? B? C? Leur combination?
  
  Revert juste Feature B:
    ├─ Field_X perd les picklist values de B
    ├─ Validation rule de C reste (but incompatible?)
    ├─ Feature A's changes toujours là (but combinaison broken?)
    └─ État potentiellement INCOHÉRENT
```

### Edge Case 3 : Dépendances Circulaires

```
Feature A: Custom Object Account_Ext (depends on Account)
Feature B: Trigger on Account_Ext
Feature C: Flow on Account that triggers Feature B

Problem: Feature B cassée

Gearset:
  A ← B ← C → A (cycle!)
  
  Revert B:
    ├─ Flow en C orpheline (references B qui n'existe plus)
    ├─ Account_Ext en A peut appeler B qui n'existe plus
    └─ Ordre de revert? Revert A & C aussi?
```

### Edge Case 4 : Side Effects en Cascade

```
Feature A: Create Field + Validation Rule
Feature B: Create Flow (triggered by validation)
Feature C: Create Process Builder (triggered by flow)

Flow en C déclenche 500 records/day processing

Problem: Field cassé (Feature A)

Revert juste Feature A:
  ├─ Validation disparaît
  ├─ Flow perd son trigger
  ├─ Process Builder perd son input
  ├─ 500 records en attente → timeout/error
  ├─ Database grows unchecked
  └─ NEW problem créé par le revert!
```

### Edge Case 5 : État Incohérent Post-Revert

```
Commit 1: Feature A → Create Picklist Field with values ["Active", "Inactive"]
Commit 2: Feature B → Create Validation: "Status must be 'Active' or 'Pending'"

Bug: Validation rule uses undefined picklist value "Pending"

Revert Feature B:
  ├─ Validation disparaît (ok)
  └─ Field reste avec ["Active", "Inactive"]
  
  → État cohérent, c'est bon
  
BUT if problem était dans Feature A:
  Revert Feature A:
    ├─ Field disparaît
    ├─ BUT Validation en B référence la field → ORPHAN
    ├─ Salesforce déploiement échoue? (référence invalide)
    └─ Revert ne peut pas compléter!
```

---

## La Complexité Que Gearset Doit Gérer

### 1. Build & Maintain Dependency Graph (DAG)

```yaml
metadata_dependency_graph:
  CustomField.Account.Status__c:
    referenced_by:
      - ValidationRule.StatusValidation
      - Flow.RefreshStatus
      - Apex.AccountHelper
      - RecordType.Account.NewType
    depends_on: []
    
  ValidationRule.StatusValidation:
    referenced_by:
      - Process.AutoUpdate
    depends_on:
      - CustomField.Account.Status__c
      
  Flow.RefreshStatus:
    referenced_by:
      - Apex.CallFlow
    depends_on:
      - CustomField.Account.Status__c
      - Apex.Helper (for sub-flow)
      
  # ... etc, très complexe

Problèmes à gérer:
  ├─ Detect cycles (A → B → C → A)
  ├─ Keep graph updated (chaque deploy peut l'invalider)
  ├─ Handle indirect refs (A refs B refs C, but A implicitly depends C)
  └─ Parse metadata XML correctement (nombres de ref patterns...)
```

### 2. Topological Sort pour Revert Sûr

```
Si vous voulez revert Feature B (qui casse Layout_Y):
  
Dependency Graph dit:
  Field_X → Layout_Y → Flow_Z → Apex_W
  
Question: Peut-on juste revert Layout_Y ?

Answer: NO si Flow_Z en dépend
        OUI si on revert aussi Flow_Z (et Apex_W)
        
Gearset doit décider:
  Option 1: "Can't revert B safely, it breaks C & D"
            → Reject auto-revert, human decides
            
  Option 2: "Revert B AND C AND D together"
            → Transactional rollback (all-or-nothing)
            → Deploy order: D first, then C, then B
            
  Option 3: "Revert B and 'stub' C & D temporarily"
            → Replace Flow_Z with noop
            → Deploy stub, then real revert
            → More complex
```

### 3. Artefacts Modifiés Partiellement

```
Field_X modified by Feature A, B, C

Current state:
  - Field_X.label = "Account Status" (from A)
  - Field_X.picklist = [Active, Inactive, Pending] (from B & C)
  - Field_X.required = true (from C)

Revert Feature B:
  Gearset must know:
    "What part of Field_X did B change ?"
    
  Options:
    a) Store full state snapshots (memory heavy)
       Before B: label, picklist, required
       After B: label, picklist, required
       Diff = "B added 'Pending' to picklist"
       Revert = remove 'Pending' only
       
    b) Store delta per feature (complex)
       Feature A delta: +label
       Feature B delta: +picklist values
       Feature C delta: +required
       Revert = apply inverse of B's delta only
       
       Problem: C's "required" depends on B's picklist
       Remove B's picklist → C's constraint becomes meaningless
       
    c) Accept partial revert is unsafe
       Revert B → must also revert C
```

---

## Comment Gearset Devrait Gérer Ça

### Strategy 1: Conservative (Safe but Limited)

```
Revert request: "Remove Feature B"

Gearset algorithm:
  1. Find all artifacts in Feature B
  2. Find all artifacts that reference B
  3. Find all artifacts that reference those (recursive)
  4. Scope = B + all recursive dependents
  
  If Scope > threshold:
    Reject: "Too many artifacts affected, requires manual decision"
    Output: "Revert would affect 47 other artifacts"
    Human: Click "force revert all" or "cancel"
    
  If Scope ≤ threshold:
    Generate rollback for all in scope
    Deploy transactionally (all succeed or all fail)
    If any fail: rollback entire operation
```

### Strategy 2: Optimistic (Attempts Granular)

```
Revert request: "Remove Feature B (Layout_Y)"

Gearset algorithm:
  1. Extract Feature B artifacts
  2. For each artifact:
    a. Find dependents
    b. Check if revert breaks them
    c. Validate inverse deploy won't error
  3. Try revert in staging first
  4. If staging OK: deploy to prod
  5. If staging FAIL: 
     - Recommend also reverting dependents
     - Ask human confirmation
```

### Strategy 3: State Machine (Most Realistic)

```
Feature Lifecycle States:
  DEPLOYED
  DEPLOYED_WITH_ISSUES
  CANDIDATE_FOR_REVERT
  REVERTED
  REVERTED_WITH_ISSUES
  
When revert requested:
  Current state: DEPLOYED
  Target state: REVERTED
  
  Pre-flight checks:
    ├─ Dependency analysis (find cycles, orphans)
    ├─ Validation dry-run (will revert succeed?)
    ├─ Impact assessment (what breaks?)
    └─ Human approval if risk > threshold
    
  Revert execution:
    ├─ Deploy inverse to staging
    ├─ Validate staging (no errors)
    ├─ Deploy to production
    ├─ Smoke tests
    ├─ Update state to REVERTED
    └─ If any step fails: state = REVERTED_WITH_ISSUES, alert team
```

---

## Les Vraies Contraintes

### 1. **Ordre de Revert Importent**

```
Revert A, B, C ensemble (B depends on A, C depends on B):

Order 1 (WRONG):
  Revert C → Flow_Z disappears
  Revert B → Layout_Y disappears
  Revert A → Field_X disappears
  ✅ Works (bottom-up)

Order 2 (WRONG):
  Revert A → Field_X disappears
  Layout_Y en B orpheline (references Field_X gone!)
  Deploy FAILS
  ❌ Cascading failure
  
Gearset must: topological sort + reverse order
```

### 2. **Transactionalité**

```
Deploy inverse de Feature B (3 artifacts):

Scenario 1: All succeed
  ✅ State consistent
  
Scenario 2: 2 succeed, 1 fails
  ❌ Partial revert = BROKEN state
  ├─ 2 artifacts reverted
  ├─ 1 artifact still in old state
  ├─ References mismatch
  └─ Production broken WORSE than before
  
Gearset must:
  ├─ Deploy as atomic unit (if possible)
  ├─ OR validate each step can fail independently
  ├─ OR rollback entire operation if any step fails
  └─ = major complexity vs simple git revert
```

### 3. **Validation Runtime**

```
Revert Feature B layout:
  
Pre-deploy validation:
  ├─ Does layout reference non-existent fields? (would fail)
  ├─ Does flow reference non-existent layout? (would break)
  ├─ Does any custom code hardcode layout name? (discovery hard!)
  └─ Are there active sessions using the layout? (UX break)

During deploy:
  ├─ Salesforce API returns error? (recover how?)
  ├─ Partially deployed? (state inconsistent)
  └─ Logs? Audit trail?

Post-deploy validation:
  ├─ Re-run tests to confirm no regressions from revert
  ├─ Monitor for orphaned references
  ├─ Alert if new issues appear
  └─ Alert team for manual verification
```

---

## Pourquoi C'était Un Edge Case Important

Vous aviez raison de souligner : **ces dépendances entrecroisées sont OÙ Gearset crée vraiment de la valeur**.

Sans ça, Gearset serait juste "oh, Git mais avec UI pour granular revert". 

**Avec** la gestion des dépendances, Gearset devient :
- **Dependency solver** (like npm/maven mais pour Salesforce)
- **Impact analyzer** ("what breaks if...")
- **Safe deployment orchestrator** (ordre, validation, rollback)
- **Risk quantifier** ("50 artifacts affected")

---

## Résumé : Les Vraies Contraintes

```
Edge cases Gearset doit gérer:

1. Chaînes de dépendances
   → Topological sort pour revert order

2. Modifications partielles/partagées  
   → State snapshots ou delta tracking

3. Cycles et références indirectes
   → Graph analysis, cycle detection

4. Artefacts orphelins après revert
   → Validation pre-revert et post-revert

5. Ordre de déploiement
   → Bottom-up, pas top-down

6. Atomicité du revert
   → Transactional ou all-or-nothing

7. Side effects en cascade
   → Processus déclenchés, data processing

8. Validation à la revert-time
   → Peut-on vraiment revert en toute sécurité?

9. Artefacts non-découvrables
   → Code hardcoded, requêtes SQL, etc

10. Human decision points
    → Quand refuser auto-revert?
```

C'est pour ça que c'est complexe. Et c'est exactement ce qui justifie d'avoir une **tool spécialisée** vs juste Git. 🎯
