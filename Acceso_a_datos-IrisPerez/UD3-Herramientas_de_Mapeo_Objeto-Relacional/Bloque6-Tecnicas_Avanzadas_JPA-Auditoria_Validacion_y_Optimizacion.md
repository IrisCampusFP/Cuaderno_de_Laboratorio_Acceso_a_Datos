## Objetivos de aprendizaje

Al finalizar este bloque, el alumno sera capaz de:

- Implementar auditoria automatica de entidades con Spring Data JPA
    
- Configurar campos de fecha y usuario de creacion y modificacion
    
- Aplicar validaciones declarativas con Bean Validation
    
- Crear validadores personalizados para reglas de negocio complejas
    
- Identificar y resolver el problema N+1 en consultas JPA
    
- Utilizar `@EntityGraph` para optimizar la carga de relaciones
    
- Aplicar estrategias de batching para mejorar el rendimiento
    
- Disenar proyecciones y DTOs para consultas eficientes

## Criterios de evaluacion relacionados

Este bloque mejora y complementa:

- **CE 3c**: Se han definido configuraciones de mapeo (10%)
    
- **CE 3d**: Se han aplicado mecanismos de persistencia a los objetos (25%)
    
- **CE 3e**: Se han desarrollado aplicaciones que modifican y recuperan objetos persistentes (25%)

## 1. Auditoria automatica con Spring Data JPA

### 1.1. Introduccion a la auditoria

La auditoria de entidades consiste en registrar automaticamente informacion sobre cuando y quien creo o modifico cada registro en la base de datos. Esta informacion es fundamental para:

- Cumplimiento normativo y trazabilidad
    
- Depuracion y resolucion de problemas
    
- Analisis de cambios historicos
    
- Seguridad y control de acceso

Spring Data JPA proporciona soporte integrado para auditoria mediante anotaciones que eliminan la necesidad de codigo repetitivo.

### 1.2. Configuracion de la auditoria

Para habilitar la auditoria automatica, debemos seguir tres pasos fundamentales.

#### Paso 1: Habilitar la auditoria en la configuracion

import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@Configuration
@EnableJpaAuditing
public class JpaAuditingConfig {
    // La anotacion @EnableJpaAuditing activa el mecanismo de auditoria
}

#### Paso 2: Crear una clase base auditable

import jakarta.persistence.Column;
import jakarta.persistence.EntityListeners;
import jakarta.persistence.MappedSuperclass;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class EntidadAuditable {

    @CreatedDate
    @Column(name = "fecha_creacion", nullable = false, updatable = false)
    private LocalDateTime fechaCreacion;

    @LastModifiedDate
    @Column(name = "fecha_modificacion")
    private LocalDateTime fechaModificacion;

    @CreatedBy
    @Column(name = "creado_por", updatable = false)
    private String creadoPor;

    @LastModifiedBy
    @Column(name = "modificado_por")
    private String modificadoPor;

    // Getters y setters
    public LocalDateTime getFechaCreacion() {
        return fechaCreacion;
    }

    public LocalDateTime getFechaModificacion() {
        return fechaModificacion;
    }

    public String getCreadoPor() {
        return creadoPor;
    }

    public String getModificadoPor() {
        return modificadoPor;
    }
}

#### Paso 3: Implementar AuditorAware para el usuario actual

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.domain.AuditorAware;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

import java.util.Optional;

@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaAuditingConfig {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return new AuditorAwareImpl();
    }
}

import org.springframework.data.domain.AuditorAware;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
public class AuditorAwareImpl implements AuditorAware<String> {

    @Override
    public Optional<String> getCurrentAuditor() {
        // Opcion 1: Obtener usuario de Spring Security
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        
        if (authentication == null || !authentication.isAuthenticated()) {
            return Optional.of("sistema");
        }
        
        return Optional.of(authentication.getName());
    }
}

### 1.3. Anotaciones de auditoria disponibles

|Anotacion|Descripcion|Tipo de dato|
|---|---|---|
|`@CreatedDate`|Fecha y hora de creacion|LocalDateTime, Date, Long|
|`@LastModifiedDate`|Fecha y hora de ultima modificacion|LocalDateTime, Date, Long|
|`@CreatedBy`|Usuario que creo el registro|String, Long, entidad Usuario|
|`@LastModifiedBy`|Usuario que realizo la ultima modificacion|String, Long, entidad Usuario|

### 1.4. Ejemplo completo de entidad auditable

import jakarta.persistence.*;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import java.math.BigDecimal;

@Entity
@Table(name = "productos")
@Getter
@Setter
@NoArgsConstructor
public class Producto extends EntidadAuditable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String nombre;

    @Column(length = 500)
    private String descripcion;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal precio;

    @Column(nullable = false)
    private Integer stock;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "categoria_id")
    private Categoria categoria;

    public Producto(String nombre, BigDecimal precio, Integer stock) {
        this.nombre = nombre;
        this.precio = precio;
        this.stock = stock;
    }
}

### 1.5. Auditoria sin Spring Security

En aplicaciones que no usan Spring Security, podemos implementar un mecanismo alternativo:

import org.springframework.data.domain.AuditorAware;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
public class AuditorAwareSimple implements AuditorAware<String> {

    // ThreadLocal para almacenar el usuario actual en el hilo
    private static final ThreadLocal<String> currentUser = new ThreadLocal<>();

    public static void setCurrentUser(String username) {
        currentUser.set(username);
    }

    public static void clear() {
        currentUser.remove();
    }

    @Override
    public Optional<String> getCurrentAuditor() {
        String user = currentUser.get();
        return Optional.ofNullable(user != null ? user : "anonimo");
    }
}

Uso en la aplicacion:

@Service
@RequiredArgsConstructor
public class ProductoService {

    private final ProductoRepository productoRepository;

