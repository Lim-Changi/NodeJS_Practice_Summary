# 전체 정리

# Test 및 전체 Project 품질관리

## 1. 최우선적으로 API 테스트를 진행하자

- -> 가장 기본적인 Test 임과 동시에 프로젝트 전체를 가장 넓게 커버할 수 있는 테스트
- `Postman`, `Swagger`, `Mocha`, 등등 여러가지 도구가 존재

이후에 시간이 난다면, 추가적인 단위,DB,퍼포먼스 테스트등을 진행하도록하자

<br>

---

## 2. Test Code 를 작성할때는

- `무엇을,`
- `어떤 환경에서,`
- `어떤 결과를 예상하고,`
- 이 세가지를 꼭 포함하도록하자

##### 잘 지켜진 예) <br>

> 똑같은 기기에서 한 달내에 가입한 계정이 있으면, // 어떤 환경 <br>
> 회원가입이 // 무엇 <br>
> Error Code 401 을 발생하며 실패해야 한다. // 어떤 결과를 예상

##### 못 지켜진 예) <br>

> 회원가입 작동한다. // 실패시, 어떤 Case에서 왜 Error 가 나는지 확인하기 힘듦

Node JS <u>Mocha Test</u> 의 경우,

`describe` 에 기능을 (무엇) 명세,
`it` 에 환경과 예상결과를 적어 Test 진행

<br>

---

## 3. Test Code 는

- `Arrange(준비)`
- `Act(실행)`
- `Assert(확인)`
- <u>AAA</u> 서순에 맞게 작성하고,
- 각 부분을 보기 편하도록 나누자 (줄바꿈 및 주석을 활용)

[위의 예시](#잘-지켜진-예)를 활용 하자면,

```javascript
// Arrange(준비, 환경세팅) =>
User.push({device: 1, name: 찬기, createdAt: 22.01.01});

// Act(실행) =>
User.push({device: 1, name: 임찬, createdAt: 22.01.06});

// Assert(확인)
expect(statusCode).to.be(401);
```

확인하기 편하게 Step 별로 구분 짓도록 하자.

<br>

---

## 4. Code Linter 를 활용,

기본적인 Code Quality를 테스트 전에 확인하도록하자

`ESLint` 와 `Prettier` 를 활용하면 개발 단계에서도 쉽게 관리할 수 있다.

<br>

---

## 5. Test 시, DB 혹은 저장공간에 존재하는 <u>**공용 Test Data (Seed Data)의 활용을 피하고**</u> 각 Test 마다 <u>**새로운 Data 를 추가**</u>하도록 하자

개별 Test가 공통된 Data 및 기존 Data를 활용하면 Test 간에 서로 간섭이 이루어져,<br>
기존에 설계한 환경 및 결과가 의도와 다르게 실행될 수 있다.

##### 예)

```javascript
let User = { name: 찬기, todaySteps: 0, todayCash: 0 };

// Test1. 유저는 10 Step 당 1원을 벌 수 있다
User.todaySteps += 1000;

expect(User.todayCash).to.be(100);
// 통과

// Test2. 유저가 영상을 시청하면 100원을 벌 수 있다
User.watchAd();

expect(User.todayCash).to.be(100);
// 통과 X => todayCash = 200
// Test2 만 놓고 보면, watchAd가 200원을 줬다고 착각할 수 있다.
```

위와 같은 일이 발생 할 수 있으므로 `각 Test는 개별 Data를 추가`해서 써야한다.

<br>

---

## 6. 지속적으로 보안에 취약한 Dependency(Module)를 점검하자

`npm audit` 및 `snyk.io` 등 도구를 사용하면 취약점을 쉽게 파악할 수 있다.

<u>node_modules</u> 에 대한 추가적인 학습이 필요! (동작 및 보안 원리)

<br>

---

## 7. Test들을 기능과 목적에 맞게 묶음단위(키워드)로 관리하자

> 개발자가 기능을 하나 개발하면서 Test를 진행할때,<br>
> 관련이 없는 프로젝트의 모든 Test 가 실행되면 개발속도가 매우 저하될 수 있다.<br>
> -> Test 를 꺼려하는 원인이 될 수 있음!

> 기본적인 빌드 및 입출력이 없는 간단한 Test는 지속적으로 확인하되,<br>
> 인수 Test(시나리오 Test) 즉, 속도가 오래걸리는 Test는 기능 개발이 완료되면 진행하는 것이 좋다<br>

> 이를 효율적으로 관리하기 위해,<br>
> Test들을 키워드로 묶어 관리하고, (하나의 describe로 묶어서 관리) <br>
> 개발중에 Test 를 진행할때는 개발중인 기능과 관련된 Test만 따로 실행하도록 한다.<br>

Mocha 로 예시를 들자면 여러가지 방법이 있는데,

```javascript
// ex) 회원가입 추가기능 개발 Test
// 회원가입은 Account 키워드로 묶일 수 있다. => signUp, login, logout , etc ...

// 개발중인 기능의 Test들이 담긴 폴더만 실행하는 방법
mocha test\\Account\\*.test.js

// Test 가 끝난 기존 Test File 들을 describe.skip 을 활용하여 test skip 처리하는 방법
describe.skip('Purchase') // 회원가입과 관련없는 기능
// test가 필요한 경우, skip을 지우고 test script를 실행하면 된다.
mocha test\\**\\*.test.js
```

목적은 개발중 Test의 Overhead가 커지지 않게 함에 있다.


# 보안

## 6.1 Code Linter를 통해 보안취약점과 문제점을 캐치하자

> `eslint-plugin-security` 라는 ESLint 보안 플러그인을 통해 코드상 보안문제를 방지할 수 있다

> `eslint-plugin-security` 가 잡는 보안에 취약한 코드의 몇가지 예시)

