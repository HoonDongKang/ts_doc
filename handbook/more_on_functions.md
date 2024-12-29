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

함수를 작성할 때, 입력값의 타입은 출력값의 타입과 연관되거나 입력되는 두 가지 입력값의 타입이 동일한 경우가 존재합니다. 배열의 첫 요소를 반환하는 함수를 살펴봅시다.

```typescript
function firstElement(arr: any[]) {
  return arr[0];
}
```

이 함수는 동작하는데 문제는 없지만 반환 타입이 `any`로 되어 있습니다. 반환 값에도 타입을 지정해준다면 더 좋겠죠.

타입스크립트에서 `generics`는 두 값 간의 대응 관계를 설명하고자 할 때 사용됩니다. 함수 시그니처에 `type parameter`를 선언하여 사용해봅시다.

```typescript
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}
```

`Type`이라는 타입 매개변수를 함수에 추가해주고 두 곳에서 사용해줌으로써, 함수의 입력값(배열)과 출력값(반환값)에 연관관계를 형성시켜주었습니다. 이제 함수를 호출하면 더 자세한 타입이 나올 것입니다.

```typescript
// s is of type 'string'
const s = firstElement(["a", "b", "c"]);
// n is of type 'number'
const n = firstElement([1, 2, 3]);
// u is of type undefined
const u = firstElement([]);
```

#### Inference

우리는 `Type`이라는 것을 구별짓지 않았습니다. 타입은 타입스크립트에 의해 추론되어 자동적으로 선택된 것입니다.

또한, 여러 개의 타입 매개변수를 사용할 수 도 있습니다. 

```typescript
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}
 
// Parameter 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```

해당 예제에서는 타입스크립트가 `Input` 타입 매개변수(문자열 배열)와 함수 표현식의 반환값을 기반으로 얻은 숫자 타입의 `Output`을 추론할 수 있었습니다.

#### Constraints

우리는 어떠한 값에도 동작이 가능한 제네릭 함수를 사용해왔습니다. 하지만 때로는 두 값을 연결짓지만 특정 값의 하위 집합에서만 실행시킬 수도 있습니다. **제약조건(`constraint`)** 을 통해 타입 매개변수가 받을 수 있는 타입의 값을 제한할 수 있습니다.

두 개의 값 중 더 긴 값을 반환하는 함수를 작성해봅시다. 이를 위해서 숫자 타입의 `length` 속성이 필요합니다. 우리는 `extends`절을 통해 타입 매개변수를 제한할 수 있습니다.

```typescript
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
 
// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
//Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.
```

이 예제에서는 흥미로운 점이 있습니다. 우리는 타입스크립트가 `longest`의 반환 값의 타입을 추론하게 하였습니다. 반환 값 추론은 제네릭 함수에서도 가능합니다.

우리는 `Type`을 `{ length: number }`로 제한했기 때문에 `a` 와 `b` 매개변수의 `.length` 속성에 접근할 수 있었습니다. 해당 타입 제약조건이 없었다면 매개변수는 `length`속성이 없는 다른 타입일 수도 있기 때문에 이러한 속성들에 접근할 수 없었을 것입니다.

`longerArray`와 `longerString`의 타입들은 매개변수를 통해 추론되었습니다. 제네릭은 동일한 타입으로 두 개 이상의 값을 연결하는데 초점이 맞춰져있습니다.

마지막으로 `longest(10, 100)`은 `number` 타입이 `.length` 속성을 갖고 있지 않기 때문에 거부되었습니다.

#### Working with Contrained Value

제네릭 제약조건을 이용할 때, 자주 등장하는 에러입니다.
```typescript
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum };
//Type '{ length: number; }' is not assignable to type 'Type'.
//'{ length: number; }' is assignable to the constraint of type 'Type', 
// but 'Type' could be instantiated with a different subtype of constraint '{ length: number; }'.
  }
}
```

