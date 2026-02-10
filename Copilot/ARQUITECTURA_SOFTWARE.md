# Arquitectura de Software - Sistema de Gestión de Conferencias

## 1. ARQUITECTURA GENERAL

### 1.1 Estilo Arquitectónico: Arquitectura en Capas (N-Tier) + Microservicios

```
┌─────────────────────────────────────────────────────────────────────┐
│                     CAPA DE PRESENTACIÓN (Frontend)                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   Portal de  │  │   Portal de  │  │   Sistema de │              │
│  │   Autores    │  │   Revisores  │  │ Administración             │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
└─────────────────────────────────────────────────────────────────────┘
                              ▲
                              │ REST/GraphQL API
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                 CAPA DE APLICACIÓN (API Backend)                    │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌───────────┐ │
│  │ Controlador  │ │ Controlador  │ │ Controlador  │ │ Auth Ctrl │ │
│  │ de Papers    │ │ de Revisores │ │ Admin        │ │           │ │
│  └──────────────┘ └──────────────┘ └──────────────┘ └───────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                              ▲
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│               CAPA DE LÓGICA DE NEGOCIO (Domain)                    │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌───────────┐ │
│  │  Servicio    │ │  Servicio    │ │  Servicio    │ │ Servicio  │ │
│  │  de Papers   │ │  de Review   │ │  de Decisión │ │ de Notif. │ │
│  └──────────────┘ └──────────────┘ └──────────────┘ └───────────┘ │
│                                                                     │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌───────────┐ │
│  │  Servicio    │ │  Servicio    │ │  Servicio    │ │ Servicio  │ │
│  │  de Usuarios │ │  de Validación │  de Programa │ │ de Cache  │ │
│  └──────────────┘ └──────────────┘ └──────────────┘ └───────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                              ▲
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                 CAPA DE ACCESO A DATOS (Data)                       │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌───────────┐ │
│  │   Repository │ │   Repository │ │   Repository │ │ External  │ │
│  │   de Papers  │ │ de Revisiones│ │   de Usuarios │ │Services   │
│  └──────────────┘ └──────────────┘ └──────────────┘ └───────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                              ▲
                              │
                ┌─────────────┼─────────────┬─────────────┐
                ▼             ▼             ▼             ▼
        ┌────────────┐  ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ Base Datos │  │  Storage │ │   Redis  │ │   Queue  │
        │  (SQL)     │  │   Files  │ │  (Cache) │ │(Mensajes)│
        └────────────┘  └──────────┘ └──────────┘ └──────────┘
```

## 2. COMPONENTES Y SUS RESPONSABILIDADES

### 2.1 CAPA DE PRESENTACIÓN (Frontend)

#### 2.1.1 Portal de Autores
**Responsabilidades:**
- Autenticación de autores
- Envío y seguimiento de papers
- Visualización de comentarios de revisores
- Descarga de informes
- Reenvío de versiones revisadas

**Características:**
- Interfaz responsive (mobile, tablet, desktop)
- Formularios de envío con validación
- Dashboard de estado
- Notificaciones en tiempo real

#### 2.1.2 Portal de Revisores
**Responsabilidades:**
- Acceso a papers asignados
- Interfaz de evaluación/formulario
- Seguimiento de plazos
- Envío de informes

**Características:**
- Visualizador de PDFs
- Formulario de revisión estructurado
- Gestión de borradores

#### 2.1.3 Panel de Administración
**Responsabilidades:**
- Gestión de usuarios del sistema
- Distribución de papers
- Monitoreo de progreso
- Generación de reportes
- Gestión de plazos

**Características:**
- Dashboard con indicadores KPI
- Gestión de roles y permisos
- Herramientas de análisis
- Exportación de datos

### 2.2 CAPA DE APLICACIÓN (API Backend)

#### 2.2.1 Controlador de Autenticación
```
Endpoints:
POST /auth/register        - Registro de nuevo usuario
POST /auth/login          - Autenticación
POST /auth/refresh        - Renovación de token
POST /auth/logout         - Cierre de sesión
GET  /auth/me             - Perfil actual del usuario
```

