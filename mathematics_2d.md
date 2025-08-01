# DirectX 11을 위한 2D 수학 기초

## 수학이 DirectX에서 왜 중요한가?

DirectX 11로 게임이나 그래픽 애플리케이션을 만들려면 **수학**이 필수입니다. 화면에 보이는 모든 것들 - 캐릭터의 움직임, 회전하는 바퀴, 날아가는 총알 - 이 모든 것이 수학 계산의 결과입니다.

컴퓨터 그래픽에서 수학은 다음과 같은 역할을 합니다:

```
현실 세계의 물리 법칙
        ↓
   수학적 공식화
        ↓
  프로그래밍 코드
        ↓
    화면 출력
```

예를 들어, 공이 튀는 모습을 화면에 그리려면:
1. **물리학**: 중력, 충돌, 탄성 등의 법칙
2. **수학**: 이를 식으로 표현 (위치 = 초기위치 + 속도×시간 + ½×가속도×시간²)
3. **프로그래밍**: 수학 공식을 코드로 구현
4. **DirectX**: 계산 결과를 화면에 그리기

## 좌표계 (Coordinate System)

### 화면 좌표계 vs 수학 좌표계

우리가 수학 시간에 배운 좌표계와 컴퓨터 화면의 좌표계는 조금 다릅니다.

#### 수학 좌표계 (일반적인 xy 평면)
```
      y
      ↑
      |
      |
      |
------+------→ x
      |     
      |
      |
```
- 원점 (0,0)이 중앙
- x축: 오른쪽이 양수
- y축: 위쪽이 양수

#### 화면 좌표계 (Screen Space)
```
(0,0)------→ x
  |
  |
  |
  ↓
  y
```
- 원점 (0,0)이 왼쪽 위 모서리
- x축: 오른쪽이 양수
- y축: 아래쪽이 양수

#### DirectX에서 자주 사용하는 좌표계들

**1. Screen Space (화면 좌표)**
```cpp
// 1920x1080 화면에서
// 왼쪽 위: (0, 0)
// 오른쪽 아래: (1919, 1079)
```

**2. NDC (Normalized Device Coordinates)**
```cpp
// DirectX에서 최종 출력 좌표
// 왼쪽 위: (-1, 1)
// 오른쪽 아래: (1, -1)
// 중앙: (0, 0)
```

**3. UV 좌표 (텍스처 좌표)**
```cpp
// 텍스처 맵핑용 좌표
// 왼쪽 위: (0, 0)
// 오른쪽 아래: (1, 1)
```

### 좌표 변환의 필요성

게임에서는 여러 좌표계를 오가며 계산해야 합니다:

```cpp
// 예시: 마우스 클릭으로 게임 객체 선택
마우스 화면 좌표 (534, 267)
        ↓
게임 월드 좌표 (15.2, -8.7)
        ↓
객체가 클릭됐는지 판정
```

## 벡터 (Vector) - 방향과 크기를 나타내는 도구

### 벡터란 무엇인가?

**벡터**는 **크기(길이)**와 **방향**을 모두 가진 수학적 개념입니다. 화살표로 표현할 수 있습니다.

#### 스칼라 vs 벡터
```cpp
// 스칼라 (크기만 있음)
float speed = 50.0f;        // 속력: 50
float temperature = 25.0f;   // 온도: 25도

// 벡터 (크기 + 방향)
Vector2 velocity = {30, 40}; // 속도: 크기 50, 방향 northeast
Vector2 force = {0, -9.8};   // 중력: 크기 9.8, 방향 down
```

#### 벡터의 표현
2D 벡터는 두 개의 숫자로 표현됩니다:

```cpp
struct Vector2 {
    float x;  // x 성분
    float y;  // y 성분
};

Vector2 position = {3.0f, 4.0f};  // 점 (3, 4)를 나타내는 벡터
```

