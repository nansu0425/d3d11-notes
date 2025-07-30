# DirectX 11 Swap Chain (스왑 체인)

## 스왑 체인이란 무엇인가?

**Swap Chain**은 화면에 이미지를 표시하기 위한 메커니즘입니다. 쉽게 말해, GPU에서 그린 그림을 모니터 화면에 보여주는 "전시관" 역할을 합니다.

게임이나 3D 애플리케이션에서 매 프레임마다 새로운 이미지를 그리는데, 이 이미지들을 어떻게 매끄럽게 화면에 표시할지를 관리하는 것이 스왑 체인의 역할입니다.

## 왜 스왑 체인이 필요할까?

### 문제상황: 직접 화면에 그리기
만약 GPU가 화면에 직접 그린다면 어떻게 될까요?

```
모니터가 화면을 갱신하는 중간에 GPU가 그리기 시작
→ 화면 위쪽은 이전 프레임, 아래쪽은 새 프레임
→ 화면이 찢어져 보이는 현상 (Screen Tearing) 발생
```

### 해결책: 더블 버퍼링
스왑 체인은 여러 개의 백버퍼를 사용해서 이 문제를 해결합니다:

1. **Back Buffer**: GPU가 그림을 그리는 숨겨진 캔버스
2. **Front Buffer**: 실제로 화면에 표시되는 이미지
3. **Swap**: 그리기가 완료되면 백버퍼와 프론트버퍼를 교체

## 스왑 체인의 구조

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Back Buffer   │ -> │  Back Buffer    │ -> │   Front Buffer  │
│   (그리는 중)    │    │   (대기 중)      │    │  (화면에 표시)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         ↑                       ↑                       ↓
    GPU가 여기에            다음에 표시될              지금 화면에
      그림을 그림              이미지                   보이는 이미지
```

## 주요 구성 요소

### 1. Buffer Count (버퍼 개수)
```cpp
DXGI_SWAP_CHAIN_DESC swapChainDesc = {};
swapChainDesc.BufferCount = 2;  // 더블 버퍼링 (Front + Back)
// swapChainDesc.BufferCount = 3;  // 트리플 버퍼링 (더 부드러움)
```

- **더블 버퍼링 (2개)**: 가장 일반적, 메모리 효율적
- **트리플 버퍼링 (3개)**: 더 부드럽지만 메모리 사용량 증가

### 2. Buffer Format (버퍼 포맷)
```cpp
swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
// R(빨강) G(초록) B(파랑) A(투명도) 각각 8비트 = 32비트 컬러
```

### 3. Refresh Rate (주사율)
```cpp
swapChainDesc.BufferDesc.RefreshRate.Numerator = 60;    // 분자
swapChainDesc.BufferDesc.RefreshRate.Denominator = 1;   // 분모
// 결과: 60Hz (초당 60프레임)
```

### 4. Swap Effect (스왑 방식)
스왑 체인이 백버퍼와 프론트버퍼를 교체하는 방식을 결정합니다:

#### DXGI_SWAP_EFFECT_DISCARD
```cpp
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
```
- **동작 방식**: Present() 호출 후 백버퍼 내용을 버림 (undefined 상태)
- **특징**: 
  - 가장 일반적이고 호환성이 좋음
  - 메모리 사용량이 적음
  - 이전 프레임 내용에 의존하지 않는 렌더링에 적합
- **사용 시기**: 매 프레임 전체 화면을 다시 그리는 대부분의 게임

#### DXGI_SWAP_EFFECT_SEQUENTIAL
```cpp
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_SEQUENTIAL;
```
- **동작 방식**: 백버퍼들이 순차적으로 회전하며 내용 보존
- **특징**:
  - 이전 프레임 내용이 보존됨
  - 메모리 사용량이 더 많음
  - 부분 업데이트가 가능
- **사용 시기**: UI 애플리케이션이나 부분 렌더링이 필요한 경우

#### DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL (권장)
```cpp
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL;
```
- **동작 방식**: 하드웨어 레벨에서 포인터만 교체 (실제 복사 없음)
- **특징**:
  - 매우 빠른 성능
  - 낮은 지연시간 (Low Latency)
  - Windows 8 이상에서 지원
  - 전체화면 최적화 활성화
- **제한사항**: BufferCount는 최소 2 이상이어야 함
- **사용 시기**: 고성능이 필요한 최신 게임

#### DXGI_SWAP_EFFECT_FLIP_DISCARD (최고 성능)
```cpp
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;
```
- **동작 방식**: FLIP_SEQUENTIAL + 백버퍼 내용 버리기
- **특징**:
  - 최고의 성능과 최저 지연시간
  - Windows 10 이상에서 지원
  - Variable Refresh Rate (VRR) 지원
  - Auto HDR 지원
- **제한사항**: BufferCount는 최소 2 이상이어야 함
- **사용 시기**: 최신 하드웨어와 OS에서 최고 성능이 필요한 경우

## FLIP 모델에서 BufferCount가 2 이상인 이유

### 전통적인 BitBlt 모델 vs FLIP 모델

#### BitBlt 모델 (DISCARD, SEQUENTIAL)
```
┌─────────────┐    복사    ┌─────────────┐
│ Back Buffer │ ---------> │Front Buffer │
│  (앱이 그림) │            │ (화면 출력)  │
└─────────────┘            └─────────────┘
     1개만 있어도 OK          (시스템 관리)
