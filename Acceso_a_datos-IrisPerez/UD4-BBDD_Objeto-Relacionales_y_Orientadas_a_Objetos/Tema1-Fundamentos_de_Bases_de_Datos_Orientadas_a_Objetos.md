## CE 4a: Identificar ventajas e inconvenientes de las BDOO

En este tema estudiaremos los fundamentos teóricos de las bases de datos que almacenan objetos, sus tipos, ventajas, inconvenientes y casos de uso apropiados.

## 1.1 Evolución de los Modelos de Bases de Datos

La historia de las bases de datos ha pasado por varias etapas evolutivas:

|Década|Modelo|Características|
|---|---|---|
|1960s|**Jerárquico**|Estructura de árbol, navegación padre-hijo (IMS de IBM)|
|1960s-70s|**En Red**|Grafos de relaciones, CODASYL|
|1970s-presente|**Relacional**|Tablas, SQL, normalización (Oracle, PostgreSQL, MySQL)|
|1990s|**Orientado a Objetos**|Objetos persistentes, herencia, ODMG|
|1990s-presente|**Objeto-Relacional**|SQL extendido con tipos objeto (Oracle, PostgreSQL)|
|2000s-presente|**NoSQL**|Documentos, clave-valor, grafos (MongoDB, Redis)|

## 1.2 El Desajuste de Impedancia Objeto-Relacional

El **Object-Relational Impedance Mismatch** es el conjunto de dificultades técnicas que surgen cuando un modelo de objetos en memoria debe mapearse a un modelo relacional en base de datos.

### Problemas principales

|Problema|Descripción|
|---|---|
|**Granularidad**|Los objetos pueden tener estructuras más complejas que las filas|
|**Herencia**|SQL no soporta herencia de forma nativa|
|**Identidad**|Diferencia entre identidad de objeto (==) y equivalencia (equals)|
|**Asociaciones**|Las referencias entre objetos requieren foreign keys|
|**Navegación**|Los objetos navegan por referencias, SQL hace JOINs|

### Ejemplo del problema

En Java tenemos un grafo de objetos natural:

// Navegación natural en objetos
Pedido pedido = cliente.getPedidos().get(0);
String nombreProducto = pedido.getLineas().get(0).getProducto().getNombre();

En SQL debemos hacer múltiples JOINs:

-- Navegación mediante JOINs
SELECT p.nombre
FROM clientes c
JOIN pedidos pe ON c.id = pe.cliente_id
JOIN lineas_pedido lp ON pe.id = lp.pedido_id
JOIN productos p ON lp.producto_id = p.id
WHERE c.id = 1;

## 1.3 Tipos de Bases de Datos que Almacenan Objetos

Existen dos enfoques principales para almacenar objetos en bases de datos:

### OODBMS (Object-Oriented Database Management System)

Bases de datos **puramente orientadas a objetos** que almacenan objetos directamente.

Nota

**Ejemplos**: ObjectDB, db4o, Versant, Objectivity/DB

**Características:**

- Persistencia transparente de objetos Java/C++
    
- Navegación directa por referencias (sin JOINs)
    
- Herencia y polimorfismo nativos
    
- Estándar ODMG (Object Data Management Group)
    

### ORDBMS (Object-Relational Database Management System)

Bases de datos **relacionales extendidas** con capacidades de objetos.

Nota

**Ejemplos**: Oracle Database, PostgreSQL, IBM DB2

**Características:**

- SQL extendido con tipos objeto (SQL:1999, SQL:2003)
    
- Compatibilidad hacia atrás con aplicaciones relacionales
    
- Tipos definidos por usuario (UDT)
    
- Herencia de tipos y tablas
    

## 1.4 Comparativa OODBMS vs ORDBMS vs RDBMS

|Característica|RDBMS|ORDBMS|OODBMS|
|---|---|---|---|
|**Modelo de datos**|Tablas|Tablas + Tipos objeto|Objetos|
|**Lenguaje**|SQL|SQL extendido|OQL / API nativa|
|**Herencia**|No|Sí (tipos)|Sí (clases)|
|**Métodos**|Procedimientos almacenados|Métodos en tipos|Métodos Java|
|**Identidad**|Clave primaria|OID|OID|
|**Navegación**|JOINs|JOINs + REF|Referencias directas|
|**Madurez**|Muy alta|Alta|Media|
|**Ecosistema**|Muy amplio|Amplio|Limitado|

## 1.5 Ventajas de las Bases de Datos que Almacenan Objetos

### Ventajas Comunes (OODBMS y ORDBMS)

1. **Eliminación del desajuste de impedancia**: El modelo de datos coincide mejor con el modelo de objetos de la aplicación.
    
2. **Soporte nativo para tipos complejos**: Arrays, estructuras anidadas, colecciones.
    
