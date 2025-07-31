# DirectX 11 Shader (셰이더)

## 셰이더란 무엇인가?

**Shader(셰이더)** 는 GPU에서 실행되는 작은 프로그램입니다. 쉽게 말해, 그래픽 카드가 이해할 수 있는 "언어"로 작성된 지시서라고 생각하면 됩니다.

게임이나 3D 애플리케이션에서 화면에 보이는 모든 것들 - 캐릭터의 모습, 물의 반짝임, 빛과 그림자 - 이 모든 것이 셰이더의 계산을 통해 만들어집니다. GPU의 수천 개 코어가 동시에 셰이더를 실행하여 실시간으로 아름다운 그래픽을 만들어냅니다.

## 왜 셰이더가 필요할까?

### 문제상황: CPU로만 그래픽 처리
만약 CPU가 혼자서 모든 그래픽 계산을 한다면 어떻게 될까요?

```
1920x1080 화면 = 2,073,600 픽셀
각 픽셀마다 복잡한 조명 계산 필요
CPU 코어 수: 4-16개 (순차 처리)
→ 초당 1-2 프레임도 어려움
```

### 해결책: GPU의 병렬 처리
GPU는 수천 개의 작은 코어를 가지고 있어서 동시에 많은 계산을 할 수 있습니다:

```
GPU 코어 수: 1,000-10,000개 (병렬 처리)
각 코어가 하나의 픽셀 처리
동시에 수천 개 픽셀 계산 가능
→ 초당 60-120 프레임 가능
```

하지만 GPU는 CPU와 다른 구조를 가지고 있어서, 특별한 프로그램 언어가 필요합니다. 이것이 바로 **셰이더**입니다.

## DirectX 11 셰이더 파이프라인

DirectX 11에서 그래픽이 그려지는 과정은 여러 단계로 나뉘어져 있고, 각 단계마다 다른 종류의 셰이더가 실행됩니다:

```
3D 모델 데이터 입력
       ↓
┌──────────────────┐
│  Vertex Shader   │ ← 정점 위치 변환
│   (정점 셰이더)   │
└──────────────────┘
       ↓
┌──────────────────┐
│  Hull Shader     │ ← 테셀레이션 제어 (선택적)
│   (헐 셰이더)     │
└──────────────────┘
       ↓
┌──────────────────┐
│ Domain Shader    │ ← 테셀레이션 평가 (선택적)
│  (도메인 셰이더)  │
└──────────────────┘
       ↓
┌──────────────────┐
│ Geometry Shader  │ ← 기하 처리 (선택적)
│  (지오메트리 셰이더)│
└──────────────────┘
       ↓
┌──────────────────┐
│   Rasterizer     │ ← 삼각형을 픽셀로 변환 (고정 기능)
│   (래스터라이저)  │
└──────────────────┘
       ↓
┌──────────────────┐
│  Pixel Shader    │ ← 각 픽셀의 색상 계산
│   (픽셀 셰이더)   │
└──────────────────┘
       ↓
    최종 화면 출력
```

### 필수 셰이더 vs 선택적 셰이더

#### 필수 셰이더 (반드시 있어야 함)
- **Vertex Shader**: 모든 3D 렌더링에 필수
- **Pixel Shader**: 색상 계산에 필수

#### 선택적 셰이더 (필요시에만 사용)
- **Hull Shader + Domain Shader**: 테셀레이션 (표면 세분화)
- **Geometry Shader**: 기하학적 변형, 파티클 생성 등

## 주요 셰이더 타입 상세 설명

### 1. Vertex Shader (정점 셰이더)

#### 역할과 목적
정점 셰이더는 3D 공간의 각 정점(꼭짓점)에 대해 실행됩니다. 주요 역할은 **좌표 변환**입니다.

```
월드 공간 좌표 → 화면 좌표 변환
```

#### 좌표 변환 과정
```cpp
// 입력: 3D 모델의 정점 (Local Space)
float3 localPosition = input.position;

// 1. 월드 변환 (World Transform)
float3 worldPosition = mul(localPosition, worldMatrix);

// 2. 뷰 변환 (View Transform) 
float3 viewPosition = mul(worldPosition, viewMatrix);

// 3. 투영 변환 (Projection Transform)
float4 clipPosition = mul(float4(viewPosition, 1.0), projectionMatrix);

// 출력: 화면 좌표계 정점
output.position = clipPosition;
```

