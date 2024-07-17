# yield from을 사용해서 여러 제너레이터를 합성하자 

## 예시 상황 
ex. 제너레이터를 사용해 화면의 이미지를 움직이게 하는 그래픽 프로그램이 있다 

- 애니메이션의 각 부분에 필요한 화면상 이동 변위(delta)를 만들 때 사용할 2가지 제너레이터
``` python
def move(period, speed):
    for _ in range(period): # speed 값을 period 만큼 출력 
        yield speed

def pause(delay):
    for _ in range(delay): # 0을 delay 만큼 출력 
        yield 0
```

최종 애니메이션을 만들려면 move, pause를 합성해서 변위 시퀀스를 하나만 만들어야 한다. 

애니메이션의 각 단계마다 제너레이터를 호출 ⇒ 차례대로 이터레이션 ⇒ 각 이터레이션에서 나온 변위들을 순서대로 내보내는 방식으로  
아래와 같은 시퀀스를 만들었다. 
``` python
def animate():
    for delta in move(4, 5.0): # 5.0이 4번 출력 
        yield delta 
    for delta in pause(3): # 0이 3번 출력 
        yield delta
    for delta in move(2, 3.0): # 3.0이 2번 출력 
        yield delta
```

- 화면에 표시한다.
``` python

def render(delta):
    print(f'Delta: {delta:.1f}')  # 화면에서 이미지를 이동시킨다

def run(func):
    for delta in func():
        render(delta)

run(animate)
# Output:
# Delta: 5.0
# Delta: 5.0
# Delta: 5.0
# Delta: 5.0
# Delta: 0.0
# Delta: 0.0
# Delta: 0.0
# Delta: 3.0
# Delta: 3.0
```

이 코드의 문제점은 animate가 너무 반복적이라는 것이다. for문과 yield가 반복되면서 가독성이 줄어들었다. 

## yield from 적용 

`yield from`은 고급 제너레이터 기능으로 제어를 부모 제너레이터에게 전달하기 전에 내포된 제너레이터가 모든 값을 내보낸다. 

- animate 함수에 yield from 적용
``` python
def animate_composed():
    yield from move(4, 5.0) # move(4, 5.0)를 실행했을 때 반환되는 결과를 반환 
    yield from pause(3) # pause(3)를 실행했을 때 반환되는 결과를 반환
    yield from move(2, 3.0) # move(2, 3.0)을 실행했을 때 반환되는 결과를 반환 

run(animate_composed)
# Output:
# Delta: 5.0
# Delta: 5.0
# Delta: 5.0
# Delta: 5.0
# Delta: 0.0
# Delta: 0.0
# Delta: 0.0
# Delta: 3.0
# Delta: 3.0
```

확실이 이전보다 코드가 명확해졌다. 

ex. timeit 내장 모듈을 통해 micro benchmark를 실행해서 성능이 개선되는지 살펴보자. 

``` python
import timeit

def child():
    for i in range(1_000_000):
        yield i

def slow():
    for i in child():
        yield i

def fast():
    yield from child()

baseline = timeit.timeit(
    stmt='for _ in slow(): pass',
    globals=globals(),
    number=50)
print(f'수동 내포: {baseline:.2f}s')

comparison = timeit.timeit(
    stmt='for _ in fast(): pass',
    globals=globals(),
    number=50)
print(f'합성 사용: {comparison:.2f}s')

reduction = (baseline - comparison) / baseline
print(f'{reduction:.1%} 시간 단축됨')
# Output:
# 수동 내포: 2.81s
# 합성 사용: 2.56s
# 8.8% 시간 단축됨
```
























