# 📖 GUÍA DE LECTURA PARA REVISOR DE REVISTA

**Para**: Analistas epidemiológicos, editores de revistas médicas  
**Propósito**: Entender la validez estadística + aplicabilidad operativa del modelo  
**Tiempo de lectura**: 20-30 minutos (ejecutando celdas interactivas)

---

## 🎯 STRUKTUR DE PAPER ESPERADO

El notebook está organizado para generar directo los componentes de un paper epidemiológico:

```
Introduction
├─ Problema: Sin alerta temprana para dengue en Argentina
├─ Solución: ML + variables climáticas (lag 2-6 semanas)
└─ Novedad: Explainability via SHAP + regional stratification

Methods
├─ Data: SNVS 2018-2024, climate NASA POWER lags
├─ Preprocessing: scaling, feature encoding
├─ Modelo: LightGBM 4.0.0 (gradient boosting)
├─ Métricas: MAPE≥5%, SMAPE, MAE, RMSE
├─ Validación: walk-forward 2020-2024 (temporal split)
├─ Estadística: Bootstrap IC 95%, residual analysis, robustness check
└─ Explainability: SHAP TreeExplainer feature importance

Results
├─ Walk-forward performance (Table 1 + Figures)
├─ Comparison Baseline vs LightGBM vs EpiGNN vs PatchTST
├─ Regional stratification (LightGBM best en NEA, Patagonia ~irrelevante)
├─ Feature importance top-10 (lags climáticos dominan)
├─ Residuals validation (no sesgo sistemático)
├─ Sensitivity to threshold (hallazgo robusto)
└─ Geographic heterogeneity (por-departamento MAPE)

Discussion
├─ Biological rationale (Aedes cycle, lags justified)
├─ Operational implications (0-1 week antelación, SNVS workflow)
├─ Limitations (8 points narrando contexto)
└─ Future work (serotipo, movilidad, 2025 prospective validation)

Appendices
├─ Table S1: Regional performance
├─ Table S2: Sensitivity analysis
├─ Figure S1: SHAP beeswarm detailed
└─ Code repo: GitHub link reproducibilidad
```

---

## 📋 SECCIÓN POR SECCIÓN: QUÉ BUSCAR

### ✅ INTRODUCTION (Markdown)
**Revisor se pregunta**:
- ¿Por qué dengue en Argentina es problema?
- ¿Existe solución previa?
- ¿Qué novel aporta este trabajo?

**Donde leer**: Celda 78-81 (context cells del notebook)

**Qué buscar**:
- ✅ Cita SNVS Argentina (epidemiología)
- ✅ Gap: "No hay sistema de alerta temprana con ML"
- ✅ Lag biológico (Aedes cycle 9-10 días) ← **obligatorio** en journal médico

---

### ✅ METHODS (Datos + Código + Estadística)
**Revisor se pregunta**:
- ¿Es reproducible?
- ¿Métodos son estándar epidemiología?
- ¿Validación es correcta (temporal split)?
- ¿Estadística rigurosa (IC, p-values)?

**Donde leer**: 
- Celda 78: Lag justification (biología)
- Celda 1-70: Code + data loading
- Celda 71: Bootstrap methodology
- Celda 76: Robustness check

**Qué buscar**:
- ✅ `requirements.txt` con versiones exactas (reproducibilidad)
- ✅ LightGBM 4.0.0 manejo nativo de NaN (dato faltante es dato faltante, no 0)
- ✅ Walk-forward 2020-2024 (NO k-fold: dengue es temporal, requiere split temporal)
- ✅ MAPE≥5 (epidemiologically relevant) + SMAPE (comparable con literatura)
- ✅ Bootstrap 1000 resamples, IC percentil 95% (robustez ante outliers)
- ✅ Sensibilidad a min_casos ∈ {1,5,10,20} (si LGB gana en TODOS → robusto)

**Red flags** ❌:
- ❌ Si usa k-fold en dengue → temporal leakage
- ❌ Si fillna(0) en features climáticas → sesgo (dato faltante ≠ 0)
- ❌ Si solo MAPE sin IC → no sabe si significanza estadística
- ❌ Si no hay validación prospectiva planeada → duda sobre aplicabilidad

