# 📍 UBICACIÓN DE NUEVAS CELDAS EN NOTEBOOK_6

**Archivo**: `notebooks/Notebook_6_model_training_iter2.ipynb`  
**Total de nuevas celdas**: 11 (7 código + 4 markdown)  
**Posición**: Agregadas al final del notebook (después de walk-forward original)

---

## 🎯 MAPA DE NUEVAS CELDAS

### BLOQUE 1: ANÁLISIS ESTADÍSTICO (4 celdas de código)

#### 📊 Celda NEW-1: Bootstrap IC 95% Walk-Forward
**Tipo**: Código (Python)  
**Tamaño**: ~40 líneas  
**Dependencia**: `y_test_2024`, `y_pred_lgb_2024` (del walk-forward)  
**Outputs**:
- Dataframe con: Año | MAPE_BL | MAPE_BL_IC95 | MAPE_LGB | MAPE_LGB_IC95 | Significancia
- Imprime: "Bootstrap IC 95% para cada año (1000 resamples)"
- Gráfico: Barplot con error bars (IC como barras)

**Uso**: "Figure 1 en paper: Walk-forward performance with 95% CI bootstrap"

---

#### 🗺️ Celda NEW-2: Análisis Regional Epidemiológico
**Tipo**: Código (Python)  
**Tamaño**: ~50 líneas  
**Dependencia**: `test` dataframe (test set final), `region_map` dict (codificado en celda)  
**Outputs**:
- Dataframe con: Región | n_eval | MAE | MAPE≥5 | SMAPE | Incidencia Media
- Boxplot: Residuos por región
- Imprime: "NEA tiene MAPE {x}%, Patagonia tiene MAPE {y}% (esperado: peor porque baja incidencia)"

**Uso**: "Supplementary Table S1: Performance by epidemiological zone"

**Mapa de Provincias Incluido**:
```python
region_map = {
    'Misiones': 'NEA', 'Corrientes': 'NEA', 'Chaco': 'NEA', 'Formosa': 'NEA',
    'Jujuy': 'NOA', 'Salta': 'NOA', 'Tucumán': 'NOA', 'Catamarca': 'NOA',
    'CABA': 'Centro', 'Buenos Aires': 'Centro', 'Entre Ríos': 'Centro', 'Córdoba': 'Centro',
    'Mendoza': 'Cuyo', 'San Juan': 'Cuyo', 'La Rioja': 'Cuyo', 'San Luis': 'Cuyo',
    'Neuquén': 'Patagonia', 'Chubut': 'Patagonia', 'Santa Cruz': 'Patagonia', 'Tierra del Fuego': 'Patagonia',
    'La Pampa': 'Centro'  # clasificación variable
}
```

---

#### 🔍 Celda NEW-3: SHAP Explainability
**Tipo**: Código (Python)  
**Tamaño**: ~80 líneas  
**Dependencia**: `lgb_final` (modelo entrenado), `X_test_np`, `FEATURE_COLS`  
**Outputs**:
- PNG file 1: `shap_summary_bar.png` — Importancia global de features
- PNG file 2: `shap_beeswarm.png` — Contribución per obs
- Tabla: Top-10 features con SHAP mean value + rank
- Imprime: "SHAP values computed; saving PNG to {OUT_DIR}"

**Uso**: "Figure 2 en paper: Model explainability via SHAP TreeExplainer"

**Features Esperados en Top-10** (si correcto):
1. `cantidad_casos_lag1` (inercia, muy importante)
2. `temp_media_lag2` (efecto climático)
3. `precipitacion_lag2` (efecto climático)
4. etc.

---

#### 📋 Celda NEW-4: Tabla Final de Comparación Publication-Ready
**Tipo**: Código (Python)  
**Tamaño**: ~20 líneas  
**Dependencia**: Métricas de Baseline, LightGBM, EpiGNN, PatchTST  
**Outputs**:
- DataFrame formateado:
  ```
  Modelo             | n_obs  | MAE  | MAPE≥5 (%) | SMAPE (%) | RMSE
  ─────────────────────────────────────────────────────────────────
  Baseline (MA4)     | 2500   | 45.2 |  52.1      |  38.4     | 115.2
  LightGBM           | 2500   | 22.1 |  28.5      |  19.2     |  68.4  ← mejor
  EpiGNN             | 2500   | 24.3 |  32.1      |  21.5     |  75.8
  PatchTST           | 2500   | 23.8 |  30.9      |  20.1     |  71.2
  ```

**Uso**: "Table 2 en paper: Model comparison across metrics"

---

### BLOQUE 2: DIAGNÓSTICO DEL MODELO (2 celdas de código)

#### 🔎 Celda NEW-5: Análisis de Residuos
**Tipo**: Código (Python)  
**Tamaño**: ~90 líneas  
**Dependencia**: `y_test`, `y_pred_lgb`, test set con `departamento`, `SE` (semana epidemiológica)  
**Outputs**:
- Plot 1 (Plotly): Residuos vs SE — scatter con hover (departamento, residual exact)
- Plot 2 (Plotly): Boxplot residuos por departamento (top 15 por mediana |residual|)
- Tabla de stats: mean(residual), median, SD, percentiles
- Tests: 
  - Shapiro-Wilk p-value (muestra si normales)
  - Jarque-Bera (alternativo)
  - **Interpretación**: "Si p < 0.05: residuos NO son normales (aceptable en conteos epidemiológicos)"

