---
title: TypeScript 유틸리티 타입
date: 2021-05-26 22:05:27
category: typescript
thumbnail: { thumbnailSrc }
draft: false
---

> 타입 스크립트의 장점 중 하나는 기존의 타입을 이용하여 새로운 타입을 정의(type transformation)할 수 있는 유연성을 제공한다는 것입니다. 

타입 스크립트는 자주 사용되는 type transformation에 대해서 전역적으로 사용할 수 있는 유용한 **유틸리티 타입**을 제공하고 있습니다. 유틸리티 타입에 대해 기능과 함께 어떻게 구현되어 있는지 살펴보면서 맵드 타입, 조건부 타입과 같은 타입스크립트 고급 기능에 대해서도 알아봅시다.

## Partial<Type>

Type의 모든 프로퍼티를 옵셔널로 만듭니다. 따라서 기존 타입의 모든 가능한 부분 집합의 타입을 나타냅니다.

```tsx
type Partial<T> = {
    [P in keyof T]?: T[P];
};

// example

interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Pick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
```

### Mapped Type

Partial는 맵드 타입으로 만들어져 있습니다. 맵드 타입이란 `keyof` 연산자를 통해 객체 타입의 프로퍼티 키로 이루어진 유니언 타입을 생성하고 프로퍼티 키(유니언 멤버)에 대해 순회하면서 새로운 타입을 만드는 제네릭 타입입니다.

아래의 예제는 맵드 타입을 이용하여 Type의 기존의 모든 프로퍼티의 값이 boolean의 타입을 가지도록 만드는 타입을 만들어주는 OptionsFlgas 제네릭 타입을 정의하고 있습니다.

```tsx
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```

## Required<Type>

Partial과 반대로 Type의 모든 프로퍼티를 required로 만듭니다.

```tsx
type Required<T> = {
    [P in keyof T]-?: T[P];
};
// '-' prefix를 통해 optionality modifier('?') 제거 → required

// example

interface Props {
  a?: number;
  b?: string;
}

const obj: Props = { a: 5 };

const obj2: Required<Props> = { a: 5 };
// Property 'b' is missing in type '{ a: number; }' but required in type 'Required<Props>'.
```

### Mapping Modifiers

Required 또한 Partial과 같이 맵드 타입을 이용하고 있습니다. 맵드 타입에서는 `?`와 `readonly` modifier로 기존의 프로퍼티를 옵셔널하게 만들거나 읽기 전용으로 만들 수 있습니다. 또한 `-`나 `+`를 앞에 붙여서 해당 modifier를 더하거나 제거할 수 있습니다. (생략되면 기본적으로  `+`가 붙습니다)

## Readonly<Type>

Type의 모든 프로퍼티를 readonly로 만든 타입을 반환합니다. 각 프로퍼티가 재할당될 수 없도록 만듭니다.

```tsx
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

불변 객체를 생성하는 Object.freeze의 타입은 Readonly를 사용하여 정의되어 있습니다. 

```tsx
// Object.freeze
freeze<T>(o: T): Readonly<T>;
```

## Record<Keys, Type>

프로퍼티 키의 집합을 나타내는 Keys에서 각 프로퍼티에 대해 Type 값을 갖는 객체 타입을 만듭니다.

```tsx
// keyof any : string | number | symbol
type Record<K extends keyof any, T> = {
    [P in K]: T;
};

// example

interface CatInfo {
  age: number;
  breed: string;
}

type CatName = "miffy" | "boris" | "mordred";

const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
```

## Pick<Type, Keys>

Type으로부터 프로퍼티 키의 집합을 나타내는 Keys를 통해 사용할 프로퍼티만을 뽑아낸 새로운 타입을 만듭니다.

```tsx
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

## Omit<Type, Keys>

Pick과 반대의 역할을 합니다. Type으로부터 모든 키를 선택하고 프로퍼티 키의 집합을 나타내는 Keys에 속하는 프로퍼티를 제거합니다.

```tsx
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

## Exclude<Type, ExcludedUnion>

Type에서 ExcludedUnion에 속하는 멤버를 삭제합니다.

```tsx
type Exclude<T, U> = T extends U ? never : T;
// Distributive Conditional Types
// 조건부 타입이 각 유니언 멤버에 대해 순회하면서 적용된다.

