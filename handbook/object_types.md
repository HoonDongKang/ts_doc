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


### Extending Types


미국에서 편지나 소포를 보내는데 필요한 필드들을 설명하는 `BasicAddress`처럼 더 구체적인 버전의 타입을 사용할 때도 있습니다.

```ts
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

이 정도로 충분할 수도 있지만 주소에는 유닛 번호가 포함되어야 할 수도 있습니다. `AddressWithUnit`은 이렇게 생성될 수 있습니다.

```ts
interface AddressWithUnit {
  name?: string;
  unit: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

이렇게 해도 동작은 잘하지만 변경 사항이 단순히 추가된 것임에도 `BasicAdress`의 모든 필드를 반복해야한다는 단점이 있습니다. 대신 우리는 `BasicAddress` 유형을 확장하고 `AddressWithUnit`에 고유한 새 필드만 추가할 수 있습니다.

`interface`에 `extends`를 사용하면 기존 타입의 속성들을 효과적으로 복사하여 새로운 속성들을 추가할 수 있습니다. 이것은 타입 선언의 반복 작업을 줄이는데 효율적일 뿐만 아니라 동일한 속성의 여러 선언들이 연관되어 있음을 나타내는 데에도 도움을 줍니다.

예를 들어 `AddressWithUnit`은 `street`이 `BasicAddress`에서 유래하여 반복할 필요가 없으며 사용자는 해당 두 타입들이 연관되어 있음을 알 수 있습니다.

`interface`은 다양하게 확장될 수 있습니다.

```ts
interface Colorful {
  color: string;
}
 
interface Circle {
  radius: number;
}
 
interface ColorfulCircle extends Colorful, Circle {}
 
const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};
```

### Intersection Types

`interface`는 다른 타입들 위에 새로운 타입들을 추가함으로써 확장시킬 수 있습니다. 타입스크립트는 교차 타입(`intersection`)을 제공하며, 기존 객체를 결합하는데 주로 사용됩니다.

교차 타입은 `&` 연산자를 사용합니다.

```ts
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
 
type ColorfulCircle = Colorful & Circle;
```

여기서 `Colorful`과 `Circle`을 교차하여 두 타입의 모든 속성들을 갖는 새로운 타입을 생성하였습니다.

```ts
function draw(circle: Colorful & Circle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}
 
// okay
draw({ color: "blue", radius: 42 });
 
// oops
draw({ color: "red", raidus: 42 });
//oject literal may only specify known properties, but 'raidus' does not exist in type 'Colorful & Circle'. Did you mean to write 'radius'?
```


### Interface Extension vs. Intersection

우리는 방금 두 가지 타입 결합 방식을 살펴보았는데, 이들은 비슷하지만 실제로 미묘하게 다릅니다. `interface`에서는 `extends`절을 사용하여 다른 타입을 확장시킬 수 있었고 교차타입에서는 이를 유사하게 구현하여 결과를 타입 별칠으오 이름 지을 수 있었습니다.

두 방식의 주요 차이점은 충돌을 처리하는 방식이며, 이 차이점은 인터페이스와 교차 타입 중 하나를 선택해야할 때 고려할 대표적인 이유입니다.

만약 인터페이스가 동일한 이름으로 정의되었다면 타입스크립트는 해당 속성이 호환이 가능한 경우 병합하려고 시도합니다. 하지만 호환되지 않는다면(동일한 이름을 갖지만 다른 타입을 갖는 경우) 에러를 발생시킵니다.

교차 타입의 경우 서로 다른 타입을 가진 속성은 자동으로 병합됩니다. 타입이 나중에 사용될 때, 타입스크립트는 동시에 두 타입을 모두 만족해야한다고 간주하기 때문에 예상치 못한 결과를 초래할 수 있습니다.

예를 들어, 아래 코드는 속성들이 호환될 수 없기 때문에 에러가 발생합니다.
```ts
interface Person {
  name: string;
}
interface Person {
  name: number;
}
```
반대로 아래 코드는 컴파일은 되지만 `never`타입을 갖게 됩니다.
```ts
interface Person1 {
  name: string;
}
 
interface Person2 {
  name: number;
}
 
type Staff = Person1 & Person2
 
declare const staffer: Staff;
staffer.name;
         //(property) name: never
```

이런 상황에서 `Staff`는 `name`속성으로 `string`과 `number`를 모두 요구하기에 `never`타입이 됩니다.

### Generic Object Types
`Box`타입이 `string`이나 `number` 혹은 `Giraffe`와 같은 아무런 값을 포함한다고 상상해봅시다.

```ts
interface Box {
  contents: any;
}
```

`contents` 속성의 타입은 `any`로, 동작은 하겠지만 사고로 이어질 수 있습니다.

`unknown`을 사용할 수도 있지만 이미 `contents`의 타입을 알고 있는 경우에도 예방적인 체크가 필요하거나 오류를 유발할 수 있는 타입 단언(`type assertion`)을 사용해야 합니다.

```ts
interface Box {
  contents: unknown;
}
 
let x: Box = {
  contents: "hello world",
};
 
// we could check 'x.contents'
if (typeof x.contents === "string") {
  console.log(x.contents.toLowerCase());
}
 
// or we could use a type assertion
console.log((x.contents as string).toLowerCase());
```

한 가지 안전한 방법은 각각의 `contents` 타입에 대해 서로 다른 `Box` 타입을 설계하는 것입니다.

```ts
interface NumberBox {
  contents: number;
}
 
interface StringBox {
  contents: string;
}
 
interface BooleanBox {
  contents: boolean;
}
```

하지만 이것은 우리가 해당 함수를 사용하기 위해 서로 다른 함수들을 생성하고 오버로드 해야한다는 것입니다.

```ts
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
  box.contents = newContents;
}
```

복사본이 너무 많습니다. 더욱이 우리는 새로운 타입과 오버로드들을 더 생성해야할 수도 있습니다. 상자의 타입들과 오버로드들은 사실상 모두 동일하기 때문에 절망적입니다.

대신에 우리는 제네릭 `Box`타입을 이용하여 타입 파라미터를 선언할 수 있습니다.

```ts
interface Box<Type> {
  contents: Type;
}
```

`Type`의 `Box`는 `contents`의 `Type`을 갖는 것이라고 읽을 수 있습니다. 나중에 `box`를 참조할 때, 우리는 `type` 대신 타입 인수를 제공해야 합니다.

```ts
let box: Box<string>;
```

`Box`를 실제 타입에 대한 템플릿으로 생각하면 됩니다. `Type`은 다른 타입으로 대체될 대체자입니다. 타입스크립트가 `Box<string>`을 발견하면 `Box<Type>`의 `Type`을 `String`으로 모두 대체하고 `{ contents: string }` 처럼 작동하게 됩니다. 다시 말해, `Box<string>`은 `StringBox`와 동일하게 동작합니다.

```ts
interface Box<Type> {
  contents: Type;
}
interface StringBox {
  contents: string;
}
 
let boxA: Box<string> = { contents: "hello" };
boxA.contents;
        
//(property) Box<string>.contents: string
 
let boxB: StringBox = { contents: "world" };
boxB.contents;
        
//(property) StringBox.contents: string
```

`Type`은 어떤 타입으로든 대체될 수 있기 때문에 `Box`은 재사용이 가능합니다. 이 말은 새로운 타입의 박스가 필요하면 새로운 `Box`를 선언할 필요가 없다는 것을 의미합니다.

```ts
interface Box<Type> {
  contents: Type;
}
 
interface Apple {
  // ....
}
 
// Same as '{ contents: Apple }'.
type AppleBox = Box<Apple>;
```

이것은 또한 generic functions을 사용하여 오버로드를 회피할 수 있다는 의미입니다.
```ts
function setContents<Type>(box: Box<Type>, newContents: Type) {
  box.contents = newContents;
}
```

타입 별칭(`type aliases`)는 제네릭일 수 있다는 점을 주목할 필요가 있습니다. 우리는 새로운 `Box<Type>`인터페이스를 선언할 수 있습니다.

```ts
interface Box<Type> {
  contents: Type;
}
```

타입 별칭을 사용하기 대신에요
```ts
type Box<Type> = {
  contents: Type;
};
```

타입 별칭은 인터페이스와 달리 객체 타입 뿐만 아니라 다른 종류의 타입도 설명할 수 있기 때문에, 이를 사용하여 다양한 종류의 제네릭 헬퍼 타입들을 작성할 수 있습니다.


```ts
type OrNull<Type> = Type | null;
 
type OneOrMany<Type> = Type | Type[];
 
type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;
           
//type OneOrManyOrNull<Type> = OneOrMany<Type> | null
 
type OneOrManyOrNullStrings = OneOrManyOrNull<string>;
               
//type OneOrManyOrNullStrings = OneOrMany<string> | null
```

#### The `Array` Type

제네릭 객체 타입은 보통 특정 요소의 타입과 독립으로 작동하는 일종의 컨테이너 타입입니다.데이터 구조가 이렇게 동작하는 것은 이상적이며 이를 통해 다양한 데이터 타입을 재사용할 수 있습니다.

이 핸드북에서 우리가 계속 사용해 온 타입 중 하나가 바로 이런 타입인데, 그것은 바로 배열 타입입니다. 우리가 사용하는 `number[]`나 `string[]`은 사실상 `Array<number>`나 `Array<string>`의 축약형입니다.

```ts
function doSomething(value: Array<string>) {
  // ...
}
 
let myArray: string[] = ["hello", "world"];
 
// either of these work!
doSomething(myArray);
doSomething(new Array("hello", "world"));
```

`Box`타입처럼 `Array` 그 자체로도 제네릭 타입입니다.
```ts
interface Array<Type> {
//Global type 'Array' must have 1 type parameter(s).
All declarations of 'Array' must have identical type parameters.
  /**
   * Gets or sets the length of the array.
   */
  length: number;
 
  /**
   * Removes the last element from an array and returns it.
   */
  pop(): Type | undefined;
 
  /**
   * Appends new elements to an array, and returns the new length of the array.
   */
  push(...items: Type[]): number;
//A rest parameter must be of an array type.
 
  // ...
}
```

모던 자바스크립트 또한 `Map<K, V>`나 `Set<T>`, `Promise<T>`같은 제네릭의 데이터 구조를 제공합니다. 즉, `Map`, `Set`, `Promise` 작동방식 덕에 어떠한 타입과도 함께 사용할 수 있다는 것입니다.

#### The `ReadonlyArray` type

`ReadonlyArray` 은 변하지 않는 배열을 설명하는 특별한 타입입니다.

```ts
function doStuff(values: ReadonlyArray<string>) {
  // We can read from 'values'...
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);
 
  // ...but we can't mutate 'values'.
  values.push("hello!");
//Property 'push' does not exist on type 'readonly string[]'.
}
```

`readonly` 수정자와 마찬가지로 이것은 주로 의도를 전달하기 위한 도구입니다. 만약 함수가 `ReadonlyArray`를 반환한다면 그 내용을 전혀 변경해서는 안되는다는 것을 알려줍니다. 반대로 함수가 `ReadonlyArray`를 매개변수로 받는다면, 해당 함수에 어떠한 함수를 전달해도 배열이 변경되지 않는다는 것을 알 수 있습니다.

`Array`와 달리 `ReadonlyArray` 생성자는 존재하지 않습니다.

```ts
new ReadonlyArray("red", "green", "blue");
//readonlyArray' only refers to a type, but is being used as a value here.
```

대신 `Array`에 `ReadonlyArray`에 할당이 가능합니다.

```ts
const roArray: ReadonlyArray<string> = ["red", "green", "blue"];
```

타입스크립트가 `Array<Type>`의 축약형 `Type[]`이 있는 것처럼 `ReadonlyArray>Type>`은 `readonly Type[]`으로 사용할 수 있습니다.

```ts
let x: readonly string[] = [];
let y: string[] = [];
 
x = y;
y = x;
//The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'.
```

#### Tuple Types
튜플 타입은 또 다른 형태의 배열 타입으로, 몇 개의 요소를 포함하는지와 특정 위치에 어떤 타입의 요소가 있는지를 정확히 알고 있는 타입입니다.

```ts
type StringNumberPair = [string, number];
```

`StringNumberPair`은 `string`과 `number`의 튜플 타입이다. `ReadonlyArray`처럼 런타임에 별도의 영향을 끼치진 않지만 타입스크립트에서는 중요한 역활을 합니다. 타입 시스템 관점에서 `StringNumberPair`는 0번째 인덱스에 문자열, 1번째 인덱스에 숫자가 들어 있는 배열을 의미합니다.

```ts
function doSomething(pair: [string, number]) {
  const a = pair[0];
       //const a: string
  const b = pair[1];
       //const b: number
  // ...
}
 
doSomething(["hello", 42]);
```

만약 인덱스를 초과한 값에 접근하면 에러가 발생됩니다.

```ts
function doSomething(pair: [string, number]) {
  // ...
 
  const c = pair[2];
//Tuple type '[string, number]' of length '2' has no element at index '2'.
}
```

자바스크립트의 구조 분해 할당처럼 튜플도 구조분해가 가능합니다.

```ts
function doSomething(stringHash: [string, number]) {
  const [inputString, hash] = stringHash;
 
  console.log(inputString);
                  //const inputString: string
 
  console.log(hash);
               //const hash: number
}
```
> 튜플 타입은 각 요소의 의미가 명확한 강한 규칙 기반의 API에서 유용합니다. 이를 통해 구조 분해를 할 때 변수 이름을 원하는대로 지정할 수 있는 유연성을 제공합니다. 위 예시 처럼 0번과 1번 요소에 원하는 이름을 자유롭게 붙일 수 있습니다.

> 그러나 모든 사용자가 "명확"하다고 여기는 기준이 같지 않을 수 있으므로 설명적인 프로퍼티 이름을 가진 객체를 사용하는 것이 API 설계에 더 적합한지 고민해볼 필요가 있습니다.
