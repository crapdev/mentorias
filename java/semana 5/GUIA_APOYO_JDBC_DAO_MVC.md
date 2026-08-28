# Guía de Apoyo — JDBC, DAO, CRUD y MVC en Corporate Talent Hub

Esta guía está pensada para otros Coders que estén trabajando una Historia de Usuario similar a **Corporate Talent Hub**, especialmente cuando el proyecto pasa de guardar información en memoria con `ArrayList` o `HashMap` a guardar información de forma persistente en **PostgreSQL mediante JDBC**.

La intención no es solamente mostrar código, sino explicar **qué hace cada parte, por qué existe, cómo se relaciona con las demás capas y qué errores son comunes**.

---

# 1. ¿Qué cambia cuando pasamos de memoria a base de datos?

Antes, un proyecto puede guardar empleados así:

```java
List<Employee> empleados = new ArrayList<>();
```

o:

```java
HashMap<String, Employee> empleadosPorId = new HashMap<>();
```

Eso funciona mientras el programa está abierto.

El problema es:

```text
Programa abierto
    ↓
Los empleados existen en memoria

Programa cerrado
    ↓
La memoria se libera
    ↓
Los empleados desaparecen
```

Con PostgreSQL el flujo cambia:

```text
Java
 ↓
JDBC
 ↓
PostgreSQL
 ↓
Los datos quedan almacenados
```

Ahora, aunque se cierre el programa, los empleados siguen existiendo en la base de datos.

---

# 2. ¿Qué es JDBC?

JDBC significa:

```text
Java Database Connectivity
```

Es la API que Java utiliza para comunicarse con bases de datos relacionales.

Java conoce tipos como:

```java
Connection
PreparedStatement
ResultSet
DriverManager
```

pero necesita un **driver específico** para cada motor.

En este proyecto usamos PostgreSQL, por lo tanto Maven debe descargar el driver JDBC de PostgreSQL.

Ejemplo de dependencia:

```xml
<dependencies>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.7.13</version>
    </dependency>
</dependencies>
```

## ¿Qué hace Maven aquí?

```text
pom.xml
   ↓
Maven detecta la dependencia
   ↓
Descarga el driver
   ↓
Java puede comunicarse con PostgreSQL
```

Sin el driver, JDBC existe en Java, pero no sabe hablar específicamente con PostgreSQL.

---

# 3. Clase de conexión

Una clase como `DatabaseConnection` tiene una única responsabilidad:

> Saber cómo abrir una conexión con PostgreSQL.

Ejemplo:

```java
public class DatabaseConnection {

    private static final String URL = "jdbc:postgresql://localhost:5432/corporate_talent_hub";
    private static final String USER = "postgres";
    private static final String PASSWORD = "TU_CONTRASEÑA";

    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USER, PASSWORD);
    }
}
```

## Explicación

### `URL`

```java
jdbc:postgresql://localhost:5432/corporate_talent_hub
```

Se puede leer así:

```text
jdbc
→ estoy usando JDBC

postgresql
→ motor de base de datos

localhost
→ la base está en mi propio computador

5432
→ puerto por defecto de PostgreSQL

corporate_talent_hub
→ nombre de la base de datos
```

### `USER`

```java
private static final String USER = "postgres";
```

Es el usuario con el que Java intenta autenticarse.

### `PASSWORD`

Es la contraseña del usuario PostgreSQL.

> En proyectos reales no es recomendable subir contraseñas directamente al repositorio.

### `getConnection()`

```java
public static Connection getConnection() throws SQLException {
    return DriverManager.getConnection(URL, USER, PASSWORD);
}
```

Este método intenta abrir una conexión y devuelve un objeto de tipo:

```java
Connection
```

Mentalmente:

```text
Connection
   ↓
representa una conexión abierta
   ↓
PostgreSQL
```

---

# 4. ¿Qué es `Connection`?

`Connection` es un tipo de referencia.

Ejemplo:

```java
Connection connection = DatabaseConnection.getConnection();
```

Esto significa:

```text
Connection
→ tipo de referencia

connection
→ variable

DatabaseConnection.getConnection()
→ crea/obtiene la conexión
```

