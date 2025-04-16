# `useInterval` 구현

## 개요
[면접자/면접관 화면 상단의 초시계를 커스텀 훅과 Ref를 사용하여 구현(useInterval)](https://github.com/C12H22O12/PITCHIT/blob/main/frontend/src/action/hooks/useInterval.js)의 내용을 셀프 리뷰하는 글입니다. 

> 참고 | [번역 / 리액트 훅스 컴포넌트에서 setInterval 사용 시의 문제점](https://velog.io/@jakeseo_me/%EB%B2%88%EC%97%AD-%EB%A6%AC%EC%95%A1%ED%8A%B8-%ED%9B%85%EC%8A%A4-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EC%97%90%EC%84%9C-setInterval-%EC%82%AC%EC%9A%A9-%EC%8B%9C%EC%9D%98-%EB%AC%B8%EC%A0%9C%EC%A0%90)<br />
> 원본 글 | [dan abramov의 Making setInterval Declarative with React Hooks](https://overreacted.io/making-setinterval-declarative-with-react-hooks/)

<br />
<br />
<br />

## 리뷰하기 전

해당 컴포넌트는 화면에 보여지는 초시계에서, 초를 세는 데 사용되는 커스텀훅입니다. 구현은 했지만 프로젝트에서 사용되진 않았습니다.

얼마 전에 본 과제에서 초시계를 구현하는 내용이 있었습니다. 빠르게 쳐내야 하기에, 그리고 익숙한 방법으로 죄책감 없이 `useState`를 사용했습니다. 그러나 면접에서 "매번 state를 변경하게 된다면 그 횟수만큼 재렌더링이 일어날텐데, 어떻게 해결할 수 있을까요?"라는 질문을 받았습니다. 대답을 하지 못했습니다. 답을 찾는 과정 중 이전에 제가 이를 토대로 코드를 작성했고, 제대로 활용하지 못했다는 점도 함께 알게 되었습니다. 반성을 하는 마음으로 참고 글을 다시 복기하고 분석해보려 합니다.

참고로, 해당 글은 2019년에 작성되었습니다. 현재는 2025년이죠. 이 빠른 프론트엔드의 흐름 사이에 어쩌면 이 글은 빛이 바랬을 지도 모릅니다. 그래서, 먼저 글을 분석하고 현재는 어떻게 달라졌는지 확인해보도록 하겠습니다.

<br />
<br />
<br />

## 최종 코드

``` js
import { useEffect, useRef } from 'react';

//총 시간 흘러가게 하는 훅
function useInterval(callback, delay) {
  const savedCallback = useRef();

  // Remember the latest callback.
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  // Set up the interval.
  useEffect(() => {
    function tick() {
      savedCallback.current();
    }

    if (delay !== null) {
      let id = setInterval(tick, delay);
      
      return () => clearInterval(id);
    }
  }, [delay]);
}

export default useInterval
```

<br />
<br />
<br />

## 리액트 훅과 setInterval

`setInterval`은 두 번째 인자로 받은 밀리초마다 첫 번째 인자의 함수를 반복 실행합니다.