## Objetivos

- Comprender que es Lombok y sus beneficios
    
- Configurar Lombok en proyectos Maven/Spring Boot
    
- Utilizar las anotaciones principales de Lombok
    
- Aplicar Lombok en entidades JPA

## 1. Introduccion a Lombok

### 1.1. Que es Lombok

**Project Lombok** es una biblioteca Java que genera automaticamente codigo repetitivo (boilerplate) en tiempo de compilacion mediante anotaciones.

Permite eliminar:

- Getters y Setters
    
- Constructores
    
- Metodos `toString()`, `equals()`, `hashCode()`
    
- Builders
    
- Loggers

### 1.2. Ventajas

|Aspecto|Sin Lombok|Con Lombok|
|---|---|---|
|Lineas de codigo|+100 lineas por clase|10-20 lineas|
|Mantenimiento|Alto|Bajo|
|Legibilidad|Ruido visual|Codigo limpio|
|Errores|Facil olvidar metodos|Generacion automatica|

## 2. Configuracion de Lombok

### 2.1. Dependencia Maven

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
    <scope>provided</scope>
</dependency>

En proyectos **Spring Boot**, la version ya esta gestionada por el parent:

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>

### 2.2. Configuracion del IDE

**IntelliJ IDEA:**

1. Instalar plugin «Lombok» desde Settings > Plugins
    
2. Habilitar «Enable annotation processing» en Settings > Build > Compiler

**Eclipse:**

1. Descargar lombok.jar
    
2. Ejecutar `java -jar lombok.jar`
    
3. Seleccionar instalacion de Eclipse

## 3. Anotaciones Principales

### 3.1. @Getter y @Setter

// A nivel de clase
@Getter
@Setter
public class Producto {
    private Long id;
    private String nombre;
    private Double precio;
}

// A nivel de campo
public class Producto {
    @Getter @Setter
    private Long id;
    
    @Getter  // Solo getter, sin setter
    private String nombre;
}

### 3.2. @ToString

@ToString
public class Producto {
    private Long id;
    private String nombre;
}
// Genera: Producto(id=1, nombre=Laptop)

// Excluir campos
@ToString(exclude = {"password", "token"})
public class Usuario { ... }

// Incluir solo campos especificos
@ToString(onlyExplicitlyIncluded = true)
public class Usuario {
    @ToString.Include
    private String email;
}

### 3.3. @EqualsAndHashCode

@EqualsAndHashCode
public class Producto {
    private Long id;
    private String nombre;
}

// Solo por ID (comun en entidades JPA)
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public class Producto {
    @EqualsAndHashCode.Include
    private Long id;
    private String nombre;
}

### 3.4. Constructores

@NoArgsConstructor      // Constructor sin argumentos
@AllArgsConstructor     // Constructor con todos los campos
@RequiredArgsConstructor // Constructor con campos final y @NonNull
public class Producto {
    private Long id;
    @NonNull
    private String nombre;
    private Double precio;
}

### 3.5. @Data (Anotacion Combinada)

`@Data` equivale a combinar:

- `@Getter` (todos los campos)
    
- `@Setter` (todos los campos no final)
    
- `@ToString`
    
- `@EqualsAndHashCode`
    
- `@RequiredArgsConstructor`

@Data
public class Producto {
    private Long id;
    private String nombre;
    private Double precio;
}

Advertencia

**Cuidado con @Data en entidades JPA:**

- `@EqualsAndHashCode` puede causar problemas con relaciones bidireccionales
    
- `@ToString` puede generar bucles infinitos en relaciones
    
- Mejor usar anotaciones individuales en entidades
    

### 3.6. @Builder

Genera el patron Builder para la clase:

@Builder
@Getter
public class Producto {
    private Long id;
    private String nombre;
    private Double precio;
    private String descripcion;
}

// Uso
Producto producto = Producto.builder()
    .nombre("Laptop HP")
    .precio(899.99)
    .descripcion("Laptop profesional")
    .build();

### 3.7. @Value (Objetos Inmutables)

Crea clases inmutables (todos los campos son `private final`):

@Value
public class ProductoDTO {
    Long id;
    String nombre;
    Double precio;
}
// Campos son final, solo getters, no setters

### 3.8. @Slf4j (Logging)

@Slf4j
@Service
public class ProductoService {
    
    public Producto crear(Producto producto) {
        log.info("Creando producto: {}", producto.getNombre());
        log.debug("Detalles: {}", producto);
        // ...
        log.error("Error al crear producto", exception);
    }
}

Otras variantes:

- `@Log` - java.util.logging
    
- `@Log4j` - Log4j 1.x
    
- `@Log4j2` - Log4j 2.x
    
- `@Slf4j` - SLF4J (recomendado con Spring Boot)

## 4. Lombok con Entidades JPA

### 4.1. Configuracion Recomendada

@Entity
@Table(name = "productos")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
@ToString(exclude = {"categoria", "proveedor"})
public class Producto {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @EqualsAndHashCode.Include
    private Long id;
    
    @Column(nullable = false)
    private String nombre;
    
    private Double precio;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "categoria_id")
    private Categoria categoria;
}

### 4.2. Buenas Practicas

|Practica|Razon|
|---|---|
|`@EqualsAndHashCode(onlyExplicitlyIncluded = true)`|Evita problemas con proxies de Hibernate|
|`@ToString(exclude = {...})`|Evita bucles infinitos en relaciones|
|`@NoArgsConstructor`|Requerido por JPA|
|Evitar `@Data` en entidades|Genera equals/hashCode problematicos|

## 5. Ejemplo Completo

### 5.1. Entidad con Lombok

@Entity
@Table(name = "categorias")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
@ToString(exclude = "productos")
public class Categoria {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @EqualsAndHashCode.Include
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String nombre;
    
    private String descripcion;
    
    @OneToMany(mappedBy = "categoria", cascade = CascadeType.ALL)
    @Builder.Default
    private List<Producto> productos = new ArrayList<>();
}

### 5.2. DTO con Lombok

@Value
@Builder
public class ProductoDTO {
    Long id;
    String nombre;
    Double precio;
    String categoriaNombre;
}

### 5.3. Servicio con Lombok

@Slf4j
@Service
@RequiredArgsConstructor
public class ProductoService {
    
    private final ProductoRepository productoRepository;
    
    public List<ProductoDTO> listarTodos() {
        log.info("Listando todos los productos");
        return productoRepository.findAll().stream()
            .map(this::convertirADTO)
            .collect(Collectors.toList());
    }
    
    private ProductoDTO convertirADTO(Producto producto) {
        return ProductoDTO.builder()
            .id(producto.getId())
            .nombre(producto.getNombre())
            .precio(producto.getPrecio())
            .categoriaNombre(producto.getCategoria().getNombre())
            .build();
    }
}

## 6. Tabla de Referencia

|Anotacion|Genera|
|---|---|
|`@Getter`|Metodos getter|
|`@Setter`|Metodos setter|
|`@ToString`|Metodo toString()|
|`@EqualsAndHashCode`|Metodos equals() y hashCode()|
|`@NoArgsConstructor`|Constructor sin argumentos|
|`@AllArgsConstructor`|Constructor con todos los campos|
|`@RequiredArgsConstructor`|Constructor con campos final/@NonNull|
|`@Data`|Todo lo anterior combinado|
|`@Value`|Clase inmutable (final)|
|`@Builder`|Patron Builder|
|`@Slf4j`|Logger SLF4J|
|`@NonNull`|Validacion null en setter/constructor|