No contiene la base de datos dentro.

Representa una sesión de comunicación abierta entre Java y PostgreSQL.

---

# 5. Try-with-resources

Los recursos JDBC deben cerrarse.

Ejemplos:

```java
Connection
PreparedStatement
ResultSet
```

Si se abren muchos recursos y nunca se cierran, pueden quedar conexiones abiertas y agotarse los recursos disponibles.

Por eso usamos:

```java
try (Connection connection = DatabaseConnection.getConnection()) {

    // usar conexión
}
```

Cuando termina el bloque, Java cierra automáticamente el recurso.

## Forma mental

```text
Abrir recurso
    ↓
Usarlo
    ↓
Salir del try
    ↓
Java ejecuta close()
```

---

# 6. Legacy vs moderno

Antes de `try-with-resources` era común cerrar manualmente los recursos en `finally`.

Ejemplo Legacy:

```java
Connection connection = null;

try {
    connection = DriverManager.getConnection(...);

} catch (SQLException e) {
    e.printStackTrace();

} finally {
    if (connection != null) {
        try {
            connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

Con Java moderno:

```java
try (Connection connection = DatabaseConnection.getConnection()) {

    // usar conexión
}
```

Ventaja:

```text
Menos código
Menor riesgo de olvidar close()
Mejor manejo de recursos
Código más fácil de mantener
```

> `try-with-resources` existe desde Java 7 y sigue siendo el enfoque recomendado en Java moderno.

---

# 7. ¿Qué es PreparedStatement?

`PreparedStatement` representa una sentencia SQL preparada.

Ejemplo:

```java
String sql = "DELETE FROM employees WHERE id = ?";

PreparedStatement statement = connection.prepareStatement(sql);
```

El `?` es un parámetro.

Después:

```java
statement.setInt(1, id);
```

significa:

```text
Primer ?
↓
recibe el valor de id
```

Ejemplo:

```sql
DELETE FROM employees WHERE id = ?
```

Si:

```java
id = 5;
```

conceptualmente se envía:

```sql
DELETE FROM employees WHERE id = 5;
```

pero sin concatenar manualmente el dato dentro del SQL.

---

# 8. ¿Por qué usamos PreparedStatement?

Porque evita hacer cosas como:

```java
String sql = "DELETE FROM employees WHERE id = " + id;
```

Con `PreparedStatement` el SQL y los datos se manejan por separado.

Esto:

```java
statement.setInt(1, id);
```

trata el valor como dato y no como parte de la estructura SQL.

Por eso se utiliza para reducir el riesgo de inyección SQL.

---

# 9. ¿Qué es ResultSet?

`ResultSet` representa el resultado de una consulta `SELECT`.

Ejemplo:

```java
String sql = "SELECT * FROM employees";

try (Connection connection = DatabaseConnection.getConnection();
     PreparedStatement statement = connection.prepareStatement(sql);
     ResultSet resultSet = statement.executeQuery()) {

}
```

Mentalmente:

```text
SELECT
   ↓
PostgreSQL busca filas
   ↓
ResultSet recibe esas filas
```

---

# 10. ¿Qué hace `resultSet.next()`?

Cuando obtenemos un `ResultSet`, el cursor empieza antes de la primera fila.

```text
cursor
  ↓
[antes de la primera fila]

fila 1
fila 2
fila 3
```

Cuando hacemos:

```java
resultSet.next();
```

el cursor avanza.

```text
fila 1 ← cursor
fila 2
fila 3
```

Por eso normalmente se usa:

```java
while (resultSet.next()) {
    // leer fila actual
}
```

Así recorremos todas las filas.

---

# 11. ¿Qué significa mapear?

Mapear significa:

> Convertir datos de una representación a otra.

En JDBC normalmente hay dos direcciones.

## Java → PostgreSQL

```text
Employee
   ↓
PreparedStatement
   ↓
columnas SQL
```

Ejemplo:

```java
employee.getId()
```

se guarda en:

```text
id
```

```java
employee.getNombre()
```

se guarda en:

```text
nombre
```

## PostgreSQL → Java

```text
ResultSet
   ↓
