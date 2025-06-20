
일반적으로 사용할 때는 

for line in logs:

이렇게 사용한다 로그의 내용에 하나씩 바로 접근할 수 있다

인덱스가 필요한 경우에는

for i in range(len(logs)):
	line = logs[i]

이렇게 사용한다 인덱스와 요소에 한번에 접근할 수 있다

하지만 복잡하기 때문에 반복자를 사용한다

for i,line in enumerate(logs):

편의성도 있게 로그의 인덱스와 요소에 한번에 접근할 수있다