겉으로 보기에는 문제없어 보입니다. `Type`은 `{ length: number }`로 제한되어 있고 함수는 `Type` 혹은 제약 조건과 일치하는 값을 반환합니다. **문제는 단순히 제약 조건에 일치하는 어떤 객체도 아닌 함수가 전달된 값과 동일한 종류의 객체를 반환해준다는 것입니다.** 만약 이 코드가 유효하다면 분명히 작동하지 않을 코드를 작성할 수도 있습니다.

```typescript
// 'arr' gets value { length: 6 }
const arr = minimumLength([1, 2, 3], 6);
// and crashes here because arrays have
// a 'slice' method, but not the returned object!
console.log(arr.slice(0));
```

#### Specifying Type Arguments
타입스크립트는 보통 제네릭에서 의도된 타입 매개변수를 추론할 수 있지만 항상은 아닙니다.

```typescript
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}
```

일반적으로 잘못된 배열과 함께 함수를 호출하면 에러가 발생할 것입니다.

```ts
const arr = combine([1, 2, 3], ["hello"]);
//Type 'string' is not assignable to type 'number'.
```

이러한 의도로 작성하기 위해서는 `Type`을 정의해야합니다.

```ts
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```

#### Guidelines for Writing Good Generic Functions

제네릭 함수를 사용하는 것은 재밌지만 타입 매개변수를 과도하게 사용하는 경우가 있습니다. 과도한 타입 매개변수 사용이나 불필요한 제약 조건을 추가하면 타임 추론이 제대로 작동하지 않아 함수 호출자에게 혼란을 줄 수 있습니다.

**Push Type Parameters Down**
```ts
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}
 
function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}
 
// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```

두 함수는 처음에 동일해 보일 수 있지만 `firstElement1`이 훨씬 더 좋은 방식입니다. `Type`을 추론하여 반환하지만 `firstElement2`는 `any`를 반환 타입으로 추론합니다. 타입스크립트는 `arr[0]`표현식을 호출 시점까지 기다리지 않고 제약 조건에 따라 해결하기 때문입니다.

- 가능하다면 타입 매개변수를 제한하지 말고, 타입 매개변수 그 자체로 사용하세요.

**Use Fewer Type Parameters**

```ts
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}
 
function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func
): Type[] {
  return arr.filter(func);
}
```
`Func`라는 타입 매개변수를 만들었지만 두 값 간의 관계를 나타내지 않습니다. 이건 항상 경고 신호입니다. 이런 경우에는 호출자는 아무런 이유없이 추가 적인 타입 매개변수를 강제로 지정하여 호출해야 합니다. `Func`는 아무 의미가 없지만 함수의 가독성이나 이해를 해칩니다.

- 항상 타입 매개변수는 최대한 적게 사용하라

**Type Paraemters Should Appear Twice**

```ts
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}
 
greet("world");
```

``` ts
function greet(s: string) {
  console.log("Hello, " + s);
}
```

타입 매개변수는 여러 값의 타입을 연결하는 데 사용됩니다. 만약 타입 매개변수가 함수 시그니처에서 단 한 번만 사용된다면 이는 아무 것도 연결짓지 않는 것이죠. 

이 규칙은 반환 값 추론에도 사용됩니다. 예를 들어, `Str`이 `greet` 함수의 추론된 반환 타입의 일부라면 이는 매개변수와 반환 타입을 연결하므로 코드에 한 번 나오더라도 결국 두 번 사용하게 된 것입니다.

- 타입 매개변수가 한 곳에서만 사용된다면 해당 타입 매개변수가 정말 필요한 지 신중히 고민하세요.

### Optional Parameters

자바스크립트에서 함수는 몇 개의 인수를 받지만 `toFixed`처럼 선택적으로 받기도 합니다.

```ts
function f(n: number) {
  console.log(n.toFixed()); // 0 arguments
  console.log(n.toFixed(3)); // 1 argument
}
```

