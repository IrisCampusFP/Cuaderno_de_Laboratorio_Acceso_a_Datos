## Enunciado

Debes completar el desarrollo del sistema **TechDAM** implementando todas las funcionalidades requeridas y demostrando dominio de JDBC avanzado.

## Objetivos de la Actividad

Esta actividad evalúa los siguientes **Criterios de Evaluación (CE)**:

|CE|Descripción|Peso|
|---|---|---|
|CE2.1|Establecer conexiones con BD usando controladores|2 pts|
|CE2.2|Uso de sentencias DDL, DML y de control|2 pts|
|CE2.3|Uso de procedimientos y funciones|2 pts|
|CE2.4|Control de transacciones|2 pts|
|CE2.5|Seguridad frente a inyección SQL|2 pts|
|CE2.6|Documentación de pruebas|2 pts|
|**TOTAL**||**12 pts**|

## Parte 1: Base de Datos (CE2.2)

### Requisitos

1. **Crear script SQL** `techdam_completo.sql` que incluya:
    
    - [ ] Creación de base de datos
        
    - [ ] 3 tablas: `empleados`, `proyectos`, `asignaciones`
        
    - [ ] Claves primarias y foráneas
        
    - [ ] Índices apropiados
        
    - [ ] Al menos 5 registros de prueba en cada tabla
        
    - [ ] 2 procedimientos almacenados
        
    - [ ] 1 función almacenada

### Ejemplo de Procedimiento Requerido

DELIMITER $$
CREATE PROCEDURE actualizar_salario_departamento(
    IN p_departamento VARCHAR(50),
    IN p_porcentaje DECIMAL(5,2),
    OUT p_empleados_actualizados INT
)
BEGIN
    UPDATE empleados 
    SET salario = salario * (1 + p_porcentaje / 100)
    WHERE departamento = p_departamento AND activo = TRUE;
    
    SET p_empleados_actualizados = ROW_COUNT();
END$$
DELIMITER ;

## 💻 Parte 2: Código Java (CE2.1, CE2.5)

### 2.1 Configuración de Conexión

**Archivo**: `DatabaseConfig.java` o `DatabaseConfigPool.java`

Requisitos:

- [ ] Usar HikariCP (pool de conexiones)
    
- [ ] Configuración en archivo `.properties` (no hardcoded)
    
- [ ] Método `getConnection()` que devuelve conexión del pool
    
- [ ] Método `close()` para cerrar el pool

**Puntuación**:

- DriverManager simple: 1 punto
    
- HikariCP configurado: 2 puntos

### 2.2 Modelo de Datos

**Archivos**: `Empleado.java`, `Proyecto.java`, `Asignacion.java`

Requisitos:

- [ ] Getters y setters para todos los campos
    
- [ ] Constructor vacío y constructor con parámetros
    
- [ ] Método `toString()` sobrescrito
    
- [ ] Uso de tipos apropiados (BigDecimal para dinero, LocalDate para fechas)

### 2.3 Capa DAO (CE2.2, CE2.5)

**Archivos**: `EmpleadoDAO.java`, `ProyectoDAO.java`

Requisitos obligatorios:

- [ ] **CRUD completo** para Empleados y Proyectos:
    
    - `crear(T objeto)` → devuelve ID generado
        
    - `obtenerTodos()` → devuelve `List<T>`
        
    - `obtenerPorId(int id)` → devuelve `Optional<T>`
        
    - `actualizar(T objeto)` → devuelve `boolean`
        
    - `eliminar(int id)` → devuelve `boolean`
        
- [ ] **PreparedStatement** en TODAS las consultas (nunca Statement)
    
- [ ] Try-with-resources o finally para cerrar recursos
    
- [ ] Manejo apropiado de excepciones SQLException

**Puntuación CE2.5 (Seguridad)**:

- Statement con concatenación: 0 puntos ❌
    
- PreparedStatement en 50% consultas: 1 punto
    
- PreparedStatement en 100% consultas: 2 puntos ✅

## Parte 3: Procedimientos Almacenados (CE2.3)

**Archivo**: `ProcedimientosService.java`

