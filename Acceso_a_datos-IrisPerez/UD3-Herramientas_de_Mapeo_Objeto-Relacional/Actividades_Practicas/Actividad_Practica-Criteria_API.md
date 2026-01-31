## Objetivos

- Construir consultas dinamicas con Criteria API
    
- Usar predicados y combinaciones logicas
    
- Implementar Specifications de Spring Data
    
- Aplicar joins y fetch en consultas

## Contexto

Trabajaremos con un sistema de e-commerce con las entidades:

- `Producto`: id, nombre, precio, stock, activo
    
- `Categoria`: id, nombre
    
- Relacion: Producto ManyToOne Categoria

## Ejercicio 1: Consulta basica con Criteria

### Enunciado

Implementa un metodo que busque todos los productos con precio mayor a un valor dado, ordenados por precio descendente.

Ver solucion

public List<Producto> findByPrecioMayorQue(BigDecimal precioMinimo) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Predicado: precio > precioMinimo
    Predicate precioPred = cb.greaterThan(root.get("precio"), precioMinimo);
    
    // Construir consulta
    query.select(root)
         .where(precioPred)
         .orderBy(cb.desc(root.get("precio")));
    
    return entityManager.createQuery(query).getResultList();
}

**Pasos clave:**

1. Obtener `CriteriaBuilder` del EntityManager
    
2. Crear `CriteriaQuery` con el tipo de resultado
    
3. Definir `Root` (entidad FROM)
    
4. Crear predicados con el builder
    
5. Aplicar select, where y orderBy


## Ejercicio 2: Multiples predicados

### Enunciado

Crea un metodo que busque productos que cumplan:

- Nombre contenga un texto (case insensitive)
    
- Precio entre un minimo y maximo
    
- Stock mayor a 0
    
- Activo = true


Todos los parametros son opcionales.

Ver solucion

public List<Producto> busquedaAvanzada(String nombre, 
                                        BigDecimal precioMin, 
                                        BigDecimal precioMax,
                                        Boolean soloConStock,
                                        Boolean soloActivos) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Lista para acumular predicados
    List<Predicate> predicados = new ArrayList<>();
    
    // Nombre (opcional)
    if (nombre != null && !nombre.isBlank()) {
        predicados.add(
            cb.like(cb.lower(root.get("nombre")), 
                   "%" + nombre.toLowerCase() + "%")
        );
    }
    
    // Precio minimo (opcional)
    if (precioMin != null) {
        predicados.add(
            cb.greaterThanOrEqualTo(root.get("precio"), precioMin)
        );
    }
    
    // Precio maximo (opcional)
    if (precioMax != null) {
        predicados.add(
            cb.lessThanOrEqualTo(root.get("precio"), precioMax)
        );
    }
    
    // Solo con stock (opcional)
    if (Boolean.TRUE.equals(soloConStock)) {
        predicados.add(
            cb.greaterThan(root.get("stock"), 0)
        );
    }
    
    // Solo activos (opcional)
    if (Boolean.TRUE.equals(soloActivos)) {
        predicados.add(
            cb.isTrue(root.get("activo"))
        );
    }
    
    // Aplicar predicados si hay alguno
    if (!predicados.isEmpty()) {
        query.where(cb.and(predicados.toArray(new Predicate[0])));
    }
    
    query.select(root).orderBy(cb.asc(root.get("nombre")));
    
    return entityManager.createQuery(query).getResultList();
}

**Patron importante:**

- Usar `List<Predicate>` para acumular condiciones
    
- Solo agregar predicado si el parametro tiene valor
    
- Combinar con `cb.and()` al final
    

## Ejercicio 3: Consultas con JOIN

### Enunciado

Implementa un metodo que busque productos por nombre de categoria, incluyendo un fetch join para evitar N+1.

Ver solucion

public List<Producto> findByCategoriaNombre(String nombreCategoria) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Fetch JOIN para cargar categoria
    root.fetch("categoria", JoinType.LEFT);
    
    // JOIN para filtrar
    Join<Producto, Categoria> categoriaJoin = root.join("categoria");
    
    // Condicion sobre la categoria
    Predicate condicion = cb.equal(categoriaJoin.get("nombre"), nombreCategoria);
    
    query.select(root).where(condicion);
    
    return entityManager.createQuery(query).getResultList();
}

// Alternativa usando navegacion implicita:
public List<Producto> findByCategoriaNombreSimple(String nombreCategoria) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Navegacion implicita (genera JOIN automatico)
    Predicate condicion = cb.equal(
        root.get("categoria").get("nombre"), 
        nombreCategoria
    );
    
    query.select(root).where(condicion);
    
    return entityManager.createQuery(query).getResultList();
}

**Diferencia fetch vs join:**

- `fetch()`: Carga la relacion en la misma consulta (evita N+1)
    
- `join()`: Solo para filtrar, no carga la relacion
    

## Ejercicio 4: Specifications de Spring Data

### Enunciado

Crea Specifications reutilizables para los filtros del Ejercicio 2 y usalas en un repositorio Spring Data.

Ver solucion

**Clase de Specifications:**

public class ProductoSpecs {
    
    public static Specification<Producto> nombreContiene(String nombre) {
        return (root, query, cb) -> {
            if (nombre == null || nombre.isBlank()) {
                return cb.conjunction();  // true
            }
            return cb.like(
                cb.lower(root.get("nombre")), 
                "%" + nombre.toLowerCase() + "%"
            );
        };
    }
    
