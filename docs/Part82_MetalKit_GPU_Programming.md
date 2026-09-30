# Part 82: MetalKit และ GPU Programming - การเขียนโปรแกรม GPU ด้วย Metal

## บทนำ

Metal เป็น low-level graphics API ของ Apple ที่ให้การเข้าถึง GPU โดยตรง ด้วย overhead ที่ต่ำมาก Metal ถูกออกแบบมาเพื่อการ render กราฟิก 3D, การประมวลผลภาพ, และการคำนวณ general purpose บน GPU (GPGPU) บทนี้จะพาคุณไปเรียนรู้ Metal ตั้งแต่พื้นฐานจนถึงการสร้าง Particle System แบบ interactive

---

## 1. Metal Overview และ GPU Architecture พื้นฐาน

### 1.1 ทำไมต้องใช้ Metal?

```
CPU vs GPU Architecture:

CPU:
┌────────────────────────────────────┐
│  Core 1  │  Core 2  │  Core 3  │  Core 4  │
│  (Complex │  (Complex │  (Complex │  (Complex │
│   Logic)  │   Logic)  │   Logic)  │   Logic)  │
└────────────────────────────────────┘
เหมาะกับงาน Sequential ที่ซับซ้อน

GPU:
┌─────────────────────────────────────────────┐
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ ░░░░░ Thousands of Simple Cores ░░░░░░░░░░░ │
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
└─────────────────────────────────────────────┘
เหมาะกับงาน Parallel ที่ทำซ้ำๆ กัน
```

Metal ทำงานโดยตรงกับ GPU hardware ทำให้:
- **Latency ต่ำ**: ไม่มี overhead จาก OpenGL state machine
- **Parallel Processing**: ใช้ GPU cores หลายพันตัวพร้อมกัน
- **Unified Memory Architecture**: CPU และ GPU share memory เดียวกันบน Apple Silicon
- **Metal Shading Language (MSL)**: ภาษาที่คล้าย C++ สำหรับ GPU programming

### 1.2 Metal Architecture Overview

```
App
 │
 ├── MTLDevice (GPU)
 │    ├── MTLCommandQueue
 │    │    └── MTLCommandBuffer
 │    │         ├── MTLRenderCommandEncoder  ← Render 3D graphics
 │    │         ├── MTLComputeCommandEncoder ← GPGPU computation
 │    │         └── MTLBlitCommandEncoder    ← Memory copy
 │    ├── MTLBuffer (Vertex data, uniforms)
 │    ├── MTLTexture (Images)
 │    └── MTLLibrary → MTLFunction → MTLPipelineState
 │
 └── CAMetalLayer / MTKView (Display)
```

---

## 2. Setting Up Metal

### 2.1 MTLDevice, MTLCommandQueue, MTLCommandBuffer

```swift
// MetalSetup/MetalRenderer.swift

import Metal
import MetalKit
import simd

// MARK: - Metal Device Setup
class MetalRenderer: NSObject {
    
    // MARK: - Metal Core Objects
    let device: MTLDevice
    let commandQueue: MTLCommandQueue
    var library: MTLLibrary
    
    // MARK: - Initialization
    init?(view: MTKView) {
        // 1. สร้าง MTLDevice - ตัวแทนของ GPU
        guard let device = MTLCreateSystemDefaultDevice() else {
            print("❌ ไม่พบ Metal-compatible GPU")
            return nil
        }
        self.device = device
        
        // 2. สร้าง Command Queue - เก็บ sequence ของ GPU commands
        guard let commandQueue = device.makeCommandQueue() else {
            print("❌ ไม่สามารถสร้าง Command Queue")
            return nil
        }
        self.commandQueue = commandQueue
        
        // 3. โหลด Metal Library (compiled shaders)
        guard let library = device.makeDefaultLibrary() else {
            print("❌ ไม่พบ Metal shader library")
            return nil
        }
        self.library = library
        
        super.init()
        
        // 4. ตั้งค่า MTKView
        view.device = device
        view.delegate = self
        view.clearColor = MTLClearColor(red: 0.1, green: 0.1, blue: 0.15, alpha: 1.0)
        view.colorPixelFormat = .bgra8Unorm
        view.depthStencilPixelFormat = .depth32Float
        
        print("✅ Metal initialized: \(device.name)")
    }
    
    // MARK: - Create Command Buffer (called every frame)
    func createCommandBuffer() -> MTLCommandBuffer? {
        return commandQueue.makeCommandBuffer()
    }
}

// MARK: - MTKViewDelegate
extension MetalRenderer: MTKViewDelegate {
    func mtkView(_ view: MTKView, drawableSizeWillChange size: CGSize) {
        // Handle resize
    }
    
    func draw(in view: MTKView) {
        guard let commandBuffer = createCommandBuffer(),
              let drawable = view.currentDrawable,
              let renderPassDescriptor = view.currentRenderPassDescriptor else {
            return
        }
        
        // สร้าง render command encoder
        guard let encoder = commandBuffer.makeRenderCommandEncoder(descriptor: renderPassDescriptor) else {
            return
        }
        
        // ... render commands go here ...
        
        encoder.endEncoding()
        commandBuffer.present(drawable)
        commandBuffer.commit()
    }
}
```

### 2.2 CAMetalLayer Setup

```swift
// MetalSetup/CAMetalLayerSetup.swift

import QuartzCore
import Metal
import UIKit

// MARK: - Custom Metal View using CAMetalLayer (without MTKView)
class MetalView: UIView {
    
    // Override layerClass ให้ใช้ CAMetalLayer
    override class var layerClass: AnyClass {
        return CAMetalLayer.self
    }
    
    var metalLayer: CAMetalLayer {
        return layer as! CAMetalLayer
    }
    
    private var device: MTLDevice?
    private var displayLink: CADisplayLink?
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupMetal()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setupMetal()
    }
    
    private func setupMetal() {
        guard let device = MTLCreateSystemDefaultDevice() else { return }
        self.device = device
        
        // ตั้งค่า CAMetalLayer
        metalLayer.device = device
        metalLayer.pixelFormat = .bgra8Unorm
        metalLayer.framebufferOnly = true  // Optimize for display only
        metalLayer.frame = layer.frame
        
        // สร้าง Display Link สำหรับ render loop
        displayLink = CADisplayLink(target: self, selector: #selector(render))
        displayLink?.preferredFrameRateRange = CAFrameRateRange(minimum: 60, maximum: 120, preferred: 120)
        displayLink?.add(to: .main, forMode: .default)
    }
    
    @objc private func render() {
        autoreleasepool {
            guard let drawable = metalLayer.nextDrawable() else { return }
            // Render frame
            _ = drawable
        }
    }
    
    deinit {
        displayLink?.invalidate()
    }
}
```

---

## 3. The Metal Rendering Pipeline

### 3.1 Render Pipeline State

```swift
// Pipeline/RenderPipelineSetup.swift

import Metal

class PipelineManager {
    let device: MTLDevice
    
    init(device: MTLDevice) {
        self.device = device
    }
    
    // MARK: - สร้าง Basic Render Pipeline
    func makeBasicRenderPipeline(
        library: MTLLibrary,
        pixelFormat: MTLPixelFormat = .bgra8Unorm
    ) throws -> MTLRenderPipelineState {
        
        // 1. โหลด Vertex และ Fragment Functions
        guard let vertexFunction = library.makeFunction(name: "vertex_main"),
              let fragmentFunction = library.makeFunction(name: "fragment_main") else {
            throw PipelineError.functionNotFound
        }
        
        // 2. สร้าง Pipeline Descriptor
        let pipelineDescriptor = MTLRenderPipelineDescriptor()
        pipelineDescriptor.label = "Basic Render Pipeline"
        pipelineDescriptor.vertexFunction = vertexFunction
        pipelineDescriptor.fragmentFunction = fragmentFunction
        
        // 3. ตั้งค่า Color Attachment
        pipelineDescriptor.colorAttachments[0].pixelFormat = pixelFormat
        
        // 4. ตั้งค่า Blending (สำหรับ transparency)
        pipelineDescriptor.colorAttachments[0].isBlendingEnabled = true
        pipelineDescriptor.colorAttachments[0].rgbBlendOperation = .add
        pipelineDescriptor.colorAttachments[0].alphaBlendOperation = .add
        pipelineDescriptor.colorAttachments[0].sourceRGBBlendFactor = .sourceAlpha
        pipelineDescriptor.colorAttachments[0].destinationRGBBlendFactor = .oneMinusSourceAlpha
        pipelineDescriptor.colorAttachments[0].sourceAlphaBlendFactor = .one
        pipelineDescriptor.colorAttachments[0].destinationAlphaBlendFactor = .oneMinusSourceAlpha
        
        // 5. ตั้งค่า Depth Buffer
        pipelineDescriptor.depthAttachmentPixelFormat = .depth32Float
        
        // 6. ตั้งค่า Vertex Descriptor
        pipelineDescriptor.vertexDescriptor = makeVertexDescriptor()
        
        // 7. Compile Pipeline
        return try device.makeRenderPipelineState(descriptor: pipelineDescriptor)
    }
    
    // MARK: - Vertex Descriptor
    func makeVertexDescriptor() -> MTLVertexDescriptor {
        let descriptor = MTLVertexDescriptor()
        
        // Attribute 0: Position (float3)
        descriptor.attributes[0].format = .float3
        descriptor.attributes[0].offset = 0
        descriptor.attributes[0].bufferIndex = 0
        
        // Attribute 1: Normal (float3)
        descriptor.attributes[1].format = .float3
        descriptor.attributes[1].offset = MemoryLayout<SIMD3<Float>>.stride
        descriptor.attributes[1].bufferIndex = 0
        
        // Attribute 2: UV Coordinates (float2)
        descriptor.attributes[2].format = .float2
        descriptor.attributes[2].offset = MemoryLayout<SIMD3<Float>>.stride * 2
        descriptor.attributes[2].bufferIndex = 0
        
        // Attribute 3: Color (float4)
        descriptor.attributes[3].format = .float4
        descriptor.attributes[3].offset = MemoryLayout<SIMD3<Float>>.stride * 2 + MemoryLayout<SIMD2<Float>>.stride
        descriptor.attributes[3].bufferIndex = 0
        
        // Layout 0: Stride สำหรับ per-vertex data
        let stride = MemoryLayout<Vertex>.stride
        descriptor.layouts[0].stride = stride
        descriptor.layouts[0].stepFunction = .perVertex
        
        return descriptor
    }
    
    // MARK: - Depth Stencil State
    func makeDepthStencilState() -> MTLDepthStencilState? {
        let descriptor = MTLDepthStencilDescriptor()
        descriptor.depthCompareFunction = .lessEqual
        descriptor.isDepthWriteEnabled = true
        return device.makeDepthStencilState(descriptor: descriptor)
    }
    
    enum PipelineError: Error {
        case functionNotFound
        case compilationFailed(String)
    }
}

// MARK: - Vertex Structure (must match MSL struct)
struct Vertex {
    var position: SIMD3<Float>
    var normal: SIMD3<Float>
    var texCoord: SIMD2<Float>
    var color: SIMD4<Float>
}
```

### 3.2 Render Pass Descriptor

```swift
// Pipeline/RenderPassManager.swift

import Metal

class RenderPassManager {
    
    // MARK: - Main Render Pass (จาก MTKView)
    func makeMainRenderPass(from view: MTKView) -> MTLRenderPassDescriptor? {
        return view.currentRenderPassDescriptor
    }
    
    // MARK: - Offscreen Render Pass (Render to Texture)
    func makeOffscreenRenderPass(
        colorTexture: MTLTexture,
        depthTexture: MTLTexture? = nil
    ) -> MTLRenderPassDescriptor {
        let descriptor = MTLRenderPassDescriptor()
        
        // Color Attachment
        descriptor.colorAttachments[0].texture = colorTexture
        descriptor.colorAttachments[0].loadAction = .clear
        descriptor.colorAttachments[0].storeAction = .store
        descriptor.colorAttachments[0].clearColor = MTLClearColor(red: 0, green: 0, blue: 0, alpha: 1)
        
        // Depth Attachment
        if let depth = depthTexture {
            descriptor.depthAttachment.texture = depth
            descriptor.depthAttachment.loadAction = .clear
            descriptor.depthAttachment.storeAction = .dontCare
            descriptor.depthAttachment.clearDepth = 1.0
        }
        
        return descriptor
    }
    
    // MARK: - Shadow Map Render Pass
    func makeShadowRenderPass(shadowMap: MTLTexture) -> MTLRenderPassDescriptor {
        let descriptor = MTLRenderPassDescriptor()
        descriptor.depthAttachment.texture = shadowMap
        descriptor.depthAttachment.loadAction = .clear
        descriptor.depthAttachment.storeAction = .store
        descriptor.depthAttachment.clearDepth = 1.0
        return descriptor
    }
}
```

