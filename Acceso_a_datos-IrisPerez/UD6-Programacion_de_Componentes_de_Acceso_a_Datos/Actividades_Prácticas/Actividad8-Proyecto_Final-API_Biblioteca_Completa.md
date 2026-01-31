## Objetivos

- Integrar todos los conceptos del RA6
    
- Desarrollar una API REST completa y funcional
    
- Aplicar buenas practicas de desarrollo
    
- Documentar y probar la aplicacion
    

## Descripcion del Proyecto

Desarrollar una API REST para la gestion de una biblioteca digital que incluya:

### Entidades

- **Libro**: id, titulo, autor, isbn, anioPublicacion, genero, disponible
    
- **Usuario**: id, username, email, password, roles, fechaRegistro
    
- **Prestamo**: id, libro, usuario, fechaPrestamo, fechaDevolucion, estado
    

### Funcionalidades Requeridas

1. **CRUD completo** para Libros y Usuarios
    
2. **Gestion de prestamos**: solicitar, devolver, listar historial
    
3. **Seguridad**: autenticacion basica, roles USER/ADMIN
    
4. **Busquedas avanzadas**: por titulo, autor, genero, disponibilidad
    
5. **Estadisticas**: libros mas prestados, usuarios activos
    
6. **Documentacion**: OpenAPI/Swagger
    
7. **Tests**: unitarios y de integracion
    

## Ejercicio 1: Estructura del Proyecto

### Enunciado

Crea la estructura base del proyecto con:

1. pom.xml con todas las dependencias
    
2. Estructura de paquetes
    
3. application.properties configurado
    
4. Clase principal de Spring Boot
    

Ver solucion

**Estructura de paquetes:**

src/main/java/com/biblioteca/
├── BibliotecaApplication.java
├── config/
│   ├── SecurityConfig.java
│   └── OpenApiConfig.java
├── model/
│   ├── Libro.java
│   ├── Usuario.java
│   ├── Prestamo.java
│   ├── EstadoPrestamo.java
│   └── Rol.java
├── repository/
│   ├── LibroRepository.java
│   ├── UsuarioRepository.java
│   └── PrestamoRepository.java
├── service/
│   ├── LibroService.java
│   ├── UsuarioService.java
│   └── PrestamoService.java
├── controller/
│   ├── LibroController.java
│   ├── UsuarioController.java
│   ├── PrestamoController.java
│   └── EstadisticasController.java
├── dto/
│   ├── LibroDTO.java
│   ├── UsuarioDTO.java
│   ├── PrestamoDTO.java
│   └── EstadisticasDTO.java
├── mapper/
│   ├── LibroMapper.java
│   ├── UsuarioMapper.java
│   └── PrestamoMapper.java
└── exception/
    ├── GlobalExceptionHandler.java
    ├── ResourceNotFoundException.java
    └── BusinessException.java

**pom.xml:**

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>
    
    <groupId>com.biblioteca</groupId>
    <artifactId>biblioteca-api</artifactId>
    <version>1.0.0</version>
    <name>Biblioteca API</name>
    <description>API REST para gestion de biblioteca</description>
    
    <properties>
        <java.version>17</java.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Base de datos -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Documentacion API -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.3.0</version>
        </dependency>
        
        <!-- Utilidades -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

**application.properties:**

# Aplicacion
spring.application.name=Biblioteca API
server.port=8080

# Perfil activo (dev usa H2, prod usa PostgreSQL)
spring.profiles.active=dev

# H2 (desarrollo)
spring.datasource.url=jdbc:h2:mem:bibliotecadb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# JPA
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# OpenAPI
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html

# Logging
logging.level.com.biblioteca=DEBUG
logging.level.org.springframework.security=DEBUG

## Ejercicio 2: Modelo de Datos

### Enunciado

Implementa las entidades JPA con sus relaciones:

1. Libro (entidad independiente)
    
2. Usuario (con roles)
    
3. Prestamo (relacion ManyToOne con Libro y Usuario)
    

Ver solucion

**Libro.java:**

