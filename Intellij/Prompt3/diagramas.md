# Diagramas de Arquitectura - Sistema gAPI (Motor Gráfico)

## 1. Diagrama de Arquitectura General del Sistema

```mermaid
flowchart TB
    subgraph Clients["Aplicaciones Cliente"]
        CppGame["Juego C++"]
        CsGame["Juego C# (.NET)"]
        JavaGame["Juego Java"]
        PythonScript["Script Python"]
    end

    subgraph LanguageBindings["Language Bindings"]
        CppAPI["C++ API\n(Native)"]
        CsAPI["C# API\n(P/Invoke + C++/CLI)"]
        JavaAPI["Java API\n(JNI Wrappers)"]
        SWIGGen["SWIG Generated\nBindings"]
    end

    subgraph gAPICore["gAPI Core Engine"]
        subgraph SceneManagement["Gestión de Escena"]
            SceneMgr["Scene Manager"]
            EntityMgr["Entity Manager"]
            CameraMgr["Camera Manager"]
        end
        
        subgraph CharacterSys["Sistema de Personajes"]
            CharSys["Character System"]
            EquipSys["Equipment System\n(Layer Management)"]
            AnimSys["Animation System"]
        end
        
        subgraph RenderPipeline["Pipeline de Renderizado"]
            Pipeline["Render Pipeline"]
            RenderPasses["Render Passes\nManager"]
            BatchSys["Batching System"]
        end
        
        subgraph ResourceMgmt["Gestión de Recursos"]
            ResMgr["Resource Manager"]
            MemMgr["Memory Manager"]
            Streaming["Asset Streaming"]
        end
        
        subgraph EffectsSys["Sistema de Efectos"]
            PostFX["Post Effects\nManager"]
            LightSys["Lighting System"]
            ShadowSys["Shadow Mapping"]
        end
    end

    subgraph GraphicsBackends["Graphics Backends"]
        subgraph DXBackend["DirectX Backend"]
            DX12["DirectX 12"]
            DX11["DirectX 11"]
        end
        
        subgraph VulkanBackend["Vulkan Backend"]
            Vulkan["Vulkan 1.3"]
        end
        
        subgraph OpenGLBackend["OpenGL Backend"]
            GL46["OpenGL 4.6"]
            GLES3["OpenGL ES 3.0"]
        end
        
        subgraph MetalBackend["Metal Backend"]
            Metal["Metal 2/3"]
        end
    end

    %% Conexiones
    CppGame --> CppAPI
    CsGame --> CsAPI
    JavaGame --> JavaAPI
    PythonScript --> SWIGGen
    
    CppAPI --> gAPICore
    CsAPI --> gAPICore
    JavaAPI --> gAPICore
    SWIGGen --> gAPICore
    
    gAPICore --> GraphicsBackends
    
    GraphicsBackends --> DX12
    GraphicsBackends --> Vulkan
    GraphicsBackends --> GL46
    GraphicsBackends --> Metal
```

---

## 2. Diagrama de Arquitectura de Capas

