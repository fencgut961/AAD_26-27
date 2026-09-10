# 🏬 UD01. Práctica Evaluatoria: AadTex Store Ingestion Pipeline
## Sistema de Ingestión, Cifrado, Transcodificación, Auditoría Nocturna, Validación de Parejas y Programación de Tareas (Scheduler)

**Módulo:** Acceso a Datos (AAD)  
**Unidad Didáctica:** UD01 – Manejo de Ficheros (RA1)  
**Duración Estimada:** 4 Horas  
**Entregables:** Repositorio **GitHub** con el proyecto Spring Boot / Java, archivo **`ra1_act1.zip`** subido a Moodle y defensa oral presencial.

---

## 1. Contexto de Negocio Empresarial y Problema a Resolver (Caso AadTex)

### ❓ ¿Cuál es el problema real que estamos resolviendo?
Una multinacional textil con tiendas físicas distribuidas por todo el mundo (**AadTex**) necesita **centralizar e ingerir automáticamente cada noche las ventas, balances financieros e imágenes de firmas de todas sus tiendas** (España, EE. UU., China, etc.) sin intervención humana manual.

Sin un pipeline de ingeniería de software automatizado, la empresa sufre problemas de integración críticos:
1. **Fugas de Información Confidencial**: Los balances financieros no pueden viajar desprotegidos por Internet; requieren **cifrado simétrico en origen (AES)**.
2. **Corrupción de Caracteres Internacionales (*Mojibake*)**: Las tiendas de China envían caracteres CJK (Chino, Japonés, Coreano) y las de España tildes y eñes. Si no se fuerzan codificaciones explícitas (**`UTF-8`**), los nombres de productos y tiendas se corrompen.
3. **Peligro de Colapso de Memoria (RAM)**: Cargar imágenes o ficheros gigantes enteros en la memoria del servidor provocaría un error `OutOfMemoryError`. Se exige el uso de **flujos con buffer de 4 KB**.
4. **Búsquedas Lentas de Auditoría**: Buscar el estado de procesamiento de una tienda de hace 6 meses en un fichero `.txt` requiere leer millones de filas. Se exige un libro diario binario de **acceso aleatorio (`RandomAccessFile`) con registros fijos de 32 bytes** para realizar consultas instantáneas mediante saltos directos en bytes (`seek`).
5. **Persistencia Heterogénea y Nube**: Los informes consolidados deben respaldarse emulando un almacén de objetos en la nube (**Bucket S3**) bajo un espacio de nombres organizado por fecha.
6. **Integridad de Parejas de Ficheros**: Un lote de tienda válido debe contener **tanto el fichero de ventas como la imagen de la firma correspondiente**. Si llega un archivo huérfano, vacío (0 bytes) o corrupto, se debe emitir una alerta simulada a soporte técnico (`soporte@aadtex.com`) y moverlo a la carpeta de errores.

---

## 2. Lote Diario por Tienda y Estructura Completa de `/store_data/incoming/`

### 📦 ¿Qué deposita cada tienda física al cerrar?
Cada noche a las 03:00 AM, **cada tienda deposita un lote (*batch*) indivisible compuesto por 2 a 3 ficheros** en la carpeta `/store_data/incoming/`:

| Fichero de la Tienda | Formato / Extensión | Finalidad y Tratamiento Técnico |
| :--- | :---: | :--- |
| **1. Ventas de la Tienda** | `.csv` (Texto) **o** `.enc` (Cifrado) | Transacciones del día. Si son estándar van en `.csv` (`UTF-8`); si son datos financieros confidenciales vienen encriptados en `.enc` (se descifran en caliente en RAM). |
| **2. Comprobante de Firma** | `.jpg` (Binario) | Imagen escaneada con la conformidad del encargado (`signature_*.jpg`). Se copia mediante buffer de 4 KB. |
| **3. Fichero Defectuoso / Incompleto** | `.csv` / `.enc` (Vacío / 0B / Sin firma) | Lote corrupto o incompleto. El pipeline debe detectarlo, lanzar alerta a soporte técnico (`soporte@aadtex.com`) y moverlo a `/error`. |

---

### 🏬 Estructura real que encuentra el servidor central en `/store_data/incoming/`

Al llegar la hora de proceso, la carpeta `/store_data/incoming/` contendrá la siguiente estructura real de ficheros de prueba:

```text
/store_data/incoming/
├── daily_sales_spain.csv         <-- Lote 1: Tienda España (UTF-8 con tildes y eñes)
├── signature_spain.jpg           <-- Lote 1: Comprobante de firma de España (.jpg)
├── daily_sales_china.csv         <-- Lote 2: Tienda China (UTF-8 con caracteres CJK)
├── signature_china.jpg           <-- Lote 2: Comprobante de firma de China (.jpg)
├── daily_sales_financial.enc     <-- Lote 3: Tienda Central/Financiera (Cifrado AES)
├── signature_financial.jpg       <-- Lote 3: Comprobante de firma de la central (.jpg)
└── daily_sales_broken_store.csv  <-- Lote 4: Fichero defectuoso de 0 bytes (dispara alerta e-mail/log)
```

