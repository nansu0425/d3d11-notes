# DirectX 11을 위한 2D 물리 기초

## 물리학이 게임에서 왜 중요한가?

게임에서 물리학은 **현실감**과 **재미**를 만드는 핵심 요소입니다. 캐릭터가 점프할 때 자연스럽게 포물선을 그리거나, 총알이 날아가고, 공이 벽에 튕기는 모든 것들이 물리학의 법칙을 따릅니다.

물리학 없는 게임은 마치 인형극처럼 어색해 보입니다:

```
물리학 없음: 캐릭터가 뚝뚝 끊어지며 이동
     ↓
물리학 적용: 부드럽고 자연스러운 움직임
     ↓
플레이어의 몰입감 증가
```

### 게임에서 자주 보는 물리 현상들

- **중력**: 캐릭터가 떨어지는 것
- **관성**: 빙판에서 미끄러지는 것
- **충돌**: 공이 벽에 튕기는 것
- **마찰**: 자동차가 서서히 멈추는 것
- **탄성**: 용수철이나 고무공의 탄력

## 기본 물리량들

### 위치 (Position)

**위치**는 객체가 공간상 어디에 있는지를 나타냅니다.

```cpp
struct Vector2 {
    float x, y;
};

Vector2 position = {100.0f, 200.0f};  // 화면상 (100, 200) 위치
```

### 속도 (Velocity)

**속도**는 위치의 변화율입니다. 즉, 시간당 얼마나 이동하는가를 나타냅니다.

```cpp
Vector2 velocity = {50.0f, 30.0f};  // 초당 오른쪽으로 50, 위로 30 이동
```

**속도의 의미**:
- **크기**: 얼마나 빠른가 (속력)
- **방향**: 어느 방향으로 움직이는가

#### 속도로 위치 업데이트
```cpp
void UpdatePosition(float deltaTime) {
    // 위치 = 이전 위치 + 속도 × 시간
    position.x += velocity.x * deltaTime;
    position.y += velocity.y * deltaTime;
}

// 예시: 60fps 게임에서 deltaTime = 1/60 = 0.0167초
// 속도가 (60, 0)이면 1초에 60픽셀, 1프레임에 1픽셀 이동
```

### 가속도 (Acceleration)

**가속도**는 속도의 변화율입니다. 힘이 가해지면 가속도가 생깁니다.

```cpp
Vector2 acceleration = {0.0f, -980.0f};  // 중력 가속도 (아래 방향)
```

#### 가속도로 속도 업데이트
```cpp
void UpdateVelocity(float deltaTime) {
    // 속도 = 이전 속도 + 가속도 × 시간
    velocity.x += acceleration.x * deltaTime;
    velocity.y += acceleration.y * deltaTime;
}
```

### 질량 (Mass)

**질량**은 물체의 무게와 관성을 결정합니다.

```cpp
float mass = 1.0f;  // kg 단위 (게임에서는 상대적 값)
```

**질량의 효과**:
- 질량이 클수록 → 같은 힘으로도 천천히 가속
- 질량이 작을수록 → 같은 힘으로 빠르게 가속

## 뉴턴의 운동 법칙

### 1법칙: 관성 법칙

> **정지한 물체는 계속 정지하고, 움직이는 물체는 계속 직진한다** (힘이 가해지지 않는 한)

```cpp
// 관성 구현: 힘이 없으면 등속도 운동
void UpdateWithInertia(float deltaTime) {
    if (noForceApplied) {
        // 속도 변화 없음 - 계속 같은 속도로 이동
        position += velocity * deltaTime;
    }
}
```

**게임에서의 활용**:
- 우주선 게임: 추진력 없으면 계속 직진
- 에어하키: 퍽이 마찰 없이 계속 움직임

### 2법칙: F = ma (힘 = 질량 × 가속도)

> **힘이 클수록, 질량이 작을수록 가속도가 크다**

```cpp
Vector2 CalculateAcceleration(Vector2 force, float mass) {
    // a = F / m
    return Vector2{force.x / mass, force.y / mass};
}

void ApplyForce(Vector2 force, float deltaTime) {
    Vector2 acceleration = CalculateAcceleration(force, mass);
    velocity += acceleration * deltaTime;
}
```

