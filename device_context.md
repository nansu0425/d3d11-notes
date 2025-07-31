# DirectX 11 Device Context (디바이스 컨텍스트)

## 디바이스 컨텍스트란 무엇인가?

DirectX 11에서 **Device Context**는 GPU에 실제 명령을 내리는 "지휘관" 역할을 합니다. Device가 리소스(버퍼, 텍스처, 셰이더 등)를 생성하는 "공장장"이라면, Device Context는 이 리소스들을 사용해서 실제로 그래픽을 그리는 "화가"입니다.

쉽게 비유하면:
- **Device**: 붓, 물감, 캔버스를 준비하는 사람
- **Device Context**: 실제로 그림을 그리는 화가

Device Context 없이는 아무리 좋은 리소스가 있어도 화면에 아무것도 그릴 수 없습니다. 모든 렌더링 명령은 반드시 Device Context를 통해서만 실행됩니다.

## Device Context가 필요한 이유

### 문제상황: GPU 명령의 복잡성
GPU에서 무언가를 그리려면 수많은 설정과 명령이 필요합니다:

```
1. 어떤 정점 버퍼를 사용할지?
2. 어떤 셰이더를 실행할지?
3. 어떤 텍스처를 바인딩할지?
4. 어떤 렌더 타겟에 그릴지?
5. 실제 그리기 명령 실행
```

이 모든 것을 매번 개별적으로 관리하면 매우 복잡하고 실수하기 쉽습니다.

### 해결책: 통합된 렌더링 인터페이스
Device Context는 이 모든 복잡한 과정을 하나의 일관된 인터페이스로 제공합니다:

```cpp
// Device Context를 통한 간단한 렌더링
deviceContext->IASetVertexBuffers(0, 1, &vertexBuffer, &stride, &offset);
deviceContext->VSSetShader(vertexShader, nullptr, 0);
deviceContext->PSSetShader(pixelShader, nullptr, 0);
deviceContext->Draw(vertexCount, 0);
```

## Device Context의 두 가지 타입

DirectX 11에는 두 종류의 Device Context가 있습니다:

### 1. Immediate Context (즉시 컨텍스트)
```cpp
ID3D11DeviceContext* immediateContext;
device->GetImmediateContext(&immediateContext);
```

**특징:**
- Device 생성 시 자동으로 하나 만들어짐
- 명령을 호출하면 즉시 GPU에 전달
- 단일 스레드에서만 사용 가능
- 가장 일반적으로 사용되는 타입

### 2. Deferred Context (지연 컨텍스트)
```cpp
ID3D11DeviceContext* deferredContext;
device->CreateDeferredContext(0, &deferredContext);
```

**특징:**
- 명령을 즉시 실행하지 않고 기록만 함
- 여러 스레드에서 동시에 사용 가능
- Command List를 생성하여 나중에 실행
- 멀티스레딩 렌더링에 사용

## Device Context의 주요 역할

### 1. 파이프라인 상태 설정
그래픽 파이프라인의 각 단계에 필요한 리소스들을 바인딩합니다.

### 2. 렌더링 명령 실행
실제로 정점을 그리는 명령을 GPU에 전달합니다.

### 3. 리소스 업데이트
버퍼나 텍스처의 내용을 동적으로 변경합니다.

### 4. GPU와 CPU 동기화
GPU 작업의 완료를 기다리거나 데이터를 복사합니다.

## 그래픽 파이프라인과 Device Context

DirectX 11 그래픽 파이프라인의 각 단계마다 Device Context가 제공하는 함수들이 있습니다:

```
입력 어셈블러 (Input Assembler)
       ↓
정점 셰이더 (Vertex Shader)
       ↓
헐 셰이더 (Hull Shader) - 선택적
       ↓
테셀레이터 (Tessellator) - 고정 기능
       ↓
도메인 셰이더 (Domain Shader) - 선택적
       ↓
지오메트리 셰이더 (Geometry Shader) - 선택적
       ↓
래스터라이저 (Rasterizer) - 고정 기능
       ↓
픽셀 셰이더 (Pixel Shader)
       ↓
출력 머지 (Output Merger)
```

### 1. Input Assembler (IA) 단계

