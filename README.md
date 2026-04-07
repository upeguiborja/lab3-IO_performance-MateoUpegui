# Laboratorio 3: Acceso a Disco y Costo de E/S (I/O)
**Autor:** Mateo Upegui Borja  
**Curso:** Diseño de Bases de Datos (UdeA)

## 1. Especificaciones del equipo

| Parámetro                        | Valor de Referencia   |
| :------------------------------- | :-------------------- |
| **Sistema Operativo**            | macOS 15.4 (24E5263i) |
| **Modelo de CPU**                | Apple M1 Max          |
| **Núcleos de CPU**               | 10 Cores (8P + 2E)    |
| **RAM Total**                    | 32 GB                 |
| **Tecnología de Almacenamiento** | Apple NVMe SSD        |

## 2. Resultados del experimento

Las siguientes gráficas fueron generadas mediante la ejecución del notebook `disk_io_lab_guided.ipynb` utilizando un archivo de prueba de 8 GiB y el flag `F_NOCACHE` para mitigar el sesgo del caché sistema operativo.

### Comparación de Throughput (MiB/s)
![Throughput](images/fig_throughput.png)

### Secuencial: Teoría vs. Práctica
![Tiempo Secuencial](images/fig_tiempo_teoria_vs_practica_secuencial.png)

### Aleatorio: Teoría vs. Práctica
![Tiempo Aleatorio](images/fig_tiempo_teoria_vs_practica_aleatorio.png)

### Speedup (Ventaja Secuencial)
![Speedup](images/fig_speedup.png)

## 3. Análisis y conclusiones

### Preguntas de Análisis Científico

**1. Diferencial de Desempeño**
El patrón de acceso secuencial resultó ser significativamente más eficiente que el aleatorio en todos los tamaños de bloque evaluados. La mayor diferencia que observamos se dió con bloques de **4 KB**, donde el acceso secuencial alcanzó **194.91 MiB/s** frente a los **35.44 MiB/s** del aleatorio, lo que representa un speedup de aproximadamente **5.5x**.  

Esta diferencia confirma la hipótesis de que el costo de "búsqueda" lógica y el overhead del controlador dominan el tiempo de ejecución cuando el volumen de datos por operación es pequeño, incluso en ausencia de componentes mecánicos.

**2. Efecto del Tamaño de Bloque**
El incremento en el tamaño de bloque de lectura en cierto sentido mitiga el costo de latencia ($\alpha$). En el acceso aleatorio, el throughput aumentó al pasar de **35.44 MiB/s** (4 KB) a **1242.91 MiB/s** (256 KB).  

Al transferir más datos por cada evento de acceso, el costo fijo de localizar el bloque se vuelve despreciable frente al tiempo de transferencia de los datos ($\beta \cdot DataSize$), permitiendo que el hardware se acerque a su ancho de banda teórico máximo.

**3. Correlación con la Teoría**
El hardware se alejó más del modelo teórico en el patrón de acceso secuencial a través de los diferentes tamaños de bloque. Mientras que el modelo $T = \alpha M + \beta \cdot DataSize$ predice un comportamiento casi constante para el patrón secuencial ($M=1$), en la práctica observamos que el rendimiento real se multiplicó por **6.5x** al aumentar el bloque de 4 KB a 256 KB.  

Factores físicos como el las optmizaciones del hardware NVMe y el prefetching a nivel de firmware explican que el dispositivo real sea mucho más capaz de optimizar transferencias grandes de lo que sugiere el modelo lineal simple.

**4. Costo de Acceso en SSD**
Incluso en un SSD, el acceso aleatorio es más costoso que el acceso secuencia, esto se debe a que el un patrón de acceso aleatorio impide que el controlador prediga y pre-cargue los siguientes bloques en la memoria interna del dispositivo. En nuestras pruebas, esto se tradujo en una penalización de tiempo de **231.18 s** para leer el dataset de forma aleatoria (4 KB) frente a solo **42.03 s** de forma secuencial.

**5. Implicaciones en Sistemas**
Para el diseño de un Motor de Base de Datos, los resultados sugieren una estrategia de **I/O secuencial**. Idealmente se implementarían estructuras de datos que garantizen las escrituras secuenciales e indices que puedan asegurar que los registros relacionados residan en bloques contiguos.

Al maximizar la localidad de datos, el sistema puede operar en el rango de los **1.2 GiB/s** medidos en bloques grandes, evitando caer en el cuello de botella de las decenas de MiB/s asociado a las lecturas aleatorias pequeñas.
