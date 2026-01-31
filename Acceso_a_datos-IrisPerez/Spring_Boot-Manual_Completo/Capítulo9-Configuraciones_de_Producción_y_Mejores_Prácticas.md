## 9.1 Perfiles de Configuración

**Configuración por entornos:**

**application-dev.yml** (Desarrollo):

spring:
  datasource:
    url: jdbc:h2:mem:devdb
    username: sa
    password: 
  h2:
    console:
      enabled: true
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    format-sql: true
    
logging:
  level:
    com.ejemplo: DEBUG
    org.springframework.security: DEBUG

**application-prod.yml** (Producción):

spring:
  datasource:
    url: jdbc:mysql://${DB_HOST:localhost}:3306/${DB_NAME:app_db}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 10
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    
logging:
  level:
    root: WARN
    com.ejemplo: INFO
  file:
    name: app.log
    
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics

## 9.2 Configuración de Seguridad para Producción

@Configuration
@EnableWebSecurity
@Profile("prod")
public class ConfiguracionSeguridadProduccion {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/actuator/**").hasRole("ADMIN")
                .requestMatchers("/api/publico/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .headers(headers -> headers
                .contentSecurityPolicy("default-src 'self'")
                .frameOptions().sameOrigin()
                .httpStrictTransportSecurity(hsts -> hsts
                    .includeSubDomains(true)
                    .maxAgeInSeconds(31536000)
                )
            )
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            );
            
        return http.build();
    }
}