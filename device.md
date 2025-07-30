# DirectX 11 Device (디바이스)

## 디바이스란 무엇인가?

DirectX 11에서 **Device**는 그래픽 하드웨어(GPU)를 추상화한 객체입니다. 쉽게 말해, 컴퓨터의 그래픽 카드와 소통하기 위한 "통역사" 역할을 합니다.

게임이나 3D 애플리케이션을 만들 때, 우리는 화면에 무언가를 그리고 싶어합니다. 하지만 그래픽 카드에 직접 명령을 내리는 것은 매우 복잡하고 어렵습니다. 이때 Device가 중간에서 우리의 명령을 그래픽 카드가 이해할 수 있는 형태로 변환해줍니다.

## Device의 주요 역할

### 1. 리소스 생성
- **텍스처(Texture)**: 이미지나 색상 정보를 저장
- **버퍼(Buffer)**: 정점 데이터, 인덱스 데이터 등을 저장
- **셰이더(Shader)**: GPU에서 실행되는 작은 프로그램들

### 2. 파이프라인 상태 설정
그래픽 파이프라인의 각 단계에서 어떤 셰이더를 사용할지, 어떤 설정을 적용할지 결정합니다.

### 3. 렌더링 명령 실행
실제로 화면에 그리는 작업을 수행합니다.

## Device가 생성하는 주요 리소스들

Device의 가장 중요한 역할 중 하나는 GPU에서 사용할 다양한 리소스들을 생성하는 것입니다. 각 리소스는 특별한 목적과 용도를 가지고 있습니다.

### 1. 버퍼(Buffer) 리소스

#### 정점 버퍼(Vertex Buffer)
3D 모델의 정점 정보(위치, 색상, 텍스처 좌표 등)를 저장합니다.

```cpp
// 정점 데이터 구조체
struct Vertex {
    float x, y, z;          // 위치
    float r, g, b, a;       // 색상
    float u, v;             // 텍스처 좌표
};

// 정점 버퍼 생성
D3D11_BUFFER_DESC bufferDesc = {};
bufferDesc.Usage = D3D11_USAGE_DEFAULT;
bufferDesc.ByteWidth = sizeof(Vertex) * vertexCount;
bufferDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;

D3D11_SUBRESOURCE_DATA initData = {};
initData.pSysMem = vertices;  // 정점 배열

ID3D11Buffer* vertexBuffer;
device->CreateBuffer(&bufferDesc, &initData, &vertexBuffer);
```

**왜 필요한가?**
- GPU는 CPU와 다른 메모리 공간을 사용
- 정점 데이터를 GPU 메모리로 미리 올려놔야 빠른 처리 가능
- 매 프레임마다 데이터를 전송하는 것보다 훨씬 효율적

#### 인덱스 버퍼(Index Buffer)
정점들을 어떤 순서로 연결해서 삼각형을 만들지 정의합니다.

```cpp
// 인덱스 배열 (삼각형 2개로 사각형 만들기)
UINT indices[] = {
    0, 1, 2,    // 첫 번째 삼각형
    2, 3, 0     // 두 번째 삼각형
};

D3D11_BUFFER_DESC indexBufferDesc = {};
indexBufferDesc.Usage = D3D11_USAGE_DEFAULT;
indexBufferDesc.ByteWidth = sizeof(UINT) * indexCount;
indexBufferDesc.BindFlags = D3D11_BIND_INDEX_BUFFER;

D3D11_SUBRESOURCE_DATA indexData = {};
indexData.pSysMem = indices;

ID3D11Buffer* indexBuffer;
device->CreateBuffer(&indexBufferDesc, &indexData, &indexBuffer);
```

**왜 사용하는가?**
- 정점을 재사용할 수 있어 메모리 절약
- 복잡한 모델일수록 더 큰 효과

#### 상수 버퍼(Constant Buffer)
셰이더에서 사용할 상수 값들(변환 행렬, 라이트 정보 등)을 저장합니다.

