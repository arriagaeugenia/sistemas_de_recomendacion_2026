# Revisión de la notebook final — Sistemas de Recomendación 2026

Revisión hecha contra: el enunciado (`Sistemas de Recomendación 2026 - Enunciado.pdf`) y el PDF de diseño entregado para la cursada (`TP Sistemas de Recomendación.pdf`). La notebook (`SistemasDeRecomendacion.ipynb`) se ejecutó de punta a punta en un entorno limpio para confirmar los hallazgos, no solo se leyó el código. Este documento se fue actualizando a medida que se implementaron las correcciones — refleja el estado **actual** de la notebook (71 celdas), no el de la entrega original.

---

## Qué falta para la entrega final

De los ocho puntos identificados, quedan dos abiertos:

- [ ] **1.B — Declaración de IA generativa en la notebook.** Pendiente de discutir en equipo (ver detalle en la sección 1). Es un requisito explícito del enunciado ("en ambas entregas") y hoy no está en la notebook, solo en el PDF de diseño.
- [ ] **Confirmar cuál notebook se sube a Classroom.** Hay dos archivos en la carpeta: `SistemasDeRecomendacion.ipynb` (71 celdas, la que se corrigió y está al día) y `SistemasDeRecomendacion.ejecutada.ipynb` (36 celdas, una versión vieja e incompleta que **no** se tocó en esta revisión). Subir solo la primera, y considerar borrar o renombrar la segunda para que no genere confusión en la corrección.

Todo lo demás (1.A, 1.C, 1.D, 1.E, y los dos hallazgos de bugs reales) ya está resuelto e incorporado a la notebook, con la re-ejecución completa hecha después de cada cambio. El detalle de cada uno está más abajo.

Quedan además, sin resolver por decisión explícita del equipo (no son errores, son alcance descartado a propósito — ver el detalle de cada uno en las secciones correspondientes):
- La Cross-Validation completa (tercera modificación de la tarea 5) — quedó preparada (`cv_splits`, sección 6) pero sin ejecutar, por su costo de cómputo.
- El umbral de "libro popular" que hace que `serendipity@k` dé siempre 0 — quedó documentado como limitación, sin ajustar.
- Un fallback simétrico en `recommend_user_user`/`recommend_content_for_user` para usuarios cold-start (hoy devuelven listas vacías, a diferencia de `predict_user_user`, que sí tiene fallback) — quedó documentado como hallazgo de fairness en sí mismo, no como algo a parchear.

Antes de subir la entrega: si tocan cualquier celda más (incluida la sección de IA generativa), correr **Kernel → Restart & Run All** una vez más como último paso, para que los outputs guardados sigan correspondiendo al código.

---

## 1. Qué falta respecto al enunciado (requisitos explícitos)

- [x] **A. No usan ninguna librería de recomendación existente — incumplimiento directo. RESUELTO.**
  El enunciado pide explícitamente "Surprise, RecBole, ClayRS, Elliot entre otras" y "ejecutar uno o más algoritmos usando una librería preexistente". `requirements.txt` solo tenía `pandas, numpy, matplotlib, seaborn, scipy, scikit-learn, nltk` — nada de recomendación. Todo (popularidad, contenido, colaborativo usuario-usuario) estaba implementado a mano con `sklearn.NearestNeighbors` y similitud coseno artesanal.


  *⚠️​SI PEDIA USAR UNA LIBRERIA PREEXISTENTE, DIRECTAMENTE SACARIA LO DE "HECHO A MANO" QUE ERA LO DEL COSENO Y DEJARIA LO QUE ESTA IMPLEMENTADO CON UNA LIBRERIA*


  **Qué se agregó:** una nueva sección **5.5 "Validación con librería externa (Surprise)"**, con `surprise.KNNBasic` (similitud coseno, user-based) entrenado sobre el mismo subconjunto de usuarios/libros activos (`collab_train`, sección 4.4) y evaluado sobre exactamente el mismo test set (`evaluable_test`, sección 5.1) que el CF hecho a mano, para que la comparación sea directa. Se agregó `scikit-surprise` a `requirements.txt`. Se reutilizaron las funciones de test estadístico ya definidas (`independent_ttest`, `one_way_anova`, `fairness_summary`), y de paso se usó por primera vez `paired_model_ttest` (definida en 5.3 pero nunca invocada — ver punto D).

  **Resultado final de la comparación** (números tal como quedaron después de todos los cambios posteriores, incluido el fix del punto 4 más abajo):

  | | MAE global |
  |---|---|
  | CF a mano (5.1) | 1.3316 |
  | Surprise KNNBasic (5.5) | 1.4500 |

  El CF a mano da mejor MAE. No es un bug: la implementación manual centra cada rating restando la media del usuario antes de calcular la similitud (ver `predict_user_user`, sección 4.4), algo que `KNNBasic` de Surprise **no hace** — el equivalente algorítmico más parecido sería `KNNWithMeans`, no `KNNBasic`. No es "nuestro código le gana a una librería profesional" sin matices, sino dos variantes distintas de user-user CF sobre los mismos datos. Si en algún momento quieren una comparación más pareja, cambiar `KNNBasic` por `KNNWithMeans` en la celda de 5.5 es un cambio de una palabra. El paired t-test entre ambos modelos (10.693 usuarios evaluados por los dos) da altamente significativo (p≈1.5e-08): confirma que producen errores sistemáticamente distintos, no por azar.

  El hallazgo más interesante que salió de esta comparación fue sobre el criterio de historial — ver el punto 4 de la sección 3.

