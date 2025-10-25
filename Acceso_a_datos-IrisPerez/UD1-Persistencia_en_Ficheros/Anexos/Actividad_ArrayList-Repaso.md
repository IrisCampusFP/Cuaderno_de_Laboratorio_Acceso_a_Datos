# REPASO: ArrayList en Java - Actividad Guiada

## 1. Recordatorio: ¿Qué es un ArrayList?

**ArrayList** es una clase de Java que implementa una **lista dinámica**. A diferencia de los arrays tradicionales, puede crecer y decrecer en tamaño automáticamente.

### Diferencias clave: Array vs ArrayList

```java
// ARRAY TRADICIONAL (tamaño fijo)
String[] nombres = new String[3];
nombres[0] = "Ana";
nombres[1] = "Luis";
nombres[2] = "María";
// Si queremos añadir un 4º elemento... ¡problema! Hay que crear un nuevo array

// ARRAYLIST (tamaño dinámico)
ArrayList<String> nombresLista = new ArrayList<>();
nombresLista.add("Ana");
nombresLista.add("Luis");
nombresLista.add("María");
nombresLista.add("Carlos"); // ¡Sin problema! Se ajusta automáticamente
```

## 2. Importación y Declaración

```java
import java.util.ArrayList;

// Declaración con tipo genérico
ArrayList<TipoDato> nombreLista = new ArrayList<>();

// Ejemplos:
ArrayList<String> nombres = new ArrayList<>();
ArrayList<Integer> edades = new ArrayList<>();
ArrayList<Double> precios = new ArrayList<>();
```

## 🔧 3. Operaciones Básicas - Ejemplo Práctico

### Ejemplo: Gestión de Estudiantes

```java
import java.util.ArrayList;

public class GestionEstudiantes {
    public static void main(String[] args) {
        
        // 1. CREAR ArrayList
        ArrayList<String> estudiantes = new ArrayList<>();
        
        // 2. AÑADIR elementos (.add)
        estudiantes.add("Ana García");
        estudiantes.add("Luis Martínez");
        estudiantes.add("María López");
        estudiantes.add("Carlos Ruiz");
        
        System.out.println("Lista inicial: " + estudiantes);
        System.out.println("Número de estudiantes: " + estudiantes.size());
        
        // 3. ACCEDER a elementos (.get)
        String primerEstudiante = estudiantes.get(0);
        System.out.println("\nPrimer estudiante: " + primerEstudiante);
        
        // 4. MODIFICAR elementos (.set)
        estudiantes.set(1, "Luis Fernández"); // Cambiamos el apellido
        System.out.println("\nDespués de modificar: " + estudiantes);
        
        // 5. ELIMINAR elementos (.remove)
        estudiantes.remove(2); // Elimina por índice
        estudiantes.remove("Carlos Ruiz"); // Elimina por objeto
        System.out.println("\nDespués de eliminar: " + estudiantes);
        
        // 6. BUSCAR elementos (.contains / .indexOf)
        boolean existe = estudiantes.contains("Ana García");
        System.out.println("\n¿Ana García está en la lista? " + existe);
        
        int posicion = estudiantes.indexOf("Luis Fernández");
        System.out.println("Posición de Luis Fernández: " + posicion);
        
        // 7. RECORRER con for-each
        System.out.println("\nListado completo:");
        for (String estudiante : estudiantes) {
            System.out.println("- " + estudiante);
        }
        
        // 8. LIMPIAR la lista (.clear)
        // estudiantes.clear();
        
        // 9. VERIFICAR si está vacía (.isEmpty)
        System.out.println("\n¿La lista está vacía? " + estudiantes.isEmpty());
    }
}
```

## Tabla Resumen de Métodos

|Método|Descripción|Ejemplo|
|---|---|---|
|`add(elemento)`|Añade al final|`lista.add("Java")`|
|`add(indice, elemento)`|Añade en posición|`lista.add(1, "Python")`|
|`get(indice)`|Obtiene elemento|`lista.get(0)`|
|`set(indice, elemento)`|Modifica elemento|`lista.set(0, "C++")`|
|`remove(indice)`|Elimina por posición|`lista.remove(1)`|
|`remove(objeto)`|Elimina por valor|`lista.remove("Java")`|
|`size()`|Tamaño de la lista|`lista.size()`|
|`contains(objeto)`|¿Contiene elemento?|`lista.contains("Java")`|
|`indexOf(objeto)`|Posición del elemento|`lista.indexOf("Java")`|
|`clear()`|Vacía la lista|`lista.clear()`|
|`isEmpty()`|¿Está vacía?|`lista.isEmpty()`|

## ACTIVIDAD GUIADA: Sistema de Gestión de Productos

### Objetivo

Crear un programa que gestione un inventario de productos usando ArrayList.

### Paso 1: Crear la clase Producto

```java
public class Producto {
    private String codigo;
    private String nombre;
    private double precio;
    
    // Constructor
    public Producto(String codigo, String nombre, double precio) {
        this.codigo = codigo;
        this.nombre = nombre;
        this.precio = precio;
    }
    
    // Getters
    public String getCodigo() { return codigo; }
    public String getNombre() { return nombre; }
    public double getPrecio() { return precio; }
    
    // Setter para precio (puede cambiar)
    public void setPrecio(double precio) { this.precio = precio; }
    
    @Override
    public String toString() {
        return codigo + " - " + nombre + " (" + precio + "€)";
    }
}
```

### Paso 2: Ejercicios para los Alumnos

**EJERCICIO 1: Crear el inventario base**

```java
// TODO: Crear un ArrayList de Productos llamado "inventario"
// TODO: Añadir 5 productos al inventario con estos datos:
// P001 - Portátil - 899.99€
// P002 - Ratón - 25.50€
// P003 - Teclado - 45.00€
// P004 - Monitor - 199.99€
// P005 - Webcam - 59.90€
// TODO: Mostrar todos los productos
```

**EJERCICIO 2: Búsqueda y consulta**

```java
// TODO: Buscar el producto con código "P003" y mostrar sus datos
// TODO: Verificar si existe un producto llamado "Ratón"
// TODO: Mostrar cuántos productos hay en el inventario
```

**EJERCICIO 3: Modificaciones**

```java
// TODO: Cambiar el precio del Monitor a 179.99€
// TODO: Eliminar la Webcam del inventario
// TODO: Añadir un nuevo producto: P006 - Altavoces - 35.00€
```

**EJERCICIO 4: Operaciones avanzadas**

```java
// TODO: Calcular el precio total del inventario
// TODO: Encontrar el producto más caro
// TODO: Mostrar solo productos con precio superior a 50€
```

## Consejos Didácticos

**Para recordar:**

- ArrayList es **dinámico** (crece/decrece automáticamente)
    
- Usa **tipos genéricos** `<TipoDato>` para seguridad de tipos
    
- Los índices empiezan en **0**
    
- Es parte del paquete `java.util`
    
- Muy útil cuando no sabemos el tamaño exacto de antemano

**Errores comunes que debes evitar:**

```java
// ❌ INCORRECTO
ArrayList<int> numeros; // Debe usar Integer, no int

// ✅ CORRECTO
ArrayList<Integer> numeros = new ArrayList<>();
```


> [!NOTE] Actividad ArrayList Resuelta:
> https://github.com/IrisCampusFP/ActividadesAccesoADatos/tree/main/UD1-Persistencia_en_Ficheros/ActArrayList-Sistema_Gestion_de_Productos
