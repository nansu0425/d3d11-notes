# Windows Programming for DirectX 11 (DirectX 11을 위한 윈도우 프로그래밍)

## Windows Programming이란 무엇인가?

**Windows Programming**은 Microsoft Windows 운영체제에서 실행되는 애플리케이션을 만드는 프로그래밍입니다. DirectX 11을 사용하기 위해서는 반드시 Windows 프로그래밍의 기초를 이해해야 합니다.

DirectX 11은 Windows에서만 동작하며, 윈도우 창을 만들고 관리하는 것부터 시작됩니다. 게임이나 3D 애플리케이션도 결국은 Windows 애플리케이션의 한 종류이기 때문입니다.

쉽게 비유하면:
- **Windows Programming**: 집을 짓는 기초 공사
- **DirectX 11**: 집 안에 설치하는 고급 가전제품

아무리 좋은 DirectX 11 코드를 작성해도, Windows 기초가 없으면 실행할 창조차 만들 수 없습니다.

## 왜 Windows Programming이 필요할까?

### 문제상황: DirectX 11만으로는 불가능한 것들
DirectX 11은 강력한 그래픽 API이지만, 혼자서는 아무것도 할 수 없습니다:

```
DirectX 11만으로는:
❌ 윈도우 창을 만들 수 없음
❌ 사용자 입력을 받을 수 없음
❌ 메시지 처리를 할 수 없음
❌ 창 크기 변경을 감지할 수 없음
❌ 프로그램 종료를 처리할 수 없음
```

### 해결책: Windows API의 역할
Windows API가 제공하는 기능들:

```
Windows API로 가능한 것들:
✅ 윈도우 창 생성 및 관리
✅ 키보드, 마우스 입력 처리
✅ 시스템 메시지 처리
✅ 창 크기 변경 감지
✅ 프로그램 생명주기 관리
✅ DirectX와의 연동
```

## Windows API 기본 구조

Windows 프로그래밍은 **이벤트 기반(Event-Driven)** 시스템입니다. 프로그램이 계속 실행되면서 사용자의 행동(마우스 클릭, 키보드 입력 등)에 반응합니다.

### 전체 구조 개요
```
프로그램 시작
    ↓
윈도우 클래스 등록
    ↓
윈도우 창 생성
    ↓
DirectX 초기화
    ↓
┌─────────────────┐
│   메시지 루프    │ ← 계속 반복
│  (Message Loop)  │
└─────────────────┘
    ↓
정리 작업 및 종료
```

## 핵심 데이터 타입들

Windows API는 특별한 데이터 타입들을 사용합니다. 이들을 이해하는 것이 첫 번째 단계입니다.

### 1. 기본 타입들

#### 핸들(Handle) 타입
```cpp
HWND hwnd;          // 윈도우 핸들
HINSTANCE hInstance; // 인스턴스 핸들
HDC hdc;            // 디바이스 컨텍스트 핸들
HICON hIcon;        // 아이콘 핸들
HCURSOR hCursor;    // 커서 핸들
```

**핸들이란?**
- Windows가 관리하는 객체(윈도우, 아이콘 등)를 식별하는 "번호표"
- 실제 객체의 메모리 주소가 아닌 추상적인 식별자
- Windows가 내부적으로 관리하며, 우리는 이 번호로만 접근

#### 문자열 타입
```cpp
LPCWSTR lpszWindowName;  // 유니코드 문자열 (읽기 전용)
LPWSTR lpszBuffer;       // 유니코드 문자열 (쓰기 가능)
LPCSTR lpszAnsiString;   // ANSI 문자열 (읽기 전용)

// 실제 사용 예시
LPCWSTR windowTitle = L"My DirectX 11 Application";  // L 접두사 = 유니코드
```

#### 수치 타입
```cpp
DWORD dwStyle;          // 32비트 부호 없는 정수
UINT uMsg;              // 부호 없는 정수 (메시지 ID)
WPARAM wParam;          // 메시지 매개변수 1
LPARAM lParam;          // 메시지 매개변수 2
LRESULT lResult;        // 메시지 처리 결과
```

### 2. 구조체들

#### WNDCLASSEX (윈도우 클래스 정보)
```cpp
typedef struct {
    UINT cbSize;            // 구조체 크기
    UINT style;             // 윈도우 클래스 스타일
    WNDPROC lpfnWndProc;    // 윈도우 프로시저 함수 포인터
    int cbClsExtra;         // 추가 클래스 메모리
    int cbWndExtra;         // 추가 윈도우 메모리
    HINSTANCE hInstance;    // 인스턴스 핸들
    HICON hIcon;            // 아이콘
    HCURSOR hCursor;        // 커서
    HBRUSH hbrBackground;   // 배경 브러시
    LPCWSTR lpszMenuName;   // 메뉴 이름
    LPCWSTR lpszClassName;  // 클래스 이름
    HICON hIconSm;          // 작은 아이콘
} WNDCLASSEX;
```

