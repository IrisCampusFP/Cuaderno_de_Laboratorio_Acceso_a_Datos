## Objetivos

- Integrar todos los conceptos aprendidos
    
- Construir un sistema completo funcional
    
- Aplicar arquitectura en capas
    
- Implementar casos de uso reales

## Arquitectura del Sistema

┌──────────────────────────────────────┐
│         CAPA PRESENTACIÓN            │  Main.java
└──────────────────────────────────────┘  (Menú CLI)
              ↕
┌──────────────────────────────────────┐
│         CAPA SERVICIO                │  EmpleadoService
│  (Lógica de negocio + Transacciones) │  ProyectoService
└──────────────────────────────────────┘  AsignacionService
              ↕
┌──────────────────────────────────────┐
│         CAPA DAO                     │  EmpleadoDAO
│  (Acceso a datos)                    │  ProyectoDAO
└──────────────────────────────────────┘  AsignacionDAO
              ↕
┌──────────────────────────────────────┐
│         BASE DE DATOS                │  MySQL
│  + Procedimientos Almacenados        │  + HikariCP
└──────────────────────────────────────┘

## 💻 Sistema Completo

### Main.java - Menú Principal

package com.techdam;

import com.techdam.service.*;
import java.util.Scanner;

public class Main {
    
    private static Scanner scanner = new Scanner(System.in);
    private static EmpleadoService empleadoService = new EmpleadoService();
    private static ProyectoService proyectoService = new ProyectoService();
    
    public static void main(String[] args) {
        boolean salir = false;
        
        while (!salir) {
            mostrarMenu();
            int opcion = scanner.nextInt();
            scanner.nextLine();
            
            switch (opcion) {
                case 1 -> gestionarEmpleados();
                case 2 -> gestionarProyectos();
                case 3 -> gestionarAsignaciones();
                case 4 -> generarInformes();
                case 0 -> salir = true;
            }
        }
        
        System.out.println("¡Hasta luego!");
    }
    
    private static void mostrarMenu() {
        System.out.println("\n" + "=".repeat(50));
        System.out.println("   🏢 SISTEMA TECHDAM");
        System.out.println("=".repeat(50));
        System.out.println("1. 👥 Gestión de Empleados");
        System.out.println("2. 📂 Gestión de Proyectos");
        System.out.println("3. 🔗 Asignaciones");
        System.out.println("4. 📊 Informes");
        System.out.println("0. ❌ Salir");
        System.out.print("\nOpción: ");
    }
    
    // ... métodos de gestión
}

## 📋 Casos de Uso Implementados

### CU-01: Crear Empleado con Validaciones

public class EmpleadoService {
    
    private EmpleadoDAO dao = new EmpleadoDAO();
    
    public boolean crearEmpleado(Empleado empleado) {
        // Validaciones de negocio
        if (!validarEmail(empleado.getEmail())) {
            System.out.println("❌ Email inválido");
            return false;
        }
        
        if (empleado.getSalario().compareTo(BigDecimal.ZERO) <= 0) {
            System.out.println("❌ Salario debe ser positivo");
            return false;
        }
        
        // Verificar email único
        if (dao.existeEmail(empleado.getEmail())) {
            System.out.println("❌ Email ya registrado");
            return false;
        }
        
        // Crear empleado
        int id = dao.crear(empleado);
        return id > 0;
    }
}

### CU-02: Asignar Múltiples Empleados a Proyecto

public boolean asignarEquipo(int proyectoId, List<Integer> empleadoIds) {
    Connection conn = null;
    
    try {
        conn = DatabaseConfigPool.getConnection();
        conn.setAutoCommit(false);
        
        for (int empId : empleadoIds) {
            asignacionDAO.crear(conn, empId, proyectoId);
        }
        
        conn.commit();
        return true;
        
    } catch (SQLException e) {
        rollback(conn);
        return false;
    }
}

### CU-03: Dashboard de Métricas

