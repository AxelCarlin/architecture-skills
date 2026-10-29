---
name: screaming-onion-architecture
description: Aplica una arquitectura orientada al negocio que combina Screaming Architecture para organizar por capacidades y casos de uso, Onion/Clean Architecture para controlar dependencias y Vertical Slices para mantener cada flujo cohesivo. Úsala al diseñar, implementar, revisar o refactorizar aplicaciones, especialmente .NET + Vue/TypeScript.
---

# Skill: Screaming + Onion + Clean Architecture

## 1. Misión

Ayudar a diseñar y modificar software cuya estructura revele **qué hace el negocio**, cuyos módulos sean fáciles de localizar y evolucionar, y cuyas reglas de negocio no dependan de frameworks, HTTP ni persistencia.

La integración propuesta tiene tres dimensiones complementarias:

1. **Screaming Architecture — organización:** el repositorio comunica las capacidades del negocio (`Permissions`, `Users`, `Cutoffs`) en vez de comunicar únicamente tecnologías (`Controllers`, `Services`, `Repositories`).
2. **Onion Architecture — dirección de dependencias:** el código externo depende de contratos y reglas internas; el dominio no conoce ASP.NET Core, EF Core, SQL Server, Vue ni detalles de infraestructura.
3. **Clean Architecture — límites y políticas:** separa entidades/reglas, casos de uso, adaptadores y frameworks; mantiene las dependencias apuntando hacia las políticas internas.
4. **Vertical Slice — unidad de cambio opcional:** dentro de una capacidad, organiza el código por flujo/caso de uso cuando eso mejora la cohesión (`AuthorizePermission`, `CancelPermission`). No es sinónimo de las otras tres arquitecturas ni obliga a duplicar toda la estructura en cada slice.

**Regla de síntesis:** organiza primero por negocio; dentro de cada capacidad, agrupa por caso de uso cuando sea útil; conserva los límites de dependencias de Onion/Clean.

## 2. Cuándo utilizar esta skill

Actívala cuando el usuario solicite:
- diseñar la arquitectura de una aplicación nueva;
- crear módulos, features, endpoints o casos de uso;
- evaluar una arquitectura existente;
- refactorizar proyectos por capas hacia módulos de negocio;
- definir nombres, carpetas, proyectos, contratos o reglas de dependencias;
- revisar si una implementación respeta Screaming, Onion o Clean Architecture.

Antes de proponer cambios, determina si el trabajo es diseño nuevo, feature nueva, auditoría o migración. En sistemas existentes, inspecciona evidencia real antes de recomendar movimientos.

## 3. Objetivos verificables

Una solución cumple esta skill cuando:

- El árbol de carpetas revela las capacidades principales del producto.
- Las reglas de negocio importantes pueden probarse sin arrancar HTTP, base de datos ni infraestructura.
- Los casos de uso expresan una acción de negocio y coordinan el flujo sin convertirse en una segunda capa de controladores.
- La infraestructura implementa contratos requeridos por la aplicación; la aplicación no importa la infraestructura.
- Los controladores son adaptadores del protocolo HTTP, no el lugar habitual de las reglas de negocio.
- Los DTOs de entrada/salida no se confunden automáticamente con entidades de dominio o entidades de persistencia.
- Los módulos comparten código solo cuando existe una responsabilidad realmente compartida.
- Las excepciones arquitectónicas se documentan y prueban.
- Los cambios se verifican con pruebas y, cuando corresponda, análisis automático de dependencias.

No se exige que todo proyecto tenga muchos proyectos .NET, repositorios genéricos, CQRS, MediatR, Domain Events o una interfaz por clase. La complejidad debe responder a necesidades reales.

## 4. Vocabulario y límites conceptuales

### Screaming Architecture

Pregunta: **¿Dónde encuentro el comportamiento del negocio?**

Organiza el código para que un desarrollador identifique rápidamente permisos, usuarios, cortes, áreas, reportes, etc. Favorece cohesión y localidad de cambio.

