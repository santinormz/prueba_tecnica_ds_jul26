# Process & Methodology Document (PROCESS.md)

## 1. Resumen Ejecutivo
Este documento detalla la arquitectura de la solución, las decisiones metodológicas y el stack tecnológico utilizado para resolver la prueba técnica de Data Science. 
El enfoque principal fue transicionar de un análisis descriptivo a una solución de **Machine Learning** para resolver un problema crítico de negocio: la **falta de especificidad estratégica** en la gestión logística y promocional de las tiendas.

---

## 2. Stack Tecnológico y Herramientas

* **Lenguaje:** Python 3.13
* **Procesamiento de Datos:** `pandas`, `numpy`
* **Machine Learning:** `scikit-learn` (Regresión Lineal, K-Means, PCA, StandardScaler)
* **Visualización:** `matplotlib`, `seaborn` (estilo corporativo para presentaciones de negocio)
* **Herramientas de IA (AI Chatbot):** `Gemini 3.1 Pro`

---

## 3. Arquitectura del Proceso

El flujo de trabajo se dividió en 4 fases principales para garantizar la reproducibilidad y el rigor estadístico:

### Fase 1: Data Wrangling & Quality Assurance
Se detectaron valores nulos en variables críticas (`amount_cash`, `units_sold`) derivados (posiblemente) de fallas de conectividad en puntos de venta. Se implementó una estrategia de imputación en dos niveles para preservar la integridad de las distribuciones:
1.  **Imputación Algebraica:** Recuperación exacta de valores mediante el balance de ecuaciones transaccionales (Ej. `amount_cash` = `amount_total` - `amount_card`).
2.  **Imputación Predictiva (Machine Learning):** En lugar de usar la media, se entrenaron **6 modelos de Regresión Lineal** (uno por cada categoría de producto) para predecir `units_sold` basándonos en `amount_total`. Se logró un $R^2$ consistente de ~0.78, lo que garantizó la preservación de la varianza natural del volumen de ventas.
3.  **Data Mart:** Se construyó una tabla base analítica (ABT) consolidando `transactions`, `stores` y `calendar` mediante *Inner Joins*.

### Fase 2: Exploratory Data Analysis (EDA) & Time Series
El análisis multivariado y de series temporales (agrupación semanal) reveló *insights* que formaron la hipótesis del modelo:
* **Estacionalidad Extrema:** Picos masivos durante el *Buen Fin* (impulsado por Electrónica) y *Navidad*.
* **Logística Reactiva:** Fuerte correlación ($\rho = 0.81$) entre `units_sold` y `replenishment_signal`, lo que indica un sistema que reacciona a la demanda en lugar de anticiparse (riesgo de *Out-of-Stock*).
* **Ineficiencia Promocional:** El volumen global no mostró una diferencia estadística significativa entre los días con o sin promoción.

Estos prompt fue usado para generar el codigo de esta etapa, compartiendo las bases de datos proporcionadas, el resultado fue revisado y ajustado posteriormente para mayor claridad.

> quiero que se limpien los datos (usando Python) recuperando amount_cash, avg_ticket, cash_transactions y ademas recuperar units_sold usando una regresion lineal y las variables amount_total y por cada category (para estas regesiones dame el R ^2 de cada regresion), por ultimo agrupa las 3 bases de datos usando inner join, para posteriormente comenzar el analisis con la base limpia y con toda la informacion, recuerda explicar bien el proceso para agregarlo a la documentacion por favor.

> ahora que la base esta limpia y con la información de las tres tablas, me gustaria estructurar el problema de negocio sobre Segmentación de Patrones de Consumo (Clustering) usando todas las variables dadas y encontrar areas de mejora en segmentos con pocas ventas o el impacto de la variable replenishment_signal en el performance de las ventas, pero comenzando por un análisis exploratorio de datos, para obtener Insights, encontar areas de mejora y plantear el problema de negocio, para posteriormente plantear posibles soluciones, recuerda explicar bien el proceso para agregarlo a la documentacion por favor.

> agrega un análisis sobre las tendencias de venta diaria (series de tiempo) por region, category y store_format para ver tendecias en la venta, agrega gráficos claros y atractivos usando seaborn sobre este analisis y el analisis anterior, recuerda explicar bien el proceso para agregarlo a la documentación por favor. 


### Fase 3: Feature Engineering
Para agrupar tiendas por su **comportamiento latente** (y no solo por su formato o región), se agregaron los datos a nivel sucursal y se crearon variables sintéticas de negocio:
* `promo_lift`: Índice de elasticidad promocional (Ventas con Promo / Ventas sin Promo).
* `peak_dependence`: Porcentaje de ingresos dependientes exclusivamente del Buen Fin y Navidad.
* `avg_replenishment`: Presión promedio sobre la cadena de suministro local.

### Fase 4: Machine Learning (K-Means & PCA)
* **Clustering:** Se utilizó un escalado estándar (`StandardScaler`) y el algoritmo **K-Means**. Aunque el *Silhouette Score* sugería $K=2$, se forzó el modelo a **$K=4$** para lograr una granularidad táctica útil para el negocio.
* **Interpretabilidad (PCA):** Para visualizar el hiperplano multidimensional y evitar el efecto de "caja negra", se aplicó un Análisis de Componentes Principales (PCA) a 2 componentes. 
    * Al extraer los autovectores (`pca.components_`), se demostró matemáticamente que el **Componente 1 (Eje X)** separa a las tiendas por su *Volumen y Riesgo Logístico* (`avg_replenishment`, `total_units`), mientras que el **Componente 2 (Eje Y)** las separa por su *Rentabilidad y Sensibilidad Promocional* (`avg_ticket`, `promo_lift`).

Esto prompt fue usado para generar el codigo de la etapa 3 y 4.

> continuemos con el Feature Engeneering y el modelo de K-means, recuerda explicar todo y hacer una grafico de los clústeres finales por favor.
---

## 4. Conclusión y Casos de Uso del Modelo
La segmentación descubrió 4 perfiles de tiendas, permitiendo pasar de una estrategia "One-Size-Fits-All" a iniciativas hechas a medida:
1.  **Clúster 2 (Gigantes):** Tiendas con volumen crítico y alto estrés logístico. Requieren transición urgente a logística predictiva (*Push*) previo a picos estacionales.
2.  **Clúster 1 (Premium):** Tiendas con ticket alto y elasticidad promocional positiva. El presupuesto de marketing debe concentrarse aquí para maximizar ROI.
3.  **Clúster 0 (Medio):** Foco en volumen continuo y estrategia EDLP (Every Day Low Prices).
4.  **Clúster 3 (Rezago):** Riesgo de capital. Requieren reestructuración de piso de venta, eliminación de categorías de baja rotación en favor de aquellas con mayor rotacion o estrategias de Marketing para incrementar las ventas.
