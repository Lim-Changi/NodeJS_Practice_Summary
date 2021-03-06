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
