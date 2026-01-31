## Objetivos

- Comprender los estados de una entidad JPA
    
- Implementar callbacks del ciclo de vida
    
- Usar EntityListeners para auditoria
    
- Gestionar correctamente las transiciones de estado

## Ejercicio 1: Identificar estados de entidad

### Enunciado

Dado el siguiente codigo, indica en que estado (NEW, MANAGED, DETACHED, REMOVED) se encuentra la entidad `producto` en cada punto marcado con comentario:

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

Producto producto = new Producto("Laptop", 999.99);
// PUNTO A: Estado del producto?

em.persist(producto);
// PUNTO B: Estado del producto?

em.getTransaction().commit();
// PUNTO C: Estado del producto?

em.close();
// PUNTO D: Estado del producto?

EntityManager em2 = emf.createEntityManager();
Producto prod2 = em2.find(Producto.class, producto.getId());
// PUNTO E: Estado de prod2?

em2.getTransaction().begin();
em2.remove(prod2);
// PUNTO F: Estado de prod2?

em2.getTransaction().commit();
em2.close();

Ver solucion

|Punto|Estado|Explicacion|
|---|---|---|
|A|**NEW (Transient)**|Objeto recien creado con `new`, no asociado a ningun EntityManager|
|B|**MANAGED**|Despues de `persist()`, la entidad esta gestionada por el EntityManager|
|C|**MANAGED**|El commit sincroniza con la BD pero la entidad sigue gestionada|
|D|**DETACHED**|Al cerrar el EntityManager, todas sus entidades pasan a detached|
|E|**MANAGED**|`find()` devuelve una entidad gestionada por em2|
|F|**REMOVED**|Marcada para eliminacion, aun en memoria pero sera borrada en commit|

**Diagrama de transiciones:**

     new()
        |
        v
      [NEW] --persist()--> [MANAGED] <--merge()-- [DETACHED]
                              |                       ^
                              |                       |
                         remove()              close()/clear()
                              |                       |
                              v                       |
                          [REMOVED]                   |
                              |                       |
                           commit()                   |
                              +-----------------------+

## Ejercicio 2: Callbacks del ciclo de vida

### Enunciado

Implementa una entidad `Pedido` que:

- Establezca automaticamente la fecha de creacion al persistir
    
- Actualice la fecha de modificacion antes de cada update
    
- Valide que el total sea positivo antes de persistir/actualizar
    
- Registre en consola cuando se cargue desde la base de datos

Usa las anotaciones de callback: `@PrePersist`, `@PreUpdate`, `@PostLoad`

Ver solucion

@Entity
@Table(name = "pedidos")
public class Pedido {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String cliente;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal total;
    
    @Column(name = "fecha_creacion", updatable = false)
    private LocalDateTime fechaCreacion;
    
    @Column(name = "fecha_modificacion")
    private LocalDateTime fechaModificacion;
    
    @Enumerated(EnumType.STRING)
    private EstadoPedido estado = EstadoPedido.PENDIENTE;
    
    // Callback antes de INSERT
    @PrePersist
    protected void onCreate() {
        this.fechaCreacion = LocalDateTime.now();
        this.fechaModificacion = LocalDateTime.now();
        validarTotal();
        System.out.println("[PrePersist] Creando pedido para: " + cliente);
    }
    
    // Callback antes de UPDATE
    @PreUpdate
    protected void onUpdate() {
        this.fechaModificacion = LocalDateTime.now();
        validarTotal();
        System.out.println("[PreUpdate] Actualizando pedido #" + id);
    }
    
    // Callback despues de cargar de BD
    @PostLoad
    protected void onLoad() {
        System.out.println("[PostLoad] Pedido #" + id + " cargado desde BD");
    }
    
    // Callback despues de persistir
    @PostPersist
    protected void afterPersist() {
        System.out.println("[PostPersist] Pedido #" + id + " guardado exitosamente");
    }
    
    // Callback antes de eliminar
    @PreRemove
    protected void onRemove() {
        System.out.println("[PreRemove] Eliminando pedido #" + id);
    }
    
    // Metodo de validacion
    private void validarTotal() {
        if (total != null && total.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalStateException("El total no puede ser negativo");
        }
    }
    
    // Constructores
    public Pedido() {}
    
    public Pedido(String cliente, BigDecimal total) {
        this.cliente = cliente;
        this.total = total;
    }
    
    // Getters y setters...
}

enum EstadoPedido {
    PENDIENTE, PROCESANDO, ENVIADO, ENTREGADO, CANCELADO
}

**Anotaciones de callback disponibles:**

|Anotacion|Momento de ejecucion|
|---|---|
|@PrePersist|Antes de INSERT|
|@PostPersist|Despues de INSERT|
|@PreUpdate|Antes de UPDATE|
|@PostUpdate|Despues de UPDATE|
|@PreRemove|Antes de DELETE|
|@PostRemove|Despues de DELETE|
|@PostLoad|Despues de SELECT|

