# **Analizador de Espectro de Audio FHT**

**Asignatura:** TA134 - Taller de Sistemas Embebidos


**Microcontrolador:** STM32F103RB (NUCLEO-F103RB)


**Fecha:** 13/11/26 

---
**Integrantes:** 

- Sorroza Fausto Esteban
- Cingolani Camilo 
- Maizares Bruno

---

## **1. Selección del proyecto a implementar**

### **1.1 Objetivo del proyecto y resultados esperados**

El objetivo del Trabajo Final es diseñar e implementar un analizador de espectro de audio en tiempo real que capture señales analógicas de audio desde un conector Jack de 3,5 mm, las separe en sus componentes de frecuencia mediante procesamiento digital utilizando aritmética de punto fijo (al no contar con FPU) , y visualice la energía de cada banda de frecuencia como barras de colores en una matriz de LEDs RGB.

Se busca además que el procesamiento sea lo suficientemente eficiente para convivir con el resto del sistema dentro de un ejecutor cíclico (Super-Loop) cuya vuelta completa dure menos de 1 ms, sin código bloqueante. El sistema incorpora una interfaz local para visualización de las configuraciones (OLED, dip-switch, potenciómetro), persistencia de la configuración (modo SET_UP) y telemetría por Bluetooth.

**Resultados esperados:**

- Espectro de audio en vivo, de aproximadamente 0 a 8 kHz, mostrado en 8 columnas de una matriz WS2812 de 8x8.
- Super-Loop medido con el contador de ciclos DWT, con un peor caso mucho menor a 1 ms con todos los módulos que requieran utilizar la CPU funcionando en simultáneo.
- Al menos tres modos de operación (NORMAL, SET_UP y FALLA) con menú interactivo y configuración persistente con almacenamiento en la memoria flash presenta en la placa de desarrollo.

### **1.2 Proyectos similares**

A lo largo de la lluvia de ideas que se realizó para elegir un proyecto a desarrollar, fueron consideradas las siguientes tres ideas:

1. **Analizador de Espectro por Software (FHT) con Matriz LED RGB (8x8).**
2. **Reloj Interactivo (Omnitrix).**
3. **Efecto de Audio Digital en Tiempo Real (ej. Pedal de Delay).**

Para compararlos se tienen en cuenta seis aspectos:

1. **Disponibilidad del hardware:** facilidad para conseguir los componentes en el mercado regional.
2. **Atractivo visual e interfaz:** capacidad de ofrecer una experiencia fluida, vistosa y fácil de configurar.
3. **Complejidad de procesamiento (software):** dificultad algorítmica y aprovechamiento de la arquitectura del microcontrolador.
4. **Tiempo de implementación:** tiempo total para diseñar e integrar hardware y software.
5. **Costo:** costo total del hardware requerido.
6. **Interés personal:** motivación para desarrollar la solución.

Se ponderan los aspectos del 1 al 10: Disponibilidad: 10, Atractivo visual: 8, Complejidad: 6, Tiempo: 8, Costo: 5, Interés personal: 8.

| Proyecto | Analizador FHT (Software DSP) | Omnitrix (Reloj interactivo) | Efecto de Audio Digital en Tiempo Real |
| ----- | ----- | ----- | ---- |
| **Disponibilidad de Hardware (10)** | 8 (80) | 9 (90) | 8 (80) |
| **Atractivo Visual (8)** | 10 (80) | 8 (64) | 4 (32) |
| **Complejidad de Software (6)** | 10 (60) | 4 (24) | 6 (36) |
| **Tiempo de Implementación (8)** | 10 (80) | 7 (56) | 7 (56) |
| **Costo (5)** | 5 (25) | 5 (25) | 7 (35) |
| **Interés Personal (8)** | 10 (80) | 4 (32) | 8 (64) |
| **Puntaje Total** | **405** | **291** | **303** |

*Tabla 1.2.1: Comparación de proyectos*

### **1.3 Selección de proyecto**

Considerando la Tabla 1.2.1, se elige el **Analizador de Espectro por Software (FHT) con Matriz WS2812**. El reloj interactivo se descarta principalmente por el bajo interés personal presente en el grupo, y el pedal de efectos por su bajo atractivo visual y por requerir un DAC de alta fidelidad, el cual eleva el costo y la complejidad electrónica con respecto a los demás.

