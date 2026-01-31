## CE 5e: Modificar y eliminar datos almacenados

En este tema aprenderemos a modificar documentos XML almacenados usando XQuery Update Facility y la API XML:DB.

## 5.1 XQuery Update Facility

### Introduccion

**XQuery Update Facility (XQUF)** es una extension de XQuery que permite modificar documentos XML.

### Operaciones Disponibles

|Operacion|Descripcion|
|---|---|
|`insert`|Insertar nodos|
|`delete`|Eliminar nodos|
|`replace`|Reemplazar nodos|
|`rename`|Renombrar elementos/atributos|
|`replace value of`|Cambiar valor de nodo|

## 5.2 Operaciones INSERT

### Insertar Elemento

```xquery
(: Insertar un nuevo libro :)
let $nuevoLibro := 
    <libro id="L004">
        <titulo>El principito</titulo>
        <autor>Antoine de Saint-Exupery</autor>
        <anio>1943</anio>
        <disponible>true</disponible>
    </libro>
return
    update insert $nuevoLibro into doc('/db/biblioteca/libros.xml')//libros

```
### Posiciones de Insercion

```xquery
(: Insertar al final (into) :)
update insert <genero>Clasico</genero> 
    into //libro[@id='L001']

(: Insertar como primer hijo (as first into) :)
update insert <codigo>COD001</codigo> 
    as first into //libro[@id='L001']

(: Insertar como ultimo hijo (as last into) :)
update insert <nota>Edicion especial</nota> 
    as last into //libro[@id='L001']

(: Insertar antes de un nodo (before) :)
update insert <subtitulo>Primera parte</subtitulo> 
    before //libro[@id='L001']/autor

(: Insertar despues de un nodo (after) :)
update insert <editorial>Catedra</editorial> 
    after //libro[@id='L001']/autor
```

### Insertar Atributo

```xquery
(: Anadir atributo a elemento :)
update insert attribute idioma { 'es' } 
    into //libro[@id='L001']

(: Anadir multiples atributos :)
update insert (attribute formato { 'papel' }, attribute paginas { '500' }) 
    into //libro[@id='L001']
```

## 5.3 Operaciones DELETE

### Eliminar Elementos

```xquery
(: Eliminar un libro especifico :)
update delete //libro[@id='L004']

(: Eliminar todos los libros no disponibles :)
update delete //libro[disponible='false']

(: Eliminar un elemento hijo :)
update delete //libro[@id='L001']/genero

(: Eliminar multiples elementos :)
update delete //libro[@id='L001']/(genero | nota)
```

### Eliminar Atributos

```xquery
(: Eliminar un atributo :)
update delete //libro[@id='L001']/@idioma

(: Eliminar atributo de todos los libros :)
update delete //libro/@formato
```

## 5.4 Operaciones REPLACE

### Reemplazar Elemento Completo

```xquery
(: Reemplazar elemento completo :)
update replace //libro[@id='L001']/titulo 
    with <titulo>El ingenioso hidalgo Don Quijote de la Mancha</titulo>

(: Reemplazar con estructura mas compleja :)
update replace //libro[@id='L001']/autor 
    with 
        <autor>
            <nombre>Miguel</nombre>
            <apellido>de Cervantes Saavedra</apellido>
        </autor>

```
### Reemplazar Solo el Valor

```xquery
(: Cambiar solo el texto, no la estructura :)
update replace value of //libro[@id='L001']/titulo 
    with 'Don Quijote'

(: Cambiar valor de atributo :)
update replace value of //libro[@id='L001']/@id 
    with 'LIBRO001'

(: Cambiar disponibilidad :)
update replace value of //libro[@id='L001']/disponible 
    with 'false'
```

## 5.5 Operaciones RENAME

```xquery
(: Renombrar elemento :)
update rename //libro/anio as 'ano_publicacion'

(: Renombrar atributo :)
update rename //libro/@id as 'codigo'

(: Renombrar condicionalmente :)
for $libro in //libro[anio < 1900]
return update rename $libro as 'libro-clasico'
```

## 5.6 Modificaciones Multiples

### Bloque de Actualizaciones

```xquery
(: Multiples operaciones atomicas :)
let $libro := //libro[@id='L001']
return (
    update replace value of $libro/titulo with 'Don Quijote (Edicion 2025)',
    update insert attribute edicion { '2025' } into $libro,
    update delete $libro/nota
)
```

### Actualizacion Condicional

```xquery
(: Actualizar todos los libros antiguos :)
for $libro in //libro[anio < 1900]
return (
    update insert attribute clasico { 'true' } into $libro,
    update insert <categoria>Patrimonio literario</categoria> into $libro
)
```

### Incrementar Valores

```xquery
(: Incrementar contador de prestamos :)
let $libro := //libro[@id='L001']
let $prestamos := xs:integer($libro/@prestamos)
return 
    update replace value of $libro/@prestamos 
        with $prestamos + 1
```