```cpp
// 상수 버퍼에 들어갈 데이터 구조체
struct ConstantBufferData {
    DirectX::XMMATRIX world;        // 월드 변환 행렬
    DirectX::XMMATRIX view;         // 뷰 변환 행렬
    DirectX::XMMATRIX projection;   // 투영 변환 행렬
    DirectX::XMFLOAT4 lightColor;   // 조명 색상
};

D3D11_BUFFER_DESC constantBufferDesc = {};
constantBufferDesc.Usage = D3D11_USAGE_DYNAMIC;  // 자주 업데이트됨
constantBufferDesc.ByteWidth = sizeof(ConstantBufferData);
constantBufferDesc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
constantBufferDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;

ID3D11Buffer* constantBuffer;
device->CreateBuffer(&constantBufferDesc, nullptr, &constantBuffer);
```

**왜 필요한가?**
- 셰이더는 외부 데이터에 직접 접근할 수 없음
- 카메라 이동, 오브젝트 회전 등 매 프레임 변하는 값들을 전달
- GPU에서 효율적으로 접근 가능한 형태로 데이터 제공

### 2. 텍스처(Texture) 리소스

#### 2D 텍스처
이미지 데이터를 저장하여 3D 모델 표면에 입힙니다.

```cpp
D3D11_TEXTURE2D_DESC textureDesc = {};
textureDesc.Width = 512;
textureDesc.Height = 512;
textureDesc.MipLevels = 1;
textureDesc.ArraySize = 1;
textureDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;  // RGBA 8비트
textureDesc.SampleDesc.Count = 1;
textureDesc.Usage = D3D11_USAGE_DEFAULT;
textureDesc.BindFlags = D3D11_BIND_SHADER_RESOURCE;

D3D11_SUBRESOURCE_DATA textureData = {};
textureData.pSysMem = imageData;        // 이미지 픽셀 데이터
textureData.SysMemPitch = 512 * 4;      // 한 줄당 바이트 수

ID3D11Texture2D* texture;
device->CreateTexture2D(&textureDesc, &textureData, &texture);

// 셰이더에서 사용하기 위한 뷰 생성
ID3D11ShaderResourceView* textureView;
device->CreateShaderResourceView(texture, nullptr, &textureView);
```

**왜 사용하는가?**
- 단순한 색상만으로는 현실적인 표현 한계
- 나무 질감, 금속 표면, 피부 등 복잡한 디테일 표현
- 메모리 효율적인 압축 포맷 지원

#### 렌더 타겟 텍스처
화면이 아닌 메모리상의 텍스처에 렌더링할 때 사용합니다.

```cpp
D3D11_TEXTURE2D_DESC renderTargetDesc = {};
renderTargetDesc.Width = 1024;
renderTargetDesc.Height = 1024;
renderTargetDesc.MipLevels = 1;
renderTargetDesc.ArraySize = 1;
renderTargetDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
renderTargetDesc.SampleDesc.Count = 1;
renderTargetDesc.Usage = D3D11_USAGE_DEFAULT;
renderTargetDesc.BindFlags = D3D11_BIND_RENDER_TARGET | D3D11_BIND_SHADER_RESOURCE;

ID3D11Texture2D* renderTargetTexture;
device->CreateTexture2D(&renderTargetDesc, nullptr, &renderTargetTexture);

ID3D11RenderTargetView* renderTargetView;
device->CreateRenderTargetView(renderTargetTexture, nullptr, &renderTargetView);
```

**사용 목적:**
- 그림자 맵 생성
- 후처리 효과 (블러, 글로우 등)
- 미니맵 렌더링
- 큐브맵 생성

#### 프레임 버퍼 렌더 타겟 뷰
SwapChain의 백 버퍼를 사용하여 화면에 직접 렌더링할 때 사용합니다.

