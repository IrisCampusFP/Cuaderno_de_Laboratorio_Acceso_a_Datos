## Resultado de Aprendizaje

**RA4:** Desarrolla aplicaciones que gestionan la información almacenada en bases de datos objeto-relacionales y orientadas a objetos valorando sus características y utilizando los mecanismos de acceso incorporados.

## Información del Módulo

|Elemento|Valor|
|---|---|
|**Módulo**|0486 - Acceso a Datos|
|**Ciclo**|DAM (Desarrollo de Aplicaciones Multiplataforma)|
|**Curso**|2 DAM|
|**Peso del RA**|5% del módulo|
|**Horas estimadas**|2 horas|
|**Trimestre**|1|

## Criterios de Evaluación

|CE|Descripción|Peso|
|---|---|---|
|**4a**|Se han identificado las ventajas e inconvenientes de las bases de datos que almacenan objetos|10%|
|**4b**|Se han establecido y cerrado conexiones|10%|
|**4c**|Se ha gestionado la persistencia de objetos simples|10%|
|**4d**|Se ha gestionado la persistencia de objetos estructurados|20%|
|**4e**|Se han desarrollado aplicaciones que realizan consultas|20%|
|**4f**|Se han modificado los objetos almacenados|10%|
|**4g**|Se han gestionado las transacciones|10%|
|**4h**|Se han probado y documentado las aplicaciones desarrolladas|10%|

## Contenidos

Este resultado de aprendizaje cubre los siguientes contenidos principales:

1. **Fundamentos de BDOO y BDOR** - Evolución, características, ventajas e inconvenientes
    
2. **Conexiones** - Configuración y gestión de conexiones a Oracle, PostgreSQL y ObjectDB
    
3. **Objetos Simples** - Tipos objeto básicos y persistencia con JPA
    
4. **Objetos Estructurados** - Tipos anidados, colecciones, herencia y relaciones
    
5. **Consultas** - SQL extendido para objetos y JPQL
    
6. **Modificaciones** - Actualización de objetos y colecciones
    
7. **Transacciones** - ACID, aislamiento y bloqueos
    
8. **Pruebas** - Testing con JUnit y documentación

## Tecnologías Utilizadas

|Tecnología|Versión|Propósito|
|---|---|---|
|**Oracle Database XE**|21c|ORDBMS empresarial|
|**PostgreSQL**|15+|ORDBMS open source|
|**ObjectDB**|2.8.9|OODBMS compatible JPA|
|**Java**|17+|Lenguaje de programación|
|**JPA**|3.1|API de persistencia|
|**Maven**|3.9+|Gestión de dependencias|
|**JUnit**|5.10+|Framework de testing|

## Comparativa de Tecnologías

|Característica|Oracle|PostgreSQL|ObjectDB|
|---|---|---|---|
|**Tipo**|ORDBMS|ORDBMS|OODBMS|
|**Licencia**|Comercial|Open Source|Comercial/Free|
|**Tipos objeto**|CREATE TYPE AS OBJECT|CREATE TYPE (compuesto)|Clases Java|
|**Herencia tipos**|UNDER|INHERITS (tablas)|Herencia Java|
|**Métodos en tipos**|Sí|No|Métodos Java|
|**Colecciones**|VARRAY, NESTED TABLE|Arrays|Collections Java|
|**Uso empresarial**|Muy alto|Alto|Nicho|

## Proyecto Práctico

A lo largo de este RA desarrollaremos un **Sistema de Gestión de Biblioteca Multimedia** que permite demostrar:

- Herencia de tipos (recurso base con especializaciones: Libro, DVD)
    
- Objetos embebidos (dirección de usuarios)
    
- Colecciones (teléfonos, historial de préstamos)
    
- Relaciones complejas (préstamos con recursos y usuarios)
    
- Métodos en tipos objeto

## Conexión con RA3

Este RA complementa los conocimientos de JPA/Hibernate adquiridos en RA3, mostrando que las mismas APIs pueden usarse con diferentes tipos de bases de datos. Además, introduce las capacidades nativas de Oracle y PostgreSQL para el manejo de objetos.

Nota

Oracle es el SGBD más utilizado en grandes empresas en España (banca, seguros, administración pública). PostgreSQL es la alternativa open source más potente y cada vez más adoptada.