## CE 4d: Gestionar la persistencia de objetos estructurados

En este tema aprenderemos a trabajar con objetos que contienen otros objetos, colecciones y herencia.

## 4.1 Concepto de Objeto Estructurado

Un **objeto estructurado** es aquel que:

- Contiene **referencias a otros objetos** (composición)
    
- Incluye **objetos embebidos** (sin identidad propia)
    
- Tiene **colecciones** de valores u objetos
    
- Participa en **jerarquías de herencia**

### Ejemplos de estructuras

Cliente
├── direccion: Direccion (embebido)
├── telefonos: List<String> (colección)
└── pedidos: List<Pedido> (relación)

Recurso (abstracto)
├── Libro (hereda)
└── DVD (hereda)

## 4.2 Tipos Anidados en Oracle

Oracle permite anidar tipos dentro de otros tipos:

### Objeto Embebido

-- Tipo para direccion
CREATE OR REPLACE TYPE tipo_direccion AS OBJECT (
    calle           VARCHAR2(200),
    ciudad          VARCHAR2(100),
    codigo_postal   VARCHAR2(10),
    pais            VARCHAR2(50)
);
/

-- Tipo cliente con direccion embebida
CREATE OR REPLACE TYPE tipo_cliente AS OBJECT (
    id          NUMBER,
    nombre      VARCHAR2(100),
    email       VARCHAR2(100),
    direccion   tipo_direccion      -- Objeto anidado
);
/

-- Crear tabla
CREATE TABLE clientes OF tipo_cliente (
    id PRIMARY KEY
);

### Insertar y Consultar Objetos Anidados en Oracle

-- Insertar cliente con dirección
INSERT INTO clientes VALUES (
    tipo_cliente(
        1,
        'Juan García',
        'juan@email.com',
        tipo_direccion('Calle Mayor 10', 'Madrid', '28001', 'España')
    )
);

-- Consultar accediendo a objeto anidado
SELECT c.nombre, c.direccion.ciudad, c.direccion.codigo_postal
FROM clientes c
WHERE c.direccion.ciudad = 'Madrid';

-- Actualizar campo del objeto anidado
UPDATE clientes c
SET c.direccion.ciudad = 'Barcelona',
    c.direccion.codigo_postal = '08001'
WHERE c.id = 1;

## 4.3 Colecciones en Oracle

Oracle ofrece dos tipos de colecciones:

### VARRAY (Array de tamaño fijo)

-- Tipo para teléfono
CREATE OR REPLACE TYPE tipo_telefono AS OBJECT (
    tipo    VARCHAR2(20),   -- 'móvil', 'casa', 'trabajo'
    numero  VARCHAR2(20)
);
/

-- VARRAY de teléfonos (máximo 5)
CREATE OR REPLACE TYPE lista_telefonos AS VARRAY(5) OF tipo_telefono;
/

-- Tipo cliente completo
CREATE OR REPLACE TYPE tipo_cliente_v2 AS OBJECT (
    id          NUMBER,
    nombre      VARCHAR2(100),
    direccion   tipo_direccion,
    telefonos   lista_telefonos     -- Colección VARRAY
);
/

### NESTED TABLE (Tabla anidada)

-- Tipo para email
CREATE OR REPLACE TYPE tipo_email AS OBJECT (
    tipo    VARCHAR2(20),
    email   VARCHAR2(100)
);
/

-- Tabla anidada de emails (sin límite)
CREATE OR REPLACE TYPE tabla_emails AS TABLE OF tipo_email;
/

-- Tabla con nested table
CREATE TABLE contactos (
    id          NUMBER PRIMARY KEY,
    nombre      VARCHAR2(100),
    emails      tabla_emails
) NESTED TABLE emails STORE AS emails_nt;

### Operaciones con Colecciones en Oracle

-- Insertar con VARRAY
INSERT INTO clientes_v2 VALUES (
    tipo_cliente_v2(
        1,
        'María López',
        tipo_direccion('Av. Principal 50', 'Valencia', '46001', 'España'),
        lista_telefonos(
            tipo_telefono('móvil', '666111222'),
            tipo_telefono('trabajo', '961234567')
        )
    )
);

-- Consultar desanidando la colección
SELECT c.nombre, t.tipo, t.numero
FROM clientes_v2 c, TABLE(c.telefonos) t;

