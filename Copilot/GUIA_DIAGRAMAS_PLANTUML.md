# Guía de Uso: Diagramas PlantUML

## 📊 Archivos de Diagramas Creados

Se han creado 3 archivos con diagramas PlantUML:

| Archivo | Cantidad | Tipo | Uso |
|---|---|---|---|
| **DIAGRAMAS_PLANTUML_1_CONTEXTO_COMPONENTES.puml** | 8 diagramas | C4, UML, Secuencia | Visión General, Modelado, Flujos |
| **DIAGRAMAS_PLANTUML_2_FLUJOS_DESPLIEGUE.puml** | 9 diagramas | Actividad, Despliegue, UseCase | Procesos, Infraestructura, Casos de Uso |
| **DIAGRAMAS_PLANTUML_3_DATOS_DETALLES.puml** | 9 diagramas | ER, Secuencia, Módulos, Estados | Datos, Detalles, Arquitectura |

**Total: 26 diagramas diferentes**

---

## 🎨 Cómo Visualizar los Diagramas

### Opción 1: PlantUML Online (más rápido)

1. Ve a: https://www.plantuml.com/plantuml/uml/
2. Copia el contenido de un diagrama (entre @startuml y @enduml)
3. Pega en el editor
4. ¡Visualiza automáticamente!

### Opción 2: VS Code Extension

1. Instala: **"PlantUML"** by jebbs
2. Abre archivo .puml
3. Presiona `Alt + D` para obtener vista previa
4. Haz clic derecho → "Export to SVG/PNG"

### Opción 3: CLI command

```bash
# Instalar PlantUML
npm install -g plantuml

# Generar PNG
plantuml archivo.puml -o output_folder

# Generar SVG (mejor para web)
plantuml archivo.puml -tsvg -o output_folder
```

### Opción 4: Docker

```bash
docker run --rm -v $(pwd):/data rkrueger/plantuml \
  /data/archivo.puml -o /data/output
```

---

## 📋 Catálogo de Diagramas

### ARCHIVO 1: Contexto y Componentes

#### 1. **C4_Contexto** 
Muestra actores externos y sistema principal
```plaintext
Uso: Explicar a stakeholders qué es el sistema
Audiencia: Product managers, clientes
Formato: Diagrama de contexto
```

#### 2. **C4_Contenedores**
Desglosa tecnologías principales
```plaintext
Componentes:
- Frontend (React/Vue)
- API (Django REST)
- Base de Datos (PostgreSQL)
- Cache (Redis)
- Queue (RabbitMQ)
- Servicios Externos

Uso: Entender stack tecnológico
Audiencia: Architects, leads técnicos
```

#### 3. **Componentes_Backend**
Detalle de servicios backend y sus relaciones
```plaintext
Capas:
- API Controllers
- Business Services
- Repositories
- Database

Uso: Diseño de backend
Audiencia: Backend developers
```

#### 4. **StateMachine_Paper**
Estados por los que pasa un paper
```plaintext
Estados: DRAFT → SUBMITTED → REGISTERED → 
         UNDER_REVIEW → EVALUATED → DECISION → 
         ACCEPTED/REJECTED/CHANGES_REQUIRED

Uso: Entender ciclo de vida
Audiencia: QA, product, developers
```

#### 5. **UML_Classes**
Clases y sus reaciones (ERD conceptual)
```plaintext
Entidades principales:
- User
- Paper
- Review
- Decision

Uso: Modelado de dominio
Audiencia: Architecture, backend devs
```

#### 6. **Sequence_SubmitPaper**
Flujo completo de envío de paper
```plaintext
Actores:
- Autor (frontend)
- API (validación)
- Services (lógica)
- Database (persistencia)

Uso: Entender flujo end-to-end
Audiencia: Developers, QA
```

#### 7. **Sequence_Distribution**
Cómo se distribuyen papers a revisores
```plaintext
Paso a paso:
1. Admin inicia
2. Engine asigna 3 revisores/paper
3. Notificaciones enviadas
4. Queue de emails

Uso: Entender distribución
Audiencia: Admin, developers
```

#### 8. **Sequence_Selection**
Cómo el comité selecciona papers
```plaintext
Proceso:
1. Para cada paper: evalúa 3 reviews
2. Aplica criterios
3. Toma decisión
4. Notifica autores

Uso: Entender selección
Audiencia: Comité, admins
```

---

### ARCHIVO 2: Flujos y Despliegue