    public Producto crearProducto(String usuario, ProductoDTO dto) {
        try {
            // Establecer el usuario antes de la operacion
            AuditorAwareSimple.setCurrentUser(usuario);
            
            Producto producto = new Producto();
            producto.setNombre(dto.getNombre());
            producto.setPrecio(dto.getPrecio());
            producto.setStock(dto.getStock());
            
            return productoRepository.save(producto);
        } finally {
            // Limpiar el ThreadLocal
            AuditorAwareSimple.clear();
        }
    }
}

### 1.6. Auditoria con JPA puro usando EntityListeners

Sin Spring Data JPA, podemos implementar auditoria usando callbacks de JPA:

import jakarta.persistence.PrePersist;
import jakarta.persistence.PreUpdate;

import java.time.LocalDateTime;

public class AuditListener {

    @PrePersist
    public void prePersist(Object entity) {
        if (entity instanceof Auditable auditable) {
            LocalDateTime now = LocalDateTime.now();
            auditable.setFechaCreacion(now);
            auditable.setFechaModificacion(now);
            auditable.setCreadoPor(obtenerUsuarioActual());
            auditable.setModificadoPor(obtenerUsuarioActual());
        }
    }

    @PreUpdate
    public void preUpdate(Object entity) {
        if (entity instanceof Auditable auditable) {
            auditable.setFechaModificacion(LocalDateTime.now());
            auditable.setModificadoPor(obtenerUsuarioActual());
        }
    }

    private String obtenerUsuarioActual() {
        // Logica para obtener el usuario actual
        return "sistema";
    }
}

// Interfaz para entidades auditables
public interface Auditable {
    void setFechaCreacion(LocalDateTime fecha);
    void setFechaModificacion(LocalDateTime fecha);
    void setCreadoPor(String usuario);
    void setModificadoPor(String usuario);
}

@Entity
@EntityListeners(AuditListener.class)
public class MiEntidad implements Auditable {
    // Implementacion de la interfaz
}

## 2. Validacion con Bean Validation

### 2.1. Introduccion a Bean Validation

Bean Validation (JSR 380) es una especificacion de Java que permite definir restricciones de validacion directamente en las clases mediante anotaciones. Hibernate Validator es la implementacion de referencia que se integra perfectamente con JPA.

Beneficios de usar Bean Validation:

- Validacion declarativa y legible
    
- Reutilizacion de reglas de validacion
    
- Mensajes de error personalizables
    
- Validacion automatica antes de persistir
    
- Grupos de validacion para diferentes contextos
    

### 2.2. Dependencia Maven

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

Para proyectos sin Spring Boot:

<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>8.0.1.Final</version>
</dependency>
<dependency>
    <groupId>org.glassfish.expressly</groupId>
    <artifactId>expressly</artifactId>
    <version>5.0.0</version>
</dependency>

### 2.3. Anotaciones de validacion principales

#### Validaciones de nulidad y vaciedad

|Anotacion|Descripcion|Aplicable a|
|---|---|---|
|`@NotNull`|No permite valores null|Cualquier tipo|
|`@NotEmpty`|No null ni vacio|String, Collection, Map, Array|
|`@NotBlank`|No null, no vacio, no solo espacios|String|
|`@Null`|Debe ser null|Cualquier tipo|

#### Validaciones de texto

|Anotacion|Descripcion|Ejemplo|
|---|---|---|
|`@Size`(min, max)|Longitud entre min y max|`@Size`(min=2, max=50)|
|`@Pattern`(regexp)|Debe coincidir con expresion regular|`@Pattern`(regexp=»[A-Z]{2}[0-9]{4}»)|
|`@Email`|Formato de email valido|`@Email`|

#### Validaciones numericas

|Anotacion|Descripcion|Ejemplo|
|---|---|---|
|`@Min`(value)|Valor minimo|`@Min`(0)|
|`@Max`(value)|Valor maximo|`@Max`(100)|
|`@Positive`|Mayor que cero|`@Positive`|
|`@PositiveOrZero`|Mayor o igual a cero|`@PositiveOrZero`|
|`@Negative`|Menor que cero|`@Negative`|
|`@NegativeOrZero`|Menor o igual a cero|`@NegativeOrZero`|
|`@Digits`(integer, fraction)|Digitos permitidos|`@Digits`(integer=6, fraction=2)|
|`@DecimalMin`(value)|Minimo decimal|`@DecimalMin`(«0.01»)|
|`@DecimalMax`(value)|Maximo decimal|`@DecimalMax`(«9999.99»)|

#### Validaciones de fechas

|Anotacion|Descripcion|
|---|---|
|`@Past`|Fecha en el pasado|
|`@PastOrPresent`|Fecha pasada o actual|
|`@Future`|Fecha en el futuro|
|`@FutureOrPresent`|Fecha futura o actual|

#### Otras validaciones

|Anotacion|Descripcion|
|---|---|
|`@AssertTrue`|Debe ser true|
|`@AssertFalse`|Debe ser false|

### 2.4. Ejemplo completo de entidad validada

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import java.math.BigDecimal;
import java.time.LocalDate;

