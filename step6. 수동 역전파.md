## Variable 클래스 추가 구현
```python
class Variable:
	def __init__(self, data):
		self.data = data
		self.grad = None
```

- 이와 같이 새로 grad라는 인스턴스 변수를 추가했다. 
- 인스턴스 변수인 data와 grad는 모두 넘파이의 다차원 배열이라고 가정한다.
- 또한 grad는 None으로 초기화해둔 다음, 나중에 실제로 역전파를 하면 미분값을 계산하여 대입한다.
## Function 클래스 추가 구현
```python
class Function:
	def __call__(self, input):
		x = input.data
		y = self.forward(x)
		output = Variable(y)
		self.input = input # 입력 변수를 기억한다.
		return output

	def forward(self, x):
		raise NotImplementedError()

	def backward(self, gy):
		raise NotImplementedError()
```
- 코드에서 보듯 `__call__` 메서드에서 입력된 input을 인스턴스 변수인 self.input에 저장한다.
- 이렇게 해서 나중에 backward 메서드에서 함수에 입력한 변수가 필요할 때 self.input에서 가져와 사용할 수 있다.