## Ejercicio 3: EntityListener para auditoria

### Enunciado

Crea un sistema de auditoria reutilizable:

1. Una clase `Auditable` con campos: `creadoPor`, `fechaCreacion`, `modificadoPor`, `fechaModificacion`
    
2. Un `AuditoriaListener` que rellene estos campos automaticamente
    
3. Aplica el listener a las entidades `Pedido` y `Producto`


Simula el usuario actual con una clase estatica `UsuarioActual`.

Ver solucion

**Clase para simular usuario actual:**

public class UsuarioActual {
    private static String usuario = "sistema";
    
    public static String getUsuario() {
        return usuario;
    }
    
    public static void setUsuario(String usuario) {
        UsuarioActual.usuario = usuario;
    }
}

**Interface Auditable (o clase @MappedSuperclass):**

@MappedSuperclass
public abstract class Auditable {
    
    @Column(name = "creado_por", updatable = false)
    private String creadoPor;
    
    @Column(name = "fecha_creacion", updatable = false)
    private LocalDateTime fechaCreacion;
    
    @Column(name = "modificado_por")
    private String modificadoPor;
    
    @Column(name = "fecha_modificacion")
    private LocalDateTime fechaModificacion;
    
    // Getters y setters
    public String getCreadoPor() { return creadoPor; }
    public void setCreadoPor(String creadoPor) { this.creadoPor = creadoPor; }
    
    public LocalDateTime getFechaCreacion() { return fechaCreacion; }
    public void setFechaCreacion(LocalDateTime fechaCreacion) { 
        this.fechaCreacion = fechaCreacion; 
    }
    
    public String getModificadoPor() { return modificadoPor; }
    public void setModificadoPor(String modificadoPor) { 
        this.modificadoPor = modificadoPor; 
    }
    
    public LocalDateTime getFechaModificacion() { return fechaModificacion; }
    public void setFechaModificacion(LocalDateTime fechaModificacion) { 
        this.fechaModificacion = fechaModificacion; 
    }
}

**EntityListener de auditoria:**

public class AuditoriaListener {
    
    @PrePersist
    public void prePersist(Auditable entidad) {
        LocalDateTime ahora = LocalDateTime.now();
        String usuario = UsuarioActual.getUsuario();
        
        entidad.setCreadoPor(usuario);
        entidad.setFechaCreacion(ahora);
        entidad.setModificadoPor(usuario);
        entidad.setFechaModificacion(ahora);
    }
    
    @PreUpdate
    public void preUpdate(Auditable entidad) {
        entidad.setModificadoPor(UsuarioActual.getUsuario());
        entidad.setFechaModificacion(LocalDateTime.now());
    }
}

**Entidad que usa el listener:**

@Entity
@Table(name = "pedidos")
@EntityListeners(AuditoriaListener.class)
public class Pedido extends Auditable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String cliente;
    private BigDecimal total;
    
    // ... resto de la entidad
}

@Entity
@Table(name = "productos")
@EntityListeners(AuditoriaListener.class)
public class Producto extends Auditable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private BigDecimal precio;
    
    // ... resto de la entidad
}

**Uso:**

UsuarioActual.setUsuario("admin");

Pedido pedido = new Pedido("Cliente 1", new BigDecimal("100.00"));
em.persist(pedido);

// pedido.getCreadoPor() -> "admin"
// pedido.getFechaCreacion() -> fecha actual

## Ejercicio 4: Merge vs Persist

### Enunciado

Explica que ocurre en cada caso y cual es la diferencia:

**Caso 1:**

Producto p = new Producto("Laptop", 999.99);
em.persist(p);

**Caso 2:**

Producto p = new Producto("Laptop", 999.99);
Producto pManaged = em.merge(p);

**Caso 3:**

Producto p = em.find(Producto.class, 1L);
em.detach(p);
p.setPrecio(899.99);
em.merge(p);

Ver solucion

**Caso 1: persist() con entidad nueva**

Producto p = new Producto("Laptop", 999.99);  // Estado: NEW
em.persist(p);                                 // Estado: MANAGED
// p ahora esta gestionado
// Se generara INSERT al hacer commit
// p.getId() tendra valor despues del persist (si es IDENTITY)

- `persist()` hace que la misma instancia pase a MANAGED
    
- La entidad debe ser NEW (sin ID o con ID no existente)
    

---

**Caso 2: merge() con entidad nueva**

Producto p = new Producto("Laptop", 999.99);  // Estado: NEW
Producto pManaged = em.merge(p);               // pManaged: MANAGED, p: sigue NEW
// p NO esta gestionado, pManaged SI
// Los cambios deben hacerse en pManaged

- `merge()` devuelve una COPIA gestionada
    
- La instancia original sigue siendo NEW/DETACHED
    
