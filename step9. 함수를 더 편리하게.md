## 파이썬 함수로 이용하기

지금까지는 함수를 파이썬 클래스로 정의해 사용했다. 그래서 가령 Square클래스를 사용하는 계산을 하려면 코드를 다음처럼 작성해야 했다.
```python
x = Variable(np.array(0.5))
f = Square()
y = f(x)
```
이와 같이 클래스의 인스턴스를 생성한 다음, 이어서 그 인스턴스를 호출하는 두 단계로 구분해 진행해야 한다.
이는 조금 번거롭다.

```python
def Square(x):
	f = Square()
	return f(x)

def exp(x):
	f = exp()
	return f(x)
```
위와 같이 함수 형태로 사용하면 코드를 간소화할 수 있다.

## backward 메서드 간소화

두 번째 개선은 역전파 시 사용자의 번거로움을 줄이기 위한 것.
구체적으로는 y.grad = np.array(1.0) 부분을 생략하려 한다. 

```python
class Variable:
	...

    def backward(self):
		if self.grad is None:
			self.grad = np.ones_like(self.data) # 위 두줄 추가

        funcs = [self.creator]
        while funcs:
	        f = funcs.pop()
	        x, y = f.input, f.output 
	        x.grad = f.backward(y.grad) 
	        
		    if x.creator is not None:
			    funcs.append(x.creator)
```

이와 같이 만약 변수의 grad가 None이면 자동으로 미분값을 생성한다. 

## ndarray만 취급하기
Variable에 ndarray 인스턴스만 담는 상자가 되도록 해보자.

```python
class Variable:
	def __init__(self, data):
		if data is not None:
			if not isinstance(data, np.ndarray):
				raise TypeError('{}은 지원하지 않습니다.'.format(type(data)))

		self.data = data
		self.grad = None
		self.creator = None
```

