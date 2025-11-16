# Bloque 1: Arquitectura JDBC y Drivers (2h)

## Objetivos del Bloque

- Comprender la arquitectura en capas de JDBC
    
- Conocer los 4 tipos de drivers JDBC
    
- Aprender a obtener metadatos de la base de datos
    
- Configurar la conexión para el proyecto TechDAM

## Teoría: ¿Qué es JDBC?

### Definición

**JDBC (Java Database Connectivity)** es una API de Java que permite:

- 🔗 Conectar aplicaciones Java con bases de datos
    
- 📤 Enviar consultas SQL
    
- 📥 Procesar resultados

### Arquitectura JDBC en Capas

┌──────────────────────────────────────┐
│     APLICACIÓN JAVA                  │  ← Tu código
│  (Lógica de negocio)                 │
└──────────────────────────────────────┘
             ↕
┌──────────────────────────────────────┐
│         JDBC API                     │  ← Java SE
│  (java.sql.*, javax.sql.*)           │
└──────────────────────────────────────┘
             ↕
┌──────────────────────────────────────┐
│      JDBC DRIVER MANAGER             │  ← Gestiona drivers
└──────────────────────────────────────┘
             ↕
┌──────────────────────────────────────┐
│      JDBC DRIVER                     │  ← mysql-connector-java
│  (Implementación específica)         │
└──────────────────────────────────────┘
             ↕
┌──────────────────────────────────────┐
│      BASE DE DATOS                   │  ← MySQL, PostgreSQL...
│      (MySQL, Oracle, etc.)           │
└──────────────────────────────────────┘

## 🔌 Tipos de Drivers JDBC

### Comparativa de los 4 Tipos

|Tipo|Nombre|Descripción|Uso Actual|
|---|---|---|---|
|**Tipo 1**|JDBC-ODBC Bridge|Traduce JDBC a ODBC|❌ Obsoleto (eliminado en Java 8)|
|**Tipo 2**|Native-API|Convierte JDBC a llamadas nativas del DB|⚠️ Poco usado (dependiente de SO)|
|**Tipo 3**|Network Protocol|Usa servidor middleware|⚠️ Poco común (complejidad añadida)|
|**Tipo 4**|Thin Driver|100% Java, comunicación directa|✅ **MÁS USADO** (recomendado)|

### Tipo 4: Thin Driver (El que usaremos)

**Ventajas:**

- ✅ 100% Java (portable)
    
- ✅ No requiere software adicional
    
- ✅ Mejor rendimiento
    
- ✅ Fácil de configurar
    

**Ejemplo**: `mysql-connector-java` es un driver Tipo 4


## 💻 Práctica 1: Configuración Inicial

### Paso 1: Crear el proyecto Maven

#### `pom.xml`

%%bash
cat > pom.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.dam</groupId>
    <artifactId>techdam-jdbc</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- MySQL JDBC Driver (Tipo 4) -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.33</version>
        </dependency>

        <!-- HikariCP (Pool de Conexiones) -->
        <dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
            <version>5.0.1</version>
        </dependency>

        <!-- SLF4J (Logging) -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>2.0.7</version>
        </dependency>
    </dependencies>
</project>
EOF

### Paso 2: Clase de Configuración de Conexión

Vamos a crear una clase `DatabaseConfig` que gestione la configuración de conexión de forma centralizada.

#### `src/main/java/com/techdam/config/DatabaseConfig.java`

package com.techdam.config;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

/**
 * Configuración de conexión a la base de datos TechDAM
 * 
 * @author DAM - Acceso a Datos
 * @version 1.0
 */
public class DatabaseConfig {
    
    // Configuración de conexión
    private static final String URL = "jdbc:mysql://localhost:3306/techdam";
    private static final String USER = "root";
    private static final String PASSWORD = "";
    
    // Parámetros adicionales de conexión
    private static final String PARAMS = "?useSSL=false&serverTimezone=Europe/Madrid&allowPublicKeyRetrieval=true";
    
