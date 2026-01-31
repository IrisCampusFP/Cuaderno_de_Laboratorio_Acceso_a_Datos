## 1.1 Información Actualizada sobre Spring Boot

Spring Boot 3.5.6, lanzado en septiembre de 2025, representa la culminación de años de evolución en el desarrollo de aplicaciones Java empresariales. Esta versión se basa en **Spring Framework 6.2.x** y requiere **Java 17 como mínimo**, con soporte completo hasta **Java 21+**.

**Características clave de Spring Boot 3.5:**

**Configuración mejorada:** La versión 3.5 introduce importaciones de variables de entorno múltiples mediante `spring.config.import=env:MY_CONFIGURATION`, permitiendo configuraciones más flexibles y orientadas a contenedores. Las reglas de validación de perfiles se han vuelto más estrictas, aceptando letras, dígitos, guiones, guiones bajos, puntos, signos más y arrobas.

**Características cloud-native:** El soporte SSL del lado cliente se ha extendido a múltiples tecnologías incluyendo Cassandra, Couchbase, Elasticsearch, Kafka, MongoDB, RabbitMQ y Redis. Las imágenes de contenedor por defecto ahora utilizan `paketobuildpacks/builder-noble-java-tiny` para imágenes más pequeñas y eficientes.

**Mejoras en desarrollo web:** WebClient ahora incluye configuración global para timeouts y redirecciones, siguiendo redirecciones por defecto. El logging estructurado soporta nativamente formato JSON para mejor integración con sistemas como ELK, Loki y Datadog.

**Optimizaciones de rendimiento:** La inicialización de beans en segundo plano utiliza un bean `bootstrapExecutor` auto-configurado para mejorar los tiempos de arranque. Spring Data JPA 3.5.4 incluye significativas mejoras de rendimiento a través del uso de JPQL para consultas derivadas.

## 1.2 Arquitectura Pedagógica de Spring Boot

Para estudiantes de DAM, es fundamental comprender que Spring Boot opera bajo el principio de **«convención sobre configuración»**. Esto significa que el framework asume configuraciones predeterminadas sensatas, permitiendo que los desarrolladores se enfoquen en la lógica de negocio en lugar de configuraciones repetitivas.

**Los tres pilares educativos de Spring Boot:**

1. **Starters**: Dependencias preconfiguradas que incluyen todas las librerías necesarias para una funcionalidad específica
    
2. **Auto-configuración**: Configuración automática basada en las dependencias presentes en el classpath
    
3. **Actuator**: Herramientas de monitoreo y gestión para aplicaciones en producción