#### MSG (메시지 정보)
```cpp
typedef struct {
    HWND hwnd;          // 메시지를 받을 윈도우
    UINT message;       // 메시지 ID
    WPARAM wParam;      // 추가 정보 1
    LPARAM lParam;      // 추가 정보 2
    DWORD time;         // 메시지 발생 시간
    POINT pt;           // 마우스 커서 위치
} MSG;
```

#### RECT (사각형 좌표)
```cpp
typedef struct {
    LONG left;      // 왼쪽 좌표
    LONG top;       // 위쪽 좌표
    LONG right;     // 오른쪽 좌표
    LONG bottom;    // 아래쪽 좌표
} RECT;

// 사용 예시
RECT clientRect;
GetClientRect(hwnd, &clientRect);  // 윈도우 클라이언트 영역 크기 가져오기
int width = clientRect.right - clientRect.left;
int height = clientRect.bottom - clientRect.top;
```

## 핵심 함수들

### 1. 윈도우 클래스 관련 함수들

#### RegisterClassEx (윈도우 클래스 등록)
```cpp
ATOM RegisterClassEx(const WNDCLASSEX* lpWndClass);
```

**역할:**
- 윈도우의 "설계도"를 Windows에 등록
- 어떤 아이콘, 커서, 배경색을 사용할지 정의
- 메시지 처리 함수 지정

**사용 예시:**
```cpp
WNDCLASSEX wc = {};
wc.cbSize = sizeof(WNDCLASSEX);
wc.style = CS_HREDRAW | CS_VREDRAW;     // 크기 변경 시 다시 그리기
wc.lpfnWndProc = WindowProc;            // 메시지 처리 함수
wc.hInstance = hInstance;
wc.hCursor = LoadCursor(NULL, IDC_ARROW);
wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1);
wc.lpszClassName = L"DirectX11WindowClass";

if (!RegisterClassEx(&wc)) {
    MessageBox(NULL, L"윈도우 클래스 등록 실패", L"오류", MB_OK);
    return -1;
}
```

### 2. 윈도우 생성 및 관리 함수들

#### CreateWindowEx (윈도우 생성)
```cpp
HWND CreateWindowEx(
    DWORD dwExStyle,        // 확장 스타일
    LPCWSTR lpClassName,    // 윈도우 클래스 이름
    LPCWSTR lpWindowName,   // 윈도우 제목
    DWORD dwStyle,          // 윈도우 스타일
    int X,                  // x 좌표
    int Y,                  // y 좌표
    int nWidth,             // 너비
    int nHeight,            // 높이
    HWND hWndParent,        // 부모 윈도우
    HMENU hMenu,            // 메뉴
    HINSTANCE hInstance,    // 인스턴스
    LPVOID lpParam          // 추가 매개변수
);
```

**실제 사용 예시:**
```cpp
HWND hwnd = CreateWindowEx(
    0,                              // 확장 스타일 없음
    L"DirectX11WindowClass",        // 등록한 클래스 이름
    L"DirectX 11 Application",      // 윈도우 제목
    WS_OVERLAPPEDWINDOW,            // 표준 윈도우 스타일
    CW_USEDEFAULT,                  // x 좌표 (기본값)
    CW_USEDEFAULT,                  // y 좌표 (기본값)
    800,                            // 너비
    600,                            // 높이
    NULL,                           // 부모 윈도우 없음
    NULL,                           // 메뉴 없음
    hInstance,                      // 애플리케이션 인스턴스
    NULL                            // 추가 매개변수 없음
);

if (!hwnd) {
    MessageBox(NULL, L"윈도우 생성 실패", L"오류", MB_OK);
    return -1;
}
```

#### 주요 윈도우 스타일들
```cpp
// 기본 윈도우 스타일들
WS_OVERLAPPEDWINDOW     // 표준 윈도우 (제목표시줄, 테두리, 최소화/최대화 버튼)
WS_POPUP                // 팝업 윈도우 (테두리 없음)
WS_CHILD                // 자식 윈도우
WS_VISIBLE              // 보이는 윈도우
WS_MAXIMIZE             // 최대화된 상태로 시작
WS_MINIMIZE             // 최소화된 상태로 시작

// 조합 사용 예시
DWORD windowStyle = WS_OVERLAPPEDWINDOW & ~(WS_THICKFRAME | WS_MAXIMIZEBOX);
// 크기 조절 불가능하고 최대화 버튼이 없는 윈도우
```

#### ShowWindow (윈도우 표시)
```cpp
BOOL ShowWindow(HWND hWnd, int nCmdShow);
```

**표시 옵션들:**
```cpp
SW_HIDE         // 윈도우 숨기기
SW_SHOW         // 윈도우 보이기
SW_MAXIMIZE     // 최대화
SW_MINIMIZE     // 최소화
SW_RESTORE      // 원래 크기로 복원

// 사용 예시
ShowWindow(hwnd, SW_SHOW);      // 윈도우 보이기
UpdateWindow(hwnd);             // 즉시 화면 업데이트
```

### 3. 메시지 처리 함수들

