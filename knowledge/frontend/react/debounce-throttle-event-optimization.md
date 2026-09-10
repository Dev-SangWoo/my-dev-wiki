# Debounce와 Throttle 이벤트 최적화

## Current Understanding

1. 빈번하게 발생하는 이벤트를 그대로 처리하면 필요 이상으로 계산이나 API 요청이 생길 수 있어서 `debounce`와 `throttle`로 실행 횟수를 조절한다고 이해한다.

2. `Debounce`는 이벤트가 계속 발생하면 기다렸다가 마지막 이벤트 이후 일정 시간 동안 추가 이벤트가 없을 때 실행한다.

```text
끝난 뒤 한 번
→ Debounce
```

검색처럼 중간 입력보다 마지막 입력값이 중요한 경우에 적합하다.

3. 검색 입력에서는 화면에 보여주는 원본 값과 실제 검색에 사용하는 값을 분리한다.

```text
keyword
→ 사용자가 입력하면 즉시 변경

debouncedKeyword
→ 일정 시간 추가 입력이 없으면 변경
```

입력 UI 자체를 debounce해서 느리게 만드는 것이 아니라, API 요청에 사용할 값만 늦춘다고 이해한다.

4. TanStack Query와 연결할 때는 `debouncedKeyword`를 Query Key와 Query Function의 검색 조건에 사용한다.

```text
사용자 입력
→ keyword 즉시 변경
→ Debounce
→ debouncedKeyword 변경
→ Query Key 변경
→ Query 실행
→ 결과 캐싱
```

검색어가 다르면 검색 결과도 다르므로 검색어는 Query Key에 포함되어야 하고, 검색어별로 캐시를 구분할 수 있다.

5. `enabled`는 Query Key와 역할이 다르다.

```text
queryKey
→ 어떤 검색 결과인가?

enabled
→ 지금 이 Query를 실행할 것인가?
```

예를 들어 검색어가 두 글자 이상일 때만 Query 실행을 허용하는 식으로 불필요한 요청을 막을 수 있다.

6. Debounce와 요청 취소는 서로 다른 문제다.

```text
Debounce
→ 요청이 시작되는 횟수를 줄임

AbortSignal
→ 이미 시작됐지만 더 이상 필요 없는 요청을 취소
```

TanStack Query가 `queryFn`에 전달하는 `signal`을 실제 `fetch`에 넘겨야 HTTP 요청 취소까지 연결될 수 있다고 이해한다.

7. 검색 UI에서도 `isPending`과 `isFetching`을 구분한다.

```text
isPending
→ 아직 보여줄 Query 데이터가 없는 최초 로딩

isFetching
→ 지금 queryFn이 실행 중인지 여부
```

기존 결과가 있는 상태에서 새 검색을 하는 경우에는 화면을 전부 비우기보다 기존 결과를 유지하면서 작은 로딩 표시를 보여주는 방법도 있다고 이해한다. 다만 Query Key가 바뀔 때 이전 결과를 계속 보여줄지는 Query 설정과 UI 전략에 따라 달라지므로, 이전 검색 결과가 새 검색어의 결과처럼 오해되지 않게 해야 한다.

8. 한글 입력은 IME 조합 과정에서 중간값이 생길 수 있어서 필요하다면 화면 입력값과 검색에 사용할 값을 분리하고 `compositionstart`, `compositionend` 등을 고려할 수 있다. 모든 검색창에 무조건 적용하는 것은 아니고 실제 동작을 보고 필요할 때 사용한다.

9. `Throttle`은 이벤트가 계속 발생하더라도 일정 시간 동안 최대 한 번만 실행하도록 제한한다.

```text
진행 중 일정 간격
→ Throttle
```

스크롤 위치, 마우스 이동, 드래그처럼 행동 중간의 상태도 계속 필요할 때 적합하다.

10. Debounce와 Throttle의 기준은 다음처럼 기억한다.

```text
행동이 끝난 뒤 최종 결과만 필요
→ Debounce

행동 중간의 상태도 필요
→ Throttle
```

검색은 일반적으로 Debounce가 적합하고, 스크롤 진행률처럼 중간 값이 필요한 경우에는 Throttle이 적합하다.

11. 스크롤 관련 작업이라고 무조건 Throttle을 쓰는 것은 아니다.

```text
스크롤 위치 자체가 필요
→ Throttle

요소가 화면에 들어왔는지만 필요
→ IntersectionObserver

화면 프레임에 맞춘 업데이트가 필요
→ requestAnimationFrame 고려
```

도구를 이벤트 종류가 아니라 실제 필요한 정보에 따라 선택한다고 이해한다.

12. `leading`은 이벤트 시작 시 즉시 실행, `trailing`은 연속 이벤트가 끝난 뒤 실행하는 방식으로 이해한다. 검색 Debounce는 보통 마지막 입력 뒤 실행되는 trailing 방식이 자연스럽다.

13. 검색 최적화 전체 흐름은 다음처럼 연결한다.