```mermaid
flowchart TB
    subgraph Application["Application Layer"]
        GameCode["Game Application Code\n(C++, C#, Java)"]
        GameLoop["Game Loop"]
        Input["Input Handling"]
    end

    subgraph gAPIAbstract["gAPI Abstract Layer"]
        LanguageBindings["Language Bindings\n(C++, C#, Java, Python)"]
        PublicAPI["Public API Interfaces"]
        SceneAPI["Scene API"]
        CharacterAPI["Character API"]
        RenderAPI["Render API"]
    end

    subgraph EngineCore["Engine Core Layer"]
        subgraph Systems["Core Systems"]
            SceneMgr["Scene Manager"]
            EntityMgr["Entity Manager"]
            ResourceMgr["Resource Manager"]
            MemoryMgr["Memory Manager"]
        end
        
        subgraph CharacterCore["Character Core"]
            CharFactory["Character Factory"]
            EquipFactory["Equipment Factory"]
            AnimController["Animation Controller"]
            LayerManager["Layer Ordering Manager"]
        end
        
        subgraph RenderCore["Render Core"]
            RenderQueue["Render Command Queue"]
            PassManager["Pass Manager"]
            BatchOptimizer["Batch Optimizer"]
            ViewFrustum["View Frustum Culling"]
        end
    end

    subgraph BackendAbstraction["Backend Abstraction Layer"]
        BackendIF["IRenderer Interface"]
        SwapChain["Swap Chain"]
        DeviceContext["Device Context"]
        CommandBuffer["Command Buffer"]
        
        subgraph BackendImpl["Backend Implementations"]
            DXImpl["DirectX Implementation"]
            VulkanImpl["Vulkan Implementation"]
            GLImpl["OpenGL Implementation"]
            MetalImpl["Metal Implementation"]
        end
    end

    subgraph OSGraphics["OS & Graphics APIs"]
        subgraph OS["Operating Systems"]
            Windows["Windows\n(DirectX/Vulkan)"]
            Linux["Linux\n(Vulkan/OpenGL)"]
            MacOS["macOS\n(Metal/OpenGL)"]
            IOS["iOS\n(Metal)"]
            Android["Android\n(Vulkan/OpenGL ES)"]
        end
    end

    %% Flujo de datos
    GameCode --> GameLoop
    GameLoop --> Input
    Input --> LanguageBindings
    
    LanguageBindings --> PublicAPI
    PublicAPI --> SceneAPI
    PublicAPI --> CharacterAPI
    PublicAPI --> RenderAPI
    
    SceneAPI --> SceneMgr
    CharacterAPI --> CharFactory
    RenderAPI --> RenderQueue
    
    SceneMgr --> EntityMgr
    SceneMgr --> ResourceMgr
    CharFactory --> EquipFactory
    CharFactory --> AnimController
    AnimController --> LayerManager
    
    ResourceMgr --> MemoryMgr
    RenderQueue --> PassManager
    PassManager --> BatchOptimizer
    
    BatchOptimizer --> BackendIF
    BackendIF --> CommandBuffer
    
    CommandBuffer --> DXImpl
    CommandBuffer --> VulkanImpl
    CommandBuffer --> GLImpl
    CommandBuffer --> MetalImpl
    
    DXImpl --> Windows
    VulkanImpl --> Linux
    MetalImpl --> MacOS
```

---

## 3. Diagrama de Componentes del Motor

```mermaid
flowchart TB
    subgraph gAPI["gAPI Engine Components"]
        subgraph SceneSystem["Scene System"]
            Scene["Scene"]
            Entities["Entities"]
            Transforms["Transforms"]
            Cameras["Cameras"]
            Lights["Lights"]
            Occlusion["Occlusion Culler"]
        end
        
        subgraph CharacterSystem["Character System"]
            Characters["Characters"]
            Skeletons["Skeletons"]
            Animations["Animations"]
            EquipmentSlots["Equipment Slots"]
            LayerOrder["Layer Order Manager"]
        end
        
        subgraph EquipmentSystem["Equipment System"]
            EquipmentItems["Equipment Items"]
            Attachments["Attachment Points"]
            LayerSorting["Layer Z-Sorting"]
            ClipPrevention["Clipping Prevention"]
            Instancing["GPU Instancing"]
        end
        
        subgraph RenderSystem["Render System"]
            subgraph Passes["Render Passes"]
                GeometryPass["Geometry Pass"]
                ShadowPass["Shadow Pass"]
                LightingPass["Lighting Pass"]
                PostFXPass["Post Effects Pass"]
                CompositePass["Composite Pass"]
            end
            
            subgraph Optimizations["Optimizations"]
                Batching["Dynamic Batching"]
                LOD["LOD Selection"]
                Culling["Frustum Culling"]
                Occlusion["Occlusion Culling"]
            end
            
            subgraph Resources["Graphics Resources"]
                Meshes["Meshes"]
                Materials["Materials"]
                Textures["Textures"]
                Shaders["Shaders"]
            end
        end
    end

    %% Conexiones
    SceneSystem --> CharacterSystem
    CharacterSystem --> EquipmentSystem
    EquipmentSystem --> RenderSystem
    
    RenderSystem --> Passes
    Passes --> Optimizations
    Optimizations --> Resources
```

