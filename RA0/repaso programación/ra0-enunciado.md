# 🧠 Reto de Diseño y POO: El Motor de Personal de AadTex (v3 - Edición Alumnos)

Este documento contiene el enunciado oficial del reto de diseño y programación orientada a objetos para realizar en el aula. El objetivo es diseñar primero el modelo conceptual (diagrama de clases UML) de forma colectiva en la pizarra y, posteriormente, implementar la solución en vuestro entorno de desarrollo local.

---

## 🏢 1. Contexto del Problema

La multinacional textil **AadTex (Inditex)** necesita desarrollar un núcleo lógico (microservicio simulado en memoria) para gestionar el personal de sus oficinas centrales de tecnología (IT) y de sus tiendas físicas de retail.

Como arquitectos de software, vuestra misión es diseñar un sistema extremadamente flexible, tolerante a cambios rápidos y libre de los vicios típicos de desarrollo (como el acoplamiento fuerte, la mutabilidad descontrolada o el uso de condicionales de tipo para resolver comportamiento). El diseño debe realizarse íntegramente en la pizarra antes de abrir el entorno de desarrollo IntelliJ.

---

## 📋 2. Elementos de Clasificación (Enums de Dominio)

Para garantizar la tipificación estricta del sistema y evitar el uso de cadenas de texto libres ("strings mágicos"), debéis modelar los siguientes enumerados:

### A) Roles de Tienda (`ShopRole`)
*   `CLERK` (Dependiente de comercio / atención al público).
*   `SHOP_MANAGER` (Responsable o encargado del establecimiento).

### B) Roles de Tecnología (`ITRole`)
*   `DEVELOPER` (Desarrollador de software de aplicaciones).
*   `UX_DESIGNER` (Diseñador de experiencia de usuario e interfaces).
*   `PROJECT_MANAGER` (Jefe de proyecto / líder técnico de equipo).

### C) Tipos de Equipamiento Corporativo (`EquipmentType`)
*   `PC_PORTATIL` (Ordenadores portátiles para el trabajo diario).
*   `MONITOR` (Pantallas externas para desarrollo o gestión).
*   `SILLA_ERGONOMICA` (Sillas de oficina homologadas para teletrabajo o presencial).
*   `MOVIL_CORPORATIVO` (Teléfonos móviles corporativos o terminales PDA de tienda).
*   `COCHE_EMPRESA` (Vehículos de empresa asignados a puestos específicos).
*   `ROPA_TRABAJO` (Uniformes y ropa corporativa para empleados de tienda).

---

## 🔄 3. El Mapa de Relaciones (Pizarra y UML)

Debéis representar en la pizarra el diagrama de clases que soporte las siguientes estructuras y tipos de relaciones en memoria (utilizando nombres de clases, atributos y métodos en inglés profesional):

### Relación 1: Herencia y Especialización
*   **`Employee` (Clase Madre Abstracta)**: Agrupa los datos esenciales comunes a cualquier trabajador (`id`, `name`, `email`, `baseSalary`, `netSalary`).
*   **`EmployeeShop` (Clase Hija)**: Representa al personal de tienda. Añade su rol específico (`ShopRole`) y un plus mensual por comisiones de venta de la tienda (`salesBonus`).
*   **`EmployeeIT` (Clase Hija)**: Representa al equipo de ingeniería. Añade su rol específico (`ITRole`), un plus por proyecto tecnológico (`projectBonus`), un indicador booleano de teletrabajo (`remote`) y su listado de proyectos.

### Relación 2: Asociación Simple con Centros de Trabajo (`1:N`)
*   Cada empleado pertenece a un único centro de trabajo: **`WorkCenter`** (`id`, `name`, `city`). Un centro alberga múltiples trabajadores.
*   *Nota de diseño*: Para aquellos empleados de IT que trabajen de forma 100% remota (`remote = true`), se les asociará conceptualmente a un centro de trabajo virtual denominado *"Virtual Remote Hub"*.

### Relación 3: Asociación Autorreferencial Unidireccional de Reporte (`1:1`)
*   Cualquier empleado (`Employee`) puede tener asignado un **`supervisor`** (que es, a su vez, otro `Employee`). Si un empleado no tiene supervisor (por ejemplo, el director general o el máximo responsable del centro), esta referencia será `null`.
*   *Pregunta de debate*: ¿Debería poder un dependiente (`CLERK`) de tienda tener como supervisor a un desarrollador (`DEVELOPER`) de IT? ¿Cómo controlamos la coherencia de jerarquías en la pizarra?

### Relación 4: Asociación de Equipamientos Corporativos (`1:N`)
*   Cada empleado de la compañía tiene asignado un inventario de múltiples dispositivos y recursos de trabajo corporativos. Debéis modelar la clase **`Equipment`** (`id`, `serialNumber`, `model`, `type` de tipo `EquipmentType`) y relacionarla con el empleado de modo que cada uno contenga un listado de sus equipos (`List<Equipment>`).