```text
사용자 입력
→ Local State 즉시 변경
→ Debounce
→ debouncedKeyword 변경
→ Query Key 변경
→ enabled 확인
→ queryFn 실행
→ AbortSignal로 불필요한 요청 취소 가능
→ Server State 응답
→ Query Cache 저장
→ 화면 렌더링
```

14. Debounced Value를 직접 구현할 때는 `setTimeout`만 쓰는 것이 아니라 이전 타이머를 `clearTimeout`으로 정리하는 cleanup이 중요하다고 이해한다.

```text
새로운 입력
→ 이전 effect cleanup
→ 이전 타이머 제거
→ 새로운 타이머 생성
```

cleanup이 없으면 이전 검색어의 타이머도 살아남아서 여러 지연 값이 순서대로 반영될 수 있다.

15. Debounce는 검색 문제 전체를 해결하는 만능 기능이 아니라고 이해한다. 검색 UX에서는 다음 문제들을 함께 본다.

```text
빈 검색어 처리
최소 검색 글자 수
enabled
이전 요청 취소
로딩 / 에러 / 검색 결과 없음
검색어별 캐싱
한글 IME 입력
모바일 네트워크
```

즉 Debounce는 그중 **요청 시작 횟수를 줄이는 역할**을 맡는다.

16. callback 형태의 debounce 함수를 컴포넌트 렌더링마다 새로 만들면 새로운 함수와 타이머가 계속 생길 수 있어서 debounce 자체가 제대로 유지되지 않을 수 있다고 이해한다. 이런 방식을 사용할 때는 함수 참조를 안정적으로 유지하고 cleanup도 고려해야 한다.

17. Debounce delay는 길수록 좋은 것이 아니다. 요청 수를 줄이는 것과 사용자가 느끼는 검색 응답 속도 사이의 균형을 봐야 한다.

```text
delay 너무 짧음
→ 요청이 많이 발생할 수 있음

delay 너무 김
→ 검색이 느리게 느껴질 수 있음
```

사용자 입력 속도, API 응답 속도, 검색 결과 중요도, 서버 비용, 모바일 환경 등을 보고 결정한다.

18. `passive: true`는 이벤트 리스너가 `preventDefault()`로 기본 동작을 취소하지 않을 것이라고 브라우저에 알려주는 옵션으로 이해한다. 기본 `scroll` 이벤트 자체는 취소할 수 없어서 `passive` 여부의 효과를 크게 신경 쓸 필요가 없고, 스크롤을 기본 동작으로 가지는 `wheel`, `touchstart`, `touchmove` 같은 취소 가능한 이벤트에서 의미가 더 크다. `preventDefault()`가 필요한 이벤트에는 `passive: true`를 사용하면 안 된다.

## 한 문장 정리

> Debounce는 사용자의 행동이 끝나기를 기다렸다가 최종 값으로 작업하고, Throttle은 행동이 계속되는 동안 일정 간격으로 작업한다. 검색에서는 입력값은 즉시 보여주고 Debounced Value를 Query Key에 사용하며, `enabled`로 실행 조건을 제한하고 `AbortSignal`로 이미 시작된 불필요한 요청까지 취소한다.

## Open Questions

- 실제 검색 UX에서 debounce delay를 어느 정도로 잡는 것이 적절한지는 사용자 입력 속도, API 응답 속도, 서버 비용 등을 보면서 판단할 경험이 더 필요하다.
- 한글 IME Composition 처리가 실제 서비스에서 필요한 경우와 브라우저 기본 동작만으로 충분한 경우를 더 경험해볼 필요가 있다.

## Understanding Timeline

- 2026-09-09: Debounce는 행동이 끝난 뒤 최종값을 처리하고, Throttle은 행동 중간에도 일정 간격으로 처리한다는 기준으로 구분했다.
- 2026-09-09: 검색 입력값 자체를 늦추는 것이 아니라 화면 입력값은 즉시 변경하고 API에 사용할 `debouncedKeyword`만 지연시키는 구조로 이해했다.
- 2026-09-09: Debounce는 요청 시작 횟수를 줄이고, AbortSignal은 이미 시작된 불필요한 요청을 취소한다는 서로 다른 역할을 연결했다.
- 2026-09-09: 검색 최적화를 Debounce 하나가 아니라 Query Key, `enabled`, AbortSignal, Query Cache까지 이어지는 전체 흐름으로 이해했다.

## Connections

- [TanStack Query 서버 상태와 캐시 관리](tanstack-query-server-state-cache.md) — Debounced 검색어가 Query Key를 바꾸고, `enabled`와 AbortSignal을 통해 Query 실행과 요청 취소를 제어한 뒤 결과를 Query Cache에 저장하는 흐름으로 이어진다.

## Sources

- 2026-09-09 Debounce와 Throttle로 이벤트 최적화하기 4강 학습 자료
- 2026-09-09 Debounce와 Throttle 핵심 요약 대화