#### 2.2.2 Controlador de Papers
```
Endpoints:
POST   /papers            - Crear/enviar nuevo paper
GET    /papers            - Listar papers (filtrado por rol)
GET    /papers/{id}       - Detalles del paper
PUT    /papers/{id}       - Actualizar paper
DELETE /papers/{id}       - Eliminar paper
GET    /papers/{id}/reviews - Obtener reviews de un paper
POST   /papers/{id}/versions - Subir nueva versión
```

#### 2.2.3 Controlador de Revisiones
```
Endpoints:
GET    /reviews           - Listar mis revisiones
POST   /reviews/{id}      - Enviar revisión
GET    /reviews/{id}      - Detalles de revisión
PUT    /reviews/{id}      - Actualizar revisión (borrador)
GET    /papers/{id}/stats - Estadísticas de un paper
```

#### 2.2.4 Controlador de Administración
```
Endpoints:
POST   /admin/papers/distribute      - Distribuir papers a revisores
POST   /admin/papers/select          - Seleccionar papers aceptados
POST   /admin/notifications/send     - Enviar notificaciones
GET    /admin/program                - Generar programa final
POST   /admin/users/import           - Importar revisores
GET    /admin/reports/*              - Diversos reportes
```

### 2.3 CAPA DE LÓGICA DE NEGOCIO (Services)

#### 2.3.1 PaperService
**Métodos principales:**
- `submitPaper(author, paperData)` - Enviar paper
- `validatePaper(paper)` - Validar formato y contenido
- `getPapersByAuthor(authorId)` - Obtener papers de un autor
- `getPapersByReviewer(reviewerId)` - Obtener papers asignados
- `updatePaperStatus(paperId, status)` - Actualizar estado
- `uploadVersion(paperId, versionFile)` - Subir nueva versión
- `getUnreviewedPapers()` - Obtener papers sin revisar

#### 2.3.2 ReviewService
**Métodos principales:**
- `createReview(paperId, reviewerId)` - Crear revisión
- `submitReview(reviewId, reviewData)` - Enviar revisión
- `getReviewStatistics(paperId)` - Estadísticas de reviews
- `getRemainingReviews()` - Reviews pendientes
- `sendReminders()` - Enviar recordatorios
- `compileReviews(paperId)` - Compilar todas las reviews

#### 2.3.3 SelectionService
**Métodos principales:**
- `selectPapers()` - Proceso de selección final
- `generateDecisions()` - Generar decisiones por paper
- `calculateScore(paperId)` - Calcular puntuación
- `identifyRequiredChanges(paperId)` - Identificar cambios necesarios
- `checkAcceptanceCriteria(paperId)` - Validar criterios

#### 2.3.4 NotificationService
**Métodos principales:**
- `notifySubmissionReceived(authorId, paperId)` - Confirmación envío
- `notifyDistribution(reviewerId, papers)` - Notificación distribución
- `notifyReviewDeadline(reviewerId, paperId)` - Recordatorio deadline
- `notifyDecision(authorId, paperId, decision)` - Notificación decisión
- `notifyRequiredChanges(authorId, paperId, changes)` - Cambios requeridos
- `notifyAcceptance(authorId, paperId)` - Aceptación
- `notifyRejection(authorId, paperId)` - Rechazo

#### 2.3.5 ProgramService
**Métodos principales:**
- `buildFinalProgram()` - Construir programa final
- `scheduleSessions()` - Agendar sesiones
- `orderPresentations()` - Ordenar presentaciones
- `generateProgramReport()` - Generar reporte
- `exportProgram(format)` - Exportar (PDF, Excel, etc.)

#### 2.3.6 UserService
**Métodos principales:**
- `createUser(userData)` - Crear usuario
- `importReviewers(file)` - Importar revisores
- `assignRole(userId, role)` - Asignar rol
- `updateUserProfile(userId, data)` - Actualizar perfil
- `deactivateUser(userId)` - Desactivar usuario

#### 2.3.7 ValidationService
**Métodos principales:**
- `validatePaperFormat(file)` - Validar formato PDF
- `validatePaperLength(pages)` - Validar extensión
- `validateMetadata(metadata)` - Validar metadatos
- `validateReviewFormat(review)` - Validar formato review
- `validateEmail(email)` - Validar email

