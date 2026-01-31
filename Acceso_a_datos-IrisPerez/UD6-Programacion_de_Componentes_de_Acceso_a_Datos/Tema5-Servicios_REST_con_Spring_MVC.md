## Objetivos

- Comprender los principios de arquitectura REST
    
- Crear controladores REST con Spring MVC
    
- Manejar peticiones HTTP (GET, POST, PUT, DELETE)
    
- Implementar DTOs y validacion

## 1. Arquitectura REST

**REST** (Representational State Transfer) es un estilo arquitectonico para APIs web.

### 1.1 Principios REST

|Principio|Descripcion|
|---|---|
|**Sin estado**|Cada peticion es independiente|
|**Recursos**|URLs identifican recursos|
|**Metodos HTTP**|GET, POST, PUT, DELETE|
|**Representaciones**|JSON, XML|
|**HATEOAS**|Links en respuestas|

### 1.2 Metodos HTTP y Operaciones CRUD

|Metodo|Operacion|URL|Descripcion|
|---|---|---|---|
|**GET**|Read|`/libros`|Listar todos|
|**GET**|Read|`/libros/1`|Obtener uno|
|**POST**|Create|`/libros`|Crear nuevo|
|**PUT**|Update|`/libros/1`|Actualizar completo|
|**PATCH**|Update|`/libros/1`|Actualizar parcial|
|**DELETE**|Delete|`/libros/1`|Eliminar|

## 2. Controladores REST

### 2.1 `@RestController`

```java
@RestController  // = @Controller + @ResponseBody
@RequestMapping("/api/libros")
public class LibroController {
    
    private final LibroService libroService;
    
    public LibroController(LibroService libroService) {
        this.libroService = libroService;
    }
    
    // Endpoints aqui...
}
```

### 2.2 Endpoints CRUD Completo

```java
@RestController
@RequestMapping("/api/libros")
public class LibroController {
    
    private final LibroService service;
    
    // GET /api/libros - Listar todos
    @GetMapping
    public List<Libro> listar() {
        return service.listarTodos();
    }
    
    // GET /api/libros/1 - Obtener por ID
    @GetMapping("/{id}")
    public ResponseEntity<Libro> buscar(@PathVariable Long id) {
        return service.buscarPorId(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    // POST /api/libros - Crear
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Libro crear(@RequestBody Libro libro) {
        return service.guardar(libro);
    }
    
    // PUT /api/libros/1 - Actualizar
    @PutMapping("/{id}")
    public ResponseEntity<Libro> actualizar(@PathVariable Long id, 
                                             @RequestBody Libro libro) {
        return service.buscarPorId(id)
            .map(existente -> {
                libro.setId(id);
                return ResponseEntity.ok(service.guardar(libro));
            })
            .orElse(ResponseEntity.notFound().build());
    }
    
    // DELETE /api/libros/1 - Eliminar
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminar(@PathVariable Long id) {
        return service.buscarPorId(id)
            .map(libro -> {
                service.eliminar(id);
                return ResponseEntity.noContent().<Void>build();
            })
            .orElse(ResponseEntity.notFound().build());
    }
}
```

## 3. Parametros de Peticion

### 3.1 `@PathVariable` - Variables de Ruta

```java
// GET /api/libros/1
@GetMapping("/{id}")
public Libro buscar(@PathVariable Long id) { ... }

// GET /api/usuarios/123/prestamos/456
@GetMapping("/usuarios/{usuarioId}/prestamos/{prestamoId}")
public Prestamo buscarPrestamo(@PathVariable Long usuarioId,
                                @PathVariable Long prestamoId) { ... }
```

### 3.2 `@RequestParam` - Parametros de Query

```java
// GET /api/libros?autor=Cervantes&disponible=true
@GetMapping
public List<Libro> buscar(
        @RequestParam(required = false) String autor,
        @RequestParam(defaultValue = "true") boolean disponible) {
    return service.buscar(autor, disponible);
}

// GET /api/libros?pagina=0&tamanio=10&ordenar=titulo
@GetMapping
public Page<Libro> listar(
        @RequestParam(defaultValue = "0") int pagina,
        @RequestParam(defaultValue = "10") int tamanio,
        @RequestParam(defaultValue = "titulo") String ordenar) {
    return service.listarPaginado(pagina, tamanio, ordenar);
}
```

