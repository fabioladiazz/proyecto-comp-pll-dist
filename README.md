# Optimización del Flujo de Datos mediante Procesamiento con Polars

### **Integrantes:** Fabiola Díaz, Valerie Espinoza y Alejandro Ocampo

**Computación Paralela y Distribuida**

Este proyecto implementa un benchmark reproducible para comparar el rendimiento entre:

* **Pandas** → motor basado en NumPy, ejecución secuencial
* **Polars** → motor columnar en Rust, paralelización nativa

El objetivo es evaluar diferencias en:

* Tiempo de ejecución
* Uso de memoria
* Ventajas y limitaciones de cada librería

Incluye 10 operaciones representativas de análisis de datos reales: lectura, filtrado, agregaciones, transformaciones, correlaciones y cálculos estadísticos.

---

## Dataset

Se utiliza el dataset público de taxis de Nueva York:

**NYC Yellow Taxi Trips – January 2015**
Archivo: `yellow_tripdata_2015-01.csv`
Tamaño: ~1.3 GB

Contiene columnas como:

* `trip_distance`
* `fare_amount`
* `total_amount`
* `passenger_count`
* `VendorID`
* `RateCodeID`
* `tpep_pickup_datetime`, `tpep_dropoff_datetime`
* Coordenadas de pickup/dropoff

Este dataset es ideal para pruebas de benchmarking por su tamaño y variedad de tipos de datos.

---

## Configuración del entorno

### Instalar dependencias

```bash
pip install pandas polars psutil matplotlib seaborn jupyterlab
```

Dependencias clave:

* **pandas**
* **polars**
* **psutil** → medición de memoria
* **matplotlib / seaborn** → gráficos comparativos

---

## Ejecución del Benchmark

El notebook ejecuta de forma automatizada:

1. Lectura del dataset con Pandas y Polars
2. Ejecución de las 10 operaciones
3. Medición de:
   * Tiempo de ejecución (`time.perf_counter`)
   * Delta de memoria (`psutil`)
4. Exportación de resultados a CSV
5. Visualización con gráficos en Seaborn

---

##  Operaciones Medidas

A continuación, las **10 operaciones** ejecutadas con Pandas y Polars:

| Nº | Operación                                    | Pandas | Polars | Descripción                        |
| -- | -------------------------------------------- | ------ | ------ | ---------------------------------- |
| 1  | Filtro de viajes largos                      | ✔      | ✔      | `trip_distance > 10`               |
| 2  | Mean de `total_amount` por `passenger_count` | ✔      | ✔      | `groupby` + agregación             |
| 3  | Mean de `fare_amount` por hora del día       | ✔      | ✔      | Transformación + agrupación        |
| 4  | Top 1000 viajes según `tip_amount`           | ✔      | ✔      | `nlargest` / `sort desc`           |
| 5  | Conteo por tipo de pago                      | ✔      | ✔      | `value_counts` / `group_by`        |
| 6  | Promedio de distancia por fecha              | ✔      | ✔      | Procesamiento de fechas            |
| 7  | Correlación entre distancia y total_amount   | ✔      | ✔      | Estadística                        |
| 8  | Percentil 95 de `total_amount`               | ✔      | ✔      | `quantile`                         |
| 9  | Filtro geográfico en Manhattan               | ✔      | ✔      | Coordenadas dentro de bounding box |
| 10 | Agregación por VendorID + RateCodeID         | ✔      | ✔      | Groupby multicolumna               |

---

## Visualización de Resultados

Se generaron dos gráficos comparativos:

### **Comparación de Tiempo de Ejecución**

* Rojo → **Pandas**
* Azul → **Polars**
* Barras por operación

### **Comparación de Delta de Memoria**

* Misma codificación de colores
* Permite ver eficiencia del motor columnar


---

## Conclusiones
Los resultados del benchmark muestran de manera clara las ventajas estructurales de **Polars** frente a **Pandas**, especialmente cuando se trabaja con datasets grandes. La diferencia más notoria se observa en la **lectura del archivo**, donde Pandas requiere **31.13 segundos**, mientras que Polars completa la misma operación en apenas **4.24 segundos**, evidenciando un rendimiento casi **8 veces más rápido** gracias a su motor columnar implementado en Rust y su paralelización interna por defecto. Esta tendencia se refleja en la mayoría de las operaciones analíticas: cálculos como *group-by*, *quantiles*, correlaciones y transformaciones temporales muestran ganancias constantes para Polars, con tiempos significativamente menores (por ejemplo, `op2_mean_total_by_passengers` pasa de **0.19 s** en Pandas a **0.05 s** en Polars, y `op7_corr_distance_total` de **0.13 s** a **0.04 s**). Aunque no todas las tareas son más rápidas en Polars —como `op4_top_tips`, donde Pandas obtiene un mejor desempeño— la mayoría de las operaciones demuestran una aceleración significativa.

En cuanto al **uso de memoria**, las diferencias también son evidentes. Durante la lectura del dataset, Pandas incrementa su consumo en **+390 MB**, mientras que Polars reporta un delta **negativo (-302 MB)**, lo cual indica liberación o reutilización más eficiente de buffers internos. En múltiples operaciones, Polars mantiene deltas de memoria menores que Pandas o incluso negativos, como en `op3_mean_fare_by_hour` y `op4_top_tips`, reflejando la eficiencia del formato columnar Apache Arrow y del sistema de ejecución optimizado. Sin embargo, algunas operaciones muestran deltas positivos elevados en Polars (`op9_manhattan_bbox_count`: **+2000 MB**), lo que indica que ciertas transformaciones pueden implicar materialización de columnas o evaluaciones no diferidas, recordando que la estrategia de memoria depende del patrón de acceso.

En conclusión, los resultados permiten concluir que Polars es considerablemente más eficiente tanto en tiempo como en memoria para cargas de trabajo analíticas en datasets medianos y grandes, ofreciendo mejor escalabilidad y aprovechamiento del hardware. Por su parte, Pandas sigue siendo una excelente opción para análisis exploratorio, prototipado rápido y datasets pequeños o medianos, donde la sencillez de su ecosistema y su amplia compatibilidad son prioridades. Polars se recomienda cuando el volumen de datos supera la memoria disponible, cuando se busca paralelización transparente o cuando se ejecutan pipelines complejos donde la optimización interna puede reducir costos de cómputo.