**게임에서의 활용**:
```cpp
// 중력 적용
Vector2 gravity = {0, -980.0f};  // 9.8m/s² × 100 (픽셀 단위, 아래 방향)
ApplyForce(gravity * mass, deltaTime);

// 엔진 추진력
Vector2 thrustForce = {500.0f, 0};
if (inputPressed) {
    ApplyForce(thrustForce, deltaTime);
}
```

### 3법칙: 작용-반작용

> **모든 작용에는 크기가 같고 방향이 반대인 반작용이 있다**

```cpp
void HandleCollision(Object& obj1, Object& obj2) {
    Vector2 force = CalculateCollisionForce(obj1, obj2);
    
    // 작용
    obj1.ApplyForce(force, deltaTime);
    
    // 반작용 (반대 방향)
    obj2.ApplyForce(-force, deltaTime);
}
```

## 운동학 (Kinematics)

### 등속도 운동

힘이 없을 때의 운동입니다.

```cpp
// s = v × t (거리 = 속도 × 시간)
Vector2 displacement = velocity * deltaTime;
position += displacement;
```

### 등가속도 운동

일정한 힘이 가해질 때의 운동입니다.

#### 기본 공식들
```cpp
// 1. v = v₀ + at (속도 = 초기속도 + 가속도×시간)
Vector2 finalVelocity = initialVelocity + acceleration * time;

// 2. s = v₀t + ½at² (거리 = 초기속도×시간 + ½×가속도×시간²)
Vector2 displacement = initialVelocity * time + 0.5f * acceleration * time * time;

// 3. v² = v₀² + 2as (속도² = 초기속도² + 2×가속도×거리)
// 점프 높이 계산 등에 사용
```

#### 실제 구현 (오일러 적분)
```cpp
void UpdatePhysics(float deltaTime) {
    // 1. 힘으로부터 가속도 계산
    Vector2 acceleration = totalForce / mass;
    
    // 2. 가속도로 속도 업데이트
    velocity += acceleration * deltaTime;
    
    // 3. 속도로 위치 업데이트
    position += velocity * deltaTime;
    
    // 4. 다음 프레임을 위해 힘 초기화
    totalForce = {0, 0};
}
```

### 자유낙하

중력만 작용하는 운동입니다.

```cpp
class FallingObject {
private:
    Vector2 position;
    Vector2 velocity;
    const float gravity = -980.0f;  // 9.8m/s² × 100 (아래 방향)

public:
    void Update(float deltaTime) {
        // 중력 가속도 적용
        velocity.y += gravity * deltaTime;
        
        // 위치 업데이트
        position += velocity * deltaTime;
        
        // 땅에 닿으면 정지
        if (position.y <= groundLevel) {
            position.y = groundLevel;
            velocity.y = 0;
        }
    }
};
```

### 포물선 운동

초기 속도를 가지고 중력의 영향을 받는 운동입니다.

```cpp
class Projectile {
private:
    Vector2 position;
    Vector2 velocity;
    Vector2 initialVelocity;
    Vector2 startPosition;        // 시작 위치 저장
    const Vector2 gravity = {0, -980.0f};  // 아래 방향
    float elapsedTime;

public:
    void Launch(Vector2 startPos, float angle, float speed) {
        startPosition = startPos;     // 시작 위치 저장
        position = startPos;
        elapsedTime = 0;
        
        // 각도와 속력으로 초기 속도 계산
        initialVelocity.x = speed * cos(angle);
        initialVelocity.y = speed * sin(angle);
        velocity = initialVelocity;
    }
    
    void Update(float deltaTime) {
        elapsedTime += deltaTime;
        
        // 포물선 운동 공식 적용
        position.x = startPosition.x + initialVelocity.x * elapsedTime;
        position.y = startPosition.y + initialVelocity.y * elapsedTime + 
                     0.5f * gravity.y * elapsedTime * elapsedTime;
        
        // 속도도 업데이트 (충돌 처리용)
        velocity.x = initialVelocity.x;
        velocity.y = initialVelocity.y + gravity.y * elapsedTime;
    }
};
```

**활용 예시**:
- 대포 게임의 포탄
- 플랫폼 게임의 점프
- 농구 게임의 공 던지기

## 힘과 에너지

### 중력 (Gravity)

