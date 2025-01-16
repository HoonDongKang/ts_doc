### The `typeof` type operator

자바스크립트는 이미 표현문에서 사용가능한 `typeof` 연산자가 있습니다.

```ts
// Prints "string"
console.log(typeof "Hello world");
```

타입스크립트는 `typeof` 연산자를 타입 문맥에서 변수나 프로퍼티의 타입을 참조할 수 있도록 추가하였습니다.

```ts
let s = "hello";
let n: typeof s;
   //let n: string
```

기본적인 타입에 대해서는 딱히 유용하진 않지만, 다른 타입 연산자들과 사용되면 `typeof`를 다양한 패턴을 표현하는데 편리하게 사용됩니다.

예를 들어, `ReturnType<T>`라는 타입라는 미리 정의된 타입을 살펴보겠습니다. 이 타입은 함수 타입을 입력받아 해당 함수의 반환 타입을 생성합니다.

```ts
type Predicate = (x: unknown) => boolean;
type K = ReturnType<Predicate>;
    //type K = boolean
```

`ReturnType`을 함수 이름에 직접 사용하려고 하면, 오류 메시지를 볼 수 있습니다.

```ts
function f() {
  return { x: 10, y: 3 };
}
type P = ReturnType<f>;
//'f' refers to a value, but is being used as a type here. Did you mean 'typeof f'?
```

값과 타입은 서로 다르다는 것을 기억하세요. 값 `f`의 타입을 참조하기 위해서는 `typeof`가 사용됩니다.


```ts
function f() {
  return { x: 10, y: 3 };
}
type P = ReturnType<typeof f>;
    //type P = {x: number; y: number;}
```

#### Limitations
타입스크립트는 `typeof`를 사용할 수 있는 표현식을 의도적으로 제한합니다.

구체적으로, `typeof`는 식별자(변수명) 또는 그 속성에만 사용할 수 있습니다. 이는 실행될 수 있다고 믿는 코드에서 에러가 발생하는 혼란을 회피하는데 도움이 될 것입니다.
```ts
// Meant to use = ReturnType<typeof msgbox>
let shouldContinue: typeof msgbox("Are you sure you want to continue?");
//',' expected.

```
