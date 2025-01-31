# DeepKeys
``` ts
type DeepKeysPrefix<T, TPrefix, TDepth extends any[]> = TPrefix extends keyof T & (number | string)
  ? `${TPrefix}.${DeepKeys<T[TPrefix], [...TDepth, any]> & string}`
  : never

export type DeepKeys<T, TDepth extends any[] = []> = TDepth['length'] extends 5
  ? never
  : unknown extends T
    ? string
    : T extends readonly any[] & IsTuple<T>
      ? AllowedIndexes<T> | DeepKeysPrefix<T, AllowedIndexes<T>, TDepth>
      : T extends any[]
        ? DeepKeys<T[number], [...TDepth, any]>
        : T extends Date
          ? never
          : T extends object
            ? (keyof T & string) | DeepKeysPrefix<T, keyof T, TDepth>
            : never
```

- `DeepKeyPrefix`와 순환 구조를 가짐
  - 객체의 key들을 string 형식으로 조합함
  - 객체의 속성들을 .으로 연결된 문자열 형태로 변환
  - 동작 구조 
    1. TPrefix는 객체 T의 Key이면서 number 혹은 string 타입인 지 먼저 검사
    2. 참이라면 [템플릿 리터럴](https://toss.tech/article/template-literal-types)을 통해 깊은 탐색 시작
        - DeepKeys의 두번째 제네릭 파라미터에 any가 들어간 배열을 넣음으로써, 재귀 종료 조건 삽입
        - DeepKeys<T[TPrefix], [...TDepth, any]> 이 유니온을 포함할 수 있으므로 유효 문자열 타입으로 변경
    3. 거짓이라면 never를 반환하고 재귀 종료
- `DeepKey`
  - 깊은 탐색을 하며 키의 값(string)들을 유니온으로 연결
  - Ex ) `type Example = DeepKeys<{ a: { b: { c: string }[] } }>;` 
    - `"a" | "a.b" | "a.b.[number]" | "a.b.[number].c"`
  1. 재귀 횟수가 5번 미만인가?(`TDepth['length'] extends 5`)
      - ⭕️ 재귀 종료(never 반환)
      - ❌ 2로 이동
  2. T의 타입이 unknown인가?
      - ⭕️ 재귀 종료(string 반환)
      - ❌ 3으로 이동
  3. T가 튜플이고, 그 길이가 40 이하인가?
      - ⭕️ T를 이용하여 다시 DeepKey 재귀 -> 키 값 추출
      - ❌ 4로 이동
  4. T가 Date 객체인가?
      - ⭕️ 재귀 종료(never 반환)
      - ❌ 5로 이동
  5. T가 객체인가?
      - ⭕️ T의 key와 그 내부의 값을 다시 탐색 -> DeepKeysPrefix
      - ❌ 재귀 종료(never 반환)


<br />
<br />
<br />

# 보충 - chatGPT 답변
## 위 타입에 대한 질문에 답변
## **📌 `DeepKeys<T, TDepth extends any[] = []>` 분석**

이 타입은 **객체 `T`의 깊숙한 키들을 문자열로 추출하는 타입 유틸리티**입니다.  
단, **최대 5단계 깊이까지만 탐색**하도록 제한됩니다.

---

### **1️⃣ `TDepth['length'] extends 5` → 재귀 제한 조건인가?**
✅ **네, 이 부분은 재귀 제한 조건입니다.**  

```ts
TDepth['length'] extends 5 ? never : ...
```
- `TDepth`는 `[]`에서 시작하며, 깊이가 하나 더 들어갈 때마다 `[...TDepth, any]`을 추가  
- 즉, `TDepth = [] → [any] → [any, any] → ...` 이런 식으로 길이가 증가  
- `TDepth['length']`가 `5`가 되면 `never`을 반환하여 **재귀를 멈춤**  

✔ **이 방식으로 무한 재귀를 방지함**.  
✔ 더 깊은 탐색을 원한다면 `5`를 `10` 같은 더 큰 값으로 변경 가능.

---

### **2️⃣ `unknown extends T` → `unknown`의 역할은?**
```ts
unknown extends T ? string : ...
```
✅ **`T`가 `unknown`이면 `string`을 반환하는 역할**  

#### 🔹 `unknown extends T`가 `true`가 되는 경우?
- `T`가 `unknown`이거나 `any`인 경우  
- 즉, **객체의 타입을 알 수 없을 때**  
- 이런 경우, `DeepKeys<T>`는 `"string"`을 반환 → **모든 키를 문자열로 가정**  

✔ **이 처리는 `T`가 `unknown`일 때, `string`으로 키 타입을 일반화하는 안전장치**.

---