지구가 모든 물체를 아래로 당기는 힘입니다.

```cpp
class GravitySystem {
private:
    const Vector2 earthGravity = {0, -980.0f};  // 9.8m/s² (아래 방향)

public:
    void ApplyGravity(GameObject& obj, float deltaTime) {
        Vector2 gravityForce = earthGravity * obj.mass;
        obj.ApplyForce(gravityForce, deltaTime);
    }
};
```

**다양한 중력 구현**:
```cpp
// 1. 일반 중력
Vector2 normalGravity = {0, -980.0f};

// 2. 달 중력 (지구의 1/6)
Vector2 moonGravity = {0, -163.0f};

// 3. 역중력 (위로 올라감)
Vector2 reverseGravity = {0, 980.0f};

// 4. 측면 중력 (벽에 붙는 효과)
Vector2 sideGravity = {-980.0f, 0};
```

### 마찰력 (Friction)

표면 간의 접촉으로 생기는 저항력입니다.

#### 정지 마찰과 운동 마찰
```cpp
class FrictionSystem {
private:
    float staticFriction = 0.8f;   // 정지 마찰 계수
    float kineticFriction = 0.6f;  // 운동 마찰 계수

public:
    Vector2 CalculateFriction(const GameObject& obj, Vector2 appliedForce) {
        float normalForce = obj.mass * 980.0f;  // 수직항력 (중력 크기)
        
        if (Length(obj.velocity) < 0.1f) {
            // 정지 상태: 정지 마찰
            float maxStaticFriction = staticFriction * normalForce;
            Vector2 opposingForce = -appliedForce;
            
            if (Length(opposingForce) > maxStaticFriction) {
                // 정지 마찰 한계 초과 → 운동 시작
                return Normalize(opposingForce) * maxStaticFriction;
            } else {
                // 정지 마찰로 힘 상쇄
                return opposingForce;
            }
        } else {
            // 운동 상태: 운동 마찰 (속도 반대 방향)
            Vector2 frictionDirection = -Normalize(obj.velocity);
            float kineticFrictionForce = kineticFriction * normalForce;
            return frictionDirection * kineticFrictionForce;
        }
    }
};
```

**간단한 마찰 구현**:
```cpp
void ApplySimpleFriction(float deltaTime) {
    float frictionCoefficient = 0.9f;  // 0~1 범위
    
    // 속도에 마찰 계수를 곱해서 감속
    velocity *= pow(frictionCoefficient, deltaTime);
    
    // 매우 작은 속도는 0으로 처리
    if (Length(velocity) < 0.1f) {
        velocity = {0, 0};
    }
}
```

### 공기 저항 (Air Resistance)

공기와의 마찰로 인한 저항력입니다.

```cpp
Vector2 CalculateAirResistance(Vector2 velocity, float dragCoefficient = 0.1f) {
    // 공기 저항 = -k × v² × 방향
    float speed = Length(velocity);
    if (speed > 0) {
        Vector2 direction = Normalize(velocity);
        float dragForce = dragCoefficient * speed * speed;
        return -direction * dragForce;
    }
    return {0, 0};
}

void Update(float deltaTime) {
    Vector2 airResistance = CalculateAirResistance(velocity);
    ApplyForce(airResistance, deltaTime);
    
    // 나머지 물리 업데이트...
}
```

### 탄성력 (Spring Force)

용수철이나 고무줄처럼 늘어나거나 압축되면 복원하려는 힘입니다.

#### 후크의 법칙 (Hooke's Law)
```cpp
class Spring {
private:
    float springConstant;    // 용수철 상수 (k)
    float restLength;        // 자연 길이
    Vector2 anchorPoint;     // 고정점

public:
    Vector2 CalculateSpringForce(Vector2 objectPosition) {
        // F = -k × (x - x₀)
        Vector2 displacement = objectPosition - anchorPoint;
        float currentLength = Length(displacement);
        float extension = currentLength - restLength;
        
        if (currentLength > 0) {
            Vector2 direction = Normalize(displacement);
            float springForce = -springConstant * extension;
            return direction * springForce;
        }
        
        return {0, 0};
    }
};
```

