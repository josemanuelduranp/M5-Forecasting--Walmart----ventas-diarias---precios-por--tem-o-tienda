Elasticidad Precio–Demanda y Optimización de Margen (M5 Forecasting / Walmart)

Resumen del proyecto

Este proyecto desarrolla un flujo práctico de estrategia de precios usando el dataset M5 Forecasting (Walmart): estimamos elasticidad precio–demanda y, con esas elasticidades, simulamos escenarios de precio para recomendar ajustes que maximicen profit por categoría y mercado.

Idea central: la granularidad importa. A nivel SKU–tienda la elasticidad suele ser ruidosa e inestable; al agregar y hacer pooling se obtienen resultados más accionables.

Problema de negocio
Un equipo de pricing quiere responder:

1.-¿Qué tan sensible es la demanda a cambios de precio? (Elasticidad)

2.-¿Conviene subir o bajar precios para maximizar profit?

3.-¿La recomendación cambia por mercado y categoría?

4.-¿El pricing puede “arreglar” un problema de rentabilidad o hay temas estructurales (costos/mix)?

Entregables del proyecto:

-Elasticidades con inferencia estadística
-Simulación de unidades, revenue y profit por cambios de precio
-Recomendación accionable “subir / bajar / mantener” por segmento
### Dataset
Fuente: Kaggle_ M5 Forecasting dataset (Walmart). Archivos principales:

-sales_train_* (ventas diarias por item/tienda)
-sell_prices.csv (precios semanales por item/tienda/semana)
-calendar.csv (mapea d_# a fecha y wm_yr_wk)
Enfoque multi-mercado: los stores/states se mapean a tres países (US/MX/ES) para simular un contexto de estrategia de precios internacional sin perder realismo.

Estructura del repositorio / notebooks
01_data_loading_and_cleaning.ipynb

-Carga, validación de tipos, joins con calendario y precios, y creación de tablas limpias cuidando performance.

02_eda_sales_and_prices.ipynb

-EDA de ventas y precios; comparación por país; validación de cobertura de precios.

03_price_elasticity_model.ipynb

-Modelo log-log (OLS), filtros de calidad, agregación semanal, pooling por segmento y errores robustos.

04_margin_optimization.ipynb

-Costos unitarios fijos por dept, baseline de profit, simulación de escenarios y tabla ejecutiva de recomendaciones.

Metodología
Preparación de datos (Notebook 01)
Se construye una tabla modelable con:

-item_id, store_id, dept_id, country
-date, wm_yr_wk
-units_sold
-sell_price
Nota de performance: evitamos “melt” masivo (muy pesado en laptop) cuando no es necesario, para prevenir crashes del kernel.

Exploración (Notebook 02)
Validamos:

-cobertura de precios por semanas y mercados
-distribución de precios por país
-consistencia de joins sales ↔ calendar ↔ prices
-señales para justificar agregación semanal
Resultado: el EDA motiva pasar de diario a semanal para estabilizar el modelado.

Elasticidad (Notebook 03)
Se intenta a nivel SKU–tienda y se observa inestabilidad; se pivotea a modelos agregados.

Aprendizaje clave: a nivel SKU–tienda suele haber:

-ruido alto
-poca variación de precio
-efectos no observados (promos/estacionalidad)
Optimización de margen (Notebook 04)
Con elasticidades estimadas:

-simulamos cambios de precio
-calculamos unidades, revenue, profit esperados
-elegimos el precio que maximiza profit por segmento
Modelos matemáticos
A) Elasticidad log-log (OLS)

Modelo estándar de elasticidad constante:

log(Q) = β₀ + β₁ log(P) + β₂ t + ε

-Q: unidades vendidas
-P: precio
-𝑡:tendencia temporal (índice semanal)
-β1 : elasticidad precio–demanda

    -Ejemplo: 𝛽1= −2 → subir 1% el precio reduce ~2% la demanda.