타입스크립트에서는 매개변수를 optional하게 받기 위해 `?`를 사용합니다.

```ts
function f(x?: number) {
  // ...
}
f(); // OK
f(10); // OK
```

매개변수가 `number` 타입이어도 `x`는 자바스크립트에서 특정되지 않는 매개변수는 `undefined`를 갖기에, 실제로 `number | undefined`로 타입을 형성할 것입니다.

`f` 함수에서 `x`는 `undefined`한 인수들은 `10`으로 변경되기에 `number` 타입을 갖게 됩니다. 만약 매개변수가 optional하다면 호출자는 항상 `undefined`를 전달할 수 있다는 것이며 이는 단순히 누락된 인수를 시뮬레이션하는 것과 같습니다.

```ts
// All OK
f();
f(10);
f(undefined);
```

#### Optional Parameters in Callbacks

당신이 optional parameter와 함수 타입 표현식을 배웠다면, 다음 예제는 콜백함수를 호출하는 함수를 작성할 때 가장 접하기 쉬운 실수일 것입니다.

```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```

사람들이 `index?`를 선택적 매개변수로 작성할 때, 일반적으로 의도하는 것은 아래 두 가지 호출 모두 유효하길 원한다는 것입니다.

```ts
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```

이것이 실제로 의미하는 바는, 콜백이 하나의 인수로 호출될 수 있다는 것입니다. 다시 말해, 함수 정의는 다음과 같이 구현됩니다.

```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // I don't feel like providing the index today
    callback(arr[i]);
  }
}
```

결과적으로 타입스크립트는 이 의미를 강제하며, 실제로 발생하지 않는 오류를 발생시킬 수 있습니다.

```ts
myForEach([1, 2, 3], (a, i) => {
  console.log(i.toFixed());
//'i' is possibly 'undefined'.
});
```

자바스크립트에서 매개변수보다 더 많은 인수를 전달받을 경우, 초과된 인수들을 무시됩니다. **타입스크립트도 동일하게, 매개변수 수가 적더라도 함수의 타입이 호환된다면 더 많은 매개변수를 갖는 함수 자리에 사용될 수 있습니다.**


```ts
// 더 많은 매개변수를 갖는 타입
type CallbackWithThreeParams = (a: number, b: number, c: number) => void;

// 매개변수 수가 적은 함수
const callbackWithTwoParams: (a: number, b: number) => void = (a, b) => {
    console.log(a, b);
};

// 함수 타입이 호환되지만 초과된 매개변수는 무시
const example: CallbackWithThreeParams = callbackWithTwoParams;

example(1, 2, 3); // 1, 2
```

### Function Overloads
몇몇 자바스크립트 함수들은 다양한 개수와 타입의 인수들과 함께 호출됩니다. 예를 들어, 당신은 타임스탬프를 지정(하나의 인수로)하거나 월/일/년(세 개의 인수)를 받아 `Date`를 생성하는 함수를 작성할 수 있습니다.

타입스크립트에서는 `overload signatrues`를 통해 함수를 다양한 방식으로 호출할 수 있습니다. 이를 위해, 두 개 이상의 함수 시그니처를 정의해봅시다.

```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);

const d3 = makeDate(1, 3);
//No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.
```

위 예제에서 두 개의 오버로드를 작성하였습니다. 하나는 하나의 인수를 받고 다른 하나는 3개의 인수를 받습니다.


그 다음 우리는 호환 가능한 시그니처를 가진 함수 구현을 작성하였습니다. 함수들은 구현 시그니처를 가지고 있지만, 이 시그니처를 직접 호출할 수 는 없습니다. 필수 매개변수 하나 뒤에 두 개의 선택적 매개변수를 작성했음에도 두 개의 매개변수로 호출할 수 없습니다.

#### Overload Signatures and the Implementation Signature

