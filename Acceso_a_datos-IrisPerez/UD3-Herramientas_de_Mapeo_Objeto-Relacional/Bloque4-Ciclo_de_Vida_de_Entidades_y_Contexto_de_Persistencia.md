## Objetivos de aprendizaje

Al finalizar este bloque, el alumno sera capaz de:

- Identificar y comprender los cuatro estados del ciclo de vida de una entidad JPA
    
- Utilizar correctamente los metodos del EntityManager para gestionar entidades
    
- Comprender el funcionamiento del contexto de persistencia
    
- Aprovechar las ventajas del cache de primer nivel (first-level cache)
    
- Entender y aplicar el mecanismo de dirty checking
    
- Configurar y utilizar los diferentes modos de flush
    
- Diagnosticar problemas comunes relacionados con el ciclo de vida de entidades

## Criterios de evaluacion relacionados

Este bloque desarrolla principalmente:

- **CE 3d**: Se han aplicado mecanismos de persistencia a los objetos (25%)
    

Contribuye tambien a:

- **CE 3e**: Se han desarrollado aplicaciones que modifican y recuperan objetos persistentes (25%)

## 1. Introduccion al ciclo de vida de entidades

### 1.1. El concepto de ciclo de vida

En JPA, una entidad no es simplemente un objeto Java con anotaciones. Las entidades tienen un ciclo de vida gestionado por el contexto de persistencia que determina como interactuan con la base de datos.

Comprender este ciclo de vida es fundamental para:

- Saber cuando los cambios se sincronizan con la base de datos
    
- Evitar excepciones comunes como LazyInitializationException
    
- Optimizar el rendimiento de las aplicaciones
    
- Implementar correctamente la logica de negocio

### 1.2. Los cuatro estados de una entidad

Una entidad JPA puede encontrarse en uno de los siguientes estados:

|Estado|Descripcion|Sincronizado con BD|
|---|---|---|
|Transient (New)|Objeto recien creado, sin identidad persistente|No|
|Managed (Persistent)|Asociado al contexto de persistencia|Si|
|Detached|Previamente gestionado, ahora desconectado|No|
|Removed|Marcado para eliminacion|Pendiente|

El siguiente diagrama muestra las transiciones entre estados:

                    +-------------+
                    |   TRANSIENT |
                    |    (New)    |
                    +------+------+
                           |
                           | persist()
                           v
+-------------+     +------+------+     +-------------+
|   REMOVED   |<----|   MANAGED   |---->|  DETACHED   |
|             |     | (Persistent)|     |             |
+------+------+     +------+------+     +------+------+
       |                   ^                   |
       |                   |                   |
       +-------------------+-------------------+
           persist()            merge()

## 2. Estado Transient (New)

### 2.1. Caracteristicas del estado Transient

Una entidad se encuentra en estado Transient cuando:

- Ha sido creada con el operador new
    
- No tiene identidad persistente (el ID puede ser null o un valor temporal)
    
- No esta asociada a ningun contexto de persistencia
    
- No existe en la base de datos

// Creacion de una entidad en estado Transient
Libro libro = new Libro();
libro.setTitulo("El Quijote");
libro.setAutor("Miguel de Cervantes");
libro.setIsbn("978-84-376-0494-7");
// En este punto, libro esta en estado TRANSIENT
// No tiene ID asignado y no existe en la base de datos

### 2.2. Comportamiento en estado Transient

Las entidades en estado Transient:

- No son rastreadas por el EntityManager
    
- Los cambios en sus atributos no se sincronizan con la base de datos
    
- Pueden ser recolectadas por el garbage collector si no hay referencias
    
- No participan en relaciones gestionadas

// Ejemplo: modificaciones en estado Transient no afectan a la BD
Libro libro = new Libro();
libro.setTitulo("Titulo inicial");

// Esta modificacion NO se persiste porque el objeto es Transient
libro.setTitulo("Titulo modificado");

// La base de datos no sabe nada de este objeto

### 2.3. Transicion a Managed: persist()

Para que una entidad Transient pase a estado Managed, debemos usar el metodo persist():

EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();

try {
    tx.begin();
    
    Libro libro = new Libro();
    libro.setTitulo("El Quijote");
    libro.setAutor("Miguel de Cervantes");
    // Estado: TRANSIENT
    
    em.persist(libro);
    // Estado: MANAGED
    // El ID se asigna (segun la estrategia de generacion)
    
    System.out.println("ID asignado: " + libro.getId());
    
    tx.commit();
    // Los datos se guardan en la base de datos
    
} catch (Exception e) {
    if (tx.isActive()) tx.rollback();
    throw e;
} finally {
    em.close();
}

## 3. Estado Managed (Persistent)

### 3.1. Caracteristicas del estado Managed

Una entidad se encuentra en estado Managed cuando:

- Esta asociada a un contexto de persistencia activo
    
- Tiene una identidad persistente (ID asignado)
    
- Existe o existira en la base de datos
    
- Sus cambios son rastreados automaticamente
    

### 3.2. Formas de obtener una entidad Managed

Existen varias formas de obtener una entidad en estado Managed:

**Mediante persist():**

