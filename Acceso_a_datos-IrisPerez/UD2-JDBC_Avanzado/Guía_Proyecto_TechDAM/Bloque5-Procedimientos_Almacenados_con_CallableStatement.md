## Objetivos del Bloque

- Comprender qué son los procedimientos almacenados
    
- Dominar CallableStatement para invocarlos desde Java
    
- Trabajar con parámetros IN, OUT e INOUT
    
- Llamar funciones almacenadas
    
- Implementar lógica de negocio en la BD de TechDAM

## Teoría: Procedimientos Almacenados

### ¿Qué son?

**Procedimientos almacenados** son **programas** guardados EN la base de datos que:

- ✅ Se ejecutan en el servidor BD
    
- ✅ Encapsulan lógica de negocio
    
- ✅ Pueden recibir parámetros
    
- ✅ Pueden devolver valores

### Ventajas

|Ventaja|Descripción|
|---|---|
|**Rendimiento**|Se ejecutan cerca de los datos|
|**Seguridad**|Controlan el acceso a datos|
|**Mantenimiento**|Lógica centralizada|
|**Red**|Menos tráfico (solo params/results)|
|**Transacciones**|Control completo|
|**Reutilización**|Múltiples apps usan el mismo código|

### Desventajas

|Desventaja|Descripción|
|---|---|
|**Portabilidad**|Sintaxis específica de cada DBMS|
|**Debugging**|Más difícil que código Java|
|**Versionado**|No usa Git fácilmente|
|**Testing**|Requiere herramientas especiales|

## 🔧 CallableStatement

### Jerarquía JDBC

Statement
    ↑
    ├── PreparedStatement (consultas parametrizadas)
    │       ↑
    │       └── CallableStatement (procedimientos almacenados)

### Sintaxis Básica

// Sintaxis estándar JDBC (portable)
CallableStatement cstmt = conn.prepareCall("{call nombre_procedimiento(?, ?, ?)}");

// Con funciones (que devuelven valor)
CallableStatement cstmt = conn.prepareCall("{? = call nombre_funcion(?)}");

## 💻 Práctica 1: Procedimiento Simple (Solo IN)

### Procedimiento: incrementar_salario

Ya creamos este procedimiento en el script SQL. Tiene:

- **IN**: `p_empleado_id` (INT)
    
- **IN**: `p_porcentaje` (DECIMAL)
    
- **OUT**: `p_salario_anterior` (DECIMAL)
    
- **OUT**: `p_salario_nuevo` (DECIMAL)

### Código Java

package com.techdam.service;

import com.techdam.config.DatabaseConfig;
import java.math.BigDecimal;
import java.sql.*;

/**
 * Servicio para invocar procedimientos almacenados
 */
public class ProcedimientosService {
    
    /**
     * Incrementa el salario de un empleado usando procedimiento almacenado
     * 
     * @param empleadoId ID del empleado
     * @param porcentaje Porcentaje de incremento
     */
    public static void incrementarSalario(int empleadoId, double porcentaje) {
        Connection conn = null;
        CallableStatement cstmt = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            
            // Preparar la llamada al procedimiento
            // Sintaxis: {call nombre_procedimiento(?, ?, ?, ?)}
            String sql = "{call incrementar_salario(?, ?, ?, ?)}";
            cstmt = conn.prepareCall(sql);
            
            // Establecer parámetros IN
            cstmt.setInt(1, empleadoId);           // p_empleado_id
            cstmt.setBigDecimal(2, BigDecimal.valueOf(porcentaje)); // p_porcentaje
            
            // Registrar parámetros OUT
            cstmt.registerOutParameter(3, Types.DECIMAL); // p_salario_anterior
            cstmt.registerOutParameter(4, Types.DECIMAL); // p_salario_nuevo
            
            // Ejecutar el procedimiento
            cstmt.execute();
            
            // Obtener valores OUT
            BigDecimal salarioAnterior = cstmt.getBigDecimal(3);
            BigDecimal salarioNuevo = cstmt.getBigDecimal(4);
            
            // Mostrar resultados
            System.out.println("\n✅ Salario incrementado");
            System.out.println("   Empleado ID: " + empleadoId);
            System.out.println("   Incremento: " + porcentaje + "%");
            System.out.println("   Salario anterior: $" + salarioAnterior);
            System.out.println("   Salario nuevo: $" + salarioNuevo);
            System.out.println("   Diferencia: $" + 
                             salarioNuevo.subtract(salarioAnterior));
            
        } catch (SQLException e) {
            System.err.println("❌ Error al incrementar salario: " + e.getMessage());
            e.printStackTrace();
        } finally {
            try {
                if (cstmt != null) cstmt.close();
            } catch (SQLException e) {}
            DatabaseConfig.closeConnection(conn);
        }
    }
    
    public static void main(String[] args) {
        // Incrementar salario del empleado 1 en un 10%
        incrementarSalario(1, 10.0);
    }
}

