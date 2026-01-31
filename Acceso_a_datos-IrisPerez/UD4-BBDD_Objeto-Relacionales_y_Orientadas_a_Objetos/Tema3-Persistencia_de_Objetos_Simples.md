## CE 4c: Gestionar la persistencia de objetos simples

En este tema aprenderemos a crear y persistir objetos simples usando tipos objeto en Oracle, PostgreSQL y JPA.

## 3.1 Concepto de Objeto Simple

Un **objeto simple** es aquel que:

- Contiene solo atributos de **tipos primitivos** o **wrappers**
    
- No tiene **relaciones** con otros objetos persistentes
    
- No contiene **colecciones** de otros objetos

### Tipos de atributos en objetos simples

|Tipo Java|Oracle|PostgreSQL|
|---|---|---|
|`String`|`VARCHAR2(n)`|`VARCHAR(n)`|
|`Integer/int`|`NUMBER`|`INTEGER`|
|`Long/long`|`NUMBER`|`BIGINT`|
|`Double/double`|`NUMBER(p,s)`|`NUMERIC(p,s)`|
|`BigDecimal`|`NUMBER(p,s)`|`NUMERIC(p,s)`|
|`Date`|`DATE`|`DATE`|
|`LocalDate`|`DATE`|`DATE`|
|`Boolean`|`CHAR(1)`|`BOOLEAN`|

## 3.2 Tipos Objeto en Oracle

Oracle permite crear **tipos objeto** con `CREATE TYPE ... AS OBJECT`:

### Sintaxis básica

CREATE OR REPLACE TYPE nombre_tipo AS OBJECT (
    atributo1   TIPO1,
    atributo2   TIPO2,
    ...
    -- Métodos opcionales
    MEMBER FUNCTION nombre_metodo RETURN tipo
);
/

### Ejemplo: Tipo Producto en Oracle

-- Crear tipo objeto simple
CREATE OR REPLACE TYPE tipo_producto AS OBJECT (
    codigo      NUMBER,
    nombre      VARCHAR2(100),
    precio      NUMBER(10,2),
    stock       NUMBER,
    fecha_alta  DATE,
    
    -- Método para calcular valor del stock
    MEMBER FUNCTION calcular_valor_stock RETURN NUMBER
);
/

-- Implementar el cuerpo del tipo
CREATE OR REPLACE TYPE BODY tipo_producto AS
    MEMBER FUNCTION calcular_valor_stock RETURN NUMBER IS
    BEGIN
        RETURN SELF.precio * SELF.stock;
    END;
END;
/

### Crear Tabla de Objetos en Oracle

-- Tabla de objetos (cada fila es un objeto)
CREATE TABLE productos OF tipo_producto (
    codigo PRIMARY KEY,
    nombre NOT NULL
);

-- Alternativa: tabla relacional con columna objeto
CREATE TABLE inventario (
    id          NUMBER PRIMARY KEY,
    producto    tipo_producto,
    ubicacion   VARCHAR2(50)
);

### Operaciones CRUD en Oracle

-- INSERT: Usar constructor del tipo
INSERT INTO productos VALUES (
    tipo_producto(1, 'Laptop HP', 899.99, 50, SYSDATE)
);

INSERT INTO productos VALUES (
    tipo_producto(2, 'Monitor Dell', 299.99, 30, SYSDATE)
);

-- SELECT: Acceso a atributos con notación punto
SELECT p.codigo, p.nombre, p.precio
FROM productos p
WHERE p.precio > 500;

-- SELECT: Llamar a métodos
SELECT p.nombre, p.calcular_valor_stock() AS valor_total
FROM productos p;

-- UPDATE
UPDATE productos p
SET p.precio = p.precio * 1.10
WHERE p.codigo = 1;

-- DELETE
DELETE FROM productos p
WHERE p.stock = 0;

## 3.3 Tipos Compuestos en PostgreSQL

PostgreSQL soporta **tipos compuestos** (composite types):

### Sintaxis básica

CREATE TYPE nombre_tipo AS (
    atributo1   TIPO1,
    atributo2   TIPO2,
    ...
);

Advertencia

A diferencia de Oracle, PostgreSQL **no permite métodos** en tipos compuestos. Para lógica de negocio, usar funciones SQL.

### Ejemplo: Tipo Producto en PostgreSQL

-- Crear tipo compuesto
CREATE TYPE tipo_producto AS (
    codigo      INTEGER,
    nombre      VARCHAR(100),
    precio      NUMERIC(10,2),
    stock       INTEGER,
    fecha_alta  DATE
);

-- Tabla con columna de tipo compuesto
CREATE TABLE productos (
    id SERIAL PRIMARY KEY,
    datos tipo_producto
);

-- Función equivalente al método de Oracle
CREATE OR REPLACE FUNCTION calcular_valor_stock(p tipo_producto)
RETURNS NUMERIC AS $$
BEGIN
    RETURN p.precio * p.stock;
END;
$$ LANGUAGE plpgsql;

### Operaciones CRUD en PostgreSQL

-- INSERT: Usar ROW() y casting
INSERT INTO productos (datos) VALUES (
    ROW(1, 'Laptop HP', 899.99, 50, CURRENT_DATE)::tipo_producto
);

INSERT INTO productos (datos) VALUES (
    ROW(2, 'Monitor Dell', 299.99, 30, CURRENT_DATE)::tipo_producto
);