---

## 4. Metal Shading Language (MSL)

### 4.1 Basic Vertex และ Fragment Shaders

```metal
// Shaders/BasicShaders.metal

#include <metal_stdlib>
using namespace metal;

// MARK: - Data Types และ Structures

// Vertex Input (ตรงกับ Swift Vertex struct)
struct VertexIn {
    float3 position  [[attribute(0)]];
    float3 normal    [[attribute(1)]];
    float2 texCoord  [[attribute(2)]];
    float4 color     [[attribute(3)]];
};

// Vertex Output / Fragment Input
struct VertexOut {
    float4 position   [[position]];  // Clip space position (required)
    float3 worldPos;                  // World space position
    float3 normal;                    // World space normal
    float2 texCoord;
    float4 color;
    float3 viewPos;                   // View space position (for lighting)
};

// Uniform data (ส่งจาก CPU)
struct Uniforms {
    float4x4 modelMatrix;
    float4x4 viewMatrix;
    float4x4 projectionMatrix;
    float4x4 normalMatrix;
    float3 cameraPosition;
    float time;
};

// Light data
struct Light {
    float3 position;
    float3 direction;
    float4 color;
    float intensity;
    int type;  // 0=directional, 1=point, 2=spot
};

// Fragment uniforms
struct FragmentUniforms {
    Light lights[4];
    int lightCount;
    float4 ambientColor;
};

// MARK: - Vertex Shader
vertex VertexOut vertex_main(
    VertexIn in [[stage_in]],
    constant Uniforms& uniforms [[buffer(1)]]
) {
    VertexOut out;
    
    // Transform position to clip space
    float4 worldPos = uniforms.modelMatrix * float4(in.position, 1.0);
    float4 viewPos = uniforms.viewMatrix * worldPos;
    out.position = uniforms.projectionMatrix * viewPos;
    
    // Pass world position
    out.worldPos = worldPos.xyz;
    out.viewPos = viewPos.xyz;
    
    // Transform normal to world space
    out.normal = normalize((uniforms.normalMatrix * float4(in.normal, 0.0)).xyz);
    
    // Pass through
    out.texCoord = in.texCoord;
    out.color = in.color;
    
    return out;
}

// MARK: - Fragment Shader
fragment float4 fragment_main(
    VertexOut in [[stage_in]],
    constant FragmentUniforms& uniforms [[buffer(0)]],
    texture2d<float> diffuseTexture [[texture(0)]],
    sampler textureSampler [[sampler(0)]]
) {
    // Sample texture
    float4 textureColor = diffuseTexture.sample(textureSampler, in.texCoord);
    
    // Base color from texture or vertex color
    float4 baseColor = textureColor * in.color;
    
    // Ambient lighting
    float3 ambient = uniforms.ambientColor.rgb * baseColor.rgb;
    
    // Accumulated lighting
    float3 lighting = ambient;
    
    for (int i = 0; i < uniforms.lightCount; i++) {
        Light light = uniforms.lights[i];
        
        float3 lightDir;
        float attenuation = 1.0;
        
        if (light.type == 0) {
            // Directional light
            lightDir = normalize(-light.direction);
        } else {
            // Point light
            float3 toLight = light.position - in.worldPos;
            float distance = length(toLight);
            lightDir = normalize(toLight);
            attenuation = 1.0 / (1.0 + 0.09 * distance + 0.032 * distance * distance);
        }
        
        // Diffuse
        float diff = max(dot(in.normal, lightDir), 0.0);
        float3 diffuse = diff * light.color.rgb * baseColor.rgb;
        
        // Specular (Blinn-Phong)
        float3 viewDir = normalize(-in.viewPos);
        float3 halfDir = normalize(lightDir + viewDir);
        float spec = pow(max(dot(in.normal, halfDir), 0.0), 64.0);
        float3 specular = spec * light.color.rgb * 0.5;
        
        lighting += (diffuse + specular) * attenuation * light.intensity;
    }
    
    return float4(lighting, baseColor.a);
}

// MARK: - Simple Color Shader (ไม่มี lighting)
vertex VertexOut vertex_color(
    VertexIn in [[stage_in]],
    constant Uniforms& uniforms [[buffer(1)]]
) {
    VertexOut out;
    float4 worldPos = uniforms.modelMatrix * float4(in.position, 1.0);
    out.position = uniforms.projectionMatrix * uniforms.viewMatrix * worldPos;
    out.worldPos = worldPos.xyz;
    out.normal = in.normal;
    out.texCoord = in.texCoord;
    out.color = in.color;
    out.viewPos = (uniforms.viewMatrix * worldPos).xyz;
    return out;
}

fragment float4 fragment_color(VertexOut in [[stage_in]]) {
    return in.color;
}
```

### 4.2 Compute Shaders

```metal
// Shaders/ComputeShaders.metal

#include <metal_stdlib>
using namespace metal;

// MARK: - Image Processing: Gaussian Blur
kernel void gaussian_blur(
    texture2d<float, access::read> inputTexture [[texture(0)]],
    texture2d<float, access::write> outputTexture [[texture(1)]],
    uint2 gid [[thread_position_in_grid]]
) {
    // ตรวจสอบ bounds
    uint width = inputTexture.get_width();
    uint height = inputTexture.get_height();
    
    if (gid.x >= width || gid.y >= height) return;
    
    // Gaussian kernel 5x5
    float kernel[5][5] = {
        {0.0030, 0.0133, 0.0219, 0.0133, 0.0030},
        {0.0133, 0.0596, 0.0983, 0.0596, 0.0133},
        {0.0219, 0.0983, 0.1621, 0.0983, 0.0219},
        {0.0133, 0.0596, 0.0983, 0.0596, 0.0133},
        {0.0030, 0.0133, 0.0219, 0.0133, 0.0030}
    };
    
    float4 color = float4(0.0);
    
    for (int dy = -2; dy <= 2; dy++) {
        for (int dx = -2; dx <= 2; dx++) {
            int2 coord = int2(gid) + int2(dx, dy);
            coord = clamp(coord, int2(0, 0), int2(width - 1, height - 1));
            color += inputTexture.read(uint2(coord)) * kernel[dy + 2][dx + 2];
        }
    }
    
    outputTexture.write(color, gid);
}

// MARK: - Edge Detection (Sobel)
kernel void sobel_edge_detection(
    texture2d<float, access::read> inputTexture [[texture(0)]],
    texture2d<float, access::write> outputTexture [[texture(1)]],
    uint2 gid [[thread_position_in_grid]]
) {
    uint width = inputTexture.get_width();
    uint height = inputTexture.get_height();
    
    if (gid.x >= width || gid.y >= height) return;
    
    // Sobel operators
    float sobelX[3][3] = {{-1, 0, 1}, {-2, 0, 2}, {-1, 0, 1}};
    float sobelY[3][3] = {{-1, -2, -1}, {0, 0, 0}, {1, 2, 1}};
    
    float gx = 0, gy = 0;
    
    for (int dy = -1; dy <= 1; dy++) {
        for (int dx = -1; dx <= 1; dx++) {
            int2 coord = int2(gid) + int2(dx, dy);
            coord = clamp(coord, int2(0, 0), int2(width - 1, height - 1));
            
            float luminance = dot(inputTexture.read(uint2(coord)).rgb, float3(0.299, 0.587, 0.114));
            gx += luminance * sobelX[dy + 1][dx + 1];
            gy += luminance * sobelY[dy + 1][dx + 1];
        }
    }
    
    float magnitude = sqrt(gx * gx + gy * gy);
    magnitude = clamp(magnitude, 0.0, 1.0);
    
    outputTexture.write(float4(magnitude, magnitude, magnitude, 1.0), gid);
}

// MARK: - Particle System Compute Shader
struct Particle {
    float2 position;
    float2 velocity;
    float4 color;
    float life;
    float maxLife;
    float size;
    float rotation;
};

kernel void update_particles(
    device Particle* particles [[buffer(0)]],
    constant float& deltaTime [[buffer(1)]],
    constant float2& gravity [[buffer(2)]],
    constant float& time [[buffer(3)]],
    uint id [[thread_position_in_grid]]
) {
    Particle p = particles[id];
    
    // Update life
    p.life -= deltaTime;
    
    if (p.life <= 0) {
        // Reset particle
        float angle = float(id) * 2.399963; // Golden angle
        float speed = 0.3 + fmod(float(id) * 0.1, 0.7);
        p.position = float2(0.0, -0.3);
        p.velocity = float2(cos(angle) * speed, sin(angle) * speed + 1.5);
        p.life = p.maxLife;
        p.color = float4(
            0.5 + 0.5 * sin(time + float(id) * 0.5),
            0.5 + 0.5 * cos(time + float(id) * 0.3),
            0.5 + 0.5 * sin(time * 0.7 + float(id) * 0.8),
            1.0
        );
    } else {
        // Physics update
        p.velocity += gravity * deltaTime;
        p.position += p.velocity * deltaTime;
        p.rotation += 2.0 * deltaTime;
        
        // Fade out
        float lifeRatio = p.life / p.maxLife;
        p.color.a = lifeRatio;
        p.size = 0.02 * (0.5 + 0.5 * lifeRatio);
    }
    
    particles[id] = p;
}

// MARK: - Render Particles (Vertex Shader)
struct ParticleVertexOut {
    float4 position [[position]];
    float4 color;
    float size [[point_size]];
    float2 texCoord [[point_coord]];
};

vertex ParticleVertexOut particle_vertex(
    device Particle* particles [[buffer(0)]],
    constant float4x4& projectionMatrix [[buffer(1)]],
    uint vertexId [[vertex_id]]
) {
    Particle p = particles[vertexId];
    
    ParticleVertexOut out;
    out.position = projectionMatrix * float4(p.position, 0.0, 1.0);
    out.color = p.color;
    out.size = p.size * 500.0;  // Convert to pixel size
    
    return out;
}

fragment float4 particle_fragment(
    ParticleVertexOut in [[stage_in]],
    float2 pointCoord [[point_coord]]
) {
    // วาด circle สำหรับแต่ละ particle
    float2 center = pointCoord - float2(0.5);
    float dist = length(center);
    
    if (dist > 0.5) discard_fragment();
    
    // Soft edge
    float alpha = 1.0 - smoothstep(0.3, 0.5, dist);
    return float4(in.color.rgb, in.color.a * alpha);
}
```

---

## 5. Buffers and Textures

### 5.1 MTLBuffer Management

