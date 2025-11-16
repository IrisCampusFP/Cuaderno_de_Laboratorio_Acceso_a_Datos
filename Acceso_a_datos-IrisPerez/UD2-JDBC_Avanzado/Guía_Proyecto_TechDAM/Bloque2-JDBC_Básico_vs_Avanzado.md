## Objetivos del Bloque

- Identificar las diferencias entre JDBC básico y avanzado
    
- Comprender por qué usar técnicas avanzadas
    
- Reconocer vulnerabilidades de seguridad
    
- Aplicar mejores prácticas desde el inicio

## Teoría: Evolución de JDBC

### JDBC Básico (Lo que ya sabes)

// Conexión simple
Connection conn = DriverManager.getConnection(url, user, pass);

// Statement básico
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM empleados");

// Procesamiento
while (rs.next()) {
    System.out.println(rs.getString("nombre"));
}

### JDBC Avanzado (Lo que aprenderás)

// Pool de conexiones
HikariDataSource dataSource = new HikariDataSource(config);
Connection conn = dataSource.getConnection();

// PreparedStatement con parámetros
String sql = "SELECT * FROM empleados WHERE departamento = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, "Desarrollo");

// Transacciones
conn.setAutoCommit(false);
// ... operaciones ...
conn.commit();

// Procedimientos almacenados
CallableStatement cstmt = conn.prepareCall("{call incrementar_salario(?, ?, ?)}");

## Comparativa Detallada

|Aspecto|JDBC Básico|JDBC Avanzado|
|---|---|---|
|**Sentencias SQL**|Statement|PreparedStatement|
|**Seguridad**|⚠️ Vulnerable a inyección SQL|✅ Protegido|
|**Rendimiento**|❌ Compila cada vez|✅ Pre-compilado|
|**Gestión de conexiones**|Manual (open/close)|Pool automático|
|**Transacciones**|Autocommit activo|Control manual|
|**Procedimientos**|No usa|CallableStatement|
|**Metadatos**|Raramente|DatabaseMetaData|
|**Complejidad**|Simple|Más sofisticado|
|**Casos de uso**|Scripts rápidos|Aplicaciones empresariales|

## Problema 1: Inyección SQL

### Ejemplo de Ataque

#### ❌ Código Vulnerable

// NUNCA HAGAS ESTO
public List<Empleado> buscarPorNombre(String nombre) {
    Connection conn = null;
    Statement stmt = null;
    ResultSet rs = null;
    List<Empleado> empleados = new ArrayList<>();
    
    try {
        conn = DatabaseConfig.getConnection();
        stmt = conn.createStatement();
        
        // ⚠️ VULNERABLE: Concatenación directa
        String sql = "SELECT * FROM empleados WHERE nombre = '" + nombre + "'";
        
        rs = stmt.executeQuery(sql);
        
        while (rs.next()) {
            // mapear empleado...
        }
    } catch (SQLException e) {
        e.printStackTrace();
    }
    
    return empleados;
}

// Llamada normal:
buscarPorNombre("Juan");
// SQL generado: SELECT * FROM empleados WHERE nombre = 'Juan'

// 🔥 Llamada maliciosa:
buscarPorNombre("Juan' OR '1'='1");
// SQL generado: SELECT * FROM empleados WHERE nombre = 'Juan' OR '1'='1'
// ¡DEVUELVE TODOS LOS EMPLEADOS!

// 💀 Ataque destructivo:
buscarPorNombre("Juan'; DROP TABLE empleados; --");
// SQL generado: SELECT * FROM empleados WHERE nombre = 'Juan'; DROP TABLE empleados; --'
// ¡ELIMINA LA TABLA!

#### ✅ Código Seguro

