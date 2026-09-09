# 🎯 Solución Óptima de Referencia Completa (v2): Sistema AadTex (UD0 - Repaso POO)

Esta es la solución de referencia definitiva (versión 2) diseñada para el docente, alineada al 100% con el enunciado oficial **`ra0-enunciado-alumnos.md`**. Implementa con total precisión y de manera simétrica las **5 relaciones del paradigma de orientación a objetos en memoria**, cumpliendo rigurosamente con las reglas de diseño arquitectónico y de buenas prácticas.

---

## 1. Diseño de Arquitectura y Relaciones (UML Conceptual)

En la pizarra con los alumnos, el diseño se estructura mediante la siguiente jerarquía de clases y mapa de asociaciones, mapeando de forma explícita las 5 relaciones:

```text
                  +--------------------------+
                  |        WorkCenter        |
                  +--------------------------+
                  | - id: Long               |
                  | - name: String           |
                  | - city: String           |
                  +--------------------------+
                               ^
                               | 1
                               | 
                               | *
+------------------------------------------------------------------+
|                        Employee (Abstract)                       |
+------------------------------------------------------------------+
| - id: Long                                                       |
| - name: String                                                   |
| - email: String                                                  |
| - baseSalary: double                                             |
| - netSalary: double                                              |
| - workCenter: WorkCenter (RELACIÓN 2 - Asociación Simple 1:N)    |
| - supervisor: Employee (RELACIÓN 3 - Autorreferencia 1:1)        |
| - equipments: List<Equipment> (RELACIÓN 4 - Asociación 1:N)      |
+------------------------------------------------------------------+
| + calculateGrossSalary(): double {abstract}                      |
| + processPayroll(): void {abstract}                              |
| + addEquipment(equipment: Equipment): void                       |
+------------------------------------------------------------------+
         ^                                                  ^
         | [RELACIÓN 1 - Herencia]                          | [RELACIÓN 1 - Herencia]
  +------+-----+                                     +------+-----+
  | EmployeeIT |                                     |EmployeeShop|
  +------------+                                     +------------+
  | - role: ITRole (Enum)                            | - role: ShopRole (Enum)
  | - projectBonus: double                           | - salesBonus: double
  | - remote: boolean                                |            |
  | - projects: Set<Project> (RELACIÓN 5 - N:M Set)  |            |
  +------------+                                     +------------+
```

### 💡 Análisis del Mapa de Relaciones:
1. **RELACIÓN 1 (Herencia y Especialización)**: Clase madre abstracta `Employee` con especialización en clases hijas `EmployeeShop` (para Retail de tienda) y `EmployeeIT` (para tecnología).
2. **RELACIÓN 2 (Asociación Simple `1:N`)**: Cada empleado pertenece a un único centro de trabajo (`WorkCenter`). Para aquellos perfiles de IT en teletrabajo (`remote = true`), se les asocia conceptualmente al centro virtual *"Virtual Remote Hub"*.
3. **RELACIÓN 3 (Asociación Autorreferencial Unidireccional `1:1`)**: El campo `supervisor` de tipo `Employee` permite tejer la jerarquía de reporte organizativa recursivamente sin duplicidad de clases.
4. **RELACIÓN 4 (Asociación de Equipamientos `1:N`)**: Cada empleado gestiona una colección `List<Equipment>` para albergar múltiples dispositivos o recursos asignados de manera flexible.
5. **RELACIÓN 5 (Relación Muchos a Muchos `N:M`)**: Los técnicos de IT colaboran en varios proyectos corporativos simultáneamente (`Set<Project>`). Se utiliza un conjunto `Set` para garantizar la unicidad de las asignaciones y evitar proyectos duplicados en memoria.

---

## 2. Elementos de Clasificación: Enums de Dominio

Ubicados en el paquete `com.fencgut961.aad.model` para garantizar tipificación fuerte:

### `ShopRole.java`
```java
package com.fencgut961.aad.model;

public enum ShopRole {
    CLERK,
    SHOP_MANAGER
}
```

### `ITRole.java`
```java
package com.fencgut961.aad.model;

public enum ITRole {
    DEVELOPER,
    UX_DESIGNER,
    PROJECT_MANAGER
}
```