```swift
// Buffers/BufferManager.swift

import Metal
import simd

// MARK: - Buffer Storage Modes
/*
 MTLStorageMode:
 - .shared     : CPU และ GPU เข้าถึงได้ (Apple Silicon unified memory)
 - .managed    : มี copy ทั้ง CPU และ GPU ต้อง sync ด้วย didModifyRange
 - .private    : GPU เข้าถึงได้อย่างเดียว (ต้องใช้ blit encoder ถ่าย data)
 - .memoryless : เฉพาะ Apple Silicon tile memory
*/

class BufferManager {
    let device: MTLDevice
    
    init(device: MTLDevice) {
        self.device = device
    }
    
    // MARK: - Shared Buffer (CPU + GPU, unified memory)
    func makeSharedBuffer<T>(data: [T]) -> MTLBuffer? {
        let byteCount = data.count * MemoryLayout<T>.stride
        return device.makeBuffer(bytes: data, length: byteCount, options: .storageModeShared)
    }
    
    // MARK: - Private Buffer (GPU only, better performance)
    func makePrivateBuffer<T>(
        from data: [T],
        commandBuffer: MTLCommandBuffer
    ) -> MTLBuffer? {
        let byteCount = data.count * MemoryLayout<T>.stride
        
        // สร้าง staging buffer ใน shared memory
        guard let stagingBuffer = device.makeBuffer(
            bytes: data,
            length: byteCount,
            options: .storageModeShared
        ) else { return nil }
        
        // สร้าง private buffer
        guard let privateBuffer = device.makeBuffer(
            length: byteCount,
            options: .storageModePrivate
        ) else { return nil }
        
        // Copy จาก staging ไป private ด้วย blit encoder
        guard let blitEncoder = commandBuffer.makeBlitCommandEncoder() else {
            return nil
        }
        
        blitEncoder.copy(
            from: stagingBuffer, sourceOffset: 0,
            to: privateBuffer, destinationOffset: 0,
            size: byteCount
        )
        blitEncoder.endEncoding()
        
        return privateBuffer
    }
    
    // MARK: - Triple Buffering (ป้องกัน CPU/GPU conflict)
    class TripleBuffer<T> {
        private var buffers: [MTLBuffer]
        private var currentIndex = 0
        private let semaphore = DispatchSemaphore(value: 3)
        
        init(device: MTLDevice, count: Int) {
            let byteCount = MemoryLayout<T>.stride
            buffers = (0..<3).compactMap {
                _ in device.makeBuffer(length: byteCount, options: .storageModeShared)
            }
        }
        
        func beginFrame() -> (MTLBuffer, UnsafeMutablePointer<T>) {
            semaphore.wait()
            currentIndex = (currentIndex + 1) % 3
            let buffer = buffers[currentIndex]
            let pointer = buffer.contents().bindMemory(to: T.self, capacity: 1)
            return (buffer, pointer)
        }
        
        func endFrame() {
            semaphore.signal()
        }
    }
}

// MARK: - Uniforms Buffer
struct SceneUniforms {
    var modelMatrix: float4x4
    var viewMatrix: float4x4
    var projectionMatrix: float4x4
    var normalMatrix: float4x4
    var cameraPosition: SIMD3<Float>
    var time: Float
    
    init() {
        modelMatrix = matrix_identity_float4x4
        viewMatrix = matrix_identity_float4x4
        projectionMatrix = matrix_identity_float4x4
        normalMatrix = matrix_identity_float4x4
        cameraPosition = .zero
        time = 0
    }
}
```

### 5.2 MTLTexture Creation และ Sampling

```swift
// Textures/TextureManager.swift

import Metal
import MetalKit
import UIKit

class TextureManager {
    let device: MTLDevice
    let textureLoader: MTKTextureLoader
    
    init(device: MTLDevice) {
        self.device = device
        self.textureLoader = MTKTextureLoader(device: device)
    }
    
    // MARK: - Load Texture from Asset
    func loadTexture(named name: String) async throws -> MTLTexture {
        let options: [MTKTextureLoader.Option: Any] = [
            .textureUsage: MTLTextureUsage.shaderRead.rawValue,
            .textureStorageMode: MTLStorageMode.private.rawValue,
            .generateMipmaps: true  // Auto-generate mipmaps
        ]
        
        return try await textureLoader.newTexture(name: name, scaleFactor: 1.0,
                                                   bundle: nil, options: options)
    }
    
    // MARK: - Load Texture from UIImage
    func loadTexture(from image: UIImage) throws -> MTLTexture {
        guard let cgImage = image.cgImage else {
            throw TextureError.invalidImage
        }
        
        let options: [MTKTextureLoader.Option: Any] = [
            .textureUsage: MTLTextureUsage.shaderRead.rawValue,
            .generateMipmaps: true
        ]
        
        return try textureLoader.newTexture(cgImage: cgImage, options: options)
    }
    
    // MARK: - Create Empty Texture (Render Target)
    func makeRenderTargetTexture(
        width: Int,
        height: Int,
        pixelFormat: MTLPixelFormat = .bgra8Unorm
    ) -> MTLTexture? {
        let descriptor = MTLTextureDescriptor.texture2DDescriptor(
            pixelFormat: pixelFormat,
            width: width,
            height: height,
            mipmapped: false
        )
        descriptor.usage = [.renderTarget, .shaderRead]
        descriptor.storageMode = .private
        
        return device.makeTexture(descriptor: descriptor)
    }
    
    // MARK: - Create Depth Texture
    func makeDepthTexture(width: Int, height: Int) -> MTLTexture? {
        let descriptor = MTLTextureDescriptor.texture2DDescriptor(
            pixelFormat: .depth32Float,
            width: width,
            height: height,
            mipmapped: false
        )
        descriptor.usage = [.renderTarget, .shaderRead]
        descriptor.storageMode = .private
        
        return device.makeTexture(descriptor: descriptor)
    }
    
    // MARK: - Sampler State
    func makeLinearSampler() -> MTLSamplerState? {
        let descriptor = MTLSamplerDescriptor()
        descriptor.minFilter = .linear
        descriptor.magFilter = .linear
        descriptor.mipFilter = .linear
        descriptor.sAddressMode = .repeat
        descriptor.tAddressMode = .repeat
        descriptor.maxAnisotropy = 16  // Anisotropic filtering
        return device.makeSamplerState(descriptor: descriptor)
    }
    
    func makeNearestSampler() -> MTLSamplerState? {
        let descriptor = MTLSamplerDescriptor()
        descriptor.minFilter = .nearest
        descriptor.magFilter = .nearest
        descriptor.mipFilter = .nearest
        descriptor.sAddressMode = .clampToEdge
        descriptor.tAddressMode = .clampToEdge
        return device.makeSamplerState(descriptor: descriptor)
    }
    
    // MARK: - Generate Checkerboard Texture (สำหรับ Testing)
    func makeCheckerboardTexture(size: Int = 256, squareSize: Int = 32) -> MTLTexture? {
        let descriptor = MTLTextureDescriptor.texture2DDescriptor(
            pixelFormat: .rgba8Unorm,
            width: size,
            height: size,
            mipmapped: true
        )
        descriptor.usage = [.shaderRead]
        
        guard let texture = device.makeTexture(descriptor: descriptor) else { return nil }
        
        var pixels = [UInt8](repeating: 0, count: size * size * 4)
        
        for y in 0..<size {
            for x in 0..<size {
                let isWhite = ((x / squareSize) + (y / squareSize)) % 2 == 0
                let baseIndex = (y * size + x) * 4
                let value: UInt8 = isWhite ? 255 : 64
                pixels[baseIndex + 0] = value  // R
                pixels[baseIndex + 1] = value  // G
                pixels[baseIndex + 2] = value  // B
                pixels[baseIndex + 3] = 255    // A
            }
        }
        
        let region = MTLRegion(
            origin: MTLOrigin(x: 0, y: 0, z: 0),
            size: MTLSize(width: size, height: size, depth: 1)
        )
        texture.replace(region: region, mipmapLevel: 0, withBytes: pixels, bytesPerRow: size * 4)
        
        return texture
    }
    
    enum TextureError: Error {
        case invalidImage
        case creationFailed
    }
}
```

---

## 6. Drawing Primitives

### 6.1 Triangle Rendering

```swift
// Drawing/TriangleRenderer.swift

import Metal
import MetalKit
import simd

class TriangleRenderer: NSObject {
    let device: MTLDevice
    var pipelineState: MTLRenderPipelineState!
    var vertexBuffer: MTLBuffer!
    var uniformBuffer: MTLBuffer!
    
    // Triangle vertices (position + color)
    let vertices: [Vertex] = [
        Vertex(position: SIMD3<Float>( 0.0,  0.5, 0.0),
               normal: SIMD3<Float>(0, 0, 1),
               texCoord: SIMD2<Float>(0.5, 0.0),
               color: SIMD4<Float>(1.0, 0.0, 0.0, 1.0)),  // Top - Red
        
        Vertex(position: SIMD3<Float>(-0.5, -0.5, 0.0),
               normal: SIMD3<Float>(0, 0, 1),
               texCoord: SIMD2<Float>(0.0, 1.0),
               color: SIMD4<Float>(0.0, 1.0, 0.0, 1.0)),  // Bottom-left - Green
        
        Vertex(position: SIMD3<Float>( 0.5, -0.5, 0.0),
               normal: SIMD3<Float>(0, 0, 1),
               texCoord: SIMD2<Float>(1.0, 1.0),
               color: SIMD4<Float>(0.0, 0.0, 1.0, 1.0)),  // Bottom-right - Blue
    ]
    
    init(device: MTLDevice, library: MTLLibrary) {
        self.device = device
        super.init()
        
        setupBuffers()
        setupPipeline(library: library)
    }
    
    func setupBuffers() {
        vertexBuffer = device.makeBuffer(
            bytes: vertices,
            length: vertices.count * MemoryLayout<Vertex>.stride,
            options: .storageModeShared
        )
        vertexBuffer.label = "Triangle Vertices"
        
        var uniforms = SceneUniforms()
        uniformBuffer = device.makeBuffer(
            bytes: &uniforms,
            length: MemoryLayout<SceneUniforms>.stride,
            options: .storageModeShared
        )
    }
    
    func setupPipeline(library: MTLLibrary) {
        let descriptor = MTLRenderPipelineDescriptor()
        descriptor.vertexFunction = library.makeFunction(name: "vertex_color")
        descriptor.fragmentFunction = library.makeFunction(name: "fragment_color")
        descriptor.colorAttachments[0].pixelFormat = .bgra8Unorm
        
        // Vertex descriptor
        let vertexDescriptor = MTLVertexDescriptor()
        vertexDescriptor.attributes[0].format = .float3
        vertexDescriptor.attributes[0].offset = 0
        vertexDescriptor.attributes[0].bufferIndex = 0
        vertexDescriptor.attributes[3].format = .float4
        vertexDescriptor.attributes[3].offset = MemoryLayout<SIMD3<Float>>.stride * 2 + MemoryLayout<SIMD2<Float>>.stride
        vertexDescriptor.attributes[3].bufferIndex = 0
        vertexDescriptor.layouts[0].stride = MemoryLayout<Vertex>.stride
        descriptor.vertexDescriptor = vertexDescriptor
        
        pipelineState = try! device.makeRenderPipelineState(descriptor: descriptor)
    }
    
    func draw(encoder: MTLRenderCommandEncoder, time: Float) {
        // Update uniforms
        let uniformsPtr = uniformBuffer.contents().bindMemory(to: SceneUniforms.self, capacity: 1)
        uniformsPtr.pointee.time = time
        
        // Rotate triangle
        let angle = time * 0.5
        let rotation = float4x4(rotationZ: angle)
        uniformsPtr.pointee.modelMatrix = rotation
        
        encoder.setRenderPipelineState(pipelineState)
        encoder.setVertexBuffer(vertexBuffer, offset: 0, index: 0)
        encoder.setVertexBuffer(uniformBuffer, offset: 0, index: 1)
        encoder.drawPrimitives(type: .triangle, vertexStart: 0, vertexCount: 3)
    }
}
```

### 6.2 Index Buffers

