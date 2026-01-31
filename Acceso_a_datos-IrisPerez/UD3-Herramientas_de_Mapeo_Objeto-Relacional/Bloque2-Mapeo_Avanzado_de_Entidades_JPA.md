## Objetivos de aprendizaje

Al finalizar este bloque, el alumno será capaz de:

- Dominar las estrategias de generación de identificadores en JPA
    
- Configurar atributos de columna con precisión usando `@Column`
    
- Mapear tipos de datos especiales (fechas, objetos grandes, campos transitorios)
    
- Implementar todos los tipos de relaciones entre entidades
    
- Gestionar correctamente la cascada y el fetching de datos
    
- Aplicar estrategias de herencia en modelos de datos complejos
    
- Utilizar componentes embebidos para modelar objetos de valor

## Criterios de evaluación relacionados

Este bloque desarrolla principalmente:

- **CE 3c**: Se han definido configuraciones de mapeo (10%)
    
- **CE 3d**: Se han aplicado mecanismos de persistencia a los objetos (25%)

## 1. Anotaciones de mapeo JPA avanzadas

### 1.1. Estrategias de generación de identificadores

La anotación `@GeneratedValue` define cómo se generan automáticamente los valores de las claves primarias. JPA proporciona cuatro estrategias principales:

#### GenerationType.IDENTITY

Delega la generación del ID a la base de datos mediante columnas auto-incrementales. Es la estrategia más común con MySQL.

@Entity
@Table(name = "productos")
public class Producto {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private BigDecimal precio;
    
    // Constructor, getters y setters
}

Características de IDENTITY:

- Cada INSERT obtiene el ID de la base de datos después de ejecutarse
    
- No permite batch inserts eficientes (cada insert es individual)
    
- Funciona con MySQL AUTO_INCREMENT, PostgreSQL SERIAL, SQL Server IDENTITY
    
- El ID no está disponible hasta después del flush

#### GenerationType.SEQUENCE

Utiliza secuencias de base de datos para generar IDs. Es la estrategia preferida para PostgreSQL y Oracle.

@Entity
@Table(name = "clientes")
public class Cliente {
    
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "cliente_seq")
    @SequenceGenerator(
        name = "cliente_seq",
        sequenceName = "seq_clientes",
        initialValue = 1000,
        allocationSize = 50
    )
    private Long id;
    
    private String nombre;
    private String email;
}

Parámetros de `@SequenceGenerator`:

|Parámetro|Descripción|Valor por defecto|
|---|---|---|
|name|Nombre del generador (referenciado en generator)|Obligatorio|
|sequenceName|Nombre de la secuencia en la BD|Nombre del generador|
|initialValue|Valor inicial de la secuencia|1|
|allocationSize|Número de IDs que reserva en cada llamada|50|

El parámetro `allocationSize` es importante para el rendimiento: si es 50, Hibernate solicita 50 IDs a la secuencia y los distribuye en memoria, reduciendo las llamadas a la base de datos.

#### GenerationType.TABLE

Simula secuencias usando una tabla especial. Es la estrategia más portable pero menos eficiente.

@Entity
@Table(name = "facturas")
public class Factura {
    
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "factura_gen")
    @TableGenerator(
        name = "factura_gen",
        table = "generador_ids",
        pkColumnName = "nombre_entidad",
        valueColumnName = "ultimo_valor",
        pkColumnValue = "factura",
        allocationSize = 100
    )
    private Long id;
    
    private LocalDate fecha;
    private BigDecimal total;
}

La tabla generadora tendría esta estructura:

CREATE TABLE generador_ids (
    nombre_entidad VARCHAR(50) PRIMARY KEY,
    ultimo_valor BIGINT NOT NULL
);

#### GenerationType.AUTO

Deja que el proveedor JPA elija la estrategia más apropiada según la base de datos.

@Entity
public class Entidad {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
}

En Hibernate 6 con MySQL, AUTO suele traducirse a SEQUENCE (usando una secuencia simulada con tabla). Esto puede causar confusión si esperas IDENTITY.

#### GenerationType.UUID (JPA 3.1+)

A partir de JPA 3.1, existe soporte nativo para UUIDs:

@Entity
public class Documento {
    
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    
    private String titulo;
    private byte[] contenido;
}

Para versiones anteriores, puedes usar:

@Entity
public class Documento {
    
    @Id
    @GeneratedValue(generator = "uuid2")
    @GenericGenerator(name = "uuid2", strategy = "uuid2")
    @Column(columnDefinition = "BINARY(16)")
    private UUID id;
}

#### Comparativa de estrategias

|Estrategia|Ventajas|Inconvenientes|Uso recomendado|
|---|---|---|---|
|IDENTITY|Simple, nativo de BD|No permite batch insert|MySQL en aplicaciones simples|
|SEQUENCE|Eficiente, permite batch|No todas las BD lo soportan|PostgreSQL, Oracle|
|TABLE|Portable|Lento, bloqueos|Solo si se necesita portabilidad total|
|AUTO|Automático|Impredecible|Prototipos rápidos|
|UUID|Distribuido, sin colisiones|Mayor tamaño, no ordenado|Sistemas distribuidos|

### 1.2. Configuración detallada de columnas con `@Column`

La anotación `@Column` permite un control preciso sobre cómo se mapea un atributo a una columna de la base de datos.

@Entity
@Table(name = "empleados")
public class Empleado {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(
        name = "nombre_completo",
        nullable = false,
        length = 150
    )
    private String nombreCompleto;
    
    @Column(
        name = "email",
        nullable = false,
        unique = true,
        length = 100
    )
    private String email;
    
    @Column(
        name = "salario",
        nullable = false,
        precision = 10,
        scale = 2
    )
    private BigDecimal salario;
    
    @Column(
        name = "activo",
        nullable = false,
        columnDefinition = "BOOLEAN DEFAULT TRUE"
    )
    private Boolean activo;
    
    @Column(
        name = "codigo_interno",
        insertable = false,
        updatable = false
    )
    private String codigoInterno;
}

Atributos de `@Column`:

|Atributo|Tipo|Descripción|Valor por defecto|
|---|---|---|---|
|name|String|Nombre de la columna en la BD|Nombre del atributo|
|nullable|boolean|Permite valores NULL|true|
|unique|boolean|Restricción UNIQUE|false|
|length|int|Longitud máxima (solo String)|255|
|precision|int|Dígitos totales (BigDecimal)|0 (sin límite)|
|scale|int|Dígitos decimales (BigDecimal)|0|
|insertable|boolean|Incluir en INSERT|true|
|updatable|boolean|Incluir en UPDATE|true|
|columnDefinition|String|DDL exacto de la columna|(generado)|

### 1.3. Mapeo de fechas y tiempos

#### API moderna de Java (java.time)

Desde JPA 2.2, los tipos de `java.time` se mapean automáticamente:

@Entity
@Table(name = "eventos")
public class Evento {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    // Solo fecha (DATE en SQL)
    private LocalDate fechaEvento;
    
    // Solo hora (TIME en SQL)
    private LocalTime horaInicio;
    
    // Fecha y hora sin zona (DATETIME/TIMESTAMP en SQL)
    private LocalDateTime fechaHoraCreacion;
    
    // Fecha y hora con zona horaria
    private OffsetDateTime fechaHoraConZona;
    
    // Instante en el tiempo (TIMESTAMP en SQL)
    private Instant instanteRegistro;
    
    // Duración (almacenada como BIGINT en nanosegundos)
    private Duration duracionEvento;
}

Correspondencia de tipos:

|Tipo Java|Tipo SQL típico|Descripción|
|---|---|---|
|LocalDate|DATE|Fecha sin hora|
|LocalTime|TIME|Hora sin fecha|
|LocalDateTime|DATETIME/TIMESTAMP|Fecha y hora local|
|OffsetDateTime|TIMESTAMP WITH TIME ZONE|Con desplazamiento de zona|
|ZonedDateTime|TIMESTAMP WITH TIME ZONE|Con zona horaria completa|
|Instant|TIMESTAMP|Momento específico en UTC|

#### API legacy (java.util.Date y Calendar)

Para código legacy que usa `java.util.Date` o `Calendar`, se requiere `@Temporal`:

@Entity
@Table(name = "registros_legacy")
public class RegistroLegacy {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // Solo fecha
    @Temporal(TemporalType.DATE)
    private Date fechaNacimiento;
    
    // Solo hora
    @Temporal(TemporalType.TIME)
    private Date horaEntrada;
    
    // Fecha y hora completa
    @Temporal(TemporalType.TIMESTAMP)
    private Date fechaHoraRegistro;
    
    @Temporal(TemporalType.TIMESTAMP)
    private Calendar ultimaModificacion;
}

Recomendación: Siempre que sea posible, utiliza la API `java.time` de Java 8+ en lugar de `java.util.Date`. Es más expresiva, inmutable y no requiere anotaciones adicionales.

### 1.4. Campos grandes con `@Lob`

La anotación `@Lob` (Large Object) se usa para almacenar datos de gran tamaño:

@Entity
@Table(name = "documentos")
public class Documento {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String titulo;
    
    // CLOB - Character Large Object (textos grandes)
    @Lob
    @Column(name = "contenido_texto")
    private String contenidoTexto;
    
    // BLOB - Binary Large Object (archivos binarios)
    @Lob
    @Column(name = "archivo_binario")
    private byte[] archivoBinario;
    
    @Column(name = "tipo_mime", length = 100)
    private String tipoMime;
    
    @Column(name = "nombre_archivo", length = 255)
    private String nombreArchivo;
}

Consideraciones sobre `@Lob`:

1. El tipo SQL depende del tipo Java: String produce CLOB, byte[] produce BLOB
    
2. Los LOBs suelen cargarse de forma lazy por defecto en algunas implementaciones
    
3. Para archivos muy grandes, considera almacenar la ruta en BD y el archivo en el sistema de ficheros
    
4. El rendimiento puede verse afectado si se cargan LOBs innecesariamente
    

Ejemplo de uso con archivos:

// Guardar un documento con archivo adjunto
public void guardarDocumento(String titulo, Path archivo) throws IOException {
    Documento doc = new Documento();
    doc.setTitulo(titulo);
    doc.setNombreArchivo(archivo.getFileName().toString());
    doc.setTipoMime(Files.probeContentType(archivo));
    doc.setArchivoBinario(Files.readAllBytes(archivo));
    
    entityManager.persist(doc);
}

