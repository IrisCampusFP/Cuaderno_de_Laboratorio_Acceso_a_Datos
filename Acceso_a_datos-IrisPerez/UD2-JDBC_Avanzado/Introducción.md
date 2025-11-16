# JDBC Avanzado en Java

## ¿Qué es JDBC Avanzado?

**JDBC (Java Database Connectivity)** es la API estándar de Java que permite la conexión y manipulación de bases de datos relacionales. En su versión avanzada, no solo ejecuta consultas simples, sino que ofrece herramientas para:

- Optimizar las conexiones mediante **pools de conexiones**.
    
- Ejecutar sentencias **DDL** y **DML** con seguridad.
    
- Consultar y manipular datos con **ResultSet** y metadatos.
    
- Llamar a **procedimientos almacenados** con parámetros de entrada y salida.
    
- Controlar **transacciones** con `commit`, `rollback` y `savepoints`.

Estas características son fundamentales en aplicaciones empresariales donde se requiere **seguridad, escalabilidad y consistencia**.

## Arquitectura de JDBC

JDBC funciona como una capa de abstracción entre Java y la base de datos. Sus componentes principales son:

- **DriverManager**: gestiona los drivers y crea conexiones.
    
- **Driver JDBC**: traduce las llamadas JDBC al lenguaje nativo del SGBD.
    
- **Connection**: representa la sesión activa con la base de datos.
    
- **Statement, PreparedStatement y CallableStatement**: permiten ejecutar consultas.
    
- **ResultSet**: almacena los resultados de las consultas.
    
- **MetaData**: información sobre la BD (`DatabaseMetaData`) o sobre los resultados (`ResultSetMetaData`).

### Tipos de drivers JDBC

|Tipo|Descripción|Estado|
|---|---|---|
|Tipo 1|JDBC-ODBC Bridge|Obsoleto|
|Tipo 2|Driver nativo|Poco usado|
|Tipo 3|Middleware|Raro en la práctica|
|Tipo 4|Driver puro Java (TCP/IP)|Estándar actual|

## Diferencias entre JDBC básico y avanzado

|Aspecto|JDBC básico|JDBC avanzado|
|---|---|---|
|Conexión|Una por operación|Reutilización con pool|
|Sentencias|`Statement`|`PreparedStatement`, `CallableStatement`|
|Transacciones|Automáticas|Manuales (`commit`, `rollback`, `savepoint`)|
|Seguridad|Baja (SQL Injection)|Alta (parametrización)|
|Escalabilidad|Limitada|Alta (pooling)|

## Sentencias SQL en JDBC

### DDL (Data Definition Language)

Usadas para crear o modificar estructuras: `CREATE`, `ALTER`, `DROP`.

### DML (Data Manipulation Language)

Usadas para manipular datos: `INSERT`, `UPDATE`, `DELETE`, `SELECT`.

En JDBC se utilizan principalmente con **PreparedStatement**, evitando concatenación de strings.

## Consultas avanzadas con JDBC

- **ResultSet**: cursor que permite recorrer filas de una consulta.
    
- **Tipos de ResultSet**: `TYPE_FORWARD_ONLY`, `TYPE_SCROLL_INSENSITIVE`, `TYPE_SCROLL_SENSITIVE`.
    
- **Metadatos**:
    
    - `ResultSetMetaData`: información sobre columnas devueltas.
        
    - `DatabaseMetaData`: información general de la base de datos.

## Procedimientos almacenados

Permiten definir lógica de negocio directamente en la base de datos. En JDBC se ejecutan con **CallableStatement**.

- **Parámetros IN** → enviados desde Java.
    
- **Parámetros OUT** → devueltos desde la BD.
    
- **Parámetros INOUT** → sirven como ambos.

Ejemplo en MySQL:

```MySQL
CREATE PROCEDURE incrementar_salario(IN empId INT, IN aumento DECIMAL(10,2), OUT nuevoSalario DECIMAL(10,2))
BEGIN
    UPDATE empleados SET salario = salario + aumento WHERE id = empId;
    SELECT salario INTO nuevoSalario FROM empleados WHERE id = empId;
END;
```

## Transacciones en JDBC

Una transacción es una unidad de trabajo que cumple las propiedades **ACID**. En JDBC se controlan con el objeto `Connection`.

- **commit()** → confirma los cambios.
    
- **rollback()** → revierte la transacción.
    
- **savepoint** → marca un punto intermedio al que se puede volver.

Niveles de aislamiento disponibles:

- `READ UNCOMMITTED`
    
- `READ COMMITTED`
    
- `REPEATABLE READ`
    
- `SERIALIZABLE`

## Pool de Conexiones

Para mejorar el rendimiento en aplicaciones con muchos usuarios concurrentes se utiliza un **connection pool**.

- **HikariCP**: rápido y ligero, estándar actual.
    
- **Apache DBCP**, **C3P0**: alternativas usadas en proyectos legacy.

Ejemplo con HikariCP:

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/empresa");
config.setUsername("root");
config.setPassword("root");
config.setMaximumPoolSize(5);

