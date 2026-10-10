# Lab 12 — Reparto del trabajo (pareja)

El lab es en parejas. Se divide por partes para que cada integrante mida, prediga y responda lo suyo, y la síntesis se arma en conjunto.

| Integrante | Partes | Modos que corre | CSV que entrega | Gráficas |
|---|---|---|---|---|
| **A (este repo)** | Preparación (datos del equipo), **Parte 1** (Amdahl) y **Parte 2** (Ts honesto + granularidad) | `amdahl` ×5 (fp = 1.00, 0.95, 0.90, 0.75, 0.50), `suma` | 5 `amdahl_fp*.csv` + `suma.csv` + `amdahl_con_S_E.csv`, `suma_con_S_E.csv` | `amdahl_speedup.png`, `amdahl_eficiencia.png`, `suma_speedup.png` |
| **B (compañero)** | **Parte 3** (desbalance) y **Parte 4** (Gustafson) | `desbalance` ×3 (static, dynamic, guided), `gustafson` | 3 `desb_*.csv` + `gustafson.csv` + `desbalance_con_S_E.csv`, `gustafson_con_S_E.csv` | `desbalance_speedup.png`, `desbalance_eficiencia.png`, `gustafson_speedup.png`, `gustafson_eficiencia.png` |
| **Ambos** | Síntesis (fugas η_f, η_c, η_b, η_r con número medido) y reto opcional | — | — | — |

## Qué cubre cada mitad del entregable

**A — Partes 1 y 2**
- Datos del equipo (CPU, núcleos físicos/lógicos, núcleos de rendimiento/eficiencia, SMT) con captura de `sysctl`.
- Predicciones antes de cada experimento y comparación con lo medido.
- Tabla de la Parte 1 completa y cálculo a mano de S, E, C y F para al menos dos filas.
- Preguntas 1.1 a 1.8 (incluye el **primer modelo**, Amdahl) y 2.1 a 2.7.
- Capturas de `make`, de `amdahl` y de `suma`.

**B — Partes 3 y 4**
- Preguntas 3.1 a 3.5 y 4.1 a 4.5 (incluye el **segundo modelo**, Gustafson).
- Capturas de `desbalance` y de `gustafson`.
- Si corre en otra máquina, sus datos de equipo deben ir aparte: los resultados no se pueden comparar entre equipos distintos.

**Síntesis conjunta**: cada uno aporta la fuga de sus partes con el número medido; lo que no encaja en ninguna fuga (preguntas 1.1 y 2.7) lo aporta A.

## Cómo se juntan las mitades
1. B corre sus modos con el mismo `lab12_speedup.c` y deja sus CSV en `resultados/`.
2. `cd resultados && python3 ../graficar.py *.csv` regenera las 7 gráficas y los `*_con_S_E.csv`.
3. Se unen los reportes en `reporte_lab12.pdf` y se arma `resultados_lab12.zip` con los 10 CSV y los `*_con_S_E.csv`.