Libro libro = new Libro();
libro.setTitulo("Nuevo libro");
em.persist(libro);
// libro ahora esta Managed

**Mediante find():**

Libro libro = em.find(Libro.class, 1L);
// Si existe, libro esta Managed
// Si no existe, libro es null

**Mediante consultas JPQL:**

TypedQuery<Libro> query = em.createQuery(
    "SELECT l FROM Libro l WHERE l.isbn = :isbn", Libro.class);
query.setParameter("isbn", "978-84-376-0494-7");
Libro libro = query.getSingleResult();
// libro esta Managed

**Mediante merge() de una entidad Detached:**

Libro libroDetached = obtenerLibroDesconectado();
Libro libroManaged = em.merge(libroDetached);
// libroManaged esta Managed
// libroDetached sigue Detached

**Mediante getReference():**

Libro libro = em.getReference(Libro.class, 1L);
// libro es un proxy Managed
// La carga real ocurre al acceder a sus atributos

### 3.3. Dirty Checking: sincronizacion automatica

Una de las caracteristicas mas importantes del estado Managed es el dirty checking. El EntityManager detecta automaticamente los cambios en las entidades gestionadas y los sincroniza con la base de datos.

EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();

try {
    tx.begin();
    
    // Obtener entidad Managed
    Libro libro = em.find(Libro.class, 1L);
    
    // Modificar atributos
    // NO es necesario llamar a ningun metodo de actualizacion
    libro.setTitulo("Titulo actualizado");
    libro.setPrecio(new BigDecimal("29.99"));
    
    tx.commit();
    // Los cambios se guardan AUTOMATICAMENTE
    // JPA detecta que el objeto ha cambiado (dirty checking)
    
} catch (Exception e) {
    if (tx.isActive()) tx.rollback();
    throw e;
} finally {
    em.close();
}

### 3.4. El contexto de persistencia

El contexto de persistencia (Persistence Context) es el entorno donde residen las entidades Managed. Funciona como un cache de primer nivel y garantiza:

**Identidad de objeto:**

EntityManager em = emf.createEntityManager();

Libro libro1 = em.find(Libro.class, 1L);
Libro libro2 = em.find(Libro.class, 1L);

// La segunda llamada no va a la base de datos
// Devuelve la misma instancia del cache
System.out.println(libro1 == libro2); // true

**Escritura retrasada (write-behind):**

tx.begin();

Libro libro = em.find(Libro.class, 1L);
libro.setTitulo("Cambio 1");
libro.setTitulo("Cambio 2");
libro.setTitulo("Cambio 3");

tx.commit();
// Solo se ejecuta UN UPDATE con el valor final "Cambio 3"

### 3.5. Metodo refresh()

El metodo refresh() recarga el estado de una entidad desde la base de datos, descartando cualquier cambio local:

tx.begin();

Libro libro = em.find(Libro.class, 1L);
System.out.println("Titulo original: " + libro.getTitulo());

// Modificamos el titulo localmente
libro.setTitulo("Titulo modificado localmente");
System.out.println("Titulo modificado: " + libro.getTitulo());

// Recargamos desde la base de datos
em.refresh(libro);
System.out.println("Titulo tras refresh: " + libro.getTitulo());
// Muestra el titulo original de la BD

tx.commit();

Casos de uso tipicos para refresh():

- Sincronizar con cambios realizados por triggers de base de datos
    
- Descartar modificaciones locales no deseadas
    
- Obtener valores generados por la base de datos
    

## 4. Estado Detached

### 4.1. Caracteristicas del estado Detached

Una entidad se encuentra en estado Detached cuando:

- Tiene una identidad persistente (existe en la BD)
    
- Ya no esta asociada a ningun contexto de persistencia
    
- Fue gestionada previamente
    

### 4.2. Formas de obtener una entidad Detached

**Cierre del EntityManager:**

EntityManager em = emf.createEntityManager();
Libro libro = em.find(Libro.class, 1L);
// libro esta Managed

em.close();
// libro ahora esta Detached

**Mediante detach() explicito:**

Libro libro = em.find(Libro.class, 1L);
// libro esta Managed

em.detach(libro);
// libro ahora esta Detached
// El EntityManager sigue abierto

**Mediante clear():**

Libro libro1 = em.find(Libro.class, 1L);
Libro libro2 = em.find(Libro.class, 2L);
// Ambos estan Managed

em.clear();
// TODAS las entidades pasan a Detached
// El EntityManager sigue abierto pero vacio

**Serializacion:**

// Cuando una entidad se serializa y deserializa
// el resultado es una entidad Detached

### 4.3. Comportamiento en estado Detached

Las entidades Detached:

- Conservan su identidad (ID)
    
- No son rastreadas por ningun EntityManager
    
- Los cambios en sus atributos no se sincronizan
    
- Pueden causar LazyInitializationException al acceder a relaciones no cargadas
    

// Peligro: LazyInitializationException
EntityManager em = emf.createEntityManager();
Libro libro = em.find(Libro.class, 1L);
em.close();
// libro esta Detached

// Si 'prestamos' es una relacion LAZY no inicializada:
libro.getPrestamos().size(); // LazyInitializationException!

