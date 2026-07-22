# 실무 Python 핵심 기술

파이썬은 간결한 문법 덕분에 배우기 쉽지만, 안정적이고 빠른 프로그램을 만들려면 메모리 관리, 데이터 흐름, 동시 실행 같은 핵심 원리를 이해해야 한다. 이 문서는 대용량 데이터 처리, 네트워크 통신, 하드웨어 계측처럼 개발에서 자주 마주치는 상황을 중심으로, 실무에서 꼭 필요한 Python 기술 계층을 단계별로 정리한다.

## 메모리와 객체의 본질 메커니즘

> **풀고 싶은 문제**
> 함수에 리스트를 전달했을 뿐인데 원본 데이터가 뒤바뀌는 일이 생겼다. 네트워크 설정값을 실수로 덮어써서 시스템 전체가 엉키기도 한다. 데이터를 안전하게 전달하고 복사하는 정확한 방법은 무엇일까?

### 뮤터블(Mutable) vs 이뮤터블(Immutable)

뮤터블(변경 가능) 타입인 list, dict, set은 내부 값이 바뀌어도 메모리 주소(id)가 그대로 유지된다. 함수 인자로 전달한 뒤 내부를 수정하면 원본 데이터가 오염(Side Effect)된다.

이뮤터블(변경 불가능) 타입인 int, float, str, tuple은 값을 수정할 때 기존 객체를 고치는 것이 아니라 새로운 메모리 공간에 객체를 만들고 참조 주소를 바꾼다.

```python
nums = [10, 20, 30]        # 뮤터블: 리스트
text = "hello"             # 이뮤터블: 문자열

print(f"수정 전 id(nums): {id(nums)}")
nums.append(40)
print(f"수정 후 id(nums): {id(nums)}  ← 주소 그대로 (같은 객체)")

print(f"수정 전 id(text): {id(text)}")
text = text + " world"
print(f"수정 후 id(text): {id(text)}  ← 주소 바뀜 (새 객체)")

# 함수 인자로 넘긴 뮤터블 객체는 내부 수정이 원본에 반영됨
def add_item(lst, value):
    lst.append(value)

original = [1, 2, 3]
add_item(original, 99)
print(f"함수 호출 뒤 original: {original}")  # [1, 2, 3, 99] — 원본 오염
```

**코드 해설**

nums.append(40)은 리스트 객체 자체를 수정하므로 id()가 바뀌지 않는다. text = text + " world"는 새 문자열 객체를 만들고 text 변수가 그 새 객체를 가리키게 한다. add_item은 lst.append로 리스트 내부를 수정하기 때문에 함수 밖의 original도 함께 바뀐다.

**실행 결과와 확인할 점**

nums의 id가 수정 전후에 같고, text의 id는 수정 전후에 다른 것을 확인한다. 함수 호출 뒤 original에 99가 추가되어 있는 것도 확인한다.

### 얕은 복사(Shallow) vs 깊은 복사(Deep)

참조 대입(=)은 주소만 복사하므로 두 변수가 완전히 같은 메모리 영역을 가리킨다.

얕은 복사(.copy(), [:])는 겉껍데기 컨테이너는 새로 만들지만, 내부의 중첩 객체(2차원 배열, 딕셔너리 내부의 리스트 등)는 원본 주소를 그대로 공유한다. 내부 요소를 수정하면 원본이 오염된다.

깊은 복사(copy.deepcopy())는 내부에 중첩된 모든 하위 객체까지 추적하여 메모리에 완전히 독립된 공간으로 복제한다.

```python
import copy

# 중첩된 설정(Config) 데이터의 안전한 방어적 복사
original_config = {"server": "127.0.0.1", "ports": [8080, 8081]}

# 얕은 복사의 함정: 내부 리스트(ports)의 주소는 공유됨
shallow_clone = original_config.copy()
shallow_clone["ports"][0] = 9000
print(original_config["ports"][0])  # 출력: 9000 (원본이 오염됨!)

# 깊은 복사: 완전한 독립 공간 확보
original_config["ports"][0] = 8080  # 원상복구
deep_clone = copy.deepcopy(original_config)
deep_clone["ports"][0] = 9000
print(original_config["ports"][0])  # 출력: 8080 (원본 보호 완벽)
```

**코드 해설**

shallow_clone은 original_config 딕셔너리의 겉 구조만 새로 만든 복사본이다. "ports" 키가 가리키는 내부 리스트는 두 변수가 여전히 같은 객체를 공유하기 때문에, shallow_clone을 통해 수정하면 original_config도 같이 바뀐다. copy.deepcopy()를 쓰면 내부 리스트까지 완전히 새 객체로 복제되므로 어느 쪽을 수정해도 서로 영향을 주지 않는다.

**실행 결과와 확인할 점**

얕은 복사 구간에서 9000이 출력되어 원본이 오염된 것이 보여야 한다. 깊은 복사 구간에서는 8080이 그대로 출력되어 원본이 보호된 것을 확인한다. 중첩이 없는 단순한 딕셔너리라면 .copy()로도 충분하지만, 리스트나 딕셔너리가 안에 있으면 deepcopy를 써야 한다.

> **한 걸음 더**
> original_config에서 "server" 값만 따로 shallow_clone을 통해 바꾸어 보고, 이번에는 원본이 오염되지 않는 이유를 설명해 본다. 문자열은 이뮤터블이라 수정 시 새 객체가 만들어지기 때문이다.

### 변수 가시성 (LEGB 스코프)

변수를 참조할 때 파이썬은 Local(함수 내부) -> Enclosed(부모 함수) -> Global(파일 전역) -> Built-in(Python 내장) 순서로 이름을 검색한다. 함수 내부에서 전역 변수를 수정하려면 global, 상위 함수의 변수를 수정하려면 nonlocal을 명시해야 한다.

```python
counter = 0  # Global 변수

def outer():
    score = 100  # Enclosed 변수

    def inner():
        nonlocal score   # 부모 함수 score를 수정하겠다는 선언
        global counter   # 전역 counter를 수정하겠다는 선언
        score += 10
        counter += 1
        print(f"inner 호출 후 score={score}, counter={counter}")

    inner()
    inner()
    print(f"outer 종료 시 score={score}")

outer()
print(f"전역 counter={counter}")
```

**코드 해설**

선언 없이 inner 안에서 score = score + 10을 실행하면 파이썬은 score를 Local 변수로 취급하려다 초기화 전에 읽는 상황이 되어 UnboundLocalError를 낸다. nonlocal score를 선언하면 검색 범위가 Enclosed까지 확장되어 outer의 score를 합법적으로 수정할 수 있다. global counter도 마찬가지로 전역 영역의 변수를 직접 변경하겠다는 명시이다.

**실행 결과와 확인할 점**

inner를 두 번 호출하면 score는 100 -> 110 -> 120으로, counter는 0 -> 1 -> 2로 변한다. nonlocal이나 global 키워드를 제거하고 실행해서 어떤 에러가 발생하는지도 확인해 본다.

### 가비지 콜렉션(GC)과 참조 계수

파이썬은 메모리 관리를 가비지 콜렉터(GC)에게 위임한다. 기본 전략은 참조 계수(Reference Counting)이다. 객체를 가리키는 변수나 컨테이너가 하나라도 있으면 참조 계수가 1 이상이므로 메모리가 해제되지 않는다. 참조 계수가 0이 되는 순간 즉시 메모리를 반환한다.

두 객체가 서로를 참조하는 순환 참조(Circular Reference)는 참조 계수가 영원히 0이 되지 않아 메모리 누수가 생긴다. 파이썬의 주기적 GC(gc 모듈)가 이를 탐지해 정리하지만, 대용량 처리에서 누적되면 문제가 된다.

```python
import gc

class Node:
    def __init__(self, val):
        self.val = val
        self.ref = None

# 순환 참조 만들기
a = Node("A")
b = Node("B")
a.ref = b   # a -> b
b.ref = a   # b -> a (순환!)

# 변수 삭제 후에도 a, b 내부 참조가 남아 참조 계수가 0이 되지 않음
del a, b
collected = gc.collect()   # 순환 참조 탐지 및 수거
print(f"GC가 수거한 객체 수: {collected}")
```

**코드 해설**

del a, b는 변수만 삭제한다. 두 Node 객체는 서로를 참조하고 있으므로 참조 계수가 1로 남아 자동 소멸되지 않는다. gc.collect()를 호출하면 GC가 순환 그래프를 탐지하고 메모리를 회수한다. 실무에서 핸들러나 콜백이 객체를 오래 붙들고 있으면 비슷한 누수가 생기므로, 명시적 해제나 weakref 모듈을 고려해야 한다.

**실행 결과와 확인할 점**

collected 값이 2(두 객체) 이상으로 출력되어 순환 참조가 수거된 것을 확인한다. 순환 참조 없는 단순한 객체로 테스트하면 gc.collect()가 0을 반환하는 것도 비교해 본다.

---

## 고성능 자원 관리 및 데이터 흐름 최적화

> **풀고 싶은 문제**
> 10GB짜리 로그 파일을 분석하려고 모든 줄을 리스트에 담았더니 프로그램이 멈춰 버렸다. 메모리를 넘치지 않게 데이터를 한 줄씩 처리하는 방법은 없을까?

### 제어 구조(Control Structures)와 루프 최적화

if-elif-else 구조에서는 가장 빈번하게 발생하는 조건을 최상단에 배치하여 불필요한 비교 연산 비용을 줄인다.

루프 내부에서 매번 외부 함수를 호출하거나 불변 연산을 반복하는 것은 성능에 악영향을 준다. 불변 연산은 루프 밖으로 꺼내고(Hoist), 자주 쓰는 메서드는 로컬 변수에 바인딩하여 탐색 비용을 줄인다.

```python
import time

data = list(range(1_000_000))

# 최적화 전: 루프 내부에서 매번 len()과 data.append 탐색
result_slow = []
start = time.time()
for i in range(len(data)):          # len(data) 매 반복 재계산
    if data[i] % 2 == 0:
        result_slow.append(data[i])
print(f"최적화 전: {(time.time() - start) * 1000:.1f}ms, 결과 수: {len(result_slow)}")

# 최적화 후: 불변 연산 호이스팅 + 메서드 로컬 바인딩
result_fast = []
_append = result_fast.append       # 메서드 탐색 1회로 고정
n = len(data)                      # 루프 밖으로 호이스팅
start = time.time()
for i in range(n):
    if data[i] % 2 == 0:
        _append(data[i])
print(f"최적화 후:  {(time.time() - start) * 1000:.1f}ms, 결과 수: {len(result_fast)}")
```

**코드 해설**