@Entity
@Table(name = "empleados")
@Getter
@Setter
@NoArgsConstructor
public class Empleado extends EntidadAuditable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "El nombre es obligatorio")
    @Size(min = 2, max = 50, message = "El nombre debe tener entre {min} y {max} caracteres")
    @Column(nullable = false, length = 50)
    private String nombre;

    @NotBlank(message = "Los apellidos son obligatorios")
    @Size(min = 2, max = 100, message = "Los apellidos deben tener entre {min} y {max} caracteres")
    @Column(nullable = false, length = 100)
    private String apellidos;

    @NotBlank(message = "El DNI es obligatorio")
    @Pattern(regexp = "^[0-9]{8}[A-Z]$", message = "El DNI debe tener 8 digitos seguidos de una letra mayuscula")
    @Column(nullable = false, unique = true, length = 9)
    private String dni;

    @NotBlank(message = "El email es obligatorio")
    @Email(message = "El formato del email no es valido")
    @Column(nullable = false, unique = true)
    private String email;

    @Pattern(regexp = "^[6-9][0-9]{8}$", message = "El telefono debe ser un movil espanol valido")
    @Column(length = 9)
    private String telefono;

    @NotNull(message = "El salario es obligatorio")
    @Positive(message = "El salario debe ser positivo")
    @Digits(integer = 8, fraction = 2, message = "El salario debe tener maximo 8 digitos enteros y 2 decimales")
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal salario;

    @NotNull(message = "La fecha de contratacion es obligatoria")
    @PastOrPresent(message = "La fecha de contratacion no puede ser futura")
    @Column(name = "fecha_contratacion", nullable = false)
    private LocalDate fechaContratacion;

    @Future(message = "La fecha de fin de contrato debe ser futura")
    @Column(name = "fecha_fin_contrato")
    private LocalDate fechaFinContrato;

    @Min(value = 0, message = "Los dias de vacaciones no pueden ser negativos")
    @Max(value = 30, message = "Los dias de vacaciones no pueden superar 30")
    @Column(name = "dias_vacaciones")
    private Integer diasVacaciones = 22;

    @AssertTrue(message = "El empleado debe aceptar las condiciones")
    @Column(name = "condiciones_aceptadas")
    private Boolean condicionesAceptadas;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "departamento_id")
    private Departamento departamento;
}

### 2.5. Validacion en cascada

Cuando una entidad contiene otras entidades o colecciones, podemos validar en cascada usando `@Valid`:

@Entity
@Table(name = "pedidos")
@Getter
@Setter
public class Pedido extends EntidadAuditable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "El numero de pedido es obligatorio")
    @Column(name = "numero_pedido", nullable = false, unique = true)
    private String numeroPedido;

    @NotNull(message = "El cliente es obligatorio")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "cliente_id", nullable = false)
    private Cliente cliente;

    @Valid // Valida cada linea del pedido
    @NotEmpty(message = "El pedido debe tener al menos una linea")
    @OneToMany(mappedBy = "pedido", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<LineaPedido> lineas = new ArrayList<>();

    @Valid // Valida la direccion de envio
    @NotNull(message = "La direccion de envio es obligatoria")
    @Embedded
    private DireccionEnvio direccionEnvio;
}

@Embeddable
@Getter
@Setter
public class DireccionEnvio {

    @NotBlank(message = "La calle es obligatoria")
    @Size(max = 200)
    private String calle;

    @NotBlank(message = "La ciudad es obligatoria")
    @Size(max = 100)
    private String ciudad;

    @NotBlank(message = "El codigo postal es obligatorio")
    @Pattern(regexp = "^[0-9]{5}$", message = "El codigo postal debe tener 5 digitos")
    @Column(name = "codigo_postal")
    private String codigoPostal;

    @NotBlank(message = "El pais es obligatorio")
    @Size(max = 50)
    private String pais;
}

### 2.6. Grupos de validacion

Los grupos permiten aplicar diferentes validaciones segun el contexto:

// Definicion de grupos
public interface ValidacionCreacion {}
public interface ValidacionActualizacion {}

@Entity
public class Usuario {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Null(groups = ValidacionCreacion.class, message = "El ID no debe especificarse en la creacion")
    @NotNull(groups = ValidacionActualizacion.class, message = "El ID es obligatorio para actualizar")
    private Long id;

    @NotBlank(groups = {ValidacionCreacion.class, ValidacionActualizacion.class})
    private String nombre;

    @NotBlank(groups = ValidacionCreacion.class, message = "La contrasena es obligatoria al crear")
    private String password;
}

Uso en el servicio:

@Service
@RequiredArgsConstructor
public class UsuarioService {

    private final Validator validator;
    private final UsuarioRepository repository;

    public Usuario crear(Usuario usuario) {
        validar(usuario, ValidacionCreacion.class);
        return repository.save(usuario);
    }

    public Usuario actualizar(Usuario usuario) {
        validar(usuario, ValidacionActualizacion.class);
        return repository.save(usuario);
    }

    private void validar(Usuario usuario, Class<?>... grupos) {
        Set<ConstraintViolation<Usuario>> violations = validator.validate(usuario, grupos);
        if (!violations.isEmpty()) {
            throw new ConstraintViolationException(violations);
        }
    }
}

### 2.7. Validadores personalizados

Para reglas de negocio complejas, podemos crear validadores personalizados.

#### Paso 1: Crear la anotacion

import jakarta.validation.Constraint;
import jakarta.validation.Payload;

import java.lang.annotation.*;

