## Objetivos

- Crear repositorios con Spring Data JPA
    
- Usar Query Methods y derivar consultas del nombre
    
- Implementar consultas personalizadas con `@Query`
    
- Aplicar paginacion y ordenacion
    
- Extender repositorios con metodos custom

## Contexto

Sistema de gestion de biblioteca con:

- `Libro`: id, titulo, isbn, fechaPublicacion, disponible, autor
    
- `Autor`: id, nombre, nacionalidad, fechaNacimiento
    
- `Prestamo`: id, libro, usuario, fechaPrestamo, fechaDevolucion
    
- `Usuario`: id, nombre, email, activo

## Ejercicio 1: Repositorios basicos y Query Methods

### Enunciado

Crea los repositorios para `Libro` y `Autor` con los siguientes metodos derivados:

**LibroRepository:**

1. Buscar libros por titulo (contenga texto, case insensitive)
    
2. Buscar libros disponibles
    
3. Buscar libros por autor ordenados por fecha de publicacion
    
4. Contar libros de un autor

**AutorRepository:**

1. Buscar autor por nombre exacto
    
2. Buscar autores por nacionalidad
    
3. Verificar si existe autor con nombre

Ver solucion

**LibroRepository.java:**

@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    
    // 1. Buscar por titulo (LIKE case insensitive)
    List<Libro> findByTituloContainingIgnoreCase(String titulo);
    
    // 2. Buscar disponibles
    List<Libro> findByDisponibleTrue();
    
    // Alternativa con parametro
    List<Libro> findByDisponible(boolean disponible);
    
    // 3. Buscar por autor ordenados
    List<Libro> findByAutorOrderByFechaPublicacionDesc(Autor autor);
    
    // Por ID del autor
    List<Libro> findByAutorIdOrderByFechaPublicacionAsc(Long autorId);
    
    // 4. Contar por autor
    long countByAutor(Autor autor);
    
    long countByAutorId(Long autorId);
    
    // Metodos adicionales utiles
    Optional<Libro> findByIsbn(String isbn);
    
    List<Libro> findByFechaPublicacionBetween(LocalDate inicio, LocalDate fin);
    
    List<Libro> findByAutorNombreContaining(String nombreAutor);
}

**AutorRepository.java:**

@Repository
public interface AutorRepository extends JpaRepository<Autor, Long> {
    
    // 1. Buscar por nombre exacto
    Optional<Autor> findByNombre(String nombre);
    
    // 2. Buscar por nacionalidad
    List<Autor> findByNacionalidad(String nacionalidad);
    
    // Con ordenacion
    List<Autor> findByNacionalidadOrderByNombreAsc(String nacionalidad);
    
    // 3. Verificar existencia
    boolean existsByNombre(String nombre);
    
    boolean existsByNombreIgnoreCase(String nombre);
    
    // Metodos adicionales
    List<Autor> findByFechaNacimientoAfter(LocalDate fecha);
    
    List<Autor> findByNombreStartingWith(String prefijo);
}

**Palabras clave de Query Methods:**

|Palabra|Ejemplo|SQL generado|
|---|---|---|
|`findBy`|`findByNombre`|`WHERE nombre = ?`|
|`Containing`|`findByTituloContaining`|`WHERE titulo LIKE %?%`|
|`IgnoreCase`|`findByNombreIgnoreCase`|`WHERE LOWER(nombre) = LOWER(?)`|
|`OrderBy`|`OrderByFechaDesc`|`ORDER BY fecha DESC`|
|`Between`|`findByFechaBetween`|`WHERE fecha BETWEEN ? AND ?`|
|`True/False`|`findByActivoTrue`|`WHERE activo = true`|
|`existsBy`|`existsByEmail`|`SELECT COUNT(*) > 0`|
|`countBy`|`countByCategoria`|`SELECT COUNT(*)`|

## Ejercicio 2: Consultas personalizadas con `@Query`

### Enunciado

Implementa las siguientes consultas en `PrestamoRepository` usando `@Query`:

1. Prestamos activos (sin fecha de devolucion) de un usuario
    
2. Historial de prestamos de un libro
    
3. Usuarios con mas de N prestamos activos
    
4. Libros mas prestados (top N)

Ver solucion

@Repository
public interface PrestamoRepository extends JpaRepository<Prestamo, Long> {
    