시각적으로는 원점에서 시작하는 화살표로 그릴 수 있습니다:
```
      y
      ↑
    4 |   ●  ← (3, 4)
      |  /
    3 | /
      |/
------+------→ x
      0   3
```

### 벡터의 기본 연산

#### 1. 벡터의 덧셈
두 벡터를 더하면 각 성분끼리 더해집니다:

```cpp
Vector2 a = {3, 2};
Vector2 b = {1, 4};
Vector2 result = a + b;  // {4, 6}
```

**기하학적 의미**: 화살표를 이어 붙이기
```
첫 번째 벡터 a로 이동 → 그 지점에서 두 번째 벡터 b로 이동
```

**게임에서의 활용**:
```cpp
// 캐릭터 이동
Vector2 currentPosition = {10, 20};
Vector2 movement = {5, -3};
Vector2 newPosition = currentPosition + movement;  // {15, 17}

// 힘의 합성
Vector2 gravity = {0, -9.8};
Vector2 wind = {2, 0};
Vector2 totalForce = gravity + wind;  // {2, -9.8}
```

#### 2. 벡터의 뺄셈
```cpp
Vector2 a = {5, 7};
Vector2 b = {2, 3};
Vector2 result = a - b;  // {3, 4}
```

**기하학적 의미**: b에서 a로 가는 방향벡터
```cpp
// 두 점 사이의 방향 구하기
Vector2 playerPos = {100, 200};
Vector2 enemyPos = {150, 180};
Vector2 direction = enemyPos - playerPos;  // {50, -20}
// 플레이어에서 적으로 가는 방향
```

#### 3. 스칼라 곱셈
벡터에 숫자를 곱하면 길이가 변합니다:

```cpp
Vector2 v = {3, 4};
Vector2 doubled = v * 2;    // {6, 8} - 길이 2배
Vector2 halved = v * 0.5f;  // {1.5, 2} - 길이 절반
Vector2 opposite = v * -1;  // {-3, -4} - 반대 방향
```

### 벡터의 크기 (길이)

벡터의 크기는 피타고라스 정리로 구할 수 있습니다:

```cpp
float Length(Vector2 v) {
    return sqrt(v.x * v.x + v.y * v.y);
}

Vector2 v = {3, 4};
float length = Length(v);  // sqrt(9 + 16) = sqrt(25) = 5
```

**활용 예시**:
```cpp
// 두 점 사이의 거리
Vector2 distance = enemyPos - playerPos;
float distanceLength = Length(distance);

if (distanceLength < 50.0f) {
    // 적이 50 픽셀 이내에 있으면 공격
    Attack();
}
```

### 단위 벡터 (Unit Vector)

크기가 1인 벡터를 **단위 벡터**라고 합니다. 순수하게 방향만 나타냅니다.

```cpp
Vector2 Normalize(Vector2 v) {
    float length = Length(v);
    if (length == 0) return {0, 0};
    return {v.x / length, v.y / length};
}

Vector2 direction = {6, 8};
Vector2 unitDirection = Normalize(direction);  // {0.6, 0.8}
```

**활용**: 일정한 속도로 이동하기
```cpp
// 적을 향해 일정한 속도로 이동하는 미사일
Vector2 missilePos = {0, 0};
Vector2 targetPos = {100, 100};

Vector2 direction = Normalize(targetPos - missilePos);
float speed = 200.0f;  // 초당 200 픽셀
missilePos += direction * speed * deltaTime;
```

### 내적 (Dot Product)

두 벡터의 **내적**은 매우 유용한 연산입니다:

```cpp
float Dot(Vector2 a, Vector2 b) {
    return a.x * b.x + a.y * b.y;
}
```

#### 내적의 기하학적 의미

```cpp
float dot = Dot(a, b);
```

- `dot > 0`: 두 벡터가 같은 방향 (예각)
- `dot = 0`: 두 벡터가 수직 (직각)
- `dot < 0`: 두 벡터가 반대 방향 (둔각)

#### 내적의 활용

