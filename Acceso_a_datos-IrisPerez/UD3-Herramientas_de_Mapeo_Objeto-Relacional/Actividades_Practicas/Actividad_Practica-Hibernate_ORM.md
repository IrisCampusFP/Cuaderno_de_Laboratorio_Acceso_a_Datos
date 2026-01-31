## Objetivos

- Configurar Hibernate como proveedor JPA
    
- Usar caracteristicas especificas de Hibernate
    
- Aplicar optimizaciones de rendimiento
    
- Trabajar con Session y SessionFactory

## Ejercicio 1: Configuracion Hibernate

### Enunciado

Configura un proyecto Hibernate standalone (sin Spring) con:

- hibernate.cfg.xml para MySQL
    
- Pool de conexiones HikariCP
    
- Dialecto MySQL 8
    
- Validacion de esquema

Ver solucion

**hibernate.cfg.xml:**

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- Conexion MySQL -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/miapp?useSSL=false&amp;serverTimezone=UTC</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">password</property>
        
        <!-- Dialecto MySQL 8 -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQL8Dialect</property>
        
        <!-- Pool HikariCP -->
        <property name="hibernate.connection.provider_class">org.hibernate.hikaricp.internal.HikariCPConnectionProvider</property>
        <property name="hibernate.hikari.minimumIdle">5</property>
        <property name="hibernate.hikari.maximumPoolSize">20</property>
        <property name="hibernate.hikari.idleTimeout">30000</property>
        
        <!-- Esquema -->
        <property name="hibernate.hbm2ddl.auto">validate</property>
        
        <!-- SQL y formato -->
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        <property name="hibernate.use_sql_comments">true</property>
        
        <!-- Entidades -->
        <mapping class="com.ejemplo.Producto"/>
        <mapping class="com.ejemplo.Categoria"/>
    </session-factory>
</hibernate-configuration>

**HibernateUtil.java:**

public class HibernateUtil {
    private static SessionFactory sessionFactory;
    
    static {
        try {
            sessionFactory = new Configuration()
                .configure("hibernate.cfg.xml")
                .buildSessionFactory();
        } catch (Exception e) {
            throw new ExceptionInInitializerError(e);
        }
    }
    
    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
    
    public static Session openSession() {
        return sessionFactory.openSession();
    }
    
    public static void shutdown() {
        if (sessionFactory != null) {
            sessionFactory.close();
        }
    }
}

## Ejercicio 2: Session vs EntityManager

### Enunciado

Implementa el mismo metodo de busqueda usando: a) API de Session nativa de Hibernate b) API de EntityManager (JPA)

El metodo debe buscar productos por nombre usando LIKE.

Ver solucion

**a) Con Session (Hibernate nativo):**

public List<Producto> buscarPorNombreSession(String nombre) {
    try (Session session = HibernateUtil.openSession()) {
        // HQL (Hibernate Query Language)
        String hql = "FROM Producto p WHERE p.nombre LIKE :nombre";
        
        Query<Producto> query = session.createQuery(hql, Producto.class);
        query.setParameter("nombre", "%" + nombre + "%");
        
        return query.getResultList();
    }
}

// Con Criteria API de Hibernate
public List<Producto> buscarPorNombreCriteria(String nombre) {
    try (Session session = HibernateUtil.openSession()) {
        CriteriaBuilder cb = session.getCriteriaBuilder();
        CriteriaQuery<Producto> cq = cb.createQuery(Producto.class);
        Root<Producto> root = cq.from(Producto.class);
        
        cq.select(root)
          .where(cb.like(cb.lower(root.get("nombre")), 
                        "%" + nombre.toLowerCase() + "%"));
        
        return session.createQuery(cq).getResultList();
    }
}

**b) Con EntityManager (JPA estandar):**

@PersistenceContext
private EntityManager em;