## 💡 Tipos de Parámetros

### IN: Entrada al procedimiento

// Java → BD
cstmt.setInt(1, valor);
cstmt.setString(2, "texto");
cstmt.setBigDecimal(3, decimal);

### OUT: Salida del procedimiento

// BD → Java
cstmt.registerOutParameter(4, Types.INTEGER);
cstmt.execute();
int resultado = cstmt.getInt(4);

### INOUT: Entrada y Salida

// Java → BD → Java
cstmt.setInt(5, valorInicial);
cstmt.registerOutParameter(5, Types.INTEGER);
cstmt.execute();
int valorFinal = cstmt.getInt(5);


## 💻 Práctica 2: Asignar Empleado a Proyecto

### Procedimiento con Validaciones

/**
 * Asigna un empleado a un proyecto con validaciones
 */
public static void asignarEmpleadoProyecto(int empleadoId, int proyectoId, 
                                          int horas, String rol) {
    Connection conn = null;
    CallableStatement cstmt = null;
    
    try {
        conn = DatabaseConfig.getConnection();
        
        String sql = "{call asignar_empleado_proyecto(?, ?, ?, ?, ?)}";
        cstmt = conn.prepareCall(sql);
        
        // Parámetros IN
        cstmt.setInt(1, empleadoId);
        cstmt.setInt(2, proyectoId);
        cstmt.setInt(3, horas);
        cstmt.setString(4, rol);
        
        // Parámetro OUT
        cstmt.registerOutParameter(5, Types.VARCHAR);
        
        // Ejecutar
        cstmt.execute();
        
        // Obtener resultado
        String resultado = cstmt.getString(5);
        
        if (resultado.startsWith("ÉXITO")) {
            System.out.println("✅ " + resultado);
        } else {
            System.out.println("⚠️ " + resultado);
        }
        
    } catch (SQLException e) {
        // El procedimiento lanza excepción si hay errores
        System.err.println("❌ Error: " + e.getMessage());
    } finally {
        try {
            if (cstmt != null) cstmt.close();
        } catch (SQLException e) {}
        DatabaseConfig.closeConnection(conn);
    }
}

## 💻 Práctica 3: Procedimiento con Múltiples ResultSets

### Procedimiento: reporte_empleado

/**
 * Genera reporte completo de un empleado
 * Devuelve 2 ResultSets: datos empleado + proyectos asignados
 */
