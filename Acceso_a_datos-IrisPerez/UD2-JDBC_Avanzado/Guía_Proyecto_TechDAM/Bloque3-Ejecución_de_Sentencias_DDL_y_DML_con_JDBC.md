## Objetivos del Bloque

- Crear tablas usando sentencias DDL desde Java
    
- Implementar operaciones CRUD completas
    
- Dominar `PreparedStatement` para prevenir inyección SQL
    
- Construir el modelo de datos de TechDAM

## Teoría: DDL vs DML

### Data Definition Language (DDL)

Sentencias que **definen la estructura** de la base de datos:

CREATE TABLE ...    -- Crear tablas
ALTER TABLE ...     -- Modificar estructura
DROP TABLE ...      -- Eliminar tablas
TRUNCATE TABLE ...  -- Vaciar tabla

### Data Manipulation Language (DML)

Sentencias que **manipulan los datos**:

SELECT ...  -- Leer datos
INSERT ...  -- Insertar datos
UPDATE ...  -- Actualizar datos
DELETE ...  -- Eliminar datos

## Statement vs PreparedStatement

### ❌ Statement (NUNCA USAR EN PRODUCCIÓN)

// VULNERABLE A INYECCIÓN SQL
String nombre = "Juan'; DROP TABLE empleados; --";
String sql = "SELECT * FROM empleados WHERE nombre = '" + nombre + "'";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sql);

**Problema**: El nombre malicioso ejecutaría: `DROP TABLE empleados`

### ✅ PreparedStatement (SIEMPRE USAR)

// SEGURO - Los parámetros se escapan automáticamente
String sql = "SELECT * FROM empleados WHERE nombre = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, nombre); // El '?' se reemplaza de forma segura
ResultSet rs = pstmt.executeQuery();

### Ventajas de PreparedStatement

1. **Seguridad**: Previene inyección SQL
    
2. **Rendimiento**: Se compila una vez y se reutiliza
    
3. **Legibilidad**: Código más limpio
    
4. **Tipos**: Validación automática de tipos de datos

## 🗃️ Práctica 1: Crear el Modelo de Datos TechDAM

### Script SQL Completo

#### `scripts/01_crear_tablas.sql`

-- ============================================
-- Script: Creación de tablas para TechDAM
-- Autor: DAM - Acceso a Datos
-- ============================================

USE techdam;

-- Eliminar tablas si existen (en orden inverso por FKs)
DROP TABLE IF EXISTS asignaciones;
DROP TABLE IF EXISTS proyectos;
DROP TABLE IF EXISTS empleados;

-- ============================================
-- TABLA: empleados
-- ============================================
CREATE TABLE empleados (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    salario DECIMAL(10, 2) NOT NULL CHECK (salario > 0),
    fecha_contrato DATE NOT NULL,
    departamento VARCHAR(50) NOT NULL,
    activo BOOLEAN DEFAULT TRUE,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_modificacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_departamento (departamento),
    INDEX idx_email (email)
) ENGINE=InnoDB COMMENT='Tabla de empleados de TechDAM';

-- ============================================
-- TABLA: proyectos
-- ============================================
CREATE TABLE proyectos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT,
    presupuesto DECIMAL(12, 2) NOT NULL CHECK (presupuesto >= 0),
    fecha_inicio DATE NOT NULL,
    fecha_fin DATE,
    estado ENUM('PLANIFICACION', 'EN_CURSO', 'PAUSADO', 'COMPLETADO', 'CANCELADO') 
           DEFAULT 'PLANIFICACION',
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_modificacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_estado (estado),
    CHECK (fecha_fin IS NULL OR fecha_fin >= fecha_inicio)
) ENGINE=InnoDB COMMENT='Tabla de proyectos';

-- ============================================
-- TABLA: asignaciones
-- ============================================
CREATE TABLE asignaciones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    empleado_id INT NOT NULL,
    proyecto_id INT NOT NULL,
    fecha_asignacion DATE NOT NULL,
    horas_asignadas INT NOT NULL CHECK (horas_asignadas > 0),
    rol VARCHAR(50) DEFAULT 'Desarrollador',
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (empleado_id) REFERENCES empleados(id) 
        ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (proyecto_id) REFERENCES proyectos(id) 
        ON DELETE CASCADE ON UPDATE CASCADE,
        
    UNIQUE KEY uk_empleado_proyecto (empleado_id, proyecto_id),
    INDEX idx_empleado (empleado_id),
    INDEX idx_proyecto (proyecto_id)
) ENGINE=InnoDB COMMENT='Asignación de empleados a proyectos';

