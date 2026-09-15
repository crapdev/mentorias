# Eventify — Semana 2

Guía de estudio e implementación paso a paso de la evolución de **Eventify** hacia la persistencia real, aplicando Spring Data JPA, Hibernate, un CRUD completo, manejo de errores con código `404`, y paginación/ordenamiento de resultados.

## 1. ¿Qué vamos a construir?

En la Semana 1 construimos la base arquitectónica de **Eventify**: un catálogo de eventos y lugares que vivía únicamente en memoria, con validaciones básicas y documentación en Swagger.

Ahora vamos a dar el salto que convierte a Eventify en una aplicación real: reemplazaremos las listas en memoria por una **base de datos persistente** usando **Spring Data JPA** e **Hibernate**. Sobre esa base construiremos un **CRUD completo** (crear, consultar, actualizar y eliminar), añadiremos un manejo de errores adecuado cuando un recurso no exista, y optimizaremos los listados con **paginación y ordenamiento**, de forma que el catálogo pueda crecer sin volverse pesado de consultar.

---

## (Opcional) Cómo reutilizar la carpeta de la Semana 1

No es obligatorio reescribir todo el código desde cero. Puedes duplicar el proyecto de la Semana 1 y usar esa copia como punto de partida. La única regla importante es: **copia solo el código fuente, no las carpetas que Maven o el IDE generan automáticamente.**

### Paso 1 — Cierra todo antes de copiar
Detén la aplicación si la tienes corriendo y cierra el proyecto en el IDE. Esto evita que algún archivo quede bloqueado durante la copia.

### Paso 2 — Duplica la carpeta completa
Haz una copia de toda la carpeta del proyecto y ponle un nombre distinto, por ejemplo `eventify-semana2`. Puedes hacerlo arrastrando la carpeta en tu explorador de archivos (Ctrl+C / Ctrl+V y renombrar), o con un comando:

```bash
# Linux / macOS
cp -r eventify-semana1 eventify-semana2
```

```powershell
# Windows (PowerShell)
Copy-Item -Recurse eventify-semana1 eventify-semana2
```

En este punto la copia trae de más algunas carpetas que no necesitas. El siguiente paso es borrarlas.

### Paso 3 — Borra dentro de `eventify-semana2` las carpetas que NO debes conservar
Dentro de la carpeta nueva, busca y elimina estas tres si existen:

| Carpeta a borrar | Por qué molesta si la dejas |
| :--- | :--- |
| `target/` | Tiene el código ya compilado de la Semana 1. Si la conservas, Maven puede mezclar clases viejas con el código nuevo. |
| `.idea/` (IntelliJ) o `.vscode/` (VS Code) | Guarda configuración apuntando al proyecto original. Si la conservas, el IDE puede confundirse sobre cuál carpeta es cuál. |

```bash
# Linux / macOS, parado dentro de eventify-semana2/
rm -rf target .idea .vscode
```

```powershell
# Windows (PowerShell), parado dentro de eventify-semana2\
Remove-Item -Recurse -Force target, .idea, .vscode -ErrorAction SilentlyContinue
```

Lo que **sí** debe quedar dentro de `eventify-semana2/` es: la carpeta `src/`, el archivo `pom.xml`, y si los tienes, `mvnw`, `mvnw.cmd` y la carpeta `.mvn/`.

### Paso 4 — Abre `eventify-semana2` como un proyecto nuevo
En tu IDE, usa la opción "Abrir proyecto" (u "Open") y selecciona la carpeta `eventify-semana2`. No reutilices la ventana que ya tenías abierta con la Semana 1; ábrela como si fuera un proyecto totalmente distinto.

### Paso 5 — Verifica que compile limpio
Ejecuta esto dentro de `eventify-semana2/` para forzar que Maven regenere `target/` desde cero:

```bash
mvn clean install
```

Si el comando termina en `BUILD SUCCESS`, ya tienes la Semana 1 funcionando dentro de la carpeta de la Semana 2, lista para que le agregues los cambios del **Paso 0** en adelante.

> **Si usas Git:** en vez de duplicar carpetas, una alternativa más limpia es crear una rama nueva desde el proyecto de la Semana 1 con `git checkout -b semana-2` y seguir trabajando en la misma carpeta. Así nunca tienes dos copias físicas del proyecto ni riesgo de arrastrar `target/` o `.idea/`, porque normalmente ya están excluidas en el `.gitignore`.

### Un par de detalles a tener en cuenta
- **Puerto ocupado:** Si alguna vez tienes la Semana 1 y la Semana 2 corriendo **al mismo tiempo**, solo una puede usar el puerto `8080`. Detén una antes de iniciar la otra, o cambia el puerto de una agregando `server.port=8081` en su `application.properties`.
- **Base de datos compartida:** Como ambas semanas apuntan al mismo servidor PostgreSQL (configurado en el **Paso 0** de esta guía), si usan la misma base `eventify`, van a leer y escribir exactamente las mismas tablas y registros. Si quieres mantener los datos de cada semana separados, crea una base de datos distinta para la Semana 2 (por ejemplo `eventify_semana2` desde DBeaver) y ajusta `spring.datasource.url` en su `application.properties`.

---

## 2. ¿Qué pide exactamente la Historia de Usuario?

Desglosemos los requerimientos en tareas concretas y directas:

### Requerido por la HU

1. **Modelado de Entidades y Repositorios (JPA):**
   - Transformar los POJOs `Event` y `Venue` en entidades JPA con `@Entity` y `@Table`.
   - Configurar claves primarias con `@Id` y generación automática con `@GeneratedValue`.
   - Aplicar restricciones de columna con `@Column(nullable = false, length = ...)`.
   - Crear interfaces `EventRepository` y `VenueRepository` que hereden de `JpaRepository`.
   - Implementar al menos una Consulta Derivada, como `findByNombreContaining`.

2. **Ciclo de vida CRUD y gestión de errores:**
   - Implementar `PUT` para actualizar registros existentes, validando que el recurso exista antes de modificarlo.
   - Implementar `DELETE` para el borrado físico de registros.
   - Implementar `GET /{id}` para consultar un único recurso.
   - Responder con `404 Not Found` cuando el ID solicitado no exista, en lugar de un error genérico `500`.

