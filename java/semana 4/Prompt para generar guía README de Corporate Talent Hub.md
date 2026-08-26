# Corporate Talent Hub - Arquitectura POO avanzada

## Objetivo de la Historia de Usuario

Refactorizar el sistema para soportar diferentes tipos de perfiles (Desarrolladores, Gerentes) aplicando los pilares de la POO. El Coder deberá implementar jerarquías protegidas y modelado de datos inmutable, comparando la herencia tradicional de Java 8/11 con las Sealed Classes y Records de Java 17/21, además de simplificar la lógica mediante Pattern Matching.

## TASK 1

### Herencia sellada vs abierta

Definir la estructura jerárquica del sistema protegiendo el dominio del negocio.

- Estilo Legacy (Java 8/11): crear la clase abstracta `Persona` y permitir que cualquier clase pueda heredar de ella de forma abierta.
- Estilo moderno (Java 17/21): refactorizar `Persona` a una `sealed class` que solo permita ser extendida por `Empleado` y una nueva clase `ConsultorExterno` mediante `permits`.
- Incluir un comentario técnico explicando por qué las Sealed Classes ofrecen más seguridad en el diseño de APIs frente a la herencia abierta.

## TASK 2

### Modelado inmutable con Records

Crear estructuras de datos ligeras y seguras para reportes de desempeño.

- Sintaxis Legacy (Java 8/11): contrastar con las clases POJO tradicionales que requieren constructores, getters y métodos `toString` manuales.
- Sintaxis moderna (Java 17/21): crear un `record` llamado `DesempenoReport` que contenga:
  - `int idEmpleado`
  - `double promedio`
  - `String feedback`
- Integrar el `record` en el flujo de la aplicación para emitir reportes inmutables de fin de mes.

## TASK 3

### El salto a Java 21: Polimorfismo y Pattern Matching

Implementar comportamientos específicos por rol eliminando código repetitivo de conversión.

- Crear las subclases `Desarrollador`, con atributo `lenguajePrincipal`, y `Gerente`, con atributo `presupuestoMensual`.
- Sintaxis Legacy (Java 8/11): implementar un ejemplo de validación que utilice `instanceof` seguido de casting manual obligatorio.
- Sintaxis moderna (Java 17/21): mostrar cómo rediseñar esa lógica utilizando Pattern Matching for `instanceof`, por ejemplo:

```java
if (persona instanceof Desarrollador desarrollador) {
    // acceso directo al subtipo
}
```

## TASK 4

### Abstracción y evolución de interfaces

Definir contratos de comportamiento capaces de evolucionar sin romper las clases existentes.

- Crear la interfaz `Promocionable` con un método abstracto para calcular bonos de ascenso.
- Implementar un método `default` en la interfaz para registrar el log de la operación.
- Explicar cómo Java 8 permitió agregar funcionalidades nuevas a una interfaz sin obligar a modificar inmediatamente todas las clases que ya la implementaban.
- Asegurar que los atributos de la jerarquía utilicen correctamente `private` y `protected` para mantener encapsulamiento.

## Criterios de aceptación

- El sistema utiliza una jerarquía protegida mediante una Sealed Class.
- Se utiliza un Record para manejo de datos inmutables.
- Se elimina el casting manual mediante Pattern Matching.
- Se implementa polimorfismo mediante una interfaz con al menos un método `default`.
- Se incluyen comentarios explicando las ventajas de las características modernas frente a las Legacy.

---

# Objetivo del README que debes crear

El README NO debe ser simplemente una solución terminada para copiar y pegar.

Debe funcionar como una guía práctica y educativa que permita que cada coder implemente la Historia de Usuario sobre SU propio proyecto, aunque la organización interna de sus clases, servicios, menús o archivos sea diferente.

Todos manejan una arquitectura parecida y normalmente cuentan con un package `model`, por lo que puedes sugerir que las clases relacionadas con el dominio como `Persona`, `Empleado`, `Desarrollador`, `Gerente`, `ConsultorExterno`, `Promocionable` y `DesempenoReport` podrían ubicarse allí.

