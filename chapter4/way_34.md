# send로 제너레이터에 데이터를 주입하지 말자 

ex. SW 라디오를 사용해서 신호를 보낸다.
``` python
import math

def wave(amplitude, steps):
    step_size = 2 * math.pi / steps  # 2 * 라디안 / 단계 수
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output = amplitude * fraction
        yield output

def transmit(output):
    if output is None:
        print(f'출력: None')
    else:
        print(f'출력: {output:5.1f}')

def run(it):
    for output in it:
        transmit(output)

# 기본 파형을 생성하는 코드가 잘 동작하는 걸 확인했다.
run(wave(3.0, 8))
# Output:
# 출력: 0.0
# 출력: 2.1
# 출력: 3.0
# 출력: 2.1
# 출력: 0.0
# 출력: -2.1
# 출력: -3.0
# 출력: -2.1
```

하지만 별도의 입력을 사용해 진폭을 지속적으로 변경해야 한다면 이 코드를 변경해줘야 한다.  
즉, 제너레이터를 이터레이션할 때 마다 진폭을 변조할 수 있는 방법이 필요하다. 

## send 지원 

send 메서드를 이용해서 yield 식을 양방향 채널로 만들 수 있다. 
입력값을 제너레이터에 스트리밍하는 동시에 출력을 내보낼 수 있다. 

일반적으로 제너레이터를 이터레이션할 때 yield 식이 반환하는 값은 None이다. 

``` python
def my_generator():
    received = yield 1
    print(f'받은 값: {received}')

it = iter(my_generator())
output = next(it)          # 첫 번째 제네레이터 출력을 얻는다
print(f'출력값: {output}')

try:
    next(it)               # 종료될 때까지 제네레이터를 실행한다
except StopIteration:
    pass

# Output:
# 출력값: 1
# 받은 값: None
```




















