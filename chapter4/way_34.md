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
                           # my_generator()로 이터레이터를 만들고 yield에 설정된 대로 1을 출력하고
                           # 이터레이터를 모두 사용했다.  
print(f'출력값: {output}')

try:
    next(it)               # 종료될 때까지 제너레이터를 실행한다
                           # yield를 모두 실행하고 나서 next 메소드를 호출하는 거니까 아무 일도 발생하지 않는다.
except StopIteration:
    pass

# Output:
# 출력값: 1
# 받은 값: None
```

for문 또는 next 내장함수로 제너레이터를 이터레이션하지 않고 `send 메서드를 호출`하면  
제너레이터가 재개(resume)될 때 yield가 send에 전달된 파라미터 값을 반환한다. 

하지만, 제너레이터를 맨 처음 호출했을 때 제너레이터 내부에서 구현한 yield에 아직 도달하지 못했다. 
그래서 맨 처음에 send 메소드를 호출할 때 인자로 전달할 수 있는 값은 None 뿐이다.  
(다른 값을 전달하면 TypeError : can't send non-None value to a just-started generator 발생) 

``` python
it = iter(my_generator())
output = it.send(None)  # 제너레이터를 처음 호출하는 거니까
                        # send 메소드의 인자로 None을 전달함 

print(f'출력값: {output}') # 제너레이터로 만든 이터레이터가 출력한 값 1을 전달받는다. 

try:
    it.send('안녕!')  # 이제부터 원하는 값을 제너레이터에 전달한다.
                     # yield 식에 '안녕!' 이라는 값을 전달받았으니가 제너레이터 내부에서 그 값을 출력한다. 
except StopIteration:
    pass

# 출력값: 1
# 받은 값: 안녕! 
```

이러한 동작을 바탕으로 sine 파의 진폭을 변조할 수 있도록 수정했다.  

``` python
def wave_modulating(steps):
    step_size = 2 * math.pi / steps
    amplitude = yield  # 첫 번째 진폭값을 받는다
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output = amplitude * fraction
        amplitude = yield output  # 그 이후의 진폭값을 받는다

def run_modulating(it):
    amplitudes = [
        None, 7, 7, 7, 2, 2, 2, 2, 10, 10, 10, 10]
    for amplitude in amplitudes:
        output = it.send(amplitude) # 매 iteration마다 변조에 사용할 진폭값을
                                    # 제너레이터에게 전달하도록 했다. 
        transmit(output)

def transmit(output):
    if output is None:
        print(f'출력: None')
    else:
        print(f'출력: {output:5.1f}')

run_modulating(wave_modulating(12)) # 진폭을 만들어내는 반복문을 12번 반복한다. 
출력: None
출력: 0.0
출력: 3.5
출력: 6.1
출력: 2.0
출력: 1.7
출력: 1.0
출력: 0.0
출력: -5.0
출력: -8.7
출력: -10.0
출력: -8.7
출력: -5.0
```

코드는 잘 동작하지만 처음봤을 때 어떤 동작을 하는 코드인지 이해하기 어렵다.  
대입문의 오른쪽에 yield를 사용하는 것 역시 직관적이지 않다. 

그리고 제너레이터의 고급 기능을 잘 모른다면 send와 yield 사이의 연관성을 알아보기 어렵다. 

## 좀 더 복잡한 예시 

ex. 여러 신호의 시퀀스로 이뤄진 복잡한 파형 사용 
``` python
def wave_modulating(steps):
    step_size = 2 * math.pi / steps
    amplitude = yield  # 첫 번째 진폭값을 받는다
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output = amplitude * fraction
        amplitude = yield output  # 그 이후의 진폭값을 받는다

def run_modulating(it):
    amplitudes = [
        None, 7, 7, 7, 2, 2, 2, 2, 10, 10, 10, 10]
    for amplitude in amplitudes:
        output = it.send(amplitude) # yield from을 사용한 내포된 제너레이터에서 send를 호출할 때
                                    # 기본적으로 None을 보낸다.
                                    # 이는 제너레이터가 처음 시작할 때의 동작이다.
                                    # 그러니까 wave_modulating(3)에 보낼 때 맨 처음에 None
                                    # wave_modulating(4)에 보낼 때 맨 처음에 None
                                    # wave_modulating(5)에 보낼 때 맨 처음에 None
        transmit(output)

