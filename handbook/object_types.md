자바스크립트에서 데이터를 그룹화하고 전달하는 가장 기본적인 방법은 객체를 사용하는 것입니다. 타입스크립트에서 이러한 것을 객체 타입(`object types`)라고 부릅니다.

익명으로도 사용가능합니다.
```ts
function greet(person: { name: string; age: number }) {
  return "Hello " + person.name;
}
```

또는 interface를 사용하여 이름을 지정할 수도 있고
```ts
interface Person {
  name: string;
  age: number;
}
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```

타입 별칭을 사용할 수도 있습니다.
```ts
type Person = {
  name: string;
  age: number;
};
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```

