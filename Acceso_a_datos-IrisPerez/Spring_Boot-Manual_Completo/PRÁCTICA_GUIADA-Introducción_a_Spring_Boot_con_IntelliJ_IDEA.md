**Módulo:** Acceso a Datos  
**Ciclo:** Desarrollo de Aplicaciones Multiplataforma (DAM)  
**Duración estimada:** 3-4 horas  
**Objetivo:** Configurar el entorno de desarrollo y crear tu primera aplicación Spring Boot con API REST

## Índice

1. Preparación del Entorno
    
2. Instalación de IntelliJ IDEA Ultimate
    
3. Creación del Proyecto Spring Boot
    
4. Exploración de la Estructura
    
5. Tu Primera API REST
    
6. Pruebas y Verificación

## Requisitos Previos

Antes de comenzar, asegúrate de tener:

- ✅ Cuenta de correo electrónico de estudiante (.edu o certificado de estudiante)
    
- ✅ Conexión a Internet estable
    
- ✅ Al menos 4 GB de espacio libre en disco
    
- ✅ Sistema operativo: Windows 10/11, macOS o Linux

## BLOQUE 1: Preparación del Entorno

### Paso 1.1: Verificar Instalación de Java JDK

**Tarea:**  
Verifica que tienes Java JDK 17 o superior instalado en tu sistema.

**Instrucciones:**

1. Abre una terminal o símbolo del sistema (CMD en Windows)
    
2. Ejecuta el siguiente comando:
    
    java -version
    
3. También verifica el compilador:
    
    javac -version

**Captura requerida:**  
_Inserta aquí una captura de pantalla mostrando la salida de ambos comandos_

ESPACIO PARA CAPTURA 1.1

**Resultado esperado:**

java version "17.x.x" o superior
javac 17.x.x o superior

**Mis observaciones (para repasar):**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 1.2: Descargar Java JDK (si no lo tienes)

**Tarea:**  
Si no tienes Java JDK instalado o tu versión es menor a 17, descárgalo e instálalo.

**Instrucciones:**