파이썬에서 obj.method 같은 속성 탐색은 매번 딕셔너리 검색을 거친다. 루프가 100만 번 돌면 이 탐색도 100만 번 일어난다. _append = result_fast.append처럼 로컬 변수에 한 번만 바인딩해 두면 이후 반복에서는 로컬 참조만 확인하므로 훨씬 빠르다. 리스트 컴프리헨션을 쓸 수 없는 복잡한 루프일수록 이 기법의 효과가 크다.

**실행 결과와 확인할 점**

두 버전의 소요 시간 차이를 확인한다. data의 크기를 10_000_000으로 늘려 더 큰 차이를 체감해 본다.

### 문자열 빌딩과 join 패턴

같은 루프 최적화 맥락에서 문자열 빌딩 패턴도 주의해야 한다. 루프 안에서 result += f"..." 처럼 문자열을 누적하면 매 반복마다 새 문자열 객체가 생성되어 전체 연산이 O(n²)이 된다. 요소를 리스트에 모은 뒤 "".join(parts)로 한 번에 연결하면 O(n)이 된다.

```python
import time

# O(n²) — 루프 내 += 패턴: 각 반복마다 이전 문자열 전체를 복사
start = time.time()
report = ""
for i in range(10_000):
    report += f"센서_{i}:{i*0.1:.1f} "   # 매 반복 새 문자열 객체 생성
print(f"+= 방식: {(time.time()-start)*1000:.2f}ms, 길이={len(report)}")

# O(n) — join 패턴: 제너레이터 표현식으로 요소를 한 번에 연결
start = time.time()
report = " ".join(f"센서_{i}:{i*0.1:.1f}" for i in range(10_000))
print(f"join 방식: {(time.time()-start)*1000:.2f}ms, 길이={len(report)}")
```

**코드 해설**

두 방식의 출력 결과는 동일하다. += 방식은 반복 횟수가 n이면 총 복사 문자 수가 0 + k + 2k + ... 로 O(n²)에 비례한다. join 방식은 내부에서 전체 길이를 미리 계산하여 메모리를 한 번만 할당하고 채운다. 제너레이터 표현식을 join에 직접 전달하면 중간 리스트조차 생성하지 않으므로 가장 효율적이다.

**실행 결과와 확인할 점**

join 방식이 += 방식보다 빠른 것을 확인한다. range(10_000)을 range(100_000)으로 늘리면 차이가 더욱 뚜렷해진다. 로그 행을 연결하거나 CSV 직렬화 시 join 패턴을 기본으로 사용한다.

### 이터레이터(Iterator)와 제너레이터(Generator)

제너레이터는 함수 내부에 return 대신 yield를 사용하여 데이터 스트림을 한 건씩 실시간으로 생성하고 반환한다. 10GB 이상의 대용량 로그나 센서 스트리밍 데이터를 다룰 때 모든 데이터를 메모리에 올리지 않아도 되므로, 메모리 고갈(Out of Memory)을 원천적으로 막는다.

> **이 예제로 풀 문제**
> 수백만 개의 센서 데이터를 리스트로 한꺼번에 만들면 메모리가 금방 찬다. 제너레이터로 한 건씩 만들어 내면 메모리 점유 없이 같은 처리를 할 수 있다.

```python
def sensor_data_stream(total):
    for i in range(total):
        yield i * 0.1  # 0.0, 0.1, 0.2, ... 순으로 하나씩 내보냄

# 리스트 방식: 전체 데이터를 메모리에 올림
data_list = [i * 0.1 for i in range(1_000_000)]
print(f"리스트 크기: {data_list.__sizeof__()} bytes")

# 제너레이터 방식: 객체 자체는 거의 메모리를 차지하지 않음
data_gen = sensor_data_stream(1_000_000)
print(f"제너레이터 크기: {data_gen.__sizeof__()} bytes")

# 실제 사용할 때만 값이 계산됨
count = sum(1 for v in sensor_data_stream(1_000_000) if v > 50_000)
print(f"50000 초과 데이터 수: {count}")
```

**코드 해설**

sensor_data_stream은 호출 즉시 모든 값을 만들지 않는다. next()로 값을 요청할 때마다 yield 지점에서 멈췄다가 이어서 실행된다. 이 특성 덕분에 1백만 개짜리 스트림도 메모리에는 딱 한 개 분량만 올라온다. sum(1 for ...)는 제너레이터 표현식을 이용해 임시 리스트 없이 조건부 집계를 수행하는 전형적인 패턴이다.

**실행 결과와 확인할 점**

리스트는 수 MB 이상의 메모리를 점유하지만 제너레이터 객체는 수백 바이트에 불과하다. __sizeof__() 출력 값을 비교하여 차이를 확인한다.

### 컴프리헨션(Comprehension)과 표현식

리스트 컴프리헨션 [x for x in data if x > 0]은 데이터를 가공하여 리스트로 즉시 적재한다. Python 내부 C 엔진에서 최적화가 이루어지므로 일반 for 루프보다 빠르다.

제너레이터 표현식 (x for x in data if x > 0)은 소괄호를 사용하며, 메모리를 소모하지 않고 데이터 요청이 올 때까지 계산을 미룬다.

딕셔너리 컴프리헨션 {k: v for k, v in items}와 집합 컴프리헨션 {x for x in data}도 같은 원리로 동작한다.

```python
raw_temps = [38.1, -1.0, 37.5, None, 36.9, 200.0, 37.2]

# 리스트 컴프리헨션: 유효 범위(0~50도)만 필터링하여 즉시 리스트로
valid_temps = [t for t in raw_temps if t is not None and 0 <= t <= 50]
print(f"유효 온도: {valid_temps}")

# 딕셔너리 콘프리헨션: 측정 번호를 키로 매핑
indexed = {i: t for i, t in enumerate(valid_temps)}
print(f"인덱스 매핑: {indexed}")

# 제너레이터 표현식: 평균 계산 시 임시 리스트 없이 메모리 절약
average = sum(t for t in valid_temps) / len(valid_temps)
print(f"평균 온도: {average:.2f}°C")
```

**코드 해설**

리스트 컴프리헨션의 if 절은 데이터를 필터링하는 조건이다. None 체크를 먼저 하지 않으면 None <= 50 비교에서 TypeError가 발생하므로 t is not None을 앞에 놓는다. sum(t for t in valid_temps)는 임시 리스트를 만들지 않고 합산하므로 대규모 데이터에서 특히 유리하다.

**실행 결과와 확인할 점**

유효 온도 리스트에 -1.0, None, 200.0이 빠진 [38.1, 37.5, 36.9, 37.2]가 출력되어야 한다. raw_temps에 잘못된 값을 더 넣어 필터 조건이 제대로 작동하는지 확인한다.

### with 문과 컨텍스트 매니저 (Context Manager)

__enter__와 __exit__ 메서드를 통해 소켓, 파일, DB 커넥션 등의 자원을 블록 종료 시 자동으로 반환하여 자원 누수(Leak)를 막는다.

```python
import tempfile
import os

# with 문 없이 열면 예외 발생 시 close()가 보장되지 않음
# with 문을 쓰면 블록 종료 시 __exit__가 자동으로 파일을 닫아 줌
with tempfile.NamedTemporaryFile(
    mode='w', suffix='.log', delete=False, encoding='utf-8'
) as f:
    for i in range(5):
        f.write(f"로그 {i}: 센서 데이터 수신\n")
    log_path = f.name

# with 블록 밖에서는 f가 이미 닫혀 있음
with open(log_path, 'r', encoding='utf-8') as f:
    for line in f:
        print(line, end="")

os.unlink(log_path)  # 임시 파일 정리
```

**코드 해설**

open() 후 close()를 직접 호출하면 예외가 발생했을 때 close()가 실행되지 않아 파일 핸들이 누수된다. with 문은 블록을 정상 종료하든 예외로 탈출하든 상관없이 __exit__를 통해 close()를 보장한다. 소켓, DB 커넥션, 락 등 자원을 다룰 때 with 문을 습관적으로 적용하면 자원 누수를 원천 차단할 수 있다.

**실행 결과와 확인할 점**

다섯 줄의 로그가 출력된다. with 블록 밖에서 f.write()를 시도하면 ValueError: I/O operation on closed file이 발생하는 것을 직접 확인해 본다.

### itertools와 functools.lru_cache

itertools는 데이터 스트림을 메모리 없이 조작하는 표준 이터레이터 도구 모음이다. islice는 제너레이터나 무한 스트림에서 원하는 범위만 잘라내어 처리하며, chain은 여러 이터러블을 순서대로 연결한다.

functools.lru_cache는 동일한 인자로 반복 호출되는 함수의 결과를 캐시에 저장하여 재계산을 막는다. 센서 교정 계수 조회, 설정 파싱처럼 입력이 제한적이고 연산이 비싼 함수에 적합하다.

```python
import itertools
import functools
import time

# chain: 여러 센서 데이터 스트림을 하나로 연결
zone_a = [38.1, 37.5, 39.0]
zone_b = [36.2, 37.8]
all_readings = list(itertools.chain(zone_a, zone_b))
print(f"통합 센서 데이터: {all_readings}")

# islice: 무한 제너레이터에서 앞 N개만 처리
def infinite_sensor():
    val = 0
    while True:
        yield val * 0.1
        val += 1

first_five = list(itertools.islice(infinite_sensor(), 5))
print(f"앞 5개 샘플: {first_five}")

# lru_cache: 반복 호출 시 재계산 없이 캐시에서 즉시 반환
@functools.lru_cache(maxsize=128)
def get_calibration(sensor_id: int) -> float:
    time.sleep(0.01)  # DB 조회를 모사
    return 1.0 + sensor_id * 0.002

start = time.time()
for _ in range(100):
    get_calibration(5)  # 두 번째부터는 캐시에서 반환
print(f"lru_cache 100회 호출: {(time.time() - start) * 1000:.2f}ms")
print(f"캐시 정보: {get_calibration.cache_info()}")
```

**코드 해설**

itertools.chain은 여러 이터러블을 메모리에 합치지 않고 순서대로 이어 반환한다. islice는 무한 제너레이터에서 원하는 수만큼 꺼내어 처리할 수 있으므로 스트리밍 데이터 처리에 필수적이다. lru_cache는 함수 정의에 @을 붙이기만 하면 동작하며, cache_info()로 히트/미스 현황을 확인할 수 있다.

**실행 결과와 확인할 점**

100회 반복 호출이 거의 0ms에 가깝게 완료되어야 한다. cache_info()에서 hits=99, misses=1이 나오는 것을 확인한다. maxsize=0으로 바꾸면 캐시가 비활성화되어 약 1초가 걸리는 것과 비교해 본다.

---

