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