leer columnas
   ↓
crear objeto Java
```

Ejemplo:

```java
int id = resultSet.getInt("id");
String nombre = resultSet.getString("nombre");
```

Luego:

```java
new Developer(id, nombre, ...);
```

Eso también es mapear.

---

# 12. ¿Qué es DAO?

DAO significa:

```text
Data Access Object
```

Su responsabilidad es manejar el acceso a datos.

Ejemplo:

```java
public interface EmployeeDAO {

    boolean insertar(Employee employee);

    List<Employee> listar();

    boolean actualizar(Employee employee);

    boolean eliminar(int id);
}
```

La interfaz dice:

```text
qué operaciones deben existir
```

pero no explica todavía cómo se hacen.

---

# 13. ¿Por qué interfaz + implementación?

Tenemos:

```java
EmployeeDAO
```

y:

```java
EmployeeDAOImpl
```

La idea es:

```text
EmployeeDAO
→ qué operaciones existen

EmployeeDAOImpl
→ cómo se hacen con PostgreSQL
```

Ejemplo:

```java
public class EmployeeDAOImpl implements EmployeeDAO {
```

Esto significa:

> Esta clase se compromete a implementar los métodos definidos por `EmployeeDAO`.

---

# 14. Polimorfismo con la interfaz

Podemos escribir:

```java
EmployeeDAO employeeDAO = new EmployeeDAOImpl();
```

Aquí:

```text
EmployeeDAO
→ tipo de referencia

EmployeeDAOImpl
→ tipo real del objeto
```

Esto es polimorfismo mediante interfaz.

La ventaja es que el resto del programa puede depender del contrato:

```java
EmployeeDAO
```

y no de una implementación específica.

---

# 15. CRUD

CRUD representa cuatro operaciones básicas:

```text
CREATE
READ
UPDATE
DELETE
```

En nuestro DAO:

```text
CREATE → insertar()
READ   → listar()
UPDATE → actualizar()
DELETE → eliminar()
```

---

# 16. INSERT

Ejemplo simplificado:

```java
@Override
public boolean insertar(Employee employee) {

    String sql = """
            INSERT INTO employees
            (id, nombre, edad, salario)
            VALUES (?, ?, ?, ?)
            """;

    try (Connection connection = DatabaseConnection.getConnection();
         PreparedStatement statement = connection.prepareStatement(sql)) {

        statement.setInt(1, employee.getId());
        statement.setString(2, employee.getNombre());
        statement.setShort(3, employee.getEdad());
        statement.setDouble(4, employee.getSalario());

        return statement.executeUpdate() > 0;

    } catch (SQLException e) {
        return false;
    }
}
```

## `executeUpdate()`

Se usa normalmente con:

```text
INSERT
UPDATE
DELETE
```

Devuelve cuántas filas fueron afectadas.

```text
1
→ una fila afectada

0
→ ninguna fila afectada
```

Por eso:

```java
return statement.executeUpdate() > 0;
```

devuelve `true` si la operación afectó al menos una fila.

---

# 17. `setNull()`

En el proyecto tenemos campos que aplican a un tipo y no a otro.

Ejemplo:

```text
Developer
→ main_lenguaje sí aplica
→ monthly_budget no aplica
```

Entonces:

```java
statement.setNull(9, Types.DOUBLE);
```

significa:

> En el parámetro número 9 guarda `NULL`, y ese valor corresponde a un tipo SQL numérico.

Otro ejemplo:

```java
statement.setNull(8, Types.VARCHAR);
```

significa:

> En el parámetro número 8 guarda `NULL`, y ese valor corresponde a un tipo de texto.

Forma general:

```java
statement.setNull(posicion, tipoSQL);
```

---

# 18. READ / LISTAR

Ejemplo:

```java
@Override
public List<Employee> listar() {

    List<Employee> empleados = new ArrayList<>();

    String sql = "SELECT * FROM employees ORDER BY id";

    try (Connection connection = DatabaseConnection.getConnection();
         PreparedStatement statement = connection.prepareStatement(sql);
         ResultSet resultSet = statement.executeQuery()) {

        while (resultSet.next()) {

            int id = resultSet.getInt("id");
            String nombre = resultSet.getString("nombre");

            // mapear a objeto Java
        }

    } catch (SQLException e) {
        System.out.println(e.getMessage());
    }

    return empleados;
}
```

Aquí:

```text
SELECT
↓
ResultSet
↓
while next()
↓
leer fila
↓
crear objeto
↓
agregar a List
```

---

# 19. ¿Por qué debemos saber si es Developer o Manager?

`Employee` es abstracto.

Eso significa que no podemos hacer:

```java
new Employee(...);
```

Por eso necesitamos saber qué clase concreta crear.

En PostgreSQL guardamos una columna:

```text
tipo
```

Ejemplos:

```text
DEVELOPER
MANAGER
```

Después:

```java
if (tipo.equals("DEVELOPER")) {
    employee = new Developer(...);
}
```

o:

```java
if (tipo.equals("MANAGER")) {
    employee = new Manager(...);
}
```

---

# 20. Pattern Matching

Cuando insertamos un empleado podemos tener:

```java
Employee employee = new Developer(...);
```

La referencia es:

```text
Employee
```

pero el objeto real es:

```text
Developer
```

Podemos comprobarlo así:

```java
if (employee instanceof Developer developer) {
    statement.setString(8, developer.getMainLenguaje());
}
```

Esto es Pattern Matching con `instanceof`.

Antes se hacía:

```java
if (employee instanceof Developer) {
    Developer developer = (Developer) employee;
}
```

Ahora Java puede crear directamente la referencia `developer`.

---

# 21. Arrays de calificaciones

En Java tenemos:

```java
double[] calificaciones;
```

Ejemplo:

```java
{80, 90, 95}
```

En PostgreSQL la columna es:

```sql
DOUBLE PRECISION[]
```

Para guardar ese array mediante JDBC debemos convertirlo a un arreglo compatible con `createArrayOf()`.

Versión simple:

```java
double[] notas = employee.getCalificaciones();
Double[] notasParaSQL = new Double[notas.length];

for (int i = 0; i < notas.length; i++) {
    notasParaSQL[i] = notas[i];
}

statement.setArray(5, connection.createArrayOf("float8", notasParaSQL));
```

Flujo:

```text
double[]
   ↓
Double[]
   ↓
Array SQL
   ↓
PostgreSQL
```

---

# 22. ¿Qué era `Arrays.stream()`?

También se podía escribir:

```java
Double[] notasSQL = Arrays.stream(employee.getCalificaciones())
        .boxed()
        .toArray(Double[]::new);
```

Eso hace lo mismo:

```text
double[]
↓
Stream
↓
boxed()
↓
Double[]
```

Pero para alguien que todavía no conoce Streams, la versión con `for` es más fácil de entender.

---

# 23. Leer un array desde PostgreSQL

Ejemplo:

```java
Double[] notasSQL = (Double[]) resultSet.getArray("calificaciones").getArray();
```

Se puede leer por partes.

### Paso 1

```java
resultSet.getArray("calificaciones")
```

obtiene el array SQL de PostgreSQL.

### Paso 2

```java
.getArray()
```

lo convierte a un objeto Java general.

### Paso 3

```java
(Double[])
```

hace casting.

Estamos diciendo:

> Trata este objeto como un `Double[]`.

Después podemos convertirlo a:

```java
double[]
```

si nuestro modelo usa primitivos.

---

# 24. UPDATE

Ejemplo:

```sql
UPDATE employees
SET nombre = ?, salario = ?
WHERE id = ?
```

La parte importante es:

```sql
WHERE id = ?
```

Porque indica qué fila actualizar.

Sin `WHERE`:

```sql
UPDATE employees SET salario = ?
```

podríamos cambiar muchas filas.

---

# 25. DELETE

Ejemplo:

```java
String sql = "DELETE FROM employees WHERE id = ?";
```

Después:

```java
statement.setInt(1, id);
```

y:

```java
statement.executeUpdate();
```

De nuevo:

```text
WHERE
→ define qué fila será eliminada
```

---

# 26. Modelo MVC

MVC significa:

```text
Model
View
Controller
```

En nuestro proyecto:

```text
ConsoleView
↓
EmployeeController
↓
EmployeeService
↓
EmployeeDAO
↓
EmployeeDAOImpl
↓
PostgreSQL
```

---

# 27. View

La vista se encarga de:

```text
capturar datos
mostrar información
```

Ejemplo:

```java
public int pedirId() {
    System.out.print("ID: ");
    int id = scanner.nextInt();
    scanner.nextLine();
    return id;
}
```

La vista NO debería ejecutar SQL.

---

# 28. Controller

El controlador coordina el flujo.

Ejemplo conceptual:

```java
private void registrarEmpleado() {

    int id = view.pedirId();

    // validar flujo

    Employee employee = ...;

    boolean registrado = service.registrarEmpleado(employee);

    view.mostrarMensaje(...);
}
```

El controlador:

```text
recibe
decide
coordina
```

No debería contener JDBC.

---

# 29. Service

El Service contiene reglas de negocio.

Ejemplos:

```text
calcular promedio
determinar promoción
determinar categoría salarial
calcular bono
```

Y también coordina operaciones con el DAO.

Ejemplo:

```java
public boolean registrarEmpleado(Employee employee) {
    return employeeDAO.insertar(employee);
}
```

---

# 30. DAO / DAOImpl

El DAO se ocupa de acceso a datos.

```text
DAO
→ contrato

DAOImpl
→ JDBC + SQL
```

Ejemplo:

```text
Service
↓
EmployeeDAO
↓
EmployeeDAOImpl
↓
PostgreSQL
```

---

# 31. App

`App` debería ser pequeña.

Ejemplo:

```java
public class App {

