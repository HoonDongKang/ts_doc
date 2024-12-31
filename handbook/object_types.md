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

