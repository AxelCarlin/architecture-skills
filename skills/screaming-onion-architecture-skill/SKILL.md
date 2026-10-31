---
name: screaming-onion-architecture
description: Applies a business-oriented architecture that combines Screaming Architecture to organize by capabilities and use cases, Onion/Clean Architecture to control dependencies, and Vertical Slices to keep each flow cohesive. Use it when designing, implementing, reviewing, or refactoring applications, especially .NET + Vue/TypeScript.
---

# Skill: Screaming + Onion + Clean Architecture

## 1. Mission

Help design and modify software whose structure reveals **what the business does**, whose modules are easy to locate and evolve, and whose business rules do not depend on frameworks, HTTP, or persistence.

The proposed integration has three core dimensions plus an optional fourth:

1. **Screaming Architecture — organization:** the repository communicates business capabilities (`Permissions`, `Users`, `Cutoffs`) instead of communicating only technologies (`Controllers`, `Services`, `Repositories`).
2. **Onion Architecture — dependency direction:** external code depends on contracts and internal rules; the domain does not know about ASP.NET Core, EF Core, SQL Server, Vue, or infrastructure details.
3. **Clean Architecture — boundaries and policies:** separates entities/rules, use cases, adapters, and frameworks; keeps dependencies pointing toward internal policies.
4. **Vertical Slice — conditional unit of change:** within a capability, organize code by flow/use case when that improves cohesion (`AuthorizePermission`, `CancelPermission`). It is not a synonym for the other three architectures and does not require duplicating the entire structure in every slice.

**Synthesis rule:** organize by business first; within each capability, group meaningful independent operations by use case and keep trivial operations proportionate; preserve the dependency boundaries of Onion/Clean.

## 2. When to use this skill

Enable it when the user asks to:

- design the architecture of a new application;
- create modules, features, endpoints, or use cases;
- evaluate an existing architecture;
- refactor layered projects into business modules;
- define names, folders, projects, contracts, or dependency rules;
- review whether an implementation respects Screaming, Onion, or Clean Architecture.

Before proposing changes, determine whether the work is new design, a new feature, an audit, or a migration. In existing systems, inspect actual evidence before recommending moves.

## 3. Verifiable objectives

A solution complies with this skill when:

- The folder tree reveals the product's main capabilities.
- Important business rules can be tested without starting HTTP, a database, or infrastructure.
- Use cases express a business action and coordinate the flow without becoming a second controller layer.
- Infrastructure implements contracts required by the application; the application does not import infrastructure.
- Controllers are HTTP protocol adapters, not the usual place for business rules.
- Input/output DTOs are not automatically confused with domain entities or persistence entities.
- Modules share code only when there is a truly shared responsibility.
- Architectural exceptions are documented and tested.
- Changes are verified with tests and, where appropriate, automated dependency analysis.

Every project is not required to have many .NET projects, generic repositories, CQRS, MediatR, Domain Events, or one interface per class. Complexity must respond to real needs.

## 4. Vocabulary and conceptual boundaries

### Screaming Architecture

Question: **Where do I find the business behavior?**

Organize code so a developer can quickly identify permissions, users, cutoffs, areas, reports, etc. This favors cohesion and change locality.

It does not by itself determine which project may reference which. Having a `Features/Permissions` folder does not automatically make the system a modular architecture.

### Onion Architecture

Question: **In which direction do dependencies travel?**

The domain sits at the center. The application depends on the domain. Infrastructure implements the contracts defined inward. The API and the composition root connect the pieces.

Compilation dependencies must point toward internal policies, even though the execution flow may call from a use case to an external implementation through an interface.

### Clean Architecture

Question: **Which policies are internal, and which details must be replaceable?**

It reinforces the separation between:

- **Entities / Domain:** business rules and domain concepts.
- **Use Cases / Application:** intent, coordination, and rules specific to each operation.
- **Interface Adapters:** translation between HTTP/JSON, application models, and persistence.
- **Frameworks & Drivers / Infrastructure:** ASP.NET Core, EF Core, SQL Server, email, files, and external services.

Onion and Clean share the essential rule of dependency inversion. They are not labels that must be implemented with an identical rigid template.

### Vertical Slice Architecture

Question: **How do I keep together what changes to complete a use case?**

A slice groups the code for a flow, for example `Permissions/Authorize`. It can contain a request, handler/use case, validator, and response. Shared persistence and domain code are not duplicated just to make each slice look independent.

**Do not claim:** "Screaming + Onion automatically means Vertical Slice." The combination allows slices, but they are related and distinct decisions.

## 5. Target architecture

### 5.1 Recommended option for .NET with explicit boundaries

```text
Backend/
  src/
    GestionPermisos.Api/
      Features/
        Permissions/
          PermissionsController.cs
          Http/
            AuthorizePermissionRequest.cs
            PermissionResponse.cs
        Users/
        Cutoffs/
      Middleware/
      Authorization/
      DependencyInjection.cs
      Program.cs

    GestionPermisos.Application/
      Features/
        Permissions/
          Authorize/
            AuthorizePermissionCommand.cs
            AuthorizePermissionHandler.cs
            AuthorizePermissionValidator.cs
            AuthorizePermissionResult.cs
          Create/
          Cancel/
          List/
        Users/
        Cutoffs/
      Abstractions/
        Persistence/
          IPermissionRepository.cs
          IUnitOfWork.cs
        Clock/
          IClock.cs

    GestionPermisos.Domain/
      Permissions/
        Permission.cs
        PermissionStatus.cs
        PermissionPolicy.cs
      Users/
        User.cs
      Cutoffs/
      Shared/
        DomainException.cs

    GestionPermisos.Infrastructure/
      Persistence/
        HumanResourcesContext.cs
        Configurations/
        Repositories/
          PermissionRepository.cs
        UnitOfWork.cs
      Authentication/
      Email/
      Files/
      DependencyInjection.cs

  tests/
    GestionPermisos.Domain.Tests/
    GestionPermisos.Application.Tests/
    GestionPermisos.Infrastructure.Tests/
    GestionPermisos.Api.Tests/
```

