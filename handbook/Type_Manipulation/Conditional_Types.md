대부분의 유용한 프로그램의 핵심에는 입력에 따라 결정을 내려야 하는 과정이 있습니다. 자바스크립트 또한 이와 다르지 않으며 값의 타입을 쉽게 조사할 수 있다는 점에서 이러한 결정들은 입력 값의 타입에 기반하기도 합니다. 조건부 타입(`Conditional Types`)는 입력값과 출력값의 타입 간 관계를 설명하는데 도움을 줍니다.

```ts
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}
 
type Example1 = Dog extends Animal ? number : string;
        
type Example1 = number
 
type Example2 = RegExp extends Animal ? number : string;
        
type Example2 = string
```

조건부 타입은 자바스크립트의 조건 표현식(`condition ? trueExpression : falseExpression`)과 비슷한 형태를 지닙니다.

```ts
  SomeType extends OtherType ? TrueType : FalseType;
```

`extends` 왼쪽에 위치한 타입은 '참'일 경우 오른쪽 첫 번째 분기(`true` 분기)의 타입이 할당되며 그 외에는 두 번째 분기(`false` 분기)의 타입이 할당됩니다.

위 예제로 보았을 때, 조건부 타입은 당장 유용해보이진 않습니다. `Dog`가 `Animal`을 확장할 수 있는지는 우리가 직접 판단이 가능하며 그에 따라 `number`나 `string`을 선택할 수 있으니까요! 하지만 조건부 타입은 제네릭과 함께 사용할 때, 강력한 힘을 발휘합니다.

예를 들어, `createLabel` 함수를 살펴봅시다.

```ts
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}
 
function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw "unimplemented";
}
```

`createLabel`의 오버로드는 입력값의 타입에 따라 선택되는 하나의 자바스크립트의 함수를 보여줍니다. 여기서 몇가지 중요한 점을 살펴봅시다.
1. 만약 라이브러리가 해당 API에 비슷한 유형의 함수들을 계속해서 만들어가야 하고, 점차 번거로워질 것입니다.
2. 우리는 세 가지 오버로드를 만들어야 합니다. 타입이 확실한 경우(`string`, `number`)에 대해 각각 하나씩, 그리고 가장 일반적인 경우(`string | number`)에 대해 하나를 추가로 만들어야 합니다. `createLabel`이 처리할 수 있는 새로운 타입이 추가될 때마다 필요한 오버로드의 수는 기하급수적으로 증가하게 됩니다.

대신 우리는 조건부 타입을 이용해볼 수 있습니다.
```ts
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
```

이를 통해 우리는 하나의 함수에 수많은 오버로드들을 간단히 하여 조건부 타입으로 만들어낼 수 있습니다.

```ts
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw "unimplemented";
}
 
let a = createLabel("typescript");
   //let a: NameLabel
 
let b = createLabel(2.8);
   	//let b: IdLabel
 
let c = createLabel(Math.random() ? "hello" : 42);
	//let c: NameLabel | IdLabel
```

### Conditional Type Constraints
조건부 타입에서의 검사는 종종 새로운 정보를 제공하기도 합니다. 타입 가드를 통한 `narrowing`이 더 세밀한 타입 정보를 제공하듯, 조건부 타입의 참(`true`) 분기는 검사한 타입에 따라 제네릭 타입을 더 구체적으로 제한합니다.

```ts
type MessageOf<T> = T["message"];
//Type '"message"' cannot be used to index type 'T'.
```

해당 예제에서 `T`가 `message`라는 속성을 갖고 있는지 보장할 수 없기 때문에 타입스크립트에서 오류가 발생합니다. 하지만 `T`를 제약하여 오류를 제거할 수 있습니다.

```ts
type MessageOf<T extends { message: unknown }> = T["message"];
 
interface Email {
  message: string;
}
 
type EmailMessageContents = MessageOf<Email>;
              //type EmailMessageContents = string
```

하지만 `messageOf`가 어떠한 타입도 받는 대신, `message` 속성이 없을 경우 기본값으로 `never`타입을 설정하도록 하고 싶으면 어떻게 해야할까요? 바로 제약조건을 밖으로 빼내어 조건부 타입을 설정하는 것입니다.

```ts
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;
 
interface Email {
  message: string;
}
 
interface Dog {
  bark(): void;
}
 
type EmailMessageContents = MessageOf<Email>;
              //type EmailMessageContents = string
 
type DogMessageContents = MessageOf<Dog>;
             //type DogMessageContents = never
```

참 분기에서는 타입스크립트가 `T`가 `message` 속성을 지닌다는 것을 인지하게 됩니다.

다른 예제로, `Flatten`이라는 타입을 작성하여 배열 타입을 그 요소 타입으로 평탄화(`flatten`)할 수 있습니다. 하지만 그 외에는 영향을 주지 않게 작성할 수 있습니다.

```ts
type Flatten<T> = T extends any[] ? T[number] : T;
 
// Extracts out the element type.
type Str = Flatten<string[]>;
     //type Str = string
 
// Leaves the type alone.
type Num = Flatten<number>;
     //type Num = number
```

`Flatten`이 배열 타입을 전달받으면 `number`를 통해 인덱스 접근(`indexed access`)을 통해 `string[]`의 요소 타입을 추출합니다. 그 외에는 전달받은 타입 그대로 반환합니다.

### Inferring Within Conditional Types

우리는 조건부 타입을 통해 제약조건을 설정하고 타입을 추출하였습니다. 이러한 작업은 조건부 타입을 통해 더 쉽게 처리할 수 있는 일반적인 연산일 뿐입니다.
