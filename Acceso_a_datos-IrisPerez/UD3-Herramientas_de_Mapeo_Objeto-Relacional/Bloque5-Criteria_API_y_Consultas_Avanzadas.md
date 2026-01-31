## Objetivos de aprendizaje

Al finalizar este bloque, el alumno sera capaz de:

- Comprender la arquitectura y componentes de la Criteria API de JPA
    
- Construir consultas dinamicas y type-safe utilizando CriteriaBuilder
    
- Aplicar predicados simples y compuestos para filtrar resultados
    
- Realizar operaciones de agregacion y agrupacion con Criteria API
    
- Ejecutar consultas nativas SQL cuando sea necesario
    
- Mapear resultados de consultas nativas a entidades y DTOs
    
- Seleccionar el enfoque de consulta mas apropiado segun el caso de uso
    
- Optimizar el rendimiento de consultas complejas

## Criterios de evaluacion relacionados

Este bloque desarrolla principalmente:

- **CE 3f**: Se han desarrollado aplicaciones que realizan consultas usando el lenguaje SQL (20%)

Contribuye tambien a:

- **CE 3e**: Se han desarrollado aplicaciones que modifican y recuperan objetos persistentes (25%)

## 1. Introduccion a Criteria API

### 1.1. El problema de las consultas dinamicas

En aplicaciones reales, frecuentemente necesitamos construir consultas cuyas condiciones varian en tiempo de ejecucion. Por ejemplo, un formulario de busqueda avanzada donde el usuario puede filtrar por multiples criterios opcionales.

Con JPQL, construir consultas dinamicas requiere concatenar cadenas de texto, lo cual presenta varios problemas:

// Enfoque problematico con JPQL dinamico
public List<Producto> buscar(String nombre, BigDecimal precioMin, 
                             BigDecimal precioMax, String categoria) {
    StringBuilder jpql = new StringBuilder("SELECT p FROM Producto p WHERE 1=1");
    
    if (nombre != null && !nombre.isEmpty()) {
        jpql.append(" AND p.nombre LIKE :nombre");
    }
    if (precioMin != null) {
        jpql.append(" AND p.precio >= :precioMin");
    }
    if (precioMax != null) {
        jpql.append(" AND p.precio <= :precioMax");
    }
    if (categoria != null && !categoria.isEmpty()) {
        jpql.append(" AND p.categoria.nombre = :categoria");
    }
    
    TypedQuery<Producto> query = em.createQuery(jpql.toString(), Producto.class);
    
    // Asignar parametros condicionalmente
    if (nombre != null && !nombre.isEmpty()) {
        query.setParameter("nombre", "%" + nombre + "%");
    }
    // ... repetir para cada parametro
    
    return query.getResultList();
}

Este enfoque tiene varios inconvenientes:

1. No hay verificacion en tiempo de compilacion
    
2. Los errores de sintaxis solo se detectan en tiempo de ejecucion
    
3. El codigo es dificil de mantener y propenso a errores
    
4. Los cambios en el modelo de datos no se detectan automaticamente
    

### 1.2. Que es Criteria API

Criteria API es una API de JPA que permite construir consultas de forma programatica, utilizando objetos Java en lugar de cadenas de texto. Sus principales caracteristicas son:

**Type-safe**: Los errores se detectan en tiempo de compilacion gracias al uso de Metamodel (clases generadas que representan los atributos de las entidades).

**Orientada a objetos**: Las consultas se construyen combinando objetos que representan las diferentes partes de una consulta SQL.

**Dinamica**: Permite anadir condiciones de forma condicional sin concatenar cadenas.

**Portable**: Al ser parte del estandar JPA, funciona con cualquier implementacion.

### 1.3. Componentes principales

La Criteria API se compone de varios elementos fundamentales:

+------------------+     +-------------------+     +------------------+
| CriteriaBuilder  |---->| CriteriaQuery<T>  |---->| TypedQuery<T>    |
| (Fabrica)        |     | (Definicion)      |     | (Ejecucion)      |
+------------------+     +-------------------+     +------------------+
        |                         |
        v                         v
+------------------+     +-------------------+
| Predicate        |     | Root<T>           |
| (Condiciones)    |     | (Entidad raiz)    |
+------------------+     +-------------------+
                                  |
                                  v
                         +-------------------+
                         | Join<X,Y>         |
                         | (Relaciones)      |
                         +-------------------+

**CriteriaBuilder**: Es la fabrica principal que se obtiene del EntityManager. Proporciona metodos para crear consultas, predicados, expresiones y funciones.

**CriteriaQuery**: Representa la consulta completa. Define el tipo de resultado, las clausulas SELECT, FROM, WHERE, ORDER BY, GROUP BY y HAVING.

**Root**: Representa la entidad raiz de la consulta (equivalente al FROM en SQL). Desde aqui se accede a los atributos de la entidad.

**Predicate**: Representa una condicion o expresion booleana. Multiples predicados se pueden combinar con AND/OR.

**Join**: Representa una union con otra entidad relacionada.

## 2. Construccion de consultas basicas

### 2.1. Obtencion del CriteriaBuilder

El primer paso siempre es obtener el CriteriaBuilder desde el EntityManager:

@PersistenceContext
private EntityManager entityManager;

public void ejemplo() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    // Ahora podemos crear consultas
}

### 2.2. Consulta SELECT simple

Una consulta basica que recupera todas las entidades:

public List<Producto> findAll() {
    // 1. Obtener el CriteriaBuilder
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    
    // 2. Crear la consulta indicando el tipo de resultado
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    
    // 3. Definir la entidad raiz (FROM)
    Root<Producto> root = query.from(Producto.class);
    
    // 4. Construir la consulta SELECT
    query.select(root);
    
    // 5. Ejecutar y obtener resultados
    return entityManager.createQuery(query).getResultList();
}

La consulta generada seria equivalente a:

SELECT p FROM Producto p

### 2.3. Consulta con condicion simple

Anadir una clausula WHERE con una condicion de igualdad:

public Producto findByNombre(String nombre) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Crear predicado de igualdad
    Predicate nombreIgual = cb.equal(root.get("nombre"), nombre);
    
    // Aplicar la condicion WHERE
    query.select(root).where(nombreIgual);
    
    // Ejecutar (esperamos un unico resultado)
    return entityManager.createQuery(query).getSingleResult();
}

### 2.4. Operadores de comparacion

CriteriaBuilder proporciona metodos para todos los operadores de comparacion:

public List<Producto> ejemplosComparacion(BigDecimal precio, Integer stock) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Igualdad
    Predicate igual = cb.equal(root.get("nombre"), "Laptop");
    
    // Desigualdad
    Predicate noIgual = cb.notEqual(root.get("activo"), false);
    
    // Mayor que
    Predicate mayorQue = cb.greaterThan(root.get("precio"), precio);
    
    // Mayor o igual
    Predicate mayorIgual = cb.greaterThanOrEqualTo(root.get("stock"), stock);
    
    // Menor que
    Predicate menorQue = cb.lessThan(root.get("precio"), new BigDecimal("1000"));
    
    // Menor o igual
    Predicate menorIgual = cb.lessThanOrEqualTo(root.get("stock"), 100);
    
    // Entre (BETWEEN)
    Predicate entre = cb.between(root.get("precio"), 
                                 new BigDecimal("100"), 
                                 new BigDecimal("500"));
    
    // IS NULL
    Predicate esNulo = cb.isNull(root.get("descripcion"));
    
    // IS NOT NULL
    Predicate noNulo = cb.isNotNull(root.get("categoria"));
    
    // Combinar condiciones
    query.select(root).where(cb.and(mayorQue, noNulo));
    
    return entityManager.createQuery(query).getResultList();
}

### 2.5. Operador LIKE

Para busquedas por patron con comodines:

public List<Producto> findByNombreContaining(String termino) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // LIKE con comodines
    Predicate contiene = cb.like(root.get("nombre"), "%" + termino + "%");
    
    // LIKE case-insensitive (convertir ambos a minusculas)
    Predicate contieneIgnoreCase = cb.like(
        cb.lower(root.get("nombre")), 
        "%" + termino.toLowerCase() + "%"
    );
    
    // NOT LIKE
    Predicate noContiene = cb.notLike(root.get("nombre"), "%oferta%");
    
    query.select(root).where(contieneIgnoreCase);
    
    return entityManager.createQuery(query).getResultList();
}

### 2.6. Operador IN

Para verificar si un valor esta en una lista:

public List<Producto> findByCategorias(List<String> nombresCategorias) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // IN con lista de valores
    Predicate enCategorias = root.get("categoria").get("nombre").in(nombresCategorias);
    
    // NOT IN
    Predicate noEnCategorias = cb.not(root.get("estado").in("DESCATALOGADO", "AGOTADO"));
    
    query.select(root).where(enCategorias);
    
    return entityManager.createQuery(query).getResultList();
}

## 3. Predicados compuestos y consultas dinamicas

### 3.1. Combinacion de predicados con AND/OR

Los predicados se pueden combinar usando operadores logicos:

public List<Producto> findByMultipleCriteria(String nombre, 
                                             BigDecimal precioMin,
                                             BigDecimal precioMax) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Crear predicados individuales
    Predicate nombreLike = cb.like(root.get("nombre"), "%" + nombre + "%");
    Predicate precioMayor = cb.greaterThanOrEqualTo(root.get("precio"), precioMin);
    Predicate precioMenor = cb.lessThanOrEqualTo(root.get("precio"), precioMax);
    Predicate activo = cb.isTrue(root.get("activo"));
    
    // Combinar con AND
    Predicate condicionAnd = cb.and(nombreLike, precioMayor, precioMenor, activo);
    
    // Combinar con OR
    Predicate descatalogado = cb.equal(root.get("estado"), "DESCATALOGADO");
    Predicate sinStock = cb.equal(root.get("stock"), 0);
    Predicate condicionOr = cb.or(descatalogado, sinStock);
    
    // Combinar AND y OR
    // (nombre AND precio AND activo) OR (descatalogado OR sinStock)
    Predicate condicionCompleja = cb.or(condicionAnd, condicionOr);
    
    query.select(root).where(condicionCompleja);
    
    return entityManager.createQuery(query).getResultList();
}

### 3.2. Construccion dinamica de consultas

El verdadero poder de Criteria API esta en construir consultas cuyas condiciones se determinan en tiempo de ejecucion:

public List<Producto> busquedaAvanzada(String nombre, 
                                       BigDecimal precioMin,
                                       BigDecimal precioMax, 
                                       String categoria,
                                       Boolean soloActivos,
                                       Boolean soloConStock) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Lista para acumular predicados
    List<Predicate> predicados = new ArrayList<>();
    
    // Anadir condiciones solo si el parametro tiene valor
    if (nombre != null && !nombre.trim().isEmpty()) {
        predicados.add(cb.like(
            cb.lower(root.get("nombre")), 
            "%" + nombre.toLowerCase() + "%"
        ));
    }
    
    if (precioMin != null) {
        predicados.add(cb.greaterThanOrEqualTo(root.get("precio"), precioMin));
    }
    
    if (precioMax != null) {
        predicados.add(cb.lessThanOrEqualTo(root.get("precio"), precioMax));
    }
    
    if (categoria != null && !categoria.trim().isEmpty()) {
        predicados.add(cb.equal(root.get("categoria").get("nombre"), categoria));
    }
    
    if (soloActivos != null && soloActivos) {
        predicados.add(cb.isTrue(root.get("activo")));
    }
    
    if (soloConStock != null && soloConStock) {
        predicados.add(cb.greaterThan(root.get("stock"), 0));
    }
    
    // Aplicar todos los predicados con AND
    if (!predicados.isEmpty()) {
        query.where(cb.and(predicados.toArray(new Predicate[0])));
    }
    
    query.select(root);
    
    return entityManager.createQuery(query).getResultList();
}

### 3.3. Clase Specification de Spring Data JPA

Spring Data JPA proporciona la interfaz Specification que simplifica la creacion de consultas dinamicas:

// Definicion de Specifications reutilizables
public class ProductoSpecifications {
    
    public static Specification<Producto> nombreContiene(String nombre) {
        return (root, query, cb) -> {
            if (nombre == null || nombre.trim().isEmpty()) {
                return cb.conjunction(); // Siempre true
            }
            return cb.like(cb.lower(root.get("nombre")), 
                          "%" + nombre.toLowerCase() + "%");
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
            return cb.and(predicados.toArray(new Predicate[0]));
        };
    }
    
    public static Specification<Producto> categoriaEs(String categoria) {
        return (root, query, cb) -> {
            if (categoria == null || categoria.trim().isEmpty()) {
                return cb.conjunction();
            }
            return cb.equal(root.get("categoria").get("nombre"), categoria);
        };
    }
    
    public static Specification<Producto> estaActivo() {
        return (root, query, cb) -> cb.isTrue(root.get("activo"));
    }
    
    public static Specification<Producto> tieneStock() {
        return (root, query, cb) -> cb.greaterThan(root.get("stock"), 0);
    }
}

Uso en el repositorio:

// El repositorio debe extender JpaSpecificationExecutor
public interface ProductoRepository extends JpaRepository<Producto, Long>,
                                           JpaSpecificationExecutor<Producto> {
}

