## Objetivos

- Implementar una API REST completa
    
- Usar DTOs y validacion
    
- Manejar excepciones globalmente
    

## Ejercicio 1: DTOs con Validacion

### Enunciado

Crea los siguientes DTOs con validaciones:

1. `LibroCreateDTO`: Para crear libros (titulo obligatorio, isbn formato valido)
    
2. `LibroUpdateDTO`: Para actualizar (campos opcionales)
    
3. `LibroResponseDTO`: Para respuestas (sin campos sensibles)
    

Ver solucion

@Data
public class LibroCreateDTO {
    
    @NotBlank(message = "El titulo es obligatorio")
    @Size(min = 2, max = 200)
    private String titulo;
    
    @NotBlank(message = "El autor es obligatorio")
    private String autor;
    
    @Pattern(regexp = "\\d{10,13}", message = "ISBN debe tener 10-13 digitos")
    private String isbn;
    
    @Min(value = 1000)
    @Max(value = 2100)
    private Integer anioPublicacion;
}

@Data
public class LibroUpdateDTO {
    @Size(min = 2, max = 200)
    private String titulo;
    
    private String autor;
    private Boolean disponible;
}

@Data
@AllArgsConstructor
public class LibroResponseDTO {
    private Long id;
    private String titulo;
    private String autor;
    private String isbn;
    private Integer anioPublicacion;
    private boolean disponible;
}

## Ejercicio 2: Mapper

### Enunciado

Crea un componente `LibroMapper` que convierta entre Entity y DTOs.

Ver solucion

@Component
public class LibroMapper {
    
    public Libro toEntity(LibroCreateDTO dto) {
        Libro libro = new Libro();
        libro.setTitulo(dto.getTitulo());
        libro.setAutor(dto.getAutor());
        libro.setIsbn(dto.getIsbn());
        libro.setAnioPublicacion(dto.getAnioPublicacion());
        libro.setDisponible(true);
        return libro;
    }
    
    public LibroResponseDTO toDTO(Libro libro) {
        return new LibroResponseDTO(
            libro.getId(),
            libro.getTitulo(),
            libro.getAutor(),
            libro.getIsbn(),
            libro.getAnioPublicacion(),
            libro.isDisponible()
        );
    }
    
    public List<LibroResponseDTO> toDTOList(List<Libro> libros) {
        return libros.stream()
            .map(this::toDTO)
            .collect(Collectors.toList());
    }
    
    public void updateEntity(Libro libro, LibroUpdateDTO dto) {
        if (dto.getTitulo() != null) libro.setTitulo(dto.getTitulo());
        if (dto.getAutor() != null) libro.setAutor(dto.getAutor());
        if (dto.getDisponible() != null) libro.setDisponible(dto.getDisponible());
    }
}

## Ejercicio 3: Manejo Global de Excepciones

### Enunciado

Crea un `@RestControllerAdvice` que maneje:

1. `EntityNotFoundException` -> 404
    
2. `MethodArgumentNotValidException` -> 400
    
3. `Exception` generica -> 500
    

Ver solucion

@Data
@AllArgsConstructor
public class ErrorResponse {
    private String codigo;
    private String mensaje;
    private List<String> detalles;
    private LocalDateTime timestamp;
    
    public ErrorResponse(String codigo, String mensaje) {
        this(codigo, mensaje, Collections.emptyList(), LocalDateTime.now());
    }
}

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(EntityNotFoundException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        List<String> errores = ex.getBindingResult().getFieldErrors()
            .stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.toList());
        
        return new ErrorResponse(
            "VALIDATION_ERROR", 
            "Errores de validacion", 
            errores,
            LocalDateTime.now()
        );
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        return new ErrorResponse("INTERNAL_ERROR", "Error interno del servidor");
    }
}

## Ejercicio 4: Controller Completo

### Enunciado

Actualiza el `LibroController` para usar DTOs, validacion y el mapper.

Ver solucion

@RestController
@RequestMapping("/api/libros")
@RequiredArgsConstructor
public class LibroController {
    
    private final LibroService service;
    private final LibroMapper mapper;
    
    @GetMapping
    public List<LibroResponseDTO> listar() {
        return mapper.toDTOList(service.listarTodos());
    }
    
    @GetMapping("/{id}")
    public LibroResponseDTO buscar(@PathVariable Long id) {
        Libro libro = service.buscarPorId(id)
            .orElseThrow(() -> new EntityNotFoundException("Libro no encontrado: " + id));
        return mapper.toDTO(libro);
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public LibroResponseDTO crear(@RequestBody @Valid LibroCreateDTO dto) {
        Libro libro = mapper.toEntity(dto);
        Libro guardado = service.guardar(libro);
        return mapper.toDTO(guardado);
    }
    
    @PutMapping("/{id}")
    public LibroResponseDTO actualizar(@PathVariable Long id, 
                                        @RequestBody @Valid LibroUpdateDTO dto) {
        Libro libro = service.buscarPorId(id)
            .orElseThrow(() -> new EntityNotFoundException("Libro no encontrado: " + id));
        
        mapper.updateEntity(libro, dto);
        Libro actualizado = service.guardar(libro);
        return mapper.toDTO(actualizado);
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void eliminar(@PathVariable Long id) {
        if (!service.existePorId(id)) {
            throw new EntityNotFoundException("Libro no encontrado: " + id);
        }
        service.eliminar(id);
    }
}