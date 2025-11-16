## Introducción

El acceso a datos es una parte fundamental en el desarrollo de aplicaciones empresariales. Java ofrece **JDBC (Java Database Connectivity)** como API estándar para conectarse y trabajar con bases de datos relacionales.

En este tema aprenderás a:

- Comprender la arquitectura de JDBC y sus componentes.
    
- Diferenciar entre JDBC básico y avanzado.
    
- Ejecutar consultas con `Statement`, `PreparedStatement` y `CallableStatement`.
    
- Gestionar resultados con `ResultSet` y metadatos.
    
- Usar procedimientos almacenados.
    
- Controlar transacciones (commit, rollback, savepoints).
    
- Optimizar el acceso con **pools de conexiones**.

## 1. Arquitectura de JDBC

### 1.1. Visión general

JDBC es una **capa de abstracción** entre una aplicación Java y el motor de base de datos. Permite escribir código independiente del proveedor (MySQL, PostgreSQL, Oracle…), cambiando únicamente el driver JDBC.

### 1.2. Flujo de trabajo

1. La aplicación Java usa la API JDBC.
    
2. El **DriverManager** selecciona el controlador adecuado en función de la URL.
    
3. El **Driver JDBC** traduce las instrucciones a comandos específicos del SGBD.
    
4. Se abre una **Connection**, que representa la sesión con la BD.
    
5. Se crean **sentencias** para ejecutar SQL.
    
6. Los resultados se leen mediante **ResultSet**.
    
7. Al final se cierran recursos para liberar memoria.

### 1.3. Componentes principales

- **DriverManager**: gestiona los controladores disponibles.
    
- **Driver**: traduce llamadas JDBC a instrucciones nativas del SGBD.
    
- **Connection**: representa una conexión activa con la BD.
    
- **Statement/PreparedStatement/CallableStatement**: objetos que ejecutan sentencias SQL.
    
- **ResultSet**: contiene los resultados de consultas.
    
- **Metadata**: proporciona información sobre la BD (`DatabaseMetaData`) y sobre consultas (`ResultSetMetaData`).

### 1.4. Tipos de drivers JDBC

| Tipo   | Descripción      | Ventajas                  | Desventajas                  | Estado      |
| ------ | ---------------- | ------------------------- | ---------------------------- | ----------- |
| Tipo 1 | JDBC-ODBC Bridge | Simple                    | Depende de ODBC, lento       | Obsoleto    |
| Tipo 2 | Driver nativo    | Muy rápido                | Dependiente de SO            | Poco usado  |
| Tipo 3 | Middleware       | Multiplataforma           | Requiere servidor intermedio | Raro        |
| Tipo 4 | Puro Java        | Portátil, estándar actual | Depende del proveedor        | ✅ Usado hoy |

## 2. Diferencias entre JDBC básico y avanzado

### 2.1. JDBC básico

- Una conexión por cada operación.
    
- Uso de `Statement` con SQL fijo.
    
- Autocommit activado (cada sentencia es una transacción).
    
- Limitado en seguridad y rendimiento.

### 2.2. JDBC avanzado

- Uso de **pools de conexiones** (reutilización).
    
- Consultas parametrizadas con `PreparedStatement`.
    
- Ejecución de **procedimientos almacenados** con `CallableStatement`.
    
- Control manual de transacciones (`commit`, `rollback`, `savepoint`).
    
- Acceso a metadatos.

### 2.3. Tabla comparativa

|Aspecto|JDBC básico|JDBC avanzado|
|---|---|---|
|Conexión|Una conexión por operación|Pool de conexiones|
|Sentencias|`Statement`|`PreparedStatement`, `CallableStatement`|
|Consultas|Simples|Parametrizadas, por lotes, metadatos|
|Transacciones|Automáticas (autocommit)|Manuales y controladas|
|Procedimientos|No usados|Usados en la BD|

## 3. Statement, PreparedStatement y CallableStatement

### 3.1. Statement

- Ejecuta SQL fijo, útil para consultas simples.
    
- Vulnerable a **SQL Injection** si se usan datos externos.

Ejemplo:

```java
Statement stmt = con.createStatement();
ResultSet rs = stmt.executeQuery("SELECT id, nombre FROM empleados");
```

### 3.2. PreparedStatement

- SQL parametrizado.
    
- Mayor rendimiento: el plan de ejecución se reutiliza.
    
- Seguridad frente a inyección SQL.

Ejemplo:

```java
PreparedStatement ps = con.prepareStatement(
    "SELECT id, nombre FROM empleados WHERE id = ?");
ps.setInt(1, 2);
ResultSet rs = ps.executeQuery();
```

### 3.3. CallableStatement

- Permite llamar a **procedimientos almacenados** en la BD.

Procedimiento en MySQL:

```MySQL
DELIMITER //
CREATE PROCEDURE obtener_empleado(IN empId INT)
BEGIN
    SELECT id, nombre FROM empleados WHERE id = empId;
END //
DELIMITER ;
```