### `EquipmentType.java`
```java
package com.fencgut961.aad.model;

public enum EquipmentType {
    PC_PORTATIL,
    MONITOR,
    SILLA_ERGONOMICA,
    MOVIL_CORPORATIVO,
    COCHE_EMPRESA,
    ROPA_TRABAJO
}
```

---

## 3. Entidades de Soporte y Asociaciones

Ubicadas en el paquete `com.fencgut961.aad.model`:

### `WorkCenter.java`
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

### `Project.java`
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
public class Project {
    private Long id;
    private String name;
    private String technology;
}
```

### `Equipment.java`
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
public class Equipment {
    private Long id;
    private String serialNumber;
    private String model;
    private EquipmentType type; // Tipo de equipamiento restringido por el Enum
}
```

---

## 4. Jerarquía de Clases y Polimorfismo

Ubicadas en el paquete `com.fencgut961.aad.model` para modelar de forma precisa el comportamiento del personal:

### `Employee.java` (Clase Madre Abstracta - RELACIÓN 1)
```java
package com.fencgut961.aad.model;

import lombok.Builder;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.NoArgsConstructor;
import lombok.experimental.SuperBuilder;

import java.util.ArrayList;
import java.util.List;

@Data
@NoArgsConstructor
@SuperBuilder
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public abstract class Employee { // RELACIÓN 1: Herencia y Especialización (Madre)

    @EqualsAndHashCode.Include
    private Long id;
    private String name;
    private String email;
    private double baseSalary;
    private double netSalary; // Salario neto tras procesar nómina

    // RELACIÓN 2: Asociación Simple con Centros de Trabajo (1:N)
    private WorkCenter workCenter;

    // RELACIÓN 3: Asociación Autorreferencial Unidireccional de Reporte (1:1)
    private Employee supervisor;

    // RELACIÓN 4: Asociación de Equipamientos Corporativos (1:N)
    @Builder.Default
    private List<Equipment> equipments = new ArrayList<>();

    public void addEquipment(Equipment equipment) {
        this.equipments.add(equipment);
    }

    /**
     * Fuerza polimórficamente a las subclases a calcular su propio salario bruto
     * según los pluses y variables exclusivas de su área.
     */
    public abstract double calculateGrossSalary();

    /**
     * Fuerza polimórficamente a las subclases a procesar su propia nómina
     * aplicando el IRPF correspondiente según su rol y deduciendo el neto.
     */
    public abstract void processPayroll();
}
```

### `EmployeeShop.java` (Subclase de Retail - RELACIÓN 1)
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
public class EmployeeShop extends Employee { // RELACIÓN 1: Herencia y Especialización (Hijo Shop)

    private ShopRole role;
    private double salesBonus;

    @Override
    public double calculateGrossSalary() {
        // Bruto = Salario Base + Comisiones por venta de tienda
        return getBaseSalary() + salesBonus;
    }

    @Override
    public void processPayroll() {
        double gross = calculateGrossSalary();
        double irpfRate = 0.0;

        // Regla de Oro 2: El IRPF se deduce según su rol en Tienda
        if (role == ShopRole.SHOP_MANAGER) {
            irpfRate = 12.0; // 12% IRPF para encargados
        } else {
            irpfRate = 10.0; // 10% IRPF para dependientes (CLERK)
        }

        double taxWithholding = gross * (irpfRate / 100.0);
        setNetSalary(gross - taxWithholding);
    }
}
```

### `EmployeeIT.java` (Subclase de Tecnología - RELACIÓN 1 y RELACIÓN 5)
```java
package com.fencgut961.aad.model;

import lombok.Builder;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.NoArgsConstructor;
import lombok.experimental.SuperBuilder;

import java.util.HashSet;
import java.util.Set;

@Data
@NoArgsConstructor
@SuperBuilder
@EqualsAndHashCode(callSuper = true)
public class EmployeeIT extends Employee { // RELACIÓN 1: Herencia y Especialización (Hijo IT)

    private ITRole role;
    private double projectBonus;
    private boolean remote;

    // RELACIÓN 5: Relación Muchos a Muchos en Tecnología (N:M)
    @Builder.Default
    private Set<Project> projects = new HashSet<>();

    public void addProject(Project project) {
        this.projects.add(project);
    }

