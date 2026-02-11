# gAPI — Arquitectura General

Servicio gráfico (gAPI) para exponer una API gráfica estable a C++, C# y Java y traducir peticiones al motor gráfico que usa DirectX o OpenGL.

```mermaid
graph TB
  subgraph Clients["Librerías Cliente"]
    Cpp["C++ Client"]
    CSharp["C# Client"]
    Java["Java Client"]
  end

  LanguageLayer["Language Bindings\n(C ABI +_wrappers)"]
  gAPI["gAPI (Facade / Public API)"]
  Engine["Rendering Engine (Scene Manager, Renderer)"]

  subgraph Backends["Backends Gráficos"]
    DX["DirectX"]
    GL["OpenGL / Vulkan (opcional)"]
  end

  OS["Sistema Operativo / Drivers / GPU"]

  Clients --> LanguageLayer
  LanguageLayer --> gAPI
  gAPI --> Engine
  Engine --> DX
  Engine --> GL
  DX --> OS
  GL --> OS

  subgraph Tools["Asset & Build Tools"]
    AssetPipeline["Asset Pipeline"]
    ShaderCompiler["Shader Compiler"]
  end
  AssetPipeline --> Engine
  ShaderCompiler --> Engine
```

- Capas: Clientes → Bindings (C ABI) → gAPI (Facade) → Engine → Backend (DirectX/OpenGL) → OS/GPU.
- Principios: API estable (C ABI), mínima dependencia de lenguajes, backend swappable, performance-first.
- Responsabilidad del engine: escena, orden de dibujado, composición de equipamiento, gestión de recursos GPU.
