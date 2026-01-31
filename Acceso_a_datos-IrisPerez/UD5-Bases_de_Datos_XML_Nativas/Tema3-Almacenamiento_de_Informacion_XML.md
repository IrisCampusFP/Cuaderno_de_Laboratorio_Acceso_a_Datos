## CE 5c: Almacenar informacion en formato XML

En este tema aprenderemos a almacenar documentos XML en bases de datos nativas, gestionar colecciones, indices y validar con esquemas.

## 3.1 Estructura de Almacenamiento

### Jerarquia de Recursos

/db (raiz)
├── biblioteca/
│   ├── libros/
│   │   ├── libro_001.xml
│   │   ├── libro_002.xml
│   │   └── libro_003.xml
│   ├── autores/
│   │   └── autores.xml
│   └── prestamos/
│       └── prestamos_2025.xml
├── configuracion/
│   └── config.xml
└── apps/
    └── ...

### Tipos de Recursos

| Tipo               | Extension  | Descripcion                       |
| ------------------ | ---------- | --------------------------------- |
| **XMLResource**    | .xml       | Documento XML parseado            |
| **BinaryResource** | cualquiera | Ficheros binarios (imagenes, PDF) |
| **Collection**     | -          | Contenedor de recursos            |

## 3.2 Almacenar Documentos XML

### Metodo 1: Desde String

```java
public class AlmacenarDocumentos {
    
    public static void guardarXMLDesdeString(String colPath, 
            String nombreDoc, String contenidoXML) throws Exception {
        
        Collection col = ConexionExistDB.conectar(colPath);
        
        try {
            // Crear recurso XML
            XMLResource recurso = (XMLResource) col.createResource(
                nombreDoc, "XMLResource");
            
            // Establecer contenido
            recurso.setContent(contenidoXML);
            
            // Almacenar en la coleccion
            col.storeResource(recurso);
            
            System.out.println("Documento guardado: " + nombreDoc);
            
        } finally {
            ConexionExistDB.cerrar(col);
        }
    }
    
    public static void main(String[] args) throws Exception {
        String xml = """
            <?xml version="1.0" encoding="UTF-8"?>
            <libro id="1">
                <titulo>Don Quijote de la Mancha</titulo>
                <autor>Miguel de Cervantes</autor>
                <anio>1605</anio>
                <genero>Novela</genero>
                <disponible>true</disponible>
            </libro>
            """;
        
        guardarXMLDesdeString("/db/biblioteca/libros", 
            "libro_001.xml", xml);
    }
}

```

### Metodo 2: Desde Fichero

```java
import java.io.File;
import java.nio.file.Files;

public static void guardarXMLDesdeFichero(String colPath, 
        String nombreDoc, File fichero) throws Exception {
    
    Collection col = ConexionExistDB.conectar(colPath);
    
    try {
        XMLResource recurso = (XMLResource) col.createResource(
            nombreDoc, "XMLResource");
        
        // Leer contenido del fichero
        String contenido = Files.readString(fichero.toPath());
        recurso.setContent(contenido);
        
        col.storeResource(recurso);
        
    } finally {
        ConexionExistDB.cerrar(col);
    }
}

// Uso
File fichero = new File("datos/libros.xml");
guardarXMLDesdeFichero("/db/biblioteca", "libros.xml", fichero);
```

### Metodo 3: Desde DOM

```java
import org.w3c.dom.Document;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;

public static void guardarXMLDesdeDOM(String colPath, 
        String nombreDoc, Document doc) throws Exception {
    
    Collection col = ConexionExistDB.conectar(colPath);
    
    try {
        XMLResource recurso = (XMLResource) col.createResource(
            nombreDoc, "XMLResource");
        
        // Establecer contenido desde DOM
        recurso.setContentAsDOM(doc);
        
        col.storeResource(recurso);
        
    } finally {
        ConexionExistDB.cerrar(col);
    }
}
```

## 3.3 Recuperar Documentos XML

### Obtener Documento Completo

```java
public static String obtenerDocumento(String colPath, 
        String nombreDoc) throws Exception {
    
    Collection col = ConexionExistDB.conectar(colPath);
    
    try {
        XMLResource recurso = (XMLResource) col.getResource(nombreDoc);
        
        if (recurso == null) {
            throw new Exception("Documento no encontrado: " + nombreDoc);
        }
        
        return (String) recurso.getContent();
        
    } finally {
        ConexionExistDB.cerrar(col);
    }
}

// Uso
String xml = obtenerDocumento("/db/biblioteca/libros", "libro_001.xml");
System.out.println(xml);
```

### Obtener como DOM

