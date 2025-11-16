## Introducción

Hasta ahora hemos visto cómo ejecutar sentencias SQL básicas (DDL y DML). En este tema vamos a profundizar en **las consultas de selección (`SELECT`)** y en el manejo de los resultados con JDBC, a través de las clases:

- `ResultSet`: conjunto de resultados de una consulta.
    
- `ResultSetMetaData`: información sobre las columnas de la consulta.
    
- `DatabaseMetaData`: información sobre la base de datos y sus características.

## 1. ResultSet

Un `ResultSet` es un **cursor** que se sitúa sobre la primera fila de los resultados de una consulta y que se puede desplazar fila a fila para leer los datos.

### 1.1. Métodos principales

- `next()` → mueve el cursor a la siguiente fila.
    
- `getInt("columna")`, `getString("columna")`, `getDouble("columna")` → obtiene valores.
    
- `wasNull()` → comprueba si el último valor leído era `NULL`.

### 1.2. Tipos de ResultSet

|Tipo|Descripción|
|---|---|
|`TYPE_FORWARD_ONLY`|Solo permite avanzar hacia delante (por defecto).|
|`TYPE_SCROLL_INSENSITIVE`|Permite moverse hacia delante y atrás, pero no refleja cambios en la BD mientras se navega.|
|`TYPE_SCROLL_SENSITIVE`|Permite desplazamiento completo y refleja los cambios hechos en la BD en tiempo real.|

Además, se puede definir la **concurrencia**:

- `CONCUR_READ_ONLY` → solo lectura (por defecto).
    
- `CONCUR_UPDATABLE` → permite modificar directamente los datos desde el ResultSet.

### 1.3. Ejemplo básico

```java
Statement stmt = con.createStatement(
    ResultSet.TYPE_SCROLL_INSENSITIVE,
    ResultSet.CONCUR_READ_ONLY);

ResultSet rs = stmt.executeQuery("SELECT id, nombre, salario FROM empleados");

while (rs.next()) {
    int id = rs.getInt("id");
    String nombre = rs.getString("nombre");
    double salario = rs.getDouble("salario");
    System.out.println(id + " - " + nombre + " - " + salario);
}
```

## 2. ResultSet actualizable

Un `ResultSet` puede configurarse como actualizable. Esto permite modificar filas directamente sin usar `UPDATE`.

### Ejemplo

```java
Statement stmt = con.createStatement(
    ResultSet.TYPE_SCROLL_INSENSITIVE,
    ResultSet.CONCUR_UPDATABLE);

ResultSet rs = stmt.executeQuery("SELECT id, nombre, salario FROM empleados");

// Mover al primer registro
if (rs.next()) {
    rs.updateDouble("salario", 2500.00);
    rs.updateRow(); // aplica el cambio en la BD
}
```

## 3. ResultSetMetaData

La clase `ResultSetMetaData` describe la estructura de un conjunto de resultados:

- Número de columnas.
    
- Nombre de cada columna.
    
- Tipo de datos SQL.
    
- Precisión y tamaño.

### Ejemplo

```java
ResultSet rs = stmt.executeQuery("SELECT * FROM empleados");
ResultSetMetaData meta = rs.getMetaData();

int columnas = meta.getColumnCount();
System.out.println("Número de columnas: " + columnas);

for (int i = 1; i <= columnas; i++) {
    System.out.println("Columna " + i + ": " +
                       meta.getColumnName(i) +
                       " (" + meta.getColumnTypeName(i) + ")");
}
```

**Salida esperada**:

```MySQL
Número de columnas: 3

Columna 1: id (INT)
Columna 2: nombre (VARCHAR)
Columna 3: salario (DECIMAL)
```

## 4. DatabaseMetaData

La clase `DatabaseMetaData` permite obtener información sobre la base de datos en uso:

- Nombre del producto y versión.
    
- Nombre del driver JDBC y versión.
    
- Tablas disponibles en la BD.
    
- Capacidades soportadas (transacciones, tipos de ResultSet, etc.).

### Ejemplo

```java
DatabaseMetaData dbMeta = con.getMetaData();

System.out.println("Producto BD: " + dbMeta.getDatabaseProductName());
System.out.println("Versión: " + dbMeta.getDatabaseProductVersion());
System.out.println("Driver: " + dbMeta.getDriverName());
```

## 5. Ejemplo integrado

```java
try (Connection con = DriverManager.getConnection(url, user, pass);
     Statement stmt = con.createStatement()) {

    ResultSet rs = stmt.executeQuery("SELECT * FROM empleados");
    ResultSetMetaData meta = rs.getMetaData();

    System.out.println("La tabla empleados tiene " + meta.getColumnCount() + " columnas:");

    while (rs.next()) {
        System.out.println(rs.getInt("id") + " - " +
                           rs.getString("nombre") + " - " +
                           rs.getDouble("salario"));
    }

    DatabaseMetaData dbMeta = con.getMetaData();
    System.out.println("Conectado a: " + dbMeta.getDatabaseProductName());

} catch (Exception e) {
    e.printStackTrace();
}

```
## 6. Errores comunes

- Usar índices de columna incorrectos (recordar que empiezan en **1**, no en 0).
    
- No comprobar si `next()` devuelve `true` antes de leer valores.
    
- Confundir `ResultSetMetaData` con `DatabaseMetaData`.

## 7. Buenas prácticas

- Usar nombres de columna en `getXXX()` para mayor claridad (`rs.getString("nombre")`).
    
- Liberar siempre `ResultSet` y `Statement`.
    
- Usar `ResultSetMetaData` para consultas dinámicas (ej.: exportar resultados a CSV).

## 8. Actividad práctica

### Versión para alumnos

#### Objetivo

Aprender a usar **ResultSet**, **ResultSetMetaData** y **DatabaseMetaData** en JDBC.

#### Tareas

1. Conéctate a la base de datos `empresa`.
    
2. Ejecuta una consulta `SELECT * FROM empleados`.
    
3. Muestra por consola los datos de todos los empleados.
    
4. Usa `ResultSetMetaData` para imprimir:
    
    - Número de columnas.
        
    - Nombre y tipo de cada columna.
        
5. Usa `DatabaseMetaData` para imprimir:
    
    - Nombre del producto de base de datos.
        
    - Versión del motor.
        
    - Nombre del driver JDBC.

#### Entregables

- Código fuente en Java.
    
- Capturas de pantalla con la ejecución y los resultados mostrados en consola.

> [!Actividad 2.4]
> https://github.com/IrisCampusFP/ActividadesAccesoADatos/tree/main/UD2-JDBC_Avanzado/Act2.4