# Eventify

Guía de estudio e implementación paso a paso del cimiento arquitectónico de **Eventify**, aplicando el patrón Spring MVC, inyección de dependencias por constructor, estereotipos, almacenamiento en memoria, documentación OpenAPI/Swagger y pruebas unitarias con JUnit 5 y Mockito.

## 1. ¿Qué vamos a construir?

Vamos a construir la base arquitectónica de **Eventify**, una plataforma para gestionar eventos y lugares (*venues*). 

En esta primera etapa, implementaremos un catálogo interno que permita **registrar** y **consultar** eventos y lugares en memoria (sin base de datos física todavía). Crearemos una estructura limpia dividida en capas siguiendo el patrón **Spring MVC**, inyectaremos dependencias a través de constructores, documentaremos los endpoints con **Swagger/OpenAPI** y aseguraremos la lógica de negocio mediante **pruebas unitarias** aisladas.

---

## 2. ¿Qué pide exactamente la Historia de Usuario?

Desglosemos los requerimientos en tareas concretas y directas:

### Requerido por la HU

1. **Configuración del proyecto:**
   - Crear el proyecto con Spring Initializr incluyendo dependencias web (`spring-boot-starter-web`) y Lombok (`lombok`).
   - Agregar la librería para documentación OpenAPI/Swagger (`springdoc-openapi-starter-webmvc-ui`).
   - Usar inyección de dependencias estricta por constructor.
   - Cargar datos de prueba al iniciar la aplicación mediante una clase `@Configuration` con un método `@Bean`.

2. **Modelos de datos (POJOs):**
   - `Event`: con los campos `id`, `nombre`, `fecha` y `descripcion`.
   - `Venue`: con los campos `id`, `nombre`, `direccion` y `capacidad`.

3. **Capa de Acceso a Datos (Repository):**
   - Clases anotadas con `@Repository` que almacenen los datos en memoria utilizando colecciones (`List` / `ArrayList`).
   - Métodos mínimos para guardar (`save`) y listar (`findAll`).

4. **Capa de Negocio (Service):**
   - Clases anotadas con `@Service`.
   - Validar que el nombre no esté vacío ni sea nulo antes de guardar.
   - Si los datos son inválidos, rechazar la operación e impedir que lleguen al repositorio.

5. **Capa de Presentación (Controller):**
   - Clases anotadas con `@RestController`.
   - Exponer endpoints `POST /api/events` y `GET /api/events`.
   - Exponer endpoints `POST /api/venues` y `GET /api/venues`.
   - Responder con estado HTTP `201 Created` al registrar y `200 OK` al listar.

6. **Documentación:**
   - Swagger accesible en `/swagger-ui.html` con títulos y descripciones personalizadas para cada endpoint.

7. **Pruebas Unitarias:**
   - Probar los `@Service` de forma aislada con JUnit 5 y Mockito, sin levantar el contexto de Spring.
   - Validar camino feliz (registro exitoso), camino de error (nombre vacío) y consulta.

---

## 3. ¿Qué vamos a crear?

La aplicación sigue la arquitectura en capas estándar de Spring Boot:

```text
src/main/java/com/eventify/
├── EventifyApplication.java           <-- Clase principal que arranca la aplicación
├── config/
│   └── DataSeederConfig.java         <-- Carga datos iniciales con @Configuration y @Bean
├── controller/
│   ├── EventController.java          <-- Expone los endpoints REST para eventos
│   └── VenueController.java          <-- Expone los endpoints REST para lugares
├── exception/
│   └── InvalidDataException.java     <-- Excepción de negocio con estado HTTP 400
├── model/
│   ├── Event.java                    <-- Entidad/POJO del evento
│   └── Venue.java                    <-- Entidad/POJO del lugar
├── repository/
│   ├── EventRepository.java          <-- Almacenamiento en memoria para eventos
│   └── VenueRepository.java          <-- Almacenamiento en memoria para lugares
└── service/
    ├── EventService.java             <-- Lógica y validaciones para eventos
    └── VenueService.java             <-- Lógica y validaciones para lugares

src/test/java/com/eventify/service/
├── EventServiceTest.java             <-- Pruebas unitarias aisladas de EventService
└── VenueServiceTest.java             <-- Pruebas unitarias aisladas de VenueService
```

### Responsabilidad de cada capa

| Capa | Responsabilidad principal | ¿Qué hace en esta HU? |
| :--- | :--- | :--- |
| **Model** | Representar los datos del negocio | Define qué atributos componen un `Event` y un `Venue`. |
| **Repository** | Acceder y gestionar el almacenamiento | Guarda y recupera objetos de una lista en memoria (`List`). |
| **Service** | Reglas del negocio y validaciones | Verifica que los datos sean válidos antes de pasarlos al repositorio. |
| **Controller** | Punto de entrada HTTP | Recibe la petición web, llama al servicio y devuelve la respuesta HTTP adecuada. |
| **Config** | Configuración de Beans del sistema | Inicializa registros de prueba al arrancar la aplicación. |
| **Exception** | Señalizar fallos de negocio | Notifica cuando una regla se incumple y asocia el código HTTP `400 Bad Request`. |

---

## 4. Flujo de funcionamiento

