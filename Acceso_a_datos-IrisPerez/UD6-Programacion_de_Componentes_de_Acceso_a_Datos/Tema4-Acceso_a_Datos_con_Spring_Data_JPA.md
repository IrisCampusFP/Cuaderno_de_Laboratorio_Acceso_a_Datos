## Objetivos

- Comprender el patron Repository de Spring Data
    
- Crear repositorios con JpaRepository
    
- Utilizar Query Methods y consultas personalizadas
    
- Implementar paginacion y ordenacion

## 1. Introduccion a Spring Data JPA

**Spring Data JPA** simplifica el desarrollo de la capa de acceso a datos eliminando codigo repetitivo.

### 1.1 Caracteristicas

|Caracteristica|Beneficio|
|---|---|
|**Interfaces Repository**|Sin implementacion manual|
|**Query Methods**|Consultas derivadas del nombre del metodo|
|**Paginacion**|Soporte integrado|
|**Auditing**|Campos de auditoria automaticos|
|**Specifications**|Consultas dinamicas|

### 1.2 Configuracion

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

```properties
# application.properties
spring.datasource.url=jdbc:postgresql://localhost:5432/biblioteca
spring.datasource.username=biblioteca
spring.datasource.password=biblioteca123
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

## 2. Jerarquia de Interfaces Repository

```
┌─────────────────────────────────────────────────────────────┐
│                     Repository<T, ID>                        │
│                     (interface marcadora)                    │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │
┌─────────────────────────────────────────────────────────────┐
│                   CrudRepository<T, ID>                      │
│   save(), findById(), findAll(), delete(), count()...        │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │
┌─────────────────────────────────────────────────────────────┐
│              ListCrudRepository<T, ID>                       │
│   findAll() -> List<T>                                       │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │
┌─────────────────────────────────────────────────────────────┐
│               JpaRepository<T, ID>                           │
│   flush(), saveAndFlush(), findAll(Sort), findAll(Pageable)  │
└─────────────────────────────────────────────────────────────┘
```

## 3. Crear un Repository

### 3.1 Entidad JPA

```java
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
    
    @Column(unique = true, length = 20)
    private String isbn;
    
    private Integer anioPublicacion;
    
    @Enumerated(EnumType.STRING)
    private Genero genero;
    
    private boolean disponible = true;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal precio;
}

public enum Genero {
    FICCION, NO_FICCION, CIENCIA, HISTORIA, TECNOLOGIA
}
```

### 3.2 Interface Repository

```java
@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    // Spring Data genera la implementacion automaticamente
}
```

### 3.3 Metodos Heredados de JpaRepository

```java
// CRUD basico
Libro libro = repository.save(new Libro());          // INSERT o UPDATE
Optional<Libro> opt = repository.findById(1L);       // SELECT por ID
List<Libro> todos = repository.findAll();            // SELECT *
repository.deleteById(1L);                           // DELETE
long total = repository.count();                     // COUNT
boolean existe = repository.existsById(1L);          // EXISTS

// Operaciones en lote
List<Libro> guardados = repository.saveAll(listaLibros);
repository.deleteAll(listaLibros);
repository.deleteAllById(listaIds);

// Flush
repository.flush();                                  // Forzar sincronizacion
Libro l = repository.saveAndFlush(libro);            // Guardar y flush
```

## 4. Query Methods (Metodos de Consulta)

Spring Data genera consultas automaticamente basandose en el nombre del metodo.

### 4.1 Sintaxis

```java
findBy + Campo + Operador + [And/Or + Campo + Operador]
```

### 4.2 Ejemplos de Query Methods

```java
public interface LibroRepository extends JpaRepository<Libro, Long> {
    
    // Busqueda exacta
    List<Libro> findByAutor(String autor);
    Optional<Libro> findByIsbn(String isbn);
    
    // Busqueda con LIKE
    List<Libro> findByTituloContaining(String texto);           // %texto%
    List<Libro> findByTituloStartingWith(String prefijo);       // prefijo%
    List<Libro> findByTituloEndingWith(String sufijo);          // %sufijo
    List<Libro> findByAutorContainingIgnoreCase(String autor);  // Case insensitive
    
    // Busqueda con comparadores
    List<Libro> findByAnioPublicacionGreaterThan(Integer anio);      // >
    List<Libro> findByAnioPublicacionLessThanEqual(Integer anio);    // <=
    List<Libro> findByAnioPublicacionBetween(Integer desde, Integer hasta);
    
    // Busqueda por booleano
    List<Libro> findByDisponibleTrue();
    List<Libro> findByDisponibleFalse();
    
    // Busqueda con NULL
    List<Libro> findByAutorIsNull();
    List<Libro> findByAutorIsNotNull();
    
    // Busqueda en coleccion
    List<Libro> findByGeneroIn(Collection<Genero> generos);
    List<Libro> findByGeneroNotIn(Collection<Genero> generos);
    
    // Combinaciones AND/OR
    List<Libro> findByAutorAndDisponibleTrue(String autor);
    List<Libro> findByGeneroOrAnioPublicacionGreaterThan(Genero g, Integer anio);
    