#### 9. **Activity_CompleteCycle**
Flujo completo desde convocatoria hasta programa
```plaintext
Fases:
1. Anuncio
2. Envío
3. Validación
4. Distribución
5. Evaluación
6. Decisión
7. Cambios/Final
8. Programa

Uso: Overview del proceso
Audiencia: Everyone
```

#### 10. **Activity_SubmitReview**
Proceso de un revisor completando una evaluación
```plaintext
Pasos:
1. Accede
2. Ve papers
3. Lee paper
4. Completa formulario
5. Valida
6. Envía o guarda borrador

Uso: Flujo de revisor
Audiencia: Revisores, support
```

#### 11. **Deployment_Architecture**
Arquitectura Kubernetes en producción
```plaintext
Componentes:
- Ingress Controller (nginx)
- API Pods (3 réplicas)
- Workers (Celery)
- Stateful Services (DB, Redis, RabbitMQ)
- Servicios externos

Uso: DevOps, deployment
Audiencia: DevOps, SRE, ops
```

#### 12 & 13. **Activity_Notifications** & **Activity_Notifications**
Sistema de notificaciones y colas asincrónicas
```plaintext
Flujo:
1. Evento generado
2. Entra en queue
3. Listeners procesan
4. Background jobs
5. Envío de emails

Uso: Entender notificaciones
Audiencia: Backend devs, ops
```

#### 14. **Integration_Services**
Integraciones con servicios externos
```plaintext
Integraciones:
- Email API (SendGrid)
- Storage (S3)
- Analytics (MixPanel)
- Auth (Auth0)

Uso: Entender terceros
Audiencia: Architects, devops
```

#### 15. **Activity_Security**
Capas de seguridad
```plaintext
Capas:
1. HTTPS/TLS
2. Firewall
3. Autenticación
4. Autorización
5. Validación
6. Lógica
7. DB
8. Auditoría

Uso: Entender seguridad
Audiencia: Security, leads
```

#### 16. **Activity_AcceptanceLogic**
Lógica de decisión para aceptar papers
```plaintext
Criterios:
- Score promedio
- Número de aceptaciones
- Consenso

Uso: Entender algoritmo
Audiencia: Product, developers
```

#### 17. **UseCase_Main**
Casos de uso principales del sistema
```plaintext
Actores: Autor, Revisor, Admin, Comité
Casos: Enviar, Distribuir, Evaluar, Seleccionar

Uso: Requerimientos
Audiencia: Everyone
```

---

### ARCHIVO 3: Datos y Detalles

#### 18. **ERDiagram_Database**
Modelo completo de datos
```plaintext
Entidades principales:
- USERS
- PAPERS
- REVIEWS
- DECISIONS
- NOTIFICATIONS
- AUDIT_LOG

Relaciones:
- Llaves foráneas
- Cardinalidades

Uso: Diseño de BD
Audiencia: DBA, backend devs
```

#### 19. **Activity_MessageQueue**
Flujo asincrónico de workers
```plaintext
Workers:
- Email Sender
- PDF Generator
- Analytics
- Indexer

Uso: Entender workers
Audiencia: Backend, devops
```

#### 20. **Sequence_ReviewerWorkflow**
Flujo completo de un revisor
```plaintext
Desde: Login
Hasta: Submensión de todas las evaluaciones

Uso: User story de revisor
Audiencia: Revisores, UX/UI
```

#### 21. **Activity_CompilationDecision**
Proceso de compilación y toma de decisión
```plaintext
Pasos:
1. Cierre de reviews
2. Compilación por paper
3. Cálculos
4. Comité vota
5. Notificaciones

Uso: Entender decisión final
Audiencia: Comité, admin
```

#### 22. **ModuleView**
Vista de módulos del sistema
```plaintext
Módulos:
- core (base)
- auth
- papers
- reviews
- selections
- notifications
- admin

Uso: Arquitectura modular
Audiencia: Architects, leads
```

#### 23. **UI_AuthorDashboard**
Mockup de dashboard de autor
```plaintext
Contenidos:
- Mis papers
- Estado y evaluaciones
- Enviar nuevo paper

Uso: Como debería verse
Audiencia: UI/UX, product
```

#### 24. **StateMachine_Review**
Máquina de estados de una revisión
```plaintext
Estados:
CREATED → DRAFT → SUBMITTED → COMPLETED → GRADED

Uso: Entender ciclo de review
Audiencia: Developers
```

