# TASK 4 — `removeIf`, `var` y reporte final

# Qué pide la HU

- Eliminar empleados que no cumplan un desempeño mínimo.
- Utilizar `removeIf`.
- Demostrar `var`.
- Mostrar total de empleados.
- Calcular promedio salarial desde el `ArrayList`.

---

# PASO 1 — Asegurar que exista el promedio

La clase ya tiene:

```java
private double promedioDesempeno;
```

y:

```java
public void setPromedioDesempeno(
        double promedioDesempeno) {

    this.promedioDesempeno = promedioDesempeno;
}
```

Cuando calculen el desempeño deben guardar:

```java
empleado.setPromedioDesempeno(promedio);
```

Si no hacen esto, después `removeIf` no tendrá un valor útil para filtrar.

---

# PASO 2 — Entender `removeIf`

La estructura general:

```java
coleccion.removeIf(
        elemento -> condicion
);
```

Para empleados:

```java
empleados.removeIf(
        empleado ->
                empleado.getPromedioDesempeno()
                        < puntajeMinimo
);
```

---

# PASO 3 — Capturar el mínimo

```java
private static void filtrarPorDesempeno(
        Scanner scanner,
        List<Empleado> empleados,
        Map<String, Empleado> empleadosPorId) {

    System.out.print("Puntaje mínimo requerido: ");

    var puntajeMinimo = scanner.nextDouble();
    scanner.nextLine();

    // TODO validar 0 - 100
}
```

Validación reutilizable:

```java
if (puntajeMinimo < NOTA_MINIMA
        || puntajeMinimo > NOTA_MAXIMA) {

    System.out.println(
            "El puntaje debe estar entre 0 y 100."
    );

    return;
}
```

---

# PASO 4 — Saber cuántos fueron eliminados

Antes:

```java
var cantidadAntes = empleados.size();
```

Filtran:

```java
empleados.removeIf(
        empleado ->
                empleado.getPromedioDesempeno()
                        < puntajeMinimo
);
```

Después:

```java
var cantidadDespues = empleados.size();
```

Completen:

```java
var eliminados =
        __________________ - __________________;
```

---

# PASO 5 — Sincronizar el HashMap

Después del `removeIf`, el mapa podría seguir teniendo empleados eliminados.

Primero:

```java
empleadosPorId.clear();
```

Luego:

```java
for (var empleado : empleados) {

    var clave =
            String.valueOf(empleado.getId());

    empleadosPorId.put(
            clave,
            empleado
    );
}
```

---

# PASO 6 — Método incompleto

```java
private static void filtrarPorDesempeno(
        Scanner scanner,
        List<Empleado> empleados,
        Map<String, Empleado> empleadosPorId) {

    System.out.print("Puntaje mínimo: ");
    var puntajeMinimo = scanner.nextDouble();
    scanner.nextLine();

    if (__________________________________) {
        System.out.println("Puntaje inválido.");
        return;
    }

    var cantidadAntes = empleados.size();

    empleados.removeIf(
            empleado ->
                    __________________________________
    );

    var cantidadDespues = empleados.size();

    var eliminados =
            _________________________________;

    empleadosPorId.clear();

    for (var empleado : empleados) {
        empleadosPorId.put(
                ________________________________,
                empleado
        );
    }

    System.out.println(
            "Empleados eliminados: " + eliminados
    );
}
```

---

# PASO 7 — Comparar tipo explícito y `var`

Java 8:

```java
Empleado empleado = empleados.get(0);
double sumaSalarios = 0.0;
int totalEmpleados = empleados.size();
```

Con `var`:

```java
var empleado = empleados.get(0);
var sumaSalarios = 0.0;
var totalEmpleados = empleados.size();
```

El compilador sigue sabiendo:

```text
empleado       -> Empleado
sumaSalarios   -> double
totalEmpleados -> int
```

---

# PASO 8 — Calcular promedio salarial

Firma:

```java
private static double calcularPromedioSalarios(
        List<Empleado> empleados) {
```

Comiencen:

```java
if (empleados.isEmpty()) {
    return 0.0;
}

var sumaSalarios = 0.0;

for (var empleado : empleados) {
    sumaSalarios += _______________________;
}

return sumaSalarios / _______________________;
```

Pistas:

```text
getSalario()
size()
```

---

# PASO 9 — Reporte final

```java
private static void mostrarReporteFinal(
        List<Empleado> empleados) {

    if (empleados.isEmpty()) {
        System.out.println(
                "No hay empleados para generar el reporte."
        );
        return;
    }

    var totalEmpleados = empleados.size();

    var promedioSalarios =
            calcularPromedioSalarios(empleados);

    System.out.println(
            "============================="
    );
    System.out.println("REPORTE FINAL");
    System.out.println(
            "============================="
    );

    System.out.println(
            "Total de empleados: "
                    + totalEmpleados
    );

    System.out.printf(
            "Promedio salarial: %.2f%n",
            promedioSalarios
    );
}
```

Aquí la estructura está casi completa porque el objetivo principal es que ustedes implementen correctamente `calcularPromedioSalarios()`.

---

# PASO 10 — Menú

```java
case 7:
    filtrarPorDesempeno(
            scanner,
            empleados,
            empleadosPorId
    );
    break;

case 8:
    mostrarReporteFinal(empleados);
    break;
```

---

# PASO 11 — Prueba manual

```text
Ana      90
Bruno    72
Carlos   85
Diana    60
```

Filtren con:

```text
80
```

Esperado:

```text
Ana
Carlos
```

Comprueben también que los IDs de Bruno y Diana ya no estén en el `HashMap`.

---

# PASO 12 — Integración

Completen:

```java
var cantidadAntes = empleados.________();

empleados.________(
        empleado ->
                empleado.getPromedioDesempeno()
                        < puntajeMinimo
);

var eliminados =
        cantidadAntes - empleados.________();

empleadosPorId.________();

for (var empleado : empleados) {

    empleadosPorId.________(
            String.valueOf(empleado.getId()),
            empleado
    );
}
```

---

# Errores frecuentes

- usar `removeIf` antes de calcular el promedio;
- filtrar solamente el `ArrayList`;
- olvidar actualizar el `HashMap`;
- dividir entre cero;
- creer que `var` significa tipado dinámico;
- calcular el promedio usando una vieja `cantidadEmpleados`.

---

# Checklist

- [ ] El promedio queda guardado en cada empleado.
- [ ] Uso `removeIf`.
- [ ] Valido el puntaje mínimo.
- [ ] Puedo saber cuántos empleados fueron eliminados.
- [ ] Sincronizo nuevamente el `HashMap`.
- [ ] Utilizo `var` en variables locales.
- [ ] Calculo salarios recorriendo el `ArrayList`.
- [ ] El promedio utiliza `empleados.size()`.
- [ ] Muestro el total.
- [ ] Muestro el promedio salarial.