```cpp
// 1. SwapChain으로부터 백 버퍼 텍스처 얻기
ID3D11Texture2D* backBuffer = nullptr;
HRESULT hr = swapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), (void**)&backBuffer);
if (FAILED(hr)) {
    // 에러 처리
    return false;
}

// 2. 백 버퍼로부터 렌더 타겟 뷰 생성
ID3D11RenderTargetView* frameBufferRTV = nullptr;
hr = device->CreateRenderTargetView(backBuffer, nullptr, &frameBufferRTV);
backBuffer->Release(); // 텍스처 참조 해제 (RTV가 참조를 가지고 있음)

if (FAILED(hr)) {
    // 에러 처리
    return false;
}

// 3. 깊이-스텐실 버퍼도 함께 생성 (일반적으로 필요)
D3D11_TEXTURE2D_DESC depthStencilDesc = {};
depthStencilDesc.Width = windowWidth;
depthStencilDesc.Height = windowHeight;
depthStencilDesc.MipLevels = 1;
depthStencilDesc.ArraySize = 1;
depthStencilDesc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;  // 24비트 깊이 + 8비트 스텐실
depthStencilDesc.SampleDesc.Count = 1;
depthStencilDesc.SampleDesc.Quality = 0;
depthStencilDesc.Usage = D3D11_USAGE_DEFAULT;
depthStencilDesc.BindFlags = D3D11_BIND_DEPTH_STENCIL;

ID3D11Texture2D* depthStencilTexture = nullptr;
device->CreateTexture2D(&depthStencilDesc, nullptr, &depthStencilTexture);

ID3D11DepthStencilView* depthStencilView = nullptr;
device->CreateDepthStencilView(depthStencilTexture, nullptr, &depthStencilView);
depthStencilTexture->Release();

// 4. 렌더 타겟과 깊이-스텐실 뷰를 파이프라인에 바인딩
deviceContext->OMSetRenderTargets(1, &frameBufferRTV, depthStencilView);
```

**프레임 버퍼 RTV의 특징:**
- **직접 화면 출력**: SwapChain과 연결되어 사용자가 실제로 보는 화면
- **더블 버퍼링**: 백 버퍼에 그리고 Present()로 프론트 버퍼와 교체
- **윈도우 크기 종속**: 윈도우 크기가 변경되면 재생성 필요

**윈도우 크기 변경 시 처리:**
```cpp
void OnWindowResize(int newWidth, int newHeight) {
    // 기존 뷰들 해제
    if (frameBufferRTV) {
        frameBufferRTV->Release();
        frameBufferRTV = nullptr;
    }
    if (depthStencilView) {
        depthStencilView->Release();
        depthStencilView = nullptr;
    }
    
    // SwapChain 백 버퍼 크기 조정
    swapChain->ResizeBuffers(0, newWidth, newHeight, DXGI_FORMAT_UNKNOWN, 0);
    
    // 새로운 크기로 렌더 타겟 뷰 재생성
    CreateFrameBufferRTV();
    CreateDepthStencilView(newWidth, newHeight);
}
```

**사용 시나리오:**
- **메인 렌더링**: 게임의 최종 화면 출력
- **UI 렌더링**: 사용자 인터페이스 요소들
- **최종 후처리**: 모든 효과가 적용된 최종 이미지

### 3. 셰이더(Shader) 리소스

#### 정점 셰이더(Vertex Shader)
각 정점에 대해 실행되어 위치를 변환합니다.

```cpp
// HLSL 셰이더 코드를 컴파일한 바이트코드 로드
std::vector<BYTE> vertexShaderBytecode = LoadShaderFile("VertexShader.cso");

ID3D11VertexShader* vertexShader;
device->CreateVertexShader(
    vertexShaderBytecode.data(),
    vertexShaderBytecode.size(),
    nullptr,
    &vertexShader
);
```