이것은 흔히 혼란을 일으키는 문제입니다. 사람들은 이렇게 코드를 작성하고 왜 오류가 발생하는 지 알지 못합니다.

```ts
function fn(x: string): void;
function fn() {
  // ...
}
// Expected to be able to call with zero arguments
fn();
Expected 1 arguments, but got 0.
```

함수 본문을 작성하는데 사용된 시그니처는 외부에서 보이지 않습니다.
- 구현의 시그니처는 외부에서 볼 수 없습니다. 오버로드 된 함수를 작성할 때 당신은 항상 두개 이상의 시그니처를 함수 구현 위에 작성하여야 합니다.

구현 시그니처는 오버로드된 시그니처와 호환되어야 합니다. 예로 이러한 함수들은 구현된 시그니처가 오버로드된 시그니처와 일치하지 않아 발생하는 오류입니다.

```ts
function fn(x: boolean): void;

// Argument type isn't right
function fn(x: string): void;
//This overload signature is not compatible with its implementation signature.

function fn(x: boolean) {}

////////////////////////////////////////////////////////

function fn(x: string): string;

// Return type isn't right
function fn(x: number): boolean;
//This overload signature is not compatible with its implementation signature.

function fn(x: string | number) {
  return "oops";
}
```

### Writing Good Overloads

제네릭처럼 함수 오버로드를 작성하기 위해 따라야할 몇가지 가이드라인이 있습니다. 이러한 원칙들을 따르는 것은 당신이 작성한 함수가 더 쉽게 호출되고, 쉽게 이해되며 쉽게 구현하게 만들어줍니다.

문자열 혹은 배열의 길이를 반환하는 함수를 작성해봅시다.

```ts
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
  return x.length;
}
```

해당 함수는 문자열들 혹은 배열로 호출되니 괜찮아보이지만 문자열일 수도 있고 배열일 수도 있는 값을 사용하여 호출할 수는 없습니다.

왜냐하면 타입스크립트는 **함수 호출을 단일 오버로드에만 매핑**할 수 있기 때문입니다.

```ts
len(""); // OK
len([0]); // OK
len(Math.random() > 0.5 ? "hello" : [0]);
//No overload matches this call.
//  Overload 1 of 2, '(s: string): number', gave the following error.
//    Argument of type 'number[] | "hello"' is not assignable to parameter of type 'string'.
//      Type 'number[]' is not assignable to type 'string'.
//  Overload 2 of 2, '(arr: any[]): number', gave the following error.
//    Argument of type 'number[] | "hello"' is not assignable to parameter of type 'any[]'.
//      Type 'string' is not assignable to type 'any[]'.
```

두 오버로드 모두 같은 인수 개수와 같은 반환 타입을 갖고 있기 때문에 우리는 해당 함수를 오버로드 하지 않고 작성해야 합니다.

```ts
function len(x: any[] | string) {
  return x.length;
}
```

호출자는 두 가지 유형의 값으로 이 함수를 호출할 수 있고 올바른 구현 시그니처를 고민할 필요가 없어집니다.

- 가능한 경우 오버로드보다 유니온 타입을 선호하는 매개변수를 사용하세요.

### Declaring `this` in a Function

타입스크립트는 코드 흐름 분석을 통해 함수 내에서 `this`가 무엇이어야 하는지 추론합니다.

```ts
const user = {
  id: 123,
 
  admin: false,
  becomeAdmin: function () {
    this.admin = true;
  },
};
```
타입스크립트는 `user.becomeAdmin` 함수가 외부 객체 `user`에 `this`가 일치한다는 것을 알고 있습니다. `this`가 어떤 객체를 나타내는 지에 대해 더 많은 제어가 필요한 경우도 있습니다.

자바스크립트 명세에 따르면 `this`는 함수 매개변수로 사용할 수 없습니다. 타입스크립트는 함수 본문에서 구문적 공간을 사용하여 `this`의 타입을 선언하도록 지원합니다.

