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


### Generic Types
이전 섹션에서 우리는 여러 타입들과 함께 동작할 수 있는 제네릭 identity functions를 생성하였습니다. 이번 섹션에서는 함수 자체의 타입과 제네릭 인터페이스를 만드는 방법에 대해서 살펴볼 것입니다.

제네릭 함수의 타입은 제네릭이 아닌 함수와 비슷하며 함수 선언문과 마찬가지로 타입 매개변수가 먼저 등장합니다.

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: <Type>(arg: Type) => Type = identity;
```
타입에서 제네릭 타입 매개변수의 이름을 다르게 사용할 수도 있습니다.단 타입 변수의 개수와 타입 변수가 사용되는 방식이 일치해야 합니다.

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: <Input>(arg: Input) => Input = identity;
```

제네릭 타입을 객체 리터럴 타입의 호출 시그니처(call signature)로 작성할 수 있습니다.
```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: { <Type>(arg: Type): Type } = identity;
```

이제 첫 제네릭 인터페이스를 작성해보시다. 객체 리터럴을 가져와 인터페이스로 옮겨 보겠습니다.

```ts
interface GenericIdentityFn {
  <Type>(arg: Type): Type;
}
 
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: GenericIdentityFn = identity;
```

비슷한 예로 우리는 제네릭 매개변수를 전체 인터페이스의 매개변수로 이동하고 싶을 수 있습니다. 이렇게 하면 우리가 어떤 타입에 대해 제네릭을 사용하는 지 명확히 알 수 있습니다.(`Dictionary` 대신 `Dictionary<string>`) 이것은 이는 인터페이스의 모든 요소에서 타입 매개변수를 사용할 수 있게 해줍니다.

```ts
interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}
 
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: GenericIdentityFn<number> = identity;
```

우리의 예제가 조금 달라졌다는 점에 대해 인지해 주세요. 제네릭 함수에 대해 설명하기 보다 우리는 제네릭 타입의 일부로서 비제네릭 함수 시그니처를 갖게 되었습니다.	`GenericIdentityFn`을 사용할 때, 우리는 호출 시그니처가 적용될 타입을 고정하기 위해 일치하는 타입 인수(`number`)를 명시해야 합니다.

타입 매개변수를 **호출 시그니처에 직접 설정**할 지 혹은 **인터페이스 자체에 설정**할 지 이해하는 것은 타입의 어떤 측면이 제네릭인 지 설명하는데 유용할 것입니다.

또한 제네릭 클래스도 생성할 수 있습니다. 다만 제네릭 enum이나 namespaces를 생성하는 것을 불가능합니다.

### Generic Classes
제네릭 클래스는 제네릭 인터페이스와 비슷한 형태를 갖춥니다. 제네릭 클래스는 클래스명 뒤 각괄호(`<>`)에 제네릭 타입 매개변수를 갖습니다.

```ts
class GenericNumber<NumType> {
  zeroValue: NumType;
  add: (x: NumType, y: NumType) => NumType;
}
 
let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
  return x + y;
};
```

이것은 `GenericNumber` 클래스를 매우 직관적으로 사용하는 예입니다. 하지만 이 클래스가 `number`타입만 사용하도록 제한되어 있지 않다는 것을 인지하셨을 수도 있습니다. 우리는 `string` 대신 더 복잡한 객체를 사용할 수도 있습니다.

```ts
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function (x, y) {
  return x + y;
};
 
console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

인터페이스와 같이 클래스 자체에 타입 매개변수를 설정하면 모든 속성이 동일한 타입으로 작동하도록 보장할 수 있습니다.

클래스는 타입의 두 가지 측면(정적 측면과 인스턴스 측면)을 가지고 있습니다 제네릭 클래스는 정적 측면이 아닌 인스턴스 측면에 대해서만 제네릭으로 동작합니다. 따라서 클래스 작업 시 정적 멤보든에 대해서는 클래스 타입 매개변수를 사용할 수 없습니다.

### Generic Constraints

이전 예제에서 기억하듯이, 당신이 어느정도 알고 있는 특정 타입 집합에 대해서 동작하는 제네릭 함수를 작성하고 싶을 수도 있습니다. `loggingIdentity`와 같이 `arg`의 속성`.legth`에 접근하고 싶지만 컴파일러는 모든 타입이 `.length`를 갖고 있는 것을 증명하지 못하기 때문에 추론할 수 없다고 경고하였었습니다.

```ts
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
//Property 'length' does not exist on type 'Type'.
  return arg;
}
```
모든 타입을 다루는 대신, `.length`속성을 가진 타입들에만 작동하도록 제약을 두고 싶습니다. 타입에 최소한으로 이 속성이 있어야만 합니다. 이를 위해 우리가 정의하는 `Type`에 대한 요구사항을 제약조건으로 나열해야 합니다.

그렇게 하기 위해 우리는 제약조건을 갖는 인터페이스를 생성할 것입니다. 여기서 우리는 `.length` 속성 하나만 갖는 인터페이스를 생성하고 `extends`를 통해 제약조건을 명시할 것입니다.

```ts
interface Lengthwise {
  length: number;
}
 
