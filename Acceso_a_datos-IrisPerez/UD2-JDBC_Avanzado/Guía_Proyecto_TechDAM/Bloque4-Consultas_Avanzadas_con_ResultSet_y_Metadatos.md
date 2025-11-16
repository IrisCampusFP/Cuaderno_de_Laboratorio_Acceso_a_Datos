## 🎯 Objetivos del Bloque

- Dominar ResultSetMetaData para consultas dinámicas
    
- Usar DatabaseMetaData para introspección de BD
    
- Trabajar con diferentes tipos de ResultSet
    
- Implementar consultas flexibles y genéricas
    
- Crear informes dinámicos en TechDAM


## 📖 Teoría: Metadatos

### ¿Qué son los Metadatos?

**Metadatos** = «Datos sobre los datos»

DATOS:      Juan, García, 35000
METADATOS:  Columna 1: nombre (VARCHAR, 50)
            Columna 2: apellido (VARCHAR, 50)
            Columna 3: salario (DECIMAL, 10,2)

### Tipos de Metadatos en JDBC

1. **DatabaseMetaData**: Información de la base de datos
    
    - Tablas
        
    - Columnas
        
    - Índices
        
    - Claves foráneas
        
    - Procedimientos
    
2. **ResultSetMetaData**: Información del resultado de una consulta
    
    - Número de columnas
        
    - Nombres de columnas
        
    - Tipos de datos
        
    - Tamaño de columnas


## 💻 Práctica 1: ResultSetMetaData

### Consulta Genérica que Imprime Cualquier Tabla

package com.techdam.util;

import com.techdam.config.DatabaseConfig;
import java.sql.*;

/**
 * Utilidad para imprimir cualquier consulta de forma dinámica
 */
public class ConsultaDinamica {
    
    /**
     * Ejecuta una consulta e imprime los resultados en formato tabla
     * 
     * @param sql Consulta SELECT a ejecutar
     */
    public static void imprimirConsulta(String sql) {
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            pstmt = conn.prepareStatement(sql);
            rs = pstmt.executeQuery();
            
            // Obtener metadatos del ResultSet
            ResultSetMetaData rsmd = rs.getMetaData();
            int numColumnas = rsmd.getColumnCount();
            
            // Imprimir encabezados
            System.out.println("\n" + "=".repeat(80));
            for (int i = 1; i <= numColumnas; i++) {
                System.out.printf("%-20s", rsmd.getColumnLabel(i));
            }
            System.out.println("\n" + "-".repeat(80));
            
            // Imprimir filas
            int filaNum = 0;
            while (rs.next()) {
                filaNum++;
                for (int i = 1; i <= numColumnas; i++) {
                    Object valor = rs.getObject(i);
                    System.out.printf("%-20s", 
                        valor != null ? valor.toString() : "NULL");
                }
                System.out.println();
            }
            
            System.out.println("=".repeat(80));
            System.out.println("Total: " + filaNum + " filas\n");
            
        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
        } finally {
            try {
                if (rs != null) rs.close();
                if (pstmt != null) pstmt.close();
            } catch (SQLException e) {}
            DatabaseConfig.closeConnection(conn);
        }
    }
    
    /**
     * Muestra información detallada de las columnas
     */
    public static void analizarColumnas(String sql) {
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            pstmt = conn.prepareStatement(sql);
            rs = pstmt.executeQuery();
            
            ResultSetMetaData rsmd = rs.getMetaData();
            int numColumnas = rsmd.getColumnCount();
            
            System.out.println("\n📊 ANÁLISIS DE COLUMNAS");
            System.out.println("=".repeat(100));
            
            for (int i = 1; i <= numColumnas; i++) {
                System.out.println("\n🔹 Columna #" + i);
                System.out.println("   Nombre:           " + rsmd.getColumnName(i));
                System.out.println("   Label:            " + rsmd.getColumnLabel(i));
                System.out.println("   Tipo SQL:         " + rsmd.getColumnTypeName(i));
                System.out.println("   Tipo Java:        " + rsmd.getColumnClassName(i));
                System.out.println("   Tamaño:           " + rsmd.getColumnDisplaySize(i));
                System.out.println("   Precisión:        " + rsmd.getPrecision(i));
                System.out.println("   Escala:           " + rsmd.getScale(i));
                System.out.println("   Permite NULL:     " + 
                    (rsmd.isNullable(i) == ResultSetMetaData.columnNullable ? "Sí" : "No"));
                System.out.println("   Auto-incremento:  " + rsmd.isAutoIncrement(i));
                System.out.println("   Solo lectura:     " + rsmd.isReadOnly(i));
                System.out.println("   Tabla origen:     " + rsmd.getTableName(i));
                System.out.println("   Catálogo:         " + rsmd.getCatalogName(i));
            }
            
            System.out.println("\n" + "=".repeat(100));
            
        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
        } finally {
            try {
                if (rs != null) rs.close();
                if (pstmt != null) pstmt.close();
            } catch (SQLException e) {}
            DatabaseConfig.closeConnection(conn);
        }
    }
    
    public static void main(String[] args) {
        // Ejemplo 1: Imprimir empleados
        imprimirConsulta("SELECT * FROM empleados");
        
        // Ejemplo 2: Consulta con JOIN
        imprimirConsulta(
            "SELECT e.nombre, e.apellido, p.nombre AS proyecto " +
            "FROM empleados e " +
            "INNER JOIN asignaciones a ON e.id = a.empleado_id " +
            "INNER JOIN proyectos p ON a.proyecto_id = p.id"
        );
        
        // Ejemplo 3: Análisis de columnas
        analizarColumnas("SELECT * FROM empleados LIMIT 1");
    }
}

