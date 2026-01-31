## CE 4h: Probar y documentar las aplicaciones desarrolladas

En este tema aprenderemos a crear pruebas automatizadas y documentar aplicaciones que trabajan con bases de datos orientadas a objetos.

## 8.1 Tipos de Pruebas

|Tipo|Descripción|Herramientas|
|---|---|---|
|**Unitarias**|Prueban clases/métodos aislados|JUnit, Mockito|
|**Integración**|Prueban interacción con BD real|JUnit, Testcontainers|
|**E2E**|Prueban flujos completos|Selenium, RestAssured|

Para aplicaciones con bases de datos, las **pruebas de integración** son especialmente importantes.

## 8.2 Configuración de JUnit 5

### Dependencias Maven

<dependencies>
    <!-- JUnit 5 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.1</version>
        <scope>test</scope>
    </dependency>
    
    <!-- AssertJ para aserciones fluidas -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.24.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>

## 8.3 Pruebas de Persistencia con JPA

### Estructura Básica de Test

import org.junit.jupiter.api.*;
import jakarta.persistence.*;
import static org.junit.jupiter.api.Assertions.*;

class ProductoDAOTest {
    
    private static EntityManagerFactory emf;
    private EntityManager em;
    
    @BeforeAll
    static void initFactory() {
        // Usar unidad de persistencia de test (H2 en memoria)
        emf = Persistence.createEntityManagerFactory("test-unit");
    }
    
    @BeforeEach
    void setUp() {
        em = emf.createEntityManager();
        em.getTransaction().begin();
    }
    
    @AfterEach
    void tearDown() {
        if (em.getTransaction().isActive()) {
            em.getTransaction().rollback(); // Rollback para aislar tests
        }
        em.close();
    }
    
    @AfterAll
    static void closeFactory() {
        if (emf != null) emf.close();
    }
}

### Tests CRUD

@Test
@DisplayName("Persistir producto genera ID")
void testPersistirProducto() {
    Producto p = new Producto("Test", new BigDecimal("50.00"), 10);
    
    em.persist(p);
    em.flush();
    
    assertNotNull(p.getId(), "El ID debe generarse");
}

@Test
@DisplayName("Encontrar producto por ID")
void testFindById() {
    // Arrange
    Producto p = new Producto("Laptop", new BigDecimal("999.99"), 5);
    em.persist(p);
    em.flush();
    em.clear(); // Limpiar caché
    
    // Act
    Producto encontrado = em.find(Producto.class, p.getId());
    
    // Assert
    assertNotNull(encontrado);
    assertEquals("Laptop", encontrado.getNombre());
}

@Test
@DisplayName("Actualizar precio de producto")
void testActualizarPrecio() {
    Producto p = new Producto("Monitor", new BigDecimal("300.00"), 10);
    em.persist(p);
    em.flush();
    
    p.setPrecio(new BigDecimal("350.00"));
    em.flush();
    em.clear();
    
    Producto actualizado = em.find(Producto.class, p.getId());
    assertEquals(new BigDecimal("350.00"), actualizado.getPrecio());
}

@Test
@DisplayName("Eliminar producto")
void testEliminarProducto() {
    Producto p = new Producto("Temporal", new BigDecimal("10.00"), 1);
    em.persist(p);
    em.flush();
    Long id = p.getId();
    
    em.remove(p);
    em.flush();
    
    assertNull(em.find(Producto.class, id));
}

### Tests de Objetos Embebidos

@Test
@DisplayName("Persistir cliente con dirección embebida")
void testClienteConDireccion() {
    Direccion dir = new Direccion();
    dir.setCalle("Gran Vía 100");
    dir.setCiudad("Madrid");
    dir.setCodigoPostal("28013");
    dir.setPais("España");
    
    Cliente cliente = new Cliente();
    cliente.setNombre("Juan");
    cliente.setDireccion(dir);
    
    em.persist(cliente);
    em.flush();
    em.clear();
    
    Cliente encontrado = em.find(Cliente.class, cliente.getId());
    
    assertNotNull(encontrado.getDireccion());
    assertEquals("Madrid", encontrado.getDireccion().getCiudad());
}

### Tests de Colecciones

@Test
@DisplayName("Agregar teléfonos a cliente")
void testColeccionTelefonos() {
    Cliente cliente = new Cliente();
    cliente.setNombre("María");
    cliente.getTelefonos().add("666111222");
    cliente.getTelefonos().add("933445566");
    
    em.persist(cliente);
    em.flush();
    em.clear();
    
    Cliente encontrado = em.find(Cliente.class, cliente.getId());
    
    assertEquals(2, encontrado.getTelefonos().size());
    assertTrue(encontrado.getTelefonos().contains("666111222"));
}

### Tests de Herencia

@Test
@DisplayName("Consulta polimórfica de recursos")
void testHerenciaRecursos() {
    Libro libro = new Libro();
    libro.setTitulo("Don Quijote");
    libro.setAutor("Cervantes");
    libro.setIsbn("978-84-376-0494-7");
    
    DVD dvd = new DVD();
    dvd.setTitulo("Matrix");
    dvd.setDirector("Wachowski");
    dvd.setDuracion(136);
    
    em.persist(libro);
    em.persist(dvd);
    em.flush();
    em.clear();
    
    // Consulta polimórfica
    List<Recurso> recursos = em.createQuery(
        "SELECT r FROM Recurso r", Recurso.class)
        .getResultList();
    
    assertEquals(2, recursos.size());
    
    // Verificar tipos
    long libros = recursos.stream()
        .filter(r -> r instanceof Libro)
        .count();
    assertEquals(1, libros);
}