### 3.3 `@RequestBody` - Cuerpo de Peticion

```java
// POST /api/libros
// Content-Type: application/json
// {"titulo": "Don Quijote", "autor": "Cervantes"}
@PostMapping
public Libro crear(@RequestBody Libro libro) {
    return service.guardar(libro);
}

```
### 3.4 `@RequestHeader` - Cabeceras

```java
@GetMapping
public String saludar(
        @RequestHeader("Accept-Language") String idioma,
        @RequestHeader(value = "Authorization", required = false) String token) {
    return "Idioma: " + idioma;
}
```

## 4. ResponseEntity

`ResponseEntity` permite controlar la respuesta HTTP completa.

```java
@RestController
@RequestMapping("/api/libros")
public class LibroController {
    
    // Respuesta con codigo de estado
    @PostMapping
    public ResponseEntity<Libro> crear(@RequestBody Libro libro) {
        Libro guardado = service.guardar(libro);
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(guardado);
    }
    
    // Respuesta con cabeceras
    @PostMapping
    public ResponseEntity<Libro> crearConLocation(@RequestBody Libro libro) {
        Libro guardado = service.guardar(libro);
        URI location = URI.create("/api/libros/" + guardado.getId());
        return ResponseEntity
            .created(location)
            .body(guardado);
    }
    
    // Not Found
    @GetMapping("/{id}")
    public ResponseEntity<Libro> buscar(@PathVariable Long id) {
        return service.buscarPorId(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    // No Content
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminar(@PathVariable Long id) {
        service.eliminar(id);
        return ResponseEntity.noContent().build();
    }
}
```

## 5. DTOs (Data Transfer Objects)

### 5.1 Por que usar DTOs?

- Separar modelo de dominio de la API
    
- Controlar que datos se exponen
    
- Evitar exponer entidades JPA directamente
    
- Diferentes vistas del mismo recurso

### 5.2 Ejemplo de DTOs

```java
// DTO para crear
@Data
public class LibroCreateDTO {
    @NotBlank(message = "El titulo es obligatorio")
    private String titulo;
    
    @NotBlank
    private String autor;
    
    @Pattern(regexp = "\\d{10,13}")
    private String isbn;
    
    private String genero;
}

// DTO para respuesta
@Data
@AllArgsConstructor
public class LibroResponseDTO {
    private Long id;
    private String titulo;
    private String autor;
    private boolean disponible;
}

// DTO para listado (resumen)
@Data
public class LibroResumenDTO {
    private Long id;
    private String titulo;
}
```

### 5.3 Mapper entre Entity y DTO

```java
@Component
public class LibroMapper {
    
    public LibroResponseDTO toDTO(Libro libro) {
        return new LibroResponseDTO(
            libro.getId(),
            libro.getTitulo(),
            libro.getAutor(),
            libro.isDisponible()
        );
    }
    
    public Libro toEntity(LibroCreateDTO dto) {
        Libro libro = new Libro();
        libro.setTitulo(dto.getTitulo());
        libro.setAutor(dto.getAutor());
        libro.setIsbn(dto.getIsbn());
        libro.setGenero(Genero.valueOf(dto.getGenero()));
        return libro;
    }
    
    public List<LibroResponseDTO> toDTOList(List<Libro> libros) {
        return libros.stream()
            .map(this::toDTO)
            .collect(Collectors.toList());
    }
}

// Uso en Controller
@RestController
@RequestMapping("/api/libros")
public class LibroController {
    private final LibroService service;
    private final LibroMapper mapper;
    
    @GetMapping
    public List<LibroResponseDTO> listar() {
        return mapper.toDTOList(service.listarTodos());
    }
    
    @PostMapping
    public LibroResponseDTO crear(@RequestBody @Valid LibroCreateDTO dto) {
        Libro libro = mapper.toEntity(dto);
        Libro guardado = service.guardar(libro);
        return mapper.toDTO(guardado);
    }
}

```
## 6. Validacion

### 6.1 Configurar Validacion

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 6.2 Anotaciones de Validacion

