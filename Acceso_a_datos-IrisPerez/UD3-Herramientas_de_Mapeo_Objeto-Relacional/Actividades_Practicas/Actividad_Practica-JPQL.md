## Objetivos

- Dominar la sintaxis JPQL
    
- Realizar consultas con JOIN y subconsultas
    
- Usar funciones de agregacion
    
- Aplicar proyecciones a DTOs

## Contexto

Sistema de tienda online con:

- `Producto`: id, nombre, precio, stock, activo
    
- `Categoria`: id, nombre
    
- `Pedido`: id, fecha, total, cliente
    
- `LineaPedido`: id, cantidad, precioUnitario, producto, pedido

## Ejercicio 1: Consultas basicas

### Enunciado

Escribe las consultas JPQL para:

1. Todos los productos activos ordenados por nombre
    
2. Productos con precio mayor a 100 y stock mayor a 0
    
3. Productos cuyo nombre contenga «laptop» (case insensitive)

Ver solucion

// 1. Productos activos ordenados por nombre
@Query("SELECT p FROM Producto p WHERE p.activo = true ORDER BY p.nombre")
List<Producto> findActivosOrdenados();

// Alternativa con parametro
@Query("SELECT p FROM Producto p WHERE p.activo = :activo ORDER BY p.nombre ASC")
List<Producto> findByActivo(@Param("activo") boolean activo);

// 2. Productos con precio > 100 y stock > 0
@Query("SELECT p FROM Producto p " +
       "WHERE p.precio > :precioMin AND p.stock > 0 " +
       "ORDER BY p.precio DESC")
List<Producto> findDisponiblesConPrecioMinimo(@Param("precioMin") BigDecimal precioMin);

// 3. Buscar por nombre (case insensitive)
@Query("SELECT p FROM Producto p " +
       "WHERE LOWER(p.nombre) LIKE LOWER(CONCAT('%', :termino, '%'))")
List<Producto> buscarPorNombre(@Param("termino") String termino);

// Alternativa mas clara
@Query("SELECT p FROM Producto p " +
       "WHERE LOWER(p.nombre) LIKE :termino")
List<Producto> buscarPorNombreV2(@Param("termino") String termino);
// Llamada: repo.buscarPorNombreV2("%" + termino.toLowerCase() + "%");

**Uso con EntityManager:**

public List<Producto> buscarLaptops() {
    return em.createQuery(
        "SELECT p FROM Producto p " +
        "WHERE LOWER(p.nombre) LIKE :term", Producto.class)
        .setParameter("term", "%laptop%")
        .getResultList();
}

## Ejercicio 2: Consultas con JOIN

### Enunciado

Escribe las consultas JPQL para:

1. Productos de una categoria especifica
    
2. Pedidos con sus lineas (evitando N+1)
    
3. Productos que han sido pedidos al menos una vez


Ver solucion

// 1. Productos de una categoria
@Query("SELECT p FROM Producto p " +
       "JOIN p.categoria c " +
       "WHERE c.nombre = :nombreCategoria")
List<Producto> findByCategoria(@Param("nombreCategoria") String nombreCategoria);

// Con JOIN implicito (navegacion)
@Query("SELECT p FROM Producto p " +
       "WHERE p.categoria.nombre = :nombre")
List<Producto> findByCategoriaSimple(@Param("nombre") String nombre);

// 2. Pedidos con lineas (FETCH JOIN)
@Query("SELECT DISTINCT pe FROM Pedido pe " +
       "LEFT JOIN FETCH pe.lineas " +
       "WHERE pe.fecha >= :desde")
List<Pedido> findPedidosConLineas(@Param("desde") LocalDate desde);

// Pedidos con lineas Y productos de cada linea
@Query("SELECT DISTINCT pe FROM Pedido pe " +
       "LEFT JOIN FETCH pe.lineas li " +
       "LEFT JOIN FETCH li.producto " +
       "WHERE pe.id = :id")
Optional<Pedido> findPedidoCompleto(@Param("id") Long id);

// 3. Productos que han sido pedidos
@Query("SELECT DISTINCT p FROM Producto p " +
       "JOIN LineaPedido lp ON lp.producto = p")
List<Producto> findProductosPedidos();

// Alternativa con EXISTS
@Query("SELECT p FROM Producto p " +
       "WHERE EXISTS (SELECT 1 FROM LineaPedido lp WHERE lp.producto = p)")
List<Producto> findProductosConPedidos();

**DISTINCT importante:**

- Con FETCH JOIN de colecciones, sin DISTINCT se duplican resultados
    
- JPA elimina duplicados en memoria, no en SQL


## Ejercicio 3: Funciones de agregacion

### Enunciado

Escribe las consultas JPQL para:

1. Numero total de productos y precio promedio
    
2. Productos por categoria (nombre, cantidad, precio promedio)
    
3. Top 5 productos mas vendidos (por cantidad)


Ver solucion

// 1. Estadisticas globales
@Query("SELECT COUNT(p), AVG(p.precio), MIN(p.precio), MAX(p.precio) " +
       "FROM Producto p WHERE p.activo = true")
Object[] getEstadisticasProductos();

// Uso:
Object[] stats = repository.getEstadisticasProductos();
Long count = (Long) stats[0];
Double avgPrecio = (Double) stats[1];

// 2. Productos por categoria con GROUP BY
@Query("SELECT c.nombre, COUNT(p), AVG(p.precio) " +
       "FROM Producto p JOIN p.categoria c " +
       "GROUP BY c.nombre " +
       "ORDER BY COUNT(p) DESC")
List<Object[]> getEstadisticasPorCategoria();

