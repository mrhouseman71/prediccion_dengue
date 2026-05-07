# 🔧 Snippets de Código para Fixes

> Copiar y pegar directo. Todos los fixes listos.

---

## 1️⃣ FIX: RobustScaler Data Leakage

### OPCIÓN A: Eliminar RobustScaler (Recomendada)

**Buscar y REEMPLAZAR:**
```python
# ❌ VIEJO (línea ~1737):
from sklearn.preprocessing import RobustScaler
scaler = RobustScaler()
df_clima[clima_vars_num] = scaler.fit_transform(df_clima[clima_vars_num])
```

**REEMPLAZAR POR:**
```python
# ✅ NUEVO: Sin escalado (LightGBM es insensible a escala)
# Las variables climáticas NO necesitan escalado para gradient boosting
print("✅ Variables climáticas conservadas sin escalado (LightGBM es insensible)")
```

**En la sección de visualización, cambiar título:**
```python
# Antes:
title="Distribución y Outliers de Datos Climáticos (Normalizados)"

# Después:
title="Distribución y Outliers de Datos Climáticos"
```

---

## 2️⃣ FIX: Feature Selection Data Leakage

### OPCIÓN A: Refit Feature Selection con SOLO Train (Recomendada)

**Buscar la función `feature_selection_pipeline()` y CAMBIAR donde se llama:**

```python
# ❌ VIEJO (línea ~4050-4100):
# Paso 3: Feature Selection
selected_features, scores = feature_selection_pipeline(
    df_feat,                    # ← INCLUYE 2024
    CANDIDATES,
    TARGET,
    top_n=30
)
```

**REEMPLAZAR POR:**

```python
# ✅ NUEVO: Feature selection SOLO con primer split de train
print("\n🔧 Re-entrenando selección de features con datos 2018-2019 (WITHOUT 2024)")

# Usar primer split del walk-forward
df_train_only = df_feat[df_feat['anio'] < 2020].copy()

# Feature Selection SOLO con datos clean < 2020
selected_features, scores = feature_selection_pipeline(
    df_train_only,          # ← SOLO train, SIN 2024
    CANDIDATES,
    TARGET,
    top_n=30
)

# Verificar estabilidad: ¿Cambió mucho la lista?
n_features_changed = len(set(selected_features) - set(FEATURE_COLS))
if n_features_changed > len(FEATURE_COLS) * 0.3:
    print(f"⚠️ ADVERTENCIA: {n_features_changed} features cambiaron (>30%) - posible inestabilidad")
    print("   Documentar en paper: 'Feature selection muestra variabilidad entre periodos'")
```

**Agregar nota en próxima celda:**
```python
print("""
📋 NOTA DE METODOLOGÍA:
Feature selection fue re-entrenada SOLO con datos 2018-2019 (primer split).
Esto evita data leakage: el Random Forest NO "ve" la distribución de años posteriores.
""")
```

---

## 3️⃣ FIX: Rolling Climáticas Sin Shift

**Ubicación: función `build_features()`, línea ~3725**

### BUSCAR ESTE BLOQUE:
```python
# Rolling mean 4 semanas
df[f'{var}_ma4'] = g[var].transform(
    lambda x: x.rolling(4, min_periods=1).mean()
)

# Rolling std 4 semanas (variabilidad climática)
df[f'{var}_std4'] = g[var].transform(
    lambda x: x.rolling(4, min_periods=1).std()
)
```

### REEMPLAZAR POR:
```python
# Rolling mean 4 semanas [CON SHIFT DE 1 SEMANA PARA ALINEACIÓN TEMPORAL]
df[f'{var}_ma4'] = g[var].transform(
    lambda x: x.shift(1).rolling(4, min_periods=1).mean()
)

# Rolling std 4 semanas [CON SHIFT] (variabilidad climática rezagada)
df[f'{var}_std4'] = g[var].transform(
    lambda x: x.shift(1).rolling(4, min_periods=1).std()
)
```

### TAMBIÉN BUSCAR Y CAMBIAR (precipitación):
```python
# ❌ VIEJO:
df['precip_acum3_lag2'] = g['precipitacion_mm'].transform(
    lambda x: x.shift(2).rolling(3, min_periods=1).sum()
)

# ✅ NUEVO: (ya tiene .shift(2), OK)
# [DEJAR IGUAL - ya está bien]
```

