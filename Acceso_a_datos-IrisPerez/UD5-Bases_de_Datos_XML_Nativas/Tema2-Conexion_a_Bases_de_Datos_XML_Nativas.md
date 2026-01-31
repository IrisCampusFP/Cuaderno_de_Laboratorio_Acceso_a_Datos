## CE 5b: Establecer conexiones con bases de datos XML nativas

En este tema aprenderemos a configurar y establecer conexiones con bases de datos XML nativas como eXist-db y BaseX utilizando diferentes metodos de acceso.

## 2.1 Metodos de Acceso a Bases de Datos XML

### Opciones de Conexion

|Metodo|Protocolo|Uso Principal|
|---|---|---|
|**XML:DB API**|XMLRPC|Aplicaciones Java|
|**REST API**|HTTP|Servicios web, cualquier lenguaje|
|**WebDAV**|HTTP|Gestion de ficheros|
|**Cliente nativo**|TCP|Administracion, scripts|

### Arquitectura de Conexion

┌─────────────────┐     ┌─────────────────┐
│  Aplicacion     │     │   eXist-db      │
│     Java        │────>│    Server       │
│                 │     │                 │
│  XML:DB API     │     │  Puerto 8080    │
└─────────────────┘     └─────────────────┘

┌─────────────────┐     ┌─────────────────┐
│  Cliente HTTP   │     │   BaseX         │
│  (cualquier)    │────>│   REST API      │
│                 │     │                 │
│  REST/JSON      │     │  Puerto 8984    │
└─────────────────┘     └─────────────────┘

## 2.2 Instalacion de eXist-db con Docker

### Docker Compose

```yml
# docker-compose.yml
version: '3.8'

services:
  existdb:
    image: existdb/existdb:6.2.0
    container_name: existdb
    ports:
      - "8080:8080"      # Dashboard y REST
      - "8443:8443"      # HTTPS
    volumes:
      - existdb_data:/exist/data
    environment:
      - EXIST_ENV=development
    restart: unless-stopped

volumes:
  existdb_data:
```

### Iniciar el Contenedor

```yml
# Iniciar eXist-db
docker-compose up -d

# Verificar estado
docker-compose ps

# Ver logs
docker-compose logs -f existdb
```

### Acceso al Dashboard

