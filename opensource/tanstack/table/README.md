# 개요

[tanstack-table 오픈소스](https://github.com/TanStack/table) 분석

react-table에 집중할 예정

현재 버전인 v8과 main 브랜치를 기준으로 함

# 사용 스택
- nx
- vite
- pnpm
- testing-library
- typescript

<br />

# 폴더 구조

- nx를 이용한 모노레포 구조
- `packages` 폴더 하위에 주요 코드들이 위치
  

## scripts

 types.d.ts 파일을 제외하고, 모두 js 파일이지만 @ts-check을 통해 타입 체크 활성화 되도록 함

- `config.js`
  - 일관된 배포를 위한 중앙 설정 관리 파일
  - publish.js 에서 이 파일의 설정 기반으로 배포
  - `import('./types').Package[]` 타입의 `packages` 배열
    - npm 패키지가 나열된 배열
    - 배열의 0번째(table-core)를 기준으로 버전이 관리됨
  - 브랜치 배포 여부를 설정할 수 있는 `branchConfigs` 객체
    - main 외 모두 `prerelease` 값이 true
- `getRollupConfig.js`
  - (작성중)
- `publish.js`
  - [@tanstack/config](https://github.com/TanStack/config)의 publish 함수 실행시키는 파일
  - ES2022의 [Top-level-await](https://v8.dev/features/top-level-await) 사용됨

## packages
폴더 하위에는 table-core 을 포함, 각 스택에 맞는 폴더 존재

- Ex 1) 스택 별 : react-table, vue-table 등
- Ex 2) 용도 별 : react-table-devtools, soild-table 등

여기서는 table-core, react-table 위주로 분석할 예정이며, 필요에 따라 분석 폴더가 추가될 수 있음

분석 대상은 별도의 하위 폴더로 관리될 예정

### `/table-core`
- 셀 생성, 정렬, 확장, 페이지네이션 등 테이블의 기능에 직결되는 유틸 폴더

- `/core` : cell, column, headers, row, table의 각 create 함수를 포함하는 폴더
- `/feature` : 컬럼 및 행과 관련된 정렬, 필터링, 페이징, 고정 등 다양한 기능을 제공하는 폴더
- `/utills` : 데이터의 필터링, 정렬, 그룹화, 페이징 및 팩셋 처리를 위한 유틸리티 함수들을 포함하는 폴더
- `/aggregationFns.ts` : 집합과 관련된 함수들
- `/columnHelper.ts` : 테이블 컬럼을 정의하고 접근할 수 있도록 도와주는 유틸리티 함수 제공
- `/aggregationFns.ts` : 집합과 관련된 함수들
- `/filterFns.ts` : 단어 포함 여부 확인 함수, 배열 포함 여부 확인 함수 등 필터링과 관련된 함수들
- `/index.ts` : 핵심 모듈, 유틸리티 등을 내보내는 역할
- `/sortingFns.ts` : 알파벳 순, 시간 순 등 정렬 방식과 관련된 함수들
- `/types.ts` : (작성중)
- `/utils.ts` : 타입을 가공하기 위한 유틸 타입들
### `/react-table/src/index.tsx`
- 테이블을 생성하고 옵션을 설정, 해당 객체를 반환하는 커스텀훅과 이를 보조하는 함수가 함께 위치