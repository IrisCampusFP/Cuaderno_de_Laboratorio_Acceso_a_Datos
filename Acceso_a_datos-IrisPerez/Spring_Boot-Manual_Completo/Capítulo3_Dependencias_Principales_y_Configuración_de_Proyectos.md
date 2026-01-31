## 3.1 Estructura de Proyecto Educativo Recomendada

La estructura estándar Maven/Gradle recomendada por la documentación oficial de Spring Boot y recursos educativos sigue consistentemente:

src/
├── main/
│   ├── java/
│   │   └── com/ejemplo/aplicacion/
│   │       ├── AplicacionPrincipal.java
│   │       ├── controller/
│   │       │   └── EstudianteController.java
│   │       ├── service/
│   │       │   └── EstudianteService.java
│   │       ├── repository/
│   │       │   └── EstudianteRepository.java
│   │       ├── model/
│   │       │   └── Estudiante.java
│   │       └── config/
│   │           └── ConfiguracionBase.java
│   └── resources/
│       ├── static/
│       ├── templates/
│       ├── application.properties
│       └── data.sql
└── test/
    └── java/
        └── com/ejemplo/aplicacion/
            └── AplicacionPrincipalTest.java

## 3.2 Dependencias Fundamentales para Acceso a Datos

**Configuración Maven esencial para proyectos educativos:**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.5.6</version>
    <relativePath/>
</parent>

<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <maven.compiler.parameters>true</maven.compiler.parameters>
</properties>

<dependencies>
    <!-- Starter web para desarrollo de APIs REST -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Acceso a datos con JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- Base de datos H2 para desarrollo -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- Seguridad -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <!-- Validación de datos -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    
    <!-- Herramientas de desarrollo -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    
    <!-- Testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- Actuator para monitoreo -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

## 3.3 Spring Data JPA vs JDBC - Criterios de Decisión Educativa

**Matriz de decisión para proyectos educativos:**

|**Criterio**|**Spring Data JPA**|**Spring Data JDBC**|**Spring JDBC**|
|---|---|---|---|
|**Curva de aprendizaje**|Moderada a Alta|Baja a Moderada|Baja|
|**Rendimiento**|Bueno (con caché)|Muy Bueno|Excelente|
|**Características**|Características ORM ricas|ORM básico|Sin ORM|
|**Complejidad**|Alta|Baja|Media|
|**Caso de uso**|Modelos de dominio complejos|Operaciones CRUD simples|Requisitos SQL personalizados|

**Cuándo elegir cada enfoque:**

**Spring Data JPA** es apropiado para relaciones de objetos complejas (One-to-Many, Many-to-Many), necesidad de generación automática de consultas, cuando el aprendizaje de conceptos ORM es objetivo educativo, y requisitos de portabilidad de base de datos.

**Spring Data JDBC** funciona mejor para modelos de dominio simples con relaciones mínimas, cuando el rendimiento es crítico, para entender la ejecución SQL, y cuando se prefiere control explícito sobre operaciones de base de datos.