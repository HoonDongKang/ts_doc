# The Basics

```typescript
message.toLowerCase();
message();
```

`message`라는 값에 대해 확실하게 알지 못하는 상황에서 해당 코드에 대해서는 다음과 같이 유추해볼 수 밖에 없다.
- `message`는 호출 가능한(callable) 값인가?
- `message`에 `toLowerCase`라는 property가 있는 가
- 만약 그렇다면 `toLowerCase` 또한 호출 가능한가
- 모두 호출 가능한 값이라면, 어떤 값을 반환하는가

```typescript
const message = "Hello World!";
```

`message`가 이렇게 선언이 되어있다고 할 때, `message.toLowerCase()`은 정상적으로 동작할 것이지만 `message()`에 대해서는 에러가 발생할 것이다.

```
TypeError: message is not a function
```

자바스크립트 런타임은 값의 타입을 파악하여 해당 값의 동작과 기능을 결정하고 이를 통해 `TypeError` 를 발생시킬 수 있다.

즉, 자바스크립트는 **동적 타이핑(dynamic typing)** 만 지원하며 실제 코드를 실행시켜서만 결과를 확인할 수 있다.

반면, **정적 타입 시스템(static type system)** 은 코드 실행 전 여떤 결과가 나올지 예측하며 더 높은 안정성을 제공해준다.

### Static type-checking

약간의 변경을 고친 코드에서 에러가 발생하였을 경우에는 즉각적으로 문제를 해결할 수 있지만 

- 에러를 아예 발견하지 못했거나
- 발견하더라도 이미 대규모 리팩토링 혹은 더 많은 코드가 위에 추가가 되었을 경우

에는 쉽게 코드를 해결하기 힘든 상황을 마주한다.

그렇기에 이상적으로 코드 실행 전, 버그를 찾아주는 도구, `TypeScript` 와 같은 static type-checker가 필요하다.


### Non-exception Failures

[ECMAScript specification](https://tc39.es/ecma262/)에서는 예기치 못한 상황을 마주하였을 때, 언어가 어떻게 동작해야하는 지 명확한 지침을 제공해준다. 

```typescript
const user = {
  name: "Daniel",
  age: 26,
};
user.location; // returns undefined
```
예를 들어, 해당 지침에 따르면 호출 불가능한 무언 가를 호출하였을 때 에러를 발생시켜야 한다.
이건  `당연한 동작`처럼 들리지만, 객체 내부에 존재하지 않는 속성에 접근하는 것에 대해서는 자바스크립트는 `undefined`를 반환한다.

궁극적으로 static type system은 자바스크립트가 바로 에러 처리하지 않는 코드 중에서 문제를 발견할 수 있어야 한다.

이런 것들은 코드 작성에 trade-off를 유발시킬 수 있지만 TS는 합법적인 버그들을 골라내는데 주력하고 있다.

```typescript
const announcement = "Hello World!";
 
// How quickly can you spot the typos?
announcement.toLocaleLowercase();
announcement.toLocalLowerCase();
 
// We probably meant to write this...
announcement.toLocaleLowerCase();

function flipCoin() {
  // Meant to be Math.random()
  return Math.random < 0.5;
Operator '<' cannot be applied to types '() => number' and 'number'.
}
```

### Types for Tooling

TS는 코드에서 실수를 잡아주는 것 뿐만 아니라 실수가 발생하는 것을 미리 예방해주기도 한다. 

`type-checker`는 정보가 있다면 어떤 property를 사용할 것인지 미리 제안해주는 기능을 제공한다.

### tsc, the TypeScript compiler

```typescript
tsc hello.ts

// This is an industrial-grade general-purpose greeter function:
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date}!`);
}
 
greet("Brendan");

// Expected 2 arguments, but got 1.
```

`tsc`를 통해 .ts 파일을 컴파일하고 js파일로 변환시킨다.

컴파일 과정에서 오류가 발생하면 해당 오류를 발생시킨다.

### Emitting with Errors

"타입스크립트보다 개발자가 더 잘 알고 있다"라는 타입스크립트의 핵심 가치에 따라서 type-checking에 대한 범위를 제한하고 규칙을 설정할 수 있다.

즉, 타입스크립트는 코드를 막진 않는다. 다만 `noEmitOnError`를 통해 더 철저하게 타입 검사를 받을 수 있으며 이는 타입 오류가 있을 경우 컴파일 시에 JS 파일을 생성하지 않게 설정해준다.

### Explicit Types

```typescript
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

//Argument of type 'string' is not assignable to parameter of type 'Date'.
greet("Maddison", Date());

// O
greet("Maddison", new Date());
```

`greet`이라는 함수가 받는 매개변수에 타입을 명시해주었을 때, `Date()`는 자바스크립트에서 String 값을 반환하기 때문에 오류가 발생한다.

항상 명시적으로 타입을 작성할 필요는 없다. 대부분 타입스크립트는 타입을 작성하지 않아도 자동적으로 추론할 수 있기 때문에, 동일한 타입으로 추론될 것이 예상된다면 주석을 추가하지 않는게 가장 좋다.

### Erased Types
```javascript
"use strict";
function greet(person, date) {
    console.log("Hello ".concat(person, ", today is ").concat(date.toDateString(), "!"));
}
greet("Maddison", new Date());
```

`tsc`를 통해 컴파일하였을 때, 
1. `person`과 `date` 매개변수는 타입 주석이 사라지고
2. `template string`이 `concat`을 통해 순수 string으로 변환되었다.

type annotation은 자바스크립트의 기능이 아니기 때문에 해당 코드를 실행시키기 위해선 결국 타입스크립트는 컴파일러가 필요하며 이를 통해 타입스크립트 만의 특정 코드들은 제거되며 컴파일이 이루어진다.

### Downleveling

타입스크립트는 ECMAScript의 새로운 버전부터 구버전까지 변환시킬 수 있는 기능이 있어 새로운 버전의 기능들을 낮은 버전으로 변경시킬 수 있다. (downleveling)

기본적으로 TS는 굉장히 오래된 버전인 ES5를 기준으로 삼으며 `tsc --target es2015 hello.ts` 처럼 target 버전을 설정할 수 있다.

### Strictness
타입스크립트를 사용하는 사람들의 용도는 모두 다르기 때문에 기본적으로 타입은 optional / interface는 관대하게 적용되며 / null, undefiend 값에 대한 검사는 이루어지지 않는다.

하지만 엄격한 검사를 활성화 시키기 위한 설정도 존재한다.
`tsconfig.json` 에서 `strict: true` 활성화시키면 모든 설정이 활성화되며 개별적으로 나누어서 타입 엄격 검사를 활성화 시킬 수 있다.

- `noImplicitAny` : 암시적으로 `any`타입을 추론시키지 않으며 오류를 발생시킨다.
- `strictNullChecks`: `null`과 `undefined`를 처리하였는 지 확인하며 명시적으로 타입을 처리하도록 설정