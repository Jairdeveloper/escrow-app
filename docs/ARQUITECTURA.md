# Especificación Arquitectónica — Plataforma de Escrow Inteligente

**Versión:** 1.0  
**Clasificación:** Confidencial — Arquitectura de Sistema  
**Propósito:** Documento base para desarrollo de plataforma de depósito en garantía (Escrow) multi-activo, multi-escenario.  
**Autor:** Arquitecto Principal  

---

## Índice

1. [Visión del Producto](#1-visión-del-producto)
2. [Concepto de Escrow](#2-concepto-de-escrow)
3. [Actores del Sistema](#3-actores-del-sistema)
4. [Casos de Uso](#4-casos-de-uso)
5. [Requerimientos Funcionales](#5-requerimientos-funcionales)
6. [Requerimientos No Funcionales](#6-requerimientos-no-funcionales)
7. [Flujo de una Operación Escrow](#7-flujo-de-una-operación-escrow)
8. [Arquitectura Funcional — Módulos](#8-arquitectura-funcional--módulos)
9. [Modelo de Datos Conceptual](#9-modelo-de-datos-conceptual)
10. [Seguridad](#10-seguridad)
11. [Arquitectura Técnica](#11-arquitectura-técnica)
12. [Integraciones Externas](#12-integraciones-externas)
13. [Inteligencia Artificial](#13-inteligencia-artificial)
14. [Riesgos](#14-riesgos)
15. [Roadmap](#15-roadmap)
16. [Recomendaciones Técnicas](#16-recomendaciones-técnicas)
17. [Funcionalidades Futuras](#17-funcionalidades-futuras)

---

# 1. Visión del Producto

## 1.1 Propósito

Diseñar y construir una plataforma de Escrow (depósito en garantía) neutral, programable y multi-activo que actúe como tercera parte de confianza en transacciones entre partes que no confían plenamente entre sí. La plataforma retiene, protege, administra y libera activos únicamente cuando se cumplen las condiciones previamente acordadas, eliminando el riesgo de incumplimiento en cualquier tipo de transacción.

## 1.2 Problema que Resuelve

Toda transacción entre dos partes que no tienen una relación de confianza plena enfrenta un problema fundamental: **quién cumple primero**. El comprador teme pagar y no recibir; el vendedor teme entregar y no cobrar. Este dilema:

- Bloquea transacciones de alto valor (inmuebles, vehículos, dominios, propiedad intelectual).
- Genera costosos contratos legales y litigios.
- Excluye a pequeñas empresas y personas de mercados que requieren confianza.
- Limita el comercio internacional donde las jurisdicciones dificultan el recurso legal.
- Fomenta el uso de intermediarios costosos e ineficientes (notarías, agentes, bancos).

La plataforma resuelve esto automatizando la retención y liberación condicionada de activos mediante contratos inteligentes programables, reduciendo drásticamente el riesgo, el costo y el tiempo de las transacciones.

## 1.3 Propuesta de Valor

| Problema | Solución |
|---|---|
| Desconfianza entre partes | Tercero neutral automatizado que retiene activos |
| Fraude en transacciones | Verificación de identidad (KYC/KYB), trazabilidad inmutable |
| Procesos lentos y burocráticos | Liberación condicionada automatizada |
| Costos elevados de intermediarios | Comisiones transparentes y competitivas |
| Falta de flexibilidad | Soporte multi-activo, multi-escenario |
| Sin acceso a mercados globales | Cumplimiento regulatorio multi-jurisdicción |
| Disputas no resueltas | Arbitraje integrado con resolución estructurada |

## 1.4 Ventajas Competitivas

- **Multi-activo**: Dinero fiduciario, criptomonedas, bienes digitales, documentos, propiedad intelectual, servicios profesionales, NFTs, licencias, dominios. No limitado a una clase de activo.
- **Multi-escenario**: Marketplace, freelance, inmuebles, vehículos, comercio internacional, B2B, crowdfunding, licitaciones. Una plataforma, infinitos casos de uso.
- **Programabilidad**: Condiciones de liberación arbitrarias definidas por acuerdo entre partes, no plantillas rígidas.
- **Arbitraje integrado**: Resolución de disputas dentro de la plataforma sin recurrir a sistemas externos.
- **IA nativa**: Detección de fraude, evaluación de riesgo, clasificación de disputas, asistentes virtuales, extracción documental.
- **Cumplimiento por diseño**: KYC/KYB, AML, listas de sanciones, registro inmutable, firmas digitales con validez legal.
- **API pública**: Integrable por terceros para construir sobre la plataforma (Embedded Escrow).

## 1.5 Oportunidades de Mercado

- **Freelance y economía gig**: Más de 1.5B de trabajadores independientes globalmente. Las plataformas existentes cobran comisiones del 10-20% y no ofrecen protección real.
- **Comercio internacional**: Las cartas de crédito bancarias son costosas y lentas. Un Escrow digital reduce costos 5-10x y acelera tiempos.
- **Activos digitales y NFTs**: Mercado en explosión con alta incidencia de fraude. Los compradores necesitan garantías.
- **Bienes raíces**: Las transacciones inmobiliarias implican semanas de proceso y altos costos de intermediación.
- **Contratos B2B**: Miles de millones en contratos entre empresas que operan sin garantías de cumplimiento.
- **Marketplaces verticales**: Plataformas de nicho (dominios, software, vehículos, propiedad intelectual) necesitan Escrow como funcionalidad core.
- **Crowdfunding**: Los patrocinadores necesitan garantías de que los fondos solo se liberan si se cumplen los hitos del proyecto.

---

# 2. Concepto de Escrow

## 2.1 Definición

Escrow (depósito en garantía) es un acuerdo legal y financiero en el que un tercero neutral retiene un activo hasta que se cumplen condiciones predefinidas entre dos o más partes. El activo retenido puede ser dinero, bienes, documentos, propiedad intelectual, software, dominios, o cualquier elemento de valor.

## 2.2 Funcionamiento General

```
Fase 1 — Acuerdo:     Partes negocian y firman condiciones
Fase 2 — Depósito:     Comprador deposita activo en Escrow
Fase 3 — Confirmación: Escrow confirma recepción y validez del activo
Fase 4 — Ejecución:    Vendedor cumple su obligación (entrega, servicio, etc.)
Fase 5 — Validación:   Comprador confirma conformidad (o expira período de revisión)
Fase 6 — Liberación:   Escrow libera activo al vendedor
  — O Disputa:         Si hay desacuerdo, se activa arbitraje
Fase 7 — Cierre:       Transacción completada, registro inmutable
```

## 2.3 Participantes y Responsabilidades

| Participante | Responsabilidad |
|---|---|
| **Comprador / Depositante** | Deposita el activo; recibe el bien/servicio |
| **Vendedor / Beneficiario** | Entrega el bien/servicio; recibe el activo |
| **Agente Escrow** | Custodia neutral, verifica condiciones, ejecuta liberación |
| **Árbitro** | Resuelve disputas cuando las partes no acuerdan |
| **Plataforma** | Sistema tecnológico que automatiza el proceso |
| **Auditor** | Verifica la integridad del proceso (interno o regulatorio) |
| **Regulador** | Supervisa cumplimiento normativo (opcional según jurisdicción) |

## 2.4 Flujo Completo (Detallado)

### 2.4.1 Escenario Estándar (Sin Disputa)

1. **Iniciación**: El comprador o la plataforma crea una operación Escrow con parámetros iniciales (activo, monto, condiciones, partes).
2. **Negociación**: Las partes negocian condiciones: precio, hitos, fechas, criterios de aceptación, período de revisión, reglas de arbitraje.
3. **Firma**: Las partes firman digitalmente el acuerdo Escrow.
4. **Depósito**: El comprador transfiere el activo a la wallet/account custodiada por la plataforma.
5. **Confirmación**: La plataforma verifica la recepción y validez del activo. Notifica al vendedor.
6. **Ejecución**: El vendedor cumple su obligación (entrega, presta servicio, transfiere propiedad).
7. **Período de Revisión**: El comprador tiene un período definido para examinar y aceptar.
8. **Aceptación Tácita**: Si el comprador no objeta dentro del período, se considera aceptación automática.
9. **Liberación**: La plataforma transfiere el activo al vendedor.
10. **Cierre**: La operación se marca como completada. Se genera certificado de cierre. Registro inmutable.

### 2.4.2 Escenario con Disputa

1-6. Igual que el escenario estándar.
7. **Rechazo / Objeción**: El comprador notifica que el bien/servicio no cumple condiciones.
8. **Negociación Directa**: Período para que comprador y vendedor resuelvan directamente.
9. **Escalación**: Si no hay acuerdo, cualquiera de las partes puede escalar a arbitraje.
10. **Designación de Árbitro**: Se asigna un árbitro (automático o por selección).
11. **Presentación de Evidencias**: Ambas partes presentan documentos, capturas, evidencias.
12. **Revisión**: El árbitro revisa el caso y las condiciones originales.
13. **Resolución**: El árbitro emite una decisión vinculante: liberar al vendedor, devolver al comprador, o liberación parcial.
14. **Ejecución de Resolución**: La plataforma ejecuta automáticamente la decisión.
15. **Cierre**: Operación cerrada. Registro inmutable incluyendo la resolución.

### 2.4.3 Escenario de Cancelación

1-3. Igual.
4. Cualquier parte solicita cancelación (según términos del contrato).
5. Si el activo no ha sido depositado, cancelación inmediata.
6. Si el activo está depositado, se aplican cargos por cancelación (definidos en contrato).
7. El activo se devuelve al comprador menos cargos aplicables.
8. Cierre de operación.

## 2.5 Modalidades de Escrow

| Modalidad | Descripción | Casos de Uso |
|---|---|---|
| **Escrow Simple** | Liberación única contra cumplimiento único | Compra/venta, freelance puntual |
| **Escrow por Hitos** | Liberaciones parciales contra hitos cumplidos | Desarrollo software, construcción, consultoría |
| **Escrow Recurrente** | Pagos periódicos retenidos y liberados condicionalmente | Servicios gestionados, SaaS, mantenimiento |
| **Escrow de Documentos** | Depósito de documentos + dinero; se liberan ambos simultáneamente | Transferencia de dominios, propiedad intelectual |
| **Escrow Subasta** | Puja retenida; ganador liberado al vendedor | Subastas, licitaciones |
| **Escrow Colaborativo** | Múltiples depositantes y múltiples beneficiarios | Crowdfunding, inversiones colectivas |
| **Escrow Condicional** | Liberación vinculada a eventos externos verificables (oráculos) | Comercio internacional, seguros, derivados |
| **Escroll Híbrido** | Combina activos de distinto tipo (dinero + documento + bien físico) | Compra inmueble, compra vehículo |

## 2.6 Ventajas del Modelo Escrow

- **Elimina el riesgo de contraparte**: Ninguna parte necesita confiar en la otra.
- **Transparencia total**: Todas las condiciones son visibles y verificables.
- **Automatización**: Las liberaciones ocurren automáticamente al cumplirse condiciones.
- **Traza completa**: Cada evento queda registrado de forma inmutable.
- **Flexibilidad**: Aplica a cualquier activo y cualquier condición.
- **Seguridad jurídica**: El acuerdo Escrow + firmas digitales tienen validez legal.
- **Reducción de costos**: Elimina intermediarios costosos (abogados, notarios, bancos).

## 2.7 Limitaciones del Modelo Escrow

- **Dependencia de la plataforma**: La seguridad de la transacción depende de la plataforma.
- **Cargos por servicio**: Cada operación tiene un costo (comisión).
- **Tiempo de retención**: Los fondos permanecen retenidos durante el proceso.
- **Arbitraje imperfecto**: El árbitro puede no tener expertise en casos muy especializados.
- **Adopción**: Requiere que ambas partes acepten usar el servicio.
- **Regulatorio**: Algunas jurisdicciones tienen requisitos específicos para servicios de Escrow.

## 2.8 Escenarios de Uso (Ejemplos Prácticos)

### Ejemplo 1: Marketplace de Freelancers
María (diseñadora) y Carlos (cliente) acuerdan un logo por $2,000. Carlos deposita $2,000 en Escrow. María entrega el logo. Carlos tiene 5 días para revisar. Si aprueba (o no objeta), los $2,000 se liberan a María. Si Carlos rechaza, se activa el proceso de disputa y arbitraje.

### Ejemplo 2: Compra de Dominio
Pedro vende el dominio "midominio.com" a Ana por $15,000. Ana deposita $15,000. Pedro transfiere el dominio. Un verificador automático confirma la transferencia WHOIS. Los fondos se liberan a Pedro.

### Ejemplo 3: Desarrollo de Software por Hitos
Una startup contrata a una agencia para desarrollar un MVP por $50,000 en 3 hitos:
- Hito 1 ($15,000): Diseño y prototipo aprobados.
- Hito 2 ($20,000): Backend funcional con pruebas.
- Hito 3 ($15,000): Deploy a producción y documentación.
Cada hito requiere aprobación del comprador para liberación.

### Ejemplo 4: Compra/Venta de Vehículo Usado
Juan vende su auto a Sofía por $25,000. Sofía deposita $25,000. Juan entrega el auto. Sofía lo lleva a un mecánico independiente para inspección (3 días). Si pasa, liberación a Juan. Si no, disputa.

### Ejemplo 5: Comercio Internacional
Empresa en México compra mercancía a proveedor en China por $100,000. Depositan en Escrow. El proveedor envía la mercancía. Un agente de inspección externo verifica la calidad y cantidad. Liberación condicionada al informe de inspección.

### Ejemplo 6: Crowdfunding con Protección
Un proyecto busca $100,000 para desarrollar un juego indie. Los patrocinadores depositan en Escrow. Los fondos se liberan por hitos: prototipo ($30,000), beta ($30,000), lanzamiento ($40,000). Si el proyecto no cumple un hito, los fondos restantes se devuelven.

---

# 3. Actores del Sistema

## 3.1 Actores Humanos

| Actor | Descripción | Privilegios Base |
|---|---|---|
| **Comprador** | Persona que deposita el activo para recibir un bien/servicio | Crear operaciones, depositar, aprobar/objetar, iniciar disputas |
| **Vendedor** | Persona que entrega el bien/servicio para recibir el activo | Aceptar operaciones, ejecutar obligaciones, responder disputas |
| **Árbitro** | Tercero neutral que resuelve disputas | Revisar evidencias, emitir resoluciones vinculantes |
| **Administrador** | Gestiona la plataforma, usuarios, configuraciones | Acceso total a panel administrativo |
| **Agente Escrow** | Operador humano que supervisa operaciones de alto valor o complejas | Monitorear, intervenir, aprobar liberaciones excepcionales |
| **Soporte** | Atención al usuario, resolución de incidencias no técnicas | Ver tickets, comunicarse con usuarios, escalar |
| **Auditor** | Revisa la integridad del sistema (interno/externo) | Acceso de solo lectura a registros y auditoría |
| **Compliance Officer** | Supervisa cumplimiento KYC/KYB/AML | Revisar documentos, aprobar/rechazar verificaciones |
| **Desarrollador** | Integra sistemas externos mediante API | API keys, logs de integración, documentación |
| **Inversor** | Aporta capital a proyectos (crowdfunding/híbrido) | Similar a comprador pero colectivo |

## 3.2 Actores Institucionales

| Actor | Descripción |
|---|---|
| **Empresa** | Entidad legal que actúa como comprador o vendedor (persona moral) |
| **Institución Financiera** | Banco, procesador de pagos, emisor de stablecoins |
| **Proveedor KYC/KYB** | Servicio externo de verificación de identidad |
| **Regulador** | Entidad gubernamental que supervisa actividades financieras |
| **Aseguradora** | Provee seguros para operaciones de alto valor |
| **Notario Digital** | Valida firmas y documentos con fe pública |
| **Agencia de Crédito** | Buró de crédito, historial financiero para scoring |

## 3.3 Actores No Humanos (Sistemas)

| Actor | Descripción |
|---|---|
| **API Externa** | Sistema de tercero que consume la API pública de Escrow |
| **Blockchain** | Red descentralizada para contratos inteligentes o registro inmutable |
| **Gateway de Pagos** | Procesador de pagos fiduciarios (tarjeta, transferencia, SPEI, etc.) |
| **Oráculo** | Fuente externa de datos verificables (tasa de cambio, eventos, status de envío) |
| **Sistema Contable** | ERP o CRM que consume datos de la plataforma |
| **Agente IA** | Módulo automatizado que detecta fraude, clasifica, recomienda |

## 3.4 Modelo de Actores Extensible

El sistema debe permitir la creación de nuevos tipos de actor sin modificar la arquitectura base. Esto se logra mediante:

- **Roles dinámicos** definidos en base de datos, no hardcodeados.
- **Permisos granulares** asociados a combinaciones rol-recurso-acción.
- **Profiles extensibles** mediante schemas JSON que definen atributos específicos de cada tipo de actor.
- **Herencia de roles**: Un actor puede tener múltiples roles simultáneamente.

---

# 4. Casos de Uso

## 4.1 Gestión de Usuarios y Cuentas (12)

| ID | Caso de Uso | Actor Principal | Descripción Breve |
|---|---|---|---|
| UC-01 | Registrarse como usuario individual | Visitante | Crear cuenta con email, teléfono, password y verificación |
| UC-02 | Registrarse como empresa | Visitante | Crear cuenta corporativa con datos fiscales y representante legal |
| UC-03 | Iniciar sesión | Usuario | Autenticación multifactor con credenciales |
| UC-04 | Recuperar contraseña | Usuario | Restablecer acceso mediante email/SMS verificado |
| UC-05 | Completar verificación KYC | Usuario | Subir documentos de identidad, selfie, comprobante de domicilio |
| UC-06 | Completar verificación KYB | Empresa | Subir acta constitutiva, RFC, identificación del representante |
| UC-07 | Gestionar perfil | Usuario | Editar datos personales, preferencias, foto, contacto |
| UC-08 | Configurar 2FA | Usuario | Activar/desactivar autenticación de dos factores |
| UC-09 | Gestionar métodos de pago | Usuario | Agregar/eliminar cuentas bancarias, tarjetas, wallets |
| UC-10 | Ver historial de actividad | Usuario | Consultar operaciones pasadas, estados, documentos |
| UC-11 | Cerrar cuenta | Usuario | Solicitar baja con verificación y devolución de saldos |
| UC-12 | Migrar de individual a empresa | Usuario | Solicitar upgrade a cuenta corporativa |

## 4.2 Gestión de Operaciones Escrow (15)

| ID | Caso de Uso | Actor Principal | Descripción Breve |
|---|---|---|---|
| UC-13 | Crear operación Escrow | Comprador | Definir activo, monto, partes, condiciones iniciales |
| UC-14 | Invitar contraparte | Comprador | Enviar invitación a vendedor para unirse a la operación |
| UC-15 | Aceptar invitación | Vendedor | Aceptar términos y unirse a la operación |
| UC-16 | Negociar condiciones | Comprador/Vendedor | Modificar términos, hitos, fechas, penalizaciones |
| UC-17 | Firmar acuerdo Escrow | Comprador/Vendedor | Firmar digitalmente el contrato de depósito en garantía |
| UC-18 | Depositar activo | Comprador | Transferir el activo a la wallet custodiada |
| UC-19 | Confirmar recepción de activo | Sistema | Verificar validez y cantidad del activo depositado |
| UC-20 | Notificar cumplimiento | Vendedor | Marcar obligación como cumplida para iniciar revisión |
| UC-21 | Aprobar entrega | Comprador | Aceptar el bien/servicio y autorizar liberación |
| UC-22 | Observar período de revisión | Sistema | Contar tiempo de revisión; liberación automática si no hay objeción |
| UC-23 | Solicitar devolución | Comprador | Pedir cancelación y devolución del activo (sujeto a términos) |
| UC-24 | Cancelar operación | Comprador/Vendedor | Cancelar antes o después del depósito (con cargos si aplica) |
| UC-25 | Ver detalle de operación | Comprador/Vendedor | Consultar estado actual, documentos, timeline, historial |
| UC-26 | Compartir operación | Comprador/Vendedor | Invitar a terceros (auditores, asesores, abogados) como observadores |
| UC-27 | Aceptar liberación parcial automática | Sistema | Liberar automáticamente al cumplir hito definido en contrato |

## 4.3 Disputas y Arbitraje (8)

| ID | Caso de Uso | Actor Principal | Descripción Breve |
|---|---|---|---|
| UC-28 | Iniciar disputa | Comprador/Vendedor | Reportar desacuerdo sobre cumplimiento de condiciones |
| UC-29 | Negociar directamente | Comprador/Vendedor | Período de conciliación dentro de la plataforma (chat mediado) |
| UC-30 | Escalar a arbitraje | Comprador/Vendedor | Solicitar intervención de un árbitro |
| UC-31 | Presentar evidencia | Comprador/Vendedor | Subir documentos, capturas, videos que respalden su posición |
| UC-32 | Revisar caso | Árbitro | Analizar evidencias, condiciones contractuales, historial |
| UC-33 | Emitir resolución | Árbitro | Dictaminar liberación total, parcial o devolución |
| UC-34 | Ejecutar resolución | Sistema | Transferir activos según la decisión del árbitro |
| UC-35 | Apelar resolución | Comprador/Vendedor | Solicitar revisión por un panel de árbitros de mayor jerarquía |

## 4.4 Pagos, Wallet y Facturación (7)

| ID | Caso de Uso | Actor Principal | Descripción Breve |
|---|---|---|---|
| UC-36 | Depositar fondos a wallet interna | Comprador | Transferir desde banco/tarjeta a su wallet en la plataforma |
| UC-37 | Retirar fondos de wallet | Usuario | Transferir desde wallet a cuenta bancaria externa |
| UC-38 | Consultar saldo | Usuario | Ver saldo disponible, retenido, en tránsito |
| UC-39 | Ver historial de transacciones | Usuario | Listar depósitos, retiros, liberaciones, comisiones |
| UC-40 | Descargar factura | Usuario | Obtener comprobante fiscal de comisiones pagadas |
| UC-41 | Configurar comisiones | Administrador | Definir tarifas por tipo de operación, activo, monto |
| UC-42 | Conciliar transacciones | Administrador | Verificar movimientos contra extractos bancarios/blockchain |

## 4.5 Administración y Configuración (10)

| ID | Caso de Uso | Actor Principal | Descripción Breve |
|---|---|---|---|
| UC-43 | Gestionar usuarios | Administrador | Listar, buscar, suspender, eliminar cuentas |
| UC-44 | Revisar documentos KYC/KYB | Compliance | Aprobar o rechazar documentos de verificación |
| UC-45 | Configurar parámetros del sistema | Administrador | Límites, tiempos de revisión, porcentajes de comisión |
| UC-46 | Gestionar tipos de activo | Administrador | Agregar/editar tipos de activo soportados |
| UC-47 | Monitorear operaciones activas | Administrador | Dashboard de operaciones en curso, alertas |
| UC-48 | Generar reportes | Administrador | Reportes de volumen, ingresos, disputas, usuarios |
| UC-49 | Gestionar administradores | Súper Admin | Crear/modificar permisos de administradores |
| UC-50 | Ver logs de auditoría | Administrador | Consultar registro de eventos inmutable |
| UC-51 | Gestionar plantillas de contrato | Administrador | Definir/customizar plantillas de acuerdo Escrow |
| UC-52 | Configurar reglas de arbitraje | Administrador | Definir pool de árbitros, costos, reglas de apelación |

## 4.6 Notificaciones y Mensajería (5)

| ID | Caso de Uso | Actor Principal | Descripción Breve |
|---|---|---|---|
| UC-53 | Enviar mensaje en operación | Comprador/Vendedor | Chatear con la contraparte dentro del contexto de la operación |
| UC-54 | Recibir notificación | Usuario | Email/SMS/push sobre cambios de estado en operaciones |
| UC-55 | Configurar preferencias de notificación | Usuario | Elegir canales y tipos de notificaciones deseadas |
| UC-56 | Adjuntar archivo a mensaje | Comprador/Vendedor | Enviar documentos/evidencias en el chat |
| UC-57 | Recibir alertas de seguridad | Usuario | Notificaciones sobre intentos de acceso, cambios de contraseña |

## 4.7 API e Integraciones (5)

| ID | Caso de Uso | Actor Principal | Descripción Breve |
|---|---|---|---|
| UC-58 | Crear operación mediante API | Desarrollador | Integrar creación de Escrow en sistemas externos |
| UC-59 | Consultar estado de operación por API | Desarrollador | Webhook/API polling para updates de estado |
| UC-60 | Verificar identidad por API | Desarrollador | Integrar KYC/KYB en flujo embebido |
| UC-61 | Gestionar webhooks | Desarrollador | Configurar endpoints para recibir eventos en tiempo real |
| UC-62 | Generar link de pago para Escrow | Comprador | Crear link compartible para que el comprador deposite |

## 4.8 Auditoría y Cumplimiento (4)

| ID | Caso de Uso | Actor Principal | Descripción Breve |
|---|---|---|---|
| UC-63 | Consultar pista de auditoría | Auditor | Revisar todos los eventos de una operación |
| UC-64 | Exportar reporte de cumplimiento | Compliance | Generar reporte para entidad regulatoria |
| UC-65 | Verificar firmas digitales | Auditor | Validar integridad y autenticidad de documentos firmados |
| UC-66 | Monitorear transacciones sospechosas | Compliance | Alertas automáticas por patrones de lavado de dinero |

---

# 5. Requerimientos Funcionales

## 5.1 Autenticación y Autorización

| RF-ID | Requerimiento | Prioridad |
|---|---|---|
| RF-01 | El sistema debe soportar registro con email + password, Google OAuth, Apple ID, y Microsoft Entra ID. | Alta |
| RF-02 | El sistema debe requerir verificación de email y/o teléfono antes de permitir operaciones. | Alta |
| RF-03 | El sistema debe implementar autenticación multifactor (TOTP, SMS, llave de seguridad FIDO2/WebAuthn). | Alta |
| RF-04 | El sistema debe soportar sesiones concurrentes con control de dispositivos. | Media |
| RF-05 | El sistema debe permitir Single Sign-On (SSO) para cuentas empresariales (SAML 2.0, OIDC). | Alta |
| RF-06 | El sistema debe gestionar permisos mediante RBAC con posibilidad de extender a ABAC. | Alta |
| RF-07 | El sistema debe permitir a administradores crear roles personalizados con permisos granulares. | Alta |
| RF-08 | El sistema debe invalidar sesiones al cambiar contraseña o roles. | Alta |

## 5.2 Perfiles y Verificación de Identidad

| RF-ID | Requerimiento | Prioridad |
|---|---|---|
| RF-09 | El sistema debe soportar perfiles de persona física y persona moral (empresa). | Alta |
| RF-10 | El sistema debe soportar KYC automatizado con OCR de documentos (INE, pasaporte, cédula, licencia). | Alta |
| RF-11 | El sistema debe verificar identidad mediante comparación de selfie vs documento (liveness detection). | Alta |
| RF-12 | El sistema debe soportar KYB: acta constitutiva, identificación del representante, comprobante fiscal. | Alta |
| RF-13 | El sistema debe permitir verificación progresiva (niveles de verificación según límites de operación). | Media |
| RF-14 | El sistema debe consultar listas de sanciones internacionales (OFAC, ONU, UE) y PEP automáticamente. | Alta |
| RF-15 | El sistema debe permitir al usuario consultar su nivel de verificación y los límites asociados. | Media |

## 5.3 Contratos Escrow

| RF-ID | Requerimiento | Prioridad |
|---|---|---|
| RF-16 | El sistema debe generar un contrato Escrow digital con validez legal al inicio de cada operación. | Alta |
| RF-17 | El sistema debe soportar plantillas de contrato configurables por tipo de operación. | Alta |
| RF-18 | El sistema debe permitir condiciones de liberación programables (fecha, evento, aprobación, hitos). | Alta |
| RF-19 | El sistema debe soportar múltiples firmantes en un mismo contrato. | Alta |
| RF-20 | El sistema debe implementar firma electrónica avanzada con valor probatorio (eIDAS, ESIGN). | Alta |
| RF-21 | El sistema debe permitir anexar documentos al contrato (términos, especificaciones, garantías). | Media |
| RF-22 | El sistema debe versionar los contratos cuando se renegocian condiciones. | Alta |
| RF-23 | El sistema debe notificar a todas las partes al firmar, modificar o ejecutar el contrato. | Media |

## 5.4 Depósitos, Retención y Liberación de Activos

| RF-ID | Requerimiento | Prioridad |
|---|---|---|
| RF-24 | El sistema debe soportar depósitos en múltiples monedas fiduciarias (USD, EUR, MXN, BRL, etc.). | Alta |
| RF-25 | El sistema debe soportar depósitos en criptomonedas (BTC, ETH, USDC, USDT, DAI). | Alta |
| RF-26 | El sistema debe soportar depósito de documentos, licencias, y activos digitales no financieros. | Alta |
| RF-27 | El sistema debe confirmar la recepción del activo antes de notificar al vendedor. | Alta |
| RF-28 | El sistema debe retener el activo en una cuenta/wallet segregada por operación. | Alta |
| RF-29 | El sistema debe soportar liberación total única y liberación parcial por hitos. | Alta |
| RF-30 | El sistema debe ejecutar liberaciones automáticamente al cumplirse las condiciones. | Alta |
| RF-31 | El sistema debe notificar a todas las partes inmediatamente después de cada liberación. | Media |
| RF-32 | El sistema debe generar comprobante de liberación con validez legal. | Media |
| RF-33 | El sistema debe soportar la conversión de moneda al momento de liberación si está acordado. | Media |

## 5.5 Disputas y Arbitraje

| RF-ID | Requerimiento | Prioridad |
|---|---|---|
| RF-34 | El sistema debe permitir iniciar una disputa desde cualquier operación en estado de revisión. | Alta |
| RF-35 | El sistema debe ofrecer un período de negociación directa antes de escalar a arbitraje. | Alta |
| RF-36 | El sistema debe asignar automáticamente un árbitro del pool disponible según expertise. | Alta |
| RF-37 | El sistema debe permitir a las partes presentar evidencias (documentos, imágenes, videos). | Alta |
| RF-38 | El sistema debe garantizar que el árbitro vea toda la información relevante (contrato, chat, evidencias). | Alta |
| RF-39 | El sistema debe ejecutar automáticamente la resolución del árbitro. | Alta |
| RF-40 | El sistema debe permitir apelación ante un panel de árbitros de mayor jerarquía. | Media |
| RF-41 | El sistema debe registrar todo el proceso de disputa como evidencia inmutable. | Alta |
| RF-42 | El sistema debe gestionar el pago al árbitro (comisión de la parte perdedora o dividida). | Media |
| RF-43 | El sistema debe permitir arbitraje con múltiples árbitros para casos de alto valor. | Media |

## 5.6 Mensajería y Notificaciones

| RF-ID | Requerimiento | Prioridad |
|---|---|---|
| RF-44 | El sistema debe proporcionar chat en tiempo real dentro del contexto de cada operación. | Alta |
| RF-45 | El sistema debe permitir compartir archivos en el chat. | Alta |
| RF-46 | El sistema debe notificar eventos críticos por email, SMS y push notification. | Alta |
| RF-47 | El sistema debe permitir configurar preferencias de notificación por canal y tipo de evento. | Media |
| RF-48 | El sistema debe enviar recordatorios automáticos (pendiente de firma, revisión próxima a vencer). | Media |
| RF-49 | El sistema debe mantener el historial de mensajes como parte del registro de auditoría. | Alta |

## 5.7 Panel Administrativo

| RF-ID | Requerimiento | Prioridad |
|---|---|---|
| RF-50 | El sistema debe proveer un dashboard con KPIs en tiempo real (operaciones activas, volumen, ingresos). | Alta |
| RF-51 | El sistema debe permitir búsqueda y filtro avanzado de operaciones, usuarios y transacciones. | Alta |
| RF-52 | El sistema debe permitir gestión de usuarios (suspender, verificar, asignar roles). | Alta |
| RF-53 | El sistema debe permitir configuración de parámetros del sistema (límites, tiempos, comisiones). | Alta |
| RF-54 | El sistema debe permitir visualizar logs de auditoría en tiempo real con búsqueda. | Alta |
| RF-55 | El sistema debe generar reportes exportables (CSV, PDF, XLSX). | Alta |
| RF-56 | El sistema debe permitir gestión de tipos de activo (agregar, editar, deshabilitar). | Alta |

## 5.8 Reportes y Auditoría

| RF-ID | Requerimiento | Prioridad |
|---|---|---|
| RF-57 | El sistema debe mantener un registro de auditoría inmutable de todas las operaciones y eventos. | Alta |
| RF-58 | El sistema debe soportar trail de auditoría por operación (quién, qué, cuándo, desde dónde). | Alta |
| RF-59 | El sistema debe generar reportes de cumplimiento regulatorio KYC/KYB/AML. | Alta |
| RF-60 | El sistema debe generar reportes financieros (volumen, comisiones, ingresos, pérdidas). | Alta |
| RF-61 | El sistema debe permitir exportación de datos para auditoría externa. | Media |

## 5.9 API Pública

| RF-ID | Requerimiento | Prioridad |
|---|---|---|
| RF-62 | El sistema debe exponer una API RESTful pública con autenticación por API Key + JWT. | Alta |
| RF-63 | El sistema debe soportar webhooks para notificar eventos a sistemas externos. | Alta |
| RF-64 | El sistema debe proveer SDKs oficiales (JS/TS, Python, Java, Go). | Media |
| RF-65 | El sistema debe incluir documentación interactiva de la API (OpenAPI 3.1 + Swagger UI). | Alta |
| RF-66 | El sistema debe permitir rate limiting por API key. | Alta |
| RF-67 | El sistema debe soportar el modelo Embedded Escrow (branding white-label para integradores). | Media |

## 5.10 Cumplimiento Normativo

| RF-ID | Requerimiento | Prioridad |
|---|---|---|
| RF-68 | El sistema debe aplicar reglas AML automáticas (monitoreo de transacciones, detección de patrones). | Alta |
| RF-69 | El sistema debe consultar listas de sanciones (OFAC SDN, UN, EU, listas locales) en onboarding y transacciones. | Alta |
| RF-70 | El sistema debe detectar y reportar operaciones inusuales (STR — Suspicious Transaction Report). | Alta |
| RF-71 | El sistema debe aplicar límites por nivel de verificación (segmentación de clientes). | Alta |
| RF-72 | El sistema debe retener documentación KYC/KYB por el período legal exigido (5-10 años según jurisdicción). | Alta |
| RF-73 | El sistema debe soportar la generación de reportes regulatorios automáticos. | Media |

---

# 6. Requerimientos No Funcionales

## 6.1 Disponibilidad

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-01 | El sistema debe mantener disponibilidad del 99.95% (excepto ventanas de mantenimiento programadas). | 99.95% uptime mensual |
| RNF-02 | El sistema debe implementar despliegue multi-zona de disponibilidad (mínimo 3 AZ). | Alta disponibilidad |
| RNF-03 | Las operaciones críticas (depósito, liberación) deben ser tolerantes a fallos de zona. | Sin pérdida de datos |
| RNF-04 | El sistema debe tener redundancia activa en todos los componentes críticos. | N+1 por componente |
| RNF-05 | El sistema debe tener ventanas de mantenimiento que no excedan 4 horas al mes. | < 4h/mes |

## 6.2 Escalabilidad

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-06 | El sistema debe escalar horizontalmente (microservicios) para manejar incrementos de carga. | Auto-scaling horizontal |
| RNF-07 | El sistema debe soportar al menos 10,000 operaciones Escrow concurrentes sin degradación. | 10K concurrentes |
| RNF-08 | El sistema debe manejar picos de 100,000 usuarios activos diarios. | 100K DAU |
| RNF-09 | La capa de base de datos debe escalar horizontalmente (sharding + read replicas). | Escalamiento horizontal |
| RNF-10 | El sistema debe soportar crecimiento de datos histórico sin degradación de consultas. | Datos ilimitados |

## 6.3 Rendimiento

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-11 | El tiempo de respuesta de API (P95) debe ser inferior a 500ms para operaciones estándar. | < 500ms |
| RNF-12 | El tiempo de respuesta de API (P99) debe ser inferior a 2s. | < 2s |
| RNF-13 | La carga de páginas web debe ser inferior a 2s (LCP < 2.5s). | Core Web Vitals |
| RNF-14 | La confirmación de depósitos cripto debe ser en tiempo real o ≤ 1 bloque de confirmación. | Tiempo real |
| RNF-15 | El procesamiento de liberaciones debe ser inmediato (≤ 1s desde la activación de condición). | < 1s |
| RNF-16 | Las notificaciones deben entregarse en menos de 5 segundos desde el evento. | < 5s |

## 6.4 Resiliencia

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-17 | El sistema debe recuperarse automáticamente de fallos sin intervención manual. | Self-healing |
| RNF-18 | Las operaciones en curso no deben perderse ante fallos de componentes individuales. | Sin pérdida de datos |
| RNF-19 | El sistema debe implementar circuit breakers para dependencias externas. | Degradación graceful |
| RNF-20 | El sistema debe tener retry policy con backoff exponencial para operaciones fallidas. | Resiliencia |
| RNF-21 | El sistema debe garantizar consistencia eventual o fuerte según criticidad de la operación. | Consistencia configurable |
| RNF-22 | El sistema debe soportar failover automático entre regiones (DRP). | RTO < 15 min, RPO < 1 min |

## 6.5 Observabilidad

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-23 | El sistema debe exponer métricas de negocio y técnicas en tiempo real (Prometheus, Datadog, Grafana). | Monitoreo completo |
| RNF-24 | El sistema debe implementar tracing distribuido (OpenTelemetry) entre microservicios. | Trazabilidad |
| RNF-25 | El sistema debe centralizar logs estructurados con búsqueda y correlación. | Logging centralizado |
| RNF-26 | El sistema debe generar alertas automáticas basadas en umbrales de rendimiento y errores. | Alertas proactivas |
| RNF-27 | El sistema debe tener dashboards de estado por dominio (negocio, operaciones, técnico, seguridad). | Visibilidad |
| RNF-28 | El sistema debe auditar cada transacción financiera con metadatos completos (IP, user-agent, timestamp). | Traza completa |

## 6.6 Mantenibilidad

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-29 | El código debe seguir una arquitectura limpia con separación clara de capas y dominios. | Mantenible |
| RNF-30 | Cada microservicio debe tener su propio repositorio con documentación y tests. | Autónomo |
| RNF-31 | El sistema debe tener cobertura de tests unitarios ≥ 85% y tests de integración ≥ 70%. | Calidad |
| RNF-32 | La documentación de API debe generarse automáticamente desde el código (OpenAPI). | Auto-documentado |
| RNF-33 | El sistema debe soportar feature flags para despliegues graduales. | Deploy seguro |

## 6.7 Portabilidad

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-34 | La plataforma debe ejecutarse en contenedores Docker orquestados por Kubernetes. | Cloud-agnostic |
| RNF-35 | La infraestructura debe definirse como código (Terraform, Pulumi, CDK). | Infraestructura reproducible |
| RNF-36 | El sistema no debe tener dependencias bloqueantes de un proveedor cloud específico. | Multi-cloud ready |
| RNF-37 | El sistema debe soportar despliegue en nube pública, privada o híbrida. | Flexibilidad de despliegue |

## 6.8 Privacidad

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-38 | El sistema debe cumplir con GDPR, LGPD, CCPA y leyes locales de protección de datos. | Cumplimiento |
| RNF-39 | Los datos personales deben cifrarse en reposo (AES-256) y en tránsito (TLS 1.3). | Cifrado completo |
| RNF-40 | El sistema debe permitir la exportación y eliminación de datos personales (right to be forgotten). | Control de datos |
| RNF-41 | El sistema debe minimizar la recolección de datos al mínimo necesario para operar. | Privacidad por diseño |
| RNF-42 | Los datos sensibles deben enmascararse en logs y reportes. | Protección de datos |

## 6.9 Accesibilidad

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-43 | La interfaz web debe cumplir con WCAG 2.2 AA como mínimo. | Accesibilidad |
| RNF-44 | El sistema debe ser navegable completamente por teclado. | Accesibilidad |
| RNF-45 | El sistema debe soportar lectores de pantalla con etiquetas ARIA adecuadas. | Accesibilidad |
| RNF-46 | Los contrastes de color deben cumplir relación de contraste WCAG AA (4.5:1 texto normal, 3:1 texto grande). | Accesibilidad |

## 6.10 Internacionalización

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-47 | El sistema debe soportar internacionalización (i18n) desde el diseño inicial. | Multi-idioma |
| RNF-48 | Los idiomas iniciales deben ser español, inglés y portugués. | 3 idiomas |
| RNF-49 | El sistema debe soportar múltiples monedas con conversión en tiempo real. | Multi-moneda |
| RNF-50 | El sistema debe soportar formatos regionales (fecha, hora, número, moneda, dirección). | Localización |
| RNF-51 | El sistema debe detectar automáticamente el locale del usuario (navegador, IP, preferencia). | UX |

## 6.11 Cumplimiento Normativo

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-52 | El sistema debe cumplir con regulaciones de servicios de pago (PSD2, NAFIN, CNBV, etc.) según jurisdicción. | Regulatorio |
| RNF-53 | El sistema debe cumplir con normas de prevención de lavado de dinero (GAFI/FATF recomendaciones). | AML |
| RNF-54 | El sistema debe mantener registros por el período legal exigido en cada jurisdicción. | Retención legal |
| RNF-55 | El sistema debe implementar controles de acceso según SOX (Sarbanes-Oxley) si aplica. | Controles financieros |
| RNF-56 | El sistema debe permitir auditorías externas con acceso controlado a datos. | Auditabilidad |

## 6.12 Recuperación ante Desastres

| RNF-ID | Requerimiento | Objetivo |
|---|---|---|
| RNF-57 | El sistema debe tener un plan de recuperación ante desastres documentado y probado trimestralmente. | DRP |
| RNF-58 | El sistema debe replicar datos críticos entre regiones en tiempo real. | RPO < 1 minuto |
| RNF-59 | El sistema debe poder restaurar operaciones completas en región secundaria en menos de 15 minutos. | RTO < 15 min |
| RNF-60 | El sistema debe tener backups automatizados con retención configurable (diario, semanal, mensual, anual). | Backup |
| RNF-61 | Los backups deben almacenarse cifrados en región geográficamente separada. | Seguridad de backups |

---

# 7. Flujo de una Operación Escrow

## 7.1 Mapa de Estados

```
[CREADO] → [NEGOCIACIÓN] → [ACORDADO] → [DEPÓSITO_PENDIENTE] → [DEPOSITADO] → [EN_EJECUCIÓN]
    → [ENTREGADO] → [EN_REVISIÓN] → [APROBADO] → [LIBERADO] → [COMPLETADO]
                                         → [RECHAZADO] → [DISPUTA] → [ARBITRAJE] → [RESUELTO] → [COMPLETADO]
    → [CANCELADO] (desde cualquier estado excepto COMPLETADO)
```

## 7.2 Paso a Paso Detallado

### Fase 1: Creación y Acuerdo

**Paso 1.1 — Creación de la Operación**
- El comprador inicia el flujo.
- Selecciona el tipo de activo, monto, moneda, y la contraparte (email/ID de usuario).
- Define condiciones generales: propósito, plazo máximo.
- El sistema genera un ID único de operación y la coloca en estado `CREADO`.

**Paso 1.2 — Invitación y Registro de Contraparte**
- El sistema envía notificación al vendedor con link para unirse.
- Si el vendedor no está registrado, debe crear cuenta y completar KYC básico.
- El vendedor acepta la invitación y accede a los términos iniciales.
- La operación pasa a `NEGOCIACIÓN`.

**Paso 1.3 — Negociación de Condiciones**
- Ambas partes negocian dentro de la plataforma o mediante propuestas estructuradas.
- Elementos negociables:
  - Monto y moneda (o tipo de activo no monetario).
  - Hitos (fechas, entregables, criterios de aceptación).
  - Período de revisión (por defecto 3-14 días).
  - Reglas de arbitraje (número de árbitros, costo, jurisdicción).
  - Penalizaciones por cancelación.
  - Gastos de comisión (quién paga: comprador, vendedor, dividido).
- Cualquier modificación genera una nueva versión del acuerdo.
- Cuando ambas partes aceptan, pasan a firma.
- La operación pasa a `ACORDADO`.

**Paso 1.4 — Firma Digital del Acuerdo Escrow**
- El sistema genera el contrato Escrow con todas las condiciones acordadas.
- Ambas partes firman digitalmente (firma electrónica avanzada).
- El contrato firmado se almacena como evidencia inmutable.
- Se notifica a ambas partes que el acuerdo está vigente.
- La operación pasa a `DEPÓSITO_PENDIENTE`.

### Fase 2: Depósito y Confirmación

**Paso 2.1 — Depósito del Activo**
- El comprador deposita el activo en la cuenta/wallet segregada de la operación.
- Métodos soportados:
  - Transferencia bancaria (nacional e internacional).
  - Tarjeta de crédito/débito.
  - Criptomonedas (BTC, ETH, USDC, USDT).
  - Depósito de documentos digitales (subida cifrada).
  - Depósito de tokens no fungibles (NFT).
  - Notificación de depósito de bien físico (con evidencia fotográfica/video).
- La plataforma asigna una referencia única de depósito vinculada a la operación.

**Paso 2.2 — Confirmación de Recepción**
- Sistema financiero: El gateway de pagos confirma la liquidación (inmediato para tarjeta/cripto; 1-3 días para transferencia).
- Sistema documental: Validación de integridad (hash), formato, tamaño.
- Sistema de bienes físicos: El comprador sube evidencia de depósito en custodia externa.
- Una vez confirmado, la operación pasa a `DEPOSITADO`.
- Se notifica al vendedor que el activo está retenido y puede proceder.

### Fase 3: Ejecución y Entrega

**Paso 3.1 — Ejecución por Parte del Vendedor**
- El vendedor cumple su obligación según lo acordado:
  - Entrega de bien/servicio.
  - Transferencia de dominio/propiedad.
  - Finalización de hito de desarrollo.
  - Prestación de servicio profesional.
- El vendedor marca la obligación como cumplida y puede adjuntar evidencia.
- La operación pasa a `ENTREGADO` y comienza el período de revisión.

**Paso 3.2 — Período de Revisión**
- El comprador tiene el tiempo acordado para revisar el entregable.
- El sistema muestra un contador regresivo visible para ambas partes.
- Durante la revisión, comprador y vendedor pueden comunicarse por el chat interno.
- Opciones del comprador:
  - **Aprobar**: La operación avanza a liberación.
  - **Rechazar**: Se inicia el flujo de disputa.
  - **Solicitar cambios**: El comprador puede solicitar modificaciones (no obligatorio para el vendedor).
- Si el tiempo expira sin acción: aceptación tácita (configurable por tipo de operación).
- La operación pasa a `EN_REVISIÓN`.

### Fase 4: Liberación

**Paso 4.1 — Liberación del Activo**
- Si el comprador aprueba (o expira el período de revisión sin objeción):
  - El sistema ejecuta la liberación automática.
  - El activo se transfiere al vendedor.
  - Se genera comprobante de liberación.
  - Se notifica a ambas partes.
- Si es un Escrow por hitos:
  - Solo se libera el monto correspondiente al hito actual.
  - La operación permanece activa para los siguientes hitos.
- La operación pasa a `LIBERADO`.

**Paso 4.2 — Cierre de Operación**
- Si es una operación de hito único o último hito:
  - Se genera el certificado de cierre.
  - Se archiva toda la documentación.
  - Se envía resumen final a ambas partes por email.
  - La operación pasa a `COMPLETADO`.

### Fase 5: Disputa y Arbitraje

**Paso 5.1 — Inicio de Disputa**
- El comprador rechaza el entregable y selecciona razón:
  - No cumple especificaciones.
  - Incompleto.
  - Defectuoso.
  - No entregado.
  - Fuera de plazo.
- El vendedor es notificado y puede responder.
- La operación pasa a `DISPUTA`.

**Paso 5.2 — Negociación Directa**
- Período de conciliación (por defecto 5 días).
- Chat mediado con posibilidad de adjuntar evidencias.
- Cualquiera de las partes puede proponer una solución:
  - Liberación parcial.
  - Descuento sobre el monto.
  - Nueva entrega con plazo extendido.
  - Devolución total.
- Si ambas partes acuerdan, se ejecuta la solución acordada.
- Si no hay acuerdo, cualquiera puede escalar a arbitraje.

**Paso 5.3 — Arbitraje**
- El sistema asigna un árbitro del pool (automático o por selección de tema de expertise).
- El árbitro recibe:
  - Contrato original y sus versiones.
  - Historial de mensajes.
  - Evidencias de ambas partes.
  - Timeline de la operación.
- Plazo para resolución: 7-14 días (configurable).
- El árbitro puede solicitar información adicional a cualquiera de las partes.
- La decisión del árbitro es vinculante:
  - Liberar al vendedor (total o parcial).
  - Devolver al comprador (total o parcial).
  - Dividir el activo en proporción especificada.
- La operación pasa a `ARBITRAJE`.

**Paso 5.4 — Ejecución de Resolución**
- El sistema ejecuta automáticamente la decisión del árbitro.
- Se generan comprobantes de la resolución.
- Se descuenta la comisión de arbitraje (pagada por la parte perdedora o dividida según reglas).
- La parte perdedora puede apelar dentro de un período (opcional).
- Si se apela, un panel de 3 árbitros revisa el caso.
- La operación pasa a `RESUELTO` y luego a `COMPLETADO`.

## 7.3 Variantes por Escenario

### Escenario: Escrow por Hitos (Desarrollo de Software)
- Múltiples ciclos de `ENTREGADO → EN_REVISIÓN → APROBADO → LIBERACIÓN_PARCIAL`.
- Cada liberación parcial reduce el saldo retenido.
- El comprador puede rechazar un hito sin cancelar toda la operación.
- Si un hito se rechaza, se puede renegociar el plan o escalar a disputa limitada a ese hito.

### Escenario: Escrow de Documentos (Transferencia de Dominio)
- Depósito simultáneo: comprador deposita dinero, vendedor deposita documento/credenciales.
- Liberación simultánea: al cumplirse condiciones, ambos activos se liberan a la vez.
- Verificación automática: el sistema verifica la transferencia WHOIS del dominio.

### Escenario: Escrow Colaborativo (Crowdfunding)
- Múltiples depositantes (compradores colectivos).
- Un beneficiario (proyecto).
- Liberaciones vinculadas a hitos del proyecto validados por un representante de los patrocinadores o un auditor.
- Si el proyecto no cumple, devolución proporcional a cada patrocinador.

### Escenario: Escrow Híbrido (Compra de Inmueble)
- Depósito de diferentes tipos de activo: dinero del comprador + documentos de título del vendedor.
- Participan múltiples actores: comprador, vendedor, notario, banco, agente inmobiliario (como observadores).
- Liberación atómica: todos los activos se liberan simultáneamente.
- Verificación por tercero: un notario digital verifica la transferencia de propiedad.

---

# 8. Arquitectura Funcional — Módulos

## 8.1 Mapa de Módulos

```
┌─────────────────────────────────────────────────────────────────────┐
│                          API GATEWAY                                │
│                   Autenticación · Rate Limiting · Routing           │
├──────────┬──────────┬──────────┬──────────┬──────────┬──────────────┤
│ USUARIOS │IDENTIDAD │CONTRATOS │ ESCROW   │  WALLET  │   PAGOS      │
│ Registro │ KYC/KYB  │Plantillas│ Motor    │ Saldos   │  Pasarela    │
│ Perfiles │ PEP/SANCI│Negociac.│ Condic.  │ Movim.   │  Facturac.   │
│ Roles    │ Biometría│ Firmas   │ Hitos    │ Segregac.│  Conciliac.  │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────────┤
│ ACTIVOS  │DISPUTAS  │ARBITRAJE │MENSAJERÍA│NOTIFICAC │ REPORTES     │
│ Tipos    │ Negoc.   │ Pool     │ Chat     │ Email    │  Dashboard   │
│ Validad. │ Evidenci │ Resoluc. │ Archivos │ SMS/Push │  Exportac.   │
│ Custodia │ Escalac. │ Apelac.  │ Historial│ Plantill │  Cumplimient │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────────┤
│ADMINISTR │ AUDITORÍA│   IA     │ SEGURID  │INTEGRAC. │ CONFIGURAC.  │
│ Usuarios │ Logs     │ Fraude   │ MFA      │ API Publ │  Param.      │
│ Config.  │ Inmutable│ Riesgo   │ OAuth    │ Webhooks │  Comisiones  │
│ Monitoreo│ Traza    │ Document │ Cifrado  │ SDKs     │  Límites     │
└──────────┴──────────┴──────────┴──────────┴──────────┴──────────────┘
```

## 8.2 Descripción de Módulos

### Módulo: Usuarios
- **Responsabilidad**: Gestión del ciclo de vida de cuentas de usuario (personas físicas y morales).
- **Subfuncionalidades**: Registro, autenticación, perfiles, roles, sesiones, preferencias, historial de actividad.
- **Dependencias**: Identidad (para niveles de verificación), Notificaciones.

### Módulo: Identidad
- **Responsabilidad**: Verificación de identidad y cumplimiento KYC/KYB/AML.
- **Subfuncionalidades**: OCR de documentos, liveness detection, verificación biométrica, listas de sanciones, PEP, scoring de identidad, niveles de verificación progresiva.
- **Dependencias**: Usuarios, almacenamiento de documentos, servicios externos KYC.

### Módulo: Contratos
- **Responsabilidad**: Gestión del ciclo de vida de acuerdos Escrow.
- **Subfuncionalidades**: Plantillas, negociación, versionado, firma electrónica, almacenamiento, validación de integridad.
- **Dependencias**: Usuarios, Identidad (para validar firmantes), Escrow (para condiciones).

### Módulo: Escrow (Core)
- **Responsabilidad**: Motor central que orquesta el ciclo de vida de las operaciones de depósito en garantía.
- **Subfuncionalidades**: Máquina de estados, evaluación de condiciones, gestión de hitos, liberaciones automáticas, temporizadores, reglas de cancelación.
- **Dependencias**: Contratos, Wallet, Activos, Disputas, Notificaciones.

### Módulo: Wallet
- **Responsabilidad**: Custodia y gestión de saldos de activos financieros.
- **Subfuncionalidades**: Saldos segregados por operación, movimientos, conciliación, generación de direcciones cripto, conversión de moneda.
- **Dependencias**: Pagos, Activos, Escrow.

### Módulo: Pagos
- **Responsabilidad**: Procesamiento de pagos fiduciarios y facturación.
- **Subfuncionalidades**: Integración con gateways de pago, depósitos, retiros, comisiones, facturación fiscal (CFDI, invoice), conciliación bancaria.
- **Dependencias**: Wallet, Contratos, Configuración.

### Módulo: Activos
- **Responsabilidad**: Gestión de tipos de activo no monetario y su validación.
- **Subfuncionalidades**: Catálogo de tipos de activo, validación de formato, hash de integridad, custodia documental, verificación de titularidad (dominios, NFTs).
- **Dependencias**: Escrow, Almacenamiento, Blockchain (para verificación on-chain).

### Módulo: Disputas
- **Responsabilidad**: Gestión del ciclo de vida de disputas entre partes.
- **Subfuncionalidades**: Inicio de disputa, negociación directa, escalación a arbitraje, gestión de evidencias.
- **Dependencias**: Escrow, Mensajería, Notificaciones, Almacenamiento.

### Módulo: Arbitraje
- **Responsabilidad**: Gestión del proceso de arbitraje formal.
- **Subfuncionalidades**: Pool de árbitros, asignación automática, presentación de evidencias, resolución, apelación, panel de múltiples árbitros, pago a árbitros.
- **Dependencias**: Disputas, Escrow, Pagos.

### Módulo: Mensajería
- **Responsabilidad**: Comunicación entre partes dentro del contexto de una operación.
- **Subfuncionalidades**: Chat en tiempo real, adjuntos, historial, moderación, notificaciones de mensajes.
- **Dependencias**: Escrow, Usuarios, Notificaciones, Almacenamiento.

### Módulo: Notificaciones
- **Responsabilidad**: Entrega de notificaciones multi-canal.
- **Subfuncionalidades**: Email transaccional, SMS, push notifications, webhooks internos, plantillas, preferencias de usuario.
- **Dependencias**: Todos los módulos (como consumidor de eventos).

### Módulo: Reportes
- **Responsabilidad**: Generación de reportes operativos, financieros y de cumplimiento.
- **Subfuncionalidades**: Dashboard en tiempo real, reportes programados, exportación (CSV, PDF, XLSX), reportes regulatorios, análisis de tendencias.
- **Dependencias**: Escrow, Pagos, Usuarios, Identidad.

### Módulo: Administración
- **Responsabilidad**: Gestión centralizada de configuración del sistema.
- **Subfuncionalidades**: Gestión de usuarios, roles, permisos, parámetros del sistema, tipos de activo, plantillas, comisiones, límites.
- **Dependencias**: Todos los módulos.

### Módulo: Auditoría
- **Responsabilidad**: Registro inmutable de todos los eventos del sistema.
- **Subfuncionalidades**: Logs estructurados con firma digital, consulta y búsqueda, exportación, alertas de integridad.
- **Dependencias**: Todos los módulos (como consumidor de eventos).

### Módulo: IA
- **Responsabilidad**: Servicios de inteligencia artificial para automatización y análisis.
- **Subfuncionalidades**: Detección de fraude, scoring de riesgo, clasificación de disputas, OCR y extracción documental, asistente virtual, recomendaciones, predicción de conflictos.
- **Dependencias**: Escrow, Disputas, Identidad, Reportes.

### Módulo: API Gateway
- **Responsabilidad**: Punto de entrada único para clientes externos e internos.
- **Subfuncionalidades**: Enrutamiento, autenticación, rate limiting, transformación de protocolos, documentación OpenAPI, gestión de API keys.
- **Dependencias**: Todos los módulos (como proxy/router).

### Módulo: Integraciones
- **Responsabilidad**: Conexión con sistemas externos.
- **Subfuncionalidades**: Webhooks, SDKs, conectores bancarios, integración blockchain, ERP/CRM connectors.
- **Dependencias**: API Gateway, Pagos, Activos, Notificaciones.

### Módulo: Seguridad
- **Responsabilidad**: Funcionalidades transversales de seguridad.
- **Subfuncionalidades**: MFA, gestión de claves, cifrado, HSM, detección de intrusiones, análisis de comportamiento, rate limiting.
- **Dependencias**: Todos los módulos (capa transversal).

### Módulo: Configuración
- **Responsabilidad**: Parámetros globales del sistema.
- **Subfuncionalidades**: Límites por nivel de verificación, comisiones, tiempos de revisión, reglas de arbitraje, parámetros de cumplimiento.
- **Dependencias**: Administración.

---

# 9. Modelo de Datos Conceptual

## 9.1 Entidades Principales

### Usuario
- Representa a una persona física o moral registrada en la plataforma.
- Atributos clave: ID único, tipo (individual/empresa), email, teléfono, nivel de verificación, estado (activo/suspendido/cerrado).
- Relaciones: tiene perfiles, posee wallets, es parte de operaciones, emite transacciones.

### Empresa
- Extensión del usuario para personas morales.
- Atributos clave: Razón social, RFC/NIF, representante legal, documentos constitutivos, industria.
- Relaciones: vinculada a un usuario (representante), tiene empleados con roles, participa en operaciones B2B.

### Perfil
- Representa los datos personales o corporativos asociados a un usuario.
- Atributos clave: Nombre, dirección, fecha de nacimiento, nacionalidad, documentos de identidad.
- Relaciones: pertenece a un usuario, contiene documentos KYC.

### Documento
- Representa cualquier archivo subido al sistema (identificación, evidencia, contrato).
- Atributos clave: Hash SHA-256, tipo MIME, tamaño, metadata de verificación, estado (pendiente/verificado/rechazado).
- Relaciones: asociado a un usuario, contrato, operación, evidencia de disputa.

### Verificación
- Representa un intento de verificación de identidad o documento.
- Atributos clave: Tipo (KYC, KYB, PEP, Sanctions), estado, resultado, proveedor externo, timestamp, datos de la verificación.
- Relaciones: pertenece a un usuario o empresa.

### Acuerdo (Contrato Escrow)
- Representa el contrato legal entre las partes para una operación de depósito en garantía.
- Atributos clave: Versión, estado (borrador/firmado/enmendado), fecha de firma, hash del documento firmado.
- Relaciones: pertenece a una operación, tiene múltiples firmas, contiene condiciones.

### Firma
- Representa la acción de firma digital de un contrato.
- Atributos clave: Hash de la firma, método (biométrica, token, certificado), IP, user-agent, geolocalización, timestamp.
- Relaciones: asociada a un acuerdo y a un usuario.

### Operación Escrow
- Entidad central del sistema. Representa una transacción de depósito en garantía.
- Atributos clave: Estado (máquina de estados), tipo de activo, monto/moneda, condiciones, hitos, fechas clave, comisión, partes.
- Relaciones: tiene un comprador, un vendedor, un acuerdo asociado, cero o más transacciones financieras, cero o más disputas.

### Hito
- Representa un hito en operaciones multi-hito.
- Atributos clave: Orden, descripción, monto a liberar, fecha estimada, criterios de aceptación, estado (pendiente/completado/rechazado).
- Relaciones: pertenece a una operación Escrow.

### Transacción Financiera
- Representa cualquier movimiento de valor dentro del sistema.
- Atributos clave: Tipo (depósito, liberación, devolución, comisión, retiro), monto, moneda, estado, referencia externa, fees.
- Relaciones: asociada a una operación, wallet, o pago externo.

### Wallet
- Representa una billetera o cuenta de custodia.
- Atributos clave: Saldo disponible, saldo retenido, moneda, tipo (fiduciaria, cripto), dirección (para cripto).
- Relaciones: pertenece a un usuario, contiene transacciones.

### Activo Digital
- Representa un activo no financiero depositado en Escrow.
- Atributos clave: Tipo (documento, licencia, dominio, NFT), hash de contenido, metadata, estado de verificación, URL de referencia.
- Relaciones: asociado a una operación Escrow, vinculado a documentos.

### Disputa
- Representa un desacuerdo entre partes sobre el cumplimiento de condiciones.
- Atributos clave: Estado (abierta/negociación/arbitraje/resuelta), razón, fecha de inicio, parte iniciadora.
- Relaciones: asociada a una operación Escrow, tiene evidencias, tiene resolución.

### Evidencia
- Representa un elemento de prueba presentado durante una disputa.
- Atributos clave: Tipo (documento, imagen, video, peritaje), parte que lo presenta, timestamp.
- Relaciones: pertenece a una disputa, contiene documentos.

### Resolución
- Representa la decisión final de un arbitraje.
- Atributos clave: Decisión (liberar/devolver/dividir), porcentajes, motivación, árbitro responsable, fecha.
- Relaciones: pertenece a una disputa, ejecutada por el sistema en operación Escrow.

### Mensaje
- Representa un mensaje en el chat de operación.
- Atributos clave: Contenido, remitente, timestamp, tipo (texto/archivo/sistema), leído por destinatarios.
- Relaciones: pertenece a una operación Escrow, adjunta documentos.

### Notificación
- Representa un evento de notificación enviado a un usuario.
- Atributos clave: Tipo, canal (email/SMS/push), estado (enviado/entregado/leído), contenido, referencia a entidad relacionada.
- Relaciones: dirigida a un usuario, asociada a una operación.

### Evento de Auditoría
- Representa un evento inmutable registrado por el sistema.
- Atributos clave: Tipo de evento, actor, recurso afectado, datos del evento (JSON), IP, user-agent, hash de integridad, hash del evento anterior (cadena).
- Relaciones: forma una cadena hash-encadenada, referencias a entidades relacionadas.

### Configuración
- Representa parámetros de configuración del sistema.
- Atributos clave: Clave, valor, tipo, ámbito (global/por rol/por tipo de operación).
- Relaciones: jerarquía de configuraciones con herencia.

## 9.2 Relaciones de Alto Nivel

```
Usuario 1──N Perfil
Usuario 1──N Wallet
Usuario 1──N Verificación
Usuario N──N OperacionEscrow (como comprador o vendedor)
Usuario N──M Empresa (pertenece/representa)
Empresa 1──N Verificación

OperacionEscrow 1──1 Acuerdo
OperacionEscrow 1──N Hito
OperacionEscrow 1──N Disputa
OperacionEscrow 1──N Mensaje
OperacionEscrow N──N ActivoDigital
OperacionEscrow 1──N TransaccionFinanciera

Acuerdo 1──N Firma
Acuerdo 1──N Version (historial de versiones)

Disputa 1──N Evidencia
Disputa 0──1 Resolucion

Usuario 1──N Notificacion
Usuario 1──N EventoAuditoria
OperacionEscrow 1──N EventoAuditoria
TransaccionFinanciera 1──N EventoAuditoria
```

---

# 10. Seguridad

## 10.1 Principios de Diseño

- **Defense in depth**: Múltiples capas de seguridad superpuestas.
- **Least privilege**: Cada actor tiene solo los permisos mínimos necesarios.
- **Zero trust**: Ninguna entidad es confiable por defecto, siempre verificar.
- **Secure by default**: Las configuraciones seguras son el valor por defecto.
- **Fail secure**: Ante fallo, el sistema niega acceso en lugar de permitirlo.
- **Privacy by design**: La protección de datos se integra en el diseño, no es un añadido.

## 10.2 Autenticación y Control de Acceso

| Mecanismo | Descripción | Implementación |
|---|---|---|
| **OAuth 2.0 / OIDC** | Autenticación delegada para usuarios y aplicaciones | Proveedor OIDC interno (Keycloak, Auth0, Cognito) |
| **MFA** | Segundo factor obligatorio para operaciones sensibles | TOTP, SMS, FIDO2/WebAuthn, Push |
| **SSO** | Inicio de sesión único empresarial | SAML 2.0, OIDC |
| **RBAC** | Control de acceso basado en roles | Roles predefinidos + personalizados, permisos granulares por recurso |
| **ABAC** | Control de acceso basado en atributos | Políticas dinámicas (ej: "monto > $10,000 requiere aprobación de 2FA adicional") |
| **API Keys** | Autenticación para integraciones | Keys con scopes, rotación forzada cada 90 días |
| **JWT** | Tokens de sesión y API | Access token (15 min) + Refresh token (7 días), firmados con RS256 |

## 10.3 Cifrado

| Capa | Algoritmo | Detalle |
|---|---|---|
| **Tránsito (TLS)** | TLS 1.3 | Certificados gestionados (Let's Encrypt / ACME). HSTS obligatorio. |
| **Reposo (Base de datos)** | AES-256-GCM | Cifrado a nivel de tabla para datos sensibles (documentos, datos bancarios). |
| **Reposo (Archivos)** | AES-256-GCM | Cifrado lado servidor con clave gestionada por KMS. |
| **Claves** | HSM / KMS | Claves maestras almacenadas en HSM (AWS CloudHSM, Azure Dedicated HSM, HashiCorp Vault). |
| **Secrets** | Vault | API keys, credenciales de base de datos, claves privadas en Vault con rotación automática. |
| **Datos efímeros** | No almacenar en claro | CVV, tokens de acceso nunca en logs ni base de datos. |

## 10.4 Gestión de Identidad y Cumplimiento

### KYC (Know Your Customer)
- Verificación documental automatizada con OCR.
- Comparación facial con liveness detection (anti-spoofing).
- Cruce contra bases de datos gubernamentales (según disponibilidad por país).
- Scoring de identidad basado en múltiples fuentes.
- Verificación progresiva: niveles básico, estándar, avanzado, institucional.

### KYB (Know Your Business)
- Validación de registro legal (acta constitutiva, RFC/NIF).
- Verificación de representante legal y poder notarial.
- Validación contra registros públicos (SAT, SHCP, Companies House, etc.).
- Validación de estructura de propiedad (UBO — Ultimate Beneficial Owner).

### AML (Anti-Money Laundering)
- Monitoreo automatizado de transacciones en tiempo real.
- Reglas configurables (montos, frecuencia, patrones, países).
- Scoring de riesgo transaccional.
- Generación automática de STR (Suspicious Transaction Report).
- Integración con sistemas de listas de sanciones.

### Listas de Sanciones
- Consulta de listas OFAC (SDN, Consolidated), UN, UE, listas locales.
- Verificación en onboarding y en cada transacción.
- Fuzzy matching para variaciones nominales (typos, transliteraciones).
- Screening periódico de toda la base de usuarios.

## 10.5 Prevención de Fraude

| Capa | Técnica | Descripción |
|---|---|---|
| **Transaccional** | Anomaly detection | ML para detectar patrones inusuales en depósitos, retiros, liberaciones |
| **Comportamental** | Behavioral analytics | Perfil de usuario: velocidad de operación, horarios, geolocalización, device fingerprint |
| **Rate Limiting** | Por usuario, IP, API key, operación | Límites adaptativos basados en historial y riesgo |
| **Device fingerprint** | Huella digital del dispositivo | Detección de dispositivos previamente asociados a fraude |
| **Geo-blocking** | Restricción geográfica | Bloqueo de países de alto riesgo (configurable) |
| **Velocity checks** | Límites de velocidad | Alertas por múltiples operaciones en corto tiempo |
| **Sybil detection** | Detección de cuentas fraudulentas | Patrones de registro masivo, dispositivos compartidos, IPs repetidas |
| **Session management** | Control de sesiones | Detección de secuestro de sesión, múltiples IPs simultáneas |

## 10.6 Registro Inmutable (Audit Log)

- Cada evento del sistema se registra en una estructura de cadena hash-encadenada (blockchain-like).
- Cada entrada contiene: hash del evento anterior, timestamp, actor, acción, recurso, datos, firma digital.
- Los logs no pueden ser modificados ni eliminados (append-only).
- Los logs se replican en almacenamiento inmutable (S3 Object Lock, Azure Immutable Blob).
- Verificación periódica de integridad de la cadena (consistency check).
- Acceso de solo lectura para auditores y compliance.

## 10.7 Firmas Digitales

- Firma electrónica avanzada con plena validez legal (eIDAS, ESIGN, ley mexicana, etc.).
- Métodos soportados:
  - Firma biométrica (dibujo en dispositivo táctil).
  - Firma con token digital / certificado (e.firma, DNIe).
  - Firma OTP (código enviado a email/teléfono como manifestación de voluntad).
- Cada firma incluye: hash del documento firmado, timestamp calificado, metadata (IP, dispositivo, geolocalización).
- Los documentos firmados se almacenan con su cadena de firmas para verificación posterior.

## 10.8 Seguridad de Infraestructura

| Componente | Medida |
|---|---|
| **Red** | VPC privada, subnets privadas para servicios, subnets públicas solo para load balancers |
| **WAF** | Reglas OWASP Top 10, rate limiting, bloqueo de IPs maliciosas |
| **DDoS** | Protección a nivel de CloudFront/Cloudflare + AWS Shield / Azure DDoS Protection |
| **Bastion Host** | Acceso SSH solo mediante bastión, con MFA y registro de sesiones |
| **Hardening** | CIS Benchmarks para imágenes base (AMI), escaneo de vulnerabilidades semanal |
| **Contenedores** | Escaneo de imágenes (Trivy, Snyk), ejecución sin root, read-only filesystem, seccomp |
| **Kubernetes** | Pod Security Standards (restricted), network policies, OPA/Gatekeeper, service mesh mTLS |
| **CI/CD** | Firma de imágenes, escaneo de dependencias (SCA), análisis SAST/DAST en pipeline |
| **Backup** | Cifrado en reposo y tránsito, almacenamiento en región separada, pruebas de restauración trimestrales |

## 10.9 Gestión de Secrets

- **HashiCorp Vault** (o AWS Secrets Manager / Azure Key Vault) como sistema centralizado.
- Rotación automática de credenciales de base de datos.
- Rotación forzada de API keys cada 90 días.
- Nunca almacenar secrets en código, variables de entorno, o repositorios.
- Inyección de secrets en contenedores mediante sidecar o CSI driver.

## 10.10 Protección contra Ataques Comunes

| Ataque | Mitigación |
|---|---|
| **Inyección SQL** | ORM con parameterized queries, WAF, escaneo SAST |
| **XSS** | CSP headers, sanitización de output, Content Security Policy estricta |
| **CSRF** | Tokens CSRF en formularios, SameSite cookies |
| **SSRF** | Whitelist de destinos, bloqueo de metadatos cloud, network policies |
| **IDOR** | Pruebas de autorización a nivel de API, object-level authorization |
| **Man-in-the-middle** | TLS 1.3, HSTS, certificate pinning (opcional) |
| **Replay attack** | Nonces, timestamps en requests, expiración de JWT |
| **Brute force** | Rate limiting, account locking, MFA, CAPTCHA progresivo |
| **Phishing** | DMARC/DKIM/DANE, verificación de dominios, educación de usuarios |

---

# 11. Arquitectura Técnica

## 11.1 Filosofía General

La arquitectura sigue un modelo de **microservicios orientados al dominio** (Domain-Driven Design) con las siguientes características:

- **Separación por dominio de negocio**: Cada módulo funcional es un microservicio independiente.
- **Comunicación asíncrona preferente**: Eventos (message broker) para operaciones no críticas en tiempo real.
- **Comunicación síncrona controlada**: APIs REST/gRPC para operaciones que requieren respuesta inmediata.
- **CQRS/Event Sourcing**: Separación de lecturas y escrituras para escalabilidad independiente.
- **API Gateway como punto único de entrada**: Seguridad, rate limiting, routing, transformación.
- **Infraestructura como código**: Todo el entorno se define y despliega mediante código versionado.
- **Cloud-agnostic con preferencia Kubernetes**: Portabilidad entre nubes sin vendor lock-in.

## 11.2 Stack Tecnológico Recomendado

### Backend — Lenguaje y Frameworks

| Componente | Opción Principal | Alternativa | Justificación |
|---|---|---|---|
| **Lenguaje** | TypeScript/Node.js (NestJS) | Go (Gin/Fiber), Rust (Actix) | NestJS ofrece estructura modular nativa (módulos, decoradores, DI) que alinea con DDD. TypeScript proporciona tipado fuerte para un dominio financiero. |
| **Framework API** | NestJS | Fastify, Express | NestJS tiene soporte nativo para microservicios, GraphQL, WebSockets, y RabbitMQ/Kafka. |
| **Lenguaje para servicios críticos** | Go | Rust | Go ofrece rendimiento predecible, baja latencia y excelente soporte de concurrencia para servicios de pagos y wallet. |
| **Lenguaje para IA/ML** | Python (FastAPI) | — | Ecosistema ML/DL maduro (PyTorch, TensorFlow, scikit-learn, LangChain). |

### Frontend

| Componente | Opción | Justificación |
|---|---|---|
| **Framework** | Next.js 14+ (React) | SSR, App Router, soporte i18n nativo, renderizado híbrido |
| **UI Library** | Tailwind CSS + Radix UI / shadcn | Componentes accesibles, customizables, bundle pequeño |
| **State Management** | Zustand + React Query (TanStack Query) | Server state con React Query, client state mínimo con Zustand |
| **Formularios** | React Hook Form + Zod | Validación tipada y eficiente |
| **PWA** | Next.js PWA | App mobile-installable sin app store para usuarios no críticos |
| **Mobile** | React Native (Expo) | Si se requiere app nativa, comparte lógica y tipado con el frontend web |

### Base de Datos Relacional

| Opción | Ventajas | Desventajas | Recomendación |
|---|---|---|---|
| **PostgreSQL** | Madurez, extensiones (pg_cron, postgis, pgvector), JSONB, particionamiento, réplicas | — | **Principal**. Base de datos transaccional central. |
| **CockroachDB** | SQL compatible con Postgres, escalamiento horizontal nativo, multi-región | Menos maduro, operación más compleja | Alternativa si se requiere escalamiento multi-región desde el día 1. |

**Uso de PostgreSQL**:
- Tablas transaccionales principales con particionamiento por fecha.
- JSONB para atributos flexibles (condiciones, metadata de operaciones).
- Extensiones: `pgcrypto` (cifrado), `pg_cron` (tareas programadas), `pgvector` (búsqueda semántica para IA).
- Read replicas para consultas de reportes y dashboard.
- Connection pooling con PgBouncer.

### Base de Datos Documental

| Opción | Ventajas | Recomendación |
|---|---|---|
| **MongoDB** | Schema-flexible, escalamiento horizontal nativo | Para documentos de contratos, mensajes de chat, logs de auditoría no financieros. |

### Caché

| Opción | Uso | Justificación |
|---|---|---|
| **Redis** | Caché de sesiones, rate limiting, colas (Bull), publish/subscribe, caché de consultas frecuentes | En memoria, bajo mantenimiento, ecosistema maduro. |

### Cola de Mensajes / Event Bus

| Opción | Uso | Justificación |
|---|---|---|
| **Apache Kafka** | Event sourcing, transacciones financieras, auditoría (garantía de orden y durabilidad) | Throughput masivo, persistencia, replay de eventos, exactly-once semantics. |
| **RabbitMQ** | Mensajería interna entre microservicios, notificaciones, tareas asíncronas | Menor latencia, enrutamiento flexible, más simple de operar. |

**Arquitectura de eventos**:
- Kafka: Eventos de dominio críticos (EscrowDeposited, EscrowReleased, DisputeOpened, ArbitrationResolved). Retención configurable para replay.
- RabbitMQ: Comandos internos (SendEmail, ProcessNotification, GenerateReport). Colas con TTL, DLQ.

### Motor de Búsqueda

| Opción | Uso | Justificación |
|---|---|---|
| **Meilisearch** | Búsqueda full-text para usuarios, operaciones, transacciones | Más simple que Elasticsearch, auto-hosted, baja latencia. |
| **Elasticsearch** | Dashboards analíticos, agregaciones complejas, logs | Si se requiere análisis avanzado (Kibana, Logstash). |

### Almacenamiento de Archivos

| Opción | Uso |
|---|---|
| **S3 / MinIO** | Documentos KYC, contratos firmados, evidencias, avatares |
| **S3 Object Lock / Azure Immutable Blob** | Registro de auditoría inmutable (WORM) |
| **CloudFront / CDN** | Distribución de assets estáticos, documentos firmados con acceso controlado |

### API Gateway

| Opción | Ventajas | Recomendación |
|---|---|---|
| **Kong** | Open source, plugins extensibles, Kubernetes-native | Principal |
| **Traefik** | Auto-discovery en Kubernetes, TLS automático | Alternativa simple |
| **AWS API Gateway** | Serverless, integración nativa AWS | Si se despliega completamente en AWS |

### Contenedores y Orquestación

| Componente | Opción |
|---|---|
| **Contenedores** | Docker con imágenes multi-stage, distroless para producción |
| **Orquestación** | Kubernetes (EKS, AKS, GKE, o self-managed con kOps) |
| **Service Mesh** | Istio o Linkerd (mTLS, observabilidad, traffic splitting) |
| **Helm** | Paquetes Kubernetes versionados |
| **Kustomize** | Overlays por entorno (dev, staging, prod) |

### CI/CD

| Componente | Opción |
|---|---|
| **Pipeline** | GitHub Actions o GitLab CI |
| **Registry** | Docker Registry privado (ECR, GCR, Harbor) |
| **SAST** | SonarQube, CodeQL, Semgrep |
| **SCA** | Snyk, Dependabot, Trivy |
| **DAST** | OWASP ZAP, Burp Suite Enterprise |
| **Secrets Scan** | GitLeaks, TruffleHog |
| **Deploy** | ArgoCD (GitOps) |

### Observabilidad

| Componente | Opción | Uso |
|---|---|---|
| **Métricas** | Prometheus + Grafana | Dashboards técnicos y de negocio |
| **Tracing** | OpenTelemetry + Jaeger/Tempo | Trazabilidad distribuida entre microservicios |
| **Logs** | Loki + Grafana, o ELK | Logs estructurados centralizados |
| **Alertas** | Alertmanager + PagerDuty/Opsgenie | Alertas basadas en umbrales |
| **Uptime monitoring** | Checkly / Better Uptime | Monitoreo sintético de APIs críticas |
| **APM** | Datadog / New Relic / Grafana Faro | Monitoreo de rendimiento de aplicación |
| **Real user monitoring** | Grafana Faro / Sentry | Errores de frontend, rendimiento percibido |

### Infraestructura como Código

| Componente | Opción |
|---|---|
| **Infraestructura** | Terraform (multi-cloud) o Pulumi (si se prefiere TypeScript) |
| **Kubernetes** | Helm + ArgoCD para GitOps |
| **Configuración** | Helm values + Vault (para secrets) |

### Cloud

| Opción | Ventajas | Consideraciones |
|---|---|---|
| **AWS** | Mayor madurez, servicios financieros (Q,...), multi-AZ, EKS, RDS, S3 Glacier | Costo puede escalar rápido |
| **Azure** | Integración con Active Directory, cumplimiento regulatorio (más servicios en LATAM) | Menor madurez en Kubernetes |
| **GCP** | BigQuery, Spanner, liderazgo en datos e IA | Menor presencia en LATAM |
| **Multi-cloud** | Evita vendor lock-in | Mayor complejidad operativa |

**Recomendación**: AWS como cloud principal por madurez y servicios. Terraform multi-cloud para mantener portabilidad. Estrategia de salida documentada.

## 11.3 Patrón de Microservicios

```
                    ┌──────────┐
                    │  Cliente  │
                    │ (Web/Mov) │
                    └─────┬─────┘
                          │ HTTPS
                    ┌─────▼──────┐
                    │ API Gateway│ ← Auth, Rate Limiting, Routing, WAF
                    │  (Kong)    │
                    └──┬───┬──┬──┘
                       │   │  │
           ┌───────────┘   │  └───────────┐
           │               │              │
    ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐
    │   Servicio  │ │   Servicio  │ │   Servicio  │ ...
    │   Usuarios  │ │   Escrow    │ │    Pagos    │
    └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
           │               │              │
           └───────┬───────┴───────┬───────┘
                   │               │
          ┌────────▼──────┐ ┌──────▼────────┐
          │  PostgreSQL   │ │    Kafka      │
          │  (por servicio)│ │ (Event Bus)   │
          └───────────────┘ └───────────────┘
```

## 11.4 Estrategia de Bases de Datos (Database-per-Service)

Cada microservicio tiene su propio esquema de base de datos, garantizando aislamiento:

| Servicio | Base de Datos | Propósito |
|---|---|---|
| Usuarios | PostgreSQL | Perfiles, roles, preferencias |
| Identidad | PostgreSQL + S3 | Documentos KYC, verificaciones, biometría |
| Contratos | PostgreSQL + S3 | Acuerdos, firmas, versiones |
| Escrow (Core) | PostgreSQL | Operaciones, hitos, estados, eventos de dominio |
| Wallet | PostgreSQL | Saldos, movimientos, transacciones (ACID estricto) |
| Pagos | PostgreSQL | Comisiones, facturas, conciliación |
| Disputas | MongoDB | Casos, evidencias, resoluciones (esquema variable) |
| Mensajería | MongoDB | Chats, mensajes, adjuntos |
| Auditoría | PostgreSQL + S3 (inmutable) | Eventos de auditoría encadenados |
| Reportes | Read replica PostgreSQL | Dashboards, agregaciones, exportaciones |

## 11.5 Comparación: Microservicios vs Monolito Modular

| Aspecto | Microservicios | Monolito Modular | Recomendación |
|---|---|---|---|
| **Complejidad inicial** | Alta | Baja | Monolito modular en MVP |
| **Escalabilidad** | Alta (escalado independiente) | Media (escalado vertical + horizontal) | Microservicios en producción |
| **Velocidad de desarrollo early-stage** | Lenta | Rápida | Monolito para early-stage |
| **Aislamiento de fallos** | Alto | Medio | Microservicios |
| **Consistencia de datos** | Eventual (eventual consistency) | Fuerte (transaccional) | Híbrido según criticidad |
| **Costo operativo** | Alto (múltiples servicios) | Bajo | Monolito al inicio |
| **Complejidad de deploys** | Alta | Baja | Monolito early-stage |
| **Madurez del equipo** | Requiere equipo senior | Equipo estándar | Escalar gradualmente |

**Estrategia recomendada**: Comenzar con un **monolito modular** bien estructurado (carpetas por dominio, módulos de NestJS). Transicionar a microservicios cuando:
- El equipo crece a 3+ equipos independientes.
- Un dominio específico requiere escalamiento independiente (ej. wallet con alta carga transaccional).
- La frecuencia de deploys por dominio diverge significativamente.

---

# 12. Integraciones Externas

## 12.1 Proveedores de Pago

| Proveedor | Región | Activos | Características |
|---|---|---|---|
| **Stripe** | Global (40+ países) | Tarjetas, ACH, transferencias, wallets | API excellence, Connect (marketplace), Stripe Treasury |
| **Mercado Pago** | LATAM | Tarjetas, OXXO, SPEI, Pix, cash | Cobertura LATAM insuperable, split de pagos |
| **Adyen** | Global | Tarjetas, wallets locales, métodos de pago locales | Plataforma unificada, expansión global |
| **PayPal/Braintree** | Global | PayPal, tarjetas, Venmo | Alta adopción de consumidores |
| **Square** | USA, CA, JP, AU | Tarjetas, ACH | POS integrado |
| **Plaid** | USA, CA, UK | Conexión bancaria (ACH, verificación de cuentas) | Open Banking |
| **Bancos locales** | Por país | Transferencias SPEI, PIX, SEPA, Fedwire | Integración directa o mediante agregadores |

**Estrategia**: Integrar Stripe como gateway principal (API y madurez). Agregar Mercado Pago para cobertura LATAM. Usar Plaid o equivalente local para Open Banking y verificación de cuentas bancarias.

## 12.2 Blockchain y Cripto

| Integración | Propósito |
|---|---|
| **Blockchain explorers** | Verificación de transacciones on-chain (Etherscan, Blockchair) |
| **Wallets custodiadas** | Fireblocks, BitGo, Coinbase Prime — custodia institucional de activos digitales |
| **Smart contracts** | Contratos inteligentes para liberación automática en blockchain (Ethereum, Solana, Polygon) |
| **Stablecoins** | USDC, USDT, DAI para depósitos y liberaciones en cripto |
| **NFT verification** | Verificación de titularidad y transferencia de NFTs (OpenSea API, Alchemy) |
| **Layer 2** | Optimism, Arbitrum, Polygon para reducir costos de transacción |
| **Oracles** | Chainlink para precios de activos, verificación de eventos externos |

## 12.3 KYC/KYB Providers

| Proveedor | Cobertura | Características |
|---|---|---|
| **Persona** | Global | KYC, KYB, liveness, document verification, list screening |
| **Onfido** | Global | Document verification, facial biometrics, watchlist |
| **Jumio** | Global | KYC, AML, automated identity verification |
| **Mitek** | Global | Document capture, identity verification, liveness |
| **Veriff** | Global / EU | KYC/KYB, compliance, i18n (200+ países, 50+ idiomas) |
| **EncodeID** | LATAM | Enfoque LATAM, CURP, RFC, actas constitutivas |
| **ComplyAdvantage** | Global | AML screening, transaction monitoring, sanctions |
| **WorldChecker** | Global | PEP, sanctions, adverse media screening |

**Estrategia**: Integrar un proveedor principal de KYC (Veriff o Persona por cobertura global + LATAM) y complementar con ComplyAdvantage o similar para AML/screening continuo.

## 12.4 Firma Electrónica

| Proveedor | Características | Validez Legal |
|---|---|---|
| **DocuSign** | Líder global, workflow de firmas, plantillas | eIDAS, ESIGN, UETA |
| **HelloSign (Dropbox)** | API simple, asequible | ESIGN, eIDAS |
| **Lexia** | LATAM (México), e.firma SAT, FIEL | Validez legal México y LATAM |
| **Firmalo** | LATAM, multi-plataforma | Validez legal LATAM |
| **Signaturit** | Europa (eIDAS cualificado) | Sello de tiempo cualificado, firma cualificada |

**Estrategia**: Integrar DocuSign como principal (madurez, cobertura). Agregar Lexia para México/LATAM donde se requiere e.firma.

## 12.5 Comunicaciones

| Servicio | Propósito | Consideraciones |
|---|---|---|
| **SendGrid (Twilio)** | Email transaccional | APIs robustas, entregabilidad, plantillas dinámicas |
| **Resend** | Email alternativo | Simplicidad, buen delivery, developer experience |
| **Twilio** | SMS, WhatsApp, Voice | Cobertura global, canales de comunicación |
| **Firebase Cloud Messaging** | Push notifications | Gratuito, integración con Android/iOS |
| **OneSignal** | Push notifications multi-plataforma | Alternativa a FCM con más funcionalidades |
| **Sendbird / Stream Chat** | Chat in-app | Chat integrado con moderación, archivos, historial |

## 12.6 Almacenamiento

| Servicio | Propósito |
|---|---|
| **AWS S3 / MinIO** | Almacenamiento principal de documentos |
| **CloudFront** | CDN para distribución segura con Signed URLs |
| **AWS Glacier / S3 Glacier** | Archivo de documentos legales (retención a largo plazo) |
| **AWS S3 Object Lock** | Registro de auditoría inmutable (WORM) |

## 12.7 ERP, CRM y Contabilidad

| Integración | Propósito |
|---|---|
| **QuickBooks / Xero** | Facturación y contabilidad automatizada |
| **HubSpot / Salesforce** | CRM para gestión de clientes empresariales |
| **SAP / Oracle** | ERP para integraciones B2B enterprise |
| **Belvo / Finerio** | Open Banking LATAM, agregación financiera |

## 12.8 Estrategia de Integración

- **API-first**: Cada integración externa se encapsula tras una abstracción (Adapter Pattern).
- **Circuit Breaker**: Fallos de proveedores externos no afectan la operación del sistema.
- **Retry con backoff exponencial**: Para operaciones fallidas transitorias.
- **Idempotencia**: Cada integración debe soportar reintentos seguros (idempotency keys).
- **Modo degradado**: Si un proveedor externo falla, el sistema debe continuar funcionando con funcionalidad reducida.
- **Mock de integraciones**: En entornos de desarrollo/testing, todas las integraciones externas deben ser mockeables.

---

# 13. Inteligencia Artificial

## 13.1 Áreas de Aplicación

### 13.1.1 Detección de Fraude (Alta Prioridad)

| Capacidad | Descripción | Tecnología |
|---|---|---|
| **Anomaly detection** | Detectar patrones inusuales en transacciones (montos, frecuencias, geografía) | Isolation Forest, Autoencoders, LSTM |
| **Graph analysis** | Detectar redes de fraude (cuentas conectadas, collusión) | Graph Neural Networks (GNN) |
| **Behavioral biometrics** | Perfil de comportamiento del usuario: velocidad de tecleo, patrones de navegación | ML supervisado + feature engineering |
| **Device fingerprinting** | Detectar dispositivos fraudulentos conocidos | Hash de huella digital + ML |
| **Sybil detection** | Identificar cuentas falsas creadas masivamente | Clustering + anomaly detection |
| **Real-time scoring** | Score de riesgo en milisegundos para cada transacción | Modelo ensemble (XGBoost + NN) |

### 13.1.2 Evaluación de Riesgo

- **Credit scoring**: Evaluación de solvencia de compradores/vendedores para operaciones de alto valor.
- **Transaction risk scoring**: Puntaje de riesgo de cada operación basado en múltiples variables (monto, activo, partes, historial).
- **Counterparty risk**: Evaluación de la contraparte basada en historial de cumplimiento y disputas.
- **Country risk**: Score de riesgo por país de origen/destino de cada parte.

### 13.1.3 Análisis Documental y OCR

- **Extracción automática de datos**: OCR + NLP para extraer información de documentos KYC (INE, pasaporte, actas).
- **Validación documental**: Detección de alteraciones, manipulación de documentos, diferencias de metadatos.
- **Clasificación automática**: Identificar tipo de documento y encaminar al workflow correcto.
- **Extracción de condiciones**: Analizar contratos legales y extraer condiciones clave (montos, fechas, partes).

### 13.1.4 Clasificación y Resolución de Disputas

- **Clasificación automática**: Categorizar disputas por tipo (calidad, plazo, especificaciones, comunicación).
- **Recomendación de resolución**: Basada en casos históricos similares, sugerir una resolución al árbitro.
- **Lenguaje natural**: Analizar el lenguaje de las partes en la disputa para detectar tono, urgencia, veracidad.
- **Predicción de escalación**: Predecir qué disputas escalarán a arbitraje para priorizar intervención temprana.

### 13.1.5 Asistentes Virtuales y Chatbots

- **Asistente de usuario**: Chatbot para resolver dudas sobre el proceso Escrow, estados, documentación requerida.
- **Asistente de arbitraje**: Guía a las partes en el proceso de disputa, qué evidencias presentar, plazos.
- **Asistente de administrador**: Ayuda al equipo de operaciones a resolver incidencias y configurar la plataforma.
- **Onboarding guiado**: Asiste al usuario en el proceso de registro y KYC.

### 13.1.6 Predicción y Recomendaciones

- **Predicción de conflictos**: Identificar operaciones con alto riesgo de disputa antes de que ocurran.
- **Recomendación de hitos**: Sugerir estructura de hitos óptima basada en el tipo de transacción y las partes.
- **Recomendación de árbitro**: Asignar el mejor árbitro según el tipo de disputa y su expertise demostrado.
- **Scoring de usuarios**: Calcular "confiabilidad" de un usuario basada en historial (trust score).

### 13.1.7 Cumplimiento Regulatorio

- **Monitoreo AML**: ML para detectar patrones sospechosos que las reglas fijas no capturan.
- **Clasificación de operaciones sospechosas**: Priorizar alertas AML por probabilidad de riesgo real.
- **Screening inteligente**: Fuzzy matching mejorado con ML para detección de nombres en listas de sanciones.
- **Detección de estructuras**: Identificar operaciones estructuradas (smurfing) para evadir reportes.

## 13.2 Arquitectura de IA

```
┌─────────────────────────────────────────────────────────────────┐
│                       Servicio de IA (Python/FastAPI)            │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌──────────┐ │
│  │ Modelos ML   │ │    NLP      │ │ Computer    │ │ Rag/LLM  │ │
│  │ (fraud/risk) │ │ (disputas)  │ │ Vision(OCR) │ │ Chatbot  │ │
│  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘ └────┬─────┘ │
│         │               │               │             │        │
│  ┌──────▼───────────────▼───────────────▼─────────────▼──────┐ │
│  │                     Feature Store (Redis)                  │ │
│  └───────────────────────────────────────────────────────────┘ │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              Model Registry (MLflow)                       │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

- **Feature Store**: Redis o Feast para features en tiempo real (historial del usuario, transacciones recientes).
- **Model Registry**: MLflow para versionado, experimentación y deploy de modelos.
- **Inferencia en tiempo real**: Endpoints FastAPI con modelos serializados (ONNX, TorchServe).
- **Inferencia batch**: Jobs programados para scoring masivo y detección de patrones.
- **RAG (Retrieval Augmented Generation)**: Para el chatbot, combinar LLM con base de conocimiento de la plataforma.
- **LLM**: OpenAI GPT-4 / Claude para procesamiento de lenguaje natural, análisis de disputas, asistente virtual. Alternativa open-source: Llama 3.1 (70B+) desplegada on-prem para datos sensibles.

## 13.3 Consideraciones de Privacidad para IA

- Los datos de usuarios nunca deben enviarse a modelos externos sin anonimización.
- Usar modelos on-premise o VPC para procesamiento de documentos KYC/KYB.
- Los embeddings y features no deben contener PII (Personally Identifiable Information).
- Consentimiento explícito para usar datos en entrenamiento de modelos.
- Auditoría de decisiones de IA (explicabilidad, fairness).

---

# 14. Riesgos

## 14.1 Riesgos Técnicos

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| **Fallo de base de datos transaccional** | Baja | Crítico | Clustering, backups, DRP, multi-AZ |
| **Pérdida de datos financieros** | Baja | Catastrófico | WAL archiving, replicación síncrona, backups cifrados |
| **Degradación de rendimiento en picos** | Media | Alto | Auto-scaling, caching, read replicas |
| **Fallo de proveedor cloud** | Baja | Alto | Multi-AZ, DRP multi-región, cloud-agnostic design |
| **Dependencia de servicios externos** | Media | Medio | Circuit breakers, modos degradados, proveedores alternativos |
| **Seguridad de contenedores** | Media | Alto | Escaneo de imágenes, runtimes seguros, restricciones K8s |
| **Deuda técnica temprana** | Alta | Medio | Refactoring plan, code reviews, standards desde el inicio |

## 14.2 Riesgos Legales y Regulatorios

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| **Operar sin licencia financiera** | Media | Crítico | Asesoría legal, licencias según jurisdicción, partnership con entidades reguladas |
| **Incumplimiento GDPR/LGPD** | Media | Alto | Privacidad por diseño, DPO, consentimiento, data mapping |
| **Incumplimiento AML** | Media | Crítico | KYC/KYB robusto, monitoreo continuo, reportes regulatorios |
| **Validez legal de contratos digitales** | Baja | Alto | Firmas con validez legal (eIDAS, ESIGN), respaldo legal |
| **Cambios regulatorios** | Alta | Alto | Monitoreo legal continuo, arquitectura flexible, compliance team |
| **Diferentes jurisdicciones** | Alta | Alto | Módulo de cumplimiento configurable por país, asesoría local |

## 14.3 Riesgos Financieros

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| **Fraude con tarjetas de crédito** | Media | Alto | 3D Secure, velocity checks, scoring de riesgo, chargeback management |
| **Lavado de dinero en la plataforma** | Baja | Catastrófico | AML riguroso, límites por nivel, monitoreo continuo |
| **Error de redondeo o conciliación** | Baja | Alto | Contabilidad de precisión (PostgreSQL numeric), conciliación diaria |
| **Quiebra de custodio cripto** | Baja | Alto | Múltiples custodios, cold storage, seguro |
| **Cargos no autorizados** | Baja | Medio | Confirmación de 2FA para cargos, notificaciones inmediatas |
| **Comisiones insuficientes** | Media | Medio | Pricing basado en valor, revisiones periódicas, freemium para adopción |

## 14.4 Riesgos Operativos

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| **Error humano en liberación manual** | Media | Alto | Automatización, workflows de aprobación, segregación de funciones |
| **Tiempo de inactividad no planificado** | Media | Alto | SLA, monitoreo, auto-scaling, DRP |
| **Pérdida de documentos legales** | Baja | Crítico | Almacenamiento redundante, backups, S3 Object Lock |
| **Fuga de información privilegiada** | Baja | Alto | Control de acceso, DLP, monitoreo de usuarios internos |
| **Fraude interno (insider threat)** | Baja | Catastrófico | Segregación de funciones, 2FA, monitoreo de accesos, rotación de roles |
| **Onboarding lento de usuarios** | Alta | Medio | UX optimizada, KYC automatizado, progresivo (empezar con límites bajos) |

## 14.5 Riesgos de Seguridad

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| **Brecha de datos personales** | Baja | Catastrófico | Cifrado en reposo/tránsito, acceso mínimo, monitoreo |
| **Ataque de ingeniería social** | Media | Alto | Educación de usuarios y empleados, MFA, verificación de identidad |
| **Robo de API keys** | Media | Alto | Scopes, rotación, rate limiting, IP whitelist |
| **Exploit de smart contract** | Baja | Alto | Auditoría de contratos, bug bounty, tests exhaustivos |
| **Ataque DDoS** | Media | Alto | WAF, CDN, DDoS protection, auto-scaling |
| **Vulnerabilidad zero-day** | Baja | Alto | Parcheo rápido, WAF virtual patching, monitoreo |

## 14.6 Riesgos de Escalabilidad

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| **Crecimiento más rápido que la infraestructura** | Media | Alto | Auto-scaling, arquitectura escalable desde el diseño |
| **Cuello de botella en base de datos** | Media | Alto | Sharding, read replicas, caching, optimización de consultas |
| **Costos de infraestructura descontrolados** | Alta | Medio | Monitoreo de costos, reserved instances, auto-scaling eficiente |

## 14.7 Riesgos de Experiencia de Usuario

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| **Abandono en onboarding** | Alta | Alto | KYC progresivo, onboarding guiado, UX optimizada |
| **Confusión sobre el proceso Escrow** | Alta | Medio | Educación in-app, tutoriales, asistente IA |
| **Desconfianza en la plataforma** | Media | Alto | Transparencia, reputación, casos de éxito, garantías |
| **Fricción en el registro** | Alta | Medio | OAuth social, registro mínimo, verificación progresiva |

---

# 15. Roadmap

## 15.1 Fase 0 — Fundación (Meses 1-3)

| Objetivo | Establecer la base técnica y regulatoria mínima para operar |
|---|---|
| **Módulos** | Usuarios (registro y autenticación), Identidad (KYC básico), Escrow Core (estándar), Wallet (fiduciaria), Contratos (plantilla única), Pagos (Stripe) |
| **Prioridad** | Crítica |
| **Dependencias** | Stripe account, proveedor KYC (Persona/Veriff), asesoría legal |
| **Complejidad** | Alta (setup regulatorio y financiero) |
| **Riesgos** | Regulatorio (licencias), técnico (setup inicial) |

**Entregables**:
- MVP funcional: una persona puede registrarse, verificar identidad, crear operación Escrow simple, depositar fondos, liberar a contraparte.
- Integración con Stripe para pagos fiduciarios.
- KYC básico con un proveedor.
- Contrato Escrow con firma digital básica.
- Panel administrativo básico.

## 15.2 Fase 1 — Core Expansión (Meses 4-6)

| Objetivo | Completar funcionalidades core con disputas y multi-activo |
|---|---|
| **Módulos** | Disputas, Arbitraje, Mensajería, Notificaciones, Activos (cripto), Escrow por hitos |
| **Prioridad** | Alta |
| **Dependencias** | Fase 0, pool de árbitros, proveedor cripto |
| **Complejidad** | Alta (disputas/arbitraje es de los módulos más complejos) |
| **Riesgos** | Calidad de arbitraje, seguridad cripto, adopción |

**Entregables**:
- Disputas con negociación directa y escalación a arbitraje.
- Pool de al menos 10 árbitros.
- Soporte para criptomonedas (USDC, USDT, ETH).
- Escrow por hitos.
- Chat interno en operaciones.
- Notificaciones multi-canal.

## 15.3 Fase 2 — Cumplimiento y Seguridad (Meses 7-9)

| Objetivo | Robustecer cumplimiento normativo, seguridad y escalabilidad |
|---|---|
| **Módulos** | AML avanzado, listas de sanciones, auditoría inmutable, ABAC, MFA obligatorio |
| **Prioridad** | Alta |
| **Dependencias** | Fase 1, equipo de compliance, auditoría externa |
| **Complejidad** | Alta |
| **Riesgos** | Regulatorio, performance de screening |

**Entregables**:
- AML automatizado con detección de patrones sospechosos.
- Integración con listas de sanciones (OFAC, UN, UE).
- Registro de auditoría inmutable (hash chain).
- ABAC para control de acceso dinámico.
- MFA obligatorio para operaciones sensibles.
- Penetration test y auditoría de seguridad.

## 15.4 Fase 3 — API e Integraciones (Meses 10-12)

| Objetivo | Abrir la plataforma para integradores externos (Embedded Escrow) |
|---|---|
| **Módulos** | API Gateway, Webhooks, SDKs, White-label, Documentación |
| **Prioridad** | Media |
| **Dependencias** | Fase 2 |
| **Complejidad** | Media |
| **Riesgos** | Seguridad de API, abuso, rate limiting, documentación |

**Entregables**:
- API RESTful pública con autenticación por API Key.
- Webhooks para eventos en tiempo real.
- SDKs iniciales (JS/TS, Python).
- Documentación interactiva (OpenAPI 3.1 + Swagger).
- Portal para desarrolladores.
- Modo Embedded Escrow con branding personalizable.

## 15.5 Fase 4 — Inteligencia Artificial (Meses 13-15)

| Objetivo | Incorporar IA para automatización, detección y asistencia |
|---|---|
| **Módulos** | Fraude ML, OCR documental, Asistente virtual, Clasificación de disputas |
| **Prioridad** | Media |
| **Dependencias** | Fase 2 (datos históricos para entrenar modelos), features históricas |
| **Complejidad** | Alta |
| **Riesgos** | Precisión de modelos, privacidad, bias, explicabilidad |

**Entregables**:
- Modelo de detección de fraude en tiempo real.
- OCR para extracción automática de datos de documentos KYC.
- Asistente virtual para usuarios y administradores.
- Clasificación automática de disputas con recomendación de resolución.
- Feature store y pipeline de ML.

## 15.6 Fase 5 — Expansión y Premium (Meses 16-18+)

| Objetivo | Nuevos casos de uso, escalamiento global, funcionalidades avanzadas |
|---|---|
| **Módulos** | Crowdfunding, Inmuebles, Subastas, Contratos recurrentes, IA predictiva |
| **Prioridad** | Baja (depende de adopción) |
| **Dependencias** | Fase 4, partners estratégicos |
| **Complejidad** | Alta (cada nuevo caso de uso requiere adaptación) |
| **Riesgos** | Complejidad, soporte, desenfoque del core |

**Entregables**:
- Escrow colaborativo (crowdfunding).
- Escrow para compra de inmuebles y vehículos.
- Escrow para subastas.
- Contratos recurrentes y por suscripción.
- Modelo predictivo de conflictos.
- Funcionalidades avanzadas (véase sección 17).

---

# 16. Recomendaciones Técnicas

## 16.1 Arquitectura

**Recomendación**: DDD (Domain-Driven Design) con Monolito Modular → Microservicios progresivos.

**Justificación**: Comenzar con un monolito modular basado en dominios (NestJS modules) permite un desarrollo rápido inicial mientras se mantiene la separación de responsabilidades. Cada módulo está claramente delimitado (bounded context) y podría extraerse a un microservicio independiente cuando la escala lo requiera. Esta estrategia evita la sobrecarga operativa de microservicios desde el día 1, que sería una fuente importante de fricción y retraso.

**Cuándo migrar a microservicios**:
- Cuando el equipo de desarrollo supera 3 squads.
- Cuando un dominio específico requiere escalamiento independiente (ej. wallet por alta carga transaccional).
- Cuando la frecuencia de deploys de diferentes dominios diverge y requiere ciclos independientes.

## 16.2 Escalabilidad

**Recomendación**: Escalamiento horizontal + CQRS + Caching agresivo + Sharding de base de datos futuro.

**Justificación**:
- **Horizontal**: Los microservicios stateless permiten escalar añadiendo instancias detrás del load balancer. La capa de estado (base de datos) se escala con read replicas para consultas y sharding futuro para escrituras.
- **CQRS**: Separar operaciones de lectura (consultas, reportes) de escritura (transacciones). Permite escalar cada lado independientemente. Las consultas pueden servirse desde read replicas o caché.
- **Caching**: Redis para sesiones, rate limiting, consultas frecuentes. Invalidación por eventos (cuando un recurso cambia, se invalida su caché).
- **Sharding**: Implementar sharding por `user_id` o `operation_id` cuando la base de datos supere los 2 TB o 10K transacciones/segundo.

## 16.3 Mantenibilidad

**Recomendación**: Clean Architecture + Conventional Commits + CI/CD robusto + Documentación viva.

**Justificación**:
- **Clean Architecture**: Separación en capas (infrastructure, application, domain) con dependencias hacia adentro. Permite cambiar frameworks, bases de datos o proveedores externos sin afectar la lógica de negocio.
- **Conventional Commits**: `feat:`, `fix:`, `chore:`, `docs:` con referencias a issues. Permite generar changelogs automáticos y versionado semántico.
- **CI/CD**: Cada PR ejecuta lint → typecheck → tests unitarios → tests de integración → build → deploy a staging. Los deploys a producción requieren aprobación manual + tests de humo automáticos.
- **Documentación viva**: OpenAPI generado desde código, ADRs (Architecture Decision Records) para decisiones importantes, README por microservicio.

## 16.4 Patrones de Diseño

| Patrón | Uso | Justificación |
|---|---|---|
| **Saga (Coreografía)** | Transacciones distribuidas entre microservicios | Consistencia eventual sin 2PC. Cada servicio publica eventos y reacciona a eventos de otros servicios. |
| **Outbox Pattern** | Publicación confiable de eventos | Escribir evento en misma transacción que la operación de negocio. Un worker lee la outbox table y publica en Kafka. Garantiza que el evento se publica exactamente una vez. |
| **CQRS** | Separación de lecturas y escrituras | Las consultas (reportes, historial, búsqueda) no compiten con las transacciones (depósitos, liberaciones) por los mismos recursos. |
| **Event Sourcing** | Auditoría y trazabilidad | Cada cambio de estado es un evento inmutable. Permite reconstruir el estado en cualquier punto del tiempo. |
| **Strategy Pattern** | Cálculo de comisiones por tipo de operación | Diferentes estrategias de comisión (porcentaje, fija, escalonada) sin modificar el core. |
| **Observer/Event** | Notificaciones y webhooks | Desacopla la generación de eventos de su consumo. Se pueden agregar nuevos consumidores sin modificar productores. |
| **Adapter Pattern** | Integraciones externas | Cada proveedor externo se encapsula tras una interfaz. Cambiar de proveedor solo implica implementar un nuevo adapter. |
| **Circuit Breaker** | Resiliencia con dependencias externas | Fallos de proveedores no cascadan. El sistema opera en modo degradado. |
| **Idempotency Receiver** | Garantía de procesamiento único | Cada request lleva un idempotency key. Si se recibe duplicado, se retorna el resultado previo. |

## 16.5 Estrategia de Despliegue

**Recomendación**: GitOps con ArgoCD + Canary Deployments + Blue/Green para servicios críticos.

| Entorno | Propósito | Actualizaciones |
|---|---|---|
| **Development** | Desarrollo local e integración | Cada push a `develop` |
| **Staging** | Validación pre-producción | Cada merge a `main` |
| **Production** | Producción | Canary (rampa 5% → 25% → 50% → 100%) con rollback automático si errores > umbral |

- **Feature flags**: Liberación de funcionalidades detrás de feature flags (LaunchDarkly, Flagsmith).
- **Rollback inmediato**: Helm rollback con un comando si el canary detecta anomalías.
- **Database migrations**: Migraciones hacia adelante y hacia atrás (no breaking changes en producción). Migraciones ejecutadas como jobs pre-deploy.

## 16.6 Observabilidad

**Recomendación**: OpenTelemetry como estándar + Three Pillars (métricas, logs, tracing).

- **Toda request genera un trace distribuido** que cruza microservicios.
- **Logs estructurados** en formato JSON con `request_id`, `user_id`, `operation_id` como dimensiones comunes.
- **SLI/SLO/SLA** definidos para cada servicio crítico.
- **Dashboards por audiencia**: Técnico (latencia, errores, throughput), Negocio (operaciones, volumen, ingresos), Seguridad (intentos de acceso, anomalías).
- **Alertas basadas en SLO**: Burn rate alerts (si quemas el presupuesto de error muy rápido, alerta inmediata; si lento, alerta en horas/días).
- **Post-mortems sin blame**: Cultura de aprendizaje, no de culpables.

## 16.7 Pruebas

| Tipo | Cobertura Objetivo | Herramientas |
|---|---|---|
| **Unitarias** | ≥ 85% | Jest/Vitest (TS), Go test, pytest |
| **Integración** | ≥ 70% (APIs críticas: 100%) | Supertest, Testcontainers |
| **Contract** | 100% (entre servicios) | Pact |
| **E2E** | Flujos críticos (login → operación → liberación) | Playwright, Cypress |
| **Carga** | 2x pico esperado | k6, Locust |
| **Seguridad** | Trimestral + cada cambio mayor | OWASP ZAP, Burp, SonarQube |
| **Penetration** | Anual externa | Empresa especializada |

- **Estrategia de testing**: Pirámide de testing invertida para servicios financieros — más tests de integración que unitarios, porque el valor está en que los componentes funcionen juntos correctamente.
- **Testcontainers**: Base de datos real (PostgreSQL) en tests de integración, no mocks.
- **Cobertura obligatoria en PRs**: No se puede mergear un PR que reduzca la cobertura general.

## 16.8 Seguridad

**Recomendación**: Shift-left security + Bug bounty + Auditoría externa.

- **SAST en cada commit**: Semgrep/CodeQL en el pipeline de CI.
- **SCA semanal**: Snyk/Dependabot para dependencias.
- **DAST trimestral**: OWASP ZAP contra staging.
- **Bug bounty**: Programa privado con HackerOne o Bugcrowd después de Fase 2.
- **Penetration test anual**: Empresa externa especializada en FinTech.
- **Security champions**: Un ingeniero por equipo entrenado en seguridad.

## 16.9 Cumplimiento

**Recomendación**: Compliance as Code + Auditoría automatizada + Asesoría legal continua.

- **Políticas como código**: OPA/Gatekeeper para Kubernetes, reglas de firewall como código.
- **Controles automatizados**: Pruebas automáticas de que los controles de cumplimiento están activos.
- **Evidencia digital**: Todo control de cumplimiento genera evidencia digital que puede exportarse para auditores.
- **Asesoría legal**: Abogado especializado en FinTech desde el día 1.

## 16.10 Rendimiento

| Aspecto | Estrategia |
|---|---|
| **Base de datos** | Índices compuestos, partial indexes, materialized views para reportes, particionamiento por fecha |
| **API** | Paginación cursor-based, compresión, campos selectivos (GraphQL o sparse fields), response caching |
| **Frontend** | SSR con streaming, lazy loading, code splitting, image optimization, bundle analysis |
| **Caché** | Redis: caché de consultas, rate limiting, sesiones. Invalidación por evento. |
| **CDN** | CloudFront: assets estáticos, documentos firmados con Signed URLs |
| **Asincronía** | Operaciones lentas (reportes, exportaciones, procesamiento de documentos) como jobs asíncronos con notificación al completarse |

## 16.11 Optimización de Costos

| Estrategia | Ahorro Estimado |
|---|---|
| **Reserved Instances / Savings Plans** para workloads predecibles | 30-50% vs on-demand |
| **Spot instances** para workers batch (reportes, ML) | 60-90% vs on-demand |
| **Auto-scaling** agresivo en horas bajas (ej. reducir réplicas de 20 a 3 en la noche) | 40-60% |
| **S3 Glacier** para documentos legales que requieren retención pero no acceso frecuente | 80% vs S3 Standard |
| **Read replicas** para reportes (evitar competencia con base transaccional) | Evita escalamiento vertical |
| **Caché eficiente** (reducir hits a base de datos) | Reduce necesidad de réplicas |
| **Monitoreo de costos** por servicio + alerts de gasto anormal | Evita sorpresas |

---

# 17. Funcionalidades Futuras

## 17.1 Marketplace de Servicios de Confianza

- Directorio de servicios complementarios integrados: inspección de bienes, notarios digitales, seguros, tasadores, traductores oficiales.
- Los usuarios pueden contratar estos servicios directamente desde la plataforma.
- Los pagos a estos proveedores también pasan por Escrow.

## 17.2 Escrow Recurrente (SaaS/Contratos)

- Para servicios gestionados, suscripciones o contratos de mantenimiento.
- Depósito periódico automático con liberación condicionada mensual.
- Ideal para acuerdos de nivel de servicio (SLA) entre empresas.

## 17.3 Tokenización de Activos

- Representación digital de activos físicos o financieros como tokens transferibles dentro de la plataforma.
- Los tokens pueden fraccionarse (ej. comprar 10% de un inmueble mediante Escrow).
- Marketplace secundario de tokens.

## 17.4 Seguro Integrado (Escrow + Insurance)

- Seguro contra fraude o incumplimiento para operaciones de alto valor.
- Prima calculada dinámicamente según scoring de riesgo de la operación.
- Pago de prima integrado en el flujo de creación de Escrow.

## 17.5 Escrow para Propiedad Intelectual

- Registro de timestamp de creación de obra (protección de derechos de autor).
- Depósito del trabajo creativo en Escrow.
- Liberación condicionada al pago de regalías.
- Ideal para músicos, escritores, fotógrafos, diseñadores.

## 17.6 Wallet Multi-firma Corporativa

- Para empresas que requieren aprobación de múltiples personas para liberar fondos.
- Políticas configurables: 2 de 3, 3 de 5, cualquier combinación.
- Integración con tesorería empresarial.

## 17.7 Programa de Referidos y Lealtad

- Trust Score basado en historial de transacciones exitosas.
- Descuentos en comisiones para usuarios con alto Trust Score.
- Programa de referidos con bonificaciones.

## 17.8 Portal de Transparencia Pública

- Dashboard público con métricas agregadas de la plataforma.
- Estadísticas de operaciones, montos retenidos, disputas resueltas.
- Reporte de estado financiero para demostrar solvencia y transparencia.

## 17.9 Escrow para Apuestas y Gaming

- Depósitos en garantía para torneos y competiciones.
- Liberación automática al ganador al finalizar el evento.
- Verificación de resultados mediante oráculos.

## 17.10 DeFi Integration

- Integración con protocolos DeFi para generar rendimiento sobre fondos retenidos.
- Los fondos en Escrow pueden generar intereses (yield) mientras están retenidos.
- Reparto de rendimientos entre las partes según acuerdo (o donación a causa social).
- **Riesgo**: Evaluar cuidadosamente el riesgo de los protocolos DeFi (hacks, impermanent loss, rug pulls).

## 17.11 Cumplimiento Inteligente (RegTech)

- Monitoreo regulatorio automatizado: cambios en leyes y regulaciones.
- Actualización automática de reglas de cumplimiento.
- Reportes regulatorios generados por IA sin intervención manual.
- Adaptación dinámica a nuevas jurisdicciones.

## 17.12 Escrow con Condiciones Off-Chain Verificables

- Integración con APIs externas para verificar condiciones del mundo real.
- Ejemplos: verificación de envío (tracking number), inspección de calidad (informe de laboratorio), cumplimiento normativo (permiso gubernamental).
- Oráculos automatizados que confirman condiciones y gatillan liberaciones.

## 17.13 Asistente de Negociación con IA

- IA que analiza las posiciones de ambas partes y sugiere términos de compromiso.
- Basada en datos históricos de operaciones similares.
- Reduce el tiempo de negociación y aumenta la tasa de acuerdos exitosos.
- Detecta puntos muertos y sugiere alternativas creativas.

## 17.14 Análisis Predictivo de Riesgo de Contraparte

- Trust Score dinámico que evoluciona con cada transacción.
- Historial de cumplimiento, disputas, tiempos de respuesta, comunicación.
- Perfil de riesgo visible para la contraparte antes de aceptar una operación.
- Alertas si una contraparte muestra cambios de comportamiento sospechosos.

## 17.15 Escrow como Servicio (EaaS) White-Label

- Plataforma completa que otras empresas pueden integrar con su propia marca.
- APIs completas, portal personalizable, flujos configurables.
- Modelo de ingresos: SaaS + comisión compartida.

---

# Apéndice A — Glosario

| Término | Definición |
|---|---|
| **ABAC** | Attribute-Based Access Control. Control de acceso basado en atributos. |
| **AML** | Anti-Money Laundering. Prevención de lavado de dinero. |
| **CQRS** | Command Query Responsibility Segregation. Separación de operaciones de lectura y escritura. |
| **DRP** | Disaster Recovery Plan. Plan de recuperación ante desastres. |
| **EaaS** | Escrow as a Service. Escrow como servicio. |
| **eIDAS** | Regulación europea de identificación electrónica y servicios de confianza. |
| **Escrow** | Depósito en garantía. Acuerdo donde un tercero neutral retiene un activo hasta cumplir condiciones. |
| **HSM** | Hardware Security Module. Módulo de seguridad hardware para custodia de claves. |
| **KYC** | Know Your Customer. Proceso de verificación de identidad de clientes. |
| **KYB** | Know Your Business. Proceso de verificación de identidad de empresas. |
| **MFA** | Multi-Factor Authentication. Autenticación multifactor. |
| **OIDC** | OpenID Connect. Capa de autenticación sobre OAuth 2.0. |
| **PEP** | Persona Expuesta Políticamente. |
| **RAG** | Retrieval Augmented Generation. Técnica de IA que combina recuperación de información con generación de texto. |
| **RBAC** | Role-Based Access Control. Control de acceso basado en roles. |
| **RPO** | Recovery Point Objective. Pérdida máxima aceptable de datos. |
| **RTO** | Recovery Time Objective. Tiempo máximo aceptable de recuperación. |
| **SAST** | Static Application Security Testing. Análisis de seguridad estático. |
| **SCA** | Software Composition Analysis. Análisis de composición de software (dependencias). |
| **STR** | Suspicious Transaction Report. Reporte de transacción sospechosa. |
| **UBO** | Ultimate Beneficial Owner. Beneficiario final o controlador real. |
| **WAF** | Web Application Firewall. Cortafuegos de aplicaciones web. |
| **WORM** | Write Once Read Many. Almacenamiento inmutable. |

---

*Este documento es confidencial y propiedad del proyecto. Ninguna parte puede ser reproducida sin autorización.*

