## Objetivos

- Comprender la arquitectura de Spring Framework
    
- Entender los conceptos de IoC (Inversion de Control) y DI (Inyeccion de Dependencias)
    
- Conocer el contenedor de Spring y la gestion de beans
    
- Aprender a configurar Spring con anotaciones

## 1. Que es Spring Framework?

**Spring** es un framework de desarrollo de aplicaciones Java que proporciona una infraestructura completa para crear aplicaciones empresariales.

### 1.1 Caracteristicas Principales

|Caracteristica|Descripcion|
|---|---|
|**IoC Container**|Gestion del ciclo de vida de objetos|
|**Dependency Injection**|Inyeccion automatica de dependencias|
|**AOP**|Programacion orientada a aspectos|
|**Modularidad**|Uso solo de los modulos necesarios|
|**Testing**|Soporte completo para testing|

### 1.2 Modulos de Spring

┌─────────────────────────────────────────────────────────────────┐
│                         SPRING FRAMEWORK                         │
├─────────────────────────────────────────────────────────────────┤
│  DATA ACCESS          │  WEB            │  AOP / ASPECTS        │
│  ─────────────────    │  ───────────    │  ─────────────────    │
│  • JDBC               │  • Web          │  • AOP                │
│  • ORM (JPA)          │  • WebMVC       │  • Aspects            │
│  • Transactions       │  • WebSocket    │  • Instrumentation    │
├─────────────────────────────────────────────────────────────────┤
│                            CORE CONTAINER                        │
│  ─────────────────────────────────────────────────────────────  │
│  • Beans    • Core    • Context    • SpEL (Expression Language) │
└─────────────────────────────────────────────────────────────────┘

## 2. Inversion de Control (IoC)

### 2.1 Concepto

La **Inversion de Control** es un principio donde el control del flujo del programa se invierte:

- En programacion tradicional: el codigo de la aplicacion llama a las librerias
    
- Con IoC: el framework llama al codigo de la aplicacion

```java
// SIN IoC - Control en la aplicacion
public class MiApp {
    public static void main(String[] args) {
        LibroService service = new LibroService();
        LibroRepository repo = new LibroRepositoryImpl();
        service.setRepository(repo); // Nosotros controlamos la creacion
    }
}

// CON IoC - Control en Spring
@SpringBootApplication
public class MiApp {
    public static void main(String[] args) {
        SpringApplication.run(MiApp.class, args);
        // Spring crea y conecta todos los objetos automaticamente
    }
}
```

### 2.2 El Contenedor IoC

Spring proporciona dos tipos de contenedores:

|Contenedor|Interface|Descripcion|
|---|---|---|
|**BeanFactory**|`BeanFactory`|Contenedor basico, carga lazy|
|**ApplicationContext**|`ApplicationContext`|Contenedor avanzado, carga eager|

```java
// Crear ApplicationContext
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

// Obtener un bean
LibroService service = context.getBean(LibroService.class);
```

## 3. Beans de Spring

### 3.1 Que es un Bean?

Un **Bean** es un objeto gestionado por el contenedor de Spring. El contenedor:

- Crea la instancia
    
- Inyecta las dependencias
    
- Gestiona el ciclo de vida
    
- Destruye cuando ya no se necesita

### 3.2 Definir Beans con Anotaciones

```java
// Estereotipos de Spring
@Component    // Bean generico
@Service      // Bean de capa de servicio
@Repository   // Bean de capa de acceso a datos
@Controller   // Bean de capa web
@RestController // Controller + ResponseBody

// Ejemplo de Service
@Service
public class LibroService {
    private final LibroRepository repository;
    
    public LibroService(LibroRepository repository) {
        this.repository = repository;
    }
    
    public List<Libro> listarTodos() {
        return repository.findAll();
    }
}

// Ejemplo de Repository
@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
}
```

### 3.3 Definir Beans con `@Configuration`

```java
@Configuration
public class AppConfig {
    
    @Bean
    public DataSource dataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:postgresql://localhost:5432/biblioteca");
        ds.setUsername("biblioteca");
        ds.setPassword("biblioteca123");
        return ds;
    }
    
    @Bean
    public EmailService emailService() {
        return new EmailServiceImpl("smtp.gmail.com", 587);
    }
    
    @Bean
    @Profile("dev")  // Solo en perfil de desarrollo
    public LogService devLogService() {
        return new ConsoleLogService();
    }
    
    @Bean
    @Profile("prod")  // Solo en produccion
    public LogService prodLogService() {
        return new FileLogService("/var/log/app.log");
    }
}
```

### 3.4 Scope de Beans

