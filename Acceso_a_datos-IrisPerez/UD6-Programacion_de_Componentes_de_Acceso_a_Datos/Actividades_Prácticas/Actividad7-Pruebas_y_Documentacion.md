## Objetivos

- Escribir tests unitarios con JUnit 5
    
- Usar Mockito para simular dependencias
    
- Crear tests de integracion con Spring Boot Test
    
- Documentar API con OpenAPI/Swagger
    

## Ejercicio 1: Tests Unitarios de Servicio

### Enunciado

Escribe tests unitarios para `LibroService` que prueben:

1. Guardar libro nuevo
    
2. Buscar libro existente
    
3. Buscar libro no existente (debe lanzar excepcion)
    
4. Listar todos los libros
    

Ver solucion

@ExtendWith(MockitoExtension.class)
class LibroServiceTest {
    
    @Mock
    private LibroRepository repository;
    
    @InjectMocks
    private LibroService service;
    
    private Libro libro;
    
    @BeforeEach
    void setUp() {
        libro = new Libro();
        libro.setId(1L);
        libro.setTitulo("Don Quijote");
        libro.setAutor("Cervantes");
        libro.setIsbn("9788420412146");
    }
    
    @Test
    @DisplayName("Guardar libro debe retornar libro con ID")
    void guardarLibro_DebeRetornarLibroConId() {
        // Arrange
        Libro libroSinId = new Libro();
        libroSinId.setTitulo("Nuevo Libro");
        
        Libro libroGuardado = new Libro();
        libroGuardado.setId(1L);
        libroGuardado.setTitulo("Nuevo Libro");
        
        when(repository.save(any(Libro.class))).thenReturn(libroGuardado);
        
        // Act
        Libro resultado = service.guardar(libroSinId);
        
        // Assert
        assertNotNull(resultado.getId());
        assertEquals("Nuevo Libro", resultado.getTitulo());
        verify(repository, times(1)).save(any(Libro.class));
    }
    
    @Test
    @DisplayName("Buscar libro existente debe retornar libro")
    void buscarPorId_LibroExiste_DebeRetornarLibro() {
        // Arrange
        when(repository.findById(1L)).thenReturn(Optional.of(libro));
        
        // Act
        Libro resultado = service.buscarPorId(1L);
        
        // Assert
        assertNotNull(resultado);
        assertEquals("Don Quijote", resultado.getTitulo());
        assertEquals("Cervantes", resultado.getAutor());
    }
    
    @Test
    @DisplayName("Buscar libro no existente debe lanzar excepcion")
    void buscarPorId_LibroNoExiste_DebeLanzarExcepcion() {
        // Arrange
        when(repository.findById(99L)).thenReturn(Optional.empty());
        
        // Act & Assert
        assertThrows(EntityNotFoundException.class, () -> {
            service.buscarPorId(99L);
        });
    }
    
    @Test
    @DisplayName("Listar todos debe retornar lista de libros")
    void listarTodos_DebeRetornarListaLibros() {
        // Arrange
        List<Libro> libros = Arrays.asList(libro, new Libro());
        when(repository.findAll()).thenReturn(libros);
        
        // Act
        List<Libro> resultado = service.listarTodos();
        
        // Assert
        assertEquals(2, resultado.size());
        verify(repository, times(1)).findAll();
    }
}

## Ejercicio 2: Tests de Controller con MockMvc

### Enunciado

Escribe tests para `LibroController` usando MockMvc:

1. GET /api/libros - debe retornar lista
    
2. GET /api/libros/{id} - libro existente
    
3. GET /api/libros/{id} - libro no existente (404)
    
4. POST /api/libros - crear libro
    
5. POST /api/libros - validacion fallida (400)
    

Ver solucion