public static void generarReporteEmpleado(int empleadoId) {
    Connection conn = null;
    CallableStatement cstmt = null;
    
    try {
        conn = DatabaseConfig.getConnection();
        
        String sql = "{call reporte_empleado(?)}";
        cstmt = conn.prepareCall(sql);
        cstmt.setInt(1, empleadoId);
        
        // Ejecutar
        boolean hasResults = cstmt.execute();
        
        // Procesar múltiples ResultSets
        int resultSetNum = 1;
        
        while (hasResults) {
            ResultSet rs = cstmt.getResultSet();
            
            if (resultSetNum == 1) {
                // Primer ResultSet: Datos del empleado
                System.out.println("\n👤 DATOS DEL EMPLEADO");
                System.out.println("=".repeat(80));
                
                if (rs.next()) {
                    System.out.println("ID:              " + rs.getInt("id"));
                    System.out.println("Nombre:          " + 
                        rs.getString("nombre") + " " + rs.getString("apellido"));
                    System.out.println("Email:           " + rs.getString("email"));
                    System.out.println("Salario:         $" + rs.getBigDecimal("salario"));
                    System.out.println("Departamento:    " + rs.getString("departamento"));
                    System.out.println("Fecha contrato:  " + rs.getDate("fecha_contrato"));
                    System.out.println("Antigüedad:      " + 
                        rs.getInt("antiguedad_años") + " años");
                }
                
            } else if (resultSetNum == 2) {
                // Segundo ResultSet: Proyectos
                System.out.println("\n📂 PROYECTOS ASIGNADOS");
                System.out.println("=".repeat(80));
                
                int count = 0;
                while (rs.next()) {
                    count++;
                    System.out.println("\n🔹 Proyecto #" + count);
                    System.out.println("   Nombre:           " + 
                        rs.getString("proyecto_nombre"));
                    System.out.println("   Rol:              " + rs.getString("rol"));
                    System.out.println("   Horas asignadas:  " + 
                        rs.getInt("horas_asignadas"));
                    System.out.println("   Fecha asignación: " + 
                        rs.getDate("fecha_asignacion"));
                    System.out.println("   Estado proyecto:  " + 
                        rs.getString("proyecto_estado"));
                }
                
                if (count == 0) {
                    System.out.println("   Sin proyectos asignados");
                }
            }
            
            rs.close();
            resultSetNum++;
            
            // Mover al siguiente ResultSet
            hasResults = cstmt.getMoreResults();
        }
        
        System.out.println("\n" + "=".repeat(80));
        
    } catch (SQLException e) {
        System.err.println("❌ Error: " + e.getMessage());
    } finally {
        try {
            if (cstmt != null) cstmt.close();
        } catch (SQLException e) {}
        DatabaseConfig.closeConnection(conn);
    }
}

## 💻 Práctica 4: Funciones Almacenadas

### Diferencias: Función vs Procedimiento

|Característica|Procedimiento|Función|
|---|---|---|
|Devuelve valor|No (usa OUT)|Sí (RETURN)|
|Uso en SELECT|No|Sí|
|Múltiples resultados|Sí|No|
|Llamada|`{call proc()}`|`{? = call func()}`|

### Función: contar_empleados_departamento

/**
 * Cuenta empleados de un departamento usando función almacenada
 */
public static int contarEmpleadosDepartamento(String departamento) {
    Connection conn = null;
    CallableStatement cstmt = null;
    
    try {
        conn = DatabaseConfig.getConnection();
        
        // Sintaxis para funciones: {? = call funcion(?)}
        String sql = "{? = call contar_empleados_departamento(?)}";
        cstmt = conn.prepareCall(sql);
        
        // Registrar valor de retorno (primer parámetro)
        cstmt.registerOutParameter(1, Types.INTEGER);
        
        // Parámetro de entrada
        cstmt.setString(2, departamento);
        
        // Ejecutar
        cstmt.execute();
        
        // Obtener resultado
        int total = cstmt.getInt(1);
        
        System.out.println("📊 Empleados en " + departamento + ": " + total);
        
        return total;
        
    } catch (SQLException e) {
        System.err.println("❌ Error: " + e.getMessage());
        return -1;
    } finally {
        try {
            if (cstmt != null) cstmt.close();
        } catch (SQLException e) {}
        DatabaseConfig.closeConnection(conn);
    }
}

### Función: calcular_costo_proyecto

/**
 * Calcula el costo total de un proyecto en salarios
 */