    public static Specification<Producto> precioEntre(BigDecimal min, BigDecimal max) {
        return (root, query, cb) -> {
            List<Predicate> predicados = new ArrayList<>();
            if (min != null) {
                predicados.add(cb.greaterThanOrEqualTo(root.get("precio"), min));
            }
            if (max != null) {
                predicados.add(cb.lessThanOrEqualTo(root.get("precio"), max));
            }
            return predicados.isEmpty() 
                ? cb.conjunction() 
                : cb.and(predicados.toArray(new Predicate[0]));
        };
    }
    
    public static Specification<Producto> tieneStock() {
        return (root, query, cb) -> cb.greaterThan(root.get("stock"), 0);
    }
    
    public static Specification<Producto> estaActivo() {
        return (root, query, cb) -> cb.isTrue(root.get("activo"));
    }
    
    public static Specification<Producto> enCategoria(Long categoriaId) {
        return (root, query, cb) -> {
            if (categoriaId == null) {
                return cb.conjunction();
            }
            return cb.equal(root.get("categoria").get("id"), categoriaId);
        };
    }
}

**Repositorio:**

@Repository
public interface ProductoRepository extends JpaRepository<Producto, Long>,
                                           JpaSpecificationExecutor<Producto> {
}

**Servicio:**

@Service
public class ProductoService {
    
    @Autowired
    private ProductoRepository repository;
    
    public Page<Producto> buscar(FiltroProducto filtro, Pageable pageable) {
        Specification<Producto> spec = Specification
            .where(ProductoSpecs.nombreContiene(filtro.getNombre()))
            .and(ProductoSpecs.precioEntre(filtro.getPrecioMin(), filtro.getPrecioMax()))
            .and(ProductoSpecs.enCategoria(filtro.getCategoriaId()));
        
        if (filtro.isSoloActivos()) {
            spec = spec.and(ProductoSpecs.estaActivo());
        }
        
        if (filtro.isSoloConStock()) {
            spec = spec.and(ProductoSpecs.tieneStock());
        }
        
        return repository.findAll(spec, pageable);
    }
}

**Ventajas de Specifications:**

- Reutilizables
    
- Componibles con `.and()` y `.or()`
    
- Integracion con paginacion
    
- Testables de forma aislada
    

## Ejercicio 5: Agregaciones con Criteria

### Enunciado

Implementa un metodo que devuelva estadisticas por categoria:

- Nombre de la categoria
    
- Numero de productos
    
- Precio promedio
    
- Stock total


Solo para categorias con mas de 5 productos.

Ver solucion

public List<Object[]> getEstadisticasPorCategoria() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Object[]> query = cb.createQuery(Object[].class);
    Root<Producto> root = query.from(Producto.class);
    
    // Join a categoria
    Join<Producto, Categoria> catJoin = root.join("categoria");
    
    // Expresiones de agregacion
    Expression<String> nombreCat = catJoin.get("nombre");
    Expression<Long> count = cb.count(root);
    Expression<Double> avgPrecio = cb.avg(root.get("precio"));
    Expression<Integer> sumStock = cb.sum(root.get("stock"));
    
    // SELECT nombre, COUNT(*), AVG(precio), SUM(stock)
    query.multiselect(nombreCat, count, avgPrecio, sumStock);
    
    // GROUP BY categoria.nombre
    query.groupBy(nombreCat);
    
    // HAVING COUNT(*) > 5
    query.having(cb.greaterThan(count, 5L));
    
    // ORDER BY count DESC
    query.orderBy(cb.desc(count));
    
    return entityManager.createQuery(query).getResultList();
}

// Version con DTO
public List<EstadisticaCategoriaDTO> getEstadisticasDTO() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<EstadisticaCategoriaDTO> query = 
        cb.createQuery(EstadisticaCategoriaDTO.class);
    Root<Producto> root = query.from(Producto.class);
    Join<Producto, Categoria> catJoin = root.join("categoria");
    
    // Constructor del DTO
    query.select(cb.construct(
        EstadisticaCategoriaDTO.class,
        catJoin.get("nombre"),
        cb.count(root),
        cb.avg(root.get("precio")),
        cb.sum(root.get("stock"))
    ));
    
    query.groupBy(catJoin.get("nombre"));
    query.having(cb.greaterThan(cb.count(root), 5L));
    
    return entityManager.createQuery(query).getResultList();
}

**DTO:**

public class EstadisticaCategoriaDTO {
    private String nombreCategoria;
    private Long totalProductos;
    private Double precioPromedio;
    private Long stockTotal;
    
    public EstadisticaCategoriaDTO(String nombreCategoria, 
                                    Long totalProductos,
                                    Double precioPromedio, 
                                    Long stockTotal) {
        this.nombreCategoria = nombreCategoria;
        this.totalProductos = totalProductos;
        this.precioPromedio = precioPromedio;
        this.stockTotal = stockTotal;
    }
    // getters...
}

## Reto adicional

Implementa una Specification que soporte busqueda en multiples campos con un solo termino:

- Buscar el termino en nombre, descripcion y codigo de producto
    
- Usar OR para combinar las condiciones


Consejo

Usa `cb.or()` para combinar los predicados de cada campo.