#### 실제 HLSL 코드 예시
```hlsl
// 상수 버퍼 (CPU에서 전달받는 데이터)
cbuffer ConstantBuffer : register(b0)
{
    matrix World;      // 월드 변환 행렬
    matrix View;       // 뷰 변환 행렬  
    matrix Projection; // 투영 변환 행렬
};

// 입력 구조체
struct VS_INPUT
{
    float3 Position : POSITION;  // 정점 위치
    float3 Normal   : NORMAL;    // 법선 벡터
    float2 TexCoord : TEXCOORD0; // 텍스처 좌표
};

// 출력 구조체
struct VS_OUTPUT
{
    float4 Position : SV_POSITION; // 화면 좌표
    float3 WorldPos : TEXCOORD1;    // 월드 좌표 (조명 계산용)
    float3 Normal   : NORMAL;       // 법선 벡터
    float2 TexCoord : TEXCOORD0;    // 텍스처 좌표
};

// 정점 셰이더 메인 함수
VS_OUTPUT main(VS_INPUT input)
{
    VS_OUTPUT output;
    
    // 1. 월드 좌표 계산
    float4 worldPos = mul(float4(input.Position, 1.0), World);
    output.WorldPos = worldPos.xyz;
    
    // 2. 뷰 좌표 계산
    float4 viewPos = mul(worldPos, View);
    
    // 3. 투영 좌표 계산 (최종 화면 좌표)
    output.Position = mul(viewPos, Projection);
    
    // 4. 법선 벡터 변환 (조명 계산용)
    output.Normal = mul(input.Normal, (float3x3)World);
    
    // 5. 텍스처 좌표 전달
    output.TexCoord = input.TexCoord;
    
    return output;
}
```

#### 정점 셰이더의 실행 방식
```
삼각형 1개 = 정점 3개
정점 셰이더가 3번 실행됨

1000개 삼각형 = 3000개 정점
→ 정점 셰이더가 3000번 실행됨
→ GPU의 여러 코어에서 동시에 병렬 실행
```

### 2. Pixel Shader (픽셀 셰이더)

#### 역할과 목적
픽셀 셰이더는 화면의 각 픽셀에 대해 실행되어 **최종 색상**을 결정합니다.

#### 주요 계산
1. **텍스처 샘플링**: 이미지에서 색상 가져오기
2. **조명 계산**: 빛과 표면의 상호작용
3. **재질 표현**: 금속, 플라스틱, 천 등의 질감
4. **특수 효과**: 투명도, 발광, 반사 등

#### 실제 HLSL 코드 예시
```hlsl
// 텍스처와 샘플러
Texture2D DiffuseTexture : register(t0);  // 기본 텍스처
SamplerState LinearSampler : register(s0); // 텍스처 샘플링 방법

// 상수 버퍼
cbuffer LightBuffer : register(b0)
{
    float3 LightDirection;   // 조명 방향
    float3 LightColor;       // 조명 색상
    float3 AmbientColor;     // 환경광 색상
};

// 입력 구조체 (정점 셰이더에서 전달받음)
struct PS_INPUT
{
    float4 Position : SV_POSITION; // 화면 좌표 (사용 안함)
    float3 WorldPos : TEXCOORD1;    // 월드 좌표
    float3 Normal   : NORMAL;       // 법선 벡터
    float2 TexCoord : TEXCOORD0;    // 텍스처 좌표
};

// 출력 구조체
struct PS_OUTPUT
{
    float4 Color : SV_TARGET; // 최종 픽셀 색상
};

// 픽셀 셰이더 메인 함수
PS_OUTPUT main(PS_INPUT input)
{
    PS_OUTPUT output;
    
    // 1. 텍스처에서 기본 색상 샘플링
    float4 diffuseColor = DiffuseTexture.Sample(LinearSampler, input.TexCoord);
    
    // 2. 법선 벡터 정규화
    float3 normal = normalize(input.Normal);
    
    // 3. 조명 방향 벡터 계산
    float3 lightDir = normalize(-LightDirection);
    
    // 4. 디퓨즈 조명 계산 (Lambert 공식)
    float dotProduct = dot(normal, lightDir);
    float diffuseIntensity = max(dotProduct, 0.0);
    
    // 5. 최종 색상 계산
    float3 diffuse = diffuseColor.rgb * LightColor * diffuseIntensity;
    float3 ambient = diffuseColor.rgb * AmbientColor;
    float3 finalColor = diffuse + ambient;
    
    output.Color = float4(finalColor, diffuseColor.a);
    return output;
}
```

#### 픽셀 셰이더의 실행 방식
```
1920x1080 화면 = 2,073,600 픽셀
각 픽셀마다 픽셀 셰이더 1번 실행
→ 200만 번 이상 실행됨!
→ GPU의 수천 개 코어가 동시에 처리
```

### 3. Compute Shader (컴퓨트 셰이더)

#### 역할과 목적
그래픽 렌더링과 무관한 **범용 병렬 계산**을 위한 셰이더입니다.

#### 사용 예시
- **물리 시뮬레이션**: 파티클, 유체, 천 시뮬레이션
- **AI 계산**: 신경망, 머신러닝
- **이미지 처리**: 블러, 샤프닝, 필터 효과
- **과학 계산**: 수치 해석, 시뮬레이션

