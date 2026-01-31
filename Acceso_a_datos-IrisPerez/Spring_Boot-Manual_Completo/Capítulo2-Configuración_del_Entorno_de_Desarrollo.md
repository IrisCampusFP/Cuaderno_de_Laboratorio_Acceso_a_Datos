## 2.1 Instalación y Configuración de IntelliJ IDEA

**Selección de edición para uso educativo:**

**IntelliJ IDEA Ultimate Edition** es la opción recomendada para instituciones educativas debido a su integración completa con Spring Initializr, soporte avanzado para Spring Boot con configuraciones de ejecución dedicadas, ventana de herramientas Spring avanzada, y visualización de beans. Las licencias educativas están disponibles a costo reducido.

**IntelliJ IDEA Community Edition** sirve como alternativa gratuita adecuada para desarrollo básico de Spring Boot, aunque requiere uso externo de Spring Initializr y configuración manual de ejecución, con características específicas de Spring limitadas.

## 2.2 Pasos de Instalación y Configuración Inicial

**Proceso de instalación:**

1. Visitar [https://www.jetbrains.com/idea/download/](https://www.jetbrains.com/idea/download/) y seleccionar la edición apropiada
    
2. Para educación: Solicitar licencia educativa o usar Community Edition
    
3. Configurar JDK (Java 17 o posterior recomendado para Spring Boot 3.x)
    
4. Establecer preferencias de estructura de proyecto
    
5. Habilitar auto-importación para dependencias Maven/Gradle

**Configuración específica para educación:**

- Habilitar «Build project automatically» bajo Settings → Build, Execution, Deployment → Compiler
    
- Configurar estándares de formateo de código para entregas consistentes de estudiantes
    
- Establecer esquemas de estilo de código compartidos para consistencia en el aula
    
- Configurar parámetros del compilador Java agregando `-parameters` bajo Java Compiler → Additional command line parameters

## 2.3 Plugins Esenciales para Desarrollo Spring Boot

**Para usuarios de Community Edition:**

**Spring Boot Assistant** (Plugin ID: 17747) proporciona auto-completado para application.properties/yml y asistencia de configuración Spring Boot. **Database Navigator** reemplaza las herramientas de base de datos faltantes en Community Edition, esencial para proyectos Spring Data JPA.

**Plugins educativos recomendados (ambas ediciones):**

**SonarLint** ofrece retroalimentación en tiempo real sobre calidad de código, ayudando a los estudiantes a aprender mejores prácticas con retroalimentación instantánea sobre errores potenciales y code smells.

**RestfulTool** permite pruebas de API directamente dentro del IDE con generación de peticiones HTTP y gestión de endpoints.