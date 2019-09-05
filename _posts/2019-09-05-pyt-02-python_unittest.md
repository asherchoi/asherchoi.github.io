---
title: "[Python] Use unittest to test whole things"
excerpt:
last_modified_at: 2019-09-05

categories:
  - Python

tags:
  - test
  - QA
  - unit test
  - integration test
  - python
  - test case
  - setup
  - teardown
  - assertion
  - type safety

---
![python-version-3.4.1](https://img.shields.io/badge/python-v3.7.1-blue.svg)

# Why we need to test
파이썬에는 정적 타입 검사 기능이 없다. 컴파일러는 프로그램을 실행하면 제대로 동작함을 보장하지 않는다. 파이썬으로는 프로그램에서 호출하는 함수가 런타임에 정의되어 있을지 알 수 없으며, 심지어 해당 함수가 소스코드에 있다고 해도 알 수 없다. 이런 동적인 동작은 축복이 될 수도 있지만 저주가 될 수도 있다.  
<br>
많은 파이썬 프로그래머가 간결성과 단순함으로 얻을 수 있는 생산성 때문에 이 동작이 가치 있다고 말한다. 하지만 대부분의 사람이 적어도 한 번쯤은 파이썬 프로그램이 런타임에 어처구니 없는 오류에 맞닥뜨린다는 얘기를 듣는다.  
<br>
최악의 사례 중 하나는 동적 임포트의 부작용으로 `SyntaxError`가 제품에서 발생했다는 것이다. 어떤 언어로 작성하든 항상 코드를 테스트해야 한다. 하지만 파이썬과 다른 언어의 가장 큰 차이는 파이썬 프로그램을 신뢰할 수 있는 유일한 방법이 직접 테스트를 작성하는 것밖에 없다는 점이다. 파이썬의 동적 특성과 쉽게 오버라이트할 수 있는 동작으로 테스트를 구현해서 프로그램이 예상대로 잘 동작함을 보장할 수 있다.
<br>
테스트를 코드에 대한 보장 정책으로 생각해야 한다. 좋은 테스트는 코드가 올바르게 동작한다는 신뢰를 준다. 코드를 리팩토링하거나 확장하는 용도로 테스트를 사용하면 동작이 어떻게 바뀌는지 쉽게 구별할 수 있다. 직관적이지 않게 들리지만, 좋은 테스트를 만들어두면 실제로 파이썬 코드를 더욱 쉽게 수정할 수 있다.  

# Unit test module
## How to use unittest 
테스트를 작성하는 가장 간단한 방법은 내장 모듈 `unittest`를 사용하는 것이다. 예를 들어  `utils.py`에 다음과 같은 유틸리티 함수를 정의했다고 하자.

```python 
# utils.py
def to_str(data):
    if isinstance(data, str):
        return data
    elif isinstance(data, bytes):
        return data.decode('utf-8')
    else:
        raise TypeError('Must supply str or bytes, found: %r' % data)
```
기대하는 각 동작에 대한 테스트를 담은 `test_util.py` 또는 `utils_test.py`라는 두 번째 파일을 생성한다.

```python 
# utils_test.py
from unittest import TestCase, main
from utils import to_str


class UtilsTestCase(TestCase):
    def test_to_str_bytes(self):
        self.assertEqual('hello', to_str(b'hello'))

    def test_to_str_str(self):
        self.assertEqual('hello', to_str('hello'))

    def test_to_str_bad(self):
        self.assertRaises(TypeError, to_str, object())


if __name__ == '__main__':
    main()
```
테스트는 `TestCase` 클래스로 구성되어 있다. 각 테스트는 test 단어로 시작하는 메서드다. 테스트 메서드가 어떤 종류의 `Exception`(`assert` 문에서 일어나는 `AssertionError` 포함)을 일으키지 않고 실행된다면 테스트가 성공적으로 통과한 것이다.  
<br>
`TestCase` 클래스는 테스트에서 assertion을 하는 데 필요한 헬퍼 메서드를 제공한다. 예를 들어 동등성을 검사하는 `assertEqual`, 부울 표현식을 검사하는 `assertTrue`, 적절할 때 예외가 발생하는지 검사하는 `assertRaises`(자세한 내용은 `help(TestCase)`로 확인하기 바란다) 등이다. `TestCase` 서브 클래스에 자신만의 헬퍼 메서드를 정의하여 테스트를 더 이해하기 쉽게 만들 수 있다. 헬퍼 메서드의 이름이 test로 시작하지 않게만 하면 된다.  
<br>

## Set test environment to use TestCase Class
때때로 `TestCase` 클래스에서 테스트 메서드를 실행하기 전에 테스트 환경을 설정해야 한다. 그려려면 `setUp`과 `tearDown` 메서드를 오버라이드해야 한다. 이 메서드들은 각 테스트 메서드를 실행하기 전후에 각각 호출되며, 각 테스트가 분리되어 실행됨을 보장한다(적절한 테스트의 모범 사례다.) 예를 들어 다음 코드는 각 테스트 전에 임시 디렉터리를 생성하고, 각 테스트가 종료한 후에 그 내용을 삭제하는 `TestCase`를 정의한다. 

```python  
from tempfile import TemporaryDirectory


class MyTest(TestCase):
    def setUp(self):
        self.test_dir = TemporaryDirectory()
    
    def teatDown(self):
        self.test_dir.cleanup()
    
    # Test method
    # ...
```
보통 관련 테스트의 각 집합별로 `TestCase` 하나를 정의한다. 때로는 예외상황이 많은 각 함수별로 `TestCase`를 만든다. `TestCase` 하나로 한 모듈에 속한 모든 함수를 처리하기도 한다. 또한 단일 클래스와 그 클래스의 메서드를 테스트할 용도로 TestCase를 생성하기도 한다.  
<br>
프로그램이 복잡해지면 코드를 별도로 테스트하는 대신에 모듈 간의 상호작용을 검증할 테스트를 추가하려고 할 것이다. 이게 바로 단위 테스트(unittest)와 통합 테스트(integration test)의 차이다. 파이썬에서는 똑같은 이유로 두 종류의 테스트를 작성해야 한다. 모듈을 검증하지 않으면 실제로 함께 동작하는 지 보장하지 못한다.

# Sum up
 1. 파이썬 프로그램을 신뢰할 수 있는 유일한 방법은 테스트를 작성하는 것이다.
 2. 내장 모듈 `unittest`는 좋은 테스트를 작성하는 데 필요한 대부분의 기능을 제공한다.
 3. `TestCase`를 상속하고 테스트하려는 동작별로 메서드 하나를 정의해서 테스트를 정의할 수 있다. `TestCase` 클래스에 있는 테스트 메서드는 `test`라는 단어로 시작해야 한다.
 4. 단위 테스트(고립된 기능용)와 통합 테스트(상호 작용하는 모듈용) 모두를 작성해야 한다.

# Citation & Reference
 - Brett Slatkin. "Effective Python: 59 Specific Ways to Write Better Python. Trans." _Know How to closure Interact with variable scope._ Pearson Education, 2016. 289-292.