The exact structure must adapt to the size of the system. If the solution does not need four assemblies, it can keep fewer projects and apply the same boundaries through namespaces, controlled references, and architecture tests. Separating projects is useful when it turns rules into compilation constraints; it is not an aesthetic goal.

### 5.2 Responsibility map

| Part | Responsibility | Must not do |
|---|---|---|
| `Api` | HTTP, transport authentication, endpoint authorization, model binding, HTTP status codes, composition, and DI | Contain central permission rules or execute complex business queries directly |
| `Application` | Use cases, coordination, operation validation, required contracts, and results | Depend on `Infrastructure`, `DbContext`, `IActionResult`, or HTTP types |
| `Domain` | Entities, value objects, invariants, and business rules | Import EF Core, ASP.NET Core, JSON, files, or external services |
| `Infrastructure` | Implement persistence, email, files, the real clock, and other external adapters | Decide business policies that should belong in Domain/Application |
| `Frontend/features` | User interface and flows grouped by capability | Become the source of authority for security permissions |
| `Shared` / `Common` | Truly cross-cutting, stable, and small types | Become a dumping ground for helpers, DTOs, or logic that belongs to a feature |

### 5.3 Allowed dependency direction

```text
API ───────────────► Application ───────────────► Domain
 │                         ▲
 │                         │ implements contracts
 └────► Infrastructure ────┴────────────────────► Domain
```

Interpretation:

- `Application` can declare `IPermissionRepository`.
- `Infrastructure` implements `IPermissionRepository` with EF Core.
- `Application` does not reference the `Infrastructure` project.
- `Api` or the composition root registers `IPermissionRepository -> PermissionRepository`.
- `Domain` knows none of the outer projects.

Suggested project dependencies:

| Project | May reference |
|---|---|
| `Domain` | BCL and minimal packages that are truly neutral |
| `Application` | `Domain` |
| `Infrastructure` | `Application`, `Domain`, and necessary technical libraries |
| `Api` | `Application`, `Infrastructure` for composition and, if needed, `Domain` for explicit mapping |
| Domain tests | `Domain` |
| Application tests | `Application`, `Domain`, and test doubles |
| Infrastructure tests | `Infrastructure` and test providers |
| API tests | `Api` and integration tooling |

Avoid cycles. The table is an initial policy, not an obligation for everyone to need the same assemblies.

## 6. Normative rules of the skill

The labels mean:

- **MUST / Mandatory:** default rule; violating it requires an explicit exception.
- **SHOULD / Recommended:** apply unless there is a concrete reason not to.
- **MAY / Optional:** use it if it adds value.

### R1 — Organize by business capability (MUST)

Prefer `Features/Permissions`, `Features/Users`, `Features/Cutoffs` over global folders that scatter a single capability across `Controllers/`, `Services/`, `Repositories/`, and `DTOs/`.

Global technical folders remain valid for true technical boundaries: `Middleware`, `Persistence`, `Authentication`, `Observability`.

### R2 — Dependency direction (MUST)

Domain rules must not depend on external details. The application declares the abstractions it needs; infrastructure implements them.

Do not introduce references from `Domain` or `Application` to `Infrastructure` to satisfy implementation convenience.

### R3 — Pure domain by default (MUST)

Domain entities must not depend on:

- `Microsoft.AspNetCore.*`
- `Microsoft.EntityFrameworkCore.*`
- HTTP DTOs, `IActionResult`, `HttpContext`
- SQL clients, email, the file system, or the UI.

Persistence annotations and EF mapping should live in Infrastructure configurations when practical. No external package is dogmatically prohibited; any domain dependency must be justified by stability and value.

### R4 — Explicit use cases (SHOULD)

If an operation coordinates rules, business authorization, multiple writes, transactions, email, or files, represent it as a use case in Application.

Examples: `AuthorizePermission`, `CancelPermission`, `ImportCutoffs`, `RegisterUser`.

For trivial read CRUD, do not create a five-class chain that only forwards parameters. A simple query can have a lightweight query handler or a read adapter, but it must not bypass dependency boundaries.

### R5 — No pass-through services (MUST)

Do not create `IPermissionService` + `PermissionService` if the class only calls another class and adds no policy, coordination, validation, transformation, or useful boundary.

The absence of a pass-through service **does not** authorize putting all logic in the controller. The endpoint must remain a thin adapter; meaningful logic goes into the use case or domain.

### R6 — Contracts where a dependency is inverted (SHOULD)

Create interfaces for boundaries that need to be substituted, tested, or implemented by an outer ring: persistence, clock, email delivery, file storage, or an external provider.

Do not create an interface for every class out of habit. The contract must belong to the inner layer that needs the service, not to the concrete layer that implements it.

### R7 — DTOs with a clear purpose (MUST)

Distinguish by intent, not only by suffix:

- `AuthorizePermissionRequest`: HTTP input.
- `AuthorizePermissionCommand`: application intent.
- `AuthorizePermissionResult`: use-case result.
- `PermissionResponse`: HTTP output.
- `Permission`: domain entity.
- `PermissionEntity` only if a separate persistence model truly exists and the convention needs it.

A DTO may be reused across boundaries when it is stable and does not leak transport or persistence details. Do not duplicate identical types without a reason.

### R8 — Validation in the correct layer (MUST)

- HTTP format and structure: API/model binding.
- Operation rules and permissions on the resource: Application.
- Invariants that must always hold: Domain.
- Integrity constraints: also in the database when appropriate.

Frontend validation improves the experience, but it never replaces server-side validation.

### R9 — Security independent of organization (MUST)

- Every endpoint must have explicit authentication and authorization.
- Check roles/policies and access to the resource; being authenticated does not mean being authorized.
- Do not trust the UI or an API key alone to protect an operation.
- Do not log secrets, tokens, passwords, unnecessary personal data, or connection strings.
- Use appropriate password hashing algorithms and salt; do not use plain SHA-256 for passwords.
- Verify CORS, antiforgery when applicable, secret management, and safe error handling.
- Architectural changes are not considered a security fix until tests demonstrate it.

### R10 — Transactions and integrity (MUST when applicable)

A use case that requires multiple atomic writes must define the transaction boundary. The transaction implementation belongs to Infrastructure or to an Application contract implemented by Infrastructure. Do not assume that multiple repository calls are atomic.

### R11 — Explicit errors and results (SHOULD)

Use cases should return typed results or throw well-defined application/domain exceptions according to the project convention. The API translates those results to HTTP (`200`, `201`, `400`, `403`, `404`, `409`, etc.) and `ProblemDetails` when appropriate. Do not leak stack traces to the client.

### R12 — Precise and consistent names (MUST)

Use business terms, verbs for actions, and names that explain intent. Avoid vague names (`Manager`, `Helper`, `CommonService`, `ProcessData`, `DoStuff`) unless the domain justifies the term.

### R13 — Frontend/backend symmetry without rigid coupling (SHOULD)

Maintain conceptual correspondence, not necessarily a 1:1 structure. For example, `Frontend/src/features/permissions` consumes permission endpoints but may split components by UX flow. Do not make the frontend import internal Domain or EF types.

### R14 — Shared under control (MUST)

Move code to `Shared` only if it is used by multiple modules and its semantics are truly common. Do not move a function to `Shared` just because it is used twice. Do not create `Common` as the default destination for everything that does not fit.

### R15 — Explicit exceptions (MUST)

A deviation must document: reason, scope, risk, discarded alternative, and the condition for re-evaluating it. Exceptions do not become a pattern through copy and paste.

### R16 — Verify before claiming (MUST)

Separate the following in reports:

- **Observed:** direct evidence from code or configuration.
- **Inferred:** probable conclusion, not verified.
- **Unverified:** missing evidence or execution.

Do not claim that tests pass if they were not run. Do not claim that a system complies with Clean/Onion only because it has projects with those names.

### R17 — Vertical slice decision policy (SHOULD)

Within each business capability, prefer a dedicated use-case slice when an operation has its own authorization, validation, business branching, transaction, side effects, distinct output, or independent change cadence. A slice is an **operation boundary**, not a new assembly, database, or domain model. A trivial stable CRUD read may remain a small query function or handler without ceremony. Record the reason when a substantial flow is not sliced.

### R18 — Slice cohesion and isolation (MUST when using slices)

Keep operation-specific request/command, validation, orchestration, result, and tests close together. One slice must not import another slice's handler, private DTO, validator, or implementation to reuse behavior. Coordinate cross-operation workflows at an explicitly named application use case; extract truly shared policies to Domain or stable ports/services to Application. Reuse Domain entities and Infrastructure adapters rather than cloning them per slice.

### R19 — Slice boundary ownership (MUST)

HTTP contracts and status mapping belong to API; operation intent and orchestration to Application; invariants to Domain; external implementations to Infrastructure. A physical slice may span several projects while retaining one recognizable operation name. A folder named `Features` does not waive these boundaries. Never put ASP.NET, EF Core, SQL clients, or transport response types inside Domain/Application merely to collocate a slice.

### R20 — Test decisions by behavior and risk (MUST)

For each new or modified operation, identify inputs, successful output, domain invariants, invalid input, authorization/resource ownership, missing data, conflict/concurrency, dependency failure, and transaction effects **as applicable**. Test the smallest boundary that can prove each behavior. Do not mechanically create a test for every class or every input field. Add regression tests for fixed bugs, characterization tests before risky refactors, integration tests for real persistence behavior, and API tests for routing/security/serialization. Record intentionally omitted cases and reasons.

### R21 — Automated architecture checks (SHOULD)

Where tooling permits, enforce project reference directions, forbid Infrastructure/API namespaces in Domain/Application, detect reference cycles, and guard slice-to-slice implementation imports. Run build, unit tests, integration tests where available, and static architecture checks in CI. Do not present folder structure alone as evidence of isolation. Keep the checks independent of folder-name fashions.

### R22 — Avoid premature extraction (MUST)

Before introducing shared code, identify a stable common policy or contract and at least two genuine consumers, or document a strong domain reason for earlier extraction. Similar-looking operations are not necessarily the same policy. Prefer a little local duplication over coupling unrelated features through a generic abstraction. Revisit only when evidence justifies it.

### R23 — Read/write design is proportional (MUST)

