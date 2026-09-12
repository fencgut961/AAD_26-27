# RA1. Desarrolla aplicaciones que gestionan información almacenada en ficheros identificando el campo de aplicación de los mismos y utilizando clases específicas.

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

🗂️ Ejemplo de Índice B-Tree por ID: El B-Tree organiza los valores del índice en forma de árbol
                                     para localizar rápidamente los datos sin necesidad de recorrer toda la tabla.
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
```
[Simulador de árboles B](https://meskeia.com/simulador-arboles-b/)

3.  **Era de la Interconectividad y el Big Data**: Con la expansión de Internet y la comunicación entre sistemas heterogéneos, los ficheros volvieron a cobrar protagonismo como formato estándar de intercambio. Surgieron formatos universales legibles por humanos (CSV, XML, JSON, YAML). Además, la explosión del Big Data implicó trabajar con volúmenes masivos de datos en sistemas de archivos distribuidos (como Hadoop Distributed File System [HDFS] en Hadoop) o almacenes de objetos en la nube (como Amazon S3, Google Cloud Storage o Azure Blob Storage) accesibles mediante APIs.

```text
🌐 Ejemplo de Formatos Universales de Intercambio: Misma información, diferentes representaciones

┌────────────────────────────────────┐      ┌────────────────────────────────────┐
│ JSON                               │      │ YAML                               │
│ Web & APIs REST                    │      │ Configuraciones                    │
│                                    │      │                                    │
│ {                                  │      │ employees:                         │
│   "employees": [                   │      │                                    │
│                                    │      │   - id: 1                          │
│     {                              │      │     name: Clara Oswald             │
│       "id": 1,                     │      │     role: SHOP_MANAGER             │
│       "name": "Clara Oswald",      │      │     salary: 2550.00                │
│       "role": "SHOP_MANAGER",      │      │                                    │
│       "salary": 2550.00            │      │   - id: 2                          │
│     },                             │      │     name: Pedro Almodovar          │
│                                    │      │     role: CLERK                    │
│     {                              │      │     salary: 1350.00                │
│       "id": 2,                     │      │                                    │
│       "name": "Pedro Almodovar",   │      │   ...                              │
│       "role": "CLERK",             │      │                                    │
│       "salary": 1350.00            │      │                                    │
│     }                              │      │                                    │
│                                    │      │                                    │
│   ]                                │      │                                    │
│ }                                  │      │                                    │
└────────────────────────────────────┘      └────────────────────────────────────┘
┌────────────────────────────────────┐      ┌────────────────────────────────────┐
│ CSV                                │      │ XML                                │
│ Tabular / Plano                    │      │ Sistemas Legacy                    │
│                                    │      │                                    │
│ id,name,role,salary                │      │ <employees>                        │
│ 1,Clara Oswald,SHOP_MANAGER,2550   │      │                                    │
│ 2,Pedro Almodovar,CLERK,1350       │      │   <employee>                       │
│ 3,Jara Li,SHOP_MANAGER,2550        │      │     <id>1</id>                     │
│ 4,Bruce Wayne,CEO,5200             │      │     <name>Clara Oswald</name>      │
│                                    │      │     <role>SHOP_MANAGER</role>      │
│ ...                                │      │     <salary>2550.00</salary>       │
│                                    │      │   </employee>                      │
│                                    │      │                                    │
│                                    │      │   ...                              │
│                                    │      │ </employees>                       │
└────────────────────────────────────┘      └────────────────────────────────────┘
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

*   **Procesamiento Masivo**: Soporte esencial para Big Data, Machine Learning y procesos de **Extracción**, **Transformación** y **Carga** [ETL](./ETL.png).
*   **Integración con la Nube**: Subir, descargar, versionar e interactuar con ficheros remotos de forma automatizada.

---


### 1.4. Almacenamiento en la Nube: Buckets y Object Storage

Cuando trabajamos con archivos en un ordenador, normalmente utilizamos un **sistema de archivos**:

```text
💻 DISCO LOCAL

📁 documentos
   ├── 📄 informe.pdf
   └── 📄 datos.json

📁 fotos
   ├── 🖼️ foto1.jpg
   └── 🖼️ foto2.jpg
````

En la nube podemos utilizar **Object Storage**, donde los archivos se almacenan como **objetos dentro de un bucket**:

```text
☁️ OBJECT STORAGE

🪣 mi-bucket
   ├── documentos/informe.pdf
   ├── documentos/datos.json
   ├── fotos/foto1.jpg
   └── fotos/foto2.jpg
```

#### 🪣 ¿Qué es un Bucket?

Un **bucket** es simplemente un **contenedor para almacenar objetos**.

Un objeto está formado principalmente por:

```text
📦 OBJETO

┌───────────────────────────────────┐
│ 🔑 Key: fotos/foto1.jpg           │
│ 📄 Datos: contenido del archivo   │
│ ℹ️ Metadatos                      │
└───────────────────────────────────┘
```

La **Key** es el nombre que identifica al objeto.

Por ejemplo:

```text
fotos/foto1.jpg
```

Aunque parece que `fotos` es una carpeta, **en Object Storage normalmente forma parte del nombre (Key) del objeto**.

#### 🔑 ¿Qué diferencia hay con un archivo local?

| Sistema de archivos                        | Object Storage                              |
| ------------------------------------------ | ------------------------------------------- |
| 📁 Carpetas y archivos                     | 🪣 Buckets y objetos                        |
| El sistema operativo gestiona los archivos | Se accede mediante APIs                     |
| Se puede modificar una parte del archivo   | Normalmente se reemplaza el objeto completo |
| Acceso mediante el sistema de archivos     | Acceso mediante HTTP/HTTPS                  |
| Ej.: NTFS, ext4                            | Ej.: Amazon S3                              |

#### 🚀 Ejemplo con Java

Una aplicación **Spring Boot** puede utilizar un SDK para comunicarse con un servicio de Object Storage:

```text
┌──────────────────┐
│   Spring Boot    │
│    Aplicación    │
└────────┬─────────┘
         │
         │ AWS SDK
         ▼
