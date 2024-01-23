---
 title: "우아한 타입스크립트 with 리액트 - 3장 고급 타입"
 date: 2024-01-19 11:00:00 +0900
 categories: typescript
 tags:
   - typescript
   - woowahan
---

타입스크립트만의 독자적 타입 시스템을 알아보자.

# 타입스크립트의 독자적 타입 시스템

## any

any 타입은 자바스크립트에 존재하는 모든 값을 오류 없이 받을 수 있다.
**즉, 타입을 명시하지 않은 것과 동일한 효과를 나타낸다.**

이러한 효과는 **타입스크립트에서 달성하고자하는 정적 타이핑을 무색하게 만든다.**
그렇기 때문에 가능하다면 타입스크립트에서 any 타입을 사용하지 않는 것이 바람직하다.
되도록 `tsconfig.json`에서 `noImplicitAny` 옵션을 활성화하여 타입이 명시되지 않은 변수의 암묵적인 any 타입에 대한 경고를 발생시키자.

하지만 때로는 어쩔 수 없이 any 타입을 사용해야할 경우가 있다.

1. 개발 단계에서 임시로 값을 지정해야 할 때
2. 어떤 값을 받아올지 또는 넘겨줄지 정할 수 없을 때
3. 값을 예측할 수 없을 때 암묵적으로 사용

1번의 경우 그럴 수 있다고 납득이 된다. 다만, 나중에 타입을 명시하는 과정에서 누락하지 않도록 주의해야한다.

2번의 경우 유니온 타입으로 액션을 모두 나타내기 정말 정말 복잡하다면 그럴 수도 있겠다는 생각을 했다.

3번의 경우는 다른 방법이 있지 않나 생각한다. 
예를 들어 API 요청 및 응답의 경우는 런타임에 object validation을 수행하는 것이 좋다고 생각한다.
잘못된 응답을 그대로 수용한다면 해당 값이 계속해서 코드 흐름을 타고 전파되어 더 큰 문제를 만들어 낼 수 있다.
이를 조기에 validation하여 발견한다면 문제를 빠르게 잡아낼 수 있을 것이다.

## unknown

unknown 타입은 any 타입과 유사하게 모든 타입의 값이 할당될 수 있다.
**그러나 any를 제외한 다른 타입으로 선언된 변수에는 unknonw 타입 값을 할당할 수 없다.**

예를 들면 아래의 경우이다.

```typescript
let unknownValue: unknown;

unknownValue = 100;
unknownValue = "hi";

let anyValue: any = unknownValue; // OK
let numberValue: number = unknownValue; // ERROR
let stringValue: string = unknownValue; // ERROR
```

unknown 타입은 어떤 타입이 할당되었는지 알 수 없음을 나타내기 때문에 값을 가져오거나 내부 속성에 접근할 수 없다.
이는 unknown 타입으로 할당된 변수는 어떤 값이든 올 수 있음을 의미하는 동시에 **개발자에게 엄격한 타입 검사를 강제하는 의도를 담고 있다.**

즉, any 타입을 이용하면 컴파일 타임에 잘못된 호출, 속성 접근이 발생해도 문제를 알아 차릴 수 없지만, unknown 타입의 경우 타입이 확실해지는 경우를 식별한 후에 접근할 수 있기 때문에 안전하다.

따라서 대부분의 상황에서는 any를 사용하기보다는 unknown을 사용하는 것이 바람직하다고 볼 수 있다. 다만, 타입 체킹이 굉장히 번거로운 상황에서는 any 사용이 불가피할 수도 있겠다.

## void

함수가 반환하는 값이 없을 때 반환 값은 void가 된다. 자바스크립트에서는 함수에서 명시적인 반환문을 작성하지 않으면 기본적으로 undefined가 반환된다. 하지만 타입스크립트에서는 void 타입이 사용되는데 **이는 undefined가 아니다.**

보통 함수의 반환값이 없을 때는 void로 명시하지 않는 편이다. 컴파일러가 추론해주기 때문이다.

## never

never 타입은 함수에서 값을 반환할 수 없는 경우와 타입 검사 목적으로 사용된다.

