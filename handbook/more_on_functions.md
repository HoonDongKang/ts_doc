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