- **URL**: [http://localhost:8080/exist/apps/dashboard/](http://localhost:8080/exist/apps/dashboard/)
    
- **Usuario**: admin
    
- **Password**: (vacio por defecto, configurar en primer acceso)

## 2.3 Instalacion de BaseX con Docker

### Docker Compose

```yml
# docker-compose.yml
version: '3.8'

services:
  basex:
    image: basex/basexhttp:latest
    container_name: basex
    ports:
      - "1984:1984"      # Servidor BaseX
      - "8984:8984"      # REST/HTTP
    volumes:
      - basex_data:/srv/basex/data
    environment:
      - BASEX_JVM=-Xmx2g
    restart: unless-stopped

volumes:
  basex_data:
```

### Credenciales por Defecto

- **Usuario**: admin
    
- **Password**: admin

### Acceso REST

```yml
# Verificar conexion
curl -u admin:admin http://localhost:8984/rest
```

## 2.4 Conexion con XML:DB API (Java)

### Dependencias Maven

```xml
<dependencies>
    <!-- eXist-db XML:DB driver -->
    <dependency>
        <groupId>org.exist-db</groupId>
        <artifactId>exist-core</artifactId>
        <version>6.2.0</version>
    </dependency>
    
    <!-- XML:DB API -->
    <dependency>
        <groupId>xmldb</groupId>
        <artifactId>xmldb-api</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```

### Clase de Conexion

```java
import org.xmldb.api.DatabaseManager;
import org.xmldb.api.base.*;
import org.xmldb.api.modules.*;

public class ConexionExistDB {
    
    private static final String DRIVER = "org.exist.xmldb.DatabaseImpl";
    private static final String URI = "xmldb:exist://localhost:8080/exist/xmlrpc";
    private static final String USER = "admin";
    private static final String PASSWORD = "";
    
    public static Collection conectar(String coleccion) throws Exception {
        // Registrar el driver
        Class<?> cl = Class.forName(DRIVER);
        Database database = (Database) cl.getDeclaredConstructor().newInstance();
        DatabaseManager.registerDatabase(database);
        
        // Obtener la coleccion
        String fullUri = URI + coleccion;
        Collection col = DatabaseManager.getCollection(fullUri, USER, PASSWORD);
        
        if (col == null) {
            throw new Exception("No se pudo conectar a: " + fullUri);
        }
        
        return col;
    }
    
    public static void cerrar(Collection col) {
        if (col != null) {
            try {
                col.close();
            } catch (XMLDBException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### Ejemplo de Uso

```java
public class TestConexion {
    
    public static void main(String[] args) {
        Collection col = null;
        
        try {
            // Conectar a la coleccion /db
            col = ConexionExistDB.conectar("/db");
            
            System.out.println("Conexion exitosa!");
            System.out.println("Nombre coleccion: " + col.getName());
            System.out.println("Numero de recursos: " + col.getResourceCount());
            
            // Listar recursos
            String[] recursos = col.listResources();
            for (String recurso : recursos) {
                System.out.println("  - " + recurso);
            }
            
            // Listar subcolecciones
            String[] subcolecciones = col.listChildCollections();
            for (String sub : subcolecciones) {
                System.out.println("  [DIR] " + sub);
            }
            
        } catch (Exception e) {
            System.err.println("Error: " + e.getMessage());
            e.printStackTrace();
        } finally {
            ConexionExistDB.cerrar(col);
        }
    }
}
```

**Salida esperada:**

```cmd
Conexion exitosa!
Nombre coleccion: db
Numero de recursos: 0
  [DIR] apps
  [DIR] system
```

## 2.5 Conexion REST con eXist-db

### Endpoints REST

|Metodo|Endpoint|Descripcion|
|---|---|---|
|GET|`/exist/rest/db/...`|Obtener documento/coleccion|
|PUT|`/exist/rest/db/...`|Crear/actualizar documento|
|DELETE|`/exist/rest/db/...`|Eliminar documento|
|POST|`/exist/rest/db`|Ejecutar XQuery|

### Ejemplo con cURL

```XQuery
# Listar coleccion raiz
curl -u admin: http://localhost:8080/exist/rest/db

# Obtener documento
curl -u admin: http://localhost:8080/exist/rest/db/biblioteca/libros.xml

# Crear documento
curl -u admin: -X PUT \
  -H "Content-Type: application/xml" \
  -d '<libro><titulo>Nuevo libro</titulo></libro>' \
  http://localhost:8080/exist/rest/db/test.xml

# Ejecutar XQuery
curl -u admin: -X POST \
  -H "Content-Type: application/xquery" \
  -d 'for $x in //libro return $x/titulo' \
  http://localhost:8080/exist/rest/db
```

### Cliente REST en Java

```java
import java.net.http.*;
import java.net.URI;
import java.util.Base64;

public class ExistDBRestClient {
    
    private static final String BASE_URL = "http://localhost:8080/exist/rest";
    private static final String USER = "admin";
    private static final String PASSWORD = "";
    
    private final HttpClient client;
    private final String authHeader;
    
    public ExistDBRestClient() {
        this.client = HttpClient.newHttpClient();
        String credentials = USER + ":" + PASSWORD;
        this.authHeader = "Basic " + Base64.getEncoder()
            .encodeToString(credentials.getBytes());
    }
    
    public String getDocument(String path) throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(BASE_URL + path))
            .header("Authorization", authHeader)
            .GET()
            .build();
            
        HttpResponse<String> response = client.send(
            request, HttpResponse.BodyHandlers.ofString());
            
        if (response.statusCode() != 200) {
            throw new Exception("Error: " + response.statusCode());
        }
        
        return response.body();
    }
    
    public String executeXQuery(String xquery) throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(BASE_URL + "/db"))
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

## 2.6 Conexion a BaseX

### Cliente Java Nativo

```xml
<dependency>
    <groupId>org.basex</groupId>
    <artifactId>basex</artifactId>
    <version>10.7</version>
</dependency>
```

```java
import org.basex.api.client.ClientSession;

public class BaseXConexion {
    
    public static void main(String[] args) {
        try (ClientSession session = new ClientSession(
                "localhost", 1984, "admin", "admin")) {
            
            // Crear base de datos
            session.execute("CREATE DB biblioteca");
            
            // Ejecutar XQuery
            String result = session.execute("XQUERY 1 + 1");
            System.out.println("Resultado: " + result);
            
            // Listar bases de datos
            String dbs = session.execute("LIST");
            System.out.println("Bases de datos:\n" + dbs);
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### REST API de BaseX

```xquery
# Listar bases de datos
curl -u admin:admin http://localhost:8984/rest

# Ejecutar XQuery
curl -u admin:admin \
  "http://localhost:8984/rest?query=1+to+10"