El proyecto destaca por usar la **Transformada Rápida de Hartley (FHT) en aritmética de punto fijo (Q15)**, lo que resuelve la ausencia de unidad de punto flotante (FPU) en el STM32F103RB (núcleo Cortex-M3). Para la visualización se utiliza una matriz económica **WS2812 (8x8)**, gobernada por un timer en modo PWM con DMA mediante un driver de terceros (repositorio en GitHub de T. Majerle), en lugar de las matrices con protocolo SPI (APA102), de mayor costo. Ese driver trabaja con DMA circular e interrupciones de media transferencia y transferencia completa, por lo que la transmisión es asíncrona respecto del Super-Loop, aunque consume algo de CPU en cada callback en caso de interrupciones.

La señal de audio se muestrea a **16 kHz** y la etapa analógica de entrada incluye un **filtro pasabajos pasivo** sencillo (filtro de primer orden). La entrada de audio proviene de un conector Jack de 3,5 mm (sección 6). Parte del procesamiento de la visualización se simplifica con **tablas constantes almacenadas en la flash** del microcontrolador (sección 5.6).

Complementan el proyecto una pantalla OLED por I2C para visualización del estado y menús de configuración, persistencia de la configuración en la flash interna, telemetría por Bluetooth (HM-10), un potenciómetro para modificar la ganancia y dip-switches y un botón para seleccionar el modo, con antirrebote resuelto por software.

---

## **2. Alcance: producto mínimo y extras**

El trabajo se divide en un producto mínimo viable (MVP) y extras que se abordarían si el tiempo lo permite.

**MVP (compromiso):**

- Captura de audio con ADC disparado por timer y DMA con doble buffer (256 muestras por buffer), a 16 kHz.
- Acondicionamiento analógico de la entrada por Jack (acople, polarización y filtro pasabajos pasivo).
- FHT de 256 puntos en Q15, magnitud y agrupado en 8 bandas, mostrado en la matriz WS2812.
- Super-Loop con tick de 1 ms (SysTick con callback), tareas no bloqueantes y bajo consumo (`__WFI()`).
- Antirrebote por software del botón y de los dip-switches.
- Máquina de estados con modos NORMAL, SET_UP, FALLA y menú en OLED (I2C).
- Ganancia ajustable con potenciómetro y guardada en la flash interna (datos cargados en SET_UP).
- Modo FALLA con buzzer por detección de saturación (clipping).
- Telemetría por HM-10 (UART + DMA): una línea de texto con la frecuencia pico y el modo de operación actual.
- Uso de tablas constantes en flash (paleta de colores, corrección gamma, mapeo de píxeles, umbrales de nivel) para simplificar el mapeo del espectro.

**Extras (en orden aproximado de prioridad):**

1. Modos de visualización seleccionables: color por columna o color por altura (tipo VU).
2. Trama de LEDs precodificada por columna y nivel en flash, si alcanzara la memoria. (sección 5.6).
3. Filtros digitales o ecualización seleccionables por dip-switch.

---

## **3. Elicitación de requisitos**