**활용 예시**:
```cpp
// 1. 플랫폼 게임의 점프대
Spring jumpPad(500.0f, 0.0f, platformPosition);

// 2. 카메라 따라가기 (부드러운 추적)
class SmoothCamera {
    Vector2 position;
    Vector2 target;
    float springStrength = 100.0f;
    float damping = 0.8f;
    
    void Update(float deltaTime) {
        Vector2 springForce = (target - position) * springStrength;
        velocity += springForce * deltaTime;
        velocity *= damping;  // 감쇠
        position += velocity * deltaTime;
    }
};
```

## 충돌 처리 (Collision Response)

### 충돌 감지 복습

기본적인 충돌 감지는 수학 문서에서 다뤘지만, 물리에서는 **충돌 후 어떻게 반응할지**가 중요합니다.

### 탄성 충돌 (Elastic Collision)

에너지가 보존되는 충돌입니다. 완전히 튕겨나갑니다.

#### 벽과의 충돌
```cpp
void HandleWallCollision(Vector2& velocity, Vector2 wallNormal) {
    // 속도를 벽의 법선 벡터에 대해 반사
    // v' = v - 2(v·n)n
    float dotProduct = Dot(velocity, wallNormal);
    velocity = velocity - 2.0f * dotProduct * wallNormal;
}

// 사용 예시
if (position.x <= 0) {  // 왼쪽 벽 충돌
    Vector2 leftWallNormal = {1, 0};
    HandleWallCollision(velocity, leftWallNormal);
    position.x = 0;
}
```

#### 두 물체 간의 충돌
```cpp
void HandleElasticCollision(GameObject& obj1, GameObject& obj2) {
    // 충돌 법선 (obj1에서 obj2로의 방향)
    Vector2 collisionNormal = Normalize(obj2.position - obj1.position);
    
    // 상대 속도
    Vector2 relativeVelocity = obj1.velocity - obj2.velocity;
    float velocityAlongNormal = Dot(relativeVelocity, collisionNormal);
    
    // 이미 분리되고 있으면 무시
    if (velocityAlongNormal > 0) return;
    
    // 충돌 후 속도 계산 (질량 보존 + 운동량 보존)
    float restitution = 1.0f;  // 완전 탄성 충돌
    float impulse = -(1 + restitution) * velocityAlongNormal;
    impulse /= (1/obj1.mass + 1/obj2.mass);
    
    Vector2 impulseVector = impulse * collisionNormal;
    obj1.velocity += impulseVector / obj1.mass;
    obj2.velocity -= impulseVector / obj2.mass;
}
```

### 비탄성 충돌 (Inelastic Collision)

에너지가 손실되는 충돌입니다. 반발 계수로 조절합니다.

```cpp
class CollisionSystem {
private:
    float restitution = 0.7f;  // 반발 계수 (0~1)

public:
    void HandleInelasticCollision(GameObject& obj1, GameObject& obj2) {
        Vector2 collisionNormal = Normalize(obj2.position - obj1.position);
        Vector2 relativeVelocity = obj1.velocity - obj2.velocity;
        float velocityAlongNormal = Dot(relativeVelocity, collisionNormal);
        
        if (velocityAlongNormal > 0) return;
        
        // 반발 계수 적용
        float impulse = -(1 + restitution) * velocityAlongNormal;
        impulse /= (1/obj1.mass + 1/obj2.mass);
        
        Vector2 impulseVector = impulse * collisionNormal;
        obj1.velocity += impulseVector / obj1.mass;
        obj2.velocity -= impulseVector / obj2.mass;
    }
};
```

**반발 계수의 의미**:
- `restitution = 1.0`: 완전 탄성 (고무공)
- `restitution = 0.7`: 농구공
- `restitution = 0.3`: 젖은 테니스공
- `restitution = 0.0`: 완전 비탄성 (점토)

### 충돌 위치 보정

충돌 후 물체들이 겹쳐있는 문제를 해결합니다.

```cpp
void CorrectPosition(GameObject& obj1, GameObject& obj2, float penetration, Vector2 normal) {
    const float percent = 0.8f;  // 보정 강도
    const float slop = 0.01f;    // 허용 겹침
    
    float correctionAmount = max(penetration - slop, 0.0f) * percent;
    Vector2 correction = correctionAmount * normal / (1/obj1.mass + 1/obj2.mass);
    
    obj1.position -= correction / obj1.mass;
    obj2.position += correction / obj2.mass;
}
```