Vertical Slices do **not** require CQRS, MediatR, event buses, one handler interface per operation, or one repository per slice. Commands and queries are naming options, not architectural obligations. A simple read may use a narrow application query port implemented in Infrastructure. Writes must protect invariants, authorization, and atomicity as applicable. Choose complexity by observed needs.

### R24 — Cross-slice transactions and side effects (MUST when applicable)

When one business action spans multiple aggregates, repositories, or integrations, make the coordinating use case and consistency boundary explicit. Distinguish atomic database work from external effects that cannot join the same transaction. Define idempotency, retries, outbox/compensation, and concurrency controls only when the workflow warrants them; test the relevant failure modes. Do not hide a distributed workflow inside a repository or another slice's handler.

## 7. Recommended naming for C# and TypeScript

Naming should respect the conventions already established in the repository when they are coherent. In a new project:

| Element | Convention | Example |
|---|---|---|
| Project/namespace | `PascalCase` | `GestionPermisos.Application` |
| Class, record, interface, enum | `PascalCase` | `AuthorizePermissionHandler`, `IPermissionRepository` |
| Public method/property | `PascalCase` | `AuthorizeAsync`, `PermissionStatus` |
| Parameter/local variable | `camelCase` | `permissionId`, `currentUser` |
| Private field | `_camelCase` | `_permissionRepository` |
| Constants | `PascalCase` in C# | `MaximumEvidenceSize` |
| Use case | Verb + object | `AuthorizePermission`, `CancelPermission` |
| Port/interface | `I` + capability | `IPermissionRepository`, `IClock` |
| Implementation | Concrete name | `EfPermissionRepository`, `SystemClock` |
| HTTP input | Action + `Request` | `AuthorizePermissionRequest` |
| Command | Action + `Command` | `AuthorizePermissionCommand` |
| Query | Action/query + `Query` | `GetPermissionByIdQuery` |
| Result | Action + `Result` | `AuthorizePermissionResult` |
| HTTP response | Object + `Response` | `PermissionResponse` |
| Validator | Type + `Validator` | `AuthorizePermissionValidator` |
| Test | Type/action + behavior | `AuthorizePermissionHandlerTests` |
| Vue component | `PascalCase.vue` | `PermissionDetailsView.vue` |
| TypeScript functions/variables | `camelCase` | `authorizePermission()` |
| TypeScript types/interfaces | `PascalCase` | `PermissionResponse` |

Do not automatically introduce `Impl`, `Manager`, or `Helper` suffixes. `Repository` is reserved for a persistence abstraction/implementation with a defined responsibility; not every data access needs a generic repository.

## 8. Complete example: authorize a work permission

### 8.1 Flow

```text
HTTP POST /api/permissions/{id}/authorize
    ↓
PermissionsController (API: validates transport and translates HTTP)
    ↓
AuthorizePermissionHandler (Application: coordinates the use case)
    ├── IPermissionRepository (internal contract)
    ├── domain policy/rule (Permission.Authorize)
    └── IUnitOfWork (if the operation needs a transaction)
             ↑ implementations
    EfPermissionRepository + EfUnitOfWork (Infrastructure)
             ↓
        SQL Server / EF Core
```

The API must not call EF Core directly to decide whether a permission can be authorized. Application coordinates the flow, and Domain protects the invariants.

### 8.2 Domain

```csharp
namespace GestionPermisos.Domain.Permissions;

public enum PermissionStatus
{
    Pending,
    Authorized,
    Rejected,
    Cancelled
}

public sealed class Permission
{
    public Guid Id { get; private set; }
    public PermissionStatus Status { get; private set; }

    private Permission() { } // For mapping when the configuration requires it.

    public Permission(Guid id)
    {
        Id = id;
        Status = PermissionStatus.Pending;
    }

    public void Authorize()
    {
        if (Status != PermissionStatus.Pending)
            throw new InvalidPermissionTransitionException(
                "Only pending permissions can be authorized.");

        Status = PermissionStatus.Authorized;
    }
}
```

`InvalidPermissionTransitionException` would be a domain exception defined in the Domain project. This example is illustrative: the real rules must be confirmed with the business; do not assume that all permissions share this state flow.

### 8.3 Contract in Application

```csharp
using GestionPermisos.Domain.Permissions;

namespace GestionPermisos.Application.Abstractions.Persistence;

public interface IPermissionRepository
{
    Task<Permission?> GetByIdAsync(
        Guid permissionId,
        CancellationToken cancellationToken);

    void Update(Permission permission);
}
```

The contract lives in Application because the use case needs to persist permissions. Infrastructure will implement it.

### 8.4 Use case in Application

```csharp
namespace GestionPermisos.Application.Features.Permissions.Authorize;

public sealed record AuthorizePermissionCommand(
    Guid PermissionId,
    Guid AuthorizedByUserId);

public sealed record AuthorizePermissionResult(
    Guid PermissionId,
    string Status);

public sealed class AuthorizePermissionHandler
{
    private readonly IPermissionRepository _permissions;
    private readonly IUnitOfWork _unitOfWork;

    public AuthorizePermissionHandler(
        IPermissionRepository permissions,
        IUnitOfWork unitOfWork)
    {
        _permissions = permissions;
        _unitOfWork = unitOfWork;
    }

    public async Task<AuthorizePermissionResult> HandleAsync(
        AuthorizePermissionCommand command,
        CancellationToken cancellationToken)
    {
        var permission = await _permissions.GetByIdAsync(
            command.PermissionId, cancellationToken);

        if (permission is null)
            throw new PermissionNotFoundException(command.PermissionId);

        // Identity/resource authorization must be checked here
        // or through an explicit application policy.
        // Being authenticated is not enough.

        permission.Authorize();

        _permissions.Update(permission);
        await _unitOfWork.SaveChangesAsync(cancellationToken);

        return new AuthorizePermissionResult(
            permission.Id,
            permission.Status.ToString());
    }
}
```