// Uso en el servicio
@Service
public class ProductoService {
    
    @Autowired
    private ProductoRepository productoRepository;
    
    public List<Producto> busquedaAvanzada(FiltroProducto filtro) {
        Specification<Producto> spec = Specification
            .where(ProductoSpecifications.nombreContiene(filtro.getNombre()))
            .and(ProductoSpecifications.precioEntre(filtro.getPrecioMin(), 
                                                    filtro.getPrecioMax()))
            .and(ProductoSpecifications.categoriaEs(filtro.getCategoria()));
        
        if (filtro.isSoloActivos()) {
            spec = spec.and(ProductoSpecifications.estaActivo());
        }
        
        if (filtro.isSoloConStock()) {
            spec = spec.and(ProductoSpecifications.tieneStock());
        }
        
        return productoRepository.findAll(spec);
    }
}

## 4. Joins y navegacion entre entidades

### 4.1. Join implicito

Cuando accedemos a una relacion mediante get(), JPA realiza un join implicito:

public List<Producto> findByCategoriaNombre(String nombreCategoria) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Join implicito: root.get("categoria").get("nombre")
    // JPA genera: ... INNER JOIN categorias c ON p.categoria_id = c.id
    //             WHERE c.nombre = ?
    Predicate condicion = cb.equal(root.get("categoria").get("nombre"), nombreCategoria);
    
    query.select(root).where(condicion);
    
    return entityManager.createQuery(query).getResultList();
}

### 4.2. Join explicito

Para mayor control, podemos definir joins explicitamente:

public List<Producto> findByProveedorCiudad(String ciudad) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> productoRoot = query.from(Producto.class);
    
    // Join explicito con la entidad Proveedor
    Join<Producto, Proveedor> proveedorJoin = productoRoot.join("proveedor");
    
    // Ahora podemos usar el join para acceder a atributos del proveedor
    Predicate condicion = cb.equal(proveedorJoin.get("ciudad"), ciudad);
    
    query.select(productoRoot).where(condicion);
    
    return entityManager.createQuery(query).getResultList();
}

### 4.3. Tipos de Join

JPA soporta diferentes tipos de join:

public List<Pedido> findPedidosConDetalles() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Pedido> query = cb.createQuery(Pedido.class);
    Root<Pedido> pedidoRoot = query.from(Pedido.class);
    
    // INNER JOIN (por defecto)
    Join<Pedido, DetallePedido> innerJoin = pedidoRoot.join("detalles");
    
    // LEFT JOIN - incluye pedidos sin detalles
    Join<Pedido, DetallePedido> leftJoin = pedidoRoot.join("detalles", JoinType.LEFT);
    
    // RIGHT JOIN (no soportado en todas las implementaciones)
    // Join<Pedido, DetallePedido> rightJoin = pedidoRoot.join("detalles", JoinType.RIGHT);
    
    query.select(pedidoRoot).distinct(true);
    
    return entityManager.createQuery(query).getResultList();
}

### 4.4. Fetch Join para evitar N+1

El fetch join carga las relaciones en una sola consulta:

public List<Pedido> findPedidosConClienteYDetalles() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Pedido> query = cb.createQuery(Pedido.class);
    Root<Pedido> pedidoRoot = query.from(Pedido.class);
    
    // Fetch join: carga el cliente junto con el pedido
    pedidoRoot.fetch("cliente", JoinType.LEFT);
    
    // Fetch join para la coleccion de detalles
    pedidoRoot.fetch("detalles", JoinType.LEFT);
    
    query.select(pedidoRoot).distinct(true);
    
    return entityManager.createQuery(query).getResultList();
}

### 4.5. Join multiple y cadena de joins

Para relaciones mas complejas:

public List<Producto> findByFabricanteYDistribuidor(String paisFabricante, 
                                                    String ciudadDistribuidor) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> productoRoot = query.from(Producto.class);
    
    // Cadena de joins: Producto -> Categoria -> Departamento
    Join<Producto, Categoria> categoriaJoin = productoRoot.join("categoria");
    Join<Categoria, Departamento> departamentoJoin = categoriaJoin.join("departamento");
    
    // Otro join desde el root
    Join<Producto, Proveedor> proveedorJoin = productoRoot.join("proveedores");
    
    // Condiciones usando ambos joins
    Predicate condicionPais = cb.equal(departamentoJoin.get("pais"), paisFabricante);
    Predicate condicionCiudad = cb.equal(proveedorJoin.get("ciudad"), ciudadDistribuidor);
    
    query.select(productoRoot).where(cb.and(condicionPais, condicionCiudad));
    
    return entityManager.createQuery(query).getResultList();
}

## 5. Ordenacion, paginacion y proyecciones

### 5.1. Ordenacion con ORDER BY

public List<Producto> findAllOrderByPrecio(boolean ascendente) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Ordenar ascendente o descendente
    if (ascendente) {
        query.orderBy(cb.asc(root.get("precio")));
    } else {
        query.orderBy(cb.desc(root.get("precio")));
    }
    
    query.select(root);
    
    return entityManager.createQuery(query).getResultList();
}

public List<Producto> findAllOrderByMultiple() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Ordenacion multiple: primero por categoria, luego por precio desc
    query.orderBy(
        cb.asc(root.get("categoria").get("nombre")),
        cb.desc(root.get("precio"))
    );
    
    query.select(root);
    
    return entityManager.createQuery(query).getResultList();
}

### 5.2. Paginacion

La paginacion se aplica al TypedQuery, no al CriteriaQuery:

public List<Producto> findAllPaginated(int pagina, int tamanoPagina) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    query.select(root).orderBy(cb.asc(root.get("id")));
    
    TypedQuery<Producto> typedQuery = entityManager.createQuery(query);
    
    // Configurar paginacion
    typedQuery.setFirstResult(pagina * tamanoPagina); // Offset
    typedQuery.setMaxResults(tamanoPagina);           // Limit
    
    return typedQuery.getResultList();
}

// Metodo auxiliar para contar total de registros
public Long countAll() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Long> query = cb.createQuery(Long.class);
    Root<Producto> root = query.from(Producto.class);
    
    query.select(cb.count(root));
    
    return entityManager.createQuery(query).getSingleResult();
}

### 5.3. Proyecciones: seleccionar campos especificos

En lugar de recuperar entidades completas, podemos seleccionar solo los campos necesarios:

// Proyeccion a Object[]
public List<Object[]> findNombreYPrecio() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Object[]> query = cb.createQuery(Object[].class);
    Root<Producto> root = query.from(Producto.class);
    
    // Seleccionar multiples campos
    query.multiselect(
        root.get("nombre"),
        root.get("precio"),
        root.get("stock")
    );
    
    List<Object[]> resultados = entityManager.createQuery(query).getResultList();
    
    // Procesar resultados
    for (Object[] fila : resultados) {
        String nombre = (String) fila[0];
        BigDecimal precio = (BigDecimal) fila[1];
        Integer stock = (Integer) fila[2];
        // ...
    }
    
    return resultados;
}

