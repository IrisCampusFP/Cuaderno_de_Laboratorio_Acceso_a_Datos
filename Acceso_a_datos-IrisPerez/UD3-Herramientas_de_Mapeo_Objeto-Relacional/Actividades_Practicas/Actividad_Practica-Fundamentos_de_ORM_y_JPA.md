## Objetivos

- Configurar un proyecto JPA desde cero
    
- Crear entidades basicas con anotaciones JPA
    
- Realizar operaciones CRUD usando EntityManager
    
- Comprender el mapeo objeto-relacional

## Ejercicio 1: Configuracion del proyecto

### Enunciado

Crea un archivo `persistence.xml` para configurar una unidad de persistencia llamada `biblioteca-pu` que:

- Use Hibernate como proveedor JPA
    
- Se conecte a una base de datos H2 en memoria
    
- Genere automaticamente las tablas (hbm2ddl.auto = create)
    
- Muestre las consultas SQL en consola

Ver solucion

<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="https://jakarta.ee/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence
             https://jakarta.ee/xml/ns/persistence/persistence_3_0.xsd"
             version="3.0">
    
    <persistence-unit name="biblioteca-pu" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        
        <properties>
            <!-- Conexion a H2 en memoria -->
            <property name="jakarta.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="jakarta.persistence.jdbc.url" value="jdbc:h2:mem:biblioteca;DB_CLOSE_DELAY=-1"/>
            <property name="jakarta.persistence.jdbc.user" value="sa"/>
            <property name="jakarta.persistence.jdbc.password" value=""/>
            
            <!-- Dialecto Hibernate -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            
            <!-- Generacion automatica de tablas -->
            <property name="hibernate.hbm2ddl.auto" value="create"/>
            
            <!-- Mostrar SQL en consola -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
        </properties>
    </persistence-unit>
</persistence>

**Explicacion:**

- `transaction-type="RESOURCE_LOCAL"`: Gestion manual de transacciones (sin servidor de aplicaciones)
    
- `hbm2ddl.auto=create`: Crea las tablas cada vez que se inicia la aplicacion
    
- `show_sql` y `format_sql`: Muestran las consultas SQL formateadas

## Ejercicio 2: Creacion de entidad basica

### Enunciado

Crea una entidad `Libro` con los siguientes atributos:

- `id`: Long, clave primaria autogenerada
    
- `isbn`: String, unico y no nulo, maximo 13 caracteres
    
- `titulo`: String, no nulo, maximo 200 caracteres
    
- `autor`: String, maximo 100 caracteres
    
- `precio`: BigDecimal con precision 10 y escala 2
    
- `fechaPublicacion`: LocalDate
    
- `disponible`: Boolean con valor por defecto true

La tabla debe llamarse `libros`.

Ver solucion

import jakarta.persistence.*;
import java.math.BigDecimal;
import java.time.LocalDate;

@Entity
@Table(name = "libros")
public class Libro {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false, length = 13)
    private String isbn;
    
    @Column(nullable = false, length = 200)
    private String titulo;
    
    @Column(length = 100)
    private String autor;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal precio;
    
    @Column(name = "fecha_publicacion")
    private LocalDate fechaPublicacion;
    
    @Column(columnDefinition = "boolean default true")
    private Boolean disponible = true;
    
    // Constructor vacio requerido por JPA
    public Libro() {}
    
    // Constructor con parametros
    public Libro(String isbn, String titulo, String autor) {
        this.isbn = isbn;
        this.titulo = titulo;
        this.autor = autor;
    }
    
    // Getters y Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getIsbn() { return isbn; }
    public void setIsbn(String isbn) { this.isbn = isbn; }
    
    public String getTitulo() { return titulo; }
    public void setTitulo(String titulo) { this.titulo = titulo; }
    
    public String getAutor() { return autor; }
    public void setAutor(String autor) { this.autor = autor; }
    
    public BigDecimal getPrecio() { return precio; }
    public void setPrecio(BigDecimal precio) { this.precio = precio; }
    
    public LocalDate getFechaPublicacion() { return fechaPublicacion; }
    public void setFechaPublicacion(LocalDate fechaPublicacion) { 
        this.fechaPublicacion = fechaPublicacion; 
    }
    
    public Boolean getDisponible() { return disponible; }
    public void setDisponible(Boolean disponible) { this.disponible = disponible; }
    
    @Override
    public String toString() {
        return "Libro{id=" + id + ", isbn='" + isbn + "', titulo='" + titulo + "'}";
    }
}

**Puntos clave:**

- @Entity marca la clase como entidad JPA
    
- `@Table(name = "libros")` define el nombre de la tabla
    
- @Id y @GeneratedValue definen la clave primaria autogenerada
    
- @Column permite personalizar el mapeo de cada campo
    
- El constructor vacio es obligatorio para JPA
    

## Ejercicio 3: Operaciones CRUD basicas

