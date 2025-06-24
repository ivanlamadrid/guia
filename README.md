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


Copias-pega, remplaza los literales <root-package> (paquete base con tu código), <context> (simkl ó work) y los nombres variables en cada caso.

1 · Carpeta shared (idéntica para WS51 y WS53)
Auditable.java — timestamps comunes [E]
java
Copiar
Editar
package <root-package>.shared.domain.model;

import jakarta.persistence.Column;
import jakarta.persistence.EntityListeners;
import jakarta.persistence.MappedSuperclass;
import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.OffsetDateTime;

/**
 * Super-type that adds automatic creation/update timestamps
 * to every aggregate root that extends it.
 */
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class Auditable {

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private OffsetDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at", nullable = false)
    private OffsetDateTime updatedAt;
}
BaseEntity.java — ID autogenerado [E] (opcional, pero útil)
java
Copiar
Editar
package <root-package>.shared.domain.model;

import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.MappedSuperclass;
import lombok.Getter;

/** Adds a surrogate Long primary key to any entity. */
@Getter
@MappedSuperclass
public abstract class BaseEntity extends Auditable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
BusinessRuleException.java — excepción de dominio [E]
java
Copiar
Editar
package <root-package>.shared.domain.exceptions;

/** Thrown when a business invariant is violated. */
public class BusinessRuleException extends RuntimeException {
    public BusinessRuleException(String message) {
        super(message);
    }
}
SnakePluralNamingStrategy.java — convención snake_case + plural [E]
java
Copiar
Editar
package <root-package>.shared.infrastructure.naming;

import org.hibernate.boot.model.naming.Identifier;
import org.hibernate.boot.model.naming.PhysicalNamingStrategy;
import org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl;
import org.hibernate.engine.jdbc.env.spi.JdbcEnvironment;

/**
 * Converts CamelCase → snake_case and pluralises table names automatically.
 */
public class SnakePluralNamingStrategy extends PhysicalNamingStrategyStandardImpl
        implements PhysicalNamingStrategy {

    @Override
    public Identifier toPhysicalTableName(Identifier name, JdbcEnvironment env) {
        String snake = camelToSnake(name.getText());
        if (!snake.endsWith("s")) snake += "s";
        return Identifier.toIdentifier(snake);
    }

    @Override
    public Identifier toPhysicalColumnName(Identifier name, JdbcEnvironment env) {
        return Identifier.toIdentifier(camelToSnake(name.getText()));
    }

    private String camelToSnake(String text) {
        return text.replaceAll("([a-z])([A-Z]+)", "$1_$2").toLowerCase();
    }
}
JpaConfiguration.java — activa JPA + auditoría [E]
java
Copiar
Editar
package <root-package>.shared.infrastructure.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

/** Global JPA configuration shared by every context. */
@Configuration
@EnableJpaAuditing
public class JpaConfiguration { }
OpenApiConfig.java — Swagger / springdoc [E]
java
Copiar
Editar
package <root-package>.shared.infrastructure.config;

import io.swagger.v3.oas.annotations.OpenAPIDefinition;
import io.swagger.v3.oas.annotations.info.Info;
import org.springframework.context.annotation.Configuration;

/** Publishes the OpenAPI definition at /swagger-ui.html */
@Configuration
@OpenAPIDefinition(
        info = @Info(title = "SI729 Practice API", version = "v1")
)
public class OpenApiConfig { }
2 · Bounded-context simkl (WS51 – Movies)
Paquete raíz: io.apiary.platform.u<codigo>.simkl

2.1 Dominio
MovieType.java [Enum] [V]
java
Copiar
Editar
package <root-package>.simkl.domain.model;

import java.util.Locale;

/** Enumeration of supported movie genres. */
public enum MovieType {
    COMEDY, HORROR, FANTASY, ROMANCE, ACTION, ADVENTURE, THRILLER;

    /** Case-insensitive factory used by the service layer. */
    public static MovieType of(String name) {
        return MovieType.valueOf(name.toUpperCase(Locale.ROOT));
    }
}
Value Objects [V]
java
Copiar
Editar
package <root-package>.simkl.domain.model.valueobject;