## 5.7 Modificaciones desde Java

### Actualizar Documento Completo

```java
public class ModificacionesXML {
    
    public static void actualizarDocumento(String colPath, 
            String nombreDoc, String nuevoContenido) throws Exception {
        
        Collection col = ConexionExistDB.conectar(colPath);
        
        try {
            XMLResource recurso = (XMLResource) col.getResource(nombreDoc);
            
            if (recurso != null) {
                recurso.setContent(nuevoContenido);
                col.storeResource(recurso);
                System.out.println("Documento actualizado: " + nombreDoc);
            }
            
        } finally {
            ConexionExistDB.cerrar(col);
        }
    }
}
```

### Ejecutar XQuery Update

```java
public static void ejecutarUpdate(String colPath, 
        String xqueryUpdate) throws Exception {
    
    Collection col = ConexionExistDB.conectar(colPath);
    
    try {
        XQueryService service = (XQueryService) col.getService(
            "XQueryService", "1.0");
        
        service.query(xqueryUpdate);
        System.out.println("Update ejecutado correctamente");
        
    } finally {
        ConexionExistDB.cerrar(col);
    }
}

// Uso
String updateQuery = """
    update replace value of 
        doc('/db/biblioteca/libros/libro_001.xml')//disponible 
        with 'false'
    """;
ejecutarUpdate("/db", updateQuery);
```

### Clase de Servicio CRUD

```java
public class LibroService {
    
    private static final String COLECCION = "/db/biblioteca/libros";
    
    public void cambiarDisponibilidad(String libroId, 
            boolean disponible) throws Exception {
        
        String updateQuery = String.format("""
            update replace value of 
                //libro[@id='%s']/disponible 
                with '%s'
            """, libroId, disponible);
        
        ejecutarUpdate(COLECCION, updateQuery);
    }
    
    public void agregarGenero(String libroId, 
            String genero) throws Exception {
        
        String updateQuery = String.format("""
            update insert <genero>%s</genero> 
                into //libro[@id='%s']
            """, genero, libroId);
        
        ejecutarUpdate(COLECCION, updateQuery);
    }
    
    public void eliminarLibro(String libroId) throws Exception {
        String updateQuery = String.format(
            "update delete //libro[@id='%s']", libroId);
        
        ejecutarUpdate(COLECCION, updateQuery);
    }
}
```

## 5.8 Modificaciones via REST

### Actualizar con PUT

```powershell
# Reemplazar documento completo
curl -u admin: -X PUT \
  -H "Content-Type: application/xml" \
  -d '<libro id="L001"><titulo>Titulo actualizado</titulo></libro>' \
  http://localhost:8080/exist/rest/db/biblioteca/libro_001.xml

```
### Ejecutar XQuery Update via POST

```powershell
curl -u admin: -X POST \
  -H "Content-Type: application/xquery" \
  -d 'update replace value of doc("/db/biblioteca/libro_001.xml")//titulo with "Nuevo titulo"' \
  http://localhost:8080/exist/rest/db
```

### Eliminar con DELETE

```powershell
# Eliminar documento
curl -u admin: -X DELETE \
  http://localhost:8080/exist/rest/db/biblioteca/libro_004.xml
```

## 5.9 Transacciones y Concurrencia

### Bloqueo de Documentos

```java
public void actualizarConBloqueo(String colPath, String nombreDoc,
        Consumer<XMLResource> actualizacion) throws Exception {
    
    Collection col = ConexionExistDB.conectar(colPath);
    
    try {
        // Obtener con bloqueo exclusivo
        XMLResource recurso = (XMLResource) col.getResource(nombreDoc);
        
        if (recurso != null) {
            // Aplicar actualizacion
            actualizacion.accept(recurso);
            
            // Guardar cambios
            col.storeResource(recurso);
        }
        
    } finally {
        ConexionExistDB.cerrar(col);
    }
}
```

### Patron Optimistic Locking

```xquery
(: Agregar version al documento :)
<libro id="L001" version="1">
    ...
</libro>

(: Actualizar solo si la version coincide :)
let $libro := //libro[@id='L001']
let $versionActual := xs:integer($libro/@version)
where $versionActual = 1  (: version esperada :)
return (
    update replace value of $libro/@version with $versionActual + 1,
    update replace value of $libro/titulo with 'Nuevo titulo'
)

```
## 5.10 Resumen del Tema

Conceptos Clave

1. **XQuery Update Facility** permite modificar XML con operaciones declarativas.
    
2. Las operaciones principales son **insert**, **delete**, **replace** y **rename**.
    
3. **insert** puede posicionarse: into, as first, as last, before, after.
    
4. **replace value of** cambia solo el contenido, no la estructura.
    
5. Las modificaciones se ejecutan via **XQueryService** o **REST API**.
    
6. Para concurrencia, usar **bloqueos** o **versionado optimista**.