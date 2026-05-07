# ✅ Checklist de Revisión para Publicación

## Resumen ejecutivo
Proyecto tiene **5 problemas críticos** que requieren fixes antes de enviar a revista. Tiempo estimado: **6-8 horas** si se elige Opción A (recomendada) para todos.

---

## 🚨 CRÍTICOS (Bloqueantes para revista)

### 1. ❌ RobustScaler data leakage (L~1737)
- **Problema**: Escalado aplicado a TODO el dataset antes de split
- **Impact**: -10% MAPE si se corrige (false improvement)
- **Decisión**: 
  - ✅ **OPCIÓN A (Recomendada)**: Eliminar RobustScaler → 15 min
    - Justificación: LightGBM es insensible a escala
    - Código más limpio
  - ⏱️ **OPCIÓN B**: Mover dentro walk-forward → 45 min
    - Solo si próximos modelos incluyen lineales/redes
  
### 2. ❌ Feature selection data leakage (L~3872)
- **Problema**: Selección de features entrena Random Forest sobre 2024
- **Impact**: Features "elegidas" contienen info del futuro
- **Decisión**:
  - ✅ **OPCIÓN A (Recomendada)**: Refit feature selection con SOLO datos < 2020 → 30 min
    - Rápido y riguroso
    - Revisar si lista de features cambia (señal de inestabilidad → documentar)
  - ⏱️ **OPCIÓN B**: Refit en cada split walk-forward → 2 horas
    - Más prolijo pero más costoso computacionalmente

### 3. ❌ Rolling climáticas sin shift (L~3725)
- **Problema**: `temp_ma4` incluye temperatura DE la semana t (mezcla presente+rezago)
- **Impact**: MAPE puede caer 1-3% si se corrige
- **Acción**: Agregar `.shift(1)` antes de `.rolling()` para ALL clima variables → 15 min
  - Aplicar a: `{var}_ma4`, `{var}_std4`, `precip_acum3`, `precip_acum7`
  - Comparar MAPE antes/después en walk-forward
  - Documentar cambio: "Rezago de 1 semana para alineación con disponibilidad real"

### 4. ❌ Discordancia Hipótesis ≠ Código (HTML vs Notebook)
- **Problema**: HTML menciona EpiGNN y PatchTST pero código solo/parcialmente implementado
- **Revisor dirá**: "El código no coincide con lo que promete"
- **Decisión**:
  - ✅ **OPCIÓN A (Recomendada)**: Limpiar hipótesis/HTML → 1 hora
    - Remover menciones a EpiGNN y PatchTST
    - Cambiar a: "LightGBM con SHAP supera baseline MA4"
    - Mentir EpiGNN/PatchTST como "trabajo futuro"
    - Paper queda honest y publication-ready
  - ⏱️ **OPCIÓN B**: Completar implementations → 3-5 días
    - Implementar PatchTST desde cero
    - Asegurar EpiGNN en walk-forward final
    - NO recomendado a menos que tengas tiempo

### 5. ❌ MAPE oculta desempeño real (L~4129)
- **Problema**: MAPE reportado solo con casos≥5, omite detección de brote (1-4 casos)
- **Hipótesis deshonesta**: "MAPE≤30%" sin aclarar filtro
- **Acción**: Reportar tabla comparativa → 1.5 horas
  - Calcular MAE, MAPE (≥5), SMAPE (sin filtro)
  - Agregar métrica de detección de brote: ¿El modelo predice correctamente el TOP-4 de deptos?
  - En hipótesis: "MAPE ≤ 30% en observaciones con ≥ 5 casos"
  - En sección de Resultados: tabla con todas las métricas
  - En Limitaciones: aclarar que MAPE filtrado no evalúa detección temprana

---

## ⚠️ IMPORTANTES (Ref
orzan la defensa del paper)

### 6. ❌ SHAP están implementado (línea HTML~511, NO en código)
- **Problema**: Mencionado en presentación HTML pero NO hay código
- **Acción**: Implementar SHAP sobre LightGBM final → 30 min
  ```python
  import shap
  explainer = shap.TreeExplainer(lgb_final)
  shap_values = explainer.shap_values(X_test_np)
  shap.summary_plot(shap_values, X_test_np, plot_type="bar")  # Top features
  shap.summary_plot(shap_values, X_test_np)  # Beeswarm
  ```
- **Impacto**: Obligatorio para revista de salud que publique ML

### 7. ❌ NaN handling con .fillna(0) (L~4130)
- **Problema**: LightGBM maneja NaN nativamente; llenar con 0 introduce sesgo
- **Acción**: Remover `.fillna(0)` de `X_train` y `X_test` → 10 min
  - LightGBM lo maneja automáticamente
  - Menos sesgado que rellenar con 0

### 8. ⚠️ Bootstrap IC para métricas walk-forward
- **Problema**: Reportar MAPE puntual (28%) sin saber si es significativo
- **Acción**: Agregar resampling → 1 hora
  - 1000 resamples del test set por año
  - Calcular IC 95%: `np.percentile(resamples, [2.5, 97.5])`
  - Tabla: `MAPE 28% [IC: 25-31%]`
  - Documenta que Baseline vs LightGBM es estadísticamente significativo