## 원형 운동 (Circular Motion)

### 등속 원운동

일정한 속력으로 원을 그리는 운동입니다.

```cpp
class OrbitingObject {
private:
    Vector2 center;
    float radius;
    float angularSpeed;  // 각속도 (라디안/초)
    float currentAngle;

public:
    void Update(float deltaTime) {
        currentAngle += angularSpeed * deltaTime;
        
        position.x = center.x + radius * cos(currentAngle);
        position.y = center.y + radius * sin(currentAngle);
        
        // 속도 계산 (접선 방향)
        velocity.x = -radius * angularSpeed * sin(currentAngle);
        velocity.y = radius * angularSpeed * cos(currentAngle);
    }
};
```

### 구심력 (Centripetal Force)

원운동을 유지하는 데 필요한 힘입니다.

```cpp
Vector2 CalculateCentripetalForce(float mass, float speed, float radius, Vector2 centerDirection) {
    // F = mv²/r (중심 방향)
    float centripetal = mass * speed * speed / radius;
    return centerDirection * centripetal;
}

// 행성 공전 시뮬레이션
void UpdateOrbit(float deltaTime) {
    Vector2 toCenter = center - position;
    float distance = Length(toCenter);
    Vector2 centerDirection = Normalize(toCenter);
    
    float speed = Length(velocity);
    Vector2 centripetalForce = CalculateCentripetalForce(mass, speed, distance, centerDirection);
    
    ApplyForce(centripetalForce, deltaTime);
}
```

## 관절과 제약 (Constraints)

### 거리 제약 (Distance Constraint)

두 점 사이의 거리를 일정하게 유지합니다.

```cpp
class DistanceConstraint {
private:
    GameObject* obj1;
    GameObject* obj2;
    float targetDistance;
    float stiffness;

public:
    void Solve() {
        Vector2 delta = obj2->position - obj1->position;
        float currentDistance = Length(delta);
        
        if (currentDistance > 0) {
            float difference = (targetDistance - currentDistance) / currentDistance;
            Vector2 correction = delta * difference * 0.5f * stiffness;
            
            obj1->position -= correction;
            obj2->position += correction;
        }
    }
};
```

**활용 예시**:
- 체인, 로프, 줄
- 관절 연결
- 천 시뮬레이션

### 위치 제약 (Position Constraint)

특정 위치에 고정시킵니다.

```cpp
class PinConstraint {
private:
    GameObject* object;
    Vector2 pinPosition;
    float stiffness;

public:
    void Solve() {
        Vector2 correction = (pinPosition - object->position) * stiffness;
        object->position += correction;
    }
};
```

## 유체 동역학 기초

### 부력 (Buoyancy)

물체가 유체에서 받는 위쪽 힘입니다.

```cpp
class BuoyancySystem {
private:
    float waterLevel;
    float waterDensity;
    float gravity;

public:
    Vector2 CalculateBuoyancy(GameObject& obj) {
        float submergedVolume = CalculateSubmergedVolume(obj, waterLevel);
        
        if (submergedVolume > 0) {
            // 부력 = 물의 밀도 × 잠긴 부피 × 중력
            float buoyancyForce = waterDensity * submergedVolume * gravity;
            return Vector2{0, buoyancyForce};  // 위쪽으로
        }
        
        return {0, 0};
    }
    
private:
    float CalculateSubmergedVolume(GameObject& obj, float waterLevel) {
        if (obj.position.y - obj.radius > waterLevel) {
            return 0;  // 완전히 물 위
        } else if (obj.position.y + obj.radius < waterLevel) {
            return obj.volume;  // 완전히 물 속
        } else {
            // 부분적으로 잠김 - 원의 일부 부피 계산
            return CalculatePartialCircleVolume(obj, waterLevel);
        }
    }
};
```

### 점성 (Viscosity)

유체의 끈적거림으로 인한 저항입니다.

```cpp
Vector2 CalculateViscosity(Vector2 velocity, float viscosityCoefficient) {
    // 점성력 = -k × v (속도에 비례하는 저항)
    return -velocity * viscosityCoefficient;
}

void UpdateInWater(float deltaTime) {
    if (isInWater) {
        Vector2 buoyancy = buoyancySystem.CalculateBuoyancy(*this);
        Vector2 viscosity = CalculateViscosity(velocity, 0.5f);
        
        ApplyForce(buoyancy, deltaTime);
        ApplyForce(viscosity, deltaTime);
    }
}
```