### 4.4. Reincorporacion con merge()

Para volver a gestionar una entidad Detached, usamos merge():

// Primera sesion
EntityManager em1 = emf.createEntityManager();
em1.getTransaction().begin();
Libro libroDetached = em1.find(Libro.class, 1L);
em1.getTransaction().commit();
em1.close();
// libroDetached esta Detached

// Modificamos mientras esta Detached
libroDetached.setTitulo("Titulo actualizado offline");

// Segunda sesion: reincorporamos con merge
EntityManager em2 = emf.createEntityManager();
em2.getTransaction().begin();

Libro libroManaged = em2.merge(libroDetached);
// libroManaged es una COPIA Managed
// libroDetached sigue Detached

System.out.println(libroDetached == libroManaged); // false

em2.getTransaction().commit();
em2.close();

Puntos importantes sobre merge():

- Devuelve una nueva instancia Managed
    
- La instancia original permanece Detached
    
- Copia el estado de la entidad Detached a la Managed
    
- Si la entidad no existe en BD, la crea
    

// Comparativa: persist vs merge
Libro nuevo = new Libro();
nuevo.setTitulo("Nuevo libro");

// Con persist: el mismo objeto se vuelve Managed
em.persist(nuevo);
// nuevo esta Managed

// Con merge: se devuelve una copia Managed
Libro nuevoCopia = em.merge(nuevo);
// nuevo sigue Transient, nuevoCopia esta Managed

## 5. Estado Removed

### 5.1. Caracteristicas del estado Removed

Una entidad se encuentra en estado Removed cuando:

- Ha sido marcada para eliminacion
    
- Todavia esta asociada al contexto de persistencia
    
- Sera eliminada de la base de datos al hacer flush/commit
    

### 5.2. Transicion a Removed: remove()

EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();

try {
    tx.begin();
    
    Libro libro = em.find(Libro.class, 1L);
    // libro esta Managed
    
    em.remove(libro);
    // libro esta Removed
    // Todavia existe en la BD hasta el commit
    
    // Podemos seguir leyendo sus atributos
    System.out.println("Eliminando: " + libro.getTitulo());
    
    tx.commit();
    // DELETE se ejecuta
    // libro pasa a Transient (sin ID valido)
    
} catch (Exception e) {
    if (tx.isActive()) tx.rollback();
    throw e;
} finally {
    em.close();
}

### 5.3. Restricciones del estado Removed

Una entidad Removed:

- No puede ser utilizada en relaciones
    
- No debe modificarse (comportamiento indefinido)
    
- Puede ser «rescatada» con persist() antes del commit
    

tx.begin();

Libro libro = em.find(Libro.class, 1L);
em.remove(libro);
// libro esta Removed

// Podemos cancelar la eliminacion
em.persist(libro);
// libro vuelve a estar Managed

tx.commit();
// No se ejecuta DELETE

### 5.4. Eliminacion en cascada

Cuando eliminamos entidades con relaciones, debemos considerar la cascada:

@Entity
public class Biblioteca {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    @OneToMany(mappedBy = "biblioteca", cascade = CascadeType.ALL, 
               orphanRemoval = true)
    private List<Libro> libros = new ArrayList<>();
}

// Al eliminar la biblioteca, los libros tambien se eliminan
tx.begin();
Biblioteca biblioteca = em.find(Biblioteca.class, 1L);
em.remove(biblioteca);
// Todos los libros de esta biblioteca seran eliminados
tx.commit();

## 6. El EntityManager en detalle

### 6.1. Obtencion del EntityManager

En JPA puro:

EntityManagerFactory emf = Persistence
    .createEntityManagerFactory("miUnidadPersistencia");
EntityManager em = emf.createEntityManager();

En Spring:

@Repository
public class LibroDao {
    
    @PersistenceContext
    private EntityManager em;
    
    // El EntityManager es inyectado automaticamente
}

### 6.2. Metodos principales del EntityManager

|Metodo|Descripcion|Estado resultante|
|---|---|---|
|persist(entity)|Hace una entidad Transient Managed|Managed|
|find(class, id)|Busca por clave primaria|Managed o null|
|getReference(class, id)|Obtiene proxy sin cargar|Managed (proxy)|
|merge(entity)|Fusiona entidad Detached|Managed (copia)|
|remove(entity)|Marca para eliminacion|Removed|
|detach(entity)|Desasocia del contexto|Detached|
|refresh(entity)|Recarga desde BD|Managed|
|clear()|Limpia el contexto|Todas Detached|
|flush()|Sincroniza con BD|Sin cambio|
|contains(entity)|Comprueba si esta Managed|Sin cambio|

### 6.3. Metodo contains()

Permite verificar si una entidad esta en estado Managed:

Libro libro = new Libro();
System.out.println(em.contains(libro)); // false (Transient)

em.persist(libro);
System.out.println(em.contains(libro)); // true (Managed)

em.detach(libro);
System.out.println(em.contains(libro)); // false (Detached)

### 6.4. Diferencia entre find() y getReference()

**find():**

- Ejecuta un SELECT inmediatamente
    
- Devuelve la entidad completamente cargada
    
