# 🚀 QUICK START: Implementar los 5 Fixes Críticos

> Guía paso a paso. Tiempo total: ~4 horas para los 5 críticos.

---

## ANTES DE EMPEZAR

1. **Leer:** `/workspaces/prediccion_dengue/RESUMEN_EJECUTIVO.md` (5 min)
2. **Tener abierto:** `/workspaces/prediccion_dengue/Notebook_6_model_training_iter2.ipynb`
3. **Tener abierto:** `/workspaces/prediccion_dengue/FIXES_CODE_SNIPPETS.md` (para copiar)

---

## ✅ FIX #1: Eliminar RobustScaler (15 min)

### Paso 1: Localizar el código
- Buscar en notebook: `RobustScaler` (Ctrl+F)
- Debería encontrar algo como:
  ```python
  from sklearn.preprocessing import RobustScaler
  scaler = RobustScaler()
  df_clima[clima_vars_num] = scaler.fit_transform(df_clima[clima_vars_num])
  ```

### Paso 2: ELIMINAR esas líneas
- Seleccionar las 3 líneas del RobustScaler
- Borrar (Delete)
- Reemplazar con comentario:
  ```python
  # Variables climáticas sin escalado (LightGBM es insensible a escala)
  ```

### Paso 3: Verificar
- Runear la celda: debe funcionar sin errores
- Verificar que `df_clima` tenga las variables originales (sin transform)

### ✅ DONE: Fix #1

---

## ✅ FIX #2: Feature Selection Solo Train (30 min)

### Paso 1: Localizar dónde se llama `feature_selection_pipeline()`
- Buscar: `feature_selection_pipeline(` (Ctrl+F → segundo resultado)
- Debería estar alrededor de línea 4050-4100
- Encontrar la celda que contiene:
  ```python
  selected_features, scores = feature_selection_pipeline(
      df_feat,                    # ← AQUÍ incluye 2024
      CANDIDATES,
      TARGET,
      top_n=30
  )
  ```

### Paso 2: REEMPLAZAR ese bloque
- Copiar del archivo FIXES_CODE_SNIPPETS.md, sección "FIX #2"
- Pegar reemplazando el antiguo
- Debería ahora ser:
  ```python
  df_train_only = df_feat[df_feat['anio'] < 2020].copy()
  selected_features, scores = feature_selection_pipeline(
      df_train_only,              # ← ONLY train
      CANDIDATES,
      TARGET,
      top_n=30
  )
  ```

### Paso 3: Agregar diagnóstico
- Copiar las líneas de verificación:
  ```python
  n_features_changed = len(set(selected_features) - set(FEATURE_COLS))
  if n_features_changed > len(FEATURE_COLS) * 0.3:
      print(f"⚠️ ADVERTENCIA: {n_features_changed} features cambieron")
  ```

### Paso 4: Runear y verificar
- Ejecutar la celda
- Comparar `selected_features` con run anterior (si es muy diferente → documento)

### ✅ DONE: Fix #2

---

## ✅ FIX #3: Agregar .shift(1) a Rolling Climáticas (15 min)

### Paso 1: Localizar función `build_features()`
- Buscar: `def build_features(` (Ctrl+F)
- Encontrar el bloque:
  ```python
  # Rolling mean 4 semanas
  df[f'{var}_ma4'] = g[var].transform(
      lambda x: x.rolling(4, min_periods=1).mean()
  )
  ```

### Paso 2: REEMPLAZAR Rolling climáticas
- Cambiar `.rolling()` a `.shift(1).rolling()`
- ANTES:
  ```python
  lambda x: x.rolling(4, min_periods=1).mean()
  ```
- DESPUÉS:
  ```python
  lambda x: x.shift(1).rolling(4, min_periods=1).mean()
  ```

### Paso 3: TAMBIÉN cambiar rolling std
- Buscar: `{var}_std4` en la misma función
- Cambiar igual:
  ```python
  # ANTES:
  lambda x: x.rolling(4, min_periods=1).std()
  
  # DESPUÉS:
  lambda x: x.shift(1).rolling(4, min_periods=1).std()
  ```

### Paso 4: Runear feature engineering
- Ejecutar la celda de `build_features()`
- Verificar que no hay errores

