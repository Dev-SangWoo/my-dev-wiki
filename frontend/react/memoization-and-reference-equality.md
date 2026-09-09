# React 메모이제이션과 참조 동일성

## Current Understanding

1. `React.memo`는 부모가 리렌더링될 때 이전 props와 새 props를 비교해서, 같으면 자식 컴포넌트의 렌더링을 건너뛰는 최적화로 이해한다. 다만 자식 자신의 state가 바뀌거나 구독 중인 context가 바뀌면 다시 렌더링된다.

2. `React.memo`는 얕은 비교를 하기 때문에 객체, 배열, 함수처럼 렌더링 때 새로 만들어지는 값은 내용이 같아도 참조가 다를 수 있다고 이해한다. 그래서 새 객체나 새 함수가 props로 내려오면 `React.memo`가 props 변경으로 볼 수 있다.

3. 객체나 함수가 새 참조가 되는 것은 `React.memo`가 복사해서가 아니라, 부모 컴포넌트가 다시 실행되면서 객체 리터럴이나 함수 표현식이 다시 만들어지기 때문이다.

4. `useMemo`는 dependency가 같으면 이전에 계산한 값을 계속 재사용하는 것으로 이해한다. 컴포넌트가 리렌더링되더라도 비싼 계산을 다시 하지 않게 할 수 있고, 객체나 배열의 참조를 안정적으로 유지하는 데도 쓸 수 있다.

5. `useCallback`은 dependency가 같으면 이전 함수 참조를 재사용하는 것으로 이해한다. dependency가 바뀌면 새로운 함수 참조가 만들어진다. `useCallback` 자체는 비싼 계산을 줄이는 기능이라기보다 함수 참조를 유지하는 역할에 가깝다.

6. `React.memo`, `useMemo`, `useCallback`은 레벨이 다른 최적화라고 이해한다. `React.memo`는 컴포넌트 렌더링을 건너뛸지 판단하고, `useMemo`와 `useCallback`은 값이나 함수 참조를 안정적으로 유지하는 데 쓸 수 있어서 같이 사용되기도 한다.

7. `React.memo`가 없다면 `useMemo`나 `useCallback`만으로 부모 리렌더링에 따른 자식 리렌더링 자체를 막는 것은 아니다. 다만 `useMemo`는 렌더링 중의 불필요한 재계산을 줄일 수 있고, `useCallback`은 함수 참조를 안정적으로 유지해 다른 최적화나 dependency 관리에 활용할 수 있다.

8. 메모이제이션은 무조건 붙이는 기능이 아니라 실제로 불필요한 렌더링이나 비싼 계산이 문제가 될 때 고려한다. 구조적인 문제라면 먼저 상태 위치와 컴포넌트 책임을 본다.

## Quick Recall

```text
React.memo  → props가 같으면 자식 컴포넌트 렌더링을 건너뛸 수 있음
useMemo     → dependency가 같으면 이전 계산값 재사용
useCallback → dependency가 같으면 이전 함수 참조 재사용
```

```text
React.memo  → 컴포넌트 레벨
useMemo     → 값/계산 레벨
useCallback → 함수 참조 레벨
```

```text
새 객체/배열/함수
→ 내용이 같아도 참조가 다를 수 있음
→ React.memo의 얕은 비교에서 props 변경으로 판단될 수 있음
```

## Open Questions

없음.

## Understanding Timeline

- 2026-09-08: `React.memo`를 단순히 "내 값이 바뀔 때만 리렌더링"하는 기능으로 보던 이해를, 부모 리렌더링 시 props가 같으면 렌더링을 건너뛰는 최적화로 교정했다.
- 2026-09-09: `React.memo`의 얕은 비교와 객체/배열/함수의 참조 동일성을 연결했다.
- 2026-09-09: `useMemo`는 값, `useCallback`은 함수 참조를 재사용하며, 이들이 `React.memo`와 서로 다른 레벨에서 협력할 수 있다는 관계를 이해했다.
- 2026-09-09: `React.memo`가 없어도 `useMemo`는 렌더링 중 재계산을 줄일 수 있지만 자식 리렌더링 자체를 막지는 못하며, `useCallback`은 주로 함수 참조 안정화를 위한 것이라고 구분했다.

## Connections

- [React 렌더링과 상태 위치](rendering-and-state.md) — 메모이제이션을 적용하기 전에 렌더링 범위와 상태 위치를 먼저 보는 관점과 연결된다.

## Sources

- 2026-09-08 대화에서 사용자가 제공한 React 렌더링/상태 학습 자료
- 2026-09-09 React 렌더링과 메모이제이션 복습 대화