- Devuelve null si no existe
    

**getReference():**

- Devuelve un proxy sin ejecutar SELECT
    
- La carga real ocurre al acceder a los atributos
    
- Lanza EntityNotFoundException si no existe (al acceder)
    

// find() - carga inmediata
Libro libro1 = em.find(Libro.class, 1L);
// SELECT ejecutado aqui

// getReference() - carga diferida
Libro libro2 = em.getReference(Libro.class, 2L);
// Ningun SELECT todavia

System.out.println(libro2.getTitulo());
// SELECT ejecutado aqui (si existe)
// EntityNotFoundException si no existe

Uso tipico de getReference():

// Util para establecer relaciones sin cargar la entidad completa
Prestamo prestamo = new Prestamo();
prestamo.setFecha(LocalDate.now());

// No necesitamos cargar el libro completo
prestamo.setLibro(em.getReference(Libro.class, 1L));

em.persist(prestamo);
// Solo INSERT del prestamo, sin SELECT del libro

## 7. Cache de primer nivel (First-Level Cache)

### 7.1. Funcionamiento del cache

El contexto de persistencia actua como cache de primer nivel:

EntityManager em = emf.createEntityManager();

// Primera llamada: ejecuta SELECT
Libro libro1 = em.find(Libro.class, 1L);
System.out.println("Cargado: " + libro1.getTitulo());

// Segunda llamada: NO ejecuta SELECT (usa cache)
Libro libro2 = em.find(Libro.class, 1L);
System.out.println("Del cache: " + libro2.getTitulo());

// Tercera llamada: tampoco ejecuta SELECT
Libro libro3 = em.find(Libro.class, 1L);

// Todas son la misma instancia
System.out.println(libro1 == libro2); // true
System.out.println(libro2 == libro3); // true

### 7.2. Alcance del cache

El cache de primer nivel:

- Es local a cada EntityManager
    
- Vive mientras el EntityManager este abierto
    
- Se limpia con clear() o al cerrar el EntityManager
    
- No es compartido entre EntityManagers
    

EntityManager em1 = emf.createEntityManager();
EntityManager em2 = emf.createEntityManager();

Libro libro1 = em1.find(Libro.class, 1L);
Libro libro2 = em2.find(Libro.class, 1L);

// Cada EntityManager tiene su propia copia
System.out.println(libro1 == libro2); // false
// Se ejecutaron DOS SELECT (uno por cada EM)

em1.close();
em2.close();

### 7.3. Invalidacion del cache

En algunos casos es necesario invalidar el cache:

tx.begin();

Libro libro = em.find(Libro.class, 1L);
// libro en cache

// Actualizacion masiva (bypasses cache)
em.createQuery("UPDATE Libro l SET l.precio = l.precio * 1.1")
  .executeUpdate();

// libro en cache tiene precio antiguo!
System.out.println(libro.getPrecio()); // Valor desactualizado

// Solucion: refresh o clear
em.refresh(libro);
// o
em.clear(); // Limpia todo el cache

tx.commit();

## 8. Modos de Flush

### 8.1. Que es el Flush

El flush es el proceso de sincronizar el estado del contexto de persistencia con la base de datos. Durante el flush:

- Se ejecutan los INSERT para entidades nuevas
    
- Se ejecutan los UPDATE para entidades modificadas
    
- Se ejecutan los DELETE para entidades eliminadas
    

### 8.2. FlushModeType

JPA define dos modos de flush:

**AUTO (por defecto):**

- Flush automatico antes de cada consulta
    
- Flush automatico al hacer commit
    
- Garantiza consistencia de lecturas
    

**COMMIT:**

- Flush solo al hacer commit
    
- Las consultas pueden no ver cambios pendientes
    
- Puede mejorar rendimiento en algunos casos
    

// Configurar modo de flush
em.setFlushMode(FlushModeType.COMMIT);

tx.begin();

Libro libro = new Libro();
libro.setTitulo("Nuevo libro");
em.persist(libro);

// Con AUTO: se haria flush antes de la consulta
// Con COMMIT: NO se hace flush, la consulta no ve el libro nuevo
List<Libro> libros = em.createQuery("SELECT l FROM Libro l", Libro.class)
    .getResultList();

tx.commit(); // Flush ocurre aqui con COMMIT

### 8.3. Flush manual

Podemos forzar un flush explicitamente:

tx.begin();

for (int i = 0; i < 10000; i++) {
    Libro libro = new Libro();
    libro.setTitulo("Libro " + i);
    em.persist(libro);
    
    // Flush y clear periodicos para evitar OutOfMemoryError
    if (i % 50 == 0) {
        em.flush();  // Sincroniza con BD
        em.clear();  // Libera memoria
    }
}

tx.commit();

### 8.4. Orden de ejecucion en Flush

Durante el flush, las operaciones se ejecutan en un orden especifico:

1. INSERT de entidades nuevas
    
2. UPDATE de entidades modificadas
    
3. DELETE de colecciones huerfanas
    
4. DELETE de entidades eliminadas
    

Este orden garantiza la integridad referencial.

## 9. Problemas comunes y soluciones

### 9.1. LazyInitializationException