    // Ordenacion
    List<Libro> findByGeneroOrderByTituloAsc(Genero genero);
    List<Libro> findByDisponibleTrueOrderByAnioPublicacionDesc();
    
    // Limitar resultados
    List<Libro> findTop5ByOrderByAnioPublicacionDesc();
    Libro findFirstByAutorOrderByAnioPublicacionDesc(String autor);
    
    // Contar y existir
    long countByGenero(Genero genero);
    boolean existsByIsbn(String isbn);
    
    // Eliminar
    void deleteByIsbn(String isbn);
    long deleteByDisponibleFalse();
}

```
## 5. Consultas Personalizadas con `@Query`

### 5.1 JPQL (Java Persistence Query Language)

```java
public interface LibroRepository extends JpaRepository<Libro, Long> {
    
    // Consulta JPQL simple
    @Query("SELECT l FROM Libro l WHERE l.disponible = true")
    List<Libro> buscarDisponibles();
    
    // Con parametros posicionales
    @Query("SELECT l FROM Libro l WHERE l.autor = ?1 AND l.anioPublicacion > ?2")
    List<Libro> buscarPorAutorYAnio(String autor, Integer anio);
    
    // Con parametros nombrados
    @Query("SELECT l FROM Libro l WHERE l.genero = :genero AND l.precio < :precioMax")
    List<Libro> buscarPorGeneroYPrecio(@Param("genero") Genero genero, 
                                        @Param("precioMax") BigDecimal precio);
    
    // LIKE con parametro
    @Query("SELECT l FROM Libro l WHERE LOWER(l.titulo) LIKE LOWER(CONCAT('%', :texto, '%'))")
    List<Libro> buscarPorTitulo(@Param("texto") String texto);
    
    // Proyecciones (campos especificos)
    @Query("SELECT l.titulo, l.autor FROM Libro l WHERE l.genero = :genero")
    List<Object[]> buscarTitulosYAutores(@Param("genero") Genero genero);
    
    // Funciones de agregacion
    @Query("SELECT COUNT(l) FROM Libro l WHERE l.anioPublicacion = :anio")
    long contarPorAnio(@Param("anio") Integer anio);
    
    @Query("SELECT AVG(l.precio) FROM Libro l WHERE l.genero = :genero")
    BigDecimal precioPromedioPorGenero(@Param("genero") Genero genero);
}

```
### 5.2 Consultas Nativas SQL

```java
public interface LibroRepository extends JpaRepository<Libro, Long> {
    
    // SQL nativo
    @Query(value = "SELECT * FROM libros WHERE stock > 0 ORDER BY precio", 
           nativeQuery = true)
    List<Libro> buscarConStock();
    
    // SQL nativo con parametros
    @Query(value = "SELECT * FROM libros WHERE EXTRACT(YEAR FROM fecha_publicacion) = :anio",
           nativeQuery = true)
    List<Libro> buscarPorAnioPublicacion(@Param("anio") int anio);
    
    // Llamar a funcion de base de datos
    @Query(value = "SELECT calcular_multa(:prestamo_id)", nativeQuery = true)
    BigDecimal calcularMulta(@Param("prestamo_id") Long prestamoId);
}
```

### 5.3 Operaciones de Modificacion

```java
public interface LibroRepository extends JpaRepository<Libro, Long> {
    
    // UPDATE masivo
    @Modifying
    @Query("UPDATE Libro l SET l.disponible = :disponible WHERE l.genero = :genero")
    int actualizarDisponibilidadPorGenero(@Param("disponible") boolean disponible,
                                          @Param("genero") Genero genero);
    
    // DELETE masivo
    @Modifying
    @Query("DELETE FROM Libro l WHERE l.anioPublicacion < :anio")
    int eliminarAntiguos(@Param("anio") Integer anio);
    
    // UPDATE con incremento
    @Modifying
    @Query("UPDATE Libro l SET l.precio = l.precio * :factor")
    int actualizarPrecios(@Param("factor") BigDecimal factor);
}

```
**Importante:** Los metodos con `@Modifying` deben ejecutarse dentro de una transaccion.

## 6. Paginacion y Ordenacion

### 6.1 Pageable y Page

```java
// En el Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    Page<Libro> findByGenero(Genero genero, Pageable pageable);
    Page<Libro> findByDisponibleTrue(Pageable pageable);
}

// En el Service
@Service
public class LibroService {
    private final LibroRepository repository;
    
    public Page<Libro> listarPaginado(int pagina, int tamanio) {
        Pageable pageable = PageRequest.of(pagina, tamanio);
        return repository.findAll(pageable);
    }
    
    public Page<Libro> listarOrdenado(int pagina, int tamanio, String campoOrden) {
        Pageable pageable = PageRequest.of(pagina, tamanio, Sort.by(campoOrden).ascending());
        return repository.findAll(pageable);
    }
    
    public Page<Libro> listarPorGenero(Genero genero, int pagina, int tamanio) {
        Pageable pageable = PageRequest.of(pagina, tamanio, Sort.by("titulo"));
        return repository.findByGenero(genero, pageable);
    }
}
```

### 6.2 Usar el Objeto Page

```java
// En el Controller
@RestController
@RequestMapping("/api/libros")
public class LibroController {
    