```
- GPU가 백버퍼 내용을 프론트버퍼로 **복사**
- 복사하는 동안 백버퍼는 사용 가능
- BufferCount = 1이어도 동작 가능

#### FLIP 모델 (FLIP_SEQUENTIAL, FLIP_DISCARD)
```
┌─────────────┐    포인터   ┌─────────────┐
│ Buffer A    │ <--------> │ Buffer B    │
│ (현재 출력)  │   교체     │ (다음 준비)  │
└─────────────┘            └─────────────┘
     최소 2개 필요 - 역할을 서로 바꿔가며 사용
```
- GPU가 버퍼들의 **포인터만 교체** (복사 없음)
- 한 버퍼가 화면에 출력되는 동안, 다른 버퍼에 그려야 함
- **물리적으로 최소 2개의 버퍼가 동시에 필요**

### 구체적인 동작 과정

#### 2개 버퍼로 FLIP 동작
```cpp
// 초기 상태
Buffer A: [화면에 표시 중]
Buffer B: [GPU가 새로운 프레임 그리는 중]

// Present() 호출 시
Buffer A: [GPU가 새로운 프레임 그리는 중] ← 역할 바뀜
Buffer B: [화면에 표시 중] ← 역할 바뀜
```

#### 만약 1개 버퍼만 있다면?
```cpp
// 불가능한 상황
Buffer A: [화면에 표시 중] + [GPU가 그리려고 함]
// → 동시에 두 가지 역할을 할 수 없음!
```

### 왜 복사 대신 포인터 교체를 사용하는가?

#### 성능 비교
```cpp
// BitBlt 모델 (복사 방식)
1. GPU가 백버퍼에 그림
2. Present() 호출
3. 백버퍼 전체를 프론트버퍼로 복사 ← 느림!
4. 복사 완료까지 대기

// FLIP 모델 (포인터 교체)
1. GPU가 백버퍼에 그림
2. Present() 호출
3. 포인터만 교체 (거의 즉시 완료) ← 빠름!
4. 대기 시간 거의 없음
```

#### 메모리 대역폭 절약
```cpp
// 1920x1080 32비트 화면 기준
복사해야 할 데이터: 1920 × 1080 × 4 = 8.3MB
60fps에서: 8.3MB × 60 = 498MB/초 ← 대역폭 낭비

// FLIP에서는 복사가 없으므로 대역폭 절약
```

### 트리플 버퍼링의 장점

```cpp
// 더블 버퍼링 (BufferCount = 2)
Buffer A: [화면 출력]
Buffer B: [GPU 렌더링]
// GPU가 빨리 끝나면 A가 교체될 때까지 대기

// 트리플 버퍼링 (BufferCount = 3)
Buffer A: [화면 출력]
Buffer B: [GPU 렌더링]
Buffer C: [다음 프레임 대기]
// GPU가 빨리 끝나면 C에서 바로 다음 프레임 시작 가능
```

### 실제 구현에서의 고려사항

```cpp
// FLIP 모델 생성 시 필수 조건
DXGI_SWAP_CHAIN_DESC swapChainDesc = {};
swapChainDesc.BufferCount = 2;  // 최소 2개!
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;

// 1개로 설정하면 생성 실패
swapChainDesc.BufferCount = 1;  // ← 이렇게 하면 에러!
HRESULT hr = D3D11CreateDeviceAndSwapChain(...);
// hr == DXGI_ERROR_INVALID_CALL
```

### 메모리 사용량 비교

```cpp
// 1920x1080 화면 기준 메모리 사용량
BitBlt (BufferCount=1): 8.3MB (백버퍼만)
FLIP (BufferCount=2):  16.6MB (2개 버퍼)
FLIP (BufferCount=3):  24.9MB (3개 버퍼)