┌──────────────────┐
│    Amazon S3     │
│                  │
│ 🪣 mi-bucket     │
│  ├─ foto.jpg     │
│  ├─ datos.json   │
│  └─ informe.pdf  │
└──────────────────┘
```

La aplicación puede **subir, descargar, consultar o eliminar objetos** mediante la API de S3.

> 💡 **Idea clave:** Un **bucket** es un contenedor en la nube y un **objeto** es el archivo almacenado dentro de él. La aplicación accede a estos objetos mediante una **API**, no directamente mediante el sistema de archivos del ordenador.

---

## 2. Tipos de Ficheros según su Contenido

Aunque a nivel de hardware todos los ficheros son secuencias binarias de bytes, la forma en que el software los interpreta nos permite dividirlos en dos grandes grupos:

### 2.1. Ficheros de Texto
Están compuestos por bytes que representan **caracteres codificados bajo un estándar específico** (normalmente UTF-8).

```text
  💡 ANALOGÍA
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
public void run(String... args) {
    Path path = Paths.get("students.csv");

    try {
        String csvData = """
                ID,Name,Role
                1,Sophia,Developer
                2,Marcus,Project Manager
                """;

        Files.writeString(path, csvData, StandardCharsets.UTF_8);
        log.info("Fichero escrito correctamente: {}", path);

        String retrievedContent = Files.readString(path, StandardCharsets.UTF_8);

        log.info("--- Contenido recuperado ---");
        log.info("\n{}", retrievedContent);

    } catch (IOException e) {
        log.error("Error al procesar el fichero", e);
    }
}
```

---

### 2.2. Ficheros Binarios
Almacenan información en formato de **bytes raw**, codificados siguiendo una especificación técnica de bajo nivel.

```text
  💡 ANALOGÍA
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
public void run(String... args) {
    File file = new File("image.jpg");

    if (!file.exists()) {
        log.warn("No se encuentra el fichero '{}'", file.getName());
    } else {
        try (BufferedInputStream input =
                     new BufferedInputStream(new FileInputStream(file))) {

            MessageDigest digest = MessageDigest.getInstance("SHA-256");

            byte[] buffer = new byte[4096];
            int bytesRead;

            while ((bytesRead = input.read(buffer)) != -1) {
                digest.update(buffer, 0, bytesRead);
            }

            String hash = HexFormat.of().formatHex(digest.digest());

            log.info("📄 Fichero: {}", file.getName());
            log.info("📦 Tamaño: {} KB", file.length() / 1024);
            log.info("🔐 SHA-256: {}", hash);

        } catch (IOException | NoSuchAlgorithmException e) {
            log.error("Error al procesar el fichero", e);
        }
    }
}
```

---

### 2.3. Formatos híbridos modernos y Base64

En muchas aplicaciones actuales encontramos formatos que combinan **texto y datos binarios**.

Esto permite almacenar o transportar información estructurada junto con imágenes, documentos, sonidos u otros recursos.

#### 📦 DOCX, XLSX y PPTX: un archivo que contiene muchos archivos

Los formatos modernos de Microsoft Office (`.docx`, `.xlsx` y `.pptx`) son en realidad **contenedores ZIP**.

En su interior encontramos diferentes tipos de información:

* **XML** → estructura, contenido, estilos y metadatos.
* **Imágenes y otros recursos** → ficheros binarios.
* **Ficheros auxiliares** → configuración y relaciones entre los elementos.

Por ejemplo, un documento Word puede contener:

```text
📄 Documento.docx
       │
       ▼
   📦 Contenedor ZIP
       │
       ├── 📄 XML → contenido del documento
       ├── 📄 XML → estilos
       ├── 📄 XML → configuración
       │
       └── 📁 media/
             ├── 🖼️ image1.png
             └── 🖼️ image2.jpg
```

> 💡 **Idea clave:** algunos formatos que aparentemente son un único fichero son, internamente, contenedores que reúnen diferentes tipos de datos.

---

#### 🔤 Base64: convertir datos binarios en texto

En ocasiones necesitamos enviar un fichero binario a través de un sistema que trabaja principalmente con **texto**.

Por ejemplo, una API REST puede utilizar JSON:

```json
{
    "filename": "avatar.png",
    "mimeType": "image/png",
    "data": "..."
}
```

Pero JSON trabaja con texto, mientras que una imagen está formada por **bytes**.

Aquí podemos utilizar **Base64**.

Base64 es una técnica que **codifica datos binarios como una cadena de caracteres**:

```text
🖼️ imagen.png
      │
      │ bytes
      ▼
┌────────────────┐
│     Base64     │
│  codificación  │
└───────┬────────┘
        │
        ▼
"iVBORw0KGgoAAAANSUhEUg..."
        │
        ▼
      JSON
```

El receptor puede realizar el proceso inverso:

```text
        JSON
          │
          ▼
   Cadena Base64
          │
          │ decodificar
          ▼
    Bytes originales
          │
          ▼
      🖼️ imagen.png
```

#### ⚠️ Base64 no comprime ni cifra

Es importante distinguir estos conceptos:

| Técnica        | ¿Qué hace?                           |
| -------------- | ------------------------------------ |
| **Base64**     | Convierte datos binarios en texto    |
| **Compresión** | Reduce el tamaño de los datos        |
| **Cifrado**    | Protege los datos mediante una clave |

Base64 **no proporciona seguridad** y tampoco reduce el tamaño del archivo. De hecho, el resultado ocupa aproximadamente un **33 % más** que los datos binarios originales.

---

#### ☕ Ejemplo práctico en Java

Java proporciona la clase `Base64` para realizar la codificación y decodificación.

En este ejemplo:

1. Leemos una imagen como bytes.
2. La convertimos a Base64.
3. Simulamos que esa cadena se envía como parte de un JSON.
4. Decodificamos la cadena.
5. Recuperamos el archivo original.

```java
@Override
public void run(String... args) {

    Path imagePath = Path.of("image.jpg");
    Path restoredPath = Path.of("restored_image.jpg");

    if (Files.exists(imagePath)) {

        try {
            // Leer el fichero binario
            byte[] binaryData = Files.readAllBytes(imagePath);

            log.info("Tamaño original: {} bytes", binaryData.length);

            // Codificar los bytes en Base64
            String base64 = Base64.getEncoder().encodeToString(binaryData);

            log.info("Tamaño en Base64: {} caracteres", base64.length());
            log.info("Base64: {}...", base64.substring(0, Math.min(50, base64.length())));

            // Decodificar Base64 para recuperar los bytes originales
            byte[] restoredData = Base64.getDecoder().decode(base64);

            // Restaurar el fichero
            Files.write(restoredPath, restoredData);

            log.info("Fichero restaurado correctamente: {}", restoredPath);

        } catch (IOException e) {
            log.error("Error al procesar el fichero", e);
        }

    } else {
        log.warn("No se encuentra el fichero: {}", imagePath);
    }
}
```

El proceso completo puede resumirse así:

```text
┌──────────────────┐
│   🖼️ image.jpg   │
│      bytes       │
└────────┬─────────┘
         │
         │ Base64.encode()
         ▼
┌──────────────────────────┐
│ "iVBORw0KGgoAAAANS..."   │
│          🔤              │
└────────┬─────────────────┘
         │
         │ JSON / API REST
         ▼
┌──────────────────────────┐
│      📡 TRANSMISIÓN      │
└────────┬─────────────────┘
         │
         │ Base64.decode()
         ▼
┌─────────────────────────┐
│ restored_image.jpg      │
│        🖼️              │
└─────────────────────────┘
```

> 💡 **Idea clave:** Base64 no convierte realmente una imagen en texto. **Codifica sus bytes en una representación textual** que puede transportarse fácilmente dentro de estructuras como JSON.

> 🚀 **En aplicaciones reales:** esta técnica puede utilizarse para transportar pequeños archivos o imágenes dentro de una petición JSON. Para archivos grandes, normalmente resulta más eficiente utilizar una subida binaria (`multipart/form-data`) o almacenamiento de objetos.

---

### 2.4. Codificaciones de texto: de bytes a caracteres

Cuando guardamos texto en un fichero, el ordenador no almacena directamente letras como `á`, `ñ` o `日`.

El texto debe convertirse en **bytes** mediante una codificación.

```text
        ✍️ TEXTO
           │
           │ codificar
           ▼
      🔢 BYTES
           │
           │ guardar
           ▼
       💾 FICHERO
