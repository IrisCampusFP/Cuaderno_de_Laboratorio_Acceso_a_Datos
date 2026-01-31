## Objetivos

- Comprender las diferencias entre SQL y JPQL
    
- Dominar la sintaxis basica de JPQL
    
- Utilizar funciones de agregacion y manipulacion de cadenas
    
- Aplicar JOINs y subconsultas
    
- Optimizar consultas con FETCH JOIN y proyecciones

## 1. Introduccion a JPQL

### 1.1. Que es JPQL

JPQL (Java Persistence Query Language) es el lenguaje de consulta definido por la especificacion JPA (Jakarta Persistence API) para realizar consultas sobre entidades persistentes.

A diferencia de SQL, que opera directamente sobre tablas y columnas, **JPQL trabaja con objetos Java y sus atributos**.

### 1.2. Diferencias entre SQL y JPQL

|Aspecto|SQL|JPQL|
|---|---|---|
|Opera sobre|Tablas y columnas|Entidades y atributos|
|Nombre de origen|Nombre de la tabla|Nombre de la clase Java|
|Campos|Nombres de columnas|Nombres de atributos Java|
|Relaciones|JOINs con claves foraneas|Navegacion por atributos|
|Resultado|Filas y columnas|Objetos o proyecciones|
|Portabilidad|Especifico del SGBD|Independiente del SGBD|

### 1.3. Ejemplo Comparativo

**Entidad Java:**

@Entity
@Table(name = "productos")
public class Producto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "nombre_producto")
    private String nombre;
    
    private Double precio;
    
    @ManyToOne
    @JoinColumn(name = "categoria_id")
    private Categoria categoria;
}

**Consulta SQL:**

SELECT * FROM productos
WHERE nombre_producto LIKE '%laptop%' AND precio < 1000;

**Consulta JPQL equivalente:**

SELECT p FROM Producto p
WHERE p.nombre LIKE '%laptop%' AND p.precio < 1000

Observa las diferencias:

- Usamos el **nombre de la clase** (Producto), no el nombre de la tabla (productos)
    
- Usamos el **nombre del atributo** (nombre), no el nombre de la columna (nombre_producto)
    
- Definimos un **alias obligatorio** (p) para la entidad

## 2. Sintaxis Basica de JPQL

### 2.1. Estructura General

SELECT [DISTINCT] expresion_seleccion
FROM entidad [alias]
[WHERE condicion]
[GROUP BY expresion_agrupacion]
[HAVING condicion_grupo]
[ORDER BY expresion_ordenacion [ASC|DESC]]

### 2.2. Tipos de Seleccion

// Entidad completa
SELECT p FROM Producto p

// Atributo unico
SELECT p.nombre FROM Producto p

// Multiples atributos (proyeccion)
SELECT p.nombre, p.precio FROM Producto p

// Constructor DTO (proyeccion tipada)
SELECT NEW com.ejemplo.dto.ProductoDTO(p.nombre, p.precio) FROM Producto p

// Con DISTINCT
SELECT DISTINCT p.categoria FROM Producto p

## 3. Clausulas Principales

### 3.1. Clausula WHERE

#### Operadores de Comparacion

|Operador|Descripcion|Ejemplo|
|---|---|---|
|=|Igualdad|`p.precio = 100.0`|
|<> o !=|Desigualdad|`p.precio <> 100.0`|
|>|Mayor que|`p.precio > 50.0`|
|>=|Mayor o igual|`p.precio >= 50.0`|
|<|Menor que|`p.precio < 200.0`|
|<=|Menor o igual|`p.precio <= 200.0`|

#### Operadores Logicos

// AND - ambas condiciones deben cumplirse
SELECT p FROM Producto p WHERE p.precio > 50 AND p.stock > 0

// OR - al menos una condicion debe cumplirse
SELECT p FROM Producto p WHERE p.categoria.nombre = 'Electronica'
   OR p.categoria.nombre = 'Informatica'

// NOT - niega la condicion
SELECT p FROM Producto p WHERE NOT p.activo = true

#### Operadores Especiales

// BETWEEN
SELECT p FROM Producto p WHERE p.precio BETWEEN 100 AND 500

// IN
SELECT p FROM Producto p WHERE p.categoria.nombre IN ('Electronica', 'Informatica')

// LIKE (% = cualquier secuencia, _ = un caracter)
SELECT p FROM Producto p WHERE p.nombre LIKE 'Laptop%'
SELECT p FROM Producto p WHERE p.nombre LIKE '%Gaming%'

// IS NULL / IS NOT NULL
SELECT p FROM Producto p WHERE p.descripcion IS NULL
SELECT p FROM Producto p WHERE p.categoria IS NOT NULL

### 3.2. ORDER BY y GROUP BY

// Ordenacion
SELECT p FROM Producto p ORDER BY p.nombre ASC
SELECT p FROM Producto p ORDER BY p.precio DESC
SELECT p FROM Producto p ORDER BY p.categoria.nombre ASC, p.precio DESC

