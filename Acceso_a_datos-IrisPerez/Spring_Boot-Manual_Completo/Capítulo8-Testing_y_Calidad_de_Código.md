## 8.1 Estrategias de Testing para Spring Boot

**Testing de repositorio con @DataJpaTest:**

@DataJpaTest
class EstudianteRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private EstudianteRepository estudianteRepository;
    
    @Test
    void findByEmail_DeberiaRetornarEstudiante() {
        // Given
        Estudiante estudiante = new Estudiante("Juan", "Pérez", "juan@universidad.edu");
        entityManager.persistAndFlush(estudiante);
        
        // When
        Optional<Estudiante> encontrado = estudianteRepository.findByEmail("juan@universidad.edu");
        
        // Then
        assertThat(encontrado).isPresent();
        assertThat(encontrado.get().getNombre()).isEqualTo("Juan");
    }
    
    @Test
    void findByNombreContaining_DeberiaRetornarEstudiantesCoincidentes() {
        // Given
        entityManager.persistAndFlush(new Estudiante("Juan", "Pérez", "juan@universidad.edu"));
        entityManager.persistAndFlush(new Estudiante("Juana", "López", "juana@universidad.edu"));
        entityManager.persistAndFlush(new Estudiante("Pedro", "García", "pedro@universidad.edu"));
        
        // When
        List<Estudiante> encontrados = estudianteRepository.findByNombreContaining("Juan");
        
        // Then
        assertThat(encontrados).hasSize(2);
        assertThat(encontrados)
            .extracting(Estudiante::getNombre)
            .containsExactlyInAnyOrder("Juan", "Juana");
    }
}

**Testing de controladores con @WebMvcTest:**

@WebMvcTest(EstudianteController.class)
class EstudianteControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private EstudianteService estudianteService;
    
    @Test
    @WithMockUser(roles = "ADMIN")
    void crearEstudiante_ConDatosValidos_DeberiaRetornar201() throws Exception {
        // Given
        EstudianteSolicitud solicitud = new EstudianteSolicitud(
            "Ana García", "ana@universidad.edu", 20
        );
        EstudianteResponse respuestaEsperada = new EstudianteResponse(
            1L, "Ana García", "ana@universidad.edu", 20
        );
        
        when(estudianteService.crearEstudiante(any(EstudianteSolicitud.class)))
            .thenReturn(respuestaEsperada);
        
        // When & Then
        mockMvc.perform(post("/api/estudiantes")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "nombre": "Ana García",
                        "email": "ana@universidad.edu",
                        "edad": 20
                    }
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.nombre").value("Ana García"))
            .andExpect(jsonPath("$.email").value("ana@universidad.edu"))
            .andExpect(jsonPath("$.edad").value(20));
    }
    
    @Test
    void crearEstudiante_ConDatosInvalidos_DeberiaRetornar400() throws Exception {
        mockMvc.perform(post("/api/estudiantes")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "nombre": "",
                        "email": "email-invalido",
                        "edad": 15
                    }
                    """))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.erroresCampos.nombre").exists())
            .andExpect(jsonPath("$.erroresCampos.email").exists())
            .andExpected(jsonPath("$.erroresCampos.edad").exists());
    }
}

## 8.2 Testing de Integración Completo

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestPropertySource(locations = "classpath:application-integrationtest.properties")
@Transactional
class EstudianteIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private EstudianteRepository estudianteRepository;
    
    @LocalServerPort
    private int puerto;
    
    @Test
    void flujoCompletoEstudiante_DeberiaFuncionarCorrectamente() {
        String baseUrl = "http://localhost:" + puerto + "/api/estudiantes";
        
        // 1. Crear estudiante
        EstudianteSolicitud solicitudCreacion = new EstudianteSolicitud(
            "María González", "maria@universidad.edu", 22
        );
        
        ResponseEntity<EstudianteResponse> respuestaCreacion = restTemplate.postForEntity(
            baseUrl, solicitudCreacion, EstudianteResponse.class
        );
        
        assertThat(respuestaCreacion.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        Long estudianteId = respuestaCreacion.getBody().getId();
        
        // 2. Obtener estudiante creado
        ResponseEntity<EstudianteResponse> respuestaObtencion = restTemplate.getForEntity(
            baseUrl + "/" + estudianteId, EstudianteResponse.class
        );
        
        assertThat(respuestaObtencion.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(respuestaObtencion.getBody().getNombre()).isEqualTo("María González");
        
        // 3. Actualizar estudiante
        EstudianteSolicitud solicitudActualizacion = new EstudianteSolicitud(
            "María González Pérez", "maria.gonzalez@universidad.edu", 23
        );
        
        restTemplate.put(baseUrl + "/" + estudianteId, solicitudActualizacion);
        
        // 4. Verificar actualización
        ResponseEntity<EstudianteResponse> respuestaVerificacion = restTemplate.getForEntity(
            baseUrl + "/" + estudianteId, EstudianteResponse.class
        );
        
        assertThat(respuestaVerificacion.getBody().getNombre()).isEqualTo("María González Pérez");
        assertThat(respuestaVerificacion.getBody().getEdad()).isEqualTo(23);
        
        // 5. Eliminar estudiante
        restTemplate.delete(baseUrl + "/" + estudianteId);
        
        // 6. Verificar eliminación
        ResponseEntity<String> respuestaEliminacion = restTemplate.getForEntity(
            baseUrl + "/" + estudianteId, String.class
        );
        
        assertThat(respuestaEliminacion.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
    }
}