@Documented
@Constraint(validatedBy = NifValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface NifValido {
    String message() default "El NIF no es valido";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

#### Paso 2: Implementar el validador

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

public class NifValidator implements ConstraintValidator<NifValido, String> {

    private static final String LETRAS_NIF = "TRWAGMYFPDXBNJZSQVHLCKE";

    @Override
    public void initialize(NifValido constraintAnnotation) {
        // Inicializacion si es necesaria
    }

    @Override
    public boolean isValid(String nif, ConstraintValidatorContext context) {
        if (nif == null || nif.isBlank()) {
            return true; // @NotNull/@NotBlank se encarga de esto
        }

        // Validar formato
        if (!nif.matches("^[0-9]{8}[A-Z]$")) {
            return false;
        }

        // Validar letra
        try {
            int numero = Integer.parseInt(nif.substring(0, 8));
            char letraCalculada = LETRAS_NIF.charAt(numero % 23);
            char letraProporcionada = nif.charAt(8);
            return letraCalculada == letraProporcionada;
        } catch (NumberFormatException e) {
            return false;
        }
    }
}

#### Uso del validador personalizado

@Entity
public class Persona {

    @NifValido
    @NotBlank(message = "El NIF es obligatorio")
    @Column(nullable = false, unique = true, length = 9)
    private String nif;
}

### 2.8. Validador de clase completa

Para validaciones que involucran multiples campos:

@Documented
@Constraint(validatedBy = FechasCoherentesValidator.class)
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface FechasCoherentes {
    String message() default "Las fechas no son coherentes";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    
    String fechaInicio();
    String fechaFin();
}

public class FechasCoherentesValidator 
        implements ConstraintValidator<FechasCoherentes, Object> {

    private String campoFechaInicio;
    private String campoFechaFin;

    @Override
    public void initialize(FechasCoherentes constraintAnnotation) {
        this.campoFechaInicio = constraintAnnotation.fechaInicio();
        this.campoFechaFin = constraintAnnotation.fechaFin();
    }

    @Override
    public boolean isValid(Object objeto, ConstraintValidatorContext context) {
        try {
            LocalDate fechaInicio = (LocalDate) obtenerValorCampo(objeto, campoFechaInicio);
            LocalDate fechaFin = (LocalDate) obtenerValorCampo(objeto, campoFechaFin);

            if (fechaInicio == null || fechaFin == null) {
                return true; // Otras validaciones se encargan de null
            }

            return !fechaFin.isBefore(fechaInicio);
        } catch (Exception e) {
            return false;
        }
    }

    private Object obtenerValorCampo(Object objeto, String nombreCampo) throws Exception {
        java.lang.reflect.Field field = objeto.getClass().getDeclaredField(nombreCampo);
        field.setAccessible(true);
        return field.get(objeto);
    }
}

@Entity
@FechasCoherentes(
    fechaInicio = "fechaInicio",
    fechaFin = "fechaFin",
    message = "La fecha de fin debe ser posterior a la fecha de inicio"
)
public class Proyecto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    private String nombre;

    @NotNull
    @Column(name = "fecha_inicio")
    private LocalDate fechaInicio;

    @Column(name = "fecha_fin")
    private LocalDate fechaFin;
}

### 2.9. Integracion JPA y Bean Validation

Hibernate valida automaticamente las entidades antes de las operaciones de persistencia. Este comportamiento se configura en persistence.xml o application.properties:

# Modos de validacion disponibles:
# AUTO - valida si hay un proveedor disponible (por defecto)
# CALLBACK - valida siempre, falla si no hay proveedor
# NONE - desactiva la validacion

spring.jpa.properties.jakarta.persistence.validation.mode=AUTO

# Grupos de validacion por operacion
spring.jpa.properties.jakarta.persistence.validation.group.pre-persist=jakarta.validation.groups.Default
spring.jpa.properties.jakarta.persistence.validation.group.pre-update=jakarta.validation.groups.Default
spring.jpa.properties.jakarta.persistence.validation.group.pre-remove=

### 2.10. Manejo de errores de validacion

@RestControllerAdvice
public class ValidationExceptionHandler {

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<Map<String, List<String>>> handleConstraintViolation(
            ConstraintViolationException ex) {
        
        Map<String, List<String>> errors = ex.getConstraintViolations().stream()
            .collect(Collectors.groupingBy(
                v -> v.getPropertyPath().toString(),
                Collectors.mapping(ConstraintViolation::getMessage, Collectors.toList())
            ));

        return ResponseEntity.badRequest().body(errors);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, List<String>>> handleMethodArgumentNotValid(
            MethodArgumentNotValidException ex) {
        
        Map<String, List<String>> errors = ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.groupingBy(
                FieldError::getField,
                Collectors.mapping(FieldError::getDefaultMessage, Collectors.toList())
            ));

        return ResponseEntity.badRequest().body(errors);
    }
}

## 3. Optimizacion de consultas JPA

### 3.1. El problema N+1

El problema N+1 es uno de los problemas de rendimiento mas comunes en aplicaciones JPA. Ocurre cuando una consulta inicial recupera N entidades y luego se ejecutan N consultas adicionales para cargar las relaciones de cada entidad.

#### Ejemplo del problema

@Entity
public class Autor {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    @OneToMany(mappedBy = "autor", fetch = FetchType.LAZY)
    private List<Libro> libros;
}

@Entity
public class Libro {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String titulo;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "autor_id")
    private Autor autor;
}

// Codigo que causa N+1
List<Autor> autores = autorRepository.findAll(); // 1 consulta

for (Autor autor : autores) {
    System.out.println(autor.getNombre() + " tiene " + 
        autor.getLibros().size() + " libros"); // N consultas adicionales
}

Consultas ejecutadas:

-- Consulta 1: Obtener todos los autores
SELECT * FROM autores;

-- Consulta 2: Libros del autor 1
SELECT * FROM libros WHERE autor_id = 1;

-- Consulta 3: Libros del autor 2
SELECT * FROM libros WHERE autor_id = 2;

-- ... N consultas mas

### 3.2. Solucion 1: JOIN FETCH en JPQL

public interface AutorRepository extends JpaRepository<Autor, Long> {

    @Query("SELECT DISTINCT a FROM Autor a LEFT JOIN FETCH a.libros")
    List<Autor> findAllConLibros();
}

Resultado: Una sola consulta con JOIN:

SELECT DISTINCT a.*, l.* 
FROM autores a 
LEFT JOIN libros l ON a.id = l.autor_id;

### 3.3. Solucion 2: `@EntityGraph`

`@EntityGraph` permite definir que relaciones cargar sin modificar las consultas JPQL.

#### EntityGraph con atributos

public interface AutorRepository extends JpaRepository<Autor, Long> {

    @EntityGraph(attributePaths = {"libros"})
    List<Autor> findAll();

    @EntityGraph(attributePaths = {"libros", "libros.categoria"})
    List<Autor> findAllConLibrosYCategorias();
}

#### EntityGraph nombrado