#### 정점 버퍼 바인딩
```cpp
// 정점 버퍼 설정
UINT stride = sizeof(Vertex);
UINT offset = 0;
deviceContext->IASetVertexBuffers(
    0,                  // 시작 슬롯
    1,                  // 버퍼 개수
    &vertexBuffer,      // 버퍼 배열
    &stride,           // 각 정점의 크기
    &offset            // 시작 오프셋
);
```

**내부 동작:**
1. GPU의 Input Assembler에 정점 버퍼 주소 전달
2. stride와 offset 정보로 정점 읽기 방식 설정
3. 다음 그리기 명령 시 이 버퍼에서 정점 데이터 읽음

#### 인덱스 버퍼 바인딩
```cpp
deviceContext->IASetIndexBuffer(
    indexBuffer,               // 인덱스 버퍼
    DXGI_FORMAT_R32_UINT,     // 인덱스 형식 (32비트 unsigned int)
    0                         // 오프셋
);
```

#### 입력 레이아웃 설정
```cpp
deviceContext->IASetInputLayout(inputLayout);
```

**역할:**
- 정점 버퍼의 데이터 구조를 GPU에게 알려줌
- 셰이더 입력과 정점 데이터를 올바르게 매핑

#### 프리미티브 토폴로지 설정
```cpp
deviceContext->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
```

**토폴로지 옵션:**
- `D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST`: 독립적인 삼각형들
- `D3D11_PRIMITIVE_TOPOLOGY_TRIANGLESTRIP`: 연결된 삼각형 띠
- `D3D11_PRIMITIVE_TOPOLOGY_LINELIST`: 선분들
- `D3D11_PRIMITIVE_TOPOLOGY_POINTLIST`: 점들

### 2. Vertex Shader (VS) 단계

#### 정점 셰이더 바인딩
```cpp
deviceContext->VSSetShader(
    vertexShader,      // 정점 셰이더 객체
    nullptr,           // 클래스 인스턴스 (일반적으로 nullptr)
    0                  // 클래스 인스턴스 개수
);
```

#### 상수 버퍼 바인딩
```cpp
deviceContext->VSSetConstantBuffers(
    0,                 // 시작 슬롯 (b0, b1, b2...)
    1,                 // 버퍼 개수
    &constantBuffer    // 상수 버퍼 배열
);
```

**상수 버퍼 업데이트 과정:**
```cpp
// 1. 상수 버퍼 맵핑 (CPU에서 접근 가능하게)
D3D11_MAPPED_SUBRESOURCE mappedResource;
HRESULT hr = deviceContext->Map(
    constantBuffer,                // 업데이트할 버퍼
    0,                            // 서브리소스 인덱스
    D3D11_MAP_WRITE_DISCARD,      // 맵핑 타입
    0,                            // 맵핑 플래그
    &mappedResource               // 맵핑된 메모리 정보
);

if (SUCCEEDED(hr)) {
    // 2. 데이터 복사
    ConstantBufferData* cbData = (ConstantBufferData*)mappedResource.pData;
    cbData->worldMatrix = worldMatrix;
    cbData->viewMatrix = viewMatrix;
    cbData->projectionMatrix = projectionMatrix;
    
    // 3. 맵핑 해제 (GPU가 다시 접근 가능하게)
    deviceContext->Unmap(constantBuffer, 0);
}
```

### 3. Pixel Shader (PS) 단계

#### 픽셀 셰이더 바인딩
```cpp
deviceContext->PSSetShader(pixelShader, nullptr, 0);
```

#### 텍스처 리소스 바인딩
```cpp
deviceContext->PSSetShaderResources(
    0,                    // 시작 슬롯 (t0, t1, t2...)
    1,                    // 리소스 개수
    &textureResourceView  // 셰이더 리소스 뷰 배열
);
```

#### 샘플러 상태 바인딩
```cpp
deviceContext->PSSetSamplers(
    0,              // 시작 슬롯 (s0, s1, s2...)
    1,              // 샘플러 개수
    &samplerState   // 샘플러 상태 배열
);
```