#### GetMessage (메시지 가져오기)
```cpp
BOOL GetMessage(
    LPMSG lpMsg,        // 메시지 구조체
    HWND hWnd,          // 메시지를 받을 윈도우 (NULL = 모든 윈도우)
    UINT wMsgFilterMin, // 최소 메시지 값
    UINT wMsgFilterMax  // 최대 메시지 값
);
```

**반환값:**
- `TRUE (1)`: 메시지를 성공적으로 가져옴
- `FALSE (0)`: WM_QUIT 메시지를 받음 (프로그램 종료)
- `-1`: 오류 발생

#### PeekMessage (메시지 확인하기)
```cpp
BOOL PeekMessage(
    LPMSG lpMsg,        // 메시지 구조체
    HWND hWnd,          // 윈도우 핸들
    UINT wMsgFilterMin, // 최소 메시지 값
    UINT wMsgFilterMax, // 최대 메시지 값
    UINT wRemoveMsg     // 메시지 제거 옵션
);
```

**GetMessage vs PeekMessage:**
```cpp
// GetMessage: 메시지가 올 때까지 대기 (블로킹)
while (GetMessage(&msg, NULL, 0, 0)) {
    TranslateMessage(&msg);
    DispatchMessage(&msg);
}

// PeekMessage: 메시지가 없어도 즉시 반환 (논블로킹)
while (true) {
    if (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE)) {
        if (msg.message == WM_QUIT) break;
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    
    // 메시지가 없을 때 다른 작업 수행 가능 (게임 루프 등)
    UpdateGame();
    RenderFrame();
}
```

#### TranslateMessage & DispatchMessage
```cpp
BOOL TranslateMessage(const MSG* lpMsg);    // 키보드 메시지 변환
LRESULT DispatchMessage(const MSG* lpMsg);  // 윈도우 프로시저로 메시지 전달
```

#### TranslateMessage의 상세 동작 원리

`TranslateMessage`는 키보드 입력 처리에서 매우 중요한 역할을 합니다. 가상 키 코드(Virtual Key Code)를 실제 문자 메시지로 변환하는 과정을 자세히 알아봅시다.

##### 1. 가상 키 코드 vs 문자 메시지
```cpp
// 사용자가 'A' 키를 눌렀을 때의 메시지 흐름

// 1단계: 하드웨어에서 발생하는 원시 메시지
WM_KEYDOWN: wParam = VK_A (가상 키 코드 0x41)

// 2단계: TranslateMessage 호출 후 추가로 생성되는 메시지
WM_CHAR: wParam = 'A' (실제 문자 코드 0x41)
// 또는 Shift+A인 경우
WM_CHAR: wParam = 'a' (소문자, 0x61)
```

##### 2. 메시지 변환 과정과 큐 처리
```cpp
// 메시지 루프에서의 실제 처리 순서
while (GetMessage(&msg, NULL, 0, 0)) {
    // 1. GetMessage로 WM_KEYDOWN 메시지 가져옴
    
    // 2. TranslateMessage 호출
    TranslateMessage(&msg);  // ← 여기서 WM_CHAR 메시지가 큐에 추가됨!
    
    // 3. 현재 메시지(WM_KEYDOWN) 처리
    DispatchMessage(&msg);
    
    // 4. 다음 GetMessage 호출 시 WM_CHAR 메시지가 반환됨
}
```

**중요한 점:** `TranslateMessage`는 새로운 메시지를 **메시지 큐에 추가**합니다. 즉시 처리하지 않습니다!

##### 3. 키보드 상태와 문자 변환
```cpp
// TranslateMessage가 고려하는 요소들
// 1. 현재 키보드 레이아웃 (한글, 영문 등)
// 2. Shift, Ctrl, Alt 키 상태
// 3. Caps Lock 상태
// 4. Num Lock 상태 (숫자 키패드)

// 예시: 'A' 키를 눌렀을 때
if (GetKeyState(VK_SHIFT) & 0x8000) {
    // Shift가 눌려있으면 → WM_CHAR: wParam = 'A' (대문자)
} else {
    // Shift가 안 눌려있으면 → WM_CHAR: wParam = 'a' (소문자)
}

// 특수 키들의 경우
VK_RETURN → WM_CHAR: wParam = 13 (캐리지 리턴)
VK_BACK   → WM_CHAR: wParam = 8  (백스페이스)
VK_TAB    → WM_CHAR: wParam = 9  (탭)
```

##### 4. 메시지 처리 순서 예시
```cpp
// 사용자가 'Hello'를 타이핑했을 때의 메시지 순서
// (Shift + H, e, l, l, o)

// 'H' 키 입력:
WM_KEYDOWN: wParam = VK_H
TranslateMessage() 호출 → WM_CHAR: wParam = 'H' 큐에 추가
DispatchMessage() → WindowProc에서 WM_KEYDOWN 처리
다음 GetMessage() → WM_CHAR: wParam = 'H' 받음
DispatchMessage() → WindowProc에서 WM_CHAR 처리

WM_KEYUP: wParam = VK_H
// TranslateMessage는 WM_KEYUP에서는 문자 메시지 생성 안 함

// 'e' 키 입력:
WM_KEYDOWN: wParam = VK_E
TranslateMessage() 호출 → WM_CHAR: wParam = 'e' 큐에 추가
// ... 반복
```

