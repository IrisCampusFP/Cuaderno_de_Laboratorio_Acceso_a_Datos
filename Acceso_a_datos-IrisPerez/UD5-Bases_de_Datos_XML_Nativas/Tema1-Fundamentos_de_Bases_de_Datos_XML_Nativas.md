## CE 5a: Identificar las caracteristicas de las bases de datos nativas XML

En este tema estudiaremos los fundamentos teoricos de las bases de datos XML nativas, sus caracteristicas, ventajas, inconvenientes y casos de uso apropiados.

## 1.1 Introduccion a XML y Bases de Datos

### Que es XML?

**XML (eXtensible Markup Language)** es un metalenguaje que permite definir lenguajes de marcado personalizados para estructurar, almacenar y transportar datos.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<biblioteca>
    <libro id="1" isbn="978-84-376-0494-7">
        <titulo>Don Quijote de la Mancha</titulo>
        <autor>Miguel de Cervantes</autor>
        <anio>1605</anio>
        <genero>Novela</genero>
    </libro>
    <libro id="2" isbn="978-84-376-0495-4">
        <titulo>Cien anos de soledad</titulo>
        <autor>Gabriel Garcia Marquez</autor>
        <anio>1967</anio>
        <genero>Realismo magico</genero>
    </libro>
</biblioteca>
```

### Caracteristicas de XML

|Caracteristica|Descripcion|
|---|---|
|**Autodesriptivo**|Los datos incluyen metadatos sobre su estructura|
|**Extensible**|Se pueden crear etiquetas personalizadas|
|**Jerarquico**|Estructura de arbol con anidamiento|
|**Independiente**|Separacion entre datos y presentacion|
|**Estandar**|Definido por W3C, ampliamente soportado|

## 1.2 Tipos de Almacenamiento XML

Existen diferentes formas de almacenar datos XML:

### 1. XML en Sistema de Ficheros

Almacenar documentos XML como ficheros en disco.

**Ventajas:**

- Simple y directo
    
- Sin dependencias externas

**Inconvenientes:**

- Sin indices
    
- Consultas ineficientes
    
- Sin concurrencia

### 2. XML en RDBMS (XML-Enabled)

Bases de datos relacionales con soporte XML (Oracle XML DB, SQL Server, PostgreSQL).

```postgresql
-- PostgreSQL con tipo XML
CREATE TABLE documentos (
    id SERIAL PRIMARY KEY,
    contenido XML
);

