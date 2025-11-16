## OBJETIVO DE LA ACTIVIDAD

Crear un proyecto Maven en IntelliJ que:

1. Incluya la dependencia **MySQL Connector/J**.
    
2. Tenga una clase `TestConexion` que se conecte a una base de datos `empresa`.
    
3. Compile y ejecute correctamente con `mvn compile` / `mvn package`.
    
4. Muestre los metadatos de la conexión (versión del SGBD, usuario, etc.).

## ESTRUCTURA DE CARPETAS MAVEN

Después de crearlo en IntelliJ → “Nuevo Proyecto → Maven”, la estructura debe quedar así:

MiPrimerProyectoMaven/
│
├── pom.xml
├── src/
│   └── main/
│       ├── java/
│       │   └── conexion/
│       │       └── TestConexion.java
│       └── resources/
│           └── db.properties
└── scripts/
    └── empresa.sql

## 1. ARCHIVO `pom.xml`

Copia este contenido en la raíz del proyecto:

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.campusfp</groupId>
    <artifactId>MiPrimerProyectoMaven</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- Driver MySQL -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>8.0.33</version>
        </dependency>
    </dependencies>

</project>

✅ Explicación para clase:

- `groupId` → identifica al autor o la organización del proyecto.
    
- `artifactId` → nombre del proyecto.
    
- `dependencies` → librerías que Maven descargará automáticamente.

## 2. ARCHIVO `db.properties` (en `src/main/resources`)

db.url=jdbc:mysql://localhost:3306/empresa
db.user=root
db.password=root

💡 Cambiar los valores si tu MySQL usa otro usuario o contraseña. Este archivo se usa para separar la configuración de conexión del código fuente.

## 3. SCRIPT SQL (opcional para probar)

Guarda como `scripts/empresa.sql` y ejecútalo en MySQL Workbench o consola:

CREATE DATABASE IF NOT EXISTS empresa;
USE empresa;

CREATE TABLE empleados (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50),
    salario DOUBLE
);

INSERT INTO empleados (nombre, salario)
VALUES ('Ana', 25000), ('Luis', 28000), ('Marta', 32000);

## 4. CLASE `TestConexion.java`

Crea el paquete `conexion` dentro de `src/main/java` y pega este código completo:

package conexion;

import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

public class TestConexion {
    public static void main(String[] args) {
        // 1. Cargar configuración desde db.properties
        Properties props = new Properties();
        try (InputStream input = TestConexion.class.getClassLoader().getResourceAsStream("db.properties")) {
            if (input == null) {
                System.err.println("❌ No se encontró el archivo db.properties");
                return;
            }
            props.load(input);
        } catch (Exception e) {
            e.printStackTrace();
            return;
        }

        // 2. Obtener datos de conexión
        String url = props.getProperty("db.url");
        String user = props.getProperty("db.user");
        String password = props.getProperty("db.password");

        // 3. Probar conexión
        try (Connection con = DriverManager.getConnection(url, user, password)) {
            System.out.println("✅ Conexión establecida con éxito a la base de datos.");
            
            // Mostrar metadatos
            DatabaseMetaData meta = con.getMetaData();
            System.out.println("🔹 Driver: " + meta.getDriverName());
            System.out.println("🔹 Versión del driver: " + meta.getDriverVersion());
            System.out.println("🔹 Base de datos: " + meta.getDatabaseProductName());
            System.out.println("🔹 Versión BD: " + meta.getDatabaseProductVersion());
            System.out.println("🔹 Usuario conectado: " + meta.getUserName());
            System.out.println("🔹 URL de conexión: " + meta.getURL());

            // 4. Consulta de ejemplo
            Statement st = con.createStatement();
            ResultSet rs = st.executeQuery("SELECT * FROM empleados");

            System.out.println("\n=== EMPLEADOS ===");
            while (rs.next()) {
                System.out.printf("ID: %d | Nombre: %s | Salario: %.2f €%n",
                        rs.getInt("id"), rs.getString("nombre"), rs.getDouble("salario"));
            }

        } catch (SQLException e) {
            System.err.println("❌ Error al conectar a la base de datos: " + e.getMessage());
        }
    }
}

## 5. EJECUCIÓN EN INTELLIJ IDEA

1. Clic derecho sobre el proyecto → **Add as Maven Project** (si no lo está ya).
    
2. Espera que descargue el conector MySQL (verás en consola de Maven).
    
3. Clic derecho sobre la clase `TestConexion.java` → **Run “TestConexion.main()”**
    
4. En la consola de IntelliJ deberías ver algo como:

✅ Conexión establecida con éxito a la base de datos.
🔹 Driver: MySQL Connector/J
🔹 Versión del driver: 9.0.0
🔹 Base de datos: MySQL
🔹 Versión BD: 8.0.36
🔹 Usuario conectado: root@localhost
🔹 URL de conexión: jdbc:mysql://localhost:3306/empresa

=== EMPLEADOS ===
ID: 1 | Nombre: Ana | Salario: 25000.00 €
ID: 2 | Nombre: Luis | Salario: 28000.00 €
ID: 3 | Nombre: Marta | Salario: 32000.00 €

## 6. OPCIONAL – Ejecutar desde terminal (para ver Maven en acción)

Desde IntelliJ o PowerShell, sitúate en la raíz del proyecto y ejecuta:

mvn compile
mvn package

Maven creará la carpeta `target/` con el `.class` y el `.jar` del proyecto. Puedes ejecutar el `.jar` con:

java -cp target/MiPrimerProyectoMaven-1.0-SNAPSHOT.jar;target/dependency/* conexion.TestConexion

## 7. Entregar la actividad en PDF y el enlace del repositorio a GitHub píblico.

> [!Mi primer proyecto Maven]
> https://github.com/IrisCampusFP/ActividadesAccesoADatos/tree/main/UD2-JDBC_Avanzado/MiPrimerProyectoMaven