# Patrones de Diseño y Casos de Uso

## 1. Diagrama de Casos de Uso (Use Cases)

```mermaid
graph TB
    subgraph Sistema["🎯 SISTEMA DE GESTIÓN DE CONFERENCIAS"]
        UC["(Sistema)"]
    end
    
    subgraph Actores
        Autor["👤 AUTOR"]
        Revisor["👁️ REVISOR"]
        Admin["🔧 ADMIN"]
        Org["📋 ORGANIZADOR"]
    end
    
    subgraph UseCasesA["Casos de Uso - AUTOR"]
        UC1["Registrarse"]
        UC2["Autenticarse"]
        UC3["Completar Perfil"]
        UC4["Enviar Paper"]
        UC5["Ver Estado Paper"]
        UC6["Recibir Feedback"]
        UC7["Enviar Cambios"]
    end
    
    subgraph UseCasesR["Casos de Uso - REVISOR"]
        UC8["Registrarse"]
        UC9["Autenticarse"]
        UC10["Ver Papers Asignados"]
        UC11["Revisar Paper"]
        UC12["Enviar Evaluación"]
        UC13["Ver Historiales"]
    end
    
    subgraph UseCasesO["Casos de Uso - ORGANIZADOR"]
        UC14["Crear Conferencia"]
        UC15["Lanzar Call for Papers"]
        UC16["Gestionar Panel"]
        UC17["Distribuir Papers"]
        UC18["Ver Avances"]
        UC19["Generar Programa"]
        UC20["Publicar Programa"]
    end
    
    subgraph UseCasesAdm["Casos de Uso - ADMIN"]
        UC21["Gestionar Usuarios"]
        UC22["Exportar Datos"]
        UC23["Ver Auditoría"]
        UC24["Respaldar Sistema"]
    end
    
    Autor --> UC1
    Autor --> UC2
    Autor --> UC3
    Autor --> UC4
    Autor --> UC5
    Autor --> UC6
    Autor --> UC7
    
    Revisor --> UC8
    Revisor --> UC9
    Revisor --> UC10
    Revisor --> UC11
    Revisor --> UC12
    Revisor --> UC13
    
    Org --> UC14
    Org --> UC15
    Org --> UC16
    Org --> UC17
    Org --> UC18
    Org --> UC19
    Org --> UC20
    
    Admin --> UC21
    Admin --> UC22
    Admin --> UC23
    Admin --> UC24
    
    UC1 -.-> UC["Crear Usuario"]
    UC2 -.-> UC["Validar Credenciales"]
    UC3 -.-> UC["Actualizar Perfil"]
    UC4 -.-> UC["Procesar Paper"]
    UC5 -.-> UC["Consultar Estado"]
    UC7 -.-> UC["Actualizar Paper"]
    UC11 -.-> UC["Almacenar Evaluación"]
    UC17 -.-> UC["Asignar Revisores"]
    UC19 -.-> UC["Agrupar Papers"]
```

## 2. Patrones de Diseño Aplicables

### Patrón: Observer (Observador)
**Problemática**: Cuando un paper cambia de estado, múltiples sistemas necesitan reaccionar:
- Notificación al autor
- Actualización de estadísticas
- Auditoría
- Emails

**Solución**:
```typescript
interface Observer {
  update(paper: Paper, evento: string): void;
}

class PaperStateChangeObserver implements Observer {
  constructor(private notificationService: NotificationService) {}
  
  update(paper: Paper, evento: string) {
    switch(evento) {
      case 'ASIGNADO':
        this.notificationService.notifyAssignment(paper);
        break;
      case 'DECISION_TOMADA':
        this.notificationService.notifyDecision(paper);
        break;
    }
  }
}

class Paper {
  private observers: Observer[] = [];
  
  subscribe(observer: Observer) {
    this.observers.push(observer);
  }
  
  changeStatus(nuevoEstado: string) {
    this.estado = nuevoEstado;
    this.notifyObservers('STATUS_CHANGED');
  }
  
  private notifyObservers(evento: string) {
    this.observers.forEach(obs => obs.update(this, evento));
  }
}
```

### Patrón: Strategy (Estrategia)
**Problemática**: La decisión de aceptar/rechazar un paper puede tener diferentes algoritmos:
- Promedio simple
- Ponderado por experiencia del revisor
- Con veto (si alguien rechaza, se rechaza)

