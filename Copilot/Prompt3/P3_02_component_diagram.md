# Diagrama de Componentes — gAPI

```mermaid
graph TB
  subgraph Frontends["Clientes / Herramientas"]
    Cpp["C++ / Game Code"]
    CSharp["C# / Scripting"]
    Java["Java / Tools"]
    Editor["Editor de Niveles (Tools)"]
  end

  subgraph API["Language Bindings & gAPI"]
    CABI["C ABI (functions exportadas)"]
    Wrappers["Wrappers C#/Java (PInvoke/JNI)"]
    Facade["gAPI Facade"]
  end

  subgraph Engine["Motor Gráfico"]
    Scene["Scene Graph / ECS"]
    Resource["Resource Manager (meshes, textures, shaders)"]
    MeshRenderer["Mesh Renderer"]
    TerrainRenderer["Terrain Renderer"]
    StructureRenderer["Structure Renderer"]
    CharacterRenderer["Character Renderer"]
    EquipmentManager["Equipment Manager / Layerer"]
    Animation["Animation & Skeleton"]
    Culling["Culling & LOD Manager"]
    Postfx["PostProcessing (shadows, lighting)"]
    RenderQueue["Render Queue / Sorter"]
  end

  subgraph Backend["Renderer Backend"]
    DX["DirectX Adapter"]
    GL["OpenGL Adapter"]
  end

  subgraph Infra["Infra / OS"]
    GPU["GPU / Drivers"]
    OS["Sistema Operativo"]
  end

  Cpp --> CABI
  CSharp --> Wrappers
  Java --> Wrappers
  Wrappers --> Facade
  Facade --> Scene
  Facade --> Resource

  Scene --> Culling
  Scene --> RenderQueue
  Resource --> MeshRenderer
  Resource --> CharacterRenderer
  Resource --> TerrainRenderer
  Resource --> StructureRenderer

  CharacterRenderer --> EquipmentManager
  CharacterRenderer --> Animation
  MeshRenderer --> Backend
  TerrainRenderer --> Backend
  StructureRenderer --> Backend
  Postfx --> Backend

  Backend --> GPU
  GPU --> OS
```

- `EquipmentManager` monta los objetos del personaje en orden correcto antes de encolar draw calls.
- `Render Queue` gestiona ordering y batching por materiales/shaders para minimizar cambios de estado.
- `Resource Manager` provee streaming y pooling para meshes/textures/shaders.