**샘플러 상태 생성 예시:**
```cpp
D3D11_SAMPLER_DESC samplerDesc = {};
samplerDesc.Filter = D3D11_FILTER_MIN_MAG_MIP_LINEAR;  // 선형 필터링
samplerDesc.AddressU = D3D11_TEXTURE_ADDRESS_WRAP;     // U 방향 래핑
samplerDesc.AddressV = D3D11_TEXTURE_ADDRESS_WRAP;     // V 방향 래핑
samplerDesc.AddressW = D3D11_TEXTURE_ADDRESS_WRAP;     // W 방향 래핑
samplerDesc.ComparisonFunc = D3D11_COMPARISON_NEVER;
samplerDesc.MinLOD = 0;
samplerDesc.MaxLOD = D3D11_FLOAT32_MAX;

ID3D11SamplerState* samplerState;
device->CreateSamplerState(&samplerDesc, &samplerState);
```

### 4. Output Merger (OM) 단계

#### 렌더 타겟 설정
```cpp
deviceContext->OMSetRenderTargets(
    1,                    // 렌더 타겟 개수
    &renderTargetView,    // 렌더 타겟 뷰 배열
    depthStencilView      // 깊이-스텐실 뷰
);
```

#### 블렌드 상태 설정
```cpp
float blendFactor[4] = {1.0f, 1.0f, 1.0f, 1.0f};
deviceContext->OMSetBlendState(
    blendState,        // 블렌드 상태 객체
    blendFactor,       // 블렌드 팩터
    0xffffffff         // 샘플 마스크
);
```

#### 깊이-스텐실 상태 설정
```cpp
deviceContext->OMSetDepthStencilState(
    depthStencilState,  // 깊이-스텐실 상태
    1                   // 스텐실 참조값
);
```

## 렌더링 명령 함수들

Device Context의 핵심은 실제 그리기 명령을 실행하는 것입니다.

### 1. Draw (기본 그리기)
```cpp
deviceContext->Draw(
    vertexCount,    // 그릴 정점 개수
    startVertex     // 시작 정점 인덱스
);
```

**내부 동작 과정:**
```
1. Input Assembler가 정점 버퍼에서 데이터 읽기
2. 각 정점에 대해 Vertex Shader 실행
3. 프리미티브 어셈블리 (정점들을 삼각형으로 조합)
4. 래스터화 (삼각형을 픽셀로 변환)
5. 각 픽셀에 대해 Pixel Shader 실행
6. Output Merger에서 최종 색상 계산 및 출력
```

### 2. DrawIndexed (인덱스 기반 그리기)
```cpp
deviceContext->DrawIndexed(
    indexCount,         // 그릴 인덱스 개수
    startIndexLocation, // 시작 인덱스 위치
    baseVertexLocation  // 정점 오프셋
);
```

**장점:**
- 정점 재사용으로 메모리 절약
- 복잡한 메시에서 성능 향상

**예시:**
```cpp
// 사각형을 그리기 위한 4개 정점과 6개 인덱스
Vertex vertices[4] = {
    {{-1, -1, 0}, {0, 1}},  // 좌하
    {{-1,  1, 0}, {0, 0}},  // 좌상
    {{ 1,  1, 0}, {1, 0}},  // 우상
    {{ 1, -1, 0}, {1, 1}}   // 우하
};

UINT indices[6] = {
    0, 1, 2,    // 첫 번째 삼각형
    0, 2, 3     // 두 번째 삼각형
};

deviceContext->DrawIndexed(6, 0, 0);  // 6개 인덱스로 2개 삼각형 그리기
```

### 3. DrawInstanced (인스턴스 렌더링)
```cpp
deviceContext->DrawInstanced(
    vertexCountPerInstance,  // 인스턴스당 정점 개수
    instanceCount,           // 인스턴스 개수
    startVertexLocation,     // 시작 정점 위치
    startInstanceLocation    // 시작 인스턴스 위치
);
```

**사용 목적:**
- 동일한 메시를 여러 번 그리기 (나무, 풀, 적 유닛 등)
- GPU에서 인스턴스별 변환 처리
- 드로우 콜 횟수 대폭 감소