```swift
// Drawing/CubeRenderer.swift

import Metal
import simd

class CubeRenderer {
    let device: MTLDevice
    var vertexBuffer: MTLBuffer!
    var indexBuffer: MTLBuffer!
    
    // Cube vertices
    let cubeVertices: [Vertex] = [
        // Front face
        Vertex(position: SIMD3<Float>(-0.5, -0.5,  0.5), normal: SIMD3<Float>(0, 0, 1), texCoord: SIMD2<Float>(0, 1), color: SIMD4<Float>(1, 0, 0, 1)),
        Vertex(position: SIMD3<Float>( 0.5, -0.5,  0.5), normal: SIMD3<Float>(0, 0, 1), texCoord: SIMD2<Float>(1, 1), color: SIMD4<Float>(1, 0, 0, 1)),
        Vertex(position: SIMD3<Float>( 0.5,  0.5,  0.5), normal: SIMD3<Float>(0, 0, 1), texCoord: SIMD2<Float>(1, 0), color: SIMD4<Float>(1, 0, 0, 1)),
        Vertex(position: SIMD3<Float>(-0.5,  0.5,  0.5), normal: SIMD3<Float>(0, 0, 1), texCoord: SIMD2<Float>(0, 0), color: SIMD4<Float>(1, 0, 0, 1)),
        
        // Back face
        Vertex(position: SIMD3<Float>( 0.5, -0.5, -0.5), normal: SIMD3<Float>(0, 0, -1), texCoord: SIMD2<Float>(0, 1), color: SIMD4<Float>(0, 1, 0, 1)),
        Vertex(position: SIMD3<Float>(-0.5, -0.5, -0.5), normal: SIMD3<Float>(0, 0, -1), texCoord: SIMD2<Float>(1, 1), color: SIMD4<Float>(0, 1, 0, 1)),
        Vertex(position: SIMD3<Float>(-0.5,  0.5, -0.5), normal: SIMD3<Float>(0, 0, -1), texCoord: SIMD2<Float>(1, 0), color: SIMD4<Float>(0, 1, 0, 1)),
        Vertex(position: SIMD3<Float>( 0.5,  0.5, -0.5), normal: SIMD3<Float>(0, 0, -1), texCoord: SIMD2<Float>(0, 0), color: SIMD4<Float>(0, 1, 0, 1)),
        
        // Top face
        Vertex(position: SIMD3<Float>(-0.5,  0.5,  0.5), normal: SIMD3<Float>(0, 1, 0), texCoord: SIMD2<Float>(0, 1), color: SIMD4<Float>(0, 0, 1, 1)),
        Vertex(position: SIMD3<Float>( 0.5,  0.5,  0.5), normal: SIMD3<Float>(0, 1, 0), texCoord: SIMD2<Float>(1, 1), color: SIMD4<Float>(0, 0, 1, 1)),
        Vertex(position: SIMD3<Float>( 0.5,  0.5, -0.5), normal: SIMD3<Float>(0, 1, 0), texCoord: SIMD2<Float>(1, 0), color: SIMD4<Float>(0, 0, 1, 1)),
        Vertex(position: SIMD3<Float>(-0.5,  0.5, -0.5), normal: SIMD3<Float>(0, 1, 0), texCoord: SIMD2<Float>(0, 0), color: SIMD4<Float>(0, 0, 1, 1)),
        
        // Bottom face
        Vertex(position: SIMD3<Float>(-0.5, -0.5, -0.5), normal: SIMD3<Float>(0, -1, 0), texCoord: SIMD2<Float>(0, 1), color: SIMD4<Float>(1, 1, 0, 1)),
        Vertex(position: SIMD3<Float>( 0.5, -0.5, -0.5), normal: SIMD3<Float>(0, -1, 0), texCoord: SIMD2<Float>(1, 1), color: SIMD4<Float>(1, 1, 0, 1)),
        Vertex(position: SIMD3<Float>( 0.5, -0.5,  0.5), normal: SIMD3<Float>(0, -1, 0), texCoord: SIMD2<Float>(1, 0), color: SIMD4<Float>(1, 1, 0, 1)),
        Vertex(position: SIMD3<Float>(-0.5, -0.5,  0.5), normal: SIMD3<Float>(0, -1, 0), texCoord: SIMD2<Float>(0, 0), color: SIMD4<Float>(1, 1, 0, 1)),
        
        // Right face
        Vertex(position: SIMD3<Float>( 0.5, -0.5,  0.5), normal: SIMD3<Float>(1, 0, 0), texCoord: SIMD2<Float>(0, 1), color: SIMD4<Float>(1, 0, 1, 1)),
        Vertex(position: SIMD3<Float>( 0.5, -0.5, -0.5), normal: SIMD3<Float>(1, 0, 0), texCoord: SIMD2<Float>(1, 1), color: SIMD4<Float>(1, 0, 1, 1)),
        Vertex(position: SIMD3<Float>( 0.5,  0.5, -0.5), normal: SIMD3<Float>(1, 0, 0), texCoord: SIMD2<Float>(1, 0), color: SIMD4<Float>(1, 0, 1, 1)),
        Vertex(position: SIMD3<Float>( 0.5,  0.5,  0.5), normal: SIMD3<Float>(1, 0, 0), texCoord: SIMD2<Float>(0, 0), color: SIMD4<Float>(1, 0, 1, 1)),
        
        // Left face
        Vertex(position: SIMD3<Float>(-0.5, -0.5, -0.5), normal: SIMD3<Float>(-1, 0, 0), texCoord: SIMD2<Float>(0, 1), color: SIMD4<Float>(0, 1, 1, 1)),
        Vertex(position: SIMD3<Float>(-0.5, -0.5,  0.5), normal: SIMD3<Float>(-1, 0, 0), texCoord: SIMD2<Float>(1, 1), color: SIMD4<Float>(0, 1, 1, 1)),
        Vertex(position: SIMD3<Float>(-0.5,  0.5,  0.5), normal: SIMD3<Float>(-1, 0, 0), texCoord: SIMD2<Float>(1, 0), color: SIMD4<Float>(0, 1, 1, 1)),
        Vertex(position: SIMD3<Float>(-0.5,  0.5, -0.5), normal: SIMD3<Float>(-1, 0, 0), texCoord: SIMD2<Float>(0, 0), color: SIMD4<Float>(0, 1, 1, 1)),
    ]
    
    // Index buffer - 6 faces * 2 triangles * 3 vertices = 36 indices
    let cubeIndices: [UInt16] = [
        0,  1,  2,  2,  3,  0,   // Front
        4,  5,  6,  6,  7,  4,   // Back
        8,  9,  10, 10, 11, 8,   // Top
        12, 13, 14, 14, 15, 12,  // Bottom
        16, 17, 18, 18, 19, 16,  // Right
        20, 21, 22, 22, 23, 20   // Left
    ]
    
    init(device: MTLDevice) {
        self.device = device
        setupBuffers()
    }
    
    func setupBuffers() {
        vertexBuffer = device.makeBuffer(
            bytes: cubeVertices,
            length: cubeVertices.count * MemoryLayout<Vertex>.stride,
            options: .storageModeShared
        )
        
        indexBuffer = device.makeBuffer(
            bytes: cubeIndices,
            length: cubeIndices.count * MemoryLayout<UInt16>.stride,
            options: .storageModeShared
        )
    }
    
    func draw(encoder: MTLRenderCommandEncoder) {
        encoder.setVertexBuffer(vertexBuffer, offset: 0, index: 0)
        encoder.drawIndexedPrimitives(
            type: .triangle,
            indexCount: cubeIndices.count,
            indexType: .uint16,
            indexBuffer: indexBuffer,
            indexBufferOffset: 0
        )
    }
}
```

### 6.3 Instanced Rendering

```swift
// Drawing/InstancedRenderer.swift

import Metal
import simd

struct InstanceData {
    var transform: float4x4
    var color: SIMD4<Float>
}

class InstancedRenderer {
    let device: MTLDevice
    var instanceBuffer: MTLBuffer!
    let instanceCount: Int
    
    init(device: MTLDevice, instanceCount: Int) {
        self.device = device
        self.instanceCount = instanceCount
        setupInstanceBuffer()
    }
    
    func setupInstanceBuffer() {
        instanceBuffer = device.makeBuffer(
            length: instanceCount * MemoryLayout<InstanceData>.stride,
            options: .storageModeShared
        )
        
        updateInstances(time: 0)
    }
    
    func updateInstances(time: Float) {
        let ptr = instanceBuffer.contents().bindMemory(
            to: InstanceData.self,
            capacity: instanceCount
        )
        
        for i in 0..<instanceCount {
            let fi = Float(i)
            let angle = fi * (2 * .pi / Float(instanceCount)) + time * 0.5
            let radius: Float = 3.0
            
            let x = cos(angle) * radius
            let z = sin(angle) * radius
            let y = sin(fi * 0.5 + time) * 0.5
            
            let translation = float4x4(translation: SIMD3<Float>(x, y, z))
            let scale = float4x4(scale: SIMD3<Float>(0.3, 0.3, 0.3))
            let rotation = float4x4(rotationY: time + fi)
            
            ptr[i].transform = translation * rotation * scale
            ptr[i].color = SIMD4<Float>(
                0.5 + 0.5 * sin(fi * 0.3 + time),
                0.5 + 0.5 * cos(fi * 0.5 + time),
                0.5 + 0.5 * sin(fi * 0.7 + time * 0.7),
                1.0
            )
        }
    }
    
    func draw(encoder: MTLRenderCommandEncoder, indexBuffer: MTLBuffer, indexCount: Int) {
        encoder.setVertexBuffer(instanceBuffer, offset: 0, index: 2)
        encoder.drawIndexedPrimitives(
            type: .triangle,
            indexCount: indexCount,
            indexType: .uint16,
            indexBuffer: indexBuffer,
            indexBufferOffset: 0,
            instanceCount: instanceCount
        )
    }
}
```

---

## 7. 3D Transformations

### 7.1 Matrix Mathematics ด้วย simd

```swift
// Math/MatrixMath.swift

import simd
import Foundation

// MARK: - float4x4 Extensions
extension float4x4 {
    
    // MARK: - Translation Matrix
    init(translation: SIMD3<Float>) {
        self = matrix_identity_float4x4
        columns.3 = SIMD4<Float>(translation.x, translation.y, translation.z, 1)
    }
    
    // MARK: - Scale Matrix
    init(scale: SIMD3<Float>) {
        self = matrix_identity_float4x4
        columns.0.x = scale.x
        columns.1.y = scale.y
        columns.2.z = scale.z
    }
    
    // MARK: - Rotation Matrices
    init(rotationX angle: Float) {
        self = matrix_identity_float4x4
        columns.1.y = cos(angle)
        columns.1.z = sin(angle)
        columns.2.y = -sin(angle)
        columns.2.z = cos(angle)
    }
    
    init(rotationY angle: Float) {
        self = matrix_identity_float4x4
        columns.0.x = cos(angle)
        columns.0.z = -sin(angle)
        columns.2.x = sin(angle)
        columns.2.z = cos(angle)
    }
    
    init(rotationZ angle: Float) {
        self = matrix_identity_float4x4
        columns.0.x = cos(angle)
        columns.0.y = sin(angle)
        columns.1.x = -sin(angle)
        columns.1.y = cos(angle)
    }
    
    // MARK: - Perspective Projection
    init(perspectiveFov fov: Float, aspectRatio: Float, near: Float, far: Float) {
        let y = 1 / tan(fov * 0.5)
        let x = y / aspectRatio
        let z = far / (near - far)
        let w = z * near
        
        let col0 = SIMD4<Float>(x, 0, 0, 0)
        let col1 = SIMD4<Float>(0, y, 0, 0)
        let col2 = SIMD4<Float>(0, 0, z, -1)
        let col3 = SIMD4<Float>(0, 0, w, 0)
        
        self.init(col0, col1, col2, col3)
    }
    
    // MARK: - Orthographic Projection
    init(orthographicLeft left: Float, right: Float, bottom: Float, top: Float, near: Float, far: Float) {
        let tx = -(right + left) / (right - left)
        let ty = -(top + bottom) / (top - bottom)
        let tz = -(far + near) / (far - near)
        
        let col0 = SIMD4<Float>(2 / (right - left), 0, 0, 0)
        let col1 = SIMD4<Float>(0, 2 / (top - bottom), 0, 0)
        let col2 = SIMD4<Float>(0, 0, -2 / (far - near), 0)
        let col3 = SIMD4<Float>(tx, ty, tz, 1)
        
        self.init(col0, col1, col2, col3)
    }
    
    // MARK: - Look At Matrix (View Matrix)
    static func lookAt(eye: SIMD3<Float>, center: SIMD3<Float>, up: SIMD3<Float>) -> float4x4 {
        let z = normalize(eye - center)  // Forward (pointing away from scene)
        let x = normalize(cross(up, z))  // Right
        let y = cross(z, x)              // Up (recalculated)
        
        let col0 = SIMD4<Float>(x.x, y.x, z.x, 0)
        let col1 = SIMD4<Float>(x.y, y.y, z.y, 0)
        let col2 = SIMD4<Float>(x.z, y.z, z.z, 0)
        let col3 = SIMD4<Float>(-dot(x, eye), -dot(y, eye), -dot(z, eye), 1)
        
        return float4x4(col0, col1, col2, col3)
    }
    
    // MARK: - Normal Matrix (inverse transpose)
    var normalMatrix: float4x4 {
        return transpose(inverse(self))
    }
}

// MARK: - Camera
class Camera {
    var position: SIMD3<Float> = SIMD3<Float>(0, 2, 5)
    var target: SIMD3<Float> = .zero
    var up: SIMD3<Float> = SIMD3<Float>(0, 1, 0)
    
    var fov: Float = 65 * .pi / 180  // 65 degrees
    var aspectRatio: Float = 1.0
    var nearPlane: Float = 0.1
    var farPlane: Float = 1000.0
    
    // Orbit controls
    var theta: Float = 0    // Horizontal angle
    var phi: Float = 0.3    // Vertical angle
    var radius: Float = 5.0
    
    var viewMatrix: float4x4 {
        float4x4.lookAt(eye: position, center: target, up: up)
    }
    
    var projectionMatrix: float4x4 {
        float4x4(
            perspectiveFov: fov,
            aspectRatio: aspectRatio,
            near: nearPlane,
            far: farPlane
        )
    }
    
    func orbit(deltaX: Float, deltaY: Float) {
        theta += deltaX * 0.01
        phi = max(-1.5, min(1.5, phi + deltaY * 0.01))
        updatePosition()
    }
    
    func zoom(delta: Float) {
        radius = max(1.0, min(20.0, radius - delta * 0.1))
        updatePosition()
    }
    
    private func updatePosition() {
        position = SIMD3<Float>(
            target.x + radius * cos(phi) * sin(theta),
            target.y + radius * sin(phi),
            target.z + radius * cos(phi) * cos(theta)
        )
    }
}
```

