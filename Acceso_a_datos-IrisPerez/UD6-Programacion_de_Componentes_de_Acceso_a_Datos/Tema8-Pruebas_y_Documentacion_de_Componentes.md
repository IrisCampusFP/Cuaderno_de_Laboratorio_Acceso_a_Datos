## Objetivos

- Implementar pruebas unitarias con JUnit 5
    
- Utilizar Mockito para crear mocks
    
- Crear pruebas de integracion con Spring Boot
    
- Documentar APIs con Swagger/OpenAPI
    
- Documentar codigo con Javadoc

## 1. Pruebas Unitarias con JUnit 5

### 1.1 Configuracion

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

Incluye: JUnit 5, Mockito, AssertJ, Hamcrest

### 1.2 Estructura de Tests

```java
class LibroServiceTest {
    
    @Test
    @DisplayName("Debe listar todos los libros")
    void debeListarTodosLosLibros() {
        // GIVEN (Dado que...)
        // Preparar datos de prueba
        
        // WHEN (Cuando...)
        // Ejecutar accion
        
        // THEN (Entonces...)
        // Verificar resultado
    }
}
```

### 1.3 Anotaciones JUnit 5

```java
class LibroServiceTest {
    
    @BeforeAll
    static void setupClass() {
        // Se ejecuta una vez antes de todos los tests
    }
    
    @BeforeEach
    void setup() {
        // Se ejecuta antes de cada test
    }
    
    @Test
    void testBasico() {
        // Test normal
    }
    
    @Test
    @Disabled("En desarrollo")
    void testDeshabilitado() {
        // No se ejecuta
    }
    
    @ParameterizedTest
    @ValueSource(strings = {"Cervantes", "Garcia Marquez", "Borges"})
    void testParametrizado(String autor) {
        assertNotNull(service.buscarPorAutor(autor));
    }
    
    @AfterEach
    void tearDown() {
        // Se ejecuta despues de cada test
    }
    
    @AfterAll
    static void tearDownClass() {
        // Se ejecuta una vez despues de todos los tests
    }
}

```
### 1.4 Assertions

```java
@Test
void testAssertions() {
    Libro libro = new Libro(1L, "Don Quijote", "Cervantes", "1234567890");
    
    // JUnit Assertions
    assertEquals("Don Quijote", libro.getTitulo());
    assertNotNull(libro.getId());
    assertTrue(libro.isDisponible());
    assertFalse(libro.getTitulo().isEmpty());
    
    // Verificar excepciones
    assertThrows(IllegalArgumentException.class, () -> {
        service.guardar(null);
    });
    
    // AssertJ (mas fluido)
    assertThat(libro.getTitulo())
        .isNotNull()
        .isNotEmpty()
        .startsWith("Don")
        .contains("Quijote");
    
    assertThat(libro.getId()).isPositive();
    
    List<Libro> libros = service.listarTodos();
    assertThat(libros)
        .hasSize(5)
        .extracting(Libro::getAutor)
        .contains("Cervantes", "Borges");
}
```

## 2. Mockito

### 2.1 Crear Mocks

```java
@ExtendWith(MockitoExtension.class)
class LibroServiceTest {
    
    @Mock
    private LibroRepository libroRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private LibroService libroService;
    
    @Test
    void debeGuardarLibro() {
        // Given
        Libro libro = new Libro(null, "Test", "Autor", "1234");
        Libro libroGuardado = new Libro(1L, "Test", "Autor", "1234");
        
        when(libroRepository.save(any(Libro.class))).thenReturn(libroGuardado);
        
        // When
        Libro resultado = libroService.guardar(libro);
        
        // Then
        assertThat(resultado.getId()).isEqualTo(1L);
        verify(libroRepository, times(1)).save(libro);
    }
}
```

### 2.2 Stubbing con Mockito

