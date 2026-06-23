# Plan : Modification de Main.xaml et ManagerReview.xaml

## Steps

### Step 1: Réécrire Main.xaml — Structure TryCatch linéaire
Remplacer le corps de Main.xaml par :
- Un bloc **TryCatch** global en conteneur racine
- À l'intérieur : une Sequence linéaire avec les 4 variables globales (`main_EmployeeID`, `main_StartDate`, `main_EndDate`, `main_IsConflict`)
- **InvokeWorkflowFile ExtractEmail.xaml** avec arguments `OutArgument` pour `out_EmployeeID`, `out_StartDate`, `out_EndDate` → variables `main_*` (correction des directions actuellement erronées)
- **InvokeWorkflowFile ConflictCheck.xaml** avec `InArgument` pour `in_StartDate`/`in_EndDate` et `OutArgument` pour `out_IsConflict` → `main_IsConflict` (correction)
- **InvokeWorkflowFile ManagerReview.xaml** avec `InArgument` pour `in_IsConflict`, `in_EmployeeID`, `in_StartDate`, `in_EndDate` ← variables `main_*`
- **Catch `BusinessRuleException`** : LogMessage d'erreur avec le message de l'exception

### Step 2: Réécrire ManagerReview.xaml — Gouvernance complète Human-in-the-Loop
Remplacer entièrement le corps de ManagerReview.xaml (supprimer ExcelApplicationCard actuel avec ReadRange + ForEachRow vide) par la structure complète du blueprint :

**a. If Gate — Conflit détecté ?** (`in_IsConflict = True`)
- **Then** (conflit) : LogMessage `[EMAIL-REJET] Conflit détecté pour l'employé {in_EmployeeID}`
- **Else** (pas de conflit) : contient toutes les étapes suivantes dans une Sequence

**b. ExcelApplicationCard #1 — Écriture de la ligne Pending**
- `ReadRangeX` → `dt_Log` (lire MasterLog pour compter les lignes existantes)
- `Assign` : `int_RowIndex = dt_Log.Rows.Count + 2` (position de la nouvelle ligne Excel)
- `Assign` : reconstruire `dt_Log` comme DataTable à 1 ligne `[in_EmployeeID, in_StartDate, in_EndDate, "Pending", Now, "Awaiting action"]`
- `AppendRangeX` → append `dt_Log` sur la feuille MasterLog

**c. LogMessage — Notification manager** (placeholder email manager)

**d. While Loop — Polling** (`str_CurrentStatus = "Pending"`)
- `Delay` : 10 secondes
- `ExcelApplicationCard #2` → `ReadCellX` cellule `"D" & int_RowIndex` → `str_CurrentStatus`

**e. If Final Gate — Décision** (`str_CurrentStatus = "Approved"`)
- **Then** : LogMessage `[EMAIL-APPROUVÉ] Notifier l'employé {in_EmployeeID}`
- **Else** : LogMessage `[EMAIL-REFUSÉ] Notifier l'employé {in_EmployeeID}`

### Step 3: Vérifier les deux fichiers
Lire les deux fichiers écrits pour confirmer que la structure XML est valide et cohérente avec le blueprint.
