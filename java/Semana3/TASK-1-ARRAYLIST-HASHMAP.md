# TASK 1 — Migración a `ArrayList` y `HashMap`

## Objetivo

La primera tarea pide reemplazar el almacenamiento fijo de empleados por colecciones dinámicas.

La HU solicita:

- sustituir `Empleado[]` por un `ArrayList<Empleado>`;
- implementar un `HashMap<String, Empleado>`;
- agregar empleados;
- listar empleados;
- eliminar empleados;
- buscar empleados por ID.

---

# 1. ¿Qué problema tenemos actualmente?

En el código anterior aparece algo parecido a:

```java
var empleados = new Empleado[MAXIMO_EMPLEADOS];
```

Un array necesita conocer su tamaño desde el comienzo.

Si fue creado con 50 posiciones:

```java
new Empleado[50]
```

siempre tendrá exactamente 50 posiciones.

No puede convertirse automáticamente en uno de 51.

Por eso antes también necesitábamos:

```java
var cantidadEmpleados = 0;
```

y código como:

```java
empleados[posicion] = nuevoEmpleado;
cantidadEmpleados++;
```

Estamos administrando manualmente:

- la posición;
- la cantidad;
- el límite;
- las posiciones vacías.

---

# 2. Migrar a `ArrayList`

Una forma recomendada de declararlo es:

```java
List<Empleado> empleados = new ArrayList<>();
```

Imports:

```java
import java.util.ArrayList;
import java.util.List;
```

## ¿Qué significa?

### Parte izquierda

```java
List<Empleado> empleados
```

La variable acepta una lista cuyos elementos sean objetos `Empleado`.

### Parte derecha

```java
new ArrayList<>()
```

Creamos realmente un `ArrayList` vacío.

Inicialmente:

```text
empleados = []
```

Cuando hacemos:

```java
empleados.add(empleado);
```

la lista aumenta dinámicamente.

---

# 3. ¿Qué ventaja tiene frente al array?

### Antes

```java
Empleado[] empleados = new Empleado[50];
```

Debíamos decidir el tamaño desde el inicio.

### Ahora

```java
List<Empleado> empleados = new ArrayList<>();
```

Podemos ir agregando:

```java
empleados.add(empleado1);
empleados.add(empleado2);
empleados.add(empleado3);
```

sin administrar manualmente una posición.

Para conocer cuántos empleados existen:

```java
empleados.size()
```

Por lo tanto, probablemente ya no necesitemos:

```java
cantidadEmpleados
```

ni:

```java
MAXIMO_EMPLEADOS
```

si la nueva HU ya no exige un máximo.

---

# 4. Métodos básicos de `ArrayList`

## Agregar

```java
empleados.add(empleado);
```

## Obtener por posición

```java
empleados.get(0);
```

## Tamaño

```java
empleados.size();
```

## Eliminar un objeto

```java
empleados.remove(empleado);
```

## Comprobar si está vacía

```java
empleados.isEmpty();
```

## Recorrer

```java
for (var empleado : empleados) {
    System.out.println(empleado.getNombre());
}
```

---

# 5. ¿Por qué también necesitamos un `HashMap`?

El `ArrayList` es excelente para:

- mantener empleados en una secuencia;
- recorrerlos;
- obtenerlos por posición.

Pero buscar un empleado por ID normalmente requiere recorrer la lista:

```java
for (var empleado : empleados) {
    if (empleado.getId() == idBuscado) {
        // encontrado
    }
}
```

La HU pide utilizar también:

```java
HashMap<String, Empleado>
```

La idea es guardar:

```text
CLAVE  -> VALOR

"101"  -> empleadoCristian
"102"  -> empleadoLaura
"103"  -> empleadoMateo
```

---

# 6. Declarar el mapa

Una forma recomendable:

```java
Map<String, Empleado> empleadosPorId = new HashMap<>();
```

Imports:

```java
import java.util.HashMap;
import java.util.Map;
```

Aquí ocurre exactamente la misma idea que con `List`:

```java
Map<String, Empleado>
```

es el contrato con el que trabajamos.

```java
new HashMap<>()
```

es la implementación concreta.

---

# 7. Pero nuestro ID es `int`

La clase `Empleado` actualmente tiene:

```java
private final int id;
```

La HU pide específicamente:

```java
HashMap<String, Empleado>
```

Por eso, al utilizar el ID como clave, pueden convertirlo:

```java
String.valueOf(empleado.getId())
```

Ejemplo:

```java
empleadosPorId.put(String.valueOf(empleado.getId()), empleado);
```

Al buscar:

```java
var empleado = empleadosPorId.get(String.valueOf(idBuscado));
```