---

## 4. Diagrama de Pipeline de Renderizado

```mermaid
flowchart TB
    subgraph Application["Game Application"]
        GameLoop["Game Loop"]
        Update["Update State"]
    end

    subgraph Preparation["Scene Preparation"]
        UpdateTrans["Update Transforms"]
        UpdateAnim["Update Animations"]
        UpdateEquip["Update Equipment"]
        FrustumCull["Frustum Culling"]
        LODSelect["LOD Selection"]
        SortLayers["Sort by Layer\n(Equipment)"]
        BuildBatches["Build Render Batches"]
    end

    subgraph GeometryPass["Geometry Pass"]
        subgraph TerrainLayer["Layer 0: Terrain"]
            TerrainMesh["Terrain Mesh"]
            TerrainMat["Terrain Material"]
        end
        
        subgraph StructureLayer["Layer 1: Structures"]
            Buildings["Buildings"]
            Props["Props"]
            Occluders["Occluders"]
        end
        
        subgraph CharacterLayer["Layer 2: Characters"]
            CharacterBody["Character Body"]
            Skeleton["Skeleton"]
        end
        
        subgraph EquipmentLayer["Layer 3: Equipment"]
            LayerOrder3["Render by Z-Order"]
            InstancedDraw["GPU Instancing"]
        end
    end

    subgraph ShadowPass["Shadow Pass"]
        ShadowMaps["Shadow Maps\n(Directional + Point)"]
        PCF["PCF Filtering"]
        VSM["Variance Shadow Maps"]
    end

    subgraph LightingPass["Lighting Pass"]
        GBuffer["G-Buffer\n(Albedo, Normal, Depth)"]
        LightPrep["Light Prepass"]
        DeferredLighting["Deferred Lighting\n(Forward/Deferred)"]
        SSAO["SSAO"]
    end

    subgraph PostEffectsPass["Post Effects Pass"]
        Bloom["Bloom"]
        MotionBlur["Motion Blur"]
        DOF["Depth of Field"]
        ColorGrade["Color Grading"]
        ToneMap["Tone Mapping"]
    end

    subgraph Output["Output"]
        FrameBuffer["Frame Buffer"]
        Present["Present to Screen"]
        HDR["HDR to SDR"]
    end

    %% Flujo
    GameLoop --> Update
    Update --> UpdateTrans
    UpdateTrans --> UpdateAnim
    UpdateAnim --> UpdateEquip
    UpdateEquip --> FrustumCull
    FrustumCull --> LODSelect
    LODSelect --> SortLayers
    SortLayers --> BuildBatches
    
    BuildBatches --> GeometryPass
    GeometryPass --> ShadowPass
    ShadowPass --> LightingPass
    LightingPass --> PostEffectsPass
    PostEffectsPass --> Output
```

---

## 5. Diagrama de Sistema de Equipamiento (Capas)

