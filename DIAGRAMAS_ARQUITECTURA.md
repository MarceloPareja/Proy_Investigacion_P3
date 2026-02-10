# Diagramas de Arquitectura - Sistema de Gestión de Conferencias

## DIAGRAMA 1: Arquitectura General del Sistema

```
┌────────────────────────────────────────────────────────────────────┐
│                    SISTEMA DE CONFERENCIAS WEB                     │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐ │
│  │  MÓDULO USUARIOS │  │ MÓDULO PAPERS    │  │ MÓDULO REVISIÓN  │ │
│  │  & AUTENTICACIÓN │  │                  │  │                  │ │
│  │                  │  │ - Almacenamiento │  │ - Asignación     │ │
│  │ - Login/Registro │  │ - Validación     │  │ - Recopilación   │ │
│  │ - Roles/Permisos │  │ - Versiones      │  │ - Tracking       │ │
│  └──────────────────┘  │ - Búsqueda       │  │ - Recordatorios  │ │
│                        └──────────────────┘  └──────────────────┘ │
│                                                                    │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐ │
│  │ MÓDULO SELECCIÓN │  │  MÓDULO           │  │ MÓDULO DE        │ │
│  │  & DECISIONES    │  │  NOTIFICACIONES  │  │  PROGRAMA         │ │
│  │                  │  │                  │  │                  │ │
│  │ - Compilación    │  │ - Emails         │  │ - Construcción   │ │
│  │ - Decisiones     │  │ - Recordatorios  │  │ - Programación   │ │
│  │ - Reportes       │  │ - Alertas        │  │ - Reportes       │ │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘ │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
              ┌────────────┐      ┌───────────┐
              │  BASE DE   │      │  SISTEMA  │
              │   DATOS    │      │ NOTIF.    │
              └────────────┘      └───────────┘
```

## DIAGRAMA 2: Modelo Conceptual de Datos

```
┌──────────────┐           ┌──────────────┐
│   USUARIO    │           │   PAPER      │
├──────────────┤           ├──────────────┤
│ ID           │           │ ID           │
│ Nombre       │           │ Título       │
│ Email        │           │ Autores      │
│ Rol          │           │ Resumen      │
│ Contraseña   │◄──────────┤ Archivo      │
│              │ (autor)   │ Estado       │
└──────────────┘           │ Fecha envío  │
      │                    └──────────────┘
      │                           │
      │                    ┌──────┴──────┐
      │                    │             │
      │              ┌─────────┐   ┌──────────┐
      │              │ REVISIÓN│   │ DECISIÓN │
      │              ├─────────┤   ├──────────┤
      │              │ ID      │   │ ID       │
      └─────────────►│ Revisor │   │ Paper    │
         (revisor)  │ Paper   │   │ Estado   │
                    │ Informe │   │ Cambios  │
                    │ Puntuaje│   └──────────┘
                    │ Comentarios
                    └─────────────┘

┌──────────────────────┐
│   PANEL REVISOR      │
├──────────────────────┤
│ ID                   │
│ Especialidad         │
│ Papers Asignados     │
│ Plazo                │
└──────────────────────┘
```

## DIAGRAMA 3: Ciclo de Vida del Paper

```
                    ┌─────────────┐
                    │  DRAFT      │ (Preparación por autor)
                    └──────┬──────┘
                           │ Autor envía
                           ▼
                    ┌─────────────┐
                    │  RECIBIDO   │ (Registrado por admin)
                    └──────┬──────┘
                           │ Admin distribuye
                           ▼
                    ┌──────────────────┐
                    │  EN_REVISIÓN     │ (Con 3 revisores)
                    │                  │
                    │ Revisores evalúan│
                    └──────┬───────────┘
                           │ 3 informes completados
                           ▼
                    ┌──────────────────┐
                    │  EVALUADO        │ (Listo para decisión)
                    └──────┬───────────┘
                           │ Comité decide
                           ▼
              ┌────────────┴─────────────┐
              │                          │
              ▼                          ▼
       ┌─────────────┐          ┌──────────────┐
       │ ACEPTADO    │          │  RECHAZADO   │
       └──────┬──────┘          └──────────────┘
              │                       │
       Puede requerer cambios        FIN
              │
              ▼
       ┌──────────────────┐
       │ CAMBIOS_REQUERIDO│
       └──────┬───────────┘
              │ Autor envía versión final
              ▼
       ┌──────────────────┐
       │  FINAL_ENVIADO   │
       └──────┬───────────┘
              │ En programa
              ▼
       ┌──────────────────┐
       │   EN_PROGRAMA    │ (Listo para presentación)
       └──────────────────┘
```

## DIAGRAMA 4: Flujo de Interacción entre Actores

```
AUTOR                ADMIN               REVISOR              COMITÉ
  │                   │                    │                    │
  │ 1. Envía paper    │                    │                    │
  ├──────────────────►│                    │                    │
  │                   │ 2. Registra        │                    │
  │                   │    y valida        │                    │
  │                   │ 3. Distribuye papers              │
  │                   ├───────────────────►│                    │
  │                   │                    │                    │
  │                   │                    │ 4. Evalúa &        │
  │                   │                    │    genera informe   │
  │                   │                    ├──────────────────►│
  │                   │                    │                   │
  │                   │ 5. Compila evaluaciones & decide        │
  │                   │◄──────────────────────────────────────┤
  │                   │                                         │
  │ 6. Notificación   │                                         │
  │◄──────────────────┤                                         │
  │   (resultado)     │                                         │
  │                   │                                         │
  │ 7. Envía cambios  │                                         │
  │ (si aplica)       │                                         │
  ├──────────────────►│                                         │
  │                   │ 8. Integra versión final               │
  │                   │    en programa                         │
  │                   │                                         │
```