function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // Now we know it has a .length property, so no more error
  return arg;
}
```

제네릭 함수는 제약조건이 설정되어 있기 때문에 모든 타입과 함께 사용될 수는 없습니다.

```ts
loggingIdentity(3);
//Argument of type 'number' is not assignable to parameter of type 'Lengthwise'.
```

대신 요구되는 속성을 충족하는 값에 대해서는 통과됩니다.

```ts
loggingIdentity({ length: 10, value: 3 });
```

### Using Type Parameters in Generic Constraints

타입 매개변수를 다른 타입 매개변수로 제약할 수 있습니다. 예를 들어 객체와 해당 객체의 속성 이름을 입력받아 특정 속성을 가져오고 싶다고 가정해 봅시다. 이때, `obj`에 없는 속성을 잘못 가져오지 않도록 하기 위해 두 타입 간에 제약조건을 설정할 수 있습니다.

```ts
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}
 
let x = { a: 1, b: 2, c: 3, d: 4 };
 
getProperty(x, "a");
getProperty(x, "m");
//Argument of type '"m"' is not assignable to parameter of type '"a" | "b" | "c" | "d"'.
```

### Using Class Types in Generics
타입스크립트에서 제네릭을 이용한 팩토리 패턴을 사용할 때, 클래스 타입을 생성자 함수로 참조하여야 합니다.

```ts
function create<Type>(c: { new (): Type }): Type {
  return new c();
}
```

더 고급 예제에서는 prototype 속성을 사용하여 생성자 함수와 클래스 타입의 인스턴스 측 간의 관계를 추론하고 제한합니다.

```ts
class BeeKeeper {
  hasMask: boolean = true;
}
 
class ZooKeeper {
  nametag: string = "Mikle";
}
 
class Animal {
  numLegs: number = 4;
}
 
class Bee extends Animal {
  numLegs = 6;
  keeper: BeeKeeper = new BeeKeeper();
}
 
class Lion extends Animal {
  keeper: ZooKeeper = new ZooKeeper();
}
 
function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}
 
createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;
```

이러한 패턴은 mixin 디자인 패턴을 구현하는데 사용됩니다.


### Generic Parameter Defaults

제네릭 타입 매개변수의 기본값을 선언하면 해당 타입 인수를 명시적으로 지정하지 않아도 됩니다. 예를 들어, 새로운 `HTMLelement`를 생성하는 함수를 만들 때 아무런 인자를 전달하지 않으면 `HTMLDivElement`를 생성하고 첫번째 인자로 요소를 전달하면 해당 요소의 타입에 따라 요소를 생성합니다. 또한 선택적으로 자식 요소에 대한 리스트를 전달할 수도 있습니다.

이전에는 해당 함수를 다음과 같이 정의해야 했습니다.
```ts
declare function create(): Container<HTMLDivElement, HTMLDivElement[]>;
declare function create<T extends HTMLElement>(element: T): Container<T, T[]>;
declare function create<T extends HTMLElement, U extends HTMLElement>(
  element: T,
  children: U[]
): Container<T, U[]>;
```

제네릭 매개변수 기본값을 사용하면 다음과 같이 사용할 수 있습니다.

```ts
declare function create<T extends HTMLElement = HTMLDivElement, U extends HTMLElement[] = T[]>(
  element?: T,
  children?: U
): Container<T, U>;
 
const div = create();
      //const div: Container<HTMLDivElement, HTMLDivElement[]>
 
const p = create(new HTMLParagraphElement());
     //const p: Container<HTMLParagraphElement, HTMLParagraphElement[]>
