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

-------------

>*리터럴에 관하여 처음접하기에 조금 찾아보았다. 찾아봐도 모르겠다... *
>
>#### 리터럴이란
>
>: (어구뜻이) 문자 그대로의,
>
>**리터럴**이란, 어떠한 값을 명칭하는 것이 아니라 변수 및 상수에 저장되는 **'값 자체'**를 일컫는 말이다. 일반적으로 객체 지향 언어에서 객체의 **리터럴 표기법**을 지원하게 된다. 리터럴 표기법이란, 변수를 선언함과 동시에 그 값을 지정해주는 표기법을 말한다.
>
>리터럴 표기법은 비 정규적인 방법이 아니다. 성능 저하를 불러오지도 않으며, 코드는 더 짧다. 코드가 짧으니 자바스크립트 인터프리터의 해석분량도 줄어들어 더 빨라진다.

-------------------

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

​	화살표 함수를 통하여 return문을 줄일 수 있다.  기존의 function과의 다른 점은 this 바인드 방식이다.

~~~javascript
var relationship1 = {
    name: 'zero',
    friends: ['nero', 'hero', 'xero'],
    logFriends: function(){
        var that = this;
        this.friends.forEach(function(friend){
            console.log(that.name, friend);
        });
    }
};
relationship1.logFriends();
// zero nero
// zero hero
// zero xero
const relationship2 = {
    name: 'zero',
    friends: ['nero', 'hero', 'xero'],
    logFriends() {
        this.friends.forEach(friend =>{
            console.log(this.name, friend);
        });
	}
};
relationship2.logFriends();
~~~

​	relationship1.logFriends() 안의 forEach문에서는 function 선언문을 사용했습니다. 각자 다른 함수 스코프의 this를 가지므로 that이라는 변수를 사용해서 relationship1에 간접적으로 접근하고 있습니다. 

​	하지만 relationship2, logFriends() 안의 forEach문에서는 화살표 함수를 사용했습니다. 따라서 바깥 스코프인 logFriends()의 this를 그대로 사용할 수 있습니다. 상위 스코프의 this를 그대로 물려받는 것입니다. 

### 1.5 프로미스

> "A promise is an object that may produce a single value some this in the future"
>
> 프로미스는 자바스크립트 비동기 처리에 사용되는 객체이다.
>
> 프로미스는 주로 서버에서 받아온 데이터를 화면에 표시할 때 사용합니다. 일반적으로 웹 어플리케이션을 구현할 때 서버에서 데이터를 요청하고 받아오기 위해 아래와 같은 API를 사용합니다.
>
> ~~~javascript
> $.get('url 주소/products/1', function (response){
>     //...
> });
> ~~~
>
> 위 API가 실행되면 서버에대가 '데이터 하나 보내주세요'라는 요청을 보낸다. 그런데 여기서 데이터를 받아오기도 전에 마치 데이터를 다 받아온 것 마냥 화면에 데이터를 표시하려고 하면 오류가 발생하거나 빈 화면이 뜹니다. 이와 같은 문제점을 해결하기 위한 방법 중 하나가 프로미스입니다.

​	자바스크립트와 노드에서는 주로 비동기 프로그래밍을 합니다. 특히 이벤트 주도 방식 때문에 콜백 함수를 자주 사용합니다. ES2015부터는 자바스크립트와 노드의 API들이 콜백 대신 프로미스(Promise) 기반으로 재구성됩니다. 그래서 악명 높은 콜백 헬(callback hell)을 극복했다는 평가를 받고 있습니다.



~~~javascript
const condition = true; //true면 resolve, false면 reject
const promise = new Promise((resolve, reject) => {
    if (condition) {
        resolve('성공');
    }else {
        reject('실패');
    }
});

promise
    .then((message) => {
    	console.log(message);
	})
    .catch((error) => {
    	console.error(error);
	});
~~~

