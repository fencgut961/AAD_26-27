# 🧠 Reto de diseño y POO: El sistema de personal de AadTex

## 🎯 Objetivo

Antes de comenzar a trabajar con **Acceso a Datos**, vamos a recuperar algunos de los conceptos de Java y Programación Orientada a Objetos que necesitaremos durante el curso.

En este reto tendréis que analizar un problema realista, identificar las entidades que intervienen, diseñar sus relaciones y construir una pequeña aplicación de consola.

El reto se realizará en dos fases:

1. **Diseño en la pizarra:** análisis del problema y diseño del modelo UML.
2. **Implementación en IntelliJ:** desarrollo de la solución en Java, Spring Boot y JUnit.

> ⚠️ **Importante:** no comencéis a programar hasta haber terminado el diseño inicial en la pizarra.

---

# 🏢 1. El problema

**AadTex** es una multinacional del sector textil que cuenta con empleados en diferentes áreas de la compañía.

Entre ellos encontramos trabajadores de sus **tiendas** y profesionales de su departamento de **Tecnología**.

La empresa necesita una pequeña aplicación que permita gestionar esta información y realizar algunas operaciones básicas sobre sus empleados.

El sistema debe ser capaz de representar:

* Los empleados y sus características comunes.
* Los distintos tipos de puestos existentes.
* Los centros de trabajo.
* Las relaciones jerárquicas entre empleados.
* El equipamiento corporativo asignado.
* Los proyectos tecnológicos en los que participa el personal de IT.
* El procesamiento de la nómina de los empleados.

Inicialmente, toda la información se almacenará **en memoria**. No habrá base de datos ni API REST.

---

# 🧩 2. Analiza antes de programar

Antes de crear ninguna clase, identifica las entidades y conceptos que aparecen en el problema.

Pregúntate:

* ¿Qué objetos tienen identidad propia?
* ¿Qué información pertenece a cada objeto?
* ¿Existen objetos que compartan características?
* ¿Hay diferentes tipos de empleados?
* ¿Qué relaciones existen entre ellos?
* ¿Qué relaciones son de uno a muchos o de muchos a muchos?
* ¿Qué comportamiento debería depender del tipo de empleado?
* ¿Qué información debería representarse mediante un `enum`?
* ¿Qué colecciones serían adecuadas para representar las relaciones?

Todas estas decisiones deberán quedar reflejadas en vuestro **diagrama de clases UML**.

---

# 👥 3. Empleados

El sistema debe permitir representar, como mínimo, dos grandes grupos de empleados.

## Personal de tienda

Un empleado de tienda tiene:

* Identificador.
* Nombre.
* Correo electrónico.
* Salario base.
* Rol dentro de la tienda.
* Bonus mensual asociado a las ventas.

Los roles disponibles son:

* `CLERK`
* `SHOP_MANAGER`

## Personal de IT

Un empleado de tecnología tiene:

* Identificador.
* Nombre.
* Correo electrónico.
* Salario base.
* Rol dentro del departamento de IT.
* Bonus asociado a proyectos.
* Información sobre si trabaja de forma remota.
* Proyectos en los que participa.

Los roles disponibles son:

* `DEVELOPER`
* `UX_DESIGNER`
* `PROJECT_MANAGER`

### 💭 Para debatir

> ¿Tiene sentido crear una única clase `Employee` con todos estos atributos?

> ¿Qué información es común a todos los empleados?

> ¿Qué información pertenece únicamente a determinados tipos de empleados?

> ¿Utilizaríais herencia? ¿Dónde?

---

# 🏢 4. Centros de trabajo

Cada empleado pertenece a un centro de trabajo.

Un centro de trabajo debe disponer, como mínimo, de:

* `id`
* `name`
* `city`

Un centro puede tener **varios empleados**, mientras que cada empleado pertenece a **un único centro**.