```

Cuando volvemos a leerlo, hacemos el proceso contrario:

```text
       💾 FICHERO
           │
           │ bytes
           ▼
        🔢 BYTES
           │
           │ decodificar
           ▼
        ✍️ TEXTO
```

La codificación indica **cómo deben interpretarse esos bytes para obtener los caracteres correctos**.

#### 🌍 UTF-8: una codificación universal

Actualmente, **UTF-8 es la codificación más utilizada para trabajar con texto**.

Permite representar caracteres de diferentes idiomas:

```text
Español    →  Canción, España, Árbol
Inglés     →  Software, Computer
Chino      →  数据库
Japonés    →  こんにちは
Emoji      →  🚀 🔒 💻
```

UTF-8 utiliza entre **1 y 4 bytes por carácter**, dependiendo del carácter.

Además, es compatible con ASCII: los caracteres básicos del inglés utilizan exactamente los mismos valores que en ASCII.

#### 💥 ¿Qué ocurre si utilizamos una codificación incorrecta?

El problema aparece cuando un fichero se escribe utilizando una codificación y se lee utilizando otra.

Por ejemplo, el texto:

```text
Canción
```

se puede almacenar en UTF-8 utilizando estos bytes:

```text
43 61 6E 63 69 C3 B3 6E
```

Si esos bytes se leen correctamente como UTF-8:

```text
43 61 6E 63 69 C3 B3 6E
 ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓
 C  a  n  c  i  ó     n

Resultado → Canción
```

Pero si los mismos bytes se interpretan utilizando una codificación incorrecta, pueden aparecer caracteres extraños:

```text
Canción
   ↓
CanciÃ³n
```

Este fenómeno se conoce habitualmente como **Mojibake**.

```text
┌───────────────────────┐
│ 💾 Fichero            │
│                       │
│ Bytes almacenados     │
│       C3 B3           │
└──────────┬────────────┘
           │
           │ UTF-8
           ▼
       ✅ "ó"

           │
           │ Codificación incorrecta
           ▼
       ❌ "Ã³"
```

> 💡 **Idea clave:** los bytes no contienen por sí mismos una letra. Necesitamos conocer la **codificación utilizada** para convertir esos bytes correctamente en caracteres.

#### ☕ Java: indicar explícitamente la codificación

Cuando trabajamos con ficheros de texto, es recomendable indicar explícitamente la codificación que queremos utilizar.

Por ejemplo, utilizando `UTF-8`:

```java
@Override
public void run(String... args) {

    Path path = Path.of("mensaje.txt");

    try {
        Files.writeString(
                path,
                "¡Hola! Canción, España, 数据库, こんにちは 🚀",
                StandardCharsets.UTF_8
        );

        String content = Files.readString(
                path,
                StandardCharsets.UTF_8
        );

        log.info("Contenido: {}", content);

    } catch (IOException e) {
        log.error("Error al trabajar con el fichero", e);
    }
}
```

En este caso, tanto la escritura como la lectura utilizan **UTF-8**:

```text
              UTF-8
Texto ──────────────────► Bytes
  ▲                         │
  │                         │
  └─────────────────────────┘
              UTF-8
```

#### ⚠️ No confundir codificación con idioma

UTF-8 **no traduce** un texto de un idioma a otro.

Por ejemplo:

```text
"Hola" ──┐
         │
"Hello" ─┼──► UTF-8 ──► Bytes
         │
"こんにちは" ─┘
```

UTF-8 simplemente define **cómo representar esos caracteres mediante bytes**.

La traducción entre idiomas es otra cuestión completamente diferente.

> 🚀 **En aplicaciones reales:** problemas de codificación aparecen frecuentemente al importar CSV, leer archivos generados por otros sistemas, consumir APIs, procesar datos de diferentes países o intercambiar información entre aplicaciones.


---

## 3. Acceso clásico (`java.io`) vs. acceso moderno (`java.nio`)

Cuando una aplicación necesita trabajar con un archivo, realmente hay **dos problemas diferentes**:

1. **Identificar dónde está el archivo**.
2. **Realizar una operación sobre él**: leerlo, escribirlo, copiarlo, moverlo, eliminarlo, etc.

Java dispone de dos APIs principales para resolver estos problemas.

```text
             SISTEMA DE ARCHIVOS
                     │
                     ▼
              ¿Dónde está?
                     │
              ┌──────┴──────┐
              │             │
            File           Path
              │             │
          java.io      java.nio.file
                            │
                            ▼
                    ¿Qué queremos hacer?
                            │
                          Files
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
           Leer          Escribir        Copiar
```

### Las dos APIs

| API             | Elemento principal | Papel                                                          |
| --------------- | ------------------ | -------------------------------------------------------------- |
| `java.io`       | `File`             | Representar una ruta y realizar operaciones básicas            |
| `java.nio.file` | `Path` + `Files`   | Representar rutas y realizar operaciones de forma más completa |

> **Idea fundamental:** `File` y `Path` representan **rutas**. No contienen el contenido del archivo.

---

### 3.1. La API clásica: `java.io.File`

`File` es la forma tradicional de representar un archivo o directorio en Java.

Por ejemplo:

```java
File file = new File("datos/alumnos.csv");
```

Aquí Java no está leyendo `alumnos.csv`.

Simplemente estamos creando un objeto que representa:

```text
datos/
   │
   └── alumnos.csv
             ▲
             │
          File
```

A partir de ese objeto podemos preguntar por las características del recurso:

```text
File
 │
 ├── ¿Existe?              → exists()
 ├── ¿Es un archivo?       → isFile()
 ├── ¿Es un directorio?    → isDirectory()
 ├── ¿Qué nombre tiene?    → getName()
 ├── ¿Qué tamaño tiene?    → length()
 └── ¿Dónde está?          → getAbsolutePath()
```

#### ¿Para qué resulta útil `File`?

Principalmente para **consultar y gestionar archivos y directorios**.

Por ejemplo, una aplicación podría comprobar si existe un archivo de configuración antes de intentar utilizarlo:

```java
File config = new File("config.properties");

