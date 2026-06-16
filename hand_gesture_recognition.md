# 손동작 인식 프로그래밍
## OpenCV, MediaPipe로 만드는 공중 제어 인터페이스

> 이 교재는 비전 프로세싱을 처음 다뤄본 입문자도 끝까지 따라올 수 있도록 설계하였다.  
> 최종 목표는 양손 동작 인식 구현 코드의 모든 줄을 스스로 이해하고 설명할 수 있는 수준이다.

---

## 목차

1. [준비하기 — 설치 및 환경 설정](#1장-준비하기--설치-및-환경-설정)
2. [카메라 열기 — OpenCV 기초](#2장-카메라-열기--opencv-기초)
3. [이미지의 정체 — 숫자로 이루어진 그림](#3장-이미지의-정체--숫자로-이루어진-그림)
4. [FPS란 무엇인가 — 초당 프레임 수 측정](#4장-fps란-무엇인가--초당-프레임-수-측정)
5. [MediaPipe 연결하기 — AI 손 추적의 시작](#5장-mediapipe-연결하기--ai-손-추적의-시작)
6. [손 랜드마크 이해하기 — 21개의 점](#6장-손-랜드마크-이해하기--21개의-점)
7. [좌표 변환 — 정규화 좌표를 픽셀로](#7장-좌표-변환--정규화-좌표를-픽셀로)
8. [스무딩 — 손 떨림 없애기](#8장-스무딩--손-떨림-없애기)
9. [양손으로 오브젝트 제어하기 — Pan/Zoom/Rotate](#9장-양손으로-오브젝트-제어하기--panzoomrotate)
10. [손 방향과 플릭 제스처 인식](#10장-손-방향과-플릭-제스처-인식)
11. [플랫폼 최적화 — Windows와 라즈베리파이5](#11장-플랫폼-최적화--windows와-라즈베리파이5)
12. [양손 동작 인식 구현](#12장-양손-동작-인식-구현)
- [부록 A. MediaPipe 랜드마크 심화](#부록-a-mediapipe-랜드마크-심화)
- [부록 B. 수학 공식 정리](#부록-b-수학-공식-정리)

---

## 1장. 준비하기 — 설치 및 환경 설정

### 1.1 필요한 도구

이 교재에서 사용하는 도구는 세 가지이다.

| 도구 | 역할 |
|------|------|
| **Python 3.12.x** | 프로그래밍 언어 |
| **OpenCV** | 카메라 영상을 읽고 화면에 그리는 라이브러리 |
| **** | Google이 만든 AI 손 인식 라이브러리 |

MediaPipe 관련 자세히 설명은 다음 링크를 참조한다.

```html
https://developers.google.com/edge/mediapipe/solutions/guide
```

### 1.2 설치

터미널(명령 프롬프트)을 열고 아래 명령을 실행한다.

```sh
pip install mediapipe
pip install opencv-python
```

### 1.3 AI 모델

손 동작 인식에 필요한 모델을 다운로드한다.

```sh
curl https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task -o hand_landmarker.task
```
> **왜 파일이 필요할까?**  
> AI 모델은 수백만 장의 손 사진으로 미리 학습된 거대한 수식 묶음이다. 이 파일 하나에 손의 모든 모양 패턴이 들어 있다. 프로그램이 실행될 때 이 파일을 메모리에 올려서 사용한다.

---

## 2장. 카메라 열기 — OpenCV 기초

### 2.1 첫 번째 카메라 프리뷰

아래 코드를 `step1.py`로 저장하고 실행해 보자.

```python
import cv2

cap = cv2.VideoCapture(0)  # 0번 = 기본 카메라

while cap.isOpened():
    ret, frame = cap.read()   # 프레임 한 장 읽기
    if not ret:
        break

    cv2.imshow("Camera", frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):  # q를 누르면 종료
        break

cap.release()
cv2.destroyAllWindows()
```

실행하면 카메라 화면이 나타난다. `q` 키를 누르면 종료된다.

### 2.2 코드 한 줄씩 이해하기

```python
cap = cv2.VideoCapture(0)
```
카메라를 열어 `cap`이라는 변수에 저장한다. 숫자 `0`은 컴퓨터에 연결된 첫 번째 카메라를 의미한다. USB 카메라가 두 개 있으면 `1`을 쓰기도 한다.

```python
ret, frame = cap.read()
```
카메라에서 이미지 한 장(프레임)을 읽는다. `ret`은 성공 여부(`True`/`False`)이고, `frame`은 실제 이미지 데이터이다.

```python
cv2.imshow("Camera", frame)
```
`"Camera"`라는 제목의 창에 `frame`을 표시한다.

```python
cv2.waitKey(1) & 0xFF == ord('q')
```
1밀리초 동안 키 입력을 기다린다. `q` 키가 눌리면 루프를 탈출한다.

### 2.3 좌우 반전 — 거울 모드

카메라를 켜면 움직임이 좌우 반대로 보여 어색하다. 거울처럼 만들려면 아래 한 줄을 추가한다.

```python
frame = cv2.flip(frame, 1)  # 1 = 좌우 반전
```

> **왜 반전이 필요할까?**  
> 카메라는 렌즈를 통해 빛을 그대로 기록하기 때문에, 오른손을 들면 화면의 왼쪽에 손이 나타난다. 사람은 거울을 보듯 자신의 움직임과 화면이 일치하는 것이 훨씬 자연스럽다. `cv2.flip(frame, 1)`은 이미지를 좌우로 뒤집는다.

---

## 3장. 이미지의 정체 — 숫자로 이루어진 그림

### 3.1 픽셀(Pixel)이란

컴퓨터에서 이미지는 사실 **숫자 배열**이다. 640×480 해상도의 이미지는 가로 640개, 세로 480개의 작은 점(픽셀)으로 이루어져 있다. 각 픽셀은 세 가지 색의 밝기를 0~255 사이의 숫자 세 개로 표현한다.

### 3.2 BGR 색상 순서

OpenCV는 일반적인 RGB(빨강-초록-파랑) 순서와 반대인 **BGR(파랑-초록-빨강)** 순서를 사용한다.

| 배열 위치 | 의미 |
|-----------|------|
| `[0]` | Blue (파랑) |
| `[1]` | Green (초록) |
| `[2]` | Red (빨강) |

예를 들어, 순수한 빨간 픽셀의 값은 `[0, 0, 255]`이다.

### 3.3 frame.shape

```python
h, w = frame.shape[:2]
```

`frame.shape`는 `(세로높이, 가로너비, 채널수)` 형태의 숫자 세 개를 반환한다. 채널 수는 BGR이므로 항상 3이다.

```python
frame.shape  # -> (480, 640, 3)
```

> **최적화 포인트**  
> `frame.shape[:2]`는 루프가 실행될 때마다 이미지 크기를 새로 확인한다. 그러나 카메라 해상도는 실행 중에 바뀌지 않는다. 따라서 루프 시작 전에 한 번만 확인하고 변수에 저장하면 루프마다 반복되는 확인 연산을 없앨 수 있다.
> ```python
> # 루프 밖에서 한 번만
> h, w = cam_h, cam_w
>
> while True:
>     ret, frame = cap.read()
>     # h, w = frame.shape[:2]  ← 이 줄이 필요 없어진다
> ```

---

## 4장. FPS란 무엇인가 — 초당 프레임 수 측정

### 4.1 FPS 개념

**FPS(Frames Per Second)**는 1초 동안 화면이 몇 번 갱신되는지를 나타내는 수치이다. 게임에서 60FPS는 1초에 60장의 그림을 보여준다는 의미이다.

| FPS | 체감 |
|-----|------|
| 60 이상 | 매우 부드러움 |
| 30 | 보통 |
| 15 이하 | 끊겨 보임 |

### 4.2 FPS 측정 방법

```python
import time

prev_time = time.time()

while True:
    # ... 프레임 처리 ...

    now = time.time()
    elapsed = now - prev_time   # 이전 프레임과 현재 프레임 사이의 시간(초)
    prev_time = now

    fps = 1.0 / elapsed         # 1초 ÷ 걸린 시간 = 초당 프레임 수
```

### 4.3 EMA로 FPS 부드럽게 표시하기

측정된 FPS는 프레임마다 크게 흔들린다. **지수 이동 평균(EMA, Exponential Moving Average)**을 적용하면 숫자가 부드럽게 변한다.

$$\text{FPS}_{\text{표시}} = \text{FPS}_{\text{표시}} \times 0.9 + \text{FPS}_{\text{현재}} \times 0.1$$

이 공식은 "표시 FPS의 90%를 유지하면서 현재 FPS의 10%씩 반영한다"는 의미이다. 숫자가 갑자기 튀지 않고 천천히 수렴한다.

```python
fps_display = 0.0

# 루프 안에서
if elapsed > 0:
    fps_display = fps_display * 0.9 + (1.0 / elapsed) * 0.1

cv2.putText(frame, f"FPS: {fps_display:.1f}", (500, 30),
            cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 255), 2)
```

> **최적화 포인트**  
> `time.time()`은 운영체제에 현재 시간을 물어보는 작업이다. 매 프레임마다 FPS용으로 한 번, 타임스탬프용으로 또 한 번 호출하면 쓸데없는 비용이 발생한다. `now = time.time()`으로 한 번만 호출한 뒤, FPS 계산과 타임스탬프 모두 `now` 변수를 공유하면 된다.

---

## 5장. MediaPipe 연결하기 — AI 손 추적의 시작

### 5.1 MediaPipe Tasks API

MediaPipe에는 구식 API(`mp.solutions`)와 신식 Tasks API(`mp.tasks`) 두 가지가 있다. 이 교재는 더 정확하고 빠른 **Tasks API**를 사용한다.

### 5.2 실행 모드 세 가지

MediaPipe HandLandmarker는 세 가지 실행 모드를 지원한다.

| 모드 | 특징 | 언제 사용하나 |
|------|------|---------------|
| `IMAGE` | 이미지 한 장 처리 | 정지 이미지 분석 |
| `VIDEO` | 영상 프레임 순서대로 처리 | **실시간 카메라 (추천)** |
| `LIVE_STREAM` | 비동기 처리 (콜백 방식) | 복잡한 멀티스레드 구조 |

이 교재에서는 **VIDEO 모드**를 사용한다. VIDEO 모드는 프레임을 순서대로 처리하기 때문에 이전 프레임의 추적 정보를 활용할 수 있어 속도가 더 빠르다.

> **VIDEO 모드의 핵심 원리**  
> 손을 인식하는 과정은 두 단계로 나뉜다.  
> 1단계 (팜 디텍션): 이미지 전체에서 손의 위치를 찾는다. 이 작업은 무겁다.  
> 2단계 (랜드마크): 찾은 손의 영역 안에서만 21개 관절을 찾는다. 이 작업은 가볍다.  
> VIDEO 모드에서는 이전 프레임에서 손을 찾은 위치를 기억해두고, 다음 프레임에서는 그 근방만 추적한다. 손을 잃어버렸을 때만 1단계를 다시 실행하므로 전체 속도가 크게 향상된다.

### 5.3 기본 연결 코드

```python
import cv2
import mediapipe as mp
import time

BaseOptions = mp.tasks.BaseOptions
HandLandmarker = mp.tasks.vision.HandLandmarker
HandLandmarkerOptions = mp.tasks.vision.HandLandmarkerOptions
VisionRunningMode = mp.tasks.vision.RunningMode

options = HandLandmarkerOptions(
    base_options=BaseOptions(model_asset_path='hand_landmarker.task'),
    running_mode=VisionRunningMode.VIDEO,
    num_hands=2
)

cap = cv2.VideoCapture(0)
start_time = time.time()

with HandLandmarker.create_from_options(options) as landmarker:
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret: break

        frame = cv2.flip(frame, 1)

        # BGR -> RGB 변환 (MediaPipe는 RGB 형식을 요구한다)
        rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)

        # MediaPipe 이미지 객체로 변환
        mp_image = mp.Image(image_format=mp.ImageFormat.SRGB, data=rgb_frame)

        # 타임스탬프 계산 (VIDEO 모드 필수)
        ts_ms = int((time.time() - start_time) * 1000)

        # 추론 실행
        result = landmarker.detect_for_video(mp_image, ts_ms)

        print(result)  # 결과 확인

        cv2.imshow("Test", frame)
        if cv2.waitKey(1) & 0xFF == ord('q'): break

cap.release()
cv2.destroyAllWindows()
```

### 5.4 왜 BGR을 RGB로 변환하나

OpenCV는 역사적인 이유로 색상을 BGR 순서로 저장한다. 반면 MediaPipe를 포함한 대부분의 AI 라이브러리는 RGB 순서를 기대한다. 두 순서를 혼용하면 파랑과 빨강이 뒤바뀐 이상한 색상으로 손을 인식하게 된다.

```python
rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
```

> **최적화 포인트 — `writeable=False`**  
> MediaPipe는 이미지 데이터를 내부에서 처리할 때 원본 배열을 보호하기 위해 복사본을 만든다. `rgb_frame.flags.writeable = False`로 배열을 읽기 전용으로 표시하면, MediaPipe가 "복사 없이 직접 읽어도 된다"고 판단하여 불필요한 메모리 복사를 건너뛴다.
> ```python
> rgb_frame.flags.writeable = False
> mp_image = mp.Image(image_format=mp.ImageFormat.SRGB, data=rgb_frame)
> # rgb_frame은 이후 사용하지 않으므로 writeable=True 복원도 필요 없다
> ```

### 5.5 신뢰도(Confidence) 파라미터

```python
options = HandLandmarkerOptions(
    ...
    min_hand_detection_confidence=0.3,   # 손 탐지 최소 신뢰도
    min_hand_presence_confidence=0.3,    # 손 존재 최소 신뢰도
    min_tracking_confidence=0.3          # 추적 최소 신뢰도
)
```

각 파라미터의 기본값은 `0.5`이다. 값을 낮추면(0.3) 모션 블러처럼 손이 흐리게 찍혀도 놓치지 않게 되지만, 손이 없는데도 손으로 잘못 인식하는 오탐지(False Positive)가 늘어날 수 있다.

`min_tracking_confidence`를 낮게 설정하면 추적이 성공했다고 판단하는 기준이 낮아져, 무거운 팜 디텍션 단계를 더 자주 건너뛴다. 이는 속도 향상에 도움이 된다.

---

## 6장. 손 랜드마크 이해하기 — 21개의 점

### 6.1 랜드마크란

MediaPipe가 손을 인식하면 손의 21개 관절 위치를 좌표로 반환한다. 이 21개의 점을 **랜드마크(Landmark)**라고 한다.

```
        8   12  16  20
        |   |   |   |
    4   7   11  15  19
    |   |   |   |   |
    3   6   10  14  18
    |   |   |   |   |
    2   5   9   13  17
     \  |   |   |  /
      1  \  |  /  
       \  \ | /  
        \   0   
```

| 인덱스 | 이름 | 설명 |
|--------|------|------|
| 0 | Wrist | 손목 |
| 1~4 | Thumb | 엄지 (CMC -> MCP -> IP -> Tip) |
| 5~8 | Index | 검지 |
| 9~12 | Middle | 중지 |
| 13~16 | Ring | 약지 |
| 17~20 | Pinky | 새끼 |

> **핵심 인덱스 암기법**  
> - 각 손가락의 끝(Tip)은 4, 8, 12, 16, 20 — 4의 배수이다.  
> - 각 손가락의 MCP(손등 쪽 첫 관절)는 1, 5, 9, 13, 17이다.  
> - 9번(중지 MCP)은 손바닥의 정중앙에 가까워 손 전체의 대표 좌표로 자주 쓰인다.

### 6.2 정규화 좌표

랜드마크의 `x`, `y` 값은 0.0~1.0 사이의 **정규화된 좌표**이다. 이미지의 크기와 관계없이 위치를 비율로 표현한다.

$$x = 0.0 \Rightarrow \text{화면 왼쪽 끝}, \quad x = 1.0 \Rightarrow \text{화면 오른쪽 끝}$$
$$y = 0.0 \Rightarrow \text{화면 위쪽 끝}, \quad y = 1.0 \Rightarrow \text{화면 아래쪽 끝}$$

좌우 반전(`cv2.flip`)을 하면 좌표도 자동으로 반전된다. 따라서 반전 후에 추론하면 왼손, 오른손 레이블도 사용자 기준으로 올바르게 나온다.

### 6.3 결과 데이터 구조

```python
result = landmarker.detect_for_video(mp_image, ts_ms)

# 감지된 손의 수
num_hands = len(result.hand_landmarks)  # 0, 1, 또는 2

# 첫 번째 손의 9번 랜드마크 접근
lm = result.hand_landmarks[0][9]
print(lm.x, lm.y, lm.z)

# 어느 손인지 확인 (미러 이후 'Left' = 사용자 왼손)
label = result.handedness[0][0].category_name  # 'Left' 또는 'Right'
```

### 6.4 랜드마크를 점으로 그리기

```python
import cv2
import mediapipe as mp
import time

# ... (5장의 options, cap, landmarker 설정과 동일) ...

with HandLandmarker.create_from_options(options) as landmarker:
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret: break
        frame = cv2.flip(frame, 1)
        h, w = frame.shape[:2]

        ts_ms = int((time.time() - start_time) * 1000)
        rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        rgb_frame.flags.writeable = False
        mp_image = mp.Image(image_format=mp.ImageFormat.SRGB, data=rgb_frame)
        result = landmarker.detect_for_video(mp_image, ts_ms)

        for hand in result.hand_landmarks:
            for lm in hand:
                # 정규화 좌표 -> 픽셀 좌표 변환
                cx = int(lm.x * w)
                cy = int(lm.y * h)
                cv2.circle(frame, (cx, cy), 5, (0, 255, 0), -1)

        cv2.imshow("Landmarks", frame)
        if cv2.waitKey(1) & 0xFF == ord('q'): break
```

실행하면 손의 모든 관절에 초록 점이 나타난다.

---

## 7장. 좌표 변환 — 정규화 좌표를 픽셀로

### 7.1 픽셀 복원 공식

MediaPipe가 반환하는 좌표는 0.0~1.0 사이의 비율이다. 화면에 점을 그리려면 실제 픽셀 위치로 변환해야 한다.

$$\text{픽셀 X} = \text{int}(\text{lm.x} \times \text{화면 너비})$$
$$\text{픽셀 Y} = \text{int}(\text{lm.y} \times \text{화면 높이})$$

```python
h, w = frame.shape[:2]

# 손의 9번 랜드마크 픽셀 좌표
cx = int(hand[9].x * w)
cy = int(hand[9].y * h)
```

### 7.2 두 점 사이의 거리 — 유클리드 거리

손가락이 접혔는지, 양손이 가까운지 판별하려면 두 점 사이의 거리를 계산해야 한다.

$$d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$

파이썬에서는 `math.hypot(dx, dy)`를 사용하면 더 간결하고 빠르게 계산할 수 있다.

```python
import math

# 두 점 사이 거리 계산
dist = math.hypot(x2 - x1, y2 - y1)
```

> **`math.hypot` vs `math.sqrt`**  
> `math.sqrt((dx)**2 + (dy)**2)`와 `math.hypot(dx, dy)`는 같은 결과를 반환한다. 그러나 `math.hypot`는 C 언어 레벨에서 최적화되어 있어 더 빠르다. 또한 제곱 연산(`**2`)을 명시적으로 쓰지 않아도 되므로 코드가 간결해진다.

### 7.3 각도 계산 — atan2

두 손의 상대적인 방향(각도)을 구하려면 `math.atan2`를 사용한다.

$$\theta = \text{atan2}(\Delta y, \Delta x)$$

`atan2`는 `(0, 0)`을 기준으로 어떤 방향에 있는지를 -180°~+180° 범위의 각도로 반환한다.

```python
import math

# 손1에서 손2를 바라보는 방향 각도
angle = math.degrees(
    math.atan2(y2 - y1, x2 - x1)
)
```

> **왜 `atan`이 아닌 `atan2`인가?**  
> `atan(dy/dx)`는 분모 `dx`가 0일 때 오류가 발생하고, 반환 범위가 -90°~+90°로 방향을 구분하지 못하는 경우가 있다. `atan2(dy, dx)`는 분모가 0인 경우를 자동으로 처리하고 -180°~+180° 전체 방향을 올바르게 반환한다.

---

## 8장. 스무딩 — 손 떨림 없애기

### 8.1 왜 손이 떨리나

카메라로 추적한 손의 좌표는 매 프레임마다 조금씩 다르게 측정된다. 이는 카메라 노이즈, 조명 변화, AI 모델의 미세한 불확실성 때문이다. 이 원시 좌표(Raw Coordinate)를 그대로 쓰면 오브젝트가 심하게 떨린다.

### 8.2 지수 이동 평균 (EMA) 스무딩

**EMA(Exponential Moving Average)**는 과거 값을 유지하면서 새 값을 조금씩 반영하는 방법이다.

$$s_{\text{다음}} = s_{\text{현재}} + (\text{raw} - s_{\text{현재}}) \times \alpha$$

- $\alpha = 0.0$: 전혀 움직이지 않는다.  
- $\alpha = 1.0$: 즉시 원시 좌표로 이동한다.  
- $\alpha = 0.3$: 부드럽게 추종한다.

```python
smooth_x = 320.0  # 초기값

# 루프 안에서
raw_x = int(hand[9].x * w)
alpha = 0.3
smooth_x += (raw_x - smooth_x) * alpha
```

### 8.3 속도 기반 가변 스무딩

고정 $\alpha$ 방식은 한 가지 문제가 있다. $\alpha$가 작으면(느린 추종) 손이 정지했을 때는 좋지만, 손을 빠르게 움직일 때는 좌표가 뒤처져 반응이 느리다. $\alpha$가 크면 반대로 정지 시 떨림이 크다.

해결책은 **손의 이동 속도에 따라 $\alpha$를 동적으로 바꾸는 것**이다.

```
손이 빠를 때  -> alpha 크게 -> 빠르게 추종
손이 느릴 때  -> alpha 작게 -> 부드럽게 유지
```

$$\text{speed} = \text{hypot}(\text{raw}_x - s_x, \; \text{raw}_y - s_y)$$
$$\alpha = \text{clip}(0.12 + \frac{\text{speed}}{25.0}, \; 0.12, \; 0.95)$$

```python
speed = math.hypot(raw_x - smooth_x, raw_y - smooth_y)

# 2.5픽셀 이하의 미세 노이즈는 '정지'로 취급 (데드존)
if speed < 2.5:
    speed = 0.0

alpha = min(max(0.12 + speed / 25.0, 0.12), 0.95)

smooth_x += (raw_x - smooth_x) * alpha
smooth_y += (raw_y - smooth_y) * alpha
```

> **데드존(Dead Zone)이란?**  
> 2.5픽셀 이하의 아주 작은 움직임은 사람이 손을 정지시켰을 때도 카메라 노이즈로 발생한다. 이 범위를 0으로 강제하면 완전 정지 시 오브젝트가 전혀 흔들리지 않는다.

> **최적화 포인트 — `min/max` vs `np.clip`**  
> `np.clip(0.12 + speed/25.0, 0.12, 0.95)`와 `min(max(0.12 + speed/25.0, 0.12), 0.95)`는 같은 결과이다. 그러나 숫자 하나(스칼라)를 처리할 때는 numpy 함수가 배열 처리를 위한 내부 초기화 과정을 거치므로 순수 파이썬의 `min/max`가 더 빠르다.

---

## 9장. 양손으로 오브젝트 제어하기 — Pan/Zoom/Rotate

### 9.1 아이디어

두 손의 위치 관계를 이용하면 세 가지 변환을 동시에 제어할 수 있다.

| 동작 | 계산 방법 | 제어 |
|------|-----------|------|
| 이동(Pan) | 두 손의 중점 | 오브젝트가 중점을 따라 이동 |
| 확대(Zoom) | 두 손 사이의 거리 | 거리 비율로 크기 변경 |
| 회전(Rotation) | 두 손을 잇는 선의 각도 | 각도 변화량만큼 회전 |

### 9.2 이동 (Pan)

두 손의 중점이 오브젝트의 목표 위치가 된다.

```python
mid_x = (smooth_h1[0] + smooth_h2[0]) / 2
mid_y = (smooth_h1[1] + smooth_h2[1]) / 2

# 스무딩을 적용하면서 중점을 따라감
smooth_center[0] += (mid_x - smooth_center[0]) * box_alpha
smooth_center[1] += (mid_y - smooth_center[1]) * box_alpha
```

### 9.3 확대/축소 (Zoom)

두 손의 거리 변화 비율을 오브젝트의 크기에 적용한다. "기준 거리"를 처음 양손이 잡힌 순간 저장해 두고, 현재 거리와의 비율로 배율을 계산한다.

```python
current_dist = math.hypot(smooth_h1[0] - smooth_h2[0],
                          smooth_h1[1] - smooth_h2[1])

# 처음 잡힌 순간의 거리를 기준으로 저장
if base_hand_dist is None:
    base_hand_dist = current_dist
    base_box_size = smooth_size
else:
    scale_factor = current_dist / (base_hand_dist + 1e-6)
    # +1e-6: 분모가 0이 되는 것을 방지 (base_hand_dist가 0이면 오류 발생)
    target_size = min(max(base_box_size * scale_factor, 20), 200)
    smooth_size += (target_size - smooth_size) * box_alpha
```

> **`+ 1e-6`의 역할**  
> 수학에서 분모가 0인 나눗셈은 무한대가 되어 오류를 일으킨다. `1e-6`은 `0.000001`을 의미한다. 분모에 이 극소값을 더하면 값이 거의 바뀌지 않으면서도 0으로 나누는 오류를 완전히 막는다.

### 9.4 회전 (Rotation)

두 손을 잇는 선의 각도 변화량만큼 오브젝트를 회전시킨다.

```python
current_angle = math.degrees(
    math.atan2(smooth_h2[1] - smooth_h1[1], smooth_h2[0] - smooth_h1[0])
)

if base_hand_angle is None:
    base_hand_angle = current_angle
else:
    target_angle = current_angle - base_hand_angle
    # 각도 차이를 -180 ~ +180 범위로 정규화
    angle_diff = (target_angle - smooth_angle + 180) % 360 - 180
    smooth_angle += angle_diff * box_alpha
```

> **각도 정규화: `(diff + 180) % 360 - 180`**  
> 각도가 179°에서 -179°로 갑자기 넘어가면 차이가 358°가 계산된다. 그러면 오브젝트가 한 방향으로 거의 한 바퀴를 도는 것처럼 보인다. 실제로는 2° 회전했을 뿐인데 말이다. 이 공식은 어떤 각도 차이든 항상 -180°~+180° 범위 안으로 변환하여 최단 경로로 회전하도록 보장한다.

### 9.5 회전 사각형 그리기

OpenCV의 `cv2.boxPoints`와 `cv2.drawContours`를 사용하면 기울어진 사각형을 그릴 수 있다.

```python
import numpy as np

# ((중심 x, 중심 y), (너비, 높이), 회전 각도)
rect = ((smooth_center[0], smooth_center[1]),
        (smooth_size * 2, smooth_size * 2),
        smooth_angle)

box_points = cv2.boxPoints(rect)      # 사각형의 네 꼭짓점 좌표 계산
box_points = np.int32(box_points)     # 정수형으로 변환
cv2.drawContours(frame, [box_points], 0, (0, 255, 255), 3)
```

---

## 10장. 손 방향과 플릭 제스처 인식

### 10.1 손 방향 판별

손이 수평인지 수직인지를 판별하려면 손목(0번)에서 중지 MCP(9번)으로 향하는 벡터의 각도를 계산한다.

```
수직 손: 0번 -> 9번이 위아래 방향  (angle ≈ 90°)
수평 손: 0번 -> 9번이 좌우 방향    (angle ≈ 0° 또는 180°)
```

$$\theta = \left| \text{atan2}\left(\Delta y_{0 \to 9},\; \Delta x_{0 \to 9}\right) \right|$$

```python
def hand_orientation(hand, w, h):
    angle = abs(math.degrees(math.atan2(
        (hand[9].y - hand[0].y) * h,
        (hand[9].x - hand[0].x) * w
    )))
    if angle < 35 or angle > 145:
        return 'H'   # 수평 (Horizontal)
    if 55 < angle < 125:
        return 'V'   # 수직 (Vertical)
    return 'U'       # 불명확 (Undefined)
```

| 각도 범위 | 손 방향 |
|-----------|---------|
| 0°~35°, 145°~180° | 수평 `'H'` |
| 55°~125° | 수직 `'V'` |
| 35°~55°, 125°~145° | 불명확 `'U'` |

정규화 좌표에 픽셀 크기(`w`, `h`)를 곱하는 이유는 화면이 정사각형이 아니기 때문이다. 640×480처럼 가로가 긴 화면에서 정규화 좌표 그대로 각도를 계산하면 실제 방향과 차이가 생긴다.

### 10.2 플릭(Flick) 제스처란

**플릭**은 손가락이나 손을 빠르게 튕기는 동작이다. 속도(velocity) 기반으로 감지한다.

$$\text{velocity}_x = \text{raw}_x(\text{현재}) - \text{raw}_x(\text{이전 프레임})$$

일정 임계값(`FLICK_VEL_THRESHOLD`) 이상의 속도가 감지되면 플릭으로 판정한다.

### 10.3 플릭 제스처 감지 로직

**모드 1: 왼손 수평 + 오른손 수직**
- 오른손이 빠르게 **좌측**으로 움직이면 -> `<< FLICK LEFT`
- 오른손이 빠르게 **우측**으로 움직이면 -> `FLICK RIGHT >>`

**모드 2: 왼손 수직 + 오른손 수평**
- 오른손이 빠르게 **위**로 움직이면 -> `FLICK UP`
- 오른손이 빠르게 **아래**로 움직이면 -> `FLICK DOWN`

```python
FLICK_VEL_THRESHOLD = 22.0   # 플릭으로 인정하는 최소 속도 (픽셀/프레임)
FLICK_COOLDOWN = 20           # 플릭 후 무시 구간 (프레임)

flick_cooldown = 0
right_raw_prev = None

# 루프 안에서
if right_raw_prev is not None and flick_cooldown == 0:
    rvx = right_raw_cx - right_raw_prev[0]   # X 방향 속도
    rvy = right_raw_cy - right_raw_prev[1]   # Y 방향 속도

    left_ori  = hand_orientation(left_hand,  w, h)
    right_ori = hand_orientation(right_hand, w, h)

    # 모드 1: 왼손 수평 + 오른손 수직 -> 좌우 플릭
    if left_ori == 'H' and right_ori == 'V':
        if abs(rvx) > FLICK_VEL_THRESHOLD and abs(rvx) > abs(rvy) * 1.5:
            gesture = "<< FLICK LEFT" if rvx < 0 else "FLICK RIGHT >>"

    # 모드 2: 왼손 수직 + 오른손 수평 -> 상하 플릭
    elif left_ori == 'V' and right_ori == 'H':
        if abs(rvy) > FLICK_VEL_THRESHOLD and abs(rvy) > abs(rvx) * 1.5:
            gesture = "FLICK UP" if rvy < 0 else "FLICK DOWN"

right_raw_prev = (right_raw_cx, right_raw_cy)
flick_cooldown = max(0, flick_cooldown - 1)
```

> **쿨다운(Cooldown)이 필요한 이유**  
> 사람이 손을 한 번 튕기면 손이 관성으로 계속 움직이다가 멈춘다. 이 과정에서 여러 프레임에 걸쳐 속도 임계값을 넘을 수 있다. 쿨다운은 한 번 플릭이 감지된 후 일정 프레임 동안 추가 감지를 무시하여 한 번의 동작이 여러 번 이벤트로 발생하는 것을 막는다.

> **`abs(rvx) > abs(rvy) * 1.5`의 의미**  
> X 방향 속도가 Y 방향 속도의 1.5배 이상일 때만 좌우 플릭으로 인정한다. 이 조건이 없으면 대각선으로 손을 움직여도 좌우 플릭이 발동된다. 방향의 '순수성'을 요구하는 것이다.

---

## 11장. 플랫폼 최적화 — Windows와 라즈베리파이5

### 11.1 카메라 백엔드

OpenCV는 카메라를 열 때 운영체제에 따라 다른 방법(백엔드)을 사용할 수 있다.

| 플랫폼 | 백엔드 | 코드 |
|--------|--------|------|
| Windows | DirectShow | `cv2.VideoCapture(0, cv2.CAP_DSHOW)` |
| Linux/macOS | 기본 (V4L2 등) | `cv2.VideoCapture(0)` |

`cv2.CAP_DSHOW`(DirectShow)는 Windows 전용이다. Linux(라즈베리파이)에서 사용하면 오류가 발생한다.

```python
import sys

if sys.platform == "win32":
    cap = cv2.VideoCapture(0, cv2.CAP_DSHOW)
else:
    cap = cv2.VideoCapture(0)
```

> **`sys.platform`이란?**  
> 현재 운영체제를 문자열로 반환한다. Windows는 `"win32"`, macOS는 `"darwin"`, Linux는 `"linux"`이다. 이를 이용하면 같은 코드가 여러 운영체제에서 올바르게 동작하도록 분기할 수 있다.

### 11.2 해상도 제한

라즈베리파이5는 데스크탑 PC보다 처리 능력이 낮다. 카메라 해상도를 640×480으로 제한하면 MediaPipe가 처리해야 하는 픽셀 수가 줄어 속도가 향상된다.

```python
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
```

MediaPipe는 내부적으로 192×192 해상도로 리사이즈하여 처리한다. 따라서 입력 해상도가 1280×720이든 640×480이든 AI 추론 결과는 거의 동일하다. 반면 `cv2.cvtColor` 등의 전처리 비용은 픽셀 수에 비례하므로 해상도를 낮추는 것이 유리하다.

> **왜 직접 256×256으로 줄이면 안 될까?**  
> `cv2.resize`로 입력을 256×256으로 먼저 줄이면 `cvtColor` 비용을 아낄 수 있을 것처럼 보인다. 그러나 640×480(4:3)을 256×256(1:1)으로 줄이면 손이 납작하게 왜곡된다. 왜곡된 이미지에서는 팜 디텍션 정확도가 떨어진다. 또한 추론 직전에 MediaPipe가 어차피 내부적으로 192×192로 리사이즈하므로 `cv2.resize` 비용만 추가되고 실질적인 이득이 없다.

### 11.3 MJPG 포맷 (Linux 전용)

USB 카메라는 기본적으로 YUYV(비압축) 포맷으로 영상을 전송한다. 이 포맷은 USB 대역폭을 많이 사용한다. MJPG(Motion JPEG) 포맷으로 전환하면 카메라가 JPEG으로 압축한 데이터를 보내므로 전송량이 크게 줄어든다.

```python
# Linux에서만 적용
if sys.platform != "win32":
    cap.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc(*'MJPG'))
    cap.set(cv2.CAP_PROP_FPS, 30)
```

> **주의: Windows에서 MJPG 설정을 하면 초기화 시간이 2초에서 5초로 늘어난다.**  
> Windows의 DirectShow 백엔드는 `CAP_PROP_FOURCC`를 설정할 때 카메라와 포맷을 재협상(re-negotiation)하는 과정이 발생한다. 이 과정이 수초간 블로킹을 일으킨다. Linux의 V4L2 백엔드는 이런 문제가 없으므로 플랫폼을 구분해서 설정해야 한다.

### 11.4 최적화 효과 종합

| 최적화 항목 | 방법 | 효과 |
|-------------|------|------|
| 루프 밖 좌표 계산 | `h, w = cam_h, cam_w` | `frame.shape` 반복 호출 제거 |
| `time.time()` 통합 | `now` 변수 공유 | syscall 1회 감소 |
| `writeable=False` | 배열 읽기 전용 표시 | MediaPipe 내부 복사 방지 |
| `math.hypot` | `sqrt(dx²+dy²)` 대체 | C 레벨 최적화 내장 |
| `min/max` | `np.clip` 스칼라 대체 | numpy 오버헤드 제거 |
| 해상도 640×480 | `CAP_PROP_FRAME_WIDTH/HEIGHT` | 전처리 비용 절감 |
| MJPG 포맷 (Linux) | `CAP_PROP_FOURCC` | USB 전송량 감소 |

---

## 12장. 양손 동작 인식 구현

이 장에서는 최종 구현인 양손 동작 인식 코드의 모든 줄을 위에서 배운 내용과 연결하여 설명한다.

```python
import sys
import cv2
import mediapipe as mp
import math
import os
import numpy as np
import time

BaseOptions = mp.tasks.BaseOptions
HandLandmarker = mp.tasks.vision.HandLandmarker
HandLandmarkerOptions = mp.tasks.vision.HandLandmarkerOptions
VisionRunningMode = mp.tasks.vision.RunningMode

def main():
    current_dir = os.path.dirname(os.path.abspath(__file__))
    model_path = os.path.join(current_dir, 'hand_landmarker.task')

    # 모델의 검출 및 추적 최소 신뢰도 기준을 낮춰 모션 블러 상황에서도 손을 놓치지 않게 합니다.
    options = HandLandmarkerOptions(
        base_options=BaseOptions(
            model_asset_path=model_path,
            delegate=mp.tasks.BaseOptions.Delegate.CPU
        ),
        running_mode=VisionRunningMode.VIDEO,  # 프레임 순차 처리가 동기식으로 보장되는 VIDEO 모드
        num_hands=2,
        min_hand_detection_confidence=0.3,
        min_hand_presence_confidence=0.3,
        min_tracking_confidence=0.3 # 무거운 재탐색 루프를 억제하여 스피드 향상 (기본값은 0.5)
    )

    print("[SYSTEM] 카메라 초기화 중...")
    # CAP_DSHOW는 Windows 전용 — Linux(RPi5)에서는 기본 백엔드 사용
    if sys.platform == "win32":
        cap = cv2.VideoCapture(0, cv2.CAP_DSHOW)
    else:
        cap = cv2.VideoCapture(0)
    if not cap.isOpened():
        print("[ERROR] 카메라를 열 수 없습니다.")
        return

    # 640x480으로 제한해 추론 부하를 줄임
    cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
    cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
    # MJPG/FPS 설정은 Linux(RPi5)에서만 적용 — Windows+CAP_DSHOW에서는 재협상으로 수초 지연 발생
    if sys.platform != "win32":
        cap.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc(*'MJPG'))
        cap.set(cv2.CAP_PROP_FPS, 30)

    cam_w = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
    cam_h = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
    print(f"[SYSTEM] 캡처 해상도: {cam_w}x{cam_h}")

    # 오브젝트 초기 상태
    base_hand_dist = None
    base_hand_angle = None
    base_box_size = 80

    # 가변 스무딩을 위한 변수
    smooth_h1, smooth_h2 = None, None
    smooth_center = [cam_w / 2, cam_h / 2]
    smooth_size = 80.0
    smooth_angle = 0.0

    # h, w는 루프 안에서 변하지 않으므로 루프 밖에서 한 번만 계산
    h, w = cam_h, cam_w

    start_time = time.time()
    prev_frame_time = start_time
    fps_display = 0.0

    print("[SYSTEM] AI 모델 로드 중 (VIDEO 모드 순차 동기화 엄격 적용)...")
    with HandLandmarker.create_from_options(options) as landmarker:
        print("[SYSTEM] 미디어파이프 양손 제어 시스템 실행.")

        while cap.isOpened():
            ret, frame = cap.read()
            if not ret: break

            frame = cv2.flip(frame, 1)

            # time.time()을 루프 당 1회만 호출 — 타임스탬프와 FPS 계산에 공유
            now = time.time()
            elapsed = now - prev_frame_time
            prev_frame_time = now
            if elapsed > 0:
                fps_display = fps_display * 0.9 + (1.0 / elapsed) * 0.1

            # VIDEO 모드용 타임스탬프 계산
            frame_timestamp_ms = int((now - start_time) * 1000)

            rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            
            # writeable=False: MediaPipe 내부 불필요한 배열 복사 방지
            rgb_frame.flags.writeable = False
            mp_image = mp.Image(image_format=mp.ImageFormat.SRGB, data=rgb_frame)
            # writeable 복원 불필요 — rgb_frame은 이후 미사용
            
            # 현재 프레임의 추론 결과를 동기식으로 유실 없이 리턴 받음
            detection_result = landmarker.detect_for_video(mp_image, frame_timestamp_ms)

            # 양손이 정상 검출된 경우
            landmarks = detection_result.hand_landmarks
            if len(landmarks) == 2:
                hand1 = landmarks[0]
                hand2 = landmarks[1]
                
                # 각 손 객체의 9번 인덱스 랜드마크에 직접 접근
                h1_cx, h1_cy = int(hand1[9].x * w), int(hand1[9].y * h)
                h2_cx, h2_cy = int(hand2[9].x * w), int(hand2[9].y * h)

                # 속도 기반 가변 스무딩 알고리즘 적용
                if smooth_h1 is None or smooth_h2 is None:
                    smooth_h1, smooth_h2 = [float(h1_cx), float(h1_cy)], [float(h2_cx), float(h2_cy)]
                    alpha1, alpha2 = 0.5, 0.5  # [수정 완료] 첫 진입 시 변수 바인딩 에러 예방
                else:
                    # 이전 프레임 대비 손의 실시간 이동 속도(거리) 계산
                    speed_h1 = math.hypot(h1_cx - smooth_h1[0], h1_cy - smooth_h1[1])
                    speed_h2 = math.hypot(h2_cx - smooth_h2[0], h2_cy - smooth_h2[1])
                    
                    # 정지 상태 정밀도 향상: 2.5 픽셀 이하의 미세 노이즈 진동은 완전히 차단 (데드존)
                    if speed_h1 < 2.5: speed_h1 = 0.0
                    if speed_h2 < 2.5: speed_h2 = 0.0

                    # 가변 스무딩 계수 적용 (최소 0.12로 무겁게 고정, 빠르면 0.95로 즉각 추종)
                    alpha1 = min(max(0.12 + speed_h1 / 25.0, 0.12), 0.95)
                    alpha2 = min(max(0.12 + speed_h2 / 25.0, 0.12), 0.95)

                    smooth_h1[0] += (h1_cx - smooth_h1[0]) * alpha1
                    smooth_h1[1] += (h1_cy - smooth_h1[1]) * alpha1
                    smooth_h2[0] += (h2_cx - smooth_h2[0]) * alpha2
                    smooth_h2[1] += (h2_cy - smooth_h2[1]) * alpha2

                # 그래픽 출력 (보정된 손 좌표 반영)
                sh1x, sh1y = int(smooth_h1[0]), int(smooth_h1[1])
                sh2x, sh2y = int(smooth_h2[0]), int(smooth_h2[1])
                cv2.circle(frame, (sh1x, sh1y), 10, (255, 0, 0), -1)
                cv2.circle(frame, (sh2x, sh2y), 10, (0, 0, 255), -1)
                cv2.line(frame, (sh1x, sh1y), (sh2x, sh2y), (255, 255, 255), 2)

                box_alpha = (alpha1 + alpha2) / 2

                # 이동(Pan)
                mid_x = (smooth_h1[0] + smooth_h2[0]) / 2
                mid_y = (smooth_h1[1] + smooth_h2[1]) / 2
                smooth_center[0] += (mid_x - smooth_center[0]) * box_alpha
                smooth_center[1] += (mid_y - smooth_center[1]) * box_alpha

                # 확대(Zoom) 및 회전(Rotation)
                current_hand_dist = math.hypot(smooth_h1[0] - smooth_h2[0], smooth_h1[1] - smooth_h2[1])
                current_hand_angle = math.degrees(math.atan2(smooth_h2[1] - smooth_h1[1], smooth_h2[0] - smooth_h1[0]))

                if base_hand_dist is None or base_hand_angle is None:
                    base_hand_dist = current_hand_dist
                    base_hand_angle = current_hand_angle
                    base_box_size = smooth_size
                else:
                    scale_factor = current_hand_dist / (base_hand_dist + 1e-6)
                    target_size = min(max(base_box_size * scale_factor, 20), 200)
                    smooth_size += (target_size - smooth_size) * box_alpha
                    
                    target_angle = current_hand_angle - base_hand_angle
                    angle_diff = (target_angle - smooth_angle + 180) % 360 - 180
                    smooth_angle += angle_diff * box_alpha

                cv2.putText(frame, "SPATIAL CONTROL ACTIVE", (30, 50),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)
            else:
                # 손을 완전히 치웠을 때 초기화
                base_hand_dist = None
                base_hand_angle = None
                smooth_h1, smooth_h2 = None, None

            # 최종 보정된 상태값으로 회전 사각형 그리기
            rect = ((smooth_center[0], smooth_center[1]), (smooth_size * 2, smooth_size * 2), smooth_angle)
            box_points = cv2.boxPoints(rect)
            box_points = np.int32(box_points)
            cv2.drawContours(frame, [box_points], 0, (0, 255, 255), 3)

            # FPS OSD 표시 (계산은 루프 상단에서 수행)
            cv2.putText(frame, f"FPS: {fps_display:.1f}", (cam_w - 130, 30),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 255), 2)

            cv2.imshow("Minority Report - Spatial UI", frame)
            if cv2.waitKey(1) & 0xFF == ord('q'):
                break

    cap.release()
    cv2.destroyAllWindows()

if __name__ == "__main__":
    main()
```


### 12.1 임포트 및 설정

```python
import sys      # 운영체제 감지 (sys.platform)
import cv2      # 카메라, 이미지 처리, 화면 출력
import mediapipe as mp  # AI 손 인식
import math     # hypot, atan2, degrees (수학 함수)
import os       # 파일 경로 처리
import numpy as np  # boxPoints, int32 변환
import time     # 타임스탬프, FPS 계산

BaseOptions = mp.tasks.BaseOptions
HandLandmarker = mp.tasks.vision.HandLandmarker
HandLandmarkerOptions = mp.tasks.vision.HandLandmarkerOptions
VisionRunningMode = mp.tasks.vision.RunningMode
```

`mp.tasks.vision.HandLandmarker`처럼 긴 경로를 변수에 저장해두면 코드가 짧아지고 오타도 줄어든다.

### 12.2 `hand_orientation` 함수

```python
def hand_orientation(hand, w, h):
    angle = abs(math.degrees(math.atan2(
        (hand[9].y - hand[0].y) * h,
        (hand[9].x - hand[0].x) * w
    )))
    if angle < 35 or angle > 145:
        return 'H'
    if 55 < angle < 125:
        return 'V'
    return 'U'
```

손목(0번)에서 중지 MCP(9번)를 향하는 벡터의 각도를 계산하여 손이 수평(`'H'`)인지 수직(`'V'`)인지 반환한다. 곱하기 `w`, `h`는 정규화 좌표를 픽셀 스케일로 변환하여 가로 세로 비율을 올바르게 반영하기 위해 필요하다.

### 12.3 모델 경로 및 옵션 설정

```python
current_dir = os.path.dirname(os.path.abspath(__file__))
model_path = os.path.join(current_dir, 'hand_landmarker.task')
```

`__file__`은 현재 실행 중인 파이썬 파일의 경로이다. `os.path.dirname`으로 폴더 경로를 얻고, `os.path.join`으로 모델 파일 경로를 완성한다. 이 방식은 어느 폴더에서 실행해도 항상 올바른 경로를 찾는다.

```python
options = HandLandmarkerOptions(
    base_options=BaseOptions(
        model_asset_path=model_path,
        delegate=mp.tasks.BaseOptions.Delegate.CPU  # GPU 없이 CPU로 실행
    ),
    running_mode=VisionRunningMode.VIDEO,   # 5장에서 배운 VIDEO 모드
    num_hands=2,                            # 최대 2개 손 추적
    min_hand_detection_confidence=0.3,      # 탐지 임계값 낮춤 (모션 블러 대응)
    min_hand_presence_confidence=0.3,
    min_tracking_confidence=0.3             # 추적 성공 기준 낮춤 (속도 향상)
)
```

### 12.4 카메라 초기화

```python
if sys.platform == "win32":
    cap = cv2.VideoCapture(0, cv2.CAP_DSHOW)
else:
    cap = cv2.VideoCapture(0)
if not cap.isOpened():
    print("[ERROR] 카메라를 열 수 없습니다.")
    return
```

플랫폼에 따라 백엔드를 다르게 설정한다. 카메라를 열지 못하면 즉시 종료하여 이후 코드에서 발생할 오류를 미리 막는다.

```python
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
if sys.platform != "win32":
    cap.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc(*'MJPG'))
    cap.set(cv2.CAP_PROP_FPS, 30)
```

해상도를 640×480으로 제한하고, Linux에서만 MJPG 포맷을 설정한다.

### 12.5 상태 변수 초기화

```python
base_hand_dist = None     # 줌 기준 거리 (양손이 처음 잡힌 순간 저장)
base_hand_angle = None    # 회전 기준 각도
base_box_size = 80        # 줌 기준 크기

smooth_h1, smooth_h2 = None, None       # 스무딩된 손 좌표 (첫 프레임 전 None)
smooth_center = [cam_w / 2, cam_h / 2] # 오브젝트 중심 (화면 중앙으로 초기화)
smooth_size = 80.0                      # 오브젝트 크기
smooth_angle = 0.0                      # 오브젝트 회전 각도

FLICK_COOLDOWN = 20
FLICK_VEL_THRESHOLD = 22.0
GESTURE_DISPLAY_FRAMES = 50
flick_cooldown = 0
gesture_timer = 0
last_gesture = ""
right_raw_prev = None

h, w = cam_h, cam_w  # 최적화: 루프 밖에서 한 번만 계산
```

`smooth_center`의 초기값이 하드코딩된 `[320, 240]`이 아닌 `[cam_w / 2, cam_h / 2]`인 이유는 실제 카메라 해상도가 640×480이 아닐 수도 있기 때문이다. 카메라가 실제로 반환한 해상도를 기준으로 초기화하면 어떤 카메라에서도 오브젝트가 화면 중앙에서 시작한다.

### 12.6 메인 루프 — 프레임 처리

```python
while cap.isOpened():
    ret, frame = cap.read()
    if not ret: break

    frame = cv2.flip(frame, 1)  # 거울 모드

    # 최적화: time.time()을 한 번만 호출하여 FPS와 타임스탬프에 공유
    now = time.time()
    elapsed = now - prev_frame_time
    prev_frame_time = now
    if elapsed > 0:
        fps_display = fps_display * 0.9 + (1.0 / elapsed) * 0.1  # EMA FPS

    frame_timestamp_ms = int((now - start_time) * 1000)  # VIDEO 모드용 타임스탬프
```

`now = time.time()`은 루프 전체에서 딱 한 번 호출된다. 이 값을 FPS 계산(`elapsed = now - prev_frame_time`)과 타임스탬프 계산(`now - start_time`) 모두에 재사용한다.

```python
    rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    rgb_frame.flags.writeable = False
    mp_image = mp.Image(image_format=mp.ImageFormat.SRGB, data=rgb_frame)
    detection_result = landmarker.detect_for_video(mp_image, frame_timestamp_ms)
```

BGR -> RGB 변환 후 읽기 전용으로 표시하여 MediaPipe에 전달한다. `detect_for_video`는 결과를 동기적으로 반환한다. `rgb_frame`은 이후 사용하지 않으므로 `writeable=True`로 되돌릴 필요가 없다.

### 12.7 양손 감지 및 스무딩

```python
    landmarks = detection_result.hand_landmarks  # 최적화: 속성 1회만 접근
    if len(landmarks) == 2:
        hand1 = landmarks[0]
        hand2 = landmarks[1]

        h1_cx, h1_cy = int(hand1[9].x * w), int(hand1[9].y * h)
        h2_cx, h2_cy = int(hand2[9].x * w), int(hand2[9].y * h)
```

`detection_result.hand_landmarks`를 `landmarks` 변수에 캐싱한다. 이후 `landmarks[0]`, `landmarks[1]`처럼 쓸 때마다 발생하는 속성 조회 비용을 줄인다.

```python
        if smooth_h1 is None:
            smooth_h1 = [float(h1_cx), float(h1_cy)]
            smooth_h2 = [float(h2_cx), float(h2_cy)]
            alpha1, alpha2 = 0.5, 0.5  # 첫 프레임: alpha를 미리 정의
        else:
            speed_h1 = math.hypot(h1_cx - smooth_h1[0], h1_cy - smooth_h1[1])
            speed_h2 = math.hypot(h2_cx - smooth_h2[0], h2_cy - smooth_h2[1])

            if speed_h1 < 2.5: speed_h1 = 0.0  # 데드존
            if speed_h2 < 2.5: speed_h2 = 0.0

            alpha1 = min(max(0.12 + speed_h1 / 25.0, 0.12), 0.95)
            alpha2 = min(max(0.12 + speed_h2 / 25.0, 0.12), 0.95)

            smooth_h1[0] += (h1_cx - smooth_h1[0]) * alpha1
            smooth_h1[1] += (h1_cy - smooth_h1[1]) * alpha1
            smooth_h2[0] += (h2_cx - smooth_h2[0]) * alpha2
            smooth_h2[1] += (h2_cy - smooth_h2[1]) * alpha2
```

`smooth_h1`이 `None`인 첫 진입 시 `alpha1`, `alpha2`를 `0.5`로 미리 정의한다. 이는 이 블록 이후 `box_alpha = (alpha1 + alpha2) / 2` 계산에서 `alpha1`이 정의되지 않아 발생하는 `UnboundLocalError`를 방지하기 위함이다.

### 12.8 Pan/Zoom/Rotate

```python
        box_alpha = (alpha1 + alpha2) / 2  # 두 손 알파의 평균을 오브젝트에 적용

        # Pan: 중점 추종
        mid_x = (smooth_h1[0] + smooth_h2[0]) / 2
        mid_y = (smooth_h1[1] + smooth_h2[1]) / 2
        smooth_center[0] += (mid_x - smooth_center[0]) * box_alpha
        smooth_center[1] += (mid_y - smooth_center[1]) * box_alpha

        # 거리 및 각도 계산
        current_hand_dist  = math.hypot(smooth_h1[0] - smooth_h2[0],
                                        smooth_h1[1] - smooth_h2[1])
        current_hand_angle = math.degrees(
            math.atan2(smooth_h2[1] - smooth_h1[1], smooth_h2[0] - smooth_h1[0])
        )

        if base_hand_dist is None:
            # 양손이 처음 잡힌 순간 기준값 저장
            base_hand_dist  = current_hand_dist
            base_hand_angle = current_hand_angle
            base_box_size   = smooth_size
        else:
            # Zoom: 거리 비율로 크기 계산
            scale_factor = current_hand_dist / (base_hand_dist + 1e-6)
            target_size  = min(max(base_box_size * scale_factor, 20), 200)
            smooth_size  += (target_size - smooth_size) * box_alpha

            # Rotate: 각도 변화량으로 회전
            target_angle = current_hand_angle - base_hand_angle
            angle_diff   = (target_angle - smooth_angle + 180) % 360 - 180
            smooth_angle += angle_diff * box_alpha
```

### 12.9 플릭 제스처 감지

```python
        label0 = detection_result.handedness[0][0].category_name
        if label0 == 'Left':
            left_hand, right_hand = hand1, hand2
            right_raw_cx, right_raw_cy = h2_cx, h2_cy
        else:
            left_hand, right_hand = hand2, hand1
            right_raw_cx, right_raw_cy = h1_cx, h1_cy
```

`handedness`로 어느 손이 왼손인지 오른손인지 판별한다. 미러(좌우 반전)를 먼저 적용했기 때문에 `'Left'`는 사용자 기준 왼손이다.

```python
        if right_raw_prev is not None and flick_cooldown == 0:
            rvx = right_raw_cx - right_raw_prev[0]
            rvy = right_raw_cy - right_raw_prev[1]
            left_ori  = hand_orientation(left_hand,  w, h)
            right_ori = hand_orientation(right_hand, w, h)

            if left_ori == 'H' and right_ori == 'V':
                if abs(rvx) > FLICK_VEL_THRESHOLD and abs(rvx) > abs(rvy) * 1.5:
                    last_gesture   = "<< FLICK LEFT" if rvx < 0 else "FLICK RIGHT >>"
                    gesture_timer  = GESTURE_DISPLAY_FRAMES
                    flick_cooldown = FLICK_COOLDOWN

            elif left_ori == 'V' and right_ori == 'H':
                if abs(rvy) > FLICK_VEL_THRESHOLD and abs(rvy) > abs(rvx) * 1.5:
                    last_gesture   = "FLICK UP" if rvy < 0 else "FLICK DOWN"
                    gesture_timer  = GESTURE_DISPLAY_FRAMES
                    flick_cooldown = FLICK_COOLDOWN

        right_raw_prev = (right_raw_cx, right_raw_cy)
```

### 12.10 손이 없을 때 초기화

```python
    else:
        base_hand_dist = None
        base_hand_angle = None
        smooth_h1, smooth_h2 = None, None
        right_raw_prev = None
```

손이 화면에서 사라지면 기준 거리, 각도와 스무딩 좌표를 모두 `None`으로 초기화한다. 다음에 손을 다시 올릴 때 이전 상태가 남아있지 않도록 보장한다.

### 12.11 최종 렌더링

앞서 소개한 양손 동작 인식을 프로젝트에 응용하기 위한 클래스 버전으로, 이를 이용하면 다양한 프로젝트에 양손 동작 인식을 적용할 수 있다.

```python
    flick_cooldown = max(0, flick_cooldown - 1)
    if gesture_timer > 0:
        gesture_timer -= 1

    # 회전 사각형 그리기
    rect        = ((smooth_center[0], smooth_center[1]),
                   (smooth_size * 2, smooth_size * 2),
                   smooth_angle)
    box_points  = cv2.boxPoints(rect)
    box_points  = np.int32(box_points)
    cv2.drawContours(frame, [box_points], 0, (0, 255, 255), 3)

    # 제스처 텍스트 표시 (타이머가 남아있을 때만)
    if gesture_timer > 0:
        cv2.putText(frame, last_gesture, (30, cam_h - 40),
                    cv2.FONT_HERSHEY_SIMPLEX, 1.1, (0, 120, 255), 3)

    # FPS 표시
    cv2.putText(frame, f"FPS: {fps_display:.1f}", (cam_w - 130, 30),
                cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 255), 2)

    cv2.imshow("Minority Report - Spatial UI", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

매 프레임마다 쿨다운과 제스처 타이머를 1씩 감소시킨다. 그 후 스무딩된 상태값으로 회전 사각형을 그리고, 제스처가 감지된 상태(`gesture_timer > 0`)라면 화면 하단에 텍스트를 표시한다.

---

## 13. 응용 코드


```python
import sys
import cv2
import mediapipe as mp
import math
import os
import numpy as np
import time
from typing import Callable, Dict, Optional, Tuple

BaseOptions = mp.tasks.BaseOptions
HandLandmarker = mp.tasks.vision.HandLandmarker
HandLandmarkerOptions = mp.tasks.vision.HandLandmarkerOptions
VisionRunningMode = mp.tasks.vision.RunningMode


class HandGestureController:
    def __init__(self, model_path: Optional[str] = None, camera_index: int = 0):
        current_dir = os.path.dirname(os.path.abspath(__file__))
        self.model_path = model_path or os.path.join(current_dir, "hand_landmarker.task")
        self.camera_index = camera_index

        self.callbacks: Dict[str, Optional[Callable]] = {
            "on_scale_up": None,
            "on_scale_down": None,
            "on_rotate": None,
            "on_left_angle_with_right_fixed": None,
        }

        self.base_hand_dist: Optional[float] = None
        self.base_hand_angle: Optional[float] = None
        self.base_box_size = 80.0

        self.smooth_points = {"Left": None, "Right": None}
        self.smooth_center = [320.0, 240.0]
        self.smooth_size = 80.0
        self.smooth_angle = 0.0

        self.last_scale_event = 1.0
        self.last_rotation_event = 0.0
        self.right_still_frames = 0
        self.is_right_fixed = False
        self.last_left_angle_event: Optional[float] = None

        self.scale_event_threshold = 0.03
        self.rotate_event_threshold = 1.5
        self.right_fixed_speed_threshold = 2.0
        self.right_fixed_frame_count = 6

    def set_callback(self, event_name: str, callback: Callable):
        if event_name not in self.callbacks:
            raise ValueError(f"Unsupported event name: {event_name}")
        self.callbacks[event_name] = callback

    def _emit(self, event_name: str, *args):
        callback = self.callbacks.get(event_name)
        if callback is not None:
            callback(*args)

    def _create_options(self) -> HandLandmarkerOptions:
        return HandLandmarkerOptions(
            base_options=BaseOptions(
                model_asset_path=self.model_path,
                delegate=mp.tasks.BaseOptions.Delegate.CPU,
            ),
            running_mode=VisionRunningMode.VIDEO,
            num_hands=2,
            min_hand_detection_confidence=0.3,
            min_hand_presence_confidence=0.3,
            min_tracking_confidence=0.3,
        )

    def _extract_handed_points(self, detection_result, width: int, height: int) -> Dict[str, Tuple[float, float]]:
        points: Dict[str, Tuple[float, float]] = {}
        hand_landmarks = detection_result.hand_landmarks
        handedness = detection_result.handedness

        for idx, landmarks in enumerate(hand_landmarks):
            if idx >= len(handedness) or not handedness[idx]:
                continue
            label = handedness[idx][0].category_name
            if label not in ("Left", "Right"):
                continue

            center_x = float(landmarks[9].x * width)
            center_y = float(landmarks[9].y * height)
            points[label] = (center_x, center_y)

        return points

    def _smooth_point(self, label: str, raw_x: float, raw_y: float) -> Tuple[float, float, float]:
        smooth_point = self.smooth_points[label]
        if smooth_point is None:
            self.smooth_points[label] = [raw_x, raw_y]
            return raw_x, raw_y, 0.5

        speed = math.hypot(raw_x - smooth_point[0], raw_y - smooth_point[1])
        if speed < 2.5:
            speed = 0.0

        alpha = min(max(0.12 + speed / 25.0, 0.12), 0.95)
        smooth_point[0] += (raw_x - smooth_point[0]) * alpha
        smooth_point[1] += (raw_y - smooth_point[1]) * alpha
        return smooth_point[0], smooth_point[1], alpha

    def _reset_two_hand_state(self):
        self.base_hand_dist = None
        self.base_hand_angle = None
        self.smooth_points["Left"] = None
        self.smooth_points["Right"] = None
        self.last_scale_event = 1.0
        self.last_rotation_event = 0.0
        self.right_still_frames = 0
        self.is_right_fixed = False
        self.last_left_angle_event = None

    def _dispatch_events(
        self,
        scale_factor: float,
        target_angle: float,
        smooth_left: Tuple[float, float],
        smooth_right: Tuple[float, float],
        right_speed: float,
    ):
        if (
            scale_factor >= 1.0 + self.scale_event_threshold
            and abs(scale_factor - self.last_scale_event) >= self.scale_event_threshold
        ):
            self._emit("on_scale_up", scale_factor)
            self.last_scale_event = scale_factor
        elif (
            scale_factor <= 1.0 - self.scale_event_threshold
            and abs(scale_factor - self.last_scale_event) >= self.scale_event_threshold
        ):
            self._emit("on_scale_down", scale_factor)
            self.last_scale_event = scale_factor

        rotate_delta = target_angle - self.last_rotation_event
        rotate_delta = (rotate_delta + 180) % 360 - 180
        if abs(rotate_delta) >= self.rotate_event_threshold:
            self._emit("on_rotate", target_angle)
            self.last_rotation_event = target_angle

        if right_speed < self.right_fixed_speed_threshold:
            self.right_still_frames += 1
        else:
            self.right_still_frames = 0
            self.is_right_fixed = False
            self.last_left_angle_event = None

        if self.right_still_frames >= self.right_fixed_frame_count:
            self.is_right_fixed = True

        if self.is_right_fixed:
            left_angle = math.degrees(math.atan2(smooth_left[1] - smooth_right[1], smooth_left[0] - smooth_right[0]))
            if self.last_left_angle_event is None:
                self._emit("on_left_angle_with_right_fixed", left_angle)
                self.last_left_angle_event = left_angle
            else:
                left_angle_diff = (left_angle - self.last_left_angle_event + 180) % 360 - 180
                if abs(left_angle_diff) >= self.rotate_event_threshold:
                    self._emit("on_left_angle_with_right_fixed", left_angle)
                    self.last_left_angle_event = left_angle

    def run(self):
        options = self._create_options()

        print("[SYSTEM] 카메라 초기화 중...")
        if sys.platform == "win32":
            cap = cv2.VideoCapture(self.camera_index, cv2.CAP_DSHOW)
        else:
            cap = cv2.VideoCapture(self.camera_index)

        if not cap.isOpened():
            print("[ERROR] 카메라를 열 수 없습니다.")
            return

        cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
        cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
        if sys.platform != "win32":
            cap.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc(*"MJPG"))
            cap.set(cv2.CAP_PROP_FPS, 30)

        cam_w = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
        cam_h = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
        self.smooth_center = [cam_w / 2.0, cam_h / 2.0]
        print(f"[SYSTEM] 캡처 해상도: {cam_w}x{cam_h}")

        h, w = cam_h, cam_w
        start_time = time.time()
        prev_frame_time = start_time
        fps_display = 0.0

        print("[SYSTEM] AI 모델 로드 중 (VIDEO 모드 순차 동기화 엄격 적용)...")
        with HandLandmarker.create_from_options(options) as landmarker:
            print("[SYSTEM] 미디어파이프 양손 제어 시스템 실행.")

            while cap.isOpened():
                ret, frame = cap.read()
                if not ret:
                    break

                frame = cv2.flip(frame, 1)
                now = time.time()
                elapsed = now - prev_frame_time
                prev_frame_time = now
                if elapsed > 0:
                    fps_display = fps_display * 0.9 + (1.0 / elapsed) * 0.1

                frame_timestamp_ms = int((now - start_time) * 1000)

                rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
                rgb_frame.flags.writeable = False
                mp_image = mp.Image(image_format=mp.ImageFormat.SRGB, data=rgb_frame)

                detection_result = landmarker.detect_for_video(mp_image, frame_timestamp_ms)
                handed_points = self._extract_handed_points(detection_result, w, h)

                if "Left" in handed_points and "Right" in handed_points:
                    left_raw = handed_points["Left"]
                    right_raw = handed_points["Right"]

                    left_x, left_y, alpha_left = self._smooth_point("Left", left_raw[0], left_raw[1])
                    right_x, right_y, alpha_right = self._smooth_point("Right", right_raw[0], right_raw[1])

                    shlx, shly = int(left_x), int(left_y)
                    shrx, shry = int(right_x), int(right_y)
                    cv2.circle(frame, (shlx, shly), 10, (255, 0, 0), -1)
                    cv2.circle(frame, (shrx, shry), 10, (0, 0, 255), -1)
                    cv2.line(frame, (shlx, shly), (shrx, shry), (255, 255, 255), 2)

                    box_alpha = (alpha_left + alpha_right) / 2
                    mid_x = (left_x + right_x) / 2
                    mid_y = (left_y + right_y) / 2
                    self.smooth_center[0] += (mid_x - self.smooth_center[0]) * box_alpha
                    self.smooth_center[1] += (mid_y - self.smooth_center[1]) * box_alpha

                    current_hand_dist = math.hypot(left_x - right_x, left_y - right_y)
                    current_hand_angle = math.degrees(math.atan2(left_y - right_y, left_x - right_x))

                    if self.base_hand_dist is None or self.base_hand_angle is None:
                        self.base_hand_dist = current_hand_dist
                        self.base_hand_angle = current_hand_angle
                        self.base_box_size = self.smooth_size
                    else:
                        scale_factor = current_hand_dist / (self.base_hand_dist + 1e-6)
                        target_size = min(max(self.base_box_size * scale_factor, 20), 200)
                        self.smooth_size += (target_size - self.smooth_size) * box_alpha

                        target_angle = current_hand_angle - self.base_hand_angle
                        angle_diff = (target_angle - self.smooth_angle + 180) % 360 - 180
                        self.smooth_angle += angle_diff * box_alpha

                        right_speed = math.hypot(right_raw[0] - right_x, right_raw[1] - right_y)
                        self._dispatch_events(
                            scale_factor=scale_factor,
                            target_angle=target_angle,
                            smooth_left=(left_x, left_y),
                            smooth_right=(right_x, right_y),
                            right_speed=right_speed,
                        )

                    cv2.putText(
                        frame,
                        "SPATIAL CONTROL ACTIVE",
                        (30, 50),
                        cv2.FONT_HERSHEY_SIMPLEX,
                        0.7,
                        (0, 255, 0),
                        2,
                    )
                else:
                    self._reset_two_hand_state()

                rect = (
                    (self.smooth_center[0], self.smooth_center[1]),
                    (self.smooth_size * 2, self.smooth_size * 2),
                    self.smooth_angle,
                )
                box_points = cv2.boxPoints(rect)
                box_points = np.int32(box_points)
                cv2.drawContours(frame, [box_points], 0, (0, 255, 255), 3)

                cv2.putText(
                    frame,
                    f"FPS: {fps_display:.1f}",
                    (cam_w - 130, 30),
                    cv2.FONT_HERSHEY_SIMPLEX,
                    0.7,
                    (0, 255, 255),
                    2,
                )

                cv2.imshow("Minority Report - Spatial UI", frame)
                if cv2.waitKey(1) & 0xFF == ord("q"):
                    break

        cap.release()
        cv2.destroyAllWindows()


def main():
    controller = HandGestureController()

    controller.set_callback("on_scale_up", lambda scale: print(f"[EVENT] SCALE_UP: {scale:.3f}"))
    controller.set_callback("on_scale_down", lambda scale: print(f"[EVENT] SCALE_DOWN: {scale:.3f}"))
    controller.set_callback("on_rotate", lambda angle: print(f"[EVENT] ROTATE: {angle:.2f} deg"))
    controller.set_callback(
        "on_left_angle_with_right_fixed",
        lambda angle: print(f"[EVENT] LEFT_ANGLE_WHEN_RIGHT_FIXED: {angle:.2f} deg"),
    )

    controller.run()


if __name__ == "__main__":
    main()
```

## 부록 A. MediaPipe 랜드마크 심화

### A.1 정규화 좌표계

출력되는 $x$와 $y$ 값은 이미지의 실제 픽셀 크기와 관계없이 $0.0$에서 $1.0$ 사이의 상대적인 값을 가진다.
- $(x: 0.0, y: 0.0)$: 이미지의 좌측 상단 (Top-Left)
- $(x: 1.0, y: 1.0)$: 이미지의 우측 하단 (Bottom-Right)

### A.2 z 좌표의 의미

`z` 좌표는 절대적인 카메라와의 거리가 아니다. 손목(0번)을 원점(`z=0`)으로 한 **상대적 깊이**이다. 손목보다 카메라에 가까우면 음수(`-`), 멀면 양수(`+`)이다. 주로 손가락이 앞으로 구부러졌는지 뒤로 젖혀졌는지 판별하는 용도로 사용한다.

### A.3 거리 정규화의 중요성

카메라와 사용자 사이의 거리가 바뀌면 픽셀 거리가 변한다. 따라서 항상 손목(0번)과 중지 MCP(9번) 사이의 거리를 기준 단위(1.0)로 잡고, 다른 거리들을 이에 대해 정규화(Normalization)하여 사용해야 거리 변화에 강인한 시스템이 된다.

### A.4 Face Mesh 핵심 랜드마크

| 부위 | 인덱스 | 활용 |
|------|--------|------|
| 코 끝 | 4 | 고개 방향 추적 |
| 왼쪽 눈 윤곽 | 362~398 | 눈 깜빡임(EAR) |
| 오른쪽 눈 윤곽 | 33~246 | 눈 깜빡임(EAR) |
| 입술 윤곽 | 61~308 | 입 개폐 여부 |
| 홍채 (478개 모델) | 468~477 | 시선 추적(Gaze) |

### A.5 눈 종횡비 EAR (Eye Aspect Ratio)

눈이 열렸는지 닫혔는지를 수치로 판별하는 공식이다.

$$EAR = \frac{\|P_2 - P_6\| + \|P_3 - P_5\|}{2 \|P_1 - P_4\|}$$

EAR이 특정 임계값(예: 0.2) 이하로 떨어지면 눈을 감은 것으로 판단한다.

### A.6 벡터 내적으로 관절 각도 계산

관절의 구부러진 각도를 계산하여 손가락이 접혔는지 판별할 때 사용한다. 세 점 $A$, $B$, $C$가 이루는 사잇각 $\theta$는 두 벡터 $\vec{BA}$와 $\vec{BC}$의 내적으로 구한다.

$$\theta = \arccos\left(\frac{\vec{BA} \cdot \vec{BC}}{\|\vec{BA}\| \|\vec{BC}\|}\right)$$

```python
import numpy as np

def calculate_angle(a, b, c):
    a, b, c = np.array(a), np.array(b), np.array(c)
    ba = a - b
    bc = c - b
    cosine = np.dot(ba, bc) / (np.linalg.norm(ba) * np.linalg.norm(bc))
    return np.degrees(np.arccos(np.clip(cosine, -1.0, 1.0)))
```

---

## 부록 B. 수학 공식 정리

| 공식 | 수식 | 코드 |
|------|------|------|
| 픽셀 변환 | $p_x = \text{int}(lm.x \times w)$ | `int(lm.x * w)` |
| 유클리드 거리 | $d = \sqrt{\Delta x^2 + \Delta y^2}$ | `math.hypot(dx, dy)` |
| 방향 각도 | $\theta = \text{atan2}(\Delta y, \Delta x)$ | `math.degrees(math.atan2(dy, dx))` |
| EMA 스무딩 | $s_{n+1} = s_n + (r - s_n) \cdot \alpha$ | `s += (r - s) * alpha` |
| 각도 정규화 | $(d + 180) \bmod 360 - 180$ | `(d + 180) % 360 - 180` |
| 가변 알파 | $\alpha = \text{clip}(0.12 + v/25, 0.12, 0.95)$ | `min(max(0.12 + v/25, 0.12), 0.95)` |
| EAR | $\frac{\|P_2-P_6\|+\|P_3-P_5\|}{2\|P_1-P_4\|}$ | 부록 A.5 참고 |

