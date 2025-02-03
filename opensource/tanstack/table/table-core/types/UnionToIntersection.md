# UnionToIntersection

``` Typescript
export type UnionToIntersection<T> = (
  T extends any ? (x: T) => any : never
) extends (x: infer R) => any ? R : never
```

- [분산 조건부 타입](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#distributive-conditional-types)과 [조건부 타입의 추론](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#type-inference-in-conditional-types)을 이용하여, 유니온 타입을 교차 타입으로 변환

1. 들어오는 T가 유니온일 경우 `T extends any ? (x: T) => any : never`은 T 각각의 요소에 적용
2. 1에서 적용된 각 요소를 다시 infer을 통해 추출, 이를 통해 매개변수 타입을 추론하면 인터섹션이 됨 

<br />
<br />
<br />

## 연관 링크

- [Transform union type to intersection type - StackOverflow](https://stackoverflow.com/questions/50374908/transform-union-type-to-intersection-type)
