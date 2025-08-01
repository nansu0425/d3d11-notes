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

**렌더 타겟 뷰 사용 과정:**

##### 1. 기본 렌더링 과정
```cpp
void RenderToBackBuffer() {
    // 1. 백 버퍼를 렌더 타겟으로 설정
    deviceContext->OMSetRenderTargets(1, &backBufferRTV, depthStencilView);
    
    // 2. 렌더 타겟 클리어 (캔버스 청소)
    float clearColor[4] = {0.0f, 0.2f, 0.4f, 1.0f};  // 진한 파란색
    deviceContext->ClearRenderTargetView(backBufferRTV, clearColor);
    
    // 3. 3D 객체들 그리기
    RenderObjects();
    
    // 4. 화면에 출력
    swapChain->Present(1, 0);
}
```

##### 2. 오프스크린 렌더링 과정 (예: 그림자 맵 생성)
```cpp
void RenderShadowMap() {
    // 1. 그림자 맵 텍스처를 렌더 타겟으로 설정
    deviceContext->OMSetRenderTargets(1, &shadowMapRTV, shadowMapDepthView);
    
    // 2. 그림자 맵 클리어
    float clearColor[4] = {1.0f, 1.0f, 1.0f, 1.0f};  // 흰색 (멀리 있음)
    deviceContext->ClearRenderTargetView(shadowMapRTV, clearColor);
    
    // 3. 조명 시점에서 깊이 정보만 렌더링
    RenderDepthOnly();
    
    // 4. 그림자 맵을 다른 렌더링에서 텍스처로 사용
    deviceContext->PSSetShaderResources(1, 1, &shadowMapSRV);
}
```

##### 3. 멀티 렌더 타겟 (MRT) 사용
```cpp
// 지연 렌더링(Deferred Rendering)에서 사용
ID3D11RenderTargetView* multipleRTVs[4] = {
    diffuseRTV,    // 확산 색상
    normalRTV,     // 법선 벡터
    specularRTV,   // 반사 색상
    positionRTV    // 월드 위치
};

// 4개의 렌더 타겟에 동시에 렌더링
deviceContext->OMSetRenderTargets(4, multipleRTVs, depthStencilView);

// 픽셀 셰이더에서 각 타겟에 다른 값 출력
```

**멀티 렌더 타겟용 픽셀 셰이더 예시:**
```hlsl
struct PSOutput {
    float4 diffuse  : SV_Target0;  // 첫 번째 렌더 타겟
    float4 normal   : SV_Target1;  // 두 번째 렌더 타겟
    float4 specular : SV_Target2;  // 세 번째 렌더 타겟
    float4 position : SV_Target3;  // 네 번째 렌더 타겟
};

PSOutput main(VertexOutput input) {
    PSOutput output;
    output.diffuse = CalculateDiffuse(input);
    output.normal = CalculateNormal(input);
    output.specular = CalculateSpecular(input);
    output.position = input.worldPos;
    return output;
}
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

## 렌더 타겟 전환 시 주의사항

### 상태 관리
```cpp
// 렌더 타겟 전환 시 항상 클리어
void SwitchRenderTarget(ID3D11RenderTargetView* newRTV, ID3D11DepthStencilView* newDSV) {
    // 1. 새 렌더 타겟 설정
    deviceContext->OMSetRenderTargets(1, &newRTV, newDSV);
    
    // 2. 반드시 클리어 (이전 내용 제거)
    float clearColor[4] = {0.0f, 0.0f, 0.0f, 0.0f};
    deviceContext->ClearRenderTargetView(newRTV, clearColor);
    if (newDSV) {
        deviceContext->ClearDepthStencilView(newDSV, 
            D3D11_CLEAR_DEPTH | D3D11_CLEAR_STENCIL, 1.0f, 0);
    }
    
    // 3. 뷰포트도 적절히 설정
    SetViewportForRenderTarget(newRTV);
}
```

### 성능 최적화
```cpp
// 렌더 타겟 전환 최소화
void OptimizedMultiPassRendering() {
    // 1. 모든 그림자 맵을 한 번에 생성
    RenderAllShadowMaps();
    
    // 2. 모든 후처리를 연속으로 처리
    RenderSceneToTexture();
    ApplyAllPostProcessEffects();
    
    // 3. 최종 조합을 백 버퍼에서 수행
    CombineToBackBuffer();
}
```

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

## Present 호출 후 렌더 타겟 뷰의 동작

많은 초보자들이 궁금해하는 중요한 질문: "SwapChain Present를 호출해서 백 버퍼가 교체된 후에 Device Context에서 렌더 타겟 뷰를 다시 설정해야 하나요?"

**답: 아니요! 렌더 타겟 뷰를 다시 설정할 필요가 없습니다.**

### 왜 그럴까요?

SwapChain의 Present 동작 과정을 이해하면 답을 알 수 있습니다:

```cpp
// 더블 버퍼링 시스템의 내부 동작
┌─────────────────────┐    ┌─────────────────────┐
│   프론트 버퍼       │    │    백 버퍼          │
│  (현재 화면에 표시) │    │ (다음 프레임 준비)  │
└─────────────────────┘    └─────────────────────┘
        ↑                          ↑
    사용자가 봄              Device Context가 렌더링하는 곳