## 🗃️ Práctica 2: DatabaseMetaData Avanzado

### Explorador Completo de Base de Datos

package com.techdam.util;

import com.techdam.config.DatabaseConfig;
import java.sql.*;

/**
 * Explorador avanzado de metadatos de la base de datos
 */
public class ExploradorBD {
    
    /**
     * Genera un informe completo de la estructura de la BD
     */
    public static void generarInformeCompleto() {
        Connection conn = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            DatabaseMetaData dbmd = conn.getMetaData();
            
            System.out.println("\n" + "=".repeat(80));
            System.out.println("📊 INFORME COMPLETO DE LA BASE DE DATOS TECHDAM");
            System.out.println("=".repeat(80));
            
            // 1. Información general
            mostrarInfoGeneral(dbmd);
            
            // 2. Listar todas las tablas
            listarTablasDetallado(dbmd);
            
            // 3. Analizar cada tabla
            analizarTodasLasTablas(dbmd);
            
            // 4. Procedimientos almacenados
            listarProcedimientos(dbmd);
            
        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
        } finally {
            DatabaseConfig.closeConnection(conn);
        }
    }
    
    private static void mostrarInfoGeneral(DatabaseMetaData dbmd) throws SQLException {
        System.out.println("\n🏢 INFORMACIÓN GENERAL");
        System.out.println("-".repeat(80));
        System.out.println("Producto:              " + dbmd.getDatabaseProductName());
        System.out.println("Versión:               " + dbmd.getDatabaseProductVersion());
        System.out.println("Driver:                " + dbmd.getDriverName());
        System.out.println("Versión Driver:        " + dbmd.getDriverVersion());
        System.out.println("URL:                   " + dbmd.getURL());
        System.out.println("Usuario:               " + dbmd.getUserName());
        System.out.println("Solo lectura:          " + dbmd.isReadOnly());
        System.out.println("Transacciones:         " + dbmd.supportsTransactions());
        System.out.println("Savepoints:            " + dbmd.supportsSavepoints());
        System.out.println("Batch Updates:         " + dbmd.supportsBatchUpdates());
        System.out.println("Stored Procedures:     " + dbmd.supportsStoredProcedures());
    }
    
    private static void listarTablasDetallado(DatabaseMetaData dbmd) throws SQLException {
        System.out.println("\n📋 TABLAS EN LA BASE DE DATOS");
        System.out.println("-".repeat(80));
        
        ResultSet rs = dbmd.getTables(null, null, "%", new String[]{"TABLE"});
        
        int count = 0;
        while (rs.next()) {
            count++;
            String tableName = rs.getString("TABLE_NAME");
            String tableType = rs.getString("TABLE_TYPE");
            String remarks = rs.getString("REMARKS");
            
            System.out.println("\n🔹 Tabla: " + tableName);
            System.out.println("   Tipo: " + tableType);
            if (remarks != null && !remarks.isEmpty()) {
                System.out.println("   Comentario: " + remarks);
            }
            
            // Contar filas
            int numFilas = contarFilas(tableName);
            System.out.println("   Filas: " + numFilas);
        }
        rs.close();
        
        System.out.println("\nTotal de tablas: " + count);
    }
    
    private static void analizarTodasLasTablas(DatabaseMetaData dbmd) throws SQLException {
        System.out.println("\n🔍 ANÁLISIS DETALLADO DE TABLAS");
        System.out.println("=".repeat(80));
        
        ResultSet rs = dbmd.getTables(null, null, "%", new String[]{"TABLE"});
        
        while (rs.next()) {
            String tableName = rs.getString("TABLE_NAME");
            analizarTabla(dbmd, tableName);
        }
        rs.close();
    }
    
    private static void analizarTabla(DatabaseMetaData dbmd, String tableName) 
            throws SQLException {
        
        System.out.println("\n📊 Tabla: " + tableName.toUpperCase());
        System.out.println("-".repeat(80));
        
        // Columnas
        System.out.println("\nColumnas:");
        ResultSet rsColumns = dbmd.getColumns(null, null, tableName, "%");
        
        System.out.println(String.format("%-25s %-15s %-10s %-10s %-10s",
            "Nombre", "Tipo", "Tamaño", "Nullable", "Default"));
        System.out.println("-".repeat(80));
        
        while (rsColumns.next()) {
            String colName = rsColumns.getString("COLUMN_NAME");
            String colType = rsColumns.getString("TYPE_NAME");
            int colSize = rsColumns.getInt("COLUMN_SIZE");
            String nullable = rsColumns.getString("IS_NULLABLE");
            String defaultVal = rsColumns.getString("COLUMN_DEF");
            
            System.out.println(String.format("%-25s %-15s %-10d %-10s %-10s",
                colName, colType, colSize, nullable, 
                defaultVal != null ? defaultVal : "-"));
        }
        rsColumns.close();
        
        // Claves primarias
        System.out.println("\n🔑 Claves Primarias:");
        ResultSet rsPK = dbmd.getPrimaryKeys(null, null, tableName);
        
        while (rsPK.next()) {
            System.out.println("   - " + rsPK.getString("COLUMN_NAME"));
        }
        rsPK.close();
        
        // Claves foráneas
        System.out.println("\n🔗 Claves Foráneas:");
        ResultSet rsFK = dbmd.getImportedKeys(null, null, tableName);
        
        while (rsFK.next()) {
            String fkColumn = rsFK.getString("FKCOLUMN_NAME");
            String pkTable = rsFK.getString("PKTABLE_NAME");
            String pkColumn = rsFK.getString("PKCOLUMN_NAME");
            
            System.out.println("   - " + fkColumn + " → " + 
                             pkTable + "." + pkColumn);
        }
        rsFK.close();
        
        // Índices
        System.out.println("\n📇 Índices:");
        ResultSet rsIdx = dbmd.getIndexInfo(null, null, tableName, false, false);
        
        while (rsIdx.next()) {
            String indexName = rsIdx.getString("INDEX_NAME");
            String columnName = rsIdx.getString("COLUMN_NAME");
            boolean nonUnique = rsIdx.getBoolean("NON_UNIQUE");
            
            if (indexName != null) {
                System.out.println("   - " + indexName + 
                                 " (" + columnName + ")" +
                                 (nonUnique ? "" : " [UNIQUE]"));
            }
        }
        rsIdx.close();
    }
    
    private static void listarProcedimientos(DatabaseMetaData dbmd) throws SQLException {
        System.out.println("\n⚙️ PROCEDIMIENTOS Y FUNCIONES");
        System.out.println("-".repeat(80));
        
        ResultSet rs = dbmd.getProcedures(null, null, "%");
        
        int count = 0;
        while (rs.next()) {
            count++;
            String procName = rs.getString("PROCEDURE_NAME");
            String remarks = rs.getString("REMARKS");
            int procType = rs.getShort("PROCEDURE_TYPE");
            
            String tipo = "";
            switch (procType) {
                case DatabaseMetaData.procedureNoResult:
                    tipo = "PROCEDIMIENTO";
                    break;
                case DatabaseMetaData.procedureReturnsResult:
                    tipo = "FUNCIÓN";
                    break;
                default:
                    tipo = "DESCONOCIDO";
            }
            
            System.out.println("\n🔹 " + procName + " (" + tipo + ")");
            if (remarks != null && !remarks.isEmpty()) {
                System.out.println("   " + remarks);
            }
            
            // Parámetros
            ResultSet rsParams = dbmd.getProcedureColumns(null, null, procName, "%");
            System.out.println("   Parámetros:");
            
            while (rsParams.next()) {
                String paramName = rsParams.getString("COLUMN_NAME");
                int paramType = rsParams.getInt("COLUMN_TYPE");
                String dataType = rsParams.getString("TYPE_NAME");
                
                String paramTypeStr = "";
                switch (paramType) {
                    case DatabaseMetaData.procedureColumnIn:
                        paramTypeStr = "IN";
                        break;
                    case DatabaseMetaData.procedureColumnOut:
                        paramTypeStr = "OUT";
                        break;
                    case DatabaseMetaData.procedureColumnInOut:
                        paramTypeStr = "INOUT";
                        break;
                    case DatabaseMetaData.procedureColumnReturn:
                        paramTypeStr = "RETURN";
                        break;
                }
                
                if (paramName != null) {
                    System.out.println("      " + paramTypeStr + " " + 
                                     paramName + " (" + dataType + ")");
                }
            }
            rsParams.close();
        }
        rs.close();
        
        System.out.println("\nTotal: " + count);
    }
    
    private static int contarFilas(String tableName) {
        try (Connection conn = DatabaseConfig.getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT COUNT(*) FROM " + tableName)) {
            
            if (rs.next()) {
                return rs.getInt(1);
            }
        } catch (SQLException e) {
            return -1;
        }
        return 0;
    }
    
    public static void main(String[] args) {
        generarInformeCompleto();
    }
}