## 함수 설계 원칙과 가변 인자 메커니즘

> **풀고 싶은 문제**
> 팀원이 만든 함수를 반복 호출하니 어느 순간부터 이상한 데이터가 쌓이기 시작했다. 언제 이런 버그가 생기는지, 그리고 인자를 유연하게 받는 함수를 어떻게 설계하는지 알고 싶다.

### 함수 기본값 정의의 안티 패턴

함수 매개변수에 기본값으로 뮤터블 객체(def func(data=[]))를 선언하면 안 된다. 파이썬은 함수가 정의되는 시점에 기본값 객체를 딱 한 번만 생성하므로, 함수를 여러 번 호출할 때 해당 객체가 메모리상에서 공유되어 데이터가 누적되는 버그가 생긴다. 기본값은 반드시 None으로 설정하고 내부에서 초기화해야 한다.

```python
# 안티 패턴: 기본값에 뮤터블 객체를 직접 사용
def collect_event_bad(event, log=[]):
    log.append(event)
    return log

print(collect_event_bad("시작"))    # ['시작']
print(collect_event_bad("오류"))    # ['시작', '오류']  ← 이전 호출 값이 누적!

# 올바른 패턴: None으로 선언하고 내부에서 매번 새로 초기화
def collect_event_good(event, log=None):
    if log is None:
        log = []
    log.append(event)
    return log

print(collect_event_good("시작"))   # ['시작']
print(collect_event_good("오류"))   # ['오류']  ← 독립적, 누적 없음
```

**코드 해설**

collect_event_bad의 log=[]는 함수 정의 시점에 단 한 번 생성된 리스트 객체이다. 모든 호출이 이 하나의 객체를 공유하므로 호출할 때마다 값이 쌓인다. collect_event_good은 호출할 때마다 조건 분기에서 새 리스트 []를 만들기 때문에 각 호출이 독립된 상태로 시작한다.

**실행 결과와 확인할 점**

collect_event_bad를 두 번 호출하면 두 번째에서 ['시작', '오류']가 나와 오염이 보인다. collect_event_good은 매번 독립된 ['시작'], ['오류']가 출력된다.

### 가변 인자 패킹 및 언패킹 (*args, **kwargs)

*args는 위치 인자들을 튜플(tuple)로 묶어(Packing) 받고, **kwargs는 키워드 인자들을 딕셔너리(dict)로 묶어 받는다. 반대로 함수를 호출할 때 리스트나 딕셔너리 앞에 *, **를 붙이면 요소를 풀어서(Unpacking) 전달할 수 있어서, 유연한 래퍼(Wrapper) 함수나 API 설계가 가능해진다.

```python
def send_log_packet(message, tags=None, *args, **kwargs):
    # 안티 패턴: def send_log_packet(message, tags=[]) 이렇게 쓰면 tags가 누적 오염됨
    if tags is None:
        tags = []  # 호출 시마다 독립적인 리스트 할당
    tags.append("system")
    print(f"[LOG] {message} | 태그: {tags}")
    if kwargs:
        print(f" └ 추가 메타데이터: {kwargs}")

meta_info = {"device_id": 45, "ip": "192.168.1.100"}
send_log_packet("하드웨어 연결 성공", ["HW", "Auth"], **meta_info)
send_log_packet("재연결 시도")  # tags가 이전 호출의 값 없이 새로 시작되는지 확인
```

**코드 해설**

두 번째 호출에서 tags가 None이 되어 새 리스트 []로 시작되는 것을 확인한다. 만약 def send_log_packet(message, tags=[])로 선언했다면 첫 번째 호출에서 tags에 "system"이 추가된 뒤 그 리스트가 그대로 남아 두 번째 호출에서는 ["system", "system"]이 되어 버린다. **meta_info는 딕셔너리를 keyword=value 형태로 풀어서 전달하는 언패킹 기법이다.

**실행 결과와 확인할 점**

두 번의 호출에서 tags 목록이 각각 독립적으로 ["HW", "Auth", "system"]과 ["system"]으로 출력되어야 한다. tags=[] 방식으로 바꾼 뒤 두 번 호출하면 어떻게 달라지는지 직접 비교해 본다.

---

## 일급 객체 특성을 활용한 기능 추상화

> **풀고 싶은 문제**
> 실행 시간 측정, 로그 기록, 인증 확인 같은 공통 로직을 여러 함수에 각각 붙여 쓰고 있다. 나중에 수정할 때마다 모든 함수를 하나씩 고쳐야 한다. 함수를 수정하지 않고 공통 로직을 한 곳에서 관리할 방법은?

### 람다(Lambda)

이름이 없는 일회성 익명 함수로, 정렬의 기준 세팅(key=lambda x: x[1])이나 단순 데이터 매핑 처리에 주로 쓰인다.

```python
sensors = [("A구역", 38.2), ("B구역", 35.1), ("C구역", 40.5), ("D구역", 36.8)]

# lambda를 정렬 기준으로 사용
sorted_sensors = sorted(sensors, key=lambda item: item[1], reverse=True)
for name, temp in sorted_sensors:
    print(f"{name}: {temp}°C")

# filter + lambda: 38도 초과 구역만 추출
hot_zones = list(filter(lambda item: item[1] > 38.0, sensors))
print(f"고온 경보 구역: {[name for name, _ in hot_zones]}")
```

**코드 해설**

lambda item: item[1]은 튜플 item에서 두 번째 요소(온도값)를 꺼내는 함수를 한 줄로 표현한 것이다. def로 정의하면 네 줄이 필요한 내용을 한 줄로 줄인다. 다만 복잡한 로직에 lambda를 남용하면 오히려 가독성이 떨어지므로, 단순한 키 추출이나 필터 조건에만 사용하는 것이 좋다.

**실행 결과와 확인할 점**

C구역(40.5), A구역(38.2), D구역(36.8), B구역(35.1) 순으로 출력되어야 한다. 고온 경보 구역은 ["A구역", "C구역"]이 나온다.

### 클로저(Closure)

상위 함수가 실행을 마치고 메모리에서 해제되었음에도, 하위 함수가 상위 함수의 로컬 변수 환경을 가두어(Enclosure) 기억하고 유지하는 구조이다. 전역 변수를 쓰지 않고 특정 상태를 안전하게 은닉할 수 있다.

```python
def make_threshold_alarm(threshold):
    call_count = [0]  # 리스트로 감싸서 nonlocal 없이 수정 가능

    def alarm(value):
        call_count[0] += 1
        if value > threshold:
            print(f"[경보 #{call_count[0]}] 측정값 {value}가 임계값 {threshold}를 초과!")
        else:
            print(f"[정상 #{call_count[0]}] 측정값 {value} (임계값 {threshold})")

    return alarm

temp_alarm = make_threshold_alarm(38.0)
pressure_alarm = make_threshold_alarm(1.5)

temp_alarm(37.5)
temp_alarm(39.1)
pressure_alarm(1.2)
pressure_alarm(1.8)
```

**코드 해설**

make_threshold_alarm이 반환하고 나면 threshold와 call_count는 일반적으로 소멸해야 한다. 그러나 반환된 alarm 함수가 이들을 참조하고 있으므로, 파이썬은 이 환경을 살아있는 상태로 유지한다. 이것이 클로저다. temp_alarm과 pressure_alarm은 각자 독립된 threshold와 call_count를 갖고 있어 서로 간섭하지 않는다.

**실행 결과와 확인할 점**

temp_alarm은 38.0을, pressure_alarm은 1.5를 기억한다. 각각 독립적으로 호출 횟수를 세므로, 번호가 따로 쌓인다.

### 데코레이터(Decorator)

기존 함수의 소스 코드를 전혀 수정하지 않고 앞뒤에 공통 로직(실행 시간 측정, 사용자 인증, 로깅, 예외 복구 등)을 가로채어 실행하는 모듈화 기법이다.

```python
import time

def execution_timer(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        duration = (time.time() - start_time) * 1000
        print(f"[LOG] [{func.__name__}] 소요 시간: {duration:.2f}ms")
        return result
    return wrapper

@execution_timer
def heavy_data_processing():
    total = sum(range(1_000_000))
    return total

result = heavy_data_processing()
print(f"연산 결과: {result}")
```

**코드 해설**

@execution_timer는 heavy_data_processing = execution_timer(heavy_data_processing)과 동일하다. heavy_data_processing()을 호출하면 실제로는 wrapper가 실행되고, 그 안에서 원본 함수를 감싸며 시간을 잰다. 원본 함수를 전혀 건드리지 않아도 되므로, 로깅이나 권한 검사 같은 횡단 관심사(Cross-Cutting Concern)를 한 곳에서 관리할 수 있다.

**실행 결과와 확인할 점**

함수 이름과 소요 시간이 로그로 출력된 뒤 연산 결과가 나온다. @execution_timer를 제거하면 시간 측정 없이 그냥 실행된다는 점도 비교해 본다.

---

## 내장 메모리 버퍼와 직렬화 아키텍처

> **풀고 싶은 문제**
> 마이크로컨트롤러에서 전송된 6바이트 패킷을 파이썬에서 받았다. 이 바이트 덩어리를 장비 ID와 온도값으로 분리하려면 어디서부터 시작해야 할까?

### bytes와 bytearray

시스템 및 네트워크 레이어는 Python 객체가 아닌 이진 바이트 스트림만 이해한다. bytes는 수정 불가능한 고정 버퍼이며, bytearray는 동적으로 데이터를 추가하고 수정할 수 있는 가변 버퍼이다.

```python
# bytes: 고정 버퍼 (수신 완료된 패킷 보관용)
fixed_buf = bytes([0x01, 0x02, 0xA3, 0xFF])
print(f"bytes: {fixed_buf}, 첫 바이트: {fixed_buf[0]}")

# bytearray: 가변 버퍼 (패킷을 조각씩 조립할 때)
dynamic_buf = bytearray()
dynamic_buf.append(0x01)
dynamic_buf.extend([0x02, 0xA3])
dynamic_buf.append(0xFF)
print(f"bytearray: {dynamic_buf}")

# bytes로 변환하면 수정 불가 상태로 고정
final_packet = bytes(dynamic_buf)
print(f"최종 패킷: {final_packet}")
```

**코드 해설**

조각 수신 중에는 bytearray로 데이터를 조립하고, 수신이 완료되면 bytes()로 변환하여 이후 처리에서 내용이 바뀌지 않도록 고정하는 패턴이 실무에서 자주 쓰인다. bytes와 bytearray 모두 인덱스로 개별 바이트에 접근할 수 있으며, 슬라이싱도 동일하게 동작한다.

**실행 결과와 확인할 점**

