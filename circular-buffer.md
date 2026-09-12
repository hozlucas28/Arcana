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

- Notas sobre costos ocultos (reallocs, rehash, recorridos, copias).

### Detalles operativos
- Puede haber underflow (hacer pop en buffer vacía)
- Posee tamaño fijo, el tamaño del buffer se define al momento de su creación y no puede cambiarse. 
- Si el buffer se encuentra lleno, se pisa el v
- Casos especiales: operaciones en estructura vacía/llena, duplicados, orden, límites de tamaño.
- Comportamiento en concurrencia o fallos (si aplica).

Debe responder a: "¿qué puedo hacer y cuánto cuesta?"

## 3. Implementación

### Idea de implementación

- Descripción de la(s) estrategia(s) típica(s) para implementar la estructura.
- Algoritmos clave y pasos principales.

### Invariantes

- Lista de comprobaciones e invariantes que el código debe garantizar siempre (por ejemplo: punteros no nulos, tamaño consistente, heap property, ordenamiento mantenido).

### Ejemplo de código

- Proporciona 1-2 snippets claros y mínimos (en Python).
- Ejemplo de uso típico con entrada y salida esperada.

Debe responder a: "¿cómo lo programo sin romperlo?"

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
