# Índice de Diagramas PlantUML - Referencia Rápida

## 📊 Resumen Rápido

**Total de diagramas creados: 26**

| # | Diagrama | Tipo | Archivo | Líneas |
|---|----------|------|---------|--------|
| 1 | C4_Contexto | C4 Model | 1 | 1-34 |
| 2 | C4_Contenedores | C4 Container | 1 | 36-74 |
| 3 | Componentes_Backend | Component | 1 | 76-155 |
| 4 | StateMachine_Paper | State | 1 | 157-186 |
| 5 | UML_Classes | Class | 1 | 188-281 |
| 6 | Sequence_SubmitPaper | Sequence | 1 | 283-332 |
| 7 | Sequence_Distribution | Sequence | 1 | 334-384 |
| 8 | Sequence_Selection | Sequence | 1 | 386-433 |
| 9 | Activity_CompleteCycle | Activity | 2 | 1-91 |
| 10 | Activity_SubmitReview | Activity | 2 | 93-129 |
| 11 | Deployment_Architecture | Deployment | 2 | 131-196 |
| 12 | Activity_Notifications | Activity | 2 | 198-249 |
| 13 | Activity_Notifications (alt) | Activity | 2 | 251-301 |
| 14 | Integration_Services | Component | 2 | 303-357 |
| 15 | Activity_Security | Activity | 2 | 359-427 |
| 16 | Activity_AcceptanceLogic | Activity | 2 | 429-489 |
| 17 | UseCase_Main | Use Case | 2 | 491-551 |
| 18 | ERDiagram_Database | ER | 3 | 1-119 |
| 19 | Activity_MessageQueue | Activity | 3 | 121-170 |
| 20 | Sequence_ReviewerWorkflow | Sequence | 3 | 172-241 |
| 21 | Activity_CompilationDecision | Activity | 3 | 243-305 |
| 22 | ModuleView | Module | 3 | 307-367 |
| 23 | UI_AuthorDashboard | UI Mock | 3 | 369-428 |
| 24 | StateMachine_Review | State | 3 | 430-445 |
| 25 | Activity_DistributionRules | Activity | 3 | 447-527 |
| 26 | Sequence_APICall | Sequence | 3 | 529-595 |

---

## 🎯 Diagramas por Propósito

### Comunicación con Stakeholders
- ✅ C4_Contexto
- ✅ C4_Contenedores
- ✅ Activity_CompleteCycle
- ✅ UseCase_Main
- ✅ UI_AuthorDashboard

### Diseño de Arquitectura
- ✅ Componentes_Backend
- ✅ ModuleView
- ✅ Deployment_Architecture
- ✅ Integration_Services
- ✅ ERDiagram_Database

### Modelado de Datos
- ✅ UML_Classes
- ✅ ERDiagram_Database
- ✅ StateMachine_Paper
- ✅ StateMachine_Review

### Procesos y Flujos
- ✅ Activity_CompleteCycle
- ✅ Activity_SubmitReview
- ✅ Activity_Notifications
- ✅ Activity_Security
- ✅ Activity_AcceptanceLogic
- ✅ Activity_DistributionRules
- ✅ Activity_MessageQueue
- ✅ Activity_CompilationDecision

### Interacciones y Secuencias
- ✅ Sequence_SubmitPaper
- ✅ Sequence_Distribution
- ✅ Sequence_Selection
- ✅ Sequence_ReviewerWorkflow
- ✅ Sequence_APICall

### Infraestructura y DevOps
- ✅ Deployment_Architecture
- ✅ Activity_MessageQueue
- ✅ Integration_Services

---

## 🎨 Tipos de Diagramas Utilizados

