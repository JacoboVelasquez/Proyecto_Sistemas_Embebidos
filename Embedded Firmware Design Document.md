# Entrega Final

# Diagrama de bloques del sistema embebido:
![alt text](32.jpg)

# Esquematico del circuito

![alt text](31.jpg)

# Arquitectura de Firmware

El firmware del sistema embebido fue diseñado utilizando una arquitectura modular por capas, permitiendo separar la lógica de control, las comunicaciones, el manejo de hardware y la presentación de información. Esta estructura facilita la mantenibilidad, escalabilidad y depuración del sistema.

La arquitectura implementada se organiza de la siguiente manera: 

```text
app_main()              ← Capa de aplicación (lógica principal)
    ├── uart_process()  ← Capa de comunicaciones UART
    ├── lcd_update()    ← Capa de presentación (LCD I2C)
    ├── log_msg()       ← Capa de logging y monitoreo
    ├── adc_avg()       ← Capa de adquisición de sensores (HAL)
    └── gpio_set_level()← Capa de actuación (HAL)
```
## Descripción de capas
- **Capa de aplicación:**
La función app_main() contiene la lógica principal del sistema, encargándose del monitoreo de sensores y control de actuadores.
- **Capa de comunicaciones:**
La función uart_process() permite recibir comandos UART y enviar información del sistema mediante comunicación serial.
- **Capa de presentación:**
La función lcd_update() actualiza la pantalla LCD 16x2 utilizando protocolo I2C para mostrar el estado del sistema.
- **Capa de logging:**
La función log_msg() registra eventos, alertas y errores mediante mensajes seriales con diferentes niveles de severidad.
- **Capa HAL:**
Las funciones HAL abstraen el acceso al hardware mediante control de ADC, GPIO, UART e I2C.


---

# Justificación de la Arquitectura

La arquitectura del firmware fue diseñada bajo criterios de modularidad, simplicidad, mantenibilidad y desacoplamiento del hardware.

## 1. Uso de HAL (Hardware Abstraction Layer)

La capa HAL encapsula el acceso al hardware mediante funciones independientes para ADC, GPIO, UART e I2C.

Esto permite:
- Facilitar mantenimiento.
- Simplificar depuración.
- Reducir dependencias directas con el hardware.
- Facilitar migración a otros microcontroladores.
- La lógica de control no cambia

---

## 2. Modularidad del sistema

Cada subsistema fue implementado de manera independiente:

- Sensado.
- Comunicación UART.
- Logging.
- Control de actuadores.
- Interfaz LCD.

Esto permite aislar errores y agregar o quitar componentes sin afectar el funcionamiento general del sistema.

---

## 3. Separación de responsabilidades

La función `app_main()` únicamente toma decisiones de control, por ejemplo if temperatura < min → encender lámpara, mientras las funciones HAL y de comunicación manejan la interacción directa con el hardware.

Esto mejora la organización y claridad del firmware.

---

## 4. Arquitectura secuencial determinística

El sistema utiliza un único loop principal sincronizado mediante:

```text
vTaskDelay(pdMS_TO_TICKS(1000))
```
Esta decisión fue tomada debido a que las variables monitoreadas:

- Temperatura
- Nivel de bebedero
- Nivel del tanque

No requieren tiempos críticos independientes ni procesamiento concurrente complejo.

El uso de una sola tarea principal:

- Reduce complejidad.
- Evita problemas de sincronización.
- Simplifica la depuración.
- Disminuye riesgo de race conditions.

## 5. Configuración dinámica mediante UART

Las variables g_temp_min y g_temp_max fueron declaradas como volatile debido a que pueden modificarse dinámicamente mediante comandos UART durante la ejecución del sistema.

Esto garantiza:

- Correcta lectura de valores actualizados.
- Prevención de optimizaciones incorrectas del compilador.
- Configuración en tiempo real sin recompilar firmware.

# Diagrama de Estados del Firmware

![alt text](Diagrama_estados.png)

# Estrategia de Manejo de Errores

El firmware implementa una estrategia de manejo de errores basada en validación de sensores, registro de eventos y operación segura del sistema.

Cada sensor es validado mediante rangos de operación permitidos antes de utilizar sus valores dentro de la lógica de control.

## Validación de sensores

El sistema verifica continuamente que las lecturas ADC se encuentren dentro de límites válidos previamente definidos.

Ejemplos:

```c
#define LM35_RAW_MIN        50
#define LM35_RAW_MAX        4000

#define SENSOR_RAW_MIN      10
#define SENSOR_RAW_MAX      4090
```
Si un sensor retorna valores fuera de rango, el sistema interpreta la condición como un posible fallo de lectura o desconexión del sensor.

### Códigos de error

El firmware implementa códigos específicos para identificar cada tipo de fallo:

- ERR_SENSOR_LM35
- ERR_SENSOR_BEBEDERO
- ERR_SENSOR_TANQUE

Esto permite facilitar:

- Depuración
- Monitoreo
- Trazabilidad
- Mantenimiento del sistema

### Logging de errores

Los errores son registrados mediante mensajes UART utilizando distintos niveles de severidad:

- INFO
- WARN
- ERROR

Ejemplo:
```c
[120] [ERROR] [ERR_01] ERR_SENSOR_LM35: valor fuera de rango
```
Cada mensaje incluye timestamp y código de error correspondiente.

### Estrategia de operación segura

Ante la detección de errores críticos, el sistema toma acciones de protección automática para evitar comportamientos inseguros.

Ejemplo:

- Si el sensor LM35 presenta valores inválidos, la lámpara calefactora es desactivada automáticamente.
```c
gpio_set_level(LAMPARA_GPIO, 0);
```
De esta manera se evita un posible sobrecalentamiento debido a fallos de sensado.

### Continuidad de operación

El sistema fue diseñado para continuar funcionando incluso si uno de los sensores presenta fallos.

Por ejemplo:

- Un error en el sensor de temperatura no detiene el monitoreo de niveles de agua,
- Un error en el bebedero no afecta el sistema de alertas del tanque principal.

Esto incrementa la robustez y tolerancia a fallos del sistema embebido.

### Alertas y monitoreo

El sistema genera alertas automáticas mediante:

- Mensajes seriales
- Activación de buzzer
- Actualización visual en LCD

Esto permite detectar fallos y condiciones críticas en tiempo real.

# Estructura de Directorios del Proyecto y Flujo de Trabajo del Repositorio

El proyecto fue organizado utilizando una estructura modular dentro del repositorio GitHub, permitiendo separar documentación, evidencias, imágenes y firmware del sistema embebido.

La estructura principal del repositorio es la siguiente:

```text
Proyecto_Sistemas_Embebidos/
│
├── images/
│   ├── 12.png
│   ├── Diagrama_estados.png
│   ├── tc1.png
│   └── ...
│
│
├── README.md
├── Entrega_2_Requisitos.md
├── Entrega_3.md
└── Embedded Firmware Design Document.md
```

