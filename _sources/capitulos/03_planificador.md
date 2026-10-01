# Planificador de siembra y agua

El planificador responde a la pregunta del agricultor: *si quiero sembrar estos cultivos en esta fecha, ¿cuánta agua necesito tener para asegurar el ciclo?* Está implementado como aplicación web en `apps/planificador_siembra.py` (Dash). Se ejecuta con `python apps/planificador_siembra.py` y se abre en `http://127.0.0.1:8050`.

## Entradas

| Entrada | Símbolo | Descripción |
|---|---|---|
| Cultivos y hectáreas | $x_i$ | Uno o varios cultivos en la misma temporada |
| Coeficientes de agua | $K_c$ y duración por etapa, o $c_i^{\text{fijo}}$ | Referencia FAO-56 editable, o consumo directo en m³/ha/día |
| Fecha de siembra | $d_0$ | Inicio del ciclo |
| Tierra disponible | $H$ | ha |
| Capacidad del reservorio | $J$ | m³ en condiciones normales |
| Nivel al sembrar | $\lambda$ | Fracción de $J$ |
| Captación | $A_c$, $C$ | Área que escurre al reservorio y coeficiente de escorrentía |
| Agua externa | $Q_{\text{ext}}$ | m³ por semana (río, pozo) |
| Eficiencia de riego | $\eta$ | Goteo 0,90; aspersión 0,75; gravedad 0,60 |
| Pronóstico estacional | $\pi_{\text{seca}}, \pi_{\text{normal}}, \pi_{\text{lluviosa}}$ | Salida del nivel 2 (1/3 cada uno sin pronóstico) |
| Seguridad | $p$ | 80 %, 90 % o 95 % de los años |

## Formulación

Consumo del cultivo por hectárea y día:

$$c_{i,d} = 10\cdot K_{c,i}(d)\cdot ET_{0,d}\quad[\text{m}^3/\text{ha/día}]\qquad\text{o bien}\qquad c_{i,d}=c_i^{\text{fijo}}$$

Necesidad de riego semanal, descontando la lluvia efectiva sobre el cultivo ($P^{ef}_d=\alpha P_d$ si $P_d\ge P_{\min}$):

$$NR_{i,w}=\frac{\max\left(0,\ \sum_{d\in w}c_{i,d}-10\,P^{ef}_w\right)}{\eta}$$

Reservorio de capacidad constante $J$, que se vacía con el riego y se llena con la lluvia captada y el agua externa:

$$S_w=\min\Big(J,\ S_{w-1}+10\,P_w A_c C+Q_{\text{ext}}-\sum_i NR_{i,w}\,x_i\Big),\qquad S_0=\lambda J$$

Reserva mínima para no quedarse sin agua en el año $y$, por el algoritmo del pico secuente (Thomas y Burden, 1963), con uso $U_w$ y entrada $E_w$:

$$K_w=\max(0,\ K_{w-1}+U_w-E_w),\qquad R^{(y)}=\max_w K_w$$

Cada año desde 1981 es un escenario (Day, 1985). Los pesos vienen del pronóstico por terciles (Werner et al., 2004):

$$\omega_y=\frac{\pi_{k(y)}}{N_{k(y)}},\qquad P(\text{éxito})=\sum_y \omega_y\,\mathbb{1}\left[\min_w S^{(y)}_w\ge 0\right]$$

La reserva reportada es el cuantil ponderado $R_p$ de $R^{(y)}$, y el área máxima manteniendo la proporción $\rho_i$ entre cultivos es:

$$A^*=\max\{A\le H:\ P(\text{éxito}\mid x_i=\rho_i A)\ge p\}$$

## Salidas

- Frase de decisión: reserva necesaria para el nivel de seguridad elegido, probabilidad de éxito con el reservorio actual y área máxima.
- Tabla por cultivo: ciclo, consumo total (m³/ha), consumo diario (m³/ha/día), riego en año normal y en año seco.
- Figuras: demanda semanal por cultivo, nivel del reservorio con banda entre años, reserva que habría hecho falta cada año y reserva por hectárea según la fecha de siembra.

## Coeficientes de referencia incluidos

| Cultivo | $K_c$ inicial | $K_c$ medio | $K_c$ final | Etapas (días) | Ciclo |
|---|---|---|---|---|---|
| Maíz | 0,30 | 1,20 | 0,35 | 20 / 35 / 40 / 30 | 125 días |
| Millo (sorgo) | 0,30 | 1,05 | 0,55 | 20 / 35 / 40 / 30 | 125 días |
| Frijol | 0,40 | 1,15 | 0,35 | 20 / 30 / 40 / 20 | 110 días |
| Yuca | 0,30 | 0,80 | 0,30 | 20 / 40 / 90 / 60 | 210 días |
| Patilla | 0,40 | 1,00 | 0,75 | 20 / 30 / 30 / 30 | 110 días |

Valores de referencia de FAO-56 (tablas 11 y 12). Deben ajustarse a la variedad sembrada y validarse con asistencia técnica local.

El notebook 04 presenta la versión de optimización lineal del mismo balance, útil cuando se quiere repartir la tierra entre varios cultivos según su beneficio por hectárea.