**1. 각도 계산**
```cpp
// 두 벡터 사이의 각도 (라디안)
float angle = acos(Dot(Normalize(a), Normalize(b)));
```

**2. 투영 (Projection)**
```cpp
// 벡터 a를 벡터 b 방향으로 투영
Vector2 b_unit = Normalize(b);
float projection_length = Dot(a, b_unit);
Vector2 projection = b_unit * projection_length;
```

**3. 게임에서의 실용적 활용**
```cpp
// 적이 플레이어 앞쪽에 있는지 확인
Vector2 playerForward = {1, 0};  // 플레이어가 바라보는 방향
Vector2 toEnemy = Normalize(enemyPos - playerPos);
float dot = Dot(playerForward, toEnemy);

if (dot > 0.5f) {  // 약 60도 범위 안
    // 적이 앞쪽에 있음 - 발견!
    DetectEnemy();
}
```

## 행렬 (Matrix) - 좌표 변환의 핵심

### 행렬이란?

**행렬**은 숫자를 사각형 모양으로 배열한 것입니다. DirectX에서는 주로 **좌표 변환**에 사용됩니다.

#### 2x2 행렬
```cpp
struct Matrix2x2 {
    float m[2][2];
    // m[0][0]  m[0][1]
    // m[1][0]  m[1][1]
};
```

#### 3x3 행렬 (2D 변환용)
```cpp
struct Matrix3x3 {
    float m[3][3];
    // m[0][0]  m[0][1]  m[0][2]
    // m[1][0]  m[1][1]  m[1][2]
    // m[2][0]  m[2][1]  m[2][2]
};
```

### 행렬과 벡터의 곱셈

행렬에 벡터를 곱하면 벡터가 **변환**됩니다:

```cpp
Vector2 MultiplyMatrixVector(Matrix2x2 matrix, Vector2 vector) {
    return {
        matrix.m[0][0] * vector.x + matrix.m[0][1] * vector.y,
        matrix.m[1][0] * vector.x + matrix.m[1][1] * vector.y
    };
}
```

### 기본 변환 행렬들

#### 1. 항등 행렬 (Identity Matrix)
벡터를 변화시키지 않는 행렬:

```cpp
Matrix2x2 identity = {
    1, 0,
    0, 1
};
// 어떤 벡터와 곱해도 원래 벡터가 나옴
```

#### 2. 크기 변환 (Scale)
벡터의 크기를 변경하는 행렬:

```cpp
Matrix2x2 CreateScale(float scaleX, float scaleY) {
    return {
        scaleX, 0,
        0,      scaleY
    };
}

// 사용 예시
Matrix2x2 scale = CreateScale(2.0f, 1.5f);  // x축 2배, y축 1.5배
Vector2 original = {3, 4};
Vector2 scaled = MultiplyMatrixVector(scale, original);  // {6, 6}
```

#### 3. 회전 변환 (Rotation)
벡터를 원점 중심으로 회전시키는 행렬:

```cpp
Matrix2x2 CreateRotation(float angleRadians) {
    float cos_a = cos(angleRadians);
    float sin_a = sin(angleRadians);
    
    return {
        cos_a, -sin_a,
        sin_a,  cos_a
    };
}

// 사용 예시
Matrix2x2 rotation = CreateRotation(PI / 4);  // 45도 회전
Vector2 original = {1, 0};
Vector2 rotated = MultiplyMatrixVector(rotation, original);  // {0.707, 0.707}
```

#### 4. 이동 변환 (Translation) - 3x3 행렬 필요

2D에서 이동 변환을 행렬로 표현하려면 **동차 좌표**를 사용해야 합니다:

```cpp
struct Vector3 {
    float x, y, w;  // w는 보통 1
};

Matrix3x3 CreateTranslation(float translateX, float translateY) {
    return {
        1, 0, translateX,
        0, 1, translateY,
        0, 0, 1
    };
}

Vector3 MultiplyMatrixVector3(Matrix3x3 matrix, Vector3 vector) {
    return {
        matrix.m[0][0] * vector.x + matrix.m[0][1] * vector.y + matrix.m[0][2] * vector.w,
        matrix.m[1][0] * vector.x + matrix.m[1][1] * vector.y + matrix.m[1][2] * vector.w,
        matrix.m[2][0] * vector.x + matrix.m[2][1] * vector.y + matrix.m[2][2] * vector.w
    };
}

// 사용 예시
Matrix3x3 translation = CreateTranslation(10, 5);
Vector3 original = {3, 4, 1};
Vector3 translated = MultiplyMatrixVector3(translation, original);  // {13, 9, 1}
```

### 행렬의 합성 (Matrix Composition)

여러 변환을 한 번에 적용하려면 행렬들을 곱합니다:

```cpp
Matrix3x3 MultiplyMatrix(Matrix3x3 a, Matrix3x3 b) {
    Matrix3x3 result = {};
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            for (int k = 0; k < 3; k++) {
                result.m[i][j] += a.m[i][k] * b.m[k][j];
            }
        }
    }
    return result;
}

// 복합 변환: 크기 → 회전 → 이동
Matrix3x3 scale = CreateScale(2.0f, 2.0f);
Matrix3x3 rotation = CreateRotation(PI / 6);  // 30도
Matrix3x3 translation = CreateTranslation(100, 50);

// 주의: 행렬 곱셈은 순서가 중요! (오른쪽부터 적용)
Matrix3x3 combined = MultiplyMatrix(translation, MultiplyMatrix(rotation, scale));
```

### DirectX에서의 행렬 사용

DirectX에서는 이런 변환들이 **셰이더**에서 실행됩니다:

```hlsl
// 정점 셰이더에서
cbuffer TransformBuffer : register(b0) {
    matrix WorldMatrix;      // 월드 변환 행렬
    matrix ViewMatrix;       // 뷰 변환 행렬
    matrix ProjectionMatrix; // 투영 변환 행렬
};

float4 main(float2 position : POSITION) : SV_POSITION {
    float4 worldPos = mul(float4(position, 0, 1), WorldMatrix);
    float4 viewPos = mul(worldPos, ViewMatrix);
    float4 clipPos = mul(viewPos, ProjectionMatrix);
    return clipPos;
}
```

## 삼각함수 (Trigonometry)

### 삼각함수의 기본

DirectX에서 삼각함수는 주로 **회전**과 **원형 운동**에 사용됩니다.

#### 단위원에서의 삼각함수
```
       y
       ↑
     1 |     
       |   ●(cos θ, sin θ)
       |  /|
       | / |
-------+---+-----→ x
       |   1
       |
      -1
```

- `cos(θ)` = x 좌표
- `sin(θ)` = y 좌표

#### 각도의 단위
```cpp
// 도(degree)를 라디안(radian)으로 변환
float DegreesToRadians(float degrees) {
    return degrees * PI / 180.0f;
}

// 라디안을 도로 변환
float RadiansToDegrees(float radians) {
    return radians * 180.0f / PI;
}
```

### 삼각함수의 활용

#### 1. 원형 운동
```cpp
// 시계 바늘, 행성 공전 등
float angle = 0;
float radius = 100;
float centerX = 400, centerY = 300;

void Update(float deltaTime) {
    angle += deltaTime;  // 시간에 따라 각도 증가
    
    float x = centerX + cos(angle) * radius;
    float y = centerY + sin(angle) * radius;
    
    // 객체를 (x, y) 위치에 그리기
}
```

#### 2. 진동 운동
```cpp
// 스프링, 진동 등
float amplitude = 50;  // 진폭
float frequency = 2;   // 주파수
float time = 0;

void Update(float deltaTime) {
    time += deltaTime;
    
    float offset = sin(time * frequency) * amplitude;
    float y = baseY + offset;  // 위아래로 진동
}
```