Antes de escribir código, observemos cómo viaja la información dentro del sistema:

```text
[Cliente / Postman / Swagger]
            │
            ▼ (1) Petición HTTP: POST /api/events con JSON
    [EventController]
            │
            ▼ (2) Llama a eventService.save(event)
     [EventService] ──── ¿Nombre vacío? ───► SÍ ──► Lanza InvalidDataException (HTTP 400)
            │ (NO, datos válidos)
            ▼ (3) Llama a eventRepository.save(event)
    [EventRepository]
            │
            ▼ (4) Asigna ID autoincremental y guarda en List<Event>
    [Retorno de datos]
            ▲
            └─ Repositorio devuelve evento con ID ──► Servicio devuelve a Controlador ──► Retorna HTTP 201 Created con el JSON
```

---

## 5. Implementación paso a paso

---

### Paso 0 — Crear el proyecto Spring Boot

Para iniciar el proyecto desde cero, utilizaremos **Spring Initializr** ([start.spring.io](https://start.spring.io)) o el asistente de tu IDE preferido:

- **Project:** Maven
- **Language:** Java
- **Spring Boot:** 3.3.x (o la versión 3.x estable disponible)
- **Group:** `com.eventify`
- **Artifact:** `eventify`
- **Name:** `eventify`
- **Package name:** `com.eventify`
- **Packaging:** Jar
- **Java:** 17 o 21

#### Dependencias iniciales en Spring Initializr
1. **Spring Web (`spring-boot-starter-web`):** Proporciona las librerías necesarias para crear APIs REST basadas en Spring MVC y el servidor embebido Tomcat.
2. **Lombok (`lombok`):** Herramienta que genera constructores, getters y setters en tiempo de compilación para no escribir código repetitivo.

Una vez descargado y abierto el proyecto, abre el archivo `pom.xml` y añade la dependencia para Swagger/OpenAPI:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.6.0</version>
</dependency>
```

Esta dependencia genera automáticamente la interfaz gráfica interactiva de documentación sin requerir configuraciones complejas.

---

### Paso 1 — Crear los modelos (POJOs): Event y Venue

#### Concepto necesario: POJO y Entidad en memoria
Un **POJO** (*Plain Old Java Object*) es una clase Java común y corriente que sólo contiene atributos privados, constructores y métodos para acceder o modificar esos atributos (getters y setters). En esta fase no usamos base de datos relacional ni JPA, por lo que estos modelos representan los datos que viajan en memoria.

#### Concepto necesario: Anotaciones de Lombok
Para no ensuciar la clase con decenas de líneas de código repetitivo, usaremos las anotaciones precisas de Lombok.

### @Getter y @Setter
**Explicación sencilla:** Generan automáticamente los métodos para leer (`getCampo()`) y escribir (`setCampo()`) los valores de los atributos.  
**Explicación técnica:** El procesador de anotaciones de Lombok genera los métodos de acceso en el bytecode durante la compilación.  
**¿Qué cambia en nuestro proyecto?** Podemos usar `event.getNombre()` o `event.setId(...)` sin tener que escribir esos métodos a mano.  
**Comentario mental:** *"Acceso y modificación de datos garantizados sin escribir código de relleno."*

### @NoArgsConstructor y @AllArgsConstructor
**Explicación sencilla:** Generan un constructor vacío (sin parámetros) y un constructor con todos los atributos de la clase.  
**Explicación técnica:** `@NoArgsConstructor` crea `public Event() {}`, indispensable para que librerías como Jackson puedan convertir JSON a objetos Java. `@AllArgsConstructor` crea un constructor con todos los argumentos en orden de declaración.  
**¿Qué cambia en nuestro proyecto?** Disponemos de constructores listos para deserialización y para crear instancias de prueba rápidamente.  
**Comentario mental:** *"Dos constructores listos: uno vacío para frameworks y uno completo para nosotros."*

> **Nota sobre @Data:** Evitamos `@Data` porque genera métodos como `equals()`, `hashCode()` y `toString()` con comportamientos predeterminados que muchas veces no necesitamos o que pueden provocar efectos colaterales. Usar `@Getter`, `@Setter`, `@NoArgsConstructor` y `@AllArgsConstructor` es más explícito y seguro.

#### Código: `src/main/java/com/eventify/model/Event.java`

```java
package com.eventify.model;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Event {
    private Long id;
    private String nombre;
    private String fecha;
    private String descripcion;
}
```

#### Código: `src/main/java/com/eventify/model/Venue.java`

```java
package com.eventify.model;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Venue {
    private Long id;
    private String nombre;
    private String direccion;
    private Integer capacidad;
}
```

**Qué logramos:** Definimos la estructura fundamental de datos de Eventify.

---

### Paso 2 — Crear la excepción de validación

#### Concepto necesario: Excepción de Negocio y Códigos HTTP
Cuando un usuario envía datos incorrectos (como un nombre vacío), la aplicación no debe caerse con un error interno (`500 Internal Server Error`). Debe responder con un código HTTP que indique claramente que el cliente envió información errónea: `400 Bad Request`.

### @ResponseStatus
**Explicación sencilla:** Asocia una excepción de Java directamente con un código de respuesta HTTP.  
**Explicación técnica:** Le indica a Spring Web que, cuando esta excepción se propague fuera del controlador sin ser capturada manualmente, debe interceptarla y generar una respuesta con el código HTTP configurado.  
**¿Qué cambia en nuestro proyecto?** Al lanzar `throw new InvalidDataException(...)`, Spring responderá al cliente automáticamente con un estado HTTP 400 en lugar de un error 500.  
**Comentario mental:** *"Si lanzo esta excepción, devuélvele al cliente el código HTTP indicado."*

#### Código: `src/main/java/com/eventify/exception/InvalidDataException.java`

```java
package com.eventify.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.BAD_REQUEST)
public class InvalidDataException extends RuntimeException {
    public InvalidDataException(String mensaje) {
        super(mensaje);
    }
}
```

**Qué logramos:** Contamos con un mecanismo sencillo y directo para cortar la ejecución cuando una validación falle y retornar inmediatamente un estado HTTP 400.

---

### Paso 3 — Crear la capa de acceso a datos (@Repository en memoria)

#### Concepto necesario: Repository
**Explicación sencilla:** Es el componente encargado de almacenar, buscar y gestionar la información.  
**Explicación técnica:** Es un patrón de diseño que aísla el dominio del mecanismo de persistencia (sea una base de datos SQL, NoSQL o una colección en memoria).

### @Repository
**Explicación sencilla:** Le avisa a Spring: *"Esta clase se encarga de guardar y buscar datos, conviértela en un componente administrado (Bean)"*.  
**Explicación técnica:** Especialización de `@Component`. Registra la clase en el contenedor de inversión de control (IoC) de Spring y habilita la traducción automática de excepciones de persistencia.  
**¿Qué cambia en nuestro proyecto?** Spring creará una única instancia (*Singleton*) de esta clase para inyectarla donde sea solicitada.  
**Comentario mental:** *"Spring administra esta clase como la encargada de los datos."*

#### Manejo de IDs y colecciones
Para cumplir con la HU de persistencia temporal:
- Usamos `List<Event>` inicializada como `new ArrayList<>()`.
- Usamos un contador simple `private Long idCounter = 1L;` y luego `idCounter++`.
- No usamos estructuras concurrentes ni `AtomicLong` porque en esta etapa inicial no hay requerimientos de concurrencia y buscamos la solución más limpia y comprensible.

#### Código: `src/main/java/com/eventify/repository/EventRepository.java`

```java
package com.eventify.repository;

import com.eventify.model.Event;
import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.List;

@Repository
public class EventRepository {

    private final List<Event> events = new ArrayList<>();
    private Long idCounter = 1L;

    public Event save(Event event) {
        event.setId(idCounter++);
        events.add(event);
        return event;
    }

    public List<Event> findAll() {
        return events;
    }
}
```

#### Código: `src/main/java/com/eventify/repository/VenueRepository.java`

```java
package com.eventify.repository;

import com.eventify.model.Venue;
import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.List;

@Repository
public class VenueRepository {

    private final List<Venue> venues = new ArrayList<>();
    private Long idCounter = 1L;

    public Venue save(Venue venue) {
        venue.setId(idCounter++);
        venues.add(venue);
        return venue;
    }

    public List<Venue> findAll() {
        return venues;
    }
}
```

**Qué logramos:** Repositorios funcionales que pueden almacenar y consultar registros en memoria manteniendo un identificador único para cada elemento.

---

### Paso 4 — Crear la capa de negocio (@Service con validaciones)

#### Concepto necesario: Service y Reglas de Negocio
El **Service** es el cerebro de la aplicación. Aquí residen las decisiones lógicas: si los datos son correctos, si se cumplen las restricciones o cómo deben coordinarse distintos componentes. El controlador no debe validar reglas de negocio, y el repositorio solo debe encargarse de guardar.

### @Service
**Explicación sencilla:** Le indica a Spring: *"Esta clase contiene la lógica de negocio y debe ser administrada como un Bean"*.  
**Explicación técnica:** Es otra especialización de `@Component` pensada semánticamente para la capa de servicios.  
**¿Qué cambia en nuestro proyecto?** Spring la detecta automáticamente y permite inyectarla en los controladores o configuraciones.  
**Comentario mental:** *"Aquí vive la lógica de negocio administrada por Spring."*

#### Concepto necesario: Inyección de Dependencias por Constructor
En lugar de usar `@Autowired` sobre los atributos (lo que dificulta probar la clase sin levantar Spring), declaramos las dependencias como `private final` y creamos un constructor explícito:

```java
public EventService(EventRepository eventRepository) {
    this.eventRepository = eventRepository;
}
```

Cuando Spring arranca, detecta que `EventService` necesita un `EventRepository`, busca el bean existente en su contenedor y se lo pasa al constructor automáticamente.

#### Código: `src/main/java/com/eventify/service/EventService.java`

```java
package com.eventify.service;

import com.eventify.exception.InvalidDataException;
import com.eventify.model.Event;
import com.eventify.repository.EventRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class EventService {

    private final EventRepository eventRepository;

    public EventService(EventRepository eventRepository) {
        this.eventRepository = eventRepository;
    }

    public Event save(Event event) {
        if (event.getNombre() == null || event.getNombre().trim().isEmpty()) {
            throw new InvalidDataException("El nombre del evento no puede estar vacío");
        }
        return eventRepository.save(event);
    }

    public List<Event> findAll() {
        return eventRepository.findAll();
    }
}
```

#### Código: `src/main/java/com/eventify/service/VenueService.java`

```java
package com.eventify.service;

import com.eventify.exception.InvalidDataException;
import com.eventify.model.Venue;
import com.eventify.repository.VenueRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class VenueService {

    private final VenueRepository venueRepository;

    public VenueService(VenueRepository venueRepository) {
        this.venueRepository = venueRepository;
    }

    public Venue save(Venue venue) {
        if (venue.getNombre() == null || venue.getNombre().trim().isEmpty()) {
            throw new InvalidDataException("El nombre del lugar no puede estar vacío");
        }
        return venueRepository.save(venue);
    }

    public List<Venue> findAll() {
        return venueRepository.findAll();
    }
}
```

**Qué logramos:** Proteger los repositorios contra datos corruptos mediante validaciones estrictas y desacoplar las clases mediante inyección por constructor.

---

### Paso 5 — Crear los controladores REST (@RestController)

#### Concepto necesario: REST Controller
Es la puerta de entrada de la API. Traduce peticiones HTTP (GET, POST) a llamadas a métodos Java y transforma los objetos devueltos en formato JSON.

### @RestController
**Explicación sencilla:** Indica que esta clase atenderá llamadas web y que el resultado devuelto debe enviarse directamente en formato JSON.  
**Explicación técnica:** Combina `@Controller` y `@ResponseBody`.  
**¿Qué cambia en nuestro proyecto?** Todo método público devolverá datos serializados en JSON en el cuerpo de la respuesta HTTP.  
**Comentario mental:** *"Esta clase recibe llamadas HTTP y responde con datos JSON."*

### @RequestMapping("/api/...")
**Explicación sencilla:** Define el prefijo de la dirección URL común para todos los métodos de este controlador.  
**Explicación técnica:** Configura el mapeo base de rutas a nivel de clase para el enrutador de Spring MVC.  
**¿Qué cambia en nuestro proyecto?** Todas las rutas de este controlador comenzarán por la ruta especificada (ej. `/api/events`).  
**Comentario mental:** *"La dirección base para este grupo de endpoints."*

### @PostMapping y @GetMapping
**Explicación sencilla:** Indican qué método HTTP activa cada función Java (`POST` para registrar o crear, `GET` para consultar).  
**Explicación técnica:** Atajos de `@RequestMapping(method = RequestMethod.POST)` y `@RequestMapping(method = RequestMethod.GET)`.  
**¿Qué cambia en nuestro proyecto?** Asocia verbos HTTP específicos a métodos individuales de la clase.  
**Comentario mental:** *"GET consulta datos, POST recibe datos para crear."*

### @RequestBody
**Explicación sencilla:** Toma el texto JSON que envió el cliente en el cuerpo de la petición y lo convierte en un objeto Java.  
**Explicación técnica:** Utiliza el convertidor de mensajes de Spring (`HttpMessageConverter` respaldado por Jackson) para deserializar el payload HTTP en una instancia de la clase indicada.  
**¿Qué cambia en nuestro proyecto?** Recibimos directamente un objeto `Event` o `Venue` listo para usar en el parámetro del método.  
**Comentario mental:** *"Transforma el JSON de la petición en mi objeto Java."*

### @ResponseStatus(HttpStatus.CREATED)
**Explicación sencilla:** Hace que el endpoint responda con el código `201 Created` al guardar con éxito.  
**Explicación técnica:** Fija el estado HTTP de respuesta en 201 en lugar del código predeterminado `200 OK`.  
**¿Qué cambia en nuestro proyecto?** Cumplimos con el estándar REST que exige `201 Created` tras la creación exitosa de un recurso.  
**Comentario mental:** *"Cuando este método termine con éxito, responde con 201 Created."*

#### Código: `src/main/java/com/eventify/controller/EventController.java`

```java
package com.eventify.controller;

import com.eventify.model.Event;
import com.eventify.service.EventService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/events")
@Tag(name = "Eventos", description = "Operaciones para registrar y consultar eventos")
public class EventController {

    private final EventService eventService;

    public EventController(EventService eventService) {
        this.eventService = eventService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @Operation(summary = "Registrar un nuevo evento", description = "Valida y almacena un evento en memoria")
    public Event create(@RequestBody Event event) {
        return eventService.save(event);
    }

    @GetMapping
    @ResponseStatus(HttpStatus.OK)
    @Operation(summary = "Listar todos los eventos", description = "Retorna la colección completa de eventos registrados")
    public List<Event> getAll() {
        return eventService.findAll();
    }
}
```

#### Código: `src/main/java/com/eventify/controller/VenueController.java`

```java
package com.eventify.controller;

import com.eventify.model.Venue;
import com.eventify.service.VenueService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/venues")
@Tag(name = "Lugares", description = "Operaciones para registrar y consultar lugares (venues)")
public class VenueController {

    private final VenueService venueService;

    public VenueController(VenueService venueService) {
        this.venueService = venueService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @Operation(summary = "Registrar un nuevo lugar", description = "Valida y almacena un lugar en memoria")
    public Venue create(@RequestBody Venue venue) {
        return venueService.save(venue);
    }

    @GetMapping
    @ResponseStatus(HttpStatus.OK)
    @Operation(summary = "Listar todos los lugares", description = "Retorna la colección completa de lugares registrados")
    public List<Venue> getAll() {
        return venueService.findAll();
    }
}
```

**Qué logramos:** Endpoints REST completos con códigos de estado HTTP precisos (`201` y `200`) e inyección estricta por constructor.

---

### Paso 6 — Cargar datos iniciales con @Configuration y @Bean (Seeder)

#### Concepto necesario: @Configuration y @Bean
La HU solicita cargar datos iniciales (*Seeders*) al arrancar la aplicación utilizando estrictamente `@Configuration` y `@Bean`.

### @Configuration
**Explicación sencilla:** Le dice a Spring: *"Esta clase contiene recetas para crear y configurar componentes al arrancar"*.  
**Explicación técnica:** Marca la clase como fuente de definiciones de beans para el contenedor de Spring. Permite que Spring intercepte métodos para mantener el ciclo de vida de los componentes.  
**¿Qué cambia en nuestro proyecto?** Spring procesa los métodos internos anotados con `@Bean` durante el inicio.  
**Comentario mental:** *"Clase de configuración donde defino componentes del sistema."*

### @Bean
**Explicación sencilla:** Indica que el objeto devuelto por ese método debe ser registrado y gestionado por Spring.  
**Explicación técnica:** Expone un Bean al `ApplicationContext`. Cuando Spring ejecuta el método, inyecta automáticamente los parámetros necesarios que ya existan en el contenedor.  
**¿Qué cambia en nuestro proyecto?** Al arrancar la aplicación, Spring invoca este método para construir el bean y, en ese mismo momento, se ejecutan las llamadas para registrar los datos iniciales.  
**Comentario mental:** *"Ejecuta este método al iniciar y registra el resultado en el contenedor."*

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

> **Decisión de diseño en esta guía:** Para cumplir estrictamente con la HU usando únicamente `@Configuration` y `@Bean` (sin añadir interfaces adicionales como `CommandLineRunner`), creamos un `@Bean` que devuelve un valor booleano. Spring llama al método `seedInitialData`, inyecta ambos servicios y puebla los datos iniciales en memoria de inmediato.
> 
> *Para probar el Escenario 3 (catálogo vacío):* Basta con comentar temporalmente la anotación `@Bean` o los métodos dentro de esta clase para comprobar que la API devuelve listas vacías `[]`.

---

### Paso 7 — Documentar la API con Swagger / OpenAPI

#### Concepto necesario: OpenAPI y Swagger UI
- **OpenAPI:** Es una especificación estandarizada (en formato JSON o YAML) que describe todos los endpoints de una API: sus rutas, parámetros, cuerpos de petición y respuestas.
- **Swagger UI:** Es una herramienta web que lee esa especificación y genera una interfaz gráfica interactiva en el navegador para explorar y probar los endpoints sin necesidad de herramientas externas como Postman.

En el **Paso 5** ya incorporamos las anotaciones de personalización en los controladores:
- `@Tag(name = "...", description = "...")`: Agrupa los endpoints bajo una sección con nombre y descripción amigables.
- `@Operation(summary = "...", description = "...")`: Proporciona un título corto y una explicación clara de lo que hace cada endpoint individual.

Con la dependencia `springdoc-openapi-starter-webmvc-ui` agregada en el `pom.xml`, no necesitamos ninguna clase de configuración adicional. La interfaz queda disponible automáticamente en:
`http://localhost:8080/swagger-ui.html`

---

### Paso 8 — Pruebas Unitarias con JUnit 5 y Mockito

#### Concepto necesario: Pruebas Unitarias Aisladas
Una prueba unitaria debe verificar una única unidad de código (en este caso, la clase `EventService` o `VenueService`) de manera rápida y sin dependencias externas. 
- **No levantamos Spring Context (`@SpringBootTest`):** Levantar todo el contexto de Spring hace las pruebas lentas y dependientes del entorno.
- **Usamos Mockito:** Creamos un "simulador" (*Mock*) del repositorio. Así probamos únicamente la lógica del servicio.

### @ExtendWith(MockitoExtension.class)
**Explicación sencilla:** Activa el motor de Mockito dentro de la prueba de JUnit 5.  
**Explicación técnica:** Extensión de JUnit Jupiter que inicializa los objetos anotados con `@Mock` y los inyecta en los `@InjectMocks`.  
**¿Qué cambia en nuestro proyecto?** Permite usar mocks sin llamar manualmente a `MockitoAnnotations.openMocks(this)`.  
**Comentario mental:** *"Habilita las herramientas de simulación de Mockito en esta clase de prueba."*

### @Mock
**Explicación sencilla:** Crea un objeto falso o simulador de una clase.  
**Explicación técnica:** Genera una implementación proxy que no ejecuta el código real de la clase simulada.  
**¿Qué cambia en nuestro proyecto?** Creamos un `EventRepository` simulado que responde exactamente lo que nosotros le indiquemos durante la prueba.  
**Comentario mental:** *"Crea un doble de prueba para esta dependencia."*

### @InjectMocks
**Explicación sencilla:** Crea la instancia real que queremos probar e introduce en ella los simuladores (`@Mock`).  
**Explicación técnica:** Instancia la clase objetivo utilizando inyección por constructor con los mocks disponibles.  
**¿Qué cambia en nuestro proyecto?** Nos entrega un `EventService` real que utilizará nuestro repositorio falso.  
**Comentario mental:** *"Instancia la clase que quiero probar e inyéctale los simuladores."*

#### Estructura AAA (Arrange, Act, Assert)
Cada prueba se organiza en tres momentos claros:
1. **Arrange (Preparar):** Configuramos los datos y definimos qué responderán los mocks (`when(...).thenReturn(...)`).
2. **Act (Actuar):** Ejecutamos el método que estamos probando.
3. **Assert (Verificar):** Comprobamos que el resultado sea el esperado (`assertEquals`, `assertThrows`) y verificamos que el mock haya sido llamado (`verify(...)`).

#### Código: `src/test/java/com/eventify/service/EventServiceTest.java`

```java
package com.eventify.service;

import com.eventify.exception.InvalidDataException;
import com.eventify.model.Event;
import com.eventify.repository.EventRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.ArrayList;
import java.util.List;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class EventServiceTest {

    @Mock
    private EventRepository eventRepository;

    @InjectMocks
    private EventService eventService;

    private Event validEvent;

    @BeforeEach
    void setUp() {
        validEvent = new Event(null, "Conferencia Java", "2026-10-10", "Charla técnica de backend");
    }

    @Test
    void save_ValidEvent_ReturnsSavedEvent() {
        // Arrange (Preparar)
        Event savedMock = new Event(1L, "Conferencia Java", "2026-10-10", "Charla técnica de backend");
        when(eventRepository.save(validEvent)).thenReturn(savedMock);

        // Act (Actuar)
        Event result = eventService.save(validEvent);

        // Assert (Verificar)
        assertNotNull(result);
        assertEquals(1L, result.getId());
        assertEquals("Conferencia Java", result.getNombre());
        verify(eventRepository, times(1)).save(validEvent);
    }

    @Test
    void save_EmptyName_ThrowsInvalidDataException() {
        // Arrange (Preparar)
        Event invalidEvent = new Event(null, "   ", "2026-10-10", "Descripción");

        // Act & Assert (Actuar y Verificar)
        assertThrows(InvalidDataException.class, () -> eventService.save(invalidEvent));
        verify(eventRepository, never()).save(any());
    }

    @Test
    void save_NullName_ThrowsInvalidDataException() {
        // Arrange (Preparar)
        Event invalidEvent = new Event(null, null, "2026-10-10", "Descripción");

        // Act & Assert (Actuar y Verificar)
        assertThrows(InvalidDataException.class, () -> eventService.save(invalidEvent));
        verify(eventRepository, never()).save(any());
    }

    @Test
    void findAll_ReturnsListOfEvents() {
        // Arrange (Preparar)
        List<Event> mockList = new ArrayList<>();
        mockList.add(new Event(1L, "Evento 1", "2026-10-10", "Desc 1"));
        when(eventRepository.findAll()).thenReturn(mockList);

        // Act (Actuar)
        List<Event> result = eventService.findAll();

        // Assert (Verificar)
        assertNotNull(result);
        assertEquals(1, result.size());
        verify(eventRepository, times(1)).findAll();
    }
}
```

#### Código: `src/test/java/com/eventify/service/VenueServiceTest.java`

```java
package com.eventify.service;

import com.eventify.exception.InvalidDataException;
import com.eventify.model.Venue;
import com.eventify.repository.VenueRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.ArrayList;
import java.util.List;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class VenueServiceTest {

    @Mock
    private VenueRepository venueRepository;

    @InjectMocks
    private VenueService venueService;

    private Venue validVenue;

    @BeforeEach
    void setUp() {
        validVenue = new Venue(null, "Auditorio Principal", "Calle 50 #20-10", 300);
    }

    @Test
    void save_ValidVenue_ReturnsSavedVenue() {
        // Arrange (Preparar)
        Venue savedMock = new Venue(1L, "Auditorio Principal", "Calle 50 #20-10", 300);
        when(venueRepository.save(validVenue)).thenReturn(savedMock);

        // Act (Actuar)
        Venue result = venueService.save(validVenue);

        // Assert (Verificar)
        assertNotNull(result);
        assertEquals(1L, result.getId());
        assertEquals("Auditorio Principal", result.getNombre());
        verify(venueRepository, times(1)).save(validVenue);
    }

    @Test
    void save_EmptyName_ThrowsInvalidDataException() {
        // Arrange (Preparar)
        Venue invalidVenue = new Venue(null, "", "Calle 50 #20-10", 300);

        // Act & Assert (Actuar y Verificar)
        assertThrows(InvalidDataException.class, () -> venueService.save(invalidVenue));
        verify(venueRepository, never()).save(any());
    }

    @Test
    void findAll_ReturnsListOfVenues() {
        // Arrange (Preparar)
        List<Venue> mockList = new ArrayList<>();
        mockList.add(new Venue(1L, "Lugar A", "Dirección A", 100));
        when(venueRepository.findAll()).thenReturn(mockList);

        // Act (Actuar)
        List<Venue> result = venueService.findAll();

        // Assert (Verificar)
        assertNotNull(result);
        assertEquals(1, result.size());
        verify(venueRepository, times(1)).findAll();
    }
}
```

**Qué logramos:** Suites de pruebas independientes, ultrarrápidas y que no dependen de la red ni de levantar Spring para asegurar la lógica de negocio.

---

## 6. ¿Cómo funciona todo junto?

Repasemos el ciclo de vida completo de una solicitud una vez que la aplicación está en marcha:

1. **Arranque:** Spring Boot lee `EventifyApplication`. Detecta las clases con `@Repository`, `@Service`, `@RestController` y `@Configuration`.
2. **Construcción de Beans:** Instancia los repositorios. Luego instancia los servicios inyectándoles los repositorios. Luego instancia los controladores inyectándoles los servicios.
3. **Carga de Datos (Seeder):** Spring ejecuta el `@Bean` en `DataSeederConfig`, llamando a los métodos `save` para precargar dos eventos y dos lugares en memoria.
4. **Llegada de Petición:** Un usuario envía una petición `POST /api/events` con un cuerpo JSON.
5. **Recepción:** `EventController` recibe la petición, Jackson convierte el JSON en una instancia de `Event` y el controlador delega en `eventService.save(event)`.
6. **Lógica de Negocio:** `EventService` valida que el nombre no sea nulo ni esté en blanco.
   - Si no es válido, lanza `InvalidDataException`. Spring intercepta la excepción y retorna `400 Bad Request` sin tocar el repositorio.
   - Si es válido, llama a `eventRepository.save(event)`.
7. **Persistencia en Memoria:** `EventRepository` asigna el siguiente identificador (`idCounter++`), agrega el objeto a la lista `events` y retorna la entidad con ID asignado.
8. **Respuesta:** El objeto regresa por el servicio hasta el controlador, que lo serializa a JSON y responde al cliente con el código `201 Created`.

---

## 7. Cómo probar la Historia de Usuario

Inicia la aplicación ejecutando la clase principal `EventifyApplication` en tu IDE o mediante la terminal con:

```bash
mvn spring-boot:run
```

Abre tu navegador en:
`http://localhost:8080/swagger-ui/index.html` (o `http://localhost:8080/swagger-ui.html`)

---

### Prueba del Escenario 1: Registro Exitoso (Camino Feliz)

- **Objetivo:** Registrar un evento con datos válidos y recibir `201 Created`.
- **Método y URL:** `POST /api/events`
- **Body (JSON):**
  ```json
  {
    "nombre": "Cumbre Internacional de IA",
    "fecha": "2026-11-30",
    "descripcion": "Encuentro de ponentes de inteligencia artificial"
  }
  ```
- **Resultado esperado:** Código HTTP `201 Created` y cuerpo de respuesta con el ID asignado:
  ```json
  {
    "id": 3,
    "nombre": "Cumbre Internacional de IA",
    "fecha": "2026-11-30",
    "descripcion": "Encuentro de ponentes de inteligencia artificial"
  }
  ```

---

### Prueba del Escenario 2: Intento de Registro Inválido (Camino de Error)

- **Objetivo:** Validar que un evento con nombre vacío sea rechazado con un error controlado.
- **Método y URL:** `POST /api/events`
- **Body (JSON):**
  ```json
  {
    "nombre": "",
    "fecha": "2026-11-30",
    "descripcion": "Evento sin nombre"
  }
  ```
- **Resultado esperado:** Código HTTP `400 Bad Request`. El dato no se almacena en el repositorio.

---

### Prueba del Escenario 3: Consulta de Catálogo Vacío (Caso de Borde)

- **Objetivo:** Asegurar que si no hay datos cargados, la API devuelva una lista vacía `[]` con código `200 OK`.
- **Pasos:** 
  1. Comenta temporalmente la anotación `@Bean` en `DataSeederConfig.java`.
  2. Reinicia la aplicación.
  3. Ejecuta una petición `GET /api/events`.
- **Resultado esperado:** Código HTTP `200 OK` con un cuerpo de respuesta:
  ```json
  []
  ```

---

### Prueba del Escenario 4: Verificación de Documentación en Swagger

- **Objetivo:** Validar que los métodos estén correctamente expuestos y documentados.
- **Acceso:** Entra en `http://localhost:8080/swagger-ui/index.html`.
- **Resultado esperado:**
  - Grupo **Eventos**: Endpoints `GET /api/events` y `POST /api/events` con sus resúmenes ("Listar todos los eventos", "Registrar un nuevo evento").
  - Grupo **Lugares**: Endpoints `GET /api/venues` y `POST /api/venues` con sus resúmenes correspondientes.
  - Posibilidad de probar peticiones interactivamente usando el botón **"Try it out"**.

---

### Ejecución de Pruebas Unitarias

Para verificar que todos los tests unitarios pasen sin levantar el servidor:

```bash
mvn test
```

Resultado esperado en consola:
```text
[INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: ... s -- in com.eventify.service.EventServiceTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: ... s -- in com.eventify.service.VenueServiceTest
[INFO] BUILD SUCCESS
```

---

## 8. Errores comunes

### 1. `Cannot invoke "...getName()" because "event" is null` (NullPointerException)
- **Por qué ocurre:** Se envió una petición `POST` sin cuerpo o con un JSON mal formateado, o bien el objeto llegó nulo al método del servicio.
- **Qué revisar:** En el servicio, valida primero si el objeto o el atributo son nulos antes de llamar a `.trim()`:
  ```java
  if (event.getNombre() == null || event.getNombre().trim().isEmpty())
  ```

### 2. Error 404 al intentar acceder a Swagger UI
- **Por qué ocurre:** Versión incorrecta de la librería `springdoc` para Spring Boot 3 o URL incorrecta.
- **Qué revisar:** Para Spring Boot 3.x, la dependencia debe ser `springdoc-openapi-starter-webmvc-ui` (no `springfox` ni versiones 1.x). Asegúrate de navegar a `http://localhost:8080/swagger-ui/index.html`.

### 3. Las pruebas unitarias fallan con `NullPointerException` en el servicio
- **Por qué ocurre:** Se olvidó anotar la clase de prueba con `@ExtendWith(MockitoExtension.class)` o faltó la anotación `@InjectMocks` sobre la variable del servicio.
- **Qué revisar:** Verifica que los mocks estén declarados con `@Mock` y el servicio bajo prueba con `@InjectMocks`.

### 4. La aplicación no inicia por error de inyección de dependencias
- **Por qué ocurre:** Faltó anotar alguna clase con `@Service` o `@Repository`, por lo que Spring no la reconoce como Bean y no sabe cómo inyectarla en el constructor que la requiere.
- **Qué revisar:** Confirma que `EventRepository` tenga `@Repository` y `EventService` tenga `@Service`.

---

## 9. Checklist final

Verifica que cada uno de los siguientes puntos esté completado antes de dar por terminada la Historia de Usuario:

- [ ] Las entidades `Event` y `Venue` están creadas con los atributos exactos solicitados.
- [ ] Los repositorios `EventRepository` y `VenueRepository` están anotados con `@Repository` y gestionan datos en listas en memoria.
- [ ] Los servicios `EventService` y `VenueService` están anotados con `@Service` e implementan inyección estricta por constructor.
- [ ] Los servicios validan que el nombre no esté vacío ni en blanco antes de llamar al repositorio.
- [ ] La excepción personalizada `InvalidDataException` está anotada con `@ResponseStatus(HttpStatus.BAD_REQUEST)`.
- [ ] Los controladores `EventController` y `VenueController` están anotados con `@RestController` y responden con `201 Created` en el POST y `200 OK` en el GET.
- [ ] La clase `DataSeederConfig` usa `@Configuration` y `@Bean` para precargar registros de prueba al arrancar.
- [ ] Swagger UI está accesible en `/swagger-ui/index.html` con nombres y descripciones personalizadas.
- [ ] Las pruebas unitarias con JUnit 5 y Mockito para ambos servicios se ejecutan de forma aislada y pasan exitosamente (`mvn test`).
- [ ] Se verificaron los 4 escenarios de aceptación (registro exitoso, registro inválido, catálogo vacío y Swagger).

---

## 10. ¿Qué aprendimos?

Durante esta Historia de Usuario comprendimos los pilares de la arquitectura backend con Spring Boot:

1. **Separación de responsabilidades (SoC):** Cada clase tiene un único rol definido. El controlador solo atiende peticiones web, el servicio contiene las reglas del negocio y el repositorio encapsula el almacenamiento de datos.
2. **Estereotipos de Spring (`@Component`, `@Repository`, `@Service`, `@RestController`):** Son etiquetas que le indican a Spring qué rol cumple cada clase para que las administre en su contenedor de dependencias.
3. **Inyección de Dependencias por Constructor:** Es la forma más limpia y recomendada de conectar componentes. Hace que las clases sean inmutables (`final`), explícitas respecto a lo que necesitan y sumamente sencillas de probar sin depender de Spring.
4. **Testing Aislado con Mocks:** Aprendimos a testear la lógica de negocio en milisegundos simulando el comportamiento de las dependencias externas con Mockito, sin necesidad de arrancar el servidor web ni el contenedor de Spring.
5. **Documentación viva:** Con Swagger/OpenAPI la documentación se genera a partir del propio código y se mantiene sincronizada con cada cambio que hagamos en los controladores.

---

## 11. Mini repaso

Intenta responder las siguientes preguntas para comprobar tu aprendizaje:

1. ¿Por qué es preferible usar inyección de dependencias por constructor en lugar de colocar `@Autowired` directamente sobre los atributos?
2. ¿Qué diferencia práctica existe entre `@Repository` y `@Service` si ambas son administradas por Spring como Beans?
3. En una prueba unitaria con Mockito, ¿cuál es la diferencia entre el objeto marcado con `@Mock` y el marcado con `@InjectMocks`?
4. ¿Por qué es una mala práctica dejar que la capa `@Repository` valide si un nombre viene vacío?
5. ¿Qué ventaja ofrece anotar una excepción con `@ResponseStatus(HttpStatus.BAD_REQUEST)` en lugar de capturar la excepción con un `try-catch` dentro del controlador?