```java
@Test
void ejemplosStubbing() {
    // Retornar valor
    when(repository.findById(1L)).thenReturn(Optional.of(new Libro()));
    
    // Retornar valores diferentes en llamadas sucesivas
    when(repository.count())
        .thenReturn(0L)
        .thenReturn(1L)
        .thenReturn(2L);
    
    // Lanzar excepcion
    when(repository.findById(999L))
        .thenThrow(new EntityNotFoundException("No encontrado"));
    
    // Responder segun argumento
    when(repository.save(any(Libro.class)))
        .thenAnswer(invocation -> {
            Libro libro = invocation.getArgument(0);
            libro.setId(100L);
            return libro;
        });
}
```

### 2.3 Verificaciones con Mockito

```java
@Test
void ejemplosVerify() {
    // Verificar que se llamo al metodo
    verify(repository).save(any(Libro.class));
    
    // Verificar numero de invocaciones
    verify(repository, times(1)).save(any());
    verify(repository, never()).delete(any());
    verify(repository, atLeast(1)).findAll();
    verify(repository, atMost(3)).findById(anyLong());
    
    // Verificar argumentos
    ArgumentCaptor<Libro> captor = ArgumentCaptor.forClass(Libro.class);
    verify(repository).save(captor.capture());
    
    Libro libroCapturado = captor.getValue();
    assertThat(libroCapturado.getTitulo()).isEqualTo("Esperado");
    
    // Verificar orden de llamadas
    InOrder inOrder = inOrder(repository, emailService);
    inOrder.verify(repository).save(any());
    inOrder.verify(emailService).enviar(any(), any(), any());
}

```

## 3. Tests de Integracion con Spring Boot

### 3.1 `@SpringBootTest`

```java
@SpringBootTest
@Transactional
class LibroServiceIntegrationTest {
    
    @Autowired
    private LibroService libroService;
    
    @Autowired
    private LibroRepository libroRepository;
    
    @Test
    void debeGuardarYRecuperarLibro() {
        // Given
        Libro libro = new Libro(null, "Test", "Autor", "1234567890");
        
        // When
        Libro guardado = libroService.guardar(libro);
        Optional<Libro> recuperado = libroRepository.findById(guardado.getId());
        
        // Then
        assertThat(recuperado).isPresent();
        assertThat(recuperado.get().getTitulo()).isEqualTo("Test");
    }
}
```

### 3.2 `@DataJpaTest` (Solo JPA)

```java
@DataJpaTest
class LibroRepositoryTest {
    
    @Autowired
    private LibroRepository repository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Test
    void debeBuscarPorAutor() {
        // Given
        Libro libro = new Libro(null, "Test", "Cervantes", "123");
        entityManager.persistAndFlush(libro);
        
        // When
        List<Libro> resultado = repository.findByAutor("Cervantes");
        
        // Then
        assertThat(resultado).hasSize(1);
        assertThat(resultado.get(0).getAutor()).isEqualTo("Cervantes");
    }
}

```
### 3.3 `@WebMvcTest` (Solo Controllers)

```java
@WebMvcTest(LibroController.class)
class LibroControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private LibroService libroService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    void debeListarLibros() throws Exception {
        // Given
        List<Libro> libros = List.of(
            new Libro(1L, "Libro 1", "Autor 1", "111"),
            new Libro(2L, "Libro 2", "Autor 2", "222")
        );
        when(libroService.listarTodos()).thenReturn(libros);
        
        // When & Then
        mockMvc.perform(get("/api/libros")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$", hasSize(2)))
            .andExpect(jsonPath("$[0].titulo").value("Libro 1"));
    }
    
    @Test
    void debeCrearLibro() throws Exception {
        // Given
        LibroCreateDTO dto = new LibroCreateDTO("Nuevo", "Autor", "123");
        Libro guardado = new Libro(1L, "Nuevo", "Autor", "123");
        when(libroService.guardar(any())).thenReturn(guardado);
        
        // When & Then
        mockMvc.perform(post("/api/libros")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(dto)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1));
    }
}

```

## 4. Documentacion con OpenAPI/Swagger

### 4.1 Configuracion

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

```properties
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.operationsSorter=method

```
### 4.2 Anotaciones OpenAPI