### Enunciado

Implementa una clase `LibroDAO` que contenga los siguientes metodos:

1. `guardar(Libro libro)`: Persiste un nuevo libro
    
2. `buscarPorId(Long id)`: Busca un libro por su ID
    
3. `buscarTodos()`: Devuelve todos los libros
    
4. `actualizar(Libro libro)`: Actualiza un libro existente
    
5. `eliminar(Long id)`: Elimina un libro por su ID
    

Usa EntityManager y gestiona las transacciones correctamente.

Ver solucion

import jakarta.persistence.*;
import java.util.List;

public class LibroDAO {
    
    private EntityManagerFactory emf;
    
    public LibroDAO() {
        this.emf = Persistence.createEntityManagerFactory("biblioteca-pu");
    }
    
    // CREATE
    public Libro guardar(Libro libro) {
        EntityManager em = emf.createEntityManager();
        try {
            em.getTransaction().begin();
            em.persist(libro);
            em.getTransaction().commit();
            return libro;
        } catch (Exception e) {
            if (em.getTransaction().isActive()) {
                em.getTransaction().rollback();
            }
            throw e;
        } finally {
            em.close();
        }
    }
    
    // READ por ID
    public Libro buscarPorId(Long id) {
        EntityManager em = emf.createEntityManager();
        try {
            return em.find(Libro.class, id);
        } finally {
            em.close();
        }
    }
    
    // READ todos
    public List<Libro> buscarTodos() {
        EntityManager em = emf.createEntityManager();
        try {
            return em.createQuery("SELECT l FROM Libro l", Libro.class)
                     .getResultList();
        } finally {
            em.close();
        }
    }
    
    // UPDATE
    public Libro actualizar(Libro libro) {
        EntityManager em = emf.createEntityManager();
        try {
            em.getTransaction().begin();
            Libro libroActualizado = em.merge(libro);
            em.getTransaction().commit();
            return libroActualizado;
        } catch (Exception e) {
            if (em.getTransaction().isActive()) {
                em.getTransaction().rollback();
            }
            throw e;
        } finally {
            em.close();
        }
    }
    
    // DELETE
    public void eliminar(Long id) {
        EntityManager em = emf.createEntityManager();
        try {
            em.getTransaction().begin();
            Libro libro = em.find(Libro.class, id);
            if (libro != null) {
                em.remove(libro);
            }
            em.getTransaction().commit();
        } catch (Exception e) {
            if (em.getTransaction().isActive()) {
                em.getTransaction().rollback();
            }
            throw e;
        } finally {
            em.close();
        }
    }
    
    // Cerrar factory al terminar
    public void cerrar() {
        if (emf != null && emf.isOpen()) {
            emf.close();
        }
    }
}

**Buenas practicas aplicadas:**

- Gestion de transacciones con try-catch-finally
    
- Rollback en caso de error
    
- Cierre del EntityManager en finally
    
- `persist()` para nuevas entidades, `merge()` para actualizaciones
    
- `find()` antes de `remove()` para evitar errores
    

## Ejercicio 4: Programa principal de prueba

### Enunciado

Crea un programa principal que:

1. Cree 3 libros y los guarde en la base de datos
    
2. Busque un libro por ID y muestre sus datos
    
3. Actualice el precio de un libro
    
4. Liste todos los libros
    
5. Elimine un libro
    
6. Vuelva a listar para verificar la eliminacion
    

Ver solucion

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.List;

public class Main {
    
    public static void main(String[] args) {
        LibroDAO dao = new LibroDAO();
        
        try {
            // 1. Crear y guardar libros
            System.out.println("=== CREANDO LIBROS ===");
            
            Libro libro1 = new Libro("978-0134685991", "Effective Java", "Joshua Bloch");
            libro1.setPrecio(new BigDecimal("45.99"));
            libro1.setFechaPublicacion(LocalDate.of(2018, 1, 6));
            dao.guardar(libro1);
            
            Libro libro2 = new Libro("978-0596009205", "Head First Design Patterns", "Eric Freeman");
            libro2.setPrecio(new BigDecimal("39.99"));
            libro2.setFechaPublicacion(LocalDate.of(2004, 10, 25));
            dao.guardar(libro2);
            
            Libro libro3 = new Libro("978-0132350884", "Clean Code", "Robert C. Martin");
            libro3.setPrecio(new BigDecimal("35.99"));
            libro3.setFechaPublicacion(LocalDate.of(2008, 8, 1));
            dao.guardar(libro3);
            
            System.out.println("Libros creados: " + libro1.getId() + ", " + 
                              libro2.getId() + ", " + libro3.getId());
            
            // 2. Buscar por ID
            System.out.println("\n=== BUSCAR POR ID ===");
            Libro encontrado = dao.buscarPorId(1L);
            System.out.println("Libro encontrado: " + encontrado);
            System.out.println("Precio: " + encontrado.getPrecio());
            
            // 3. Actualizar precio
            System.out.println("\n=== ACTUALIZAR PRECIO ===");
            encontrado.setPrecio(new BigDecimal("49.99"));
            dao.actualizar(encontrado);
            System.out.println("Precio actualizado a: " + encontrado.getPrecio());
            
            // 4. Listar todos
            System.out.println("\n=== LISTAR TODOS ===");
            List<Libro> libros = dao.buscarTodos();
            libros.forEach(l -> System.out.println("  - " + l));
            
            // 5. Eliminar libro
            System.out.println("\n=== ELIMINAR LIBRO ID=2 ===");
            dao.eliminar(2L);
            System.out.println("Libro eliminado");
            
            // 6. Verificar eliminacion
            System.out.println("\n=== LISTAR DESPUES DE ELIMINAR ===");
            libros = dao.buscarTodos();
            libros.forEach(l -> System.out.println("  - " + l));
            System.out.println("Total libros: " + libros.size());
            
        } finally {
            dao.cerrar();
        }
    }
}