@Entity
@Table(name = "libros")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Libro {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 200)
    private String titulo;
    
    @Column(length = 100)
    private String autor;
    
    @Column(unique = true, length = 13)
    private String isbn;
    
    private Integer anioPublicacion;
    
    @Column(length = 50)
    private String genero;
    
    @Builder.Default
    private boolean disponible = true;
    
    @OneToMany(mappedBy = "libro")
    @JsonIgnore
    private List<Prestamo> prestamos = new ArrayList<>();
}

**Usuario.java:**

@Entity
@Table(name = "usuarios")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Usuario implements UserDetails {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false, length = 50)
    private String username;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    @JsonIgnore
    private String password;
    
    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "usuario_roles", 
        joinColumns = @JoinColumn(name = "usuario_id"))
    @Enumerated(EnumType.STRING)
    @Builder.Default
    private Set<Rol> roles = new HashSet<>();
    
    @Builder.Default
    private LocalDateTime fechaRegistro = LocalDateTime.now();
    
    @Builder.Default
    private boolean activo = true;
    
    @OneToMany(mappedBy = "usuario")
    @JsonIgnore
    private List<Prestamo> prestamos = new ArrayList<>();
    
    // UserDetails implementation
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return roles.stream()
            .map(rol -> new SimpleGrantedAuthority("ROLE_" + rol.name()))
            .collect(Collectors.toList());
    }
    
    @Override
    public boolean isAccountNonExpired() { return true; }
    
    @Override
    public boolean isAccountNonLocked() { return true; }
    
    @Override
    public boolean isCredentialsNonExpired() { return true; }
    
    @Override
    public boolean isEnabled() { return activo; }
}

public enum Rol {
    USER, ADMIN
}

**Prestamo.java:**

@Entity
@Table(name = "prestamos")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Prestamo {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "libro_id", nullable = false)
    private Libro libro;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "usuario_id", nullable = false)
    private Usuario usuario;
    
    @Builder.Default
    private LocalDateTime fechaPrestamo = LocalDateTime.now();
    
    private LocalDateTime fechaDevolucion;
    
    @Builder.Default
    private LocalDate fechaLimite = LocalDate.now().plusDays(14);
    
    @Enumerated(EnumType.STRING)
    @Builder.Default
    private EstadoPrestamo estado = EstadoPrestamo.ACTIVO;
}

public enum EstadoPrestamo {
    ACTIVO,
    DEVUELTO,
    VENCIDO
}

## Ejercicio 3: Servicios de Negocio

### Enunciado

Implementa los servicios con la logica de negocio:

1. LibroService: CRUD + busquedas
    
2. PrestamoService: solicitar, devolver, verificar disponibilidad
    

Ver solucion

**LibroService.java:**

@Service
@Transactional
@RequiredArgsConstructor
public class LibroService {
    
    private final LibroRepository repository;
    
    @Transactional(readOnly = true)
    public List<Libro> listarTodos() {
        return repository.findAll();
    }
    
    @Transactional(readOnly = true)
    public Page<Libro> listarPaginado(Pageable pageable) {
        return repository.findAll(pageable);
    }
    
    @Transactional(readOnly = true)
    public Optional<Libro> buscarPorId(Long id) {
        return repository.findById(id);
    }
    
    @Transactional(readOnly = true)
    public List<Libro> buscarPorTitulo(String titulo) {
        return repository.findByTituloContainingIgnoreCase(titulo);
    }
    
    @Transactional(readOnly = true)
    public List<Libro> buscarPorAutor(String autor) {
        return repository.findByAutorContainingIgnoreCase(autor);
    }
    
    @Transactional(readOnly = true)
    public List<Libro> buscarDisponibles() {
        return repository.findByDisponibleTrue();
    }
    
    public Libro guardar(Libro libro) {
        if (libro.getIsbn() != null && repository.existsByIsbn(libro.getIsbn())) {
            throw new BusinessException("Ya existe un libro con ese ISBN");
        }
        return repository.save(libro);
    }
    
