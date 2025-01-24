반복을 줄이고 싶다면 가끔은 하나의 타입이 다른 타입을 기반으로 해야 할 때도 있습니다.

`Mapped types`는 인덱스 시그니처의 문법에서 동작하며 기존에 선언되지 않은 속성의 타입들을 선언하는데 사용됩니다.

```ts
type OnlyBoolsAndHorses = {
  [key: string]: boolean | Horse;
};
 
const conforms: OnlyBoolsAndHorses = {
  del: true,
  rodney: false,
};
```

mapped type은 `PropertyKeys`(`keyof`를 통해 생성)의 유니온을 사용하여 키를 반복해, 타입을 생성하는 제네릭 타입입니다. 

```ts
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```

해당 예제에서 `OptionsFlags`는 `Type`의 모든 속성들을 가져와서 그 값들을 모두 `boolean`타입으로 변경합니다.

```ts
type Features = {
  darkMode: () => void;
  newUserProfile: () => void;
};
 
type FeatureOptions = OptionsFlags<Features>;
           
type FeatureOptions = {
    darkMode: boolean;
    newUserProfile: boolean;
}
```

### Mapping Modifiers

매핑 중에 두 가지 추가적인 수정자(`readonly`와 `?`)를 사용할 수 있으며 각각 변경불가능성(mutability)과 선택성(optionality)에 영향을 끼칩니다. 

`-` 혹은 `+` 접두사를 통해 해당 수정자들을 추가하거나 제거할 수 있습니다. 만약 접두사를 추가하지 않는다면 기본적으로 `+`가 적용됩니다.

```ts
// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};
 
type LockedAccount = {
  readonly id: string;
  readonly name: string;
};
 
type UnlockedAccount = CreateMutable<LockedAccount>;
           //type UnlockedAccount = {
		   //		id: string;
    	   //		name: string;
		   //	}
```
```ts
// Removes 'optional' attributes from a type's properties
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};
 
type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};
 
type User = Concrete<MaybeUser>;
      //type User = {
      //	id: string;
      //	name: string;
      //	age: number;
	  //  }
```

### Key Remapping via `as`

타입스크립트 4.1 이후 부터는 mapped type에 `as` 절과 함께 키를 재 매핑(re-map)할 수 있습니다.

템플릿 리터럴 타입을 활용하여 이전 것을 토대로 새로운 속성명을 생성할 수 있습니다.

```ts
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};
 
interface Person {
    name: string;
    age: number;
    location: string;
}
 
type LazyPerson = Getters<Person>;
         //type LazyPerson = {
   		 //	 	getName: () => string;
    	 //  	getAge: () => number;
    	 //	 	getLocation: () => string;
		 //   }
```

또한 `never`를 통해 조건부 타입에서 키들을 필터링할 수 있습니다.

```ts
// Remove the 'kind' property
type RemoveKindField<Type> = {
    [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
};
 
interface Circle {
    kind: "circle";
    radius: number;
}
 
type KindlessCircle = RemoveKindField<Circle>;
           
type KindlessCircle = {
    radius: number;
}
```

`string | number | symbol` 뿐만 아니라 어떠한 타입의 유니온들을 매핑할 수 있습니다.


```ts
type EventConfig<Events extends { kind: string }> = {
    [E in Events as E["kind"]]: (event: E) => void;
}
 
type SquareEvent = { kind: "square", x: number, y: number };
type CircleEvent = { kind: "circle", radius: number };
 
type Config = EventConfig<SquareEvent | CircleEvent>
       //type Config = {
       //		square: (event: SquareEvent) => void;
       //		circle: (event: CircleEvent) => void;
	   //  }
```

### Further Exploration
Mapped Types은 타입 조작의 다른 기능과 조합이 좋습니다. 예를 들어, mapped type이 객체의 `pii` 속성이 리터럴 `true`로 설정되어 있는지 여부에 따라 `true` 또는 `false`를 반환합니다.

```ts
type ExtractPII<Type> = {
  [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false;
};
 
type DBFields = {
  id: { format: "incrementing" };
  name: { type: string; pii: true };
};
 
type ObjectsNeedingGDPRDeletion = ExtractPII<DBFields>;
                 // type ObjectsNeedingGDPRDeletion = {
    			 //		id: false;
    			 // 	name: true;
				 //  }
```