-- ============================================
-- Insertar datos de prueba
-- ============================================

-- Empleados
INSERT INTO empleados (nombre, apellido, email, salario, fecha_contrato, departamento) VALUES
('Ana', 'García', 'ana.garcia@techdam.com', 35000.00, '2022-01-15', 'Desarrollo'),
('Carlos', 'Martínez', 'carlos.martinez@techdam.com', 42000.00, '2021-06-20', 'Desarrollo'),
('Laura', 'Rodríguez', 'laura.rodriguez@techdam.com', 38000.00, '2022-03-10', 'QA'),
('Miguel', 'López', 'miguel.lopez@techdam.com', 45000.00, '2020-09-05', 'DevOps'),
('Elena', 'Sánchez', 'elena.sanchez@techdam.com', 50000.00, '2019-11-12', 'Arquitectura');

-- Proyectos
INSERT INTO proyectos (nombre, descripcion, presupuesto, fecha_inicio, fecha_fin, estado) VALUES
('Sistema CRM', 'Desarrollo de CRM para gestión de clientes', 150000.00, '2024-01-01', '2024-06-30', 'EN_CURSO'),
('App Móvil', 'Aplicación móvil multiplataforma', 80000.00, '2024-02-15', '2024-08-15', 'EN_CURSO'),
('Migración Cloud', 'Migración de infraestructura a AWS', 200000.00, '2023-10-01', '2024-03-31', 'COMPLETADO');

-- Asignaciones
INSERT INTO asignaciones (empleado_id, proyecto_id, fecha_asignacion, horas_asignadas, rol) VALUES
(1, 1, '2024-01-01', 160, 'Desarrollador Backend'),
(2, 1, '2024-01-01', 160, 'Desarrollador Frontend'),
(3, 1, '2024-01-15', 80, 'Tester QA'),
(1, 2, '2024-02-15', 80, 'Desarrollador Backend'),
(4, 3, '2023-10-01', 200, 'DevOps Engineer'),
(5, 3, '2023-10-01', 100, 'Arquitecto de Soluciones');

-- Verificación
SELECT 'Tablas creadas e inicializadas correctamente' AS Resultado;
SELECT COUNT(*) AS total_empleados FROM empleados;
SELECT COUNT(*) AS total_proyectos FROM proyectos;
SELECT COUNT(*) AS total_asignaciones FROM asignaciones;

### Ejecutar el Script desde Java

#### `src/main/java/com/techdam/util/DatabaseInitializer.java`

package com.techdam.util;

import com.techdam.config.DatabaseConfig;
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;

/**
 * Inicializador de la base de datos
 * Ejecuta scripts SQL para crear la estructura
 */
public class DatabaseInitializer {
    
    /**
     * Ejecuta un script SQL desde un archivo
     * 
     * @param rutaScript Ruta al archivo SQL
     */
    public static void ejecutarScript(String rutaScript) {
        Connection conn = null;
        Statement stmt = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            stmt = conn.createStatement();
            
            // Leer el script
            String script = leerScript(rutaScript);
            
            // Dividir por punto y coma (cuidado con los procedimientos)
            String[] sentencias = script.split(";");
            
            System.out.println("📜 Ejecutando script: " + rutaScript);
            int ejecutadas = 0;
            
            for (String sentencia : sentencias) {
                sentencia = sentencia.trim();
                
                // Ignorar líneas vacías y comentarios
                if (!sentencia.isEmpty() && 
                    !sentencia.startsWith("--") && 
                    !sentencia.startsWith("//")) {
                    
                    try {
                        stmt.execute(sentencia);
                        ejecutadas++;
                    } catch (SQLException e) {
                        System.err.println("⚠️ Error en sentencia: " + 
                                         sentencia.substring(0, Math.min(50, sentencia.length())));
                        System.err.println("   Motivo: " + e.getMessage());
                    }
                }
            }
            
            System.out.println("✅ Script ejecutado: " + ejecutadas + " sentencias");
            
        } catch (SQLException | IOException e) {
            System.err.println("❌ Error al ejecutar script: " + e.getMessage());
            e.printStackTrace();
        } finally {
            try {
                if (stmt != null) stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
            DatabaseConfig.closeConnection(conn);
        }
    }
    
