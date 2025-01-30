# Record
``` ts
interface PageInfo {
  title: string;
}
 
type Page = "home" | "about" | "contact";
 
const nav: Record<Page, PageInfo> = {
  about: { title: "about" },
  contact: { title: "contact" },
  home: { title: "home" },
};
 
nav.about;
// const nav: Record<Page, PageInfo>
```

- `Record<Keys,Type>`와 같은 형태로 사용되며, 키가 Keys 타입, value가 Type인 객체 타입을 생성
- [공식 문서] 타입 Type의 프로퍼티 키의 집합으로 타입을 생성합니다. 이 유틸리티는 타입의 프로퍼티를 다른 타입에 매핑 시키는데 사용될 수 있습니다.

### 출처
- [타입스크립트 공식 문서 - Utility Types](https://www.typescriptlang.org/ko/docs/handbook/utility-types.html)