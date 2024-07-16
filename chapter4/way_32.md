# 길이가 긴 list comprehension 보다는 제너레이터 식을 사용하자 

## 리스트 컴프리헨션의 문제점 
입력 시퀀스와 같은 수의 원소가 들어있는 리스트 인스턴스를 만들어 낼 수 있다.  

때문에, 입력이 커지면 메모리를 상당히 많이 사용하면서 프로그램이 중단될 수 있다. 

ex. 파일을 읽어서 각 줄에 들어있는 문자 수를 반환하고 싶다
``` python
# 파일에서 읽은 줄에는 항상 문자가 들어 있으므로 길이가 눈에 보이는 길이보다 1만큼 더 크다

value = [len(x) for x in open('my_file.txt')]
print(value)

# Output: [100, 57, 15, 1, 12, 75, 5, 8, 89, 11]
```

리스트 컴프리헨션을 통해 작업하면서 파일의 각 줄의 길이를 메모리에 저장하게 되는데  
파일이 아주 크거나 절대 끝나지 않는 네트워크 소켓이라면 앞서 언급한 문제가 발생할 수 있다. 

## 제너레이터 식 (generator expression)

제너레이터 식 : 리스트 컴프리헨션과 제너레이터를 일반화한 것 

제너레이터 식을 실행해도 출력 시퀀스 전체가 실체화되지는 않는다.  
대신 제너레이터 식에 들어있는 식으로부터 원소를 하나씩 만들어내는 이터레이터가 생성된다. 

ex. 앞선 예시와 동일한 작업을 하는 제너레이터 식 
``` python
it = (len(x) for x in open('my_file.txt')) # () 사이에 리스트 컴프리헨션과 비슷한 구문을 넣었음
                                           # 제너레이터 식 생성됨  
print(it)
# Output: <generator object <genexpr> at 0x108993dd0>

print(next(it)) # 이터레이터니까 next 함수를 사용해야 제너레이터 식에서 다음 값을 얻을 수 있음 
print(next(it))
# Output: 
# 100
# 57
```

### 제너레이터 식 합성 

제너레이터의 강력한 특징 중 하나는 두 제너레이터 식을 합성할 수 있다는 점이다. 

``` python
# 제너레이터 식이 반환한 이터레이터를
# 또 다른 제너레이터 식의 입력으로 사용함
roots = ((x, x**0.5) for x in it)

print(next(roots))
# Output: (15, 3.872983346207417)
```

여기서는 `root 이터레이터`를 만들기 위해서 `앞서 만든 it 이터레이터`를 사용함. 

즉, `root 이터레이터`를 전진(next)시킬때 마다 내부의 이터레이터도 전진(next)되면서 연쇄적으로 루프가 실행된다. 

이처럼 제너레이터를 함께 연결한 코드를 파이썬은 아주 빠르게 실행할 수 있다.  
아주 큰 입력 스트림에 대해 여러 기능을 합성해 적용해야 한다면 제너레이터 식을 고려해보자. 

다만 제너레이터가 반환하는 이터레이터에는 상태가 있기 때문에 이터레이터를 한 번만 사용해야 한다. 


















