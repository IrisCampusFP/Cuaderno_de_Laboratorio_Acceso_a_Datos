# Actividad Practica: Tecnicas Avanzadas JPA

## Objetivos

- Implementar estrategias de herencia
    
- Configurar cache de segundo nivel
    
- Aplicar auditoria automatica
    
- Optimizar rendimiento con EntityGraph

## Ejercicio 1: Herencia SINGLE_TABLE

### Enunciado

Modela una jerarquia de empleados:

- `Empleado` (clase base): id, nombre, salarioBase
    
- `Desarrollador`: lenguajePrincipal, nivelSenioridad
    
- `Gerente`: departamento, presupuesto

Usa estrategia SINGLE_TABLE con discriminador.

Ver solucion

@Entity
@Table(name = "empleados")
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(
    name = "tipo_empleado",
    discriminatorType = DiscriminatorType.STRING
)
public abstract class Empleado {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String nombre;
    
    @Column(name = "salario_base", precision = 10, scale = 2)
    private BigDecimal salarioBase;
    
    // Metodo abstracto para calcular salario total
    public abstract BigDecimal calcularSalarioTotal();
    
    // Constructores, getters, setters...
}

@Entity
@DiscriminatorValue("DEV")
public class Desarrollador extends Empleado {
    
    @Column(name = "lenguaje_principal")
    private String lenguajePrincipal;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "nivel_senioridad")
    private NivelSenioridad nivelSenioridad;
    
    @Override
    public BigDecimal calcularSalarioTotal() {
        BigDecimal bonus = switch (nivelSenioridad) {
            case JUNIOR -> new BigDecimal("0.10");
            case MID -> new BigDecimal("0.20");
            case SENIOR -> new BigDecimal("0.35");
        };
        return getSalarioBase().multiply(BigDecimal.ONE.add(bonus));
    }
}

@Entity
@DiscriminatorValue("MGR")
public class Gerente extends Empleado {
    
    private String departamento;
    
    @Column(precision = 12, scale = 2)
    private BigDecimal presupuesto;
    
    @Override
    public BigDecimal calcularSalarioTotal() {
        // Bonus del 5% del presupuesto (max 50% del salario)
        BigDecimal bonusPresupuesto = presupuesto.multiply(new BigDecimal("0.05"));
        BigDecimal maxBonus = getSalarioBase().multiply(new BigDecimal("0.50"));
        BigDecimal bonus = bonusPresupuesto.min(maxBonus);
        return getSalarioBase().add(bonus);
    }
}

enum NivelSenioridad { JUNIOR, MID, SENIOR }

**Tabla generada:**

CREATE TABLE empleados (
    id BIGINT PRIMARY KEY,
    tipo_empleado VARCHAR(31),     -- Discriminador
    nombre VARCHAR(255) NOT NULL,
    salario_base DECIMAL(10,2),
    lenguaje_principal VARCHAR(255),  -- Solo DEV
    nivel_senioridad VARCHAR(255),    -- Solo DEV
    departamento VARCHAR(255),        -- Solo MGR
    presupuesto DECIMAL(12,2)         -- Solo MGR
);

## Ejercicio 2: Cache de segundo nivel

### Enunciado

Configura cache de segundo nivel con Ehcache para la entidad `Categoria`:

- Maximo 100 entradas
    
- TTL de 1 hora
    
- Cachear tambien las consultas de findAll


Ver solucion

**1. Dependencia (pom.xml):**

<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-jcache</artifactId>
</dependency>
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>

**2. Configuracion (application.properties):**

spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.use_query_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
spring.jpa.properties.hibernate.javax.cache.provider=org.ehcache.jsr107.EhcacheCachingProvider
spring.jpa.properties.hibernate.javax.cache.uri=classpath:ehcache.xml

**3. ehcache.xml:**

<?xml version="1.0" encoding="UTF-8"?>
<config xmlns="http://www.ehcache.org/v3">
    
    <cache alias="com.ejemplo.Categoria">
        <expiry>
            <ttl unit="hours">1</ttl>
        </expiry>
        <heap unit="entries">100</heap>
    </cache>
    
    <cache alias="default-query-results-region">
        <expiry>
            <ttl unit="minutes">30</ttl>
        </expiry>
        <heap unit="entries">50</heap>
    </cache>
    
</config>

**4. Entidad cacheable:**