- Hay que usar la instancia retornada
    

---

**Caso 3: merge() con entidad detached**

Producto p = em.find(Producto.class, 1L);  // Estado: MANAGED
em.detach(p);                               // Estado: DETACHED
p.setPrecio(899.99);                        // Modificacion en detached (no se detecta)
Producto pActualizado = em.merge(p);        // Estado: MANAGED (copia)
// pActualizado tiene el nuevo precio y esta gestionado
// Se generara UPDATE al hacer commit

- `merge()` copia el estado de la entidad detached a una gestionada
    
- Si ya existe en el contexto, actualiza esa; si no, carga de BD
    

---

**Tabla comparativa:**

|Aspecto|persist()|merge()|
|---|---|---|
|Entidad original|Pasa a MANAGED|Sigue igual|
|Retorna|void|Entidad gestionada|
|Con entidad NEW|Inserta|Inserta|
|Con entidad DETACHED|Error|Actualiza|
|Con entidad MANAGED|No hace nada|No hace nada|
|Uso tipico|Crear nuevas|Actualizar/reattach|

**Regla de oro:**

- Usa `persist()` para entidades nuevas
    
- Usa `merge()` para reattach entidades detached
    
- Siempre usa el retorno de `merge()`

## Ejercicio 5: Refresh y sincronizacion

### Enunciado

Implementa un metodo que:

1. Cargue un producto de la base de datos
    
2. Modifique su precio localmente
    
3. Simule que otro proceso modifica el mismo producto en BD
    
4. Use `refresh()` para recargar el estado desde BD
    
5. Muestre que los cambios locales se pierden

Usa una segunda conexion para simular el cambio concurrente.

Ver solucion

public class EjemploRefresh {
    
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("test-pu");
        
        // Crear producto inicial
        EntityManager em1 = emf.createEntityManager();
        em1.getTransaction().begin();
        Producto producto = new Producto("Monitor", new BigDecimal("299.99"));
        em1.persist(producto);
        em1.getTransaction().commit();
        Long productoId = producto.getId();
        System.out.println("Producto creado con ID: " + productoId);
        
        // Conexion 1: Cargar y modificar localmente
        System.out.println("\n--- Conexion 1: Carga y modifica ---");
        Producto p1 = em1.find(Producto.class, productoId);
        System.out.println("Precio original: " + p1.getPrecio());
        
        p1.setPrecio(new BigDecimal("349.99"));
        System.out.println("Precio modificado localmente: " + p1.getPrecio());
        // NO hacemos commit todavia
        
        // Conexion 2: Otro proceso modifica en BD
        System.out.println("\n--- Conexion 2: Modificacion concurrente ---");
        EntityManager em2 = emf.createEntityManager();
        em2.getTransaction().begin();
        Producto p2 = em2.find(Producto.class, productoId);
        p2.setPrecio(new BigDecimal("199.99"));  // Rebaja!
        em2.getTransaction().commit();
        System.out.println("Otro proceso cambio precio a: " + p2.getPrecio());
        em2.close();
        
        // Conexion 1: Verificar estado local vs BD
        System.out.println("\n--- Conexion 1: Antes de refresh ---");
        System.out.println("Precio en memoria: " + p1.getPrecio());  // 349.99
        
        // Refresh para sincronizar con BD
        System.out.println("\n--- Ejecutando refresh() ---");
        em1.refresh(p1);
        
        System.out.println("\n--- Conexion 1: Despues de refresh ---");
        System.out.println("Precio sincronizado: " + p1.getPrecio());  // 199.99
        
        em1.close();
        emf.close();
    }
}

**Salida:**

Producto creado con ID: 1

--- Conexion 1: Carga y modifica ---
Precio original: 299.99
Precio modificado localmente: 349.99

--- Conexion 2: Modificacion concurrente ---
Otro proceso cambio precio a: 199.99

--- Conexion 1: Antes de refresh ---
Precio en memoria: 349.99

--- Ejecutando refresh() ---

--- Conexion 1: Despues de refresh ---
Precio sincronizado: 199.99

**Conceptos clave:**

- `refresh()` recarga el estado desde la BD, descartando cambios locales
    
- Util cuando sospechas que los datos pueden haber cambiado
    
- Lanza `EntityNotFoundException` si la entidad fue eliminada
    
- Solo funciona con entidades MANAGED

## Reto adicional

Implementa un sistema de «soft delete» usando callbacks:

- En lugar de eliminar fisicamente, marca la entidad como `eliminado = true` y guarda `fechaEliminacion`
    
- Usa `@PreRemove` para interceptar la eliminacion
    
- Lanza una excepcion para cancelar el delete real
    
- Modifica todas las consultas para filtrar los registros eliminados

Consejo

Puedes usar `@Where(clause = "eliminado = false")` de Hibernate para filtrar automaticamente.