| Grupo | ID | Descripción | Prioridad |
| ----- | ----- | ----- | ----- |
| **Captura de Audio** | 1.1 | Entrada de audio por conector Jack 3.5 mm con acople por capacitor, polarización a mitad de la escala del ADC (~1,65 V). | MVP |
|  | 1.2 | Filtro pasabajos pasivo (RC de primer orden) antes del ADC para limitar el aliasing. | MVP |
|  | 1.3 | Muestreo ininterrumpido con ADC1 disparado por timer y transferencia por DMA a un doble buffer en RAM (bloques de 256 muestras, fs = 16 kHz). | MVP |
| **Procesamiento** | 2.1 | Cálculo de la FHT de 256 puntos en punto fijo Q15, con escalado por etapa para evitar desbordes. | MVP |
|  | 2.2 | Cálculo de magnitudes (aproximación alfa-máx + beta-mín, sin raíz cuadrada, con opción de magnitud exacta) y agrupado de los bins en 8 bandas en escala logarítmica. | MVP |
|  | 2.3 | Aplicación de la ganancia (potenciómetro) antes de cuantizar. | MVP |
| **Visualización** | 3.1 | Representación del espectro como 8 barras de colores en una matriz WS2812 de 8x8. | MVP |
|  | 3.2 | Envío de datos a la matriz mediante PWM por timer asistido por DMA, sin esperas activas. | MVP |
|  | 3.3 | Límite de brillo máximo, para acotar el consumo de corriente. | MVP |
|  | 3.4 | Uso de tablas constantes en flash (paleta de colores, corrección gamma, mapeo de píxeles, umbrales de nivel) para simplificar el armado de la trama. | MVP |
|  | 3.5 | Trama precodificada por columna y nivel almacenada en flash. | Extra |
|  | 3.6 | Modos de visualización: color por columna o por altura. | Extra |
| **Interfaz Local** | 4.1 | Pantalla OLED (I2C) con estado y menús, actualizada de forma no bloqueante (DMA o interrupciones). | MVP |
|  | 4.2 | Potenciómetro analógico (ADC2, lectura no bloqueante) para ajustar la ganancia. | MVP |
|  | 4.3 | Dip-switches para alternar entre modo NORMAL y SET_UP, y botón de confirmación. | MVP |
|  | 4.4 | Antirrebote por software de botón y dip-switches: se muestrean en el tick de 1 ms y un cambio se acepta solo tras varias lecturas consecutivas iguales, sin esperas bloqueantes ni componentes adicionales. | MVP |
| **Persistencia** | 5.1 | Guardado de ganancia y modo en la flash interna, escrito únicamente en SET_UP. | MVP |
|  | 5.2 | Guardado del modo de visualización. | Extra |
| **Telemetría** | 6.1 | Transmisión por UART con DMA hacia el módulo bluetooth HM-10 (frecuencia pico y modo). | MVP |
|  | 6.2 | Visualización en una aplicación móvil. | MVP |
| **Alertas** | 7.1 | Buzzer y modo FALLA ante saturación de la entrada, con aviso en el OLED. | MVP |
| **Arquitectura** | 8.1 | Bare metal (sin sistema operativo), Event-Triggered, Super-Loop con vuelta menor a 1 ms. | MVP |
|  | 8.2 | Tick de 1 ms por SysTick con callback que solo levanta banderas. | MVP |
|  | 8.3 | Código estructurado y modular (módulos separados por función); tareas no bloqueantes. | MVP |
|  | 8.4 | Bajo consumo: `__WFI()` al finalizar la vuelta cuando no hay trabajo pendiente. | MVP |

*Tabla 3.1: Requisitos del proyecto*

---

## **4. Casos de uso**

| Elemento | Definición |
| ----- | ----- |
| **Disparador** | El usuario reproduce música por la entrada de audio (Jack) y desea visualizar el espectro. |
| **Precondiciones** | El sistema está encendido y el dip-switch está en modo NORMAL. |
| **Flujo principal** | El ADC muestrea de forma continua y, al completarse cada bloque, el callback del DMA avisa a la CPU para comenzar el procesamiento. El Super-Loop calcula la FHT, las magnitudes y las agrupa en 8 bandas, arma la trama con ayuda de las tablas en flash y ordena al DMA enviarla a la matriz WS2812. El OLED muestra el estado del sistema y el módulo Bluetooth envía la frecuencia de mayor energía a la aplicación del usuario. |
| **Flujos alternativos** | **a.** *(Extra)* Si la entrada satura, el sistema pasa al modo FALLA, activa el buzzer y lo notifica en el OLED. |

*Tabla 4.1: Caso de uso 1 - Análisis en tiempo real*

| Elemento | Definición |
| ----- | ----- |
| **Disparador** | El usuario desea cambiar la ganancia de las barras y guardarla. |
| **Precondiciones** | El sistema opera en modo NORMAL y las barras quedan muy bajas porque la señal de audio es débil. |
| **Flujo principal** | El usuario cambia el dip-switch a SET_UP. El análisis se pausa y el OLED muestra el menú de ajustes. El usuario gira el potenciómetro hasta alcanzar la ganancia deseada y presiona el botón de confirmación y el sistema guarda el valor en la flash interna. Al devolver el dip-switch a la posición que se corresponde con el estado NORMAL, el análisis se reanuda con la nueva ganancia. |

