우리는 `indexed accesss` 타입을 사용하여 다른 타입에서 특정 속성을 조회할 수 있습니다.

```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"];
     //type Age = number
```

인덱싱 타입 그 자체로 타입이기에, 우리는 유니온, `keyof`, 혹은 다른 타입을 전체적으로 사용할 수 있습니다.

```ts
type I1 = Person["age" | "name"];
     //type I1 = string | number
 
type I2 = Person[keyof Person];
     //type I2 = string | number | boolean
 
type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName];
     //type I3 = string | boolean
```

만약 존재하지 않는 속성을 인덱싱할 경우, 에러가 발생합니다.

```ts
type I1 = Person["alve"];
//Property 'alve' does not exist on type 'Person'.
```

임의의 타입으로 인덱싱하는 또 다른 예는 `number`를 사용하여 배열 요소들의 타입을 가져올 때입니다. 우리는 `typeof`와 함께 사용하여 객체 리터럴의 요소 타입들을 편리하게 잡아낼 수 있습니다.

```ts

const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];
 
type Person = typeof MyArray[number];
       //type Person = { name: string; age: number; }
type Age = typeof MyArray[number]["age"];
     //type Age = number
// Or
type Age2 = Person["age"];
      //type Age2 = number
```

타입은 인덱싱할 때만 사용할 수 있으며, `const`를 사용하여 변수를 참조할 수 없습니다.
```ts
const key = "age";
type Age = Person[key];
```

하지만 타입 별칭(type alias)를 사용하여 비슷하게 리팩토링이 가능합니다.

```ts
type key = "age";
type Age = Person[key];
```