public static BigDecimal calcularCostoProyecto(int proyectoId) {
    Connection conn = null;
    CallableStatement cstmt = null;
    
    try {
        conn = DatabaseConfig.getConnection();
        
        String sql = "{? = call calcular_costo_proyecto(?)}";
        cstmt = conn.prepareCall(sql);
        
        // Valor de retorno
        cstmt.registerOutParameter(1, Types.DECIMAL);
        
        // Parámetro
        cstmt.setInt(2, proyectoId);
        
        // Ejecutar
        cstmt.execute();
        
        // Resultado
        BigDecimal costo = cstmt.getBigDecimal(1);
        
        System.out.println("💰 Costo del proyecto " + proyectoId + ": $" + costo);
        
        return costo;
        
    } catch (SQLException e) {
        System.err.println("❌ Error: " + e.getMessage());
        return BigDecimal.ZERO;
    } finally {
        try {
            if (cstmt != null) cstmt.close();
        } catch (SQLException e) {}
        DatabaseConfig.closeConnection(conn);
    }
}


## 💻 Práctica 5: Transferencia con Procedimiento

### Procedimiento: transferir_presupuesto

/**
 * Transfiere presupuesto entre proyectos usando procedimiento
 * (alternativa a TransferService que lo hace en Java)
 */
public static void transferirPresupuesto(int origenId, int destinoId, 
                                        BigDecimal monto) {
    Connection conn = null;
    CallableStatement cstmt = null;
    
    try {
        conn = DatabaseConfig.getConnection();
        
        String sql = "{call transferir_presupuesto(?, ?, ?, ?)}";
        cstmt = conn.prepareCall(sql);
        
        // Parámetros IN
        cstmt.setInt(1, origenId);
        cstmt.setInt(2, destinoId);
        cstmt.setBigDecimal(3, monto);
        
        // Parámetro OUT
        cstmt.registerOutParameter(4, Types.VARCHAR);
        
        // Ejecutar
        System.out.println("\n💸 Transfiriendo $" + monto + "...");
        System.out.println("   Proyecto origen: " + origenId);
        System.out.println("   Proyecto destino: " + destinoId);
        
        cstmt.execute();
        
        // Resultado
        String resultado = cstmt.getString(4);
        
        if (resultado.startsWith("ÉXITO")) {
            System.out.println("\n✅ " + resultado);
        } else {
            System.out.println("\n❌ " + resultado);
        }
        
    } catch (SQLException e) {
        System.err.println("\n❌ Error: " + e.getMessage());
    } finally {
        try {
            if (cstmt != null) cstmt.close();
        } catch (SQLException e) {}
        DatabaseConfig.closeConnection(conn);
    }
}

## Ejercicio Guiado: Prueba de Todos los Procedimientos

### TestProcedimientos.java

package com.techdam;

import com.techdam.service.ProcedimientosService;
import java.math.BigDecimal;

public class TestProcedimientos {
    
    public static void main(String[] args) {
        System.out.println("\n" + "=".repeat(80));
        System.out.println("🧪 PRUEBA DE PROCEDIMIENTOS ALMACENADOS");
        System.out.println("=".repeat(80));
        
        // Test 1: Incrementar salario
        System.out.println("\n--- TEST 1: Incrementar Salario ---");
        ProcedimientosService.incrementarSalario(1, 5.0);
        
        // Test 2: Asignar empleado a proyecto
        System.out.println("\n--- TEST 2: Asignar Empleado ---");
        ProcedimientosService.asignarEmpleadoProyecto(1, 2, 80, "Backend Developer");
        
        // Test 3: Reporte de empleado
        System.out.println("\n--- TEST 3: Reporte Empleado ---");
        ProcedimientosService.generarReporteEmpleado(1);
        
        // Test 4: Contar empleados
        System.out.println("\n--- TEST 4: Contar Empleados por Departamento ---");
        int totalDesarrollo = ProcedimientosService.contarEmpleadosDepartamento("Desarrollo");
        int totalQA = ProcedimientosService.contarEmpleadosDepartamento("QA");
        System.out.println("\nTotal general: " + (totalDesarrollo + totalQA));
        
        // Test 5: Calcular costo de proyecto
        System.out.println("\n--- TEST 5: Calcular Costo Proyecto ---");
        BigDecimal costo1 = ProcedimientosService.calcularCostoProyecto(1);
        BigDecimal costo2 = ProcedimientosService.calcularCostoProyecto(2);
        System.out.println("\nCosto total de ambos proyectos: $" + costo1.add(costo2));
        
        // Test 6: Transferir presupuesto
        System.out.println("\n--- TEST 6: Transferir Presupuesto ---");
        ProcedimientosService.transferirPresupuesto(1, 2, new BigDecimal("5000.00"));
        
        System.out.println("\n" + "=".repeat(80));
        System.out.println("✅ Pruebas completadas");
        System.out.println("=".repeat(80));
    }
}