*Tabla 4.2: Caso de uso 2 - Configuración persistente*

---

## **5. Arquitectura propuesta**

### **5.1 Esquema de módulos y flujo de datos**

Esquema general de los módulos del sistema:

```
 +------------------------------ ENTRADAS ------------------------------+
 |  Jack 3,5 mm       Dip-switch     Boton        Potenciómetro         |
 |  (audio)            (modo)       (extra)              (ADC2)         |
 +--------+---------------+-------------+----------------+--------------+
          |               |             |                |
          v               +------+------+                |
 +------------------+            v                       |
 | Acondicionamiento|     +--------------+               |
 | (atenuacion,     |     | Antirrebote  |               |
 | polarizacion y   |     | por software |               |
 | filtro RC pasivo)|     +------+-------+               |
 +--------+---------+            |                       |
          v                      |                       |
 +-------------------+           |                       |
 | ADC1 + Timer + DMA|           |                       |
 | 16 kHz, doble     |           |                       |
 | buffer            |           |                       |
 +--------+----------+           |                       |
          | bloque listo         |                       |
          v                      v                       v
 +----------------------------------------------------------------------+
 |                             SUPER-LOOP                               |
 |         tick 1 ms (SysTick) -> banderas -> tareas -> __WFI()         |
 |                                                                      |
 |   [Espectro]            [Maquina de estados]      [Configuracion]    |
 |   quitar DC             NORMAL/SET_UP/FALLA       ganancia y modo    |
 |    FHT 256                                                           |
 |  magnitud y bandas                                                   |
 +----+------------------+------------------+------------------+--------+
      |                  |                  |                  |
      v                  v                  v                  v
 +-----------+      +-----------+      +------------+     +------------+
 | Driver    |      | Pantalla  |      | Telemetria |     | Flash      |
 | LEDs      |      | OLED      |      | UART + DMA |     | interna    |
 | PWM + DMA |      | (I2C)     |      | (HM-10)    |     | (config y  |
 +-----+-----+      +-----------+      +------------+     |  tablas)   |
       |                                                  +------------+
       v
 +-------------+
 | Matriz      |        Buzzer (alerta en modo FALLA)
 | WS2812 8x8  |
 +-------------+
```

Flujo de datos del audio:

```
Jack 3,5 mm -> [Acople + polarizacion 1,65 V]
      -> [Filtro pasabajos pasivo] -> ADC1 (disparo por timer, fs = 16 kHz)
      -> DMA circular (doble buffer, 2 x 256 muestras)
      -> callback HT/TC: levanta bandera "bloque listo"
      -> Super-Loop: Arreglo 256 Q15 -> FHT 256 -> magnitud -> 8 bandas -> 8 niveles
      -> armado de trama (tablas en flash) -> Timer PWM + DMA -> matriz WS2812
```

Parámetros: fs = 16 kHz y N = 256, lo que da una resolución de 62,5 Hz por bin ("pares" de frecuencias), un ancho de banda máximo de 8 kHz (Nyquist) y un bloque nuevo para procesar cada 16 ms (aproximadamente 62 actualizaciones por segundo). Se muestran las frecuencias hasta aproximadamente 8 kHz.

### **5.2 Agrupado en bandas**

Las 8 columnas de la matriz agrupan los bins de la FHT en escala aproximadamente logarítmica (angostas en graves, anchas en agudos). Propuesta inicial, a ajustar con audio real:

| Columna | Bins | Frecuencia aprox. |
| ----- | ----- | ----- |
| 1 | 1 | 62 Hz |
| 2 | 2 | 125 Hz |
| 3 | 3-4 | 190-250 Hz |
| 4 | 5-8 | 310-500 Hz |
| 5 | 9-16 | 560-1000 Hz |
| 6 | 17-32 | 1,1-2 kHz |
| 7 | 33-64 | 2,1-4 kHz |
| 8 | 65-127 | 4,1-7,9 kHz |