// Recuperar y escribir a fichero
public void exportarArchivo(Long documentoId, Path destino) throws IOException {
    Documento doc = entityManager.find(Documento.class, documentoId);
    if (doc != null && doc.getArchivoBinario() != null) {
        Files.write(destino, doc.getArchivoBinario());
    }
}

### 1.5. Campos transitorios con `@Transient`

Los campos marcados con `@Transient` no se persisten en la base de datos:

@Entity
@Table(name = "productos")
public class Producto {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal precioBase;
    
    @Column(precision = 5, scale = 2)
    private BigDecimal porcentajeIva;
    
    // Campo calculado - no se persiste
    @Transient
    private BigDecimal precioConIva;
    
    // Campo auxiliar para la aplicación
    @Transient
    private boolean seleccionado;
    
    // Método para calcular el precio con IVA
    public BigDecimal getPrecioConIva() {
        if (precioBase == null || porcentajeIva == null) {
            return BigDecimal.ZERO;
        }
        BigDecimal multiplicador = BigDecimal.ONE.add(
            porcentajeIva.divide(new BigDecimal("100"), 4, RoundingMode.HALF_UP)
        );
        return precioBase.multiply(multiplicador).setScale(2, RoundingMode.HALF_UP);
    }
    
    // Callback para calcular después de cargar de la BD
    @PostLoad
    private void calcularCamposDerivados() {
        this.precioConIva = getPrecioConIva();
    }
}

Diferencias entre `@Transient` y transient:

|Aspecto|`@Transient` (JPA)|transient (Java)|
|---|---|---|
|Ámbito|Solo persistencia JPA|Serialización Java|
|El campo se serializa|Sí|No|
|El campo se persiste en BD|No|Depende del proveedor|
|Uso recomendado|Campos calculados, auxiliares|Campos sensibles|

### 1.6. Conversores de tipos con `@Convert`

Los conversores permiten transformar tipos Java a tipos de base de datos de forma personalizada:

// Definir el conversor
@Converter
public class EstadoPedidoConverter implements AttributeConverter<EstadoPedido, String> {
    
    @Override
    public String convertToDatabaseColumn(EstadoPedido estado) {
        if (estado == null) {
            return null;
        }
        return estado.getCodigo();
    }
    
    @Override
    public EstadoPedido convertToEntityAttribute(String codigo) {
        if (codigo == null) {
            return null;
        }
        return EstadoPedido.fromCodigo(codigo);
    }
}

// El enum
public enum EstadoPedido {
    PENDIENTE("P"),
    PROCESANDO("PR"),
    ENVIADO("E"),
    ENTREGADO("EN"),
    CANCELADO("C");
    
    private final String codigo;
    
    EstadoPedido(String codigo) {
        this.codigo = codigo;
    }
    
    public String getCodigo() {
        return codigo;
    }
    
    public static EstadoPedido fromCodigo(String codigo) {
        for (EstadoPedido estado : values()) {
            if (estado.codigo.equals(codigo)) {
                return estado;
            }
        }
        throw new IllegalArgumentException("Código desconocido: " + codigo);
    }
}

// Usar el conversor en la entidad
@Entity
@Table(name = "pedidos")
public class Pedido {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Convert(converter = EstadoPedidoConverter.class)
    @Column(name = "estado", length = 2)
    private EstadoPedido estado;
    
    private LocalDateTime fechaPedido;
}

Para aplicar un conversor automáticamente a todos los campos de un tipo:

@Converter(autoApply = true)
public class EstadoPedidoConverter implements AttributeConverter<EstadoPedido, String> {
    // ... implementación
}

Otros usos comunes de conversores:

// Conversor para Boolean a S/N
@Converter
public class BooleanSiNoConverter implements AttributeConverter<Boolean, String> {
    
    @Override
    public String convertToDatabaseColumn(Boolean valor) {
        return valor != null && valor ? "S" : "N";
    }
    
    @Override
    public Boolean convertToEntityAttribute(String valor) {
        return "S".equals(valor);
    }
}

// Conversor para lista de tags separados por comas
@Converter
public class ListaStringsConverter implements AttributeConverter<List<String>, String> {
    
    @Override
    public String convertToDatabaseColumn(List<String> lista) {
        if (lista == null || lista.isEmpty()) {
            return null;
        }
        return String.join(",", lista);
    }
    
    @Override
    public List<String> convertToEntityAttribute(String valor) {
        if (valor == null || valor.isEmpty()) {
            return new ArrayList<>();
        }
        return new ArrayList<>(Arrays.asList(valor.split(",")));
    }
}

## 2. Relaciones entre entidades

Las relaciones representan cómo se conectan las entidades en el modelo de dominio. JPA proporciona cuatro tipos de relaciones.

### 2.1. Relación `@OneToOne`

Representa una correspondencia uno a uno entre dos entidades.

#### OneToOne unidireccional

@Entity
@Table(name = "empleados")
public class Empleado {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    @OneToOne(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "parking_id", referencedColumnName = "id")
    private PlazaParking plazaParking;
}

@Entity
@Table(name = "plazas_parking")
public class PlazaParking {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String numero;
    
    private String ubicacion;
}

La tabla empleados tendrá una columna `parking_id` como clave foránea.

#### OneToOne bidireccional

@Entity
@Table(name = "usuarios")
public class Usuario {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String username;
    
    @OneToOne(
        mappedBy = "usuario",
        cascade = CascadeType.ALL,
        orphanRemoval = true,
        fetch = FetchType.LAZY
    )
    private PerfilUsuario perfil;
    
    // Método de conveniencia para sincronizar ambos lados
    public void setPerfil(PerfilUsuario perfil) {
        if (perfil == null) {
            if (this.perfil != null) {
                this.perfil.setUsuario(null);
            }
        } else {
            perfil.setUsuario(this);
        }
        this.perfil = perfil;
    }
}

@Entity
@Table(name = "perfiles_usuario")
public class PerfilUsuario {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String biografia;
    private String avatarUrl;
    private LocalDate fechaNacimiento;
    
    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "usuario_id", nullable = false)
    private Usuario usuario;
}

Conceptos importantes:

- `mappedBy`: Indica que la otra entidad (PerfilUsuario) es la propietaria de la relación y contiene la clave foránea
    
- La entidad que NO tiene mappedBy es la propietaria (la que tiene `@JoinColumn`)
    
- Los métodos de conveniencia aseguran la sincronización de ambos lados
    

#### OneToOne con clave primaria compartida

Optimización donde ambas entidades comparten la misma clave primaria:

@Entity
@Table(name = "usuarios")
public class Usuario {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    
    @OneToOne(mappedBy = "usuario", cascade = CascadeType.ALL)
    @PrimaryKeyJoinColumn
    private DetallesUsuario detalles;
}

@Entity
@Table(name = "detalles_usuario")
public class DetallesUsuario {
    
    @Id
    private Long id;  // Mismo ID que Usuario
    
    private String direccion;
    private String telefono;
    
    @OneToOne
    @MapsId  // Indica que el ID se mapea desde la relación
    @JoinColumn(name = "id")
    private Usuario usuario;
}

### 2.2. Relación `@ManyToOne` y `@OneToMany`

Es la relación más común. Representa cardinalidad de muchos a uno.

#### Relación unidireccional ManyToOne

La forma más simple: solo el lado «muchos» conoce la relación.

@Entity
@Table(name = "libros")
public class Libro {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String titulo;
    private String isbn;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "editorial_id", nullable = false)
    private Editorial editorial;
}

@Entity
@Table(name = "editoriales")
public class Editorial {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private String pais;
}

#### Relación bidireccional OneToMany/ManyToOne

Ambas entidades conocen la relación:

@Entity
@Table(name = "departamentos")
public class Departamento {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String nombre;
    
    @OneToMany(
        mappedBy = "departamento",
        cascade = CascadeType.ALL,
        orphanRemoval = true
    )
    private List<Empleado> empleados = new ArrayList<>();
    
    // Métodos de conveniencia
    public void addEmpleado(Empleado empleado) {
        empleados.add(empleado);
        empleado.setDepartamento(this);
    }
    
    public void removeEmpleado(Empleado empleado) {
        empleados.remove(empleado);
        empleado.setDepartamento(null);
    }
}

@Entity
@Table(name = "empleados")
public class Empleado {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private String email;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "departamento_id")
    private Departamento departamento;
    
    // equals y hashCode basados en clave de negocio
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Empleado)) return false;
        Empleado empleado = (Empleado) o;
        return email != null && email.equals(empleado.email);
    }
    
    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}

Puntos importantes:

1. Siempre usar `mappedBy` en `@OneToMany` para evitar tablas de unión innecesarias
    
2. El lado `@ManyToOne` es el propietario de la relación (tiene la FK)
    
3. Los métodos de conveniencia mantienen la consistencia bidireccional
    
4. `orphanRemoval = true` elimina empleados huérfanos cuando se quitan de la lista
    

### 2.3. Relación `@ManyToMany`

Representa relaciones de muchos a muchos, que requieren una tabla intermedia.

#### ManyToMany básica

@Entity
@Table(name = "estudiantes")
public class Estudiante {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private String matricula;
    
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "estudiantes_asignaturas",
        joinColumns = @JoinColumn(name = "estudiante_id"),
        inverseJoinColumns = @JoinColumn(name = "asignatura_id")
    )
    private Set<Asignatura> asignaturas = new HashSet<>();
    
    // Métodos de conveniencia
    public void matricularEn(Asignatura asignatura) {
        this.asignaturas.add(asignatura);
        asignatura.getEstudiantes().add(this);
    }
    
    public void desmatricularDe(Asignatura asignatura) {
        this.asignaturas.remove(asignatura);
        asignatura.getEstudiantes().remove(this);
    }
}

@Entity
@Table(name = "asignaturas")
public class Asignatura {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String nombre;
    
    private Integer creditos;
    
    @ManyToMany(mappedBy = "asignaturas")
    private Set<Estudiante> estudiantes = new HashSet<>();
}

La tabla intermedia generada:

CREATE TABLE estudiantes_asignaturas (
    estudiante_id BIGINT NOT NULL,
    asignatura_id BIGINT NOT NULL,
    PRIMARY KEY (estudiante_id, asignatura_id),
    FOREIGN KEY (estudiante_id) REFERENCES estudiantes(id),
    FOREIGN KEY (asignatura_id) REFERENCES asignaturas(id)
);

#### ManyToMany con atributos adicionales

Cuando la relación necesita campos propios, debemos modelarla como entidad:

// Entidad intermedia con atributos adicionales
@Entity
@Table(name = "inscripciones")
public class Inscripcion {
    
    @EmbeddedId
    private InscripcionId id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("estudianteId")
    @JoinColumn(name = "estudiante_id")
    private Estudiante estudiante;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("asignaturaId")
    @JoinColumn(name = "asignatura_id")
    private Asignatura asignatura;
    
    // Atributos propios de la relación
    private LocalDate fechaInscripcion;
    
    @Column(precision = 4, scale = 2)
    private BigDecimal calificacion;
    
    @Enumerated(EnumType.STRING)
    private EstadoInscripcion estado;
    
    // Constructor
    public Inscripcion() {}
    
    public Inscripcion(Estudiante estudiante, Asignatura asignatura) {
        this.estudiante = estudiante;
        this.asignatura = asignatura;
        this.id = new InscripcionId(estudiante.getId(), asignatura.getId());
        this.fechaInscripcion = LocalDate.now();
        this.estado = EstadoInscripcion.ACTIVA;
    }
}

// Clave primaria compuesta
@Embeddable
public class InscripcionId implements Serializable {
    
    @Column(name = "estudiante_id")
    private Long estudianteId;
    
    @Column(name = "asignatura_id")
    private Long asignaturaId;
    
    public InscripcionId() {}
    
    public InscripcionId(Long estudianteId, Long asignaturaId) {
        this.estudianteId = estudianteId;
        this.asignaturaId = asignaturaId;
    }
    
    // equals y hashCode obligatorios
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof InscripcionId)) return false;
        InscripcionId that = (InscripcionId) o;
        return Objects.equals(estudianteId, that.estudianteId) &&
               Objects.equals(asignaturaId, that.asignaturaId);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(estudianteId, asignaturaId);
    }
}

public enum EstadoInscripcion {
    ACTIVA, FINALIZADA, ABANDONADA
}

Las entidades principales ahora referencian la entidad intermedia:

@Entity
@Table(name = "estudiantes")
public class Estudiante {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    @OneToMany(mappedBy = "estudiante", cascade = CascadeType.ALL, orphanRemoval = true)
    private Set<Inscripcion> inscripciones = new HashSet<>();
    
    public void inscribirEn(Asignatura asignatura) {
        Inscripcion inscripcion = new Inscripcion(this, asignatura);
        inscripciones.add(inscripcion);
        asignatura.getInscripciones().add(inscripcion);
    }
}

### 2.4. Cascada de operaciones

La cascada determina qué operaciones se propagan automáticamente a las entidades relacionadas.

@Entity
public class Pedido {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToMany(
        mappedBy = "pedido",
        cascade = CascadeType.ALL,  // Propaga todas las operaciones
        orphanRemoval = true
    )
    private List<LineaPedido> lineas = new ArrayList<>();
}

Tipos de CascadeType:

|Tipo|Operación|Descripción|
|---|---|---|
|PERSIST|persist()|Al persistir el padre, persiste los hijos|
|MERGE|merge()|Al fusionar el padre, fusiona los hijos|
|REMOVE|remove()|Al eliminar el padre, elimina los hijos|
|REFRESH|refresh()|Al refrescar el padre, refresca los hijos|
|DETACH|detach()|Al desconectar el padre, desconecta los hijos|
|ALL|Todas|Equivale a todos los anteriores|

Ejemplo práctico de cascada:

// Con CascadeType.PERSIST
Pedido pedido = new Pedido();
pedido.setFecha(LocalDate.now());

LineaPedido linea1 = new LineaPedido();
linea1.setProducto("Producto A");
linea1.setCantidad(2);
linea1.setPedido(pedido);

LineaPedido linea2 = new LineaPedido();
linea2.setProducto("Producto B");
linea2.setCantidad(1);
linea2.setPedido(pedido);

pedido.getLineas().add(linea1);
pedido.getLineas().add(linea2);

// Solo persistimos el pedido, las líneas se persisten automáticamente
entityManager.persist(pedido);

#### orphanRemoval

`orphanRemoval = true` elimina automáticamente las entidades hijas que quedan «huérfanas»:

// Las líneas eliminadas de la lista se borran de la BD
pedido.getLineas().remove(0);  // La primera línea se eliminará al hacer flush

// También funciona al reemplazar toda la colección
pedido.setLineas(nuevasLineas);  // Las líneas antiguas se eliminan

Diferencia entre CascadeType.REMOVE y orphanRemoval:

- `CascadeType.REMOVE`: Solo elimina hijos cuando se elimina el padre
    
- `orphanRemoval`: Elimina hijos cuando se quitan de la colección O cuando se elimina el padre
    

### 2.5. Tipos de Fetch: LAZY vs EAGER

FetchType determina cuándo se cargan las entidades relacionadas.

@Entity
public class Autor {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    // LAZY: Los libros se cargan solo cuando se accede a la colección
    @OneToMany(mappedBy = "autor", fetch = FetchType.LAZY)
    private List<Libro> libros = new ArrayList<>();
}

@Entity
public class Libro {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String titulo;
    
    // EAGER: El autor se carga junto con el libro
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "autor_id")
    private Autor autor;
}

Valores por defecto:

|Tipo de relación|FetchType por defecto|
|---|---|
|`@OneToOne`|EAGER|
|`@ManyToOne`|EAGER|
|`@OneToMany`|LAZY|
|`@ManyToMany`|LAZY|

Recomendación: Usa siempre LAZY por defecto y carga datos cuando los necesites:

@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "autor_id")
private Autor autor;

#### El problema N+1

El problema N+1 ocurre cuando se ejecutan N consultas adicionales para cargar entidades relacionadas:

// Problema N+1: 1 consulta para libros + N consultas para autores
List<Libro> libros = libroRepository.findAll();  // 1 consulta
for (Libro libro : libros) {
    System.out.println(libro.getAutor().getNombre());  // N consultas
}

Soluciones al problema N+1:

// Solución 1: JOIN FETCH en JPQL
@Query("SELECT l FROM Libro l JOIN FETCH l.autor")
List<Libro> findAllWithAutor();

// Solución 2: EntityGraph
@EntityGraph(attributePaths = {"autor"})
List<Libro> findAll();

// Solución 3: Hibernate @BatchSize
@Entity
public class Autor {
    
    @BatchSize(size = 25)  // Carga hasta 25 autores por consulta
    @OneToMany(mappedBy = "autor")
    private List<Libro> libros;
}

## 3. Herencia en JPA

JPA ofrece varias estrategias para mapear jerarquías de clases a tablas de base de datos.

### 3.1. Estrategia SINGLE_TABLE

Todas las clases de la jerarquía se almacenan en una única tabla, con una columna discriminadora.

@Entity
@Table(name = "vehiculos")
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(
    name = "tipo_vehiculo",
    discriminatorType = DiscriminatorType.STRING
)
public abstract class Vehiculo {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String marca;
    private String modelo;
    private Integer anio;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal precio;
}

@Entity
@DiscriminatorValue("COCHE")
public class Coche extends Vehiculo {
    
    private Integer numeroPuertas;
    private String tipoCombustible;
    private Integer potenciaCv;
}

@Entity
@DiscriminatorValue("MOTO")
public class Moto extends Vehiculo {
    
    private Integer cilindrada;
    private String tipoCarnet;  // A1, A2, A
}

@Entity
@DiscriminatorValue("CAMION")
public class Camion extends Vehiculo {
    
    private BigDecimal capacidadCarga;  // Toneladas
    private Integer numeroEjes;
    private Boolean tieneRemolque;
}

Tabla generada:

CREATE TABLE vehiculos (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    tipo_vehiculo VARCHAR(31) NOT NULL,  -- Discriminador
    marca VARCHAR(255),
    modelo VARCHAR(255),
    anio INT,
    precio DECIMAL(10,2),
    -- Columnas de Coche
    numero_puertas INT,
    tipo_combustible VARCHAR(255),
    potencia_cv INT,
    -- Columnas de Moto
    cilindrada INT,
    tipo_carnet VARCHAR(255),
    -- Columnas de Camion
    capacidad_carga DECIMAL(19,2),
    numero_ejes INT,
    tiene_remolque BOOLEAN
);

Ventajas:

- Consultas muy eficientes (sin JOINs)
    
- Polimorfismo simple y rápido
    

Inconvenientes:

- Columnas específicas de subclases no pueden ser NOT NULL
    
- Tabla puede crecer con muchas columnas vacías
    
- Rompe la normalización de la base de datos
    

### 3.2. Estrategia JOINED

Cada clase tiene su propia tabla. Las tablas hijas referencian a la tabla padre.

@Entity
@Table(name = "personas")
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Persona {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private String apellidos;
    private String dni;
    private LocalDate fechaNacimiento;
}

@Entity
@Table(name = "empleados")
@PrimaryKeyJoinColumn(name = "persona_id")
public class Empleado extends Persona {
    
    private String numeroEmpleado;
    private LocalDate fechaContratacion;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal salario;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "departamento_id")
    private Departamento departamento;
}

@Entity
@Table(name = "clientes")
@PrimaryKeyJoinColumn(name = "persona_id")
public class Cliente extends Persona {
    
    private String numeroCliente;
    private String direccion;
    private String telefono;
    
    @Column(precision = 15, scale = 2)
    private BigDecimal limiteCredito;
}

@Entity
@Table(name = "proveedores")
@PrimaryKeyJoinColumn(name = "persona_id")
public class Proveedor extends Persona {
    
    private String cif;
    private String razonSocial;
    private String contacto;
    private Integer diasPago;
}

Tablas generadas:

CREATE TABLE personas (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(255),
    apellidos VARCHAR(255),
    dni VARCHAR(255),
    fecha_nacimiento DATE
);

CREATE TABLE empleados (
    persona_id BIGINT PRIMARY KEY,
    numero_empleado VARCHAR(255),
    fecha_contratacion DATE,
    salario DECIMAL(10,2),
    departamento_id BIGINT,
    FOREIGN KEY (persona_id) REFERENCES personas(id),
    FOREIGN KEY (departamento_id) REFERENCES departamentos(id)
);

CREATE TABLE clientes (
    persona_id BIGINT PRIMARY KEY,
    numero_cliente VARCHAR(255),
    direccion VARCHAR(255),
    telefono VARCHAR(255),
    limite_credito DECIMAL(15,2),
    FOREIGN KEY (persona_id) REFERENCES personas(id)
);

Ventajas:

- Base de datos normalizada
    
