## Objetivos

- Comprender los conceptos basicos de la programacion orientada a componentes
    
- Valorar las ventajas e inconvenientes de este paradigma
    
- Identificar las caracteristicas de un buen componente de software
    
- Conocer los patrones de diseno mas utilizados

## 1. Introduccion a los Componentes de Software

### 1.1 Que es un componente?

Un **componente de software** es una unidad de composicion con interfaces especificadas contractualmente y dependencias de contexto explicitas. Es una pieza de software reutilizable que puede desplegarse independientemente.

**Caracteristicas principales:**

|Caracteristica|Descripcion|
|---|---|
|**Encapsulamiento**|Oculta su implementacion interna|
|**Reutilizacion**|Puede usarse en diferentes contextos|
|**Independencia**|Tiene baja dependencia de otros componentes|
|**Sustituibilidad**|Puede reemplazarse por otro que cumpla la misma interfaz|
|**Composicion**|Puede combinarse con otros componentes|

### 1.2 Diferencia entre Componente, Modulo y Libreria

┌─────────────────────────────────────────────────────────────┐
│                      LIBRERIA                                │
│   Coleccion de funciones/clases reutilizables               │
│   No tiene estado propio ni ciclo de vida                    │
│   Ejemplo: Apache Commons, Guava                             │
├─────────────────────────────────────────────────────────────┤
│                       MODULO                                 │
│   Unidad de organizacion de codigo                           │
│   Agrupa clases relacionadas                                 │
│   Ejemplo: java.base, java.sql                               │
├─────────────────────────────────────────────────────────────┤
│                     COMPONENTE                               │
│   Unidad desplegable independiente                           │
│   Tiene interfaz definida y ciclo de vida                    │
│   Ejemplo: Spring Bean, EJB, Microservicio                   │
└─────────────────────────────────────────────────────────────┘

## 2. Ventajas e Inconvenientes (CE 6a)

### 2.1 Ventajas de la Programacion Orientada a Componentes

1. **Reutilizacion de codigo**
    
    - Los componentes bien disenados pueden reutilizarse en multiples proyectos
        
    - Reduce el tiempo de desarrollo
        
2. **Mantenibilidad**
    
    - Cambios localizados en componentes especificos
        
    - Menor impacto en el resto del sistema
        
3. **Desarrollo en paralelo**
    
    - Equipos pueden trabajar en componentes diferentes simultaneamente
        
    - Integracion mediante interfaces definidas
        
4. **Testing independiente**
    
    - Cada componente puede probarse de forma aislada
        
    - Facilita la deteccion de errores
        
5. **Escalabilidad**
    
    - Componentes pueden escalarse independientemente
        
    - Optimizacion de recursos

### 2.2 Inconvenientes de la Programacion Orientada a Componentes

1. **Complejidad inicial**
    
    - Mayor curva de aprendizaje
        
    - Requiere planificacion cuidadosa
        
2. **Overhead de comunicacion**
    
    - Los componentes deben comunicarse mediante interfaces
        
    - Posible impacto en rendimiento
        
3. **Gestion de dependencias**
    
    - Multiples versiones de componentes
        
    - Conflictos de compatibilidad
        
4. **Diseno de interfaces**
    
    - Interfaces mal disenadas limitan la flexibilidad
        
    - Dificil modificar interfaces una vez publicadas

## 3. Principios SOLID

Los principios SOLID son fundamentales para disenar buenos componentes:

### 3.1 Single Responsibility Principle (SRP)

**Una clase debe tener una unica razon para cambiar.**

```java
// MAL: Multiples responsabilidades
public class Usuario {
    public void guardarEnBD() { ... }
    public void enviarEmail() { ... }
    public void generarPDF() { ... }
}

// BIEN: Responsabilidad unica
public class Usuario {
    private String nombre;
    private String email;
    // Solo datos del usuario
}

public class UsuarioRepository {
    public void guardar(Usuario u) { ... }
}

public class EmailService {
    public void enviar(Usuario u, String asunto) { ... }
}

```
### 3.2 Open/Closed Principle (OCP)

**Las entidades deben estar abiertas para extension, cerradas para modificacion.**

