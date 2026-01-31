## Objetivos

- Crear un proyecto Spring Boot desde cero
    
- Configurar dependencias con Maven
    
- Implementar un endpoint REST basico
    
- Configurar propiedades de la aplicacion
    

## Ejercicio 1: Crear Proyecto

### Enunciado

Crea un nuevo proyecto Spring Boot con las siguientes especificaciones:

- **Group**: com.biblioteca
    
- **Artifact**: biblioteca-api
    
- **Java**: 17
    
- **Dependencias**: Spring Web, Spring Data JPA, H2 Database, Lombok
    

Puedes usar Spring Initializr ([start.spring.io](http://start.spring.io/)) o IntelliJ IDEA.

Ver solucion

**Opcion 1: Spring Initializr**

1. Ir a [https://start.spring.io](https://start.spring.io/)
    
2. Configurar:
    
    - Project: Maven
        
    - Language: Java
        
    - Spring Boot: 3.2.x
        
    - Group: com.biblioteca
        
    - Artifact: biblioteca-api
        
    - Packaging: Jar
        
    - Java: 17
        
3. Agregar dependencias: Spring Web, Spring Data JPA, H2 Database, Lombok
    
4. Click «Generate»
    

**pom.xml resultante:**

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
    <name>Biblioteca API</name>
    
    <properties>
        <java.version>17</java.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
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

## Ejercicio 2: Configurar Propiedades

### Enunciado

Configura el archivo `application.properties` con:

1. Puerto del servidor: 8081
    
2. Nombre de la aplicacion: Biblioteca API
    
3. Base de datos H2 en memoria llamada «bibliotecadb»
    
4. Consola H2 habilitada
    
5. Mostrar SQL de Hibernate
    
6. Crear esquema automaticamente
    

Ver solucion

**src/main/resources/application.properties:**

# Servidor
server.port=8081
spring.application.name=Biblioteca API

# Base de datos H2
spring.datasource.url=jdbc:h2:mem:bibliotecadb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# Consola H2
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# JPA/Hibernate
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Logging
logging.level.org.hibernate.SQL=DEBUG
logging.level.com.biblioteca=DEBUG

**Acceso a la consola H2:** [http://localhost:8081/h2-console](http://localhost:8081/h2-console)

## Ejercicio 3: Crear Entidad y Repository

### Enunciado

Crea una entidad `Libro` con los siguientes campos:

- id (Long, autogenerado)
    
- titulo (String, obligatorio, max 200)
    
- autor (String, max 100)
    
- isbn (String, unico, 13 caracteres)
    
- anioPublicacion (Integer)
    
- disponible (boolean, por defecto true)
    

Crea tambien el Repository correspondiente.

Ver solucion

**src/main/java/com/biblioteca/model/Libro.java:**

package com.biblioteca.model;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Table(name = "libros")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Libro {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 200)
    private String titulo;
    
    @Column(length = 100)
    private String autor;
    
    @Column(unique = true, length = 13)
    private String isbn;
    
    private Integer anioPublicacion;
    
    private boolean disponible = true;
}

**src/main/java/com/biblioteca/repository/LibroRepository.java:**

package com.biblioteca.repository;

import com.biblioteca.model.Libro;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    List<Libro> findByDisponibleTrue();
    Optional<Libro> findByIsbn(String isbn);
    List<Libro> findByAutorContainingIgnoreCase(String autor);
}

## Ejercicio 4: Crear Controller REST

### Enunciado

Crea un controlador REST `LibroController` con los siguientes endpoints:

|Metodo|URL|Descripcion|
|---|---|---|
|GET|/api/libros|Listar todos|
|GET|/api/libros/{id}|Buscar por ID|
|POST|/api/libros|Crear libro|
|PUT|/api/libros/{id}|Actualizar libro|
|DELETE|/api/libros/{id}|Eliminar libro|

Ver solucion

**src/main/java/com/biblioteca/controller/LibroController.java:**

package com.biblioteca.controller;

import com.biblioteca.model.Libro;
import com.biblioteca.repository.LibroRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/libros")
@RequiredArgsConstructor
public class LibroController {
    
    private final LibroRepository repository;
    
    @GetMapping
    public List<Libro> listar() {
        return repository.findAll();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Libro> buscar(@PathVariable Long id) {
        return repository.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Libro crear(@RequestBody Libro libro) {
        return repository.save(libro);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Libro> actualizar(@PathVariable Long id, 
                                             @RequestBody Libro libro) {
        return repository.findById(id)
            .map(existente -> {
                libro.setId(id);
                return ResponseEntity.ok(repository.save(libro));
            })
            .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminar(@PathVariable Long id) {
        return repository.findById(id)
            .map(libro -> {
                repository.delete(libro);
                return ResponseEntity.noContent().<Void>build();
            })
            .orElse(ResponseEntity.notFound().build());
    }
}

## Ejercicio 5: Probar la API

### Enunciado

Ejecuta la aplicacion y prueba los endpoints usando curl o Postman:

1. Crear un libro
    
2. Listar todos los libros
    
3. Buscar el libro creado por ID
    
4. Actualizar el libro
    
5. Eliminar el libro
    

Ver solucion

**1. Ejecutar aplicacion:**

mvn spring-boot:run

**2. Crear libro:**

curl -X POST http://localhost:8081/api/libros \
  -H "Content-Type: application/json" \
  -d '{"titulo": "Don Quijote", "autor": "Cervantes", "isbn": "9788420412146", "anioPublicacion": 1605}'

**Respuesta:**

{
  "id": 1,
  "titulo": "Don Quijote",
  "autor": "Cervantes",
  "isbn": "9788420412146",
  "anioPublicacion": 1605,
  "disponible": true
}

**3. Listar libros:**

curl http://localhost:8081/api/libros

**4. Buscar por ID:**

curl http://localhost:8081/api/libros/1

**5. Actualizar:**

curl -X PUT http://localhost:8081/api/libros/1 \
  -H "Content-Type: application/json" \
  -d '{"titulo": "Don Quijote de la Mancha", "autor": "Miguel de Cervantes", "isbn": "9788420412146", "anioPublicacion": 1605}'

**6. Eliminar:**

curl -X DELETE http://localhost:8081/api/libros/1