# guia

```text
src/
└── main/
    ├── java/
    │   └── <root-package>/                      # [V]  io.apiary.platform.u<codigo>  |  pe.medelect.platform.u<codigo>
    │       ├── shared/                          # [E]  utilidades reutilizables
    │       │   ├── domain/
    │       │   │   ├── model/
    │       │   │   │   ├── Auditable.java       # [E]  timestamps createdAt / updatedAt
    │       │   │   │   └── BaseEntity.java      # [E]  superclase opcional con id
    │       │   │   └── exceptions/
    │       │   │       └── BusinessRuleException.java   # [E]  dominio → HTTP 409/400
    │       │   └── infrastructure/
    │       │       ├── naming/
    │       │       │   └── SnakePluralNamingStrategy.java   # [E]  CamelCase → snake_case + plural
    │       │       └── config/
    │       │           ├── JpaConfiguration.java    # [E]  habilita JPA + auditing
    │       │           └── OpenApiConfig.java       # [E]  activa springdoc-openapi
    │       └── <context>/                           # [V]  simkl | work
    │           ├── domain/
    │           │   ├── model/
    │           │   │   ├── <AggregateRoot>.java     # [V]  Movie.java | WorkOrder.java
    │           │   │   ├── <Enum>.java              # [V]  MovieType.java | WorkType.java
    │           │   │   └── valueobject/
    │           │   │       ├── FirstVO.java         # [V]  CountryId / MedicalEquipmentId
    │           │   │       ├── SecondVO.java        # [V]  DirectorId / StaffId
    │           │   │       └── ThirdVO.java         # [V]  DistributedId / HealthInstitutionId
    │           │   └── repository/
    │           │       └── <AggregateRoot>Repository.java   # [E]  interface Spring Data
    │           ├── application/
    │           │   ├── command/
    │           │   │   └── Create<AggregateRoot>Command.java   # [V]  DTO de entrada POST
    │           │   └── service/
    │           │       └── <AggregateRoot>CommandService.java  # [E]  orquesta reglas de negocio
    │           ├── infrastructure/
    │           │   └── persistence/
    │           │       └── Jpa<AggregateRoot>RepositoryImpl.java   # [V]  solo si necesitas SQL custom
    │           └── interfaces/
    │               ├── rest/
    │               │   └── <AggregateRoot>Controller.java    # [E]  endpoint REST
    │               └── mapper/
    │                   └── <AggregateRoot>Mapper.java        # [E]  Entity ↔ Resource/DTO
    └── resources/
        ├── application.yml                # [E]  server.port (8096 / 8095), schema (apiary / medelect)
        └── db/
            └── migration/
                └── V1__init.sql           # [V]  DDL con Flyway/Liquibase (opcional)
```