## 🔄 Práctica 3: Tipos de ResultSet

### Tipos Disponibles

|Tipo|Descripción|Navegación|Actualizable|
|---|---|---|---|
|`TYPE_FORWARD_ONLY`|Solo hacia adelante|`next()`|No (por defecto)|
|`TYPE_SCROLL_INSENSITIVE`|Scroll, no ve cambios|`next()`, `previous()`, `absolute()`|Opcional|
|`TYPE_SCROLL_SENSITIVE`|Scroll, ve cambios|`next()`, `previous()`, `absolute()`|Opcional|

### Concurrencia

|Modo|Descripción|
|---|---|
|`CONCUR_READ_ONLY`|Solo lectura (por defecto)|
|`CONCUR_UPDATABLE`|Permite actualizar|

### Ejemplo: ResultSet Scrollable

public void ejemploScrollable() {
    String sql = "SELECT * FROM empleados";
    
    try (Connection conn = DatabaseConfig.getConnection();
         PreparedStatement pstmt = conn.prepareStatement(sql,
             ResultSet.TYPE_SCROLL_INSENSITIVE,
             ResultSet.CONCUR_READ_ONLY);
         ResultSet rs = pstmt.executeQuery()) {
        
        // Ir al final
        rs.afterLast();
        System.out.println("\n📍 Navegando desde el final:");
        
        // Navegar hacia atrás
        while (rs.previous()) {
            System.out.println(rs.getString("nombre"));
        }
        
        // Ir a posición específica
        rs.absolute(3);  // Ir a la fila 3
        System.out.println("\n📍 Fila 3: " + rs.getString("nombre"));
        
        // Ir al inicio
        rs.first();
        System.out.println("📍 Primera fila: " + rs.getString("nombre"));
        
        // Ir al final
        rs.last();
        System.out.println("📍 Última fila: " + rs.getString("nombre"));
        
        // Número de filas
        int numFilas = rs.getRow();
        System.out.println("\nTotal de filas: " + numFilas);
        
    } catch (SQLException e) {
        System.err.println("Error: " + e.getMessage());
    }
}

