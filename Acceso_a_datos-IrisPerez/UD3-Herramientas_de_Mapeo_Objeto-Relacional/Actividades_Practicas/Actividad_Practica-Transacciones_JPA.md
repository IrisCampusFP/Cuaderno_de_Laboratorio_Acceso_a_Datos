## Objetivos

- Gestionar transacciones con EntityManager
    
- Aplicar `@Transactional` de Spring
    
- Comprender niveles de aislamiento
    
- Manejar rollback y excepciones

## Ejercicio 1: Transaccion manual con EntityManager

### Enunciado

Implementa una transferencia bancaria entre dos cuentas que:

- Retire dinero de la cuenta origen
    
- Deposite en la cuenta destino
    
- Registre la operacion en una tabla de movimientos
    
- Todo debe ser atomico (o todo o nada)


Ver solucion

public class TransferenciaService {
    
    private EntityManagerFactory emf;
    
    public void transferir(Long cuentaOrigenId, Long cuentaDestinoId, 
                          BigDecimal monto) {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        
        try {
            tx.begin();
            
            // Cargar cuentas con bloqueo pesimista
            Cuenta origen = em.find(Cuenta.class, cuentaOrigenId, 
                                   LockModeType.PESSIMISTIC_WRITE);
            Cuenta destino = em.find(Cuenta.class, cuentaDestinoId, 
                                    LockModeType.PESSIMISTIC_WRITE);
            
            // Validar
            if (origen == null || destino == null) {
                throw new IllegalArgumentException("Cuenta no encontrada");
            }
            
            if (origen.getSaldo().compareTo(monto) < 0) {
                throw new SaldoInsuficienteException(
                    "Saldo insuficiente: " + origen.getSaldo());
            }
            
            // Realizar transferencia
            origen.setSaldo(origen.getSaldo().subtract(monto));
            destino.setSaldo(destino.getSaldo().add(monto));
            
            // Registrar movimientos
            Movimiento movOrigen = new Movimiento(
                origen, TipoMovimiento.RETIRO, monto.negate(),
                "Transferencia a cuenta " + cuentaDestinoId
            );
            Movimiento movDestino = new Movimiento(
                destino, TipoMovimiento.DEPOSITO, monto,
                "Transferencia de cuenta " + cuentaOrigenId
            );
            
            em.persist(movOrigen);
            em.persist(movDestino);
            
            tx.commit();
            System.out.println("Transferencia exitosa");
            
        } catch (Exception e) {
            if (tx.isActive()) {
                tx.rollback();
            }
            System.err.println("Error en transferencia: " + e.getMessage());
            throw e;
        } finally {
            em.close();
        }
    }
}

**Puntos clave:**

- `tx.begin()` inicia la transaccion
    
- `tx.commit()` confirma todos los cambios
    
- `tx.rollback()` deshace todo si hay error
    
- `LockModeType.PESSIMISTIC_WRITE` evita condiciones de carrera

## Ejercicio 2: `@Transactional` de Spring

### Enunciado

Reescribe el servicio anterior usando `@Transactional` de Spring. Configura:

- Rollback para `SaldoInsuficienteException`
    
- Sin rollback para `NotificacionException`
    
- Timeout de 30 segundos


Ver solucion

@Service
public class TransferenciaServiceSpring {
    
    @Autowired
    private CuentaRepository cuentaRepository;
    
    @Autowired
    private MovimientoRepository movimientoRepository;
    
    @Autowired
    private NotificacionService notificacionService;
    
    @Transactional(
        rollbackFor = SaldoInsuficienteException.class,
        noRollbackFor = NotificacionException.class,
        timeout = 30
    )
    public void transferir(Long cuentaOrigenId, Long cuentaDestinoId, 
                          BigDecimal monto) {
        
        // Cargar cuentas
        Cuenta origen = cuentaRepository.findById(cuentaOrigenId)
            .orElseThrow(() -> new EntityNotFoundException("Cuenta origen no existe"));
        Cuenta destino = cuentaRepository.findById(cuentaDestinoId)
            .orElseThrow(() -> new EntityNotFoundException("Cuenta destino no existe"));
        
        // Validar saldo
        if (origen.getSaldo().compareTo(monto) < 0) {
            throw new SaldoInsuficienteException("Saldo insuficiente");
        }
        
        // Realizar transferencia
        origen.retirar(monto);
        destino.depositar(monto);
        
        // Guardar (opcional con JPA, se guarda al commit)
        cuentaRepository.save(origen);
        cuentaRepository.save(destino);
        
        // Registrar movimientos
        movimientoRepository.save(new Movimiento(origen, TipoMovimiento.RETIRO, monto.negate()));
        movimientoRepository.save(new Movimiento(destino, TipoMovimiento.DEPOSITO, monto));
        
        // Notificar (no hace rollback si falla)
        try {
            notificacionService.notificarTransferencia(origen, destino, monto);
        } catch (NotificacionException e) {
            // Log pero no rollback
            log.warn("No se pudo notificar: {}", e.getMessage());
        }
    }
}