    public static void main(String[] args) {

        EmployeeService service = new EmployeeService();
        ConsoleView view = new ConsoleView();

        EmployeeController controller = new EmployeeController(service, view);

        controller.iniciar();
    }
}
```

Su función es:

```text
crear objetos
↓
conectarlos
↓
iniciar aplicación
```

---

# 32. Records

Un record es útil cuando queremos transportar datos sin crear una clase muy verbosa.

Ejemplo:

```java
public record PerformanceReport(int idEmployee, double average, String feedback) {
}
```

Java genera automáticamente:

```text
constructor
accesores
equals()
hashCode()
toString()
```

---

# 33. Accesores de un record

POJO tradicional:

```java
reporte.getAverage();
```

Record:

```java
reporte.average();
```

El nombre del componente se convierte en el método de acceso.

---

# 34. Record + JDBC

Podemos hacer:

```text
SELECT
↓
ResultSet
↓
PerformanceReport
```

Ejemplo:

```java
PerformanceReport report = new PerformanceReport(
        resultSet.getInt("id"),
        resultSet.getDouble("promedio_desempeno"),
        resultSet.getString("feedback")
);
```

Eso es mapear un resultado SQL directamente a un record.

---

# 35. Text Blocks

Java moderno permite escribir texto multilínea así:

```java
String reporte = """
        ========================
        REPORTE DE DESEMPEÑO
        ========================
        ID: %d
        Promedio: %.2f
        Feedback: %s
        ========================
        """;
```

Ventaja:

```text
menos \n
menos concatenaciones
más legible
```

---

# 36. Tabla PostgreSQL utilizada

```sql
CREATE TABLE employees (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(250) NOT NULL,
    edad SMALLINT NOT NULL,
    salario DECIMAL(12,2) NOT NULL,
    calificaciones DOUBLE PRECISION[] NOT NULL,
    promedio_desempeno DECIMAL(8,2) NOT NULL,
    tipo VARCHAR(20) NOT NULL,
    main_lenguaje VARCHAR(100),
    monthly_budget DECIMAL(12,2)
);
```

---

# 37. Tipos Java y PostgreSQL

| Java | PostgreSQL |
|---|---|
| `int` | `INTEGER` |
| `byte` | `SMALLINT` |
| `String` | `VARCHAR` |
| `double` | `DOUBLE PRECISION` / `DECIMAL` |
| `double[]` | `DOUBLE PRECISION[]` |

---

# 38. Flujo completo al insertar

```text
Usuario
↓
ConsoleView pide datos
↓
EmployeeController recibe datos
↓
crea Developer o Manager
↓
EmployeeService procesa reglas
↓
EmployeeDAO.insertar()
↓
EmployeeDAOImpl
↓
PreparedStatement
↓
INSERT
↓
PostgreSQL
```

---

# 39. Flujo completo al listar

```text
Controller
↓
Service
↓
DAO
↓
SELECT
↓
ResultSet
↓
mapear filas
↓
List<Employee>
↓
Service
↓
Controller
↓
View
↓
Usuario
```

---

# 40. Errores frecuentes

## Error 1: poner `try` directamente dentro de una clase

Incorrecto:

```java
public class DatabaseConnection {