### 2.4 CAPA DE ACCESO A DATOS (Repositories)

#### 2.4.1 Repository Pattern
```
PaperRepository
├── findById(id)
├── findByAuthor(authorId)
├── findByStatus(status)
├── findAll(filters, pagination)
├── save(paper)
├── update(paper)
├── delete(id)
└── findUnreviewedPapers()

ReviewRepository
├── findById(id)
├── findByPaper(paperId)
├── findByReviewer(reviewerId)
├── save(review)
├── findPendingReviews()
└── findOverdueReviews()

UserRepository
├── findById(id)
├── findByEmail(email)
├── findByRole(role)
├── save(user)
├── update(user)
└── findReviewers()

DecisionRepository
├── findByPaper(paperId)
├── save(decision)
├── findAccepted()
├── findRejected()
├── findRequiringChanges()
└── updateStatus(paperId, status)
```

## 3. BASE DE DATOS

### 3.1 Modelo Relacional

```
USUARIOS
├── id (PK)
├── email (UNIQUE)
├── nombre
├── apellido
├── contraseña (hash)
├── rol (ENUM: ADMIN, AUTHOR, REVIEWER)
├── afiliacion
├── especialidad
├── activo
├── fecha_creacion
└── fecha_actualizacion

PAPERS
├── id (PK)
├── titulo
├── resumen
├── palabras_clave
├── autor_id (FK -> USUARIOS)
├── coautores (JSON)
├── archivo_ruta
├── version (INT)
├── estado (ENUM: DRAFT, SUBMITTED, UNDER_REVIEW, EVALUATED, 
│                   ACCEPTED, REJECTED, CHANGES_REQUIRED, FINAL_SUBMITTED)
├── fecha_envio
├── fecha_actualizacion
└── metadata (JSON)

ASIGNACIONES_REVISOR
├── id (PK)
├── paper_id (FK -> PAPERS)
├── revisor_id (FK -> USUARIOS)
├── fecha_asignacion
├── fecha_deadline
└── estado (ENUM: PENDING, IN_PROGRESS, COMPLETED)

REVISIONES
├── id (PK)
├── paper_id (FK -> PAPERS)
├── revisor_id (FK -> USUARIOS)
├── puntuacion (1-10)
├── resumen_ejecutivo
├── fortalezas (TEXT)
├── debilidades (TEXT)
├── comentarios_detallados (TEXT)
├── recomendacion (ENUM: ACCEPT, REJECT, REVIEW)
├── cambios_sugeridos (JSON)
├── fecha_envio
├── estado (ENUM: DRAFT, SUBMITTED)
└── numero_revision (INT)

DECISIONES
├── id (PK)
├── paper_id (FK -> PAPERS, UNIQUE)
├── decision (ENUM: ACCEPTED, REJECTED, CHANGES_REQUIRED)
├── comentarios_comite (TEXT)
├── cambios_requeridos (JSON)
├── fecha_decision
├── puntuacion_promedio (DECIMAL)
├── numero_aceptaciones (INT)
└── numero_rechazos (INT)

NOTIFICACIONES
├── id (PK)
├── usuario_id (FK -> USUARIOS)
├── tipo (ENUM: SUBMISSION, DISTRIBUTION, DEADLINE, DECISION, etc.)
├── titulo
├── mensaje
├── leida (BOOLEAN)
├── fecha_creacion
├── enlace_relacionado
└── metadata (JSON)

PROGRAMA_CONFERENCIA
├── id (PK)
├── titulo
├── fecha_inicio
├── fecha_fin
├── ubicacion
└── sesiones (JSON array de sesiones)

SESIONES
├── id (PK)
├── programa_id (FK -> PROGRAMA_CONFERENCIA)
├── titulo
├── fecha
├── hora_inicio
├── hora_fin
├── sala
├── papers (FK array -> PAPERS)
└── orden_presentacion (INT array)

AUDITORÍA
├── id (PK)
├── usuario_id (FK -> USUARIOS)
├── accion (VARCHAR)
├── entidad_tipo (VARCHAR)
├── entidad_id (INT)
├── cambios_anteriores (JSON)
├── cambios_nuevos (JSON)
├── timestamp
└── ip_address
```