    // 1. Prestamos activos de un usuario
    @Query("SELECT p FROM Prestamo p " +
           "WHERE p.usuario.id = :usuarioId " +
           "AND p.fechaDevolucion IS NULL")
    List<Prestamo> findPrestamosActivosByUsuario(@Param("usuarioId") Long usuarioId);
    
    // Alternativa con objeto Usuario
    @Query("SELECT p FROM Prestamo p " +
           "WHERE p.usuario = :usuario " +
           "AND p.fechaDevolucion IS NULL " +
           "ORDER BY p.fechaPrestamo DESC")
    List<Prestamo> findActivosByUsuario(@Param("usuario") Usuario usuario);
    
    // 2. Historial de prestamos de un libro
    @Query("SELECT p FROM Prestamo p " +
           "WHERE p.libro.id = :libroId " +
           "ORDER BY p.fechaPrestamo DESC")
    List<Prestamo> findHistorialByLibro(@Param("libroId") Long libroId);
    
    // Con paginacion
    @Query("SELECT p FROM Prestamo p " +
           "WHERE p.libro = :libro " +
           "ORDER BY p.fechaPrestamo DESC")
    Page<Prestamo> findHistorialByLibro(@Param("libro") Libro libro, Pageable pageable);
    
    // 3. Usuarios con mas de N prestamos activos
    @Query("SELECT p.usuario FROM Prestamo p " +
           "WHERE p.fechaDevolucion IS NULL " +
           "GROUP BY p.usuario " +
           "HAVING COUNT(p) > :minPrestamos")
    List<Usuario> findUsuariosConMuchosPrestamos(@Param("minPrestamos") long minPrestamos);
    
    // Con conteo incluido
    @Query("SELECT p.usuario, COUNT(p) as total FROM Prestamo p " +
           "WHERE p.fechaDevolucion IS NULL " +
           "GROUP BY p.usuario " +
           "HAVING COUNT(p) > :min " +
           "ORDER BY total DESC")
    List<Object[]> findUsuariosConConteo(@Param("min") long min);
    
    // 4. Libros mas prestados
    @Query("SELECT p.libro, COUNT(p) as veces FROM Prestamo p " +
           "GROUP BY p.libro " +
           "ORDER BY veces DESC")
    List<Object[]> findLibrosMasPrestados(Pageable pageable);
    
    // Con proyeccion a DTO
    @Query("SELECT new com.biblioteca.dto.LibroEstadisticaDTO(" +
           "p.libro.id, p.libro.titulo, COUNT(p)) " +
           "FROM Prestamo p " +
           "GROUP BY p.libro.id, p.libro.titulo " +
           "ORDER BY COUNT(p) DESC")
    List<LibroEstadisticaDTO> findTopLibrosPrestados(Pageable pageable);
}

**Uso en servicio:**

@Service
public class EstadisticasService {
    
    @Autowired
    private PrestamoRepository prestamoRepo;
    
    public List<LibroEstadisticaDTO> getTop10LibrosMasPrestados() {
        return prestamoRepo.findTopLibrosPrestados(PageRequest.of(0, 10));
    }
    
    public List<Usuario> getUsuariosConMasDe3Prestamos() {
        return prestamoRepo.findUsuariosConMuchosPrestamos(3);
    }
}

**DTO para proyeccion:**

public class LibroEstadisticaDTO {
    private Long id;
    private String titulo;
    private Long vecesPrestado;
    
    public LibroEstadisticaDTO(Long id, String titulo, Long vecesPrestado) {
        this.id = id;
        this.titulo = titulo;
        this.vecesPrestado = vecesPrestado;
    }
    // getters...
}

## Ejercicio 3: Paginacion y ordenacion

### Enunciado

Implementa un servicio de busqueda de libros que:

1. Permita buscar por titulo, autor, disponibilidad
    
2. Soporte paginacion configurable
    
3. Permita ordenar por cualquier campo
    
4. Devuelva metadatos de paginacion (total, paginas, etc.)

Ver solucion

**DTO de busqueda:**

public class LibroBusquedaDTO {
    private String titulo;
    private String autorNombre;
    private Boolean disponible;
    private int pagina = 0;
    private int tamanio = 10;
    private String ordenarPor = "titulo";
    private String direccion = "ASC";
    
    // getters y setters...
    
    public Pageable toPageable() {
        Sort sort = direccion.equalsIgnoreCase("DESC") 
            ? Sort.by(ordenarPor).descending()
            : Sort.by(ordenarPor).ascending();
        return PageRequest.of(pagina, tamanio, sort);
    }
}