@Entity
@NamedEntityGraph(
    name = "Autor.conLibros",
    attributeNodes = @NamedAttributeNode("libros")
)
@NamedEntityGraph(
    name = "Autor.completo",
    attributeNodes = {
        @NamedAttributeNode(value = "libros", subgraph = "libros-subgraph")
    },
    subgraphs = {
        @NamedSubgraph(
            name = "libros-subgraph",
            attributeNodes = {
                @NamedAttributeNode("categoria"),
                @NamedAttributeNode("editorial")
            }
        )
    }
)
public class Autor {
    // ...
}

public interface AutorRepository extends JpaRepository<Autor, Long> {

    @EntityGraph(value = "Autor.conLibros")
    List<Autor> findByNombreContaining(String nombre);

    @EntityGraph(value = "Autor.completo")
    Optional<Autor> findById(Long id);
}

#### Tipos de EntityGraph

|Tipo|Descripcion|
|---|---|
|FETCH|Carga las relaciones especificadas como EAGER, las demas como LAZY|
|LOAD|Carga las relaciones especificadas como EAGER, las demas segun su configuracion|

@EntityGraph(attributePaths = {"libros"}, type = EntityGraph.EntityGraphType.FETCH)
List<Autor> findAllFetch();

@EntityGraph(attributePaths = {"libros"}, type = EntityGraph.EntityGraphType.LOAD)
List<Autor> findAllLoad();

### 3.4. Solucion 3: `@BatchSize`

`@BatchSize` permite cargar relaciones LAZY en lotes, reduciendo el numero de consultas:

@Entity
public class Autor {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    @OneToMany(mappedBy = "autor", fetch = FetchType.LAZY)
    @BatchSize(size = 10) // Carga libros de 10 autores por consulta
    private List<Libro> libros;
}

Con 25 autores, en lugar de 25 consultas adicionales, se ejecutan solo 3:

-- Consulta para autores 1-10
SELECT * FROM libros WHERE autor_id IN (1,2,3,4,5,6,7,8,9,10);

-- Consulta para autores 11-20
SELECT * FROM libros WHERE autor_id IN (11,12,13,14,15,16,17,18,19,20);

-- Consulta para autores 21-25
SELECT * FROM libros WHERE autor_id IN (21,22,23,24,25);

Configuracion global en application.properties:

spring.jpa.properties.hibernate.default_batch_fetch_size=25

### 3.5. Solucion 4: Proyecciones y DTOs

En lugar de cargar entidades completas, podemos proyectar solo los datos necesarios.

#### Proyeccion basada en interfaz (cerrada)

public interface AutorResumen {
    Long getId();
    String getNombre();
    int getNumeroLibros();
}

public interface AutorRepository extends JpaRepository<Autor, Long> {

    @Query("""
        SELECT a.id as id, a.nombre as nombre, SIZE(a.libros) as numeroLibros
        FROM Autor a
        """)
    List<AutorResumen> findAllResumen();
}

#### Proyeccion basada en interfaz (abierta)

public interface AutorInfo {
    String getNombre();
    
    @Value("#{target.nombre + ' (' + target.libros.size() + ' libros)'}")
    String getResumen();
}

#### Proyeccion basada en clase (DTO)

public record AutorDTO(
    Long id,
    String nombre,
    Long numeroLibros
) {}

public interface AutorRepository extends JpaRepository<Autor, Long> {

    @Query("""
        SELECT new com.ejemplo.dto.AutorDTO(
            a.id, 
            a.nombre, 
            COUNT(l)
        )
        FROM Autor a
        LEFT JOIN a.libros l
        GROUP BY a.id, a.nombre
        """)
    List<AutorDTO> findAllAsDTO();
}

#### Proyecciones dinamicas

public interface AutorRepository extends JpaRepository<Autor, Long> {

    <T> List<T> findByNombreContaining(String nombre, Class<T> type);
    
    <T> Optional<T> findById(Long id, Class<T> type);
}

Uso:

// Obtener como entidad completa
List<Autor> autores = repository.findByNombreContaining("Garcia", Autor.class);

// Obtener como DTO
List<AutorResumen> resumenes = repository.findByNombreContaining("Garcia", AutorResumen.class);

### 3.6. Comparativa de soluciones

|Solucion|Ventajas|Inconvenientes|Uso recomendado|
|---|---|---|---|
|JOIN FETCH|Control total, consulta explicita|Modifica el repositorio|Consultas especificas|
|`@EntityGraph`|Flexible, reutilizable|Complejidad con subgrafos|Multiples metodos con misma carga|
|`@BatchSize`|Minimo cambio de codigo|No elimina consultas, las agrupa|Carga diferida optimizada|
|Proyecciones|Maximo rendimiento|Requiere DTOs adicionales|APIs de solo lectura|

### 3.7. Deteccion del problema N+1

#### Usando estadisticas de Hibernate

@Configuration
public class HibernateStatsConfig {

    @Bean
    public StatisticsService statisticsService(EntityManagerFactory emf) {
        SessionFactory sessionFactory = emf.unwrap(SessionFactory.class);
        sessionFactory.getStatistics().setStatisticsEnabled(true);
        return new StatisticsServiceImpl(sessionFactory.getStatistics());
    }
}

@Service
@RequiredArgsConstructor
public class StatisticsServiceImpl implements StatisticsService {

    private final Statistics statistics;

    public void logStatistics() {
        log.info("Queries ejecutadas: {}", statistics.getQueryExecutionCount());
        log.info("Entidades cargadas: {}", statistics.getEntityLoadCount());
        log.info("Colecciones cargadas: {}", statistics.getCollectionLoadCount());
    }

    public void reset() {
        statistics.clear();
    }
}

#### Configuracion de logging

# Mostrar SQL
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Estadisticas
spring.jpa.properties.hibernate.generate_statistics=true
logging.level.org.hibernate.stat=DEBUG

# Advertencia de N+1 (desde Hibernate 5.4)
spring.jpa.properties.hibernate.session.events.log.LOG_QUERIES_SLOWER_THAN_MS=25

