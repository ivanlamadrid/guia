# guia

```text
src/
└── main/
    ├── java/
    │   └── pe.medelect.platform.u<codigo>/
    │       ├── shared/
    │       │   ├── domain/
    │       │   │   ├── model/
    │       │   │   │   ├── Auditable.java
    │       │   │   │   └── BaseEntity.java
    │       │   │   └── exceptions/
    │       │   │       └── BusinessRuleException.java
    │       │   └── infrastructure/
    │       │       ├── naming/
    │       │       │   └── SnakePluralNamingStrategy.java
    │       │       └── config/
    │       │           ├── JpaConfiguration.java
    │       │           └── OpenApiConfig.java
    │       └── work/
    │           ├── domain/
    │           │   ├── model/
    │           │   │   ├── WorkOrder.java
    │           │   │   ├── WorkType.java
    │           │   │   └── valueobject/
    │           │   │       ├── MedicalEquipmentId.java
    │           │   │       ├── StaffId.java
    │           │   │       └── HealthInstitutionId.java
    │           │   └── repository/
    │           │       └── WorkOrderRepository.java
    │           ├── application/
    │           │   ├── command/
    │           │   │   └── CreateWorkOrderCommand.java
    │           │   └── service/
    │           │       └── WorkOrderCommandService.java
    │           ├── infrastructure/
    │           │   └── persistence/
    │           │       └── JpaWorkOrderRepositoryImpl.java
    │           └── interfaces/
    │               ├── rest/
    │               │   └── WorkOrderController.java
    │               └── mapper/
    │                   └── WorkOrderMapper.java
    └── resources/
        ├── application.yml
        └── db/
            └── migration/
                └── V1__init.sql