#### HLSL 코드 예시 (파티클 시뮬레이션)
```hlsl
// 파티클 데이터 구조체
struct Particle
{
    float3 Position;    // 위치
    float3 Velocity;    // 속도
    float Life;         // 생명력
    float Size;         // 크기
};

// 입출력 버퍼
RWStructuredBuffer<Particle> ParticleBuffer : register(u0);

// 상수 버퍼
cbuffer SimulationData : register(b0)
{
    float DeltaTime;    // 프레임 시간
    float3 Gravity;     // 중력
    int ParticleCount;  // 파티클 개수
};

// 컴퓨트 셰이더 (32개 스레드가 하나의 그룹)
[numthreads(32, 1, 1)]
void main(uint3 id : SV_DispatchThreadID)
{
    // 현재 스레드가 담당할 파티클 인덱스
    uint index = id.x;
    
    // 범위 체크
    if (index >= ParticleCount)
        return;
        
    // 현재 파티클 가져오기
    Particle particle = ParticleBuffer[index];
    
    // 물리 계산
    particle.Velocity += Gravity * DeltaTime;      // 중력 적용
    particle.Position += particle.Velocity * DeltaTime; // 위치 업데이트
    particle.Life -= DeltaTime;                    // 생명력 감소
    
    // 바닥 충돌 처리
    if (particle.Position.y < 0.0)
    {
        particle.Position.y = 0.0;
        particle.Velocity.y = -particle.Velocity.y * 0.8; // 반발
    }
    
    // 결과 저장
    ParticleBuffer[index] = particle;
}
```

#### 컴퓨트 셰이더 실행 방식
```
Dispatch(64, 1, 1) 호출 시:
64개 그룹 × 32개 스레드 = 2048개 파티클 동시 처리
→ 수천 개 파티클도 한 번에 시뮬레이션 가능
```

## HLSL 기초 문법

### 1. 데이터 타입

#### 스칼라 타입
```hlsl
bool    isVisible = true;
int     count = 10;
uint    index = 0;
float   temperature = 25.5f;
double  precision = 3.141592653589793;
```

#### 벡터 타입
```hlsl
float2 texCoord = float2(0.5, 0.5);    // UV 좌표
float3 position = float3(1, 2, 3);     // 3D 위치
float4 color = float4(1, 0, 0, 1);     // RGBA 색상

// 벡터 성분 접근 방법
float x = position.x;     // x 성분
float y = position.y;     // y 성분
float z = position.z;     // z 성분

// 스위즐링 (성분 재배열)
float2 xy = position.xy;     // x,y만 추출
float3 zyx = position.zyx;   // z,y,x 순서로 재배열
float4 xxxx = position.xxxx; // x값을 4번 반복
```

#### 행렬 타입
```hlsl
float4x4 transformMatrix;    // 4x4 변환 행렬
float3x3 rotationMatrix;     // 3x3 회전 행렬

// 행렬 연산
float4 transformedPos = mul(originalPos, transformMatrix);
```

#### 구조체
```hlsl
struct Material
{
    float3 Diffuse;      // 확산 색상
    float3 Specular;     // 반사 색상
    float Shininess;     // 광택도
    float Transparency;  // 투명도
};

Material material;
material.Diffuse = float3(1, 0, 0);    // 빨간색
material.Shininess = 32.0;
```

### 2. 함수와 제어문

#### 함수 정의
```hlsl
// 조명 계산 함수
float3 CalculateDiffuse(float3 normal, float3 lightDir, float3 lightColor)
{
    float intensity = max(dot(normal, lightDir), 0.0);
    return lightColor * intensity;
}

// 거리 계산 함수
float Distance(float3 a, float3 b)
{
    float3 diff = a - b;
    return sqrt(dot(diff, diff));
}

// 정규화 함수 (내장 함수 normalize도 있음)
float3 Normalize(float3 vector)
{
    float length = sqrt(dot(vector, vector));
    return vector / length;
}
```

#### 제어문
```hlsl
// if-else문
if (intensity > 0.5)
{
    color = float3(1, 1, 1);    // 밝은 색
}
else
{
    color = float3(0.2, 0.2, 0.2); // 어두운 색
}

// for 루프
float3 averageColor = float3(0, 0, 0);
for (int i = 0; i < 4; i++)
{
    averageColor += textureColors[i];
}
averageColor /= 4.0;

// while 루프 (사용 주의 - GPU에서 비효율적일 수 있음)
int steps = 0;
while (steps < maxSteps && rayDistance < maxDistance)
{
    // 레이 마칭 등의 반복 계산
    steps++;
}
```

### 3. 내장 함수들

#### 수학 함수
```hlsl
// 기본 수학
float result = abs(-5.0);           // 절댓값: 5.0
float result = sqrt(16.0);          // 제곱근: 4.0
float result = pow(2.0, 3.0);       // 거듭제곱: 8.0
float result = sin(3.14159);        // 사인값
float result = cos(0.0);            // 코사인값: 1.0

// 최소/최대값
float result = min(3.0, 7.0);       // 최솟값: 3.0
float result = max(3.0, 7.0);       // 최댓값: 7.0
float result = clamp(5.0, 0.0, 3.0); // 범위 제한: 3.0

// 선형 보간
float result = lerp(0.0, 10.0, 0.5); // 0과 10 사이의 50% 지점: 5.0
```

#### 벡터 함수
```hlsl
float3 a = float3(1, 0, 0);
float3 b = float3(0, 1, 0);

float dotResult = dot(a, b);        // 내적: 0.0
float3 crossResult = cross(a, b);   // 외적: (0, 0, 1)
float lengthResult = length(a);     // 길이: 1.0
float3 normalResult = normalize(a); // 정규화: (1, 0, 0)
float distResult = distance(a, b);  // 거리: 1.414...
```

