## CE 4e: Desarrollar aplicaciones que realizan consultas

En este tema aprenderemos a realizar consultas sobre tipos objeto en Oracle, PostgreSQL y con JPA/JPQL.

## 5.1 Consultas Básicas en Oracle

### Acceso a Atributos de Objetos

-- Consulta básica sobre tabla de objetos
SELECT p.codigo, p.nombre, p.precio
FROM productos p
WHERE p.precio > 500;

-- Llamar a métodos del tipo
SELECT p.nombre, 
       p.precio,
       p.stock,
       p.calcular_valor_stock() AS valor_total
FROM productos p
WHERE p.stock > 0;

### Funciones VALUE() y REF()

-- VALUE(): Obtener el objeto completo
SELECT VALUE(p) 
FROM productos p
WHERE p.codigo = 1;

-- REF(): Obtener referencia al objeto
SELECT REF(p) AS referencia
FROM productos p
WHERE p.codigo = 1;

-- DEREF(): Resolver una referencia
SELECT DEREF(referencia_producto)
FROM lineas_pedido;

### Consultas con Objetos Anidados en Oracle

-- Acceso a objetos anidados con notación punto
SELECT c.nombre, 
       c.direccion.calle,
       c.direccion.ciudad,
       c.direccion.codigo_postal
FROM clientes c
WHERE c.direccion.ciudad = 'Madrid';

-- Filtrar por atributo anidado
SELECT c.nombre, c.email
FROM clientes c
WHERE c.direccion.pais = 'España'
  AND c.direccion.codigo_postal LIKE '28%';

### Consultas con Colecciones en Oracle

-- Desanidar VARRAY con TABLE()
SELECT c.nombre, t.tipo, t.numero
FROM clientes_v2 c, TABLE(c.telefonos) t;

-- Filtrar por elemento de colección
SELECT c.nombre
FROM clientes_v2 c, TABLE(c.telefonos) t
WHERE t.tipo = 'móvil';

-- Contar elementos en colección
SELECT c.nombre, 
       CARDINALITY(c.telefonos) AS num_telefonos
FROM clientes_v2 c;

-- Verificar si colección contiene elemento
SELECT c.nombre
FROM clientes_v2 c
WHERE EXISTS (
    SELECT 1 FROM TABLE(c.telefonos) t 
    WHERE t.numero = '666111222'
);

### Consultas Polimórficas en Oracle

-- Consultar todos los tipos de recurso
SELECT r.id, r.titulo, r.anio
FROM recursos r;

-- Filtrar por tipo con IS OF
SELECT r.titulo
FROM recursos r
WHERE VALUE(r) IS OF (tipo_libro);

-- Casting con TREAT para acceder a atributos específicos
SELECT r.titulo,
       TREAT(VALUE(r) AS tipo_libro).autor,
       TREAT(VALUE(r) AS tipo_libro).isbn
FROM recursos r
WHERE VALUE(r) IS OF (tipo_libro);

-- Solo tipo específico (excluyendo subtipos)
SELECT r.titulo
FROM recursos r
WHERE VALUE(r) IS OF (ONLY tipo_libro);

## 5.2 Consultas en PostgreSQL

### Acceso a Tipos Compuestos

-- Acceso a campos de tipo compuesto (requiere paréntesis)
SELECT (datos).codigo, (datos).nombre, (datos).precio
FROM productos
WHERE (datos).precio > 500;

-- Llamar a función
SELECT (datos).nombre, calcular_valor_stock(datos) AS valor_total
FROM productos
WHERE (datos).stock > 0;

### Consultas con Tipos Anidados en PostgreSQL

-- Acceso a campos anidados
SELECT nombre, 
       (direccion).calle,
       (direccion).ciudad,
       (direccion).codigo_postal
FROM clientes
WHERE (direccion).ciudad = 'Madrid';

-- Filtrar por atributo anidado
SELECT nombre, email
FROM clientes
WHERE (direccion).pais = 'España'
  AND (direccion).codigo_postal LIKE '28%';

### Operaciones con Arrays en PostgreSQL

-- Acceso a elementos de array (1-indexed)
SELECT nombre, 
       telefonos[1] AS primer_telefono,
       array_length(telefonos, 1) AS num_telefonos
FROM clientes;

-- Verificar si array contiene elemento
SELECT nombre
FROM clientes
WHERE '666111222' = ANY(telefonos);

-- Desanidar array con unnest()
SELECT nombre, unnest(telefonos) AS telefono
FROM clientes;

-- Array contiene todos los elementos
SELECT nombre
FROM clientes
WHERE telefonos @> ARRAY['666111222', '911234567'];

-- Array intersecta con otro
SELECT nombre
FROM clientes
WHERE telefonos && ARRAY['666111222', '999888777'];

### Consultas sobre Tablas Heredadas en PostgreSQL

-- Consulta polimórfica (incluye tablas hijas)
SELECT titulo, anio FROM recursos;

-- Solo tabla padre (excluye hijas)
SELECT titulo, anio FROM ONLY recursos;

-- Identificar tabla origen de cada fila
SELECT titulo,
       tableoid::regclass AS tabla_origen
FROM recursos;

-- Con discriminador textual
SELECT titulo,
       CASE tableoid::regclass::text
           WHEN 'libros' THEN 'Libro'
           WHEN 'dvds' THEN 'DVD'
           ELSE 'Recurso'
       END AS tipo
