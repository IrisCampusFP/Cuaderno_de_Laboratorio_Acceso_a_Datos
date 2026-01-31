## Objetivos de aprendizaje

Al finalizar este bloque, el alumno será capaz de:

- Comprender las propiedades ACID y su importancia en sistemas de bases de datos
    
- Gestionar transacciones manualmente con EntityTransaction en JPA puro
    
- Utilizar la anotación `@Transactional` de Spring para gestión declarativa
    
- Configurar los diferentes modos de propagación de transacciones
    
- Seleccionar el nivel de aislamiento apropiado según el escenario
    
- Implementar estrategias de rollback personalizadas
    
- Aplicar bloqueo optimista y pesimista para control de concurrencia
    
- Diagnosticar y resolver problemas comunes relacionados con transacciones

## Criterios de evaluación relacionados

Este bloque desarrolla principalmente:

- **CE 3g**: Se han gestionado las transacciones (10%)

## 1. Fundamentos de transacciones

### 1.1. Concepto de transacción

Una transacción es una unidad lógica de trabajo que agrupa una o más operaciones sobre la base de datos. Estas operaciones se ejecutan como un todo indivisible: o todas se completan exitosamente, o ninguna tiene efecto.

En el contexto de aplicaciones empresariales, las transacciones garantizan que el sistema pase de un estado consistente a otro estado consistente, incluso ante fallos del sistema o errores de la aplicación.

Consideremos el ejemplo clásico de una transferencia bancaria: cuando transferimos dinero de una cuenta a otra, necesitamos realizar dos operaciones: debitar la cuenta origen y acreditar la cuenta destino. Ambas operaciones deben ejecutarse como una unidad indivisible. Si falla la segunda operación, la primera debe revertirse para mantener la integridad de los datos.

// Sin transacciones - PELIGROSO
public void transferirSinTransaccion(Long cuentaOrigenId, Long cuentaDestinoId, BigDecimal monto) {
    Cuenta origen = cuentaRepository.findById(cuentaOrigenId);
    origen.setSaldo(origen.getSaldo().subtract(monto));
    cuentaRepository.save(origen);  // Se guarda
    
    // Si ocurre un error aquí, el dinero desaparece
    
    Cuenta destino = cuentaRepository.findById(cuentaDestinoId);
    destino.setSaldo(destino.getSaldo().add(monto));
    cuentaRepository.save(destino);  // Puede fallar
}

// Con transacciones - CORRECTO
@Transactional
public void transferirConTransaccion(Long cuentaOrigenId, Long cuentaDestinoId, BigDecimal monto) {
    Cuenta origen = cuentaRepository.findById(cuentaOrigenId);
    origen.setSaldo(origen.getSaldo().subtract(monto));
    cuentaRepository.save(origen);
    
    // Si ocurre un error, todo se revierte automáticamente
    
    Cuenta destino = cuentaRepository.findById(cuentaDestinoId);
    destino.setSaldo(destino.getSaldo().add(monto));
    cuentaRepository.save(destino);
}

### 1.2. Propiedades ACID

Las transacciones deben cumplir cuatro propiedades fundamentales conocidas como ACID:

#### Atomicidad (Atomicity)

La atomicidad garantiza que todas las operaciones de una transacción se ejecuten como una unidad indivisible. Si alguna operación falla, todas las operaciones anteriores se revierten (rollback). No existen estados intermedios visibles.

@Transactional
public void crearPedidoCompleto(Pedido pedido, List<LineaPedido> lineas) {
    // Si cualquier operación falla, ninguna se persiste
    pedidoRepository.save(pedido);
    
    for (LineaPedido linea : lineas) {
        linea.setPedido(pedido);
        lineaPedidoRepository.save(linea);
        
        // Actualizar stock
        Producto producto = linea.getProducto();
        producto.reducirStock(linea.getCantidad());
        productoRepository.save(producto);
    }
}

#### Consistencia (Consistency)

La consistencia asegura que una transacción lleva la base de datos de un estado válido a otro estado válido, respetando todas las reglas de integridad definidas (claves primarias, foráneas, restricciones, triggers).

@Transactional
public void asignarEmpleadoADepartamento(Long empleadoId, Long departamentoId) {
    Empleado empleado = empleadoRepository.findById(empleadoId)
        .orElseThrow(() -> new EntityNotFoundException("Empleado no encontrado"));
    
    Departamento departamento = departamentoRepository.findById(departamentoId)
        .orElseThrow(() -> new EntityNotFoundException("Departamento no encontrado"));
    
    // La consistencia garantiza que el departamento existe antes de asignarlo
    empleado.setDepartamento(departamento);
    empleadoRepository.save(empleado);
}

#### Aislamiento (Isolation)

El aislamiento determina cómo y cuándo los cambios realizados por una transacción son visibles para otras transacciones concurrentes. Los diferentes niveles de aislamiento ofrecen distintos compromisos entre consistencia y rendimiento.

@Transactional(isolation = Isolation.READ_COMMITTED)
public BigDecimal calcularSaldoTotal(Long clienteId) {
    // Esta transacción no verá cambios sin confirmar de otras transacciones
    List<Cuenta> cuentas = cuentaRepository.findByClienteId(clienteId);
    return cuentas.stream()
        .map(Cuenta::getSaldo)
        .reduce(BigDecimal.ZERO, BigDecimal::add);
}

#### Durabilidad (Durability)

La durabilidad garantiza que una vez que una transacción ha sido confirmada (commit), sus cambios permanecen permanentemente en la base de datos, incluso ante fallos del sistema, cortes de energía o caídas del servidor.

### 1.3. Estados de una transacción

Una transacción puede encontrarse en diferentes estados durante su ciclo de vida:

|Estado|Descripción|
|---|---|
|Activa|La transacción está en ejecución|
|Parcialmente confirmada|Todas las operaciones han sido ejecutadas|
|Confirmada (Committed)|Los cambios se han hecho permanentes|
|Fallida|Se ha detectado un error|
|Abortada (Rolled back)|Los cambios han sido revertidos|

     +--------+
     | ACTIVA |
     +--------+
         |
         v
+--------------------+
| PARCIALMENTE       |
| CONFIRMADA         |
+--------------------+
    |           |
    v           v
+--------+  +--------+
| COMMIT |  | FALLO  |
+--------+  +--------+
    |           |
    v           v
+----------+ +----------+
| DURABLES | | ROLLBACK |
+----------+ +----------+

## 2. Transacciones en JPA puro

### 2.1. EntityTransaction

Cuando trabajamos con JPA sin Spring, debemos gestionar las transacciones manualmente utilizando la interfaz `EntityTransaction` obtenida del `EntityManager`.

public class TransaccionManualDemo {
    
    private EntityManagerFactory emf;
    
    public TransaccionManualDemo() {
        emf = Persistence.createEntityManagerFactory("mi-unidad-persistencia");
    }
    
    public void crearProducto(String nombre, BigDecimal precio) {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = null;
        
        try {
            tx = em.getTransaction();
            tx.begin();
            
            Producto producto = new Producto();
            producto.setNombre(nombre);
            producto.setPrecio(precio);
            producto.setFechaCreacion(LocalDateTime.now());
            
            em.persist(producto);
            
            tx.commit();
            System.out.println("Producto creado con ID: " + producto.getId());
            
        } catch (Exception e) {
            if (tx != null && tx.isActive()) {
                tx.rollback();
                System.err.println("Transacción revertida: " + e.getMessage());
            }
            throw e;
        } finally {
            em.close();
        }
    }
}

