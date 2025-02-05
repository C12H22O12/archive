# 타입 분석

이미지 업로드 기준일 : 2025.02.04

[figjam](https://www.figma.com/board/0R4jnf1gv5yP8AsWMp7ACL/Untitled?node-id=0-1&t=ylFlYPbxjI7wgMOA-1)을 통해 table-core 폴더 내에 선언된 타입 분석

### 비고
- `types.ts`파일 타입 모두 추가 및 기능 확인
- 상속받아 요소를 구성하는 인터페이스가 다수고, 타 파일로부터 import를 다수 받는`types.ts` 파일의 특성상 별도로 화살표 연결은 생략함
- 이후 다른 파일 정리하는 과정에서 review 정리할 예정

<img width="744" alt="스크린샷 2025-02-04 오후 9 02 03" src="https://github.com/user-attachments/assets/31b78b44-6824-4d94-96f8-664a34017543" />

## 목적

- 타입 분석을 통해 파일의 목적과 관계를 알기 위함
- 생소한 타입 사용 용례 파악 및 정리 -> 별도 md 파일로 정리

## 파일 별 목적 요약

- `utills.ts`
  - 타입을 가공하기 위한 유틸 타입들이 주로 있다(`IsAny`, `Index40` ...)
  - 선언만 한 타입도 많고, 사용성이 높지 않은 타입이 많다.
  - 범용성이 높아야 하기에 타입들이 복잡하다.