```mermaid
flowchart TB
    subgraph Character["Character Entity"]
        Body["Base Body Mesh"]
    end

    subgraph EquipmentLayers["Equipment Rendering Order"]
        subgraph Layer0["Layer 0 - Inside"]
            Underwear["Underwear/Skin"]
            Socks["Socks"]
        end
        
        subgraph Layer1["Layer 1 - Main"]
            Boots["Boots/Footwear"]
            Trousers["Trousers"]
            Belt["Belt Equipment"]
            Gloves["Gloves"]
            Tunic["Tunic/Shirt"]
            Armor["Armor"]
            Robe["Robe"]
        end
        
        subgraph Layer2["Layer 2 - Accessories"]
            BackEquip["Back Equipment"]
            Helmet["Helmet/Hood"]
            CapeBack["Cape (Back)"]
        end
        
        subgraph Layer3["Layer 3 - Weapons"]
            MainHand["Main Hand Weapon"]
            OffHand["Off Hand/Shield"]
        end
        
        subgraph Layer4["Layer 4 - Front"]
            CapeFront["Cape (Front)"]
        end
    end

    subgraph ZOrdering["Z-Order Calculation"]
        CalcDepth["Calculate Z-Depth"]
        SortItems["Sort Items"]
        GenerateKeys["Generate Sort Keys"]
    end

    subgraph Prevention["Clipping Prevention"]
        ZBuffer["Z-Buffer Test"]
        StencilMask["Stencil Mask"]
        BlendOrder["Blend Order"]
    end

    subgraph GPU["GPU Optimization"]
        Instancing["GPU Instancing"]
        DynamicBuffers["Dynamic Buffers"]
        Culling["Pre-Cull Hidden"]
    end

    %% Conexiones
    Body --> Layer0
    Layer0 --> Layer1
    Layer1 --> Layer2
    Layer2 --> Layer3
    Layer3 --> Layer4
    
    Layer0 --> ZOrdering
    Layer1 --> ZOrdering
    Layer2 --> ZOrdering
    Layer3 --> ZOrdering
    Layer4 --> ZOrdering
    
    ZOrdering --> Prevention
    
    Prevention --> GPU
```

---

## 6. Diagrama de Clases Principal

```mermaid
classDiagram
    %% Interfaces principales
    class IGAPI {
        <<interface>>
        +createScene() IScene
        +getRenderer() IRenderer
        +getCharacterSystem() ICharacterSystem
        +initialize() bool
        +shutdown()
    }
    
    class IScene {
        <<interface>>
        +addEntity(IEntity)
        +removeEntity(IEntity)
        +update(float)
        +setEnvironment(EnvironmentParams)
        +getCamera() ICamera
    }
    
    class IRenderer {
        <<interface>>
        +beginFrame()
        +renderScene(IScene)
        +endFrame()
        +setViewport(x, y, w, h)
        +togglePostEffect(PostEffectType, bool)
    }
    
    class ICharacterSystem {
        <<interface>>
        +createCharacter(CharacterDesc) ICharacter
        +equipItem(ICharacter, EquipmentSlot, IEquipmentItem)
        +unequipItem(ICharacter, EquipmentSlot)
        +updateCharacters(float)
    }
    
    %% Clases concretas
    class GAPIImpl {
        -renderer: IRendererImpl
        -sceneManager: SceneManager
        -characterSystem: CharacterSystem
        +initialize() bool
        +createScene() Scene
        +getRenderer() IRendererImpl
    }
    
    class Scene {
        -entities: List~IEntity~
        -camera: Camera
        -environment: Environment
        +addEntity(IEntity)
        +update(float)
    }
    
    class RendererImpl {
        -device: GraphicsDevice
        -commandQueue: CommandQueue
        -renderPasses: List~RenderPass~
        -frameBuffer: FrameBuffer
        +beginFrame()
        +renderScene(Scene)
        +endFrame()
    }
    
    class CharacterSystemImpl {
        -characters: List~Character~
        -equipmentManager: EquipmentManager
        +createCharacter(CharacterDesc) Character
        +equipItem(Character, EquipmentSlot, IEquipmentItem)
        +updateCharacters(float)
    }
    
    class Character {
        -skeleton: Skeleton
        -animations: AnimationController
        -equipment: EquipmentSlots
        -transform: Transform
        +update(float)
        +playAnimation(string)
    }
    
    class EquipmentManager {
        -equipmentItems: Map~EquipmentSlot, IEquipmentItem~
        -layerOrder: LayerOrderConfig
        +equipItem(EquipmentSlot, IEquipmentItem)
        +getRenderOrder() List~RenderItem~
    }
    
    %% Clases de equipamiento
    class EquipmentItem {
        <<interface>>
        +getSlot() EquipmentSlot
        +getLayer() uint8
        +getMesh() IMesh
        +getMaterial() IMaterial
    }
    
    class ArmorItem {
        -mesh: Mesh
        -material: Material
        -layer: uint8
        -slot: EquipmentSlot
    }
    
    class WeaponItem {
        -mesh: Mesh
        -material: Material
        -attachmentPoint: AttachmentPoint
        -layer: uint8
    }
    
    %% Backends
    class IRendererBackend {
        <<interface>>
        +initialize(params) bool
        +createMesh(MeshData) IMesh
        +createTexture(TextureData) ITexture
        +createShader(ShaderData) IShader
        +drawIndexed(indexCount, instanceCount)
    }
    
    class DirectX12Backend {
        -device: ID3D12Device
        -commandQueue: ID3D12CommandQueue
        -swapChain: IDXGISwapChain4
        +initialize(params) bool
        +drawIndexed(indexCount, instanceCount)
    }
    
    class VulkanBackend {
        -device: VkDevice
        -physicalDevice: VkPhysicalDevice
        -commandBuffers: VkCommandBuffer
        +initialize(params) bool
        +drawIndexed(indexCount, instanceCount)
    }
    
    %% Relaciones
    IGAPI <|.. GAPIImpl
    IScene <|.. Scene
    IRenderer <|.. RendererImpl
    ICharacterSystem <|.. CharacterSystemImpl
    
    GAPIImpl --> SceneManager
    GAPIImpl --> RendererImpl
    GAPIImpl --> CharacterSystemImpl
    
    SceneManager --> Scene
    RendererImpl --> IRendererBackend
    CharacterSystemImpl --> Character
    Character --> EquipmentManager
    Character --> Skeleton
    
    EquipmentManager --> EquipmentItem
    EquipmentItem <|.. ArmorItem
    EquipmentItem <|.. WeaponItem
    
    IRendererBackend <|.. DirectX12Backend
    IRendererBackend <|.. VulkanBackend
    IRendererBackend <|.. OpenGLBackend
    IRendererBackend <|.. MetalBackend
```

