# Marco de referencia

## Clima del Caribe colombiano

La variabilidad hidrológica de Colombia muestra efectos estacionales diferenciados asociados a ENSO: las anomalías son más fuertes en diciembre–febrero y más débiles en marzo–mayo, y el efecto es mayor sobre caudales que sobre la precipitación por su acción conjunta sobre la humedad del suelo y la evapotranspiración (Poveda et al., 2001). En general, El Niño produce temporadas secas más secas y prolongadas y La Niña intensifica las temporadas húmedas. Esto justifica usar la humedad del suelo como variable con memoria; incorporar el ONI como predictora queda como trabajo futuro, porque el dataset de este proyecto contiene solo las variables de NASA POWER de la finca.

El ciclo anual bimodal con una disminución a mitad de año (veranillo, *midsummer drought*) está documentado en el Caribe y Centroamérica (Magaña et al., 1999) y se asocia al chorro de bajo nivel del Caribe, cuya variabilidad interanual responde a los gradientes de temperatura del mar entre el Pacífico y el Atlántico tropical (Wang, 2007).

## Demanda de agua de los cultivos

La evapotranspiración del cultivo se estima con el coeficiente de cultivo de FAO-56, $ET_c = K_c \cdot ET_0$, con $ET_0$ por Penman-Monteith y $K_c$ variable por etapa fenológica (Allen et al., 1998). La mayor sensibilidad del rendimiento al déficit hídrico en floración se documenta en FAO-66 (Steduto et al., 2012). La respuesta del rendimiento al agua por etapas, con coeficientes $K_y$, viene de FAO-33 (Doorenbos y Kassam, 1979) y se usa en el capítulo 7.

## Pronóstico por escenarios históricos ponderados

El planificador se basa en la predicción por conjuntos con trazas históricas (*Extended Streamflow Prediction*, ESP): cada año del registro se asume como una realización posible del futuro y se simula un escenario con él (Day, 1985). Incorporar información climática ponderando las trazas con índices como Niño-3.4 es una práctica establecida (Werner et al., 2004), y restringir el conjunto a años con fase de ENSO similar produce conjuntos más concentrados (Hamlet y Lettenmaier, 1999). En este proyecto, mientras no haya un pronóstico estacional, todos los años reciben el mismo peso (1/3 por tercil).

## Machine Learning para el pronóstico de lluvia

Los bosques aleatorios (Breiman, 2001) y el *gradient boosting* (Friedman, 2001; Ke et al., 2017) capturan relaciones no lineales y se han usado para pronóstico estacional de precipitación con resultados interpretables (Gibson et al., 2021). La lluvia tiene muchos ceros, lo que motiva los modelos en dos partes de ocurrencia y cantidad (Stern y Coe, 1984). La incertidumbre se cuantifica con regresión cuantílica (Koenker y Bassett, 1978) o predicción conformal (Angelopoulos y Bates, 2023), y los modelos se interpretan con SHAP (Lundberg y Lee, 2017).

## Combinación de modelos (apilamiento)

El apilamiento (*stacking*) combina los pronósticos de varios modelos con un segundo modelo, el "juez", que se entrena con pronósticos fuera de muestra para no premiar al modelo que memoriza (Wolpert, 1992). Breiman (1996) mostró que, en regresión, restringir los pesos del juez a valores no negativos evita combinaciones inestables y mejora el error frente a elegir el mejor modelo individual. El modelo híbrido del capítulo 5 aplica esta idea al SVR lineal y al random forest.

## Comparación de pronósticos

Para decidir si un pronóstico es mejor que otro no basta con comparar dos números de error: la diferencia puede deberse al azar del periodo evaluado. La prueba de Diebold y Mariano (1995) compara, semana a semana, las pérdidas de dos pronósticos y tiene en cuenta la autocorrelación de esas diferencias; la corrección de Harvey, Leybourne y Newbold (1997) la ajusta para muestras pequeñas. Los intervalos de confianza de métricas como el AUC o la tasa de detección se obtienen con bootstrap por bloques móviles, que remuestrea bloques de semanas consecutivas para conservar la dependencia temporal (Künsch, 1989).

Entrenar con datos agrupados de varias unidades espaciales puede superar a los modelos locales: en Colombia, un bosque aleatorio entrenado con datos nacionales agrupados pronosticó mejor por departamento que los modelos locales (Zhao et al., 2020). Este resultado fundamenta la extensión regional que se propone como trabajo futuro.

## Validación con dependencia espacial y temporal

Con datos estructurados en el tiempo o el espacio, la validación aleatoria subestima el error; se recomienda validar por bloques que respeten esa estructura (Roberts et al., 2017). Para pronósticos probabilísticos por categorías se usa el *Ranked Probability Score* (Epstein, 1969).

## Calidad de los datos grillados

NASA POWER se ha usado como fuente de clima para modelación de cultivos (Bai et al., 2010), pero su desempeño en precipitación varía por región y es menor en eventos intensos. Se propone contrastarlo con estaciones del IDEAM, con CHIRPS (Funk et al., 2015) o con ERA5 (Hersbach et al., 2020), este último con registro desde 1940.
