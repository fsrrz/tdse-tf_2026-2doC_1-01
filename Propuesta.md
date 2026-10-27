**1. Selección del proyecto a implementar**

*1.1 Objetivo del proyecto*

El objetivo de este proyecto es diseñar e implementar un sistema embebido interactivo en donde buscamos implementar un juguete basado en el reloj "omnitrix" de la serie para niños Ben 10, en donde el protagonista tiene un reloj con el que gira un dial para intercambiar en que alienigena convertirse al presionarlo. El sistema integrará diversos periféricos de entrada/salida para navegar menús, rotar entre los diferentes aliens, reproducir secuencias visuales y emitir sonidos mediante eventos temporizados, y comunicarse con una aplicación móvil que actuará como panel de estado, mostrando datos y trivia del alien seleccionado, permitiendo desbloquear más personajes contestando preguntas de datos de la serie. Todo el procesamiento se realizará bajo un patrón de diseño de software no bloqueante (Super-loop), respetando las limitaciones arquitectónicas del microcontrolador.

<img src="https://i.redd.it/h8cr1vmo8hbf1.jpeg" alt="image2" width="50%">
 
<img src="ben10.gif" alt="Ben 10" width="300"/>

*1.2 Proyectos similares analizados*

Antes de decidir que proyecto llevar a cabo, evaluamos otro posible proyecto, se evaluaron tres alternativas considerando los requisitos de la cátedra y las limitaciones de hardware (Cortex-M3 sin FPU/DSP):

Analizador de Espectro de Audio FFT: Captura de audio analógico y cálculo de Transformada Rápida de Fourier para controlar tiras LED, similar al que se ve en los equipos de música (estereo), pero con perfiles personalizables y múltiples entradas.

Omnitrix Interactivo (Seleccionado): Dispositivo de entretenimiento (juguete) con interfaz física (dial analógico, botones), display LED, efectos de luces/sonido y conexión Bluetooth.

A continuación se adjunta una pequeña tabla de análisis de riesgos para estos proyectos.

| Criterio | Analizador Espectro FFT | Omnitrix Interactivo |
| :--- | :---: | :---: |
| **Uso óptimo de CPU (Cortex-M3)** | Bajo (Riesgo de bloqueo) | Alto |
| **Cumplimiento de requisitos obligatorios** | Medio | Alto |
| **Interés del grupo** | Alto | Alto |
| **Viabilidad técnica** | Baja (Falta de DSP) | Alta |

Si bien nos interesaba mucho el proyecto del Analizador de Espectro, notamos que podríamos exceder los límites de hardware de nuestro microprocesador, no pudiendo completar todo lo necesario en 1 ms. Esto último debido a que al carecer de una únidad de cálculo de punto flotante dedicada, es muy dificil efectuar una FFT sin que esta demande demasiado poder de procesamiento.

En cuanto a disponibilidad de hardware o costo del mismo no debería de ser un impedimento para la implementación del proyecto, puesto que el componente más caro en un principio sería el display en donde se mostrarán los alienigenas.

## 1.3 Selección y Justificación
Se selecciona el **Omnitrix Interactivo** dado que permite cumplir con la totalidad de los requisitos obligatorios de hardware (ADC, I2C/SPI, Bluetooth, etc) sin comprometer el factor de uso de la CPU. Al basarse en máquinas de estado y eventos temporizados en lugar de calculos matemáticos intensivos, asegura que las iteraciones del ejecutor cíclico se mantengan  por debajo de 1 ms.