#### 픽셀 셰이더(Pixel Shader)
각 픽셀의 최종 색상을 계산합니다.

```cpp
std::vector<BYTE> pixelShaderBytecode = LoadShaderFile("PixelShader.cso");

ID3D11PixelShader* pixelShader;
device->CreatePixelShader(
    pixelShaderBytecode.data(),
    pixelShaderBytecode.size(),
    nullptr,
    &pixelShader
);
```

**왜 셰이더가 필요한가?**
- GPU의 병렬 처리 능력을 최대한 활용
- 복잡한 조명 계산, 재질 표현 등을 실시간으로 처리
- 프로그래머가 원하는 시각적 효과 구현 가능

### 4. 상태 객체(State Objects)

#### 래스터라이저 상태(Rasterizer State)
삼각형을 픽셀로 변환하는 방식을 제어합니다.

```cpp
D3D11_RASTERIZER_DESC rasterizerDesc = {};
rasterizerDesc.FillMode = D3D11_FILL_SOLID;      // 면 채우기
rasterizerDesc.CullMode = D3D11_CULL_BACK;       // 뒷면 제거
rasterizerDesc.FrontCounterClockwise = false;
rasterizerDesc.DepthClipEnable = true;

ID3D11RasterizerState* rasterizerState;
device->CreateRasterizerState(&rasterizerDesc, &rasterizerState);
```

#### 블렌드 상태(Blend State)
투명도 처리 및 색상 혼합 방식을 정의합니다.

```cpp
D3D11_BLEND_DESC blendDesc = {};
blendDesc.RenderTarget[0].BlendEnable = true;
blendDesc.RenderTarget[0].SrcBlend = D3D11_BLEND_SRC_ALPHA;
blendDesc.RenderTarget[0].DestBlend = D3D11_BLEND_INV_SRC_ALPHA;
blendDesc.RenderTarget[0].BlendOp = D3D11_BLEND_OP_ADD;

ID3D11BlendState* blendState;
device->CreateBlendState(&blendDesc, &blendState);
```

**상태 객체의 이점:**
- 렌더링 파이프라인 설정을 미리 정의
- 런타임에 빠른 상태 전환 가능
- 여러 객체에서 동일한 상태 재사용

### 5. 입력 레이아웃(Input Layout)

정점 데이터의 구조를 GPU에게 알려주는 역할을 합니다.

```cpp
D3D11_INPUT_ELEMENT_DESC inputElements[] = {
    {"POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D11_INPUT_PER_VERTEX_DATA, 0},
    {"COLOR", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 12, D3D11_INPUT_PER_VERTEX_DATA, 0},
    {"TEXCOORD", 0, DXGI_FORMAT_R32G32_FLOAT, 0, 28, D3D11_INPUT_PER_VERTEX_DATA, 0}
};

ID3D11InputLayout* inputLayout;
device->CreateInputLayout(
    inputElements,
    ARRAYSIZE(inputElements),
    vertexShaderBytecode.data(),
    vertexShaderBytecode.size(),
    &inputLayout
);
```

**왜 필요한가?**
- 정점 데이터가 메모리에 어떻게 배치되어 있는지 GPU가 알아야 함
- 셰이더의 입력과 정점 버퍼를 올바르게 연결
- 다양한 정점 구조체를 유연하게 지원

## 리소스 관리의 중요성

### 메모리 해제
```cpp
// 모든 리소스는 사용 후 반드시 해제해야 함
if (vertexBuffer) vertexBuffer->Release();
if (indexBuffer) indexBuffer->Release();
if (texture) texture->Release();
if (vertexShader) vertexShader->Release();
// ... 기타 리소스들
```

### 리소스 재사용
- 동일한 텍스처나 셰이더는 여러 객체에서 공유
- 상태 객체는 필요한 조합만 미리 생성해서 재사용
- 상수 버퍼는 크기별로 풀(Pool)을 만들어 관리

