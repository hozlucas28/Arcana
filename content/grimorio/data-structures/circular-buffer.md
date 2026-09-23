---
title: Buffer Circular
tags:
  - data-structures
alias:
  - circular queue
  - cyclic buffer
  - ring buffer
---

## 1. Qué es y cómo funciona

### Intuición

Un **buffer circular** es una estructura especial que utiliza un [[struct]] para almacenar dos punteros (**head** y **tail**), un [[array]] y la cantidad de elementos guardados por el array. Su puntero **tail** se utiliza para leer los datos que almacena, mientras que su puntero **head** se utiliza para escribir los datos a guardar. Aunque se comporte como una [[queue]], al llenarse sobrescribe los elementos más antiguos. Esta estructura es útil para almacenar datos de manera temporal, como en la transmisión de datos en tiempo real en sistemas embebidos, donde se requiere un almacenamiento eficiente y rápido de datos que deben ser procesados en orden.

### Definición / propiedades

- Acceso limitado: solo permite acceder al elemento apuntado por tail.
- Tamaño fijo: el tamaño del buffer se define al momento de su creación y no puede cambiarse.
- Respeta el orden de inserción: los elementos se leen en el mismo orden en el que fueron escritos.
- Similitud con queue: se comporta como una cola FIFO, pero con la particularidad de que cuando se llena, los elementos más antiguos se sobrescriben.

### Representación

![Representación del buffer circular](circular-buffer.svg)

- `buffer`: array de tamaño fijo que almacena elementos.
- `length`: cantidad de elementos que el buffer tiene almacenados.
- `head`: puntero a la posición donde se insertara un nuevo elemento en el buffer.
- `tail`: puntero a la posición donde se leerá el elemento del buffer.

## 2. Operaciones y complejidad

### Operaciones principales

- `push(x)` inserta un elemento, en caso de estar lleno, pisa el elemento mas antiguo
- `pop()` retornar elemento mas antiguo
- `peek()` / `top()` consultar elemento mas antiguo
- `isEmpty()` verifica si el buffer no contiene elementos
- `isFull()` verifica si el buffer alcanzo la capacidad máxima de elementos
- `clear()` / `reset()` vacía el buffer y reinicia los punteros

### Complejidad

|                       | Mejor caso | Caso promedio | Peor caso | Espacial |
| --------------------- | ---------- | ------------- | --------- | -------- |
| `push(x)`             | O(1)       | O(1)          | O(1)      | O(1)     |
| `pop()`               | O(1)       | O(1)          | O(1)      | O(1)     |
| `peek()` / `top()`    | O(1)       | O(1)          | O(1)      | O(1)     |
| `isEmpty()`           | O(1)       | O(1)          | O(1)      | O(1)     |
| `isFull()`            | O(1)       | O(1)          | O(1)      | O(1)     |
| `clear()` / `reset()` | O(1)       | O(1)          | O(1)      | O(1)     |
| `find()`              | O(1)       | O(n)          | O(n)      | O(1)     |

### Detalles operativos

- Puede haber underflow (hacer pop en buffer vacía)
- Posee tamaño fijo, el tamaño del buffer se define al momento de su creación y no puede cambiarse.
- Si el buffer se encuentra lleno, se pisa el elemento mas antiguo

## 3. Implementación

### Idea de implementación

Una implementación típica del BufferCircular utiliza un [[array]] de tamaño fijo y 2 punteros índices: `head` para escribir, `tail` para leer.

- `lenght` para mantener la cantidad de elementos almacenados.
- `tail` siempre apunta al elemento más antiguo.
- `head` siempre apunta a la próxima posición de escritura.
- Tanto `head` como `tail` deben avanzar utilizando `% capacity` para nunca salir de los límites del array. Cuando alguno de los índices alcanza el final del array, vuelve a la posición `0` utilizando el operador módulo `%`.

### Invariantes

- `0 <= head < capacity`
- `0 <= tail < capacity`
- `0 <= lenght <= capacity`
- Se inserta siempre en _head_ y se lee siempre de _tail_

### Ejemplo de código