// Proyeccion a DTO con constructor
public List<ProductoResumenDTO> findResumen() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<ProductoResumenDTO> query = cb.createQuery(ProductoResumenDTO.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Usar construct() para mapear a DTO
    query.select(cb.construct(
        ProductoResumenDTO.class,
        root.get("id"),
        root.get("nombre"),
        root.get("precio"),
        root.get("categoria").get("nombre")
    ));
    
    return entityManager.createQuery(query).getResultList();
}

El DTO debe tener un constructor con los parametros en el mismo orden:

public class ProductoResumenDTO {
    private Long id;
    private String nombre;
    private BigDecimal precio;
    private String categoriaNombre;
    
    // Constructor requerido para cb.construct()
    public ProductoResumenDTO(Long id, String nombre, 
                              BigDecimal precio, String categoriaNombre) {
        this.id = id;
        this.nombre = nombre;
        this.precio = precio;
        this.categoriaNombre = categoriaNombre;
    }
    
    // Getters...
}

### 5.4. Proyeccion con Tuple

Tuple proporciona acceso tipado a los resultados:

public List<Tuple> findProductosTuple() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Tuple> query = cb.createTupleQuery();
    Root<Producto> root = query.from(Producto.class);
    
    // Definir alias para los campos
    Path<String> nombrePath = root.get("nombre");
    Path<BigDecimal> precioPath = root.get("precio");
    
    query.multiselect(
        nombrePath.alias("nombre"),
        precioPath.alias("precio")
    );
    
    List<Tuple> resultados = entityManager.createQuery(query).getResultList();
    
    // Acceso tipado por alias
    for (Tuple tuple : resultados) {
        String nombre = tuple.get("nombre", String.class);
        BigDecimal precio = tuple.get("precio", BigDecimal.class);
        // O por indice
        String nombre2 = tuple.get(0, String.class);
    }
    
    return resultados;
}

## 6. Funciones de agregacion y agrupacion

### 6.1. Funciones de agregacion

public Map<String, Object> getEstadisticasProductos() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Object[]> query = cb.createQuery(Object[].class);
    Root<Producto> root = query.from(Producto.class);
    
    query.multiselect(
        cb.count(root),                          // COUNT
        cb.sum(root.get("stock")),               // SUM
        cb.avg(root.get("precio")),              // AVG
        cb.min(root.<BigDecimal>get("precio")),  // MIN
        cb.max(root.<BigDecimal>get("precio"))   // MAX
    );
    
    Object[] resultado = entityManager.createQuery(query).getSingleResult();
    
    Map<String, Object> estadisticas = new HashMap<>();
    estadisticas.put("totalProductos", resultado[0]);
    estadisticas.put("stockTotal", resultado[1]);
    estadisticas.put("precioPromedio", resultado[2]);
    estadisticas.put("precioMinimo", resultado[3]);
    estadisticas.put("precioMaximo", resultado[4]);
    
    return estadisticas;
}

### 6.2. GROUP BY y HAVING

public List<Object[]> getProductosPorCategoria() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Object[]> query = cb.createQuery(Object[].class);
    Root<Producto> root = query.from(Producto.class);
    
    // SELECT categoria.nombre, COUNT(*), AVG(precio)
    Expression<String> categoria = root.get("categoria").get("nombre");
    Expression<Long> cantidad = cb.count(root);
    Expression<Double> precioPromedio = cb.avg(root.get("precio"));
    
    query.multiselect(categoria, cantidad, precioPromedio);
    
    // GROUP BY categoria.nombre
    query.groupBy(categoria);
    
    // HAVING COUNT(*) > 5
    query.having(cb.greaterThan(cantidad, 5L));
    
    // ORDER BY cantidad DESC
    query.orderBy(cb.desc(cantidad));
    
    return entityManager.createQuery(query).getResultList();
}

public List<Object[]> getVentasPorMes() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Object[]> query = cb.createQuery(Object[].class);
    Root<Venta> root = query.from(Venta.class);
    
    // Extraer mes y anio de la fecha
    Expression<Integer> mes = cb.function("MONTH", Integer.class, root.get("fecha"));
    Expression<Integer> anio = cb.function("YEAR", Integer.class, root.get("fecha"));
    
    Expression<Long> totalVentas = cb.count(root);
    Expression<Number> ingresoTotal = cb.sum(root.get("total"));
    
    query.multiselect(anio, mes, totalVentas, ingresoTotal);
    query.groupBy(anio, mes);
    query.orderBy(cb.desc(anio), cb.desc(mes));
    
    return entityManager.createQuery(query).getResultList();
}

### 6.3. COUNT DISTINCT

public Long countCategoriasActivas() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Long> query = cb.createQuery(Long.class);
    Root<Producto> root = query.from(Producto.class);
    
    // COUNT(DISTINCT categoria_id) WHERE activo = true
    query.select(cb.countDistinct(root.get("categoria")));
    query.where(cb.isTrue(root.get("activo")));
    
    return entityManager.createQuery(query).getSingleResult();
}

## 7. Subconsultas

### 7.1. Subconsulta en WHERE

// Productos con precio superior al promedio
public List<Producto> findProductosPrecioSuperiorPromedio() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Subconsulta: SELECT AVG(precio) FROM Producto
    Subquery<Double> subquery = query.subquery(Double.class);
    Root<Producto> subRoot = subquery.from(Producto.class);
    subquery.select(cb.avg(subRoot.get("precio")));
    
    // WHERE precio > (subconsulta)
    query.select(root).where(cb.greaterThan(root.get("precio"), subquery));
    
    return entityManager.createQuery(query).getResultList();
}

// Clientes que han realizado pedidos
public List<Cliente> findClientesConPedidos() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Cliente> query = cb.createQuery(Cliente.class);
    Root<Cliente> clienteRoot = query.from(Cliente.class);
    
    // Subconsulta: EXISTS (SELECT 1 FROM Pedido WHERE cliente_id = ?)
    Subquery<Pedido> subquery = query.subquery(Pedido.class);
    Root<Pedido> pedidoRoot = subquery.from(Pedido.class);
    subquery.select(pedidoRoot);
    subquery.where(cb.equal(pedidoRoot.get("cliente"), clienteRoot));
    
    // WHERE EXISTS (subconsulta)
    query.select(clienteRoot).where(cb.exists(subquery));
    
    return entityManager.createQuery(query).getResultList();
}

