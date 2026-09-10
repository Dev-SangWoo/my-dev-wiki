# React 렌더링과 상태 위치

## Current Understanding

1. 리액트에서 컴포넌트는 UI 그 자체가 아니라 그 UI를 만드는 함수이다.

2. props나 state, context 등 이미 존재하는 값으로 계산하여 만들 수 있는 값은 또 state에 저장하지 않는다. 이를 Single Source of Truth와 연결해서 이해한다.

3. 컴포넌트가 리렌더링되는 대표적인 경우는 다음 세 가지로 이해한다.
   - 자신의 state가 바뀌었을 때
   - 부모 컴포넌트가 리렌더링되었을 때
   - 내가 구독하고 있는 context가 바뀌었을 때

4. 렌더링 흐름을 `새 React 트리를 만든다 → 이전 트리와 비교한다 → 실제 반영한다`로 연결해서 이해한다.

5. Reconciliation은 리렌더링의 원인이 아니라 이전 결과와 새로운 결과를 비교하는 행위다. 더 정확히는 Render Phase 안에서 이전/새 React 트리를 비교하는 작업으로 이해한다.

```text
React 업데이트
   ↓
Render Phase
   ├─ 컴포넌트 실행
   ├─ 새 UI 결과 계산
   └─ Reconciliation
      이전 트리 ↔ 새 트리 비교
   ↓
Commit Phase
   └─ 필요한 DOM 변경 반영
```

6. state의 위치에 따라 리렌더링되는 요소가 달라진다. 자주 변경되는 state를 너무 높은 컴포넌트가 소유하면 그 아래의 관련 없는 자식 컴포넌트도 기본적으로 다시 렌더링될 수 있다.

그러므로 구조적인 문제를 캐싱으로 가리는 것보다 상태 범위를 줄이는 것이 먼저다.

```text
1순위: 상태 위치 조정
2순위: 컴포넌트 책임 분리
3순위: 데이터 구조 개선
4순위: 필요한 경우 memoization
```

7. 데이터의 종류를 다음처럼 구분한다.
   - Local State
   - Server State
   - URL State
   - Global State
   - Form State
   - Derived Value

## Quick Recall

```text
Render Phase → 새 UI 계산 + 이전/새 트리 비교
Reconciliation → 이전/새 React 트리를 비교하는 작업
Commit Phase → 필요한 DOM 변경 반영
```

## Open Questions

없음.

## Understanding Timeline

- 2026-09-08: React 컴포넌트를 UI 자체가 아니라 UI를 계산하는 함수로 이해하기 시작했다.
- 2026-09-08: 계산 가능한 값을 별도 state로 저장하지 않는 원칙을 Single Source of Truth와 연결했다.
- 2026-09-08: 성능 문제를 보면 먼저 memoization을 붙이기보다 state 위치와 컴포넌트 구조를 먼저 본다는 관점을 형성했다.
- 2026-09-08: Reconciliation을 리렌더링 원인이 아니라 이전/새 결과를 비교하는 과정으로 분리해서 이해했다.
- 2026-09-09: `Render → Reconciliation → Commit`을 완전히 독립된 세 단계로 보기보다, Render Phase 안에서 새 UI 계산과 Reconciliation이 이루어지고 이후 Commit Phase에서 실제 변경을 반영하는 흐름으로 이해를 다듬었다.

## Connections

- [React 상태 저장 위치 결정 플로우](state-location-decision-flow.md) — 상태를 사용하는 곳 가까이에 둘수록 불필요한 리렌더링 범위를 줄일 수 있다는 이해와 연결된다.
- [React 메모이제이션과 참조 동일성](memoization-and-reference-equality.md) — 구조와 상태 위치를 먼저 정리한 뒤 필요한 경우 렌더링/계산 최적화를 적용한다는 흐름으로 연결된다.

## Sources

- 2026-09-08 대화에서 사용자가 제공한 React 렌더링/상태 학습 자료
- 2026-09-09 React 렌더링과 메모이제이션 복습 대화
