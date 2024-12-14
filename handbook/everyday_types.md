### The primitives: string, number, and boolean


- `string`: `Hello, world`같은 string 값
- `number`: `42`같은 숫자 값. 자바스크립트는 `int`나 `float`같은 정수에 대한 특정 값이 없으며 모든 것은 `number`로 통합
- `boolean`: `true` / `false` 같은 값

### Arrays
`[1,2,3]` 같은 배열은 `number[]` 로 타입을 명시할 수 있으며 모든 타입에 대해 배열 타입을 지정할 수 있다. (`string[]`,,,) 

또한 `Array<number>`와 같이 제네릭을 사용한 `T<U>` syntax로 사용 가능하다.

### any
type checking 문제를 유발하는 특적 값에 대해 타입을 명시하고 싶지 않을 때 사용한다.

```typescript
let obj: any = { x: 0 };
// None of the following lines of code will throw compiler errors.
// Using `any` disables all further type checking, and it is assumed
// you know the environment better than TypeScript.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```

- `noImplicitAny` : 타입을 특정짓지 않고 해당 값의 타입을 추론할 수 없을 경우, 타입스크립트 컴파일러는 기본적으로 `any`타입을 부여하지한다. 해당 설정은 암묵적으로 `any` 타입 부여를 막고 에러로 표시한다.

### Type Annotations on Variables

```typescript
let myName: string = "Alice";
```

변수를 선언할 때 타입은 변수의 우측에 명시한다.

대부분 타입스크립트는 자동적으로 타입 추론을 하기 때문에 이러한 코드에서 타입 명시는 크게 필요하진 않을 것이다.

### Functions

타입스크립트는 함수의 `input`과 `output`의 타입을 모두 지정할 수 있다.

#### Parameter Type Annotations
```typescript
// Parameter type annotation
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}

// Would be a runtime error if executed!
greet(42);
```

#### Return Type Annotations
```typescript
function getFavoriteNumber(): number {
  return 26;
}

async function getFavoriteNumber(): Promise<number> {
  return 26;
}
```
함수의 반환값 타입 또한 타입 추론을 통해 지정되기 때문에 명백한 리턴 타입을 지정할 경우를 제외하고는 개인 선호에 따라 타입을 명시할 수 있다.

#### Anonymous Functions

익명 함수는 기존 함수 선언들과는 방식이 조금 다르다. 함수 호출 시점에 함수의 매개변수 타입이 자동으로 지정된다.

```typescript
const names = ["Alice", "Bob", "Eve"];
 
// Contextual typing for function - parameter s inferred to have type string
names.forEach(function (s) {
  console.log(s.toUpperCase());
});
 
// Contextual typing also applies to arrow functions
names.forEach((s) => {
  console.log(s.toUpperCase());
});
```

`s`에 대한 타입이 지정되진 않았지만 `forEach`함수를 통해 배열을 유추하고 `s`의 타입을 추론할 수 있다.

이러한 과정을 **컨텍스트 기반 타입 지정(contextual typing)** 이라고 하며 함수가 발생한 컨텍스트에 따라 타입을 추론한다.

### Object Types

```typescript
// The parameter's type annotation is an object type
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });
```

매개변수 속 모든 property의 타입은 `number`이며 `,` / `;` 를 통해 속성을 분리하고 마지막에는 구분자를 추가하지 않아도 된다.

property에 대한 타입 지정은 optional이며 지정하지 않을 경우,`any` 타입으로 가정된다.

#### Optional Properties
```typescript
function printName(obj: { first: string; last?: string }) {
  // Error - might crash if 'obj.last' wasn't provided!
  console.log(obj.last.toUpperCase());
'obj.last' is possibly 'undefined'.
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }
 
  // A safe alternative using modern JavaScript syntax:
  console.log(obj.last?.toUpperCase());
}
```
`?` 값을 추가함으로써 해당 property를 `optional`로 지정할 수 있다. 

자바스크립트에서 존재하지 않는 property에 접근 시, 에러가 아닌 `undefined`를 출력하며 이 때문에 optional property를 호출하기 전에 반드시 `undefined`인지 확인할 필요가 있다.


