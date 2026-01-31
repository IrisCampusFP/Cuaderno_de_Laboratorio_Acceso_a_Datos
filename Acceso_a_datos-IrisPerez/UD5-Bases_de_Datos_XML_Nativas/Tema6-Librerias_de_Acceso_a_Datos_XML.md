## CE 5f: Utilizar librerias especificas de acceso a datos XML

En este tema estudiaremos las diferentes librerias y APIs disponibles para acceder a bases de datos XML desde aplicaciones Java.

## 6.1 Panorama de APIs de Acceso

### APIs Disponibles

|API|Tipo|Uso Principal|
|---|---|---|
|**XML:DB**|Estandar|Acceso generico a NXD|
|**XQJ**|Estandar|XQuery desde Java|
|**REST**|HTTP|Acceso web/servicios|
|**JAXP**|Estandar|Procesamiento XML local|
|**Cliente nativo**|Propietario|Funciones avanzadas|

### Comparativa

|Caracteristica|XML:DB|XQJ|REST|Nativo|
|---|---|---|---|---|
|Portabilidad|Alta|Alta|Alta|Baja|
|Rendimiento|Medio|Alto|Medio|Alto|
|Complejidad|Media|Media|Baja|Alta|
|Funcionalidad|Completa|Consultas|Basica|Completa|

## 6.2 XML:DB API en Profundidad

### Arquitectura XML:DB

┌────────────────────────────────────────────┐
│            Aplicacion Java                 │
├────────────────────────────────────────────┤
│              XML:DB API                    │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │Collection│ │ Resource │ │ Service  │  │
│  └──────────┘ └──────────┘ └──────────┘  │
├────────────────────────────────────────────┤
│         Driver Especifico                  │
│  (eXist, BaseX, Sedna, etc.)              │
├────────────────────────────────────────────┤
│         Base de Datos XML                  │
└────────────────────────────────────────────┘

### Interfaces Principales

|Interface|Descripcion|
|---|---|
|`Database`|Registro del driver|
|`Collection`|Contenedor de recursos|
|`Resource`|Documento (XML o binario)|
|`XMLResource`|Documento XML|
|`BinaryResource`|Recurso binario|
|`Service`|Servicios adicionales|

### Servicios XML:DB

```java
// Servicio XQuery
XQueryService xquery = (XQueryService) col.getService(
    "XQueryService", "1.0");

// Servicio de gestion de colecciones
CollectionManagementService mgt = (CollectionManagementService) col.getService(
    "CollectionManagementService", "1.0");

// Servicio XPath (deprecated, usar XQuery)
XPathQueryService xpath = (XPathQueryService) col.getService(
    "XPathQueryService", "1.0");

// Servicio de transacciones (si soportado)
TransactionService tx = (TransactionService) col.getService(
    "TransactionService", "1.0");
```

### Clase Utility Completa

```java
public class XMLDBUtil {
    
    private static boolean driverRegistrado = false;
    
    public static synchronized void registrarDriver(String driverClass) 
            throws Exception {
        if (!driverRegistrado) {
            Class<?> cl = Class.forName(driverClass);
            Database database = (Database) cl.getDeclaredConstructor()
                .newInstance();
            DatabaseManager.registerDatabase(database);
            driverRegistrado = true;
        }
    }
    
    public static Collection getCollection(String uri, 
            String user, String password) throws XMLDBException {
        return DatabaseManager.getCollection(uri, user, password);
    }
    
    public static void closeQuietly(Collection col) {
        if (col != null) {
            try { col.close(); } catch (XMLDBException e) { }
        }
    }
    
    public static List<String> listarRecursos(Collection col) 
            throws XMLDBException {
        return Arrays.asList(col.listResources());
    }
    
    public static List<String> listarSubcolecciones(Collection col) 
            throws XMLDBException {
        return Arrays.asList(col.listChildCollections());
    }
}
```

## 6.3 Cliente REST Avanzado

### Clase Cliente REST Completa

