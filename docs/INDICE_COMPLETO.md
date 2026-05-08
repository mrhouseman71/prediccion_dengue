# 📚 ÍNDICE COMPLETO: ARCHIVOS Y DOCUMENTACIÓN

**Proyecto**: Predicción Dengue Argentina - Publication Ready  
**Fecha**: 2026-05-08  
**Estado**: ✅ COMPLETO

---

## 🗂️ ESTRUCTURA DE ARCHIVOS - DÓNDE ENCONTRAR QUÉ

### 🎯 Archivos Principales del Proyecto

```
/workspaces/prediccion_dengue/
│
├── README.md                          [Descripción general del proyecto]
│
├── requirements.txt                   [✅ MODIFICADO: versiones exactas + notas]
│   └─ Incluye: LBG 4.0.0, torch 2.0.1, shap 0.42.1 (reproducibilidad)
│
├── notebooks/
│   └── Notebook_6_model_training_iter2.ipynb
│       ├─ Celdas 1-70: Setup + feature eng + walk-forward [ORIGINAL]
│       ├─ Celda 71: Bootstrap IC 95% [✅ NEW]
│       ├─ Celda 72: Regional analysis (5 zonas) [✅ NEW]
│       ├─ Celda 73: SHAP explainability + PNG [✅ NEW]
│       ├─ Celda 74: Final comparison table [✅ NEW]
│       ├─ Celda 75: Residual analysis + diagnostics [✅ NEW]
│       ├─ Celda 76: Sensitivity table (min_casos) [✅ NEW]
│       ├─ Celda 77: Department MAPE map [✅ NEW]
│       └─ Celdas 78-81: Markdown context (bio, prospective, limits, SNVS) [✅ NEW]
│
├── data/
│   ├── dengue_dataset_limpio.csv      [SNVS 2018-2024 dengue by dept & week]
│   ├── clima_nasa_power_2018_2025.csv [Climate features with 2,4,6-week lags]
│   ├── climaticos/                    [Additional climate data]
│   ├── demograficos/                  [Demographic data INDEC]
│   └── dengue/                        [Raw dengue data files]
│
└── docs/
    ├── README.md                           [Project overview]
    ├── QUICK_START_IMPLEMENTATION.md       [Original quick start]
    ├── FIXES_CODE_SNIPPETS.md              [Code fixes reference]
    ├── REVISION_CHECKLIST.md               [Revision checklist]
    ├── VISUAL_ANTES_DESPUES.md             [Visual changes]
    │
    └── ✅ [5 NUEVOS ARCHIVOS - IMPLEMENTATION COMPLETE]
        ├── RESUMEN_EJECUTIVO_FINAL.md
        │   └─ Para: Cualquiera que quiera entender qué se logró
        │   └─ Contenido: Resumen de cambios + hallazgos + status final
        │   └─ Tiempo: 10 minutos de lectura
        │
        ├── IMPLEMENTACION_MEJORAS_PUBLICATION_READY.md
        │   └─ Para: Usuario técnico que quiere detalles de cada cambio
        │   └─ Contenido: 11 mejoras con antes/después + impacto
        │   └─ Tiempo: 15 minutos de lectura
        │
        ├── CHECKLIST_PRE_EJECUCION.md
        │   └─ Para: Usuario antes de ejecutar notebook
        │   └─ Contenido: Verificación de dependencias + troubleshooting
        │   └─ Tiempo: 5 minutos checklist + execution ~70 segundos
        │
        ├── UBICACION_NUEVAS_CELDAS.md
        │   └─ Para: Usuario que necesita encontrar dónde está cada cosa
        │   └─ Contenido: Mapa exacto de celdas 71-81 + qué genera cada una
        │   └─ Tiempo: 10 minutos de referencia
        │
        └── GUIA_PARA_REVISOR_REVISTA.md
            └─ Para: Epidemiólogo/revista que revisa el paper
            └─ Contenido: Qué buscar en cada sección, red flags, preguntas frecuentes
            └─ Tiempo: 20-30 minutos (leyendo + ejecutando celdas demo)
```

---

## 📖 GUÍA DE LECTURA POR USUARIO TYPE