### 2.2. Patrón try-with-resources mejorado

Para código más limpio, podemos crear una utilidad que gestione las transacciones:

public class TransactionHelper {
    
    private final EntityManagerFactory emf;
    
    public TransactionHelper(EntityManagerFactory emf) {
        this.emf = emf;
    }
    
    public <T> T executeInTransaction(Function<EntityManager, T> operation) {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        
        try {
            tx.begin();
            T result = operation.apply(em);
            tx.commit();
            return result;
        } catch (Exception e) {
            if (tx.isActive()) {
                tx.rollback();
            }
            throw new RuntimeException("Error en transacción", e);
        } finally {
            em.close();
        }
    }
    
    public void executeInTransaction(Consumer<EntityManager> operation) {
        executeInTransaction(em -> {
            operation.accept(em);
            return null;
        });
    }
}

// Uso
TransactionHelper txHelper = new TransactionHelper(emf);

txHelper.executeInTransaction(em -> {
    Producto producto = new Producto("Laptop", new BigDecimal("999.99"));
    em.persist(producto);
});

Producto encontrado = txHelper.executeInTransaction(em -> {
    return em.find(Producto.class, 1L);
});

### 2.3. Operaciones dentro de una transacción

Las operaciones típicas que se realizan dentro de una transacción incluyen:

public void ejemploOperacionesTransaccionales(EntityManager em) {
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    
    try {
        // CREATE - persist
        Categoria categoria = new Categoria("Electrónica");
        em.persist(categoria);
        
        // La entidad obtiene ID después del flush
        em.flush();
        System.out.println("Categoría ID: " + categoria.getId());
        
        // READ - find
        Producto producto = em.find(Producto.class, 1L);
        
        // UPDATE - los cambios se detectan automáticamente (dirty checking)
        producto.setPrecio(producto.getPrecio().multiply(new BigDecimal("1.10")));
        
        // DELETE - remove
        Producto obsoleto = em.find(Producto.class, 999L);
        if (obsoleto != null) {
            em.remove(obsoleto);
        }
        
        // JPQL UPDATE masivo
        Query updateQuery = em.createQuery(
            "UPDATE Producto p SET p.precio = p.precio * 0.9 WHERE p.categoria = :cat"
        );
        updateQuery.setParameter("cat", categoria);
        int actualizados = updateQuery.executeUpdate();
        
        tx.commit();
        
    } catch (Exception e) {
        tx.rollback();
        throw e;
    }
}

## 3. Transacciones declarativas con Spring

### 3.1. La anotación `@Transactional`

Spring proporciona gestión declarativa de transacciones mediante la anotación `@Transactional`. Esta anotación puede aplicarse a nivel de clase o método.

@Service
public class ProductoService {
    
    @Autowired
    private ProductoRepository productoRepository;
    
    @Autowired
    private CategoriaRepository categoriaRepository;
    
    // Transacción a nivel de método
    @Transactional
    public Producto crearProducto(ProductoDTO dto) {
        Categoria categoria = categoriaRepository.findById(dto.getCategoriaId())
            .orElseThrow(() -> new EntityNotFoundException("Categoría no encontrada"));
        
        Producto producto = new Producto();
        producto.setNombre(dto.getNombre());
        producto.setPrecio(dto.getPrecio());
        producto.setCategoria(categoria);
        
        return productoRepository.save(producto);
    }
    
    // Transacción de solo lectura (optimizada)
    @Transactional(readOnly = true)
    public List<Producto> buscarPorCategoria(Long categoriaId) {
        return productoRepository.findByCategoriaId(categoriaId);
    }
}

### 3.2. Transacciones a nivel de clase

Cuando se aplica `@Transactional` a nivel de clase, todos los métodos públicos heredan esa configuración:

@Service
@Transactional  // Aplica a todos los métodos
public class PedidoService {
    
    @Autowired
    private PedidoRepository pedidoRepository;
    
    @Autowired
    private ProductoRepository productoRepository;
    
    public Pedido crearPedido(PedidoDTO dto) {
        // Este método es transaccional
        Pedido pedido = new Pedido();
        pedido.setCliente(dto.getCliente());
        pedido.setFecha(LocalDateTime.now());
        return pedidoRepository.save(pedido);
    }
    
    @Transactional(readOnly = true)  // Sobrescribe la configuración de clase
    public Pedido buscarPorId(Long id) {
        return pedidoRepository.findById(id).orElse(null);
    }
    
    @Transactional(timeout = 30)  // Timeout de 30 segundos
    public void procesarPedidosEnLote(List<Long> ids) {
        for (Long id : ids) {
            procesarPedido(id);
        }
    }
}

### 3.3. Atributos de `@Transactional`

La anotación `@Transactional` ofrece múltiples atributos de configuración:

|Atributo|Descripción|Valor por defecto|
|---|---|---|
|propagation|Comportamiento ante transacciones existentes|REQUIRED|
|isolation|Nivel de aislamiento|DEFAULT (del SGBD)|
|timeout|Tiempo máximo en segundos|-1 (sin límite)|
|readOnly|Optimización para solo lectura|false|
|rollbackFor|Excepciones que provocan rollback|RuntimeException|
|noRollbackFor|Excepciones que NO provocan rollback|ninguna|
|transactionManager|Gestor de transacciones a usar|transactionManager|

@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    timeout = 60,
    readOnly = false,
    rollbackFor = {SQLException.class, IOException.class},
    noRollbackFor = {EntityNotFoundException.class}
)
public void operacionCompleja() {
    // Código de la operación
}

## 4. Propagación de transacciones

### 4.1. Concepto de propagación

La propagación define cómo se comporta un método transaccional cuando es invocado dentro del contexto de otra transacción. Es crucial entender este concepto cuando tenemos métodos transaccionales que llaman a otros métodos transaccionales.

### 4.2. Modos de propagación

#### REQUIRED (por defecto)

Si existe una transacción activa, el método participa en ella. Si no existe, crea una nueva.

@Service
public class PedidoServiceRequired {
    
    @Autowired
    private LineaPedidoService lineaService;
    
    @Transactional(propagation = Propagation.REQUIRED)
    public void crearPedidoConLineas(Pedido pedido, List<LineaPedido> lineas) {
        pedidoRepository.save(pedido);
        
        for (LineaPedido linea : lineas) {
            // lineaService.agregarLinea usa la MISMA transacción
            lineaService.agregarLinea(pedido, linea);
        }
        // Si algo falla, TODO se revierte
    }
}

@Service
public class LineaPedidoService {
    
    @Transactional(propagation = Propagation.REQUIRED)
    public void agregarLinea(Pedido pedido, LineaPedido linea) {
        linea.setPedido(pedido);
        lineaRepository.save(linea);
    }
}

#### REQUIRES_NEW

Siempre crea una nueva transacción independiente. Si existe una transacción activa, la suspende temporalmente.

