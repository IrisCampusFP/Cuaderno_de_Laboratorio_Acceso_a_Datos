# Actividad Practica: Mapeo Avanzado con JPA

## Objetivos

- Implementar relaciones OneToMany y ManyToOne
    
- Configurar relaciones ManyToMany
    
- Aplicar cascadas y fetch types
    
- Manejar relaciones bidireccionales

## Contexto: Sistema de Gestion de Biblioteca

Vamos a modelar un sistema de biblioteca con las siguientes entidades:

- **Autor**: puede escribir muchos libros
    
- **Libro**: pertenece a un autor y puede tener muchas categorias
    
- **Categoria**: puede contener muchos libros
    
- **Editorial**: publica muchos libros

## Ejercicio 1: Relacion OneToMany / ManyToOne

### Enunciado

Crea las entidades `Autor` y `Libro` con una relacion bidireccional:

- Un autor puede tener muchos libros
    
- Un libro pertenece a un solo autor

**Requisitos:**

- La relacion debe ser bidireccional
    
- Al eliminar un autor, sus libros deben eliminarse (cascade)
    
- Los libros deben cargarse de forma LAZY
    
- Implementa metodos helper para mantener la consistencia

Ver solucion

**Entidad Autor:**

@Entity
@Table(name = "autores")
public class Autor {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String nombre;
    
    @Column(length = 50)
    private String nacionalidad;
    
    @Column(name = "fecha_nacimiento")
    private LocalDate fechaNacimiento;
    
    @OneToMany(mappedBy = "autor", 
               cascade = CascadeType.ALL, 
               orphanRemoval = true,
               fetch = FetchType.LAZY)
    private List<Libro> libros = new ArrayList<>();
    
    // Constructor vacio
    public Autor() {}
    
    public Autor(String nombre, String nacionalidad) {
        this.nombre = nombre;
        this.nacionalidad = nacionalidad;
    }
    
    // Metodos helper para mantener consistencia bidireccional
    public void addLibro(Libro libro) {
        libros.add(libro);
        libro.setAutor(this);
    }
    
    public void removeLibro(Libro libro) {
        libros.remove(libro);
        libro.setAutor(null);
    }
    
    // Getters y setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    
    public String getNacionalidad() { return nacionalidad; }
    public void setNacionalidad(String nacionalidad) { this.nacionalidad = nacionalidad; }
    
    public List<Libro> getLibros() { return libros; }
    public void setLibros(List<Libro> libros) { this.libros = libros; }
}

**Entidad Libro:**

@Entity
@Table(name = "libros")
public class Libro {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String titulo;
    
    @Column(unique = true, length = 13)
    private String isbn;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "autor_id", nullable = false)
    private Autor autor;
    
    // Constructor vacio
    public Libro() {}
    
    public Libro(String titulo, String isbn) {
        this.titulo = titulo;
        this.isbn = isbn;
    }
    
    // Getters y setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getTitulo() { return titulo; }
    public void setTitulo(String titulo) { this.titulo = titulo; }
    
    public String getIsbn() { return isbn; }
    public void setIsbn(String isbn) { this.isbn = isbn; }
    
    public Autor getAutor() { return autor; }
    public void setAutor(Autor autor) { this.autor = autor; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Libro)) return false;
        Libro libro = (Libro) o;
        return isbn != null && isbn.equals(libro.getIsbn());
    }
    
    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}

**Conceptos clave:**

- `mappedBy` indica el lado inverso de la relacion
    
- `orphanRemoval = true` elimina libros huerfanos
    
- Los metodos helper (`addLibro`, `removeLibro`) mantienen ambos lados sincronizados
    
- `equals/hashCode` basados en el identificador natural (isbn)
    

## Ejercicio 2: Relacion ManyToMany

### Enunciado

Anade una relacion ManyToMany entre `Libro` y `Categoria`:

- Un libro puede pertenecer a varias categorias
    
- Una categoria puede contener varios libros
    
- La tabla intermedia debe llamarse `libro_categoria`

Crea la entidad `Categoria` y modifica `Libro` para incluir la relacion.

Ver solucion

**Entidad Categoria:**

@Entity
@Table(name = "categorias")
public class Categoria {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String nombre;
    
    private String descripcion;
    
    @ManyToMany(mappedBy = "categorias")
    private Set<Libro> libros = new HashSet<>();
    