### Ejemplo: ResultSet Actualizable

public void ejemploActualizable() {
    String sql = "SELECT * FROM empleados WHERE departamento = 'Desarrollo'";
    
    try (Connection conn = DatabaseConfig.getConnection();
         PreparedStatement pstmt = conn.prepareStatement(sql,
             ResultSet.TYPE_SCROLL_INSENSITIVE,
             ResultSet.CONCUR_UPDATABLE);
         ResultSet rs = pstmt.executeQuery()) {
        
        System.out.println("\n💰 Incrementando salarios en 5%...");
        
        while (rs.next()) {
            BigDecimal salarioActual = rs.getBigDecimal("salario");
            BigDecimal nuevoSalario = salarioActual.multiply(new BigDecimal("1.05"));
            
            // Actualizar directamente en el ResultSet
            rs.updateBigDecimal("salario", nuevoSalario);
            rs.updateRow();  // Aplicar cambio a la BD
            
            System.out.println(rs.getString("nombre") + ": " + 
                             salarioActual + " → " + nuevoSalario);
        }
        
        System.out.println("\n✅ Salarios actualizados");
        
    } catch (SQLException e) {
        System.err.println("Error: " + e.getMessage());
    }
}


## 📊 Práctica 4: Generador de Informes Dinámico

### Clase GeneradorInformes