**Salida esperada:**

=== CREANDO LIBROS ===
Libros creados: 1, 2, 3

=== BUSCAR POR ID ===
Libro encontrado: Libro{id=1, isbn='978-0134685991', titulo='Effective Java'}
Precio: 45.99

=== ACTUALIZAR PRECIO ===
Precio actualizado a: 49.99

=== LISTAR TODOS ===
  - Libro{id=1, isbn='978-0134685991', titulo='Effective Java'}
  - Libro{id=2, isbn='978-0596009205', titulo='Head First Design Patterns'}
  - Libro{id=3, isbn='978-0132350884', titulo='Clean Code'}

=== ELIMINAR LIBRO ID=2 ===
Libro eliminado

=== LISTAR DESPUES DE ELIMINAR ===
  - Libro{id=1, isbn='978-0134685991', titulo='Effective Java'}
  - Libro{id=3, isbn='978-0132350884', titulo='Clean Code'}
Total libros: 2

## Ejercicio 5: Consultas con JPQL

### Enunciado

Anade los siguientes metodos de consulta al `LibroDAO`:

1. `buscarPorAutor(String autor)`: Busca libros por autor (coincidencia parcial)
    
2. `buscarPorRangoPrecio(BigDecimal min, BigDecimal max)`: Libros en un rango de precio
    
3. `buscarDisponibles()`: Solo libros disponibles ordenados por titulo
    
4. `contarPorAutor(String autor)`: Cuenta libros de un autor
    

Ver solucion

// Anadir estos metodos a LibroDAO

public List<Libro> buscarPorAutor(String autor) {
    EntityManager em = emf.createEntityManager();
    try {
        return em.createQuery(
            "SELECT l FROM Libro l WHERE LOWER(l.autor) LIKE LOWER(:autor)", 
            Libro.class)
            .setParameter("autor", "%" + autor + "%")
            .getResultList();
    } finally {
        em.close();
    }
}

public List<Libro> buscarPorRangoPrecio(BigDecimal min, BigDecimal max) {
    EntityManager em = emf.createEntityManager();
    try {
        return em.createQuery(
            "SELECT l FROM Libro l WHERE l.precio BETWEEN :min AND :max ORDER BY l.precio", 
            Libro.class)
            .setParameter("min", min)
            .setParameter("max", max)
            .getResultList();
    } finally {
        em.close();
    }
}

public List<Libro> buscarDisponibles() {
    EntityManager em = emf.createEntityManager();
    try {
        return em.createQuery(
            "SELECT l FROM Libro l WHERE l.disponible = true ORDER BY l.titulo", 
            Libro.class)
            .getResultList();
    } finally {
        em.close();
    }
}

public Long contarPorAutor(String autor) {
    EntityManager em = emf.createEntityManager();
    try {
        return em.createQuery(
            "SELECT COUNT(l) FROM Libro l WHERE LOWER(l.autor) LIKE LOWER(:autor)", 
            Long.class)
            .setParameter("autor", "%" + autor + "%")
            .getSingleResult();
    } finally {
        em.close();
    }
}

**Conceptos JPQL aplicados:**

- `LIKE` con `%` para coincidencias parciales
    
- `LOWER()` para busqueda case-insensitive
    
- `BETWEEN` para rangos
    
- `ORDER BY` para ordenacion
    
- `COUNT()` para agregaciones
    
- Parametros nombrados con `:nombre`
    

## Reto adicional

Implementa un metodo `buscarConFiltros` que acepte un objeto `FiltroLibro` con campos opcionales (autor, precioMin, precioMax, disponible) y construya la consulta JPQL dinamicamente segun los filtros proporcionados.

Consejo

Usa StringBuilder para construir la consulta y una lista de parametros. Solo anade condiciones cuando el filtro tenga valor.