### 3.8. Otras optimizaciones

#### Paginacion eficiente

public interface LibroRepository extends JpaRepository<Libro, Long> {

    // Paginacion estandar
    Page<Libro> findByTituloContaining(String titulo, Pageable pageable);

    // Slice para paginacion infinita (mas eficiente que Page)
    Slice<Libro> findByAutorId(Long autorId, Pageable pageable);
    
    // Keyset pagination para grandes volumenes
    @Query("""
        SELECT l FROM Libro l 
        WHERE l.id > :lastId 
        ORDER BY l.id
        """)
    List<Libro> findNextPage(@Param("lastId") Long lastId, Pageable pageable);
}

#### Actualizaciones en masa

public interface LibroRepository extends JpaRepository<Libro, Long> {

    @Modifying
    @Query("UPDATE Libro l SET l.precio = l.precio * :factor WHERE l.categoria.id = :categoriaId")
    int actualizarPreciosPorCategoria(
        @Param("factor") BigDecimal factor, 
        @Param("categoriaId") Long categoriaId
    );

    @Modifying
    @Query("DELETE FROM Libro l WHERE l.stock = 0 AND l.fechaCreacion < :fecha")
    int eliminarLibrosAgotadosAnterioresA(@Param("fecha") LocalDateTime fecha);
}

#### Cacheo de segundo nivel

# Habilitar cache de segundo nivel
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
spring.jpa.properties.hibernate.cache.use_query_cache=true

@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Categoria {
    // Entidad que cambia poco, ideal para cachear
}

## 4. Ejercicio guiado: Sistema de gestion de inventario

Vamos a crear un sistema completo que integre auditoria, validacion y optimizacion.

### 4.1. Modelo de datos

El sistema gestiona productos, categorias, proveedores y movimientos de inventario.

### 4.2. Estructura del proyecto

📁 inventario-avanzado/
   📁 src/main/java/com/dam/inventario/
      📁 config/
         📄 JpaAuditingConfig.java
         📄 AuditorAwareImpl.java
      📁 entity/
         📁 base/
            📄 EntidadAuditable.java
         📄 Categoria.java
         📄 Proveedor.java
         📄 Producto.java
         📄 MovimientoInventario.java
      📁 dto/
         📄 ProductoResumenDTO.java
         📄 MovimientoDTO.java
         📄 EstadisticasDTO.java
      📁 repository/
         📄 CategoriaRepository.java
         📄 ProveedorRepository.java
         📄 ProductoRepository.java
         📄 MovimientoRepository.java
      📁 service/
         📄 ProductoService.java
         📄 InventarioService.java
      📁 validation/
         📄 CodigoProductoValido.java
         📄 CodigoProductoValidator.java
      📄 InventarioAvanzadoApp.java
   📁 src/main/resources/
      📄 application.properties
      📄 data.sql
   📄 pom.xml

### 4.3. Entidad base auditable

package com.dam.inventario.entity.base;

import jakarta.persistence.*;
import lombok.Getter;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter
public abstract class EntidadAuditable {

    @CreatedDate
    @Column(name = "fecha_creacion", nullable = false, updatable = false)
    private LocalDateTime fechaCreacion;

    @LastModifiedDate
    @Column(name = "fecha_modificacion")
    private LocalDateTime fechaModificacion;

    @CreatedBy
    @Column(name = "creado_por", length = 50, updatable = false)
    private String creadoPor;

    @LastModifiedBy
    @Column(name = "modificado_por", length = 50)
    private String modificadoPor;
}

### 4.4. Entidad Producto con validaciones completas

package com.dam.inventario.entity;

import com.dam.inventario.entity.base.EntidadAuditable;
import com.dam.inventario.validation.CodigoProductoValido;
import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "productos", indexes = {
    @Index(name = "idx_producto_codigo", columnList = "codigo"),
    @Index(name = "idx_producto_nombre", columnList = "nombre"),
    @Index(name = "idx_producto_categoria", columnList = "categoria_id")
})
@NamedEntityGraph(
    name = "Producto.completo",
    attributeNodes = {
        @NamedAttributeNode("categoria"),
        @NamedAttributeNode("proveedor")
    }
)
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Producto extends EntidadAuditable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CodigoProductoValido
    @NotBlank(message = "El codigo es obligatorio")
    @Column(nullable = false, unique = true, length = 20)
    private String codigo;

    @NotBlank(message = "El nombre es obligatorio")
    @Size(min = 3, max = 100, message = "El nombre debe tener entre {min} y {max} caracteres")
    @Column(nullable = false, length = 100)
    private String nombre;

    @Size(max = 500, message = "La descripcion no puede exceder {max} caracteres")
    @Column(length = 500)
    private String descripcion;

    @NotNull(message = "El precio es obligatorio")
    @Positive(message = "El precio debe ser positivo")
    @Digits(integer = 8, fraction = 2, message = "Formato de precio invalido")
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal precio;

    @NotNull(message = "El stock es obligatorio")
    @PositiveOrZero(message = "El stock no puede ser negativo")
    @Column(nullable = false)
    private Integer stock;

    @PositiveOrZero(message = "El stock minimo no puede ser negativo")
    @Column(name = "stock_minimo")
    private Integer stockMinimo = 10;

    @Column(nullable = false)
    private Boolean activo = true;

    @NotNull(message = "La categoria es obligatoria")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "categoria_id", nullable = false)
    private Categoria categoria;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "proveedor_id")
    private Proveedor proveedor;

    @OneToMany(mappedBy = "producto", cascade = CascadeType.ALL)
    @OrderBy("fechaCreacion DESC")
    @Builder.Default
    private List<MovimientoInventario> movimientos = new ArrayList<>();

    // Metodos de negocio
    public boolean necesitaReposicion() {
        return stock <= stockMinimo;
    }

    public void ajustarStock(int cantidad) {
        int nuevoStock = this.stock + cantidad;
        if (nuevoStock < 0) {
            throw new IllegalArgumentException("Stock insuficiente");
        }
        this.stock = nuevoStock;
    }
}