@Service
public class AuditoriaService {
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void registrarEvento(String evento, String detalle) {
        AuditoriaLog log = new AuditoriaLog();
        log.setEvento(evento);
        log.setDetalle(detalle);
        log.setFecha(LocalDateTime.now());
        
        auditoriaRepository.save(log);
        // Este log se guarda AUNQUE la transacción principal falle
    }
}

@Service
public class TransferenciaService {
    
    @Autowired
    private AuditoriaService auditoriaService;
    
    @Transactional
    public void realizarTransferencia(Long origenId, Long destinoId, BigDecimal monto) {
        // Registrar inicio (se guarda aunque falle la transferencia)
        auditoriaService.registrarEvento("TRANSFERENCIA_INICIADA", 
            String.format("%d -> %d: %s", origenId, destinoId, monto));
        
        try {
            Cuenta origen = cuentaRepository.findById(origenId).orElseThrow();
            Cuenta destino = cuentaRepository.findById(destinoId).orElseThrow();
            
            origen.debitar(monto);
            destino.acreditar(monto);
            
            cuentaRepository.save(origen);
            cuentaRepository.save(destino);
            
            auditoriaService.registrarEvento("TRANSFERENCIA_COMPLETADA", "OK");
            
        } catch (Exception e) {
            // Este registro se guarda aunque hagamos rollback
            auditoriaService.registrarEvento("TRANSFERENCIA_FALLIDA", e.getMessage());
            throw e;
        }
    }
}

#### SUPPORTS

Participa en una transacción existente si la hay, pero puede ejecutarse sin transacción.

@Service
public class ConsultaService {
    
    @Transactional(propagation = Propagation.SUPPORTS, readOnly = true)
    public List<Producto> buscarProductos(String termino) {
        // Funciona con o sin transacción activa
        return productoRepository.findByNombreContaining(termino);
    }
}

#### MANDATORY

Requiere que exista una transacción activa. Lanza excepción si no hay ninguna.

@Service
public class StockService {
    
    @Transactional(propagation = Propagation.MANDATORY)
    public void reducirStock(Long productoId, int cantidad) {
        // DEBE ser llamado dentro de una transacción existente
        Producto producto = productoRepository.findById(productoId).orElseThrow();
        producto.setStock(producto.getStock() - cantidad);
        productoRepository.save(producto);
    }
}

// Uso correcto
@Transactional
public void procesarVenta(Long productoId, int cantidad) {
    stockService.reducirStock(productoId, cantidad);  // OK
}

// Uso incorrecto - lanza IllegalTransactionStateException
public void ventaSinTransaccion(Long productoId) {
    stockService.reducirStock(productoId, 1);  // ERROR!
}

#### NOT_SUPPORTED

Suspende cualquier transacción existente y ejecuta sin transacción.

@Service
public class ReporteService {
    
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public ReporteDTO generarReportePesado() {
        // Operación larga que no necesita transacción
        // Evita bloquear recursos transaccionales
        return construirReporteComplejo();
    }
}

#### NEVER

Lanza excepción si existe una transacción activa.

@Service
public class CacheService {
    
    @Transactional(propagation = Propagation.NEVER)
    public void limpiarCache() {
        // Esta operación NO debe ejecutarse dentro de una transacción
        cacheManager.clear();
    }
}

#### NESTED

Crea una transacción anidada con un savepoint. Si la transacción anidada falla, solo se revierte hasta el savepoint.

@Service
public class ImportacionService {
    
    @Transactional
    public ImportacionResultado importarDatos(List<RegistroDTO> registros) {
        ImportacionResultado resultado = new ImportacionResultado();
        
        for (RegistroDTO registro : registros) {
            try {
                procesarRegistroAnidado(registro);
                resultado.incrementarExitos();
            } catch (Exception e) {
                // Solo este registro falla, los demás continúan
                resultado.agregarError(registro, e.getMessage());
            }
        }
        
        return resultado;
    }
    
    @Transactional(propagation = Propagation.NESTED)
    public void procesarRegistroAnidado(RegistroDTO registro) {
        // Si falla, solo se revierte este registro
        Entidad entidad = convertir(registro);
        repository.save(entidad);
    }
}

### 4.3. Tabla resumen de propagación

|Propagación|Transacción existente|Sin transacción|
|---|---|---|
|REQUIRED|Usa existente|Crea nueva|
|REQUIRES_NEW|Suspende y crea nueva|Crea nueva|
|SUPPORTS|Usa existente|Sin transacción|
|NOT_SUPPORTED|Suspende|Sin transacción|
|MANDATORY|Usa existente|Lanza excepción|
|NEVER|Lanza excepción|Sin transacción|
|NESTED|Crea savepoint|Crea nueva|

## 5. Niveles de aislamiento

### 5.1. Problemas de concurrencia

Cuando múltiples transacciones acceden a los mismos datos simultáneamente, pueden ocurrir varios problemas:

#### Dirty Read (Lectura sucia)

Una transacción lee datos que otra transacción aún no ha confirmado.

Transacción A                    Transacción B
----------------                 ----------------
                                 UPDATE cuenta SET saldo = 5000
                                 WHERE id = 1
SELECT saldo FROM cuenta         (sin commit aún)
WHERE id = 1
-- Lee 5000 (dato no confirmado)
                                 ROLLBACK
-- El valor 5000 nunca existió realmente

#### Non-Repeatable Read (Lectura no repetible)

Una transacción lee el mismo dato dos veces y obtiene valores diferentes.

Transacción A                    Transacción B
----------------                 ----------------
SELECT saldo FROM cuenta
WHERE id = 1
-- Lee 1000
                                 UPDATE cuenta SET saldo = 2000
                                 WHERE id = 1
                                 COMMIT
SELECT saldo FROM cuenta
WHERE id = 1
-- Lee 2000 (diferente!)

#### Phantom Read (Lectura fantasma)

Una transacción ejecuta la misma consulta dos veces y obtiene conjuntos de filas diferentes.

Transacción A                    Transacción B
----------------                 ----------------
SELECT COUNT(*) FROM productos
WHERE precio > 100
-- Cuenta 10 productos
                                 INSERT INTO productos
                                 VALUES ('Nuevo', 150)
                                 COMMIT
SELECT COUNT(*) FROM productos
WHERE precio > 100
-- Cuenta 11 productos (fantasma!)

### 5.2. Niveles de aislamiento en JPA

JPA define cuatro niveles de aislamiento, ordenados de menor a mayor restricción:

#### READ_UNCOMMITTED

El nivel más bajo. Permite leer datos no confirmados (dirty reads).

@Transactional(isolation = Isolation.READ_UNCOMMITTED)
public BigDecimal consultarSaldoAproximado(Long cuentaId) {
    // Puede ver cambios no confirmados
    // Útil cuando la precisión no es crítica y se prioriza el rendimiento
    return cuentaRepository.findById(cuentaId)
        .map(Cuenta::getSaldo)
        .orElse(BigDecimal.ZERO);
}

#### READ_COMMITTED

Previene dirty reads. Solo lee datos confirmados.

@Transactional(isolation = Isolation.READ_COMMITTED)
public void procesarSaldo(Long cuentaId) {
    // Solo ve datos que han sido confirmados (commit)
    Cuenta cuenta = cuentaRepository.findById(cuentaId).orElseThrow();
    
    // Pero si vuelvo a leer, podría obtener un valor diferente
    // si otra transacción hizo commit mientras tanto
}