**인스턴스 데이터 예시:**
```cpp
struct InstanceData {
    DirectX::XMFLOAT4X4 worldMatrix;  // 각 인스턴스의 변환 행렬
    DirectX::XMFLOAT4 color;          // 각 인스턴스의 색상
};

// 인스턴스 버퍼 생성
D3D11_BUFFER_DESC instanceBufferDesc = {};
instanceBufferDesc.Usage = D3D11_USAGE_DYNAMIC;
instanceBufferDesc.ByteWidth = sizeof(InstanceData) * instanceCount;
instanceBufferDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;
instanceBufferDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;

ID3D11Buffer* instanceBuffer;
device->CreateBuffer(&instanceBufferDesc, nullptr, &instanceBuffer);

// 인스턴스 버퍼 바인딩 (슬롯 1에)
UINT strides[2] = {sizeof(Vertex), sizeof(InstanceData)};
UINT offsets[2] = {0, 0};
ID3D11Buffer* buffers[2] = {vertexBuffer, instanceBuffer};

deviceContext->IASetVertexBuffers(0, 2, buffers, strides, offsets);
```

### 4. DrawIndexedInstanced (인덱스 + 인스턴스 렌더링)
```cpp
deviceContext->DrawIndexedInstanced(
    indexCountPerInstance,   // 인스턴스당 인덱스 개수
    instanceCount,           // 인스턴스 개수
    startIndexLocation,      // 시작 인덱스 위치
    baseVertexLocation,      // 정점 오프셋
    startInstanceLocation    // 시작 인스턴스 위치
);
```

**최고의 성능:**
- 정점 재사용 + 인스턴스 렌더링의 조합
- 대량의 동일 객체 렌더링에 최적

## 리소스 업데이트 함수들

### 1. Map/Unmap (동적 리소스 업데이트)

#### Map 함수
```cpp
D3D11_MAPPED_SUBRESOURCE mappedResource;
HRESULT hr = deviceContext->Map(
    resource,                    // 업데이트할 리소스
    subresourceIndex,           // 서브리소스 인덱스 (보통 0)
    mapType,                    // 맵핑 타입
    mapFlags,                   // 맵핑 플래그
    &mappedResource             // 맵핑된 메모리 정보
);
```

#### Map 타입들
```cpp
// 1. WRITE_DISCARD: 기존 데이터 무시하고 새로 쓰기 (가장 빠름)
D3D11_MAP_WRITE_DISCARD

// 2. WRITE_NO_OVERWRITE: 사용하지 않는 부분만 업데이트
D3D11_MAP_WRITE_NO_OVERWRITE

// 3. READ: 데이터 읽기 (STAGING 리소스만 가능)
D3D11_MAP_READ

// 4. WRITE: 데이터 쓰기
D3D11_MAP_WRITE
```

#### 실제 사용 예시
```cpp
// 상수 버퍼 업데이트
void UpdateConstantBuffer(ID3D11DeviceContext* context, 
                         ID3D11Buffer* constantBuffer,
                         const ConstantBufferData& data) {
    D3D11_MAPPED_SUBRESOURCE mapped;
    HRESULT hr = context->Map(constantBuffer, 0, D3D11_MAP_WRITE_DISCARD, 0, &mapped);
    
    if (SUCCEEDED(hr)) {
        // 메모리 복사
        memcpy(mapped.pData, &data, sizeof(ConstantBufferData));
        
        // 맵핑 해제
        context->Unmap(constantBuffer, 0);
    }
}
```

### 2. UpdateSubresource (즉시 업데이트)
```cpp
deviceContext->UpdateSubresource(
    resource,           // 업데이트할 리소스
    subresourceIndex,   // 서브리소스 인덱스
    destBox,           // 업데이트할 영역 (nullptr = 전체)
    srcData,           // 소스 데이터
    srcRowPitch,       // 한 행의 바이트 수
    srcDepthPitch      // 한 깊이 슬라이스의 바이트 수
);
```

**Map vs UpdateSubresource:**
- **Map**: 큰 데이터, 자주 업데이트할 때 효율적
- **UpdateSubresource**: 작은 데이터, 가끔 업데이트할 때 간편

### 3. CopyResource (리소스 복사)
```cpp
// 전체 리소스 복사
deviceContext->CopyResource(destResource, srcResource);

// 부분 리소스 복사
deviceContext->CopySubresourceRegion(
    destResource,       // 대상 리소스
    destSubresource,    // 대상 서브리소스
    destX, destY, destZ, // 대상 위치
    srcResource,        // 소스 리소스
    srcSubresource,     // 소스 서브리소스
    srcBox             // 소스 영역
);
```

## 렌더링 상태 관리