---

## 3. Objetivos de Ingeniería de Software

El alumno debe desarrollar una solución profesional en Java / Spring Boot que demuestre el dominio del **RA1 (Manejo de Ficheros)** e integración con Spring Boot:

* [x] **Automatización con Spring Scheduler (`@EnableScheduling` / `@Scheduled`)**: Tarea programada en segundo plano para escaneo automático continuo de ficheros.
* [x] **Gestión Moderna de Archivos (NIO.2)**: Uso de `Path`, `Files.exists()`, `Files.createDirectories()`, `Files.move()` y `Files.list()` para controlar los directorios `/incoming`, `/processed` y `/error`.
* [x] **Validación de Integridad de Parejas y Alertas**: Detección de archivos huérfanos sin firma o vacíos (0 bytes), emitiendo logs de error de alerta a soporte (`soporte@aadtex.com`).
* [x] **Criptografía de Ficheros (`javax.crypto`)**: Desencriptación atómica en caliente en la memoria RAM de los ficheros cifrados (`.enc`) sin dejar copia en claro en disco.
* [x] **Transcodificación y Charsets (`java.io`)**: Lectura de archivos CSV usando `InputStreamReader` con `StandardCharsets.UTF_8` de forma explícita para dar soporte multi-idioma (España, EE. UU., China).
* [x] **Procesamiento Binario con Buffer**: Copia eficiente de comprobantes gráficos (`.jpg`) mediante `BufferedInputStream` y `BufferedOutputStream` en bloques de 4 KB.
* [x] **Acceso Aleatorio Indexado (`RandomAccessFile`)**: Mantenimiento de un libro diario binario (`audit_ledger.dat`) con registros fijos de **32 bytes** y saltos con `seek()`.
* [x] **Simulación de Cloud Object Storage**: Abstracción de un servicio de almacenamiento en la nube guardando los reportes JSON en la carpeta local `store_data/bucket/`.

---

## 4. Arquitectura del Sistema y Estructura de Directorios

El programa debe gestionar automáticamente el siguiente árbol de directorios dentro de la raíz de la aplicación:

```text
/workspace/
└── store_data/
    ├── incoming/                 <-- Directorio de entrada con los ficheros de las tiendas
    ├── processed/                <-- Ficheros procesados correctamente
    │   └── signatures/           <-- Ficheros binarios de firma copiados (.jpg)
    ├── error/                    <-- Ficheros corruptos, vacíos o incompletos movidos
    ├── keys/                     <-- Clave secreta de descifrado (AadTexSecret.key)
    ├── audit/                    <-- Fichero binario de acceso aleatorio (audit_ledger.dat)
    └── bucket/                   <-- SIMULACIÓN LOCAL DEL BUCKET EN LA NUBE
        └── aadtex-daily-reports/
            └── 2026/
                └── 09/
                    ├── summary_batch_1.json
                    └── summary_batch_2.json
```

---

## 5. Requisitos Detallados de Desarrollo

### 📍 Requisito 1: Automatización del Demonio con Spring Scheduler (`@Scheduled`)
1. Activar el motor de programación de tareas en la clase principal de Spring Boot (`AadApplication.java`) anotándola con **`@EnableScheduling`**.
2. Crear un componente de servicio `StoreIngestionScheduler` anotado con **`@Component`**.
3. Implementar el método `processIncomingStoreBatches()` anotado con **`@Scheduled`**:
   - **En Producción**: Explicar en los comentarios que se usa la expresión Cron `@Scheduled(cron = "0 0 3 * * *")` (ejecución diaria a las 03:00 AM).
   - **En el Aula para Pruebas**: Configurar la ejecución periódica cada 5 segundos mediante `@Scheduled(fixedRate = 5000)`.

### 📍 Requisito 2: Escaneo, Validación de Parejas y Control de Flujo con NIO.2
1. En cada ciclo del Scheduler, verificar la existencia de los directorios. Si no existen, crearlos automáticamente mediante `Files.createDirectories()`.
2. Escanear el directorio `store_data/incoming/` buscando ficheros `.csv` y `.enc`.
3. **Validación de Ficheros Vacíos / Corruptos**: Si cualquier fichero tiene tamaño 0 bytes (`Files.size(path) == 0`) o está corrupto, el sistema debe:
   - Emitir un log de error severo simulando un envío de alerta a soporte:  
     `log.error("[ALERT EMAIL SENT -> soporte@aadtex.com] CRITICAL: Empty/Corrupt batch: {}", fileName);`
   - Mover el archivo de forma atómica al directorio `store_data/error/` usando `Files.move(..., StandardCopyOption.REPLACE_EXISTING)`.