**Atributos de @Transactional:**

|Atributo|Descripcion|
|---|---|
|`propagation`|Como se comporta con transacciones existentes|
|`isolation`|Nivel de aislamiento|
|`timeout`|Tiempo maximo en segundos|
|`readOnly`|Optimizacion para solo lectura|
|`rollbackFor`|Excepciones que causan rollback|
|`noRollbackFor`|Excepciones que NO causan rollback|

## Ejercicio 3: Propagacion de transacciones

### Enunciado

Dado el siguiente codigo, indica que ocurre en cada escenario:

@Service
public class ServicioA {
    @Autowired private ServicioB servicioB;
    
    @Transactional
    public void metodoA() {
        // operacion 1
        servicioB.metodoB();
        // operacion 2 - lanza excepcion
    }
}

@Service
public class ServicioB {
    @Transactional(propagation = ???)
    public void metodoB() {
        // operacion 3
    }
}

Que pasa con operacion 1 y 3 si operacion 2 falla, para cada propagacion: a) REQUIRED b) REQUIRES_NEW c) NESTED

Ver solucion

**a) REQUIRED (default):**

metodoA() ----[TX1]------------------->
                 |
                 +--> metodoB() usa TX1
                 |
                 X excepcion en op2
                 |
            ROLLBACK TODO

Resultado: op1, op2 y op3 -> ROLLBACK

- metodoB se une a la transaccion existente
    
- Si cualquier parte falla, todo se deshace


---

**b) REQUIRES_NEW:**

metodoA() ----[TX1]------------------->
                 |      
                 | (TX1 suspendida)
                 +--> metodoB() [TX2] --> COMMIT
                 | (TX1 resumida)
                 X excepcion en op2
                 |
            ROLLBACK TX1

Resultado: 
- op3 -> COMMIT (TX2 independiente)
- op1, op2 -> ROLLBACK (TX1)

- metodoB crea su propia transaccion
    
- Si TX2 commitea, sus cambios persisten aunque TX1 falle


---

**c) NESTED:**

metodoA() ----[TX1]------------------->
                 |      
                 +--> metodoB() [SAVEPOINT]
                 |
                 X excepcion en op2
                 |
            ROLLBACK TODO (incluyendo savepoint)

Resultado: op1, op2 y op3 -> ROLLBACK

- metodoB crea un savepoint dentro de TX1
    
- Si metodoB falla, solo se deshace hasta el savepoint
    
- Si metodoA falla despues, todo se deshace


---

**Tabla resumen:**

|Propagacion|Usa TX existente|Crea nueva TX|Rollback afecta a|
|---|---|---|---|
|REQUIRED|Si|Si (si no hay)|Todo|
|REQUIRES_NEW|No|Siempre|Solo la nueva|
|NESTED|Si (savepoint)|No|Hasta savepoint|
|MANDATORY|Si (error si no hay)|No|Todo|
|SUPPORTS|Si|No crea|Depende|
|NOT_SUPPORTED|No (suspende)|No|Nada|
|NEVER|Error si hay|No|Nada|

## Ejercicio 4: Niveles de aislamiento

### Enunciado

Describe que problemas de concurrencia pueden ocurrir en cada nivel y da un ejemplo de cuando usarlo:

1. READ_UNCOMMITTED
    
2. READ_COMMITTED
    
3. REPEATABLE_READ
    
4. SERIALIZABLE


Ver solucion

|Nivel|Dirty Read|Non-repeatable Read|Phantom Read|Uso tipico|
|---|---|---|---|---|
|READ_UNCOMMITTED|Posible|Posible|Posible|Casi nunca (reportes aproximados)|
|READ_COMMITTED|No|Posible|Posible|Default en la mayoria de BDs|
|REPEATABLE_READ|No|No|Posible|Consultas que se repiten|
|SERIALIZABLE|No|No|No|Operaciones financieras criticas|