### Relación 5: Relación Muchos a Muchos en Tecnología (`N:M`)
*   Los profesionales de IT colaboran de forma transversal en proyectos tecnológicos de la compañía. Se debe modelar la entidad **`Project`** (`id`, `name`, `technology`).
*   **Restricciones**:
    *   Un empleado de IT puede estar asignado a **uno o varios** proyectos simultáneamente (con un mínimo obligatorio de 1 proyecto activo).
    *   Un proyecto puede contar con múltiples ingenieros asignados a la vez.
    *   *Pregunta de debate*: ¿Por qué es óptimo modelar la colección de proyectos en el empleado de IT como un conjunto (`Set<Project>`) en lugar de una lista (`List<Project>`) en memoria?

---

## 🚫 4. Las 4 Reglas de Oro del Diseño (Restricciones Técnicas)

Para forzar un diseño robusto y de calidad profesional que evite malas prácticas de programación, vuestro código deberá cumplir estrictamente las siguientes limitaciones de arquitectura:

### 🧠 Regla 1: Prohibido preguntar \"Qué eres\" (Cero `instanceof` en el servicio)
*   En el bucle del servicio que se encarga de procesar las nóminas de la plantilla, **está totalmente prohibido realizar comprobaciones de tipo de clase** (como `if (emp instanceof EmployeeIT)` o similares). La capa que procesa las nóminas no debe saber qué tipos específicos de empleados existen; simplemente debe delegar la responsabilidad en el comportamiento polimórfico definido en el dominio a través del método abstracto `processPayroll()`.

### 💸 Regla 2: Autonomía Fiscal y Salarial (Polimorfismo de Comportamiento)
*   El cálculo del **salario bruto** de cada empleado debe resolverse de manera autónoma en función de sus variables específicas de comisiones o bonus.
*   La **deducción impositiva (IRPF)** no es plana, sino que varía según la responsabilidad de su rol específico dentro de la organización:
    *   **Área de Tiendas (`EmployeeShop`)**:
        *   Si su rol es `CLERK` (Dependiente): Retención del **10%** de su salario bruto.
        *   Si su rol es `SHOP_MANAGER` (Responsable): Retención del **12%** de su salario bruto.
    *   **Área de IT (`EmployeeIT`)**:
        *   Si su rol es `DEVELOPER` o `UX_DESIGNER`: Retención del **15%** de su salario bruto.
        *   Si su rol es `PROJECT_MANAGER` (Jefe de Proyecto): Retención del **18%** de su salario bruto.
*   El método `processPayroll()` de cada subclase debe calcular el bruto, deducir de forma autónoma el porcentaje impositivo correspondiente al rol del enum y fijar el salario neto (`netSalary`) resultante.

### 🛠️ Regla 3: El Test del Minuto (Principio Open-Closed)
*   El diseño del microservicio de nóminas debe ser capaz de superar "El Test del Minuto": si mañana Inditex decide incorporar una nueva división de **Logística** (con sus propios pluses de nocturnidad, roles y tramos de IRPF específicos), **debemos poder integrarla en el sistema creando una única clase nueva, sin tener que modificar ni una sola línea de código de vuestro bucle de procesamiento de nóminas masivo**.

### 🔒 Regla 4: Inyección de Dependencias por Constructor e Inmutabilidad
*   Para asegurar el desacoplamiento de componentes y prevenir fallos en tiempo de ejecución (como el temido `NullPointerException`), la relación entre vuestro `EmployeeService` y el `EmployeeRepository` debe ser inmutable. Declararéis la dependencia como `private final` y realizaréis la inyección obligatoriamente por constructor utilizando la anotación `@RequiredArgsConstructor` de Lombok (quedando prohibido el uso de `@Autowired` de campo).

---

## ⚙️ 5. Requisitos Técnicos y de Consola

*   **Persistencia en Memoria**: El almacenamiento de datos en vuestro repositorio (`EmployeeRepository`) debe gestionarse de forma segura utilizando colecciones concurrentes (un mapa de tipo `ConcurrentHashMap`) para simular la persistencia real.
*   **Cero API REST**: Para evitar distracciones con controladores web, serializadores Jackson o configuraciones HTTP, el proyecto será de consola pura.
*   **CommandLineRunner**: Al arrancar la aplicación de Spring Boot, el método `run()` del runner debe precargar un escenario representativo de prueba (con centros de trabajo, proyectos corporativos, múltiples equipamientos asignados a cada empleado y jerarquías claras de quién supervisa a quién). Posteriormente, debe lanzar de manera masiva el procesamiento de nóminas y mostrar por consola (haciendo uso del logger `@Slf4j` de Lombok) los resultados del salario neto definitivo y el desglose de inventario de cada empleado.
