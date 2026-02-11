# Servicio de Calendario Multi‑Plataforma (con cifrado)

## Descripción general del sistema
Servicio de calendario accesible desde **web** y aplicaciones instalables en múltiples dispositivos (móvil, tablet, PC). Los calendarios almacenan información sensible y se guardan **cifrados**. Para operar sobre un calendario, el usuario debe autenticarse y contar con permisos; el sistema descifra bajo demanda y prepara vistas según el cliente.

**Funciones principales**
- Autenticación (usuario/contraseña) y autorización.
- CRUD de calendarios y eventos.
- Diferentes **visualizaciones** según cliente (web/móvil/desktop).
- Alertas configurables por evento y envío de notificaciones a múltiples apps.
- Cifrado de calendarios en repositorio (data-at-rest).

## Arquitectura

### Estilo recomendado
**Arquitectura de servicios (API + notificaciones) con separación clara de responsabilidades**.

Un diseño típico:
- **API Gateway / Backend API** (stateless) para operaciones de calendario.
- **Servicio de Identidad** (authn/authz).
- **Servicio de Calendario** (dominio y reglas).
- **Servicio de Cifrado/Claves** (KMS o módulo dedicado).
- **Servicio de Notificaciones** (programación y entrega).

### Patrones
- **Clean/Hexagonal** en el servicio de calendario.
- **Token-based auth** (p.ej. OAuth2/OIDC) para soportar múltiples clientes.
- **CQRS ligero** (opcional): consultas optimizadas para vistas vs comandos.
- **Job Scheduler** para alertas.
- **Event-driven** para disparar notificaciones tras cambios en eventos.

### Tecnologías (sugerencia)
- Backend: Java/Spring Boot o Node.js/NestJS.
- Almacenamiento: PostgreSQL.
- Cola: RabbitMQ o Redis (para jobs/notificaciones).
- Notificaciones: Push (FCM/APNs) + WebPush/Email según plataforma.
- Cifrado: AES-GCM para data; claves protegidas por KMS/HSM (o equivalente).

## Componentes principales

### Clientes
- **Web App**: vista mensual/semanal/diaria.
- **Apps instalables**: vistas específicas, caché local (según plataforma).

### Backend
- **Identity Service**: login, emisión/rotación de tokens, gestión de sesiones.
- **Calendar Service**:
	- validación de permisos por recurso (calendario compartido, propietario).
	- operaciones de eventos.
	- generación de “view models” por cliente.
- **Encryption/Key Service**:
	- descifrado bajo demanda para operaciones.
	- rotación de claves y políticas.
- **Notification Service**:
	- programación de alertas.
	- envío a canales (push/web/email).

### Persistencia
- DB con calendarios cifrados (payload cifrado) + índices mínimos no sensibles.
- Registro de dispositivos y “push tokens” por usuario.

## Flujo de datos

### Acceso a calendario (lectura)
1. Cliente → Identity: autenticación.
2. Cliente → Calendar API: solicita calendario.
3. Calendar Service valida permisos.
4. Recupera blob cifrado → Encryption Service descifra → Calendar Service arma vista.
5. Respuesta a cliente (solo datos necesarios para la vista).

### Alertas
1. Usuario crea/edita evento con alertas.
2. Calendar Service publica evento interno “AlertScheduled/Updated”.
3. Notification Service programa job.
4. Al disparar la alerta, Notification Service envía notificación a dispositivos registrados.

## Decisiones técnicas
- **Modelo de cifrado**: por usuario o por calendario; el sistema nunca persiste en claro.
- **Gestión de claves**: separación del material de clave (KMS) del storage de calendarios.
- **Autorización**: permisos por calendario (propietario/compartido/lectura/escritura).
- **Sincronización multi‑cliente**: timestamps/ETags para evitar conflictos y soportar caché.
- **Privacidad**: logs sin incluir payloads descifrados; auditoría de accesos.