3. **Paginación, ordenamiento y calidad:**
   - Integrar `Pageable` y `Sort` en los endpoints de listado para soportar parámetros como `?page=0&size=10&sort=nombre,asc`.
   - Reflejar en Swagger los parámetros de paginación y los nuevos códigos de respuesta (`404`, `204`).
   - Implementar pruebas de integración con `@DataJpaTest` para validar que las entidades se guardan correctamente y que las consultas personalizadas funcionan contra el motor de base de datos.

---

## 3. ¿Qué vamos a crear?

Partimos de la estructura de la Semana 1 y la ampliamos. Los repositorios dejan de ser clases con listas en memoria y pasan a ser interfaces de Spring Data JPA; las entidades ganan anotaciones de persistencia; los servicios y controladores ganan las operaciones de actualizar, eliminar y consultar por ID.

```text
src/main/java/com/eventify/
├── EventifyApplication.java
├── config/
│   └── DataSeederConfig.java         <-- Sigue cargando datos iniciales, ahora vía JPA
├── controller/
│   ├── EventController.java          <-- Suma GET /{id}, PUT /{id}, DELETE /{id} y paginación
│   └── VenueController.java          <-- Suma GET /{id}, PUT /{id}, DELETE /{id} y paginación
├── exception/
│   ├── InvalidDataException.java     <-- Ya existía (HTTP 400)
│   └── ResourceNotFoundException.java <-- NUEVA: excepción de negocio con estado HTTP 404
├── model/
│   ├── Event.java                    <-- Ahora es una entidad JPA (@Entity)
│   └── Venue.java                    <-- Ahora es una entidad JPA (@Entity)
├── repository/
│   ├── EventRepository.java          <-- Ahora es una interfaz que extiende JpaRepository
│   └── VenueRepository.java          <-- Ahora es una interfaz que extiende JpaRepository
└── service/
    ├── EventService.java             <-- Suma findById, update y delete
    └── VenueService.java             <-- Suma findById, update y delete

src/main/resources/
└── application.properties            <-- NUEVO: configuración de conexión a PostgreSQL

src/test/java/com/eventify/
├── service/
│   ├── EventServiceTest.java         <-- Se amplía con pruebas de update/delete/findById
│   └── VenueServiceTest.java         <-- Se amplía con pruebas de update/delete/findById
└── repository/
    ├── EventRepositoryTest.java      <-- NUEVA: prueba de integración con @DataJpaTest
    └── VenueRepositoryTest.java      <-- NUEVA: prueba de integración con @DataJpaTest
```

### Responsabilidad de cada capa (actualizada)

| Capa | Responsabilidad principal | ¿Qué cambia en esta HU? |
| :--- | :--- | :--- |
| **Model** | Representar los datos del negocio | Pasan de ser POJOs simples a entidades JPA mapeadas a tablas reales. |
| **Repository** | Acceder y gestionar el almacenamiento | Dejan de ser clases con `List`; ahora son interfaces que heredan de `JpaRepository` y Spring genera la implementación. |
| **Service** | Reglas del negocio y validaciones | Además de validar y guardar, ahora orquesta actualizar, eliminar y consultar por ID, lanzando `404` cuando corresponde. |
| **Controller** | Punto de entrada HTTP | Suma verbos `PUT` y `DELETE`, y expone paginación mediante parámetros de query. |
| **Exception** | Señalizar fallos de negocio | Se suma `ResourceNotFoundException` para diferenciar "datos inválidos" (`400`) de "recurso inexistente" (`404`). |

---

## 4. Flujo de funcionamiento

Observemos cómo viaja la información al actualizar un evento existente, el escenario que mejor resume los cambios de esta semana:

```text
[Cliente / Postman / Swagger]
            │
            ▼ (1) Petición HTTP: PUT /api/events/5 con JSON
    [EventController]
            │
            ▼ (2) Llama a eventService.update(5, event)
     [EventService] ──── ¿Existe el ID 5? ───► NO ──► Lanza ResourceNotFoundException (HTTP 404)
            │ (SÍ, existe)
            ▼ (3) ¿Nombre vacío? ───► SÍ ──► Lanza InvalidDataException (HTTP 400)
            │ (NO, datos válidos)
            ▼ (4) Llama a eventRepository.save(eventoActualizado)
    [EventRepository extends JpaRepository]
            │
            ▼ (5) Hibernate genera el UPDATE SQL y lo ejecuta contra la base de datos
    [Base de datos]
            │
            ▼ (6) Confirma la actualización y retorna la entidad persistida
    [Retorno de datos]
            ▲
            └─ Repositorio devuelve el evento actualizado ──► Servicio devuelve al controlador ──► Retorna HTTP 200 OK con el JSON
```

---

## 5. Implementación paso a paso

---

### Paso 0 — Conectar el proyecto a una base de datos PostgreSQL real

#### Concepto necesario: ORM, Spring Data JPA y un cliente de base de datos
Un **ORM** (*Object-Relational Mapper*) es una herramienta que traduce automáticamente entre objetos Java y filas de una tabla relacional, para que no tengamos que escribir SQL a mano. **Hibernate** es el ORM que usa Spring por debajo; **Spring Data JPA** es la capa que nos entrega repositorios listos para usar sobre ese ORM, sin necesidad de implementar nosotros mismos el acceso a datos.

Esta semana usamos **PostgreSQL**, un motor de base de datos real que corre como un servidor independiente (a diferencia de H2, que puede vivir embebida dentro de la propia aplicación). Para crear la base de datos y revisar los datos de forma visual, usaremos **DBeaver**, un cliente gráfico universal de bases de datos: te permite conectarte a un servidor, ver las tablas, ejecutar consultas SQL y explorar los registros sin escribir código.

##### 0.1 — Levanta un servidor PostgreSQL
Necesitas PostgreSQL corriendo antes de continuar. La forma más rápida, sin instalar nada en tu sistema operativo, es con Docker:

```bash
docker run --name eventify-postgres \
  -e POSTGRES_USER=eventify \
  -e POSTGRES_PASSWORD=eventify \
  -e POSTGRES_DB=eventify \
  -p 5432:5432 \
  -d postgres:16
```

Esto levanta un contenedor de PostgreSQL 16 con la base de datos `eventify` ya creada, accesible en `localhost:5432`.

Si prefieres instalar PostgreSQL directamente (sin Docker), descárgalo desde postgresql.org, instálalo con el usuario y contraseña que prefieras, y crea la base de datos `eventify` manualmente en el siguiente paso, usando DBeaver.

