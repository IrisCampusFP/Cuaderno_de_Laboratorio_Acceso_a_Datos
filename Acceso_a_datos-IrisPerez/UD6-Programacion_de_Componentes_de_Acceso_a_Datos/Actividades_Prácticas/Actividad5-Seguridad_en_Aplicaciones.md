## Objetivos

- Configurar Spring Security basico
    
- Implementar autenticacion con usuarios en memoria
    
- Proteger endpoints por roles
    

## Ejercicio 1: Configuracion Basica

### Enunciado

Configura Spring Security con los siguientes requisitos:

1. GET `/api/libros` publico
    
2. POST/PUT/DELETE `/api/libros/**` requiere rol USER
    
3. `/api/admin/**` requiere rol ADMIN
    
4. Dos usuarios: «usuario» (USER) y «admin» (USER, ADMIN)
    

Ver solucion

**pom.xml:**

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

**SecurityConfig.java:**

@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(HttpMethod.GET, "/api/libros", "/api/libros/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/libros/**").hasRole("USER")
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());
        
        return http.build();
    }
    
    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.builder()
            .username("usuario")
            .password(passwordEncoder().encode("pass123"))
            .roles("USER")
            .build();
        
        UserDetails admin = User.builder()
            .username("admin")
            .password(passwordEncoder().encode("admin123"))
            .roles("USER", "ADMIN")
            .build();
        
        return new InMemoryUserDetailsManager(user, admin);
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

## Ejercicio 2: Seguridad por Metodo

### Enunciado

Habilita la seguridad por metodo y protege el servicio:

1. `listarTodos()` - acceso publico
    
2. `guardar()` - solo USER
    
3. `eliminar()` - solo ADMIN
    

Ver solucion

**Habilitar:**

@Configuration
@EnableMethodSecurity
public class MethodSecurityConfig {
}

**Servicio protegido:**

@Service
public class LibroService {
    
    private final LibroRepository repository;
    
    // Publico
    public List<Libro> listarTodos() {
        return repository.findAll();
    }
    
    // Solo USER
    @PreAuthorize("hasRole('USER')")
    public Libro guardar(Libro libro) {
        return repository.save(libro);
    }
    
    // Solo ADMIN
    @PreAuthorize("hasRole('ADMIN')")
    public void eliminar(Long id) {
        repository.deleteById(id);
    }
    
    // Solo el propietario o ADMIN
    @PreAuthorize("#username == authentication.name or hasRole('ADMIN')")
    public List<Prestamo> verPrestamos(String username) {
        return prestamoRepository.findByUsuarioUsername(username);
    }
}

## Ejercicio 3: Obtener Usuario Autenticado

### Enunciado

Crea un endpoint `/api/perfil` que devuelva informacion del usuario autenticado.

Ver solucion

@Data
@AllArgsConstructor
public class PerfilDTO {
    private String username;
    private List<String> roles;
}

@RestController
@RequestMapping("/api/perfil")
public class PerfilController {
    
    @GetMapping
    public PerfilDTO obtenerPerfil(@AuthenticationPrincipal UserDetails userDetails) {
        List<String> roles = userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList());
        
        return new PerfilDTO(userDetails.getUsername(), roles);
    }
    
    // Alternativa con SecurityContextHolder
    @GetMapping("/v2")
    public PerfilDTO obtenerPerfilV2() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        
        List<String> roles = auth.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList());
        
        return new PerfilDTO(auth.getName(), roles);
    }
}

## Ejercicio 4: Probar Seguridad

### Enunciado

Prueba los endpoints con diferentes usuarios usando curl.

Ver solucion

# GET publico (sin autenticacion)
curl http://localhost:8080/api/libros

# POST como usuario
curl -X POST http://localhost:8080/api/libros \
  -u usuario:pass123 \
  -H "Content-Type: application/json" \
  -d '{"titulo": "Test", "autor": "Autor"}'

# DELETE como admin
curl -X DELETE http://localhost:8080/api/libros/1 \
  -u admin:admin123

# DELETE como usuario (debe fallar - 403)
curl -X DELETE http://localhost:8080/api/libros/1 \
  -u usuario:pass123

# Endpoint admin
curl http://localhost:8080/api/admin/stats \
  -u admin:admin123

# Ver perfil
curl http://localhost:8080/api/perfil \
  -u usuario:pass123

**Respuestas esperadas:**

- 200: Acceso permitido
    
- 401: No autenticado
    
- 403: Autenticado pero sin permisos