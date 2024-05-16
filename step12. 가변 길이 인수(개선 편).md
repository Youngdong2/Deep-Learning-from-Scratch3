## 첫 번째 개선: 함수를 사용하기 쉽게

```python
class Function:
	def __call__(self, *inputs): # 1 별표를 붙인다
		xs = [x.data for x in inputs]
		ys = self.forward(xs)
		outputs = [Variable(as_array(y)) for y in ys]

		for output in outputs:
			output.set_creator(self)
		self.inputs = inputs
		self.outputs = outputs

		# 2. 리스트의 원소가 하나라면 첫 번째 원소를 반환한다.
		return outputs if len(outputs) > 1 else outputs[0]
```

1. 함수를 정의할 때 인수 앞에 별표를 붙였다. 이렇게 하면 리스트를 사영하는 대신 임의 개수의 인수를 건내 함수를 호출할 수 있다.
2. outputs에 원소가 하나뿐이라면 리스트가 아니라 그 원소만을 반환한다.

## 두 번째 개선: 함수를 구현하기 쉽도록

```python
class Function:
	def __call__(self, *inputs): 
		xs = [x.data for x in inputs]
		ys = self.forward(*xs) # 1 별표를 붙여 언팩
		if not isinstance(ys, tuple): # 2 튜플이 아닌 경우 추가 지원
			ys = (ys,)
		outputs = [Variable(as_array(y)) for y in ys]

		for output in outputs:
			output.set_creator(self)
		self.inputs = inputs
		self.outputs = outputs

		return outputs if len(outputs) > 1 else outputs[0]
```

1. 함수를 호출할 때 별표를 붙였는데, 이렇게 하면 리스트 언팩이 이루어진다. 즉, `self.forward(*xs)`는 self.forward(x0, x1)과 같다.
2. ys가 튜플이 아닌 경우 튜플로 변경한다. 이제 forward 메서드는 반환 원소가 하나뿐이라면 해당 원소를 직접 반환한다.

이상의 수정으로 Add클래스를 다음처럼 구현할 수 있다.
```python
class Add(Function):
	def forward(self, x0, x1):
		y = x0 + x1
		return y
```
