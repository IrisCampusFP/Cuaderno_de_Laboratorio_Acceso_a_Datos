## RA2:

**“Desarrolla aplicaciones que manipulen la información almacenada en bases de datos utilizando APIs específicas.”**

### CE vinculados al RA2

1. **CE2.1**: Se han establecido conexiones con la base de datos utilizando diferentes controladores y métodos de acceso.
    
    - ✔ Cubierto con: configuración de `DriverManager` y uso de **HikariCP (pool de conexiones)**.
    
2. **CE2.2**: Se han utilizado sentencias SQL de definición, manipulación y control de datos.
    
    - ✔ Cubierto con: ejecución de **DDL (crear tablas)** y **DML (INSERT, UPDATE, DELETE, SELECT)** mediante `Statement` y `PreparedStatement`.
    
3. **CE2.3**: Se han utilizado procedimientos almacenados y funciones.
    
    - ✔ Cubierto con: creación e invocación de procedimientos (`asignar_empleado`, `incrementar_salario`) y función (`contar_empleados`).
    
4. **CE2.4**: Se han realizado transacciones controlando la atomicidad de las operaciones.
    
    - ✔ Cubierto con: implementación de **transacciones con commit, rollback y savepoints**.
    
5. **CE2.5**: Se han utilizado mecanismos de seguridad para evitar inyecciones SQL.
    
    - ✔ Cubierto con: uso de **PreparedStatement** en todas las consultas parametrizadas.
    
6. **CE2.6**: Se han documentado las pruebas y resultados obtenidos.
    
    - ✔ Cubierto con: requerimiento de **capturas de ejecución y documentación en PDF** en la actividad global.

## Otros RA reforzados indirectamente

- **RA1 (opcionalmente reforzado)**: “Almacena información en sistemas persistentes analizando y utilizando mecanismos disponibles.”
    
    - Relación: se ve reflejado en la **persistencia de datos** en las tablas y la correcta definición de **DDL**.

## Tabla de relación RA–CE–Actividad

| RA              | CE    | Descripción                     | Evidencia en la actividad                                    |
| --------------- | ----- | ------------------------------- | ------------------------------------------------------------ |
| RA2             | CE2.1 | Conexión con BD                 | Uso de DriverManager y HikariCP                              |
| RA2             | CE2.2 | Uso de DDL y DML                | Creación de tablas, CRUD completo                            |
| RA2             | CE2.3 | Uso de procedimientos/funciones | Procedimiento `asignar_empleado`, función `contar_empleados` |
| RA2             | CE2.4 | Transacciones                   | Transferencias con commit, rollback, savepoint               |
| RA2             | CE2.5 | Seguridad en consultas          | PreparedStatement para evitar SQL Injection                  |
| RA2             | CE2.6 | Documentación de pruebas        | Capturas y explicación en PDF entregado                      |
| RA1 (indirecto) | CE1.x | Persistencia de información     | Creación de tablas y almacenamiento de datos                 |

> [!Actividad Global (Proyecto TechDAM)]
> https://github.com/IrisCampusFP/ActividadesAccesoADatos/tree/main/UD2-JDBC_Avanzado/perez_iris_techdam