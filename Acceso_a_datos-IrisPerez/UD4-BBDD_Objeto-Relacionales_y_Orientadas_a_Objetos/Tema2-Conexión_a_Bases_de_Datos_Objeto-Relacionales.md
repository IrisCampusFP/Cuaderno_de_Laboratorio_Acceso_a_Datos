## CE 4b: Establecer y cerrar conexiones

En este tema aprenderemos a configurar y gestionar conexiones a Oracle, PostgreSQL y ObjectDB utilizando JDBC y JPA.

## 2.1 Arquitecturas de Conexión

Existen dos modelos principales de conexión:

### Modo Embebido

La base de datos se ejecuta dentro del mismo proceso que la aplicación.

- **Ventajas**: Sin configuración de red, menor latencia
    
- **Desventajas**: Un solo proceso puede acceder a la vez
    
- **Ejemplo**: ObjectDB en modo embebido, H2, SQLite

### Modo Cliente-Servidor

La base de datos se ejecuta como un servicio independiente.

- **Ventajas**: Múltiples clientes, escalabilidad
    
- **Desventajas**: Configuración de red, latencia
    
- **Ejemplo**: Oracle, PostgreSQL, ObjectDB Server

## 2.2 Configuración Maven

Para trabajar con las tres tecnologías, necesitamos las siguientes dependencias:

<dependencies>
    <!-- Driver Oracle -->
    <dependency>
        <groupId>com.oracle.database.jdbc</groupId>
        <artifactId>ojdbc11</artifactId>
        <version>23.3.0.23.09</version>
    </dependency>
    
    <!-- Driver PostgreSQL -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.7.1</version>
    </dependency>
    
    <!-- ObjectDB (incluye JPA) -->
    <dependency>
        <groupId>com.objectdb</groupId>
        <artifactId>objectdb</artifactId>
        <version>2.8.9</version>
    </dependency>
    
    <!-- JPA API -->
    <dependency>
        <groupId>jakarta.persistence</groupId>
        <artifactId>jakarta.persistence-api</artifactId>
        <version>3.1.0</version>
    </dependency>
</dependencies>

## 2.3 Conexión a Oracle Database

### Cadena de Conexión

Oracle soporta varios formatos de URL JDBC:

# Formato básico (SID)
jdbc:oracle:thin:@host:puerto:SID

# Formato con Service Name
jdbc:oracle:thin:@//host:puerto/serviceName

# Ejemplo para Oracle XE local
jdbc:oracle:thin:@localhost:1521:XE
jdbc:oracle:thin:@//localhost:1521/XEPDB1

### Ejemplo de Conexión JDBC a Oracle

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConexionOracle {
    
    private static final String URL = "jdbc:oracle:thin:@localhost:1521:XE";
    private static final String USUARIO = "biblioteca";
    private static final String PASSWORD = "biblioteca";
    
    public static Connection obtenerConexion() throws SQLException {
        return DriverManager.getConnection(URL, USUARIO, PASSWORD);
    }
    
    public static void main(String[] args) {
        try (Connection conn = obtenerConexion()) {
            System.out.println("Conexión establecida con Oracle");
            System.out.println("Versión: " + conn.getMetaData().getDatabaseProductVersion());
        } catch (SQLException e) {
            System.err.println("Error de conexión: " + e.getMessage());
        }
    }
}

## 2.4 Conexión a PostgreSQL

### Cadena de Conexión

# Formato básico
jdbc:postgresql://host:puerto/basedatos

# Con parámetros
jdbc:postgresql://host:puerto/basedatos?parametro=valor

# Ejemplo local
jdbc:postgresql://localhost:5432/biblioteca

### Ejemplo de Conexión JDBC a PostgreSQL

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConexionPostgreSQL {
    
    private static final String URL = "jdbc:postgresql://localhost:5432/biblioteca";
    private static final String USUARIO = "postgres";
    private static final String PASSWORD = "postgres";
    
    public static Connection obtenerConexion() throws SQLException {
        return DriverManager.getConnection(URL, USUARIO, PASSWORD);
    }
    
    public static void main(String[] args) {
        try (Connection conn = obtenerConexion()) {
            System.out.println("Conexión establecida con PostgreSQL");
            System.out.println("Versión: " + conn.getMetaData().getDatabaseProductVersion());
        } catch (SQLException e) {
            System.err.println("Error de conexión: " + e.getMessage());
        }
    }
}

## 2.5 Conexión a ObjectDB

ObjectDB utiliza JPA como API principal. Soporta dos modos:

### Modo Embebido

La base de datos es un fichero local `.odb`:

// persistence.xml
<persistence-unit name="biblioteca-odb">
    <provider>com.objectdb.jpa.Provider</provider>
    <properties>
        <property name="javax.persistence.jdbc.url" 
                  value="objectdb:biblioteca.odb"/>
    </properties>
</persistence-unit>

### Modo Cliente-Servidor

Conexión a un servidor ObjectDB remoto:

<property name="javax.persistence.jdbc.url" 
          value="objectdb://localhost:6136/biblioteca.odb"/>

### Ejemplo de Conexión JPA a ObjectDB