```ts
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}
 
const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```

이러한 패턴은 흔히 콜백 스타일의 API에서 자주 사용됩니다. 이런 경우 다른 객체가 보통 언제 당신의 함수가 호출될 지를 제어합니다. 이러한 동작을 사용하려면 화살표 함수가 아닌 `function`을 사용해야 합니다.

```ts
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}
 
const db = getDB();
const admins = db.filterUsers(() => this.admin);
//The containing arrow function captures the global value of 'this'.
//Element implicitly has an 'any' type because type 'typeof globalThis' has no index signature.
```

### Other Types to Know About

함수 타입을 다루면서 등장하는 또다른 타입들을 접하신 적이 있으실 겁니다. 모든 타입들처럼 당신도 모든 곳에서 사용할 수 있지만 아래 타입들은 특히 함수 문맥에서 관련이 깊습니다.

#### `void`

`void`는 반환하지 않는 함수에서 반환할 때 사용됩니다. 함수에서 `return` 문이 존재하지 않거나 명시하지 않았을 경우에 추론되는 타입입니다.

```ts
// The inferred return type is void
function noop() {
  return;
}
```

자바스크립트에서 값을 반환하지 않는 함수는 암묵적으로 `undefined`값을 반환합니다. 하지만 **타입스크립트에서 `void`와 `undefined`는 동일하지 않습니다. **

#### `object`

원시값(`string`, `number`, `bigint`, `boolean`, `symbol`, `null`, `undefined`)이 아닌 모든 값은 `object` 타입입니다. 이것은 빈 객체 타입(`{}`)와는 다릅니다. 또한 전역 타입 `Object` 과도 다릅니다. 대부분의 경우, `Object`타입은 거의 사용하지 않습니다.
 
 - `object`는 `Object`가 아닙니다. 항상 `object`를 사용하세요.
 
자바스크립트에서 함수는 객체입니다. 함수는 속성을 가질 수 있고 `Object.prototype`이 프로토타입 체인에 포함되며 `instanceof Object`에도 해당되고 `Object.keys`를 호출할 수도 있습니다.
 
 이러한 이유로 함수는 타입스크립트에서 `object`로 간주됩니다.
 
 #### `unknown` 
 
 `unknown`타입은 모든 값을 나타냅니다. `any`타입과 유사하지만 `unknown` 값으로 어떠한 작업도 수행할 수 없기 때문에 더 안전합니다.
 
 ```ts
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b();
//'a' is of type 'unknown'.
}
```

이것은 함수 타입을 설명할 때 유용합니다. 함수 본문에 `any`타입이 아닌 모든 값들을 받을 때 유용합니다.

반대로 반환 값의 타입을 알 수 없을 때도 사용 가능합니다.

```ts
function safeParse(s: string): unknown {
  return JSON.parse(s);
}
 
// Need to be careful with 'obj'!
const obj = safeParse(someRandomString);
```

#### `never`
반환값이 절대 존재하지 않는 함수도 있다.

```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```

`never`타입은 절대로 관찰될 수 없는 값을 의미합니다. 해당 타입의 반환값은 예외 처리하거나 프로그램의 종료를 의미합니다.

`never`는 또한 타입스크립트 유니온 타입에서 더이상 남지 않았을 경우에 등장합니다.

```ts
function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    x; // has type 'never'!
  }
}
```

#### Function
전역 타입 `Function`은 자바스크립트의 모든 함수 값에 존재하는 `bind`, `call`, `apply` 와 같은 속성을 설명합니다. 또한 `Function` 타입의 값은 항상 호출할 수 있는 특별한 속성을 가지며 해당 호출은 `any`를 반환합니다.

```ts
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```

이것은 타입이 지정되지 않은 함수 호출이며 `any`를 반환하기 때문에 일반적으로 피하는 것이 좋습니다. 만약 당신이 임의를 함수를 받아야 하지만 호출할 의도가 없다면 `() => void` 처럼 타입을 지정하는 것이 더 안전합니다.

