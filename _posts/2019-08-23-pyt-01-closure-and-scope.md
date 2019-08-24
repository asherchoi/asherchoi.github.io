---
title: "[Python] How Closure Interact with Variable Scope"
excerpt:
last_modified_at: 2019-08-23

categories:
  - Python

tags:
  - algorithm
  - python
  - closure
  - nonlocal
  - global
  - variable scope
  - call

---
![python-version-3.7.1](https://img.shields.io/badge/python-v3.7.1-blue.svg)

# Overview
숫자 리스트를 정렬할 때 특정 그룹의 숫자들이 먼저 오도록 우선순위를 매기려고 한다고 하자.
이런 패턴은 사용자 인터페이스를 표현하거나, 다른 것보다 중요한 메시지나 예외 이벤트를 먼저 보여줘야 할 때 유용하다. <br><br>

이렇게 만드는 일반적인 방법은 리스트의 `sort` 메소드에 헬퍼 함수를 `key` 인수로 넘기는 것이다.
헬퍼의 반환 값은 리스트에 있는 각 아이템을 정렬하는 값으로 사용된다.
헬퍼는 주어진 아이템이 중요한 그룹에 있는지 확인하고 그에 따라 정렬 키를 다르게 할 수 있다. <br>

```python
def sort_priority(value, group):
    def helper(x):
        if x in group:
            return 0, x
        return 1, x
    value.sort(key=helper)
```

이 함수를 간단한 입력 값에 사용한다.

```python
numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = {2, 3, 5, 7}
sort_priority(numbers, group)
print(numbers)
>>> [2, 3, 5, 7, 1, 4, 6, 8]
```

위 함수가 예상대로 동작하는 이유는 세 가지다. <br>

# Python closure
- 파이썬은 클로저(closure)를 지원한다. **클로저란 자신이 정의된 스코프에 있는 변수를 참조하는 함수** 다. 바로 이 점 덕분에 `helper` 함수가 `sort_priority`의 `group` 인수에 접근할 수 있다.

- 함수는 파이썬에서 일급 객체(first-class object)다. 이 말은 함수를 직접 함조하고, 변수에 할당하고, 다른 함수의 인수로 전달하고, 표현식과 if 문 등에서 비교할 수 있다는 의미다. 따라서 `sort` 메소드에서 클로저 함수를 `key` 인수로 받을 수 있다.

- 파이썬에는 튜플을 비교하는 특정한 규칙이 있다. 먼저 인덱스 0으로 아이템을 비교하고 그 다음으로 인덱스 1, 다음은 인덱스 2와 같이 진행한다. `helper` 클로저의 반환 값이 정렬 순서를 분리된 두 그룹으로 나뉘게 한 건 이 규칙 때문이다.

함수에서 우선순위가 높은 아이템을 발견했는지 여부를 반환해서 사용자 인터페이스 코드가 그에 따라 동작하게 하면 좋을 것이다. 이런 동작을 추가하는 일은 쉬워 보인다.
<br>

# Common mistake using closure
이미 각 숫자가 어느 그룹에 포함되어 있는지 판별하는 클로저 함수가 있다. 우선순위가 높은 아이템을 발견했을 때 플래그를 뒤집는 데 클로저를 사용하는 건 어떨까? 그러면 함수는 클로저가 수정한 플래그 값을 반환할 수 있다. <br>

```python
def sort_priority2(numbers, group):
    found = False
    def helper(x):
        if x in group:
            found = True
            return 0, x
        return 1, x
    numbers.sort(key=helper)
    return found
```

앞에서 사용한 것과 같은 입력으로 함수를 실행할 수 있다.

```python
found = sort_priority2(numbers, group)
print('Found:', found)
print(numbers)
>>> Found: False
>>> [2, 3, 5, 7, 1, 4, 6, 8]
```
정렬된 결과는 올바르지만 `found` 결과는 틀렸다. `group`에 속한 아이템을 `numbers`에서 찾을 수 있었지만 함수는 `False`를 반환했다. 어째서 이런 일이 일어났을까? <br><br>

## Python interpreter scope search
표현식에서 변수를 참조할 때 파이썬 인터프리터는 참조를 해결하려고 다음과 같은 순서로 스코프를 탐색한다.

1. 현재 함수의 스코프
2. (현재 스코프를 담고 있는 다른 함수 같은) 감싸고 있는 스코프
3. 코드를 포함하고 있는 모듈의 스코프(전역 스코프라고도 함)
4. (`len`이나 `str` 같은 함수를 담고 있는) 내장 스코프

이 중 어느 스코프에도 참조한 이름으로 된 변수가 정의되어 있지 않으면 `NameError` 예외가 일어난다.<br><br>

변수에 값을 할당할 때는 다른 방식으로 동작한다. 변수가 이미 현재 스코프에 정의되어 있다면 새로운 값을 얻는다. **파이썬은 변수가 현재 스코프에 존재하지 않으면 변수 정의로 취급** 한다. 새로 정의되는 변수의 스코프는 그 할당을 포함하고 있는 함수가 된다. <br><br>

이 할당 동작은 `sort_priority2` 함수의 반환 값이 잘못된 이유를 설명해 준다. `found` 변수는 `helper` 클로저에서 `True`로 새로 할당된다. 클로저 할당은 `sort_priority2`에서 일어나는 할당이 아닌 helper 안에서 일어나는 새 변수 정의로 처리된다. <br>

```python
def sort_priority2(numbers, group):
    found = False  # scope: sort_priority2
    def helper(x):
        if x in group:
            found = True  # scope: helper
            return 0, x
        return 1, x
    numbers.sort(key=helper)
    return found
```

이 문제는 언어 설계자가 의도한 결과다. 이 동작은 함수의 지역 변수가 자신을 포함하는 모듈을 오염시키는 문제를 막아준다. 그렇지 않았다면 함수 안에서 일어나는 모든 할당이 전역 모듈 스코프에 쓰레기를 넣는 결과로 이어졌을 것이다. 그렇게 되면 불필요한 할당에 그치지 않고 결과로 만들어지는 전역 변수들의 상호 작용으로 알기 힘든 버그가 생긴다. <br>

# How to get data from closure
## nonlocal
파이썬 3에는 클로저에서 데이터를 얻어오는 특별한 문법이 있다. **`nonlocal`문은 특정 변수 이름에 할당할 때 스코프 탐색이 일어나야 함을 나타낸다.** 유일한 제약은 `nonlocal`이 (전역 변수의 오염을 피하려고) 모듈 수준 스코프까지는 탐색할 수 없다는 점이다. <br><br>

다음은 `nonlocal`을 사용하여 같은 함수를 다시 정의한 예다.

```python
def sort_priority3(numbers, group):
    found = False
    def helper(x):
        nonlocal found
        if x in group:
            found = True
            return 0, x
        return 1, x
    numbers.sort(key=helper)
    return found
```

```python
found = sort_priority3(numbers, group)
print('Found:', found)
print(numbers)
>>> Found: True
>>> [2, 3, 5, 7, 1, 4, 6, 8]
```

## nonlocal vs global
`nonlocal`문은 클로저에서 데이터를 다른 스코프에 할당하는 시점을 알아보기 쉽게 해준다.
`nonlocal`문은 변수 할당이 모듈 스코프에 직접 들어가게 하는 `global`문을 보완한다. <br>
<br>
하지만 전역 변수의 안티패턴(anti-pattern)과 마찬가지로 간단한 함수 이외에는 `nonlocal`을 사용하지 않도록 주의해야 한다. `nonlocal`의 부작용은 알아내기가 상당히 어렵다. 특히 `nonlocal` 문과 관련 변수에 대한 할당이 멀리 떨어진 긴 함수에서는 이해하기가 더욱 어렵다.<br>
<br>

# More intuitive - Use class
`nonlocal`을 사용할 떄 복잡해지기 시작하면 헬퍼 클래스로 감싸는 방법을 이용하는 게 낫다. 이제 `nonlocal`을 사용할 떄와 같은 결과를 얻는 클래스를 정의해보자. 코드는 약간 더 길지만 이해하기는 훨씬 쉽다. <br>

``` python
class Sorter(object):
    def __init__(self, group):
        self.group = group
        self.found = False

    def __call__(self, x):
        if x in self.group:
            self.found = True
            return 0, x
        return 1, x
```

```python
sorter = Sorter(group)
numbers.sort(key=sorter)
assert sorter.found is True
```

# Sum up
 1. 클로저 함수는 자신이 정의된 스코프 중 어디에 있는 변수도 참조할 수 있다.
 2. 기본적으로 클로저에서 변수를 할당하면 바깥쪽 스코프에는 영향을 미치지 않는다.
 3. Python3에서는 `nonlocal`문을 사용하여 클로저를 감싸고 있는 스코프의 변수를 수정할 수 있음을 알린다.
 4. 간단한 함수 에서만 `nonlocal`을 사용하고, 복잡해지면 `class`를 사용하자.

# Citation & Reference
 - Brett Slatkin. "Effective Python: 59 Specific Ways to Write Better Python. Trans." _Know How to closure Interact with variable scope._ Pearson Education, 2016. 57-60.
 - [https://tech.ssut.me/python-functions-are-first-class/](https://tech.ssut.me/python-functions-are-first-class/)
