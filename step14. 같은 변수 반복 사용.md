현재까지의 구현은 문제가 있다. 같은 변수를 반복해서 사용할 경우 의도대로 동작하지 않을 수 있다는 것.
즉, 미분값이 틀리게 나온다는 것이다.

## 문제의 원인
원인은 Variable 클래스의 다음 위치에 있다.
```python
class Variable:
	...

	def backward(self):
		if self.grad is None:
			self.grad = np.ones_like(self.data)

		funcs = [self.creator]
		while funcs:
			gys = [output.grad for output in f.ooutputs] # 1
			gxs = f.backward(*gys) # 2
			if not isinstance(gxs, tuple): # 3
				gxs = (gxs,)

			for x, gx in zip(f.inputs, gxs): # 4
				x.grad = gx # 여기가 문제!

				if x.creator is not None:
					funcs.append(x.creator)

```

이 코드에서 알 수 있듯이 현재 구현에서는 출력 쪽에서 전해지는 미분값을 그대로 대입한다.
따라서 같은 변수를 반복해서 사용하면 전파되는 미분값이 덮어 써지는 것.

## 해결책
```python
class Variable:
	...

	def backward(self):
		if self.grad is None:
			self.grad = np.ones_like(self.data)

		funcs = [self.creator]
		while funcs:
			gys = [output.grad for output in f.ooutputs] # 1
			gxs = f.backward(*gys) # 2
			if not isinstance(gxs, tuple): # 3
				gxs = (gxs,)

			for x, gx in zip(f.inputs, gxs): # 4
				if x.grad is None:
					x.grad = gx
				else:
					x.grad = x.grad + gx

				if x.creator is not None:
					funcs.append(x.creator)

```