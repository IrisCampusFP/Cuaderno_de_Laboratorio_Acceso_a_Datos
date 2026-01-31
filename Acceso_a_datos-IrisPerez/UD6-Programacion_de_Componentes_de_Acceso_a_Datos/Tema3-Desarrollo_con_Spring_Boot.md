## Objetivos

- Comprender las ventajas de Spring Boot sobre Spring Framework
    
- Crear proyectos Spring Boot desde cero
    
- Conocer los starters y la autoconfiguracion
    
- Configurar aplicaciones con application.properties/yaml

## 1. Que es Spring Boot?

**Spring Boot** facilita la creacion de aplicaciones Spring de produccion con configuracion minima.

### 1.1 Caracteristicas Principales

|Caracteristica|Descripcion|
|---|---|
|**Autoconfiguracion**|Configura beans automaticamente|
|**Starters**|Dependencias preconfiguradas|
|**Servidor embebido**|Tomcat/Jetty incluido|
|**Production-ready**|Metricas, health checks|
|**Sin XML**|Configuracion solo con anotaciones|

### 1.2 Spring Boot vs Spring Framework

┌─────────────────────────────────────────────────────────────────┐
│           SPRING FRAMEWORK (Tradicional)                        │
├─────────────────────────────────────────────────────────────────┤
│  • Configuracion XML extensa                                    │
│  • Gestion manual de dependencias                               │
│  • Configurar servidor web externo                              │
│  • Multiples archivos de configuracion                          │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│              SPRING BOOT (Moderno)                              │
├─────────────────────────────────────────────────────────────────┤
│  • Configuracion automatica                                     │
│  • Starters con dependencias compatibles                        │
│  • Servidor embebido (Tomcat)                                   │
│  • Un solo archivo application.properties                       │
│  • Ejecutar con java -jar                                       │
└─────────────────────────────────────────────────────────────────┘

## 2. Crear Proyecto Spring Boot

### 2.1 Usando Spring Initializr

```
Acceder a [start.spring.io](https://start.spring.io/) o usar el asistente de IntelliJ:

Project:     Maven
Language:    Java
Spring Boot: 3.2.x
Group:       com.biblioteca
Artifact:    biblioteca-api
Packaging:   Jar
Java:        17

Dependencies:
  - Spring Web
  - Spring Data JPA
  - PostgreSQL Driver
  - Lombok
  - Spring Boot DevTools
```

### 2.2 Estructura del Proyecto

```
biblioteca-api/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/biblioteca/
│   │   │       ├── BibliotecaApplication.java    # Clase principal
│   │   │       ├── controller/
│   │   │       ├── service/
│   │   │       ├── repository/
│   │   │       ├── model/
│   │   │       └── config/
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── static/         # Archivos estaticos
│   │       └── templates/      # Plantillas Thymeleaf
│   └── test/
│       └── java/
│           └── com/biblioteca/
└── target/

```
### 2.3 pom.xml Basico

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
        <version>3.2.0</version>
    </parent>
    
    <groupId>com.biblioteca</groupId>
    <artifactId>biblioteca-api</artifactId>
    <version>1.0.0</version>
    
    <properties>
        <java.version>17</java.version>
    </properties>
    
    <dependencies>
        <!-- Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <!-- PostgreSQL -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- H2 para desarrollo -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- Testing -->
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
            </plugin>
        </plugins>
    </build>
</project>
```

### 2.4 Clase Principal

```java
package com.biblioteca;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication  // = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class BibliotecaApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(BibliotecaApplication.class, args);
    }
}
```

## 3. Starters de Spring Boot

Los **starters** son dependencias que agrupan librerias relacionadas.

### 3.1 Starters Principales

|Starter|Incluye|
|---|---|
|`spring-boot-starter-web`|Spring MVC, Tomcat, JSON|
|`spring-boot-starter-data-jpa`|JPA, Hibernate, Spring Data|
|`spring-boot-starter-security`|Spring Security|
|`spring-boot-starter-validation`|Bean Validation, Hibernate Validator|
|`spring-boot-starter-test`|JUnit, Mockito, AssertJ|
|`spring-boot-starter-actuator`|Metricas, health checks|
|`spring-boot-starter-mail`|JavaMail|
|`spring-boot-starter-cache`|Cache con Caffeine, EhCache|

### 3.2 Starters de Bases de Datos

|Starter|Base de Datos|
|---|---|
|`spring-boot-starter-data-mongodb`|MongoDB|
|`spring-boot-starter-data-redis`|Redis|
|`spring-boot-starter-data-elasticsearch`|Elasticsearch|

## 4. Autoconfiguracion

### 4.1 Como Funciona

Spring Boot detecta las dependencias y configura beans automaticamente:

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Escanea el classpath                                        │
│  2. Detecta clases de configuracion (@ConditionalOn...)         │
│  3. Evalua condiciones                                          │
│  4. Registra beans si se cumplen las condiciones                │
└─────────────────────────────────────────────────────────────────┘
```

Ejemplo: Si detecta H2 en el classpath, configura DataSource automaticamente.

### 4.2 Condicionales de Autoconfiguracion

```java
@Configuration
@ConditionalOnClass(DataSource.class)  // Si existe la clase
@ConditionalOnProperty(name = "app.datasource.enabled", havingValue = "true")
public class DataSourceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean  // Solo si no existe otro DataSource
    public DataSource dataSource() {
        return new HikariDataSource();
    }
}
```

