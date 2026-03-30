Documento de Requerimientos y Arquitectura de Software (SRS)

Proyecto:  Plataforma Fintech de Crédito y Pagos Locales (Modelo BaaS)

Ubicación Objetivo: Cruz del Eje, Córdoba y Noroeste Provincial.
Plazo de Ejecución: 8 Meses (7 de desarrollo + 1 de inserción de mercado).
Versión del Documento: 2.0

1. Resumen Ejecutivo

El proyecto consiste en el desarrollo de una billetera virtual y sistema de gestión de crédito B2B2C orientado al comercio de cercanía. El objetivo es digitalizar el crédito informal ("fiado"), reducir la morosidad mediante un motor de Score Crediticio Local y proveer herramientas de cobro digital (QR) sin asumir el riesgo regulatorio directo de intermediación financiera, utilizando una arquitectura de Banking as a Service (BaaS).

2. Planteo del Problema (Dolores a Resolver)

Falta de Trazabilidad en Créditos Informales: El comercio local pierde liquidez por la gestión manual (libretas).

Riesgo Ciego: El comerciante otorga crédito sin conocer el historial financiero del cliente en la red local.

Gestión de Impagos Inexistente: Los comercios no tienen herramientas automáticas para cobrar deudas vencidas ni para reportar malos pagadores.

Exclusión Financiera: Gran parte de la población no posee tarjetas de crédito bancarias.

Fricción Fiscal y Regulatoria: Riesgo de doble imposición y retenciones (SIRCREB) al manejar dinero en una cuenta única.

3. Arquitectura Financiera y Legal: El Modelo BaaS

Para evitar el riesgo de custodia de fondos, la plataforma se integrará con un proveedor BaaS (ej. Pomelo, BIND).

Creación de CVU: Generación de Clave Virtual Uniforme (CVU) a través de la API del BaaS para cada usuario y comercio.

Custodia Delegada: El dinero reside en cuentas técnicas del proveedor (entidad regulada por el BCRA).

Flujo Transaccional: HakaiPay actúa como un orquestador (capa de software) que envía instrucciones de transferencia (A2A) a la API del BaaS.

4. Requerimientos Funcionales (Core del Sistema)

Módulo 1: Gestión de Identidad, Roles y KYC

Roles del Sistema (RBAC):

SuperAdmin (Equipo Hakai): Acceso a métricas globales y configuración del motor de score.

Comercio_Admin: Dueño del local. Puede setear límites de crédito y ver reportes financieros.

Comercio_Cajero: Solo puede cobrar vía QR y otorgar crédito hasta un límite pre-aprobado.

Usuario_Final: Consumidor.

Onboarding: Integración con API de validación de identidad para escaneo de DNI (frente/dorso) y prueba de vida (Biometría facial).

Módulo 2: Motor de Pagos y Billetera Virtual

Generación de código QR estandarizado (Transferencias 3.0).

Lógica de lectura de QR y procesamiento de pagos con saldo en cuenta.

Historial de movimientos (Cash-in, Cash-out, Pagos, Pagos de Cuotas).

Webhooks para actualización de saldos en tiempo real desde el BaaS.

Módulo 3: Motor de Crédito Inteligente y Otorgamiento

Score Crediticio Local (Algoritmo): Clasificación dinámica (Ej: 0 a 1000 puntos) basada en:

Historial de pagos a término (peso 50%).

Antigüedad en la plataforma (peso 20%).

Nivel de consumo mensual (peso 30%).

Límites de "Fiado": El Comercio_Admin define un límite global de riesgo (ej. $500,000 mensuales en créditos) y límites individuales sugeridos por el Score.

Firma Digital (Pagaré Electrónico): Al aceptar un crédito, el usuario acepta un Contrato de Adhesión mediante token/PIN, generando un comprobante legal exportable en PDF con validez probatoria.

Módulo 4: Ciclo de Vida de la Deuda y Gestión de Impagos

Este módulo gestiona automáticamente el riesgo cuando un usuario no paga a término:

Mora Temprana (1 a 30 días):

Cálculo y aplicación automática de intereses punitorios diarios.

Notificaciones Push y WhatsApp automatizadas (Recordatorios amigables).

Mora Media (31 a 60 días):

Congelamiento de Cuenta: Bloqueo preventivo de la cuenta del usuario para solicitar nuevos créditos en toda la red de comercios asociados.

Débito Automático (Garnishment): Si el usuario ingresa dinero a su billetera, el sistema intercepta los fondos y cancela total o parcialmente la deuda pendiente con el comercio automáticamente.

Mora Grave / Incobrabilidad (+60 días):

Caída drástica en el Score Crediticio del usuario (impacta su reputación en la ciudad).

Generación de un "Certificado de Deuda" en PDF con los logs transaccionales, listo para que el comercio inicie acciones legales (Veraz local).

Módulo 5: Dashboard de Analítica y Reportes

Panel del Comercio:

Flujo de caja diario, semanal y mensual.

Índice de morosidad local (Ej: "Tienes $50,000 en deuda a cobrar").

Retenciones fiscales aplicadas por el BaaS.

Módulo de Disputas: Sistema de tickets interno por si un usuario declara haber pagado en efectivo y el comercio no lo registró en el sistema.

5. Requerimientos No Funcionales

Transaccionalidad (ACID): Base de datos relacional. Garantizar la atomicidad en pagos divididos y débitos automáticos.

Seguridad y Privacidad:

Encriptación AES-256 para PII (Personally Identifiable Information).

Cumplimiento de la Ley 25.326 de Protección de Datos Personales de Argentina.

Autenticación mediante JWT y refresh tokens.

Auditoría (Audit Logs): Tabla inmutable en la base de datos que registre quién hizo qué y cuándo (ej: "El Cajero Juan modificó el límite de crédito del usuario Pedro").

Disponibilidad y Concurrencia: Arquitectura de microservicios orientada a eventos (Kafka o RabbitMQ) para manejar picos transaccionales sin bloquear el hilo principal.

Offline First (Parcial): La app móvil del cajero debe poder registrar créditos temporalmente sin internet y sincronizarlos al recuperar la conexión.

6. Stack Tecnológico

Infraestructura: A determinar

Backend: Node.js (NestJS para arquitectura modular) o Go.

Base de Datos Principal: PostgreSQL.

Base de Datos Caché/Colas: Redis.

Frontend Mobile (Usuarios): React Native (Expo).

Frontend Web (Comercios/Admin): Next.js con Tailwind CSS.

Integraciones: API Pomelo/BIND (BaaS), API Meta (WhatsApp Notifications).

7. Plan de Ejecución y Roadmap (8 Meses)

Mes 1: Arquitectura y Diseño. Modelado ERD (Entity-Relationship Diagram), diseño UX/UI, y firma de entorno Sandbox con BaaS.

Mes 2-3: Core y Billetera. Desarrollo de microservicios de usuarios, conexión BaaS, generación de CVUs y transacciones QR.

Mes 4: Motor de Crédito. Desarrollo del algoritmo de Score, firma de pagarés y lógica de cuotificación.

Mes 5: Gestión de Impagos. Implementación de jobs recurrentes (CRON) para cálculo de intereses, débitos automáticos cruzados y reportes de mora.

Mes 6: Frontend y Dashboards. Conexión de las apps móviles y paneles web al backend.

Mes 7: QA y Auditoría. Pruebas de estrés transaccional, testeo de vulnerabilidades (Pentesting básico) y validación de ACID.

Mes 8: Go-to-Market. Despliegue en producción (Closed Beta) con comercios ancla en Cruz del Eje.
