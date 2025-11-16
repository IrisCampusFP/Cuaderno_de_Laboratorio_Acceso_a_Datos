## Objetivos

- Entender el problema de crear conexiones repetidamente
    
- Configurar HikariCP para pool de conexiones
    
- Optimizar rendimiento en TechDAM
    
- Monitorizar métricas del pool

## Teoría: ¿Por qué un Pool?

### Problema: Crear Conexión es Caro

// Cada vez que haces esto...
Connection conn = DriverManager.getConnection(url, user, pass);

// ... se ejecutan estos pasos (LENTOS):
// 1. Abrir socket TCP
// 2. Autenticar usuario
// 3. Configurar sesión
// 4. Reservar recursos
// Tiempo: ~50-200ms

### Solución: Pool de Conexiones

┌─────────────────────┐
│   POOL (HikariCP)   │
├─────────────────────┤
│ [Conn1] ✅ Libre    │ ← Reutiliza conexiones
│ [Conn2] 🔒 En uso   │
│ [Conn3] ✅ Libre    │
│ [Conn4] ✅ Libre    │
└─────────────────────┘
         ↕
    Aplicación

**Ventajas**:

- 100x más rápido
    
- Limita conexiones concurrentes
    
- Reutiliza conexiones
    
- Valida conexiones automáticamente

## 💻 Práctica 1: Configurar HikariCP

### DatabaseConfigPool.java

package com.techdam.config;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import java.sql.Connection;
import java.sql.SQLException;

public class DatabaseConfigPool {
    
    private static HikariDataSource dataSource;
    
    static {
        HikariConfig config = new HikariConfig();
        
        // Configuración básica
        config.setJdbcUrl("jdbc:mysql://localhost:3306/techdam");
        config.setUsername("root");
        config.setPassword("");
        
        // Pool settings
        config.setMaximumPoolSize(10);        // Max conexiones
        config.setMinimumIdle(5);             // Min conexiones idle
        config.setConnectionTimeout(30000);   // Timeout: 30s
        config.setIdleTimeout(600000);        // Idle: 10min
        config.setMaxLifetime(1800000);       // Lifetime: 30min
        
        // Optimizaciones
        config.setAutoCommit(false);
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        
        dataSource = new HikariDataSource(config);
        
        System.out.println("✅ Pool HikariCP inicializado");
    }
    
    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
    
    public static void close() {
        if (dataSource != null) {
            dataSource.close();
        }
    }
}

## 💻 Práctica 2: Comparativa de Rendimiento

### Test: DriverManager vs HikariCP

public class BenchmarkPool {
    
    public static void main(String[] args) {
        int numConsultas = 100;
        
        // Test 1: Sin pool
        long tiempoSinPool = testSinPool(numConsultas);
        
        // Test 2: Con HikariCP
        long tiempoConPool = testConPool(numConsultas);
        
        System.out.println("\n📊 RESULTADOS");
        System.out.println("Sin pool:  " + tiempoSinPool + " ms");
        System.out.println("Con pool:  " + tiempoConPool + " ms");
        System.out.println("Mejora:    " + (tiempoSinPool / tiempoConPool) + "x más rápido");
    }
    
    private static long testSinPool(int n) {
        long inicio = System.currentTimeMillis();
        
        for (int i = 0; i < n; i++) {
            try (Connection conn = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/techdam", "root", "");
                 Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery("SELECT 1")) {
                // Vacío
            } catch (SQLException e) {}
        }
        
        return System.currentTimeMillis() - inicio;
    }
    
    private static long testConPool(int n) {
        long inicio = System.currentTimeMillis();
        
        for (int i = 0; i < n; i++) {
            try (Connection conn = DatabaseConfigPool.getConnection();
                 Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery("SELECT 1")) {
                // Vacío
            } catch (SQLException e) {}
        }
        
        return System.currentTimeMillis() - inicio;
    }
}

**Resultado esperado**: Pool es ~50-100x más rápido

## Métricas y Monitoreo

// Obtener métricas del pool
HikariPoolMXBean poolBean = dataSource.getHikariPoolMXBean();

System.out.println("Conexiones activas: " + poolBean.getActiveConnections());
System.out.println("Conexiones idle: " + poolBean.getIdleConnections());
System.out.println("Total conexiones: " + poolBean.getTotalConnections());
System.out.println("Threads esperando: " + poolBean.getThreadsAwaitingConnection());

## ⚙️ Configuración Óptima

### Fórmula del Tamaño de Pool

conexiones = ((núcleos_cpu × 2) + discos_spindle)

Ejemplo:
  CPU: 4 núcleos
  Discos: 1 SSD
  Pool size: (4 × 2) + 1 = 9 conexiones

### Configuración Recomendada

|Parámetro|Desarrollo|Producción|Descripción|
|---|---|---|---|
|`maximumPoolSize`|10|20|Máximo de conexiones|
|`minimumIdle`|5|10|Conexiones mínimas idle|
|`connectionTimeout`|30000|20000|Timeout en ms|
|`idleTimeout`|600000|300000|Tiempo idle antes de cerrar|
|`maxLifetime`|1800000|1800000|Tiempo máximo de vida|

## 💻 Ejercicio: Refactorizar TechDAM

### Tarea

1. Cambia `DatabaseConfig` por `DatabaseConfigPool` en todos los DAOs
    
2. Ejecuta el benchmark
    
3. Monitoriza las métricas
    
4. Ajusta el tamaño del pool según la carga

## Resumen

- ✅ Pool de conexiones mejora rendimiento 50-100x
    
- ✅ HikariCP es el pool más rápido para Java
    
- ✅ Configuración depende de CPU y carga
    
- ✅ Monitorizar métricas en producción