### 7.2. Subconsulta correlacionada

// Producto mas caro de cada categoria
public List<Producto> findProductosMasCarosPorCategoria() {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Subconsulta correlacionada
    Subquery<BigDecimal> subquery = query.subquery(BigDecimal.class);
    Root<Producto> subRoot = subquery.from(Producto.class);
    
    // MAX(precio) WHERE categoria_id = p.categoria_id
    subquery.select(cb.max(subRoot.get("precio")));
    subquery.where(cb.equal(
        subRoot.get("categoria"), 
        root.get("categoria")
    ));
    
    // WHERE precio = (subconsulta)
    query.select(root).where(cb.equal(root.get("precio"), subquery));
    
    return entityManager.createQuery(query).getResultList();
}

## 8. Metamodel: consultas type-safe

### 8.1. El problema de los strings magicos

En los ejemplos anteriores, accedemos a los atributos usando strings:

root.get("nombre")  // "nombre" es un string magico
root.get("precio")  // Si se renombra el campo, el compilador no lo detecta

Esto puede causar errores en tiempo de ejecucion si:

1. Se renombra un atributo en la entidad
    
2. Se escribe mal el nombre del atributo
    
3. Se cambia el tipo del atributo
    

### 8.2. Generacion del Metamodel

El Metamodel son clases generadas automaticamente que representan los atributos de las entidades. Para generarlas, anade la dependencia del procesador de anotaciones:

<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-jpamodelgen</artifactId>
    <scope>provided</scope>
</dependency>

Configuracion en el pom.xml para Maven:

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>17</source>
                <target>17</target>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.hibernate.orm</groupId>
                        <artifactId>hibernate-jpamodelgen</artifactId>
                        <version>6.4.1.Final</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>

### 8.3. Clases Metamodel generadas

Para una entidad como:

@Entity
public class Producto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private BigDecimal precio;
    private Integer stock;
    private Boolean activo;
    
    @ManyToOne
    private Categoria categoria;
    
    // getters y setters...
}

Se genera automaticamente:

@StaticMetamodel(Producto.class)
public class Producto_ {
    public static volatile SingularAttribute<Producto, Long> id;
    public static volatile SingularAttribute<Producto, String> nombre;
    public static volatile SingularAttribute<Producto, BigDecimal> precio;
    public static volatile SingularAttribute<Producto, Integer> stock;
    public static volatile SingularAttribute<Producto, Boolean> activo;
    public static volatile SingularAttribute<Producto, Categoria> categoria;
}

### 8.4. Uso del Metamodel en consultas

Ahora las consultas son completamente type-safe:

public List<Producto> findByNombreTypeSafe(String nombre) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    // Usando Metamodel en lugar de strings
    Predicate condicion = cb.equal(root.get(Producto_.nombre), nombre);
    
    query.select(root).where(condicion);
    
    return entityManager.createQuery(query).getResultList();
}

public List<Producto> busquedaTypeSafe(String nombre, 
                                       BigDecimal precioMin,
                                       BigDecimal precioMax) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
    Root<Producto> root = query.from(Producto.class);
    
    List<Predicate> predicados = new ArrayList<>();
    
    if (nombre != null) {
        predicados.add(cb.like(
            cb.lower(root.get(Producto_.nombre)), 
            "%" + nombre.toLowerCase() + "%"
        ));
    }
    
    if (precioMin != null) {
        predicados.add(cb.greaterThanOrEqualTo(root.get(Producto_.precio), precioMin));
    }
    
    if (precioMax != null) {
        predicados.add(cb.lessThanOrEqualTo(root.get(Producto_.precio), precioMax));
    }
    
    // Acceso a relaciones tambien es type-safe
    // root.get(Producto_.categoria).get(Categoria_.nombre)
    
    if (!predicados.isEmpty()) {
        query.where(cb.and(predicados.toArray(new Predicate[0])));
    }
    
    query.select(root).orderBy(cb.asc(root.get(Producto_.nombre)));
    
    return entityManager.createQuery(query).getResultList();
}

Ventajas del Metamodel:

1. Errores de compilacion si el atributo no existe
    
2. Autocompletado en el IDE
    
3. Refactoring automatico cuando se renombran atributos
    
4. Documentacion implicita del tipo de cada atributo
    

## 9. Consultas nativas SQL

### 9.1. Cuando usar SQL nativo

A pesar de la potencia de JPQL y Criteria API, hay situaciones donde SQL nativo es necesario:

1. **Funciones especificas del SGBD**: Full-text search, funciones geoespaciales, funciones de ventana
    
2. **Optimizacion extrema**: Hints del optimizador, indices especificos
    
3. **Consultas a tablas no mapeadas**: Tablas de sistema, vistas complejas
    
4. **Compatibilidad legacy**: Consultas SQL existentes que funcionan bien
    
5. **Operaciones masivas**: Bulk updates/deletes muy grandes
    

### 9.2. Consultas nativas con EntityManager

// Consulta nativa simple que devuelve entidades
@SuppressWarnings("unchecked")
public List<Producto> findByFullTextSearch(String termino) {
    String sql = """
        SELECT * FROM productos 
        WHERE MATCH(nombre, descripcion) AGAINST(:termino IN NATURAL LANGUAGE MODE)
        ORDER BY MATCH(nombre, descripcion) AGAINST(:termino) DESC
        """;
    
    Query query = entityManager.createNativeQuery(sql, Producto.class);
    query.setParameter("termino", termino);
    
    return query.getResultList();
}

// Consulta nativa que devuelve valores escalares
public List<Object[]> getEstadisticasAvanzadas() {
    String sql = """
        SELECT 
            c.nombre AS categoria,
            COUNT(*) AS total_productos,
            AVG(p.precio) AS precio_promedio,
            STDDEV(p.precio) AS desviacion_estandar,
            PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY p.precio) AS mediana
        FROM productos p
        JOIN categorias c ON p.categoria_id = c.id
        WHERE p.activo = true
        GROUP BY c.nombre
        HAVING COUNT(*) > 5
        ORDER BY total_productos DESC
        """;
    
    return entityManager.createNativeQuery(sql).getResultList();
}

### 9.3. Mapeo de resultados con `@SqlResultSetMapping`

Para mapear resultados a DTOs o combinaciones de entidades:

@Entity
@SqlResultSetMapping(
    name = "ProductoConEstadisticas",
    classes = @ConstructorResult(
        targetClass = ProductoEstadisticasDTO.class,
        columns = {
            @ColumnResult(name = "id", type = Long.class),
            @ColumnResult(name = "nombre", type = String.class),
            @ColumnResult(name = "precio", type = BigDecimal.class),
            @ColumnResult(name = "categoria_nombre", type = String.class),
            @ColumnResult(name = "total_ventas", type = Long.class),
            @ColumnResult(name = "ingresos_totales", type = BigDecimal.class)
        }
    )
)
public class Producto {
    // ...
}