    /**
     * Lee un archivo SQL completo
     */
    private static String leerScript(String rutaScript) throws IOException {
        StringBuilder script = new StringBuilder();
        
        try (BufferedReader br = new BufferedReader(new FileReader(rutaScript))) {
            String linea;
            while ((linea = br.readLine()) != null) {
                script.append(linea).append("\n");
            }
        }
        
        return script.toString();
    }
    
    public static void main(String[] args) {
        ejecutarScript("scripts/01_crear_tablas.sql");
    }
}


## 💻 Práctica 2: CRUD de Empleados

### Entidad Empleado

#### `src/main/java/com/techdam/model/Empleado.java`

package com.techdam.model;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.time.LocalDateTime;

/**
 * Entidad que representa un empleado de TechDAM
 */
public class Empleado {
    
    private Integer id;
    private String nombre;
    private String apellido;
    private String email;
    private BigDecimal salario;
    private LocalDate fechaContrato;
    private String departamento;
    private Boolean activo;
    private LocalDateTime fechaCreacion;
    private LocalDateTime fechaModificacion;
    
    // Constructor vacío
    public Empleado() {
    }
    
    // Constructor sin ID (para inserciones)
    public Empleado(String nombre, String apellido, String email, 
                   BigDecimal salario, LocalDate fechaContrato, String departamento) {
        this.nombre = nombre;
        this.apellido = apellido;
        this.email = email;
        this.salario = salario;
        this.fechaContrato = fechaContrato;
        this.departamento = departamento;
        this.activo = true;
    }
    
    // Getters y Setters
    public Integer getId() {
        return id;
    }
    
    public void setId(Integer id) {
        this.id = id;
    }
    
    public String getNombre() {
        return nombre;
    }
    
    public void setNombre(String nombre) {
        this.nombre = nombre;
    }
    
    public String getApellido() {
        return apellido;
    }
    
    public void setApellido(String apellido) {
        this.apellido = apellido;
    }
    
    public String getEmail() {
        return email;
    }
    
    public void setEmail(String email) {
        this.email = email;
    }
    
    public BigDecimal getSalario() {
        return salario;
    }
    
    public void setSalario(BigDecimal salario) {
        this.salario = salario;
    }
    
    public LocalDate getFechaContrato() {
        return fechaContrato;
    }
    
    public void setFechaContrato(LocalDate fechaContrato) {
        this.fechaContrato = fechaContrato;
    }
    
    public String getDepartamento() {
        return departamento;
    }
    
    public void setDepartamento(String departamento) {
        this.departamento = departamento;
    }
    
    public Boolean getActivo() {
        return activo;
    }
    
    public void setActivo(Boolean activo) {
        this.activo = activo;
    }
    
    public LocalDateTime getFechaCreacion() {
        return fechaCreacion;
    }
    
    public void setFechaCreacion(LocalDateTime fechaCreacion) {
        this.fechaCreacion = fechaCreacion;
    }
    
    public LocalDateTime getFechaModificacion() {
        return fechaModificacion;
    }
    
    public void setFechaModificacion(LocalDateTime fechaModificacion) {
        this.fechaModificacion = fechaModificacion;
    }
    
    // Métodos de utilidad
    public String getNombreCompleto() {
        return nombre + " " + apellido;
    }
    
    @Override
    public String toString() {
        return String.format("Empleado{id=%d, nombre='%s', apellido='%s', email='%s', " +
                           "salario=%.2f, departamento='%s'}",
                id, nombre, apellido, email, salario, departamento);
    }
}

### DAO (Data Access Object) de Empleados

#### `src/main/java/com/techdam/dao/EmpleadoDAO.java`

package com.techdam.dao;

import com.techdam.config.DatabaseConfig;
import com.techdam.model.Empleado;

import java.math.BigDecimal;
import java.sql.*;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

/**
 * DAO para operaciones CRUD sobre la tabla empleados
 * Implementa el patrón Repository
 */
public class EmpleadoDAO {
    
    // Consultas SQL preparadas
    private static final String INSERT = 
        "INSERT INTO empleados (nombre, apellido, email, salario, fecha_contrato, departamento) " +
        "VALUES (?, ?, ?, ?, ?, ?)";
    
