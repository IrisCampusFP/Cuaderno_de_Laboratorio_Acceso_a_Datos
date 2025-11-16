## Introducción

Uno de los usos más comunes de JDBC es la ejecución de sentencias SQL contra una base de datos. Estas sentencias pueden clasificarse en dos grandes grupos:

1. **DDL (Data Definition Language)**: sirven para **definir** o modificar la estructura de la base de datos (crear tablas, alterar columnas, borrar objetos, etc.).
    
    - Ejemplos: `CREATE`, `ALTER`, `DROP`.
        
2. **DML (Data Manipulation Language)**: permiten **manipular datos** dentro de las tablas.
    
    - Ejemplos: `INSERT`, `UPDATE`, `DELETE`, `SELECT`.

JDBC nos proporciona las clases `Statement` y `PreparedStatement` para ejecutar estas operaciones de forma sencilla y controlada.

## 1. Sentencias DDL con JDBC

Las operaciones DDL cambian la estructura de la base de datos. Normalmente se ejecutan con `Statement` o `PreparedStatement`, pero no devuelven resultados (`ResultSet`), sino que confirman si la ejecución fue correcta.

### Ejemplo: crear tabla

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;

public class CrearTabla {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/empresa";
        String usuario = "root";
        String password = "root";

        try (Connection con = DriverManager.getConnection(url, usuario, password);
             Statement stmt = con.createStatement()) {

            String sql = "CREATE TABLE departamentos (" +
                         "id INT AUTO_INCREMENT PRIMARY KEY, " +
                         "nombre VARCHAR(100) NOT NULL)";
            stmt.executeUpdate(sql);
            System.out.println("Tabla creada correctamente.");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 2. Sentencias DML con JDBC

Las operaciones DML modifican datos. En JDBC, para estos casos se usa principalmente `PreparedStatement`, ya que permite parametrizar consultas y evita **inyecciones SQL**.

### Ejemplo: insertar datos

```java
import java.sql.*;

public class InsertarEmpleado {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/empresa";
        String usuario = "root";
        String password = "root";

        try (Connection con = DriverManager.getConnection(url, usuario, password)) {
            String sql = "INSERT INTO empleados (nombre, salario) VALUES (?, ?)";
            PreparedStatement ps = con.prepareStatement(sql);

            ps.setString(1, "Carlos");
            ps.setDouble(2, 2100.00);
            int filas = ps.executeUpdate();

            System.out.println("Filas insertadas: " + filas);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

### Ejemplo: actualizar datos

```java
String sql = "UPDATE empleados SET salario = ? WHERE id = ?";
PreparedStatement ps = con.prepareStatement(sql);
ps.setDouble(1, 2200.00);
ps.setInt(2, 1);
int filas = ps.executeUpdate();
System.out.println("Filas actualizadas: " + filas);
```

### Ejemplo: eliminar datos

```java
String sql = "DELETE FROM empleados WHERE id = ?";
PreparedStatement ps = con.prepareStatement(sql);
ps.setInt(1, 4);
int filas = ps.executeUpdate();
System.out.println("Filas eliminadas: " + filas);
```

## 3. `executeQuery` vs `executeUpdate`

| Método          | Uso                                             | Devuelve                                 |
| --------------- | ----------------------------------------------- | ---------------------------------------- |
| `executeQuery`  | Consultas `SELECT`                              | Un `ResultSet` con los resultados        |
| `executeUpdate` | Operaciones `INSERT`, `UPDATE`, `DELETE`, `DDL` | Número de filas afectadas                |
| `execute`       | Ejecuta cualquier SQL                           | `true` si hay `ResultSet`, `false` si no |

## 4. Ejemplo integrado

```java
try (Connection con = DriverManager.getConnection(url, usuario, password)) {
    // INSERT
    PreparedStatement psInsert = con.prepareStatement(
        "INSERT INTO empleados (nombre, salario) VALUES (?, ?)");
    psInsert.setString(1, "Laura");
    psInsert.setDouble(2, 1900.00);
    psInsert.executeUpdate();

    // UPDATE
    PreparedStatement psUpdate = con.prepareStatement(
        "UPDATE empleados SET salario = ? WHERE nombre = ?");
    psUpdate.setDouble(1, 2000.00);
    psUpdate.setString(2, "Laura");
    psUpdate.executeUpdate();

    // DELETE
    PreparedStatement psDelete = con.prepareStatement(
        "DELETE FROM empleados WHERE nombre = ?");
    psDelete.setString(1, "Laura");
    psDelete.executeUpdate();

    System.out.println("Operaciones DML ejecutadas correctamente.");
}
```

## 5. Errores comunes

- Usar `Statement` concatenando strings en lugar de `PreparedStatement`.
    
- Olvidar cerrar los objetos `PreparedStatement`.
    
- Confundir `executeQuery` (solo para `SELECT`) con `executeUpdate` (para DML y DDL).

## 6. Buenas prácticas

- Usar siempre `PreparedStatement` para consultas con parámetros.
    
- Manejar excepciones con `try-catch-finally` o `try-with-resources`.
    
- Comprobar el número de filas afectadas en operaciones DML.

## 7. Actividad práctica

### Versión para alumnos

#### Objetivo

Practicar la ejecución de sentencias **DDL** y **DML** con JDBC.

#### Tareas

1. **Crear tabla `proyectos`** con los campos:
    
    - `id` (INT, autoincremental, clave primaria).
        
    - `nombre` (VARCHAR 100).
        
    - `presupuesto` (DECIMAL 10,2).
    
2. **Insertar tres proyectos** usando `PreparedStatement`.
    
3. **Actualizar el presupuesto** de uno de los proyectos.
    
4. **Eliminar un proyecto** por id.
    
5. **Listar todos los proyectos** con `SELECT` y mostrarlos por consola.

#### Entregables

- Código fuente en Java.
    
- Script SQL de la tabla.
    
- Capturas de pantalla de la ejecución mostrando las operaciones.

> [!Actividad 2.3]
> https://github.com/IrisCampusFP/ActividadesAccesoADatos/tree/main/UD2-JDBC_Avanzado/Act2.3