**Solución**:
```typescript
interface DecisionStrategy {
  decide(reviews: Review[]): Decision;
}

class AverageStrategy implements DecisionStrategy {
  decide(reviews: Review[]): Decision {
    const avg = reviews.reduce((s, r) => s + r.calificacion, 0) / reviews.length;
    return avg >= 7 ? Decision.ACCEPT : Decision.REJECT;
  }
}

class WeightedVotingStrategy implements DecisionStrategy {
  decide(reviews: Review[]): Decision {
    const votos = {
      aceptar: reviews.filter(r => r.recomendacion === 'ACEPTAR').length,
      rechazar: reviews.filter(r => r.recomendacion === 'RECHAZAR').length
    };
    return votos.aceptar > votos.rechazar ? Decision.ACCEPT : Decision.REJECT;
  }
}

class ReviewService {
  constructor(private strategy: DecisionStrategy) {}
  
  makeDecision(paper: Paper): Decision {
    const reviews = this.getReviews(paper.id);
    return this.strategy.decide(reviews);
  }
}
```

### Patrón: Builder (Constructor)
**Problemática**: Crear un Programa es complejo (múltiples sesiones, horarios, validaciones)

**Solución**:
```typescript
class ProgramBuilder {
  private sessions: Session[] = [];
  private acceptedPapers: Paper[] = [];
  
  addPAPER(paper: Paper): this {
    this.acceptedPapers.push(paper);
    return this;
  }
  
  createSessionFrom(papers: Paper[], date: Date): this {
    const session = new Session(date);
    session.addPapers(papers);
    this.sessions.push(session);
    return this;
  }
  
  validateAndBuild(): Program {
    if (this.sessions.length === 0) throw new Error('No sessions');
    if (this.acceptedPapers.length === 0) throw new Error('No papers');
    
    return new Program(this.sessions, this.acceptedPapers);
  }
}

// Uso
const program = new ProgramBuilder()
  .addPaper(paper1)
  .addPaper(paper2)
  .createSessionFrom([paper1, paper2], new Date('2024-06-15'))
  .validateAndBuild();
```

### Patrón: Repository
**Problemática**: Necesitamos abstracción para cambiar la BD sin afectar la lógica

**Solución**:
```typescript
interface PaperRepository {
  create(paper: Paper): Promise<Paper>;
  findById(id: number): Promise<Paper | null>;
  findByAuthor(authorId: number): Promise<Paper[]>;
  update(paper: Paper): Promise<void>;
  delete(id: number): Promise<void>;
}

class PostgreSQLPaperRepository implements PaperRepository {
  async create(paper: Paper): Promise<Paper> {
    const result = await db.query(
      'INSERT INTO paper (...) VALUES (...) RETURNING *',
      [paper.titulo, paper.resumen, ...]
    );
    return result.rows[0];
  }
  // ...
}

class PaperService {
  constructor(private repo: PaperRepository) {}
  
  async getPaper(id: number) {
    return this.repo.findById(id);
  }
}
```

### Patrón: Dependency Injection (DI)
**Problemática**: Acoplamiento fuerte entre servicios

**Solución** (con NestJS):
```typescript
@Injectable()
export class ReviewService {
  constructor(
    private readonly reviewRepository: ReviewRepository,
    private readonly notificationService: NotificationService,
    private readonly assignmentService: AssignmentService
  ) {}
  
  async submitReview(review: Review): Promise<void> {
    await this.reviewRepository.save(review);
    await this.notificationService.notifyPaperReviewedEvent(review);
    await this.assignmentService.updateAssignmentStatus(review.assignmentId);
  }
}
```

---

## 3. Flujo de Asignación de Papers (Algoritmo)

```mermaid
flowchart TD
    A["📥 Se inicia asignación de papers"] --> B["📊 Papers listos: N papers"]
    B --> C["👥 Revisores disponibles: M revisores"]
    C --> D["⚙️ Algoritmo de asignación"]
    
    D --> E["1️⃣ Para cada paper:"]
    E --> F["Obtener 3 mejores revisores"]
    F --> G["Basado en:"]
    G -->|checkbox| G1["✓ Área de especialidad"]
    G -->|checkbox| G2["✓ Carga actual < 5 papers"]
    G -->|checkbox| G3["✓ NO conflicto (mismo lab/institución)"]
    G -->|checkbox| G4["✓ Puntuación de confiabilidad"]
    
    F --> H["Revisores seleccionados"]
    H --> I["✔️ Crear Asignación"]
    I --> J["📧 Notificar revisor"]
    J --> K["⏱️ Establecer fecha límite"]
    K --> L["💾 Guardar en BD"]
    L --> M{¿Más papers?}
    M -->|Sí| E
    M -->|No| N["✅ Asignación completada"]
```

---

## 4. Máquina de Estados: Paper

