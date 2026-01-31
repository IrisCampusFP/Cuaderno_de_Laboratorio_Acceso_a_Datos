## Objetivos

- Identificar componentes de software en una arquitectura
    
- Analizar ventajas e inconvenientes de diferentes enfoques
    
- Aplicar principios SOLID en el diseno
    

## Ejercicio 1: Identificacion de Componentes

### Enunciado

Analiza el siguiente diagrama de arquitectura y responde:

┌─────────────────────────────────────────────────────────────┐
│                    APLICACION BIBLIOTECA                     │
├─────────────────────────────────────────────────────────────┤
│  ┌───────────┐  ┌───────────┐  ┌───────────┐               │
│  │ UI Web    │  │ API REST  │  │ Admin CLI │               │
│  └───────────┘  └───────────┘  └───────────┘               │
├─────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────┐  │
│  │              CAPA DE SERVICIOS                         │  │
│  │  LibroService  |  UsuarioService  |  PrestamoService  │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────┐  │
│  │              CAPA DE REPOSITORIOS                      │  │
│  │  LibroRepo  |  UsuarioRepo  |  PrestamoRepo           │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ PostgreSQL  │  │   Redis     │  │ FileSystem  │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘

1. Identifica los componentes de la aplicacion
    
2. Clasifica cada componente por su funcion
    
3. Indica las dependencias entre componentes
    

Ver solucion

**1. Componentes identificados:**

|Componente|Tipo|Funcion|
|---|---|---|
|UI Web|Presentacion|Interfaz grafica web|
|API REST|Presentacion|Endpoints para clientes|
|Admin CLI|Presentacion|Administracion por consola|
|LibroService|Servicio|Logica de negocio de libros|
|UsuarioService|Servicio|Logica de negocio de usuarios|
|PrestamoService|Servicio|Logica de prestamos|
|LibroRepo|Repositorio|Acceso a datos de libros|
|UsuarioRepo|Repositorio|Acceso a datos de usuarios|
|PrestamoRepo|Repositorio|Acceso a datos de prestamos|

**2. Clasificacion por capas:**

- Capa de Presentacion: UI Web, API REST, Admin CLI
    
- Capa de Negocio: Services
    
- Capa de Acceso a Datos: Repositories
    
- Capa de Persistencia: PostgreSQL, Redis, FileSystem
    

**3. Dependencias:**

- Presentacion -> Servicios (los controladores usan servicios)
    
- Servicios -> Repositorios (los servicios usan repositorios)
    
- Repositorios -> BD (los repositorios acceden a bases de datos)
    
- PrestamoService -> LibroService, UsuarioService (dependencia horizontal)
    

## Ejercicio 2: Analisis de Principios SOLID

### Enunciado

Analiza el siguiente codigo e identifica que principios SOLID se violan:

public class GestorBiblioteca {
    private Connection connection;
    
    public GestorBiblioteca() {
        this.connection = DriverManager.getConnection("jdbc:mysql://localhost/biblioteca");
    }
    
    public void guardarLibro(Libro libro) {
        String sql = "INSERT INTO libros VALUES (?, ?, ?)";
        PreparedStatement ps = connection.prepareStatement(sql);
        ps.setString(1, libro.getTitulo());
        ps.setString(2, libro.getAutor());
        ps.setString(3, libro.getIsbn());
        ps.executeUpdate();
        
        // Enviar email de confirmacion
        Properties props = new Properties();
        props.put("mail.smtp.host", "smtp.gmail.com");
        Session session = Session.getDefaultInstance(props);
        Message msg = new MimeMessage(session);
        msg.setSubject("Nuevo libro: " + libro.getTitulo());
        Transport.send(msg);
        
        // Generar PDF
        Document document = new Document();
        PdfWriter.getInstance(document, new FileOutputStream("libro.pdf"));
        document.open();
        document.add(new Paragraph(libro.getTitulo()));
        document.close();
    }
    
    public List<Libro> buscarLibros() { ... }
    public void eliminarLibro(Long id) { ... }
    public void prestarLibro(Long libroId, Long usuarioId) { ... }
    public void devolverLibro(Long prestamoId) { ... }
}

Ver solucion

**Principios SOLID violados:**

**1. SRP (Single Responsibility Principle) - VIOLADO**

- La clase tiene multiples responsabilidades:
    
    - Acceso a base de datos
        
    - Envio de emails
        
    - Generacion de PDFs
        
    - Gestion de libros
        
    - Gestion de prestamos
        