#### REPEATABLE_READ

Previene dirty reads y non-repeatable reads. Las lecturas son consistentes durante toda la transacción.

@Transactional(isolation = Isolation.REPEATABLE_READ)
public ReporteBalance generarBalance(Long cuentaId) {
    // Primera lectura
    Cuenta cuenta = cuentaRepository.findById(cuentaId).orElseThrow();
    BigDecimal saldoInicial = cuenta.getSaldo();
    
    // ... operaciones intermedias ...
    
    // Segunda lectura - garantizado mismo valor
    cuenta = cuentaRepository.findById(cuentaId).orElseThrow();
    BigDecimal saldoFinal = cuenta.getSaldo();
    
    // saldoInicial == saldoFinal (garantizado)
    return new ReporteBalance(saldoInicial, saldoFinal);
}

#### SERIALIZABLE

El nivel más alto. Previene todos los problemas de concurrencia. Las transacciones se ejecutan como si fueran secuenciales.

@Transactional(isolation = Isolation.SERIALIZABLE)
public void cierreContableMensual() {
    // Nadie puede modificar datos mientras se ejecuta el cierre
    List<Cuenta> cuentas = cuentaRepository.findAll();
    
    BigDecimal totalActivo = calcularTotalActivo(cuentas);
    BigDecimal totalPasivo = calcularTotalPasivo(cuentas);
    
    CierreContable cierre = new CierreContable();
    cierre.setMes(YearMonth.now().minusMonths(1));
    cierre.setTotalActivo(totalActivo);
    cierre.setTotalPasivo(totalPasivo);
    
    cierreRepository.save(cierre);
}

### 5.3. Tabla resumen de niveles de aislamiento

|Nivel|Dirty Read|Non-Repeatable Read|Phantom Read|
|---|---|---|---|
|READ_UNCOMMITTED|Posible|Posible|Posible|
|READ_COMMITTED|Prevenido|Posible|Posible|
|REPEATABLE_READ|Prevenido|Prevenido|Posible|
|SERIALIZABLE|Prevenido|Prevenido|Prevenido|

### 5.4. Selección del nivel apropiado

La elección del nivel de aislamiento depende del equilibrio entre consistencia y rendimiento:

@Service
public class OperacionesFinancierasService {
    
    // Para consultas informativas - rendimiento prioritario
    @Transactional(isolation = Isolation.READ_COMMITTED, readOnly = true)
    public EstadisticasGenerales obtenerEstadisticas() {
        return estadisticasRepository.calcularEstadisticas();
    }
    
    // Para operaciones de lectura que requieren consistencia
    @Transactional(isolation = Isolation.REPEATABLE_READ, readOnly = true)
    public ReporteSaldos generarReporteSaldos() {
        List<Cuenta> cuentas = cuentaRepository.findAll();
        // Las lecturas son consistentes durante todo el reporte
        return construirReporte(cuentas);
    }
    
    // Para operaciones críticas - máxima consistencia
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void realizarCierreContable() {
        // Nadie puede interferir durante el cierre
        cierreService.ejecutarCierre();
    }
}

## 6. Rollback y manejo de excepciones

### 6.1. Comportamiento por defecto

Por defecto, Spring hace rollback solo para excepciones no verificadas (RuntimeException y sus subclases) y errores (Error). Las excepciones verificadas NO provocan rollback automáticamente.

@Service
public class EjemploRollbackService {
    
    @Transactional
    public void operacionConRuntimeException() {
        productoRepository.save(new Producto("Test", new BigDecimal("100")));
        
        throw new RuntimeException("Error de prueba");
        // ROLLBACK automático - el producto NO se guarda
    }
    
    @Transactional
    public void operacionConCheckedException() throws IOException {
        productoRepository.save(new Producto("Test", new BigDecimal("100")));
        
        throw new IOException("Error de E/S");
        // SIN rollback por defecto - el producto SE GUARDA
    }
}

### 6.2. Configuración de rollback personalizado

Podemos personalizar qué excepciones provocan rollback:

@Service
public class RollbackPersonalizadoService {
    
    // Rollback para excepciones verificadas específicas
    @Transactional(rollbackFor = {IOException.class, SQLException.class})
    public void operacionConRollbackExtendido() throws IOException {
        productoRepository.save(new Producto("Test", new BigDecimal("100")));
        throw new IOException("Error de archivo");
        // ROLLBACK porque IOException está en rollbackFor
    }
    
    // Rollback para CUALQUIER excepción
    @Transactional(rollbackFor = Exception.class)
    public void operacionConRollbackTotal() throws Exception {
        productoRepository.save(new Producto("Test", new BigDecimal("100")));
        throw new Exception("Cualquier error");
        // ROLLBACK
    }
    
    // Evitar rollback para excepciones específicas
    @Transactional(noRollbackFor = {EntityNotFoundException.class})
    public Producto buscarOCrear(Long id, String nombreDefault) {
        try {
            return productoRepository.findById(id).orElseThrow(
                () -> new EntityNotFoundException("No encontrado")
            );
        } catch (EntityNotFoundException e) {
            // NO hace rollback, crea un nuevo producto
            return productoRepository.save(
                new Producto(nombreDefault, BigDecimal.ZERO)
            );
        }
    }
}

### 6.3. Rollback programático

También podemos forzar un rollback programáticamente:

@Service
public class RollbackProgramaticoService {
    
    @Transactional
    public void operacionConRollbackManual() {
        productoRepository.save(new Producto("Test", new BigDecimal("100")));
        
        if (algunaCondicionDeFallo()) {
            // Marcar la transacción para rollback
            TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
            return;
        }
        
        // Continuar con más operaciones...
    }
    
    // Alternativa usando TransactionTemplate
    @Autowired
    private PlatformTransactionManager transactionManager;
    
    public void operacionConTransactionTemplate() {
        TransactionTemplate template = new TransactionTemplate(transactionManager);
        
        template.execute(status -> {
            productoRepository.save(new Producto("Test", new BigDecimal("100")));
            
            if (algunaCondicionDeFallo()) {
                status.setRollbackOnly();
                return null;
            }
            
            return productoRepository.save(new Producto("Otro", new BigDecimal("200")));
        });
    }
}

## 7. Bloqueo optimista

### 7.1. Concepto

El bloqueo optimista asume que los conflictos de concurrencia son raros. No bloquea los registros, sino que detecta conflictos en el momento de la actualización mediante un campo de versión.

### 7.2. Implementación con `@Version`

@Entity
@Table(name = "productos")
public class Producto {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    private BigDecimal precio;
    
    private Integer stock;
    
    @Version
    private Long version;
    
    // Getters y setters
}

### 7.3. Funcionamiento del bloqueo optimista

@Service
public class ProductoServiceOptimista {
    
    @Transactional
    public void actualizarPrecio(Long productoId, BigDecimal nuevoPrecio) {
        Producto producto = productoRepository.findById(productoId)
            .orElseThrow(() -> new EntityNotFoundException("Producto no encontrado"));
        
        // Hibernate genera: UPDATE productos SET precio = ?, version = version + 1
        //                   WHERE id = ? AND version = ?
        producto.setPrecio(nuevoPrecio);
        productoRepository.save(producto);
        
        // Si otro proceso modificó el producto mientras tanto,
        // la versión no coincidirá y se lanzará OptimisticLockException
    }
}