## DIAGRAMA 5: Estados Principales de Datos

```
PAPEL - CICLO DE VIDA:
┌─────────┐  ┌──────────┐  ┌─────────┐  ┌──────────┐  ┌───────────┐
│Enviado  │→ │Registrado│→ │En Review│→ │Evaluado  │→ │Seleccionad│
└─────────┘  └──────────┘  └─────────┘  └──────────┘  └───────────┘
                                                               │
                                          No aceptado←────────┘
                                               │
                                          ┌────▼────┐
                                          │Rechazado│
                                          └─────────┘

REVISIÓN - PROGRESO:
┌─────────┐  ┌────────────┐  ┌─────────────┐
│Asignada │→ │En Progreso │→ │Completada   │
└─────────┘  └────────────┘  └─────────────┘

USUARIO - ESTADOS:
┌──────────┐  ┌────────────┐
│Registrado│→ │Activo/Inact│
└──────────┘  └────────────┘
```

## DIAGRAMA 6: Árbol de Decisiones para Aceptación

```
                    ┌─────────────────────────────┐
                    │  3 INFORMES DE REVISORES    │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
       ┌──────────────────────┐    ┌──────────────────────┐
       │  ≥2 Recomendaciones  │    │  ≥2 Recomendaciones  │
       │    ACEPTAR           │    │    RECHAZAR          │
       └──────────┬───────────┘    └──────────┬───────────┘
                  │                           │
                  ▼                           ▼
            ┌───────────┐               ┌───────────┐
            │ ACEPTADO  │               │ RECHAZADO │
            │(con/sin   │               └───────────┘
            │ cambios)  │
            └─────┬─────┘
                  │
        ┌─────────┴─────────┐
        │                   │
        ▼                   ▼
   ┌─────────┐         ┌──────────────┐
   │Sin       │         │Con cambios   │
   │cambios   │         │requeridos    │
   └──────────┘         └──────┬───────┘
                                │
                         Autor realiza
                         cambios y
                         reenvía
                                │
                                ▼
                         ┌──────────────┐
                         │ Agregado al  │
                         │ programa     │
                         └──────────────┘
```

## DIAGRAMA 7: Hitos Temporales del Proceso

```
LÍNEA DE TIEMPO:

┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  T1          T2              T3           T4         T5          │
│  └───────────┴────────────────┴────────────┴──────────┴──────┘  │
│                                                                 │
│  Anuncio    Recepción de     Distribución  Recolección  Decisión│
│  Call for   papers           a revisores   de informes  Final   │
│  Papers                                                         │
│                                                                 │
│            (Plazo abierto)   (Plazo de    (Revisores   (Comité │
│                              revisión)     envían)     selecciona)
│
│  T6                 T7              T8
│  └────────────────────┴──────────────┘
│
│  Notificación a      Plazo para      Programa
│  autores             versión final   Final
│                                      
│  (Autores            (Autores        (Publicado)
│   reciben            reenvían)
│   resultado)
│
└─────────────────────────────────────────────────────────────────┘
```

## DIAGRAMA 8: Responsabilidades por Rol

```
╔════════════════════════════════════════════════════════════════╗
║                    MATRIZ DE RESPONSABILIDADES                 ║
╠═══════════╦═══════════╦═══════════╦═══════════╦════════════════╣
║ ACTIVIDAD ║ADMINISTRA ║  AUTOR    ║ REVISOR   ║   COMITÉ      ║
║           ║DOR        ║           ║           ║                ║
╠═══════════╬═══════════╬═══════════╬═══════════╬════════════════╣
║ Anuncio   ║     ✓     ║           ║           ║                ║
║ Call for  ║           ║           ║           ║                ║
╟───────────╫───────────╫───────────╫───────────╫────────────────╢
║ Envío de  ║           ║     ✓     ║           ║                ║
║ papers    ║           ║           ║           ║                ║
╟───────────╫───────────╫───────────╫───────────╫────────────────╢
║ Registro  ║     ✓     ║           ║           ║                ║
║ de papers ║           ║           ║           ║                ║
╟───────────╫───────────╫───────────╫───────────╫────────────────╢
║ Distribu- ║     ✓     ║           ║           ║                ║
║ ción      ║           ║           ║           ║                ║
╟───────────╫───────────╫───────────╫───────────╫────────────────╢
║ Revisión  ║           ║           ║     ✓     ║                ║
║ de papers ║           ║           ║           ║                ║
╟───────────╫───────────╫───────────╫───────────╫────────────────╢
║ Decisión  ║           ║           ║           ║        ✓       ║
║ final     ║           ║           ║           ║                ║
╟───────────╫───────────╫───────────╫───────────╫────────────────╢
║ Notif.    ║     ✓     ║           ║           ║                ║
║ resultados║           ║           ║           ║                ║
╟───────────╫───────────╫───────────╫───────────╫────────────────╢
║ Cambios y ║           ║     ✓     ║           ║                ║
║ versión   ║           ║           ║           ║                ║
║ final     ║           ║           ║           ║                ║
╟───────────╫───────────╫───────────╫───────────╫────────────────╢
║ Programa  ║     ✓     ║           ║           ║                ║
║ final     ║           ║           ║           ║                ║
╚═══════════╩═══════════╩═══════════╩═══════════╩════════════════╝

✓ = Responsable de realizar
```