**Problema:**

EntityManager em = emf.createEntityManager();
Autor autor = em.find(Autor.class, 1L);
em.close();

// Error! El EntityManager esta cerrado
for (Libro libro : autor.getLibros()) {
    System.out.println(libro.getTitulo());
}

**Soluciones:**

1. Inicializar antes de cerrar:
    

EntityManager em = emf.createEntityManager();
Autor autor = em.find(Autor.class, 1L);
autor.getLibros().size(); // Fuerza la carga
em.close();

// Ahora funciona
for (Libro libro : autor.getLibros()) {
    System.out.println(libro.getTitulo());
}

1. Usar JOIN FETCH:
    

Autor autor = em.createQuery(
    "SELECT a FROM Autor a JOIN FETCH a.libros WHERE a.id = :id",
    Autor.class)
    .setParameter("id", 1L)
    .getSingleResult();
em.close();

// Los libros ya estan cargados
for (Libro libro : autor.getLibros()) {
    System.out.println(libro.getTitulo());
}

1. Usar `@EntityGraph`:
    

@NamedEntityGraph(
    name = "autor-con-libros",
    attributeNodes = @NamedAttributeNode("libros")
)
@Entity
public class Autor { ... }

// Uso
EntityGraph<?> graph = em.getEntityGraph("autor-con-libros");
Autor autor = em.find(Autor.class, 1L, 
    Map.of("javax.persistence.fetchgraph", graph));

### 9.2. Entidad Detached pasada a persist()

**Problema:**

// libro es Detached (tiene ID)
em.persist(libro);
// DetachedEntityException!

**Solucion:**

// Usar merge en lugar de persist
Libro libroManaged = em.merge(libro);

### 9.3. Modificaciones no guardadas

**Problema:**

tx.begin();
Libro libro = new Libro();
libro.setTitulo("Titulo");
// Olvidamos persist()
tx.commit();
// No se guarda nada!

**Solucion:**

tx.begin();
Libro libro = new Libro();
libro.setTitulo("Titulo");
em.persist(libro); // No olvidar!
tx.commit();

### 9.4. Entidad eliminada referenciada

**Problema:**

em.remove(libro);
prestamo.setLibro(libro); // Error logico
em.persist(prestamo);
// Comportamiento indefinido o excepcion

**Solucion:**

// Eliminar la referencia antes de remove
libro.getPrestamos().clear();
for (Prestamo p : libro.getPrestamos()) {
    p.setLibro(null);
}
em.remove(libro);

## 10. Ejercicio guiado: Sistema de Biblioteca

En este ejercicio implementaremos un sistema de gestion de biblioteca que demuestra todos los estados del ciclo de vida.

### 10.1. Modelo de datos

@Entity
@Table(name = "bibliotecas")
public class Biblioteca {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String nombre;
    
    @Column(length = 200)
    private String direccion;
    
    @OneToMany(mappedBy = "biblioteca", cascade = CascadeType.ALL,
               orphanRemoval = true, fetch = FetchType.LAZY)
    private List<Libro> libros = new ArrayList<>();
    
    // Metodos de conveniencia
    public void addLibro(Libro libro) {
        libros.add(libro);
        libro.setBiblioteca(this);
    }
    
    public void removeLibro(Libro libro) {
        libros.remove(libro);
        libro.setBiblioteca(null);
    }
    
    // Constructores, getters y setters
}

@Entity
@Table(name = "libros")
public class Libro {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 200)
    private String titulo;
    
    @Column(length = 100)
    private String autor;
    
    @Column(unique = true, length = 20)
    private String isbn;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal precio;
    
    @Column(name = "fecha_publicacion")
    private LocalDate fechaPublicacion;
    
    @Enumerated(EnumType.STRING)
    private EstadoLibro estado = EstadoLibro.DISPONIBLE;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "biblioteca_id")
    private Biblioteca biblioteca;
    
    @OneToMany(mappedBy = "libro", cascade = CascadeType.ALL,
               orphanRemoval = true)
    private List<Prestamo> prestamos = new ArrayList<>();
    
    // Constructores, getters y setters
}

public enum EstadoLibro {
    DISPONIBLE, PRESTADO, EN_REPARACION, PERDIDO
}

@Entity
@Table(name = "socios")
public class Socio {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String nombre;
    
    @Column(unique = true, length = 50)
    private String email;
    
    @Column(name = "fecha_alta")
    private LocalDate fechaAlta = LocalDate.now();
    
    @Column(name = "activo")
    private boolean activo = true;
    
    @OneToMany(mappedBy = "socio", cascade = CascadeType.ALL)
    private List<Prestamo> prestamos = new ArrayList<>();
    
    // Constructores, getters y setters
}

@Entity
@Table(name = "prestamos")
public class Prestamo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "libro_id", nullable = false)
    private Libro libro;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "socio_id", nullable = false)
    private Socio socio;
    
    @Column(name = "fecha_prestamo", nullable = false)
    private LocalDate fechaPrestamo;
    
    @Column(name = "fecha_devolucion_prevista")
    private LocalDate fechaDevolucionPrevista;
    
    @Column(name = "fecha_devolucion_real")
    private LocalDate fechaDevolucionReal;
    
    @Enumerated(EnumType.STRING)
    private EstadoPrestamo estado = EstadoPrestamo.ACTIVO;
    
    // Constructores, getters y setters
}