## 4. TECNOLOGÍAS RECOMENDADAS

### 4.1 Stack Tecnológico

#### Backend Principal
| Componente | Recomendación | Alternativa |
|---|---|---|
| **Lenguaje** | Python (Django/FastAPI) | Node.js (Express/Fastify) |
| **Framework** | Django (full-stack) / FastAPI (lightweight) | Spring Boot (Java) |
| **API** | REST (OpenAPI/Swagger) | GraphQL |
| **Base de Datos** | PostgreSQL | MySQL 8+, SQL Server |
| **Cache** | Redis | Memcached |
| **Queue** | Celery + RabbitMQ | Apache Kafka, Bull |
| **Almacenamiento** | MinIO / AWS S3 | Google Cloud Storage |

#### Frontend
| Componente | Recomendación | Alternativa |
|---|---|---|
| **Framework JS** | React / Vue.js | Angular |
| **State Management** | Redux / Vuex | MobX |
| **UI Components** | Material-UI / Bootstrap | Tailwind CSS |
| **Testing** | Jest + React Testing Library | Cypress |

#### DevOps & Deployment
| Componente | Recomendación | Alternativa |
|---|---|---|
| **Containerización** | Docker | Podman |
| **Orquestación** | Kubernetes (k8s) | Docker Swarm |
| **CI/CD** | GitHub Actions / GitLab CI | Jenkins, CircleCI |
| **Monitoreo** | Prometheus + Grafana | DataDog, New Relic |

### 4.2 Justificación de Tecnologías

#### Python + Django/FastAPI
- ✅ Excelente para lógica de negocio compleja
- ✅ ORM robusto (Django ORM o SQLAlchemy)
- ✅ Ecosistema maduro para web
- ✅ Facilita mantenimiento y escalabilidad
- ✅ Buena comunidad académica/científica
- ⚠️ Puede requerir más recursos que Node.js

#### Alternativa: Node.js
- ✅ Mejor rendimiento en I/O
- ✅ JavaScript full-stack (frontend y backend)
- ✅ Comunidad muy activa
- ⚠️ Lógica compleja más difícil de mantener

#### PostgreSQL
- ✅ ACID compliance
- ✅ Manejo excelente de transacciones
- ✅ JSON native (para datos flexibles)
- ✅ Escalable horizontalmente
- ✅ Gratuito y open-source

#### Redis
- ✅ Cache en memoria para notificaciones
- ✅ Pub/Sub para comunicación en tiempo real
- ✅ Session storage
- ✅ Rate limiting

## 5. PATRONES ARQUITECTÓNICOS

### 5.1 Patrón: Layers (N-Tier)
```
Capa de Presentación
        ↓
Capa de Aplicación (Controllers)
        ↓
Capa de Lógica de Negocio (Services)
        ↓
Capa de Acceso a Datos (Repositories)
        ↓
Base de Datos
```
**Ventajas:** Separación clara de responsabilidades, fácil de entender, escalable
**Desventajas:** Puede llevar a código monolítico

### 5.2 Patrón: MVC/MVT
- **Model:** Entidades del dominio (Paper, Review, User)
- **View:** Templates/componentes frontend
- **Controller:** Lógica de enrutamiento y procesamiento

### 5.3 Patrón: Repository
```
Service llama a Repository
Repository abstrae acceso a datos
Permite cambiar base de datos sin afectar lógica
```

### 5.4 Patrón: Service Locator / Dependency Injection
```
Los servicios se inyectan donde se necesitan
Facilita testing con mocks
Mejora mantenibilidad
```

### 5.5 Patrón: Observer/Pub-Sub
```
Cuando ocurre evento (paper aceptado, deadline):
  ├── Servicio genera evento
  ├── Listeners suscritos reciben evento
  ├── Notificaciones se envían
  └── Auditoría se registra
```

### 5.6 Patrón: CQRS (Command Query Responsibility Segregation)
```
Commands (escritura):
  - SubmitPaper
  - CreateReview
  - SelectPapers

Queries (lectura):
  - GetPapersByAuthor
  - GetReviews
  - GetStatistics
```