### Paso 5: Comparar MAPE
- Ejecutar walk-forward después del cambio
- Comparar MAPE anterior vs nuevo
- Documentar cambio: "Expected 1-3% drop due to temporal alignment"

### ✅ DONE: Fix #3

---

## ✅ FIX #4: Actualizar Hipótesis (1 hora)

### Paso 1: Limpiar HTML (elige OPCIÓN A)
- Buscar en HTML: "EpiGNN y transformers (PatchTST)"
- REEMPLAZAR por:
  ```html
  "LightGBM que integra variables climáticas 
   y epidemiológicas con rezago"
  ```

### Paso 2: Actualizar bullets de modelos
- Buscar: `- **Modelos neuronales:** EpiGNN...`
- REEMPLAZAR por:
  ```markdown
  - **Modelo candidato:** LightGBM (gradient boosting)
  - **Interpretabilidad:** SHAP
  - **Trabajo futuro:** EpiGNN y PatchTST
  ```

### Paso 3: Remover tarjetas de EpiGNN/PatchTST
- Si están en HTML como tarjetas, comentarlas o marcar como "Futuro":
  ```html
  <!-- Future Work: EpiGNN implementation -->
  <div class="card" style="opacity: 0.7;">
      <div class="card-title">🔮 Trabajo Futuro: EpiGNN</div>
  </div>
  ```

### Paso 4: Testear HTML
- Viewear el HTML actualizado en navegador
- Verificar que NO hay referencias a EpiGNN/PatchTST como modelos implementados

### ✅ DONE: Fix #4

---

## ✅ FIX #5: Tabla de Métricas Completa (1.5 horas)

### Paso 1: Agregar funciones de métricas
- Antes del walk-forward, agregar:
  ```python
  def mape(y_true, y_pred, min_casos=5):
      mask = y_true >= min_casos
      if mask.sum() == 0:
          return np.nan
      return float(np.mean(np.abs((y_true[mask] - y_pred[mask]) / y_true[mask])) * 100)
  
  def smape(y_true, y_pred):
      denom = (np.abs(y_true) + np.abs(y_pred)) + 1
      return float(100 * np.mean(2 * np.abs(y_true - y_pred) / denom))
  
  def mae(y_true, y_pred):
      return float(np.mean(np.abs(y_true - y_pred)))
  
  def rmse(y_true, y_pred):
      return float(np.sqrt(np.mean((y_true - y_pred) ** 2)))
  ```

### Paso 2: Copiar función `brote_detection_accuracy()`
- Del archivo FIXES_CODE_SNIPPETS.md, sección FIX #5
- Copiar completa

### Paso 3: Buscar print de resultados
- Buscar: `print('=== Test set: brote 2024 ===')` (Ctrl+F)
- REEMPLAZAR TODO ese bloque de print por:
  ```python
  # Calcular TODAS las métricas
  mae_bl = mae(y_test_np, y_baseline)
  mape_bl_5 = mape(y_test_np, y_baseline, min_casos=5)
  smape_bl = smape(y_test_np, y_baseline)
  rmse_bl = rmse(y_test_np, y_baseline)
  brote_bl = brote_detection_accuracy(y_test_np, y_baseline, threshold=5)
  
  mae_lgb = mae(y_test_np, y_lgb)
  mape_lgb_5 = mape(y_test_np, y_lgb, min_casos=5)
  smape_lgb = smape(y_test_np, y_lgb)
  rmse_lgb = rmse(y_test_np, y_lgb)
  brote_lgb = brote_detection_accuracy(y_test_np, y_lgb, threshold=5)
  
  # Tabla
  print('\n' + '='*90)
  print('TEST SET 2024 - BROTE DENGUE')
  print('='*90)
  print(f'{"Modelo":<15} {"MAE":>9} {"MAPE≥5%":>10} {"SMAPE%":>9} {"RMSE":>9} {"Brote%":>9}')
  print('-'*90)
  print(f'{"Baseline (MA4)":<15} {mae_bl:>9.1f} {mape_bl_5:>10.1f} {smape_bl:>9.1f} {rmse_bl:>9.1f} {brote_bl:>9.1f}')
  print(f'{"LightGBM":<15} {mae_lgb:>9.1f} {mape_lgb_5:>10.1f} {smape_lgb:>9.1f} {rmse_lgb:>9.1f} {brote_lgb:>9.1f}')
  print('-'*90)
  print(f'{"Mejora (pp)":<15} {mae_bl-mae_lgb:>9.1f} {mape_bl_5-mape_lgb_5:>10.1f} {smape_bl-smape_lgb:>9.1f} {rmse_bl-rmse_lgb:>9.1f} {brote_bl-brote_lgb:>9.1f}')
  print('='*90)
  ```

