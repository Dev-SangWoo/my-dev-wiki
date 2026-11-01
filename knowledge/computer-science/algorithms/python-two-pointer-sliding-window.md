# Python Two Pointer와 Sliding Window의 구간 이동

## Current Understanding

1. 정렬된 배열에서 두 수의 합을 맞추려 할 때, **값을 키워야하니까 뭐든 오른쪽으로 가야하는데 이제 한쪽은 끝이니까 1이 2로 가서 11이 될듯**이라고 판단했다. 이후 정렬된 배열에서 합이 작으면 왼쪽 포인터를 늘리고 합이 크면 오른쪽 포인터를 줄이는 원리를 확인했다.

2. 두 수의 합을 하나 찾으면 끝내는 코드에 대해 **이 코드는 하나의 경우 밖에 출력 못하고 만약 삐걱 거리는 경우는 없나?**라고 질문했다. 이어서 중복 없는 배열에서는 답을 발견했을 때 **한쪽만 움직이면 이제 무조건 틀리니까 양쪽 둘다 움직이는게 좋을꺼 같은데**라고 판단했고, 두 포인터를 양쪽 끝이 아닌 가운데서 시작해도 되는지 질문했다.

3. 고정 길이 Sliding Window의 계산은 **2를 빼고 1을 더하겠지?**라고 설명했다. 코드 작성에서는 처음에 빠지는 값의 위치를 `right-2`라고 혼동했지만 `right-k`라고 교정했고, 예시 `right=4, k=3`에서 빠지는 값 1과 새로 들어오는 값 3을 맞혔다.

4. 가변 길이 Sliding Window를 다시 배울 때 **연속 구간을 찾으니까, 연속된 구간을 찾을 때 뒤로 다시 가지 않도록**이라고 설명했다. 조건 만족 시 왼쪽을 줄이고 합이 부족해지면 **오른쪽으로 구간을 늘린다**고 판단했다.

5. 가변 길이 구간에서는 처음에 **최악을 가정하면 n^2일꺼 같은데**라고 추측했다. 이후 포인터가 전체 배열을 통틀어 앞으로만 움직인다는 설명을 듣고, `N=10`에서 `left=6`까지 이동했다면 남은 최대 이동 횟수는 **4번이지 않을까**라고 답했다.

6. 최종 실습에서 양수 배열의 최소 길이 구간을 직접 다음과 같이 구현했다.

```python
arr = [1, 2, 3, 4, 5]
target = 9

left = 0
window = 0
best = float('inf')

for right in range(len(arr)):
    window += arr[right]

    while window >= target:
        best = min(best, right - left + 1)
        window -= arr[left]
        left += 1
```

## Quick Recall

```text
정렬된 배열에서 두 수의 합 → 양끝 Two Pointer
연속 구간에서 이전 합 재사용 → Sliding Window
고정 길이 → window - arr[right-k] + arr[right]
가변 길이(모두 양수) → 합이 부족하면 확장, 충분하면 축소
while이 중첩되어도 두 포인터가 앞만 향하고 각각 최대 N번 이동 → O(N)
```

## Open Questions

- 원소에 0이나 음수가 포함된 경우에는 가변 길이 Sliding Window의 이동 규칙을 그대로 적용할 수 있는지 별도 연습이 필요하다.
- 답이 없는 경우와 길이 1인 구간 등 경계 조건을 스스로 점검하는 습관은 더 연습이 필요하다.

## Understanding Timeline

- 2026-10-20: 정렬된 배열의 양끝 Two Pointer와 고정 길이 Sliding Window의 포인터 이동 및 합 재사용을 연습했다.
- 2026-11-02: 가변 길이 Sliding Window의 O(N) 이유를 포인터 전체 이동 횟수와 연결했고, 최소 길이 문제의 확장·축소 코드를 직접 구현했다.

## Connections

- [시간복잡도와 입력 크기로 풀이 판단하기](time-complexity-and-input-size.md) — 중첩 반복문의 모양이 아니라, 두 포인터의 전체 이동 횟수로 O(N)을 판단하는 관점이 연결된다.

## Sources

- `Studying-Algorithm-Python` Day 8 학습 대화