fixed_buf[0]은 정수 1이 출력된다. bytearray에 조각을 붙이는 과정과 최종 bytes 변환 결과를 비교하여 두 타입의 차이를 확인한다.

### 내장 struct 모듈

Python 변수(정수, 실수 등)를 하드웨어나 C/C++ 시스템 규격에 맞는 고정 바이트 데이터 구조체로 인코딩(pack) 하고 디코딩(unpack) 한다.

```python
import struct

# 패킷 규격: 장비ID(2바이트 정수 'H'), 센서값(4바이트 실수 'f') -> 총 6바이트
packet_format = "<Hf"

device_id = 101
sensor_value = 36.5

# Python 데이터를 바이너리 스트림으로 직렬화 (패킹)
tx_packet = struct.pack(packet_format, device_id, sensor_value)
print(f"송신 패킷 (바이너리): {tx_packet}, 크기: {len(tx_packet)}바이트")

# 수신한 바이트 스트림을 Python 데이터로 역직렬화 (언패킹)
rx_device_id, rx_sensor_value = struct.unpack(packet_format, tx_packet)
print(f"복원 -> 장비ID: {rx_device_id}, 센서값: {rx_sensor_value:.1f}")
```

**코드 해설**

포맷 문자열 "<Hf"에서 <는 리틀 엔디안(Little-Endian), H는 2바이트 부호 없는 정수, f는 4바이트 부동소수점을 뜻한다. C 구조체와 1대 1로 대응되므로, 임베디드 장치와 정확한 바이너리 통신을 할 수 있다. struct.calcsize(packet_format)으로 크기를 미리 확인할 수 있다.

**실행 결과와 확인할 점**

tx_packet이 6바이트짜리 bytes 객체로 출력되고, 언패킹 후 복원한 값이 원본과 같아야 한다. 부동소수점 특성상 :.1f로 반올림해서 확인한다.

### memoryview를 이용한 제로 카피(Zero-Copy)

바이트 버퍼를 잘라낼(Slicing) 때 새로운 복사본 메모리를 할당하지 않고, 원본 메모리의 특정 주소 범위를 그대로 투영(Window)하여 대규모 자원 이동 시 성능 저하를 방지한다.

```python
import struct

PACKET_SIZE = 6  # struct "<Hf" 기준 6바이트
packet_count = 1000

# 테스트용 버퍼 생성
raw_buffer = bytearray()
for i in range(packet_count):
    raw_buffer.extend(struct.pack("<Hf", i, i * 0.5))

# 일반 슬라이싱: 매번 새로운 bytes 복사본이 생성됨
target_idx = 500
normal_slice = bytes(raw_buffer[target_idx * PACKET_SIZE:(target_idx + 1) * PACKET_SIZE])

# memoryview: 원본 메모리를 복사 없이 가리키는 창(Window)
mv = memoryview(raw_buffer)
mv_slice = mv[target_idx * PACKET_SIZE:(target_idx + 1) * PACKET_SIZE]

dev_id, sensor_val = struct.unpack("<Hf", mv_slice)
print(f"패킷 {target_idx}: 장비ID={dev_id}, 값={sensor_val:.1f}")
```

**코드 해설**

raw_buffer[a:b]는 해당 범위의 데이터를 새 bytes 객체로 복사한다. 버퍼가 수백 MB라면 슬라이싱만으로 메모리가 두 배가 된다. memoryview(raw_buffer)로 감싸면 mv[a:b]는 복사 없이 원본 버퍼의 a번째~b번째를 가리키는 뷰 객체만 만든다. struct.unpack은 이 뷰를 직접 읽어 처리하므로 메모리 낭비가 없다.

**실행 결과와 확인할 점**

패킷 500번의 장비ID가 500, 값이 250.0으로 출력되면 정확히 동작하는 것이다.

---

## 모듈 시스템과 실행 진입점 통제 아키텍처

> **풀고 싶은 문제**
> 유틸리티 모듈을 import하는 순간 테스트 코드까지 실행되어 버렸다. 모듈을 가져올 때는 함수만 불러오고, 직접 실행할 때만 코드가 동작하게 만들 수 없을까?

### import module vs from module import object

import module은 모듈 전체를 가져오며, 내부 객체를 쓸 때 반드시 module.object처럼 네임스페이스를 명시해야 한다. 코드가 명확해지고 현재 파일의 다른 전역 변수와 이름이 충돌할 위험이 없어 실무에서 권장된다.

from module import object는 모듈 내 특정 객체만 현재 파일의 네임스페이스로 직접 불러온다. module. 생략이 가능해 코드가 간결해지지만, 현재 파일에 동일한 이름의 변수나 함수가 있으면 덮어써지는 문제가 생긴다.

from module import *는 모듈 내 객체를 명시 없이 전부 들여오므로 네임스페이스를 오염시킨다. 사용을 금지한다.

```python
import math
from math import sqrt, pi

# import math 방식: 어디서 온 함수인지 명확
area1 = math.pi * math.pow(5, 2)
print(f"import math 방식: {area1:.2f}")

# from math import 방식: 코드가 간결하지만 이름 충돌 위험 있음
area2 = pi * pow(5, 2)
hyp = sqrt(3**2 + 4**2)
print(f"from import 방식: area={area2:.2f}, 빗변={hyp:.1f}")

# 이름 충돌 예시
sqrt = "덮어쓴 문자열"
try:
    print(sqrt(9))  # TypeError 발생
except TypeError as e:
    print(f"충돌 에러: {e}")
```

**코드 해설**

sqrt = "덮어쓴 문자열"을 대입하는 순간 from math import sqrt로 불러온 함수가 지워진다. import math 방식이었다면 math.sqrt는 여전히 살아있다. 프로젝트 규모가 커질수록 이름 충돌 위험도 커지므로, 공통 유틸리티 모듈은 import module 방식을 기본으로 쓴다.

**실행 결과와 확인할 점**

area1과 area2의 계산 결과가 같음을 확인한다. sqrt를 문자열로 덮어쓴 뒤 TypeError가 발생하는 것을 보고, math.sqrt는 영향받지 않음을 비교한다.

### 실행 환경 식별자 __name__ 과 진입점 격리 패턴

Python 파일이 실행될 때 인터프리터는 특수 전역 변수인 __name__을 자동으로 생성하고 값을 할당한다.

직접 실행한 파일(예: python main.py) 내부에서 __name__은 항상 "__main__"이 된다. 다른 파일에 의해 import될 때는 해당 파일의 모듈 이름(파일명에서 .py를 뺀 것)이 들어간다.

if __name__ == "__main__": 조건 분기를 적용하면 파일이 독립 실행될 때만 메인 로직이 실행되도록 차단벽을 세울 수 있다. 이 격리벽이 없으면 외부에서 특정 함수만 import하려 했을 뿐인데 모듈 내부의 전역 코드가 모두 자동 실행되는 오작동이 생긴다.

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

print(f"현재 __name__ = '{__name__}'")

if __name__ == "__main__":
    # 이 블록은 파일을 직접 실행할 때만 동작
    print("직접 실행 모드: 내부 테스트를 수행합니다.")
    print(f"add(3, 5) = {add(3, 5)}")
    print(f"subtract(10, 4) = {subtract(10, 4)}")
else:
    print(f"모듈 import 모드: '{__name__}' 모듈이 로드되었습니다.")
```

**코드 해설**

파일을 직접 실행하면 __name__이 "__main__"이므로 if 블록 안의 테스트가 실행된다. 다른 파일에서 import하면 __name__에 모듈 이름이 들어가 if 조건이 거짓이 되므로 테스트 코드는 실행되지 않는다. 이 패턴 덕분에 하나의 파일이 라이브러리와 독립 실행 스크립트 두 역할을 동시에 수행할 수 있다.

**실행 결과와 확인할 점**

파일을 직접 실행하면 직접 실행 모드: 내부 테스트를 수행합니다.가 나온다. 같은 코드를 다른 스크립트에서 import하면 모듈 import 모드: 가 나오고 테스트 코드는 실행되지 않는 것을 확인한다.

---

## 파일 I/O와 데이터 직렬화

> **풀고 싶은 문제**
> 장비 설정값을 하드코딩하지 않고 JSON 파일에서 읽고 싶다. 센서 로그도 CSV로 저장해서 나중에 분석하고 싶다. 파이썬에서 파일을 안전하게 읽고 쓰는 표준 방법은 무엇일까?

### pathlib을 이용한 파일 경로 관리

os.path 대신 pathlib.Path를 사용하면 파일 경로 조작이 객체지향 방식으로 가능해진다. 플랫폼(Windows/Linux) 차이를 내부적으로 처리하므로 이식성이 높다.

```python
from pathlib import Path

base_dir = Path("data")
config_file = base_dir / "config.json"  # / 연산자로 경로 조합

print(f"경로: {config_file}")
print(f"부모 디렉터리: {config_file.parent}")
print(f"파일 확장자: {config_file.suffix}")
print(f"존재 여부: {config_file.exists()}")

# 디렉터리 자동 생성 (중간 경로 포함, 이미 있어도 에러 없음)
base_dir.mkdir(parents=True, exist_ok=True)
print(f"data 디렉터리 생성 완료: {base_dir.exists()}")
```

**코드 해설**

Path 객체에 / 연산자를 쓰면 OS에 맞는 구분자(\\, /)를 자동으로 처리한다. mkdir(parents=True, exist_ok=True)는 중간 디렉터리도 함께 만들며 이미 있으면 에러를 내지 않는다. 파일 경로를 문자열로 직접 연결하는 방식은 OS 간 이식성 문제를 일으킬 수 있다.

**실행 결과와 확인할 점**

config_file.exists()가 False인 상태에서 mkdir 실행 후 base_dir.exists()가 True가 되는 것을 확인한다. config_file.parent, config_file.suffix가 각각 data와 .json을 반환하는 것도 확인한다.

### JSON 파일 읽기/쓰기

json 모듈로 딕셔너리와 리스트를 JSON 형식으로 직렬화(dump)하고 역직렬화(load)한다. MQTT 페이로드, REST API 응답, 장비 설정 파일 처리에 가장 많이 쓰인다.

```python
import json
from pathlib import Path

config = {
    "device_id": 42,
    "server": "192.168.1.100",
    "port": 1883,
    "sensors": ["temp", "humidity", "pressure"],
    "thresholds": {"temp_max": 40.0, "humidity_min": 20.0}
}

config_path = Path("data/device_config.json")
Path("data").mkdir(exist_ok=True)

# 쓰기: indent로 가독성 있게 저장
with open(config_path, "w", encoding="utf-8") as f:
    json.dump(config, f, indent=2, ensure_ascii=False)
print(f"설정 파일 저장 완료: {config_path}")

