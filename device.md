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