import jakarta.validation.constraints.*;

/** 2-letter ISO country code. */
public record CountryId(
        @NotBlank @Size(max = 2) String value) {
    public CountryId {
        value = value.toUpperCase();
    }
}

public record DirectorId(
        @NotNull @Positive Long value) { }

public record DistributedId(
        @NotNull @Positive Long value) { }
Movie.java — aggregate root [V]
java
Copiar
Editar
package <root-package>.simkl.domain.model;

import <root-package>.shared.domain.model.BaseEntity;
import <root-package>.shared.domain.exceptions.BusinessRuleException;
import <root-package>.simkl.domain.model.valueobject.*;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.Getter;
import lombok.NoArgsConstructor;

/**
 * Aggregate root representing a Movie in Simkl's dataset.
 */
@Getter
@NoArgsConstructor
@Entity
@Table(
    uniqueConstraints = @UniqueConstraint(
        columnNames = {"title", "director_id", "movie_type"}))
public class Movie extends BaseEntity {

    @Column(length = 100, nullable = false)
    private String title;

    @Embedded
    private CountryId countryId;

    @Embedded
    @AttributeOverrides(@AttributeOverride(
        name = "value", column = @Column(name = "director_id", nullable = false)))
    private DirectorId directorId;

    @Embedded
    @AttributeOverrides(@AttributeOverride(
        name = "value", column = @Column(name = "distributed_id", nullable = false)))
    private DistributedId distributedId;

    @Enumerated(EnumType.ORDINAL)
    @Column(name = "movie_type", nullable = false)
    private MovieType movieType;

    @Column(length = 240, nullable = false)
    private String sinopsis;

    @Min(1) @Max(10) @Column(nullable = false)
    private Integer imdbRating;

    @Positive @Column(nullable = false)
    private Float budget;

    @Column(nullable = false)
    private java.time.LocalDate releaseAt;

    /* Factory method with business validations */
    public static Movie create(
            String title, CountryId countryId, DirectorId directorId,
            DistributedId distributedId, MovieType movieType,
            String sinopsis, Integer imdbRating,
            Float budget, java.time.LocalDate releaseAt) {

        if (budget <= 0) throw new BusinessRuleException("Budget must be positive");
        if (imdbRating < 1 || imdbRating > 10)
            throw new BusinessRuleException("IMDb rating must be between 1 and 10");

        Movie m = new Movie();
        m.title = title;
        m.countryId = countryId;
        m.directorId = directorId;
        m.distributedId = distributedId;
        m.movieType = movieType;
        m.sinopsis = sinopsis;
        m.imdbRating = imdbRating;
        m.budget = budget;
        m.releaseAt = releaseAt;
        return m;
    }
}
2.2 Repositorio
java
Copiar
Editar
package <root-package>.simkl.domain.repository;

import <root-package>.simkl.domain.model.Movie;
import <root-package>.simkl.domain.model.MovieType;
import <root-package>.simkl.domain.model.valueobject.DirectorId;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MovieRepository extends JpaRepository<Movie, Long> {
    boolean existsByTitleAndDirectorIdAndMovieType(
            String title, DirectorId directorId, MovieType movieType);
}
2.3 Aplicación (Command side)
CreateMovieCommand.java [V]
java
Copiar
Editar
package <root-package>.simkl.application.command;

import jakarta.validation.constraints.*;
import java.time.LocalDate;

/** DTO for POST /directors/{id}/movies */
public record CreateMovieCommand(
        @NotBlank @Size(max = 100) String title,
        @NotBlank @Size(max = 2) String countryId,
        @NotNull Long distributedId,
        @NotBlank @Size(max = 240) String sinopsis,
        @Min(1) @Max(10) Integer imdbRating,
        @Positive Float budget,
        @NotBlank String movieType,
        @NotNull LocalDate releaseAt
) { }
MovieCommandService.java [E]
java
Copiar
Editar
package <root-package>.simkl.application.service;

