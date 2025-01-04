자바스크립트에서 데이터를 그룹화하고 전달하는 가장 기본적인 방법은 객체를 사용하는 것입니다. 타입스크립트에서 이러한 것을 객체 타입(`object types`)라고 부릅니다.

익명으로도 사용가능합니다.
```ts
function greet(person: { name: string; age: number }) {
  return "Hello " + person.name;
}
```

또는 interface를 사용하여 이름을 지정할 수도 있고
```ts
interface Person {
  name: string;
  age: number;
}
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```

타입 별칭을 사용할 수도 있습니다.
```ts
type Person = {
  name: string;
  age: number;
};
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```

세 가지 예제들 모두 `string` 타입의 `name`과 `number`타입의 `age`를 갖는 객체를 활용한 함수를 활용하였습니다.

> 타입스크립트에서 제공해주는 cheat-sheet [types and interface - cheat-sheets](https://www.typescriptlang.org/cheatsheets/)


### Property Modifiers

객체 타입의 각 속성은 타입, optional, 쓰여질 수가 있는 지 등을 지정할 수 있습니다.

#### Optional Properties

대부분의 경우, 특정 속성이 적용될 수도 있는 객체를 다루게 됩니다. 이러한 상황에서 우리는 속성명 끝에 물음표(`?`)를 추가하여 해당 속성을 선택적으로 표시할 수 있습니다.

```ts
interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}
 
function paintShape(opts: PaintOptions) {
  // ...
}
 
const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```

이 예제에서 `xPos`와 `yPos`는 선택적으로 적용되었습니다. 우리는 둘 중 하나를 제공하거나 안할 수 있으므로, `paintShpat`에 대한 모든 호출은 유효합니다. 선택적 속성이라는 것은 해당 속성이 설정되면 더 세밀한 타입을 가지게 된다는 것을 알려주는 것입니다.

또한 `strictNullChecks`가 설정되어 있을 때, 타입스크립트는 해당 속성들이 잠재적으로 `undefined`라는 것을 보여줍니다.

```ts
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos;                 
//(property) PaintOptions.xPos?: number | undefined
  let yPos = opts.yPos;
//(property) PaintOptions.yPos?: number | undefined
  // ...
}
```

자바스크립트에서 속성이 정해지지 않았더라도 우리는 해당 속성에 접근할 수 있었습니다. `undefined` 값을 반환하였고 `undefined`를 통해 해당 값을 검사할 수 있습니다.

```ts
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos === undefined ? 0 : opts.xPos;
///let xPos: number
  let yPos = opts.yPos === undefined ? 0 : opts.yPos;
///let yPos: number
  // ...
}
```

아래와 같이 특정되지 않은 값의 기본값을 설정하는 것은 자바스크립트가 지원하는 문법 중 하나입니다.

```ts
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
  console.log("x coordinate at", xPos);
//(parameter) xPos: number
  console.log("y coordinate at", yPos);
//(parameter) yPos: number
  // ...
}
```

 `paintShape`의 매개변수에 구조 분해 할당을 사용하고 `xPos`와 `yPos`에 기본값을 제공하였습니다. 이제 `xPost`와 `yPos` 모두 확실히 존재하지만 `paintShape`를 호출하는 쪽에서 선택적으로 전달할 수 있습니다.

 현재 구조 분해 패턴 내에 타입 주석을 추가하는 방법은 없습니다. 이는 다음과 같은 뭄법이 이미 자바스크립트에서 다른 의미를 가지고 있기 때문입니다.

 ```js
 function draw({ shape: Shape, xPos: number = 100 /*...*/ }) {
  render(shape);
//Cannot find name 'shape'. Did you mean 'Shape'?
  render(xPos);
//Cannot find name 'xPos'.
}
 ```

 객체 구조분해 패턴에서 `shape: Shape`는 "속성 `shape`를 가져와 로컬 변수 `Shape`로 재정의한다"라는 의미를 갖습니다. 마찬가지로 `xPos: number`는 매개변수 `xPos`값을 기반으로 변수 `numver`를 생성합니다.

 #### `readonly` Properties

 타입스크립트에서 속성들은 `readonly`로 설정될 수 있습니다. 이는 런타임에서 동작에 아무런 영향을 주지 않지만, 타입 검사를 수행하는 동안 `readonly`로 설정된 속성들은 쓸 수 없게 만듭니다.

 ```ts
interface SomeType {
  readonly prop: string;
}
 
function doSomething(obj: SomeType) {
  // We can read from 'obj.prop'.
  console.log(`prop has the value '${obj.prop}'.`);
 
  // But we can't re-assign it.
  obj.prop = "hello";
//Cannot assign to 'prop' because it is a read-only property.
}
```

`readonly`를 사용한다고 해서 값이 완전히 변경 불가능하다는 것을 의미하지 않습니다. 즉, 내부 내용이 변경될 수 없다는 것은 아닙니다. 단지 해당 속성 자체를 다시 쓸 수 없다는 것을 의미합니다.

```ts
interface Home {
  readonly resident: { name: string; age: number };
}
 
function visitForBirthday(home: Home) {
  // We can read and update properties from 'home.resident'.
  console.log(`Happy birthday ${home.resident.name}!`);
  home.resident.age++;
}
 
function evict(home: Home) {
  // But we can't write to the 'resident' property itself on a 'Home'.
  home.resident = {
//Cannot assign to 'resident' because it is a read-only property.
    name: "Victor the Evictor",
    age: 42,
  };
}
```

`readonly`가 무엇을 암시하는지 제대로 파악하는 것이 중요합니다. 타입스크립트에서는 개발 시점에 객체가 어떻게 사용되는 지에 대한 의도를 나타나는데에 `readonly`가 유용합니다. 하지만 타입스크립트는 두 개의 타입이 호환 가능할 때, `readonly`를 고려하지 않습니다. 따라서 `readonly` 속성들은 별칭(`aliasing`)을 통해 변경될 수 있습니다.

```ts
interface Person {
  name: string;
  age: number;
}
 
interface ReadonlyPerson {
  readonly name: string;
  readonly age: number;
}
 
let writablePerson: Person = {
  name: "Person McPersonface",
  age: 42,
};
 
// works
let readonlyPerson: ReadonlyPerson = writablePerson;
 
console.log(readonlyPerson.age); // prints '42'
writablePerson.age++;
console.log(readonlyPerson.age); // prints '43'
```


#### Index Signatures

가끔은 타입의 모든 속성들의 이름을 미리 알 수 없지만 값의 형태는 알고 있을 때가 있습니다. 이런 경우에 인덱스 시그니처(`index signature`)를 통해 값의 타입을 설명할 수 있습니다.

```ts
interface StringArray {
  [index: number]: string;
}
 
const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
//const secondItem: string
```

위에 우리는 인덱스 시그니처를 갖는 `StringArray` 인터페이스가 있고 인덱스 시그니처는 `StringArray`가 숫자형으로 인덱싱 되어 있을 경우, 문자열을 반환한다는 것을 알려줍니다.

인덱스 시그니처 속성에는 다음과 같은 타입만 사용 가능합니다. - `string`, `number`, `symbol`, 템플릿 문자열 패턴, 이러한 타입들로 구성된 `union` 타입

여러 타입의 인덱서를 지원하는 것도 가능합니다. 하지만 `number`와 `string` 인덱서를 함께 사용할 때는 숫자 인덱서를 통해 반환되는 타입은 문자열 인덱서를 통해 반환되는 타입의 서브타입이어야 합니다.

그 이유는 숫자로 인덱싱을 할 때, 자바스크립트는 실제로 숫자를 문자열로 변환시킨 후 객체에 접근하기 때문입니다. 예를 들어 숫자 `100`을 인덱싱한단느 것은 문자열 `"100"`을 인덱싱하는 것과 동일하기 때문에 일관성을 유지할 필요가 있습니다. 

```ts
interface Animal {
  name: string;
}
 
interface Dog extends Animal {
  breed: string;
}
 
// Error: indexing with a numeric string might get you a completely separate type of Animal!
interface NotOkay {
  [x: number]: Animal;
//'number' index type 'Animal' is not assignable to 'string' index type 'Dog'.
  [x: string]: Dog;
}
```

문자열 인덱스는 dictionary pattern을 설명하는 강력한 방법이지만 동시에 모든 속성의 반환 타입이 일치하도록 강제합니다. 이것은 문자열 인덱스가 `obj.property`가 `obj["property"]`로도 사용할 수 있도록 선언하기 때문입니다.
(* dictionary pattern: 데이터를 키를 통해 빠르게 조회가능한 데이터 구조)

아래 예시에서 `name`타입은 문자열 인덱스 타입과 일치하지 않기 때문에 오류가 발생합니다.

```ts
interface NumberDictionary {
  [index: string]: number;
 
  length: number; // ok
  name: string;
//Property 'name' of type 'string' is not assignable to 'string' index type 'number'.
}
```
하지만 인덱스 시그니처가 속성 타입들의 유니온으로 지정되어 있다면 다른 타입도 사용 가능합니다.
```ts
interface NumberOrStringDictionary {
  [index: string]: number | string;
  length: number; // ok, length is a number
  name: string; // ok, name is a string
}
```
마지막으로 인덱스 시그니처를 `readonly`로 설정하여 해당 인덱스에 값을 할당하지 못하도록 지정할 수 있습니다.
```ts
interface ReadonlyStringArray {
  readonly [index: number]: string;
}
 
let myArray: ReadonlyStringArray = getReadOnlyStringArray();
myArray[2] = "Mallory";
//Index signature in type 'ReadonlyStringArray' only permits reading.
```

### Excess Property Checks

객체가 타입을 할당받는 위치와 방식은 타입 시스템에서 차이를 만들 수 있습니다. 그 중 핵심적인 예시로는 초과 속성 검사(`exess property checking`)이 있습니다. 이는 객체가 생성되고 타입이 할당될 때 더 철저히 검사하는 방식입니다.

```ts
interface SquareConfig {
  color?: string;
  width?: number;
}
 
function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || "red",
    area: config.width ? config.width * config.width : 20,
  };
}
 
let mySquare = createSquare({ colour: "red", width: 100 });
//Object literal may only specify known properties, but 'colour' does not exist in type 'SquareConfig'. Did you mean to write 'color'?
```

`createSquare` 함수에 전달된 인수가 `color` 대신 `colour`로 전달되어 오류가 발생하였습니다. 일반적으로 자바스크립트에서는 이러한 오류가 조용히 발생합니다.

위 코드가 올바르게 타입이 지정되었다고 주장할 수도 있습니다. `width`속성은 호환이되며 `color`속성은 전달되지 않았고 `colour`속성은 중요하지 않기 때문입니다.

하지만 타입스크립트는 이 코드에 버그가 있을 가능성이 높다고 간주합니다. 객체 리터럴은 특별한 처리를 받아, 다른 변수에 할당되거나 인수로 전달될 때 초과 속성 검사를 수행합니다. 만약 객체 리터럴이 대상 타입에 없는 속성일 경우, 오류가 발생합니다.

해당 검사들을 우회하는 방식은 굉장히 간단합니다. 가장 쉬운 방식은 타입 표명(`type assertion`)을 사용하는 것입니다.

```ts
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

하지만, 만약에 당신이 해당 객체가 특별한 곳에 사용되는 추가적인 속성들이 있다고 확신한다면 문자열 인덱스 시그니처를 사용하는 것이 더 좋은 접근입니다. 만약 `squareConfig`가 `color`와 `width`도 있지만 다른 속성들 또한 가질 수 있다면 이렇게 정의할 수 있습니다.

```ts
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: unknown;
}
```
여기 우리는 `SquareConfg`가 다른 속성들을 가질 수 있도록 설정하고 이 속성들이 `color`나 `width`가 아니라면 타입은 중요하지 않다고 명시하고 있습니다.

이 검사를 우회하는 마지막 방법은 객체를 다른 변수에 할당하는 것입니다. `squareOptions`가 초과 속성 검사를 받지 않는다면, 컴파일러는 오류를 발생시키지 않을 것입니다.

```ts
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

위의 우회 방법은 `squareOptions`와 `squareConfig`간에 공통 속성이 있을 경우에만 동작할 것입니다. 이 예제에서 공통 속성은 `width`였습니다. 하지만 변수에 공통 객체 속성이 존재하지 않는다면 에러가 발생할 것입니다.

```ts
let squareOptions = { colour: "red" };
let mySquare = createSquare(squareOptions);
//Type '{ colour: string; }' has no properties in common with type 'SquareConfig'.
```

위와 같은 단순한 코드에서 이러한 검사를 우회하려고하는 것은 좋지 않습니다. 하지만 메서드가 있고 상태를 유지하는 복잡한 객체 리터럴을 다룰 때 이러한 기술을 염두해야 할 수도 있습니다. 그러나 대부분의 초과 속성 오류는 실제로 버그인 경우가 많습니다.

즉, option bag 와 같은 상황에서 초과 속성 검사 문제가 발생한다면 타입 선언을 수정해야할 수도 있습니다. 예를 들어 `createSquare`에 `color`나 `colour`속성이 전달되어도 상관없다면 `SquareConfig`에 정의를 수정하여 이를 반영하여야 합니다.