```java
@Data
public class LibroCreateDTO {
    
    @NotBlank(message = "El titulo es obligatorio")
    @Size(min = 2, max = 200, message = "El titulo debe tener entre 2 y 200 caracteres")
    private String titulo;
    
    @NotBlank
    private String autor;
    
    @Pattern(regexp = "\\d{10,13}", message = "ISBN debe tener 10-13 digitos")
    private String isbn;
    
    @Min(value = 1000, message = "Anio minimo 1000")
    @Max(value = 2100)
    private Integer anioPublicacion;
    
    @Positive
    @DecimalMax("1000.00")
    private BigDecimal precio;
    
    @Email
    private String emailContacto;
}

```
### 6.3 Aplicar Validacion en Controller

```java
@RestController
@RequestMapping("/api/libros")
public class LibroController {
    
    @PostMapping
    public ResponseEntity<LibroResponseDTO> crear(@RequestBody @Valid LibroCreateDTO dto) {
        // Si la validacion falla, lanza MethodArgumentNotValidException
        Libro libro = mapper.toEntity(dto);
        return ResponseEntity.ok(mapper.toDTO(service.guardar(libro)));
    }
}
```

|Anotacion|Descripcion|
|---|---|
|`@NotNull`|No puede ser null|
|`@NotBlank`|No null, no vacio, no espacios|
|`@NotEmpty`|No null, no vacio|
|`@Size`|Longitud minima/maxima|
|`@Min`, `@Max`|Valor minimo/maximo|
|`@Positive`|Mayor que 0|
|`@Email`|Formato email valido|
|`@Pattern`|Expresion regular|

## 7. Manejo de Excepciones

### 7.1 `@ExceptionHandler` Local

```java
@RestController
@RequestMapping("/api/libros")
public class LibroController {
    
    @ExceptionHandler(LibroNoEncontradoException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse manejarNoEncontrado(LibroNoEncontradoException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }
}
```

### 7.2 `@ControllerAdvice` Global

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse manejarNoEncontrado(EntityNotFoundException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse manejarValidacion(MethodArgumentNotValidException ex) {
        List<String> errores = ex.getBindingResult().getFieldErrors()
            .stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.toList());
        return new ErrorResponse("VALIDATION_ERROR", "Errores de validacion", errores);
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse manejarGeneral(Exception ex) {
        return new ErrorResponse("INTERNAL_ERROR", "Error interno del servidor");
    }
}

// DTO para errores
@Data
@AllArgsConstructor
public class ErrorResponse {
    private String codigo;
    private String mensaje;
    private List<String> detalles;
    
    public ErrorResponse(String codigo, String mensaje) {
        this(codigo, mensaje, Collections.emptyList());
    }
}

```
## 8. Documentacion con OpenAPI/Swagger

### 8.1 Configuracion

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

```properties
# application.properties
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
```

### 8.2 Anotaciones OpenAPI

```java
@RestController
@RequestMapping("/api/libros")
@Tag(name = "Libros", description = "API de gestion de libros")
public class LibroController {
    
    @Operation(summary = "Listar todos los libros")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Lista de libros")
    })
    @GetMapping
    public List<LibroResponseDTO> listar() { ... }
    
    @Operation(summary = "Crear un nuevo libro")
    @ApiResponses({
        @ApiResponse(responseCode = "201", description = "Libro creado"),
        @ApiResponse(responseCode = "400", description = "Datos invalidos")
    })
    @PostMapping
    public LibroResponseDTO crear(@RequestBody @Valid LibroCreateDTO dto) { ... }
}
```

Acceder a:

- Swagger UI: `http://localhost:8080/swagger-ui.html`
    
- OpenAPI JSON: `http://localhost:8080/api-docs`

## 9. Resumen

### Anotaciones Principales

|Anotacion|Uso|
|---|---|
|`@RestController`|Controller REST|
|`@RequestMapping`|URL base|
|`@GetMapping`|HTTP GET|
|`@PostMapping`|HTTP POST|
|`@PutMapping`|HTTP PUT|
|`@DeleteMapping`|HTTP DELETE|
|`@PathVariable`|Variable de ruta|
|`@RequestParam`|Parametro query|
|`@RequestBody`|Cuerpo JSON|
|`@Valid`|Activar validacion|
|`@ResponseStatus`|Codigo de estado|