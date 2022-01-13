---
layout: post
author: Bruce Lee
title: Node.js 마이크로서비스 코딩 공작소
---

# Node.js 마이크로서비스 코딩 공작소
## **모놀리식 아키텍처란**

- 모놀리식 아키텍처는 널리 활용하는 전통적인 아키텍처로 하나의 애플리케이션 안에 모든 컴포넌트를 포함하는 구조이다. 구조가 단순해 개발과 배포가 간편하다는 장점이 있다.
- 반면에 분산 처리가 비효율적이며 코드를 관리하기 어렵고 코드 구조가 갈수록 복잡해지며 새로운 기술을 적용하기가 어렵다는 단점이 있다.

### **분산 아키텍처의 필요성**

- 코드 수정에 부담이 없고 기존 코드에 영향을 주지 않으며, 필요한 기능만 분산 처리할 수 있는, 유기적으로 동작하는 아키텍처가 필요했다.

## Node.js 이해
## **동기 프로그래밍**

- Node js 는 모든 함수와 모듈이 비동기 프로그래밍을 기본으로 한다.

```jsx
function func(callback){
	callback("callback!!");
}

func((param) => {
	console.log(param);
});
```

- 위 코드는 동일한 스레드 위에서 동기적으로 동작한다. func 함수 내부에서 비동기적으로 콜백하려면 process.nextTick 함수를 이용해야한다.

```jsx
function func(callback){
	process.nextTick(callback, "callback!");
}

try{
	func((param)=>{
		a.a = 0;
	});
} catch(e){
	console.log("exception !!");
}
```

- 위 코드의 경우, try catch문의 실행되지 않고 프로세스 실행 에러가 발생한다. process.nextTick 함수는 비동기 처리를 위해 Node.js 내부의 스레드 풀로 다른 스레드 위에서 콜백 함수를 동작하게 된다. try catch 문은 같은 스레드 위에서만 동작하기 때문에 서로 다른 스레드 간의 예외 처리가 불가능하다. 이처럼 process.nextTick 함수를 이용하면 Node js 가 CPU 를 효율적으로 사용하는 대신 try catch 문만으로는 예외 처리가 불가능하다.

## **싱글 스레드 프로그래밍**

- Node js 는 싱글 스레드 기반으로 동작한다. 하지만 싱글 스레드라고 해서 모두 같은 스레드 위에서 동작하지는 않는다.
- 비동기 호출을 할 경우, 함수를 호출한 영역과 콜백을 처리하는 영역이 각기 다른 스레드 위에서 동작한다. 이때 try catch 문으로 모든 예외를 처리하기에는 무리가 있다.
- Node js 는 모든 스레드에서 예외 처리를 할 수 있도록 uncaughtException 이벤트를 제공한다.

```jsx
function func(callback){
	process.nextTick(callback, "callback!!");
}

try{
	func((param) => {
		a.a=0;
	});
} catch(e){
	console.log("exception!! : "+ e.getMessage);
}

process.on("uncaughtException", (error) => {
	console.log("uncaughtException!! " + error);
});
```

### **HTTP 서버**

- HTTP 는 전 세계에서 가장 인기 있는 네트워크 프로토콜 이다. 월드와이드웹으로 이미 많은 시스템에서 활용 중이며 HTML 뿐만 아니라 JSON, XML 등 다양한 문서 포맷을 사용하기에도 편리하다.
- 대용량 패킷 전달, 상태 관리, 보안 처리 등 각종 네트워크 이슈에 대한 대비책도 마련되어 있으며 요즘에는 RESTful 설계 방식이 널리 활용된다.
- Node.js 에서는 기본 모듈로 http 모듈을 제공하는데 이를 이용하면 매우 쉽게 HTTP 서버를 만들 수 있다.

```jsx
const http = require('http');
var server = http.createServer((req, res) =>{
  res.end("hello world");
});

server.listen(8000);
```

- 복잡한 웹 어플리케이션을 개발할 때는 UI 템플릿, 쿠키 처리, 라우팅 처리 등 웹 애플리케이션을 개발하는 데 필요한 일련의 기능을 제공하는 Express 라는 유명한 확장 모듈을 이용할 수 있다.

### **HTTP 클라이언트**

- Node.js 에서는 http 모듈을 이용해 HTTP 서버와 HTTP 클라이언트 개발에 필요한 모든 API 를 제공한다.
- 하기와 같이 간단하게 클라이언트를 구성가능하다.

```jsx
var http = require('http');

var options = {
  host : "127.0.0.1",
  port: 8000,
  path: "/"
};

var req = http.request(options, (res) => {
  var data = "";
  res.on('data', (chunk)=>{
    data+=chunk;
  });

  res.on('end', ()=>{
    console.log(data);
  });
});
req.end();
```