@Entity
@Table(name = "categorias")
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Categoria {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    // Solo cachear si la coleccion se accede frecuentemente
    @OneToMany(mappedBy = "categoria")
    @Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    private List<Producto> productos;
}

**5. Query cacheable en repositorio:**

@Repository
public interface CategoriaRepository extends JpaRepository<Categoria, Long> {
    
    @QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "true"))
    List<Categoria> findAll();
    
    @Query("SELECT c FROM Categoria c WHERE c.activa = true")
    @QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "true"))
    List<Categoria> findActivas();
}

**Estrategias de concurrencia:**

|Estrategia|Descripcion|
|---|---|
|READ_ONLY|Datos que nunca cambian|
|READ_WRITE|Datos que cambian, con bloqueo|
|NONSTRICT_READ_WRITE|Cambios poco frecuentes, sin bloqueo|
|TRANSACTIONAL|Soporte transaccional completo (JTA)|

## Ejercicio 3: Auditoria con Spring Data

### Enunciado

Implementa auditoria automatica usando Spring Data JPA Auditing:

- `createdBy`, `createdDate`
    
- `lastModifiedBy`, `lastModifiedDate`


El usuario actual debe obtenerse de Spring Security.

Ver solucion

**1. Habilitar auditoria:**

@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaAuditingConfig {
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> {
            // Obtener usuario de Spring Security
            Authentication auth = SecurityContextHolder.getContext().getAuthentication();
            if (auth == null || !auth.isAuthenticated()) {
                return Optional.of("sistema");
            }
            return Optional.of(auth.getName());
        };
    }
}

**2. Clase base auditable:**

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class Auditable {
    
    @CreatedBy
    @Column(name = "created_by", updatable = false)
    private String createdBy;
    
    @CreatedDate
    @Column(name = "created_date", updatable = false)
    private LocalDateTime createdDate;
    
    @LastModifiedBy
    @Column(name = "last_modified_by")
    private String lastModifiedBy;
    
    @LastModifiedDate
    @Column(name = "last_modified_date")
    private LocalDateTime lastModifiedDate;
    
    // Getters...
}

**3. Entidad que hereda auditoria:**

@Entity
@Table(name = "productos")
public class Producto extends Auditable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private BigDecimal precio;
    
    // getters, setters...
}

**4. Uso:**

@Service
public class ProductoService {
    
    @Transactional
    public Producto crear(ProductoDTO dto) {
        Producto producto = new Producto();
        producto.setNombre(dto.getNombre());
        producto.setPrecio(dto.getPrecio());
        
        // createdBy y createdDate se rellenan automaticamente
        return repository.save(producto);
    }
    
    @Transactional
    public Producto actualizar(Long id, ProductoDTO dto) {
        Producto producto = repository.findById(id).orElseThrow();
        producto.setNombre(dto.getNombre());
        
        // lastModifiedBy y lastModifiedDate se actualizan automaticamente
        return repository.save(producto);
    }
}

**Anotaciones disponibles:**

- @CreatedBy - Usuario que creo
    
- @CreatedDate - Fecha de creacion
    
- @LastModifiedBy - Ultimo usuario que modifico
    
- @LastModifiedDate - Fecha de ultima modificacion

## Ejercicio 4: EntityGraph para optimizacion

### Enunciado

Dado un `Pedido` con relaciones a `Cliente`, `Direccion` y lista de `LineaPedido` (que a su vez tiene `Producto`):

1. Define un EntityGraph nombrado que cargue todo en una consulta
    
2. Crea un EntityGraph ad-hoc en el repositorio

Ver solucion

**1. EntityGraph nombrado en la entidad:**

@Entity
@Table(name = "pedidos")
@NamedEntityGraph(
    name = "Pedido.completo",
    attributeNodes = {
        @NamedAttributeNode("cliente"),
        @NamedAttributeNode("direccionEnvio"),
        @NamedAttributeNode(value = "lineas", subgraph = "lineas-subgraph")
    },
    subgraphs = {
        @NamedSubgraph(
            name = "lineas-subgraph",
            attributeNodes = {
                @NamedAttributeNode("producto")
            }
        )
    }
)
public class Pedido {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Cliente cliente;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Direccion direccionEnvio;
    
    @OneToMany(mappedBy = "pedido", fetch = FetchType.LAZY)
    private List<LineaPedido> lineas;
    
    // ...
}

**2. Uso en repositorio - EntityGraph nombrado:**