Las frecuencias son bin x 62,5 Hz (resolución espectral). El bin 0 (componente continua) se descarta, ya que no resulta útil para la visualización en la matriz LED. Con fs = 16 kHz las dos columnas más graves quedan con un solo bin cada una. Las bandas se definen en Hz y se traducen a bins al iniciar, de modo que un cambio de fs no obliga a reescribir tablas. Dentro de cada banda se toma el máximo o el promedio de las magnitudes.

Para pasar de amplitud a altura de barra (8 niveles en la matriz) se usa una escala logarítmica (por representación en dB), implementada con una tabla de umbrales en la memoria flash.

### **5.3 Tareas del Super-Loop**

Todas las tareas son no bloqueantes y se disparan por flags desde callbacks.

| Tarea | Disparo | Descripción |
| ----- | ----- | ----- |
| Espectro | Bloque de audio listo | FHT, magnitud, agrupado y cuantización.|
| Pantalla LED | Espectro nuevo y driver libre | Armado de la trama con las tablas en flash y lanzamiento del DMA; no se inicia un envío si el anterior sigue en curso. |
| Entradas | Tick de 1 ms | Antirrebote por software de botón y dip-switches e inicio/lectura no bloqueante de las conversiones del ADC2 (potenciómetro). |
| Máquina de estados | Eventos de entradas | Transiciones NORMAL / SET_UP / FALLA. |
| OLED | Cambio de estado o menú | Actualización por fragmentos, sin esperas. |
| Flash | Confirmación en SET_UP | Borrado y escritura de la página de configuración (solo con el análisis pausado). |
| Telemetría | Periódica (por ejemplo, 1 Hz) | Armado de la línea de texto y envío por UART con DMA. |
| Bajo consumo | Fin de la vuelta | `__WFI()` hasta el próximo evento. |

### **5.4 Máquina de estados**

- **NORMAL:** Muestreo, procesamiento y análisis activos, barras en la matriz y telemetría en app.
- **SET_UP:** Análisis pausado, menú de ajuste de ganancia en el OLED y guardado de la configuración en memoria flash.
- **FALLA:** Saturación detectada, con buzzer y aviso en pantalla OLED.

### **5.5 Asignación de periféricos y DMA (preliminar)**

El DMA1 del STM32F103RB dispone de 7 canales con peticiones fijas por periférico. La asignación final debe analizarse para evitar conflictos entre perisféricos.

| Función | Periférico | DMA | Observaciones |
| ----- | ----- | ----- | ----- |
| Audio | ADC1 + timer de disparo | DMA1 Canal 1 (ADC1) | Modo circular con interrupciones de media y fin de transferencia. Para 16 kHz, con el timer a 72 MHz: PSC = 0 y ARR = 4499. |
| Matriz WS2812 | Timer PWM (canal a definir) | Canal según timer | ARR = 89 a 72 MHz para 800 kHz |
| Telemetría | USART + HM-10 | Canal TX según USART | Envío no bloqueante. |
| OLED | I2C | DMA o interrupciones | Actualización por fragmentos. |
| Potenciómetro | ADC2 | No disponible (se está utilizando en el ADC1) | Lectura por polling no bloqueante a baja frecuencia, alternando canales. |
| Tick | SysTick | - | Callback cada 1 ms. |

### **5.6 Memoria y uso de la flash**

La F103RB cuenta con 20 KB de RAM y 128 KB de flash. En RAM se estima: el doble buffer del ADC (2 x 256 muestras de 16 bits, 1 KB), los buffers de entrada y salida de la FHT (2 x 512 B), las magnitudes, la tabla de twiddles copiada a RAM para lecturas rápidas y el buffer del driver de la matriz (hasta unos 3 KB con la trama completa), además del espacio necesario para los caracteres del OLED (~1 KB).

La flash, en cambio, tiene espacio de sobra, y se aprovecha para **aliviar el procesamiento de los LEDs y de la interfaz** guardando datos precalculados como constantes en tablas. Se plantean tres niveles, a evaluar según convenga durante la integración:

| Nivel | Tabla en flash | Tamaño aproximado | Qué ahorra |
| ----- | ----- | ----- | ----- |
| 1 (MVP) | Paleta de colores por columna, corrección gamma (256 valores), mapeo de píxeles en la matriz, umbrales de nivel de las barras, bandas en bins | Menos de 1 KB | Evita calcular colores, brillo, posiciones y logaritmos en cada cuadro. |
| 2 | Tabla de expansión: para cada valor de color de 8 bits, los 8 valores PWM correspondientes | ~2 KB (elementos de 8 bits) a ~4 KB (16 bits) | Reemplaza la conversión bit a bit de cada color por una copia desde la tabla. |
| 3 (extra) | Trama PWM ya codificada para cada columna y cada nivel (8 columnas x 9 niveles x 8 LEDs x 24 bits) | ~14 KB (8 bits) a ~28 KB (16 bits) | Armar un cuadro se reduce a copiar 8 tramos; el brillo máximo queda fijo en la tabla. |

Además se guardan en la memoria flash las fuentes y textos del menú del OLED y la página de configuración persistente (ganancia, modo). Las tablas de uso muy frecuente (como los twiddles de la FHT) se copian a RAM al iniciar para evitar los estados de espera de la flash; las tablas pequeñas de los LEDs se leen directamente de la flash. Las opciones 2 y 3 requieren adaptar o complementar el driver de terceros y la forma exacta de cableado de la matriz, por lo que se evaluarán después de tener funcionando el nivel 1.

---

## **6. Entrada de audio y hardware**

### **6.1 Hardware del sistema**

- Placa NUCLEO-F103RB.
- Matriz WS2812 8x8 con alimentación de 5 V externa, masa común.
- Etapa analógica de entrada (sección 6.2) con conector Jack 3.5 mm.
- Pantalla OLED I2C, módulo HM-10, dip-switches, botón, potenciómetro y buzzer.
- Interconexión: **placa experimental con componentes y conectores soldados**.
### **6.2 Acondicionamiento de la señal**

El ADC de la F103 mide entre 0 y 3,3 V, mientras que el audio es una señal alterna centrada en 0 V. La etapa de entrada debe:

1. **Polarizarla** a mitad de escala (~1,65 V) con un divisor resistivo desacoplado, para que la señal oscile alrededor de ese punto.
2. **Limitar** su amplitud a la ventana del ADC y proteger la entrada (resistencia serie y diodos a los rieles de 0 V y 3,3 V). La salida de línea o de auriculares de un celular o una PC puede entregar del orden de 1 V, que a volumen alto supera la ventana del ADC, por lo que se prevé una atenuación (divisor o potenciómetro).
3. **Filtrarla** con un pasabajos antes del ADC (sección 6.3).

### **6.3 Filtro antialiasing**

Se adopta un **filtro pasabajos pasivo RC de primer orden**, con frecuencia de corte del orden de unos pocos kHz; los valores definitivos de R y C se ajustarán probando con el circuito real. Un filtro de primer orden atenúa poco cerca de su corte, por lo que no elimina por completo el aliasing; esa limitación se compensa eligiendo una frecuencia de muestreo (16 kHz) bastante mayor que el doble del rango que se muestra (~8 kHz). Se pueden llegar a evualuar posibles mejores, como un filtro activo de segundo orden.

---

## **7. Resultados preliminares**

Se implementó y midió en la placa el núcleo de procesamiento (FHT de 256 puntos y magnitudes) sobre una señal de entrada de ruido pseudoaleatorio de amplitud máxima, que es la más exigente para el cálculo de magnitudes. Todas las mediciones se realizaron con optimización **-O2** y con el contador de ciclos DWT (mínimo de 20 repeticiones), sobre un núcleo de 72 MHz.

| Tramo | Ciclos | Tiempo |
| ----- | ----- | ----- |
| FHT de 256 puntos | ~20.550 | 285 µs |
| Magnitudes, aproximación alfa-máx + beta-mín | ~3.600 | ~50 µs |
| **Total (FHT + magnitudes aproximadas)** | **24.147** | **335 µs** |
| Magnitudes con raíz exacta (alternativa) | ~20.230 | ~281 µs |
| Total con magnitudes exactas | 40.781 | 566 µs |

