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

### BitBlt 모델 구조 (DISCARD, SEQUENTIAL)
```
더블 버퍼링 (BufferCount = 1):
┌─────────────────┐    Present()    ┌─────────────────┐
│   Back Buffer   │ -------------> │   Front Buffer  │
│   (GPU가 그림)   │     복사       │  (화면에 표시)   │ ← 별도 존재
└─────────────────┘                └─────────────────┘
   BufferCount=1                        시스템 관리

트리플 버퍼링 (BufferCount = 2):
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Back Buffer 1 │    │   Back Buffer 2 │    │   Front Buffer  │
│   (GPU가 그림)   │    │   (대기 중)      │    │  (화면에 표시)   │ ← 별도 존재
└─────────────────┘    └─────────────────┘    └─────────────────┘
         ↑                       ↑                       ↓
    GPU가 여기에            다음에 렌더링할           지금 화면에
      그림을 그림              버퍼                   보이는 이미지
```

### FLIP 모델 구조 (FLIP_SEQUENTIAL, FLIP_DISCARD)
```
더블 버퍼링 (BufferCount = 2):
┌─────────────────┐    Present()    ┌─────────────────┐
│  Back Buffer A  │ <------------>  │  Back Buffer B  │
│ (현재 렌더링 중)  │     역할 교체    │(Front 역할 수행) │
└─────────────────┘                 └─────────────────┘
                                           ↓
                                    실제로 화면에 표시
                                    (별도 프론트버퍼 없음)

트리플 버퍼링 (BufferCount = 3):
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Back Buffer A  │    │  Back Buffer B  │    │  Back Buffer C  │
│ (현재 렌더링 중) │    │   (대기 중)      │    │(Front 역할 수행)│
└─────────────────┘    └─────────────────┘    └─────────────────┘
         ↑                       ↑                       ↓
    GPU가 여기에            다음에 렌더링할           실제로 화면에 표시
      그림을 그림              버퍼               (별도 프론트버퍼 없음)
```

## 주요 구성 요소

### 1. Buffer Count와 버퍼링 방식

**중요**: BufferCount는 **백버퍼의 개수**만을 지정하며, 스왑 모델에 따라 실제 버퍼링 방식이 달라집니다.

#### 모델별 BufferCount 매핑표

| 스왑 모델 | BufferCount | 실제 버퍼링 방식 | 총 버퍼 개수 | 프론트 버퍼 |
|-----------|-------------|------------------|--------------|-------------|
| **BitBlt** | 1 | 더블 버퍼링 | 2개 (프론트1 + 백1) | 별도 존재 |
| **BitBlt** | 2 | 트리플 버퍼링 | 3개 (프론트1 + 백2) | 별도 존재 |
| **FLIP** | 2 | 더블 버퍼링 | 2개 (백버퍼만, 역할 교체) | 없음 |
| **FLIP** | 3 | 트리플 버퍼링 | 3개 (백버퍼만, 역할 교체) | 없음 |

#### 핵심 차이점
- **BitBlt 모델**: 프론트 버퍼가 물리적으로 별도 존재, 복사 방식
- **FLIP 모델**: 백버퍼들이 front/back 역할을 교체, 포인터 교체 방식

```cpp
// BitBlt 모델 예시
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
swapChainDesc.BufferCount = 1;  // 더블 버퍼링

// FLIP 모델 예시
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;
swapChainDesc.BufferCount = 2;  // 더블 버퍼링 (최소값)
// swapChainDesc.BufferCount = 3;  // 트리플 버퍼링 (권장)
```

### 2. Buffer Format (버퍼 포맷)
백버퍼에 저장되는 픽셀 데이터의 형식을 지정합니다.

#### 포맷 이름 해석 방법
DXGI 포맷 이름은 다음과 같은 규칙으로 구성됩니다:
```
DXGI_FORMAT_[컴포넌트][비트수]_[타입]

예시: DXGI_FORMAT_R8G8B8A8_UNORM
- R8: Red 채널 8비트
- G8: Green 채널 8비트  
- B8: Blue 채널 8비트
- A8: Alpha 채널 8비트
- UNORM: Unsigned Normalized (0.0~1.0 범위)
```