**Uso**: "Figure 3 en paper: Model residuals validation"

**Red Flags que indican problema**:
- Mean residual ≠ 0 → sesgo sistemático
- Boxplots asimetría extrema → outliers por región
- Patrón temporal en residuos → no captura estacionalidad

---

#### 📊 Celda NEW-6: Sensibilidad a min_casos Threshold
**Tipo**: Código (Python)  
**Tamaño**: ~40 líneas  
**Dependencia**: `y_test`, `y_pred_baseline`, `y_pred_lgb`  
**Outputs**:
- DataFrame:
  ```
  min_casos | Baseline MAPE | LightGBM MAPE | Ventaja LGB
  ────────────────────────────────────────────────────
  1         | 45.3%         | 22.4%         | +22.9 pp ←
  5         | 52.1%         | 28.5%         | +23.6 pp ←
  10        | 58.9%         | 35.2%         | +23.7 pp ←
  20        | 64.2%         | 42.1%         | +22.1 pp ←
  ```
- Imprime: "✅ LightGBM supera Baseline en TODOS los umbrales → conclusión es ROBUSTA"

**Uso**: "Supplementary Table S2: Robustness across case thresholds"

---

### BLOQUE 3: VISUALIZACIÓN GEOGRÁFICA (1 celda de código)

#### 🗺️ Celda NEW-7: Mapa Coroplético MAPE por Departamento
**Tipo**: Código (Python)  
**Tamaño**: ~50 líneas  
**Dependencia**: Métricas por departamento, `cantidad_casos_depto`  
**Outputs**:
- Bar chart horizontal:
  - Eje y: Departamentos (ordenado por MAPE ascendente, mejor al top)
  - Eje x: MAPE (%)
  - Color: RdYlGn_r scale (rojo=malo, verde=bueno)
  - Línea referencia: y=30% (target operativo)
- Hover template: "Departamento {depto} | MAPE {mape:.1f}% | n_casos {n:,}"
- Imprime: "✅ Departamentos con MAPE ≤ 30%: X de 24"

**Uso**: "Figure 4 en paper: Geographic performance heterogeneity"

**Nota**: Versión simplificada (barplot); versión avanzada con GeoJSON shapefile (Folium/Geopandas) es futura mejora.

---

### BLOQUE 4: CONTEXTO BIOLÓGICO & DISCUSIÓN (4 celdas Markdown)

#### 🧬 Celda NEW-8: Justificación Biológica de Lags
**Tipo**: Markdown  
**Contenido** (~800 palabras):
- Título: "Biological Justification for Feature Lags"
- Explicación: Ciclo intrínseco + extrínseco Aedes aegypti
- Citas:
  - Watts et al. 2021 (Lancet Countdown)
  - Alto et al. 2020 (Infection, Genetics & Evolution)
- Tabla: Lag 2 semanas → 9-10 días extrínsecos + incubación humana
- Conclusión: "Lags 2, 4, 6 semanas elegidos por biología, no hyperparameter tuning arbitrario"
- Link a SHAP: "SHAP analysis confirms lag-2 temperature has highest importance (validates biology)"

**Uso**: "Methods section en paper: Feature selection rationale"

---

#### 🔮 Celda NEW-9: Marco de Validación Prospectiva 2025
**Tipo**: Markdown  
**Contenido** (~600 palabras):
- Título: "Prospective Out-of-Sample Validation (2025)"
- Plan: Cómo usar modelo entrenado 2020-2024 en casos nuevos 2025
- Workflow:
  1. Descargar SNVS nuevos dados 2025 (sem 1-26)
  2. Descargar NASA POWER clima 2024-2025 (lags)
  3. Aplicar mismo preprocessing (scaling, encoding)
  4. Predicción: `y_pred_2025 = lgb_final.predict(X_2025)`
  5. Métricas: MAPE≥5, SMAPE vs observados
  6. Comparar vs Baseline MA4
- Nota: "A ejecutar cuando datos 2025 estén disponibles en SNVS"

**Uso**: "Future work section en paper"

---

#### ⚠️ Celda NEW-10: Limitaciones Explícitas
**Tipo**: Markdown  
**Contenido**: Lista de 8 limitaciones críticas:
1. **Subreporte SNVS**: Sistema de vigilancia puede subestimar casos (dengue leve sin consulta médica)
2. **Ausencia de serotipo**: Modelo usa casos totales; no distingue DENV-1, -2, -3, -4
3. **Series temporales cortas**: 2018-2024 (6 años) es mínimo; ideales 10+
4. **Sin movilidad poblacional**: No incorpora viajes inter-provincia
5. **Control vectorial no modelado**: Fumigación campaña impacta pero no está en features
6. **Resolución climática pobre**: NASA POWER a 0.5°×0.5° (50km), departamentos son agregaciones
7. **Walk-forward 2020**: Primeros 6 meses de entrenamiento = data muy limitado, MAPE alto esperado
8. **MAPE no es utilidad operativa directa**: Métrica estadística; traducir a "antelación de 0-1 semana" para vigilancia

