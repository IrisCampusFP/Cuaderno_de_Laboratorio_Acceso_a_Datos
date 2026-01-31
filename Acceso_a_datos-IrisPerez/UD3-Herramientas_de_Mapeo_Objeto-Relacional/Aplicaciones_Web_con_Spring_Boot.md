Objetivo de aprendizaje

Aprender a crear aplicaciones web completas con Spring Boot, comprendiendo la arquitectura MVC, el sistema de plantillas Thymeleaf y la integración con JPA para la persistencia de datos.

## Visión General

Spring Boot facilita enormemente la creación de aplicaciones web gracias a su configuración automática y su ecosistema de dependencias. En este documento aprenderemos a construir una aplicación web completa siguiendo el patrón **MVC (Model-View-Controller)**.

┌─────────────────────────────────────────────────────────────────────────┐
│                        ARQUITECTURA WEB SPRING BOOT                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌──────────┐     ┌────────────┐     ┌──────────┐     ┌────────────┐   │
│   │ CLIENTE  │────▶│ CONTROLLER │────▶│ SERVICE  │────▶│ REPOSITORY │   │
│   │ (Browser)│◀────│   (@Web)   │◀────│ (@Logic) │◀────│   (@Data)  │   │
│   └──────────┘     └────────────┘     └──────────┘     └────────────┘   │
│        │                 │                                    │          │
│        │                 │                                    │          │
│        ▼                 ▼                                    ▼          │
│   ┌──────────┐     ┌────────────┐                      ┌────────────┐   │
│   │  HTML    │     │  THYMELEAF │                      │   MySQL    │   │
│   │  CSS/JS  │     │  Templates │                      │  Database  │   │
│   └──────────┘     └────────────┘                      └────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘

**Capas de la aplicación:**

|Capa|Responsabilidad|
|---|---|
|**Controller**|Recibir peticiones HTTP|
|**Service**|Lógica de negocio|
|**Repository**|Acceso a datos|
|**Model**|Entidades JPA|
|**View**|Plantillas Thymeleaf|

**Tecnologías que usaremos:**

|Tecnología|Propósito|
|---|---|
|Spring Web|Controladores MVC|
|Thymeleaf|Motor de plantillas|
|Spring Data JPA|Persistencia|
|Lombok|Reducir boilerplate|
|Bootstrap 5|Estilos CSS|

## 1. Configuración del Proyecto

### 1.1. Dependencias Maven (pom.xml)

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Spring Boot Parent: gestiona versiones de dependencias -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.1</version>
        <relativePath/>
    </parent>

    <groupId>es.campusfp</groupId>
    <artifactId>webapp-springboot</artifactId>
    <version>1.0.0</version>
    <name>Aplicación Web Spring Boot</name>
    <description>Proyecto de ejemplo - Acceso a Datos RA3</description>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- ═══════════════════════════════════════════════════════════ -->
        <!-- SPRING WEB: Controladores MVC, manejo de peticiones HTTP    -->
        <!-- ═══════════════════════════════════════════════════════════ -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- ═══════════════════════════════════════════════════════════ -->
        <!-- THYMELEAF: Motor de plantillas HTML para vistas dinámicas   -->
        <!-- ═══════════════════════════════════════════════════════════ -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <!-- ═══════════════════════════════════════════════════════════ -->
        <!-- SPRING DATA JPA: Repositorios y persistencia con Hibernate  -->
        <!-- ═══════════════════════════════════════════════════════════ -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- ═══════════════════════════════════════════════════════════ -->
        <!-- MYSQL CONNECTOR: Driver JDBC para conectar con MySQL        -->
        <!-- ═══════════════════════════════════════════════════════════ -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- ═══════════════════════════════════════════════════════════ -->
        <!-- LOMBOK: Genera automáticamente getters, setters, etc.       -->
        <!-- ═══════════════════════════════════════════════════════════ -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- ═══════════════════════════════════════════════════════════ -->
        <!-- VALIDATION: Validación de formularios con anotaciones       -->
        <!-- ═══════════════════════════════════════════════════════════ -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- ═══════════════════════════════════════════════════════════ -->
        <!-- DEVTOOLS: Reinicio automático durante desarrollo            -->
        <!-- ═══════════════════════════════════════════════════════════ -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- ═══════════════════════════════════════════════════════════ -->
        <!-- TEST: Pruebas unitarias e integración                       -->
        <!-- ═══════════════════════════════════════════════════════════ -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

### 1.2. Configuración (application.properties)

# ═══════════════════════════════════════════════════════════════════════════
# CONFIGURACIÓN DEL SERVIDOR
# ═══════════════════════════════════════════════════════════════════════════
server.port=8080
server.servlet.context-path=/