#### 텍스처 샘플링 함수
```hlsl
Texture2D myTexture : register(t0);
SamplerState mySampler : register(s0);

// 기본 샘플링
float4 color = myTexture.Sample(mySampler, float2(0.5, 0.5));

// LOD 레벨 지정 샘플링
float4 color = myTexture.SampleLevel(mySampler, float2(0.5, 0.5), 2);

// 그래디언트 샘플링
float4 color = myTexture.SampleGrad(mySampler, uv, ddx_uv, ddy_uv);
```

### 4. 시맨틱(Semantics)

시맨틱은 셰이더 입출력 데이터의 의미를 DirectX에게 알려주는 키워드입니다.

#### 정점 셰이더 입력 시맨틱
```hlsl
struct VS_INPUT
{
    float3 Position : POSITION;     // 정점 위치
    float3 Normal   : NORMAL;       // 법선 벡터
    float4 Color    : COLOR;        // 정점 색상
    float2 TexCoord : TEXCOORD0;    // 텍스처 좌표 0
    float2 TexCoord2: TEXCOORD1;    // 텍스처 좌표 1
    float3 Tangent  : TANGENT;      // 접선 벡터
    float3 Binormal : BINORMAL;     // 종법선 벡터
};
```

#### 정점 셰이더 출력 / 픽셀 셰이더 입력 시맨틱
```hlsl
struct VS_OUTPUT / PS_INPUT
{
    float4 Position : SV_POSITION;  // 시스템 값: 화면 좌표
    float4 Color    : COLOR;        // 색상
    float2 TexCoord : TEXCOORD0;    // 텍스처 좌표
    float3 WorldPos : TEXCOORD1;    // 월드 좌표 (조명 계산용)
    float3 Normal   : NORMAL;       // 법선 벡터
};
```

#### 픽셀 셰이더 출력 시맨틱
```hlsl
struct PS_OUTPUT
{
    float4 Color : SV_TARGET;       // 렌더 타겟 0
    float4 Color1 : SV_TARGET1;     // 렌더 타겟 1 (MRT)
    float Depth : SV_DEPTH;         // 깊이 값 (선택적)
};
```

## 셰이더 컴파일과 생성

### 1. HLSL 파일 작성

#### VertexShader.hlsl
```hlsl
cbuffer ConstantBuffer : register(b0)
{
    matrix World;
    matrix View;
    matrix Projection;
};

struct VS_INPUT
{
    float3 Position : POSITION;
    float2 TexCoord : TEXCOORD0;
};

struct VS_OUTPUT
{
    float4 Position : SV_POSITION;
    float2 TexCoord : TEXCOORD0;
};

VS_OUTPUT main(VS_INPUT input)
{
    VS_OUTPUT output;
    
    float4 worldPos = mul(float4(input.Position, 1.0), World);
    float4 viewPos = mul(worldPos, View);
    output.Position = mul(viewPos, Projection);
    output.TexCoord = input.TexCoord;
    
    return output;
}
```

#### PixelShader.hlsl
```hlsl
Texture2D DiffuseTexture : register(t0);
SamplerState LinearSampler : register(s0);

struct PS_INPUT
{
    float4 Position : SV_POSITION;
    float2 TexCoord : TEXCOORD0;
};

float4 main(PS_INPUT input) : SV_TARGET
{
    return DiffuseTexture.Sample(LinearSampler, input.TexCoord);
}
```

### 2. 런타임 컴파일 방법

```cpp
#include <d3dcompiler.h>

// 셰이더 파일을 읽어서 컴파일하는 함수
HRESULT CompileShaderFromFile(const wchar_t* filename, const char* entryPoint, 
                             const char* shaderModel, ID3DBlob** blobOut)
{
    HRESULT hr = S_OK;
    
    // 1. 파일 읽기
    std::ifstream file(filename, std::ios::binary);
    if (!file.is_open()) {
        return E_FAIL;
    }
    
    // 파일 내용을 문자열로 읽기
    std::string shaderCode((std::istreambuf_iterator<char>(file)),
                           std::istreambuf_iterator<char>());
    
    // 2. 컴파일 플래그 설정
    DWORD shaderFlags = D3DCOMPILE_ENABLE_STRICTNESS;
#ifdef _DEBUG
    // 디버그 모드에서는 최적화 끄고 디버그 정보 포함
    shaderFlags |= D3DCOMPILE_DEBUG | D3DCOMPILE_SKIP_OPTIMIZATION;
#endif
    
    // 3. 셰이더 컴파일
    ID3DBlob* errorBlob = nullptr;
    hr = D3DCompile(
        shaderCode.c_str(),     // 셰이더 소스 코드
        shaderCode.length(),    // 소스 코드 길이
        filename,               // 파일명 (에러 메시지용)
        nullptr,                // 매크로 정의
        nullptr,                // 인클루드 핸들러
        entryPoint,             // 진입점 함수명 ("main")
        shaderModel,            // 셰이더 모델 ("vs_5_0", "ps_5_0" 등)
        shaderFlags,            // 컴파일 플래그
        0,                      // 이펙트 플래그 (사용 안함)
        blobOut,                // 컴파일된 바이트코드
        &errorBlob              // 에러 메시지
    );
    
    // 4. 컴파일 에러 처리
    if (FAILED(hr))
    {
        if (errorBlob != nullptr)
        {
            // 에러 메시지 출력
            const char* errorMsg = (char*)errorBlob->GetBufferPointer();
            MessageBoxA(nullptr, errorMsg, "Shader Compile Error", MB_OK);
            errorBlob->Release();
        }
        return hr;
    }
    
    if (errorBlob) errorBlob->Release();
    return S_OK;
}

// 정점 셰이더 생성
ID3DBlob* vertexShaderBlob = nullptr;
HRESULT hr = CompileShaderFromFile(L"VertexShader.hlsl", "main", "vs_5_0", &vertexShaderBlob);
if (SUCCEEDED(hr))
{
    ID3D11VertexShader* vertexShader = nullptr;
    hr = device->CreateVertexShader(
        vertexShaderBlob->GetBufferPointer(),
        vertexShaderBlob->GetBufferSize(),
        nullptr,
        &vertexShader
    );
}

// 픽셀 셰이더 생성
ID3DBlob* pixelShaderBlob = nullptr;
hr = CompileShaderFromFile(L"PixelShader.hlsl", "main", "ps_5_0", &pixelShaderBlob);
if (SUCCEEDED(hr))
{
    ID3D11PixelShader* pixelShader = nullptr;
    hr = device->CreatePixelShader(
        pixelShaderBlob->GetBufferPointer(),
        pixelShaderBlob->GetBufferSize(),
        nullptr,
        &pixelShader
    );
}
```

