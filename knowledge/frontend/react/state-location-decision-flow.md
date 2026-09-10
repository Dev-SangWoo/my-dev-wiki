# React 상태 저장 위치 결정 플로우

## Current Understanding

## 데이터를 어디에 보관할지 생각하는 플로우

```text
1. 이미 있는 값으로 계산 가능한가?
   → Derived Value
   → state로 저장하지 않는다.

2. 서버가 원본을 가지고 있는가?
   → Server State

3. 새로고침, 뒤로가기, 링크 공유가 필요한 화면 정보인가?
   → URL State

4. 특정 컴포넌트에서만 사용하는가?
   → Local State

5. 형제 컴포넌트끼리 같이 사용하는가?
   → 가장 가까운 공통 부모로 올린다.
   → Lifting State Up

6. 멀리 떨어진 여러 영역이나 앱 전반에서 공유하는
   클라이언트 소유 상태인가?
   → Global State 후보
```

짧게 외우면:

```text
계산 가능? → Derived
서버 소유? → Server
URL에 남아야 해? → URL
나만 써? → Local
형제끼리 써? → Lift Up
여기저기 같이 써? → Global 후보
```

### 중요한 기준

- 여러 컴포넌트가 사용한다고 무조건 Global State는 아니다.
- 형제가 같이 사용하면 먼저 공통 부모로 상태를 올릴 수 있는지 본다.
- 서버 데이터라고 해서 일반 `useState`로 관리하는 것이 최선은 아니다.
- 계산 가능한 값은 state로 중복 저장하지 않는다.
- URL로 표현되어야 하는 화면 조건은 URL에 둔다.
- 상태는 가능한 한 실제 사용하는 곳 가까이에 둔다.

## Open Questions

없음.

## Understanding Timeline

- 2026-09-09: 값을 보면 바로 `useState`를 떠올리기보다 계산 가능 여부, 서버 소유 여부, URL 필요 여부, 사용하는 범위를 순서대로 판단해 상태의 위치를 결정하는 흐름을 정리했다.

## Connections

- [React 렌더링과 상태 위치](rendering-and-state.md) — 상태를 사용하는 곳 가까이에 둘수록 불필요한 리렌더링 범위를 줄일 수 있다는 이해와 연결된다.
- [TanStack Query 서버 상태와 캐시 관리](tanstack-query-server-state-cache.md) — 서버가 원본인 데이터를 Server State로 판단한 뒤, 그 서버 데이터의 클라이언트 복사본을 캐싱하고 동기화하는 방법으로 이어진다.

## Sources

- 2026-09-09 React 상태 종류와 저장 위치를 구분하는 2강 학습 및 정리 대화