// Uso del mapping
public List<ProductoEstadisticasDTO> getProductosConEstadisticas() {
    String sql = """
        SELECT 
            p.id,
            p.nombre,
            p.precio,
            c.nombre AS categoria_nombre,
            COUNT(dv.id) AS total_ventas,
            COALESCE(SUM(dv.cantidad * dv.precio_unitario), 0) AS ingresos_totales
        FROM productos p
        LEFT JOIN categorias c ON p.categoria_id = c.id
        LEFT JOIN detalle_ventas dv ON p.id = dv.producto_id
        GROUP BY p.id, p.nombre, p.precio, c.nombre
        ORDER BY ingresos_totales DESC
        """;
    
    return entityManager.createNativeQuery(sql, "ProductoConEstadisticas")
                       .getResultList();
}

### 9.4. SQL nativo con Spring Data JPA

@Repository
public interface ProductoRepository extends JpaRepository<Producto, Long> {
    
    // SQL nativo basico
    @Query(value = "SELECT * FROM productos WHERE precio > ?1", nativeQuery = true)
    List<Producto> findByPrecioMayorNativo(BigDecimal precio);
    
    // SQL nativo con parametros nombrados
    @Query(value = "SELECT * FROM productos WHERE nombre LIKE %:nombre%", 
           nativeQuery = true)
    List<Producto> findByNombreContainsNativo(@Param("nombre") String nombre);
    
    // SQL nativo para funciones especificas de MySQL
    @Query(value = """
        SELECT * FROM productos 
        WHERE MATCH(nombre, descripcion) AGAINST(:termino IN BOOLEAN MODE)
        """, nativeQuery = true)
    List<Producto> busquedaFullText(@Param("termino") String termino);
    
    // SQL nativo con funciones de ventana
    @Query(value = """
        SELECT *, 
               RANK() OVER (PARTITION BY categoria_id ORDER BY precio DESC) as ranking
        FROM productos
        WHERE activo = true
        """, nativeQuery = true)
    List<Object[]> findProductosConRanking();
    
    // SQL nativo para paginacion
    @Query(value = "SELECT * FROM productos WHERE activo = true",
           countQuery = "SELECT COUNT(*) FROM productos WHERE activo = true",
           nativeQuery = true)
    Page<Producto> findActivosNativo(Pageable pageable);
}

### 9.5. Operaciones de modificacion nativas

@Repository
public interface ProductoRepository extends JpaRepository<Producto, Long> {
    
    // UPDATE nativo
    @Modifying
    @Query(value = "UPDATE productos SET precio = precio * (1 + :porcentaje/100) " +
                   "WHERE categoria_id = :categoriaId", nativeQuery = true)
    int aplicarIncrementoPrecio(@Param("porcentaje") double porcentaje,
                                @Param("categoriaId") Long categoriaId);
    
    // DELETE nativo
    @Modifying
    @Query(value = "DELETE FROM productos WHERE activo = false " +
                   "AND DATEDIFF(NOW(), fecha_modificacion) > :dias", nativeQuery = true)
    int eliminarInactivosAntiguos(@Param("dias") int dias);
    
    // INSERT nativo (para casos especiales)
    @Modifying
    @Query(value = "INSERT INTO productos_historico " +
                   "SELECT *, NOW() as fecha_archivo FROM productos WHERE id = :id", 
           nativeQuery = true)
    void archivarProducto(@Param("id") Long id);
}

Importante: Las operaciones `@Modifying` requieren `@Transactional` en el servicio:

@Service
@Transactional
public class ProductoService {
    
    @Autowired
    private ProductoRepository productoRepository;
    
    public void actualizarPrecios(double porcentaje, Long categoriaId) {
        int actualizados = productoRepository.aplicarIncrementoPrecio(porcentaje, categoriaId);
        log.info("Se actualizaron {} productos", actualizados);
    }
}

## 10. Comparativa de enfoques de consulta

### 10.1. Tabla comparativa

|Caracteristica|JPQL|Criteria API|Query Methods|SQL Nativo|
|---|---|---|---|---|
|Legibilidad|Alta|Media-Baja|Muy alta|Alta|
|Type-safety|No|Si (con Metamodel)|Si|No|
|Consultas dinamicas|Dificil|Excelente|No|Dificil|
|Curva aprendizaje|Baja|Alta|Muy baja|Baja|
|Portabilidad|Alta|Alta|Alta|Baja|
|Funciones SGBD|Limitado|Limitado|No|Total|
|Rendimiento|Bueno|Bueno|Bueno|Optimo|
|Mantenibilidad|Buena|Buena|Excelente|Media|

### 10.2. Guia de seleccion

**Usar JPQL cuando**:

- La consulta es estatica y conocida en tiempo de compilacion
    
- Se necesita buena legibilidad
    
- La consulta involucra multiples entidades con joins
    
- Se requiere portabilidad entre bases de datos
    

@Query("SELECT p FROM Producto p JOIN FETCH p.categoria WHERE p.activo = true")
List<Producto> findActivosConCategoria();

**Usar Criteria API cuando**:

- La consulta debe construirse dinamicamente
    
- Se necesitan condiciones opcionales (formularios de busqueda)
    
- Se requiere type-safety con Metamodel
    
- Se construyen consultas programaticamente
    

public List<Producto> buscar(FiltroProducto filtro) {
    // Construccion dinamica basada en el filtro
}

**Usar Query Methods cuando**:

- La consulta es simple (1-3 condiciones)
    
- Se sigue el patron de naming de Spring Data
    
- Se valora la legibilidad y simplicidad
    
- No hay logica dinamica
    

List<Producto> findByActivoTrueAndPrecioLessThan(BigDecimal precio);

**Usar SQL Nativo cuando**:

- Se necesitan funciones especificas del SGBD
    
- El rendimiento es critico
    
- Se trabaja con tablas no mapeadas
    
- Se requieren operaciones masivas optimizadas
    

@Query(value = "SELECT * FROM productos WHERE MATCH(...)", nativeQuery = true)
List<Producto> busquedaFullText(String termino);

### 10.3. Combinacion de enfoques

En aplicaciones reales, es comun combinar diferentes enfoques segun las necesidades:

@Repository
public interface ProductoRepository extends JpaRepository<Producto, Long>,
                                           JpaSpecificationExecutor<Producto> {
    
    // Query Methods para consultas simples
    List<Producto> findByActivoTrue();
    Optional<Producto> findBySku(String sku);
    
    // JPQL para consultas con JOIN FETCH
    @Query("SELECT p FROM Producto p JOIN FETCH p.categoria WHERE p.id = :id")
    Optional<Producto> findByIdConCategoria(@Param("id") Long id);
    
    // SQL nativo para funciones especificas
    @Query(value = "SELECT * FROM productos WHERE MATCH(nombre) AGAINST(:q)", 
           nativeQuery = true)
    List<Producto> busquedaFullText(@Param("q") String query);
}

// Criteria API via Specification para busquedas dinamicas
@Service
public class ProductoService {
    
    public Page<Producto> busquedaAvanzada(FiltroProducto filtro, Pageable pageable) {
        Specification<Producto> spec = buildSpecification(filtro);
        return productoRepository.findAll(spec, pageable);
    }
}

## 11. Ejercicio practico guiado

### Sistema de E-commerce con busquedas avanzadas

Vamos a implementar un sistema de busqueda de productos para una tienda online que demuestra todos los tipos de consultas.

**Paso 1**: Modelo de datos

@Entity
@Table(name = "productos")
public class Producto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String nombre;
    
    @Column(columnDefinition = "TEXT")
    private String descripcion;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal precio;
    
    private Integer stock;
    
    private Boolean activo = true;
    
    @Column(name = "fecha_creacion")
    private LocalDateTime fechaCreacion;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "categoria_id")
    private Categoria categoria;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "marca_id")
    private Marca marca;
    
    @ElementCollection
    @CollectionTable(name = "producto_etiquetas")
    private Set<String> etiquetas = new HashSet<>();
    
    // Constructores, getters, setters...
}

@Entity
@Table(name = "categorias")
public class Categoria {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Categoria padre;
    
    @OneToMany(mappedBy = "padre")
    private Set<Categoria> hijas = new HashSet<>();
}

@Entity
@Table(name = "marcas")
public class Marca {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private String pais;
}

**Paso 2**: DTO para filtros

public class FiltroProducto {
    private String nombre;
    private BigDecimal precioMin;
    private BigDecimal precioMax;
    private Long categoriaId;
    private Long marcaId;
    private Boolean soloActivos;
    private Boolean soloConStock;
    private Set<String> etiquetas;
    private String ordenarPor;
    private boolean ordenAscendente;
    
    // Getters, setters, builder...
}

**Paso 3**: Repositorio con multiples enfoques

@Repository
public interface ProductoRepository extends JpaRepository<Producto, Long>,
                                           JpaSpecificationExecutor<Producto> {
    
    // Query Methods simples
    List<Producto> findByActivoTrueOrderByFechaCreacionDesc();
    
    List<Producto> findByCategoriaIdAndActivoTrue(Long categoriaId);
    
    long countByActivoTrueAndStockGreaterThan(Integer stock);
    
    // JPQL para JOIN FETCH
    @Query("SELECT DISTINCT p FROM Producto p " +
           "LEFT JOIN FETCH p.categoria " +
           "LEFT JOIN FETCH p.marca " +
           "WHERE p.activo = true")
    List<Producto> findAllActivosConRelaciones();
    
    @Query("SELECT p FROM Producto p WHERE p.categoria.id = :catId " +
           "AND p.precio BETWEEN :min AND :max ORDER BY p.precio")
    List<Producto> findByCategoriaYRangoPrecio(
        @Param("catId") Long categoriaId,
        @Param("min") BigDecimal precioMin,
        @Param("max") BigDecimal precioMax);
    
    // SQL nativo para Full-Text Search
    @Query(value = """
        SELECT p.* FROM productos p
        WHERE MATCH(p.nombre, p.descripcion) 
        AGAINST(:termino IN NATURAL LANGUAGE MODE)
        AND p.activo = true
        ORDER BY MATCH(p.nombre, p.descripcion) 
        AGAINST(:termino IN NATURAL LANGUAGE MODE) DESC
        LIMIT :limite
        """, nativeQuery = true)
    List<Producto> busquedaFullText(
        @Param("termino") String termino, 
        @Param("limite") int limite);
    
    // SQL nativo para estadisticas
    @Query(value = """
        SELECT 
            c.nombre as categoria,
            COUNT(p.id) as total,
            AVG(p.precio) as precio_promedio,
            SUM(p.stock) as stock_total
        FROM productos p
        JOIN categorias c ON p.categoria_id = c.id
        WHERE p.activo = true
        GROUP BY c.id, c.nombre
        ORDER BY total DESC
        """, nativeQuery = true)
    List<Object[]> getEstadisticasPorCategoria();
}

**Paso 4**: Specifications para busqueda dinamica

public class ProductoSpecifications {
    