    /**
     * Obtiene una conexión a la base de datos
     * 
     * @return Connection - Conexión activa
     * @throws SQLException si hay error de conexión
     */
    public static Connection getConnection() throws SQLException {
        try {
            // Cargar el driver (opcional desde JDBC 4.0)
            Class.forName("com.mysql.cj.jdbc.Driver");
            
            // Establecer conexión
            Connection conn = DriverManager.getConnection(URL + PARAMS, USER, PASSWORD);
            
            System.out.println("✅ Conexión establecida con éxito");
            return conn;
            
        } catch (ClassNotFoundException e) {
            System.err.println("❌ Error: Driver MySQL no encontrado");
            throw new SQLException("Driver no encontrado", e);
        }
    }
    
    /**
     * Cierra una conexión de forma segura
     * 
     * @param conn Conexión a cerrar
     */
    public static void closeConnection(Connection conn) {
        if (conn != null) {
            try {
                conn.close();
                System.out.println("🔒 Conexión cerrada");
            } catch (SQLException e) {
                System.err.println("⚠️ Error al cerrar conexión: " + e.getMessage());
            }
        }
    }
}

### Paso 3: Probar la Conexión

#### `src/main/java/com/techdam/TestConexion.java`

package com.techdam;

import com.techdam.config.DatabaseConfig;
import java.sql.Connection;
import java.sql.SQLException;

public class TestConexion {
    
    public static void main(String[] args) {
        Connection conn = null;
        
        try {
            System.out.println("🔄 Intentando conectar a la base de datos...");
            
            conn = DatabaseConfig.getConnection();
            
            System.out.println("📊 Base de datos: " + conn.getCatalog());
            System.out.println("🔗 URL: " + conn.getMetaData().getURL());
            System.out.println("👤 Usuario: " + conn.getMetaData().getUserName());
            
        } catch (SQLException e) {
            System.err.println("❌ Error de conexión: " + e.getMessage());
            e.printStackTrace();
        } finally {
            DatabaseConfig.closeConnection(conn);
        }
    }
}

## 🗃️ Práctica 2: Metadatos de la Base de Datos

### ¿Qué son los Metadatos?

Los **metadatos** son «datos sobre los datos». Nos permiten obtener información sobre:

- Tablas disponibles
    
- Claves primarias y foráneas
    
- Tipos de datos de columnas
    
- Índices
    
- Información del servidor
    

### Clase DatabaseMetaData

#### `src/main/java/com/techdam/util/MetadataExplorer.java`

package com.techdam.util;

import com.techdam.config.DatabaseConfig;
import java.sql.*;

/**
 * Explorador de metadatos de la base de datos
 */
public class MetadataExplorer {
    
