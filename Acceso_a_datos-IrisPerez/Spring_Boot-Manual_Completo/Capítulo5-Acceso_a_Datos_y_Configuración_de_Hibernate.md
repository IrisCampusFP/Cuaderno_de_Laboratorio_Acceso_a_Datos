## 5.1 Configuración Moderna de Hibernate

**Propiedades de configuración clave para proyectos educativos:**

spring:
  datasource:
    url: jdbc:h2:mem:testdb
    username: sa
    password: 
    driver-class-name: org.h2.Driver
    hikari:
      pool-name: SpringBootHikariCP
      maximum-pool-size: 10
      minimum-idle: 5
      idle-timeout: 30000
      max-lifetime: 1800000
      connection-timeout: 30000
      
  jpa:
    hibernate:
      ddl-auto: create-drop  # Para entornos de desarrollo/educación
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
    show-sql: true
    format-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.H2Dialect
        enable_lazy_load_no_trans: false
        cache:
          use_second_level_cache: false
        jdbc:
          batch_size: 20
        order_inserts: true
        order_updates: true
  
  h2:
    console:
      enabled: true
      path: /h2-console

## 5.2 Patrones de Diseño de Entidades para Educación

@Entity
@Table(name = "estudiantes")
public class Estudiante {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    @NotBlank(message = "El nombre es obligatorio")
    private String nombre;
    
    @Column(unique = true, nullable = false)
    @Email(message = "Debe proporcionar un email válido")
    private String email;
    
    @Column(nullable = false)
    @Min(value = 18, message = "El estudiante debe tener al menos 18 años")
    private Integer edad;
    
    // Carga perezosa para rendimiento
    @OneToMany(mappedBy = "estudiante", fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    private Set<Matricula> matriculas = new HashSet<>();
    
    // Educacional: Siempre sobrescribir equals/hashCode para entidades
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Estudiante)) return false;
        Estudiante estudiante = (Estudiante) o;
        return Objects.equals(id, estudiante.id);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
    
    // Constructores, getters y setters
}

## 5.3 Repositorios y Patrones de Consulta

**Interfaz de repositorio con consultas personalizadas:**

@Repository
public interface RepositorioEstudiante extends JpaRepository<Estudiante, Long> {
    
    // Método de consulta derivado
    List<Estudiante> findByNombreContaining(String nombre);
    
    // Consulta personalizada con JPQL
    @Query("SELECT e FROM Estudiante e WHERE e.email = ?1")
    Optional<Estudiante> findByEmail(String email);
    
    // Consulta nativa para casos específicos
    @Query(value = "SELECT * FROM estudiantes WHERE edad BETWEEN ?1 AND ?2", 
           nativeQuery = true)
    List<Estudiante> findEstudiantesPorRangoEdad(Integer edadMinima, Integer edadMaxima);
    
    // Consulta con paginación y ordenamiento
    @Query("SELECT e FROM Estudiante e WHERE e.nombre LIKE %:nombre%")
    Page<Estudiante> findByNombreLike(@Param("nombre") String nombre, Pageable pageable);
}