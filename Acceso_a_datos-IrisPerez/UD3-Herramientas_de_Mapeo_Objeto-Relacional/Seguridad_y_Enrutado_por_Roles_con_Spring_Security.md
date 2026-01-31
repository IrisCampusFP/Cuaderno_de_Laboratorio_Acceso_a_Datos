Objetivo de aprendizaje

Implementar un sistema completo de autenticación y autorización con Spring Security, gestionando el acceso a las diferentes páginas de la aplicación según los roles asignados a cada usuario.

## Visión General

La seguridad es un aspecto crítico en cualquier aplicación web. **Spring Security** proporciona un framework robusto para implementar autenticación (¿quién eres?) y autorización (¿qué puedes hacer?).

┌─────────────────────────────────────────────────────────────────────────────┐
│                          FLUJO DE AUTENTICACIÓN                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌──────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────┐   │
│   │ USUARIO  │────▶│   FILTRO     │────▶│ AUTENTICACIÓN│────▶│ ACCESO   │   │
│   │ (Login)  │     │  SECURITY    │     │   MANAGER    │     │ GRANTED  │   │
│   └──────────┘     └──────────────┘     └──────────────┘     └──────────┘   │
│        │                  │                    │                    │        │
│        │                  ▼                    ▼                    ▼        │
│        │           ┌──────────────┐     ┌──────────────┐     ┌──────────┐   │
│        │           │   SECURITY   │     │   USER       │     │  PÁGINA  │   │
│        │           │   CONTEXT    │     │  DETAILS     │     │ PROTEGIDA│   │
│        │           └──────────────┘     │  SERVICE     │     └──────────┘   │
│        │                               └──────────────┘                      │
│        │                                      │                              │
│        │                                      ▼                              │
│        │                               ┌──────────────┐                      │
│        └──────────────────────────────▶│   BASE DE    │                      │
│                 Credenciales           │    DATOS     │                      │
│                                        └──────────────┘                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘

**Conceptos clave:**

|Término|Descripción|
|---|---|
|**Authentication**|Verificar identidad del usuario|
|**Authorization**|Verificar permisos del usuario|
|**Principal**|Usuario autenticado actualmente|
|**GrantedAuthority**|Permiso/rol del usuario|
|**SecurityContext**|Almacén del usuario actual|

**Roles típicos:**

|Rol|Permisos|
|---|---|
|**ROLE_ADMIN**|Acceso total, gestión usuarios|
|**ROLE_USER**|Operaciones básicas CRUD|
|**ROLE_VIEWER**|Solo lectura|
|**ANONYMOUS**|Páginas públicas|

## 1. Configuración del Proyecto

### 1.1. Dependencias Maven

<!-- ═══════════════════════════════════════════════════════════════════ -->
<!-- SPRING SECURITY: Autenticación y autorización                       -->
<!-- ═══════════════════════════════════════════════════════════════════ -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- ═══════════════════════════════════════════════════════════════════ -->
<!-- THYMELEAF EXTRAS SECURITY: Integración con plantillas               -->
<!-- Permite usar sec:authorize en las vistas                            -->
<!-- ═══════════════════════════════════════════════════════════════════ -->
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity6</artifactId>
</dependency>

### 1.2. Configuración application.properties

# ═══════════════════════════════════════════════════════════════════════════
# CONFIGURACIÓN DE SEGURIDAD
# ═══════════════════════════════════════════════════════════════════════════

# Logging de seguridad (útil para depuración)
logging.level.org.springframework.security=DEBUG

# Desactivar la página de error por defecto de Spring Security
server.error.whitelabel.enabled=false

## 2. Modelo de Datos: Usuarios y Roles

### 2.1. Entidad Usuario

package es.campusfp.webapp.model;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;
import java.time.LocalDateTime;
import java.util.HashSet;
import java.util.Set;

/**
 * Entidad que representa un usuario del sistema.
 * Implementa los campos necesarios para Spring Security.
 */