#### 25. **Activity_DistributionRules**
Reglas de distribución a revisores
```plaintext
Filtros aplicados:
- Especialidad
- No conflicto
- Disponibilidad
- Track record

Uso: Algoritmo de distribución
Audiencia: Developers, product
```

#### 26. **Sequence_APICall**
Detalles técnicos de una llamada API
```plaintext
Desde: Cliente
Hasta: Respuesta desde BD

Incluyendo:
- Validación
- Autenticación
- Routing
- Serialización

Uso: Debugging, entendimiento técnico
Audiencia: Backend developers
```

---

## 🎯 Cómo Usar por Rol

### Product Manager
1. Ver: `C4_Contexto`
2. Ver: `Activity_CompleteCycle`
3. Ver: `UseCase_Main`

### Architect
1. Ver: `C4_Contexto`
2. Ver: `C4_Contenedores`
3. Ver: `Componentes_Backend`
4. Ver: `Deployment_Architecture`
5. Ver: `ERDiagram_Database`

### Backend Developer
1. Ver: `Componentes_Backend`
2. Ver: `ERDiagram_Database`
3. Ver: `Sequence_SubmitPaper`
4. Ver: `UML_Classes`
5. Ver: `ModuleView`
6. Ver: `Sequence_APICall`

### Frontend Developer
1. Ver: `UI_AuthorDashboard`
2. Ver: `Sequence_SubmitPaper`
3. Ver: `Sequence_ReviewerWorkflow`

### DevOps / Infrastructure
1. Ver: `Deployment_Architecture`
2. Ver: `Integration_Services`
3. Ver: `Activity_MessageQueue`
4. Ver: `Activity_Notifications`

### QA / Testing
1. Ver: `Activity_CompleteCycle`
2. Ver: `UseCase_Main`
3. Ver: `Activity_DistributionRules`
4. Ver: `Activity_AcceptanceLogic`

---

## 🔧 Personalizar Diagramas

### Cambiar colores

```plantuml
' Agregar skinparam al inicio
skinparam classBackgroundColor #FFE4E1
skinparam classBorderColor #FF69B4
skinparam backgroundColor #F0F8FF
```

### Usar tema diferente

```plantuml
!theme tufte
' o
!theme plain
!theme superhero
```

### Cambiar orientación

```plantuml
left to right direction
' o
top to bottom direction
```

### Agregar notas

```plantuml
note right : Esta es una nota
note left of Entity1 : Otra nota
```

---

## 📁 Estructura Recomendada

```
proyecto/
├── docs/
│   ├── diagrams/
│   │   ├── DIAGRAMAS_PLANTUML_1_*.puml
│   │   ├── DIAGRAMAS_PLANTUML_2_*.puml
│   │   ├── DIAGRAMAS_PLANTUML_3_*.puml
│   │   ├── output/
│   │   │   ├── diagrams-png/
│   │   │   ├── diagrams-svg/
│   │   │   └── diagrams-pdf/
│   │   └── README.md (este archivo)
│   └── architecture/
│       ├── ARQUITECTURA_GENERAL.md
│       ├── ARQUITECTURA_SOFTWARE.md
│       └── RECOMENDACIONES.md
```

---

## 🚀 Exportar para Presentaciones

### Generar todos los diagramas

```bash
# Script bash para generar todo
parallel -j4 'plantuml {} -tsvg -o output' ::: *.puml
```

### Crear documento combinado

```bash
# Combinar todos los SVGs en un PDF
convert output/*.svg diagrams-combined.pdf
```

### Para PowerPoint

1. Guardar los diagramas como PNG
2. Insertarten PowerPoint
3. Usar como background o contenido

---

## 💡 Tips Útiles

1. **Zoom en diagramas complejos:** Usa la extensión VS Code
2. **Versionamiento:** Guarda cambios en Git
3. **Documentación:** Usa la sección de notas para explicar
4. **Updates:** Si la lógica cambia, actualiza los .puml
5. **CI/CD:** Incluye generación de diagramas en el pipeline

---

## 🎓 Aprender Más

- **Documentación oficial:** https://plantuml.com/
- **Galería de ejemplos:** https://plantuml.com/guide
- **Cheat sheet:** https://plantuml.com/es/command-line
- **Community:** https://github.com/plantuml/plantuml

---

## 📞 Soporte

Si necesitas:
- **Modificar diagramas:** Edita los archivos .puml
- **Agregar nuevos:** Copia el patrón de un existente
- **Exportar a otros formatos:** PlantUML soporta: PNG, SVG, PDF, EPS

---

**¡Disfruta usando los diagramas!** 🎨