---

## 7. Diagrama de Estados de Renderizado

```mermaid
stateDiagram-v2
    [*] --> Initializing: Engine Start
    
    Initializing --> Ready: Initialization Complete
    
    Ready --> FrameBegin: Start Frame
    
    FrameBegin --> Update: Game Update
    Update --> PrepareScene: Prepare Scene
    PrepareScene --> Culling: Frustum Culling
    Culling --> LODSelect: LOD Selection
    LODSelect --> SortLayers: Layer Sorting
    SortLayers --> BuildBatches: Build Batches
    BuildBatches --> GeometryPass: Start Render Pass
    
    GeometryPass --> Terrain: Draw Terrain
    Terrain --> Structures: Draw Structures
    Structures --> Characters: Draw Characters
    Characters --> Equipment: Draw Equipment
    Equipment --> PostEffects: Post Effects Pass
    
    PostEffects --> FrameEnd: End Frame
    FrameEnd --> Present: Present to Screen
    Present --> Ready: Next Frame
    
    Ready --> Shutdown: Engine Shutdown
    Shutdown --> [*]
    
    note right of Equipment
        Equipment rendered in Z-order:
        Layer 0: Underwear
        Layer 1: Base Clothing
        Layer 2: Armor
        Layer 3: Boots
        Layer 4: Gloves
        Layer 5: Trousers
        Layer 6: Belt
        Layer 7: Tunic
        Layer 8: Cape Back
        Layer 9: Back Equipment
        Layer 10: Helmet
        Layer 11: Cape Front
        Layer 12: Weapons
    end note
```

---

## 8. Diagrama de Despliegue