// example

type T = Exclude<"a" | "b" | "c", "a" | "b">;
// type T = "c"
```

### Distributive Conditional Types

조건부 타입이 제네릭에서 사용되고 유니언 타입이 입력되었을 때, 조건부 타입은 각 유니언 멤버에 대해 적용됩니다.

```tsx
type ToArray<Type> = Type extends any ? Type[] : never;

type StrArrOrNumArr = ToArray<string | number>;
// type StrArrOrNumArr = string[] | number[]

// 만약 각 유니언 멤버에 대해 적용되는 것을 원하지 않는다면?
// extends 키워드를 사용한 양쪽에 [] 키워드로 감싸준다
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;

type StrOrNumArr = ToArrayNonDist<string | number>;
// type StrOrNumArr = (string | number)[]
```

## Extract<Type, Union>

Exclude와 반대의 역할을 합니다. Type의 유니언 멤버에서 Union에 속하는 멤버를 추출합니다.

```tsx
type Extract<T, U> = T extends U ? T : never;

// example

type T = Extract<"a" | "b" | "c", "a" | "f">;
// type T = "a"
```

## NonNullable<Type>

Type에서 null과 undefined를 제거합니다.

```tsx
type NonNullable<T> = T extends null | undefined ? never : T;

// example

type T = NonNullable<string[] | null | undefined>;
// type T = string[]
```

## Parameters<Type>

함수 타입의 Type에서 파라미터의 튜플 타입을 추출합니다.

```tsx
type Parameters<T extends (...args: any) => any> 
	= T extends (...args: infer P) => any ? P : never;
//infer 키워드를 통해 파라미터 튜플 타입을 변수 P를 담을 수 있다.

// example

type T0 = Parameters<() => string>;
// type T0 = []

type T1 = Parameters<(s: string) => void>;
// type T1 = [s: string]

type T2 = Parameters<(arg: { a: number; b: string }) => void>;
// type T2 = [arg: {
//    a: number;
//    b: string;
//  }]

type T3 = Parameters<string>; // never
// Type 'string' does not satisfy the constraint '(...args: any) => any'.
```

### Tuple Type

Parameters는 파라미터의 튜플 타입을 추출한다고 했습니다. 튜플 타입은 배열 타입에 속하며 고정된 길이의 요소를 갖고 각 요소의 타입이 정해져 있습니다. string과 number의 쌍을 갖는 값을 정의하고 싶을 때 다음과 같이 튜플 타입을 사용할 수 있습니다.

```tsx
type StringNumberPair = [string, number];
```

### infer

조건부 타입을 사용하면서 새로운 타입을 추출하고자 할 때 사용될 수 있습니다. extends 키워드 뒤에 infer 키워드를 사용하여 새로운 제네릭 타입 변수를 만들고 원하는 타입을 담을 수 있습니다.

```tsx
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;

type Unpromisify<T> = T extends Promise<infer R> ? R : T
```

## ReturnType<Type>

함수 타입의 Type에서 리턴 타입을 추출합니다.

```tsx
type ReturnType<T extends (...args: any) => any> 
	= T extends (...args: any) => infer R ? R : any;

type T0 = ReturnType<() => string>;
// type T0 = string

type T1 = ReturnType<(s: string) => void>;   
// type T1 = void

type T2 = ReturnType<<T>() => T>;
// type T2 = unknown
```

## 이외에도 ..

- InstanceType<Type>
- ThisParameterType<Type>
- OmitThisParameter<Type>
- ThisType<Type>
- Uppercase<StringType>
- Lowercase<StringType>
- Capitalize<StringType>
- Uncapitalize<StringType>

와 같이 인스턴스나 this, 그리고 문자열 관련된 다양한 유틸리티 타입이 제공되고 있습니다. 관심이 있다면 공식 문서의 유틸리티 타입에 대해서 살펴보는 것을 추천드립니다 👍

## 🔗 참고

[https://www.typescriptlang.org/docs/handbook/utility-types.html](https://www.typescriptlang.org/docs/handbook/utility-types.html)