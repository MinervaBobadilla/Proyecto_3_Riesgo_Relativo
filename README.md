# **Proyecto_3_Riesgo_Relativo**
**Ficha Técnica: Proyecto 3 Riesgo Relativo**

**Título del Proyecto**

**Proyecto 3  Riesgo Relativo**

**Objetivo**
Armar un score crediticio a partir de un análisis de datos y la evaluación del riesgo relativo que pueda clasificar a los solicitantes en diferentes categorías de riesgo basadas en su probabilidad de incumplimiento. Esta clasificación permitirá al banco tomar decisiones informadas sobre a quién otorgar crédito, reduciendo así el riesgo de préstamos no reembolsables. Además, la integración de la métrica existente de pagos atrasados fortalecerá la capacidad del modelo para identificar riesgos, lo que en última instancia contribuirá a la solidez financiera y la eficiencia operativa del banco.


**Equipo**
Trabajo personal

**Herramientas y Tecnologías**

1) Listado de herramientas y tecnologías utilizadas en el proyecto:
2) Big Query (SQL).
3) Google Colab (Python).
4) Looker Studio.
5) Workspace Google ( Presentaciones, Gemini y Documentos).
6) Videos Looms.


**Procesamiento y análisis**
**Proceso de Análisis de Datos:**

**1.1 Procesar y preparar base de datos**

a) Identificar y manejar valores nulos.

b) Identificar y manejar valores duplicados.

c) Identificar y manejar datos fuera del alcance del análisis.

d) Identificar y manejar datos discrepantes en variables categóricas.

e) Identificar y manejar datos discrepantes en variables numéricas.

f) Comprobar y cambiar tipo de dato.

g) Unir tablas.

h) Crear nuevas variables.

i) Construir tablas auxiliares.


A continuación ejemplos de lo anterior:





```
SELECT 
C.user_id,
C.age,
C.sex,
C.last_month_salary,
C.number_dependents,
D.default_flag,
E.more_90_days_overdue,
E.using_lines_not_secured_personal_assets,
E.number_times_delayed_payment_loan_30_59_days,
E.debt_ratio,
E.number_times_delayed_payment_loan_60_89_days



FROM
 `Dataset.default` D
 
LEFT JOIN

 `Dataset.user_info` C
 
ON
  C.user_id = D.user_id

LEFT JOIN
`Dataset.loans_detail` E
  
ON
C.user_id = E.user_id

  WHERE
C.user_id IS NOT NULL


  
```





```

WITH
  debt_ratio_percentiles AS (
    SELECT
 
      PERCENTILE_CONT(debt_ratio, 0.02) OVER () AS lower_debt_ratio,
      PERCENTILE_CONT(debt_ratio, 0.99) OVER () AS upper_debt_ratio
    FROM
      `Dataset.view_user_info_general_VI`
    WHERE
   debt_ratio
 IS NOT NULL
  ),

  winsorizacion AS (
    SELECT
      V.*,
      CASE 
       WHEN V.debt_ratio < dr.lower_debt_ratio THEN dr.lower_debt_ratio
      WHEN V.debt_ratio > dr.upper_debt_ratio THEN dr.upper_debt_ratio
      ELSE V.debt_ratio
      END AS debt_ratio_winsorizado
    FROM
       `Dataset.view_user_info_general_VI` V,
      (SELECT DISTINCT lower_debt_ratio, upper_debt_ratio FROM debt_ratio_percentiles) dr
  )
SELECT
  *
FROM
  winsorizacion;


  
```




**1.2 Análisis exploratorio**

a) Agrupar datos según variables categóricas.

b) Visualizar las variables categóricas.

c) Aplicar medidas de tendencia central.

d) Visualizar distribución.

e) Aplicar medidas de dispersión.

f) Calcular cuartiles, deciles o percentiles.

g) Calcular correlación entre variables.


A continuación ejemplos de lo anterior:






