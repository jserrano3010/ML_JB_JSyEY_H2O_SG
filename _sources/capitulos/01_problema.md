# Planteamiento del problema

## Problema

El departamento del Atlántico combina una lluvia anual moderada con una distribución muy desigual: dos temporadas húmedas (abril–junio y agosto–noviembre) separadas por una temporada seca prolongada entre diciembre y marzo y un periodo más seco a mitad de año (veranillo). Para una finca de Sabanagrande, sembrar más área de la que el agua disponible puede sostener expone al cultivo a estrés hídrico, sobre todo en floración, y al agricultor a perder la inversión.

La decisión se toma antes de sembrar y abarca el ciclo completo del cultivo: cuántas hectáreas sembrar de cada cultivo, en qué fecha y cuánta agua reservar. A esa escala la lluvia diaria no es predecible, pero la lluvia semanal sí lo es parcialmente durante algunas semanas, gracias al ciclo estacional y a la memoria del suelo y de la lluvia reciente.

## Pregunta de investigación

¿Con qué habilidad, frente a la climatología y a la persistencia, pueden un modelo lineal (SVR), uno no lineal (random forest) y su combinación (modelo híbrido) pronosticar la lluvia semanal en Sabanagrande a partir de variables agroclimáticas de NASA POWER, en particular las semanas de aguaceros fuertes, y cómo se traduce ese pronóstico, con su incertidumbre, en el área sembrable y el agua que la finca debe reservar para un ciclo de cultivo?

## Objetivos

**General.** Desarrollar un sistema de pronóstico de lluvia y apoyo a la decisión que estime, para cultivos, hectáreas y fecha de siembra definidos por el agricultor, la reserva de agua necesaria y el área máxima sembrable con un nivel de seguridad dado.

**Específicos.**

1. Construir un dataset diario de variables agroclimáticas de NASA POWER (1981–2025) para la celda de la finca.
2. Caracterizar mediante un EDA la estacionalidad, la dependencia temporal y la deriva del clima en la finca.
3. Implementar líneas base (Dummy, climatología, ingenuo estacional y persistencia semanal) y un modelo base lineal (SVR) para el pronóstico semanal de lluvia, con validación cronológica.
4. Evaluar un modelo no lineal (random forest) y un modelo híbrido que combina SVR y random forest, frente a las líneas base, con énfasis en las semanas de lluvia extrema, y elegir el modelo final con la validación cruzada temporal.
5. Determinar el horizonte *n* hasta el cual el pronóstico supera a la climatología con 95 % de confianza.
6. Traducir el pronóstico, con su incertidumbre, en el área sembrable y la reserva de agua mediante el balance hídrico FAO-56 y un modelo del reservorio.

## Arquitectura del sistema

| Nivel | Pregunta | Objetivo (variable) | Horizonte | Modelos |
|---|---|---|---|---|
| 1. Pronóstico semanal | ¿Cuánto lloverá la semana *h*? | Lluvia semanal (mm), regresión | 1 a *n* semanas | Híbrido SVR + random forest (elegido), frente a SVR, random forest, gradient boosting y las referencias Dummy, climatología, ingenuo estacional y persistencia |
| 2. Decisión | ¿Cuánto sembrar y cuánta agua reservar? | — | Ciclo del cultivo | Balance hídrico FAO-56 y modelo del reservorio con escenarios del pronóstico |

El pronóstico estacional (terciles de la temporada con el índice ONI) y el análisis con una grilla regional de sitios quedan como trabajo futuro (ver Limitaciones).

## Variable objetivo del nivel 1

Para cada fecha de emisión *t* (un pronóstico por semana, los lunes), el objetivo es la lluvia acumulada en la finca en la semana *h* posterior:

$$y_h(t) = \sum_{d=7(h-1)+1}^{7h} P_{t+d}\qquad [\text{mm}]$$

Como 1 mm sobre 1 ha equivale a 10 m³, el mismo valor por hectárea es $10\,y_h$ m³/ha.

## Horizonte confiable *n*

$$SS_h = 1-\frac{RMSE_{\text{modelo},h}}{RMSE_{\text{climatología},h}}$$

Se usan dos criterios y se adopta el más conservador:

- **Prueba:** $n_{\text{prueba}} = \max\{h:\ \text{IC}_{95}(SS_h)_{\text{inf}} > 0\}$, con intervalos por bootstrap de bloques en 2021–2025.
- **Validación:** $n_{\text{val}}$ = mayor horizonte en el que el SVR supera a la climatología en al menos 4 de los 5 bloques de la validación cruzada temporal (1981–2020).
- $n = \min(n_{\text{prueba}},\ n_{\text{val}})$.

Para las semanas posteriores a *n* se usan los cuantiles climatológicos.

## Diseño de validación

Partición cronológica: entrenamiento 1981–2020, con purga de las filas cuya semana objetivo cae en 2021, y prueba 2021–2025, usada una sola vez al final. La validación cruzada es de ventana creciente con una separación de 7h + 3 días. Todas las predictoras usan datos con al menos 3 días de antigüedad (latencia de publicación de NASA POWER).

## Justificación del dataset

NASA POWER ofrece series diarias continuas desde 1981 de todas las variables necesarias para calcular la evapotranspiración FAO-56 y para caracterizar el estado del suelo, con acceso abierto y reproducible por API. Sus 45 años permiten separar un periodo de prueba de cinco años completos sin perder la historia necesaria para estimar la climatología y entrenar los modelos.