**DTO de respuesta:**

public class PaginaLibrosDTO {
    private List<LibroDTO> libros;
    private int paginaActual;
    private int totalPaginas;
    private long totalElementos;
    private int tamanio;
    private boolean primera;
    private boolean ultima;
    
    public static PaginaLibrosDTO fromPage(Page<Libro> page) {
        PaginaLibrosDTO dto = new PaginaLibrosDTO();
        dto.libros = page.getContent().stream()
            .map(LibroDTO::fromEntity)
            .collect(Collectors.toList());
        dto.paginaActual = page.getNumber();
        dto.totalPaginas = page.getTotalPages();
        dto.totalElementos = page.getTotalElements();
        dto.tamanio = page.getSize();
        dto.primera = page.isFirst();
        dto.ultima = page.isLast();
        return dto;
    }
}

**Repository con Specification:**

@Repository
public interface LibroRepository extends JpaRepository<Libro, Long>,
                                         JpaSpecificationExecutor<Libro> {
}

**Specifications:**

public class LibroSpecs {
    
    public static Specification<Libro> tituloContiene(String titulo) {
        return (root, query, cb) -> {
            if (titulo == null || titulo.isBlank()) return cb.conjunction();
            return cb.like(cb.lower(root.get("titulo")), 
                          "%" + titulo.toLowerCase() + "%");
        };
    }
    
    public static Specification<Libro> autorNombreContiene(String nombre) {
        return (root, query, cb) -> {
            if (nombre == null || nombre.isBlank()) return cb.conjunction();
            return cb.like(cb.lower(root.get("autor").get("nombre")),
                          "%" + nombre.toLowerCase() + "%");
        };
    }
    
    public static Specification<Libro> estaDisponible(Boolean disponible) {
        return (root, query, cb) -> {
            if (disponible == null) return cb.conjunction();
            return cb.equal(root.get("disponible"), disponible);
        };
    }
}

**Servicio:**

@Service
@Transactional(readOnly = true)
public class LibroBusquedaService {
    
    @Autowired
    private LibroRepository repository;
    
    public PaginaLibrosDTO buscar(LibroBusquedaDTO filtro) {
        Specification<Libro> spec = Specification
            .where(LibroSpecs.tituloContiene(filtro.getTitulo()))
            .and(LibroSpecs.autorNombreContiene(filtro.getAutorNombre()))
            .and(LibroSpecs.estaDisponible(filtro.getDisponible()));
        
        Page<Libro> pagina = repository.findAll(spec, filtro.toPageable());
        
        return PaginaLibrosDTO.fromPage(pagina);
    }
}

**Controller:**

@RestController
@RequestMapping("/api/libros")
public class LibroController {
    
    @Autowired
    private LibroBusquedaService busquedaService;
    
    @GetMapping
    public ResponseEntity<PaginaLibrosDTO> buscar(
            @RequestParam(required = false) String titulo,
            @RequestParam(required = false) String autor,
            @RequestParam(required = false) Boolean disponible,
            @RequestParam(defaultValue = "0") int pagina,
            @RequestParam(defaultValue = "10") int tamanio,
            @RequestParam(defaultValue = "titulo") String ordenarPor,
            @RequestParam(defaultValue = "ASC") String direccion) {
        
        LibroBusquedaDTO filtro = new LibroBusquedaDTO();
        filtro.setTitulo(titulo);
        filtro.setAutorNombre(autor);
        filtro.setDisponible(disponible);
        filtro.setPagina(pagina);
        filtro.setTamanio(tamanio);
        filtro.setOrdenarPor(ordenarPor);
        filtro.setDireccion(direccion);
        
        return ResponseEntity.ok(busquedaService.buscar(filtro));
    }
}

**Ejemplo de llamada:**

GET /api/libros?titulo=java&disponible=true&pagina=0&tamanio=5&ordenarPor=fechaPublicacion&direccion=DESC

**Respuesta JSON:**

{
  "libros": [],
  "paginaActual": 0,
  "totalPaginas": 3,
  "totalElementos": 15,
  "tamanio": 5,
  "primera": true,
  "ultima": false
}

## Ejercicio 4: Proyecciones

### Enunciado

Implementa diferentes tipos de proyecciones para consultas de libros:

1. Proyeccion cerrada (interface) para vista resumida
    
2. Proyeccion abierta con SpEL
    
3. Proyeccion basada en clase (DTO)
    

Ver solucion

**1. Proyeccion cerrada (Interface-based):**

// Solo getters, Spring Data implementa automaticamente
public interface LibroResumen {
    Long getId();
    String getTitulo();
    String getIsbn();
    boolean isDisponible();
    
    // Navegacion a relaciones
    AutorResumen getAutor();
    
    interface AutorResumen {
        String getNombre();
    }
}

// En el repositorio
@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    
    List<LibroResumen> findByDisponibleTrue();
    
    Optional<LibroResumen> findResumenById(Long id);
    
    // Proyeccion dinamica
    <T> List<T> findByAutorId(Long autorId, Class<T> tipo);
}

**2. Proyeccion abierta (con SpEL):**

public interface LibroConInfo {
    String getTitulo();
    
    @Value("#{target.autor.nombre}")
    String getNombreAutor();
    
    @Value("#{target.titulo + ' - ' + target.autor.nombre}")
    String getTituloCompleto();
    
    @Value("#{target.disponible ? 'Disponible' : 'Prestado'}")
    String getEstado();
    
    // Expresion con metodo default (Java 8+)
    default String getDescripcion() {
        return getTitulo() + " por " + getNombreAutor();
    }
}

**3. Proyeccion basada en clase (DTO):**

public class LibroDTO {
    private final Long id;
    private final String titulo;
    private final String autorNombre;
    private final boolean disponible;
    
    // Constructor debe coincidir con SELECT
    public LibroDTO(Long id, String titulo, String autorNombre, boolean disponible) {
        this.id = id;
        this.titulo = titulo;
        this.autorNombre = autorNombre;
        this.disponible = disponible;
    }
    
    // Solo getters (inmutable)
    public Long getId() { return id; }
    public String getTitulo() { return titulo; }
    public String getAutorNombre() { return autorNombre; }
    public boolean isDisponible() { return disponible; }
}

// En el repositorio
@Query("SELECT new com.biblioteca.dto.LibroDTO(" +
       "l.id, l.titulo, l.autor.nombre, l.disponible) " +
       "FROM Libro l " +
       "WHERE l.autor.id = :autorId")
List<LibroDTO> findDTOByAutorId(@Param("autorId") Long autorId);

**Uso de proyeccion dinamica:**

@Service
public class LibroService {
    
    @Autowired
    private LibroRepository repository;
    
    // Elegir proyeccion en runtime
    public <T> List<T> getLibrosPorAutor(Long autorId, Class<T> tipo) {
        return repository.findByAutorId(autorId, tipo);
    }
    
    // Uso
    public void ejemplo() {
        // Entidad completa
        List<Libro> completos = getLibrosPorAutor(1L, Libro.class);
        
        // Solo resumen
        List<LibroResumen> resumenes = getLibrosPorAutor(1L, LibroResumen.class);
        
        // Con SpEL
        List<LibroConInfo> conInfo = getLibrosPorAutor(1L, LibroConInfo.class);
    }
}

**Comparativa de proyecciones:**

|Tipo|Ventaja|Desventaja|
|---|---|---|
|Interface cerrada|Simple, automatica|Solo getters directos|
|Interface abierta|SpEL flexible|Mas lenta (evaluacion)|
|Clase DTO|Control total|Mas codigo, constructor|

## Ejercicio 5: Repositorio personalizado

### Enunciado

Extiende `LibroRepository` con metodos personalizados que no pueden expresarse con Query Methods:

1. Busqueda full-text en titulo y descripcion
    
2. Actualizacion masiva de disponibilidad
    
3. Estadisticas complejas con agrupaciones multiples
    

Ver solucion

**1. Interface personalizada:**

public interface LibroRepositoryCustom {
    
    List<Libro> busquedaFullText(String termino);
    
    int actualizarDisponibilidadMasiva(List<Long> ids, boolean disponible);
    
    List<EstadisticaLibroDTO> getEstadisticasComplejas();
}

**2. Implementacion:**

@Repository
public class LibroRepositoryImpl implements LibroRepositoryCustom {
    
    @PersistenceContext
    private EntityManager em;
    
