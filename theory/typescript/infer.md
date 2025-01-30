# Infer
- 타입을 추론하는 역할
- 조건부 타입에서 미리 정의되지 않은 타입을 유연하게 정의할 수 있게 도와주는 문법
- 삼항 연산자를 사용한 조건문의 형태를 가지는데, extends로 조건을 서술하고 infer로 타입을 추론하는 방식을 취함
- 복잡한 타입 코드를 줄여줌

## 예시 코드 1
```ts
 type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any
```

위 타입은 제네릭으로 받은 T가 Promise로 래핑된 경우 K 반환, 그렇지 않은 경우에는 any를 반환한다. Promise<infer K>는 Promise의 반환값을 추론해 해당 값의 타입을 K로 한다는 말이다.


### 출처
- 우아한 타입스크립트 5장 타입 활용하기 일부 - 우아한형제들 웹프론트엔드개발그룹 저, 우아한 테크/한빛미디어
- [타입스크립트 핸드북 - infer란?](https://joshua1988.github.io/ts/usage/infer.html#infer%EB%9E%80)