public enum EstadoPrestamo {
    ACTIVO, DEVUELTO, VENCIDO
}

### 10.2. Demostracion de estados

📁 public class DemoEstados {/
   📁 private EntityManagerFactory emf;/
   📁 public DemoEstados() {/
      📄 emf = Persistence.createEntityManagerFactory("bibliotecaPU");
   📁 }/
   📁 /**/
   📁 * Demuestra el estado Transient y la transicion a Managed/
   📁 */
   📁 public void demostrarTransientAManaged() {/
      📄 System.out.println("=== TRANSIENT A MANAGED ===");
      📄 EntityManager em = emf.createEntityManager();
      📄 EntityTransaction tx = em.getTransaction();
      📁 try {/
         📄 tx.begin();
         📄 // 1. Crear entidad (TRANSIENT)
         📁 Libro libro = new Libro();/
         📄 libro.setTitulo("Don Quijote de la Mancha");
         📄 libro.setAutor("Miguel de Cervantes");
         📄 libro.setIsbn("978-84-376-0494-7");
         📄 System.out.println("Estado inicial:");
         📄 System.out.println("  - ID: " + libro.getId());
         📄 System.out.println("  - En contexto: " + em.contains(libro));
         📄 // 2. Persistir (TRANSIENT -> MANAGED)
         📄 em.persist(libro);
         📄 System.out.println("\nTras persist():");
         📄 System.out.println("  - ID: " + libro.getId());
         📄 System.out.println("  - En contexto: " + em.contains(libro));
         📄 // 3. Commit
         📄 tx.commit();
         📄 System.out.println("\nTras commit:");
         📄 System.out.println("  - ID: " + libro.getId());
         📄 System.out.println("  - En contexto: " + em.contains(libro));
      📁 } catch (Exception e) {/
         📄 if (tx.isActive()) tx.rollback();
         📄 e.printStackTrace();
      📁 } finally {/
         📄 em.close();
      📁 }/
   📁 }/
   📁 /**/
   📁 * Demuestra el dirty checking/
   📁 */
   📁 public void demostrarDirtyChecking() {/
      📄 System.out.println("\n=== DIRTY CHECKING ===");
      📄 EntityManager em = emf.createEntityManager();
      📄 EntityTransaction tx = em.getTransaction();
      📁 try {/
         📄 tx.begin();
         📄 // 1. Obtener entidad Managed
         📄 Libro libro = em.find(Libro.class, 1L);
         📄 System.out.println("Titulo original: " + libro.getTitulo());
         📄 // 2. Modificar sin llamar a update (no existe!)
         📄 libro.setTitulo("El ingenioso hidalgo Don Quijote");
         📄 libro.setPrecio(new BigDecimal("24.99"));
         📄 System.out.println("Titulo modificado: " + libro.getTitulo());
         📄 System.out.println("(No hemos llamado a ningun metodo update)");
         📄 // 3. El commit detecta los cambios automaticamente
         📄 tx.commit();
         📄 System.out.println("Cambios guardados automaticamente!");
      📁 } catch (Exception e) {/
         📄 if (tx.isActive()) tx.rollback();
         📄 e.printStackTrace();
      📁 } finally {/
         📄 em.close();
      📁 }/
   📁 }/
   📁 /**/
   📁 * Demuestra el cache de primer nivel/
   📁 */
   📁 public void demostrarCachePrimerNivel() {/
      📄 System.out.println("\n=== CACHE DE PRIMER NIVEL ===");
      📄 EntityManager em = emf.createEntityManager();
      📁 try {/
         📄 System.out.println("Primera busqueda (ejecuta SELECT):");
         📄 Libro libro1 = em.find(Libro.class, 1L);
         📄 System.out.println("  - Libro: " + libro1.getTitulo());
         📄 System.out.println("\nSegunda busqueda (usa cache):");
         📄 Libro libro2 = em.find(Libro.class, 1L);
         📄 System.out.println("  - Libro: " + libro2.getTitulo());
         📄 System.out.println("\nVerificacion de identidad:");
         📄 System.out.println("  - libro1 == libro2: " + (libro1 == libro2));
      📁 } finally {/
         📄 em.close();
      📁 }/
   📁 }/
   📁 /**/
   📁 * Demuestra la transicion a Detached/
   📁 */
   📁 public void demostrarDetached() {/
      📄 System.out.println("\n=== ESTADO DETACHED ===");
      📁 Libro libroDetached;/
      📁 // Primera sesion/
      📄 EntityManager em1 = emf.createEntityManager();
      📁 try {/
         📄 libroDetached = em1.find(Libro.class, 1L);
         📄 System.out.println("En primera sesion:");
         📄 System.out.println("  - En contexto: " + em1.contains(libroDetached));
      📁 } finally {/
         📄 em1.close();
      📁 }/
      📁 // Tras cerrar/
      📄 System.out.println("\nTras cerrar EntityManager:");
      📄 System.out.println("  - Libro existe: " + (libroDetached != null));
      📄 System.out.println("  - Podemos leer: " + libroDetached.getTitulo());
      📁 // Modificar mientras Detached/
      📄 libroDetached.setTitulo("Titulo modificado offline");
      📄 System.out.println("  - Modificado offline: " + libroDetached.getTitulo());
      📁 // Segunda sesion: reincorporar con merge/
      📄 EntityManager em2 = emf.createEntityManager();
      📄 EntityTransaction tx = em2.getTransaction();
      📁 try {/
         📄 tx.begin();
         📄 Libro libroManaged = em2.merge(libroDetached);
         📄 System.out.println("\nTras merge():");
         📄 System.out.println("  - libroDetached en contexto: " +
            📄 em2.contains(libroDetached));
         📄 System.out.println("  - libroManaged en contexto: " +
            📄 em2.contains(libroManaged));
         📄 System.out.println("  - Son el mismo objeto: " +
            📁 (libroDetached == libroManaged));/
         📄 tx.commit();
         📄 System.out.println("Cambios guardados!");
      📁 } catch (Exception e) {/
         📄 if (tx.isActive()) tx.rollback();
         📄 e.printStackTrace();
      📁 } finally {/
         📄 em2.close();
      📁 }/
   📁 }/
   📁 /**/
   📁 * Demuestra la eliminacion (estado Removed)/
   📁 */
   📁 public void demostrarRemoved() {/
      📄 System.out.println("\n=== ESTADO REMOVED ===");
      📄 EntityManager em = emf.createEntityManager();
      📄 EntityTransaction tx = em.getTransaction();
      📁 try {/
         📁 // Primero creamos un libro para eliminar/
         📄 tx.begin();
         📁 Libro libro = new Libro();/
         📄 libro.setTitulo("Libro para eliminar");
         📄 libro.setAutor("Autor temporal");
         📄 em.persist(libro);
         📄 tx.commit();
         📄 Long id = libro.getId();
         📄 System.out.println("Libro creado con ID: " + id);
         📁 // Ahora lo eliminamos/
         📄 tx.begin();
         📄 Libro libroAEliminar = em.find(Libro.class, id);
         📄 System.out.println("\nAntes de remove:");
         📄 System.out.println("  - En contexto: " + em.contains(libroAEliminar));
         📄 em.remove(libroAEliminar);
         📄 System.out.println("\nTras remove (antes de commit):");
         📄 System.out.println("  - En contexto: " + em.contains(libroAEliminar));
         📄 System.out.println("  - Podemos leer titulo: " + libroAEliminar.getTitulo());
         📄 tx.commit();
         📄 System.out.println("\nTras commit:");
         📄 Libro verificacion = em.find(Libro.class, id);
         📄 System.out.println("  - Busqueda por ID: " + verificacion);
      📁 } catch (Exception e) {/
         📄 if (tx.isActive()) tx.rollback();
         📄 e.printStackTrace();
      📁 } finally {/
         📄 em.close();
      📁 }/
   📁 }/
   📁 /**/
   📁 * Demuestra el metodo refresh/
   📁 */
   📁 public void demostrarRefresh() {/
      📄 System.out.println("\n=== METODO REFRESH ===");
      📄 EntityManager em = emf.createEntityManager();
      📄 EntityTransaction tx = em.getTransaction();
      📁 try {/
         📄 tx.begin();
         📄 Libro libro = em.find(Libro.class, 1L);
         📄 String tituloOriginal = libro.getTitulo();
         📄 System.out.println("Titulo original: " + tituloOriginal);
         📁 // Modificamos localmente/
         📄 libro.setTitulo("Titulo temporal que no queremos guardar");
         📄 System.out.println("Titulo modificado: " + libro.getTitulo());
         📁 // Refresh recarga de la BD/
         📄 em.refresh(libro);
         📄 System.out.println("Titulo tras refresh: " + libro.getTitulo());
         📄 tx.commit();
      📁 } catch (Exception e) {/
         📄 if (tx.isActive()) tx.rollback();
         📄 e.printStackTrace();
      📁 } finally {/
         📄 em.close();
      📁 }/
   📁 }/
   📁 /**/
   📁 * Demuestra los modos de flush/
   📁 */
   📁 public void demostrarFlushModes() {/
      📄 System.out.println("\n=== MODOS DE FLUSH ===");
      📄 EntityManager em = emf.createEntityManager();
      📄 EntityTransaction tx = em.getTransaction();
      📁 try {/
         📄 tx.begin();
         📁 // Modo AUTO (por defecto)/
         📄 em.setFlushMode(FlushModeType.AUTO);
         📄 System.out.println("FlushMode: " + em.getFlushMode());
         📁 Libro libro = new Libro();/
         📄 libro.setTitulo("Libro para flush test");
         📄 libro.setIsbn("TEST-" + System.currentTimeMillis());
         📄 em.persist(libro);
         📄 System.out.println("Libro persistido (no flush todavia)");
         📁 // Con AUTO, la siguiente consulta fuerza flush/
         📄 Long count = em.createQuery("SELECT COUNT(l) FROM Libro l", Long.class)
            📄 .getSingleResult();
         📄 System.out.println("Consulta ejecutada, count: " + count);
         📄 System.out.println("(El INSERT ya se ejecuto por el flush automatico)");
         📄 tx.commit();
      📁 } catch (Exception e) {/
         📄 if (tx.isActive()) tx.rollback();
         📄 e.printStackTrace();
      📁 } finally {/
         📄 em.close();
      📁 }/
   📁 }/
   📁 public void cerrar() {/
      📄 if (emf != null && emf.isOpen()) {
         📄 emf.close();
      📁 }/
   📁 }/
   📁 public static void main(String[] args) {/
      📁 DemoEstados demo = new DemoEstados();/
      📁 try {/
         📄 demo.demostrarTransientAManaged();
         📄 demo.demostrarDirtyChecking();
         📄 demo.demostrarCachePrimerNivel();
         📄 demo.demostrarDetached();
         📄 demo.demostrarRemoved();
         📄 demo.demostrarRefresh();
         📄 demo.demostrarFlushModes();
      📁 } finally {/
         📄 demo.cerrar();
      📁 }/
   📁 }/
📁 }/