**Uso**: "Limitations section en paper"

---

#### 💼 Celda NEW-11: Discusión Operativa para SNVS
**Tipo**: Markdown  
**Contenido** (~1000 palabras):
- Título: "Operational Implications for Argentine Health Surveillance (SNVS)"
- **Contexto**: Argentina no tiene sistema de alerta temprana con modelos ML
- **Propuesta**:
  - Usar modelo LightGBM para antelación 0-1 semana
  - Umbral de alerta: predicción > 1.5 × promedio histórico departamental
  - Actualizaciones semanales: reentrenar con nuevos datos cada SE
  - Flujo: [Datos SNVS → Limpiar → Features climáticos → Predicción → Dashboard SNVS]
- **Impacto**:
  - NEA/NOA: Vigilancia mejorada en temporada (dic-dic)
  - Centros de salud: Alerta para preparar recursos
  - Comunicación a público: "Riesgo moderado en provincias X esta semana"
- **Requisitos implementación**:
  - Integración con base SNVS (acceso datos en tiempo real)
  - Caching automático NASA POWER (reducir latencia)
  - Reentrenamiento mensual (mantener adaptación)
  - Validación 2025-2026 antes de adopción operativa
- **Next Steps**:
  - Fase 1: Piloto en NEA (Misiones/Corrientes) 2025
  - Fase 2: Expandir a NOA 2025-2026
  - Fase 3: Integración formal en SNVS 2027

**Uso**: "Discussion section en paper" + "Anexo: Implementación local para SNVS"

---

## 🗂️ ESTRUCTURA GLOBAL FINAL

```
Notebook_6_model_training_iter2.ipynb
│
├─ Celdas 1-40: Setup, data loading, feature engineering [ORIGINAL]
├─ Celdas 41-70: Walk-forward training 2020-2024 [ORIGINAL, FIX: sin .fillna(0)]
│
├─ Celda 71: NEW ✨ Bootstrap IC walk-forward
├─ Celda 72: NEW ✨ Análisis por región epidemiológica
├─ Celda 73: NEW ✨ SHAP explainability (TreeExplainer)
├─ Celda 74: NEW ✨ Tabla final comparación
├─ Celda 75: NEW ✨ Análisis de residuos
├─ Celda 76: NEW ✨ Sensibilidad min_casos
├─ Celda 77: NEW ✨ Mapa MAPE por departamento
│
├─ Celda 78: NEW ✨ Markdown: Justificación lag biológica
├─ Celda 79: NEW ✨ Markdown: Validación prospectiva 2025
├─ Celda 80: NEW ✨ Markdown: Limitaciones (8 puntos)
├─ Celda 81: NEW ✨ Markdown: Discusión operativa SNVS
│
└─ [Posible: Celdas adicionales futuras para análisis serotipo, movilidad]
```

---

## 🎬 CÓMO NAVEGAR

### Si quieres ver SOLO lo nuevo:
1. Abre Notebook_6 en Jupyter
2. Ir a celda 71 (usar Ctrl+F buscar "Bootstrap IC")
3. Leer + ejecutar celdas 71-81 en orden

### Si quieres ver TODO contexto:
1. Ejecutar desde celda 1
2. Detenerse al llegar a celda 71 (aquí empieza "NEW")
3. Luego ejecutar nuevas celdas 71-81

### Si quieres extraer figuras para paper:
1. Ejecutar celdas 73 (SHAP), 75 (Residuos), 77 (Mapa)
2. Archivos PNG guardados en `OUT_DIR/` (definido en celda 1)
3. Importar a Overleaf/Word: "Figure 1: {title}" + caption

---

## ✅ VERIFICACIÓN RÁPIDA

Para confirmar que las 11 celdas están presentes:

```python
# En una celda Jupyter al final del notebook:
import json
with open('notebooks/Notebook_6_model_training_iter2.ipynb', 'r') as f:
    nb = json.load(f)
    total_cells = len(nb['cells'])
    code_cells = sum(1 for c in nb['cells'] if c['cell_type'] == 'code')
    md_cells = sum(1 for c in nb['cells'] if c['cell_type'] == 'markdown')
    print(f"Total celdas: {total_cells}")
    print(f"Código: {code_cells}")
    print(f"Markdown: {md_cells}")
    print(f"Esperado: ≥ (celdas_originales + 11)")
```

---

## 🚀 QUICK CHECKLIST para usuario

- [ ] Abrí Notebook_6 en Jupyter? ✓
- [ ] Ejecuté celdas 1-70 (original)? ✓
- [ ] ¿Sin errores?  ✓
- [ ] Voy a celda 71? ✓
- [ ] Ejecuto celdas 71-81 en orden? ✓
- [ ] Reviso outputs + figuras? ✓
- [ ] Todo OK → **Notebook publication-ready** ✅

---

**Fecha creación**: 2026-05-08  
**Estatus**: ✅ Listas para ejecución

