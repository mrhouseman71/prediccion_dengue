# Presentation Script — Dengue AI
## Departmental Dengue Outbreak Prediction in Argentina using Spatial Machine Learning
**Balda · Caracoix · Casas | UCA 2026 | ~10 minutes | Language: English**

---

> **HOW TO USE THIS SCRIPT**
> - Each section has a **[TAB]** cue telling you which dashboard tab to display.
> - *Italics* = slide/tab action. **Bold** = key term to emphasise.
> - Word counts and timings are guides — speak naturally.
> - Practice tip: read aloud 3 times. Target pace: ~130 words/min.

---

## SECTION 1 — Title and Problem Statement
**[TAB: 🎯 Problema] · 0:00–0:45 (~90 words)**

> *Open on the Problem tab. The 5 KPI cards are visible.*

"Good morning, everyone.

My name is [your name], and today — together with Javier Balda and Juan Caracoix — we're presenting our final project for Data Consulting: Laboratory III.

Our project is called **Dengue AI**: a departmental outbreak prediction system for Argentina, built entirely on open-access data.

Let me start with the problem.

*(point to the KPI cards)*

In 2024, Argentina recorded **582,000 confirmed dengue cases** — ten times the previous historical maximum. The national surveillance system issued alerts **reactively**: case counts arrive with a two-to-three-week delay, when transmission is already established. By then, it's too late for preventive action.

We asked: *what if we could predict next week's cases before they happen?*"

---

## SECTION 2 — Motivation
**[TAB: 🎯 Problema, bottom cards] · 0:45–1:30 (~90 words)**

> *Keep the Problem tab open. Point to the bar comparison card at the bottom right.*

"Why did we choose this problem?

Because **dengue is predictable** — it follows seasonal and climate patterns, it propagates along geographic corridors, and it leaves a signal in the data weeks before a peak is visible in the surveillance system.

The value of solving this is concrete: four weeks of anticipation is enough time to mobilise fumigation teams, reinforce hospital capacity, and launch prevention campaigns.

And the key constraint we imposed on ourselves: **everything runs on public, open-access data**. No proprietary datasets. No paid APIs. If it works here, any health ministry in Latin America can replicate it tomorrow."

---

## SECTION 3 — Dataset
**[TAB: 📦 Datos] · 1:30–2:30 (~130 words)**

> *Switch to the Data tab.*

"Our dataset integrates **five open-access sources**.

*(point to the timeline on the left)*

The core source is **SNVS 2.0** — Argentina's national health surveillance system — which provides weekly confirmed dengue cases at the departmental level, from 2018 to 2025. One of our biggest data engineering challenges was that the SNVS changed its reporting schema between years, so we had to build an automatic normalisation pipeline to reconcile both formats.

We enriched that with **NASA POWER** for climate variables — 16 daily variables aggregated weekly — and the **2022 National Census** for demographic proxies like water and sanitation access.

The final dataset has approximately **17,000 observations** across 520 departments and 7 years.

*(point to the feature engineering cards on the right)*

We engineered over 90 features: epidemiological lags, lagged climate variables, cyclical encodings for seasonality, and biological index variables. The final 30 were selected using a strict temporal cutoff — only data from 2018 to 2019 — to prevent **data leakage**."

---

## SECTION 4 — Product
**[TAB: 🚀 Valor, then briefly show 🎯 Problema] · 2:30–3:30 (~120 words)**

> *Switch to the Value tab. Show the pitch card at the top.*

"The product we built has two components.

The first is the **prediction framework** itself: a pipeline that takes weekly SNVS data, runs it through LightGBM, and outputs a risk score for each of Argentina's 520+ departments for the following week. The inference takes milliseconds per department and can run on any standard laptop.

*(briefly switch back to Problem tab, point to the bar chart)*

The second is **this interactive dashboard** — which is what you're seeing right now. It integrates all results, allows filtering by department, displays walk-forward metrics with confidence intervals, and shows SHAP feature explanations so a health coordinator can understand *why* a specific department received a high-risk alert.