![image](https://github.com/user-attachments/assets/d40a0c8e-1b48-4aec-abcb-e1225e4fc6e0)








```
SELECT 
CORR (more_90_days_overdue, number_times_delayed_payment_loan_30_59_days) AS correlation_value
FROM `Dataset.user_info_II`


  
```





```
SELECT 
CORR (more_90_days_overdue,number_times_delayed_payment_loan_60_89_days) AS correlation_value
FROM `Dataset.user_info_I`


  
```






![image](https://github.com/user-attachments/assets/77ed28d6-92f6-42c9-afd2-fa6ee4b01629)










```
WITH QUARTILES AS (
    SELECT user_id,
           age,
           default_flag,
           more_90_days_overdue_winsorizado,
           number_times_delayed_payment_loan_30_59_days_winsorizado,
           number_times_delayed_payment_loan_60_89_days_winsorizado,
           debt_ratio_winsorizado,
           last_month_salary_winsorizado,
           using_lines_not_secured_personal_assets,
           NTILE(4) OVER (ORDER BY age) AS quartile_age,
           NTILE(4) OVER (ORDER BY default_flag) AS quartile_default_flag,
           NTILE(4) OVER (ORDER BY more_90_days_overdue_winsorizado) AS quartile_m90do,
           NTILE(4) OVER (ORDER BY number_times_delayed_payment_loan_30_59_days_winsorizado) AS quartile_ntdpl3059_days,
           NTILE(4) OVER (ORDER BY number_times_delayed_payment_loan_60_89_days_winsorizado) AS quartile_ntdpl6089_days,
           NTILE(4) OVER (ORDER BY debt_ratio_winsorizado) AS quartile_debt_ratio,
           NTILE(4) OVER (ORDER BY last_month_salary_winsorizado) AS quartile_lmsw,
           NTILE(4) OVER (ORDER BY using_lines_not_secured_personal_assets_winsorizado) AS quartile_ulnspa
    FROM `Dataset.view_user_info_general_VII`
),
categorized AS (
    SELECT
        q.*,
        CASE
            WHEN QUARTILES.quartile_age = 1 THEN 'q1age'
            WHEN QUARTILES.quartile_age = 2 THEN 'q2age'
            WHEN QUARTILES.quartile_age = 3 THEN 'q3age'
            WHEN QUARTILES.quartile_age = 4 THEN 'q4age'
        END AS cat_q_age,
        CASE
            WHEN QUARTILES.quartile_default_flag = 1 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_default_flag = 2 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_default_flag = 3 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_default_flag = 4 THEN 'mal_pagador'
        END AS cat_q_default_flag,
        CASE
            WHEN QUARTILES.quartile_m90do = 1 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_m90do = 2 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_m90do = 3 THEN 'buen_pagadoro'
            WHEN QUARTILES.quartile_m90do = 4 THEN 'mal_pagador'
        END AS cat_q_m90do,
        CASE
            WHEN QUARTILES.quartile_ntdpl3059_days = 1 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_ntdpl3059_days = 2 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_ntdpl3059_days = 3 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_ntdpl3059_days = 4 THEN 'mal_pagador'
        END AS cat_q_ntdpl3059_days,
        CASE
            WHEN QUARTILES.quartile_ntdpl6089_days = 1 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_ntdpl6089_days = 2 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_ntdpl6089_days = 3 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_ntdpl6089_days = 4 THEN 'mal_pagador'
        END AS cat_q_ntdpl6089_days,
        CASE
            WHEN QUARTILES.quartile_debt_ratio = 1 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_debt_ratio = 2 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_debt_ratio = 3 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_debt_ratio = 4 THEN 'mal_pagador'
        END AS cat_q_debt_ratio,
        CASE
            WHEN QUARTILES.quartile_lmsw = 1 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_lmsw = 2 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_lmsw = 3 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_lmsw = 4 THEN 'mal_pagador'
        END AS cat_q_lmsw,
        CASE
            WHEN QUARTILES.quartile_ulnspa = 1 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_ulnspa = 2 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_ulnspa = 3 THEN 'buen_pagador'
            WHEN QUARTILES.quartile_ulnspa = 4 THEN 'mal_pagador'
        END AS cat_q_ulnspa
    FROM `Dataset.view_user_info_general_VII` q
    LEFT JOIN QUARTILES
    ON q.user_id = QUARTILES.user_id
),
risk_classification AS (
    SELECT
        *,
        (CASE WHEN cat_q_age = 'mal_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_default_flag = 'mal_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_m90do = 'mal_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_ntdpl3059_days = 'mal_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_ntdpl6089_days = 'mal_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_debt_ratio = 'mal_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_lmsw = 'mal_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_ulnspa = 'mal_pagador' THEN 1 ELSE 0 END) AS mal_pagador_count,
        (CASE WHEN cat_q_age = 'buen_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_default_flag = 'buen_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_m90do = 'buen_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_ntdpl3059_days = 'buen_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_ntdpl6089_days = 'buen_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_debt_ratio = 'buen_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_lmsw = 'buen_pagador' THEN 1 ELSE 0 END +
         CASE WHEN cat_q_ulnspa = 'buen_pagador' THEN 1 ELSE 0 END) AS buen_pagador_count
    FROM categorized
)
SELECT
    *,
    CASE
        WHEN mal_pagador_count >= 3 THEN 'riesgo_alto'
        ELSE 'riesgo_bajo'
    END AS risk_level
FROM
    risk_classification
ORDER BY
    user_id;



  
```













**1.3 Aplicar técnica de análisis**

a) Aplicar segmentación.

b) Calcular riesgo relativo

