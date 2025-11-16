## Glosario de Términos

|Término|Definición|
|---|---|
|**ACID**|Propiedades de transacciones: Atomicity, Consistency, Isolation, Durability|
|**CallableStatement**|Interfaz para invocar procedimientos almacenados|
|**Connection**|Sesión activa con una base de datos|
|**DAO**|Data Access Object - Patrón de diseño para acceso a datos|
|**DatabaseMetaData**|Información sobre la estructura de la base de datos|
|**Driver**|Software que permite conectar Java con un DBMS específico|
|**HikariCP**|Pool de conexiones de alto rendimiento|
|**JDBC**|Java Database Connectivity - API para acceso a bases de datos|
|**PreparedStatement**|Sentencia SQL precompilada con parámetros|
|**ResultSet**|Conjunto de resultados de una consulta SELECT|
|**ResultSetMetaData**|Información sobre las columnas de un ResultSet|
|**Savepoint**|Punto intermedio en una transacción para rollback parcial|
|**Statement**|Sentencia SQL básica (no usar, usar PreparedStatement)|
|**Transaction**|Conjunto de operaciones que se ejecutan como unidad atómica|

## Enlaces Útiles

### Documentación Oficial

- [Java JDBC Tutorial (Oracle)](https://docs.oracle.com/javase/tutorial/jdbc/)
    
- [JDBC API Specification](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/module-summary.html)
    
- [MySQL Connector/J Documentation](https://dev.mysql.com/doc/connector-j/en/)
    
- [HikariCP GitHub](https://github.com/brettwooldridge/HikariCP)
    
- [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/)

### Libros Recomendados

1. **«JDBC Database Access with Java»** - Graham Hamilton, Rick Cattell, Maydene Fisher
    
2. **«Pro Java EE Performance Management and Optimization»** - Steven Haines
    
3. **«High-Performance Java Persistence»** - Vlad Mihalcea
    
4. **«Effective Java» (3rd Edition)** - Joshua Bloch (Capítulo sobre recursos)

### Tutoriales y Cursos

- [Baeldung - JDBC Guides](https://www.baeldung.com/java-jdbc)
    
- [Jenkov JDBC Tutorial](http://tutorials.jenkov.com/jdbc/index.html)
    
- [Vogella JDBC Tutorial](https://www.vogella.com/tutorials/MySQLJava/article.html)

## Herramientas Recomendadas

### IDEs

- **IntelliJ IDEA** Community Edition (recomendado)
    
- **Eclipse IDE** for Java Developers
    
- **VS Code** con Extension Pack for Java

### Gestores de Base de Datos

- **MySQL Workbench** (oficial de MySQL)
    
- **HeidiSQL** (ligero y potente)
    
- **DBeaver** (universal, soporta múltiples DBs)
    
- **DataGrip** (JetBrains, de pago)

### Control de Versiones

- **Git** + **GitHub** / **GitLab**
    
- **SourceTree** (GUI para Git)

## Troubleshooting Común

### Error: ClassNotFoundException: com.mysql.cj.jdbc.Driver

**Causa**: Driver MySQL no está en el classpath

**Solución**:

# Verificar que está en pom.xml
mvn dependency:tree | grep mysql

# Recompilar
mvn clean install

### Error: Communications link failure

**Causa**: MySQL no está corriendo o URL incorrecta

**Solución**:

# Verificar que MySQL está activo
sudo systemctl status mysql

# Verificar puerto
netstat -an | grep 3306

# Probar conexión
mysql -u root -p

### Error: Access denied for user

**Causa**: Credenciales incorrectas

**Solución**:

-- Crear usuario
CREATE USER 'techdam'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON techdam.* TO 'techdam'@'localhost';
FLUSH PRIVILEGES;

### Error: Table doesn’t exist

**Causa**: Script SQL no ejecutado o base de datos incorrecta

**Solución**:

# Ejecutar script
mysql -u root -p < techdam_completo.sql

# Verificar tablas
mysql -u root -p -e "USE techdam; SHOW TABLES;"

### Error: ResultSet is closed

**Causa**: Intentar usar ResultSet después de cerrar Connection

**Solución**: Procesa el ResultSet ANTES de cerrar la conexión

// ❌ Mal
List<String> nombres = new ArrayList<>();
try (Connection conn = ...; Statement stmt = ...;
     ResultSet rs = stmt.executeQuery("SELECT nombre FROM empleados")) {
    // No procesa aquí
}
// ¡rs está cerrado!
while (rs.next()) { ... }  // ERROR

// ✅ Bien
List<String> nombres = new ArrayList<>();
try (Connection conn = ...; Statement stmt = ...;
     ResultSet rs = stmt.executeQuery("SELECT nombre FROM empleados")) {
    while (rs.next()) {
        nombres.add(rs.getString("nombre"));
    }
} // Se cierra aquí
// Usar la lista después

## Comandos Útiles

### MySQL

-- Ver bases de datos
SHOW DATABASES;

-- Usar base de datos
USE techdam;

-- Ver tablas
SHOW TABLES;

-- Describir tabla
DESCRIBE empleados;

-- Ver procedimientos
SHOW PROCEDURE STATUS WHERE Db = 'techdam';

-- Ver funciones
SHOW FUNCTION STATUS WHERE Db = 'techdam';

-- Ver código de procedimiento
SHOW CREATE PROCEDURE incrementar_salario;

-- Ejecutar procedimiento desde MySQL
CALL incrementar_salario(1, 10.0, @anterior, @nuevo);
SELECT @anterior, @nuevo;

### Maven

# Compilar
mvn compile

# Limpiar y compilar
mvn clean compile

# Ejecutar clase Main
mvn exec:java -Dexec.mainClass="com.techdam.Main"

# Ver dependencias
mvn dependency:tree

# Crear JAR
mvn package

# Ejecutar tests
mvn test

### Git

# Inicializar repo
git init

# Añadir archivos
git add .

# Commit
git commit -m "Implementado CRUD de empleados"

# Ver historial
git log --oneline

# Crear rama
git checkout -b feature/procedimientos

# Subir a GitHub
git push origin main

## Conceptos Avanzados (Fuera del Curso)

Si quieres profundizar más allá de este curso:

### ORM (Object-Relational Mapping)

- **Hibernate** - ORM más popular de Java
    
- **JPA (Java Persistence API)** - Estándar de persistencia
    
- **Spring Data JPA** - Simplifica JPA con Spring

### NoSQL y Bases de Datos Alternativas

- **MongoDB** (documentos)
    
- **Redis** (clave-valor)
    
- **Cassandra** (columnas)
    
- **Neo4j** (grafos)

### Frameworks Web

- **Spring Boot** - Framework empresarial
    
- **Quarkus** - Framework nativo de la nube
    
- **Micronaut** - Microservicios de bajo consumo

### Testing

- **JUnit 5** - Framework de testing
    
- **Mockito** - Mocking para tests
    
- **H2 Database** - BD en memoria para tests
    
- **Testcontainers** - Contenedores Docker para tests

## Checklist de Mejores Prácticas

### Seguridad

- [ ] Usar PreparedStatement SIEMPRE
    
- [ ] No hardcodear credenciales
    
- [ ] Validar entradas del usuario
    
- [ ] Usar permisos mínimos en BD
    
- [ ] No exponer mensajes de error detallados al usuario

### Rendimiento

- [ ] Usar pool de conexiones (HikariCP)
    
- [ ] Cerrar recursos con try-with-resources
    
- [ ] Usar batch para inserciones múltiples
    
- [ ] Índices apropiados en tablas
    
- [ ] Evitar consultas N+1

### Mantenibilidad

- [ ] Separar capas (DAO, Service, Controller)
    
- [ ] Documentar con JavaDoc
    
- [ ] Nombres descriptivos de variables
    
- [ ] Manejar excepciones apropiadamente
    
- [ ] Escribir tests unitarios

### Arquitectura

- [ ] Patrón DAO para acceso a datos
    
- [ ] Separación de responsabilidades
    
- [ ] Configuración externalizada
    
- [ ] Inyección de dependencias
    
- [ ] Principios SOLID

## 📞 Soporte y Contacto

### Durante el Curso

- **Profesor**: [David Valbuena Segura](mailto:david.valbuena%40campusfp.es)
    
- **Horario tutorías**: [Horario]
    
- **Aula virtual**: [URL]

### Comunidad

- **Stack Overflow**: [Etiqueta JDBC](https://stackoverflow.com/questions/tagged/jdbc)
    
- **Reddit**: r/java, r/learnprogramming
    
- **Discord**: Java Programming


## 🎉 ¡Fin del Curso!

Enhorabuena por completar el curso de **JDBC Avanzado**. Ahora tienes las habilidades para:

✅ Conectar aplicaciones Java con bases de datos de forma profesional  
✅ Escribir código seguro contra inyección SQL  
✅ Gestionar transacciones complejas  
✅ Optimizar rendimiento con pools de conexiones  
✅ Usar procedimientos almacenados  
✅ Construir aplicaciones escalables

### Siguientes Pasos

1. **Practica** - Crea proyectos personales usando JDBC
    
2. **Explora** - Aprende JPA/Hibernate para el siguiente nivel
    
3. **Contribuye** - Comparte tu conocimiento en la comunidad
    
4. **Certifícate** - Considera certificaciones como Oracle Certified Professional


**¡Gracias por tu dedicación y éxito en tu carrera como desarrollador! 🚀**