    public Libro actualizar(Long id, Libro datosNuevos) {
        return repository.findById(id)
            .map(libro -> {
                libro.setTitulo(datosNuevos.getTitulo());
                libro.setAutor(datosNuevos.getAutor());
                libro.setGenero(datosNuevos.getGenero());
                libro.setAnioPublicacion(datosNuevos.getAnioPublicacion());
                return repository.save(libro);
            })
            .orElseThrow(() -> new ResourceNotFoundException("Libro", id));
    }
    
    public void eliminar(Long id) {
        if (!repository.existsById(id)) {
            throw new ResourceNotFoundException("Libro", id);
        }
        repository.deleteById(id);
    }
}

**PrestamoService.java:**

@Service
@Transactional
@RequiredArgsConstructor
public class PrestamoService {
    
    private final PrestamoRepository prestamoRepository;
    private final LibroRepository libroRepository;
    private final UsuarioRepository usuarioRepository;
    
    public Prestamo solicitarPrestamo(Long libroId, String username) {
        Libro libro = libroRepository.findById(libroId)
            .orElseThrow(() -> new ResourceNotFoundException("Libro", libroId));
        
        if (!libro.isDisponible()) {
            throw new BusinessException("El libro no esta disponible");
        }
        
        Usuario usuario = usuarioRepository.findByUsername(username)
            .orElseThrow(() -> new ResourceNotFoundException("Usuario", username));
        
        // Verificar prestamos activos del usuario
        long prestamosActivos = prestamoRepository.countByUsuarioAndEstado(
            usuario, EstadoPrestamo.ACTIVO);
        if (prestamosActivos >= 3) {
            throw new BusinessException("Has alcanzado el limite de prestamos activos (3)");
        }
        
        // Crear prestamo
        Prestamo prestamo = Prestamo.builder()
            .libro(libro)
            .usuario(usuario)
            .build();
        
        // Marcar libro como no disponible
        libro.setDisponible(false);
        libroRepository.save(libro);
        
        return prestamoRepository.save(prestamo);
    }
    
    public Prestamo devolverPrestamo(Long prestamoId) {
        Prestamo prestamo = prestamoRepository.findById(prestamoId)
            .orElseThrow(() -> new ResourceNotFoundException("Prestamo", prestamoId));
        
        if (prestamo.getEstado() != EstadoPrestamo.ACTIVO) {
            throw new BusinessException("El prestamo ya fue devuelto");
        }
        
        prestamo.setFechaDevolucion(LocalDateTime.now());
        prestamo.setEstado(EstadoPrestamo.DEVUELTO);
        
        // Marcar libro como disponible
        Libro libro = prestamo.getLibro();
        libro.setDisponible(true);
        libroRepository.save(libro);
        
        return prestamoRepository.save(prestamo);
    }
    
    @Transactional(readOnly = true)
    public List<Prestamo> listarPrestamosUsuario(String username) {
        Usuario usuario = usuarioRepository.findByUsername(username)
            .orElseThrow(() -> new ResourceNotFoundException("Usuario", username));
        return prestamoRepository.findByUsuarioOrderByFechaPrestamoDesc(usuario);
    }
    
    @Transactional(readOnly = true)
    public List<Prestamo> listarPrestamosActivos() {
        return prestamoRepository.findByEstado(EstadoPrestamo.ACTIVO);
    }
    
    // Tarea programada para marcar vencidos
    @Scheduled(cron = "0 0 0 * * *") // Cada dia a medianoche
    public void marcarVencidos() {
        List<Prestamo> vencidos = prestamoRepository
            .findByEstadoAndFechaLimiteBefore(EstadoPrestamo.ACTIVO, LocalDate.now());
        
        vencidos.forEach(p -> p.setEstado(EstadoPrestamo.VENCIDO));
        prestamoRepository.saveAll(vencidos);
    }
}

## Ejercicio 4: Controllers REST

### Enunciado

Implementa los controllers con documentacion OpenAPI:

1. LibroController: CRUD + busquedas
    
2. PrestamoController: solicitar, devolver, historial
    

Ver solucion

**PrestamoController.java:**

@RestController
@RequestMapping("/api/prestamos")
@Tag(name = "Prestamos", description = "Gestion de prestamos de libros")
@RequiredArgsConstructor
public class PrestamoController {
    
    private final PrestamoService service;
    private final PrestamoMapper mapper;
    