# ═══════════════════════════════════════════════════════════════════════════
# CONEXIÓN A BASE DE DATOS
# ═══════════════════════════════════════════════════════════════════════════
spring.datasource.url=jdbc:mysql://localhost:3306/webapp_db?useSSL=false&serverTimezone=Europe/Madrid&allowPublicKeyRetrieval=true
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# ═══════════════════════════════════════════════════════════════════════════
# CONFIGURACIÓN JPA / HIBERNATE
# ═══════════════════════════════════════════════════════════════════════════
# Estrategia de creación de tablas: update = actualiza sin borrar datos
spring.jpa.hibernate.ddl-auto=update

# Dialecto de MySQL 8
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# Mostrar consultas SQL en consola (útil para desarrollo)
spring.jpa.show-sql=true

# Formatear SQL para mejor legibilidad
spring.jpa.properties.hibernate.format_sql=true

# ═══════════════════════════════════════════════════════════════════════════
# CONFIGURACIÓN THYMELEAF
# ═══════════════════════════════════════════════════════════════════════════
# Desactivar caché en desarrollo para ver cambios sin reiniciar
spring.thymeleaf.cache=false

# Ubicación de plantillas (por defecto)
spring.thymeleaf.prefix=classpath:/templates/

# Extensión de archivos
spring.thymeleaf.suffix=.html

# Codificación UTF-8
spring.thymeleaf.encoding=UTF-8

# ═══════════════════════════════════════════════════════════════════════════
# MENSAJES Y ERRORES
# ═══════════════════════════════════════════════════════════════════════════
# Incluir mensajes de error en respuestas
server.error.include-message=always
server.error.include-binding-errors=always

# ═══════════════════════════════════════════════════════════════════════════
# LOGGING
# ═══════════════════════════════════════════════════════════════════════════
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate.SQL=DEBUG

## 2. Estructura del Proyecto

src/
├── main/
│   ├── java/es/campusfp/webapp/
│   │   ├── WebAppApplication.java          # Clase principal
│   │   ├── model/                          # Entidades JPA
│   │   │   └── Producto.java
│   │   ├── repository/                     # Interfaces de acceso a datos
│   │   │   └── ProductoRepository.java
│   │   ├── service/                        # Lógica de negocio
│   │   │   └── ProductoService.java
│   │   └── controller/                     # Controladores web
│   │       └── ProductoController.java
│   └── resources/
│       ├── application.properties          # Configuración
│       ├── static/                         # Archivos estáticos (CSS, JS, imágenes)
│       │   └── css/
│       │       └── estilos.css
│       └── templates/                      # Plantillas Thymeleaf
│           ├── layout/
│           │   └── base.html               # Plantilla base
│           └── productos/
│               ├── lista.html              # Listado de productos
│               ├── formulario.html         # Crear/Editar producto
│               └── detalle.html            # Ver producto
└── test/                                   # Tests

## 3. Capa Model: Entidad JPA

### 3.1. Entidad Producto

package es.campusfp.webapp.model;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;

/**
 * Entidad JPA que representa un producto en la base de datos.
 * Utiliza Lombok para reducir código boilerplate.
 */
@Entity
@Table(name = "productos")
@Data                   // Genera getters, setters, toString, equals, hashCode
@NoArgsConstructor      // Constructor vacío (requerido por JPA)
@AllArgsConstructor     // Constructor con todos los parámetros
@Builder                // Patrón Builder para crear objetos
public class Producto {

    // ═══════════════════════════════════════════════════════════════════
    // IDENTIFICADOR
    // ═══════════════════════════════════════════════════════════════════
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // ═══════════════════════════════════════════════════════════════════
    // CAMPOS OBLIGATORIOS CON VALIDACIÓN
    // ═══════════════════════════════════════════════════════════════════

    @NotBlank(message = "El nombre es obligatorio")
    @Size(min = 2, max = 100, message = "El nombre debe tener entre 2 y 100 caracteres")
    @Column(nullable = false, length = 100)
    private String nombre;

    @Size(max = 500, message = "La descripción no puede superar 500 caracteres")
    @Column(length = 500)
    private String descripcion;

    @NotNull(message = "El precio es obligatorio")
    @DecimalMin(value = "0.01", message = "El precio debe ser mayor que 0")
    @Digits(integer = 8, fraction = 2, message = "Formato de precio inválido")
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal precio;

    @NotNull(message = "El stock es obligatorio")
    @Min(value = 0, message = "El stock no puede ser negativo")
    @Column(nullable = false)
    private Integer stock;

    @NotBlank(message = "La categoría es obligatoria")
    @Column(nullable = false, length = 50)
    private String categoria;