#### 주요 포맷 비교

##### R8G8B8A8_UNORM (일반적)
```cpp
swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
```
- **메모리 레이아웃**: `[R][G][B][A]` (Red가 최하위 바이트)
- **용도**: 대부분의 일반적인 렌더링
- **특징**: 표준적인 RGBA 순서
- **호환성**: 모든 GPU에서 지원

##### B8G8R8A8_UNORM (Windows 최적화)
```cpp
swapChainDesc.BufferDesc.Format = DXGI_FORMAT_B8G8R8A8_UNORM;
```
- **메모리 레이아웃**: `[B][G][R][A]` (Blue가 최하위 바이트)
- **용도**: Windows UI, Desktop Window Manager
- **특징**: Windows 기본 포맷 (BGR 순서)
- **성능**: Windows에서 더 빠른 경우가 많음

#### 포맷 선택 가이드

```cpp
// 일반적인 3D 렌더링
swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;

// Windows UI나 2D 렌더링 (더 빠를 수 있음)
swapChainDesc.BufferDesc.Format = DXGI_FORMAT_B8G8R8A8_UNORM;

// HDR 렌더링 (높은 정밀도)
swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R16G16B16A16_FLOAT;

// 메모리 절약 (알파 채널 불필요)
swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM; // 권장
// 또는
swapChainDesc.BufferDesc.Format = DXGI_FORMAT_B8G8R8X8_UNORM; // X는 사용 안함
```

#### 타입 설명 (Suffix)

- **UNORM**: Unsigned Normalized (0~255 → 0.0~1.0)
- **SNORM**: Signed Normalized (-128~127 → -1.0~1.0)  
- **UINT**: Unsigned Integer (0~255 그대로)
- **SINT**: Signed Integer (-128~127 그대로)
- **FLOAT**: 부동소수점 (16비트 또는 32비트)

#### 실제 메모리 레이아웃 비교

```cpp
// 빨간색 픽셀 (255, 0, 0, 255)을 저장할 때

// R8G8B8A8_UNORM: 메모리에서 [FF][00][00][FF]
uint32_t rgba = 0xFF0000FF; // Red=255, Green=0, Blue=0, Alpha=255

// B8G8R8A8_UNORM: 메모리에서 [00][00][FF][FF]  
uint32_t bgra = 0x0000FFFF; // Blue=0, Green=0, Red=255, Alpha=255
```

#### 성능 고려사항

**R8G8B8A8_UNORM을 사용해야 하는 경우:**
- 표준 3D 렌더링
- OpenGL과 호환성이 필요한 경우
- 크로스 플랫폼 개발

**B8G8R8A8_UNORM을 사용해야 하는 경우:**
- Windows 전용 애플리케이션
- UI 렌더링이 주요 목적
- Desktop Window Manager와의 호환성
- Windows에서 최적화된 성능이 필요한 경우

### 3. Refresh Rate (주사율)
```cpp
swapChainDesc.BufferDesc.RefreshRate.Numerator = 60;    // 분자
swapChainDesc.BufferDesc.RefreshRate.Denominator = 1;   // 분모
// 결과: 60Hz (초당 60프레임)
```

### 4. Swap Effect (스왑 방식)

#### SwapEffect 종류와 특징 비교표

| SwapEffect | 모델 | 최소 BufferCount | 성능 | 호환성 | 특징 |
|------------|------|------------------|------|--------|------|
| **DISCARD** | BitBlt | 1 | 기본 | 최고 | 가장 일반적, 모든 OS 지원 |
| **SEQUENTIAL** | BitBlt | 1 | 기본 | 높음 | 이전 프레임 내용 보존 |
| **FLIP_SEQUENTIAL** | FLIP | 2 | 높음 | 중간 | Windows 8+, 낮은 지연시간 |
| **FLIP_DISCARD** | FLIP | 2 | 최고 | 낮음 | Windows 10+, VRR/HDR 지원 |

#### 각 SwapEffect 상세 설명

##### DXGI_SWAP_EFFECT_DISCARD (BitBlt)
```cpp
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
```
- **사용 시기**: 매 프레임 전체 화면을 다시 그리는 대부분의 게임
- **장점**: 가장 높은 호환성, 메모리 사용량 적음
- **단점**: 기본적인 성능