    @Operation(
        summary = "Solicitar prestamo",
        description = "Solicita el prestamo de un libro para el usuario autenticado"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "201", description = "Prestamo creado"),
        @ApiResponse(responseCode = "400", description = "Libro no disponible o limite alcanzado"),
        @ApiResponse(responseCode = "404", description = "Libro no encontrado")
    })
    @PostMapping("/solicitar/{libroId}")
    @ResponseStatus(HttpStatus.CREATED)
    @PreAuthorize("hasRole('USER')")
    public PrestamoResponseDTO solicitar(
            @PathVariable Long libroId,
            @AuthenticationPrincipal UserDetails user) {
        Prestamo prestamo = service.solicitarPrestamo(libroId, user.getUsername());
        return mapper.toDTO(prestamo);
    }
    
    @Operation(summary = "Devolver libro")
    @PutMapping("/devolver/{prestamoId}")
    @PreAuthorize("hasRole('USER')")
    public PrestamoResponseDTO devolver(@PathVariable Long prestamoId) {
        Prestamo prestamo = service.devolverPrestamo(prestamoId);
        return mapper.toDTO(prestamo);
    }
    
    @Operation(summary = "Ver mis prestamos")
    @GetMapping("/mis-prestamos")
    @PreAuthorize("hasRole('USER')")
    public List<PrestamoResponseDTO> misPrestamos(
            @AuthenticationPrincipal UserDetails user) {
        return service.listarPrestamosUsuario(user.getUsername())
            .stream()
            .map(mapper::toDTO)
            .collect(Collectors.toList());
    }
    
    @Operation(summary = "Ver todos los prestamos activos (solo ADMIN)")
    @GetMapping("/activos")
    @PreAuthorize("hasRole('ADMIN')")
    public List<PrestamoResponseDTO> prestamosActivos() {
        return service.listarPrestamosActivos()
            .stream()
            .map(mapper::toDTO)
            .collect(Collectors.toList());
    }
}

**EstadisticasController.java:**

@RestController
@RequestMapping("/api/estadisticas")
@Tag(name = "Estadisticas", description = "Estadisticas de la biblioteca")
@RequiredArgsConstructor
public class EstadisticasController {
    
    private final LibroRepository libroRepository;
    private final PrestamoRepository prestamoRepository;
    private final UsuarioRepository usuarioRepository;
    
    @Operation(summary = "Obtener estadisticas generales")
    @GetMapping
    @PreAuthorize("hasRole('ADMIN')")
    public EstadisticasDTO obtenerEstadisticas() {
        return EstadisticasDTO.builder()
            .totalLibros(libroRepository.count())
            .librosDisponibles(libroRepository.countByDisponibleTrue())
            .totalUsuarios(usuarioRepository.count())
            .prestamosActivos(prestamoRepository.countByEstado(EstadoPrestamo.ACTIVO))
            .prestamosVencidos(prestamoRepository.countByEstado(EstadoPrestamo.VENCIDO))
            .librosMasPrestados(obtenerLibrosMasPrestados())
            .generadoEn(LocalDateTime.now())
            .build();
    }
    
    private List<LibroEstadisticaDTO> obtenerLibrosMasPrestados() {
        return prestamoRepository.findLibrosMasPrestados(PageRequest.of(0, 5))
            .stream()
            .map(arr -> new LibroEstadisticaDTO(
                (String) arr[0],  // titulo
                (Long) arr[1]     // total prestamos
            ))
            .collect(Collectors.toList());
    }
}

## Ejercicio 5: Configuracion de Seguridad

### Enunciado

Configura Spring Security con:

1. Autenticacion basica
    
2. Usuarios en BD (no en memoria)
    
3. Endpoints publicos y protegidos
    
4. CORS configurado
    

Ver solucion

**SecurityConfig.java:**

