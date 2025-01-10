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

이제 우리는 `Type`이라는 타입 변수를 identity function에 추가하였습니다. 이 `Type`은 사용자가 제공하는 타입(`number` ,,, )를 포착하여 이 후에 이 정보를 사용할 수 있게 해줍니다. 여기 우리는 반환 타입으로 다시 `Type`을 사용합니다. 이제 동일한 타입이 인수와 반환 타입에 사용되는 것을 알 수 있고 이는 함수의 한 쪽에서 타입 정보를 전달받아 다른 한 쪽으로 내보내는 것을 가능하게 해줍니다.

우리는 이러한 `identity` 함수가 여러 타입에 대해 동작할 수 있기에 제네릭이라고 말합니다. `any`를 사용하는 것과 달리, 첫 번재 `identity` 함수처럼 아무런 정보를 잃지 않고 인수와 반환 타입으로 숫자 타입을 정확하게 표시되는 것을 볼 수 있습니다.

제네릭 아이덴티티 함수를 작성한 후, 우리는 두 가지 방법으로 호출할 수 있습니다. 첫 번째 방법은 타입 인수를 포함하여 모든 인수를 전달하는 것입니다.

```ts
let output = identity<string>("myString");
      //let output: string
```

명시적으로 함수 호출의 인수 중 하나로 `Type`이 `string`이라는 것을 설정해주었으며 `()` 대신 `<>`로 인수를 둘러싸는 방식으로 나타냅니다.

두 번재 방법은 가장 일반적인 방법일 것입니다. 타입 인수 추론(type argument inference)를 사용하는 것입니다. 즉, 우리가 전달하는 인수의 타입을 컴파일러가 자동으로 추론하여 `Type`의 값을 설정하는 것입니다.

```ts
let output = identity("myString");
      //let output: string
```

우리가 각괄호(`<>`) 안에 타입을 명시적으로 전달할 필요가 없다는 점에 주목하세요. 컴파일러는 단순히 값 `myString`을 확인하고 `Type`을 해당 값의 타입으로 설정하였습니다. 타입 인수 추론은 코드를 더 짧고 가독성 높게 도와주지만 더 복잡한 예제에서 컴파일러가 추론을 실패할 경우에는 명시적으로 타입을 전달해야할 수도 있습니다.

### Working with Generic Type Vaiables

만약 제네릭을 사용하려고 한다면, 당신이 `identity`와 같은 제네릭 함수를 만들 때 컴파일러는 함수의 본문에 제네릭 타입 매개변수를 올바르게 사용하도록 강제한다는 것을 알게될 것입니다. 즉 이러한 매개변수를 실제로 어떤 타입이든 될 수 있는 것처럼 다뤄야 합니다.

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
```

만약 콘솔에 `arg`인수의 길이를 매 호출마다 출력시키고 싶으면 어떻게 해야 할까요? 아마 이렇게 작성할 것 같습니다.

```ts
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
//Property 'length' does not exist on type 'Type'.
  return arg;
}
```

이렇게 작성하면 컴파일러는 `arg`에 `.length`를 갖고 있다는 것을 명시하지 않은 체 사용하였다고 에러를 발생시킵니다. 앞서 언급하였듯이 우리는 타입변수는 모든 타입을 대체할 수 있기 때문에 누군가 `.length`가 없는 `number`를 전달할 수도 있습니다.

이제 이 함수가 `Type` 자체가 아니라 `Type` 배열에 대해 작동하도록 의도했다고 가정해봅시다. 배열을 다루고 있기 때문에 `.length`는 사용할 수 있습니다. 

```ts
function loggingIdentity<Type>(arg: Type[]): Type[] {
  console.log(arg.length);
  return arg;
}
```

`logginIdentity`의 타입을 "제네릭 함수 `loggingIdentity`는 타입 매개변수 `Type`을 받고 `Type`의 배열인 인수 `arg`를 받아 `Type`의 배열을 반환한다. 만약 우리가 숫자 배열을 전달하면 반환값으로 숫자 배열을 얻을 것입니다. 이 때 `Type`은 `number`로 바인딩됩니다.

이것은 제네릭 타입 변수 `Type`을 우리가 작업하는 타입의 일부로 사용할 수 있으며 전체 타입이 아니라 더 유연하게 사용할 수 있습니다.

우리는 해당 예제를 이런 식으로도 사용할 수 있습니다.
```ts
function loggingIdentity<Type>(arg: Array<Type>): Array<Type> {
  console.log(arg.length); // Array has a .length, so no more error
  return arg;
}
```

여러 언어에서 이러한 스타일의 타입에 익숙할 수 있습니다. 다음 섹션에서는 `Array<Type>`과 같은 제네릭 타입을 직접 생성하는 방법을 다룰 것입니다.




