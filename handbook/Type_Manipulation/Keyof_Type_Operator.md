### The `keyof` type operator

`keyof` 연산자는 `object`타입을 갖고 키 값으로 문자열 또는 숫자 리터럴의 유니언 타입을 갖습니다.
아래 `P` 타입은 `type P = "x" | "y"` 와 같습니다.

```ts
type Point = { x: number; y: number };
type P = keyof Point;
    //type P = keyof Point
```

만약 타입이 `string` 혹은 `number` 인덱스 시그니처를 갖는다면 `keyof`는 이러한 타입을 반환합니다.

```ts
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;
    //type A = number
 
type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
    //type M = string | number
```

자바스크립트 객체의 키들은 항상 문자열로 강제 반환되기에 `obj[0]`은 `obj["0"]`과 동일합니다. 그렇기에 해당 예제에서 `M`은 `string | number`라는 것을 명심하세요.

`keyof` 타입은 **매핑된 타입(mapped types)**과 결합될 때 특히 유용해지며, 이에 대해서는 이후에 더 자세히 배우게 될 것입니다.
