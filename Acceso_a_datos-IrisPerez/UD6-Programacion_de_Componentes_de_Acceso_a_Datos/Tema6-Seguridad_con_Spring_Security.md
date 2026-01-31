## Objetivos

- Comprender los conceptos de autenticacion y autorizacion
    
- Configurar Spring Security en una aplicacion Spring Boot
    
- Implementar seguridad basada en roles
    
- Introduccion a JWT (JSON Web Tokens)

## 1. Introduccion a Spring Security

**Spring Security** es el framework de seguridad estandar para aplicaciones Spring.

### 1.1 Conceptos Clave

|Concepto|Descripcion|
|---|---|
|**Autenticacion**|Verificar identidad (quien eres)|
|**Autorizacion**|Verificar permisos (que puedes hacer)|
|**Principal**|Usuario autenticado|
|**Granted Authority**|Permiso/rol otorgado|
|**Security Context**|Almacena informacion de seguridad|

### 1.2 Configuracion Basica

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

```
Al agregar esta dependencia, Spring Security:

- Protege todos los endpoints
    
- Genera un usuario por defecto (user/password en consola)
    
- Agrega pagina de login

## 2. Configuracion de Seguridad

### 2.1 Configuracion con `SecurityFilterChain`

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())  // Deshabilitar CSRF para APIs REST
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/libros/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());  // Autenticacion HTTP Basic
        
        return http.build();
    }
}
```

### 2.2 Usuarios en Memoria (Desarrollo)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.builder()
            .username("usuario")
            .password(passwordEncoder().encode("password123"))
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

```
## 3. Usuarios en Base de Datos

### 3.1 Entidad Usuario

```java
@Entity
@Table(name = "usuarios")
@Data
public class Usuario {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String username;
    
    @Column(nullable = false)
    private String password;
    
