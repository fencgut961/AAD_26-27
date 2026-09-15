# 🧠 Reto de diseño y POO: El sistema de personal de AadTex

## 🎯 Objetivo

Antes de comenzar a trabajar con **Acceso a Datos**, vamos a recuperar algunos de los conceptos de Java y Programación Orientada a Objetos que necesitaremos durante el curso.

En este reto tendréis que analizar un problema realista, identificar las entidades que intervienen, diseñar sus relaciones y construir una pequeña aplicación de consola.

El reto se realizará en dos fases:

1. **Diseño en la pizarra:** análisis del problema y diseño del modelo UML.
2. **Implementación en IntelliJ:** desarrollo de la solución en Java y Spring Boot.

> **Importante:** no comencéis a programar hasta haber terminado el diseño inicial.

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
* ¿Qué información puede repetirse?
* ¿Qué relaciones pueden ser de uno a muchos o de muchos a muchos?
* ¿Qué comportamiento debería depender del tipo de empleado?
* ¿Qué información debería representarse mediante un `enum`?

Todas estas decisiones deberán quedar reflejadas en vuestro **diagrama de clases UML**.

---

# 👥 3. Empleados

El sistema debe permitir representar, como mínimo, dos grandes grupos de empleados:

### Personal de tienda

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

### Personal de IT

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

Antes de diseñar las clases, plantead:

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

No existe una única respuesta correcta: **lo importante es justificar vuestra decisión de diseño**.

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

### Personal de tienda

| Rol            | Retención |
| -------------- | --------: |
| `CLERK`        |      10 % |
| `SHOP_MANAGER` |      12 % |

### Personal de IT

| Rol               | Retención |
| ----------------- | --------: |
| `DEVELOPER`       |      15 % |
| `UX_DESIGNER`     |      15 % |
| `PROJECT_MANAGER` |      18 % |

El sistema deberá obtener finalmente el **salario neto**.

---

# 🧠 9. Una decisión importante de diseño

El servicio encargado de procesar las nóminas debe poder trabajar con cualquier tipo de empleado.

Por ejemplo, conceptualmente:

```text
para cada empleado
    procesar su nómina
```

El servicio **no debe preguntar qué tipo concreto de empleado está procesando**.

Por tanto, no se permitirá utilizar:

```java
instanceof
```

ni construir un gran bloque de:

```java
if / else if / else
```

para decidir qué tipo de empleado es.

El comportamiento deberá resolverse mediante **polimorfismo**.

### 💭 Para debatir

> ¿Dónde debería vivir la lógica específica del cálculo de la nómina?

> ¿Qué método común podría definir `Employee`?

> ¿Cómo conseguiríamos que cada tipo de empleado realizase su propio cálculo?

---

# 🔄 10. El reto del cambio

Imaginad que dentro de unos meses AadTex crea una nueva división:

### 🚚 Logística

Aparecen nuevos empleados con:

* Sus propios roles.
* Complementos específicos.
* Reglas salariales diferentes.
* Sus propias características.

El código que actualmente procesa las nóminas **no debería necesitar modificaciones** para incorporar esta nueva categoría.

El objetivo es que podamos incorporar el nuevo comportamiento creando las clases necesarias en el dominio.

### 🎯 Test del minuto

Pregúntate:

> **Si mañana aparece un nuevo tipo de empleado, ¿cuántos lugares de mi programa tendría que modificar?**

Cuanto menor sea ese número, mejor será vuestro diseño.

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

# 🔒 13. Requisitos técnicos

La solución deberá cumplir estas condiciones:

* Java moderno.
* POO correctamente aplicada.
* Herencia cuando esté justificada.
* Polimorfismo.
* Encapsulación.
* Uso adecuado de `enum`.
* Colecciones adecuadas al problema.
* Sin `instanceof` para resolver el procesamiento de nóminas.
* Dependencias inyectadas por constructor.
* Dependencias gestionadas mediante atributos `final`.
* Uso de Lombok cuando resulte apropiado.
* Repositorio separado de la lógica de negocio.
* `ConcurrentHashMap` para el almacenamiento en memoria.
* Aplicación Spring Boot de consola.
* `CommandLineRunner` para ejecutar el escenario inicial.
* `@Slf4j` para los mensajes de ejecución.

---

# 🧪 14. Comprobación final

Antes de dar el reto por terminado, comprobad:

### Modelo

* [ ] El modelo UML representa correctamente las entidades.
* [ ] Las relaciones tienen multiplicidades coherentes.
* [ ] La herencia está justificada.
* [ ] Las responsabilidades están correctamente distribuidas.

### POO

* [ ] No existen bloques de `instanceof` para decidir el comportamiento.
* [ ] Se utiliza polimorfismo.
* [ ] Las clases mantienen una responsabilidad clara.
* [ ] Las colecciones elegidas tienen sentido.

### Aplicación

* [ ] La aplicación arranca correctamente.
* [ ] Los empleados se almacenan en el repositorio.
* [ ] Se pueden recuperar los empleados.
* [ ] Las nóminas se procesan correctamente.
* [ ] Se muestra el salario bruto y neto.
* [ ] Se muestran equipamientos y proyectos.
* [ ] Se muestran las relaciones jerárquicas.

---

# 🚀 15. La pregunta final

Cuando terminéis, pensad en la siguiente situación:

> **La aplicación funciona perfectamente, pero cerramos IntelliJ y volvemos a ejecutarla.**

¿Qué ha ocurrido con todos nuestros empleados?

¿Con los proyectos?

¿Con el equipamiento?

¿Con las relaciones entre empleados?

Todo ha desaparecido.

### ¿Cómo conseguiríamos que los datos sobrevivieran al cierre de la aplicación?

**Esta será precisamente una de las preguntas que comenzaremos a responder en Acceso a Datos.**