```javascript
const insecure = crypto.pseudoRandomBytes(5);
```

```javascript
const path = req.body.userinput;
fs.readFile(path);
```

```javascript
const userinput = req.body.userinput;
eval(userinput);
```

```javascript
const unsafe = new RegExp("/(x+x+)+y/)");
```

<br>

---

## 6.2 동시다발적인(concurrent) 요청들을 제한하자.

> `클라우드 로드밸런서`, `nginx` 및 `rate-limiter-flexible`, `express-rate-limiter`등 미들웨어를 활용하여
> 시간당 요청수를 제한하도록 하자.

> 이를 통해 `DDoS 공격`을 방지할 수 있다. [DDoS 공격이란?](https://www.cloudflare.com/ko-kr/learning/ddos/what-is-a-ddos-attack/)

<br>

---

## 6.3 기밀은 설정파일에서 빼거나 패키지를 활용하여 암호화하자

> 아이디와 비밀번호등 중요한 기밀사항은 절대 평문 형태로 설정파일 혹은 소스코드에 저장하지 않도록한다.

- `Vault`, `Kubernetes/Docker Secrets` 등 기밀관리 시스템을 활용하자
- 부득이하게 보안에 취약한곳에 저장을 해야한다면, 반드시 **암호화**하여 저장하자
- 실수로 기밀사항을 pre-commit하거나 push하지 않도록 한다. <u> _git 및 AWS 에서 경고가 엄청 날라온다_</u> /// ~~개인경험..~~
  - `git secret` 을 활용하면 이를 방지할 수 있다.

<br>

---

## 6.4 ORM/ODM 라이브러리를 사용하여 DB Data Injection 공격을 방지하자

> DB 쿼리를 직접 작성하거나, **Data 검증(Validation)** 이 원활하게 이루어지지 않으면 이러한 공격에 취약해질 수 있다.

> `joi`, `yup`과 같은 Data Validation 라이브러리를 사용하거나, 다음과 같은 **ORM/ODM** 을 쓰는것 만으로도 이러한 위험을 방지할 수 있다.

- TypeORM
- Sequelize
- Mongoose
- Knex
- Objection.js
- waterline

> ORM/ODM 은 복잡한 쿼리를 사용하기 쉽게 만들어줄 뿐만 아니라, 보안적인 문제도 해결해준다는 여러 장점이 있기에,<br>
> DB 에 저장할 Data 를 완벽하게 Validate 할 자신이 없다면 꼭 사용하기로 하자.

<br>

---

## 6.5 NodeJS 및 전체적인 서버의 여러 보안 모범사례

- SSL/TLS 프로토콜을 활용하여 클라이언트/서버 연결을 암호화하자 [[참고]](https://www.cloudflare.com/ko-kr/learning/ssl/what-is-ssl/)

  - 우리가 흔히 아는 HTTPS 생각하면 된다<br><br>

- 보안이 중요한 데이터의 값이 일치하는지 확인할때는 안전한 메소드를 사용하자

  - 일반적인 비교연산자 대신, NodeJS 에서 제공하는 crypto.timingSafeEqual(a,b) 사용하자
  - 일반적인 비교연산자를 활용하면 중요한 정보가 노출될 수 있다.<br><br>

- 무작위 Random String을 생성할때는 NodeJS 의 모듈을 사용하자

  - 특정 Random String Generator Method는 생각과는 다르게, 무작위로 생성되지 않을 수 있다. => 위험에 노출이 됨
  - NodeJS 에서 제공하는 `crypto.randomBytes(size, [callback])` 을 사용하자<br><br>

- 상용사이트를 띄울때는 `security.txt` 파일을 첨부하여 보안 취약점을 신고받을 수 있는 공간을 마련해두자

- 오픈소스에 코드를 올릴떄는 `security.md` 파일을 첨부하여 보안 취약점을 신고받을 수 있는 공간을 마련해두자
  <br><br>

> 아래는 <u>**OWASP**</u> 에서 권장하는 보안규칙이다 [[OWASP 란?]](https://ko.wikipedia.org/wiki/OWASP)

- 인증

  - 로그인 횟수에 제한을 두자
  - 로그인 실패시, 아이디 비밀번호중 어떤 것이 틀렸는지 알려주지 말자
  - 강한 비밀번호 규칙을 설정하자
  - 비밀번호와 액세스키를 자주 변경하자<br><br>

- 권한

  - 꼭 필요한 최소한의 권한까지만 제공하도록 하자. // 필요이상의 권한을 부여하지 말자
  - root 계정(모든 Access)으로 작업 하지말자
  - 특정 개인에게 권한을 부여하지말고, 그룹단위로 권한을 관리하자<br><br>

- 보안 구성

  - 내부 서비스에 접근할때는 꼭 보안 네트워크를 통해 접근할 수 있도록 하자
  - 내부 네트워크 액세스에 접근제한 설정을 해두자
  - 쿠키를 사용한다면 "Secured" "Same Site" "HttpOnly" 설정을 해서 사용하자
  - HTTPS 와 TCP 로드밸런서를 사용하자
  - 주기적으로 보안안정성 검사를 하자<br><br>

- 데이터 노출

  - SSL/TLS 연결만 허용하자
  - 네트워크를 각자 최소 권한만 가지도록 분리하자
  - 비밀값은 AWS KMS, Google Cloud KMS 등 보안이 안전한 프로덕트에 보관하자
  - 로그에 비밀값을 남기지 말자
  - 비밀값을 저장할때는 꼭 해싱처리하고, 클라이언트 상에서 보이지 않도록 하자<br><br>

- 구성요소 보안성

  - 도커 이미지 보안 검사를 실행하자
  - 자동 인스턴스 업데이트를 활성화하여 최신 OS 버전을 유지하자
  - Refresh Token 을 활용하여, Access Token 의 주기를 짧게 설정하자
  - Cloud API 혹은 관리 시스템 접근에 대한 로그를 남기자 > AWS CloudTrail<br><br>

- 로그 및 모니터링

  - 수상한 조작은 알림 처리하자
  - 비정상적인 로그인시도를 알림 처리하자
  - DB 레코드를 Update 한 시간과 Update한 유저이름을 남기도록 하자<br><br>

- XSS 공격 [[참고]](https://noirstar.tistory.com/266)

  - XSS 공격에 안전한 템플릿, 프레임워크를 사용하자 -> EJS, Pug, React, Angular
  - 검증되지 않은 HTTP 요청은 무시하자
  - Content Security Policy 설정을 활성화 하자<br><br>

- 개인정보
  - 개인정보는 항상 암호화하자
  - 각 국가의 개인 정보 보안 규정을 따르자<br><br>

<br>

---

## 6.6 HTTP 응답 Header 설정을 통해 보안을 강화하자

> 보안 Header 를 통해 XSS, 클릭재킹과 같은 공격을 막을 수 있다.

> 다음과 같은 Header 설정을 통해 보안을 강화할 수 있다 -> `Helmet` 모듈 활용

- HTTP Strict Transport Security (HSTS)
- Public Key Pinning for HTTP (HPKP)
- X-Frame-Options
- X-XSS-Protection
- X-Content-Type-Options
- Referrer-Policy
- Expect-CT
- Content-Security-Policy
- Additional Resources

<br>

---

## 6.7 지속적으로 보안이 취약한 모듈이 있는지 확인하자

> `npm audit` 및 `snyk` 등 도구를 활용하여 취약한 모듈을 지속적으로 감시하고 패치하자.

> 모듈 보안을 확인하는 절차를 CI에 포함시켜 상용서버에 보안이 취약한 모듈이 포함되는것을 방지하자.


# 성능 Performance

## 7.1 이벤트 루프를 막지 말자

> CPU 에 부담이 가는 Blocking 작업들은 Single Thread 인 Event Loop 를 블로킹할 수 있으므로 조심해서 사용하도록 하자.

### NodeJS의 `Event Loop`는 대부분 싱글스레드 위에서 여러 큐에 의해 핸들링 된다.

- Event Loop<br>
  <img src="https://evan-moon.github.io/static/dfece5e35b7d9e50b6eac8adb1a348b8/949b7/nodejs-event-loop-phase.png" width=400 > [[출처]](https://evan-moon.github.io/2019/08/01/nodejs-event-loop-workflow/)<br>
- 실행 순서<br>
  <img src= "https://media.vlpt.us/images/dami/post/8e5c1b68-101c-4fff-8090-7024ff2730c0/%E1%84%86%E1%85%A1%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%B3%E1%84%85%E1%85%A9%E1%84%90%E1%85%A2%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3.gif" width=600> [[출처]](https://velog.io/@dami/JS-Microtask-Queue) <br>

- 예시

  ```javascript
  setTimeout(() => alert("timeout"));

  Promise.resolve().then(() => alert("promise"));

  alert("code");

  // code -> promise -> timeout
  ```

## Blocking 작업 예시

- Large JSON parsing
- Large Array 로직 적용
- Unsafe RegEx
- 작업량이 큰 I/O

> Event Loop가 Blocking 되는 것을 방지하기 위해서 이러한 작업들을 **`libuv`** 의 판단아래, OS Kernel 혹은 Worker Thread 로 넘겨 처리한다. -> <u>Offloading</u>

<br><hr>

## 7.2 외부 Util 라이브러리 대신, (내장된) Native 자바스크립트 Method 사용하도록하자

> Native Method 대신 외부 Util 라이브러리를 쓰는 것은 불필요한 의존성과 성능 저하를 야기할 수 있다.

### **`Lodash`** vs **`V8 Native Method`**

 <img src="https://github.com/goldbergyoni/nodebestpractices/raw/master/assets/images/sampleMeanDiag.png" width=1000>

- 같은 요청을 처리할때, Lodash Method 가 V8 Engine Method 에 비해 약 146% 시간이 더 오래걸린다.
- 특정 Method의 경우, Lodash 가 빠른 경우도 있지만, 전체적인 효율성을 생각해서 **`Native Method`** 를 사용하도록 하자.

<br><br><br><hr>

# Docker 모범 사례

## 8.1 안전한 도커 이미지를 위해 Multi Stage 빌드를 활용하자

### **`Docker Multi Stage`** ?

> 컨테이너 이미지를 만들면서 빌드 등에는 필요하지만 최종 컨테이너 이미지에는 필요 없는 환경을 제거할 수 있도록 단계를 나누어서 기반 이미지를 만드는 방법

- Multi Stage 빌드를 통해, 보안위협 방지와 상용서버에 불필요한 요소를 쉽게 관리할 수 있다. [[참고]](https://velog.io/@guri_coding/%EC%95%BC-Docker-%EC%9D%B4%EB%A6%AC%EB%82%98%EC%99%80-5-Multi-Stage-builds)

<br><hr>

## 8.2 npm start 대신, node 커맨드를 활용하여 BootStrap 하자

### **`CMD ["node", "dist/index.js"]`** <br><br>

> node command 를 통해 부팅해야 <u>child-process</u>, <u>signal-handling</u>, <u>graceful shutdown</u> 등 문제를 방지할 수 있다.

> npm 스크립트는 OS signal를 통과하지 않는다.
>
> > 정상적인 종료(Graceful Shutdown) 가 이루어지지 않을 수 있다.

### **`Graceful Shutdown`** ? <-> Hard Shutdown

> 프로그램이 종료될 때, 최대한 사이드 이펙트 없이 로직을 잘 처리하고 종료하는 것을 말한다.

- ex) 퇴근시,
  - 코드를 커밋하고 푸시한 뒤 컴퓨터를 끄고 퇴근하는 것: `graceful shutdown`
  - 그대로 컴퓨터를 바로 끄고 퇴근하는 것: hard shutdown
    > 현재 처리중인 요청이나 데이터가 유실되지 않고 전부 처리가 된 후에 정상적으로 종료되는 것으로 볼 수 있다.


# Docker 모범 사례

## 8.12 도커 이미지의 모든 범위에서의 취약점을 검사하자

> 코드상의 취약점을 검사하는 것도 중요하지만, 이외에도 OS Binary 상의 보안 취약점이 존재하기 때문에, 이미지를 상용화 하기 전에 전체 범위(E2E)의 보안 검사를 진행하는 것은 필수적이다.

> OS-level binaries 예시 >> `Shell`, `OpenSSL`, `TarBall`

- 검사 도구는 크게 3가지로 분류할 수 있는데,
  1. <u>취약점이 DB에 캐싱되어 있는 로컬/CI 바이너리</u>
  2. 클라우드 검사 서비스
  3. 도커 빌드 자체적으로 실행하는 검사도구

> 이 중 첫번째 케이스가 가장 빠르면서 흔히 쓰이는데, 대표적으로 `Trivvy`, `Anchore`, `Snyk` 등이 있다.

> 높은 검사 기준을 세워, 추후에 발생할 수 있는 문제를 방지하자.

<br><br>

## 8.13 node_modules cache 를 지우자

> 컨테이너에 Dependency 를 전부 받은 후에는, 로컬 캐시를 지우도록 하자.

> 캐시를 지우는 라인 하나만으로도 무의미한 30%의 용량을 덜어낼 수 있다.

```docker
FROM node:12-slim AS build

WORKDIR /usr/src/app
COPY package.json package-lock.json ./
RUN npm ci --production && npm cache clean --force # --force flag를 추가하여 non-zero code로 인한 캐싱관련 빌드 실패를 방지하자.
```

<br><br>

## 8.14 모범사례 예시

- ADD 대신 COPY 커맨드를 사용하자

- Base OS Binary 를 업데이트하는것은 피하자

  - _apt-get update_ 등, build 중에 로컬 바이너리를 업데이트하는 것은 일관성없는 이미지를 생성하게되고, 더 많은 권한을 요구하게 된다.
  - 대신, 자주 업데이트되는 base image를 사용하도록 하자

<br>

- 이미지를 분류하자

  - 이미지에 대한 설명을 덧 붙이는 것을 통해, 작업자가 도커 이미지를 이해하고 적절한 작업을 진행하는데에 많은 도움을 줄 수 있다.

<br>

- 권한이 제한된 컨테이너를 사용하자

  - 컨테이너가 root 와 같은 권한으로 실행이 되어야 하는 경우는 매우 드물다.
  - Node 이미지에서는 `'node'` 유저를 사용하는것이 통상적이다.

<br>

- 최종 결과를 검사 및 확인하자

  - `Dive` 와 같은 Tool 을 사용하여 확인하기 쉬운 빌드 문제점을 미리 확인할 수 있다.

<br>

- 무결성 검사를 하자

  - 이미지를 가져올때, 네트워크가 위험한 이미지를 잘못 가져올 가능성이 존재한다.
  - 기본 도커 프로토콜에는 위와 같은 문제를 방지할 도구가 없지만, `Docker Notary` 라는 Tool을 통해 이를 방지할 수 있다.

<br><br>

## 8.15 도커파일을 lint 하자

> 특수한 도커 린터를 사용하여 잠재적인 결함을 확인함으로써, 성능 및 보안 개선 사항을 쉽게 식별할 수 있으며, 상용코드에서 허비되는 시간과 보안문제를 해결할 수 있다.

> 대표적인 오픈소스 도커 린터로는 `Hadolint` 가 있다. [[Github Link]](https://github.com/hadolint/hadolint)