    // ═══════════════════════════════════════════════════════════════════
    // CAMPOS DE ESTADO
    // ═══════════════════════════════════════════════════════════════════

    @Column(nullable = false)
    private Boolean activo = true;

    // ═══════════════════════════════════════════════════════════════════
    // CAMPOS DE AUDITORÍA (se rellenan automáticamente)
    // ═══════════════════════════════════════════════════════════════════

    @Column(name = "fecha_creacion", updatable = false)
    private LocalDateTime fechaCreacion;

    @Column(name = "fecha_actualizacion")
    private LocalDateTime fechaActualizacion;

    // ═══════════════════════════════════════════════════════════════════
    // CALLBACKS JPA: Se ejecutan automáticamente
    // ═══════════════════════════════════════════════════════════════════

    @PrePersist
    protected void onCreate() {
        this.fechaCreacion = LocalDateTime.now();
        this.fechaActualizacion = LocalDateTime.now();
        if (this.activo == null) {
            this.activo = true;
        }
    }

    @PreUpdate
    protected void onUpdate() {
        this.fechaActualizacion = LocalDateTime.now();
    }
}

Anotaciones de validación

Las anotaciones `@NotBlank`, `@Size`, `@Min`, etc. validan automáticamente los datos cuando se envía un formulario. Si hay errores, Spring los captura y los podemos mostrar en la vista.

## 4. Capa Repository: Acceso a Datos

### 4.1. Interfaz ProductoRepository

package es.campusfp.webapp.repository;

import es.campusfp.webapp.model.Producto;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import java.math.BigDecimal;
import java.util.List;

/**
 * Repositorio de productos.
 * JpaRepository proporciona métodos CRUD automáticamente.
 */
@Repository
public interface ProductoRepository extends JpaRepository<Producto, Long> {

    // ═══════════════════════════════════════════════════════════════════
    // QUERY METHODS: Spring genera la consulta a partir del nombre
    // ═══════════════════════════════════════════════════════════════════

    // Buscar por categoría
    List<Producto> findByCategoria(String categoria);

    // Buscar activos
    List<Producto> findByActivoTrue();

    // Buscar por nombre (contiene, ignora mayúsculas)
    List<Producto> findByNombreContainingIgnoreCase(String nombre);

    // Buscar por rango de precio
    List<Producto> findByPrecioBetween(BigDecimal min, BigDecimal max);

    // Buscar por categoría y activo, ordenados por nombre
    List<Producto> findByCategoriaAndActivoTrueOrderByNombreAsc(String categoria);

    // Buscar con stock bajo
    List<Producto> findByStockLessThan(Integer cantidad);

    // ═══════════════════════════════════════════════════════════════════
    // CONSULTAS JPQL PERSONALIZADAS
    // ═══════════════════════════════════════════════════════════════════

    @Query("SELECT p FROM Producto p WHERE p.activo = true ORDER BY p.fechaCreacion DESC")
    List<Producto> findProductosActivosRecientes();

    @Query("SELECT DISTINCT p.categoria FROM Producto p WHERE p.activo = true ORDER BY p.categoria")
    List<String> findCategoriasActivas();

    @Query("SELECT p FROM Producto p WHERE " +
           "(:nombre IS NULL OR LOWER(p.nombre) LIKE LOWER(CONCAT('%', :nombre, '%'))) AND " +
           "(:categoria IS NULL OR p.categoria = :categoria) AND " +
           "p.activo = true")
    List<Producto> buscarConFiltros(@Param("nombre") String nombre,
                                     @Param("categoria") String categoria);
}

## 5. Capa Service: Lógica de Negocio

### 5.1. Servicio de Productos

package es.campusfp.webapp.service;

import es.campusfp.webapp.model.Producto;
import es.campusfp.webapp.repository.ProductoRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;
import java.util.Optional;

/**
 * Servicio que contiene la lógica de negocio para productos.
 * Actúa como intermediario entre Controller y Repository.
 */
@Service
@RequiredArgsConstructor  // Inyección de dependencias por constructor (Lombok)
@Transactional           // Todas las operaciones son transaccionales
public class ProductoService {

    private final ProductoRepository productoRepository;

    // ═══════════════════════════════════════════════════════════════════
    // OPERACIONES DE LECTURA
    // ═══════════════════════════════════════════════════════════════════

    /**
     * Obtiene todos los productos activos.
     */
    @Transactional(readOnly = true)
    public List<Producto> listarTodos() {
        return productoRepository.findByActivoTrue();
    }

    /**
     * Obtiene todos los productos (incluidos inactivos) - para admin.
     */
    @Transactional(readOnly = true)
    public List<Producto> listarTodosAdmin() {
        return productoRepository.findAll();
    }

