## 7.1 역전파 자동화의 시작
- 역전파 자동화로 가는 길은 변수와 함수의 '관계'를 이해하는 데서 출발한다.
- 함수 입장에서 변수는 '입력'과 '출력'에 쓰인다. 
- 변수 관점에서 함수는 '창조자'이다. 창조자인 함수가 존재하지 않는 변수는 사용자에 의해 만들어진 변수로 간주된다.
```python
class Variable:
	def __init__(self, data):
		self.data = data
		self.grad = None
		self.creator = None

	def set_creator(self, func):
		self.creator = func

class Function:

	def __call__(self, input):
		x = input.data
		y = self.forward(x)
		output = Variable(y)
		output.set_creator(self) # Set parent(function)
		self.input = input
		self.output = output # Set output
		
		return output

  

	def forward(self, x):
		raise NotImplementedError()

	def backward(self, gy):
		raise NotImplementedError()
```
- 순전파를 계산하면 그 결과로 output이라는 Variable 인스턴스가 생성된다. 이때 생성된 output에 '내가 너의 창조자임'을 기억시킨다. 이 부분이 '연결'을 동적으로 만드는 기법의 핵심.
## 역전파 도전!
- 우선 y에서 b까지의 역전파를 시도해보자. 
```python
y.grad = np.array(1.0)

C = y.creator # 1. 함수를 가져온다.
b = C.input # 2. 함수의 입력을 가져온다.
b.grad = C.backward(y.grad) # 3. 함수의 backward 메서드를 호출한다.
```

- 이어서 변수 b에서 a로의 역전파를 보겠다.
```python
B = b.creator # 1. 함수를 가져온다.
a = B.input # 2. 함수의 입력을 가져온다.
a.grad = B.backward(b.grad) # 3. 함수의 backward 메서드를 호출한다.
```

똑같은 흐름이다. 구체적으로 다음과 같은 순서로 진행된다.
1. 함수를 가져온다.
2. 함수의 입력을 가져온다.
3. 함수의 backward 메서드를 호출한다.

## Backward 메서드 추가

방금 본 역전파 코드에는 똑같은 처리 흐름이 반복해서 나타났다. 변수에서 하나 앞의 변수로 거슬러 올라가는 로직이 그러했다.
그러므로 이 작업을 자동화할 수 있도록 Variable 클래스에 backward라는 새로운 메서드를 추가하겠다.
```python
class Variable:
    def __init__(self, data):
        self.data = data
        self.grad = None
        self.creator = None

    def set_creator(self, func):
        self.creator = func

    def backward(self):
        f = self.creator  # 1. Get a function
        if f is not None:
            x = f.input  # 2. Get the function's input
            x.grad = f.backward(self.grad)  # 3. Call the function's backward
            x.backward()
```