> Si la HU permitiera cambiar el tipo de la clave, `Map<Integer, Empleado>` sería también una opción natural porque el ID del modelo es `int`. Pero si el criterio exige `HashMap<String, Empleado>`, respeten ese requisito.

---

# 8. Registrar un empleado en las dos colecciones

Cuando creen:

```java
var empleado = new Empleado(id, nombre, edad, salario);
```

tienen que mantener sincronizadas ambas estructuras.

Conceptualmente:

```java
empleados.add(empleado);
empleadosPorId.put(String.valueOf(empleado.getId()), empleado);
```

El mismo objeto `Empleado` queda referenciado desde:

```text
ArrayList
   |
   +----> Empleado

HashMap
   |
   +----> misma instancia de Empleado
```

No se crean dos empleados diferentes.

---

# 9. Validar ID repetido usando el mapa

Antes existía un método parecido a:

```java
idRepetido(...)
```

que recorría todo el array.

Ahora el mapa puede utilizarse para preguntar si la clave ya existe:

```java
empleadosPorId.containsKey(String.valueOf(id))
```

Ejemplo conceptual:

```java
if (empleadosPorId.containsKey(String.valueOf(id))) {
    System.out.println("Ya existe un empleado con ese ID.");
    return false;
}
```

Esto hace que el propósito del código sea mucho más evidente:

> ¿Existe esta clave?

---

# 10. Buscar por ID

Con el mapa:

```java
var empleado = empleadosPorId.get(String.valueOf(idBuscado));
```

Si no existe esa clave:

```java
empleado == null
```

Entonces pueden hacer:

```java
if (empleado == null) {
    System.out.println("Empleado no encontrado.");
}
```

---

# 11. Eliminar empleados

Aquí hay un detalle importante.

Como están utilizando **dos colecciones**, eliminar de una sola dejaría datos inconsistentes.

Si eliminan únicamente:

```java
empleados.remove(empleado);
```

el `HashMap` todavía conservaría la referencia.

Si eliminan únicamente del mapa, el empleado seguiría apareciendo al listar el `ArrayList`.

Por eso deben eliminarlo de ambas.

Conceptualmente:

```java
var clave = String.valueOf(idBuscado);
var empleado = empleadosPorId.get(clave);

if (empleado != null) {
    empleados.remove(empleado);
    empleadosPorId.remove(clave);
}
```

---

# 12. Array vs ArrayList

| Característica | Array | ArrayList |
|---|---|---|
| Tamaño | Fijo | Dinámico |
| Agregar | Asignando posición | `.add()` |
| Eliminar | Manual | `.remove()` |
| Tamaño usado | Debemos controlarlo | `.size()` |
| Métodos de colección | Muy pocos | Muchos |
| Acceso por índice | Sí | Sí |

---

# 13. Ventajas y desventajas de `ArrayList`

## Ventajas

- tamaño dinámico;
- mantiene orden de inserción;
- acceso sencillo mediante índice;
- gran cantidad de métodos;
- compatible con APIs modernas de colecciones.

## Desventajas

- buscar por un atributo como ID requiere recorrerla si no tenemos otra estructura;
- insertar o eliminar en ciertas posiciones puede implicar mover elementos;
- permite duplicados.

Por eso en esta HU se complementa con un `HashMap`.

---

# 14. Ventajas y desventajas de `HashMap`

## Ventajas

- pensado para trabajar con relaciones clave → valor;
- una clave es única;
- búsqueda directa mediante la clave;
- muy útil para IDs, códigos, usernames, etc.

## Desventajas

- no debe utilizarse esperando un orden de inserción;
- cada elemento necesita una clave;
- si mantenemos el mismo dato también en una lista, debemos mantener ambas estructuras sincronizadas.

---

# Reto de implementación

Intenten modificar primero solamente estas partes:

1. declaración de empleados;
2. registro;
3. validación del ID;
4. listado;
5. eliminación.

No implementen todavía `List.of`, `getFirst` ni `removeIf`.

---

# Checklist del TASK 1

- [ ] Reemplacé `Empleado[]` por `List<Empleado>`.
- [ ] La implementación creada es un `ArrayList`.
- [ ] Eliminé el manejo manual de posiciones que ya no necesito.
- [ ] Creé un `Map<String, Empleado>`.
- [ ] La implementación creada es un `HashMap`.
- [ ] Al registrar agrego el empleado a ambas colecciones.
- [ ] No permito IDs repetidos.
- [ ] Puedo buscar por ID usando el mapa.
- [ ] Al eliminar, retiro el empleado de ambas colecciones.
- [ ] Puedo listar los empleados recorriendo el `ArrayList`.

Cuando todo esto funcione, continúen con el TASK 2.