    @Override
    public double calculateGrossSalary() {
        // Bruto = Salario Base + Plus por criticidad de proyecto
        return getBaseSalary() + projectBonus;
    }

    @Override
    public void processPayroll() {
        double gross = calculateGrossSalary();
        double irpfRate = 0.0;

        // Regla de Oro 2: El IRPF se deduce según su rol tecnológico de IT
        if (role == ITRole.PROJECT_MANAGER) {
            irpfRate = 18.0; // 18% IRPF para jefes de proyecto
        } else {
            irpfRate = 15.0; // 15% IRPF para DEVELOPER y UX_DESIGNER
        }

        double taxWithholding = gross * (irpfRate / 100.0);
        setNetSalary(gross - taxWithholding);
    }
}
```

---

## 5. Capas de la Arquitectura (Spring Core Inmutable)

### `EmployeeRepository.java` (Persistencia Simulada en Memoria)
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

    // Almacenamiento seguro multihilo para simular la persistencia en memoria
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

### `EmployeeService.java` (Lógica de Negocio Desacoplada)
```java
package com.fencgut961.aad.service;

import com.fencgut961.aad.model.Employee;
import com.fencgut961.aad.repository.EmployeeRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
@RequiredArgsConstructor // Inyecta campos final por constructor de forma inmutable (Evita NullPointerException)
@Slf4j
public class EmployeeService {

    private final EmployeeRepository repository;

    public Employee registerEmployee(Employee employee) {
        // Ejecución del cálculo polimórfico en el dominio (Cero instanceof)
        employee.processPayroll();
        return repository.save(employee);
    }

    /**
     * Motor de procesamiento masivo de nóminas (Cumple SOLID Open-Closed Principle).
     * Si mañana se añade "Logística", este bucle de nóminas no cambia en absoluto.
     */
    public void runPayrollProcessing() {
        log.info(">>> Iniciando motor masivo de nóminas polimórficas (AadTex) <<<");
        List<Employee> employees = repository.findAll();
        for (Employee emp : employees) {
            emp.processPayroll(); // Resolución dinámica en tiempo de ejecución por la JVM (Cero instanceof)
            repository.save(emp);
        }
        log.info(">>> Proceso finalizado para {} empleados <<<", employees.size());
    }

    public List<Employee> getAllEmployees() {
        return repository.findAll();
    }
}
```

---

## 6. Orquestador de Arranque y Simulación: `AadApplication.java`

Esta clase de Spring Boot cablea todo el modelo lógico en memoria. Simula un escenario representativo de prueba cubriendo todos los roles y las **cinco tipologías de relaciones** de forma exacta, lanzando el motor masivo e imprimiendo el resultado en consola de forma limpia utilizando Lombok `@Slf4j`.

```java
package com.fencgut961.aad;

import com.fencgut961.aad.model.*;
import com.fencgut961.aad.service.EmployeeService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.util.ArrayList;
import java.util.List;
import java.util.Set;
import java.util.stream.Collectors;

@SpringBootApplication
@RequiredArgsConstructor // Autocableado seguro por constructor libre de @Autowired
@Slf4j
public class AadApplication implements CommandLineRunner {

    private final EmployeeService service;

