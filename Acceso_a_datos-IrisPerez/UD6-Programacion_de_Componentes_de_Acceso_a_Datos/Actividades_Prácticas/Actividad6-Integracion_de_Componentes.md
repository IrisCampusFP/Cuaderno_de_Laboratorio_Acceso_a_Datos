## Objetivos

- Integrar multiples fuentes de datos en una aplicacion
    
- Configurar varios DataSources
    
- Implementar servicios que combinen diferentes repositorios
    

## Ejercicio 1: Configuracion Multi-DataSource

### Enunciado

Configura una aplicacion Spring Boot con dos DataSources:

1. PostgreSQL para datos principales (libros, usuarios)
    
2. H2 para logs y auditoria
    

Cada DataSource debe tener su propio EntityManager y gestor de transacciones.

Ver solucion

**application.properties:**

# DataSource Principal (PostgreSQL)
spring.datasource.primary.url=jdbc:postgresql://localhost:5432/biblioteca
spring.datasource.primary.username=biblioteca
spring.datasource.primary.password=biblioteca123
spring.datasource.primary.driver-class-name=org.postgresql.Driver

# DataSource Auditoria (H2)
spring.datasource.audit.url=jdbc:h2:mem:auditdb
spring.datasource.audit.username=sa
spring.datasource.audit.password=
spring.datasource.audit.driver-class-name=org.h2.Driver

**PrimaryDataSourceConfig.java:**

@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    basePackages = "com.biblioteca.repository.primary",
    entityManagerFactoryRef = "primaryEntityManagerFactory",
    transactionManagerRef = "primaryTransactionManager"
)
public class PrimaryDataSourceConfig {
    
    @Primary
    @Bean
    @ConfigurationProperties("spring.datasource.primary")
    public DataSourceProperties primaryDataSourceProperties() {
        return new DataSourceProperties();
    }
    
    @Primary
    @Bean
    public DataSource primaryDataSource() {
        return primaryDataSourceProperties()
            .initializeDataSourceBuilder()
            .type(HikariDataSource.class)
            .build();
    }
    
    @Primary
    @Bean
    public LocalContainerEntityManagerFactoryBean primaryEntityManagerFactory(
            EntityManagerFactoryBuilder builder) {
        return builder
            .dataSource(primaryDataSource())
            .packages("com.biblioteca.model.primary")
            .persistenceUnit("primary")
            .properties(hibernateProperties())
            .build();
    }
    
    @Primary
    @Bean
    public PlatformTransactionManager primaryTransactionManager(
            @Qualifier("primaryEntityManagerFactory") EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
    
    private Map<String, Object> hibernateProperties() {
        Map<String, Object> props = new HashMap<>();
        props.put("hibernate.hbm2ddl.auto", "update");
        props.put("hibernate.dialect", "org.hibernate.dialect.PostgreSQLDialect");
        return props;
    }
}

**AuditDataSourceConfig.java:**

@Configuration
@EnableJpaRepositories(
    basePackages = "com.biblioteca.repository.audit",
    entityManagerFactoryRef = "auditEntityManagerFactory",
    transactionManagerRef = "auditTransactionManager"
)
public class AuditDataSourceConfig {
    
    @Bean
    @ConfigurationProperties("spring.datasource.audit")
    public DataSourceProperties auditDataSourceProperties() {
        return new DataSourceProperties();
    }
    
    @Bean
    public DataSource auditDataSource() {
        return auditDataSourceProperties()
            .initializeDataSourceBuilder()
            .type(HikariDataSource.class)
            .build();
    }
    
    @Bean
    public LocalContainerEntityManagerFactoryBean auditEntityManagerFactory(
            EntityManagerFactoryBuilder builder) {
        return builder
            .dataSource(auditDataSource())
            .packages("com.biblioteca.model.audit")
            .persistenceUnit("audit")
            .properties(hibernateProperties())
            .build();
    }
    
