## Objetivos

- Integrar componentes de acceso a ficheros (CE 6c)
    
- Integrar componentes con conectores JDBC (CE 6d)
    
- Integrar componentes ORM con Spring Data JPA (CE 6e)
    
- Integrar componentes para bases de datos objeto-relacionales (CE 6f)
    
- Integrar componentes para bases de datos documentales (CE 6g)
    
- Construir una aplicacion integrando todos los componentes (CE 6i)

## 1. Componentes de Acceso a Ficheros (CE 6c)

### 1.1 Servicio de Ficheros

```java
@Service
public class FicheroService {
    
    @Value("${app.storage.path}")
    private String storagePath;
    
    public String guardarFichero(MultipartFile file) throws IOException {
        Path directorio = Paths.get(storagePath);
        if (!Files.exists(directorio)) {
            Files.createDirectories(directorio);
        }
        
        String nombreUnico = UUID.randomUUID() + "_" + file.getOriginalFilename();
        Path destino = directorio.resolve(nombreUnico);
        Files.copy(file.getInputStream(), destino, StandardCopyOption.REPLACE_EXISTING);
        
        return nombreUnico;
    }
    
    public Resource cargarFichero(String nombre) throws MalformedURLException {
        Path path = Paths.get(storagePath).resolve(nombre);
        Resource resource = new UrlResource(path.toUri());
        
        if (resource.exists() && resource.isReadable()) {
            return resource;
        }
        throw new RuntimeException("No se puede leer el fichero: " + nombre);
    }
    
    public void eliminarFichero(String nombre) throws IOException {
        Path path = Paths.get(storagePath).resolve(nombre);
        Files.deleteIfExists(path);
    }
}
```

### 1.2 Componente de Configuracion JSON/XML

```java
@Service
public class ConfiguracionService {
    
    private final ObjectMapper objectMapper;
    
    @Value("${app.config.path}")
    private String configPath;
    
    public <T> T cargarConfiguracion(String archivo, Class<T> tipo) throws IOException {
        Path path = Paths.get(configPath).resolve(archivo);
        return objectMapper.readValue(path.toFile(), tipo);
    }
    
    public void guardarConfiguracion(String archivo, Object configuracion) throws IOException {
        Path path = Paths.get(configPath).resolve(archivo);
        objectMapper.writerWithDefaultPrettyPrinter()
            .writeValue(path.toFile(), configuracion);
    }
    
    // Para XML
    public <T> T cargarXML(String archivo, Class<T> tipo) throws Exception {
        JAXBContext context = JAXBContext.newInstance(tipo);
        Unmarshaller unmarshaller = context.createUnmarshaller();
        Path path = Paths.get(configPath).resolve(archivo);
        return (T) unmarshaller.unmarshal(path.toFile());
    }
}
```

## 2. Componentes con JDBC (CE 6d)

### 2.1 JdbcTemplate Component

```java
@Repository
public class LibroJdbcRepository {
    
    private final JdbcTemplate jdbcTemplate;
    
    public LibroJdbcRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    
    public List<Libro> findAll() {
        String sql = "SELECT id, titulo, autor, isbn FROM libros";
        return jdbcTemplate.query(sql, this::mapRowToLibro);
    }
    
    public Optional<Libro> findById(Long id) {
        String sql = "SELECT id, titulo, autor, isbn FROM libros WHERE id = ?";
        List<Libro> results = jdbcTemplate.query(sql, this::mapRowToLibro, id);
        return results.isEmpty() ? Optional.empty() : Optional.of(results.get(0));
    }
    
    public Libro save(Libro libro) {
        String sql = "INSERT INTO libros (titulo, autor, isbn) VALUES (?, ?, ?)";
        KeyHolder keyHolder = new GeneratedKeyHolder();
        
        jdbcTemplate.update(connection -> {
            PreparedStatement ps = connection.prepareStatement(sql, new String[]{"id"});
            ps.setString(1, libro.getTitulo());
            ps.setString(2, libro.getAutor());
            ps.setString(3, libro.getIsbn());
            return ps;
        }, keyHolder);
        
        libro.setId(keyHolder.getKey().longValue());
        return libro;
    }
    
    private Libro mapRowToLibro(ResultSet rs, int rowNum) throws SQLException {
        return new Libro(
            rs.getLong("id"),
            rs.getString("titulo"),
            rs.getString("autor"),
            rs.getString("isbn")
        );
    }
}

```
### 2.2 NamedParameterJdbcTemplate