### Union Types

타입스크립트에서는 기존 타입을 합쳐 새로운 타입을 생성할 수 있다.

#### Defining a Union Types

`union type`은 두 개 이상의 타입을 결합하여 생성된 타입으로, 해당 타입의 값은 지정된 타입들 중 하나일 수 있음을 나타낸다.

```typescript
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId({ myID: 22342 });
Argument of type '{ myID: number; }' is not assignable to parameter of type 'string | number'.
```

#### Working with Union Types

유니언 타입을 통해 선언된 값은 해당 유니언 타입의 모든 타입 (every memver of the union) 에서 유효한 operation만 허용한다.

`narrow`를 통해 특정 타입만의 메서드를 사용할 수 있다.

```typescript
function printId(id: number | string) {
  console.log(id.toUpperCase());
// Property 'toUpperCase' does not exist on type 'string | number'.
// Property 'toUpperCase' does not exist on type 'number'.
}

//Solution
function printId(id: number | string) {
  if (typeof id === "string") {
    // In this branch, id is of type 'string'
    console.log(id.toUpperCase());
  } else {
    // Here, id is of type 'number'
    console.log(id);
  }
}
```

`union type` 이 마치 각 타입의 교집합처럼 보여 혼란스러울 수도 있지만 `union`이라는 이름은 타입 철학에서 유래한 것이다.


`number | string`과 같은 유니언 타입은 각 타입의 값을 합친 결과이며 두 타입이 각각의 속성을 가지고 있을 때, 해당 타입의 값은 두 타입의 교집합만 적용된다는 점을 유의해야 한다.

예를 들어, **모자를 쓴 키 큰 사람들** 과 **모자를 쓴 스페인 사람들**이 있다면 그들 무리에서 알 수 있는 유일한 사실을 **모자를 쓰고 있다는 점** 이다.


### Type Alias

`type alias`는 특정 타입에 대한 이름을 지정하여 여러 번 사용 가능하게 한다.

타입 별칭은 단순히 타입에 대한 별칭을 붙이는 것일 뿐 새로운 타입을 생성하는 것은 아니다.

```typescript
type UserInputSanitizedString = string;
 
function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}
 
// Create a sanitized input
let userInput = sanitizeInput(getInput());
 
// Can still be re-assigned with a string though
userInput = "new input";
```

### Interfaces

`interface declaration`은 객체 타입에 이름을 지정하는 하나의 방식이다.

```typescript
interface Point {
  x: number;
  y: number;
}
 
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```

`type alias`를 사용하였을 때도 해당 객체 타입에 대해서는 동일하게 동작할 것이다. 타입스크립트는 전달되는 **값의 구조**에 대해서만 고려하고 그 값이 기대하는 속성을 가지고 있는지 여부를 확인한다.

이처럼 타입의 **구조**와 **능력**에만 관심을 가지는 특징을 **구조적 타입 시스템(Structural Typed Type System)**이라고 부른다.

#### Differences Between Type Aliases and Interfaces

`Type aliases`와 `interfaces`는 매우 비슷하며 대부분의 경우에는 자유롭게 선택 가능하다.

거의 모든 `interfaces` 기능은 `Type aliases`에서 사용할 수 있으며 가장 주요한 차이점은 `Type aliases`는 새로운 속성 추가를 위해 re-open이 불가하지만 `interface`는 가능하다.

`Interface`
- Extending an interface
``` typescript
interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}

const bear = getBear();
bear.name;
bear.honey;
```

- Adding new fields to an existing interface
``` typescript
interface Window {
  title: string;
}

interface Window {
  ts: TypeScriptAPI;
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```

`Type`
- Extending a type via intersections
``` typescript
type Animal = {
  name: string;
}

type Bear = Animal & { 
  honey: boolean;
}

const bear = getBear();
bear.name;
bear.honey;
        
```

- A type **cannot** be changed after being created
``` typescript
type Window = {
  title: string;
}

type Window = {
  ts: TypeScriptAPI;
}

 // Error: Duplicate identifier 'Window'.
```