Sin embargo, no debes asumir que todos tienen exactamente la misma arquitectura ni los mismos métodos.

Debes enseñarles qué necesitan implementar, por qué y cómo adaptarlo a su código.

---

# Nivel de conocimiento de los coders

Asume que los coders:

- Ya conocen clases y objetos.
- Conocen constructores.
- Conocen herencia básica.
- Conocen getters y setters.
- Han visto arquitectura por capas.
- Han trabajado con `private` y conocen de manera básica `protected`.
- Están aprendiendo por primera vez:
  - `sealed`
  - `permits`
  - `non-sealed`
  - Records
  - Pattern Matching
  - una aplicación más profunda del polimorfismo
  - evolución de interfaces con métodos `default`

Por esta razón, no des por sentado que entienden estas características modernas.

Explícalas desde cero, pero sin tratarlos como si nunca hubieran programado.

---

# Forma de explicar cada TASK

Quiero que cada TASK tenga aproximadamente esta estructura:

## 1. ¿Qué te está pidiendo realmente esta tarea?

Traducir el requerimiento técnico a palabras sencillas.

Por ejemplo, no limitarte a decir:

> Crea una sealed class.

Sino explicar qué problema está intentando solucionar la actividad.

---

## 2. Conceptos que necesitas entender antes

Presentar los conceptos nuevos utilizados en esa TASK.

Por ejemplo, para TASK 1 explicar:

- herencia abierta;
- `abstract`;
- `sealed`;
- `permits`;
- `non-sealed`;
- `final`.

Además debes explicar claramente que estos modificadores NO significan lo mismo.

Por ejemplo:

- `abstract` responde a si una clase puede ser instanciada directamente;
- `sealed` controla quién puede heredar;
- `non-sealed` vuelve a abrir una rama de herencia;
- `final` evita que una clase tenga nuevos hijos.

Usa diagramas de texto cuando ayuden.

Ejemplo conceptual:

```text
Persona
│
├── Empleado
│   ├── Desarrollador
│   └── Gerente
│
└── ConsultorExterno
```

---

## 3. ¿Para qué sirve esto en un proyecto real?

No quiero que la guía enseñe solamente sintaxis.

Explica qué problema de diseño resuelve la característica.

Ejemplo:

Una `sealed class` permite que el sistema controle qué clases pueden formar parte de una jerarquía importante del dominio.

Explica también cuándo NO sería necesario utilizar esa característica.

---

## 4. ¿Dónde podría implementarlo?

Como todos probablemente tengan un package `model`, puedes recomendar allí las clases relacionadas con el dominio.

Por ejemplo:

```text
model
├── Persona.java
├── Empleado.java
├── Desarrollador.java
├── Gerente.java
├── ConsultorExterno.java
├── DesempenoReport.java
└── Promocionable.java
```

Aclara que esto es una recomendación y que cada coder debe adaptarlo a la arquitectura existente de su proyecto.

Si algo pertenece a lógica de negocio, puedes sugerir hacerlo en su capa `service`, si cuentan con ella.

Si algo solamente presenta información, puedes sugerir su capa `view`.

No asumir nombres concretos de archivos fuera del dominio.

---

## 5. Implementación guiada

Explica paso por paso qué debería modificar el coder.

No entregues el proyecto completo.

Puedes proporcionar pequeñas piezas de código que ilustren el concepto.

Por ejemplo:

```java
public abstract sealed class Persona permits Empleado, ConsultorExterno {
}
```

Luego explica cada palabra.

Otro ejemplo:

```java
public non-sealed class Empleado extends Persona {
}
```

Explica por qué podría utilizarse `non-sealed`.

Pero NO entregues todas las clases completas conectadas entre sí de manera que solo tengan que copiar y pegar.

La intención es enseñar y orientar.

---

# Muy importante sobre el código

Los ejemplos deben ser fáciles de leer y naturales.

NO hagas saltos de línea innecesarios como:

```java
public Desarrollador(
        int id,
        String nombre,
        byte edad,
        String lenguajePrincipal
) {
```

Si cabe correctamente en una sola línea, escribe:

```java
public Desarrollador(int id, String nombre, byte edad, String lenguajePrincipal) {
```

Lo mismo aplica a llamadas a métodos.

Evita:

```java
view.mostrarEmpleado(
        empleado,
        promedio,
        feedback
);
```

si puede escribirse perfectamente:

```java
view.mostrarEmpleado(empleado, promedio, feedback);
```

Solo divide líneas cuando realmente sean demasiado largas.

---

# TASK 1: conceptos que debes explicar especialmente

Además de la implementación, quiero una explicación clara sobre:

### Herencia abierta

Mostrar conceptualmente el estilo Legacy:

```java
public abstract class Persona {
}
```

y explicar que cualquier clase podría extenderla.

Después compararlo con:

```java
public abstract sealed class Persona permits Empleado, ConsultorExterno {
}
```

Explicar cómo `permits` limita los hijos directos.

---

### `abstract` vs `sealed`

Debe quedar extremadamente claro que:

```text
abstract
→ controla si podemos crear directamente objetos de esa clase.

sealed
→ controla quién puede heredar.
```

Pueden utilizarse juntos.

---

### `non-sealed`

Explicar por qué, si `Empleado` debe permitir posteriormente:

```text
Empleado
├── Desarrollador
└── Gerente
```

podría utilizarse:

```java
public abstract non-sealed class Empleado extends Persona {
}
```

Explica también que `abstract` y `non-sealed` cumplen responsabilidades diferentes.

---

### `final`

Explicar que un hijo directo de una clase sealed también podría declararse `final` si no queremos permitir más herencia.

Ejemplo conceptual:

```java
public final class ConsultorExterno extends Persona {
}
```

---

### `super()`

Explicar muy bien qué ocurre al mover atributos comunes como:

```text
id
nombre
edad
```

a `Persona`.

Mostrar que una subclase puede llamar:

```java
super(id, nombre, edad);
```

y explicar que NO se están creando dos objetos.

Se sigue creando un solo objeto hijo que contiene la parte heredada del padre.

---

# Explicar `final` en atributos

Incluye una pequeña explicación porque es una duda frecuente.

Ejemplo:

```java
private final int id;
```

Explicar que significa que la variable no puede reasignarse después de inicializarse.

Pero NO enseñes que todo atributo debe ser `final`.

Explica que la decisión depende del dominio.

Dar ejemplos:

```text
id
→ normalmente no debería cambiar.

precio
→ podría cambiar.

stock
→ probablemente cambia.

salario
→ podría cambiar.

fechaNacimiento
→ normalmente no cambia.
```

Enseña esta pregunta como regla práctica:

> ¿Este dato debería cambiar después de crear el objeto?

Si no debería cambiar, `final` podría ser apropiado.

Si representa un estado que evoluciona, probablemente no.

También aclara que:

```java
private final double[] notas;
```

no vuelve inmutable el contenido del arreglo.

`final` protege la referencia, no necesariamente el objeto mutable al que apunta.

---

# TASK 2: Records

Explica primero cómo sería una clase POJO Legacy.

No es necesario implementar toda una clase enorme, pero sí mostrar conceptualmente que normalmente requiere:

```text
atributos
constructor
getters
toString()
equals()
hashCode()
```

Luego comparar con:

```java
public record DesempenoReport(int idEmpleado, double promedio, String feedback) {
}
```

Explicar que Java genera automáticamente:

- constructor;
- accesores;
- `equals`;
- `hashCode`;
- `toString`.

---

## Accesores de un Record

Destaca especialmente esta diferencia:

Clase normal:

```java
empleado.getId();
```

Record:

```java
reporte.idEmpleado();
```

No asumir que el coder conoce esta diferencia.

---

## Inmutabilidad

Explica por qué un record es apropiado para un reporte.

Usa la idea de:

> un reporte representa una fotografía de ciertos datos en un momento específico.

