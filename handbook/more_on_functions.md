지역 함수나 다른 모듈에서 불러온 함수, 혹은 클래스의 메서드에서도 함수는 어느 어플리케이션에서 기초적인 구성 요소를 합니다. 함수는 값입니다. 타입스크립트는 함수를 호출할 수 있는 다양한 방법을 제시합니다.

### Function Type Expressions
가장 간단하게 함수를 작성하는 방법은 **함수 타입 표현식(`function type expression`)** 입니다. 이 타입은 화살표 함수와 문법적으로 유사합니다.

```typescript
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}
 
function printToConsole(s: string) {
  console.log(s);
}
 
greeter(printToConsole);
```
`(a: string) => void`라는 문법은 "문자열 타입의 `a`라는 매개변수를 받고 아무 것도 반환하지 않는 함수"를 의미합니다. 함수 선언처럼 매개변수의 타입이 지정되지 않으면 암묵적으로 `any`가 됩니다.

- 매개변수 명은 필수입니다. `(string) => void`는 `any`타입의 `string`이라는 매개변수 명을 갖는 함수를 의미합니다.

당연히 저희는 함수 타입에 타입 별칭(type alias)을 사용할 수 있습니다.

```typescript
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
```

### Call Signatures
자바스크립트에서 함수는 호출 가능할 뿐만 아니라 프로퍼티도 가질 수 있습니다. 하지만 함수 타입 표현식에서는 프로퍼티를 정의하지 못합니다. 만약 프로퍼티를 호출하는 무언가를 작성하고 싶다면 object 타입의 `call signatures`를 사용할 수 있습니다.

```typescript
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
 
function myFunc(someArg: number) {
  return someArg > 3;
}
myFunc.description = "default description";
 
doSomething(myFunc);
```

함수 타입 표현식과 비교하였을 때, 문법이 조금 달라집니다 - `=>` 가 아닌 `:`를 사용

### Construct Signatures
자바스크립트 함수는 `new` 연산자와 함께 호출 가능하다. 타입스크립트는 이를 새로운 객체 생성에 사용되기 때문에 생성자로 간자합니다. `call signature` 앞에 `new`를 추가하여 `construct signature`를 만들 수 있습니다.

```typescript
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```
`Date`와 같은 어떤 객체들은 `new`를 사용하거나 사용하지 않아도 호출할 수 있습니다. 이처럼 `call` 혹은 `construct signatures`를 같은 타입에 임의로 합칠 수 있습니다.

```typescript
interface CallOrConstruct {
  (n?: number): string;
  new (s: string): Date;
}
 
function fn(ctor: CallOrConstruct) {
  // Passing an argument of type `number` to `ctor` matches it against
  // the first definition in the `CallOrConstruct` interface.
  console.log(ctor(10));
               
//(parameter) ctor: CallOrConstruct
//(n?: number) => string
 
  // Similarly, passing an argument of type `string` to `ctor` matches it
  // against the second definition in the `CallOrConstruct` interface.
  console.log(new ctor("10"));
                   
//(parameter) ctor: CallOrConstruct
//new (s: string) => Date
//}
 
fn(Date);
```

### Generic Functions
