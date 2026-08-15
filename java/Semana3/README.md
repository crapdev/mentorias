# Corporate Talent Hub — Guía práctica de la HU de Colecciones

Esta guía acompaña la evolución del proyecto de la semana anterior hacia el uso de colecciones modernas de Java.

> **No es una solución para copiar y pegar.**
>
> Encontrarán código suficiente para saber **qué deben modificar y cómo comenzar**, pero varias partes quedan intencionalmente incompletas para que ustedes implementen la lógica.

---

# Punto de partida

El proyecto parte de:

```text
corporate-talent-hub-control-flujo/
├── pom.xml
└── src/
    └── main/
        └── java/
            └── com/
                └── corporatetalenthub/
                    ├── App.java
                    └── modelo/
                        └── Empleado.java
```

La clase `Empleado` ya existe:

```java
public class Empleado {

    private final int id;
    private final String nombre;
    private final byte edad;
    private final double salario;
    private double promedioDesempeno;

    // constructor, getters y setter...
}
```

En la semana anterior los empleados se guardaban así:

```java
var empleados = new Empleado[MAXIMO_EMPLEADOS];
var cantidadEmpleados = 0;
```

Esta HU busca eliminar progresivamente ese manejo manual.

---

# Orden de trabajo

Trabajen un archivo por vez:

1. [`TASK-1-ARRAYLIST-HASHMAP.md`](TASK-1-ARRAYLIST-HASHMAP.md)
2. [`TASK-2-LIST-OF-MAP-OF.md`](TASK-2-LIST-OF-MAP-OF.md)
3. [`TASK-3-SEQUENCED-COLLECTIONS-JAVA21.md`](TASK-3-SEQUENCED-COLLECTIONS-JAVA21.md)
4. [`TASK-4-REMOVEIF-VAR-REPORTE.md`](TASK-4-REMOVEIF-VAR-REPORTE.md)

Cada guía tiene esta estructura:

```text
1. Qué pide la HU
2. Qué parte del código anterior cambia
3. Código de partida
4. Código que deben construir
5. Huecos para completar
6. Prueba manual
7. Errores frecuentes
8. Checklist
```

---

# Colecciones de esta HU

## Array

```java
Empleado[] empleados = new Empleado[50];
```

Tamaño fijo.

## ArrayList

```java
List<Empleado> empleados = new ArrayList<>();
```

Tamaño dinámico.

## HashMap

```java
Map<String, Empleado> empleadosPorId = new HashMap<>();
```

Relaciona una clave única con un empleado.

```text
"101" -> Empleado
"102" -> Empleado
```

## List.of

```java
List<String> tecnologias = List.of(
        "Java",
        "Spring Boot",
        "PostgreSQL"
);
```

Lista inmutable para datos de configuración.

## Map.of

```java
Map<String, String> sedes = Map.of(
        "BAQ", "Barranquilla",
        "BOG", "Bogotá"
);
```

Mapa inmutable para relaciones fijas.

---

# ¿Por qué escribir esto?

```java
List<Empleado> empleados = new ArrayList<>();
```

y no simplemente:

```java
ArrayList<Empleado> empleados = new ArrayList<>();
```

Las dos formas funcionan.

Pero:

```java
List<Empleado>
```

indica la **abstracción con la que queremos trabajar**.

Mientras:

```java
new ArrayList<>()
```

indica la **implementación concreta que estamos creando**.

Pueden leerlo como:

```text
Quiero una LISTA de empleados
implementada actualmente con ARRAYLIST.
```

Eso permite cambiar después:

```java
List<Empleado> empleados = new LinkedList<>();
```

sin cambiar el tipo usado por el resto del código.

---

# Menú sugerido para practicar

No es obligatorio que todas estas opciones formen parte de la entrega final, pero son útiles para demostrar cada criterio:

```text
1. Registrar empleado
2. Mostrar reporte
3. Consultar categorías salariales
4. Eliminar empleado
5. Consultar tecnologías y sedes
6. Consultar orden de empleados
7. Filtrar empleados por desempeño mínimo
8. Generar reporte final
0. Salir
```

---

# Regla para trabajar esta guía

Cuando encuentren:

```java
// TODO
```

o:

```java
???
```

esa parte queda para ustedes.

La guía les mostrará suficiente contexto para saber qué debe ir allí, pero sin entregar el método terminado.
