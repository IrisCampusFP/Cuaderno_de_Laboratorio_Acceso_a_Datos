## 6.1 Controladores REST y Mejores Prácticas

**Implementación de controlador REST educativo:**

@RestController
@RequestMapping("/api/estudiantes")
@Validated
@CrossOrigin(origins = "http://localhost:3000") // Para desarrollo con frontend
public class EstudianteController {
    
    @Autowired
    private EstudianteService estudianteService;
    
    @GetMapping
    public ResponseEntity<Page<EstudianteResponse>> obtenerTodosLosEstudiantes(
            @RequestParam(defaultValue = "0") int pagina,
            @RequestParam(defaultValue = "10") int tamaño,
            @RequestParam(defaultValue = "id") String ordenarPor) {
        
        Pageable pageable = PageRequest.of(pagina, tamaño, Sort.by(ordenarPor));
        Page<EstudianteResponse> estudiantes = estudianteService.findAll(pageable);
        return ResponseEntity.ok(estudiantes);
    }
    
    @PostMapping
    public ResponseEntity<EstudianteResponse> crearEstudiante(
            @Valid @RequestBody EstudianteSolicitud solicitud) {
        
        EstudianteResponse response = estudianteService.crearEstudiante(solicitud);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<EstudianteResponse> obtenerEstudiante(
            @PathVariable @Min(1) Long id) {
        
        EstudianteResponse estudiante = estudianteService.findById(id);
        return ResponseEntity.ok(estudiante);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<EstudianteResponse> actualizarEstudiante(
            @PathVariable Long id,
            @Valid @RequestBody EstudianteSolicitud solicitud) {
        
        EstudianteResponse response = estudianteService.actualizarEstudiante(id, solicitud);
        return ResponseEntity.ok(response);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminarEstudiante(@PathVariable Long id) {
        estudianteService.eliminarEstudiante(id);
        return ResponseEntity.noContent().build();
    }
}

## 6.2 Validación de Datos y Manejo de Errores

**DTO con validaciones completas:**

public class EstudianteSolicitud {
    
    @NotBlank(message = "El nombre es obligatorio")
    @Size(min = 2, max = 50, message = "El nombre debe tener entre 2 y 50 caracteres")
    private String nombre;
    
    @NotNull(message = "El email es obligatorio")
    @Email(message = "Debe proporcionar un email válido")
    private String email;
    
    @NotNull(message = "La edad es obligatoria")
    @Min(value = 18, message = "El estudiante debe tener al menos 18 años")
    @Max(value = 100, message = "La edad debe ser menor a 100")
    private Integer edad;
    
    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Formato de número de teléfono inválido")
    private String numeroTelefono;
    
    @Valid // Validación de objeto anidado
    private Direccion direccion;
    
    // Getters y setters
}

**Manejador global de excepciones:**

@RestControllerAdvice
public class ManejadorExcepcionGlobal {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<RespuestaErrorValidacion> manejarExcepcionesValidacion(
            MethodArgumentNotValidException ex) {
        
        Map<String, String> errores = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> 
            errores.put(error.getField(), error.getDefaultMessage())
        );
        
        RespuestaErrorValidacion respuesta = RespuestaErrorValidacion.builder()
            .timestamp(LocalDateTime.now())
            .estado(HttpStatus.BAD_REQUEST.value())
            .error("Validación Fallida")
            .mensaje("La validación de entrada falló")
            .erroresCampos(errores)
            .build();
            
        return new ResponseEntity<>(respuesta, HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(EntidadNoEncontradaException.class)
    public ResponseEntity<RespuestaError> manejarEntidadNoEncontrada(
            EntidadNoEncontradaException ex) {
        
        RespuestaError respuesta = RespuestaError.builder()
            .timestamp(LocalDateTime.now())
            .estado(HttpStatus.NOT_FOUND.value())
            .error("Recurso No Encontrado")
            .mensaje(ex.getMessage())
            .build();
            
        return new ResponseEntity<>(respuesta, HttpStatus.NOT_FOUND);
    }
}