// SIEMPRE HAZ ESTO
public List<Empleado> buscarPorNombreSeguro(String nombre) {
    Connection conn = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    List<Empleado> empleados = new ArrayList<>();
    
    try {
        conn = DatabaseConfig.getConnection();
        
        // ✅ SEGURO: PreparedStatement con parámetros
        String sql = "SELECT * FROM empleados WHERE nombre = ?";
        pstmt = conn.prepareStatement(sql);
        pstmt.setString(1, nombre);  // El valor se escapa automáticamente
        
        rs = pstmt.executeQuery();
        
        while (rs.next()) {
            // mapear empleado...
        }
    } catch (SQLException e) {
        e.printStackTrace();
    }
    
    return empleados;
}

// Llamada maliciosa:
buscarPorNombreSeguro("Juan' OR '1'='1");
// PreparedStatement trata todo el string como un valor literal
// Busca literalmente un empleado llamado "Juan' OR '1'='1"
// No encuentra nada → ✅ SEGURO

## Problema 2: Rendimiento

### Statement: Compila Cada Vez

// ❌ Ineficiente con Statement
for (int i = 0; i < 1000; i++) {
    String sql = "INSERT INTO empleados (nombre) VALUES ('Empleado" + i + "')";
    stmt.executeUpdate(sql);
    // Cada iteración:
    // 1. Construye SQL
    // 2. Envía al servidor
    // 3. Servidor PARSEA la consulta
    // 4. Servidor OPTIMIZA
    // 5. Servidor EJECUTA
    // = 5000 operaciones
}

### PreparedStatement: Pre-compilado

// ✅ Eficiente con PreparedStatement
String sql = "INSERT INTO empleados (nombre) VALUES (?)";
PreparedStatement pstmt = conn.prepareStatement(sql);

for (int i = 0; i < 1000; i++) {
    pstmt.setString(1, "Empleado" + i);
    pstmt.executeUpdate();
    // Solo en la primera iteración:
    // 1. PARSEA
    // 2. OPTIMIZA
    // Después, solo:
    // 3. Cambia parámetros
    // 4. EJECUTA
    // = 2 + (2 × 1000) = 2002 operaciones
}
// ⚡ ~60% más rápido


## 💻 Demostración Práctica

### Comparativa de Rendimiento

package com.techdam;

import com.techdam.config.DatabaseConfig;
import java.sql.*;

public class ComparativaRendimiento {
    
    public static void main(String[] args) {
        int numInserciones = 1000;
        
        System.out.println("========================================");
        System.out.println("⚡ COMPARATIVA DE RENDIMIENTO");
        System.out.println("========================================\n");
        
        // Crear tabla temporal para pruebas
        crearTablaTemp();
        
        // Test 1: Statement
        System.out.println("--- Test 1: Statement ---");
        long inicioStatement = System.currentTimeMillis();
        insertarConStatement(numInserciones);
        long tiempoStatement = System.currentTimeMillis() - inicioStatement;
        System.out.println("Tiempo: " + tiempoStatement + " ms\n");
        
        limpiarTabla();
        
        // Test 2: PreparedStatement
        System.out.println("--- Test 2: PreparedStatement ---");
        long inicioPrepared = System.currentTimeMillis();
        insertarConPreparedStatement(numInserciones);
        long tiempoPrepared = System.currentTimeMillis() - inicioPrepared;
        System.out.println("Tiempo: " + tiempoPrepared + " ms\n");
        
        // Resultados
        System.out.println("========================================");
        System.out.println("📊 RESULTADOS");
        System.out.println("========================================");
        System.out.println("Statement:         " + tiempoStatement + " ms");
        System.out.println("PreparedStatement: " + tiempoPrepared + " ms");
        System.out.println("\nMejora: " + 
                         (100 - (tiempoPrepared * 100 / tiempoStatement)) + "%");
        
        eliminarTablaTemp();
    }
    
