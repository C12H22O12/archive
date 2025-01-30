# 질문 정리

### 1. UI는 어떻게 확인할 수 있는지? 
보통 localhost 주소로 확인했는데, 해당 파일은 빌드 시 확인할 수 있는 주소가 뜨지 않는다. 대신 다음과 같이 빌드 완료 여부가 여럿 뜸

----


### 2. rollup.config.mjs의 목적은 무엇인가?
/packages 하위 모든 폴더마다 해당 파일이 존재

----


### 3. /scripts 폴더는 정확히 어떤 목적과 성격을 가지고 있는가?
/scripts 폴더 하위에는 다음 파일들 존재
- config.js
- getRollupConfig.js
- publich.js
- types.d.ts

각 파일의 목적 파악 후에 폴더 성격을 확인할 수 있을 것으로 기대

----

### 4. /packages/table-core/types.ts에서 `TableMeta` 부분은  제네릭 부분의 색이 나타나지 않고, 바로 하단의 `ColumnMeta`은 제네릭 부분에 밑줄이 쳐져 있고 'All type parameters are unused.' 라는 경고 표시가 뜬다. 왜일까?


----

### 5. ColumnFaceting 파일의 FacetedColumn 타입
```ts
export interface FacetedColumn<TData extends RowData> {
  ...
  /**
   * A function that **computes and returns** a min/max tuple derived from `column.getFacetedRowModel`. Useful for displaying faceted result values.
   * > ⚠️ Requires that you pass a valid `getFacetedMinMaxValues` function to `options.getFacetedMinMaxValues`. A default implementation is provided via the exported `getFacetedMinMaxValues` function.
   * @link [API Docs](https://tanstack.com/table/v8/docs/api/features/column-faceting#getfacetedminmaxvalues)
   * @link [Guide](https://tanstack.com/table/v8/docs/guide/column-faceting)
   */
  getFacetedMinMaxValues: () => undefined | [number, number]
  /**
   * Returns the row model with all other column filters applied, excluding its own filter. Useful for displaying faceted result counts.
   * > ⚠️ Requires that you pass a valid `getFacetedRowModel` function to `options.facetedRowModel`. A default implementation is provided via the exported `getFacetedRowModel` function.
   * @link [API Docs](https://tanstack.com/table/v8/docs/api/features/column-faceting#getfacetedrowmodel)
   * @link [Guide](https://tanstack.com/table/v8/docs/guide/column-faceting)
   */
  getFacetedRowModel: () => RowModel<TData>
  /**
   * A function that **computes and returns** a `Map` of unique values and their occurrences derived from `column.getFacetedRowModel`. Useful for displaying faceted result values.
   * > ⚠️ Requires that you pass a valid `getFacetedUniqueValues` function to `options.getFacetedUniqueValues`. A default implementation is provided via the exported `getFacetedUniqueValues` function.
   * @link [API Docs](https://tanstack.com/table/v8/docs/api/features/column-faceting#getfaceteduniquevalues)
   * @link [Guide](https://tanstack.com/table/v8/docs/guide/column-faceting)
   */
  ...
}
```

유효한 함수를 전달해야 한다는 주석이 여럿 달려있다. 여기서 '유효한' 함수는 무엇인가?