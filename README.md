# Riesgo relativo para préstamos bancarios

## Enlaces

Bitácora [enlace](https://docs.google.com/document/d/10HBY7hmzNywzdb3slroqzfaB9khmlWiie5k556U57IM/edit)<br>
Dashboard Looker Studio [enlace](https://www.google.com/url?q=https://lookerstudio.google.com/reporting/7893ce7d-40d0-4d16-ba5c-b9275a236f84/page/p_778ch8jqkd/edit&sa=D&source=docs&ust=1726039376046579&usg=AOvVaw2UbAPXp76nCRbfaiSDrke_)<br>
Presentación Google Slides [enlace](https://docs.google.com/presentation/d/1Crpgxey_52Pm-UbDiK37PEK-PRMR0aVUuv8OIcBj5_k/edit#slide=id.p)<br>
Proyecto BigQuery [enlace](https://console.cloud.google.com/bigquery?hl=es&project=laboratoria-426816&ws=!1m0)<br>
Tablas riesgo relativo y Correlación Pearson Google Sheets [enlace](https://docs.google.com/spreadsheets/d/1llGpm6qXcjecN5HqJBCZ8O1bd2yvLPsxzYjOPIBRCbU/edit?gid=0#gid=0)<br>

## Resumen

La disminución de tasas de interés en el mercado ha desencadenado un aumento notable en la demanda de solicitudes de crédito. A su vez, la creciente tasa de incumplimiento (default) ha aumentado la presión sobre la insitución bancaria para identificar y mitigar los riesgos asociados con el crédito. Ante ello, la metodología manual del banco Super Caja para evaluación de solicitud de préstamo ha resultado en un proceso ineficiente y demorado, afectando negativamente la eficacia y la rapidez de procesamiento. Por tanto, se requiere determinar a cabalidad las variables de mayor riesgo de incumplimiento de pago y la automatización del proceso de análisis, con el objetivo de optimizar la eficiencia, la precisión y la rapidez en la evaluación de las solicitudes de crédito. 

## Herramientas utilizadas

BigQuery, Google Sheets, Looker Studio y Google Slides.

## Lenguajes utilizados

SQL

## Metodología

- Preparación base de datos: Limpieza y creación de nuevas variables.
- Técnicas de análisis utilizadas: Segmentación de clientes por características y comportamientos. Medidas estadísticas como Correlación de Pearson y desviación estándar. Cálculo de riesgo relativo de variables para descubrir patrones y tendencias en relación a malos pagadores. Gráficas como scatter plots, histogramas y box plots. 
- Validación de hipótesis: Entender los resultados del cálculo de riesgo relativo para validar o refutar las 3 hipótesis proporcionadas por el banco.
- Descubrimiento de información oculta en los datos: Explorar y analizar los datos para encontrar información adicional que pueda influir en la toma de decisiones y con ello desarrollar estrategias.
- Visualización de datos mediante gráficos y tablas en dashboard.

## Hipótesis

1. Los clientes más jóvenes tienen un mayor riesgo de impago.  
2. Las personas con más cantidad de préstamos activos tienen mayor riesgo de ser malos pagadores.
3. Las personas que han retrasado sus pagos por más de 90 días tienen mayor riesgo de ser malos pagadores.

## Composición dataset

Se trabajó en base a 4 tablas que contienen datos de caracterización de los clientes y sus respectivos comportamientos de pago. Las variables que componen el conjunto de datos original son las siguientes:
- *user_info*
  - *user_id*: número de identificación del cliente.
  - *age*: edad del cliente.
  - *sex*: género del cliente.
  - *last_month_salary*: último salario mensual que el cliente reportó al banco.
  - *number_dependents*: número de dependientes a cargo del cliente.
- *loans_outstanding*
  - *user_id*
  - *loan_id*: número de identificación de cada préstamo.
  - *loan_type*: tipo de préstamo (*real estate* = inmobiliario, *others* = otro).
  - *using_lines_not_secured_personal_assets*: cuánto está utilizando el cliente en relación con su límite de crédito en líneas que no están garantizadas con bienes personales, como inmuebles y automóviles.
  - *user_id*
  - *more_90_days_overdue*: número de veces que el cliente tuvo su pago vencido más de 90 días.
  - *number_times_delayed_payment_loan_60_89_days*: número de veces que el cliente retrasó el pago de un préstamo entre 60 y 89 días.
  - *number_times_delayed_payment_loan_30_59_days*: número de veces que el cliente se retrasó en el pago de un préstamo entre 30 y 59 días.
  - *debt_ratio*: relación entre las deudas y el patrimonio del prestatario. Ratio de deuda = Deudas / Patrimonio.
- *defaul_flag*
  - *user_id*
  - *default_flag*: clasificación de los clientes morosos (1 para clientes que pagan mal, 0 para clientes que pagan bien).

## Procesar y preparar base de datos

## Identificar y manejar valores nulos 

La única tabla con nulos es *user_info*.

```sql
-- Identificar nulos en tabla user_info
SELECT *
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
  WHERE user_id IS NULL
  OR age IS NULL
  OR sex IS NULL
  OR last_month_salary IS NULL
  OR number_dependents IS NULL
```
Resultado: 7199 filas con valores nulos en *last_month_salary* y *number_dependents*. Esto constituye aproximadamente el 20% de nuestra base de datos de usuarios.

## Identificar y manejar valores duplicados

La única tabla con valores duplicados es *loans_outstanding*.

```sql
-- Identificar duplicados en user_id de tabla loans_outstanding
SELECT user_id, COUNT(*) AS veces_repetido
FROM `laboratoria-426816.proyecto3_riesgorelativo.loans_outstanding`
GROUP BY user_id
HAVING COUNT(*) > 1
```
Resultado: 34.510 ID de cliente duplicados. Esto se debe a que un mismo cliente puede tener varios préstamos. Los ID se repiten desde 2 a 57 veces. Más adelante se creará la nueva variable *total_loans* con el total de préstamos por ID, para agruparlos y gestionar duplicados.


## Identificar y manejar datos fuera del alcance del análisis

## Correlación de Pearson

### Tabla *loans_detail*

Para la tabla *loans_detail*, las variables con alta correlación entre sí son las de los 3 tipos de retraso y también los retrasos en relación a *using_lines_not_secured_personal_assets*.

```sql
SELECT CORR (more_90_days_overdue, number_times_delayed_payment_loan_30_59_days) AS correlation_value
FROM `laboratoria-426816.proyecto3_riesgorelativo.loans_detail`
```
Resultado: 0.98291680661459857 ≈ 0.98 (correlación positiva alta, en la que los valores de ambas variables tienden a incrementarse juntos)

```sql
SELECT CORR (more_90_days_overdue, number_times_delayed_payment_loan_60_89_days) AS correlation_value
FROM `laboratoria-426816.proyecto3_riesgorelativo.loans_detail`
```
Resultado: 0.99217552634075257 ≈ 0.99 (correlación positiva alta, en la que los valores de ambas variables tienden a incrementarse juntos)

```sql
SELECT CORR (more_90_days_overdue, using_lines_not_secured_personal_assets) AS correlation_value
FROM `laboratoria-426816.proyecto3_riesgorelativo.loans_detail`
```
Resultado: 0.99217552634075257 ≈ 0.99 (correlación positiva alta, en la que los valores de ambas variables tienden a incrementarse juntos)

### Tabla *user_info*

Se detectaron dos correlaciones positivas entre variables, las cuales no son significativas.

```sql
SELECT CORR (age, last_month_salary) AS correlation_value
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
```
Resultado: 0.035978007340201588  ≈ 0.036 (correlación positiva baja, valor muy cercano a 0, lo que sugiere que hay una relación lineal muy débil entre las dos variables. La relación es tan débil que en la mayoría de los casos puede considerarse como casi nula)

```sql
SELECT CORR (last_month_salary, number_dependents) AS correlation_value
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
````
Resultado: 0.077995880946669469  ≈ 0.078 (correlación positiva baja, en la que los valores de ambas variables tienden a incrementarse juntos)

## Desviación estándar

### Tabla *loans_detail*

De las variables overdue con alta correlación, number_times_delayed_payment_loan_30_59_days* es la que poseee una desviación mayor (4.14).

```sql
SELECT STDDEV(more_90_days_overdue) AS stddev_more_90_days_overdue
FROM `laboratoria-426816.proyecto3_riesgorelativo.loans_detail`
WHERE more_90_days_overdue IS NOT NULL
```
Resultado: 4.12136466842672 ≈ 4.12

```sql
SELECT STDDEV(number_times_delayed_payment_loan_30_59_days) AS stddev_number_times_delayed_payment_loan_30_59_days
FROM `laboratoria-426816.proyecto3_riesgorelativo.loans_detail`
WHERE number_times_delayed_payment_loan_30_59_days IS NOT NULL
```
Resultado: 4.144020438225871 ≈ 4.14

```sql
SELECT STDDEV(number_times_delayed_payment_loan_60_89_days) AS stddev_number_times_delayed_payment_loan_60_89_days
FROM `laboratoria-426816.proyecto3_riesgorelativo.loans_detail`
WHERE stddev_number_times_delayed_payment_loan_60_89_days IS NOT NULL
```
Resultado: 4.1055147551019706 ≈ 4.11

```sql
SELECT STDDEV(using_lines_not_secured_personal_assets) AS stddev_using_lines_not_secured_personal_assets
FROM `laboratoria-426816.proyecto3_riesgorelativo.loans_detail`
WHERE stddev_using_lines_not_secured_personal_assets IS NOT NULL
```
Resultado: 223.407144459142 ≈ 223.41

```sql
SELECT STDDEV(debt_ratio) AS stddev_debt_ratio
FROM `laboratoria-426816.proyecto3_riesgorelativo.loans_detail`
WHERE debt_ratio IS NOT NULL
```
Resultado: 2011.6353409106493 ≈ 2011.64

### Tabla *user_info*

```sql
SELECT STDDEV(age) AS age_stddev
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
WHERE age IS NOT NULL
```
Resultado: 14.791331229276 ≈ 14.79

```sql
SELECT STDDEV(last_month_salary) AS last_month_salary_stddev
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
WHERE last_month_salary IS NOT NULL
```
Resultado: 12961.77847729006 ≈ 12961.78

```sql
SELECT STDDEV(number_dependents) AS number_dependents_stddev
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
WHERE number_dependents IS NOT NULL
```
Resultado: 1.1187389026209864 ≈ 1.12

## Identificar y manejar datos inconsistentes en variables categóricas

La tabla *loans_outstanding* presenta las siguientes inconsistencias en la columna *loan_type*:

- *REAL ESTATE, real estate, Real Estate*		  
- *OTHER, other, Other, others*

```sql
-- Crear vista ‘loans_outstanding_clean’
CREATE OR REPLACE VIEW `laboratoria-426816.proyecto3_riesgorelativo.loans_outstanding_clean` AS

-- Estandarizar variable categórica 'other'
WITH loans_normalized AS (
 SELECT
   user_id,
   CASE
    WHEN LOWER(loan_type) = 'others' THEN 'other'
    ELSE LOWER(loan_type)
   END AS loan_type_normalized
 FROM `laboratoria-426816.proyecto3_riesgorelativo.loans_outstanding`
)

-- Estandarizar variable categórica 'real_estate' y crear nuevas variables ‘total_loans’, ‘real_estate_loans’ y ‘other_loans’
SELECT
 user_id,
 COUNT(*) AS total_loans,
 COUNT(CASE WHEN loan_type_normalized = 'real estate' THEN 1 END) AS real_estate_loans,
 COUNT(CASE WHEN loan_type_normalized = 'other' THEN 1 END) AS other_loans
FROM loans_normalized
GROUP BY user_id
```

## Identificar y manejar datos discrepantes en variables numéricas

### Identificar los datos *outliers* en *last_month_salary*

Obtener medidas de tendencia (mínimo, máximo y promedio) de la columna *last_month_salary*, sin imputar los datos y excluyendo los nulos, para tener un parámetro de comparación al recalcular las medidas de tendencia central con los datos ya imputados.

```sql
-- Calcular medidas de tendencia central de last_month_salary sin imputar y excluyendo nulos
WITH mediana AS (
 SELECT
   PERCENTILE_CONT(last_month_salary, 0.5) OVER() AS last_month_salary_mediana
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
WHERE last_month_salary IS NOT NULL
LIMIT 1
)
SELECT
 MIN(last_month_salary) AS last_month_salary_min,
 MAX(last_month_salary) AS last_month_salary_max,
 AVG(last_month_salary) AS last_month_salary_avg,
 (SELECT last_month_salary_mediana FROM mediana) AS last_month_salary_mediana
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
WHERE last_month_salary IS NOT NULL
```
last_month_salary_min:	0.0<br>
last_month_salary_max:	1560100.0<br>
last_month_salary_avg:	6675.0520468039576<br>
last_month_salary_mediana:	5400.0<br>
P1 = 0<br>
P2 = 260<br>
P99 = 25113.999999999971

```sql
-- Identificar outliers en last_month_salary
WITH percentiles AS (
SELECT
  PERCENTILE_CONT(last_month_salary, 0.02) OVER() AS P1,
  PERCENTILE_CONT(last_month_salary, 0.99) OVER() AS P99
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
WHERE last_month_salary IS NOT NULL
),
outliers AS (
SELECT
  *,
  CASE
    WHEN last_month_salary < P2 THEN 'outlier'
    WHEN last_month_salary > P99 THEN 'outlier'
    ELSE 'within'
  END AS outlier_flag
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`, (SELECT DISTINCT P2, P99 FROM percentiles)
WHERE last_month_salary IS NOT NULL
)
SELECT *
FROM outliers
WHERE outlier_flag = 'outlier'
```
Resultado: Se detectaron 866 outliers en *last_month_salary*.

## Tratar *outliers*: *capping* mediante winzorización en *last_month_salary*

Dado que los datos de *last_month_salary* están en una distribución que no es normal, para imputar los datos se deben usar la mediana y no el promedio.

### Imputar nulos en *last_month_salary*

Imputamos los nulos de *last_month_salary* reemplazándolos por la mediana. Se utiliza mediana en vez de promedio, ya que los datos de *last_month_salary* están en una distribución que no es normal. Para ésto debemos calcular la mediana excluyendo los nulos. La calculamos una vez imputados los 866 *outliers*, ya que de haberlo hecho antes se hubiesen visto afectado el P50 (mediana).

Primero calculamos la mediana para *last_month_salary* inicial y *last_month_salary_winzorizado*.

```sql
-- Crear percentiles P2 y P99 para last_month_salary excluyendo nulos
WITH percentiles AS (
 SELECT
   PERCENTILE_CONT(last_month_salary, 0.02) OVER() AS P2,
   PERCENTILE_CONT(last_month_salary, 0.99) OVER() AS P99
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
WHERE last_month_salary IS NOT NULL
),

-- Imputar mediante winzorización los outliers de last_month_salary
winzorizacion AS (
 SELECT
   *,
   CASE
     WHEN last_month_salary < P2 THEN P2
     WHEN last_month_salary > P99 THEN P99
     ELSE last_month_salary
   END AS last_month_salary_winzorizado
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`, (SELECT DISTINCT P2, P99 FROM percentiles)
),

-- Calcular la mediana de last_month_salary excluyendo nulos
inicial_mediana AS (
 SELECT
   PERCENTILE_CONT(last_month_salary, 0.5) OVER() AS mediana_inicial
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
WHERE last_month_salary IS NOT NULL
),

-- Calcular la mediana de last_month_salary_winzorizado excluyendo nulos
winzorizado_mediana AS (
 SELECT
   PERCENTILE_CONT(last_month_salary_winzorizado, 0.5) OVER() AS mediana_winzorizacion
FROM winzorizacion
WHERE last_month_salary_winzorizado IS NOT NULL
)

-- Visualizar percentiles P2, P99, P50_winzorizado y P50_inicial
SELECT DISTINCT
(SELECT P2 FROM percentiles LIMIT 1) AS P2,
(SELECT P99 FROM percentiles LIMIT 1) AS P99,
(SELECT mediana_winzorizacion FROM winzorizado_mediana LIMIT 1) AS P50_winzorizado,
(SELECT mediana_inicial FROM inicial_mediana LIMIT 1) AS P50_inicial
FROM percentiles
```
Resultado: La mediana original y la mediana de los datos ya winzorizados es igual (5400), lo cual sugiere estabilidad en los datos y que los valores imputados no han afectado significativamente la parte central de la distribución de éstos. También puede indicar que los datos imputados son consistentes con la distribución general de la variable, lo que significa que los valores imputados no introdujeron un sesgo significativo.

A continuación imputamos los nulos de *last_month_salary* con el valor de la mediana (5400) y creamos una nueva tabla *user_info* con los datos imputados y sin la columna *sex*, la cual dejamos fuera ya que produce un sesgo que puede incidir en el aumento de la brecha de género.

```sql
-- Crear vista last_month_salary_clean
CREATE OR REPLACE VIEW `laboratoria-426816.proyecto3_riesgorelativo.last_month_salary_clean` AS

-- Crear percentiles P2 y P99 para last_month_salary excluyendo nulos
WITH percentiles AS (
SELECT
  PERCENTILE_CONT(last_month_salary, 0.02) OVER() AS P2,
  PERCENTILE_CONT(last_month_salary, 0.99) OVER() AS P99
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
WHERE last_month_salary IS NOT NULL
),

-- Imputar mediante winzorización los outliers de last_month_salary
winzorizacion AS (
SELECT
  *,
  CASE
    WHEN last_month_salary < P2 THEN P2
    WHEN last_month_salary > P99 THEN P99
    ELSE last_month_salary
  END AS last_month_salary_winzorizado
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`, (SELECT DISTINCT P2, P99 FROM percentiles)
)

-- Crear una tabla limpia que contenga la columna last_month_salary sin outliers ni nulos, excluyendo la variable sex
SELECT
 user_id,
 age,
 number_dependents,
 CASE
   WHEN last_month_salary_winzorizado IS NULL THEN 5400
   ELSE last_month_salary_winzorizado
 END AS last_month_salary_limpio
FROM winzorizacion
```

## Tratar outliers: mediana en *number_dependents*

La variable *number_dependents* presenta una distribución que no es normal y um rango de valores de datos que no es amplio (0 a 13). Para imputar los nulos de *number_dependents* (7199 de un total de 36000, es decir, un 20%) utilizaremos la mediana, que es 0. No se considerarán datos como *outliers*, ya que si bien el máximo de 13 es un número que podría parecer alto para el número de dependientes, no es improbable y además no altera la mediana.

Actualizamos la consulta anterior:

```sql
-- Crear percentiles P2 y P99 para last_month_salary excluyendo nulos
WITH percentiles AS (
SELECT
  PERCENTILE_CONT(last_month_salary, 0.02) OVER() AS P2,
  PERCENTILE_CONT(last_month_salary, 0.99) OVER() AS P99
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
WHERE last_month_salary IS NOT NULL
),

-- Imputar mediante winzorización los outliers de last_month_salary
winzorizacion AS (
SELECT
  *,
  CASE
    WHEN last_month_salary < P2 THEN P2
    WHEN last_month_salary > P99 THEN P99
    ELSE last_month_salary
  END AS last_month_salary_winzorizado
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`, (SELECT DISTINCT P2, P99 FROM percentiles)
)

-- Crear una tabla limpia que contenga imputadas las columnas last_month_salary, number_dependents y age, y excluyendo la columna sex
SELECT
user_id,
CASE
  WHEN age > 82 THEN 82
  ELSE age
END AS age_limpio,
CASE
  WHEN number_dependents IS NULL THEN 0
  ELSE number_dependents
END AS number_dependents_limpio,
CASE
  WHEN last_month_salary_winzorizado IS NULL THEN 5400
  ELSE last_month_salary_winzorizado
END AS last_month_salary_limpio
FROM winzorizacion
```

## Tratar outliers: Z-score en *age*

La variable *age* presenta una distribución de datos normal, por lo que podemos utilizar el Z-score. El umbral de 2 para el Z-score se utiliza comúnmente en estadística para identificar outliers basados en la distribución normal de los datos. Para imputar los 886 outliers utilizaremos la edad de corte que nos arroje el Z-score.

Una exploración visual de los datos nos arrojó que son bastantes los *user_id* que figuran con edades sobre 80 años (hasta 109 años). En total son 886 *outliers* detectados con el Z-Score.

```sql
-- Calcular media y desviación estándar de la variable age de la tabla user_info.
WITH tendencia_age AS (
 SELECT
   AVG(age) AS media_age,
   STDDEV(age) AS stddev_age
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
),

-- Calcular Z-score de la variable age de la tabla user_info.
z_scores AS (
 SELECT
   age,
   (age - tendencia_age.media_age) / tendencia_age.stddev_age AS z_score
 FROM
   `laboratoria-426816.proyecto3_riesgorelativo.user_info`, tendencia_age
)

-- Traer la tabla con el outlier_flag que identifica outliers del extremo alto (valores significativamente mayores que la media)
SELECT
 age,
 z_score,
 CASE
   WHEN z_score > 2 THEN 'outlier'
   ELSE 'within'
 END AS outlier_flag_age
 FROM z_scores
```
Resultado: La edad de corte queda en 82 años, siendo *outliers* las edades de 83 a 109 años. Por lo tanto, imputaremos los *outliers* con el valor 82. 
El z-score de la menor edad (21) es -2.12404253408125 y el de la mayor (109) es 3.8253881585277063.

```sql
-- Crear vista user_info_clean
CREATE OR REPLACE VIEW `laboratoria-426816.proyecto3_riesgorelativo.user_info_clean` AS

-- Crear percentiles P2 y P99 para last_month_salary excluyendo nulos
WITH percentiles AS (
SELECT
 PERCENTILE_CONT(last_month_salary, 0.02) OVER() AS P2,
 PERCENTILE_CONT(last_month_salary, 0.99) OVER() AS P99
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`
WHERE last_month_salary IS NOT NULL
),

-- Imputar mediante winzorización los outliers de last_month_salary
winzorizacion AS (
SELECT
 *,
 CASE
   WHEN last_month_salary < P2 THEN P2
   WHEN last_month_salary > P99 THEN P99
   ELSE last_month_salary
 END AS last_month_salary_winzorizado
FROM `laboratoria-426816.proyecto3_riesgorelativo.user_info`, (SELECT DISTINCT P2, P99 FROM percentiles)
)

-- Crear una tabla limpia que contenga imputadas las columnas last_month_salary, number_dependents y age, y excluyendo la columna sex
SELECT
user_id,
CASE
  WHEN age > 82 THEN 82
  ELSE age
END AS age_limpio,
CASE
  WHEN number_dependents IS NULL THEN 0
END AS number_dependents_limpio,
CASE
  WHEN last_month_salary_winzorizado IS NULL THEN 5400
  ELSE last_month_salary_winzorizado
END AS last_month_salary_limpio
FROM winzorizacion
```

## Comprobar y cambiar tipo de dato

Revisando las variables de las 4 tablas, se constató que *user_id* debería ser STRING en vez de INTEGER. Usamos SAFE_CAST para cambiar el tipo de dato en las 4 tablas.

```sql
-- Crear vista default_clean cambiando user_id a STRING
CREATE OR REPLACE VIEW `laboratoria-426816.proyecto3_riesgorelativo.default_clean` AS

SELECT
   SAFE_CAST(user_id AS STRING) AS user_id,
   default_flag
FROM
 `laboratoria-426816.proyecto3_riesgorelativo.default`
```

```sql
-- Crear vista loans_detail_clean y cambiar user_id a STRING
CREATE OR REPLACE VIEW `laboratoria-426816.proyecto3_riesgorelativo.loans_detail_clean` AS

SELECT
 CAST(user_id AS STRING) AS user_id,
 more_90_days_overdue,
 using_lines_not_secured_personal_assets,
 number_times_delayed_payment_loan_30_59_days,
 debt_ratio,
 number_times_delayed_payment_loan_60_89_days
FROM `laboratoria-426816.proyecto3_riesgorelativo.loans_detail`
```

## Unir tablas

Al unir tablas usamos INNER JOINT para descartar datos que de incluirse serían nulos, ya que user_info tiene 36.000 filas pero loans_outstanding tiene 35.575. Con esto estamos descartando 425 filas (1.18%) que de usar LEFT JOIN serían nulos en total_loans, real_estate_loans y other_loans. Estas filas representan un porcentaje bajo del total del dataset, por lo que no sería problemático eliminarlas.

```sql
-- Crear vista track_all_data y unir tablas user_info, loans_outstanding, loans detail y default
CREATE OR REPLACE VIEW `laboratoria-426816.proyecto3_riesgorelativo.track_all_data` AS

SELECT
ui.user_id,
ui.age_limpio AS age,
ui.number_dependents_limpio AS number_dependents,
ui.last_month_salary_limpio AS last_month_salary,
lo.total_loans,
lo.real_estate_loans,
lo.other_loans,
ld.number_times_delayed_payment_loan_30_59_days,
ld.number_times_delayed_payment_loan_60_89_days,
ld.more_90_days_overdue,
ld.using_lines_not_secured_personal_assets,
ld.debt_ratio,
d.default_flag
FROM
 `laboratoria-426816.proyecto3_riesgorelativo.user_info_clean` AS ui
INNER JOIN
 `laboratoria-426816.proyecto3_riesgorelativo.loans_outstanding_clean` AS lo
ON
 ui.user_id = lo.user_id
INNER JOIN
 `laboratoria-426816.proyecto3_riesgorelativo.loans_detail_clean` AS ld
ON
 ui.user_id = ld.user_id
INNER JOIN
 `laboratoria-426816.proyecto3_riesgorelativo.default_clean` as d
ON
 ui.user_id = d.user_id
 ```

## Identificar nuevos datos atípicos

Se detectaron 866 valores atípicos con más de una posición decimal en *last_month_salary*. Resultaron ser los datos imputados con winzorización, ya que coinciden en cantidad de datos (866) y los valores son los mismos que el P2 y el P99. Por lo tanto, solo los redondeamos con ROUND.

```sql
-- Seleccionar valores atípicos que tienen más de una posición decimal en last_month_salary, luego contar cuántas veces se repite cada uno y ordenar la cantidad de repeticiones de forma descendente
SELECT
 last_month_salary,
 COUNT(*) AS count
FROM
 `laboratoria-426816.proyecto3_riesgorelativo.track_all_data`
WHERE
 ROUND(last_month_salary, 1) != last_month_salary
GROUP BY
 last_month_salary
ORDER BY
 count DESC
```

## Crear nuevas variables

Nuevas variables creadas:
- *total_loans*: total general de préstamos (incluye *real estate* y *others*)
- *real_estate_loans*: total de préstamos inmobiliarios
- *other_loans*: total de préstamos de otras categorías
- *age_group*: grupo etario. Joven: 21 a 34 años. Adulto: 35 a 54 años. Senior: más de 54 años.
- *total_delayed_payments*: cantidad de veces que cliente se ha retrasado con pago(s).
- *unsecured_credit_ratio*: ratio entre el uso de líneas de crédito no aseguradas y el salario mensual. (*unsecured_credit_ratio* = *using_lines_not_secured_personal_assets* / *last_month_salary*)
- *unsecured_credit_ratio_category*:  0 - 1: Valores bajos. 1 - 2.5: Valores moderados. > 2.5: Valores altos.
- *debt_to_income_ratio*: proporción/cantidad de deuda en relación con salario mensual. (*debt_to_income_ratio* = *debt_ratio* / *last_month_salary*)
- *debt_to_income_ratio_category*: 0 - 0.35: Bajo riesgo. 0.36 - 0.6: Riesgo moderado. > 0.7: Alto riesgo. 
- *payment_score*: puntaje de acuerdo a cantidad de retrasos y tipo de retraso, con categorías excelente, bueno y deficiente. Esta categoría penaliza con * 2 si el retraso es de 60_89_days y * 3 si es more_90_days.
- *high_debt_ratio_indicator*: identifica a prestatarios que tienen un alto nivel de deuda en relación con su patrimonio. 1 = alto riesgo (debt_ratio > 0,4), 0 = moderado y bajo riesgo.
- *income_category*: categorías de salario según *last_month_salary*. Bajo: 0 a 4679. Medio: 4680 a 6249. Alto: 6250 o más.
- *using_lines_not_secured_personal_assets_category*: 0 - 5,000: Valores Bajos. 5,001 - 15,000: Valores Moderados. 15,001 - 22,000: Valores Altos. 
- *default_flag_category*: mal pagador para 1, buen pagador para 0. Esto es para poder crear gráficos más comprensibles.


### Query actualizada de *track_all_data* con las nuevas variables

```sql
-- Unir tablas "user_info", "loans_outstanding", "loans_detail" y "default" para crear vista temporal "dataset"
WITH dataset AS (
 SELECT
   ui.user_id,
   ui.age_limpio AS age,
   ui.number_dependents_limpio AS number_dependents,
   ui.last_month_salary_limpio AS last_month_salary,
   lo.total_loans,
   lo.real_estate_loans,
   lo.other_loans,
   ld.number_times_delayed_payment_loan_30_59_days,
   ld.number_times_delayed_payment_loan_60_89_days,
   ld.more_90_days_overdue,
   ld.using_lines_not_secured_personal_assets,
   ld.debt_ratio,
   d.default_flag
FROM
 `laboratoria-426816.proyecto3_riesgorelativo.user_info_clean` AS ui
INNER JOIN
 `laboratoria-426816.proyecto3_riesgorelativo.loans_outstanding_clean` AS lo
ON
 ui.user_id = lo.user_id
INNER JOIN
 `laboratoria-426816.proyecto3_riesgorelativo.loans_detail_clean` AS ld
ON
 ui.user_id = ld.user_id
INNER JOIN
 `laboratoria-426816.proyecto3_riesgorelativo.default_clean` as d
ON
 ui.user_id = d.user_id
),

-- Crear vista temporal "variables" con nuevas variables a partir de "dataset"
variables AS (
 SELECT
 user_id,
 CASE
   WHEN age BETWEEN 21 AND 34 THEN 'joven'
   WHEN age BETWEEN 35 AND 54 THEN 'adulto'
   WHEN age >= 55 THEN 'senior'
 END AS age_group, -- Grupos etarios
 NTILE(3) OVER (ORDER BY total_loans) AS total_loans_tertiles, -- Terciles total_loans
 CASE
   WHEN last_month_salary < 4680 THEN 'bajo'
   WHEN last_month_salary >= 4680 AND last_month_salary < 6250 THEN 'medio'
   ELSE 'alto'
 END AS income_category, -- Grupos salariales
 NTILE(3) OVER (ORDER BY using_lines_not_secured_personal_assets) AS using_lines_not_secured_personal_assets_tertiles, -- Terciles using_lines_not_secured_personal_assets
 CASE
   WHEN using_lines_not_secured_personal_assets <= 5000 THEN 'bajo'
   WHEN using_lines_not_secured_personal_assets < 5000 AND using_lines_not_secured_personal_assets = 15000 THEN 'moderado'
   ELSE 'alto'
 END AS using_lines_not_secured_personal_assets_category, -- Categorías using_lines_not_secured_personal_assets
 CASE
   WHEN debt_ratio > 0.4 THEN 1
   ELSE 0
 END AS high_debt_indicator, -- Indicador de deuda alta
 CASE
   WHEN debt_ratio > 0.4 THEN 'deuda alta'
   ELSE 'deuda moderada a baja'
 END AS high_debt_category, -- Variables categóricas no booleanas para high_debt_indicator
 (number_times_delayed_payment_loan_30_59_days +
  number_times_delayed_payment_loan_60_89_days +
  more_90_days_overdue) AS total_delayed_payments, -- Total de retrasos
 (using_lines_not_secured_personal_assets / last_month_salary) AS unsecured_credit_ratio, -- unsecured_credit_ratio
 CASE
   WHEN (using_lines_not_secured_personal_assets / last_month_salary) <= 1 THEN 'bajo'
   WHEN (using_lines_not_secured_personal_assets / last_month_salary) > 1 AND (using_lines_not_secured_personal_assets / last_month_salary) <= 2.5 THEN 'moderado'
   ELSE 'alto'
 END AS unsecured_credit_ratio_category, -- Categorías unsecured_credit_ratio
 (debt_ratio / last_month_salary) AS debt_to_income_ratio, -- debt_to_income_ratio
 CASE
   WHEN (debt_ratio / last_month_salary) <= 0.35 THEN 'bajo'
   WHEN (debt_ratio / last_month_salary) > 0.35 AND (debt_ratio / last_month_salary) <= 0.6 THEN 'moderado'
   ELSE 'alto'
 END AS debt_to_income_ratio_category, -- Categorías debt_to_income_ratio
 CASE
   WHEN (number_times_delayed_payment_loan_30_59_days * 1) + (number_times_delayed_payment_loan_60_89_days * 2) + (more_90_days_overdue * 3) = 0 THEN 'excelente'
   WHEN (number_times_delayed_payment_loan_30_59_days * 1) + (number_times_delayed_payment_loan_60_89_days * 2) + (more_90_days_overdue * 3) BETWEEN 1 AND 5 THEN 'bueno'
   ELSE 'deficiente'
 END AS payment_score, -- Score de retrasos
 CASE
   WHEN default_flag = 1 THEN 'mal pagador'
   ELSE 'buen pagador'
 END AS default_flag_category -- Variables categóricas no booleanas para default_flag
FROM dataset
)

-- Unir vistas temporales "dataset" y "variables" para crear vista final "track_all_data"
SELECT
   d.user_id,
   d.age,
   v.age_group,
   d.number_dependents,
   d.last_month_salary,
   v.income_category,
   d.total_loans,
   v.total_loans_tertiles,
   d.real_estate_loans,
   d.other_loans,
   d.number_times_delayed_payment_loan_30_59_days,
   d.number_times_delayed_payment_loan_60_89_days,
   d.more_90_days_overdue,
   v.total_delayed_payments,
   d.using_lines_not_secured_personal_assets,
   v.using_lines_not_secured_personal_assets_category,
   d.debt_ratio,
   v.high_debt_indicator,
   v.high_debt_category,
   v.debt_to_income_ratio,
   v.debt_to_income_ratio_category,
   v.unsecured_credit_ratio,
   v.unsecured_credit_ratio_category,
   d.default_flag,
   v.default_flag_category
FROM
 dataset d
INNER JOIN
 variables v
ON
 d.user_id = v.user_id
```

## Hacer un análisis exploratorio de Datos (AED)

Empleamos:
- Visualización de datos a través de gráficos: distribución y relaciones entre variables.
- Resumen estadístico a través de cálculos de medidas descriptivas: calcular la media, mediana, desviación estándar, percentiles, etc, para entender la tendencia central, dispersión y forma de la distribución de los datos.
- Análisis de tendencias y patrones: observar tendencias temporales, estacionales u otros patrones interesantes en los datos.
- Segmentación de datos: dividir los datos en grupos o segmentos para analizarlos por separado y encontrar diferencias significativas.

## Correlación de Pearson

Se calculó la Correlación de Pearson para todas las variables que pudieran guardar una relación que pudiera servir para predecir riesgo o para evaluar si puede producirse multicolinealidad (por ejemplo, con las variables de retraso en pagos). Las correlaciones más significativas son:

Correlaciones positivas fuertes:
- 0.859 para *more_90_days_overdue* y *total_delayed_payments*
- 0.753 para *unsecured_credit_ratio* y *using_lines_not_secured_personal_assets*
- 0.718 para *more_90_days_overdue* y *number_times_delayed_payment_loan_60_89_days*

Correlaciones positivas moderadas:
 -0.632 para *number_times_delayed_payment_loan_60_89* y *number_times_delayed_payment_loan_30_59_days*
 -0.557 para *more_90_days_overdue* y *number_times_delayed_payment_loan_30_59_days*

No hay correlaciones negativas ni altas ni moderadas. Todas las demás correlaciones entre variables son bajas o sin correlación significativa.

De las variables de retraso en los pagos, el retraso de más de 90 días es el que mayor correlación positiva tiene con el total de retrasos y con los demás tipos de retraso, por lo que sería la variable a tener en consideración para el análisis entre las de ese grupo. *Unsecured_credit_ratio* y *using_lines_not_secured_personal_asset*s resultaron poseer una correlación positiva alta.

## Aplicar técnica de análisis

## Cálculo de riesgo relativo

A través del cálculo de riesgo relativo buscaremos validar las hipótesis proporcionadas y realizar nuevos hallazgos. Luego de validar las hipótesis según el resultado del cálculo del riesgo relativo, construiremos una tabla con el rango de datos de cada variable que tenga mayor riesgo de ser mal pagadora.

### Riesgo relativo *age_group*

Tanto jóvenes como adultos son grupos de riesgo, pero el riesgo relativo es un poco más alto en el grupo adulto. Es decir, el grupo de mayor riesgo es el joven (21 a 34 años), con aproximadamente un 93.56% más de probabilidad de ser mal pagador en comparación con el resto. El grupo adulto (35 a 54 años) tiene aproximadamente un 82.19% más de probabilidad de ser mal pagador.  

joven (21 a 34 años): 1.93559518627230<br>
adulto (35 a 54 años): 1.821859206911<br>
senior (más de 54 años): 0.3196280966791<br>

```sql
-- Calcular cantidad de malos y buenos pagadores por grupo de edad
WITH age_group_stats AS(
 SELECT
   age_group,
   COUNTIF(default_flag = 1) AS total_bad_payers,
   COUNTIF(default_flag = 0) AS total_good_payers
 FROM `laboratoria-426816.proyecto3_riesgorelativo.track_all_data`
 GROUP BY age_group
),

-- Calcular los totales combinados de cada grupo de edad
combined_groups AS (
 SELECT
   'adulto + senior' AS comparison_group,
   SUM(CASE WHEN age_group IN ('adulto', 'senior') THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN age_group IN ('adulto', 'senior') THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM age_group_stats
 UNION ALL
 SELECT
   'joven + senior' AS comparison_group,
   SUM(CASE WHEN age_group IN ('joven', 'senior') THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN age_group IN ('joven', 'senior') THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM age_group_stats
 UNION ALL
 SELECT
   'joven + adulto' AS comparison_group,
   SUM(CASE WHEN age_group IN ('joven', 'adulto') THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN age_group IN ('joven', 'adulto') THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM age_group_stats
)

-- Calcular riesgo relativo comparando grupos
SELECT
 CASE
   WHEN a.age_group = 'joven' THEN 'joven vs adulto + senior'
   WHEN a.age_group = 'adulto' THEN 'adulto vs joven + senior'
   WHEN a.age_group = 'senior' THEN 'senior vs joven + adulto'
 END AS comparison,
 a.age_group,
 a.total_bad_payers AS a,
 a.total_good_payers AS b,
 c.total_bad_payers AS c,
 c.total_good_payers AS d,
 -- Cálculo del riesgo relativo
 (a.total_bad_payers / (a.total_bad_payers + a.total_good_payers)) / (c.total_bad_payers / (c.total_bad_payers + c.total_good_payers)) AS relative_risk
FROM
 age_group_stats a
JOIN
 combined_groups c
ON
 (a.age_group = 'joven' AND c.comparison_group = 'adulto + senior') OR
 (a.age_group = 'adulto' AND c.comparison_group = 'joven + senior') OR
 (a.age_group = 'senior' AND c.comparison_group = 'joven + adulto')
```

## Riesgo relativo *income_category*

El grupo de riesgo es el de aquellos clientes con salario bajo (hasta $4679), quienes tienen aproximadamente un 44.83% más de probabilidad de ser mal pagadores en comparación con los demás grupos.

bajo: 1.448313757245493<br>
medio: 0.851616388703431<br>
alto: 0.48279507384264<br>

```sql
-- Calcular cantidad de malos y buenos pagadores por grupo de salario
WITH last_month_salary_stats AS(
 SELECT
   income_category,
   COUNTIF(default_flag = 1) AS total_bad_payers,
   COUNTIF(default_flag = 0) AS total_good_payers
 FROM `laboratoria-426816.proyecto3_riesgorelativo.track_all_data`
 GROUP BY income_category
),

-- Calcular los totales combinados de cada grupo de salario
combined_groups AS (
 SELECT
   'medio + bajo' AS comparison_group,
   SUM(CASE WHEN income_category IN ('medio', 'bajo') THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN income_category IN ('medio', 'bajo') THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM last_month_salary_stats
 UNION ALL
 SELECT
   'alto + bajo' AS comparison_group,
   SUM(CASE WHEN income_category IN ('alto', 'bajo') THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN income_category IN ('alto', 'bajo') THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM last_month_salary_stats
 UNION ALL
 SELECT
   'alto + medio' AS comparison_group,
   SUM(CASE WHEN income_category IN ('alto', 'bajo') THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN income_category IN ('alto', 'bajo') THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM last_month_salary_stats
)

-- Calcular riesgo relativo comparando grupos
SELECT
 CASE
   WHEN a.income_category = 'alto' THEN 'alto vs medio + bajo'
   WHEN a.income_category = 'medio' THEN 'medio vs alto + bajo'
   WHEN a.income_category = 'bajo' THEN 'bajo vs alto + medio'
 END AS comparison,
 a.income_category,
 a.total_bad_payers AS a,
 a.total_good_payers AS b,
 c.total_bad_payers AS c,
 c.total_good_payers AS d,
 -- Cálculo del riesgo relativo
 (a.total_bad_payers / (a.total_bad_payers + a.total_good_payers)) / (c.total_bad_payers / (c.total_bad_payers + c.total_good_payers)) AS relative_risk
FROM
 last_month_salary_stats a
JOIN
 combined_groups c
ON
 (a.income_category = 'alto' AND c.comparison_group = 'medio + bajo') OR
 (a.income_category = 'medio' AND c.comparison_group = 'alto + bajo') OR
 (a.income_category = 'bajo' AND c.comparison_group = 'alto + medio')
```

## Riesgo relativo *debt_to_income_ratio_category*

Los clientes con una proporción/cantidad baja de deuda en relación con su salario mensual (hasta 35%) son los de mayor riesgo de incumplimiento de pago, con aproximadamente un 37.66% más de probabilidad de ser malos pagadores. 

bajo: 1.37662130897093<br>
moderado: 0.8386023294509<br>
alto: 0.62600526548764<br>

```sql
-- Calcular cantidad de malos y buenos pagadores por grupo de debt_to_income_ratio
WITH debt_to_income_ratio_stats AS(
 SELECT
   debt_to_income_ratio_category,
   COUNTIF(default_flag = 1) AS total_bad_payers,
   COUNTIF(default_flag = 0) AS total_good_payers
 FROM `laboratoria-426816.proyecto3_riesgorelativo.track_all_data`
 GROUP BY debt_to_income_ratio_category
),

-- Calcular los totales combinados de cada grupo de debt_to_income_ratio
combined_groups AS (
 SELECT
   'moderado + bajo' AS comparison_group,
   SUM(CASE WHEN debt_to_income_ratio_category IN ('moderado', 'bajo') THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN debt_to_income_ratio_category IN ('moderado', 'bajo') THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM debt_to_income_ratio_stats
 UNION ALL
 SELECT
   'alto + bajo' AS comparison_group,
   SUM(CASE WHEN debt_to_income_ratio_category IN ('alto', 'bajo') THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN debt_to_income_ratio_category IN ('alto', 'bajo') THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM debt_to_income_ratio_stats
 UNION ALL
   SELECT
   'alto + moderado' AS comparison_group,
   SUM(CASE WHEN debt_to_income_ratio_category IN ('alto', 'moderado') THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN debt_to_income_ratio_category IN ('alto', 'moderado') THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM debt_to_income_ratio_stats
)

-- Calcular riesgo relativo comparando grupos
SELECT
 CASE
   WHEN a.debt_to_income_ratio_category = 'alto' THEN 'alto vs moderado + bajo'
   WHEN a.debt_to_income_ratio_category = 'moderado' THEN 'moderado vs alto + bajo'
   WHEN a.debt_to_income_ratio_category = 'bajo' THEN 'bajo vs alto + moderado'
 END AS comparison,
 a.debt_to_income_ratio_category,
 a.total_bad_payers AS a,
 a.total_good_payers AS b,
 c.total_bad_payers AS c,
 c.total_good_payers AS d,
 -- Cálculo del riesgo relativo
 (a.total_bad_payers / NULLIF(a.total_bad_payers + a.total_good_payers, 0)) /
(c.total_bad_payers / NULLIF(c.total_bad_payers + c.total_good_payers, 0)) AS relative_risk
FROM
 debt_to_income_ratio_stats a
JOIN
 combined_groups c
ON
 (a.debt_to_income_ratio_category = 'alto' AND c.comparison_group = 'moderado + bajo') OR
 (a.debt_to_income_ratio_category = 'moderado' AND c.comparison_group = 'alto + bajo') OR
 (a.debt_to_income_ratio_category = 'bajo' AND c.comparison_group = 'alto + moderado')
```

## Riesgo relativo *using_lines_not_secured_personal_assets_category*

Los prestatarios en el tercil 3 (0.361026158 a 22000) tienen aproximadamente un 6711.39% más de probabilidad de incumplir con sus pagos. 

Tercil 1 (0 a 0.052055761): 0.02605643455004<br>
Tercil 2 (0.052071134 a 0.360954358): 0.0326811165237444<br>
Tercil 3 (0.361026158 a 22000): 67.11394089316166<br>

Riesgo relativo tercil 3 *using_lines_not_secured_personal_assets_category*
```sql
-- Calcular terciles para using_lines_not_secured_personal_assets
WITH terciles AS (
SELECT
  user_id,
  using_lines_not_secured_personal_assets,
  default_flag,
  NTILE(3) OVER (ORDER BY using_lines_not_secured_personal_assets) AS tercil
FROM `laboratoria-426816.proyecto3_riesgorelativo.track_all_data`
),

tercil_grouping AS (
 SELECT
   user_id,
   using_lines_not_secured_personal_assets,
   CASE
     WHEN tercil = 3 THEN 'tercil 3'
     ELSE 'tercil 1 + tercil 2'
   END AS tercil_group,
   default_flag
 FROM terciles
),

-- Calcular valores mínimo y máximo de tercil
tercil_stats AS (
 SELECT
   MIN(using_lines_not_secured_personal_assets) AS min_using_lines_not_secured_personal_assets,
   MAX(using_lines_not_secured_personal_assets) AS max_using_lines_not_secured_personal_assets
 FROM terciles
 WHERE tercil = 3
),

-- Contabilizar flags de default_flag
event_counts AS (
SELECT
  tercil_group,
  SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS defaults,
  SUM(CASE WHEN default_flag = 0 THEN 1 ELSE 0 END) AS non_defaults -- Considerar 0 como 1 para poder sumar
FROM tercil_grouping
GROUP BY tercil_group
),

-- Calcular riesgo relativo
relative_risk AS (
SELECT
  -- Variables para "tercil 3"
  MAX(CASE WHEN tercil_group = 'tercil 3' THEN defaults ELSE 0 END) AS a,
  MAX(CASE WHEN tercil_group = 'tercil 3' THEN non_defaults ELSE 0 END) AS b,
  -- Variables para "tercil 1 + tercil 2"
  MAX(CASE WHEN tercil_group = 'tercil 1 + tercil 2' THEN defaults ELSE 0 END) AS c,
  MAX(CASE WHEN tercil_group = 'tercil 1 + tercil 2' THEN non_defaults ELSE 0 END) AS d
FROM event_counts
)

SELECT
 a, b, c, d,
 SAFE_DIVIDE(a, (a + b)) / SAFE_DIVIDE(c, (c + d)) AS relative_risk,
 t.min_using_lines_not_secured_personal_assets,
 t.max_using_lines_not_secured_personal_assets
FROM relative_risk, tercil_stats t

-- Resultado: RR = 67.11394089316166, tercil 3 = 0.361026158 a 22000.0
```

## Riesgo relativo *total_loans*

El grupo de clientes con menor cantidad de préstamos, el tercil 1 (1 a 6 préstamos), es el de mayor riesgo, con aproximadamente un 115.32% más de probabilidad de incumplir con sus pagos. 

tercil 1 (1 a 6 loans): 2.153151755347555<br>
tercil 2 (6 a 10 loans): 0.6928396703033<br>
tercil 3 (10 a 57 loans): 0.5773439306710<br>

```sql
-- Calcular terciles para total_loans y agregar rango de valores
WITH total_loans_terciles AS (
 SELECT
   total_loans,
   default_flag,
   NTILE(3) OVER (ORDER BY total_loans) AS total_loans_tercil
 FROM `laboratoria-426816.proyecto3_riesgorelativo.track_all_data`
),

-- Calcular los valores mínimo y máximo de total_loans para cada tercil
tercil_ranges AS (
 SELECT
   total_loans_tercil,
   MIN(total_loans) AS min_total_loans,
   MAX(total_loans) AS max_total_loans
 FROM total_loans_terciles
 GROUP BY total_loans_tercil
),

-- Calcular cantidad de malos y buenos pagadores por total_loans_tercil
total_loans_stats AS (
 SELECT
   total_loans_tercil,
   COUNTIF(default_flag = 1) AS total_bad_payers,
   COUNTIF(default_flag = 0) AS total_good_payers
 FROM total_loans_terciles
 GROUP BY total_loans_tercil
),

-- Calcular los totales combinados de cada grupo de total_loans_tercil
combined_groups AS (
 SELECT
   'tercil 1 + tercil 2' AS comparison_group,
   SUM(CASE WHEN total_loans_tercil IN (1, 2) THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN total_loans_tercil IN (1, 2) THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM total_loans_stats
 UNION ALL
 SELECT
   'tercil 2 + tercil 3' AS comparison_group,
   SUM(CASE WHEN total_loans_tercil IN (2, 3) THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN total_loans_tercil IN (2, 3) THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM total_loans_stats
 UNION ALL
 SELECT
   'tercil 1 + tercil 3' AS comparison_group,
   SUM(CASE WHEN total_loans_tercil IN (1, 3) THEN total_bad_payers ELSE 0 END) AS total_bad_payers,
   SUM(CASE WHEN total_loans_tercil IN (1, 3) THEN total_good_payers ELSE 0 END) AS total_good_payers
 FROM total_loans_stats
)

-- Calcular riesgo relativo comparando grupos
SELECT
 a.total_loans_tercil,
 r.min_total_loans,
 r.max_total_loans,
 CASE
   WHEN a.total_loans_tercil = 1 THEN 'tercil 1 vs tercil 2 + tercil 3'
   WHEN a.total_loans_tercil = 2 THEN 'tercil 2 vs tercil 1 + tercil 3'
   WHEN a.total_loans_tercil = 3 THEN 'tercil 3 vs tercil 1 + tercil 2'
 END AS comparison,
 a.total_bad_payers AS a,
 a.total_good_payers AS b,
 c.total_bad_payers AS c,
 c.total_good_payers AS d,
 -- Cálculo del riesgo relativo
 CASE
   WHEN (a.total_bad_payers + a.total_good_payers = 0) OR (c.total_bad_payers + c.total_good_payers = 0) THEN NULL
   ELSE (a.total_bad_payers / (a.total_bad_payers + a.total_good_payers)) /
        (c.total_bad_payers / (c.total_bad_payers + c.total_good_payers))
 END AS relative_risk
FROM
 total_loans_stats a
JOIN
 combined_groups c
ON
 (a.total_loans_tercil = 1 AND c.comparison_group = 'tercil 2 + tercil 3') OR
 (a.total_loans_tercil = 2 AND c.comparison_group = 'tercil 1 + tercil 3') OR
 (a.total_loans_tercil = 3 AND c.comparison_group = 'tercil 1 + tercil 2')
JOIN
 tercil_ranges r
ON
 a.total_loans_tercil = r.total_loans_tercil;
```

## Riesgo relativo *more_90_days_overdue*

Los clientes que sí registran retrasos de más de 90 días poseen un riesgo severamente alto, de un 3.699.900% más de probabilidad de ser malos pagadores en relación al grupo que no registra retrasos de ese tipo. 

Más de 90 días de retraso: 192.34871702059<br>
Sin más de 90 días de retraso: 0.0051988909283597886<br>

```sql
Riesgo relativo *more_90_days_overdue* grupo 1
-- Clasificar clientes según si se han atrasado o no más de 90 días
WITH overdue_classification AS (
 SELECT
   user_id,
   CASE
     WHEN COUNTIF(more_90_days_overdue > 0) THEN 'atrasado'
     ELSE 'no_atrasado'
   END AS overdue_status
 FROM `laboratoria-426816.proyecto3_riesgorelativo.track_all_data`
),

-- Calcular cantidad de malos y buenos pagadores por estado de atraso
total_loans_stats AS (
 SELECT
   overdue_status,
   COUNTIF(default_flag = 1) AS total_bad_payers,
   COUNTIF(default_flag = 0) AS total_good_payers
 FROM `laboratoria-426816.proyecto3_riesgorelativo.track_all_data` a
 JOIN overdue_classification b
 ON a.user_id = b.user_id
 GROUP BY overdue_status
),

-- Calcular los totales combinados de cada estado de atraso
combined_groups AS (
 SELECT
   'atrasado vs no_atrasado' AS comparison_group,
   SUM(CASE WHEN overdue_status = 'atrasado' THEN total_bad_payers ELSE 0 END) AS total_bad_payers_atrasado,
   SUM(CASE WHEN overdue_status = 'atrasado' THEN total_good_payers ELSE 0 END) AS total_good_payers_atrasado,
   SUM(CASE WHEN overdue_status = 'no_atrasado' THEN total_bad_payers ELSE 0 END) AS total_bad_payers_no_atrasado,
   SUM(CASE WHEN overdue_status = 'no_atrasado' THEN total_good_payers ELSE 0 END) AS total_good_payers_no_atrasado
 FROM total_loans_stats
)

-- Calcular riesgo relativo comparando grupos
SELECT
 c.comparison_group,
 c.total_bad_payers_atrasado AS bad_payers_atrasado,
 c.total_good_payers_atrasado AS good_payers_atrasado,
 c.total_bad_payers_no_atrasado AS bad_payers_no_atrasado,
 c.total_good_payers_no_atrasado AS good_payers_no_atrasado,
 -- Cálculo del riesgo relativo
 CASE
   WHEN (c.total_bad_payers_atrasado + c.total_good_payers_atrasado = 0) OR (c.total_bad_payers_no_atrasado + c.total_good_payers_no_atrasado = 0) THEN NULL
   ELSE (c.total_bad_payers_atrasado / NULLIF(c.total_bad_payers_atrasado + c.total_good_payers_atrasado, 0)) /
        (c.total_bad_payers_no_atrasado / NULLIF(c.total_bad_payers_no_atrasado + c.total_good_payers_no_atrasado, 0))
 END AS relative_risk
FROM combined_groups c
```

## Riesgo relativo *high_debt_indicator*

Los clientes con alto nivel de endeudamiento (ratio de deuda por encima del 40%-43%) posee aproximadamente un 58.06% más de probabilidad de incumplir con sus pagos, constituyendo el grupo de riesgo.

Deuda alta: 1.38871227575673<br>
Deuda baja o moderada: 0.7200915678916<br>

Riesgo relativo deuda alta 
```sql
-- Contabilizar flags de high_debt_indicator (1 = alto endeudamiento, 0  = no hay alto endeudamiento)
WITH event_counts AS (
SELECT
  high_debt_indicator,
  SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS defaults,
  SUM(CASE WHEN default_flag = 0 THEN 1 ELSE 0 END) AS non_defaults -- Considerar 0 como 1 para poder sumar
FROM `laboratoria-426816.proyecto3_riesgorelativo.track_all_data`
GROUP BY high_debt_indicator
),

-- Calcular riesgo relativo
relative_risk AS (
SELECT
  -- Variables para el grupo "1"
  MAX(CASE WHEN high_debt_indicator = 1 THEN defaults ELSE 0 END) AS a,
  MAX(CASE WHEN high_debt_indicator = 1 THEN non_defaults ELSE 0 END) AS b,
  -- Variables para el grupo "2"
  MAX(CASE WHEN high_debt_indicator = 0 THEN defaults ELSE 0 END) AS c,
  MAX(CASE WHEN high_debt_indicator = 0 THEN non_defaults ELSE 0 END) AS d
FROM event_counts
)

SELECT
a, b, c, d,
SAFE_DIVIDE(a, (a + b)) / SAFE_DIVIDE(c, (c + d)) AS relative_risk
FROM relative_risk
```

## Riesgo relativo *total_delayed_payments*

El único grupo de riesgo (alto) es el de clientes con 1 a 5 retrasos, con aproximadamente 402.75% más de probabilidad de incumplir sus pagos en comparación con los demás grupos, cifra altamente significativa.

0 retrasos = 0.0<br>
1 a 5 retrasos = 5.0275475788715891<br>
6 a 18 retrasos = 0.74869109947643<br>

Riesgo relativo tercil 1 *total_delayed_payments*
```sql
-- Calcular terciles para total_delayed_payments
WITH terciles AS (
SELECT
  user_id,
  total_delayed_payments,
  default_flag,
  NTILE(3) OVER (ORDER BY total_delayed_payments) AS total_delayed_payments_tercil
FROM `laboratoria-426816.proyecto3_riesgorelativo.track_all_data`
),

tercil_grouping AS (
 SELECT
   user_id,
   total_delayed_payments,
   CASE
     WHEN total_delayed_payments_tercil = 1 THEN 'tercil 1'
     ELSE 'tercil 2 + tercil 3'
   END AS tercil_group,
   default_flag
 FROM terciles
),

-- Calcular valores mínimo y máximo de tercil
tercil_stats AS (
SELECT
  MIN(total_delayed_payments) AS min_total_delayed_payments,
  MAX(total_delayed_payments) AS max_total_delayed_payments
FROM terciles
WHERE total_delayed_payments_tercil = 1
),

-- Contabilizar flags de default_flag
event_counts AS (
SELECT
  tercil_group,
  SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS defaults,
  SUM(CASE WHEN default_flag = 0 THEN 1 ELSE 0 END) AS non_defaults -- Considerar 0 como 1 para poder sumar
FROM tercil_grouping
GROUP BY tercil_group
),

-- Calcular riesgo relativo
relative_risk AS (
SELECT
  -- Variables para "tercil 1"
  MAX(CASE WHEN tercil_group = 'tercil 1' THEN defaults ELSE 0 END) AS a,
  MAX(CASE WHEN tercil_group = 'tercil 1' THEN non_defaults ELSE 0 END) AS b,
  -- Variables para "tercil 2 + tercil 3"
  MAX(CASE WHEN tercil_group = 'tercil 2 + tercil 3' THEN defaults ELSE 0 END) AS c,
  MAX(CASE WHEN tercil_group = 'tercil 2 + tercil 3' THEN non_defaults ELSE 0 END) AS d
FROM event_counts
)

SELECT
a, b, c, d,
SAFE_DIVIDE(a, (a + b)) / SAFE_DIVIDE(c, (c + d)) AS relative_risk,
t.min_total_delayed_payments,
t.max_total_delayed_payments
FROM relative_risk, tercil_stats t
```

## Riesgo relativo *number_dependents*

El grupo de clientes con 1 a 5 dependientes a su cargo es el con mayor riesgo, con aproximadamente un un 168.91% más de riesgo de incumplir con sus pagos en comparación con los clientes de los otros grupos. Le sigue el grupo de clientes con 6 o más dependientes, con aproximadamente un 60.74% más de riesgo de posibilidad de ser mal pagador. Ambos grupos tienen una alta probabilidad, pero la de 6 o más dependientes es altamente significativa.

0 dependientes: 0.6148548117543<br>
1 a 5 dependientes: 1.6073752889681<br>
6 a 13 dependientes: 2.6891407512116<br>

Riesgo relativo “1 a 5 dependientes” *number_dependents*
```sql
-- Calcular terciles para number_dependents
WITH categories AS (
 SELECT
   user_id,
   number_dependents,
   default_flag,
   CASE
     WHEN number_dependents = 0 THEN '0 dependientes'
     WHEN number_dependents BETWEEN 1 AND 5 THEN '1 a 5 dependientes'
     ELSE '6 o mas dependientes'
   END AS number_dependents_category
 FROM `laboratoria-426816.proyecto3_riesgorelativo.track_all_data`
),

category_grouping AS (
 SELECT
   user_id,
   number_dependents,
   CASE
     WHEN number_dependents_category = '1 a 5 dependientes' THEN '1 a 5 dependientes'
     ELSE '0 dependientes + 6 o mas dependientes'
   END AS category_groups,
   default_flag
 FROM categories
),

-- Calcular valores mínimo y máximo de tercil
group_stats AS (
 SELECT
   MIN(number_dependents) AS min_number_dependents,
   MAX(number_dependents) AS max_number_dependents
 FROM category_grouping
 WHERE category_groups = '1 a 5 dependientes'
),

-- Contabilizar flags de default_flag
event_counts AS (
 SELECT
   category_groups,
   SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS defaults,
   SUM(CASE WHEN default_flag = 0 THEN 1 ELSE 0 END) AS non_defaults
 FROM category_grouping
 GROUP BY category_groups
),

-- Calcular riesgo relativo
relative_risk AS (
 SELECT
   -- Variables para "1 a 5 dependientes"
   MAX(CASE WHEN category_groups = '1 a 5 dependientes' THEN defaults ELSE 0 END) AS a,
   MAX(CASE WHEN category_groups = '1 a 5 dependientes' THEN non_defaults ELSE 0 END) AS b,
   -- Variables para "0 dependientes + 6 o mas dependientes"
   MAX(CASE WHEN category_groups = '0 dependientes + 6 o mas dependientes' THEN defaults ELSE 0 END) AS c,
   MAX(CASE WHEN category_groups = '0 dependientes + 6 o mas dependientes' THEN non_defaults ELSE 0 END) AS d
 FROM event_counts
)

SELECT
 a, b, c, d,
 SAFE_DIVIDE(a, (a + b)) / SAFE_DIVIDE(c, (c + d)) AS relative_risk,
 g.min_number_dependents,
 g.max_number_dependents
FROM relative_risk, group_stats g

-- Resultado: 1.6073752889681, "1 a 5 dependientes".
```

Riesgo relativo “6 o mas dependientes” *number_dependents*
```sql
-- Calcular terciles para number_dependents
WITH categories AS (
 SELECT
   user_id,
   number_dependents,
   default_flag,
   CASE
     WHEN number_dependents = 0 THEN '0 dependientes'
     WHEN number_dependents BETWEEN 1 AND 5 THEN '1 a 5 dependientes'
     ELSE '6 o mas dependientes'
   END AS number_dependents_category
 FROM `laboratoria-426816.proyecto3_riesgorelativo.track_all_data`
),

category_grouping AS (
 SELECT
   user_id,
   number_dependents,
   CASE
     WHEN number_dependents_category = '6 o mas dependientes' THEN '6 o mas dependientes'
     ELSE '0 dependientes + 1 a 5 dependientes'
   END AS category_groups,
   default_flag
 FROM categories
),

-- Calcular valores mínimo y máximo de tercil
group_stats AS (
 SELECT
   MIN(number_dependents) AS min_number_dependents,
   MAX(number_dependents) AS max_number_dependents
 FROM category_grouping
 WHERE category_groups = '6 o mas dependientes'
),

-- Contabilizar flags de default_flag
event_counts AS (
 SELECT
   category_groups,
   SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS defaults,
   SUM(CASE WHEN default_flag = 0 THEN 1 ELSE 0 END) AS non_defaults
 FROM category_grouping
 GROUP BY category_groups
),

-- Calcular riesgo relativo
relative_risk AS (
 SELECT
   -- Variables para "6 o mas dependientes"
   MAX(CASE WHEN category_groups = '6 o mas dependientes' THEN defaults ELSE 0 END) AS a,
   MAX(CASE WHEN category_groups = '6 o mas dependientes' THEN non_defaults ELSE 0 END) AS b,
   -- Variables para "0 dependientes + 1 a 5 dependientes"
   MAX(CASE WHEN category_groups = '0 dependientes + 1 a 5 dependientes' THEN defaults ELSE 0 END) AS c,
   MAX(CASE WHEN category_groups = '0 dependientes + 1 a 5 dependientes' THEN non_defaults ELSE 0 END) AS d
 FROM event_counts
)

SELECT
 a, b, c, d,
 SAFE_DIVIDE(a, (a + b)) / SAFE_DIVIDE(c, (c + d)) AS relative_risk,
 g.min_number_dependents,
 g.max_number_dependents
FROM relative_risk, group_stats g

-- Resultado: 2.6891407512116, "6 o mas dependientes".
```

## Riesgo relativo *unsecured_credit_ratio*

Los resultados arrojan null para los primeros dos segmentos y 0.0 para el último segmento, por lo que se descartará el riesgo relativo de esta variable.

bajo (0 - 1): null<br>
medio (1 - 2.5): null<br>
alto (> 2.5): 0.0<br>

## Conclusiones riesgo relativo

Factores de mayor riesgo resumidos:
- *age*: Grupo etario joven (21 a 34 años), con aproximadamente un 82.19% más de probabilidad de ser mal pagador. Le sigue el grupo adulto (35 a 54 años), con aproximadamente un 93.56% más de probabilidad de ser mal pagador. 
- *income_category*: Clientes con salario bajo (hasta $4679), quienes tienen aproximadamente un 44.83% más de probabilidad de ser mal pagadores.
- *debt_to_income_ratio*: Clientes con una proporción/cantidad baja de deuda en relación con salario mensual (hasta 35%), con aproximadamente un 37.66% más de probabilidad de ser malos pagadores. 
- *using_lines_not_secured_personal_assets_category*: Clientes en el tercil 3 (0.361026158 a 22000) de cuánto están utilizando en relación con su límite de crédito (en líneas que no están garantizadas con bienes personales, como inmuebles y automóviles) tienen aproximadamente un 6711.39% más de probabilidad de incumplir con sus pagos.
- *total_loans*: Clientes con menor cantidad de préstamos, tercil 1 (1 a 6 préstamos), con aproximadamente un 115.32% más de probabilidad de incumplir con sus pagos. 
- *more_90_days_overdue*: Clientes que sí registran retrasos de más de 90 días, con aproximadamente un 3.699.900% más de probabilidad de ser malos pagadores.
- *high_debt_indicator*: Clientes con alto nivel de endeudamiento (ratio de deuda por encima del 40%-43%), con aproximadamente un 58.06% más de probabilidad de incumplir con sus pagos.
- *total_delayed_payments*: Clientes con 1 a 5 retrasos de pagos, con aproximadamente 402.75% más de probabilidad de incumplir sus pagos.
- *number_dependents*: Clientes con 6 o más dependientes a su cargo, con aproximadamente un un 168.91% más de riesgo de incumplir con sus pagos. Le sigue el grupo de clientes con 1 a 5 dependientes, con aproximadamente un 60.74% más de riesgo de posibilidad de ser mal pagador.

## Validación hipótesis

1. Los más jóvenes tienen un mayor riesgo de impago.  ✅ **VERDADERO** Son los clientes jóvenes (21 a 34 años) quienes representan el mayor grupo de riesgo.
2. Las personas con más cantidad de préstamos activos tienen mayor riesgo de ser malos pagadores. ❎ **FALSO** Son los clientes con baja cantidad de préstamos quienes representan el mayor grupo de riesgo (tercil 1, de 1 a 6 préstamos).
3. Las personas que han retrasado sus pagos por más de 90 días tienen mayor riesgo de ser malos pagadores. ✅ **VERDADERO** La incurrencia en retrasos de más de 90 días posee un riesgo relativo de 192,35, siendo extremadamente significativa.

## Hallazgos 

- Los malos pagadores constituyen un 16,4% (622) del total de prestatarios (35.575).
- Las variables altamente significativas a la hora de determinar riesgo de impago son contar con retrasos de más de 90 días y luego un uso alto de líneas de crédito no aseguradas en bienes personales. Esto subraya la severidad del impacto que los retrasos prolongados en los pagos y una alta exposición al crédito no garantizado pueden tener en la probabilidad de incumplimiento.
- Los retrasos de más de 90 días son más relevantes que los de 60 a 89 días y los de 30 a 59 días.
- La probabilidad de incumplimiento de pago disminuye con la edad, lo cual puede reflejar una mayor estabilidad financiera y experiencia en la gestión de crédito a medida que las personas envejecen. Factores como la estabilidad laboral, la carga de deuda, los ingresos y el historial crediticio podrían estar influyendo en el riesgo.
- Los resultados muestran una clara relación inversa entre el nivel de ingresos y el riesgo de incumplimiento de pagos, así como una relación inversa entre el total de préstamos y el riesgo de impago.
- El grupo de clientes con 1 a 5 dependientes a su cargo (intermedio) es el de mayor riesgo.
- Los clientes con una proporción baja de deuda en relación con su salario mensual (hasta 35%) son los de mayor riesgo de incumplimiento de pago. Esto puede parecer contraintuitivo, pero podría indicar que otros factores, como otros ratios de deuda, el historial crediticio o el comportamiento financiero también juegan un papel importante.
- Los clientes con alto nivel de endeudamiento en relación a su patrimonio (ratio de deuda por encima del 40%-43%) poseen mayor probabilidad de incumplir con sus pagos.
- Los créditos de otros tipo (vs. los inmobiliarios) son aquellos donde más clientes riesgosos se concentran . Representan el 88,07% del total de créditos, aunque poseen porcentaje de incumplimiento de pago prácticamente igual al de los inmobiliarios (1,41% vs. 1,40%).

## Hallazgos 
- Los malos pagadores constituyen un 16,4% (622) del total de prestatarios (35.575).
- Las variables altamente significativas a la hora de determinar riesgo de impago son contar con retrasos de más de 90 días y luego un uso alto de líneas de crédito no aseguradas en bienes personales. Esto subraya la severidad del impacto que los retrasos prolongados en los pagos y una alta exposición al crédito no garantizado pueden tener en la probabilidad de incumplimiento.
- Los retrasos de más de 90 días son más relevantes que los de 60 a 89 días y los de 30 a 59 días.
- La probabilidad de incumplimiento de pago disminuye con la edad, lo cual puede reflejar una mayor estabilidad financiera y experiencia en la gestión de crédito a medida que las personas envejecen. Factores como la estabilidad laboral, la carga de deuda, los ingresos y el historial crediticio podrían estar influyendo en el riesgo.
- Los resultados muestran una clara relación inversa entre el nivel de ingresos y el riesgo de incumplimiento de pagos, así como una relación inversa entre el total de préstamos y el riesgo de impago.
- El grupo de clientes con 1 a 5 dependientes a su cargo (intermedio) es el de mayor riesgo.
- Los clientes con una proporción baja de deuda en relación con su salario mensual (hasta 35%) son los de mayor riesgo de incumplimiento de pago. Esto puede parecer contraintuitivo, pero podría indicar que otros factores, como otros ratios de deuda, el historial crediticio o el comportamiento financiero también juegan un papel importante.
- Los clientes con alto nivel de endeudamiento en relación a su patrimonio (ratio de deuda por encima del 40%-43%) poseen mayor probabilidad de incumplir con sus pagos.
- Los créditos de otros tipo (vs. los inmobiliarios) son aquellos donde más clientes riesgosos se concentran . Representan el 88,07% del total de créditos, aunque poseen porcentaje de incumplimiento de pago prácticamente igual al de los inmobiliarios (1,41% vs. 1,40%).
