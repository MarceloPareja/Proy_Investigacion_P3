# Arquitectura del Sistema de Gestión de Conferencias

## 1. ACTORES DEL SISTEMA

### 1.1 Administrador
- Responsable de recibir y registrar papers
- Gestiona usuarios del sistema
- Distribuye papers a revisores
- Notifica a autores de resultados
- Construye el programa final

### 1.2 Autor
- Envía papers para la conferencia
- Recibe notificaciones de estado
- Realiza cambios solicitados por revisores
- Envía versión final del paper

### 1.3 Revisor
- Evalúa papers asignados (máximo 3 por revisor)
- Genera informes de revisión
- Proporciona recomendaciones (aceptar, rechazar, revisar)
- Sugiere cambios si es necesario

### 1.4 Comité de Selección
- Toma decisiones finales sobre aceptación/rechazo
- Usa evaluaciones de revisores como referencia
- Selecciona papers para el programa

## 2. FASES DEL PROCESO

### Fase 1: CONVOCATORIA (Call for Papers)
**Inicio:** Anuncio público
**Responsable:** Administrador
**Salida:** Papers recibidos

### Fase 2: RECEPCIÓN DE PAPERS
**Responsable:** Administrador
**Actividades:**
- Recibir papers de autores
- Registrar información del paper (título, autores, resumen, archivos)
- Validar formato y completitud
- Asignar ID único a cada paper
**Salida:** Papers registrados en el sistema

### Fase 3: DISTRIBUCIÓN A REVISORES
**Responsable:** Administrador
**Timing:** En fecha determinada
**Actividades:**
- Seleccionar panel de revisores
- Asignar 3 revisores a cada paper
- Enviar papers a revisores
- Establecer plazo de revisión
**Salida:** Papers distribuidos

### Fase 4: REVISIÓN Y EVALUACIÓN
**Responsable:** Revisores
**Actividades:**
- Analizar paper
- Evaluar calidad, originalidad, relevancia
- Generar informe de revisión
- Proporcionar recomendación (Aceptar/Rechazar/Revisar)
- Sugiere cambios si es necesario
**Salida:** Reportes de revisión completados

### Fase 5: SELECCIÓN DE PAPERS
**Responsable:** Comité de Selección
**Timing:** En fecha establecida
**Actividades:**
- Revisar evaluaciones de los 3 revisores
- Tomar decisión final
- Clasificar papers (Aceptados/Rechazados/Requiere cambios)
**Salida:** Decisiones finales

### Fase 6: NOTIFICACIÓN Y REVISIONES
**Responsable:** Administrador
**Actividades:**
- Notificar a autores resultados
- Papers aceptados con cambios solicitados
- Establecer plazo para envío de versión final
- Autores realizan cambios solicitados
**Salida:** Versiones finales recibidas

### Fase 7: CONSTRUCCIÓN DEL PROGRAMA FINAL
**Responsable:** Administrador
**Actividades:**
- Compilar papers aceptados
- Organizar sesiones y cronograma
- Definir orden de presentaciones
- Publicar programa final
**Salida:** Programa de conferencia publicado

## 3. ENTIDADES PRINCIPALES DEL SISTEMA

### 3.1 Paper
- ID único
- Título
- Autores (nombres, afiliaciones, emails)
- Resumen
- Palabras clave
- Archivo PDF
- Fecha de envío
- Estado (Registrado, En revisión, Aceptado, Rechazado, Revisión requerida)
- Versión (inicial, revisada)

### 3.2 Revisor
- ID único
- Nombre
- Afiliación
- Email
- Especialidad/Área de expertise
- Papers asignados

### 3.3 Revisión/Informe
- ID único
- ID del Paper
- ID del Revisor
- Fecha de envío
- Puntuación/Calificación
- Comentarios generales
- Comentarios detallados
- Recomendación (Aceptar/Rechazar/Revisar)
- Cambios sugeridos

### 3.4 Decisión Final
- ID único
- ID del Paper
- Decisión (Aceptado/Rechazado/Requiere cambios)
- Comentarios del comité
- Cambios requeridos (si aplica)
- Fecha de notificación

### 3.5 Usuario del Sistema
- ID único
- Rol (Administrador, Autor, Revisor)
- Nombre
- Email
- Contraseña (hash)
- Activo/Inactivo

## 4. MÓDULOS DEL SISTEMA

### Módulo 1: GESTIÓN DE USUARIOS Y AUTENTICACIÓN
**Responsabilidades:**
- Registro de usuarios
- Autenticación y autorización
- Gestión de roles y permisos
- Control de acceso