4. **Validación de Integridad de Parejas**: Verificar que para cada fichero de ventas exista su correspondiente comprobante de firma `signature_*.jpg`. Si falta la firma o está el archivo huérfano, emitir log de alerta a soporte y moverlo a `store_data/error/`.

### 📍 Requisito 3: Criptografía y Desencriptación Cifrada (`AES-128`)
1. Los lotes cifrados (`daily_sales_financial.enc`) deben ser desencriptados en la memoria RAM mediante `CryptoService`.
2. Utilizar la clave secreta almacenada en `keys/AadTexSecret.key` (`"AadTexSecret2026"`).
3. Pasar la secuencia de bytes descifrada directamente al lector CSV usando `ByteArrayInputStream` sin guardar el texto en claro en el disco duro.

### 📍 Requisito 4: Transcodificación y Lectura de Datos CSV (`java.io`)
1. Leer las líneas del CSV usando `BufferedReader` e `InputStreamReader` especificando explícitamente `StandardCharsets.UTF_8`.
2. Soportar ficheros en español, inglés y chino (caracteres CJK) sin corromper la información.
3. Formato de las líneas CSV:
   ```csv
   StoreID,StoreName,SalesAmount,Category,TransactionDate
   ZARA_001,Zara Puerta del Sol Madrid,14500.50,WOMEN,2026-09-10
   ZARA_CH_01,Zara 上海南京东路店,28900.00,WOMEN,2026-09-10
   ```
4. Calcular el total acumulado de ventas del lote.

### 📍 Requisito 5: Copia de Ficheros Binarios mediante Buffers (4 KB)
1. Si en el lote existe una imagen de comprobante de firma (`signature_*.jpg`), copiarla a `store_data/processed/signatures/`.
2. Utilizar un buffer de transferencia de **4 KB (`byte[4096]`)** combinando `BufferedInputStream` y `BufferedOutputStream`.

### 📍 Requisito 6: Libro Diario de Auditoría en Acceso Aleatorio (`RandomAccessFile`)
1. Mantener el archivo binario `store_data/audit/audit_ledger.dat`.
2. Cada procesamiento de lote debe escribir **un registro de tamaño fijo de exactamente 32 bytes**:
   - `BatchID` (`int` -> **4 bytes**)
   - `StoreID Hash` (`int` -> **4 bytes**)
   - `Timestamp` (`long` -> **8 bytes** - `System.currentTimeMillis()`)
   - `TotalSales` (`double` -> **8 bytes**)
   - `StatusFlag` (`int` -> **4 bytes**: 1=SUCCESS, 2=ERROR)
   - `Reserved Padding` (`int` -> **4 bytes**: libre)
   - **Total:** 4 + 4 + 8 + 8 + 4 + 4 = 32 bytes.
3. Implementar un método `searchBatchAudit(int batchIndex)` que calcule el desplazamiento exacto `seek(batchIndex * 32)` y lea los datos del lote de forma instantánea sin recorrer el resto del fichero.

### 📍 Requisito 7: Generador de Ficheros de Prueba Multilenguaje
1. Implementar la clase `MultiLanguageDataGenerator.java` que genere automáticamente los 7 ficheros de prueba en `store_data/incoming/` al iniciar la aplicación (`CommandLineRunner`).

### 📍 Requisito 8: Reporte JSON y Simulación de Bucket en la Nube
1. Crear un archivo JSON de resumen `summary_batch_[ID].json`.
2. Invocando la clase de servicio `CloudBucketService`, guardar el objeto en la estructura de almacenamiento emulando un Bucket en la Nube dentro de `store_data/bucket/`:
   - **Bucket:** `aadtex-daily-reports`
   - **Key:** `2026/09/summary_batch_[ID].json`

---

## 6. Normativa Oficial de Entrega Moodle y Defensa Oral Presencial

1. **Entrega en Moodle (`ra1_act1.zip`)**:
   - La entrega se realizará exclusivamente a través de la plataforma **Moodle** subiendo un único archivo comprimido en formato ZIP denominado estrictamente **`ra1_act1.zip`**.
   - El archivo `.zip` debe contener todo el proyecto Spring Boot estructurado y listo con sus archivos de datos de prueba para que, al descomprimirlo, pueda compilarse y ejecutarse sin requerir ajustes adicionales.
2. **Vincular Repositorio GitHub en `README.md`**:
   - En la raíz del proyecto, el archivo **`README.md`** debe incluir obligatoriamente un enlace directo (*link*) al repositorio de GitHub donde se encuentra alojado el código fuente versionado.
3. **Defensa Oral Presencial**:
   - Para la defensa presencial, el profesor utilizará **únicamente el contenido del archivo `ra1_act1.zip` entregado en Moodle**.
   - La aplicación deberá arrancar con normalidad (`mvn spring-boot:run` o desde el IDE) y demostrar el procesamiento automatizado en tiempo real.
   - El alumno responderá de forma individual a preguntas teóricas y sobre el código Java desarrollado.