    /**
     * Busca un producto por su ID.
     */
    @Transactional(readOnly = true)
    public Optional<Producto> buscarPorId(Long id) {
        return productoRepository.findById(id);
    }

    /**
     * Busca productos por nombre.
     */
    @Transactional(readOnly = true)
    public List<Producto> buscarPorNombre(String nombre) {
        return productoRepository.findByNombreContainingIgnoreCase(nombre);
    }

    /**
     * Obtiene productos por categoría.
     */
    @Transactional(readOnly = true)
    public List<Producto> listarPorCategoria(String categoria) {
        return productoRepository.findByCategoriaAndActivoTrueOrderByNombreAsc(categoria);
    }

    /**
     * Obtiene la lista de categorías disponibles.
     */
    @Transactional(readOnly = true)
    public List<String> obtenerCategorias() {
        return productoRepository.findCategoriasActivas();
    }

    /**
     * Busca con filtros opcionales.
     */
    @Transactional(readOnly = true)
    public List<Producto> buscarConFiltros(String nombre, String categoria) {
        return productoRepository.buscarConFiltros(
            nombre != null && !nombre.isBlank() ? nombre : null,
            categoria != null && !categoria.isBlank() ? categoria : null
        );
    }

    // ═══════════════════════════════════════════════════════════════════
    // OPERACIONES DE ESCRITURA
    // ═══════════════════════════════════════════════════════════════════

    /**
     * Guarda un nuevo producto o actualiza uno existente.
     */
    public Producto guardar(Producto producto) {
        return productoRepository.save(producto);
    }

    /**
     * Elimina un producto (borrado lógico).
     */
    public void eliminar(Long id) {
        productoRepository.findById(id).ifPresent(producto -> {
            producto.setActivo(false);
            productoRepository.save(producto);
        });
    }

    /**
     * Elimina un producto permanentemente (borrado físico).
     * CUIDADO: Esta operación no se puede deshacer.
     */
    public void eliminarPermanente(Long id) {
        productoRepository.deleteById(id);
    }

    /**
     * Reactiva un producto eliminado.
     */
    public void reactivar(Long id) {
        productoRepository.findById(id).ifPresent(producto -> {
            producto.setActivo(true);
            productoRepository.save(producto);
        });
    }

    /**
     * Actualiza el stock de un producto.
     */
    public void actualizarStock(Long id, Integer cantidad) {
        productoRepository.findById(id).ifPresent(producto -> {
            int nuevoStock = producto.getStock() + cantidad;
            if (nuevoStock < 0) {
                throw new IllegalArgumentException("El stock no puede ser negativo");
            }
            producto.setStock(nuevoStock);
            productoRepository.save(producto);
        });
    }
}

## 6. Capa Controller: Controladores Web

### 6.1. Controlador de Productos

package es.campusfp.webapp.controller;

import es.campusfp.webapp.model.Producto;
import es.campusfp.webapp.service.ProductoService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

/**
 * Controlador web para gestión de productos.
 * Maneja las peticiones HTTP y devuelve vistas Thymeleaf.
 */
@Controller
@RequestMapping("/productos")
@RequiredArgsConstructor
public class ProductoController {

    private final ProductoService productoService;

    // ═══════════════════════════════════════════════════════════════════
    // LISTAR PRODUCTOS
    // GET /productos
    // ═══════════════════════════════════════════════════════════════════
    @GetMapping
    public String listar(
            @RequestParam(required = false) String nombre,
            @RequestParam(required = false) String categoria,
            Model model) {

        // Obtener productos (con o sin filtros)
        var productos = productoService.buscarConFiltros(nombre, categoria);

        // Pasar datos a la vista
        model.addAttribute("productos", productos);
        model.addAttribute("categorias", productoService.obtenerCategorias());
        model.addAttribute("filtroNombre", nombre);
        model.addAttribute("filtroCategoria", categoria);

        return "productos/lista";  // → templates/productos/lista.html
    }

    // ═══════════════════════════════════════════════════════════════════
    // VER DETALLE DE PRODUCTO
    // GET /productos/{id}
    // ═══════════════════════════════════════════════════════════════════
    @GetMapping("/{id}")
    public String detalle(@PathVariable Long id, Model model) {
        return productoService.buscarPorId(id)
            .map(producto -> {
                model.addAttribute("producto", producto);
                return "productos/detalle";
            })
            .orElse("redirect:/productos");  // Si no existe, volver a lista
    }