import <root-package>.shared.domain.exceptions.BusinessRuleException;
import <root-package>.simkl.domain.model.*;
import <root-package>.simkl.domain.model.valueobject.*;
import <root-package>.simkl.domain.repository.MovieRepository;
import <root-package>.simkl.application.command.CreateMovieCommand;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/** Handles write-side use-cases for Movies. */
@Service
@RequiredArgsConstructor
public class MovieCommandService {

    private final MovieRepository repo;

    @Transactional
    public Movie addMovie(Long directorIdValue, CreateMovieCommand c) {
        DirectorId directorId = new DirectorId(directorIdValue);
        MovieType type = MovieType.of(c.movieType());

        if (repo.existsByTitleAndDirectorIdAndMovieType(c.title(), directorId, type))
            throw new BusinessRuleException("Duplicate movie for same director and type");

        Movie movie = Movie.create(
                c.title(),
                new CountryId(c.countryId()),
                directorId,
                new DistributedId(c.distributedId()),
                type,
                c.sinopsis(),
                c.imdbRating(),
                c.budget(),
                c.releaseAt()
        );
        return repo.save(movie);
    }
}
2.4 Interfaces (REST)
MovieMapper.java — Entity ↔ Resource [E]
java
Copiar
Editar
package <root-package>.simkl.interfaces.mapper;

import <root-package>.simkl.domain.model.Movie;
import org.springframework.stereotype.Component;

/** Simple manual mapper (MapStruct también sería válido). */
@Component
public class MovieMapper {

    public MovieResource toResource(Movie m) {
        return new MovieResource(
                m.getId(), m.getTitle(), m.getMovieType().name(),
                m.getImdbRating(), m.getBudget(), m.getReleaseAt());
    }

    public record MovieResource(
            Long id, String title, String movieType,
            Integer imdbRating, Float budget, java.time.LocalDate releaseAt) { }
}
MovieController.java [E]
java
Copiar
Editar
package <root-package>.simkl.interfaces.rest;

import <root-package>.simkl.application.command.CreateMovieCommand;
import <root-package>.simkl.application.service.MovieCommandService;
import <root-package>.simkl.interfaces.mapper.MovieMapper.MovieResource;
import <root-package>.simkl.interfaces.mapper.MovieMapper;
import io.swagger.v3.oas.annotations.Operation;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

/** Exposes the POST endpoint required by the practice. */
@RestController
@RequestMapping("/api/v1/directors/{directorId}/movies")
@RequiredArgsConstructor
public class MovieController {

    private final MovieCommandService service;
    private final MovieMapper mapper;

    @Operation(summary = "Add new movie for a director")
    @PostMapping
    public ResponseEntity<MovieResource> add(
            @PathVariable Long directorId,
            @Valid @RequestBody CreateMovieCommand cmd) {

        MovieResource body = mapper.toResource(service.addMovie(directorId, cmd));
        return ResponseEntity.status(HttpStatus.CREATED).body(body);
    }
}
3 · Bounded-context work (WS53 – Work Orders)
Paquete raíz: pe.medelect.platform.u<codigo>.work

(El patrón es idéntico; solo se muestran los archivos variables)

3.1 Dominio
WorkType.java
java
Copiar
Editar
package <root-package>.work.domain.model;

import java.util.Locale;

public enum WorkType {
    PREVENTIVE_MAINTENANCE,
    CORRECTIVE_MAINTENANCE,
    PARAMETER_VERIFICATION,
    REPOWERING;

    public static WorkType of(String name) {
        return WorkType.valueOf(name.toUpperCase(Locale.ROOT));
    }
}
Value Objects
java
Copiar
Editar
package <root-package>.work.domain.model.valueobject;

import jakarta.validation.constraints.*;

public record MedicalEquipmentId(
        @NotBlank @Size(max = 100) String value) { }

public record StaffId(
        @NotNull @Positive Long value) { }

public record HealthInstitutionId(
        @NotNull @Positive Long value) { }
WorkOrder.java
java
Copiar
Editar
package <root-package>.work.domain.model;

import <root-package>.shared.domain.model.BaseEntity;
import <root-package>.shared.domain.exceptions.BusinessRuleException;
import <root-package>.work.domain.model.valueobject.*;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.Getter;
import lombok.NoArgsConstructor;
import java.time.LocalDate;

