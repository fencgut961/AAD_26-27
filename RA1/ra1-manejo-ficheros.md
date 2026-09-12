# RA1. Manejo de ficheros

---

## 1. Introducción al Almacenamiento y Gestión de Ficheros

En el desarrollo de aplicaciones empresariales, la ingeniería de datos y la administración de sistemas, **los ficheros constituyen el mecanismo primario e indispensable para garantizar la persistencia de la información**. Permiten que los datos sobrevivan a la finalización de un programa o al apagado físico del hardware, sirviendo como puente de comunicación en el tiempo y el espacio.

### 1.1. ¿Qué es realmente un Fichero?
Un fichero (o archivo) es una **unidad lógica de almacenamiento** de información direccionable que reside en un dispositivo físico secundario. Su naturaleza cambia según el nivel de abstracción desde el que se analice:

```text
  👤 NIVEL DE USUARIO (Abstracción Lógica)
  ┌─────────────────────────────────────────────────────────────────┐
  │ "alumnos.csv" ➔ Archivo estructurado con filas de texto.       │
  └────────────────────────────────┬────────────────────────────────┘
                                   ▼
  💻 NIVEL DE SISTEMA OPERATIVO (Metadatos y Organización)
  ┌─────────────────────────────────────────────────────────────────┐
  │ Ruta: /var/datos/alumnos.csv                                    │
  │ Permisos: Lectura [R] | Escritura [W]                           │
  │ Metadatos: Tamaño (4 KB), Propietario, Fecha de Modificación.   │
  └────────────────────────────────┬────────────────────────────────┘
                                   ▼
  💾 NIVEL DE HARDWARE (Estructura Física de Bajo Nivel)
  ┌─────────────────────────────────────────────────────────────────┐
  │ [01001001 01000100 00101100 01001110 01101111 01101101...]      │
  │ Secuencia física e ininterrumpida de bytes en sectores de disco.│
  └─────────────────────────────────────────────────────────────────┘
```

Todo fichero cuenta obligatoriamente con los siguientes componentes de control administrados por el sistema de archivos (*file system*):
*   **Nombre e Identificador**: Cadena única que lo distingue dentro de un directorio.
*   **Ruta de Acceso (Path)**: La localización lógica jerárquica en el volumen de almacenamiento.
*   **Permisos de Acceso**: Atributos de seguridad que determinan qué usuarios o procesos pueden leer, escribir o ejecutar el archivo.
*   **Metadatos**: Información de control gestionada automáticamente por el sistema (tamaño exacto en bytes, autor/propietario, marcas de tiempo de creación y última modificación).

---

### 1.2. Evolución de la Persistencia: Del Tambor Magnético a la Nube

La interacción del software con los datos almacenados ha pasado por tres grandes eras tecnológicas:

```text
  1960 - 1970                     1980 - 2000                     2000 - Actualidad
 ┌────────────────────────┐      ┌────────────────────────┐      ┌────────────────────────┐
 │   FICHEROS PLANOS      │      │   BASES DE DATOS (SQL) │      │   SISTEMAS HÍBRIDOS    │
 ├────────────────────────┤      ├────────────────────────┤      ├────────────────────────┤
 │ • Acceso lineal/rígido │   ➔ │ • Estructura en tablas │  ➔  │ • JSON, YAML, XML, CSV │
 │ • COBOL y FORTRAN      │      │ • Transacciones ACID   │      │ • Almacén de objetos   │
 │ • Sin índices globales │      │ • Consultas complejas  │      │ • Big Data y Logs      │
 └────────────────────────┘      └────────────────────────┘      └────────────────────────┘
```

1.  **Era de los Ficheros Planos (Flat Files)**: Los datos se organizaban en registros y campos dentro de ficheros de texto o binarios sin índices globales. La manipulación era lineal y muy rígida. Lenguajes como COBOL o FORTRAN trabajaban directamente con estos ficheros.

```text
📄 Ejemplo de Registro en Fichero Plano (Longitud Fija COBOL/DAT):
  Posición de Bytes: [0...3] [4...............23] [24............38] [39......46]
  Campos Fijos:      [ ID  ] [      Nombre      ] [     Puesto     ] [  Salario ]
  Registro 1:        0001    Clara Oswald         Shop Manager       02550.00
  Registro 2:        0002    Pedro Almodovar      Clerk              01350.00
  ➔ Inconveniente: Para buscar al empleado 2, el sistema debe leer obligatoriamente los 47 bytes del Registro 1.
```

2.  **Era de las Bases de Datos Relacionales (Relational Database Management System [RDBMS])**: Sistemas como Oracle, SQL Server o MySQL superaron las limitaciones de los ficheros planos. Aportaron consultas complejas (SQL), transacciones seguras (ACID), seguridad avanzada y concurrencia multiusuario. Los ficheros directos quedaron relegados a tareas de soporte (logs, configuraciones y exportaciones).

```text
🗄️ Ejemplo de Abstracción Relacional (Tabla SQL e Índice B-Tree):
  Consulta SQL: SELECT name, salary FROM employees WHERE id = 2;
  ┌────┬────────────────┬──────────────┬────────┐
  │ ID │ Name           │ Role         │ Salary │  ➔ El motor RDBMS consulta un índice
  ├────┼────────────────┼──────────────┼────────┤     B-Tree en disco y salta de forma
  │ 1  │ Clara Oswald   │ SHOP_MANAGER │ 2550.00│     instantánea a la fila con ID=2
  │ 2  │ Pedro Almodovar│ CLERK        │ 1350.00│     sin escanear la tabla entera.
  └────┴────────────────┴──────────────┴────────┘

🗂️ Ejemplo de Índice B-Tree por ID: El B-Tree organiza los valores del índice en forma de árbol para localizar rápidamente los datos sin necesidad de recorrer toda la tabla.

                 [ 4 ]
                /     \
             [2, 3]   [6, 7]
             /   \     /   \
            1     2   5     8
                  ↑
              ID = 2
                  ↓
        ┌──────────────────┐
        │ ID = 2           │
        │ Pedro Almodovar  │
        │ CLERK            │
        │ 1350.00 €        │
        └──────────────────┘

[Simulador de Árbol B (B-Tree)](https://meskeia.com/simulador-arboles-b/)
```

3.  **Era de la Interconectividad y el Big Data**: Con la expansión de Internet y la comunicación entre sistemas heterogéneos, los ficheros volvieron a cobrar protagonismo como formato estándar de intercambio. Surgieron formatos universales legibles por humanos (CSV, XML, JSON, YAML). Además, la explosión del Big Data implicó trabajar con volúmenes masivos de datos en sistemas de archivos distribuidos (como HDFS en Hadoop) o almacenes de objetos en la nube (como Amazon S3, Google Cloud Storage o Azure Blob Storage) accesibles mediante APIs.

```text
🌐 Ejemplo de Formatos Universales de Intercambio (Misma información):

  JSON (Web & APIs REST):               YAML (Configuraciones Microservicios):
  {                                     employee:
    "id": 1,                              id: 1
    "name": "Clara Oswald",               name: Clara Oswald
    "role": "SHOP_MANAGER",               role: SHOP_MANAGER
    "salary": 2550.00                     salary: 2550.00
  }

  CSV (Tabular Plano):                  XML (Sistemas Legados Enterprise):
  id,name,role,salary                   <employee>
  1,"Clara Oswald",SHOP_MANAGER,2550.00   <id>1</id><name>Clara Oswald</name>
                                        </employee>
```

---

### 1.3. Áreas de Aplicación Actual de los Ficheros
En las arquitecturas de software modernas, los ficheros desempeñan un papel fundamental en múltiples áreas estratégicas:

*   **Persistencia Básica**: Guardar información rápida sin necesidad de desplegar una base de datos.
*   **Intercambio de Datos**: Enviar y recibir información entre plataformas heterogéneas mediante formatos estándar (CSV, JSON).
*   **Logs y Auditoría**: Registrar de forma secuencial la actividad del sistema para tareas de depuración y seguridad.

```text
📝 Ejemplo de Entrada en Fichero de Log de Servidor (access.log):
2026-09-10 10:15:22.401 [WARN] [ShopService] - User ID 42 updated inventory item #101. Response status: 200 OK
```

*   **Configuración de Aplicaciones**: Definir el comportamiento del sistema mediante ficheros legibles (formatos `.properties`, `.yaml` o `.xml`) sin necesidad de volver a compilar el código.

```yaml
# Ejemplo de Fichero de Configuración (application.yaml)
server:
  port: 8080
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/aadtex_db
```

*   **Procesamiento Masivo**: Soporte esencial para Big Data, Machine Learning y procesos de extracción, transformación y carga (ETL).
*   **Integración con la Nube**: Subir, descargar, versionar e interactuar con ficheros remotos de forma automatizada.

---

### 1.4. Almacenamiento en la Nube: Buckets y Almacenamiento de Objetos (Object Storage)

En la arquitectura de aplicaciones modernas desplegadas en la nube (AWS, Google Cloud, Microsoft Azure), la gestión de archivos ha evolucionado desde los sistemas de archivos locales hacia el **Almacenamiento de Objetos (Object Storage)** organizado en **Buckets**.

#### 🪣 ¿Qué es un Bucket y cómo se diferencia de un Sistema de Archivos Local?

Un **Bucket** es un contenedor lógico en la nube que almacena datos en forma de **objetos** dentro de un espacio de nombres totalmente plano (*flat namespace*), a diferencia de la estructura de árbol jerárquica con carpetas e inodos de los sistemas de archivos tradicionales (FAT32, NTFS, ext4).