    // ═══════════════════════════════════════════════════════════════════
    // FORMULARIO NUEVO PRODUCTO
    // GET /productos/nuevo
    // ═══════════════════════════════════════════════════════════════════
    @GetMapping("/nuevo")
    public String mostrarFormularioNuevo(Model model) {
        model.addAttribute("producto", new Producto());
        model.addAttribute("categorias", productoService.obtenerCategorias());
        model.addAttribute("titulo", "Nuevo Producto");
        model.addAttribute("accion", "/productos/guardar");
        return "productos/formulario";
    }

    // ═══════════════════════════════════════════════════════════════════
    // FORMULARIO EDITAR PRODUCTO
    // GET /productos/editar/{id}
    // ═══════════════════════════════════════════════════════════════════
    @GetMapping("/editar/{id}")
    public String mostrarFormularioEditar(@PathVariable Long id, Model model) {
        return productoService.buscarPorId(id)
            .map(producto -> {
                model.addAttribute("producto", producto);
                model.addAttribute("categorias", productoService.obtenerCategorias());
                model.addAttribute("titulo", "Editar Producto");
                model.addAttribute("accion", "/productos/guardar");
                return "productos/formulario";
            })
            .orElse("redirect:/productos");
    }

    // ═══════════════════════════════════════════════════════════════════
    // GUARDAR PRODUCTO (Crear o Actualizar)
    // POST /productos/guardar
    // ═══════════════════════════════════════════════════════════════════
    @PostMapping("/guardar")
    public String guardar(
            @Valid @ModelAttribute("producto") Producto producto,
            BindingResult result,
            Model model,
            RedirectAttributes redirectAttributes) {

        // Si hay errores de validación, volver al formulario
        if (result.hasErrors()) {
            model.addAttribute("categorias", productoService.obtenerCategorias());
            model.addAttribute("titulo", producto.getId() == null ? "Nuevo Producto" : "Editar Producto");
            model.addAttribute("accion", "/productos/guardar");
            return "productos/formulario";
        }

        // Guardar producto
        productoService.guardar(producto);

        // Mensaje flash de éxito
        redirectAttributes.addFlashAttribute("mensaje",
            producto.getId() == null ? "Producto creado correctamente" : "Producto actualizado correctamente");
        redirectAttributes.addFlashAttribute("tipoMensaje", "success");

        return "redirect:/productos";
    }

    // ═══════════════════════════════════════════════════════════════════
    // ELIMINAR PRODUCTO (Borrado lógico)
    // POST /productos/eliminar/{id}
    // ═══════════════════════════════════════════════════════════════════
    @PostMapping("/eliminar/{id}")
    public String eliminar(@PathVariable Long id, RedirectAttributes redirectAttributes) {
        productoService.eliminar(id);
        redirectAttributes.addFlashAttribute("mensaje", "Producto eliminado correctamente");
        redirectAttributes.addFlashAttribute("tipoMensaje", "warning");
        return "redirect:/productos";
    }
}

Diferencia entre @Controller y @RestController

- `@Controller`: Devuelve nombres de vistas (plantillas HTML)
    
- `@RestController`: Devuelve datos directamente (JSON/XML) para APIs REST
    

En aplicaciones web tradicionales usamos `@Controller` con Thymeleaf.

## 7. Capa View: Plantillas Thymeleaf

### 7.1. Plantilla Base (layout/base.html)

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title th:text="${titulo} ?: 'Aplicación Web'">Aplicación Web</title>

    <!-- Bootstrap 5 CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css"
          rel="stylesheet">
    <!-- Bootstrap Icons -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.1/font/bootstrap-icons.css"
          rel="stylesheet">

    <!-- Estilos personalizados -->
    <style>
        body {
            background-color: #f8f9fa;
        }
        .navbar-brand {
            font-weight: bold;
        }
        .card {
            box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
        }
        .table th {
            background-color: #f8f9fa;
        }
    </style>
</head>
<body>
    <!-- Barra de navegación -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary mb-4">
        <div class="container">
            <a class="navbar-brand" th:href="@{/}">
                <i class="bi bi-box-seam"></i> Mi Tienda
            </a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse"
                    data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav">
                    <li class="nav-item">
                        <a class="nav-link" th:href="@{/productos}">
                            <i class="bi bi-list"></i> Productos
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" th:href="@{/productos/nuevo}">
                            <i class="bi bi-plus-circle"></i> Nuevo
                        </a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>

    <!-- Contenido principal -->
    <main class="container">
        <!-- Mensajes flash -->
        <div th:if="${mensaje}"
             th:class="'alert alert-' + (${tipoMensaje} ?: 'info') + ' alert-dismissible fade show'"
             role="alert">
            <span th:text="${mensaje}">Mensaje</span>
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
        </div>

        <!-- Contenido de la página (se reemplaza en cada vista) -->
        <div th:replace="~{::contenido}">
            Contenido por defecto
        </div>
    </main>

    <!-- Footer -->
    <footer class="mt-5 py-3 bg-light text-center text-muted">
        <div class="container">
            <p class="mb-0">© 2025 - Acceso a Datos RA3 - Spring Boot</p>
        </div>
    </footer>

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>