    public Categoria() {}
    
    public Categoria(String nombre) {
        this.nombre = nombre;
    }
    
    // Getters y setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    
    public String getDescripcion() { return descripcion; }
    public void setDescripcion(String descripcion) { this.descripcion = descripcion; }
    
    public Set<Libro> getLibros() { return libros; }
    public void setLibros(Set<Libro> libros) { this.libros = libros; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Categoria)) return false;
        Categoria that = (Categoria) o;
        return nombre != null && nombre.equals(that.getNombre());
    }
    
    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}

**Modificacion en Libro:**

@Entity
@Table(name = "libros")
public class Libro {
    // ... campos anteriores ...
    
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "libro_categoria",
        joinColumns = @JoinColumn(name = "libro_id"),
        inverseJoinColumns = @JoinColumn(name = "categoria_id")
    )
    private Set<Categoria> categorias = new HashSet<>();
    
    // Metodos helper
    public void addCategoria(Categoria categoria) {
        this.categorias.add(categoria);
        categoria.getLibros().add(this);
    }
    
    public void removeCategoria(Categoria categoria) {
        this.categorias.remove(categoria);
        categoria.getLibros().remove(this);
    }
    
    // Getter y setter
    public Set<Categoria> getCategorias() { return categorias; }
    public void setCategorias(Set<Categoria> categorias) { this.categorias = categorias; }
}

**Notas importantes:**

- Usamos `Set` en lugar de `List` para evitar duplicados
    
- @JoinTable define la tabla intermedia
    
- Solo `PERSIST` y `MERGE` en cascade (no `REMOVE` para evitar eliminar categorias usadas)
    
- El lado propietario (Libro) define @JoinTable
    
- El lado inverso usa `mappedBy`
    

## Ejercicio 3: Relacion OneToOne

### Enunciado

Cada libro puede tener una unica `FichaTecnica` con informacion adicional:

- numero de paginas
    
- idioma
    
- formato (TAPA_DURA, TAPA_BLANDA, EBOOK)
    
- peso en gramos

Implementa la relacion OneToOne entre `Libro` y `FichaTecnica`.

Ver solucion

**Enum Formato:**

public enum Formato {
    TAPA_DURA,
    TAPA_BLANDA,
    EBOOK
}

**Entidad FichaTecnica:**

@Entity
@Table(name = "fichas_tecnicas")
public class FichaTecnica {
    
    @Id
    private Long id;  // Mismo ID que el libro
    
    @Column(name = "num_paginas")
    private Integer numeroPaginas;
    
    @Column(length = 50)
    private String idioma;
    
    @Enumerated(EnumType.STRING)
    @Column(length = 20)
    private Formato formato;
    
    @Column(name = "peso_gramos")
    private Integer pesoGramos;
    
    @OneToOne(fetch = FetchType.LAZY)
    @MapsId  // Usa el ID del libro
    @JoinColumn(name = "id")
    private Libro libro;
    
    public FichaTecnica() {}
    
    public FichaTecnica(Libro libro) {
        this.libro = libro;
    }
    
    // Getters y setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public Integer getNumeroPaginas() { return numeroPaginas; }
    public void setNumeroPaginas(Integer numeroPaginas) { this.numeroPaginas = numeroPaginas; }
    
    public String getIdioma() { return idioma; }
    public void setIdioma(String idioma) { this.idioma = idioma; }
    
    public Formato getFormato() { return formato; }
    public void setFormato(Formato formato) { this.formato = formato; }
    
    public Integer getPesoGramos() { return pesoGramos; }
    public void setPesoGramos(Integer pesoGramos) { this.pesoGramos = pesoGramos; }
    
    public Libro getLibro() { return libro; }
    public void setLibro(Libro libro) { this.libro = libro; }
}

**Modificacion en Libro:**

@Entity
@Table(name = "libros")
public class Libro {
    // ... campos anteriores ...
    
    @OneToOne(mappedBy = "libro", 
              cascade = CascadeType.ALL, 
              orphanRemoval = true,
              fetch = FetchType.LAZY)
    private FichaTecnica fichaTecnica;
    
