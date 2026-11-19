<img src="https://www.fi.uba.ar/images/logo-fiuba.png" alt="Logo FIUBA" width="50%">
 
 # **Analizador de Espectro de Audio con FHT**
 
 ## **Informe de Avances**

**Autores**

**Sorroza Fausto Esteban \- Padrón 112909**
**Maizares Bruno \- Padrón 112959**
**Cingolani Camilo \- Padrón 112744**

**Fecha: 19/11/2026**

**2do cuatrimestre 2026**

A continuación se detalla el informe de avances del TF (Analizador de Espectro de Audio con FHT) a partir de los requisitos presentados en el documento de la propuesta.

| Estado | Descripción      |
|-----|---------------------|
| 🟢 | Ya implementado |
| 🟡 | En proceso de implementarse |
| 🔴 | No se implementará |

#### **Captura de audio**  

| Req ID | Descripción | Estado | 
|--------|-------------|--------|  
| 1.1 | El sistema capturará audio desde un conector Jack de 3,5 mm, con acople, polarización y protección de la entrada. | 🟡 |
| 1.2 | Un filtro pasabajos RC de primer orden limitará el aliasing antes del ADC. | 🟡 |
| 1.3 | El muestreo será continuo, a 16 kHz, con el ADC disparado por un timer y los datos transferidos por DMA a un doble buffer (tamaño para 256 muestras c/u). | 🟡 |

#### **Procesamiento**  

| Req ID | Descripción | Estado |  
|--------|-------------|--------|  
| 2.1 | El sistema calculará una FHT de 256 puntos en aritmética de punto fijo. | 🟢 |
| 2.2 | El sistema agrupará el espectro de 0 a 8 kHz en 8 bandas y cuantizará cada una a 8 niveles en escala logarítmica. | 🟢 |
| 2.3 | La ganancia ajustada con el potenciómetro se aplicará antes de cuantizar. | 🟡 |

#### **Visualización**  

| Req ID | Descripción | Estado |  
|--------|-------------|--------|  
| 3.1 | El sistema mostrará el espectro como 8 barras de colores en una matriz WS2812 de 8x8. | 🟡 |
| 3.2 | El envío de datos a la matriz se hará mediante timer PWM con DMA, sin esperas bloqueantes. | 🟡 |
| 3.3 | El brillo máximo de la matriz estará limitado para acotar el consumo de corriente. | 🟡 |

#### **Interfaz local**  

| Req ID | Descripción | Estado |  
|--------|-------------|--------|  
| 4.1 | Una pantalla OLED (I2C) mostrará el estado y los menús, y se actualizará sin bloquear el sistema. | 🟡 |
| 4.2 | Un potenciómetro permitirá ajustar la ganancia. | 🟡 |
| 4.3 | Los dip-switches permitirán alternar entre los modos NORMAL y SET_UP, y un botón confirmará las selecciones. | 🟡 |
| 4.4 | El botón y los dip-switches tendrán antirrebote por software, sin esperas bloqueantes. | 🟡 |
#### **Persistencia**
| Req ID | Descripción | Estado |  
|--------|-------------|--------|  
| 5.1 | El sistema guardará la ganancia y el modo en la flash interna, únicamente desde el modo SET_UP. | 🟡 |
| 5.2 | Al encender, el sistema cargará la configuración guardada en la flash. | 🟡 |

#### **Telemetría**
| Req ID | Descripción | Estado |  
|--------|-------------|--------|  
| 6.1 | El sistema enviará por UART con DMA, hacia el módulo Bluetooth HM-10, una línea de texto con la frecuencia pico y el modo de operación. | 🟡 |
| 6.2 | Los datos podrán verse en un celular mediante una aplicación de terminal Bluetooth. | 🟡 |

#### **Alertas**
| Req ID | Descripción | Estado |  
|--------|-------------|--------|  
| 7.1 | Si la entrada satura, el sistema pasará al modo FALLA, activará un buzzer y mostrará un aviso en el OLED. | 🟡 |
| 7.2 | El sistema volverá al modo NORMAL cuando la señal deje de saturar. | 🟡 |

#### **Arquitectura**
| Req ID | Descripción | Estado |  
|--------|-------------|--------|  
| 8.1 | El sistema será bare metal (sin sistema operativo), con un Super-Loop cuya vuelta dure menos de 1 ms. | 🟡 |
| 8.2 | El tick de 1 ms se generará con el SysTick, y su callback solo levantará flags. | 🟡 |
| 8.3 | El código será modular, con tareas no bloqueantes. | 🟡 |
| 8.4 | El sistema ejecutará `__WFI()` al terminar la vuelta cuando no haya trabajo pendiente. | 🟡 |
