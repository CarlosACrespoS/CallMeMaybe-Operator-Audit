# 📞 Identificación de Operadores Ineficaces — CallMeMaybe

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-1.5%2B-150458.svg)](https://pandas.pydata.org/)
[![SciPy](https://img.shields.io/badge/SciPy-Statistical--Testing-8CAAE6.svg)](https://scipy.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 📌 Resumen Ejecutivo del Proyecto

**CallMeMaybe** es un servicio de telefonía e infraestructura de comunicación enfocado en proporcionar inteligencia analítica a centros de atención al cliente (call centers). El presente proyecto tiene como **objetivo central de negocio** la identificación sistemática y estadísticamente rigurosa de **operadores ineficaces** cuyo rendimiento impacta negativamente la experiencia del cliente y la rentabilidad operativa.

Mediante una metodología basada en el desarrollo de *pipelines* de datos, análisis exploratorio avanzado, ingeniería de métricas de calidad y pruebas de hipótesis estadísticas con calibración de significancia, se construyó un marco analítico capaz de segmentar el desempeño operativo sin incurrir en sesgos por falsos positivos.
El sistema opera sobre dos fuentes primarias interconectadas por la variable `user_id`:

### 1. `telecom_dataset_new.csv` (Registros de Llamadas)
* **`user_id`**: Identificador único de la cuenta/cliente corporativo.
* **`date`**: Timestamp y fecha de registro del bloque de llamadas.
* **`direction`**: Dirección del flujo telefónico (`in` = Entrante, `out` = Saliente).
* **`internal`**: Indicador booleano de llamada interna entre asesores (`True`/`False`).
* **`operator_id`**: Identificador único del operador/asesor asignado.
* **`is_missed_call`**: Estado de atención (`True` = Llamada perdida / no contestada, `False` = Atendida).
* **`calls_count`**: Cantidad total de llamadas agrupadas en dicho bloque.
* **`call_duration`**: Tiempo neto de conversación telefónica (en segundos).
* **`total_call_duration`**: Duración total de la línea ocupada (incluye tiempo en cola/espera).

### 2. `telecom_clients.csv` (Catálogo de Clientes)
* **`user_id`**: Identificador único del cliente corporativo.
* **`tariff_plan`**: Plan tarifario contratado (`A`, `B`, `C`).
* **`date_start`**: Fecha de alta del cliente en el servicio.

---

## ⚙️ Fases Analíticas y Hallazgos Principales

### 🔍 Fase 1: Carga, Auditoría y Preparación de Datos
* **Auditoría de Calidad:** Se evaluó el volumen original de **53,902 registros** en llamadas y 732 clientes.
* **Estrategia de Tratamiento de Nulos:**
  * La variable `operator_id` presentaba un **15.16% de valores faltantes** (8,172 registros). La validación demostró que el **97.55% de estos nulos correspondían a llamadas entrantes (`in`) no asignadas**. Se decidió de forma estratégica **conservar estos registros**, ya que eliminarlos destruiría la métrica base del volumen de llamadas perdidas e inflaría artificialmente el desempeño del call center.
  * La variable categórica `internal` (0.22% nulos) fue imputada con éxito mediante la moda estadística.
* **Ingeniería de Características:** Se calculó la métrica crítica de espera:
  $$\text{wait\_time} = \text{total\_call\_duration} - \text{call\_duration}$$

### 📈 Fase 2: Análisis Exploratorio de Datos (EDA)
* **Tendencias Temporales:** Evaluación de la estabilidad cronológica del tráfico telefónico por tipo de red y dirección.
* **Depuración de Outliers:** Aplicación de gráficos de caja en escala logarítmica simétrica (`symlog`) y filtrado estricto mediante el **Percentil 99.5%** para eliminar errores informáticos sin afectar a los operadores con alta carga operativa.
* **Perfilamiento Individual:** Agregación por `operator_id` con fusiones de tipo *outer join* para consolidar volúmenes, tasa de llamadas perdidas (`missed_rate`) y tiempos promedio de espera (`avg_wait`).

### 🎯 Fase 3: Identificación, Calibración y Segmentación de Operadores
Para evitar clasificaciones arbitrarias, la ineficacia operativa se evaluó con base en dos ejes críticos de negocio:

1. **Eje Entrante (Inbound):**
   * **Métrica:** Tasa de llamadas perdidas (`missed_rate`) y Tiempo promedio de espera en cola (`avg_wait`).
   * **Umbral de Ineficacia:** Asesores en el cuartil superior de llamadas no contestadas o con esperas prolongadas antes de contestar.
2. **Eje Saliente (Outbound):**
   * **Métrica:** Volumen de llamadas salientes realizadas por día/periodo.
   * **Umbral de Ineficacia:** Operadores con baja efectividad de marcación y tiempos de conversación deficientes respecto a sus pares.

### 🧪 Fase 4: Pruebas de Hipótesis Estadísticas
Se diseñaron dos pruebas de hipótesis formales utilizando `scipy.stats` para validar si las diferencias observadas entre los grupos de operadores eran estadísticamente significativas ($ \alpha = 0.05 $):

* **Hipótesis 1 (Tiempo de Espera Entrante):**
  * $H_0$: No existe diferencia significativa en el tiempo medio de espera entre operadores eficaces e ineficaces.
  * $H_1$: El tiempo medio de espera de los operadores catalogados como ineficaces es significativamente mayor.
* **Hipótesis 2 (Duración de Llamadas Salientes):**
  * $H_0$: No existe diferencia en el volumen/duración media de llamadas salientes entre ambos grupos.
  * $H_1$: Los operadores ineficaces registran una duración significativamente menor en llamadas salientes.