if (config.exists() && config.isFile()) {
    log.info("Archivo de configuración encontrado: {}", config.getAbsolutePath());
} else {
    log.warn("No se encuentra el archivo de configuración.");
}
```

Esto muestra una idea importante:

> **Crear un objeto `File` no significa abrir ni leer el archivo.**

Para leer o escribir su contenido, `java.io` utiliza otras clases como `FileInputStream`, `FileOutputStream`, `FileReader` o `FileWriter`.

---

#### 📋 Métodos básicos de `File`

| Método              | Retorno    | ¿Qué permite hacer?                       |
| ------------------- | ---------- | ----------------------------------------- |
| `exists()`          | `boolean`  | Comprobar si existe                       |
| `isFile()`          | `boolean`  | Comprobar si es un archivo                |
| `isDirectory()`     | `boolean`  | Comprobar si es un directorio             |
| `getName()`         | `String`   | Obtener el nombre                         |
| `getAbsolutePath()` | `String`   | Obtener la ruta absoluta                  |
| `length()`          | `long`     | Obtener el tamaño en bytes                |
| `lastModified()`    | `long`     | Obtener la fecha de modificación          |
| `delete()`          | `boolean`  | Eliminar un recurso                       |
| `mkdir()`           | `boolean`  | Crear un directorio                       |
| `mkdirs()`          | `boolean`  | Crear una estructura de directorios       |
| `list()`            | `String[]` | Obtener los nombres de un directorio      |
| `listFiles()`       | `File[]`   | Obtener sus elementos como objetos `File` |

#### `mkdir()` frente a `mkdirs()`

La diferencia es sencilla:

```text
mkdir()
   │
   └── crea únicamente el directorio indicado

mkdirs()
   │
   └── crea también los directorios intermedios
```

Por ejemplo:

```text
datos/2026/logs/
```

Si `datos` y `2026` no existen:

* `mkdir()` → no puede crear toda la estructura.
* `mkdirs()` → crea los directorios necesarios.

---

### 3.2. ¿Por qué aparece `java.nio.file`?

`java.io.File` funciona y sigue formando parte de Java. Sin embargo, con el tiempo se necesitó una API más completa para trabajar con el sistema de archivos.

Por eso Java incorporó **NIO.2**, cuyo paquete principal para este trabajo es:

```text
java.nio.file
```

Sus dos protagonistas son:

```text
             java.nio.file
                  │
          ┌───────┴───────┐
          ▼               ▼
        Path             Files
          │               │
       ¿Dónde?         ¿Qué hacer?
          │               │
          │         ┌─────┼─────┐
          │         ▼     ▼     ▼
          │       Leer Escribir Copiar
          │
          └── representa la ruta
```

Esta separación hace que el modelo sea mucho más claro:

#### `Path` → la ruta

Representa la ubicación de un archivo o directorio.

```java
Path path = Path.of("datos", "alumnos.csv");
```

#### `Files` → la operación

Proporciona métodos para trabajar con esa ruta:

```text
Files
 │
 ├── exists()
 ├── createFile()
 ├── createDirectories()
 ├── readString()
 ├── writeString()
 ├── copy()
 ├── move()
 ├── delete()
 ├── list()
 └── walk()
```

Por tanto:

```text
Path
 │
 │  "datos/alumnos.csv"
 │
 ▼
Files
 │
 ├── leer
 ├── escribir
 ├── copiar
 ├── mover
 └── eliminar
```

---

### 3.3. `Path`: trabajar con rutas de forma cómoda

Una ventaja importante de `Path` es que permite **construir y manipular rutas** sin tener que escribir manualmente los separadores del sistema operativo.

En lugar de:

```java
Path path = Path.of("datos/alumnos.csv");
```

también podemos construirla por partes:

```java
Path path = Path.of("datos", "alumnos.csv");
```

Java se encarga de utilizar el separador adecuado en cada sistema.

Además, `Path` permite trabajar con las diferentes partes de una ruta:

```text
datos/2026/alumnos.csv
   │      │       │
   │      │       └── getFileName()
   │      └────────── getParent()
   └───────────────── estructura de la ruta
```

Algunos métodos importantes son:

| Método             | Función                                     |
| ------------------ | ------------------------------------------- |
| `getFileName()`    | Obtiene el nombre final                     |
| `getParent()`      | Obtiene el directorio padre                 |
| `toAbsolutePath()` | Convierte la ruta en absoluta               |
| `resolve()`        | Añade una ruta a otra                       |
| `normalize()`      | Simplifica elementos redundantes de la ruta |

---

### 3.4. `Files`: realizar operaciones sobre archivos

Una vez tenemos un `Path`, la clase `Files` proporciona los métodos necesarios para trabajar con él.

Por ejemplo:

```text
Path config = Path.of("config", "app.properties");

             │
             ▼
        Files.exists()
             │
             ▼
       ¿Existe el archivo?
```

Y podemos realizar operaciones como:

| Operación                  | Método                      |
| -------------------------- | --------------------------- |
| Comprobar existencia       | `Files.exists()`            |
| Comprobar si es archivo    | `Files.isRegularFile()`     |
| Comprobar si es directorio | `Files.isDirectory()`       |
| Crear archivo              | `Files.createFile()`        |
| Crear directorios          | `Files.createDirectories()` |
| Leer texto                 | `Files.readString()`        |
| Escribir texto             | `Files.writeString()`       |
| Copiar                     | `Files.copy()`              |
| Mover / renombrar          | `Files.move()`              |
| Eliminar                   | `Files.delete()`            |
| Listar directorio          | `Files.list()`              |
| Recorrer directorios       | `Files.walk()`              |

Además, muchas operaciones de `Files` utilizan excepciones como `IOException` para comunicar los errores, lo que permite conocer mejor qué ha sucedido.

---

### 3.5. Un ejemplo real: crear un directorio de logs

Supongamos que una aplicación necesita guardar sus registros en:

```text
logs/
   └── application.log
```

Con NIO.2 podemos expresar la operación de forma muy directa:

```java
@Override
public void run(String... args) {

    Path logFile = Path.of("logs", "application.log");

    try {

        Files.createDirectories(logFile.getParent());

        Files.writeString(
                logFile,
                "Aplicación iniciada correctamente.\n"
        );

        log.info("Archivo de log: {}", logFile.toAbsolutePath());
        log.info("Tamaño: {} bytes", Files.size(logFile));

    } catch (IOException e) {
        log.error("Error al trabajar con el archivo de log", e);
    }
}
```

Aquí se ve claramente la responsabilidad de cada elemento:

```text
Path
 │
 └── "logs/application.log"
             │
             │ indica dónde
             ▼
           Files
             │
             ├── createDirectories()
             │
             ├── writeString()
             │
             └── size()
```

---

### 3.6. Una ventaja importante: copiar, mover y eliminar

Una de las mejoras más evidentes de NIO.2 es que operaciones habituales se expresan directamente mediante métodos de `Files`.

Por ejemplo:

```text
                 Path origen
                     │
                     ▼
               Files.copy()
                     │
                     ▼
                Path destino
```

No necesitamos implementar manualmente un proceso de lectura y escritura para realizar una copia.

Lo mismo ocurre con:

* `Files.copy()` → copiar.
* `Files.move()` → mover o renombrar.
* `Files.delete()` → eliminar.

Esto hace que el código sea más **claro, corto y fácil de mantener**.

---

### 3.7. Recorrer directorios con `Files.walk()`

Otra funcionalidad especialmente interesante de NIO.2 es `Files.walk()`.

Permite recorrer un directorio y sus subdirectorios:

```text
proyecto/
├── src/
│   ├── Main.java
│   └── Utils.java
├── logs/
│   └── application.log
└── datos/
    └── alumnos.csv
