# Pronóstico de agua y planificación de siembras bajo sequía — Sabanagrande, Atlántico

Edwin Yunis y Jairo Serrano — Maestría en Ingeniería Industrial. Proyecto del curso de Machine Learning; informe en Jupyter Book.

## Estructura

```
├── _config.yml, _toc.yml, intro.md   # configuración del Jupyter Book
├── capitulos/                        # planteamiento, marco de referencia, planificador, limitaciones, referencias
├── apps/planificador_siembra.py      # herramienta de decisión (Dash)
├── notebooks/
│   ├── 00_construccion_dataset.ipynb # grilla regional + finca, NASA POWER + ONI + ET0 → data/processed/dataset_sabanagrande.csv.gz
│   ├── 01_base_de_datos.ipynb        # reserva de test, tamaño, diccionario, calidad
│   ├── 02_eda.ipynb                  # secciones 2.1 a 2.9 de la guía (solo train)
│   ├── 03_modelo_base.ipynb          # líneas base, SVR lineal, IC bootstrap, horizonte n
│   └── 04_optimizacion.ipynb         # área máxima y reserva de agua
├── src/
│   ├── config.py                     # TODOS los parámetros (los de la finca en None)
│   ├── hidrologia.py                 # FAO-56, necesidad de riego, tanque, optimización
│   ├── modelado.py                   # objetivo, predictoras, partición, métricas
│   └── espacial.py                   # haversine, pesos, I de Moran
├── data/raw, data/processed          # se llenan al ejecutar el notebook 00
└── requirements.txt
```

## Cómo reproducir

```bash
python -m venv .venv && source .venv/bin/activate      # en Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab                                              # ejecutar los notebooks 00 → 04 en orden y guardar
jupyter-book build .                                     # genera _build/html/index.html
```

El libro está configurado con `execute_notebooks: "off"`: muestra las salidas guardadas en cada notebook.

## Parámetros pendientes

En `src/config.py` quedan en `None` los parámetros de la finca (`FINCA`), de lluvia efectiva (`LLUVIA_EFECTIVA`) y de cultivos (`CULTIVOS`). Los notebooks 00–03 no los necesitan; el notebook 04 lista los que faltan y solo corre cuando están completos.