public void mostrarDashboard() {
    System.out.println("\n📊 DASHBOARD TECHDAM");
    System.out.println("=".repeat(60));
    
    // Métricas generales
    int totalEmpleados = empleadoDAO.contar();
    int totalProyectos = proyectoDAO.contar();
    BigDecimal presupuestoTotal = proyectoDAO.sumarPresupuestos();
    
    System.out.println("👥 Empleados activos: " + totalEmpleados);
    System.out.println("📂 Proyectos totales: " + totalProyectos);
    System.out.println("💰 Presupuesto total: $" + presupuestoTotal);
    
    // Métricas por departamento
    Map<String, Integer> porDepto = empleadoDAO.contarPorDepartamento();
    System.out.println("\n📋 Por departamento:");
    porDepto.forEach((depto, count) -> 
        System.out.println("   " + depto + ": " + count));
    
    // Top 5 proyectos por presupuesto
    System.out.println("\n🏆 Top 5 proyectos:");
    proyectoDAO.obtenerTop(5).forEach(p -> 
        System.out.println("   " + p.getNombre() + ": $" + p.getPresupuesto()));
}


## 🧪 Testing del Sistema

### Test de Integración

public class IntegrationTest {
    
    public static void main(String[] args) {
        testFlujoCompleto();
    }
    
    private static void testFlujoCompleto() {
        System.out.println("🧪 TEST DE INTEGRACIÓN");
        
        // 1. Crear empleados
        EmpleadoService empService = new EmpleadoService();
        Empleado emp1 = empService.crear("Test", "User", "test@techdam.com", ...);
        
        // 2. Crear proyecto
        ProyectoService proyService = new ProyectoService();
        Proyecto proy = proyService.crear("Test Project", ...);
        
        // 3. Asignar empleado
        AsignacionService asigService = new AsignacionService();
        asigService.asignar(emp1.getId(), proy.getId(), 80, "Developer");
        
        // 4. Incrementar salario con procedimiento
        ProcedimientosService.incrementarSalario(emp1.getId(), 10.0);
        
        // 5. Transferir presupuesto con transacción
        TransferService.transferir(proy.getId(), otroProyId, 1000);
        
        // 6. Generar reporte
        ProcedimientosService.generarReporte(emp1.getId());
        
        // 7. Cleanup
        empService.eliminar(emp1.getId());
        proyService.eliminar(proy.getId());
        
        System.out.println("\n✅ Test completado");
    }
}


## Estructura Final del Proyecto

src/main/java/com/techdam/
├── Main.java
├── config/
│   └── DatabaseConfigPool.java
├── model/
│   ├── Empleado.java
│   ├── Proyecto.java
│   └── Asignacion.java
├── dao/
│   ├── EmpleadoDAO.java
│   ├── ProyectoDAO.java
│   └── AsignacionDAO.java
├── service/
│   ├── EmpleadoService.java
│   ├── ProyectoService.java
│   ├── AsignacionService.java
│   ├── TransferService.java
│   └── ProcedimientosService.java
└── util/
    ├── MetadataExplorer.java
    ├── ConsultaDinamica.java
    └── GeneradorInformes.java

## Evaluación

### Checklist de Integración

- [ ] Todas las capas están separadas correctamente
    
- [ ] Se usa HikariCP para conexiones
    
- [ ] PreparedStatement en todas las consultas
    
- [ ] Transacciones en operaciones críticas
    
- [ ] Procedimientos almacenados invocados
    
- [ ] Manejo de excepciones apropiado
    
- [ ] Código documentado con JavaDoc
    
- [ ] Tests de integración funcionan

## Resumen

Hemos integrado:

- ✅ Arquitectura en capas (Presentación → Servicio → DAO → BD)
    
- ✅ Pool de conexiones con HikariCP
    
- ✅ PreparedStatement + CallableStatement
    
- ✅ Transacciones con savepoints
    
- ✅ Procedimientos almacenados
    
- ✅ Consultas avanzadas con metadatos
    
- ✅ Sistema completo funcional