아래의 두 함수를 보면 코드 흐름이 더이상 없다. 이럴 때 never를 사용한다.

```typescript
function error(message: string): never {
  throw new Error(message);
}

function loop(): never {
  while (true) {
    console.log("hi");
  }
}
```

never 타입은 모든 타입의 하위 타입이다. **즉, never 자신을 제외한 어떤 타입도 never에 할당될 수 없다.**

## Array

엄밀히 말하자면 자바스크립트에서는 배열을 객체에 속하는 타입으로 분류한다. 
타입스크립트에서는 Array 타입을 사용하기 위해 특수한 문법을 함께 다뤄야하기 때문에 책에서 따로 다루고 있다.

Array 선언은 아래의 두 가지가 가능하다.

```typescript
const array1: number[] = [1, 2, 3];
const array2: Array<number> = [1, 2, 3];
```

타입스크립트에는 튜플이 존재한다. 튜플이란 고정된 길이의 배열이며 원소 갯수와 타입을 보장한다.

```typescript
const tuple: [number, string] = [1, "hi"];
```

튜플은 리액트 훅에서 쉽게 볼 수 있다. useState API는 배열 원소의 자리마다 명확한 의미를 부여한다.

```typescript
const [value, setValue] = useState(false);
```

튜플이 객체보다 나은 점은 사전에 선언된 속성 이름을 따르지 않아도 되어 유연하다는 점이다.
객체의 경우 구조분해할당을 할 때 선언된 속성 이름을 통해 값을 가져와야 하므로 유연성이 떨어진다. 
대신 객체의 경우 속성 순서를 마음대로 정할 수 있는 장점이 있다.

```typescript
const useStateWithObject = (initialValue: any) => {
  // ...
  return { value, setValue};
}

const { value: userName, setValue: setUserName } = useStateWithObject(false);
```

튜플을 활용하면 아래와 같이 배열에서 개수 제한 없이 원소를 받을 수 있다.

```typescript
const httpStatusFromPaths: [number, string, ...string[]] = [
  400,
  "Bad Request",
  "/users/:id",
  "/users/:userId",
  "/users/:uuid",
]
```

## enum

열거형이라 불리는 enum 타입은 다른 언어에서 흔히 볼 수 있는 타입이다.

독특하다고 느낀 부분은 아래와 같이 역방향 접근이 가능하다는 점이다.

```typescript
enum Fruit {
  Apple,
  Orange,
  Strawberry
}

Fruit[2]; // Strawberry
```

주의할 점은 역방향으로 접근할 때 범위를 벗어나는 연산을 수행해도 컴파일 타임에 에러를 보여주지 않는 것이다.

```typescript
Fruit[100]; // No error!
```

이러한 위험을 막기 위해서는 const enum을 사용해볼 수 있겠다. const enum은 역방향 접근을 허용하지 않기 때문이다.

```typescript
const enum Fruit {
  Apple,
  Orange,
  Strawberry
}

Fruit[2]; // Error!
```