```

Podemos recorrer todo el árbol y quedarnos, por ejemplo, únicamente con los archivos `.log`:

```text
Files.walk()
      │
      ▼
  todos los recursos
      │
      ▼
isRegularFile()
      │
      ▼
  solo archivos
      │
      ▼
  extensión .log
      │
      ▼
 archivos encontrados
```

Este tipo de operación resulta muy útil en aplicaciones reales para buscar documentos, localizar archivos de configuración o analizar archivos de registro.

```java
@Override
public void run(String... args) {

    Path root = Path.of(".");

    try (Stream<Path> paths = Files.walk(root)) {

        paths.filter(Files::isRegularFile)
                .filter(path -> path.toString().endsWith(".log"))
                .forEach(path ->
                        log.info(
                                "{} | {} bytes",
                                path.toAbsolutePath(),
                                Files.size(path)
                        )
                );

    } catch (IOException e) {
        log.error("Error al recorrer el directorio", e);
    }
}
```

> `Files.walk()` devuelve un `Stream<Path>`, por lo que podemos utilizar las operaciones de Streams para filtrar y procesar los archivos encontrados.

---

### 3.8. ¿Cuál debemos utilizar?

La pregunta no es realmente **"¿`File` o `Path`?"**, sino entender la evolución de la API.

```text
java.io
   │
   └── File
       └── API clásica
            └── todavía válida


java.nio.file
   │
   ├── Path
   │    └── representa la ruta
   │
   └── Files
        └── realiza las operaciones
             ├── leer
             ├── escribir
             ├── copiar
             ├── mover
             ├── eliminar
             └── recorrer
```

#### 🧠 Qué debemos recordar

> **`File` y `Path` representan la ubicación de un recurso.**

> **`Files` proporciona las operaciones para trabajar con ese recurso.**

Por tanto, en aplicaciones nuevas utilizaremos normalmente:

```text
Path + Files
```

mientras que `File` es fundamental para **entender código Java existente y la API clásica de `java.io`**.

La siguiente idea será especialmente importante:

**`Path` nos dice dónde está el recurso; `Files` nos permite trabajar con él.**


---

## 4. Formas de acceso a ficheros

Cuando una aplicación trabaja con un archivo, existen diferentes formas de localizar y procesar sus datos.

Las dos estrategias fundamentales son:

* **Acceso secuencial** → los datos se procesan siguiendo un orden.
* **Acceso aleatorio o directo** → podemos desplazarnos directamente a una posición determinada.

La elección depende de **cómo están organizados los datos y de cómo necesita utilizarlos la aplicación**.

```text
                    FICHERO
                       │
              ¿Cómo accedemos?
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
       SECUENCIAL            ALEATORIO
             │                   │
        uno tras otro       posición concreta
             │                   │
        CSV · logs          registros fijos
        texto · JSON        archivos binarios
```

---

### 4.1. Acceso secuencial

En el acceso secuencial, los datos se procesan **en orden**, normalmente desde el principio hasta el final del archivo.

```text
Fichero:

┌─────┬─────┬─────┬─────┬─────┐
│  1  │  2  │  3  │  4  │  5  │
└─────┴─────┴─────┴─────┴─────┘
   ↓     ↓     ↓     ↓     ↓
   1  →  2  →  3  →  4  →  5
```

Si queremos procesar el dato número 4, normalmente debemos haber recorrido antes los datos anteriores.

#### Ejemplo cotidiano

Es parecido a leer un libro desde el principio:

```text
Página 1 → Página 2 → Página 3 → Página 4
```

Si queremos llegar a la página 4, seguimos el orden de las páginas.

#### ¿Cuándo resulta adecuado?

Es especialmente útil cuando necesitamos **procesar muchos o todos los datos del archivo**:

* Archivos CSV.
* Archivos de texto.
* Logs de aplicaciones.
* Ficheros JSON o XML.
* Procesamiento de datos por lotes.

Por ejemplo, para analizar un archivo de logs y contar cuántos errores contiene, tiene sentido recorrer sus líneas una detrás de otra.

#### Ventajas

* Es sencillo de implementar.
* Resulta muy adecuado para procesar grandes cantidades de datos de principio a fin.
* Permite procesar los datos progresivamente sin necesidad de cargar todo el archivo en memoria.

#### Inconveniente

Si necesitamos localizar repetidamente un dato situado en una posición concreta de un archivo grande, recorrer todos los datos anteriores puede resultar poco eficiente.

---

#### 🚀 Ejemplo: procesar un archivo de logs línea a línea

Supongamos que una aplicación genera un archivo:

```text
application.log
```

y queremos localizar las líneas que contienen errores.

Con `Files.lines()` podemos procesar el archivo **línea a línea**:

```java
@Override
public void run(String... args) {

    Path path = Path.of("application.log");

    try (Stream<String> lines = Files.lines(path)) {

        lines.filter(line -> line.contains("ERROR"))
                .forEach(line -> log.info("Error encontrado: {}", line));

    } catch (IOException e) {
        log.error("Error al leer el archivo de logs", e);
    }
}
```

El flujo puede representarse así:

```text
application.log
      │
      ▼
   línea 1 ──► ¿ERROR?
      │
      ▼
   línea 2 ──► ¿ERROR?
      │
      ▼
   línea 3 ──► ¿ERROR?
      │
      ▼
     ...
```

> **Importante:** `Files.lines()` devuelve un `Stream<String>` que permite procesar las líneas progresivamente. No es necesario cargar todo el contenido del archivo en un `String`.

---

### 4.2. Acceso aleatorio o directo

El acceso aleatorio permite **desplazarnos directamente a una posición concreta del archivo**, sin tener que procesar previamente todos los datos que se encuentran antes.

```text
Fichero:

┌──────────┬──────────┬──────────┬──────────┐
│ Registro │ Registro │ Registro │ Registro │
│    0     │    1     │    2     │    3     │
└──────────┴──────────┴──────────┴──────────┘
                         ▲
                         │
                    acceder aquí
```

En Java, una de las clases clásicas para realizar este tipo de acceso es:

```text
RandomAccessFile
```

Su principal característica es que permite **mover el puntero de lectura/escritura** mediante:

```java
seek(posicion)
```

Por ejemplo:

```text
seek(0)   → principio del archivo
seek(16)  → posición 16
seek(32)  → posición 32
seek(48)  → posición 48
```

---

#### ¿Cuándo es especialmente útil?

El acceso aleatorio resulta interesante cuando trabajamos con archivos cuyos registros tienen una **estructura conocida**, especialmente cuando cada registro ocupa un tamaño fijo.

Por ejemplo:

```text
employees.dat

┌────────────────┬────────────────┬────────────────┐
│ Registro 0     │ Registro 1     │ Registro 2     │
│ 16 bytes       │ 16 bytes       │ 16 bytes       │
└────────────────┴────────────────┴────────────────┘
       0               16               32