- Las columnas específicas pueden ser NOT NULL
    
- Sin datos duplicados
    

Inconvenientes:

- Consultas polimórficas requieren JOINs (más lentas)
    
- Inserciones y actualizaciones tocan múltiples tablas
    

### 3.3. Estrategia TABLE_PER_CLASS

Cada clase concreta tiene su propia tabla completa con todos los campos.

@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Pago {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)  // No puede ser IDENTITY
    private Long id;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal importe;
    
    private LocalDateTime fechaPago;
    private String referencia;
}

@Entity
@Table(name = "pagos_tarjeta")
public class PagoTarjeta extends Pago {
    
    private String numeroTarjeta;  // Últimos 4 dígitos
    private String titular;
    private String entidadBancaria;
    private String codigoAutorizacion;
}

@Entity
@Table(name = "pagos_transferencia")
public class PagoTransferencia extends Pago {
    
    private String ibanOrigen;
    private String ibanDestino;
    private String concepto;
}

@Entity
@Table(name = "pagos_efectivo")
public class PagoEfectivo extends Pago {
    
    @Column(precision = 10, scale = 2)
    private BigDecimal importeEntregado;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal cambioDevuelto;
}

Tablas generadas:

CREATE TABLE pagos_tarjeta (
    id BIGINT PRIMARY KEY,
    importe DECIMAL(10,2),
    fecha_pago DATETIME,
    referencia VARCHAR(255),
    numero_tarjeta VARCHAR(255),
    titular VARCHAR(255),
    entidad_bancaria VARCHAR(255),
    codigo_autorizacion VARCHAR(255)
);

CREATE TABLE pagos_transferencia (
    id BIGINT PRIMARY KEY,
    importe DECIMAL(10,2),
    fecha_pago DATETIME,
    referencia VARCHAR(255),
    iban_origen VARCHAR(255),
    iban_destino VARCHAR(255),
    concepto VARCHAR(255)
);

CREATE TABLE pagos_efectivo (
    id BIGINT PRIMARY KEY,
    importe DECIMAL(10,2),
    fecha_pago DATETIME,
    referencia VARCHAR(255),
    importe_entregado DECIMAL(10,2),
    cambio_devuelto DECIMAL(10,2)
);

Ventajas:

- Cada tabla es independiente y completa
    
- No hay JOINs para consultas de una clase específica
    
- Las columnas pueden ser NOT NULL
    

Inconvenientes:

- Consultas polimórficas requieren UNION de todas las tablas
    
- No se puede usar GenerationType.IDENTITY
    
- Cambios en la clase padre afectan a todas las tablas
    

### 3.4. `@MappedSuperclass`

Para compartir atributos comunes sin crear una tabla para la clase padre.

@MappedSuperclass
public abstract class EntidadAuditable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "fecha_creacion", updatable = false)
    private LocalDateTime fechaCreacion;
    
    @Column(name = "fecha_modificacion")
    private LocalDateTime fechaModificacion;
    
    @Column(name = "creado_por", length = 50, updatable = false)
    private String creadoPor;
    
    @Column(name = "modificado_por", length = 50)
    private String modificadoPor;
    
    @PrePersist
    protected void onCreate() {
        fechaCreacion = LocalDateTime.now();
        fechaModificacion = fechaCreacion;
    }
    
    @PreUpdate
    protected void onUpdate() {
        fechaModificacion = LocalDateTime.now();
    }
    
    // Getters y setters
}

@Entity
@Table(name = "productos")
public class Producto extends EntidadAuditable {
    
    private String nombre;
    private String descripcion;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal precio;
}

@Entity
@Table(name = "categorias")
public class Categoria extends EntidadAuditable {
    
    private String nombre;
    private String descripcion;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "categoria_padre_id")
    private Categoria categoriaPadre;
}

Características de `@MappedSuperclass`:

- No es una entidad (no tiene tabla propia)
    
- No se pueden hacer consultas sobre ella
    
- Solo sirve para heredar atributos y mappings
    
- Las subclases obtienen todos los campos en sus propias tablas
    

### 3.5. Comparativa de estrategias de herencia

|Criterio|SINGLE_TABLE|JOINED|TABLE_PER_CLASS|
|---|---|---|---|
|Consulta polimórfica|Muy rápida|Lenta (JOINs)|Muy lenta (UNION)|
|Consulta específica|Rápida|Rápida|Rápida|
|Normalización|Baja|Alta|Media|
|NOT NULL en subclases|No|Sí|Sí|
|Espacio en disco|Mayor (nulls)|Óptimo|Mayor (duplicación)|
|Mantenibilidad|Baja|Alta|Media|
|Integridad referencial|Completa|Completa|Limitada|

Recomendaciones:

- **SINGLE_TABLE**: Pocas subclases con pocos atributos propios, rendimiento crítico
    
- **JOINED**: Modelo normalizado, muchas subclases, integridad importante
    
- **TABLE_PER_CLASS**: Subclases muy diferentes, pocas consultas polimórficas
    
- **`@MappedSuperclass`**: Solo compartir código, sin polimorfismo en BD
    

## 4. Componentes embebidos

Los componentes embebidos permiten agrupar atributos relacionados sin crear entidades separadas.

### 4.1. `@Embeddable` y `@Embedded`

@Embeddable
public class Direccion {
    
    @Column(length = 200)
    private String calle;
    
    @Column(length = 10)
    private String numero;
    
    @Column(length = 10)
    private String piso;
    
    @Column(length = 10)
    private String codigoPostal;
    
    @Column(length = 100)
    private String ciudad;
    
    @Column(length = 100)
    private String provincia;
    
    @Column(length = 100)
    private String pais;
    
    // Constructor vacío requerido
    public Direccion() {}
    
    // Constructor completo
    public Direccion(String calle, String numero, String codigoPostal, 
                     String ciudad, String provincia, String pais) {
        this.calle = calle;
        this.numero = numero;
        this.codigoPostal = codigoPostal;
        this.ciudad = ciudad;
        this.provincia = provincia;
        this.pais = pais;
    }
    
    // Getters y setters
}

@Entity
@Table(name = "clientes")
public class Cliente {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private String email;
    
    @Embedded
    private Direccion direccion;
    
    // Si el embebido puede ser null
    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "calle", column = @Column(name = "facturacion_calle")),
        @AttributeOverride(name = "numero", column = @Column(name = "facturacion_numero")),
        @AttributeOverride(name = "piso", column = @Column(name = "facturacion_piso")),
        @AttributeOverride(name = "codigoPostal", column = @Column(name = "facturacion_cp")),
        @AttributeOverride(name = "ciudad", column = @Column(name = "facturacion_ciudad")),
        @AttributeOverride(name = "provincia", column = @Column(name = "facturacion_provincia")),
        @AttributeOverride(name = "pais", column = @Column(name = "facturacion_pais"))
    })
    private Direccion direccionFacturacion;
}

La tabla generada contiene todas las columnas:

CREATE TABLE clientes (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(255),
    email VARCHAR(255),
    -- Columnas de direccion
    calle VARCHAR(200),
    numero VARCHAR(10),
    piso VARCHAR(10),
    codigo_postal VARCHAR(10),
    ciudad VARCHAR(100),
    provincia VARCHAR(100),
    pais VARCHAR(100),
    -- Columnas de direccion_facturacion (con prefijo)
    facturacion_calle VARCHAR(200),
    facturacion_numero VARCHAR(10),
    facturacion_piso VARCHAR(10),
    facturacion_cp VARCHAR(10),
    facturacion_ciudad VARCHAR(100),
    facturacion_provincia VARCHAR(100),
    facturacion_pais VARCHAR(100)
);

### 4.2. Embebidos con relaciones

Los componentes embebidos pueden contener relaciones:

@Embeddable
public class InformacionContacto {
    
    @Column(length = 100)
    private String email;
    
    @Column(length = 20)
    private String telefonoPrincipal;
    
    @Column(length = 20)
    private String telefonoSecundario;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "preferencia_contacto_id")
    private PreferenciaContacto preferenciaContacto;
}

@Entity
public class Empleado {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    @Embedded
    private InformacionContacto contacto;
}

### 4.3. Colecciones de embebidos

JPA permite tener colecciones de tipos embebidos:

@Embeddable
public class Telefono {
    
    @Column(length = 20)
    private String numero;
    
    @Enumerated(EnumType.STRING)
    @Column(length = 20)
    private TipoTelefono tipo;
    
    private Boolean principal;
    
    public enum TipoTelefono {
        MOVIL, FIJO, TRABAJO, FAX
    }
}

@Entity
@Table(name = "contactos")
public class Contacto {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    
    @ElementCollection
    @CollectionTable(
        name = "contacto_telefonos",
        joinColumns = @JoinColumn(name = "contacto_id")
    )
    private List<Telefono> telefonos = new ArrayList<>();
    
    // Método de conveniencia
    public void addTelefono(String numero, Telefono.TipoTelefono tipo, boolean principal) {
        Telefono tel = new Telefono();
        tel.setNumero(numero);
        tel.setTipo(tipo);
        tel.setPrincipal(principal);
        telefonos.add(tel);
    }
}

Tabla generada para la colección:

CREATE TABLE contacto_telefonos (
    contacto_id BIGINT NOT NULL,
    numero VARCHAR(20),
    tipo VARCHAR(20),
    principal BOOLEAN,
    FOREIGN KEY (contacto_id) REFERENCES contactos(id)
);

## 5. Ejercicio práctico guiado: Sistema de Gestión Académica

En este ejercicio implementaremos un sistema completo que demuestra todos los conceptos de mapeo avanzado.

### 5.1. Descripción del sistema

Desarrollaremos un sistema para gestionar una academia que imparte cursos online con las siguientes características:

- Diferentes tipos de usuarios (Estudiantes, Profesores, Administradores)
    
- Cursos con módulos y lecciones
    
- Inscripciones con seguimiento de progreso
    
- Materiales multimedia (documentos, videos)
    
- Sistema de evaluaciones y certificados
    

### 5.2. Diagrama del modelo

                    +-------------------+
                    |  EntidadBase      |  @MappedSuperclass
                    |-------------------|
                    | id                |
                    | fechaCreacion     |
                    | fechaModificacion |
                    +-------------------+
                            △
                            |
        +-------------------+-------------------+
        |                   |                   |