### 1. 뷰포트 설정
```cpp
D3D11_VIEWPORT viewport = {};
viewport.TopLeftX = 0.0f;
viewport.TopLeftY = 0.0f;
viewport.Width = static_cast<float>(windowWidth);
viewport.Height = static_cast<float>(windowHeight);
viewport.MinDepth = 0.0f;
viewport.MaxDepth = 1.0f;

deviceContext->RSSetViewports(1, &viewport);
```

**뷰포트의 역할:**
- 3D 좌표를 화면 픽셀 좌표로 변환
- 렌더링할 화면 영역 정의
- 깊이 범위 설정

### 2. 시저 사각형 (Scissor Rectangle)
```cpp
D3D11_RECT scissorRect = {0, 0, windowWidth, windowHeight};
deviceContext->RSSetScissorRects(1, &scissorRect);
```

**용도:**
- 렌더링을 특정 영역으로 제한
- UI 클리핑, 그림자 렌더링 등에 사용

### 3. 래스터라이저 상태
```cpp
deviceContext->RSSetState(rasterizerState);
```

**설정 가능한 항목:**
- Fill Mode (SOLID, WIREFRAME)
- Cull Mode (FRONT, BACK, NONE)
- 깊이 클리핑, 시저 테스트 등

## 완전한 렌더링 순서

이제 모든 내용을 종합하여 완전한 렌더링 과정을 살펴보겠습니다:

### 1. 기본 렌더링 순서
```cpp
void Render() {
    // 1. 렌더 타겟 클리어
    float clearColor[4] = {0.0f, 0.2f, 0.4f, 1.0f};  // 진한 파란색
    deviceContext->ClearRenderTargetView(renderTargetView, clearColor);
    deviceContext->ClearDepthStencilView(depthStencilView, 
        D3D11_CLEAR_DEPTH | D3D11_CLEAR_STENCIL, 1.0f, 0);
    
    // 2. 파이프라인 상태 설정
    SetupInputAssembler();
    SetupVertexShader();
    SetupPixelShader();
    SetupOutputMerger();
    
    // 3. 실제 그리기
    deviceContext->DrawIndexed(indexCount, 0, 0);
    
    // 4. 화면에 출력
    swapChain->Present(1, 0);  // V-Sync 사용
}

void SetupInputAssembler() {
    // 정점 버퍼 바인딩
    UINT stride = sizeof(Vertex);
    UINT offset = 0;
    deviceContext->IASetVertexBuffers(0, 1, &vertexBuffer, &stride, &offset);
    
    // 인덱스 버퍼 바인딩
    deviceContext->IASetIndexBuffer(indexBuffer, DXGI_FORMAT_R32_UINT, 0);
    
    // 입력 레이아웃 설정
    deviceContext->IASetInputLayout(inputLayout);
    
    // 프리미티브 토폴로지 설정
    deviceContext->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
}

void SetupVertexShader() {
    // 정점 셰이더 바인딩
    deviceContext->VSSetShader(vertexShader, nullptr, 0);
    
    // 상수 버퍼 업데이트 및 바인딩
    UpdateConstantBuffer();
    deviceContext->VSSetConstantBuffers(0, 1, &constantBuffer);
}

void SetupPixelShader() {
    // 픽셀 셰이더 바인딩
    deviceContext->PSSetShader(pixelShader, nullptr, 0);
    
    // 텍스처 및 샘플러 바인딩
    deviceContext->PSSetShaderResources(0, 1, &textureResourceView);
    deviceContext->PSSetSamplers(0, 1, &samplerState);
}

void SetupOutputMerger() {
    // 렌더 타겟 설정
    deviceContext->OMSetRenderTargets(1, &renderTargetView, depthStencilView);
    
    // 블렌드 상태 설정
    float blendFactor[4] = {1.0f, 1.0f, 1.0f, 1.0f};
    deviceContext->OMSetBlendState(blendState, blendFactor, 0xffffffff);
    
    // 깊이-스텐실 상태 설정
    deviceContext->OMSetDepthStencilState(depthStencilState, 1);
}
```

