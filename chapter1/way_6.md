# 인덱스를 사용하는 대신 대입을 사용해서 데이터를 언패킹하자 

## 튜플값에 접근

### 인덱스 사용
``` python
>>> items = ('호박엿', '식혜')
>>> items[0]
'호박엿'
>>> items[1]
'식혜'
```

### 언패킹 사용 - 이 방법을 사용하자. 
``` python
>>> items = ('호박엿', '식혜')
>>> first, second = items # 언패킹 사용
                          # 보다시피, 한 줄에 여러 개의 변수에 값을 대입했다.
>>> first 
'호박엿'
>>> second
'식혜'
```

이와 같이 `언패킹`을 사용하면 인덱스를 사용했을 때 보다 깔끔하게 변수들을 사용할 수 있다.  
튜플 뿐만 아니라 리스트, 시퀀스, iterable 등 다양한 패턴에서 언패킹 구문을 사용할 수 있다. 

ex. 
``` python
cell_phone = {
  '갤럭시' : ('S24', 2024),
  '아이폰' : ('iPhone 15', 2023), 
}

# 언패킹을 사용해서 원하는 이름의 변수에 값을 대입함 
((cellphone1, (type1, year1)),
 (cellphone2, (type2, year2))) = cell_phone.items()
```

## for루프에서 사용한 언패킹 

``` python
def bubble_sort(a) :
  for _ in range(len(a)) ;
    for i in range(1, len(a)) :

      # 인덱스를 사용한 경우 
      if a[i] < a[i-1] :
        temp = a[i]
        a[i] = a[i-1]
        a[i-1] = temp

      # 언패킹을 사용한 경우
      if a[i] < a[i-1] :
        a[i-1], a[i] = a[i], a[i-1]
```

위와 같이 언패킹을 사용했을 때 어떤 동작이 이뤄지는지 살펴보자. 

1. `우항(a[i], a[i-1])`들이 일단 계산이 완료되었다.
2. 그 결과값들이 임시 tuple에 저장된다.
3. 임시 튜플의 값들을 `언패킹`을 통해 `좌항(a[i-1], a[i])`에 있는 변수들에 각각 대입한다.
4. 모든 작업이 끝나면 임시 튜플은 삭제된다.

## 리스트의 원소를 언패킹 

``` python
snacks = [('베이컨', 350), ('도넛', 240), ('머핀', 190)]

# 인덱스를 사용한 경우
for i in range(len(snacks)) :
    item = snack[i]
    name = item[0]
    calories = item[1]

# 언패킹을 사용한 경우 
for rank, (name, calories) in enumerate(snacks, 1) :
  # rank는 순번
  # name과 calories는 리스트의 원소인 튜플의 0, 1번째 값이 저장되어 사용할 수 있음 
```

위와 같이 `언패킹`을 이용하는 방식이 python다운 방식이다.  
때문에 일반적으로는 인덱스를 사용해서 무언가에 접근할 필요가 없다.










