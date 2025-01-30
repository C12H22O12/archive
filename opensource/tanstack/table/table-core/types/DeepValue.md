# DeepValue

```Typescript
export type DeepValue<T, TProp> =
  T extends Record<string | number, any>
    ? TProp extends `${infer TBranch}.${infer TDeepProp}`
      ? DeepValue<T[TBranch], TDeepProp>
      : T[TProp & string]
    : never

```

- `/table-core/src/utills.ts` 에 위치한 코드
- 재귀적 유틸리티 타입 : 타입스크립트에서 타입을 재귀적으로 정의하여 타입을 추론하는 유틸리티 타입
- 제네릭 파라미터 T가 key는 string | number 타입, value는 any인 객체 타입인지 여부에 따라 값이 달라진다.
    1. 1의 내용이 참일 경우 TProp이 `${infer TBranch}.${infer TDeepProp}`인지 확인한다. 이 때 TBranch와 TDeepProp에는 infer을 통해 추출된 값이 들어간다.
        1. `${infer TBranch}.${infer TDeepProp}`는 [템플릿 리터럴 타입](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)으로 TProp이 더 이상 ‘.’을 포함하지 않을 때까지 재귀적으로 실행
        2. 1-1의 내용이 참일 경우 DeepValue는 DeepValue<T[TBranch], TDeepProp>으로 재귀가 동작한다.
        3. 1-1의 내용이 거짓일 경우 T의 모든 key의 타입을 string으로 강제 변환한다.
    2. 1의 내용이 거짓일 경우(T가 객체가 아닐 경우) DeepValue에는 never 타입이 할당된다


<br />
<br />
<br />

# 보충 - chatGPT 답변
## 검증 요청 당시 답변
### 1. 분석 내용 확인  
당신이 분석한 내용은 대부분 정확합니다. 특히 **재귀 구조**와 `T`가 `Record<string | number, any>`인지 판별하는 부분은 올바르게 이해하셨습니다. 하지만 몇 가지 추가할 수 있는 설명이 있습니다.

---

### 2. 부족한 내용 보충  

#### 부족한 점 1: `T[TProp & string]`의 역할  
```ts
: T[TProp & string]
```
- 여기서 `TProp & string`을 사용한 이유는 `TProp`이 `string | number | symbol`일 가능성이 있기 때문입니다.
- 하지만 `T`는 `Record<string | number, any>`이므로, 유효한 키는 `string | number`입니다.
- `TProp & string`은 `TProp`이 `string`일 경우 그대로 두고, `number`인 경우 `string`으로 강제 변환하는 역할을 합니다.
  - 이는 TypeScript에서 `Object.keys()`의 반환값이 항상 `string[]`이기 때문입니다.
  - 예를 들어 `const obj = { 1: "a" }`일 때, `Object.keys(obj)`는 `["1"]`을 반환합니다.

#### 부족한 점 2: `DeepValue<T[TBranch], TDeepProp>`의 재귀적 작동 방식  
- `TProp`이 `"user.address.street"` 같은 형태라면 `infer`를 사용하여 `"user"`와 `"address.street"`로 나눕니다.
- `DeepValue<T[TBranch], TDeepProp>`는 `T[TBranch]`을 기준으로 다시 `DeepValue`를 실행하여 `"address.street"`을 처리합니다.
- 즉, `"user.address.street"`가 `"user" | "address.street"` → `"address" | "street"` → `"street"` 형태로 하나씩 분리되며 작동합니다.

#### 부족한 점 3: `never` 반환의 의미  
```ts
: never
```
- `T`가 객체(`Record<string | number, any>`)가 아니면 `never`을 반환합니다.
- 이는 `DeepValue`가 **객체에서 깊은 속성 값을 참조할 때만 의미가 있기 때문**입니다.
- 예를 들어 `DeepValue<number, "prop">`는 의미가 없으므로 `never`을 반환합니다.

---

### 3. `TProp extends \`${infer TBranch}.${infer TDeepProp}\``의 상세 설명

#### **구문 분석**
```ts
TProp extends `${infer TBranch}.${infer TDeepProp}`
```
이 구문은 **[템플릿 리터럴 타입](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)**을 활용한 패턴 매칭입니다.

#### **동작 원리**
- `TProp`이 `"user.address.street"`처럼 `"."`을 포함하는 문자열인지 검사합니다.
- 만약 `"."`이 있다면:
  - `TBranch`에는 `"user"`가 들어가고,
  - `TDeepProp`에는 `"address.street"`가 들어갑니다.
- 만약 `"."`이 없다면 패턴이 일치하지 않아 `TProp & string`이 실행됩니다.

#### **예제 분석**
##### (1) `"user.address.street"`
```ts
TProp = "user.address.street"
TProp extends `${infer TBranch}.${infer TDeepProp}` ? ...
```
- `"user.address.street"`는 `"."`을 포함하고 있으므로, `TBranch = "user"`, `TDeepProp = "address.street"`로 분해됩니다.
- 그다음 `DeepValue<T["user"], "address.street">`를 호출하여 다시 재귀적으로 실행됩니다.

##### (2) `"username"`
```ts
TProp = "username"
TProp extends `${infer TBranch}.${infer TDeepProp}` ? ...
```
- `"username"`은 `"."`을 포함하지 않으므로 `TBranch`와 `TDeepProp`을 추출할 수 없습니다.
- 따라서 조건이 만족되지 않고, `T[TProp & string]`이 실행됩니다.