## Device의 CreateBuffer 내부 동작 과정

`device->CreateBuffer()`를 호출하면 내부적으로 어떤 일이 일어날까요? 이 과정을 이해하면 GPU 메모리 관리와 성능 최적화에 대한 깊은 통찰을 얻을 수 있습니다.

### CreateBuffer 호출 시 전체 흐름

```cpp
// 사용자 코드
ID3D11Buffer* vertexBuffer;
HRESULT hr = device->CreateBuffer(&bufferDesc, &initData, &vertexBuffer);
```

이 한 줄의 코드 뒤에서는 복잡한 과정들이 순차적으로 진행됩니다:

```
1. CPU 측 검증 및 준비 (DirectX Runtime)
   ↓
2. GPU 드라이버와 통신 (WDDM)
   ↓  
3. GPU 메모리 할당
   ↓
4. 데이터 전송 (initData가 있는 경우)
   ↓
5. DirectX 객체 생성 및 반환
```

### 단계별 상세 과정

#### 1단계: CPU 측 검증 및 최적화 (DirectX Runtime)

```cpp
// DirectX Runtime 내부에서 수행되는 검증 (의사코드)
bool ValidateBufferDesc(const D3D11_BUFFER_DESC* desc) {
    // 크기 검증
    if (desc->ByteWidth == 0 || desc->ByteWidth > MAX_BUFFER_SIZE) {
        return false;
    }
    
    // Usage와 CPUAccessFlags 조합 검증
    if (desc->Usage == D3D11_USAGE_IMMUTABLE && desc->CPUAccessFlags != 0) {
        return false; // IMMUTABLE은 CPU 접근 불가
    }
    
    // BindFlags 유효성 검증
    if (desc->BindFlags == 0) {
        return false; // 최소 하나의 바인딩 플래그 필요
    }
    
    return true;
}
```

**이 단계에서 하는 일:**
- 파라미터 유효성 검사
- 메모리 정렬 요구사항 계산 (16바이트 경계 등)
- 최적의 메모리 타입 결정
- GPU 드라이버에 전달할 명령 구성

#### 2단계: GPU 드라이버 통신 (WDDM)

DirectX Runtime이 Windows Display Driver Model (WDDM)을 통해 GPU 드라이버와 통신합니다:

```cpp
// 의사코드: 드라이버 통신 과정
struct DriverCommand {
    CommandType type = CREATE_BUFFER;
    BufferCreateInfo info;
    MemoryRequirements memReq;
};

// GPU 드라이버에게 버퍼 생성 요청
HRESULT SubmitToDriver(DriverCommand& cmd) {
    // 1. 드라이버 큐에 명령 추가
    driverCommandQueue.Push(cmd);
    
    // 2. GPU와 동기화 (필요시)
    if (requiresImmediateExecution) {
        WaitForGPUIdle();
    }
    
    return S_OK;
}
```

#### 3단계: GPU 메모리 할당 결정

GPU 드라이버는 Usage 플래그에 따라 메모리 위치를 결정합니다:

##### D3D11_USAGE_DEFAULT
```cpp
// GPU 전용 메모리 (VRAM)에 할당
Location: GPU Local Memory (VRAM)
Access: GPU 읽기/쓰기 최적화
Bandwidth: 최고 (500+ GB/s)
Latency: 최저

메모리 레이아웃:
┌─────────────────────┐
│    시스템 RAM       │  ← CPU가 직접 접근
├─────────────────────┤
│    PCIe Bus         │  ← 데이터 전송 경로
├─────────────────────┤
│    GPU VRAM         │  ← 버퍼가 여기에 생성 ★
│  ┌───────────────┐  │
│  │ Vertex Buffer │  │
│  └───────────────┘  │
└─────────────────────┘
```

