Javascript에서의 깊은 복사와 얕은 복사
================================

### javascript에는 7가지의 타입이 있다.
+ number
+ string
+ object
+ function
+ boolean
+ object
+ undefined

***

# 깊은복사
우리는 아래처럼 평소 필요한 값을 위해 변수간 복사를 수행한다.
~~~
let a = 5;
let b = a;
b=7;
console.log(a)
console.log(b)
~~~
위의 예제의 콘솔에 뭐가 찍힐지 생각해보면 a의 값을 b에게 복사하고 b에 새로운 7이라는 값을 할당하였으니 5와 7이 나올것이다. 이러한 실행을 깊은 복사라고한다. 아직 너무 당연한 부분이라 이게 어쨌다는 것인지 할 수도 있다.
깊은 복사는 number, string, boolean타입등에서 나타나고 object에서는 얕은 복사가 나타난다.
아래 object에서 비슷한과정을 실행하고 비교해보자

# 얕은복사
~~~
let a = [1, 2, 3]
let b = a
b[0] = 999
console.log(a)
console.log(b)
~~~
위의 코드를 깊은 복사처럼 생각한다면 첫번째 로그는 [1,2,3] 두번째 로그는[999,2,3]이 나온다고 생각할 것이다. 하지만 위 코드의 결과는 둘 다 [999,2,3]이 나올것이다.

즉, 얕은복사는 c언어의 포인터처럼 지정한값의 레퍼런스를 참조하는(참조 메모리를 복사하는)것이다.

## 깊은복사하는 방법
배열객체
~~~
let old = [1,2,3,4,5]
let new = a.slice()
~~~

object
~~~
let new = Object.assign({}, old);  
~~~