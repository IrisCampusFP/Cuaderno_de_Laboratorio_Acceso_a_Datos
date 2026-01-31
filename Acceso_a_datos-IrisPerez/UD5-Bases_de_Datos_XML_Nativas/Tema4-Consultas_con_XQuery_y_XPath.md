## CE 5d: Desarrollar aplicaciones que realizan consultas XQuery/XPath

En este tema aprenderemos XPath para navegar documentos XML y XQuery para realizar consultas complejas sobre bases de datos XML nativas.

## 4.1 Fundamentos de XPath

### Que es XPath?

**XPath (XML Path Language)** es un lenguaje para seleccionar nodos en documentos XML.

### Ejes de Navegacion

|Eje|Descripcion|Ejemplo|
|---|---|---|
|`/`|Raiz o hijo directo|`/biblioteca/libros`|
|`//`|Descendiente en cualquier nivel|`//libro`|
|`.`|Nodo actual|`./titulo`|
|`..`|Nodo padre|`../autor`|
|`@`|Atributo|`@id`, `@isbn`|
|`*`|Cualquier elemento|`/biblioteca/*`|
|`@*`|Cualquier atributo|`libro/@*`|

### Documento de Ejemplo

```xml
<biblioteca>
    <libros>
        <libro id="L001" isbn="978-84-376-0494-7">
            <titulo>Don Quijote de la Mancha</titulo>
            <autor>Miguel de Cervantes</autor>
            <anio>1605</anio>
            <disponible>true</disponible>
        </libro>
        <libro id="L002" isbn="978-84-376-0495-4">
            <titulo>Cien anos de soledad</titulo>
            <autor>Gabriel Garcia Marquez</autor>
            <anio>1967</anio>
            <disponible>true</disponible>
        </libro>
        <libro id="L003">
            <titulo>1984</titulo>
            <autor>George Orwell</autor>
            <anio>1949</anio>
            <disponible>false</disponible>
        </libro>
    </libros>
</biblioteca>
```

### Ejemplos de XPath

```xquery
(: Todos los libros :)
//libro

(: Titulos de todos los libros :)
//libro/titulo

(: Texto de los titulos :)
//libro/titulo/text()

(: Atributo id de los libros :)
//libro/@id

(: Primer libro :)
//libro[1]

(: Ultimo libro :)
//libro[last()]

(: Libros con isbn :)
//libro[@isbn]

(: Libro con id especifico :)
//libro[@id='L001']

(: Libros publicados despues de 1950 :)
//libro[anio > 1950]

(: Libros disponibles :)
//libro[disponible='true']

(: Libros de Cervantes :)
//libro[autor='Miguel de Cervantes']

(: Contar libros :)
count(//libro)
```

## 4.2 Predicados y Funciones XPath

### Predicados (Filtros)

```xquery
(: Predicado numerico - posicion :)
//libro[1]                    (: Primer libro :)
//libro[position() <= 2]      (: Primeros dos libros :)

(: Predicado de comparacion :)
//libro[anio = 1967]
//libro[anio != 1967]
//libro[anio > 1950]
//libro[anio >= 1950 and anio <= 2000]

(: Predicado de existencia :)
//libro[@isbn]                (: Libros que tienen isbn :)
//libro[not(@isbn)]           (: Libros sin isbn :)

(: Predicados combinados :)
//libro[anio > 1900 and disponible='true']
//libro[anio < 1700 or anio > 1950]
```

### Funciones de Cadena

```xquery
(: Contiene texto :)
//libro[contains(titulo, 'Quijote')]

(: Empieza por :)
//libro[starts-with(titulo, 'Don')]

(: Termina con :)
//libro[ends-with(autor, 'Marquez')]

(: Longitud :)
//libro[string-length(titulo) > 20]

(: Mayusculas/minusculas :)
//libro[lower-case(titulo) = 'don quijote de la mancha']
```

### Funciones Numericas y Agregacion

```xquery
count(//libro)                (: Numero de libros :)
sum(//libro/paginas)          (: Suma de paginas :)
avg(//libro/anio)             (: Promedio de anos :)
min(//libro/anio)             (: Ano minimo :)
max(//libro/anio)             (: Ano maximo :)
```

