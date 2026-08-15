# TASK 1 — Migración a `ArrayList` y `HashMap`

# Qué pide la HU

- Reemplazar el array fijo de empleados por `ArrayList`.
- Crear un `HashMap<String, Empleado>`.
- Registrar empleados.
- Listarlos.
- Buscarlos por ID.
- Eliminarlos.

---

# PASO 1 — Cambiar la declaración de empleados

## Antes

En `main` tenían algo parecido a:

```java
var empleados = new Empleado[MAXIMO_EMPLEADOS];
var cantidadEmpleados = 0;
```

Ahora importen:

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
```

Y creen:

```java
List<Empleado> empleados = new ArrayList<>();

Map<String, Empleado> empleadosPorId = new HashMap<>();
```

## Acción

Reemplacen:

```java
Empleado[]
```

por:

```java
List<Empleado>
```

en los métodos que necesiten recibir empleados.

Por ejemplo:

```java
private static boolean registrarEmpleado(
        Scanner scanner,
        List<Empleado> empleados,
        Map<String, Empleado> empleadosPorId) {

    // lógica
}
```

Observen algo importante:

```java
int posicion
```

probablemente ya no sea necesario.

¿Por qué?

Porque con `ArrayList` ya no necesitamos hacer:

```java
empleados[posicion] = empleado;
```

---

# PASO 2 — Crear el empleado

La captura de datos puede seguir casi igual:

```java
System.out.print("ID positivo: ");
var id = scanner.nextInt();
scanner.nextLine();

System.out.print("Nombre: ");
var nombre = scanner.nextLine().trim();

System.out.print("Edad: ");
var edadIngresada = scanner.nextInt();

System.out.print("Salario: ");
var salario = scanner.nextDouble();
scanner.nextLine();
```

Mantengan las validaciones de la semana anterior.

Al final ya pueden crear:

```java
var empleado = new Empleado(
        id,
        nombre,
        (byte) edadIngresada,
        salario
);
```

Pero todavía no lo agreguen.

Primero necesitan validar el ID.

---

# PASO 3 — Reemplazar `idRepetido`

## Antes

Tenían que recorrer los empleados:

```java
for (var indice = 0; indice < cantidadEmpleados; indice++) {
    if (empleados[indice].getId() == idBuscado) {
        return true;
    }
}
```

Con el mapa pueden hacer:

```java
var clave = String.valueOf(id);
```

Y comprobar:

```java
if (empleadosPorId.containsKey(clave)) {
    System.out.println("Ya existe un empleado con ese ID.");
    return false;
}
```

### Acción

Después de esto revisen su proyecto.

Pregúntense:

> ¿Todavía necesito el método `idRepetido()`?

Si ya no tiene ningún uso, elimínenlo.

---

# PASO 4 — Agregar a las colecciones

Una vez validado el empleado, deben almacenarlo en **las dos colecciones**.

Empiecen con:

```java
empleados.add(empleado);
```

Ahora completen el mapa:

```java
empleadosPorId.put(
        ???,
        ???
);
```

Pista:

```text
clave = ID convertido a String
valor = objeto Empleado recién creado
```

---

# PASO 5 — ¿Qué pasa con `cantidadEmpleados`?

Antes:

```java
cantidadEmpleados++;
```

Ahora pueden conocer la cantidad directamente:

```java
empleados.size()
```

Por ejemplo:

```java
System.out.println(
        "Total registrados: " + empleados.size()
);
```

### Acción

Busquen todas las apariciones de:

```java
cantidadEmpleados
```

y revisen cuáles ya no son necesarias.

---

# PASO 6 — Listar empleados

Creen un método:

```java
private static void listarEmpleados(
        List<Empleado> empleados) {

    if (empleados.isEmpty()) {
        System.out.println("No hay empleados registrados.");
        return;
    }

    for (var empleado : empleados) {
        // TODO mostrar ID, nombre y salario
    }
}
```

Pueden utilizar:

```java
empleado.getId()
empleado.getNombre()
empleado.getSalario()
```

Resultado esperado:

```text
ID: 101 | Nombre: Laura | Salario: 3500000
ID: 102 | Nombre: Mateo | Salario: 4200000
```

---

# PASO 7 — Buscar empleado por ID

Creen:

```java
private static void buscarEmpleado(
        Scanner scanner,
        Map<String, Empleado> empleadosPorId) {

    System.out.print("ID del empleado: ");
    var id = scanner.nextInt();
    scanner.nextLine();

    var clave = String.valueOf(id);

    var empleado = empleadosPorId.get(clave);

    if (empleado == null) {
        System.out.println("Empleado no encontrado.");
        return;
    }

    // TODO mostrar datos del empleado encontrado
}
```

Aquí ya no necesitan recorrer el `ArrayList`.

---

# PASO 8 — Eliminar empleado

Firma sugerida:

```java
private static void eliminarEmpleado(
        Scanner scanner,
        List<Empleado> empleados,
        Map<String, Empleado> empleadosPorId) {
```

Comiencen:

```java
System.out.print("ID del empleado a eliminar: ");
var id = scanner.nextInt();
scanner.nextLine();

var clave = String.valueOf(id);
var empleado = empleadosPorId.get(clave);
```

Validen:

```java
if (empleado == null) {
    System.out.println("Empleado no encontrado.");
    return;
}
```

Ahora deben completar:

```java
empleados.???(empleado);
empleadosPorId.???(clave);
```

Pista:

```text
ArrayList -> remove
HashMap   -> remove
```

---

# PASO 9 — Conectar al menú

Pueden agregar:

```java
case 4:
    eliminarEmpleado(
            scanner,
            empleados,
            empleadosPorId
    );
    break;
```

---

# Ejercicio: completar este flujo

```java
var empleado = new Empleado(id, nombre, edad, salario);

var clave = ___________________________;

empleados.________(empleado);

empleadosPorId.________(
        clave,
        empleado
);
```

---

# Comparación final

## Antes

```java
empleados[posicion] = empleado;
cantidadEmpleados++;
```

## Ahora

```java
empleados.add(empleado);
```

y:

```java
empleadosPorId.put(clave, empleado);
```

---

# Prueba manual

Registren:

```text
101 - Laura
102 - Mateo
103 - Andrea
```

Comprueben:

```text
empleados.size() == 3
```

Busquen `102`: debe aparecer Mateo.

Eliminen `102`.

Luego:

```text
empleados.size() == 2
```

y buscar `102` debe indicar que ya no existe.

---

# Errores frecuentes

- seguir usando `empleados[posicion]`;
- mantener `cantidadEmpleados` sin necesidad;
- agregar al `ArrayList` pero olvidar el `HashMap`;
- eliminar solo de una colección;
- usar el `int` directamente cuando el mapa fue declarado `Map<String, Empleado>`;
- seguir recorriendo la lista para buscar IDs aunque ya existe el mapa.

---

# Checklist

- [ ] Declaré `List<Empleado> empleados = new ArrayList<>()`.
- [ ] Declaré `Map<String, Empleado> empleadosPorId = new HashMap<>()`.
- [ ] Reemplacé parámetros `Empleado[]` donde corresponde.
- [ ] Uso `.add()` para registrar.
- [ ] Uso `.containsKey()` para validar IDs.
- [ ] Uso `.get()` para buscar.
- [ ] Uso `.remove()` al eliminar.
- [ ] Mantengo lista y mapa sincronizados.
- [ ] Puedo explicar por qué necesito las dos colecciones.