## 11. Ejercicios propuestos

### Ejercicio 1: Gestion de estados basica (2 puntos)

Implementa una clase `GestorLibros` con los siguientes metodos:

1. `crearLibro(String titulo, String autor)`: Crea un libro y lo persiste
    
2. `actualizarTitulo(Long id, String nuevoTitulo)`: Actualiza usando dirty checking
    
3. `eliminarLibro(Long id)`: Elimina un libro verificando que no tenga prestamos
    
4. `verificarEstado(Libro libro)`: Imprime el estado actual de la entidad
    

### Ejercicio 2: Operaciones Detached (2 puntos)

Implementa un servicio de edicion offline:

1. Obtener un libro y desconectarlo del EntityManager
    
2. Permitir multiples modificaciones mientras esta Detached
    
3. Reincorporar los cambios con merge
    
4. Manejar el caso de que el libro haya sido eliminado mientras estaba Detached
    

### Ejercicio 3: Cache y rendimiento (2 puntos)

1. Implementa un metodo que cargue 100 libros por ID individualmente
    
2. Mide el tiempo de ejecucion
    
3. Implementa una version optimizada usando una sola consulta JPQL
    
4. Compara los tiempos y explica la diferencia
    

### Ejercicio 4: Flush y procesamiento batch (2 puntos)