```java
import java.net.http.*;
import java.net.URI;
import java.util.Base64;
import java.util.Optional;

public class ExistDBRestClient {
    
    private final String baseUrl;
    private final HttpClient client;
    private final String authHeader;
    
    public ExistDBRestClient(String host, int port, 
            String user, String password) {
        this.baseUrl = String.format("http://%s:%d/exist/rest", host, port);
        this.client = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(10))
            .build();
        this.authHeader = "Basic " + Base64.getEncoder()
            .encodeToString((user + ":" + password).getBytes());
    }
    
    // GET - Obtener documento
    public Optional<String> getDocument(String path) throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + path))
            .header("Authorization", authHeader)
            .GET()
            .build();
            
        HttpResponse<String> response = client.send(
            request, HttpResponse.BodyHandlers.ofString());
            
        if (response.statusCode() == 200) {
            return Optional.of(response.body());
        } else if (response.statusCode() == 404) {
            return Optional.empty();
        } else {
            throw new Exception("Error " + response.statusCode());
        }
    }
    
    // PUT - Crear/actualizar documento
    public void putDocument(String path, String content) throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + path))
            .header("Authorization", authHeader)
            .header("Content-Type", "application/xml")
            .PUT(HttpRequest.BodyPublishers.ofString(content))
            .build();
            
        HttpResponse<String> response = client.send(
            request, HttpResponse.BodyHandlers.ofString());
            
        if (response.statusCode() != 201 && response.statusCode() != 200) {
            throw new Exception("Error creando documento: " 
                + response.statusCode());
        }
    }
    
    // DELETE - Eliminar documento
    public boolean deleteDocument(String path) throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + path))
            .header("Authorization", authHeader)
            .DELETE()
            .build();
            
        HttpResponse<String> response = client.send(
            request, HttpResponse.BodyHandlers.ofString());
            
        return response.statusCode() == 200;
    }
    
    // POST - Ejecutar XQuery
    public String executeXQuery(String xquery) throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/db"))
            .header("Authorization", authHeader)
            .header("Content-Type", "application/xquery")
            .POST(HttpRequest.BodyPublishers.ofString(xquery))
            .build();
            
        HttpResponse<String> response = client.send(
            request, HttpResponse.BodyHandlers.ofString());
            
        return response.body();
    }
}
```

## 6.4 Patron DAO para XML

### Interface DAO Generica

```java
public interface XMLDao<T> {
    
    void guardar(T entidad) throws Exception;
    
    Optional<T> buscarPorId(String id) throws Exception;
    
    List<T> buscarTodos() throws Exception;
    
    void actualizar(T entidad) throws Exception;
    
    void eliminar(String id) throws Exception;
    
    List<T> buscar(String xquery) throws Exception;
}
```

### Implementacion LibroDAO

```java
public class LibroDAO implements XMLDao<Libro> {
    
    private static final String COLECCION = "/db/biblioteca/libros";
    
    @Override
    public void guardar(Libro libro) throws Exception {
        String xml = libro.toXML();
        String nombreDoc = "libro_" + libro.getId() + ".xml";
        
        Collection col = ConexionExistDB.conectar(COLECCION);
        try {
            XMLResource res = (XMLResource) col.createResource(
                nombreDoc, "XMLResource");
            res.setContent(xml);
            col.storeResource(res);
        } finally {
            ConexionExistDB.cerrar(col);
        }
    }
    
    @Override
    public Optional<Libro> buscarPorId(String id) throws Exception {
        String xquery = String.format(
            "collection('%s')//libro[@id='%s']", COLECCION, id);
        
        List<Libro> resultados = buscar(xquery);
        return resultados.isEmpty() ? 
            Optional.empty() : Optional.of(resultados.get(0));
    }
    
    @Override
    public List<Libro> buscarTodos() throws Exception {
        String xquery = String.format(
            "collection('%s')//libro", COLECCION);
        return buscar(xquery);
    }
    
    @Override
    public List<Libro> buscar(String xquery) throws Exception {
        List<Libro> libros = new ArrayList<>();
        
        Collection col = ConexionExistDB.conectar(COLECCION);
        try {
            XQueryService service = (XQueryService) col.getService(
                "XQueryService", "1.0");
            
            ResourceSet result = service.query(xquery);
            ResourceIterator it = result.getIterator();
            
            while (it.hasMoreResources()) {
                XMLResource res = (XMLResource) it.nextResource();
                Libro libro = Libro.fromXML((String) res.getContent());
                libros.add(libro);
            }
        } finally {
            ConexionExistDB.cerrar(col);
        }
        
        return libros;
    }
    
    @Override
    public void eliminar(String id) throws Exception {
        String update = String.format(
            "update delete collection('%s')//libro[@id='%s']", 
            COLECCION, id);
        
        Collection col = ConexionExistDB.conectar(COLECCION);
        try {
            XQueryService service = (XQueryService) col.getService(
                "XQueryService", "1.0");
            service.query(update);
        } finally {
            ConexionExistDB.cerrar(col);
        }
    }
    
    // Metodos adicionales
    public List<Libro> buscarDisponibles() throws Exception {
        String xquery = String.format(
            "collection('%s')//libro[disponible='true']", COLECCION);
        return buscar(xquery);
    }
    
    public List<Libro> buscarPorAutor(String autor) throws Exception {
        String xquery = String.format(
            "collection('%s')//libro[contains(autor, '%s')]", 
            COLECCION, autor);
        return buscar(xquery);
    }
}
```

## 6.5 Modelo de Dominio con XML

### Clase Libro

