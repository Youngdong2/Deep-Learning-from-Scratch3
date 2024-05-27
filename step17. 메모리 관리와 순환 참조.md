## 메모리 관리
파이썬은 필요 없어진 객체를 메모리에서 자동으로 삭제한다. 그렇더라도 코드를 제대로 작성하지 않으면 때때로 메모리 누수 또는 메모리 부족 등의 문제가 발생한다.
그렇다면 파이썬은 어떤 식으로 메모리를 관리하고 있을까? 메모리 관리는 두 가지 방식으로 진행된다.
1. 참조  수를 세는 방식(참조 카운트)
2. 세대를 기준으로 쓸모없어진 객체를 회수하는 방식 (GC(가비지 컬렉션))

## 참조 카운트 방식의 메모리 관리
파이썬 메모리 관리의 기본은 참조 카운트이다.
모든 객체는 참조 카운트가 0인 상태로 생성되고, 다른 객체가 참조할 때마다 1씩 증가한다.
반대로 객체에 대한 참조가 끊길 때마다 1만큼 감소하다가 0이 되면 파이썬 인터프리터가 회수해간다.
이런 방식으로 객체가 더 이상 필요없어지면 즉시 메모리에서 삭제된다.

참조로 가령 다음과 같은 경우에 참조 카운트가 증가한다.
- 대입 연산자를 사용할 때
- 함수에 인수로 전달할 때
- 컨테이너 타입 객체에 추가할 때
## 순환 참조
파이썬의 참조 카운트 방식은 수많은 메모리 관리 문제를 해결해준다.하지만 참조 카운트로는 순환 참조를 해결할 수 없다.

그래서 또 다른 메모리 관리 방식이 등장한다.(GC)

GC는 참조 카운트보다 영리한 방법으로 불필요한 객체를 찾아낸다. GC는 참조 카운트와 달리 메모리가 부족해지는 시점에 파이썬 인터프리터에 의해 자동으로 호출된다. 

현재 DeZero에는 순환 참조가 존재한다. 바로 '변수'와 '함수'를 연결하는 방식에 순환 참조가 숨어있다.
Function 인스턴스는 두 개의 Variable 인스턴스(입력과 출력)를 참조한다. 그리고 출력 Variable 인스턴스는 창조자인 Function 인스턴스를 참조한다. 이때 Function 인스턴스와 Variable 인스턴스가 순환 참조 관계를 만든다.

다행히 이 순환 참조는 표준 파이썬 모듈인 weakref로 해결할 수 있다.

## weakref 모듈
파이썬에서는 weakref.ref 함수를 사용하여 약한 참조를 만들 수 있다. 약한참조란 다른 객체를 참조하되 참조 카운트는 증가시키지 않는 기능이다.

```python
import weakref
import numpy as np

a = np.array([1,2,3])
b = weakref.ref(a)
b # <weakref at 0x1057f8130; to 'numpy.ndarray' at 0x1060a3690>
```
b는 약한 참조를 갖게 했다. 이 상태로 b를 출력해보면 ndarray를 가리키는 약한 참조임을 확인할 수 있다.
그럼 앞의 코드에 바로 이어서 a=None을 실행하면 어떻게 될까?

```python
a = None
b # <weakref at 0x1057f8130; to 'numpy.ndarray' at 0x1060a3690>
```
이와 같이 ndarray 인스턴스는 참조 카운트 방식에 따라 메모리에서 삭제된다. b도 참조를 가지고 있지만 약한 참조이기 때문에 참조 카운트에 영향을 주지 못하는 것. 

이 weakref를 Dezero에도 도입해보자. 
```python
import weakref

class Function(object):
	def __call__(self, *input):
		xs = [x.data for x in inputs]
		ys = self.forward(*xs)
		if not isinstance(ys, tuple):
			ys = (ys,)
		outputs = [Variable(as_array(y)) for y in ys]

		self.generation = max([x.generation for x in inputs])
		for output in outputs:
			output.set_creator(self)
		self.inputs = inputs
		self.outputs = [weakref.ref(output) for output in outputs] #추가
		
		return outputs if len(outputs) > 1 else outputs[0]
```

```python
class Variable:
	...
	def backward(self):
		...
		while funcs:
		f = funcs.pop()
		# 수정 전 : gys = [output.grad for output in f.outputs]
		gys = [output().grad for output in f.outputs]
```

