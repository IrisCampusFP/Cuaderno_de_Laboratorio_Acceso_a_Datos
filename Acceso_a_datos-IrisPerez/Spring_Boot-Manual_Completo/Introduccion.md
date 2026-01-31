# Manual Completo de Spring Boot para Acceso a Datos

## ¿Qué es Spring Boot?

**Spring Boot** es un framework de desarrollo de aplicaciones Java que simplifica la creación de aplicaciones empresariales robustas y de nivel de producción. Basado en el ecosistema Spring, automatiza la configuración y reduce significativamente el código boilerplate, permitiendo que los desarrolladores se enfoquen en la lógica de negocio.

Spring Boot es especialmente valioso cuando necesitas:

- Desarrollar aplicaciones web y APIs REST de manera rápida y eficiente.
    
- Integrar múltiples tecnologías (bases de datos, seguridad, web) con configuración mínima.
    
- Crear aplicaciones listas para producción con métricas y monitoreo integrados.
    
- Implementar microservicios con patrones empresariales establecidos.

## Tecnologías y dependencias principales

En este manual trabajamos con las **tecnologías más actuales** de Spring Boot 3.5.6 y su ecosistema:

- **Spring Web**: Desarrollo de controladores REST y manejo de peticiones HTTP.
    
- **Spring Data JPA**: Acceso a datos mediante el patrón Repository con Hibernate como ORM.
    
- **Spring Security**: Autenticación, autorización y protección de aplicaciones con JWT.
    
- **Spring Boot Validation**: Validación de datos de entrada con anotaciones Bean Validation.
    
- **Spring Boot DevTools**: Herramientas de desarrollo para recarga automática y debugging.
    
- **Spring Boot Actuator**: Monitoreo y métricas de aplicaciones en producción.
    
- **H2 Database**: Base de datos en memoria para desarrollo y testing.

## Distribución de Tecnologías por Temas

|Tecnología|Tema Principal|Temas Secundarios|Descripción|
|---|---|---|---|
|**Spring Web**|Capítulo 6: Desarrollo de APIs REST|Capítulo 7: Actividades Prácticas|Desarrollo de controladores REST, manejo de peticiones HTTP, ResponseEntity|
|**Spring Data JPA**|Capítulo 5: Acceso a Datos y Hibernate|Capítulo 8: Testing  <br>Capítulo 10: Proyectos Avanzados|Acceso a datos con patrón Repository, configuración de Hibernate|
|**Spring Security**|Capítulo 4: Spring Security|Capítulo 9: Configuraciones de Producción|Autenticación, autorización, JWT, configuraciones de seguridad|
|**Spring Boot Validation**|Capítulo 6: Desarrollo de APIs REST|Capítulo 7: Actividades Prácticas|Validación de datos de entrada, anotaciones Bean Validation|
|**Spring Boot DevTools**|Capítulo 2: Configuración del Entorno|-|Herramientas de desarrollo, recarga automática, debugging|
|**Spring Boot Actuator**|Capítulo 9: Configuraciones de Producción|Capítulo 10: Proyectos Avanzados|Monitoreo, métricas, endpoints de gestión en producción|
|**H2 Database**|Capítulo 5: Acceso a Datos|Capítulo 3: Dependencias  <br>Capítulo 8: Testing|Base de datos en memoria para desarrollo y testing|
|**Spring Boot Testing**|Capítulo 8: Testing y Calidad|Capítulo 7: Actividades Prácticas|@SpringBootTest, @WebMvcTest, @DataJpaTest, MockMvc|
|**Configuración Spring**|Capítulo 3: Dependencias y Configuración|Capítulo 1: Fundamentos|@SpringBootApplication, @Configuration, application.yml|
|**Microservicios**|Capítulo 10: Proyectos Avanzados|-|Spring Cloud, Eureka, Circuit Breaker|

Esta tabla te permite ver de un vistazo dónde se desarrolla cada tecnología en profundidad (tema principal) y dónde se complementa o se usa en ejemplos (temas secundarios).

## ¿Qué tecnologías y patrones utilizaremos?

A lo largo del manual, trabajamos con las clases y patrones más importantes:

|Área de conocimiento|Tecnologías principales|
|---|---|
|**Configuración**|`@SpringBootApplication`, `@Configuration`, `@Bean`, `application.yml`|
|**Web/REST**|`@RestController`, `@RequestMapping`, `@PathVariable`, `@RequestBody`, `ResponseEntity`|
|**Acceso a Datos**|`@Entity`, `@Repository`, `JpaRepository`, `@Query`, `@Transactional`|
|**Seguridad**|`SecurityFilterChain`, `@PreAuthorize`, `PasswordEncoder`, `JWT`|
|**Validación**|`@Valid`, `@NotNull`, `@Email`, `@Size`, `@Min`, `@Max`|
|**Testing**|`@SpringBootTest`, `@WebMvcTest`, `@DataJpaTest`, `MockMvc`, `TestRestTemplate`|

## ¿Qué habilidades desarrollaremos?

Durante este manual completo, trabajamos con ejemplos reales paso a paso, explicando cada concepto de forma didáctica, lo que nos permite:

- Configurar y estructurar proyectos Spring Boot siguiendo mejores prácticas industriales.
    
- Implementar APIs REST completas con operaciones CRUD, paginación y manejo de errores.
    
- Diseñar y gestionar bases de datos relacionales usando JPA/Hibernate con patrones de diseño apropiados.
    
- Implementar sistemas de seguridad robustos con autenticación, autorización y JWT.
    
- Desarrollar aplicaciones testeable con cobertura completa de pruebas unitarias e integración.
    
- Aplicar metodologías pedagógicas basadas en evidencia para aprendizaje efectivo.

## ¿Por qué es importante este tema para DAM?

Como futuros **Técnicos Superiores en Desarrollo de Aplicaciones Multiplataforma**, dominar Spring Boot es esencial porque:

- Es el framework Java más demandado en el mercado laboral actual (2024-2025).
    
- Permite desarrollar aplicaciones empresariales escalables y mantenibles siguiendo estándares industriales.
    
- Prepara para trabajar con arquitecturas modernas como microservicios y aplicaciones cloud-native.
    
- Proporciona fundamentos sólidos para especializarse en desarrollo backend empresarial.
    
- Facilita la integración con tecnologías emergentes y sistemas distribuidos.

## Estructura pedagógica del manual

Este manual está diseñado específicamente para el **ciclo DAM** con enfoque educativo:

- **Metodología PRIMM**: Predict, Run, Investigate, Modify, Make para aprendizaje efectivo.
    
- **Progresión estructurada**: 10 capítulos que van desde fundamentos hasta implementaciones avanzadas.
    
- **Ejemplos actualizados**: Código basado en Spring Boot 3.5.6 y mejores prácticas de 2024-2025.
    
- **Proyectos prácticos**: Sistema completo de gestión académica como proyecto integrador.
    
- **Evaluación integral**: Estrategias de assessment formativo y sumativo para docentes.

## Recursos incluidos en el manual

- **Configuraciones completas** para IntelliJ IDEA con todas las dependencias necesarias.
    
- **Ejemplos de código** completamente funcionales y comentados línea por línea.
    
- **Actividades prácticas** progresivas para cada capítulo con soluciones detalladas.
    
- **Testing completo**: Ejemplos de pruebas unitarias, integración y funcionales.
    
- **Configuraciones de producción** con perfiles de entorno y mejores prácticas de seguridad.
    
- **Proyecto integrador**: Sistema de gestión académica que integra todos los conceptos aprendidos.


Este manual completo y profesional está diseñado específicamente para la asignatura de Acceso a Datos del ciclo de Desarrollo de Aplicaciones Multiplataforma (DAM) de grado superior. El documento proporciona una base sólida en Spring Boot 3.5.6, la versión estable más reciente disponible en septiembre de 2025, junto con metodologías pedagógicas basadas en evidencia para el aprendizaje efectivo de frameworks Java empresariales.

Spring Boot ha revolucionado el desarrollo de aplicaciones Java al simplificar significativamente la configuración y el despliegue, mientras mantiene toda la potencia del ecosistema Spring. Para estudiantes de DAM, dominar Spring Boot significa adquirir competencias altamente valoradas en el mercado laboral actual, donde las aplicaciones empresariales requieren desarrollo rápido, escalable y mantenible.

---

**Manual desarrollado para la asignatura de Acceso a Datos del ciclo DAM**  
**Versión:** 2024-2025 (Spring Boot 3.5.6)  
**Nivel:** Grado Superior - Técnico en Desarrollo de Aplicaciones Multiplataforma  
**IDE recomendado:** IntelliJ IDEA  
**Metodología:** Aprendizaje basado en proyectos con enfoque PRIMM


