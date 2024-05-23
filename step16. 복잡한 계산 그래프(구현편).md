## 세대 추가
먼저 Variable 클래스와 Function 클래스에 인스턴스 변수 generation을 추가해보자.
몇 번째 세대의 함수(또는 변수)인지 나타내는 변수이다. 

```python
class Variable:
	def __init__(self, data):
		if data is not None:
			if not isinstance(data, np.adarray):
				raise TypeError('{}은 지원하지 않습니다.'.format(type(data)))
	
		self.data = data
		self.grad = None
		self.creator =None
		self.generation = 0 # 세대 수를 기록하는 변수

	def set_creator(self, func):
		self.creator = func
		self.generation = func.generation + 1 # 세대를 기록한다(부모 세대 + 1)

	...
	
```
- Variable 클래스는 generation을 0으로 초기화한다. 그리고 set_creator 메서드가 호출될 때 부모 함수의 세대보다 1만큼 큰 값을 설정한다.


다음 차례는 Function 클래스이다. Function 클래스의 generation은 입력 변수와 같은 값으로 설정한다.
입력 변수가 둘 이상이라면 가장 큰 generation의 수를 선택한다.
```python
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
		self.outputs = outputs

		return outputs if len(outputs) > 1 else outputs[0]
```

## Variable 클래스의 backward
```python
class Variable:
	...

	def backward(self):
		if self.grad is None:
			self.grad = np.ones_like(self.data)

		funcs = []
		seen_set = set()

		def add_func(f):
			if f not in seen_set:
			funcs.append(f)
			seen_set.add(f)
			funcs.sort(key=lambda x: x.generation)

		add_func(self.creator)

		while funcs:
			f = funcs.pop()
			gys = [output.grad for output in f.outputs]
			gxs = f.backward(*gys)
			if not isinstance(gxs, tuple):
				gxs = (gxs,)
	
			for x, gx in zip(f.inputs, gxs):
				if x.grad is None:
					x.grad = gx
				else:
					x.grad = x.grad + gx

				if x.creator is not None:
					add_func(x.creator)
```

가장 큰 변화는 새로 추가된 add_func 함수이다. 이 함수가 함수 리스트를 세대 순으로 정렬하는 역할을 한다.
그 결과 funcs.pop()은 자동으로 세대가 가장 큰 함수를 꺼내게 된다.