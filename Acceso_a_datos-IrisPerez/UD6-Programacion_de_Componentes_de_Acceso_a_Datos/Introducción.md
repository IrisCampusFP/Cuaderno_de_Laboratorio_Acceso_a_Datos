Este modulo aborda el **Resultado de Aprendizaje 6 (RA6)** del modulo profesional **0486 - Acceso a Datos** del ciclo formativo de Desarrollo de Aplicaciones Multiplataforma (DAM).

**RA6: Programa componentes de acceso a datos identificando las caracteristicas que debe poseer un componente y utilizando herramientas de desarrollo.**

## Criterios de Evaluacion

|CE|Descripcion|Peso|
|---|---|---|
|6a|Valorar las ventajas e inconvenientes de utilizar programacion orientada a componentes|10%|
|6b|Identificar herramientas de desarrollo de componentes|10%|
|6c|Programar componentes que gestionan informacion almacenada en ficheros|10%|
|6d|Programar componentes que gestionan mediante conectores informacion almacenada en bases de datos|10%|
|6e|Programar componentes que gestionan informacion usando mapeo objeto relacional|10%|
|6f|Programar componentes que gestionan informacion almacenada en bases de datos objeto relacionales y orientadas a objetos|15%|
|6g|Programar componentes que gestionan informacion almacenada en una base de datos documental nativa|15%|
|6h|Probar y documentar los componentes desarrollados|10%|
|6i|Integrar los componentes desarrollados en aplicaciones|10%|

## Contenidos

### Teoria (Temas)

1. **Fundamentos de Componentes** - Programacion orientada a componentes, ventajas e inconvenientes
    
2. **Introduccion a Spring** - Spring Framework, IoC, DI, beans y contexto
    
3. **Spring Boot** - Autoconfig, starters, propiedades, perfiles
    
4. **Spring Data JPA** - Repositorios, Query Methods, consultas personalizadas
    
5. **Spring MVC y REST** - Controladores, servicios REST, DTOs
    
6. **Spring Security** - Autenticacion, autorizacion, JWT
    
7. **Integracion de Componentes** - Integracion de ficheros, JDBC, JPA, BDOO y NoSQL
    
8. **Pruebas y Documentacion** - JUnit, Mockito, Javadoc, Swagger/OpenAPI

### Actividades Practicas

1. **Fundamentos de Componentes** - Analisis de arquitecturas y patrones
    
2. **Primer Proyecto Spring Boot** - Configuracion y estructura basica
    
3. **Repositorios con Spring Data JPA** - CRUD y consultas avanzadas
    
4. **API REST con Spring MVC** - Desarrollo de servicios RESTful
    
5. **Seguridad en Aplicaciones** - Implementacion de autenticacion y autorizacion
    
6. **Integracion de Componentes** - Conexion con multiples fuentes de datos
    
7. **Pruebas de Componentes** - Testing unitario e integracion
    
8. **Proyecto Final Integrador** - Aplicacion completa con todos los componentes

## Tecnologias

- **Java 17+** - Lenguaje de programacion
    
- **Spring Framework 6.x** - Framework de desarrollo
    
- **Spring Boot 3.x** - Framework de desarrollo rapido
    
- **Spring Data JPA** - Acceso a datos con JPA
    
- **Spring MVC** - Desarrollo web y REST
    
- **Spring Security 6.x** - Seguridad de aplicaciones
    
- **Maven** - Gestion de proyectos
    
- **JUnit 5 / Mockito** - Testing
    
- **PostgreSQL / H2** - Bases de datos
    
- **Docker** - Contenedores

## Requisitos

- JDK 17 o superior
    
- Maven 3.8+
    
- Docker Desktop
    
- IDE (IntelliJ IDEA recomendado)
    
- Postman o similar para testing de APIs

## Estructura del Libro

RA6/
├── _config.yml              # Configuracion del libro
├── _toc.yml                 # Tabla de contenidos
├── content/
│   ├── intro.md             # Esta introduccion
│   ├── img/                 # Imagenes
│   ├── temas/               # Notebooks de teoria
│   ├── actividades/         # Notebooks de actividades
│   └── references.bib       # Bibliografia
├── aplicacion/              # Aplicacion Spring Boot de ejemplo
└── docker/                  # Configuracion Docker