### 2. 다중 객체 렌더링
```cpp
void RenderMultipleObjects() {
    // 1. 공통 설정 (한 번만)
    SetupCommonState();
    
    // 2. 각 객체별로 렌더링
    for (const auto& object : objects) {
        // 객체별 상수 버퍼 업데이트
        UpdateObjectConstants(object);
        
        // 객체별 텍스처 바인딩
        deviceContext->PSSetShaderResources(0, 1, &object.texture);
        
        // 객체별 정점/인덱스 버퍼 바인딩
        UINT stride = sizeof(Vertex);
        UINT offset = 0;
        deviceContext->IASetVertexBuffers(0, 1, &object.vertexBuffer, &stride, &offset);
        deviceContext->IASetIndexBuffer(object.indexBuffer, DXGI_FORMAT_R32_UINT, 0);
        
        // 그리기
        deviceContext->DrawIndexed(object.indexCount, 0, 0);
    }
    
    // 3. 화면 출력
    swapChain->Present(1, 0);
}
```

## Device Context 내부 동작 과정

### Draw 호출 시 내부에서 일어나는 일

```cpp
deviceContext->DrawIndexed(indexCount, 0, 0);
```

이 한 줄 뒤에서는 복잡한 과정이 진행됩니다:

#### 1단계: 명령 큐잉 (Command Queuing)
```
CPU 측:
1. DirectX Runtime이 렌더링 명령 생성
2. 드라이버가 GPU 명령어로 번역
3. GPU 명령 큐에 추가
4. 함수 즉시 리턴 (비동기)
```

#### 2단계: GPU 파이프라인 실행
```
GPU 측 (병렬 처리):
1. Input Assembler: 인덱스로 정점 데이터 수집
2. Vertex Shader: 각 정점에 대해 변환 계산
3. Primitive Assembly: 정점들을 삼각형으로 조합
4. Rasterizer: 삼각형을 픽셀들로 분해
5. Pixel Shader: 각 픽셀의 색상 계산
6. Output Merger: 깊이 테스트, 블렌딩 등 최종 처리
```

#### 3단계: 메모리 쓰기
```
렌더 타겟에 최종 픽셀 색상 저장
깊이 버퍼에 깊이 값 저장
```

### Map 호출 시 내부 동작

```cpp
deviceContext->Map(buffer, 0, D3D11_MAP_WRITE_DISCARD, 0, &mapped);
```

#### WRITE_DISCARD 모드의 경우:
```
1. GPU가 현재 버퍼를 사용 중인지 확인
2. 사용 중이면 새로운 메모리 블록 할당 (리네이밍)
3. CPU가 접근 가능한 메모리 주소 반환
4. CPU가 데이터 쓰기 완료 후 Unmap 호출
5. GPU가 다음 렌더링 시 새 버퍼 사용
```

**리네이밍의 장점:**
- GPU와 CPU가 동시에 작업 가능 (병렬성)
- 동기화 대기 시간 최소화
- 성능 향상

## 성능 최적화 팁

### 1. 드로우 콜 최소화
```cpp
// 나쁜 예: 각 객체마다 별도 그리기
for (auto& object : objects) {
    SetupObject(object);
    deviceContext->DrawIndexed(object.indexCount, 0, 0);  // 드로우 콜 많음
}

// 좋은 예: 인스턴스 렌더링 사용
SetupInstanceBuffer(objects);
deviceContext->DrawIndexedInstanced(
    indexCountPerInstance, 
    objects.size(),  // 모든 객체를 한 번에
    0, 0, 0
);
```

### 2. 상태 변경 최소화
```cpp
// 나쁜 예: 상태를 자주 변경
deviceContext->PSSetShader(shader1, nullptr, 0);
DrawObject1();
deviceContext->PSSetShader(shader2, nullptr, 0);
DrawObject2();
deviceContext->PSSetShader(shader1, nullptr, 0);
DrawObject3();

// 좋은 예: 상태별로 그룹화
deviceContext->PSSetShader(shader1, nullptr, 0);
DrawObject1();
DrawObject3();
deviceContext->PSSetShader(shader2, nullptr, 0);
DrawObject2();
```

### 3. 적절한 버퍼 사용법
```cpp
// 자주 변경되는 데이터: DYNAMIC
D3D11_BUFFER_DESC dynamicDesc = {};
dynamicDesc.Usage = D3D11_USAGE_DYNAMIC;
dynamicDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;

// 정적 데이터: IMMUTABLE
D3D11_BUFFER_DESC immutableDesc = {};
immutableDesc.Usage = D3D11_USAGE_IMMUTABLE;
immutableDesc.CPUAccessFlags = 0;  // CPU 접근 불가
```