### 7.2. Lista de Productos (productos/lista.html)

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout/base :: html(titulo='Productos')}">

<th:block th:fragment="contenido">
    <div class="card">
        <div class="card-header d-flex justify-content-between align-items-center">
            <h4 class="mb-0"><i class="bi bi-box-seam"></i> Productos</h4>
            <a th:href="@{/productos/nuevo}" class="btn btn-success">
                <i class="bi bi-plus-circle"></i> Nuevo Producto
            </a>
        </div>

        <div class="card-body">
            <!-- Filtros de búsqueda -->
            <form th:action="@{/productos}" method="get" class="row g-3 mb-4">
                <div class="col-md-4">
                    <input type="text" name="nombre" class="form-control"
                           placeholder="Buscar por nombre..."
                           th:value="${filtroNombre}">
                </div>
                <div class="col-md-3">
                    <select name="categoria" class="form-select">
                        <option value="">Todas las categorías</option>
                        <option th:each="cat : ${categorias}"
                                th:value="${cat}"
                                th:text="${cat}"
                                th:selected="${cat == filtroCategoria}">
                        </option>
                    </select>
                </div>
                <div class="col-md-2">
                    <button type="submit" class="btn btn-primary w-100">
                        <i class="bi bi-search"></i> Buscar
                    </button>
                </div>
                <div class="col-md-2">
                    <a th:href="@{/productos}" class="btn btn-outline-secondary w-100">
                        <i class="bi bi-x-circle"></i> Limpiar
                    </a>
                </div>
            </form>

            <!-- Tabla de productos -->
            <div class="table-responsive">
                <table class="table table-striped table-hover">
                    <thead>
                        <tr>
                            <th>ID</th>
                            <th>Nombre</th>
                            <th>Categoría</th>
                            <th>Precio</th>
                            <th>Stock</th>
                            <th class="text-center">Acciones</th>
                        </tr>
                    </thead>
                    <tbody>
                        <!-- Iterar sobre productos -->
                        <tr th:each="producto : ${productos}">
                            <td th:text="${producto.id}">1</td>
                            <td th:text="${producto.nombre}">Producto</td>
                            <td>
                                <span class="badge bg-secondary"
                                      th:text="${producto.categoria}">Cat</span>
                            </td>
                            <td th:text="${#numbers.formatDecimal(producto.precio, 1, 2)} + ' €'">
                                0.00 €
                            </td>
                            <td>
                                <span th:class="${producto.stock < 10 ? 'text-danger fw-bold' : ''}"
                                      th:text="${producto.stock}">0</span>
                                <i th:if="${producto.stock < 10}"
                                   class="bi bi-exclamation-triangle text-danger"
                                   title="Stock bajo"></i>
                            </td>
                            <td class="text-center">
                                <!-- Ver detalle -->
                                <a th:href="@{/productos/{id}(id=${producto.id})}"
                                   class="btn btn-sm btn-info" title="Ver">
                                    <i class="bi bi-eye"></i>
                                </a>
                                <!-- Editar -->
                                <a th:href="@{/productos/editar/{id}(id=${producto.id})}"
                                   class="btn btn-sm btn-warning" title="Editar">
                                    <i class="bi bi-pencil"></i>
                                </a>
                                <!-- Eliminar -->
                                <form th:action="@{/productos/eliminar/{id}(id=${producto.id})}"
                                      method="post" class="d-inline"
                                      onsubmit="return confirm('¿Eliminar este producto?')">
                                    <button type="submit" class="btn btn-sm btn-danger"
                                            title="Eliminar">
                                        <i class="bi bi-trash"></i>
                                    </button>
                                </form>
                            </td>
                        </tr>
                        <!-- Mensaje si no hay productos -->
                        <tr th:if="${#lists.isEmpty(productos)}">
                            <td colspan="6" class="text-center text-muted py-4">
                                <i class="bi bi-inbox fs-1"></i>
                                <p>No se encontraron productos</p>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>

            <!-- Contador de resultados -->
            <p class="text-muted" th:if="${!#lists.isEmpty(productos)}">
                Mostrando <strong th:text="${#lists.size(productos)}">0</strong> productos
            </p>
        </div>
    </div>
</th:block>
</html>

### 7.3. Formulario de Producto (productos/formulario.html)

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout/base :: html(titulo=${titulo})}">