## 8.4 Testcontainers

Testcontainers permite ejecutar bases de datos reales en contenedores Docker durante los tests:

### Configuración

<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>

### Test con PostgreSQL Container

import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
class IntegracionPostgreSQLTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test")
        .withInitScript("init.sql"); // Script de inicialización
    
    @Test
    void testConexion() throws SQLException {
        try (Connection conn = DriverManager.getConnection(
                postgres.getJdbcUrl(),
                postgres.getUsername(),
                postgres.getPassword())) {
            
            assertTrue(conn.isValid(5));
            
            // Ejecutar consultas reales
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery(
                "SELECT (direccion).ciudad FROM clientes");
            
            // Verificar resultados
        }
    }
}

## 8.5 Documentación con Javadoc

### Documentar Entidades

/**
 * Representa un producto en el inventario.
 * 
 * <p>Los productos pueden tener diferentes estados y
 * participan en pedidos a través de líneas de pedido.</p>
 * 
 * @author David Valbuena
 * @version 1.0
 * @since 2025
 */
@Entity
@Table(name = "productos")
public class Producto {
    
    /**
     * Identificador único del producto.
     * Generado automáticamente por la base de datos.
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    /**
     * Calcula el valor total del stock disponible.
     * 
     * @return precio * stock como BigDecimal
     * @throws IllegalStateException si precio o stock son null
     */
    public BigDecimal calcularValorStock() {
        if (precio == null || stock == null) {
            throw new IllegalStateException("Precio y stock requeridos");
        }
        return precio.multiply(BigDecimal.valueOf(stock));
    }
}

### Documentar DAOs

/**
 * DAO para operaciones de persistencia de productos.
 * 
 * <p>Proporciona métodos CRUD y consultas especializadas
 * para la entidad Producto.</p>
 * 
 * <h2>Uso típico:</h2>
 * <pre>{@code
 * ProductoDAO dao = new ProductoDAO(em);
 * Producto p = dao.findById(1L);
 * }</pre>
 */
public class ProductoDAO {
    
    /**
     * Busca productos por rango de precio.
     * 
     * @param minimo precio mínimo (inclusivo)
     * @param maximo precio máximo (inclusivo)
     * @return lista de productos en el rango, puede estar vacía
     * @throws IllegalArgumentException si mínimo > máximo
     */
    public List<Producto> findByPrecioEntre(BigDecimal minimo, BigDecimal maximo) {
        // implementación
    }
}

## 8.6 Documentación de Scripts SQL

### Oracle

/*
 * Script: 01_tipos_biblioteca.sql
 * Descripción: Crea los tipos objeto para el sistema de biblioteca
 * Autor: David Valbuena
 * Fecha: 2025-01
 * Requisitos: Oracle 21c o superior
 */

-- Tipo para direcciones postales
-- Usado por: tipo_usuario, tipo_editorial
CREATE OR REPLACE TYPE tipo_direccion AS OBJECT (
    calle           VARCHAR2(200),  -- Nombre de la calle y número
    ciudad          VARCHAR2(100),  -- Ciudad
    codigo_postal   VARCHAR2(10),   -- Código postal
    pais            VARCHAR2(50)    -- País (ISO 3166)
);
/

### PostgreSQL

/*
 * Script: 01_tipos_biblioteca.sql
 * Descripción: Crea los tipos compuestos para el sistema de biblioteca
 * Autor: David Valbuena
 * Fecha: 2025-01
 * Requisitos: PostgreSQL 15 o superior
 */

-- Tipo para direcciones postales
CREATE TYPE tipo_direccion AS (
    calle           VARCHAR(200),   -- Nombre de la calle y número
    ciudad          VARCHAR(100),   -- Ciudad
    codigo_postal   VARCHAR(10),    -- Código postal
    pais            VARCHAR(50)     -- País (ISO 3166)
);

COMMENT ON TYPE tipo_direccion IS 'Dirección postal estándar';

## 8.7 Logging

### Configuración SLF4J + Logback

<!-- logback.xml -->
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- Log de SQL generado por Hibernate -->
    <logger name="org.hibernate.SQL" level="DEBUG"/>
    <logger name="org.hibernate.type.descriptor.sql" level="TRACE"/>
    
    <root level="INFO">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>

### Uso en Código

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ProductoDAO {
    private static final Logger logger = LoggerFactory.getLogger(ProductoDAO.class);
    
    public Producto findById(Long id) {
        logger.debug("Buscando producto con ID: {}", id);
        
        Producto p = em.find(Producto.class, id);
        
        if (p == null) {
            logger.warn("Producto no encontrado: {}", id);
        } else {
            logger.info("Producto encontrado: {}", p.getNombre());
        }
        
        return p;
    }
}

## 8.8 Resumen del Tema

Conceptos Clave

1. **JUnit 5** con `@BeforeEach`/`@AfterEach` para setup y teardown de EntityManager.
    
2. Usar **rollback en tests** para aislar las pruebas entre sí.
    
3. `em.flush()` y `em.clear()` para forzar sincronización y limpiar caché.
    
4. **Testcontainers** ejecuta contenedores Docker con bases de datos reales.
    
5. **Javadoc** documenta clases, métodos y parámetros.
    
6. **SLF4J + Logback** para logging estructurado con niveles DEBUG, INFO, WARN, ERROR.