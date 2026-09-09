# RA0: Proyecto de Repaso - AadTex (POO y Arquitectura en Capas)

Este proyecto práctico de repaso sirve como transición entre los fundamentos de la **Programación Orientada a Objetos (POO)** y la persistencia de datos real que abordaremos en las siguientes unidades académicas. Inspirado en sistemas corporativos de Recursos Humanos como el de la multinacional **Inditex** (aquí denominado en clave **AadTex**), el sistema modela de forma simplificada la gestión de empleados y nóminas pertenecientes a las áreas de **Tiendas (Shop)** e **Informática (IT)**.

A través de esta práctica, repasaremos la herencia, el polimorfismo, la inyección de dependencias por constructor y la arquitectura de software en capas sin añadir complejidad de frameworks o serializadores web.

---

## Índice
1. [Estructura del Modelo de Dominio (POO Polimórfica)](#1-estructura-del-modelo-de-dominio-poo-polimorfica)
2. [Implementación de Capas Clean (Repository y Service)](#2-implementación-de-capas-clean-repository-y-service)
3. [Ejecución Polimórfica en Consola](#3-ejecución-polimórfica-en-consola)
4. [Enlace Pedagógico hacia las bases de datos (RA2 y RA3)](#4-enlace-pedagógico-hacia-las-bases-de-datos-ra2-y-ra3)

---

## 1. Estructura del Modelo de Dominio (POO Polimórfica)

Para modelar la lógica de negocio de AadTex, implementaremos una clase base abstracta `Employee`, dos subclases específicas y su asociación con un centro de trabajo (`WorkCenter`). 

Aplicaremos el **Polimorfismo** puro: tanto el método de obtención del salario bruto (`calculateGrossSalary()`) como el método de cálculo final de la nómina (`processPayroll()`) serán sobreescritos por cada tipo concreto de empleado para resolver sus reglas particulares de negocio.

Al no requerir endpoints HTTP ni exportaciones a archivos, **eliminamos por completo Jackson** de nuestras clases, manteniendo el dominio 100% limpio y centrado en Java estándar.

### 1.1 Objeto de Asociación: `WorkCenter.java`
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

### 1.2 Clase Base Polimórfica: `Employee.java`
Clase abstracta pura de Java. Hacemos uso de `@SuperBuilder` de Lombok para heredar el patrón Builder en las clases hijas.

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

    /**
     * Método polimórfico abstracto para calcular el salario bruto del empleado.
     */
    public abstract double calculateGrossSalary();

    /**
     * Método polimórfico abstracto para procesar la nómina completa del empleado,
     * calculando su retención de impuestos (IRPF) y su salario neto final.
     */
    public abstract void processPayroll();
}
```

### 1.3 Subclases Polimórficas (Especialización de Comportamientos)

#### A) Empleado de Informática: `EmployeeIT.java`
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
        
        // Retención fiscal del 18% para el área de Informática (IT)
        double withholding = gross * 0.18;
        setNetSalary(gross - withholding);
    }
}
```

#### B) Empleado de Tienda: `EmployeeShop.java`
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
        
        // Retención fiscal reducida del 12% para el área de Tiendas (Shop)
        double withholding = gross * 0.12;
        setNetSalary(gross - withholding);
    }
}
```

---

## 2. Implementación de Capas Clean (Repository y Service)

Para acoplarnos a las buenas prácticas de desarrollo en Spring, utilizaremos **Inyección de Dependencias por Constructor** marcada como `final` apoyándonos en la anotación de Lombok `@RequiredArgsConstructor`. Esto evita el uso de la anotación `@Autowired` en campos (field injection), facilitando las pruebas unitarias y garantizando la inmutabilidad de nuestras dependencias.

### 2.1 Capa Repository (Simulada en Memoria): `EmployeeRepository.java`
```java
package com.fencgut961.aad.repository;

import com.fencgut961.aad.model.Employee;
import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.concurrent.ConcurrentHashMap;

@Repository
public class EmployeeRepository {

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

    public Optional<Employee> findById(Long id) {
        return Optional.ofNullable(memoryDatabase.get(id));
    }
}
```

### 2.2 Capa Service (Lógica de Negocio): `EmployeeService.java`
Observa el uso de `@RequiredArgsConstructor` para inyectar automáticamente el repositorio de forma obligatoria por constructor.

```java
package com.fencgut961.aad.service;

import com.fencgut961.aad.model.Employee;
import com.fencgut961.aad.repository.EmployeeRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
@RequiredArgsConstructor // Genera automáticamente el constructor para los campos marcados como final
@Slf4j
public class EmployeeService {

    private final EmployeeRepository repository;

    public Employee registerEmployee(Employee employee) {
        // Invocación polimórfica pura del procesamiento de nómina
        employee.processPayroll();
        
        log.info("Processed payroll for {}. Gross: {}€ | Net: {}€", 
                 employee.getName(), employee.getGrossSalary(), employee.getNetSalary());
                 
        return repository.save(employee);
    }

    public List<Employee> getAllEmployees() {
        return repository.findAll();
    }
}
```

---

## 3. Ejecución Polimórfica en Consola

Para validar todo nuestro flujo e inyección de dependencias de forma directa y simple al arrancar el microservicio, utilizaremos la clase principal de Spring Boot implementando `CommandLineRunner`:

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
@RequiredArgsConstructor // Inyección por constructor del EmployeeService
@Slf4j
public class AadApplication implements CommandLineRunner {

    private final EmployeeService service;

    public static void main(String[] args) {
        SpringApplication.run(AadApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        log.info("--- Initializing Seed Data ---");

        // 1. Crear Centros de Trabajo
        WorkCenter centralArteixo = WorkCenter.builder().id(1L).name("Arteixo Central IT").city("A Coruña").build();
        WorkCenter solShop = WorkCenter.builder().id(2L).name("Zara Puerta del Sol").city("Madrid").build();

        // 2. Crear instancias polimórficas
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

        // 3. Registrar empleados (provoca la llamada polimórfica en cascada)
        service.registerEmployee(sofia);
        service.registerEmployee(pedro);

        log.info("--- Executing Business Logic Verification ---");
        for (Employee emp : service.getAllEmployees()) {
            log.info("Employee: {} [Type: {}] | Work Center: {} | Net Salary: {}€", 
                     emp.getName(), 
                     emp.getClass().getSimpleName(), 
                     emp.getWorkCenter().getName(), 
                     emp.getNetSalary());
        }

        log.info("--- Review Unit 0 Completed Successfully ---");
    }
}
```

---

## 4. Enlace Pedagógico hacia las bases de datos (RA2 y RA3)

Esta arquitectura de capas limpia en memoria sienta las bases perfectas para los retos de persistencia reales que abordaremos en las siguientes unidades:

*   **Reto 1: Mapeo de Herencia Relacional (RA2)**: 
    ¿Cómo guardamos en una base de datos PostgreSQL de Docker a nuestros empleados si en SQL no existen las clases ni la herencia? Estudiaremos las tres estrategias de Hibernate/JPA:
    1.  *Single Table*: Una única tabla `employees` que junta todas las columnas (con muchos nulos para las columnas exclusivas).
    2.  *Joined Table*: Una tabla base `employees` unida por claves primarias a `employees_it` y `employees_shop`.
    3.  *Table Per Class*: Tablas físicas totalmente independientes para cada tipo.
*   **Reto 2: Mapeo de Relaciones de Asociación (RA3)**:
    ¿Cómo representamos en SQL que un empleado pertenece a un `WorkCenter`? Introducción práctica a las claves foráneas y relaciones relacionales `ManyToOne` (Muchos Empleados a Un Centro de Trabajo).