```java
public class Libro {
    
    private String id;
    private String isbn;
    private String titulo;
    private String autor;
    private int anio;
    private boolean disponible;
    
    // Constructores, getters, setters...
    
    public String toXML() {
        return String.format("""
            <libro id="%s" isbn="%s">
                <titulo>%s</titulo>
                <autor>%s</autor>
                <anio>%d</anio>
                <disponible>%s</disponible>
            </libro>
            """, id, isbn, titulo, autor, anio, disponible);
    }
    
    public static Libro fromXML(String xml) throws Exception {
        DocumentBuilder builder = DocumentBuilderFactory
            .newInstance().newDocumentBuilder();
        Document doc = builder.parse(
            new InputSource(new StringReader(xml)));
        
        Element root = doc.getDocumentElement();
        
        Libro libro = new Libro();
        libro.setId(root.getAttribute("id"));
        libro.setIsbn(root.getAttribute("isbn"));
        libro.setTitulo(getTextContent(root, "titulo"));
        libro.setAutor(getTextContent(root, "autor"));
        libro.setAnio(Integer.parseInt(getTextContent(root, "anio")));
        libro.setDisponible(
            Boolean.parseBoolean(getTextContent(root, "disponible")));
        
        return libro;
    }
    
    private static String getTextContent(Element parent, String tagName) {
        NodeList nodes = parent.getElementsByTagName(tagName);
        if (nodes.getLength() > 0) {
            return nodes.item(0).getTextContent();
        }
        return "";
    }
}
```

## 6.6 JAXB para Mapeo XML-Java

### Anotaciones JAXB

```java
import jakarta.xml.bind.annotation.*;

@XmlRootElement(name = "libro")
@XmlAccessorType(XmlAccessType.FIELD)
public class Libro {
    
    @XmlAttribute
    private String id;
    
    @XmlAttribute
    private String isbn;
    
    @XmlElement
    private String titulo;
    
    @XmlElement
    private String autor;
    
    @XmlElement
    private int anio;
    
    @XmlElement
    private boolean disponible;
    
    // Constructor, getters, setters...
}
```

### Serializacion/Deserializacion

```java
public class JAXBUtil {
    
    public static <T> String toXML(T objeto, Class<T> clase) 
            throws Exception {
        JAXBContext context = JAXBContext.newInstance(clase);
        Marshaller marshaller = context.createMarshaller();
        marshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);
        
        StringWriter writer = new StringWriter();
        marshaller.marshal(objeto, writer);
        return writer.toString();
    }
    
    public static <T> T fromXML(String xml, Class<T> clase) 
            throws Exception {
        JAXBContext context = JAXBContext.newInstance(clase);
        Unmarshaller unmarshaller = context.createUnmarshaller();
        
        return clase.cast(unmarshaller.unmarshal(
            new StringReader(xml)));
    }
}

// Uso
Libro libro = new Libro("L001", "978...", "Don Quijote", 
    "Cervantes", 1605, true);

String xml = JAXBUtil.toXML(libro, Libro.class);
Libro recovered = JAXBUtil.fromXML(xml, Libro.class);
```

## 6.7 Aplicacion Completa

### Estructura del Proyecto

biblioteca-xml/
├── pom.xml
├── src/main/java/com/biblioteca/
│   ├── App.java
│   ├── model/
│   │   ├── Libro.java
│   │   ├── Usuario.java
│   │   └── Prestamo.java
│   ├── dao/
│   │   ├── XMLDao.java
│   │   ├── LibroDAO.java
│   │   ├── UsuarioDAO.java
│   │   └── PrestamoDAO.java
│   ├── service/
│   │   └── BibliotecaService.java
│   └── util/
│       ├── ConexionExistDB.java
│       └── JAXBUtil.java
└── src/main/resources/
    └── existdb.properties

### Clase Principal

```java
public class App {
    
    public static void main(String[] args) {
        try {
            // Registrar driver
            XMLDBUtil.registrarDriver(
                "org.exist.xmldb.DatabaseImpl");
            
            // Crear DAO
            LibroDAO libroDAO = new LibroDAO();
            
            // Crear libro
            Libro libro = new Libro();
            libro.setId("L001");
            libro.setTitulo("Don Quijote");
            libro.setAutor("Cervantes");
            libro.setAnio(1605);
            libro.setDisponible(true);
            
            // Guardar
            libroDAO.guardar(libro);
            
            // Buscar
            List<Libro> disponibles = libroDAO.buscarDisponibles();
            disponibles.forEach(l -> 
                System.out.println(l.getTitulo()));
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 6.8 Resumen del Tema

Conceptos Clave

1. **XML:DB API** es el estandar para acceso a BD XML desde Java.
    
2. **REST API** permite acceso multiplataforma via HTTP.
    
3. El patron **DAO** encapsula el acceso a datos XML.
    
4. **JAXB** simplifica el mapeo objeto-XML con anotaciones.
    
5. Combinar **XML:DB + DAO + JAXB** proporciona una arquitectura limpia.
    
6. La **gestion de recursos** (try-finally) es esencial para evitar fugas.