##### D3D11_USAGE_DYNAMIC
```cpp
// 시스템 메모리 또는 GPU의 업로드 힙에 할당
Location: System RAM + GPU Upload Heap
Access: CPU 쓰기, GPU 읽기 최적화
Bandwidth: 중간 (30-50 GB/s)
Update Frequency: 매 프레임 가능

메모리 레이아웃:
┌─────────────────────┐
│   시스템 RAM        │  ← CPU가 여기에 쓰기
│ ┌─────────────────┐ │
│ │ Dynamic Buffer  │ │  ★ 버퍼가 여기에 생성
│ └─────────────────┘ │
├─────────────────────┤
│   GPU Upload Heap   │  ← GPU가 여기서 읽기
│ ┌─────────────────┐ │
│ │   Shadow Copy   │ │
│ └─────────────────┘ │
└─────────────────────┘
```

##### D3D11_USAGE_STAGING
```cpp
// 시스템 메모리에 할당 (GPU-CPU 데이터 교환용)
Location: System RAM
Access: CPU 읽기/쓰기, GPU 복사 작업만
Bandwidth: 낮음 (10-20 GB/s)
Purpose: 데이터 교환 중계소

메모리 레이아웃:
┌─────────────────────┐
│   시스템 RAM        │
│ ┌─────────────────┐ │  ★ 버퍼가 여기에 생성
│ │ Staging Buffer  │ │  ← CPU와 GPU 모두 접근 가능
│ └─────────────────┘ │
└─────────────────────┘
        ↕ (복사 작업)
┌─────────────────────┐
│    GPU VRAM         │
│ ┌─────────────────┐ │
│ │ Target Buffer   │ │
│ └─────────────────┘ │
└─────────────────────┘
```

#### 4단계: 초기 데이터 전송 (initData가 있는 경우)

`D3D11_SUBRESOURCE_DATA* initData`가 제공된 경우:

```cpp
if (initData != nullptr) {
    // 전송 방법은 메모리 타입에 따라 달라짐
    
    if (usage == D3D11_USAGE_DEFAULT) {
        // GPU VRAM으로 직접 업로드
        UploadToGPUMemory(initData->pSysMem, bufferSize);
    }
    else if (usage == D3D11_USAGE_DYNAMIC) {
        // 시스템 메모리에 복사
        memcpy(systemBuffer, initData->pSysMem, bufferSize);
    }
}
```

##### DEFAULT 버퍼 초기화 과정
```
1. CPU 메모리에서 GPU 업로드 힙으로 복사
   [시스템 RAM] → [GPU Upload Heap]
   
2. GPU 내부에서 업로드 힙에서 VRAM으로 복사
   [GPU Upload Heap] → [GPU VRAM]
   
3. 업로드 힙 메모리 해제
   [GPU Upload Heap] (해제됨)
```

#### 5단계: DirectX 객체 생성

마지막으로 DirectX Runtime이 관리 객체를 생성합니다:

```cpp
// 의사코드: DirectX 버퍼 객체 생성
class D3D11Buffer : public ID3D11Buffer {
private:
    void* gpuMemoryHandle;      // GPU 메모리 핸들
    UINT size;                  // 버퍼 크기
    D3D11_USAGE usage;         // 사용법
    UINT bindFlags;            // 바인딩 플래그
    
public:
    // 참조 카운팅으로 메모리 관리
    ULONG AddRef() override;
    ULONG Release() override;
};

// 사용자에게 반환되는 포인터
*ppBuffer = new D3D11Buffer(gpuMemoryHandle, bufferDesc);
```

### 메모리 타입별 성능 특성