<th:block th:fragment="contenido">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">
                    <h4 class="mb-0">
                        <i class="bi" th:classappend="${producto.id == null} ? 'bi-plus-circle' : 'bi-pencil'"></i>
                        <span th:text="${titulo}">Formulario</span>
                    </h4>
                </div>

                <div class="card-body">
                    <form th:action="${accion}" th:object="${producto}" method="post">
                        <!-- ID oculto para edición -->
                        <input type="hidden" th:field="*{id}">

                        <!-- Nombre -->
                        <div class="mb-3">
                            <label for="nombre" class="form-label">Nombre *</label>
                            <input type="text"
                                   class="form-control"
                                   th:class="${#fields.hasErrors('nombre')} ? 'form-control is-invalid' : 'form-control'"
                                   id="nombre"
                                   th:field="*{nombre}"
                                   placeholder="Nombre del producto">
                            <div class="invalid-feedback" th:if="${#fields.hasErrors('nombre')}"
                                 th:errors="*{nombre}">Error</div>
                        </div>

                        <!-- Descripción -->
                        <div class="mb-3">
                            <label for="descripcion" class="form-label">Descripción</label>
                            <textarea class="form-control"
                                      id="descripcion"
                                      th:field="*{descripcion}"
                                      rows="3"
                                      placeholder="Descripción opcional"></textarea>
                            <div class="invalid-feedback" th:if="${#fields.hasErrors('descripcion')}"
                                 th:errors="*{descripcion}">Error</div>
                        </div>

                        <div class="row">
                            <!-- Precio -->
                            <div class="col-md-4 mb-3">
                                <label for="precio" class="form-label">Precio (€) *</label>
                                <input type="number"
                                       class="form-control"
                                       th:class="${#fields.hasErrors('precio')} ? 'form-control is-invalid' : 'form-control'"
                                       id="precio"
                                       th:field="*{precio}"
                                       step="0.01" min="0.01"
                                       placeholder="0.00">
                                <div class="invalid-feedback" th:if="${#fields.hasErrors('precio')}"
                                     th:errors="*{precio}">Error</div>
                            </div>

                            <!-- Stock -->
                            <div class="col-md-4 mb-3">
                                <label for="stock" class="form-label">Stock *</label>
                                <input type="number"
                                       class="form-control"
                                       th:class="${#fields.hasErrors('stock')} ? 'form-control is-invalid' : 'form-control'"
                                       id="stock"
                                       th:field="*{stock}"
                                       min="0"
                                       placeholder="0">
                                <div class="invalid-feedback" th:if="${#fields.hasErrors('stock')}"
                                     th:errors="*{stock}">Error</div>
                            </div>

                            <!-- Categoría -->
                            <div class="col-md-4 mb-3">
                                <label for="categoria" class="form-label">Categoría *</label>
                                <input type="text"
                                       class="form-control"
                                       th:class="${#fields.hasErrors('categoria')} ? 'form-control is-invalid' : 'form-control'"
                                       id="categoria"
                                       th:field="*{categoria}"
                                       list="listaCategorias"
                                       placeholder="Categoría">
                                <datalist id="listaCategorias">
                                    <option th:each="cat : ${categorias}" th:value="${cat}">
                                </datalist>
                                <div class="invalid-feedback" th:if="${#fields.hasErrors('categoria')}"
                                     th:errors="*{categoria}">Error</div>
                            </div>
                        </div>

                        <!-- Activo (solo visible en edición) -->
                        <div class="mb-3 form-check" th:if="${producto.id != null}">
                            <input type="checkbox" class="form-check-input"
                                   id="activo" th:field="*{activo}">
                            <label class="form-check-label" for="activo">Producto activo</label>
                        </div>

                        <!-- Botones -->
                        <div class="d-flex gap-2">
                            <button type="submit" class="btn btn-primary">
                                <i class="bi bi-check-circle"></i> Guardar
                            </button>
                            <a th:href="@{/productos}" class="btn btn-secondary">
                                <i class="bi bi-x-circle"></i> Cancelar
                            </a>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</th:block>
</html>

## 8. Controlador de Inicio

### 8.1. HomeController

package es.campusfp.webapp.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

/**
 * Controlador para la página de inicio.
 */
@Controller
public class HomeController {

    @GetMapping("/")
    public String inicio() {
        return "redirect:/productos";  // Redirige a la lista de productos
    }
}

## 9. Script SQL para la Base de Datos

-- ═══════════════════════════════════════════════════════════════════════════
-- CREAR BASE DE DATOS
-- ═══════════════════════════════════════════════════════════════════════════
CREATE DATABASE IF NOT EXISTS webapp_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_spanish_ci;