import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.Persistence;

public class ConexionObjectDB {
    
    public static void main(String[] args) {
        EntityManagerFactory emf = null;
        EntityManager em = null;
        
        try {
            // Crear factory (costoso, una vez por aplicación)
            emf = Persistence.createEntityManagerFactory("biblioteca-odb");
            
            // Crear EntityManager (una por unidad de trabajo)
            em = emf.createEntityManager();
            
            System.out.println("Conexión establecida con ObjectDB");
            
        } finally {
            if (em != null) em.close();
            if (emf != null) emf.close();
        }
    }
}

## 2.6 Conexión JPA Unificada

La ventaja de JPA es que podemos usar la misma API para diferentes bases de datos. El fichero `persistence.xml` determina la conexión:

<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="https://jakarta.ee/xml/ns/persistence"
             version="3.0">
    
    <!-- Unidad para ObjectDB -->
    <persistence-unit name="biblioteca-objectdb">
        <provider>com.objectdb.jpa.Provider</provider>
        <class>com.biblioteca.modelo.Producto</class>
        <properties>
            <property name="javax.persistence.jdbc.url" 
                      value="objectdb:biblioteca.odb"/>
        </properties>
    </persistence-unit>
    
    <!-- Unidad para Oracle -->
    <persistence-unit name="biblioteca-oracle">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <class>com.biblioteca.modelo.Producto</class>
        <properties>
            <property name="jakarta.persistence.jdbc.driver" 
                      value="oracle.jdbc.OracleDriver"/>
            <property name="jakarta.persistence.jdbc.url" 
                      value="jdbc:oracle:thin:@localhost:1521:XE"/>
            <property name="jakarta.persistence.jdbc.user" value="biblioteca"/>
            <property name="jakarta.persistence.jdbc.password" value="biblioteca"/>
            <property name="hibernate.dialect" 
                      value="org.hibernate.dialect.OracleDialect"/>
        </properties>
    </persistence-unit>
    
    <!-- Unidad para PostgreSQL -->
    <persistence-unit name="biblioteca-postgresql">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <class>com.biblioteca.modelo.Producto</class>
        <properties>
            <property name="jakarta.persistence.jdbc.driver" 
                      value="org.postgresql.Driver"/>
            <property name="jakarta.persistence.jdbc.url" 
                      value="jdbc:postgresql://localhost:5432/biblioteca"/>
            <property name="jakarta.persistence.jdbc.user" value="postgres"/>
            <property name="jakarta.persistence.jdbc.password" value="postgres"/>
            <property name="hibernate.dialect" 
                      value="org.hibernate.dialect.PostgreSQLDialect"/>
        </properties>
    </persistence-unit>
    
</persistence>

## 2.7 Cierre Correcto de Conexiones

El cierre correcto de conexiones es **fundamental** para evitar:

- **Memory leaks**: Recursos no liberados
    
- **Connection exhaustion**: Pool de conexiones agotado
    
- **Datos inconsistentes**: Transacciones no cerradas
    

### Patrón try-with-resources (JDBC)

// El try-with-resources cierra automáticamente
try (Connection conn = DriverManager.getConnection(URL, USER, PASS);
     PreparedStatement stmt = conn.prepareStatement(SQL);
     ResultSet rs = stmt.executeQuery()) {
    
    while (rs.next()) {
        // procesar resultados
    }
} catch (SQLException e) {
    // manejar error
}
// Connection, Statement y ResultSet se cierran automáticamente

### Patrón try-finally (JPA)

EntityManagerFactory emf = null;
EntityManager em = null;

try {
    emf = Persistence.createEntityManagerFactory("mi-unidad");
    em = emf.createEntityManager();
    
    em.getTransaction().begin();
    // operaciones
    em.getTransaction().commit();
    
} catch (Exception e) {
    if (em != null && em.getTransaction().isActive()) {
        em.getTransaction().rollback();
    }
    throw e;
} finally {
    if (em != null) em.close();
    if (emf != null) emf.close();
}

## 2.8 Connection Pooling

En aplicaciones de producción, se recomienda usar un **pool de conexiones** para reutilizar conexiones:

### HikariCP (Recomendado)

<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.1.0</version>
</dependency>

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

public class PoolConexiones {
    private static HikariDataSource dataSource;
    
    static {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://localhost:5432/biblioteca");
        config.setUsername("postgres");
        config.setPassword("postgres");
        config.setMaximumPoolSize(10);
        config.setMinimumIdle(2);
        
        dataSource = new HikariDataSource(config);
    }
    
    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
}

## 2.9 Resumen del Tema

Conceptos Clave

1. **Oracle** usa driver `ojdbc11` y URL `jdbc:oracle:thin:@host:puerto:SID`
    
2. **PostgreSQL** usa driver `postgresql` y URL `jdbc:postgresql://host:puerto/db`
    
3. **ObjectDB** usa JPA directamente con URL `objectdb:fichero.odb`
    
4. **JPA** permite usar la misma API para diferentes bases de datos
    
5. Siempre usar **try-with-resources** o **try-finally** para cerrar conexiones
    
6. En produccion, usar **connection pooling** (HikariCP)