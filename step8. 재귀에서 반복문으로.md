## 현재의 Variable 클래스
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

- 이 backward 메서드에서 눈에 밟히는 부분은 하나 앞 변수의 backward를 호출하는 코드이다.

## 반복문을 이용한 구현
```python
class Variable:
    def __init__(self, data):
        self.data = data
        self.grad = None
        self.creator = None

    def set_creator(self, func):
        self.creator = func

    def backward(self):
        funcs = [self.creator]
        while funcs:
	        f = funcs.pop() # 함수를 가져옴
	        x, y = f.input, f.output # 함수의 입력, 출력을 가져옴
	        x.grad = f.backward(y.grad) # backward 메서드를 호출
	        
		    if x.creator is not None:
			    funcs.append(x.creator) # 하나 앞의 함수를 리스트에 추가
```

- 주목할 점은 처리해야 할 함수들을 funcs 이라는 리스트에 차례로 집어넣는 다는 점.
- while 블록 안에서 funcs.pop()을 호출하여 처리할 함수 f를 꺼내고, f의 backward 메서드를 호출한다. 