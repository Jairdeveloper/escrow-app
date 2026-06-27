# Rol

Actúa como un Software Architect, Product Architect, Solution Architect, Security Architect y Technical Lead con amplia experiencia en plataformas financieras (FinTech), sistemas de pagos, contratos digitales, marketplaces, gestión de riesgos, cumplimiento normativo y servicios de Escrow.

Tu misión NO es escribir código.

Tu objetivo es crear una especificación profesional, completa y lista para ser utilizada como documento base para el desarrollo de una aplicación Web, Mobile o ambas.

---

# Contexto

El proyecto consiste en desarrollar una plataforma de **Escrow (Depósito en Garantía)**.

Concepto inicial:

Un servicio de Escrow actúa como un tercero neutral que recibe, protege, administra y libera dinero, activos o cualquier elemento de valor únicamente cuando se cumplen las condiciones previamente acordadas entre las partes involucradas en una transacción.

El sistema debe garantizar confianza, transparencia, trazabilidad y seguridad durante todo el ciclo de vida de una operación.

NO limites el concepto únicamente al dinero.

Permite que el sistema soporte múltiples tipos de activos:

* dinero
* criptomonedas
* bienes digitales
* licencias
* software
* dominios
* NFT
* documentos
* propiedad intelectual
* servicios profesionales
* contratos
* bienes físicos
* cualquier otro activo susceptible de ser retenido bajo condiciones.

Utiliza tu conocimiento para expandir el concepto sin modificar su esencia.

---

# Objetivo

Diseñar una plataforma moderna de Escrow que pueda utilizarse en distintos escenarios.

Ejemplos:

* Marketplace
* Compra/Venta
* Freelancers
* Desarrollo de software
* Compra de vehículos
* Compra de inmuebles
* Comercio internacional
* Servicios profesionales
* Licitaciones
* Contratos B2B
* Inversiones
* Crowdfunding
* Comercio electrónico
* Activos digitales

No asumir un único caso de uso.

La arquitectura debe ser flexible y extensible.

---

# Libertad del modelo

Utiliza tu conocimiento para definir automáticamente:

* requerimientos funcionales
* requerimientos técnicos
* requerimientos de seguridad
* arquitectura
* mejores prácticas
* riesgos
* integraciones
* módulos
* modelos de negocio
* mejoras
* funcionalidades innovadoras

No te limites únicamente a la información proporcionada.

---

# Entregables

## 1. Visión del producto

Describe:

* propósito
* problema que resuelve
* propuesta de valor
* ventajas competitivas
* diferenciadores
* oportunidades de mercado

---

## 2. Concepto de Escrow

Explica profundamente:

* definición
* funcionamiento
* participantes
* responsabilidades
* flujo completo
* ventajas
* limitaciones
* escenarios de uso
* variantes del modelo Escrow

Incluye múltiples ejemplos prácticos.

---

## 3. Actores del sistema

Identifica todos los actores posibles.

Ejemplos:

* comprador
* vendedor
* árbitro
* administrador
* agente de Escrow
* empresa
* institución financiera
* proveedor de pagos
* auditor
* soporte
* API externa

Permite agregar nuevos actores.

---

## 4. Casos de uso

Genera al menos 60 casos de uso.

Agrúpalos por categorías.

---

## 5. Requerimientos funcionales

Define exhaustivamente:

* autenticación
* autorización
* registro
* perfiles
* verificación de identidad (KYC)
* validación empresarial (KYB)
* creación de contratos
* negociación
* aprobación
* depósitos
* retención de fondos
* liberación parcial
* liberación total
* cancelaciones
* devoluciones
* disputas
* arbitraje
* chat
* notificaciones
* historial
* auditoría
* reportes
* panel administrativo
* configuración
* API pública
* integraciones

Añade cualquier funcionalidad que consideres necesaria.

---

## 6. Requerimientos no funcionales

Incluye:

* disponibilidad
* escalabilidad
* rendimiento
* resiliencia
* observabilidad
* mantenibilidad
* portabilidad
* privacidad
* accesibilidad
* internacionalización
* cumplimiento normativo
* recuperación ante desastres

---

## 7. Flujo de una operación Escrow

Describe paso a paso el ciclo completo:

* creación
* negociación
* aceptación
* contrato
* depósito
* confirmación
* entrega
* validación
* liberación
* cierre
* disputa
* resolución
* auditoría

Incluye variantes para distintos escenarios.

---

## 8. Arquitectura funcional

Define módulos como:

* Usuarios
* Identidad
* Contratos
* Escrow
* Wallet
* Pagos
* Facturación
* Activos
* Disputas
* Arbitraje
* Mensajería
* Notificaciones
* Reportes
* Administración
* Configuración
* Auditoría
* IA
* API Gateway
* Integraciones
* Seguridad

Añade cualquier módulo adicional que consideres necesario.

---

## 9. Modelo de datos conceptual

No escribir SQL.

Solo identificar entidades y relaciones de alto nivel.

Ejemplos:

* Usuario
* Empresa
* Contrato
* Escrow
* Transacción
* Wallet
* Activo
* Pago
* Disputa
* Documento
* Firma
* Evidencia
* Historial
* Evento
* Notificación
* Auditoría

---

## 10. Seguridad

Diseña una arquitectura de seguridad completa.

Incluye:

* autenticación multifactor
* OAuth/OIDC
* RBAC/ABAC
* cifrado en tránsito y en reposo
* gestión de secretos
* prevención de fraude
* AML
* KYC/KYB
* listas de sanciones
* monitoreo
* registros inmutables
* firmas digitales
* control de acceso
* rate limiting
* detección de anomalías

Añade cualquier mecanismo relevante.

---

## 11. Arquitectura técnica

Selecciona y justifica un stack tecnológico moderno.

Explica las ventajas de cada decisión.

Considera:

* Frontend
* Backend
* Base de datos relacional
* Base de datos documental
* Caché
* Cola de mensajes
* Motor de búsqueda
* Almacenamiento de archivos
* Balanceadores
* CDN
* API Gateway
* Contenedores
* Kubernetes
* CI/CD
* Observabilidad
* Infraestructura como código
* Cloud
* Servicios administrados
* Bases de datos vectoriales (si aportan valor)
* Integración con IA

No te limites a una única alternativa; compara opciones cuando sea útil.

---

## 12. Integraciones externas

Identifica posibles integraciones con:

* proveedores de pago
* bancos
* Open Banking
* blockchain
* proveedores KYC/KYB
* firma electrónica
* correo electrónico
* SMS
* mensajería
* almacenamiento
* ERP
* CRM
* herramientas contables

---

## 13. Inteligencia Artificial

Describe cómo la IA puede aportar valor.

Ejemplos:

* detección de fraude
* evaluación de riesgo
* análisis documental
* OCR
* extracción de datos
* clasificación de disputas
* asistentes virtuales
* resúmenes automáticos
* predicción de conflictos
* recomendaciones

Añade cualquier otra aplicación relevante.

---

## 14. Riesgos

Analiza:

* técnicos
* legales
* regulatorios
* financieros
* operativos
* seguridad
* privacidad
* escalabilidad
* experiencia de usuario
* fraude
* abuso
* ingeniería social

---

## 15. Roadmap

Divide el proyecto en fases.

Para cada fase indica:

* objetivos
* módulos
* prioridad
* dependencias
* complejidad
* riesgos

---

## 16. Recomendaciones técnicas

Como arquitecto principal del proyecto, genera recomendaciones sobre:

* arquitectura
* escalabilidad
* mantenibilidad
* patrones de diseño
* microservicios vs monolito modular
* estrategia de despliegue
* observabilidad
* pruebas
* automatización
* seguridad
* cumplimiento
* rendimiento
* optimización de costes

Justifica cada recomendación.

---

## 17. Funcionalidades futuras

Propón funcionalidades que no hayan sido solicitadas explícitamente, pero que puedan aportar un alto valor al producto y mejorar su competitividad.

---

# Restricciones

No escribir código.

No generar SQL.

No generar diagramas UML.

No generar APIs REST.

No asumir un framework específico salvo que exista una justificación técnica.

El documento debe ser independiente de la tecnología y servir como base para que posteriormente otro agente genere:

* arquitectura detallada
* backlog
* historias de usuario
* diseño UI/UX
* APIs
* base de datos
* infraestructura
* documentación técnica
* plan de pruebas
* implementación

---

# Nivel de detalle esperado

Produce un documento de calidad empresarial, equivalente a una especificación utilizada por una startup FinTech o una empresa que desarrolla infraestructura financiera crítica.

No simplifiques.

Justifica todas las decisiones importantes.

Cuando existan varias alternativas, compáralas y explica sus ventajas y desventajas.

Completa cualquier aspecto que no haya sido definido utilizando buenas prácticas de arquitectura de software, seguridad, producto y experiencia de usuario.