Requisitos:

- [ ] Implementar al menos 2 métodos que invoquen procedimientos almacenados
    
- [ ] Usar `CallableStatement`
    
- [ ] Manejar parámetros IN y OUT correctamente
    
- [ ] Mostrar resultados de los parámetros OUT

### Ejemplo Requerido

public int actualizarSalariosDepartamento(String departamento, double porcentaje) {
    try (Connection conn = DatabaseConfig.getConnection();
         CallableStatement cstmt = conn.prepareCall(
             "{call actualizar_salario_departamento(?, ?, ?)}")) {
        
        cstmt.setString(1, departamento);
        cstmt.setBigDecimal(2, BigDecimal.valueOf(porcentaje));
        cstmt.registerOutParameter(3, Types.INTEGER);
        
        cstmt.execute();
        
        return cstmt.getInt(3);
        
    } catch (SQLException e) {
        e.printStackTrace();
        return -1;
    }
}

**Puntuación**:

- 0-1 procedimientos: 1 punto
    
- 2+ procedimientos funcionando: 2 puntos

## Parte 4: Transacciones (CE2.4)

**Archivo**: `TransaccionesService.java`

Debes implementar **AL MENOS DOS** de estas operaciones transaccionales:

### Opción A: Transferencia de Presupuesto

public boolean transferirPresupuesto(int proyectoOrigenId, 
                                    int proyectoDestinoId, 
                                    BigDecimal monto) {
    Connection conn = null;
    try {
        conn = DatabaseConfig.getConnection();
        conn.setAutoCommit(false);  // Iniciar transacción
        
        // 1. Restar de origen
        proyectoDAO.restarPresupuesto(conn, proyectoOrigenId, monto);
        
        // 2. Sumar a destino
        proyectoDAO.sumarPresupuesto(conn, proyectoDestinoId, monto);
        
        conn.commit();  // Confirmar
        return true;
        
    } catch (SQLException e) {
        if (conn != null) {
            try {
                conn.rollback();  // Revertir
            } catch (SQLException ex) {}
        }
        return false;
    } finally {
        DatabaseConfig.closeConnection(conn);
    }
}

### Opción B: Asignación Múltiple con Savepoint

public void asignarEmpleadosConSavepoint(int proyectoId, List<Integer> empleadoIds) {
    Connection conn = null;
    try {
        conn = DatabaseConfig.getConnection();
        conn.setAutoCommit(false);
        
        for (int empId : empleadoIds) {
            Savepoint sp = conn.setSavepoint("SP_" + empId);
            try {
                asignacionDAO.crear(conn, empId, proyectoId);
            } catch (SQLException e) {
                conn.rollback(sp);  // Rollback parcial
            }
        }
        
        conn.commit();
    } catch (SQLException e) {
        // rollback...
    }
}

**Puntuación**:

- Solo commit, sin rollback: 0.5 puntos
    
- Commit + rollback básico: 1 punto
    
- Commit + rollback + savepoints: 2 puntos

## Parte 5: Documentación (CE2.6)

**Archivo**: `DOCUMENTACION.pdf` o `README.md`

Debe incluir:

### 1. Portada

- Nombre del alumno
    
- Fecha
    
- Título: «Proyecto TechDAM - JDBC Avanzado»

### 2. Diagrama de Base de Datos

- Diagrama ER con las 3 tablas
    
- Indicar claves primarias y foráneas

### 3. Capturas de Pantalla

Incluir capturas de:

- [ ] Tablas creadas en MySQL Workbench / HeidiSQL
    
- [ ] Ejecución exitosa de CRUD de empleados
    
- [ ] Ejecución exitosa de CRUD de proyectos
    
- [ ] Invocación de procedimiento almacenado
    
- [ ] Transacción con commit
    
- [ ] Transacción con rollback
    
- [ ] Pool de conexiones configurado (logs o métricas)

### 4. Explicación de Decisiones Técnicas

Responde:

1. **¿Por qué usar PreparedStatement en lugar de Statement?**
    
2. **¿Qué ventajas tiene el pool de conexiones?**
    