@Entity
@Table(name = "usuarios")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Usuario {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "El nombre de usuario es obligatorio")
    @Size(min = 3, max = 50, message = "El usuario debe tener entre 3 y 50 caracteres")
    @Column(nullable = false, unique = true, length = 50)
    private String username;

    @NotBlank(message = "El email es obligatorio")
    @Email(message = "El email no es válido")
    @Column(nullable = false, unique = true, length = 100)
    private String email;

    @NotBlank(message = "La contraseña es obligatoria")
    @Column(nullable = false, length = 255)
    private String password;  // Almacenada con BCrypt

    @Column(nullable = false, length = 100)
    private String nombre;

    @Column(length = 100)
    private String apellidos;

    // ═══════════════════════════════════════════════════════════════════
    // CAMPOS DE ESTADO DE LA CUENTA
    // ═══════════════════════════════════════════════════════════════════

    @Column(nullable = false)
    @Builder.Default
    private Boolean activo = true;           // Cuenta habilitada

    @Column(nullable = false)
    @Builder.Default
    private Boolean cuentaBloqueada = false; // Cuenta bloqueada por intentos fallidos

    @Column(nullable = false)
    @Builder.Default
    private Boolean credencialesExpiradas = false;  // Contraseña expirada

    // ═══════════════════════════════════════════════════════════════════
    // RELACIÓN CON ROLES (Many-to-Many)
    // ═══════════════════════════════════════════════════════════════════

    @ManyToMany(fetch = FetchType.EAGER)  // EAGER para cargar roles con el usuario
    @JoinTable(
        name = "usuarios_roles",
        joinColumns = @JoinColumn(name = "usuario_id"),
        inverseJoinColumns = @JoinColumn(name = "rol_id")
    )
    @Builder.Default
    private Set<Rol> roles = new HashSet<>();

    // ═══════════════════════════════════════════════════════════════════
    // CAMPOS DE AUDITORÍA
    // ═══════════════════════════════════════════════════════════════════

    @Column(name = "fecha_creacion", updatable = false)
    private LocalDateTime fechaCreacion;

    @Column(name = "ultimo_acceso")
    private LocalDateTime ultimoAcceso;

    @Column(name = "intentos_fallidos")
    @Builder.Default
    private Integer intentosFallidos = 0;

    @PrePersist
    protected void onCreate() {
        this.fechaCreacion = LocalDateTime.now();
    }

    // ═══════════════════════════════════════════════════════════════════
    // MÉTODOS DE UTILIDAD
    // ═══════════════════════════════════════════════════════════════════

    /**
     * Añade un rol al usuario.
     */
    public void addRol(Rol rol) {
        this.roles.add(rol);
    }

    /**
     * Elimina un rol del usuario.
     */
    public void removeRol(Rol rol) {
        this.roles.remove(rol);
    }

    /**
     * Verifica si el usuario tiene un rol específico.
     */
    public boolean tieneRol(String nombreRol) {
        return this.roles.stream()
            .anyMatch(rol -> rol.getNombre().equals(nombreRol));
    }

    /**
     * Verifica si el usuario es administrador.
     */
    public boolean esAdmin() {
        return tieneRol("ROLE_ADMIN");
    }
}

### 2.2. Entidad Rol

package es.campusfp.webapp.model;

import jakarta.persistence.*;
import lombok.*;

/**
 * Entidad que representa un rol/permiso en el sistema.
 * Los roles deben comenzar con "ROLE_" por convención de Spring Security.
 */
@Entity
@Table(name = "roles")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Rol {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 50)
    private String nombre;  // Ej: ROLE_ADMIN, ROLE_USER, ROLE_VIEWER

    @Column(length = 200)
    private String descripcion;

    // ═══════════════════════════════════════════════════════════════════
    // CONSTANTES DE ROLES PREDEFINIDOS
    // ═══════════════════════════════════════════════════════════════════

    public static final String ADMIN = "ROLE_ADMIN";
    public static final String USER = "ROLE_USER";
    public static final String VIEWER = "ROLE_VIEWER";
}

## 3. Repositorios

### 3.1. UsuarioRepository

package es.campusfp.webapp.repository;

import es.campusfp.webapp.model.Usuario;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;
import java.time.LocalDateTime;
import java.util.Optional;

@Repository
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {

    // Buscar por username (para login)
    Optional<Usuario> findByUsername(String username);

    // Buscar por email
    Optional<Usuario> findByEmail(String email);

    // Verificar si existe username
    boolean existsByUsername(String username);

    // Verificar si existe email
    boolean existsByEmail(String email);

    // Actualizar último acceso
    @Modifying
    @Query("UPDATE Usuario u SET u.ultimoAcceso = :fecha WHERE u.username = :username")
    void actualizarUltimoAcceso(String username, LocalDateTime fecha);

    // Incrementar intentos fallidos
    @Modifying
    @Query("UPDATE Usuario u SET u.intentosFallidos = u.intentosFallidos + 1 WHERE u.username = :username")
    void incrementarIntentosFallidos(String username);

    // Resetear intentos fallidos
    @Modifying
    @Query("UPDATE Usuario u SET u.intentosFallidos = 0 WHERE u.username = :username")
    void resetearIntentosFallidos(String username);

    // Bloquear cuenta
    @Modifying
    @Query("UPDATE Usuario u SET u.cuentaBloqueada = true WHERE u.username = :username")
    void bloquearCuenta(String username);
}

### 3.2. RolRepository

package es.campusfp.webapp.repository;

import es.campusfp.webapp.model.Rol;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.Optional;

@Repository
public interface RolRepository extends JpaRepository<Rol, Long> {

    Optional<Rol> findByNombre(String nombre);

    boolean existsByNombre(String nombre);
}

## 4. Servicio UserDetailsService

package es.campusfp.webapp.service;

import es.campusfp.webapp.model.Usuario;
import es.campusfp.webapp.repository.UsuarioRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.*;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.stream.Collectors;

/**
 * Servicio que carga los datos del usuario para Spring Security.
 * Implementa UserDetailsService, interfaz requerida por Spring Security.
 */
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UsuarioRepository usuarioRepository;

    /**
     * Carga un usuario por su username.
     * Este método es llamado automáticamente por Spring Security durante el login.
     *
     * @param username El nombre de usuario
     * @return UserDetails con la información del usuario
     * @throws UsernameNotFoundException Si el usuario no existe
     */
    @Override
    @Transactional(readOnly = true)
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // Buscar usuario en base de datos
        Usuario usuario = usuarioRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException(
                "Usuario no encontrado: " + username));

        // Convertir roles a GrantedAuthority (formato que entiende Spring Security)
        var authorities = usuario.getRoles().stream()
            .map(rol -> new SimpleGrantedAuthority(rol.getNombre()))
            .collect(Collectors.toList());

        // Construir y devolver UserDetails
        return User.builder()
            .username(usuario.getUsername())
            .password(usuario.getPassword())  // Ya está encriptada con BCrypt
            .authorities(authorities)
            .accountExpired(false)
            .accountLocked(usuario.getCuentaBloqueada())
            .credentialsExpired(usuario.getCredencialesExpiradas())
            .disabled(!usuario.getActivo())
            .build();
    }
}

