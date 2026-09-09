# 🚀 Reto Técnico de Programación: El Sistema "AadTex"

## 🏢 Contexto del Caso de Estudio
La multinacional textil **AadTex** (perteneciente al sector de la moda retail) necesita un prototipo rápido en consola para gestionar de manera unificada las nóminas de su personal de oficinas centrales y de tiendas. 

Como ingenieros de software de su departamento de sistemas, se os ha encomendado diseñar un microservicio base utilizando **Spring Boot** que repase los principios más importantes de la **Programación Orientada a Objetos (POO)** y la **Arquitectura Limpia**, preparando el terreno para la futura migración a bases de datos relacionales.

---

## 🎯 Objetivos de Aprendizaje
1. **Modelar jerarquías de herencia** correctas para evitar la duplicación de datos.
2. **Aplicar polimorfismo puro** en la resolución de reglas de negocio dinámicas en tiempo de ejecución.
3. **Implementar una arquitectura limpia en capas** (Controller-Service-Repository) acoplada mediante **Inyección de Dependencias por Constructor**.

---

## 📋 Especificaciones del Modelo de Dominio

Debéis diseñar un modelo de objetos en inglés que represente la siguiente estructura de negocio:

### 1. El Centro de Trabajo (`WorkCenter`)
Representa los lugares físicos donde operan los empleados (por ejemplo, las oficinas centrales o las tiendas de calle). Debe tener:
* Un identificador único numérico.
* Un nombre descriptivo.
* La ciudad donde está ubicado.

### 2. El Empleado (`Employee` - Clase Abstracta)
Es la entidad base de la que parten todos los trabajadores de la compañía. Almacena las propiedades comunes:
* Identificador único numérico, nombre completo, correo electrónico, salario base (`baseSalary`) y salario neto final calculado (`netSalary`).
* La asociación con el centro de trabajo (`WorkCenter`) al que pertenece.
* **Métodos abstractos obligatorios (Polimorfismo)**:
  * `calculateGrossSalary()`: Debe devolver el salario bruto calculado.
  * `processPayroll()`: Debe procesar la nómina completa del empleado (calcular su salario bruto, restarle las deducciones fiscales correspondientes de IRPF y almacenar el salario neto final).

### 3. Especializaciones de Empleados (Subclases)
La empresa cuenta actualmente con dos tipos de empleados con lógicas de nómina totalmente distintas:

* **Empleado de Informática (`EmployeeIT`)**:
  * Añade un atributo exclusivo: un plus fijo por proyecto tecnológico (`projectBonus`).
  * *Cálculo de Bruto*: Es la suma de su salario base más el plus de proyecto.
  * *Deducción Fiscal*: Al pertenecer a un sector con salarios altos, se le aplica una retención de IRPF fija del **18%** para calcular su salario neto.
* **Empleado de Tienda (`EmployeeShop`)**:
  * Añade un atributo exclusivo: una comisión o plus por ventas comerciales mensuales (`salesBonus`).
  * *Cálculo de Bruto*: Es la suma de su salario base más el plus por ventas.
  * *Deducción Fiscal*: Para incentivar al sector de tiendas, disfrutan de una retención reducida del **12%** para calcular su neto.

---

## 🏗️ Requisitos de Arquitectura y Flujo

El sistema debe estructurarse siguiendo un diseño limpio y modular en tres capas diferenciadas:

1. **Capa de Datos (`EmployeeRepository`)**:
  * No utilizaremos bases de datos reales en este repaso.
  * La persistencia se simulará en memoria utilizando una **colección concurrente** (adecuada para entornos multi-hilo) para almacenar y recuperar los empleados.
  * Debe incluir métodos para guardar un empleado (generando un identificador incremental automático si es nuevo) y recuperar todos los registrados.
2. **Capa de Negocio (`EmployeeService`)**:
  * Debe contener la lógica para registrar un nuevo empleado.
  * **Punto Crítico Polimórfico**: Al registrar al empleado, el servicio debe invocar su procesamiento de nómina. La máquina virtual de Java deberá decidir dinámicamente en tiempo de ejecución si aplica las reglas fiscales de informática (18%) o de tienda (12%).
  * Debe utilizar **Inyección de Dependencias por Constructor** (con ayuda de Lombok) para recibir su repositorio inmutable.
3. **Capa de Inicialización (`CommandLineRunner`)**:
  * Al levantar la aplicación Spring Boot, el sistema debe precargar automáticamente dos perfiles de prueba en inglés:
    * Un empleado IT en el centro *"Arteixo Central IT"* con su correspondiente plus de proyecto.
    * Un empleado de tienda en *"Zara Puerta del Sol"* con su comisión de ventas.
  * El programa debe mostrar por la consola de IntelliJ el resultado de procesar polimórficamente a ambos empleados, mostrando el desglose de su bruto y neto final.

---