```java
// BIEN: Extensible mediante interfaces
public interface NotificacionService {
    void enviar(String mensaje, String destinatario);
}

public class EmailNotificacion implements NotificacionService {
    @Override
    public void enviar(String mensaje, String destinatario) {
        // Enviar por email
    }
}

public class SMSNotificacion implements NotificacionService {
    @Override
    public void enviar(String mensaje, String destinatario) {
        // Enviar por SMS
    }
}

// Nuevos tipos de notificacion sin modificar codigo existente
public class PushNotificacion implements NotificacionService {
    @Override
    public void enviar(String mensaje, String destinatario) {
        // Enviar push notification
    }
}
```

### 3.3 Liskov Substitution Principle (LSP)

**Los objetos de una superclase deben poder sustituirse por objetos de sus subclases.**

```java
// BIEN: Subtipos sustituibles
public abstract class Ave {
    public abstract void mover();
}

public class Paloma extends Ave {
    @Override
    public void mover() {
        volar();
    }
    
    private void volar() { ... }
}

public class Pinguino extends Ave {
    @Override
    public void mover() {
        caminar();
    }
    
    private void caminar() { ... }
}
```

### 3.4 Interface Segregation Principle (ISP)

**Los clientes no deben verse forzados a depender de interfaces que no usan.**

```java
// MAL: Interface demasiado grande
public interface Trabajador {
    void trabajar();
    void comer();
    void dormir();
}

// BIEN: Interfaces segregadas
public interface Trabajable {
    void trabajar();
}

public interface Alimentable {
    void comer();
}

public interface Descansable {
    void dormir();
}

public class Empleado implements Trabajable, Alimentable, Descansable {
    // Implementa todo
}

public class Robot implements Trabajable {
    // Solo trabaja
}
```

### 3.5 Dependency Inversion Principle (DIP)

**Depender de abstracciones, no de implementaciones concretas.**

```java
// MAL: Dependencia de clase concreta
public class ServicioNotificacion {
    private EmailSender emailSender = new EmailSender();
    
    public void notificar(String mensaje) {
        emailSender.enviar(mensaje);
    }
}

// BIEN: Inyeccion de dependencias
public class ServicioNotificacion {
    private final NotificacionService notificador;
    
    // Inyeccion por constructor
    public ServicioNotificacion(NotificacionService notificador) {
        this.notificador = notificador;
    }
    
    public void notificar(String mensaje, String destino) {
        notificador.enviar(mensaje, destino);
    }
}

```
## 4. Patrones de Arquitectura

### 4.1 Arquitectura en Capas

┌─────────────────────────────────────┐
│         CAPA PRESENTACION           │  Controllers, Views
├─────────────────────────────────────┤
│           CAPA SERVICIO             │  Business Logic
├─────────────────────────────────────┤
│        CAPA REPOSITORIO             │  Data Access
├─────────────────────────────────────┤
│            CAPA DATOS               │  Database
└─────────────────────────────────────┘

**Ejemplo en Spring:**

```java
// Capa Controlador
@RestController
@RequestMapping("/api/libros")
public class LibroController {
    private final LibroService libroService;
    
    @GetMapping
    public List<LibroDTO> listar() {
        return libroService.listarTodos();
    }
}

// Capa Servicio
@Service
public class LibroService {
    private final LibroRepository libroRepository;
    
    public List<LibroDTO> listarTodos() {
        return libroRepository.findAll()
            .stream()
            .map(this::convertirADTO)
            .collect(Collectors.toList());
    }
}

// Capa Repositorio
@Repository
public interface LibroRepository extends JpaRepository<Libro, Long> {
    List<Libro> findByAutor(String autor);
}
```

### 4.2 Patron DAO (Data Access Object)

Separa la logica de acceso a datos de la logica de negocio:

```java
// Interface DAO
public interface LibroDAO {
    Libro buscarPorId(Long id);
    List<Libro> buscarTodos();
    void guardar(Libro libro);
    void actualizar(Libro libro);
    void eliminar(Long id);
}

// Implementacion JDBC
public class LibroDAOJdbc implements LibroDAO {
    private final DataSource dataSource;
    
    @Override
    public Libro buscarPorId(Long id) {
        String sql = "SELECT * FROM libros WHERE id = ?";
        // Implementacion JDBC...
    }
}

// Implementacion JPA
public class LibroDAOJpa implements LibroDAO {
    private final EntityManager em;
    
    @Override
    public Libro buscarPorId(Long id) {
        return em.find(Libro.class, id);
    }
}
```

### 4.3 Patron Repository