    /**
     * Muestra información del servidor de base de datos
     */
    public static void mostrarInfoServidor() {
        Connection conn = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            DatabaseMetaData metadata = conn.getMetaData();
            
            System.out.println("\n========================================");
            System.out.println("📊 INFORMACIÓN DEL SERVIDOR");
            System.out.println("========================================");
            
            System.out.println("🏢 Nombre del DBMS: " + metadata.getDatabaseProductName());
            System.out.println("📌 Versión del DBMS: " + metadata.getDatabaseProductVersion());
            System.out.println("🔧 Versión del Driver: " + metadata.getDriverVersion());
            System.out.println("📍 URL de conexión: " + metadata.getURL());
            System.out.println("👤 Usuario conectado: " + metadata.getUserName());
            
            // Capacidades del driver
            System.out.println("\n✨ CAPACIDADES DEL DRIVER:");
            System.out.println("  ✓ Soporta transacciones: " + metadata.supportsTransactions());
            System.out.println("  ✓ Soporta savepoints: " + metadata.supportsSavepoints());
            System.out.println("  ✓ Soporta batch updates: " + metadata.supportsBatchUpdates());
            System.out.println("  ✓ Soporta procedimientos almacenados: " + 
                             metadata.supportsStoredProcedures());
            
        } catch (SQLException e) {
            System.err.println("❌ Error al obtener metadatos: " + e.getMessage());
        } finally {
            DatabaseConfig.closeConnection(conn);
        }
    }
    
    /**
     * Lista todas las tablas de la base de datos
     */
    public static void listarTablas() {
        Connection conn = null;
        ResultSet rs = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            DatabaseMetaData metadata = conn.getMetaData();
            
            System.out.println("\n========================================");
            System.out.println("📋 TABLAS EN LA BASE DE DATOS");
            System.out.println("========================================");
            
            // Obtener tablas del usuario actual
            rs = metadata.getTables(null, null, "%", new String[]{"TABLE"});
            
            int count = 0;
            while (rs.next()) {
                count++;
                String tableName = rs.getString("TABLE_NAME");
                String tableType = rs.getString("TABLE_TYPE");
                String remarks = rs.getString("REMARKS");
                
                System.out.println("\n📊 Tabla #" + count + ": " + tableName);
                System.out.println("   Tipo: " + tableType);
                if (remarks != null && !remarks.isEmpty()) {
                    System.out.println("   Comentario: " + remarks);
                }
            }
            
            System.out.println("\n✅ Total de tablas: " + count);
            
        } catch (SQLException e) {
            System.err.println("❌ Error al listar tablas: " + e.getMessage());
        } finally {
            try {
                if (rs != null) rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
            DatabaseConfig.closeConnection(conn);
        }
    }
    
    /**
     * Muestra la estructura de una tabla específica
     * 
     * @param nombreTabla Nombre de la tabla a explorar
     */
    public static void mostrarEstructuraTabla(String nombreTabla) {
        Connection conn = null;
        ResultSet rs = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            DatabaseMetaData metadata = conn.getMetaData();
            
            System.out.println("\n========================================");
            System.out.println("🔍 ESTRUCTURA DE LA TABLA: " + nombreTabla);
            System.out.println("========================================");
            
            // Obtener columnas
            rs = metadata.getColumns(null, null, nombreTabla, "%");
            
            System.out.println("\n📋 COLUMNAS:");
            System.out.println(String.format("%-20s %-15s %-10s %-8s %-10s",
                "Columna", "Tipo", "Tamaño", "Nulo", "Default"));
            System.out.println("-".repeat(70));
            
            while (rs.next()) {
                String columnName = rs.getString("COLUMN_NAME");
                String columnType = rs.getString("TYPE_NAME");
                int columnSize = rs.getInt("COLUMN_SIZE");
                String isNullable = rs.getString("IS_NULLABLE");
                String defaultValue = rs.getString("COLUMN_DEF");
                
                System.out.println(String.format("%-20s %-15s %-10d %-8s %-10s",
                    columnName, columnType, columnSize, isNullable, 
                    defaultValue != null ? defaultValue : "-"));
            }
            rs.close();
            
            // Obtener claves primarias
            rs = metadata.getPrimaryKeys(null, null, nombreTabla);
            System.out.println("\n🔑 CLAVES PRIMARIAS:");
            while (rs.next()) {
                System.out.println("   " + rs.getString("COLUMN_NAME"));
            }
            
        } catch (SQLException e) {
            System.err.println("❌ Error al obtener estructura: " + e.getMessage());
        } finally {
            try {
                if (rs != null) rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
            DatabaseConfig.closeConnection(conn);
        }
    }
    
    public static void main(String[] args) {
        mostrarInfoServidor();
        listarTablas();
        
        // Si ya tienes la tabla empleados creada:
        // mostrarEstructuraTabla("empleados");
    }
}

## Ejercicio Guiado 1: Primera Conexión a TechDAM

### Objetivo

Crear la base de datos `techdam` y probar la conexión.

### Paso 1: Crear la Base de Datos

-- Script SQL: crear_bd_techdam.sql

-- Eliminar la BD si existe
DROP DATABASE IF EXISTS techdam;

-- Crear la BD
CREATE DATABASE techdam CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Usar la BD
USE techdam;