Se usan errores estándar robustos HC3 para heterocedasticidad.

B) Agregación semanal

-Unidades: suma semanal
-Precio: promedio semanal
C) Filtros de calidad (semanal)

-Para asegurar señal suficiente:
    -MIN_WEEKS = 26
    -MIN_UNIQUE_PRICES_W = 3
D) Modelo pooled (categoría × país)

Pooling a nivel dept_id × country × week.
Resultado pooled del run:

-Elasticidad pooled: −4.325
-p-value: ~8.9e−55
-R²: 0.268
Interpretación: en agregado, +1% precio → ~−4.3% unidades (con evidencia fuerte).

E) Simulación de demanda con elasticidad

`Q_new = Q_base × (P_new / P_base)^ε`
F) Profit

`Profit = Q × (P − C)`
con 𝐶 costo unitario fijo por dept en esta versión.

Resultados clave (del run)
Elasticidades por segmento (dept × país)

Ejemplos:

-FOODS_3 (US): elasticidad ≈ −8.50 (muy sensible al precio)
-Otros segmentos muestran elasticidades débiles o inestables (común en retail).
Recomendaciones óptimas (costos fijos por dept)

Costos unitarios usados:

-FOODS_3: 1.80
-HOBBIES_1: 6.50
Hallazgos:

-FOODS_3 (US): bajar precio 10% maximiza profit (gran mejora vs baseline).
-FOODS_3 (MX/ES): subir precio 10% mejora profit por captura de margen.
-HOBBIES_1: profit sigue negativo en todos los escenarios; subir precio reduce pérdidas pero no vuelve rentable.
Conclusión de negocio: FOODS tiene oportunidades claras de pricing diferenciado por mercado; HOBBIES sugiere problema estructural (costos/mix/demanda), no resoluble solo con precio.

Limitaciones y supuestos
-Elasticidad no causal (datos observacionales; promos/competencia no modeladas por completo).

-Supuesto de elasticidad constante.

-Costos fijos por dept (simplificación).

-No se modela explícitamente:

-promociones como driver separado
-competencia
-sustitución, inventario, marketing
-En segmentos con poca variación de precio, la elasticidad puede ser inestable.
Limitaciones y supuestos
-Elasticidad no causal (datos observacionales; promos/competencia no modeladas por completo).
-Supuesto de elasticidad constante.
-Costos fijos por dept (simplificación).
-No se modela explícitamente:
    -promociones como driver separado
    -competencia
    -sustitución, inventario, marketing
-En segmentos con poca variación de precio, la elasticidad puede ser inestable.
Tech stack
-Python 3.x (Anaconda)

-Jupyter Notebook

Librerías:

-pandas, numpy -statsmodels (OLS + HC3) -matplotlib

(Opcional futuro):

-scikit-learn (Ridge/Lasso)

Cómo correr el proyecto
Descarga el dataset M5 de Kaggle y colócalo en data/.

Crea/activa tu entorno (Anaconda recomendado).

Ejecuta los notebooks en orden:

-01 → 02 → 03 → 04
Estructura sugerida:

project/ data/ calendar.csv sell_prices.csv sales_train_validation.csv notebooks/ 01_data_loading_and_cleaning.ipynb 02_eda_sales_and_prices.ipynb 03_price_elasticity_model.ipynb 04_margin_optimization.ipynb

¿Qué haría después?...
1.-Elasticidad con enfoque causal / cuasi-causal

-Agregar variables de promos/holiday y controles; explorar métodos tipo diff-in-diff o instrumentos.
2.- Modelos regularizados / jerárquicos

-Ridge/Lasso o partial pooling para estabilizar elasticidad a nivel SKU.
3.-Elasticidad no constante

-Modelos por tramos (price bands) o splines.
4.- Costos más realistas

-Costos por item, logística, constraints de margen; análisis de sensibilidad de costos.
5.-Soporte a decisiones

-Dashboard ligero y un playbook de pricing por categoría/mercado.