##### 5. 언제 TranslateMessage를 호출하지 말아야 할까?
```cpp
// 게임에서의 키 입력 처리 - 문자가 아닌 액션용
while (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE)) {
    if (msg.message == WM_KEYDOWN || msg.message == WM_KEYUP) {
        // 게임 액션용 키는 TranslateMessage 건너뛰기
        // WM_CHAR 메시지를 원하지 않는 경우
        DispatchMessage(&msg);
    } else {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
}

// 텍스트 입력이 필요한 경우만 TranslateMessage 사용
// 예: 채팅창, 이름 입력, 콘솔 명령어 등
```

#### DispatchMessage의 상세 동작 원리

##### 1. 메시지 라우팅 과정
```cpp
LRESULT result = DispatchMessage(&msg);

// 내부적으로 수행하는 작업:
// 1. msg.hwnd로 윈도우 식별
// 2. 해당 윈도우의 윈도우 프로시저 함수 포인터 찾기
// 3. 윈도우 프로시저 호출: WindowProc(hwnd, uMsg, wParam, lParam)
// 4. 반환값을 DispatchMessage의 반환값으로 전달
```

##### 2. 윈도우 프로시저에서의 메시지 처리
```cpp
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg) {
    case WM_KEYDOWN:
        // 가상 키 코드 처리 (게임 액션, 단축키 등)
        if (wParam == VK_SPACE) {
            // 스페이스바 액션
        }
        return 0;
        
    case WM_CHAR:
        // 문자 입력 처리 (텍스트 편집, 채팅 등)
        if (wParam >= 32 && wParam <= 126) {  // 출력 가능한 ASCII 문자
            AddCharacterToTextBuffer((char)wParam);
        }
        return 0;
        
    default:
        return DefWindowProc(hwnd, uMsg, wParam, lParam);
    }
}
```

##### 3. 메시지 처리 성능 고려사항
```cpp
// 효율적인 메시지 처리
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    // 자주 발생하는 메시지를 먼저 처리
    switch (uMsg) {
    case WM_MOUSEMOVE:    // 가장 자주 발생
        // 빠른 처리
        return 0;
        
    case WM_PAINT:        // 두 번째로 자주 발생
        // 렌더링 처리
        return 0;
        
    case WM_KEYDOWN:      // 사용자 입력
    case WM_CHAR:
        // 입력 처리
        return 0;
        
    case WM_CREATE:       // 한 번만 발생
    case WM_DESTROY:
        // 초기화/정리 작업
        return 0;
        
    default:
        return DefWindowProc(hwnd, uMsg, wParam, lParam);
    }
}
```

#### 실제 사용 예시: 텍스트 에디터

```cpp
// 간단한 텍스트 입력 처리 예시
std::wstring g_textBuffer;
int g_cursorPosition = 0;

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg) {
    case WM_KEYDOWN:
        // 특수 키 처리 (문자가 아닌 액션)
        switch (wParam) {
        case VK_LEFT:
            if (g_cursorPosition > 0) g_cursorPosition--;
            break;
        case VK_RIGHT:
            if (g_cursorPosition < g_textBuffer.length()) g_cursorPosition++;
            break;
        case VK_BACK:
            if (g_cursorPosition > 0) {
                g_textBuffer.erase(g_cursorPosition - 1, 1);
                g_cursorPosition--;
            }
            break;
        case VK_DELETE:
            if (g_cursorPosition < g_textBuffer.length()) {
                g_textBuffer.erase(g_cursorPosition, 1);
            }
            break;
        }
        InvalidateRect(hwnd, NULL, TRUE);  // 화면 다시 그리기 요청
        return 0;
        
    case WM_CHAR:
        // 실제 문자 입력 처리
        if (wParam >= 32) {  // 출력 가능한 문자만
            g_textBuffer.insert(g_cursorPosition, 1, (wchar_t)wParam);
            g_cursorPosition++;
            InvalidateRect(hwnd, NULL, TRUE);
        }
        return 0;
        
    // ... 다른 메시지 처리 ...
    }
    
    return DefWindowProc(hwnd, uMsg, wParam, lParam);
}
```

**핵심 정리:**
- `TranslateMessage`: 키보드 하드웨어 신호를 사용자가 의도한 문자로 변환
- `DispatchMessage`: 변환된 메시지를 적절한 윈도우 프로시저로 전달
- 메시지 처리 순서: WM_KEYDOWN → (TranslateMessage) → WM_CHAR 큐 추가 → 다음 루프에서 WM_CHAR 처리

## 윈도우 프로시저 (Window Procedure)

윈도우 프로시저는 Windows 메시지를 처리하는 **콜백 함수**입니다. Windows가 우리 프로그램에게 "이런 일이 일어났어요"라고 알려줄 때 호출됩니다.

