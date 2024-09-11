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
  - *user_id*: número de identificación del cliente (único para cada cliente).
  - *age*: edad del cliente.
  - *sex*: género del cliente.
  - *last_month_salary*: último salario mensual que el cliente reportó al banco.
  - *number_dependents*: número de dependientes.
- *loans_outstanding*
  - *loan_id*
  - *loan_type*
  - *using_lines_not_secured_personal_assets*
  - *more_90_days_overdue*
  - *number_times_delayed_payment_loan_60_89_days*
  - *number_times_delayed_payment_loan_30_59_days*
  - *debt_ratio*
- *defaul_flag*
  - *user_id*
  - *default_flag*