    private static void crearTablaTemp() {
        try (Connection conn = DatabaseConfig.getConnection();
             Statement stmt = conn.createStatement()) {
            
            stmt.execute("DROP TABLE IF EXISTS test_rendimiento");
            stmt.execute("CREATE TABLE test_rendimiento (" +
                       "id INT AUTO_INCREMENT PRIMARY KEY, " +
                       "dato VARCHAR(50))");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    private static void insertarConStatement(int num) {
        try (Connection conn = DatabaseConfig.getConnection();
             Statement stmt = conn.createStatement()) {
            
            for (int i = 0; i < num; i++) {
                String sql = "INSERT INTO test_rendimiento (dato) " +
                           "VALUES ('Dato" + i + "')";
                stmt.executeUpdate(sql);
            }
            
            System.out.println("✅ Insertadas " + num + " filas con Statement");
            
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    private static void insertarConPreparedStatement(int num) {
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(
                 "INSERT INTO test_rendimiento (dato) VALUES (?)")) {
            
            for (int i = 0; i < num; i++) {
                pstmt.setString(1, "Dato" + i);
                pstmt.executeUpdate();
            }
            
            System.out.println("✅ Insertadas " + num + " filas con PreparedStatement");
            
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    private static void limpiarTabla() {
        try (Connection conn = DatabaseConfig.getConnection();
             Statement stmt = conn.createStatement()) {
            stmt.execute("TRUNCATE TABLE test_rendimiento");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    private static void eliminarTablaTemp() {
        try (Connection conn = DatabaseConfig.getConnection();
             Statement stmt = conn.createStatement()) {
            stmt.execute("DROP TABLE IF EXISTS test_rendimiento");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}


## 🔄 Refactorización: De Básico a Avanzado

### Antes (JDBC Básico)

// ❌ Código básico con problemas
public void actualizarSalario(int id, double nuevoSalario) {
    Connection conn = null;
    Statement stmt = null;
    
    try {
        Class.forName("com.mysql.cj.jdbc.Driver");
        conn = DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/techdam", "root", "");
        
        stmt = conn.createStatement();
        
        // Vulnerable a inyección SQL
        String sql = "UPDATE empleados SET salario = " + nuevoSalario + 
                    " WHERE id = " + id;
        
        stmt.executeUpdate(sql);
        
        System.out.println("Actualizado");
        
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        try {
            if (stmt != null) stmt.close();
            if (conn != null) conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

### Después (JDBC Avanzado)

// ✅ Código avanzado optimizado
public boolean actualizarSalario(int id, BigDecimal nuevoSalario) {
    // Try-with-resources (cierre automático)
    String sql = "UPDATE empleados SET salario = ? WHERE id = ?";
    
    try (Connection conn = DatabaseConfig.getConnection();
         PreparedStatement pstmt = conn.prepareStatement(sql)) {
        
        // Parámetros seguros
        pstmt.setBigDecimal(1, nuevoSalario);
        pstmt.setInt(2, id);
        
        int filasActualizadas = pstmt.executeUpdate();
        
        if (filasActualizadas > 0) {
            System.out.println("✅ Salario actualizado");
            return true;
        } else {
            System.out.println("⚠️ Empleado no encontrado");
            return false;
        }
        
    } catch (SQLException e) {
        System.err.println("❌ Error al actualizar: " + e.getMessage());
        return false;
    }
    // No necesita finally, try-with-resources cierra automáticamente
}

## Ejercicio: Detecta los Problemas

### Código con Múltiples Problemas

public void buscarEmpleados(String departamento) {
    Connection conn = null;
    Statement stmt = null;
    
    try {
        conn = DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/techdam", "root", "root");
        
        stmt = conn.createStatement();
        
        String sql = "SELECT * FROM empleados WHERE departamento = '" + 
                    departamento + "' AND activo = 1";
        
        ResultSet rs = stmt.executeQuery(sql);
        
        while (rs.next()) {
            System.out.println(rs.getString(2));
        }
        
    } catch (SQLException e) {
        // Ignorar error
    }
}

### Encuentra 10 Problemas:

1. ❌ **Inyección SQL**: Concatenación de strings
    
2. ❌ **Statement en lugar de PreparedStatement**
    
3. ❌ **Credenciales hardcodeadas**
    
4. ❌ **No cierra ResultSet**
    
5. ❌ **No cierra Statement**
    
6. ❌ **No cierra Connection**
    
7. ❌ **Ignora excepciones** (catch vacío)
    
8. ❌ **Acceso por índice** (rs.getString(2))
    
9. ❌ **No usa try-with-resources**
    
10. ❌ **No usa clase de configuración**

### Versión Corregida:

public List<Empleado> buscarEmpleadosCorregido(String departamento) {
    List<Empleado> empleados = new ArrayList<>();
    String sql = "SELECT * FROM empleados WHERE departamento = ? AND activo = TRUE";
    
    try (Connection conn = DatabaseConfig.getConnection();
         PreparedStatement pstmt = conn.prepareStatement(sql)) {
        
        pstmt.setString(1, departamento);
        
        try (ResultSet rs = pstmt.executeQuery()) {
            while (rs.next()) {
                Empleado emp = new Empleado();
                emp.setId(rs.getInt("id"));
                emp.setNombre(rs.getString("nombre"));
                emp.setApellido(rs.getString("apellido"));
                // ... resto de campos
                empleados.add(emp);
            }
        }
        
    } catch (SQLException e) {
        System.err.println("Error al buscar empleados: " + e.getMessage());
        e.printStackTrace();
    }
    
    return empleados;
}


## 📋 Checklist de Mejores Prácticas

### ✅ Siempre hacer:

1. ✅ Usar **PreparedStatement** (nunca Statement)
    
2. ✅ Usar **try-with-resources** para cerrar recursos
    
3. ✅ **Validar parámetros** antes de ejecutar
    
4. ✅ **Manejar excepciones** apropiadamente
    
5. ✅ Usar **nombres de columnas** en ResultSet
    
6. ✅ **Separar configuración** del código
    
7. ✅ Usar **tipos apropiados** (BigDecimal para dinero)
    
8. ✅ **Documentar** con JavaDoc

### ❌ Nunca hacer:

1. ❌ Concatenar strings para construir SQL
    
2. ❌ Usar Statement con datos del usuario
    
3. ❌ Hardcodear credenciales
    
4. ❌ Dejar recursos abiertos
    
5. ❌ Ignorar excepciones (catch vacío)
    
6. ❌ Acceder por índice numérico sin constantes
    
7. ❌ Usar tipos primitivos para dinero (float, double)
    
8. ❌ Repetir código de conexión

## Evaluación del Bloque

### Autoevaluación

- [ ] Identifico las diferencias entre Statement y PreparedStatement
    
- [ ] Entiendo qué es la inyección SQL y cómo prevenirla
    
- [ ] Sé por qué PreparedStatement es más rápido
    
- [ ] Puedo refactorizar código básico a avanzado
    
- [ ] Aplico try-with-resources correctamente
    
- [ ] Reconozco código vulnerable al verlo

## Resumen

### Conceptos Clave

1. **PreparedStatement > Statement** siempre
    
2. **Inyección SQL** es un riesgo real y grave
    
3. **Pre-compilación** mejora el rendimiento
    
4. **Try-with-resources** simplifica el código
    
5. **Configuración centralizada** facilita mantenimiento

### Regla de Oro

> «Si el SQL incluye datos del usuario, USA PreparedStatement. Sin excepciones.»

## Siguiente Paso

Ahora que conoces las diferencias, en el **Bloque 3** aplicarás estas técnicas avanzadas para crear el CRUD completo de TechDAM.

**→ [Ir al Bloque 3: Sentencias DDL y DML](https://campusfp.dvsweb.es/accesoDatos/ud2/content/Proyecto/03_ddl_dml_jdbc.html)**

## Notas del Profesor

> **Anécdota real**: En 2019, la base de datos de una empresa de telecomunicaciones fue comprometida por inyección SQL. Resultado: 106 millones de registros de clientes robados, multa de $80 millones.

> **Ejercicio extra**: Busca en Google «SQL injection examples» y observa casos reales de ataques.