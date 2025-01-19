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