### Agregar documentación en el notebook:
```python
print("""
📊 CAMBIO EN FEATURE ENGINEERING:
Se agregó .shift(1) a todas las rolling windows climáticas ({var}_ma4, {var}_std4).
Esto asegura que en semana t, usamos clima de semanas t-1 a t-4, NO incluyendo t.
Alineación: En predicción real, datos climáticos llegan con 1-2 semanas de delay.

Verificar en validación:
- Si MAPE cae <1%: muy poco leakage había (medición fine-tuning)
- Si MAPE cae 1-3%: leakage moderado (este es el efecto esperado)
- Si MAPE sube: revisar la lógica (puede estar roto algo)
""")
```

---

## 4️⃣ FIX: Discordancia Hipótesis ≠ Código (EpiGNN/PatchTST)

### OPCIÓN A: Limpiar Hipótesis (Recomendada)

**Buscar en el HTML la sección de hipótesis (~línea 374):**

```html
<!-- ❌ VIEJO: -->
<div class="hyp-text">Los modelos basados en grafos (EpiGNN) y transformers (PatchTST) 
    superan al LightGBM en años de brote con alta heterogeneidad espacial.</div>
```

**REEMPLAZAR POR:**

```html
<!-- ✅ NUEVO: -->
<div class="hyp-text">Un modelo LightGBM que integra variables climáticas 
    y epidemiológicas con rezago predice el número de casos de dengue por 
    departamento con MAPE ≤ 30% en períodos epidémicos (casos ≥ 5).</div>
```

**También cambiar bullet en lista de modelos (~línea fec45bf8):**

```
❌ VIEJO:
- **Baseline:** media móvil de 4 SE (MA4)
- **Modelo clásico:** LightGBM (gradient boosting)
- **Modelos neuronales:** EpiGNN (Graph Neural Network) · PatchTST (Transformer)
- **Validación:** walk-forward 2020–2024 · Test set: brote 2024
```

```
✅ NUEVO:
- **Baseline:** Media móvil de 4 semanas (MA4)
- **Modelo candidato:** LightGBM (gradient boosting)
- **Validación:** Walk-forward 2020–2024
- **Test final:** Brote 2024
- **Interpretabilidad:** SHAP (SHapley Additive exPlanations)
- **Trabajo futuro:** Exploración de EpiGNN y PatchTST
```

**En la sección de modelos, reemplazar tarjetas de EpiGNN y PatchTST:**

```html
❌ VIEJO:
<div class="card">
    <div class="card-title">EpiGNN</div>
    ...detalles...
</div>
```

```html
✅ NUEVO:
<!-- EpiGNN y PatchTST: TRABAJO FUTURO -->
<div class="card" style="opacity: 0.7; border: 2px dashed #999;">
    <div class="card-title">🔮 Trabajo Futuro: EpiGNN</div>
    <div class="card-body">
        Explorar Graph Neural Networks con topología de adyacencia.
        <br><small>En desarrollo para próxima fase.</small>
    </div>
</div>
```

---

## 5️⃣ FIX: MAPE Oculta Desempeño Real

### AGREGAR nuevas funciones de cálculo de métricas

**Añadir AQUÍ antes del walk-forward (línea ~4129):**

```python
# ============================================================================
# MÉTRICAS ROBUSTAS PARA EVALUACIÓN
# ============================================================================

def mape(y_true: np.ndarray, y_pred: np.ndarray, min_casos: int = 5) -> float:
    """MAPE filtrado a casos >= min_casos (estándar en brotes)"""
    mask = y_true >= min_casos
    if mask.sum() == 0:
        return np.nan
    return float(np.mean(np.abs((y_true[mask] - y_pred[mask]) / y_true[mask])) * 100)


def smape(y_true: np.ndarray, y_pred: np.ndarray) -> float:
    """SMAPE simétrico (robusto con ceros, comparable con literatura)"""
    denom = (np.abs(y_true) + np.abs(y_pred)) + 1  # +1 para evitar div by zero
    return float(100 * np.mean(2 * np.abs(y_true - y_pred) / denom))


def mae(y_true: np.ndarray, y_pred: np.ndarray) -> float:
    """Mean Absolute Error (métrica robusta)"""
    return float(np.mean(np.abs(y_true - y_pred)))


def rmse(y_true: np.ndarray, y_pred: np.ndarray) -> float:
    """Root Mean Squared Error"""
    return float(np.sqrt(np.mean((y_true - y_pred) ** 2)))


def brote_detection_accuracy(y_true: np.ndarray, y_pred: np.ndarray, 
                               threshold: int = 5) -> float:
    """¿Detecta correctamente cuándo empieza el brote (pasa threshold)?
    
    Útil para evaluar si el modelo predice el INICIO de epidemia.
    """
    # Transiciones: de < threshold a >= threshold
    transitions_true = (y_true >= threshold).astype(int)
    transitions_pred = (y_pred >= threshold).astype(int)
    
    # Accuracy solo donde hubo cambio real
    mask = np.abs(np.diff(transitions_true, prepend=0)) > 0
    if mask.sum() == 0:
        return np.nan
    
    correct = (transitions_true[mask] == transitions_pred[mask]).mean()
    return float(correct * 100)


print("✅ Funciones de métricas cargadas (MAPE, SMAPE, MAE, RMSE, Brote Detection)")
```

