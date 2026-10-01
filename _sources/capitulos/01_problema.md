# Planteamiento del problema

## Problema

El departamento del Atlántico combina una lluvia anual moderada con una distribución muy desigual: dos temporadas húmedas (abril–junio y agosto–noviembre) separadas por una temporada seca prolongada entre diciembre y marzo y un periodo más seco a mitad de año (veranillo). Para una finca de Sabanagrande, sembrar más área de la que el agua disponible puede sostener expone al cultivo a estrés hídrico, sobre todo en floración, y al agricultor a perder la inversión.

La decisión se toma antes de sembrar y abarca el ciclo completo del cultivo: cuántas hectáreas sembrar de cada cultivo, en qué fecha y cuánta agua reservar. A esa escala la lluvia diaria no es predecible, pero sí lo es parcialmente la tendencia de la temporada, gracias a la influencia de El Niño–Oscilación del Sur (ENSO).

## Pregunta de investigación

¿Con qué habilidad, frente a la climatología, pueden los modelos de Machine Learning pronosticar la lluvia semanal y la categoría de lluvia de la temporada en Sabanagrande a partir de variables agroclimáticas e índices oceánicos, y cómo se traduce ese pronóstico, con su incertidumbre, en el área máxima sembrable y el volumen de agua que la finca debe reservar para asegurar un ciclo de cultivo?

## Objetivos

**General.** Desarrollar un sistema de pronóstico de lluvia y apoyo a la decisión que estime, para cultivos, hectáreas y fecha de siembra definidos por el agricultor, la reserva de agua necesaria y el área máxima sembrable con un nivel de seguridad dado.

**Específicos del proyecto** (el alcance de cada entregable se indica entre paréntesis).

1. Construir un dataset diario de variables agroclimáticas de NASA POWER (1981–2025) para la finca (entregable 1) y para una grilla regional del Caribe colombiano con el índice ONI (entregable 2).
2. Caracterizar mediante un EDA la estacionalidad, la dependencia temporal y la deriva del clima en la finca (entregable 1), y la dependencia espacial y la influencia de ENSO con la grilla regional (entregable 2).
3. Implementar líneas base y un modelo base para el pronóstico semanal de lluvia, validado de forma cronológica (entregable 1) y luego en un sitio no visto (entregable 2).
4. Determinar el horizonte *n* hasta el cual el pronóstico supera a la climatología con 95 % de confianza.
5. Implementar un planificador que combine el pronóstico con el balance hídrico FAO-56 y un modelo del reservorio, con los coeficientes de agua de cada cultivo como entrada del agricultor.

## Arquitectura del sistema

| Nivel | Pregunta | Objetivo (variable) | Horizonte | Modelo base | Entregable |
|---|---|---|---|---|---|
| 1. Semanal | ¿Cuánto lloverá la semana *h*? | Lluvia semanal (mm), regresión | 1 a *n* semanas | SVR lineal frente a climatología | 1 |
| 2. Estacional | ¿La temporada será seca, normal o lluviosa? | Tercil de lluvia de la temporada, clasificación | 1 a 12 meses | Regresión logística multinomial frente a climatología | 2 |
| 3. Decisión | ¿Cuánto sembrar y cuánta agua reservar? | — | Ciclo del cultivo | Simulación por escenarios ponderados | 1 (herramienta) |

## Variable objetivo del nivel 1

Para cada sitio *s* (en este entregable, solo la finca) y fecha de emisión *t* (un pronóstico por semana, los lunes), el objetivo es la lluvia acumulada en la semana *h* posterior:

$$y_h(s,t) = \sum_{d=7(h-1)+1}^{7h} P_{s,\,t+d}\qquad [\text{mm}]$$

Como 1 mm sobre 1 ha equivale a 10 m³, el mismo valor por hectárea es $10\,y_h$ m³/ha.

## Horizonte confiable *n*

$$SS_h = 1-\frac{RMSE_{\text{modelo},h}}{RMSE_{\text{climatología},h}}$$

Se usan dos criterios y se adopta el más conservador:

- **Prueba:** $n_{\text{prueba}} = \max\{h:\ \text{IC}_{95}(SS_h)_{\text{inf}} > 0\}$, con intervalos por bootstrap de bloques en 2021–2025.
- **Validación:** $n_{\text{val}}$ = mayor horizonte en el que el SVR supera a la climatología en al menos 4 de los 5 bloques de la validación cruzada temporal (1981–2020).
- $n = \min(n_{\text{prueba}},\ n_{\text{val}})$.

Para las semanas posteriores a *n* se usan los cuantiles climatológicos; para la planificación de la temporada, el nivel 2.

## Diseño de validación

**En este entregable (un sitio):** partición cronológica. Entrenamiento 1981–2020, con purga de las filas cuya semana objetivo cae en 2021, y prueba 2021–2025, usada una sola vez al final. La validación cruzada es de ventana creciente con una separación de 7h + 3 días.

**Diseño regional (segundo entregable):** entrenamiento con los sitios regionales hasta 2020 y evaluación en la finca como sitio no visto en 2021–2025. Los datos de 2021–2025 de los sitios regionales no se usan, porque comparten los mismos eventos climáticos que la finca y producirían fuga. Al construir el dataset se excluye además la celda regional más cercana a la finca.

## Justificación del dataset

NASA POWER ofrece series diarias continuas desde 1981 de todas las variables necesarias para calcular la evapotranspiración FAO-56 y para caracterizar el estado del suelo, con acceso abierto y reproducible por API. Una grilla regional multiplica los ejemplos de entrenamiento y permite evaluar la generalización espacial; el índice ONI aporta la señal de ENSO, la principal fuente de predictibilidad estacional en Colombia.