### 7.4. Manejo de OptimisticLockException

@Service
public class ProductoServiceConReintentos {
    
    private static final int MAX_REINTENTOS = 3;
    
    @Autowired
    private ProductoRepository productoRepository;
    
    public void actualizarStockConReintentos(Long productoId, int cantidad) {
        int intentos = 0;
        
        while (intentos < MAX_REINTENTOS) {
            try {
                actualizarStock(productoId, cantidad);
                return; // Éxito
            } catch (OptimisticLockException e) {
                intentos++;
                if (intentos >= MAX_REINTENTOS) {
                    throw new ConcurrenciaException(
                        "No se pudo actualizar el stock después de " + MAX_REINTENTOS + " intentos"
                    );
                }
                // Esperar un poco antes de reintentar
                try {
                    Thread.sleep(100 * intentos);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                }
            }
        }
    }
    
    @Transactional
    public void actualizarStock(Long productoId, int cantidad) {
        Producto producto = productoRepository.findById(productoId)
            .orElseThrow(() -> new EntityNotFoundException("Producto no encontrado"));
        
        producto.setStock(producto.getStock() + cantidad);
        productoRepository.save(producto);
    }
}

## 8. Bloqueo pesimista

### 8.1. Concepto

El bloqueo pesimista bloquea los registros desde el momento en que se leen, impidiendo que otras transacciones los modifiquen hasta que la transacción actual termine.

### 8.2. Tipos de bloqueo pesimista

public interface ProductoRepository extends JpaRepository<Producto, Long> {
    
    // Bloqueo de lectura - otros pueden leer pero no escribir
    @Lock(LockModeType.PESSIMISTIC_READ)
    @Query("SELECT p FROM Producto p WHERE p.id = :id")
    Optional<Producto> findByIdConBloqueoLectura(@Param("id") Long id);
    
    // Bloqueo de escritura - exclusivo, nadie más puede acceder
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT p FROM Producto p WHERE p.id = :id")
    Optional<Producto> findByIdConBloqueoEscritura(@Param("id") Long id);
    
    // Bloqueo con incremento de versión forzado
    @Lock(LockModeType.PESSIMISTIC_FORCE_INCREMENT)
    @Query("SELECT p FROM Producto p WHERE p.id = :id")
    Optional<Producto> findByIdConBloqueoYVersion(@Param("id") Long id);
}

### 8.3. Uso práctico del bloqueo pesimista

@Service
public class ReservaService {
    
    @Autowired
    private AsientoRepository asientoRepository;
    
    @Transactional
    public void reservarAsiento(Long asientoId, Long clienteId) {
        // Bloquear el asiento mientras se procesa la reserva
        Asiento asiento = asientoRepository.findByIdConBloqueoEscritura(asientoId)
            .orElseThrow(() -> new EntityNotFoundException("Asiento no encontrado"));
        
        if (asiento.isReservado()) {
            throw new AsientoNoDisponibleException("El asiento ya está reservado");
        }
        
        asiento.setReservado(true);
        asiento.setClienteId(clienteId);
        asiento.setFechaReserva(LocalDateTime.now());
        
        asientoRepository.save(asiento);
        // El bloqueo se libera al hacer commit
    }
}

### 8.4. Timeout en bloqueos

@Service
public class BloqueoConTimeoutService {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Transactional
    public Producto obtenerConTimeout(Long id, int timeoutMs) {
        Map<String, Object> properties = new HashMap<>();
        properties.put("jakarta.persistence.lock.timeout", timeoutMs);
        
        try {
            return entityManager.find(
                Producto.class, 
                id, 
                LockModeType.PESSIMISTIC_WRITE,
                properties
            );
        } catch (LockTimeoutException e) {
            throw new RecursoOcupadoException(
                "No se pudo obtener acceso al producto en " + timeoutMs + "ms"
            );
        }
    }
}

### 8.5. Comparación: Optimista vs Pesimista

|Aspecto|Bloqueo Optimista|Bloqueo Pesimista|
|---|---|---|
|Implementación|Campo `@Version`|`@Lock` en consultas|
|Momento de detección|Al hacer UPDATE|Al hacer SELECT|
|Rendimiento|Mejor en baja contención|Mejor en alta contención|
|Bloqueo de BD|No bloquea|Bloquea registros|
|Deadlocks|No posibles|Posibles|
|Caso de uso|Edición de formularios|Reservas, pagos|

## 9. Problemas comunes y soluciones

### 9.1. Auto-invocación de métodos transaccionales

Uno de los errores más comunes es llamar a un método transaccional desde otro método de la misma clase:

@Service
public class ServicioConProblema {
    
    public void metodoNoTransaccional() {
        // PROBLEMA: Esta llamada NO crea transacción
        metodoTransaccional();  // No pasa por el proxy de Spring
    }
    
    @Transactional
    public void metodoTransaccional() {
        // Este código NO está en una transacción cuando
        // se llama desde metodoNoTransaccional()
        productoRepository.save(new Producto("Test", BigDecimal.TEN));
    }
}

**Soluciones:**

// Solución 1: Inyectar el propio servicio
@Service
public class ServicioCorregido1 {
    
    @Autowired
    private ServicioCorregido1 self;  // Auto-inyección
    
    public void metodoNoTransaccional() {
        self.metodoTransaccional();  // Pasa por el proxy
    }
    
    @Transactional
    public void metodoTransaccional() {
        productoRepository.save(new Producto("Test", BigDecimal.TEN));
    }
}

// Solución 2: Separar en diferentes servicios
@Service
public class ServicioExterno {
    
    @Autowired
    private ServicioTransaccional servicioTransaccional;
    
    public void metodoNoTransaccional() {
        servicioTransaccional.metodoTransaccional();
    }
}

@Service
public class ServicioTransaccional {
    
    @Transactional
    public void metodoTransaccional() {
        productoRepository.save(new Producto("Test", BigDecimal.TEN));
    }
}

### 9.2. Captura de excepciones que impide rollback

@Service
public class ServicioConCapturaIncorrecta {
    
    @Transactional
    public void operacionIncorrecta() {
        try {
            productoRepository.save(new Producto("Test", BigDecimal.TEN));
            throw new RuntimeException("Error");
        } catch (Exception e) {
            // PROBLEMA: La excepción no se propaga, no hay rollback
            log.error("Error capturado", e);
        }
        // El producto SE GUARDA aunque hubo error
    }
    
    @Transactional
    public void operacionCorrecta() {
        try {
            productoRepository.save(new Producto("Test", BigDecimal.TEN));
            throw new RuntimeException("Error");
        } catch (Exception e) {
            log.error("Error capturado", e);
            throw e;  // Re-lanzar para que ocurra rollback
        }
    }
}

### 9.3. Transacciones demasiado largas

@Service
public class ServicioTransaccionLarga {
    
    // PROBLEMA: Transacción muy larga
    @Transactional
    public void procesarMuchosRegistrosIncorrecto(List<RegistroDTO> registros) {
        for (RegistroDTO dto : registros) {
            // Si son miles de registros, la transacción puede durar minutos
            procesarRegistro(dto);
        }
    }
    