    try (...) {
    }
}
```

Un `try` debe estar dentro de un método, constructor o bloque válido.

## Error 2: olvidar `WHERE`

Incorrecto:

```sql
DELETE FROM employees;
```

Eso elimina todas las filas.

Correcto:

```sql
DELETE FROM employees WHERE id = ?;
```

## Error 3: concatenar valores en SQL

Evitar:

```java
String sql = "SELECT * FROM employees WHERE nombre = '" + nombre + "'";
```

Preferir:

```java
String sql = "SELECT * FROM employees WHERE nombre = ?";
```

## Error 4: guardar con ArrayList pero creer que ya persiste

Esto:

```java
new ArrayList<>();
```

guarda en memoria.

No en PostgreSQL.

## Error 5: usar directamente `new Employee(...)`

Si `Employee` es abstracto:

```java
new Employee(...);
```

no compila.

Hay que crear:

```java
new Developer(...);
```

o:

```java
new Manager(...);
```

## Error 6: mezclar Scanner con Controller o Service

Si la HU exige que Scanner esté solo en View:

```text
Scanner
→ ConsoleView
```

No:

```text
Scanner
→ Controller
```

---

# 41. Qué debería poder explicar un Coder al terminar

### ¿Qué es JDBC?

API de Java para comunicarse con bases de datos relacionales.

### ¿Qué hace Connection?

Representa una conexión abierta con la base de datos.

### ¿Qué hace PreparedStatement?

Prepara y ejecuta SQL parametrizado.

### ¿Qué hace ResultSet?

Contiene los resultados de un `SELECT`.

### ¿Qué hace try-with-resources?

Cierra automáticamente recursos JDBC.

### ¿Qué es DAO?

Una abstracción dedicada al acceso a datos.

### ¿Qué es mapear?

Convertir datos entre SQL y objetos Java.

### ¿Qué hace MVC?

Separa presentación, coordinación y modelo.

### ¿Por qué usamos record?

Para transportar datos de forma compacta y con menos código repetitivo.

### ¿Por qué PostgreSQL y no ArrayList?

Porque PostgreSQL permite persistencia real después de cerrar el programa.

---

# 42. Resumen final

```text
JDBC
→ conecta Java con PostgreSQL

Connection
→ abre la comunicación

PreparedStatement
→ prepara SQL seguro y parametrizado

ResultSet
→ recibe resultados

DAO
→ define operaciones de persistencia

DAOImpl
→ implementa JDBC y SQL

MVC
→ separa responsabilidades

Record
→ transporta datos con poco código

Text Blocks
→ mejora textos multilínea

PostgreSQL
→ mantiene los datos persistentes
```

---

# 43. Recomendación para practicar

Antes de copiar el proyecto completo, intenta explicar con tus propias palabras este flujo:

```text
View
↓
Controller
↓
Service
↓
DAO
↓
DAOImpl
↓
PostgreSQL
```

Después intenta responder:

```text
¿Qué capa captura?
¿Qué capa coordina?
¿Qué capa tiene reglas?
¿Qué capa ejecuta SQL?
¿Qué objeto recibe un SELECT?
¿Qué objeto representa la conexión?
```

Si puedes responder eso sin mirar el código, ya tienes una base sólida para volver a implementar la misma arquitectura en otro proyecto.