No determina por sí sola qué proyecto puede referenciar a cuál. Tener una carpeta `Features/Permissions` no convierte automáticamente el sistema en una arquitectura modular.

### Onion Architecture

Pregunta: **¿En qué dirección viajan las dependencias?**

El dominio ocupa el centro. La aplicación depende del dominio. La infraestructura implementa los contratos definidos hacia dentro. La API y el composition root conectan las piezas.

Las dependencias de compilación deben apuntar hacia políticas internas, aunque el flujo de ejecución pueda llamar desde un caso de uso a una implementación externa a través de una interfaz.

### Clean Architecture

Pregunta: **¿Qué políticas son internas y qué detalles deben poder sustituirse?**

Refuerza la separación entre:
- **Entities / Domain:** reglas empresariales y conceptos del dominio.
- **Use Cases / Application:** intención, coordinación y reglas específicas de cada operación.
- **Interface Adapters:** traducción entre HTTP/JSON, modelos de aplicación y persistencia.
- **Frameworks & Drivers / Infrastructure:** ASP.NET Core, EF Core, SQL Server, correo, archivos y servicios externos.

Onion y Clean comparten la regla esencial de inversión de dependencias. No son etiquetas que deban implementarse con una plantilla rígida idéntica.

### Vertical Slice Architecture

Pregunta: **¿Cómo mantengo junto lo que cambia para completar un caso de uso?**

Una slice agrupa el código propio de un flujo, por ejemplo `Permissions/Authorize`. Puede contener request, handler/use case, validator y response. La persistencia y el dominio compartidos no se duplican solo para que cada slice parezca independiente.

**No afirmar:** “Screaming + Onion automáticamente es Vertical Slice”. La combinación permite slices, pero son decisiones relacionadas y distintas.

## 5. Arquitectura objetivo

### 5.1 Opción recomendada para .NET con límites explícitos

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

La estructura exacta debe adaptarse al tamaño del sistema. Si la solución no necesita cuatro ensamblados, puede conservar menos proyectos y aplicar los mismos límites mediante namespaces, referencias controladas y pruebas arquitectónicas. Separar proyectos es útil cuando convierte las reglas en restricciones de compilación; no es un fin estético.

### 5.2 Mapa de responsabilidades

| Parte | Responsabilidad | No debe hacer |
|---|---|---|
| `Api` | HTTP, autenticación del transporte, autorización de endpoints, model binding, códigos HTTP, composición y DI | Contener reglas centrales del permiso o ejecutar consultas de negocio complejas directamente |
| `Application` | Casos de uso, coordinación, validación de operación, contratos necesarios y resultados | Depender de `Infrastructure`, `DbContext`, `IActionResult` o tipos HTTP |
| `Domain` | Entidades, objetos de valor, invariantes y reglas del negocio | Importar EF Core, ASP.NET Core, JSON, archivos o servicios externos |
| `Infrastructure` | Implementar persistencia, correo, archivos, reloj real y otros adaptadores externos | Decidir políticas de negocio que deberían estar en Domain/Application |
| `Frontend/features` | Interfaz y flujos de usuario agrupados por capacidad | Convertirse en fuente de autoridad para permisos de seguridad |
| `Shared` / `Common` | Tipos realmente transversales, estables y pequeños | Ser un vertedero de helpers, DTOs o lógica que pertenece a una feature |

### 5.3 Dirección permitida de dependencias

```text
API ───────────────► Application ───────────────► Domain
 │                         ▲
 │                         │ implementa contratos
 └────► Infrastructure ────┴────────────────────► Domain
```

Interpretación:
- `Application` puede declarar `IPermissionRepository`.
- `Infrastructure` implementa `IPermissionRepository` con EF Core.
- `Application` no referencia el proyecto `Infrastructure`.
- `Api` o el composition root registra `IPermissionRepository -> PermissionRepository`.
- `Domain` no conoce a ninguno de los proyectos exteriores.

Dependencias de proyecto sugeridas:

| Proyecto | Puede referenciar |
|---|---|
| `Domain` | BCL y paquetes mínimos realmente neutrales |
| `Application` | `Domain` |
| `Infrastructure` | `Application`, `Domain` y librerías técnicas necesarias |
| `Api` | `Application`, `Infrastructure` para composición y, si hace falta, `Domain` para mapeo explícito |
| Tests de Domain | `Domain` |
| Tests de Application | `Application`, `Domain` y dobles de prueba |
| Tests de Infrastructure | `Infrastructure` y proveedores de prueba |
| Tests de API | `Api` y herramientas de integración |

Evitar ciclos. La tabla es una política inicial, no una obligación de que todo el mundo necesite los mismos ensamblados.

## 6. Reglas normativas de la skill

Las etiquetas significan:
- **MUST / Obligatorio:** regla por defecto; incumplir requiere una excepción explícita.
- **SHOULD / Recomendado:** aplicar salvo que exista una razón concreta.
- **MAY / Opcional:** utilizar si añade valor.

### R1 — Organizar por capacidad de negocio (MUST)

Preferir `Features/Permissions`, `Features/Users`, `Features/Cutoffs` sobre carpetas globales que dispersan una sola capacidad entre `Controllers/`, `Services/`, `Repositories/` y `DTOs/`.

Las carpetas técnicas globales siguen siendo válidas para verdaderos límites técnicos: `Middleware`, `Persistence`, `Authentication`, `Observability`.

### R2 — Dirección de dependencias (MUST)

Las reglas de dominio no deben depender de detalles externos. La aplicación declara las abstracciones que necesita; infraestructura las implementa.

No introducir referencias de `Domain` o `Application` a `Infrastructure` para resolver una comodidad de implementación.

### R3 — Dominio puro por defecto (MUST)

Las entidades de dominio no deben depender de:
- `Microsoft.AspNetCore.*`
- `Microsoft.EntityFrameworkCore.*`
- DTOs de HTTP, `IActionResult`, `HttpContext`
- clientes SQL, correo, sistema de archivos o UI.

Las anotaciones de persistencia y el mapeo de EF deben vivir en configuraciones de Infrastructure cuando sea práctico. No se prohíbe cada paquete externo de forma dogmática; cualquier dependencia del dominio debe justificarse por estabilidad y valor.

### R4 — Casos de uso explícitos (SHOULD)

Si una operación coordina reglas, autorización de negocio, varias escrituras, transacciones, correo o archivos, representarla como caso de uso en Application.

Ejemplos: `AuthorizePermission`, `CancelPermission`, `ImportCutoffs`, `RegisterUser`.

Para un CRUD trivial de lectura, no crear una cadena de cinco clases que solo reenvían parámetros. Una consulta sencilla puede tener un query handler ligero o un adaptador de lectura, pero no debe saltarse los límites de dependencias.

### R5 — Sin servicios pasamanos (MUST)

No crear `IPermissionService` + `PermissionService` si la clase únicamente llama a otra clase y no agrega política, coordinación, validación, transformación ni un límite útil.

La ausencia de un servicio pasamanos **no** autoriza a colocar toda la lógica en el controller. El endpoint debe seguir siendo un adaptador fino; la lógica significativa va al caso de uso o dominio.

### R6 — Contratos donde se invierte una dependencia (SHOULD)

Crear interfaces para límites que necesitan ser sustituidos, probados o implementados por un anillo exterior: persistencia, reloj, envío de correo, almacenamiento de archivos o proveedor externo.

No crear una interfaz para cada clase por costumbre. El contrato debe pertenecer a la capa interna que necesita el servicio, no a la capa concreta que lo implementa.

### R7 — DTOs con propósito claro (MUST)

Distinguir por intención, no solo por sufijo:
- `AuthorizePermissionRequest`: entrada HTTP.
- `AuthorizePermissionCommand`: intención de aplicación.
- `AuthorizePermissionResult`: resultado del caso de uso.
- `PermissionResponse`: salida HTTP.
- `Permission`: entidad de dominio.
- `PermissionEntity` solo si realmente existe un modelo de persistencia separado y la convención lo necesita.