```mermaid
flowchart TB
    subgraph Developer["Entorno de Desarrollo"]
        IDE["IDE (VS/CLion)"]
        BuildSys["CMake Build System"]
        Debugger["Debugger (RenderDoc)"]
        Profiler["Profiler"]
    end

    subgraph Build["Pipeline de Build"]
        Compiler["C++ Compiler\n(MSVC/Clang/GCC)"]
        Linker["Linker"]
        SWIG["SWIG\n(Binding Generation)"]
        Tests["Unit Tests\n(Google Test)"]
    end

    subgraph Runtime["Runtime Environment"]
        subgraph Windows["Windows Platform"]
            DX12Runtime["DirectX 12 Runtime"]
            VulkanSDK["Vulkan SDK"]
            WinLibs["gAPI DLLs"]
        end
        
        subgraph Linux["Linux Platform"]
            VulkanLinux["Vulkan Runtime"]
            OpenGLLibs["OpenGL Libraries"]
            LinuxLibs["gAPI Shared Libs"]
        end
        
        subgraph MacOS["macOS Platform"]
            MetalSDK["Metal SDK"]
            OpenGLMac["OpenGL Framework"]
            MacLibs["gAPI Frameworks"]
        end
    end

    subgraph Target["Target Hardware"]
        subgraph GPUs["GPUs"]
            NVIDIAGPU["NVIDIA GPUs"]
            AMDGPU["AMD GPUs"]
            IntelGPU["Intel Integrated"]
            AppleSilicon["Apple Silicon"]
        end
        
        subgraph Consoles["Consoles"]
            Xbox["Xbox Series"]
            PlayStation["PlayStation"]
            Switch["Nintendo Switch"]
        end
    end

    %% Flujo
    IDE --> Compiler
    IDE --> Debugger
    IDE --> Profiler
    
    Compiler --> Linker
    Linker --> WinLibs
    Linker --> LinuxLibs
    Linker --> MacLibs
    
    SWIG --> CsAPI["C# Bindings"]
    SWIG --> JavaAPI["Java Bindings"]
    
    WinLibs --> DX12Runtime
    WinLibs --> VulkanSDK
    
    LinuxLibs --> VulkanLinux
    LinuxLibs --> OpenGLLibs
    
    MacLibs --> MetalSDK
    MacLibs --> OpenGLMac
    
    DX12Runtime --> NVIDIAGPU
    DX12Runtime --> AMDGPU
    
    VulkanSDK --> NVIDIAGPU
    VulkanSDK --> AMDGPU
    VulkanSDK --> IntelGPU
    
    MetalSDK --> AppleSilicon
```

---

## 9. Diagrama de Arquitectura de Memoria

```mermaid
flowchart TB
    subgraph SystemMemory["System Memory (RAM)"]
        subgraph EngineMem["Engine Memory"]
            Code["Code + Static Data"]
            Heap["Dynamic Heap"]
            Stack["Call Stack"]
        end
        
        subgraph AssetMem["Asset Storage"]
            Meshes["Mesh Data"]
            Textures["Texture Data"]
            Shaders["Shader Code"]
            Audio["Audio Data"]
        end
        
        subgraph StreamMem["Streaming Buffer"]
            Queue["Asset Load Queue"]
            Cache["LRU Cache"]
            Pool["Memory Pool"]
        end
    end

    subgraph VideoMemory["Video Memory (VRAM)"]
        subgraph GPUResources["GPU Resources"]
            VB["Vertex Buffers"]
            IB["Index Buffers"]
            Tex["Textures\n(Compressed)"]
            UB["Uniform Buffers"]
            SB["Shader Storage"]
        end
        
        subgraph RenderTargets["Render Targets"]
            GBuffer["G-Buffer\n(Albedo + Normal + Depth)"]
            ShadowMaps["Shadow Maps"]
            HDRBuffer["HDR Framebuffer"]
            PostFXBuffers["Post-FX Buffers"]
        end
        
        subgraph GPUCache["GPU Cache"]
            ConstantPool["Constant Pool"]
            DescriptorHeap["Descriptor Heap"]
            CommandQueue["Command Queue"]
        end
    end

    %% Transfer paths
    SystemMemory -->|PCIe Transfer| VideoMemory
    StreamMem -->|Async Load| GPUResources
```

---

## 10. Diagrama de sequence - Renderizado de Frame