The example deliberately omits the implementation of `IUnitOfWork`, the auxiliary exceptions, and the complete authorization check so that the architectural boundary remains visible. It must not be copied to production without implementing those requirements. In particular, `AuthorizedByUserId` does not by itself prove that the user is allowed to authorize.

### 8.5 Infrastructure adapter

```csharp
namespace GestionPermisos.Infrastructure.Persistence.Repositories;

internal sealed class EfPermissionRepository : IPermissionRepository
{
    private readonly HumanResourcesContext _db;

    public EfPermissionRepository(HumanResourcesContext db) => _db = db;

    public Task<Permission?> GetByIdAsync(
        Guid permissionId,
        CancellationToken cancellationToken) =>
        _db.Permissions.SingleOrDefaultAsync(
            x => x.Id == permissionId, cancellationToken);

    public void Update(Permission permission) =>
        _db.Permissions.Update(permission);
}
```

The mapping between domain and persistence depends on how the model was designed. If domain entities are persisted directly with EF Core, the EF configuration stays outside Domain. If separate persistence models are used, Infrastructure performs the explicit mapping.

### 8.6 HTTP adapter

```csharp
[ApiController]
[Route("api/permissions")]
public sealed class PermissionsController : ControllerBase
{
    private readonly AuthorizePermissionHandler _authorize;

    public PermissionsController(AuthorizePermissionHandler authorize) =>
        _authorize = authorize;

    [HttpPost("{id:guid}/authorize")]
    [Authorize(Policy = "CanAuthorizePermissions")]
    public async Task<ActionResult<PermissionResponse>> Authorize(
        Guid id,
        CancellationToken cancellationToken)
    {
        var userId = GetAuthenticatedUserId(); // Identity adapter, validated.
        var result = await _authorize.HandleAsync(
            new AuthorizePermissionCommand(id, userId),
            cancellationToken);

        return Ok(new PermissionResponse(result.PermissionId, result.Status));
    }
}
```

The `GetAuthenticatedUserId` method, the authorization policy, and the exception mapping are illustrative points that must be implemented explicitly. The HTTP policy does not replace resource authorization inside the use case when access depends on the state or owner of the permission.

## 9. Applying to the `itsmRH_GestionPermisos` system

### 9.1 Starting point observed in the report

The audit report describes a .NET web monolith with a Vue SPA, `Core`, `Data`, `Services`, `Crosscutting`, API, and test projects. It observes that:

- `Core` mixes entities, DTOs, ViewModels, enums, and interfaces.
- `Services` depends on `Data` and `Crosscutting`.
- `Crosscutting` also depends on `Data`.
- Controllers perform some orchestration and data access tasks.
- There are features in the presentation layer, but not complete modular boundaries.
- The described test suite does not cover all controllers, filters, auth, or E2E flows.
- The audit reports security risks that must be treated as separate work, not as an automatic consequence of changing folders.

These points are the baseline of the received report, not an independent certification of the repository's current state. Before changing it, re-inspect the branch, commit, and current code.

### 9.2 Gradual transition, not a massive rewrite

| Observed area | Conceptual target | Prudent strategy |
|---|---|---|
| API/controllers | `GestionPermisos.Api/Features/...` | Keep them as HTTP adapters; extract business logic as each flow is migrated |
| `Core` | `Domain` + `Application` contracts/DTOs | Classify each file by responsibility; do not move all of `Core` into Domain |
| `Services` | `Application/Features/...` for use cases; `Infrastructure` for integrations | Separate case by case according to actual dependencies |
| `Data` | `Infrastructure/Persistence` | Keep EF Core and `DbContext` outside Domain/Application |
| `Crosscutting` | API, Application, or Infrastructure depending on the function | Do not move everything as a block: HTTP auth, logging, hashing, and configuration have different owners |
| Tests | Domain/Application/Infrastructure/API tests | Add regression tests before and during each migration |
| Vue `features` | `src/features/permissions`, `users`, `cutoffs`, etc. | Preserve business organization and avoid importing internal backend models |

**Do not:** delete `Services`, `Core`, or `Crosscutting` in one operation; rename folders and declare that Onion was achieved; move all `Core` classes to Domain without classification; introduce a generic repository for every table without checking that it adds value.

### 9.3 Suggested migration order

1. **Freeze the baseline:** record the commit, build, tests executed, and limitations. Do not edit until you understand the current state.
2. **Inventory dependencies:** build the actual reference graph between projects and a call map for one feature.
3. **Choose a bounded flow:** for example, query or authorize a permission, according to complexity and risk.
4. **Add characterization tests:** capture current behavior before extracting logic.
5. **Define the Application contract:** declare only the ports needed for that use case.
6. **Extract the use case:** move coordination out of the controller without changing observable behavior.
7. **Separate the domain carefully:** move a rule to Domain only if it is a real business rule, not a simple utility method.
8. **Implement adapters:** infrastructure implements the contracts; the API registers dependencies in the composition root.
9. **Verify:** build, unit tests, integration with the real provider when applicable, and API/authorization tests.
10. **Repeat per feature:** incremental migration, with small commits and a clear rollback.
11. **Automate boundaries:** prevent forbidden references through architecture tests, solution rules, or dependency analysis.
12. **Update documentation:** record decisions, exceptions, and what was actually verified.

