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
..