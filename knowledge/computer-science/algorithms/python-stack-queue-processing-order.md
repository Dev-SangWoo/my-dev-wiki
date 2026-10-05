# Python Stack / Queue와 처리 순서

## Current Understanding

1. Stack은 **LIFO**로 기억한다. **과거 기록이나 뭐 이런 거에 쓰고, 뒤에꺼(나중에 들어온 거)가 나가야 하니까 pop을 쓴다**고 이해한다.

2. Queue는 **FIFO니까 먼저 들어온 거 나가야 하니까 popleft를 한다**고 이해한다.

3. Queue를 리스트로 구현하면서 `pop(0)`을 쓰면 **앞에 지우고 데이터들을 한 칸씩 땡겨야 해서 무거워지니까 popleft를 쓴다**고 이해한다. `deque`는 앞쪽 원소를 제거할 때 나머지 원소 전체를 한 칸씩 옮기지 않도록 설계된 구조로 이해한다.

4. 괄호 검사에서는 닫는 괄호가 나왔을 때 **가장 최근에 열린 괄호**, 즉 `stack[-1]`과 짝이 맞는지 확인해야 한다고 이해했다. 중간에 같은 괄호를 찾아 지우는 `remove()`가 아니라 마지막 값을 꺼내는 `pop()`이 Stack 동작에 맞는다.

5. 문자열의 연속된 같은 문자를 제거하는 문제에서도 현재 문자와 `stack[-1]`이 같으면 `pop()`, 다르면 `append()`하는 방식으로 한 번 순회해서 풀 수 있다고 이해했다.

## Quick Recall

```text
Stack → LIFO → append / pop → 최근 것부터
Queue → FIFO → append / popleft → 먼저 온 것부터
괄호 → stack[-1]로 가장 최근 열린 괄호 확인
list.pop(0) → O(N)
deque.popleft() → O(1)
```

## Open Questions

- Stack / Queue를 단순 명령 처리 말고 실제 알고리즘 문제에서 언제 먼저 떠올려야 하는지 문제를 더 풀면서 감각을 익힐 필요가 있다.

## Understanding Timeline

- 2026-10-06: Stack을 최근에 들어온 것을 먼저 처리하는 LIFO 구조로, Queue를 먼저 들어온 것을 먼저 처리하는 FIFO 구조로 구분했다.
- 2026-10-06: 괄호 검사에서 `remove()`가 아니라 가장 최근 값을 확인하고 `pop()`해야 한다는 점을 이해했다.
- 2026-10-06: `list.pop(0)`은 앞 원소 제거 후 나머지 원소 이동이 필요해 O(N)이고, Queue에서는 `deque.popleft()`를 쓰는 이유를 연결했다.
- 2026-10-06: 프로그래머스 '짝지어 제거하기'를 Stack으로 직접 풀어 최근 값 비교 패턴을 적용했다.

## Connections

- [시간복잡도와 입력 크기로 풀이 판단하기](time-complexity-and-input-size.md) — Queue 구현에서 `list.pop(0)`의 O(N) 대신 `deque.popleft()`의 O(1)을 선택하는 판단과 연결된다.
- [Python 리스트·문자열 순회와 슬라이싱](python-list-string-traversal.md) — 문자열을 한 번 순회하면서 Stack의 마지막 값과 비교해 상태를 갱신하는 패턴과 연결된다.

## Sources

- 2026-10-06 `Studying-Algorithm-Python` Day 5 학습 대화