```

Si sabemos que cada registro ocupa **16 bytes**, podemos calcular dónde comienza cualquier registro:

```text
posición = número_de_registro × tamaño_del_registro
```

Por ejemplo:

```text
Registro 0 → 0 × 16 = 0
Registro 1 → 1 × 16 = 16
Registro 2 → 2 × 16 = 32
Registro 3 → 3 × 16 = 48
```

Así podemos acceder directamente al registro que necesitamos.

---

#### 🧮 Ejemplo: registros de tamaño fijo

Supongamos que cada empleado se almacena mediante:

| Campo     |       Tamaño |
| --------- | -----------: |
| `ID`      |      4 bytes |
| `Edad`    |      4 bytes |
| `Salario` |      8 bytes |
| **Total** | **16 bytes** |

El archivo tendría esta estructura:

```text
0                16               32               48
│                 │                │                │
▼                 ▼                ▼                ▼
┌────────────────┬────────────────┬────────────────┐
│   Registro 0   │   Registro 1   │   Registro 2   │
│    16 bytes    │    16 bytes    │    16 bytes    │
└────────────────┴────────────────┴────────────────┘
```

Para acceder al **Registro 2**:

```text
2 × 16 = 32 bytes
```

Por tanto:

```java
raf.seek(32);
```

El puntero se sitúa directamente al comienzo del registro.

---

#### ¿Y si queremos únicamente el salario?

Dentro de cada registro:

```text
Registro
┌──────────┬──────────┬────────────────┐
│ ID       │ Edad     │ Salario        │
│ 4 bytes  │ 4 bytes  │ 8 bytes        │
└──────────┴──────────┴────────────────┘
0          4          8                16
```

El salario comienza en el byte **8** del registro.

Para obtener el salario del Registro 2:

```text
Inicio del registro → 2 × 16 = 32
Desplazamiento       → 8
--------------------------------
Posición final       → 40
```

Por tanto:

```java
raf.seek(40);
```

La idea general es:

```text
posición del campo =
    (número de registro × tamaño del registro)
    + desplazamiento del campo
```

> Este mecanismo es especialmente útil cuando conocemos de antemano la estructura y el tamaño de los registros.

---

#### 🚀 Ejemplo práctico con `RandomAccessFile`

Vamos a utilizar un archivo binario de empleados y modificar directamente el salario de un registro concreto.

```java
@Override
public void run(String... args) {

    Path path = Path.of("employees.dat");

    final int RECORD_SIZE = 16;

    try (RandomAccessFile raf = new RandomAccessFile(path.toFile(), "rw")) {

        raf.setLength(0);

        // Registro 0
        raf.writeInt(1);
        raf.writeInt(25);
        raf.writeDouble(1200.00);

        // Registro 1
        raf.writeInt(2);
        raf.writeInt(30);
        raf.writeDouble(1500.00);

        // Registro 2
        raf.writeInt(3);
        raf.writeInt(40);
        raf.writeDouble(1800.00);

        // Acceder directamente al Registro 2
        long recordPosition = 2L * RECORD_SIZE;

        raf.seek(recordPosition);

        int id = raf.readInt();
        int age = raf.readInt();
        double salary = raf.readDouble();

        log.info(
                "Empleado encontrado: ID={}, Edad={}, Salario={} €",
                id, age, salary
        );

        // El salario comienza 8 bytes después del inicio del registro
        long salaryPosition = recordPosition + 8;

        raf.seek(salaryPosition);
        raf.writeDouble(2500.00);

        log.info("Salario actualizado directamente en el archivo.");

    } catch (IOException e) {
        log.error("Error al acceder al archivo", e);
    }
}
```

Aquí se ve claramente la ventaja del acceso directo:

```text
employees.dat

Registro 0 ────────────────┐
Registro 1 ────────────────┤
Registro 2 ────────────────┤
                           │
                           ▼
                    seek(32)
                           │
                           ▼
                     Registro 2
                           │
                           ▼
                    seek(40)
                           │
                           ▼
                      Salario
```

No necesitamos leer previamente los registros 0 y 1 para situarnos en el Registro 2.

---

### 4.3. Acceso secuencial vs. acceso aleatorio

| Característica              | Secuencial           | Aleatorio                               |
| --------------------------- | -------------------- | --------------------------------------- |
| Forma de acceso             | En orden             | Posición concreta                       |
| Operación habitual          | Recorrer datos       | Saltar a un registro                    |
| Ideal para                  | Logs, CSV, texto     | Registros binarios                      |
| Necesita conocer posiciones | No                   | Sí, normalmente                         |
| Complejidad                 | Baja                 | Mayor                                   |
| Ejemplo Java                | `Files.lines()`      | `RandomAccessFile`                      |
| Uso típico                  | Procesar información | Consultar/modificar registros concretos |

#### Una forma sencilla de recordarlo

```text
SECUENCIAL
──────────

1 → 2 → 3 → 4 → 5

"Voy recorriendo los datos"


ALEATORIO
─────────

1   2   3   4   5
        ↑
      seek()

"Voy directamente donde necesito"
```

---

### 4.4. ¿Qué estrategia elegir?

La decisión depende de **cómo va a utilizar la aplicación los datos**.

#### Elegiremos acceso secuencial cuando...

Necesitemos procesar la información de forma ordenada:

```text
CSV → leer todas las filas
LOG → analizar todas las entradas
JSON → procesar el documento
TXT → recorrer las líneas
```

#### Elegiremos acceso aleatorio cuando...

Necesitemos consultar o modificar registros concretos:

```text
employees.dat
      │
      ├── Buscar empleado 125
      ├── Modificar empleado 240
      └── Consultar empleado 780
```

Si los registros tienen tamaño fijo, podemos calcular directamente su posición.

---

### 4.5. En las aplicaciones reales pueden combinarse

Las dos estrategias no son excluyentes.

Una aplicación puede utilizar **acceso secuencial para generar o procesar información** y **acceso directo para consultar determinados datos**.

Por ejemplo, imaginemos una aplicación que mantiene un archivo de registros:

```text
                employees.dat
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
   Procesamiento             Consulta concreta
    secuencial                  directa
          │                       │
          ▼                       ▼
  recorrer registros         seek(posición)
```

Lo importante no es memorizar una clase concreta, sino entender **qué estrategia necesita la aplicación según la forma en que va a utilizar los datos**.

#### 🧠 Idea clave

> **Acceso secuencial:** recorremos los datos siguiendo un orden.

> **Acceso aleatorio:** nos desplazamos directamente a una posición conocida.

> **Si conocemos la estructura y el tamaño de los registros, podemos calcular su posición y acceder directamente a ellos.**


## 5. El ciclo de vida de las operaciones sobre ficheros

Cuando una aplicación trabaja con un fichero, podemos entender la operación como un ciclo:

1. **Acceder al recurso.**
2. **Leer o escribir los datos.**
3. **Posicionarse en una parte concreta**, si es necesario.
4. **Liberar el recurso** cuando ya no se necesita.

En Java moderno, la API principal para trabajar con ficheros es `java.nio.file`, especialmente mediante `Path` y `Files`.

```text
                     FICHERO
                        │
                        ▼
                  ┌──────────┐
                  │   Path   │
                  │ ubicación│
                  └────┬─────┘
                       │
                       ▼
                  ┌──────────┐
                  │  Files   │
                  │operaciones│
                  └────┬─────┘
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
        Operaciones        Operaciones
         sencillas          avanzadas
              │                 │
              ▼                 ▼
    readString(), etc.   FileChannel