| Tipo | Cantidad | Casos de Uso |
|------|----------|-------------|
| **Sequence** | 5 | Flujos paso a paso, interacciones |
| **Activity** | 8 | Procesos complejos, decisiones |
| **Class** | 1 | Modelado OOP |
| **State** | 2 | Máquinas de estado |
| **Component** | 3 | Arquitectura |
| **Deployment** | 1 | Infraestructura |
| **Use Case** | 1 | Requerimientos |
| **ER** | 1 | Modelo de datos |
| **Module** | 1 | Estructura de código |
| **C4** | 2 | Arquitectura a nivel sistema |
| **UI Mock** | 1 | Interfaces |

---

## 📋 Cómo Extraer un Diagrama Específico

### Desde archivo 1: DIAGRAMAS_PLANTUML_1_CONTEXTO_COMPONENTES.puml

```bash
# Extraer diagrama específico
sed -n '/@startuml Sequence_SubmitPaper/,/@enduml/p' DIAGRAMAS_PLANTUML_1_CONTEXTO_COMPONENTES.puml > sequence_submit.puml

# Luego visualizar
plantuml sequence_submit.puml -o output
```

### Script para extraer todos

```bash
#!/bin/bash
# extract_diagrams.sh

for file in DIAGRAMAS_PLANTUML_*.puml; do
    echo "Procesando $file..."
    
    # Extraer nombre del diagrama (entre @startuml y @enduml)
    grep -o "@startuml [^[:space:]]*" "$file" | cut -d' ' -f2 | while read diagram; do
        echo "  - Extrayendo $diagram"
        sed -n "/@startuml $diagram/,/@enduml/p" "$file" > "diagrams/$diagram.puml"
    done
done

echo "✓ Diagramas extraídos en carpeta 'diagrams/'"
```

---

## 🚀 Generar Todos los Diagramas a la Vez

### Opción 1: Uno por uno

```bash
plantuml DIAGRAMAS_PLANTUML_1_CONTEXTO_COMPONENTES.puml -o output -tpng
plantuml DIAGRAMAS_PLANTUML_2_FLUJOS_DESPLIEGUE.puml -o output -tpng
plantuml DIAGRAMAS_PLANTUML_3_DATOS_DETALLES.puml -o output -tpng
```

### Opción 2: Todos a la vez

```bash
for file in DIAGRAMAS_PLANTUML_*.puml; do
    plantuml "$file" -o output -tpng
done
```

### Opción 3: Con Docker

```bash
docker run --rm -v $(pwd):/data plantuml/plantuml:latest \
  bash -c 'for f in /data/DIAGRAMAS*.puml; do plantuml $f -o /data/output -tpng; done'
```

### Opción 4: Generar SVG (recomendado para web)

```bash
for file in DIAGRAMAS_PLANTUML_*.puml; do
    plantuml "$file" -o output -tsvg
done
```

---

## 📍 Flujos Relacionados

### Flujo completo: Envío de Paper
1. `Sequence_SubmitPaper` - Detalles técnicos
2. `UI_AuthorDashboard` - Cómo se ve
3. `Activity_CompleteCycle` - Contexto general

### Flujo completo: Revisión
1. `Activity_SubmitReview` - Cómo lo ve revisor
2. `Sequence_ReviewerWorkflow` - Detalles
3. `StateMachine_Review` - Estados
4. `Activity_MessageQueue` - Notificaciones

### Flujo completo: Distribución
1. `Sequence_Distribution` - Técnico
2. `Activity_DistributionRules` - Lógica
3. `Activity_Notifications` - Notificaciones

### Flujo completo: Selección
1. `Sequence_Selection` - Técnico
2. `Activity_AcceptanceLogic` - Lógica
3. `Activity_CompilationDecision` - Detalles
4. `Activity_Notifications` - Notificaciones

---

## 🔗 Relaciones entre Diagramas