UserDetails vs Usuario

**UserDetails** es la interfaz de Spring Security que representa al usuario autenticado. **Usuario** es nuestra entidad JPA que almacenamos en base de datos.

El servicio `CustomUserDetailsService` actúa como puente entre ambos.

## 5. Configuración de Seguridad

### 5.1. SecurityConfig

package es.campusfp.webapp.config;

import es.campusfp.webapp.service.CustomUserDetailsService;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;

/**
 * Configuración principal de Spring Security.
 */
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)  // Habilita @PreAuthorize y @PostAuthorize
@RequiredArgsConstructor
public class SecurityConfig {

    private final CustomUserDetailsService userDetailsService;

    // ═══════════════════════════════════════════════════════════════════════
    // ENCODER DE CONTRASEÑAS (BCrypt)
    // ═══════════════════════════════════════════════════════════════════════
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    // ═══════════════════════════════════════════════════════════════════════
    // PROVEEDOR DE AUTENTICACIÓN
    // ═══════════════════════════════════════════════════════════════════════
    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    // ═══════════════════════════════════════════════════════════════════════
    // AUTHENTICATION MANAGER
    // ═══════════════════════════════════════════════════════════════════════
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }

    // ═══════════════════════════════════════════════════════════════════════
    // CADENA DE FILTROS DE SEGURIDAD
    // Aquí se define QUÉ rutas están protegidas y QUIÉN puede acceder
    // ═══════════════════════════════════════════════════════════════════════
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // ───────────────────────────────────────────────────────────────
            // CONFIGURACIÓN DE AUTORIZACIÓN POR RUTAS
            // ───────────────────────────────────────────────────────────────
            .authorizeHttpRequests(auth -> auth
                // Recursos estáticos: acceso público
                .requestMatchers("/css/**", "/js/**", "/img/**", "/webjars/**").permitAll()

                // Páginas públicas
                .requestMatchers("/", "/login", "/registro", "/error").permitAll()

                // ═══════════════════════════════════════════════════════════
                // RUTAS POR ROL
                // ═══════════════════════════════════════════════════════════

                // ADMIN: Acceso total a administración
                .requestMatchers("/admin/**").hasRole("ADMIN")

                // ADMIN: Gestión de usuarios
                .requestMatchers("/usuarios/**").hasRole("ADMIN")

                // ADMIN o USER: CRUD de productos
                .requestMatchers("/productos/nuevo", "/productos/editar/**",
                                 "/productos/guardar", "/productos/eliminar/**")
                    .hasAnyRole("ADMIN", "USER")

                // VIEWER, USER o ADMIN: Ver productos (solo lectura)
                .requestMatchers("/productos", "/productos/{id}").hasAnyRole("ADMIN", "USER", "VIEWER")

                // Cualquier otra ruta requiere autenticación
                .anyRequest().authenticated()
            )

            // ───────────────────────────────────────────────────────────────
            // CONFIGURACIÓN DEL FORMULARIO DE LOGIN
            // ───────────────────────────────────────────────────────────────
            .formLogin(form -> form
                .loginPage("/login")                    // URL de la página de login
                .loginProcessingUrl("/login")           // URL que procesa el login (POST)
                .usernameParameter("username")          // Nombre del campo usuario
                .passwordParameter("password")          // Nombre del campo contraseña
                .defaultSuccessUrl("/productos", true)  // Redirección tras login exitoso
                .failureUrl("/login?error=true")        // Redirección si falla
                .permitAll()
            )

            // ───────────────────────────────────────────────────────────────
            // CONFIGURACIÓN DEL LOGOUT
            // ───────────────────────────────────────────────────────────────
            .logout(logout -> logout
                .logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
                .logoutSuccessUrl("/login?logout=true")
                .invalidateHttpSession(true)            // Invalida la sesión
                .deleteCookies("JSESSIONID")            // Elimina cookies
                .clearAuthentication(true)
                .permitAll()
            )

            // ───────────────────────────────────────────────────────────────
            // MANEJO DE EXCEPCIONES
            // ───────────────────────────────────────────────────────────────
            .exceptionHandling(ex -> ex
                .accessDeniedPage("/acceso-denegado")   // Página para 403 Forbidden
            )

            // ───────────────────────────────────────────────────────────────
            // REMEMBER ME (Recordar sesión)
            // ───────────────────────────────────────────────────────────────
            .rememberMe(remember -> remember
                .key("mi-clave-secreta-unica")          // Clave para encriptar token
                .tokenValiditySeconds(86400 * 7)        // 7 días
                .userDetailsService(userDetailsService)
            );

        return http.build();
    }
}

hasRole vs hasAuthority

- `hasRole("ADMIN")` → Spring añade automáticamente el prefijo «ROLE_», busca «ROLE_ADMIN»
    
- `hasAuthority("ROLE_ADMIN")` → Busca exactamente «ROLE_ADMIN»
    

Por convención, siempre guardamos los roles con el prefijo «ROLE_» en base de datos.

## 6. Tabla de Permisos por Rol