### 3. 사전 컴파일 방법 (권장)

실제 게임 개발에서는 성능을 위해 셰이더를 미리 컴파일해둡니다:

#### 명령줄 컴파일
```batch
:: fxc.exe는 DirectX SDK에 포함된 셰이더 컴파일러
fxc.exe /T vs_5_0 /E main /Fo VertexShader.cso VertexShader.hlsl
fxc.exe /T ps_5_0 /E main /Fo PixelShader.cso PixelShader.hlsl

:: 옵션 설명:
:: /T - 타겟 셰이더 모델
:: /E - 진입점 함수명
:: /Fo - 출력 파일명
:: /Od - 최적화 끄기 (디버그용)
:: /Zi - 디버그 정보 포함
```

#### 컴파일된 셰이더 로드
```cpp
// 컴파일된 바이트코드 파일 로드
std::vector<BYTE> LoadCompiledShader(const wchar_t* filename)
{
    std::ifstream file(filename, std::ios::binary);
    if (!file.is_open()) {
        throw std::runtime_error("Failed to open shader file");
    }
    
    file.seekg(0, std::ios::end);
    size_t size = file.tellg();
    file.seekg(0, std::ios::beg);
    
    std::vector<BYTE> data(size);
    file.read((char*)data.data(), size);
    
    return data;
}

// 사용 예시
std::vector<BYTE> vsData = LoadCompiledShader(L"VertexShader.cso");
std::vector<BYTE> psData = LoadCompiledShader(L"PixelShader.cso");

ID3D11VertexShader* vertexShader = nullptr;
device->CreateVertexShader(vsData.data(), vsData.size(), nullptr, &vertexShader);

ID3D11PixelShader* pixelShader = nullptr;
device->CreatePixelShader(psData.data(), psData.size(), nullptr, &pixelShader);
```

## Input Layout (입력 레이아웃)

셰이더가 정점 데이터를 올바르게 해석할 수 있도록 데이터 구조를 설명하는 객체입니다.

### Input Layout이 필요한 이유

```cpp
// C++ 정점 구조체
struct Vertex {
    DirectX::XMFLOAT3 position;    // 12바이트
    DirectX::XMFLOAT3 normal;      // 12바이트  
    DirectX::XMFLOAT2 texCoord;    // 8바이트
    // 총 32바이트
};

// HLSL 정점 셰이더 입력
struct VS_INPUT {
    float3 Position : POSITION;
    float3 Normal   : NORMAL;
    float2 TexCoord : TEXCOORD0;
};
```

GPU는 메모리의 바이트 덩어리만 받기 때문에, 어떤 바이트가 위치이고 어떤 바이트가 법선인지 알 수 없습니다. Input Layout이 이 정보를 제공합니다.

### Input Layout 생성

