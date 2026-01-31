## 7.1 Enfoque Pedagógico Basado en Evidencia

La enseñanza efectiva de Spring Boot requiere un enfoque **PRIMM (Predict, Run, Investigate, Modify, Make)** basado en investigación educativa:

**Predict (Predecir):** Los estudiantes examinan aplicaciones Spring Boot de ejemplo y predicen la funcionalidad antes de ejecutar el código.

**Run (Ejecutar):** Los estudiantes ejecutan aplicaciones existentes para verificar sus predicciones y observar el comportamiento real.

**Investigate (Investigar):** Análisis profundo de la estructura del código, anotaciones y patrones de configuración.

**Modify (Modificar):** Realización de cambios incrementales a aplicaciones existentes para comprender el impacto de modificaciones específicas.

**Make (Crear):** Desarrollo de aplicaciones originales utilizando los patrones aprendidos, progresando desde aplicaciones simples CRUD hasta microservicios complejos.

## 7.2 Progresión de Aprendizaje Estructurada

**Semana 1-2: Fundamentos y Configuración**

- Configuración del entorno de desarrollo
    
- Creación del primer proyecto Spring Boot
    
- Comprensión de la estructura de proyecto
    
- Implementación de controladores REST básicos

**Semana 3-4: Acceso a Datos Básico**

- Configuración de base de datos H2
    
- Creación de entidades JPA
    
- Implementación de repositorios
    
- Operaciones CRUD básicas

**Semana 5-6: APIs REST Avanzadas**

- Implementación de validación de datos
    
- Manejo de excepciones
    
- Paginación y ordenamiento
    
- Documentación de API

**Semana 7-8: Seguridad e Implementación**

- Configuración de Spring Security
    
- Autenticación y autorización
    
- Implementación JWT
    
- Testing de seguridad

## 7.3 Proyecto Integrador: Sistema de Gestión Académica

**Descripción del Proyecto:** Desarrollo de un sistema completo de gestión académica que incluye gestión de estudiantes, cursos, profesores y matrículas. Este proyecto integra todos los conceptos aprendidos en un contexto real y relevante.

**Funcionalidades requeridas:**

// Entidad Curso con relaciones complejas
@Entity
@Table(name = "cursos")
public class Curso {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    @NotBlank(message = "El título del curso es obligatorio")
    private String titulo;
    
    @Column(columnDefinition = "TEXT")
    private String descripcion;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "profesor_id")
    private Profesor profesor;
    
    @OneToMany(mappedBy = "curso", fetch = FetchType.LAZY)
    private Set<Matricula> matriculas = new HashSet<>();
    
    @Column(name = "creditos")
    @Min(value = 1, message = "El curso debe tener al menos 1 crédito")
    @Max(value = 10, message = "El curso no puede tener más de 10 créditos")
    private Integer creditos;
    
    // Constructores, getters, setters, equals, hashCode
}

**Controlador con operaciones avanzadas:**

@RestController
@RequestMapping("/api/cursos")
@PreAuthorize("hasRole('ADMIN') or hasRole('PROFESOR')")
public class CursoController {
    
    @Autowired
    private CursoService cursoService;
    
    @GetMapping("/buscar")
    public ResponseEntity<Page<CursoResponse>> buscarCursos(
            @RequestParam(required = false) String titulo,
            @RequestParam(required = false) String profesor,
            @RequestParam(defaultValue = "0") int pagina,
            @RequestParam(defaultValue = "10") int tamaño) {
        
        CursoCriteriosBusqueda criterios = CursoCriteriosBusqueda.builder()
            .titulo(titulo)
            .nombreProfesor(profesor)
            .build();
            
        Pageable pageable = PageRequest.of(pagina, tamaño);
        Page<CursoResponse> cursos = cursoService.buscarCursos(criterios, pageable);
        
        return ResponseEntity.ok(cursos);
    }
    
    @PostMapping("/{cursoId}/matricular/{estudianteId}")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<MatriculaResponse> matricularEstudiante(
            @PathVariable Long cursoId,
            @PathVariable Long estudianteId) {
        
        MatriculaResponse matricula = cursoService.matricularEstudiante(cursoId, estudianteId);
        return ResponseEntity.status(HttpStatus.CREATED).body(matricula);
    }
}

## 7.4 Estrategias de Evaluación Integral

**Evaluación formativa:**

- Revisiones de código semanales con retroalimentación específica
    
- Mini-proyectos incrementales que construyen habilidades progresivamente
    
- Mapas conceptuales para visualizar relaciones entre componentes del framework
    
- Desafíos de debugging para desarrollar habilidades de resolución de problemas

**Evaluación sumativa:**

- Portafolio de proyectos que demuestra progresión de habilidades
    
- Exámenes prácticos hands-on en entorno controlado
    
- Documentos de diseño de arquitectura e implementación
    
- Presentación y defensa oral de decisiones de proyecto