##### DXGI_SWAP_EFFECT_SEQUENTIAL (BitBlt)
```cpp
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_SEQUENTIAL;
```
- **사용 시기**: UI 애플리케이션이나 부분 렌더링이 필요한 경우
- **장점**: 이전 프레임 내용 보존, 부분 업데이트 가능
- **단점**: 메모리 사용량 증가

##### DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL (FLIP)
```cpp
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL;
```
- **사용 시기**: 고성능이 필요한 최신 게임 (Windows 8+)
- **장점**: 하드웨어 레벨 최적화, 낮은 지연시간, 전체화면 최적화
- **단점**: BufferCount ≥ 2 필수, MSAA 불가

##### DXGI_SWAP_EFFECT_FLIP_DISCARD (FLIP)
```cpp
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;
```
- **사용 시기**: 최신 하드웨어에서 최고 성능이 필요한 경우 (Windows 10+)
- **장점**: 최고 성능, 최저 지연시간, VRR/Auto HDR 지원
- **단점**: BufferCount ≥ 2 필수, MSAA 불가, 제한된 호환성

#### SwapEffect 선택 가이드

```cpp
// OS 버전에 따른 자동 선택 예시
DXGI_SWAP_EFFECT GetOptimalSwapEffect() {
    if (IsWindows10OrGreater()) {
        return DXGI_SWAP_EFFECT_FLIP_DISCARD;    // 최고 성능
    }
    else if (IsWindows8OrGreater()) {
        return DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL; // 균형잡힌 성능
    }
    else {
        return DXGI_SWAP_EFFECT_DISCARD;         // 호환성 우선
    }
}
```

## FLIP 모델 상세 분석

### FLIP 모델의 핵심 개념
FLIP 모델에서는 별도의 프론트 버퍼가 없습니다. 대신 백버퍼들이 front/back 역할을 서로 교체합니다:

```
BitBlt 모델 (복사 방식):
백버퍼 → [복사] → 프론트버퍼 (별도 존재)

FLIP 모델 (역할 교체):
백버퍼 A ↔ 백버퍼 B (역할만 바뀜, 복사 없음)
```

### FLIP 모델이 BufferCount ≥ 2를 요구하는 이유

#### 불가능한 시나리오 (BufferCount = 1)
```cpp
// FLIP 모델에서 BufferCount = 1은 불가능
Back Buffer A: [화면 표시 중] + [새로운 프레임 그리기] 
// → 동시에 읽기(표시)와 쓰기(렌더링)를 할 수 없음!
```

#### 가능한 시나리오 (BufferCount = 2)
```cpp
// FLIP 더블 버퍼링 - 최소 구성
Back Buffer A: [Front 역할] ← 화면에 표시 중, GPU 접근 불가
Back Buffer B: [렌더링 역할] ← GPU가 새로운 프레임 그리는 중

// Present() 후 역할 교체
Back Buffer A: [렌더링 역할] ← 이제 GPU가 그림
Back Buffer B: [Front 역할] ← 이제 화면에 표시
```

#### 이상적인 시나리오 (BufferCount = 3)
```cpp
// FLIP 트리플 버퍼링 - 권장 구성
Back Buffer A: [Front 역할] ← 화면에 표시 중
Back Buffer B: [렌더링 역할] ← GPU가 현재 작업 중  
Back Buffer C: [대기 역할] ← 즉시 다음 렌더링 시작 가능

// 장점: GPU 대기 시간 없음, 최고 성능
```

### 성능과 메모리 비교

#### 메모리 대역폭 절약
```cpp
// 1920x1080 32비트 화면 기준
BitBlt 복사 비용: 1920 × 1080 × 4 = 8.3MB per frame
60fps에서: 8.3MB × 60 = 498MB/초 메모리 대역폭 사용

FLIP 포인터 교체: 거의 0MB/초 메모리 대역폭 사용
```