@Getter
@NoArgsConstructor
@Entity
@Table(uniqueConstraints = @UniqueConstraint(
        columnNames = {"medical_equipment_id", "staff_id", "planned_at"}))
public class WorkOrder extends BaseEntity {

    @Embedded
    @AttributeOverride(name = "value",
            column = @Column(name = "medical_equipment_id", length = 100, nullable = false))
    private MedicalEquipmentId medicalEquipmentId;

    @Embedded
    @AttributeOverride(name = "value",
            column = @Column(name = "staff_id", nullable = false))
    private StaffId staffId;

    @Embedded
    @AttributeOverride(name = "value",
            column = @Column(name = "health_institution_id", nullable = false))
    private HealthInstitutionId healthInstitutionId;

    @Enumerated(EnumType.ORDINAL)
    @Column(name = "work_type", nullable = false)
    private WorkType workType;

    @Column(length = 200, nullable = false)
    private String description;

    @Min(1) @Max(3) @Column(nullable = false)
    private Integer priority;

    @Positive @Column(nullable = false)
    private Float amount;

    @Column(name = "planned_at", nullable = false)
    private LocalDate plannedAt;

    public static WorkOrder create(
            MedicalEquipmentId medicalEquipmentId, StaffId staffId,
            HealthInstitutionId healthInstitutionId, WorkType workType,
            String description, Integer priority, Float amount, LocalDate plannedAt) {

        if (amount <= 0) throw new BusinessRuleException("Amount must be positive");
        if (priority < 1 || priority > 3)
            throw new BusinessRuleException("Priority must be between 1 and 3");

        WorkOrder w = new WorkOrder();
        w.medicalEquipmentId = medicalEquipmentId;
        w.staffId = staffId;
        w.healthInstitutionId = healthInstitutionId;
        w.workType = workType;
        w.description = description;
        w.priority = priority;
        w.amount = amount;
        w.plannedAt = plannedAt;
        return w;
    }
}
3.2 Repositorio
java
Copiar
Editar
package <root-package>.work.domain.repository;

import <root-package>.work.domain.model.WorkOrder;
import <root-package>.work.domain.model.valueobject.MedicalEquipmentId;
import <root-package>.work.domain.model.valueobject.StaffId;
import org.springframework.data.jpa.repository.JpaRepository;

import java.time.LocalDate;

public interface WorkOrderRepository extends JpaRepository<WorkOrder, Long> {
    boolean existsByMedicalEquipmentIdAndStaffIdAndPlannedAt(
            MedicalEquipmentId medicalEquipmentId, StaffId staffId, LocalDate plannedAt);
}
3.3 Aplicación
CreateWorkOrderCommand.java
java
Copiar
Editar
package <root-package>.work.application.command;

import jakarta.validation.constraints.*;
import java.time.LocalDate;

public record CreateWorkOrderCommand(
        @NotNull Long staffId,
        @NotNull Long healthInstitutionId,
        @NotBlank String workType,
        @NotBlank @Size(max = 200) String description,
        @Min(1) @Max(3) Integer priority,
        @Positive Float amount,
        @NotNull LocalDate plannedAt
) { }
WorkOrderCommandService.java
java
Copiar
Editar
package <root-package>.work.application.service;

import <root-package>.shared.domain.exceptions.BusinessRuleException;
import <root-package>.work.domain.model.*;
import <root-package>.work.domain.model.valueobject.*;
import <root-package>.work.domain.repository.WorkOrderRepository;
import <root-package>.work.application.command.CreateWorkOrderCommand;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;

@Service
@RequiredArgsConstructor
public class WorkOrderCommandService {

    private final WorkOrderRepository repo;