|Scope|Descripcion|Uso|
|---|---|---|
|**singleton**|Una instancia por contenedor (por defecto)|Services, Repositories|
|**prototype**|Nueva instancia cada vez que se solicita|Objetos con estado|
|**request**|Una instancia por peticion HTTP|Web|
|**session**|Una instancia por sesion HTTP|Web|
|**application**|Una instancia por ServletContext|Web|

```java
@Service
@Scope("singleton")  // Por defecto, no necesario especificar
public class LibroService {
}

@Component
@Scope("prototype")  // Nueva instancia cada vez
public class CarritoCompra {
    private List<Producto> items = new ArrayList<>();
}

@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class UsuarioSesion {
    private Usuario usuario;
}
```

## 4. Inyeccion de Dependencias en Spring

### 4.1 Inyeccion por Constructor (Recomendada)

```java
@Service
public class LibroService {
    private final LibroRepository repository;
    private final EmailService emailService;
    private final AuditService auditService;
    
    // Spring inyecta automaticamente si hay un unico constructor
    public LibroService(LibroRepository repository, 
                        EmailService emailService,
                        AuditService auditService) {
        this.repository = repository;
        this.emailService = emailService;
        this.auditService = auditService;
    }
}
```

**Ventajas:**

- Dependencias inmutables (final)
    
- Facilita testing
    
- Falla rapido si falta dependencia

### 4.2 Inyeccion con `@Autowired`

```java
// Por campo (no recomendado, pero comun)
@Service
public class LibroService {
    @Autowired
    private LibroRepository repository;
}

// Por setter
@Service
public class LibroService {
    private LibroRepository repository;
    
    @Autowired
    public void setRepository(LibroRepository repository) {
        this.repository = repository;
    }
}

// Dependencia opcional
@Service
public class LibroService {
    @Autowired(required = false)
    private CacheService cacheService;  // Puede ser null
}
```

### 4.3 Resolver Ambiguedades con `@Qualifier`

Cuando hay multiples beans del mismo tipo:

```java
// Dos implementaciones
@Service("emailNotificacion")
public class EmailNotificacion implements NotificacionService {
}

@Service("smsNotificacion")
public class SMSNotificacion implements NotificacionService {
}

// Especificar cual usar
@Service
public class AlertaService {
    private final NotificacionService notificador;
    
    public AlertaService(@Qualifier("emailNotificacion") NotificacionService notificador) {
        this.notificador = notificador;
    }
}

// Alternativa: @Primary
@Service
@Primary  // Sera la opcion por defecto
public class EmailNotificacion implements NotificacionService {
}
```

## 5. Ciclo de Vida de Beans

### 5.1 Fases del Ciclo de Vida

┌─────────────────────────────────────────────────────────────┐
│                    CICLO DE VIDA DE UN BEAN                  │
├─────────────────────────────────────────────────────────────┤
│  1. Instanciacion         → Spring crea el objeto           │
│  2. Inyeccion DI          → Se inyectan dependencias        │
│  3. @PostConstruct        → Metodo de inicializacion        │
│  4. Uso                   → El bean esta listo              │
│  5. @PreDestroy           → Antes de destruir               │
│  6. Destruccion           → Spring destruye el bean         │
└─────────────────────────────────────────────────────────────┘

### 5.2 Callbacks de Ciclo de Vida

```java
@Service
public class ConexionService {
    private Connection connection;
    
    // Se ejecuta despues de inyectar dependencias
    @PostConstruct
    public void inicializar() {
        System.out.println("Inicializando conexion...");
        connection = crearConexion();
    }
    
    // Se ejecuta antes de destruir el bean
    @PreDestroy
    public void limpiar() {
        System.out.println("Cerrando conexion...");
        if (connection != null) {
            connection.close();
        }
    }
}

// Alternativa con interfaces
@Service
public class ConexionService implements InitializingBean, DisposableBean {
    
    @Override
    public void afterPropertiesSet() throws Exception {
        // Inicializacion
    }
    
    @Override
    public void destroy() throws Exception {
        // Limpieza
    }
}

```
## 6. Configuracion con Propiedades

### 6.1 `@Value` para Inyectar Propiedades

```properties
# application.properties
app.nombre=Biblioteca
app.version=1.0.0
db.url=jdbc:postgresql://localhost:5432/biblioteca
db.timeout=30
```

```java
@Service
public class AppInfo {
    
    @Value("${app.nombre}")
    private String nombreApp;
    
    @Value("${app.version}")
    private String version;
    
    @Value("${db.timeout:60}")  // Valor por defecto 60
    private int timeout;
    
    @Value("${debug:false}")
    private boolean debugMode;
}
```

### 6.2 `@ConfigurationProperties` para Grupos de Propiedades

