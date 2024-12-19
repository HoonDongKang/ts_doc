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