    private static final String UPDATE = 
        "UPDATE empleados SET nombre = ?, apellido = ?, email = ?, " +
        "salario = ?, departamento = ?, activo = ? WHERE id = ?";
    
    private static final String DELETE = 
        "DELETE FROM empleados WHERE id = ?";
    
    private static final String SELECT_ALL = 
        "SELECT * FROM empleados ORDER BY id";
    
    private static final String SELECT_BY_ID = 
        "SELECT * FROM empleados WHERE id = ?";
    
    private static final String SELECT_BY_DEPARTAMENTO = 
        "SELECT * FROM empleados WHERE departamento = ? AND activo = TRUE";
    
    /**
     * Crea un nuevo empleado en la base de datos
     * 
     * @param empleado Empleado a crear
     * @return ID generado
     */
    public int crear(Empleado empleado) {
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            
            // PreparedStatement con RETURN_GENERATED_KEYS para obtener el ID
            pstmt = conn.prepareStatement(INSERT, Statement.RETURN_GENERATED_KEYS);
            
            // Establecer parámetros
            pstmt.setString(1, empleado.getNombre());
            pstmt.setString(2, empleado.getApellido());
            pstmt.setString(3, empleado.getEmail());
            pstmt.setBigDecimal(4, empleado.getSalario());
            pstmt.setDate(5, Date.valueOf(empleado.getFechaContrato()));
            pstmt.setString(6, empleado.getDepartamento());
            
            // Ejecutar
            int filasAfectadas = pstmt.executeUpdate();
            
            if (filasAfectadas > 0) {
                // Obtener ID generado
                rs = pstmt.getGeneratedKeys();
                if (rs.next()) {
                    int id = rs.getInt(1);
                    empleado.setId(id);
                    System.out.println("✅ Empleado creado con ID: " + id);
                    return id;
                }
            }
            
        } catch (SQLException e) {
            System.err.println("❌ Error al crear empleado: " + e.getMessage());
            e.printStackTrace();
        } finally {
            cerrarRecursos(rs, pstmt, conn);
        }
        