package com.techdam.service;

import com.techdam.config.DatabaseConfig;
import java.sql.*;
import java.io.*;

/**
 * Generador de informes dinámicos basado en consultas SQL
 */
public class GeneradorInformes {
    
    /**
     * Genera un informe en formato CSV
     */
    public static void generarCSV(String sql, String archivoSalida) {
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        PrintWriter writer = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            pstmt = conn.prepareStatement(sql);
            rs = pstmt.executeQuery();
            
            writer = new PrintWriter(new FileWriter(archivoSalida));
            
            // Obtener metadatos
            ResultSetMetaData rsmd = rs.getMetaData();
            int numColumnas = rsmd.getColumnCount();
            
            // Escribir encabezados
            for (int i = 1; i <= numColumnas; i++) {
                writer.print(rsmd.getColumnLabel(i));
                if (i < numColumnas) writer.print(",");
            }
            writer.println();
            
            // Escribir datos
            while (rs.next()) {
                for (int i = 1; i <= numColumnas; i++) {
                    Object valor = rs.getObject(i);
                    writer.print(valor != null ? valor.toString() : "");
                    if (i < numColumnas) writer.print(",");
                }
                writer.println();
            }
            
            System.out.println("✅ Informe CSV generado: " + archivoSalida);
            
        } catch (SQLException | IOException e) {
            System.err.println("Error: " + e.getMessage());
        } finally {
            if (writer != null) writer.close();
            try {
                if (rs != null) rs.close();
                if (pstmt != null) pstmt.close();
            } catch (SQLException e) {}
            DatabaseConfig.closeConnection(conn);
        }
    }
    
    /**
     * Genera un informe en formato HTML
     */
    public static void generarHTML(String sql, String titulo, String archivoSalida) {
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        PrintWriter writer = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            pstmt = conn.prepareStatement(sql);
            rs = pstmt.executeQuery();
            
            writer = new PrintWriter(new FileWriter(archivoSalida));
            
            // HTML Header
            writer.println("<!DOCTYPE html>");
            writer.println("<html>");
            writer.println("<head>");
            writer.println("<meta charset='UTF-8'>");
            writer.println("<title>" + titulo + "</title>");
            writer.println("<style>");
            writer.println("table { border-collapse: collapse; width: 100%; }");
            writer.println("th, td { border: 1px solid black; padding: 8px; text-align: left; }");
            writer.println("th { background-color: #4CAF50; color: white; }");
            writer.println("tr:nth-child(even) { background-color: #f2f2f2; }");
            writer.println("</style>");
            writer.println("</head>");
            writer.println("<body>");
            writer.println("<h1>" + titulo + "</h1>");
            writer.println("<table>");
            
            // Metadatos
            ResultSetMetaData rsmd = rs.getMetaData();
            int numColumnas = rsmd.getColumnCount();
            
            // Encabezados
            writer.println("<thead><tr>");
            for (int i = 1; i <= numColumnas; i++) {
                writer.println("<th>" + rsmd.getColumnLabel(i) + "</th>");
            }
            writer.println("</tr></thead>");
            
            // Datos
            writer.println("<tbody>");
            while (rs.next()) {
                writer.println("<tr>");
                for (int i = 1; i <= numColumnas; i++) {
                    Object valor = rs.getObject(i);
                    writer.println("<td>" + 
                        (valor != null ? valor.toString() : "") + "</td>");
                }
                writer.println("</tr>");
            }
            writer.println("</tbody>");
            
            writer.println("</table>");
            writer.println("</body>");
            writer.println("</html>");
            
            System.out.println("✅ Informe HTML generado: " + archivoSalida);
            
        } catch (SQLException | IOException e) {
            System.err.println("Error: " + e.getMessage());
        } finally {
            if (writer != null) writer.close();
            try {
                if (rs != null) rs.close();
                if (pstmt != null) pstmt.close();
            } catch (SQLException e) {}
            DatabaseConfig.closeConnection(conn);
        }
    }
    
    public static void main(String[] args) {
        // Informe 1: Todos los empleados
        generarCSV(
            "SELECT * FROM empleados",
            "informe_empleados.csv"
        );
        
        // Informe 2: Proyectos activos
        generarHTML(
            "SELECT p.nombre, p.presupuesto, COUNT(a.id) AS num_empleados " +
            "FROM proyectos p " +
            "LEFT JOIN asignaciones a ON p.id = a.proyecto_id " +
            "WHERE p.estado = 'EN_CURSO' " +
            "GROUP BY p.id",
            "Proyectos en Curso - TechDAM",
            "proyectos_activos.html"
        );
    }
}