// 성능 향상을 위한 합리적인 메모리 투자
```

## SwapEffect 선택 가이드

### 호환성 우선
```cpp
// 모든 환경에서 안정적으로 동작
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
swapChainDesc.BufferCount = 1;
```

### 성능과 호환성의 균형
```cpp
// Windows 8 이상에서 좋은 성능
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL;
swapChainDesc.BufferCount = 2;
```

### 최고 성능 (권장)
```cpp
// Windows 10 이상에서 최고 성능
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;
swapChainDesc.BufferCount = 2;  // 또는 3 (트리플 버퍼링)
```

### 런타임 선택 예시
```cpp
// OS 버전에 따른 자동 선택
DXGI_SWAP_EFFECT GetOptimalSwapEffect() {
    // Windows 10 이상인지 확인
    if (IsWindows10OrGreater()) {
        return DXGI_SWAP_EFFECT_FLIP_DISCARD;
    }
    // Windows 8 이상인지 확인
    else if (IsWindows8OrGreater()) {
        return DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL;
    }
    // 이전 버전
    else {
        return DXGI_SWAP_EFFECT_DISCARD;
    }
}
```

## SwapEffect와 다른 설정의 관계

### BufferCount 제한
```cpp
// DISCARD: 1개 이상
swapChainDesc.BufferCount = 1;  // 가능
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;

// FLIP 계열: 2개 이상 필수
swapChainDesc.BufferCount = 2;  // 최소값
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;
```

### MSAA 제한
```cpp
// FLIP 계열에서는 MSAA 사용 불가
swapChainDesc.SampleDesc.Count = 1;      // 필수
swapChainDesc.SampleDesc.Quality = 0;    // 필수
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;
```

### 전체화면 최적화
FLIP 계열 SwapEffect는 윈도우 모드에서도 전체화면 최적화를 활성화합니다:
- 더 나은 성능
- 낮은 입력 지연시간
- Variable Refresh Rate 지원
- Auto HDR 지원 (Windows 11)

## 스왑 체인 생성하기

### 1. DXGI_SWAP_CHAIN_DESC 설정
```cpp
DXGI_SWAP_CHAIN_DESC swapChainDesc = {};

// 버퍼 설정
swapChainDesc.BufferCount = 2;
swapChainDesc.BufferDesc.Width = 1920;
swapChainDesc.BufferDesc.Height = 1080;
swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
swapChainDesc.BufferDesc.RefreshRate.Numerator = 60;
swapChainDesc.BufferDesc.RefreshRate.Denominator = 1;

// 사용법 설정
swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
swapChainDesc.OutputWindow = hwnd;  // 윈도우 핸들
swapChainDesc.SampleDesc.Count = 1;
swapChainDesc.SampleDesc.Quality = 0;
swapChainDesc.Windowed = TRUE;  // 창 모드
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
```

### 2. Device와 함께 생성
```cpp
ID3D11Device* device = nullptr;
ID3D11DeviceContext* deviceContext = nullptr;
IDXGISwapChain* swapChain = nullptr;

HRESULT hr = D3D11CreateDeviceAndSwapChain(
    nullptr,                    // 기본 어댑터 사용
    D3D_DRIVER_TYPE_HARDWARE,   // 하드웨어 가속
    nullptr,
    0,
    nullptr,
    0,
    D3D11_SDK_VERSION,
    &swapChainDesc,             // 스왑체인 설정
    &swapChain,                 // 생성된 스왑체인
    &device,
    nullptr,
    &deviceContext
);
```

## 렌더 타겟 뷰 생성

스왑 체인의 백버퍼를 사용하려면 **Render Target View**를 만들어야 합니다:

```cpp
// 1. 백버퍼 가져오기
ID3D11Texture2D* backBuffer = nullptr;
swapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), (void**)&backBuffer);

// 2. 렌더 타겟 뷰 생성
ID3D11RenderTargetView* renderTargetView = nullptr;
device->CreateRenderTargetView(backBuffer, nullptr, &renderTargetView);

// 3. 백버퍼 참조 해제 (렌더 타겟 뷰가 참조하고 있음)
backBuffer->Release();