```python
class CircularBuffer:
	def __init__(self, capacity):
		if capacity <= 0:
			raise Exception("Capacity debe ser un entero positivo")

		self.capacity = capacity
		self.buffer = [None] * capacity
		self.lenght = 0
		self.head = 0
		self.tail = 0

	def is_empty(self):
		return self.lenght == 0

	def is_full(self):
		return self.lenght == len(self.buffer)

	def enqueue(self, item):
		if self.is_full():
			self.tail = (self.tail + 1) % self.capacity
		else:
			self.lenght += 1

		self.buffer[self.head] = item
		self.head = (self.head + 1) % self.capacity

	def dequeue(self):
		if self.is_empty():
			raise Exception("Ring vacío")

		item = self.buffer[self.tail]
		# self.buffer[self.tail] = None # conceptualmente no necesario, pero el garbage collector no va a limpiarlo.
		self.tail = (self.tail + 1) % self.capacity
		self.lenght -= 1

		return item

	def peek(self):
		if self.is_empty():
			raise Exception("Ring vacío")
		return self.buffer[self.tail]
```

### Ejemplo de uso típico

```python
# Uso para retener las ultimas 3 señales enviadas por un sensor de temperatura
sensor_temp = CircularBuffer(3)
# Simulando que nuestro sensor recibió 5 señales antes de que lo leamos
for x in [18.5, 19.0, 20.2, 21.5, 22.0]
	sensor_temp.enqueue(x)
# Al leerlo nos encontramos con
while not sensor_temp.is_empty():
    print("dequeue ->", sensor_temp.dequeue())
# 20.2 21.5 22.0
```

```bash
Run started

Initializing environment

Running code

dequeue -> 20.2

dequeue -> 21.5

dequeue -> 22.0

Run completed in 6.199999995529652ms
```

## 4. Uso y criterio

Proporciona una manera de almacenar y administrar datos en un búfer de tamaño fijo como si estuvieran conectados de un extremo a otro, lo cual es una solución elegante para administrar flujos de datos que llegan a velocidades impredecibles o en ráfagas. Esta estructura FIFO (primero en entrar, primero en salir) es particularmente útil en aplicaciones donde el búfer se puede llenar y vaciar a diferentes velocidades, lo que garantiza que los datos más antiguos se procesen primero sin necesidad de una indexación compleja o una mezcla de datos.

### Casos de uso

**Telecomunicaciones y redes**

| Caso de uso                            | Aplicación del buffer circular                                                                                          |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Mitigación del jitter en VoIP          | Almacena temporalmente los paquetes de audio y los entrega a una velocidad constante, reduciendo cortes y distorsiones. |
| Recepción de paquetes de red           | Conserva los paquetes recibidos hasta que el sistema pueda procesarlos y permite absorber ráfagas breves de tráfico.    |
| Comunicación por UART, USB o Bluetooth | Guarda temporalmente los datos recibidos mientras el procesador realiza otras tareas.                                   |

**Audio y video**

| Caso de uso                       | Aplicación del buffer circular                                                                       |
| --------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Streaming en tiempo real          | Compensa variaciones temporales en la velocidad de descarga para mantener una reproducción fluida.   |
| Grabación y reproducción de audio | Permite que un componente escriba muestras nuevas mientras otro procesa o reproduce las anteriores.  |
| Grabación continua                | Conserva los últimos minutos de una cámara y reemplaza automáticamente las grabaciones más antiguas. |

**Sistemas operativos y hardware**

| Caso de uso                     | Aplicación del buffer circular                                                                          |
| ------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Subsistemas de entrada y salida | Compensa las diferencias de velocidad entre los dispositivos periféricos y el procesador.               |
| Controladores de dispositivos   | Almacena datos generados por un dispositivo hasta que el sistema operativo pueda procesarlos.           |
| Acceso directo a memoria (DMA)  | Permite que el hardware escriba datos en una región circular mientras el procesador lee los anteriores. |
| Trazas del sistema              | Conserva los eventos más recientes del sistema para tareas de seguimiento y depuración.                 |

**Sensores y sistemas embebidos**

| Caso de uso                       | Aplicación del buffer circular                                                                            |
| --------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Lecturas de sensores              | Mantiene las últimas N mediciones y reemplaza las más antiguas cuando alcanza la capacidad.               |
| Telemetría                        | Almacena temporalmente datos generados por vehículos, máquinas o dispositivos IoT antes de transmitirlos. |
| Dispositivos con memoria limitada | Mantiene un consumo de memoria fijo y reutiliza continuamente el mismo espacio.                           |