---

## 8. Lighting and Materials

### 8.1 Phong Lighting ใน MSL

```metal
// Shaders/LightingShaders.metal

#include <metal_stdlib>
using namespace metal;

// MARK: - Material Properties
struct Material {
    float4 ambient;
    float4 diffuse;
    float4 specular;
    float shininess;
    bool hasTexture;
};

// MARK: - Phong Lighting Function
float3 phongLighting(
    float3 position,
    float3 normal,
    float3 viewDirection,
    constant Light* lights,
    int lightCount,
    Material material,
    float4 textureColor
) {
    float3 ambient = material.ambient.rgb * 0.1;
    float3 result = ambient;
    
    for (int i = 0; i < lightCount; i++) {
        Light light = lights[i];
        
        float3 lightDir;
        float attenuation = 1.0;
        
        if (light.type == 0) {
            // Directional Light
            lightDir = normalize(-light.direction);
        } else if (light.type == 1) {
            // Point Light
            float3 toLight = light.position - position;
            float dist = length(toLight);
            lightDir = normalize(toLight);
            
            // Attenuation: 1 / (constant + linear*d + quadratic*d^2)
            attenuation = 1.0 / (1.0 + 0.09 * dist + 0.032 * dist * dist);
        } else {
            // Spot Light
            float3 toLight = light.position - position;
            lightDir = normalize(toLight);
            float dist = length(toLight);
            
            float theta = dot(lightDir, normalize(-light.direction));
            float epsilon = 0.2;
            float spotIntensity = clamp((theta - 0.8) / epsilon, 0.0, 1.0);
            attenuation = spotIntensity / (1.0 + 0.09 * dist);
        }
        
        // Diffuse
        float diffFactor = max(dot(normal, lightDir), 0.0);
        float3 diffuse = diffFactor * light.color.rgb * material.diffuse.rgb;
        
        // Specular (Blinn-Phong)
        float3 halfVector = normalize(lightDir + viewDirection);
        float specFactor = pow(max(dot(normal, halfVector), 0.0), material.shininess);
        float3 specular = specFactor * light.color.rgb * material.specular.rgb;
        
        result += (diffuse + specular) * attenuation * light.intensity;
    }
    
    // Combine with texture
    float3 baseColor = material.hasTexture ? textureColor.rgb : material.diffuse.rgb;
    return result * baseColor;
}

// MARK: - Normal Mapping Fragment Shader
fragment float4 fragment_normal_mapped(
    VertexOut in [[stage_in]],
    constant FragmentUniforms& uniforms [[buffer(0)]],
    texture2d<float> diffuseTexture [[texture(0)]],
    texture2d<float> normalMap [[texture(1)]],
    sampler textureSampler [[sampler(0)]]
) {
    // Sample textures
    float4 diffuseColor = diffuseTexture.sample(textureSampler, in.texCoord);
    float4 normalSample = normalMap.sample(textureSampler, in.texCoord);
    
    // Convert normal from [0,1] to [-1,1]
    float3 normal = normalize(normalSample.rgb * 2.0 - 1.0);
    
    // Transform to world space using TBN matrix
    // (Simplified - in practice you need tangent/bitangent)
    float3 worldNormal = normalize(in.normal);
    
    // Calculate view direction
    float3 viewDir = normalize(-in.viewPos);
    
    // Apply Phong lighting
    Material mat;
    mat.ambient = float4(0.1, 0.1, 0.1, 1.0);
    mat.diffuse = float4(1.0, 1.0, 1.0, 1.0);
    mat.specular = float4(0.5, 0.5, 0.5, 1.0);
    mat.shininess = 64.0;
    mat.hasTexture = true;
    
    float3 litColor = phongLighting(
        in.worldPos,
        worldNormal,
        viewDir,
        uniforms.lights,
        uniforms.lightCount,
        mat,
        diffuseColor
    );
    
    return float4(litColor, diffuseColor.a);
}
```

---

## 9. Compute Shaders for GPGPU

### 9.1 Image Processing Pipeline

```swift
// Compute/ImageProcessor.swift

import Metal
import CoreImage

class MetalImageProcessor {
    let device: MTLDevice
    let commandQueue: MTLCommandQueue
    var blurPipeline: MTLComputePipelineState!
    var edgeDetectionPipeline: MTLComputePipelineState!
    
    init(device: MTLDevice, library: MTLLibrary) throws {
        self.device = device
        self.commandQueue = device.makeCommandQueue()!
        
        // Setup compute pipelines
        guard let blurFunction = library.makeFunction(name: "gaussian_blur"),
              let edgeFunction = library.makeFunction(name: "sobel_edge_detection") else {
            throw ComputeError.functionNotFound
        }
        
        blurPipeline = try device.makeComputePipelineState(function: blurFunction)
        edgeDetectionPipeline = try device.makeComputePipelineState(function: edgeFunction)
    }
    
    // MARK: - Apply Gaussian Blur
    func applyBlur(to inputTexture: MTLTexture) -> MTLTexture? {
        guard let outputTexture = makeMatchingTexture(for: inputTexture),
              let commandBuffer = commandQueue.makeCommandBuffer(),
              let encoder = commandBuffer.makeComputeCommandEncoder() else {
            return nil
        }
        
        encoder.setComputePipelineState(blurPipeline)
        encoder.setTexture(inputTexture, index: 0)
        encoder.setTexture(outputTexture, index: 1)
        
        // Calculate thread groups
        let threadGroupSize = MTLSize(width: 16, height: 16, depth: 1)
        let threadGroups = MTLSize(
            width: (inputTexture.width + threadGroupSize.width - 1) / threadGroupSize.width,
            height: (inputTexture.height + threadGroupSize.height - 1) / threadGroupSize.height,
            depth: 1
        )
        
        encoder.dispatchThreadgroups(threadGroups, threadsPerThreadgroup: threadGroupSize)
        encoder.endEncoding()
        
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        return outputTexture
    }
    
    // MARK: - Apply Edge Detection
    func applyEdgeDetection(to inputTexture: MTLTexture) -> MTLTexture? {
        guard let outputTexture = makeMatchingTexture(for: inputTexture),
              let commandBuffer = commandQueue.makeCommandBuffer(),
              let encoder = commandBuffer.makeComputeCommandEncoder() else {
            return nil
        }
        
        encoder.setComputePipelineState(edgeDetectionPipeline)
        encoder.setTexture(inputTexture, index: 0)
        encoder.setTexture(outputTexture, index: 1)
        
        // Use non-uniform threadgroups (more efficient on Apple GPU)
        let w = blurPipeline.threadExecutionWidth
        let h = blurPipeline.maxTotalThreadsPerThreadgroup / w
        let threadGroupSize = MTLSize(width: w, height: h, depth: 1)
        
        encoder.dispatchThreads(
            MTLSize(width: inputTexture.width, height: inputTexture.height, depth: 1),
            threadsPerThreadgroup: threadGroupSize
        )
        
        encoder.endEncoding()
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        return outputTexture
    }
    
    // MARK: - Chained Processing Pipeline
    func processImage(
        _ texture: MTLTexture,
        operations: [ImageOperation]
    ) -> MTLTexture? {
        guard let commandBuffer = commandQueue.makeCommandBuffer() else { return nil }
        
        var currentTexture = texture
        
        for operation in operations {
            guard let outputTexture = makeMatchingTexture(for: currentTexture),
                  let encoder = commandBuffer.makeComputeCommandEncoder() else {
                return nil
            }
            
            switch operation {
            case .blur:
                encoder.setComputePipelineState(blurPipeline)
            case .edgeDetection:
                encoder.setComputePipelineState(edgeDetectionPipeline)
            }
            
            encoder.setTexture(currentTexture, index: 0)
            encoder.setTexture(outputTexture, index: 1)
            
            let threadGroupSize = MTLSize(width: 16, height: 16, depth: 1)
            let threadGroups = MTLSize(
                width: (currentTexture.width + 15) / 16,
                height: (currentTexture.height + 15) / 16,
                depth: 1
            )
            encoder.dispatchThreadgroups(threadGroups, threadsPerThreadgroup: threadGroupSize)
            encoder.endEncoding()
            
            currentTexture = outputTexture
        }
        
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        return currentTexture
    }
    
    // MARK: - Helper
    private func makeMatchingTexture(for texture: MTLTexture) -> MTLTexture? {
        let descriptor = MTLTextureDescriptor.texture2DDescriptor(
            pixelFormat: texture.pixelFormat,
            width: texture.width,
            height: texture.height,
            mipmapped: false
        )
        descriptor.usage = [.shaderRead, .shaderWrite]
        return device.makeTexture(descriptor: descriptor)
    }
    
    enum ImageOperation {
        case blur
        case edgeDetection
    }
    
    enum ComputeError: Error {
        case functionNotFound
    }
}
```

---

## 10. Metal Performance Shaders (MPS)

### 10.1 Image Filters ด้วย MPS