#### GPU 메모리 계층 구조
```
CPU 관점에서의 접근 속도:
┌─────────────────────┐  ← 가장 빠름 (CPU 기준)
│   CPU Cache         │
├─────────────────────┤
│   시스템 RAM        │  ← DYNAMIC/STAGING 버퍼 위치
├─────────────────────┤
│   PCIe Bus          │  ← 병목 지점 (16-32 GB/s)
├─────────────────────┤
│   GPU VRAM          │  ← DEFAULT 버퍼 위치
└─────────────────────┘  ← CPU가 직접 접근 불가

GPU 관점에서의 접근 속도:
┌─────────────────────┐  ← 가장 빠름 (GPU 기준)
│   GPU Cache         │
├─────────────────────┤
│   GPU VRAM          │  ← DEFAULT 버퍼 위치
├─────────────────────┤
│   PCIe Bus          │  ← 병목 지점
├─────────────────────┤
│   시스템 RAM        │  ← DYNAMIC/STAGING 버퍼 위치
└─────────────────────┘  ← 가장 느림 (GPU 기준)
```

#### 실제 성능 수치 (대략적)

| 메모리 타입 | GPU→메모리 대역폭 | CPU→메모리 대역폭 | 지연시간 (GPU) |
|-------------|-------------------|-------------------|----------------|
| **GPU VRAM** | 500-1000 GB/s | 접근 불가 | 1-2 ns |
| **시스템 RAM** | 30-50 GB/s | 50-100 GB/s | 100-500 ns |
| **PCIe 전송** | 16-32 GB/s | 16-32 GB/s | 1-10 μs |

### 버퍼 타입별 최적 사용 시나리오

#### Usage별 권장 사용법

```cpp
// 정적 데이터 (모델의 정점 정보)
D3D11_USAGE_DEFAULT + D3D11_CPU_ACCESS_NONE
→ GPU VRAM에 생성, 최고 렌더링 성능

// 매 프레임 변하는 데이터 (변환 행렬)
D3D11_USAGE_DYNAMIC + D3D11_CPU_ACCESS_WRITE  
→ 시스템 RAM에 생성, CPU 업데이트 용이

// 데이터 읽기/디버깅용
D3D11_USAGE_STAGING + D3D11_CPU_ACCESS_READ
→ 시스템 RAM에 생성, GPU→CPU 데이터 전송 가능
```

### 성능 최적화 팁

#### 1. 배치 생성 (Batch Creation)
```cpp
// 비효율적: 개별 생성
for (int i = 0; i < 1000; i++) {
    device->CreateBuffer(&desc, &data[i], &buffers[i]);  // 1000번의 드라이버 호출
}

// 효율적: 큰 버퍼 하나 생성 후 서브버퍼 사용
device->CreateBuffer(&bigDesc, &allData, &bigBuffer);   // 1번의 드라이버 호출
```

#### 2. 메모리 타입 선택
```cpp
// 자주 변하는 데이터는 DYNAMIC
constantBufferDesc.Usage = D3D11_USAGE_DYNAMIC;     // 매 프레임 업데이트

// 정적 데이터는 DEFAULT  
vertexBufferDesc.Usage = D3D11_USAGE_DEFAULT;       // 한 번 설정 후 변경 없음
```

#### 3. 초기 데이터 제공
```cpp
// 가능하면 생성 시 데이터 제공 (복사 횟수 최소화)
device->CreateBuffer(&desc, &initData, &buffer);    // 권장

// 보다는
device->CreateBuffer(&desc, nullptr, &buffer);      // 피할 것
context->UpdateSubresource(buffer, 0, nullptr, data, 0, 0);  // 추가 복사
```

### 메모리 누수 방지

```cpp
// RAII 패턴 사용 권장
class BufferManager {
private:
    std::vector<ID3D11Buffer*> buffers;
    
public:
    ~BufferManager() {
        // 자동으로 모든 버퍼 해제
        for (auto* buffer : buffers) {
            if (buffer) buffer->Release();
        }
    }
    
    ID3D11Buffer* CreateBuffer(const D3D11_BUFFER_DESC& desc, 
                              const D3D11_SUBRESOURCE_DATA* data = nullptr) {
        ID3D11Buffer* buffer = nullptr;
        HRESULT hr = device->CreateBuffer(&desc, data, &buffer);
        if (SUCCEEDED(hr)) {
            buffers.push_back(buffer);
        }
        return buffer;
    }
};
```