```java
@Repository
public class LibroNamedJdbcRepository {
    
    private final NamedParameterJdbcTemplate namedJdbc;
    
    public List<Libro> buscarPorAutorYAnio(String autor, Integer anio) {
        String sql = "SELECT * FROM libros WHERE autor = :autor AND anio >= :anio";
        
        MapSqlParameterSource params = new MapSqlParameterSource()
            .addValue("autor", autor)
            .addValue("anio", anio);
        
        return namedJdbc.query(sql, params, new BeanPropertyRowMapper<>(Libro.class));
    }
    
    public int actualizarPrecio(BigDecimal factor, String genero) {
        String sql = "UPDATE libros SET precio = precio * :factor WHERE genero = :genero";
        
        MapSqlParameterSource params = new MapSqlParameterSource()
            .addValue("factor", factor)
            .addValue("genero", genero);
        
        return namedJdbc.update(sql, params);
    }
}

```
## 3. Componentes ORM con Spring Data JPA (CE 6e)

### 3.1 Repository con Consultas Avanzadas

```java
@Repository
public interface LibroRepository extends JpaRepository<Libro, Long>, JpaSpecificationExecutor<Libro> {
    
    // Query Methods
    List<Libro> findByAutorContainingIgnoreCase(String autor);
    
    // JPQL
    @Query("SELECT l FROM Libro l JOIN FETCH l.categorias WHERE l.disponible = true")
    List<Libro> findDisponiblesConCategorias();
    
    // Nativo
    @Query(value = "SELECT * FROM libros WHERE EXTRACT(YEAR FROM fecha_creacion) = :anio", 
           nativeQuery = true)
    List<Libro> findByAnioCreacion(@Param("anio") int anio);
    
    // Proyeccion
    @Query("SELECT new com.biblioteca.dto.LibroResumenDTO(l.id, l.titulo, l.autor) FROM Libro l")
    List<LibroResumenDTO> findAllResumen();
}

```
### 3.2 Servicio con Transacciones

```java
@Service
@Transactional(readOnly = true)
public class BibliotecaService {
    
    private final LibroRepository libroRepository;
    private final PrestamoRepository prestamoRepository;
    private final UsuarioRepository usuarioRepository;
    
    @Transactional
    public Prestamo realizarPrestamo(Long usuarioId, Long libroId) {
        Usuario usuario = usuarioRepository.findById(usuarioId)
            .orElseThrow(() -> new EntityNotFoundException("Usuario no encontrado"));
        
        Libro libro = libroRepository.findById(libroId)
            .orElseThrow(() -> new EntityNotFoundException("Libro no encontrado"));
        
        if (!libro.isDisponible()) {
            throw new BusinessException("El libro no esta disponible");
        }
        
        libro.setDisponible(false);
        libroRepository.save(libro);
        
        Prestamo prestamo = new Prestamo();
        prestamo.setUsuario(usuario);
        prestamo.setLibro(libro);
        prestamo.setFechaPrestamo(LocalDate.now());
        prestamo.setFechaDevolucion(LocalDate.now().plusDays(15));
        
        return prestamoRepository.save(prestamo);
    }
    
    @Transactional
    public void devolverLibro(Long prestamoId) {
        Prestamo prestamo = prestamoRepository.findById(prestamoId)
            .orElseThrow(() -> new EntityNotFoundException("Prestamo no encontrado"));
        
        Libro libro = prestamo.getLibro();
        libro.setDisponible(true);
        libroRepository.save(libro);
        
        prestamo.setFechaDevolucionReal(LocalDate.now());
        prestamoRepository.save(prestamo);
    }
}
```

## 4. Componentes para BD Objeto-Relacionales (CE 6f)

### 4.1 Tipos Embebidos con JPA

```java
@Embeddable
@Data
public class Direccion {
    private String calle;
    private String ciudad;
    private String codigoPostal;
    private String pais;
}

@Entity
@Data
public class Editorial {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    @Embedded
    private Direccion direccion;
    
    @ElementCollection
    @CollectionTable(name = "editorial_telefonos")
    private List<String> telefonos = new ArrayList<>();
}

```
### 4.2 Herencia con JPA

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tipo")
@Data
public abstract class Recurso {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String titulo;
    private LocalDate fechaAdquisicion;
    private boolean disponible = true;
}

@Entity
@DiscriminatorValue("LIBRO")
@Data
public class Libro extends Recurso {
    private String isbn;
    private String autor;
    private Integer paginas;
}

@Entity
@DiscriminatorValue("REVISTA")
@Data
public class Revista extends Recurso {
    private String issn;
    private Integer numero;
    private LocalDate fechaPublicacion;
}

@Entity
@DiscriminatorValue("DVD")
@Data
public class DVD extends Recurso {
    private Integer duracion;
    private String director;
    private String formato;
}
```

### 4.3 Repository Polimorfico

```java
@Repository
public interface RecursoRepository extends JpaRepository<Recurso, Long> {
    