```swift
// MPS/MPSImageFilters.swift

import Metal
import MetalPerformanceShaders

class MPSImageFilters {
    let device: MTLDevice
    let commandQueue: MTLCommandQueue
    
    init(device: MTLDevice) {
        self.device = device
        self.commandQueue = device.makeCommandQueue()!
    }
    
    // MARK: - Gaussian Blur (MPS version - ง่ายกว่า custom shader)
    func gaussianBlur(
        inputTexture: MTLTexture,
        sigma: Float = 10.0
    ) -> MTLTexture? {
        guard let outputTexture = makeOutputTexture(matching: inputTexture),
              let commandBuffer = commandQueue.makeCommandBuffer() else {
            return nil
        }
        
        let blur = MPSImageGaussianBlur(device: device, sigma: sigma)
        blur.encode(commandBuffer: commandBuffer,
                   sourceTexture: inputTexture,
                   destinationTexture: outputTexture)
        
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        return outputTexture
    }
    
    // MARK: - Sobel Edge Detection (MPS)
    func sobelEdgeDetection(inputTexture: MTLTexture) -> MTLTexture? {
        guard let outputTexture = makeOutputTexture(matching: inputTexture),
              let commandBuffer = commandQueue.makeCommandBuffer() else {
            return nil
        }
        
        let sobel = MPSImageSobel(device: device)
        sobel.encode(commandBuffer: commandBuffer,
                    sourceTexture: inputTexture,
                    destinationTexture: outputTexture)
        
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        return outputTexture
    }
    
    // MARK: - Histogram
    func calculateHistogram(for texture: MTLTexture) -> [UInt32]? {
        guard let commandBuffer = commandQueue.makeCommandBuffer() else { return nil }
        
        var histogramInfo = MPSImageHistogramInfo(
            numberOfHistogramEntries: 256,
            histogramForAlpha: false,
            minPixelValue: vector_float4(0, 0, 0, 0),
            maxPixelValue: vector_float4(1, 1, 1, 1)
        )
        
        let histogram = MPSImageHistogram(device: device, histogramInfo: &histogramInfo)
        let bufferLength = histogram.histogramSize(forSourceFormat: texture.pixelFormat)
        
        guard let histogramBuffer = device.makeBuffer(
            length: bufferLength,
            options: .storageModeShared
        ) else { return nil }
        
        histogram.encode(to: commandBuffer,
                        sourceTexture: texture,
                        histogram: histogramBuffer,
                        histogramOffset: 0)
        
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        let ptr = histogramBuffer.contents().bindMemory(to: UInt32.self, capacity: bufferLength / 4)
        return Array(UnsafeBufferPointer(start: ptr, count: bufferLength / 4))
    }
    
    // MARK: - MPS Neural Network (Core ML Integration)
    func runMLInference(inputTexture: MTLTexture, model: MPSCNNConvolution) -> MTLTexture? {
        // MPS สามารถรัน neural network layers โดยตรงบน GPU
        // ปกติใช้ Core ML ผ่าน Metal ได้ง่ายกว่า
        return nil
    }
    
    private func makeOutputTexture(matching input: MTLTexture) -> MTLTexture? {
        let descriptor = MTLTextureDescriptor.texture2DDescriptor(
            pixelFormat: input.pixelFormat,
            width: input.width,
            height: input.height,
            mipmapped: false
        )
        descriptor.usage = [.shaderRead, .shaderWrite, .renderTarget]
        return device.makeTexture(descriptor: descriptor)
    }
}
```

---

## 11. MetalKit Integration

### 11.1 MTKView Delegate Pattern แบบสมบูรณ์

```swift
// MetalKit/FullMetalRenderer.swift

import Metal
import MetalKit
import simd

class FullMetalRenderer: NSObject, MTKViewDelegate {
    
    // MARK: - Core Metal
    let device: MTLDevice
    let commandQueue: MTLCommandQueue
    
    // MARK: - Pipelines
    var mainPipeline: MTLRenderPipelineState!
    var shadowPipeline: MTLRenderPipelineState!
    var depthState: MTLDepthStencilState!
    
    // MARK: - Geometry
    var cubeRenderer: CubeRenderer!
    var instancedRenderer: InstancedRenderer!
    
    // MARK: - Textures
    var diffuseTexture: MTLTexture?
    var shadowMapTexture: MTLTexture?
    var renderTargetTexture: MTLTexture?
    
    // MARK: - Buffers (Triple buffering)
    var uniformBuffers: [MTLBuffer] = []
    var currentBufferIndex = 0
    let maxFramesInFlight = 3
    let frameSemaphore: DispatchSemaphore
    
    // MARK: - Camera
    var camera = Camera()
    var time: Float = 0
    
    init?(view: MTKView) {
        guard let device = MTLCreateSystemDefaultDevice(),
              let commandQueue = device.makeCommandQueue() else {
            return nil
        }
        
        self.device = device
        self.commandQueue = commandQueue
        self.frameSemaphore = DispatchSemaphore(value: maxFramesInFlight)
        
        super.init()
        
        setupView(view)
        setupPipelines(view: view)
        setupGeometry()
        setupUniformBuffers()
        setupTextures()
    }
    
    // MARK: - Setup
    func setupView(_ view: MTKView) {
        view.device = device
        view.colorPixelFormat = .bgra8Unorm_srgb
        view.depthStencilPixelFormat = .depth32Float
        view.sampleCount = 1
        view.clearColor = MTLClearColor(red: 0.05, green: 0.05, blue: 0.1, alpha: 1.0)
        view.delegate = self
    }
    
    func setupPipelines(view: MTKView) {
        guard let library = device.makeDefaultLibrary() else { return }
        
        let pipelineManager = PipelineManager(device: device)
        
        mainPipeline = try? pipelineManager.makeBasicRenderPipeline(
            library: library,
            pixelFormat: view.colorPixelFormat
        )
        
        depthState = pipelineManager.makeDepthStencilState()
    }
    
    func setupGeometry() {
        cubeRenderer = CubeRenderer(device: device)
        instancedRenderer = InstancedRenderer(device: device, instanceCount: 50)
    }
    
    func setupUniformBuffers() {
        for _ in 0..<maxFramesInFlight {
            let buffer = device.makeBuffer(
                length: MemoryLayout<SceneUniforms>.stride,
                options: .storageModeShared
            )!
            uniformBuffers.append(buffer)
        }
    }
    
    func setupTextures() {
        let textureManager = TextureManager(device: device)
        diffuseTexture = textureManager.makeCheckerboardTexture()
    }
    
    // MARK: - MTKViewDelegate
    func mtkView(_ view: MTKView, drawableSizeWillChange size: CGSize) {
        camera.aspectRatio = Float(size.width / size.height)
        
        // Recreate render targets
        let textureManager = TextureManager(device: device)
        renderTargetTexture = textureManager.makeRenderTargetTexture(
            width: Int(size.width),
            height: Int(size.height)
        )
    }
    
    func draw(in view: MTKView) {
        // Wait for available buffer
        frameSemaphore.wait()
        
        time += 1.0 / 60.0
        currentBufferIndex = (currentBufferIndex + 1) % maxFramesInFlight
        
        guard let commandBuffer = commandQueue.makeCommandBuffer(),
              let renderPassDescriptor = view.currentRenderPassDescriptor,
              let drawable = view.currentDrawable else {
            frameSemaphore.signal()
            return
        }
        
        // Update uniforms
        updateUniforms()
        
        // Update instances
        instancedRenderer.updateInstances(time: time)
        
        // Main render pass
        guard let encoder = commandBuffer.makeRenderCommandEncoder(
            descriptor: renderPassDescriptor
        ) else {
            frameSemaphore.signal()
            return
        }
        
        encoder.setRenderPipelineState(mainPipeline)
        encoder.setDepthStencilState(depthState)
        encoder.setCullMode(.back)
        encoder.setFrontFacing(.counterClockwise)
        
        // Set uniform buffer
        encoder.setVertexBuffer(
            uniformBuffers[currentBufferIndex],
            offset: 0, index: 1
        )
        
        // Set texture
        if let texture = diffuseTexture {
            encoder.setFragmentTexture(texture, index: 0)
        }
        
        // Draw cube
        cubeRenderer.draw(encoder: encoder)
        
        encoder.endEncoding()
        
        commandBuffer.present(drawable)
        
        // Signal semaphore when GPU done
        commandBuffer.addCompletedHandler { [weak self] _ in
            self?.frameSemaphore.signal()
        }
        
        commandBuffer.commit()
    }
    
    // MARK: - Update Uniforms
    func updateUniforms() {
        let uniformsPtr = uniformBuffers[currentBufferIndex].contents()
            .bindMemory(to: SceneUniforms.self, capacity: 1)
        
        let model = float4x4(rotationY: time * 0.3) * float4x4(rotationX: sin(time * 0.2) * 0.3)
        
        uniformsPtr.pointee.modelMatrix = model
        uniformsPtr.pointee.viewMatrix = camera.viewMatrix
        uniformsPtr.pointee.projectionMatrix = camera.projectionMatrix
        uniformsPtr.pointee.normalMatrix = model.normalMatrix
        uniformsPtr.pointee.cameraPosition = camera.position
        uniformsPtr.pointee.time = time
    }
}
```

---

## 12. SwiftUI Metal Integration กับ TimelineView

```swift
// SwiftUI/MetalSwiftUIView.swift

import SwiftUI
import MetalKit

// MARK: - Metal View Wrapper สำหรับ SwiftUI
struct MetalViewRepresentable: UIViewRepresentable {
    let renderer: FullMetalRenderer?
    
    func makeUIView(context: Context) -> MTKView {
        let view = MTKView()
        view.preferredFramesPerSecond = 120
        view.enableSetNeedsDisplay = false
        view.isPaused = false
        return view
    }
    
    func updateUIView(_ uiView: MTKView, context: Context) {
        // Update renderer if needed
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator()
    }
    
    class Coordinator {
        var renderer: FullMetalRenderer?
    }
}

// MARK: - Metal View ด้วย TimelineView
struct MetalTimelineView: View {
    @State private var renderer: FullMetalRenderer?
    @State private var mtkView = MTKView()
    
    var body: some View {
        TimelineView(.animation) { timeline in
            MetalCanvas(time: timeline.date.timeIntervalSinceReferenceDate)
        }
        .ignoresSafeArea()
    }
}

// MARK: - Metal Canvas (ใช้กับ Canvas View)
struct MetalCanvas: View {
    let time: Double
    
    var body: some View {
        Canvas { context, size in
            // สำหรับ 2D Metal-accelerated drawing
            // ใช้ context.withCGContext สำหรับ Metal integration
        }
        .background(Color.black)
    }
}

// MARK: - Shader-based SwiftUI View (iOS 17+)
struct ShaderView: View {
    @State private var startDate = Date()
    
    var body: some View {
        TimelineView(.animation) { timeline in
            let time = Float(timeline.date.timeIntervalSince(startDate))
            
            Rectangle()
                .colorEffect(
                    ShaderLibrary.default.waveShader(
                        .float(time),
                        .float2(400, 300)
                    )
                )
        }
        .ignoresSafeArea()
    }
}

// MARK: - SwiftUI Shader Library (iOS 17+)
// Shaders/SwiftUIShaders.metal
/*
#include <metal_stdlib>
#include <SwiftUI/SwiftUI_Metal.h>
using namespace metal;

[[ stitchable ]] half4 waveShader(
    float2 pos,
    half4 color,
    float time,
    float2 size
) {
    float2 uv = pos / size;
    float wave = sin(uv.x * 10.0 + time * 3.0) * 0.5 + 0.5;
    float3 col = mix(
        float3(0.1, 0.1, 0.5),
        float3(0.5, 0.8, 1.0),
        wave * (1.0 - uv.y)
    );
    return half4(half3(col), 1.0);
}
*/
```

---

## 13. Performance Profiling กับ Metal Frame Debugger

### 13.1 Metal Performance Markers

