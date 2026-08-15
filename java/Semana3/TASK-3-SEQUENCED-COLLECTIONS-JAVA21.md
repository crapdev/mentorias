# TASK 3 — Sequenced Collections con Java 21

# Qué pide la HU

Comparar:

```java
lista.get(0)
lista.get(lista.size() - 1)
```

con:

```java
lista.getFirst()
lista.getLast()
lista.reversed()
```

---

# PASO 0 — Verificar Java 21

Este task necesita Java 21.

En Maven:

```xml
<properties>
    <maven.compiler.release>21</maven.compiler.release>
</properties>
```

Comprueben:

```bash
java -version
mvn -version
```

---

# PASO 1 — Primer elemento con sintaxis Legacy

```java
var primerEmpleadoLegacy = empleados.get(0);
```

Significa:

```text
dame el elemento ubicado en el índice cero
```

---

# PASO 2 — Último con sintaxis Legacy

```java
var ultimoEmpleadoLegacy =
        empleados.get(empleados.size() - 1);
```

Si hay tres elementos:

```text
size = 3
índices = 0, 1, 2
último = 3 - 1
```

---

# PASO 3 — Java 21

```java
var primerEmpleadoModerno = empleados.getFirst();

var ultimoEmpleadoModerno = empleados.getLast();
```

Comparen:

```java
empleados.get(0);
empleados.getFirst();
```

y:

```java
empleados.get(empleados.size() - 1);
empleados.getLast();
```

---

# PASO 4 — Validar lista vacía

```java
if (empleados.isEmpty()) {
    System.out.println("No hay empleados registrados.");
    return;
}
```

Después ya pueden consultar extremos.

---

# PASO 5 — Método de demostración

```java
private static void mostrarOrdenEmpleados(
        List<Empleado> empleados) {

    if (empleados.isEmpty()) {
        System.out.println("No hay empleados registrados.");
        return;
    }

    var primero = empleados.________();

    var ultimo = empleados.________();

    System.out.println(
        "Primer empleado: " + primero.getNombre()
    );

    System.out.println(
        "Último empleado: " + ultimo.getNombre()
    );
}
```

Completen con:

```text
getFirst
getLast
```

---

# PASO 6 — Recorrido normal

```java
System.out.println("
ORDEN NORMAL");

for (var empleado : empleados) {
    System.out.println(empleado.getNombre());
}
```

---

# PASO 7 — `reversed()`

```java
System.out.println("
ORDEN INVERSO");

for (var empleado : empleados.reversed()) {
    System.out.println(empleado.getNombre());
}
```

Así no necesitan:

```java
for (var i = empleados.size() - 1; i >= 0; i--) {
    ...
}
```

---

# PASO 8 — Menú

```text
6. Consultar orden de empleados
```

Conexión:

```java
case 6:
    mostrarOrdenEmpleados(empleados);
    break;
```

---

# PASO 9 — Comparación pedagógica

```java
var primeroLegacy = empleados.get(0);
var ultimoLegacy = empleados.get(empleados.size() - 1);

var primeroModerno = empleados.getFirst();
var ultimoModerno = empleados.getLast();
```

No necesitan conservar ambas formas para siempre; sirven para demostrar la evolución pedida.

---

# Comentario sugerido

```java
/*
 * En Java Legacy los extremos se consultaban mediante índices:
 * get(0) y get(size() - 1).
 *
 * Java 21 incorpora getFirst() y getLast(), que expresan directamente
 * la intención. reversed() permite recorrer la secuencia en sentido inverso.
 */
```

---

# Ejercicio accionable

```java
private static void mostrarOrdenEmpleados(
        List<Empleado> empleados) {

    if (__________________) {
        System.out.println("No hay empleados registrados.");
        return;
    }

    var primero = empleados.________________();

    var ultimo = empleados.________________();

    System.out.println(primero.getNombre());
    System.out.println(ultimo.getNombre());

    for (var empleado : empleados.________________()) {
        System.out.println(empleado.getNombre());
    }
}
```

---

# Prueba

```text
101 - Ana
102 - Bruno
103 - Carlos
```

Esperado:

```text
Primer empleado: Ana
Último empleado: Carlos

ORDEN INVERSO
Carlos
Bruno
Ana
```

---

# Errores frecuentes

- intentar usar estos métodos compilando con Java 17;
- escribir `get(size())`;
- olvidar validar lista vacía;
- crear otra lista manualmente solo para imprimir al revés;
- pensar que `get(index)` deja de existir.

---

# Checklist

- [ ] Compilo con Java 21.
- [ ] Demostré la sintaxis Legacy.
- [ ] Utilicé `getFirst()`.
- [ ] Utilicé `getLast()`.
- [ ] Utilicé `reversed()`.
- [ ] Validé lista vacía.
- [ ] Puedo explicar por qué la sintaxis moderna es más legible.