#### 메모리 사용량 비교
```cpp
// 1920x1080 화면 기준 메모리 사용량 (픽셀당 4바이트)

BitBlt 모델:
- 더블 버퍼링 (BufferCount=1): 16.6MB (백1 + 프론트1)
- 트리플 버퍼링 (BufferCount=2): 24.9MB (백2 + 프론트1)

FLIP 모델:
- 더블 버퍼링 (BufferCount=2): 16.6MB (백2개만)
- 트리플 버퍼링 (BufferCount=3): 24.9MB (백3개만)

// FLIP 모델의 메모리 효율성:
// BitBlt 트리플 버퍼링 = FLIP 더블 버퍼링 (같은 메모리로 더 좋은 성능)
```

#### FLIP 모델 (FLIP_SEQUENTIAL, FLIP_DISCARD)
```
┌─────────────┐    포인터   ┌─────────────┐
│Back Buffer A│ <--------> │Back Buffer B│
│ (현재 렌더링) │   교체     │(Front 역할) │
└─────────────┘            └─────────────┘
   BufferCount = 2 (백버퍼 2개) 필수
   
   별도의 프론트 버퍼 없음 - 백버퍼가 역할 교체
```
- **백버퍼들 간의 역할만 교체** (복사 없음)
- Present() 호출 시 백버퍼 중 하나가 front buffer 역할로 바뀜
- **물리적으로는 모두 백버퍼**, 논리적으로만 front/back 역할 분담
- **BufferCount = 2 (백버퍼 2개) = 더블 버퍼링** (별도 프론트버퍼 없음)

**FLIP 모델의 핵심**: 별도의 프론트 버퍼가 없습니다. 대신 백버퍼들이 front/back 역할을 서로 교체합니다.
- 렌더링 중인 백버퍼 → Present() 후 → front buffer 역할 (화면 표시)
- front buffer 역할인 백버퍼 → Present() 후 → 렌더링용 백버퍼 역할

### 구체적인 동작 과정

#### FLIP 모델에서 2개 백버퍼 동작 (BufferCount = 2)

**중요**: 2개 백버퍼 환경에서는 새로운 렌더링을 위해 **대기가 필요**할 수 있습니다.

```cpp
// 초기 상태 - 총 2개의 백버퍼만 존재 (별도 프론트 버퍼 없음)
// (이전 Present() 호출로 Buffer B가 front buffer 역할 중이라고 가정)
Back Buffer A: [GPU가 새로운 프레임 그리는 중] (렌더링 역할)
Back Buffer B: [Front Buffer 역할] ← 화면에 표시 중, GPU 접근 불가

// Present() 호출 시 - Buffer A 렌더링 완료 후 역할 교체
Back Buffer A: [Front Buffer 역할] ← 이제 화면에 표시, GPU 접근 불가  
Back Buffer B: [렌더링 역할] ← 이제 새로운 프레임 그리기 시작

// 다음 Present() 호출 시 - Buffer B 렌더링 완료 후 역할 교체
Back Buffer A: [렌더링 역할] ← 다시 새로운 프레임 그리기 시작
Back Buffer B: [Front Buffer 역할] ← 다시 화면에 표시, GPU 접근 불가

// 핵심: 물리적으론 모두 백버퍼, 논리적으로만 front/back 역할 교체
```

#### 애플리케이션 시작부터의 완전한 시나리오

```cpp
// 1. 애플리케이션 시작 직후 (최초 상태)
// 화면: [빈 화면 또는 이전 애플리케이션 내용]
Back Buffer A: [사용 가능] ← 첫 번째 프레임 렌더링 시작
Back Buffer B: [사용 가능] ← 아직 사용되지 않음

// 2. 첫 번째 프레임 렌더링 완료 후 Present() 호출
Back Buffer A: [Front Buffer 역할] ← 첫 번째 프레임이 화면에 표시
Back Buffer B: [렌더링 역할] ← 두 번째 프레임 렌더링 시작

// 3. 두 번째 프레임 렌더링 완료 후 Present() 호출  
Back Buffer A: [렌더링 역할] ← 세 번째 프레임 렌더링 시작
Back Buffer B: [Front Buffer 역할] ← 두 번째 프레임이 화면에 표시

// 4. 이후 계속 A ↔ B 역할 교체하며 렌더링
// → 이 상태가 앞서 설명한 "초기 상태"에 해당
```

