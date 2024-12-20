```typescript
function padLeft(padding: number | string, input: string): string {
  throw new Error("Not implemented yet!");
}
```

만약 `padding`이 숫자형 타입이라면, `input`에 추가할 공백의 개수로 취급할 것이다. 반면에 `padding`이 문자열일 경우, 해당 문자열을 `input` 값 앞에 그대로 추가할 것이다.

 `padding`이 숫자일 경우에 대해서 로직을 설계해보자.

 ```typescript
function padLeft(padding: number | string, input: string): string {
  return " ".repeat(padding) + input;
Argument of type 'string | number' is not assignable to parameter of type 'number'.
  Type 'string' is not assignable to type 'number'.
}
```

`padding`에 대해서 에러가 발생하였다. 타입스크립트는 `number | string` 타입에 `number` 타입만 갖고 있는 `repeat` 함수를 호출하는 것에 대해 오류를 발생시켰다.

다시 말해, 우리는 `padding`이 숫자인지 먼저 명시적으로 확인하지 않았으며, 문자열일 경우를 처리하지 않았습니다.

```typescript
function padLeft(padding: number | string, input: string): string {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

타입스크립트의 타입 시스템은 **타입 안전성을 확보** 하면서 최대한 일반적인 자바스크립트 코드를 쉽게 만들어주는 것을 목표로 한다.

그렇게 보이지는 않겠지만, 사실 이 코드 아래에서는 수많은 일들이 발생하고 있습니다. 타입스크립트가 정적 타입을 이용하여 런타임 값들을 분석하는 것처럼, `if/else`, 삼항 연산자, 반복문, 참/거짓 검사 등과 같은 자바스크립트의 런타임 제어 흐름 구조의 타입 분석과 겹쳐보입니다.

`if` 검사 안에서 타입스크립트는 `typeof padding === "number"` 를 보고 `type guard`라고 불리는 특별한 형태의 코드를 이해합니다. 타입스크립트는 주어진 상황에서 값이 가질 수 있는 가장 정확한 타입을 분석하는 방법을 선택합니다.

**특별한 검사(type guards)와 할당** 그리고 **더 구체적인 타입으로 선언**하는 과정을 **`narrowing`**이라고 합니다.

### `typeof` type guards

자바스크립트는 `typeof` 연산자를 제공하여 런타임에서 값들의 타입에 대한 가장 기본적인 정보를 줍니다. 타입스크립트는 이 연산자가 아래의 집합을 리턴해줄 것이라 예상합니다.

- `string` / `number` / `bigint` / `boolean` / `symbol` / `undefined` / `object` / `function`

`padLeft`에서 보았듯, 이 연산자는 자바스크립트 라이브러리에서 자주 나오며 타입스크립트는 여러 갈래에서 좁혀진 타입들로 인식을 합니다.

타입스크립트에서 `typeof`로 값을 검사하는 것을 type guard라고 합니다.

```typescript
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
//'strs' is possibly 'null'.
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```

`printAll` 함수에서 우리는 `strs`가 배열일 경우에 `object`일 것이라 검사합니다. 하지만 자바스크립트에서 `typeof null` 또한 `object`입니다.

타입스크립트는 우리가 `strs`이 `string[]`이 아닌 `string[] | null` 이라고 좁혀서 알려주었을 뿐입니다.

### Truthiness narrowing

Truthiness는 사전에 있는 단어는 아니지만 자바스크립트에서 많이 들었을 겁니다.

JavaScript에서는 조건문, `&&`, `||`, if 문, 불리언 부정(!) 등에서 어떤 표현식이든 사용할 수 있습니다. 예를 들어, if 문은 조건이 항상 boolean 타입일 것으로 기대하지 않습니다.

```typescript
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```

자바스크립트에서 `if`와 같은 구조에서 조건을 이해하기 위해 먼저 조건값을 `boolean` 형태로 강제 변환시킨 후에 `true` / `false`에 따라 값은 처리합니다.

- `0` / `NaN` / `""` / `0n` / `null` / `undefined` 

값들은 모두 `false`로 처리되며 이 외에는 `true`로 변환됩니다. 당신은 `Boolean` 함수 혹은 이중 부정 연산자(`!!`) 일 이용해서 boolean 값으로 변환시킬 수 있습니다.

```typescript
// both of these result in 'true'
Boolean("hello"); // type: boolean, value: true
!!"world"; // type: true,    value: true
//This kind of expression is always truthy.
```

`null`이나 `undefined`와 같은 값들로부터 guarding 하기 위해 사용되는 방식이기도 합니다.

```typescript
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
```

`strs`가 참인 지 검사하는 방식으로 에러를 제거할 수 있습니다.. 최소한 `TypeError: null is not iterable` 와 같은 오류를 예방시켜줄 수 있습니다.


원시값에 참/거짓을 검사하는 것은 가끔 오류가 발생할 수도 있습니다.

```typescript
function printAll(strs: string | string[] | null) {
  // !!!!!!!!!!!!!!!!
  //  DON'T DO THIS!
  //   KEEP READING
  // !!!!!!!!!!!!!!!!
  if (strs) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

해당 함수 전체를 truthy 검사로 감싸는 방식은 미묘한 단점을 가질 수 있습니다. (빈 문자열 `""` 을 올바르게 처리하지 못한다는 점이죠)

타입스크립트는 초기 단계에서 버그를 잡는데 도움이 될 수 있지만, 개발자가 해당 값에 대해서 아무런 조치를 취하지 않는다면 할 수 있는 일이 제한적입니다.
(빈 문자열`""` 에 대해서 개발자가 아무런 조치를 취하지 않음 -> TS에서 버그를 잡지 않음)

참/거짓 검사에서 narrowing을 하는 한 가지 추가적인 방법은`!` boolean 부정을 통해 검사하는 방식이다.

```typescript
function multiplyAll(
  values: number[] | undefined,
  factor: number
): number[] | undefined {
  if (!values) {
    return values;
  } else {
    return values.map((x) => x * factor);
  }
}
```


### Equality narrowing

```typescript
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // We can now call any 'string' method on 'x' or 'y'.
    x.toUpperCase();
          
(method) String.toUpperCase(): string
    y.toLowerCase();
          
(method) String.toLowerCase(): string
  } else {
    console.log(x);
               
(parameter) x: string | number
    console.log(y);
               
(parameter) y: string | boolean
  }
}
```

해당 예제에서 `x`와 `y`값은 동일하다고 검사하고 있지만 타입스크립트는 해당 타입들 또한 같아야 한다고 인지하고 있습니다. `string`은 `x`와 `y`가 갖는 공통 타입이고 첫번째 분기에서 두 값은 동일하다고 인지합니다.

변수가 아닌 특정 리터럴 값 또한 비교할 수 있습니다. `truthiness narrowing` 부분에서 `printAll`함수는 빈 문자열을 제대로 ㅓ처리하지 못하여 에러가 발생하였습니다. 대신에 우리는 `null`타입을 막는 검사를 추가하면 `strs`에서 `null`을 완전히 제거할 수 있습니다.

```typescript
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

자바스크립트에서 `==`와 `!=`와 같은 느슨한 비교 또한 정확하게 좁혀집니다. `== null`은 `null` 뿐만 아니라 `undefined` 또한 포함되며 그 반대도 성립됩니다.


```typescript
interface Container {
  value: number | null | undefined;
}
 
function multiplyValue(container: Container, factor: number) {
  // Remove both 'null' and 'undefined' from the type.
  if (container.value != null) {
    console.log(container.value);
    // Now we can safely multiply 'container.value'.
    container.value *= factor;
  }
}
```

### The `in` operator narrowing

자바스크립트에는 객체이거나 그 프로토타입 체인에 특정 이름의 속성을 갖는지 확인하는 `in` 연산자가 있습니다. 타입스크립트는 이를 이용하여 타입을 좁혀가는 방식을 사용합니다.

예를 들어, `value in x`에서 `value`는 문자열 리터럴이고 `x`는 유니온 타입입니다. true인 경우에 `x`는 필수 혹은 선택적(optional)으로 `value` 속성을 갖으며 false일 때는 선택적 혹은 `value`를 갖고 있지 않습니다.

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };
 
function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }
 
  return animal.fly();
}
```

다시 말해, 선택적 속성은 narrowing에서 양쪽 모두에서 등장할 수 있습니다. 예로 사람은 모두 수영을 하거나 날 수 있으며 그렇기에 `in` 검사에서 양쪽에 등장해야합니다.

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };
 
