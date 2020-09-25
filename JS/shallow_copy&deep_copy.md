JS 깊은 복사와 얕은 복사
================================

## 얕은복사(Shallow Copy)
얕은복사는 복사된 값이 원래값을 한개라도 참조하고 있는방식으로의 복사를 말한다.

```javascript
//array
let arr1 = [1,2,3,4,5]
let arr2 = arr1.slice()
arr2[0] = 999;

console.log(arr1); //[ 1, 2, 3, 4, 5 ]
console.log(arr2); //[ 999, 2, 3, 4, 5 ]

//object
let obj1 = { a: 1, b: 2 }
let obj2 = Object.assign({}, obj1)
obj2.a = 999
console.log(obj1) //{ a: 1, b: 2 }
console.log(obj2) //{ a: 999, b: 2 } 
```

Array의 slice 메서드나 Object의 assign 메서드는 모두 1차원에서만 적용된다 (얕은복사의 특징).  
``` javascript
//array
let a = [[9],1,2,3];
let b = a.slice();
b[0][0] = 999;
console.log(a); //[ [ 999 ], 1, 2, 3 ]
console.log(b); //[ [ 999 ], 1, 2, 3 ]

//object
let obj1 = {
    a: 1,
    b: {
        c: 2,
        d: 3
    }
}
let obj2 = Object.assign({}, obj1)
obj2.b.c = 999
console.log(obj1) //{ a: 1, b: { c: 999, d: 3 } }
console.log(obj2) //{ a: 1, b: { c: 999, d: 3 } }
```

## 깊은복사(Deep Copy)
깊은복사는 원본과의 참조가 완전히 끊어진 객체를 말한다.
```javascript
let obj1 = {
    a: 1,
    b: {
        c: 2,
        d: 3
    }
}
let obj2 = JSON.parse(JSON.stringify(obj1))
obj2.b.c = 999;

console.log(obj1); // { a: 1, b: { c: 2, d: 3 } }
console.log(obj2); // { a: 1, b: { c: 999, d: 3 } }
```