3. **Herencia a nivel de base de datos**: Los tipos pueden heredar de otros tipos.
    
4. **Reutilización de tipos**: Los tipos definidos se pueden usar en múltiples tablas.
    
5. **Modelado más natural**: Mejor representación de la realidad del dominio.
    

### Ventajas Específicas de OODBMS

- **Navegación directa**: Acceso a objetos relacionados sin JOINs
    
- **Persistencia transparente**: Los objetos se guardan automáticamente
    
- **Alto rendimiento** en dominios con estructuras complejas
    

### Ventajas Específicas de ORDBMS

- **Compatibilidad SQL**: Se pueden usar herramientas SQL existentes
    
- **Migración gradual**: Adopción incremental de características objeto
    
- **Ecosistema maduro**: Amplio soporte de herramientas y drivers
    

## 1.6 Inconvenientes de las Bases de Datos que Almacenan Objetos

### Inconvenientes Comunes

1. **Curva de aprendizaje**: Requiere conocer nuevos conceptos y sintaxis.
    
2. **Complejidad de diseño**: Los esquemas orientados a objetos pueden ser más complejos.
    
3. **Portabilidad limitada**: Sintaxis específica de cada fabricante.
    
4. **Overhead en consultas simples**: Para operaciones CRUD básicas puede ser excesivo.
    

### Inconvenientes Específicos de OODBMS

- **Ecosistema reducido**: Menos herramientas, menos desarrolladores con experiencia
    
- **Menor soporte de BI/Reporting**: Las herramientas de análisis esperan SQL
    
- **Integración con sistemas legacy**: Difícil conectar con sistemas relacionales existentes
    

### Inconvenientes Específicos de ORDBMS

- **Costes de licenciamiento**: Oracle tiene costes elevados
    
- **Sintaxis propietaria**: Diferencias significativas entre Oracle y PostgreSQL
    

## 1.7 Casos de Uso Apropiados

Las bases de datos orientadas a objetos son especialmente útiles en:

|Dominio|Justificación|
|---|---|
|**CAD/CAM**|Estructuras geométricas complejas|
|**GIS (Sistemas de Información Geográfica)**|Datos espaciales, polígonos, rutas|
|**Multimedia**|Imágenes, vídeo, audio con metadatos|
|**Telecomunicaciones**|Configuraciones de red complejas|
|**Aplicaciones científicas**|Datos experimentales estructurados|
|**Sistemas financieros**|Tipos monetarios personalizados|
|**Jerarquías de productos**|Catálogos con herencia de atributos|

### Cuándo elegir cada tecnología

Guía de Decisión

**Elige RDBMS tradicional cuando:**

- Los datos son tabulares y normalizados
    
- Se requiere máxima portabilidad
    
- El equipo tiene experiencia SQL
    

**Elige ORDBMS (Oracle/PostgreSQL) cuando:**

- Necesitas tipos complejos pero manteniendo SQL
    
- Tienes aplicaciones legacy que migrar gradualmente
    
- Requieres herramientas de BI y reporting
    

**Elige OODBMS (ObjectDB) cuando:**

- El dominio es intrínsecamente orientado a objetos
    
- La navegación entre objetos es intensiva
    
- No necesitas integración con sistemas SQL
    

## 1.8 Tecnologías del Curso

En este RA trabajaremos con tres tecnologías:

### Oracle Database XE 21c (ORDBMS)

- SGBD empresarial líder en el mercado
    
- Soporte completo de tipos objeto con `CREATE TYPE AS OBJECT`
    
- Herencia de tipos con `UNDER`
    
- Colecciones: `VARRAY` y `NESTED TABLE`
    
- Referencias entre objetos: `REF` y `DEREF`
    

### PostgreSQL 15+ (ORDBMS)

- SGBD open source más avanzado
    
- Tipos compuestos con `CREATE TYPE`
    
- Herencia de tablas con `INHERITS`
    
- Arrays nativos
    
- Soporte JSON/JSONB
    

### ObjectDB 2.8.9 (OODBMS)

- OODBMS compatible con JPA
    
- Persistencia transparente de objetos Java
    
- No requiere mapeo O/R
    
- Ideal para aprender conceptos OODBMS con API conocida (JPA)
    

## 1.9 Resumen del Tema

Conceptos Clave

1. El **desajuste de impedancia** es el problema fundamental que las BDOO intentan resolver.
    
2. **OODBMS** almacenan objetos directamente, eliminando el mapeo O/R.
    
3. **ORDBMS** extienden SQL con capacidades de objetos, manteniendo compatibilidad.
    
4. Las **ventajas** incluyen mejor modelado, tipos complejos y herencia.
    
5. Los **inconvenientes** incluyen curva de aprendizaje y menor ecosistema (OODBMS).
    
6. La elección depende del **dominio**, la **infraestructura existente** y los **requisitos de integración**.