- [ ] **B. Sin declaración de IA generativa en la notebook. PENDIENTE.**
  El enunciado dice "en **ambas** entregas". La sección está muy bien hecha en el PDF de diseño (con prompts reales), pero no aparece en ningún lado de la notebook final. Si usaron alguna IA para escribir código o texto de la notebook, hay que agregar la sección (pueden reciclar el mismo formato del PDF). Si no usaron ninguna para la notebook en sí, alcanza con una aclaración breve de que el uso de IA generativa se limitó a la etapa de diseño.

- [x] **C. Falta el "resumen y análisis de los resultados" final. RESUELTO.**
  Es un ítem explícito de la entrega ("La notebook tiene que tener... un resumen y análisis de los resultados"). La notebook terminaba en la celda de extensiones (sección 6) con código, sin ningún cierre narrativo.

  **Qué se agregó:** la sección **"## 7. Resumen y análisis de resultados"** al final de la notebook, con: comparación de los tres modelos (incluida la aclaración del supuesto de diversidad del diseño que resultó incorrecto, y por qué `serendipity@k` da 0 en los tres), el resumen de fairness por edad/país/historial, la comparación con Surprise, los resultados de las dos extensiones ejecutadas (sección 6.1/6.2), una lista honesta de limitaciones, y una conclusión general. Todo tomado de outputs ya calculados en la notebook — nada inventado. Se actualizó varias veces a medida que los fixes posteriores (punto 4, tarea 5, punto 1.E) cambiaron los números — hoy está al día con el estado final de la notebook.

- [x] **D. La tarea 5 del enunciado ("explorar impacto de modificaciones simples") — PARCIALMENTE RESUELTO (decisión de equipo).**
  La sección 6 dejaba *preparada* la infraestructura (5 splits con semillas distintas, tabla descriptiva least_misery vs. average, bins de edad más finos) pero nunca reentrenaba ni reevaluaba el modelo con esas variantes.

  **Decisión tomada:** de las tres modificaciones del diseño, se ejecutaron las dos que no requieren reentrenar el modelo colaborativo, y se dejó la Cross-Validation preparada pero sin ejecutar, por su costo de cómputo (reentrenar el pipeline completo 5 veces).

  - **6.1 Granularidad etaria (4 vs. 9 grupos):** ANOVA sigue sin ser significativa (p=0.84 fino vs. p=0.82 base). La brecha absoluta crece (0.29 vs. 0.05) pero no hay evidencia estadística de que sea sistemática — con más grupos, menos usuarios por grupo, más ruido. Segmentar más fino no cambia la conclusión de fairness por edad.
  - **6.2 Average vs. Least Misery — impacto en precisión:** usando el valor agregado del grupo como predicción del rating real de cada integrante: MAE=2.4683 (Least Misery) vs. MAE=1.5631 (Average), diferencia muy significativa (paired t-test p≈0, n=41.671). Confirma cuantitativamente que Average predice mejor en promedio — tal como anticipaba el diseño — sin que eso invalide la elección de Least Misery para las recomendaciones grupales (esa comparación no mide lo que Least Misery optimiza: evitar la insatisfacción extrema de un integrante).

  **Cross-Validation, sin ejecutar (decisión consciente):** reentrenar el pipeline del colaborativo (activos, matriz, vecinos, predicción) 5 veces y comparar media/desvío del MAE contra el holdout único queda como trabajo futuro si da el tiempo — es la parte más cara en cómputo de toda la tarea 5. El `paired_model_ttest` ya está probado (se usó en 5.5), así que la maquinaria para comparar configuraciones de a pares está lista para reutilizar acá si la retoman.