### 윈도우 프로시저의 구조
```cpp
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg) {
    case WM_CREATE:
        // 윈도우가 생성될 때
        return 0;
        
    case WM_SIZE:
        // 윈도우 크기가 변경될 때
        return 0;
        
    case WM_PAINT:
        // 윈도우를 다시 그려야 할 때
        return 0;
        
    case WM_KEYDOWN:
        // 키가 눌렸을 때
        return 0;
        
    case WM_DESTROY:
        // 윈도우가 파괴될 때
        PostQuitMessage(0);
        return 0;
        
    default:
        // 처리하지 않는 메시지는 기본 처리
        return DefWindowProc(hwnd, uMsg, wParam, lParam);
    }
}
```

### 주요 메시지들

#### WM_CREATE (윈도우 생성)
```cpp
case WM_CREATE:
{
    // DirectX 초기화를 여기서 수행
    if (FAILED(InitializeDirectX(hwnd))) {
        MessageBox(hwnd, L"DirectX 초기화 실패", L"오류", MB_OK);
        return -1;  // 윈도우 생성 실패로 처리
    }
    
    // 타이머 설정 (60 FPS)
    SetTimer(hwnd, 1, 16, NULL);
    return 0;
}
```

#### WM_SIZE (윈도우 크기 변경)
```cpp
case WM_SIZE:
{
    int newWidth = LOWORD(lParam);   // 새로운 너비
    int newHeight = HIWORD(lParam);  // 새로운 높이
    
    // DirectX 백 버퍼 크기 조정
    if (g_swapChain && newWidth > 0 && newHeight > 0) {
        ResizeDirectXBuffers(newWidth, newHeight);
    }
    return 0;
}
```

#### WM_PAINT (화면 그리기)
```cpp
case WM_PAINT:
{
    PAINTSTRUCT ps;
    HDC hdc = BeginPaint(hwnd, &ps);
    
    // DirectX로 렌더링하는 경우, 여기서는 보통 아무것도 안 함
    // ValidateRect(hwnd, NULL);  // 그리기 요청 제거
    
    EndPaint(hwnd, &ps);
    return 0;
}
```

#### WM_KEYDOWN / WM_KEYUP (키보드 입력)
```cpp
case WM_KEYDOWN:
{
    switch (wParam) {
    case VK_ESCAPE:  // ESC 키
        PostMessage(hwnd, WM_CLOSE, 0, 0);
        break;
        
    case 'W':
    case 'w':
        g_input.moveForward = true;
        break;
        
    case VK_SPACE:   // 스페이스바
        g_input.jump = true;
        break;
    }
    return 0;
}

case WM_KEYUP:
{
    switch (wParam) {
    case 'W':
    case 'w':
        g_input.moveForward = false;
        break;
        
    case VK_SPACE:
        g_input.jump = false;
        break;
    }
    return 0;
}
```

#### WM_MOUSEMOVE / WM_LBUTTONDOWN (마우스 입력)
```cpp
case WM_MOUSEMOVE:
{
    int mouseX = LOWORD(lParam);
    int mouseY = HIWORD(lParam);
    
    // 마우스 델타 계산 (카메라 회전용)
    static int lastMouseX = mouseX;
    static int lastMouseY = mouseY;
    
    int deltaX = mouseX - lastMouseX;
    int deltaY = mouseY - lastMouseY;
    
    g_camera.RotateByMouse(deltaX, deltaY);
    
    lastMouseX = mouseX;
    lastMouseY = mouseY;
    return 0;
}

case WM_LBUTTONDOWN:
{
    int mouseX = LOWORD(lParam);
    int mouseY = HIWORD(lParam);
    
    // 마우스 캡처 (윈도우 밖으로 나가도 계속 추적)
    SetCapture(hwnd);
    g_input.leftMouseDown = true;
    return 0;
}

case WM_LBUTTONUP:
{
    // 마우스 캡처 해제
    ReleaseCapture();
    g_input.leftMouseDown = false;
    return 0;
}
```

#### WM_DESTROY (윈도우 파괴)
```cpp
case WM_DESTROY:
{
    // DirectX 정리
    CleanupDirectX();
    
    // 종료 메시지 게시
    PostQuitMessage(0);
    return 0;
}
```

## 메시지 루프 (Message Loop)

메시지 루프는 Windows 프로그램의 심장입니다. 계속해서 시스템 메시지를 확인하고 처리합니다.

### 1. 기본 메시지 루프 (일반 애플리케이션용)
```cpp
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, 
                   LPSTR lpCmdLine, int nCmdShow)
{
    // 윈도우 클래스 등록
    WNDCLASSEX wc = {};
    // ... 윈도우 클래스 설정 ...
    RegisterClassEx(&wc);
    
    // 윈도우 생성
    HWND hwnd = CreateWindowEx(/* ... */);
    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);
    
    // 메시지 루프
    MSG msg = {};
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    
    return (int)msg.wParam;
}
```

**특징:**
- 메시지가 올 때까지 대기 (블로킹)
- CPU 사용률이 낮음
- 실시간 렌더링에는 부적합