    public static Specification<Producto> nombreContiene(String nombre) {
        return (root, query, cb) -> {
            if (nombre == null || nombre.isBlank()) {
                return cb.conjunction();
            }
            return cb.like(cb.lower(root.get("nombre")), 
                          "%" + nombre.toLowerCase() + "%");
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
    
    public static Specification<Producto> enCategoria(Long categoriaId) {
        return (root, query, cb) -> {
            if (categoriaId == null) {
                return cb.conjunction();
            }
            return cb.equal(root.get("categoria").get("id"), categoriaId);
        };
    }
    
    public static Specification<Producto> deMarca(Long marcaId) {
        return (root, query, cb) -> {
            if (marcaId == null) {
                return cb.conjunction();
            }
            return cb.equal(root.get("marca").get("id"), marcaId);
        };
    }
    
    public static Specification<Producto> estaActivo() {
        return (root, query, cb) -> cb.isTrue(root.get("activo"));
    }
    
    public static Specification<Producto> tieneStock() {
        return (root, query, cb) -> cb.greaterThan(root.get("stock"), 0);
    }
    
    public static Specification<Producto> tieneEtiqueta(Set<String> etiquetas) {
        return (root, query, cb) -> {
            if (etiquetas == null || etiquetas.isEmpty()) {
                return cb.conjunction();
            }
            return root.join("etiquetas").in(etiquetas);
        };
    }
    
    public static Specification<Producto> conRelaciones() {
        return (root, query, cb) -> {
            if (Long.class != query.getResultType()) {
                root.fetch("categoria", JoinType.LEFT);
                root.fetch("marca", JoinType.LEFT);
            }
            return cb.conjunction();
        };
    }
}

**Paso 5**: Servicio de busqueda

@Service
@Transactional(readOnly = true)
public class ProductoBusquedaService {
    
    private final ProductoRepository productoRepository;
    private final EntityManager entityManager;
    
    public ProductoBusquedaService(ProductoRepository productoRepository,
                                   EntityManager entityManager) {
        this.productoRepository = productoRepository;
        this.entityManager = entityManager;
    }
    
    /**
     * Busqueda avanzada usando Specifications
     */
    public Page<Producto> busquedaAvanzada(FiltroProducto filtro, Pageable pageable) {
        Specification<Producto> spec = Specification
            .where(ProductoSpecifications.nombreContiene(filtro.getNombre()))
            .and(ProductoSpecifications.precioEntre(
                filtro.getPrecioMin(), filtro.getPrecioMax()))
            .and(ProductoSpecifications.enCategoria(filtro.getCategoriaId()))
            .and(ProductoSpecifications.deMarca(filtro.getMarcaId()))
            .and(ProductoSpecifications.tieneEtiqueta(filtro.getEtiquetas()));
        
        if (Boolean.TRUE.equals(filtro.getSoloActivos())) {
            spec = spec.and(ProductoSpecifications.estaActivo());
        }
        
        if (Boolean.TRUE.equals(filtro.getSoloConStock())) {
            spec = spec.and(ProductoSpecifications.tieneStock());
        }
        
        return productoRepository.findAll(spec, pageable);
    }
    
    /**
     * Busqueda usando Criteria API directamente
     */
    public List<Producto> busquedaConCriteria(FiltroProducto filtro) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Producto> query = cb.createQuery(Producto.class);
        Root<Producto> root = query.from(Producto.class);
        
        // Fetch joins para evitar N+1
        root.fetch("categoria", JoinType.LEFT);
        root.fetch("marca", JoinType.LEFT);
        
        List<Predicate> predicados = new ArrayList<>();
        
        if (filtro.getNombre() != null && !filtro.getNombre().isBlank()) {
            predicados.add(cb.like(
                cb.lower(root.get("nombre")),
                "%" + filtro.getNombre().toLowerCase() + "%"
            ));
        }
        
        if (filtro.getPrecioMin() != null) {
            predicados.add(cb.greaterThanOrEqualTo(
                root.get("precio"), filtro.getPrecioMin()));
        }
        
        if (filtro.getPrecioMax() != null) {
            predicados.add(cb.lessThanOrEqualTo(
                root.get("precio"), filtro.getPrecioMax()));
        }
        
        if (filtro.getCategoriaId() != null) {
            predicados.add(cb.equal(
                root.get("categoria").get("id"), filtro.getCategoriaId()));
        }
        
        if (Boolean.TRUE.equals(filtro.getSoloActivos())) {
            predicados.add(cb.isTrue(root.get("activo")));
        }
        
        if (Boolean.TRUE.equals(filtro.getSoloConStock())) {
            predicados.add(cb.greaterThan(root.get("stock"), 0));
        }
        
        if (!predicados.isEmpty()) {
            query.where(cb.and(predicados.toArray(new Predicate[0])));
        }
        
        // Ordenacion dinamica
        if (filtro.getOrdenarPor() != null) {
            Path<?> ordenPath = root.get(filtro.getOrdenarPor());
            query.orderBy(filtro.isOrdenAscendente() 
                ? cb.asc(ordenPath) 
                : cb.desc(ordenPath));
        }
        
        query.select(root).distinct(true);
        
        return entityManager.createQuery(query).getResultList();
    }
    
    /**
     * Busqueda Full-Text usando SQL nativo
     */
    public List<Producto> busquedaTextoCompleto(String termino, int limite) {
        return productoRepository.busquedaFullText(termino, limite);
    }
    
    /**
     * Estadisticas usando SQL nativo
     */
    public List<EstadisticaCategoriaDTO> getEstadisticasPorCategoria() {
        return productoRepository.getEstadisticasPorCategoria().stream()
            .map(row -> new EstadisticaCategoriaDTO(
                (String) row[0],
                ((Number) row[1]).longValue(),
                (BigDecimal) row[2],
                ((Number) row[3]).intValue()
            ))
            .toList();
    }
}

## 12. Ejercicios propuestos

### Ejercicio 1: Busqueda con ordenacion multiple

Implementa un metodo en Criteria API que permita ordenar productos por multiples criterios (por ejemplo, primero por categoria, luego por precio descendente, luego por nombre).

### Ejercicio 2: Subconsulta de productos destacados

Crea una consulta que devuelva los productos cuyo precio esta por encima del promedio de su categoria.

### Ejercicio 3: Specification reutilizable con JOIN

Implementa una Specification que busque productos por nombre de marca (no ID), requiriendo un JOIN a la tabla de marcas.

### Ejercicio 4: SQL nativo con paginacion

Implementa una consulta nativa que utilice funciones de ventana para asignar un ranking a los productos dentro de cada categoria, con soporte para paginacion.

### Ejercicio 5: Comparativa de rendimiento

Implementa la misma busqueda usando JPQL, Criteria API y SQL nativo. Mide y compara los tiempos de ejecucion con un conjunto grande de datos.

## 13. Resumen y conceptos clave

En este bloque hemos aprendido:

**Criteria API**:

- Es la solucion para consultas dinamicas en JPA
    
- CriteriaBuilder, CriteriaQuery, Root, Predicate son los componentes principales
    
- Permite construir condiciones programaticamente
    
- Con Metamodel se obtiene type-safety completa
    
- Spring Data JPA lo integra via JpaSpecificationExecutor
    

**Consultas nativas SQL**:

- Necesarias para funciones especificas del SGBD
    
- Se usan con `@Query`(nativeQuery = true) o EntityManager.createNativeQuery()
    
- `@SqlResultSetMapping` permite mapear resultados complejos
    
- Requieren `@Modifying` para UPDATE/DELETE
    

**Comparativa de enfoques**:

- Query Methods: simples, legibles, para casos basicos
    
- JPQL: estaticas, legibles, buen balance
    
- Criteria API: dinamicas, type-safe, mas verbosas
    
- SQL Nativo: maximo control, menor portabilidad
    

**Buenas practicas**:

1. Usar Query Methods para consultas simples
    
2. JPQL para consultas estaticas con joins
    
3. Criteria API / Specification para busquedas dinamicas
    
4. SQL nativo solo cuando sea estrictamente necesario
    
5. Siempre usar parametros nombrados o posicionales (nunca concatenar)
    
6. Preferir fetch joins sobre consultas adicionales
    

## 14. Referencias y recursos adicionales

**Documentacion oficial**:

- Jakarta Persistence Specification (JPA 3.1)
    
- Hibernate ORM Documentation - Criteria API
    
- Spring Data JPA Reference - Specifications
    

**Libros recomendados**:

- «Pro JPA 2 in Java EE 8» - Mike Keith
    
- «High-Performance Java Persistence» - Vlad Mihalcea
    

**Articulos utiles**:

- «JPA Criteria API vs JPQL vs Native SQL» - Baeldung
    
- «Dynamic Queries with Spring Data JPA Specifications» - [Spring.io](http://spring.io/)