- [x] **E. Las métricas de calidad (diversidad, cobertura, serendipia) no están desagregadas por grupo. RESUELTO.**
  El enunciado pide (tarea 3) evaluar métricas estándar "de forma separada para cada grupo" en general. Antes solo el MAE se calculaba por grupo (sección 5.1); `quality_results` (sección 5.2) daba un solo valor global por modelo.

  **Qué se agregó:** una tabla "Métricas de calidad por grupo" en la sección 5.2, para edad/país/historial, reutilizando las listas de recomendaciones ya generadas y excluyendo grupos con menos de 5 usuarios evaluados (mismo criterio que `fairness_summary`). Se subió `EVAL_USERS` de 100 a 300 para que los subgrupos (sobre todo país, con 11 categorías) tuvieran una cantidad razonable de usuarios cada uno.

  **Resultado — el hallazgo más contundente de todo el análisis de fairness:** desagregado por historial, el grupo `short` tiene **acierto y cobertura en cero, tanto en contenido como en colaborativo** (contra 25% de acierto y 20.6% de cobertura de `long` en colaborativo). No es que el sistema los prediga peor (eso ya lo mostraba el MAE, punto 4) — directamente **no les genera ninguna recomendación**. Para colaborativo la causa ya se conocía (`MAX_USERS`), pero en contenido aparece por una razón nueva: `recommend_content_for_user` necesita al menos un rating ≥7 en train, y con 2-4 ratings totales muchos usuarios de historial corto no tienen ninguno.

---

## 2. Incoherencias diseño vs. notebook

- **Cross-Validation:** el diseño promete K-Folds "de verdad" (cada dato usado una vez como test). Lo que se preparó en la sección 6 es en realidad 5 holdouts independientes con semillas distintas (Monte Carlo / repeated random subsampling), no K-Fold. Como esta parte quedó sin ejecutar (por decisión del equipo, ver punto 1.D), hoy no se nota en los outputs, pero si la retoman conviene nombrarla bien o cambiarla.

- **Hipótesis de diversidad de popularidad, contradicha por el propio resultado (no es un bug, es un hallazgo):** el diseño anticipaba "los modelos de popularidad registrarán una diversidad... muy baja". El resultado real da lo contrario: `diversity_author ≈ 0.98` para el modelo de popularidad, más alto que colaborativo y contenido. No hay error de cálculo — la métrica implementada (variedad de autores *dentro* de una misma lista de 10) es la que describe el propio diseño, pero la intuición previa confundía eso con diversidad *entre usuarios* (personalización), que es un concepto distinto. Ya quedó explicado en la sección 7 de la notebook.

- El diseño ya dejaba ambigua la elección de librería ("mediante la librería de recomendación seleccionada", sin nombrarla) — resuelto agregando Surprise (ver punto 1.A).

---

## 3. Bugs y resultados sin sentido — confirmados ejecutando la notebook

### 🔴 Bug #1 (confirmado y CORREGIDO): grupo "historial corto" vacío → t-test en NaN

Al ejecutar la notebook tal cual estaba entregada, el t-test de historial corto vs. largo daba `NaN`/`NaN`, y la tabla de MAE por `history_group` tenía una sola fila ("long") — un resultado degenerado, no evidencia de equidad.

**Causa raíz:** `history_threshold` se calculaba como la mediana de `interaction_count` entre usuarios con al menos 1 rating (daba `1.0`), haciendo que "short" = usuarios con 0 o 1 rating. Pero `split_by_user` (el holdout) descarta por completo a cualquier usuario con menos de 2 ratings, así que ningún usuario "short" podía aparecer nunca en `test_ratings` ni en ninguna métrica calculada sobre test. No era casualidad de semilla: era estructural.

*⚠️​NO ENTENDI BIEN PORQUE PASA ESTO, LOS USUARIOS CON 1 RATING DEBERIAN SER INCLUIDOS EN SHORT, NOSE SI PASA*