Se puede reutilizar un DTO entre límites cuando es estable y no filtra detalles de transporte o persistencia. No duplicar tipos idénticos sin una razón.

### R8 — Validación en la capa correcta (MUST)

- Formato y estructura HTTP: API/model binding.
- Reglas de la operación y permisos sobre el recurso: Application.
- Invariantes que siempre deben cumplirse: Domain.
- Restricciones de integridad: también en la base de datos cuando corresponda.

Una validación frontend mejora la experiencia, pero nunca reemplaza la validación de servidor.

### R9 — Seguridad independiente de la organización (MUST)

- Cada endpoint debe tener autenticación y autorización explícitas.
- Comprobar roles/policies y acceso al recurso; estar autenticado no significa estar autorizado.
- No confiar en que la UI o una API key por sí sola protejan una operación.
- No registrar secretos, tokens, contraseñas, datos personales innecesarios ni cadenas de conexión.
- Usar algoritmos de hashing de contraseñas apropiados y sal; no SHA-256 simple para contraseñas.
- Verificar CORS, antiforgery cuando aplique, gestión de secretos y manejo seguro de errores.
- Los cambios arquitectónicos no se consideran una corrección de seguridad hasta que existan pruebas que la demuestren.

### R10 — Transacciones e integridad (MUST cuando aplique)

Un caso de uso que requiere varias escrituras atómicas debe definir el límite transaccional. La implementación de transacción pertenece a Infrastructure o a un contrato de Application implementado por Infrastructure. No asumir que varias llamadas a repositorio son atómicas.

### R11 — Errores y resultados explícitos (SHOULD)

Los casos de uso deben devolver resultados tipados o lanzar excepciones de aplicación/domain bien definidas según la convención del proyecto. La API traduce esos resultados a HTTP (`200`, `201`, `400`, `403`, `404`, `409`, etc.) y `ProblemDetails` cuando sea apropiado. No filtrar stack traces al cliente.

### R12 — Nombres precisos y consistentes (MUST)

Usar términos del negocio, verbos para acciones y nombres que expliquen intención. Evitar nombres vagos (`Manager`, `Helper`, `CommonService`, `ProcessData`, `DoStuff`) salvo que el dominio justifique el término.

### R13 — Simetría frontend/backend sin acoplamiento rígido (SHOULD)

Mantener correspondencia conceptual, no necesariamente una estructura 1:1. Por ejemplo, `Frontend/src/features/permissions` consume endpoints de permisos, pero puede dividir componentes por flujo UX. No hacer que el frontend importe tipos internos de Domain o de EF.

### R14 — Shared bajo control (MUST)

Mover código a `Shared` solo si lo usan varios módulos y su semántica es realmente común. No mover una función a `Shared` solo porque se usa dos veces. No crear `Common` como destino por defecto de todo lo que no encaja.

### R15 — Excepciones explícitas (MUST)

Una desviación debe documentar: motivo, alcance, riesgo, alternativa descartada y condición para reevaluarla. Las excepciones no se convierten en patrón por copiar y pegar.

### R16 — Verificación antes de afirmar (MUST)

Separar en informes:
- **Observado:** evidencia directa del código/configuración.
- **Inferido:** conclusión probable, no comprobada.
- **No verificado:** falta evidencia o ejecución.
No afirmar que las pruebas pasan si no se ejecutaron. No afirmar que un sistema cumple Clean/Onion solo porque tiene proyectos con esos nombres.

## 7. Nomenclatura recomendada para C# y TypeScript

La nomenclatura debe respetar las convenciones ya establecidas en el repositorio si son coherentes. En un proyecto nuevo:

| Elemento | Convención | Ejemplo |
|---|---|---|
| Proyecto/namespace | `PascalCase` | `GestionPermisos.Application` |
| Clase, record, interfaz, enum | `PascalCase` | `AuthorizePermissionHandler`, `IPermissionRepository` |
| Método/propiedad pública | `PascalCase` | `AuthorizeAsync`, `PermissionStatus` |
| Parámetro/variable local | `camelCase` | `permissionId`, `currentUser` |
| Campo privado | `_camelCase` | `_permissionRepository` |
| Constantes | `PascalCase` en C# | `MaximumEvidenceSize` |
| Caso de uso | Verbo + objeto | `AuthorizePermission`, `CancelPermission` |
| Puerto/interfaz | `I` + capacidad | `IPermissionRepository`, `IClock` |
| Implementación | Nombre concreto | `EfPermissionRepository`, `SystemClock` |
| Entrada HTTP | Acción + `Request` | `AuthorizePermissionRequest` |
| Comando | Acción + `Command` | `AuthorizePermissionCommand` |
| Query | Acción/consulta + `Query` | `GetPermissionByIdQuery` |
| Resultado | Acción + `Result` | `AuthorizePermissionResult` |
| Respuesta HTTP | Objeto + `Response` | `PermissionResponse` |
| Validador | Tipo + `Validator` | `AuthorizePermissionValidator` |
| Prueba | Tipo/acción + comportamiento | `AuthorizePermissionHandlerTests` |
| Vue component | `PascalCase.vue` | `PermissionDetailsView.vue` |
| TypeScript functions/variables | `camelCase` | `authorizePermission()` |
| TypeScript types/interfaces | `PascalCase` | `PermissionResponse` |

No introducir sufijos `Impl`, `Manager` o `Helper` automáticamente. `Repository` se reserva para una abstracción/implementación de persistencia con una responsabilidad definida; no todo acceso a datos necesita un repositorio genérico.

## 8. Ejemplo completo: autorizar un permiso laboral

### 8.1 Flujo

```text
HTTP POST /api/permissions/{id}/authorize
    ↓
PermissionsController (API: valida transporte y traduce HTTP)
    ↓
AuthorizePermissionHandler (Application: coordina el caso de uso)
    ├── IPermissionRepository (contrato interno)
    ├── política/regla de dominio (Permission.Authorize)
    └── IUnitOfWork (si la operación necesita una transacción)
             ↑ implementaciones
    EfPermissionRepository + EfUnitOfWork (Infrastructure)
             ↓
        SQL Server / EF Core
```

La API no debe llamar directamente a EF Core para decidir si un permiso se puede autorizar. Application coordina el flujo y Domain protege las invariantes.

### 8.2 Dominio

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

    private Permission() { } // Para mapeo cuando la configuración lo requiera.

    public Permission(Guid id)
    {
        Id = id;
        Status = PermissionStatus.Pending;
    }

    public void Authorize()
    {
        if (Status != PermissionStatus.Pending)
            throw new InvalidPermissionTransitionException(
                "Solo se pueden autorizar permisos pendientes.");

        Status = PermissionStatus.Authorized;
    }
}
```

`InvalidPermissionTransitionException` sería una excepción de dominio definida en el proyecto Domain. Este ejemplo es ilustrativo: las reglas reales deben confirmarse con el negocio; no asumir que todos los permisos comparten este flujo de estados.

### 8.3 Contrato en Application

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

El contrato está en Application porque es el caso de uso quien necesita persistir permisos. Infrastructure lo implementará.

### 8.4 Caso de uso en Application

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

        // La autorización de identidad/recurso debe comprobarse aquí
        // o mediante una política de aplicación explícita.
        // No basta con que el usuario esté autenticado.

        permission.Authorize();

        _permissions.Update(permission);
        await _unitOfWork.SaveChangesAsync(cancellationToken);

        return new AuthorizePermissionResult(
            permission.Id,
            permission.Status.ToString());
    }
}
```

El ejemplo omite deliberadamente la implementación de `IUnitOfWork`, las excepciones auxiliares y la comprobación completa de autorización para mantener visible el límite arquitectónico. No debe copiarse a producción sin implementar esos requisitos. En particular, `AuthorizedByUserId` no prueba por sí mismo que ese usuario tenga permiso para autorizar.

