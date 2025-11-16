## Datos generales

- **Unidad Didáctica (UD2)**: JDBC Avanzado
    
- **Duración**: 18–20 horas
    
- **Resultado de Aprendizaje (RA) principal**:
    
    - **RA2**: _“Desarrolla aplicaciones que manipulen la información almacenada en bases de datos utilizando APIs específicas.”_

## Contenidos – Bloques

|Bloque|Contenido principal|Duración estimada|Actividades|
|---|---|---|---|
|1|Arquitectura de JDBC y drivers|2h|Explicación + mini-ejercicio sobre `DatabaseMetaData`|
|2|Diferencias entre JDBC básico y avanzado|1h|Cuadro comparativo y debate en clase|
|3|Ejecución de sentencias DDL y DML con JDBC|3h|Actividad CRUD con `PreparedStatement`|
|4|Consultas avanzadas con ResultSet y Metadatos|3h|Actividad con `ResultSetMetaData` y `DatabaseMetaData`|
|5|Procedimientos almacenados (CallableStatement)|3h|Actividad de `incrementar_salario` con IN y OUT|
|6|Gestión de transacciones (commit, rollback, savepoints)|3h|Actividad de transferencia bancaria con rollback parcial|
|7|Pool de conexiones (HikariCP)|2h|Actividad de configuración de pool y consultas múltiples|
|8|Ejemplo integrado y comparativa final|2h|Caso global de empleados y proyectos|
|9|Actividad global de evaluación|2h|Proyecto final TechDAM|

## Relación RA – CE – Contenidos

|RA|CE|Descripción|Bloques relacionados|Evidencias|
|---|---|---|---|---|
|RA2|CE2.1|Establecer conexiones con BD usando controladores|1, 7|Ejemplo con DriverManager y pool HikariCP|
|RA2|CE2.2|Uso de sentencias DDL, DML y de control|3|CRUD completo sobre empleados y proyectos|
|RA2|CE2.3|Uso de procedimientos y funciones|5|Procedimiento `incrementar_salario` y función `contar_empleados`|
|RA2|CE2.4|Control de transacciones|6|Transferencia bancaria con rollback y savepoint|
|RA2|CE2.5|Seguridad frente a inyección SQL|3, 4|Uso de `PreparedStatement` en todas las consultas|
|RA2|CE2.6|Documentación de pruebas|8, 9|Capturas de pantalla y PDF explicativo en actividad global|

## Actividades diseñadas

1. **Actividades intermedias por bloque**
    
    - Ejercicios guiados de conexión, CRUD, consultas con metadatos, procedimientos y transacciones.
        
    - Incluyen mini-prácticas de refuerzo con resultados esperados.

2. **Actividad global de evaluación**
    
    - Proyecto final **TechDAM**:
        
        - Creación de BD con empleados, proyectos y asignaciones.
            
        - CRUD con `PreparedStatement`.
            
        - Procedimientos almacenados (`CallableStatement`).
            
        - Transacciones con `commit`, `rollback` y `savepoint`.
            
        - Consultas avanzadas con `ResultSetMetaData`.
            
        - Pool de conexiones (HikariCP).
            
        - Documentación en PDF con resultados.

## Evaluación

- **Instrumentos**:
    
    - Rúbrica (12 puntos normalizada a 10).
        
    - Observación del profesor en clase.
        
    - Entregables: código, script SQL, documentación PDF.

- **Criterios principales**:
    
    - Conexiones (2 ptos).
        
    - CRUD con SQL (2 ptos).
        
    - Procedimientos (2 ptos).
        
    - Transacciones (2 ptos).
        
    - Seguridad con `PreparedStatement` (2 ptos).
        
    - Documentación de pruebas (2 ptos).


> [!Actividades UD2]
> https://github.com/IrisCampusFP/ActividadesAccesoADatos/tree/main/UD2-JDBC_Avanzado