Implementa un importador de libros que:

1. Lea datos de un fichero CSV
    
2. Cree entidades Libro
    
3. Use flush y clear periodicos para procesar grandes volumenes
    
4. Maneje errores sin perder los datos ya procesados
    

### Ejercicio 5: Diagnostico de problemas (2 puntos)

El siguiente codigo tiene varios problemas. Identificalos y corrigelos:

public void procesoProblematico() {
    EntityManager em = emf.createEntityManager();
    
    Libro libro = new Libro();
    libro.setTitulo("Test");
    
    em.getTransaction().begin();
    em.persist(libro);
    em.getTransaction().commit();
    
    em.close();
    
    // Operaciones posteriores
    libro.setAutor("Nuevo autor");
    
    EntityManager em2 = emf.createEntityManager();
    em2.getTransaction().begin();
    em2.persist(libro);
    em2.getTransaction().commit();
    em2.close();
    
    // Cargar con relaciones lazy
    EntityManager em3 = emf.createEntityManager();
    Biblioteca biblioteca = em3.find(Biblioteca.class, 1L);
    em3.close();
    
    for (Libro l : biblioteca.getLibros()) {
        System.out.println(l.getTitulo());
    }
}

## 12. Resumen

### Tabla de transiciones de estado

|Estado origen|Metodo|Estado destino|
|---|---|---|
|Transient|persist()|Managed|
|Transient|merge()|Managed (copia)|
|Managed|detach()|Detached|
|Managed|remove()|Removed|
|Managed|clear()|Detached|
|Managed|close()|Detached|
|Detached|merge()|Managed (copia)|
|Detached|persist()|Excepcion|
|Removed|persist()|Managed|
|Removed|commit|Transient|

### Puntos clave

1. **Transient**: Entidades nuevas no gestionadas
    
2. **Managed**: Entidades rastreadas con dirty checking automatico
    
3. **Detached**: Entidades desconectadas que conservan su identidad
    
4. **Removed**: Entidades marcadas para eliminacion
    

5. El **contexto de persistencia** actua como cache de primer nivel
    
6. **Dirty checking** detecta cambios automaticamente
    
7. **Flush** sincroniza el contexto con la base de datos
    
8. **merge()** devuelve una copia, no modifica el original
    

## 13. Referencias

- JPA 3.0 Specification (Jakarta Persistence)
    
- Hibernate ORM Documentation
    
- Java Persistence with Hibernate, Second Edition
    
- Pro JPA 2 in Java EE 8
    

**Material didactico para el modulo 0486 - Acceso a Datos** **Ciclo Formativo**: Desarrollo de Aplicaciones Multiplataforma (DAM) **Bloque 4**: Ciclo de Vida de Entidades y Contexto de Persistencia