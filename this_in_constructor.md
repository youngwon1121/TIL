JAVASCRIPT의 this에 관하여
=======================

생성자에서의 this
```
function A(){
	this.name = "youngwon"
	console.log(this instanceof A)
}
```
위 코드이후에 new A()를 호출하면 name프로퍼티를 가진 객체를 반환하고 console.log는 true를 출력한다.

```
function A(){
	this.name = "youngwon"
	console.log(this instanceof A)
	['aaa'].forEach(function(name) => {
		console.log(this instanceof A)
	})
}
```
배열을 반복하는 forEach안에 내부함수가있는것에 주목하자.
이때 this는 무엇일까?

다시 new A()를 호출하면 내부함수의 콘솔은 false를 출력하는것을 볼 수 있다.
내부함수에서의 this는 window객체가 되기 때문이다.(웹상에서)

우리는 지금까지 이런경우에 내부함수에서 this를 사용하기위해서 아래와 같은 코드를 사용하였다.
```
var self = this;
```

그 결과 이런코드가 나오게 되었다.
```
function A(){
	this.name = "youngwon"
	var self = this
	['aaa'].forEach(function(name) => {
		console.log(self instanceof A)
	})
}
```
위 코드의 내부함수에서는 self가 생성자의 this역할을 잘 해주고있다.

하지만 ES6부터는 좀 더 헷갈리지 않게 사용할 수 있는 arrow function이 나타났다.
arrow function을 적용하면 var self = this라는 코드를 사용하지않아도 된다.
```
function A(){
	this.name = "youngwon"
	['aaa'].forEach(name => {
		console.log(this instance of A)
	})
}
```