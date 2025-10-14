Has compartido una serie de recomendaciones muy acertadas sobre el uso de la clase `Scanner` en Java. A continuación se explican en profundidad con ejemplos y justificación técnica.

## 1. Cerrar siempre con `sc.close()` para liberar recursos

El objeto `Scanner` implementa la interfaz `Closeable`, por lo que debe cerrarse una vez que ya no se necesite. Esto es especialmente importante cuando se usan recursos como archivos o flujos de entrada/salida.

```java
Scanner sc = new Scanner(System.in);
// uso del scanner
sc.close();
```

Cuidado: Si cierras `System.in` con `sc.close()`, no podrás reutilizarlo después. En aplicaciones largas, puede ser mejor evitar cerrarlo y dejar que el sistema lo gestione.

## 2. Si se usan métodos mixtos (`nextInt()` y `nextLine()`), limpiar el buffer

Este es un error clásico: `nextInt()` o similares no consumen la línea nueva (`\n`) que el usuario introduce tras escribir el número. Si luego se llama a `nextLine()`, este leerá esa línea vacía.

```java
Scanner sc = new Scanner(System.in);

System.out.print("Introduce un número: ");
int num = sc.nextInt();
sc.nextLine(); // limpiar el buffer

System.out.print("Introduce tu nombre: ");
String nombre = sc.nextLine();

System.out.println("Hola " + nombre + ", tu número es " + num);
```

## 3. Para proyectos grandes, combinarlo con clases POJO para estructurar datos

Es una buena práctica separar la lógica de entrada/salida de los datos propiamente dichos.

```java
public class Persona {
    private String nombre;
    private int edad;
	
	// Constructor
	public Persona(String nombre, int edad) {
		this.nombre = nombre;
		this.edad = edad;
	}
	
	// Getters
    public String getNombre() {
        return nombre;
    }
    
    public int getEdad() {
        return edad;
    }
    
    // Setters
    public void setNombre(String nombre) {
        this.nombre = nombre;
    }
	
	public void setEdad(int edad) {
        this.edad = edad;
    }
    
    // toString
    @Override
    public String toString() {
        return "Persona{nombre='" + nombre + "', edad=" + edad + "}";
    }
}
```

```java
Scanner sc = new Scanner(System.in);
System.out.print("Nombre: ");
String nombre = sc.nextLine();

System.out.print("Edad: ");
int edad = sc.nextInt();

Persona p = new Persona(nombre, edad);
System.out.println(p);
```

Esto mejora el diseño, facilita pruebas unitarias y la separación de responsabilidades (principio SOLID: SRP).

## 4. Evitar `Scanner` con ficheros enormes. Usar `BufferedReader`

`Scanner` realiza parsing y validaciones que lo hacen más lento para grandes volúmenes de datos. En su lugar se recomienda `BufferedReader`.

```java
BufferedReader br = new BufferedReader(new FileReader("datos.txt"));
String linea;
while ((linea = br.readLine()) != null) {
    System.out.println(linea);
}
br.close();
```

## Conclusión

| Práctica                               | ¿Por qué es útil?                     |
| -------------------------------------- | ------------------------------------- |
| `sc.close()`                           | Libera recursos del sistema           |
| `sc.nextLine()` después de `nextInt()` | Evita errores de entrada vacía        |
| Uso con POJOs                          | Mejora estructura y mantenibilidad    |
| Evitar en ficheros grandes             | Mejora rendimiento con BufferedReader |