    // SOLUCIÓN: Dividir en lotes
    public void procesarMuchosRegistrosCorrectos(List<RegistroDTO> registros) {
        List<List<RegistroDTO>> lotes = Lists.partition(registros, 100);
        
        for (List<RegistroDTO> lote : lotes) {
            procesarLote(lote);  // Cada lote en su propia transacción
        }
    }
    
    @Transactional
    public void procesarLote(List<RegistroDTO> lote) {
        for (RegistroDTO dto : lote) {
            procesarRegistro(dto);
        }
    }
}

### 9.4. Uso incorrecto de readOnly

@Service
public class ServicioReadOnlyIncorrecto {
    
    // PROBLEMA: readOnly = true pero intenta escribir
    @Transactional(readOnly = true)
    public Producto crearProductoIncorrecto(String nombre) {
        Producto producto = new Producto(nombre, BigDecimal.TEN);
        return productoRepository.save(producto);
        // Puede fallar o comportarse de forma impredecible
    }
    
    // CORRECTO: readOnly para consultas, sin readOnly para escritura
    @Transactional(readOnly = true)
    public List<Producto> listarProductos() {
        return productoRepository.findAll();
    }
    
    @Transactional
    public Producto crearProducto(String nombre) {
        Producto producto = new Producto(nombre, BigDecimal.TEN);
        return productoRepository.save(producto);
    }
}

## 10. Ejercicio guiado: Sistema bancario con transacciones

Vamos a implementar un sistema bancario que demuestre el uso correcto de transacciones:

### Paso 1: Entidades del dominio

@Entity
@Table(name = "cuentas")
public class Cuenta {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true, length = 20)
    private String numeroCuenta;
    
    @Column(nullable = false, length = 100)
    private String titular;
    
    @Column(nullable = false, precision = 15, scale = 2)
    private BigDecimal saldo;
    
    @Column(nullable = false)
    private Boolean activa = true;
    
    @Version
    private Long version;
    
    @Column(name = "fecha_apertura", nullable = false)
    private LocalDate fechaApertura;
    
    // Constructores
    public Cuenta() {}
    
    public Cuenta(String numeroCuenta, String titular, BigDecimal saldoInicial) {
        this.numeroCuenta = numeroCuenta;
        this.titular = titular;
        this.saldo = saldoInicial;
        this.fechaApertura = LocalDate.now();
    }
    
    // Métodos de negocio
    public void depositar(BigDecimal monto) {
        if (monto.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("El monto debe ser positivo");
        }
        if (!activa) {
            throw new CuentaInactivaException("La cuenta está inactiva");
        }
        this.saldo = this.saldo.add(monto);
    }
    
    public void retirar(BigDecimal monto) {
        if (monto.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("El monto debe ser positivo");
        }
        if (!activa) {
            throw new CuentaInactivaException("La cuenta está inactiva");
        }
        if (this.saldo.compareTo(monto) < 0) {
            throw new SaldoInsuficienteException(
                "Saldo insuficiente. Disponible: " + this.saldo
            );
        }
        this.saldo = this.saldo.subtract(monto);
    }
    
    // Getters y setters
}

@Entity
@Table(name = "movimientos")
public class Movimiento {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "cuenta_id", nullable = false)
    private Cuenta cuenta;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private TipoMovimiento tipo;
    
    @Column(nullable = false, precision = 15, scale = 2)
    private BigDecimal monto;
    
    @Column(name = "saldo_resultante", nullable = false, precision = 15, scale = 2)
    private BigDecimal saldoResultante;
    
    @Column(length = 200)
    private String descripcion;
    
    @Column(name = "fecha_hora", nullable = false)
    private LocalDateTime fechaHora;
    
    @Column(name = "referencia_transferencia")
    private Long referenciaTransferencia;
    
    // Constructores y getters/setters
}

public enum TipoMovimiento {
    DEPOSITO,
    RETIRO,
    TRANSFERENCIA_ENVIADA,
    TRANSFERENCIA_RECIBIDA
}

### Paso 2: Repositorios

public interface CuentaRepository extends JpaRepository<Cuenta, Long> {
    
    Optional<Cuenta> findByNumeroCuenta(String numeroCuenta);
    
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT c FROM Cuenta c WHERE c.id = :id")
    Optional<Cuenta> findByIdConBloqueo(@Param("id") Long id);
    
    @Query("SELECT c FROM Cuenta c WHERE c.activa = true ORDER BY c.titular")
    List<Cuenta> findAllActivas();
}

public interface MovimientoRepository extends JpaRepository<Movimiento, Long> {
    
    List<Movimiento> findByCuentaIdOrderByFechaHoraDesc(Long cuentaId);
    
    @Query("SELECT m FROM Movimiento m WHERE m.cuenta.id = :cuentaId " +
           "AND m.fechaHora BETWEEN :inicio AND :fin ORDER BY m.fechaHora")
    List<Movimiento> findByCuentaAndPeriodo(
        @Param("cuentaId") Long cuentaId,
        @Param("inicio") LocalDateTime inicio,
        @Param("fin") LocalDateTime fin
    );
}

### Paso 3: Servicio de auditoría (transacción independiente)

@Service
public class AuditoriaBancariaService {
    
    @Autowired
    private AuditoriaRepository auditoriaRepository;
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void registrarOperacion(String operacion, String detalles, boolean exitosa) {
        AuditoriaLog log = new AuditoriaLog();
        log.setOperacion(operacion);
        log.setDetalles(detalles);
        log.setExitosa(exitosa);
        log.setFechaHora(LocalDateTime.now());
        log.setUsuario(obtenerUsuarioActual());
        
        auditoriaRepository.save(log);
        // Se guarda en transacción independiente
    }
    
    private String obtenerUsuarioActual() {
        // En una app real, obtendría del contexto de seguridad
        return "sistema";
    }
}

### Paso 4: Servicio principal con transacciones

@Service
@Transactional
public class OperacionesBancariasService {
    
    private static final Logger log = LoggerFactory.getLogger(OperacionesBancariasService.class);
    
    @Autowired
    private CuentaRepository cuentaRepository;
    
    @Autowired
    private MovimientoRepository movimientoRepository;
    
    @Autowired
    private AuditoriaBancariaService auditoriaService;
    
    public Cuenta abrirCuenta(String titular, BigDecimal depositoInicial) {
        String numeroCuenta = generarNumeroCuenta();
        
        Cuenta cuenta = new Cuenta(numeroCuenta, titular, depositoInicial);
        cuenta = cuentaRepository.save(cuenta);
        
        if (depositoInicial.compareTo(BigDecimal.ZERO) > 0) {
            registrarMovimiento(cuenta, TipoMovimiento.DEPOSITO, 
                depositoInicial, "Depósito inicial");
        }
        
        auditoriaService.registrarOperacion("APERTURA_CUENTA",
            "Cuenta: " + numeroCuenta + ", Titular: " + titular, true);
        
        log.info("Cuenta abierta: {} para {}", numeroCuenta, titular);
        return cuenta;
    }
    