## Comparativa: Lógica en Java vs BD

### Escenario: Incrementar salarios del departamento

#### Opción 1: Todo en Java

// Java hace TODO el trabajo
public void incrementarSalariosJava(String depto, double pct) {
    // 1. SELECT para obtener empleados
    List<Empleado> empleados = dao.obtenerPorDepartamento(depto);
    
    // 2. Calcular nuevos salarios en Java
    for (Empleado emp : empleados) {
        BigDecimal nuevoSalario = emp.getSalario()
            .multiply(BigDecimal.valueOf(1 + pct/100));
        emp.setSalario(nuevoSalario);
        
        // 3. UPDATE individual para cada empleado
        dao.actualizar(emp);
    }
}
// Tráfico de red: N SELECTs + N UPDATEs

#### Opción 2: Procedimiento Almacenado

// Procedimiento en MySQL
CREATE PROCEDURE incrementar_salarios_departamento(
    IN p_departamento VARCHAR(50),
    IN p_porcentaje DECIMAL(5,2)
)
BEGIN
    UPDATE empleados 
    SET salario = salario * (1 + p_porcentaje / 100)
    WHERE departamento = p_departamento AND activo = TRUE;
END;

// Java solo invoca
public void incrementarSalariosProc(String depto, double pct) {
    CallableStatement cstmt = conn.prepareCall(
        "{call incrementar_salarios_departamento(?, ?)}");
    cstmt.setString(1, depto);
    cstmt.setBigDecimal(2, BigDecimal.valueOf(pct));
    cstmt.execute();
}
// Tráfico de red: 1 CALL

### Comparativa de Rendimiento

|Aspecto|Java|Procedimiento|
|---|---|---|
|Consultas a BD|N+1|1|
|Tráfico de red|Alto|Bajo|
|Procesamiento|Cliente|Servidor|
|Velocidad|Lento|Rápido|
|Mantenimiento|Fácil|Medio|
|Portabilidad|Alta|Baja|

## Evaluación del Bloque

### Checklist

- [ ] Entiendo qué son los procedimientos almacenados
    
- [ ] Sé cuándo usar procedimientos vs código Java
    
- [ ] Uso CallableStatement correctamente
    
- [ ] Manejo parámetros IN, OUT e INOUT
    
- [ ] Llamo funciones almacenadas
    
- [ ] Proceso múltiples ResultSets
    
- [ ] Manejo excepciones de procedimientos

## Resumen

En este bloque hemos dominado:

- ✅ Procedimientos almacenados y sus ventajas
    
- ✅ CallableStatement para invocarlos
    
- ✅ Parámetros IN, OUT e INOUT
    
- ✅ Funciones que devuelven valores
    
- ✅ Procesamiento de múltiples ResultSets
    
- ✅ Cuándo usar lógica en BD vs Java
    

### Regla de Oro

> «Usa procedimientos para operaciones que afecten muchos datos. Usa Java para lógica de negocio compleja.»

**→ [Ir al Bloque 6: Gestión de Transacciones](https://campusfp.dvsweb.es/accesoDatos/ud2/content/Proyecto/06_gestion_transacciones.html)**

## 💡 Notas del Profesor

> **Buena práctica**: Documenta tus procedimientos almacenados en el propio código SQL con comentarios.

> **Debate**: ¿Dónde debería estar la lógica? En Java (portable, testeable) o en BD (rápido, cerca de los datos). Depende del caso de uso.