```

No todas las operaciones requieren que gestionemos manualmente cada una de estas fases. Los métodos de alto nivel de `Files` pueden encargarse internamente de abrir y cerrar el fichero cuando realizamos una operación puntual.

---

### 5.1. Acceder al fichero

El primer paso consiste en identificar el fichero con el que queremos trabajar.

Para ello utilizamos `Path`:

```java
// Representa la ubicación del fichero.
// Path no lee ni modifica el contenido por sí mismo.
Path path = Path.of("application.log");
```

`Path` representa la **ruta del fichero o directorio**, pero no realiza por sí mismo operaciones de lectura o escritura.

Las operaciones las proporciona `Files`.

Por ejemplo, para comprobar si existe un fichero:

```java
// Comprobamos si existe un fichero en la ruta indicada.
if (Files.exists(path)) {

    // El fichero existe y podemos continuar trabajando con él.
    log.info("El fichero existe.");

} else {

    // La ruta no corresponde actualmente a un fichero existente.
    log.warn("El fichero no existe.");
}
```

Esta separación entre **representar la ubicación (`Path`)** y **realizar la operación (`Files`)** es una de las características fundamentales de la API moderna de ficheros de Java.

---

### 5.2. Lectura y escritura

Una vez identificado el recurso, podemos realizar las operaciones necesarias.

Para operaciones sencillas sobre archivos de texto, Java proporciona métodos de alto nivel como:

```java
// Lee todo el contenido del fichero y lo almacena en un String.
String content = Files.readString(path);
```

y:

```java
// Escribe el texto indicado en el fichero.
Files.writeString(
        path,
        "Aplicación iniciada correctamente."
);
```

Estas operaciones son apropiadas cuando queremos leer o escribir el contenido completo de un fichero y su tamaño es razonable.

Por ejemplo:

```java
@Override
public void run(String... args) {

    // Path identifica el fichero de configuración.
    Path path = Path.of("config.txt");

    try {

        // Escribimos una configuración sencilla en el fichero.
        // Si el fichero existe, su contenido se reemplaza.
        Files.writeString(
                path,
                "server.port=8080\n"
                        + "app.name=GestorPedidos"
        );

        // Leemos de nuevo todo el contenido del fichero.
        String content = Files.readString(path);

        // Mostramos la configuración obtenida.
        log.info("Configuración:\n{}", content);

    } catch (IOException e) {

        // Gestionamos cualquier error de entrada/salida.
        log.error("Error al trabajar con el fichero", e);
    }
}
```

En este caso, `Files` se encarga internamente de realizar la apertura, lectura o escritura y cierre necesarios para completar cada operación.

---

### 5.3. Procesamiento progresivo

No siempre queremos cargar todo el fichero en memoria.

Si trabajamos con un archivo grande, puede ser más apropiado procesar su contenido progresivamente.

Por ejemplo, `Files.lines()` permite obtener un `Stream<String>` con las líneas del fichero:

```java
@Override
public void run(String... args) {

    // Fichero de logs que queremos analizar.
    Path path = Path.of("application.log");

    // Files.lines() permite procesar las líneas progresivamente.
    // try-with-resources garantiza que el Stream se cierre correctamente.
    try (Stream<String> lines = Files.lines(path)) {

        // Nos quedamos únicamente con las líneas que contienen ERROR.
        lines.filter(line -> line.contains("ERROR"))

                // Procesamos cada línea encontrada.
                .forEach(line ->
                        log.info("Error encontrado: {}", line)
                );

    } catch (IOException e) {

        // Gestionamos los errores producidos durante la lectura.
        log.error("Error al procesar el fichero", e);
    }
}
```

Aquí el fichero se procesa **línea a línea**, en lugar de cargar todo su contenido de una sola vez.

Como `Files.lines()` devuelve un `Stream` asociado a un recurso abierto, debemos cerrarlo correctamente. Por eso utilizamos **try-with-resources**.

> **Regla práctica:** para ficheros pequeños y operaciones sencillas, `readString()` y `writeString()` son muy cómodos. Para ficheros grandes o procesamiento progresivo, podemos utilizar streams, lectores o escritores.

---

### 5.4. Posicionamiento y acceso avanzado

Cuando necesitamos controlar con mayor precisión dónde leer o escribir, podemos utilizar las clases de `java.nio.channels`.

Una de las más importantes es:

```text
FileChannel
```

`FileChannel` permite trabajar con archivos mediante `ByteBuffer` y realizar operaciones en posiciones concretas.

Por ejemplo:

```text
FICHERO

0          16          32          48
│           │           │           │
▼           ▼           ▼           ▼
┌───────────┬───────────┬───────────┬───────────┐
│ Registro 0│ Registro 1│ Registro 2│ Registro 3│
└───────────┴───────────┴───────────┴───────────┘
                        ▲
                        │
                  posición 32
```

Podemos leer directamente desde una posición determinada:

```java
// Lee los datos comenzando en la posición 32 del fichero.
channel.read(buffer, 32);
```

En este caso, la lectura se realiza comenzando en el byte `32`.

Esto resulta especialmente útil cuando trabajamos con **ficheros binarios estructurados**, registros de tamaño fijo o aplicaciones que necesitan un mayor control sobre las operaciones de entrada y salida.

---

### 5.5. `FileChannel` y `ByteBuffer`

Cuando utilizamos `FileChannel`, los datos se intercambian normalmente mediante un `ByteBuffer`.

El proceso puede representarse así:

```text
Fichero
   │
   │ bytes
   ▼
FileChannel
   │
   │
   ▼
ByteBuffer
   │
   ▼
