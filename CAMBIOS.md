# Cambios sobre `SistemasDeRecomendacion-CORREGIDA.ipynb`

Resumen de lo implementado para resolver los dos puntos pendientes de la devolución del docente: verificación de supuestos antes de usar t-test/ANOVA, y segunda modificación experimental en la sección 6.1.

Archivos tocados: `SistemasDeRecomendacion-CORREGIDA.ipynb` (79 → **85 celdas**), `SistemasDeRecomendacion.ejecutada.ipynb` (sincronizada, ahora es copia idéntica de la anterior con sus outputs), `.gitignore` (una línea, para no trackear el venv armado para poder ejecutar en Linux). Nada más se tocó, las secciones 1-4, 4.4, 5.1, 5.2 y 5.4 quedaron byte a byte iguales en su texto.

## 1. Supuestos del t-test/ANOVA (sección 5.3)

**Celda nueva** (antes de `independent_ttest`): función `check_assumptions(*samples, alpha=0.05)` corre `scipy.stats.normaltest` (D'Agostino-Pearson) por grupo y `scipy.stats.levene(center='median')` entre grupos, y devuelve si se cumple normalidad y homogeneidad de varianzas.

**`independent_ttest`**: antes hacía siempre Welch t-test. Ahora: si `check_assumptions` da normalidad en ambos grupos usa Welch t-test; si no, `mannwhitneyu`. La función devuelve un campo nuevo `test` indicando cuál corrió. La celda que la llama (con el `print`/`display`) no cambió de forma, solo el texto del `print` ("T-test..." → "Comparación...", porque ya no siempre es un t-test).

**`one_way_anova`**: antes siempre `f_oneway`. Ahora: si hay normalidad *y* homogeneidad de varianzas en todos los grupos, `f_oneway`; si no, `kruskal`. Mismo patrón: campo `test` nuevo, prints renombrados a "Comparación por edad/país".

**`paired_model_ttest`**: antes siempre `ttest_rel`. Ahora chequea normalidad de las *diferencias* (no de las muestras crudas, que es el supuesto correcto para pareado); si se cumple usa `ttest_rel`, si no `wilcoxon`.

### Resultado real con los datos del dataset

Ninguna de las 5 comparaciones cumplía normalidad, así que las 5 pasaron a su alternativa no paramétrica:

| Comparación | Antes | Ahora | ¿Cambia la conclusión? |
|---|---|---|---|
| historial corto vs. largo | Welch t-test, p=0.172 (NS) | Mann-Whitney, **p=0.0012 (significativo)** | **SÍ** |
| edad (ANOVA) | p=0.300 (NS) | Kruskal-Wallis, p=0.154 (NS) | No |
| país (ANOVA) | p=0.00337 (sig.) | Kruskal-Wallis, p=0.0182 (sig.) | No |
| CF a mano vs. Surprise (pareado) | p=0.000095 (sig.) | Wilcoxon, p=0.000245 (sig.) | No |
| edad_fine, 9 grupos (6.1) | p=0.393 (NS) | Kruskal-Wallis, p=0.239 (NS) | No |

### Textos markdown editados por esto

Se agregó una aclaración corta al final de cada uno, sin borrar lo que ya tenían escrito:

- **Interpretación de historial corto/largo**: reescrita del todo, ahora dice que sí hay diferencia significativa pero de magnitud chica (~0.02 de MAE).
- **Interpretación edad/país, interpretación pareado, interpretación edad_fine (6.1)**: se les agregó una frase aclarando qué test corrió realmente y que la conclusión no cambia.

## 2. Contradicción en el resumen final (sección 7)

Encontrada de yapa durante la revisión: la celda decía primero *"no hay diferencias significativas entre los modelos"* y después, sobre la misma comparación, *"se vio diferencia significativa"*. Se corrigió la primera frase para que diga lo que el test real confirma (Wilcoxon, p=0.000245, sí hay diferencia significativa, el de coseno predice mejor).

También se ajustó la frase que decía que el modelo era "robusto" porque el rendimiento "no variaba" entre usuarios con pocas/muchas interacciones, ya no es cierto con el test correcto (Mann-Whitney, p=0.0012), así que ahora dice que sí hay diferencia estadística pero de magnitud chica y sin relevancia práctica.

## 3. Sección 6.1 faltaba el punto "2."

La lista de "las tres modificaciones propuestas" saltaba de "1." a "3.". Se agregó el "2.": una frase describiendo la nueva sección 6.2 (variación de k).

## 4. Sección 6.2 nueva (5 celdas, entre 6.1 y "## 7")

- Markdown de introducción.
- `make_user_user_predictor(k)` + `mae_by_user_for_k(k)`: reconsultan el mismo `user_model` ya entrenado en 4.4 con otro `k` (10 y 60), sin reentrenar nada. Evalúa MAE global con k=10/30/60.
- Celda de fairness por país para los 3 valores de k (reusa `fairness_summary`, ya existente).
- Celda de comparación pareada k=10 vs. k=60 (reusa `paired_model_ttest`, ya modificada).
- Markdown de interpretación con los números reales:
  - MAE global: 1.3999 (k=10) / 1.4077 (k=30) / 1.4139 (k=60). k chico predice mejor en promedio.
  - Fairness por país: worst_to_best_ratio 1.45 (k=10) → 1.43 (k=60). k grande reparte mejor el error entre países.
  - Trade-off explícito entre precisión promedio y fairness. Alemania sigue siendo el país peor predicho en los tres casos (Wilcoxon k=10 vs. k=60: p<0.001).

## Estado final

- 85 celdas, corridas de punta a punta, **0 errores**.
- `SistemasDeRecomendacion.ejecutada.ipynb` es ahora copia exacta (código + outputs) de la `CORREGIDA` final.
- No se tocó `aggregate_profiles` (sigue siendo código no usado en 6.1) ni nada de las secciones 1-5.2/5.4.
- Nada está commiteado todavía.

## Pendiente de decisión (no tocado)

- `aggregate_profiles` (celda de la sección 6.1) calcula `least_misery_age_fine`, que no se usa en el análisis real de esa sección (el ANOVA/fairness de 6.1 usa `mae_by_user_fine_age`, un merge directo). Es resabio del enfoque de "Profile Aggregation" que se descartó según el prompt de la sección 8. No rompe nada, pero es código muerto.