Feature priority and order must be defined after reviewing risk, dependencies, change frequency, and coverage. Do not assume that permissions is automatically the first correct module.

## 10. Testing and quality controls

### Minimum test pyramid per boundary

- **Domain tests:** invariants, state transitions, and rules without infrastructure.
- **Application tests:** use cases with port doubles; success, missing resource, conflict, dependency failure, and authorization.
- **Infrastructure integration tests:** EF Core and the real or compatible database engine; do not rely only on `EF InMemory` to validate SQL, transactions, or constraints.
- **API tests:** routes, serialization, HTTP status codes, authentication, authorization, and error handling.
- **Frontend tests:** loading/error states, UI permissions, and main flows; security remains the backend's responsibility.

### Dependency checklist

- [ ] `Domain` does not reference API, Infrastructure, or EF Core.
- [ ] `Application` does not reference API or Infrastructure.
- [ ] Infrastructure implements internal interfaces.
- [ ] There are no cycles between projects.
- [ ] The composition root registers the implementations.
- [ ] Tests do not unnecessarily depend on external services.
- [ ] Authorization and resource access are tested on the server.
- [ ] The exact commands executed and the actual results are recorded.

### Relevant standards and guidelines

Use these references as design and quality guidelines, not as automatic certification:

- **SOLID:** especially dependency inversion (DIP), single responsibility (SRP), and interface segregation (ISP).
- **Clean Code:** clear names, single-responsibility functions, low coupling, and avoiding valueless abstractions.
- **Clean Architecture / Onion:** internal policies independent of external mechanisms.
- **Screaming Architecture:** the system's purpose should be evident in its organization.
- **OWASP ASVS and OWASP Top 10:** web security verification criteria; select requirements according to risk.
- **Microsoft .NET / ASP.NET Core guidance:** idiomatic conventions, DI, configuration, error handling, and security.
- **RFC 9110 / HTTP semantics:** method and status code semantics; document the API contract.
- **OpenAPI:** describe and verify the HTTP contract when the project exposes an API.
- **Automated tests and CI:** reproducible builds and tests; do not consider a standard satisfied just because it is mentioned in the README.

Do not assign a compliance level to a standard without a specific, documented assessment.

## 11. Antipatterns this skill should detect

1. **Cosmetic Screaming:** `Features/` exists, but the rules remain scattered across global services and controllers.
2. **Onion in name only:** the projects are called Domain/Application/Infrastructure, but Application references Infrastructure.
3. **Fat controller:** queries the database, applies policies, writes files, and decides business transitions.
4. **Imposed anemic domain or forced DDD:** adding complex entities, aggregates, and events where the problem does not need them; or leaving critical invariants scattered across controllers.
5. **Ritual repositories:** one interface and one implementation per table without need, test value, or a useful boundary.
6. **Pass-through:** services that only delegate without adding a responsibility.
7. **`Common` as a catch-all:** unrelated modules end up coupled.
8. **Ceremonial duplication:** creating identical Request, Command, DTO, Model, Entity, and Response types in every case even when no boundary justifies it.
9. **Mandatory 1:1 symmetry:** forcing the frontend structure to copy the backend exactly.
10. **Confusing authentication with authorization:** the user signed in, but does not necessarily have permission to execute that operation.
11. **Big-bang migration:** moving hundreds of files at once without characterization tests or small commits.
12. **Certification by structure:** asserting that an architecture is correct only by looking at folder names.

## 12. Agent workflow protocol

When receiving a task:

1. **Clarify the goal and scope** without asking for data already present in the repository or context.
2. **Inspect evidence:** solution/projects, tree, references, paths, dependencies, tests, and existing conventions.
3. **Classify claims** as observed, inferred, or unverified.
4. **Locate the affected capability and use case.**
5. **Propose the minimum coherent change** consistent with Screaming + Onion/Clean; do not refactor the entire system for a small feature.
6. **Design names and location** before generating files.
7. **Implement following the flow** API → Application → Domain/ports; Infrastructure implements the ports, and the composition root connects them.
8. **Add or update tests** for behavior and boundaries.
9. **Run the available checks** and report exactly what happened, what failed, and what could not be executed.
10. **Deliver a summary:** affected files/capabilities, architectural decisions, tests, risks, exceptions, and the next step.

For a read-only audit, explicitly respect that boundary: do not edit, move, or delete files. Separate findings from recommendations and recommendations from actions executed.

## 13. Short architecture decision template

For each exception or non-obvious decision, document:

```text
Decision:
Context and problem:
Options considered:
Decision made:
Screaming rule applied:
Onion/Clean rule applied:
Dependency impact:
Supporting tests:
Risks / costs:
Condition to review the decision:
```

## 14. Definition of Done

A feature is considered done from this skill's perspective when:

- [ ] It lives in the right business capability.
- [ ] The controller/adapter is limited to transport and delegation.
- [ ] The use case has an action name and a clear responsibility.
- [ ] Business invariants are protected in Domain/Application, according to their nature.
- [ ] Dependencies point inward.
- [ ] Infrastructure implements the necessary contracts.
- [ ] Authorization is verified on the server and on the resource where applicable.
- [ ] DTOs and models have a clear purpose and clear names.
- [ ] Relevant tests exist and were run, or the reason they were not run was reported.
- [ ] Documentation describes the actual state, including limitations.
- [ ] No layers, interfaces, or abstractions were added without justification.