+-------+-------+   +-------+-------+   +-------+-------+
|   Usuario     |   |    Curso      |   |  Material     |
|---------------|   |---------------|   |---------------|
| email         |   | titulo        |   | titulo        |
| password      |   | descripcion   |   | tipo          |
| nombreCompleto|   | nivel         |   | url/contenido |
| datosContacto |   | precio        |   +---------------+
+---------------+   | duracionHoras |           △
        △           | activo        |           |
        |           +---------------+   +-------+-------+
+-------+-------+           |           |               |
|       |       |           |       +---+---+       +---+---+
|   +---+---+   |   +-------+       |Documento|     |Video  |
|   |Profesor|  |   |               +---------+     +-------+
|   +--------+  |   |
|   |Estudiante |   +-------+
|   +-----------+           |
|                   +-------+-------+
+-------------------+  Inscripcion  |
                    |---------------|
                    | fechaInscripcion|
                    | progreso      |
                    | estado        |
                    | calificacion  |
                    +---------------+

### 5.3. Implementación paso a paso

#### Paso 1: Clase base con auditoría

package com.academia.modelo;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@MappedSuperclass
public abstract class EntidadBase {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "fecha_creacion", updatable = false)
    private LocalDateTime fechaCreacion;
    
    @Column(name = "fecha_modificacion")
    private LocalDateTime fechaModificacion;
    
    @Version
    private Long version;
    
    @PrePersist
    protected void onCreate() {
        fechaCreacion = LocalDateTime.now();
        fechaModificacion = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        fechaModificacion = LocalDateTime.now();
    }
    
    // Getters
    public Long getId() { return id; }
    public LocalDateTime getFechaCreacion() { return fechaCreacion; }
    public LocalDateTime getFechaModificacion() { return fechaModificacion; }
    public Long getVersion() { return version; }
    
    // Solo setter para version (útil en tests)
    protected void setId(Long id) { this.id = id; }
}

#### Paso 2: Componente embebido para datos de contacto

package com.academia.modelo;

import jakarta.persistence.*;

@Embeddable
public class DatosContacto {
    
    @Column(length = 20)
    private String telefono;
    
    @Column(length = 200)
    private String direccion;
    
    @Column(length = 100)
    private String ciudad;
    
    @Column(length = 100)
    private String pais;
    
    @Column(name = "codigo_postal", length = 10)
    private String codigoPostal;
    
    // Constructor vacío
    public DatosContacto() {}
    
    // Constructor con parámetros
    public DatosContacto(String telefono, String ciudad, String pais) {
        this.telefono = telefono;
        this.ciudad = ciudad;
        this.pais = pais;
    }
    
    // Getters y Setters
    public String getTelefono() { return telefono; }
    public void setTelefono(String telefono) { this.telefono = telefono; }
    
    public String getDireccion() { return direccion; }
    public void setDireccion(String direccion) { this.direccion = direccion; }
    
    public String getCiudad() { return ciudad; }
    public void setCiudad(String ciudad) { this.ciudad = ciudad; }
    
    public String getPais() { return pais; }
    public void setPais(String pais) { this.pais = pais; }
    
    public String getCodigoPostal() { return codigoPostal; }
    public void setCodigoPostal(String codigoPostal) { this.codigoPostal = codigoPostal; }
    
    @Override
    public String toString() {
        return String.format("%s, %s %s, %s", 
            direccion != null ? direccion : "", 
            codigoPostal != null ? codigoPostal : "",
            ciudad != null ? ciudad : "", 
            pais != null ? pais : "");
    }
}

#### Paso 3: Jerarquía de usuarios con JOINED

package com.academia.modelo;

import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "usuarios")
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Usuario extends EntidadBase {
    
    @Column(nullable = false, unique = true, length = 100)
    private String email;
    
    @Column(nullable = false, length = 255)
    private String password;
    
    @Column(name = "nombre_completo", nullable = false, length = 150)
    private String nombreCompleto;
    
    @Column(nullable = false)
    private Boolean activo = true;
    
    @Embedded
    private DatosContacto datosContacto;
    
    @ElementCollection
    @CollectionTable(
        name = "usuario_roles",
        joinColumns = @JoinColumn(name = "usuario_id")
    )
    @Enumerated(EnumType.STRING)
    @Column(name = "rol")
    private Set<Rol> roles = new HashSet<>();
    
    public enum Rol {
        ADMIN, PROFESOR, ESTUDIANTE
    }
    
    // Getters y Setters
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
    
    public String getNombreCompleto() { return nombreCompleto; }
    public void setNombreCompleto(String nombreCompleto) { this.nombreCompleto = nombreCompleto; }
    
    public Boolean getActivo() { return activo; }
    public void setActivo(Boolean activo) { this.activo = activo; }
    
    public DatosContacto getDatosContacto() { return datosContacto; }
    public void setDatosContacto(DatosContacto datosContacto) { this.datosContacto = datosContacto; }
    
    public Set<Rol> getRoles() { return roles; }
    public void setRoles(Set<Rol> roles) { this.roles = roles; }
    
    public void addRol(Rol rol) {
        this.roles.add(rol);
    }
    
    // Método abstracto para obtener el tipo de usuario
    public abstract String getTipoUsuario();
}

package com.academia.modelo;

import jakarta.persistence.*;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

@Entity
@Table(name = "profesores")
@PrimaryKeyJoinColumn(name = "usuario_id")
public class Profesor extends Usuario {
    
    @Column(length = 500)
    private String biografia;
    
    @Column(length = 200)
    private String especialidad;
    
    @Column(name = "fecha_incorporacion")
    private LocalDate fechaIncorporacion;
    
    @Column(name = "linkedin_url", length = 200)
    private String linkedinUrl;
    
    @OneToMany(mappedBy = "profesor", cascade = CascadeType.ALL)
    private List<Curso> cursosImpartidos = new ArrayList<>();
    
    @ElementCollection
    @CollectionTable(
        name = "profesor_certificaciones",
        joinColumns = @JoinColumn(name = "profesor_id")
    )
    @Column(name = "certificacion", length = 200)
    private Set<String> certificaciones = new HashSet<>();
    
    public Profesor() {
        addRol(Rol.PROFESOR);
    }
    
    @Override
    public String getTipoUsuario() {
        return "PROFESOR";
    }
    
    // Método de conveniencia
    public void addCurso(Curso curso) {
        cursosImpartidos.add(curso);
        curso.setProfesor(this);
    }
    
    // Getters y Setters
    public String getBiografia() { return biografia; }
    public void setBiografia(String biografia) { this.biografia = biografia; }
    
    public String getEspecialidad() { return especialidad; }
    public void setEspecialidad(String especialidad) { this.especialidad = especialidad; }
    
    public LocalDate getFechaIncorporacion() { return fechaIncorporacion; }
    public void setFechaIncorporacion(LocalDate fechaIncorporacion) { 
        this.fechaIncorporacion = fechaIncorporacion; 
    }
    
    public String getLinkedinUrl() { return linkedinUrl; }
    public void setLinkedinUrl(String linkedinUrl) { this.linkedinUrl = linkedinUrl; }
    
    public List<Curso> getCursosImpartidos() { return cursosImpartidos; }
    
    public Set<String> getCertificaciones() { return certificaciones; }
    
    public void addCertificacion(String certificacion) {
        this.certificaciones.add(certificacion);
    }
}

package com.academia.modelo;

import jakarta.persistence.*;
import java.time.LocalDate;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "estudiantes")
@PrimaryKeyJoinColumn(name = "usuario_id")
public class Estudiante extends Usuario {
    
    @Column(name = "numero_matricula", unique = true, length = 20)
    private String numeroMatricula;
    
    @Column(name = "fecha_nacimiento")
    private LocalDate fechaNacimiento;
    
    @Column(name = "nivel_estudios", length = 100)
    private String nivelEstudios;
    
    @OneToMany(mappedBy = "estudiante", cascade = CascadeType.ALL, orphanRemoval = true)
    private Set<Inscripcion> inscripciones = new HashSet<>();
    
    public Estudiante() {
        addRol(Rol.ESTUDIANTE);
    }
    
    @Override
    public String getTipoUsuario() {
        return "ESTUDIANTE";
    }
    
    // Método de conveniencia para inscribirse
    public Inscripcion inscribirseEn(Curso curso) {
        Inscripcion inscripcion = new Inscripcion(this, curso);
        this.inscripciones.add(inscripcion);
        curso.getInscripciones().add(inscripcion);
        return inscripcion;
    }
    
    // Getters y Setters
    public String getNumeroMatricula() { return numeroMatricula; }
    public void setNumeroMatricula(String numeroMatricula) { 
        this.numeroMatricula = numeroMatricula; 
    }
    
    public LocalDate getFechaNacimiento() { return fechaNacimiento; }
    public void setFechaNacimiento(LocalDate fechaNacimiento) { 
        this.fechaNacimiento = fechaNacimiento; 
    }
    
    public String getNivelEstudios() { return nivelEstudios; }
    public void setNivelEstudios(String nivelEstudios) { 
        this.nivelEstudios = nivelEstudios; 
    }
    
    public Set<Inscripcion> getInscripciones() { return inscripciones; }
    
    public int getCursosActivos() {
        return (int) inscripciones.stream()
            .filter(i -> i.getEstado() == Inscripcion.EstadoInscripcion.ACTIVA)
            .count();
    }
}

#### Paso 4: Entidad Curso con relaciones

package com.academia.modelo;

import jakarta.persistence.*;
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

@Entity
@Table(name = "cursos")
public class Curso extends EntidadBase {
    
    @Column(nullable = false, length = 200)
    private String titulo;
    
    @Lob
    @Column(columnDefinition = "TEXT")
    private String descripcion;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private NivelCurso nivel;
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal precio;
    
    @Column(name = "duracion_horas")
    private Integer duracionHoras;
    
    @Column(nullable = false)
    private Boolean activo = true;
    