Explica que sus componentes no pueden reasignarse después de crear el record.

Aclara también que la inmutabilidad puede ser superficial si uno de sus componentes es un objeto mutable.

---

## Record vs entidad

Explicar claramente por qué no necesariamente conviene convertir `Empleado` en record.

Por ejemplo:

```text
Empleado
→ representa una entidad cuyo estado puede evolucionar.

DesempenoReport
→ representa información generada para transportar o mostrar datos.
```

---

## Integración real del Record

No dejes el record creado sin utilizar.

Explica que la TASK exige integrarlo al flujo.

Sugiere algo como:

1. El servicio obtiene/calcula el promedio.
2. El servicio determina un `feedback`.
3. El servicio construye un `DesempenoReport`.
4. La capa de presentación recibe el record y muestra sus datos.

Conceptualmente:

```text
Empleado
   ↓
Service
   ↓
DesempenoReport
   ↓
View
```

Si el proyecto cuenta con un menú de opciones, puedes sugerir:

> Puedes agregar una opción como "Generar reportes mensuales" y desde allí solicitar al servicio que genere un `DesempenoReport` por cada empleado.

NO entregues necesariamente todo el código del menú.

Puedes mostrar solamente la idea:

```java
var reporte = servicio.generarReporteDesempeno(empleado);
```

y explicar dónde podría utilizarse.

También aclara que si el requisito no pide almacenar los reportes permanentemente, no es obligatorio crear un Repository adicional solo para ellos.

---

# TASK 3: Polimorfismo y Pattern Matching

Explicar primero la nueva jerarquía:

```text
Persona
│
├── Empleado
│   ├── Desarrollador
│   └── Gerente
│
└── ConsultorExterno
```

Mostrar que:

```text
Desarrollador ES UN Empleado.
Gerente ES UN Empleado.
Empleado ES UNA Persona.
```

---

## Polimorfismo

Explicar con:

```java
Empleado empleado = new Desarrollador(...);
```

y:

```java
Empleado empleado = new Gerente(...);
```

Explicar que la referencia es del tipo padre pero el objeto real puede ser de distintos subtipos.

Conectar esto con:

```java
List<Empleado>
```

Explicar que una lista de empleados puede contener tanto desarrolladores como gerentes.

---

## Limitación de una referencia padre

Explicar algo fundamental:

Si tenemos:

```java
Empleado empleado = new Desarrollador(...);
```

podemos llamar métodos conocidos por `Empleado`, pero no directamente algo exclusivo como:

```java
empleado.getLenguajePrincipal();
```

porque ese método pertenece específicamente a `Desarrollador`.

Esto debe introducir la necesidad del casting.

---

## Legacy: instanceof + casting

Mostrar:

```java
if (empleado instanceof Desarrollador) {
    Desarrollador desarrollador = (Desarrollador) empleado;
    System.out.println(desarrollador.getLenguajePrincipal());
}
```

Explicar:

1. `instanceof` comprueba el tipo real.
2. `(Desarrollador)` realiza el casting manual.
3. Ahora podemos utilizar los métodos exclusivos del hijo.

Explicar también por qué hacer casting sin verificar primero puede causar `ClassCastException`.

---

## Pattern Matching moderno

Compararlo con:

```java
if (empleado instanceof Desarrollador desarrollador) {
    System.out.println(desarrollador.getLenguajePrincipal());
}
```

Explicar que Pattern Matching:

1. comprueba el tipo;
2. crea automáticamente una variable ya tratada como ese subtipo.

Mostrar la comparación visual:

```text
Legacy:
comprobar → casting → utilizar

Moderno:
comprobar + obtener variable → utilizar
```

---

## Legacy vs moderno

Incluye comentarios técnicos que el coder pueda adaptar.

Por ejemplo:

```java
/*
 * Java 8/11 requería verificar el tipo con instanceof y posteriormente
 * realizar un casting manual para acceder a miembros específicos del subtipo.
 *
 * Pattern Matching permite combinar ambas operaciones, reduciendo
 * código repetitivo y haciendo más clara la intención.
 */
```