c) Validar hipótesis.

A continuación ejemplos de lo anterior:










```
SELECT
user_id,
age,
number_dependents_,
last_month_salary_winsorizado,
number_times_delayed_payment_loan_30_59_days_winsorizado,
number_times_delayed_payment_loan_60_89_days_winsorizado,
more_90_days_overdue_winsorizado,
using_lines_not_secured_personal_assets_winsorizado,
debt_ratio_winsorizado,
default_flag,
total_loan,
real_state,
other,

CASE WHEN number_times_delayed_payment_loan_30_59_days_winsorizado > 0 AND number_times_delayed_payment_loan_30_59_days_winsorizado < 4 THEN 1 ELSE 0 END AS dummy_ntdpl_30_59_days_wins,
CASE WHEN  number_times_delayed_payment_loan_60_89_days_winsorizado > 0 AND  number_times_delayed_payment_loan_60_89_days_winsorizado < 2 THEN 1 ELSE 0 END AS dummy_ntdpl_60_89_days_wins,
CASE WHEN more_90_days_overdue_winsorizado > 0 AND more_90_days_overdue_winsorizado <= 3 THEN 1 ELSE 0 END AS dummy_more_90_days_overdue_winsorizado,
CASE WHEN total_loan > 1 AND  total_loan <= 5 THEN 1 ELSE 0 END AS dummy_total_loan,
CASE WHEN using_lines_not_secured_personal_assets_winsorizado > 0.564294998 AND using_lines_not_secured_personal_assets_winsorizado < 1.09591878639 THEN 1 ELSE 0 END AS dummy_ulnspawins



FROM `Dataset.view_user_info_general_VII`



  
```






![image](https://github.com/user-attachments/assets/3414cf80-b0fd-4488-b64b-8a1aa6b70be7)










```
WITH Cuartiles AS (
    SELECT
        more_90_days_overdue_winsorizado,
        default_flag,
        NTILE(4) OVER (ORDER BY more_90_days_overdue_winsorizado) AS cuartil
    FROM
      `Dataset.view_user_info_general_VII`
),
ConteoPorCuartil AS (
    SELECT
        cuartil,
        default_flag,
        COUNT(*) AS cantidad,
        MIN(more_90_days_overdue_winsorizado) AS min_more_90_days_overdue_winsorizado,
        MAX(more_90_days_overdue_winsorizado) AS max_more_90_days_overdue_winsorizado
    FROM
        Cuartiles
    GROUP BY
        cuartil,
        default_flag
),
RiskCalculation AS (
    SELECT
        cuartil,
        MIN(min_more_90_days_overdue_winsorizado) AS rango_min_more_90_days_overdue_winsorizado,
        MAX(max_more_90_days_overdue_winsorizado) AS rango_max_more_90_days_overdue_winsorizado,
        SUM(CASE WHEN default_flag = 1 THEN cantidad ELSE 0 END) AS default_count,
        SUM(CASE WHEN default_flag = 0 THEN cantidad ELSE 0 END) AS non_default_count,
        SUM(cantidad) AS total_count,
        SUM(CASE WHEN default_flag = 1 THEN cantidad ELSE 0 END) * 1.0 / SUM(cantidad) AS risk
    FROM
        ConteoPorCuartil
    GROUP BY
        cuartil
),
RelativeRisk AS (
    SELECT
        q2.cuartil AS non_exposed_quartile,
        q2.risk AS non_exposed_risk,
        q1.risk AS exposed_risk,
        q2.risk / q1.risk AS risk_relative_m90dayswins
    FROM
        RiskCalculation q1
    CROSS JOIN
        RiskCalculation q2
    WHERE
        q1.cuartil = 1 -- Cuartil expuesto
)
SELECT
    r.non_exposed_quartile,
    r.risk_relative_m90dayswins,
    rc.rango_min_more_90_days_overdue_winsorizado,
    rc.rango_max_more_90_days_overdue_winsorizado,
    rc.default_count AS cantidad_default_flag_1,
    rc.non_default_count AS cantidad_default_flag_0,
FROM
    RelativeRisk r
JOIN
    RiskCalculation rc
ON
    r.non_exposed_quartile = rc.cuartil
ORDER BY
    non_exposed_quartile;



  
```






![image](https://github.com/user-attachments/assets/36b1c511-5f13-4c81-859f-7b4d37bbbdc5)









**Resultados y Conclusiones**

Durante el desarrollo del proyecto se utilizó el proceso de Análisis de Datos, cálculo de riesgo relativo y la metodología  de segmentación (Cuartiles) para validación  de hipótesis  y clasificar por grupos con mayor riesgo relativo. La entidad bancaria planteó tres hipótesis, sobre qué hace que un cliente sea más riesgoso o mal pagador. Estas hipótesis se detallan a continuación y de esto se puede concluir lo siguiente:

I) Los más jóvenes tienen un mayor riesgo de impago:
De acuerdo a los resultados de la tabla N° 1 podemos indicar que el cuartil 1 con más riesgo de impago son el segmento de 21 hasta 41 años  de edad, por lo anterior se valida la hipótesis que los más jóvenes tienen un mayor riesgo de impago.

II) Las personas con más cantidad de préstamos activos tienen mayor riesgo de ser malos pagadores.
Con los resultados de la tabla N° 2 podemos indicar que el cuartil 1 de la variable total de crédito, son quienes tienen mayor riesgo relativo y son el segmento de 1 hasta 5 créditos, por lo anterior se debe refutar la hipótesis dado que no tenemos especificación de cuáles son los créditos activos.