@Configuration
@EnableWebSecurity
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    
    private final UsuarioRepository usuarioRepository;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                // Publicos
                .requestMatchers(HttpMethod.GET, "/api/libros", "/api/libros/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/api-docs/**", "/v3/api-docs/**").permitAll()
                .requestMatchers("/h2-console/**").permitAll()
                // Admin
                .requestMatchers("/api/admin/**", "/api/estadisticas/**").hasRole("ADMIN")
                // Resto requiere autenticacion
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults())
            .headers(headers -> headers.frameOptions(f -> f.disable())); // Para H2
        
        return http.build();
    }
    
    @Bean
    public UserDetailsService userDetailsService() {
        return username -> usuarioRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("Usuario no encontrado: " + username));
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("http://localhost:3000", "http://localhost:4200"));
        config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        config.setAllowedHeaders(List.of("*"));
        config.setAllowCredentials(true);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }
}

**DataInitializer.java (datos iniciales):**

@Component
@RequiredArgsConstructor
public class DataInitializer implements CommandLineRunner {
    
    private final UsuarioRepository usuarioRepository;
    private final LibroRepository libroRepository;
    private final PasswordEncoder passwordEncoder;
    
    @Override
    public void run(String... args) {
        // Crear usuarios
        if (usuarioRepository.count() == 0) {
            Usuario admin = Usuario.builder()
                .username("admin")
                .email("admin@biblioteca.com")
                .password(passwordEncoder.encode("admin123"))
                .roles(Set.of(Rol.USER, Rol.ADMIN))
                .build();
            
            Usuario user = Usuario.builder()
                .username("usuario")
                .email("usuario@biblioteca.com")
                .password(passwordEncoder.encode("user123"))
                .roles(Set.of(Rol.USER))
                .build();
            
            usuarioRepository.saveAll(List.of(admin, user));
        }
        
        // Crear libros de ejemplo
        if (libroRepository.count() == 0) {
            List<Libro> libros = List.of(
                Libro.builder().titulo("Don Quijote").autor("Cervantes")
                    .isbn("9788420412146").genero("Novela").anioPublicacion(1605).build(),
                Libro.builder().titulo("Cien anos de soledad").autor("Garcia Marquez")
                    .isbn("9788437604947").genero("Realismo Magico").anioPublicacion(1967).build(),
                Libro.builder().titulo("El Aleph").autor("Borges")
                    .isbn("9788499089515").genero("Cuento").anioPublicacion(1949).build()
            );
            libroRepository.saveAll(libros);
        }
    }
}

## Ejercicio 6: Tests Completos

### Enunciado

Escribe tests para verificar el funcionamiento:

1. Tests unitarios del servicio de prestamos
    
2. Tests de integracion del flujo completo
    
3. Tests de seguridad
    

Ver solucion

**PrestamoServiceTest.java:**

@ExtendWith(MockitoExtension.class)
class PrestamoServiceTest {
    
    @Mock private PrestamoRepository prestamoRepository;
    @Mock private LibroRepository libroRepository;
    @Mock private UsuarioRepository usuarioRepository;
    
    @InjectMocks private PrestamoService service;
    
    private Libro libro;
    private Usuario usuario;
    
    @BeforeEach
    void setUp() {
        libro = Libro.builder()
            .id(1L).titulo("Test").disponible(true).build();
        usuario = Usuario.builder()
            .id(1L).username("testuser").build();
    }
    
    @Test
    @DisplayName("Solicitar prestamo exitoso")
    void solicitarPrestamo_Exitoso() {
        // Arrange
        when(libroRepository.findById(1L)).thenReturn(Optional.of(libro));
        when(usuarioRepository.findByUsername("testuser")).thenReturn(Optional.of(usuario));
        when(prestamoRepository.countByUsuarioAndEstado(any(), any())).thenReturn(0L);
        when(prestamoRepository.save(any())).thenAnswer(inv -> inv.getArgument(0));
        
        // Act
        Prestamo resultado = service.solicitarPrestamo(1L, "testuser");
        
        // Assert
        assertNotNull(resultado);
        assertEquals(EstadoPrestamo.ACTIVO, resultado.getEstado());
        assertFalse(libro.isDisponible());
        verify(libroRepository).save(libro);
    }
    
    @Test
    @DisplayName("Solicitar prestamo - libro no disponible")
    void solicitarPrestamo_LibroNoDisponible_LanzaExcepcion() {
        libro.setDisponible(false);
        when(libroRepository.findById(1L)).thenReturn(Optional.of(libro));
        
        assertThrows(BusinessException.class, () -> {
            service.solicitarPrestamo(1L, "testuser");
        });
    }
    