Los empleados que trabajen completamente en remoto estarán asociados conceptualmente a un centro virtual denominado:

**Virtual Remote Hub**

### 💭 Para debatir

Representad en UML:

* ¿Qué tipo de relación existe?
* ¿Cuál es la multiplicidad?
* ¿En qué clase debería aparecer la referencia?

---

# 👔 5. Jerarquía de empleados

Los empleados pueden tener un supervisor.

Un empleado puede:

* Tener un supervisor.
* No tener supervisor si ocupa la máxima posición de su estructura.
* Ser supervisor de otros empleados.

El supervisor es también un `Employee`.

### 💭 Para debatir

Representad esta relación en UML.

Después responded:

> ¿Qué multiplicidades aparecen?

> ¿Puede un empleado supervisar a varios empleados?

> ¿Puede existir un empleado sin supervisor?

> ¿Debería el modelo impedir que cualquier empleado pueda supervisar a cualquier otro?

Por ejemplo:

> ¿Tiene sentido que un `CLERK` tenga como supervisor directo a un `DEVELOPER`?

No existe necesariamente una única respuesta correcta: **lo importante es justificar vuestra decisión de diseño**.

---

# 💻 6. Equipamiento corporativo

La empresa proporciona diferentes recursos y dispositivos a sus empleados.

Cada equipamiento debe disponer, como mínimo, de:

* `id`
* `serialNumber`
* `model`
* Tipo de equipamiento.

Los tipos disponibles son:

* `PC_LAPTOP`
* `MONITOR`
* `ERGONOMIC_CHAIR`
* `CORPORATE_PHONE`
* `COMPANY_CAR`
* `WORK_CLOTHING`

Un empleado puede tener **varios elementos de equipamiento**.

### 💭 Para debatir

Diseñad la relación entre empleado y equipamiento.

Pensad también:

> ¿El equipamiento tiene identidad propia?

> ¿Puede un mismo equipamiento estar asignado simultáneamente a dos empleados?

> ¿Qué colección utilizaríais para representar el inventario?

---

# 🚀 7. Proyectos tecnológicos

El personal de IT participa en diferentes proyectos de la compañía.

Cada proyecto debe disponer, como mínimo, de:

* `id`
* `name`
* `technology`

Un empleado de IT puede participar en uno o varios proyectos.

Un proyecto puede tener varios empleados de IT.

### 💭 Para debatir

Representad esta relación en UML.

> ¿Qué tipo de relación existe?

> ¿Qué multiplicidades tiene?

> ¿Utilizaríais `List` o `Set` para representar los proyectos de un empleado?

> ¿Qué problema podría aparecer si un empleado se añade dos veces al mismo proyecto?

No os limitéis a elegir una colección: **justificad la decisión**.

---

# 💰 8. Procesamiento de nóminas

La aplicación debe poder procesar la nómina de todos los empleados.

Cada empleado tiene un salario bruto que se obtiene a partir de su salario base y los complementos que correspondan.

Además, se aplica una retención en función del rol del empleado.

## Personal de tienda

| Rol            | Retención |
| -------------- | --------: |
| `CLERK`        |      10 % |
| `SHOP_MANAGER` |      12 % |

## Personal de IT

| Rol               | Retención |
| ----------------- | --------: |
| `DEVELOPER`       |      15 % |
| `UX_DESIGNER`     |      15 % |
| `PROJECT_MANAGER` |      18 % |

El sistema deberá obtener finalmente el **salario neto**.

### 💭 Para debatir

> ¿Dónde debería vivir la lógica específica del cálculo de la nómina?

> ¿Qué comportamiento es común a todos los empleados?

> ¿Qué comportamiento cambia según el tipo de empleado?

> ¿Debería el porcentaje de retención formar parte de los propios `enum`?

---

# 🧠 9. Polimorfismo: no preguntes qué eres

El servicio encargado de procesar las nóminas debe poder trabajar con cualquier tipo de empleado.