    public void depositar(Long cuentaId, BigDecimal monto, String descripcion) {
        Cuenta cuenta = cuentaRepository.findByIdConBloqueo(cuentaId)
            .orElseThrow(() -> new EntityNotFoundException("Cuenta no encontrada"));
        
        cuenta.depositar(monto);
        cuentaRepository.save(cuenta);
        
        registrarMovimiento(cuenta, TipoMovimiento.DEPOSITO, monto, descripcion);
        
        auditoriaService.registrarOperacion("DEPOSITO",
            String.format("Cuenta: %s, Monto: %s", cuenta.getNumeroCuenta(), monto), true);
        
        log.info("Depósito de {} en cuenta {}", monto, cuenta.getNumeroCuenta());
    }
    
    public void retirar(Long cuentaId, BigDecimal monto, String descripcion) {
        Cuenta cuenta = cuentaRepository.findByIdConBloqueo(cuentaId)
            .orElseThrow(() -> new EntityNotFoundException("Cuenta no encontrada"));
        
        try {
            cuenta.retirar(monto);
            cuentaRepository.save(cuenta);
            
            registrarMovimiento(cuenta, TipoMovimiento.RETIRO, monto, descripcion);
            
            auditoriaService.registrarOperacion("RETIRO",
                String.format("Cuenta: %s, Monto: %s", cuenta.getNumeroCuenta(), monto), true);
            
            log.info("Retiro de {} de cuenta {}", monto, cuenta.getNumeroCuenta());
            
        } catch (SaldoInsuficienteException e) {
            auditoriaService.registrarOperacion("RETIRO_FALLIDO",
                String.format("Cuenta: %s, Monto: %s, Error: %s", 
                    cuenta.getNumeroCuenta(), monto, e.getMessage()), false);
            throw e;
        }
    }
    
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public Long transferir(Long cuentaOrigenId, Long cuentaDestinoId, 
                          BigDecimal monto, String concepto) {
        
        if (cuentaOrigenId.equals(cuentaDestinoId)) {
            throw new IllegalArgumentException("Las cuentas deben ser diferentes");
        }
        
        // Ordenar para evitar deadlocks
        Long primeraCuenta = Math.min(cuentaOrigenId, cuentaDestinoId);
        Long segundaCuenta = Math.max(cuentaOrigenId, cuentaDestinoId);
        
        Cuenta cuenta1 = cuentaRepository.findByIdConBloqueo(primeraCuenta)
            .orElseThrow(() -> new EntityNotFoundException("Cuenta no encontrada: " + primeraCuenta));
        Cuenta cuenta2 = cuentaRepository.findByIdConBloqueo(segundaCuenta)
            .orElseThrow(() -> new EntityNotFoundException("Cuenta no encontrada: " + segundaCuenta));
        
        Cuenta origen = cuentaOrigenId.equals(primeraCuenta) ? cuenta1 : cuenta2;
        Cuenta destino = cuentaDestinoId.equals(primeraCuenta) ? cuenta1 : cuenta2;
        
        try {
            // Generar referencia única para la transferencia
            Long referenciaTransferencia = System.currentTimeMillis();
            
            origen.retirar(monto);
            destino.depositar(monto);
            
            cuentaRepository.save(origen);
            cuentaRepository.save(destino);
            
            // Registrar movimientos
            Movimiento movOrigen = registrarMovimiento(origen, 
                TipoMovimiento.TRANSFERENCIA_ENVIADA, monto, 
                "Transferencia a " + destino.getNumeroCuenta() + ": " + concepto);
            movOrigen.setReferenciaTransferencia(referenciaTransferencia);
            
            Movimiento movDestino = registrarMovimiento(destino, 
                TipoMovimiento.TRANSFERENCIA_RECIBIDA, monto,
                "Transferencia de " + origen.getNumeroCuenta() + ": " + concepto);
            movDestino.setReferenciaTransferencia(referenciaTransferencia);
            
            auditoriaService.registrarOperacion("TRANSFERENCIA",
                String.format("%s -> %s: %s", origen.getNumeroCuenta(), 
                    destino.getNumeroCuenta(), monto), true);
            
            log.info("Transferencia de {} desde {} hacia {}", 
                monto, origen.getNumeroCuenta(), destino.getNumeroCuenta());
            
            return referenciaTransferencia;
            
        } catch (Exception e) {
            auditoriaService.registrarOperacion("TRANSFERENCIA_FALLIDA",
                String.format("%s -> %s: %s, Error: %s", 
                    origen.getNumeroCuenta(), destino.getNumeroCuenta(), 
                    monto, e.getMessage()), false);
            throw e;
        }
    }
    
    @Transactional(readOnly = true)
    public BigDecimal consultarSaldo(Long cuentaId) {
        return cuentaRepository.findById(cuentaId)
            .map(Cuenta::getSaldo)
            .orElseThrow(() -> new EntityNotFoundException("Cuenta no encontrada"));
    }
    
    @Transactional(readOnly = true)
    public List<Movimiento> obtenerMovimientos(Long cuentaId) {
        return movimientoRepository.findByCuentaIdOrderByFechaHoraDesc(cuentaId);
    }
    
    @Transactional(readOnly = true, isolation = Isolation.REPEATABLE_READ)
    public EstadoCuenta generarEstadoCuenta(Long cuentaId, YearMonth periodo) {
        Cuenta cuenta = cuentaRepository.findById(cuentaId)
            .orElseThrow(() -> new EntityNotFoundException("Cuenta no encontrada"));
        
        LocalDateTime inicio = periodo.atDay(1).atStartOfDay();
        LocalDateTime fin = periodo.atEndOfMonth().atTime(23, 59, 59);
        
        List<Movimiento> movimientos = movimientoRepository
            .findByCuentaAndPeriodo(cuentaId, inicio, fin);
        
        return new EstadoCuenta(cuenta, movimientos, periodo);
    }
    
    private Movimiento registrarMovimiento(Cuenta cuenta, TipoMovimiento tipo, 
                                           BigDecimal monto, String descripcion) {
        Movimiento movimiento = new Movimiento();
        movimiento.setCuenta(cuenta);
        movimiento.setTipo(tipo);
        movimiento.setMonto(monto);
        movimiento.setSaldoResultante(cuenta.getSaldo());
        movimiento.setDescripcion(descripcion);
        movimiento.setFechaHora(LocalDateTime.now());
        
        return movimientoRepository.save(movimiento);
    }
    
    private String generarNumeroCuenta() {
        return "ES" + String.format("%018d", System.nanoTime() % 1000000000000000000L);
    }
}

### Paso 5: Excepciones personalizadas

public class SaldoInsuficienteException extends RuntimeException {
    public SaldoInsuficienteException(String mensaje) {
        super(mensaje);
    }
}

public class CuentaInactivaException extends RuntimeException {
    public CuentaInactivaException(String mensaje) {
        super(mensaje);
    }
}

public class ConcurrenciaException extends RuntimeException {
    public ConcurrenciaException(String mensaje) {
        super(mensaje);
    }
}

### Paso 6: Aplicación de demostración

@SpringBootApplication
public class BancoTransaccionesApp implements CommandLineRunner {
    
    private static final Logger log = LoggerFactory.getLogger(BancoTransaccionesApp.class);
    
    @Autowired
    private OperacionesBancariasService bancarioService;
    
    public static void main(String[] args) {
        SpringApplication.run(BancoTransaccionesApp.class, args);
    }
    
