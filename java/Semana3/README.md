# Corporate Talent Hub — Guía de la HU de Colecciones

Esta guía está pensada para resolver **paso a paso** la HU de evolución de colecciones desde Java Legacy hasta Java 21.

La idea no es copiar una solución terminada. El objetivo es entender:

- qué problema tiene el código de la semana anterior;
- qué colección resuelve mejor cada necesidad;
- qué cambia entre Java 8, Java 11 y Java 21;
- por qué se usa cada estructura;
- y cómo comprobar que la implementación funciona.

---

## Punto de partida

El proyecto parte aproximadamente de esta estructura:

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

En la versión anterior los empleados se almacenaban en:

```java
var empleados = new Empleado[MAXIMO_EMPLEADOS];
```

Eso significa que se está usando un **array de tamaño fijo**.

También se utilizaba:

```java
var cantidadEmpleados = 0;
```

para controlar manualmente cuántas posiciones del array estaban ocupadas.

La nueva HU busca reemplazar parte de ese manejo manual por las APIs de colecciones de Java.

---

# Orden recomendado

Trabajen los archivos en este orden:

1. [`TASK-1-ARRAYLIST-HASHMAP.md`](TASK-1-ARRAYLIST-HASHMAP.md)
2. [`TASK-2-LIST-OF-MAP-OF.md`](TASK-2-LIST-OF-MAP-OF.md)
3. [`TASK-3-SEQUENCED-COLLECTIONS-JAVA21.md`](TASK-3-SEQUENCED-COLLECTIONS-JAVA21.md)
4. [`TASK-4-REMOVEIF-VAR-REPORTE.md`](TASK-4-REMOVEIF-VAR-REPORTE.md)

No intenten implementar los cuatro tasks al mismo tiempo.

---

# Mapa mental de las estructuras usadas en esta HU

| Estructura | Para qué la usamos | Tamaño | Orden | Se modifica |
|---|---|---:|---|---|
| `Empleado[]` | Código anterior | Fijo | Sí | Sí |
| `ArrayList<Empleado>` | Lista dinámica de empleados | Dinámico | Sí | Sí |
| `List.of(...)` | Configuración fija | Fijo | Sí | No |
| `HashMap<String, Empleado>` | Buscar empleados por ID | Dinámico | No confiar en orden | Sí |
| `Map.of(...)` | Configuración clave → valor | Fijo | No confiar en orden | No |

---

# Algo importante: `List` y `ArrayList` no significan exactamente lo mismo

Van a encontrar declaraciones como:

```java
List<Empleado> empleados = new ArrayList<>();
```

Aquí hay dos cosas diferentes.

### `List<Empleado>`

Es el **tipo de la variable**.

Dice:

> Esta variable trabajará con cualquier implementación que cumpla el contrato de una lista.

### `new ArrayList<>()`

Es el **objeto concreto que realmente se crea en memoria**.

Entonces:

```java
List<Empleado> empleados = new ArrayList<>();
```

se puede leer como:

> Quiero trabajar con una lista de empleados y, por ahora, voy a implementarla usando un `ArrayList`.

También sería válido:

```java
ArrayList<Empleado> empleados = new ArrayList<>();
```

Pero normalmente se prefiere:

```java
List<Empleado> empleados = new ArrayList<>();
```

porque el código queda menos acoplado a una implementación concreta.

Por ejemplo, más adelante podríamos cambiar:

```java
List<Empleado> empleados = new LinkedList<>();
```

sin cambiar el tipo de la variable.

---

# ¿Por qué no usar siempre una sola colección?

Porque cada estructura resuelve un problema diferente.

Si necesitan:

- recorrer empleados en orden → `ArrayList`;
- buscar rápidamente por una clave → `HashMap`;
- guardar opciones que nunca deben cambiar → `List.of`;
- relacionar una clave fija con un valor fijo → `Map.of`.

Elegir una colección forma parte del diseño del programa.

---

# Opciones extra del menú

Además de las opciones originales, pueden agregar estas opciones para practicar la HU:

```text
5. Consultar tecnologías y sedes
6. Consultar orden de empleados
7. Filtrar empleados por desempeño mínimo
```

Estas opciones no tienen que convertirse necesariamente en nuevas funcionalidades complejas.

La idea es utilizarlas para **probar de forma visible** lo que pide cada task.

Por ejemplo:

- opción 5 → comprobar `List.of()` y `Map.of()`;
- opción 6 → comprobar `getFirst()`, `getLast()` y `reversed()`;
- opción 7 → comprobar `removeIf()`.

---

# Recomendación de trabajo

Antes de escribir código, para cada task respondan:

1. ¿Qué problema del código anterior estoy eliminando?
2. ¿Qué colección nueva voy a utilizar?
3. ¿Por qué esa colección es adecuada?
4. ¿Qué código anterior ya no necesito?
5. ¿Cómo voy a comprobar que funciona?

Si pueden responder esas cinco preguntas, probablemente entienden lo que están implementando y no solo están copiando código.
