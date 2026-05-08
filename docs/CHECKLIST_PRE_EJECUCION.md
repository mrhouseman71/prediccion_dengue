# ✅ CHECKLIST: Antes de Ejecutar el Notebook

**Fecha**: 2026-05-08  
**Objetivo**: Validar que todo está en orden antes de ejecutar Notebook_6

---

## 🔍 VERIFICACIÓN DE INSTALACIÓN

### 1. Dependencias Instaladas
```bash
pip install -r requirements.txt
```
**Versiones críticas a comprobar**:
- [ ] `lightgbm==4.0.0` (no anterior; NaN handling)
- [ ] `torch==2.0.1` (o superior; si falla pytorch-geometric, usar `pip install torch==2.0.1+cu118`)
- [ ] `shap==0.42.1` (o 0.43.1 si issues de compatibilidad)
- [ ] `pandas==2.0.3`, `numpy==1.24.3`, `scikit-learn==1.3.0` (exacto para reproducibilidad)

**Validar instalación**:
```python
import pandas as pd
import numpy as np
import lightgbm as lgb
import torch
import shap
print(f"LightGBM: {lgb.__version__}")
print(f"Torch: {torch.__version__}")
print(f"SHAP: {shap.__version__}")
print("✅ Todas las librerías OK" if (
    lgb.__version__.startswith("4.0") and 
    torch.__version__.startswith("2.0") and
    shap.__version__.startswith("0.4")
) else "❌ Versiones incompatibles")
```

---

## 📁 ESTRUCTURA DE ARCHIVOS

- [ ] `/workspaces/prediccion_dengue/notebooks/Notebook_6_model_training_iter2.ipynb` — **existe**
- [ ] `/workspaces/prediccion_dengue/requirements.txt` — **versiones fijas**
- [ ] `/workspaces/prediccion_dengue/data/` — **carpeta con CSVs**
  - [ ] `dengue_dataset_limpio.csv` (dataset principal SNVS)
  - [ ] `clima_nasa_power_2018_2025.csv` (features climáticas)
- [ ] `/workspaces/prediccion_dengue/docs/` — documentación
  - [ ] `IMPLEMENTACION_MEJORAS_PUBLICATION_READY.md` (acabamos de crear)

---

## 🔧 CONFIGURACIÓN DEL NOTEBOOK

### Rutas y Output
- [ ] Verificar en celda 1 que `OUT_DIR` existe (buscar línea: `OUT_DIR = ...` u `os.makedirs(OUT_DIR, exist_ok=True)`)
- [ ] Si falta: agregar `OUT_DIR = './output_publication'` antes de SHAP plots

### Dataset Principal
- [ ] Buscar en notebook: línea que carga `dengue_dataset_limpio.csv`
- [ ] Confirmación: `df.shape` debe ser ~(>1000, ~20 cols)
- [ ] Confirmación: columnas incluyen `cantidad_casos` (target), departamentos, año, SE, etc.

### Features Climáticas
- [ ] Línea que carga `clima_nasa_power_2018_2025.csv`
- [ ] Confirmación: lags 2, 4, 6 semanas presentes (columnas: `TEMP_lag2`, `TEMP_lag4`, `PREC_lag2`, etc.)

---

## 📊 CELDAS A EJECUTAR (ORDEN)

**Fase 1: Setup + Data Loading** (celdas 1-10)
- [ ] Ejecutar celda 1: imports, setup paths, `OUT_DIR` creada
- [ ] Ejecutar celda 2-5: load datasets, validación shapes, descripción básica
- **Esperado**: Ningún error; outputs muestran n_rows=2000+, features=20+

**Fase 2: Feature Engineering** (celdas 11-40)
- [ ] Ejecutar todo el feature engineering (lags, escalado, encoder categórico)
- [ ] Saltar si genera errores en transformers (usar fill method = "forward_fill" si falla interpolación)

**Fase 3: Walk-Forward Training** (celdas 41-70)
- [ ] Ejecutar walk-forward loop 2020-2024
- **Esperado**: Output para cada año con MAPE, MAE, RMSE
- **Marca warning**: Las primeras líneas (2019-early 2020) tendrán accuracia terrible (normal para modelo recién entrenado, 6 meses de datos)

**Fase 4: NUEVAS CELDAS POST-PUBLICATION** (celdas 71-81)
- [ ] **Celda 71**: Bootstrap IC — esperar ~2-3 segundos (1000 resamples = lento pero normal)
- [ ] **Celda 72**: Análisis regional — salida: tabla con 5-6 regiones
- [ ] **Celda 73**: SHAP explainability — genera PNG files; esperar ~5-10 segundos
- [ ] **Celda 74**: Tabla final comparación — resumen limpio
- [ ] **Celda 75**: Residuos — 2 plots interactivos
- [ ] **Celda 76**: Sensibilidad min_casos — tabla robustez
- [ ] **Celda 77**: Mapa coroplético — barplot MAPE por depto
- [ ] **Celdas 78-81**: Markdown de contexto (automático, no requiere ejecución)

---

## 🎯 ERRORES ESPERADOS vs CRÍTICOS

### ✅ Errores NORMALES (no detener):
- `FutureWarning` de pandas (deprecated tipo_encoding)
- `sklearn` warnings de versión
- Primer año walk-forward (2020) MAPE > 100% — **ESPERADO** (poco data de entrenamiento)
- SHAP plot si tiene NaN en features — agregar `.fillna(0)` a X_test_np si ocurre

