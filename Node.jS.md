# Node.jS

> *Node.js* 책을 요약한 문서입니다.
>
> 구조나 특징은 Link: [Node.js 이벤트 루프, 타이머, `process.nextTick()`](https://nodejs.org/ko/docs/guides/event-loop-timers-and-nexttick/) 공식문서를 참조차세요.  시간이 남으면 다시 정리할 계획입니다.

##  1. 알아두어야할 자바스크립트

## 1.1 const, let

​	var은 function scope, const와 let은 block scope이다. 다음의 예를 본다면 이해가 쉽다.

~~~javascript
if(true){
    var x = 3;
}
console.log(x); //3

if(true){
    const y = 3;
}
console.log(y); //Uncaught ReferenceError: y is not defined
~~~

  var은 function scope를 가지므로 if문의 블록과 관계없이 접근할 수 있다. 하지만 const와 let은 block scope를 가지므로 블록 밖에서는 변수에 접근할 수 없다.  이를 통하여 hosting과 같은 문제도 해결되고 코드 관리도 수월해졌다.

> hosting이란.... 나중에 적어야겠다.

   const는 한 번 대입하면 다른 값을 대입할 수 없다. 또한 초기화 시 값을 대입하지 않으면 에러가 발생한다. 따라서 기본적으로 변수 선언 시에는 const를 사용하고, 다른 값을 대입해야 하는 상황이 생겼을 때 let을 사용하는게 바람직하다.

### 1.2 템플릿 문자열

​	ES2015부터는 다음과 같은 것이 지원된다.

~~~javascript
const num1 = 1;
const num2 = 2;
const result1 = 3;
const string1 = `${num1} 더하기 ${num2}는 '${result1}'`;
console.log(string1); // 1 더하기 2 는 '3'
~~~

   깔끔하다.. 그리고 백틱(`)을 사용하기 때문에 큰따옴표나 작은따옴표와 함께 사용할 수도 있다.

### 1.3 객체 리터럴

​	객체 리터럴에는 편리한 기능들이 추가되었다.

~~~javascript
var sayNode = function(){
    console.log('Node');
};
var es = 'ES';
const newObject = {
	sayJS(){
		console.log('JS');
	},
	sayNode,
	[es + 6]: 'Fantastic',
};
newObject.sayNode(); // Node
newObject.sayJS(); // JS
console.log(newObject.ES6); // Fantastic
~~~

​	객체에 동적으로 속성을 추가할 수 있다. 위의 sayNode와 newObject를 비교해 보자.  

### 1.4 화살표 함수

​	화살표 함수(arrow function)라는 새로운 함수가 추가되었으며, 기존의 function() {}도 그대로 사용할 수 있다. 

~~~ javascript
function add1(x,y){
	return x + y;
}

const add2 = (x,y) => {
	return x + y;
}

const add3 = (x,y) => x + y;

const add4 = (x,y) => (x + y);
~~~

​	화살표 함수를 통하여 return문을 줄일 수 있다. 기존의 function과의 다른 점은 this 바인드 방식이다.