    @Column(name = "imagen_url", length = 500)
    private String imagenUrl;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "profesor_id")
    private Profesor profesor;
    
    @OneToMany(mappedBy = "curso", cascade = CascadeType.ALL, orphanRemoval = true)
    @OrderBy("orden ASC")
    private List<Modulo> modulos = new ArrayList<>();
    
    @OneToMany(mappedBy = "curso", cascade = CascadeType.ALL)
    private Set<Inscripcion> inscripciones = new HashSet<>();
    
    @ManyToMany
    @JoinTable(
        name = "curso_etiquetas",
        joinColumns = @JoinColumn(name = "curso_id"),
        inverseJoinColumns = @JoinColumn(name = "etiqueta_id")
    )
    private Set<Etiqueta> etiquetas = new HashSet<>();
    
    public enum NivelCurso {
        PRINCIPIANTE, INTERMEDIO, AVANZADO, EXPERTO
    }
    
    // Métodos de conveniencia
    public void addModulo(Modulo modulo) {
        modulo.setOrden(modulos.size() + 1);
        modulos.add(modulo);
        modulo.setCurso(this);
    }
    
    public void removeModulo(Modulo modulo) {
        modulos.remove(modulo);
        modulo.setCurso(null);
        // Reordenar
        for (int i = 0; i < modulos.size(); i++) {
            modulos.get(i).setOrden(i + 1);
        }
    }
    
    public void addEtiqueta(Etiqueta etiqueta) {
        etiquetas.add(etiqueta);
        etiqueta.getCursos().add(this);
    }
    
    public int getNumeroInscritos() {
        return (int) inscripciones.stream()
            .filter(i -> i.getEstado() == Inscripcion.EstadoInscripcion.ACTIVA)
            .count();
    }
    
    public int getTotalLecciones() {
        return modulos.stream()
            .mapToInt(m -> m.getLecciones().size())
            .sum();
    }
    
    // Getters y Setters
    public String getTitulo() { return titulo; }
    public void setTitulo(String titulo) { this.titulo = titulo; }
    
    public String getDescripcion() { return descripcion; }
    public void setDescripcion(String descripcion) { this.descripcion = descripcion; }
    
    public NivelCurso getNivel() { return nivel; }
    public void setNivel(NivelCurso nivel) { this.nivel = nivel; }
    
    public BigDecimal getPrecio() { return precio; }
    public void setPrecio(BigDecimal precio) { this.precio = precio; }
    
    public Integer getDuracionHoras() { return duracionHoras; }
    public void setDuracionHoras(Integer duracionHoras) { this.duracionHoras = duracionHoras; }
    
    public Boolean getActivo() { return activo; }
    public void setActivo(Boolean activo) { this.activo = activo; }
    
    public String getImagenUrl() { return imagenUrl; }
    public void setImagenUrl(String imagenUrl) { this.imagenUrl = imagenUrl; }
    
    public Profesor getProfesor() { return profesor; }
    public void setProfesor(Profesor profesor) { this.profesor = profesor; }
    
    public List<Modulo> getModulos() { return modulos; }
    public Set<Inscripcion> getInscripciones() { return inscripciones; }
    public Set<Etiqueta> getEtiquetas() { return etiquetas; }
}

#### Paso 5: Módulos y Lecciones

package com.academia.modelo;

import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "modulos")
public class Modulo extends EntidadBase {
    
    @Column(nullable = false, length = 200)
    private String titulo;
    
    @Column(length = 1000)
    private String descripcion;
    
    @Column(nullable = false)
    private Integer orden;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "curso_id", nullable = false)
    private Curso curso;
    
    @OneToMany(mappedBy = "modulo", cascade = CascadeType.ALL, orphanRemoval = true)
    @OrderBy("orden ASC")
    private List<Leccion> lecciones = new ArrayList<>();
    
    // Métodos de conveniencia
    public void addLeccion(Leccion leccion) {
        leccion.setOrden(lecciones.size() + 1);
        lecciones.add(leccion);
        leccion.setModulo(this);
    }
    
    public int getDuracionTotalMinutos() {
        return lecciones.stream()
            .mapToInt(Leccion::getDuracionMinutos)
            .sum();
    }
    
    // Getters y Setters
    public String getTitulo() { return titulo; }
    public void setTitulo(String titulo) { this.titulo = titulo; }
    
    public String getDescripcion() { return descripcion; }
    public void setDescripcion(String descripcion) { this.descripcion = descripcion; }
    
    public Integer getOrden() { return orden; }
    public void setOrden(Integer orden) { this.orden = orden; }
    
    public Curso getCurso() { return curso; }
    public void setCurso(Curso curso) { this.curso = curso; }
    
    public List<Leccion> getLecciones() { return lecciones; }
}

package com.academia.modelo;

import jakarta.persistence.*;

@Entity
@Table(name = "lecciones")
public class Leccion extends EntidadBase {
    
    @Column(nullable = false, length = 200)
    private String titulo;
    
    @Lob
    @Column(columnDefinition = "TEXT")
    private String contenido;
    
    @Column(name = "duracion_minutos", nullable = false)
    private Integer duracionMinutos;
    
    @Column(nullable = false)
    private Integer orden;
    
    @Column(name = "video_url", length = 500)
    private String videoUrl;
    
    @Column(name = "es_gratuita")
    private Boolean esGratuita = false;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "modulo_id", nullable = false)
    private Modulo modulo;
    
    // Getters y Setters
    public String getTitulo() { return titulo; }
    public void setTitulo(String titulo) { this.titulo = titulo; }
    
    public String getContenido() { return contenido; }
    public void setContenido(String contenido) { this.contenido = contenido; }
    
    public Integer getDuracionMinutos() { return duracionMinutos; }
    public void setDuracionMinutos(Integer duracionMinutos) { 
        this.duracionMinutos = duracionMinutos; 
    }
    
    public Integer getOrden() { return orden; }
    public void setOrden(Integer orden) { this.orden = orden; }
    
    public String getVideoUrl() { return videoUrl; }
    public void setVideoUrl(String videoUrl) { this.videoUrl = videoUrl; }
    
    public Boolean getEsGratuita() { return esGratuita; }
    public void setEsGratuita(Boolean esGratuita) { this.esGratuita = esGratuita; }
    
    public Modulo getModulo() { return modulo; }
    public void setModulo(Modulo modulo) { this.modulo = modulo; }
}

#### Paso 6: Inscripción con clave compuesta

package com.academia.modelo;

import jakarta.persistence.*;
import java.io.Serializable;
import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.Objects;

@Entity
@Table(name = "inscripciones")
public class Inscripcion {
    
    @EmbeddedId
    private InscripcionId id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("estudianteId")
    @JoinColumn(name = "estudiante_id")
    private Estudiante estudiante;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("cursoId")
    @JoinColumn(name = "curso_id")
    private Curso curso;
    
    @Column(name = "fecha_inscripcion", nullable = false)
    private LocalDate fechaInscripcion;
    
    @Column(name = "fecha_finalizacion")
    private LocalDate fechaFinalizacion;
    
    @Column(nullable = false)
    private Integer progreso = 0;  // Porcentaje 0-100
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private EstadoInscripcion estado;
    
    @Column(precision = 4, scale = 2)
    private BigDecimal calificacion;
    
    @Column(name = "certificado_emitido")
    private Boolean certificadoEmitido = false;
    
    public enum EstadoInscripcion {
        ACTIVA, COMPLETADA, ABANDONADA, SUSPENDIDA
    }
    
    // Constructor vacío
    public Inscripcion() {}
    
    // Constructor con parámetros
    public Inscripcion(Estudiante estudiante, Curso curso) {
        this.estudiante = estudiante;
        this.curso = curso;
        this.id = new InscripcionId(estudiante.getId(), curso.getId());
        this.fechaInscripcion = LocalDate.now();
        this.estado = EstadoInscripcion.ACTIVA;
        this.progreso = 0;
    }
    
    // Métodos de negocio
    public void actualizarProgreso(int nuevoProgreso) {
        this.progreso = Math.min(100, Math.max(0, nuevoProgreso));
        if (this.progreso == 100 && this.estado == EstadoInscripcion.ACTIVA) {
            this.estado = EstadoInscripcion.COMPLETADA;
            this.fechaFinalizacion = LocalDate.now();
        }
    }
    
    public void asignarCalificacion(BigDecimal calificacion) {
        if (calificacion.compareTo(BigDecimal.ZERO) < 0 || 
            calificacion.compareTo(BigDecimal.TEN) > 0) {
            throw new IllegalArgumentException("La calificación debe estar entre 0 y 10");
        }
        this.calificacion = calificacion;
    }
    
    public boolean puedeEmitirCertificado() {
        return estado == EstadoInscripcion.COMPLETADA && 
               calificacion != null && 
               calificacion.compareTo(new BigDecimal("5")) >= 0 &&
               !certificadoEmitido;
    }
    
    // Getters y Setters
    public InscripcionId getId() { return id; }
    public Estudiante getEstudiante() { return estudiante; }
    public Curso getCurso() { return curso; }
    
    public LocalDate getFechaInscripcion() { return fechaInscripcion; }
    public void setFechaInscripcion(LocalDate fechaInscripcion) { 
        this.fechaInscripcion = fechaInscripcion; 
    }
    
    public LocalDate getFechaFinalizacion() { return fechaFinalizacion; }
    
    public Integer getProgreso() { return progreso; }
    
    public EstadoInscripcion getEstado() { return estado; }
    public void setEstado(EstadoInscripcion estado) { this.estado = estado; }
    
    public BigDecimal getCalificacion() { return calificacion; }
    
    public Boolean getCertificadoEmitido() { return certificadoEmitido; }
    public void setCertificadoEmitido(Boolean certificadoEmitido) { 
        this.certificadoEmitido = certificadoEmitido; 
    }
}

package com.academia.modelo;

import jakarta.persistence.*;
import java.io.Serializable;
import java.util.Objects;

@Embeddable
public class InscripcionId implements Serializable {
    
    @Column(name = "estudiante_id")
    private Long estudianteId;
    
    @Column(name = "curso_id")
    private Long cursoId;
    
    public InscripcionId() {}
    
    public InscripcionId(Long estudianteId, Long cursoId) {
        this.estudianteId = estudianteId;
        this.cursoId = cursoId;
    }
    
    public Long getEstudianteId() { return estudianteId; }
    public Long getCursoId() { return cursoId; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof InscripcionId)) return false;
        InscripcionId that = (InscripcionId) o;
        return Objects.equals(estudianteId, that.estudianteId) &&
               Objects.equals(cursoId, that.cursoId);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(estudianteId, cursoId);
    }
}

#### Paso 7: Etiquetas para categorización

package com.academia.modelo;

import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "etiquetas")
public class Etiqueta extends EntidadBase {
    
    @Column(nullable = false, unique = true, length = 50)
    private String nombre;
    
    @Column(length = 7)
    private String color;  // Formato hexadecimal, ej: #FF5733
    
    @ManyToMany(mappedBy = "etiquetas")
    private Set<Curso> cursos = new HashSet<>();
    
    // Getters y Setters
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    
    public String getColor() { return color; }
    public void setColor(String color) { this.color = color; }
    
