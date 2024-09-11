# Riesgo relativo para préstamos bancarios

## Resumen

La disminución de tasas de interés en el mercado ha desencadenado un aumento notable en la demanda de solicitudes de crédito. A su vez, la creciente tasa de incumplimiento (default) ha aumentado la presión sobre la insitución bancaria para identificar y mitigar los riesgos asociados con el crédito. Ante ello, la metodología manual del banco Super Caja para evaluación de solicitud de préstamo ha resultado en un proceso ineficiente y demorado, afectando negativamente la eficacia y la rapidez de procesamiento. Por tanto, se requiere determinar a cabalidad las variables de mayor riesgo de incumplimiento de pago y la automatización del proceso de análisis, con el objetivo de optimizar la eficiencia, la precisión y la rapidez en la evaluación de las solicitudes de crédito. 

## Herramientas utilizadas

BigQuery, Google Colab, Google Sheets, Looker Studio y Google Slides.

## Lenguajes utilizados

SQL y Pyhton

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

### Correlación de Pearson

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

### Desviación estándar

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







