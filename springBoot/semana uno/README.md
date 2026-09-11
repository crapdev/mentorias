# Eventify

Guía de estudio e implementación paso a paso del cimiento arquitectónico de **Eventify**, aplicando el patrón Spring MVC, inyección de dependencias por constructor, estereotipos, almacenamiento en memoria, documentación OpenAPI/Swagger y pruebas unitarias con JUnit 5 y Mockito.

---

## 📋 Tabla de contenidos

- [1. ¿Qué vamos a construir?](#1-qué-vamos-a-construir)
- [2. ¿Qué pide exactamente la Historia de Usuario?](#2-qué-pide-exactamente-la-historia-de-usuario)
- [3. Conceptos que debemos entender antes de comenzar](#3-conceptos-que-debemos-entender-antes-de-comenzar)
- [4. ¿Qué vamos a modificar o crear?](#4-qué-vamos-a-modificar-o-crear)
- [5. Flujo de funcionamiento](#5-flujo-de-funcionamiento)
- [6. Implementación paso a paso](#6-implementación-paso-a-paso)
- [7. Explicación del código](#7-explicación-del-código)
- [8. Ejemplos](#8-ejemplos)
- [9. ¿Cómo funciona todo junto?](#9-cómo-funciona-todo-junto)
- [10. Cómo probar la Historia de Usuario](#10-cómo-probar-la-historia-de-usuario)
- [11. Errores comunes](#11-errores-comunes)
- [12. Checklist final](#12-checklist-final)
- [13. ¿Qué aprendimos?](#13-qué-aprendimos)
- [14. Mini repaso](#14-mini-repaso)

---

## 1. ¿Qué vamos a construir?

Construiremos el backend inicial de **Eventify** organizado bajo el patrón arquitectónico Spring MVC. La aplicación expondrá endpoints REST para registrar y consultar eventos y lugares almacenados temporalmente en memoria, documentará automáticamente sus rutas con Swagger UI y validará las reglas de negocio de los servicios mediante pruebas unitarias aisladas con Mockito.

---

## 2. ¿Qué pide exactamente la Historia de Usuario?

### Requerido por la HU

- **Configuración inicial:** Proyecto creado con `spring-boot-starter-web` y `lombok`.
- **Inyección de dependencias:** Obligatoriamente por constructor en todos los componentes.
- **Capa de persistencia (`@Repository`):** Clases que almacenen colecciones (`List`) simulando una base de datos en memoria para `Event` y `Venue`.
- **Capa de negocio (`@Service`):** Clases con reglas de validación (por ejemplo, rechazar nombres vacíos) antes de invocar a los repositorios.
- **Capa de presentación (`@RestController`):** Endpoints para registrar y listar entidades, respondiendo con código HTTP `201 Created` en registros y `200 OK` en consultas.
- **Modelos POJO:**
  - `Event`: `id`, `nombre`, `fecha`, `descripcion`.
  - `Venue`: `id`, `nombre`, `direccion`, `capacidad`.
- **Carga inicial de datos:** Clase `@Configuration` con métodos `@Bean` para sembrar datos de prueba en el arranque.
- **Documentación con Swagger:** Dependencia `springdoc-openapi-starter-webmvc-ui` con nombres de endpoints y tags personalizados.
- **Pruebas unitarias:** Tests con JUnit 5 y Mockito para la capa `@Service` de forma aislada, sin levantar el contexto de Spring.

### Recomendado

- Configuración condicional en el Seeder mediante `application.properties` para alternar entre catálogo con datos y catálogo vacío sin alterar código Java.

---

## 3. Conceptos que debemos entender antes de comenzar

- **Patrón MVC (Modelo - Vista - Controlador):**
  - *Qué es:* Es un patrón de arquitectura de software que separa la aplicación en tres áreas con responsabilidades distintas: datos (Modelo), interfaz o serialización (Vista) y flujo de peticiones (Controlador).
  - *Para qué sirve:* Evita mezclar código de red con lógica de negocio o acceso a datos.
  - *Por qué aparece aquí:* Permite que el Controller solo reciba peticiones HTTP, el Service procese las reglas y el Repository guarde los datos.

- **Inyección de Dependencias por Constructor:**
  - *Qué es:* Pasar las clases colaboradoras a través del constructor público de una clase en lugar de instanciarlas manualmente con `new` o inyectarlas directamente en campos privados con `@Autowired`.
  - *Para qué sirve:* Hace explícitas las dependencias de una clase, asegura que los atributos puedan ser inmutables (`final`) y facilita pasar objetos simulados (*mocks*) en pruebas unitarias.
  - *Ejemplo:*
    ```java
    public class EventService {
        private final EventRepository eventRepository;

        public EventService(EventRepository eventRepository) {
            this.eventRepository = eventRepository;
        }
    }
    ```

- **Estereotipos de Spring:**
  - *Qué es:* Anotaciones que asignan un rol específico a una clase para que Spring gestione su ciclo de vida como un Bean dentro del contenedor (*ApplicationContext*).
  - *Por qué aparecen aquí:* Utilizaremos `@Repository` para datos, `@Service` para reglas de negocio y `@RestController` para la API web.

- **Mocks y Pruebas Unitarias Aisladas:**
  - *Qué es:* Una prueba donde se evalúa una sola clase en milisegundos sustituyendo sus dependencias reales por dobles de prueba programables (*mocks*).
  - *Por qué aparecen aquí:* La TASK 3 exige validar los servicios sin levantar todo el servidor ni el contexto pesado de Spring.

---

## 4. ¿Qué vamos a modificar o crear?

### Estructura del proyecto

```text
src/main/java/com/eventify/
│
├── EventifyApplication.java           # Clase principal que inicia Spring Boot
│
├── config/
│   └── DataSeeder.java                # Carga inicial con @Configuration y @Bean
│
├── controller/
│   ├── EventController.java           # Controlador REST para Eventos
│   └── VenueController.java           # Controlador REST para Lugares
│
├── exception/
│   └── ValidationException.java       # Excepción de reglas de negocio
│
├── model/
│   ├── Event.java                     # Entidad POJO de Evento
│   └── Venue.java                     # Entidad POJO de Lugar
│
├── repository/
│   ├── EventRepository.java           # Persistencia en memoria de Eventos
│   └── VenueRepository.java           # Persistencia en memoria de Lugares
│
└── service/
    ├── EventService.java              # Lógica y validaciones de Eventos
    └── VenueService.java              # Lógica y validaciones de Lugares

src/test/java/com/eventify/
└── service/
    ├── EventServiceTest.java          # Pruebas unitarias de EventService
    └── VenueServiceTest.java          # Pruebas unitarias de VenueService
```

### Responsabilidad de cada capa

- **`model`:** Define las estructuras de datos puras mediante atributos, getters, setters y constructores.
- **`repository`:** Administra la lista en memoria y la asignación de IDs autoincrementales.
- **`service`:** Verifica que los datos sean coherentes (no nulos ni vacíos) antes de transferirlos al repositorio.
- **`controller`:** Mapea las URLs HTTP, recibe el cuerpo de la petición (JSON) y devuelve la respuesta con su respectivo código de estado HTTP.
- **`config`:** Contiene clases que declaran Beans gestionados en el inicio de la aplicación.
- **`exception`:** Aloja los errores personalizados para interrumpir flujos no válidos.

---

## 5. Flujo de funcionamiento

```text
[Cliente HTTP (Swagger / Postman)]
        │  1. Petición POST /api/events con JSON
        ▼
[EventController]
        │  2. Mapea el JSON a un objeto Event
        │  3. Llama a eventService.create(event)
        ▼
[EventService]
        │  4. Ejecuta validaciones:
        │     - ¿Nombre nulo o vacío? -> Lanza ValidationException
        │     - ¿Fecha nula? -> Lanza ValidationException
        │  5. Si todo es correcto, llama a eventRepository.save(event)
        ▼
[EventRepository]
        │  6. Asigna ID secuencial y almacena en List<Event>
        │  7. Retorna el objeto guardado
        ▼
[EventController]
        │  8. Recibe el resultado y devuelve HTTP 201 Created con el objeto
        ▼
[Cliente HTTP] Recibe confirmación
```

---

## 6. Implementación paso a paso

### Paso 1. Configurar las dependencias (`pom.xml`)

#### ¿Qué vamos a hacer?

Modificar el archivo de dependencias del proyecto para incluir Spring Web, Lombok, documentación OpenAPI con Swagger y las librerías de prueba unitaria.

#### Concepto necesario: Spring Initializr y Starters

Los *Starters* son paquetes de dependencias coordinados por Spring Boot para que no existan discrepancias de versiones entre librerías compatibles.

#### Archivo: `pom.xml` (Archivo existente a modificar)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version>
        <relativePath/>
    </parent>
    
    <groupId>com.eventify</groupId>
    <artifactId>eventify</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eventify</name>
    <description>Cimiento Arquitectonico de Eventify</description>
    
    <properties>
        <java.version>17</java.version>
        <springdoc.version>2.5.0</springdoc.version>
    </properties>
    
    <dependencies>
        <!-- Servidor web embebido y patron MVC -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Lombok para reducir codigo repetitivo (getters, setters, constructores) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Documentacion OpenAPI y Swagger UI -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>${springdoc.version}</version>
        </dependency>

        <!-- JUnit 5 y Mockito para pruebas -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

---

### Paso 2. Crear las entidades POJO (`Event` y `Venue`)

#### ¿Qué vamos a hacer?

Definir las clases que modelan la información requerida por la HU utilizando Lombok solo para constructores, getters y setters estándar.

#### Concepto necesario: POJO (Plain Old Java Object)

Clase simple que únicamente contiene atributos privados, constructores y métodos de acceso sin depender de interfaces complejas del framework.

#### Archivo nuevo: `src/main/java/com/eventify/model/Event.java`

```java
package com.eventify.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDate;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Event {
    private Long id;
    private String nombre;
    private LocalDate fecha;
    private String descripcion;
}
```

#### Archivo nuevo: `src/main/java/com/eventify/model/Venue.java`

```java
package com.eventify.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Venue {
    private Long id;
    private String nombre;
    private String direccion;
    private Integer capacidad;
}
```

*Explicación:* `@Data` genera en tiempo de compilación los métodos `get...()`, `set...()`, `toString()`, `equals()` y `hashCode()`. `@NoArgsConstructor` añade el constructor vacío requerido para deserializar JSON y `@AllArgsConstructor` crea el constructor con todos los atributos.

---

### Paso 3. Crear la excepción de validación

#### ¿Qué vamos a hacer?

Crear una excepción personalizada para que el servicio detenga la operación si los datos del evento o lugar son incorrectos.

#### Concepto necesario: `RuntimeException`

Al extender de `RuntimeException`, creamos una excepción no comprobada (*unchecked*), lo que evita declarar cláusulas `throws` forzosas en los métodos.

#### Archivo nuevo: `src/main/java/com/eventify/exception/ValidationException.java`

```java
package com.eventify.exception;

public class ValidationException extends RuntimeException {
    public ValidationException(String message) {
        super(message);
    }
}
```

---

### Paso 4. Implementar los Repositorios en memoria (`@Repository`)

#### ¿Qué vamos a hacer?

Crear las clases encargadas de manipular la colección de datos en memoria y administrar el incremento secuencial del identificador `id`.

#### Concepto necesario: Persistencia en Colecciones

Utilizamos un `ArrayList` estándar alojado en una clase anotada con `@Repository`. Como el componente es un Bean singleton, la lista se mantiene viva en la memoria del programa mientras la aplicación se encuentre en ejecución.

#### Archivo nuevo: `src/main/java/com/eventify/repository/EventRepository.java`

```java
package com.eventify.repository;

import com.eventify.model.Event;
import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.List;

@Repository
public class EventRepository {

    private final List<Event> database = new ArrayList<>();
    private Long idCounter = 1L;

    public Event save(Event event) {
        if (event.getId() == null) {
            event.setId(idCounter);
            idCounter++;
        }
        database.add(event);
        return event;
    }

    public List<Event> findAll() {
        return new ArrayList<>(database);
    }
}
```

#### Archivo nuevo: `src/main/java/com/eventify/repository/VenueRepository.java`

```java
package com.eventify.repository;

import com.eventify.model.Venue;
import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.List;

@Repository
public class VenueRepository {

    private final List<Venue> database = new ArrayList<>();
    private Long idCounter = 1L;

    public Venue save(Venue venue) {
        if (venue.getId() == null) {
            venue.setId(idCounter);
            idCounter++;
        }
        database.add(venue);
        return venue;
    }

    public List<Venue> findAll() {
        return new ArrayList<>(database);
    }
}
```

*Explicación:* El método `findAll()` devuelve `new ArrayList<>(database)`. Retornar una copia defensiva previene que modificaciones sobre la lista obtenida alteren directamente los elementos internos del repositorio.

---

### Paso 5. Implementar la Capa de Negocio (`@Service`)

#### ¿Qué vamos a hacer?

Crear `EventService` y `VenueService` aplicando inyección por constructor para conectar con sus respectivos repositorios y aplicar las validaciones de negocio requeridas.

#### Concepto necesario: Desacoplamiento de Lógica

El servicio valida que los campos requeridos existan y tengan valores coherentes. Si la validación falla, se lanza `ValidationException` inmediatamente, garantizando que el dato no llegue al repositorio (cumpliendo el Escenario 2).

#### Archivo nuevo: `src/main/java/com/eventify/service/EventService.java`

```java
package com.eventify.service;

import com.eventify.exception.ValidationException;
import com.eventify.model.Event;
import com.eventify.repository.EventRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class EventService {

    private final EventRepository eventRepository;

    // Inyección de dependencias estricta por constructor
    public EventService(EventRepository eventRepository) {
        this.eventRepository = eventRepository;
    }

    public Event create(Event event) {
        if (event == null) {
            throw new ValidationException("El evento no puede ser nulo.");
        }
        if (event.getNombre() == null || event.getNombre().trim().isEmpty()) {
            throw new ValidationException("El nombre del evento es obligatorio.");
        }
        if (event.getFecha() == null) {
            throw new ValidationException("La fecha del evento es obligatoria.");
        }

        return eventRepository.save(event);
    }

    public List<Event> findAll() {
        return eventRepository.findAll();
    }
}
```

#### Archivo nuevo: `src/main/java/com/eventify/service/VenueService.java`

```java
package com.eventify.service;

import com.eventify.exception.ValidationException;
import com.eventify.model.Venue;
import com.eventify.repository.VenueRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class VenueService {

    private final VenueRepository venueRepository;

    // Inyección de dependencias estricta por constructor
    public VenueService(VenueRepository venueRepository) {
        this.venueRepository = venueRepository;
    }

    public Venue create(Venue venue) {
        if (venue == null) {
            throw new ValidationException("El lugar no puede ser nulo.");
        }
        if (venue.getNombre() == null || venue.getNombre().trim().isEmpty()) {
            throw new ValidationException("El nombre del lugar es obligatorio.");
        }
        if (venue.getCapacidad() == null || venue.getCapacidad() <= 0) {
            throw new ValidationException("La capacidad del lugar debe ser mayor a cero.");
        }

        return venueRepository.save(venue);
    }

    public List<Venue> findAll() {
        return venueRepository.findAll();
    }
}
```

---

### Paso 6. Implementar los Controladores REST (`@RestController`)

#### ¿Qué vamos a hacer?

Crear las clases que recibirán las peticiones web en `/api/events` y `/api/venues`, documentando sus operaciones mediante Swagger OpenAPI y retornando los códigos HTTP `201 Created` y `200 OK`.

#### Concepto necesario: Códigos de Estado HTTP y Documentación Declarativa

- `ResponseEntity.status(HttpStatus.CREATED).body(...)` emite el código HTTP `201`.
- `ResponseEntity.ok(...)` emite el código HTTP `200`.
- Con `springdoc`, las anotaciones `@Tag`, `@Operation` y `@ApiResponse` permiten personalizar la interfaz gráfica sin necesidad de clases de configuración complejas.

#### Archivo nuevo: `src/main/java/com/eventify/controller/EventController.java`

```java
package com.eventify.controller;

import com.eventify.model.Event;
import com.eventify.service.EventService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/api/events")
@Tag(name = "Eventos", description = "Endpoints para la administracion de eventos")
public class EventController {

    private final EventService eventService;

    // Inyección de dependencias por constructor
    public EventController(EventService eventService) {
        this.eventService = eventService;
    }

    @PostMapping
    @Operation(summary = "Registrar un nuevo evento", description = "Valida los datos y registra un evento en el catalogo.")
    @ApiResponse(responseCode = "201", description = "Evento creado exitosamente")
    public ResponseEntity<Event> create(@RequestBody Event event) {
        Event createdEvent = eventService.create(event);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdEvent);
    }

    @GetMapping
    @Operation(summary = "Listar eventos", description = "Retorna todos los eventos registrados.")
    @ApiResponse(responseCode = "200", description = "Consulta realizada exitosamente")
    public ResponseEntity<List<Event>> findAll() {
        List<Event> events = eventService.findAll();
        return ResponseEntity.ok(events);
    }
}
```

#### Archivo nuevo: `src/main/java/com/eventify/controller/VenueController.java`

```java
package com.eventify.controller;

import com.eventify.model.Venue;
import com.eventify.service.VenueService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/api/venues")
@Tag(name = "Lugares (Venues)", description = "Endpoints para la administracion de lugares")
public class VenueController {

    private final VenueService venueService;

    // Inyección de dependencias por constructor
    public VenueController(VenueService venueService) {
        this.venueService = venueService;
    }

    @PostMapping
    @Operation(summary = "Registrar un nuevo lugar", description = "Valida y almacena un nuevo lugar para eventos.")
    @ApiResponse(responseCode = "201", description = "Lugar registrado exitosamente")
    public ResponseEntity<Venue> create(@RequestBody Venue venue) {
        Venue createdVenue = venueService.create(venue);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdVenue);
    }

    @GetMapping
    @Operation(summary = "Listar lugares", description = "Retorna todos los recintos disponibles.")
    @ApiResponse(responseCode = "200", description = "Consulta realizada exitosamente")
    public ResponseEntity<List<Venue>> findAll() {
        List<Venue> venues = venueService.findAll();
        return ResponseEntity.ok(venues);
    }
}
```

---

### Paso 7. Configuración del Seeder inicial (`DataSeeder`)

#### ¿Qué vamos a hacer?

Crear una clase `@Configuration` que utilice métodos anotados con `@Bean` para cargar los datos iniciales al arrancar la aplicación, permitiendo desactivarla por configuración.

#### Concepto necesario: Métodos `@Bean` como Seeders

Al anotar un método con `@Bean` dentro de una clase `@Configuration`, Spring invoca el método durante la etapa de inicialización del contexto. El objeto devuelto se registra como un Bean gestionado, ejecutando de forma natural el guardado en el repositorio.

#### Archivo nuevo: `src/main/java/com/eventify/config/DataSeeder.java`

```java
package com.eventify.config;

import com.eventify.model.Event;
import com.eventify.model.Venue;
import com.eventify.repository.EventRepository;
import com.eventify.repository.VenueRepository;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.LocalDate;

@Configuration
@ConditionalOnProperty(name = "eventify.seeder.enabled", havingValue = "true", matchIfMissing = true)
public class DataSeeder {

    @Bean
    public Event initialEvent(EventRepository eventRepository) {
        Event event = new Event(
                1L,
                "Evento de prueba",
                LocalDate.of(2026, 10, 20),
                "Evento inicial de Eventify"
        );
        return eventRepository.save(event);
    }

    @Bean
    public Venue initialVenue(VenueRepository venueRepository) {
        Venue venue = new Venue(
                1L,
                "Centro de Convenciones",
                "Calle 50 # 10-20",
                500
        );
        return venueRepository.save(venue);
    }
}
```

#### Archivo de configuración: `src/main/resources/application.properties`

```properties
spring.application.name=eventify
server.port=8080

# Control del Seeder: true para cargar datos, false para arrancar con catalogo vacio
eventify.seeder.enabled=true
```

---

### Paso 8. Pruebas Unitarias Aisladas con JUnit 5 y Mockito

#### ¿Qué vamos a hacer?

Escribir las pruebas unitarias para `EventService` y `VenueService` evaluando:

1. El registro exitoso (camino feliz).
2. El rechazo ante nombres vacíos o nulos sin llegar al repositorio (camino de error).
3. La consulta de catálogos sin datos (caso de borde).

#### Concepto necesario: Mocks con Mockito

- `@Mock`: Crea una implementación falsa del repositorio.
- `@InjectMocks`: Crea la instancia real del servicio e inyecta los mocks en su constructor.
- `@BeforeEach`: Inicializa datos antes de cada método de test.
- `when(...)` y `thenReturn(...)`: Fijan el comportamiento del mock.
- `assertEquals(...)`: Compara el valor esperado con el obtenido.
- `verify(...)`: Verifica que los métodos del mock hayan sido invocados (o no, con `never()`).

#### Archivo nuevo: `src/test/java/com/eventify/service/EventServiceTest.java`

```java
package com.eventify.service;

import com.eventify.exception.ValidationException;
import com.eventify.model.Event;
import com.eventify.repository.EventRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.never;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class EventServiceTest {

    @Mock
    private EventRepository eventRepository;

    @InjectMocks
    private EventService eventService;

    private Event validEvent;

    @BeforeEach
    void setUp() {
        validEvent = new Event(null, "Conferencia Tech", LocalDate.of(2026, 11, 10), "Encuentro de programadores");
    }

    @Test
    @DisplayName("Escenario 1: Registro Exitoso - Debe almacenar y retornar el evento")
    void create_WhenValidEvent_ShouldSaveAndReturn() {
        Event savedMock = new Event(1L, "Conferencia Tech", LocalDate.of(2026, 11, 10), "Encuentro de programadores");

        when(eventRepository.save(any(Event.class))).thenReturn(savedMock);

        Event result = eventService.create(validEvent);

        assertNotNull(result);
        assertEquals(1L, result.getId());
        assertEquals("Conferencia Tech", result.getNombre());
        verify(eventRepository, times(1)).save(validEvent);
    }

    @Test
    @DisplayName("Escenario 2: Intento con Nombre Vacio - Debe lanzar excepcion y no llamar al repositorio")
    void create_WhenNameIsEmpty_ShouldThrowExceptionAndNeverCallRepository() {
        Event invalidEvent = new Event(null, "   ", LocalDate.of(2026, 11, 10), "Sin nombre");

        ValidationException exception = assertThrows(ValidationException.class, () -> {
            eventService.create(invalidEvent);
        });

        assertEquals("El nombre del evento es obligatorio.", exception.getMessage());
        verify(eventRepository, never()).save(any(Event.class));
    }

    @Test
    @DisplayName("Escenario 2b: Intento con Fecha Nula - Debe lanzar excepcion y no llamar al repositorio")
    void create_WhenDateIsNull_ShouldThrowExceptionAndNeverCallRepository() {
        Event invalidEvent = new Event(null, "Evento sin fecha", null, "Descripcion valida");

        assertThrows(ValidationException.class, () -> eventService.create(invalidEvent));
        verify(eventRepository, never()).save(any(Event.class));
    }

    @Test
    @DisplayName("Escenario 3: Consulta de Catalogo Vacio - Debe devolver lista vacia []")
    void findAll_WhenNoData_ShouldReturnEmptyList() {
        when(eventRepository.findAll()).thenReturn(new ArrayList<>());

        List<Event> result = eventService.findAll();

        assertNotNull(result);
        assertTrue(result.isEmpty());
        verify(eventRepository, times(1)).findAll();
    }
}
```

#### Archivo nuevo: `src/test/java/com/eventify/service/VenueServiceTest.java`

```java
package com.eventify.service;

import com.eventify.exception.ValidationException;
import com.eventify.model.Venue;
import com.eventify.repository.VenueRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.ArrayList;
import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.never;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class VenueServiceTest {

    @Mock
    private VenueRepository venueRepository;

    @InjectMocks
    private VenueService venueService;

    private Venue validVenue;

    @BeforeEach
    void setUp() {
        validVenue = new Venue(null, "Auditorio Principal", "Carrera 7 # 32-10", 350);
    }

    @Test
    @DisplayName("Registro Exitoso de Venue")
    void create_WhenValidVenue_ShouldSaveAndReturn() {
        Venue savedMock = new Venue(1L, "Auditorio Principal", "Carrera 7 # 32-10", 350);

        when(venueRepository.save(any(Venue.class))).thenReturn(savedMock);

        Venue result = venueService.create(validVenue);

        assertNotNull(result);
        assertEquals(1L, result.getId());
        assertEquals("Auditorio Principal", result.getNombre());
        verify(venueRepository, times(1)).save(validVenue);
    }

    @Test
    @DisplayName("Intento con Capacidad Menor o Igual a Cero - Lanza excepcion")
    void create_WhenCapacityInvalid_ShouldThrowException() {
        Venue invalidVenue = new Venue(null, "Salon Invalido", "Calle 10", 0);

        ValidationException exception = assertThrows(ValidationException.class, () -> {
            venueService.create(invalidVenue);
        });

        assertEquals("La capacidad del lugar debe ser mayor a cero.", exception.getMessage());
        verify(venueRepository, never()).save(any(Venue.class));
    }

    @Test
    @DisplayName("Consulta de Catalogo Vacio de Venues")
    void findAll_WhenNoData_ShouldReturnEmptyList() {
        when(venueRepository.findAll()).thenReturn(new ArrayList<>());

        List<Venue> result = venueService.findAll();

        assertNotNull(result);
        assertTrue(result.isEmpty());
        verify(venueRepository, times(1)).findAll();
    }
}
```

---

## 7. Explicación del código

### Anotaciones de Spring MVC y Estereotipos

#### @RestController

- **Explicación sencilla:** Le avisa a Spring que esta clase atenderá solicitudes que llegan por internet y que lo que devuelva debe entregarse directamente en formato JSON.
- **Explicación técnica:** Anotación compuesta que combina `@Controller` y `@ResponseBody`. Convierte automáticamente las respuestas de los métodos en JSON mediante la librería Jackson.
- **¿Qué cambia en nuestro proyecto?** Hace que los métodos de `EventController` y `VenueController` queden publicados como endpoints web.
- **Comentario mental:** *"Esta clase recibe peticiones de nuestra API."*

#### @Service

- **Explicación sencilla:** Le indica a Spring que esta clase es el cerebro de la aplicación y contiene las reglas del negocio.
- **Explicación técnica:** Especialización de `@Component` con significado semántico de lógica de negocio. Registra la clase como un Bean singleton administrado.
- **¿Qué cambia en nuestro proyecto?** Contiene las validaciones que evitan que datos corruptos lleguen a los repositorios.
- **Comentario mental:** *"Aquí reside la lógica de negocio y las validaciones."*

#### @Repository

- **Explicación sencilla:** Marca la clase encargada exclusivamente de guardar, consultar o modificar los datos.
- **Explicación técnica:** Especialización de `@Component` orientada a la persistencia. En aplicaciones con bases de datos reales, traduce excepciones de SQL a la jerarquía de excepciones de Spring.
- **¿Qué cambia en nuestro proyecto?** Declara los componentes que gestionan las listas en memoria.
- **Comentario mental:** *"Esta clase interactúa directamente con los datos almacenados."*

#### @PostMapping

- **Explicación sencilla:** Enlaza un método de Java con solicitudes que utilicen el verbo HTTP POST.
- **Explicación técnica:** Variante simplificada de `@RequestMapping(method = RequestMethod.POST)`.
- **¿Qué cambia en nuestro proyecto?** Permite que una llamada a `POST /api/events` active el método `create()`.
- **Comentario mental:** *"Se ejecuta cuando envían información nueva por POST."*

#### @GetMapping

- **Explicación sencilla:** Enlaza un método de Java con solicitudes de consulta usando el verbo HTTP GET.
- **Explicación técnica:** Variante simplificada de `@RequestMapping(method = RequestMethod.GET)`.
- **¿Qué cambia en nuestro proyecto?** Permite que una llamada a `GET /api/events` invoque el método `findAll()`.
- **Comentario mental:** *"Se ejecuta cuando piden consultar datos por GET."*

#### @Configuration y @Bean

- **Explicación sencilla:** `@Configuration` marca una clase como taller de configuración. `@Bean` colocado sobre un método le dice a Spring: *"ejecuta este método y quédate con el objeto que devuelva"*.
- **Explicación técnica:** `@Configuration` define una clase de configuración gestionada por proxies de Spring. Los métodos anotados con `@Bean` instancian y configuran componentes en el `ApplicationContext`.
- **¿Qué cambia en nuestro proyecto?** Permite a la clase `DataSeeder` guardar el evento y el lugar iniciales apenas arranca la aplicación.
- **Comentario mental:** *"Fabrico Beans manualmente para que Spring los administre."*

### Anotaciones de Swagger / OpenAPI

#### @Tag

- **Explicación sencilla:** Etiqueta para agrupar endpoints bajo un mismo título en la interfaz gráfica de Swagger.
- **Explicación técnica:** Metadato de OpenAPI que clasifica visualmente los controladores en Swagger UI.
- **¿Qué cambia en nuestro proyecto?** Organiza las rutas en dos secciones claras: "Eventos" y "Lugares (Venues)".

#### @Operation

- **Explicación sencilla:** Describe qué hace una operación concreta en Swagger UI.
- **Explicación técnica:** Define atributos como `summary` y `description` en el contrato OpenAPI.
- **¿Qué cambia en nuestro proyecto?** Hace que cada endpoint muestre una explicación amigable en lugar de su nombre técnico.

#### @ApiResponse

- **Explicación sencilla:** Documenta qué código HTTP devuelve el endpoint y qué significa.
- **Explicación técnica:** Declara formalmente el código de estado (por ejemplo `201` o `200`) y su descripción dentro de la especificación OpenAPI.
- **¿Qué cambia en nuestro proyecto?** Muestra en Swagger qué respuestas esperar de cada llamado.

### Conceptos de Pruebas Unitarias (JUnit 5 y Mockito)

#### @BeforeEach

- **Explicación sencilla:** Código que se ejecuta antes de cada prueba individual.
- **Explicación técnica:** Anotación de ciclo de vida de JUnit 5 que prepara variables o estados limpios antes de cada método anotado con `@Test`.
- **¿Qué cambia en nuestro proyecto?** Inicializa un `validEvent` o `validVenue` limpio antes de cada test para evitar contaminación entre pruebas.

#### when(...) y thenReturn(...)

- **Explicación sencilla:** Entrena al doble de prueba (*mock*). Le indica: *"cuando llamen a este método con estos argumentos, responde esto"*.
- **Explicación técnica:** Método estático de Mockito para configurar el comportamiento programado (*stubbing*) de un objeto simulado.
- **Ejemplo:**
  ```java
  when(eventRepository.save(any(Event.class))).thenReturn(savedMock);
  ```

#### assertEquals(...)

- **Explicación sencilla:** Verifica que el valor obtenido sea idéntico al valor esperado. Si son distintos, el test se detiene y falla en rojo.
- **Explicación técnica:** Aserción básica de JUnit 5 para comparar igualdad entre objetos o primitivos.
- **Ejemplo:**
  ```java
  assertEquals("Conferencia Tech", result.getNombre());
  ```

#### verify(...)

- **Explicación sencilla:** Comprueba si un método del mock fue llamado y cuántas veces ocurrió.
- **Explicación técnica:** Inspección de llamadas en Mockito. Permite asegurar que se invocó una dependencia (`times(1)`) o que nunca fue tocada ante datos inválidos (`never()`).
- **Ejemplo:**
  ```java
  verify(eventRepository, never()).save(any(Event.class));
  ```

---

## 8. Ejemplos

### Registro de un Evento (Camino Feliz)

- **Petición:** `POST http://localhost:8080/api/events`
- **Body:**

```json
{
  "nombre": "Conferencia Latinoamericana de Java",
  "fecha": "2026-11-20",
  "descripcion": "Encuentro de desarrolladores de software"
}
```

- **Respuesta esperada (Status: 201 Created):**

```json
{
  "id": 2,
  "nombre": "Conferencia Latinoamericana de Java",
  "fecha": "2026-11-20",
  "descripcion": "Encuentro de desarrolladores de software"
}
```

### Consulta de Eventos

- **Petición:** `GET http://localhost:8080/api/events`
- **Respuesta esperada con Seeder activo (Status: 200 OK):**

```json
[
  {
    "id": 1,
    "nombre": "Evento de prueba",
    "fecha": "2026-10-20",
    "descripcion": "Evento inicial de Eventify"
  }
]
```

---

## 9. ¿Cómo funciona todo junto?

1. **Arranque:** Spring Boot inicia y escanea los paquetes del proyecto. Detecta los `@Repository`, los `@Service`, los `@RestController` y la clase de configuración `@Configuration`.
2. **Inyección:** Spring crea primero los repositorios en memoria. Luego instancia los servicios inyectándoles el repositorio por constructor. Finalmente, instancia los controladores inyectándoles el servicio por constructor.
3. **Poblado inicial:** Al estar activa la configuración `eventify.seeder.enabled=true`, los métodos `@Bean` de `DataSeeder` se ejecutan, guardando un evento inicial y un lugar inicial en los repositorios.
4. **Llegada de una solicitud:** Un cliente realiza un `POST /api/events`. El `DispatcherServlet` canaliza la petición al método `create` de `EventController`.
5. **Validación:** `EventController` delega a `EventService`. El servicio verifica que el nombre no esté en blanco y que la fecha esté presente.
6. **Almacenamiento:** Si la validación es exitosa, el servicio llama a `EventRepository.save()`, el cual asigna el ID secuencial y añade el objeto a su `List`.
7. **Respuesta:** El objeto creado vuelve al controlador, que lo empaqueta con código `201 Created` y lo devuelve en formato JSON al cliente.

---

## 10. Cómo probar la Historia de Usuario

### 1. Iniciar la aplicación

Ejecuta en la terminal:

```bash
./mvnw spring-boot:run
```

O corre la clase `EventifyApplication.java` directamente desde tu entorno de desarrollo.

### 2. Verificar la Documentación Swagger (Escenario 4)

- Abre el navegador en: `http://localhost:8080/swagger-ui.html`
- Comprueba que figuren las secciones **Eventos** y **Lugares (Venues)** con sus endpoints `GET` y `POST` documentados.

### 3. Probar el Camino Feliz (Escenario 1)

- En Swagger UI, despliega `POST /api/events` y presiona **Try it out**.
- Ingresa un evento con nombre y fecha válidos.
- Presiona **Execute** y valida que el estado de respuesta sea `201 Created`.

### 4. Probar el Catálogo Vacío (Escenario 3)

- Detén la aplicación.
- En `src/main/resources/application.properties`, cambia el valor:
  ```properties
  eventify.seeder.enabled=false
  ```
- Reinicia la aplicación y ejecuta `GET /api/events` desde Swagger o el navegador.
- Confirma que responde `200 OK` con un arreglo vacío `[]`.

### 5. Ejecutar la Suite de Pruebas Unitarias

Ejecuta en la terminal:

```bash
./mvnw test
```

Todas las pruebas de `EventServiceTest` y `VenueServiceTest` deben ejecutarse en milisegundos y pasar en verde.

---

## 11. Errores comunes

### Error 1: `NullPointerException` al ejecutar los tests unitarios

- **Qué significa:** El servicio intentó llamar a un método del repositorio, pero la referencia es `null`.
- **Por qué ocurre:** Se omitió `@ExtendWith(MockitoExtension.class)` en la cabecera de la clase de prueba, o faltó la anotación `@Mock` sobre el repositorio o `@InjectMocks` sobre el servicio.
- **Qué revisar:** Que la clase de prueba tenga:
  ```java
  @ExtendWith(MockitoExtension.class)
  class EventServiceTest {
      @Mock private EventRepository eventRepository;
      @InjectMocks private EventService eventService;
  ```

### Error 2: Error 404 al acceder a `/swagger-ui.html`

- **Qué significa:** El navegador no encuentra la interfaz gráfica de Swagger.
- **Por qué ocurre:** Se utilizó una dependencia antigua (como `springfox`) en lugar del starter oficial para Spring Boot 3 (`springdoc-openapi-starter-webmvc-ui` versión 2.x).
- **Qué revisar:** Validar la presencia de `springdoc-openapi-starter-webmvc-ui` en el `pom.xml`.

### Error 3: El seeder no carga datos

- **Qué significa:** Al hacer `GET` el catálogo aparece vacío a pesar de esperar datos iniciales.
- **Por qué ocurre:** La propiedad `eventify.seeder.enabled` está en `false` en `application.properties` o el nombre del parámetro no coincide con el definido en `@ConditionalOnProperty`.
- **Qué revisar:** Que la propiedad en `application.properties` sea exactamente `eventify.seeder.enabled=true`.

---

## 12. Checklist final

- [ ] Dependencias agregadas en `pom.xml` (`web`, `lombok`, `springdoc-openapi`, `test`).
- [ ] Entidades POJO `Event` y `Venue` creadas con sus atributos correspondientes.
- [ ] Repositorios en memoria implementados con `@Repository` y listas `List`.
- [ ] Servicios implementados con `@Service` con validación de nombre obligatorio.
- [ ] Inyección de dependencias aplicada estrictamente por constructor en todas las capas.
- [ ] Endpoints implementados con `@RestController` respondiendo `201 Created` en POST y `200 OK` en GET.
- [ ] Documentación OpenAPI añadida con `@Tag`, `@Operation` y `@ApiResponse`.
- [ ] Seeder configurado con `@Configuration` y métodos `@Bean` que devuelven las entidades guardadas.
- [ ] Pruebas unitarias aisladas implementadas con JUnit 5 y Mockito (`@BeforeEach`, `when`, `thenReturn`, `assertEquals`, `verify`).
- [ ] Escenario 1 probado: Registro exitoso con estado `201 Created`.
- [ ] Escenario 2 probado: Intento de registro con nombre vacío bloqueado por el servicio sin llegar al repositorio.
- [ ] Escenario 3 probado: Catálogo vacío retornando `[]` con estado `200 OK`.
- [ ] Escenario 4 probado: Swagger UI accesible y operativo en `/swagger-ui.html`.

---

## 13. ¿Qué aprendimos?

- **Inyección por constructor:** Asegura componentes desacoplados, facilita la inmutabilidad y permite probar clases unitariamente sin depender del framework.
- **Separación de responsabilidades:** El controlador solo atiende la red, el servicio hace cumplir las reglas de negocio y el repositorio guarda la información.
- **Testing aislado con Mockito:** Aprendimos a aislar una clase de negocio para probar caminos felices y caminos de error en milisegundos mediante `@Mock` y verificaciones con `verify(..., never())`.
- **Sembrado de datos declarativo:** Comprendimos cómo Spring ejecuta métodos anotados con `@Bean` dentro de una clase `@Configuration` durante la inicialización de la aplicación.

---

## 14. Mini repaso

1. ¿Por qué es recomendable utilizar inyección de dependencias por constructor en lugar de inyectar dependencias con `@Autowired` en atributos privados?
2. ¿Qué diferencia de responsabilidad existe entre la anotación `@RestController` y la anotación `@Service`?
3. En una prueba con Mockito, ¿para qué sirve la instrucción `verify(eventRepository, never()).save(any())`?
4. ¿Qué ventaja ofrece retornar una copia (`new ArrayList<>(database)`) en lugar de la lista original en el repositorio en memoria?
5. ¿Qué papel cumple la anotación `@BeforeEach` dentro de una clase de pruebas unitarias?