III)  Las personas que han retrasado sus pagos por más de 90 días tienen mayor riesgo de ser malos pagadores.
Con los resultados obtenidos en la figura N° 3 , se puede indicar que la variable retraso en más de 90 días en el pago, son quienes tienen mayor valor en el cálculo del riesgo relativo. 
Cabe hacer notar que el cuartil 4 es el segmento de 0 hasta 3 cantidad de veces, por lo anterior se valida la hipótesis “Las personas que han retrasado sus pagos por más de 90 días tienen mayor riesgo de ser malos pagadores”.

Además, se informa que del universo de clientes existen 1775 clientes denominados Mal pagador, lo cual corresponde a un 5,1 % del total,  cumpliendo con más de tres condiciones de riesgo (number times delayed payment loan 30 - 59 days, number times delayed payment loan 60 - 89 days y more 90 days overdure, total loan y  using_lines_not_secured_personal_assets_wins).


Dado que la hipótesis  II) Las personas con más cantidad de préstamos activos tienen mayor riesgo de ser malos pagadores. no pudo ser validada, entonces se solicita entregar más información: del tipo de crédito, por ejemplo si está activo o se encuentra caducado, datos en el tiempo de clientes, etc, para optimizar análisis encontrando mayor exactitud a los resultados presentados.
Con los resultados obtenidos en parámetros de matriz de confusión,  se puede indicar lo siguiente: el modelo tiene un rendimiento muy bueno en la clasificación de la Clase 0 (Buen Pagador), pero un rendimiento deficiente en la clasificación de la Clase 1 (Mal Pagador).

En cuanto a los resultados del modelo de acuerdo a la matriz de confusión, éste necesita ser mejorado, especialmente en su capacidad para identificar correctamente a los malos pagadores. Es necesario realizar un análisis más profundo y aplicar técnicas de rebalanceo de datos para obtener un modelo más preciso y confiable.


**Limitaciones/Próximos Pasos**
Sin observaciones.

**Enlaces de interés**

[Proyecto_3_Riesgo_Relativo.pdf](https://drive.google.com/file/d/1ws1om_FeAC3lTGDj2if46MP8QsVhcLmj/view?usp=drive_link)


[Dashboard](https://lookerstudio.google.com/reporting/f3bf4299-e090-4b0e-9043-0110cf64ccc3) 


[Proyecto 3 Riesgo Relativo Matriz de confusion.ipynb](https://colab.research.google.com/drive/1prdVy0Z4MhFwBQAdp48CzWua8iK8tmdu?usp=sharing)


[Proyecto 3  Riesgo Relativo Análisis Bancario (M. Bobadilla)](https://docs.google.com/presentation/d/1_Y92Fv69cQo3SMJMb2j6CGvj83PefTUt40iuy1YE4VU/edit?usp=sharing)


[Presentación video](https://www.loom.com/share/4c3831ccf421481484991de5001eefb9?sid=c0f6ed5c-079a-49be-9af0-f9fffd9dd35f)







