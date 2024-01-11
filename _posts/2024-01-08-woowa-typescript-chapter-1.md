---
 title: "우아한 타입스크립트 with 리액트 - 1장"
 date: 2024-01-11 11:00:00 +0900
 categories: typescript
 tags:
   - typescript
   - woowahan
---

# 웹 개발의 역사

자바스크립트는 웹 페이지에서 다양한 콘텐츠를 표현하기 위해 탄생했다. 
자바스크립트는 C, Java와 유사한 기본 문법을 가지고 있으며 프로토타입 기반 상속 개념과 일급 함수 개념을 차용한 경량의 프로그래밍 언어였다.

과거에는 넷스케이프와 마이크로소프트에서 자신들의 브라우저에 새로운 기능을 빠르게 늘리기 시작했다. 이때는 각 기능이 각자의 브라우저에서만 동작했다.
때문에 당시 개발자들은 두 개의 스크립트를 따로 개발하는 어려움을 겪었다.

또한 초기의 자바스크립트는 브라우저 생태계를 고려하여 작성된 것이 아니었다. 이에 따라 자바스크립트와 브라우저의 발전 속도 간의 차이가 나기 시작했다.
자바스크립트에 기능이 추가되어도 런타임 환경인 브라우저가 지원하지 않는다면 무용지물이다. 이런 문제를 해결하기 위해 **폴리필**과 **트랜스파일** 같은 개념이 등장했다.

넷스케이프는 컴퓨터 시스템의 표준을 관리하는 Ecma 인터네셔널에 자바스크립트의 표준화를 위한 자바스크립트 규격을 제출했다. 
Ecma 인터네셔널은 ECMAScript라는 이름으로 자바스크립트 표준화를 공식화했다.

**웹사이트 vs 웹 애플리케이션**

웹페이지는 수집된 데이터 및 정보를 특정 페이지에 표시하기 위한 정적인 웹을 의미한다. 사용자와 상호작용하지 않으며, 콘텐츠가 동적으로 업데이트 되지 않는다.

이와 다르게 웹 애플리케이션은 사용자오 상호작용하는 쌍방향 소통의 웹을 말한다. 
검색, 댓글, 채팅, 좋아요 기능 등 웹페이지 내부에 수많은 애플리케이션이 동작하고 있다.

**컴포넌트 베이스 개발**

대규모 웹 애플리케이션이 등장하면서 표현해야 하는 화면이 점차 복잡해졌다. 이러한 상황과 맞물려 컴포넌트 베이스 개발 방법론이 등장했다. 
작은 단위의 컴포넌트를 조합하여 더 큰 컴포넌트를 만드는 방식이다. 작은 단위의 컴포넌트일수록 다른 컴포넌트에 의존하지 않고 독립적으로 존재한다.
조합 과정에서 필연적으로 컴포넌트간에 의존성이 생기게 된다.

**협업의 필요성 증가**

결과물이 커졌기 때문에 유지보수 및 협업의 중요성은 커졌다. 다른 사람이 작성한 수많은 코드를 이해야한다. 
때문에 코드 가독성을 높이는 것이 중요해졌다. 그런데 자바스크립트는 유지보수에 적합한 언어인가?

# 자바스크립트의 한계

## 동적 타입 언어

자바스크립트의 특징 중 하나는 동적 타입 언어라는 것이다. 이는 변수의 타입이 **런타임**에 결정된다는 것을 의미한다.
이는 실수를 유발하기 쉽고 다른 사람이 작성한 코드를 이해하기 어렵게 만든다.

예를 들어 아래의 코드를 살펴보자. 두 번째, 세 번째 실행은 어떠한 예외를 던지지 않고 정상적으로 수행된다. 그런데 결과물이 정상적이라고 볼 수 있을까?

```javascript
const sumNumber = (a, b) => {
  return a + b;
}

sumNumber(1, 2) // 3
sumNumber(10) // NaN
sumNumber("a", "b") // ab
```

이러한 일이 일어나는 이유는 자바스크립트가 동적 타입 언어이기 때문이다. 
두 번째 실행 결과는 b가 undefined가 되어 `10 + undefined`의 결과가 적절한 타입인 **NaN로 형변환**되어 반환된다.
이렇든 자바스크립트는 관대하게 형변환을 허용하기 때문에 예측할 수 없는 결과물로 프로그램에 치명적인 오류를 만들 가능성이 높다.

## 극복 방안

**JSDoc**

클래스, 메소드 등에 API 문서 생성을 위한 도구이다. 이때 @ts check를 추가하여 타입 및 에러 확인이 가능하다. 
하지만 어디까지나 주석의 성격을 지니기 때문에 강제성을 부여할 수 없다.

**propTypes**

리액트에서 컴포넌트의 props의 타입을 검사하기 위해 사용하는 속성이다. 전체 애플리케이션에 타입 검사하는 용도로는 사용할 수 없다.

**Dart**

구글에서 자바스크립트를 대체하기 위해 제시한 새로운 타이핑이 가능한 언어이다.

## 타입스크립트의 등장

**안정성 보장**

타입스크립트는 컴파일 단계에서 타입 검사를 해주기 때문에 자바스크립트에서 빈번하게 발생하는 타입 에러를 줄일 수 있고, 
런타임 에러를 사전에 방지할 수 있어 안정성이 크게 향상된다.

**개발 생산성 향상**

변수의 타입이 추론되어 IDE에서 자동 완성 기능을 사용할 수 있다. 매번 어떤 props를 넘겨야 하는지 직접 코드를 타가며 확인하지 않아도 된다.

**협업에 유리**

다른 사람이 작성한 코드를 살펴보며 어떤 타입인지 쉽게 파악할 수 있어 협업에 유리하다.

**자바스크립트에 점진적으로 적용 가능**

타입스크립트는 자바스크립트의 슈퍼셋이기 때문에 일괄 전환이 아닌 점진적 도입이 가능하다.