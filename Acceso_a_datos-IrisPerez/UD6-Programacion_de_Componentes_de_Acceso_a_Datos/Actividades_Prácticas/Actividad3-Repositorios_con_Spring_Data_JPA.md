## Objetivos

- Crear Query Methods personalizados
    
- Implementar consultas con `@Query`
    
- Usar paginacion y ordenacion
    

## Ejercicio 1: Query Methods

### Enunciado

Amplia el `LibroRepository` con los siguientes Query Methods:

1. Buscar libros por autor (exacto)
    
2. Buscar libros cuyo titulo contenga un texto (case insensitive)
    
3. Buscar libros publicados despues de un anio
    
4. Buscar libros por autor y disponibilidad
    
5. Contar libros por autor
    
6. Verificar si existe un libro por ISBN
    

Ver solucion

@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    
    // 1. Buscar por autor exacto
    List<Libro> findByAutor(String autor);
    
    // 2. Buscar por titulo (contiene, case insensitive)
    List<Libro> findByTituloContainingIgnoreCase(String titulo);
    
    // 3. Buscar publicados despues de un anio
    List<Libro> findByAnioPublicacionGreaterThan(Integer anio);
    
    // 4. Buscar por autor y disponibilidad
    List<Libro> findByAutorAndDisponible(String autor, boolean disponible);
    
    // 5. Contar por autor
    long countByAutor(String autor);
    
    // 6. Verificar existencia por ISBN
    boolean existsByIsbn(String isbn);
    
    // Extras utiles
    List<Libro> findByDisponibleTrue();
    List<Libro> findByAnioPublicacionBetween(Integer desde, Integer hasta);
    List<Libro> findTop5ByOrderByAnioPublicacionDesc();
}

## Ejercicio 2: Consultas JPQL con `@Query`

### Enunciado

Implementa las siguientes consultas usando `@Query`:

1. Buscar libros disponibles ordenados por titulo
    
2. Buscar libros de un autor con precio menor a un valor
    
3. Obtener el promedio de paginas por genero
    
4. Actualizar disponibilidad de todos los libros de un autor
    

Ver solucion

@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    
    // 1. Libros disponibles ordenados
    @Query("SELECT l FROM Libro l WHERE l.disponible = true ORDER BY l.titulo")
    List<Libro> buscarDisponiblesOrdenados();
    
    // 2. Por autor y precio maximo
    @Query("SELECT l FROM Libro l WHERE l.autor = :autor AND l.precio < :precioMax")
    List<Libro> buscarPorAutorYPrecio(@Param("autor") String autor, 
                                       @Param("precioMax") BigDecimal precio);
    
    // 3. Promedio de paginas por genero
    @Query("SELECT l.genero, AVG(l.paginas) FROM Libro l GROUP BY l.genero")
    List<Object[]> promediosPaginasPorGenero();
    
    // 4. Actualizar disponibilidad (requiere @Modifying)
    @Modifying
    @Query("UPDATE Libro l SET l.disponible = :disponible WHERE l.autor = :autor")
    int actualizarDisponibilidadPorAutor(@Param("autor") String autor, 
                                          @Param("disponible") boolean disponible);
    
    // Consulta con LIKE
    @Query("SELECT l FROM Libro l WHERE LOWER(l.titulo) LIKE LOWER(CONCAT('%', :texto, '%'))")
    List<Libro> buscarPorTextoEnTitulo(@Param("texto") String texto);
}

## Ejercicio 3: Paginacion y Ordenacion

### Enunciado

Implementa un servicio que permita:

1. Listar libros paginados (pagina, tamanio)
    
2. Listar libros ordenados por campo dinamico
    
3. Listar libros de un genero con paginacion
    

Ver solucion

**Repository:**

@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    Page<Libro> findByGenero(String genero, Pageable pageable);
}

**Service:**

@Service
@Transactional(readOnly = true)
public class LibroService {
    
    private final LibroRepository repository;
    
    public LibroService(LibroRepository repository) {
        this.repository = repository;
    }
    
    // 1. Listar paginados
    public Page<Libro> listarPaginado(int pagina, int tamanio) {
        Pageable pageable = PageRequest.of(pagina, tamanio);
        return repository.findAll(pageable);
    }
    
    // 2. Listar ordenados
    public Page<Libro> listarOrdenado(int pagina, int tamanio, 
                                       String campo, String direccion) {
        Sort sort = direccion.equalsIgnoreCase("desc") 
            ? Sort.by(campo).descending() 
            : Sort.by(campo).ascending();
        Pageable pageable = PageRequest.of(pagina, tamanio, sort);
        return repository.findAll(pageable);
    }
    
    // 3. Por genero paginado
    public Page<Libro> listarPorGenero(String genero, int pagina, int tamanio) {
        Pageable pageable = PageRequest.of(pagina, tamanio, Sort.by("titulo"));
        return repository.findByGenero(genero, pageable);
    }
}

**Controller:**

@RestController
@RequestMapping("/api/libros")
public class LibroController {
    
    @GetMapping
    public Page<Libro> listar(
            @RequestParam(defaultValue = "0") int pagina,
            @RequestParam(defaultValue = "10") int tamanio,
            @RequestParam(defaultValue = "titulo") String ordenar,
            @RequestParam(defaultValue = "asc") String direccion) {
        return service.listarOrdenado(pagina, tamanio, ordenar, direccion);
    }
}

## Ejercicio 4: Proyecciones DTO

### Enunciado

Crea un DTO `LibroResumenDTO` que solo contenga id, titulo y autor, y una consulta que devuelva esta proyeccion.

Ver solucion

**DTO:**

@Data
@AllArgsConstructor
public class LibroResumenDTO {
    private Long id;
    private String titulo;
    private String autor;
}

**Repository:**

@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    
    @Query("SELECT new com.biblioteca.dto.LibroResumenDTO(l.id, l.titulo, l.autor) " +
           "FROM Libro l WHERE l.disponible = true")
    List<LibroResumenDTO> findResumenDisponibles();
    
    // Alternativa: Interface-based projection
    // List<LibroResumen> findByDisponibleTrue();
}

**Interface projection (alternativa):**

public interface LibroResumen {
    Long getId();
    String getTitulo();
    String getAutor();
}