No entregar una solución completa, pero sí este tipo de comentarios educativos.

---

# TASK 4: Interfaces

Explicar desde cero qué representa `Promocionable`.

Usa esta idea:

```text
extends
→ relación "ES UN"

implements
→ capacidad o contrato que una clase cumple
```

Ejemplo:

```text
Desarrollador ES UN Empleado.

Desarrollador ES Promocionable / puede cumplir el contrato Promocionable.
```

Aclara que esta es una simplificación conceptual útil para aprender.

---

## Método abstracto en la interfaz

Mostrar:

```java
public interface Promocionable {
    double calcularBonoAscenso();
}
```

Explicar que la interfaz dice:

> cualquier clase que implemente Promocionable debe saber calcular un bono.

Pero la interfaz no define obligatoriamente cómo hacerlo.

---

## Implementación diferente

Mostrar una idea como:

```java
@Override
public double calcularBonoAscenso() {
    // cálculo propio del Desarrollador
}
```

y explicar que `Gerente` podría implementar una regla distinta.

NO inventes reglas de negocio como 10% o 15% sin aclararlo.

Si necesitas porcentajes para un ejemplo, debes indicar explícitamente:

> Este porcentaje es únicamente ilustrativo porque la Historia de Usuario no proporciona una fórmula.

---

# Método `default`

Explicar muy bien cuál fue el problema que Java 8 buscó solucionar.

Ejemplo conceptual:

Había una interfaz:

```java
public interface Promocionable {
    double calcularBonoAscenso();
}
```

Muchas clases ya la implementaban.

Si posteriormente agregáramos:

```java
void registrarLog();
```

todas quedarían obligadas a implementar el método nuevo.

Mostrar entonces:

```java
default void registrarLog() {
    System.out.println("Operación registrada.");
}
```

Explicar:

```text
método abstracto
→ la clase debe implementarlo.

método default
→ la interfaz ya proporciona una implementación.
→ la clase puede utilizarla directamente.
→ también puede sobrescribirla.
```

Explica que esto permitió evolucionar interfaces existentes con menor impacto sobre las implementaciones anteriores.

---

# private vs protected

Incluye una sección clara porque esta duda es importante.

Explicar:

```java
private double salario;
```

significa que solamente la propia clase puede acceder directamente al atributo.

Mientras:

```java
protected double salario;
```

permite acceso desde las subclases, además de las reglas de acceso del mismo package propias de Java.

Para no complicar demasiado a los principiantes, enfócate principalmente en la relación padre/hijo.

---

## Recomendación sobre atributos

No recomendar utilizar `protected` en atributos simplemente porque la TASK lo menciona.

Explicar que normalmente es preferible:

```java
private double salario;
```

porque mantiene el estado encapsulado.

Un hijo no debería poder modificar arbitrariamente:

```java
salario = -500000;
```

---

# Cuándo usar protected

Explica que `protected` puede ser especialmente útil en métodos auxiliares que solamente deben utilizar la clase padre y sus hijos.

Ejemplo:

```java
protected double calcularPorcentajeSalario(double porcentaje) {
    return salario * porcentaje;
}
```

El atributo continúa siendo:

```java
private double salario;
```

pero las clases hijas pueden reutilizar una operación controlada.

---

# protected vs abstract

Esta explicación es obligatoria porque es una de las dudas más importantes.

Usa esta idea:

```text
protected
→ "quiero DARLE una herramienta a mis hijos."

abstract
→ "quiero OBLIGAR a mis hijos a definir su propio comportamiento."
```

Ejemplo:

```java
protected double calcularPorcentajeSalario(double porcentaje) {
    return salario * porcentaje;
}
```

La clase padre YA sabe cómo ejecutar esa operación.

En cambio:

```java
public abstract double calcularBonoAscenso();
```

significa que la clase padre sabe que todos los hijos deben tener ese comportamiento, pero no sabe cuál implementación concreta corresponde a cada uno.

Presentar esta regla:

```text
¿Todos los hijos necesitan este comportamiento?
        ↓
       Sí

¿La implementación es igual para todos?
        ↓
Sí → método concreto reutilizable, posiblemente protected.

No → método abstracto para que cada hijo lo sobrescriba.
```

También aclarar que `protected` es un modificador de acceso y `abstract` define que un método no tiene implementación concreta en esa clase.

Por eso no son conceptos opuestos exactos, aunque ayudan a resolver decisiones distintas.

---

# Métodos protected vs atributos protected

Recomienda generalmente:

```text
atributos
→ private

comportamiento auxiliar para hijos
→ protected cuando tenga sentido
```

Explica el motivo: mantener el estado interno protegido mientras permitimos reutilizar operaciones controladas.

---

# Polimorfismo mediante interfaz

Mostrar conceptualmente:

```java
Promocionable promocionable = desarrollador;
promocionable.calcularBonoAscenso();
```

y:

```java
Promocionable promocionable = gerente;
promocionable.calcularBonoAscenso();
```

Explicar que la misma referencia `Promocionable` puede ejecutar diferentes implementaciones dependiendo del objeto real.

---

# Integración con el programa

Para cada TASK, además de explicar las clases del `model`, incluye recomendaciones sobre cómo demostrar que realmente funciona dentro del programa.

Por ejemplo:

### TASK 2

Si existe un menú, sugerir:

```text
Generar reportes mensuales
```

y explicar que esa opción podría:

1. obtener empleados;
2. generar `DesempenoReport`;
3. enviarlos a la vista.

### TASK 3

Podría existir una opción:

```text
Consultar información específica por rol
```

para demostrar Pattern Matching con Desarrolladores y Gerentes.

### TASK 4

Podría existir:

```text
Consultar bono de ascenso
```

para demostrar polimorfismo y el método `default`.

No proporciones todo el flujo terminado para copiar.

Indica qué deberían modificar y proporciona fragmentos representativos.

---

# Errores comunes

Incluye errores frecuentes en cada TASK.

Algunos ejemplos:

## TASK 1

- olvidar poner en `permits` un hijo directo;
- intentar extender una clase sealed desde una clase no permitida;
- no declarar correctamente qué pasa con la herencia del hijo (`final`, `sealed` o `non-sealed`);
- volver `Empleado` abstracto y seguir intentando hacer `new Empleado()`.

## TASK 2

- crear el record pero nunca utilizarlo;
- intentar usar `getPromedio()` cuando el accesor generado es `promedio()`;
- asumir que record significa que cualquier objeto interno también es completamente inmutable.

## TASK 3

- hacer casting sin comprobar el subtipo;
- pensar que Pattern Matching y polimorfismo son exactamente lo mismo;
- intentar utilizar métodos exclusivos del hijo mediante una referencia padre sin realizar ninguna comprobación.

## TASK 4

- creer que todos los métodos de interfaz deben tener implementación;
- confundir `default` con `static`;
- utilizar atributos `protected` innecesariamente;
- implementar la misma lógica de bono mediante muchos `if` en Service en vez de aprovechar el polimorfismo.

---

# Preguntas de comprobación

Al final de cada TASK agrega entre 3 y 5 preguntas cortas que permitan comprobar si el coder entendió.

No deben ser preguntas demasiado académicas.

Ejemplos:

### TASK 1

- ¿Qué controla `sealed` que `abstract` no controla?
- ¿Por qué `Empleado` podría ser `non-sealed`?
- ¿Qué pasaría si intentaras crear directamente una instancia de una clase abstracta?

### TASK 2

- ¿Por qué un reporte puede ser buen candidato para un record?
- ¿Cómo accederías al componente `promedio` de un record?
- ¿Por qué `Empleado` no necesariamente debería convertirse en record?

### TASK 3

- ¿Qué diferencia hay entre el tipo de la referencia y el tipo real del objeto?
- ¿Qué problema evita `instanceof` antes del casting?
- ¿Qué elimina Pattern Matching?

