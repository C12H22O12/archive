# CSS 라이브러리에 관하여

taliwind, styled-component, emotion 등등 세상에는 CSS를 간편히 하기 위한 수많은 노력들이 코드의 형체로 존재한다. 선택지가 많으면 고민하는 시간도 길어진다. 수많은 라이브러리들의 특징과 장단점을 파악하기 위해 이번 페이지를 열었다.

# CSS-in-JS

## 런타임 CSS-in-JS

어플이 실행될 때 라이브러리가 스타일을 해석 및 적용하는 CSS-in-JS를 뜻한다. 

#### 대표 라이브러리
- [styled-component](https://styled-components.com/)
- [emotion](https://emotion.sh/docs/introduction)

### 장점
1. 지역 스코프 스타일
   - 일반 CSS를 사용할 경우, 클래스 이름 충돌의 문제 존재
   - 그러나 CSS-in-JS는 기본적으로 스타일을 지역 스코프 단위로 지정하여 문제를 해결
2. 배치
    - 컴포넌트 단위로 모듈화할 수 있고, 컴포넌트 단위로 유지보수할 수 있음
    - 직관적으로 컴포넌트를 관리할 수 있고, 컴포넌트 별 단일 책임 원칙을 지킬 수 있음
3. 편한 동적 스타일링
    - 변수, react state, react props 등을 이용하여 인라인 스타일을 더하지 않고 동적 스타일링을 간단하게 할 수 있음
    - 인라인 스타일의 경우 동일한 스타일이 여러 곳에 적용되면 성능 상 좋지 못하나, CSS-in-JS는 이를 사용하지 않고도 같은 스타일을 여러 곳에 적용할 수 있음

### 단점
1. 런타임 오버헤드 증가
    - 렌더링 시점에 스타일 생성 및 주입
    - 컴포넌트가 렌더링될 때 CSS-in-JS 라이브러리는 CSS 스타일을 Document(`❇︎ HTML으로 판단`)에 삽입할 수 있도록 변환, 이 과정에서 CPU 소모(* 첨언 필요)
2. JS 번들 크기 증가
    - 스타일이 JS 파일 내에 포함
    - 사이트 방문자는 CSS-in-JS 라이브러리를 다운 받아야 함


### 참고

- [(번역) 우리가 CSS-in-JS와 헤어지는 이유](https://junghan92.medium.com/%EB%B2%88%EC%97%AD-%EC%9A%B0%EB%A6%AC%EA%B0%80-css-in-js%EC%99%80-%ED%97%A4%EC%96%B4%EC%A7%80%EB%8A%94-%EC%9D%B4%EC%9C%A0-a2e726d6ace6)
- [styled components vs tailwind css 어떤 것을 사용해야 할까? - Elancer 개발 블로그](https://www.elancer.co.kr/blog/detail/290)