​	new Promise로 프롬스를 생성할 수 있으며, 안에 resolve와 reject를 매개변수로 갖는 콜백 함수를 넣어줍니다. 이렇게 만든 promise 변수에 then과 catch 메서드를 붙일 수 있습니다. 프로미스 내부에서 resolve가 호출되면 then이 실행되고, reject가 호출되면 catch가 실행됩니다.

콜백을 프로미스로 바꾸는 예시입니다.

~~~javascript
function findAndSaveUser(User){
    Users.findOne({}, (err, user) => { // 첫 번째 콜백
        if(err) {
            return console.error(err);
        }
        user.name = 'zero';
        user.save((err) => { // 두 번째 콜백
            if(err) {
                return console.error(err);
            }
            Users.findOne({ gender: 'm' }, (err, user) => { // 세 번째 콜백
                
            })
        })
    })
}
~~~

콜백 함수가 세 번 중첩되어 있습니다.  콜백 함수가 나올 때마다 코드의 깊이가 깊어집니다. 각 콜백 함수마다 에러도 따로 처리해줘야 합니다. 이 코드를 다음과 같이 바꿀 수 있습니다.

~~~javascript
function findAndSaveUser(Users) {
    Users.findOne({})
    	.then((user) => {
        	user.name = 'zero';
        	return user.save();
    	})
        .then((user) => {
        	return Users.findOne({gender: 'm'});
    	})
        .then((user) => {
        	//생략
    	})
        .catch(err => {
        	console.error(err);
    	});
}
~~~

코드의 깊이가 더 이상 깊어지지 않습니다. then 메서드들은 순차적으로 실행됩니다. 콜백에서 매번 따로 처리해야 했던 에러도 마지막 catch에서 한번에 처리할 수 있습니다. 

### 1.6 asyn/await

​	노드 7.6 버전부터 지원되는 기능입니다. 자바스크립트 스펙은 ES2017입니다. 최신 기능이면서 정말 혁신적인 기능입니다. 특히 노드처럼 비동기 프로그래밍을 해야 할 때 도움이 많이 됩니다. 프로미스가 콜백 지옥을 해결했다지만, 여전히 코드가 장황합니다.  async/await 문법은 프로미스를 사용한 코드를 한 번 더 깔끔하게 줄여줍니다. 

~~~javascript
async function findAndSaveUser(Users){
    try{
        let user = await Users.findOne({});
        user.name = 'zero';
        user = await user.save();
        user = await Users.findOne({ gender: 'm'});
        // 생략
    } catch(error){
        console.error(error);
    }
}
~~~

코드가 확실히 짧아졌습니다. 함수는 해당 프로미스가 resolve될 때까지 기다린 뒤 다음 로직으로 넘어갑니다. 예를 들면 await Users.findOne({})이 resolve될 때까지 기다린 뒤,  user 변수를 초기화하는 것입니다.

> node.js에서 mongoDB를 사용함에 있어 asyn/await를 유용하게 사용하였다. 
>
> ~~~javascript
> const mongoose = require('mongoose');
> mongoose.connect('mongodb://localhost:27017/users', { useNewUrlParser: true, useCreateIndex: true });
> var conn = mongoose.connection;
> 
> var Schema = mongoose.Schema({
> 	email: String,
> 	password: String,
> });
> var User = mongoose.model('usersInfo',Schema);
> 
> const user1 = new User({email: 'AA@gmail.com', password: '123'});
> const user2 = new User({email: 'BB@gmail.com', password: 'pass'});
> conn.on('error',console.error.bind(console,'connection error:'));
> 
> async function run() {
>   console.log(`Mongoose: ${mongoose.version}`);
> 	await conn.dropDatabase();
> 	await user1.save();
> 	await user2.save();
> 	return conn.close();
> }
> run();
> ~~~
>
> 전체 소스코드 이지만 run함수만 보면 된다. javascript의 특성인 비동기 I/O로 인하여user1.save(), user2.save()가 완료되기 전에 conn.close()로 DB연동을 끊는다. 따라서 에러가 발생하게 된다. anync/await를 사용하게 된다면 save가 모두 끝난 이후에 conn을 끊게 된다. 