#### 🔑 Conceptos Clave de Object Storage en la Nube:
1. **Espacio de Nombres Plano (Key-Value)**: No existen "carpetas reales" en un Bucket. La apariencia de directorios (como ) es solo un truco visual logrado usando el carácter diagonal  dentro de la **Clave (Key)** del objeto.
2. **Inmutabilidad de los Objetos**: Los archivos en un Bucket son atómicos e inmutables. No se puede modificar un byte individual a mitad de un archivo como en un disco local; para actualizar un objeto se sube una nueva versión completa que reemplaza o versiona a la anterior.
3. **Escalabilidad Infinita y Durabilidad**: Los proveedores de nube garantizan una durabilidad del 99.999999999% (los llamados "11 nueves"), replicando los bytes del objeto automáticamente en múltiples centros de datos físicos.
4. **Acceso mediante APIs REST HTTP/HTTPS**: Los archivos en la nube no se leen abriendo descriptores del sistema operativo (), sino realizando peticiones web seguras de red (, , ) mediante SDKs oficiales del proveedor.

#### 🚀 Ejemplo Práctico en Java: Operaciones con Buckets de Amazon S3 (AWS SDK v2)
Este código muestra cómo una aplicación Spring Boot interactúa con un Bucket en la nube para verificar la existencia de un contenedor, subir un archivo encriptado y descargarlo mediante el cliente oficial de AWS.



---

## 2. Tipos de Ficheros según su Contenido

Aunque a nivel de hardware todos los ficheros son secuencias binarias de bytes, la forma en que el software los interpreta nos permite dividirlos en dos grandes grupos:

### 2.1. Ficheros de Texto
Están compuestos por bytes que representan **caracteres codificados bajo un estándar específico** (normalmente UTF-8).

```text
  💡 ANALOGÍA INTUITIVA
  Un fichero de texto es como una carta escrita a mano: cualquier persona que conozca
  el alfabeto (la codificación) puede abrirla y leer su contenido directamente.
```

*   **Formatos representativos**:
    *   `.txt`: Texto plano sin estructura predefinida.
    *   `.csv`: Filas de valores separados por comas o puntos y comas.
    *   `.json`, `.xml`, `.yaml`: Estructuras clave-valor o en árbol con metadatos.
*   👍 **Ventajas**: Altamente legibles por seres humanos, fáciles de editar con cualquier herramienta básica y con una portabilidad universal absoluta entre plataformas.
*   👎 **Inconvenientes**: Consumen más espacio físico de almacenamiento y su velocidad de procesamiento es menor en grandes volúmenes de datos, ya que requieren un proceso intermedio de traducción (parseo) a objetos de memoria.

#### 🚀 Ejemplo Práctico en Java: Escritura y lectura de texto en formato UTF-8 utilizando NIO.2
Este código permite crear un archivo, escribir datos de texto en formato estructurado CSV y recuperarlos de forma portable.

```java
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.Files;
import java.nio.charset.StandardCharsets;
import java.io.IOException;

public class TextFileProcessor {
    public static void main(String[] args) {
        // Definir la ruta del fichero utilizando la API NIO.2
        Path path = Paths.get("students.csv");
        
        try {
            // Datos en formato CSV estructurado
            String csvData = "ID,Name,Role
1,Sophia,Developer
2,Marcus,Project Manager";
            
            // Escribir el contenido en el fichero forzando la codificación UTF-8
            Files.writeString(path, csvData, StandardCharsets.UTF_8);
            System.out.println("File written successfully using UTF-8.");
            
            // Leer el contenido completo del fichero en un String
            String retrievedContent = Files.readString(path, StandardCharsets.UTF_8);
            System.out.println("
--- Retrieved Content ---");
            System.out.println(retrievedContent);
            
        } catch (IOException e) {
            // Gestionar posibles excepciones de entrada y salida
            System.err.println("Error processing the text file: " + e.getMessage());
        }
    }
}
```

---

### 2.2. Ficheros Binarios
Almacenan información en formato de **bytes crudos**, codificados siguiendo una especificación técnica de bajo nivel.

```text
  💡 ANALOGÍA INTUITIVA
  Un fichero binario es como un código QR o una cinta perforada: a simple vista parece
  una secuencia incomprensible de marcas, pero un lector especializado (el software correcto)
  puede traducirlo instantáneamente en una imagen, un sonido o un programa ejecutable.
```

```text
📦 Estructura Interna de un Fichero Binario (.PNG / .CLASS / .ZIP):
  ┌────────────────────┬────────────────────┬──────────────────────────────────────┐
  │ Cabecera (Header)  │ Metadata del For.  │ Cuerpo de Datos Crudos (Payload)     │
  ├────────────────────┼────────────────────┼──────────────────────────────────────┤
  │ Bytes mágicos:     │ Ancho, Alto, Color │ Secuencia comprimida de píxeles o    │
  │ 89 50 4E 47 (.PNG) │ o Tabla de Símb.   │ instrucciones de bytecode compilado  │
  └────────────────────┴────────────────────┴──────────────────────────────────────┘
```

*   **Formatos representativos**: Imágenes (`.png`, `.jpg`), audio (`.mp3`, `.wav`), ejecutables (`.class`), comprimidos (`.zip`) o modelos de Inteligencia Artificial.
*   👍 **Ventajas**: Extremadamente compactos, eficientes en espacio y con una velocidad de lectura/escritura muy elevada al evitar transformaciones de caracteres.
*   👎 **Inconvenientes**: Totalmente ilegibles sin la herramienta específica que conozca su estructura interna. Un solo byte corrupto o desviado de su posición invalida el archivo por completo.

#### 🚀 Ejemplo Práctico en Java: Lectura binaria optimizada con Buffer intermedio
Este código muestra cómo procesar secuencialmente los bytes de un archivo binario (como una imagen) de forma eficiente y segura.

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.BufferedInputStream;
import java.io.IOException;

public class BinaryFileInspector {
    public static void main(String[] args) {
        // Referencia al archivo binario de origen
        File binaryFile = new File("image.jpg");
        
        // Verificar existencia previa para evitar fallos
        if (!binaryFile.exists()) {
            System.out.println("Please provide an 'image.jpg' file in the root directory to run this test.");
            return;
        }
        
        // Uso de try-with-resources para asegurar el cierre de flujos binarios
        try (BufferedInputStream input = new BufferedInputStream(new FileInputStream(binaryFile))) {
            byte[] buffer = new byte[1024]; // Bloque temporal de 1 KB para transferencia rápida
            int bytesRead;
            int totalBytes = 0;
            
            // Leer secuencialmente bloques de bytes hasta llegar al final (-1)
            while ((bytesRead = input.read(buffer)) != -1) {
                totalBytes += bytesRead;
            }
            
            System.out.println("Binary file read successfully.");
            System.out.println("Total physical size: " + totalBytes + " bytes.");
            
        } catch (IOException e) {
            // Controlar excepciones si ocurre algún fallo en el flujo físico
            System.err.println("Exception occurred during binary processing: " + e.getMessage());
        }
    }
}
```

#### 🖼️ Concepto Avanzado: ¿Cómo traduce el software un archivo binario para renderizarlo en la consola?
Un gran ejemplo de manipulación binaria e interpretación de datos es cargar una imagen (`image.jpg`) y dibujarla directamente en una consola de texto en base a sus píxeles físicos. El software sigue este flujo de mapeo y transformación:

```text
  [ Archivo Binario .JPG ] ➔ Descifrado de bytes ➔ [ Matriz de Píxeles de Color ]
                                                           │
        ┌──────────────────────────────────────────────────┴──────────────────────────────────────────────────┐
        ▼ (Aproximación Escala de Grises)                                                                     ▼ (Aproximación Bloques ANSI Color)
  1. Escalar la imagen a tamaño consola.                                                       1. Escalar la imagen a tamaño consola.
  2. Traducir el color de cada píxel a escala de grises.                                       2. Obtener el color exacto (Rojo, Verde, Azul).
  3. Mapear el nivel de gris a un carácter tipográfico.                                        3. Emitir el código de escape de color ANSI de 24 bits.
     - Píxel Oscuro ➔ Carácter '@' o '#'                                                           4. Imprimir un bloque sólido "█" pintado con ese color.
     - Píxel Claro  ➔ Carácter ',' o '.'
```

*   **El truco del bloque doble (▀ / ▄) para alta definición**: Un carácter de consola estándar es más alto que ancho. Para corregir la proporción del aspecto de la imagen y duplicar la resolución vertical de renderizado en consola, los programas avanzados procesan los píxeles de dos en dos verticalmente. El píxel superior define el **color de fuente** del carácter, el píxel inferior define el **color de fondo** del carácter, y se imprime el símbolo del bloque superior coloreado (`▀`). ¡Esto permite representar dos píxeles verticales reales en el espacio de un único carácter de texto!

#### 🚀 Ejemplo Práctico en Java: Renderizado en consola usando código de color ANSI y bloques dobles (▀)
Este ejemplo avanzado es un código completamente funcional que tus alumnos pueden copiar y pegar para cargar una imagen local y renderizarla a color de 24 bits dentro de su propia terminal de desarrollo de IntelliJ.

```java
import java.awt.Color;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import javax.imageio.ImageIO;

