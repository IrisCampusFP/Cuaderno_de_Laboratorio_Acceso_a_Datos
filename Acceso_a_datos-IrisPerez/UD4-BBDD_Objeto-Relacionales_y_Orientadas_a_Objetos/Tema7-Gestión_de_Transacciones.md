## CE 4g: Gestionar las transacciones

En este tema aprenderemos a gestionar transacciones en bases de datos objeto-relacionales con Oracle, PostgreSQL y JPA.

## 7.1 Propiedades ACID

Las transacciones garantizan las propiedades ACID:

|Propiedad|Descripción|
|---|---|
|**Atomicidad**|Todo o nada. Si falla una operación, se deshacen todas.|
|**Consistencia**|La BD pasa de un estado válido a otro estado válido.|
|**Aislamiento**|Las transacciones concurrentes no interfieren entre sí.|
|**Durabilidad**|Los cambios confirmados son permanentes.|

## 7.2 Transacciones en Oracle

### Transacción Básica

-- Oracle inicia transacción implícitamente con la primera sentencia DML

-- Operaciones de la transacción
INSERT INTO productos VALUES (
    tipo_producto(100, 'Nuevo Producto', 50.00, 10, SYSDATE)
);

UPDATE productos p 
SET p.stock = p.stock - 1 
WHERE p.codigo = 100;

-- Confirmar transacción
COMMIT;

-- O deshacer transacción
ROLLBACK;

### Savepoints en Oracle

-- Los savepoints permiten rollback parcial

INSERT INTO productos VALUES (
    tipo_producto(101, 'Producto A', 100.00, 20, SYSDATE)
);

SAVEPOINT despues_insert_a;

INSERT INTO productos VALUES (
    tipo_producto(102, 'Producto B', 200.00, 15, SYSDATE)
);

SAVEPOINT despues_insert_b;

UPDATE productos p SET p.precio = p.precio * 2 WHERE p.codigo = 101;

-- Ups, queremos deshacer solo el UPDATE
ROLLBACK TO despues_insert_b;

-- Confirmar el resto
COMMIT;

### Bloque PL/SQL con Manejo de Excepciones

DECLARE
    v_error BOOLEAN := FALSE;
BEGIN
    -- Insertar producto
    INSERT INTO productos VALUES (
        tipo_producto(200, 'Producto Test', 99.99, 5, SYSDATE)
    );
    
    -- Actualizar stock
    UPDATE productos p 
    SET p.stock = p.stock - 1 
    WHERE p.codigo = 200;
    
    -- Verificar regla de negocio
    IF SQL%ROWCOUNT = 0 THEN
        v_error := TRUE;
    END IF;
    
    IF v_error THEN
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('Transacción cancelada');
    ELSE
        COMMIT;
        DBMS_OUTPUT.PUT_LINE('Transacción completada');
    END IF;
    
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END;
/

## 7.3 Transacciones en PostgreSQL

### Transacción Básica

-- Iniciar transacción explícitamente
BEGIN;

-- Operaciones
INSERT INTO productos (datos) VALUES (
    ROW(100, 'Nuevo Producto', 50.00, 10, CURRENT_DATE)::tipo_producto
);

UPDATE productos 
SET datos.stock = (datos).stock - 1 
WHERE (datos).codigo = 100;

-- Confirmar
COMMIT;

-- O deshacer
ROLLBACK;

### Savepoints en PostgreSQL

BEGIN;

INSERT INTO productos (datos) VALUES (
    ROW(101, 'Producto A', 100.00, 20, CURRENT_DATE)::tipo_producto
);

SAVEPOINT despues_insert_a;

INSERT INTO productos (datos) VALUES (
    ROW(102, 'Producto B', 200.00, 15, CURRENT_DATE)::tipo_producto
);

-- Deshacer solo hasta el savepoint
ROLLBACK TO SAVEPOINT despues_insert_a;

-- Confirmar (solo Producto A se guarda)
COMMIT;

### Transacción con Manejo de Excepciones

DO $$
DECLARE
    v_resultado INTEGER;
BEGIN
    -- Insertar producto
    INSERT INTO productos (datos) VALUES (
        ROW(200, 'Producto Test', 99.99, 5, CURRENT_DATE)::tipo_producto
    );
    
    -- Actualizar y verificar
    UPDATE productos 
    SET datos.stock = (datos).stock - 1 
    WHERE (datos).codigo = 200;
    
    GET DIAGNOSTICS v_resultado = ROW_COUNT;
    
    IF v_resultado = 0 THEN
        RAISE EXCEPTION 'No se actualizó ningún producto';
    END IF;
    
EXCEPTION
    WHEN OTHERS THEN
        RAISE NOTICE 'Error: %', SQLERRM;
        -- El bloque DO hace rollback automático en caso de excepción
END;
$$;

## 7.4 Transacciones con JPA

### API EntityTransaction

EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();

try {
    tx.begin();
    
    // Operaciones de negocio
    Producto p = new Producto("Nuevo", new BigDecimal("99.99"), 10);
    em.persist(p);
    
    Cliente c = em.find(Cliente.class, 1L);
    c.getDireccion().setCiudad("Barcelona");
    
    tx.commit();
    
} catch (Exception e) {
    if (tx.isActive()) {
        tx.rollback();
    }
    throw e;
} finally {
    em.close();
}