Tabla 4 Matriz de permisos
|Ruta / Acción|ADMIN|USER|VIEWER|Anónimo|
|---|---|---|---|---|
|`/` (Inicio)|✅|✅|✅|✅|
|`/login`, `/registro`|✅|✅|✅|✅|
|`/productos` (Listar)|✅|✅|✅|❌|
|`/productos/{id}` (Ver)|✅|✅|✅|❌|
|`/productos/nuevo` (Crear)|✅|✅|❌|❌|
|`/productos/editar/{id}` (Editar)|✅|✅|❌|❌|
|`/productos/eliminar/{id}` (Eliminar)|✅|✅|❌|❌|
|`/usuarios/**` (Gestión usuarios)|✅|❌|❌|❌|
|`/admin/**` (Panel admin)|✅|❌|❌|❌|

## 7. Controladores de Autenticación

### 7.1. AuthController

package es.campusfp.webapp.controller;

import es.campusfp.webapp.model.Rol;
import es.campusfp.webapp.model.Usuario;
import es.campusfp.webapp.repository.RolRepository;
import es.campusfp.webapp.repository.UsuarioRepository;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

/**
 * Controlador para autenticación y registro de usuarios.
 */
@Controller
@RequiredArgsConstructor
public class AuthController {

    private final UsuarioRepository usuarioRepository;
    private final RolRepository rolRepository;
    private final PasswordEncoder passwordEncoder;

    // ═══════════════════════════════════════════════════════════════════════
    // PÁGINA DE LOGIN
    // ═══════════════════════════════════════════════════════════════════════
    @GetMapping("/login")
    public String login(
            @RequestParam(value = "error", required = false) String error,
            @RequestParam(value = "logout", required = false) String logout,
            Model model) {

        if (error != null) {
            model.addAttribute("error", "Usuario o contraseña incorrectos");
        }

        if (logout != null) {
            model.addAttribute("mensaje", "Has cerrado sesión correctamente");
        }

        return "auth/login";
    }

    // ═══════════════════════════════════════════════════════════════════════
    // FORMULARIO DE REGISTRO
    // ═══════════════════════════════════════════════════════════════════════
    @GetMapping("/registro")
    public String mostrarRegistro(Model model) {
        model.addAttribute("usuario", new Usuario());
        return "auth/registro";
    }

    // ═══════════════════════════════════════════════════════════════════════
    // PROCESAR REGISTRO
    // ═══════════════════════════════════════════════════════════════════════
    @PostMapping("/registro")
    public String registrar(
            @Valid @ModelAttribute("usuario") Usuario usuario,
            BindingResult result,
            @RequestParam("confirmarPassword") String confirmarPassword,
            Model model,
            RedirectAttributes redirectAttributes) {

        // Validar que las contraseñas coincidan
        if (!usuario.getPassword().equals(confirmarPassword)) {
            result.rejectValue("password", "error.usuario", "Las contraseñas no coinciden");
        }

        // Validar username único
        if (usuarioRepository.existsByUsername(usuario.getUsername())) {
            result.rejectValue("username", "error.usuario", "El nombre de usuario ya existe");
        }

        // Validar email único
        if (usuarioRepository.existsByEmail(usuario.getEmail())) {
            result.rejectValue("email", "error.usuario", "El email ya está registrado");
        }

        // Si hay errores, volver al formulario
        if (result.hasErrors()) {
            return "auth/registro";
        }

        // Encriptar contraseña
        usuario.setPassword(passwordEncoder.encode(usuario.getPassword()));

        // Asignar rol por defecto (ROLE_USER)
        Rol rolUser = rolRepository.findByNombre(Rol.USER)
            .orElseThrow(() -> new RuntimeException("Rol USER no encontrado"));
        usuario.addRol(rolUser);

        // Guardar usuario
        usuarioRepository.save(usuario);

        redirectAttributes.addFlashAttribute("mensaje",
            "Registro exitoso. Ya puedes iniciar sesión.");
        return "redirect:/login";
    }

    // ═══════════════════════════════════════════════════════════════════════
    // PÁGINA DE ACCESO DENEGADO
    // ═══════════════════════════════════════════════════════════════════════
    @GetMapping("/acceso-denegado")
    public String accesoDenegado() {
        return "auth/acceso-denegado";
    }
}

### 7.2. Controlador de Usuarios (Solo Admin)

package es.campusfp.webapp.controller;

import es.campusfp.webapp.model.*;
import es.campusfp.webapp.repository.*;
import lombok.RequiredArgsConstructor;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;
import java.util.List;

/**
 * Controlador para gestión de usuarios.
 * Solo accesible para administradores.
 */
@Controller
@RequestMapping("/usuarios")
@RequiredArgsConstructor
@PreAuthorize("hasRole('ADMIN')")  // Protege TODO el controlador
public class UsuarioController {

    private final UsuarioRepository usuarioRepository;
    private final RolRepository rolRepository;
    private final PasswordEncoder passwordEncoder;

    // ═══════════════════════════════════════════════════════════════════════
    // LISTAR USUARIOS
    // ═══════════════════════════════════════════════════════════════════════
    @GetMapping
    public String listar(Model model) {
        List<Usuario> usuarios = usuarioRepository.findAll();
        model.addAttribute("usuarios", usuarios);
        return "usuarios/lista";
    }

    // ═══════════════════════════════════════════════════════════════════════
    // EDITAR ROLES DE USUARIO
    // ═══════════════════════════════════════════════════════════════════════
    @GetMapping("/editar/{id}")
    public String editarRoles(@PathVariable Long id, Model model) {
        Usuario usuario = usuarioRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Usuario no encontrado"));

        model.addAttribute("usuario", usuario);
        model.addAttribute("todosLosRoles", rolRepository.findAll());
        return "usuarios/editar-roles";
    }