### ❌ Errores CRÍTICOS (detener e investigar):
- `ModuleNotFoundError: lightgbm` → pip install faltó
- `TypeError: 'numpy.ndarray' in LightGBM` → versión LightGBM <4.0, necesita downgrade
- `KeyError: 'temperatura'` → archivo CSV no cargado correctamente
- `IndexError` en residuos analysis → longitud desigual y_test vs y_pred (versión anterior quebrada)

### 📌 Si falla alguna celda nueva (71-81):
1. Verificar que `y_test` y `y_pred_lgb` existen (generadas en celdas 60-70)
2. Verificar que `X_test_np` existe (features para SHAP)
3. Si `bootstrap_mape_ci` error: asegurar `from sklearn.utils import resample` está en imports celda 1

---

## 📈 OUTPUTS ESPERADOS (POST-EJECUCIÓN)

### Archivos generados:
```
output_publication/
├── shap_summary_bar.png        [SHAP: importancia global de features]
├── shap_beeswarm.png           [SHAP: per-observation contributions]
├── residuos_vs_se.png          [Residuals time series]
└── residuos_por_depto.png      [Residuals boxplot by department]
```

### Tablas impresas (console output):
1. **Bootstrap IC Walk-Forward**: Año | MAPE Baseline ± IC | MAPE LightGBM ± IC | RMSE
2. **Regional Analysis**: Región | n_eval | MAE | MAPE≥5% | SMAPE% | Incidencia Media
3. **SHAP Top-10**: feature_name | mean(|shap_value|) | impact
4. **Final Comparison**: Modelo | MAE | MAPE≥5% | SMAPE% | RMSE | IC 95%
5. **Residuals Stats**: mean, median, SD, percentiles, test Shapiro-Wilk p-value
6. **Sensitivity**: min_casos | Baseline MAPE | LightGBM MAPE | Delta
7. **Department MAPE**: Departamento | MAPE | Count (n=24 deptos)

---

## 🚀 PRÓXIMOS PASOS (POST-EJECUCIÓN)

### Validación Inmediata
1. Revisar SHAP plots: ¿Features esperados tienen alto SHAP? (temperatura, precipitación, incidencia anterior)
2. Revisar residuos: ¿p-value Shapiro-Wilk > 0.05? (si dice "NO son normales", es OK para conteos epidemiológicos)
3. Revisar regiones: ¿NEA tiene MAPE mejor que Patagonia? (validar hipótesis geográfica)

### Para Submissions a Revista
1. Descargar PNG files → Figure 1, 2, 3, 4 en paper
2. Copiar Tabla Final → Table 2 (Resultados)
3. Copiar Regional → Supplementary Table S1
4. Mencionar bootstrap IC en Métodos (muestra rigor estadístico)
5. Mencionar SHAP en Resultados (demuestra interpretabilidad)

### Reproducibilidad
1. Commit a GitHub: `git add . && git commit -m "feat: add publication-ready analysis cells + bootstrap IC + SHAP explainability"`
2. Especificar en README: "Reproducir: `pip install -r requirements.txt && jupyter notebook notebooks/Notebook_6_model_training_iter2.ipynb`"
3. Notificar coauthors: "Notebook ahora tiene N= 11 nuevas celdas; ejecutar en orden, total ~45 minutos"

---

## ⏱️ TIEMPO ESTIMADO DE EJECUCIÓN

| Fase | Celdas | Tiempo Est. | Notas |
|------|--------|-------------|-------|
| Setup + Load | 1-10 | 2-3 seg | Rápido, I/O CSV |
| Feature Eng | 11-40 | 5-10 seg | Escalado, encoding |
| Walk-Forward | 41-70 | 20-30 seg | Entrenamiento 5 años ×3 modelos |
| **NEW: Bootstrap IC** | 71 | 3-5 seg | 1000 resamples |
| **NEW: Regional** | 72 | 1-2 seg | Groupby + stats |
| **NEW: SHAP** | 73 | 10-15 seg | TreeExplainer compute |
| **NEW: Table** | 74 | <1 seg | Formatting |
| **NEW: Residuals** | 75 | 2-3 seg | Plotting + stats |
| **NEW: Sensitivity** | 76 | 2-3 seg | Compute MAPE 4×thresholds |
| **NEW: Map** | 77 | 1-2 seg | Barplot |
| **Markdown** | 78-81 | 0 seg | No compute |
| **TOTAL** | | **~50-70 seg** | Menos de 2 minutos |

---

## 🆘 SOPORTE DIAGNÓSTICO

Si hay error en celda X:
1. Revisar output previa celda (Y) — ¿variables necesarias existen?
2. Copiar error completo → buscar en línea o GitHub issues
3. Revisar versión LightGBM: `pip show lightgbm` (debe ser 4.0.0 exacto)
4. Si .fillna error: revertir celda 10 de walk-forward, o pre-llenar antes de loop

---

## ✨ CHECKPOINT: TODO LISTO?

Responder sí a todas:
- [ ] ¿`pip list` muestra lightgbm==4.0.0?
- [ ] ¿Archivo CSV `/data/dengue_dataset_limpio.csv` existe?
- [ ] ¿Archivo CSV `/data/clima_nasa_power_2018_2025.csv` existe?
- [ ] ¿Jupyter/IPython puede ejecutar notebooks?
- [ ] ¿`OUT_DIR` será creado automáticamente en celda 1?

**Si todas pasan**: ✅ Proceder a ejecutar Notebook_6 de inicio a fin.