// Agrupacion
SELECT p.categoria.nombre, COUNT(p), AVG(p.precio)
FROM Producto p
GROUP BY p.categoria.nombre

// HAVING - filtrar grupos
SELECT p.categoria.nombre, COUNT(p) as total
FROM Producto p
GROUP BY p.categoria.nombre
HAVING COUNT(p) > 5

## 4. Funciones JPQL

### 4.1. Funciones de Agregacion

|Funcion|Descripcion|Ejemplo|
|---|---|---|
|COUNT|Cuenta elementos|`COUNT(p)`|
|SUM|Suma valores|`SUM(p.precio)`|
|AVG|Promedio|`AVG(p.precio)`|
|MAX|Valor maximo|`MAX(p.precio)`|
|MIN|Valor minimo|`MIN(p.precio)`|

// Estadisticas por categoria
SELECT p.categoria.nombre, COUNT(p), MIN(p.precio), MAX(p.precio), AVG(p.precio)
FROM Producto p
GROUP BY p.categoria.nombre

### 4.2. Funciones de Cadena

|Funcion|Descripcion|
|---|---|
|CONCAT|Concatena cadenas|
|SUBSTRING|Extrae subcadena|
|TRIM|Elimina espacios|
|LOWER|Convierte a minusculas|
|UPPER|Convierte a mayusculas|
|LENGTH|Longitud de la cadena|

// Buscar sin distinguir mayusculas
SELECT p FROM Producto p WHERE LOWER(p.nombre) LIKE '%laptop%'

### 4.3. Funcion CASE

SELECT p.nombre,
       CASE
           WHEN p.precio < 100 THEN 'Economico'
           WHEN p.precio < 500 THEN 'Medio'
           ELSE 'Premium'
       END as rango
FROM Producto p

## 5. Consultas con JOIN

### 5.1. Tipos de JOIN

// INNER JOIN - solo entidades con correspondencia
SELECT p FROM Producto p JOIN p.categoria c
WHERE c.nombre = 'Electronica'

// LEFT JOIN - todas las del lado izquierdo
SELECT c, p FROM Categoria c LEFT JOIN c.productos p

// FETCH JOIN - carga relaciones en una sola consulta
SELECT p FROM Producto p JOIN FETCH p.categoria
SELECT c FROM Categoria c LEFT JOIN FETCH c.productos

### 5.2. Navegacion Implicita vs JOIN Explicito

// Navegacion implicita (Hibernate genera el JOIN)
SELECT p FROM Producto p WHERE p.categoria.nombre = 'Electronica'

// JOIN explicito (control total)
SELECT p FROM Producto p JOIN p.categoria c
WHERE c.nombre = 'Electronica'

## 6. Subconsultas

// Productos con precio superior al promedio
SELECT p FROM Producto p
WHERE p.precio > (SELECT AVG(p2.precio) FROM Producto p2)

// EXISTS
SELECT c FROM Categoria c
WHERE EXISTS (SELECT p FROM Producto p WHERE p.categoria = c)

// NOT EXISTS
SELECT c FROM Categoria c
WHERE NOT EXISTS (SELECT p FROM Producto p WHERE p.categoria = c)

// Subconsulta correlacionada
SELECT p FROM Producto p
WHERE p.precio > (
    SELECT AVG(p2.precio) FROM Producto p2
    WHERE p2.categoria = p.categoria
)

## 7. Consultas de Modificacion

### 7.1. UPDATE

// Aumentar precio un 10%
UPDATE Producto p SET p.precio = p.precio * 1.10

// Actualizar multiples campos
UPDATE Producto p
SET p.precio = p.precio * 0.9, p.oferta = true
WHERE p.categoria.nombre = 'Liquidacion'

### 7.2. DELETE

// Eliminar productos inactivos
DELETE FROM Producto p WHERE p.activo = false

Advertencia

Las consultas UPDATE y DELETE:

- No activan callbacks del ciclo de vida (@PreUpdate, @PreRemove)
    
- No aplican automaticamente las operaciones en cascada
    
- No sincronizan el contexto de persistencia

## 8. Parametros en JPQL

### 8.1. Parametros Nombrados (Recomendado)

SELECT p FROM Producto p
WHERE p.precio > :precioMinimo AND p.categoria.nombre = :categoria

### 8.2. Parametros Posicionales

SELECT p FROM Producto p WHERE p.precio > ?1 AND p.categoria.nombre = ?2

### 8.3. Uso con EntityManager

TypedQuery<Producto> query = em.createQuery(
    "SELECT p FROM Producto p WHERE p.precio > :precio",
    Producto.class
);
query.setParameter("precio", 100.0);
List<Producto> productos = query.getResultList();

### 8.4. Paginacion

TypedQuery<Producto> query = em.createQuery(
    "SELECT p FROM Producto p ORDER BY p.nombre", Producto.class);