-- Oracle XML DB
CREATE TABLE libros OF XMLTYPE;
```

### 3. Bases de Datos XML Nativas (NXD)

Sistemas disenados especificamente para almacenar, consultar y gestionar XML.

**Ejemplos:** eXist-db, BaseX, MarkLogic, Sedna

## 1.3 Que es una Base de Datos XML Nativa?

Una **Base de Datos XML Nativa (NXD - Native XML Database)** es un sistema de gestion de bases de datos que:

1. **Define un modelo logico** para documentos XML
    
2. **Almacena y recupera** documentos segun ese modelo
    
3. **Tiene XML como unidad fundamental** de almacenamiento

### Definicion del XML:DB Initiative

> «Una base de datos XML nativa define un modelo logico para un documento XML y almacena y recupera documentos de acuerdo con ese modelo. Como minimo, el modelo debe incluir elementos, atributos, PCDATA y orden del documento.»

### Modelo de Datos XDM (XQuery Data Model)

El estandar W3C define el modelo de datos para XML:

|Tipo de Nodo|Descripcion|Ejemplo|
|---|---|---|
|**Document**|Nodo raiz del documento|Todo el documento|
|**Element**|Elemento XML|`<libro>...</libro>`|
|**Attribute**|Atributo de elemento|`id="1"`|
|**Text**|Contenido de texto|`Don Quijote`|
|**Comment**|Comentario XML|`<!-- comentario -->`|
|**Processing Instruction**|Instruccion de procesamiento|`<?xml-stylesheet?>`|

## 1.4 Arquitectura de una Base de Datos XML Nativa

### Componentes Principales

┌─────────────────────────────────────────────────────┐
│                   APLICACION                        │
├─────────────────────────────────────────────────────┤
│              API de Acceso                          │
│    (XML:DB API, REST, XQuery API)                   │
├─────────────────────────────────────────────────────┤
│           Motor de Consultas XQuery                 │
│    (Parser, Optimizador, Evaluador)                 │
├─────────────────────────────────────────────────────┤
│              Gestor de Indices                      │
│    (Estructurales, Texto completo, Range)           │
├─────────────────────────────────────────────────────┤
│              Motor de Almacenamiento                │
│    (Colecciones, Documentos, Transacciones)         │
├─────────────────────────────────────────────────────┤
│              Sistema de Ficheros                    │
└─────────────────────────────────────────────────────┘

### Conceptos Clave

|Concepto|Descripcion|
|---|---|
|**Coleccion**|Contenedor de documentos XML (similar a carpeta)|
|**Documento**|Unidad fundamental de almacenamiento|
|**Indice**|Estructura para acelerar consultas|
|**XQuery**|Lenguaje de consultas estandar W3C|

## 1.5 Ventajas de las Bases de Datos XML Nativas

### 1. Modelo de Datos Natural

- Los documentos se almacenan tal cual, sin descomposicion
    
- Se preserva la estructura jerarquica original
    
- No hay «impedance mismatch» con XML

### 2. Esquema Flexible

- Documentos con diferentes estructuras en la misma coleccion
    
- Evolucion del esquema sin migraciones complejas
    
- Schema-less o validacion opcional con XSD

### 3. Consultas Potentes

- XQuery es un lenguaje completo Turing
    
- Navegacion natural con XPath
    
- Transformaciones XSLT integradas

### 4. Busqueda Full-Text

- Indices de texto completo integrados
    
- Busqueda por relevancia
    
- Soporte para multiples idiomas

### 5. Estandares W3C

- XQuery, XPath, XSLT son estandares
    
- Portabilidad entre diferentes NXD
    
- Amplia documentacion y comunidad

## 1.6 Inconvenientes de las Bases de Datos XML Nativas

### 1. Rendimiento en Consultas Complejas

- JOINs entre documentos pueden ser lentos
    
- Consultas sobre grandes volumenes requieren optimizacion

### 2. Curva de Aprendizaje

- XQuery es diferente a SQL
    
- Requiere entender XPath y el modelo de datos XML

### 3. Ecosistema Menor

- Menos herramientas que para RDBMS
    
- Menor comunidad de desarrolladores
    
- Pocas opciones de hosting gestionado

### 4. Integracion

- Menos drivers y conectores disponibles
    
- Integracion con BI/Reporting limitada

### 5. No Optimo para Datos Tabulares

- Para datos puramente tabulares, RDBMS es mas eficiente
    
- Overhead de XML para datos simples

## 1.7 Comparativa: XML Nativa vs RDBMS vs NoSQL

|Caracteristica|XML Nativa|RDBMS|MongoDB (Document)|
|---|---|---|---|
|**Modelo**|Documentos XML|Tablas|Documentos JSON|
|**Esquema**|Flexible (XSD opcional)|Rigido|Flexible|
|**Lenguaje**|XQuery/XPath|SQL|MQL|
|**Relaciones**|Referencias/Anidamiento|Foreign Keys|Referencias/Embedded|
|**Full-text**|Nativo|Extension|Nativo|
|**Transacciones**|Si|Si (ACID)|Si (desde v4.0)|
|**Escalabilidad**|Vertical|Vertical|Horizontal|
|**Caso de uso**|Documentos estructurados|Datos tabulares|Datos semi-estructurados|

## 1.8 Casos de Uso de Bases de Datos XML Nativas

### Casos Ideales

|Dominio|Justificacion|
|---|---|
|**Gestion documental**|Documentos con estructura variable|
|**Publicacion digital**|Libros, revistas, articulos en XML|
|**Configuraciones**|Ficheros de configuracion complejos|
|**Intercambio B2B**|EDI, facturacion electronica|
|**Datos cientificos**|Resultados experimentales estructurados|
|**Contenido web (CMS)**|Paginas y contenido dinamico|
|**Catalogos de productos**|Atributos variables por categoria|

### Cuando NO usar XML Nativa

- Datos puramente tabulares sin jerarquia
    
- Alto volumen de transacciones OLTP simples
    
- Cuando se requiere amplio ecosistema de herramientas BI
    
- Equipo sin conocimientos de XQuery/XPath

## 1.9 Principales Bases de Datos XML Nativas

### eXist-db

- **Licencia**: LGPL (Open Source)
    
- **Lenguaje**: Java
    
- **XQuery**: 3.1 completo
    
- **Caracteristicas**: REST API, WebDAV, triggers, indices Lucene
    
- **Uso**: Educacion, proyectos medianos, prototipado

```powershell
# Docker
docker run -d -p 8080:8080 existdb/existdb:latest
```

### BaseX

- **Licencia**: BSD (Open Source)
    
- **Lenguaje**: Java
    
- **XQuery**: 3.1 completo, muy rapido
    
- **Caracteristicas**: GUI incluida, servidor HTTP, cliente standalone
    
- **Uso**: Analisis de datos XML, investigacion

```powershell
# Docker
docker run -d -p 1984:1984 -p 8984:8984 basex/basexhttp:latest
```

### MarkLogic

- **Licencia**: Comercial
    
- **Caracteristicas**: Multi-modelo (XML, JSON, RDF), escalabilidad empresarial
    
- **Uso**: Grandes empresas, gobierno, finanzas

## 1.10 Estandares y Tecnologias Relacionadas

### Estandares W3C

|Estandar|Descripcion|Version|
|---|---|---|
|**XML**|Lenguaje de marcado extensible|1.0/1.1|
|**XPath**|Expresiones de navegacion|3.1|
|**XQuery**|Lenguaje de consultas|3.1|
|**XSLT**|Transformaciones|3.0|
|**XSD**|Esquemas XML|1.1|
|**XQuery Update**|Modificacion de datos|3.0|

### APIs de Acceso

|API|Descripcion|
|---|---|
|**XML:DB**|API Java estandar para NXD|
|**REST**|Acceso HTTP estandar|
|**XQJ**|XQuery API for Java|
|**WebDAV**|Acceso como sistema de ficheros|

## 1.11 Resumen del Tema

Conceptos Clave

1. Las **bases de datos XML nativas** almacenan XML como unidad fundamental, preservando estructura y orden.
    
2. Ofrecen **esquema flexible** y consultas potentes con XQuery/XPath.
    
3. Son ideales para **documentos estructurados** con estructura variable.
    
4. **eXist-db** y **BaseX** son las opciones open source mas populares.
    
5. Los **estandares W3C** (XQuery, XPath, XSLT) garantizan portabilidad.
    
6. No son optimas para datos puramente tabulares o alto volumen OLTP.