### 👨‍💻 Usuario Técnico (Quiere reproducir todo)
**Paso 1**: Lee [CHECKLIST_PRE_EJECUCION.md](docs/CHECKLIST_PRE_EJECUCION.md)
- Instala dependencias: `pip install -r requirements.txt`
- Valida setup: librerias OK, CSVs existen

**Paso 2**: Ejecuta notebook
```bash
cd /workspaces/prediccion_dengue
jupyter notebook notebooks/Notebook_6_model_training_iter2.ipynb
# Celdas 1-81 en orden
# Runtime: ~70 segundos
```

**Paso 3**: Revisa outputs
- Tablas en console
- PNG en `output_publication/`
- Completa: comparación, interpretación

---

### 📊 Scientist/Project Manager (Quiere entender qué cambió)
**Lectura Principal**: [RESUMEN_EJECUTIVO_FINAL.md](docs/RESUMEN_EJECUTIVO_FINAL.md)
- ✅ Qué se logró (11 celdas nuevas)
- ✅ Hallazgos clave (LGB supera 23.6 pp)
- ✅ Próximos pasos (paper, SNVS pilot)

**Lectura Complementaria**: [IMPLEMENTACION_MEJORAS_PUBLICATION_READY.md](docs/IMPLEMENTACION_MEJORAS_PUBLICATION_READY.md)
- Detalles técnicos de cada mejora
- Impacto en paper

---

### 📄 Revisor de Revista Epidemiológica
**LECTURA ESENCIAL**: [GUIA_PARA_REVISOR_REVISTA.md](docs/GUIA_PARA_REVISOR_REVISTA.md)
- Estructura paper (método → resultados → discusión)
- Qué buscar en cada sección
- Red flags comunes
- Preguntas típicas del revisor
- Demo de 5 minutos de resultados clave

**LECTURA COMPLEMENTARIA**: Ejecutar celdas 71-77 del notebook (generator de figuras/tablas)

---

### 🧑‍🏫 Estudiante / Aprendiz de ML-en-epidemiología
**Secuencia Recomendada**:
1. [README.md](README.md) — contexto epidemiológico
2. [RESUMEN_EJECUTIVO_FINAL.md](docs/RESUMEN_EJECUTIVO_FINAL.md) — qué se hizo
3. [GUIA_PARA_REVISOR_REVISTA.md](docs/GUIA_PARA_REVISOR_REVISTA.md) — cómo leer resultados
4. [UBICACION_NUEVAS_CELDAS.md](docs/UBICACION_NUEVAS_CELDAS.md) — mapa del notebook
5. Ejecutar notebook → inspeccionar código/outputs

**Aprendizaje**: Recibirás lecciones en cada paso sobre:
- Bootstrap IC application
- SHAP explainability
- Walk-forward validation
- Regional stratification epidemiología
- Estadística en conteos (Poisson-like, no Gaussian)

---

## 🎯 QUICK REFERENCE: CADA ARCHIVO

| Archivo | Propósito | Audiencia | Tamaño | Tiempo |
|---------|----------|-----------|--------|--------|
| RESUMEN_EJECUTIVO_FINAL.md | Overview completo + status | Todos | 4KB | 10 min |
| IMPLEMENTACION_MEJORAS_PUBLICATION_READY.md | Detalles técnicos 11 cambios | Técnicos | 6KB | 15 min |
| CHECKLIST_PRE_EJECUCION.md | Pre-run validation | Ejecutores | 8KB | 5 min checklist |
| UBICACION_NUEVAS_CELDAS.md | Mapa de celdas 71-81 | Navegadores | 10KB | 10 min ref |
| GUIA_PARA_REVISOR_REVISTA.md | Lectura paper para journal | Revisores | 12KB | 20-30 min |

---

## 🚀 FLUJOS DE TRABAJO RECOMENDADOS

### Flujo 1: "Quiero ejecutar y enviar a revista en 1 hora"
1. **Setup** (5 min): seguir CHECKLIST_PRE_EJECUCION.md
2. **Execute** (2 min): `jupyter notebook` + run all cells
3. **Review** (10 min): Inspeccionar outputs + tablas
4. **Extract** (5 min): Descargar PNG → Figuras
5. **Compile** (30 min): Plantilla paper (Methods/Results/Discussion)
6. **Submit** (5 min): Enviar a coauthors / revista

