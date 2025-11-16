## Introducción

El uso de conexiones directas con `DriverManager` es válido para aplicaciones pequeñas, pero en entornos empresariales con muchos usuarios resulta ineficiente. Cada vez que una aplicación abre una conexión:

1. El SGBD autentica al usuario.
    
2. Se reserva memoria y recursos de red.
    
3. Se negocia el protocolo de comunicación.

Este proceso es **costoso en tiempo y recursos**. Si cada usuario abre y cierra conexiones constantemente, el sistema puede colapsar.

La solución es usar un **pool de conexiones**: un conjunto de conexiones abiertas y disponibles para reutilizar.

## 1. ¿Qué es un Pool de Conexiones?

Un _pool de conexiones_ es un **mecanismo de gestión de recursos** que mantiene un grupo de conexiones abiertas hacia la base de datos.

- Cuando un cliente necesita una conexión, la solicita al pool.
    
- Si hay una disponible, se entrega inmediatamente.
    
- Cuando termina, la conexión se devuelve al pool en lugar de cerrarse.
    
- Si no hay conexiones libres, el cliente espera o se crea una nueva hasta un máximo configurado.

## 2. Ventajas del Pool de Conexiones

1. **Rendimiento**: evita el coste de abrir/cerrar conexiones constantemente.
    
2. **Escalabilidad**: soporta más usuarios concurrentes.
    
3. **Control**: permite limitar el número máximo de conexiones para no saturar el SGBD.
    
4. **Configuración avanzada**: tiempo máximo de espera, conexiones mínimas y máximas, validación automática, etc.

## 3. Implementaciones de Pools

Las más utilizadas en proyectos Java:

- **HikariCP** → Rápido, ligero y el estándar actual en Spring Boot.
    
- **Apache DBCP** → Usado en proyectos legacy, robusto.
    
- **C3P0** → Flexible, aunque más lento.

## 4. Ejemplo práctico con HikariCP

### 4.1. Dependencia Maven

```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.1.0</version>
</dependency>
```

### 4.2. Configuración en Java

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import java.sql.Connection;

public class EjemploPool {
    public static void main(String[] args) {
        // Configuración del pool
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/empresa");
        config.setUsername("root");
        config.setPassword("root");
        config.setMaximumPoolSize(5); // máximo de conexiones simultáneas
        config.setMinimumIdle(2);     // conexiones mínimas en espera
        config.setIdleTimeout(30000); // tiempo de inactividad antes de liberar
        config.setMaxLifetime(1800000); // vida máxima de una conexión

        // Creación del DataSource (pool)
        HikariDataSource dataSource = new HikariDataSource(config);

        // Uso de la conexión
        try (Connection con = dataSource.getConnection()) {
            System.out.println("Conexión obtenida del pool");
        } catch (Exception e) {
            e.printStackTrace();
        }

        // Cerrar el pool al terminar la aplicación
        dataSource.close();
    }
}

```
## 5. Comparación: Conexiones directas vs Pool

|Característica|Conexión directa (`DriverManager`)|Pool de conexiones|
|---|---|---|
|Tiempo de apertura|Alto (cada vez crea una conexión)|Bajo (reutiliza conexiones)|
|Recursos|Alto consumo|Optimizado|
|Escalabilidad|Limitada|Alta|
|Control|Ninguno|Configurable (mínimas, máximas, timeout)|
|Uso recomendado|Aplicaciones pequeñas|Aplicaciones empresariales|

## 6. Errores comunes

- No devolver la conexión al pool (equivale a fuga de memoria).
    
- Configurar un número demasiado bajo de conexiones → bloqueos.
    
- Configurar un número demasiado alto → saturación de la BD.

## 7. Buenas prácticas

- Siempre usar `try-with-resources` para devolver conexiones al pool.
    
- Ajustar el tamaño del pool según la carga de la aplicación.
    
- Monitorizar métricas del pool (número de conexiones activas, en espera, etc.).

## 8. Mini-ejercicio

**Pregunta:**

- ¿Qué ocurre si una aplicación tiene configurado un pool con máximo 5 conexiones y 10 usuarios intentan conectarse al mismo tiempo?

**Respuesta esperada (alumno):**

- 5 usuarios obtienen conexión inmediatamente.
    
- Los otros 5 quedan en espera hasta que una conexión se libere o el sistema cree nuevas (si se permite).

## 9. Resumen

- El pool de conexiones es fundamental en aplicaciones con muchos usuarios.
    
- Aumenta el rendimiento y la escalabilidad.
    
- HikariCP es la implementación más recomendada actualmente.
    
- Siempre se deben configurar adecuadamente los parámetros del pool.

## 10. Práctica

### Actividad 1 práctica – Tema 2: Pool de Conexiones

#### Versión para alumnos

##### Objetivo

Practicar la configuración y el uso de un **pool de conexiones** en Java utilizando la librería **HikariCP**.

##### Contexto

Dispones de la base de datos **empresa** con la tabla `empleados` ya creada en la actividad anterior:

```MySQL
CREATE DATABASE empresa;

USE empresa;

CREATE TABLE empleados (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    salario DECIMAL(10,2)
);

INSERT INTO empleados (nombre, salario) VALUES 
('Ana', 1500.00),
('Luis', 1800.00),
('Marta', 2000.00),
('Pedro', 1750.00);
```

##### Tareas

1. **Configurar HikariCP en tu proyecto**
    
    - Si usas Maven, añade la dependencia en `pom.xml`:
    
```xml
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
        <version>5.1.0</version>
    </dependency>
```
    
    - Si usas un proyecto sin Maven, descarga el `.jar` de HikariCP y añádelo al classpath.
    
1. **Crear clase de configuración del pool**
    
    - Crea una clase llamada `ConexionPool` que:
        
        - Configure un pool de conexiones con:
            
            - URL: `jdbc:mysql://localhost:3306/empresa`
                
            - Usuario: `root`
                
            - Contraseña: `root`
                
            - Máximo de conexiones: `5`
                
            - Conexiones mínimas en espera: `2`
                
            - Tiempo de espera de inactividad: `30000` ms
                
        - Devuelva un objeto `Connection` al ser invocado.
    
2. **Crear clase de prueba**
    
    - Crea una clase `PruebaPool` que:
        
        - Obtenga **3 conexiones distintas** del pool.
            
        - Para cada conexión, ejecute una consulta que liste los nombres de los empleados.
            
        - Devuelva la conexión al pool automáticamente con `try-with-resources`.
    
3. **Entregables**

- Código fuente en Java (`ConexionPool.java` y `PruebaPool.java`).
    
- Capturas de pantalla mostrando la ejecución.
    
- Breve explicación (5-6 líneas) sobre por qué el uso del pool es más eficiente que abrir conexiones con `DriverManager`.

> [!Actividad 2.2]
> https://github.com/IrisCampusFP/ActividadesAccesoADatos/tree/main/UD2-JDBC_Avanzado/Act2.2