**Procesamiento de datos**

| Caso de uso                      | Aplicación del buffer circular                                                                       |
| -------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Ventanas deslizantes             | Mantiene las últimas N observaciones para calcular promedios móviles, máximos, mínimos o tendencias. |
| Procesamiento digital de señales | Conserva las muestras anteriores necesarias para aplicar filtros y otros cálculos.                   |
| Detección de patrones            | Permite analizar una cantidad limitada de valores recientes para detectar cambios o anomalías.       |

**Programación concurrente**

| Caso de uso                 | Aplicación del buffer circular                                                                       |
| --------------------------- | ---------------------------------------------------------------------------------------------------- |
| Modelo productor-consumidor | El productor incorpora datos y el consumidor los retira en orden para procesarlos.                   |
| Comunicación entre hilos    | Facilita el intercambio de información entre componentes que trabajan de manera concurrente.         |
| Cola limitada de tareas     | Conserva una cantidad fija de trabajos pendientes y aplica una política cuando alcanza su capacidad. |

**Monitoreo y diagnóstico**

| Caso de uso                 | Aplicación del buffer circular                                                               |
| --------------------------- | -------------------------------------------------------------------------------------------- |
| Registro de eventos         | Mantiene los últimos mensajes generados por una aplicación para analizar errores recientes.  |
| Monitoreo de rendimiento    | Conserva las últimas mediciones de CPU, memoria, latencia o solicitudes.                     |
| Registro previo a una falla | Funciona como una caja negra que guarda los eventos inmediatamente anteriores a un problema. |

**Videojuegos y aplicaciones**

| Caso de uso               | Aplicación del buffer circular                                                                 |
| ------------------------- | ---------------------------------------------------------------------------------------------- |
| Procesamiento de entradas | Almacena acciones del teclado, mouse o control hasta que el programa pueda procesarlas.        |
| Historial del jugador     | Conserva las últimas posiciones, movimientos o acciones realizadas.                            |
| Repeticiones instantáneas | Conserva los últimos segundos de la partida para reproducirlos sin guardar la sesión completa. |

### Cuándo NO usarlo

Antes de implementar un buffer circular es importante analizar las necesidades y limitaciones del sistema. Aunque esta estructura se destaca por utilizar eficientemente la memoria y permitir el procesamiento continuo de datos, no resulta adecuada para todos los escenarios. Su capacidad fija, la posible sobrescritura de elementos y sus limitaciones para realizar búsquedas pueden convertirse en desventajas cuando se necesita conservar toda la información o acceder frecuentemente a elementos específicos.

| Situación o requisito                                             | Buffer circular                                                                                  | Alternativa recomendada                                    | Justificación                                                                                                                        |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| No se tolera la pérdida de información                            | No es recomendable si está configurado para sobrescribir los datos más antiguos cuando se llena. | Cola bloqueante o sistema de mensajería persistente        | Evita que los datos pendientes sean reemplazados. El productor puede bloquearse o recibir un error cuando no hay espacio disponible. |
| El volumen de entrada es impredecible                             | Su capacidad fija puede resultar insuficiente ante un aumento repentino de datos.                | Cola dinámica                                              | Puede aumentar su capacidad mientras exista memoria disponible.                                                                      |
| Se procesan transacciones financieras o datos críticos            | No es adecuado si existe la posibilidad de sobrescribir operaciones todavía no procesadas.       | Cola persistente con confirmaciones                        | Permite conservar las operaciones hasta confirmar que fueron procesadas correctamente.                                               |
| Se realizan búsquedas frecuentes por clave                        | Para encontrar un elemento determinado es necesario recorrer el contenido, con un costo de O(n). | [[hash table\|HashMap]]                                    | Permite buscar elementos por clave con una complejidad promedio de O(1).                                                             |
| Se necesita acceso frecuente por una posición arbitraria          | No es su objetivo principal, aunque algunas implementaciones permiten acceder mediante índices.  | [[array\|Arreglo]] o vector                                | Permiten acceder directamente a cualquier posición con una complejidad de O(1).                                                      |
| Se debe conservar un historial completo                           | Su capacidad limitada impide almacenar indefinidamente todos los elementos recibidos.            | Lista dinámica, base de datos o almacenamiento persistente | Permite conservar la información anterior sin reemplazarla.                                                                          |
| Se dispone de memoria limitada y se conoce la capacidad necesaria | Es recomendable.                                                                                 | Buffer circular                                            | Su tamaño fijo permite controlar el uso de memoria y reutilizar el espacio disponible.                                               |
| Solo interesa conservar la información más reciente               | Es recomendable.                                                                                 | Buffer circular con sobrescritura                          | Los elementos antiguos pueden reemplazarse porque se priorizan los datos más nuevos.                                                 |