    // Consulta polimorfica - devuelve todos los tipos
    List<Recurso> findByDisponibleTrue();
    
    // Filtrar por tipo
    @Query("SELECT r FROM Recurso r WHERE TYPE(r) = :tipo")
    List<Recurso> findByTipo(@Param("tipo") Class<?> tipo);
    
    // Solo libros
    @Query("SELECT l FROM Libro l WHERE l.autor = :autor")
    List<Libro> findLibrosByAutor(@Param("autor") String autor);
}
```

## 5. Componentes para BD Documentales (CE 6g)

### 5.1 Spring Data MongoDB

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/biblioteca
```

```java
@Document(collection = "libros")
@Data
public class LibroDoc {
    @Id
    private String id;
    private String titulo;
    private String autor;
    private List<String> generos;
    private Map<String, Object> metadatos;
    private LocalDateTime fechaCreacion;
}

@Repository
public interface LibroDocRepository extends MongoRepository<LibroDoc, String> {
    List<LibroDoc> findByAutorContaining(String autor);
    List<LibroDoc> findByGenerosContaining(String genero);
    
    @Query("{ 'metadatos.paginas': { $gt: ?0 } }")
    List<LibroDoc> findByPaginasMayorQue(int paginas);
}

```

## 6. Integracion de Componentes (CE 6i)

### 6.1 Servicio que Integra Multiple Fuentes

```java
@Service
public class CatalogoService {
    
    private final LibroRepository libroRepository;       // JPA
    private final LibroDocRepository libroDocRepository; // MongoDB
    private final FicheroService ficheroService;         // Ficheros
    
    @Transactional
    public LibroDTO crearLibroCompleto(LibroCreateDTO dto, MultipartFile portada) {
        // 1. Guardar portada en sistema de ficheros
        String nombrePortada = null;
        if (portada != null && !portada.isEmpty()) {
            nombrePortada = ficheroService.guardarFichero(portada);
        }
        
        // 2. Guardar libro en BD relacional (JPA)
        Libro libro = new Libro();
        libro.setTitulo(dto.getTitulo());
        libro.setAutor(dto.getAutor());
        libro.setIsbn(dto.getIsbn());
        libro.setPortadaUrl(nombrePortada);
        libro = libroRepository.save(libro);
        
        // 3. Guardar metadatos en MongoDB
        LibroDoc doc = new LibroDoc();
        doc.setId(libro.getId().toString());
        doc.setTitulo(dto.getTitulo());
        doc.setMetadatos(dto.getMetadatos());
        doc.setFechaCreacion(LocalDateTime.now());
        libroDocRepository.save(doc);
        
        return mapToDTO(libro);
    }
    
    public LibroDetalleDTO obtenerLibroConMetadatos(Long id) {
        Libro libro = libroRepository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("Libro no encontrado"));
        
        LibroDoc doc = libroDocRepository.findById(id.toString())
            .orElse(null);
        
        return new LibroDetalleDTO(libro, doc);
    }
}
```

### 6.2 Arquitectura de la Aplicacion Integrada

```
┌─────────────────────────────────────────────────────────────────┐
│                     CONTROLLERS (REST API)                       │
├─────────────────────────────────────────────────────────────────┤
│                         SERVICES                                 │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│   │ LibroService │  │ UsuarioSvc   │  │ CatalogoSvc  │          │
│   └──────────────┘  └──────────────┘  └──────────────┘          │
├─────────────────────────────────────────────────────────────────┤
│                       REPOSITORIES                               │
│   ┌────────────┐ ┌──────────────┐ ┌────────────┐ ┌────────────┐ │
│   │ JPA Repo   │ │ JDBC Repo    │ │ Mongo Repo │ │ File Svc   │ │
│   └────────────┘ └──────────────┘ └────────────┘ └────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│                      DATA SOURCES                                │
│   ┌────────────┐ ┌──────────────┐ ┌────────────┐ ┌────────────┐ │
│   │ PostgreSQL │ │    Oracle    │ │  MongoDB   │ │ FileSystem │ │
│   └────────────┘ └──────────────┘ └────────────┘ └────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## 7. Resumen

### Componentes por Criterio de Evaluacion

|CE|Componente|Tecnologia|
|---|---|---|
|6c|FicheroService|Java NIO, Spring Resource|
|6d|JdbcRepository|JdbcTemplate, NamedParameterJdbcTemplate|
|6e|JpaRepository|Spring Data JPA, Hibernate|
|6f|RecursoRepository|JPA Inheritance, Embedded|
|6g|MongoRepository|Spring Data MongoDB|
|6i|CatalogoService|Integracion de todos los anteriores|