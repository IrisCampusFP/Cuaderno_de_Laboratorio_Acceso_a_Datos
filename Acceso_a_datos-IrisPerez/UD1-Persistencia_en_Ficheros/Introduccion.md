# INTRODUCCIÓN

Fecha: 16/09/2025

Esta unidad introduce los conceptos fundamentales de persistencia de datos mediante el uso de ficheros en Java. Se abordan técnicas de acceso secuencial y aleatorio, así como el tratamiento de ficheros de texto, binarios, y estructuras XML. Se utilizarán las clases del paquete `java.io` y técnicas de lectura/escritura eficientes y seguras.

La persistencia de la información es una característica clave en cualquier sistema informático que necesita conservar datos más allá de la ejecución de un programa. A lo largo de esta unidad aprenderemos a utilizar diferentes mecanismos de almacenamiento de datos, centrados en el uso de ficheros como medio de persistencia básica en entornos Java.

Trabajar con ficheros permite desarrollar aplicaciones que no dependen exclusivamente de bases de datos, pero que aún así conservan la información de forma organizada y estructurada. Este conocimiento sienta las bases para abordar más adelante el uso de bases de datos relacionales y no relacionales.

Comprender el funcionamiento de las rutas, los tipos de acceso, los buffers, streams, y la serialización de objetos será esencial para implementar soluciones persistentes eficaces.

## Resumen (hecho en clase):

La **persistencia en ficheros** permite almacenar información de forma permanente, fuera de la memoria volátil de la aplicación. Se realiza sobre ficheros locales utilizando texto plano, binario o XML. 

Útil para:

- guardar datos entre ejecuciones de un programa
    
- registrar eventos o logs
    
- generar informes o respaldos
    
- trabajar con configuraciones o datos estructurados en disco

### Tipos de ficheros

- **Ficheros de texto** (FileWriter, BufferedWriter, FileReader, BufferedReader) para lectura y escritura de datos en formato legible

- **Ficheros binarios** (DataInputStream, DataOutputStream y clases Serializable) para guardar objetos completos de Java

- **Ficheros serializados** (ObjectOutputStream, ObjectInputStream y clases Serializable) para guardar objetos completos de Java

- **Ficheros XML** (DocumentBuilder, Element, NodeList, XpathExpression, XPath, Unmarshaller, Marshaller) para representar y procesar estructuras jerárquicas de datos

#### Tecnologías

- DOM = Document Object Model

- JAX = Java Api XML

- SAX = Simple Api XML

## Objetivos de la Unidad

- Comprender los fundamentos de la persistencia en sistemas de ficheros.
    
- Diferenciar entre tipos de acceso: secuencial vs aleatorio.
    
- Utilizar correctamente las clases del paquete `java.io`.
    
- Crear, leer, escribir, modificar y eliminar ficheros en Java.
    
- Trabajar con rutas relativas y valores del sistema para garantizar la portabilidad.
    
- Implementar programas que manipulen archivos de texto y binarios.
    
- Aplicar técnicas de serialización de objetos y manipulación de ficheros XML con DOM, SAX y JAXB.

## Guion de contenidos de la UD1

- [T1-Introducción_a_la_Persistencia_en_Ficheros](T1-Introducción_a_la_Persistencia_en_Ficheros.md)

- [T2-Gestión_de_Ficheros_y_Directorios_con_la_Clase_File](T2-Gestión_de_Ficheros_y_Directorios_con_la_Clase_File.md)

- [T3-Lectura_y_Escritura_Secuencial_de_Ficheros](T3-Lectura_y_Escritura_Secuencial_de_Ficheros.md)

- [T4-Acceso_Aleatorio_a_Ficheros](T4-Acceso_Aleatorio_a_Ficheros.md)

- [Extra-La_clase_Scanner_en_Java](Extra-La_clase_Scanner_en_Java.md)

- [T4.1-Alternativa_moderna_con_NIO](T4.1-Alternativa_moderna_con_NIO.md)

- [T5-Ficheros_Binarios_y_Serialización_de_Objetos](T5-Ficheros_Binarios_y_Serialización_de_Objetos.md)

- [T6-Lectura_y_escritura_de_Ficheros_XML](T6-Lectura_y_escritura_de_Ficheros_XML.md)

- [T6.1-Procesamiento_XML_con_DOM](T6.1-Procesamiento_XML_con_DOM.md)

- [T6.2-Procesamiento_XML_con_SAX](T6.2-Procesamiento_XML_con_SAX.md)

- [T7-JAXP_XPath_y_Manipulación_Avanzada_de_XML](T7-JAXP_XPath_y_Manipulación_Avanzada_de_XML.md)

- [T7.1-Profundización_en_JAXP](T7.1-Profundización_en_JAXP.md)

- [T7.2-XPath_en_profundidad](T7.2-XPath_en_profundidad.md)
    
- 8. JAXB - Mapeo de Objetos Java a XML y Viceversa
    
- 9. Buenas prácticas: Rutas relativas, buffers y portabilidad

### Anexos:

- [Recomendaciones_sobre_el_uso_de_Scanner_en_Java](Recomendaciones_sobre_el_uso_de_Scanner_en_Java.md)

- [Actividad_ArrayList-Repaso](Actividad_ArrayList-Repaso.md)

- [Exportación_de_datos-CSV_XML_JSON](Exportación_de_datos-CSV_XML_JSON.md)


> [!Actividades]
> https://github.com/IrisCampusFP/ActividadesAccesoADatos/tree/main/UD1-Persistencia_en_Ficheros