```mermaid
stateDiagram-v2
    [*] --> Enviado: submit_paper()
    
    Enviado --> Validando: validate_format()
    Validando --> EnRevision: validation_passed()
    Validando --> Rechazado: validation_failed()
    
    EnRevision --> ConCambios: 1+ cambios recomendados
    EnRevision --> Aceptado: 2-3 aceptaciones
    EnRevision --> Rechazado: 2-3 rechazos
    
    ConCambios --> Modificando: author_starts_editing()
    Modificando --> EnRevisionFinal: resubmit_paper()
    EnRevisionFinal --> Aceptado: final_approval()
    EnRevisionFinal --> Rechazado: final_rejection()
    
    Aceptado --> EnPrograma: schedule_in_program()
    Aceptado --> Presentado: paper_presented()
    
    Rechazado --> [*]
    Presentado --> [*]
    
    note right of Enviado
        Validar PDF, tamaño,
        metadatos, resumen
    end note
    
    note right of EnRevision
        3 revisores evalúan
        simultáneamente
    end note
    
    note right of ConCambios
        Dar 2 semanas para
        ajustes del autor
    end note
```

---

## 5. Matriz de Permisos (RBAC)

| Recurso | Crear | Leer | Actualizar | Eliminar |
|---------|-------|------|-----------|----------|
| **Paper** |
| - Propios | ✅ | ✅ | ❌ | ❌ |
| - De otros (Autor) | ❌ | ❌ | ❌ | ❌ |
| - De otros (Admin) | ✅ | ✅ | ✅ | ✅ |
| **Review** |
| - Propias (Revisor) | ✅ | ✅ | ❌ | ❌ |
| - Ajenas (Revisor) | ❌ | ❌ | ❌ | ❌ |
| - Todas (Admin) | ✅ | ✅ | ✅ | ✅ |
| **Usuario** |
| - Propio perfil | ❌ | ✅ | ✅ | ❌ |
| - Otros (Admin) | ✅ | ✅ | ✅ | ✅ |
| **Programa** |
| - Consultar (Todos) | ❌ | ✅ | ❌ | ❌ |
| - Modificar (Org) | ✅ | ✅ | ✅ | ✅ |

---

## 6. Integraciones Necesarias

```mermaid
graph TB
    Sistema["🎯 Sistema Central<br/>Conference Manager"]
    
    Sistema -->|Envío de Emails| EmailSvc["📧 SendGrid/AWS SES"]
    Sistema -->|Almacena PDFs| StorageSvc["☁️ AWS S3 / Azure Blob"]
    Sistema -->|Autenticación| AuthSvc["🔐 OAuth 2.0 (optional)"]
    Sistema -->|Calendario| CalSvc["📅 Google Calendar API (optional)"]
    Sistema -->|Analytics| AnalyticsSvc["📊 Mixpanel / Google Analytics"]
    
    EmailSvc -->|Confirmación envío| Usuario1["Autor"]
    EmailSvc -->|Paper asignado| Usuario2["Revisor"]
    EmailSvc -->|Decisión final| Usuario3["Autor"]
    
    StorageSvc -->|PDF cargado| Sistema
    Sistema -->|Descargar PDF| Usuario1
    Sistema -->|Ver PDF| Usuario2
```

---

## 7. Ejemplo de Endpoint REST API

### POST /papers - Enviar un Paper

```
POST /papers HTTP/1.1
Host: conference.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="titulo"

Machine Learning in Medical Imaging

------WebKitFormBoundary
Content-Disposition: form-data; name="resumen"

This paper presents a novel approach to automated disease detection...

------WebKitFormBoundary
Content-Disposition: form-data; name="palabras_clave"

machine learning, medical imaging, deep learning, diagnostics

------WebKitFormBoundary
Content-Disposition: form-data; name="archivo"; filename="paper.pdf"
Content-Type: application/pdf

[Binary PDF content...]

------WebKitFormBoundary--
```

**Response (201 Created)**:
```json
{
  "id": 1024,
  "titulo": "Machine Learning in Medical Imaging",
  "resumen": "This paper presents...",
  "estado": "ENVIADO",
  "autor_id": 42,
  "fecha_envio": "2024-06-10T14:23:55Z",
  "version_actual": 1,
  "url_revision": "https://conference.example.com/paper/1024"
}
```

### GET /papers/{id}/reviews - Obtener Reviews de un Paper

```
GET /papers/1024/reviews HTTP/1.1
Host: conference.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response (200 OK)**:
```json
[
  {
    "id": 501,
    "calificacion": 8,
    "recomendacion": "ACEPTAR",
    "comentarios": "Excellent work, well executed study...",
    "fecha_entrega": "2024-06-25T10:15:30Z"
  },
  {
    "id": 502,
    "calificacion": 7,
    "recomendacion": "REQUIERE_CAMBIOS",
    "comentarios": "Good contribution but needs clarification on methodology...",
    "fecha_entrega": "2024-06-26T09:45:20Z"
  }
]
```