query.setFirstResult(0);    // Posicion inicial (0-based)
query.setMaxResults(10);    // Numero maximo de resultados
List<Producto> pagina1 = query.getResultList();

## 9. Named Queries

Las Named Queries son consultas JPQL predefinidas que se declaran en la entidad:

@Entity
@NamedQueries({
    @NamedQuery(
        name = "Producto.findAll",
        query = "SELECT p FROM Producto p ORDER BY p.nombre"
    ),
    @NamedQuery(
        name = "Producto.findByCategoria",
        query = "SELECT p FROM Producto p WHERE p.categoria.nombre = :categoria"
    )
})
public class Producto { ... }

**Ejecucion:**

TypedQuery<Producto> query = em.createNamedQuery(
    "Producto.findByCategoria", Producto.class);
query.setParameter("categoria", "Electronica");
List<Producto> productos = query.getResultList();

## 10. Spring Data JPA con `@Query`

Spring Data JPA simplifica la ejecucion de consultas JPQL:

public interface ProductoRepository extends JpaRepository<Producto, Long> {
    
    // JPQL con parametro nombrado
    @Query("SELECT p FROM Producto p WHERE p.categoria.nombre = :cat")
    List<Producto> findByCategoriaNombre(@Param("cat") String categoria);
    
    // JPQL con parametro posicional
    @Query("SELECT p FROM Producto p WHERE p.precio > ?1")
    List<Producto> findByPrecioMayorQue(Double precio);
    
    // UPDATE con @Modifying
    @Modifying
    @Query("UPDATE Producto p SET p.precio = p.precio * :factor")
    int actualizarPrecios(@Param("factor") Double factor);
}

## 11. Query Methods de Spring Data JPA

Spring Data JPA genera consultas automaticamente a partir del nombre del metodo:

|Palabra Clave|JPQL Generado|
|---|---|
|And|`WHERE x.a = ?1 AND x.b = ?2`|
|Or|`WHERE x.a = ?1 OR x.b = ?2`|
|Between|`WHERE x.a BETWEEN ?1 AND ?2`|
|LessThan|`WHERE x.a < ?1`|
|GreaterThan|`WHERE x.a > ?1`|
|IsNull|`WHERE x.a IS NULL`|
|Like|`WHERE x.a LIKE ?1`|
|Containing|`WHERE x.a LIKE %?1%`|
|OrderBy|`ORDER BY x.a ASC/DESC`|

public interface ProductoRepository extends JpaRepository<Producto, Long> {
    
    List<Producto> findByNombre(String nombre);
    List<Producto> findByPrecioGreaterThan(Double precio);
    List<Producto> findByPrecioBetween(Double min, Double max);
    List<Producto> findByNombreContainingIgnoreCase(String texto);
    List<Producto> findByCategoriaNombre(String nombreCategoria);
    List<Producto> findByActivoTrueOrderByNombreAsc();
    Long countByCategoriaNombre(String categoria);
    Boolean existsByNombre(String nombre);
    Optional<Producto> findFirstByOrderByPrecioDesc();
    List<Producto> findTop5ByOrderByPrecioDesc();
}

## 12. Optimizacion de Consultas

### 12.1. El Problema N+1

// Problema N+1
List<Producto> productos = productoRepository.findAll();  // 1 consulta
for (Producto p : productos) {
    System.out.println(p.getCategoria().getNombre());  // N consultas
}

### 12.2. Solucion con JOIN FETCH

@Query("SELECT p FROM Producto p JOIN FETCH p.categoria")
List<Producto> findAllConCategoria();

### 12.3. EntityGraph

@Entity
@NamedEntityGraph(
    name = "Producto.conCategoria",
    attributeNodes = @NamedAttributeNode("categoria")
)
public class Producto { ... }

// En el repositorio
@EntityGraph(value = "Producto.conCategoria")
List<Producto> findByActivo(Boolean activo);

// EntityGraph ad-hoc
@EntityGraph(attributePaths = {"categoria", "proveedor"})
List<Producto> findByPrecioGreaterThan(Double precio);

## 13. Tabla de Referencia Rapida

|Operacion|Sintaxis JPQL|
|---|---|
|Seleccion basica|`SELECT e FROM Entidad e`|
|Filtrado|`WHERE e.atributo = :valor`|
|Ordenacion|`ORDER BY e.atributo ASC/DESC`|
|Agrupacion|`GROUP BY e.atributo`|
|Filtro de grupos|`HAVING COUNT(e) > 5`|
|JOIN|`JOIN e.relacion r`|
|LEFT JOIN|`LEFT JOIN e.relacion r`|
|FETCH JOIN|`JOIN FETCH e.relacion`|
|Subconsulta|`WHERE e.attr > (SELECT ...)`|
|Actualizacion|`UPDATE Entidad e SET e.attr = :val`|
|Eliminacion|`DELETE FROM Entidad e WHERE ...`|