Uso en Java:

```java
CallableStatement cs = con.prepareCall("{call obtener_empleado(?)}");
cs.setInt(1, 3);
ResultSet rs = cs.executeQuery();
```

## 4. ResultSet y Metadatos

### 4.1. ResultSet

- Cursor que recorre fila a fila los resultados de una consulta.
    
- Métodos clave:
    
    - `next()` → avanza a la siguiente fila.
        
    - `getInt("columna")`, `getString("columna")` → acceden a valores.

### 4.2. Tipos de ResultSet

- `TYPE_FORWARD_ONLY` → solo hacia adelante.
    
- `TYPE_SCROLL_INSENSITIVE` → permite moverse en ambas direcciones, no refleja cambios en BD.
    
- `TYPE_SCROLL_SENSITIVE` → refleja cambios en tiempo real.

### 4.3. Metadata

- **DatabaseMetaData**: información sobre la BD.
    
- **ResultSetMetaData**: información sobre el resultado de una consulta.

## 5. Control de transacciones

### 5.1. Concepto

Una transacción es una **unidad de trabajo** que cumple las propiedades ACID:

- **Atomicidad** → se ejecuta toda o ninguna.
    
- **Consistencia** → mantiene la integridad de datos.
    
- **Aislamiento** → no interfiere con otras transacciones.
    
- **Durabilidad** → los cambios se guardan de forma permanente.

### 5.2. JDBC y transacciones

Por defecto, JDBC trabaja en **autocommit** (cada instrucción es una transacción). Podemos desactivarlo:

```java
con.setAutoCommit(false);
```

Ejemplo de control manual:

```java
try {
    con.setAutoCommit(false);

    PreparedStatement ps1 = con.prepareStatement(
        "UPDATE cuentas SET saldo = saldo - ? WHERE id = ?");
    ps1.setDouble(1, 500);
    ps1.setInt(2, 1);
    ps1.executeUpdate();

    PreparedStatement ps2 = con.prepareStatement(
        "UPDATE cuentas SET saldo = saldo + ? WHERE id = ?");
    ps2.setDouble(1, 500);
    ps2.setInt(2, 2);
    ps2.executeUpdate();

    con.commit(); // confirmar transacción
} catch (SQLException e) {
    con.rollback(); // deshacer cambios si falla
}
```

## 6. Pools de conexiones

### 6.1. Problema sin pool

- Cada conexión se abre y se cierra por cada petición.
    
- Alto coste en aplicaciones con muchos usuarios.

### 6.2. Solución: Connection Pooling

- Reutilización de conexiones abiertas.
    
- Mejor rendimiento y escalabilidad.

### 6.3. Librerías comunes

- **HikariCP** (rápido y ligero).
    
- **Apache DBCP**.
    
- **C3P0**.

Ejemplo básico con HikariCP:

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/empresa");
config.setUsername("root");
config.setPassword("root");

HikariDataSource ds = new HikariDataSource(config);

try (Connection con = ds.getConnection()) {
    System.out.println("Conexión obtenida del pool");
}
```

## 7. Actividad práctica

### 2.1. Para los alumnos

- Crear BD `empresa`.
    
- Usar `Statement` para listar empleados.
    
- Usar `PreparedStatement` para buscar empleados por id.
    
- Crear procedimiento `obtener_empleado` y llamarlo con `CallableStatement`.
    
- Entregar código Java, script SQL y capturas de pantalla.

### 7.2. Para el profesor (soluciones)

- Incluye el código resuelto para cada paso.
    
- Criterios de corrección: conexión (2 ptos), Statement (2 ptos), PreparedStatement (3 ptos), CallableStatement (3 ptos), cierre correcto (1 pto).

## 8. Resumen

- JDBC es la API estándar de Java para acceso a datos relacionales.
    
- Los componentes clave son DriverManager, Connection, Statement, ResultSet y Metadata.
    
- JDBC avanzado incluye consultas parametrizadas, procedimientos y transacciones.
    
- El uso de pools de conexiones mejora rendimiento en aplicaciones reales.
    
- Buenas prácticas: siempre cerrar recursos, usar `try-with-resources` y protegerse de SQL Injection.

## 9. Tabla comparativa final

| Elemento      | JDBC básico       | JDBC avanzado                            |
| ------------- | ----------------- | ---------------------------------------- |
| Conexión      | Una por operación | Reutilización con pool                   |
| Sentencias    | `Statement`       | `PreparedStatement`, `CallableStatement` |
| Seguridad     | Baja              | Alta (parámetros, SP)                    |
| Transacciones | Autocommit        | Manual con commit/rollback               |
| Escalabilidad | Limitada          | Alta con pooling                         |

> [!Actividad 2.1]
> https://github.com/IrisCampusFP/ActividadesAccesoADatos/tree/main/UD2-JDBC_Avanzado/Act2.1