Conceptualmente:

```text
para cada empleado
    procesar su nómina
```

El servicio **no debe preguntar qué tipo concreto de empleado está procesando**.

Por tanto, no se permitirá utilizar:

```java
instanceof
```

ni construir un gran bloque de `if / else if / else` para decidir qué tipo de empleado es.

El comportamiento deberá resolverse mediante **polimorfismo**.

### 💭 Para debatir

> ¿Dónde debería vivir la lógica específica del cálculo de la nómina?

> ¿Qué método común podría definir `Employee`?

> ¿Cómo conseguiríamos que cada tipo de empleado realizase su propio cálculo?

---

# 🔄 10. El reto del cambio

Imaginad que dentro de unos meses AadTex crea una nueva división:

## 🚚 Logística

Aparecen nuevos empleados con:

* Sus propios roles.
* Complementos específicos.
* Reglas salariales diferentes.
* Sus propias características.

El código que actualmente procesa las nóminas **no debería necesitar modificaciones** para incorporar esta nueva categoría.

### 🎯 Test del minuto

Pregúntate:

> **Si mañana aparece un nuevo tipo de empleado, ¿cuántos lugares de mi programa tendría que modificar?**

El objetivo es aplicar correctamente el principio **Open/Closed**.

---

# 🗄️ 11. Repositorio en memoria

La aplicación necesita almacenar los empleados mientras está ejecutándose.

Para ello se utilizará un componente denominado:

`EmployeeRepository`

El repositorio será responsable de operaciones básicas sobre los empleados, como:

* Añadir empleados.
* Buscar empleados.
* Obtener todos los empleados.
* Eliminar empleados.

Los datos se almacenarán únicamente **en memoria**, utilizando una estructura basada en:

`ConcurrentHashMap`

No se utilizará ninguna base de datos.

### 💭 Para debatir

> ¿Por qué es interesante separar el repositorio del servicio?

> ¿Qué ventaja tendría poder cambiar posteriormente la implementación del repositorio sin modificar la lógica de negocio?

Esta decisión será especialmente importante cuando comencemos a trabajar con **Acceso a Datos**.

---

# 🌱 12. Spring Boot

La aplicación será un proyecto sencillo de **Spring Boot**, sin API REST.

No necesitamos:

* Controladores.
* Endpoints.
* JSON.
* Jackson.
* Interfaz web.

La aplicación funcionará desde consola.

Al iniciar Spring Boot, un `CommandLineRunner` deberá crear un escenario de prueba que incluya:

* Varios centros de trabajo.
* Empleados de tienda.
* Empleados de IT.
* Diferentes roles.
* Relaciones jerárquicas.
* Equipamiento.
* Proyectos.
* Empleados participando en varios proyectos.

Una vez creado el escenario, la aplicación deberá:

1. Mostrar información relevante de los empleados.
2. Procesar sus nóminas.
3. Mostrar el salario bruto y neto.
4. Mostrar el equipamiento asignado.
5. Mostrar los proyectos correspondientes.
6. Mostrar las relaciones de supervisión.

Para los mensajes por consola se utilizará el logger de Lombok mediante `@Slf4j`.

---

# 🧪 13. Pruebas automatizadas con JUnit

Además de comprobar manualmente el funcionamiento de la aplicación, utilizaremos **JUnit** para crear pruebas automatizadas.

El objetivo es recordar cómo comprobar mediante código que nuestro programa se comporta como esperamos.

No es necesario crear una batería exhaustiva de pruebas. Nos centraremos en los comportamientos más importantes.

Como mínimo, probaremos:

### 💰 Nóminas

Diferentes casos de cálculo de nómina, incluyendo los distintos roles.

Por ejemplo:

```text
DADO un empleado con:

Salario base: 1500 €
Bonus:          200 €
Rol:           CLERK

CUANDO se procesa su nómina

ENTONCES:

Bruto = 1700 €
Neto  = 1530 €
```