```
C4_Contexto
    ├─→ C4_Contenedores
    │   ├─→ Componentes_Backend
    │   │   ├─→ UML_Classes
    │   │   ├─→ ModuleView
    │   │   └─→ Sequence_APICall
    │   ├─→ Deployment_Architecture
    │   ├─→ Integration_Services
    │   └─→ ERDiagram_Database
    │
    ├─→ Activity_CompleteCycle
    │   ├─→ Sequence_SubmitPaper
    │   ├─→ Sequence_Distribution
    │   ├─→ Sequence_ReviewerWorkflow
    │   ├─→ Sequence_Selection
    │   └─→ Activity_CompilationDecision
    │
    ├─→ UseCase_Main
    │
    └─→ Activity_Security
        └─→ Activity_Notifications
            └─→ Activity_MessageQueue
```

---

## 📊 Cobertura del Sistema

| Área | Cobertura | Diagramas |
|------|-----------|-----------|
| **Procesos de Negocio** | 100% | Activity_CompleteCycle, UseCase_Main |
| **Arquitectura Técnica** | 100% | C4_*, Componentes_Backend, ModuleView |
| **Base de Datos** | 100% | ERDiagram_Database, UML_Classes |
| **Flujos de Usuario** | 90% | Sequence_*, Activity_* |
| **Infraestructura** | 100% | Deployment_Architecture, Integration_Services |
| **Seguridad** | 95% | Activity_Security, Sequence_APICall |
| **Documentación UI** | 50% | UI_AuthorDashboard (solo 1 vista) |

---

## 🎓 Recursos Complementarios

- `00_GUIA_LECTURA.md` - Guía general
- `GUIA_DIAGRAMAS_PLANTUML.md` - Cómo usar los diagramas
- `ARQUITECTURA_SOFTWARE.md` - Detalle técnico
- `DIAGRAMAS_COMPONENTES_TECNICO.md` - Diagramas ASCII
- `RECOMENDACIONES_LENGUAJES_PATRONES.md` - Decisiones técnicas

---

## 🔄 Sincronización con Documentos

Los diagramas deben mantenerse sincronizados con:

| Documento | Diagramas Relacionados |
|-----------|----------------------|
| ARQUITECTURA_GENERAL.md | Activity_CompleteCycle, UseCase_Main |
| ARQUITECTURA_SOFTWARE.md | Componentes_Backend, ERDiagram_Database, ModuleView |
| DIAGRAMAS_COMPONENTES_TECNICO.md | Deployment_Architecture, Integration_Services |
| RECOMENDACIONES_LENGUAJES_PATRONES.md | Componentes_Backend, ModuleView |

---

## 💾 Almacenamiento y Versionamiento

```
. (root)
├── DIAGRAMAS_PLANTUML_1_CONTEXTO_COMPONENTES.puml
├── DIAGRAMAS_PLANTUML_2_FLUJOS_DESPLIEGUE.puml
├── DIAGRAMAS_PLANTUML_3_DATOS_DETALLES.puml
├── GUIA_DIAGRAMAS_PLANTUML.md
├── INDICE_DIAGRAMAS_PLANTUML.md (este archivo)
├── .gitignore
│   └── # output/ (no versionar imágenes)
└── output/ (generado)
    ├── diagrams-png/
    └── diagrams-svg/
```

---

## 🏆 Best Practices

1. ✅ Mantener archivos .puml en control de versiones
2. ✅ Regenerar imágenes antes de commits importantes
3. ✅ Usar nómbreclaro para diagramas
4. ✅ Documentar cambios en comentarios
5. ✅ Revisar diagrams en PRs
6. ✅ Mantener actualizados con cambios en arquitectura

---

## 📞 Próximos Pasos

- [ ] Generar imágenes PNG/SVG de todos los diagramas
- [ ] Incluir diagramas en documentación de wikis
- [ ] Crear diapositivas de presentación con diagramas
- [ ] Mantener diagramas sincronizados durante desarrollo
- [ ] Agregar más diagramas para funcionalidades futuras

---

**Última actualización:** Febrero 2026
**Total de diagramas:** 26
**Cobertura del sistema:** ~95%