### 5.7 Patrón: Event-Driven
```
Eventos en el sistema:
  - PaperSubmitted
  - PapersDistributed
  - ReviewSubmitted
  - DecisionMade
  - NotificationSent
```

### 5.8 Patrón: Domain-Driven Design (DDD)
```
Bounded Contexts:
  - Paper Submission Context
  - Review Context
  - Selection Context
  - Notification Context

Entities:
  - Paper (aggregate root)
  - Review (aggregate root)
  - User
  - Program

Value Objects:
  - PaperStatus
  - Review Rating
  - DecisionType
```

## 6. CONSIDERACIONES DE SEGURIDAD

### 6.1 Autenticación & Autorización
```
┌─────────────────────────────────────────┐
│  JWT Token + OAuth2                     │
├─────────────────────────────────────────┤
│ - Token con expiración temporal         │
│ - Refresh tokens para renovación        │
│ - CORS configurado correctamente        │
│ - HTTPS en producción                   │
└─────────────────────────────────────────┘
```

### 6.2 Validación & Sanitización
- Input validation en frontend y backend
- Sanitización de datos para prevenir SQL injection
- Rate limiting en endpoints
- CSRF protection

### 6.3 Almacenamiento de Archivos
- Almacenar en servidor separado o cloud
- Validación de tipo MIME
- Límites de tamaño
- Antivirus en upload
- No permitir ejecución de archivos

### 6.4 Logs y Auditoría
- Todas las acciones registradas
- Tabla de AUDITORÍA
- No loguear datos sensibles
- Logs centralizados

## 7. ESCALABILIDAD Y DESEMPEÑO

### 7.1 Estrategias de Escalado

```
┌──────────────────────────────────────┐
│  Frontend: CDN + Caching              │
│  - Static assets en CDN               │
│  - Cache en navegador                 │
└──────────────────────────────────────┘
           ▲
           │ HTTPS
           │
┌──────────────────────────────────────┐
│  Load Balancer (nginx, HAProxy)       │
│  - Distribuye tráfico                 │
│  - Health checks                      │
└──────────────────────────────────────┘
           ▲
           │
┌──────────────────────────────────────┐
│  Backend API (múltiples instancias)   │
│  - Escalable horizontalmente          │
│  - Stateless                          │
└──────────────────────────────────────┘
           ▲
           │
┌────────┬─┴──────┬──────────┐
▼        ▼        ▼          ▼
PostgreSQL  Redis   RabbitMQ  S3
(Primary)  (Cache)  (Queue)  (Files)
```

### 7.2 Caché
- Redis para sesiones
- Cache de papers frecuentemente consultados
- Cache de listados (reviews pendientes, papers sin revisar)
- Invalidación de caché inteligente

### 7.3 Base de Datos
- Índices en campos frecuentemente consultados
- Particionamiento de tabla de auditoría
- Regular backups
- Read replicas para consultas

### 7.4 Asynchronous Processing
- Envío de emails mediante cola
- Generación de reportes en background
- Procesamiento de documentos en workers
- Celery con RabbitMQ

## 8. DESPLIEGUE

### 8.1 Arquitectura de Despliegue
```
┌─────────────────────────────────────────┐
│         Entorno Local (Dev)             │
│  - Docker Compose                       │
│  - Base de datos local                  │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│         Staging (Pre-producción)        │
│  - Kubernetes                           │
│  - BD espejo de producción              │
│  - Pruebas antes de deployment          │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│         Producción                      │
│  - Kubernetes (HA)                      │
│  - Multiple replicas                    │
│  - Auto-scaling                         │
│  - Monitoreo continuo                   │
└─────────────────────────────────────────┘
```

### 8.2 Containerización
```dockerfile
Backend:
├── Python 3.11+
├── Gunicorn/uWSGI
├── Nginx como reverse proxy
├── Volumen para medias
└── Variables de entorno

Frontend:
├── Node.js
├── Build estático
├── Nginx para servir
└── CDN
```

### 8.3 CI/CD Pipeline
```
Push a repo
    ↓
Ejecutar tests
    ↓
Build Docker image
    ↓
Push a registry
    ↓
Deploy a staging
    ↓
Smoke tests
    ↓
Deploy a producción (manual)
    ↓
Monitoreo y alertas
```