**Ejemplo práctico: sistema de transacciones financieras**

Supongamos que un banco utiliza un buffer circular para almacenar temporalmente las transferencias que deben ser procesadas. El buffer tiene una capacidad máxima de 100 operaciones y está configurado para sobrescribir el dato más antiguo cuando se llena.

En condiciones normales, las transferencias ingresan al buffer y el sistema las procesa respetando su orden de llegada. Sin embargo, si el procesador presenta una falla o una demora en la conexión, las operaciones comienzan a acumularse. Cuando se alcanza el límite de 100 elementos, cada nueva transferencia sobrescribe una operación anterior que todavía no fue procesada.

Como consecuencia, algunas transferencias podrían perderse sin ser registradas correctamente. Por este motivo, el buffer circular no sería adecuado para este escenario, ya que la pérdida de información financiera es inaceptable.

Una opción más apropiada sería utilizar una cola bloqueante o un sistema de mensajería persistente. Cuando la cola alcanza su capacidad máxima, puede detener temporalmente el ingreso de nuevas operaciones o informar el problema, en lugar de eliminar datos existentes. Además, un sistema persistente conserva las transacciones aunque se produzca una caída del servicio, permitiendo procesarlas cuando el sistema vuelva a funcionar. De esta manera, se prioriza la integridad de la información por encima de la velocidad o del uso limitado de memoria.

### Comparaciones

**vs. cola dinámica basada en [[linked list]] (LinkedList / Queue dinámica)**

- No tiene un límite fijo de capacidad.
- No descarta ni sobreescribe datos viejos.
- La asignación y liberación de memoria dinámica (instanciar nuevos objetos/nodos o llamar a malloc/free) puede generar sobrecarga administrativa.
- Pérdida de localidad espacial: como los nodos se guardan en posiciones dispersas de la memoria, el procesador no puede pre-cargar los datos en su caché de forma eficiente, algo que el buffer circular sí logra gracias a su arreglo contiguo.

**vs. arreglo dinámico redimensionable ([[dynamic array]] / ArrayList)**

- Si se usa como cola extrayendo del principio, un arreglo dinámico obliga a desplazar todos los elementos hacia la izquierda (O(n)).
- Cuando se redimensiona, sufre picos de latencia al reasignar y copiar todo el bloque. El buffer circular opera en O(1) tanto en lectura como en escritura.

**vs. búfer de pila ([[stack]] estático)**

- Mecanismo: almacena los datos en un bloque contiguo, pero extrae siempre el último elemento que fue insertado (política LIFO - _Last In, First Out_).
- Orden de procesamiento: mientras el buffer circular garantiza que los datos más antiguos se procesen primero (ideal para transmisiones en vivo o colas de espera), la pila prioriza el dato más reciente.
- Similitud: ambos pueden implementarse sobre un arreglo estático de tamaño fijo para ser eficientes en memoria y tener complejidad O(1) en sus operaciones.

### Ventajas / desventajas

Ventajas:

- Eficiencia espacial y memoria predecible: al tener un tamaño fijo definido en el momento de su creación, no requiere asignación dinámica de memoria en tiempo de ejecución. Esto evita la fragmentación.
- Complejidad temporal constante: todas las operaciones principales, como añadir (`push`) y eliminar (`pop`), se ejecutan siempre de manera rápida en tiempo O(1).
- En sistemas embebidos y comunicaciones de bajo nivel permite un mejor uso de la memoria, debido al tamaño constante del buffer, para solo utilizar la cantidad de memoria que se necesita.

Desventajas:

- El tamaño fijo también puede constituir una desventaja, ya que cuando el búfer se llena, los datos nuevos sobrescribirán los más antiguos.
- Son difíciles de implementar correctamente en un entorno multihilo o multiproceso.

### Señales de reconocimiento

- **Desacoplar productor y consumidor:** si el problema describe un proceso o dispositivo que genera datos muy rápido (o en ráfagas) y otro que los lee a un ritmo distinto, indicando la necesidad de amortiguar esa diferencia de velocidades sin bloquear el sistema.
- **Memoria estática y tamaño conocido:** cuando se dispone de una cantidad de memoria fija y limitada, y se conoce o puede estimarse de antemano la capacidad máxima necesaria. El espacio utilizado puede reutilizarse continuamente.
- **Operaciones rápidas y predecibles:** cuando se necesitan inserciones y extracciones frecuentes con tiempo de operación O(1).
- **Flujo continuo con backlog acotado:** cuando los datos llegan y se consumen de manera continua y existe un límite razonable para la cantidad de elementos que pueden permanecer pendientes. Esto es habitual en sistemas de streaming, procesamiento en tiempo real y sistemas embebidos.

Las señales de reconocimiento son características del problema que permiten identificar si un buffer circular es una estructura adecuada. En general, conviene utilizarlo cuando los datos llegan continuamente, deben procesarse en orden y existe una cantidad limitada de memoria disponible.

| **Señal identificada en el problema**                            | **¿Por qué indica el uso de un buffer circular?**                                                              |
| ---------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Los datos llegan de manera continua                              | Permite insertar y retirar elementos constantemente sin tener que reorganizar toda la estructura.              |
| Se conoce la capacidad máxima necesaria                          | El buffer se crea con un tamaño fijo, lo que permite controlar el uso de memoria.                              |
| Los elementos deben procesarse en orden FIFO                     | El primer elemento que ingresa es normalmente el primero que se retira.                                        |
| Solo interesa conservar la información más reciente              | Al llenarse, puede configurarse para reemplazar los datos más antiguos por los nuevos.                         |
| Es aceptable perder datos antiguos                               | La sobrescritura no representa un problema cuando la información anterior deja de ser relevante.               |
| Se necesita reutilizar continuamente el mismo espacio de memoria | Cuando se llega al final del arreglo, la siguiente posición vuelve al comienzo sin reservar memoria adicional. |
| Las inserciones y eliminaciones deben ser rápidas                | Las operaciones sobre los extremos pueden realizarse en tiempo constante O(1).                                 |
| Se procesan flujos de audio, video o sensores                    | Estos sistemas generan datos continuamente y, muchas veces, priorizan las muestras más recientes.              |
| Se necesita almacenar un historial reciente y limitado           | Permite conservar únicamente los últimos eventos, registros o mediciones.                                      |
| Productor y consumidor trabajan a velocidades similares          | Reduce la posibilidad de que el buffer se llene y se pierdan elementos pendientes.                             |

**Ejemplo de reconocimiento**

Supongamos que se necesita desarrollar un sistema que muestre las últimas 20 mediciones de temperatura obtenidas por un sensor. El dispositivo genera una nueva medición cada segundo y las mediciones anteriores dejan de ser importantes después de un determinado tiempo.

En este problema aparecen varias señales que indican que conviene utilizar un buffer circular: los datos llegan continuamente, solo se necesita conservar una cantidad fija de elementos y es aceptable reemplazar la medición más antigua cuando ingresa una nueva. Una vez almacenadas las primeras 20 mediciones, cada nuevo valor ocupará el lugar del dato más antiguo.

De esta manera, el sistema mantiene siempre las últimas 20 temperaturas, reutiliza el mismo espacio de memoria y evita que la estructura crezca indefinidamente.

**Conclusión**

El buffer circular resulta conveniente cuando el problema menciona expresiones como "últimos N elementos", "flujo continuo de datos", "memoria limitada", "procesamiento en orden de llegada" o "reemplazar la información más antigua". Estas son las principales pistas que permiten reconocer que la estructura puede resolver el problema de manera eficiente.

## 5. Relaciones y extensiones

### Variantes

