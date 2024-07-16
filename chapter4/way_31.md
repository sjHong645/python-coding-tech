# 매개변수에 대해 이터레이션할 때는 조심하자 

ex. 미국 텍사스 주의 여행자 수를 분석하고 싶다. 

전달받은 데이터 집합은 도시별 방문자 수라고 하자. 

``` python
def normalize(numbers): # 리스트를 매개변수로 받고

    total = sum(numbers) # 리스트 안에 있는 값의 총합 total 
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent) # 각 도시별 퍼센트 값을 반환할 리스트에 추가 
    return result

visits = [15, 35, 80]
percentages = normalize(visits)
print(percentages)
assert sum(percentages) == 100.0

# Output: [11.538461538461538, 26.923076923076923, 61.53846153846154]
```

위 코드를 좀 더 확장하려면 텍사스의 모든 도시에 대한 여행자 정보가 들어있는 파일에서 데이터를 읽어야 한다.  
더 나아가 전 세계를 대상으로 여행자 분석을 할 수도 있다. 

그렇게 되면 위 코드는 너무 많은 메모리를 소모하게 될 것이다. 그래서 `제너레이터를 정의`할 필요가 있다. 

- 제너레이터를 이용