## Monopoly 서버와 클라이언트 만들기

- 하기와 같이 http 모듈을 사용하면 간단한 HTTP 서버를 만들 수 있다.

```jsx
const http = require('http');
var server = http.createServer((req, res) =>{
  res.end("hello world");
});

server.listen(8000);
```

- 테스트용 HTTP 클라이언트의 경우도, http 모듈을 사용하여 host, port, path 정보를 넣고 request 함수를 호출하고, 받은 응답 정보를 콜백 함수를 이용하여 처리 할 수 있다.

```jsx
var http = require('http');

var options = {
  host : "127.0.0.1",
  port: 8000,
  path: "/"
};

var req = http.request(options, (res) => {
  var data = "";
  res.on('data', (chunk)=>{
    data+=chunk;
  });

  res.on('end', ()=>{
    console.log(data);
  });
});
req.end();
```

## TCP 서버 만들기

- TCP 는 인터넷 프로토콜 스위트의 핵심 프로토콜 중 하나로 TCP/IP 라는 명칭으로 많이 사용되며 연결 과정이 필요한 신뢰할 수 있는 통신에 사용되는 프로토콜이다.
- Node.js 에서는 net 이라는 기본 모듈을 사용하여 TCP 서버와 TCP 클라이언트 관련 API 를 제공한다.

```jsx
var net = require('net');
var server = net.createServer((socket) => {
  socket.end("hello wolrd");
});

server.on('error', (err) =>{
  console.log(err);
});

server.listen(9000, ()=>{
  console.log('listen', server.address());
});
```

- TCP 헤더 정보는 구현하는 시스템마다 자유롭게 설계하기 때문에 TCP 서버에 접속해 테스트하려면 HTTP 와는 다르게 클라이언트를 직접 구현해서 테스트해야 한다. TCP 서버에 접속할 수 있는 클라이언트를 만들어 봅시다.

```jsx
var net = require('net');
var options = {
  port: 9000,
  host: "127.0.0.1"
};

var client = net.connect(options, ()=>{
  console.log("connected");
});

client.on('data',(data)=>{
  console.log(data.toString());
});

client.on('end', ()=>{
  console.log("disconnected");
});
```

## Monopoly REST API

- REST 는 HTTP 프로토콜의 주요 저자인 로이 필딩이 2000년 자신의 박사 학위 논문에서 소개했다. REST 는 기존 RPC 나 SOAP 등 복잡한 프로토콜로 통신하는 것 보다 이미 널리 사용되는 HTTP 프로토콜로 통신하는 것이 더 효율적이라는 내용이다. REST 는 자원 지향 구조로 접근하고자 하는 자원에 고유한 URI 를 부여하는 방식이다. ( POST 생성 / GET 조회 / PUT 수정 / DELETE 삭제 )
- 상품관리 API 를 설계해보자, URI 는 /goods 로 지정하고 입력파라미터에 따라 등록 결과 혹은 조회 결과 등을 출력해야 한다.
- 회원관리 API 를 설계해보자, URI 는 /members 로 지정하고 입력파라미터에 따라 등록 결과 혹은 조회 결과 등을 출력해야 한다.
- DB 의 경우, Maria DB 를 통해 설정하며 연동하기 위해서는 mysql 모듈을 사용한다.
- createServer 함수에서 전달되는 req 파라미터를 통해 메서드 정보와 URI 정보를 가져올 수 있으며, URI 정보를 파싱하기 위해서는 url 모듈과 querystring 모듈을 이용하여 파싱할 수 있다.

```jsx
const http = require('http');
const url = require('url');
const querystring = require('querystring');

const members = require('./monolithic_members.js');
const goods = require('./monolithic_goods.js');
const purchases = require('./monolithic_purchases.js');

/**
* HTTP 서버를 만들고 요청 처리
*/
var server = http.createServer((req,res)=>{
  var method = req.method;
  var uri = url.parse(req.url, true);
  var pathname = uri.pathname;

  if(method =="POST" || method == "PUT"){ // POST 와 PUT 이면 데이터를 읽음
    var body = "";
    req.on('data', function(data){
      body+=data;
    });
    req.on('end', function(){
      var params;
      if(req.headers['content-type'] == "application/json"){
        params = JSON.parse(body);
      } else{
        params = querystring.parse(body);
      }
      onRequest(res, method, pathname, params);
    });
  } else {
    onRequest(res, method, pathname, uri.query);
  }
}).listen(8000);

function onRequest(res, method, pathname, params){

  res.end("response!"); // 모든 요청에 response 라고 응답함
}
```

