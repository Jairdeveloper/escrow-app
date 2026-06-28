# Escrow Platform — Client Architecture

**Versión:** 1.0  
**Propósito:** Arquitectura del programa cliente que utiliza el Use Case Framework para implementar los 66 casos de uso definidos en la Sección 4 de `ARQUITECTURA.md`.  
**Relación:** Este documento describe la aplicación concreta de Escrow construida sobre el framework definido en `USE_CASE_FRAMEWORK.md`. El framework provee el pipeline, los behaviors y las abstracciones; este documento describe cómo se organizan, agrupan y conectan los componentes del dominio Escrow.

---

## Índice

1. [Principios Arquitectónicos](#1-principios-arquitectónicos)
2. [Estructura de Paquetes / Módulos](#2-estructura-de-paquetes--módulos)
3. [Mapa de Casos de Uso a Handlers](#3-mapa-de-casos-de-uso-a-handlers)
4. [Módulo: Usuarios y Cuentas](#4-módulo-usuarios-y-cuentas)
5. [Módulo: Operaciones Escrow](#5-módulo-operaciones-escrow)
6. [Módulo: Disputas y Arbitraje](#6-módulo-disputas-y-arbitraje)
7. [Módulo: Pagos, Wallet y Facturación](#7-módulo-pagos-wallet-y-facturación)
8. [Módulo: Administración y Configuración](#8-módulo-administración-y-configuración)
9. [Módulo: Notificaciones y Mensajería](#9-módulo-notificaciones-y-mensajería)
10. [Módulo: API e Integraciones](#10-módulo-api-e-integraciones)
11. [Módulo: Auditoría y Cumplimiento](#11-módulo-auditoría-y-cumplimiento)
12. [Entidades y Aggregate Roots del Dominio](#12-entidades-y-aggregate-roots-del-dominio)
13. [Puertos de Dominio (Repositories)](#13-puertos-de-dominio-repositories)
14. [Conexión con el Framework](#14-conexión-con-el-framework)
15. [Configuración del Pipeline para Escrow](#15-configuración-del-pipeline-para-escrow)
16. [Handlers Compuestos y Sagas](#16-handlers-compuestos-y-sagas)
17. [Adaptadores de Infraestructura](#17-adaptadores-de-infraestructura)
18. [Pruebas del Cliente](#18-pruebas-del-cliente)

---

# 1. Principios Arquitectónicos

Además de los principios del framework (sección 1.2 de `USE_CASE_FRAMEWORK.md`), el cliente Escrow sigue estos principios específicos:

- **Trazabilidad financiera**: Toda mutación de saldo o liberación de activos genera un evento de auditoría con firma hash-encadenada.
- **Atomicidad de liberaciones**: Una liberación de fondos implica una transacción que abarca wallet + operación + evento de auditoría.
- **Segregación de activos**: Cada operación Escrow tiene su propia línea contable; no hay "pool" de fondos mezclados.
- **Inmutabilidad de operaciones cerradas**: Una vez que una operación alcanza estado `COMPLETED` o `CANCELLED`, no puede modificarse; solo auditarla.
- **Máquina de estados explícita**: Los estados de una operación Escrow se modelan como un tipo cerrado (sealed enum/sum type) y las transiciones se validan en el handler.

---

# 2. Estructura de Paquetes / Módulos

## 2.1 Organización por Domino (DDD Bounded Contexts)

```
src/
├── context/
│   ├── identity/           # Registro, autenticación, perfiles, KYC/KYB
│   ├── escrow/             # Core: operaciones, hitos, contratos
│   ├── wallet/             # Saldos, transacciones financieras, comisiones
│   ├── dispute/            # Disputas, evidencias, arbitraje
│   ├── messaging/          # Chat, notificaciones
│   ├── compliance/         # AML, listas de sanciones, auditoría, reportes
│   ├── admin/              # Panel administrativo, configuración
│   └── integration/        # API pública, webhooks, SDK internos
├── framework/              # Adaptación local del Use Case Framework (wiring)
├── infrastructure/         # Adaptadores concretos (BD, cache, bus, cloud)
└── shared/                 # Value objects compartidos, utilerías de dominio
    ├── domain/
    │   ├── Entity.ts
    │   ├── ValueObject.ts
    │   ├── AggregateRoot.ts
    │   ├── DomainEvent.ts
    │   └── DomainError.ts
    └── kernel/
        ├── Money.ts
        ├── Currency.ts
        ├── Percentage.ts
        ├── Email.ts
        └── Phone.ts
```

## 2.2 Estructura Interna de Cada Contexto (Hexagonal)

Cada bounded context sigue una estructura hexagonal que se alinea con el framework:

```
context/escrow/
├── domain/
│   ├── EscrowOperation.ts          # Aggregate Root
│   ├── EscrowState.ts              # Máquina de estados (enum sellado)
│   ├── Milestone.ts                # Value Object
│   ├── Condition.ts                # Value Object
│   ├── events/                     # Eventos de dominio
│   │   ├── OperationCreated.ts
│   │   ├── AssetDeposited.ts
│   │   ├── DeliveryApproved.ts
│   │   └── AssetReleased.ts
│   └── ports/
│       ├── EscrowOperationRepository.ts    # Puerto (interfaz)
│       └── AssetVerificationService.ts     # Puerto (interfaz)
├── application/
│   ├── commands/                   # Handlers de comandos
│   │   ├── CreateEscrowHandler.ts
│   │   ├── DepositAssetHandler.ts
│   │   ├── ApproveDeliveryHandler.ts
│   │   └── CancelOperationHandler.ts
│   ├── queries/                    # Handlers de consultas
│   │   ├── GetOperationDetailHandler.ts
│   │   └── ListUserOperationsHandler.ts
│   └── useCases/                   # Casos de uso compuestos / sagas
│       ├── CreateEscrowFlowSaga.ts
│       └── ReleaseFlowSaga.ts
├── infrastructure/
│   ├── adapters/
│   │   └── PostgresEscrowOperationRepository.ts
│   └── listeners/
│       └── EscrowEventListeners.ts  # Suscriptores a eventos de otros contextos
└── tests/
    ├── unit/
    ├── integration/
    └── saga/
```

---

# 3. Mapa de Casos de Uso a Handlers

## 3.1 Resumen Completo (UC-01 a UC-66)

| UC ID | Nombre | Tipo | Handler / Saga | Módulo |
|---|---|---|---|---|
| **UC-01** | Registrarse como usuario individual | Command | `RegisterUserHandler` | identity |
| **UC-02** | Registrarse como empresa | Command | `RegisterCompanyHandler` | identity |
| **UC-03** | Iniciar sesión | Query → Auth | `LoginHandler` (autenticación, no pasa por el bus) | identity |
| **UC-04** | Recuperar contraseña | Command | `RequestPasswordResetHandler` + `ConfirmPasswordResetHandler` | identity |
| **UC-05** | Completar verificación KYC | Command | `SubmitKYCDocumentsHandler` → Saga: `KYCVerificationSaga` | identity |
| **UC-06** | Completar verificación KYB | Command | `SubmitKYBDocumentsHandler` → Saga: `KYBVerificationSaga` | identity |
| **UC-07** | Gestionar perfil | Command | `UpdateProfileHandler` | identity |
| **UC-08** | Configurar 2FA | Command | `SetupTwoFactorHandler` | identity |
| **UC-09** | Gestionar métodos de pago | Command | `AddPaymentMethodHandler`, `RemovePaymentMethodHandler` | wallet |
| **UC-10** | Ver historial de actividad | Query | `GetUserActivityHandler` | identity |
| **UC-11** | Cerrar cuenta | Command | `CloseAccountHandler` (Saga: verifica saldos → cierra → notifica) | identity |
| **UC-12** | Migrar de individual a empresa | Command | `MigrateToCompanyHandler` | identity |
| **UC-13** | Crear operación Escrow | Command | `CreateEscrowHandler` | escrow |
| **UC-14** | Invitar contraparte | Command | `InviteCounterpartyHandler` | escrow |
| **UC-15** | Aceptar invitación | Command | `AcceptInvitationHandler` | escrow |
| **UC-16** | Negociar condiciones | Command | `NegotiateTermsHandler` (versiona el acuerdo) | escrow |
| **UC-17** | Firmar acuerdo Escrow | Command | `SignAgreementHandler` | escrow |
| **UC-18** | Depositar activo | Command | `DepositAssetHandler` (dispara saga de confirmación) | escrow |
| **UC-19** | Confirmar recepción de activo | Command | `ConfirmAssetReceptionHandler` (sistema / webhook) | escrow |
| **UC-20** | Notificar cumplimiento | Command | `NotifyFulfillmentHandler` | escrow |
| **UC-21** | Aprobar entrega | Command | `ApproveDeliveryHandler` | escrow |
| **UC-22** | Observar período de revisión | Scheduled | `AutoReleaseOnExpirationJob` (temporizador programado) | escrow |
| **UC-23** | Solicitar devolución | Command | `RequestRefundHandler` | escrow |
| **UC-24** | Cancelar operación | Command | `CancelOperationHandler` (Saga: valida reglas → libera/cancela) | escrow |
| **UC-25** | Ver detalle de operación | Query | `GetOperationDetailHandler` | escrow |
| **UC-26** | Compartir operación | Command | `ShareOperationHandler` | escrow |
| **UC-27** | Aceptar liberación parcial automática | Event → Command | `AutoReleaseOnMilestoneHandler` (escuchá evento de hito cumplido) | escrow |
| **UC-28** | Iniciar disputa | Command | `OpenDisputeHandler` | dispute |
| **UC-29** | Negociar directamente | Command | `SendNegotiationMessageHandler` | dispute |
| **UC-30** | Escalar a arbitraje | Command | `EscalateToArbitrationHandler` | dispute |
| **UC-31** | Presentar evidencia | Command | `SubmitEvidenceHandler` | dispute |
| **UC-32** | Revisar caso | Query | `GetArbitrationCaseHandler` | dispute |
| **UC-33** | Emitir resolución | Command | `IssueResolutionHandler` | dispute |
| **UC-34** | Ejecutar resolución | Command | `ExecuteResolutionHandler` (Saga: libera fondos según fallo) | dispute |
| **UC-35** | Apelar resolución | Command | `AppealResolutionHandler` | dispute |
| **UC-36** | Depositar fondos a wallet interna | Command | `DepositToWalletHandler` | wallet |
| **UC-37** | Retirar fondos de wallet | Command | `WithdrawFromWalletHandler` | wallet |
| **UC-38** | Consultar saldo | Query | `GetWalletBalanceHandler` | wallet |
| **UC-39** | Ver historial de transacciones | Query | `GetTransactionHistoryHandler` | wallet |
| **UC-40** | Descargar factura | Query | `DownloadInvoiceHandler` | wallet |
| **UC-41** | Configurar comisiones | Command | `ConfigureFeeScheduleHandler` | admin |
| **UC-42** | Conciliar transacciones | Command | `ReconcileTransactionsHandler` | wallet |
| **UC-43** | Gestionar usuarios | Command | `AdminUserManagementHandler` (varios sub-comandos) | admin |
| **UC-44** | Revisar documentos KYC/KYB | Command | `ReviewKYCApplicationHandler`, `ReviewKYBApplicationHandler` | identity |
| **UC-45** | Configurar parámetros del sistema | Command | `UpdateSystemParameterHandler` | admin |
| **UC-46** | Gestionar tipos de activo | Command | `CreateAssetTypeHandler`, `UpdateAssetTypeHandler` | admin |
| **UC-47** | Monitorear operaciones activas | Query | `GetActiveOperationsDashboardHandler` | admin |
| **UC-48** | Generar reportes | Query | `GenerateReportHandler` (asíncrono, retorna jobId) | admin |
| **UC-49** | Gestionar administradores | Command | `AdminRoleManagementHandler` | admin |
| **UC-50** | Ver logs de auditoría | Query | `GetAuditLogHandler` | compliance |
| **UC-51** | Gestionar plantillas de contrato | Command | `ManageContractTemplateHandler` | escrow |
| **UC-52** | Configurar reglas de arbitraje | Command | `ConfigureArbitrationRulesHandler` | admin |
| **UC-53** | Enviar mensaje en operación | Command | `SendMessageHandler` | messaging |
| **UC-54** | Recibir notificación | Event → Push | Manejado por event listeners + adaptador de notificaciones | messaging |
| **UC-55** | Configurar preferencias de notificación | Command | `UpdateNotificationPreferencesHandler` | messaging |
| **UC-56** | Adjuntar archivo a mensaje | Command | `AttachFileToMessageHandler` | messaging |
| **UC-57** | Recibir alertas de seguridad | Event → Push | SecurityEventListeners | compliance |
| **UC-58** | Crear operación mediante API | Command | `CreateEscrowHandler` (mismo handler, diferente adaptador de entrada) | integration |
| **UC-59** | Consultar estado de operación por API | Query | `GetOperationDetailHandler` (mismo handler) | integration |
| **UC-60** | Verificar identidad por API | Command | `SubmitKYCDocumentsHandler` (mismo handler) | integration |
| **UC-61** | Gestionar webhooks | Command | `RegisterWebhookHandler`, `UpdateWebhookHandler` | integration |
| **UC-62** | Generar link de pago para Escrow | Command | `GeneratePaymentLinkHandler` | integration |
| **UC-63** | Consultar pista de auditoría | Query | `GetOperationAuditTrailHandler` | compliance |
| **UC-64** | Exportar reporte de cumplimiento | Query | `ExportComplianceReportHandler` (asíncrono) | compliance |
| **UC-65** | Verificar firmas digitales | Query | `VerifyDigitalSignatureHandler` | escrow |
| **UC-66** | Monitorear transacciones sospechosas | Event → Command | `FlagSuspiciousTransactionHandler` (disparado por reglas AML) | compliance |

---

# 4. Módulo: Usuarios y Cuentas

## 4.1 Entidades de Dominio

```
User (Aggregate Root)
  - id: UserId
  - type: UserType (individual | company)
  - email: Email (Value Object)
  - phone: Phone (Value Object)
  - passwordHash: string
  - verificationLevel: VerificationLevel (none | basic | standard | advanced | institutional)
  - status: UserStatus (active | suspended | closed)
  - profile: Profile (Value Object)
  - createdAt: DateTime
  - updatedAt: DateTime

Company (Aggregate Root, extiende User)
  - legalName: string
  - taxId: string
  - representativeId: UserId
  - incorporationDocuments: DocumentRef[]
  - uboDeclarations: UBO[]

UserKYC (Entity)
  - userId: UserId
  - documents: KYCDocument[]
  - status: KYCStatus (pending | under_review | approved | rejected)
  - verifiedAt: DateTime?
  - verifiedBy: UserId? (compliance officer)

IdentityVerification (Entity)
  - id: VerificationId
  - userId: UserId
  - type: VerificationType (document_ocr | liveness | sanctions_screening | pep)
  - provider: string (proveedor externo)
  - result: VerificationResult
  - metadata: JSON
```

## 4.2 Handlers Destacados

### RegisterUserHandler (UC-01)

```
RegisterUserCommand:
  - email: string
  - password: string
  - name: string
  - phone?: string

Flujo:
  1. Validation → email formato, password fortaleza
  2. Authorization → público (no requiere autenticación)
  3. Handler:
     a. Verificar que email no existe → UserRepository.findByEmail()
     b. Hashear password → PasswordHasher.hash()
     c. Crear entidad User
     d. Persistir → UserRepository.save(user)
     e. Generar token de verificación de email
     f. Emitir evento UserRegistered
  4. Eventos → UserRegistered → EmailVerificationListener envía email
```

### SubmitKYCDocumentsHandler (UC-05)

```
KYCVerificationSaga:
  Paso 1: SubmitKYCDocumentsCommand
    - Recibe documentos (INE, selfie, comprobante domicilio)
    - Almacena documentos cifrados en S3
    - Crea entidad UserKYC en estado "pending"
    - Emite evento KYCDocumentsSubmitted

  Paso 2 (asíncrono, escuchá evento):
    - OCRDocumentVerificationListener → extrae datos del documento
    - LivenessCheckListener → verifica selfie vs documento
    - SanctionsScreeningListener → cruza contra listas OFAC/UN

  Paso 3 (compliance officer o automático si score > umbral):
    - ReviewKYCApplicationHandler → aprueba/rechaza
    - Si aprueba: emite UserVerified → sube nivel de verificación del usuario
```

---

# 5. Módulo: Operaciones Escrow

## 5.1 Aggregate Root

```
EscrowOperation (Aggregate Root)
  - id: EscrowOperationId
  - buyerId: UserId
  - sellerId: UserId
  - status: EscrowState (enum sellado)
  - assetType: AssetType
  - amount: Money (para activos financieros)
  - assetRef: AssetReference (para activos no financieros)
  - conditions: Conditions (Value Object)
  - milestones: Milestone[] (para multi-hito)
  - agreementRef: AgreementId
  - disputeId: DisputeId?
  - createdAt: DateTime
  - updatedAt: DateTime
  - completedAt: DateTime?

EscrowState (sealed enum — transiciones validadas en el dominio):
  CREATED → NEGOTIATION → AGREED → DEPOSIT_PENDING →
  DEPOSITED → IN_EXECUTION → DELIVERED → IN_REVIEW →
  APPROVED → RELEASED → COMPLETED
                            → REJECTED → DISPUTE → ARBITRATION → RESOLVED → COMPLETED
  → CANCELLED (desde cualquier estado excepto COMPLETED, RELEASED)

Conditions (Value Object):
  - reviewDays: int (3-30)
  - arbitrationRules: ArbitrationRules
  - cancellationPenalty: Percentage?
  - feePayer: FeePayer (buyer | seller | split)
  - customClauses: Map<string, any>
```

## 5.2 Handlers Destacados

### CreateEscrowHandler (UC-13)

```
CreateEscrowCommand:
  - assetType: AssetType
  - amount?: Money
  - assetDescription?: string
  - counterpartyEmail: string
  - conditions: ConditionsInput

Flujo:
  1. Validation → tipos de activo válidos, monto > 0, email formato
  2. Authorization → solo usuarios con verificationLevel >= basic
  3. Handler:
     a. Validar reglas de negocio:
        - ¿El comprador tiene nivel de verificación suficiente para el monto?
        - ¿El tipo de activo está habilitado?
     b. Si la contraparte no existe como usuario:
        - Crear registro provisional (pre-user) con el email
        - Emitir evento CounterpartyInvited
     c. Si existe:
        - Asignar sellerId
     d. Crear entidad EscrowOperation en estado CREATED
     e. Emitir evento OperationCreated
```

### InviteCounterpartyHandler (UC-14)

```
InviteCounterpartyCommand:
  - escrowId: EscrowOperationId
  - counterpartyEmail: string

Flujo:
  1. Authorization → solo el buyer puede invitar
  2. Handler:
     a. EscrowOperation debe estar en estado CREATED o NEGOTIATION
     b. Si el email corresponde a un usuario existente:
        - Agregar sellerId a la operación
        - Enviar notificación in-app
     c. Si no existe:
        - Enviar email de invitación con link de registro + token
        - Al registrarse, auto-aceptar la invitación
     d. Emitir evento CounterpartyInvited
```

### ApproveDeliveryHandler (UC-21) — Handler Crítico

```
ApproveDeliveryCommand:
  - escrowId: EscrowOperationId
  - userId: UserId

Flujo:
  1. Validation → escrowId existe, usuario es el buyer
  2. Authorization → solo el buyer de esta operación
  3. Handler → transacción:
     a. Cargar EscrowOperation (repository.findById)
     b. Validar estado: debe ser IN_REVIEW
     c. Validar que el userId == buyerId de la operación
     d. Si hay milestones:
        - Aceptar milestone actual
        - Si es el último milestone: estado → APPROVED
        - Si hay más milestones: estado → IN_EXECUTION (esperando siguiente hito)
     e. Si es operación simple: estado → APPROVED
     f. Persistir operación
     g. Emitir evento DeliveryApproved
  4. Evento → DeliveryApproved → ReleaseFundsListener inicia liberación
```

### CancelOperationHandler (UC-24) — Saga

```
CancelOperationSaga:
  Paso 1: IniciarCancelacionCommand
    - Validar reglas de cancelación según estado:
      - CREATED / NEGOTIATION: cancelación inmediata, sin cargos
      - AGREED / DEPOSIT_PENDING: cargos administrativos mínimos
      - DEPOSITED / IN_EXECUTION: aplicar penalización según contrato
      - DELIVERED / IN_REVIEW / APPROVED: no cancelable, solo disputa
      - COMPLETED / CANCELLED: error (operación ya finalizada)

  Paso 2: Si hay fondos depositados → ApplyCancellationPenaltyCommand
    - Calcular penalización
    - Descontar de fondos retenidos
    - El remanente se devuelve al buyer

  Paso 3: RefundBuyerCommand
    - Transferir fondos (o devolver activo) al buyer

  Paso 4: MarkOperationCancelledCommand
    - Cambiar estado a CANCELLED
    - Emitir evento OperationCancelled
```

---

# 6. Módulo: Disputas y Arbitraje

## 6.1 Entidades de Dominio

```
Dispute (Aggregate Root)
  - id: DisputeId
  - escrowId: EscrowOperationId
  - openedBy: UserId
  - reason: DisputeReason (quality | incomplete | defective | not_delivered | late | other)
  - status: DisputeState (open | negotiation | arbitration | resolved)
  - evidence: Evidence[]
  - arbitrationCaseId: ArbitrationCaseId?
  - resolution: Resolution?
  - createdAt: DateTime

DisputeState (sealed enum):
  OPEN → NEGOTIATION → ARBITRATION → RESOLVED
                           → CLOSED (si se retira la disputa)

Evidence (Entity)
  - id: EvidenceId
  - submittedBy: UserId
  - type: EvidenceType (document | image | video | expert_report)
  - fileRef: FileReference
  - description: string
  - submittedAt: DateTime

ArbitrationCase (Aggregate Root)
  - id: ArbitrationCaseId
  - disputeId: DisputeId
  - arbitratorId: UserId
  - status: ArbitrationStatus (assigned | reviewing | decided | appealed)
  - decision: ArbitrationDecision?
  - appealOf?: ArbitrationCaseId

Resolution (Value Object)
  - type: ResolutionType (release_to_seller | refund_to_buyer | partial_release | partial_refund)
  - sellerPercentage: Percentage
  - buyerPercentage: Percentage
  - motivation: string
  - arbitratorId: UserId
  - resolvedAt: DateTime
```

## 6.2 Handlers Destacados

### IssueResolutionHandler (UC-33)

```
IssueResolutionCommand:
  - arbitrationCaseId: ArbitrationCaseId
  - arbitratorId: UserId
  - decision: ArbitrationDecision (tipo de resolución + porcentajes)
  - motivation: string

Flujo:
  1. Authorization → solo el árbitro asignado al caso
  2. Handler:
     a. Cargar ArbitrationCase → validar estado "reviewing"
     b. Cargar Dispute asociada
     c. Cargar EscrowOperation
     d. Validar que la decisión es legal (porcentajes suman 100%)
     e. Crear Resolution
     f. Cambiar ArbitrationCase → "decided"
     g. Cambiar Dispute → "resolved"
     h. Emitir evento DisputeResolved
  3. Evento → DisputeResolved → ExecuteResolutionListener ejecuta la liberación
```

### ExecuteResolutionHandler (UC-34)

```
ExecuteResolutionCommand:
  - disputeId: DisputeId

Flujo:
  1. Handler (transacción):
     a. Cargar Dispute + Resolution
     b. Cargar EscrowOperation
     c. Si resolution.type == "release_to_seller":
        - Release total al seller
     d. Si "refund_to_buyer":
        - Devolver total al buyer
     e. Si "partial_release":
        - X% al seller, Y% al buyer
     f. Descontar comisión de arbitraje (según reglas de quién paga)
     g. Cambiar EscrowOperation → RESOLVED → COMPLETED
     h. Persistir todo en una transacción
     i. Emitir DisputeExecuted + OperationCompleted
```

---

# 7. Módulo: Pagos, Wallet y Facturación

## 7.1 Entidades de Dominio

```
Wallet (Aggregate Root)
  - id: WalletId
  - userId: UserId
  - currency: Currency
  - availableBalance: MoneyAmount
  - heldBalance: MoneyAmount   (fondos retenidos en Escrow activos)
  - totalBalance: MoneyAmount  (available + held)
  - version: int (optimistic concurrency)

Transaction (Entity)
  - id: TransactionId
  - walletId: WalletId
  - type: TransactionType (deposit | withdrawal | escrow_deposit | escrow_release | fee | refund)
  - amount: MoneyAmount
  - balanceBefore: MoneyAmount
  - balanceAfter: MoneyAmount
  - referenceType: string (escrow, fee, dispute, etc.)
  - referenceId: string (ID de la entidad relacionada)
  - status: TransactionStatus (pending | confirmed | failed)
  - externalRef?: string (referencia del gateway de pago)
  - createdAt: DateTime
```

## 7.2 Handler Destacado: WithdrawFromWalletHandler (UC-37)

Este handler es un ejemplo de un comando con **concurrencia optimista**:

```
WithdrawFromWalletCommand:
  - userId: UserId
  - amount: Money
  - currency: Currency
  - destinationBankAccount: BankAccountRef

Flujo:
  1. Authorization → usuario propietario de la wallet
  2. Handler (transacción con versión):
     a. Cargar Wallet por userId (SELECT ... FOR UPDATE)
     b. Validar: availableBalance >= amount + fee
     c. Verificar límites de retiro según nivel de verificación
     d. Si aplica, verificar 2FA
     e. Calcular fee de retiro
     f. Debitar availableBalance (amount + fee)
     g. Crear Transaction (status: pending)
     h. Persistir Wallet + Transaction (misma transacción)
     i. Emitir WithdrawalInitiated
  3. Evento → WithdrawalInitiated → ejecutar transferencia bancaria real (asíncrono)
     a. Gateway de pagos procesa
     b. Si éxito: Transaction → confirmed
     c. Si falla: Transaction → failed, revertir saldo
```

---

# 8. Módulo: Administración y Configuración

## 8.1 Entidades de Dominio

```
SystemParameter (Entity)
  - key: string (ej. "review.default_days", "kyc.max_daily_attempts")
  - value: JSON
  - type: ParameterType (int | string | decimal | json | duration)
  - scope: ParameterScope (global | by_asset_type | by_verification_level)
  - updatedBy: UserId
  - updatedAt: DateTime

FeeSchedule (Aggregate Root)
  - id: FeeScheduleId
  - name: string
  - rules: FeeRule[]
  - isActive: boolean
  - validFrom: DateTime
  - validTo?: DateTime

FeeRule (Value Object)
  - assetType: AssetType
  - minAmount?: Money
  - maxAmount?: Money
  - feeType: FeeType (percentage | fixed | tiered)
  - percentage?: Percentage
  - fixedAmount?: Money
  - tiers?: FeeTier[]
```

## 8.2 Handler: GenerateReportHandler (UC-48)

Las consultas pesadas no se ejecutan en línea. Usan el patrón **asynchronous query**:

```
GenerateReportCommand:
  - reportType: ReportType (volume | revenue | disputes | users | compliance)
  - filters: ReportFilters
  - format: ExportFormat (csv | pdf | xlsx)

Flujo:
  1. Handler:
     a. Validar que el tipo de reporte existe
     b. Crear ReportJob (entity) en estado "pending"
     c. Guardar en BD
     d. Emitir ReportGenerationRequested
     e. Retornar jobId + URL de consulta de estado
  2. Evento → ReportGenerationRequested → ReportWorker:
     a. Ejecutar consulta contra read replica
     b. Generar archivo en S3
     c. Actualizar ReportJob → "completed" con URL de descarga
     d. Notificar al usuario
```

---

# 9. Módulo: Notificaciones y Mensajería

## 9.1 Entidades de Dominio

```
Message (Aggregate Root)
  - id: MessageId
  - escrowId: EscrowOperationId
  - senderId: UserId
  - content: string
  - type: MessageType (text | file | system)
  - attachments: FileReference[]
  - readBy: UserId[]  (marcar como leído)
  - createdAt: DateTime

NotificationPreference (Entity)
  - userId: UserId
  - channels: Map<NotificationChannel, boolean>
  - events: Map<NotificationEventType, NotificationChannel[]>

Notification (Entity)
  - id: NotificationId
  - userId: UserId
  - type: NotificationEventType
  - channel: NotificationChannel (email | sms | push | in_app)
  - title: string
  - body: string
  - data: JSON (payload contextual)
  - status: NotificationStatus (queued | sent | delivered | read | failed)
  - createdAt: DateTime
```

## 9.2 Event Listeners de Notificaciones

Las notificaciones no se envían desde los handlers; se disparan desde event listeners después del commit:

```
// Escuchá eventos de dominio y crea notificaciones
class EscrowNotificationListener {
  @on(OperationCreated)
  async handle(event: OperationCreated) {
    await this.notificationService.notify(event.buyerId, {
      type: 'operation_created',
      channel: ['email', 'push'],
      title: 'Operación creada exitosamente',
      data: { operationId: event.operationId }
    })
  }
  
  @on(AssetReleased)
  async handle(event: AssetReleased) {
    // Notifica al seller que recibió los fondos
    await this.notificationService.notify(event.sellerId, {
      type: 'payment_received',
      channel: ['email', 'push', 'sms'],
      title: '¡Fondos liberados!',
      data: { operationId: event.operationId, amount: event.amount }
    })
    // Notifica al buyer que la operación se completó
    await this.notificationService.notify(event.buyerId, {
      type: 'operation_completed',
      channel: ['email'],
      title: 'Operación completada',
      data: { operationId: event.operationId }
    })
  }
}
```

---

# 10. Módulo: API e Integraciones

## 10.1 Arquitectura de la API Pública

La API pública es idéntica a la API interna en términos de handlers. La diferencia está en el adaptador de entrada:

```
API Pública (REST / GraphQL)         API Interna (Web controllers / CLIs)
         │                                      │
         └───────────── UseCaseBus ──────────────┘
                              │
                    ┌─────────▼─────────┐
                    │   Pipeline (same)  │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │   Handlers (same)  │
                    └───────────────────┘
```

Esto garantiza que las reglas de negocio son exactamente las mismas sin importar el canal de entrada.

## 10.2 Adaptadores de Entrada

| Canal | Adaptador | Handler Reutilizado |
|---|---|---|
| Web app (Next.js) | `ApiController → UseCaseBus.send()` | Todos los handlers |
| Mobile app (React Native) | `ApiClient → API Gateway → ApiController` | Ídem (a través de API pública) |
| API REST pública | `Express/Fastify Router → AuthMiddleware → UseCaseBus.send()` | Todos los handlers |
| Webhooks entrantes | `WebhookController → valida firma → UseCaseBus.send()` | ConfirmAssetReceptionHandler |
| CLI (admin) | `CLICommand → UseCaseBus.send()` | Admin handlers |
| Scheduled Jobs | `CronJob → UseCaseBus.send()` | AutoReleaseHandler |

## 10.3 Webhooks Salientes (UC-61)

```
WebhookRegistration:
  - id: WebhookId
  - url: string (endpoint HTTPS)
  - events: WebhookEventType[]
  - secret: string (firma HMAC-SHA256)
  - status: WebhookStatus (active | paused | disabled)
  - retryConfig: { maxRetries: 3, backoffSeconds: 60 }

Flujo de entrega:
  1. Un evento de dominio se publica en el EventBus interno
  2. WebhookDispatcher escuchá todos los eventos relevantes
  3. Para cada webhook registrado que coincida con el tipo de evento:
     a. Construir payload JSON
     b. Firmar con HMAC-SHA256 (secret del webhook)
     c. HTTP POST a la URL configurada
     d. Si falla: reintentar con backoff (hasta maxRetries)
     e. Si todos los reintentos fallan: marcar webhook como "paused"
     f. Notificar al desarrollador
```

---

# 11. Módulo: Auditoría y Cumplimiento

## 11.1 Registro Inmutable (Audit Log)

Cada handler que muta el estado emite su evento de auditoría mediante el contexto:

```
class ReleaseAssetHandler {
  async handle(command: ReleaseAssetCommand, ctx: ExecutionContext): Promise<Result<ReleaseId>> {
    // ... lógica ...
    ctx.emitAuditEvent({
      type: 'ASSET_RELEASED',
      actorId: ctx.identity.id,
      resourceType: 'escrow_operation',
      resourceId: command.escrowId,
      data: { amount: operation.amount, milestone: currentMilestone },
      ip: ctx.metadata.get('ip'),
      userAgent: ctx.metadata.get('userAgent')
    })
  }
}
```

El `AuditEvent` se encadena ya que cada evento contiene el hash SHA-256 del evento anterior:

```
AuditEvent {
  id: string (UUID v7)
  sequence: number (secuencia global o por agregado)
  eventType: string
  timestamp: DateTime
  actorId: string
  resourceType: string
  resourceId: string
  action: string
  data: JSON
  metadata: { ip, userAgent, correlationId }
  previousHash: string (hash SHA-256 del AuditEvent anterior)
  currentHash: string (hash SHA-256 de este evento)
  digitalSignature: string (firmado con clave del sistema)
}
```

## 11.2 AML Monitoring (UC-66)

```
FlagSuspiciousTransactionHandler:
  - Se dispara por evento (TransactionCreated) o por reglas programadas
  - Evalúa reglas AML configurables:
    a. Monto > umbral por tipo de operación
    b. Múltiples operaciones en ventana de tiempo (velocity)
    c. País de alto riesgo involucrado
    d. Usuario nuevo con operaciones de alto valor
    e. Patrón de redondeo consistente (structuring)
    f. Contrapartes en listas de sanciones

  - Cada regla produce un score parcial
  - Score total > threshold → genera alerta
  - Alerta severa (score alto) → bloquea operación + notifica compliance
  - Alerta media → notifica compliance sin bloquear
  - Registro en tabla AML_ALERTS para revisión

  - Emite evento SuspiciousActivityDetected
```

---

# 12. Entidades y Aggregate Roots del Dominio

## 12.1 Catálogo de Aggregate Roots

| Aggregate Root | Contexto | ID | Comentario |
|---|---|---|---|
| `User` | Identity | UserId | Persona física o moral |
| `Company` | Identity | UserId | Extensión de User |
| `UserKYC` | Identity | UserId | Proceso de verificación |
| `EscrowOperation` | Escrow | EscrowOperationId | Aggregate central del sistema |
| `Agreement` | Escrow | AgreementId | Contrato firmado |
| `Wallet` | Wallet | WalletId | Una wallet por moneda por usuario |
| `Transaction` | Wallet | TransactionId | Movimiento financiero |
| `Dispute` | Dispute | DisputeId | Desacuerdo entre partes |
| `ArbitrationCase` | Dispute | ArbitrationCaseId | Proceso formal de arbitraje |
| `Message` | Messaging | MessageId | Mensaje en chat de operación |
| `Notification` | Messaging | NotificationId | Notificación enviada |
| `FeeSchedule` | Admin | FeeScheduleId | Configuración de comisiones |
| `WebhookRegistration` | Integration | WebhookId | Webhook de integrador |

## 12.2 Reglas de Consistencia entre Aggregates

| Regla | Agregados Involucrados | Cómo se Garantiza |
|---|---|---|
| Una operación no puede tener más de un dispute activo | EscrowOperation, Dispute | El handler verifica `operation.disputeId == null` antes de crear Dispute |
| No liberar sin fondos suficientes | EscrowOperation, Wallet, Transaction | Todo en una transacción: debitar wallet + liberar + registrar |
| Un usuario no puede ser buyer y seller de la misma operación | EscrowOperation | Validación en CreateEscrowHandler |
| El saldo disponible nunca puede ser negativo | Wallet | Transacción con CHECK CONSTRAINT o validación en handler |
| Un árbitro no puede haber participado como parte en la operación | ArbitrationCase, EscrowOperation | Validación en EscalateToArbitrationHandler |

---

# 13. Puertos de Dominio (Repositories)

## 13.1 Puertos Definidos por el Cliente

Cada aggregate root tiene su propio puerto de repositorio:

```
// Identity context
interface UserRepository {
  findById(id: UserId): Promise<User | null>
  findByEmail(email: Email): Promise<User | null>
  save(user: User): Promise<void>
  softDelete(id: UserId): Promise<void>
}

// Escrow context
interface EscrowOperationRepository {
  findById(id: EscrowOperationId): Promise<EscrowOperation | null>
  findByBuyer(buyerId: UserId, pagination: Pagination): Promise<EscrowOperation[]>
  findBySeller(sellerId: UserId, pagination: Pagination): Promise<EscrowOperation[]>
  findByStatus(status: EscrowState, pagination: Pagination): Promise<EscrowOperation[]>
  save(operation: EscrowOperation): Promise<void>
  delete(id: EscrowOperationId): Promise<void>
}

// Wallet context
interface WalletRepository {
  findByUserAndCurrency(userId: UserId, currency: Currency): Promise<Wallet | null>
  save(wallet: Wallet): Promise<void>
}

// Dispute context
interface DisputeRepository {
  findById(id: DisputeId): Promise<Dispute | null>
  findByEscrow(escrowId: EscrowOperationId): Promise<Dispute | null>
  save(dispute: Dispute): Promise<void>
}
```

## 13.2 Puertos de Servicios Externos

```
interface PaymentGateway {
  deposit(userId: UserId, amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult>
  withdraw(userId: UserId, amount: Money, destination: BankAccount): Promise<PaymentResult>
  refund(transactionRef: string): Promise<PaymentResult>
}

interface AssetVerificationService {
  verifyDocument(hash: string, metadata: DocumentMetadata): Promise<VerificationResult>
  verifyDomainTransfer(domain: string, newOwner: string): Promise<VerificationResult>
  verifyNFTTransfer(tokenId: string, newOwner: string): Promise<VerificationResult>
}

interface KYBProvider {
  verifyCompany(taxId: string, documents: DocumentRef[]): Promise<KYBResult>
}

interface ElectronicSignatureProvider {
  sendForSignature(signers: Signer[], document: Document): Promise<SignatureRequest>
  verifySignature(signatureId: string): Promise<SignatureVerificationResult>
}

interface EmailService {
  send(params: { to: string; template: string; data: any }): Promise<void>
}

interface SMSService {
  send(params: { to: string; message: string }): Promise<void>
}

interface PushNotificationService {
  send(params: { userId: UserId; title: string; body: string; data?: any }): Promise<void>
}
```

---

# 14. Conexión con el Framework

## 14.1 Wiring del Cliente

```
// Configuración del framework con adaptadores específicos de Escrow
const framework = UseCaseFramework.create({
  ports: {
    unitOfWork: new PostgresUnitOfWork(pool),
    eventBus: new KafkaEventBus(kafka, { 
      // Eventos de dominio → tópicos de Kafka
      topicResolver: (event) => `escrow.${event.eventType.toLowerCase()}`
    }),
    authorizationService: new EscrowAuthorizationService(
      roleRepository,
      policyRepository,
      operationRepository  // para ABAC: verificar ownership
    ),
    identityProvider: new JWTIdentityProvider(jwtSecret),
    clock: new SystemClock(),
    idGenerator: new UlidGenerator(),
    logger: new PinoLogger({ level: process.env.LOG_LEVEL }),
    metricsRecorder: new PrometheusMetricsRecorder(),
    validator: new ZodValidator(),  // validador concreto
  }
})

// Registro de handlers
// Identity
framework.registerCommand(CreateUserCommand, new CreateUserHandler(userRepo, passwordHasher, eventBus))
framework.registerCommand(SubmitKYCDocumentsCommand, new SubmitKYCDocumentsHandler(kycRepo, fileStorage))

// Escrow Core
framework.registerCommand(CreateEscrowCommand, new CreateEscrowHandler(escrowRepo, userRepo))
framework.registerCommand(ApproveDeliveryCommand, new ApproveDeliveryHandler(escrowRepo, walletRepo))
framework.registerCommand(CancelOperationCommand, new CancelOperationHandler(escrowRepo))

// Disputas
framework.registerCommand(OpenDisputeCommand, new OpenDisputeHandler(disputeRepo, escrowRepo))
framework.registerCommand(IssueResolutionCommand, new IssueResolutionHandler(arbitrationRepo, disputeRepo, escrowRepo))
framework.registerCommand(ExecuteResolutionCommand, new ExecuteResolutionHandler(disputeRepo, escrowRepo, walletRepo))

// Wallet
framework.registerCommand(DepositToWalletCommand, new DepositToWalletHandler(walletRepo, paymentGateway))
framework.registerCommand(WithdrawFromWalletCommand, new WithdrawFromWalletHandler(walletRepo, paymentGateway))

// ... resto de handlers
```

## 14.2 Configuración del Pipeline Específico para Escrow

```
framework.configure({
  pipeline: {
    // Pipeline estándar
    order: ['logging', 'validation', 'authorization', 'transaction', 'events', 'metrics'],
    
    // Behaviors custom de Escrow
    customBehaviors: [
      {
        name: 'rateLimit',
        behavior: new RateLimitBehavior({
          rules: [
            { command: 'CreateEscrowCommand', maxRequests: 10, windowMs: 60000 },
            { command: 'DepositAssetCommand', maxRequests: 5, windowMs: 60000 },
            { command: 'OpenDisputeCommand', maxRequests: 3, windowMs: 3600000 },
          ]
        }),
        position: { before: 'validation' }
      },
      {
        name: 'idempotency',
        behavior: new IdempotencyBehavior({
          ttl: Duration.ofHours(24),
          keyExtractor: (command) => `${command.constructor.name}:${command.idempotencyKey}`
        }),
        position: { before: 'logging' }
      },
      {
        name: 'audit',
        behavior: new AuditBehavior(auditLogRepository),
        position: { after: 'transaction' }
      }
    ]
  }
})
```

---

# 15. Configuración del Pipeline para Escrow

## 15.1 Behaviors Transversales Específicos

### IdempotencyBehavior

Crítico para operaciones financieras. Si un comando se envía dos veces con el mismo `idempotencyKey`, el framework retorna el resultado previo sin ejecutar el handler:

**Aplicación en Escrow**: `DepositAssetCommand`, `ApproveDeliveryCommand`, `ReleaseAssetCommand` — cualquier comando que mueva fondos.

### RateLimitBehavior

Protege contra abusos:

| Comando | Límite | Ventana |
|---|---|---|
| CreateEscrowCommand | 10 | 1 minuto |
| DepositAssetCommand | 5 | 1 minuto |
| OpenDisputeCommand | 3 | 1 hora |
| GeneratePaymentLinkCommand | 20 | 5 minutos |
| LoginHandler | 5 | 1 minuto (por IP) |

### AuditBehavior

Cada comando que muta el estado del sistema genera un registro de auditoría con hash encadenado. Este behavior se sitúa después de `transaction` para garantizar que solo registra operaciones exitosas.

### TenancyBehavior

Si la plataforma opera multi-tenant (varias empresas usando la plataforma), este behavior inyecta el `tenantId` en el ExecutionContext y filtra consultas automáticamente.

---

# 16. Handlers Compuestos y Sagas

## 16.1 Saga: Flujo de Liberación (ReleaseFlowSaga)

```
ReleaseFlowSaga:
  Input: EscrowOperationId (disparado por evento DeliveryApproved)

  Paso 1: ValidateReleaseConditionsQuery
    - Verificar que la operación está en estado APPROVED o RESOLVED
    - Verificar que no hay disputas activas
    - Verificar que los fondos están disponibles en la wallet retenida

  Paso 2: CalculateReleaseAmountCommand
    - Si es hito: liberar monto del milestone
    - Si es liberación total: liberar el saldo retenido completo
    - Si es resolución de disputa: liberar según porcentajes de la resolución

  Paso 3: DebitHeldBalanceCommand (Wallet context)
    - Debitar de heldBalance en la wallet del buyer
    - NO acreditar todavía al seller (la wallet del seller no se toca aquí)
    - La transacción queda en estado "settling"

  Paso 4: ExecuteExternalTransferCommand (si aplica)
    - Si el activo es fiduciario o cripto: iniciar transferencia bancaria o blockchain
    - Si es documento: liberar acceso al documento
    - Si es bien físico: notificar liberación al custodio
    - Registrar referencia externa

  Paso 5: CreditSellerWalletCommand (Wallet context)
    - Acreditar el monto en la wallet del seller
    - La transacción pasa a "completed"

  Paso 6: DeductFeeCommand (Wallet context)
    - Calcular comisión de la plataforma
    - Debitar comisión de la wallet del pagador (buyer, seller, o dividido)
    - Acreditar a la wallet de ingresos de la plataforma

  Paso 7: CompleteOperationCommand (Escrow context)
    - Cambiar estado a COMPLETED
    - Emitir OperationCompleted

  Compensaciones:
    - Si Paso 4 falla: revertir Paso 3 (re-creditar heldBalance)
    - Si Paso 5 falla después de Paso 4: revertir Paso 4 y Paso 3
    - Si Paso 6 falla: log + alerta (no reversión, la comisión se concilia después)
```

## 16.2 Saga: Creación de Operación (CreateEscrowFlowSaga)

```
CreateEscrowFlowSaga:
  Input: CreateEscrowCommand

  Paso 1: CreateEscrowHandler
    - Crea operación en estado CREATED
    - Si contraparte existe: estado → AGREED
    - Si no existe: estado → CREATED (esperando registro)
    - Emite OperationCreated

  Paso 2 (condicional): InviteCounterpartyHandler (si contraparte no registrada)
    - Envía invitación por email
    - Inicia temporizador: si no se registra en 7 días, cancelar automáticamente

  Paso 3 (condicional): NegotiateTermsHandler (si se requiere negociación)
    - Flujo de ida y vuelta: propuesta → contrapropuesta → aceptación
    - Cada cambio genera una nueva versión del acuerdo

  Paso 4: SignAgreementHandler
    - Ambas partes firman
    - El contrato se almacena en S3 con hash
    - Estado → AGREED
    - Emite AgreementSigned

  Paso 5 (condicional): Esperar depósito (estado DEPOSIT_PENDING)
    - El comprador tiene un plazo para depositar
    - Si expira: CancelOperationHandler automático
    - Cuando deposita: DepositAssetHandler → ConfirmAssetReceptionHandler → estado DEPOSITED

  Output: EscrowOperationId
```

---

# 17. Adaptadores de Infraestructura

## 17.1 Base de Datos

### PostgreSQL — Adaptadores

| Puerto | Adaptador | Tabla Principal |
|---|---|---|
| `UserRepository` | `PostgresUserRepository` | `users` |
| `EscrowOperationRepository` | `PostgresEscrowRepository` | `escrow_operations`, `milestones` |
| `WalletRepository` | `PostgresWalletRepository` | `wallets`, `transactions` |
| `DisputeRepository` | `PostgresDisputeRepository` | `disputes`, `evidence`, `arbitration_cases` |
| `MessageRepository` | `PostgresMessageRepository` | `messages` |
| `AuditLogRepository` | `PostgresAuditLogRepository` | `audit_events` (append-only, hash-encadenado) |

### MongoDB — Adaptadores

| Puerto | Adaptador | Colección |
|---|---|---|
| `NotificationRepository` | `MongoNotificationRepository` | `notifications` |
| `ContractRepository` | `MongoContractRepository` | `contracts` (documentos variables, múltiples versiones) |

## 17.2 Event Bus — Kafka

| Tópico | Eventos | Consumidores |
|---|---|---|
| `escrow.operation.created` | OperationCreated | NotificationListener, AuditListener |
| `escrow.asset.deposited` | AssetDeposited | EscrowListener (avanza estado), NotificationListener |
| `escrow.asset.released` | AssetReleased | WalletListener, NotificationListener, WebhookDispatcher |
| `escrow.dispute.opened` | DisputeOpened | ArbitrationListener, NotificationListener, IAListener |
| `escrow.dispute.resolved` | DisputeResolved | EscrowListener (ejecuta resolución), WebhookDispatcher |
| `identity.user.verified` | UserVerified | NotificationListener, EscrowListener (recalcula límites) |
| `compliance.aml.alert` | SuspiciousActivityDetected | ComplianceListener (genera STR) |

## 17.3 Cache — Redis

| Clave | Valor | TTL | Propósito |
|---|---|---|---|
| `session:{userId}` | SessionData | 15 min | Sesiones de usuario |
| `operation:{id}` | EscrowOperation (JSON) | 5 min | Caché de detalle de operación |
| `user:{id}:balance` | BalanceData | 1 min | Saldo visible (el real siempre es de BD) |
| `rate:limit:{command}:{userId}` | Counter | variable | Rate limiting |
| `idempotency:{key}` | Result (JSON) | 24 h | Idempotencia de comandos críticos |

## 17.4 File Storage — S3 / MinIO

| Bucket | Propósito | Retención | Cifrado |
|---|---|---|---|
| `escrow-documents` | Contratos firmados, evidencias, documentos KYC | 10 años | AES-256 |
| `escrow-audit-log` | Registro de auditoría inmutable | 10 años | AES-256 + Object Lock |
| `escrow-temp` | Archivos temporales (carga en proceso) | 24 h | AES-256 |
| `escrow-reports` | Reportes generados | 90 días | AES-256 |

## 17.5 Servicios Externos (adaptadores)

```
PaymentGatewayAdapter (implementa PaymentGateway):
  - StripeAdapter: tarjetas, ACH, transferencias internacionales
  - MercadoPagoAdapter: SPEI, Pix, OXXO, tarjetas LATAM
  - CryptoAdapter: Fireblocks/BitGo para cripto

KYBProviderAdapter (implementa KYBProvider):
  - PersonaAdapter: KYC/KYB global
  - EncodeIDAdapter: KYC/KYB LATAM (CURP, RFC, actas)

SignatureProviderAdapter (implementa ElectronicSignatureProvider):
  - DocuSignAdapter: firma global
  - LexiaAdapter: firma con e.firma SAT (México)

NotificationAdapter:
  - SendGridAdapter: email transaccional
  - TwilioAdapter: SMS + WhatsApp
  - FirebaseAdapter: push notifications
```

---

# 18. Pruebas del Cliente

## 18.1 Estrategia de Pruebas

La aplicación Escrow se prueba en 4 niveles:

| Nivel | Qué se Prueba | Cómo | Tiempo |
|---|---|---|---|
| **Unitarias** | Cada handler con puertos mockeados | Jest/Vitest + ts-jest | < 10ms |
| **Integración** | Handler + adaptadores reales (PostgreSQL testcontainer, Kafka mock) | Testcontainers + Docker | 1-5s |
| **Saga** | Flujo completo multi-handler con eventos | SagaTestHarness del framework | 5-30s |
| **E2E** | API completa desde controller hasta BD | Playwright + API tests | 10-60s |

## 18.2 Test de Integración: CreateEscrowHandler

```
describe('CreateEscrowHandler Integration', () => {
  let escrowRepo: PostgresEscrowRepository
  let userRepo: PostgresUserRepository
  let handler: CreateEscrowHandler
  let datasource: DataSource
  
  beforeAll(async () => {
    datasource = await startPostgresContainer()
    escrowRepo = new PostgresEscrowRepository(datasource)
    userRepo = new PostgresUserRepository(datasource)
    handler = new CreateEscrowHandler(escrowRepo, userRepo)
  })
  
  afterAll(async () => {
    await datasource.destroy()
  })
  
  it('should create an escrow operation', async () => {
    // Arrange
    const buyer = await createTestUser(userRepo)
    const seller = await createTestUser(userRepo)
    
    const command = new CreateEscrowCommand({
      buyerId: buyer.id,
      sellerEmail: seller.email,
      assetType: 'money',
      amount: Money.of(1000, 'USD'),
      conditions: { reviewDays: 5 }
    })
    
    // Act
    const result = await handler.handle(command, testContext(buyer))
    
    // Assert
    expect(result.isSuccess()).toBe(true)
    const operation = await escrowRepo.findById(result.value)
    expect(operation).not.toBeNull()
    expect(operation!.status).toBe('AGREED')
  })
})
```

## 18.3 Test de Saga: ReleaseFlowSaga

```
describe('ReleaseFlowSaga', () => {
  it('should complete full release flow', async () => {
    const { saga, bus, escrowRepo, walletRepo } = createSagaTestHarness()
    
    // Preparar: crear operación con fondos depositados
    const escrowId = await givenCompleteOperation(bus)
    
    // Act: ejecutar approve delivery
    const command = new ApproveDeliveryCommand({ escrowId, userId: buyerId })
    const result = await bus.send(command)
    
    // Assert: la saga completa liberación, comisión y cierre
    expect(result.isSuccess()).toBe(true)
    
    const operation = await escrowRepo.findById(escrowId)
    expect(operation!.status).toBe('COMPLETED')
    
    const sellerWallet = await walletRepo.findByUserAndCurrency(sellerId, 'USD')
    expect(sellerWallet!.availableBalance.amount).toBe(970)  // 1000 - 3% fee
  })
})
```

## 18.4 Pruebas de Seguridad

```
describe('Authorization', () => {
  it('should reject approve delivery when user is not the buyer', async () => {
    const result = await bus.send(new ApproveDeliveryCommand({
      escrowId: escrowId,
      userId: otherUserId  // un usuario que no es el buyer
    }))
    
    expect(result.isFailure()).toBe(true)
    expect(result.error.code).toBe('AUTH_NOT_OWNER')
  })
  
  it('should reject withdrawal when wallet balance is insufficient', async () => {
    const result = await bus.send(new WithdrawFromWalletCommand({
      userId: poorUser.id,
      amount: Money.of(999999, 'USD')
    }))
    
    expect(result.isFailure()).toBe(true)
    expect(result.error.code).toBe('BUSINESS_INSUFFICIENT_FUNDS')
  })
})
```

---

*Este documento describe la arquitectura concreta del cliente Escrow que implementa los 66 casos de uso de la Sección 4 de ARQUITECTURA.md usando el Use Case Framework definido en USE_CASE_FRAMEWORK.md. Los adaptadores de infraestructura reales (PostgreSQL, Kafka, Redis, S3, Stripe, etc.) se implementan como adaptadores concretos de los puertos definidos en este documento.*