    @Transactional
    public WorkOrder addWorkOrder(String medicalEquipmentIdRaw,
                                  CreateWorkOrderCommand c) {

        MedicalEquipmentId meId = new MedicalEquipmentId(medicalEquipmentIdRaw);
        StaffId staffId = new StaffId(c.staffId());
        LocalDate date = c.plannedAt();

        if (repo.existsByMedicalEquipmentIdAndStaffIdAndPlannedAt(meId, staffId, date))
            throw new BusinessRuleException("Duplicate work order for same equipment, staff and date");

        WorkOrder w = WorkOrder.create(
                meId, staffId,
                new HealthInstitutionId(c.healthInstitutionId()),
                WorkType.of(c.workType()),
                c.description(), c.priority(),
                c.amount(), date
        );
        return repo.save(w);
    }
}
3.4 Interfaces
WorkOrderMapper.java
java
Copiar
Editar
package <root-package>.work.interfaces.mapper;

import <root-package>.work.domain.model.WorkOrder;
import org.springframework.stereotype.Component;

@Component
public class WorkOrderMapper {

    public WorkOrderResource toResource(WorkOrder w) {
        return new WorkOrderResource(
                w.getId(),
                w.getMedicalEquipmentId().value(),
                w.getWorkType().name(),
                w.getPriority(),
                w.getAmount(),
                w.getPlannedAt());
    }

    public record WorkOrderResource(
            Long id, String medicalEquipmentId,
            String workType, Integer priority,
            Float amount, java.time.LocalDate plannedAt) { }
}
WorkOrderController.java
java
Copiar
Editar
package <root-package>.work.interfaces.rest;

import <root-package>.work.application.command.CreateWorkOrderCommand;
import <root-package>.work.application.service.WorkOrderCommandService;
import <root-package>.work.interfaces.mapper.WorkOrderMapper.WorkOrderResource;
import <root-package>.work.interfaces.mapper.WorkOrderMapper;
import io.swagger.v3.oas.annotations.Operation;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

/** POST /api/v1/medical-equipments/{id}/work-orders */
@RestController
@RequestMapping("/api/v1/medical-equipments/{medicalEquipmentId}/work-orders")
@RequiredArgsConstructor
public class WorkOrderController {

    private final WorkOrderCommandService service;
    private final WorkOrderMapper mapper;

    @Operation(summary = "Add new work order for a medical equipment")
    @PostMapping
    public ResponseEntity<WorkOrderResource> add(
            @PathVariable String medicalEquipmentId,
            @Valid @RequestBody CreateWorkOrderCommand cmd) {

        WorkOrderResource body = mapper.toResource(
                service.addWorkOrder(medicalEquipmentId, cmd));
        return ResponseEntity.status(HttpStatus.CREATED).body(body);
    }
}
4 · application.yml (cambia solo puerto & schema)
Simkl – src/main/resources/application.yml
yaml
Copiar
Editar
server:
  port: 8096

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/postgres?currentSchema=apiary
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        physical_naming_strategy: <root-package>.shared.infrastructure.naming.SnakePluralNamingStrategy
Medelect – idéntico pero:
yaml
Copiar
Editar
server.port: 8095
spring.datasource.url: jdbc:postgresql://localhost:5432/postgres?currentSchema=medelect
5 · Excepción global (opcional pero aconsejada)
java
Copiar
Editar
package <root-package>.shared.infrastructure.config;

import <root-package>.shared.domain.exceptions.BusinessRuleException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.Map;
import java.util.stream.Collectors;

@RestControllerAdvice
public class RestExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    ResponseEntity<?> badRequest(MethodArgumentNotValidException ex) {
        Map<String, String> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .collect(Collectors.toMap(FieldError::getField, FieldError::getDefaultMessage));
        return ResponseEntity.badRequest().body(errors);
    }

    @ExceptionHandler(BusinessRuleException.class)
    ResponseEntity<?> business(BusinessRuleException ex) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
                .body(Map.of("error", ex.getMessage()));
    }
}
➡️ Cómo usar estas plantillas
Copia cada bloque en su ruta correspondiente.

Sustituye \<root-package> y \<codigo> una sola vez; tu IDE ajustará el resto.

Ajusta únicamente los archivos marcados [V] cuando cambies de WS51 a WS53.

Ejecuta ./mvnw spring-boot:run y verifica el endpoint (curl o Postman).

¡Con estos archivos cubres toda la rúbrica “esencial” para ambas prácticas!