// Present() 호출 시:
1. 포인터만 교체됨 (실제 메모리는 이동하지 않음)
┌─────────────────────┐    ┌─────────────────────┐
│    백 버퍼          │    │   프론트 버퍼       │
│ (다음 프레임 준비)  │    │ (현재 화면에 표시)  │
└─────────────────────┘    └─────────────────────┘

2. 렌더 타겟 뷰는 여전히 새로운 "백 버퍼"를 가리킴
```

### Device Context의 스마트한 렌더 타겟 관리

```cpp
// Device Context 렌더링 루프 예시
void RenderLoop() {
    while (running) {
        // 1. 렌더 타겟 설정 (한 번만 설정하면 됨)
        deviceContext->OMSetRenderTargets(1, &frameBufferRTV, depthStencilView);
        
        // 2. 백 버퍼 클리어
        float clearColor[4] = {0.0f, 0.2f, 0.4f, 1.0f};
        deviceContext->ClearRenderTargetView(frameBufferRTV, clearColor);
        deviceContext->ClearDepthStencilView(depthStencilView, 
            D3D11_CLEAR_DEPTH | D3D11_CLEAR_STENCIL, 1.0f, 0);
        
        // 3. 모든 렌더링 작업
        RenderAllObjects();
        
        // 4. Present 호출 (백 버퍼와 프론트 버퍼 교체)
        swapChain->Present(1, 0);
        
        // ★ Device Context에서 렌더 타겟 뷰 재설정 불필요!
        // frameBufferRTV는 자동으로 새로운 백 버퍼를 가리킴
    }
}
```

### Present 내부 동작 과정

```cpp
// Present() 내부 동작 (의사코드)
void IDXGISwapChain::Present(UINT syncInterval, UINT flags) {
    // 1. 백 버퍼와 프론트 버퍼의 역할 교체
    SwapBufferPointers();
    
    // 2. 모든 렌더 타겟 뷰가 자동으로 새로운 백 버퍼를 참조하도록 업데이트
    UpdateAllRenderTargetViews();
    
    // 3. 화면에 새로운 프론트 버퍼 내용 출력
    DisplayToScreen();
}
```

### 렌더 타겟 뷰가 변경되지 않는 이유

**1. DXGI의 스마트한 참조 관리**
```cpp
// 렌더 타겟 뷰는 "현재 백 버퍼"에 대한 논리적 참조를 유지
// 물리적 메모리 주소가 아닌, "백 버퍼 역할"을 하는 버퍼를 추적

ID3D11RenderTargetView* frameBufferRTV;  // 항상 "현재 백 버퍼"를 가리킴
device->CreateRenderTargetView(backBuffer, nullptr, &frameBufferRTV);

// Present 후에도 frameBufferRTV는 여전히 유효하고 올바른 백 버퍼를 가리킴
// Device Context는 동일한 RTV 포인터로 올바른 백 버퍼에 렌더링 가능
```

**2. SwapChain의 버퍼 관리**
```cpp
// SwapChain은 여러 버퍼를 순환하면서 관리
// 더블 버퍼링: 2개 버퍼
// 트리플 버퍼링: 3개 버퍼