**Fix aplicado:** se recalculó el umbral como la mediana de `interaction_count` solo entre usuarios evaluables (`interaction_count >= 2`), que da `4.0` en vez de `1.0` (celda de sección 4.3, con su explicación en markdown actualizada).

### 🟡 Hallazgo relacionado — el fix de arriba no alcanzaba del todo, causa más profunda (CORREGIDO)

Después del primer fix, el t-test seguía dando NaN: el modelo colaborativo (sección 4.4) arma su matriz tomando `MAX_USERS = 5.000` usuarios **más activos**, y el menos activo de ellos ya tenía 13 ratings — muy por encima de cualquier umbral de "historial corto". El recorte a los usuarios más activos, hecho por rendimiento, excluía por completo a la población que el criterio de comportamiento quería estudiar.

**Fix aplicado (decisión de equipo, entre 3 opciones planteadas):** en `evaluable_test` (sección 5.1) se sacó el filtro `.isin(user_ids)`, dejando que `predict_user_user` use el fallback que ya tenía escrito y sin usar (media global) para usuarios fuera del pool. Se agregó una columna `used_fallback` y un print del porcentaje de filas que la usan (**42.2%**).

**Resultado:** `short` (3675 usuarios) y `long` (7018 usuarios) ya tienen representación real. Es el hallazgo más interesante de toda la notebook:
- Con el **CF a mano**, `short` sale peor que `long` (MAE 1.42 vs. 1.39), sin significancia (t-test p=0.12).
- Con **Surprise**, se **invierte**: `short` sale *mejor* que `long` (MAE 1.37 vs. 1.47), y ahí sí es significativo (p=0.00002).

La pregunta de cold-start que motivó este criterio desde el diseño tiene respuestas distintas —y hasta opuestas— según el algoritmo usado. Quedó documentado en la sección 5.5 y en el resumen (sección 7) como algo para investigar más, no como conclusión cerrada.

**Efecto colateral descubierto de paso (documentado, no parcheado):** este mismo cambio corrió también la sección 5.2 (`quality_results`), porque sus usuarios de evaluación se muestrean del mismo `evaluable_test`. `recommend_user_user` y `recommend_content_for_user` no tienen fallback (a diferencia de `predict_user_user`) y devuelven listas vacías para usuarios cold-start — lo cual terminó siendo, desagregado por grupo (punto 1.E), el hallazgo más fuerte de todo el análisis: `short` con acierto y cobertura en cero. El equipo decidió no agregarles un fallback simétrico y dejarlo documentado como hallazgo de fairness en sí mismo.

### 🟡 Hallazgo #2 (confirmado y ya resuelto solo, con la re-ejecución): la notebook entregada no reflejaba su propio código

Antes de tocar nada, se comparó los outputs guardados contra el código de las celdas: la tabla de MAE por edad en la notebook entregada mostraba grupos "18-29", "30-44", "45-64", "65-100" (5 categorías), pero el código de `age_group` definía 4 categorías ("18-35", "36-50", "51-100"). Los `execution_count` de las celdas confirmaron la causa: la sección de evaluación se había ejecutado *antes* que la de definición de grupos en la última pasada — es decir, editaron código de arriba después de correr la evaluación, pero nunca la volvieron a correr.

El enunciado pide explícitamente "el output de las celdas"; unos outputs que no corresponden al código visible es justo el tipo de cosa que salta a la vista en una corrección. Se resolvió reejecutando toda la notebook de punta a punta (y se volvió a hacer después de cada cambio posterior). **Antes de la entrega final, correr Restart & Run All una vez más como último paso.**

### 🟡 Hallazgo #3 (sin corregir, a criterio del equipo): `serendipity@k = 0.0` en los tres modelos — no aporta información

`popularity_cutoff = data['ratings_count'].quantile(0.70)` da apenas **2 votos** — el dataset es tan long-tail (75% de los libros tienen 1-2 ratings) que "estar en el 30% más popular" equivale a "tener más de 1 rating". Casi cualquier libro con señal suficiente para ser recomendable ya cae en el bucket "popular", así que el bucket "no popular y relevante" (serendipia) queda vacío para los tres modelos por igual, en conjunto y en todos los subgrupos evaluados en 1.E. No es un error de cómputo — es un umbral que, dada la dispersión del dataset, termina siendo demasiado laxo para discriminar nada.

Sugerencia si quieren que la métrica diga algo: un corte absoluto más alto (p.ej. `ratings_count >= 10`) en vez de un percentil. Quedó documentado en la sección 7, sin tocar el código.