## 15. Vertical Slice decision framework

This section operationalizes R17–R24 without changing the architectural center of gravity: **business capabilities first, dependency rules always, slices when they reduce change cost**.

### 15.1 Choose the smallest coherent unit

| Question | If yes | If no |
|---|---|---|
| Is this a distinct business action or independently evolving query? | Identify an operation and owner capability | Extend the existing operation without inventing a new feature |
| Does it contain distinct authorization, validation, branching, state change, transaction, or side effects? | Prefer an Application slice | Consider a small function/query or existing use case |
| Does it need persistence, time, email, files, or other external detail? | Define a narrow inward-facing port when needed | Keep it pure; do not add interfaces ceremonially |
| Does it reuse a stable domain invariant? | Call the shared Domain behavior | Keep operation-specific decisions local |
| Is the same policy truly shared by multiple use cases? | Extract to the correct inner boundary | Avoid speculative `Shared`/`Common` abstractions |

**Decision output for an agent:** `Capability`, `Operation`, `Slice?`, `Reason`, `Layers touched`, `Ports`, `Business invariants`, `Security checks`, `Tests`, `Exceptions`. Do not require a separate decision document for every trivial edit; include the decisions in the change summary.

### 15.2 Examples of proportional slicing

- `Permissions/Authorize`: distinct authorization, transition rule, persistence → **slice recommended**.
- `Permissions/Cancel`: separate business transition and possible conflict → **slice recommended**.
- `Permissions/GetById`: a simple read with resource authorization → **lightweight query slice or narrow query service**, without needless command/handler/validator duplication.
- `Permissions/List`: paging/filtering and access scope → **query-oriented use case**; SQL translation stays in Infrastructure.
- `Users/FormatDisplayName`: trivial presentation transformation → **not a business slice**.
- `Permissions/AuthorizeAndNotify`: coordinate database commit and external notification explicitly; consider reliable delivery when needed, not a hidden call to another handler.

### 15.3 Where files belong (not all are mandatory)

```text
GestionPermisos.Api/
  Features/Permissions/
    PermissionsController.cs            # HTTP adapter; may expose multiple operations
    Http/AuthorizePermissionRequest.cs  # Transport input only if distinct
GestionPermisos.Application/
  Features/Permissions/
    Authorize/
      AuthorizePermissionCommand.cs      # Optional when input needs its own type
      AuthorizePermissionHandler.cs      # Use-case coordination
      AuthorizePermissionValidator.cs    # Optional operation-specific validation
      AuthorizePermissionResult.cs       # Optional distinct result
    GetById/
      GetPermissionByIdQuery.cs          # Lightweight read operation
      GetPermissionByIdHandler.cs
  Abstractions/Persistence/
    IPermissionRepository.cs             # Shared port only if it is the right boundary
GestionPermisos.Domain/
  Permissions/Permission.cs              # Shared invariant, not duplicated per slice
GestionPermisos.Infrastructure/
  Persistence/Repositories/
    EfPermissionRepository.cs            # Technical adapter, not a separate copy per slice
```

A controller can serve multiple slices while remaining thin. An operation can use Minimal APIs instead of controllers. A slice may have one file or several. Physical proximity is useful, but **compile-time dependency direction wins over colocation**.

### 15.4 Cross-slice communication

**Allowed:** use another slice's stable shared Domain entity/policy; consume a deliberately defined Application abstraction; coordinate two operations through a named parent workflow without invoking internal handlers as ad-hoc libraries.

**Disallowed by default:** `AuthorizePermissionHandler` importing `CancelPermissionHandler`; sharing private slice DTOs as public contracts; making two slices write the same data with incompatible invariants; using `Shared` as an unowned integration hub.

When a cross-capability dependency is necessary, define the owning capability, public contract, direction of dependency, consistency requirements, and tests. For a large system, assess module boundaries separately; slices alone do not create independent modules or microservices.

## 16. Testing matrix and triggers

### 16.1 Choose tests from observable behavior

| Change / risk | Minimum useful evidence | Typical boundary |
|---|---|---|
| Domain invariant or state transition | Allowed and forbidden transitions, boundary values | Domain unit test |
| Operation orchestration | Success, missing resource, conflict, external failure as relevant | Application test with fakes |
| Resource-specific authorization | Allowed user, forbidden user, ownership/scope cases | Application and API integration |
| Input parsing / HTTP contract | Invalid body, status codes, serialization, validation response | API integration |
| EF Core query, SQL filter, migration, transaction | Query semantics, constraints, rollback, concurrency where relevant | Infrastructure integration with compatible/real DB |
| External integration | Success, timeout, retry/idempotency where relevant | Contract/integration test |
| Vue UI flow | Loading, empty, error, success, interaction | Component/e2e test according to risk |
| Bug fix | Reproduces failure before fix and passes afterward | Closest relevant test boundary |
| Refactor | Characterization test plus existing behavior preserved | Appropriate affected boundaries |
| Architecture rule | Forbidden references/cycles and allowed composition | Automated architecture test |

Tests are **not** counted by class count or file count. Prioritize risks and contracts. Do not claim all cases are covered without evidence.

### 16.2 Per-operation test plan

Before implementation, record succinctly:

```text
Capability / operation:
Expected success behavior and output:
Inputs and validation boundaries:
Authorization and resource scope:
Domain invariants / state transitions:
Persistence / transaction / concurrency:
External dependencies and failure behavior:
Test cases by boundary (unit / integration / API / frontend):
Not applicable or deferred (with reason):
```

