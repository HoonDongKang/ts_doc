소프트웨어 엔지니어링에서 주요한 부분 중 하나는 잘 정의되고 일관된 API를 제공하는 것 뿐만 아니라 재사용성이 높은 컴포넌트를 구축하는 것입니다. 오늘 혹은 미래의 데이터들도 수용할 수 있는 컴포넌트를 만드는 것은 대규모 소프트웨어 시스템을 구축하는데 가장 유연한 능력을 제공합니다.

`C#`이나 `Java`같은 언어에서 재사용 가능한 컴포턴트를 만드는 주요 도구 중 하나는 제네릭(`generics`)입니다. 즉, 단일 타입이 아닌 다양한 타입들과 함께 동작할 수 있는 컴포넌트를 생성할 수 있습니다. 이를 통해 사용자들은 자신들만의 타입을 사용하여 컴포넌트를 활용할 수 있습니다.

### Hello World of Generics

제네릭의 "hello world"라고 할 수 있는 `identity function`을 구현해봅시다. identity function 은 전달된 값을 그대로 반환하는 함수입니다. 이는 `echo` 명령어와 비슷하다고 생각할 수 있습니다.

제네릭 없이 indentity function을 구현하려면 특정 타입을 전달해야 합니다.

```ts
function identity(arg: number): number {
  return arg;
}
```

혹은 `any`타입을 이용하여 identity function을 구현할 수 있습니다.

```ts
function identity(arg: any): any {
  return arg;
}
```

`any`를 사용하는 것은 함수가 `arg` 타입으로 모든 유형을 받을 수 있게 해주어 제네릭이라고 볼 수 있지만 우리는 사실 함수가 반환될 때의 타입에 대한 정보를 잃게 됩니다. 만약 우리가 `number`를 전달하여도 반환되는 값의 타입은 `any`인 것이죠.

대신, 인자의 타입을 포착하고 반환값의 타입을 표시하는데 사용될 수 있는 방법을 필요로 합니다. 이를 위해 타입 변수(`type variable`)를 사용할 것입니다. 타입 변수는 값이 아닌 타입에 작용하는 특별한 변수입니다.

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
```

이제 우리는 `Type`이라는 타입 변수를 identity function에 추가하였습니다. 이 `Type`은 사용자가 제공하는 타입(`number` ,,, )를 포착하여 이 후에 이 정보를 사용할 수 있게 해줍니다. 