# 읽기: 파일에서 Python 딕셔너리로 복원
with open(config_path, "r", encoding="utf-8") as f:
    loaded = json.load(f)

print(f"장비 ID: {loaded['device_id']}")
print(f"온도 최대값: {loaded['thresholds']['temp_max']}°C")
print(f"센서 목록: {loaded['sensors']}")
print(f"원본과 동일: {loaded == config}")
```

**코드 해설**

indent=2는 들여쓰기 두 칸으로 사람이 읽기 좋은 형식을 만든다. ensure_ascii=False는 한글 같은 비 ASCII 문자가 \uXXXX 이스케이프 없이 그대로 저장되도록 한다. json.loads()/json.dumps()는 파일이 아닌 문자열을 직접 파싱/직렬화할 때 쓴다. 네트워크로 수신한 MQTT 페이로드 처리에 유용하다.

**실행 결과와 확인할 점**

data/device_config.json 파일을 텍스트 편집기로 열어 들여쓰기가 적용된 JSON이 저장되어 있는지 확인한다. loaded == config가 True로 출력되어 저장과 복원이 완전히 대칭적임을 확인한다.

### CSV 파일 읽기/쓰기

센서 로그, 측정 기록처럼 행/열 구조의 데이터는 csv 모듈로 처리한다.

```python
import csv
from pathlib import Path

log_path = Path("data/sensor_log.csv")
Path("data").mkdir(exist_ok=True)

rows = [
    {"time": "10:00", "device_id": 1, "temp": 36.5},
    {"time": "10:01", "device_id": 1, "temp": 37.1},
    {"time": "10:02", "device_id": 2, "temp": 35.8},
]