HikariDataSource ds = new HikariDataSource(config);
```

## Ejemplo integrado

Un sistema de gestión de empleados y proyectos combina:

- CRUD con `PreparedStatement`.
    
- Procedimientos almacenados con `CallableStatement`.
    
- Transacciones con control manual.
    
- Consultas avanzadas con `ResultSetMetaData`.
    
- Pool de conexiones para escalabilidad.

## Actividad Global de Evaluación

**Proyecto TechDAM**:

1. Crear las tablas `empleados`, `proyectos` y `asignaciones`.
    
2. Realizar operaciones CRUD con `PreparedStatement`.
    
3. Crear e invocar un procedimiento almacenado `asignar_empleado`.
    
4. Implementar transacciones con `commit`, `rollback` y `savepoint`.
    
5. Configurar un pool de conexiones con HikariCP.
    
6. Documentar las pruebas con capturas en PDF.

## Relación con RA y CE

- **RA2**: Desarrolla aplicaciones que manipulen la información almacenada en bases de datos utilizando APIs específicas.
    
- **CE2.1**: Conexiones con BD (`DriverManager`, HikariCP).
    
- **CE2.2**: Sentencias DDL y DML.
    
- **CE2.3**: Procedimientos almacenados y funciones.
    
- **CE2.4**: Transacciones con atomicidad.
    
- **CE2.5**: Seguridad en consultas (PreparedStatement).
    
- **CE2.6**: Documentación de pruebas y resultados.

## Guion de contenidos de la UD2 - JDBC Avanzado

- [Cuadro_Global–UD2-JDBC_Avanzado](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Cuadro_Global–UD2-JDBC_Avanzado.md)

- [Relación_de_la_Actividad_Global_con_RA_y_CE](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Relación_de_la_Actividad_Global_con_RA_y_CE.md)

- [Tema1–JDBC_Avanzado-Pool_DDL-DML_Consultas_Procedimientos_y_Transacciones](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Tema1–JDBC_Avanzado-Pool_DDL-DML_Consultas_Procedimientos_y_Transacciones.md)

- [Tema2–Pool_de_Conexiones_en_JDBC](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Tema2–Pool_de_Conexiones_en_JDBC.md)

- [Tema3–Ejecución_de_sentencias_DDL_y_DML_con JDBC](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Tema3–Ejecución_de_sentencias_DDL_y_DML_con JDBC.md)

- [Tema4–Consultas_avanzadas_con_JDBC-ResultSet_y_Metadatos](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Tema4–Consultas_avanzadas_con_JDBC-ResultSet_y_Metadatos.md)

- [Tema5–Procedimientos_almacenados_con_JDBC](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Tema5–Procedimientos_almacenados_con_JDBC.md)

- [Tema6–Gestión_de_Transacciones_en_JDBC](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Tema6–Gestión_de_Transacciones_en_JDBC.md)

- [Tema7–Ejemplos_prácticos_integrados_y_comparativa_final_de_JDBC_Avanzado](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Tema7–Ejemplos_prácticos_integrados_y_comparativa_final_de_JDBC_Avanzado.md)

## Extra

- [Tema8–Introducción_a_MAVEN](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Extra/Tema8–Introducción_a_MAVEN.md)

- [Actividad_Maven](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Extra/Actividad_Maven.md)

### Guía Proyecto TechDAM

- [-Enlace_al_proyecto](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/-Enlace_al_proyecto.md)

- [Bloque0-Introducción](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/Bloque0-Introducción.md)

- [Bloque1-Arquitectura_JDBC_y_Drivers](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/Bloque1-Arquitectura_JDBC_y_Drivers.md)

- [Bloque2-JDBC_Básico_vs_Avanzado](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/Bloque2-JDBC_Básico_vs_Avanzado.md)

- [Bloque3-Ejecución_de_Sentencias_DDL_y_DML_con JDBC](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/Bloque3-Ejecución_de_Sentencias_DDL_y_DML_con JDBC.md)

- [Bloque4-Consultas_Avanzadas_con_ResultSet_y_Metadatos](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/Bloque4-Consultas_Avanzadas_con_ResultSet_y_Metadatos.md)

- [Bloque5-Procedimientos_Almacenados_con_CallableStatement](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/Bloque5-Procedimientos_Almacenados_con_CallableStatement.md)

- [Bloque6-Gestión_de_Transacciones_con_JDBC](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/Bloque6-Gestión_de_Transacciones_con_JDBC.md)

- [Bloque7-Pool_de_Conexiones_con_HikariCP](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/Bloque7-Pool_de_Conexiones_con_HikariCP.md)

 - [Bloque8-Ejemplo_Integrado-Sistema_TechDAM_Completo](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/Bloque8-Ejemplo_Integrado-Sistema_TechDAM_Completo.md)

 - [Bloque9-Actividad_Global_de_Evaluación-Proyecto_TechDAM](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/Bloque9-Actividad_Global_de_Evaluación-Proyecto_TechDAM.md)

 - [Bloque10-Anexos_y_Referencias](Acceso_a_datos-IrisPerez/UD2-JDBC_Avanzado/Guía_Proyecto_TechDAM/Bloque10-Anexos_y_Referencias.md)