public class ConsoleImageRenderer {
    public static void main(String[] args) {
        File imageFile = new File("image.jpg");
        
        if (!imageFile.exists()) {
            System.out.println("Please place an 'image.jpg' in the root folder to see the rendering.");
            return;
        }
        
        try {
            // Leer el archivo binario JPG y decodificarlo a matriz en memoria
            BufferedImage originalImage = ImageIO.read(imageFile);
            
            // Escalar la imagen para adaptarla a las dimensiones de la consola
            int consoleWidth = 80;
            int consoleHeight = (originalImage.getHeight() * consoleWidth) / originalImage.getWidth();
            
            BufferedImage scaledImage = new BufferedImage(consoleWidth, consoleHeight, BufferedImage.TYPE_INT_RGB);
            scaledImage.getGraphics().drawImage(originalImage, 0, 0, consoleWidth, consoleHeight, null);
            
            System.out.println("--- Console 24-bit Double Block Rendering ---");
            
            // Procesar píxeles de dos en dos verticalmente para el truco de bloques dobles
            for (int y = 0; y < consoleHeight - 1; y += 2) {
                for (int x = 0; x < consoleWidth; x++) {
                    // Extraer color del píxel superior de la celda
                    Color topPixelColor = new Color(scaledImage.getRGB(x, y));
                    // Extraer color del píxel inferior de la celda
                    Color bottomPixelColor = new Color(scaledImage.getRGB(x, y + 1));
                    
                    // Ensamblar la cadena con códigos ANSI para color de fuente (top) y fondo (bottom)
                    String colorString = "[38;2;" + topPixelColor.getRed() + ";" + topPixelColor.getGreen() + ";" + topPixelColor.getBlue() + "m" +
                                         "[48;2;" + bottomPixelColor.getRed() + ";" + bottomPixelColor.getGreen() + ";" + bottomPixelColor.getBlue() + "m" +
                                         "▀"; // Carácter especial de bloque superior coloreado
                    
                    System.out.print(colorString);
                }
                // Restablecer estilos al final de cada línea de la consola
                System.out.print("[0m
");
            }
            
        } catch (IOException e) {
            System.err.println("Failed to render the image in console: " + e.getMessage());
        }
    }
}
```

---

### 2.3. Formatos Híbridos Modernos y Serialización Base64
Muchas de las estructuras que utilizamos diariamente combinan ambas tecnologías de forma transparente para ofrecer portabilidad y potencia:
*   **Formatos de Oficina (DOCX, XLSX, PPTX)**: No son un único archivo; en realidad son contenedores comprimidos (formato binario `.zip`) que albergan en su interior una jerarquía de ficheros estructurados de texto plano (XML) y recursos multimedia individuales.

```text
📂 Descomposición de un Fichero Híbrido Documento.docx (ZIP contenedor):
  Documento.docx (Renombrado a .zip y descomprimido)
  ├── [Content_Types].xml         (Texto plano XML: metadatos de tipos)
  ├── word/
  │   ├── document.xml            (Texto plano XML: el cuerpo completo del documento)
  │   ├── styles.xml              (Texto plano XML: estilos de fuente)
  │   └── media/
  │       └── image1.png          (Binario puro: imagen insertada en la página)
```

*   **Archivos PDF**: Combinan bloques de texto plano para definir la maquetación física de la página con flujos binarios comprimidos para incrustar gráficos vectoriales e imágenes.
*   **Serialización Base64**: Técnica de codificación que traduce cualquier secuencia de bytes binarios (como un archivo PDF o una imagen) en una cadena de caracteres legibles y seguros para su transmisión web. Esto permite incrustar recursos multimedia dentro de un mensaje de texto (como JSON) sin corromper el canal de comunicación.

```text
🔤 Transformación de un Archivo Binario a Cadena Base64 para JSON:
  [ Bytes Binarios Crudos ]  ➔  [ 3 Bytes = 24 bits ]  ➔  [ Dividir en 4 bloques de 6 bits ]
  01000001 01000010 01000011     010000 | 010100 | 001001 | 000011
  
  Mapear según Tabla ASCII Base64 ➔ "QUJD"
  
  Resultado incrustado en JSON:
  {
    "filename": "avatar.png",
    "mimeType": "image/png",
    "dataBase64": "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAA..."
  }
```

#### 🚀 Ejemplo Práctico en Java: Codificación y decodificación de binarios en Base64 para JSON
Este código funcional permite tomar cualquier archivo binario de imagen local, convertir sus bytes a una cadena Base64 pura y simular la recepción y restauración del archivo físico original desde esa cadena de texto.

```java
import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.Base64;

public class Base64FileProcessor {
    public static void main(String[] args) {
        Path imagePath = Paths.get("image.jpg");
        Path restoredPath = Paths.get("restored_image.jpg");

        if (!Files.exists(imagePath)) {
            System.out.println("Please provide 'image.jpg' in the root directory to test Base64 conversion.");
            return;
        }

        try {
            // 1. Leer los bytes crudos del archivo binario
            byte[] binaryBytes = Files.readAllBytes(imagePath);
            System.out.println("1. Read " + binaryBytes.length + " bytes from binary file.");

            // 2. Codificar los bytes binarios a una cadena Base64 (Texto ASCII seguro)
            String base64EncodedString = Base64.getEncoder().encodeToString(binaryBytes);
            System.out.println("2. Encoded to Base64 String. First 50 chars: " + 
                               base64EncodedString.substring(0, Math.min(50, base64EncodedString.length())) + "...");

            // 3. Simular payload JSON
            String jsonPayload = "{
  "filename": "image.jpg",
  "content": "" + base64EncodedString + ""
}";
            System.out.println("3. Simulated JSON Payload constructed successfully.");

            // 4. Decodificar la cadena Base64 de vuelta a un arreglo de bytes binarios
            byte[] decodedBytes = Base64.getDecoder().decode(base64EncodedString);

            // 5. Guardar los bytes restaurados en disco como una nueva imagen física
            Files.write(restoredPath, decodedBytes);
            System.out.println("4. Decoded bytes written to '" + restoredPath.getFileName() + "'. Matching original size: " + decodedBytes.length + " bytes.");

        } catch (IOException e) {
            System.err.println("Error processing Base64 conversion: " + e.getMessage());
        }
    }
}
```

---

### 2.4. Codificaciones de Texto: Traduciendo Bits a Caracteres y Localización (Multilenguaje)
La codificación (o juego de caracteres) es la **regla de traducción matemática** que define qué carácter gráfico corresponde a cada byte de información almacenado en el disco duro.

```text
💥 El Fenómeno "Mojibake" (Corrupción de Caracteres):
  Bytes físicos grabados en disco (UTF-8 para la palabra 'Canción'):
  [ 0x43 ] [ 0x61 ] [ 0x6E ] [ 0x63 ] [ 0x69 ] [ 0xC3 0xB3 ] [ 0x6E ]
    'C'      'a'      'n'      'c'      'i'       'ó'        'n'

  Si la aplicación lee el archivo usando la tabla ISO-8859-1 (Latin-1):
  [ 0xC3 ] ➔ Interpretado como 'Ã'
  [ 0xB3 ] ➔ Interpretado como '³'
  Resultado corrupto mostrado en pantalla: "CanciÃ³n"
```

*   **ASCII**: El estándar clásico de 7 bits. Extremadamente limitado, solo contempla 128 caracteres del alfabeto inglés básico y caracteres de control.
*   **ISO-8859-1 (Latin-1)**: Extensión de 8 bits (256 caracteres) adaptada para lenguas de Europa occidental. Presenta graves problemas de compatibilidad al migrar entre plataformas.
*   **UTF-8**: El estándar universal absoluto de ancho variable (utiliza de 1 a 4 bytes por carácter según su complejidad). Es compatible hacia atrás con ASCII y capaz de representar de forma unificada cualquier carácter del catálogo Unicode (incluyendo tildes, alfabetos asiáticos y emojis).
*   **UTF-16**: Estándar de ancho fijo (generalmente 2 bytes por carácter) que la máquina virtual de Java (JVM) utiliza internamente para representar y manipular las cadenas de texto (`String`) en la memoria RAM.

💡 **Directriz de Diseño**: Para evitar los clásicos errores de visualización de caracteres especiales (como caracteres extraños `` o fallos en las tildes), las aplicaciones deben **declarar y forzar siempre el uso de la codificación UTF-8 de manera explícita** al abrir flujos de lectura y escritura.

---

#### 🏮 El Reto de la Internacionalización: Leer Chino/Japonés y Procesarlo a Español/Inglés
Un caso real en ingeniería de software es la ingesta de ficheros provenientes de sistemas internacionales con caracteres complejos de la familia CJK (Chino, Japonés, Coreano). Si la aplicación no abre el archivo en UTF-8 o en el charset asiático específico (ej. `Shift_JIS` o `GBK`), la lectura colapsa.

#### 🚀 Ejemplo Práctico en Java: Generador y Lector Multilenguaje (Español, Inglés, Chino y Japonés)
Este programa completo e interactivo permite a los alumnos **generar automáticamente 4 ficheros de ejemplo en diferentes idiomas (Español, Inglés, Chino Simplificado y Japonés)** utilizando la codificación universal **UTF-8**, y comprobar visualmente en consola qué ocurre al leerlos con el Charset correcto versus un Charset incorrecto (efecto *Mojibake*).

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;

public class MultiLanguageCharsetDemo {

    public static void main(String[] args) {
        // Rutas de los 4 ficheros de prueba multilenguaje
        File spanishFile = new File("spanish_catalog_utf8.txt");
        File englishFile = new File("english_catalog_ascii.txt");
        File chineseFile = new File("chinese_catalog_utf8.txt");
        File japaneseFile = new File("japanese_catalog_utf8.txt");

        System.out.println("=== 1. GENERATING MULTI-LANGUAGE SAMPLE FILES ===");

        // 1. Crear Fichero en Español (con tildes, ñ, ¡, ¿, €)
        writeSampleFile(spanishFile, StandardCharsets.UTF_8, 
            "ID: 101 | Producto: Camiseta de Algodón con Cuello en V | Precio: 19.99 € | Origen: España
" +
            "ID: 102 | Producto: Pantalón Vaquero Clásico con Tildes y Ñ | Precio: 39.50 € | Estado: ¡Disponible!"
        );

        // 2. Crear Fichero en Inglés (ASCII estándar)
        writeSampleFile(englishFile, StandardCharsets.US_ASCII, 
            "ID: 201 | Item: Premium Denim Jacket | Price: 49.99 USD | Origin: USA
" +
            "ID: 202 | Item: Cotton Crewneck T-Shirt | Price: 15.00 USD | Status: In Stock"
        );

        // 3. Crear Fichero en Chino Simplificado (Familia CJK)
        writeSampleFile(chineseFile, StandardCharsets.UTF_8, 
            "ID: 301 | 商品: 亚麻混纺衬衫 (Linen Shirt) | 价格: 299.00 CNY | 发货地: 中国 (China)
" +
            "ID: 302 | 商品: 纯棉休闲长裤 (Casual Cotton Pants) | 状态: 现货 (In Stock)"
        );

        // 4. Crear Fichero en Japonés (Kanji y Hiragana)
        writeSampleFile(japaneseFile, StandardCharsets.UTF_8, 
            "ID: 401 | 商品: 富士山 シルクドレス (Silk Dress) | 価格: 8500 JPY | 原産国: 日本 (Japan)
" +
            "ID: 402 | 商品: 桜 刺繍ジャケット (Embroidered Jacket) | 数量: 15 点"
        );

        System.out.println("
=== 2. READING FILES WITH CORRECT CHARSET (UTF-8) ===");
        readAndDisplayFile(spanishFile, StandardCharsets.UTF_8, "Español (UTF-8)");
        readAndDisplayFile(chineseFile, StandardCharsets.UTF_8, "Chino (UTF-8)");
        readAndDisplayFile(japaneseFile, StandardCharsets.UTF_8, "Japonés (UTF-8)");

        System.out.println("
=== 3. DEMONSTRATING MOJIBAKE (INCORRECT CHARSET ISO-8859-1 / ASCII) ===");
        System.out.println("⚠️ Notice how CJK Chinese characters and Spanish accents get corrupted when read with ISO-8859-1:");
        readAndDisplayFile(chineseFile, StandardCharsets.ISO_8859_1, "Chino LEÍDO CON ISO-8859-1 (CORRUPTO)");
        readAndDisplayFile(spanishFile, StandardCharsets.ISO_8859_1, "Español LEÍDO CON ISO-8859-1 (CORRUPTO)");
    }

    /**
     * Escribe texto en un archivo en disco forzando un Charset determinado.
     */
    private static void writeSampleFile(File file, Charset charset, String content) {
        try (BufferedWriter writer = new BufferedWriter(
                new OutputStreamWriter(new FileOutputStream(file), charset))) {
            writer.write(content);
            System.out.println("✔ Created " + file.getName() + " using " + charset.name());
        } catch (IOException e) {
            System.err.println("Error writing " + file.getName() + ": " + e.getMessage());
        }
    }

    /**
     * Lee un archivo en disco aplicando un Charset y muestra el resultado en la consola.
     */
    private static void readAndDisplayFile(File file, Charset charset, String label) {
        System.out.println("
--- Reading [" + label + "] ---");
        try (BufferedReader reader = new BufferedReader(
                new InputStreamReader(new FileInputStream(file), charset))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            System.err.println("Error reading " + file.getName() + ": " + e.getMessage());
        }
    }
}
```

