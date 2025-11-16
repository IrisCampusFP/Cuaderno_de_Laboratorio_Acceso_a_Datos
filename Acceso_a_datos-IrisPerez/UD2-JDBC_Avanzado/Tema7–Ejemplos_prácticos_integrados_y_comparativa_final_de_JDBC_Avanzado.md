## Introducción

Tras haber estudiado cada bloque de JDBC avanzado por separado, es fundamental practicar con **ejemplos integrados** que combinen:

- Creación y manipulación de datos con **DDL y DML**.
    
- Consultas con **Statement** y **PreparedStatement**.
    
- Uso de **procedimientos almacenados** con `CallableStatement`.
    
- Manejo de **transacciones** con `commit`, `rollback` y `savepoint`.
    
- Optimización con **pool de conexiones**.

Este tema presenta un caso práctico completo y termina con una comparativa de las técnicas vistas.

## 1. Caso práctico: Gestión de empleados y proyectos

Supongamos la BD `empresa`, con las tablas:

```MySQL
CREATE TABLE empleados (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100),
    salario DECIMAL(10,2)
);

CREATE TABLE proyectos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100),
    presupuesto DECIMAL(10,2)
);

CREATE TABLE asignaciones (
    id_empleado INT,
    id_proyecto INT,
    horas INT,
    PRIMARY KEY (id_empleado, id_proyecto),
    FOREIGN KEY (id_empleado) REFERENCES empleados(id),
    FOREIGN KEY (id_proyecto) REFERENCES proyectos(id)
);

```

## 2. Inserción y consulta con PreparedStatement

```java
String sql = "INSERT INTO proyectos (nombre, presupuesto) VALUES (?, ?)";
PreparedStatement ps = con.prepareStatement(sql);
ps.setString(1, "Proyecto X");
ps.setDouble(2, 50000.00);
ps.executeUpdate();
```

Consulta con parámetros:

```java
PreparedStatement ps2 = con.prepareStatement(
    "SELECT * FROM proyectos WHERE presupuesto > ?");
ps2.setDouble(1, 10000);
ResultSet rs = ps2.executeQuery();

while (rs.next()) {
    System.out.println(rs.getInt("id") + " - " +
                       rs.getString("nombre") + " - " +
                       rs.getDouble("presupuesto"));
}
```

## 3. Procedimiento almacenado

SQL en MySQL:

```MySQL
DELIMITER //
CREATE PROCEDURE asignar_empleado(
    IN empId INT,
    IN proyId INT,
    IN horasAsignadas INT
)
BEGIN
    INSERT INTO asignaciones (id_empleado, id_proyecto, horas)
    VALUES (empId, proyId, horasAsignadas);
END //
DELIMITER ;
```

Java con `CallableStatement`:

```java
CallableStatement cs = con.prepareCall("{call asignar_empleado(?, ?, ?)}");
cs.setInt(1, 1); // empleado
cs.setInt(2, 2); // proyecto
cs.setInt(3, 20); // horas
cs.execute();
```

## 4. Transacción con múltiples operaciones

```java
try {
    con.setAutoCommit(false);

    PreparedStatement ps1 = con.prepareStatement(
        "UPDATE empleados SET salario = salario + ? WHERE id = ?");
    ps1.setDouble(1, 300);
    ps1.setInt(2, 1);
    ps1.executeUpdate();

    PreparedStatement ps2 = con.prepareStatement(
        "UPDATE proyectos SET presupuesto = presupuesto - ? WHERE id = ?");
    ps2.setDouble(1, 300);
    ps2.setInt(2, 2);
    ps2.executeUpdate();

    con.commit();
    System.out.println("Transacción completada con éxito");

} catch (SQLException e) {
    con.rollback();
    System.out.println("Error: rollback ejecutado");
}
```

## 5. Uso de pool de conexiones

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/empresa");
config.setUsername("root");
config.setPassword("root");
config.setMaximumPoolSize(5);

HikariDataSource ds = new HikariDataSource(config);

try (Connection con = ds.getConnection()) {
    System.out.println("Conexión obtenida del pool");
}

```

## 6. Comparativa final

|Elemento|Uso principal|Ventajas|Limitaciones|
|---|---|---|---|
|`Statement`|Consultas simples|Fácil de usar|Inseguro, bajo rendimiento|
|`PreparedStatement`|Consultas parametrizadas|Seguro, rápido, reutilizable|Requiere más código|
|`CallableStatement`|Procedimientos almacenados|Reutilización, seguridad, rendimiento|Depende del SGBD|
|Transacciones|Operaciones dependientes|Consistencia, control de errores|Complejidad extra|
|Pool de conexiones|Escalabilidad|Reutilización, velocidad|Requiere configuración|

## 7. Actividad práctica

### Versión para alumnos

#### Objetivo

Aplicar todos los conocimientos de JDBC avanzado en un caso integrado.

#### Tareas

1. Crea la BD con las tablas `empleados`, `proyectos` y `asignaciones`.
    
2. Inserta empleados y proyectos con **PreparedStatement**.
    
3. Implementa un procedimiento almacenado para asignar empleados a proyectos.
    
4. Crea un programa que:
    
    - Ejecute el procedimiento almacenado.
        
    - Realice una transacción que incremente el salario de un empleado y descuente ese importe del presupuesto de un proyecto.
        
    - Use un **pool de conexiones** para obtener la conexión.
        
5. Muestra el estado final de las tablas en consola.

#### Entregables

- Script SQL.
    
- Código fuente en Java.
    
- Capturas de pantalla con la ejecución.

> [!Actividad 2.7]
> https://github.com/IrisCampusFP/ActividadesAccesoADatos/tree/main/UD2-JDBC_Avanzado/Act2.7