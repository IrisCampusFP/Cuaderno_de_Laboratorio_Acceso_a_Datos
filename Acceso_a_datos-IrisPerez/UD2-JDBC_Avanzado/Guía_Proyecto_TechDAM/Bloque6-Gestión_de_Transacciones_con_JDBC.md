## Objetivos del Bloque

- Entender el concepto de transacción y propiedades ACID
    
- Implementar transacciones con commit y rollback
    
- Usar savepoints para rollback parcial
    
- Gestionar niveles de aislamiento
    
- Aplicar transacciones en operaciones críticas de TechDAM

## Teoría: Transacciones

### ¿Qué es una Transacción?

Una **transacción** es una unidad de trabajo que agrupa varias operaciones en la BD:

INICIO TRANSACCIÓN
  ├─ Operación 1: Restar dinero de cuenta A
  ├─ Operación 2: Sumar dinero a cuenta B
  └─ Operación 3: Registrar movimiento
FIN TRANSACCIÓN (COMMIT o ROLLBACK)

**Principio**: O se ejecutan **TODAS** las operaciones o **NINGUNA**.

### Propiedades ACID

|Propiedad|Descripción|Ejemplo|
|---|---|---|
|**A**tomicity|Todo o nada|Transferencia: se resta Y se suma, o no pasa nada|
|**C**onsistency|Integridad de datos|El saldo total antes = saldo total después|
|**I**solation|Transacciones independientes|Otras operaciones no ven cambios hasta commit|
|**D**urability|Los cambios persisten|Tras commit, los datos están en disco|

### Sintaxis en JDBC

