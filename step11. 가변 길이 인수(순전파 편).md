지금까지는 함수에 입출력 변수가 하나씩인 경우만 생각해왔다. 
이번 step에서는 가변 길이 입출력을 처리할 수 있도록 확장한다. Function 클래스를 조금 수정하면 대응할 수 있을 것 같다.

## Function 클래스 수정
가변 길이 입출력을 표현하려면 변수들을 리스트에 넣어 처리하면 편할 것 같다.
즉, Function 클래스는 지금까지처럼 '하나의 인수'만 받고 '하나의 값'만 반환하는 것이다. 대신 인수와 반환값의 타입을 리스트로 바꾸고, 필요한 변수들을 이 리스트에 넣으면 된다.

현재 Function 클래스가 어떻게 구현되어 있는지부터 확인하자.
```python
class Function:
	def __call__(self, input):
		x = input.data # 1
		y = self.forward(x) # 2
		output = Variable(as_array(y)) # 3
		output.set_creator(self) # 4
		self.input = input
		self.output = output
		return output

	def forward(self, x):
		raise NotImplementedError()

	def backward(self, gy):
		raise NotImplementedError()
```

Function의 __call__ 메서드는
1. Variable이라는 상자에서 실제 데이터를 꺼낸 다음
2. forward 메서드에서 구체적인 계산을 한다.
3. 그리고 계산결과를 Variable에 넣고
4. 자신이 창조자라고 원산지 표시를 한다.

이상을 염두해 두고 call 메서드의 인수와 반환값을 리스트로 바꿔보자.
```python
class Function:
	def __call__(self, inputs):
		xs = [x.data for x in inputs]
		ys = self.forward(xs)
		outputs = [Variable(as_array(y)) for y in ys]

		for output in outputs:
			output.set_creator(self)
		self.inputs = inputs
		self.outputs = outputs
		return outputs

	def forward(self, xs):
		raise NotImplementedError()

	def backward(self, gys):
		raise NotImplementedError()

```

이어서 새로운 Function 클래스를 사용하여 구체적인 함수를 구현해보자. 

## Add 클래스 구현
```python
class Add(Function):
	def forward(self, xs):
		x0, x1 = xs
		y = x0 + x1
		return (y,)
```

Add 클래스의 인수는 변수가 두 개 담긴 리스트이다. 따라서 x0, x1 = xs 형태로 리스트에서 원소 두 개를 꺼냈다. Add클래스는 다음과 같이 사용할 수 있다.

```python
xs = [Variable(np.array(2)), Variable(np.array(3))]
f = Add()
ys = f(xs)
y = ys[0]
print(y.data) # 5 출력
```

하지만 위 코드는 귀찮은 느낌이 든다. 사용하는 사람에게 입력 변수를 리스트에 담아 건네주라고 요구하거나 반환값으로 튜플을 받게 하는 것은 자연스럽지 않기 때문이다.

다음 단계에서 더 자연스러운 코드로 구현해보자.