Similar a DAO pero con enfoque en colecciones de dominio:

```java
// Repository con Spring Data JPA
public interface LibroRepository extends JpaRepository<Libro, Long> {
    // Query Methods
    List<Libro> findByTituloContaining(String titulo);
    
    List<Libro> findByAutorAndAnioGreaterThan(String autor, int anio);
    
    // Consulta personalizada
    @Query("SELECT l FROM Libro l WHERE l.disponible = true ORDER BY l.titulo")
    List<Libro> findLibrosDisponibles();
    
    // Consulta nativa
    @Query(value = "SELECT * FROM libros WHERE stock > 0", nativeQuery = true)
    List<Libro> findLibrosConStock();
}
```

## 5. Inyeccion de Dependencias (DI)

### 5.1 Tipos de Inyeccion

```java
// 1. Inyeccion por constructor (recomendada)
@Service
public class LibroService {
    private final LibroRepository repository;
    private final EmailService emailService;
    
    // El constructor define las dependencias obligatorias
    public LibroService(LibroRepository repository, EmailService emailService) {
        this.repository = repository;
        this.emailService = emailService;
    }
}

// 2. Inyeccion por setter
@Service
public class LibroService {
    private LibroRepository repository;
    
    @Autowired
    public void setRepository(LibroRepository repository) {
        this.repository = repository;
    }
}

// 3. Inyeccion por campo (no recomendada)
@Service
public class LibroService {
    @Autowired
    private LibroRepository repository;
}
```

### 5.2 Beneficios de la Inyeccion de Dependencias

|Beneficio|Descripcion|
|---|---|
|**Testabilidad**|Facil inyectar mocks para testing|
|**Bajo acoplamiento**|Componentes independientes|
|**Configurabilidad**|Cambiar implementaciones sin modificar codigo|
|**Mantenibilidad**|Codigo mas limpio y organizado|

```java
// Test con mock
@ExtendWith(MockitoExtension.class)
class LibroServiceTest {
    
    @Mock
    private LibroRepository repository;
    
    @InjectMocks
    private LibroService service;
    
    @Test
    void debeListarLibros() {
        // Configurar mock
        when(repository.findAll()).thenReturn(List.of(new Libro("Test")));
        
        // Ejecutar
        List<Libro> resultado = service.listarTodos();
        
        // Verificar
        assertEquals(1, resultado.size());
    }
}

```
## 6. Herramientas de Desarrollo de Componentes (CE 6b)

### 6.1 Frameworks de Componentes Java

|Framework|Descripcion|Uso Principal|
|---|---|---|
|**Spring Framework**|Framework completo para Java|Aplicaciones empresariales|
|**Jakarta EE (Java EE)**|Especificacion estandar|Servidores de aplicaciones|
|**Quarkus**|Framework nativo para cloud|Microservicios, Kubernetes|
|**Micronaut**|Framework ligero|Microservicios, serverless|
|**OSGi**|Sistema de modulos|Aplicaciones modulares|

### 6.2 Herramientas de Build

|Herramienta|Descripcion|
|---|---|
|**Maven**|Gestion de proyectos basada en POM|
|**Gradle**|Build tool flexible con DSL Groovy/Kotlin|
|**Ant**|Build tool basado en XML|

### 6.3 IDEs Recomendados

- **IntelliJ IDEA** - Soporte completo para Spring
    
- **Eclipse/STS** - Spring Tool Suite
    
- **VS Code** - Con extensiones Java y Spring

## 7. Resumen

### Conceptos Clave

- Un **componente** es una unidad de software reutilizable, desplegable e independiente
    
- Los principios **SOLID** guian el diseno de buenos componentes
    
- La **inyeccion de dependencias** reduce el acoplamiento
    
- La **arquitectura en capas** organiza los componentes
    
- **Spring Framework** es la herramienta principal para componentes en Java

### Criterios de Evaluacion Cubiertos

- **CE 6a**: Ventajas e inconvenientes de programacion orientada a componentes
    
- **CE 6b**: Herramientas de desarrollo de componentes

## 8. Ejercicios Propuestos

1. Identifica las violaciones de los principios SOLID en un codigo dado
    
2. Refactoriza una clase con multiples responsabilidades aplicando SRP
    
3. Disena una arquitectura de componentes para una aplicacion de biblioteca
    
4. Compara las ventajas de usar Spring vs Jakarta EE para tu proyecto