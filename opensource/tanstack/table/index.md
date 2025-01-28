# 사용 스택
- nx
- vite
- pnpm
- testing-library
- typescript

<br />

# 폴더 구조

- nx를 이용한 모노레포 구조

## scripts

 types.d.ts 파일을 제외하고, 모두 js 파일이지만 @ts-check을 통해 타입 체크 활성화 되도록 함

- config.js
  - 일관된 배포를 위한 중앙 설정 관리 파일
  - publish.js 에서 이 파일의 설정 기반으로 배포
  - `import('./types').Package[]` 타입의 `packages` 배열
    - npm 패키지가 나열된 배열
    - 배열의 0번째(table-core)를 기준으로 버전이 관리됨
  - 브랜치 배포 여부를 설정할 수 있는 `branchConfigs` 객체
    - main 외 모두 `prerelease` 값이 true
- getRollupConfig.js
  - 
- publish.js
  - [@tanstack/config](https://github.com/TanStack/config)의 publish 함수 실행시키는 파일
  - ES2022의 [Top-level-await](https://v8.dev/features/top-level-await) 사용됨

## packages
폴더 하위에는 table-core 을 포함, 각 스택에 맞는 폴더 존재

- Ex 1) 스택 별 : react-table, vue-table 등
- Ex 2) 용도 별 : react-table-devtools, soild-table 등

여기서는 table-core, react-table 위주로 분석할 예정이며, 필요에 따라 분석 폴더가 추가될 수 있음

분석 대상은 별도의 하위 폴더로 관리될 예정