    @Override
    public List<Libro> busquedaFullText(String termino) {
        if (termino == null || termino.isBlank()) {
            return Collections.emptyList();
        }
        
        String terminoLower = "%" + termino.toLowerCase() + "%";
        
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<Libro> query = cb.createQuery(Libro.class);
        Root<Libro> root = query.from(Libro.class);
        
        // Buscar en titulo, descripcion, ISBN
        Predicate enTitulo = cb.like(cb.lower(root.get("titulo")), terminoLower);
        Predicate enDescripcion = cb.like(cb.lower(root.get("descripcion")), terminoLower);
        Predicate enIsbn = cb.like(root.get("isbn"), "%" + termino + "%");
        Predicate enAutor = cb.like(cb.lower(root.get("autor").get("nombre")), terminoLower);
        
        query.select(root)
             .where(cb.or(enTitulo, enDescripcion, enIsbn, enAutor))
             .orderBy(cb.asc(root.get("titulo")));
        
        return em.createQuery(query).getResultList();
    }
    
    @Override
    @Transactional
    public int actualizarDisponibilidadMasiva(List<Long> ids, boolean disponible) {
        if (ids == null || ids.isEmpty()) {
            return 0;
        }
        
        String jpql = "UPDATE Libro l SET l.disponible = :disponible " +
                      "WHERE l.id IN :ids";
        
        return em.createQuery(jpql)
                 .setParameter("disponible", disponible)
                 .setParameter("ids", ids)
                 .executeUpdate();
    }
    
    @Override
    public List<EstadisticaLibroDTO> getEstadisticasComplejas() {
        String jpql = 
            "SELECT new com.biblioteca.dto.EstadisticaLibroDTO(" +
            "  a.nacionalidad, " +
            "  YEAR(l.fechaPublicacion), " +
            "  COUNT(l), " +
            "  SUM(CASE WHEN l.disponible = true THEN 1 ELSE 0 END), " +
            "  (SELECT COUNT(p) FROM Prestamo p WHERE p.libro = l)" +
            ") " +
            "FROM Libro l JOIN l.autor a " +
            "GROUP BY a.nacionalidad, YEAR(l.fechaPublicacion) " +
            "ORDER BY a.nacionalidad, YEAR(l.fechaPublicacion) DESC";
        
        return em.createQuery(jpql, EstadisticaLibroDTO.class)
                 .getResultList();
    }
}

**3. Repositorio que extiende ambas interfaces:**

@Repository
public interface LibroRepository extends JpaRepository<Libro, Long>,
                                         JpaSpecificationExecutor<Libro>,
                                         LibroRepositoryCustom {
    
    // Query Methods normales
    List<Libro> findByDisponibleTrue();
    
    // Los metodos de LibroRepositoryCustom se heredan automaticamente
}

**4. DTO para estadisticas:**

public class EstadisticaLibroDTO {
    private String nacionalidadAutor;
    private Integer anioPublicacion;
    private Long totalLibros;
    private Long librosDisponibles;
    private Long totalPrestamos;
    
    public EstadisticaLibroDTO(String nacionalidadAutor, Integer anioPublicacion,
                                Long totalLibros, Long librosDisponibles,
                                Long totalPrestamos) {
        this.nacionalidadAutor = nacionalidadAutor;
        this.anioPublicacion = anioPublicacion;
        this.totalLibros = totalLibros;
        this.librosDisponibles = librosDisponibles;
        this.totalPrestamos = totalPrestamos;
    }
    // getters...
}

**Uso en servicio:**

@Service
public class LibroService {
    
    @Autowired
    private LibroRepository repository;
    
    public List<Libro> buscar(String termino) {
        // Usa el metodo custom
        return repository.busquedaFullText(termino);
    }
    
    @Transactional
    public int marcarComoNoDisponibles(List<Long> ids) {
        return repository.actualizarDisponibilidadMasiva(ids, false);
    }
}

**Importante:**

- La clase debe llamarse `[NombreRepositorio]Impl`
    
- Spring Data la detecta automaticamente
    
- Puede inyectar EntityManager u otros beans
    

## Reto adicional

Implementa un sistema de auditoria automatica con Spring Data JPA:

1. Usa `@CreatedDate`, `@LastModifiedDate`, `@CreatedBy`, `@LastModifiedBy`
    
2. Configura el `AuditorAware` para obtener el usuario actual
    
3. Crea una clase base auditada que otras entidades extiendan


Consejo

Necesitas habilitar auditoria con @EnableJpaAuditing y configurar un bean `AuditorAware<String>`.