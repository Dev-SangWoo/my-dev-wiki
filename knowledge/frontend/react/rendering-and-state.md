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

8. `setState`는 DOM을 직접 수정하는 명령이 아니라 React에게 상태가 바뀌었으니 UI를 다시 계산하라고 요청하는 것으로 이해한다.

```text
setState
≠ DOM 직접 변경

setState
→ state update 요청
→ 컴포넌트 다시 실행
→ 새 UI 계산
→ 이전 결과와 비교
→ 필요한 DOM 변경만 Commit
```

그래서 React가 관리하는 UI를 일반적으로 직접 DOM 조작과 섞어 관리하지 않는다. React가 알고 있는 UI 상태와 실제 DOM 상태가 서로 어긋날 수 있기 때문이다.

9. Context도 값이 변경되면 그 Context를 구독하는 컴포넌트들의 렌더링에 영향을 줄 수 있다고 이해한다. 하나의 Context에 변경 빈도와 책임이 다른 값을 너무 많이 묶으면 예상보다 넓은 범위가 영향을 받을 수 있다.

```text
Context value 변경
↓
해당 Context 소비자에게 영향
```

그래서 `Context를 쓰면 느리다`가 아니라, **변경 빈도와 책임이 다른 상태를 무조건 하나의 Context로 묶지 않는 것**이 중요하다고 이해한다.

```text
ThemeContext
AuthContext
ModalContext
```

처럼 책임과 변경 범위를 나누는 것을 고려할 수 있다.

State Colocation과 Context 설계 모두 결국 다음 질문과 연결된다.

```text
이 값이 바뀔 때 어디까지 영향을 받아야 하지?
```

## Quick Recall

```text
setState → DOM 직접 변경이 아니라 React에 UI 재계산 요청
Render Phase → 새 UI 계산 + 이전/새 트리 비교
Reconciliation → 이전/새 React 트리를 비교하는 작업
Commit Phase → 필요한 DOM 변경 반영
```

```text
Context
→ 구독하는 값이 바뀌면 소비자가 영향을 받을 수 있음
→ 책임과 변경 빈도가 다르면 범위를 나눌지 고려
```

## Open Questions

없음.

## Understanding Timeline

- 2026-09-08: React 컴포넌트를 UI 자체가 아니라 UI를 계산하는 함수로 이해하기 시작했다.
- 2026-09-08: 계산 가능한 값을 별도 state로 저장하지 않는 원칙을 Single Source of Truth와 연결했다.
- 2026-09-08: 성능 문제를 보면 먼저 memoization을 붙이기보다 state 위치와 컴포넌트 구조를 먼저 본다는 관점을 형성했다.
- 2026-09-08: Reconciliation을 리렌더링 원인이 아니라 이전/새 결과를 비교하는 과정으로 분리해서 이해했다.
- 2026-09-09: `Render → Reconciliation → Commit`을 완전히 독립된 세 단계로 보기보다, Render Phase 안에서 새 UI 계산과 Reconciliation이 이루어지고 이후 Commit Phase에서 실제 변경을 반영하는 흐름으로 이해를 다듬었다.
- 2026-09-10: `setState`를 DOM 직접 변경이 아니라 React에 재계산을 요청하는 업데이트로 연결하고, Context도 변경 범위와 책임을 고려해 설계해야 한다는 이해를 보강했다.

## Connections

- [React 상태 저장 위치 결정 플로우](state-location-decision-flow.md) — 상태를 사용하는 곳 가까이에 둘수록 불필요한 리렌더링 범위를 줄일 수 있다는 이해와 연결된다.
- [React 메모이제이션과 참조 동일성](memoization-and-reference-equality.md) — 구조와 상태 위치를 먼저 정리한 뒤 필요한 경우 렌더링/계산 최적화를 적용한다는 흐름으로 연결된다.

## Sources

- 2026-09-08 대화에서 사용자가 제공한 React 렌더링/상태 학습 자료
- 2026-09-09 React 렌더링과 메모이제이션 복습 대화
- 2026-09-10 1강 누락 개념 복습 및 이해 확인 대화