Use a single test file when sufficient; split by behavior when it improves navigation. For example, `AuthorizePermissionHandlerTests` may cover success, conflict, missing permission, and forbidden actor without four separate classes. Prefer naming tests `Method_Scenario_ExpectedBehavior` or the repository's established convention.

### 16.3 Testability and CI gates

At minimum, run available formatting/lint, build, unit tests, architecture checks, and targeted integration tests. CI should reject newly forbidden dependencies and failing tests. For database tests, EF InMemory is **not** proof of SQL Server or PostgreSQL behavior. Where integration environments are unavailable, explicitly report what remains unverified and how to reproduce the check.

## 17. Agent implementation protocol: operation by operation

1. **Inspect first:** branch/commit, architecture, affected feature, existing conventions, contracts, tests, and runtime constraints.
2. **Name the business action:** decide its capability owner and whether it is a distinct operation.
3. **Apply the decision framework:** choose slice or lightweight implementation; explain the choice briefly.
4. **Map responsibilities:** HTTP adapter, Application coordination, Domain invariant, Infrastructure adapter; list forbidden dependencies.
5. **Design contract and security:** input/output, validation, authentication, resource authorization, failures, and transaction boundary.
6. **Plan tests by risk:** identify tests that prove behavior; add regression/characterization tests when needed.
7. **Implement a coherent vertical change:** transport → use case → domain/ports → adapter; keep the smallest useful diff.
8. **Check coupling:** no slice-to-slice implementation imports, no new pass-through services, no accidental generic repository, no misplaced framework types.
9. **Execute verification:** build, relevant tests, integration/architecture checks when possible; report commands and actual results.
10. **Report:** files changed, business capability, decision to slice or not, architecture impact, security/transaction implications, tests executed, unverified areas, and remaining risks.

For audits, execute steps 1–5 and report findings without edits. For migrations, preserve observable behavior, migrate one bounded operation at a time, and avoid big-bang rewrites.

## 18. Anti-pattern examples and remedies

| Anti-pattern | Why it is harmful | Better approach |
|---|---|---|
| One `PermissionService` with dozens of unrelated operations | Large change surface and hard-to-isolate tests | Separate independently evolving business operations |
| A directory for every getter with five empty abstractions | Ceremony exceeds value | Lightweight query with clear boundary |
| Handler calling another handler to reuse a rule | Cross-slice implementation coupling | Extract invariant to Domain or stable policy/port to Application |
| Repository copied into each slice | Duplicates persistence behavior and configuration | Reuse narrow, appropriately owned ports/adapters |
| Every DTO duplicated across API/Application/Domain | Maintenance and mapping overhead | Separate only when boundaries or semantics differ |
| Business rules in controllers or EF configurations | Framework coupling and bypassable invariants | Put invariants in Domain, orchestration in Application |
| `Shared` contains unrelated domain services | Hidden coupling between capabilities | Keep ownership explicit and extract only stable shared semantics |
| All tests mock the database | SQL/transaction behavior remains unverified | Add real-provider integration tests for data behavior |
| Declaring vertical slices solely because folders exist | No proof of cohesion or independence | Verify dependency and change boundaries |

## 19. Extended Definition of Done and assessment rubric

A sliced operation is complete only if the existing §14 checklist holds **and**:

- [ ] The slice decision is justified by the operation's behavior and complexity.
- [ ] Its capability owner and operation name are clear.
- [ ] Its own validation, orchestration, and results are discoverable without hunting through unrelated features.
- [ ] It does not import another slice's private implementation or contract.
- [ ] Shared rules have an explicit owner and are not duplicated merely for slice isolation.
- [ ] HTTP, Application, Domain, and Infrastructure responsibilities remain correctly placed.
- [ ] Authorization and resource access are enforced at the appropriate server boundaries.
- [ ] Transactions, concurrency, and external side effects are treated according to actual risk.
- [ ] Tests are selected by behavior and risk, including negative paths where applicable.
- [ ] Build/tests/architecture checks were executed or their absence is clearly reported.

### Qualitative evaluation (not automatic certification)

Score each criterion 0–2: `0` absent/violated, `1` partially demonstrated, `2` supported by direct evidence.

| Criterion | Evidence to inspect |
|---|---|
| Business-first organization | Capabilities and use cases visible in source tree |
| Inward dependencies | Project references and namespace/architecture checks |
| Domain protection | Invariants testable without framework/DB |
| Use-case cohesion | Operation-specific logic discoverable and localized |
| Slice isolation | No direct handler-to-handler implementation dependencies |
| Proportional abstraction | No ritual layers, interfaces, or duplicated DTOs |
| Authorization/integrity | Resource scope, transaction and failure tests |
| Testing quality | Behavior/risk matrix covered with appropriate boundaries |
| Delivery discipline | Build, tests, reproducibility, truthful reporting |
| Migration maintainability | Incremental changes, preserved behavior, documented exceptions |

Total: **0–20**. Do not infer a perfect score from a written skill alone: inspect an actual implementation and test results before scoring architectural compliance.

## 20. Final rule

The goal is not "more layers", "more slices", or "fewer folders." The goal is **change locality + controlled dependencies + protected business rules + proportional complexity**.

When there is tension between a pretty structure and a real boundary, prioritize the real boundary. When an abstraction does not reduce coupling, enable testing, or express a business policy, do not add it for ceremony. When an exception is necessary, make it visible, small, and verifiable.