    // Metodo helper
    public void setFichaTecnica(FichaTecnica ficha) {
        if (ficha == null) {
            if (this.fichaTecnica != null) {
                this.fichaTecnica.setLibro(null);
            }
        } else {
            ficha.setLibro(this);
        }
        this.fichaTecnica = ficha;
    }
    
    public FichaTecnica getFichaTecnica() { return fichaTecnica; }
}

**Uso de @MapsId:**

- La FichaTecnica comparte el mismo ID que el Libro
    
- Evita crear una columna FK adicional
    
- La relacion es identificadora
    

## Ejercicio 4: Embedded y ElementCollection

### Enunciado

Anade a la entidad `Autor`:

1. Un objeto embebido `Direccion` con: calle, ciudad, codigoPostal, pais
    
2. Una coleccion de elementos `Set<String>` para almacenar los premios recibidos

La tabla de premios debe llamarse `autor_premios`.

Ver solucion

**Clase Embebida Direccion:**

@Embeddable
public class Direccion {
    
    private String calle;
    
    private String ciudad;
    
    @Column(name = "codigo_postal", length = 10)
    private String codigoPostal;
    
    @Column(length = 50)
    private String pais;
    
    public Direccion() {}
    
    public Direccion(String calle, String ciudad, String codigoPostal, String pais) {
        this.calle = calle;
        this.ciudad = ciudad;
        this.codigoPostal = codigoPostal;
        this.pais = pais;
    }
    
    // Getters y setters
    public String getCalle() { return calle; }
    public void setCalle(String calle) { this.calle = calle; }
    
    public String getCiudad() { return ciudad; }
    public void setCiudad(String ciudad) { this.ciudad = ciudad; }
    
    public String getCodigoPostal() { return codigoPostal; }
    public void setCodigoPostal(String codigoPostal) { this.codigoPostal = codigoPostal; }
    
    public String getPais() { return pais; }
    public void setPais(String pais) { this.pais = pais; }
    
    @Override
    public String toString() {
        return calle + ", " + codigoPostal + " " + ciudad + ", " + pais;
    }
}

**Modificacion en Autor:**

@Entity
@Table(name = "autores")
public class Autor {
    // ... campos anteriores ...
    
    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "calle", column = @Column(name = "dir_calle")),
        @AttributeOverride(name = "ciudad", column = @Column(name = "dir_ciudad")),
        @AttributeOverride(name = "codigoPostal", column = @Column(name = "dir_cp")),
        @AttributeOverride(name = "pais", column = @Column(name = "dir_pais"))
    })
    private Direccion direccion;
    
    @ElementCollection
    @CollectionTable(
        name = "autor_premios",
        joinColumns = @JoinColumn(name = "autor_id")
    )
    @Column(name = "premio")
    private Set<String> premios = new HashSet<>();
    
    // Metodos helper para premios
    public void addPremio(String premio) {
        this.premios.add(premio);
    }
    
    public void removePremio(String premio) {
        this.premios.remove(premio);
    }
    
    // Getters y setters
    public Direccion getDireccion() { return direccion; }
    public void setDireccion(Direccion direccion) { this.direccion = direccion; }
    
    public Set<String> getPremios() { return premios; }
    public void setPremios(Set<String> premios) { this.premios = premios; }
}

**Tablas generadas:**

-- Tabla autores incluye columnas de direccion
CREATE TABLE autores (
    id BIGINT PRIMARY KEY,
    nombre VARCHAR(100),
    nacionalidad VARCHAR(50),
    dir_calle VARCHAR(255),
    dir_ciudad VARCHAR(255),
    dir_cp VARCHAR(10),
    dir_pais VARCHAR(50)
);

-- Tabla separada para premios
CREATE TABLE autor_premios (
    autor_id BIGINT,
    premio VARCHAR(255),
    FOREIGN KEY (autor_id) REFERENCES autores(id)
);

## Ejercicio 5: Programa de prueba completo

### Enunciado

Crea un programa que:

1. Cree un autor con direccion y premios
    
2. Cree varias categorias
    
3. Cree libros asignados al autor con categorias
    
4. Anade ficha tecnica a un libro
    
5. Realice consultas para verificar las relaciones
    

Ver solucion

public class MainRelaciones {
    
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("biblioteca-pu");
        EntityManager em = emf.createEntityManager();
        