Estas mediciones corresponden solo al núcleo de procesamiento y no incluyen las demás tareas del sistema ni las interrupciones de los periféricos, que se medirán durante la implementación.

**Optimizaciones aplicadas a la FHT:**

- Eliminación del reordenamiento por bit-reversal con intercambios: se fusiona con las dos primeras etapas de la FHT usando la instrucción de inversión de bits del Cortex-M3.
- Eliminación de la característica "in place", ahora la FHT escribe en un buffer distinto del de entrada, por lo que no destruye las muestras.

**Magnitudes.** La magnitud de cada frecuencia se obtiene combinando H[k] y H[N-k], como la magnitud teórica sigue la expresión: |X[k]| = raíz((H[k]^2 + H[N-k]^2) / 2).



Las barras solo requieren la energía de cada banda,por lo que se puede ignorar la fase. La aproximación alfa-máx + beta-mín evita ca la raíz cuadrada, tiene tiempo de cálculo constante e introduce un error máximo de aproximadamente 4 % (~0,4 dB), imperceptible en una visualización de 8 niveles.

**Verificación funcional:** Con un seno de amplitud 32767 en el bin 16, el espectro muestra el pico en ese bin con magnitud ~16382, coherente con el valor teórico.

---

## **8. Riesgos y mitigación**

| Riesgo | Impacto | Mitigación |
| ----- | ----- | ----- |
| Etapa analógica de entrada mal polarizada o ruidosa | Alto: sin señal válida no hay espectro | Probar la entrada desde el inicio con un generador de tonos; verificar el bin esperado en el debugger. |
| Aliasing por filtro de primer orden | Medio: columnas agudas con energía falsa | Elegir fs bien por encima del doble del rango mostrado y realizar correctamiente el diseño, eligiendo bien los valores de R y C |
| Nivel de entrada inadecuado (Jack saturado a volumen alto o señal muy débil) | Medio | Atenuación y protección en la entrada, ganancia ajustable y detección de clipping. |
| Rebotes del botón y de los dip-switches | Bajo | Antirrebote por software en el tick de 1 ms, sin esperas bloqueantes. |
| Interferencia de la matriz LED sobre el ADC y la alimentación | Alto | Fuente de 5 V aparte, masa común, brillo limitado. |
| Código bloqueante (I2C del OLED, escritura de flash, esperas de DMA) que supere 1 ms | Alto | I2C por DMA/interrupciones y por fragmentos; flash escrita solo en SET_UP, medición con DWT de cada incorporación para verificar correcto funcionamiento. |
| Conflictos de canales de DMA y de prioridades de interrupción | Medio | Tabla de asignación previa y prioridades definidas antes de configurar. |
| Acumulación de tiempo de CPU al sumar tareas | Medio | Medir el peor caso de vuelta tras cada módulo, partir la FHT en etapas por vuelta o usar tablas en flash si fuera necesario. |
| Adaptar las tablas precodificadas al driver de terceros y a la matriz | Bajo/medio | Empezar con las tablas del nivel 1 y pasar a los niveles 2 y 3 solo si se justifica. |

---

## **9. Puntos a consensuar con los docentes**

1. Uso de un driver de terceros (licencia MIT) para la matriz WS2812, con la debida cita.
2. Uso de la flash interna para la persistencia (consulta si se puede prescindir de la EPROM).
3. Aceptación de módulos comerciales (matriz WS2812, módulo OLED, módulo HM-10) como hardware adicional, junto con componentes extras para etapas de entrada analógica (filtro pasabajos pasivo y, si quedara corto, filtro activo de segundo orden).
---

## **10. Referencias**

- T. Majerle, *WS2811 and WS2812B driver for STM32 with TIM, PWM and DMA* (licencia MIT): <https://github.com/MaJerle/stm32-ws2811-ws2812-ws2812b-ws281x-tim-pwm-dma-timer>
- STMicroelectronics, *RM0008 - Reference manual STM32F101xx, STM32F102xx, STM32F103xx, STM32F105xx y STM32F107xx*.
- Pautas y requisitos del Trabajo Final, TA134 - Taller de Sistemas Embebidos (rev. 05, 2026).