        return -1;
    }
    
    /**
     * Obtiene todos los empleados
     * 
     * @return Lista de empleados
     */
    public List<Empleado> obtenerTodos() {
        List<Empleado> empleados = new ArrayList<>();
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            pstmt = conn.prepareStatement(SELECT_ALL);
            rs = pstmt.executeQuery();
            
            while (rs.next()) {
                empleados.add(mapearEmpleado(rs));
            }
            
            System.out.println("📋 Se encontraron " + empleados.size() + " empleados");
            
        } catch (SQLException e) {
            System.err.println("❌ Error al obtener empleados: " + e.getMessage());
        } finally {
            cerrarRecursos(rs, pstmt, conn);
        }
        
        return empleados;
    }
    
    /**
     * Busca un empleado por ID
     * 
     * @param id ID del empleado
     * @return Optional con el empleado si existe
     */
    public Optional<Empleado> obtenerPorId(int id) {
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            pstmt = conn.prepareStatement(SELECT_BY_ID);
            pstmt.setInt(1, id);
            
            rs = pstmt.executeQuery();
            
            if (rs.next()) {
                return Optional.of(mapearEmpleado(rs));
            }
            
        } catch (SQLException e) {
            System.err.println("❌ Error al buscar empleado: " + e.getMessage());
        } finally {
            cerrarRecursos(rs, pstmt, conn);
        }
        
        return Optional.empty();
    }
    
    /**
     * Actualiza un empleado existente
     * 
     * @param empleado Empleado con los datos actualizados
     * @return true si se actualizó correctamente
     */
    public boolean actualizar(Empleado empleado) {
        Connection conn = null;
        PreparedStatement pstmt = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            pstmt = conn.prepareStatement(UPDATE);
            
            pstmt.setString(1, empleado.getNombre());
            pstmt.setString(2, empleado.getApellido());
            pstmt.setString(3, empleado.getEmail());
            pstmt.setBigDecimal(4, empleado.getSalario());
            pstmt.setString(5, empleado.getDepartamento());
            pstmt.setBoolean(6, empleado.getActivo());
            pstmt.setInt(7, empleado.getId());
            
            int filasAfectadas = pstmt.executeUpdate();
            
            if (filasAfectadas > 0) {
                System.out.println("✅ Empleado actualizado: " + empleado.getId());
                return true;
            } else {
                System.out.println("⚠️ No se encontró el empleado con ID: " + empleado.getId());
            }
            
        } catch (SQLException e) {
            System.err.println("❌ Error al actualizar empleado: " + e.getMessage());
        } finally {
            cerrarRecursos(null, pstmt, conn);
        }
        
        return false;
    }
    
    /**
     * Elimina un empleado por ID
     * 
     * @param id ID del empleado
     * @return true si se eliminó correctamente
     */
    public boolean eliminar(int id) {
        Connection conn = null;
        PreparedStatement pstmt = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            pstmt = conn.prepareStatement(DELETE);
            pstmt.setInt(1, id);
            
            int filasAfectadas = pstmt.executeUpdate();
            
            if (filasAfectadas > 0) {
                System.out.println("✅ Empleado eliminado: " + id);
                return true;
            } else {
                System.out.println("⚠️ No se encontró el empleado con ID: " + id);
            }
            
        } catch (SQLException e) {
            System.err.println("❌ Error al eliminar empleado: " + e.getMessage());
        } finally {
            cerrarRecursos(null, pstmt, conn);
        }
        
        return false;
    }
    
    /**
     * Obtiene empleados por departamento
     */
    public List<Empleado> obtenerPorDepartamento(String departamento) {
        List<Empleado> empleados = new ArrayList<>();
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            pstmt = conn.prepareStatement(SELECT_BY_DEPARTAMENTO);
            pstmt.setString(1, departamento);
            
            rs = pstmt.executeQuery();
            
            while (rs.next()) {
                empleados.add(mapearEmpleado(rs));
            }
            
        } catch (SQLException e) {
            System.err.println("❌ Error: " + e.getMessage());
        } finally {
            cerrarRecursos(rs, pstmt, conn);
        }
        
        return empleados;
    }
    
    /**
     * Mapea un ResultSet a un objeto Empleado
     */
    private Empleado mapearEmpleado(ResultSet rs) throws SQLException {
        Empleado empleado = new Empleado();
        empleado.setId(rs.getInt("id"));
        empleado.setNombre(rs.getString("nombre"));
        empleado.setApellido(rs.getString("apellido"));
        empleado.setEmail(rs.getString("email"));
        empleado.setSalario(rs.getBigDecimal("salario"));
        
        Date fecha = rs.getDate("fecha_contrato");
        if (fecha != null) {
            empleado.setFechaContrato(fecha.toLocalDate());
        }
        
        empleado.setDepartamento(rs.getString("departamento"));
        empleado.setActivo(rs.getBoolean("activo"));
        
        Timestamp creacion = rs.getTimestamp("fecha_creacion");
        if (creacion != null) {
            empleado.setFechaCreacion(creacion.toLocalDateTime());
        }
        
        Timestamp modificacion = rs.getTimestamp("fecha_modificacion");
        if (modificacion != null) {
            empleado.setFechaModificacion(modificacion.toLocalDateTime());
        }
        
        return empleado;
    }
    
    /**
     * Cierra recursos JDBC de forma segura
     */
    private void cerrarRecursos(ResultSet rs, PreparedStatement pstmt, Connection conn) {
        try {
            if (rs != null) rs.close();
            if (pstmt != null) pstmt.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        DatabaseConfig.closeConnection(conn);
    }
}

### Clase de Prueba del CRUD

#### `src/main/java/com/techdam/TestCRUD.java`

package com.techdam;

import com.techdam.dao.EmpleadoDAO;
import com.techdam.model.Empleado;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.List;
import java.util.Optional;

public class TestCRUD {
    