@WebMvcTest(LibroController.class)
class LibroControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private LibroService service;
    
    @MockBean
    private LibroMapper mapper;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    @DisplayName("GET /api/libros debe retornar lista de libros")
    void listar_DebeRetornarListaLibros() throws Exception {
        // Arrange
        List<LibroResponseDTO> libros = Arrays.asList(
            new LibroResponseDTO(1L, "Libro 1", "Autor 1", "1234567890", 2020, true),
            new LibroResponseDTO(2L, "Libro 2", "Autor 2", "0987654321", 2021, true)
        );
        
        when(service.listarTodos()).thenReturn(List.of());
        when(mapper.toDTOList(any())).thenReturn(libros);
        
        // Act & Assert
        mockMvc.perform(get("/api/libros")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$", hasSize(2)))
            .andExpect(jsonPath("$[0].titulo").value("Libro 1"))
            .andExpect(jsonPath("$[1].titulo").value("Libro 2"));
    }
    
    @Test
    @DisplayName("GET /api/libros/{id} existente debe retornar libro")
    void buscar_LibroExiste_DebeRetornarLibro() throws Exception {
        // Arrange
        Libro libro = new Libro();
        libro.setId(1L);
        libro.setTitulo("Don Quijote");
        
        LibroResponseDTO dto = new LibroResponseDTO(
            1L, "Don Quijote", "Cervantes", "1234567890", 1605, true
        );
        
        when(service.buscarPorId(1L)).thenReturn(Optional.of(libro));
        when(mapper.toDTO(any())).thenReturn(dto);
        
        // Act & Assert
        mockMvc.perform(get("/api/libros/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.titulo").value("Don Quijote"))
            .andExpect(jsonPath("$.autor").value("Cervantes"));
    }
    
    @Test
    @DisplayName("GET /api/libros/{id} no existente debe retornar 404")
    void buscar_LibroNoExiste_DebeRetornar404() throws Exception {
        // Arrange
        when(service.buscarPorId(99L)).thenReturn(Optional.empty());
        
        // Act & Assert
        mockMvc.perform(get("/api/libros/99"))
            .andExpect(status().isNotFound());
    }
    
    @Test
    @DisplayName("POST /api/libros debe crear libro")
    void crear_LibroValido_DebeCrearLibro() throws Exception {
        // Arrange
        LibroCreateDTO createDTO = new LibroCreateDTO();
        createDTO.setTitulo("Nuevo Libro");
        createDTO.setAutor("Autor");
        createDTO.setIsbn("1234567890123");
        
        Libro libro = new Libro();
        libro.setId(1L);
        
        LibroResponseDTO responseDTO = new LibroResponseDTO(
            1L, "Nuevo Libro", "Autor", "1234567890123", null, true
        );
        
        when(mapper.toEntity(any())).thenReturn(libro);
        when(service.guardar(any())).thenReturn(libro);
        when(mapper.toDTO(any())).thenReturn(responseDTO);
        
        // Act & Assert
        mockMvc.perform(post("/api/libros")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(createDTO)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.titulo").value("Nuevo Libro"));
    }
    
    @Test
    @DisplayName("POST /api/libros con datos invalidos debe retornar 400")
    void crear_LibroInvalido_DebeRetornar400() throws Exception {
        // Arrange - titulo vacio (violacion de validacion)
        LibroCreateDTO createDTO = new LibroCreateDTO();
        createDTO.setTitulo(""); // @NotBlank falla
        
        // Act & Assert
        mockMvc.perform(post("/api/libros")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(createDTO)))
            .andExpect(status().isBadRequest());
    }
}

## Ejercicio 3: Tests de Integracion con Base de Datos

### Enunciado

Escribe tests de integracion que prueben el repositorio con una BD H2 en memoria:

1. Guardar y recuperar libro
    
2. Buscar por ISBN
    
3. Buscar por autor (case insensitive)
    
4. Query personalizada
    

Ver solucion

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@TestPropertySource(properties = {
    "spring.datasource.url=jdbc:h2:mem:testdb",
    "spring.jpa.hibernate.ddl-auto=create-drop"
})
class LibroRepositoryIntegrationTest {
    
    @Autowired
    private LibroRepository repository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Test
    @DisplayName("Guardar y recuperar libro")
    void guardarYRecuperar() {
        // Arrange
        Libro libro = new Libro();
        libro.setTitulo("Test Book");
        libro.setAutor("Test Author");
        libro.setIsbn("1234567890123");
        
        // Act
        Libro guardado = repository.save(libro);
        entityManager.flush();
        entityManager.clear();
        
        Optional<Libro> encontrado = repository.findById(guardado.getId());
        
        // Assert
        assertTrue(encontrado.isPresent());
        assertEquals("Test Book", encontrado.get().getTitulo());
    }
    