// 각 Present 호출마다:
// 1. 현재 백 버퍼 → 프론트 버퍼로 승격
// 2. 이전 프론트 버퍼 → 새로운 백 버퍼로 강등
// 3. 모든 백 버퍼 참조자들이 자동으로 새로운 백 버퍼를 가리키도록 업데이트
```

### Device Context가 주의해야 할 상황들

**윈도우 크기 변경 시에만 재설정 필요:**
```cpp
// 이때만 Device Context에서 렌더 타겟 뷰를 다시 설정해야 함
void OnWindowResize(int newWidth, int newHeight) {
    // 1. Device Context에서 렌더 타겟 해제 (필수!)
    deviceContext->OMSetRenderTargets(0, nullptr, nullptr);
    
    // 2. 기존 뷰 해제
    frameBufferRTV->Release();
    depthStencilView->Release();
    
    // 3. SwapChain 버퍼 크기 조정
    swapChain->ResizeBuffers(0, newWidth, newHeight, DXGI_FORMAT_UNKNOWN, 0);
    
    // 4. 새로운 크기로 뷰 재생성
    CreateFrameBufferViews();
    
    // 5. Device Context에 새 렌더 타겟 설정
    deviceContext->OMSetRenderTargets(1, &frameBufferRTV, depthStencilView);
}
```

**풀스크린 모드 전환 시:**
```cpp
// 풀스크린 ↔ 윈도우 모드 전환 시에도 재설정 필요
void ToggleFullscreen() {
    // 1. Device Context에서 렌더 타겟 해제
    deviceContext->OMSetRenderTargets(0, nullptr, nullptr);
    
    // 2. 뷰 해제
    ReleaseFrameBufferViews();
    
    // 3. 풀스크린 전환
    swapChain->SetFullscreenState(!isFullscreen, nullptr);
    
    // 4. 뷰 재생성 및 Device Context에 재설정
    CreateFrameBufferViews();
    deviceContext->OMSetRenderTargets(1, &frameBufferRTV, depthStencilView);
}
```

### Device Context 성능상의 이점

```cpp
// 매 프레임마다 렌더 타겟 뷰를 재설정하지 않아도 되므로:
// - Device Context 상태 변경 최소화
// - OMSetRenderTargets 호출 횟수 감소
// - CPU 오버헤드 감소
// - 더 안정적인 렌더링 루프

// 잘못된 방법 (불필요한 Device Context 오버헤드)
void BadRenderLoop() {
    while (running) {
        swapChain->GetBuffer(0, IID_PPV_ARGS(&backBuffer));          // 매 프레임 호출 ❌
        device->CreateRenderTargetView(backBuffer, nullptr, &rtv);   // 매 프레임 생성 ❌
        deviceContext->OMSetRenderTargets(1, &rtv, nullptr);        // 매 프레임 설정 ❌
        
        RenderFrame();
        
        rtv->Release();                                              // 매 프레임 해제 ❌
        backBuffer->Release();
        swapChain->Present(1, 0);
    }
}

// 올바른 방법 (효율적인 Device Context 사용)
void GoodRenderLoop() {
    // 초기화 시 한 번만 생성
    CreateFrameBufferViews();                                        // 한 번만 생성 ✅
    deviceContext->OMSetRenderTargets(1, &frameBufferRTV, dsv);     // 한 번만 설정 ✅
    
    while (running) {
        // Device Context는 동일한 렌더 타겟으로 계속 렌더링
        deviceContext->ClearRenderTargetView(frameBufferRTV, clearColor);
        RenderFrame();
        swapChain->Present(1, 0);                                    // Present만 호출 ✅
    }
    
    // 종료 시 한 번만 해제
    ReleaseFrameBufferViews();                                       // 한 번만 해제 ✅
}
```

**핵심 요약:**
- **Present 후 Device Context에서 렌더 타겟 뷰 재설정 불필요** ✅
- **SwapChain이 자동으로 백 버퍼 참조를 관리** ✅
- **윈도우 크기 변경이나 풀스크린 전환 시에만 재설정** ✅
- **매 프레임 재설정은 Device Context 성능 낭비** ❌

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