**핵심**: FLIP에서는 **동시 읽기/쓰기가 불가능**합니다. Front buffer 역할인 백버퍼는 
GPU가 접근할 수 없으며, GPU는 항상 **다른 백버퍼**에 새로운 프레임을 그립니다.

**BufferCount = 2의 한계**:
- GPU가 빠르게 렌더링하면 사용 가능한 백버퍼가 없어 **대기 발생**
- 진정한 트리플 버퍼링의 이점을 얻으려면 **BufferCount = 3 권장**
- 2개 백버퍼는 "최소 요구사항"이지, "최적 성능"은 아님

#### BufferCount = 3일 때의 이상적인 동작 (권장)
```cpp
// 3개 백버퍼 환경 - 항상 렌더링 가능한 버퍼 보장
Back Buffer A: [Front Buffer 역할] ← 화면에 표시 중, GPU 접근 불가
Back Buffer B: [렌더링 역할] ← GPU가 현재 작업 중
Back Buffer C: [대기 역할] ← 즉시 다음 렌더링 시작 가능

// Present() 후에도 항상 사용 가능한 버퍼가 1개 이상 보장됨
// → GPU 대기 시간 최소화, 최고 성능
```

#### FLIP 모델의 정확한 동작 메커니즘

```cpp
// 상세한 Present() 동작 과정
1. GPU가 Back Buffer A에서 렌더링 완료
2. Present() 호출
3. Buffer A가 Front Buffer 역할로 전환 (화면 표시 시작)
4. Buffer A는 이제 "표시 전용" 상태 → GPU 접근 금지
5. GPU의 새 렌더 타겟이 Buffer B로 변경
6. GPU가 Buffer B에서 새로운 프레임 렌더링 시작

// 핵심: 각 백버퍼는 한 번에 하나의 역할만 수행
- Front Buffer 역할(표시) 중인 백버퍼: GPU 접근 불가
- 렌더링 중인 백버퍼: 화면 표시 불가
- 대기 중인 백버퍼: 아무도 사용하지 않음 (BufferCount ≥ 3일 때만)
```

**FLIP이 최소 2개 백버퍼를 요구하는 이유**:
1. **역할 분리**: Front buffer 역할과 렌더링 역할이 **물리적으로 다른 백버퍼**에서 수행
2. **동시성 보장**: 한 백버퍼는 표시 전용, 다른 백버퍼는 렌더링 전용으로 역할 분담
3. **역할 교체 방식**: 실제 메모리 이동 없이 역할만 바뀌므로 각 백버퍼는 고정된 메모리 위치
4. **무중단 파이프라인**: Present() 호출 후에도 즉시 새로운 프레임 렌더링 시작 가능

#### 만약 백버퍼가 1개만 있다면? (BufferCount = 1)
```cpp
// 불가능한 상황 - FLIP 모델에서 BufferCount = 1
Back Buffer A: [Front Buffer 역할로 화면 표시 중] + [GPU가 그리려고 함]
// → 동시에 표시(읽기)와 렌더링(쓰기)을 할 수 없음!

// FLIP 모델의 제약
Present() 호출 시:
1. Buffer A가 Front Buffer 역할 시작 (화면 표시, 읽기 전용 상태)
2. GPU가 새 프레임을 그리려면 Buffer A에 써야 함
3. 하지만 Buffer A는 표시 중이므로 쓰기 불가능!
4. → 데드락 발생 또는 렌더링 중단
```

**FLIP에서 BufferCount = 1이 불가능한 이유**:
- 한 백버퍼가 Front Buffer 역할(표시) 중일 때 새로운 렌더링을 할 백버퍼가 없음
- 동시 읽기/쓰기 방지를 위해 물리적으로 분리된 백버퍼들이 필요
- 이것이 FLIP 계열 SwapEffect에서 BufferCount ≥ 2를 강제하는 이유

### 왜 복사 대신 포인터 교체를 사용하는가?

#### FLIP 모델의 정확한 동작 방식
FLIP 모델에서는 "백버퍼가 화면에 출력된다"는 표현보다는 "백버퍼 중 하나가 front buffer 역할을 한다"가 더 정확합니다:

```cpp
// 전통적인 이해 (부정확)
Present() → 백버퍼가 별도 프론트버퍼로 복사되어 화면에 출력

// 실제 FLIP 동작 (정확)
Present() → 백버퍼 중 하나가 front buffer 역할로 전환, 화면에 직접 표시
```

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
3. 시스템이 해당 백버퍼를 참조하도록 포인터 변경 ← 빠름!
4. 거의 즉시 완료
```

#### 메모리 대역폭 절약
```cpp
// 1920x1080 32비트 화면 기준
복사해야 할 데이터: 1920 × 1080 × 4 = 8.3MB
60fps에서: 8.3MB × 60 = 498MB/초 ← 대역폭 낭비

// FLIP에서는 복사가 없으므로 대역폭 절약
```

### 모델별 트리플 버퍼링의 차이점

#### BitBlt 모델의 트리플 버퍼링
```cpp
// BitBlt 모델 - 더블 버퍼링 (BufferCount = 1)
Front Buffer:  [화면 출력]     (별도 존재, 시스템 관리)
Back Buffer:   [GPU 렌더링]    (BufferCount = 1)
// 문제: Present() 후 GPU는 복사 완료까지 대기해야 함

// BitBlt 모델 - 트리플 버퍼링 (BufferCount = 2)
Front Buffer:   [화면 출력]      (별도 존재, 시스템 관리)
Back Buffer 1:  [GPU 렌더링]     
Back Buffer 2:  [대기 중]        
// 해결: 복사 중에도 다른 백버퍼에서 렌더링 가능
```

#### FLIP 모델의 "트리플" 버퍼링
```cpp
// FLIP 모델 - 더블 버퍼링 (BufferCount = 2)
Back Buffer 1:  [Front Buffer 역할] ← 화면에 표시 중
Back Buffer 2:  [렌더링 역할] ← GPU 작업 중
// 문제: GPU가 빠르면 사용 가능한 백버퍼가 없어 대기 발생

// FLIP 모델 - 트리플 버퍼링 (BufferCount = 3)
Back Buffer 1:  [Front Buffer 역할] ← 화면에 표시 중
Back Buffer 2:  [렌더링 역할] ← GPU 작업 중
Back Buffer 3:  [대기 역할] ← 항상 사용 가능! ← 핵심!
// 해결: 항상 사용 가능한 백버퍼 보장, GPU 대기 없음
```

**정확한 버퍼링 방식 정리**:

**BitBlt 모델**:
- **BufferCount = 1**: 더블 버퍼링 (프론트1 + 백1 = 총 2개)
- **BufferCount = 2**: 트리플 버퍼링 (프론트1 + 백2 = 총 3개)
- **BufferCount = 3**: 쿼드 버퍼링 (프론트1 + 백3 = 총 4개)

**FLIP 모델**:
- **BufferCount = 2**: 더블 버퍼링 (백버퍼 2개, 별도 프론트 없음)
- **BufferCount = 3**: 트리플 버퍼링 (백버퍼 3개, 별도 프론트 없음)
- **BufferCount = 4**: 쿼드 버퍼링 (백버퍼 4개, 별도 프론트 없음)

**성능 권장사항**:
- **BufferCount = 2**: FLIP 모델의 최소 요구사항 (대기 가능성 있음)
- **BufferCount = 3**: 최적 성능 (대기 없는 연속 렌더링)
- **BufferCount = 4+**: 메모리 낭비 (일반적으로 불필요)

### 실제 구현에서의 고려사항

```cpp
// FLIP 모델 생성 시 필수 조건
DXGI_SWAP_CHAIN_DESC swapChainDesc = {};
swapChainDesc.BufferCount = 2;  // 백버퍼 최소 2개!
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;