### 2. 게임용 메시지 루프 (DirectX 애플리케이션용)
```cpp
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, 
                   LPSTR lpCmdLine, int nCmdShow)
{
    // 윈도우 클래스 등록 및 생성
    // ... (동일) ...
    
    // DirectX 초기화
    if (FAILED(InitializeDirectX(hwnd))) {
        return -1;
    }
    
    // 게임 루프
    MSG msg = {};
    bool running = true;
    
    while (running) {
        // 메시지 처리 (논블로킹)
        while (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE)) {
            if (msg.message == WM_QUIT) {
                running = false;
                break;
            }
            
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
        
        if (running) {
            // 게임 로직 업데이트
            UpdateGame(GetDeltaTime());
            
            // 렌더링
            RenderFrame();
        }
    }
    
    // 정리
    CleanupDirectX();
    return (int)msg.wParam;
}
```

**특징:**
- 메시지가 없어도 계속 실행 (논블로킹)
- 실시간 렌더링 가능
- 높은 프레임률 유지 가능

### 3. 향상된 게임 루프 (시간 관리 포함)
```cpp
// 시간 관리 클래스
class Timer {
private:
    LARGE_INTEGER frequency;
    LARGE_INTEGER startTime;
    LARGE_INTEGER currentTime;
    LARGE_INTEGER lastTime;
    double deltaTime;
    
public:
    Timer() {
        QueryPerformanceFrequency(&frequency);
        QueryPerformanceCounter(&startTime);
        lastTime = startTime;
        deltaTime = 0.0;
    }
    
    void Update() {
        QueryPerformanceCounter(&currentTime);
        deltaTime = double(currentTime.QuadPart - lastTime.QuadPart) / 
                   double(frequency.QuadPart);
        lastTime = currentTime;
    }
    
    double GetDeltaTime() const { return deltaTime; }
    double GetTotalTime() const {
        return double(currentTime.QuadPart - startTime.QuadPart) / 
               double(frequency.QuadPart);
    }
};

// 메인 루프
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, 
                   LPSTR lpCmdLine, int nCmdShow)
{
    // ... 초기화 코드 ...
    
    Timer timer;
    MSG msg = {};
    bool running = true;
    
    while (running) {
        // 시간 업데이트
        timer.Update();
        double deltaTime = timer.GetDeltaTime();
        
        // 메시지 처리
        while (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE)) {
            if (msg.message == WM_QUIT) {
                running = false;
                break;
            }
            
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
        
        if (running) {
            // 프레임률 제한 (선택적)
            static double frameTime = 1.0 / 60.0;  // 60 FPS
            static double accumulator = 0.0;
            
            accumulator += deltaTime;
            
            // 고정 시간 간격으로 게임 업데이트
            while (accumulator >= frameTime) {
                UpdateGame(frameTime);
                accumulator -= frameTime;
            }
            
            // 렌더링은 가능한 한 자주
            RenderFrame();
        }
    }
    
    CleanupDirectX();
    return (int)msg.wParam;
}
```

## DirectX와의 연동

Windows 프로그래밍과 DirectX 11을 연결하는 방법을 알아봅시다.

### 1. SwapChain 생성 시 HWND 전달
```cpp
HRESULT InitializeDirectX(HWND hwnd)
{
    // Device와 SwapChain 생성
    DXGI_SWAP_CHAIN_DESC swapChainDesc = {};
    swapChainDesc.BufferCount = 1;
    swapChainDesc.BufferDesc.Width = 800;
    swapChainDesc.BufferDesc.Height = 600;
    swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
    swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
    swapChainDesc.OutputWindow = hwnd;  // ← Windows 윈도우와 연결!
    swapChainDesc.SampleDesc.Count = 1;
    swapChainDesc.Windowed = TRUE;
    
    HRESULT hr = D3D11CreateDeviceAndSwapChain(
        nullptr,
        D3D_DRIVER_TYPE_HARDWARE,
        nullptr,
        0,
        nullptr,
        0,
        D3D11_SDK_VERSION,
        &swapChainDesc,
        &g_swapChain,
        &g_device,
        nullptr,
        &g_deviceContext
    );
    
    return hr;
}
```