public List<Producto> buscarPorNombreJPA(String nombre) {
    // JPQL (muy similar a HQL)
    String jpql = "SELECT p FROM Producto p WHERE p.nombre LIKE :nombre";
    
    TypedQuery<Producto> query = em.createQuery(jpql, Producto.class);
    query.setParameter("nombre", "%" + nombre + "%");
    
    return query.getResultList();
}

// Acceder a Session desde EntityManager
public List<Producto> buscarUsandoSessionDesdeEM(String nombre) {
    Session session = em.unwrap(Session.class);
    // Ahora puedes usar API de Hibernate
    return session.createQuery("FROM Producto WHERE nombre LIKE :n", Producto.class)
                  .setParameter("n", "%" + nombre + "%")
                  .list();
}

**Diferencias principales:**

|Aspecto|Session (Hibernate)|EntityManager (JPA)|
|---|---|---|
|Lenguaje|HQL|JPQL|
|Metodo query|createQuery()|createQuery()|
|Obtener resultados|list() / getResultList()|getResultList()|
|Persistir|save() / persist()|persist()|
|Actualizar|update() / merge()|merge()|
|Eliminar|delete()|remove()|

## Ejercicio 3: Batch Processing

### Enunciado

Implementa un metodo que inserte 10,000 productos de forma eficiente:

- Configura el tamano de batch
    
- Limpia la sesion periodicamente
    
- Mide el tiempo de ejecucion
    

Ver solucion

**Configuracion en hibernate.cfg.xml:**

<property name="hibernate.jdbc.batch_size">50</property>
<property name="hibernate.order_inserts">true</property>
<property name="hibernate.order_updates">true</property>
<property name="hibernate.batch_versioned_data">true</property>

**Implementacion con batch:**

public class BatchInsertService {
    
    private static final int BATCH_SIZE = 50;
    
    public void insertarProductosMasivo(int cantidad) {
        long inicio = System.currentTimeMillis();
        
        Session session = HibernateUtil.openSession();
        Transaction tx = session.beginTransaction();
        
        try {
            for (int i = 0; i < cantidad; i++) {
                Producto producto = new Producto();
                producto.setNombre("Producto " + i);
                producto.setPrecio(new BigDecimal(Math.random() * 1000));
                producto.setStock((int) (Math.random() * 100));
                
                session.persist(producto);
                
                // Flush y clear cada BATCH_SIZE registros
                if (i > 0 && i % BATCH_SIZE == 0) {
                    session.flush();  // Ejecuta INSERTs pendientes
                    session.clear();  // Libera memoria
                    System.out.println("Procesados: " + i);
                }
            }
            
            tx.commit();
            
        } catch (Exception e) {
            tx.rollback();
            throw e;
        } finally {
            session.close();
        }
        
        long fin = System.currentTimeMillis();
        System.out.println("Insertados " + cantidad + " en " + (fin - inicio) + "ms");
    }
    
    // Alternativa con StatelessSession (mas rapido, sin cache)
    public void insertarConStateless(int cantidad) {
        long inicio = System.currentTimeMillis();
        
        StatelessSession session = HibernateUtil.getSessionFactory().openStatelessSession();
        Transaction tx = session.beginTransaction();
        
        try {
            for (int i = 0; i < cantidad; i++) {
                Producto producto = new Producto();
                producto.setNombre("Producto " + i);
                producto.setPrecio(new BigDecimal(Math.random() * 1000));
                
                session.insert(producto);
            }
            
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
            throw e;
        } finally {
            session.close();
        }
        
        long fin = System.currentTimeMillis();
        System.out.println("Insertados " + cantidad + " en " + (fin - inicio) + "ms");
    }
}

**Comparativa de rendimiento tipica:**

|Metodo|10,000 registros|
|---|---|
|Sin batch|~30 segundos|
|Con batch (50)|~3 segundos|
|StatelessSession|~1.5 segundos|

## Ejercicio 4: Natural ID

### Enunciado

Implementa una entidad `Usuario` con:

- ID tecnico autogenerado
    
- Natural ID: email (inmutable)
    
- Metodo de busqueda por natural ID
    

Ver solucion

**Entidad con Natural ID:**

@Entity
@Table(name = "usuarios")
@NaturalIdCache  // Cache para Natural IDs
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Usuario {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NaturalId
    @Column(nullable = false, unique = true, updatable = false)
    private String email;
    
    private String nombre;
    private String password;
    
    public Usuario() {}
    
    public Usuario(String email, String nombre) {
        this.email = email;
        this.nombre = nombre;
    }
    
    // equals y hashCode basados en Natural ID
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Usuario)) return false;
        Usuario that = (Usuario) o;
        return email != null && email.equals(that.email);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(email);
    }
    
    // getters, setters...
}

**Busqueda por Natural ID:**

public class UsuarioRepository {
    
    public Optional<Usuario> findByEmail(String email) {
        try (Session session = HibernateUtil.openSession()) {
            return session.byNaturalId(Usuario.class)
                          .using("email", email)
                          .loadOptional();
        }
    }
    
    // Con referencia (solo carga proxy, no ejecuta SELECT)
    public Usuario getReferenceByEmail(String email) {
        try (Session session = HibernateUtil.openSession()) {
            return session.byNaturalId(Usuario.class)
                          .using("email", email)
                          .getReference();
        }
    }
}

**Ventajas de Natural ID:**

- Cache especifica mas eficiente
    
- Busqueda sin conocer el ID tecnico
    
- Semantica de negocio clara
    
- Inmutabilidad garantizada
    

## Ejercicio 5: Filtros Hibernate

### Enunciado

Implementa filtros dinamicos para:

- Filtrar productos por rango de precio
    
- Activar/desactivar el filtro segun el contexto


Ver solucion

**Definicion del filtro en la entidad:**

@Entity
@Table(name = "productos")
@FilterDef(
    name = "rangoPrecio",
    parameters = {
        @ParamDef(name = "precioMin", type = BigDecimal.class),
        @ParamDef(name = "precioMax", type = BigDecimal.class)
    }
)
@Filter(
    name = "rangoPrecio",
    condition = "precio >= :precioMin AND precio <= :precioMax"
)
@FilterDef(name = "soloActivos")
@Filter(name = "soloActivos", condition = "activo = true")
public class Producto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private BigDecimal precio;
    private Boolean activo;
    
    // ...
}

**Uso de filtros:**

@Service
public class ProductoService {
    
    @PersistenceContext
    private EntityManager em;
    
    public List<Producto> buscarConFiltros(BigDecimal min, BigDecimal max, 
                                           boolean soloActivos) {
        Session session = em.unwrap(Session.class);
        
        // Activar filtro de precio
        if (min != null && max != null) {
            session.enableFilter("rangoPrecio")
                   .setParameter("precioMin", min)
                   .setParameter("precioMax", max);
        }
        
        // Activar filtro de activos
        if (soloActivos) {
            session.enableFilter("soloActivos");
        }
        
        // La consulta aplicara los filtros automaticamente
        return em.createQuery("FROM Producto ORDER BY nombre", Producto.class)
                 .getResultList();
    }
    
    public void desactivarFiltros() {
        Session session = em.unwrap(Session.class);
        session.disableFilter("rangoPrecio");
        session.disableFilter("soloActivos");
    }
}

**SQL generado con filtros activos:**

SELECT * FROM productos 
WHERE precio >= ? AND precio <= ?  -- filtro rangoPrecio
AND activo = true                   -- filtro soloActivos
ORDER BY nombre

## Reto adicional

Implementa un interceptor de Hibernate que:

- Registre todas las operaciones de INSERT, UPDATE, DELETE
    
- Guarde en una tabla de auditoria quien, cuando y que cambio


Consejo

Extiende `EmptyInterceptor` e implementa los metodos `onSave`, `onFlushDirty`, `onDelete`.