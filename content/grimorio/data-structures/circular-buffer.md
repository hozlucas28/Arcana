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
- Tanto `head` como `tail` deben avanzar utilizando `% capacity` para nunca salir de los límites del array.
Cuando alguno de los índices alcanza el final del array, vuelve a la posición `0` utilizando el operador módulo `%`.
### Invariantes

- `0 <= head < capacity`
- `0 <= tail < capacity`
- `0 <= lenght <= capacity`
- Se inserta siempre en *head* y se lee siempre de *tail*

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

### Casos de uso

- Situaciones y problemas donde la estructura encaja naturalmente.

### Cuándo NO usarlo

- Escenarios donde su uso es contraproducente o subóptimo.

### Comparaciones

- Alternativas comunes y cuándo elegir cada una (lista comparativa breve).

### Ventajas / desventajas

- Trade-offs prácticos en rendimiento, memoria, simplicidad, y facilidad de implementación.

### Señales de reconocimiento

- Pistas en el enunciado de un problema que indican que esta estructura es adecuada.

Debe responder a: "¿cuándo conviene usarlo?"

## 5. Relaciones y extensiones

### Variantes

- Variantes y mejoras (por ejemplo: versiones balanceadas, persistentes, acotadas, indexadas, con hashing, etc.).

### Relación con otras estructuras

- Dependencias conceptuales y cómo se combina con otras estructuras.

### Notas avanzadas

- Temas avanzados como persistencia, concurrencia, paralelismo, ordenamientos aleatorios, caching, tuning de parámetros.

Debe responder a: "¿cómo encaja en el mapa general de estructuras de datos?"

## 6. Referencias y recursos

- Enlaces y libros de referencia, artículos científicos.
- Visualizaciones y demostraciones.
