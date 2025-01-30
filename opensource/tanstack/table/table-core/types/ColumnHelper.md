# ColumnHelper

``` Typescript
export type ColumnHelper<TData extends RowData> = {
  accessor: <
    TAccessor extends AccessorFn<TData> | DeepKeys<TData>,
    TValue extends TAccessor extends AccessorFn<TData, infer TReturn>
      ? TReturn
      : TAccessor extends DeepKeys<TData>
        ? DeepValue<TData, TAccessor>
        : never,
  >(
    accessor: TAccessor,
    column: TAccessor extends AccessorFn<TData>
      ? DisplayColumnDef<TData, TValue>
      : IdentifiedColumnDef<TData, TValue>
  ) => TAccessor extends AccessorFn<TData>
    ? AccessorFnColumnDef<TData, TValue>
    : AccessorKeyColumnDef<TData, TValue>
  display: (column: DisplayColumnDef<TData>) => DisplayColumnDef<TData, unknown>
  group: (column: GroupColumnDef<TData>) => GroupColumnDef<TData, unknown>
```

- `/table-core/src/columnHelper.ts` 에 위치한 코드
- `accessor`
  - 제네릭 파라미터
    1. `TAccessor` : 아래 두 값 중 하나를 가짐
        - Accessory<TData>를 상속받은 TAccessor
        - DeepKeys<TData>의 값을 가지는 제네릭 파라미터
    2. `TValue`
        1. TAccessor가 AccessorFn<TData, 할당된 타입>일 경우 `할당된 타입`이 TReturn 타입이 되고, TValue에는 TReturn의 값을 가짐
        2. 2-1의 조건을 만족하지 못하면, TAccessor가 DeepKeys<TData>인 지 확인
        3. 2-2의 조건을 충족하면 TValue는 DeepVlaue<TData, TAccessor> 타입이 되고, 아닐 경우 never가 할당
  - 함수 파라미터
    - `accessor` : 제네릭 파라미터 TAccessor을 따름
    - `column` : 제네릭 파라미터 TAccessor의 형태가 AccessorFn<TData>이면 DisplayColumnDef<TData, TValue>를, 아닐 경우 IdentifiedColumnDef<TData, TValue>의 타입을 가짐
  - 함수 반환값
    - 제네릭 파라미터 TAccessor의 형태가 AccessorFn<TData>이면 AccessorFnColumnDef<TData, TValue>를, 아닐 경우 AccessorKeyColumnDef<TData, TValue>의 타입을 가짐