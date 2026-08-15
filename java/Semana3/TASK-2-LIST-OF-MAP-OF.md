# TASK 2 — `List.of()` y `Map.of()`

## Objetivo

La segunda tarea introduce los **factory methods** modernos de las colecciones.

La HU pide inicializar datos de configuración como:

- tecnologías;
- sedes.

utilizando:

```java
List.of(...)
```

y:

```java
Map.of(...)
```

---

# 1. Primero: no confundir `ArrayList` con `List.of`

Estas dos líneas crean listas, pero tienen objetivos diferentes.

## Lista dinámica

```java
List<Empleado> empleados = new ArrayList<>();
```

La vamos modificando durante la ejecución:

```java
empleados.add(...);
empleados.remove(...);
```

## Lista fija de configuración

```java
List<String> tecnologias = List.of(
        "Java",
        "Spring Boot",
        "PostgreSQL"
);
```

Está pensada para valores conocidos desde el inicio.

---

# 2. ¿Por qué usar `List.of()` para tecnologías?

Imaginen que las tecnologías permitidas por el sistema son:

```text
Java
Spring Boot
PostgreSQL
Docker
```

Si esas opciones son configuración y no deberían modificarse accidentalmente, podemos declararlas:

```java
var tecnologias = List.of(
        "Java",
        "Spring Boot",
        "PostgreSQL",
        "Docker"
);
```

También podrían usar el tipo explícito:

```java
List<String> tecnologias = List.of(
        "Java",
        "Spring Boot",
        "PostgreSQL",
        "Docker"
);
```

---

# 3. ¿Qué significa que sea inmutable?

Después de crear:

```java
var tecnologias = List.of("Java", "Docker");
```

esto no está permitido:

```java
tecnologias.add("Python");
```

Tampoco:

```java
tecnologias.remove("Java");
```

La colección fue creada para permanecer con esos elementos.

---

# 4. ¿Por qué puede ser más segura para configuración?

Supongamos que utilizamos:

```java
var tecnologias = new ArrayList<String>();
tecnologias.add("Java");
tecnologias.add("Docker");
```

Más adelante otro método podría hacer accidentalmente:

```java
tecnologias.clear();
```

y eliminar toda la configuración.

Con:

```java
List.of(...)
```

la intención queda mucho más clara:

> Estos datos fueron definidos y no deberían modificarse durante la ejecución.

---

# 5. Comentario que pueden agregar al código

La HU pide explicar la diferencia.

No necesitan copiar exactamente este texto, pero la idea debería quedar clara:

```java
/*
 * List.of crea una lista inmutable, adecuada para datos de configuración
 * que no deberían cambiar durante la ejecución. Esto evita modificaciones
 * accidentales mediante add(), remove() o clear().
 *
 * A diferencia de un ArrayList tradicional, esta lista no permite agregar
 * ni eliminar elementos después de su creación.
 */
```

Lo importante no es el comentario exacto.

Lo importante es que sepan explicar **por qué** se eligió `List.of()`.

---

# 6. Utilizar `Map.of()`

Ahora supongamos que las sedes tienen un código:

```text
BAQ -> Barranquilla
BOG -> Bogotá
MED -> Medellín
```

Podemos representarlo como:

```java
var sedes = Map.of(
        "BAQ", "Barranquilla",
        "BOG", "Bogotá",
        "MED", "Medellín"
);
```

El mapa representa:

```text
clave -> valor

BAQ -> Barranquilla
BOG -> Bogotá
MED -> Medellín
```

---

# 7. ¿Por qué un `Map` para sedes?

Porque tenemos una relación natural:

```text
código -> nombre de sede
```

Buscar:

```java
sedes.get("BAQ")
```

obtendría:

```text
Barranquilla
```

Conceptualmente es más expresivo que tener dos listas separadas.

---

# 8. `HashMap` vs `Map.of`

Ambos son mapas, pero los usamos con propósitos diferentes.

## Empleados

```java
Map<String, Empleado> empleadosPorId = new HashMap<>();
```

Debe cambiar constantemente:

```java
put
remove
```

## Sedes

```java
Map<String, String> sedes = Map.of(...);
```

Es configuración fija.

No esperamos registrar ni eliminar sedes durante la ejecución.

---

# 9. Comparación

| Característica | `ArrayList` | `List.of()` |
|---|---|---|
| Se puede agregar | Sí | No |
| Se puede eliminar | Sí | No |
| Uso ideal | Datos dinámicos | Configuración |
| Tamaño cambia | Sí | No |
| Mantiene orden de lista | Sí | Sí |

| Característica | `HashMap` | `Map.of()` |
|---|---|---|
| `put()` | Sí | No |
| `remove()` | Sí | No |
| Uso ideal | Datos dinámicos | Configuración fija |
| Clave → valor | Sí | Sí |

---

# 10. ¿Dónde colocarlas?

Para este ejercicio pueden mantenerlas en `App` si todavía están practicando fundamentos y la estructura del proyecto es pequeña.

Por ejemplo, podrían ser variables dentro de `main`:

```java
var tecnologias = List.of(...);
var sedes = Map.of(...);
```

o constantes de clase si quieren reutilizarlas en varios métodos.

Lo importante para esta HU es practicar correctamente las colecciones.

---

# 11. Opción extra del menú

Pueden agregar:

```text
5. Consultar tecnologías y sedes
```

Y crear un método parecido conceptualmente a:

```java
private static void mostrarConfiguracion(
        List<String> tecnologias,
        Map<String, String> sedes) {
    // recorrer e imprimir
}
```

No copien el método sin analizarlo.

Pregúntense:

- ¿qué datos necesita?
- ¿por qué recibe `List` y `Map`?
- ¿necesita modificar esas colecciones?
- si no las modifica, ¿qué ventaja tiene que sean inmutables?

---

# 12. Ejercicio de comprensión

¿Qué colección usarían para cada caso?

### Caso A

Lista de empleados que entran y salen del sistema.

Respuesta esperada:

```text
ArrayList
```

porque los datos cambian.

### Caso B

Tecnologías oficialmente soportadas por la empresa.

Respuesta esperada:

```text
List.of
```

si son configuración fija.

### Caso C

Empleado identificado por su ID.

Respuesta esperada:

```text
HashMap
```

porque existe una clave única.

### Caso D

Código fijo de sede relacionado con nombre de sede.

Respuesta esperada:

```text
Map.of
```

si la configuración no debe cambiar.

---

# Checklist del TASK 2

- [ ] Creé una lista de tecnologías usando `List.of()`.
- [ ] Entiendo que no puedo usar `.add()` sobre esa lista.
- [ ] Creé un mapa de sedes usando `Map.of()`.
- [ ] Entiendo la diferencia entre el `HashMap` de empleados y el `Map.of()` de configuración.
- [ ] Agregué un comentario explicando por qué las colecciones inmutables son apropiadas para configuración.
- [ ] Puedo mostrar tecnologías y sedes desde una opción de prueba.

Cuando esto funcione, continúen con Java 21.
