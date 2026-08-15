# TASK 3 — Sequenced Collections en Java 21

## Objetivo

La tercera tarea compara la forma tradicional de trabajar con los extremos de una lista con las APIs incorporadas en Java 21.

La HU solicita utilizar:

```java
getFirst()
getLast()
reversed()
```

sobre la colección de empleados.

---

# 1. Antes de Java 21

Si tenemos:

```java
List<Empleado> empleados = new ArrayList<>();
```

tradicionalmente obteníamos el primero usando:

```java
empleados.get(0);
```

Y el último:

```java
empleados.get(empleados.size() - 1);
```

Esto funciona.

No está mal.

Pero obliga al programador a pensar en índices.

---

# 2. El problema del último índice

En una lista de tamaño:

```text
5
```

los índices son:

```text
0
1
2
3
4
```

Por eso el último es:

```java
size() - 1
```

Un error típico sería escribir:

```java
empleados.get(empleados.size());
```

Eso intenta acceder a una posición que no existe.

---

# 3. Java 21

Con las Sequenced Collections, podemos expresar directamente nuestra intención.

## Primer empleado

```java
var primero = empleados.getFirst();
```

## Último empleado

```java
var ultimo = empleados.getLast();
```

Ahora el código dice exactamente lo que queremos hacer.

No necesitamos calcular índices.

---

# 4. Comparación de legibilidad

## Legacy

```java
var primero = empleados.get(0);
var ultimo = empleados.get(empleados.size() - 1);
```

## Java 21

```java
var primero = empleados.getFirst();
var ultimo = empleados.getLast();
```

La versión moderna expresa directamente:

```text
primero
último
```

en lugar de:

```text
índice cero
tamaño menos uno
```

---

# 5. Importante: lista vacía

Antes de obtener el primero o el último deben verificar que existan empleados.

```java
if (empleados.isEmpty()) {
    System.out.println("No hay empleados registrados.");
    return;
}
```

Después:

```java
var primero = empleados.getFirst();
var ultimo = empleados.getLast();
```

`getFirst()` y `getLast()` evitan tener que calcular manualmente índices, pero no significa que puedan obtener elementos de una lista vacía.

---

# 6. `reversed()`

Java 21 también permite obtener una vista en orden inverso:

```java
var empleadosInvertidos = empleados.reversed();
```

Si la lista original conceptualmente es:

```text
[Ana, Bruno, Carlos, Diana]
```

la vista inversa se recorre como:

```text
[Diana, Carlos, Bruno, Ana]
```

---

# 7. ¿Por qué no necesitamos hacer un algoritmo manual?

Sin esta operación podríamos pensar en:

- recorrer desde `size() - 1` hasta `0`;
- crear otra lista;
- intercambiar posiciones;
- utilizar otras utilidades.

Pero para simplemente **consultar la secuencia en sentido contrario**, `reversed()` expresa directamente la intención.

Ejemplo:

```java
for (var empleado : empleados.reversed()) {
    System.out.println(empleado.getNombre());
}
```

---

# 8. `reversed()` y la lista original

Para esta HU piensen en `reversed()` como una **vista invertida de la secuencia**.

No necesitan crear manualmente otro `ArrayList` solo para imprimir los empleados al revés.

Esto reduce código accidental y hace más evidente la intención.

---

# 9. Opción extra del menú

Agreguen:

```text
6. Consultar orden de empleados
```

Esa opción puede mostrar:

```text
Primer empleado
Último empleado
Lista normal
Lista invertida
```

Esto les permite demostrar claramente el criterio de aceptación de Java 21.

---

# 10. Método sugerido como ejercicio

Pueden diseñar un método con esta responsabilidad:

```java
private static void mostrarOrdenEmpleados(List<Empleado> empleados)
```

Dentro, ustedes deben decidir:

1. cómo validar lista vacía;
2. cómo obtener el primero;
3. cómo obtener el último;
4. cómo recorrer el orden normal;
5. cómo recorrer el orden inverso.

---

# 11. Comentario sobre evolución Java

La HU pide comentar la mejora.

Un comentario posible sería:

```java
/*
 * En versiones Legacy el primer y último elemento se obtenían mediante
 * índices: get(0) y get(size() - 1). Java 21 permite expresar directamente
 * la intención mediante getFirst() y getLast(), reduciendo cálculos manuales
 * de índices. reversed() permite recorrer la secuencia en sentido contrario
 * sin implementar manualmente el recorrido inverso.
 */
```

No memoricen el comentario.

Entiendan la idea:

```text
menos manipulación de índices
+
mayor expresividad
+
menos posibilidades de escribir mal el último índice
```

---

# 12. ¿Significa que `get(index)` dejó de servir?

No.

Todavía necesitamos:

```java
get(indice)
```

cuando realmente queremos una posición específica.

Por ejemplo:

```java
empleados.get(3);
```

`getFirst()` y `getLast()` resuelven únicamente casos donde nuestro objetivo es específicamente trabajar con los extremos de la secuencia.

---

# 13. Comprobación manual

Registren tres empleados:

```text
101 - Ana
102 - Bruno
103 - Carlos
```

La opción debería permitir comprobar:

```text
Primero: Ana
Último: Carlos
```

Orden normal:

```text
Ana
Bruno
Carlos
```

Orden inverso:

```text
Carlos
Bruno
Ana
```

---

# Checklist del TASK 3

- [ ] Mi proyecto está configurado para compilar con Java 21.
- [ ] Verifico `isEmpty()` antes de consultar extremos.
- [ ] Utilizo `getFirst()`.
- [ ] Utilizo `getLast()`.
- [ ] Utilizo `reversed()`.
- [ ] Puedo explicar cómo se hacía antes con índices.
- [ ] Agregué un comentario comparando Legacy con Java 21.
- [ ] Puedo demostrar el comportamiento desde una opción del menú.