// Con DTO (mejor practica)
@Query("SELECT new com.ejemplo.dto.CategoriaStatsDTO(" +
       "c.nombre, COUNT(p), AVG(p.precio)) " +
       "FROM Producto p JOIN p.categoria c " +
       "GROUP BY c.nombre " +
       "HAVING COUNT(p) >= :minProductos")
List<CategoriaStatsDTO> getStatsCategoriasConMinimo(
    @Param("minProductos") long minProductos);

// 3. Top 5 productos mas vendidos
@Query("SELECT p.nombre, SUM(lp.cantidad) as totalVendido " +
       "FROM LineaPedido lp " +
       "JOIN lp.producto p " +
       "GROUP BY p.id, p.nombre " +
       "ORDER BY totalVendido DESC")
List<Object[]> getTopProductosVendidos(Pageable pageable);

// Uso:
List<Object[]> top5 = repository.getTopProductosVendidos(PageRequest.of(0, 5));

**DTO para resultados:**

public class CategoriaStatsDTO {
    private String nombre;
    private Long cantidad;
    private Double precioPromedio;
    
    public CategoriaStatsDTO(String nombre, Long cantidad, Double precioPromedio) {
        this.nombre = nombre;
        this.cantidad = cantidad;
        this.precioPromedio = precioPromedio;
    }
    // getters...
}

## Ejercicio 4: Subconsultas

### Enunciado

Escribe las consultas JPQL para:

1. Productos con precio mayor al promedio de su categoria
    
2. Categorias sin productos
    
3. El producto mas caro de cada categoria
    

Ver solucion

// 1. Productos con precio mayor al promedio de SU categoria
@Query("SELECT p FROM Producto p " +
       "WHERE p.precio > (" +
       "  SELECT AVG(p2.precio) FROM Producto p2 " +
       "  WHERE p2.categoria = p.categoria" +
       ")")
List<Producto> findProductosPrecioSobrePromedio();

// 2. Categorias sin productos (NOT EXISTS)
@Query("SELECT c FROM Categoria c " +
       "WHERE NOT EXISTS (" +
       "  SELECT p FROM Producto p WHERE p.categoria = c" +
       ")")
List<Categoria> findCategoriasVacias();

// Alternativa con LEFT JOIN
@Query("SELECT c FROM Categoria c " +
       "LEFT JOIN c.productos p " +
       "WHERE p IS NULL")
List<Categoria> findCategoriasVaciasV2();

// 3. Producto mas caro de cada categoria
@Query("SELECT p FROM Producto p " +
       "WHERE p.precio = (" +
       "  SELECT MAX(p2.precio) FROM Producto p2 " +
       "  WHERE p2.categoria = p.categoria" +
       ")")
List<Producto> findProductosMasCarosPorCategoria();

// Con ALL (productos cuyo precio >= todos de su categoria)
@Query("SELECT p FROM Producto p " +
       "WHERE p.precio >= ALL (" +
       "  SELECT p2.precio FROM Producto p2 " +
       "  WHERE p2.categoria = p.categoria" +
       ")")
List<Producto> findProductosMasCarosV2();

**Operadores de subconsulta:**

- `ALL`: Compara con todos los valores
    
- `ANY/SOME`: Compara con al menos un valor
    
- `EXISTS`: Verifica existencia de resultados
    
- `IN`: Valor esta en el resultado de subconsulta
    

## Ejercicio 5: UPDATE y DELETE masivos

### Enunciado

Escribe las consultas JPQL para:

1. Incrementar el precio de todos los productos de una categoria en un 10%
    
2. Desactivar productos sin stock
    
3. Eliminar pedidos antiguos (mas de 2 anos)


Ver solucion

// 1. Incrementar precios de una categoria
@Modifying
@Query("UPDATE Producto p " +
       "SET p.precio = p.precio * (1 + :porcentaje / 100) " +
       "WHERE p.categoria.id = :categoriaId")
int incrementarPreciosPorCategoria(
    @Param("categoriaId") Long categoriaId,
    @Param("porcentaje") double porcentaje);

// 2. Desactivar productos sin stock
@Modifying
@Query("UPDATE Producto p " +
       "SET p.activo = false " +
       "WHERE p.stock <= 0 AND p.activo = true")
int desactivarProductosSinStock();

// 3. Eliminar pedidos antiguos
@Modifying
@Query("DELETE FROM Pedido p " +
       "WHERE p.fecha < :fechaLimite")
int eliminarPedidosAntiguos(@Param("fechaLimite") LocalDate fechaLimite);

// Uso en servicio:
@Service
@Transactional
public class MantenimientoService {
    
    @Autowired
    private ProductoRepository productoRepo;
    
    @Autowired
    private PedidoRepository pedidoRepo;
    
    public void mantenimientoDiario() {
        // Desactivar sin stock
        int desactivados = productoRepo.desactivarProductosSinStock();
        log.info("Productos desactivados: {}", desactivados);
        
        // Eliminar pedidos de hace 2 anos
        LocalDate hace2Anos = LocalDate.now().minusYears(2);
        int eliminados = pedidoRepo.eliminarPedidosAntiguos(hace2Anos);
        log.info("Pedidos eliminados: {}", eliminados);
    }
}

**Importante:**

- @Modifying es obligatorio para UPDATE/DELETE
    
- Estas operaciones NO disparan callbacks (@PreUpdate, etc.)
    
- NO actualizan el cache de primer nivel
    
- Usar `@Modifying(clearAutomatically = true)` para limpiar cache


## Reto adicional

Implementa una consulta que genere un reporte de ventas:

- Agrupar por mes y categoria
    
- Mostrar: mes, categoria, unidades vendidas, ingresos totales
    
- Solo meses con ventas > 1000 euros
    
- Ordenar por ingresos descendente


Consejo

Usa `FUNCTION('MONTH', fecha)` y `FUNCTION('YEAR', fecha)` para extraer partes de fechas en JPQL.