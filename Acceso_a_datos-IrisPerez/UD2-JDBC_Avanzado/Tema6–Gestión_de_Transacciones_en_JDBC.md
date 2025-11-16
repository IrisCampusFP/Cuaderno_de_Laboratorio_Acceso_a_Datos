## Introducción

En los sistemas de bases de datos, una **transacción** es una unidad de trabajo que agrupa una o varias operaciones SQL. Su propósito es garantizar que los datos se mantengan **consistentes y fiables**, incluso en situaciones de fallo del sistema, errores de red o interrupciones inesperadas.

En JDBC, las transacciones se controlan principalmente a través del objeto **`Connection`**, que ofrece métodos para confirmar o deshacer cambios.

## 1. Propiedades ACID de las transacciones

Una transacción debe cumplir con las propiedades **ACID**:

1. **Atomicidad**: la transacción se ejecuta completamente o no se ejecuta en absoluto.
    
2. **Consistencia**: los datos pasan de un estado válido a otro válido.
    
3. **Aislamiento**: el resultado de una transacción no debe verse afectado por otras concurrentes.
    
4. **Durabilidad**: una vez confirmados, los cambios son permanentes.

## 2. Autocommit en JDBC

- Por defecto, cada instrucción SQL en JDBC se ejecuta como una transacción independiente (**modo autocommit**).
    
- Esto significa que al ejecutar un `INSERT` o un `UPDATE`, el cambio se confirma automáticamente en la BD.

### Ejemplo

```java
Connection con = DriverManager.getConnection(url, user, pass);
// Autocommit activado por defecto
System.out.println("Autocommit: " + con.getAutoCommit()); // true
```

## 3. Control manual de transacciones

Podemos desactivar el autocommit y controlar las transacciones manualmente:

```java
con.setAutoCommit(false); // desactiva autocommit

try {
    // varias operaciones SQL
    con.commit(); // confirmar todos los cambios
} catch (SQLException e) {
    con.rollback(); // deshacer si hay error
}
```

## 4. Ejemplo de transacción: transferencia bancaria

Supongamos dos cuentas en la tabla `cuentas` y queremos transferir 500€.

```java
try (Connection con = DriverManager.getConnection(url, user, pass)) {
    con.setAutoCommit(false); // iniciar transacción manual

    // Restar saldo a la cuenta origen
    PreparedStatement ps1 = con.prepareStatement(
        "UPDATE cuentas SET saldo = saldo - ? WHERE id = ?");
    ps1.setDouble(1, 500);
    ps1.setInt(2, 1);
    ps1.executeUpdate();

    // Sumar saldo a la cuenta destino
    PreparedStatement ps2 = con.prepareStatement(
        "UPDATE cuentas SET saldo = saldo + ? WHERE id = ?");
    ps2.setDouble(1, 500);
    ps2.setInt(2, 2);
    ps2.executeUpdate();

    con.commit(); // confirmar cambios
    System.out.println("Transferencia realizada con éxito");

} catch (SQLException e) {
    con.rollback(); // revertir cambios
    System.out.println("Error en la transacción. Se ha realizado rollback.");
}

```
## 5. Uso de Savepoints

Un **savepoint** es un punto intermedio en la transacción al que podemos volver si ocurre un error parcial.

### Ejemplo

```java
con.setAutoCommit(false);

Savepoint sp1 = null;

try {
    Statement stmt = con.createStatement();

    // Primera operación
    stmt.executeUpdate("INSERT INTO logs (mensaje) VALUES ('Paso 1 completado')");
    sp1 = con.setSavepoint("Punto1");

    // Segunda operación (puede fallar)
    stmt.executeUpdate("INSERT INTO logs (mensaje) VALUES ('Paso 2 completado')");

    con.commit();
} catch (SQLException e) {
    if (sp1 != null) {
        con.rollback(sp1); // volver al savepoint
        con.commit();
        System.out.println("Error en Paso 2, pero Paso 1 se mantuvo");
    } else {
        con.rollback();
    }
}
```

## 6. Niveles de aislamiento

Los SGBD permiten configurar el **nivel de aislamiento** de las transacciones. JDBC ofrece cuatro niveles:

|Nivel|Efecto|Problemas evitados|
|---|---|---|
|`TRANSACTION_READ_UNCOMMITTED`|Permite leer datos no confirmados|Ninguno|
|`TRANSACTION_READ_COMMITTED`|Solo permite leer datos confirmados|Lecturas sucias|
|`TRANSACTION_REPEATABLE_READ`|Bloquea filas leídas hasta que finaliza la transacción|Lecturas no repetibles|
|`TRANSACTION_SERIALIZABLE`|Máximo aislamiento, simula ejecución secuencial|Fantasmas, lecturas inconsistentes|

Se establece con:

```MySQL
con.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);
```

## 7. Errores comunes

- Olvidar cerrar la transacción con `commit()` o `rollback()`.
    
- Dejar activado el autocommit al querer controlar varias operaciones.
    
- Usar un nivel de aislamiento demasiado alto que degrade el rendimiento.

## 8. Buenas prácticas

- Desactivar autocommit solo cuando se realicen varias operaciones dependientes.
    
- Usar `commit()` tan pronto como la transacción sea consistente.
    
- Usar savepoints en operaciones críticas.
    
- Ajustar el nivel de aislamiento según la necesidad de la aplicación.

## 9. Actividad práctica

### Versión para alumnos

#### Objetivo

Aprender a manejar transacciones en JDBC, incluyendo `commit`, `rollback` y `savepoint`.

#### Tareas

1. En la base de datos `empresa`, crea la tabla `cuentas`:

```MySQL
CREATE TABLE cuentas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    titular VARCHAR(100),
    saldo DECIMAL(10,2)
);

INSERT INTO cuentas (titular, saldo) VALUES
('Ana', 2000.00),
('Luis', 1500.00);

```

2. Implementa en Java una clase `TransferenciaBancaria` que:
    
    - Realice una transferencia de 500€ de la cuenta 1 a la cuenta 2.
        
    - Use transacciones manuales (`setAutoCommit(false)`).
        
    - Aplique `commit()` si la operación se completa correctamente.
        
    - Aplique `rollback()` si ocurre un error.

3. Extiende el ejercicio para que:
    
    - Se registre en una tabla `logs` cada paso de la transacción.
        
    - Si falla el registro del segundo paso, se use un **savepoint** para mantener el primero.

#### Entregables

- Script SQL con las tablas.
    
- Código Java de la transferencia.
    
- Capturas mostrando:
    
    - Estado de las cuentas antes y después de la operación.
        
    - Estado de los logs tras un rollback parcial.

> [!Actividad 2.6]
> https://github.com/IrisCampusFP/ActividadesAccesoADatos/tree/main/UD2-JDBC_Avanzado/Act2.6