### Módulo 2: GESTIÓN DE PAPERS
**Responsabilidades:**
- Envío de papers (autores)
- Almacenamiento de papers
- Control de versiones
- Validación de papers
- Búsqueda y filtrado

### Módulo 3: GESTIÓN DE REVISIONES
**Responsabilidades:**
- Asignación de papers a revisores
- Recopilación de informes
- Almacenamiento de evaluaciones
- Tracking de estado de revisiones
- Recordatorios a revisores

### Módulo 4: SELECCIÓN Y DECISIONES
**Responsabilidades:**
- Compilación de evaluaciones
- Apoyo en toma de decisiones
- Registro de decisiones
- Generación de reportes

### Módulo 5: NOTIFICACIONES Y COMUNICACIONES
**Responsabilidades:**
- Envío de emails a autores
- Envío de emails a revisores
- Notificaciones de estado
- Recordatorios de plazos

### Módulo 6: GESTIÓN DE CONFERENCIA
**Responsabilidades:**
- Construcción de programa
- Gestión de sesiones
- Publicación del programa final
- Reportes y estadísticas

## 5. FLUJO DE DATOS GENERAL

```
┌─────────────────────────────────────────────────────────────┐
│  ADMINISTRADOR ANUNCIA CALL FOR PAPERS                      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  AUTORES ENVÍAN PAPERS (Sistema recibe y registra)          │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  ADMINISTRADOR DISTRIBUYE PAPERS A 3 REVISORES CADA UNO    │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  REVISORES EVALÚAN Y ENVÍAN INFORMES                        │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  COMITÉ TOMA DECISIONES (Aceptar/Rechazar/Cambios)         │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  ADMINISTRADOR NOTIFICA A AUTORES                           │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  AUTORES ENVÍAN VERSIÓN FINAL (si aplica)                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  ADMINISTRADOR CONSTRUYE PROGRAMA FINAL                     │
└─────────────────────────────────────────────────────────────┘
```

## 6. RESTRICCIONES Y REGLAS DE NEGOCIO

### Restricciones Técnicas
- Cada paper se asigna a exactamente 3 revisores
- Cada revisor recibe múltiples papers para evaluar
- Un revisor NO puede revisar un paper de su propia autoría
- El sistema está disponible vía web

### Restricciones de Proceso
- La distribución a revisores ocurre en una fecha determinada
- La selección final ocurre en una fecha establecida
- Los autores tienen plazo para enviar versión final
- Solo papers aceptados pueden tener versión final

### Reglas de Toma de Decisión
- La decisión se basa en evaluaciones de revisores
- Se requieren 3 evaluaciones por paper para tomar decisión
- Papers pueden requerer cambios según evaluaciones
- Decisión final: Aceptado, Rechazado, o Requiere cambios

## 7. REQUERIMIENTOS DE INFORMACIÓN

### Para Administrador
- Dashboard de papers recibidos
- Estado de distribución a revisores
- Informes de progreso de revisiones
- Resumen de decisiones
- Lista de tareas por fecha

### Para Autor
- Estado de su paper
- Comentarios de revisores (resumen)
- Decisión final
- Cambios solicitados (si aplica)

### Para Revisor
- Lista de papers asignados
- Formulario de revisión
- Plazo de entrega
- Recordatorios

### Para Comité
- Compilación de evaluaciones por paper
- Recomendaciones de revisores
- Estadísticas de evaluaciones
- Herramientas para comparar papers

## 8. REQUERIMIENTOS NO FUNCIONALES

- **Seguridad:** Autenticación segura, datos encriptados
- **Disponibilidad:** Sistema accesible vía web 24/7
- **Escalabilidad:** Soportar cientos de papers y revisores
- **Rendimiento:** Respuestas rápidas
- **Usabilidad:** Interfaz intuitiva para todos los actores
- **Confiabilidad:** Respaldos automáticos, integridad de datos
- **Auditoría:** Registro de todas las acciones del sistema

## 9. INTERFACES PRINCIPALES DEL SISTEMA

### 9.1 Portal de Autores
- Registro de cuenta
- Envío de paper
- Consulta de estado
- Descarga de comentarios
- Envío de versión revisada

### 9.2 Portal de Revisores
- Acceso a papers asignados
- Formulario de revisión
- Envío de informe
- Historial de revisiones

### 9.3 Sistema de Administración
- Gestión de usuarios
- Distribución de papers
- Monitoreo de progreso
- Generación de reportes
- Construcción de programa

### 9.4 Área del Comité
- Visualización de evaluaciones compiladas
- Herramientas de decisión
- Registro de decisiones finales
