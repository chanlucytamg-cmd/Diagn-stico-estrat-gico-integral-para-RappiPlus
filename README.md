# Diagnostico estrategico integral para RappiPlus**
El equipo de negocio no tiene claro si el servicio está cumpliendo su objetivo.

Existen dudas clave:

¿Los usuarios realmente compran más?
¿El modelo está generando ganancias?
¿Se están perdiendo oportunidades en el proceso de compra?
Para responder estas preguntas, trabajarás con datos de pedidos, catálogo y marketing.

**Objetivo del proyecto**
Responder a las siguientes preguntas

¿Podemos confiar en los datos?
¿Estamos ganando dinero?
¿Dónde se pierden los usuarios?
¿Los usuarios regresan?
¿Los cambios generan impacto?
¿Cómo comunicamos todo esto?

**Flujo del proyecto**

1¿Podemos confiar en los datos?	Dataset limpio

<head></head>

2¿El negocio es rentable?	KPIS(Revenue, cost, profit)

<head></head>

3¿Dónde se pierden usuarios?	Funnel

<head></head>

4¿Los usuarios regresan?	Cohortes

<head></head>

5¿Los cambios funcionan?	Test estadístico

<head></head>

6¿Cómo comunicamos?	Dashboard

<head></head>

**Dataset del proyecto**

rappiplus_orders_raw.csv
<head></head>

rappiplus_catalog.csv
<head></head>

rappiplus_marketing_spend.csv
<head></head>

**Herramientas utilizadas**

Jupyter Notebook
<head></head>

Python: pandas, numpy, seaborn, matplotlib
<head></head>

Tableau
<head></head>

Visualizaciones nativas (barras, líneas, tablas, KPI).
<head></head>

Modelado de datos en esquema estrella.
<head></head>

Cálculos analíticos (medidas y columnas calculadas).

**Cálculo de KPIs principales**

📊 **Rentabilidad del negocio**

¿Cuál es el ingreso total (revenue)?
<head></head>

**Revenue total**
<head></head>

51,976,346.94

¿Cuál es el costo total?
<head></head>

**Costo general**

<head></head>

43,124,069.01

<head></head>

¿Cuánto se ha invertido en marketing?

<head></head>

**Gasto marketing**

<head></head>

2,871,843.53

<head></head>

¿El negocio es rentable? (calcular profit)
<head></head>

**Ganancia neta**
<head></head>

5,980,434.4
<head></head>

**Margen neto:**
<head></head>
11.51%

<head></head>

Con estos resultados se concluyo que **El negocio si es rentable, genera una ganancia neta superior a los 5.98 millones de pesos. manteniendo una rentabilidad del 11.51% después de cubrir tanto los costos directos de producto como la estrategia de marketing.**

<head></head>

**📈 Parte 2: Comportamiento de ventas**

<img width="700" height="377" alt="Captura de pantalla 2026-10-29 a las 11 08 00 a  m" src="https://github.com/user-attachments/assets/6c149f7b-75c7-4cb4-9e3b-5636d28be0ec" />

<img width="683" height="318" alt="image" src="https://github.com/user-attachments/assets/5888e214-ec7b-4c25-888b-88f04c1d365e" />

<head></head>

**Insights de comportamiento de ventas** 

<head></head>

•Los tres canales principales (social, organic y paid_search) concentran la mayoría del presupuesto con cifras muy cercanas entre sí, oscilando entre los 863,000 y 918,000.

<head></head>

•La barra gris correspondiente a Sin dato se mantiene en un nivel bajo y controlado de $177,179.10. Cumple con equilibrar los totales globales de los KPIs de marketing sin distorsionar la lectura de los canales principales.

<head></head>

**Analisis de funnels**

<head></head>

El analisis de funnel se hizo en SQL siguiendo la lógica del Count distinct

<head></head>

query_totals = '''

SELECT 
    COUNT(DISTINCT CASE WHEN nombre_evento = 'first_visit' THEN id_usuario END) AS total_first_visit,
    COUNT(DISTINCT CASE WHEN nombre_evento = 'add_to_cart' THEN id_usuario END) AS total_add_to_cart,
    COUNT(DISTINCT CASE WHEN nombre_evento = 'add_payment_info' THEN id_usuario END) AS total_add_payment_info,
    COUNT(DISTINCT CASE WHEN nombre_evento = 'purchase' THEN id_usuario END) AS total_purchase
FROM events;
'''

totals = pd.read_sql(query_totals, con=engine)
totals

<head></head>

Se calculo la tasa de conversion del funnel