#### 3. 방향벡터 생성
```cpp
// 특정 각도로 발사체 발사
float angle = DegreesToRadians(45);  // 45도
float speed = 500;

Vector2 direction = {cos(angle), sin(angle)};
Vector2 velocity = direction * speed;
```

#### 4. 각도 계산
```cpp
// 두 점 사이의 각도 구하기
Vector2 from = {0, 0};
Vector2 to = {100, 100};
Vector2 direction = to - from;

float angle = atan2(direction.y, direction.x);  // 라디안
```

## 보간 (Interpolation)

### 보간이란?

**보간**은 두 값 사이의 중간값을 계산하는 것입니다. 게임에서 부드러운 애니메이션을 만드는 핵심 기술입니다.

### 선형 보간 (Linear Interpolation, Lerp)

가장 기본적인 보간 방법입니다:

```cpp
float Lerp(float start, float end, float t) {
    return start + (end - start) * t;
}

// t = 0: start 값
// t = 1: end 값
// t = 0.5: 중간값
```

#### 벡터 보간
```cpp
Vector2 Lerp(Vector2 start, Vector2 end, float t) {
    return {
        Lerp(start.x, end.x, t),
        Lerp(start.y, end.y, t)
    };
}

// 사용 예시: 부드러운 이동
Vector2 currentPos = {0, 0};
Vector2 targetPos = {100, 50};
float speed = 2.0f;  // 초당 2배속

void Update(float deltaTime) {
    float t = speed * deltaTime;
    if (t > 1.0f) t = 1.0f;  // 클램핑
    
    currentPos = Lerp(currentPos, targetPos, t);
}
```

### 이징 함수 (Easing Functions)

선형 보간보다 자연스러운 움직임을 만드는 함수들:

#### 1. Ease-In (서서히 빨라짐)
```cpp
float EaseIn(float t) {
    return t * t;
}
```

#### 2. Ease-Out (서서히 느려짐)
```cpp
float EaseOut(float t) {
    return 1 - (1 - t) * (1 - t);
}
```

#### 3. Ease-In-Out (양쪽 끝에서 느림)
```cpp
float EaseInOut(float t) {
    if (t < 0.5f) {
        return 2 * t * t;
    } else {
        return 1 - 2 * (1 - t) * (1 - t);
    }
}
```

### 각도 보간의 주의점

각도는 순환적 성질 때문에 특별한 처리가 필요합니다:

```cpp
float LerpAngle(float startAngle, float endAngle, float t) {
    // 각도 차이가 180도보다 크면 반대 방향으로 회전
    float diff = endAngle - startAngle;
    
    if (diff > PI) {
        endAngle -= 2 * PI;
    } else if (diff < -PI) {
        endAngle += 2 * PI;
    }
    
    return Lerp(startAngle, endAngle, t);
}
```

## 충돌 검사 (2D Collision Detection)

### 점과 사각형 충돌

```cpp
bool IsPointInRectangle(Vector2 point, Vector2 rectPos, Vector2 rectSize) {
    return (point.x >= rectPos.x && 
            point.x <= rectPos.x + rectSize.x &&
            point.y >= rectPos.y && 
            point.y <= rectPos.y + rectSize.y);
}
```

### 원과 원 충돌

```cpp
bool IsCircleColliding(Vector2 center1, float radius1, 
                      Vector2 center2, float radius2) {
    Vector2 distance = center1 - center2;
    float distanceLength = Length(distance);
    return distanceLength < (radius1 + radius2);
}
```

### 사각형과 사각형 충돌 (AABB)

```cpp
bool IsRectangleColliding(Vector2 pos1, Vector2 size1,
                         Vector2 pos2, Vector2 size2) {
    return (pos1.x < pos2.x + size2.x &&
            pos1.x + size1.x > pos2.x &&
            pos1.y < pos2.y + size2.y &&
            pos1.y + size1.y > pos2.y);
}
```

## DirectX에서의 실제 활용

### 1. 2D 스프라이트 렌더링