```java
@RestController
@RequestMapping("/api/libros")
@Tag(name = "Libros", description = "API de gestion de libros")
public class LibroController {
    
    @Operation(
        summary = "Listar todos los libros",
        description = "Obtiene una lista paginada de todos los libros"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Lista de libros",
            content = @Content(array = @ArraySchema(schema = @Schema(implementation = LibroDTO.class)))),
        @ApiResponse(responseCode = "500", description = "Error interno")
    })
    @GetMapping
    public List<LibroDTO> listar(
            @Parameter(description = "Numero de pagina") @RequestParam(defaultValue = "0") int pagina,
            @Parameter(description = "Tamanio de pagina") @RequestParam(defaultValue = "10") int tamanio) {
        return service.listarPaginado(pagina, tamanio);
    }
    
    @Operation(summary = "Obtener libro por ID")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Libro encontrado"),
        @ApiResponse(responseCode = "404", description = "Libro no encontrado")
    })
    @GetMapping("/{id}")
    public LibroDTO buscar(
            @Parameter(description = "ID del libro", required = true) @PathVariable Long id) {
        return service.buscar(id);
    }
}
```

### 4.3 Documentar DTOs

```java
@Schema(description = "Libro para crear")
@Data
public class LibroCreateDTO {
    
    @Schema(description = "Titulo del libro", example = "Don Quijote")
    @NotBlank
    private String titulo;
    
    @Schema(description = "Autor del libro", example = "Miguel de Cervantes")
    @NotBlank
    private String autor;
    
    @Schema(description = "ISBN (10 o 13 digitos)", example = "9788420412146")
    @Pattern(regexp = "\\d{10,13}")
    private String isbn;
}

@Schema(description = "Respuesta de libro")
@Data
public class LibroDTO {
    
    @Schema(description = "ID unico", example = "1")
    private Long id;
    
    @Schema(description = "Titulo")
    private String titulo;
    
    @Schema(description = "Disponible para prestamo")
    private boolean disponible;
}
```

## 5. Javadoc
### 5.1 Documentar Clases y Metodos

```java
/**
 * Servicio para la gestion de libros en la biblioteca.
 * <p>
 * Proporciona operaciones CRUD y logica de negocio para
 * la gestion del catalogo de libros.
 * </p>
 * 
 * @author David Valbuena
 * @version 1.0
 * @since 2025
 * @see LibroRepository
 */
@Service
public class LibroService {
    
    /**
     * Busca un libro por su identificador.
     * 
     * @param id identificador unico del libro
     * @return Optional con el libro si existe, vacio si no
     * @throws IllegalArgumentException si id es null
     */
    public Optional<Libro> buscarPorId(Long id) {
        if (id == null) {
            throw new IllegalArgumentException("ID no puede ser null");
        }
        return repository.findById(id);
    }
    
    /**
     * Realiza el prestamo de un libro a un usuario.
     * 
     * @param libroId ID del libro a prestar
     * @param usuarioId ID del usuario que solicita el prestamo
     * @return Prestamo creado
     * @throws EntityNotFoundException si libro o usuario no existen
     * @throws BusinessException si el libro no esta disponible
     */
    @Transactional
    public Prestamo prestar(Long libroId, Long usuarioId) {
        // Implementacion...
    }
}
```

### 5.2 Generar Javadoc

<!-- pom.xml -->
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>3.6.3</version>
    <configuration>
        <encoding>UTF-8</encoding>
        <docencoding>UTF-8</docencoding>
    </configuration>
</plugin>
```

```powershell
# Generar Javadoc
mvn javadoc:javadoc


# Resultado en target/site/apidocs/
```

## 6. Resumen

### Tipos de Tests

|Tipo|Anotacion|Proposito|
|---|---|---|
|Unitario|`@ExtendWith(MockitoExtension.class)`|Probar logica aislada|
|Integracion|`@SpringBootTest`|Probar todo el contexto|
|Repository|`@DataJpaTest`|Probar acceso a datos|
|Controller|`@WebMvcTest`|Probar endpoints REST|

### Documentacion

| Herramienta     | Uso                    |
| --------------- | ---------------------- |
| Swagger/OpenAPI | Documentar APIs REST   |
| Javadoc         | Documentar codigo Java |
| README.md       | Documentar proyecto    |