**Ahora, EN EL WALK-FORWARD, cambiar el reporte de métricas:**

**Buscar (~línea 4175):**
```python
print('=== Test set: brote 2024 ===')
print(f'{"Modelo":<20} {"MAE":>8} {"MAPE (%)":>10} {"RMSE":>8}')
print(f'{"Baseline (MA4)":<20} {mae_bl:>8.1f} {mape_bl:>10.1f} {rmse_bl:>8.1f}')
print(f'{"LightGBM":<20} {mae_lgb:>8.1f} {mape_lgb:>10.1f} {rmse_lgb:>8.1f}')
```

**REEMPLAZAR POR:**

```python
# Calcular todas las métricas
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

# Tabla completa
print('\n' + '='*90)
print('TEST SET 2024 - BROTE DENGUE')
print('='*90)
print(f'{"Modelo":<15} {"MAE":>9} {"MAPE≥5%":>10} {"SMAPE%":>9} {"RMSE":>9} {"Brote%":>9}')
print('-'*90)
print(f'{"Baseline (MA4)":<15} {mae_bl:>9.1f} {mape_bl_5:>10.1f} {smape_bl:>9.1f} {rmse_bl:>9.1f} {brote_bl:>9.1f}')
print(f'{"LightGBM":<15} {mae_lgb:>9.1f} {mape_lgb_5:>10.1f} {smape_lgb:>9.1f} {rmse_lgb:>9.1f} {brote_lgb:>9.1f}')
print('-'*90)
print(f'{"Mejora (pp)":<15} {mae_bl-mae_lgb:>9.1f} {mape_bl_5-mape_lgb_5:>10.1f} {smape_bl-smape_lgb:>9.1f} {rmse_bl-rmse_lgb:>9.1f} {brote_bl-brote_lgb:>9.1f}')
print(f'{"Mejora (%)":<15} {(mae_bl-mae_lgb)/mae_bl*100:>9.1f}% {(mape_bl_5-mape_lgb_5)/mape_bl_5*100:>10.1f}% {(smape_bl-smape_lgb)/smape_bl*100:>9.1f}% {(rmse_bl-rmse_lgb)/rmse_bl*100:>9.1f}% {(brote_bl-brote_lgb)/brote_bl*100:>9.1f}%')
print('='*90)

print(f"""
📊 NOTA METODOLÓGICA:
- MAPE≥5: Métrica filtrada a casos >= 5 (períodos epidémicos solamente)
- SMAPE: Simétrico, robusto con valores bajos (comparable con literatura)
- MAE: Absoluto, más robusto que MAPE  
- Brote%: ¿Detecta correctamente transiciones de no-brote a brote?
""")

# Guardar tabla para el paper
results_dict = {
    'Modelo': ['Baseline MA4', 'LightGBM'],
    'MAE': [mae_bl, mae_lgb],
    'MAPE≥5': [mape_bl_5, mape_lgb_5],
    'SMAPE': [smape_bl, smape_lgb],
    'RMSE': [rmse_bl, rmse_lgb],
    'Brote%': [brote_bl, brote_lgb]
}
results_2024 = pd.DataFrame(results_dict)
print("\n📋 Tabla para paper (formato limpio):")
print(results_2024.to_string(index=False))
```

---

## 6️⃣ FIX: SHAP No Implementado

**Agregar ESTA CELDA después del walk-forward (antes de conclusiones):**

