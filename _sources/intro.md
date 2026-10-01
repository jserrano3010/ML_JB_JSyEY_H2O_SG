# Pronóstico de agua y planificación de siembras bajo sequía en Sabanagrande, Atlántico

**Edwin Yunis y Jairo Serrano** — Maestría en Ingeniería Industrial
Curso de Machine Learning — Profesor: Dr. Lihki Rubio
Primer entregable: base de datos, análisis exploratorio y modelo base

## Resumen

En el Atlántico la lluvia se concentra en dos temporadas y el resto del año la atmósfera demanda más agua de la que cae. Antes de sembrar, el agricultor necesita saber cuántas hectáreas puede sostener con el agua que tiene y cuánta debe reservar para todo el ciclo del cultivo, de cuatro a seis meses.

El proyecto construye un sistema de tres niveles:

1. **Pronóstico semanal** de la lluvia (regresión), para programar el riego.
2. **Pronóstico estacional**: probabilidad de que la temporada sea seca, normal o lluviosa (clasificación), para decidir la siembra.
3. **Planificador de siembra y agua**: con los cultivos, hectáreas y coeficientes de agua que ingresa el agricultor, simula el ciclo con el clima de cada año desde 1981, ponderado por el pronóstico, y calcula la reserva necesaria, el área máxima sembrable y la mejor fecha de siembra.

Este primer entregable desarrolla la base de datos, el análisis exploratorio y el modelo base del nivel 1, e incluye la formulación y la implementación del nivel 3. El nivel 2 se desarrolla en el segundo entregable.

## Datos y validación

Este entregable usa la serie real de NASA POWER de la celda de Sabanagrande (1981–2025, 16.436 días), con partición cronológica: entrenamiento 1981–2020 y prueba 2021–2025, reservada antes de cualquier análisis. El diseño regional (unos 40 sitios del Caribe, evaluación en la finca como sitio no visto) está implementado en el notebook 00 y se ejecutará en el segundo entregable.

## Resultados principales

- La ET0 (1.703 mm/año) supera a la lluvia (958 mm/año) en 10 de los 12 meses; solo septiembre y octubre tienen excedente.
- El SVR lineal supera a la línea base trivial (Dummy) y a la climatología: a una semana, el RMSE baja de 46,5 mm (Dummy) y 43,4 mm (climatología) a 40,3 mm (mejora de 7 % sobre la climatología, IC 95 % de 3,7 % a 10,7 %).
- Con el criterio conservador, el pronóstico es confiable hasta **4 semanas**.
- El periodo de prueba (2021–2025) es 61 % más lluvioso que el de entrenamiento; debe verificarse con estaciones del IDEAM.

## Estructura del libro

```{tableofcontents}
```

## Reproducibilidad

- Dependencias en `requirements.txt`; semilla global `SEED = 42` en `src/config.py`, donde están todos los parámetros.
- Para este primer entregable se usa la serie real de Sabanagrande preparada por `00b_dataset_un_sitio.ipynb`; el notebook `00_construccion_dataset.ipynb` queda reservado para el segundo entregable y no forma parte del índice de este libro.
- Para reproducir este entregable, los notebooks relevantes son `00b`, `01`, `02` y `03`; el notebook `04` contiene la herramienta de decisión como componente complementario. El planificador interactivo está en `apps/planificador_siembra.py`.
