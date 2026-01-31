## Resultado de Aprendizaje

**RA5:** Programa componentes de acceso a bases de datos nativas XML, identificando sus caracteristicas y utilizando conectores y librerias especificas.

## Informacion del Modulo

|Elemento|Valor|
|---|---|
|**Modulo**|0486 - Acceso a Datos|
|**Ciclo**|DAM (Desarrollo de Aplicaciones Multiplataforma)|
|**Curso**|2 DAM|
|**Peso del RA**|10% del modulo|
|**Horas estimadas**|20 horas|
|**Trimestre**|2|

## Criterios de Evaluacion

|CE|Descripcion|Peso|
|---|---|---|
|**5a**|Se han identificado las caracteristicas de las bases de datos nativas XML|15%|
|**5b**|Se han establecido conexiones con bases de datos XML nativas|15%|
|**5c**|Se ha almacenado informacion en formato XML|20%|
|**5d**|Se han desarrollado aplicaciones que realizan consultas XQuery/XPath|20%|
|**5e**|Se han modificado y eliminado datos almacenados|15%|
|**5f**|Se han utilizado librerias especificas de acceso a datos XML|15%|

## Contenidos

Este resultado de aprendizaje cubre los siguientes contenidos principales:

1. **Fundamentos de XML Nativas** - Caracteristicas, ventajas, estandares y comparativa con otras BBDD
    
2. **Conexiones** - Configuracion y gestion de conexiones a eXist-db y BaseX
    
3. **Almacenamiento XML** - Colecciones, documentos, indices y esquemas
    
4. **Consultas XQuery/XPath** - Expresiones XPath, FLWOR, funciones y agregaciones
    
5. **Modificaciones** - XQuery Update Facility, inserciones, actualizaciones y eliminaciones
    
6. **Librerias de Acceso** - XML:DB API, REST API, Java XML APIs

## Tecnologias Utilizadas

|Tecnologia|Version|Proposito|
|---|---|---|
|**eXist-db**|6.x|Base de datos XML nativa principal|
|**BaseX**|10.x|Base de datos XML nativa alternativa|
|**Java**|17+|Lenguaje de programacion|
|**XQuery**|3.1|Lenguaje de consultas XML|
|**XPath**|3.1|Expresiones de navegacion XML|
|**XML:DB API**|1.0|API estandar de acceso|
|**Maven**|3.9+|Gestion de dependencias|

## Comparativa de Tecnologias XML

|Caracteristica|eXist-db|BaseX|MarkLogic|
|---|---|---|---|
|**Licencia**|LGPL (Open Source)|BSD (Open Source)|Comercial|
|**XQuery**|3.1 completo|3.1 completo|1.0-ml extendido|
|**REST API**|Si|Si|Si|
|**Full-text search**|Si (Lucene)|Si|Si|
|**Indices**|Automaticos|Automaticos|Configurables|
|**Clustering**|Limitado|Limitado|Empresarial|
|**Uso**|Educativo/PYME|Educativo/PYME|Empresarial|

## Proyecto Practico

A lo largo de este RA desarrollaremos un **Sistema de Gestion de Biblioteca Digital** almacenando:

- Catalogo de libros en XML
    
- Autores y editoriales
    
- Prestamos y reservas
    
- Busquedas full-text
    
- Exportacion de datos

## Conexion con otros RAs

Este RA complementa los conocimientos adquiridos en:

- **RA1**: Manejo de ficheros XML con DOM/SAX
    
- **RA3**: ORM con JPA/Hibernate
    
- **RA4**: Bases de datos objeto-relacionales


> [!NOTE] Nota
> Las bases de datos XML nativas son especialmente utiles para almacenar documentos con estructura variable, como catalogos, configuraciones, contenido web y datos semiestructurados.

## Entorno de Trabajo

### Docker (recomendado)

```yml
# docker-compose.yml
version: '3.8'
services:
  existdb:
    image: existdb/existdb:6.2.0
    container_name: existdb
    ports:
      - "8080:8080"
    volumes:
      - existdb_data:/exist/data
    environment:
      - EXIST_ENV=development

volumes:
  existdb_data:
```

### Conexion desde Java

```java
// Conexion basica a eXist-db
String uri = "xmldb:exist://localhost:8080/exist/xmlrpc/db";
Collection col = DatabaseManager.getCollection(uri, "admin", "admin");
```