function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) {
    animal;
 else {
    animal;

}
```

### `instanceof` narrowing

자바스크립트는 하나의 값이 다른 값의 "인스턴스"인지 확인하는 연산자가 있습니다. 더 자세히는 `x instanceof Foo`는 `x`의 프로토타입 체인에 `Foo.prototype`이 있는지 검사하는 것입니다. 더 깊게 가진 않지만 클래스를 살펴볼 때 많이 접할 수 있을 것입니다. 이 연산자는 `new`와 함께할 때 더 실용적이거든요. `instanceof`는 type guard이며 타입스크립트는 `instaceof`를 통해 여러 분기로 좁혀 검사합니다.


```typescript
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString());
  } else {
    console.log(x.toUpperCase());
  }
}
```

### Assignments

아무런 변수에 할당을 할 때, 타입스크립트는 할당문의 오른쪽 값을 확인하고 왼쪽 변수의 타입을 좁힙니다.

```typescript
let x = Math.random() < 0.5 ? 10 : "hello world!"; //string | number
x = 1; // number
x = "goodbye!"; // string
```

각각의 할당이 유효하다는 점을 주목하세요. 첫 할당 이후에 `x`는 `number`로 변경되었음에도 여전히 문자열을 할당할 수 있습니다. 이것은 첫 할당 시에 `string | number`의 타입을 갖고 있었기 때문에 선언된 타입을 통해 할당 가능성(assignability) 을 검사하게 됩니다.

만약 `x`에 `boolean`을 할당하려고 했으면 선언된 타입에 포함되지 않았기 때문에 오류가 발생했었을 것입니다.

### Control flow analysis

지금까지는 타입스크립트에서 특정 분기로 좁혀가는 방식을 예로 들며 봐왔습니다. 하지만 단순히 모든 변수를 따라 올라가며 `if`, `while`, 조건문 등에서 타입 가드를 찾는 것 이상으로 이루어집니다. 예를 들어,
```typescript
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

