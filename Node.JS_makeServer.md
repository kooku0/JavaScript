# Node.JS

> 이제부터 제대로 서버를 만들어 보겠다.
>
> 이렇게 정리하면서 하려니까 내 스퇄이 아니지만, 언젠가는 도움이 되겠지

> #### 목차
>
> **4. http 모듈로 웹 서버 만들기**
>
> + 요청과 응답 이해하기
> + 쿠키와 세션 이해하기
> + REST API와 라우팅
> + http와 http2
> + cluster

## 4. http 모듈로 웹 서버 만들기

### 4.1 요청과 응답 이해하기

​	서버에는 요청을 받는 부분과 응답을 보내는 부분이 있어야 합니다. 요청과 응답은 이벤트 방식이라고 생각하면 됩니다. 클라이언트로부터 요청이 왔을 떄 어떤 작업을 수행할지 이벤트 리스너를 미리 등록해두어야 합니다.

~~~javascript
const http = require('http');

const server = http.createServer((req, res) => {
    res.write('<h1>Hello Node!');
    res.end('<p>Hello Server!</p>');
});
server.listen(8080);
server.on('listening', () => {
    console.log('8080번 포트에서 서버 대기 중입니다!');
});
server.on('error', (error) => {
    console.error(error);
});
~~~

node.js에서 가장 먼저 접하게 되는 코드입니다. listen 메서드에 콜백 함수를 넣는 대신, listening 이벤트 리스너를 붙였습니다. 추가로 error 이벤트 리스너도 붙여보았습니다.

> 포트만 다르게 해서 동시에 여러 노드 서버를 실행할 수도 있습니다.

### 4.2 쿠키와 세션 이해하기

​	클라이언트에서 보내는 요청에는 누가 요청을 보내는지 모른다는 큰 단점이 있습니다. 물론 요청을 보내는 IP주소나 브라우저의 정보를 받아올 수는 있습니다. 하지만 여러 컴퓨터가 공통으로 IP주소를 가지거나, 한 컴퓨터를 여러 사람이 사용할 수도 있습니다.

이번에는 로그인을 구현하기 위한 쿠키와 세션에 대하여 알아보겠습니다. 로그인한 후에 새로고침을 해도 로그아웃이 되지 않을 것입니다. 이건 클라이언트가 서버에게 여러분이 누구인지를 지속적으로 알려주고 있기 때문입니다.

**Client가 누구인지 기억하기 위해서, Server는 요청에 대한 응답을 할 때 쿠키라는 것을 같이 보내줍니다.** 쿠키는 name=zerocho 같이 단순한 'key-value'의 쌍입니다. 서버로부터 쿠키가 오면 웹 브라우저는 쿠키를 저장해두었다가 요청할 때마다 쿠키를 동봉해서 보내줍니다. 
**서버는 요청에 들어 있는 쿠키를 읽어서 사용자가 누구인지 파악합니다.**

브라우저는 쿠키가 있다면 자동으로 동봉해서 보내주므로 따로 처리할 필요가 없습니다. 서버에서 브라우저로 쿠키를 보낼 때만 여러분이 코드를 작성하여 처리하면 됩니다.

즉, **Server는 미리 Client에 요청자를 추정할 만한 정보를 쿠키로 만들어 보내고, 그 다음부터는 클라이언트로부터 쿠키를 받아 요청자를 파악합니다.** 쿠키가 여러분이 누구인지를 추적하고 있는 것입니다. 

쿠키는 요청과 응답의 해더에 저장됩니다. 요청과 응답은 각각 헤더와 본문을 가집니다. 

~~~javascript
const http = require('http');

const parseCookies = (cookie = '') =>
    cookie
    .split(';')
    .map(v => v.split('='))
    .map(([k, ...vs]) => [k, vs.join('=')])
    .reduce((acc, [k, v]) => {
        acc[k.trim()] = decodeURIComponent(v);
        return acc;
    }, {});


const server = http.createServer((req, res) => {
    const cookies = parseCookies(req.headers.cookie);
    console.log(req.url, cookies);
    res.writeHead(200, {'Set-Cookie': 'mycookie=test'});
    res.end('<p>Hello Cookie!</p>');   
});

server.listen(8080);
server.on('listening', () => {
    console.log('8080번 포트에서 서버 대기 중입니다!');
});

server.on('error', (error) => {
    console.error(error);
});
~~~

```html
/ { '': '' }
/favicon.ico { mycookie: 'test' }
```

만약 실행 결과가 위와 다르다면 브라우저의 쿠키를 모두 제거한 후에 다시 실행해야 합니다.

요청은 분명 한 번만 보냈는데 두 개가 기록되어 있습니다. /favicon.ico는 요청한 적이 없는데 말이죠. 첫 번째 요청('/')에서는 쿠키에 대한 정보가 없다고 나오고, 두 번째 요청('/favicon.ico')에서는 { mycookie: 'test' }가 기록되었습니다.

![favicon](.\nodejs_img\favicon.PNG)

빨간 네모가 favicon입니다.

브라우저는 favicon이 뭔지 HTML에서 유추할 수 없으면 서버에 favicon 정보에 대한 요청을 보냅니다. 현재 예제에서 HTML에 favicon에 대한 정보를 넣어두지 않았으므로 브라우저가 추가로 favicon.ico를 요청한 것입니다. 

> favicon에 대한 내용은 추후에 다루도록 하겠습니다.