### 2. 윈도우 크기 변경 처리
```cpp
// WM_SIZE 메시지 처리에서 호출
void ResizeDirectXBuffers(int newWidth, int newHeight)
{
    if (!g_swapChain) return;
    
    // 기존 렌더 타겟 뷰 해제
    g_deviceContext->OMSetRenderTargets(0, nullptr, nullptr);
    if (g_renderTargetView) {
        g_renderTargetView->Release();
        g_renderTargetView = nullptr;
    }
    
    // 백 버퍼 크기 조정
    HRESULT hr = g_swapChain->ResizeBuffers(
        1,                              // 버퍼 개수
        newWidth,                       // 새로운 너비
        newHeight,                      // 새로운 높이
        DXGI_FORMAT_R8G8B8A8_UNORM,     // 포맷
        0                               // 플래그
    );
    
    if (FAILED(hr)) {
        MessageBox(NULL, L"백 버퍼 크기 조정 실패", L"오류", MB_OK);
        return;
    }
    
    // 새로운 렌더 타겟 뷰 생성
    ID3D11Texture2D* backBuffer;
    hr = g_swapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), 
                               reinterpret_cast<void**>(&backBuffer));
    
    if (SUCCEEDED(hr)) {
        hr = g_device->CreateRenderTargetView(backBuffer, nullptr, &g_renderTargetView);
        backBuffer->Release();
        
        if (SUCCEEDED(hr)) {
            // 뷰포트 설정
            D3D11_VIEWPORT viewport = {};
            viewport.Width = (float)newWidth;
            viewport.Height = (float)newHeight;
            viewport.MinDepth = 0.0f;
            viewport.MaxDepth = 1.0f;
            g_deviceContext->RSSetViewports(1, &viewport);
        }
    }
}
```

### 3. 입력 처리와 게임 로직 연동
```cpp
// 전역 입력 상태
struct InputState {
    bool keys[256];         // 키보드 상태
    int mouseX, mouseY;     // 마우스 위치
    bool leftMouseButton;   // 마우스 왼쪽 버튼
    bool rightMouseButton;  // 마우스 오른쪽 버튼
} g_input = {};

// 윈도우 프로시저에서 입력 상태 업데이트
case WM_KEYDOWN:
    if (wParam < 256) {
        g_input.keys[wParam] = true;
    }
    return 0;

case WM_KEYUP:
    if (wParam < 256) {
        g_input.keys[wParam] = false;
    }
    return 0;

case WM_MOUSEMOVE:
    g_input.mouseX = LOWORD(lParam);
    g_input.mouseY = HIWORD(lParam);
    return 0;

// 게임 로직에서 입력 상태 사용
void UpdateGame(double deltaTime)
{
    // 키보드 입력 처리
    if (g_input.keys['W']) {
        g_camera.MoveForward(deltaTime);
    }
    if (g_input.keys['S']) {
        g_camera.MoveBackward(deltaTime);
    }
    if (g_input.keys['A']) {
        g_camera.MoveLeft(deltaTime);
    }
    if (g_input.keys['D']) {
        g_camera.MoveRight(deltaTime);
    }
    
    // 마우스 입력 처리
    static int lastMouseX = g_input.mouseX;
    static int lastMouseY = g_input.mouseY;
    
    int deltaX = g_input.mouseX - lastMouseX;
    int deltaY = g_input.mouseY - lastMouseY;
    
    if (g_input.leftMouseButton) {
        g_camera.Rotate(deltaX * 0.01f, deltaY * 0.01f);
    }
    
    lastMouseX = g_input.mouseX;
    lastMouseY = g_input.mouseY;
    
    // 게임 객체 업데이트
    g_gameWorld.Update(deltaTime);
}
```

## 전체 예제 코드

DirectX 11을 위한 완전한 Windows 프로그래밍 예제입니다:

```cpp
#include <windows.h>
#include <d3d11.h>
#include <d3dcompiler.h>
#include <directxmath.h>

#pragma comment(lib, "d3d11.lib")
#pragma comment(lib, "d3dcompiler.lib")

// DirectX 전역 변수들
ID3D11Device* g_device = nullptr;
ID3D11DeviceContext* g_deviceContext = nullptr;
IDXGISwapChain* g_swapChain = nullptr;
ID3D11RenderTargetView* g_renderTargetView = nullptr;

// 윈도우 프로시저 선언
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);

// DirectX 초기화 함수
HRESULT InitializeDirectX(HWND hwnd)
{
    // SwapChain 설정
    DXGI_SWAP_CHAIN_DESC swapChainDesc = {};
    swapChainDesc.BufferCount = 1;
    swapChainDesc.BufferDesc.Width = 800;
    swapChainDesc.BufferDesc.Height = 600;
    swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
    swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
    swapChainDesc.OutputWindow = hwnd;
    swapChainDesc.SampleDesc.Count = 1;
    swapChainDesc.Windowed = TRUE;
    swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
    
    // Device와 SwapChain 생성
    HRESULT hr = D3D11CreateDeviceAndSwapChain(
        nullptr,
        D3D_DRIVER_TYPE_HARDWARE,
        nullptr,
        0,
        nullptr,
        0,
        D3D11_SDK_VERSION,
        &swapChainDesc,
        &g_swapChain,
        &g_device,
        nullptr,
        &g_deviceContext
    );
    
    if (FAILED(hr)) return hr;
    
    // 백 버퍼에서 렌더 타겟 뷰 생성
    ID3D11Texture2D* backBuffer;
    hr = g_swapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), 
                               reinterpret_cast<void**>(&backBuffer));
    
    if (FAILED(hr)) return hr;
    
    hr = g_device->CreateRenderTargetView(backBuffer, nullptr, &g_renderTargetView);
    backBuffer->Release();
    
    if (FAILED(hr)) return hr;
    
    // 렌더 타겟 설정
    g_deviceContext->OMSetRenderTargets(1, &g_renderTargetView, nullptr);
    
    // 뷰포트 설정
    D3D11_VIEWPORT viewport = {};
    viewport.Width = 800.0f;
    viewport.Height = 600.0f;
    viewport.MinDepth = 0.0f;
    viewport.MaxDepth = 1.0f;
    g_deviceContext->RSSetViewports(1, &viewport);
    
    return S_OK;
}

// 렌더링 함수
void Render()
{
    // 화면 클리어 (파란색)
    float clearColor[4] = { 0.0f, 0.2f, 0.4f, 1.0f };
    g_deviceContext->ClearRenderTargetView(g_renderTargetView, clearColor);
    
    // 여기에 실제 렌더링 코드 추가
    // ...
    
    // 화면에 출력
    g_swapChain->Present(1, 0);
}

// DirectX 정리 함수
void CleanupDirectX()
{
    if (g_renderTargetView) {
        g_renderTargetView->Release();
        g_renderTargetView = nullptr;
    }
    
    if (g_swapChain) {
        g_swapChain->Release();
        g_swapChain = nullptr;
    }
    
    if (g_deviceContext) {
        g_deviceContext->Release();
        g_deviceContext = nullptr;
    }
    
    if (g_device) {
        g_device->Release();
        g_device = nullptr;
    }
}

// 윈도우 프로시저
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg) {
    case WM_CREATE:
        // DirectX 초기화
        if (FAILED(InitializeDirectX(hwnd))) {
            MessageBox(hwnd, L"DirectX 초기화 실패", L"오류", MB_OK);
            return -1;
        }
        return 0;
        
    case WM_SIZE:
        // 윈도우 크기 변경 처리
        if (g_swapChain && LOWORD(lParam) > 0 && HIWORD(lParam) > 0) {
            // 여기에 백 버퍼 크기 조정 코드 추가
        }
        return 0;
        
    case WM_PAINT:
        // 화면 그리기
        Render();
        ValidateRect(hwnd, NULL);
        return 0;
        
    case WM_KEYDOWN:
        if (wParam == VK_ESCAPE) {
            PostMessage(hwnd, WM_CLOSE, 0, 0);
        }
        return 0;
        
    case WM_DESTROY:
        CleanupDirectX();
        PostQuitMessage(0);
        return 0;
        
    default:
        return DefWindowProc(hwnd, uMsg, wParam, lParam);
    }
}

// 메인 함수
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, 
                   LPSTR lpCmdLine, int nCmdShow)
{
    // 윈도우 클래스 등록
    WNDCLASSEX wc = {};
    wc.cbSize = sizeof(WNDCLASSEX);
    wc.style = CS_HREDRAW | CS_VREDRAW;
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.hCursor = LoadCursor(NULL, IDC_ARROW);
    wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1);
    wc.lpszClassName = L"DirectX11WindowClass";
    
    if (!RegisterClassEx(&wc)) {
        MessageBox(NULL, L"윈도우 클래스 등록 실패", L"오류", MB_OK);
        return -1;
    }
    
    // 윈도우 생성
    HWND hwnd = CreateWindowEx(
        0,
        L"DirectX11WindowClass",
        L"DirectX 11 Application",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT,
        CW_USEDEFAULT,
        800,
        600,
        NULL,
        NULL,
        hInstance,
        NULL
    );
    
    if (!hwnd) {
        MessageBox(NULL, L"윈도우 생성 실패", L"오류", MB_OK);
        return -1;
    }
    
    // 윈도우 표시
    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);
    
    // 메시지 루프 (게임용)
    MSG msg = {};
    bool running = true;
    
    while (running) {
        // 메시지 처리
        while (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE)) {
            if (msg.message == WM_QUIT) {
                running = false;
                break;
            }
            
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
        
        if (running) {
            // 게임 로직 업데이트
            // UpdateGame();
            
            // 렌더링
            Render();
        }
    }
    
    return (int)msg.wParam;
}
```

## 정리

Windows 프로그래밍은 DirectX 11 학습의 필수 기초입니다. 핵심 개념들을 다시 한번 정리하면:

### 필수 개념들
1. **윈도우 클래스**: 윈도우의 "설계도" 등록
2. **윈도우 생성**: 실제 윈도우 창 만들기
3. **메시지 루프**: 시스템 메시지 처리
4. **윈도우 프로시저**: 메시지별 처리 로직
5. **DirectX 연동**: HWND를 통한 연결

### 실무에서 중요한 부분들
- **메시지 루프 선택**: 실시간 애플리케이션은 PeekMessage 사용
- **윈도우 크기 변경 처리**: 백 버퍼 크기 조정 필수
- **입력 처리**: 키보드/마우스 상태를 게임 로직과 연동
- **리소스 관리**: 프로그램 종료 시 모든 DirectX 리소스 해제

이 기초를 탄탄히 하면 DirectX 11의 고급 기능들을 훨씬 쉽게 이해할 수 있습니다!