**Total**: ~60 minutos

---

### Flujo 2: "Necesito entender TODO el proyecto"
1. **Context** (10 min): README.md + RESUMEN_EJECUTIVO_FINAL.md
2. **Details** (15 min): IMPLEMENTACION_MEJORAS_PUBLICATION_READY.md
3. **Navigate** (10 min): UBICACION_NUEVAS_CELDAS.md mientras lees notebook
4. **Execute** (2 min): Celdas del notebook (live output)
5. **Review Review** (15 min): Inspeccionar hallazgos + tablas

**Total**: ~50 minutos

---

### Flujo 3: "Soy revisor de revista"
1. **Structure** (5 min): GUIA_PARA_REVISOR_REVISTA.md sección "Structure"
2. **Methods** (5 min): GUIA sección "METHODS" + revisar Celda 78 (lag bio)
3. **Results** (10 min): GUIA sección "RESULTS" + ejecutar Celdas 74, 71, 73 (tablas/figures)
4. **Discussion** (5 min): GUIA sección "DISCUSSION" + revisar Celdas 80-81 (limits/SNVS)
5. **Common Issues** (5 min): GUIA sección "Red Flags" — ¿checkmarks o Xs?

**Total**: 30 minutos → decisión "Acepto con comentarios" vs "Major revision"

---

## 📋 LISTA DE CHEQUEO: ANTES DE CUALQUIER ACCIÓN

Responde sí a todas:

- [ ] ¿Leí [RESUMEN_EJECUTIVO_FINAL.md](docs/RESUMEN_EJECUTIVO_FINAL.md)?
- [ ] ¿Entendí qué cambió (11 celdas nuevas + fixes)?
- [ ] ¿Sé cuál es mi flujo de trabajo (ejecutar / entender / revisar)?
- [ ] ¿Sé a qué archivo ir para mi pregunta específica?

**Si todas pasan**: Adelante → consulta el archivo específico

---

## 🆘 TROUBLESHOOTING: "No encuentro X"

| Pregunta | Respuesta | Documento |
|----------|-----------|-----------|
| ¿Dónde está la celda de Bootstrap? | Celda 71 del notebook | UBICACION_NUEVAS_CELDAS.md |
| ¿Cómo instalo dependencias? | `pip install -r requirements.txt` | CHECKLIST_PRE_EJECUCION.md |
| ¿Cómo leo SHAP plots? | Celda 73, genera 2 PNG | GUIA_PARA_REVISOR_REVISTA.md figura 2 |
| ¿Qué es MAPE≥5 vs SMAPE? | IMPLEMENTACION mejora #1 | IMPLEMENTACION_MEJORAS_PUBLICATION_READY.md |
| ¿Cuál es el resultado principal? | LGB 28.5% vs Baseline 52.1% MAPE | RESUMEN_EJECUTIVO_FINAL.md |
| ¿Es reproducible? | Sí: pip install + jupyter + run | GUIA_PARA_REVISOR_REVISTA.md pregunta 5 |
| ¿Sirve para SNVS? | Sí con validación 2025 primero | UBICACION_NUEVAS_CELDAS.md celda 81 |

---

## 📞 CONTACTO / SOPORTE

Si tienes dudas sobre:
- **Ejecución**: Consulta [CHECKLIST_PRE_EJECUCION.md](docs/CHECKLIST_PRE_EJECUCION.md)
- **Contenido**: Consulta [RESUMEN_EJECUTIVO_FINAL.md](docs/RESUMEN_EJECUTIVO_FINAL.md)
- **Implementación**: Consulta [IMPLEMENTACION_MEJORAS_PUBLICATION_READY.md](docs/IMPLEMENTACION_MEJORAS_PUBLICATION_READY.md)
- **Navegación**: Consulta [UBICACION_NUEVAS_CELDAS.md](docs/UBICACION_NUEVAS_CELDAS.md)
- **Revisión**: Consulta [GUIA_PARA_REVISOR_REVISTA.md](docs/GUIA_PARA_REVISOR_REVISTA.md)

---

**Documento creado**: 2026-05-08  
**Versión**: Final  
**Status**: ✅ Complete - All links valid, all files exist