##### 0.2 — Conéctate con DBeaver y confirma la base de datos
1. Abre DBeaver y crea una nueva conexión: `Database → New Database Connection → PostgreSQL`.
2. Completa los datos de conexión:
   - **Host:** `localhost`
   - **Port:** `5432`
   - **Database:** `eventify` (o `postgres` si aún no la has creado y necesitas conectarte primero para crearla)
   - **Username:** `eventify` (o el que hayas definido al instalar PostgreSQL)
   - **Password:** `eventify` (o la que hayas definido)
3. Da clic en **Test Connection**. Si todo está bien, DBeaver te confirmará la conexión exitosa.
4. Si usaste el comando de Docker de arriba, la base de datos `eventify` ya existe y puedes continuar. Si instalaste PostgreSQL manualmente y aún no la tienes, créala con clic derecho sobre el servidor en el panel izquierdo → `Create → Database`, y nómbrala `eventify`.

Deja la conexión de DBeaver abierta: la usaremos más adelante para confirmar visualmente que las tablas y los datos se están creando correctamente.

##### 0.3 — Agrega las dependencias al proyecto
Abre `pom.xml` y añade:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

> **¿Por qué seguimos agregando H2 si ya no la usamos para desarrollar?** La dejamos, pero solo con `scope=test`. Como vas a ver en el **Paso 8**, Spring Boot detecta que H2 está disponible únicamente durante las pruebas y la usa automáticamente como base de datos embebida para `@DataJpaTest`, sin tocar nunca tu base PostgreSQL real. Así las pruebas siguen siendo rápidas y aisladas, y los datos que ves en DBeaver no se alteran cuando corres `mvn test`.

##### 0.4 — Configura la conexión en application.properties
Crea el archivo `src/main/resources/application.properties` con la configuración de conexión:

```properties
# Conexión a la base de datos PostgreSQL real
spring.datasource.url=jdbc:postgresql://localhost:5432/eventify
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.username=eventify
spring.datasource.password=eventify

# Hibernate crea o actualiza las tablas automáticamente según las entidades
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

Ajusta `username` y `password` si usaste valores distintos al crear tu base de datos en el Paso 0.1/0.2.

**Qué logramos:** El proyecto ya sabe cómo conectarse a un servidor PostgreSQL real, y tienes DBeaver listo para inspeccionar visualmente las tablas y los datos que Hibernate creará automáticamente en el siguiente paso.

---

### Paso 1 — Convertir los modelos en entidades JPA

#### Concepto necesario: de POJO a Entidad
En la Semana 1, `Event` y `Venue` eran objetos que solo existían mientras la aplicación estuviera encendida. Ahora necesitamos decirle a Hibernate que cada instancia de estas clases corresponde a una fila de una tabla concreta.

### @Entity
**Explicación sencilla:** Le dice a Hibernate: *"Esta clase representa una tabla en la base de datos"*.
**Explicación técnica:** Marca la clase como una entidad JPA administrada por el contexto de persistencia; cada instancia gestionada corresponde a una fila y sus atributos a columnas.
**¿Qué cambia en nuestro proyecto?** Ya podemos guardar objetos `Event` directamente en la base de datos sin escribir SQL.
**Comentario mental:** *"Esta clase ya no es solo un objeto Java, es también una fila de una tabla."*

### @Table(name = "...")
**Explicación sencilla:** Indica el nombre exacto de la tabla asociada a la entidad.
**Explicación técnica:** Sin esta anotación, Hibernate usaría por defecto el nombre de la clase; `@Table` permite fijar explícitamente el nombre y evitar ambigüedades.
**¿Qué cambia en nuestro proyecto?** Controlamos que la tabla se llame `events` o `venues` en lugar de depender del nombre de la clase Java.
**Comentario mental:** *"Esta es la tabla exacta donde vivirán mis datos."*

### @Id y @GeneratedValue
**Explicación sencilla:** `@Id` marca cuál atributo es la clave primaria; `@GeneratedValue` le dice a la base de datos que genere ese valor automáticamente.
**Explicación técnica:** `@GeneratedValue(strategy = GenerationType.IDENTITY)` delega en la base de datos la generación del identificador mediante una columna autoincremental, reemplazando el `idCounter` manual que usábamos en la Semana 1.
**¿Qué cambia en nuestro proyecto?** Ya no necesitamos gestionar contadores nosotros mismos; la base de datos asigna el siguiente ID disponible.
**Comentario mental:** *"La base de datos decide el ID, yo no tengo que contarlos."*

> **Nota:** `IDENTITY` funciona bien aquí porque PostgreSQL (desde la versión 10) soporta columnas autoincrementales nativas. Si en algún momento cambiaras a un motor que no las soporte (por ejemplo, versiones antiguas de Oracle), tocaría usar `GenerationType.SEQUENCE` en su lugar.

### @Column(nullable = false, length = ...)
**Explicación sencilla:** Define reglas para una columna, como que no pueda quedar vacía o cuántos caracteres admite.
**Explicación técnica:** Hibernate traduce estos atributos en restricciones `NOT NULL` y `VARCHAR(n)` al generar el esquema de la tabla, añadiendo una capa de integridad a nivel de base de datos, además de la validación que ya hace el `Service`.
**¿Qué cambia en nuestro proyecto?** La integridad de los datos queda protegida en dos niveles: la lógica de negocio y la propia base de datos.
**Comentario mental:** *"Aunque alguien se salte el Service, la base de datos también pone límites."*

#### Código: `src/main/java/com/eventify/model/Event.java`

```java
package com.eventify.model;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Entity
@Table(name = "events")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Event {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 150)
    private String nombre;

    @Column(nullable = false)
    private String fecha;

    @Column(length = 500)
    private String descripcion;
}
```

#### Código: `src/main/java/com/eventify/model/Venue.java`

```java
package com.eventify.model;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Entity
@Table(name = "venues")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Venue {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 150)
    private String nombre;

    @Column(nullable = false, length = 250)
    private String direccion;

    @Column(nullable = false)
    private Integer capacidad;
}
```

**Qué logramos:** `Event` y `Venue` ahora son entidades reales, mapeadas a tablas concretas con restricciones de integridad.

---

### Paso 2 — Reemplazar los repositorios manuales por JpaRepository

#### Concepto necesario: de List manual a JpaRepository
En la Semana 1 escribimos a mano los métodos `save` y `findAll` usando una `List`. Spring Data JPA elimina ese trabajo repetitivo: basta con declarar una interfaz que extienda `JpaRepository` para obtener automáticamente `save`, `findAll`, `findById`, `deleteById`, paginación, y más.

### JpaRepository<T, ID>
**Explicación sencilla:** Es una interfaz que ya trae los métodos más comunes para guardar, buscar, actualizar y eliminar, sin que tengamos que escribir su implementación.
**Explicación técnica:** En tiempo de arranque, Spring Data JPA genera dinámicamente una implementación *proxy* de la interfaz, respaldada por Hibernate, que traduce cada método en las operaciones JDBC/SQL correspondientes.
**¿Qué cambia en nuestro proyecto?** La clase `EventRepository` deja de ser una clase con una `List`; ahora es solo una interfaz vacía (o casi vacía) y Spring hace el resto.
**Comentario mental:** *"Yo declaro el contrato, Spring escribe la implementación."*

### Consultas Derivadas (Derived Queries)
**Explicación sencilla:** Son métodos cuyo nombre describe la consulta que queremos, y Spring la construye automáticamente a partir de ese nombre.
**Explicación técnica:** Spring Data JPA interpreta el nombre del método (`findByNombreContaining`) siguiendo una convención de palabras clave (`findBy`, `Containing`, `OrderBy`, etc.) y genera la consulta JPQL equivalente sin que escribamos SQL.
**¿Qué cambia en nuestro proyecto?** Podemos buscar eventos o lugares por coincidencia parcial de nombre con una sola línea de código.
**Comentario mental:** *"El nombre del método es, literalmente, la consulta."*

#### Código: `src/main/java/com/eventify/repository/EventRepository.java`

```java
package com.eventify.repository;