    public static void main(String[] args) {
        SpringApplication.run(AadApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        log.info("================================================================================");
        log.info("=== INICIANDO SIMULACIÓN MAESTRA DE PROGRAMACIÓN ORIENTADA A OBJETOS ===");
        log.info("================================================================================");

        // 1. Instanciación de Centros de Trabajo (Asociación Simple 1:N - RELACIÓN 2)
        WorkCenter centralArteixo = WorkCenter.builder().id(1L).name("Arteixo Central Office").city("A Coruña").build();
        WorkCenter solStore = WorkCenter.builder().id(2L).name("Zara Puerta del Sol").city("Madrid").build();
        WorkCenter virtualHub = WorkCenter.builder().id(3L).name("Virtual Remote Hub").city("Remote-Worldwide").build();

        // 2. Instanciación de Proyectos (Relación Muchos a Muchos N:M - RELACIÓN 5)
        Project rfidSystem = Project.builder().id(101L).name("RFID Logistics Tracking").technology("Java & RFID IoT").build();
        Project zaraApp = Project.builder().id(102L).name("Zara Transaccional App").technology("Kotlin & Spring").build();
        Project paymentsGate = Project.builder().id(103L).name("Global Payments Gateway").technology("Java & Security").build();

        // 3. Creación de Equipamientos Corporativos (Asociación 1:N - RELACIÓN 4)
        Equipment pcSofia = Equipment.builder().id(201L).serialNumber("SN-8822-LP").model("MacBook Pro 16 M3").type(EquipmentType.PC_PORTATIL).build();
        Equipment monitorSofia = Equipment.builder().id(202L).serialNumber("SN-5561-MN").model("LG UltraWide 34").type(EquipmentType.MONITOR).build();
        Equipment chairSofia = Equipment.builder().id(203L).serialNumber("SN-9988-CH").model("Steelcase Gesture").type(EquipmentType.SILLA_ERGONOMICA).build();
        
        Equipment uniformPedro = Equipment.builder().id(204L).serialNumber("SN-1100-UN").model("Uniforme Completo Zara").type(EquipmentType.ROPA_TRABAJO).build();
        Equipment pdaPedro = Equipment.builder().id(205L).serialNumber("SN-2244-PD").model("Zebra TC52 PDA").type(EquipmentType.MOVIL_CORPORATIVO).build();

        Equipment pcMarcus = Equipment.builder().id(206L).serialNumber("SN-7761-LP").model("ThinkPad P16 Gen 2").type(EquipmentType.PC_PORTATIL).build();
        Equipment carMarcus = Equipment.builder().id(207L).serialNumber("SN-0021-CAR").model("Tesla Model 3").type(EquipmentType.COCHE_EMPRESA).build();

        Equipment phoneClara = Equipment.builder().id(208L).serialNumber("SN-3321-PH").model("iPhone 15 Enterprise").type(EquipmentType.MOVIL_CORPORATIVO).build();

        // 4. Estructura Jerárquica y Registro de Empleados (Con SuperBuilders - RELACIÓN 1)

        // A) Jefe/Supervisor de la Tienda de Puerta del Sol (Shop Manager)
        EmployeeShop managerSol = EmployeeShop.builder()
                .id(1L)
                .name("Clara Oswald")
                .email("clara.oswald@aadtex.com")
                .baseSalary(2100.0)
                .workCenter(solStore) // RELACIÓN 2: Asociación Simple
                .role(ShopRole.SHOP_MANAGER)
                .salesBonus(450.0)
                .equipments(new ArrayList<>(List.of(phoneClara))) // RELACIÓN 4: Equipamientos (1 equipo)
                .build();
        service.registerEmployee(managerSol);

        // B) Dependiente (Clerk) asignado a la tienda de Sol y bajo la supervisión de Clara
        EmployeeShop clerkSol = EmployeeShop.builder()
                .id(2L)
                .name("Pedro Almodovar")
                .email("pedro.almodovar@aadtex.com")
                .baseSalary(1200.0)
                .workCenter(solStore) // RELACIÓN 2: Asociación Simple
                .role(ShopRole.CLERK)
                .salesBonus(150.0)
                .supervisor(managerSol) // RELACIÓN 3: Asociación Autorreferencial (Supervisor directo)
                .equipments(new ArrayList<>(List.of(uniformPedro, pdaPedro))) // RELACIÓN 4: Equipamientos (múltiples)
                .build();
        service.registerEmployee(clerkSol);

        // C) Jefe de Proyecto de IT (Project Manager) ubicado físicamente en la oficina central de Arteixo
        EmployeeIT pmIT = EmployeeIT.builder()
                .id(3L)
                .name("Marcus Cole")
                .email("marcus.cole@aadtex.com")
                .baseSalary(3800.0)
                .workCenter(centralArteixo) // RELACIÓN 2: Asociación Simple
                .role(ITRole.PROJECT_MANAGER)
                .projectBonus(600.0)
                .remote(false)
                .projects(Set.of(zaraApp, paymentsGate)) // RELACIÓN 5: Muchos a Muchos (N:M)
                .equipments(new ArrayList<>(List.of(pcMarcus, carMarcus))) // RELACIÓN 4: Equipamientos (múltiples)
                .build();
        service.registerEmployee(pmIT);

        // D) Desarrolladora (Developer) en remoto asignada a proyectos cruzados y bajo la supervisión de Marcus
        EmployeeIT devRemote = EmployeeIT.builder()
                .id(4L)
                .name("Sofia Martinez")
                .email("sofia.martinez@aadtex.com")
                .baseSalary(2900.0)
                .workCenter(virtualHub) // RELACIÓN 2: Asociación Simple al hub virtual de teletrabajo
                .role(ITRole.DEVELOPER)
                .projectBonus(300.0)
                .remote(true)
                .supervisor(pmIT) // RELACIÓN 3: Asociación Autorreferencial (Supervisor directo)
                .projects(Set.of(rfidSystem, zaraApp)) // RELACIÓN 5: Muchos a Muchos (Coincide en proyecto con Marcus)
                .equipments(new ArrayList<>(List.of(pcSofia, monitorSofia, chairSofia))) // RELACIÓN 4: Equipamientos (múltiples)
                .build();
        service.registerEmployee(devRemote);

        // 5. Demostración del Procesamiento Masivo del Servicio (Regla de Oro 3 - Open Closed)
        service.runPayrollProcessing();

        // 6. Impresión polimórfica detallada por consola de la verificación de datos
        log.info("=== RESULTADO DETALLADO DEL ESCENARIO DE PRUEBAS EN MEMORIA ===");
        
        service.getAllEmployees().forEach(emp -> {
            // Recorremos los equipos asociados del empleado mediante Streams
            String equipmentList = emp.getEquipments().stream()
                    .map(eq -> eq.getModel() + " (" + eq.getType() + ")")
                    .collect(Collectors.joining(", "));

            log.info("--------------------------------------------------------------------------------");
            log.info("Empleado: {} | Base: {} EUR | Neto Calculado: {} EUR", emp.getName(), emp.getBaseSalary(), emp.getNetSalary());
            log.info("  -> Ubicado en: {} ({})", emp.getWorkCenter().getName(), emp.getWorkCenter().getCity());
            log.info("  -> Supervisor Directo: {}", emp.getSupervisor() != null ? emp.getSupervisor().getName() : "Ninguno (Director General)");
            log.info("  -> Equipamientos Asignados: [{}]", equipmentList.isEmpty() ? "Ninguno" : equipmentList);

            // Campos específicos mostrados de forma segura sin romper el encapsulamiento de negocio
            if (emp instanceof EmployeeIT itEmp) {
                String projectsList = itEmp.getProjects().stream()
                        .map(Project::getName)
                        .collect(Collectors.joining(", "));
                log.info("  -> [IT] Rol: {} | Teletrabajo: {} | Proyectos: [{}]", itEmp.getRole(), itEmp.isRemote(), projectsList);
            } else if (emp instanceof EmployeeShop shopEmp) {
                log.info("  -> [TIENDA] Rol: {}", shopEmp.getRole());
            }
        });

        log.info("--------------------------------------------------------------------------------");
        log.info("=== ESCENARIO DE POO FINALIZADO CORRECTAMENTE ===");
        log.info("================================================================================");
    }
}
```

---

## 🌟 ¿Por qué es este el diseño de referencia definitivo para el aula?

Cuando realices la puesta en común y el debate en la pizarra, te sugerimos enfocar la atención en estos 3 pilares estructurales:

1. **La Alineación Exacta de las 5 Relaciones**: Las relaciones se numeran e identifican de forma explícita e idéntica en los comentarios de las clases de dominio, facilitando que los alumnos mapeen mentalmente el enunciado con el código de referencia.
2. **Polimorfismo Puro sobre Selectores de IRPF**: El servicio no contiene un solo condicional de tipo. Toda la autonomía fiscal del IRPF (**10%, 12%, 15% o 18%**) está delegada en las clases especializadas `EmployeeShop` y `EmployeeIT`, cumpliendo rigurosamente con las **Reglas de Oro 1 y 2**.
3. **Robustez y Acoplamiento Débil**: La arquitectura está completamente desacoplada e inyectada mediante constructor (`@RequiredArgsConstructor` de Lombok), y utiliza colecciones concurrentes en memoria, preparando el terreno perfecto antes de dar el salto a las bases de datos relacionales con JPA/Hibernate en la asignatura.
