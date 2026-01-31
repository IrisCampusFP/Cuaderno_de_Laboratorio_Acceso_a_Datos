## 10.1 Introducción a Microservicios con Spring Boot

Los microservicios representan un paradigma arquitectónico donde las aplicaciones se construyen como suite de servicios pequeños, independientes y débilmente acoplados. Spring Boot proporciona excelente soporte para este patrón a través del ecosistema Spring Cloud.

**Características clave de microservicios educativos:**

@SpringBootApplication
@EnableEurekaClient // Para descubrimiento de servicios
@EnableCircuitBreaker // Para tolerancia a fallos
public class ServicioEstudiantesApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServicioEstudiantesApplication.class, args);
    }
}

**Configuración de un microservicio:**

spring:
  application:
    name: servicio-estudiantes
  cloud:
    config:
      uri: http://localhost:8888 # Servidor de configuración
      
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka # Registro de servicios
      
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always