## 4.3 Fundamentos de XQuery

### Que es XQuery?

**XQuery** es un lenguaje funcional para consultar colecciones de datos XML. Extiende XPath con capacidades de programacion.

### Expresiones FLWOR

FLWOR = **F**or, **L**et, **W**here, **O**rder by, **R**eturn

```xquery
(: Estructura basica FLWOR :)
for $libro in //libro
let $titulo := $libro/titulo/text()
where $libro/anio > 1900
order by $libro/anio
return $titulo
```

### Clausulas FLWOR

|Clausula|Descripcion|Obligatorio|
|---|---|---|
|`for`|Itera sobre secuencia|Si*|
|`let`|Asigna variable|Si*|
|`where`|Filtra resultados|No|
|`order by`|Ordena resultados|No|
|`return`|Define salida|Si|

*Al menos `for` o `let` debe estar presente.

## 4.4 Ejemplos XQuery

### Consultas Basicas

```xquery
(: Listar todos los titulos :)
for $libro in //libro
return $libro/titulo/text()

(: Resultado: 
Don Quijote de la Mancha
Cien anos de soledad
1984
:)
```

```xquery
(: Libros con autor y ano :)
for $libro in //libro
return concat($libro/titulo, ' - ', $libro/autor, ' (', $libro/anio, ')')

(: Resultado:
Don Quijote de la Mancha - Miguel de Cervantes (1605)
Cien anos de soledad - Gabriel Garcia Marquez (1967)
1984 - George Orwell (1949)
:)
```

### Filtrado con WHERE

```xquery
(: Libros disponibles :)
for $libro in //libro
where $libro/disponible = 'true'
return $libro/titulo/text()

(: Libros del siglo XX :)
for $libro in //libro
where $libro/anio >= 1900 and $libro/anio < 2000
return $libro
```

### Ordenacion

```xquery
(: Ordenar por ano ascendente :)
for $libro in //libro
order by $libro/anio ascending
return $libro/titulo/text()

(: Ordenar por titulo descendente :)
for $libro in //libro
order by $libro/titulo descending
return $libro

(: Ordenacion multiple :)
for $libro in //libro
order by $libro/autor, $libro/anio descending
return $libro
```

## 4.5 Construccion de XML en XQuery

### Elementos Literales

```xquery
(: Construir nuevo XML :)
for $libro in //libro
where $libro/disponible = 'true'
return 
    <libro-resumen>
        <titulo>{$libro/titulo/text()}</titulo>
        <autor>{$libro/autor/text()}</autor>
    </libro-resumen>
```

### Envolver en Elemento Raiz

```xquery
(: Resultado como documento XML completo :)
<catalogo fecha="{current-date()}">
{
    for $libro in //libro
    order by $libro/titulo
    return 
        <item id="{$libro/@id}">
            {$libro/titulo}
            {$libro/autor}
        </item>
}
</catalogo>
```

### Constructores Computados

```xquery
(: Nombre de elemento dinamico :)
for $libro in //libro
let $tipo := if ($libro/anio < 1900) then 'clasico' else 'moderno'
return element {$tipo} { $libro/titulo/text() }

(: Resultado:
<clasico>Don Quijote de la Mancha</clasico>
<moderno>Cien anos de soledad</moderno>
<moderno>1984</moderno>
:)
```

## 4.6 Funciones y Agrupacion

### Funciones de Agregacion

```xquery
(: Estadisticas de la biblioteca :)
<estadisticas>
    <total-libros>{count(//libro)}</total-libros>
    <disponibles>{count(//libro[disponible='true'])}</disponibles>
    <anio-mas-antiguo>{min(//libro/anio)}</anio-mas-antiguo>
    <anio-mas-reciente>{max(//libro/anio)}</anio-mas-reciente>
</estadisticas>
```

### Agrupacion (group by)

```xquery
(: Contar libros por siglo :)
for $libro in //libro
let $siglo := ($libro/anio idiv 100) + 1
group by $siglo
return 
    <siglo numero="{$siglo}">
        <cantidad>{count($libro)}</cantidad>
        <titulos>{
            for $l in $libro
            return <titulo>{$l/titulo/text()}</titulo>
        }</titulos>
    </siglo>
```

