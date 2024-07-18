# 제너레이터 안에서 throw로 상태를 변화시키지 말자 

제너레이터 안에서 Exception을 던질 수 있는 throw 메서드가 있다. 

ex. 제너레이터가 throw를 호출하면 yield로투버 평소처럼 제너레이터 실행이 계속되지 않고 `throw가 제공한 Exception을 던진`다. 
``` python
class MyError(Exception):
    pass

def my_generator():
    yield 1
    yield 2
    yield 3
    

it = my_generator()
print(next(it))       # 1을 내놓음
print(next(it))       # 2를 내놓음
print(it.throw(MyError('test error')))
# 출력:
# 1
# 2
# MyError 발생!
```

throw를 호출해 제너레이터에 예외를 주입해도  
제너레이터는 try/except 복합문을 사용해 마지막으로 실행된 yield 문을 둘러쌈으로써 예외를 잡아낼 수 있다. 

``` python
def my_generator():
    yield 1 
    try:
        yield 2 
    except MyError:
        print('MyError 발생!') 
    else:
        yield 3

    yield 4 # 

it = my_generator()
print(next(it))       # 1을 내놓음
print(next(it))       # 2를 내놓음
print(it.throw(MyError('test error'))) # 이전처럼 다음 yield를 실행하지 않고
                                       # throw가 제공한 Exception을 잡아낸 부분부터 다시 시작한다. 
# 출력:
# 1
# 2
# MyError 발생!
# 4
```

제너레이터와 제너레이터를 호출하는 쪽 사이에 양방향 통신 수단을 제공한다.  
또 다른 예시를 살펴보도록 하자. 

ex. 간헐적으로 재설정할 수 있는 타이머를 제너레이터를 통해 구현함 

``` python
class Reset(Exception):
    pass

def timer(period):
    current = period
    while current:
        current -= 1
        try:
            yield current

        # Reset 예외가 발생할 때 마다 카운터가 period로 재설정됨 
        except Reset: 
            current = period
```

위 예외와 메소드를 가지고 아래와 같은 코드를 작성할 수 있다. 

``` python
def check_for_reset():
    # 외부 이벤트를 폴링합니다
    pass

def announce(remaining):
    print(f'{remaining} 틱 남음')

# 해당 메소드를 동작시킬려고 한다. 
def run():
    it = timer(4) # timer 제너레이터에 period = 4를 전달했다. 
    while True:
        try:
            if check_for_reset(): # 외부 이벤트를 실행한다. 
                it.throw(Reset()) # True면 Reset예외를 전달한다. 
            else:
                current = next(it) # 그렇지 않으면 제너레이터를 정상적으로 동작시킨다.

        except StopIteration: # StopIteration 예외가 발생하면 while문을 멈춘다. 
            break
        else: # 예외가 발생하지 않으면 announce 메소드가 실행된다. 
            announce(current)

run()
# 출력:
# 3 틱 남음
# 2 틱 남음
# 1 틱 남음
# 3 틱 남음
# 2 틱 남음
# 3 틱 남음
# 2 틱 남음
# 1 틱 남음
# 0 틱 남음
```

정리하면 `check_for_reset`이 True가 되면 Reset 예외를 이터레이터에 던지면서 제너레이터 내부의 current 값이 초기화되고  
그렇지 않으면 1씩 감소하면서 값을 전달한다. 

하지만, 역시나 가독성이 떨어진다. 

## 보다 단순한 구현 방법 

`iterable 컨테이너 객체`를 사용해 `상태가 있는 클로저`를 정의하는 것이다.  
Timer 제너레이터를 재정의하면 다음과 같다. 

``` python
class Timer:
    def __init__(self, period):
        self.current = period
        self.period = period

    def reset(self):
        self.current = self.period

    def __iter__(self):
        while self.current:
            self.current -= 1
            yield self.current

def run():
    timer = Timer(4)
    for current in timer:
        if check_for_reset(): # 해당 조건문이 True이면 
            timer.reset() # 메서드 실행
                          # 클래스 내부의 current값이 앞서 설정한 period 매개변수 4로 초기화된다
                          # 이 메서드를 실행하고 나면 해당 인스턴스를 가지고 이터레이션할 때 current=4부터 다시 시작한다.
        announce(current)

run()
# 출력:
# 3 틱 남음
# 2 틱 남음
# 1 틱 남음
# 3 틱 남음
# 2 틱 남음
# 3 틱 남음
# 2 틱 남음
# 1 틱 남음
# 0 틱 남음

```

이전보다 좀 더 가독성이 높아졌다. 

제너레이터와 예외를 섞어서 만들어야 하는 작업이 있다면 `비동기 기능`을 사용하면 더 좋게 구현할 수 있는 경우도 많다.  

따라서, 예외적인 경우를 처리해야 한다면 throw를 사용하지 말고 이터러블 클래스를 사용하도록 하자. 











