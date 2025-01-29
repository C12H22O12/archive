# NoInfer

``` Typescript
export type NoInfer<T> = [T][T extends any ? 0 : never]
```

- `/table-core/src/utils.ts` 에 위치한 코드
- 제네릭 함수에서 파라미터의 타입 추론을 제한
- 이 형태를 타입스크립트는 타입을 추론하지 않음* -> 이를 이용하여 타입 추론을 못하도록 막음
  - [T] : 튜플 타입
  - `T extends any ? 0 : never` : 조건부 타입
  - [0] : 인덱스 접근
    - `T extends any ? 0 : never`에서 T는 항상 true이므로 `[T extends any ? 0 : never]`의 값은 [0]이 됨

---
* : 클로드를 이용하여 추론한 답이므로, 사실 검증 필요

<br />
<br />
<br />

# 예시 코드 1
``` ts
const f1 = <C extends string>(colors: C[], color: C) => {
    const result = [] as C[]
    return result
}

const f2 = <C extends string>(colors: C[], color: NoInfer<C>) => {
    const result = [] as C[]
    return result
}

const a1 = f1(["red", "yellor", "green"], "red")
// ⬆️ work : a1 is ("red" | "yellor" | "green")[]

const a2 = f1(["red", "yellor", "green"], "blue")
// ⬆️ work : a2 is ("red" | "yellor" | "green" | "blue")[]

const b1 = f2(["red", "yellor", "green"], "red")
// ⬆️ work : b1 is ("red" | "yellor" | "green")[]

const b2 = f2(["red", "yellor", "green"], "blue")
// ⬆️ error : b2 is ("red" | "yellor" | "green" | "blue")[]
// Error : Argument of type '"blue"' is not assignable to parameter of type '"red" | "yellor" | "green"'.
```

**NoInfer를 사용하지 않은 f1의 경우** colors에 할당된 배열에 color에 할당한 값이 존재하지 않아도 동작, 결과값은 colors에 color가 포함된 배열이 도출

**NoInfer를 사용한 f2의 경우** colors에 할당된 배열에 color에 할당한 값이 존재하지 않을 경우 에러, 결과값은 colors에 할당된 배열이 도출

<br />
<br />
<br />

# 예시 코드 2
``` Typescript
const fn0 = <A extends any>(a0: A, a1: NoInfer<A>): A => {
 return {} as unknown as A // just for the example
}

const fn1 = <A extends any>(a0: NoInfer<A>, a1: A): A => {
 return {} as unknown as A // just for the example
}

const fn2 = <A extends any>(a0: NoInfer<A>, a1: NoInfer<A>): A => {
 return {} as unknown as A // just for the example
}

const test0 = fn0('b', 'a')
// ⬆️ error: Argument of type '"a"' is not assignable to parameter of type '"b"'.

const test1 = fn1('b', 'a')
// ⬆️ error: infer priority is Argument of type '"b"' is not assignable to parameter of type '"a"'.

const test2 = fn2('b', 'a')
// ⬆️ works: infer priority is `b` | `a`
```


<br />
<br />
<br />

# 예시 코드 3
``` Typescript
// placeholder for a real express response.send
const sendResponse = (x: any) => console.log(x);

function sendResult<T = never>(send: any, result: NoInfer<T>) {
  send(result);
}

type Num = { x: number };
sendResult<Num>(sendResponse, { x: 1 }); // okay
sendResult<Num>(sendResponse, { x: "sss" }); // error

sendResult(sendResponse, { x: 1 }); // error
```



## 연관 링크

- [ts-toolbelt의 NoInfer 항목](https://millsp.github.io/ts-toolbelt/modules/function_noinfer.html)
- [해당 문제가 처음 제기된 Stackoverflow 질문](https://stackoverflow.com/questions/56687668/a-way-to-disable-type-argument-inference-in-generics)