```swift
// Performance/MetalProfiler.swift

import Metal

// MARK: - Performance Markers สำหรับ Metal Frame Debugger
class MetalProfiler {
    
    // เพิ่ม label ให้ objects เพื่อให้อ่านง่ายใน Frame Debugger
    static func setupLabels(
        device: MTLDevice,
        commandQueue: MTLCommandQueue,
        buffer: MTLBuffer,
        texture: MTLTexture,
        pipeline: MTLRenderPipelineState
    ) {
        commandQueue.label = "Main Command Queue"
        buffer.label = "Vertex Buffer"
        texture.label = "Diffuse Texture"
        pipeline.label = "Main Render Pipeline"
    }
    
    // ใช้ Debug Groups ใน Command Buffer
    static func encodeWithDebugGroups(
        commandBuffer: MTLCommandBuffer,
        renderPassDescriptor: MTLRenderPassDescriptor,
        closure: (MTLRenderCommandEncoder) -> Void
    ) {
        commandBuffer.pushDebugGroup("Main Render Pass")
        
        guard let encoder = commandBuffer.makeRenderCommandEncoder(
            descriptor: renderPassDescriptor
        ) else {
            commandBuffer.popDebugGroup()
            return
        }
        
        encoder.pushDebugGroup("Setup State")
        // Setup pipeline, buffers...
        encoder.popDebugGroup()
        
        encoder.pushDebugGroup("Draw Objects")
        closure(encoder)
        encoder.popDebugGroup()
        
        encoder.endEncoding()
        commandBuffer.popDebugGroup()
    }
    
    // GPU Counter สำหรับ Performance Measurement
    static func measureGPUPerformance(
        device: MTLDevice,
        commandQueue: MTLCommandQueue,
        block: (MTLCommandBuffer) -> Void
    ) -> Double {
        guard let commandBuffer = commandQueue.makeCommandBuffer() else {
            return 0
        }
        
        let start = CFAbsoluteTimeGetCurrent()
        
        block(commandBuffer)
        
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        return CFAbsoluteTimeGetCurrent() - start
    }
}

// MARK: - Performance Tips
/*
 GPU Performance Best Practices:
 
 1. Triple Buffering
    - ใช้ semaphore เพื่อป้องกัน CPU รอ GPU
    - เตรียม 3 copies ของ uniform buffers
 
 2. Batch Draw Calls
    - ใช้ Instanced Rendering แทนการ draw แยกกันหลายครั้ง
    - Minimize state changes
 
 3. Minimize CPU-GPU synchronization
    - หลีกเลี่ยง waitUntilCompleted() ใน render loop
    - ใช้ completion handlers แทน
 
 4. Use Private Storage Mode
    - Vertex buffers ที่ไม่เปลี่ยนแปลง ให้ใช้ .storageModePrivate
    - ดีกว่า .storageModeShared สำหรับ read-only data
 
 5. Tile-Based Deferred Rendering (TBDR)
    - Apple GPU ใช้ TBDR architecture
    - ออกแบบ render passes ให้ใช้ tile memory อย่างมีประสิทธิภาพ
    - หลีกเลี่ยงการ load/store ที่ไม่จำเป็น
 
 6. Reduce Overdraw
    - เรียง objects จาก front-to-back
    - ใช้ depth testing อย่างถูกต้อง
 
 7. Texture Compression
    - ใช้ ASTC format สำหรับ iOS
    - ลด bandwidth และ memory usage
*/
```

---

## 14. Complete Metal App: Interactive Particle System

### 14.1 Particle System Class หลัก

```swift
// ParticleSystem/ParticleSystemRenderer.swift

import Metal
import MetalKit
import simd
import UIKit

class ParticleSystemRenderer: NSObject, MTKViewDelegate {
    
    // MARK: - Metal Core
    let device: MTLDevice
    let commandQueue: MTLCommandQueue
    
    // MARK: - Pipelines
    var renderPipeline: MTLRenderPipelineState!
    var computePipeline: MTLComputePipelineState!
    
    // MARK: - Particle Data
    struct ParticleConfig {
        var maxParticles: Int = 50000
        var emissionRate: Float = 100  // particles per second
        var gravity: SIMD2<Float> = SIMD2<Float>(0, -0.5)
        var particleLifetime: Float = 3.0
    }
    
    var config = ParticleConfig()
    var particleBuffer: MTLBuffer!
    var particleCount: Int = 0
    
    // MARK: - Uniforms
    var uniformBuffer: MTLBuffer!
    
    struct ParticleUniforms {
        var deltaTime: Float
        var time: Float
        var gravity: SIMD2<Float>
        var emitterPosition: SIMD2<Float>
        var particleCount: UInt32
        var projectionMatrix: float4x4
    }
    
    // MARK: - Touch Interaction
    var touchPosition: SIMD2<Float> = .zero
    var isTouching = false
    
    // MARK: - Timing
    var lastTime: CFTimeInterval = 0
    var time: Float = 0
    
    // MARK: - Initialization
    init?(view: MTKView) {
        guard let device = MTLCreateSystemDefaultDevice(),
              let commandQueue = device.makeCommandQueue() else {
            return nil
        }
        self.device = device
        self.commandQueue = commandQueue
        
        super.init()
        
        setupView(view)
        
        do {
            try setupPipelines(library: device.makeDefaultLibrary()!)
        } catch {
            print("Pipeline error: \(error)")
            return nil
        }
        
        setupBuffers()
        initializeParticles()
    }
    
    func setupView(_ view: MTKView) {
        view.device = device
        view.colorPixelFormat = .bgra8Unorm
        view.clearColor = MTLClearColor(red: 0.02, green: 0.02, blue: 0.05, alpha: 1.0)
        view.delegate = self
        view.preferredFramesPerSecond = 120
        view.isPaused = false
        view.enableSetNeedsDisplay = false
    }
    
    func setupPipelines(library: MTLLibrary) throws {
        // Compute Pipeline (particle update)
        guard let updateFunction = library.makeFunction(name: "update_particles") else {
            throw PipelineSetupError.functionNotFound("update_particles")
        }
        computePipeline = try device.makeComputePipelineState(function: updateFunction)
        
        // Render Pipeline (particle draw)
        guard let vertexFunction = library.makeFunction(name: "particle_vertex"),
              let fragmentFunction = library.makeFunction(name: "particle_fragment") else {
            throw PipelineSetupError.functionNotFound("particle shaders")
        }
        
        let descriptor = MTLRenderPipelineDescriptor()
        descriptor.vertexFunction = vertexFunction
        descriptor.fragmentFunction = fragmentFunction
        descriptor.colorAttachments[0].pixelFormat = .bgra8Unorm
        
        // Additive blending สำหรับ glowing particles
        descriptor.colorAttachments[0].isBlendingEnabled = true
        descriptor.colorAttachments[0].rgbBlendOperation = .add
        descriptor.colorAttachments[0].alphaBlendOperation = .add
        descriptor.colorAttachments[0].sourceRGBBlendFactor = .sourceAlpha
        descriptor.colorAttachments[0].destinationRGBBlendFactor = .one  // Additive!
        descriptor.colorAttachments[0].sourceAlphaBlendFactor = .sourceAlpha
        descriptor.colorAttachments[0].destinationAlphaBlendFactor = .one
        
        renderPipeline = try device.makeRenderPipelineState(descriptor: descriptor)
    }
    
    func setupBuffers() {
        // Particle buffer (shared mode for CPU/GPU access on initialization)
        let particleSize = MemoryLayout<ParticleSystem_MSL.Particle>.stride
        particleBuffer = device.makeBuffer(
            length: config.maxParticles * particleSize,
            options: .storageModeShared
        )
        particleBuffer.label = "Particle Buffer"
        
        uniformBuffer = device.makeBuffer(
            length: MemoryLayout<ParticleUniforms>.stride,
            options: .storageModeShared
        )
        uniformBuffer.label = "Particle Uniforms"
    }
    
    // MSL Particle struct mirrored in Swift
    enum ParticleSystem_MSL {
        struct Particle {
            var position: SIMD2<Float>
            var velocity: SIMD2<Float>
            var color: SIMD4<Float>
            var life: Float
            var maxLife: Float
            var size: Float
            var rotation: Float
        }
    }
    
    func initializeParticles() {
        let ptr = particleBuffer.contents().bindMemory(
            to: ParticleSystem_MSL.Particle.self,
            capacity: config.maxParticles
        )
        
        for i in 0..<config.maxParticles {
            ptr[i] = ParticleSystem_MSL.Particle(
                position: SIMD2<Float>(0, -0.5),
                velocity: SIMD2<Float>(0, 0),
                color: SIMD4<Float>(1, 0.5, 0, 1),
                life: -Float.random(in: 0...config.particleLifetime),  // Stagger spawn
                maxLife: config.particleLifetime,
                size: 0.02,
                rotation: 0
            )
        }
        
        particleCount = config.maxParticles
    }
    
    // MARK: - Touch Handling
    func touchMoved(to point: CGPoint, in view: MTKView) {
        let normalized = SIMD2<Float>(
            Float(point.x / view.bounds.width) * 2 - 1,
            1 - Float(point.y / view.bounds.height) * 2
        )
        touchPosition = normalized
        isTouching = true
    }
    
    func touchEnded() {
        isTouching = false
    }
    
    // MARK: - MTKViewDelegate
    func mtkView(_ view: MTKView, drawableSizeWillChange size: CGSize) {}
    
    func draw(in view: MTKView) {
        let currentTime = CACurrentMediaTime()
        let deltaTime = Float(min(currentTime - lastTime, 0.033))  // Cap at 33ms
        lastTime = currentTime
        time += deltaTime
        
        // Update uniforms
        let uniformsPtr = uniformBuffer.contents().bindMemory(
            to: ParticleUniforms.self, capacity: 1
        )
        
        let aspectRatio = Float(view.drawableSize.width / view.drawableSize.height)
        uniformsPtr.pointee = ParticleUniforms(
            deltaTime: deltaTime,
            time: time,
            gravity: isTouching ? SIMD2<Float>(0, 0.2) : config.gravity,
            emitterPosition: isTouching ? touchPosition : SIMD2<Float>(0, -0.8),
            particleCount: UInt32(particleCount),
            projectionMatrix: float4x4(
                orthographicLeft: -aspectRatio,
                right: aspectRatio,
                bottom: -1,
                top: 1,
                near: -1,
                far: 1
            )
        )
        
        guard let commandBuffer = commandQueue.makeCommandBuffer() else { return }
        commandBuffer.label = "Particle Frame"
        
        // 1. Compute Pass: Update particles
        if let computeEncoder = commandBuffer.makeComputeCommandEncoder() {
            computeEncoder.label = "Particle Update"
            computeEncoder.setComputePipelineState(computePipeline)
            computeEncoder.setBuffer(particleBuffer, offset: 0, index: 0)
            
            var dt = deltaTime
            var grav = uniformsPtr.pointee.gravity
            var t = time
            computeEncoder.setBytes(&dt, length: MemoryLayout<Float>.size, index: 1)
            computeEncoder.setBytes(&grav, length: MemoryLayout<SIMD2<Float>>.size, index: 2)
            computeEncoder.setBytes(&t, length: MemoryLayout<Float>.size, index: 3)
            
            let threadsPerGroup = min(
                computePipeline.maxTotalThreadsPerThreadgroup,
                particleCount
            )
            let groups = (particleCount + threadsPerGroup - 1) / threadsPerGroup
            
            computeEncoder.dispatchThreadgroups(
                MTLSize(width: groups, height: 1, depth: 1),
                threadsPerThreadgroup: MTLSize(width: threadsPerGroup, height: 1, depth: 1)
            )
            computeEncoder.endEncoding()
        }
        
        // 2. Render Pass: Draw particles
        guard let renderPass = view.currentRenderPassDescriptor,
              let drawable = view.currentDrawable else {
            commandBuffer.commit()
            return
        }
        
        if let renderEncoder = commandBuffer.makeRenderCommandEncoder(descriptor: renderPass) {
            renderEncoder.label = "Particle Render"
            renderEncoder.setRenderPipelineState(renderPipeline)
            renderEncoder.setVertexBuffer(particleBuffer, offset: 0, index: 0)
            renderEncoder.setVertexBuffer(uniformBuffer, offset: 0, index: 1)
            
            // Draw as points (each particle = 1 point)
            renderEncoder.drawPrimitives(
                type: .point,
                vertexStart: 0,
                vertexCount: particleCount
            )
            
            renderEncoder.endEncoding()
        }
        
        commandBuffer.present(drawable)
        commandBuffer.commit()
    }
    
    enum PipelineSetupError: Error {
        case functionNotFound(String)
    }
}

// MARK: - SwiftUI Integration
import SwiftUI

struct ParticleSystemView: UIViewRepresentable {
    func makeUIView(context: Context) -> MTKView {
        let view = MTKView()
        let renderer = ParticleSystemRenderer(view: view)
        context.coordinator.renderer = renderer
        
        // Setup gesture recognizers
        let panGesture = UIPanGestureRecognizer(
            target: context.coordinator,
            action: #selector(Coordinator.handlePan(_:))
        )
        view.addGestureRecognizer(panGesture)
        
        return view
    }
    
    func updateUIView(_ uiView: MTKView, context: Context) {}
    
    func makeCoordinator() -> Coordinator { Coordinator() }
    
    class Coordinator: NSObject {
        var renderer: ParticleSystemRenderer?
        
        @objc func handlePan(_ gesture: UIPanGestureRecognizer) {
            guard let view = gesture.view else { return }
            
            switch gesture.state {
            case .began, .changed:
                renderer?.touchMoved(to: gesture.location(in: view), in: view as! MTKView)
            case .ended, .cancelled:
                renderer?.touchEnded()
            default:
                break
            }
        }
    }
}

// App Entry Point
struct ParticleApp: App {
    var body: some Scene {
        WindowGroup {
            ParticleSystemView()
                .ignoresSafeArea()
                .statusBarHidden(true)
        }
    }
}
```