```java
public static Document obtenerDocumentoDOM(String colPath, 
        String nombreDoc) throws Exception {
    
    Collection col = ConexionExistDB.conectar(colPath);
    
    try {
        XMLResource recurso = (XMLResource) col.getResource(nombreDoc);
        
        if (recurso == null) {
            return null;
        }
        
        return (Document) recurso.getContentAsDOM();
        
    } finally {
        ConexionExistDB.cerrar(col);
    }
}
```

## 3.4 Almacenamiento de Datos Estructurados

### Modelo de Datos: Biblioteca

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- biblioteca.xml -->
<biblioteca xmlns="http://biblioteca.ejemplo.com">
    
    <libros>
        <libro id="L001" isbn="978-84-376-0494-7">
            <titulo>Don Quijote de la Mancha</titulo>
            <autor ref="A001"/>
            <editorial>Catedra</editorial>
            <anio>1605</anio>
            <generos>
                <genero>Novela</genero>
                <genero>Clasico</genero>
            </generos>
            <ejemplares>
                <ejemplar codigo="E001" estado="disponible"/>
                <ejemplar codigo="E002" estado="prestado"/>
            </ejemplares>
        </libro>
        
        <libro id="L002" isbn="978-84-376-0495-4">
            <titulo>Cien anos de soledad</titulo>
            <autor ref="A002"/>
            <editorial>Sudamericana</editorial>
            <anio>1967</anio>
            <generos>
                <genero>Realismo magico</genero>
            </generos>
            <ejemplares>
                <ejemplar codigo="E003" estado="disponible"/>
            </ejemplares>
        </libro>
    </libros>
    
    <autores>
        <autor id="A001">
            <nombre>Miguel de Cervantes</nombre>
            <nacionalidad>Espanola</nacionalidad>
            <nacimiento>1547</nacimiento>
            <fallecimiento>1616</fallecimiento>
        </autor>
        
        <autor id="A002">
            <nombre>Gabriel Garcia Marquez</nombre>
            <nacionalidad>Colombiana</nacionalidad>
            <nacimiento>1927</nacimiento>
            <fallecimiento>2014</fallecimiento>
        </autor>
    </autores>
    
    <usuarios>
        <usuario id="U001">
            <nombre>Juan Garcia</nombre>
            <email>juan@email.com</email>
            <direccion>
                <calle>Calle Mayor 10</calle>
                <ciudad>Madrid</ciudad>
                <cp>28001</cp>
            </direccion>
        </usuario>
    </usuarios>
    
    <prestamos>
        <prestamo id="P001">
            <usuario ref="U001"/>
            <ejemplar ref="E002"/>
            <fecha_prestamo>2025-01-15</fecha_prestamo>
            <fecha_devolucion>2025-01-30</fecha_devolucion>
            <estado>activo</estado>
        </prestamo>
    </prestamos>
    
</biblioteca>
```

## 3.5 Carga Masiva de Datos

### Cargar Multiples Documentos

```java
public class CargaMasiva {
    