- **Con sobrescritura** (la de este artículo): al llenarse reemplaza el más antiguo. Cajas negras, últimas N mediciones.
- **Con rechazo:** al llenarse descarta el nuevo y devuelve un error, cuando el dato pendiente vale más que el nuevo.
- **Bloqueante:** suspende al productor hasta que se libere lugar. Productor-consumidor entre hilos.
- **De doble extremo ([[deque]]):** inserta y extrae por ambos extremos, como `ArrayDeque` de Java.
- **Redimensionadle:** al llenarse duplica la capacidad y copia los elementos; pierde la memoria fija a cambio de no descartar datos.
- **Capacidad potencia de dos:** reemplaza `% capacity` por `& (capacity - 1)`, más barato. Usada en núcleos de sistemas operativos.

### Relación con otras estructuras

En la práctica suele combinarse con otras estructuras: junto a una [[hash table]] forma una caché de tamaño fijo, donde el buffer define qué elemento se descarta y la tabla permite buscar por clave; junto a semáforos constituye el clásico problema del productor-consumidor.
El principio general es que el buffer circular aporta **orden y acotamiento**, y se complementa con otra estructura que aporte la forma de búsqueda que el problema requiera.

### Notas avanzadas

- **Persistencia:** no es una estructura persistente en el sentido funcional, porque modifica la memoria en el lugar y cada sobreescritura destruye información. 
- **Caché y localidad:** el array contiguo favorece la precarga del procesador, pero cuando los datos atraviesan el final del arreglo quedan divididos en dos segmentos, lo que obliga a realizar dos copias de memoria en lugar de una.
- **Ajuste de la capacidad:** debe estimarse como la tasa máxima de producción por el tiempo máximo que el consumidor puede permanecer detenido, con un margen adicional. 
- **Tiempo real:** al no requerir asignación dinámica de memoria, ofrece un tiempo de ejecución acotado y predecible.

### ¿Cómo encaja en el mapa general de estructuras de datos?

Pertenece a las estructuras **lineales de acceso restringido**, junto con la pila ([[stack]]), la cola ([[queue]]) y la [[deque]].

```
Estructuras de datos
├── Lineales
│   ├── De acceso general
│   │   ├── Array                → acceso O(1) por índice, tamaño fijo
│   │   ├── Dynamic array        → contiguo, crece por duplicación
│   │   └── Linked list          → enlazada, sin límite de capacidad
│   └── De acceso restringido
│       ├── Stack (LIFO)
│       ├── Queue (FIFO)
│       │   ├── No acotada       → linked list / dynamic array
│       │   └── Acotada          → BUFFER CIRCULAR
│       └── Deque (ambos extremos) → buffer circular de doble extremo
└── No lineales
    ├── Jerárquicas              → árboles, heaps
    └── Asociativas              → hash tables
```

> Una cola enlazada no acota la memoria; un arreglo la acota pero no puede avanzar el frente sin desplazar. El buffer circular une ambas propiedades con aritmética modular.

Ese compromiso es lo que define a la estructura. El buffer circular no intenta almacenar todo, sino almacenar lo último de la mejor manera posible. Por eso aparece en la frontera entre el software y el hardware —controladores, DMA, comunicación entre hilos, procesamiento de señales—, donde la memoria es limitada, el tiempo de respuesta debe ser predecible y el dato más reciente es el que realmente importa.

## 6. Referencias y recursos

1. [EW Skills - Circular Buffer](https://www.ewskills.com/embedded-c/circular-buffer)
1. [TechVedas .learn - Implementación de un Buffer Circular en C](https://youtu.be/uvD9_Wdtjtw?si=jp2ikM8JuH8_sQtG)
1. [Boost - Circular Buffer](https://www.boost.org/doc/libs/latest/doc/html/circular_buffer.html)
1. [Oracle Java - HashMap](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/HashMap.html)
1. [FasterCapital - Bufer circular: el enfoque FIFO en el almacenamiento de datos](https://fastercapital.com/es/contenido/Bufer-circular--Explicacion-del-bufer-circular--el-enfoque-FIFO-en-el-almacenamiento-de-datos.html#B-fers-circulares-en-aplicaciones-del-mundo-real)
1. [Baeldung - Circular Buffer](https://www.baeldung.com/cs/circular-buffer)
1. [Generalist Programmer - Circular Buffer / Ring Buffer Complete Guide](https://generalistprogrammer.com/tutorials/circular-buffer-ring-buffer-complete-guide)
