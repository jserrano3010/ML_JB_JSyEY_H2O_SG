# Pronóstico de agua y planificación de siembras bajo sequía en Sabanagrande, Atlántico

**Edwin Yunis y Jairo Serrano** — Maestría en Ingeniería Industrial
Curso de Machine Learning — Profesor: Dr. Lihki Rubio
Proyecto final

## Resumen

En el Atlántico la lluvia se concentra en dos temporadas y el resto del año la atmósfera demanda más agua de la que cae. Antes de sembrar, el agricultor necesita saber cuántas hectáreas puede sostener con el agua que tiene y cuánta debe reservar para todo el ciclo del cultivo.

El proyecto tiene dos niveles:

1. **Pronóstico semanal de la lluvia** (regresión): un modelo lineal (SVR), uno no lineal (random forest) y su combinación en un **modelo híbrido**, comparados con cuatro líneas base: Dummy, climatología, ingenuo estacional y persistencia semanal.
2. **Decisión de siembra y agua**: con el pronóstico, el balance hídrico FAO-56 y un modelo del reservorio, se calcula el área máxima sembrable y el agua que debe reservarse, en un escenario esperado y en uno con 95 % de seguridad.

## Datos y validación

Se usa la serie diaria de NASA POWER de la celda de la finca en Sabanagrande (1981–2025, 16.436 días). La partición es cronológica: entrenamiento 1981–2020 y prueba 2021–2025, reservada antes de cualquier análisis. Todas las predictoras usan datos con al menos 3 días de antigüedad, la validación cruzada es de ventana creciente con separación entre entrenamiento y validación, y los intervalos de confianza se calculan con bootstrap por bloques.

## Resultados principales

- La lluvia semanal es muy asimétrica y estacional: dos temporadas de lluvia, una temporada seca de diciembre a marzo y semanas sin lluvia frecuentes (capítulo 2).
- El SVR lineal mejora en 7 % el RMSE de la climatología a una semana (IC 95 % de 3,7 % a 10,7 %), y el pronóstico es confiable hasta **4 semanas** (capítulo 3). La persistencia semanal es peor que la climatología en todos los horizontes.
- El random forest empata con el SVR en error y detecta más semanas de aguacero (capítulo 4).
- El **modelo híbrido SVR + random forest**, elegido con la validación cruzada temporal, tiene el menor error cuadrático medio en validación en los cuatro horizontes (a una semana, empatado con el SVR). En la prueba 2021–2025 supera con significancia estadística al promedio y a la persistencia en los cuatro horizontes, y a la climatología a una semana (prueba de Diebold-Mariano). Además, detecta entre el 41 % y el 51 % de las semanas de aguacero, frente al 20–27 % del SVR, con diferencia significativa. El gradient boosting (equivalente a XGBoost) quedó último en validación y se descartó (capítulo 5).
- Ningún modelo anticipa la magnitud de los aguaceros más fuertes. Para la pregunta "¿será una semana seca?", el clasificador logístico acierta el 79–80 % de las semanas.
- Con los parámetros de ejemplo de la finca y siembra de maíz a comienzos de enero, el área máxima es de **3,37 ha en el escenario esperado y 2,84 ha con 95 % de seguridad**; el resultado depende sobre todo de la capacidad del reservorio (capítulo 6).
- **Rúbrica final (capítulo 7).** Siete criterios con pesos fijados de antemano (error, detección de aguaceros, AUC y significancia frente a la climatología) califican a los siete modelos en los horizontes de 1 a 4 semanas. El híbrido obtiene **94,5 de 100 (Excelente)**, el primer puesto. Le siguen el gradient boosting (92,4), el random forest (88,7) y el SVR (85,6); los tres métodos tradicionales quedan por debajo de 60 (Insuficiente).
- **Decisión de siembra (capítulo 7).** Con un balance hídrico diario y la respuesta del rendimiento al agua de FAO-33, un maíz híbrido de 4 meses (8 t/ha con riego) sembrado el 1 de enero en secano rinde **0 t/ha**. Para lograr las 8 t/ha haría falta regar unos 4.886 m³ por hectárea. La mejor fecha en secano es el **15 de agosto (6,90 t/ha, 86 % del potencial)**, dentro de una ventana del 15 de julio al 1 de septiembre. Los aguaceros aportan el 52 % de la lluvia de esa temporada, y el híbrido es el modelo que más los anticipa (67 % de los de julio a noviembre en la prueba, frente a 0 % de la climatología).
- El periodo de prueba es mucho más lluvioso que el de entrenamiento (29,7 frente a 18,5 mm por semana); la serie reciente de NASA POWER debe contrastarse con estaciones del IDEAM.

## Estructura del libro

```{tableofcontents}
```

## Reproducibilidad

- Dependencias en `requirements.txt`; semilla global `SEED = 42` en `src/config.py`, donde están todos los parámetros.
- La serie de Sabanagrande se prepara en `00b_dataset_un_sitio.ipynb` a partir del Excel de NASA POWER (`data/raw/Entregable1_Sabanagrande.xlsx`).
- Los notebooks se ejecutan en orden: `00b`, `01`, `02`, `03`, `03b`, `03c`, `04` y `05`. El planificador interactivo está en `apps/planificador_siembra.py`.