## ✏️ Ejercicio: Dashboard de TechDAM

### Enunciado

Crea una clase `DashboardTechDAM` que genere un informe HTML con:

1. **Estadísticas generales**:
    
    - Total de empleados
        
    - Total de proyectos
        
    - Presupuesto total
        
    - Salario promedio
        
2. **Tabla de empleados por departamento**
    
3. **Tabla de proyectos ordenados por presupuesto**
    
4. **Gráfico (opcional)**: Usa JavaScript/Chart.js para visualizar datos

**Pistas**:

- Usa ResultSetMetaData para hacer el código genérico
    
- Puedes reutilizar el código de GeneradorInformes
    
- Incluye CSS para que se vea profesional


## 🎯 Evaluación del Bloque

### Checklist

- [ ] Uso ResultSetMetaData para consultas dinámicas
    
- [ ] Obtengo información detallada con DatabaseMetaData
    
- [ ] Entiendo los tipos de ResultSet
    
- [ ] Navego en ResultSet scrollable
    
- [ ] Actualizo filas con ResultSet updatable
    
- [ ] Genero informes en diferentes formatos


## 📚 Resumen

En este bloque hemos dominado:

- ✅ ResultSetMetaData para consultas flexibles
    
- ✅ DatabaseMetaData para introspección completa
    
- ✅ Tipos de ResultSet y su navegación
    
- ✅ ResultSet actualizable
    
- ✅ Generación de informes dinámicos

## 💡 Notas del Profesor

> **Buena práctica**: Usa ResultSetMetaData cuando necesites código genérico que funcione con cualquier consulta.

> **Advertencia**: ResultSet scrollable y updatable son más lentos que forward-only. Úsalos solo cuando sea necesario.