## Device vs Device Context

DirectX 11에서는 Device를 두 부분으로 나눴습니다:

### ID3D11Device
- **역할**: 리소스를 생성하고 관리
- **특징**: 멀티스레드에서 안전하게 사용 가능
- **비유**: 공장의 "설계부서" - 필요한 재료들을 만들어냄

```cpp
// Device를 사용한 리소스 생성 예시
ID3D11Buffer* vertexBuffer;
device->CreateBuffer(&bufferDesc, &initData, &vertexBuffer);
```

### ID3D11DeviceContext
- **역할**: 실제 렌더링 명령을 실행
- **특징**: 단일 스레드에서만 사용 가능
- **비유**: 공장의 "생산라인" - 만들어진 재료로 실제 제품을 조립

```cpp
// Device Context를 사용한 렌더링 예시
deviceContext->Draw(vertexCount, 0);
```

## Device 생성 과정

Device를 만드는 과정은 다음과 같습니다:

```cpp
// 1. 필요한 변수들 선언
ID3D11Device* device = nullptr;
ID3D11DeviceContext* deviceContext = nullptr;
IDXGISwapChain* swapChain = nullptr;

// 2. Device와 SwapChain 생성
D3D11CreateDeviceAndSwapChain(
    nullptr,                    // 어댑터 (nullptr = 기본)
    D3D_DRIVER_TYPE_HARDWARE,   // 하드웨어 가속 사용
    nullptr,                    // 소프트웨어 래스터라이저
    0,                          // 플래그
    nullptr,                    // 기능 레벨 배열
    0,                          // 기능 레벨 개수
    D3D11_SDK_VERSION,          // SDK 버전
    &swapChainDesc,             // SwapChain 설명
    &swapChain,                 // SwapChain 출력
    &device,                    // Device 출력
    nullptr,                    // 실제 기능 레벨
    &deviceContext              // Device Context 출력
);
```

## 왜 이렇게 복잡하게 나눠놨을까?

### 성능 최적화
- Device: 리소스 생성은 상대적으로 느리지만, 여러 스레드에서 동시에 할 수 있음
- Device Context: 렌더링은 빠르게 해야 하지만, 순서가 중요해서 한 번에 하나씩만

### 멀티스레딩 지원
- 여러 스레드에서 동시에 리소스를 만들 수 있음
- 하지만 실제 그리기는 메인 스레드에서만

## 실제 사용 예시

```cpp
class Renderer {
private:
    ID3D11Device* m_device;
    ID3D11DeviceContext* m_deviceContext;
    
public:
    void Initialize() {
        // Device 생성
        CreateDeviceAndSwapChain();
        
        // 필요한 리소스들 생성
        CreateVertexBuffer();
        CreateShaders();
        CreateTextures();
    }
    
    void Render() {
        // 렌더링 준비
        m_deviceContext->ClearRenderTargetView(m_renderTargetView, clearColor);
        
        // 그리기 명령
        m_deviceContext->DrawIndexed(indexCount, 0, 0);
        
        // 화면에 표시
        m_swapChain->Present(1, 0);
    }
};
```

## 핵심 정리

1. **Device**는 GPU와 소통하기 위한 인터페이스
2. **ID3D11Device**는 리소스 생성 담당 (멀티스레드 안전)
3. **ID3D11DeviceContext**는 실제 렌더링 담당 (단일스레드)
4. 둘을 나눈 이유는 성능과 멀티스레딩 지원을 위해서
5. 모든 DirectX 11 프로그래밍은 이 Device를 중심으로 돌아감

Device는 DirectX 11 프로그래밍의 시작점이자 핵심입니다. 이를 잘 이해하면 나머지 개념들도 훨씬 쉽게 이해할 수 있습니다!