```cpp
// 입력 요소 배열 정의
D3D11_INPUT_ELEMENT_DESC inputElements[] = {
    // 위치 데이터
    {
        "POSITION",                          // 시맨틱 이름
        0,                                   // 시맨틱 인덱스
        DXGI_FORMAT_R32G32B32_FLOAT,        // 데이터 포맷 (float3)
        0,                                   // 입력 슬롯
        0,                                   // 바이트 오프셋
        D3D11_INPUT_PER_VERTEX_DATA,        // 입력 분류
        0                                    // 인스턴스 데이터 스텝율
    },
    
    // 법선 데이터
    {
        "NORMAL",                            // 시맨틱 이름
        0,                                   // 시맨틱 인덱스
        DXGI_FORMAT_R32G32B32_FLOAT,        // 데이터 포맷 (float3)
        0,                                   // 입력 슬롯
        12,                                  // 바이트 오프셋 (position 다음)
        D3D11_INPUT_PER_VERTEX_DATA,        // 입력 분류
        0                                    // 인스턴스 데이터 스텝율
    },
    
    // 텍스처 좌표 데이터
    {
        "TEXCOORD",                          // 시맨틱 이름
        0,                                   // 시맨틱 인덱스
        DXGI_FORMAT_R32G32_FLOAT,           // 데이터 포맷 (float2)
        0,                                   // 입력 슬롯
        24,                                  // 바이트 오프셋 (position + normal 다음)
        D3D11_INPUT_PER_VERTEX_DATA,        // 입력 분류
        0                                    // 인스턴스 데이터 스텝율
    }
};

// Input Layout 생성
ID3D11InputLayout* inputLayout = nullptr;
HRESULT hr = device->CreateInputLayout(
    inputElements,                           // 입력 요소 배열
    ARRAYSIZE(inputElements),               // 배열 크기
    vertexShaderBlob->GetBufferPointer(),   // 정점 셰이더 바이트코드
    vertexShaderBlob->GetBufferSize(),      // 바이트코드 크기
    &inputLayout                            // 생성된 객체
);
```

### 데이터 포맷 종류

| DXGI_FORMAT | 설명 | C++ 타입 | HLSL 타입 |
|-------------|------|----------|-----------|
| `R32G32B32_FLOAT` | 32비트 float 3개 | `XMFLOAT3` | `float3` |
| `R32G32_FLOAT` | 32비트 float 2개 | `XMFLOAT2` | `float2` |
| `R32_FLOAT` | 32비트 float 1개 | `float` | `float` |
| `R8G8B8A8_UNORM` | 8비트 unsigned normalized 4개 | `BYTE[4]` | `float4` |
| `R16G16_SINT` | 16비트 signed int 2개 | `short[2]` | `int2` |

## 상수 버퍼(Constant Buffer)

CPU에서 GPU로 데이터를 전달하는 주요 방법입니다.

### 상수 버퍼가 필요한 이유

```cpp
// 매 프레임 변하는 데이터들
DirectX::XMMATRIX worldMatrix;      // 오브젝트 위치/회전/크기
DirectX::XMMATRIX viewMatrix;       // 카메라 위치/방향
DirectX::XMMATRIX projectionMatrix; // 투영 설정
DirectX::XMFLOAT3 lightDirection;   // 조명 방향
DirectX::XMFLOAT4 lightColor;       // 조명 색상
```

이런 데이터들을 어떻게 셰이더에 전달할까요? 상수 버퍼가 그 답입니다.

### 상수 버퍼 생성과 사용

#### 1. C++ 데이터 구조체 정의
```cpp
// 16바이트 정렬 규칙 준수 필요
struct ConstantBufferData
{
    DirectX::XMMATRIX world;        // 64바이트 (16바이트 정렬됨)
    DirectX::XMMATRIX view;         // 64바이트 (16바이트 정렬됨)
    DirectX::XMMATRIX projection;   // 64바이트 (16바이트 정렬됨)
    DirectX::XMFLOAT3 lightDir;     // 12바이트
    float padding1;                 // 4바이트 패딩 (16바이트 맞춤)
    DirectX::XMFLOAT4 lightColor;   // 16바이트 (이미 정렬됨)
    // 총 크기: 224바이트 (16의 배수)
};
```

#### 2. 상수 버퍼 생성
```cpp
// 상수 버퍼 생성
D3D11_BUFFER_DESC cbDesc = {};
cbDesc.ByteWidth = sizeof(ConstantBufferData);
cbDesc.Usage = D3D11_USAGE_DYNAMIC;          // CPU에서 자주 업데이트
cbDesc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
cbDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;

ID3D11Buffer* constantBuffer = nullptr;
HRESULT hr = device->CreateBuffer(&cbDesc, nullptr, &constantBuffer);
```

#### 3. 데이터 업데이트
```cpp
void UpdateConstantBuffer()
{
    // 데이터 준비
    ConstantBufferData cbData = {};
    cbData.world = XMMatrixTranspose(worldMatrix);           // 행렬 전치 필요!
    cbData.view = XMMatrixTranspose(viewMatrix);
    cbData.projection = XMMatrixTranspose(projectionMatrix);
    cbData.lightDir = lightDirection;
    cbData.lightColor = lightColor;
    
    // GPU 메모리에 데이터 쓰기
    D3D11_MAPPED_SUBRESOURCE mappedResource;
    HRESULT hr = deviceContext->Map(
        constantBuffer,                     // 대상 버퍼
        0,                                  // 서브리소스 인덱스
        D3D11_MAP_WRITE_DISCARD,           // 맵 타입
        0,                                  // 맵 플래그
        &mappedResource                     // 맵된 리소스 정보
    );
    
    if (SUCCEEDED(hr))
    {
        // 데이터 복사
        memcpy(mappedResource.pData, &cbData, sizeof(cbData));
        
        // 맵 해제
        deviceContext->Unmap(constantBuffer, 0);
    }
}
```