### **3️⃣ `T extends readonly any[] & IsTuple<T>` → 중복되지 않는가?**
```ts
T extends readonly any[] & IsTuple<T>
```
✅ **`readonly any[]`과 `IsTuple<T>`가 중복되지 않음**  

#### 🔹 이유
1. `readonly any[]`
   - `T`가 배열(또는 튜플)인지 확인
2. `IsTuple<T>`
   - `T`가 "튜플인지" 확인 (`length`가 특정한 `Index40` 범위 내인지)

💡 `number[]`처럼 가변적인 배열은 `IsTuple<T>`를 통과하지 못하므로, **`number[]`과 튜플을 구별하는 데 필요**.  

✔ **즉, 이 조건은 "정확한 길이를 가진 튜플인지"를 확인하는 필수적인 과정이다.**  

---

### **4️⃣ `DeepKeysPrefix`와 순환 구조를 이루는 이유?**
✅ **`DeepKeys<T>`는 `DeepKeysPrefix<T, ...>`와 상호 호출되며 순환 구조를 가짐**.  

#### 🔹 **재귀적 키 탐색을 위한 구조**
1. `DeepKeys<T>`는 객체 `T`의 키를 탐색함  
2. `DeepKeysPrefix<T, keyof T, TDepth>`를 통해 **"접두사(prefix) 형태로 키를 조합"**  
3. `DeepKeysPrefix` 내부에서 다시 `DeepKeys<T>`를 호출 → **재귀적으로 탐색**  

✔ **이 구조 덕분에 `DeepKeys`는 객체의 중첩된 속성까지 탐색 가능**.  
✔ `TDepth`를 활용하여 **최대 5단계까지만 탐색하도록 제한**.

---

### **5️⃣ 전체적인 분석**
```ts
export type DeepKeys<T, TDepth extends any[] = []> =
  TDepth['length'] extends 5
    ? never   // 🔹 깊이 제한 (최대 5단계까지 탐색)
    : unknown extends T
      ? string  // 🔹 T가 unknown이면 모든 키를 string으로 처리
      : T extends readonly any[] & IsTuple<T>
        ? AllowedIndexes<T> | DeepKeysPrefix<T, AllowedIndexes<T>, TDepth> // 🔹 튜플이라면 가능한 인덱스와 내부 키 탐색
        : T extends any[]
          ? DeepKeys<T[number], [...TDepth, any]> // 🔹 일반 배열이면 요소 타입을 기반으로 탐색
          : T extends Date
            ? never // 🔹 Date 객체는 키가 없으므로 탐색하지 않음
            : T extends object
              ? (keyof T & string) | DeepKeysPrefix<T, keyof T, TDepth> // 🔹 객체라면 키와 내부 키 탐색
              : never // 🔹 나머지 경우는 키 탐색 불가
```

✅ **`DeepKeys<T>`의 역할:**  
- 객체 `T`의 **모든 깊숙한 키를 문자열(`string`)로 반환**  
- **최대 5단계까지만 탐색** (무한 재귀 방지)  
- `unknown`, `Date`, 배열, 튜플, 객체 등의 다양한 데이터 타입을 처리  

✔ **`DeepKeysPrefix`와 협력하여 "중첩된 키"를 추출하는 구조를 가짐**.  
✔ **배열(`number[]`)과 튜플(`IsTuple<T>`)을 구분하여 처리하는 로직을 포함**.  

---

## **📌 최종 정리**
| 조건 | 역할 |
|------|------|
| `TDepth['length'] extends 5` | **재귀 깊이를 5로 제한** (무한 재귀 방지) |
| `unknown extends T` | `T`가 `unknown`이면 **모든 키를 `string`으로 반환** |
| `T extends readonly any[] & IsTuple<T>` | **튜플이면 가능한 인덱스 + 내부 키 탐색** |
| `T extends any[]` | **일반 배열이면 요소 타입을 기반으로 재귀 탐색** |
| `T extends Date` | **Date는 키가 없으므로 `never` 반환** |
| `T extends object` | **객체이면 `keyof T`와 내부 키를 탐색** |

---

✔ **이 타입은 객체, 배열, 튜플을 모두 다루면서 깊숙한 속성의 키를 추출하는 강력한 타입 유틸리티이다.**  
✔ `TDepth['length'] extends 5`를 활용하여 **재귀 깊이를 제한하여 무한 재귀를 방지**.  
✔ `DeepKeysPrefix`와 **순환 구조를 이루어 중첩된 속성까지 탐색 가능**.  

---

✅ **결론:** `DeepKeys<T>`는 **객체의 모든 깊숙한 속성 키를 문자열로 추출하는 타입**으로, 배열과 튜플을 구분하며, 최대 5단계까지만 탐색하여 무한 재귀를 방지하는 설계가 적용됨.