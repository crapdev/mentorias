# TASK 2 — `List.of()` y `Map.of()`

# Qué pide la HU

Crear datos de configuración utilizando:

```java
List.of(...)
Map.of(...)
```

La idea es diferenciarlos de las colecciones dinámicas del TASK 1.

---

# PASO 1 — Crear tecnologías

Import:

```java
import java.util.List;
```

Pueden crear:

```java
List<String> tecnologias = List.of(
        "Java",
        "Spring Boot",
        "PostgreSQL",
        "Git"
);
```

También:

```java
var tecnologias = List.of(
        "Java",
        "Spring Boot",
        "PostgreSQL",
        "Git"
);
```

---

# PASO 2 — Entender la inmutabilidad

Lean:

```java
tecnologias.add("Docker");
```

¿Debería funcionar?

No.

`List.of()` crea una lista inmutable.

Por eso esa línea no debe quedar en la versión final.

---

# PASO 3 — Crear sedes

```java
Map<String, String> sedes = Map.of(
        "BAQ", "Barranquilla",
        "BOG", "Bogotá",
        "MED", "Medellín"
);
```

Conceptualmente:

```text
BAQ -> Barranquilla
BOG -> Bogotá
MED -> Medellín
```

---

# PASO 4 — Consultar una sede

```java
var sede = sedes.get("BAQ");

System.out.println(sede);
```

Resultado:

```text
Barranquilla
```

Ahora piensen:

```java
sedes.put("CLO", "Cali");
```

Tampoco debería funcionar porque `Map.of()` es inmutable.

---

# PASO 5 — Crear una opción del menú

Agreguen:

```text
5. Consultar tecnologías y sedes
```

Método sugerido:

```java
private static void mostrarConfiguracion(
        List<String> tecnologias,
        Map<String, String> sedes) {

    System.out.println("
TECNOLOGÍAS");

    for (var tecnologia : tecnologias) {
        System.out.println("- " + tecnologia);
    }

    System.out.println("
SEDES");

    for (var entrada : sedes.entrySet()) {
        // TODO imprimir código y nombre
    }
}
```

Para completar el segundo `for` revisen:

```java
entrada.getKey()
entrada.getValue()
```

---

# PASO 6 — Comparar con TASK 1

Completen:

```text
empleados
→ cambian durante la ejecución
→ __________________

tecnologias
→ no deberían cambiar
→ __________________

empleadosPorId
→ cambia durante la ejecución
→ __________________

sedes
→ configuración fija
→ __________________
```

Opciones:

```text
ArrayList
HashMap
List.of
Map.of
```

---

# PASO 7 — Comentario solicitado por la HU

```java
/*
 * List.of y Map.of crean colecciones inmutables.
 * Son apropiadas para datos de configuración porque evitan cambios
 * accidentales durante la ejecución.
 *
 * A diferencia de ArrayList y HashMap, no permiten operaciones
 * de modificación como add(), put(), remove() o clear().
 */
```

---

# Mini reto

Agreguen:

```text
Docker
```

### Incorrecto

```java
tecnologias.add("Docker");
```

### Correcto

Modificar la creación:

```java
List.of(
        "Java",
        "Spring Boot",
        "PostgreSQL",
        "Git",
        ???
);
```

---

# Código accionable incompleto

```java
var tecnologias = List.of(
        "Java",
        "Spring Boot",
        ???
);

var sedes = Map.of(
        "BAQ", "Barranquilla",
        "BOG", ???
);

mostrarConfiguracion(
        ???,
        ???
);
```

---

# Errores frecuentes

- usar `new ArrayList<>()` para datos que deberían ser fijos;
- intentar `.add()` sobre `List.of()`;
- intentar `.put()` sobre `Map.of()`;
- confundir el `HashMap` dinámico con el `Map.of()` fijo.

---

# Checklist

- [ ] Utilicé `List.of()` para tecnologías.
- [ ] Utilicé `Map.of()` para sedes.
- [ ] Puedo recorrer ambas colecciones.
- [ ] Entiendo por qué no permiten modificaciones.
- [ ] Agregué un comentario sobre inmutabilidad.
- [ ] Puedo explicar cuándo elegiría `ArrayList` en lugar de `List.of()`.