## 실제 게임 구현 예시

### 1. 플랫폼 게임 캐릭터

```cpp
class PlatformCharacter {
private:
    Vector2 position;
    Vector2 velocity;
    float mass = 1.0f;
    bool isGrounded = false;
    
    // 물리 상수들
    const float gravity = -980.0f;  // 아래 방향
    const float jumpForce = 400.0f;  // 위쪽으로 점프
    const float moveSpeed = 200.0f;
    const float groundFriction = 0.8f;
    const float airFriction = 0.95f;

public:
    void Update(float deltaTime) {
        HandleInput();
        ApplyPhysics(deltaTime);
        CheckCollisions();
        UpdatePosition(deltaTime);
    }
    
private:
    void HandleInput() {
        // 좌우 이동
        if (IsKeyPressed('A')) {
            velocity.x = -moveSpeed;
        } else if (IsKeyPressed('D')) {
            velocity.x = moveSpeed;
        }
        
        // 점프 (땅에 있을 때만)
        if (IsKeyPressed(VK_SPACE) && isGrounded) {
            velocity.y = jumpForce;
            isGrounded = false;
        }
    }
    
    void ApplyPhysics(float deltaTime) {
        // 중력 적용
        if (!isGrounded) {
            velocity.y += gravity * deltaTime;
        }
        
        // 마찰 적용
        float frictionCoeff = isGrounded ? groundFriction : airFriction;
        velocity.x *= pow(frictionCoeff, deltaTime);
    }
    
    void CheckCollisions() {
        // 땅과의 충돌
        if (position.y <= groundLevel) {
            position.y = groundLevel;
            velocity.y = 0;
            isGrounded = true;
        }
        
        // 벽과의 충돌
        if (position.x <= 0 || position.x >= screenWidth) {
            velocity.x = 0;
            position.x = Clamp(position.x, 0, screenWidth);
        }
    }
};
```

### 2. 당구 게임

```cpp
class BilliardBall {
private:
    Vector2 position;
    Vector2 velocity;
    float radius;
    float mass;
    const float friction = 0.98f;
    const float restitution = 0.9f;

public:
    void Update(float deltaTime) {
        // 마찰로 감속
        velocity *= pow(friction, deltaTime);
        
        // 매우 작은 속도는 정지
        if (Length(velocity) < 1.0f) {
            velocity = {0, 0};
        }
        
        // 위치 업데이트
        position += velocity * deltaTime;
        
        // 테이블 경계와 충돌
        HandleTableCollision();
    }
    
    void HandleBallCollision(BilliardBall& other) {
        Vector2 delta = other.position - position;
        float distance = Length(delta);
        float minDistance = radius + other.radius;
        
        if (distance < minDistance) {
            // 충돌 법선
            Vector2 normal = Normalize(delta);
            
            // 겹침 보정
            float overlap = minDistance - distance;
            Vector2 correction = normal * (overlap * 0.5f);
            position -= correction;
            other.position += correction;
            
            // 탄성 충돌 처리
            Vector2 relativeVelocity = velocity - other.velocity;
            float impulse = -2 * Dot(relativeVelocity, normal) / (1/mass + 1/other.mass);
            
            Vector2 impulseVector = normal * impulse;
            velocity += impulseVector / mass * restitution;
            other.velocity -= impulseVector / other.mass * restitution;
        }
    }
    
private:
    void HandleTableCollision() {
        // 좌우 벽
        if (position.x - radius <= 0) {
            position.x = radius;
            velocity.x = -velocity.x * restitution;
        } else if (position.x + radius >= tableWidth) {
            position.x = tableWidth - radius;
            velocity.x = -velocity.x * restitution;
        }
        
        // 상하 벽
        if (position.y + radius >= tableHeight) {
            position.y = tableHeight - radius;
            velocity.y = -velocity.y * restitution;
        } else if (position.y - radius <= 0) {
            position.y = radius;
            velocity.y = -velocity.y * restitution;
        }
    }
};
```

### 3. 간단한 로프 시뮬레이션