## 2. Frond-end Javascript

> Frond-end는 Angular할때 다시 정리해야겠다. 빠르게 훑는게 우선인 것 같다.

	### 2.1 AJAX

​	**AJAX** (Asynchronous Javascript And XML)는 비동기적  웹 서비스를 개발하기 위한 기법입니다. 이름에 XML이 들어가 있지만 꼭 XML을 사용해야 하는 것은 아닙니다. 요즘에는 JSON을 많이 사용합니다. 쉽게 말해 페이지 이동 없이 서버에 요청을 보내고 응답을 받는 기술입니다. 웹 사이트 중 페이지 전환 없이 새로운 데이터를 불러오는 사이트는 대부분 AJAX 기술을 사용하고 있다고 보면 됩니다. 

## 3. 노드 기능 알아보기 

### 3.1 REPL 사용하기

​	자바스크립트는 스크립트 언어이므로 미리 컴파일을 하지 않아도 즉석에서 코드를 실행할 수 있습니다. 노드도 자바스크립트와 마찬가지로 콘솔을 제공하는데, 입력한 코드를 읽고(Read), 해석하고(Eval), 결과물을 반환하고(Print), 종료할 때까지 반복(Loop)한다고 해서 **REPL** (Read Evail Print Loop)이라고 부릅니다.  VS Code에서는 `Ctrl`+<code>`</code>를 누르면  열린다.

![repl](C:\Users\koomg\Documents\BOOK\nodejs_img\repl.PNG)



### 3.2 모듈로 만들기

​	노드는 코드를 모듈로 만들 수 있다는 점에서 브라우저의 자바스크립트와는 다릅니다. 보통 파일 하나가 모듈 하나가 됩니다. 파일별로 코드를 모듈화할 수 있어 관리하기 편리합니다.

~~~javascript
//---var.js---
const odd = '홀수입니다';
const even = '짝수입니다';

module.exports = {
    odd,
    even
};
~~~

var.js에 변수 두 개를 선언했습니다. 그리고 **__module.exports__**에 변수들을 담은 객체를 대입했습니다.  다른 파일에서 이 파일을 불러오면 __module.exports__에 대입된 값을 사용할 수 있습니다.

~~~javascript
//---func.js---
const { odd, even } = require('./var');

function checkOddOrEven(num) {
    if(num % 2) { // 홀수면
        return odd;
    }
    return even;
}

module.exports = checkOddOrEven;
~~~

require 함수 안에 불러올 모듈의 경로를 적어줍니다. 파일 경로에서 js나 json 같은 확장자는 생략할 수 있습니다.

~~~javascript
//---index.js---
const { odd, even } = require('./var');
const checkNumber = require('./func');

function checkStringOddOrEven(str){
    if(str.length % 2){ // 홀수면
        return odd;
    }
    return even;
}

console.log(checkNumber(10)); // 짝수입니다
console.log(checkStringOddOrEven('hello')); // 홀수입니다
~~~

>  ES2015가 도입되면서 자바스크립트도 자체 모듈 시스템 문법이 생겼습니다. 노드의 모듈 시스템과는 문법이 조금 다릅니다. func.js를 ES2015 모듈 스타일로 바꿔보겠습니다.
>
> ~~~javascript
> //---func.mjs---
> import { odd, even } from './var';
> 
> function checkOddOrEven(num) {
>     if(num % 2){ //홀수면
>         return odd;
>     }
>     return even;
> }
> 
> export default checkOddOrEven;
> ~~~
>
> require와 module.exports가 import, export default로 바꿔었습니다. 하지만 파일의 확장자를 mjs로 지정해야하고 실행 시 node --experimental-moduls [파일명]처럼 특별한 옵션을 붙여주어야 하는 제한이 있습니다.

### 3.3 노드 내장 객체 알아보기

> 노드에서는 기본적인 내장 객체와 내장 모듈을 제공합니다. 그래서 따로 설치하지 않아도 바로 사용할 수 있습니다. 브라우저의 window 객체와 비슷하다고 보시면 됩니다. 

#### 3.3.1 global

​	브라우저의 window와 같은 전역 객체입니다. 전역 객체이므로 모든 파일에서 접근할 수 있습니다. 위에서 사용한 require 함수에서도 global.require에서 global이 생략이 ㅗ딘것입니다. 노드 콘솔에 로그를 기록하는 console 객체도 원래는 global.console 입니다.

~~~javascript
//---console---
$ node
> global
...
~~~

global 객체 안에는 수십 가지의 속성이 담겨 있습니다. 

* console.time(레이블): console.timeEnd(레이블)과 대응되어 같은 레이블을 가진 time과 timeEnd 사이의 시간을 측정합니다.

* console.log(내용): 평범한 로그를 콘솔에 표시합니다.
* console.error(에러 내용): 에러를 콘솔에 표시합니다.
* console.dir(객체, 옵션): 객체를 콘솔에 표시할 떄 사용합니다.
* console.trace(레이블):  에러가 어디서 발생했는지 추적할 수 있게 해줍니다. 보통은 에러 발생 시 에러 위치를 알려주므로 자주 사용하지는 않지만, 위치가 나오지 않는다면 사용할만 합니다.

> 내장 객체라.. 솔직히 코딩하다가 찾아보면 될 것같다. ㅋㅋㅋ 정리는 포기!

### 3.4 노드 내장 모듈 사용하기

> 노드는 웹 브라우저에서 사용되는 자바스크립트보다 더 많은 기능을 제공합니다. 운영체제 정보에도 접근할 수 있고, 클라이언트가 요청한 주소에 대한 정보도 가져올 수 있습니다. 
>
> 노드의 모듈들은 노드 버전마다 차이가 있습니다. 따라서 버전과 상관없이 안정적이고, 유용한 기능을 지닌 모듈들 위주로 설명하겠습니다.

#### 3.4.1 os

​	먼저 os모듈입니다. 웹 브라우저에 사용되는 자바스크립트는 운영체제의 정보를 가져올 수 없지만, 노드는 os 모듈에 정보가 담겨 있어 정보를 가져올 수 있습니다.

os 모듈의 대표적인 메서드를 알아봅시다.

* os.arch(): process.arch와 동일합니다. 프로세스 아키텍처 정보입니다.
* os.platform(): process.platform과 동일합니다. 운영체제 플랫폼 정보입니다.
* os.type(): 운영체제의 종류를 보여줍니다.
* os.uptime(): 운영체제 부팅 이후 흐른 시간(초)을 보여줍니다. process.uptime()은 노드 실행 시간이었습니다.
* os.hostname(): 컴퓨터의 이름을 보여줍니다
* os.release(): 운영체제의 버전을 보여줍니다.
* os.homedir(): 홈 디렉터리 경로를 보여줍니다.
* os.timdir(): 임시 파일 저장 경로를 보여줍니다.
* os.cpus(): 컴퓨터의 코어 정보를 보여줍니다.
* os.freemem(): 사용 가능한 메모리(RAM)를 보여줍니다.
* os.totalmem(): 전체 메모리 용량을 보여줍니다.

#### 3.4.2 path

폴더와 파일의 경로를 쉽게 조작하도록 도와주는 모듈입니다.

#### 3.4.3 url

인터넷 주소를 쉽게 조작하도록 도와주는 모듈입니다.

#### 3.4.4 querystring

WHATWG 방식의 url 대신 기존 노드의 url을 사용할 때 search 부분을 사용하기 쉽게 객체로 만드는 모듈입니다.

#### 3.4.5 crypto

다양한 방식의 암호화를 도와주는 모듈입니다. 몇 가지 메서드는 익혀두면 실제 서비스에도 적용할 수 있어 정말 유용합니다. 웹 서비스에서 고객의 비밀번호는 반드시 암호화해야 합니다. 

### 3.5 파일 시스템 접근하기

​	fs 모듈은 파일 시스템에 접근하는 모듈입니다. 즉, 파일을 생성하거나 삭제하고, 읽거나 쓸 수 있습니다. 웹 브라우저에서 자바스크립트를 사용할 때는 파일 다운로드와 파일 시스템 접근이 금지되어 있으므로 노드의 fs 모듈이 낯설 것입니다. 

* 파일 읽기

~~~javascript
//---readme.txt---
저를 읽어주세요.
~~~


~~~javascript
//---readFile.js---
const fs = require('fs');
                   
fs.readFile('./readme.txt', (err, data) => {
    if(err){
        throw err;
    }
    console.log(data); // <Buffer c0 fa b8 a6 20 c0 d0 be ee ba b8 bc bc bf e4 21>
    console.log(data.toString()); // 저를 읽어주세요.
});
~~~

* 파일 쓰기

~~~javascript
//---writeFile.js---
const fs = require('fs');

fs.writeFile('./writeme.txt', '글이 입력됩니다', (err)=>{
    if(err){
        throw err;
	}
});
~~~

#### 3.5.1 동기 메서드와 비동기 메서드

​	`setTimeout` 같은 타이머와 `process.nextTick` 외에도, 노드는 대부분의 메서드를 비동기 방식으로 처리합니다. 하지만 몇몇 메서드는 동기 방식으로도 사용할 수 있습니다. 특히 fs 모듈이 그러한 메서드를 많이 가지고 있습니다.

~~~javascript
//---async.js---
const fs = require('fs');

console.log('시작');
fs.readFile('./readme.txt',(err,data)=>{
    console.log('1번', data.toString());
});
fs.readFile('./readme.txt',(err,data)=>{
    console.log('2번', data.toString());
});
fs.readFile('./readme.txt',(err,data)=>{
    console.log('3번', data.toString());
});
console.log('끝');
// 시작
// 끝
// 2번 저를 읽어보세요.
// 3번 저를 읽어보세요.
// 1번 저를 읽어보세요.
~~~

시작과 끝을 제외하고는 결과의 순서가 이 책과 다를 수도 있습니다. 비동기 메서드들을 백그라운드에 해당 파일을 읽으라고만 요청하고 다음 작업으로 넘어갑니다.

> 동기와 비동기, 블록킹과 논블로킹
>
> * 동기와 비동기: 함수가 바로 return되는지 여부
> * 블로킹과 논블로킹: 백그라운드 작업 완료 여부

순서대로 출력하고 싶다면 다음 메서드를 사용할 수도 있습니다.

~~~javascript
//---sync.js---
const fs = require('fs');

console.log('시작');
let data = fs.readFileSync('./readme.txt');
console.log('1번', data.toString());
let data = fs.readFileSync('./readme.txt');
console.log('2번', data.toString());
let data = fs.readFileSync('./readme.txt');
console.log('3번', data.toString());
console.log('끝');
// 시작
// 1번 저를 읽어보세요.
// 2번 저를 읽어보세요.
// 3번 저를 읽어보세요.
// 끝
~~~

readFileSync 메서드를 사용하면 요청이 수백 개 이상 들어왔을 때 성능에 문제가 생깁니다. Sync메서드를 사용할 때는 이전 작업이 완료되어야 다음 작업을 진행할 수 있습니다.

이전 readFile()의 콜백에 다음 readFile()을 넣어주면 순서가 어긋나지는 않는다. 콜백 지옥이 펼쳐지지만 말이다. 이를 Promise나 async/await로 해결하자!

* 콜백 함수

~~~javascript
//---asyncOrder.js---
const fs = require('fs');

console.log('시작');
fs.readFile('./readme.txt',(err,data)=>{
    console.log('1번', data.toString());
    fs.readFile('./readme.txt',(err,data)=>{
    	console.log('2번', data.toString());
        fs.readFile('./readme.txt',(err,data)=>{
    		console.log('3번', data.toString());
		});
	});
});
console.log('끝');
// 시작
// 끝
// 1번 저를 읽어보세요.
// 2번 저를 읽어보세요.
// 3번 저를 읽어보세요.
~~~