The full pipeline, notebooks, processed data, and this dashboard are publicly available on **Zenodo** under DOI 10.5281/zenodo.20517162."

---

## SECTION 5 — Models and Methodology
**[TAB: 🤖 Modelos] · 3:30–5:00 (~180 words)**

> *Switch to the Models tab. Walk through the three model cards.*

"We compared three approaches.

*(point to Baseline card)*

First, a **moving average baseline** — the four-week rolling mean. This is essentially the logic behind the current SNVS alert system. It requires no external variables and sets the floor any model must beat.

*(point to LightGBM card)*

Second, **LightGBM** — our main production model. It's a gradient boosting algorithm on decision trees, handles missing values natively, and is interpretable through SHAP. We trained it with 500 estimators, learning rate 0.05, max depth 6. This is the model we recommend for operational deployment.

*(point to EpiGNN card)*

Third, **EpiGNN** — an exploratory extension. It models departments as nodes in a geographic graph, with edges based on spatial proximity. Two GCNConv layers learn spatio-temporal representations of outbreak dynamics. It's the most complex model, and as we'll see, it wins on the 2024 test but is less stable across years.

*(point to validation timeline)*

The critical methodological decision was **walk-forward validation**: train on all years before year *t*, evaluate on year *t*. This simulates real operational conditions and prevents temporal data leakage. We also computed **95% bootstrap confidence intervals** with 1,000 resamples to verify that our improvements weren't statistical noise."

---

## SECTION 6 — Metrics and Results
**[TAB: 📊 Resultados] · 5:00–6:30 (~180 words)**

> *Switch to the Results tab. Point to the top KPI bar, then the table, then the chart.*

"Now the results.

*(point to the test 2024 table)*

On the **2024 test set** — the most extreme outbreak in Argentina's recorded history — EpiGNN achieved the best absolute performance: **MAE of 14.3** and **SMAPE of 41.6%**. LightGBM outperformed the baseline with a MAPE of 101% versus 144%.

I want to address that number directly: yes, 101% MAPE looks bad. But the 2024 outbreak was ten times the historical maximum. No model trained on 2018–2023 could anticipate that magnitude. What matters is the **relative improvement over the baseline**, which was statistically significant.

*(point to the walk-forward table)*

The walk-forward results tell the full story. Over six years — 2020 to 2025, including prospective data from 2025 that was never seen during training — **LightGBM consistently outperformed the baseline**, with a mean improvement of 22 percentage points in MAPE.

*(point to the confidence interval column)*

The 95% bootstrap confidence intervals confirm that this improvement is not random. In five of six years, the intervals don't overlap with the baseline."

---

## SECTION 7 — Contribution
**[TAB: 🚀 Valor] · 6:30–7:30 (~120 words)**

> *Switch to the Value tab. Point to the six application cards.*

"What does this actually enable?

For the **Ministry of Health**: prioritise departments for vector control before the epidemic peak. Four weeks of lead time is exactly what fumigation and resource reallocation campaigns require.

For **hospitals**: anticipate staffing and intensive care needs in high-prediction municipalities before patient arrivals overwhelm the system.

For **public communication**: target prevention campaigns where climate and epidemiological conditions are most favorable for transmission — not where cases have already appeared.

*(point to the comparison table)*

And compared to existing literature: our work operates at **departmental resolution** — not provincial — uses **100% open-access data**, applies **walk-forward validation with confidence intervals** instead of random cross-validation, and includes SHAP interpretability with biological validation. The full pipeline is reproducible on Zenodo."

---

## SECTION 8 — Limitations and Future Work
**[TAB: 🎓 Aprendizaje] · 7:30–8:45 (~140 words)**

> *Switch to the Learning tab. Point to the roadmap in the Value tab if needed.*

"We want to be transparent about what this system cannot do.