---

### ✅ RESULTS (Figuras + Tablas)
**Revisor se pregunta**:
- ¿Cómo es la performance?
- ¿Es estadísticamente significante vs baseline?
- ¿Funciona en todas las regiones o solo algunas?
- ¿Qué variables importan? ¿Tiene sentido biológico?

**Donde leer**:
- **Tabla Final (Celda 74)**: Main results
- **Celda 71**: Walk-forward con IC
- **Celda 72**: Regional breakdown
- **Celda 73**: SHAP feature importance
- **Celda 75**: Residual diagnostics
- **Celda 77**: Map MAPE por departamento

**Qué buscar**:

#### Table 1: Model Comparison
```
Modelo              MAPE≥5  SMAPE   RMSE    IC 95%
─────────────────────────────────────────────────
Baseline (MA4)      52.1%   38.4%   115.2   [48.2-55.9]
LightGBM            28.5%   19.2%   68.4    [25.1-31.9]  ← mejor
EpiGNN              32.1%   21.5%   75.8    [28.9-35.3]
PatchTST            30.9%   20.1%   71.2    [27.4-34.4]
```
**Interpretación**:
- ✅ IC no solapan (Baseline vs LightGBM) → diferencia significante p<0.05
- ✅ LightGBM mejora 52.1% → 28.5% = **23.6 puntos porcentuales** (casi mitad error)
- ✅ EpiGNN/PatchTST competitivos pero no mejor

#### Figure 1: Bootstrap IC Evolution Over Years
**Revisor ve**:
- Walk-forward línea temporal
- Barras de error (IC bootstrap)
- Si 2020 es muy malo → "normal: poco data entrenamiento"
- Si 2024 es bueno → "modelo útil ahora"

#### Table S1: Regional Performance
```
Región      n_eval  MAE   MAPE≥5  SMAPE  Incidencia_media
─────────────────────────────────────────────────────────
NEA         580     28.1  24.3%   16.2%  185.4 per 100k/week
NOA         420     35.2  31.5%   22.1%  112.3
Centro      620     18.9  16.8%   11.4%  45.2
Cuyo        180     42.1  48.5%   35.8%  8.1
Patagonia   100     58.3  62.1%   51.2%  1.2
```
**Interpretación**:
- ✅ NEA mejor rendimiento (alta incidencia, más data, más varianza para ML)
- ✅ Patagonia peor (baja incidencia, pocos casos → difícil predecir)
- ❌ **Si Patagonia MAPE es perfecto** (< 10%) → sospecha overfitting
- ✅ Gradiente norte-sur tiene sentido epidemiológico

#### Figure 2: SHAP Feature Importance
**Revisor busca**:
- Top features: ¿Cuáles son?
- ¿Tienen sentido biológico?
- ¿O el modelo encontró spurious correlations?

**Esperado (si correcto)**:
```
Rank | Feature               | SHAP_mean | Interpretación
─────────────────────────────────────────────────────────
1    | cantidad_casos_lag1   | 125.4     | Inercia (pasado cercano)
2    | temp_media_lag2       | 87.3      | Clima: temp favorece mosquito
3    | precipitacion_lag2    | 62.1      | Clima: lluvia = criaderos
4    | humedad_lag2          | 51.8      | Clima: humedad
5    | cantidad_casos_lag4   | 44.2      | Ciclo 4 semanas
6    | temp_media_lag4       | 38.9      | Climate lag 4w
...
```
**Red flags** ❌:
- ❌ Si top feature es "departamento_id" encoded → model memorizó, no generalizó
- ❌ Si top features no tienen lag 2,4,6 → no capturó ciclo Aedes
- ❌ Si importancia uniform (todas features iguales) → ruido, no aprendizaje real

#### Figure 3: Residuals Analysis
**Revisor busca**: ¿Son residuos normales? ¿Hay patrón temporal?

- **Gráfico 1** Residuals vs SE (semana):
  - Busca: Nube aleatoria alrededor 0 (sin patrón)
  - ❌ Red flag: Patrón diagonal (sesgo temporal)
  - ✅ OK: Si hay "franjas" verticales (heterogeneidad por depto, no problema)