// 4. 렌더 타겟으로 설정
deviceContext->OMSetRenderTargets(1, &renderTargetView, nullptr);
```

## 실제 렌더링 과정

```cpp
void Render() {
    // 1. 백버퍼 지우기 (새로운 프레임 시작)
    float clearColor[4] = { 0.0f, 0.0f, 1.0f, 1.0f };  // 파란색
    deviceContext->ClearRenderTargetView(renderTargetView, clearColor);
    
    // 2. 실제 그리기 작업들
    deviceContext->DrawIndexed(indexCount, 0, 0);
    // ... 더 많은 그리기 명령들
    
    // 3. 백버퍼를 프론트버퍼로 교체 (화면에 표시)
    swapChain->Present(1, 0);  // VSync 대기
    // swapChain->Present(0, 0);  // VSync 무시 (더 빠르지만 tearing 가능)
}
```

## Present() 함수의 옵션

### SyncInterval (첫 번째 매개변수)
```cpp
swapChain->Present(0, 0);  // VSync 끄기 - 최대한 빠르게
swapChain->Present(1, 0);  // VSync 켜기 - 모니터 주사율에 맞춤
swapChain->Present(2, 0);  // 2 VSync - 30fps로 제한 (60Hz 모니터에서)
```

### Flags (두 번째 매개변수)
```cpp
swapChain->Present(1, 0);                           // 일반 모드
swapChain->Present(1, DXGI_PRESENT_DO_NOT_WAIT);    // 논블로킹 모드
```

## 전체화면 vs 창모드

### 창모드 (Windowed)
```cpp
swapChainDesc.Windowed = TRUE;
swapChainDesc.BufferDesc.RefreshRate.Numerator = 0;      // 시스템이 결정
swapChainDesc.BufferDesc.RefreshRate.Denominator = 1;
```

### 전체화면 (Fullscreen)
```cpp
swapChainDesc.Windowed = FALSE;
swapChainDesc.BufferDesc.RefreshRate.Numerator = 60;     // 명시적 설정
swapChainDesc.BufferDesc.RefreshRate.Denominator = 1;

// 런타임에 전환도 가능
swapChain->SetFullscreenState(TRUE, nullptr);   // 전체화면으로
swapChain->SetFullscreenState(FALSE, nullptr);  // 창모드로
```

## 창 크기 변경 처리

창 크기가 바뀔 때는 스왑 체인도 크기를 조정해야 합니다:

```cpp
void OnWindowResize(int newWidth, int newHeight) {
    // 1. 렌더 타겟 뷰 해제
    deviceContext->OMSetRenderTargets(0, nullptr, nullptr);
    renderTargetView->Release();
    
    // 2. 스왑 체인 버퍼 크기 조정
    swapChain->ResizeBuffers(0, newWidth, newHeight, 
                           DXGI_FORMAT_UNKNOWN, 0);
    
    // 3. 새로운 렌더 타겟 뷰 생성
    ID3D11Texture2D* backBuffer;
    swapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), (void**)&backBuffer);
    device->CreateRenderTargetView(backBuffer, nullptr, &renderTargetView);
    backBuffer->Release();
    
    // 4. 다시 설정
    deviceContext->OMSetRenderTargets(1, &renderTargetView, nullptr);
}
```

## 일반적인 문제들과 해결책

### 1. Screen Tearing (화면 찢어짐)
**문제**: VSync가 꺼져있을 때 발생
**해결**: `Present(1, 0)` 사용하여 VSync 활성화

### 2. 낮은 성능
**문제**: 불필요한 VSync 대기
**해결**: `Present(0, 0)` 사용하거나 적응형 VSync 구현

### 3. Alt+Tab 문제
**문제**: 전체화면에서 창 전환 시 오류
**해결**: `DXGI_SWAP_EFFECT_FLIP_DISCARD` 사용

## 성능 최적화 팁

### 1. 적절한 버퍼 개수 선택
```cpp
// 일반적인 경우: 더블 버퍼링
swapChainDesc.BufferCount = 2;

// 높은 성능이 필요한 경우: 트리플 버퍼링
swapChainDesc.BufferCount = 3;
```

### 2. 적절한 SwapEffect 선택
```cpp
// 호환성 우선 - 모든 OS에서 동작
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
swapChainDesc.BufferCount = 1;

// 균형잡힌 성능 - Windows 8 이상
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL;
swapChainDesc.BufferCount = 2;

// 최고 성능 - Windows 10 이상 (권장)
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;
swapChainDesc.BufferCount = 2;
```

**성능 차이**:
- `DISCARD`: 기본 성능, 높은 호환성
- `FLIP_SEQUENTIAL`: 약 10-30% 성능 향상
- `FLIP_DISCARD`: 약 20-50% 성능 향상, 최저 지연시간

## 핵심 정리

1. **스왑 체인**은 GPU가 그린 이미지를 화면에 표시하는 메커니즘
2. **더블 버퍼링**을 통해 화면 찢어짐 현상을 방지
3. **Present()** 를 호출해야 실제로 화면에 표시됨
4. **VSync** 설정으로 성능과 품질의 균형을 맞출 수 있음
5. **창 크기 변경** 시에는 버퍼 크기도 함께 조정해야 함
6. **렌더 타겟 뷰**를 통해 백버퍼에 그림을 그림

스왑 체인은 DirectX 11에서 화면 출력을 담당하는 핵심 구성 요소입니다. 이를 잘 이해하면 부드럽고 효율적인 렌더링을 구현할 수 있습니다!