    @Override
    public void run(String... args) {
        log.info("=== DEMO: Sistema Bancario con Transacciones ===\n");
        
        // 1. Abrir cuentas
        log.info("1. Abriendo cuentas...");
        Cuenta cuenta1 = bancarioService.abrirCuenta("Juan García", new BigDecimal("1000.00"));
        Cuenta cuenta2 = bancarioService.abrirCuenta("María López", new BigDecimal("500.00"));
        
        log.info("   Cuenta 1: {} - Saldo: {}", cuenta1.getNumeroCuenta(), cuenta1.getSaldo());
        log.info("   Cuenta 2: {} - Saldo: {}", cuenta2.getNumeroCuenta(), cuenta2.getSaldo());
        
        // 2. Realizar depósito
        log.info("\n2. Depósito de 250€ en cuenta 1...");
        bancarioService.depositar(cuenta1.getId(), new BigDecimal("250.00"), "Ingreso en efectivo");
        log.info("   Nuevo saldo cuenta 1: {}", bancarioService.consultarSaldo(cuenta1.getId()));
        
        // 3. Realizar transferencia
        log.info("\n3. Transferencia de 300€ de cuenta 1 a cuenta 2...");
        Long refTransferencia = bancarioService.transferir(
            cuenta1.getId(), cuenta2.getId(), 
            new BigDecimal("300.00"), "Pago alquiler"
        );
        log.info("   Referencia: {}", refTransferencia);
        log.info("   Saldo cuenta 1: {}", bancarioService.consultarSaldo(cuenta1.getId()));
        log.info("   Saldo cuenta 2: {}", bancarioService.consultarSaldo(cuenta2.getId()));
        
        // 4. Intentar retiro con saldo insuficiente
        log.info("\n4. Intentando retirar 2000€ de cuenta 1 (debe fallar)...");
        try {
            bancarioService.retirar(cuenta1.getId(), new BigDecimal("2000.00"), "Retiro grande");
        } catch (SaldoInsuficienteException e) {
            log.info("   Error esperado: {}", e.getMessage());
        }
        
        // 5. Mostrar movimientos
        log.info("\n5. Movimientos de cuenta 1:");
        List<Movimiento> movimientos = bancarioService.obtenerMovimientos(cuenta1.getId());
        for (Movimiento mov : movimientos) {
            log.info("   {} | {} | {} | Saldo: {}", 
                mov.getFechaHora().format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm")),
                mov.getTipo(),
                mov.getMonto(),
                mov.getSaldoResultante()
            );
        }
        
        log.info("\n=== FIN DE LA DEMO ===");
    }
}

## 11. Ejercicios propuestos

### Ejercicio 1: Gestión de inventario con transacciones (Nivel básico)

Implementa un sistema de gestión de inventario con las siguientes operaciones transaccionales:

1. `agregarStock(Long productoId, int cantidad)`: Añadir stock a un producto
    
2. `venderProducto(Long productoId, int cantidad)`: Reducir stock al vender
    
3. `transferirStock(Long origenId, Long destinoId, int cantidad)`: Mover stock entre almacenes
    

Requisitos:

- Usar bloqueo optimista con `@Version`
    
- Validar que no quede stock negativo
    
- Registrar todos los movimientos de stock
    

### Ejercicio 2: Sistema de reservas con propagación (Nivel intermedio)

Crea un sistema de reservas de hotel que demuestre el uso de diferentes modos de propagación:

1. Servicio de reservas principal con `Propagation.REQUIRED`
    
2. Servicio de notificaciones con `Propagation.REQUIRES_NEW` (debe enviar email aunque falle la reserva)
    
3. Servicio de facturación con `Propagation.MANDATORY`
    

Implementar:

- `reservarHabitacion(Long habitacionId, LocalDate desde, LocalDate hasta, Long clienteId)`
    
- `cancelarReserva(Long reservaId)`
    

### Ejercicio 3: Venta de entradas con bloqueo pesimista (Nivel intermedio)

Implementa un sistema de venta de entradas para un evento:

1. Entidad `Asiento` con campo `@Version`
    
2. Método `reservarAsiento(Long asientoId, String cliente)` que:
    

- Verifique disponibilidad
    
- Reserve el asiento
    
- Maneje `OptimisticLockException` con reintentos
    

Requisitos:

- Implementar lógica de reintento (máximo 3 intentos)
    
- Registrar cada intento en logs
    

### Ejercicio 4: Niveles de aislamiento (Nivel avanzado)

Crea un sistema de subastas con diferentes niveles de aislamiento:

1. `obtenerPujaActual(Long subastaId)`: READ_COMMITTED, solo lectura
    
2. `realizarPuja(Long subastaId, BigDecimal monto)`: REPEATABLE_READ
    
3. `cerrarSubasta(Long subastaId)`: SERIALIZABLE
    

Requisitos:

- Validar que la nueva puja sea mayor que la actual
    
- Manejar concurrencia apropiadamente
    
- Documentar por qué se eligió cada nivel de aislamiento
    

### Ejercicio 5: Transacciones programáticas (Nivel avanzado)

Implementa un servicio de importación masiva de datos:

1. Usar `TransactionTemplate` para control granular
    
2. Procesar registros en lotes de 100
    
3. Hacer commit parcial por lote
    
4. Registrar estadísticas de éxito/fallo
    

Requisitos:

- Si un lote falla, los anteriores deben mantenerse
    
- Generar reporte final con totales
    

## 12. Resumen

### Conceptos clave

1. **Propiedades ACID**: Atomicidad, Consistencia, Aislamiento, Durabilidad
    
2. **Transacciones declarativas**: Usar `@Transactional` de Spring
    
3. **Propagación**: Cómo se comportan las transacciones anidadas
    
4. **Aislamiento**: Control de concurrencia entre transacciones
    
5. **Bloqueos**: Optimista (versión) vs Pesimista (SELECT FOR UPDATE)
    
6. **Rollback**: Automático para RuntimeException, configurable para otras
    

### Mejores prácticas

1. Colocar `@Transactional` en la capa de servicio
    
2. Mantener transacciones lo más cortas posible
    
3. Usar `readOnly = true` para consultas
    
4. Evitar auto-invocación de métodos transaccionales
    
5. No capturar excepciones que deben provocar rollback
    
6. Usar el nivel de aislamiento mínimo necesario
    
7. Preferir bloqueo optimista salvo alta concurrencia
    

### Tabla de referencia rápida

|Escenario|Configuración recomendada|
|---|---|
|Consulta simple|`@Transactional(readOnly = true)`|
|Operación CRUD|`@Transactional` (por defecto)|
|Operación crítica|`@Transactional(isolation = SERIALIZABLE)`|
|Auditoría independiente|`@Transactional(propagation = REQUIRES_NEW)`|
|Alta concurrencia|Bloqueo optimista con `@Version`|
|Baja concurrencia crítica|Bloqueo pesimista con `@Lock`|

## Referencias

- Documentación oficial Spring Transaction: [https://docs.spring.io/spring-framework/reference/data-access/transaction.html](https://docs.spring.io/spring-framework/reference/data-access/transaction.html)
    
- Jakarta Persistence API Specification: [https://jakarta.ee/specifications/persistence/](https://jakarta.ee/specifications/persistence/)
    
- Hibernate ORM Documentation: [https://hibernate.org/orm/documentation/](https://hibernate.org/orm/documentation/)