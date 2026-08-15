# TASK 4 — `removeIf`, `var` y reporte final

## Objetivo

La última tarea mezcla tres conceptos:

1. filtrado funcional mediante `removeIf`;
2. inferencia de tipos mediante `var`;
3. reporte final con total de empleados y promedio de salarios.

---

# Parte 1 — `removeIf`

## ¿Qué queremos resolver?

La HU pide eliminar empleados que no cumplan un puntaje mínimo.

La clase `Empleado` ya dispone de:

```java
private double promedioDesempeno;
```

y:

```java
public double getPromedioDesempeno()
```

Por lo tanto, podemos filtrar directamente usando ese dato.

---

# 1. Forma manual

Una forma tradicional sería escribir lógica para:

- recorrer;
- comprobar cada promedio;
- decidir cuáles eliminar;
- tener cuidado al modificar la colección mientras la recorremos.

Esto genera bastante código.

---

# 2. `removeIf`

Las colecciones permiten expresar directamente:

```text
elimina los elementos que cumplan esta condición
```

La estructura es:

```java
coleccion.removeIf(elemento -> condicion);
```

Ejemplo conceptual:

```java
empleados.removeIf(
        empleado -> empleado.getPromedioDesempeno() < puntajeMinimo
);
```

Se lee aproximadamente:

> De empleados, elimina si el promedio del empleado es menor que el puntaje mínimo.

---

# 3. Entender la lambda

Esta parte:

```java
empleado -> empleado.getPromedioDesempeno() < puntajeMinimo
```

puede separarse mentalmente así:

```text
empleado
   |
   v
revisar su promedio
   |
   v
¿es menor al mínimo?
```

Si la condición produce:

```text
true
```

el elemento se elimina.

Si produce:

```text
false
```

permanece.

---

# 4. Mucho cuidado con el `HashMap`

En el TASK 1 guardamos empleados en dos estructuras:

```java
List<Empleado> empleados
Map<String, Empleado> empleadosPorId
```

Si usan:

```java
empleados.removeIf(...)
```

solamente se modifica la lista.

El mapa podría conservar empleados que ya fueron filtrados.

Entonces deben pensar cómo mantener ambas estructuras consistentes.

---

# 5. Estrategia sencilla para mantener consistencia

Una opción educativa es:

1. filtrar el `ArrayList`;
2. limpiar el mapa;
3. volver a cargar el mapa con los empleados que sobrevivieron.

Conceptualmente:

```java
empleados.removeIf(...);

empleadosPorId.clear();

for (var empleado : empleados) {
    empleadosPorId.put(
            String.valueOf(empleado.getId()),
            empleado
    );
}
```

Así las dos colecciones vuelven a representar el mismo conjunto de empleados.

Otra solución sería filtrar ambas estructuras cuidadosamente, pero para este ejercicio esta estrategia es fácil de seguir.

---

# 6. Opción extra del menú

Agreguen:

```text
7. Filtrar empleados por desempeño mínimo
```

La opción puede:

1. pedir el puntaje mínimo;
2. aplicar `removeIf`;
3. actualizar el mapa;
4. indicar cuántos empleados permanecieron.

---

# Parte 2 — `var`

## ¿Qué hace `var`?

`var` permite que Java infiera el tipo de una variable local a partir del valor asignado.

Ejemplo:

```java
var salarioTotal = 0.0;
```

Java sabe que es:

```java
double
```

porque el valor inicial es:

```java
0.0
```

---

# 7. Comparación

## Declaración explícita

```java
Empleado empleado = empleados.get(0);
```

## Con inferencia

```java
var empleado = empleados.get(0);
```

El objeto sigue siendo un `Empleado`.

`var` no convierte Java en un lenguaje sin tipos.

El compilador sigue conociendo el tipo.

---

# 8. En bucles

Sin `var`:

```java
for (Empleado empleado : empleados) {
    // ...
}
```

Con `var`:

```java
for (var empleado : empleados) {
    // ...
}
```

En ambos casos `empleado` es tratado como `Empleado`.

---

# 9. ¿Cuándo ayuda?

Puede reducir ruido cuando el tipo ya es evidente:

```java
var empleado = empleados.getFirst();
```

También:

```java
var totalSalarios = 0.0;
```

Pero no significa que deban convertir absolutamente todas las variables a `var`.

La claridad sigue siendo importante.

---

# Parte 3 — Reporte final

La HU pide mostrar:

- total de empleados;
- promedio de salarios calculado desde el `ArrayList`.