import com.eventify.model.Event;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;

public interface EventRepository extends JpaRepository<Event, Long> {

    Page<Event> findByNombreContaining(String nombre, Pageable pageable);
}
```

#### Código: `src/main/java/com/eventify/repository/VenueRepository.java`

```java
package com.eventify.repository;

import com.eventify.model.Venue;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;

public interface VenueRepository extends JpaRepository<Venue, Long> {

    Page<Venue> findByNombreContaining(String nombre, Pageable pageable);
}
```

**Qué logramos:** Repositorios funcionales conectados a una base de datos real, con soporte de paginación y una consulta derivada, sin escribir una sola línea de SQL.

---

### Paso 3 — Crear la excepción para recursos inexistentes

#### Concepto necesario: distinguir "dato inválido" de "recurso inexistente"
`InvalidDataException` (Semana 1) responde con `400` cuando el cliente envía datos incorrectos. Ahora necesitamos una excepción distinta para cuando el cliente pide un recurso que simplemente no existe, como un evento con `ID: 9999`. Ese caso corresponde al código `404 Not Found`, no a un `400` ni a un error interno.

#### Código: `src/main/java/com/eventify/exception/ResourceNotFoundException.java`

```java
package com.eventify.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String mensaje) {
        super(mensaje);
    }
}
```

**Qué logramos:** Un mecanismo claro para que cualquier intento de consultar, actualizar o eliminar un ID inexistente termine automáticamente en un `404 Not Found` con un mensaje descriptivo.

---

### Paso 4 — Ampliar la capa de negocio con el CRUD completo

#### Concepto necesario: el Service como guardián del ciclo de vida completo
Hasta ahora el `Service` solo sabía guardar y listar. Para un CRUD completo, también debe saber **buscar por ID**, **actualizar** (validando primero que el recurso exista) y **eliminar**. El patrón se repite: primero verificar que el recurso exista (si no, `404`), luego aplicar las validaciones de datos que ya conocíamos (si fallan, `400`), y solo entonces tocar el repositorio.

#### Código: `src/main/java/com/eventify/service/EventService.java`

```java
package com.eventify.service;

import com.eventify.exception.InvalidDataException;
import com.eventify.exception.ResourceNotFoundException;
import com.eventify.model.Event;
import com.eventify.repository.EventRepository;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;

@Service
public class EventService {

    private final EventRepository eventRepository;

    public EventService(EventRepository eventRepository) {
        this.eventRepository = eventRepository;
    }

    public Event save(Event event) {
        validar(event);
        return eventRepository.save(event);
    }

    public Page<Event> findAll(Pageable pageable) {
        return eventRepository.findAll(pageable);
    }

    public Event findById(Long id) {
        return eventRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Evento no encontrado con id: " + id));
    }

    public Event update(Long id, Event datosActualizados) {
        Event eventoExistente = findById(id);
        validar(datosActualizados);

        eventoExistente.setNombre(datosActualizados.getNombre());
        eventoExistente.setFecha(datosActualizados.getFecha());
        eventoExistente.setDescripcion(datosActualizados.getDescripcion());

        return eventRepository.save(eventoExistente);
    }

    public void delete(Long id) {
        Event evento = findById(id);
        eventRepository.delete(evento);
    }

    private void validar(Event event) {
        if (event.getNombre() == null || event.getNombre().trim().isEmpty()) {
            throw new InvalidDataException("El nombre del evento no puede estar vacío");
        }
    }
}
```

#### Código: `src/main/java/com/eventify/service/VenueService.java`

```java
package com.eventify.service;

import com.eventify.exception.InvalidDataException;
import com.eventify.exception.ResourceNotFoundException;
import com.eventify.model.Venue;
import com.eventify.repository.VenueRepository;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;

@Service
public class VenueService {

    private final VenueRepository venueRepository;

    public VenueService(VenueRepository venueRepository) {
        this.venueRepository = venueRepository;
    }

    public Venue save(Venue venue) {
        validar(venue);
        return venueRepository.save(venue);
    }

    public Page<Venue> findAll(Pageable pageable) {
        return venueRepository.findAll(pageable);
    }

    public Venue findById(Long id) {
        return venueRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Lugar no encontrado con id: " + id));
    }

    public Venue update(Long id, Venue datosActualizados) {
        Venue venueExistente = findById(id);
        validar(datosActualizados);

        venueExistente.setNombre(datosActualizados.getNombre());
        venueExistente.setDireccion(datosActualizados.getDireccion());
        venueExistente.setCapacidad(datosActualizados.getCapacidad());

        return venueRepository.save(venueExistente);
    }

    public void delete(Long id) {
        Venue venue = findById(id);
        venueRepository.delete(venue);
    }