query_conversion = '''
WITH totals AS (
    SELECT 
        COUNT(DISTINCT CASE WHEN nombre_evento = 'first_visit' THEN id_usuario END) AS total_first_visit,
        COUNT(DISTINCT CASE WHEN nombre_evento = 'add_to_cart' THEN id_usuario END) AS total_add_to_cart,
        COUNT(DISTINCT CASE WHEN nombre_evento = 'add_payment_info' THEN id_usuario END) AS total_add_payment_info,
        COUNT(DISTINCT CASE WHEN nombre_evento = 'purchase' THEN id_usuario END) AS total_purchase
    FROM events
)
SELECT 
    ROUND(100.0 * total_add_to_cart / NULLIF(total_first_visit, 0), 2) AS conv_first_visit_to_cart,
    ROUND(100.0 * total_add_payment_info / NULLIF(total_add_to_cart, 0), 2) AS conv_cart_to_payment,
    ROUND(100.0 * total_purchase / NULLIF(total_add_payment_info, 0), 2) AS conv_payment_to_purchase,
    ROUND(100.0 * total_purchase / NULLIF(total_first_visit, 0), 2) AS conv_total
FROM totals;
'''

conversion = pd.read_sql(query_conversion, con=engine)
conversion

<head></head>

<img width="583" height="68" alt="image" src="https://github.com/user-attachments/assets/92a34a1c-25c1-4940-8255-5362fd4cd1db"/> 

  <head></head>

  <img width="667" height="74" alt="image" src="https://github.com/user-attachments/assets/5b0013a2-8ed8-4b45-9158-7a1fc22506f8" />

  <head></head>

**Hallazgos del funnel**

-- 97.92% de quienes visitaron pasaron a agregar algo al carrito; muy buena conversión.

-- 81.87% de los que agregaron al carrito avanzaron a ingresar info de pago; hay una caída más notoria.

-- 99.84% de los que ingresaron info de pago terminaron comprando; casi nadie se arrepiente en el último paso.

-- 80.04% es la conversión total del funnel completo (de first_visit a purchase).

La mayor pérdida de usuarios ocurre entre "agregar al carrito" y "agregar info de pago" (se pierde casi un 18% ahí), mientras que una vez que alguien llega a poner su info de pago, casi seguro compra.


  <head></head>

  **Retención por cohortes**

 Este análisis se realizo en SQL, para validar si los usuarios regresan y cuantos de ellos pasan a la siguiente etapa

    
<img width="859" height="182" alt="image" src="https://github.com/user-attachments/assets/87a62971-0c20-4a45-b619-24200b7cb350" />

**Conclusion en analisis de cohortes**


La retención en la semana 1 es 0% para todas las cohortes porque los datos de actividad solo comienzan a partir del día 7 después del registro (no hay actividad temprana registrada).

La retención se estabiliza entre 40-43% en las semanas 2 y 3,

w3 sea a veces mayor que w2 (por ejemplo, en 2025-03: 41.38% vs 43.09%) ya no es un bug, es simplemente parte del patrón real que detectamos.
 

**Test Estadístico**

Hipotesis:

  <head></head>


Ho= no hay diferencia real en la tasa de conversión entre control y tratamiento.
  <head></head>

H1= sí existe una diferencia real en la tasa de conversión entre ambos grupos.

**Prueba Z para dos proporciones**

Se elige esta prueba porque la métrica convirtio es binaria, lo cual encaja perfectamente con distribuciones binomiales. El dataset cuenta con 10,000 filas, el tamaño muestral es grande.

 <head></head>

Estadístico Z: -0.8133
P-value: 0.4161

Resultado: No rechazamos H₀. No hay evidencia estadística suficiente para afirmar que la nueva UI generó un cambio significativo.

***Decision***


Con un **p-value de 0.4161,** muy por encima de 0.05, la conclusión: **no hay evidencia suficiente para decir que la nueva UI cambió la conversión.**

**Decisión estadística: No rechazamos la Hipótesis Nula (H0).**

La evidencia no permite asegurar que la nueva interfaz de checkout haya mejorado ni empeorado el rendimiento comercial. El cambio se considera neutral.

***Insights de negocio***

•El test de hipótesis prueba Z para dos proporciones arrojó un **p-value de 0.4161,** lo que confirma que la variación observada está dentro del margen de error esperable por el azar

•Los intervalos de confianza al 95% muestran un solapamiento total entre ambas variantes, validando visualmente que no existe una separación de rendimiento clara entre el control y el tratamiento.

***Recomendaciones de negocio***

No se recomienda el despliegue masivo de la nueva UI basándose en la conversión global, ya que el cambio es estadísticamente neutral.


**Visualizaciones gráficas Tableau Public**

https://public.tableau.com/views/ProyectoFinalDiagnsticoestratgicointegralparaRappiPlus/DiagnsticoestratgicointegralparaRappiPlus?:language=es-ES&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link

