# 🏫 Taller Interactivo en el Aula: El Reto de Diseño "AadTex" (UD0)

Este documento está diseñado como una **guía del profesor** para conducir una sesión interactiva de desarrollo en la pizarra. El objetivo es que los alumnos **diseñen y piensen la solución antes de tocar una sola línea de código**, evitando el vicio de pedirle código directamente a una Inteligencia Artificial sin comprender la lógica interna de los objetos.

---

## 📌 Fase 1: El Caso AadTex (Requisitos de Negocio)

*Proyectar esta sección en clase o escribir las necesidades básicas en la pizarra.*

La empresa **AadTex** necesita un prototipo rápido de un sistema de personal para gestionar la nómina de sus trabajadores en sus oficinas centrales y tiendas de ropa. 

### Los Datos Clave
*   La empresa dispone de **Centros de Trabajo** (como oficinas tecnológicas o tiendas físicas en el centro de las ciudades).
*   Hay empleados destinados en esos centros. De momento, tenemos dos tipos de empleados:
    1.  **Especialistas en Informática (IT)**: Tienen un salario base y un plus fijo por proyecto tecnológico (`projectBonus`). Su retención de IRPF es del **18%**.
    2.  **Personal de Tienda (Shop)**: Tienen un salario base y una comisión por sus ventas mensuales (`salesBonus`). Su retención de IRPF es reducida, del **12%**.

---

## ✏️ Fase 2: El Diagrama de Clases en la Pizarra (Pensando el Modelo)

*Guía al grupo de alumnos para dibujar el modelo conceptual. Lanza las siguientes preguntas para que ellos propongan la estructura:*

### Preguntas para el debate:
1.  **¿Qué campos comunes comparten todos los empleados?** 
    *(Saldrán: id, name, email, baseSalary, netSalary...)*
2.  **¿Cómo modelamos la especialización?**
    *   ¿Metemos todos los atributos en una sola clase enorme o usamos herencia?
    *   Si usamos herencia, ¿la clase madre `Employee` puede ser instanciada directamente? *(No, debe ser abstracta).*
3.  **La relación con el Centro de Trabajo (`WorkCenter`):**
    *   ¿Un empleado tiene un centro de trabajo o un centro de trabajo tiene una lista de empleados?
    *   *Punto de reflexión*: Para este microservicio básico en memoria, ¿es más fácil que cada `Employee` apunte a su `WorkCenter`? ¿Cómo se representa esa asociación? *(Relación de asociación directa, en SQL sería una Clave Foránea).*

### Dibujo esperado en la pizarra:
```text
  +--------------------------------+
  |           WorkCenter           |
  +--------------------------------+
  | - id: Long                     |
  | - name: String                 |
  | - city: String                 |
  +--------------------------------+
                  ^
                  | (1)
                  |
                  | (N)
  +--------------------------------+
  |       Employee (Abstract)      |
  +--------------------------------+
  | - id: Long                     |
  | - name: String                 |
  | - email: String                |
  | - baseSalary: double           |
  | - netSalary: double            |
  | - workCenter: WorkCenter       |
  +--------------------------------+
  | + calculateGrossSalary(): dbl  | [Abstract]
  | + processPayroll(): void       | [Abstract]
  +--------------------------------+
         ^                  ^
         | (inherits)       | (inherits)
  +--------------------+  +--------------------+
  |     EmployeeIT     |  |    EmployeeShop    |
  +--------------------+  +--------------------+
  | - projectBonus: dbl|  | - salesBonus: dbl  |
  +--------------------+  +--------------------+
  | + gross(): double  |  | + gross(): double  |
  | + payroll(): void  |  | + payroll(): void  |
  +--------------------+  +--------------------+
```

---

## 🧠 Fase 3: El Gran Reto del Polimorfismo

*Antes de codificar, plantea la restricción lógica que obligará a los alumnos a usar POO real en lugar de condicionales procedurales.*

### La regla de oro: "Prohibido preguntar el tipo"
> **"Está terminantemente prohibido usar `instanceof` o cadenas de texto para saber si un empleado es de IT o de Tienda durante el cálculo de nóminas."**

### El debate en la pizarra:
*   Si tenemos una colección genérica `List<Employee> employees`... ¿cómo podemos procesar la nómina de todos en un bucle sin saber de qué tipo es cada uno?
*   *Solución a la que deben llegar*: Al declarar `processPayroll()` en la clase abstracta, el bucle simplemente hace `employee.processPayroll()`. La Máquina Virtual de Java (JVM) se encargará de buscar en tiempo de ejecución si el objeto real es un `EmployeeIT` o un `EmployeeShop` y ejecutará la retención correcta (18% o 12%).

---

## 💻 Fase 4: Implementación del Microservicio (Live Coding)

*Una vez consensuado el diseño en la pizarra, es hora de abrir IntelliJ y picar el código de forma guiada.*

### Paso 1: El Modelo de Dominio
Cread las clases de datos básicas utilizando **Lombok** para ahorrar código repetitivo de Getters, Setters y constructores.

#### Clase `WorkCenter.java`
```java
package com.fencgut961.aad.model;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class WorkCenter {
    private Long id;
    private String name;
    private String city;
}
```