---

## 3. Acceso Clásico (`java.io`) vs. Acceso Moderno (`java.nio`)

Para interactuar con el sistema de archivos, Java proporciona dos APIs diferenciadas:

### 3.1. La API Clásica (`java.io.File`)
Representa la aproximación original basada en flujos de datos.
*   **La clase `File` como referencia de ruta**: Un objeto `File` no representa el contenido del archivo; es simplemente una abstracción de **la ruta física o dirección** del elemento en el disco.
*   **Propiedades de `File`**: Es independiente del sistema operativo, traduciendo de forma transparente las rutas con barra invertida de Windows (`C:\`) y barras de Linux (`/home/`).
*   **Limitaciones de Diseño**:
    *   No dispone de métodos eficientes y nativos para copiar o mover archivos de forma directa, requiriendo bucles manuales de bytes.
    *   Su gestión de errores es muy limitada: la mayoría de sus métodos devuelven valores de tipo verdadero/falso (booleanos) en caso de fallo en lugar de lanzar excepciones descriptivas.

#### 📋 Tabla de Métodos Críticos de la Clase `File`
Para consultar metadatos y gestionar el sistema de archivos, los alumnos utilizarán estos métodos básicos de inspección:

| Método | Tipo Retornado | Descripción Conceptual | Casos Prácticos y Detalles |
| :--- | :--- | :--- | :--- |
| **`exists()`** | `boolean` | Comprueba la existencia física del elemento. | ¿Existe el archivo de configuración de la app antes de lanzarla? |
| **`isFile()`** | `boolean` | Valida si la ruta corresponde a un fichero regular. | Evita intentar leer un directorio como si fuera un archivo de datos. |
| **`isDirectory()`**| `boolean` | Valida si la ruta corresponde a un directorio. | Útil antes de listar los contenidos de una carpeta. |
| **`getName()`** | `String` | Devuelve el nombre del recurso. | Recupera "students.csv" de una ruta absoluta larga. |
| **`getAbsolutePath()`**| `String` | Devuelve la ruta completa del sistema de archivos. | Permite responder conceptualmente a la duda de: *"¿Dónde se ha creado exactamente este archivo?"* |
| **`length()`** | `long` | Devuelve el tamaño exacto en bytes. | Devuelve `0` si el archivo no existe. |
| **`lastModified()`**| `long` | Devuelve la marca de tiempo de modificación. | Permite comprobar si el archivo de configuración ha cambiado recientemente. |
| **`delete()`** | `boolean` | Elimina el archivo o directorio de forma inmediata. | Un directorio debe estar completamente vacío para poder ser eliminado. |
| **`mkdir()`** | `boolean` | Crea el directorio final de la ruta indicada. | Falla si alguna de las carpetas intermedias de la ruta no existe. |
| **`mkdirs()`** | `boolean` | Crea toda la jerarquía de directorios intermedia. | Si creas `/datos/2025/logs/`, creará todas las carpetas que falten. |
| **`list()`** | `String[]` | Devuelve los nombres de los elementos internos. | Listado rápido de nombres de archivo en un directorio. |
| **`listFiles()`** | `File[]` | Devuelve los objetos `File` de la carpeta. | Permite recorrer recursivamente el directorio consultando metadatos individuales. |

#### 🚀 Ejemplo Práctico en Java: Inspección de metadatos de un directorio con `java.io.File`
Este código permite inspeccionar un directorio completo y listar sus archivos mostrando atributos físicos detallados en consola.

```java
import java.io.File;
import java.util.Date;

public class DirectoryInspector {
    public static void main(String[] args) {
        // Apuntar al directorio actual de ejecución
        File currentDirectory = new File(".");
        
        System.out.println("Scanning directory: " + currentDirectory.getAbsolutePath());
        
        // Obtener el listado físico de elementos contenidos en la carpeta
        File[] fileList = currentDirectory.listFiles();
        
        if (fileList != null) {
            for (File resource : fileList) {
                // Comprobar polimórficamente el tipo de recurso
                if (resource.isDirectory()) {
                    System.out.println("[DIR]  " + resource.getName());
                } else if (resource.isFile()) {
                    // Mostrar tamaño físico y fecha de última modificación
                    System.out.println("[FILE] " + resource.getName() + 
                                       " | Size: " + resource.length() + " bytes" +
                                       " | Modified: " + new Date(resource.lastModified()));
                }
            }
        } else {
            System.err.println("Unable to scan the directory. Path might be invalid.");
        }
    }
}
```

---

### 3.2. La API Moderna NIO.2 (`java.nio.file`)
Introducida para solventar las carencias del modelo clásico, separa el direccionamiento lógico del recurso de la manipulación de datos.

```text
 🏗️ Arquitectura de Desacoplamiento en NIO.2:
  ┌────────────────────────┐         ┌────────────────────────────────────────┐
  │ Interfaz Path          │ ──────► │ Dirección Lógica URI / Rutas Portables │
  └────────────────────────┘         └────────────────────────────────────────┘
              │
              ▼
  ┌────────────────────────┐         ┌────────────────────────────────────────┐
  │ Clase Utilid. Files    │ ──────► │ Operaciones Atómicas de Alto Rendim.   │
  └────────────────────────┘         └────────────────────────────────────────┘
              │
              ▼
  ┌────────────────────────┐         ┌────────────────────────────────────────┐
  │ FileSystemProvider     │ ──────► │ Abstracción: Disco Local / ZIP / S3    │
  └────────────────────────┘         └────────────────────────────────────────┘
```

*   **La interfaz `Path`**: Representa de manera lógica la ruta de localización en el disco, abstrayendo por completo el sistema operativo subyacente y permitiendo trabajar con sistemas de archivos virtuales o distribuidos en red.
*   **La clase de utilidad `Files`**: Centraliza todas las operaciones de manipulación física (copiar, mover, borrar, leer atributos avanzados) mediante métodos estáticos de alto rendimiento optimizados a nivel del sistema operativo.
*   **Integración Funcional**: Se integra de forma nativa con los flujos de datos perezosos (Java Streams), permitiendo procesar millones de registros consumiendo el mínimo espacio en la memoria RAM.

#### 📋 Métodos Destacados de la Clase de Utilidad `Files`
*   `Files.copy(Path src, Path dest, CopyOption... options)`: Copia nativa a nivel de sistema operativo.
*   `Files.move(Path src, Path dest, CopyOption... options)`: Mueve o renombra archivos atómicamente.
*   `Files.delete(Path path)`: Elimina el recurso lanzando excepciones detalladas (`NoSuchFileException`, etc.).
*   `Files.readString(Path path)` / `Files.writeString(...)`: Operaciones inmediatas para archivos de texto pequeños.
*   `Files.walk(Path start)` / `Files.find(...)`: Recorrido perezoso de árboles de directorios complejos usando Streams.

#### 🚀 Ejemplo Práctico en Java: Manipulación robusta de ficheros usando la API NIO.2
Este código demuestra cómo verificar, crear y escribir en un archivo usando el paradigma de objetos `Path` y métodos estáticos de `Files`.

```java
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.Files;
import java.io.IOException;

public class ModernFileManager {
    public static void main(String[] args) {
        // Representación lógica de la ruta mediante la interfaz Path
        Path path = Paths.get("modern_log.txt");
        
        try {
            // Comprobación segura y atómica de la existencia física del archivo
            if (!Files.exists(path)) {
                Files.createFile(path);
                System.out.println("New file created using NIO.2 at: " + path.toAbsolutePath());
            } else {
                System.out.println("File already exists. Size: " + Files.size(path) + " bytes.");
            }
            
            // Escribir contenido directamente pasando el conjunto de bytes del texto
            String logMessage = "System launched. Status: Operational.";
            Files.write(path, logMessage.getBytes());
            System.out.println("Log message written successfully.");
            
        } catch (IOException e) {
            // Gestión controlada con excepciones nativas y detalladas de NIO
            System.err.println("NIO Exception occurred: " + e.getMessage());
        }
    }
}
```

#### 🚀 Ejemplo Práctico en Java: Recorrido y búsqueda recursiva de subcarpetas con `Files.walk()`
Este código avanzado busca recursivamente todos los archivos de logs (`.txt` o `.csv`) dentro del árbol de directorios de la aplicación de forma perezosa sin agotar la RAM.

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.stream.Stream;

public class DirectoryTreeScanner {
    public static void main(String[] args) {
        Path rootPath = Paths.get("."); // Explorar desde el directorio raíz actual
        
        System.out.println("--- Recursive Directory Walk with NIO.2 Streams ---");
        
        // Files.walk abre un Stream perezoso que explora subcarpetas anidadas de forma automática
        try (Stream<Path> stream = Files.walk(rootPath, 5)) { // Profundidad máxima de 5 niveles
            
            stream.filter(Files::isRegularFile) // Filtrar solo archivos regulares (omitir carpetas)
                  .filter(p -> p.toString().endsWith(".txt") || p.toString().endsWith(".csv")) // Filtrar por extensión
                  .forEach(filePath -> {
                      try {
                          System.out.println("Found: " + filePath.getFileName() + 
                                             " | Path: " + filePath.toAbsolutePath() + 
                                             " | Size: " + Files.size(filePath) + " bytes");
                      } catch (IOException e) {
                          System.err.println("Error reading size for: " + filePath);
                      }
                  });
                  
        } catch (IOException e) {
            System.err.println("Error walking directory tree: " + e.getMessage());
        }
    }
}
```

---

## 4. Formas de Acceso a Ficheros

La forma en que el cabezal físico o el controlador de estado sólido se desplaza por la secuencia de bytes del archivo determina la estrategia de acceso:

### 4.1. Acceso Secuencial
La información se procesa en orden lineal riguroso, desde el primer byte hasta el último.

```text
  💡 ANALOGÍA INTUITIVA (Como una cinta de casete o VHS)
  Si deseas escuchar la canción de la pista 4, estás obligado a avanzar físicamente
  la cinta por encima de las pistas 1, 2 y 3. No hay forma de saltar directamente.
```

*   **Mecanismo**: El puntero avanza de forma automatizada tras cada lectura. Si se desea leer una información en la posición `N`, el software debe leer y descartar las posiciones `1` a `N-1`.
*   **Idoneidad**: Archivos de texto plano estructurados en líneas (CSV, logs, configuraciones) que requieren ser cargados por completo en la aplicación.
*   👍 **Ventaja**: Implementación sumamente sencilla.
*   👎 **Inconveniente**: Es muy ineficiente si se necesita leer o escribir datos en posiciones aleatorias de archivos de gran tamaño.

#### 🚀 Ejemplo Práctico en Java: Lectura secuencial funcional usando Java Streams (`Files.lines`)
Este código muestra cómo procesar secuencialmente archivos de texto línea por línea de manera perezosa, evitando saturar la memoria RAM.

```java
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.Files;
import java.io.IOException;
import java.util.stream.Stream;

public class SequentialStreamReader {
    public static void main(String[] args) {
        Path path = Paths.get("students.csv");
        
        // Uso de try-with-resources para asegurar el cierre automático del Stream
        try (Stream<String> linesStream = Files.lines(path)) {
            System.out.println("--- Reading File Sequentially with Streams ---");
            // Filtrar y procesar datos sobre el flujo funcional perezoso
            linesStream.filter(line -> !line.startsWith("ID")) // Omitir cabeceras
                       .forEach(line -> System.out.println("Processed line: " + line));
                       
        } catch (IOException e) {
            System.err.println("Error processing the sequential stream: " + e.getMessage());
        }
    }
}
```

---

### 4.2. Acceso Aleatorio (o Acceso Directo)
Permite posicionar el puntero de lectura/escritura en cualquier byte arbitrario del archivo de forma instantánea, sin necesidad de recorrer la información previa.

```text
  💡 ANALOGÍA INTUITIVA (Como un disco de vinilo o un CD)
  Puedes levantar la aguja o el láser y colocarlo directamente sobre el inicio de la pista 4,
  reproduciendo la música sin perder tiempo en recorrer las pistas anteriores.
```

*   **Idoneidad**: Archivos binarios de estructura uniforme donde cada registro ocupa un tamaño exacto y predecible (bases de datos locales, índices de búsqueda).
*   **El concepto del direccionamiento por bytes**: En Java, esto se gestiona mediante la clase **`RandomAccessFile`**, configurando el modo de acceso en lectura o escritura combinada (`"r"` o `"rw"`). Utiliza el método de reposicionamiento de puntero `seek(posición_en_bytes)`.

#### 🧮 Concepto Teórico: Las Matemáticas del Acceso Aleatorio
Para que el acceso aleatorio sea viable, los registros deben tener un **tamaño fijo en bytes**. Imagina un archivo de datos binario donde guardamos fichas de empleados. Cada registro consta de tres campos fijos:
*   `ID` (tipo entero: ocupa **4 bytes**)
*   `Edad` (tipo entero: ocupa **4 bytes**)
*   `Salario` (tipo real de precisión doble: ocupa **8 bytes**)
*   **Tamaño total del registro**: `4 + 4 + 8 = 16 bytes`

```text
   Posición en bytes del archivo:
   0               16              32              48 bytes
  ┌───────────────┬───────────────┬───────────────┐
  │  Registro 0   │  Registro 1   │  Registro 2   │
  │  (Empleado 1) │  (Empleado 2) │  (Empleado 3) │
  └───────────────┴───────────────┴───────────────┘
   ▲               ▲               ▲
   │               │               │
   │               │               └─ Saltar al Registro 2 ➔ seek(2 * 16) = seek(32 bytes)
   │               └─ Saltar al Registro 1 ➔ seek(1 * 16) = seek(16 bytes)
   └─ Saltar al Registro 0 ➔ seek(0 * 16) = seek(0 bytes)
```

*   **¿Cómo saltaríamos directamente a leer el salario del tercer empleado (Registro 2)?**
    *   Primero saltamos al inicio del Registro 2: `2 * 16 bytes = 32 bytes`.
    *   Como el salario está después del `ID` (4 bytes) y de la `Edad` (4 bytes), sumamos ese desplazamiento intermedio (*offset*): `32 + 8 = 40 bytes`.
    *   Colocamos el puntero directamente allí mediante un salto: `seek(40)`. ¡Leemos la información en microsegundos sin importar el volumen total del archivo!

#### 🚀 Ejemplo Práctico en Java: Lectura, escritura y modificación aleatoria de datos binarios
Este código demuestra cómo registrar datos binarios con estructura de tamaño fijo y modificar un registro intermedio de forma directa y atómica en disco.

```java
import java.io.RandomAccessFile;
import java.io.IOException;

public class RandomAccessManager {
    public static void main(String[] args) {
        String filename = "employees.dat";
        
        // Estructura de registro de tamaño fijo: ID (4B) + Age (4B) + Salary (8B) = 16 Bytes
        final int RECORD_SIZE = 16;
        
        try (RandomAccessFile raf = new RandomAccessFile(filename, "rw")) {
            // Vaciar el archivo antes de comenzar a escribir para la simulación
            raf.setLength(0);
            
            // --- ESCRITURA DE DATOS EN DISCO ---
            // Registro 0 (Clara): ID 1, Edad 25, Salario 100.5
            raf.writeInt(1);
            raf.writeInt(25);
            raf.writeDouble(100.5);
            
            // Registro 1 (Pedro): ID 2, Edad 30, Salario 200.0
            raf.writeInt(2);
            raf.writeInt(30);
            raf.writeDouble(200.0);
            
            // Registro 2 (Marcus): ID 3, Edad 40, Salario 500.75
            raf.writeInt(3);
            raf.writeInt(40);
            raf.writeDouble(500.75);
            
            System.out.println("Three records written in fixed sizes (16 bytes each).");
            
            // --- ACCESO DIRECTO ALEATORIO ---
            // Saltar directamente al inicio del Registro 1 (segundo empleado)
            raf.seek(1 * RECORD_SIZE);
            
            int id = raf.readInt();
            int age = raf.readInt();
            double salary = raf.readDouble();
            
            System.out.println("
--- Directly read Record 1 ---");
            System.out.println("ID: " + id + " | Age: " + age + " | Salary: " + salary + " EUR");
            
            // --- MODIFICACIÓN DIRECTA ---
            // Modificar el salario del tercer empleado (Registro 2)
            // Offset: Saltar al inicio de Registro 2 (2 * 16 = 32B) + Saltarse ID y Edad (8B) = 40B
            long offset = (2 * RECORD_SIZE) + 8;
            raf.seek(offset);
            raf.writeDouble(9999.99); // Sobrescribir el campo salario directamente
            
            System.out.println("
Modified record 2 salary directly in disk.");
            
            // --- LEER TODOS LOS REGISTROS PARA VERIFICAR ---
            raf.seek(0); // Volver al inicio físico del archivo
            System.out.println("
--- Final Employee Records ---");
            for (int i = 0; i < 3; i++) {
                int currentId = raf.readInt();
                int currentAge = raf.readInt();
                double currentSalary = raf.readDouble();
                System.out.println("ID: " + currentId + " | Age: " + currentAge + " | Salary: " + currentSalary + " EUR");
            }
            
        } catch (IOException e) {
            System.err.println("Random access operation failed: " + e.getMessage());
        }
    }
}
```

---

### 4.3. Comparativa de Estrategias

| Característica | Acceso Secuencial | Acceso Aleatorio |
| :--- | :--- | :--- |
| **Forma de Lectura** | De principio a fin. | Posiciones arbitrarias. |
| **Velocidad de Búsqueda** | Lenta en datos intermedios de archivos grandes. | Rápida para saltos concretos. |
| **Complejidad de Gestión** | Baja, el puntero avanza de forma automatizada. | Alta, requiere calcular posiciones físicas en bytes. |
| **Uso Típico** | Ficheros de configuración, CSV, XML, JSON, logs. | Bases de datos indexadas, archivos multimedia. |

---

### 4.4. Acceso Combinado en Aplicaciones Modernas
En sistemas de producción reales de alta escala, ambas estrategias conviven de forma natural para optimizar el rendimiento global:
*   **Flujos de Big Data**: Procesamiento secuencial masivo de eventos continuos en colas de mensajería (como Apache Kafka) o logs distribuidos.
*   **Motores de Búsqueda (Elasticsearch, Lucene)**: Utilizan acceso secuencial para persistir sus registros de transacciones en disco, mientras que realizan búsquedas de acceso aleatorio constante sobre los ficheros de índices para localizar registros en milisegundos.
*   **Simulación en la Nube**: Los almacenes de objetos en la nube (S3) permiten realizar descargas secuenciales completas del recurso o simular accesos aleatorios solicitando únicamente rangos específicos de bytes del objeto persistido a través de cabeceras seguras de red HTTP, ahorrando ancho de banda y latencia.

---

## 5. El Ciclo de Vida de las Operaciones sobre Ficheros

Toda interacción de entrada/salida (E/S) entre el programa y el soporte físico de almacenamiento sigue un flujo compuesto por cuatro fases obligatorias:

```text
 1. APERTURA                  2. PROCESAMIENTO             3. DESPLAZAMIENTO            4. CIERRE
 Solicitar al sistema de      Transferencia de datos       Posicionar manualmente el    Liberar el descriptor,
 archivos un descriptor y     (lectura/escritura) en       puntero mediante direcciones  desbloquear el archivo
 un canal de comunicación. bloques, líneas o bytes. de bytes (opcional).  y volcar las cachés.
```

1.  **Apertura**: El programa solicita un puntero de conexión al sistema de archivos instanciando un flujo de datos (*stream*). Según el caso de uso, el flujo se abre bajo perfiles de solo lectura, escritura destructiva (sobrescritura) o modo de adición (*append*, que posiciona automáticamente el puntero al final del archivo para preservar los datos existentes).
2.  **Procesamiento (Lectura/Escritura)**: Se realiza la transferencia de información en bloques, líneas o caracteres. El puntero físico avanza automáticamente tras cada operación. Al alcanzar el límite del recurso, el sistema operativo devuelve un valor centinela estándar de finalización (EOF, habitualmente representado por el valor `-1` en los flujos de lectura).
3.  **Desplazamiento o Salto (Opcional)**: En entornos de acceso aleatorio, se recoloca de forma manual el puntero físico a una posición en bytes específica antes de realizar la siguiente operación de lectura o escritura.
4.  **Cierre**: Es la fase más crítica del ciclo de vida. Consiste en ordenar la liberación del descriptor del archivo en el sistema operativo, el desbloqueo del recurso y el volcado definitivo (*flush*) a disco de cualquier byte temporal que estuviera retenido en la caché de la memoria RAM del sistema.

---

### ⚠️ El peligro latente: Fugas de Recursos (*Resource Leaks*)
Si el software omite la fase de cierre de los canales de comunicación o esta falla en mitad de la ejecución:

```text
💥 Consecuencia de Omitir el Cierre (Saturación de Descriptores en el Kernel):
  Proceso Java ➔ Abre Stream sin close() ➔ Ocupa Descriptor FD 3
  Proceso Java ➔ Abre Stream sin close() ➔ Ocupa Descriptor FD 4
  ...
  Proceso Java ➔ Abre Stream sin close() ➔ Ocupa Descriptor FD 1024 [LÍMITE ALCANZADO]
  
  Resultante: java.io.IOException: Too many open files (Colapso del servidor)
```

*   El archivo puede quedar **bloqueado indefinidamente** por el sistema operativo, impidiendo su edición por otros procesos o la propia aplicación.
*   Se producirá una **saturación de los descriptores de archivos** en el núcleo del sistema, provocando inestabilidad y caídas en el servidor de aplicaciones.
*   Existe un alto riesgo de **corrupción o pérdida de datos** al no garantizar el volcado final (*flush*) de las memorias RAM volátiles intermedias al disco duro físico.

```text
🛡️ Garantía de Seguridad con Try-With-Resources (Java 7+):
  try (BufferedWriter writer = new BufferedWriter(new FileWriter("app.log"))) {
      writer.write("Operational log");
  } // <--- El compilador inyecta un bloque finally que ejecuta writer.close()
    //      de forma garantizada, incluso si se lanza una RuntimeException.
```

💡 **Directriz de Diseño (Try-With-Resources)**: Para automatizar el cierre seguro, el software moderno se estructura mediante bloques de control autocerrables. Al implementar las clases de flujos la interfaz `AutoCloseable`, el compilador garantiza la liberación inmediata de todos los recursos en disco al finalizar las operaciones de forma transparente para el programador, incluso ante excepciones imprevistas en tiempo de ejecución.

#### 🚀 Ejemplo Práctico en Java: Escritura de texto segura utilizando Try-With-Resources
Este código muestra cómo garantizar que el archivo se cierre de manera automática y limpia pase lo que pase durante la escritura física de datos.

```java
import java.io.FileWriter;
import java.io.BufferedWriter;
import java.io.IOException;

public class SafeFileWriter {
    public static void main(String[] args) {
        // La inicialización en la firma del try garantiza la autoliberación al finalizar
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("safe_output.txt", true))) {
            // Escribir contenido y realizar un salto de línea nativo del sistema
            writer.write("Safe transaction recorded.");
            writer.newLine();
            System.out.println("Data saved successfully. Stream closed automatically.");
            
        } catch (IOException e) {
            // Control de excepciones en caso de fallo físico de escritura
            System.err.println("Failed to write securely to disk: " + e.getMessage());
        }
    }
}
```

---

## 6. Jerarquía de Flujos de Datos (*Streams*) y Patrón Decorador

El canal de comunicación unidireccional entre un programa y un archivo físico se conceptualiza como un **Flujo de Datos (Stream)**. Java clasifica estos flujos según la unidad de información con la que operan:

```text
                              ┌────────────────────────┐
                              │  FLUJOS DE DATOS (I/O) │
                              └───────────┬────────────┘
                                          │
                  ┌───────────────────────┴───────────────────────┐
                  ▼                                               ▼
         [ Flujos de Texto ]                             [ Flujos Binarios ]
     Manejan caracteres de 16 bits.            Manejan bytes crudos de 8 bits.
     Clases Base: Reader y Writer.         Clases Base: InputStream y OutputStream.
```

### 6.1. Flujos de Texto
Se utilizan de forma exclusiva para interactuar con archivos que albergan caracteres de texto legibles.

*   **`FileReader` / `FileWriter`** (Acceso Directo Sin Buffer): Leen o escriben caracteres directamente sobre el disco. Son sencillos de instanciar pero ineficientes si se realizan operaciones continuas de pocos caracteres, ya que provocan llamadas constantes al hardware.
*   **`BufferedReader` / `BufferedWriter`** (Acceso Optimizado Con Buffer): Envuelven a los flujos directos y añaden una caché en memoria RAM. Permiten operaciones eficientes de alto nivel, como leer el archivo cómodamente línea a línea mediante cadenas de texto o añadir saltos de línea automáticos del sistema operativo.

---

### 6.2. Flujos Binarios
Se utilizan para manipular archivos que contienen secuencias de bytes crudos sin interpretar como texto plano.

*   **`FileInputStream` / `FileOutputStream`** (Acceso Directo Sin Buffer): Leen o escriben bytes de forma directa en el dispositivo de almacenamiento físico.
*   **`BufferedInputStream` / `BufferedOutputStream`** (Acceso Optimizado Con Buffer): Gestionan lecturas y escrituras binarias masivas agrupando los bytes en bloques de memoria RAM intermedia para proteger el rendimiento de la unidad de almacenamiento.

---

### 6.3. Tabla Comparativa de Flujos

| Característica | Flujos de Texto | Flujos Binarios |
| :--- | :--- | :--- |
| **Datos Manejados** | Caracteres Unicode (16 bits). | Bytes crudos sin interpretar (8 bits). |
| **Clases Base** | `Reader` y `Writer`. | `InputStream` y `OutputStream`. |
| **Uso Común** | Archivos XML, JSON, YAML, CSV y logs de texto. | Imágenes, vídeos, audios, ZIP y ejecutables. |
| **Gran Ventaja** | Sencilla manipulación y legibilidad directa. | Eficiencia total, compatible con cualquier tipo de dato. |
| **Inconveniente** | El proceso de codificación puede corromper datos si es erróneo. | Son totalmente ilegibles de forma directa en disco. |

---

### 6.4. El Patrón Decorador (Wrapper) en `java.io`: ¿Por qué anidamos flujos?
Una de las preguntas más frecuentes de los alumnos al aprender Java I/O es: *¿Por qué debo instanciar tres objetos diferentes para leer una simple línea de texto?*

```java
BufferedReader reader = new BufferedReader(
                            new InputStreamReader(
                                new FileInputStream("datos.txt"), StandardCharsets.UTF_8));
```

La respuesta es el **Patrón de Diseño Decorador (Decorator Pattern)**. En lugar de crear una clase gigante que lo haga todo, Java separa las responsabilidades en capas anidadas ("muñecas matrioshka"):

```text
🧩 Ensamblaje de Capas en el Patrón Decorador de java.io:

  ┌────────────────────────────────────────────────────────────────────────┐
  │ 3. CAPA BUFFEADA (BufferedReader)                                       │
  │    Agrega la funcionalidad de caché de 8 KB en RAM y readLine().       │
  │  ┌──────────────────────────────────────────────────────────────────┐  │
  │  │ 2. CAPA PUENTE DE RECODIFICACIÓN (InputStreamReader)             │  │
  │  │    Traduce los bytes entrantes a caracteres usando UTF-8.        │  │
  │  │  ┌────────────────────────────────────────────────────────────┐  │  │
  │  │  │ 1. CAPA ACCESO FÍSICO (FileInputStream)                    │  │  │
  │  │  │    Lee la secuencia de bytes crudos del disco.              │  │  │
  │  │  └────────────────────────────────────────────────────────────┘  │  │
  │  └──────────────────────────────────────────────────────────────────┘  │
  └────────────────────────────────────────────────────────────────────────┘
```

#### 🚀 Ejemplo Práctico en Java: Demostración paso a paso de la composición del Patrón Decorador
Este código desacopla las tres capas del patrón decorador para mostrar a los alumnos cómo cada objeto envuelve al anterior añadiendo una nueva responsabilidad.

```java
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;

public class StreamDecoratorDemo {
    public static void main(String[] args) {
        File file = new File("students.csv");

        if (!file.exists()) {
            System.out.println("Please generate 'students.csv' first using TextFileProcessor.");
            return;
        }

        try {
            // CAPA 1: Flujo de entrada de bytes crudos desde el soporte físico
            FileInputStream rawByteStream = new FileInputStream(file);
            System.out.println("Capa 1: FileInputStream creado (Acceso a bytes crudos del disco).");

            // CAPA 2: Traductor puente de bytes a caracteres con Charset explícito
            InputStreamReader characterBridgeReader = new InputStreamReader(rawByteStream, StandardCharsets.UTF_8);
            System.out.println("Capa 2: InputStreamReader envuelve a Capa 1 (Traducción de Charset a UTF-8).");

            // CAPA 3: Almacenamiento intermedio en RAM y métodos de alto nivel
            BufferedReader bufferedLineReader = new BufferedReader(characterBridgeReader);
            System.out.println("Capa 3: BufferedReader envuelve a Capa 2 (Añade memoria caché y método readLine()).
");

            // Leer usando el objeto decorado final
            String line;
            System.out.println("--- Reading lines from fully decorated stream ---");
            while ((line = bufferedLineReader.readLine()) != null) {
                System.out.println("Line: " + line);
            }

            // Cerrar la capa exterior cierra automáticamente todas las capas interiores
            bufferedLineReader.close();
            System.out.println("
All stream layers closed safely.");

        } catch (IOException e) {
            System.err.println("Error in stream decorator pipeline: " + e.getMessage());
        }
    }
}
```

---

### 6.5. Optimización mediante el Patrón Buffering (Almacenamiento Intermedio)
La interacción física directa con unidades de disco para escribir o leer datos de uno en uno es extremadamente ineficiente.

```text
  Lectura Sin Buffer: (Petición constante al disco físico - Lento)
  Programa Java <====== (Petición de 1 Byte) ======> Disco Físico (SSD/HDD)

  Lectura Con Buffer: (Lectura en bloques a memoria intermedia RAM - Rápido)
  Programa Java <== (Servicio instantáneo en RAM) == Buffer (8 KB) <== (Volcado físico) == Disco Físico
```

Para mitigar esta penalización, las clases con Buffer actúan de la siguiente manera:
*   **Lectura con Buffer**: En lugar de solicitar al disco un solo byte a la vez, el buffer realiza una petición física masiva de un bloque de datos sustancial (por ejemplo, 8 Kilobytes de una sola vez) y los almacena en la memoria RAM rápida. Las sucesivas lecturas del programa se sirven instantáneamente de la RAM, acelerando el rendimiento general de forma exponencial.
*   **Escritura con Buffer**: El buffer retiene de manera temporal las escrituras en la memoria volátil del sistema y solo ejecuta la costosa operación de volcado físico en disco (*flush*) de forma masiva cuando el buffer se satura o se ordena el cierre definitivo del flujo.

#### 🚀 Ejemplo Práctico en Java: Copia eficiente de archivos binarios utilizando Buffers físicos
Este código muestra cómo clonar una imagen en disco procesando sus bytes de forma masiva a través de almacenamiento intermedio para garantizar la máxima velocidad.

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.IOException;

public class BinaryFileCopier {
    public static void main(String[] args) {
        File source = new File("image.jpg");
        File destination = new File("image_copy.jpg");
        
        if (!source.exists()) {
            System.out.println("Please provide 'image.jpg' to execute the copy test.");
            return;
        }
        
        // Inicializar flujos binarios con buffer intermedio
        try (
            BufferedInputStream reader = new BufferedInputStream(new FileInputStream(source));
            BufferedOutputStream writer = new BufferedOutputStream(new FileOutputStream(destination))
        ) {
            byte[] cacheBuffer = new byte[4096]; // Buffer de transferencia masiva de 4 KB
            int bytesTransferred;
            
            // Leer y escribir bloques de bytes hasta alcanzar el final del archivo
            while ((bytesTransferred = reader.read(cacheBuffer)) != -1) {
                writer.write(cacheBuffer, 0, bytesTransferred);
            }
            
            System.out.println("Binary file copied successfully with high performance.");
            
        } catch (IOException e) {
            System.err.println("Copy failed due to physical disk error: " + e.getMessage());
        }
    }
}
```

---

## 7. Clases con Recodificación de Caracteres

Cuando un programa interactúa con un archivo de texto en disco, debe realizar obligatoriamente un proceso de traducción bidireccional entre la secuencia binaria de almacenamiento físico y la representación tipográfica de los caracteres Unicode que maneja internamente la memoria RAM del sistema:

```text
                     ┌─────────────────────────────┐
                     │   Bytes físicos en disco    │
                     └──────────────┬──────────────┘
                                    │
                                    ▼
                     ┌─────────────────────────────┐
                     │     InputStreamReader       │
                     │  (Aplica regla de Charset)  │
                     └──────────────┬──────────────┘
                                    │
                                    ▼
                     ┌─────────────────────────────┐
                     │       BufferedReader        │
                     │  (Lectura de líneas en RAM) │
                     └──────────────┬──────────────┘
                                    │
                                    ▼
                     ┌─────────────────────────────┐
                     │     Caracteres Unicode      │
                     └─────────────────────────────┘
```

Para coordinar este proceso de forma segura sin riesgo de caracteres corruptos, se utilizan clases puente de conversión especializadas:
*   **`InputStreamReader`**: Actúa como un traductor que toma un flujo de bytes binarios entrantes (`InputStream`) y los transforma de forma dinámica en caracteres legibles bajo una codificación o juego de caracteres específico.
*   **`OutputStreamWriter`**: Realiza el proceso inverso; toma caracteres Unicode de la memoria del programa y los codifica en la secuencia de bytes binarios correspondiente al charset de destino al escribir físicamente en disco.

💡 **Directriz de Diseño**: Si se omite especificar de forma explícita el juego de caracteres en los flujos puente, la máquina virtual recurrirá de manera silenciosa a la configuración por defecto de la plataforma local. Esto compromete gravemente la portabilidad del sistema al migrar el código entre servidores con diferentes sistemas operativos. **Especificar siempre constantes explícitas y seguras en la recodificación (como `StandardCharsets.UTF_8`) es un requisito indispensable en el desarrollo de software profesional**.

#### 🚀 Ejemplo Práctico en Java: Recodificación y transcodificación física de archivos
Este código lee un archivo codificado originalmente en formato UTF-8 y genera de manera simultánea dos copias de seguridad en codificaciones diferentes (UTF-16 e ISO-8859-1), demostrando el puente bidireccional de caracteres.

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

public class EncodingTranscoder {
    public static void main(String[] args) {
        File sourceFile = new File("students.csv");
        File utf16OutputFile = new File("students_utf16.csv");
        File isoOutputFile = new File("students_iso.csv");
        
        if (!sourceFile.exists()) {
            System.out.println("Please run 'TextFileProcessor' first to generate 'students.csv'.");
            return;
        }
        
        // Encadenar clases puente especificando el conjunto de caracteres explícito
        try (
            BufferedReader reader = new BufferedReader(
                new InputStreamReader(new FileInputStream(sourceFile), StandardCharsets.UTF_8));
            BufferedWriter utf16Writer = new BufferedWriter(
                new OutputStreamWriter(new FileOutputStream(utf16OutputFile), StandardCharsets.UTF_16));
            BufferedWriter isoWriter = new BufferedWriter(
                new OutputStreamWriter(new FileOutputStream(isoOutputFile), "ISO-8859-1"))
        ) {
            String currentLine;
            
            // Leer en UTF-8 y volcar transcodificando de forma simultánea en ambos formatos
            while ((currentLine = reader.readLine()) != null) {
                // Escribir en la copia UTF-16
                utf16Writer.write(currentLine);
                utf16Writer.newLine();
                
                // Escribir en la copia ISO-8859-1
                isoWriter.write(currentLine);
                isoWriter.newLine();
            }
            
            System.out.println("Transcoding completed successfully.");
            System.out.println("Generated File 1: students_utf16.csv (UTF-16 encoding)");
            System.out.println("Generated File 2: students_iso.csv (ISO-8859-1 encoding)");
            
        } catch (IOException e) {
            System.err.println("Transcoding failed: " + e.getMessage());
        }
    }
}
```

---

## 8. Seguridad y Confidencialidad: Encriptación de Ficheros

En entornos profesionales de desarrollo y administración de sistemas, **la persistencia y la transmisión de datos sensibles exige mecanismos de protección criptográfica**. Almacenar contraseñas, credenciales de conexión, datos personales o registros corporativos en texto plano en el sistema de archivos supone un grave riesgo de seguridad.

Para proteger los datos se emplean dos grandes aproximaciones criptográficas:
1.  **Criptografía Simétrica**: Utiliza una única **clave compartida** (o frase de paso) tanto para encriptar como para desencriptar. Es idónea para cifrar grandes volúmenes de datos locales con alta velocidad. El algoritmo estándar es **AES (Advanced Encryption Standard)**.
2.  **Criptografía Asimétrica (o de Clave Pública)**: Utiliza una pareja de claves vinculadas matemáticamente:
    *   **Clave Pública**: Se comparte libremente con los compañeros o sistemas externos. Cualquiera puede usarla para **encriptar** un archivo o mensaje dirigido a nosotros.
    *   **Clave Privada**: Se mantiene en estricto secreto. Es la **única** capaz de **desencriptar** los archivos cifrados con su correspondiente clave pública.
    *   El estándar de referencia en la industria es **RSA / OpenPGP (GnuPG)**.

---

### 8.1. Práctica de Aula: Gestión y Cifrado con GPG (GnuPG) mediante Consola de Comandos

GPG (*GNU Privacy Guard*) es la herramienta estándar en entornos Linux y servidores backend para proteger archivos mediante criptografía. Permite dos estrategias clave: **cifrado simétrico** (rápido por contraseña) y **cifrado asimétrico** (par de claves pública/privada).

---

#### 🔑 Caso 1: Cifrado Simétrico Rápido (Con Contraseña / Passphrase)
Ideal para proteger un archivo local de forma inmediata antes de almacenarlo o transferirlo, sin necesidad de gestionar un llavero de claves:

```bash
# 1. Crear un archivo de texto con datos confidenciales
echo "CONFIDENTIAL: Exam grades for Acceso a Datos 2026" > grades.txt

# 2. Cifrar de forma simétrica usando GPG (solicitará una contraseña en pantalla)
gpg --symmetric --cipher-algo AES256 grades.txt

# ➔ Resultado: Se genera un archivo binario encriptado llamado 'grades.txt.gpg'

# 3. Eliminar de forma segura el archivo original en texto plano
rm grades.txt

# 4. Intentar visualizar el archivo encriptado (se verán caracteres binarios ilegibles)
cat grades.txt.gpg

# 5. Desencriptar el archivo para recuperar la información original
gpg --decrypt grades.txt.gpg > grades_recovered.txt

# 6. Comprobar el contenido recuperado
cat grades_recovered.txt
```

---

#### 🔐 Caso 2: Cifrado Asimétrico de Clave Pública (Intercambio entre Alumnos)
Para simular el flujo real de transferencia segura entre dos entidades (Alumno A y Alumno B):

```text
       ALUMNO A                                                     ALUMNO B
  ┌──────────────────┐                                         ┌──────────────────┐
  │ 1. Genera par de │                                         │ 1. Genera par de │
  │    claves GPG    │                                         │    claves GPG    │
  └────────┬─────────┘                                         └────────┬─────────┘
           │                                                            │
           │  ──────── Envía su Clave Pública (student_a.key) ────────► │
           │                                                            │
           │                                                   ┌────────┴─────────┐
           │                                                   │ 2. Importa clave │
           │                                                   │    pública de A  │
           │                                                   └────────┬─────────┘
           │                                                            │
           │                                                   ┌────────┴─────────┐
           │                                                   │ 3. Cifra archivo │
           │                                                   │    con clave de A│
           │                                                   └────────┬─────────┘
           │                                                            │
           │  ◄────── Recibe mensaje cifrado (secret.txt.gpg) ───────── │
  ┌────────┴─────────┐
  │ 4. Desencripta   │
  │    con su clave  │
  │    privada secret│
  └──────────────────┘
```

**Comandos de Consola para Reproducir en Clase:**

```bash
# === EN EL EQUIPO DEL ALUMNO A ===
# 1. Generar la pareja de claves criptográficas (pública y privada)
gpg --generate-key

# 2. Exportar la clave pública a un archivo para enviársela al Alumno B
gpg --output student_a_public.key --export student.a@school.com

# === EN EL EQUIPO DEL ALUMNO B ===
# 3. Importar la clave pública recibida del Alumno A
gpg --import student_a_public.key

# 4. Crear un archivo con un mensaje secreto para el Alumno A
echo "Hola Alumno A, este mensaje solo lo puedes leer tú con tu clave privada." > secret_message.txt

# 5. Cifrar el archivo usando la CLAVE PÚBLICA del Alumno A
gpg --recipient student.a@school.com --encrypt secret_message.txt
# ➔ Genera el archivo encriptado 'secret_message.txt.gpg' que se envía al Alumno A

# === EN EL EQUIPO DEL ALUMNO A ===
# 6. Desencriptar el archivo recibido utilizando SU CLAVE PRIVADA
gpg --decrypt secret_message.txt.gpg > message_read.txt

# 7. Verificar el contenido desencriptado
cat message_read.txt
```

---


### 8.2. Ejemplo en Java: Cifrado Asimétrico de Ficheros (RSA)

Para comprender cómo funciona el cifrado de clave pública dentro de una aplicación Java, implementaremos un ejemplo autónomo y sencillo (`AsymmetricFileCrypto.java`). 

El programa:
1. Genera un par de claves **RSA (Pública y Privada)** de 2048 bits.
2. Utiliza la **Clave Pública** para cifrar un archivo de texto (`secret_raw.txt`), generando el fichero protegido (`secret_encrypted.enc`).
3. Utiliza la **Clave Privada** para descifrar el fichero (`secret_encrypted.enc`), recuperando los datos originales en (`secret_decrypted.txt`).

#### 🚀 Código Completo en Java: `AsymmetricFileCrypto.java`

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.PrivateKey;
import java.security.PublicKey;
import javax.crypto.Cipher;

public class AsymmetricFileCrypto {

    public static void main(String[] args) {
        File rawFile = new File("secret_raw.txt");
        File encryptedFile = new File("secret_encrypted.enc");
        File decryptedFile = new File("secret_decrypted.txt");

        try {
            // 1. Crear un archivo de texto original para la prueba
            String secretMessage = "Confidential Data: RSA Asymmetric Encryption Test in Java 2026";
            try (FileOutputStream fos = new FileOutputStream(rawFile)) {
                fos.write(secretMessage.getBytes(StandardCharsets.UTF_8));
            }
            System.out.println("1. Raw text file created: " + rawFile.getName());

            // 2. Generar par de claves RSA de 2048 bits (Clave Pública y Clave Privada)
            KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
            keyPairGen.initialize(2048);
            KeyPair keyPair = keyPairGen.generateKeyPair();
            
            PublicKey publicKey = keyPair.getPublic();   // Se usa para ENCRIPTAR
            PrivateKey privateKey = keyPair.getPrivate(); // Se usa para DESENCRIPTAR
            System.out.println("2. RSA 2048-bit KeyPair generated successfully.");

            // 3. ENCRIPTAR el archivo usando la CLAVE PÚBLICA
            encryptFile(rawFile, encryptedFile, publicKey);
            System.out.println("3. File ENCRYPTED with Public Key -> Saved to: " + encryptedFile.getName());

            // 4. DESENCRIPTAR el archivo usando la CLAVE PRIVADA
            decryptFile(encryptedFile, decryptedFile, privateKey);
            System.out.println("4. File DECRYPTED with Private Key -> Saved to: " + decryptedFile.getName());

            // 5. Leer y verificar el contenido recuperado
            try (FileInputStream fis = new FileInputStream(decryptedFile)) {
                String recoveredText = new String(fis.readAllBytes(), StandardCharsets.UTF_8);
                System.out.println("
--- Recovered Content Verification ---");
                System.out.println(recoveredText);
            }

        } catch (Exception e) {
            System.err.println("Error during RSA cryptographic processing: " + e.getMessage());
        }
    }

    /**
     * Encripta un archivo físico utilizando la Clave Pública.
     * 
     * @param inputFile Fichero en texto plano a cifrar
     * @param outputFile Fichero de salida cifrado
     * @param publicKey Clave pública receptora para realizar el cifrado
     */
    public static void encryptFile(File inputFile, File outputFile, PublicKey publicKey) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, publicKey);
        processFile(inputFile, outputFile, cipher);
    }

    /**
     * Desencripta un archivo físico cifrado utilizando la Clave Privada correspondiente.
     * 
     * @param inputFile Fichero cifrado
     * @param outputFile Fichero de salida con el texto restaurado
     * @param privateKey Clave privada secreta para realizar el descifrado
     */
    public static void decryptFile(File inputFile, File outputFile, PrivateKey privateKey) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        processFile(inputFile, outputFile, cipher);
    }

    /**
     * Lee los bytes del fichero de entrada, aplica la transformación con Cipher y escribe en el de salida.
     */
    private static void processFile(File inputFile, File outputFile, Cipher cipher) throws IOException, Exception {
        try (
            FileInputStream inputStream = new FileInputStream(inputFile);
            FileOutputStream outputStream = new FileOutputStream(outputFile)
        ) {
            byte[] inputBytes = inputStream.readAllBytes();
            byte[] outputBytes = cipher.doFinal(inputBytes);
            outputStream.write(outputBytes);
        }
    }
}
```

---
🏁 *Este es el temario teórico y práctico definitivo de la Unidad 1, maquetado de forma dinámica y adaptado a las últimas tecnologías. No posee ninguna referencia a números de página físicos, incluye esquemas de diseño y tablas comparativas claras, y añade bloques de código Java listos para ser copiados y ejecutados de forma interactiva en inglés con comentarios de soporte en español.*