---

## 4. Qué está bien (y vale la pena mantener)

- El filtrado por contenido funciona muy bien: "Harry Potter" → los 3 primeros resultados son ediciones de *La Piedra Filosofal* y el resto son otros libros de la saga, con similitud > 0.6. Los ejemplos de "The Giving Tree" y el de postcards también están bien interpretados en el propio texto de la notebook — la autocrítica sobre las limitaciones (pocos atributos textuales) es acertada y honesta.
- Buenas prácticas de ingeniería: el `assert` de que train/test son disjuntos (sección 3), las decisiones de umbral documentadas y justificadas en prosa (`min_votos=250`, límites de `MAX_USERS`/`MAX_CONTENT_BOOKS` con su motivo), el manejo separado de ratings implícitos (0) vs explícitos.
- El framework de fairness (`absolute_gap` + `worst_to_best_ratio` junto al test estadístico, sección 5.4) es un buen enfoque — va más allá de mirar solo el p-valor.
- La declaración de IA generativa del **PDF de diseño** es un ejemplo notablemente completo (prompts reales, respuestas, justificación de por qué Least Misery y no Average) — es el modelo a reciclar para la sección que falta en la notebook (punto 1.B).
- Toda la notebook (71 celdas a esta altura) corre de punta a punta en un entorno limpio sin ningún error — la lógica central es sólida y reproducible.
- El hallazgo de `short` con acierto/cobertura en cero (punto 1.E) y la inversión short/long entre CF manual y Surprise (punto 4) son, con la evidencia que hay hoy en la notebook, los dos resultados más ricos para la discusión de fairness — vale la pena que la presentación oral o cualquier resumen adicional los destaque.

---

## 5. Registro de todos los cambios aplicados a `SistemasDeRecomendacion.ipynb`

En orden cronológico. Después de cada cambio se reejecutó la notebook completa (Kernel → Restart & Run All) salvo que se indique lo contrario.

1. **Fix del umbral de `history_group`** (sección 4.3): la mediana de `interaction_count` pasó a calcularse solo sobre usuarios evaluables (`>= 2` ratings). Umbral `1.0 → 4.0`. Markdown de explicación actualizado.
2. **Sección 5.5 nueva — Surprise KNNBasic:** entrenamiento y evaluación de un segundo CF con la librería Surprise sobre los mismos datos que el CF manual, reutilizando `independent_ttest`, `one_way_anova`, `fairness_summary`, y usando por primera vez `paired_model_ttest`. Se agregó `scikit-surprise` a `requirements.txt`.
3. **Sección 7 nueva — Resumen y análisis de resultados:** 4 celdas markdown al final de la notebook (sin código).
4. **Fix del filtro de `evaluable_test`** (sección 5.1): se sacó `.isin(user_ids)`, dejando que `predict_user_user` use su fallback para usuarios fuera del pool de `MAX_USERS`. Se agregó la columna `used_fallback` y un print del porcentaje. Markdown de la sección 5.1 actualizado. Se actualizó también la nota de cierre de la sección 5.5 (ya no dice que `short` está ausente).
5. **Secciones 6.1 y 6.2 nuevas** (extensiones): comparación de ANOVA/fairness con granularidad etaria fina vs. base, y comparación de precisión Average vs. Least Misery contra ratings reales de test. Markdown de la sección 6 actualizado para reflejar qué se ejecutó y qué no (Cross-Validation).
6. **Tabla "Métricas de calidad por grupo" nueva** (sección 5.2): desagregación de hit-rate/diversidad/serendipia/cobertura por edad, país e historial, reutilizando las recomendaciones ya generadas. `EVAL_USERS` subido de 100 a 300.
7. **Sección 7 actualizada varias veces** para reflejar los números y hallazgos de los pasos 4, 5 y 6 a medida que se fueron aplicando (el resumen no se actualiza solo — quedó al día manualmente después de cada cambio).

**Backup del original sin ningún cambio:** `SistemasDeRecomendacion.ANTES_DEL_FIX.ipynb` (guardado antes del primer fix, punto 1 de esta lista).

**No se tocó** `SistemasDeRecomendacion.ejecutada.ipynb` — es una versión vieja e incompleta (36 celdas, le falta desde la sección 4.3 en adelante). Ver la acción pendiente al principio de este documento sobre cuál subir a Classroom.