-- Mensaje de confirmación
SELECT 'Base de datos techdam creada con éxito' AS Mensaje;

### Paso 2: Ejecutar desde línea de comandos

mysql -u root -p < crear_bd_techdam.sql

### Paso 3: Ejecutar TestConexion.java

mvn clean compile
mvn exec:java -Dexec.mainClass="com.techdam.TestConexion"

### Resultado Esperado

🔄 Intentando conectar a la base de datos...
✅ Conexión establecida con éxito
📊 Base de datos: techdam
🔗 URL: jdbc:mysql://localhost:3306/techdam
👤 Usuario: root@localhost
🔒 Conexión cerrada

## 📝 Ejercicio de Refuerzo 1

### Enunciado

Modifica la clase `MetadataExplorer` para añadir un método que:

1. Muestre todas las claves foráneas (Foreign Keys) de una tabla
    
2. Indique a qué tabla y columna apunta cada FK

**Pista**: Usa el método `metadata.getImportedKeys()`

### Solución (Expandir cuando lo hayas intentado)

👉 Ver solución

public static void mostrarClavesForaneas(String nombreTabla) {
    Connection conn = null;
    ResultSet rs = null;
    
    try {
        conn = DatabaseConfig.getConnection();
        DatabaseMetaData metadata = conn.getMetaData();
        
        System.out.println("\n🔗 CLAVES FORÁNEAS DE: " + nombreTabla);
        
        rs = metadata.getImportedKeys(null, null, nombreTabla);
        
        while (rs.next()) {
            String fkColumnName = rs.getString("FKCOLUMN_NAME");
            String pkTableName = rs.getString("PKTABLE_NAME");
            String pkColumnName = rs.getString("PKCOLUMN_NAME");
            
            System.out.println("  " + fkColumnName + " -> " + 
                             pkTableName + "." + pkColumnName);
        }
        
    } catch (SQLException e) {
        System.err.println("Error: " + e.getMessage());
    } finally {
        try {
            if (rs != null) rs.close();
        } catch (SQLException e) {}
        DatabaseConfig.closeConnection(conn);
    }
}

## Evaluación del Bloque 1

### Checklist de Aprendizaje

Marca lo que ya dominas:

- [ ] Explico qué es JDBC y su arquitectura en capas
    
- [ ] Diferencio los 4 tipos de drivers JDBC
    
- [ ] Sé por qué usamos un driver Tipo 4
    
- [ ] Puedo obtener una conexión con `DriverManager.getConnection()`
    
- [ ] Entiendo qué son los metadatos de una BD
    
- [ ] Sé usar `DatabaseMetaData` para obtener información
    
- [ ] He configurado correctamente el proyecto TechDAM

## Resumen del Bloque

### Conceptos Clave

1. **JDBC** es la API estándar de Java para acceso a BD
    
2. Los **drivers Tipo 4** son los más recomendados (100% Java)
    
3. **DatabaseMetaData** proporciona información sobre la estructura de la BD
    
4. Es importante **cerrar siempre las conexiones** para liberar recursos

### Código Reutilizable Creado

- ✅ `DatabaseConfig`: Clase de configuración de conexión
    
- ✅ `MetadataExplorer`: Explorador de metadatos
    
- ✅ `TestConexion`: Prueba de conexión básica

## Siguiente Paso

En el **Bloque 2** compararemos JDBC básico vs avanzado y entenderemos por qué necesitamos técnicas avanzadas.

**→ [Ir al Bloque 2: JDBC Básico vs Avanzado](https://campusfp.dvsweb.es/accesoDatos/ud2/content/Proyecto/02_jdbc_basico_vs_avanzado.html)**

## Notas del Profesor

> **Buena práctica**: Siempre usa try-with-resources o bloques finally para cerrar conexiones. Una conexión abierta consume recursos del servidor.

> **Consejo**: Guarda las credenciales de BD en archivos de configuración externos (`.properties`) en lugar de hardcodearlas en el código.