def complex_wave_modulating():
    yield from wave_modulating(3)
    yield from wave_modulating(4)
    yield from wave_modulating(5)

run_modulating(complex_wave_modulating())
# Output:
# 출력: None
# 출력: 0.0
# 출력: 6.1
# 출력: -6.1
# 출력: None
# 출력: 0.0
# 출력: 2.0
# 출력: 0.0
# 출력: -10.0
# 출력: None
# 출력: 9.5
# 출력: 5.9
```

중간중간 None 값이 출력된 걸 볼 수 있다. 왜 이런 결과가 나타났을까. 

제너레이터(`complex_wave_modulating`)에서 yield from 식이 끝날때 마다 다음 yield from 식이 실행된다. 
각각의 내포된 제너레이터(`wave_modulating`)는 send 메서드 호출로부터 값을 받기 위해서 
아무런 값도 만들어내지 않는 단순한 yield 식으로 시작한다. 

때문에, 부모 제너레이터가 자식 제너레이터를 옮길 때 마다 None이 출력된다. 좀 더 상세한 내용은 코드의 주석을 보자. 

이처럼 yield from과 send를 따로 사용할 때는 제대로 작동하던 특성들이 함께 사용할 때는 병립되지 않는다. 

물론, None 문제를 우회하는 방법이 있지만 그런 노력을 기울일 가치는 없다.  
send의 동작을 이해하기도 어려운데 yield from과 같이 사용했을 때 문제까지 이해해야 한다면 더 어려워진다. 

그래서 아예 `send 메시지를 쓰지 말고` 다른 방법을 택할 것을 권고한다. 

## 다른 방법 

- 가장 쉬운 해결책 : wave 함수에 이터레이터 전달하기

이 이터레이터는 자신에게 next 함수가 호출될 때 마다 입력으로 받은 진폭을 하나씩 돌려준다.  
이런 식으로 이전 제너레이터를 다음 제너레이터의 입력으로 연결하면 입력과 출력이 차례대로 처리될 수 있다. 

``` python
def wave_cascading(amplitude_it, steps):
    step_size = 2 * math.pi / steps
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        amplitude = next(amplitude_it)  # 다음 입력 받기
        output = amplitude * fraction
        yield output
```

이터레이터는 순회(iteration)하면서 상태(state)를 유지한다. 즉, 어디까지 순회했는지 기억하고 있다.  
그래서 내포된 각각의 제너레이터는 앞에 있던 제너레이터가 처리를 끝낸 시점부터 데이터를 가져와 처리한다. 
``` python 
def complex_wave_cascading(amplitude_it):
    yield from wave_cascading(amplitude_it, 3)
    yield from wave_cascading(amplitude_it, 4)
    yield from wave_cascading(amplitude_it, 5)
```

amplitudes 리스트에서 얻은 이터레이터를  
합성한 제너레이터에 넘기기만 하면 전체를 실행할 수 있다. 
``` python 
def run_cascading():
    amplitudes = [7, 7, 7, 2, 2, 2, 2, 10, 10, 10, 10, 10]
    it = complex_wave_cascading(iter(amplitudes))
    for amplitude in amplitudes:
        output = next(it)
        transmit(output)

run_cascading()
# 출력:
# 출력: 0.0
# 출력: 6.1
# 출력: -6.1
# 출력: 0.0
# 출력: 2.0
# 출력: 0.0
# 출력: -2.0
# 출력: 0.0
# 출력: 9.5
# 출력: 5.9
# 출력: -5.9
# 출력: -9.5
```

위 코드는 입력 제너레이터가 `thread-safe` 하다고 가정한 상태에서 동작한 코드이다.  
그렇지 않은 상황인 경우 예상했던 이터레이터의 상태와 다른 상태에 위치하면서 예상과 다른 결과가 출력될 수 있다.  

따라서, 스레드의 경계를 넘으면서 제너레이터를 사용한다면 async 함수가 더 나은 해법일 수 있다. 















