## 4.1 Arquitectura Moderna de Spring Security (2024-2025)

La arquitectura actual de Spring Security se centra en **SecurityFilterChain** como mecanismo de configuración principal, reemplazando el obsoleto `WebSecurityConfigurerAdapter`. Los componentes core incluyen **AuthenticationManager** para gestionar procesos de autenticación con soporte para múltiples proveedores, **UserDetailsService** como interfaz para cargar datos específicos del usuario desde varias fuentes, y **PasswordEncoder** para manejo seguro de almacenamiento de contraseñas con algoritmos de hashing modernos.

**Patrón de configuración actual:**

@Configuration
@EnableWebSecurity
public class ConfiguracionSeguridad {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/publico/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/estudiantes/**").hasRole("USER")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutSuccessUrl("/login?logout")
                .permitAll()
            )
            .csrf(csrf -> csrf
                .ignoringRequestMatchers("/api/**") // Deshabilitar para endpoints API
            );
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12); // Factor de fuerza 12-15 recomendado
    }
}

## 4.2 Gestión de Contraseñas y Autenticación

**Mejores prácticas de gestión de contraseñas (Directrices NIST 2024):**

Las directrices modernas requieren mínimo 8 caracteres para contraseñas elegidas por usuario y mínimo 6 caracteres para contraseñas generadas por sistema. **Las reglas de complejidad están desaconsejadas** - no se requieren caracteres especiales/números obligatorios. Se debe implementar validación de fortaleza de contraseña contra listas de contraseñas comunes/comprometidas y soporte para gestores de contraseñas permitiendo funcionalidad de copiar-pegar.

**Implementación de servicio de detalles de usuario personalizado:**

@Service
public class ServicioDetallesUsuarioPersonalizado implements UserDetailsService {
    
    @Autowired
    private RepositorioUsuario repositorioUsuario;
    
    @Override
    public UserDetails loadUserByUsername(String nombreUsuario) throws UsernameNotFoundException {
        Usuario usuario = repositorioUsuario.findByNombreUsuario(nombreUsuario)
            .orElseThrow(() -> new UsernameNotFoundException("Usuario no encontrado: " + nombreUsuario));
            
        return org.springframework.security.core.userdetails.User
            .withUsername(usuario.getNombreUsuario())
            .password(usuario.getContrasena())
            .roles(usuario.getRoles().toArray(new String[0]))
            .accountExpired(false)
            .accountLocked(false)
            .credentialsExpired(false)
            .disabled(!usuario.isHabilitado())
            .build();
    }
}

## 4.3 Implementación de Autenticación JWT

Para APIs REST sin estado, JWT (JSON Web Tokens) proporciona un mecanismo de autenticación robusto y escalable:

@Component
public class ProveedorTokenJWT {
    
    @Value("${app.jwt.secret}")
    private String claveSecretaJwt;
    
    @Value("${app.jwt.expiration}")
    private int tiempoExpiracionJwt;
    
    public String generarToken(Authentication autenticacion) {
        UserPrincipal principalUsuario = (UserPrincipal) autenticacion.getPrincipal();
        Date ahora = new Date();
        Date fechaExpiracion = new Date(ahora.getTime() + tiempoExpiracionJwt);
        
        return Jwts.builder()
            .setSubject(Long.toString(principalUsuario.getId()))
            .setIssuedAt(new Date())
            .setExpiration(fechaExpiracion)
            .signWith(SignatureAlgorithm.HS512, claveSecretaJwt)
            .compact();
    }
    
    public boolean validarToken(String tokenAuth) {
        try {
            Jwts.parser().setSigningKey(claveSecretaJwt).parseClaimsJws(tokenAuth);
            return true;
        } catch (JwtException | IllegalArgumentException ex) {
            logger.error("Token JWT inválido: {}", ex.getMessage());
        }
        return false;
    }
}