### Distinct Values

```xquery
(: Autores unicos :)
distinct-values(//libro/autor)

(: Anos unicos ordenados :)
for $anio in distinct-values(//libro/anio)
order by $anio
return $anio
```

## 4.7 Consultas sobre Colecciones (eXist-db)

### Funciones de Coleccion

```xquery
(: Todos los documentos de una coleccion :)
collection('/db/biblioteca/libros')

(: Todos los libros de la coleccion :)
for $doc in collection('/db/biblioteca/libros')
return $doc//libro

(: Documento especifico :)
doc('/db/biblioteca/libros/libro_001.xml')

(: Elemento raiz de un documento :)
doc('/db/biblioteca/libros/libro_001.xml')/libro
```

### Busqueda Full-Text (eXist-db)

```xquery
(: Importar modulo Lucene :)
import module namespace ft="http://exist-db.org/xquery/lucene";

(: Busqueda full-text :)
for $libro in collection('/db/biblioteca')//libro
where ft:query($libro/titulo, 'quijote')
return $libro

(: Busqueda con relevancia :)
for $libro in collection('/db/biblioteca')//libro
let $score := ft:score($libro)
where ft:query($libro/titulo, 'mancha~')
order by $score descending
return 
    <resultado score="{$score}">
        {$libro/titulo}
    </resultado>
```

## 4.8 Ejecutar XQuery desde Java

### Con XML:DB API

```java
public class ConsultasXQuery {
    
    public static String ejecutarXQuery(String colPath, 
            String xquery) throws Exception {
        
        Collection col = ConexionExistDB.conectar(colPath);
        
        try {
            // Obtener servicio XQuery
            XQueryService service = (XQueryService) col.getService(
                "XQueryService", "1.0");
            
            // Ejecutar consulta
            ResourceSet result = service.query(xquery);
            
            // Procesar resultados
            StringBuilder sb = new StringBuilder();
            ResourceIterator it = result.getIterator();
            
            while (it.hasMoreResources()) {
                Resource res = it.nextResource();
                sb.append(res.getContent()).append("\n");
            }
            
            return sb.toString();
            
        } finally {
            ConexionExistDB.cerrar(col);
        }
    }
    
    public static void main(String[] args) throws Exception {
        String xquery = """
            for $libro in //libro
            where $libro/disponible = 'true'
            order by $libro/titulo
            return $libro/titulo/text()
            """;
        
        String resultado = ejecutarXQuery("/db/biblioteca", xquery);
        System.out.println(resultado);
    }
}
```

### Consulta Parametrizada

```java
public static ResourceSet consultaParametrizada(String colPath,
        String xquery, Map<String, Object> params) throws Exception {
    
    Collection col = ConexionExistDB.conectar(colPath);
    
    try {
        XQueryService service = (XQueryService) col.getService(
            "XQueryService", "1.0");
        
        // Compilar consulta
        CompiledExpression compiled = service.compile(xquery);
        
        // Establecer parametros
        for (Map.Entry<String, Object> param : params.entrySet()) {
            service.declareVariable(param.getKey(), param.getValue());
        }
        
        return service.execute(compiled);
        
    } finally {
        ConexionExistDB.cerrar(col);
    }
}

// Uso
String xquery = """
    declare variable $anio_min external;
    for $libro in //libro
    where $libro/anio >= $anio_min
    return $libro/titulo
    """;
    
Map<String, Object> params = Map.of("anio_min", 1900);
ResourceSet result = consultaParametrizada("/db/biblioteca", xquery, params);
```

## 4.9 Resumen del Tema

Conceptos Clave

1. **XPath** permite navegar y seleccionar nodos en documentos XML.
    
2. **XQuery** extiende XPath con expresiones FLWOR para consultas complejas.
    
3. Las clausulas **for, let, where, order by, return** permiten iterar, filtrar y transformar.
    
4. XQuery puede **construir nuevo XML** en la clausula return.
    
5. Las funciones **collection()** y **doc()** acceden a datos almacenados.
    
6. eXist-db proporciona **busqueda full-text** con funciones Lucene.