---

# 10. Total de empleados

Con arrays utilizábamos una variable manual:

```java
cantidadEmpleados
```

Con `ArrayList`:

```java
empleados.size()
```

Entonces:

```java
var totalEmpleados = empleados.size();
```

---

# 11. Calcular promedio salarial

La fórmula es:

```text
promedio salarial = suma de salarios / número de empleados
```

Primero:

```java
var sumaSalarios = 0.0;
```

Después recorren:

```java
for (var empleado : empleados) {
    sumaSalarios += empleado.getSalario();
}
```

Finalmente:

```java
var promedioSalarios = sumaSalarios / empleados.size();
```

---

# 12. Lista vacía

No hagan:

```java
sumaSalarios / empleados.size()
```

si:

```java
empleados.size() == 0
```

porque no hay empleados sobre los cuales calcular un promedio.

Primero:

```java
if (empleados.isEmpty()) {
    System.out.println("No hay empleados para generar el reporte.");
    return;
}
```

---

# 13. ¿Dónde utilizar `calcularPromedioSalarios`?

Una buena separación es crear un método:

```java
private static double calcularPromedioSalarios(
        List<Empleado> empleados)
```

y utilizarlo desde el método que imprime el reporte final.

Conceptualmente:

```java
var promedioSalarios = calcularPromedioSalarios(empleados);

System.out.println("Total: " + empleados.size());
System.out.println("Promedio salarial: " + promedioSalarios);
```

Así `calcularPromedioSalarios` tiene una única responsabilidad:

> calcular y devolver el promedio.

No necesita imprimir nada.

---

# 14. Diferencia entre calcular y mostrar

### Método de cálculo

```java
calcularPromedioSalarios(...)
```

responsabilidad:

```text
recibir empleados
sumar salarios
calcular promedio
devolver resultado
```

### Método de reporte

responsabilidad:

```text
obtener datos
mostrar total
mostrar promedio
```

Separar ambas responsabilidades hace que el método de cálculo pueda reutilizarse.

---

# 15. Reporte esperado

Después de registrar empleados, podrían mostrar algo parecido a:

```text
=====================================
REPORTE FINAL
=====================================
Total de empleados: 4
Promedio salarial: $3.850.000,00
=====================================
```

No importa que el formato exacto sea diferente.

Lo obligatorio es que los valores se calculen desde el `ArrayList`.

---

# 16. Orden recomendado para implementar el TASK 4

No hagan todo al mismo tiempo.

### Paso A

Asegúrense de que cada empleado ya tiene su:

```java
promedioDesempeno
```

calculado.

### Paso B

Implementen:

```java
removeIf
```

### Paso C

Sincronicen el `HashMap`.

### Paso D

Comprueben que listar empleados ya no muestra los eliminados.

### Paso E

Calculen:

```java
empleados.size()
```

### Paso F

Calculen el promedio salarial.

### Paso G

Muestren el reporte final.

---

# 17. Prueba recomendada

Registren:

```text
Ana    promedio 90
Bruno  promedio 72
Carlos promedio 85
Diana  promedio 60
```

Si filtran con mínimo:

```text
80
```

deberían permanecer conceptualmente:

```text
Ana
Carlos
```

Después comprueben también que:

```text
empleadosPorId
```

ya no encuentre a Bruno ni a Diana.

---

# Checklist del TASK 4

- [ ] Uso `removeIf`.
- [ ] La condición compara `promedioDesempeno` con el mínimo.
- [ ] Después del filtrado, `ArrayList` y `HashMap` permanecen sincronizados.
- [ ] Uso `var` en variables locales donde mejora la lectura.
- [ ] Puedo explicar que `var` sigue teniendo un tipo determinado por el compilador.
- [ ] Obtengo el total mediante `empleados.size()`.
- [ ] Calculo la suma de salarios recorriendo el `ArrayList`.
- [ ] Calculo el promedio salarial.
- [ ] Valido que la lista no esté vacía.
- [ ] Muestro un reporte final.

---

# Cierre de la HU

Si terminaron los cuatro tasks, deberían poder explicar esta evolución:

```text
Empleado[]
    ↓
ArrayList<Empleado>
    ↓
HashMap<String, Empleado>
    ↓
List.of / Map.of
    ↓
getFirst / getLast / reversed
    ↓
removeIf + var
```

La meta real no es aprender métodos aislados.

La meta es empezar a reconocer qué estructura utilizar según el problema que necesitan resolver.