`padLeft`는 첫 `if`문에서 반환되기 때문에 타입스크립트는 나머지 본문에 `padding`이 `number`인 경우에 도달할 수 없는 것을 파악합니다. 그 결과, 나머지 함수 내에 `padding`의 타입에서 `number`를 제거(`sting | number` 타입에서 좁히기)할 수 있었습니다.

이와 같은 코드의 **도달 가능성에 따른 분석**을 **제어 흐름 분석(control flow analysis)** 라고 하며 타입스크립트는 이 흐름 분석을 타입 가드와 할당을 만날 때마다 활용하여 타입을 좁혀갑니다. 변수가 분석될 때, 제어 흐름은 여러 번 분기되고 병합될 수 있으며 각 지점마다 다른 타입으로 관찰될 수 있습니다.

### Using type predicates

지금까지는 현존하는 자바스크립트 구성에서 narrowing을 다루는 방식을 살펴보았지만 타입 변화에 대해 더 세밀한 조작을 원할 수도 있을 것입니다.

사용자 정의 타입 가드를 정의하려면, **타입 술어(type predicate)** 를 반환하는 함수만 정의하면 됩니다.

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

`pet is Fish`는 타입 술어에 해당합니다. 술어는 `parameterName is Type`과 같은 형태를 띄며 `parameterName`은 현 함수 시그니처에서 매개변수의 이름이어야 한다.

`isFish`가 다른 변수와 함께 사용될 때, 타입스크립트는 만약 기존 타입이 호환 가능하다면 특정 타입으로 좁혀집니다.

```typescript
let pet = getSmallPet();
 
if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```
