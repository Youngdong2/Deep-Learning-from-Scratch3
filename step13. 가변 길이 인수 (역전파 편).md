## 가변 길이 인수에 대응한 Add 클래스의 역전파

- 덧셈의 순전파는 입력이 2개, 출력이 1개이다. 역전파는 그 반대가 되어 입력이 1개, 출력이 2개다.
- 덧셈의 역전파는 출력 쪽에서 전해지는 미분값에 1을 곱한 값이 입력 변수의 미분이다. 
```python
class Add:
	def forward(self, x0, x1):
		y = x0 + x1
		return y

	def backward(self, gy):
		return gy, gy
```

이 코드처럼 여러 개의 값을 반환할 수 있게 하려면 역전파의 핵심 구현을 변경해야 한다.

## Variable 클래스 수정
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
				x.grad = gx

				if x.creator is not None:
					funcs.append(x.creator)

```
1. 출력 변수인 outputs에 담겨 있는 미분값들을 리스트에 담는다.
2. 함수 f의 역전파를 호출한다. 이때 `f.backward(*gys)`처럼 인수에 별표를 붙여 호출하여 리스트를 풀어준다.
3. gxs가 튜플이 아니라면 튜플로 변환한다.
4. 역전파로 전파되는 미분값을 Variable의 인스턴스 변수 grad에 저장해둔다. 여기에서 gxs와 f.inputs의 각 원소는 서로 대응 관계에 있다. 더 정확히 말하면 i번째 원소에 대해 `f.inputs[i]`의 미분값은`gxs[i]`에 대응한다. zip 함수와 for문을 이용해서 모든 Variable 인스턴스 각각에 알맞은 미분값을 설정한 것.