// 백버퍼를 1개로 설정하면 생성 실패
swapChainDesc.BufferCount = 1;  // ← 이렇게 하면 에러!
HRESULT hr = D3D11CreateDeviceAndSwapChain(...);
// hr == DXGI_ERROR_INVALID_CALL
```

**왜 백버퍼가 최소 2개 필요한가?**
1. **역할 분리**: Front buffer 역할인 백버퍼와 렌더링용 백버퍼가 **물리적으로 다름**
2. **동시성 보장**: 한 백버퍼는 표시 전용, 다른 백버퍼는 렌더링 전용으로 역할 분담
3. **역할 교체 방식**: 실제 메모리 이동 없이 역할만 바뀌므로 각 백버퍼는 고정된 메모리 위치
4. **무중단 파이프라인**: Present() 호출 후에도 즉시 새로운 프레임 렌더링 시작 가능

**FLIP의 장점**:
- **제로 카피**: 메모리 복사가 전혀 없어 매우 빠름
- **즉시 완료**: 역할 교체만으로 Present() 거의 즉시 완료
- **하드웨어 최적화**: GPU 하드웨어 레벨에서 직접 지원
- **메모리 효율성**: 별도 프론트버퍼가 없어 메모리 절약

### 메모리 사용량 비교

```cpp
// 1920x1080 화면 기준 메모리 사용량 (픽셀당 4바이트)

// BitBlt 모델 (별도 프론트버퍼 존재)
더블 버퍼링 (BufferCount=1): 8.3MB (백버퍼 1개) + 8.3MB (프론트버퍼) = 16.6MB
트리플 버퍼링 (BufferCount=2): 16.6MB (백버퍼 2개) + 8.3MB (프론트버퍼) = 24.9MB

// FLIP 모델 (백버퍼들이 역할 교체, 별도 프론트버퍼 없음)
더블 버퍼링 (BufferCount=2): 16.6MB (백버퍼 2개만, 별도 프론트버퍼 없음)
트리플 버퍼링 (BufferCount=3): 24.9MB (백버퍼 3개만, 별도 프론트버퍼 없음)
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

## 성능 최적화 요약

### 권장 설정

| 목표 | SwapEffect | BufferCount | 비고 |
|------|------------|-------------|------|
| **최대 호환성** | DISCARD | 1 | 모든 OS, 기본 성능 |
| **균형잡힌 성능** | FLIP_SEQUENTIAL | 2-3 | Windows 8+, 좋은 성능 |
| **최고 성능** | FLIP_DISCARD | 3 | Windows 10+, 최저 지연시간 |

### 성능 향상 정도
- **FLIP_SEQUENTIAL**: BitBlt 대비 약 10-30% 성능 향상
- **FLIP_DISCARD**: BitBlt 대비 약 20-50% 성능 향상
- **BufferCount = 3**: GPU 대기 시간 최소화

## 핵심 정리

### 스왑 체인의 핵심 개념
1. **스왑 체인**: GPU가 그린 이미지를 화면에 표시하는 메커니즘
2. **더블 버퍼링**: 화면 찢어짐 현상을 방지하는 기본 기법
3. **Present()**: 백버퍼를 실제로 화면에 표시하는 함수

### 모델별 차이점
- **BitBlt 모델**: 프론트 버퍼 별도 존재, 복사 방식, 높은 호환성
- **FLIP 모델**: 백버퍼 역할 교체, 포인터 방식, 높은 성능

### BufferCount 매핑
- **BitBlt**: BufferCount 1 = 더블버퍼링, 2 = 트리플버퍼링
- **FLIP**: BufferCount 2 = 더블버퍼링, 3 = 트리플버퍼링 (권장)

### 성능 최적화 핵심
- **호환성 우선**: DISCARD + BufferCount 1
- **균형**: FLIP_SEQUENTIAL + BufferCount 2
- **최고성능**: FLIP_DISCARD + BufferCount 3

### 실무 주의사항
- **창 크기 변경** 시 ResizeBuffers() 호출 필수
- **FLIP 모델**에서는 MSAA 사용 불가
- **VSync** 설정으로 성능과 품질 조절
- **렌더 타겟 뷰**를 통해 백버퍼에 그리기

스왑 체인은 DirectX 11에서 화면 출력을 담당하는 핵심 구성 요소입니다. BitBlt vs FLIP 모델의 차이점과 BufferCount 설정을 올바르게 이해하면 부드럽고 효율적인 렌더링을 구현할 수 있습니다!