USE webapp_db;

-- ═══════════════════════════════════════════════════════════════════════════
-- TABLA PRODUCTOS (se crea automáticamente con JPA, pero aquí está el SQL)
-- ═══════════════════════════════════════════════════════════════════════════
CREATE TABLE IF NOT EXISTS productos (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    descripcion VARCHAR(500),
    precio DECIMAL(10,2) NOT NULL,
    stock INT NOT NULL DEFAULT 0,
    categoria VARCHAR(50) NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT TRUE,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_categoria (categoria),
    INDEX idx_activo (activo),
    INDEX idx_nombre (nombre)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_spanish_ci;

-- ═══════════════════════════════════════════════════════════════════════════
-- DATOS DE EJEMPLO
-- ═══════════════════════════════════════════════════════════════════════════
INSERT INTO productos (nombre, descripcion, precio, stock, categoria) VALUES
('Portátil Gaming', 'Portátil con RTX 4070, 16GB RAM, 512GB SSD', 1299.99, 15, 'Informática'),
('Monitor 27"', 'Monitor IPS 4K 144Hz', 449.99, 25, 'Informática'),
('Teclado Mecánico', 'Teclado mecánico RGB switches Cherry MX', 89.99, 50, 'Periféricos'),
('Ratón Inalámbrico', 'Ratón gaming 16000 DPI', 59.99, 40, 'Periféricos'),
('Auriculares Bluetooth', 'Auriculares con cancelación de ruido', 199.99, 30, 'Audio'),
('Webcam HD', 'Webcam 1080p con micrófono', 69.99, 20, 'Periféricos'),
('SSD 1TB', 'Disco SSD NVMe PCIe 4.0', 99.99, 35, 'Almacenamiento'),
('Hub USB-C', 'Hub 7 en 1 con HDMI y ethernet', 45.99, 60, 'Accesorios');

## 10. Resumen de Rutas

Tabla 3 Rutas del Controlador de Productos
|Método|URL|Descripción|
|---|---|---|
|GET|/productos|Lista de productos con filtros opcionales|
|GET|/productos/{id}|Ver detalle de un producto|
|GET|/productos/nuevo|Formulario para crear producto|
|GET|/productos/editar/{id}|Formulario para editar producto|
|POST|/productos/guardar|Guardar producto (crear o actualizar)|
|POST|/productos/eliminar/{id}|Eliminar producto (borrado lógico)|

## 11. Ejercicios Propuestos

Ejercicio 1: Añadir paginación

Modifica el listado de productos para que muestre solo 10 productos por página.

**Pistas:**

- Usa `Pageable` en el repositorio
    
- Modifica el servicio para aceptar `Pageable`
    
- Añade controles de paginación en la vista
    

// En el repositorio
Page<Producto> findByActivoTrue(Pageable pageable);

// En el controlador
@GetMapping
public String listar(@RequestParam(defaultValue = "0") int page, Model model) {
    Page<Producto> pagina = productoService.listarPaginado(PageRequest.of(page, 10));
    model.addAttribute("productos", pagina);
    return "productos/lista";
}

Ejercicio 2: Añadir imagen de producto

Añade un campo `imagenUrl` a la entidad Producto y muestra la imagen en el listado.

**Pasos:**

1. Añadir campo `String imagenUrl` a la entidad
    
2. Añadir campo en el formulario
    
3. Mostrar imagen con `<img th:src="${producto.imagenUrl}">`
    

Ejercicio 3: Exportar a CSV

Crea un endpoint que exporte todos los productos a formato CSV.

@GetMapping("/exportar")
public void exportarCSV(HttpServletResponse response) throws IOException {
    response.setContentType("text/csv");
    response.setHeader("Content-Disposition", "attachment; filename=productos.csv");

    PrintWriter writer = response.getWriter();
    writer.println("ID,Nombre,Precio,Stock,Categoria");

    for (Producto p : productoService.listarTodos()) {
        writer.printf("%d,%s,%.2f,%d,%s%n",
            p.getId(), p.getNombre(), p.getPrecio(), p.getStock(), p.getCategoria());
    }
}

Ejercicio 4: Validación personalizada

Crea una validación personalizada que compruebe que el nombre del producto no contenga caracteres especiales.

@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = NombreValidoValidator.class)
public @interface NombreValido {
    String message() default "El nombre contiene caracteres no permitidos";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

## 12. Recursos Adicionales

Documentación oficial

- [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/)
    
- [Thymeleaf Tutorial](https://www.thymeleaf.org/doc/tutorials/3.1/usingthymeleaf.html)
    
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
    
- [Bootstrap 5 Documentation](https://getbootstrap.com/docs/5.3/)