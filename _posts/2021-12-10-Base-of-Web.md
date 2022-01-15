---
title: Web Server 기본
date: 2022-01-14 +0000
categories: Web
tags: [web, http, node.js, express]
---

### 목표

Node.js를 이용하여 백엔드 서버를 구축한다.

1.  Node.js 기본개념, Node.js를 이용한 비동기작업
2.  HTTP/네트워크 기초 및 응용
3.  Node.js와 Express를 이용해 1의 내용을 구현한다.
4.  Server-side 디버깅 방법을 알아본다.
<hr>

## Node.js 기본

### Node.js란 무엇인가?

[공식문서](https://nodejs.org/en/about/)에 따르면 **Node.js는 비동기로 작동하는 Java Script Runtime**이다.
스레드기반 네트워킹은 비효율적이고 다루기 어렵다.
하지만 Node.js을 사용하면 데드락(dead-locking)을 걱정하지 않아도 된다.
왜냐하면 Node.js는 Single Thread로 작동하기 때문이다. 또한 Node.js는 I/O에 직접적으로 영향을 미치는 함수가 없다.
물론 Node.js Standard Library에는 Synchronous method가 있긴 하지만 **Node.js의 가장 중요한 철학은 비동기, Asynchronous**이다.
Node.js는 Ruby(루비)의 [Event Machine](https://en.wikipedia.org/wiki/EventMachine)과 Python(파이썬)의 [Twisted](<https://en.wikipedia.org/wiki/Twisted_(software)>)로부터 영감을 받았다고 한다.
이벤트 모델을 좀 더 확장시켜 [Event Loop](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)을 만들었다. Event Loop으로 인해 Node.js의 비동기를 가능하게 되었다.
Node.js는 [REPL](https://ko.wikipedia.org/wiki/REPL)환경의 스크립트를 실행과 Event Loop을 동시에 실행한다.
스크립트에는 비동기 API call, schedule timers 등과 같은 비동기 작업이 포함되어 있을 수 있는데 그러한 비동기 작업을 만나면 Event Loop 프로세스가 작동하게 된다.
Event Loop은 timer, pending callbacks, idle prepare, poll, check, close callbacks 이렇게 6단계(phase)를 순회한다.
각 단계는 FIFO로 작동하는 내부 큐를 가지고 있어서, Event Loop가 해당 phase에 들어오면 내부 큐에 있는 모든 callback함수를 실행한 후 다음 페이즈로 넘어간다. 자세한 내용은 생략.

### Node.js와 Web Browser

Node.js 이전에는 JavaScript를 실행할 수 있는 환경(Runtime)으로 Web Browser가 유일했다.
이것은 JavaScript가 웹 브라우저를 통한 실시간 피드백이 가능하기 때문에 웹 프로그래밍에 효율적이라는 장점으로 작용한 동시에 웹 브라우저에 지나치게 의존한다는 치명적인 약점으로 작용했다.
하지만 Node.js가 만들어짐으로써 로컬환경(우리의 PC 등)에서도 자바스크립트를 실행할 수 있게 되었다.
Node.js의 출현은 자바스크립트는 가장 많이 사용되는 프로그래밍 언어로 자리매김할 수 있게 되었다.
Node.js는 브라우저가 할 수 없는 몇 가지 작업도 가능하게 하는데 [Node.js 내장모듈](https://nodejs.org/dist/latest-v14.x/docs/api/)을 통해 모든 작업을 수행한다.
**Node.js로 개발을 한다는 것은 적절한 Node.js 내장 모듈을 어떻게 활용하는가의 문제이다.**
예를 들어, DNS에 대한 지식을 알고 있다면, [DNS 모듈 사용법 문서](https://nodejs.org/dist/latest-v12.x/docs/api/dns.html)에서 관련 메소드를 사용할 수 있다.

### Node.js [fs모듈](https://nodejs.org/dist/latest-v12.x/docs/api/fs.html)

파일의 [CRUD](https://ko.wikipedia.org/wiki/CRUD)를 구현할 수 있는 모듈이다.
html파일에서 외부 스크립트 파일을 불러올 때 다음과 같이 작성했다.
`<script src="불러오려고 하는 스크립트파일.js"></script>`
Node.js에서 파일을 불러오기 위해서는 일단 `fs모듈`을 불러와야 한다.
이렇게 불러온 모듈을 이용해 파일의 읽기, 쓰기 등의 작업을 하기 위해서는 해당 API문서를 읽어봐야 한다. 예를 들어 파일을 읽기 위해서는 [readFile](https://nodejs.org/dist/latest-v12.x/docs/api/fs.html#fs_fs_readfile_path_options_callback)에서 사용방법을 찾아보면 될 것이고, 파일을 쓰기 위해서는 [writeFile](https://nodejs.org/dist/latest-v12.x/docs/api/fs.html#fs_fs_writefile_file_data_options_callback)을 읽어보면 된다.

Third-party 모듈(해당 프로그래밍 언어에서 공식적으로 제공하는 built-in 모듈이 아닌 외부에서 만든 모듈)을 Node.js에서 사용할 수 있는데 방법은 간단하다.
`npm install 외부모듈이름`
터미널 창에 위 명령을 실행시키면 외부모듈이 설치된다.

#### `fs`모듈의 `readFile` 사용방법

    `fs.readFile(path[, options], callback)`

    API공식문서에는 `readFile`메소드의 문법이 위와 같이 나와있다.
    첫 번째 인자로 **파일의 위치**,
    두 번째 인자는 옵셥인데 여러가지 옵션을 설정할 수 있다.
    세 번째 인자는 콜백함수이다.

- `path \<string>|\<Bufffer>|\<URL>|\<integer>`
  `path`는 문자열, 버퍼, URL, 정수 이렇게 네 가지 타입의 값을 넣을 수 있지만 일반적으로는 문자열(`<string>`)타입을 넣는다.
- `options \<Object>|\<string>`
  `options`은 객체 또는 문자열로 넘길 수 있다.
  문자열로 넘길 경우 인코딩 방법을 넘긴다.

```js
let options = {
	encoding: 'utf-8', // UTF-8 인코딩 방식으로 파일을 연다.
	flag: 'r' // 읽기 전용으로 파일을 연다
}

// 정의한 options에 맞춰 파일을 연다.
fs.readFile('/etc/passwd', options, ...) // ...에는 콜백함수
```

- `callback \<Function>`
- `err \<Error>`
- `data \<string>|\<Buffer>`

  콜백함수는 파일을 다 읽고 난 후 비동기적으로 실행되는 함수이다.
  콜백함수에는 `err`, `data` 두 파라미터가 있다.
  파일읽기에 성공하면(에러가 발생하지 않으면) `err`는 `null`이 되고 `data`에는 `Buffer`라는 객체가 전달된다.
  `options`에 인코딩 방식이 정해져 있지 않다면 `data` 에 `Buffer`가 전달되고, `utf-8`과 같은 인코딩 방식이 정해져있다면 `string`이 전달된다.

**예제\_01**

```js
// 01_callBack.js
const fs = require("fs");

const getDataFromFile = function (filePath, callback) {
  return fs.readFile(filePath, "utf-8", (err, data) => {
    if (err) {
      callback(err, null);
    }
    if (data) {
      callback(null, data);
    }
  });
};

// 정의된 함수 사용하기
getDataFromFile("README.md", "utf-8", (err, data) => console.log(data));

module.exports = {
  getDataFromFile,
};
```

**예제\_02**

```js
// 02_promiseConstructor.js
const fs = require("fs");

const getDataFromFilePromise = function (filePath) {
  return new Promise((resolve, reject) => {
    fs.readFile(filePath, "utf-8", (err, data) => {
      if (err) {
        reject(err);
      }
      if (data) {
        resolve(data);
      }
    });
  });
};

// 사용하기
getDataFromFilePromise("README.md").then((data) => console.log(data));

module.exports = {
  getDataFromFilePromise,
};
```

**예제\_03**
예제\_02에서 작성한 함수를 이용하여 두 개의 파일을 비동기적으로 읽어서 내용을 하나의 배열에 넣는 예제이다.
파일의 수가 늘어나면 `Promise Hell`이 열릴 것을 예상할 수 있다.

```js
// 03_basicChaning.js
const path = require("path");
const { getDataFromFilePromise } = require("02_promiseConstructor.js");

const user1Path = path.join(__dirname, "files/user1.json");
const user2Path = path.join(__dirname, "files/user2.json");

const readAllUsersChaning = function () {
  return getDataFromFilePromise(user1Path)
    .then((file1) => {
      return getDataFromFilePromise(user2Path).then((file2) => {
        return `[${file1}, ${file2}]`; // [파일1, 파일2] 배열의 형태로 받기 위해 앞뒤에 []을 붙였다. 객체로 받고 싶다면 {}을 이용하면 된다.
      });
    })
    .then((text) => JSON.parse(text));
};

// 사용하기
readAllUsersChaning();

modle.exports = {
  readAllUsersChaning,
};
```

[**설명1**]
[`path.join()`](https://nodejs.org/dist/latest-v17.x/docs/api/path.html#pathjoinpaths)의 내용을 읽어보면 다음과 같다.
`path.join([...paths])`

- `...paths`는 패스 세그먼츠 시퀀스
- `Returns:` 문자열

```js
path.join("/foo", "bar", "baz/asdf", "quux", "..");
// Returns: '/foo/bar/baz/asdf'

path.join("foo", {}, "bar");
// Throws 'TypeError: Path must be a string. Received {}'
```

예시에서 보다시피 `path.join()`은 패스 세그먼트들을 연결한 `path`를 리턴한다.
경로구분자(seperator)는 사용 플랫폼에 따라 자동으로 정해진다.
Window라면 `\`, MacOs라면 `/`으로 연결된다.
`.`은 현재 디렉토리, `..`은 이전 디렉토리를 나타낸다.
`string`이 아닌 입력에 대해서는 `TypeError`를 리턴한다.

[**설명2**]
`__dirname`: 현재 파일이 위치한 디렉토리의 절대경로
`__filename`: 현재 파일을 포함한 파일의 절대위치
예를 들어 `example.js`라는 파일의 위치가 `/Users/me/temp/exampmle.js`라면

```js
// file 명을 포함한 절대경로
console.log(__filename); // /Users/me/temp/example.js

// file 명을 제외한 절대 경로
console.log(__dirname); // /Users/me/temp
```

`path`에 대해 [잘 설명한 글](https://p-iknow.netlify.app/node-js/path-moudle/)을 읽으면 좋다.

**예제\_04**
예제\_03에서 나타났던 `Promise Hell`을 `Promise.all()`을 이용하여 해결한다.

```js
// 04_promiseAll.js
const path = require('path');
const { getDataFromFilePromise } = require('./02_promiseConstructor');

const user1Path = path.join(__dirname, 'files/user1.json');
const user2Path = path.join(__dirname, 'files/user2.json');

const readAllUsers = () => {
	return Promise.all([
		getDataFromFilePromise(user1Path);
		getDataFromFilePromise(user2Path);
	]).then(result => JSON.parse(`[${result}]`))
}

readAllUsers()

module.exports = {
  readAllUsers
}
```

[**설명**]
[`Promise.all() MDN문서`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)을 참고하면
`Promise.all(iterable);`는 `iterable`이 생략되지 않는 경우 반환되는 `Promise`의 이행 결과값은 `iterable`에 포함된 모든 값을 담을 배열이다.
설명이 복잡한데, 예를 들어 **세 개의 프로미스 `p1, p2, p3`를 배열로 담아 `Promise.all()`메소드의 파라미터로 넣으면 반환되는 프로미스의 이행 결과값은 각각의 프로미스의 이행 결과값을 담은 배열이 된다.**
이 점을 참고하면 예제\_04에서 `${result}`은 두 개의 프로미스의 목록이 된다. 목록을 배열에 담기 위해 앞뒤로 `[]`을 붙였다.

**예제\_05**
`async`와 `await`을 이용하면 예제\_04보다 좀 더 직관적으로 비동기 작업을 구현할 수 있다.

```js
// 05_asyncAwait.js
const { getDataFromFilePromise } = require("./02_promiseConstructor");

const user1Path = path.join(__dirname, "files/user1.json");
const user2Path = path.join(__dirname, "files/user2.json");

const readAllUsersAsyncAwait = async () => {
  let file1 = await getDataFromFilePromise(user1Path);
  let file2 = await getDataFromFilePromise(user2Path);
  let file = file1 + "," + file2;
  return JSON.parse(`[${file}]`);
};

readAllUsersAsyncAwait();

module.exports = {
  readAllUsersAsyncAwait,
};
```

코드가 아주 직관적으로 읽힌다.

**파일을 비동기적으로 읽기/primise-Promise.all() 이용**

```js
cosnt fs = require('fs');
const path = require('path');

const user1Path = path.join(__dirname, 'files/user1.json');
const user2Path = path.join(__dirname, 'files/user2.json');


const readAllUsersChaning = function() {
	return getDataFromFilePromise(user1Path)
		.then(file1 => {
			return getDataFromFilePromise(user2Path)
				.then(file2 => {
					return `[${file1}, ${file2}]`; .
			})
	})
	.then(text => JSON.parse(text));
};

const getDataFromFilePromise = filePath => {
  return new Promise((resolve, reject) => {
    fs.readFile(filePath, 'utf-8', (err, data) => {
      if(err) {
        reject(err);
      } else {
        resolve(data);
      }
    })
  })
};

const readAllUsers = () => {
	return Promise.all([
		getDataFromFilePromise(user1Path);
		getDataFromFilePromise(user2Path);
	]).then(result => JSON.parse(`[${result}]`))
}

readAllUsers()

module.exports = {
  readAllUsers
}
```

**파일을 비동기적으로 읽기/async-await 이용**
예제\_02와 예제\_05을 합쳐서 `async-await`로 파일을 읽는 함수를 만들어보자.

```js
const fs = require("fs");
const path = require("path");

const user1Path = path.join(__dirname, "files/user1.json");
const user2Path = path.join(__dirname, "files/user2.json");

const getDataFromFilePromise = (filePath) => {
  return new Promise((resolve, reject) => {
    fs.readFile(filePath, "utf-8", (err, data) => {
      if (err) {
        reject(err);
      } else {
        resolve(data);
      }
    });
  });
};

const readAllUsersAsyncAwait = async () => {
  let file1 = await getDataFromFilePromise(user1Path);
  let file2 = await getDataFromFilePromise(user2Path);
  let file = file1 + "," + file2;
  return JSON.parse(`[${file}]`);
};

readAllUsersAsyncAwait();

module.exports = {
  readAllUsersAsyncAwait,
};
```

### `Web API's fetch`

`fetch`요청은 대표적인 비동기 요청인 네트워크 요청이다.
**Node.js에서는 `fetch`모듈이 제공되지 않는다.**
네트워크 요청에는 다양한 형태가 있지만 가장 흔한 방식은 `URL`로 요청하는 방식이다.
Node.js에서 `URL`을 통해 비동기 네트워크 요청을 가능하게 하는 API가 `fetch API`이다.
예를 들어 `fetch`를 사용하여 Naver 메인화면에서 날씨 정보를 나타내는 엘리먼트에 필요한 정보를 외부 `URL`로부터 가져올 수 있다.
아마도, 기상청에서 제공하는 정보를 해당 `URL`로부터 가져와 해당 `DOM`의 엘리먼트를 업데이트 하는 방식을 사용할 것이다.
이러한 작업은 비동기적으로 이루어지는데 경우에 따라 실행시간이 오래 걸릴 수도 있으면 로딩되는 동안에 '로딩창'을 띄어주는 것을 항상 고려해야 한다.
`fetch`는 비동기 작업이기 때문에 `Promise`형식으로 사용할 수 있다.

**`fetch`사용법**

```js
fetch("http://example.com/movies.json")
  .then((response) => response.json())
  .then((data) => console.log(data));
```

`fetch()`는 data를 가져오려고 하는 `URL`, 하나의 파라미터를 가지며, `init`라는 optional한 파라미터도 있다. 자세한 내용은 [fetch()](https://developer.mozilla.org/en-US/docs/Web/API/fetch)를 참고.
리턴값은 [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)객체이며, 이는 `HTTP응답`전체를 담고 있는 객체이다.
`Reoponse`객체로부터 `Json`형식의 데이터를 얻기 위해서는 `Response`객체의 메소드는 [`.json()`](https://developer.mozilla.org/en-US/docs/Web/API/Response/json)을 사용하면 된다.
`text`형식으로 데이터를 가져오기 위해서는 [.text()](https://developer.mozilla.org/en-US/docs/Web/API/Response/text)메소드를 사용하면 된다.
[Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)에 아주 상세하게 나와있다. 한 번쯤은 읽어보자.

**예제\_01**
두 개의 `URL`로부터 데이터를 `fetch`한 다음 하나의 객체로 합치는 예제.
지금은 두 개의 데이터이지만 데이터의 수가 많아지면 프로미스지옥이 생길 것을 예상할 수 있다.

```js
const newsURL = 'http://localhost:5000/data/lastestNews';
const weatherURL = 'http://localhost:5000/data/weather';

function getNewsAndWeather() {
	return fetch(newsURL)
		.then(news => {
			return fetch(weatherURL)
				.then(weather => {
					reutrn {
						news: news.data
						weather
					}
				})
		})
}

if (typeof window === 'undefined') {
  module.exports = {
    getNewsAndWeatherAll
  }
}
```

```json
// http://localhost:5000/data/lastestNews
{
  "data": [
    {
      "row_id": 2,
      "title": "2021년 경제 성장률 전망 밝아",
      "source": "A신문",
      "timestamp": "2020/12/30"
    },
    {
      "row_id": 3,
      "title": "코로나19 증가추세 대폭 하락해",
      "source": "BBC",
      "timestamp": "2020/12/29"
    },
    {
      "row_id": 4,
      "title": "코드스테이츠 취업연계 파트너사 xxx건 돌파",
      "source": "스타트업 뉴스",
      "timestamp": "2020/12/31"
    }
  ]
}
```

```json
// http://localhost:5000/data/weather
{
  "status": "sunny",
  "temperature": "28",
  "fineDust": "good"
}
```

예제\_02
예제\_01의 내용을 `Promise.all()`을 이용하여 구현하기. 코드 리딩이 쉽다.

```js
const newsURL = "http://localhost:5000/data/lastestNews";
const weatherURL = "http://localhost:5000/data/weather";

function getNewsAndWeather() {
  return Promise.all([
    fetch(newsURL).then((res) => res.json()),
    fetch(weatherUR).then((res) => res.json()),
  ]).then((result) => {
    return {
      news: result[0].data,
      weather: result[1],
    };
  });
}

if (typeof window === "undefined") {
  module.exports = {
    getNewsAndWeatherAll,
  };
}
```

**예제\_03**
예제\_02의 내용을 `async`, `await`을 이용하여 작성한다.

```js
var newsURL = "http://localhost:5000/data/latestNews";
var weatherURL = "http://localhost:5000/data/weather";

async function getNewsAndWeatherAsync() {
  let news = await fetch(newsURL).then((resp) => resp.json());
  let weather = await fetch(weatherURL).then((resp) => resp.json());
  return {
    news: news.data,
    weather,
  };
}

if (typeof window === "undefined") {
  module.exports = {
    getNewsAndWeatherAsync,
  };
}
```

<hr>

## HTTP/네트워크 기초 및 응용

### HTTP/네트워크 기초

#### 클라이언트 - 서버 아키텍처

모바일 쿠팡을 이용하여 쇼핑을 하는 경우를 생각해보자.
검색어를 입력하여 검색하면 관련된 정보들이 잘 정리되어 보여진다.
이 데이터가 우리 모바일 기기에 있는가? 아니다.

1. 우리가 검색어를 입력하여 검색버튼을 누르는 순간, 우리의 요청이 쿠팡의 서버로 날아간다.
2. 서버에서는 받은 요청에 맞는 데이터를 응답으로 보낸다.
3. 서버에서 보낸 응답 데이터가 우리에게 보여진다.

요즘은 이러한 인터넷 기반 애플리케이션이 흔하지만 예전에는 달랐다.
데이터는 기기에 저장하고 이용했다.
대표적인 사례는 차량 네비게이션이다.
최신 교통정보를 사용하기 위해 정기적으로 서비스센터에 가서 네비게이션을 업데이트 해야했다.
업데이트할 때 변경되거나 추가된 교통정보를 새롭게 다운로드받는 것이다.
네비게이션은 그나마 업데이트를 통해 사용할 수 있지만 다른 서비스에는 제약이 많다.
예를 들어 쇼핑몰 어플을 만든다면 실시간 결제 시스템이 필요한데 이것은 인터넷 없이 불가능하다.

쿠팡앱과 같이 상품의 정보 같은 리소스가 존재하는 곳과 리소스를 사용하는 곳을 분리시킨 구조 혹은 그런 기술을 **2티어 아키텍처 혹은 클라이언트-서버 아키텍처**라고 한다.
리소스를 사용하는 쪽이 **클라이언트**, 리소스를 제공하는 곳이 **서버**다.
서버 쪽을 **서버API**(혹은 API서버)와 **데이터베이스**(DB)로 나누면 **3티어 아키텍처**가 된다.

클라이언트에는 주로 웹사이트(웹앱), 스마트폰/태블릿 앱, 데스크탑 앱 등이 있다.
서버에는 웹서버, 파일서버, 메일서버, DB서버 등이 있다.

#### 클라이언트 - 서버 통신

클라이언트와 서버 간의 통신은 요청과 응답으로 이루어진다.
요청이 있어야만 응답이 있다.
클라이언트 - 서버 간 통신에 필요한 규약을 표준화한 것이 프로토콜이다.
프로토콜은 일종의 약속으로서 이를 매뉴얼처럼 사용하여 정보를 주고 받는다.
보통 웹 애플리케이션 아키텍처에서 클라이언트와 서버는 [HTTP](https://ko.wikipedia.org/wiki/HTTP)라는 프로토콜을 이용해 메시지를 주고 받는다.
HTTP를 이용해 주고받는 메시지를 HTTP Message라고 한다.
컴퓨터 공학과 네트워크 관련해서 기본적이고 중요한 내용이 [OSI 7 Layers](https://ko.wikipedia.org/wiki/OSI_%EB%AA%A8%ED%98%95)이다.
한 번쯤 꼭 공부해봐야 한다.

**클라이언트가 어떠한 서버를 이용하기 위해서는 서버가 제공하는 API를 이용해야 한다.**
서버는 API를 통해 클라이언트가 서버를 이용할 수 있는 방법을 구현한 인터페이스(Interface)를 제공해야 한다. API(Application Programming Interface)

인터넷을 통해 서버에 데이터를 요구할 때에는 HTTP 프로토콜을 사용하며, URL이나 URI를 통해 접근할 수 있다.
요청을 보낼 때 메소드를 특정해줘야 하는데 가장 많이 쓰는 메소드는 GET, POST, PUT, PATCH, DELETE가 있다. 자세한 내용은 [HTTP요청메서드(MDN문서)](https://developer.mozilla.org/ko/docs/Web/HTTP/Methods)를 참고.

#### URL과 URI

[URL](https://ko.wikipedia.org/wiki/URL)은 Uniform Resource Locator의 약자로, 네트워크 상에서 웹 페이지, 이미지, 동영상 등의 파일이 위치한 정보를 나타낸다.
[URI](https://developer.mozilla.org/ko/docs/Glossary/URI)는 Uniform Resource Identifier의 약자로, URL의 기본적인 요소인 `scheme`, `hosts`, `url-path`에 더해 `query`, `bookmaker`를 포함한다.
즉 URI가 URL의 상위 카테고리이다.
![URL과 URI 설명 이미지](https://media.geeksforgeeks.org/wp-content/uploads/20210625160610/urldiag.PNG)
위 이미지에 잘 설명되어 있다.

- `scheme`은 통신 프로토콜을 결정한다. 일반적으로 웹 브라우저에서는 `http`나 `https`를 사용한다.
- `domain`은 subdomain, top-level domain 등으로 나눌 수도 있다.
  domain name을 IP주소로 대체할 수도 있다.
  원래는 어떤 서버이든지 고유의 IP주소가 있다.
  IPv4, IPv6 두 버전이 있는데 IPv4는 0~255의 범위를 가지는 세 자리수 네 개로 이루어진다(예: 168.126.01.3)이다. 대략 43억 개의 IP주소를 생성할 수 있다.
  IPv4로 부족하기 때문에 나온 것이 [IPv6](https://en.wikipedia.org/wiki/IPv6)인데 2^128개의 IP주소를 생성할 수 있다.
  IP주소는 읽기 힘들기 때문에 DNS(Domain Name System)를 통해 domain name으로 바꾸어 사용한다.
  `125.209.222.142 <- naver.com`
  웹 브라우저에 naver.com 이라고 치면 [DNS](https://ko.wikipedia.org/wiki/%EB%8F%84%EB%A9%94%EC%9D%B8_%EB%84%A4%EC%9E%84_%EC%8B%9C%EC%8A%A4%ED%85%9C)가 해당 도메인의 IP주소로 연결시켜준다.
- `path(혹은 url-path)`는 웹 서버의 루트 디렉토리부터 시작하여 웹 페이지, 동영상, 이미지 등이 위치한 경로와 파일명을 나타낸다.
- `query`를 이용해 웹 서버에 추가 질문을 할 수 있다.
- `fragment`는 웹 페이지 상에 위치한 앵커 포인트를 나타낸다.
  프래그먼트를 잘 설명한 글이 있어 [여기](https://velog.io/@roro/URL-%ED%94%84%EB%9E%98%EA%B7%B8%EB%A8%BC%ED%8A%B8-HASH) 남겨둔다.

#### HTTP Message

[MDN문서](https://developer.mozilla.org/ko/docs/Web/HTTP/Messages)에 아주 상세하게 나와 있다.
간단하게 요약해보자

![http message 구조](https://mdn.mozillademos.org/files/13827/HTTPMsgStructure2.png)

① Request Message

- Start Line
  - method: GET, PUT, POST, DELETE 등
  - url:
    - method에 따라 다르다
    - origin 형식: `?`와 쿼리 문자열이 붙는 절대 경로이다.
      예: `POST / HTTP 1.1`
      `GET /background.png HTTP/1.0`
      `HEAD /test.html?query=alibaba HTTP/1.1`
      `OPTIONS /anypage.html HTTP/1.0`
    - absolute 형식
      완전한 URL형식, [프록시](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9D%EC%8B%9C_%EC%84%9C%EB%B2%84)에 연결하는 경우 대부분 GET과 함께 사용된다.
      예: `GET http://developer.mozilla.org/en-US/docs/Web/HTTP/Messages HTTP/1.1`
    - autority 형식
      도메인 이름 및 옵션 포트(':'가 앞에 붙는다)로 이루어진 URL의 authority component 이다.
      HTTP 터널을 구축하는 경우에만 CONNECT와 함께 사용할 수 있습니다.
      예제: `CONNECT developer.mozilla.org:80 HTTP/1.1`
    - asterisk 형식
      OPTIONS와 함께 별표('_') 하나로 간단하게 서버 전체를 나타낸다.
      예: `OPTIONS _ HTTP/1.1`
  - protocol version
- Headers
  다양한 종류의 헤더가 있는데 세 종류로 나눌 수 있다.
  - General 헤더 - message 전체에 적용된다.
  - Request 헤더 - 요청의 내용을 좀 더 구체화시키거나 컨텍스(Referer)를 제공하거나 제약사항을 설정하여 요청의 내용을 수정하는 역할을 한다
  - Entity 헤더 - Request Body 에 적용되는 헤더이다.
    그러므로 Body가 없으면 Entity 헤더도 전송되지 않는다.
    예: `Content-Type: text-plain`
    `Content-Length: 350`
- Body
  GET, HEAD, DELETE, OPTIONS 처럼 리소스를 가져오는 요청은 본문이 필요하지 않다.
  - 단일 리소스 본문(single-resource bodies): `Content-Type`과 `Content-Length`로 정의된 단일 파일로 구성된 본문
  - 다중 리소스 본문(multiple-resource bodies): HTTP Form과 관련된다.

② Response Message

- Status line 상태줄
  다음과 같은 정보를 담고 있다
  - 프로토콜 버전: 보통 HTTP/1.1이다
  - 상태코드: 요청의 성공여부를 나타낸다. [200](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/), [404](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/404), [302](https://developer.mozilla.org/ko/docs/Web/HTTP/Status/302) 등
  - 상태 텍스트: 짧고 간결하게 상태코드에 대한 설명한 글
    예: `HTTP/1.1 404 Not Found.`
- Headers
  요청 메시지의 Header와 같다
- Body
  크게 세 가지 종류로 나뉜다
  - 이미 길이가 알려진 단일 파일로 구성된 single-resource bodies로서 두 개의 헤더(`Content-Type`, `Content-Length`)로 정의한다.
  - 길이를 모르는 단일 파일로 구성된 single-resource bodies로서 `chunked`로 연결되어 있으며, 파일이 청크로 나뉘어 인코딩되어 있다.
  - 서로 다른 정보를 담은 multiple-resource bodies로서 자주 보이지는 않는다.

#### 읽어봐야 할 글

1. [브라우저는 어떻게 작동하는가?](https://d2.naver.com/helloworld/59361)
2. [MIME 타입](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types)

---

### 브라우저의 작동원리

#### AJAX

AJAX는 Asynchronous JavaScript And XMLHttpRequest의 약자로, JavaScript, DOM, Fetch, XMLHttpReqest, HTML 등의 다양한 기술을 사용하는 웹 개발 기법이다.
AJAX의 가장 큰 특징은 웹 페이지에 필요한 부분에 **필요한 데이터만 비동기적으로 받아와** 화면에 그려낼 수 있다는 것이다.

- AJAX의 두 가지 핵심 기술

1.  JavaScript와 DOM
    전통적인 웹 어플리케이션에서는 `<form>` 태그를 이용해 서버에 데이터를 전송해야 했다. 또한 서버는 요청에 대한 응답으로 새로운 웹 페이지를 제공해주어야 했다. 다시 말해, 클라이언트에서 요청을 보내면 매번 새로운 페이지로 이동해야 했다.
    그러나 Fetch를 사용하면, 페이지를 이동하지 않아도 서버로부터 필요한 데이터를 받아올 수 있다. Fetch는 사용자가 현재 페이지에서 작업을 하는 동안 서버와 통신할 수 있도록 한다. 즉, 브라우저는 Fetch가 서버에 요청을 보내고 응답을 받을때까지 모든 동작을 멈추는 것이 아니라, 계속해서 페이지를 사용할 수 있게 하는 비동기적인 방식을 사용한다.
    또한 자바스크립트에서 DOM을 사용해 조작할 수 있기 때문에, Fetch를 통해 전체 페이지가 아닌 필요한 데이터만 가져와 DOM에 적용시켜 새로운 페이지로 이동하지 않고 기존 페이지에서 필요한 부분만 변경할 수 있습니다.
2.  XHR과 Fetch
    Fetch는 XHR의 단점을 보완한 새로운 Web이며, XML보다 가볍다.
    JavaScript와 호환되는 JSON을 사용한다.
    Fetch 예제
    ```js
    // Fetch를 사용
    fetch('http://52.78.213.9:3000/messages')
    	.then (function(response) {
    		return response.json();
    	})
    	.then(function (json) {
    		...
    });
    ```

- AJAX의 장점
  - 서버에서 HTML을 완성하여 보내주지 않아도 된다. 필요한 데이터를 비동기적으로 가져와 브라우저의 일부만 업데이트 하여 렌더링하는 방식으로 작동한다.
  - 브라우저에 상관없이 AJAX를 사용할 수 있게 되었다
  - 상호작용이 많은 어플리케이션 개발에 유용하다. 빠르다.
  - 필요한 데이터를 텍스트 형태(JSON, XML 등)로 보내기 때문에 대역폭(네트워크 통신에서 한 번에 보낼 수 있는 데이터의 크기)이 작아도 된다.
- AJAX의 단점
  - SEO Search Engine Optimization 에 불리하다
    AJAX 방식의 웹 어플리케이션은 한 번 받은 HTML을 렌더링 한 후, 서버에서 비동기적으로 필요한 데이터를 가져와 리렌더링하기 때문에 처음 받는 HTML파일에는 데이터를 채울 '그릇'만 작성되어 있는 경우가 많다.
    검색엔진은 전세계 사이트의 정보를 수집하는데 AJAX 방식의 웹 어플리케이션의 HTML파일에는 데이터가 없기 때문에 검색엔진에 데이터를 제공하지 못한다.
  - 뒤로가기 버튼 문제
    AJAX에서는 이전 상태를 기억하지 않는다.
    뒤로가기 기능을 구현하기 위해서는 별도의 History API를 사용해야 한다.

#### SSR과 CSR

![SSR](https://s3.ap-northeast-2.amazonaws.com/urclass-images/Dot3Zo7Dg-1619657779119.png)

- SSR은 Server Side Rendering의 줄임말이다. 웹 페이지를 브라우저에서 렌더링하는 대신에, 서버에서 렌더링한다. 브라우저가 서버의 URI로 GET 요청을 보내면, 서버는 정해진 웹 페이지 파일을 브라우저로 전송한다. 그리고 서버의 웹 페이지가 브라우저에 도착하면 완전히 렌더링된다. 서버에서 웹 페이지를 브라우저로 보내기 전에, 서버에서 완전히 렌더링했기 때문에 Server Side Rendering 이라고 한다. 웹 페이지의 내용에 데이터베이스의 데이터가 필요한 경우, 서버는 데이터베이스의 데이터를 불러온 다음 웹 페이지를 완전히 렌더링 된 페이지로 변환한 후에 브라우저에 응답으로 보낸다. 웹 페이지를 살펴보던 사용자가, 브라우저의 다른 경로로 이동하면 어떻게 될까? 브라우저가 다른 경로로 이동할 때마다 서버는 이 작업을 다시 수행한다.

  ![CSR](https://s3.ap-northeast-2.amazonaws.com/urclass-images/OfxuhVcgB-1619657798309.png)

- CSR은 Client Side Rendering 을 의미한다. 일반적으로 CSR은 SSR의 반대로 여겨진다. SSR이 서버 측에서 페이지를 렌더링한다면, CSR은 클라이언트에서 페이지를 렌더링한다. 웹 개발에서 사용하는 대표적인 클라이언트는 웹 브라우저이다. 브라우저의 요청을 서버로 보내면 서버는 웹 페이지를 렌더링하는 대신, 웹 페이지의 골격이 될 단일 페이지를 클라이언트에 보낸다. 이때 서버는 웹 페이지와 함께 JavaScript 파일을 보낸다. 클라이언트가 웹 페이지를 받으면, 웹 페이지와 함께 전달된 JavaScript 파일은 브라우저에서 웹 페이지를 완전히 렌더링 된 페이지로 바꾼다. 웹 페이지에 필요한 내용이 데이터베이스에 저장된 데이터인 경우에는 어떻게 해야 할까? 브라우저는 데이터베이스에 저장된 데이터를 가져와서 웹 페이지에 렌더링을 해야 한다. 이를 위해 API가 사용된다. 웹 페이지를 렌더링하는 데에 필요한 데이터를 API 요청으로 해소한다. 마지막으로, 브라우저가 다른 경로로 이동하면 어떻게 될까? CSR에서는 SSR과 다르게, 서버가 웹 페이지를 다시 보내지 않는다. 브라우저는 브라우저가 요청한 경로에 따라 페이지를 다시 렌더링한다. 이때 보이는 웹 페이지의 파일은 맨 처음 서버로부터 전달받은 웹 페이지 파일과 동일한 파일이다.
- SSR과 CSR을 사용해야 하는 경우
  - Use SSR
    - SEO(Search Engine Optimization) 가 우선순위인 경우, 일반적으로 SSR(Server Side Rendering) 을 사용한다.
    - 웹 페이지의 첫 화면 렌더링이 빠르게 필요한 경우에도, 단일 파일의 용량이 작은 SSR 이 적합하다.
    - 웹 페이지가 사용자와 상호작용이 적은 경우, SSR 을 활용할 수 있다.
  - Use CSR
    - SEO 가 우선순위가 아닌 경우, CSR을 이용할 수 있다.
    - 사이트에 풍부한 상호 작용이 있는 경우, CSR 은 빠른 라우팅으로 강력한 사용자 경험을 제공한다.
    - 웹 애플리케이션을 제작하는 경우, CSR을 이용해 더 나은 사용자 경험(빠른 동적 렌더링 등)을 제공할 수 있다.