### TASK 4

- ¿Qué diferencia existe entre un método abstracto y uno default?
- ¿Cuándo tendría sentido utilizar un método protected?
- ¿Por qué normalmente conviene mantener los atributos private?

Después proporciona las respuestas breves debajo, idealmente colapsables si Markdown lo permite mediante `<details>`.

---

# Conceptos que debes dominar

Al finalizar las cuatro TASK, agrega una sección resumen que conecte todos los conceptos:

```text
abstract
sealed
permits
non-sealed
final
extends
super
record
inmutabilidad
instanceof
casting
Pattern Matching
polimorfismo
interface
implements
default
private
protected
```

No debe ser un diccionario enorme.

Quiero definiciones cortas y conectadas entre sí.

Ejemplo:

```text
sealed
→ controla qué clases pueden heredar directamente.

non-sealed
→ vuelve a permitir herencia abierta desde una rama autorizada.

abstract
→ impide instanciar directamente una clase y puede contener métodos sin implementación.

record
→ estructura compacta orientada principalmente a representar datos.
```

---

# Checklist final de criterios de aceptación

Finaliza el README con una checklist.

Debe ser parecida a:

```markdown
- [ ] Existe una clase `Persona`.
- [ ] `Persona` utiliza `sealed`.
- [ ] `permits` limita correctamente sus hijos directos.
- [ ] Se explicó la diferencia con la herencia Legacy.
- [ ] Existe `DesempenoReport` como record.
- [ ] El record se utiliza realmente dentro del flujo de la aplicación.
- [ ] Existen `Desarrollador` y `Gerente`.
- [ ] Se mostró el enfoque Legacy con `instanceof` + casting.
- [ ] La implementación moderna utiliza Pattern Matching.
- [ ] Existe la interfaz `Promocionable`.
- [ ] Tiene al menos un método abstracto.
- [ ] Tiene al menos un método `default`.
- [ ] Se demuestra polimorfismo mediante la interfaz.
- [ ] Los atributos mantienen encapsulación adecuada.
- [ ] `protected` solamente se utiliza cuando aporta valor real.
- [ ] Existen comentarios comparando características Legacy y modernas.
```

Añade una última sección:

# Antes de entregar

Con unas 5 comprobaciones prácticas, como:

- Compilar el proyecto.
- Registrar o crear al menos un Desarrollador.
- Registrar o crear al menos un Gerente.
- Probar la generación de un reporte.
- Probar la lógica de Pattern Matching.
- Probar el cálculo polimórfico del bono.
- Confirmar que no quedan clases modernas creadas pero sin utilizar.

---

# Estilo general del README

Escríbelo en español.

Debe sentirse como una guía de un tutor hacia otros coders.

Debe ser:

- clara;
- amigable;
- muy didáctica;
- detallada cuando el concepto sea nuevo;
- directa cuando algo sea sencillo;
- llena de ejemplos pequeños;
- orientada a implementación real.

Evita lenguaje excesivamente técnico sin explicación.

No hagas párrafos enormes.

Utiliza títulos, diagramas de texto, tablas pequeñas cuando realmente ayuden y fragmentos de código.

No abuses de emojis.

No entregues una solución completa del proyecto.

La intención principal es que el coder termine la Historia de Usuario entendiendo qué hizo y por qué lo hizo, no que simplemente copie código.

Cuando una decisión dependa del proyecto de cada persona, dilo explícitamente:

> Adapta esta parte a la arquitectura que ya tienes.

Cuando una regla de negocio no aparezca en la Historia de Usuario, no la presentes como requisito.

Por ejemplo, si necesitas demostrar un bono del 10%, aclara que es un valor ilustrativo y que cada coder debe utilizar las reglas que tenga definidas en su proyecto.

Quiero que al terminar de leer el README una persona pueda implementar las cuatro tareas prácticamente por su cuenta y además pueda explicar conceptos como `sealed`, `record`, Pattern Matching, interfaces, `default`, `protected`, `abstract` y polimorfismo con sus propias palabras.