    @Test
    @DisplayName("Buscar por ISBN")
    void buscarPorIsbn() {
        // Arrange
        Libro libro = new Libro();
        libro.setTitulo("ISBN Test");
        libro.setIsbn("9876543210");
        entityManager.persistAndFlush(libro);
        
        // Act
        Optional<Libro> encontrado = repository.findByIsbn("9876543210");
        
        // Assert
        assertTrue(encontrado.isPresent());
        assertEquals("ISBN Test", encontrado.get().getTitulo());
    }
    
    @Test
    @DisplayName("Buscar por autor case insensitive")
    void buscarPorAutorIgnoreCase() {
        // Arrange
        Libro libro1 = new Libro();
        libro1.setTitulo("Libro 1");
        libro1.setAutor("Gabriel Garcia Marquez");
        
        Libro libro2 = new Libro();
        libro2.setTitulo("Libro 2");
        libro2.setAutor("Jorge Luis Borges");
        
        entityManager.persist(libro1);
        entityManager.persist(libro2);
        entityManager.flush();
        
        // Act
        List<Libro> encontrados = repository.findByAutorContainingIgnoreCase("garcia");
        
        // Assert
        assertEquals(1, encontrados.size());
        assertEquals("Gabriel Garcia Marquez", encontrados.get(0).getAutor());
    }
    
    @Test
    @DisplayName("Buscar libros disponibles")
    void buscarDisponibles() {
        // Arrange
        Libro disponible = new Libro();
        disponible.setTitulo("Disponible");
        disponible.setDisponible(true);
        
        Libro noDisponible = new Libro();
        noDisponible.setTitulo("No Disponible");
        noDisponible.setDisponible(false);
        
        entityManager.persist(disponible);
        entityManager.persist(noDisponible);
        entityManager.flush();
        
        // Act
        List<Libro> disponibles = repository.findByDisponibleTrue();
        
        // Assert
        assertEquals(1, disponibles.size());
        assertTrue(disponibles.get(0).isDisponible());
    }
}

## Ejercicio 4: Tests de Seguridad

### Enunciado

Escribe tests que verifiquen la configuracion de seguridad:

1. Endpoint publico accesible sin autenticacion
    
2. Endpoint protegido requiere autenticacion
    
3. Endpoint admin requiere rol ADMIN
    
4. Usuario sin permisos recibe 403
    

Ver solucion

@SpringBootTest
@AutoConfigureMockMvc
class SecurityIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    @DisplayName("GET publico accesible sin autenticacion")
    void endpointPublico_SinAuth_DebeRetornar200() throws Exception {
        mockMvc.perform(get("/api/libros"))
            .andExpect(status().isOk());
    }
    
    @Test
    @DisplayName("POST sin autenticacion debe retornar 401")
    void endpointProtegido_SinAuth_DebeRetornar401() throws Exception {
        mockMvc.perform(post("/api/libros")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{}"))
            .andExpect(status().isUnauthorized());
    }
    
    @Test
    @WithMockUser(username = "usuario", roles = {"USER"})
    @DisplayName("POST con USER debe permitir crear")
    void endpointProtegido_ConUser_DebePermitir() throws Exception {
        String libroJson = """
            {
                "titulo": "Test",
                "autor": "Autor",
                "isbn": "1234567890123"
            }
            """;
        
        mockMvc.perform(post("/api/libros")
                .contentType(MediaType.APPLICATION_JSON)
                .content(libroJson))
            .andExpect(status().isCreated());
    }
    
    @Test
    @WithMockUser(username = "usuario", roles = {"USER"})
    @DisplayName("Endpoint admin con USER debe retornar 403")
    void endpointAdmin_ConUser_DebeRetornar403() throws Exception {
        mockMvc.perform(get("/api/admin/stats"))
            .andExpect(status().isForbidden());
    }
    
    @Test
    @WithMockUser(username = "admin", roles = {"ADMIN"})
    @DisplayName("Endpoint admin con ADMIN debe permitir")
    void endpointAdmin_ConAdmin_DebePermitir() throws Exception {
        mockMvc.perform(get("/api/admin/stats"))
            .andExpect(status().isOk());
    }
    
    @Test
    @DisplayName("Basic Auth con credenciales validas")
    void basicAuth_CredencialesValidas_DebePermitir() throws Exception {
        mockMvc.perform(get("/api/perfil")
                .with(httpBasic("usuario", "pass123")))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.username").value("usuario"));
    }
    
    @Test
    @DisplayName("Basic Auth con credenciales invalidas")
    void basicAuth_CredencialesInvalidas_DebeRetornar401() throws Exception {
        mockMvc.perform(get("/api/perfil")
                .with(httpBasic("usuario", "wrongpass")))
            .andExpect(status().isUnauthorized());
    }
}