-- Insertar con NESTED TABLE
INSERT INTO contactos VALUES (
    1,
    'Pedro Sánchez',
    tabla_emails(
        tipo_email('personal', 'pedro@gmail.com'),
        tipo_email('trabajo', 'pedro@empresa.com')
    )
);

## 4.4 Herencia de Tipos en Oracle

Oracle soporta herencia con la cláusula `UNDER`:

-- Tipo base (NOT FINAL permite herencia)
CREATE OR REPLACE TYPE tipo_recurso AS OBJECT (
    id          NUMBER,
    titulo      VARCHAR2(200),
    anio        NUMBER,
    disponible  CHAR(1)
) NOT FINAL;
/

-- Tipo derivado: Libro
CREATE OR REPLACE TYPE tipo_libro UNDER tipo_recurso (
    isbn        VARCHAR2(20),
    autor       VARCHAR2(100),
    paginas     NUMBER
);
/

-- Tipo derivado: DVD
CREATE OR REPLACE TYPE tipo_dvd UNDER tipo_recurso (
    director    VARCHAR2(100),
    duracion    NUMBER  -- minutos
);
/

-- Tabla que almacena cualquier tipo de recurso
CREATE TABLE recursos OF tipo_recurso;

### Consultas Polimórficas en Oracle

-- Insertar diferentes tipos
INSERT INTO recursos VALUES (
    tipo_libro(1, 'Don Quijote', 1605, 'S', '978-84-376-0494-7', 'Cervantes', 1345)
);

INSERT INTO recursos VALUES (
    tipo_dvd(2, 'El Señor de los Anillos', 2001, 'S', 'Peter Jackson', 178)
);

-- Consulta polimórfica (devuelve todos los tipos)
SELECT VALUE(r) FROM recursos r;

-- Filtrar por tipo con IS OF
SELECT VALUE(r) FROM recursos r 
WHERE VALUE(r) IS OF (tipo_libro);

-- Casting con TREAT para acceder a atributos específicos
SELECT r.titulo, TREAT(VALUE(r) AS tipo_libro).isbn
FROM recursos r
WHERE VALUE(r) IS OF (tipo_libro);

## 4.5 Tipos Estructurados en PostgreSQL

### Tipos Compuestos Anidados

-- Tipo dirección
CREATE TYPE tipo_direccion AS (
    calle           VARCHAR(200),
    ciudad          VARCHAR(100),
    codigo_postal   VARCHAR(10),
    pais            VARCHAR(50)
);

-- Tabla cliente con tipo anidado y array
CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    email VARCHAR(100),
    direccion tipo_direccion,
    telefonos VARCHAR(20)[]     -- Array de strings
);

### Operaciones con Tipos y Arrays en PostgreSQL

-- Insertar
INSERT INTO clientes (nombre, email, direccion, telefonos) VALUES (
    'Juan García',
    'juan@email.com',
    ROW('Calle Mayor 10', 'Madrid', '28001', 'España')::tipo_direccion,
    ARRAY['666111222', '911234567']
);

-- Consultar campos anidados
SELECT nombre, (direccion).ciudad, (direccion).codigo_postal
FROM clientes
WHERE (direccion).ciudad = 'Madrid';

-- Consultar con arrays
SELECT nombre, telefonos[1] AS telefono_principal
FROM clientes;

-- Desanidar array
SELECT nombre, unnest(telefonos) AS telefono
FROM clientes;

-- Actualizar campo anidado
UPDATE clientes
SET direccion.ciudad = 'Barcelona'
WHERE id = 1;

-- Agregar elemento a array
UPDATE clientes
SET telefonos = array_append(telefonos, '933445566')
WHERE id = 1;

## 4.6 Herencia de Tablas en PostgreSQL

PostgreSQL soporta herencia a nivel de **tablas** (no tipos):

-- Tabla base
CREATE TABLE recursos (
    id SERIAL PRIMARY KEY,
    titulo VARCHAR(200) NOT NULL,
    anio INTEGER,
    disponible BOOLEAN DEFAULT true
);

-- Tabla derivada: Libros
CREATE TABLE libros (
    isbn VARCHAR(20),
    autor VARCHAR(100),
    paginas INTEGER
) INHERITS (recursos);

-- Tabla derivada: DVDs
CREATE TABLE dvds (
    director VARCHAR(100),
    duracion INTEGER
) INHERITS (recursos);

### Consultas Polimórficas en PostgreSQL