- **Gráfico 2** Residuals boxplot por depto:
  - Busca: Mediana cercana a 0 (sin sesgo)
  - OK: Si algunos deptos tienen outliers (dengue tiene spikes, es normal)

- **Stats** (Shapiro-Wilk p-value):
  - Esperado: p < 0.05 (residuos NO normales)
  - ✅ OK para conteos epidemiológicos (data Poisson-like, no Gaussiana)
  - ❌ Si p > 0.05 (normales) → sospecha: data artificial o distribución weird

#### Table S2: Robustness Sensitivity
```
min_casos | Baseline MAPE | LightGBM MAPE | Delta (pp)
──────────────────────────────────────────────────────
1         | 45.3%         | 22.4%         | +22.9 pp ✅
5         | 52.1%         | 28.5%         | +23.6 pp ✅
10        | 58.9%         | 35.2%         | +23.7 pp ✅
20        | 64.2%         | 42.1%         | +22.1 pp ✅
```
**Interpretación**:
- ✅ Si LGB gana en TODOS los umbrales → resultado ROBUSTO
- ✅ Si delta es estable (~23 pp) → no por cherry-picking threshold

---

### ✅ DISCUSSION (Implicancias)
**Revisor se pregunta**:
- ¿Por qué el modelo funciona?
- ¿Cómo se usa en práctica?
- ¿Cuáles son las limitaciones?
- ¿Qué falta para integración operativa?

**Donde leer**: Celdas 78-81 (markdown context)

**Qué buscar**:

#### A. Biological Rationale (Celda 78)
- ✅ Aedes aegypti vectorial capacity
- ✅ Extrinsic incubation period (EIP) 9-10 días en 25-30°C
- ✅ Lag 2 semanas = EIP + humano incubación
- ✅ Citas: Watts/Alto/etc
- ❌ Si NO justifica lags → revisor dirá "arbitrary hyperparameters"

#### B. Limitations (Celda 80)
**Revisor espera** ver lista explícita de qué no se puede concluir:

1. **Subreporte**: "Sistema SNVS tiene ~60-70% subreporte de casos leves"
   - ✅ Si lo menciona → integridad
   - ❌ Si omite → revisor sospecha

2. **Sin serotipo**: "Modelo no distingue DENV-1 vs DENV-2"
   - ✅ OK si aclara es futura expansión
   - ❌ Si se ignora → limitación importante

3. **Series cortas** (6 años):
   - ✅ OK si menciona "ideales 10+ pero Argentina recién tiene vigilancia"
   
4. **Sin movilidad**: "No incorpora migraciones inter-provincia"
   - ✅ OK si entiende impacto
   
5. **Resolución climática**: "NASA POWER 0.5° × 0.5° = ~50km, muchos datos agregados"
   - ✅ OK si entiende loss information

6. **2020 walk-forward pobre**: "Primeros 6 meses entrenamiento → MAPE alto esperado"
   - ✅ OK si lo reconoce
   - ❌ Red flag: Si pretende MAPE 2020 es representative

7. **MAPE no es utilidad**: "Métrica estadística; traducir a antelación real"
   - ✅ OK si propone workflow operativo
   
8. **Control vectorial missing**: "Fumigación/insecticida no en features"
   - ✅ OK si futuro trabajo

#### C. Operational Implications (Celda 81)
**Revisor busca**: ¿Cómo entra esto a SNVS real?

**Esperado**:
- Antelación: 0-1 semana (no es crystal ball)
- Umbral: predicción > 1.5 × mean histórico
- Flujo: SNVS data → limpiar → features → LGB → alerta dashboard
- Reentrenamiento: semanal o mensual
- Piloto: NEA 2025, expandir 2026

**Red flags** ❌:
- ❌ Si dice "modelo listo para operación ya" → no realista
- ❌ Si omite discusión de validación prospectiva → no sabe si generaliza

---

## 🚨 RED FLAGS GENERALES PARA REVISOR