    public static void main(String[] args) {
        EmpleadoDAO dao = new EmpleadoDAO();
        
        System.out.println("\n========================================");
        System.out.println("🧪 PRUEBA DE CRUD - EMPLEADOS");
        System.out.println("========================================\n");
        
        // CREATE
        System.out.println("\n--- 1. CREAR nuevo empleado ---");
        Empleado nuevo = new Empleado(
            "Pedro",
            "González",
            "pedro.gonzalez@techdam.com",
            new BigDecimal("40000.00"),
            LocalDate.now(),
            "Desarrollo"
        );
        
        int idNuevo = dao.crear(nuevo);
        System.out.println("Empleado creado: " + nuevo);
        
        // READ ALL
        System.out.println("\n--- 2. LEER todos los empleados ---");
        List<Empleado> todos = dao.obtenerTodos();
        todos.forEach(System.out::println);
        
        // READ BY ID
        System.out.println("\n--- 3. LEER empleado por ID ---");
        Optional<Empleado> encontrado = dao.obtenerPorId(idNuevo);
        encontrado.ifPresentOrElse(
            emp -> System.out.println("Encontrado: " + emp),
            () -> System.out.println("No encontrado")
        );
        
        // UPDATE
        System.out.println("\n--- 4. ACTUALIZAR empleado ---");
        if (encontrado.isPresent()) {
            Empleado paraActualizar = encontrado.get();
            paraActualizar.setSalario(new BigDecimal("45000.00"));
            paraActualizar.setDepartamento("Arquitectura");
            
            boolean actualizado = dao.actualizar(paraActualizar);
            System.out.println("Actualización exitosa: " + actualizado);
            
            // Verificar actualización
            Optional<Empleado> verificar = dao.obtenerPorId(idNuevo);
            verificar.ifPresent(emp -> 
                System.out.println("Salario actualizado: " + emp.getSalario())
            );
        }
        
        // READ BY DEPARTAMENTO
        System.out.println("\n--- 5. BUSCAR por departamento ---");
        List<Empleado> desarrollo = dao.obtenerPorDepartamento("Desarrollo");
        System.out.println("Empleados en Desarrollo: " + desarrollo.size());
        desarrollo.forEach(System.out::println);
        
        // DELETE
        System.out.println("\n--- 6. ELIMINAR empleado ---");
        boolean eliminado = dao.eliminar(idNuevo);
        System.out.println("Eliminación exitosa: " + eliminado);
        
        // Verificar eliminación
        Optional<Empleado> verificarEliminacion = dao.obtenerPorId(idNuevo);
        System.out.println("Empleado existe después de eliminar: " + 
                         verificarEliminacion.isPresent());
        
        System.out.println("\n✅ Pruebas completadas");
    }
}

---

## Ejercicio Guiado 2: Implementar CRUD de Proyectos

### Tu Tarea

Siguiendo el modelo de `EmpleadoDAO`, crea:

1. **Entidad**: `Proyecto.java`
    
2. **DAO**: `ProyectoDAO.java` con operaciones CRUD
    
3. **Test**: `TestCRUDProyectos.java`

### Pistas

- El proyecto tiene un campo `estado` de tipo ENUM
    
- Usa `pstmt.setString()` para el ENUM
    
- Para las fechas, maneja correctamente los NULL

## Buenas Prácticas de JDBC

### ✅ SÍ hacer

1. **Usar PreparedStatement siempre** para prevenir inyección SQL
    
2. **Cerrar recursos** en bloques finally o try-with-resources
    
3. **Manejar excepciones** apropiadamente
    
4. **Validar parámetros** antes de ejecutar consultas
    
5. **Usar constantes** para las consultas SQL
    
6. **Documentar** con JavaDoc

### ❌ NO hacer

1. **Concatenar strings** para construir SQL
    
2. **Dejar conexiones abiertas** sin cerrar
    
3. **Ignorar excepciones** con catch vacíos
    
4. **Hardcodear** valores mágicos
    
5. **Usar Statement** en lugar de PreparedStatement


## Evaluación del Bloque 3

### Checklist

- [ ] Entiendo la diferencia entre DDL y DML
    
- [ ] Sé por qué usar PreparedStatement en lugar de Statement
    
- [ ] Puedo crear tablas desde Java
    
- [ ] Implemento correctamente operaciones CREATE
    
- [ ] Implemento correctamente operaciones READ
    
- [ ] Implemento correctamente operaciones UPDATE
    
- [ ] Implemento correctamente operaciones DELETE
    
- [ ] Cierro siempre los recursos JDBC

## Resumen

En este bloque hemos:

- ✅ Creado el modelo de datos completo de TechDAM
    
- ✅ Implementado CRUD seguro con PreparedStatement
    
- ✅ Aplicado el patrón DAO
    
- ✅ Aprendido buenas prácticas de JDBC