-- Insertar en tablas hijas
INSERT INTO libros (titulo, anio, isbn, autor, paginas)
VALUES ('Don Quijote', 1605, '978-84-376-0494-7', 'Cervantes', 1345);

INSERT INTO dvds (titulo, anio, director, duracion)
VALUES ('El Señor de los Anillos', 2001, 'Peter Jackson', 178);

-- Consulta polimórfica (incluye libros y dvds)
SELECT titulo, anio FROM recursos;

-- Solo tabla padre (ONLY excluye hijas)
SELECT titulo, anio FROM ONLY recursos;

-- Identificar tipo de fila
SELECT titulo,
       CASE tableoid::regclass::text
           WHEN 'libros' THEN 'Libro'
           WHEN 'dvds' THEN 'DVD'
           ELSE 'Recurso'
       END AS tipo
FROM recursos;

-- Consultar tabla específica con todos los campos
SELECT titulo, autor, paginas FROM libros;

## 4.7 Tipos Embebidos en JPA

JPA usa `@Embeddable` y `@Embedded` para objetos sin identidad propia:

import jakarta.persistence.*;

@Embeddable
public class Direccion {
    private String calle;
    private String ciudad;
    
    @Column(name = "cp")
    private String codigoPostal;
    
    private String pais;
    
    // Constructores, getters, setters
}

@Entity
@Table(name = "clientes")
public class Cliente {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    @Embedded
    private Direccion direccion;
    
    @ElementCollection
    @CollectionTable(name = "cliente_telefonos", 
                    joinColumns = @JoinColumn(name = "cliente_id"))
    @Column(name = "telefono")
    private List<String> telefonos = new ArrayList<>();
    
    // Constructores, getters, setters
}

## 4.8 Herencia en JPA

JPA ofrece tres estrategias de mapeo de herencia:

### SINGLE_TABLE (una tabla para toda la jerarquía)

@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tipo", discriminatorType = DiscriminatorType.STRING)
public abstract class Recurso {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String titulo;
    private Integer anio;
    private Boolean disponible = true;
}

@Entity
@DiscriminatorValue("LIBRO")
public class Libro extends Recurso {
    private String isbn;
    private String autor;
    private Integer paginas;
}

@Entity
@DiscriminatorValue("DVD")
public class DVD extends Recurso {
    private String director;
    private Integer duracion;
}

### TABLE_PER_CLASS (una tabla por clase concreta)

@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Recurso {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    // ... atributos comunes
}

@Entity
@Table(name = "libros")
public class Libro extends Recurso {
    // ... atributos específicos
}

### JOINED (una tabla por clase, relacionadas por FK)

@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@Table(name = "recursos")
public abstract class Recurso {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // ... atributos comunes
}

@Entity
@Table(name = "libros")
@PrimaryKeyJoinColumn(name = "recurso_id")
public class Libro extends Recurso {
    // ... atributos específicos
}

## 4.9 Relaciones entre Entidades JPA

### Relaciones Básicas

@Entity
public class Pedido {
    @Id
    @GeneratedValue
    private Long id;
    
    // Muchos pedidos pertenecen a un cliente
    @ManyToOne
    @JoinColumn(name = "cliente_id")
    private Cliente cliente;
    
    // Un pedido tiene muchas líneas
    @OneToMany(mappedBy = "pedido", 
               cascade = CascadeType.ALL,
               orphanRemoval = true)
    private List<LineaPedido> lineas = new ArrayList<>();
    
    private LocalDateTime fecha;
}

@Entity
public class LineaPedido {
    @Id
    @GeneratedValue
    private Long id;
    
    @ManyToOne
    private Pedido pedido;
    
    @ManyToOne
    private Producto producto;
    
    private Integer cantidad;
}

## 4.10 Resumen del Tema

Conceptos Clave

1. Los **objetos estructurados** contienen otros objetos, colecciones o participan en herencia.
    
2. **Oracle** soporta objetos anidados, VARRAY, NESTED TABLE y herencia con UNDER.
    
3. **PostgreSQL** usa tipos compuestos, arrays y herencia de tablas con INHERITS.
    
4. **JPA** usa `@Embeddable`/`@Embedded` para objetos embebidos.
    
5. JPA ofrece tres estrategias de herencia: SINGLE_TABLE, TABLE_PER_CLASS, JOINED.
    
6. Las **colecciones** se mapean con `@ElementCollection` (valores) o `@OneToMany` (entidades).