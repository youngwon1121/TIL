Arrow Function
=======================
화살표 함수를 사용할때의 특징을 정리해보자.  

- 화살표함수는 항상 익명이다.

- 코드의 축약이 가능하다.

- 화살표함수의 this는 렉시컬 스코프의 this를 갖는다.




### 렉시컬 스코프의 this를 갖는다
아래의 예시처럼 function키워드를 통해 함수를 정의할 시, 생성자 함수로 사용된 **Person안의 this** 와 Person안의 **gettingOld안의 this**가 서로 다르다.

```javascript
function Person() {
	console.log(this) //Person안의 this
    this.age = 0;
    function gettingOld(){
        console.log(this); //gettingOld안의 this
        this.age++;
    };
    gettingOld();
}
var p = new Person();
```

따라서 같은 this를 사용하기 위해 this를 다른 변수에 할당하는 방식을 사용했다.
```javascript
function Person() {
	this.age = 0;
	var self = this; //this를 다른변수에 할당
    function gettingOld(){
        self.age++;
    };
    gettingOld();
}
var p = new Person();
```

화살표함수를 사용하면 this를 다른 변수에 할당하지 않고 사용할 수 있다.  
화살표함수는 자신만의 this가 없어서, 함수를 둘러싸는 렉시컬 범위의 this가 사용된다.
```javascript
function Person() {
	this.age = 0;
    gettingOld = () => {
        this.age++;
    };
    gettingOld();
}
var p = new Person();
```