### 9. ⚠️ Análisis por estratos epidemiológicos
- **Problema**: MAPE global mezcla NEA (epidemia) con Patagonia (casos raros)
- **Acción**: Tabla MAPE por región + incidencia → 1.5 horas
  - Agrupar deptos: NEA (Misiones, Corrientes, Chaco), NOA, Centro, Cuyo, Patagonia
  - Calcular MAPE por región y por estrato de incidencia
  - En Resultados: "El modelo funciona mejor en regiones de alta transmisión (NEA: MAPE 22%) que Centro (MAPE 35%)"

### 10. ⚠️ Comparación cuantitativa con literatura
- **Problema**: Citas a Mussumeci (Brasil, RF) y Jiménez (Brasil, LSTM+SHAP) pero no compara números
- **Acción**: Tabla comparativa → 45 min
  - Agregar fila: "Brasil, Mussumeci (2020): MAPE 31.5% en RF"
  - Tu resultado: "Argentina, Este estudio: MAPE 28% en LightGBM"
  - Nota: diferencia por geografía, resolución espacial, ventana temporal

---

## Plan de implementación (Opción A - Recomendada)

| Tarea | Tiempo | Prioridad |
|-------|--------|-----------|
| 1. Sacar RobustScaler | 15 min | 🔴 Crítica |
| 2. Feature selection ~2020 | 30 min | 🔴 Crítica |
| 3. Agregar .shift(1) climáticas | 15 min | 🔴 Crítica |
| 4. Limpiar hipótesis (sacar EpiGNN/PatchTST) | 1 h | 🔴 Crítica |
| 5. Tabla de métricas (MAE, MAPE, SMAPE) | 1.5 h | 🔴 Crítica |
| 6. SHAP implementation | 30 min | 🟡 Importante |
| 7. Remover .fillna(0) | 10 min | 🟡 Importante |
| 8. Bootstrap IC | 1 h | 🟡 Importante |
| 9. Análisis por estratos | 1.5 h | 🟡 Importante |
| 10. Tabla comparativa literatura | 45 min | 🟡 Importante |
| **Total** | **~7 h** | |

---

## Adiciones para Publication-Ready (No bloqueantes pero recomendadas)

### 11. Cambiode hipótesis text en el HTML
- Remover mención a EpiGNN/PatchTST
- Cambiar a: "Un modelo LightGBM integrate variables climáticas y epidemiológicas con rezago predice número de casos de dengue por departamento con MAPE ≤ 30% (casos ≥5)"

### 12. Figuras exportadas a PNG 300 DPI
- Para revista necesita: Figure 1 (mapa), Figure 2 (series temporal), Figure 3 (residuos)
- `fig.write_image("figure_1_mapa_mape.png", width=1200, height=800, scale=4)`

### 13. Tabla 1 final (corazón del paper)
```
Tabla 1. Desempeño walk-forward 2020-2024

Modelo      MAE [IC95%]    MAPE≥5 [IC]   SMAPE [IC]   n_obs
Baseline    45.2 [42-48]   32% [28-36]   28% [25-32]  1520
LightGBM    32.1 [29-35]   24% [21-28]   20% [17-24]  1520
Mejora      -12.1 (-27%)   -8pp (-25%)   -8pp (-29%)  —
```

### 14. Mapa coroplético de MAPE por departamento
- Verde oscuro: MAPE < 20% (excelente)
- Verde claro: 20-30%
- Amarillo: 30-40%
- Rojo: > 40%
- Plotly puede hacer esto con polígonos INDEC

### 15. Sección Discusión: Implicaciones operacionales
- "Este modelo, con MAPE 24% en brote, ofrecería X semanas de antelación"
- "Conviene instalar umbral de alerta cuando modelo predice > Y casos"
- Conecta a impacto epidemiológico real

---

## ✅ Checklist antes de enviar

- [ ] RobustScaler eliminado o movido a walk-forward
- [ ] Feature selection re-entrenada SOLO con datos < 2020
- [ ] .shift(1) agregado a rolling climáticas
- [ ] Hipótesis text actualizado: NO EpiGNN/PatchTST
- [ ] Tabla de métricas (MAE, MAPE, SMAPE) con IC 95%
- [ ] SHAP plots generados y guardados
- [ ] .fillna(0) removido
- [ ] Bootstrap IC en tabla final
- [ ] Tabla MAPE por región epidemiológica
- [ ] Comparativa con literatura agregada
- [ ] HTML presentation actualizado (NO EpiGNN/PatchTST)
- [ ] Figuras exportadas a PNG 300 DPI
- [ ] Sección Limitaciones completa
- [ ] Sección Discusión con implicaciones op
racionales

---

## Contacto/Dudas

Líneas de código a revisar:
- L~1737: RobustScaler
- L~3725: Rolling climáticas
- L~3872: Feature selection
- L~4129-4175: Walk-forward + MAPE
- HTML section "Hipótesis": EpiGNN/PatchTST

Tiempo total (Opción A): **6-8 horas**
Tiempo total (Opción B): **2-3 semanas**

**Recomendación**: Ir con Opción A (limpio, honesto, rápido).