    @Bean
    public PlatformTransactionManager auditTransactionManager(
            @Qualifier("auditEntityManagerFactory") EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
    
    private Map<String, Object> hibernateProperties() {
        Map<String, Object> props = new HashMap<>();
        props.put("hibernate.hbm2ddl.auto", "create-drop");
        props.put("hibernate.dialect", "org.hibernate.dialect.H2Dialect");
        return props;
    }
}

## Ejercicio 2: Entidades y Repositorios Separados

### Enunciado

Crea las entidades y repositorios para cada DataSource:

1. **Primary**: Libro (id, titulo, autor, isbn)
    
2. **Audit**: LogAcceso (id, usuario, accion, fecha, entidad)
    

Ver solucion

**Libro.java (model.primary):**

package com.biblioteca.model.primary;

@Entity
@Table(name = "libros")
@Data
public class Libro {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String titulo;
    
    private String autor;
    
    @Column(unique = true)
    private String isbn;
}

**LogAcceso.java (model.audit):**

package com.biblioteca.model.audit;

@Entity
@Table(name = "logs_acceso")
@Data
public class LogAcceso {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String usuario;
    
    @Enumerated(EnumType.STRING)
    private TipoAccion accion;
    
    private LocalDateTime fecha;
    
    private String entidad;
    
    private Long entidadId;
    
    @PrePersist
    public void prePersist() {
        this.fecha = LocalDateTime.now();
    }
}

public enum TipoAccion {
    CREAR, LEER, ACTUALIZAR, ELIMINAR
}

**LibroRepository.java (repository.primary):**

package com.biblioteca.repository.primary;

@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    Optional<Libro> findByIsbn(String isbn);
    List<Libro> findByAutorContainingIgnoreCase(String autor);
}

**LogAccesoRepository.java (repository.audit):**

package com.biblioteca.repository.audit;

@Repository
public interface LogAccesoRepository extends JpaRepository<LogAcceso, Long> {
    List<LogAcceso> findByUsuario(String usuario);
    List<LogAcceso> findByEntidadAndEntidadId(String entidad, Long entidadId);
    List<LogAcceso> findByFechaBetween(LocalDateTime desde, LocalDateTime hasta);
}

## Ejercicio 3: Servicio de Auditoria con AOP

### Enunciado

Implementa un aspecto que registre automaticamente todas las operaciones CRUD en la BD de auditoria.

Ver solucion

**AuditoriaAspect.java:**

@Aspect
@Component
@RequiredArgsConstructor
public class AuditoriaAspect {
    
    private final LogAccesoRepository logRepository;
    
    @AfterReturning(value = "execution(* com.biblioteca.service.*Service.guardar(..))", 
                    returning = "resultado")
    public void auditarCreacion(JoinPoint jp, Object resultado) {
        registrarLog(TipoAccion.CREAR, resultado);
    }
    
    @AfterReturning(value = "execution(* com.biblioteca.service.*Service.buscar*(..))",
                    returning = "resultado")
    public void auditarLectura(JoinPoint jp, Object resultado) {
        if (resultado != null) {
            registrarLog(TipoAccion.LEER, resultado);
        }
    }
    
    @AfterReturning(value = "execution(* com.biblioteca.service.*Service.actualizar(..))",
                    returning = "resultado")
    public void auditarActualizacion(JoinPoint jp, Object resultado) {
        registrarLog(TipoAccion.ACTUALIZAR, resultado);
    }
    
    @After("execution(* com.biblioteca.service.*Service.eliminar(..)) && args(id)")
    public void auditarEliminacion(JoinPoint jp, Long id) {
        LogAcceso log = new LogAcceso();
        log.setUsuario(obtenerUsuarioActual());
        log.setAccion(TipoAccion.ELIMINAR);
        log.setEntidad(obtenerNombreEntidad(jp));
        log.setEntidadId(id);
        logRepository.save(log);
    }
    