    private void validar(Venue venue) {
        if (venue.getNombre() == null || venue.getNombre().trim().isEmpty()) {
            throw new InvalidDataException("El nombre del lugar no puede estar vacío");
        }
    }
}
```

**Qué logramos:** Los servicios ahora cubren el ciclo de vida completo del recurso (crear, listar, consultar, actualizar, eliminar) y aplican la regla de negocio correcta en cada caso: `400` para datos inválidos, `404` para recursos inexistentes.

---

### Paso 5 — Ampliar los controladores con el CRUD y la paginación

#### Concepto necesario: Pageable y Sort
Cuando un catálogo crece, devolver todos los registros de una sola vez se vuelve costoso tanto para el servidor como para quien consume la API. La solución es entregar los resultados **por páginas**.

### Pageable
**Explicación sencilla:** Es un objeto que agrupa "qué página quiero" y "cuántos elementos por página", además de cómo ordenarlos.
**Explicación técnica:** Spring MVC puede construir automáticamente un `Pageable` a partir de los parámetros de query `page`, `size` y `sort` de la petición HTTP, sin que el desarrollador tenga que parsearlos manualmente.
**¿Qué cambia en nuestro proyecto?** El cliente puede pedir `GET /api/events?page=0&size=5&sort=nombre,asc` y el controlador recibe ese objeto listo para usar.
**Comentario mental:** *"El cliente pide una porción de la lista, no la lista entera."*

### @PageableDefault
**Explicación sencilla:** Define valores por defecto para la paginación cuando el cliente no los especifica.
**Explicación técnica:** Si la petición no incluye `size`, Spring usaría 20 por defecto; con `@PageableDefault(size = 10)` fijamos un valor más adecuado para nuestro catálogo.
**¿Qué cambia en nuestro proyecto?** Aun si alguien llama a `GET /api/events` sin parámetros, la respuesta ya viene paginada de forma razonable.
**Comentario mental:** *"Si no me dicen cómo paginar, yo ya sé cómo hacerlo por defecto."*

### ResponseEntity, @PathVariable y HttpStatus.NO_CONTENT
**Explicación sencilla:** `@PathVariable` extrae un valor de la URL (como el `id` en `/api/events/5`); `HttpStatus.NO_CONTENT` (`204`) le dice al cliente "la operación funcionó, pero no hay nada que devolver", que es exactamente lo que corresponde tras un `DELETE`.
**Explicación técnica:** `@PathVariable` vincula un segmento de la ruta a un parámetro del método; el código `204` es el estándar REST para confirmar una eliminación exitosa sin cuerpo de respuesta.
**¿Qué cambia en nuestro proyecto?** El endpoint `DELETE /api/events/{id}` queda alineado con las buenas prácticas REST descritas en los criterios de aceptación.
**Comentario mental:** *"Borré el recurso con éxito, no tengo nada más que mostrarte."*

#### Código: `src/main/java/com/eventify/controller/EventController.java`

```java
package com.eventify.controller;

import com.eventify.model.Event;
import com.eventify.service.EventService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/events")
@Tag(name = "Eventos", description = "Operaciones para registrar, consultar, actualizar y eliminar eventos")
public class EventController {

    private final EventService eventService;