- 이를 발전시켜서 분기시켜 보자.
- 상품 관리, 회원 관리, 구매 관리의 각 모듈을 로드하고 URI 별로 해당 모듈을 호출한다.
- 정의되지 않은 URI 라면 HTTP 프로토콜에서 정의한 404 코드를 응답한다.
- 각 모듈에서 처리가 완료도면 response 함수를 호출하고, 모듈에서 콜백이 오면 모든 요청에 동일한 응답을 하던 response 함수를 JSON 형식으로 변환해 응답한다.

```jsx
function onRequest(res, method, pathname, params){

  switch (pathname) {
    case "/members":
      members.onRequest(res,method,pathname,params, response);
      break;
    case "/goods":
      goods.onRequest(res,method,pathname,params, response);
      break;
    case "/purchases":
      purchases.onRequest(res,method,pathname,params, response);
      break;
    default:
      res.writeHead(404);
      return res.end();
  }
}

/**
* HTTP 헤더에 JSON 형식으로 응답
*/
function response(res, packet){
  res.writeHead(200, { 'Content-Type' : 'application/json'});
  res.end(JSON.stringify(packet));
}
```

## Monopoly 비즈니스 모듈 만들기

- 앞에서 monolithic.js 가 URI 별로 각 모듈의 onRequest 함수를 호출하도록 했다. 해당 파일에서 다른 파일의 함수를 호츌하려면 exports 키워드를 사용해야 한다.

```jsx
exports.onRequest = function(res, method, pathname, params, cb){

  switch (method) {
    case "POST":
      return register(method, pathname, params, (response)=>{
        process.nextTick(cb, res,response);
      });
    case "GET":
      return inquiry(method, pathname, params, (response)=>{
        process.nextTick(cb,res,response);
      });
    case "DELETE":
      return unregister(method, pathname, params, (response)=>{
        process.nextTick(cb,res,response);
      });
    default:
      return process.nextTick(cb,res,null);
  }
}
```

- switch 문을 이용해 상품 등록은 register 함수로 분기되고 상품 조회와 삭제는 각각 inquery 와 unregister 함수로 분기된다. 각 함수는 메서드, URI, 파라미터를 전달받고 모두 처리되면 콜백 함수를 호출한다.
- 회원과 구매 관리 모듈도 비슷하게 작성한다.
- 화면에 출력하는 부분을 구현하지 않았기 때문에 API 가 정상적으로 동작하는지 테스트하려면 테스트 툴이 필요하다. 해당 부분은 HTTP 클라이언트로 만들어보자.

```jsx
const http = require('http');

var options = {
  host: "127.0.0.1",
  port: 8000,
  headers:{
    'Content-Type' : 'application/json'
  }
};

function request(cb, params){
  var req = http.request(options, (res)=>{
    var data = "";
    res.on('data', (chunk) =>{
      data+=chunk;
    });

    res.on('end', ()=>{
      console.log(options, data);
      cb();
    });
  });

  if(params){
    req.write(JSON.stringify(params));
  }

  req.end();
}

/*
* 상품 관리 API 테스트
*/
function goods(callback){
  goods_post(()=>{
    goods_get(()=>{
      goods_delete(callback);
    });
  });

  function goods_post(cb){
    options.method = "POST";
    options.path = "/goods";
    request(cb, {
      name : "test Goods",
      category: "tests",
      price : 1000,
      description : "test"
    });
  }
  function goods_get(cb){
    options.method = "GET";
    options.path = "/goods";
    request(cb);
  }

  function goods_delete(cb){
    options.method = "DELETE";
    options.path= "/goods?id=1";
    request(cb);
  }
}

/*
* 회원 관리 API 테스트
*/

function members(callback){
  members_delete(()=>{
    members_post(()=>{
      console.log("start_memebers_post");
      members_get(callback);
      console.log("start_memebers_end_get");
    });
  });

  function members_post(cb){
    options.method = "POST";
    options.path = "/members";
    request(cb, {
      username:"test_account",
      password:"1234",
      passwordConfirm : "1234"
    });
  }
  function members_get(cb){
    options.method = "GET";
    options.path="/members?username=test_account&password=1234";
    request(cb);
  }
  function members_delete(cb){
    options.method = "DELETE";
    options.path="/members?username=test_account";
    request(cb);
  }
}

/*
* 구매 관리 API 테스트
*/

function purchases(callback){
  purchases_post(()=>{
    purchases_get(()=>{
      callback();
    });
  });

  function purchases_post(cb){
    options.method = "POST";
    options.path = "/purchases";
    request(cb, {
      userid : 1,
      goodsid:1
    });
  }
  function purchases_get(cb){
    options.method = "GET";
    options.path = "/purchases?userid=1";
    request(cb);
  }
}

console.log("================ members =================");
members( () => {
  console.log("================ goods =================");
  goods( () => {
    console.log("================ purchases =================");
    purchases( () => {
      console.log("done.");
    });
  });
});
```

- 위와 같이 Monopoly 구조에 대해서 알아보았다.