    private void registrarLog(TipoAccion accion, Object entidad) {
        LogAcceso log = new LogAcceso();
        log.setUsuario(obtenerUsuarioActual());
        log.setAccion(accion);
        log.setEntidad(entidad.getClass().getSimpleName());
        log.setEntidadId(obtenerIdEntidad(entidad));
        logRepository.save(log);
    }
    
    private String obtenerUsuarioActual() {
        try {
            return SecurityContextHolder.getContext()
                .getAuthentication().getName();
        } catch (Exception e) {
            return "anonimo";
        }
    }
    
    private Long obtenerIdEntidad(Object entidad) {
        try {
            return (Long) entidad.getClass().getMethod("getId").invoke(entidad);
        } catch (Exception e) {
            return null;
        }
    }
    
    private String obtenerNombreEntidad(JoinPoint jp) {
        String serviceName = jp.getTarget().getClass().getSimpleName();
        return serviceName.replace("Service", "");
    }
}

**Habilitar AOP en pom.xml:**

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>

## Ejercicio 4: Integracion con Sistema de Archivos

### Enunciado

Crea un servicio que integre:

1. Exportar libros a JSON (sistema de archivos)
    
2. Importar libros desde JSON
    
3. Registrar operaciones en BD de auditoria
    

Ver solucion

**ExportImportService.java:**

@Service
@RequiredArgsConstructor
public class ExportImportService {
    
    private final LibroRepository libroRepository;
    private final LogAccesoRepository logRepository;
    private final ObjectMapper objectMapper;
    
    @Value("${app.export.path:./exports}")
    private String exportPath;
    
    @Transactional("primaryTransactionManager")
    public Path exportarLibros() throws IOException {
        List<Libro> libros = libroRepository.findAll();
        
        Path dir = Paths.get(exportPath);
        Files.createDirectories(dir);
        
        String filename = "libros_" + 
            LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss")) + 
            ".json";
        Path filePath = dir.resolve(filename);
        
        objectMapper.writerWithDefaultPrettyPrinter()
            .writeValue(filePath.toFile(), libros);
        
        // Registrar en auditoria
        LogAcceso log = new LogAcceso();
        log.setUsuario("sistema");
        log.setAccion(TipoAccion.LEER);
        log.setEntidad("EXPORT_LIBROS");
        logRepository.save(log);
        
        return filePath;
    }
    
    @Transactional("primaryTransactionManager")
    public ImportResult importarLibros(Path archivo) throws IOException {
        List<Libro> libros = objectMapper.readValue(
            archivo.toFile(),
            new TypeReference<List<Libro>>() {}
        );
        
        int nuevos = 0;
        int actualizados = 0;
        int errores = 0;
        
        for (Libro libro : libros) {
            try {
                Optional<Libro> existente = libroRepository.findByIsbn(libro.getIsbn());
                if (existente.isPresent()) {
                    Libro lib = existente.get();
                    lib.setTitulo(libro.getTitulo());
                    lib.setAutor(libro.getAutor());
                    libroRepository.save(lib);
                    actualizados++;
                } else {
                    libro.setId(null); // Asegurar nuevo
                    libroRepository.save(libro);
                    nuevos++;
                }
            } catch (Exception e) {
                errores++;
            }
        }
        
        // Registrar en auditoria
        LogAcceso log = new LogAcceso();
        log.setUsuario("sistema");
        log.setAccion(TipoAccion.CREAR);
        log.setEntidad("IMPORT_LIBROS");
        logRepository.save(log);
        
        return new ImportResult(nuevos, actualizados, errores);
    }
}

@Data
@AllArgsConstructor
public class ImportResult {
    private int nuevos;
    private int actualizados;
    private int errores;
}

**Controller:**

@RestController
@RequestMapping("/api/export-import")
@RequiredArgsConstructor
public class ExportImportController {
    
    private final ExportImportService service;
    