    @Test
    @DisplayName("Solicitar prestamo - limite alcanzado")
    void solicitarPrestamo_LimiteAlcanzado_LanzaExcepcion() {
        when(libroRepository.findById(1L)).thenReturn(Optional.of(libro));
        when(usuarioRepository.findByUsername("testuser")).thenReturn(Optional.of(usuario));
        when(prestamoRepository.countByUsuarioAndEstado(any(), any())).thenReturn(3L);
        
        assertThrows(BusinessException.class, () -> {
            service.solicitarPrestamo(1L, "testuser");
        });
    }
    
    @Test
    @DisplayName("Devolver prestamo exitoso")
    void devolverPrestamo_Exitoso() {
        Prestamo prestamo = Prestamo.builder()
            .id(1L).libro(libro).usuario(usuario)
            .estado(EstadoPrestamo.ACTIVO).build();
        
        when(prestamoRepository.findById(1L)).thenReturn(Optional.of(prestamo));
        when(prestamoRepository.save(any())).thenAnswer(inv -> inv.getArgument(0));
        
        Prestamo resultado = service.devolverPrestamo(1L);
        
        assertEquals(EstadoPrestamo.DEVUELTO, resultado.getEstado());
        assertNotNull(resultado.getFechaDevolucion());
        assertTrue(libro.isDisponible());
    }
}

**IntegrationTest.java:**

@SpringBootTest
@AutoConfigureMockMvc
@Transactional
class IntegrationTest {
    
    @Autowired private MockMvc mockMvc;
    @Autowired private ObjectMapper objectMapper;
    @Autowired private LibroRepository libroRepository;
    
    @Test
    @DisplayName("Flujo completo: crear libro, prestar, devolver")
    @WithMockUser(username = "usuario", roles = {"USER"})
    void flujoCompleto() throws Exception {
        // 1. Crear libro (como admin)
        Libro libro = Libro.builder()
            .titulo("Test Book").autor("Test Author")
            .isbn("1234567890123").build();
        libro = libroRepository.save(libro);
        
        // 2. Solicitar prestamo
        MvcResult result = mockMvc.perform(post("/api/prestamos/solicitar/" + libro.getId()))
            .andExpect(status().isCreated())
            .andReturn();
        
        PrestamoResponseDTO prestamo = objectMapper.readValue(
            result.getResponse().getContentAsString(), PrestamoResponseDTO.class);
        
        // 3. Verificar libro no disponible
        mockMvc.perform(get("/api/libros/" + libro.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.disponible").value(false));
        
        // 4. Devolver
        mockMvc.perform(put("/api/prestamos/devolver/" + prestamo.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.estado").value("DEVUELTO"));
        
        // 5. Verificar libro disponible
        mockMvc.perform(get("/api/libros/" + libro.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.disponible").value(true));
    }
}

## Criterios de Evaluacion

|Criterio|Peso|Descripcion|
|---|---|---|
|Estructura del proyecto|10%|Organizacion de paquetes y clases|
|Modelo de datos|15%|Entidades JPA correctamente configuradas|
|Servicios|20%|Logica de negocio completa y correcta|
|Controllers REST|15%|Endpoints funcionando correctamente|
|Seguridad|15%|Autenticacion y autorizacion implementadas|
|Tests|15%|Cobertura de tests unitarios e integracion|
|Documentacion|10%|OpenAPI y comentarios en codigo|

## Comandos Utiles

# Ejecutar aplicacion
mvn spring-boot:run

# Ejecutar tests
mvn test

# Generar reporte de cobertura
mvn jacoco:report

# Crear JAR ejecutable
mvn package -DskipTests

# Ejecutar JAR
java -jar target/biblioteca-api-1.0.0.jar

**URLs importantes:**

- API: [http://localhost:8080/api](http://localhost:8080/api)
    
- Swagger UI: [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)
    
- H2 Console: [http://localhost:8080/h2-console](http://localhost:8080/h2-console)