    @PostMapping("/actualizar-roles/{id}")
    public String actualizarRoles(
            @PathVariable Long id,
            @RequestParam(required = false) List<Long> roles,
            RedirectAttributes redirectAttributes) {

        Usuario usuario = usuarioRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Usuario no encontrado"));

        // Limpiar roles actuales
        usuario.getRoles().clear();

        // Asignar nuevos roles
        if (roles != null && !roles.isEmpty()) {
            roles.forEach(rolId -> {
                rolRepository.findById(rolId).ifPresent(usuario::addRol);
            });
        }

        usuarioRepository.save(usuario);

        redirectAttributes.addFlashAttribute("mensaje",
            "Roles actualizados correctamente");
        return "redirect:/usuarios";
    }

    // ═══════════════════════════════════════════════════════════════════════
    // ACTIVAR/DESACTIVAR USUARIO
    // ═══════════════════════════════════════════════════════════════════════
    @PostMapping("/toggle-activo/{id}")
    public String toggleActivo(@PathVariable Long id, RedirectAttributes redirectAttributes) {
        Usuario usuario = usuarioRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Usuario no encontrado"));

        usuario.setActivo(!usuario.getActivo());
        usuarioRepository.save(usuario);

        String estado = usuario.getActivo() ? "activado" : "desactivado";
        redirectAttributes.addFlashAttribute("mensaje",
            "Usuario " + estado + " correctamente");
        return "redirect:/usuarios";
    }

    // ═══════════════════════════════════════════════════════════════════════
    // DESBLOQUEAR CUENTA
    // ═══════════════════════════════════════════════════════════════════════
    @PostMapping("/desbloquear/{id}")
    public String desbloquear(@PathVariable Long id, RedirectAttributes redirectAttributes) {
        Usuario usuario = usuarioRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Usuario no encontrado"));

        usuario.setCuentaBloqueada(false);
        usuario.setIntentosFallidos(0);
        usuarioRepository.save(usuario);

        redirectAttributes.addFlashAttribute("mensaje",
            "Cuenta desbloqueada correctamente");
        return "redirect:/usuarios";
    }
}

## 8. Anotaciones de Seguridad en Métodos

### 8.1. Uso de @PreAuthorize y @PostAuthorize

package es.campusfp.webapp.service;

import es.campusfp.webapp.model.Producto;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.access.prepost.PostAuthorize;
import org.springframework.stereotype.Service;

/**
 * Servicio con seguridad a nivel de método.
 */
@Service
public class ProductoServiceSeguro {

    // ═══════════════════════════════════════════════════════════════════════
    // @PreAuthorize: Verifica ANTES de ejecutar el método
    // ═══════════════════════════════════════════════════════════════════════

    /**
     * Solo ADMIN puede eliminar productos.
     */
    @PreAuthorize("hasRole('ADMIN')")
    public void eliminarProducto(Long id) {
        // Lógica de eliminación
    }

    /**
     * ADMIN o USER pueden crear productos.
     */
    @PreAuthorize("hasAnyRole('ADMIN', 'USER')")
    public Producto crearProducto(Producto producto) {
        // Lógica de creación
        return producto;
    }

    /**
     * Cualquier usuario autenticado puede listar.
     */
    @PreAuthorize("isAuthenticated()")
    public java.util.List<Producto> listarTodos() {
        // Lógica de listado
        return null;
    }

    /**
     * Verificar que el usuario actual es el propietario.
     * El #username es el parámetro del método.
     */
    @PreAuthorize("#username == authentication.principal.username or hasRole('ADMIN')")
    public void actualizarPerfil(String username, Object datos) {
        // Solo puede actualizar su propio perfil (o admin puede actualizar cualquiera)
    }

    // ═══════════════════════════════════════════════════════════════════════
    // @PostAuthorize: Verifica DESPUÉS de ejecutar el método
    // ═══════════════════════════════════════════════════════════════════════

    /**
     * Verifica después de obtener el producto que el usuario puede verlo.
     * returnObject es el valor devuelto por el método.
     */
    @PostAuthorize("returnObject.activo == true or hasRole('ADMIN')")
    public Producto obtenerProducto(Long id) {
        // Solo devuelve productos activos, a menos que sea admin
        return null;
    }
}

### 8.2. Expresiones SpEL Disponibles

Tabla 5 Expresiones de seguridad
|Expresión|Descripción|
|---|---|
|`hasRole('ADMIN')`|Usuario tiene rol ROLE_ADMIN|
|`hasAnyRole('ADMIN', 'USER')`|Usuario tiene alguno de los roles|
|`hasAuthority('PERMISO')`|Usuario tiene autoridad específica|
|`isAuthenticated()`|Usuario está autenticado|
|`isAnonymous()`|Usuario es anónimo|
|`isFullyAuthenticated()`|Autenticado (no por remember-me)|
|`principal.username`|Nombre del usuario actual|
|`authentication.principal`|Objeto UserDetails completo|
|`#paramName`|Referencia a parámetro del método|
|`returnObject`|Objeto devuelto (solo @PostAuthorize)|

## 9. Plantillas Thymeleaf con Seguridad