#### 4. 셰이더에 바인딩
```cpp
// 정점 셰이더의 레지스터 b0에 바인딩
deviceContext->VSSetConstantBuffers(0, 1, &constantBuffer);

// 픽셀 셰이더의 레지스터 b0에 바인딩
deviceContext->PSSetConstantBuffers(0, 1, &constantBuffer);
```

#### 5. HLSL에서 사용
```hlsl
// 상수 버퍼 정의 (C++ 구조체와 동일해야 함)
cbuffer ConstantBuffer : register(b0)
{
    matrix World;           // 64바이트
    matrix View;            // 64바이트
    matrix Projection;      // 64바이트
    float3 LightDirection;  // 12바이트 + 4바이트 패딩
    float4 LightColor;      // 16바이트
};

// 정점 셰이더에서 사용
VS_OUTPUT main(VS_INPUT input)
{
    VS_OUTPUT output;
    
    // 상수 버퍼의 행렬들 사용
    float4 worldPos = mul(float4(input.Position, 1.0), World);
    float4 viewPos = mul(worldPos, View);
    output.Position = mul(viewPos, Projection);
    
    return output;
}
```

### 행렬 전치(Transpose)가 필요한 이유

```cpp
// C++에서는 행렬이 행 우선(Row-major)으로 저장됨
float matrix[4][4] = {
    {m11, m12, m13, m14},  // 첫 번째 행
    {m21, m22, m23, m24},  // 두 번째 행
    {m31, m32, m33, m34},  // 세 번째 행
    {m41, m42, m43, m44}   // 네 번째 행
};

// HLSL에서는 열 우선(Column-major)으로 해석됨
// 따라서 CPU에서 XMMatrixTranspose()로 전치해서 전달해야 함
```

## 텍스처와 샘플러

### 텍스처 생성과 셰이더 바인딩

#### 1. 텍스처 생성
```cpp
// 텍스처 로드 (예: WIC 사용)
ID3D11Texture2D* texture = nullptr;
// ... 텍스처 로드 코드 ...

// 셰이더 리소스 뷰 생성
ID3D11ShaderResourceView* textureView = nullptr;
HRESULT hr = device->CreateShaderResourceView(texture, nullptr, &textureView);
```

#### 2. 샘플러 상태 생성
```cpp
// 샘플러 상태 설정
D3D11_SAMPLER_DESC samplerDesc = {};
samplerDesc.Filter = D3D11_FILTER_MIN_MAG_MIP_LINEAR;    // 선형 필터링
samplerDesc.AddressU = D3D11_TEXTURE_ADDRESS_WRAP;       // U 방향 랩핑
samplerDesc.AddressV = D3D11_TEXTURE_ADDRESS_WRAP;       // V 방향 랩핑
samplerDesc.AddressW = D3D11_TEXTURE_ADDRESS_WRAP;       // W 방향 랩핑
samplerDesc.MipLODBias = 0.0f;
samplerDesc.MaxAnisotropy = 1;
samplerDesc.ComparisonFunc = D3D11_COMPARISON_ALWAYS;
samplerDesc.MinLOD = 0;
samplerDesc.MaxLOD = D3D11_FLOAT32_MAX;

ID3D11SamplerState* samplerState = nullptr;
hr = device->CreateSamplerState(&samplerDesc, &samplerState);
```

#### 3. 셰이더에 바인딩
```cpp
// 픽셀 셰이더에 텍스처와 샘플러 바인딩
deviceContext->PSSetShaderResources(0, 1, &textureView);  // 레지스터 t0
deviceContext->PSSetSamplers(0, 1, &samplerState);        // 레지스터 s0
```

#### 4. HLSL에서 사용
```hlsl
// 텍스처와 샘플러 선언
Texture2D DiffuseTexture : register(t0);
SamplerState LinearSampler : register(s0);

// 픽셀 셰이더에서 텍스처 샘플링
float4 main(PS_INPUT input) : SV_TARGET
{
    // 텍스처에서 색상 샘플링
    float4 textureColor = DiffuseTexture.Sample(LinearSampler, input.TexCoord);
    return textureColor;
}
```

## 렌더링 파이프라인 설정과 그리기

### 전체 렌더링 과정

```cpp
void Render()
{
    // 1. 렌더 타겟 지우기
    float clearColor[4] = { 0.0f, 0.2f, 0.4f, 1.0f };
    deviceContext->ClearRenderTargetView(renderTargetView, clearColor);
    deviceContext->ClearDepthStencilView(depthStencilView, D3D11_CLEAR_DEPTH, 1.0f, 0);
    
    // 2. 셰이더 설정
    deviceContext->VSSetShader(vertexShader, nullptr, 0);
    deviceContext->PSSetShader(pixelShader, nullptr, 0);
    
    // 3. 입력 레이아웃 설정
    deviceContext->IASetInputLayout(inputLayout);
    
    // 4. 정점 버퍼 설정
    UINT stride = sizeof(Vertex);
    UINT offset = 0;
    deviceContext->IASetVertexBuffers(0, 1, &vertexBuffer, &stride, &offset);
    deviceContext->IASetIndexBuffer(indexBuffer, DXGI_FORMAT_R32_UINT, 0);
    
    // 5. 프리미티브 토폴로지 설정
    deviceContext->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
    
    // 6. 상수 버퍼 업데이트 및 바인딩
    UpdateConstantBuffer();
    deviceContext->VSSetConstantBuffers(0, 1, &constantBuffer);
    deviceContext->PSSetConstantBuffers(0, 1, &constantBuffer);
    
    // 7. 텍스처와 샘플러 바인딩
    deviceContext->PSSetShaderResources(0, 1, &textureView);
    deviceContext->PSSetSamplers(0, 1, &samplerState);
    
    // 8. 그리기
    deviceContext->DrawIndexed(indexCount, 0, 0);
    
    // 9. 화면에 표시
    swapChain->Present(1, 0);
}
```