# 쓰기
with open(log_path, "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["time", "device_id", "temp"])
    writer.writeheader()
    writer.writerows(rows)

# 읽기
with open(log_path, "r", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(f"[{row['time']}] 장비 {row['device_id']}: {row['temp']}°C")
```

**코드 해설**

DictWriter/DictReader는 헤더 이름과 딕셔너리 키를 자동으로 매핑한다. newline=""을 지정하지 않으면 Windows에서 빈 줄이 섞여 들어가는 버그가 생긴다. 대규모 CSV는 pandas를 쓰지만, 의존성을 최소화해야 하는 임베디드 환경이나 간단한 로그 파일에는 표준 라이브러리의 csv 모듈이 적합하다.

**실행 결과와 확인할 점**

세 행이 시간 순서대로 출력되어야 한다. 저장된 CSV 파일을 텍스트 편집기로 열어 헤더 행(time,device_id,temp)이 첫 줄에 있는지 확인한다.

---

## 정밀 예외 처리 및 에러 파이프라인

> **풀고 싶은 문제**
> 외부 서버 연결 코드가 오류를 내면 프로그램 전체가 꺼져 버린다. 에러 종류에 따라 다르게 대응하면서도 파일이나 소켓 자원이 새지 않게 하려면 어떻게 해야 할까?

### 구체적인 예외 분기 구조 (try - except - else - finally)

except Exception: 같이 뭉뚱그린 처리를 피하고, ConnectionError, ValueError처럼 발생 가능한 하위 예외를 구체적으로 선언해야 정밀한 대처가 가능하다.

else 구문은 예외가 하나도 없을 때만 실행할 안전 영역을 분리하며, finally는 에러 발생 여부와 상관없이 자원을 반환하는 최종 블록이다.

### 예외 되던지기 (raise) 패턴

에러를 잡은 뒤 현재 단계에서 부분적인 조치(자원 정리, 부분 로깅)를 취한 다음, 원래 에러의 성격을 유지한 채 상위 함수로 다시 던지는 기법이다.

```python
def connect_database():
    print("데이터베이스 연결 시도 중...")
    raise ConnectionRefusedError("서버 포트가 닫혀 있습니다.")

def database_service_layer():
    try:
        connect_database()
    except ConnectionRefusedError as e:
        print(f"내부 조치: 연결 실패 로그를 파일에 기록합니다. 사유: {e}")
        raise  # 1차 조치 완료 후 상위로 되던짐
    except Exception as e:
        print(f"예상치 못한 기타 에러: {e}")
    else:
        print("DB 연결에 성공하여 쿼리를 수행합니다.")
    finally:
        print("DB 연산 블록 프로세스 안전 종료.")

try:
    database_service_layer()
except ConnectionRefusedError:
    print("메인 시스템 감지: DB 서버 다운 경고 알람을 관리자에게 발송합니다.")
```

**코드 해설**

database_service_layer는 ConnectionRefusedError를 잡아 1차 처리(로그 기록)를 한 뒤 raise로 다시 위로 던진다. finally 블록은 예외가 발생해도, 되던져져도, 정상 종료되어도 반드시 실행된다. 자원 정리(소켓 닫기, 파일 닫기 등)는 항상 finally에 두어야 빠짐없이 실행됨을 보장한다.

**실행 결과와 확인할 점**

출력 순서는 연결 시도 -> 내부 조치 -> finally 종료 -> 메인 시스템 감지 순이어야 한다. connect_database에서 raise를 제거하면 else가 실행되는 것도 확인해 본다.

### 커스텀 예외 클래스 계층

도메인 전용 예외 계층을 정의하면 상위 코드에서 카테고리별로 정밀하게 대응할 수 있고, 에러 발생 원인이 명확해진다. 기본 원칙은 프로젝트 루트 예외를 하나 만들고 그 아래에 세부 예외를 정의하는 트리 구조이다.

```python
# 프로젝트 루트 예외
class SensorError(Exception):
    pass

# 세부 예외 계층
class SensorTimeoutError(SensorError):
    def __init__(self, sensor_id: int, timeout_sec: float):
        self.sensor_id = sensor_id
        self.timeout_sec = timeout_sec
        super().__init__(
            f"센서 {sensor_id}가 {timeout_sec}초 내에 응답하지 않음"
        )

class SensorValueError(SensorError):
    def __init__(self, sensor_id: int, value: float):
        self.sensor_id = sensor_id
        self.value = value
        super().__init__(
            f"센서 {sensor_id}의 측정값 {value}가 허용 범위를 벗어남"
        )

def read_sensor(sensor_id: int, value) -> float:
    if value is None:
        raise SensorTimeoutError(sensor_id, 1.0)
    if not (-40.0 <= value <= 125.0):
        raise SensorValueError(sensor_id, value)
    return value

# 카테고리별 분기: 세부 예외 먼저, 루트 예외는 마지막
for sid, val in [(1, 36.5), (2, 200.0), (3, None)]:
    try:
        result = read_sensor(sid, val)
        print(f"센서 {sid}: {result}°C")
    except SensorTimeoutError as e:
        print(f"[타임아웃] {e}")
    except SensorValueError as e:
        print(f"[범위 초과] {e}")
    except SensorError as e:
        print(f"[일반 센서 오류] {e}")
```

**코드 해설**

SensorError를 상속하는 두 예외는 각각 독립적으로 잡힐 수도 있고, except SensorError 하나로 모두 잡힐 수도 있다. 이 계층 덕분에 상위 코드는 세부 원인을 알 필요 없이 SensorError 하나로 센서 관련 에러 전체를 처리할 수 있다. __init__에 sensor_id, value 같은 속성을 저장하면 except 블록에서 e.sensor_id로 디버깅 정보를 바로 활용할 수 있다.

**실행 결과와 확인할 점**

세 번의 루프에서 정상 -> 범위 초과 -> 타임아웃 순으로 각각 다른 except가 실행되어야 한다. except SensorTimeoutError와 except SensorValueError를 except SensorError 뒤로 옮기면 세부 분기가 실행되지 않는 이유를 확인한다.

---

## 로깅 시스템과 디버깅 파이프라인

> **풀고 싶은 문제**
> print로 디버깅하다 보니 배포 후에도 출력이 남아 있고, 에러가 발생한 시간이나 어느 모듈에서 나왔는지를 알 수 없다. 레벨별로 출력을 조절하고 파일에도 남기는 표준 방법이 필요하다.

### logging 모듈 기본 설정

Python 표준 logging 모듈은 DEBUG < INFO < WARNING < ERROR < CRITICAL 다섯 단계 레벨을 제공한다. 지정한 레벨 이상의 로그만 출력되므로, 개발 시에는 DEBUG, 배포 시에는 WARNING으로 전환하여 출력 범위를 손쉽게 제어한다.

```python
import logging

# 기본 설정: 레벨, 형식, 핸들러 동시 지정
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%H:%M:%S",
    handlers=[
        logging.StreamHandler(),                              # 콘솔 출력
        logging.FileHandler("app.log", encoding="utf-8"),    # 파일 저장
    ]
)

logger = logging.getLogger("sensor.reader")  # 모듈 이름을 계층으로 표현

logger.debug("센서 초기화 시작 (개발 시에만 출력)")
logger.info("센서 연결 성공")
logger.warning("배터리 잔량 15% 이하")
logger.error("센서 응답 없음 — 재시도 예정")

try:
    raise ConnectionRefusedError("포트 닫힘")
except ConnectionRefusedError:
    logger.exception("연결 실패")  # 스택 트레이스 자동 첨부
```

**코드 해설**

logging.getLogger("sensor.reader")는 점(.)으로 구분된 계층 이름을 가진 로거를 생성한다. sensor 아래의 모든 로거 설정을 한 번에 제어할 수 있다. logger.exception()은 ERROR 레벨로 메시지를 남기면서 현재 예외의 스택 트레이스를 자동으로 추가한다. print와 달리 출력 시각, 레벨, 모듈명이 자동으로 붙는다.

**실행 결과와 확인할 점**

콘솔과 app.log 파일에 동일한 내용이 출력되어야 한다. level=logging.WARNING으로 바꾸면 debug와 info 메시지가 사라지는 것을 확인한다. app.log를 열어 logger.exception()이 스택 트레이스를 첨부한 것도 확인한다.

### 모듈별 로거 계층

실무에서는 모듈마다 독립 로거를 만들고 상위 로거에서 레벨과 핸들러를 제어하는 계층 구조를 사용한다. 각 모듈 파일 상단에 logger = logging.getLogger(__name__)을 쓰면 파일명이 자동으로 로거 이름이 되어 별도 관리가 필요 없다.

```python
import logging

# 상위 로거: 전체 설정 담당
root_logger = logging.getLogger("myapp")
root_logger.setLevel(logging.DEBUG)
handler = logging.StreamHandler()
handler.setFormatter(logging.Formatter("[%(levelname)s] %(name)s: %(message)s"))
root_logger.addHandler(handler)

# 하위 로거: 모듈명으로 계층 표현 (상위 핸들러 상속)
comm_logger = logging.getLogger("myapp.comm")
sensor_logger = logging.getLogger("myapp.sensor")

comm_logger.info("TCP 소켓 연결")
sensor_logger.debug("온도 센서 ID=5 폴링")
sensor_logger.warning("측정값 이상 — 재측정")
```

**코드 해설**

"myapp.comm"과 "myapp.sensor"는 자동으로 "myapp" 로거의 자식이 된다. root_logger에 핸들러를 하나만 추가해도 하위 로거들이 모두 그 핸들러를 상속한다. 각 모듈에서 logging.getLogger(__name__)을 쓰면 패키지 구조(myapp.comm 등)가 로거 이름에 그대로 반영된다.

**실행 결과와 확인할 점**

세 줄의 로그가 모두 출력되며, 각각 모듈 이름(myapp.comm, myapp.sensor)이 표시되어야 한다. root_logger.setLevel(logging.WARNING)으로 바꾸면 info와 debug 메시지가 사라지고 warning만 남는 것을 확인한다.

---

## 객체지향 아키텍처 및 고급 클래스 기법

> **풀고 싶은 문제**
> 여러 종류의 통신 드라이버를 동일한 방식으로 교체해서 쓰고 싶다. 그런데 특정 메서드를 빠뜨려도 실행 전까지는 알 수가 없다. 클래스 설계 단계에서 이 문제를 미리 막을 수 있을까?

### 인스턴스 메서드 vs 클래스 메서드 vs 정적 메서드

인스턴스 메서드(self)는 생성된 객체의 개별 인스턴스 변수에 접근하고 변경하는 기본 메서드이다.

클래스 메서드(@classmethod, cls)는 클래스 자체를 인자로 받으며, 주로 대체 생성자(Factory Pattern)를 구현하여 다양한 형태의 입력 데이터를 객체화할 때 사용한다.

정적 메서드(@staticmethod)는 인스턴스(self)나 클래스(cls)의 변수 영역에 전혀 관여하지 않고, 유틸리티 성격의 순수 함수를 클래스 네임스페이스 아래에 묶어 둘 때 활용한다.

```python
import struct

class SensorReading:
    UNIT = "°C"

    def __init__(self, device_id, value):
        self.device_id = device_id
        self.value = value

    def report(self):
        print(f"[장비 {self.device_id}] {self.value}{SensorReading.UNIT}")

    @classmethod
    def from_raw_bytes(cls, raw):
        device_id, value = struct.unpack("<Hf", raw)
        return cls(device_id, value)

    @staticmethod
    def is_valid(value):
        return -40.0 <= value <= 125.0

reading1 = SensorReading(1, 36.5)
reading1.report()

raw_packet = struct.pack("<Hf", 2, 37.8)
reading2 = SensorReading.from_raw_bytes(raw_packet)
reading2.report()

print(f"37.8 유효 여부: {SensorReading.is_valid(37.8)}")
print(f"200.0 유효 여부: {SensorReading.is_valid(200.0)}")
```

**코드 해설**

from_raw_bytes는 struct.unpack으로 바이트 패킷을 분해하여 클래스 인스턴스를 반환한다. cls(device_id, value)는 SensorReading(device_id, value)와 동일하지만, 상속 시에도 자식 클래스의 생성자를 호출하므로 클래스 메서드를 쓰는 것이 더 유연하다. is_valid는 SensorReading과 논리적으로 연관되지만 인스턴스 데이터가 필요 없으므로 staticmethod로 묶는다.

**실행 결과와 확인할 점**

reading1과 reading2 모두 report()를 호출할 수 있으며, 각각 36.5와 37.8을 출력해야 한다. is_valid(200.0)는 False가 나와야 한다.

### 클래스 상속과 추상 클래스 (ABC)

추상 클래스(Abstract Base Class, ABC)를 상속받아 자식 클래스들이 특정 메서드를 강제로 구현하도록 만든다. 이를 통해 인터페이스를 통일하고 다형성(Polymorphism)을 확보한다.

```python
from abc import ABC, abstractmethod

class CommunicationDriver(ABC):
    @abstractmethod
    def connect(self, address: str) -> bool:
        pass

    @abstractmethod
    def send(self, data: bytes) -> int:
        pass

    @abstractmethod
    def disconnect(self) -> None:
        pass

class SerialDriver(CommunicationDriver):
    def connect(self, address):
        print(f"[Serial] {address} 포트에 연결")
        return True

    def send(self, data):
        print(f"[Serial] {len(data)} 바이트 전송")
        return len(data)

    def disconnect(self):
        print("[Serial] 포트 닫음")

class TcpDriver(CommunicationDriver):
    def connect(self, address):
        print(f"[TCP] {address} 에 소켓 연결")
        return True

    def send(self, data):
        print(f"[TCP] {len(data)} 바이트 전송")
        return len(data)

    def disconnect(self):
        print("[TCP] 소켓 닫음")

def run_communication(driver: CommunicationDriver, address: str):
    driver.connect(address)
    driver.send(b"\x01\x02\x03")
    driver.disconnect()

run_communication(SerialDriver(), "COM3")
run_communication(TcpDriver(), "192.168.1.10:8080")
```

**코드 해설**

@abstractmethod로 선언한 메서드를 자식 클래스가 하나라도 빠뜨리면, 객체를 생성하는 시점에 바로 TypeError가 발생한다. 실행 전에 누락을 감지할 수 있다. run_communication은 driver가 SerialDriver든 TcpDriver든 같은 인터페이스로 호출하므로, 새로운 드라이버를 추가해도 이 함수는 수정할 필요가 없다.

**실행 결과와 확인할 점**

SerialDriver와 TcpDriver 모두 동일한 세 단계(연결, 전송, 해제)를 수행한다. SerialDriver에서 send를 삭제하고 인스턴스를 생성하면 TypeError가 발생하는 것도 직접 확인해 본다.

### 메타클래스(Metaclass)를 통한 클래스 생성 제어

일반 클래스가 객체(인스턴스)를 만드는 설계도라면, 메타클래스는 클래스 자체를 만드는 설계도이다. 파이썬의 모든 클래스는 기본적으로 내장 메타클래스인 type에 의해 동적으로 생성된다.

type을 상속받아 커스텀 메타클래스를 정의하면, 자식 클래스들이 생성되는 시점을 가로채어 필수 메서드 구현 여부 검증, 네이밍 규칙 강제, 플러그인 레지스트리 자동 등록 같은 고급 아키텍처 통제가 가능해진다.

```python
class PluginValidationMeta(type):
    def __new__(cls, name, bases, attrs):
        if bases:
            if not name.endswith("Plugin"):
                raise TypeError(f"아키텍처 에러: '{name}'은 반드시 'Plugin'으로 끝나야 합니다.")
            if "description" not in attrs:
                raise AttributeError(f"아키텍처 에러: '{name}'에 'description' 속성이 누락되었습니다.")
        return super().__new__(cls, name, bases, attrs)

class BasePlugin(metaclass=PluginValidationMeta):
    pass

# 규칙 위반 예시 (주석 해제하면 스크립트 실행 즉시 에러 발생)
# class SerialDriver(BasePlugin):          # 이름 규칙 위반
#     description = "시리얼 통신"
# class SerialDriverPlugin(BasePlugin):    # description 누락
#     pass

class EthernetDriverPlugin(BasePlugin):
    description = "이더넷 고속 소켓 통신 모듈"

    def execute(self):
        print(f"플러그인 기동: {self.description}")

if __name__ == "__main__":
    plugin = EthernetDriverPlugin()
    plugin.execute()
```

**코드 해설**

__new__는 클래스 객체가 생성될 때 호출되는 메서드이다. PluginValidationMeta.__new__에서 이름 규칙과 필수 속성을 검사하므로, 규칙에 맞지 않는 클래스를 선언하는 순간 모듈 로드 시점에서 바로 에러가 난다. 런타임이 오랫동안 진행된 뒤에야 발견되는 오류를 정의 단계에서 차단하는 것이 이 기법의 핵심이다.

**실행 결과와 확인할 점**

EthernetDriverPlugin 인스턴스가 정상 생성되고 execute()가 출력된다. 주석 처리된 규칙 위반 클래스 중 하나를 활성화하면 파이썬이 파일을 읽는 즉시 TypeError 또는 AttributeError가 발생한다.

### @dataclass 데이터 컨테이너

@dataclass 데코레이터는 __init__, __repr__, __eq__를 자동 생성한다. 센서 측정값, 네트워크 패킷 헤더처럼 데이터를 담는 구조체를 정의할 때 반복적인 보일러플레이트 코드를 없애 준다.

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class SensorPacket:
    device_id: int
    temperature: float
    tags: List[str] = field(default_factory=list)  # 뮤터블 기본값 안전 처리
    is_valid: bool = True

    def is_in_range(self) -> bool:
        return -40.0 <= self.temperature <= 125.0

# __init__ 자동 생성: 필드 순서대로 인자를 받음
p1 = SensorPacket(device_id=1, temperature=36.5, tags=["zone-A"])
p2 = SensorPacket(device_id=1, temperature=36.5, tags=["zone-A"])

# __repr__ 자동 생성
print(p1)

# __eq__ 자동 생성: 필드 값이 같으면 동등 비교 True
print(f"p1 == p2: {p1 == p2}")

# 메서드는 일반 클래스와 동일하게 정의
print(f"유효 범위 여부: {p1.is_in_range()}")

# field(default_factory=list) 독립성 확인
p3 = SensorPacket(device_id=2, temperature=37.0)
p1.tags.append("new-tag")
print(f"p3.tags (독립성 확인): {p3.tags}")  # []이어야 함
```

**코드 해설**

@dataclass 없이 이 클래스를 직접 작성하면 __init__, __repr__, __eq__를 각각 작성해야 한다. field(default_factory=list)는 앞서 배운 뮤터블 기본값 안티 패턴 문제를 @dataclass 환경에서 안전하게 처리하는 방식이다. @dataclass(frozen=True)로 선언하면 인스턴스가 이뮤터블해져 수정 시 FrozenInstanceError가 발생한다.

**실행 결과와 확인할 점**

print(p1)이 SensorPacket(device_id=1, temperature=36.5, ...)처럼 읽기 좋은 형식으로 출력된다. p1.tags에 항목을 추가한 뒤 p3.tags가 여전히 []인 것을 확인하여 field(default_factory=list)의 독립성을 검증한다.

### __slots__ 인스턴스 메모리 최적화

Python 인스턴스는 기본으로 속성을 __dict__라는 딕셔너리에 저장한다. 같은 클래스의 인스턴스를 수천~수만 개 생성하면 인스턴스마다 딕셔너리 객체가 따라붙어 메모리를 낭비한다. __slots__를 선언하면 __dict__ 대신 고정 크기 배열을 사용하여 인스턴스당 메모리를 크게 줄이고 속성 접근 속도도 향상된다.

```python
import sys

class Reading:
    def __init__(self, device_id, temp):
        self.device_id = device_id
        self.temp = temp

class ReadingSlots:
    __slots__ = ("device_id", "temp")

    def __init__(self, device_id, temp):
        self.device_id = device_id
        self.temp = temp

# 인스턴스 메모리 비교
r_normal = Reading(1, 36.5)
r_slots  = ReadingSlots(1, 36.5)

normal_size = sys.getsizeof(r_normal) + sys.getsizeof(r_normal.__dict__)
print(f"기본 인스턴스:  {normal_size} bytes (인스턴스 + __dict__)")
print(f"slots 인스턴스: {sys.getsizeof(r_slots)} bytes (__dict__ 없음: {hasattr(r_slots, '__dict__')})")

# 10만 개 생성 시 전체 메모리 비교
n = 100_000
normal_mem = sum(sys.getsizeof(r) + sys.getsizeof(r.__dict__)
                 for r in [Reading(i, i * 0.1) for i in range(n)])
slots_mem  = sum(sys.getsizeof(r)
                 for r in [ReadingSlots(i, i * 0.1) for i in range(n)])
print(f"\n{n:,}개 기준: 기본={normal_mem/1024:.0f}KB, slots={slots_mem/1024:.0f}KB")
print(f"절감률: {(1 - slots_mem/normal_mem)*100:.1f}%")
```

**코드 해설**

__slots__를 선언하면 파이썬은 해당 클래스 인스턴스에서 __dict__를 완전히 제거한다. 그 대신 슬롯 이름에 대응하는 고정 배열만 유지하므로 인스턴스당 메모리가 40~60% 줄어드는 경우가 많다. 단, 선언된 이름 외에 임의 속성을 동적으로 추가할 수 없다. 센서 데이터 버퍼처럼 필드 구성이 고정된 대량 객체에 적합하다.

**실행 결과와 확인할 점**

기본 인스턴스는 __dict__ 포함 크기가 크고, slots 인스턴스는 __dict__가 없어 작은 것을 확인한다. 10만 개 생성 시 절감률이 출력된다. r_slots.new_attr = 1처럼 선언되지 않은 속성을 추가하면 AttributeError가 발생하는 것도 확인한다.

---

## 코드 안전성 확보를 위한 모던 Python

> **풀고 싶은 문제**
> 다른 사람이 만든 함수에 잘못된 타입의 값을 넘겼다가 프로그램이 오랫동안 실행되다가 런타임 오류가 났다. 실행 전에 타입 오류를 미리 잡아낼 방법은 없을까?

### 타입 힌팅 (Type Hinting)

변수와 함수의 매개변수, 반환값에 정적 타입을 명시하여 가독성을 높이고, mypy 같은 정적 검사 도구로 배포 전에 타입 버그를 필터링한다. 런타임 동작에는 영향을 주지 않지만, IDE와 협업 도구가 타입 정보를 활용해 오류를 미리 표시한다.

```python
from typing import List, Optional, Tuple

def parse_sensor_csv(
    file_path: str,
    limit: int = 100,
    skip_invalid: bool = True
) -> List[Tuple[int, float]]:
    results: List[Tuple[int, float]] = []
    sample_data = ["1,36.5", "2,abc", "3,38.1", "invalid,37.0", "5,37.8"]
    for line in sample_data[:limit]:
        parts = line.strip().split(",")
        if len(parts) != 2:
            continue
        try:
            device_id = int(parts[0])
            temperature = float(parts[1])
            results.append((device_id, temperature))
        except ValueError:
            if not skip_invalid:
                raise
    return results

def average_temperature(readings: List[Tuple[int, float]]) -> Optional[float]:
    if not readings:
        return None
    return sum(temp for _, temp in readings) / len(readings)

data = parse_sensor_csv("sensor_log.csv")
avg = average_temperature(data)
print(f"파싱된 데이터: {data}")
print(f"평균 온도: {avg:.2f}°C" if avg is not None else "데이터 없음")
```

**코드 해설**

-> List[Tuple[int, float]]는 이 함수가 (장비ID, 온도) 쌍의 리스트를 반환함을 명시한다. Optional[float]는 float이 반환될 수도 있고 None이 반환될 수도 있다는 뜻이다. mypy를 실행하면 average_temperature에 잘못된 타입의 값을 넘기는 코드를 실행 전에 감지할 수 있다.

**실행 결과와 확인할 점**

"2,abc"와 "invalid,37.0"은 skip_invalid=True에 의해 건너뛰어, [(1, 36.5), (3, 38.1), (5, 37.8)]이 파싱된다. 평균은 37.47°C 근처가 나온다. skip_invalid=False로 바꾸면 ValueError가 전파되는 것도 확인한다.

### 구조적 패턴 매칭 (match - case)

Python 3.10 이상에서 지원하며, 복잡하고 중첩된 if-elif-else 구조를 대체하여 딕셔너리 데이터 구조, 패킷 커맨드, 상태 코드에 따른 분기 로직의 가독성을 높인다.

```python
from typing import Dict, Union

def analyze_packet(packet: Dict[str, Union[str, int, list]]) -> None:
    match packet:
        case {"status": "SUCCESS", "payload": [x, y, z]}:
            print(f"데이터 정상 디코딩 -> X:{x}, Y:{y}, Z:{z}")
        case {"status": "ERROR", "code": int(err_code)}:
            print(f"장치 에러 감지! 에러 코드: {err_code}")
        case _:
            print("경고: 비정상적인 패킷입니다.")

analyze_packet({"status": "SUCCESS", "payload": [10, 20, 30]})
analyze_packet({"status": "ERROR", "code": 404})
analyze_packet({"status": "UNKNOWN"})
```

**코드 해설**

case {"status": "SUCCESS", "payload": [x, y, z]}는 딕셔너리 패턴과 시퀀스 패턴을 동시에 매칭한다. payload 리스트의 세 요소가 x, y, z로 자동 추출된다. case _:는 모든 패턴에 해당하지 않을 때 실행되는 기본 분기이다.

**실행 결과와 확인할 점**

각 호출이 세 가지 case를 순서대로 매칭한다. {"status": "SUCCESS", "payload": [10, 20]} (요소가 두 개)를 넘기면 payload 패턴이 불일치하여 case _:로 빠지는 것도 확인한다.

---

## 고동시성 멀티태스킹 아키텍처 및 자원 동기화

> **풀고 싶은 문제**
> 파일을 내려받으면서 화면도 갱신하고 동시에 센서도 읽어야 하는데, 프로그램이 한 번에 한 가지 일만 한다. 여러 작업을 동시에 처리하는 방법은 무엇이고, 어떤 방식을 언제 써야 할까?

### GIL(Global Interpreter Lock)과 태스크 최적화

멀티스레딩(threading)은 GIL 제한으로 하나의 코어에서 시분할 구동된다. 소켓 수신 대기 같은 I/O 집약적 작업에는 자원 오버헤드가 적어 효율적이지만, CPU 연산 중심 작업에는 성능 개선이 없다.

멀티프로세싱(multiprocessing)은 Python 프로세스 자체를 독립적으로 복사하여 멀티코어를 완전히 병렬로 구동한다. 이미지 처리, 수치 행렬 연산 등 CPU 집약적 고부하 연산에 적합하다.

비동기 프로그래밍(asyncio)은 단일 스레드 내에서 await 키워드를 통해 대규모 네트워크 I/O 동시성을 효율적으로 스케줄링하는 모던 백엔드 핵심 기법이다.

```python
import asyncio
import time

async def fetch_sensor(sensor_id: int, delay: float) -> float:
    print(f"[센서 {sensor_id}] 요청 시작...")
    await asyncio.sleep(delay)  # 실제 네트워크 대기를 모사 (CPU 점유 없이 양보)
    value = 36.0 + sensor_id * 0.3
    print(f"[센서 {sensor_id}] 수신 완료: {value:.1f}°C")
    return value

async def main():
    start = time.time()
    # gather로 동시에 시작하면 가장 긴 대기(1.0초)만 소요
    results = await asyncio.gather(
        fetch_sensor(1, 1.0),
        fetch_sensor(2, 0.8),
        fetch_sensor(3, 0.5),
    )
    elapsed = time.time() - start
    print(f"\n전체 소요 시간: {elapsed:.2f}초 (순차라면 ~2.3초)")
    print(f"수집 결과: {[f'{v:.1f}°C' for v in results]}")

asyncio.run(main())
```

**코드 해설**

await asyncio.sleep(delay)는 이 코루틴이 delay초 동안 다른 코루틴에게 실행권을 양보한다. asyncio.gather는 여러 코루틴을 동시에 시작하고 모두 완료될 때까지 기다린다. 세 센서 요청이 겹쳐서 실행되므로 전체 대기 시간은 가장 긴 1.0초 내외가 된다.

**실행 결과와 확인할 점**

세 센서의 요청이 거의 동시에 시작되고, 0.5초짜리 센서 3이 먼저 완료된 뒤 나머지가 순차적으로 완료된다. 전체 소요 시간이 1.0초 내외임을 확인한다. asyncio.gather 대신 세 번의 await를 순서대로 쓰면 2.3초가 걸리는 것도 비교해 본다.

### 자원 동기화 및 스레드 안전(Thread-Safe) 구조

뮤텍스 락(threading.Lock)은 공유 변수나 임계 영역에 여러 스레드가 동시에 접근하여 발생하는 데이터 오염(Race Condition)을 방지한다. with lock: 패턴으로 사용한다.

스레드 안전한 큐(queue.Queue)는 생산자-소비자 아키텍처를 구현할 때 내부 락 처리가 이미 내장되어 있으므로, 직접 락을 다루지 않고도 멀티스레드 간 데이터를 안전하게 교환하는 표준 통로이다.

```python
import threading
import queue
import time

data_queue = queue.Queue(maxsize=10)

def packet_collector():
    for i in range(5):
        time.sleep(0.1)
        data = f"센서 데이터 패킷_{i}"
        data_queue.put(data)
        print(f"[수집 스레드] {data} 수집 완료")

def packet_analyzer():
    while True:
        try:
            data = data_queue.get(timeout=1.0)
            print(f"[분석 스레드] {data} 처리 중...")
            data_queue.task_done()
        except queue.Empty:
            print("[분석 스레드] 더 이상 데이터가 없어 종료합니다.")
            break

collector = threading.Thread(target=packet_collector)
analyzer = threading.Thread(target=packet_analyzer)

collector.start()
analyzer.start()

collector.join()
analyzer.join()
```

**코드 해설**

queue.Queue는 내부적으로 Lock을 사용하여 put()과 get()을 스레드 안전하게 처리한다. maxsize=10으로 설정하면 큐가 꽉 찼을 때 put()이 블록(대기)하므로 수집 속도와 분석 속도가 자동으로 조율된다. task_done()은 소비자가 처리를 완료했음을 큐에 알리는 신호이다.

**실행 결과와 확인할 점**

수집과 분석 메시지가 번갈아 출력되어야 한다. maxsize를 1로 줄이면 수집 스레드가 자주 대기하는 상황을 관찰할 수 있다.

### 멀티프로세싱과 Pool 병렬 연산

threading은 GIL 때문에 CPU 연산 작업에서 성능 향상이 없다. multiprocessing.Pool은 Python 프로세스 자체를 복수로 생성하여 각각 독립된 GIL에서 실행되므로, 멀티코어 CPU의 물리적 병렬성을 완전히 활용한다. Pool.map(func, iterable)은 iterable의 각 요소를 워커 프로세스들에 분산하고 결과를 리스트로 반환한다.

```python
import multiprocessing
import time

def compute_heavy(n: int) -> int:
    """CPU 집약 연산 모사: n 이하 소수 개수 계산"""
    count = 0
    for i in range(2, n):
        if all(i % d != 0 for d in range(2, int(i**0.5) + 1)):
            count += 1
    return count

if __name__ == "__main__":
    tasks = [30_000, 40_000, 50_000, 60_000]  # 각 코어에 배분할 작업

    # 순차 실행
    start = time.time()
    results_seq = [compute_heavy(n) for n in tasks]
    print(f"순차 실행: {time.time()-start:.2f}초, 결과={results_seq}")

    # Pool 병렬 실행: CPU 코어 수만큼 워커 생성
    start = time.time()
    with multiprocessing.Pool() as pool:
        results_par = pool.map(compute_heavy, tasks)
    print(f"Pool 병렬: {time.time()-start:.2f}초, 결과={results_par}")
```

**코드 해설**

if __name__ == "__main__": 가드는 멀티프로세싱 모듈이 새 프로세스를 생성할 때 부모 코드를 재실행하지 않도록 필수로 붙여야 한다. 이를 생략하면 무한 프로세스 생성이 일어난다. with multiprocessing.Pool() as pool:은 기본으로 CPU 코어 수만큼 워커를 생성하고, Pool.map()은 tasks 목록을 워커들에게 자동으로 분배한다. 순차 실행 대비 코어 수에 비례하여 시간이 단축된다.

스레드와 프로세스는 공유 메모리 방식이 다르다. 스레드는 같은 프로세스 메모리를 공유하므로 Queue와 Lock으로 동기화해야 한다. 프로세스는 메모리가 독립되므로 데이터를 공유하려면 multiprocessing.Queue나 multiprocessing.Value를 사용해야 한다. 이미지 처리, 배치 센서 데이터 변환처럼 GIL 병목이 명확한 CPU 집약 작업에 적합하다.

**실행 결과와 확인할 점**

Pool 버전의 소요 시간이 순차 버전 대비 코어 수에 가깝게 단축되어야 한다. multiprocessing.Pool(processes=2)처럼 워커 수를 제한하면 속도 차이를 제어할 수 있다. 작업량이 너무 작으면 프로세스 생성 오버헤드가 역효과를 낼 수 있으니 tasks의 값을 절반으로 줄여서 확인한다.

> **알아두기 — concurrent.futures.ProcessPoolExecutor**
> Python 3.2부터 추가된 concurrent.futures 모듈은 ProcessPoolExecutor와 ThreadPoolExecutor를 동일한 인터페이스로 제공한다. with ProcessPoolExecutor() as exe: 안에서 exe.map(func, tasks) 또는 exe.submit(func, arg)로 사용하며, CPU 바운드와 I/O 바운드 작업을 같은 코드 구조로 전환할 수 있다.

---

## 컬렉션 고급 자료구조

> **풀고 싶은 문제**
> 리스트의 앞뒤로 데이터를 넣고 빼야 하는데 list.insert(0, x)를 쓰니 너무 느리다. 장치별 이벤트를 자동으로 분류하는 딕셔너리도 만들고 싶다. Python 표준 라이브러리에 더 나은 자료구조가 있을까?

### deque — 양방향 큐와 링 버퍼

리스트의 pop(0)과 insert(0, x)는 O(n) 연산이다. collections.deque는 양 끝 삽입/삭제가 O(1)이며, maxlen을 설정하면 오래된 데이터가 자동으로 밀려나는 링 버퍼(Ring Buffer)로 동작한다.

```python
from collections import deque

# 양방향 큐: 리스트보다 빠른 앞뒤 삽입/삭제
packet_queue = deque()
packet_queue.append("패킷_A")        # 오른쪽 추가
packet_queue.append("패킷_B")
packet_queue.appendleft("패킷_긴급")  # 왼쪽 추가 (우선순위 삽입)
print(f"큐 상태: {list(packet_queue)}")
print(f"왼쪽 꺼내기: {packet_queue.popleft()}")

# 링 버퍼: maxlen 초과 시 반대쪽 데이터가 자동 제거
sensor_window = deque(maxlen=5)  # 최근 5개 측정값만 유지
for val in [36.5, 37.0, 38.1, 37.5, 36.9, 39.2]:
    sensor_window.append(val)
    print(f"  현재 윈도우: {list(sensor_window)}")
```

**코드 해설**

deque(maxlen=5)는 슬라이딩 윈도우(이동 평균, 최근 N개 저장)를 구현할 때 매우 편리하다. 6번째 값이 들어오면 가장 오래된 값이 자동으로 빠져나간다. 임베디드 환경에서 센서 데이터의 최근 N개를 유지하며 이동 평균을 계산하는 패턴에 자주 쓰인다.

**실행 결과와 확인할 점**

링 버퍼에 6번째 값 39.2가 들어갈 때 첫 번째 값 36.5가 사라지고 윈도우가 5개를 유지하는 것을 확인한다. appendleft로 삽입한 패킷_긴급이 popleft로 즉시 꺼내지는 것도 확인한다.

> **한 걸음 더**
> deque(maxlen=5)로 최근 5개 측정값의 이동 평균을 계산하는 코드를 작성해 본다. sum(sensor_window) / len(sensor_window)를 각 step마다 출력한다.

### defaultdict와 Counter

defaultdict는 키가 없을 때 KeyError 대신 지정한 기본값을 자동 생성한다. 장치별·카테고리별 데이터 분류에서 초기화 코드를 없애 준다. Counter는 요소 빈도 집계에 특화된 딕셔너리이다.

```python
from collections import defaultdict, Counter

# defaultdict: 장치별 이벤트 자동 분류
events = [(1, "연결"), (2, "오류"), (1, "데이터"), (3, "연결"), (2, "연결")]
device_log = defaultdict(list)  # 키가 없으면 빈 리스트를 자동 생성
for device_id, event in events:
    device_log[device_id].append(event)

for dev, log in sorted(device_log.items()):
    print(f"장치 {dev}: {log}")

# Counter: 이벤트 빈도 집계
event_types = [e for _, e in events]
counter = Counter(event_types)
print(f"\n이벤트 빈도: {counter}")
print(f"가장 많은 이벤트 top 2: {counter.most_common(2)}")
```

**코드 해설**

일반 딕셔너리였다면 if device_id not in device_log: device_log[device_id] = [] 코드가 필요하다. defaultdict(list)는 이 초기화를 자동으로 처리한다. Counter는 리스트나 문자열을 바로 집계할 수 있고 most_common(n)으로 상위 n개를 즉시 추출한다. 센서 오류 코드, 프로토콜 메시지 타입의 빈도를 분석할 때 유용하다.

**실행 결과와 확인할 점**

device_log[1]에 ["연결", "데이터"], device_log[2]에 ["오류", "연결"]이 담겨야 한다. counter.most_common(2)에서 "연결"이 3회로 1위로 나오는 것을 확인한다.

### 집합(set)과 해시 탐색의 O(1) 복잡도

리스트에서 in 연산은 첫 번째 요소부터 순서대로 비교하므로 O(n)이다. 허용 장치 목록이 10만 개라면 매 확인마다 최대 10만 번 비교를 수행한다. 집합은 해시 테이블 기반이어서 원소 수와 관계없이 O(1)에 탐색된다.

```python
import time

# 허용 장치 ID 목록 (실무에서는 수천~수만 개일 수 있음)
allowed_ids = list(range(100_000))        # 리스트
allowed_set = set(allowed_ids)            # 집합으로 변환 — 1회 O(n) 비용

stream = [i * 7 % 150_000 for i in range(50_000)]  # 임의 ID 스트림

# 리스트 in: O(n) 탐색
start = time.time()
hits_list = [x for x in stream if x in allowed_ids]
print(f"list 탐색: {(time.time()-start)*1000:.1f}ms, 히트={len(hits_list)}")

# 집합 in: O(1) 탐색
start = time.time()
hits_set = [x for x in stream if x in allowed_set]
print(f"set  탐색: {(time.time()-start)*1000:.1f}ms, 히트={len(hits_set)}")

# 교집합: 두 장치 목록의 공통 ID 추출
group_a = set(range(0, 100_000, 3))
group_b = set(range(0, 100_000, 7))
common = group_a & group_b               # O(min(a,b))
print(f"공통 ID 수: {len(common)}")
```

**코드 해설**

두 결과의 히트 수는 동일하다. list 버전은 스트림 5만 개 × 허용 목록 10만 개 = 최악 50억 번의 비교가 일어나고, set 버전은 스트림 5만 번의 해시 계산만으로 끝난다. 집합 교집합 연산(& 또는 intersection())은 이중 for 루프보다 훨씬 빠르다. 허용 목록, 오류 코드 필터, 중복 제거처럼 순서가 필요 없는 빠른 탐색이라면 리스트 대신 집합을 우선 고려한다. 내용이 변하지 않는 허용 목록에는 frozenset을 써서 해시 가능하게 만들고 딕셔너리 키나 집합 원소로도 사용할 수 있다.

**실행 결과와 확인할 점**

list 탐색이 set 탐색보다 수십 배 이상 느린 것을 확인한다. allowed_ids 크기를 500_000으로 늘리면 차이가 더 극단적으로 벌어진다.

> **한 걸음 더**
> frozenset으로 허용 목록을 선언하고 allowed_set.add(999)처럼 추가를 시도하면 AttributeError가 발생하는 것을 확인한다. frozenset은 딕셔너리 키로도 사용할 수 있다.