### Paso 4: Runear y verificar
- Ejecutar walk-forward
- Debería printear tabla con 5 columnas de métricas
- Comparar MAPE: debe estar entre 20-30% o similar al anterior

### Paso 5: Actualizar hipótesis texto
- En sección de Hipótesis del notebook (markdown), cambiar:
  - ANTES: "MAPE ≤ 30%"
  - DESPUÉS: "MAPE ≤ 30% en observaciones con ≥ 5 casos; detección de brote 80%+"

### ✅ DONE: Fix #5

---

## 📋 Verificación de los 5 Fixes

**Después de completar los 5 fixes:**

1. [ ] Fix #1: RobustScaler eliminado → notebook corre sin errores
2. [ ] Fix #2: Feature selection sin 2024 → lista de features mostrada
3. [ ] Fix #3: Rolling con .shift(1) → features calculadas correctamente
4. [ ] Fix #4: Hipótesis actualizada → NO menciona EpiGNN/PatchTST como implementados
5. [ ] Fix #5: Tabla de métricas → 5 columnas (MAE, MAPE, SMAPE, RMSE, Brote%)

**Cuando TODO esté ✅, corre el notebook COMPLETO de punta a punta:**
```bash
# En terminal dentro del notebook
# Reproducir todo: Kernel → Restart & Run All
```

---

## Siguientes Pasos (Después de 5 Críticos)

### Fix #6: SHAP (30 min)
- Agregar após walk-forward
- Código en FIXES_CODE_SNIPPETS.md, sección FIX #6
- Genera: `shap_feature_importance_bar.png`, `shap_feature_violin.png`

### Fix #7: Remove .fillna(0) (10 min)
- Buscar: `.fillna(0)` en walk-forward (Ctrl+F)
- Remover: `X_tr[...].fillna(0)` → `X_tr[...]`

### Fixes #8-10: (2-3 horas)
- Bootstrap IC
- Análisis por estratos
- Comparativa con literatura
- Ver REVISION_CHECKLIST.md

---

## 🆘 Si Algo Sale Mal

| Error | Solución |
|-------|----------|
| "RobustScaler not defined" | Verificar que lo borraste y recargaste kernel |
| MAPE sube significativamente | Normal (leakage removido). Documentar en paper |
| Feature selection cambian muchas | Señal de inestabilidad. Documentar |
| NaN en métricas | Significa año sin observaciones >= min_casos. Revisar |
| Funciones SHAP no encuentran modelo | Verificar que `lgb_final` existe |

---

## Tiempo Estimado

```
Fix #1 (RobustScaler):           15 min ✓
Fix #2 (Feature selection):       30 min ✓
Fix #3 (Rolling shift):           15 min ✓
Fix #4 (Hipótesis):              60 min ✓
Fix #5 (Métricas):              90 min ✓
────────────────────────────────────────
TOTAL 5 CRÍTICOS:               210 min = 3.5 horas

+ Fix #6-7 (SHAP + .fillna):     40 min
+ Fixes #8-10 (Bonus):         120-180 min
────────────────────────────────────────
TOTAL COMPLETO:                 6-8 horas
```

---

## ✅ Listo para Enviar

Cuando completes los 5 fixes:
1. Corre notebook completo: Kernel → Restart & Run All
2. Verifica que NO hay errores
3. Exporta a HTML: File → Download as → HTML
4. Comprime: `tar czf dengue_revision_complete.tar.gz notebooks/ data/`
5. **LISTO PARA ENVIAR A REVISTA** 🎉

---

**¡Ahora abre el notebook y comienza con Fix #1!**

Preguntas: Revisar VISUAL_ANTES_DESPUES.md o FIXES_CODE_SNIPPETS.md