@Repository
public interface PedidoRepository extends JpaRepository<Pedido, Long> {
    
    // Usar EntityGraph nombrado
    @EntityGraph(value = "Pedido.completo", type = EntityGraph.EntityGraphType.FETCH)
    Optional<Pedido> findById(Long id);
    
    @EntityGraph(value = "Pedido.completo")
    List<Pedido> findByClienteId(Long clienteId);
}

**3. EntityGraph ad-hoc en repositorio:**

@Repository
public interface PedidoRepository extends JpaRepository<Pedido, Long> {
    
    // EntityGraph ad-hoc
    @EntityGraph(attributePaths = {"cliente", "lineas"})
    List<Pedido> findByEstado(EstadoPedido estado);
    
    // Multiples niveles ad-hoc
    @EntityGraph(attributePaths = {"cliente", "lineas.producto"})
    Optional<Pedido> findConDetallesById(Long id);
}

**4. EntityGraph programatico:**

@Service
public class PedidoService {
    
    @PersistenceContext
    private EntityManager em;
    
    public Pedido buscarConGrafo(Long id, String... atributos) {
        EntityGraph<Pedido> graph = em.createEntityGraph(Pedido.class);
        for (String attr : atributos) {
            graph.addAttributeNodes(attr);
        }
        
        Map<String, Object> hints = new HashMap<>();
        hints.put("javax.persistence.fetchgraph", graph);
        
        return em.find(Pedido.class, id, hints);
    }
}

**Tipos de EntityGraph:**

- `FETCH`: Solo carga los atributos especificados
    
- `LOAD`: Carga especificados + respeta fetch type por defecto

## Ejercicio 5: Soft Delete

### Enunciado

Implementa borrado logico (soft delete) para la entidad `Usuario`:

- Columna `deleted` boolean
    
- Columna `deletedAt` timestamp
    
- Filtrar automaticamente los eliminados en todas las consultas


Ver solucion

**Entidad con Soft Delete (Hibernate):**

@Entity
@Table(name = "usuarios")
@SQLDelete(sql = "UPDATE usuarios SET deleted = true, deleted_at = NOW() WHERE id = ?")
@Where(clause = "deleted = false")
public class Usuario {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private String email;
    
    @Column(nullable = false)
    private Boolean deleted = false;
    
    @Column(name = "deleted_at")
    private LocalDateTime deletedAt;
    
    // getters, setters...
}

**Repositorio con metodos para incluir eliminados:**

@Repository
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
    
    // Por defecto, @Where filtra los eliminados
    List<Usuario> findAll();  // Solo no eliminados
    
    // Consulta nativa para incluir eliminados
    @Query(value = "SELECT * FROM usuarios", nativeQuery = true)
    List<Usuario> findAllIncludingDeleted();
    
    // JPQL explicito para incluir eliminados
    @Query("SELECT u FROM Usuario u WHERE u.deleted = true")
    List<Usuario> findDeleted();
    
    // Restaurar (hard update)
    @Modifying
    @Query("UPDATE Usuario u SET u.deleted = false, u.deletedAt = null WHERE u.id = :id")
    void restore(@Param("id") Long id);
}

**Alternativa con @Filter (dinamico):**

@Entity
@Table(name = "usuarios")
@SQLDelete(sql = "UPDATE usuarios SET deleted = true WHERE id = ?")
@FilterDef(name = "deletedFilter", parameters = @ParamDef(name = "isDeleted", type = Boolean.class))
@Filter(name = "deletedFilter", condition = "deleted = :isDeleted")
public class Usuario {
    // ...
}

// Activar/desactivar filtro
@Service
public class UsuarioService {
    
    @PersistenceContext
    private EntityManager em;
    
    public List<Usuario> buscarTodos(boolean incluirEliminados) {
        Session session = em.unwrap(Session.class);
        
        if (!incluirEliminados) {
            session.enableFilter("deletedFilter")
                   .setParameter("isDeleted", false);
        }
        
        return em.createQuery("SELECT u FROM Usuario u", Usuario.class)
                 .getResultList();
    }
}

## Reto adicional

Implementa un sistema de versionado de entidades (Entity Versioning):

- Cada vez que se modifica una entidad, guarda una copia en una tabla de historico
    
- Permitir consultar el estado de una entidad en cualquier momento del pasado


Consejo

Investiga Hibernate Envers para auditoria y versionado automatico.