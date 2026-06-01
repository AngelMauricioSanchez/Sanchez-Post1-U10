# Laboratorio: Midiendo el Efecto de la Caché

Este repositorio contiene los resultados y el análisis de la actividad sobre la jerarquía de memoria y su impacto en el rendimiento de los programas.

## Tabla de Contenidos
1. [Configuración y Detección de Caché](#paso-1)
2. [Benchmark de Acceso Secuencial](#paso-2)
3. [Comparativa: Acceso Secuencial vs Aleatorio](#paso-3)
4. [Conclusiones](#conclusiones)

---

<a name="paso-1"></a>
## 1. Configuración y Detección de Caché
Se consultó la topología de caché mediante el sistema de archivos `/sys/`. 

* **Nota:** En el entorno virtualizado (JSLinux), el acceso a los registros físicos de caché retorna 'N/A'. Para el análisis, se consideró una jerarquía estándar x86_64: L1 (32KB), L2 (256KB) y L3 (8MB).

---

<a name="paso-2"></a>
## 2. Benchmark de Acceso Secuencial
Se implementó un programa en C (`cache_bench.c`) que mide la latencia de lectura secuencial en arrays de diversos tamaños.

### Análisis de Transiciones:
La ejecución revela el comportamiento de la jerarquía de memoria mediante saltos de latencia:
* **Punto de inflexión L1 (32KB):** Al superar este umbral, los datos ya no residen completamente en L1, obligando al procesador a consultar la caché L2, lo que incrementa el tiempo de respuesta (ns/byte).
* **Punto de inflexión L2 (256KB):** Indica la saturación de la caché L2; el sistema empieza a realizar *cache misses* frecuentes que se sirven desde L3.
* **Punto de inflexión L3 (8MB) y RAM:** Al exceder la capacidad de L3, el procesador debe acceder a la memoria RAM principal, lo que representa el salto de latencia más drástico debido a la mayor distancia física y latencia de la memoria externa.

---

<a name="paso-3"></a>
## 3. Comparativa: Acceso Secuencial vs Aleatorio
Se comparó el acceso secuencial con una implementación de acceso aleatorio utilizando el algoritmo de mezcla de Fisher-Yates.

* **Resultados:** El acceso aleatorio presenta una latencia significativamente mayor que el secuencial.
* **Justificación:**
    1. **Cache Misses:** El acceso aleatorio impide el *prefetching* del procesador.
    2. **TLB Misses:** Al superar el tamaño del TLB (~1-2MB), el procesador debe realizar búsquedas adicionales en la tabla de páginas, degradando el rendimiento drásticamente.

---

<a name="conclusiones"></a>
## Conclusiones
La jerarquía de memoria es vital para el rendimiento. Los experimentos demuestran que, sin **localidad espacial**, el procesador queda limitado por la velocidad de la memoria RAM principal, perdiendo los beneficios de las memorias caché (L1/L2/L3) integradas en el silicio.
