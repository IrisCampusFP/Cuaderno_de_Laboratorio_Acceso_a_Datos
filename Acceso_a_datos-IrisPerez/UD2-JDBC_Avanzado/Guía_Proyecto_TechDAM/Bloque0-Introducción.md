# UD2: JDBC Avanzado - Proyecto TechDAM

## Bienvenido/a al curso de JDBC Avanzado

En esta unidad didáctica aprenderás a dominar las técnicas avanzadas de acceso a datos en Java utilizando JDBC. A lo largo de **18-20 horas**, construirás paso a paso el proyecto **TechDAM**, un sistema completo de gestión de empleados y proyectos.

## Objetivos de Aprendizaje

Al finalizar esta unidad serás capaz de:

1. ✅ **Comprender la arquitectura JDBC** y los diferentes tipos de drivers
    
2. ✅ **Ejecutar sentencias DDL y DML** de forma segura usando `PreparedStatement`
    
3. ✅ **Trabajar con metadatos** de base de datos y resultados
    
4. ✅ **Invocar procedimientos almacenados** con parámetros IN, OUT e INOUT
    
5. ✅ **Gestionar transacciones** con commit, rollback y savepoints
    
6. ✅ **Implementar pools de conexiones** para mejorar el rendimiento
    
7. ✅ **Aplicar buenas prácticas** de seguridad contra inyección SQL

## Proyecto TechDAM: Sistema de Gestión

### Descripción del Proyecto

**TechDAM** es una empresa tecnológica que necesita un sistema para gestionar:

- **Empleados**: información personal, salario, departamento
    
- **Proyectos**: nombre, presupuesto, fechas
    
- **Asignaciones**: qué empleados trabajan en qué proyectos

### Funcionalidades que Desarrollaremos

graph TD
    A[TechDAM] --> B[Gestión Empleados]
    A --> C[Gestión Proyectos]
    A --> D[Asignaciones]
    A --> E[Transacciones]
    A --> F[Informes]
    
    B --> B1[Crear]
    B --> B2[Leer]
    B --> B3[Actualizar]
    B --> B4[Eliminar]
    
    C --> C1[CRUD Proyectos]
    C --> C2[Presupuestos]
    
    D --> D1[Asignar Empleados]
    D --> D2[Consultar Asignaciones]
    
    E --> E1[Transferencias]
    E --> E2[Rollback]
    E --> E3[Savepoints]
    
    F --> F1[Metadatos]
    F --> F2[Estadísticas]

## Estructura del Curso

|Bloque|Tema|Duración|Proyecto|
|---|---|---|---|
|1|Arquitectura JDBC y Drivers|2h|Configuración inicial|
|2|JDBC Básico vs Avanzado|1h|Comparativa de código|
|3|Sentencias DDL y DML|3h|CRUD Empleados|
|4|Consultas Avanzadas|3h|Informes con metadatos|
|5|Procedimientos Almacenados|3h|Lógica de negocio en BD|
|6|Gestión de Transacciones|3h|Operaciones atómicas|
|7|Pool de Conexiones|2h|Optimización HikariCP|
|8|Ejemplo Integrado|2h|Sistema completo|
|9|Actividad Global|2h|Entrega final|

## Tecnologías y Herramientas

### Software Necesario

- **Java 17+** (LTS)
    
- **MySQL 8.0+** o MariaDB
    
- **Maven 3.8+** para gestión de dependencias
    
- **IDE**: IntelliJ IDEA, Eclipse o VS Code

### Dependencias Maven

<dependencies>
    <!-- JDBC Driver MySQL -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>
    
    <!-- HikariCP Pool de Conexiones -->
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
        <version>5.0.1</version>
    </dependency>
</dependencies>

## Modelo de Datos TechDAM

### Diagrama Entidad-Relación

┌─────────────────┐         ┌─────────────────────┐         ┌─────────────────┐
│   EMPLEADOS     │         │    ASIGNACIONES     │         │    PROYECTOS    │
├─────────────────┤         ├─────────────────────┤         ├─────────────────┤
│ id (PK)         │──1───N──│ empleado_id (FK)    │──N───1──│ id (PK)         │
│ nombre          │         │ proyecto_id (FK)    │         │ nombre          │
│ apellido        │         │ fecha_asignacion    │         │ descripcion     │
│ email           │         │ horas_asignadas     │         │ presupuesto     │
│ salario         │         └─────────────────────┘         │ fecha_inicio    │
│ fecha_contrato  │                                         │ fecha_fin       │
│ departamento    │                                         └─────────────────┘
└─────────────────┘

## Metodología de Aprendizaje

### Enfoque Progresivo

Cada bloque sigue esta estructura:

1. **Teoría**: Conceptos fundamentales con ejemplos
    
2. **Práctica Guiada**: Código paso a paso
    
3. **Construcción del Proyecto**: Aplicación inmediata
    
4. **Ejercicios de Refuerzo**: Para consolidar
    
5. **Mini-Evaluación**: Autoevaluación

### Código Reutilizable

Todo el código que escribas será:

- **Modular**: Clases y métodos reutilizables
    
- **Seguro**: Protegido contra inyección SQL
    
- **Documentado**: Con JavaDoc y comentarios
    
- **Probado**: Con casos de prueba

## Evaluación

### Criterios de Evaluación (CE)

|CE|Descripción|Peso|
|---|---|---|
|CE2.1|Establecer conexiones con BD|2/12|
|CE2.2|Uso de sentencias DDL, DML|2/12|
|CE2.3|Procedimientos y funciones|2/12|
|CE2.4|Control de transacciones|2/12|
|CE2.5|Seguridad (PreparedStatement)|2/12|
|CE2.6|Documentación de pruebas|2/12|

### Entregables Finales

1. **Código fuente** del proyecto TechDAM
    
2. **Script SQL** con estructura y datos
    
3. **Documentación PDF** con:
    
    - Capturas de pantalla de ejecución
        
    - Explicación de decisiones técnicas
        
    - Resultados de pruebas

## ¡Empecemos!

En el siguiente capítulo comenzaremos con la **arquitectura JDBC** y configuraremos nuestro primer driver de conexión.

### Consejo del Profesor

> «No te preocupes si al principio algunas cosas parecen complejas. Vamos a ir paso a paso, construyendo sobre lo que ya sabes. Al final de esta unidad, habrás creado un sistema completo de gestión de bases de datos usando las técnicas más profesionales. ¡Confía en el proceso!»

## Recursos Adicionales

- [Documentación oficial JDBC](https://docs.oracle.com/javase/tutorial/jdbc/)
    
- [MySQL Documentation](https://dev.mysql.com/doc/)
    
- [HikariCP GitHub](https://github.com/brettwooldridge/HikariCP)
    
- [Best Practices JDBC](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html)