##### (3) `number`
```ts
TProp = 42
TProp extends `${infer TBranch}.${infer TDeepProp}` ? ...
```
- `42`는 문자열이 아니므로 템플릿 리터럴 패턴에 매칭되지 않습니다.
- 따라서 `never`이 반환됩니다.

---

### 🔥 **최종 정리**
✔ **당신의 분석 내용은 거의 정확했으며, 다음을 보충하면 더 완벽해집니다.**  
1. `TProp & string`은 `number`를 `string`으로 변환하는 역할을 합니다.  
2. `DeepValue<T[TBranch], TDeepProp>`는 재귀적으로 깊은 속성을 탐색합니다.  
3. `never`은 `T`가 객체가 아닐 때 반환됩니다.  
4. `TProp extends \`${infer TBranch}.${infer TDeepProp}\``은 **[템플릿 리터럴 타입](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)을 이용한 패턴 매칭**으로, `"."`을 기준으로 분할하여 재귀적으로 실행됩니다.


<br />
<br />
<br />

## 정리본
### **📌 TypeScript `DeepValue` 타입 분석**  

```ts
export type DeepValue<T, TProp> =
  T extends Record<string | number, any>
    ? TProp extends `${infer TBranch}.${infer TDeepProp}`
      ? DeepValue<T[TBranch], TDeepProp>
      : T[TProp & string]
    : never;
```

---

## **1. 개요**
`DeepValue<T, TProp>`는 객체 `T`에서 `TProp`(문자열 경로)을 기반으로 깊은 값을 추출하는 **재귀적인 유틸리티 타입**입니다.  

- **객체(`T`)와 경로(`TProp`)**를 기반으로 작동합니다.  
- **점 표기법(`.`)**을 이용해 재귀적으로 탐색합니다.  
- 존재하지 않는 속성을 참조할 경우 `never`을 반환합니다.  

---

## **2. 동작 원리 분석**
### **(1) `T extends Record<string | number, any>`**
```ts
T extends Record<string | number, any>
```
- `T`가 **객체 타입**인지 검사합니다.
- `Record<string | number, any>`이므로, `T`의 키는 `string` 또는 `number`여야 합니다.
- 객체가 아니라면 `never`을 반환합니다.

✅ **예시**
```ts
type A = DeepValue<{ name: string }, "name">;  // ✅ 가능
type B = DeepValue<number, "value">;           // ❌ never (number는 객체가 아님)
```

---

### **(2) `TProp extends \`${infer TBranch}.${infer TDeepProp}\``**
```ts
TProp extends `${infer TBranch}.${infer TDeepProp}`
```
- `TProp`가 `.`을 포함하는 문자열인지 검사합니다.
- 만약 `"user.address.street"`와 같은 형태라면:
  - `TBranch = "user"`, `TDeepProp = "address.street"`로 분리됩니다.
- 이 경우 `DeepValue<T[TBranch], TDeepProp>`을 재귀적으로 호출하여 더 깊은 값을 찾습니다.

✅ **예시**
```ts
type Obj = {
  user: {
    address: {
      street: string;
    };
  };
};

type A = DeepValue<Obj, "user.address.street">;  // string
```
(→ `"user"`를 찾아 `{ address: { street: string } }`을 얻고, 다시 `"address"`를 찾아 `{ street: string }`을 얻고, 마지막으로 `"street"`을 찾아 `string`을 반환)

---

### **(3) `T[TProp & string]`**
```ts
T[TProp & string]
```
- `TProp`이 더 이상 점(`.`)을 포함하지 않는다면, `T[TProp]`을 반환합니다.
- 여기서 `TProp & string`을 사용한 이유는:
  - `TProp`이 `string | number | symbol`일 수 있기 때문입니다.
  - `TProp & string`을 사용하면 `number`를 강제로 `string`으로 변환할 수 있습니다.
  - 이는 `Object.keys()`가 항상 `string[]`을 반환하기 때문에 필요합니다.

✅ **예시**
```ts
type Obj = {
  name: string;
};

type A = DeepValue<Obj, "name">;  // string
```

---

### **(4) `never` 반환**
```ts
: never
```
- `T`가 객체가 아니거나, `TProp`이 유효한 키가 아닐 경우 `never`을 반환합니다.

✅ **예시**
```ts
type A = DeepValue<number, "value">;  // never (number는 객체가 아님)
type B = DeepValue<{ name: string }, "age">;  // never (age 키가 없음)
```

---

## **3. 최종 정리**
| 표현식 | 의미 |
|--------|------|
| `T extends Record<string | number, any>` | `T`가 객체인지 확인 |
| `TProp extends \`${infer TBranch}.${infer TDeepProp}\`` | `TProp`이 점(`.`)을 포함하는지 확인 |
| `DeepValue<T[TBranch], TDeepProp>` | `TBranch`를 찾아 다시 `DeepValue` 실행 (재귀) |
| `T[TProp & string]` | 점(`.`)이 없으면 최종 값을 반환 |
| `never` | `T`가 객체가 아니거나, `TProp`이 유효한 키가 아니면 `never` 반환 |

✅ **최종 예제**
```ts
type Obj = {
  user: {
    address: {
      street: string;
    };
    age: number;
  };
};

type A = DeepValue<Obj, "user.address.street">;  // string
type B = DeepValue<Obj, "user.age">;            // number
type C = DeepValue<Obj, "user.name">;           // never (name 키 없음)
```

### **💡 핵심 요약**
✔ **재귀적 구조**  
✔ **점 표기법(`.`)을 기준으로 분해**  
✔ **유효한 키가 없으면 `never` 반환**  
✔ **객체(`Record<string | number, any>`)에서만 동작**  