    public EventController(EventService eventService) {
        this.eventService = eventService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @Operation(summary = "Registrar un nuevo evento", description = "Valida y almacena un evento en la base de datos")
    public Event create(@RequestBody Event event) {
        return eventService.save(event);
    }

    @GetMapping
    @Operation(
            summary = "Listar eventos de forma paginada",
            description = "Soporta los parámetros 'page', 'size' y 'sort' (ej: ?page=0&size=10&sort=nombre,asc)"
    )
    public Page<Event> getAll(@PageableDefault(size = 10) Pageable pageable) {
        return eventService.findAll(pageable);
    }

    @GetMapping("/{id}")
    @Operation(summary = "Consultar un evento por ID", description = "Retorna 404 Not Found si el evento no existe")
    public Event getById(@Parameter(description = "ID del evento a consultar") @PathVariable Long id) {
        return eventService.findById(id);
    }

    @PutMapping("/{id}")
    @Operation(
            summary = "Actualizar un evento existente",
            description = "Valida que el evento exista antes de actualizar; retorna 404 Not Found si no existe"
    )
    public Event update(@PathVariable Long id, @RequestBody Event event) {
        return eventService.update(id, event);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    @Operation(
            summary = "Eliminar un evento",
            description = "Elimina el registro de forma definitiva; retorna 404 Not Found si el ID no existe"
    )
    public void delete(@PathVariable Long id) {
        eventService.delete(id);
    }
}
```

#### Código: `src/main/java/com/eventify/controller/VenueController.java`

```java
package com.eventify.controller;

import com.eventify.model.Venue;
import com.eventify.service.VenueService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/venues")
@Tag(name = "Lugares", description = "Operaciones para registrar, consultar, actualizar y eliminar lugares (venues)")
public class VenueController {

    private final VenueService venueService;

    public VenueController(VenueService venueService) {
        this.venueService = venueService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @Operation(summary = "Registrar un nuevo lugar", description = "Valida y almacena un lugar en la base de datos")
    public Venue create(@RequestBody Venue venue) {
        return venueService.save(venue);
    }

    @GetMapping
    @Operation(
            summary = "Listar lugares de forma paginada",
            description = "Soporta los parámetros 'page', 'size' y 'sort' (ej: ?page=0&size=10&sort=nombre,asc)"
    )
    public Page<Venue> getAll(@PageableDefault(size = 10) Pageable pageable) {
        return venueService.findAll(pageable);
    }

    @GetMapping("/{id}")
    @Operation(summary = "Consultar un lugar por ID", description = "Retorna 404 Not Found si el lugar no existe")
    public Venue getById(@Parameter(description = "ID del lugar a consultar") @PathVariable Long id) {
        return venueService.findById(id);
    }

    @PutMapping("/{id}")
    @Operation(
            summary = "Actualizar un lugar existente",
            description = "Valida que el lugar exista antes de actualizar; retorna 404 Not Found si no existe"
    )
    public Venue update(@PathVariable Long id, @RequestBody Venue venue) {
        return venueService.update(id, venue);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    @Operation(
            summary = "Eliminar un lugar",
            description = "Elimina el registro de forma definitiva; retorna 404 Not Found si el ID no existe"
    )
    public void delete(@PathVariable Long id) {
        venueService.delete(id);
    }
}
```

**Qué logramos:** Endpoints REST completos con las cinco operaciones del CRUD, paginación configurable desde la URL, y códigos de estado HTTP alineados con las buenas prácticas (`200`, `201`, `204`, `404`).

---

### Paso 6 — Ajustar el Seeder a la nueva capa de persistencia

El `DataSeederConfig` de la Semana 1 no necesita cambios de lógica: sigue llamando a `eventService.save(...)` y `venueService.save(...)`. La diferencia real ocurre por debajo: ahora esas llamadas terminan ejecutando un `INSERT` real contra la base de datos PostgreSQL, en lugar de agregar un elemento a una `List` en memoria.

#### Código: `src/main/java/com/eventify/config/DataSeederConfig.java`

```java
package com.eventify.config;

import com.eventify.model.Event;
import com.eventify.model.Venue;
import com.eventify.service.EventService;
import com.eventify.service.VenueService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DataSeederConfig {

    @Bean
    public boolean seedInitialData(EventService eventService, VenueService venueService) {
        venueService.save(new Venue(null, "Centro de Convenciones Principal", "Av. El Sol 123", 500));
        venueService.save(new Venue(null, "Auditorio Tecnológico", "Calle Innovación 456", 150));

        eventService.save(new Event(null, "Conferencia Tech 2026", "2026-10-15", "Encuentro anual de desarrollo"));
        eventService.save(new Event(null, "Workshop Spring Boot", "2026-11-20", "Taller práctico de backend"));

        return true;
    }
}
```

> **Nota:** Como ahora los datos se guardan en un servidor PostgreSQL real, si reinicias la aplicación varias veces el Seeder seguirá intentando insertar los mismos registros iniciales cada vez, generando duplicados. Puedes comprobarlo abriendo DBeaver, entrando a la tabla `events` y viendo cómo crece con cada reinicio. Para probar la persistencia real (Escenario 1) esto es justamente lo esperado: lo importante es verificar que los registros creados manualmente durante una sesión sigan existiendo después de reiniciar.

**Qué logramos:** Los datos de prueba ahora quedan escritos físicamente en la base de datos desde el primer arranque.

---

### Paso 7 — Documentar los nuevos comportamientos en Swagger

Las anotaciones `@Operation` que agregamos en el **Paso 5** ya explican, en lenguaje natural, qué hace cada endpoint y cuándo puede responder `404`. Adicionalmente, usamos `@Parameter` para describir el significado del `id` en la URL, de modo que quien explore la API en Swagger entienda de inmediato qué dato debe enviar.

Gracias a que `springdoc-openapi-starter-webmvc-ui` inspecciona automáticamente los tipos de retorno de los métodos, `Page<Event>` y `Page<Venue>` se documentan solos, mostrando en la interfaz de Swagger los campos de metadatos de paginación (`totalElements`, `totalPages`, `number`, `size`, entre otros) sin configuración adicional.

Puedes seguir accediendo a la documentación interactiva en:
`http://localhost:8080/swagger-ui/index.html`

---

### Paso 8 — Pruebas de integración con @DataJpaTest

#### Concepto necesario: probar contra una base de datos real
Las pruebas de la Semana 1 verificaban la lógica del `Service` simulando el repositorio con Mockito, sin tocar ninguna base de datos. Ahora necesitamos comprobar algo distinto: que las entidades realmente se guardan bien y que las consultas derivadas, como `findByNombreContaining`, funcionan tal como esperamos contra un motor de base de datos de verdad.

### @DataJpaTest
**Explicación sencilla:** Levanta únicamente la parte de Spring relacionada con JPA (repositorios, entidades, la base de datos), sin arrancar toda la aplicación.
**Explicación técnica:** Configura un contexto de prueba reducido, habilita los repositorios de Spring Data JPA y, por defecto, sustituye la base de datos configurada por una base de datos en memoria embebida, envolviendo cada prueba en una transacción que se revierte al finalizar para mantener las pruebas aisladas entre sí.
**¿Qué cambia en nuestro proyecto?** Podemos verificar que `EventRepository` guarda y consulta correctamente, en milisegundos, sin necesidad de levantar el servidor web completo ni depender del archivo `./data/eventifydb`.
**Comentario mental:** *"Solo enciendo la parte de Spring que habla con la base de datos, nada más."*

#### Código: `src/test/java/com/eventify/repository/EventRepositoryTest.java`

```java
package com.eventify.repository;

import com.eventify.model.Event;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
class EventRepositoryTest {

    @Autowired
    private EventRepository eventRepository;

    @Test
    void save_PersistsEventWithGeneratedId() {
        // Arrange
        Event event = new Event(null, "Charla de Arquitectura", "2026-12-01", "Sesión técnica sobre diseño de software");

        // Act
        Event saved = eventRepository.save(event);

        // Assert
        assertNotNull(saved.getId());
        assertEquals("Charla de Arquitectura", saved.getNombre());
    }

    @Test
    void findByNombreContaining_ReturnsOnlyMatchingEvents() {
        // Arrange
        eventRepository.save(new Event(null, "Conferencia Java", "2026-10-10", "Desc"));
        eventRepository.save(new Event(null, "Workshop Java Avanzado", "2026-10-15", "Desc"));
        eventRepository.save(new Event(null, "Taller de Python", "2026-10-20", "Desc"));
        Pageable pageable = PageRequest.of(0, 10);

        // Act
        Page<Event> resultado = eventRepository.findByNombreContaining("Java", pageable);

        // Assert
        assertEquals(2, resultado.getTotalElements());
    }

    @Test
    void deleteById_RemovesEventFromDatabase() {
        // Arrange
        Event saved = eventRepository.save(new Event(null, "Evento temporal", "2026-09-01", "Desc"));
        Long id = saved.getId();

        // Act
        eventRepository.deleteById(id);

        // Assert
        assertTrue(eventRepository.findById(id).isEmpty());
    }
}
```

#### Código: `src/test/java/com/eventify/repository/VenueRepositoryTest.java`

```java
package com.eventify.repository;

import com.eventify.model.Venue;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
class VenueRepositoryTest {

    @Autowired
    private VenueRepository venueRepository;

    @Test
    void save_PersistsVenueWithGeneratedId() {
        // Arrange
        Venue venue = new Venue(null, "Sala Norte", "Carrera 10 #20-30", 80);

        // Act
        Venue saved = venueRepository.save(venue);

        // Assert
        assertNotNull(saved.getId());
        assertEquals(80, saved.getCapacidad());
    }

    @Test
    void findByNombreContaining_ReturnsMatchingVenues() {
        // Arrange
        venueRepository.save(new Venue(null, "Auditorio Central", "Dir 1", 200));
        venueRepository.save(new Venue(null, "Auditorio Norte", "Dir 2", 100));
        venueRepository.save(new Venue(null, "Salón Sur", "Dir 3", 50));
        Pageable pageable = PageRequest.of(0, 10);

        // Act
        Page<Venue> resultado = venueRepository.findByNombreContaining("Auditorio", pageable);

        // Assert
        assertEquals(2, resultado.getTotalElements());
    }
}
```

También ampliamos las pruebas de servicio de la Semana 1 (con Mockito, sin tocar la base de datos) para cubrir `findById`, `update` y `delete`:

#### Fragmento adicional en `src/test/java/com/eventify/service/EventServiceTest.java`

```java
    @Test
    void findById_ExistingId_ReturnsEvent() {
        // Arrange
        Event event = new Event(1L, "Conferencia Java", "2026-10-10", "Desc");
        when(eventRepository.findById(1L)).thenReturn(java.util.Optional.of(event));

        // Act
        Event result = eventService.findById(1L);

        // Assert
        assertEquals("Conferencia Java", result.getNombre());
    }

    @Test
    void findById_NonExistingId_ThrowsResourceNotFoundException() {
        // Arrange
        when(eventRepository.findById(99L)).thenReturn(java.util.Optional.empty());

        // Act & Assert
        assertThrows(ResourceNotFoundException.class, () -> eventService.findById(99L));
    }

    @Test
    void delete_NonExistingId_ThrowsResourceNotFoundException() {
        // Arrange
        when(eventRepository.findById(99L)).thenReturn(java.util.Optional.empty());

        // Act & Assert
        assertThrows(ResourceNotFoundException.class, () -> eventService.delete(99L));
        verify(eventRepository, never()).delete(any());
    }
```

(No olvides agregar `import com.eventify.exception.ResourceNotFoundException;` al inicio del archivo de prueba.)

**Qué logramos:** Dos niveles de pruebas complementarios: pruebas rápidas de servicio con mocks (lógica de negocio) y pruebas de integración con `@DataJpaTest` (persistencia real contra el motor de base de datos).

---

## 6. ¿Cómo funciona todo junto?

Repasemos el ciclo de vida completo considerando los cambios de esta semana:

1. **Arranque:** Spring Boot lee la configuración de `application.properties`, se conecta al servidor PostgreSQL en `jdbc:postgresql://localhost:5432/eventify` y, gracias a `spring.jpa.hibernate.ddl-auto=update`, Hibernate crea o actualiza las tablas `events` y `venues` según las entidades anotadas con `@Entity`.
2. **Construcción de Beans:** Spring Data JPA genera automáticamente la implementación de `EventRepository` y `VenueRepository`. Los servicios y controladores se conectan igual que en la Semana 1, por inyección de constructor.
3. **Carga de Datos (Seeder):** El `@Bean` de `DataSeederConfig` inserta los registros iniciales, esta vez con `INSERT` reales contra la base de datos.
4. **Consulta paginada:** Un cliente envía `GET /api/events?page=0&size=5&sort=nombre,asc`. Spring MVC construye un `Pageable` a partir de esos parámetros y lo pasa al controlador, que delega en `eventService.findAll(pageable)`, que a su vez delega en `eventRepository.findAll(pageable)`. Hibernate traduce esto en una consulta SQL con `LIMIT` y `ORDER BY`.
5. **Actualización:** Un cliente envía `PUT /api/events/5`. El controlador delega en `eventService.update(5, event)`, que primero busca el recurso (`findById`); si no existe, lanza `ResourceNotFoundException` y Spring responde `404`. Si existe, valida los nuevos datos y guarda los cambios; Hibernate genera el `UPDATE` correspondiente.
6. **Eliminación:** Un cliente envía `DELETE /api/events/5`. El controlador delega en `eventService.delete(5)`, que verifica primero que el evento exista (mismo mecanismo de `404`) y luego llama a `eventRepository.delete(evento)`. Hibernate ejecuta el `DELETE` físico y el controlador responde `204 No Content`.
7. **Persistencia real:** Como los datos viven en un servidor PostgreSQL y no en una `List` en memoria, apagar y volver a encender la aplicación no borra la información: al reiniciar, Hibernate se reconecta al mismo servidor y los registros siguen ahí. Puedes confirmarlo en cualquier momento abriendo DBeaver y consultando directamente la tabla `events`.

---

## 7. Cómo probar la Historia de Usuario

Inicia la aplicación con:

```bash
mvn spring-boot:run
```

Abre Swagger en:
`http://localhost:8080/swagger-ui/index.html`

---

### Prueba del Escenario 1: Persistencia Post-Reinicio (Camino Feliz)

- **Objetivo:** Confirmar que los datos sobreviven a un reinicio de la aplicación.
- **Pasos:**
  1. Crea un evento con `POST /api/events` y anota el `id` recibido.
  2. Detén la aplicación (`Ctrl + C`).
  3. Vuelve a iniciarla con `mvn spring-boot:run`.
  4. Consulta `GET /api/events/{id}` con el mismo ID.
- **Resultado esperado:** El evento sigue existiendo con los mismos datos, sin necesidad de volver a crearlo.

---

### Prueba del Escenario 2: Intento de Acceso a Recurso Inexistente (Camino de Error)

- **Objetivo:** Validar que un ID inexistente responda `404` en las tres operaciones que lo usan.
- **Método y URL:** `GET`, `PUT` o `DELETE` sobre `/api/events/9999`
- **Resultado esperado:** Código HTTP `404 Not Found` con un mensaje indicando que el evento no fue encontrado. Ninguna de las tres operaciones debe modificar datos existentes.

---

### Prueba del Escenario 3: Paginación de Resultados (Caso de Volumen)

- **Objetivo:** Verificar que el listado paginado entregue exactamente los registros solicitados junto con los metadatos correspondientes.
- **Pasos:**
  1. Registra (o deja que el Seeder registre) al menos 50 eventos.
  2. Ejecuta `GET /api/events?page=0&size=5`.
- **Resultado esperado:** Código HTTP `200 OK` con un cuerpo que incluye exactamente 5 elementos en `content`, y metadatos como:
  ```json
  {
    "content": [ /* 5 eventos */ ],
    "totalElements": 50,
    "totalPages": 10,
    "number": 0,
    "size": 5
  }
  ```

---

### Prueba del Escenario 4: Eliminación Exitosa

- **Objetivo:** Confirmar que borrar un evento existente responde `204 No Content` y que el recurso deja de estar disponible.
- **Pasos:**
  1. Ejecuta `DELETE /api/events/{id}` sobre un evento existente.
  2. Verifica la respuesta.
  3. Intenta `GET /api/events/{id}` con el mismo ID.
- **Resultado esperado:** El `DELETE` responde `204 No Content` sin cuerpo. El `GET` posterior responde `404 Not Found`, confirmando que el recurso ya no existe.

---

### Ejecución de pruebas automatizadas

```bash
mvn test
```

Resultado esperado en consola (los números exactos dependerán de cuántas pruebas hayas agregado):
```text
[INFO] Tests run: 6, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: ... s -- in com.eventify.service.EventServiceTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: ... s -- in com.eventify.repository.EventRepositoryTest
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: ... s -- in com.eventify.repository.VenueRepositoryTest
[INFO] BUILD SUCCESS
```

---

## 8. Errores comunes

### 1. `org.hibernate.id.IdentifierGenerationException` o errores al insertar
- **Por qué ocurre:** Se envió un `Event` o `Venue` con un `id` distinto de `null` en una operación de creación, confundiendo a la estrategia `GenerationType.IDENTITY`.
- **Qué revisar:** Al crear un recurso nuevo, el campo `id` del JSON enviado debe omitirse o enviarse como `null`; solo se debe incluir un `id` real al actualizar (en la URL, no en el body).

### 2. `Connection to localhost:5432 refused` o `FATAL: database "eventify" does not exist`
- **Por qué ocurre:** El servidor PostgreSQL no está corriendo, o la base de datos `eventify` todavía no se creó (ver Paso 0.1/0.2).
- **Qué revisar:** Confirma que el contenedor Docker esté activo con `docker ps` (o que el servicio de PostgreSQL esté corriendo si lo instalaste directamente), y abre DBeaver para verificar que la conexión funcione y que la base `eventify` exista en el árbol de conexiones.

### 3. Un `PUT` o `DELETE` responde `500` en lugar de `404`
- **Por qué ocurre:** El `Service` llama directamente a `eventRepository.save(...)` o `eventRepository.delete(...)` sin verificar antes si el recurso existe.
- **Qué revisar:** Todo método de actualización o eliminación debe iniciar llamando a `findById(id)`, que ya se encarga de lanzar `ResourceNotFoundException` cuando corresponde.

### 4. `GET /api/events?sort=nombre,asc` no ordena los resultados
- **Por qué ocurre:** El parámetro `sort` hace referencia a un atributo que no existe en la entidad, o el controlador no está recibiendo un `Pageable` como parámetro.
- **Qué revisar:** El nombre después de `sort=` debe coincidir exactamente con un atributo de la entidad (por ejemplo, `nombre`, no `name`), y el método del controlador debe declarar `Pageable pageable` como parámetro para que Spring lo construya automáticamente.

### 5. Las pruebas con `@DataJpaTest` fallan o intentan conectarse a PostgreSQL
- **Por qué ocurre:** Falta la dependencia `h2` con `scope=test` en el `pom.xml`. Sin ella, Spring no tiene una base de datos embebida disponible y las pruebas intentan usar la configuración real de `application.properties`, apuntando a tu PostgreSQL.
- **Qué revisar:** Verifica que la dependencia de H2 esté declarada con `<scope>test</scope>` tal como se indicó en el **Paso 0.3**.

---

## 9. Checklist final

- [ ] `Event` y `Venue` están anotadas con `@Entity`, `@Table`, `@Id` y `@GeneratedValue`.
- [ ] Las columnas obligatorias usan `@Column(nullable = false, ...)`.
- [ ] `EventRepository` y `VenueRepository` son interfaces que extienden `JpaRepository` e incluyen `findByNombreContaining`.
- [ ] `ResourceNotFoundException` está anotada con `@ResponseStatus(HttpStatus.NOT_FOUND)`.
- [ ] Los servicios implementan `findById`, `update` y `delete`, verificando primero la existencia del recurso.
- [ ] Los controladores exponen `GET /{id}`, `PUT /{id}` y `DELETE /{id}`, con `204 No Content` en el borrado.
- [ ] Los endpoints de listado (`GET /api/events` y `GET /api/venues`) aceptan `page`, `size` y `sort`.
- [ ] Swagger refleja los nuevos endpoints y códigos de respuesta (`404`, `204`).
- [ ] Existen pruebas de integración con `@DataJpaTest` para ambos repositorios.
- [ ] Se verificaron los 4 escenarios de aceptación (persistencia post-reinicio, recurso inexistente, paginación y eliminación exitosa).

---

## 10. ¿Qué aprendimos?

1. **De la memoria a la persistencia real:** Entendimos la diferencia entre almacenar datos en una `List` que desaparece al apagar la aplicación, y persistirlos en una base de datos que sobrevive a los reinicios.
2. **Spring Data JPA reduce el código repetitivo:** Declarar una interfaz que extienda `JpaRepository` nos ahorra escribir a mano las operaciones básicas de acceso a datos, y las Consultas Derivadas nos permiten construir búsquedas específicas a partir del nombre del método.
3. **Separar los tipos de error importa:** Diferenciar `InvalidDataException` (`400`, datos incorrectos) de `ResourceNotFoundException` (`404`, recurso inexistente) hace que la API sea más clara y predecible para quien la consume.
4. **Paginación como buena práctica de escalabilidad:** Aprendimos a usar `Pageable` y `Sort` para que los listados sigan siendo eficientes incluso cuando el catálogo crece considerablemente.
5. **Dos niveles de pruebas:** Comprendimos cuándo usar pruebas de servicio con mocks (rápidas, para lógica de negocio) y cuándo usar `@DataJpaTest` (más lentas, pero necesarias para validar que la persistencia realmente funciona).

---

## 11. Mini repaso

1. ¿Qué diferencia hay entre `@Column(nullable = false)` en la entidad y la validación manual que hacemos en el `Service`? ¿Por qué conviene tener ambas?
2. ¿Por qué `GenerationType.IDENTITY` reemplaza la necesidad del `idCounter` manual que usábamos en la Semana 1?
3. ¿Qué ventaja tiene declarar `findByNombreContaining` como Consulta Derivada en lugar de escribir la consulta SQL a mano?
4. ¿Por qué el método `update` en el `Service` llama primero a `findById` antes de guardar los nuevos datos?
5. ¿Qué diferencia práctica existe entre una prueba de `Service` con `@Mock`/`@InjectMocks` y una prueba de repositorio con `@DataJpaTest`?