# Crear base de datos desde XML
curl -u admin:admin -X PUT \
  -H "Content-Type: application/xml" \
  -d '<root><item>test</item></root>' \
  http://localhost:8984/rest/testdb
```

## 2.7 Gestion de Colecciones

### Crear Colecciones en eXist-db

```java
public class GestionColecciones {
    
    public static void crearColeccion(String padre, String nombre) 
            throws Exception {
        Collection col = ConexionExistDB.conectar(padre);
        
        try {
            // Obtener servicio de gestion
            CollectionManagementService mgt = 
                (CollectionManagementService) col.getService(
                    "CollectionManagementService", "1.0");
            
            // Crear coleccion
            mgt.createCollection(nombre);
            System.out.println("Coleccion creada: " + padre + "/" + nombre);
            
        } finally {
            ConexionExistDB.cerrar(col);
        }
    }
    
    public static void eliminarColeccion(String padre, String nombre) 
            throws Exception {
        Collection col = ConexionExistDB.conectar(padre);
        
        try {
            CollectionManagementService mgt = 
                (CollectionManagementService) col.getService(
                    "CollectionManagementService", "1.0");
            
            mgt.removeCollection(nombre);
            System.out.println("Coleccion eliminada: " + nombre);
            
        } finally {
            ConexionExistDB.cerrar(col);
        }
    }
    
    public static void main(String[] args) throws Exception {
        // Crear estructura de colecciones
        crearColeccion("/db", "biblioteca");
        crearColeccion("/db/biblioteca", "libros");
        crearColeccion("/db/biblioteca", "autores");
        crearColeccion("/db/biblioteca", "prestamos");
    }
}
```

## 2.8 Pool de Conexiones

### Implementacion Simple

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class XMLDBConnectionPool {
    
    private final BlockingQueue<Collection> pool;
    private final String uri;
    private final String user;
    private final String password;
    
    public XMLDBConnectionPool(String uri, String user, String password, 
            int poolSize) throws Exception {
        this.uri = uri;
        this.user = user;
        this.password = password;
        this.pool = new ArrayBlockingQueue<>(poolSize);
        
        // Inicializar pool
        for (int i = 0; i < poolSize; i++) {
            pool.offer(createConnection());
        }
    }
    
    private Collection createConnection() throws Exception {
        return DatabaseManager.getCollection(uri, user, password);
    }
    
    public Collection getConnection() throws InterruptedException {
        return pool.take();
    }
    
    public void releaseConnection(Collection col) {
        if (col != null) {
            pool.offer(col);
        }
    }
    
    public void close() {
        for (Collection col : pool) {
            try {
                col.close();
            } catch (XMLDBException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 2.9 Manejo de Errores

### Excepciones Comunes

|Excepcion|Causa|Solucion|
|---|---|---|
|`XMLDBException`|Error general de la BD|Verificar mensaje detallado|
|`ConnectException`|Servidor no disponible|Verificar que eXist-db esta corriendo|
|`AuthenticationException`|Credenciales invalidas|Verificar usuario/password|
|`PermissionDeniedException`|Sin permisos|Verificar permisos de coleccion|

### Patron de Manejo de Errores

```java
public class SafeXMLDBOperation {
    
    public static <T> T execute(String colPath, 
            XMLDBOperation<T> operation) {
        Collection col = null;
        
        try {
            col = ConexionExistDB.conectar(colPath);
            return operation.execute(col);
            
        } catch (XMLDBException e) {
            System.err.println("Error XMLDB [" + e.errorCode + "]: " 
                + e.getMessage());
            throw new RuntimeException(e);
            
        } catch (Exception e) {
            System.err.println("Error: " + e.getMessage());
            throw new RuntimeException(e);
            
        } finally {
            ConexionExistDB.cerrar(col);
        }
    }
}

@FunctionalInterface
interface XMLDBOperation<T> {
    T execute(Collection col) throws Exception;
}

// Uso
String[] recursos = SafeXMLDBOperation.execute("/db/biblioteca", 
    col -> col.listResources());
```

## 2.10 Resumen del Tema

Conceptos Clave

1. Las conexiones a BD XML pueden ser via **XML:DB API**, **REST** o **cliente nativo**.
    
2. **Docker** simplifica la instalacion de eXist-db y BaseX.
    
3. La **XML:DB API** es el estandar Java para acceso a BD XML nativas.
    
4. Las **colecciones** son contenedores de documentos (similar a directorios).
    
5. Un **pool de conexiones** mejora el rendimiento en aplicaciones concurrentes.
    
6. El **manejo de errores** debe contemplar excepciones de conexion, autenticacion y permisos.