## Ejercicio 5: Documentacion OpenAPI

### Enunciado

Documenta el API REST usando anotaciones OpenAPI:

1. Configurar springdoc-openapi
    
2. Documentar endpoints del controller
    
3. Documentar DTOs
    
4. Añadir informacion de seguridad
    

Ver solucion

**pom.xml:**

<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>

**OpenApiConfig.java:**

@Configuration
public class OpenApiConfig {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("Biblioteca API")
                .version("1.0.0")
                .description("API REST para gestion de biblioteca")
                .contact(new Contact()
                    .name("Soporte")
                    .email("soporte@biblioteca.com"))
                .license(new License()
                    .name("Apache 2.0")
                    .url("http://www.apache.org/licenses/LICENSE-2.0")))
            .addSecurityItem(new SecurityRequirement().addList("basicAuth"))
            .components(new Components()
                .addSecuritySchemes("basicAuth", new SecurityScheme()
                    .type(SecurityScheme.Type.HTTP)
                    .scheme("basic")));
    }
}

**LibroController.java documentado:**

@RestController
@RequestMapping("/api/libros")
@Tag(name = "Libros", description = "Operaciones CRUD de libros")
@RequiredArgsConstructor
public class LibroController {
    
    private final LibroService service;
    private final LibroMapper mapper;
    
    @Operation(
        summary = "Listar todos los libros",
        description = "Retorna una lista de todos los libros disponibles"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Lista de libros",
            content = @Content(array = @ArraySchema(
                schema = @Schema(implementation = LibroResponseDTO.class))))
    })
    @GetMapping
    public List<LibroResponseDTO> listar() {
        return mapper.toDTOList(service.listarTodos());
    }
    
    @Operation(summary = "Buscar libro por ID")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Libro encontrado"),
        @ApiResponse(responseCode = "404", description = "Libro no encontrado",
            content = @Content(schema = @Schema(implementation = ErrorResponse.class)))
    })
    @GetMapping("/{id}")
    public LibroResponseDTO buscar(
            @Parameter(description = "ID del libro", required = true)
            @PathVariable Long id) {
        Libro libro = service.buscarPorId(id)
            .orElseThrow(() -> new EntityNotFoundException("Libro no encontrado: " + id));
        return mapper.toDTO(libro);
    }
    
    @Operation(
        summary = "Crear nuevo libro",
        security = @SecurityRequirement(name = "basicAuth")
    )
    @ApiResponses({
        @ApiResponse(responseCode = "201", description = "Libro creado"),
        @ApiResponse(responseCode = "400", description = "Datos invalidos"),
        @ApiResponse(responseCode = "401", description = "No autenticado")
    })
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public LibroResponseDTO crear(
            @RequestBody @Valid LibroCreateDTO dto) {
        Libro libro = mapper.toEntity(dto);
        return mapper.toDTO(service.guardar(libro));
    }
}

**DTOs documentados:**

@Data
@Schema(description = "Datos para crear un libro")
public class LibroCreateDTO {
    
    @Schema(description = "Titulo del libro", example = "Don Quijote", required = true)
    @NotBlank(message = "El titulo es obligatorio")
    @Size(min = 2, max = 200)
    private String titulo;
    
    @Schema(description = "Autor del libro", example = "Miguel de Cervantes")
    @NotBlank
    private String autor;
    
    @Schema(description = "ISBN del libro", example = "9788420412146", pattern = "\\d{10,13}")
    @Pattern(regexp = "\\d{10,13}")
    private String isbn;
    
    @Schema(description = "Ano de publicacion", example = "1605", minimum = "1000", maximum = "2100")
    @Min(1000) @Max(2100)
    private Integer anioPublicacion;
}

**Acceso a documentacion:**

- Swagger UI: [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)
    
- OpenAPI JSON: [http://localhost:8080/v3/api-docs](http://localhost:8080/v3/api-docs)