---

**Dirty Read:**

TX1: UPDATE cuenta SET saldo = 500 WHERE id = 1
TX2: SELECT saldo FROM cuenta WHERE id = 1  --> lee 500
TX1: ROLLBACK
TX2 leyo un valor que nunca existio realmente

**Non-repeatable Read:**

TX1: SELECT saldo FROM cuenta WHERE id = 1  --> 1000
TX2: UPDATE cuenta SET saldo = 500 WHERE id = 1; COMMIT
TX1: SELECT saldo FROM cuenta WHERE id = 1  --> 500 (diferente!)

**Phantom Read:**

TX1: SELECT COUNT(*) FROM productos WHERE precio < 100  --> 10
TX2: INSERT INTO productos (precio) VALUES (50); COMMIT
TX1: SELECT COUNT(*) FROM productos WHERE precio < 100  --> 11 (fantasma!)

---

**Ejemplo de uso:**

// Para transferencias bancarias: maximo aislamiento
@Transactional(isolation = Isolation.SERIALIZABLE)
public void transferenciaCritica(...) { }

// Para reportes que pueden tolerar datos ligeramente desactualizados
@Transactional(isolation = Isolation.READ_COMMITTED, readOnly = true)
public ReporteDTO generarReporte() { }

// Para lecturas que deben ser consistentes dentro de la transaccion
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void procesoConMultiplesLecturas() { }

## Ejercicio 5: Bloqueo optimista

### Enunciado

Implementa bloqueo optimista en la entidad `Producto` y maneja la excepcion `OptimisticLockException` cuando dos usuarios intentan modificar el mismo producto.

Ver solucion

**Entidad con @Version:**

@Entity
public class Producto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private BigDecimal precio;
    private Integer stock;
    
    @Version
    private Long version;
    
    // getters, setters...
}

**Servicio con manejo de conflictos:**

@Service
public class ProductoService {
    
    @Autowired
    private ProductoRepository repository;
    
    @Transactional
    public Producto actualizarPrecio(Long id, BigDecimal nuevoPrecio) {
        Producto producto = repository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("Producto no encontrado"));
        
        producto.setPrecio(nuevoPrecio);
        
        return repository.save(producto);
        // Si otro usuario modifico el producto, lanzara OptimisticLockException
    }
    
    public Producto actualizarConReintentos(Long id, BigDecimal nuevoPrecio, int maxReintentos) {
        int intentos = 0;
        
        while (intentos < maxReintentos) {
            try {
                return actualizarPrecio(id, nuevoPrecio);
            } catch (OptimisticLockException e) {
                intentos++;
                if (intentos >= maxReintentos) {
                    throw new ConflictoActualizacionException(
                        "No se pudo actualizar despues de " + maxReintentos + " intentos", e);
                }
                // Esperar un poco antes de reintentar
                try {
                    Thread.sleep(100 * intentos);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                }
            }
        }
        
        throw new ConflictoActualizacionException("Fallo inesperado");
    }
}

**Controller con manejo de error:**

@RestController
@RequestMapping("/productos")
public class ProductoController {
    
    @PutMapping("/{id}/precio")
    public ResponseEntity<?> actualizarPrecio(
            @PathVariable Long id,
            @RequestBody ActualizarPrecioRequest request) {
        try {
            Producto actualizado = productoService.actualizarConReintentos(
                id, request.getNuevoPrecio(), 3);
            return ResponseEntity.ok(actualizado);
        } catch (ConflictoActualizacionException e) {
            return ResponseEntity.status(HttpStatus.CONFLICT)
                .body("El producto fue modificado por otro usuario. Por favor, recargue y reintente.");
        }
    }
}

**SQL generado:**

UPDATE productos 
SET nombre = ?, precio = ?, stock = ?, version = version + 1 
WHERE id = ? AND version = ?

Si `version` no coincide, 0 filas afectadas -> `OptimisticLockException`

## Reto adicional

Implementa un sistema de saga para una operacion distribuida:

1. Crear pedido
    
2. Reservar stock
    
3. Procesar pago
    
4. Enviar confirmacion


Si cualquier paso falla, ejecuta las compensaciones en orden inverso.

Consejo

Guarda el estado de cada paso para poder compensar. Usa `REQUIRES_NEW` para que cada paso sea independiente.