1. Accede a: [https://www.oracle.com/java/technologies/downloads/](https://www.oracle.com/java/technologies/downloads/)
    
2. Selecciona **Java 17** o **Java 21** (LTS - Long Term Support)
    
3. Descarga el instalador para tu sistema operativo:
    
    - Windows: `jdk-17_windows-x64_bin.exe`
        
    - macOS: `jdk-17_macos-x64_bin.dmg`
        
    - Linux: `jdk-17_linux-x64_bin.tar.gz`
        
4. Ejecuta el instalador y sigue las instrucciones por defecto

**Captura requerida:**  
_Inserta aquí una captura del instalador de Java o de la página de descarga_

ESPACIO PARA CAPTURA 1.2

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 1.3: Configurar Variables de Entorno (Windows)

**Tarea:**  
Configura las variables de entorno JAVA_HOME y PATH (principalmente en Windows).

**Instrucciones para Windows:**

1. Botón derecho en «Este equipo» → Propiedades
    
2. Configuración avanzada del sistema → Variables de entorno
    
3. En «Variables del sistema» → Nuevo:
    
    - **Nombre:** `JAVA_HOME`
        
    - **Valor:** `C:\Program Files\Java\jdk-17` (ajusta según tu instalación)
        
4. Edita la variable `Path` → Nuevo:
    
    - Añade: `%JAVA_HOME%\bin`
        
5. Acepta y cierra todas las ventanas
    
6. **Reinicia** la terminal y verifica de nuevo con `java -version`

**Captura requerida:**  
_Captura de las variables de entorno configuradas_

ESPACIO PARA CAPTURA 1.3

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

## BLOQUE 2: Instalación de IntelliJ IDEA Ultimate

### Paso 2.1: Obtener Licencia Educativa JetBrains

**Tarea:**  
Solicita tu licencia gratuita de estudiante para IntelliJ IDEA Ultimate.

**Instrucciones:**

1. Accede a: [https://www.jetbrains.com/community/education/#students](https://www.jetbrains.com/community/education/#students)
    
2. Click en **«Apply now»** o **«Solicitar ahora»**
    
3. Completa el formulario con:
    
    - Tu correo electrónico de estudiante (.edu)
        
    - O sube un certificado de estudiante
        
4. Revisa tu correo electrónico
    
5. Activa tu cuenta JetBrains siguiendo el enlace recibido
    
6. Crea tu contraseña

**Captura requerida:**  
_Captura del correo de confirmación o del panel de estudiante JetBrains_

[ESPACIO PARA CAPTURA 2.1]

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 2.2: Descargar IntelliJ IDEA Ultimate

**Tarea:**  
Descarga el instalador de IntelliJ IDEA Ultimate.

**Instrucciones:**

1. Accede a: [https://www.jetbrains.com/idea/download/](https://www.jetbrains.com/idea/download/)
    
2. Selecciona la pestaña **«Ultimate»** (NO Community)
    
3. Descarga para tu sistema operativo
    
4. Espera a que se complete la descarga

**Captura requerida:**  
_Captura de la página de descarga con la versión Ultimate seleccionada_

[ESPACIO PARA CAPTURA 2.2]

**IMPORTANTE:**  
Asegúrate de descargar la versión **Ultimate**, no Community. Spring Boot funciona mejor con Ultimate.

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 2.3: Instalar IntelliJ IDEA Ultimate

**Tarea:**  
Instala IntelliJ IDEA Ultimate en tu equipo.

**Instrucciones:**

1. Ejecuta el instalador descargado
    
2. Acepta el acuerdo de licencia
    
3. En la pantalla de opciones, marca:
    
    - ✅ Create Desktop Shortcut
        
    - ✅ Update PATH variable (restart needed)
        
    - ✅ .java - Open files with IntelliJ IDEA
        
    - ✅ Add «Open Folder as Project»
        
4. Continúa con la instalación por defecto
    
5. Reinicia el equipo si es necesario

**Captura requerida:**  
_Captura durante el proceso de instalación mostrando las opciones marcadas_

[ESPACIO PARA CAPTURA 2.3]

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 2.4: Primera Configuración de IntelliJ IDEA

**Tarea:**  
Configura IntelliJ IDEA por primera vez y activa tu licencia de estudiante.

**Instrucciones:**

1. Abre IntelliJ IDEA
    
2. Si es la primera vez, acepta los términos y condiciones
    
3. Puedes importar configuraciones previas o empezar sin ellas
    
4. Selecciona el tema (Darcula recomendado para programadores)
    
5. En la pantalla de licencia, selecciona **«Log in to JetBrains Account»**
    
6. Introduce tus credenciales de JetBrains (las que creaste en el paso 2.1)
    
7. Verifica que aparece «Licensed to [Tu nombre] (Educational License)»

**Captura requerida:**  
_Captura de la pantalla principal de IntelliJ con la licencia activada_

[ESPACIO PARA CAPTURA 2.4]

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

## BLOQUE 3: Creación del Proyecto Spring Boot

### Paso 3.1: Iniciar Nuevo Proyecto con Spring Initializr

**Tarea:**  
Crea un nuevo proyecto Spring Boot usando Spring Initializr integrado en IntelliJ.

**Instrucciones:**

1. En la pantalla de bienvenida, haz clic en **«New Project»**
    
2. En el menú lateral izquierdo, selecciona **«Spring Initializr»**
    
3. Verifica que el servidor sea: `https://start.spring.io`
    
4. Haz clic en **«Next»**

**Captura requerida:**  
_Captura de la pantalla de selección de Spring Initializr_

[ESPACIO PARA CAPTURA 3.1]

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 3.2: Configurar Metadatos del Proyecto

**Tarea:**  
Configura los metadatos básicos de tu proyecto Spring Boot.

**Instrucciones:**  
Completa los siguientes campos:

|Campo|Valor|
|---|---|
|**Name**|`miprimeraapi`|
|**Language**|`Java`|
|**Type**|`Maven`|
|**Group**|`com.dam.accesodatos`|
|**Artifact**|`miprimeraapi`|
|**Package name**|`com.dam.accesodatos.miprimeraapi`|
|**JDK**|`17` o superior|
|**Java**|`17` o superior|
|**Packaging**|`Jar`|

**Captura requerida:**  
_Captura de la pantalla con todos los metadatos configurados_

[ESPACIO PARA CAPTURA 3.2]

**Explicación de los campos:**

- **Group:** Identifica tu organización (normalmente dominio inverso)
    
- **Artifact:** Nombre del proyecto (sin espacios, minúsculas)
    
- **Package name:** Se genera automáticamente, es el paquete base Java
    
- **Packaging:** Jar crea un ejecutable independiente

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 3.3: Seleccionar Dependencias

**Tarea:**  
Selecciona las dependencias necesarias para tu primera API REST.

**Instrucciones:**

1. Haz clic en **«Next»** después de configurar los metadatos
    
2. En la pantalla de dependencias, busca y selecciona las siguientes:

**Dependencias a añadir:**

|Categoría|Dependencia|Descripción|
|---|---|---|
|**Web**|Spring Web|Para crear aplicaciones web y APIs REST|
|**Developer Tools**|Spring Boot DevTools|Para recarga automática en desarrollo|
|**Developer Tools**|Lombok|Para reducir código boilerplate|

3. Puedes usar el buscador escribiendo el nombre
    
4. Asegúrate de marcar las tres dependencias

**Captura requerida:**  
_Captura mostrando las tres dependencias seleccionadas_

[ESPACIO PARA CAPTURA 3.3]

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 3.4: Finalizar Creación del Proyecto

**Tarea:**  
Finaliza la creación y espera a que IntelliJ descargue todas las dependencias.

**Instrucciones:**

1. Haz clic en **«Create»**
    
2. Selecciona la ubicación donde guardar el proyecto
    
3. IntelliJ comenzará a descargar todas las dependencias de Maven
    
4. Espera a que termine el proceso (puedes ver el progreso en la barra inferior)
    
5. Observa la estructura del proyecto en el panel izquierdo

**Captura requerida:**  
_Captura del proyecto recién creado con la estructura visible_

[ESPACIO PARA CAPTURA 3.4]

**Tiempo estimado:**  
La primera descarga puede tardar 5-10 minutos dependiendo de tu conexión.

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

## BLOQUE 4: Exploración de la Estructura

### Paso 4.1: Estructura de Carpetas

**Tarea:**  
Explora y documenta la estructura de carpetas generada por Spring Initializr.

**Estructura generada:**

miprimeraapi/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/dam/accesodatos/miprimeraapi/
│   │   │       └── MiprimeraapiApplication.java
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── static/
│   │       └── templates/
│   └── test/
│       └── java/
│           └── com/dam/accesodatos/miprimeraapi/
│               └── MiprimeraapiApplicationTests.java
├── target/
├── pom.xml
└── mvnw (Maven Wrapper)

**Captura requerida:**  
_Captura del árbol de carpetas expandido en IntelliJ_

[ESPACIO PARA CAPTURA 4.1]

**Descripción de carpetas:**

- **src/main/java:** Código fuente Java
    
- **src/main/resources:** Archivos de configuración y recursos
    
- **src/test:** Tests unitarios y de integración
    
- **target:** Archivos compilados (generado automáticamente)
    
- **pom.xml:** Configuración de Maven y dependencias

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 4.2: Análisis del archivo pom.xml

**Tarea:**  
Abre y analiza el archivo `pom.xml` (Project Object Model de Maven).

**Instrucciones:**

1. En el panel izquierdo, haz doble clic en `pom.xml`
    
2. Observa las siguientes secciones:
    
    - `<parent>`: Spring Boot Starter Parent
        
    - `<groupId>`, `<artifactId>`, `<version>`
        
    - `<properties>`: Versión de Java
        
    - `<dependencies>`: Las dependencias que seleccionaste

**Captura requerida:**  
_Captura del archivo pom.xml abierto mostrando las dependencias_

[ESPACIO PARA CAPTURA 4.2]

**Dependencias que deberías ver:**

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 4.3: La Clase Principal - @SpringBootApplication

**Tarea:**  
Analiza la clase principal de la aplicación Spring Boot.

**Instrucciones:**

1. Navega a: `src/main/java/com/dam/accesodatos/miprimeraapi/`
    
2. Abre el archivo `MiprimeraapiApplication.java`
    
3. Observa la estructura del código

**Código que deberías ver:**

package com.dam.accesodatos.miprimeraapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MiprimeraapiApplication {

    public static void main(String[] args) {
        SpringApplication.run(MiprimeraapiApplication.class, args);
    }

}

**Captura requerida:**  
_Captura de la clase principal abierta en el editor_

[ESPACIO PARA CAPTURA 4.3]

**📝 Explicación:**

- `@SpringBootApplication`: Anotación que combina `@Configuration`, `@EnableAutoConfiguration` y `@ComponentScan`
    
- `SpringApplication.run()`: Método que arranca la aplicación Spring Boot
    
- Esta clase es el **punto de entrada** de tu aplicación

**💡 Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 4.4: Primer Arranque de la Aplicación

**Tarea:**  
Ejecuta por primera vez tu aplicación Spring Boot.

**Instrucciones:**

1. Haz clic derecho sobre `MiprimeraapiApplication.java`
    
2. Selecciona **«Run “MiprimeraapiApplication”»**
    
3. O simplemente haz clic en el botón ▶️ verde junto al método `main`
    
4. Observa la consola inferior
    
5. Busca el mensaje: `Started MiprimeraapiApplication in X.XXX seconds`

**📸 Captura requerida:**  
_Captura de la consola mostrando que la aplicación ha arrancado correctamente_

[ESPACIO PARA CAPTURA 4.4]

**📝 Información importante en los logs:**

Tomcat started on port(s): 8080 (http)
Started MiprimeraapiApplication in X.XXX seconds

Esto significa que tu aplicación está corriendo en: `http://localhost:8080`

**💡 Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 4.5: Verificar en el Navegador

**Tarea:**  
Accede a la aplicación desde el navegador.

**Instrucciones:**

1. Abre tu navegador web favorito
    
2. Accede a: `http://localhost:8080`
    
3. Deberías ver una página de error (Whitelabel Error Page)

**Captura requerida:**  
_Captura de la página de error de Spring Boot en el navegador_

[ESPACIO PARA CAPTURA 4.5]

**¿Por qué aparece un error?**  
Es normal. No hemos creado ningún endpoint todavía, por eso Spring Boot muestra una página de error por defecto. En el siguiente bloque crearemos nuestros endpoints.

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

## BLOQUE 5: Tu Primera API REST

### Paso 5.1: Crear el Package «controller»

**Tarea:**  
Crea un nuevo paquete para organizar tus controladores REST.

**Instrucciones:**

1. Haz clic derecho sobre: `com.dam.accesodatos.miprimeraapi`
    
2. Selecciona: **New → Package**
    
3. Nombra el paquete: `controller`
    
4. El paquete completo será: `com.dam.accesodatos.miprimeraapi.controller`

**Captura requerida:**  
_Captura mostrando el nuevo paquete «controller» creado_

[ESPACIO PARA CAPTURA 5.1]

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 5.2: Crear la Clase HolaMundoController

**Tarea:**  
Crea tu primer controlador REST.

**Instrucciones:**

1. Haz clic derecho sobre el paquete `controller`
    
2. Selecciona: **New → Java Class**
    
3. Nombra la clase: `HolaMundoController`
    
4. Haz clic en OK

**Captura requerida:**  
_Captura del diálogo de creación de la nueva clase_

[ESPACIO PARA CAPTURA 5.2]

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 5.3: Implementar el Primer Endpoint

**Tarea:**  
Escribe el código para crear tu primer endpoint REST que devuelva «Hola Mundo».

**Instrucciones:**  
Escribe el siguiente código en `HolaMundoController.java`:

package com.dam.accesodatos.miprimeraapi.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HolaMundoController {

    @GetMapping("/hola")
    public String saludar() {
        return "¡Hola Mundo desde Spring Boot!";
    }
    
    @GetMapping("/")
    public String inicio() {
        return "Bienvenido a mi primera API REST con Spring Boot";
    }
}

**Captura requerida:**  
_Captura del código completo del controlador_

[ESPACIO PARA CAPTURA 5.3]

**Explicación del código:**

- `@RestController`: Indica que esta clase es un controlador REST
    
- `@GetMapping`: Define un endpoint que responde a peticiones GET
    
- `/hola`: La ruta URL del endpoint
    
- El método retorna un String que se envía como respuesta
    

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 5.4: Reiniciar la Aplicación

**Tarea:**  
Reinicia la aplicación para que los cambios surtan efecto.

**Instrucciones:**

1. Haz clic en el botón **«Stop»** (cuadrado rojo) en la barra de herramientas
    
2. Espera a que la aplicación se detenga
    
3. Vuelve a ejecutar haciendo clic en **«Run»** (▶️ verde)
    
4. Espera a que aparezca el mensaje «Started MiprimeraapiApplication»

**Captura requerida:**  
_Captura de la consola mostrando el reinicio exitoso_

ESPACIO PARA CAPTURA 5.4

**Nota sobre DevTools:**  
Con Spring Boot DevTools, muchos cambios se recargan automáticamente sin necesidad de reiniciar manualmente.

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

## BLOQUE 6: Pruebas y Verificación

### Paso 6.1: Probar el Endpoint Raíz

**Tarea:**  
Accede al endpoint raíz de tu API desde el navegador.

**Instrucciones:**

1. Abre el navegador
    
2. Accede a: `http://localhost:8080/`
    
3. Deberías ver el mensaje: «Bienvenido a mi primera API REST con Spring Boot»

**Captura requerida:**  
_Captura del navegador mostrando el mensaje de bienvenida_

ESPACIO PARA CAPTURA 6.1

**Resultado esperado:**

Bienvenido a mi primera API REST con Spring Boot

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 6.2: Probar el Endpoint /hola

**Tarea:**  
Accede al endpoint `/hola` de tu API.

**Instrucciones:**

1. En el navegador, accede a: `http://localhost:8080/hola`
    
2. Deberías ver: «¡Hola Mundo desde Spring Boot!»

**Captura requerida:**  
_Captura del navegador mostrando el saludo_

ESPACIO PARA CAPTURA 6.2

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 6.3: Crear un Endpoint con Parámetros

**Tarea:**  
Añade un nuevo endpoint que reciba un parámetro de nombre.

**Instrucciones:**  
Añade el siguiente método a tu clase `HolaMundoController`:

@GetMapping("/saludar/{nombre}")
public String saludarPersonalizado(@PathVariable String nombre) {
    return "¡Hola " + nombre + "! Bienvenido a Spring Boot";
}

**Código completo del controlador:**

package com.dam.accesodatos.miprimeraapi.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HolaMundoController {

    @GetMapping("/hola")
    public String saludar() {
        return "¡Hola Mundo desde Spring Boot!";
    }
    
    @GetMapping("/")
    public String inicio() {
        return "Bienvenido a mi primera API REST con Spring Boot";
    }
    
    @GetMapping("/saludar/{nombre}")
    public String saludarPersonalizado(@PathVariable String nombre) {
        return "¡Hola " + nombre + "! Bienvenido a Spring Boot";
    }
}

**Captura requerida:**  
_Captura del código con el nuevo método añadido_

ESPACIO PARA CAPTURA 6.3

**💡 Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 6.4: Probar el Endpoint con Parámetros

**Tarea:**  
Prueba el nuevo endpoint pasando diferentes nombres.

**Instrucciones:**

1. Reinicia la aplicación (o espera a que DevTools la recargue)
    
2. Accede a: `http://localhost:8080/saludar/Juan`
    
3. Prueba con tu nombre: `http://localhost:8080/saludar/TuNombre`

**Captura requerida:**  
_Captura del navegador mostrando el saludo personalizado con tu nombre_

[ESPACIO PARA CAPTURA 6.4]

**Explicación:**

- `{nombre}`: Es un path variable (variable de ruta)
    
- `@PathVariable`: Captura el valor de la URL y lo pasa al parámetro del método

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 6.5: Crear un Endpoint que Devuelva JSON

**Tarea:**  
Crea un nuevo paquete y clase para devolver objetos JSON.

**Instrucciones:**

**1. Crear el paquete `model`:**

- Clic derecho en `com.dam.accesodatos.miprimeraapi` → New → Package
    
- Nombre: `model`

**2. Crear la clase `Persona`:**

package com.dam.accesodatos.miprimeraapi.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Persona {
    private String nombre;
    private int edad;
    private String ciudad;
}

**3. Crear un nuevo controlador `PersonaController`:**

package com.dam.accesodatos.miprimeraapi.controller;

import com.dam.accesodatos.miprimeraapi.model.Persona;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Arrays;
import java.util.List;

@RestController
@RequestMapping("/api/personas")
public class PersonaController {

    @GetMapping
    public List<Persona> obtenerPersonas() {
        return Arrays.asList(
            new Persona("Ana García", 25, "Madrid"),
            new Persona("Carlos López", 30, "Barcelona"),
            new Persona("María Sánchez", 28, "Valencia")
        );
    }
    
    @GetMapping("/primera")
    public Persona obtenerPrimeraPersona() {
        return new Persona("Juan Pérez", 35, "Sevilla");
    }
}

**Captura requerida:**  
_Captura mostrando las tres clases creadas: HolaMundoController, Persona y PersonaController_

ESPACIO PARA CAPTURA 6.5

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 6.6: Probar el Endpoint JSON

**Tarea:**  
Accede a los nuevos endpoints y observa la respuesta JSON.

**Instrucciones:**

1. Reinicia la aplicación
    
2. Accede a: `http://localhost:8080/api/personas`
    
3. Accede también a: `http://localhost:8080/api/personas/primera`
    

**📸 Captura requerida:**  
_Captura del navegador mostrando la lista de personas en formato JSON_

ESPACIO PARA CAPTURA 6.6

**Resultado esperado:**

[
  {
    "nombre": "Ana García",
    "edad": 25,
    "ciudad": "Madrid"
  },
  {
    "nombre": "Carlos López",
    "edad": 30,
    "ciudad": "Barcelona"
  },
  {
    "nombre": "María Sánchez",
    "edad": 28,
    "ciudad": "Valencia"
  }
]

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 6.7: Instalar y Usar Postman (Opcional pero Recomendado)

**Tarea:**  
Descarga e instala Postman para probar APIs de forma profesional.

**Instrucciones:**

1. Accede a: [https://www.postman.com/downloads/](https://www.postman.com/downloads/)
    
2. Descarga Postman para tu sistema operativo
    
3. Instala y abre Postman
    
4. Crea una nueva request:
    
    - Método: **GET**
        
    - URL: `http://localhost:8080/api/personas`
        
5. Haz clic en **«Send»**

**Captura requerida:**  
_Captura de Postman mostrando la petición y respuesta JSON_

ESPACIO PARA CAPTURA 6.7

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

### Paso 6.8: Analizar los Logs de Spring Boot

**Tarea:**  
Analiza los logs que genera Spring Boot en la consola.

**Instrucciones:**

1. Observa la consola de IntelliJ
    
2. Identifica información importante:
    
    - Puerto donde corre la aplicación: `Tomcat started on port(s): 8080`
        
    - Tiempo de arranque: `Started MiprimeraapiApplication in X.XXX seconds`
        
    - Peticiones HTTP: Cuando accedas a un endpoint, verás logs como:
        
        INFO : o.a.c.c.C.[Tomcat].[localhost].[/] : Initializing Spring DispatcherServlet 'dispatcherServlet'

**Captura requerida:**  
_Captura de la consola mostrando los logs detallados_

ESPACIO PARA CAPTURA 6.8

**Mis observaciones:**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

---

## Resumen y Conceptos Clave

### Checklist de Logros

Marca lo que has conseguido en esta práctica:

- [ ] Instalado y configurado Java JDK 17+
    
- [ ] Obtenido licencia educativa de JetBrains
    
- [ ] Instalado IntelliJ IDEA Ultimate
    
- [ ] Creado proyecto Spring Boot con Spring Initializr
    
- [ ] Comprendido la estructura de un proyecto Spring Boot
    
- [ ] Analizado el archivo pom.xml
    
- [ ] Creado tu primer controlador REST
    
- [ ] Implementado endpoints GET simples
    
- [ ] Creado endpoints con path variables
    
- [ ] Devuelto objetos JSON
    
- [ ] Probado la API desde el navegador
    
- [ ] (Opcional) Usado Postman para pruebas

### Conceptos Importantes para el Examen

**Anota aquí los conceptos que consideres más importantes:**

#### 1. Anotaciones de Spring Boot:

@SpringBootApplication: _____________________________________________
@RestController: ____________________________________________________
@GetMapping: ________________________________________________________
@PathVariable: ______________________________________________________
@RequestMapping: ____________________________________________________

#### 2. Estructura del Proyecto:

¿Dónde va el código Java?: __________________________________________
¿Dónde van los archivos de configuración?: __________________________
¿Qué es el pom.xml?: ________________________________________________

#### 3. Maven:

¿Qué es Maven?: _____________________________________________________
¿Para qué sirven las dependencias?: _________________________________

#### 4. API REST:

¿Qué es un endpoint?: _______________________________________________
¿Qué es JSON?: ______________________________________________________
¿Diferencia entre @Controller y @RestController?: ___________________

## Ejercicios Adicionales (Reto)

Si has terminado antes de tiempo, intenta estos ejercicios:

### Ejercicio 1: Calculadora REST

Crea un controlador `CalculadoraController` con endpoints para:

- Sumar: `/api/calculadora/sumar/{num1}/{num2}`
    
- Restar: `/api/calculadora/restar/{num1}/{num2}`
    
- Multiplicar: `/api/calculadora/multiplicar/{num1}/{num2}`

**Captura de tu solución:**

ESPACIO PARA CAPTURA EJERCICIO 1

### Ejercicio 2: Lista de Tareas

Crea una clase `Tarea` con:

- id (int)
    
- titulo (String)
    
- completada (boolean)

Crea un `TareaController` que devuelva una lista de 3 tareas.

**Captura de tu solución:**

ESPACIO PARA CAPTURA EJERCICIO 2

## Reflexión Final

**¿Qué has aprendido en esta práctica?**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

**¿Qué dificultades has encontrado?**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

**¿Qué te ha resultado más interesante?**

_____________________________________________________________________
_____________________________________________________________________
_____________________________________________________________________

## Recursos Adicionales

- **Documentación oficial Spring Boot:** [https://spring.io/projects/spring-boot](https://spring.io/projects/spring-boot)
    
- **Guías de Spring:** [https://spring.io/guides](https://spring.io/guides)
    
- **Maven Repository:** [https://mvnrepository.com/](https://mvnrepository.com/)
    
- **Lombok:** [https://projectlombok.org/](https://projectlombok.org/)

## Datos del Alumno

**Nombre completo:** _________________________________________________  
**Curso:** DAM - Acceso a Datos  
**Fecha de realización:** ___________________________________________  
**Tiempo empleado:** ________________________________________________

## Evaluación del Profesor

|Criterio|Puntuación|Observaciones|
|---|---|---|
|Instalación correcta del entorno|/2||
|Creación del proyecto|/2||
|Implementación de endpoints básicos|/2||
|Implementación de endpoints con parámetros|/1||
|Devolución de objetos JSON|/1||
|Calidad de las capturas|/1||
|Observaciones y reflexiones|/1||
|**TOTAL**|**/10**||

**¡Enhorabuena! Has completado tu primera práctica con Spring Boot**

_Práctica elaborada por David Valbuena Segura para el módulo de Acceso a Datos - DAM_  
_Versión 1.0 - Curso 2025/2026_