        try {
            em.getTransaction().begin();
            
            // 1. Crear autor con direccion y premios
            Autor autor = new Autor("Gabriel Garcia Marquez", "Colombiana");
            autor.setFechaNacimiento(LocalDate.of(1927, 3, 6));
            autor.setDireccion(new Direccion(
                "Calle del Libro 123", 
                "Bogota", 
                "110111", 
                "Colombia"
            ));
            autor.addPremio("Premio Nobel de Literatura 1982");
            autor.addPremio("Premio Romulo Gallegos 1972");
            
            em.persist(autor);
            
            // 2. Crear categorias
            Categoria ficcion = new Categoria("Ficcion");
            ficcion.setDescripcion("Obras de ficcion literaria");
            
            Categoria realismo = new Categoria("Realismo Magico");
            realismo.setDescripcion("Genero literario latinoamericano");
            
            Categoria novela = new Categoria("Novela");
            
            em.persist(ficcion);
            em.persist(realismo);
            em.persist(novela);
            
            // 3. Crear libros con categorias
            Libro libro1 = new Libro("Cien anos de soledad", "978-0307474728");
            autor.addLibro(libro1);  // Usando metodo helper
            libro1.addCategoria(ficcion);
            libro1.addCategoria(realismo);
            libro1.addCategoria(novela);
            
            Libro libro2 = new Libro("El amor en los tiempos del colera", "978-0307387264");
            autor.addLibro(libro2);
            libro2.addCategoria(ficcion);
            libro2.addCategoria(novela);
            
            // 4. Crear ficha tecnica
            FichaTecnica ficha = new FichaTecnica(libro1);
            ficha.setNumeroPaginas(496);
            ficha.setIdioma("Espanol");
            ficha.setFormato(Formato.TAPA_BLANDA);
            ficha.setPesoGramos(350);
            libro1.setFichaTecnica(ficha);
            
            em.getTransaction().commit();
            
            // 5. Consultas de verificacion
            System.out.println("\n=== VERIFICACION ===");
            
            // Recargar autor
            em.clear();
            Autor autorCargado = em.find(Autor.class, autor.getId());
            
            System.out.println("\nAutor: " + autorCargado.getNombre());
            System.out.println("Direccion: " + autorCargado.getDireccion());
            System.out.println("Premios: " + autorCargado.getPremios());
            
            System.out.println("\nLibros del autor:");
            for (Libro libro : autorCargado.getLibros()) {
                System.out.println("  - " + libro.getTitulo());
                System.out.println("    Categorias: " + 
                    libro.getCategorias().stream()
                         .map(Categoria::getNombre)
                         .collect(Collectors.joining(", ")));
                if (libro.getFichaTecnica() != null) {
                    FichaTecnica ft = libro.getFichaTecnica();
                    System.out.println("    Ficha: " + ft.getNumeroPaginas() + 
                                      " pags, " + ft.getFormato());
                }
            }
            
            // Consulta por categoria
            System.out.println("\nLibros de Realismo Magico:");
            List<Libro> librosRealismo = em.createQuery(
                "SELECT l FROM Libro l JOIN l.categorias c WHERE c.nombre = :cat", 
                Libro.class)
                .setParameter("cat", "Realismo Magico")
                .getResultList();
            librosRealismo.forEach(l -> System.out.println("  - " + l.getTitulo()));
            
        } catch (Exception e) {
            if (em.getTransaction().isActive()) {
                em.getTransaction().rollback();
            }
            e.printStackTrace();
        } finally {
            em.close();
            emf.close();
        }
    }
}

**Salida esperada:**

=== VERIFICACION ===

Autor: Gabriel Garcia Marquez
Direccion: Calle del Libro 123, 110111 Bogota, Colombia
Premios: [Premio Nobel de Literatura 1982, Premio Romulo Gallegos 1972]

Libros del autor:
  - Cien anos de soledad
    Categorias: Ficcion, Realismo Magico, Novela
    Ficha: 496 pags, TAPA_BLANDA
  - El amor en los tiempos del colera
    Categorias: Ficcion, Novela

Libros de Realismo Magico:
  - Cien anos de soledad

## Reto adicional

Implementa una relacion reflexiva en `Categoria` para permitir subcategorias (una categoria puede tener una categoria padre y multiples categorias hijas).

Consejo

Usa @ManyToOne para el padre y `@OneToMany(mappedBy = "padre")` para las hijas en la misma entidad.