---

## 15. แบบฝึกหัดและเฉลย

### แบบฝึกหัดที่ 1: สร้าง Spinning Cube ด้วย Lighting

**โจทย์:** สร้าง rotating cube ที่มี Phong lighting จาก point light ที่โคจรรอบ cube

**เฉลย:**

```swift
// Exercise 1 Solution

class SpinningCubeScene: NSObject, MTKViewDelegate {
    let device: MTLDevice
    let commandQueue: MTLCommandQueue
    var pipeline: MTLRenderPipelineState!
    var depthState: MTLDepthStencilState!
    var cubeRenderer: CubeRenderer!
    var uniformBuffer: MTLBuffer!
    var fragmentUniformBuffer: MTLBuffer!
    var time: Float = 0
    var camera = Camera()
    
    struct LightUniforms {
        var lightPosition: SIMD3<Float>
        var lightColor: SIMD4<Float>
        var ambientColor: SIMD4<Float>
    }
    
    init?(view: MTKView) {
        guard let device = MTLCreateSystemDefaultDevice(),
              let queue = device.makeCommandQueue() else { return nil }
        self.device = device
        self.commandQueue = queue
        super.init()
        
        view.device = device
        view.colorPixelFormat = .bgra8Unorm
        view.depthStencilPixelFormat = .depth32Float
        view.clearColor = MTLClearColor(red: 0.1, green: 0.1, blue: 0.15, alpha: 1.0)
        view.delegate = self
        
        setupPipeline(view: view)
        setupBuffers()
        
        cubeRenderer = CubeRenderer(device: device)
        camera.position = SIMD3<Float>(0, 1, 3)
        camera.aspectRatio = Float(view.bounds.width / view.bounds.height)
    }
    
    func setupPipeline(view: MTKView) {
        guard let library = device.makeDefaultLibrary() else { return }
        
        let descriptor = MTLRenderPipelineDescriptor()
        descriptor.vertexFunction = library.makeFunction(name: "vertex_main")
        descriptor.fragmentFunction = library.makeFunction(name: "fragment_main")
        descriptor.colorAttachments[0].pixelFormat = view.colorPixelFormat
        descriptor.depthAttachmentPixelFormat = view.depthStencilPixelFormat
        
        let vd = PipelineManager(device: device).makeVertexDescriptor()
        descriptor.vertexDescriptor = vd
        
        pipeline = try? device.makeRenderPipelineState(descriptor: descriptor)
        depthState = PipelineManager(device: device).makeDepthStencilState()
    }
    
    func setupBuffers() {
        uniformBuffer = device.makeBuffer(
            length: MemoryLayout<SceneUniforms>.stride,
            options: .storageModeShared
        )
        fragmentUniformBuffer = device.makeBuffer(
            length: MemoryLayout<LightUniforms>.stride,
            options: .storageModeShared
        )
    }
    
    func mtkView(_ view: MTKView, drawableSizeWillChange size: CGSize) {
        camera.aspectRatio = Float(size.width / size.height)
    }
    
    func draw(in view: MTKView) {
        time += 1.0 / 60.0
        
        // Animate light in orbit
        let lightAngle = time * 1.5
        let lightPos = SIMD3<Float>(cos(lightAngle) * 3, 2, sin(lightAngle) * 3)
        
        // Update scene uniforms
        let scenePtr = uniformBuffer.contents().bindMemory(to: SceneUniforms.self, capacity: 1)
        let model = float4x4(rotationY: time) * float4x4(rotationX: time * 0.5)
        scenePtr.pointee.modelMatrix = model
        scenePtr.pointee.viewMatrix = camera.viewMatrix
        scenePtr.pointee.projectionMatrix = camera.projectionMatrix
        scenePtr.pointee.normalMatrix = model.normalMatrix
        scenePtr.pointee.cameraPosition = camera.position
        scenePtr.pointee.time = time
        
        // Update light uniforms
        let lightPtr = fragmentUniformBuffer.contents().bindMemory(
            to: LightUniforms.self, capacity: 1
        )
        lightPtr.pointee.lightPosition = lightPos
        lightPtr.pointee.lightColor = SIMD4<Float>(1.0, 0.9, 0.7, 1.0)
        lightPtr.pointee.ambientColor = SIMD4<Float>(0.1, 0.1, 0.15, 1.0)
        
        guard let commandBuffer = commandQueue.makeCommandBuffer(),
              let pass = view.currentRenderPassDescriptor,
              let drawable = view.currentDrawable,
              let encoder = commandBuffer.makeRenderCommandEncoder(descriptor: pass) else {
            return
        }
        
        encoder.setRenderPipelineState(pipeline)
        encoder.setDepthStencilState(depthState)
        encoder.setCullMode(.back)
        encoder.setVertexBuffer(uniformBuffer, offset: 0, index: 1)
        encoder.setFragmentBuffer(fragmentUniformBuffer, offset: 0, index: 0)
        
        cubeRenderer.draw(encoder: encoder)
        
        encoder.endEncoding()
        commandBuffer.present(drawable)
        commandBuffer.commit()
    }
}
```

### แบบฝึกหัดที่ 2: Custom Image Filter

**โจทย์:** สร้าง sepia tone filter ด้วย compute shader

**เฉลย:**

```metal
// Exercise 2 Solution - Sepia.metal

kernel void sepia_filter(
    texture2d<float, access::read> input [[texture(0)]],
    texture2d<float, access::write> output [[texture(1)]],
    uint2 gid [[thread_position_in_grid]]
) {
    if (gid.x >= input.get_width() || gid.y >= input.get_height()) return;
    
    float4 color = input.read(gid);
    
    float r = color.r * 0.393 + color.g * 0.769 + color.b * 0.189;
    float g = color.r * 0.349 + color.g * 0.686 + color.b * 0.168;
    float b = color.r * 0.272 + color.g * 0.534 + color.b * 0.131;
    
    output.write(float4(
        clamp(r, 0.0, 1.0),
        clamp(g, 0.0, 1.0),
        clamp(b, 0.0, 1.0),
        color.a
    ), gid);
}
```

```swift
// Swift side for Exercise 2
extension MetalImageProcessor {
    func applySepia(to inputTexture: MTLTexture) -> MTLTexture? {
        guard let outputTexture = makeOutputTexture(matching: inputTexture),
              let commandBuffer = commandQueue.makeCommandBuffer() else {
            return nil
        }
        
        guard let library = device.makeDefaultLibrary(),
              let sepiaFunction = library.makeFunction(name: "sepia_filter"),
              let sepiaPipeline = try? device.makeComputePipelineState(function: sepiaFunction) else {
            return nil
        }
        
        guard let encoder = commandBuffer.makeComputeCommandEncoder() else {
            return nil
        }
        
        encoder.setComputePipelineState(sepiaPipeline)
        encoder.setTexture(inputTexture, index: 0)
        encoder.setTexture(outputTexture, index: 1)
        
        let w = sepiaPipeline.threadExecutionWidth
        let h = sepiaPipeline.maxTotalThreadsPerThreadgroup / w
        
        encoder.dispatchThreads(
            MTLSize(width: inputTexture.width, height: inputTexture.height, depth: 1),
            threadsPerThreadgroup: MTLSize(width: w, height: h, depth: 1)
        )
        
        encoder.endEncoding()
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        return outputTexture
    }
}
```

### แบบฝึกหัดที่ 3: Procedural Terrain

**โจทย์:** สร้าง terrain mesh ที่ generate จาก noise function ใน vertex shader

**เฉลย:**

```metal
// Exercise 3 Solution - TerrainShaders.metal

// Simple noise function
float hash(float2 p) {
    return fract(sin(dot(p, float2(127.1, 311.7))) * 43758.5453);
}

float noise(float2 p) {
    float2 i = floor(p);
    float2 f = fract(p);
    float2 u = f * f * (3.0 - 2.0 * f);
    
    return mix(
        mix(hash(i + float2(0, 0)), hash(i + float2(1, 0)), u.x),
        mix(hash(i + float2(0, 1)), hash(i + float2(1, 1)), u.x),
        u.y
    );
}

float fbm(float2 p) {
    float value = 0.0;
    float amplitude = 0.5;
    float frequency = 1.0;
    
    for (int i = 0; i < 6; i++) {
        value += amplitude * noise(p * frequency);
        amplitude *= 0.5;
        frequency *= 2.0;
    }
    
    return value;
}

struct TerrainVertexIn {
    float2 gridPos [[attribute(0)]];
};

struct TerrainVertexOut {
    float4 position [[position]];
    float3 worldPos;
    float3 normal;
    float height;
};

vertex TerrainVertexOut terrain_vertex(
    TerrainVertexIn in [[stage_in]],
    constant Uniforms& uniforms [[buffer(1)]]
) {
    float2 p = in.gridPos;
    float height = fbm(p * 3.0) * 2.0;
    
    // Calculate normal using finite differences
    float eps = 0.01;
    float hL = fbm((p - float2(eps, 0)) * 3.0) * 2.0;
    float hR = fbm((p + float2(eps, 0)) * 3.0) * 2.0;
    float hD = fbm((p - float2(0, eps)) * 3.0) * 2.0;
    float hU = fbm((p + float2(0, eps)) * 3.0) * 2.0;
    
    float3 normal = normalize(float3(hL - hR, 2.0 * eps, hD - hU));
    
    float3 worldPos = float3(p.x * 10.0, height, p.y * 10.0);
    
    TerrainVertexOut out;
    out.position = uniforms.projectionMatrix * uniforms.viewMatrix * float4(worldPos, 1.0);
    out.worldPos = worldPos;
    out.normal = normal;
    out.height = height;
    
    return out;
}

fragment float4 terrain_fragment(TerrainVertexOut in [[stage_in]]) {
    // Color based on height
    float3 lowColor = float3(0.2, 0.6, 0.2);   // Green (grass)
    float3 midColor = float3(0.5, 0.4, 0.3);   // Brown (dirt)
    float3 highColor = float3(0.9, 0.9, 0.95); // White (snow)
    
    float3 color;
    float h = in.height / 2.0;  // Normalize
    
    if (h < 0.4) {
        color = mix(lowColor, midColor, h / 0.4);
    } else {
        color = mix(midColor, highColor, (h - 0.4) / 0.6);
    }
    
    // Simple directional lighting
    float3 lightDir = normalize(float3(1, 2, 1));
    float diffuse = max(dot(in.normal, lightDir), 0.2);
    
    return float4(color * diffuse, 1.0);
}
```

---

## สรุป

Metal เป็น API ที่ทรงพลังมากสำหรับ GPU programming บน Apple platforms บทนี้ครอบคลุม:

1. **Metal Architecture** - Device, CommandQueue, CommandBuffer
2. **Render Pipeline** - Vertex/Fragment shaders, Pipeline states
3. **Metal Shading Language (MSL)** - การเขียน shaders สำหรับ 3D graphics
4. **Buffers and Textures** - การจัดการ GPU memory
5. **3D Transformations** - Model/View/Projection matrices ด้วย simd
6. **Lighting** - Phong model ใน Metal shaders
7. **Compute Shaders** - Image processing และ particle systems
8. **Metal Performance Shaders** - Built-in high-performance filters
9. **Particle System** - Complete interactive app

### แนวทางการเรียนรู้ต่อไป:
- **Deferred Rendering** - สำหรับ many lights
- **Shadow Mapping** - Realistic shadows
- **Screen Space Ambient Occlusion (SSAO)** - Advanced lighting
- **Physically Based Rendering (PBR)** - Realistic materials
- **Ray Tracing** ด้วย Metal Ray Tracing API (A14+ chips)
- **MetalFX** - Upscaling และ temporal anti-aliasing
- **Core ML + Metal** - Neural network acceleration

### Useful Resources:
- Apple Developer Documentation: Metal
- Metal by Tutorials (raywenderlich.com)
- WWDC Metal Sessions
- Metal Sample Code จาก Apple

---

*จบ Part 82: MetalKit and GPU Programming*