### 8.5 Adaptador de infraestructura

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

El mapeo entre dominio y persistencia depende de cómo se haya diseñado el modelo. Si las entidades de dominio se persisten directamente con EF Core, la configuración EF se mantiene fuera de Domain. Si se usan modelos de persistencia separados, Infrastructure realiza el mapeo explícito.

### 8.6 Adaptador HTTP

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
        var userId = GetAuthenticatedUserId(); // Adaptador de identidad, validado.
        var result = await _authorize.HandleAsync(
            new AuthorizePermissionCommand(id, userId),
            cancellationToken);

        return Ok(new PermissionResponse(result.PermissionId, result.Status));
    }
}
```

El método `GetAuthenticatedUserId`, la política de autorización y el mapeo de excepciones son puntos ilustrativos que deben implementarse de forma explícita. La política HTTP no reemplaza la autorización del recurso dentro del caso de uso cuando el acceso depende del estado o propietario del permiso.

## 9. Aplicación al sistema `itsmRH_GestionPermisos`

### 9.1 Punto de partida observado en el informe

El informe de auditoría describe un monolito web .NET con SPA Vue, proyectos `Core`, `Data`, `Services`, `Crosscutting`, API y tests. Observa que:
- `Core` mezcla entidades, DTOs, ViewModels, enums e interfaces.
- `Services` depende de `Data` y `Crosscutting`.
- `Crosscutting` también depende de `Data`.
- Los controladores realizan algunas tareas de orquestación y acceso a datos.
- Hay features en presentación, pero no límites modulares completos.
- La suite de pruebas descrita no cubre todos los controladores/filtros/auth/E2E.
- La auditoría reporta riesgos de seguridad que deben tratarse como trabajo separado, no como consecuencia automática de cambiar carpetas.

Estos puntos son la línea base del informe recibido, no una certificación independiente del estado actual del repositorio. Antes de cambiarlo, volver a inspeccionar rama, commit y código vigente.

### 9.2 Traducción gradual, no reescritura masiva

| Área observada | Destino conceptual | Estrategia prudente |
|---|---|---|
| API/controllers | `GestionPermisos.Api/Features/...` | Mantenerlos como adaptadores HTTP; extraer lógica de negocio al migrar cada flujo |
| `Core` | `Domain` + contratos/DTOs de `Application` | Clasificar cada archivo por responsabilidad; no mover todo `Core` al Domain |
| `Services` | `Application/Features/...` para casos de uso; `Infrastructure` para integraciones | Separar caso por caso según dependencias reales |
| `Data` | `Infrastructure/Persistence` | Mantener EF Core y `DbContext` fuera de Domain/Application |
| `Crosscutting` | API, Application o Infrastructure según la función | No mover todo como bloque: auth HTTP, logging, hashing y configuración tienen dueños distintos |
| Tests | Domain/Application/Infrastructure/API tests | Agregar pruebas de regresión antes y durante cada migración |
| Vue `features` | `src/features/permissions`, `users`, `cutoffs`, etc. | Conservar la organización por negocio y evitar importar modelos internos del backend |

**No hacer:** borrar `Services`, `Core` o `Crosscutting` en una sola operación; renombrar carpetas y declarar que se logró Onion; mover todas las clases de `Core` a Domain sin clasificación; introducir un repositorio genérico para cada tabla sin comprobar que aporta valor.

### 9.3 Orden de migración sugerido

1. **Congelar línea base:** registrar commit, build, pruebas ejecutadas y limitaciones. No editar hasta entender el estado actual.
2. **Inventariar dependencias:** construir grafo real de referencias entre proyectos y mapa de llamadas de una feature.
3. **Elegir un flujo acotado:** por ejemplo, consultar o autorizar un permiso, según complejidad y riesgo.
4. **Agregar pruebas de caracterización:** capturar comportamiento actual antes de extraer lógica.
5. **Definir contrato de Application:** declarar solo los puertos necesarios para ese caso de uso.
6. **Extraer caso de uso:** mover coordinación fuera del controller sin cambiar el comportamiento observable.
7. **Separar dominio con cuidado:** llevar una regla al Domain solo si es una regla de negocio real, no un simple método utilitario.
8. **Implementar adaptadores:** infraestructura implementa los contratos; API registra dependencias en el composition root.
9. **Verificar:** build, pruebas unitarias, integración con el proveedor real cuando corresponda y pruebas API/autorización.
10. **Repetir por feature:** migración incremental, con commits pequeños y rollback claro.
11. **Automatizar límites:** impedir referencias prohibidas mediante tests de arquitectura, reglas de solución o análisis de dependencias.
12. **Actualizar documentación:** registrar decisiones, excepciones y lo realmente verificado.

La prioridad y el orden de las features deben definirse después de revisar riesgo, dependencias, frecuencia de cambio y cobertura. No asumir que permisos es automáticamente el primer módulo correcto.

## 10. Pruebas y controles de calidad

### Pirámide mínima por límite

- **Domain tests:** invariantes, transiciones de estado y reglas sin infraestructura.
- **Application tests:** casos de uso con dobles de los puertos; éxito, ausencia de recurso, conflicto, fallo de dependencia y autorización.
- **Infrastructure integration tests:** EF Core y el motor de base de datos real o equivalente compatible; no confiar solo en `EF InMemory` para validar SQL, transacciones o restricciones.
- **API tests:** rutas, serialización, códigos HTTP, autenticación, autorización y manejo de errores.
- **Frontend tests:** estados de carga/error, permisos de UI y flujos principales; la seguridad sigue siendo responsabilidad del backend.

### Checklist de dependencias

- [ ] `Domain` no referencia API, Infrastructure ni EF Core.
- [ ] `Application` no referencia API ni Infrastructure.
- [ ] Infrastructure implementa interfaces internas.
- [ ] No hay ciclos entre proyectos.
- [ ] El composition root registra las implementaciones.
- [ ] Los tests no dependen innecesariamente de servicios externos.
- [ ] Se prueban autorización y acceso a recursos en el servidor.
- [ ] Se registran comandos exactos ejecutados y resultados reales.

### Estándares y guías relevantes

Usar estos referentes como guías de diseño y calidad, no como una certificación automática:

- **SOLID:** especialmente inversión de dependencias (DIP), responsabilidad única (SRP) y segregación de interfaces (ISP).
- **Clean Code:** nombres claros, funciones con una responsabilidad, bajo acoplamiento y evitar abstracciones sin valor.
- **Clean Architecture / Onion:** políticas internas independientes de mecanismos externos.
- **Screaming Architecture:** el propósito del sistema debe ser evidente en la organización.
- **OWASP ASVS y OWASP Top 10:** criterios de verificación de seguridad web; seleccionar requisitos acordes al riesgo.
- **Microsoft .NET / ASP.NET Core guidance:** convenciones idiomáticas, DI, configuración, manejo de errores y seguridad.
- **RFC 9110 / HTTP semantics:** semántica de métodos y códigos HTTP; documentar el contrato API.
- **OpenAPI:** describir y comprobar el contrato HTTP cuando el proyecto exponga una API.
- **Pruebas automatizadas y CI:** build y pruebas reproducibles; no considerar un estándar satisfecho solo por mencionarlo en el README.

No atribuir un nivel de cumplimiento a un estándar sin hacer una evaluación específica y documentada.

## 11. Antipatrones que esta skill debe detectar

1. **Screaming cosmético:** existe `Features/`, pero las reglas siguen dispersas en servicios globales y controllers.
2. **Onion de nombre:** los proyectos se llaman Domain/Application/Infrastructure, pero Application referencia Infrastructure.
3. **Controller gordo:** consulta DB, aplica políticas, escribe archivos y decide transiciones de negocio.
4. **Dominio anémico impuesto o DDD forzado:** añadir entidades complejas, agregados y eventos donde el problema no los necesita; o dejar invariantes críticas repartidas por controllers.
5. **Repositorios rituales:** una interfaz y una implementación por tabla sin necesidad, valor de prueba ni frontera útil.
6. **Pasamanos:** servicios que solo delegan sin agregar una responsabilidad.
7. **`Common` como cajón de sastre:** módulos no relacionados terminan acoplados.
8. **Duplicación ceremonial:** crear Request, Command, DTO, Model, Entity y Response idénticos en todos los casos aunque no haya una frontera que lo justifique.
9. **Simetría 1:1 obligatoria:** forzar la estructura del frontend a copiar exactamente la del backend.
10. **Confundir autenticación con autorización:** el usuario inició sesión, pero no necesariamente tiene permiso para ejecutar esa operación.
11. **Migración big-bang:** mover cientos de archivos a la vez sin pruebas de caracterización ni commits pequeños.
12. **Certificación por estructura:** asegurar que una arquitectura es correcta solo por ver nombres de carpetas.

## 12. Protocolo de trabajo del agente

Al recibir una tarea:

1. **Aclarar el objetivo y el alcance** sin preguntar por datos que ya estén en el repositorio o en el contexto.
2. **Inspeccionar evidencia**: solución/proyectos, árbol, referencias, rutas, dependencias, pruebas y convenciones existentes.
3. **Clasificar afirmaciones** como observado, inferido o no verificado.
4. **Localizar la capacidad y el caso de uso** afectados.
5. **Proponer el cambio mínimo coherente** con Screaming + Onion/Clean; no refactorizar el sistema entero para una feature pequeña.
6. **Diseñar nombres y ubicación** antes de generar archivos.
7. **Implementar siguiendo el flujo** API → Application → Domain/ports; Infrastructure implementa los ports y el composition root conecta.
8. **Añadir o actualizar pruebas** del comportamiento y de los límites.
9. **Ejecutar las verificaciones disponibles** y reportar exactamente qué pasó, qué falló y qué no se pudo ejecutar.
10. **Entregar resumen**: archivos/capacidades afectados, decisiones arquitectónicas, pruebas, riesgos, excepciones y siguiente paso.

Para una auditoría de solo lectura, respetar explícitamente ese límite: no editar, mover ni borrar archivos. Separar hallazgos de recomendaciones y recomendaciones de acciones ejecutadas.

## 13. Plantilla breve de decisión arquitectónica

Para cada excepción o decisión no obvia, documentar:

```text
Decisión:
Contexto y problema:
Opciones consideradas:
Decisión tomada:
Regla Screaming aplicada:
Regla Onion/Clean aplicada:
Impacto en dependencias:
Pruebas que la respaldan:
Riesgos / costes:
Condición para revisar la decisión:
```

## 14. Definition of Done

Una feature se considera terminada desde esta skill cuando:

- [ ] Vive en la capacidad de negocio adecuada.
- [ ] El controller/adaptador se limita al transporte y delegación.
- [ ] El caso de uso tiene nombre de acción y una responsabilidad clara.
- [ ] Las invariantes de negocio están protegidas en Domain/Application, según su naturaleza.
- [ ] Las dependencias apuntan hacia el interior.
- [ ] La infraestructura implementa los contratos necesarios.
- [ ] La autorización se verifica en el servidor y sobre el recurso cuando corresponde.
- [ ] DTOs y modelos tienen propósito y nombres claros.
- [ ] Existen pruebas relevantes y se ejecutaron o se reportó por qué no.
- [ ] La documentación describe el estado real, incluidas las limitaciones.
- [ ] No se añadieron capas, interfaces ni abstracciones sin justificación.

## 15. Regla final

La meta no es “tener más capas” ni “tener menos carpetas”. La meta es **localidad de cambio + dependencias controladas + reglas de negocio protegidas + complejidad proporcional**.

Cuando haya tensión entre una estructura bonita y una frontera real, prioriza la frontera real. Cuando una abstracción no reduzca acoplamiento, no habilite pruebas ni exprese una política del negocio, no la agregues por ceremonia. Cuando una excepción sea necesaria, hazla visible, pequeña y verificable.