    public Set<Curso> getCursos() { return cursos; }
    
    public int getNumeroCursos() {
        return cursos.size();
    }
}

#### Paso 8: Jerarquía de materiales con SINGLE_TABLE

package com.academia.modelo;

import jakarta.persistence.*;

@Entity
@Table(name = "materiales")
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tipo_material", discriminatorType = DiscriminatorType.STRING)
public abstract class Material extends EntidadBase {
    
    @Column(nullable = false, length = 200)
    private String titulo;
    
    @Column(length = 500)
    private String descripcion;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "leccion_id")
    private Leccion leccion;
    
    // Método abstracto
    public abstract String getTipoMaterial();
    
    // Getters y Setters
    public String getTitulo() { return titulo; }
    public void setTitulo(String titulo) { this.titulo = titulo; }
    
    public String getDescripcion() { return descripcion; }
    public void setDescripcion(String descripcion) { this.descripcion = descripcion; }
    
    public Leccion getLeccion() { return leccion; }
    public void setLeccion(Leccion leccion) { this.leccion = leccion; }
}

package com.academia.modelo;

import jakarta.persistence.*;

@Entity
@DiscriminatorValue("DOCUMENTO")
public class Documento extends Material {
    
    @Column(name = "url_documento", length = 500)
    private String urlDocumento;
    
    @Column(name = "formato", length = 20)
    private String formato;  // PDF, DOCX, etc.
    
    @Column(name = "tamano_bytes")
    private Long tamanoBytes;
    
    @Override
    public String getTipoMaterial() {
        return "DOCUMENTO";
    }
    
    // Getters y Setters
    public String getUrlDocumento() { return urlDocumento; }
    public void setUrlDocumento(String urlDocumento) { this.urlDocumento = urlDocumento; }
    
    public String getFormato() { return formato; }
    public void setFormato(String formato) { this.formato = formato; }
    
    public Long getTamanoBytes() { return tamanoBytes; }
    public void setTamanoBytes(Long tamanoBytes) { this.tamanoBytes = tamanoBytes; }
    
    public String getTamanoFormateado() {
        if (tamanoBytes == null) return "Desconocido";
        if (tamanoBytes < 1024) return tamanoBytes + " B";
        if (tamanoBytes < 1024 * 1024) return (tamanoBytes / 1024) + " KB";
        return (tamanoBytes / (1024 * 1024)) + " MB";
    }
}

package com.academia.modelo;

import jakarta.persistence.*;

@Entity
@DiscriminatorValue("VIDEO")
public class Video extends Material {
    
    @Column(name = "url_video", length = 500)
    private String urlVideo;
    
    @Column(name = "duracion_segundos")
    private Integer duracionSegundos;
    
    @Column(name = "resolucion", length = 20)
    private String resolucion;  // 720p, 1080p, etc.
    
    @Column(name = "plataforma", length = 50)
    private String plataforma;  // YouTube, Vimeo, propio
    
    @Override
    public String getTipoMaterial() {
        return "VIDEO";
    }
    
    // Getters y Setters
    public String getUrlVideo() { return urlVideo; }
    public void setUrlVideo(String urlVideo) { this.urlVideo = urlVideo; }
    
    public Integer getDuracionSegundos() { return duracionSegundos; }
    public void setDuracionSegundos(Integer duracionSegundos) { 
        this.duracionSegundos = duracionSegundos; 
    }
    
    public String getResolucion() { return resolucion; }
    public void setResolucion(String resolucion) { this.resolucion = resolucion; }
    
    public String getPlataforma() { return plataforma; }
    public void setPlataforma(String plataforma) { this.plataforma = plataforma; }
    
    public String getDuracionFormateada() {
        if (duracionSegundos == null) return "00:00";
        int minutos = duracionSegundos / 60;
        int segundos = duracionSegundos % 60;
        return String.format("%02d:%02d", minutos, segundos);
    }
}

### 5.4. Configuración de Hibernate

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- Configuración de conexión -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/academia_db?useSSL=false&amp;serverTimezone=UTC&amp;allowPublicKeyRetrieval=true</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">root</property>
        
        <!-- Pool de conexiones -->
        <property name="hibernate.hikari.minimumIdle">5</property>
        <property name="hibernate.hikari.maximumPoolSize">20</property>
        <property name="hibernate.hikari.idleTimeout">30000</property>
        
        <!-- Dialecto -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
        
        <!-- Configuración DDL -->
        <property name="hibernate.hbm2ddl.auto">update</property>
        
        <!-- Mostrar SQL -->
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        <property name="hibernate.highlight_sql">true</property>
        
        <!-- Estadísticas -->
        <property name="hibernate.generate_statistics">true</property>
        
        <!-- Mapeo de entidades -->
        <mapping class="com.academia.modelo.Usuario"/>
        <mapping class="com.academia.modelo.Profesor"/>
        <mapping class="com.academia.modelo.Estudiante"/>
        <mapping class="com.academia.modelo.Curso"/>
        <mapping class="com.academia.modelo.Modulo"/>
        <mapping class="com.academia.modelo.Leccion"/>
        <mapping class="com.academia.modelo.Inscripcion"/>
        <mapping class="com.academia.modelo.Etiqueta"/>
        <mapping class="com.academia.modelo.Material"/>
        <mapping class="com.academia.modelo.Documento"/>
        <mapping class="com.academia.modelo.Video"/>
    </session-factory>
</hibernate-configuration>

### 5.5. Clase utilitaria HibernateUtil

package com.academia.util;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {
    
    private static final SessionFactory sessionFactory;
    
    static {
        try {
            sessionFactory = new Configuration()
                .configure("hibernate.cfg.xml")
                .buildSessionFactory();
        } catch (Throwable ex) {
            System.err.println("Error al crear SessionFactory: " + ex);
            throw new ExceptionInInitializerError(ex);
        }
    }
    
    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
    
    public static void shutdown() {
        if (sessionFactory != null) {
            sessionFactory.close();
        }
    }
}

### 5.6. Aplicación de demostración

package com.academia;

import com.academia.modelo.*;
import com.academia.util.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.Transaction;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.List;

public class AcademiaApp {
    
    public static void main(String[] args) {
        System.out.println("=== Sistema de Gestión Académica ===\n");
        
        try {
            // 1. Crear datos de prueba
            crearDatosDePrueba();
            
            // 2. Demostrar consultas polimórficas
            demostrarConsultasPolimorficas();
            
            // 3. Demostrar relaciones y navegación
            demostrarRelaciones();
            
            // 4. Demostrar herencia de materiales
            demostrarHerenciaMateriales();
            
        } finally {
            HibernateUtil.shutdown();
        }
    }
    
    private static void crearDatosDePrueba() {
        System.out.println("--- Creando datos de prueba ---\n");
        
        try (Session session = HibernateUtil.getSessionFactory().openSession()) {
            Transaction tx = session.beginTransaction();
            
            // Crear profesor
            Profesor profesor = new Profesor();
            profesor.setEmail("maria.garcia@academia.com");
            profesor.setPassword("profesor123");
            profesor.setNombreCompleto("María García López");
            profesor.setBiografia("Experta en desarrollo de software con 15 años de experiencia");
            profesor.setEspecialidad("Desarrollo Java y Spring");
            profesor.setFechaIncorporacion(LocalDate.of(2020, 3, 15));
            profesor.setDatosContacto(new DatosContacto("612345678", "Madrid", "España"));
            profesor.addCertificacion("Oracle Certified Professional Java SE 17");
            profesor.addCertificacion("Spring Professional Certification");
            
            // Crear curso
            Curso curso = new Curso();
            curso.setTitulo("Java Avanzado con Spring Boot");
            curso.setDescripcion("Curso completo de desarrollo empresarial con Java y Spring Boot");
            curso.setNivel(Curso.NivelCurso.AVANZADO);
            curso.setPrecio(new BigDecimal("299.99"));
            curso.setDuracionHoras(60);
            profesor.addCurso(curso);
            
            // Crear módulos
            Modulo modulo1 = new Modulo();
            modulo1.setTitulo("Fundamentos de Spring Boot");
            modulo1.setDescripcion("Introducción al framework Spring Boot");
            curso.addModulo(modulo1);
            
            Leccion leccion1 = new Leccion();
            leccion1.setTitulo("Configuración del proyecto");
            leccion1.setContenido("En esta lección aprenderemos a configurar un proyecto Spring Boot...");
            leccion1.setDuracionMinutos(45);
            leccion1.setVideoUrl("https://videos.academia.com/spring-intro");
            leccion1.setEsGratuita(true);
            modulo1.addLeccion(leccion1);
            
            Leccion leccion2 = new Leccion();
            leccion2.setTitulo("Inyección de dependencias");
            leccion2.setContenido("La inyección de dependencias es un patrón fundamental...");
            leccion2.setDuracionMinutos(60);
            modulo1.addLeccion(leccion2);
            
            Modulo modulo2 = new Modulo();
            modulo2.setTitulo("Spring Data JPA");
            modulo2.setDescripcion("Acceso a datos con Spring Data JPA");
            curso.addModulo(modulo2);
            
            // Crear etiquetas
            Etiqueta etJava = new Etiqueta();
            etJava.setNombre("Java");
            etJava.setColor("#FF5733");
            
            Etiqueta etSpring = new Etiqueta();
            etSpring.setNombre("Spring");
            etSpring.setColor("#6DB33F");
            
            curso.addEtiqueta(etJava);
            curso.addEtiqueta(etSpring);
            
            // Crear estudiantes
            Estudiante estudiante1 = new Estudiante();
            estudiante1.setEmail("carlos.ruiz@email.com");
            estudiante1.setPassword("estudiante123");
            estudiante1.setNombreCompleto("Carlos Ruiz Martínez");
            estudiante1.setNumeroMatricula("EST-2024-001");
            estudiante1.setFechaNacimiento(LocalDate.of(1995, 7, 22));
            estudiante1.setNivelEstudios("Grado en Informática");
            estudiante1.setDatosContacto(new DatosContacto("622334455", "Barcelona", "España"));
            
            Estudiante estudiante2 = new Estudiante();
            estudiante2.setEmail("laura.sanchez@email.com");
            estudiante2.setPassword("estudiante456");
            estudiante2.setNombreCompleto("Laura Sánchez Torres");
            estudiante2.setNumeroMatricula("EST-2024-002");
            estudiante2.setFechaNacimiento(LocalDate.of(1998, 11, 5));
            
            // Persistir entidades
            session.persist(profesor);
            session.persist(etJava);
            session.persist(etSpring);
            session.persist(estudiante1);
            session.persist(estudiante2);
            
            // Crear inscripciones
            Inscripcion insc1 = estudiante1.inscribirseEn(curso);
            insc1.actualizarProgreso(35);
            
            Inscripcion insc2 = estudiante2.inscribirseEn(curso);
            insc2.actualizarProgreso(100);
            insc2.asignarCalificacion(new BigDecimal("8.5"));
            
            // Crear materiales
            Documento doc = new Documento();
            doc.setTitulo("Guía de referencia Spring Boot");
            doc.setDescripcion("PDF con todas las anotaciones y configuraciones");
            doc.setUrlDocumento("https://docs.academia.com/spring-guide.pdf");
            doc.setFormato("PDF");
            doc.setTamanoBytes(2500000L);
            doc.setLeccion(leccion1);
            session.persist(doc);
            
            Video video = new Video();
            video.setTitulo("Tutorial práctico de Spring Boot");
            video.setDescripcion("Video tutorial paso a paso");
            video.setUrlVideo("https://videos.academia.com/spring-tutorial");
            video.setDuracionSegundos(3600);
            video.setResolucion("1080p");
            video.setPlataforma("YouTube");
            video.setLeccion(leccion1);
            session.persist(video);
            
            tx.commit();
            System.out.println("Datos creados correctamente\n");
        }
    }
    