### 9.1. Página de Login (auth/login.html)

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Iniciar Sesión</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.1/font/bootstrap-icons.css" rel="stylesheet">
    <style>
        body {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .login-card {
            background: white;
            border-radius: 15px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.2);
            overflow: hidden;
            max-width: 400px;
            width: 100%;
        }
        .login-header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 30px;
            text-align: center;
        }
        .login-header i {
            font-size: 3rem;
            margin-bottom: 10px;
        }
        .login-body {
            padding: 30px;
        }
        .form-control:focus {
            border-color: #667eea;
            box-shadow: 0 0 0 0.2rem rgba(102, 126, 234, 0.25);
        }
        .btn-login {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            border: none;
            padding: 12px;
        }
        .btn-login:hover {
            background: linear-gradient(135deg, #5a6fd6 0%, #6a4190 100%);
        }
    </style>
</head>
<body>
    <div class="login-card">
        <div class="login-header">
            <i class="bi bi-shield-lock"></i>
            <h3>Iniciar Sesión</h3>
        </div>

        <div class="login-body">
            <!-- Mensaje de error -->
            <div th:if="${error}" class="alert alert-danger" role="alert">
                <i class="bi bi-exclamation-triangle"></i>
                <span th:text="${error}">Error</span>
            </div>

            <!-- Mensaje de logout exitoso -->
            <div th:if="${mensaje}" class="alert alert-success" role="alert">
                <i class="bi bi-check-circle"></i>
                <span th:text="${mensaje}">Mensaje</span>
            </div>

            <!-- Formulario de login -->
            <form th:action="@{/login}" method="post">
                <div class="mb-3">
                    <label for="username" class="form-label">
                        <i class="bi bi-person"></i> Usuario
                    </label>
                    <input type="text"
                           class="form-control"
                           id="username"
                           name="username"
                           placeholder="Tu nombre de usuario"
                           required
                           autofocus>
                </div>

                <div class="mb-3">
                    <label for="password" class="form-label">
                        <i class="bi bi-key"></i> Contraseña
                    </label>
                    <input type="password"
                           class="form-control"
                           id="password"
                           name="password"
                           placeholder="Tu contraseña"
                           required>
                </div>

                <div class="mb-3 form-check">
                    <input type="checkbox"
                           class="form-check-input"
                           id="remember-me"
                           name="remember-me">
                    <label class="form-check-label" for="remember-me">
                        Recordarme
                    </label>
                </div>

                <button type="submit" class="btn btn-primary btn-login w-100">
                    <i class="bi bi-box-arrow-in-right"></i> Entrar
                </button>
            </form>

            <hr>

            <p class="text-center mb-0">
                ¿No tienes cuenta?
                <a th:href="@{/registro}">Regístrate aquí</a>
            </p>
        </div>
    </div>
</body>
</html>

### 9.2. Menú con Opciones según Rol

<!-- Fragmento de navegación con seguridad -->
<nav class="navbar navbar-expand-lg navbar-dark bg-primary"
     xmlns:th="http://www.thymeleaf.org"
     xmlns:sec="http://www.thymeleaf.org/extras/spring-security">

    <div class="container">
        <a class="navbar-brand" th:href="@{/}">
            <i class="bi bi-box-seam"></i> Mi Aplicación
        </a>

        <div class="collapse navbar-collapse">
            <ul class="navbar-nav me-auto">
                <!-- Visible para todos los autenticados -->
                <li class="nav-item" sec:authorize="isAuthenticated()">
                    <a class="nav-link" th:href="@{/productos}">
                        <i class="bi bi-list"></i> Productos
                    </a>
                </li>

                <!-- Solo visible para ADMIN y USER -->
                <li class="nav-item" sec:authorize="hasAnyRole('ADMIN', 'USER')">
                    <a class="nav-link" th:href="@{/productos/nuevo}">
                        <i class="bi bi-plus-circle"></i> Nuevo Producto
                    </a>
                </li>

                <!-- Solo visible para ADMIN -->
                <li class="nav-item dropdown" sec:authorize="hasRole('ADMIN')">
                    <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">
                        <i class="bi bi-gear"></i> Administración
                    </a>
                    <ul class="dropdown-menu">
                        <li>
                            <a class="dropdown-item" th:href="@{/usuarios}">
                                <i class="bi bi-people"></i> Gestión de Usuarios
                            </a>
                        </li>
                        <li>
                            <a class="dropdown-item" th:href="@{/admin/dashboard}">
                                <i class="bi bi-speedometer2"></i> Dashboard
                            </a>
                        </li>
                    </ul>
                </li>
            </ul>

            <!-- Menú de usuario (derecha) -->
            <ul class="navbar-nav">
                <!-- Usuario NO autenticado -->
                <li class="nav-item" sec:authorize="!isAuthenticated()">
                    <a class="nav-link" th:href="@{/login}">
                        <i class="bi bi-box-arrow-in-right"></i> Iniciar Sesión
                    </a>
                </li>

                <!-- Usuario autenticado -->
                <li class="nav-item dropdown" sec:authorize="isAuthenticated()">
                    <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">
                        <i class="bi bi-person-circle"></i>
                        <!-- Mostrar nombre de usuario -->
                        <span sec:authentication="principal.username">Usuario</span>
                    </a>
                    <ul class="dropdown-menu dropdown-menu-end">
                        <li>
                            <span class="dropdown-item-text text-muted">
                                <!-- Mostrar roles del usuario -->
                                Rol: <span sec:authentication="principal.authorities">ROLE</span>
                            </span>
                        </li>
                        <li><hr class="dropdown-divider"></li>
                        <li>
                            <a class="dropdown-item" th:href="@{/perfil}">
                                <i class="bi bi-person"></i> Mi Perfil
                            </a>
                        </li>
                        <li>
                            <!-- Formulario de logout (debe ser POST) -->
                            <form th:action="@{/logout}" method="post" class="d-inline">
                                <button type="submit" class="dropdown-item text-danger">
                                    <i class="bi bi-box-arrow-right"></i> Cerrar Sesión
                                </button>
                            </form>
                        </li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</nav>

### 9.3. Atributos sec: Disponibles

Tabla 6 Atributos Thymeleaf Security
|Atributo|Uso|
|---|---|
|`sec:authorize="isAuthenticated()"`|Mostrar si está autenticado|
|`sec:authorize="!isAuthenticated()"`|Mostrar si NO está autenticado|
|`sec:authorize="hasRole('ADMIN')"`|Mostrar solo para ADMIN|
|`sec:authorize="hasAnyRole('A','B')"`|Mostrar para roles A o B|
|`sec:authentication="principal.username"`|Mostrar nombre de usuario|
|`sec:authentication="principal.authorities"`|Mostrar roles/autoridades|

## 10. Script SQL para Usuarios y Roles

-- ═══════════════════════════════════════════════════════════════════════════
-- CREAR TABLAS DE SEGURIDAD
-- ═══════════════════════════════════════════════════════════════════════════

-- Tabla de roles
CREATE TABLE IF NOT EXISTS roles (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE,
    descripcion VARCHAR(200)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Tabla de usuarios
CREATE TABLE IF NOT EXISTS usuarios (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    apellidos VARCHAR(100),
    activo BOOLEAN NOT NULL DEFAULT TRUE,
    cuenta_bloqueada BOOLEAN NOT NULL DEFAULT FALSE,
    credenciales_expiradas BOOLEAN NOT NULL DEFAULT FALSE,
    intentos_fallidos INT DEFAULT 0,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ultimo_acceso TIMESTAMP NULL,

    INDEX idx_username (username),
    INDEX idx_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Tabla intermedia usuarios_roles
CREATE TABLE IF NOT EXISTS usuarios_roles (
    usuario_id BIGINT NOT NULL,
    rol_id BIGINT NOT NULL,
    PRIMARY KEY (usuario_id, rol_id),
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE CASCADE,
    FOREIGN KEY (rol_id) REFERENCES roles(id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- ═══════════════════════════════════════════════════════════════════════════
-- INSERTAR ROLES PREDEFINIDOS
-- ═══════════════════════════════════════════════════════════════════════════
INSERT INTO roles (nombre, descripcion) VALUES
('ROLE_ADMIN', 'Administrador con acceso total'),
('ROLE_USER', 'Usuario estándar con permisos CRUD'),
('ROLE_VIEWER', 'Usuario con permisos de solo lectura');

-- ═══════════════════════════════════════════════════════════════════════════
-- INSERTAR USUARIOS DE PRUEBA
-- Contraseñas encriptadas con BCrypt (todas son "password123")
-- ═══════════════════════════════════════════════════════════════════════════
INSERT INTO usuarios (username, email, password, nombre, apellidos) VALUES
('admin', 'admin@campusfp.es',
 '$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZRGdjGj/nMTCqMhcI3qKz6F.m9r6O',
 'Administrador', 'Sistema'),
('usuario', 'usuario@campusfp.es',
 '$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZRGdjGj/nMTCqMhcI3qKz6F.m9r6O',
 'Usuario', 'Normal'),
('lector', 'lector@campusfp.es',
 '$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZRGdjGj/nMTCqMhcI3qKz6F.m9r6O',
 'Usuario', 'Lectura');

-- ═══════════════════════════════════════════════════════════════════════════
-- ASIGNAR ROLES A USUARIOS
-- ═══════════════════════════════════════════════════════════════════════════
-- Admin tiene rol ADMIN
INSERT INTO usuarios_roles (usuario_id, rol_id)
SELECT u.id, r.id FROM usuarios u, roles r
WHERE u.username = 'admin' AND r.nombre = 'ROLE_ADMIN';

-- Usuario tiene rol USER
INSERT INTO usuarios_roles (usuario_id, rol_id)
SELECT u.id, r.id FROM usuarios u, roles r
WHERE u.username = 'usuario' AND r.nombre = 'ROLE_USER';

-- Lector tiene rol VIEWER
INSERT INTO usuarios_roles (usuario_id, rol_id)
SELECT u.id, r.id FROM usuarios u, roles r
WHERE u.username = 'lector' AND r.nombre = 'ROLE_VIEWER';

## 11. Diagrama de Flujo de Autenticación

┌─────────────────────────────────────────────────────────────────────────────┐
│                     FLUJO COMPLETO DE AUTENTICACIÓN                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. USUARIO ACCEDE A /productos                                              │
│     │                                                                        │
│     ▼                                                                        │
│  2. SecurityFilterChain intercepta la petición                               │
│     │                                                                        │
│     ├─── ¿Está autenticado? ───▶ NO ───▶ Redirige a /login                  │
│     │                                                                        │
│     ▼                                                                        │
│  3. Usuario introduce credenciales en /login                                 │
│     │                                                                        │
│     ▼                                                                        │
│  4. UsernamePasswordAuthenticationFilter captura POST /login                 │
│     │                                                                        │
│     ▼                                                                        │
│  5. DaoAuthenticationProvider llama a CustomUserDetailsService               │
│     │                                                                        │
│     ▼                                                                        │
│  6. loadUserByUsername(username) busca en BBDD                               │
│     │                                                                        │
│     ├─── ¿Existe? ───▶ NO ───▶ UsernameNotFoundException                    │
│     │                                                                        │
│     ▼                                                                        │
│  7. BCryptPasswordEncoder.matches(password, hash)                            │
│     │                                                                        │
│     ├─── ¿Coincide? ───▶ NO ───▶ BadCredentialsException                    │
│     │                                                                        │
│     ▼                                                                        │
│  8. Crea Authentication con UserDetails y authorities                        │
│     │                                                                        │
│     ▼                                                                        │
│  9. SecurityContext almacena Authentication                                  │
│     │                                                                        │
│     ▼                                                                        │
│ 10. Redirige a /productos (defaultSuccessUrl)                                │
│     │                                                                        │
│     ▼                                                                        │
│ 11. authorizeHttpRequests verifica rol para /productos                       │
│     │                                                                        │
│     ├─── ¿Tiene ROLE_USER, ROLE_ADMIN o ROLE_VIEWER? ───▶ SÍ ───▶ ACCESO   │
│     │                                                                        │
│     └─── NO ───▶ AccessDeniedException ───▶ /acceso-denegado                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘

## 12. Ejercicios Propuestos

Ejercicio 1: Implementar bloqueo por intentos fallidos

Bloquea automáticamente la cuenta después de 3 intentos fallidos de login.

**Pistas:**

- Crea un `AuthenticationFailureHandler` personalizado
    
- Incrementa `intentosFallidos` en cada fallo
    
- Si `intentosFallidos >= 3`, establece `cuentaBloqueada = true`
    
- En `CustomUserDetailsService`, devuelve `accountLocked = true`
    

@Component
public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {

    @Autowired
    private UsuarioRepository usuarioRepository;

    @Override
    public void onAuthenticationFailure(HttpServletRequest request,
                                        HttpServletResponse response,
                                        AuthenticationException exception) {
        String username = request.getParameter("username");
        usuarioRepository.incrementarIntentosFallidos(username);

        // Verificar si hay que bloquear
        usuarioRepository.findByUsername(username).ifPresent(u -> {
            if (u.getIntentosFallidos() >= 3) {
                usuarioRepository.bloquearCuenta(username);
            }
        });

        response.sendRedirect("/login?error=true");
    }
}

Ejercicio 2: Añadir auditoría de accesos

Registra en una tabla cada acceso exitoso y fallido al sistema.

@Entity
@Table(name = "auditoria_accesos")
public class AuditoriaAcceso {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String ip;
    private LocalDateTime fecha;
    private Boolean exitoso;
    private String motivo;  // Null si exitoso, mensaje de error si falla
}

Ejercicio 3: Implementar «Olvidé mi contraseña»

Implementa un flujo completo de recuperación de contraseña por email.

**Pasos:**

1. Formulario para introducir email
    
2. Generar token único y guardarlo en BBDD con fecha de expiración
    
3. Enviar email con enlace `/reset-password?token=xxx`
    
4. Formulario para nueva contraseña
    
5. Validar token y actualizar contraseña
    

@Entity
public class PasswordResetToken {
    @Id @GeneratedValue
    private Long id;

    private String token;

    @OneToOne
    private Usuario usuario;

    private LocalDateTime fechaExpiracion;

    public boolean isExpirado() {
        return LocalDateTime.now().isAfter(fechaExpiracion);
    }
}

Ejercicio 4: Autenticación con JWT (API REST)

Modifica la aplicación para que también soporte autenticación JWT para endpoints REST.

**Componentes necesarios:**

- `JwtTokenProvider`: Genera y valida tokens
    
- `JwtAuthenticationFilter`: Filtro que extrae token de headers
    
- Endpoints `/api/auth/login` y `/api/auth/refresh`
    

// Ejemplo de generación de token
public String generateToken(Authentication auth) {
    return Jwts.builder()
        .setSubject(auth.getName())
        .claim("roles", auth.getAuthorities())
        .setIssuedAt(new Date())
        .setExpiration(new Date(System.currentTimeMillis() + 86400000))
        .signWith(secretKey)
        .compact();
}

## 13. Resumen de Configuración

Tabla 7 Checklist de implementación
|Elemento|Estado|
|---|---|
|Dependencia `spring-boot-starter-security`|Obligatorio|
|Dependencia `thymeleaf-extras-springsecurity6`|Para vistas Thymeleaf|
|Entidades `Usuario` y `Rol`|Modelo de datos|
|`CustomUserDetailsService`|Carga usuarios de BBDD|
|`SecurityConfig` con `SecurityFilterChain`|Configuración principal|
|`PasswordEncoder` (BCrypt)|Encriptar contraseñas|
|Página `/login` personalizada|Formulario de acceso|
|Página `/acceso-denegado`|Error 403|
|Atributos `sec:` en plantillas|Mostrar/ocultar por rol|
|`@PreAuthorize` en servicios|Seguridad a nivel de método|

## 14. Recursos Adicionales

Documentación oficial

- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
    
- [Spring Security Architecture](https://spring.io/guides/topicals/spring-security-architecture)
    
- [Thymeleaf + Spring Security](https://www.thymeleaf.org/doc/articles/springsecurity.html)
    
- [BCrypt Password Encoder](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html)