The **real operational lead time** is zero to one week — not four. Because our model uses lags of two to six weeks, and SNVS data arrives with a two-week delay, by the time we predict for week *i+1*, we're using data from weeks *i−2* to *i−6*. Acknowledging this publicly is itself a methodological contribution.

Other limitations: the model doesn't include **serotype information**, which drives cross-immunity dynamics. Climate resolution is 55 km — too coarse for heterogeneous departments. And we have only five years of training data, below the ten-year minimum recommended for seasonal time series.

For future work, we'd add **human mobility data** to improve EpiGNN's spatial graph, higher-resolution climate sources like CHIRPS or ERA5, and vaccine coverage data following Argentina's Qdenga rollout in 2024."

---

## SECTION 9 — Reflection
**[TAB: 🎓 Aprendizaje] · 8:45–10:00 (~140 words)**

> *Stay on Learning tab. Speak freely — this is your moment.*

"Finally, what did we learn as a team?

The most important technical lesson: **data integration took longer than model training**. Normalising two SNVS schemas, reconciling department names across five sources, handling missing climate values — this is the invisible work that determines whether a model is actually usable in the real world.

The most important methodological lesson: **a simpler model with honest validation beats a complex model with optimistic metrics**. LightGBM outperformed EpiGNN in five of six walk-forward years. Complexity must earn its place.

And the professional skill we developed most: **communicating uncertainty honestly**. Telling stakeholders that a model has 101% MAPE in a historic outbreak, and explaining why that's still useful, requires a different kind of confidence than just showing good numbers.

*(pause — look at the audience)*

Thank you. We're happy to take any questions."

---

## QUICK REFERENCE — Tab Navigation Order

| Time | Tab to show | Key visual |
|---|---|---|
| 0:00 | 🎯 Problema | KPI strip + problem card |
| 0:45 | 🎯 Problema | Bar chart comparison |
| 1:30 | 📦 Datos | Timeline + feature cards |
| 2:30 | 🚀 Valor → 🎯 Problema | Pitch card → bar chart |
| 3:30 | 🤖 Modelos | Three model cards + validation |
| 5:00 | 📊 Resultados | Test table + WF chart |
| 6:30 | 🚀 Valor | Application cards + comparison table |
| 7:30 | 🎓 Aprendizaje | Learning timeline |
| 8:45 | 🎓 Aprendizaje | Reflection quotes |

---

## PRONUNCIATION GUIDE (key terms)

| Term | How to say it |
|---|---|
| MAPE | "MAP-ee" or say "Mean Absolute Percentage Error" first time |
| SMAPE | "S-MAP-ee" |
| LightGBM | "Light-GEE-BEE-EM" |
| EpiGNN | "EPI-JEE-EN-EN" |
| SNVS | spell it out: "ESS-EN-VEE-ESS" |
| walk-forward | "WAWK for-werd" — explain it: "rolling time-based validation" |
| SHAP | "SHAP" (rhymes with "map") |
| departmental | "dee-PART-men-tl" |
| GCNConv | "JEE-SEE-EN-CONV" or "Graph Convolutional layer" |

---

## TIPS FOR DELIVERY

**Pacing:** 10 minutes fills up fast. Do a dry run with a timer. If you're running over, cut Section 4 (Product) to 60 seconds — it's the one most covered by the visual itself.

**Opening:** Don't say "Today we will talk about...". Start with the number: *"582,000 cases. That was Argentina in 2024."* Silence for one beat. Then continue.

**The 101% MAPE moment:** Own it proactively. Don't wait for someone to challenge it. Say: *"I know what you're thinking — 101% MAPE sounds terrible. Let me explain why it isn't."*

**EpiGNN:** Describe it visually: *"Imagine a map of Argentina where every department is a dot, and dots that are neighbors are connected by a line. EpiGNN learns how outbreaks spread along those connections."*

**Closing:** End on the reflection section with a pause. Don't rush to say "any questions?" — let the last sentence land.
