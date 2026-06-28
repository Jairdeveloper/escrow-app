# Use Case Framework — Especificación Técnica

**Versión:** 1.0  
**Propósito:** Framework ágnostico para la definición, orquestación y ejecución de casos de uso. Independiente de lenguaje, framework web, base de datos o infraestructura.  
**Relación con ARQUITECTURA.md:** Este framework es el motor que ejecuta los 66 casos de uso definidos en la Sección 4.

---

## Índice

1. [Concepto y Principios](#1-concepto-y-principios)
2. [Arquitectura del Framework](#2-arquitectura-del-framework)
3. [Abstracciones Core](#3-abstracciones-core)
4. [Ciclo de Vida de un Caso de Uso](#4-ciclo-de-vida-de-un-caso-de-uso)
5. [Pipeline de Ejecución](#5-pipeline-de-ejecución)
6. [Behaviors (Cross-Cutting Concerns)](#6-behaviors-cross-cutting-concerns)
7. [Composición de Casos de Uso](#7-composición-de-casos-de-uso)
8. [Manejo de Errores y Resultados](#8-manejo-de-errores-y-resultados)
9. [Eventos de Dominio](#9-eventos-de-dominio)
10. [Puertos y Adaptadores (Ports & Adapters)](#10-puertos-y-adaptadores-ports--adapters)
11. [Autorización y Seguridad](#11-autorización-y-seguridad)
12. [Validación](#12-validación)
13. [Transacciones y Consistencia](#13-transacciones-y-consistencia)
14. [Pruebas](#14-pruebas)
15. [Extensibilidad y Puntos de Extensión](#15-extensibilidad-y-puntos-de-extensión)
16. [Glosario del Framework](#16-glosario-del-framework)

---

# 1. Concepto y Principios

## 1.1 ¿Qué es un Use Case Framework?

Un Use Case Framework es una infraestructura reutilizable que estandariza cómo se definen, validan, autorizan, ejecutan, componen y observan los casos de uso de una aplicación. Su objetivo es eliminar la duplicación de código transversal (cross-cutting) y garantizar que cada caso de uso se implemente siguiendo un contrato predecible y testeable.

## 1.2 Principios de Diseño

| Principio | Significado |
|---|---|
| **Ágnóstico de tecnología** | El framework no depende de ningún framework web, ORM, base de datos, sistema de colas o proveedor cloud. Toda interacción con el mundo exterior ocurre mediante puertos (interfaces) que la aplicación cliente implementa. |
| **Separación de responsabilidades** | Cada caso de uso se expresa en términos del dominio, no de la infraestructura. El caso de uso no sabe qué framework web lo invoca ni qué base de datos persiste sus datos. |
| **Componible** | Los casos de uso pueden combinarse, encadenarse y orquestarse sin modificar su implementación interna. |
| **Transparente** | Cada caso de uso atraviesa un pipeline predecible de behaviors (validación, autorización, logging, transacciones, eventos) sin que el código del caso de uso deba ocuparse de ello. |
| **Testeable por construcción** | Todo caso de uso se prueba en aislamiento inyectando dependencias falsas (puertos mockeados). |
| **Extensible sin modificación** | Nuevos behaviors, puertos o políticas se agregan sin tocar el código existente (Open/Closed Principle). |

## 1.3 Relación con la Arquitectura Hexagonal

Este framework implementa la **capa de aplicación** (application layer) de la Arquitectura Hexagonal (Ports & Adapters) y de Clean Architecture:

```
┌─────────────────────────────────────────────────────────┐
│                   Capa de Infraestructura                │
│  (Web controllers, CLI, DB adapters, Message consumers)  │
├─────────────────────────────────────────────────────────┤
│              Capa de Aplicación (USE CASE LAYER)          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │              USE CASE FRAMEWORK                      │ │
│  │  Pipeline · Behaviors · Validation · Authorization  │ │
│  │  ┌─────────────────────────────────────────────────┐│ │
│  │  │  CasoDeUsoX · CasoDeUsoY · CasoDeUsoZ          ││ │
│  │  └─────────────────────────────────────────────────┘│ │
│  └─────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────┤
│                   Capa de Dominio                         │
│  (Entidades, Value Objects, Aggregate Roots, Eventos)    │
└─────────────────────────────────────────────────────────┘
```

---

# 2. Arquitectura del Framework

## 2.1 Diagrama de Componentes

```
┌──────────────────────────────────────────────────────────────────┐
│                        USE CASE FRAMEWORK                         │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                    UseCaseBus (Mediator)                      │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐ │ │
│  │  │ Send()   │  │ Publish()│  │ Execute()│  │  Pipeline()  │ │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────┬───────┘ │ │
│  └──────────────────────────────────────────────────────┬───────┘ │
│                                                         │         │
│  ┌──────────────────────────────────────────────────────▼───────┐ │
│  │                    Pipeline de Behaviors                      │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐ │ │
│  │  │ Logging  │→│Validation│→│ Authz    │→│ Transaction   │ │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────┬───────┘ │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────▼───────┐ │ │
│  │  │ Metrics  │←│ Events   │←│ Domain   │←│ UseCase       │ │ │
│  │  │          │  │ Publish  │  │ Handler  │  │ Executor     │ │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────────┘ │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                    │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐ │ │
│  │ Command    │  │ Query      │  │ UseCase    │  │ Domain     │ │ │
│  │ Bus        │  │ Bus        │  │ Composer   │  │ Event Bus  │ │ │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘ │ │
└──────────────────────────────────────────────────────────────────┘
```

## 2.2 Diferenciación: Command vs Query vs UseCase

| Tipo | Mutación | Retorno | Validez | Casos de Uso Típicos |
|---|---|---|---|---|
| **Command** | Sí (escribe) | Void o ID del recurso creado | Siempre debe ejecutarse en una transacción | UC-01, UC-02, UC-13, UC-17, UC-18 |
| **Query** | No (solo lee) | Datos (DTO) | Nunca en transacción; puede cachearse | UC-10, UC-25, UC-38, UC-39, UC-50 |
| **UseCase** | Sí o No | Resultado genérico | Orquesta commands + queries + reglas de negocio | UC-16, UC-24, UC-28, UC-33 |

## 2.3 Interfaces Públicas del Framework

El framework expone las siguientes interfaces para que el cliente las implemente:

```
UseCaseBus:
  - send(command: Command): Result
  - query(query: Query): Result
  - execute(useCase: UseCase, input: Input): Result
  - publish(events: DomainEvent[]): void

Pipeline:
  - addBehavior(behavior: Behavior): void
  
UseCaseRegistry:
  - register(useCaseType: Type, handler: Handler): void
  - resolve(useCaseType: Type): Handler
```

---

# 3. Abstracciones Core

## 3.1 Command

Un Command es un objeto inmutable que representa la intención de ejecutar una acción que muta el estado del sistema. No contiene lógica, solo datos.

```
Command {
  - id: string (UUID generado por el framework)
  - timestamp: DateTime (generado por el framework)
  - correlationId: string (para trazabilidad distribuida)
  - causatonId?: string (para encadenar comandos)
  + data properties (definidas por cada command concreto)
}
```

**Reglas**:
- Todo Command debe tener un ID único (UUID v7) y timestamp generados automáticamente.
- Todo Command debe llevar un `correlationId` para tracing.
- Los datos del Command son inmutables (read-only después de la creación).
- Los Command se serializan/deserializan sin pérdida (necesario para colas y event sourcing).

## 3.2 Query

Una Query representa la intención de obtener datos sin mutar el estado.

```
Query {
  - id: string
  - timestamp: DateTime
  - correlationId: string
  + filters: Map<string, Filter>
  + pagination?: Pagination
  + sort?: Sort[]
  + fields?: string[] (projection / sparse fields)
}
```

**Reglas**:
- Las Query nunca producen efectos secundarios.
- Las Query pueden cachearse (el framework puede tener un behavior de caching opcional).
- El paginado debe ser cursor-based para colecciones grandes.

## 3.3 Handler

Un Handler es la unidad de ejecución que procesa un Command, Query o UseCase. Cada handler implementa exactamente una de estas interfaces:

```
CommandHandler<C extends Command, R> {
  handle(command: C, context: ExecutionContext): Promise<Result<R>>
}

QueryHandler<Q extends Query, R> {
  handle(query: Q, context: ExecutionContext): Promise<Result<R>>
}

UseCaseHandler<U extends UseCase, I, R> {
  handle(input: I, context: ExecutionContext): Promise<Result<R>>
}
```

**ExecutionContext** proporciona al handler:
- `identity`: Usuario autenticado (si aplica)
- `tenantId`: Identificador del tenant (multi-tenancy)
- `correlationId`: ID de trazabilidad
- `metadata`: Mapa clave-valor extensible

## 3.4 UseCase (Caso de Uso Compuesto)

Un UseCase es un handler que puede orquestar múltiples commands y queries. Representa una operación de negocio de alto nivel.

```
UseCase<I, R> {
  input: I
  execute(input: I, ctx: ExecutionContext): AsyncGenerator<DomainEvent | SubResult, R>
}
```

El caso de uso compuesto emite eventos de dominio y resultados intermedios a medida que avanza. El framework recolecta los eventos y los publica al finalizar exitosamente.

## 3.5 Result

Toda ejecución retorna un `Result` que encapsula éxito o fracaso de forma tipada y explícita:

```
Result<T> = Success<T> | Failure

Success<T> {
  value: T
}

Failure {
  error: DomainError
  details?: ValidationError[]
  correlationId: string
}
```

El Result es un tipo algebraico (sum type) que fuerza al invocador a manejar ambos casos. No hay excepciones en la capa de aplicación — los errores se representan como `Failure`.

## 3.6 DomainError

Cada error del dominio es una clase o tipo explícito:

```
DomainError {
  code: string           // ej. "ESCROW_003" — "INSUFFICIENT_FUNDS"
  message: string         // mensaje legible para el usuario
  details?: any           // detalles técnicos (no sensibles)
  httpStatus?: number     // mapeo opcional a HTTP (400, 403, 404, 409, 422, 500)
}
```

**Categorías de DomainError**:

| Categoría | Code Prefix | Ejemplo |
|---|---|---|
| Validación | VALIDATION_ | VALIDATION_EMAIL_ALREADY_EXISTS |
| Autorización | AUTH_ | AUTH_INSUFFICIENT_PERMISSIONS |
| No encontrado | NOT_FOUND_ | NOT_FOUND_OPERATION |
| Conflicto | CONFLICT_ | CONFLICT_ALREADY_SIGNED |
| Regla de negocio | BUSINESS_ | BUSINESS_CANNOT_CANCEL_AFTER_DEPOSIT |
| Límite excedido | LIMIT_ | LIMIT_KYC_NOT_COMPLETED |
| Infraestructura | INFRA_ | INFRA_PAYMENT_GATEWAY_TIMEOUT |

---

# 4. Ciclo de Vida de un Caso de Uso

## 4.1 Diagrama de Secuencia (Command)

```
Actor                    UseCaseBus               Pipeline                 Handler              Puertos
  │                         │                       │                       │                     │
  │—— send(command) —————→  │                       │                       │                     │
  │                         │—— resolve handler ——→ │                       │                     │
  │                         │                       │—— behavior[0] · log →│                     │
  │                         │                       │—— behavior[1] · val →│                     │
  │                         │                       │—— behavior[2] · auth→│                     │
  │                         │                       │—— behavior[3] · tx →│                     │
  │                         │                       │—— handler.handle() →│————— port.call() —→ │
  │                         │                       │                     │←————— response ——─ │
  │                         │                       │←—— Result<T> ———————│                     │
  │                         │                       │—— behavior[4] · ev →│                     │
  │                         │                       │—— behavior[5] · log│                     │
  │                         │←—— Result<T> ————————│                       │                     │
  │←—— Result<T> ——————————│                       │                       │                     │
```

## 4.2 Fases del Ciclo de Vida

| Fase | Behavior | Descripción | ¿Puede fallar? |
|---|---|---|---|
| **0. Resolución** | — | El UseCaseBus resuelve el handler registrado para el tipo de comando/query. | Sí (handler no registrado → 500) |
| **1. Logging** | LoggingBehavior | Registra el comando entrante con sus metadatos. | No |
| **2. Validación** | ValidationBehavior | Valida el input del comando contra reglas declarativas (schemas, invariantes). | Sí → ValidationError |
| **3. Autorización** | AuthorizationBehavior | Verifica que el actor tenga permiso para ejecutar esta acción sobre el recurso. | Sí → AuthError |
| **4. Transacción** | TransactionBehavior | Abre una transacción (si el handler es de escritura). | No (fallo en commit = rollback) |
| **5. Ejecución** | — | El handler ejecuta la lógica del caso de uso. | Sí → DomainError |
| **6. Eventos** | EventPublishBehavior | Recolecta y publica los eventos de dominio generados. | No (si falla, log + compensación) |
| **7. Métricas** | MetricsBehavior | Registra tiempo de ejecución, éxito/fallo, latencia. | No |

---

# 5. Pipeline de Ejecución

## 5.1 Estructura del Pipeline

El pipeline sigue el patrón **Chain of Responsibility** (o middleware pipeline). Cada behavior envuelve al siguiente:

```
pipeline = [
  LoggingBehavior,
  ValidationBehavior,
  AuthorizationBehavior,
  TransactionBehavior,    // solo para commands
  EventPublishBehavior,
  MetricsBehavior,
]
```

Cada behavior recibe el comando/query y un `next()` que invoca al siguiente behavior en la cadena. Esto permite que un behavior ejecute código **antes y después** de la ejecución del handler.

## 5.2 Pipeline en Pseudocódigo

```
async function executePipeline(command, handler, context):
  result = await LoggingBehavior.before(command, context)
  
  result = await ValidationBehavior.before(command, context)
  
  result = await AuthorizationBehavior.before(command, context)
  
  // Transacción solo para commands
  if command is Command:
    result = await TransactionBehavior.before(command, context)
  
  result = await handler.handle(command, context)
  
  if command is Command:
    result = await TransactionBehavior.after(result, context)
  
  result = await EventPublishBehavior.after(result, context)
  result = await MetricsBehavior.after(result, context)
  result = await LoggingBehavior.after(result, context)
  
  return result
```

## 5.3 Orden Configurable

El framework permite reordenar los behaviors mediante configuración:

```
framework.configure({
  pipeline: {
    order: ['logging', 'validation', 'authorization', 'transaction', 'events', 'metrics']
  }
})
```

También permite agregar behaviors custom en cualquier posición:

```
framework.addBehavior('audit', AuditBehavior, { position: 'after', target: 'events' })
```

---

# 6. Behaviors (Cross-Cutting Concerns)

## 6.1 LoggingBehavior

**Responsabilidad**: Registrar cada comando/query entrante y su resultado.

```
LoggingBehavior {
  before(command, ctx):
    logger.info({
      message: "Executing command",
      commandType: command.constructor.name,
      commandId: command.id,
      correlationId: ctx.correlationId,
      actorId: ctx.identity?.id,
    })
  
  after(result, ctx):
    if result.isSuccess():
      logger.info({ message: "Command succeeded", commandId: ctx.commandId })
    else:
      logger.warn({ message: "Command failed", commandId: ctx.commandId, error: result.error })
}
```

**Configuración**: Nivel de log por tipo de comando, omisión de campos sensibles (passwords, tokens, API keys).

## 6.2 ValidationBehavior

**Responsabilidad**: Validar el input del comando/query contra un schema declarativo antes de la ejecución.

La validación usa schemas definidos por cada comando:

```
CreateUserCommand {
  email: string    // pattern: /^.+@.+$/, maxLength: 255
  password: string // minLength: 8, maxLength: 128, pattern: /^(?=.*[A-Z]).../
  name: string     // required, maxLength: 100
}
```

El framework provee una interfaz de validación ágnostica que permite usar cualquier librería:

```
interface Validator {
  validate(schema: any, data: any): ValidationResult
}

ValidationResult {
  isValid: boolean
  errors: ValidationError[]
}

ValidationError {
  field: string
  message: string
  code: string
  params?: any
}
```

**Implementaciones concretas** (a cargo del cliente): Zod, Joi, Yup, Class-Validator, JsonSchema, Pydantic.

## 6.3 AuthorizationBehavior

**Responsabilidad**: Verificar que el actor tenga permiso para ejecutar la acción sobre el recurso.

El behavior invoca un **AuthorizationService** que el cliente implementa:

```
interface AuthorizationService {
  authorize(action: string, resource: string, identity: Identity): Promise<boolean>
}
```

El framework construye automáticamente `action` y `resource` a partir del tipo de comando:

| Comando | Action | Resource |
|---|---|---|
| CreateUserCommand | "create" | "user" |
| CancelOperationCommand | "cancel" | "operation:{operationId}" |
| ApproveDeliveryCommand | "approve_delivery" | "operation:{operationId}" |

**Reglas**:
- El AuthorizationBehavior se ejecuta **después** de la validación (input válido) y **antes** de la transacción.
- Si la autorización falla, retorna `Failure(AUTH_INSUFFICIENT_PERMISSIONS)` sin ejecutar el handler.
- El framework soporta RBAC y ABAC mediante el AuthorizationService.

## 6.4 TransactionBehavior

**Responsabilidad**: Garantizar atomicidad en commands que mutan el estado.

```
TransactionBehavior {
  before(command, ctx):
    if command is Command:
      transaction = await unitOfWork.begin()
      ctx.set("transaction", transaction)
  
  after(result, ctx):
    transaction = ctx.get("transaction")
    if result.isSuccess():
      await unitOfWork.commit(transaction)
    else:
      await unitOfWork.rollback(transaction)
}
```

**Interfaz que el cliente implementa**:

```
interface UnitOfWork {
  begin(): Promise<Transaction>
  commit(tx: Transaction): Promise<void>
  rollback(tx: Transaction): Promise<void>
}
```

## 6.5 EventPublishBehavior

**Responsabilidad**: Publicar eventos de dominio generados durante la ejecución del handler.

El handler puede emitir eventos mediante el contexto:

```
class CreateUserHandler implements CommandHandler<CreateUserCommand, UserId> {
  async handle(command: CreateUserCommand, ctx: ExecutionContext): Promise<Result<UserId>> {
    // ... lógica del dominio ...
    ctx.emit(new UserCreatedEvent(userId, command.email))
    return Success(userId)
  }
}
```

El behavior recolecta todos los eventos emitidos y los publica después del commit exitoso:

```
EventPublishBehavior {
  after(result, ctx):
    if result.isSuccess() and ctx.hasEvents():
      await eventBus.publishAll(ctx.getEvents())
}
```

## 6.6 MetricsBehavior

**Responsabilidad**: Registrar métricas de ejecución para observabilidad.

```
MetricsBehavior {
  before(command, ctx):
    ctx.set("startTime", now())
  
  after(result, ctx):
    duration = now() - ctx.get("startTime")
    metrics.record({
      name: "use_case.execution",
      tags: {
        commandType: command.constructor.name,
        success: result.isSuccess(),
        errorCode: result.isFailure() ? result.error.code : null
      },
      value: duration
    })
}
```

---

# 7. Composición de Casos de Uso

## 7.1 UseCase Compuesto

Un UseCase compuesto orquesta múltiples commands y queries. El framework provee el `UseCaseBus` al handler para que pueda enviar sub-comandos:

```
class CreateEscrowAndInviteUseCase implements UseCase<EscrowInput, EscrowResult> {
  constructor(private bus: UseCaseBus) {}
  
  async execute(input: EscrowInput, ctx: ExecutionContext): Promise<Result<EscrowResult>> {
    // Paso 1: Crear la operación (Command)
    const createResult = await this.bus.send(new CreateEscrowCommand(input))
    if (createResult.isFailure()) return createResult
    
    // Paso 2: Invitar contraparte (otro Command)
    const escrowId = createResult.value
    const inviteResult = await this.bus.send(new InviteCounterpartyCommand(escrowId, input.counterpartyEmail))
    if (inviteResult.isFailure()) return inviteResult
    
    // Paso 3: Notificar (Query para obtener datos + notificación asíncrona)
    ctx.emit(new EscrowCreatedEvent(escrowId))
    
    return Success({ escrowId, inviteStatus: inviteResult.value })
  }
}
```

## 7.2 Sagas (Orquestación de Largo Plazo)

Para procesos que abarcan múltiples pasos con esperas (como el flujo completo Escrow), el framework provee un mecanismo de **Saga**:

```
Saga {
  steps: Step[]
  compensations: Map<Step, Compensation>
  
  execute(input): AsyncGenerator<StepResult>
  
  onFailure(stepFailure): void  // ejecuta compensaciones en orden inverso
}
```

**Ejemplo**: Saga de operación Escrow

```
EscrowSaga:
  1. CreateEscrowCommand          → compensación: CancelEscrowCommand
  2. SignAgreementCommand         → compensación: VoidAgreementCommand
  3. DepositAssetCommand          → compensación: RefundDepositCommand
  4. NotifyFulfillmentCommand     → compensación: (ninguna, es notificación)
  5. WaitForApprovalOrDispute     → temporizador, no es un comando
  6. ReleaseAssetCommand          → compensación: ReverseReleaseCommand
```

## 7.3 Scheduling y Temporizadores

El framework debe soportar la ejecución diferida de comandos para casos como:

- **UC-22**: Observar período de revisión — liberación automática si no hay objeción.
- **Expiración de invitaciones**: Cancelar invitaciones no aceptadas después de N días.

```
await bus.schedule(
  new AutoReleaseCommand(escrowId),
  delay: Duration.ofDays(5)
)
```

---

# 8. Manejo de Errores y Resultados

## 8.1 Principios

- **Sin excepciones para flujo de negocio**: Los errores esperados (validación, autorización, reglas de negocio) son `Failure` en un `Result`.
- **Excepciones solo para errores de infraestructura**: Base de datos caída, cola de mensajes no disponible, error de red. Estas se capturan en los adapters y se convierten a `Failure` con código `INFRA_*`.
- **El resultado es explícito**: Ninguna función de la capa de aplicación puede "lanzar" una excepción de negocio.

## 8.2 Result Chain

El framework permite encadenar operaciones que retornan Result:

```
const result = await bus.send(command1)
  .chain(value => bus.send(new Command2(value.id)))
  .chain(value => bus.send(new Command3(value.id)))
```

Cada `chain` solo se ejecuta si el paso anterior fue `Success`. Si algún paso falla, el Failure se propaga.

## 8.3 Mapeo a HTTP (para controllers)

El framework provee un mapper que convierte `Result<T>` a respuestas HTTP estándar:

| Result | HTTP Status | Body |
|---|---|---|
| `Success<T>` | 200 OK / 201 Created | value serializado |
| `Failure(VALIDATION_*)` | 422 Unprocessable Entity | Errores de validación |
| `Failure(AUTH_*)` | 403 Forbidden | Mensaje de error |
| `Failure(NOT_FOUND_*)` | 404 Not Found | Mensaje de error |
| `Failure(CONFLICT_*)` | 409 Conflict | Mensaje de error |
| `Failure(BUSINESS_*)` | 400 Bad Request | Mensaje de error |
| `Failure(LIMIT_*)` | 429 Too Many Requests | Mensaje de error |
| `Failure(INFRA_*)` | 502 / 503 | Mensaje genérico |

---

# 9. Eventos de Dominio

## 9.1 Definición

Un Evento de Dominio representa algo que ocurrió en el pasado y que es relevante para el negocio. Es inmutable y lleva timestamp.

```
DomainEvent {
  eventId: string (UUID v7)
  eventType: string (ej. "escrow.deposit.confirmed")
  aggregateId: string
  aggregateType: string
  timestamp: DateTime
  correlationId: string
  causationId?: string
  data: any (payload específico del evento)
  metadata?: Map<string, string>
}
```

## 9.2 Flujo de Eventos

1. El handler emite eventos mediante `ctx.emit(event)` durante la ejecución.
2. El framework almacena los eventos en memoria (dentro del contexto de ejecución).
3. Al finalizar el handler exitosamente, el `EventPublishBehavior` publica todos los eventos.
4. Si el handler falla, los eventos se descartan (nunca se publican).

## 9.3 Suscripción a Eventos

El framework permite registrar suscriptores:

```
framework.on(UserRegisteredEvent, async (event, ctx) => {
  await bus.send(new SendWelcomeEmailCommand(event.userId))
})
```

Los suscriptores se ejecutan **después** del commit de la transacción (para evitar enviar emails basados en datos no persistidos).

## 9.4 Eventos del Sistema Escrow

| Evento | Disparado por | Consumidores potenciales |
|---|---|---|
| OperationCreated | UC-13 | Notificaciones, Contratos |
| AgreementSigned | UC-17 | Escrow Core, Auditoría |
| AssetDeposited | UC-18/UC-19 | Escrow Core, Wallet, Notificaciones |
| DeliveryApproved | UC-21 | Escrow Core, Wallet, Pagos, Notificaciones |
| AssetReleased | UC-27/UC-34 | Wallet, Notificaciones, Reportes |
| OperationCompleted | UC-27 | Reportes, IA (para scoring), Auditoría |
| DisputeOpened | UC-28 | Arbitraje, Notificaciones, IA (detección) |
| DisputeResolved | UC-33/UC-34 | Escrow Core, Wallet, Notificaciones |
| UserVerified | UC-05 | Usuarios, Notificaciones |
| SuspiciousActivity | UC-66 | Compliance, Notificaciones, Auditoría |

---

# 10. Puertos y Adaptadores (Ports & Adapters)

## 10.1 Definición

Los **puertos** son interfaces que el framework (o los handlers) necesitan para interactuar con el mundo exterior. Los **adaptadores** son implementaciones concretas de esos puertos que la aplicación cliente proporciona.

## 10.2 Puertos del Framework

| Puerto | Propósito | Métodos Clave |
|---|---|---|
| **UnitOfWork** | Gestión de transacciones | begin(), commit(), rollback() |
| **EventBus** | Publicación de eventos | publish(event), publishAll(events) |
| **AuthorizationService** | Verificación de permisos | authorize(action, resource, identity) |
| **IdentityProvider** | Resolución de identidad actual | getCurrentIdentity(): Identity |
| **Clock** | Obtención de tiempo | now(): DateTime |
| **IdGenerator** | Generación de IDs únicos | generate(): string |
| **Logger** | Logging estructurado | info(), warn(), error(), debug() |
| **MetricsRecorder** | Registro de métricas | record(name, tags, value) |

## 10.3 Puertos de Dominio (definidos por el cliente)

Cada handler define los puertos que necesita mediante su constructor (inyección de dependencias). Por ejemplo, un handler de creación de usuario podría requerir:

```
class CreateUserHandler {
  constructor(
    private userRepository: UserRepository,       // puerto de dominio
    private passwordHasher: PasswordHasher,        // puerto de dominio
    private eventBus: EventBus,                    // puerto del framework
    private idGenerator: IdGenerator               // puerto del framework
  ) {}
  
  async handle(command: CreateUserCommand, ctx: ExecutionContext): Promise<Result<UserId>> {
    // usa los puertos sin saber si la implementación es PostgreSQL, MongoDB, en memoria, etc.
  }
}
```

## 10.4 Inversión de Dependencias

El framework no conoce las implementaciones concretas. El cliente las inyecta al configurar el framework:

```
const framework = UseCaseFramework.create({
  ports: {
    unitOfWork: new PostgresUnitOfWork(dataSource),
    eventBus: new KafkaEventBus(kafkaClient),
    authorizationService: new RBACAuthorizationService(permissionRepository),
    clock: new SystemClock(),
    idGenerator: new UlidGenerator(),
    logger: new PinoLogger(),
    metricsRecorder: new PrometheusMetricsRecorder(),
  }
})
```

---

# 11. Autorización y Seguridad

## 11.1 Niveles de Autorización

| Nivel | Evaluación | Implementación |
|---|---|---|
| **Autenticación** | ¿Quién es el actor? | IdentityProvider + ExecutionContext.identity |
| **Autorización estática (RBAC)** | ¿Tiene el rol necesario? | AuthorizationService + tabla roles-permisos |
| **Autorización dinámica (ABAC)** | ¿Cumple las políticas sobre el recurso? | AuthorizationService + policy engine |
| **Autorización por recurso** | ¿Es el propietario del recurso? | Handler verifica ownership |

## 11.2 Declaración de Permisos por Caso de Uso

Cada handler puede declarar los permisos que requiere mediante anotación o configuración:

```
@Authorized({
  action: "create",
  resource: "user",
  conditions: ["isPublic"]  // sin autenticación
})
class CreateUserCommandHandler { ... }

@Authorized({
  action: "approve_delivery",
  resource: "operation",
  conditions: ["isBuyer", "operation.state == 'delivered'"]
})
class ApproveDeliveryHandler { ... }
```

## 11.3 Validación de Propiedad (Resource Ownership)

Para casos donde un usuario solo puede actuar sobre recursos que le pertenecen:

```
class CancelOperationHandler implements CommandHandler<CancelOperationCommand, void> {
  constructor(private operationRepository: OperationRepository) {}
  
  async handle(command: CancelOperationCommand, ctx: ExecutionContext): Promise<Result<void>> {
    const operation = await this.operationRepository.findById(command.operationId)
    if (!operation) return Failure(NOT_FOUND_OPERATION)
    
    // Validación de ownership
    if (!operation.isParticipant(ctx.identity.id)) {
      return Failure(AUTH_NOT_OWNER)
    }
    
    // ... lógica de cancelación ...
  }
}
```

---

# 12. Validación

## 12.1 Validación de Input (Capa de Aplicación)

Se aplica a todo comando y query ANTES de la ejecución del handler. El ValidationBehavior ejecuta el schema asociado al comando.

**Esquema por comando** (definido por el cliente):

```
CreateEscrowCommand.schema = {
  type: 'object',
  properties: {
    assetType: { type: 'string', enum: ['money', 'crypto', 'document', 'domain', 'nft'] },
    amount: { type: 'number', minimum: 0.01 },
    currency: { type: 'string', minLength: 3, maxLength: 3 },
    counterpartyEmail: { type: 'string', format: 'email' },
    conditions: { type: 'object', required: ['reviewDays'], properties: { ... } }
  },
  required: ['assetType', 'amount', 'currency', 'counterpartyEmail']
}
```

## 12.2 Validación de Negocio (Capa de Dominio)

Se aplica DENTRO del handler o en las entidades de dominio. Evalúa invariantes que requieren acceso a datos:

- ¿El email ya está registrado? → `VALIDATION_EMAIL_ALREADY_EXISTS`
- ¿La operación está en un estado que permite cancelación? → `BUSINESS_INVALID_STATE`
- ¿El depósito mínimo está cubierto? → `BUSINESS_MINIMUM_DEPOSIT_NOT_MET`

## 12.3 Separación de Responsabilidades

| Tipo | Cuándo | Dónde | Error |
|---|---|---|---|
| Sintáctica | Input | ValidationBehavior | VALIDATION_INVALID_FORMAT |
| Semántica | Input | ValidationBehavior | VALIDATION_EMAIL_EXISTS |
| De negocio | Handler | Capa de dominio | BUSINESS_* |
| De integridad | Handler | Capa de dominio | CONFLICT_* |

---

# 13. Transacciones y Consistencia

## 13.1 Estrategia por Tipo de Operación

| Tipo | Estrategia | Descripción |
|---|---|---|
| **Command simple** | Transacción ACID | Un handler, un aggregate, una transacción |
| **UseCase compuesto** | Saga coreografiada | Múltiples handlers, cada uno con su transacción, eventos para coordinar |
| **Query** | Sin transacción | Lectura directa, posible caché |
| **Comando asíncrono** | Outbox pattern | Handler escribe en BD + outbox table; worker publica eventos desde la outbox |

## 13.2 Outbox Pattern (Eventos Confiables)

Para garantizar que los eventos se publican exactamente una vez incluso si el bus de eventos falla:

1. El handler escribe datos + evento en la misma transacción de BD.
2. Un worker independiente lee la outbox table y publica en el EventBus.
3. Después de publicar, marca el evento como enviado.
4. Si el EventBus falla, el worker reintenta con backoff.

```
OUTBOX_TABLE:
  id: UUID
  event_type: string
  event_data: JSON
  aggregate_id: string
  created_at: timestamp
  status: enum('pending', 'published', 'failed')
  retry_count: integer
```

## 13.3 Consistencia Eventual vs Fuerte

| Operación | Consistencia | Justificación |
|---|---|---|
| Depósito de activo | Fuerte (ACID) | El saldo debe ser exacto |
| Liberación de fondos | Fuerte (ACID) | Implica movimiento de dinero |
| Notificación | Eventual | Si falla, se reintenta |
| Actualización de perfil | Eventual | No crítica; puede haber réplicas |
| Reporte de auditoría | Eventual | Datos históricos, no en tiempo real |

---

# 14. Pruebas

## 14.1 Testeabilidad del Framework

El framework se diseña para que:

- Cada behavior se pruebe en aislamiento.
- El pipeline se pruebe completo con un handler mock.
- Los handlers de caso de uso se prueben inyectando puertos mockeados.

## 14.2 Estrategia de Pruebas para Handlers

```
describe('CreateUserHandler', () => {
  let handler: CreateUserHandler
  let userRepo: MockUserRepository
  let hasher: MockPasswordHasher
  
  beforeEach(() => {
    userRepo = new MockUserRepository()
    hasher = new MockPasswordHasher()
    handler = new CreateUserHandler(userRepo, hasher)
  })
  
  it('should create user when email is unique', async () => {
    userRepo.findByEmail.mockResolvedValue(null)
    hasher.hash.mockResolvedValue('hashed_password')
    
    const command = new CreateUserCommand({ email: 'test@test.com', password: 'Secure123' })
    const result = await handler.handle(command, executionContext())
    
    expect(result.isSuccess()).toBe(true)
    expect(userRepo.save).toHaveBeenCalled()
  })
  
  it('should fail when email already exists', async () => {
    userRepo.findByEmail.mockResolvedValue(existingUser)
    
    const command = new CreateUserCommand({ email: 'existing@test.com', password: 'Secure123' })
    const result = await handler.handle(command, executionContext())
    
    expect(result.isFailure()).toBe(true)
    expect(result.error.code).toBe('VALIDATION_EMAIL_ALREADY_EXISTS')
  })
})
```

## 14.3 Tipos de Prueba

| Tipo | Qué prueba | Velocidad |
|---|---|---|
| **Unitarias** | Cada handler con puertos mockeados | Milisegundos |
| **Integración** | Handler + puertos reales (BD, EventBus) | Segundos |
| **Pipeline** | Behaviors + handler completo | Milisegundos |
| **Contrato** | Puertos contra sus adaptadores reales | Segundos |
| **Saga** | Flujo completo de múltiples handlers | Segundos-minutos |

---

# 15. Extensibilidad y Puntos de Extensión

## 15.1 Behaviors Custom

El cliente puede agregar behaviors al pipeline:

```
framework.addBehavior('audit', new AuditBehavior(), {
  position: 'after',
  target: 'transaction',
  onlyFor: ['commands']
})
```

## 15.2 Decoradores de Handler

El framework puede aplicar decoradores a handlers específicos:

```
@RateLimited({ maxRequests: 10, windowMs: 60000 })
@Idempotent({ ttl: Duration.ofHours(24) })
@RetryOnFailure({ maxRetries: 3, backoff: 'exponential' })
class DepositAssetHandler implements CommandHandler<DepositAssetCommand, DepositId> {
  ...
}
```

## 15.3 Middleware Global

Middleware que se ejecuta para todos los comandos/queries sin distinción:

```
framework.use(async (command, ctx, next) => {
  const start = performance.now()
  const result = await next()
  const duration = performance.now() - start
  
  if (duration > 5000) {
    ctx.logger.warn({ commandType: command.type, duration })
  }
  
  return result
})
```

## 15.4 Event Listeners Globales

```
framework.onAny((event, ctx) => {
  ctx.logger.debug({ eventType: event.eventType, aggregateId: event.aggregateId })
})
```

---

# 16. Glosario del Framework

| Término | Definición |
|---|---|
| **UseCaseBus** | Mediator central que recibe commands/queries y los enruta a sus handlers a través del pipeline |
| **Command** | Objeto inmutable que expresa la intención de mutar el estado del sistema |
| **Query** | Objeto inmutable que expresa la intención de leer datos sin mutar el estado |
| **Handler** | Unidad de ejecución que procesa un comando, query o caso de uso compuesto |
| **Result** | Tipo algebraico (Success | Failure) que encapsula el resultado de una ejecución |
| **DomainError** | Error explícito del dominio con código, mensaje y detalles |
| **Behavior** | Componente del pipeline que implementa un cross-cutting concern (validación, autorización, logging) |
| **Pipeline** | Cadena de behaviors que envuelven la ejecución del handler |
| **ExecutionContext** | Contexto de ejecución que transporta identidad, trazabilidad y metadata |
| **Puerto** | Interfaz que define una dependencia hacia el exterior (BD, bus, servicios) |
| **Adaptador** | Implementación concreta de un puerto (PostgreSQL, Kafka, etc.) |
| **Saga** | Orquestación de largo plazo con steps, compensaciones y manejo de fallos |
| **Domain Event** | Evento inmutable que representa algo que ocurrió en el dominio |
| **Outbox Pattern** | Mecanismo para publicación confiable de eventos usando una tabla en la BD |
| **Unit of Work** | Abstracción de transacciones que coordina múltiples operaciones atómicas |
| **CorrelationId** | Identificador único que permite rastrear una operación a través de múltiples servicios y handlers |

---

*Este documento es una especificación técnica para el desarrollo del Use Case Framework. No asume ningún lenguaje, framework web, base de datos o infraestructura específicos. El cliente concreto implementa los puertos y proporciona los adaptadores.*