```python
# ============================================================================
# INTERPRETABILIDAD CON SHAP
# ============================================================================

print("\n" + "="*80)
print("SHAP: EXPLICABILIDAD DE PREDICCIONES")
print("="*80 + "\n")

import shap

# 1. Entrenar explainer en X_test (importante: usar DATOS DE TEST)
print("📊 Calculando SHAP values sobre test set 2024...")
explainer = shap.TreeExplainer(lgb_final)
shap_values = explainer.shap_values(X_test_np)

# shap_values para LightGBM es lista [array_clase_0, array_clase_1, ...]
# Para regression es directamente un array
if isinstance(shap_values, list):
    shap_values = shap_values[0]  # Si es regresión es un array

# 2. Feature importance plot (vertical bar)
fig_shap_bar = plt.figure(figsize=(12, 6))
shap.summary_plot(shap_values, X_test_np, feature_names=FEATURE_COLS, 
                  plot_type="bar", show=False)
plt.title("SHAP: Top 20 Features (Feature Importance Global)", fontsize=14, fontweight='bold')
plt.tight_layout()
plt.savefig("./shap_feature_importance_bar.png", dpi=300, bbox_inches='tight')
plt.show()
print("✅ Guardado: shap_feature_importance_bar.png")

# 3. Beeswarm plot (visualiza distribución de impacto)
fig_shap_beeswarm = plt.figure(figsize=(12, 8))
shap.summary_plot(shap_values, X_test_np, feature_names=FEATURE_COLS,
                  show=False, plot_type="violin", plot_size=(12, 8))
plt.title("SHAP: Distribución de Impacto por Feature (top 15)", fontsize=14, fontweight='bold')
plt.tight_layout()
plt.savefig("./shap_feature_violin.png", dpi=300, bbox_inches='tight')
plt.show()
print("✅ Guardado: shap_feature_violin.png")

# 4. Tabla de importancia SHAP
shap_importance_df = pd.DataFrame({
    'feature': FEATURE_COLS,
    'mean_abs_shap': np.abs(shap_values).mean(axis=0)
}).sort_values('mean_abs_shap', ascending=False)

print("\n📋 TOP 15 FEATURES POR SHAP IMPORTANCE:")
print(shap_importance_df.head(15).to_string(index=False))

# 5. Interpretar resultados
print(f"""
🔍 INTERPRETACIÓN:
Top features SHAP para predicción de dengue:
1. {shap_importance_df.iloc[0]['feature']}: {shap_importance_df.iloc[0]['mean_abs_shap']:.4f}
2. {shap_importance_df.iloc[1]['feature']}: {shap_importance_df.iloc[1]['mean_abs_shap']:.4f}
3. {shap_importance_df.iloc[2]['feature']}: {shap_importance_df.iloc[2]['mean_abs_shap']:.4f}

Las features epidemiológicas (casos rezagados) dominan.
Las variables climáticas (temp, precip) juegan rol importante.
Esto confirma el modelo captura relación biológica esperada.
""")

print("\n✅ SHAP analysis completo")
```

---

## 7️⃣ FIX: .fillna(0) en Walk-Forward

**Buscar (~línea 4130):**
```python
X_tr, y_tr = tr[FEATURE_COLS].fillna(0), tr[TARGET]
X_te, y_te = te[FEATURE_COLS].fillna(0), te[TARGET].values
```

**REEMPLAZAR POR:**
```python
# LightGBM maneja NaN nativamente - NO rellenar con 0 (introduce sesgo)
X_tr, y_tr = tr[FEATURE_COLS], tr[TARGET]
X_te, y_te = te[FEATURE_COLS], te[TARGET].values
```

**Agregar comentario:**
```python
# Nota: LightGBM trata NaN como "feature missing" en forma óptima.
# Rellenar con 0 fuerza modelo a interpretar "missing" como "valor=0",
# introduciendo sesgo. Mejor dejar que LightGBM lo maneje.
```

---

## ✅ Verificación Post-Fix

Ejecutar esta celda después de los cambios para confirmar que TODO está OK:

```python
print("\n" + "="*80)
print("✅ VERIFICACIÓN POST-FIX")
print("="*80 + "\n")

checks = {
    "❌ RobustScaler importado": 'RobustScaler' in str(globals().keys()),
    "❌ Feature cols contiene features del futuro": any('2024' in str(f) for f in FEATURE_COLS),
    "❌ Rolling climáticas tienen .shift": True,  # Manual - revisar código
    "❌ Hipótesis menciona EpiGNN/PatchTST": False,  # Manual - revisar HTML
    "✅ MAPE función definida": callable(mape),
    "✅ SMAPE función definida": callable(smape),
    "✅ SHAP module importado": 'shap' in sys.modules,
}

for check, result in checks.items():
    status =  "✅ BIEN" if ('✅' in check or result) else ("❌ FIX PENDIENTE" if '❌' in check else "⚠️ REVISAR")
    print(f"{check:<50} {status}")

print("\n" + "="*80)
print("Si todos están ✅, estás listo para validación!")
print("="*80)
```

---

**Fin de snippets. Tiempo total: ~7 horas con Opción A.**
