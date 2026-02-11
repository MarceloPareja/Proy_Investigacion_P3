# Diagrama de Componentes

## Componentes del Sistema

```mermaid
graph TB
    subgraph Frontend["FRONTEND - Aplicación Web"]
        Dashboard["Dashboard"]
        AuthUI["Interfaz Autenticación"]
        PaperSubmit["Formulario Envío Papers"]
        ReviewUI["Interfaz Revisiones"]
        AdminPanel["Panel Administración"]
    end
    
    subgraph APIs["APIS REST"]
        AuthAPI["POST /auth/*<br/>Login/Logout"]
        PaperAPI["GET/POST /papers/*<br/>Gestión de papers"]
        ReviewAPI["GET/POST /reviews/*<br/>Revisiones"]
        UserAPI["GET /users/*<br/>Gestión de usuarios"]
        ProgramAPI["GET /program/*<br/>Programa final"]
        ReportAPI["GET /reports/*<br/>Reportes"]
    end
    
    subgraph Business["SERVICIOS DE NEGOCIO"]
        AuthService["AuthService<br/>Autenticación y JWT"]
        PaperService["PaperService<br/>Ingesta y validación"]
        AssignService["AssignService<br/>Distribución a revisores"]
        ReviewService["ReviewService<br/>Procesamiento de reviews"]
        NotificationService["NotificationService<br/>Envío de emails"]
        ProgramService["ProgramService<br/>Construcción del programa"]
        ReportService["ReportService<br/>Estadísticas y reportes"]
    end
    
    subgraph Data["REPOSITORIES - Acceso a Datos"]
        UserRepo["UserRepository"]
        PaperRepo["PaperRepository"]
        ReviewRepo["ReviewRepository"]
        AssignRepo["AssignRepository"]
        ProgramRepo["ProgramRepository"]
    end
    
    subgraph Database["BASE DE DATOS"]
        PostgreSQL["PostgreSQL"]
    end
    
    subgraph Cache["CACHE"]
        Redis["Redis"]
    end
    
    subgraph External["SERVICIOS EXTERNOS"]
        EmailProvider["Email Provider<br/>SendGrid/AWS SES"]
        FileStorage["File Storage<br/>AWS S3/Cloud Storage"]
    end
    
    Dashboard --> AuthAPI
    Dashboard --> PaperAPI
    Dashboard --> ReviewAPI
    Dashboard --> UserAPI
    Dashboard --> ProgramAPI
    
    AuthUI --> AuthAPI
    PaperSubmit --> PaperAPI
    ReviewUI --> ReviewAPI
    AdminPanel --> ReportAPI
    
    AuthAPI --> AuthService
    PaperAPI --> PaperService
    ReviewAPI --> ReviewService
    UserAPI --> AuthService
    ProgramAPI --> ProgramService
    ReportAPI --> ReportService
    
    AssignService -.-> NotificationService
    ReviewService -.-> NotificationService
    PaperService -.-> AssignService
    
    AuthService --> UserRepo
    PaperService --> PaperRepo
    ReviewService --> ReviewRepo
    AssignService --> AssignRepo
    ProgramService --> ProgramRepo
    
    UserRepo --> PostgreSQL
    PaperRepo --> PostgreSQL
    ReviewRepo --> PostgreSQL
    AssignRepo --> PostgreSQL
    ProgramRepo --> PostgreSQL
    
    PaperService --> Redis
    ReviewService --> Redis
    
    PaperService --> FileStorage
    NotificationService --> EmailProvider
```

## Matriz de Responsabilidades (RACI)

| Entidad | Create | Review | Actualizar | Eliminar |
|---------|--------|--------|------------|----------|
| Paper | Autor | Admin, Revisor | Autor (versiones) | Admin |
| Revisión | Revisor | - | Revisor | Admin |
| Usuario | Admin | - | Admin/self | Admin |
| Asignación | Admin/Sistema | - | Admin | Admin |
| Programa | ProgramService | Admin | Admin/Sistema | Admin |

## Interfaces Principales

### AuthService
```
- login(email, password) → Token JWT
- validateToken(token) → Usuario | null
- logout(token) → void
- assignRole(usuario_id, rol) → void
```

### PaperService
```
- submitPaper(paper, archivo, autor_id) → Paper
- getPapers(filtros) → Paper[]
- getDetailedPaper(paper_id) → Paper + Revisiones
- updatePaperVersion(paper_id, archivo) → PaperVersion
- changePaperStatus(paper_id, nuevoEstado) → void
```

### ReviewService
```
- getPapersToReview(revisor_id) → Paper[]
- submitReview(review_id, calificación, comentarios, recomendación) → Revisión
- getReviewsForPaper(paper_id) → Revisión[]
- calculateDecision(paper_id) → Decisión (Aceptar/Rechazar/Requiere Cambios)
```

### NotificationService
```
- notifySubmissionConfirmed(paper_id) → void
- notifyPaperAssigned(reviewer_id, paper_id) → void
- notifyDecision(author_id, paper_id, decisión) → void
- sendReminderEmail(revisor_id, paper_id) → void
```

### ProgramService
```
- selectAcceptedPapers() → Paper[]
- generateProgram() → Programa
- groupIntoSessions(papers) → Sesión[]
- publishProgram() → void
```