    private boolean activo = true;
    
    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "usuario_roles", joinColumns = @JoinColumn(name = "usuario_id"))
    @Column(name = "rol")
    private Set<String> roles = new HashSet<>();
}
```

### 3.2 Implementar UserDetailsService

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    private final UsuarioRepository usuarioRepository;
    
    public CustomUserDetailsService(UsuarioRepository usuarioRepository) {
        this.usuarioRepository = usuarioRepository;
    }
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Usuario usuario = usuarioRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("Usuario no encontrado: " + username));
        
        return User.builder()
            .username(usuario.getUsername())
            .password(usuario.getPassword())
            .roles(usuario.getRoles().toArray(new String[0]))
            .disabled(!usuario.isActivo())
            .build();
    }
}

```
### 3.3 Configuracion con Base de Datos

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    private final CustomUserDetailsService userDetailsService;
    
    public SecurityConfig(CustomUserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/libros/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .userDetailsService(userDetailsService)
            .httpBasic(Customizer.withDefaults());
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

## 4. Seguridad a Nivel de Metodo

### 4.1 Habilitar Seguridad por Metodo

```java
@Configuration
@EnableMethodSecurity
public class MethodSecurityConfig {
}
```

### 4.2 Anotaciones de Seguridad

```java
@Service
public class LibroService {
    
    // Solo usuarios autenticados
    @PreAuthorize("isAuthenticated()")
    public List<Libro> listarTodos() { ... }
    
    // Solo ADMIN
    @PreAuthorize("hasRole('ADMIN')")
    public void eliminar(Long id) { ... }
    
    // ADMIN o USER
    @PreAuthorize("hasAnyRole('ADMIN', 'USER')")
    public Libro buscar(Long id) { ... }
    
    // Verificar parametro
    @PreAuthorize("#username == authentication.name or hasRole('ADMIN')")
    public Usuario buscarPorUsername(String username) { ... }
    
    // Verificar despues de ejecucion
    @PostAuthorize("returnObject.propietario == authentication.name")
    public Recurso buscarRecurso(Long id) { ... }
}
```

## 5. Obtener Usuario Autenticado

```java
@RestController
@RequestMapping("/api/perfil")
public class PerfilController {
    
    // Opcion 1: @AuthenticationPrincipal
    @GetMapping
    public String perfil(@AuthenticationPrincipal UserDetails userDetails) {
        return "Usuario: " + userDetails.getUsername();
    }
    
    // Opcion 2: SecurityContextHolder
    @GetMapping("/v2")
    public String perfilV2() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        return "Usuario: " + auth.getName();
    }
    
    // Opcion 3: Principal
    @GetMapping("/v3")
    public String perfilV3(Principal principal) {
        return "Usuario: " + principal.getName();
    }
}

```

## 6. Introduccion a JWT

### 6.1 Que es JWT?

**JWT** (JSON Web Token) es un estandar para transmitir informacion de forma segura.

```
┌─────────────────────────────────────────────────────────────┐
│  HEADER.PAYLOAD.SIGNATURE                                    │
├─────────────────────────────────────────────────────────────┤
│  Header:    {"alg": "HS256", "typ": "JWT"}                  │
│  Payload:   {"sub": "usuario", "roles": ["USER"], "exp":...}│
│  Signature: HMACSHA256(header + payload, secret)             │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Dependencia JWT

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.3</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.3</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.3</version>
    <scope>runtime</scope>
</dependency>
```

### 6.3 Servicio JWT

```java
@Service
public class JwtService {
    
    @Value("${jwt.secret}")
    private String secretKey;
    
    @Value("${jwt.expiration}")
    private long expiration;  // milisegundos
    
    public String generarToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList()));
        
        return Jwts.builder()
            .claims(claims)
            .subject(userDetails.getUsername())
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(getSigningKey())
            .compact();
    }
    
    public String extraerUsername(String token) {
        return extraerClaim(token, Claims::getSubject);
    }
    
    public boolean esTokenValido(String token, UserDetails userDetails) {
        String username = extraerUsername(token);
        return username.equals(userDetails.getUsername()) && !esTokenExpirado(token);
    }
    
    private boolean esTokenExpirado(String token) {
        return extraerClaim(token, Claims::getExpiration).before(new Date());
    }
    
    private <T> T extraerClaim(String token, Function<Claims, T> resolver) {
        Claims claims = Jwts.parser()
            .verifyWith(getSigningKey())
            .build()
            .parseSignedClaims(token)
            .getPayload();
        return resolver.apply(claims);
    }
    
    private SecretKey getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}

```
### 6.4 Controlador de Autenticacion

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    
    private final AuthenticationManager authManager;
    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;
    
    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@RequestBody LoginRequest request) {
        authManager.authenticate(
            new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword())
        );
        
        UserDetails userDetails = userDetailsService.loadUserByUsername(request.getUsername());
        String token = jwtService.generarToken(userDetails);
        
        return ResponseEntity.ok(new AuthResponse(token));
    }
}

@Data
public class LoginRequest {
    private String username;
    private String password;
}

@Data
@AllArgsConstructor
public class AuthResponse {
    private String token;
}
```

## 7. Resumen

### Configuracion de Autorizacion

|Metodo|Descripcion|
|---|---|
|`permitAll()`|Acceso publico|
|`authenticated()`|Requiere autenticacion|
|`hasRole("ADMIN")`|Requiere rol especifico|
|`hasAnyRole("A", "B")`|Requiere alguno de los roles|
|`hasAuthority("READ")`|Requiere permiso especifico|

### Anotaciones de Seguridad

|Anotacion|Uso|
|---|---|
|`@EnableWebSecurity`|Habilita Spring Security|
|`@EnableMethodSecurity`|Seguridad por metodo|
|`@PreAuthorize`|Verificar antes de ejecutar|
|`@PostAuthorize`|Verificar despues de ejecutar|
|`@Secured`|Verificar roles simples|