### 4.5. Validador personalizado para codigo de producto

package com.dam.inventario.validation;

import jakarta.validation.Constraint;
import jakarta.validation.Payload;

import java.lang.annotation.*;

@Documented
@Constraint(validatedBy = CodigoProductoValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface CodigoProductoValido {
    String message() default "El codigo de producto no tiene un formato valido";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

package com.dam.inventario.validation;

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

public class CodigoProductoValidator 
        implements ConstraintValidator<CodigoProductoValido, String> {

    // Formato: XXX-00000 (3 letras, guion, 5 digitos)
    private static final String PATRON = "^[A-Z]{3}-[0-9]{5}$";

    @Override
    public boolean isValid(String codigo, ConstraintValidatorContext context) {
        if (codigo == null || codigo.isBlank()) {
            return true; // @NotBlank se encarga
        }
        return codigo.matches(PATRON);
    }
}

### 4.6. Repositorio con optimizaciones

package com.dam.inventario.repository;

import com.dam.inventario.dto.ProductoResumenDTO;
import com.dam.inventario.entity.Producto;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.*;
import org.springframework.data.repository.query.Param;

import java.math.BigDecimal;
import java.util.List;
import java.util.Optional;

public interface ProductoRepository extends JpaRepository<Producto, Long> {

    // EntityGraph para evitar N+1
    @EntityGraph(attributePaths = {"categoria", "proveedor"})
    List<Producto> findByActivoTrue();

    @EntityGraph(value = "Producto.completo")
    Optional<Producto> findByCodigo(String codigo);

    // Proyeccion DTO para listados
    @Query("""
        SELECT new com.dam.inventario.dto.ProductoResumenDTO(
            p.id, p.codigo, p.nombre, p.precio, p.stock,
            p.categoria.nombre, p.proveedor.nombre
        )
        FROM Producto p
        LEFT JOIN p.categoria
        LEFT JOIN p.proveedor
        WHERE p.activo = true
        """)
    Page<ProductoResumenDTO> findAllResumen(Pageable pageable);

    // Busqueda con filtros
    @Query("""
        SELECT p FROM Producto p
        JOIN FETCH p.categoria c
        LEFT JOIN FETCH p.proveedor
        WHERE p.activo = true
        AND (:nombre IS NULL OR LOWER(p.nombre) LIKE LOWER(CONCAT('%', :nombre, '%')))
        AND (:categoriaId IS NULL OR c.id = :categoriaId)
        AND (:precioMin IS NULL OR p.precio >= :precioMin)
        AND (:precioMax IS NULL OR p.precio <= :precioMax)
        """)
    List<Producto> buscarConFiltros(
        @Param("nombre") String nombre,
        @Param("categoriaId") Long categoriaId,
        @Param("precioMin") BigDecimal precioMin,
        @Param("precioMax") BigDecimal precioMax
    );

    // Productos que necesitan reposicion
    @Query("SELECT p FROM Producto p WHERE p.stock <= p.stockMinimo AND p.activo = true")
    List<Producto> findProductosConStockBajo();

    // Actualizacion en masa
    @Modifying
    @Query("UPDATE Producto p SET p.precio = p.precio * :factor WHERE p.categoria.id = :categoriaId")
    int actualizarPreciosPorCategoria(
        @Param("factor") BigDecimal factor,
        @Param("categoriaId") Long categoriaId
    );

    // Estadisticas por categoria
    @Query("""
        SELECT c.nombre, COUNT(p), SUM(p.stock), AVG(p.precio)
        FROM Producto p
        JOIN p.categoria c
        WHERE p.activo = true
        GROUP BY c.id, c.nombre
        ORDER BY COUNT(p) DESC
        """)
    List<Object[]> obtenerEstadisticasPorCategoria();
}

### 4.7. Servicio con auditoria y validacion

package com.dam.inventario.service;

import com.dam.inventario.dto.ProductoResumenDTO;
import com.dam.inventario.entity.MovimientoInventario;
import com.dam.inventario.entity.Producto;
import com.dam.inventario.entity.TipoMovimiento;
import com.dam.inventario.repository.MovimientoRepository;
import com.dam.inventario.repository.ProductoRepository;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.validation.annotation.Validated;

import java.math.BigDecimal;
import java.util.List;

@Service
@RequiredArgsConstructor
@Validated
@Slf4j
public class ProductoService {

    private final ProductoRepository productoRepository;
    private final MovimientoRepository movimientoRepository;

    @Transactional(readOnly = true)
    public Page<ProductoResumenDTO> listarProductos(Pageable pageable) {
        return productoRepository.findAllResumen(pageable);
    }

    @Transactional
    public Producto crear(@Valid Producto producto) {
        log.info("Creando producto: {}", producto.getCodigo());
        Producto guardado = productoRepository.save(producto);
        
        // Registrar movimiento inicial
        registrarMovimiento(guardado, TipoMovimiento.ENTRADA, 
            producto.getStock(), "Stock inicial");
        
        return guardado;
    }

    @Transactional
    public Producto actualizar(Long id, @Valid Producto datosActualizados) {
        Producto producto = productoRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("Producto no encontrado"));
        
        producto.setNombre(datosActualizados.getNombre());
        producto.setDescripcion(datosActualizados.getDescripcion());
        producto.setPrecio(datosActualizados.getPrecio());
        producto.setStockMinimo(datosActualizados.getStockMinimo());
        producto.setCategoria(datosActualizados.getCategoria());
        producto.setProveedor(datosActualizados.getProveedor());
        
        // La auditoria se actualiza automaticamente
        return productoRepository.save(producto);
    }

    @Transactional
    public void ajustarStock(Long productoId, int cantidad, String motivo) {
        Producto producto = productoRepository.findById(productoId)
            .orElseThrow(() -> new IllegalArgumentException("Producto no encontrado"));
        
        int stockAnterior = producto.getStock();
        producto.ajustarStock(cantidad);
        
        TipoMovimiento tipo = cantidad > 0 ? TipoMovimiento.ENTRADA : TipoMovimiento.SALIDA;
        registrarMovimiento(producto, tipo, Math.abs(cantidad), motivo);
        
        log.info("Stock ajustado: {} de {} a {}", 
            producto.getCodigo(), stockAnterior, producto.getStock());
    }

    private void registrarMovimiento(Producto producto, TipoMovimiento tipo, 
                                     int cantidad, String motivo) {
        MovimientoInventario movimiento = MovimientoInventario.builder()
            .producto(producto)
            .tipo(tipo)
            .cantidad(cantidad)
            .stockResultante(producto.getStock())
            .motivo(motivo)
            .build();
        
        movimientoRepository.save(movimiento);
    }

    @Transactional(readOnly = true)
    public List<Producto> buscar(String nombre, Long categoriaId, 
                                 BigDecimal precioMin, BigDecimal precioMax) {
        return productoRepository.buscarConFiltros(nombre, categoriaId, precioMin, precioMax);
    }

    @Transactional(readOnly = true)
    public List<Producto> obtenerProductosConStockBajo() {
        return productoRepository.findProductosConStockBajo();
    }

    @Transactional
    public int actualizarPreciosPorCategoria(Long categoriaId, BigDecimal factorIncremento) {
        return productoRepository.actualizarPreciosPorCategoria(factorIncremento, categoriaId);
    }
}

## 5. Ejercicios propuestos

### Ejercicio 1: Auditoria extendida (2 puntos)

Ampliar el sistema de auditoria para incluir:

1. Un campo version con `@Version` para control optimista
    
2. Un campo ipAddress que registre la IP del cliente
    
3. Crear un AuditorAware que obtenga la IP de la peticion HTTP
    

Pistas:

- Usar RequestContextHolder de Spring para acceder a la peticion
    
- Modificar EntidadAuditable para incluir los nuevos campos
    

### Ejercicio 2: Validaciones de negocio (2 puntos)

Crear las siguientes validaciones personalizadas:

1. `@RangoPrecioValido`: Validar que el precio este entre un minimo y maximo configurable
    
2. `@StockCoherente`: Validar que stockMinimo no sea mayor que stock actual
    
3. Implementar ambas como validaciones a nivel de clase
    

### Ejercicio 3: Optimizacion de consultas (2 puntos)

En el repositorio de MovimientoInventario:

1. Crear una consulta que obtenga los ultimos 10 movimientos de cada producto usando una subquery
    
2. Implementar paginacion eficiente con keyset pagination
    
3. Crear una proyeccion que devuelva solo id, fecha, tipo y cantidad
    

### Ejercicio 4: Historial de cambios (2 puntos)

Implementar un sistema de historial de cambios:

1. Crear una entidad HistorialProducto que registre cada cambio
    
2. Usar `@PreUpdate` para capturar el estado anterior
    
3. Almacenar: campo modificado, valor anterior, valor nuevo, fecha, usuario
    

### Ejercicio 5: Dashboard de metricas (2 puntos)

Crear un servicio MetricasService que proporcione:

1. Total de productos por categoria (usando proyeccion)
    
2. Valor total del inventario (precio * stock)
    
3. Productos mas vendidos del mes (basado en movimientos de salida)
    
4. Proveedores con mas productos
    
5. Todas las consultas deben estar optimizadas para evitar N+1
    

## 6. Resumen y conceptos clave

### Auditoria automatica

- `@EnableJpaAuditing` activa el mecanismo de auditoria
    
- `@CreatedDate`, `@LastModifiedDate` registran fechas automaticamente
    
- `@CreatedBy`, `@LastModifiedBy` requieren un AuditorAware
    
- `@EntityListeners`(AuditingEntityListener.class) es obligatorio en las entidades
    

### Bean Validation

- Proporciona validacion declarativa mediante anotaciones
    
- `@NotNull`, `@NotBlank`, `@NotEmpty` para nulidad
    
- `@Size`, `@Pattern`, `@Email` para texto
    
- `@Min`, `@Max`, `@Positive` para numeros
    
- `@Valid` activa validacion en cascada
    
- Se pueden crear validadores personalizados
    

### Optimizacion de consultas

- El problema N+1 ocurre con relaciones LAZY mal gestionadas
    
- JOIN FETCH en JPQL carga relaciones en una sola consulta
    
- `@EntityGraph` define grafos de carga reutilizables
    
- `@BatchSize` agrupa consultas de colecciones LAZY
    
- Las proyecciones y DTOs mejoran el rendimiento en lecturas
    

### Buenas practicas

1. Usar EntidadAuditable como clase base para todas las entidades
    
2. Validar siempre en la capa de servicio con `@Validated`
    
3. Preferir `@EntityGraph` sobre JOIN FETCH para flexibilidad
    
4. Usar proyecciones para APIs de solo lectura
    
5. Configurar `@BatchSize` como valor por defecto global
    
6. Monitorizar consultas con estadisticas de Hibernate
    

## Referencias

- Spring Data JPA Auditing: [https://docs.spring.io/spring-data/jpa/reference/auditing.html](https://docs.spring.io/spring-data/jpa/reference/auditing.html)
    
- Hibernate Validator: [https://hibernate.org/validator/documentation/](https://hibernate.org/validator/documentation/)
    
- JPA Entity Graphs: [https://www.baeldung.com/jpa-entity-graph](https://www.baeldung.com/jpa-entity-graph)
    
- Vlad Mihalcea Blog: [https://vladmihalcea.com/](https://vladmihalcea.com/)