    @PostMapping("/exportar")
    public ResponseEntity<String> exportar() throws IOException {
        Path path = service.exportarLibros();
        return ResponseEntity.ok("Exportado a: " + path.toString());
    }
    
    @PostMapping("/importar")
    public ResponseEntity<ImportResult> importar(@RequestParam String archivo) 
            throws IOException {
        ImportResult result = service.importarLibros(Paths.get(archivo));
        return ResponseEntity.ok(result);
    }
}

## Ejercicio 5: Endpoint de Estadisticas Combinadas

### Enunciado

Crea un endpoint `/api/estadisticas` que combine datos de ambas bases de datos:

1. Total de libros (Primary)
    
2. Libros por autor (Primary)
    
3. Accesos recientes (Audit)
    
4. Operaciones por tipo (Audit)
    

Ver solucion

**EstadisticasDTO.java:**

@Data
@Builder
public class EstadisticasDTO {
    private long totalLibros;
    private Map<String, Long> librosPorAutor;
    private List<LogAccesoDTO> accesosRecientes;
    private Map<String, Long> operacionesPorTipo;
    private LocalDateTime generadoEn;
}

@Data
@AllArgsConstructor
public class LogAccesoDTO {
    private String usuario;
    private String accion;
    private String entidad;
    private LocalDateTime fecha;
}

**EstadisticasService.java:**

@Service
@RequiredArgsConstructor
public class EstadisticasService {
    
    private final LibroRepository libroRepository;
    private final LogAccesoRepository logRepository;
    
    @Transactional(readOnly = true)
    public EstadisticasDTO obtenerEstadisticas() {
        // Datos de Primary (PostgreSQL)
        long totalLibros = libroRepository.count();
        
        Map<String, Long> librosPorAutor = libroRepository.findAll().stream()
            .filter(l -> l.getAutor() != null)
            .collect(Collectors.groupingBy(
                Libro::getAutor,
                Collectors.counting()
            ));
        
        // Datos de Audit (H2)
        LocalDateTime hace24h = LocalDateTime.now().minusHours(24);
        List<LogAccesoDTO> accesosRecientes = logRepository
            .findByFechaBetween(hace24h, LocalDateTime.now())
            .stream()
            .limit(10)
            .map(log -> new LogAccesoDTO(
                log.getUsuario(),
                log.getAccion().name(),
                log.getEntidad(),
                log.getFecha()
            ))
            .collect(Collectors.toList());
        
        Map<String, Long> operacionesPorTipo = logRepository.findAll().stream()
            .collect(Collectors.groupingBy(
                log -> log.getAccion().name(),
                Collectors.counting()
            ));
        
        return EstadisticasDTO.builder()
            .totalLibros(totalLibros)
            .librosPorAutor(librosPorAutor)
            .accesosRecientes(accesosRecientes)
            .operacionesPorTipo(operacionesPorTipo)
            .generadoEn(LocalDateTime.now())
            .build();
    }
}

**EstadisticasController.java:**

@RestController
@RequestMapping("/api/estadisticas")
@RequiredArgsConstructor
public class EstadisticasController {
    
    private final EstadisticasService service;
    
    @GetMapping
    public EstadisticasDTO obtenerEstadisticas() {
        return service.obtenerEstadisticas();
    }
}

**Respuesta ejemplo:**

{
  "totalLibros": 150,
  "librosPorAutor": {
    "Cervantes": 3,
    "Garcia Marquez": 5,
    "Borges": 8
  },
  "accesosRecientes": [
    {
      "usuario": "admin",
      "accion": "CREAR",
      "entidad": "Libro",
      "fecha": "2024-01-15T10:30:00"
    }
  ],
  "operacionesPorTipo": {
    "CREAR": 45,
    "LEER": 230,
    "ACTUALIZAR": 12,
    "ELIMINAR": 3
  },
  "generadoEn": "2024-01-15T14:25:00"
}