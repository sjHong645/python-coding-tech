# 변수 위치 인자를 사용해 시각적인 잡음을 줄이자 

- 위치 인자 : 가변 인자(varargs) 또는 스타 인자(* args)라고 부르기도 한다.
  - 해당 인자를 가변적으로 받을 수 있으면 함수 호출이 더 깔끔해지고 시각적 잡음도 줄어든다.

ex) 디버깅 정보를 로그에 남기기 
``` python
def log(message, values): # 매개변수의 개수가 고정되어 있다.
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')

log('내 숫자는', [1, 2])
log('안녕', []) # 로그에 남길 값(value)이 없을 때 빈 리스트를 넘겨야 한다. 인자의 개수는 2개로 고정되어 있기 때문이다. 
                # 이로 인해 코드 잡음이 발생한다는 걸 확인할 수 있다. 
```

이럴 때 두 번째 인자를 완전히 생략하면 좋을 것이다.  
이를 위해서 Python에서는 마지막에 위치한 인자 이름 앞에 `*`를 붙이면 된다. 

``` python
def log(message, *values): # values 라는 매개변수 앞에 * 를 붙여줬다. 
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')

log('내 숫자는', [1, 2])
log('안녕') # 로그에 남길 값이 없으니까 아무런 값을 넘기지 않아도 된다.
           # 첫 번째 매개변수 이후의 인자는 위치 인자로 정의했기 때문이다.
```

## 문제점 

1. 선택적인 위치 인자가 함수에 전달되기 전에 `항상 튜플로 변환`된다

이는 함수를 호출하는 쪽에서 제너레이터 앞에 `*` 연산자를 사용하여 함수에 전달하면 
제너레이터가 생성하는 모든 값을 한꺼번에 가져와서 튜플로 반환한다. 

제너레이터가 생성하는 값이 많을 경우 이 모든 값이 한꺼번에 튜플로 변환되면서 메모리를 많이 차지하게 된다. 

ex) my_func(*my_generator())와 같이 제너레이터를 호출하면,  
my_generator()가 생성하는 모든 값이 한꺼번에 메모리에 로드되어 튜플로 변환된다. 

때문에 메모리를 많이 소비하거나 프로그램이 중단될 수 있다. 
``` python 
def my_generator():
    for i in range(10):
        yield i

def my_func(*args):
  print(args)
  
it = my_generator()
my_func(*it)

출력 : (0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
```

그래서 `*args`를 받는 함수는 인자 목록에서 가변적인 부분에 들어가는 인자의 개수가 충분히 작다는 사실을 미리 알고 있는 경우에 적합하다.  
`*args`는 여러 literal이나 변수 이름을 함께 전달하는 함수 호출에 이상적이다. 

2. 함수에 새로운 위치 인자를 추가하면 해당 함수를 호출하는 모든 코드를 변경해야 한다. 

``` python
def log(sequence, message, *values):
    if not values:
        print(f'{sequence} - {message}')
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{sequence} - {message}: {values_str}')

log(1, '좋아하는 숫자는', 7, 33) # 가변인자(values) 사용 
log(1, '안녕') # values 사용없이 함수 호출 
log('좋아하는 숫자는', 7, 33) # 문제 발생
                             # 예전에 작성한 코드여서 sequence라는 새로운 매개변수를 생각하지 못하고 작성되었음
                             # 그럼에도 불구하고 예외를 발생하지 않고 코드가 작동했기 때문에 버그를 추적하기 까다로움

>> 출력값 
1 - 좋아하는 숫자는: 7, 33
1 - 안녕
좋아하는 숫자는 - 7: 33

```

그래서 `*args`를 받는 함수를 확장할 때는 `키워드 기반의 인자`만 사용하거나 `타입 애너테이션`을 사용하면 된다. 