### Type Assetions

가끔은 타입스크립트가 알지 못하는 값의 타입에 대해 사용자가 알고 있을 수도 있다.

예를 들어 `document.getElemetById`의 경우, 타입스크립트는 반환값이 `HTMLelement`의 한 종류인지만 알고 있지만 당신은 특정 ID를 갖는 `HTMLCanvasElement`라는 것을 알수도 있습니다.

```typescript
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;

const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

`Type assertion`으로 특정 타입을 규정시킬 수 있다. 
타입 단언은 컴파일러에 의해 제거되지만 런타임 과정에서 영향을 끼치진 않는다.

> 타입 단언은 컴파일 타임에만 영향을 끼치며 런타임에는 완전히 제거되기 때문에, 잘못된 타입 단언을 사용하더라도 컴파일 시, 경고가 뜨거나 런타임 예의 혹은 `null`이 발생하지 않는다.

타입스크립트는 타입 단언을 해당 타입의 더 세부적이거나 덜 세부적인 타입으로만 허용시킨다. 

```typescript
const x = "hello" as number;
//Conversion of type 'string' to type 'number' may be a mistake 
//because neither type sufficiently overlaps with the other. 
//If this was intentional, convert the expression to 'unknown' first.
```

다만, 복잡한 타입 변환을 허용하지 않기 때문에 두 번의 타입 단언을 통해 강제 변환이 가능하다.

```typescript
const a = expr as any as T;
```

### Literal Types

자바스크립트에서 `var` / `let`으로 할당된 변수는 재할당이 가능하지만 `const`는 불가하다. 이를 토대로 타입스크립트에서 literal로 타입을 생성하게 영향을 주었다.

```typescript
let x: "hello" = "hello";
// OK
x = "hello";
// ...
x = "howdy";
```
하나의 값만 갖는 변수를 선언하기 위해 타입을 지정하는 것은 딱히 쓸모가 있어보이진 않지만

`union`으로 `literal`을 합쳐서 사용하면 훨씬 효율적인 방식으로 사용이 가능하다.

```typescript
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");

function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}

interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
```

#### Literal Inference

```typescript
declare function handleRequest(url: string, method: "GET" | "POST"): void;
 
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
//Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
```

예제에서 `req`가 생성되고 `handleRequest`를 통해 `req.method`에 `GUESS`같은 새로운 `string`이 할당될 수도 있기 때문에 `req.method`는 `GET`이 아닌 `string`으로 추정된다.

1. 타입 단언을 통해 타입 추론 문제 해결
```typescript
// Change 1:
// req.method 값으로는 오로지 "GET"만 올 수 있음을 선언
const req = { url: "https://example.com", method: "GET" as "GET" };
// Change 2
// req.method에는 항상 "GET"만 온다는 것을 선언
handleRequest(req.url, req.method as "GET");
```

2. 객체 전체를 type literal로 고정 시키는 방법
```typescript
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

### `null` and `undefined`

자바스크립트에는 값이 존재하지 않거나 초기화되지 않았음을 나타내는 두 원시값 `null` 과 `undefined`가 있다. 타입스크립트에서는 해당 타입들이 어떻게 동작하는 지 설정하는 옵션들이 존재한다.

#### `strictNullChecks`
 - `off`
`null`과 `undefined`의 값을 갖는 변수에 평소처럼 접근이 가능하며 모든 타입 속성에 할당이 가능하다.

- `on`
`null` 혹은 `undefined` 일 떄, 해당 method나 property를 사용하기 전에 테스트가 선행되어야 한다. (`narrowing`)

```typescript
function doSomething(x: string | null) {
  if (x === null) {
    // do nothing
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
```

#### Non-null Assertion Operator (Postfix !)
타입스크립트에서 `!` 연산자는 `null`과 `undefined`를 타입에서 제거하는 연산자이다.
해당 연산자를 사용하면 타입 추론에서 해당 표현식이 `null` 혹은 `undefined`가 아님을 변경시킬 수 있다.

```typescript
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```