책에서는 숫자 상수로 관리되는 열거형은 선언한 값 이외의 값을 할당하거나 접근할 때 이를 방지하지 못한다고 되어있다.
Typescript 5.0 미만의 버전에서는 맞는 말이지만, 5.0 이상의 버전에서는 접근을 방지해주어 에러를 보여준다.
실제로 [Typescript playground](https://www.typescriptlang.org/play?#code/MYewdgzgLgBApmArgWxgMQE6IJawN4BQMMAggA5kA2cANETAPIYCGYA5nAQL4EGiSwAZllwAudCNgBeGACYA3EA)를 접속해보면 5.0 이상의 Typescript 버전에서 에러가 나오는 모습이다.

앞선 예제는 아래와 같이 즉시 실행 함수로 컴파일된다.

```typescript
"use strict";
var Fruit;
(function (Fruit) {
    Fruit[Fruit["Apple"] = 0] = "Apple";
    Fruit[Fruit["Orange"] = 1] = "Orange";
    Fruit[Fruit["Strawberry"] = 2] = "Strawberry";
})(Fruit || (Fruit = {}));
const fruit = 2;
```

이때 일부 번들러에서 트리쉐이킹 과정 중 즉시 실행 함수를 사용하지 않는 코드로 인식하지 못할 수 있다. 
때문에 불필요하게 코드 크기가 증가할 수 있다. 이러한 경우 const enum 또는 as const assertion을 활용하여 enum과 동일한 효과를 얻는 방법이 있다.

# 타입 조합

## 교차 타입(Intersection)

교차 타입은 교집합을 생각하면 된다. 이때 프로퍼티가 교집합이 아닌, 타입이 교집합인 것이다.

아래의 코드를 보자.
프로퍼티가 교집합이라고 생각한다면 겹치는 프로퍼티가 없으니 never 타입이 되어야한다고 생각하게된다.
하지만 타입이 교집합이기 때문에 `AB` 타입은 `A` 타입이면서 동시에 `B` 타입을 만족해야한다.
때문에 `height`와 `width`를 동시에 포함해야한다.

```typescript
type A = { height: number };
type B = { width: number };
type AB = A & B; // { height: number, width: number }
```

## 유니온 타입(Union)

유니온 타입은 합집합을 생각하면된다.

아래의 코드를 보자. `AB` 타입은 `A` 타입 혹은 `B` 타입인 것이다. 
때문에 `ab.height`처럼 어느 한 타입에만 속한 프로퍼티를 접근할 수 없는 것이다.

```typescript
type A = { height: number };
type B = { width: number };
type AB = A | B; // { height: number } | { width: number }

const ab: AB = callApi();
ab.height; // Error
```

## 인덱스 시그니쳐(Index Signatures)

인덱스 시그니처는 특정 타입의 속성 이름을 알 수 없지만 속성 값의 타입을 알고 있을 때 사용한다.

```typescript
interface IndexSigatureEx {
  [key: string]: number;
};
```

## 인덱스드 엑세스 타입(Indexed Access Types)

인덱스드 엑세스 타입은 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용한다.

```typescript
type Student = {
  name: string;
  score: number;
};

type Score = Student['score']; // number
type NameAndScore = Student['name' | 'score']; // number
```

## 맵드 타입(Mapped Types)

맵드 타입은 어느 타입을 기반으로 한 다른 타입을 만들고 싶을 때 사용한다.

아래의 코드에서 중요한 키워드는 `in`이다. 템플릿 타입 `T`의 키들(name, score)이 `K`가 되었고, 이 프로퍼티는 `?` 키워드를 통해 옵셔널 값들이 되며, 타입은 인덱스드 엑세스 타입을 이용하여 본래 타입을 선언한다.

```typescript
type Student = {
  name: string;
  score: number;
};

type Subset<T> = {
  [K in keyof T]?: T[K];
};

const student: Subset<Student> = { name: "minseong" };
```

`?`를 추가하여 optional로 바꾸는 것 외에도 `-?`와 같이 required 값으로 변경할 수 있으며,
`readonly`를 붙여주거나 `-readonly`도 가능하다.

## 템플릿 리터럴 타입(Template Literal Types)

리터럴 문자열을 사용하여 문자열 리터럴 타입을 선언할 수 있는 문법이다.

```typescript
type Stage =
  | 'init'
  | 'select-image'
  | 'edit-image';
type StageName = `${Stage}-stage`;
```

## 제네릭(Generic)

제네릭은 함수, 타입, 클래스 등에서 내부적으로 사용할 타입을 미리 정해두지 않고 타입 변수를 사용해서 해당 위치를 비워 둔 다음에, 실제로 그 값을 사용할 때 외부에서 타입 변수 자리에 타입을 지정하여 사용하는 방식을 말한다. 

이때 `extends` 키워드를 통해 제네릭 타입을 제한할 수 있으며 이를 상한 한계라고 부른다.

```typescript
type ErrorRecord<Key extends string> = Exclude<
  Key, 
  ErrorCodeType extends never 
    ? Partial<Record<Key, boolean>>
    : never;
>
```