```cpp
class RopeSegment {
public:
    Vector2 position;
    Vector2 oldPosition;
    bool isPinned;
    
    void Update(float deltaTime) {
        if (!isPinned) {
            Vector2 velocity = position - oldPosition;
            oldPosition = position;
            
            // 중력 적용
            position += velocity + Vector2{0, -980.0f} * deltaTime * deltaTime;
        }
    }
};

class Rope {
private:
    std::vector<RopeSegment> segments;
    float segmentLength;
    int constraintIterations = 3;

public:
    void Update(float deltaTime) {
        // 1. 물리 업데이트
        for (auto& segment : segments) {
            segment.Update(deltaTime);
        }
        
        // 2. 제약 조건 적용 (여러 번 반복으로 안정성 증가)
        for (int i = 0; i < constraintIterations; i++) {
            ApplyDistanceConstraints();
        }
    }
    
private:
    void ApplyDistanceConstraints() {
        for (int i = 0; i < segments.size() - 1; i++) {
            Vector2 delta = segments[i+1].position - segments[i].position;
            float distance = Length(delta);
            float difference = (segmentLength - distance) / distance;
            Vector2 correction = delta * difference * 0.5f;
            
            if (!segments[i].isPinned) {
                segments[i].position -= correction;
            }
            if (!segments[i+1].isPinned) {
                segments[i+1].position += correction;
            }
        }
    }
};
```

## 성능 최적화

### 1. 물리 업데이트 빈도 조절

```cpp
class PhysicsSystem {
private:
    float physicsTimeStep = 1.0f / 120.0f;  // 120Hz
    float accumulator = 0.0f;

public:
    void Update(float deltaTime) {
        accumulator += deltaTime;
        
        while (accumulator >= physicsTimeStep) {
            UpdatePhysics(physicsTimeStep);
            accumulator -= physicsTimeStep;
        }
        
        // 보간으로 부드러운 렌더링
        float alpha = accumulator / physicsTimeStep;
        InterpolateRenderPositions(alpha);
    }
};
```

### 2. 충돌 최적화

```cpp
// 공간 분할로 충돌 검사 횟수 줄이기
class SpatialGrid {
private:
    struct Cell {
        std::vector<GameObject*> objects;
    };
    
    std::vector<std::vector<Cell>> grid;
    float cellSize;

public:
    void UpdateCollisions() {
        ClearGrid();
        AddObjectsToGrid();
        
        // 같은 셀 내 객체들만 충돌 검사
        for (auto& row : grid) {
            for (auto& cell : row) {
                CheckCollisionsInCell(cell);
            }
        }
    }
};
```

### 3. 근사 계산 사용

```cpp
// 빠른 거리 계산 (제곱근 생략)
bool IsCloseEnough(Vector2 a, Vector2 b, float threshold) {
    float distanceSquared = (a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y);
    return distanceSquared < threshold * threshold;
}

// 빠른 정규화 (Newton-Raphson 근사)
Vector2 FastNormalize(Vector2 v) {
    float lengthSquared = v.x * v.x + v.y * v.y;
    if (lengthSquared > 0) {
        float invLength = 1.0f / sqrt(lengthSquared);  // 이 부분도 근사 가능
        return Vector2{v.x * invLength, v.y * invLength};
    }
    return {0, 0};
}
```

## 정리

2D 물리 프로그래밍의 핵심은:

### 기본 개념들
1. **운동학**: 위치, 속도, 가속도의 관계
2. **힘과 에너지**: 중력, 마찰, 탄성력
3. **충돌 처리**: 탄성/비탄성 충돌, 반발 계수
4. **제약 조건**: 거리 유지, 위치 고정

### 실무 활용
- **캐릭터 컨트롤**: 점프, 이동, 마찰
- **발사체 시뮬레이션**: 포물선 운동, 중력
- **충돌 게임**: 당구, 핀볼, 브릭 브레이커
- **로프/체인**: 거리 제약 조건

### 최적화 포인트
- **고정 시간 간격**: 안정적인 물리 시뮬레이션
- **공간 분할**: 효율적인 충돌 검사
- **근사 계산**: 성능 향상

이 2D 물리 기초를 이해하면 재미있고 현실적인 게임을 만들 수 있습니다!