### Rest Parameters and Arguments

#### Rest Parameters
고정된 인수의 개수를 넘어 다양한 인수를 받기 위해 선택적 매개변수 혹은 오버로드를 사용하는 것 이외에도 `rest parameters`를 통해 무제한 개수의 인수를 받는 함수를 저으이할 수 있습니다.

`rest parameter`는 다른 모든 매개변수 뒤에 나타나며, `...`문법을 사용합니다.

```ts
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);0
```

타입스크립트에서 이런 매개변수에 대한 타입 지정은 암묵적으로 `any[]`타입을 갖으며 모든 타입 지정은 `Array<T>` 혹은 `T[]`, 튜플 타입이어야 합니다.

#### Rest Arguments

반대로, 이터러블 객체에서 가변적인 수의 인수를 제공하려면 spread 문법을 사용할 수 있습니다. 예를 들어 `push` 배열 메서드는 가변적인 수의 인수도 받을 수 있습니다.

```ts
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```

일반적으로 타입스크립트는 배열이 불변하다고 가정하지 않습니다. 이것은 예상치 못한 동작을 야기할 수 있습니다.
```ts
// Inferred type is number[] -- "an array with zero or more numbers",
// not specifically two numbers
const args = [8, 5];
const angle = Math.atan2(...args);
//A spread argument must either have a tuple type or be passed to a rest parameter.


// Inferred as 2-length tuple
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```


### Parameter Destructuring

구조분해를 사용하면 함수 본문에서 인수로 제공된 객체를 하나 이상의 지역 변수로 편리하게 풀어낼 수 있습니다. 자바스크립트에서는 다음과 같습니다.

```js
function sum({ a, b, c }) {
  console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```

구조 분해한 객체의 타입 지정은 다음과 같습니다.
```ts
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```

다소 장황해보일 수 있지만 이름이 지정된 타입을 사용할 수 있습니다.

```ts
// Same as prior example
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

### Assignability of Functions

#### Return type `void`

함수의 반환 값의 타입이 `void` 인 경우는 일반적이진 않지만 예상되는 동작을 할 수 있습니다.

`void`반환 타입을 가진 함수에 대한 문맥적 타이핑은 함수의 값은 반환하지 않도록 강제하지 않습니다. 다시 말해, `void` 반환 타입을 가진 문맥적 함수 타입(`type voidFunc = () => void`) 은 구현 시 다른 값을 반환할 수 있지만 그 값은 무시됩니다.

따라서 다음과 같은 `() => void` 타입의 구현은 유효합니다.

```ts
type voidFunc = () => void;
 
const f1: voidFunc = () => {
  return true;
};
 
const f2: voidFunc = () => true;
 
const f3: voidFunc = function () {
  return true;
};

// 해당 함수들로부터 값을 반환받아도 void 타입을 유지한다.

const v1 = f1(); //void
 
const v2 = f2(); //void
 
const v3 = f3(); //void
```

이 동작은 `Array.prototyp.push`가 숫자를 반환하고 `Array.prototyp.forEach`가 `void`반환 타입을 가진 함수를 인자로 받길 기대하는 경우에도 유효하도록 하기 위해 존재합니다.

```ts
const src = [1, 2, 3];
const dst = [0];
 
src.forEach((el) => dst.push(el));
// push는 number를 반환하지만 forEach는 void를 반환하는 함수를 받음
// 하지만 void는 결국 인수로 다 받을 수는 있음(어차피 할당되면 void)
```

리터럴 함수 정의에서 반환 타입이 `void`인 경우, 그 함수는 아무 것도 반환할 수 없습니다.

```ts
function f2(): void {
  // @ts-expect-error
  return true;
}
 
const f3 = function (): void {
  // @ts-expect-error
  return true;
};
```