```cpp
// 스프라이트 변환 행렬 생성
Matrix3x3 CreateSpriteTransform(Vector2 position, float rotation, Vector2 scale) {
    Matrix3x3 translation = CreateTranslation(position.x, position.y);
    Matrix3x3 rotationMat = CreateRotation(rotation);
    Matrix3x3 scaleMat = CreateScale(scale.x, scale.y);
    
    // 크기 → 회전 → 이동 순서
    return MultiplyMatrix(translation, MultiplyMatrix(rotationMat, scaleMat));
}
```

### 2. 카메라 변환

```cpp
// 2D 카메라 뷰 행렬
Matrix3x3 CreateCameraMatrix(Vector2 cameraPos, float cameraRotation, float cameraZoom) {
    // 카메라 변환은 월드 변환의 역변환
    Matrix3x3 translation = CreateTranslation(-cameraPos.x, -cameraPos.y);
    Matrix3x3 rotation = CreateRotation(-cameraRotation);
    Matrix3x3 scale = CreateScale(cameraZoom, cameraZoom);
    
    return MultiplyMatrix(scale, MultiplyMatrix(rotation, translation));
}
```

### 3. 화면 좌표 변환

```cpp
// NDC(-1~1)를 화면 좌표(0~width, 0~height)로 변환
Vector2 NDCToScreen(Vector2 ndc, float screenWidth, float screenHeight) {
    return {
        (ndc.x + 1.0f) * 0.5f * screenWidth,
        (1.0f - ndc.y) * 0.5f * screenHeight  // y축 뒤집기
    };
}

// 화면 좌표를 NDC로 변환
Vector2 ScreenToNDC(Vector2 screen, float screenWidth, float screenHeight) {
    return {
        (screen.x / screenWidth) * 2.0f - 1.0f,
        1.0f - (screen.y / screenHeight) * 2.0f
    };
}
```

## 성능 최적화 팁

### 1. 제곱근 회피

거리 비교 시 제곱근 계산을 피하고 거리의 제곱을 비교:

```cpp
// 비효율적
float distance = sqrt((a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y));
if (distance < 100) { /* ... */ }

// 효율적
float distanceSquared = (a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y);
if (distanceSquared < 100 * 100) { /* ... */ }
```

### 2. 삼각함수 최적화

자주 사용되는 각도는 미리 계산해두기:

```cpp
// 시작 시 미리 계산
float sin_table[360];
float cos_table[360];

for (int i = 0; i < 360; i++) {
    sin_table[i] = sin(DegreesToRadians(i));
    cos_table[i] = cos(DegreesToRadians(i));
}

// 사용 시 테이블 조회
float FastSin(int degrees) {
    return sin_table[degrees % 360];
}
```

### 3. 정수 연산 활용

가능한 경우 정수 연산 사용:

```cpp
// 타일 기반 게임에서 정수 좌표 사용
struct TilePos {
    int x, y;
};

TilePos WorldToTile(Vector2 worldPos, float tileSize) {
    return {
        (int)(worldPos.x / tileSize),
        (int)(worldPos.y / tileSize)
    };
}
```

## 정리

2D DirectX 프로그래밍을 위한 수학의 핵심은:

### 필수 개념들
1. **좌표계**: 화면, NDC, UV 좌표계 이해
2. **벡터**: 방향과 크기를 표현하는 도구
3. **행렬**: 좌표 변환의 핵심 (이동, 회전, 크기)
4. **삼각함수**: 회전과 원형 운동
5. **보간**: 부드러운 애니메이션

### 실무 활용
- **스프라이트 렌더링**: 변환 행렬로 위치, 회전, 크기 설정
- **카메라 시스템**: 뷰 변환으로 월드를 화면에 투영
- **충돌 검사**: 게임 객체 간 상호작용
- **애니메이션**: 보간으로 부드러운 움직임 구현

이 2D 수학 기초를 확실히 이해하면, 나중에 3D 수학을 배울 때도 훨씬 쉽게 이해할 수 있습니다!