```properties
# application.properties
biblioteca.nombre=Biblioteca Central
biblioteca.max-prestamos=5
biblioteca.dias-prestamo=15
biblioteca.admin.email=admin@biblioteca.com
biblioteca.admin.nombre=Administrador
```

```java
@Configuration
@ConfigurationProperties(prefix = "biblioteca")
public class BibliotecaConfig {
    private String nombre;
    private int maxPrestamos;
    private int diasPrestamo;
    private Admin admin = new Admin();
    
    // Getters y setters
    
    public static class Admin {
        private String email;
        private String nombre;
        // Getters y setters
    }
}

// Uso
@Service
public class PrestamoService {
    private final BibliotecaConfig config;
    
    public boolean puedePrestar(Usuario u) {
        return u.getPrestamosActivos() < config.getMaxPrestamos();
    }
}
```

## 7. Perfiles de Spring

### 7.1 Definir Perfiles

```java
// Bean solo para desarrollo
@Service
@Profile("dev")
public class MockEmailService implements EmailService {
    @Override
    public void enviar(String to, String asunto, String cuerpo) {
        System.out.println("[MOCK] Email a: " + to);
    }
}

// Bean para produccion
@Service
@Profile("prod")
public class SmtpEmailService implements EmailService {
    @Override
    public void enviar(String to, String asunto, String cuerpo) {
        // Envio real via SMTP
    }
}

// Bean para multiples perfiles
@Service
@Profile({"dev", "test"})
public class H2DatabaseService implements DatabaseService {
}
```

### 7.2 Archivos de Propiedades por Perfil

```
src/main/resources/
├── application.properties           # Comun a todos
├── application-dev.properties       # Desarrollo
├── application-test.properties      # Testing
└── application-prod.properties      # Produccion

```

```properties
# application-dev.properties
spring.datasource.url=jdbc:h2:mem:testdb
logging.level.root=DEBUG

# application-prod.properties
spring.datasource.url=jdbc:postgresql://db.servidor.com:5432/biblioteca
logging.level.root=WARN

```

### 7.3 Activar Perfil

```powershell
# Por linea de comandos
java -jar app.jar --spring.profiles.active=prod

# Variable de entorno
export SPRING_PROFILES_ACTIVE=prod

# En application.properties
spring.profiles.active=dev
```

## 8. Ejemplo Completo

```java
// Entidad
@Entity
public class Libro {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String titulo;
    private String autor;
    private String isbn;
    private boolean disponible = true;
    // Getters, setters, constructores
}

// Repository
@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    List<Libro> findByDisponibleTrue();
    Optional<Libro> findByIsbn(String isbn);
}

// Service
@Service
public class LibroService {
    private final LibroRepository repository;
    
    public LibroService(LibroRepository repository) {
        this.repository = repository;
    }
    
    public List<Libro> listarDisponibles() {
        return repository.findByDisponibleTrue();
    }
    
    @Transactional
    public Libro prestar(Long id) {
        Libro libro = repository.findById(id)
            .orElseThrow(() -> new RuntimeException("Libro no encontrado"));
        
        if (!libro.isDisponible()) {
            throw new RuntimeException("Libro no disponible");
        }
        
        libro.setDisponible(false);
        return repository.save(libro);
    }
}

// Controller
@RestController
@RequestMapping("/api/libros")
public class LibroController {
    private final LibroService libroService;
    
    public LibroController(LibroService libroService) {
        this.libroService = libroService;
    }
    
    @GetMapping("/disponibles")
    public List<Libro> listarDisponibles() {
        return libroService.listarDisponibles();
    }
    
    @PostMapping("/{id}/prestar")
    public Libro prestar(@PathVariable Long id) {
        return libroService.prestar(id);
    }
}
```

## 9. Resumen

### Conceptos Clave

- **Spring Framework** es el framework mas usado en Java empresarial
    
- **IoC (Inversion de Control)** delega el control al framework
    
- **Beans** son objetos gestionados por el contenedor Spring
    
- Las anotaciones `@Component`, `@Service`, `@Repository`, `@Controller` definen beans
    
- La **inyeccion por constructor** es la forma preferida de DI
    
- Los **perfiles** permiten configuracion por entorno

### Anotaciones Principales

|Anotacion|Uso|
|---|---|
|`@Component`|Bean generico|
|`@Service`|Logica de negocio|
|`@Repository`|Acceso a datos|
|`@Controller`|Web MVC|
|`@RestController`|API REST|
|`@Autowired`|Inyeccion de dependencias|
|`@Value`|Inyeccion de propiedades|
|`@Configuration`|Clase de configuracion|
|`@Bean`|Define un bean manualmente|