-- SELECT: Acceso a campos con paréntesis
SELECT (datos).codigo, (datos).nombre, (datos).precio
FROM productos
WHERE (datos).precio > 500;

-- SELECT: Llamar a función
SELECT (datos).nombre, calcular_valor_stock(datos) AS valor_total
FROM productos;

-- UPDATE: Campo individual
UPDATE productos
SET datos.precio = (datos).precio * 1.10
WHERE (datos).codigo = 1;

-- UPDATE: Objeto completo
UPDATE productos
SET datos = ROW(1, 'Laptop HP Pro', 989.99, 45, CURRENT_DATE)::tipo_producto
WHERE id = 1;

-- DELETE
DELETE FROM productos
WHERE (datos).stock = 0;

## 3.4 Tipos Enumerados

Tanto Oracle como PostgreSQL soportan enumeraciones:

### PostgreSQL ENUM

-- Crear tipo enumerado
CREATE TYPE estado_producto AS ENUM (
    'DISPONIBLE', 'AGOTADO', 'DESCATALOGADO'
);

-- Usar en tabla
CREATE TABLE productos_v2 (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    estado estado_producto DEFAULT 'DISPONIBLE'
);

INSERT INTO productos_v2 (nombre, estado) 
VALUES ('Laptop', 'DISPONIBLE');

### Oracle (simulado con CHECK)

-- Oracle no tiene ENUM nativo, se simula con CHECK
CREATE TABLE productos_v2 (
    id NUMBER PRIMARY KEY,
    nombre VARCHAR2(100),
    estado VARCHAR2(20) DEFAULT 'DISPONIBLE'
        CHECK (estado IN ('DISPONIBLE', 'AGOTADO', 'DESCATALOGADO'))
);

## 3.5 Persistencia JPA para Objetos Simples

Con JPA, los objetos simples se mapean con anotaciones estándar:

import jakarta.persistence.*;
import java.math.BigDecimal;
import java.time.LocalDate;

@Entity
@Table(name = "productos")
public class Producto {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(length = 100, nullable = false)
    private String nombre;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal precio;
    
    private Integer stock;
    
    @Column(name = "fecha_alta")
    private LocalDate fechaAlta;
    
    @Enumerated(EnumType.STRING)
    private EstadoProducto estado;
    
    // Constructores
    public Producto() {}
    
    public Producto(String nombre, BigDecimal precio, Integer stock) {
        this.nombre = nombre;
        this.precio = precio;
        this.stock = stock;
        this.fechaAlta = LocalDate.now();
        this.estado = EstadoProducto.DISPONIBLE;
    }
    
    // Método de negocio (equivalente al de Oracle)
    public BigDecimal calcularValorStock() {
        return precio.multiply(BigDecimal.valueOf(stock));
    }
    
    // Getters y Setters
    // ...
}

public enum EstadoProducto {
    DISPONIBLE, AGOTADO, DESCATALOGADO
}

## 3.6 Operaciones CRUD con JPA

### Persistir (INSERT)

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

Producto p = new Producto("Laptop HP", new BigDecimal("899.99"), 50);
em.persist(p);

em.getTransaction().commit();
em.close();

System.out.println("ID asignado: " + p.getId());

### Recuperar (SELECT)

// Por ID
Producto p = em.find(Producto.class, 1L);

// Con JPQL
TypedQuery<Producto> query = em.createQuery(
    "SELECT p FROM Producto p WHERE p.precio > :min", Producto.class);
query.setParameter("min", new BigDecimal("500"));
List<Producto> productos = query.getResultList();

### Actualizar (UPDATE)

em.getTransaction().begin();

Producto p = em.find(Producto.class, 1L);
p.setPrecio(p.getPrecio().multiply(new BigDecimal("1.10"))); // +10%
// Los cambios se detectan automáticamente (dirty checking)

em.getTransaction().commit();

### Eliminar (DELETE)

em.getTransaction().begin();

Producto p = em.find(Producto.class, 1L);
em.remove(p);

em.getTransaction().commit();

## 3.7 Comparativa de Sintaxis

|Operación|Oracle|PostgreSQL|JPA|
|---|---|---|---|
|**Crear tipo**|`CREATE TYPE x AS OBJECT`|`CREATE TYPE x AS`|Clase Java|
|**Insertar**|`tipo(val1, val2)`|`ROW(val1, val2)::tipo`|`em.persist(obj)`|
|**Acceso atributo**|`p.nombre`|`(datos).nombre`|`obj.getNombre()`|
|**Llamar método**|`p.metodo()`|Función SQL|`obj.metodo()`|
|**Actualizar**|`SET p.attr = val`|`SET datos.attr = val`|Setter en transacción|

## 3.8 Resumen del Tema

Conceptos Clave

1. Los **objetos simples** contienen solo tipos primitivos, sin relaciones.
    
2. **Oracle** usa `CREATE TYPE AS OBJECT` con soporte para metodos.
    
3. **PostgreSQL** usa `CREATE TYPE AS` (sin metodos, usar funciones).
    
4. En Oracle se accede a atributos con `p.nombre`, en PostgreSQL con `(datos).nombre`.
    
5. **JPA** permite la misma logica con clases Java y anotaciones.
    
6. Las operaciones **CRUD** siguen patrones similares en los tres enfoques.