# Diagramas de Arquitectura - gAPI

Este documento contiene los diagramas arquitectónicos generados con Mermaid para visualizar la estructura y comportamiento de gAPI.

## 1. Diagrama de Contexto (C4 Nivel 1)
Muestra cómo gAPI interactúa con el mundo exterior (Juego y Hardware).

```mermaid
C4Context
    title Diagrama de Contexto del Sistema gAPI

    Person(developer, "Desarrollador de Juegos", "Usa gAPI para crear RPGs")
    
    System_Boundary(gapi_boundary, "Entorno gAPI") {
        System(gAPI, "gAPI Core", "Motor gráfico abstracto. Expone API para C++, C#, Java.")
    }

    System_Ext(GameLogic, "Lógica del Juego", "Código del usuario (C++, C#, Java) que define reglas y estado.")
    System_Ext(OS_DirectX, "Windows / DirectX", "API Gráfica Nativa")
    System_Ext(OS_OpenGL, "SO / OpenGL Driver", "API Gráfica Nativa Multiplataforma")

    Rel(developer, GameLogic, "Escribe y compila")
    Rel(GameLogic, gAPI, "Envía comandos de dibujo", "API Call")
    Rel(gAPI, OS_DirectX, "Renderiza usando", "COM/HAL")
    Rel(gAPI, OS_OpenGL, "Renderiza usando", "GL Calls")
```

## 2. Diagrama de Componentes (Nivel 2)
Detalla los módulos internos de gAPI y su relación implementando el patrón Facade y Bridge.

```mermaid
classDiagram
    direction TB

    %% Client Layer
    class GameClient {
        +LoadScene()
        +Update()
        +Draw()
    }

    %% Facade Layer
    class gAPIEngine {
        <<Facade>>
        -Renderer renderer
        -SceneManager sceneMgr
        +Initialize(backend: Enum)
        +RenderFrame()
    }

    %% Logic Layer
    class SceneManager {
        -List~GameObject~ entities
        -Terrain terrain
        +CullObjects()
        +SortByDepth()
    }

    class GameObject {
        +Mesh mesh
        +Material mat
        +Transform trans
    }

    class Character {
        -List~Equipment~ gear
        +Equip(item)
        +Render(renderer)
    }

    class EquipmentManager {
        +SortLayers()
        +ResolveClipping()
    }

    %% Abstraction Layer (Bridge)
    class ARenderer {
        <<Interface>>
        +Init()
        +Clear()
        +DrawMesh(mesh)
        +ApplyShader(shader)
    }

    %% Implementation Layer (Bridge)
    class DXRenderer {
        +Init()
        +DrawMesh(mesh)
    }
    
    class GLRenderer {
        +Init()
        +DrawMesh(mesh)
    }

    %% Relationships
    GameClient --> gAPIEngine : Uses
    gAPIEngine --> SceneManager : Manages
    gAPIEngine --> ARenderer : Delegate Draw
    SceneManager --> GameObject : Contains mechanism
    GameObject <|-- Character : Inherits
    Character --> EquipmentManager : Uses for layering
    ARenderer <|-- DXRenderer : Implements
    ARenderer <|-- GLRenderer : Implements
```

## 3. Diagrama de Secuencia - Pipeline de Renderizado
Muestra el flujo de ejecución de un frame, desde la llamada del juego hasta la GPU.

```mermaid
sequenceDiagram
    participant Game as Juego (Loop)
    participant API as gAPI Facade
    participant Scene as SceneManager
    participant Char as Character
    participant Render as Renderer (Bridge)
    participant GPU as DirectX/OpenGL

    Note over Game, GPU: Comienza un Frame

    Game->>API: DrawFrame()
    API->>Render: ClearScreen()
    Render->>GPU: glClear / ClearRenderTarget
    
    API->>Scene: ProcessScene()
    
    %% Terrain
    Scene->>Render: Draw(Terrain)
    Render->>GPU: DrawCall(TerrainVBO)

    %% Structures
    loop For Each Structure
        Scene->>Render: Draw(Structure)
    end

    %% Characters & Equipment
    loop For Each Character
        Scene->>Char: PrepareRender()
        Char->>Char: SortEquipmentByLayer()
        Note right of Char: Orden: Cuerpo -> Ropa -> Armadura -> Capa
        
        loop For Each EquipmentItem
            Char->>Render: Draw(ItemMesh, LayerBias)
            Render->>GPU: SetDepthBias & DrawCall
        end
    end

    %% Post-Effects
    API->>Render: ApplyPostProcessing()
    Render->>GPU: DrawFullScreenQuad(Shaders)

    API->>Render: SwapBuffers()
    Render-->>Game: Frame Complete
```

## 4. Diagrama de Composición de Equipamiento
Detalla la lógica específica para evitar el clipping mediante capas.

```mermaid
graph TD
    subgraph Character Composition
        C[Character Entity]
        
        subgraph Equipment Layers
            L0[Base Body / Skin]
            L1[Layer 1: Undergarments]
            L2[Layer 2: Armor / Tunic / Clothes]
            L3[Layer 3: Belt / Boots / Gloves]
            L4[Layer 4: Outerwear (Cloaks, Backpacks)]
        end
        
        C -->|Has| L0
        C -->|Has| L1
        C -->|Has| L2
        C -->|Has| L3
        C -->|Has| L4
    end

    subgraph Rendering Logic
        D[Draw Call]
        Z[Z-Buffer Check]
        B[Depth Bias / Stencil]
    end

    L0 -->|Draw First| D
    L1 -->|Draw Second| D
    L2 -->|Draw Third| D
    L3 -->|Draw Fourth| D
    L4 -->|Draw Last| D

    D --> Z
    D -.->|Prevent Z-Fighting| B
    B -->|Offset Z| Z

    style L0 fill:#f9f,stroke:#333
    style L1 fill:#ccf,stroke:#333
    style L2 fill:#99f,stroke:#333
    style L3 fill:#66f,stroke:#333
    style L4 fill:#33f,stroke:#333
```