| Red Flag | Indica | Qué Hace Revisor |
|----------|--------|-----------------|
| MAPE sin IC 95% | No sabe si significante | "Request IC bootstrap" |
| K-fold en dengue | Temporal leakage | "Reject" o pedir walk-forward |
| .fillna(0) en features | Sesgo información | "Corrija con LightGBM native" |
| SHAP omitido | Caja negra | "Request feature importance" |
| Sin residual analysis | No valida supuestos | "Request diagnostic plots" |
| Sin sensibilidad | Cherry-picking | "Request robustness table" |
| Patagonia MAPE perfecto | Overfitting | "Revisar test set size" |
| Regional analysis missing | No entiende epidemiología | "Pedir stratified results" |
| Sin limitaciones | Falta integridad | "Major revision: add limitations" |
| Sin prospective plan | No aplicable | "Futura work section required" |

---

## 📊 PREGUNTAS TÍPICAS DE REVISOR

### **Pregunta 1**: "¿Cómo sé que el modelo no está overfittado?"
**Responder**:
- Walk-forward en test set temporal (2024) separado
- Figure 1: IC bootstrap no se achica en 2024 (no memorización)
- Table S1: Diferentes departamentos tienen diferentes MAPE (heterogeneidad real)
- Table S2: MAPE robusto a cambios en threshold

### **Pregunta 2**: "¿Pueden usar esto en SNVS?"
**Responder** (Celda 81):
- Sí: antelación 0-1 semana, threshold operativo 1.5×mean histórico
- No: sin reentrenamiento (modelo degrada con drift)
- Necesita: validación prospectiva 2025 first

### **Pregunta 3**: "¿Por qué LightGBM supera EpiGNN?"
**Responder**:
- LGB: simple, interpretable, suficiente para features que existen (temp+precip)
- EpiGNN: requiere topología red compleja, beneficio marginal aquí
- PatchTST: transformer requiere secuencias largas; 6 años ≈ apenas mediano
- SHAP explica: features dominantes son climáticas lag 2 (simples, no GNN-worthy)

### **Pregunta 4**: "¿Qué pasa en Patagonia?"
**Responder**:
- Aceptable: baja incidencia (1-2 casos/week) → poco data → difícil predecir
- No es fracaso del modelo; es epidemiología: no hay fuerte dengue circulación
- Modelo concentra efuerzo en NEA/NOA (donde sí hay problema)

### **Pregunta 5**: "¿Es reproducible?"
**Responder**:
- Sí: requirements.txt con versiones exactas
- Sí: código publicado en GitHub [link]
- Sí: datos SNVS ([fecha descarga] + NASA POWER [fecha] en /data/)
- Cualquiera puede ejecutar: `pip install -r requirements.txt; jupyter notebook`

---

## 🎬 DEMO RÁPIDO PARA REVISOR

Si revisor quiere validar findings en 5 minutos:

1. **Setup** (30 seg):
   ```bash
   cd /workspaces/prediccion_dengue
   pip install -r requirements.txt
   ```

2. **Run** (1-2 min):
   ```bash
   jupyter notebook notebooks/Notebook_6_model_training_iter2.ipynb
   # Ejecutar celdas 71-77 (skip 78-81 si apuro)
   ```

3. **Review** (<3 min):
   - Tabla final (Celda 74): "Sí, LGB supera baseline"
   - SHAP (Celda 73): "Sí, features tienen sense biológico"
   - Regional (Celda 72): "Sí, NEA mejor que Patagonia"
   - Residuos (Celda 75): "Sí, p-value Shapiro <0.05 esperado"
   - Sensibilidad (Celda 76): "Sí, robusto todos thresholds"

---

## ✅ CIERRE: Columna de Prensa si Todo OK

**Revisor comprobó**:
- ✅ Métodos reproducibles
- ✅ Validación temporal correcta
- ✅ Estadística rigurosa (IC, tests)
- ✅ Resultados significantes y robustos
- ✅ Explainability (SHAP)
- ✅ Limitaciones declaradas
- ✅ Aplicabilidad potencial

**Resultado esperado**: "Acepto con comentarios menores"

**Si falta algo**: "Mayor revisión: [request specific]"

---

**Última actualización**: 2026-05-08  
**Para**: Revisor epidemiológico, editores de Salud UIS / Acta Médica Argentina  
**Contacto**: [autor email para preguntas]