    public static void cargarDirectorio(String colPath, 
            File directorio) throws Exception {
        
        Collection col = ConexionExistDB.conectar(colPath);
        
        try {
            File[] ficheros = directorio.listFiles(
                (dir, name) -> name.endsWith(".xml"));
            
            if (ficheros == null) return;
            
            int contador = 0;
            for (File fichero : ficheros) {
                XMLResource recurso = (XMLResource) col.createResource(
                    fichero.getName(), "XMLResource");
                
                recurso.setContent(Files.readString(fichero.toPath()));
                col.storeResource(recurso);
                
                contador++;
                if (contador % 100 == 0) {
                    System.out.println("Cargados: " + contador);
                }
            }
            
            System.out.println("Total cargados: " + contador);
            
        } finally {
            ConexionExistDB.cerrar(col);
        }
    }
}
```

### Carga via REST (cURL)

```powershell
# Cargar un directorio completo
for file in datos/*.xml; do
    curl -u admin: -X PUT \
      -H "Content-Type: application/xml" \
      -d @"$file" \
      "http://localhost:8080/exist/rest/db/biblioteca/$(basename $file)"
done
```

## 3.6 Indices en eXist-db

### Tipos de Indices

|Tipo|Uso|Configuracion|
|---|---|---|
|**Estructural**|Navegacion XPath|Automatico|
|**Range**|Comparaciones (=, <, >)|collection.xconf|
|**Full-text**|Busqueda de texto|collection.xconf|
|**N-gram**|Busqueda parcial|collection.xconf|

### Configuracion de Indices

```xml
<!-- /db/system/config/db/biblioteca/collection.xconf -->
<collection xmlns="http://exist-db.org/collection-config/1.0">
    <index xmlns:xs="http://www.w3.org/2001/XMLSchema">
        
        <!-- Indices de rango -->
        <range>
            <create qname="titulo" type="xs:string"/>
            <create qname="anio" type="xs:integer"/>
            <create qname="@id" type="xs:string"/>
            <create qname="@isbn" type="xs:string"/>
        </range>
        
        <!-- Indice full-text con Lucene -->
        <lucene>
            <analyzer class="org.apache.lucene.analysis.standard.StandardAnalyzer"/>
            <text qname="titulo"/>
            <text qname="nombre"/>
            <text qname="descripcion"/>
        </lucene>
        
        <!-- Indice n-gram para busquedas parciales -->
        <ngram qname="titulo"/>
        
    </index>
</collection>

```
### Crear Configuracion via Java

```java
public static void configurarIndices(String colPath) throws Exception {
    String configPath = "/db/system/config" + colPath;
    
    String config = """
        <collection xmlns="http://exist-db.org/collection-config/1.0">
            <index xmlns:xs="http://www.w3.org/2001/XMLSchema">
                <range>
                    <create qname="titulo" type="xs:string"/>
                    <create qname="@id" type="xs:string"/>
                </range>
                <lucene>
                    <text qname="titulo"/>
                </lucene>
            </index>
        </collection>
        """;
    
    // Crear coleccion de configuracion si no existe
    // y guardar collection.xconf
    guardarXMLDesdeString(configPath, "collection.xconf", config);
}
```

## 3.7 Validacion con XSD

### Esquema XSD para Biblioteca

```xsd
<?xml version="1.0" encoding="UTF-8"?>
<!-- biblioteca.xsd -->
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    
    <xs:element name="libro">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="titulo" type="xs:string"/>
                <xs:element name="autor" type="xs:string"/>
                <xs:element name="anio" type="xs:gYear"/>
                <xs:element name="genero" type="xs:string" 
                            minOccurs="0" maxOccurs="unbounded"/>
                <xs:element name="disponible" type="xs:boolean"/>
            </xs:sequence>
            <xs:attribute name="id" type="xs:ID" use="required"/>
            <xs:attribute name="isbn" type="xs:string"/>
        </xs:complexType>
    </xs:element>
    
</xs:schema>
```

### Validar antes de Almacenar

```java
import javax.xml.validation.*;
import javax.xml.transform.stream.StreamSource;

public class ValidadorXML {
    
    private final Validator validator;
    
    public ValidadorXML(File esquemaXSD) throws Exception {
        SchemaFactory factory = SchemaFactory.newInstance(
            XMLConstants.W3C_XML_SCHEMA_NS_URI);
        Schema schema = factory.newSchema(esquemaXSD);
        this.validator = schema.newValidator();
    }
    
    public boolean validar(String xml) {
        try {
            validator.validate(new StreamSource(
                new StringReader(xml)));
            return true;
        } catch (Exception e) {
            System.err.println("Validacion fallida: " + e.getMessage());
            return false;
        }
    }
    
    public void guardarSiValido(String colPath, String nombreDoc, 
            String xml) throws Exception {
        if (validar(xml)) {
            AlmacenarDocumentos.guardarXMLDesdeString(
                colPath, nombreDoc, xml);
        } else {
            throw new Exception("XML no valido");
        }
    }
}
```

## 3.8 Almacenamiento de Recursos Binarios

### Guardar Imagen/PDF

```java
public static void guardarBinario(String colPath, 
        String nombreDoc, File fichero) throws Exception {
    
    Collection col = ConexionExistDB.conectar(colPath);
    
    try {
        BinaryResource recurso = (BinaryResource) col.createResource(
            nombreDoc, "BinaryResource");
        
        byte[] contenido = Files.readAllBytes(fichero.toPath());
        recurso.setContent(contenido);
        
        col.storeResource(recurso);
        
    } finally {
        ConexionExistDB.cerrar(col);
    }
}

// Uso
guardarBinario("/db/biblioteca/portadas", 
    "quijote.jpg", new File("portadas/quijote.jpg"));
```

## 3.9 Eliminar Documentos

```java
public static void eliminarDocumento(String colPath, 
        String nombreDoc) throws Exception {
    
    Collection col = ConexionExistDB.conectar(colPath);
    
    try {
        Resource recurso = col.getResource(nombreDoc);
        
        if (recurso != null) {
            col.removeResource(recurso);
            System.out.println("Eliminado: " + nombreDoc);
        } else {
            System.out.println("No existe: " + nombreDoc);
        }
        
    } finally {
        ConexionExistDB.cerrar(col);
    }
}
```

## 3.10 Resumen del Tema

Conceptos Clave

1. Los documentos XML se almacenan en **colecciones** organizadas jerarquicamente.
    
2. Se pueden guardar documentos desde **String**, **fichero** o **DOM**.
    
3. Los **indices** (range, full-text, n-gram) aceleran las consultas.
    
4. La configuracion de indices se hace en **collection.xconf**.
    
5. La **validacion XSD** garantiza la integridad de los datos.
    
6. Tambien se pueden almacenar **recursos binarios** (imagenes, PDFs).