    @GetMapping
    public Page<Libro> listar(
            @RequestParam(defaultValue = "0") int pagina,
            @RequestParam(defaultValue = "10") int tamanio,
            @RequestParam(defaultValue = "titulo") String ordenar) {
        
        return service.listarOrdenado(pagina, tamanio, ordenar);
    }
}

// Propiedades de Page<T>
Page<Libro> pagina = repository.findAll(pageable);

List<Libro> contenido = pagina.getContent();      // Elementos de la pagina
int totalPaginas = pagina.getTotalPages();        // Total de paginas
long totalElementos = pagina.getTotalElements();  // Total de elementos
int numero = pagina.getNumber();                  // Numero de pagina actual
int tamanio = pagina.getSize();                   // Tamanio de pagina
boolean primera = pagina.isFirst();               // Es primera pagina?
boolean ultima = pagina.isLast();                 // Es ultima pagina?
boolean haySiguiente = pagina.hasNext();          // Hay siguiente?
```

## 7. Proyecciones (DTOs)

### 7.1 Interface-based Projections

```java
// Definir proyeccion como interface
public interface LibroResumen {
    String getTitulo();
    String getAutor();
    Integer getAnioPublicacion();
}

// Usar en Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    List<LibroResumen> findByGenero(Genero genero);
    List<LibroResumen> findByDisponibleTrue();
}
```

### 7.2 Class-based Projections (DTO)

```java
// DTO
@Data
@AllArgsConstructor
public class LibroDTO {
    private String titulo;
    private String autor;
    private BigDecimal precio;
}

// Usar en Repository con @Query
public interface LibroRepository extends JpaRepository<Libro, Long> {
    
    @Query("SELECT new com.biblioteca.dto.LibroDTO(l.titulo, l.autor, l.precio) " +
           "FROM Libro l WHERE l.disponible = true")
    List<LibroDTO> buscarDisponiblesDTO();
}

```
## 8. Ejemplo Completo

```java
// Entity
@Entity
@Data
public class Libro {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String titulo;
    private String autor;
    private String isbn;
    @Enumerated(EnumType.STRING)
    private Genero genero;
    private boolean disponible = true;
}

// Repository
@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    List<Libro> findByDisponibleTrue();
    List<Libro> findByAutorContainingIgnoreCase(String autor);
    Page<Libro> findByGenero(Genero genero, Pageable pageable);
    
    @Query("SELECT l FROM Libro l WHERE l.genero = :genero AND l.disponible = true")
    List<Libro> buscarDisponiblesPorGenero(@Param("genero") Genero genero);
}

// Service
@Service
@Transactional(readOnly = true)
public class LibroService {
    private final LibroRepository repository;
    
    public LibroService(LibroRepository repository) {
        this.repository = repository;
    }
    
    public List<Libro> listarDisponibles() {
        return repository.findByDisponibleTrue();
    }
    
    public Page<Libro> listarPorGenero(Genero genero, int pagina, int tamanio) {
        return repository.findByGenero(genero, PageRequest.of(pagina, tamanio));
    }
    
    @Transactional
    public Libro guardar(Libro libro) {
        return repository.save(libro);
    }
}

// Controller
@RestController
@RequestMapping("/api/libros")
public class LibroController {
    private final LibroService service;
    
    @GetMapping("/disponibles")
    public List<Libro> disponibles() {
        return service.listarDisponibles();
    }
    
    @GetMapping("/genero/{genero}")
    public Page<Libro> porGenero(
            @PathVariable Genero genero,
            @RequestParam(defaultValue = "0") int pagina,
            @RequestParam(defaultValue = "10") int tamanio) {
        return service.listarPorGenero(genero, pagina, tamanio);
    }
}
```

## 9. Resumen

### Query Methods - Palabras Clave

|Palabra Clave|Ejemplo|SQL Equivalente|
|---|---|---|
|`And`|`findByAutorAndGenero`|`WHERE autor = ? AND genero = ?`|
|`Or`|`findByAutorOrGenero`|`WHERE autor = ? OR genero = ?`|
|`Is, Equals`|`findByTitulo`, `findByTituloIs`|`WHERE titulo = ?`|
|`Between`|`findByPrecioBetween`|`WHERE precio BETWEEN ? AND ?`|
|`LessThan`|`findByPrecioLessThan`|`WHERE precio < ?`|
|`GreaterThanEqual`|`findByAnioGreaterThanEqual`|`WHERE anio >= ?`|
|`Like`|`findByTituloLike`|`WHERE titulo LIKE ?`|
|`Containing`|`findByTituloContaining`|`WHERE titulo LIKE %?%`|
|`OrderBy`|`findByGeneroOrderByTitulo`|`ORDER BY titulo`|
|`Not`|`findByGeneroNot`|`WHERE genero <> ?`|
|`In`|`findByGeneroIn`|`WHERE genero IN (?)`|
|`True/False`|`findByDisponibleTrue`|`WHERE disponible = true`|