FROM recursos;

-- Consultar tabla hija con todos sus campos
SELECT titulo, autor, isbn, paginas
FROM libros
WHERE autor LIKE '%Cervantes%';

## 5.3 Consultas JPQL

JPQL (Java Persistence Query Language) es el lenguaje de consultas orientado a objetos de JPA.

### Consultas Básicas

// Consulta básica
TypedQuery<Producto> query = em.createQuery(
    "SELECT p FROM Producto p WHERE p.precio > :precioMin", 
    Producto.class);
query.setParameter("precioMin", new BigDecimal("500"));
List<Producto> productos = query.getResultList();

// Con ordenación
TypedQuery<Producto> query = em.createQuery(
    "SELECT p FROM Producto p ORDER BY p.precio DESC", 
    Producto.class);

### Consultas con Objetos Embebidos

// Navegar a través de objetos embebidos
TypedQuery<Cliente> query = em.createQuery(
    "SELECT c FROM Cliente c WHERE c.direccion.ciudad = :ciudad",
    Cliente.class);
query.setParameter("ciudad", "Madrid");
List<Cliente> clientes = query.getResultList();

// Proyección de campos embebidos
TypedQuery<Object[]> query = em.createQuery(
    "SELECT c.nombre, c.direccion.ciudad, c.direccion.pais " +
    "FROM Cliente c",
    Object[].class);

### Consultas con Colecciones

// Verificar pertenencia a colección con MEMBER OF
TypedQuery<Cliente> query = em.createQuery(
    "SELECT c FROM Cliente c WHERE :telefono MEMBER OF c.telefonos",
    Cliente.class);
query.setParameter("telefono", "666111222");

// Contar elementos en colección
TypedQuery<Object[]> query = em.createQuery(
    "SELECT c.nombre, SIZE(c.telefonos) FROM Cliente c",
    Object[].class);

// JOIN con colecciones
TypedQuery<Object[]> query = em.createQuery(
    "SELECT c.nombre, t FROM Cliente c JOIN c.telefonos t",
    Object[].class);

### Consultas Polimórficas en JPA

// Consulta polimórfica (devuelve todos los subtipos)
TypedQuery<Recurso> query = em.createQuery(
    "SELECT r FROM Recurso r", 
    Recurso.class);
List<Recurso> recursos = query.getResultList();

// Filtrar por tipo con TYPE()
TypedQuery<Recurso> query = em.createQuery(
    "SELECT r FROM Recurso r WHERE TYPE(r) = Libro", 
    Recurso.class);

// Casting con TREAT
TypedQuery<String> query = em.createQuery(
    "SELECT TREAT(r AS Libro).autor FROM Recurso r WHERE TYPE(r) = Libro",
    String.class);

// Consultar directamente la subclase
TypedQuery<Libro> query = em.createQuery(
    "SELECT l FROM Libro l WHERE l.autor LIKE :autor",
    Libro.class);
query.setParameter("autor", "%Cervantes%");

### JOIN FETCH para Evitar N+1

// Cargar relaciones en una sola consulta
TypedQuery<Pedido> query = em.createQuery(
    "SELECT DISTINCT p FROM Pedido p " +
    "JOIN FETCH p.lineas l " +
    "JOIN FETCH l.producto " +
    "WHERE p.cliente.id = :clienteId",
    Pedido.class);
query.setParameter("clienteId", 1L);
List<Pedido> pedidos = query.getResultList();

## 5.4 Criteria API

Criteria API permite construir consultas de forma programática:

CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Cliente> cq = cb.createQuery(Cliente.class);
Root<Cliente> cliente = cq.from(Cliente.class);

// WHERE c.direccion.ciudad = 'Madrid'
cq.where(cb.equal(
    cliente.get("direccion").get("ciudad"), 
    "Madrid"
));

// ORDER BY c.nombre
cq.orderBy(cb.asc(cliente.get("nombre")));

List<Cliente> resultado = em.createQuery(cq).getResultList();

## 5.5 Consultas Nativas

Para usar SQL específico del SGBD:

// Oracle: Llamar a método de tipo
Query query = em.createNativeQuery(
    "SELECT p.nombre, p.calcular_valor_stock() FROM productos p");
List<Object[]> resultados = query.getResultList();

// PostgreSQL: Usar funciones de array
Query query = em.createNativeQuery(
    "SELECT nombre, array_length(telefonos, 1) FROM clientes");

// Mapear resultado a entidad
Query query = em.createNativeQuery(
    "SELECT * FROM productos WHERE precio > :min",
    Producto.class);
query.setParameter("min", 500);
List<Producto> productos = query.getResultList();

## 5.6 Resumen del Tema

Conceptos Clave

1. **Oracle**: Usa notación punto para acceder a objetos anidados (`c.direccion.ciudad`).
    
2. **Oracle**: `VALUE()` obtiene objeto completo, `TABLE()` desanida colecciones.
    
3. **PostgreSQL**: Requiere paréntesis para campos de tipos compuestos `(direccion).ciudad`.
    
4. **PostgreSQL**: `unnest()` desanida arrays, `ANY()` verifica pertenencia.
    
5. **JPQL**: Navega objetos naturalmente, usa `TYPE()` para filtrar por tipo.
    
6. **JOIN FETCH** evita el problema N+1 cargando relaciones en una consulta.