```mermaid
sequenceDiagram
    participant Game as Game Application
    participant gAPI as gAPI Engine
    participant Scene as Scene Manager
    participant Character as Character System
    participant Equipment as Equipment System
    participant Renderer as Renderer
    participant Backend as Graphics Backend
    
    Game->>gAPI: Update(deltaTime)
    
    gAPI->>Scene: update(deltaTime)
    Scene->>Scene: updateEntities()
    Scene->>Character: updateCharacters()
    
    Character->>Equipment: updateEquipment()
    Equipment->>Equipment: recalculateLayers()
    Equipment->>Equipment: sortByZOrder()
    Equipment->>Equipment: generateRenderItems()
    
    Scene->>Renderer: beginFrame()
    
    rect rgb(200, 200, 255)
        Note over Renderer: Geometry Pass
        Renderer->>Backend: setRenderTarget(G-Buffer)
        Renderer->>Backend: clearDepthStencil()
        
        Renderer->>Backend: drawTerrain()
        Backend->>GPU: Render terrain mesh
        
        Renderer->>Backend: drawStructures()
        Backend->>GPU: Render building meshes
        
        Renderer->>Backend: drawCharacters()
        Backend->>GPU: Render character bodies
        
        Renderer->>Backend: drawEquipment(LayerOrder)
        loop For each equipment layer
            Backend->>GPU: Draw equipment with Z-sorting
        end
    end
    
    rect rgb(200, 255, 200)
        Note over Renderer: Shadow Pass
        Renderer->>Backend: renderShadowMaps()
        Backend->>GPU: Generate shadow maps
    end
    
    rect rgb(255, 200, 200)
        Note over Renderer: Lighting Pass
        Renderer->>Backend: renderLighting()
        Backend->>GPU: Apply lighting with SSAO
    end
    
    rect rgb(255, 255, 200)
        Note over Renderer: Post Effects Pass
        Renderer->>Backend: applyBloom()
        Renderer->>Backend: applyMotionBlur()
        Renderer->>Backend: applyDOF()
        Renderer->>Backend: applyColorGrading()
        Renderer->>Backend: toneMapping()
    end
    
    Renderer->>Backend: presentToScreen()
    Backend->>GPU: Flip back buffer
    
    Renderer->>Game: frameComplete()
```

---

## 11. Diagrama de Arquitectura de Shaders

```mermaid
flowchart TB
    subgraph ShaderTypes["Shader Types"]
        VS["Vertex Shader"]
        HS["Hull Shader (Tessellation)"]
        DS["Domain Shader (Tessellation)"]
        GS["Geometry Shader"]
        PS["Pixel Shader"]
        CS["Compute Shader"]
    end

    subgraph RenderingTechniques["Rendering Techniques"]
        subgraph TerrainShaders["Terrain Shaders"]
            TerrainVS["Terrain Vertex Shader"]
            TerrainPS["Terrain Pixel Shader\n(Texture Splatting)"]
        end
        
        subgraph CharacterShaders["Character Shaders"]
            CharVS["Character Vertex Shader\n(Skinning)"]
            CharPS["Character Pixel Shader\n(PBR Lighting)"]
        end
        
        subgraph EquipmentShaders["Equipment Shaders"]
            EquipVS["Equipment Vertex Shader"]
            EquipPS["Equipment Pixel Shader\n(PBR + Layer Sorting)"]
        end
        
        subgraph EffectShaders["Effect Shaders"]
            ShadowVS["Shadow Vertex Shader"]
            ShadowPS["Shadow Pixel Shader"]
            BloomPS["Bloom Pixel Shader"]
            SSAOPS["SSAO Pixel Shader"]
            DOFPS["DOF Pixel Shader"]
        end
    end

    subgraph ShaderSystems["Shader Systems"]
        PBR["PBR Lighting System\n(Metallic-Roughness)"]
        Skinning["Skinning System\n(Vertex Blending)"]
        Instancing["Instancing System\n(GPU Batching)"]
    end

    %% Conexiones
    VS --> TerrainVS
    VS --> CharVS
    VS --> EquipVS
    VS --> ShadowVS
    
    PS --> TerrainPS
    PS --> CharPS
    PS --> EquipPS
    PS --> BloomPS
    PS --> SSAOPS
    PS --> DOFPS
    
    PBR --> CharPS
    PBR --> EquipPS
    PBR --> TerrainPS
    
    Skinning --> CharVS
    Skinning --> EquipVS
    
    Instancing --> EquipVS
    Instancing --> TerrainVS
```