|Condicional|Descripcion|
|---|---|
|`@ConditionalOnClass`|Si existe una clase|
|`@ConditionalOnMissingBean`|Si no existe un bean|
|`@ConditionalOnProperty`|Si existe una propiedad|
|`@ConditionalOnWebApplication`|Si es aplicacion web|

## 5. Configuracion con Propiedades

### 5.1 application.properties

```properties
# Servidor
server.port=8080
server.servlet.context-path=/api

# Base de datos
spring.datasource.url=jdbc:postgresql://localhost:5432/biblioteca
spring.datasource.username=biblioteca
spring.datasource.password=biblioteca123
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA/Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Logging
logging.level.root=INFO
logging.level.com.biblioteca=DEBUG
logging.level.org.hibernate.SQL=DEBUG

# Aplicacion
app.nombre=Biblioteca Central
app.version=1.0.0
```

### 5.2 application.yml (Alternativa)

```yml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/biblioteca
    username: biblioteca
    password: biblioteca123
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

logging:
  level:
    root: INFO
    com.biblioteca: DEBUG

app:
  nombre: Biblioteca Central
  version: 1.0.0
```

### 5.3 Propiedades por Perfil

```
src/main/resources/
├── application.properties        # Comun
├── application-dev.properties    # Desarrollo
├── application-test.properties   # Testing
└── application-prod.properties   # Produccion
```

```properties
# application.properties (comun)
spring.profiles.active=dev
app.nombre=Biblioteca

# application-dev.properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.jpa.hibernate.ddl-auto=create-drop
logging.level.root=DEBUG

# application-prod.properties
spring.datasource.url=jdbc:postgresql://db.server.com/biblioteca
spring.jpa.hibernate.ddl-auto=validate
logging.level.root=WARN
```

## 6. DevTools y Hot Reload

### 6.1 Configurar DevTools

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

### 6.2 Caracteristicas de DevTools

- **Reinicio automatico** al cambiar codigo
    
- **LiveReload** para cambios en recursos estaticos
    
- **Propiedades de desarrollo** preconfiguradas

```properties
# Opcional: configurar DevTools
spring.devtools.restart.enabled=true
spring.devtools.livereload.enabled=true

```
## 7. Ejecutar la Aplicacion

### 7.1 Desde el IDE

Ejecutar la clase principal `BibliotecaApplication.java`

### 7.2 Desde Maven

```powershell
# Ejecutar en modo desarrollo
mvn spring-boot:run

# Con perfil especifico
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# Compilar JAR
mvn clean package

# Ejecutar JAR
java -jar target/biblioteca-api-1.0.0.jar

# Ejecutar JAR con perfil
java -jar target/biblioteca-api-1.0.0.jar --spring.profiles.active=prod

```

## 8. Actuator (Monitoreo)

### 8.1 Agregar Actuator

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

### 8.2 Configurar Endpoints

# Habilitar endpoints
management.endpoints.web.exposure.include=health,info,metrics,env

# Info de la aplicacion
management.info.env.enabled=true
info.app.name=Biblioteca API
info.app.version=1.0.0

### 8.3 Endpoints Disponibles

|Endpoint|URL|Descripcion|
|---|---|---|
|health|`/actuator/health`|Estado de la aplicacion|
|info|`/actuator/info`|Informacion de la app|
|metrics|`/actuator/metrics`|Metricas del sistema|
|env|`/actuator/env`|Variables de entorno|
|beans|`/actuator/beans`|Lista de beans|

## 9. Ejemplo Completo: API Biblioteca

```java
// Entidad
@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Libro {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String titulo;
    
    private String autor;
    
    @Column(unique = true)
    private String isbn;
    
    private boolean disponible = true;
}

// Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    List<Libro> findByDisponibleTrue();
    Optional<Libro> findByIsbn(String isbn);
    List<Libro> findByAutorContainingIgnoreCase(String autor);
}

// Service
@Service
@Transactional(readOnly = true)
public class LibroService {
    private final LibroRepository repository;
    
    public LibroService(LibroRepository repository) {
        this.repository = repository;
    }
    
    public List<Libro> listarTodos() {
        return repository.findAll();
    }
    
    public Optional<Libro> buscarPorId(Long id) {
        return repository.findById(id);
    }
    
    @Transactional
    public Libro guardar(Libro libro) {
        return repository.save(libro);
    }
    
    @Transactional
    public void eliminar(Long id) {
        repository.deleteById(id);
    }
}

// Controller
@RestController
@RequestMapping("/api/libros")
public class LibroController {
    private final LibroService service;
    
    public LibroController(LibroService service) {
        this.service = service;
    }
    
    @GetMapping
    public List<Libro> listar() {
        return service.listarTodos();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Libro> buscar(@PathVariable Long id) {
        return service.buscarPorId(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Libro crear(@RequestBody Libro libro) {
        return service.guardar(libro);
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void eliminar(@PathVariable Long id) {
        service.eliminar(id);
    }
}
```

## 10. Resumen

### Conceptos Clave

- **Spring Boot** simplifica la configuracion de Spring
    
- Los **starters** agrupan dependencias relacionadas
    
- La **autoconfiguracion** detecta y configura beans automaticamente
    
- `application.properties` o `application.yml` para configuracion
    
- Los **perfiles** permiten configuracion por entorno
    
- **Actuator** proporciona endpoints de monitoreo

### Comandos Maven

```powershell
mvn spring-boot:run         # Ejecutar
mvn clean package           # Compilar JAR
mvn test                    # Ejecutar tests
```