#### Clase Base `Employee.java` (Abstracta)
*Nota didáctica: Al usar herencia en Lombok, es fundamental utilizar la anotación `@SuperBuilder` en lugar de `@Builder` para permitir heredar la construcción de campos de forma fluida.*
```java
package com.fencgut961.aad.model;

import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.NoArgsConstructor;
import lombok.experimental.SuperBuilder;

@Data
@NoArgsConstructor
@SuperBuilder
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public abstract class Employee {

    @EqualsAndHashCode.Include
    private Long id;
    private String name;
    private String email;
    private double baseSalary;
    private double grossSalary;
    private double netSalary;
    
    private WorkCenter workCenter;

    // Métodos polimórficos obligatorios
    public abstract double calculateGrossSalary();
    public abstract void processPayroll();
}
```

#### Subclase `EmployeeIT.java`
```java
package com.fencgut961.aad.model;

import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.NoArgsConstructor;
import lombok.experimental.SuperBuilder;

@Data
@NoArgsConstructor
@SuperBuilder
@EqualsAndHashCode(callSuper = true)
public class EmployeeIT extends Employee {

    private double projectBonus;

    @Override
    public double calculateGrossSalary() {
        return getBaseSalary() + projectBonus;
    }

    @Override
    public void processPayroll() {
        double gross = calculateGrossSalary();
        setGrossSalary(gross);
        // Retención fija del 18% para IT
        setNetSalary(gross * 0.82); 
    }
}
```

#### Subclase `EmployeeShop.java`
```java
package com.fencgut961.aad.model;

import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.NoArgsConstructor;
import lombok.experimental.SuperBuilder;

@Data
@NoArgsConstructor
@SuperBuilder
@EqualsAndHashCode(callSuper = true)
public class EmployeeShop extends Employee {

    private double salesBonus;

    @Override
    public double calculateGrossSalary() {
        return getBaseSalary() + salesBonus;
    }

    @Override
    public void processPayroll() {
        double gross = calculateGrossSalary();
        setGrossSalary(gross);
        // Retención reducida del 12% para Tienda
        setNetSalary(gross * 0.88); 
    }
}
```

---

### Paso 2: Las Capas de Arquitectura (Repository & Service)

#### Repositorio Simulado: `EmployeeRepository.java`
*¿Cómo almacenamos los datos sin base de datos? Pensar en una colección concurrente para simular el almacenamiento.*
```java
package com.fencgut961.aad.repository;

import com.fencgut961.aad.model.Employee;
import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

@Repository
public class EmployeeRepository {

    // Simulación de persistencia en memoria
    private final Map<Long, Employee> memoryDatabase = new ConcurrentHashMap<>();

    public Employee save(Employee employee) {
        if (employee.getId() == null) {
            long newId = memoryDatabase.size() + 1L;
            employee.setId(newId);
        }
        memoryDatabase.put(employee.getId(), employee);
        return employee;
    }

    public List<Employee> findAll() {
        return new ArrayList<>(memoryDatabase.values());
    }
}
```

#### Capa de Negocio: `EmployeeService.java`
*Aquí aplicaremos la **Inyección de Dependencias por Constructor**. Al usar Lombok `@RequiredArgsConstructor`, se autogenerará el constructor para todos los campos marcados como `private final`.*
```java
package com.fencgut961.aad.service;

import com.fencgut961.aad.model.Employee;
import com.fencgut961.aad.repository.EmployeeRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
@RequiredArgsConstructor // Inyección automática por constructor limpia de Spring
@Slf4j
public class EmployeeService {

    private final EmployeeRepository repository;

    public Employee registerEmployee(Employee employee) {
        // Llamada polimórfica: la JVM decide qué lógica de nómina ejecutar
        employee.processPayroll();
        
        log.info("Registered: {} | Gross: {}€ | Net: {}€", 
                 employee.getName(), employee.getGrossSalary(), employee.getNetSalary());
                 
        return repository.save(employee);
    }

    public List<Employee> getAllEmployees() {
        return repository.findAll();
    }
}
```

---

### Paso 3: Inicialización de Prueba (`AadApplication.java`)
Pídele a los alumnos que inyecten el servicio por constructor e implementen `CommandLineRunner` para simular la ejecución y verificar el resultado en la consola al arrancar la aplicación de Spring Boot.

```java
package com.fencgut961.aad;

import com.fencgut961.aad.model.*;
import com.fencgut961.aad.service.EmployeeService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@RequiredArgsConstructor
@Slf4j
public class AadApplication implements CommandLineRunner {

    private final EmployeeService service;

    public static void main(String[] args) {
        SpringApplication.run(AadApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        log.info("--- Levantando el Prototipo AadTex ---");

        // 1. Instanciación de Centros de Trabajo
        WorkCenter centralArteixo = WorkCenter.builder().id(1L).name("Arteixo IT").city("A Coruña").build();
        WorkCenter solShop = WorkCenter.builder().id(2L).name("Zara Puerta del Sol").city("Madrid").build();

        // 2. Creación de perfiles
        Employee sofia = EmployeeIT.builder()
                .name("Sofia Martinez")
                .email("sofia.martinez@aadtex.com")
                .baseSalary(3000.0)
                .workCenter(centralArteixo)
                .projectBonus(500.0)
                .build();

        Employee pedro = EmployeeShop.builder()
                .name("Pedro Almodovar")
                .email("pedro.almodovar@aadtex.com")
                .baseSalary(1200.0)
                .workCenter(solShop)
                .salesBonus(300.0)
                .build();

        // 3. Registro y ejecución del polimorfismo
        service.registerEmployee(sofia);
        service.registerEmployee(pedro);

        log.info("--- Ejecución del Prototipo Finalizada ---");
    }
}
```