### 4. 상수 버퍼 효율적 관리
```cpp
// 변경 빈도별로 상수 버퍼 분리
struct PerFrameConstants {    // 매 프레임 변경
    DirectX::XMFLOAT4X4 viewMatrix;
    DirectX::XMFLOAT4X4 projectionMatrix;
    DirectX::XMFLOAT3 lightDirection;
};

struct PerObjectConstants {   // 객체별로 변경
    DirectX::XMFLOAT4X4 worldMatrix;
    DirectX::XMFLOAT4 materialColor;
};

struct PerSceneConstants {    // 거의 변경 안됨
    DirectX::XMFLOAT4 ambientColor;
    float time;
};
```

## 디버깅과 오류 처리

### 1. HRESULT 확인
```cpp
HRESULT hr = deviceContext->Map(buffer, 0, D3D11_MAP_WRITE_DISCARD, 0, &mapped);
if (FAILED(hr)) {
    switch (hr) {
    case DXGI_ERROR_DEVICE_REMOVED:
        // GPU 드라이버 크래시 등
        HandleDeviceLost();
        break;
    case DXGI_ERROR_INVALID_CALL:
        // 잘못된 파라미터
        OutputDebugStringA("Invalid Map parameters\n");
        break;
    case E_INVALIDARG:
        // 올바르지 않은 인수
        OutputDebugStringA("Invalid arguments\n");
        break;
    }
}
```

### 2. 디버그 레이어 활용
```cpp
// 디바이스 생성 시 디버그 플래그 설정
UINT createDeviceFlags = 0;
#ifdef _DEBUG
    createDeviceFlags |= D3D11_CREATE_DEVICE_DEBUG;
#endif

HRESULT hr = D3D11CreateDevice(
    nullptr,
    D3D_DRIVER_TYPE_HARDWARE,
    nullptr,
    createDeviceFlags,  // 디버그 레이어 활성화
    featureLevels,
    ARRAYSIZE(featureLevels),
    D3D11_SDK_VERSION,
    &device,
    &featureLevel,
    &deviceContext
);
```

### 3. 일반적인 실수들

#### 리소스 바인딩 순서 실수
```cpp
// 잘못된 순서
deviceContext->Draw(vertexCount, 0);  // 셰이더가 바인딩되지 않음!
deviceContext->VSSetShader(vertexShader, nullptr, 0);

// 올바른 순서
deviceContext->VSSetShader(vertexShader, nullptr, 0);
deviceContext->PSSetShader(pixelShader, nullptr, 0);
deviceContext->Draw(vertexCount, 0);
```

#### 뷰포트 설정 잊기
```cpp
// 뷰포트 설정 없이 그리면 아무것도 안 보임
deviceContext->Draw(vertexCount, 0);  // 화면에 안 보임

// 뷰포트 설정 후 그리기
D3D11_VIEWPORT viewport = {0, 0, 800, 600, 0.0f, 1.0f};
deviceContext->RSSetViewports(1, &viewport);
deviceContext->Draw(vertexCount, 0);  // 정상적으로 보임
```

#### 렌더 타겟 클리어 잊기
```cpp
// 이전 프레임 내용이 남아있음
deviceContext->Draw(vertexCount, 0);

// 매 프레임 클리어 필요
float clearColor[4] = {0.0f, 0.0f, 0.0f, 1.0f};
deviceContext->ClearRenderTargetView(renderTargetView, clearColor);
deviceContext->Draw(vertexCount, 0);
```

## 요약

Device Context는 DirectX 11에서 실제 렌더링을 담당하는 핵심 객체입니다:

### 주요 역할
1. **파이프라인 상태 관리**: 각 단계별 리소스 바인딩
2. **렌더링 명령 실행**: Draw, DrawIndexed 등
3. **리소스 업데이트**: Map/Unmap, UpdateSubresource
4. **GPU-CPU 동기화**: 비동기 작업 관리

### 핵심 개념
- **즉시 실행**: 대부분의 명령이 비동기로 GPU에 전달
- **상태 기반**: 파이프라인 상태를 설정한 후 그리기
- **효율성**: 드로우 콜과 상태 변경 최소화가 성능의 핵심

Device Context를 잘 이해하면 효율적이고 고성능인 DirectX 11 애플리케이션을 만들 수 있습니다!