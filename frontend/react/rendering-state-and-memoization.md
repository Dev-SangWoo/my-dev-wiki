# React 렌더링, 상태 위치와 메모이제이션

## Current Understanding

1. 리액트에서 컴포넌트는 UI 그 자체가 아니라 그 UI를 만드는 함수이다.

2. props나 state, context 등 이미 존재하는 값으로 계산하여 만들 수 있는 값은 또 state에 저장하지 않는다. 이를 Single Source of Truth와 연결해서 이해한다.

3. 컴포넌트가 리렌더링되는 대표적인 경우는 다음 세 가지로 이해한다.
   - 자신의 state가 바뀌었을 때
   - 부모 컴포넌트가 리렌더링되었을 때
   - 내가 구독하고 있는 context가 바뀌었을 때

4. Reconciliation은 리렌더링 과정에서 이전 결과와 새로운 결과를 비교하는 과정으로 이해한다. 학습할 때는 다음 흐름으로 기억한다.

```text
State Update
   ↓
Render
   ↓
Reconciliation
   ↓
Commit
   ↓
Browser Layout / Paint
```

Reconciliation은 별도의 완전히 독립된 단계라기보다 Render 과정에서 이전/새 React 트리를 비교하는 작업으로 보는 편이 더 정확하다.

5. state의 위치에 따라 리렌더링되는 요소가 달라진다.

자주 변경되는 state를 너무 높은 컴포넌트가 소유하면, 그 컴포넌트가 리렌더링될 때 그 아래의 관련 없는 자식 컴포넌트도 기본적으로 다시 렌더링될 수 있다.

그러므로 그걸 쪼개 버리면 불필요한 계산을 막을 수 있다. 즉 구조적인 문제를 캐싱으로 가리는 것보다 상태 범위를 줄이는 것이 먼저다.

```text
1순위: 상태 위치 조정
2순위: 컴포넌트 책임 분리
3순위: 데이터 구조 개선
4순위: 필요한 경우 memoization
```

6. 데이터의 종류를 다음처럼 구분한다.
   - Local State
   - Server State
   - URL State
   - Global State
   - Form State
   - Derived Value

7. `React.memo`는 부모가 리렌더링되더라도 전달받은 props가 이전과 같으면 컴포넌트의 리렌더링을 건너뛸 수 있게 하는 최적화다. 비용이 있어서 필요한 곳에만 적용해야 한다. 컴포넌트 자신의 state가 바뀌면 다시 리렌더링된다.

8. `useMemo`는 기억할 때는 `React.memo`의 값 버전처럼 연결해서 생각한다. 정확히는 dependency가 바뀌지 않았다면 이전 계산 결과를 재사용하는 최적화다.

```text
계산이 실제로 느린가?
      ↓
Profiler나 Performance로 확인했는가?
      ↓
의존 값이 자주 변경되는가?
      ↓
캐싱 비용보다 계산 비용이 큰가?
      ↓
그렇다면 useMemo 고려
```

9. Profiler는 React 렌더링 분석에 특화되어 있고, Performance는 JS 실행부터 브라우저 렌더링까지 더 넓고 낮은 레벨에서 본다.

## Open Questions

- 실제 프로젝트에서 state 위치 조정만으로 해결되는 병목과 `React.memo`/`useMemo`가 필요한 병목을 어떻게 구분할지 더 경험이 필요하다.

## Understanding Timeline

- 2026-09-08: React 컴포넌트를 UI 자체가 아니라 UI를 계산하는 함수로 이해하기 시작했다.
- 2026-09-08: 계산 가능한 값을 별도 state로 저장하지 않는 원칙을 Single Source of Truth와 연결했다.
- 2026-09-08: 성능 문제를 보면 먼저 memoization을 붙이기보다 state 위치와 컴포넌트 구조를 먼저 본다는 관점을 형성했다.
- 2026-09-08: `React.memo`를 단순히 "내 값이 바뀔 때만 리렌더링"하는 기능으로 보던 이해를, 부모 리렌더링 시 props가 같으면 렌더링을 건너뛰는 최적화로 교정했다.
- 2026-09-08: Reconciliation을 리렌더링 원인이 아니라 이전/새 결과를 비교하는 과정으로 분리해서 이해했다.

## Connections

현재 저장소에 직접 연결할 기존 지식 파일 없음.

## Sources

- 2026-09-08 대화에서 사용자가 제공한 React 렌더링/상태 학습 자료