3. **¿En qué casos usarías procedimientos almacenados?**
    
4. **¿Por qué es importante el control de transacciones?**

### 5. Problemas Encontrados y Soluciones

Describe al menos 2 problemas que encontraste y cómo los solucionaste.

**Puntuación**:

- Sin documentación: 0 puntos
    
- Documentación básica (capturas): 1 punto
    
- Documentación completa (capturas + explicaciones): 2 puntos

## Entregables

Debes entregar un archivo ZIP con:

apellido_nombre_techdam.zip
├── README.txt              # Instrucciones de ejecución
├── techdam_completo.sql    # Script SQL completo
├── pom.xml                 # Dependencias Maven
├── src/
│   └── main/java/com/techdam/
│       ├── Main.java
│       ├── config/
│       ├── model/
│       ├── dao/
│       └── service/
└── DOCUMENTACION.pdf       # Documentación con capturas

## Rúbrica de Evaluación

### Resumen de Puntos

|Criterio|Puntos|Descripción|
|---|---|---|
|**CE2.1** Conexiones|2|HikariCP configurado|
|**CE2.2** DDL/DML|2|CRUD completo|
|**CE2.3** Procedimientos|2|2+ procedimientos funcionando|
|**CE2.4** Transacciones|2|Commit + rollback + savepoints|
|**CE2.5** Seguridad|2|PreparedStatement en 100%|
|**CE2.6** Documentación|2|Completa con capturas|
|**TOTAL**|**12**|Normalizado a 10|

### Escala de Calificación

12 puntos = 10 (Sobresaliente)
10-11     = 8-9 (Notable)
8-9       = 6-7 (Bien)
6-7       = 5 (Suficiente)
<6        = <5 (Insuficiente)

## ⏰ Plazo de Entrega

- **Fecha límite**: [A definir por el profesor]
    
- **Plataforma**: Aula Virtual / Moodle
    
- **Formato**: Archivo ZIP nombrado como `apellido_nombre_techdam.zip`

## ❓ FAQs

### ¿Puedo usar JPA o Hibernate?

No. Esta actividad evalúa JDBC puro. JPA lo veremos más adelante.

### ¿Puedo trabajar en parejas?

Consulta con el profesor. Por defecto es individual.

### ¿Qué pasa si no funciona algo?

En la documentación explica qué intentaste, qué error obtuviste y cómo lo solucionaste (o por qué no pudiste).

### ¿Debo crear tests unitarios?

No es obligatorio, pero suma puntos extra si los incluyes.

### ¿Puedo añadir funcionalidades extra?

Sí, puedes añadir un menú CLI, más procedimientos, más tablas, etc. Se valorará positivamente.

## Consejos Finales

1. ✅ **Empieza por la base de datos** - Si la BD no funciona, nada funciona
    
2. ✅ **Prueba cada parte por separado** - No intentes hacer todo de golpe
    
3. ✅ **Haz commits frecuentes** - Usa Git para versionar tu código
    
4. ✅ **Documenta mientras desarrollas** - No dejes la documentación para el final
    
5. ✅ **Haz capturas de TODO** - Es mejor tener muchas que pocas
    
6. ✅ **Revisa la rúbrica** - Asegúrate de cumplir cada punto

## Recursos de Ayuda

- Notebooks del curso (bloques 1-8)
    
- Código de ejemplo en GitHub
    
- Documentación oficial de JDBC
    
- Consultas al profesor en horario de tutoría

## ✅ Checklist Final

Antes de entregar, verifica:

- [ ] El script SQL se ejecuta sin errores
    
- [ ] El código Java compila sin errores
    
- [ ] Todas las operaciones CRUD funcionan
    
- [ ] Los procedimientos se ejecutan correctamente
    
- [ ] Las transacciones hacen commit/rollback
    
- [ ] No hay ni un solo Statement, solo PreparedStatement
    
- [ ] Tienes capturas de pantalla de todo
    
- [ ] La documentación está completa
    
- [ ] El nombre del archivo es correcto
    
- [ ] El README explica cómo ejecutar el proyecto

**¡Mucha suerte con tu proyecto TechDAM! 🚀**