### Patrón Recomendado

public class TransaccionHelper {
    
    public static <T> T ejecutarEnTransaccion(
            EntityManager em, 
            java.util.function.Supplier<T> operacion) {
        
        EntityTransaction tx = em.getTransaction();
        try {
            tx.begin();
            T resultado = operacion.get();
            tx.commit();
            return resultado;
        } catch (Exception e) {
            if (tx.isActive()) {
                tx.rollback();
            }
            throw new RuntimeException("Error en transacción", e);
        }
    }
}

// Uso
Producto resultado = TransaccionHelper.ejecutarEnTransaccion(em, () -> {
    Producto p = new Producto("Test", new BigDecimal("50"), 5);
    em.persist(p);
    return p;
});

## 7.5 Niveles de Aislamiento

|Nivel|Dirty Read|Non-Repeatable Read|Phantom Read|
|---|---|---|---|
|READ UNCOMMITTED|Sí|Sí|Sí|
|READ COMMITTED|No|Sí|Sí|
|REPEATABLE READ|No|No|Sí|
|SERIALIZABLE|No|No|No|

### Configuración en Oracle

-- Oracle soporta READ COMMITTED (default) y SERIALIZABLE
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- O a nivel de sesión
ALTER SESSION SET ISOLATION_LEVEL = SERIALIZABLE;

### Configuración en PostgreSQL

-- PostgreSQL soporta todos los niveles
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Operaciones...

COMMIT;

### Configuración en JPA (vía JDBC)

Connection conn = // obtener conexión JDBC
conn.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);

## 7.6 Bloqueo Optimista

El bloqueo optimista asume que los conflictos son raros y los detecta al hacer commit:

### JPA con `@Version`

@Entity
public class Cuenta {
    @Id
    private Long id;
    
    @Version
    private Long version;  // JPA gestiona este campo automáticamente
    
    private BigDecimal saldo;
    
    // getters y setters
}

// Si dos transacciones modifican la misma entidad:
// - La primera hace commit exitosamente
// - La segunda recibe OptimisticLockException

### Manejo de OptimisticLockException

try {
    tx.begin();
    
    Cuenta cuenta = em.find(Cuenta.class, 1L);
    cuenta.setSaldo(cuenta.getSaldo().add(new BigDecimal("100")));
    
    tx.commit();
    
} catch (OptimisticLockException e) {
    if (tx.isActive()) tx.rollback();
    
    // Estrategias de recuperación:
    // 1. Reintentar la operación
    // 2. Informar al usuario del conflicto
    // 3. Fusionar los cambios manualmente
    
    throw new RuntimeException("Conflicto de concurrencia", e);
}

## 7.7 Bloqueo Pesimista

El bloqueo pesimista adquiere bloqueos inmediatamente para prevenir conflictos:

### SQL (Oracle y PostgreSQL)

-- Bloqueo exclusivo hasta fin de transacción
SELECT * FROM productos p WHERE p.codigo = 1 FOR UPDATE;

-- Con timeout (Oracle)
SELECT * FROM productos p WHERE p.codigo = 1 FOR UPDATE WAIT 5;

-- Sin espera (ambos)
SELECT * FROM productos p WHERE p.codigo = 1 FOR UPDATE NOWAIT;

-- Bloqueo compartido (solo lectura, permite otros lectores)
SELECT * FROM productos p WHERE p.codigo = 1 FOR SHARE;

### JPA con LockModeType

// Bloqueo pesimista al leer
Producto p = em.find(Producto.class, 1L, LockModeType.PESSIMISTIC_WRITE);

// O con query
TypedQuery<Producto> query = em.createQuery(
    "SELECT p FROM Producto p WHERE p.id = :id", Producto.class);
query.setParameter("id", 1L);
query.setLockMode(LockModeType.PESSIMISTIC_WRITE);
Producto p = query.getSingleResult();

// Bloquear entidad ya cargada
em.lock(producto, LockModeType.PESSIMISTIC_WRITE);

### Tipos de LockModeType

|LockModeType|Descripción|
|---|---|
|`NONE`|Sin bloqueo|
|`OPTIMISTIC`|Bloqueo optimista (verifica versión al commit)|
|`OPTIMISTIC_FORCE_INCREMENT`|Optimista + incrementa versión|
|`PESSIMISTIC_READ`|Bloqueo compartido (FOR SHARE)|
|`PESSIMISTIC_WRITE`|Bloqueo exclusivo (FOR UPDATE)|
|`PESSIMISTIC_FORCE_INCREMENT`|Exclusivo + incrementa versión|

## 7.8 Resumen del Tema

Conceptos Clave

1. Las transacciones garantizan **ACID**: Atomicidad, Consistencia, Aislamiento, Durabilidad.
    
2. Usar **SAVEPOINT** para rollback parcial en operaciones complejas.
    
3. En JPA, siempre verificar `tx.isActive()` antes de hacer rollback.
    
4. **Bloqueo optimista** (`@Version`): Detecta conflictos al commit.
    
5. **Bloqueo pesimista** (`FOR UPDATE`): Previene conflictos bloqueando filas.
    
6. El nivel de aislamiento por defecto es **READ COMMITTED** en Oracle y PostgreSQL.