## 셰이더 디버깅과 최적화

### 디버깅 팁

#### 1. 색상으로 디버깅
```hlsl
// 법선 벡터를 색상으로 표시
float4 main(PS_INPUT input) : SV_TARGET
{
    float3 normal = normalize(input.Normal);
    // 법선을 -1~1에서 0~1로 변환
    float3 debugColor = normal * 0.5 + 0.5;
    return float4(debugColor, 1.0);
}

// 텍스처 좌표를 색상으로 표시  
float4 main(PS_INPUT input) : SV_TARGET
{
    return float4(input.TexCoord, 0.0, 1.0);
}
```

#### 2. 조건부 컴파일
```hlsl
// 디버그 모드 매크로
#define DEBUG_NORMAL 0
#define DEBUG_TEXCOORD 1

float4 main(PS_INPUT input) : SV_TARGET
{
#if DEBUG_NORMAL
    float3 normal = normalize(input.Normal);
    return float4(normal * 0.5 + 0.5, 1.0);
#elif DEBUG_TEXCOORD
    return float4(input.TexCoord, 0.0, 1.0);
#else
    // 정상 렌더링
    return DiffuseTexture.Sample(LinearSampler, input.TexCoord);
#endif
}
```

### 성능 최적화

#### 1. 셰이더 복잡도 줄이기
```hlsl
// 비효율적: 복잡한 계산을 픽셀 셰이더에서
float4 main(PS_INPUT input) : SV_TARGET
{
    // 매 픽셀마다 복잡한 계산 실행
    float3 worldPos = mul(input.LocalPos, WorldMatrix);
    float3 lightDir = normalize(LightPos - worldPos);
    // ...
}

// 효율적: 복잡한 계산을 정점 셰이더에서
VS_OUTPUT main(VS_INPUT input)
{
    VS_OUTPUT output;
    output.WorldPos = mul(input.Position, World);
    output.LightDir = normalize(LightPos - output.WorldPos); // 여기서 계산
    return output;
}
```

#### 2. 브랜치 최소화
```hlsl
// 비효율적: 브랜치 사용
if (intensity > 0.5)
    color = float3(1, 1, 1);
else
    color = float3(0, 0, 0);

// 효율적: step 함수 사용
color = step(0.5, intensity) * float3(1, 1, 1);
```

## 고급 셰이더 기법

### 1. 다중 렌더 타겟 (MRT)

#### HLSL 코드
```hlsl
struct PS_OUTPUT
{
    float4 Albedo    : SV_TARGET0;  // 기본 색상
    float4 Normal    : SV_TARGET1;  // 법선 (월드 스페이스)
    float4 Position  : SV_TARGET2;  // 위치 (월드 스페이스)
    float4 Material  : SV_TARGET3;  // 재질 정보 (roughness, metallic 등)
};

PS_OUTPUT main(PS_INPUT input)
{
    PS_OUTPUT output;
    
    output.Albedo = DiffuseTexture.Sample(LinearSampler, input.TexCoord);
    output.Normal = float4(normalize(input.WorldNormal), 1.0);
    output.Position = float4(input.WorldPosition, 1.0);
    output.Material = float4(roughness, metallic, 0.0, 1.0);
    
    return output;
}
```

### 2. 인스턴싱 (Instancing)

#### HLSL 코드 (동일한 메시를 여러 개 그리기)
```hlsl
// 인스턴스 데이터 구조체
struct InstanceData
{
    matrix World;
    float4 Color;
};

// 인스턴스 데이터 버퍼
StructuredBuffer<InstanceData> InstanceBuffer : register(t0);

VS_OUTPUT main(VS_INPUT input, uint instanceID : SV_InstanceID)
{
    VS_OUTPUT output;
    
    // 현재 인스턴스의 데이터 가져오기
    InstanceData instance = InstanceBuffer[instanceID];
    
    // 인스턴스별 월드 변환 적용
    float4 worldPos = mul(float4(input.Position, 1.0), instance.World);
    float4 viewPos = mul(worldPos, View);
    output.Position = mul(viewPos, Projection);
    
    output.Color = instance.Color;
    return output;
}
```

셰이더는 DirectX 11에서 현대적인 그래픽을 구현하기 위한 핵심 기술입니다. GPU의 강력한 병렬 처리 능력을 활용하여 복잡한 시각 효과를 실시간으로 만들어낼 수 있습니다!