    private static void demostrarConsultasPolimorficas() {
        System.out.println("--- Consultas polimórficas ---\n");
        
        try (Session session = HibernateUtil.getSessionFactory().openSession()) {
            // Consulta que devuelve todos los tipos de usuarios
            List<Usuario> usuarios = session.createQuery(
                "FROM Usuario", Usuario.class).getResultList();
            
            System.out.println("Todos los usuarios del sistema:");
            for (Usuario u : usuarios) {
                System.out.printf("  - %s (%s) - Tipo: %s%n", 
                    u.getNombreCompleto(), u.getEmail(), u.getTipoUsuario());
            }
            
            // Consulta solo de profesores
            List<Profesor> profesores = session.createQuery(
                "FROM Profesor p JOIN FETCH p.cursosImpartidos", Profesor.class)
                .getResultList();
            
            System.out.println("\nProfesores y sus cursos:");
            for (Profesor p : profesores) {
                System.out.printf("  Profesor: %s - %s%n", 
                    p.getNombreCompleto(), p.getEspecialidad());
                p.getCursosImpartidos().forEach(c -> 
                    System.out.printf("    - %s (Nivel: %s)%n", c.getTitulo(), c.getNivel()));
            }
            System.out.println();
        }
    }
    
    private static void demostrarRelaciones() {
        System.out.println("--- Navegación de relaciones ---\n");
        
        try (Session session = HibernateUtil.getSessionFactory().openSession()) {
            // Cargar curso con todas sus relaciones
            Curso curso = session.createQuery(
                "SELECT c FROM Curso c " +
                "LEFT JOIN FETCH c.modulos m " +
                "LEFT JOIN FETCH m.lecciones " +
                "LEFT JOIN FETCH c.etiquetas " +
                "WHERE c.titulo LIKE :titulo", Curso.class)
                .setParameter("titulo", "%Spring%")
                .uniqueResult();
            
            if (curso != null) {
                System.out.printf("Curso: %s%n", curso.getTitulo());
                System.out.printf("Nivel: %s | Precio: %.2f€ | Duración: %d horas%n",
                    curso.getNivel(), curso.getPrecio(), curso.getDuracionHoras());
                System.out.printf("Etiquetas: %s%n", 
                    curso.getEtiquetas().stream()
                        .map(Etiqueta::getNombre)
                        .reduce((a, b) -> a + ", " + b)
                        .orElse("Ninguna"));
                
                System.out.println("\nEstructura del curso:");
                for (Modulo m : curso.getModulos()) {
                    System.out.printf("  Módulo %d: %s (%d min)%n", 
                        m.getOrden(), m.getTitulo(), m.getDuracionTotalMinutos());
                    for (Leccion l : m.getLecciones()) {
                        System.out.printf("    %d.%d %s - %d min%s%n",
                            m.getOrden(), l.getOrden(), l.getTitulo(), 
                            l.getDuracionMinutos(),
                            l.getEsGratuita() ? " (GRATIS)" : "");
                    }
                }
                
                System.out.printf("\nTotal lecciones: %d%n", curso.getTotalLecciones());
                System.out.printf("Inscritos activos: %d%n", curso.getNumeroInscritos());
            }
            System.out.println();
        }
    }
    
    private static void demostrarHerenciaMateriales() {
        System.out.println("--- Herencia de materiales (SINGLE_TABLE) ---\n");
        
        try (Session session = HibernateUtil.getSessionFactory().openSession()) {
            // Consulta polimórfica de materiales
            List<Material> materiales = session.createQuery(
                "FROM Material", Material.class).getResultList();
            
            System.out.println("Todos los materiales:");
            for (Material m : materiales) {
                System.out.printf("  [%s] %s%n", m.getTipoMaterial(), m.getTitulo());
                
                if (m instanceof Documento doc) {
                    System.out.printf("    Formato: %s | Tamaño: %s%n", 
                        doc.getFormato(), doc.getTamanoFormateado());
                } else if (m instanceof Video vid) {
                    System.out.printf("    Duración: %s | Resolución: %s | Plataforma: %s%n",
                        vid.getDuracionFormateada(), vid.getResolucion(), vid.getPlataforma());
                }
            }
            
            // Consulta solo de videos
            List<Video> videos = session.createQuery(
                "FROM Video WHERE resolucion = :res", Video.class)
                .setParameter("res", "1080p")
                .getResultList();
            
            System.out.printf("\nVideos en 1080p: %d%n", videos.size());
        }
    }
}

## 6. Ejercicios propuestos

### Ejercicio 1: Sistema de Evaluaciones

Amplía el sistema de gestión académica añadiendo un sistema de evaluaciones:

1. Crea una entidad abstracta `Evaluacion` con los campos: id, titulo, puntuacionMaxima, fechaCreacion
    
2. Implementa dos subclases usando herencia JOINED:
    

- `ExamenTest`: numeroPreguntas, tiempoLimiteMinutos
    
- `Proyecto`: fechaEntrega, descripcion, permiteTrabajosGrupo
    

1. Crea una entidad `RespuestaEvaluacion` que relacione Estudiante con Evaluacion, incluyendo: puntuacionObtenida, fechaRealizacion, comentarioProfesor
    
2. Implementa los DAOs correspondientes
    

### Ejercicio 2: Sistema de Comentarios y Valoraciones

Añade un sistema de feedback para los cursos:

1. Crea un componente embebido `Valoracion` con: puntuacion (1-5), comentario, fecha
    
2. Crea una entidad `ComentarioCurso` que contenga la valoración embebida y relacione Estudiante con Curso
    
3. Añade métodos en el DAO de Curso para:
    

- Obtener la puntuación media de un curso
    
- Listar los comentarios más recientes
    
- Buscar cursos con puntuación superior a un valor
    

### Ejercicio 3: Conversores y Tipos Especiales

Practica con conversores y tipos especiales:

1. Crea un conversor para almacenar Set (habilidades) como una cadena JSON
    
2. Implementa un campo `@Lob` en Estudiante para almacenar su foto de perfil
    
3. Crea una entidad `Notificacion` con:
    

- Campo `@Temporal` para fecha legacy
    
- Conversor personalizado para un enum EstadoNotificacion
    
- Campo `@Transient` para indicar si fue leída en la sesión actual
    

### Ejercicio 4: Optimización de Consultas

Mejora el rendimiento de las consultas:

1. Identifica y soluciona problemas N+1 en la carga de cursos con módulos y lecciones
    
2. Implementa `@BatchSize` en las colecciones apropiadas
    
3. Crea consultas con JOIN FETCH para los casos de uso principales
    
4. Añade `@EntityGraph` para cargar selectivamente las relaciones
    

### Ejercicio 5: Auditoría Completa

Implementa un sistema de auditoría:

1. Extiende EntidadBase para incluir campos de auditoría (creadoPor, modificadoPor)
    
2. Crea una entidad `HistorialCambios` que registre modificaciones
    
3. Usa `@PreUpdate` y `@PrePersist` para registrar automáticamente los cambios
    
4. Implementa un DAO para consultar el historial de cambios de cualquier entidad
    

## 7. Resumen y conceptos clave

En este bloque hemos aprendido:

### Anotaciones de mapeo

- Las estrategias de generación de ID (IDENTITY, SEQUENCE, TABLE, AUTO, UUID) y cuándo usar cada una
    
- Configuración detallada de columnas con `@Column` y sus atributos
    
- Mapeo de fechas con java.time (automático) y con `@Temporal` (legacy)
    
- Uso de `@Lob` para campos grandes y `@Transient` para campos no persistentes
    
- Creación de conversores personalizados con `@Convert`
    

### Relaciones entre entidades

- `@OneToOne` unidireccional y bidireccional, incluyendo clave compartida
    
- `@ManyToOne` y `@OneToMany` con mappedBy para evitar tablas de unión
    
- `@ManyToMany` simple y con atributos adicionales (entidad intermedia)
    
- Tipos de CascadeType y la diferencia con orphanRemoval
    
- FetchType LAZY vs EAGER y el problema N+1
    

### Herencia en JPA

- SINGLE_TABLE: una tabla para toda la jerarquía, con discriminador
    
- JOINED: una tabla por clase, con JOINs para consultas
    
- TABLE_PER_CLASS: tablas independientes completas
    
- `@MappedSuperclass`: herencia solo de atributos, sin tabla
    

### Componentes embebidos

- `@Embeddable` para definir tipos de valor reutilizables
    
- `@Embedded` para incluir componentes en entidades
    
- `@AttributeOverride` para renombrar columnas
    
- `@ElementCollection` para colecciones de embebidos
    

### Buenas prácticas

1. Usar siempre FetchType.LAZY por defecto
    
2. Implementar métodos de conveniencia para sincronizar relaciones bidireccionales
    
3. Usar `@Version` para control de concurrencia optimista
    
4. Preferir SEQUENCE sobre IDENTITY para permitir batch inserts
    
5. Usar claves de negocio en equals() y hashCode(), no el ID
    
6. Elegir la estrategia de herencia según el patrón de consultas
    
7. Utilizar componentes embebidos para objetos de valor