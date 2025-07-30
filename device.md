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