El test deberá comprobar automáticamente que el resultado obtenido coincide con el esperado.

### 🗄️ Repositorio

También probaremos el repositorio en memoria:

* Guardar un empleado.
* Buscar un empleado.
* Obtener los empleados.
* Eliminar un empleado.
* Comprobar qué ocurre cuando buscamos un empleado que no existe.

### 🔗 Relaciones

Podremos crear pruebas para comprobar algunos comportamientos del modelo:

* Asociación con un centro de trabajo.
* Asignación de equipamiento.
* Asignación de proyectos.
* Relación de supervisión.

### 🚫 Importante

Los tests deben comprobar **comportamiento real**.

No sirve simplemente con comprobar que un objeto no es `null` o utilizar una aserción que siempre sea verdadera.

La idea es que, si introducimos un error en el código, **alguna de nuestras pruebas sea capaz de detectarlo**.

---

# ⚙️ 14. Requisitos técnicos

La aplicación deberá utilizar:

* Java moderno.
* Programación Orientada a Objetos.
* Encapsulación.
* Herencia cuando esté justificada.
* Polimorfismo.
* `enum`.
* Colecciones adecuadas al problema.
* `ConcurrentHashMap`.
* Spring Boot.
* `CommandLineRunner`.
* Lombok.
* `@Slf4j`.
* Inyección de dependencias mediante constructor.
* Atributos `final` para las dependencias.
* JUnit para las pruebas automatizadas.

No se utilizará:

* API REST.
* Base de datos.
* `instanceof` para resolver el tipo de empleado durante el procesamiento de nóminas.

---

# 🧠 15. Al finalizar el reto...

No buscamos solamente que la aplicación funcione.

El objetivo es que seáis capaces de explicar las decisiones tomadas durante el diseño.

Al terminar deberíais poder responder, entre otras, a preguntas como:

* ¿Por qué `Employee` puede ser abstracta?
* ¿Por qué utilizamos herencia?
* ¿Dónde aparece el polimorfismo?
* ¿Por qué evitamos `instanceof`?
* ¿Qué diferencia hay entre una relación 1:N y una N:M?
* ¿Por qué usamos `Set` en determinadas relaciones?
* ¿Qué responsabilidad tiene un repositorio?
* ¿Por qué el servicio no debería acceder directamente al `Map`?
* ¿Qué ventajas tiene la inyección por constructor?
* ¿Qué comprueba un test unitario?
* ¿Qué ocurre cuando cerramos la aplicación?

---

# 🚀 16. Y ahora viene Acceso a Datos...

Hasta ahora todos nuestros datos viven en memoria:

```text
EmployeeService
       ↓
EmployeeRepository
       ↓
ConcurrentHashMap
       ↓
    MEMORIA
```

Pero cerramos la aplicación...

**💥 Los datos desaparecen.**

¿Qué ocurre con los empleados?

¿Con los proyectos?

¿Con el equipamiento?

¿Con las relaciones entre empleados?

### ❓ El nuevo problema

> **¿Cómo conseguimos que nuestros datos sobrevivan al cierre de la aplicación?**

Esta será precisamente una de las preguntas que comenzaremos a responder en **Acceso a Datos**.

Durante el curso podremos evolucionar nuestro repositorio hacia:

```text
EmployeeService
       ↓
EmployeeRepository
       ↓
      JDBC
       ↓
   PostgreSQL
```

y posteriormente:

```text
EmployeeService
       ↓
EmployeeRepository
       ↓
Spring Data JPA
       ↓
   PostgreSQL
```

### 🎯 La idea que nos llevamos

**Los objetos representan nuestro dominio, los servicios contienen la lógica de negocio y el repositorio se ocupa de cómo almacenamos los datos.**

Ahora vamos a aprender qué ocurre cuando esos datos dejan de vivir solamente en memoria.