Datos procesados por Java
```

Por ejemplo, si cada registro ocupa 16 bytes:

```java
@Override
public void run(String... args) {

    // Fichero binario que contiene los registros de empleados.
    Path path = Path.of("employees.dat");

    try (FileChannel channel = FileChannel.open(
            path,
            StandardOpenOption.READ)) {

        // Cada registro ocupa 16 bytes:
        // 4 bytes para el ID
        // 4 bytes para la edad
        // 8 bytes para el salario
        ByteBuffer buffer = ByteBuffer.allocate(16);

        // El registro 2 comienza en el byte 32:
        // 2 registros × 16 bytes = 32 bytes.
        int bytesRead = channel.read(buffer, 32);

        // Comprobamos que hemos obtenido el registro completo.
        if (bytesRead == 16) {

            // Preparamos el buffer para comenzar a leer
            // los datos que acabamos de introducir.
            buffer.flip();

            // Recuperamos los campos en el mismo orden
            // en el que fueron almacenados.
            int id = buffer.getInt();
            int age = buffer.getInt();
            double salary = buffer.getDouble();

            // Mostramos la información del empleado.
            log.info(
                    "Empleado: ID={}, Edad={}, Salario={} €",
                    id,
                    age,
                    salary
            );

        } else {

            // No hemos podido obtener el registro completo.
            log.warn("No se pudo leer el registro completo.");
        }

    } catch (IOException e) {

        // Gestionamos los posibles errores de entrada/salida.
        log.error("Error al leer el fichero", e);
    }
}
```

En este ejemplo:

1. `Path` identifica el fichero.
2. `FileChannel` proporciona acceso al fichero.
3. `ByteBuffer` reserva espacio para los datos.
4. `channel.read(buffer, 32)` solicita la lectura comenzando en la posición `32`.
5. `flip()` prepara el buffer para su lectura desde Java.
6. `getInt()` y `getDouble()` interpretan los bytes según la estructura definida.
7. No es necesario recorrer los registros anteriores.

Este mecanismo proporciona un nivel de control mayor que los métodos de alto nivel de `Files`.

---

### 5.6. Cierre de recursos

Cuando trabajamos con recursos que permanecen abiertos, debemos cerrarlos al terminar.

No hacerlo puede provocar **fugas de recursos (`Resource Leaks`)**.

Por ejemplo, una aplicación que abre continuamente ficheros sin cerrarlos puede terminar alcanzando el límite de recursos permitido por el sistema:

```text
Proceso Java
     │
     ├── Abre fichero
     ├── Abre fichero
     ├── Abre fichero
     ├── Abre fichero
     │
     │       ...
     │
     ▼
Límite de recursos alcanzado
     │
     ▼
Error al intentar abrir nuevos recursos
```

Entre las consecuencias podemos encontrar:

* Recursos del sistema ocupados innecesariamente.
* Imposibilidad de abrir nuevos ficheros.
* Errores de entrada/salida.
* Problemas de rendimiento.
* Fallos en aplicaciones que trabajan con muchos recursos simultáneamente.

Por este motivo, **todo recurso que permanezca abierto debe cerrarse correctamente**.

---

### 5.7. Try-with-resources

Java proporciona **try-with-resources** para automatizar el cierre de recursos que implementan `AutoCloseable`.

Su estructura es:

```java
try (Recurso recurso = abrirRecurso()) {

    // Operaciones realizadas utilizando el recurso.

} catch (IOException e) {

    // Tratamiento de posibles errores de entrada/salida.
    log.error("Error de entrada/salida", e);
}
```

Al abandonar el bloque `try`, el recurso se cierra automáticamente, incluso si se produce una excepción.

Por ejemplo, con `FileChannel`:

```java
try (FileChannel channel = FileChannel.open(
        path,
        StandardOpenOption.READ)) {

    // Realizamos las operaciones necesarias
    // mientras el canal permanece abierto.
    // ...

} catch (IOException e) {

    // Si se produce un error, lo gestionamos aquí.
    log.error("Error al acceder al fichero", e);
}
```

No necesitamos llamar manualmente a:

```java
channel.close();
```

El propio mecanismo de **try-with-resources** se encarga del cierre.

---

### 5.8. Ejemplo práctico actual: registrar información en un log

Un caso habitual en aplicaciones Java es añadir información a un fichero de logs.

Podemos utilizar `Path`, `Files` y `StandardOpenOption`:

```java
@Override
public void run(String... args) {

    // Ruta del fichero donde almacenaremos los registros.
    Path path = Path.of("app.log");

    // Abrimos un BufferedWriter.
    // CREATE crea el fichero si todavía no existe.
    // APPEND conserva el contenido y escribe al final.
    try (BufferedWriter writer = Files.newBufferedWriter(
            path,
            StandardOpenOption.CREATE,
            StandardOpenOption.APPEND)) {

        // Escribimos el mensaje en el fichero.
        writer.write("Operación registrada correctamente.");

        // Añadimos un salto de línea para separar registros.
        writer.newLine();

        // Informamos del resultado en el log de la aplicación.
        log.info("Registro añadido al fichero de logs.");

    } catch (IOException e) {

        // Gestionamos los posibles errores de escritura.
        log.error("Error al escribir en el fichero de logs", e);
    }
}
```

Aquí utilizamos:

* `Path` para representar la ubicación.
* `Files.newBufferedWriter()` para obtener un escritor.
* `CREATE` para crear el fichero si no existe.
* `APPEND` para conservar el contenido existente y añadir el nuevo al final.
* `try-with-resources` para cerrar automáticamente el escritor.

Este patrón combina varias buenas prácticas actuales de la API de ficheros de Java.

---

### 5.9. ¿Qué debemos utilizar en Java moderno?

No todas las APIs de ficheros tienen el mismo nivel de abstracción.

| Necesidad                  | Opción recomendada           |
| -------------------------- | ---------------------------- |
| Representar una ruta       | `Path`                       |
| Comprobar existencia       | `Files.exists()`             |
| Leer texto completo        | `Files.readString()`         |
| Escribir texto completo    | `Files.writeString()`        |
| Procesar líneas            | `Files.lines()`              |
| Leer progresivamente       | `Files.newBufferedReader()`  |
| Escribir progresivamente   | `Files.newBufferedWriter()`  |
| Acceso avanzado a bytes    | `FileChannel` + `ByteBuffer` |
| Acceso directo tradicional | `RandomAccessFile`           |

#### ¿Y qué ocurre con `java.io.File` y `RandomAccessFile`?

Siguen formando parte de Java y podemos encontrarlos en aplicaciones existentes, por lo que es importante conocerlos.

Sin embargo, para **código nuevo**, nuestra primera opción debería ser normalmente:

```text
Path + Files
```

y, cuando necesitamos un control más avanzado sobre la entrada/salida:

```text
Path + FileChannel + ByteBuffer
```

`RandomAccessFile` queda como una API clásica que sigue siendo válida y que resulta interesante conocer, especialmente para comprender código existente y determinadas soluciones de acceso aleatorio.

---

### 5.10. Resumen del ciclo de vida

Podemos resumir el trabajo con ficheros en Java moderno de la siguiente forma:

```text
                    RUTA
                     │
                     ▼
                   Path
                     │
                     ▼
              ┌─────────────┐
              │    Files    │
              └──────┬──────┘
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
       Leer       Escribir   Gestionar
       datos       datos      fichero
          │          │
          └────┬─────┘
               │
               ▼
        ¿Necesitamos
       acceso avanzado?
               │
          ┌────┴────┐
         NO         SÍ
          │          │
          ▼          ▼
        Files    FileChannel
                     +
                 ByteBuffer
                     │
                     ▼
              Cerrar recurso
```

> **Idea clave**
>
> En Java moderno, comenzamos normalmente con **`Path` + `Files`**.
>
> Para ficheros pequeños y operaciones sencillas podemos utilizar `readString()` y `writeString()`.
>
> Para ficheros grandes o procesamiento progresivo podemos utilizar `lines()`, lectores y escritores.
>
> Cuando necesitamos un control más avanzado sobre bytes y posiciones, podemos utilizar **`FileChannel` + `ByteBuffer`**.
>
> Y cuando trabajamos con recursos abiertos, **try-with-resources** es la forma recomendada de garantizar su cierre.


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