**2. OCP (Open/Closed Principle) - VIOLADO**

- Para añadir nuevas funcionalidades hay que modificar la clase
    
- No hay abstracciones para extender comportamiento
    

**3. DIP (Dependency Inversion Principle) - VIOLADO**

- Dependencia directa de clases concretas (Connection, Session, Document)
    
- Crea sus propias dependencias en lugar de inyectarlas
    

**Refactorizacion sugerida:**

// Separar responsabilidades
@Service
public class LibroService {
    private final LibroRepository repository;
    private final EmailService emailService;
    private final PdfService pdfService;
    
    public LibroService(LibroRepository repository,
                        EmailService emailService,
                        PdfService pdfService) {
        this.repository = repository;
        this.emailService = emailService;
        this.pdfService = pdfService;
    }
    
    public Libro guardar(Libro libro) {
        Libro guardado = repository.save(libro);
        emailService.enviarNotificacion(libro);
        pdfService.generarFicha(libro);
        return guardado;
    }
}

@Service
public class PrestamoService {
    // Logica de prestamos separada
}

## Ejercicio 3: Diseno de Interfaces

### Enunciado

Disena las interfaces necesarias para un sistema de notificaciones que pueda enviar mensajes por diferentes canales (email, SMS, push). Aplica los principios ISP y OCP.

Requisitos:

- Algunos canales soportan adjuntos (email)
    
- Algunos canales soportan prioridad (push)
    
- Todos los canales deben poder enviar mensajes simples
    

Ver solucion

// Interface base (ISP - interfaces segregadas)
public interface NotificacionService {
    void enviar(String destinatario, String mensaje);
}

// Interface para canales con adjuntos
public interface NotificacionConAdjuntos extends NotificacionService {
    void enviarConAdjunto(String destinatario, String mensaje, File adjunto);
}

// Interface para canales con prioridad
public interface NotificacionConPrioridad extends NotificacionService {
    void enviarUrgente(String destinatario, String mensaje);
}

// Implementaciones (OCP - abiertas a extension)
@Service
public class EmailService implements NotificacionConAdjuntos {
    @Override
    public void enviar(String destinatario, String mensaje) {
        // Enviar email simple
    }
    
    @Override
    public void enviarConAdjunto(String destinatario, String mensaje, File adjunto) {
        // Enviar email con adjunto
    }
}

@Service
public class SmsService implements NotificacionService {
    @Override
    public void enviar(String destinatario, String mensaje) {
        // Enviar SMS
    }
}

@Service
public class PushService implements NotificacionConPrioridad {
    @Override
    public void enviar(String destinatario, String mensaje) {
        // Enviar push normal
    }
    
    @Override
    public void enviarUrgente(String destinatario, String mensaje) {
        // Enviar push con alta prioridad
    }
}

// Uso con DIP (depender de abstracciones)
@Service
public class AlertaService {
    private final List<NotificacionService> notificadores;
    
    public AlertaService(List<NotificacionService> notificadores) {
        this.notificadores = notificadores;
    }
    
    public void enviarATodos(String mensaje) {
        notificadores.forEach(n -> n.enviar("admin", mensaje));
    }
}

## Ejercicio 4: Ventajas e Inconvenientes

### Enunciado

Completa la siguiente tabla comparando una arquitectura monolitica vs una basada en componentes/microservicios:

|Aspecto|Monolitico|Componentes|
|---|---|---|
|Despliegue|?|?|
|Escalabilidad|?|?|
|Complejidad inicial|?|?|
|Mantenimiento|?|?|
|Testing|?|?|
|Rendimiento|?|?|

Ver solucion

|Aspecto|Monolitico|Componentes|
|---|---|---|
|**Despliegue**|Simple (un artefacto)|Complejo (multiples artefactos)|
|**Escalabilidad**|Escala todo junto|Escala componentes independientes|
|**Complejidad inicial**|Baja|Alta (infraestructura necesaria)|
|**Mantenimiento**|Dificil a medida que crece|Mas facil (cambios localizados)|
|**Testing**|Mas dificil aislar|Facil testing unitario|
|**Rendimiento**|Sin overhead de red|Overhead de comunicacion|

**Cuando elegir cada enfoque:**

- **Monolitico**: Proyectos pequeños, equipos pequeños, MVP, cuando la simplicidad es prioritaria.
    
- **Componentes/Microservicios**: Proyectos grandes, equipos multiples, necesidad de escalar partes especificas, tecnologias heterogeneas.