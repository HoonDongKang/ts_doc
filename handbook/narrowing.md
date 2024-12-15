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