```

제네릭 매개변수 기본값은 다음 규칙을 따릅니다.
- 기본값이 있는 타입 매개변수는 **optional로 간주**됩니다.
- **Required 타입 매개변수**는 optional 타입 매개변수 뒤에 올 수 없습니다.
- 타입 매개변수의 기본값은 **제약조건이 있을 경우, 해당 조건을 만족**해야 합니다.
- 타입 인수를 명시할 때, required 타입 매개변수에 대해서만 타입 인수를 전달하며 전달되지않은 타입 매개변수는 **기본값으로 설정**됩니다.
- 기본값으로 지정된 상황에서 **타입 추론에 실패할 경우, 기본값**으로 추론됩니다.
- 기존 클래스나 인터페이스 선언과 병합되는 클래스 또는 인터페이스 선언은 **기존 타입매개변수에 기본값을 추가**할 수 있습니다.
- 기존 클래스나 인터페이스 선언과 병합되는 클래스 또는 인터페이스 선언은 기본값을 지정하는 경우 **새 타입 매개변수를 도입**할 수 있습니다.


### Variance Annotations
> 매우 특별한 문제를 마주쳤을 때 해결할 수 있는 심화 과정이며, 사용해야만 하는 상황에서만 사용되어야 합니다.

`Covariance`와 `Contravariance`는 두 제네릭 타입 간의 관계를 설명하는 타입 이론 용어입니다. 다음은 이 개념에 대한 간단한 설명입니다.

예를 들어, 특정 타입을 생성할 수 있는 객체를 나타내는 인터페이스가 있다고 가정해 보겠습니다.
```ts
interface Producer<T> {
  make(): T;
}
```

우리는 `Producer<Animal>`이 필요한 곳에서 `Producer<Cat>`를 사용할 수 있습니다. 그 이유는 `Cat`은 `Animal`의 하위 타입이기 때문입니다. 이러한 관계를 **공변성(`covariance`)** 라고 합니다. 즉 `Producer<T>`와 `Producer<U>` 간의 관계는 `T`와 `U` 간의 관계와 동일합니다.

반대로 만약 특정 타입을 소비할 수 있는 인터페이스가 있다면
```ts
interface Consumer<T> {
  consume: (arg: T) => void;
}
```
따라서 `Consumer<Cat>`이 필요한 곳에서 `Consumer<Animal>`이 사용될 수 있습니다. `Animal`을 받을 수 있는 함수라면 `Cat` 또한 전달받을 수 있기 때문입니다. 이러한 관계를 **반공변성(`Contravariance`)** 라고 합니다. 즉, `Consumber<T>`와 `Consumer<U>`간의 관계는 `U`와 `T`의 관계와 동일합니다.

여기서 방향이 공변성과 반대인 점에 주목해주세요! 이 때문에 반공변성은 "스스로 상쇄"되는 반면 공변성은 그렇지 않습니다. 

타입스크립트같은 구조적 타입 시스템에서 공변성과 반공변성은 타입의 정의에서 자연스럽게 발생하는 현상입니다. 제네릭이 없어도 우리는 공변적인 혹은 반공변적인 관계를 볼 수 있습니다.

```ts
interface AnimalProducer {
  make(): Animal;
}
// A CatProducer can be used anywhere an
// Animal producer is expected
interface CatProducer {
  make(): Cat;
}
```
타입스크립트는 구조적 타입 시스템이기에, `Producer<Cat>`이 `Producer<Animal>`이 필요한 곳에 사용될 수 있는 지 확인하려면, 일반적인 알고리즘에서는 두 타입을 구조적으로 확장하여 비교해야 합니다.

하지만 **변성(`Vairance`)** 은 매우 유용한 최적화를 가능하게 해줍니다. 만약 `Producer<T>`가 `T`에 대해 공변적이라면 우리는 `Producer<Cat>`과 `Producer<Animal>`이 `Cat`과 `Animal`과 동일한 관계를 가질 것이라는 것을 알 수 있기에 단순히 `Cat`과 `Animal`만 비교하면 됩니다.

이 로직은 같은 타입의 두 인스턴스를 비교할 때만 사용할 수 있습니다. 만약 `Producer<T>`와 `FastProducer<U>`가 있을 때, `T`와 `U`가 해당 타입 내에 동일한 위치를 참조할 수 있다고 보장할 수 없기에, 구조적으로 비교를 수행해야 합니다.

변성은 구조적 타입에서 자연스럽게 발생하는 속성이기에 타입스크립트는 자동으로 모든 제네릭 타입에 대한 변성을 추론합니다. 하지만 특정한 종류의 순환타입과 같은 희귀한 경우에는 이런 추론이 부정확할 수 있습니다. 만약 이런 상황이 발생한다면 타입 매개변수에 `variance annotation`을 추가하여 특정한 변성을 강제할 수 있습니다.

```ts
// Contravariant annotation
interface Consumer<in T> {
  consume: (arg: T) => void;
}
// Covariant annotation
interface Producer<out T> {
  make(): T;
}
// Invariant annotation
interface ProducerConsumer<in out T> {
  consume: (arg: T) => void;
  make(): T;
}
```

이 작업은 구조적으로 발생해야 하는 변성과 동일한 변성을 작성할 때만 사용하셔야 합니다.

>구조적 변성과 일치하지 않는 변성 어노테이션을 작성해서는 안 됩니다!

variance annotation은 인스턴스간 비교에서만 효과가 있다는 점이 중요합니다. 구조적 비교 중에는 아무런 영향을 미치지 않습니다. 예를 들어, 변성 어노테이션을 사용하여 타입을 실제로 **불변(invariant)**으로 "강제"할 수는 없습니다:

```ts
// DON'T DO THIS - variance annotation
// does not match structural behavior
interface Producer<in out T> {
  make(): T;
}
// Not a type error -- this is a structural
// comparison, so variance annotations are
// not in effect
const p: Producer<string | number> = {
    make(): number {
        return 42;
    }
}
```
`number`를 반환하는 객체 리터럴의 `make`함수는 `number`가 `string | number`가 아니기 때문에 에러가 발생할 것이라고 추측할 수 있습니다. 하지만 객체 리터럴은 익명 타입이며 `Producer<string | number>`가 아니기 때문에 인스턴스 간 비교가 아닙니다.

> Variance annotations는 구조적 동작을 변경하지 않으며, 특정 상황에서만 참고됩니다.

vaiance annotations는 왜 사용하는지, 그 한계가 무엇인지, 어떤 상황에서 효과가 없는지를 명확히 이해할 때만 사용하셔야 합니다. 타입스크립트가 인스턴스 간 비교 혹은 구조적 비교를 사용하는 지는 명시적으로 정의된 동작이 아니며 정확성이나 성능상 사유에 따라 버전마다 변경될 수 있습니다. 따라서 variance annotation은 반드시 타입의 구조적 동작과 일치할 때만 작성해야 합니다. 

특정 변성을 강제하려고 variance annotation을 사용하지 마세요 이는 코드에서 예측할 수 없는 동작을 야기할 수 있습니다.

> 타입의 구조적 동작과 일치하지 않는 경우에는 절대로 variance annotation을 작성하지 마십시오.

타입스크립트는 자동적으로 제네릭 타입에서 변성을 추론할 수 있습니다. variance annotation을 작성하는 것은 거의 필요하지 않으며 특정 상황에서 필요로 할 때만 사용해야 합니다. Vairance annotation은 타입의 구조적 동작을 변경시키지 않으며 상황에 따라 인스턴스간 비교가 예상되는 경우에 구조적인 비교가 이루어질 수도 있습니다. Variance Annotations은 이러한 구조적 맥락에서 타입 동작 방식을 수정하는데 사용할 수 없을 것이며 해당 annotation이 구조적 정의와 동일할 때만 작성해야 합니다. 이를 정확히 구현하는 것은 어려우며 타입스크립트가 대부분 변성을 올바르게 추론할 수 있기 때문에, 일반적인 코드에서 variance annotations는 작성하지 않아야 합니다.

> 타입 검사와 같은 상황에서 vairance annotations를 사용하지 마세요. 목적성과 다릅니다.


vaiance annotation은 타입 디버깅 시에 유용할 수도 있습니다. 타입스크립트는 annotaition된 변성이 명확하게 잘못된 경우, 오류를 발생시킵니다. 이는 특정한 변성 관계가 예상과 맞지 않음을 발견할 수 있는 방법으로 유용할 수 있습니다

```ts
// Error, this interface is definitely contravariant on T
interface Foo<out T> {
  consume: (arg: T) => void;
}
```
하지만 variance annotations는 더 엄격하게 적용될 수 있습니다(실제 변성이 공변일 때 `in out`이 유효) 디버깅이 끝나면 variance annotation을 제거하는 것이 좋습니다.

마지막으로 타입 검사 성능을 극대화하려고 프로파일러를 실행하고 특정 타입이 느리다는 것을 발견한 후에 variance annotation을 작성하여 아주 복잡한 타입에서 조금의 성능 개선을 볼 수는 있습니다.

variance annotations는 타입 검사 동작을 변경하려는 용도가 아니므로 그런 용도로 사용하면 안됩니다.