Connection conn = null;
try {
    conn = DatabaseConfig.getConnection();
    
    // 1. Desactivar autocommit
    conn.setAutoCommit(false);
    
    // 2. Ejecutar operaciones
    operacion1(conn);
    operacion2(conn);
    operacion3(conn);
    
    // 3. Confirmar si todo salió bien
    conn.commit();
    System.out.println("✅ Transacción completada");
    
} catch (SQLException e) {
    // 4. Revertir si hubo error
    if (conn != null) {
        try {
            conn.rollback();
            System.out.println("⚠️ Transacción revertida");
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
    }
} finally {
    // 5. Restaurar autocommit y cerrar
    if (conn != null) {
        try {
            conn.setAutoCommit(true);
        } catch (SQLException e) {}
    }
    DatabaseConfig.closeConnection(conn);
}

## 💻 Práctica 1: Transferencia Bancaria Básica

### Escenario

Vamos a simular una transferencia de presupuesto entre dos proyectos.

#### `src/main/java/com/techdam/service/TransferService.java`

package com.techdam.service;

import com.techdam.config.DatabaseConfig;
import java.math.BigDecimal;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Servicio para transferencias de presupuesto entre proyectos
 * Implementa transacciones ACID
 */
public class TransferService {
    
    /**
     * Transfiere presupuesto de un proyecto a otro
     * 
     * @param idProyectoOrigen ID del proyecto origen
     * @param idProyectoDestino ID del proyecto destino
     * @param monto Cantidad a transferir
     * @return true si la transferencia fue exitosa
     */
    public boolean transferirPresupuesto(int idProyectoOrigen, 
                                        int idProyectoDestino, 
                                        BigDecimal monto) {
        
        Connection conn = null;
        PreparedStatement pstmtOrigen = null;
        PreparedStatement pstmtDestino = null;
        PreparedStatement pstmtConsulta = null;
        ResultSet rs = null;
        
        try {
            System.out.println("\n💰 Iniciando transferencia de presupuesto...");
            System.out.println("   Proyecto Origen: " + idProyectoOrigen);
            System.out.println("   Proyecto Destino: " + idProyectoDestino);
            System.out.println("   Monto: $" + monto);
            
            // Obtener conexión
            conn = DatabaseConfig.getConnection();
            
            // 1. DESACTIVAR AUTOCOMMIT
            conn.setAutoCommit(false);
            System.out.println("\n🔧 AutoCommit desactivado");
            
            // 2. VERIFICAR SALDO SUFICIENTE
            String sqlConsulta = "SELECT presupuesto FROM proyectos WHERE id = ?";
            pstmtConsulta = conn.prepareStatement(sqlConsulta);
            pstmtConsulta.setInt(1, idProyectoOrigen);
            rs = pstmtConsulta.executeQuery();
            
            if (rs.next()) {
                BigDecimal presupuestoActual = rs.getBigDecimal("presupuesto");
                System.out.println("✅ Presupuesto actual origen: $" + presupuestoActual);
                
                if (presupuestoActual.compareTo(monto) < 0) {
                    throw new SQLException("❌ Presupuesto insuficiente en proyecto origen");
                }
            } else {
                throw new SQLException("❌ Proyecto origen no encontrado");
            }
            rs.close();
            pstmtConsulta.close();
            
            // 3. RESTAR DEL PROYECTO ORIGEN
            String sqlRestar = "UPDATE proyectos SET presupuesto = presupuesto - ? WHERE id = ?";
            pstmtOrigen = conn.prepareStatement(sqlRestar);
            pstmtOrigen.setBigDecimal(1, monto);
            pstmtOrigen.setInt(2, idProyectoOrigen);
            
            int filasOrigen = pstmtOrigen.executeUpdate();
            System.out.println("✅ Restado $" + monto + " del proyecto " + idProyectoOrigen);
            
            // 4. SUMAR AL PROYECTO DESTINO
            String sqlSumar = "UPDATE proyectos SET presupuesto = presupuesto + ? WHERE id = ?";
            pstmtDestino = conn.prepareStatement(sqlSumar);
            pstmtDestino.setBigDecimal(1, monto);
            pstmtDestino.setInt(2, idProyectoDestino);
            
            int filasDestino = pstmtDestino.executeUpdate();
            System.out.println("✅ Sumado $" + monto + " al proyecto " + idProyectoDestino);
            
            // 5. VALIDAR QUE SE ACTUALIZARON AMBOS
            if (filasOrigen != 1 || filasDestino != 1) {
                throw new SQLException("Error: No se pudieron actualizar ambos proyectos");
            }
            
            // 6. COMMIT - Confirmar transacción
            conn.commit();
            System.out.println("\n✅ TRANSACCIÓN COMPLETADA CON ÉXITO");
            System.out.println("   Transferencia confirmada permanentemente\n");
            
            return true;
            
        } catch (SQLException e) {
            // 7. ROLLBACK - Revertir si hubo error
            System.err.println("\n❌ ERROR EN LA TRANSACCIÓN: " + e.getMessage());
            
            if (conn != null) {
                try {
                    conn.rollback();
                    System.out.println("⚠️ ROLLBACK EJECUTADO");
                    System.out.println("   Todos los cambios han sido revertidos\n");
                } catch (SQLException ex) {
                    System.err.println("❌ Error al hacer rollback: " + ex.getMessage());
                }
            }
            return false;
            
        } finally {
            // 8. LIMPIAR RECURSOS
            try {
                if (rs != null) rs.close();
                if (pstmtConsulta != null) pstmtConsulta.close();
                if (pstmtOrigen != null) pstmtOrigen.close();
                if (pstmtDestino != null) pstmtDestino.close();
                
                // 9. RESTAURAR AUTOCOMMIT
                if (conn != null) {
                    conn.setAutoCommit(true);
                    System.out.println("🔧 AutoCommit restaurado");
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            
            DatabaseConfig.closeConnection(conn);
        }
    }
    
    /**
     * Muestra el presupuesto actual de un proyecto
     */
    public void mostrarPresupuesto(int idProyecto) {
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            String sql = "SELECT nombre, presupuesto FROM proyectos WHERE id = ?";
            pstmt = conn.prepareStatement(sql);
            pstmt.setInt(1, idProyecto);
            rs = pstmt.executeQuery();
            
            if (rs.next()) {
                System.out.println("📊 Proyecto: " + rs.getString("nombre"));
                System.out.println("   Presupuesto: $" + rs.getBigDecimal("presupuesto"));
            }
            
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
}

### Test de la Transferencia

#### `src/main/java/com/techdam/TestTransferencia.java`

package com.techdam;

import com.techdam.service.TransferService;
import java.math.BigDecimal;

public class TestTransferencia {
    
    public static void main(String[] args) {
        TransferService service = new TransferService();
        
        System.out.println("\n========================================");
        System.out.println("🧪 PRUEBA DE TRANSACCIONES");
        System.out.println("========================================");
        
        // CASO 1: Transferencia exitosa
        System.out.println("\n--- CASO 1: Transferencia Exitosa ---");
        System.out.println("\n📊 ESTADO INICIAL:");
        service.mostrarPresupuesto(1);
        service.mostrarPresupuesto(2);
        
        boolean exito1 = service.transferirPresupuesto(1, 2, new BigDecimal("10000.00"));
        
        System.out.println("\n📊 ESTADO FINAL:");
        service.mostrarPresupuesto(1);
        service.mostrarPresupuesto(2);
        
        // CASO 2: Transferencia con saldo insuficiente (debe hacer rollback)
        System.out.println("\n\n--- CASO 2: Presupuesto Insuficiente ---");
        System.out.println("\n📊 ESTADO INICIAL:");
        service.mostrarPresupuesto(2);
        
        boolean exito2 = service.transferirPresupuesto(2, 1, new BigDecimal("999999.00"));
        
        System.out.println("\n📊 ESTADO FINAL (debería ser igual al inicial):");
        service.mostrarPresupuesto(2);
        
        System.out.println("\n\n✅ Pruebas completadas");
        System.out.println("   Caso 1 exitoso: " + exito1);
        System.out.println("   Caso 2 revertido: " + !exito2);
    }
}

## 💻 Práctica 2: Savepoints (Rollback Parcial)

### ¿Qué son los Savepoints?

Los **savepoints** permiten hacer rollback parcial dentro de una transacción:

INICIO TRANSACCIÓN
  ├─ Operación 1 ✅
  ├─ SAVEPOINT A
  ├─ Operación 2 ✅
  ├─ SAVEPOINT B
  ├─ Operación 3 ❌ (error)
  └─ ROLLBACK TO SAVEPOINT B  ← Sólo revierte Op3
COMMIT (Op1 y Op2 se guardan)

### Ejemplo: Asignación Múltiple con Savepoints

#### `src/main/java/com/techdam/service/AsignacionService.java`

package com.techdam.service;

import com.techdam.config.DatabaseConfig;
import java.sql.*;
import java.time.LocalDate;

/**
 * Servicio para asignar empleados a proyectos
 * Usa savepoints para rollback parcial
 */
public class AsignacionService {
    
    /**
     * Asigna múltiples empleados a un proyecto usando savepoints
     * Si un empleado falla, no se asigna pero los demás sí
     * 
     * @param idsEmpleados Array de IDs de empleados
     * @param idProyecto ID del proyecto
     * @param horasPorEmpleado Horas asignadas a cada uno
     */
    public void asignarEmpleadosConSavepoints(int[] idsEmpleados, 
                                             int idProyecto, 
                                             int horasPorEmpleado) {
        
        Connection conn = null;
        PreparedStatement pstmt = null;
        Savepoint[] savepoints = new Savepoint[idsEmpleados.length];
        
        try {
            conn = DatabaseConfig.getConnection();
            conn.setAutoCommit(false);
            
            System.out.println("\n🔧 Iniciando asignaciones múltiples...");
            
            String sql = "INSERT INTO asignaciones " +
                        "(empleado_id, proyecto_id, fecha_asignacion, horas_asignadas, rol) " +
                        "VALUES (?, ?, ?, ?, ?)";
            pstmt = conn.prepareStatement(sql);
            
            int asignacionesExitosas = 0;
            
            for (int i = 0; i < idsEmpleados.length; i++) {
                try {
                    // Crear savepoint ANTES de cada asignación
                    savepoints[i] = conn.setSavepoint("SP_Empleado_" + idsEmpleados[i]);
                    System.out.println("\n💾 Savepoint creado: " + savepoints[i].getSavepointName());
                    
                    // Intentar asignar empleado
                    pstmt.setInt(1, idsEmpleados[i]);
                    pstmt.setInt(2, idProyecto);
                    pstmt.setDate(3, Date.valueOf(LocalDate.now()));
                    pstmt.setInt(4, horasPorEmpleado);
                    pstmt.setString(5, "Desarrollador");
                    
                    pstmt.executeUpdate();
                    
                    System.out.println("✅ Empleado " + idsEmpleados[i] + 
                                     " asignado correctamente");
                    asignacionesExitosas++;
                    
                } catch (SQLException e) {
                    // Si falla UNA asignación, hacer rollback al savepoint
                    System.err.println("⚠️ Error al asignar empleado " + idsEmpleados[i] + 
                                     ": " + e.getMessage());
                    
                    if (savepoints[i] != null) {
                        conn.rollback(savepoints[i]);
                        System.out.println("   🔙 Rollback al savepoint: " + 
                                         savepoints[i].getSavepointName());
                        System.out.println("   ℹ️ Las asignaciones anteriores se mantienen");
                    }
                }
            }
            
            // Confirmar todas las asignaciones exitosas
            conn.commit();
            System.out.println("\n✅ TRANSACCIÓN COMPLETADA");
            System.out.println("   Total asignaciones exitosas: " + asignacionesExitosas + 
                             "/" + idsEmpleados.length);
            
        } catch (SQLException e) {
            System.err.println("❌ Error crítico: " + e.getMessage());
            try {
                if (conn != null) {
                    conn.rollback();
                    System.out.println("⚠️ Rollback completo ejecutado");
                }
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
        } finally {
            try {
                if (pstmt != null) pstmt.close();
                if (conn != null) conn.setAutoCommit(true);
            } catch (SQLException e) {}
            DatabaseConfig.closeConnection(conn);
        }
    }
}

### Test de Savepoints

#### `src/main/java/com/techdam/TestSavepoints.java`

package com.techdam;

import com.techdam.service.AsignacionService;

public class TestSavepoints {
    
    public static void main(String[] args) {
        AsignacionService service = new AsignacionService();
        
        System.out.println("\n========================================");
        System.out.println("🧪 PRUEBA DE SAVEPOINTS");
        System.out.println("========================================");
        
        // Array con IDs válidos e inválidos
        int[] empleados = {1, 2, 999, 3}; // 999 no existe (producirá error)
        int proyecto = 1;
        int horas = 80;
        
        System.out.println("\n📋 Intentando asignar empleados: [1, 2, 999, 3]");
        System.out.println("   (El empleado 999 no existe y fallará)");
        
        service.asignarEmpleadosConSavepoints(empleados, proyecto, horas);
        
        System.out.println("\n💡 RESULTADO ESPERADO:");
        System.out.println("   ✅ Empleado 1: Asignado");
        System.out.println("   ✅ Empleado 2: Asignado");
        System.out.println("   ❌ Empleado 999: Falló (rollback a su savepoint)");
        System.out.println("   ✅ Empleado 3: Asignado");
        System.out.println("\n   Los empleados 1, 2 y 3 quedan asignados al proyecto.");
    }
}


## 🎚️ Niveles de Aislamiento

### Problemas de Concurrencia

|Problema|Descripción|
|---|---|
|**Dirty Read**|Leer cambios no confirmados (sin commit)|
|**Non-Repeatable Read**|Leer el mismo dato dos veces y obtener resultados diferentes|
|**Phantom Read**|Aparecen/desaparecen filas entre lecturas|

### Niveles de Aislamiento en JDBC

Connection conn = DatabaseConfig.getConnection();

// Niveles disponibles (de menos a más restrictivo)
conn.setTransactionIsolation(Connection.TRANSACTION_READ_UNCOMMITTED); // ⚠️ Permite dirty reads
conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);   // ✅ Recomendado
conn.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);  // 🔒 Más seguro
conn.setTransactionIsolation(Connection.TRANSACTION_SERIALIZABLE);     // 🔐 Máxima seguridad

|Nivel|Dirty Read|Non-Repeatable|Phantom|
|---|---|---|---|
|READ_UNCOMMITTED|❌ Permite|❌ Permite|❌ Permite|
|READ_COMMITTED|✅ Previene|❌ Permite|❌ Permite|
|REPEATABLE_READ|✅ Previene|✅ Previene|❌ Permite|
|SERIALIZABLE|✅ Previene|✅ Previene|✅ Previene|

## Ejercicio Guiado: Incremento de Salarios con Transacción

### Enunciado

Implementa un método que:

1. Incremente el salario de todos los empleados de un departamento en un %
    
2. Registre el cambio en una tabla de auditoría
    
3. Si falla alguna operación, haga rollback completo

### Solución

public boolean incrementarSalariosPorDepartamento(String departamento, double porcentaje) {
    Connection conn = null;
    PreparedStatement pstmtUpdate = null;
    PreparedStatement pstmtLog = null;
    
    try {
        conn = DatabaseConfig.getConnection();
        conn.setAutoCommit(false);
        
        // 1. Incrementar salarios
        String sqlUpdate = "UPDATE empleados SET salario = salario * (1 + ? / 100) " +
                          "WHERE departamento = ? AND activo = TRUE";
        pstmtUpdate = conn.prepareStatement(sqlUpdate);
        pstmtUpdate.setDouble(1, porcentaje);
        pstmtUpdate.setString(2, departamento);
        
        int empleadosAfectados = pstmtUpdate.executeUpdate();
        System.out.println("✅ " + empleadosAfectados + " salarios incrementados");
        
        // 2. Registrar en auditoría
        String sqlLog = "INSERT INTO auditoria_salarios " +
                       "(departamento, porcentaje, fecha, empleados_afectados) " +
                       "VALUES (?, ?, NOW(), ?)";
        pstmtLog = conn.prepareStatement(sqlLog);
        pstmtLog.setString(1, departamento);
        pstmtLog.setDouble(2, porcentaje);
        pstmtLog.setInt(3, empleadosAfectados);
        pstmtLog.executeUpdate();
        
        System.out.println("✅ Auditoría registrada");
        
        // 3. Confirmar
        conn.commit();
        System.out.println("✅ Transacción completada");
        return true;
        
    } catch (SQLException e) {
        System.err.println("❌ Error: " + e.getMessage());
        try {
            if (conn != null) conn.rollback();
        } catch (SQLException ex) {}
        return false;
    } finally {
        // Cerrar recursos...
    }
}


## 📊 Mejores Prácticas con Transacciones

### ✅ SÍ hacer

1. **Transacciones cortas**: Minimizar el tiempo con autocommit desactivado
    
2. **Restaurar autocommit**: Siempre en el bloque finally
    
3. **Usar savepoints**: Para rollback parcial cuando tenga sentido
    
4. **Validar antes**: Verificar condiciones antes de iniciar la transacción
    
5. **Log de errores**: Registrar qué falló y por qué

### ❌ NO hacer

1. **Transacciones largas**: Bloquean recursos y reducen concurrencia
    
2. **Operaciones costosas**: Evitar cálculos complejos dentro de transacciones
    
3. **Interacción con usuario**: No pedir input durante una transacción
    
4. **Olvidar rollback**: Siempre manejar el caso de error


## 🎯 Evaluación del Bloque

### Checklist

- [ ] Entiendo las propiedades ACID
    
- [ ] Sé cuándo usar transacciones
    
- [ ] Puedo implementar commit y rollback
    
- [ ] Entiendo el uso de savepoints
    
- [ ] Conozco los niveles de aislamiento
    
- [ ] Aplico buenas prácticas en transacciones


## 📚 Resumen

En este bloque hemos aprendido:

- ✅ Propiedades ACID y su importancia
    
- ✅ Cómo implementar transacciones en JDBC
    
- ✅ Uso de commit, rollback y savepoints
    
- ✅ Niveles de aislamiento de transacciones
    
- ✅ Buenas prácticas para operaciones atómicas

## 💡 Notas del Profesor

> **Regla de oro**: Si tienes varias operaciones que DEBEN ejecutarse todas o ninguna, usa transacciones.

> **Consejo**: En aplicaciones reales, considera usar frameworks como Spring que gestionan transacciones con anotaciones (@Transactional).