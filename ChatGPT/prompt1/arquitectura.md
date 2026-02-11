# Sistema de Gestión de Conferencias (Call for Papers)

## Descripción general del sistema
Sistema web para soportar el proceso completo de organización de una conferencia científica, centrado en la recepción, revisión y selección de papers.

**Actores principales**
- **Autor**: envía papers y corrige versiones finales si aplica.
- **Administrador**: recibe/valida papers, registra envíos, gestiona usuarios/fechas, orquesta la distribución.
- **Revisor**: evalúa papers y entrega informes.
- **Comité/Chair (rol de decisión)**: define aceptación/rechazo y solicita cambios cuando corresponda.

**Capacidades clave**
- Publicación del **call for papers** y fechas relevantes.
- Registro y recepción de papers (metadatos + archivo).
- Asignación de **3 revisores por paper**.
- Recolección de revisiones e informes.
- Selección de papers aceptados y notificación a autores.
- Recepción de **versiones finales** (camera-ready) y construcción del programa final.

## Arquitectura

### Estilo arquitectónico recomendado
**Aplicación web modular (monolito modular) con arquitectura limpia/hexagonal**.

Motivación:
- El dominio está bien delimitado y el sistema es principalmente **workflow + reglas**.
- Se minimiza complejidad operativa vs microservicios.
- Permite evolucionar hacia servicios separados si el volumen o equipos lo justifican.

### Patrones a aplicar
- **DDD (Bounded Contexts)**: Submission, Review, Program, Identity & Access, Notifications.
- **Hexagonal/Clean**: casos de uso (aplicación) aislados de infraestructura.
- **RBAC/ABAC**: control de acceso por roles y permisos.
- **Outbox / Event-driven interno** (opcional): notificaciones y tareas asíncronas (emails, recordatorios).

### Tecnologías (sugerencia)
- **Backend**: Java + Spring Boot (fuerte en seguridad, transacciones, jobs) o Node.js/NestJS.
- **Base de datos**: PostgreSQL (transaccional para estado de workflow).
- **Almacenamiento de archivos**: Object storage (S3-compatible) para PDFs/adjuntos.
- **Mensajería (opcional)**: RabbitMQ/Redis Streams para emails/tareas diferidas.
- **Auth**: sesiones seguras u OAuth2/OIDC (si hay SSO institucional).

## Componentes principales

### Capa de presentación
- **Web UI** (Autor/Revisor/Admin): formularios, listados, dashboards, carga/descarga de archivos.
- **API HTTP**: endpoints para operaciones del dominio.

### Capa de aplicación (Casos de uso)
- `PublishCallForPapers`: publicar/actualizar convocatoria y fechas.
- `SubmitPaper`: registrar metadatos, validar archivo, persistir envío.
- `AssignReviewers`: asignar 3 revisores (considerando conflictos/recarga).
- `SubmitReview`: registrar informe y calificación.
- `MakeDecision`: consolidar evaluaciones y definir aceptación/rechazo/cambios.
- `RequestFinalVersion` y `ReceiveFinalVersion`: gestión de camera-ready.
- `BuildFinalProgram`: generar programa final (lista/agenda).

### Capa de dominio
- Entidades: `Paper`, `Submission`, `ReviewAssignment`, `ReviewReport`, `Decision`, `ConferenceEdition`.
- Reglas:
	- Un paper debe tener exactamente **3 asignaciones**.
	- Fechas límite gobiernan transiciones de estado.
	- Trazabilidad y auditoría de cambios.

### Infraestructura
- Repositorios (DB), almacenamiento de archivos, servicio de email, scheduler de fechas, logging/auditoría.

## Flujo de datos (alto nivel)
1. Admin publica convocatoria y fechas.
2. Autor envía paper: UI → API → validación → DB + storage.
3. Llegada la fecha, Admin ejecuta distribución: se crean asignaciones (3 revisores) en DB.
4. Revisores descargan paper y envían informes: UI → API → DB.
5. En fecha de decisión, Chair/Admin consolida y define resultado: DB → notificaciones.
6. Si requiere cambios, Autor envía versión final: UI → API → storage/DB.
7. Sistema genera programa final a partir de aceptados.

## Decisiones técnicas
- **Modelo de estados** del paper (ej.: `Received → UnderReview → DecisionMade → CameraReadyPending → FinalReceived`).
- **Asignación de revisores**: política determinística (carga máxima, expertise, conflictos) y registro de auditoría.
- **Integridad**: transacciones al crear asignaciones y al registrar decisiones.
- **Seguridad**: permisos por rol; acceso a papers limitado a autores y revisores asignados.
- **Auditoría**: registro de acciones (quién/qué/cuándo) para trazabilidad.

