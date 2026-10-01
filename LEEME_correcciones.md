# Correcciones al Jupyter Book publicado

Revisión de https://jserrano3010.github.io/ML_JB_JSyEY_H2O_SG/ (30 de septiembre de 2026).

El libro compila y se ve bien. Los errores son de **contenido**: textos que quedaron del diseño regional (varios sitios) y no aplican a un solo sitio, afirmaciones sin un resultado impreso que las respalde y dos interpretaciones incorrectas.

## Cómo aplicar las correcciones

1. Copiar los archivos de esta carpeta sobre el proyecto, **reemplazando** los existentes:
   - `notebooks/02_eda.ipynb` y `notebooks/03_modelo_base.ipynb` (ya vienen ejecutados, con las salidas nuevas)
   - `capitulos/01_problema.md` y `capitulos/limitaciones.md`
2. No reemplazar `intro.md` ni `_toc.yml`: se conservan los cambios que ustedes hicieron (portada con ilustración, índice).
3. Compilar y publicar de nuevo:
   ```
   jupyter-book build .
   ghp-import -n -p -f _build/html
   ```

## Errores encontrados y corrección aplicada

### 2. Análisis exploratorio (`02_eda`)

| # | Dónde | Error | Corrección |
|---|---|---|---|
| 1 | Introducción | Decía que el EDA usa "los sitios regionales hasta 2020" y que "la finca se reservó"; con un solo sitio el EDA usa la finca en 1981–2020. | Texto que distingue este entregable (un sitio) del diseño regional. |
| 2 | 2.1, segunda figura | "Climatología mensual por punto" con una sola línea, sin información nueva. | Con un sitio se grafica lluvia y ET0 media mensual. |
| 3 | 2.2, interpretación | Citaba la proporción de días lluviosos por mes (2 % a 82 %) sin tabla que la mostrara. | Nueva celda con esa tabla. |
| 4 | 2.3, interpretación | Citaba el efecto de la década (ε² ≈ 0,02) pero la prueba no se calculaba. | Kruskal-Wallis por década agregado (ε² = 0,018). |
| 5 | 2.4, interpretación | Citaba PC1 = 46 % y PC2 = 16 % sin imprimirlos. | Se imprime la varianza explicada. |
| 6 | 2.5, texto | Decía que el ONI se rezaga 60 días; son 75 y el ONI no está en este dataset. | Texto corregido. |
| 7 | 2.5, tabla de disponibilidad | Los armónicos `sin2`, `cos2`, `sin3`, `cos3` aparecían como "pasado, rezago 3 d"; son calendario. | Clasificados como "calendario". |
| 8 | 2.5, verificación | Imprimía "Sitios presentes en ambos: FINCA" y "la celda más cercana se excluyó", que no aplica a un sitio. | Mensaje condicional: con un sitio la separación es cronológica. |
| 9 | 2.6, ONI | La celda de correlación cruzada con el ONI no mostraba nada. | Mensaje explícito de que el ONI no está en este dataset. |
| 10 | 2.6, interpretación | ACF, semanas significativas, coeficiente de variación, año más seco, tendencias de lluvia y temperatura citados sin respaldo impreso; "13 semanas" y "1982: 317 mm" no coincidían con el cálculo semanal. | Nueva tabla resumen (ACF₁ = 0,48; 12 semanas; CV = 0,48; 1982 ≈ 320 mm; +168 mm/década, p = 0,007; +0,11 °C/década, p = 0,002) y texto ajustado. |
| 11 | 2.6, "Heterogeneidad entre puntos" | Boxplot de un solo punto. | Mensaje: se analiza con la grilla regional. |
| 12 | 2.7, primeras celdas | Matriz de distancias 1×1, "vecino más cercano: inf" y un mapa con un punto. | Celdas condicionadas a tener varios sitios. |
| 13 | 2.7, texto | "Con unos 40 sitios la prueba tiene potencia razonable" en un análisis de un sitio. | "Con la grilla regional (unos 40 sitios)…". |

### 3. Modelo base (`03_modelo_base`)

| # | Dónde | Error | Corrección |
|---|---|---|---|
| 14 | 3.1, interpretación | Decía que el SVR supera a la climatología **en todos los horizontes**. Falso: a 52 semanas el IC incluye el cero y en validación solo es consistente hasta 4 semanas. | Afirmación acotada a lo que muestran las tablas. |
| 15 | 3.1, tabla | La columna MAPE está en fracción (25,77 = 2.577 %) sin aclararlo. | Columna renombrada "MAPE (fracción, y>0)" y aclaración en el texto. |
| 16 | 3.3, interpretación | Decía que 2021 fue "un año más cercano a lo habitual". **Falso:** 2021 tuvo 8,6 mm/semana, menos de la mitad de la media (18,5); fue anormalmente seco, y eso pese a ser año de La Niña. | Texto corregido; el hallazgo refuerza la sospecha sobre la calidad de la serie reciente de POWER. |
| 17 | 3.4, interpretación | Afirmaba media de residuos positiva y autocorrelación a un rezago sin imprimirlas. | Se imprimen: media = 1,94 mm; ACF₁ = 0,185 frente a banda ±0,122. |
| 18 | 3.4, curva de aprendizaje | Decía "brecha estable, no hay sobreajuste" sin justificar una brecha grande (19 frente a 33 mm). | Se explica y se imprime: la validación 2016–2020 es más lluviosa (25,8 frente a 17,4 mm/semana) y la climatología también falla ahí (36 mm). |

### Capítulos

| # | Dónde | Error | Corrección |
|---|---|---|---|
| 19 | Planteamiento, objetivos | Prometían dataset espacio-temporal con ONI, análisis espacial y validación en sitio no visto, que este entregable no hace. | Objetivos del proyecto con el alcance de cada entregable entre paréntesis. |
| 20 | Planteamiento, horizonte *n* | Definía *n* solo con el criterio de prueba, pero el notebook 03 adopta el mínimo entre prueba y validación (n = 4). | Definición con los dos criterios y el mínimo. |
| 21 | Limitaciones | No mencionaba las tres limitaciones principales de este entregable. | Agregadas: un solo sitio con menos de 20.000 observaciones, calidad de la serie reciente (2021 seco, 2022–2025 muy lluvioso, salto de la ET0) y ausencia del ONI. |

## Sin cambios necesarios

Marco de referencia, planificador, notebook 04, referencias y la portada que ustedes agregaron.
