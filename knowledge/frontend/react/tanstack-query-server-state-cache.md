# TanStack Query 서버 상태와 캐시 관리

## Current Understanding

1. React Query(TanStack Query)는 API를 직접 호출하는 도구 자체라기보다, API 호출 전후의 여러 상태를 관리하고 서버 데이터를 얼마나 빠르고 효율적으로 사용자에게 보여줄지를 고민해주는 서버 상태 관리자라고 이해한다. 캐싱 역할이 매우 크다.

```text
fetch / axios
→ 실제 API 요청

TanStack Query
→ 서버 데이터의 클라이언트 복사본을 캐싱하고 관리
→ 서버와 캐시를 동기화
```

2. `queryKey`는 어떤 데이터인지 구분하는 조건이자 캐시의 주소다. 조건이 세분화되어 있고 이 Key를 기준으로 서로 다른 데이터가 캐싱된다.

```text
queryFn의 결과를 바꾸는 조건
→ queryKey에도 포함
```

예를 들면 `page`, `filter`, `category`, `keyword`, `sort`, `userId`처럼 요청 결과를 바꾸는 값은 Query Key에도 들어가야 한다.

3. `queryFn`은 실제로 호출되는 API 함수다. 머릿속에서는 다음처럼 구분한다.

```text
queryKey
→ 어떤 데이터인가?

queryFn
→ 그 데이터를 어떻게 가져오는가?
```

4. `staleTime`은 신선도 기준 시간이다.

```text
Fresh
→ 아직 최신이라고 믿는 데이터

Stale
→ 캐시에 데이터는 있지만 최신이라고는 보장하지 않는 데이터
```

`staleTime`이 지났다고 데이터를 바로 지우거나 즉시 API 요청을 보내는 것은 아니다. 신선도가 떨어져 stale 상태가 되고, 이후 마운트·포커스·재연결·invalidate·직접 refetch 같은 계기가 생기면 다시 확인할 수 있다고 이해한다.

```text
staleTime: 60초
≠ 60초마다 요청

= 60초 동안 fresh로 믿는다
```

5. `gcTime`은 보관 기간이다. 아무도 사용하지 않는 Query가 inactive가 된 뒤 캐시를 언제 삭제할지 정한다. 메모리를 계속 잡아둘 필요가 없기 때문에 보관 시간이 지나면 캐시에서 제거한다.

```text
staleTime
→ 신선도

gcTime
→ 사용하지 않는 캐시의 보관 기간
```

Fresh/Stale은 데이터의 신선도이고, Active/Inactive는 현재 그 Query를 사용하는 곳이 있는지에 대한 구분으로 따로 생각한다.

6. `isPending`과 `isFetching`은 다르게 본다.

```text
isPending
→ 아직 사용할 Query 데이터가 없는 상태인가?

isFetching
→ 지금 queryFn이 실행 중인가?
```

최초 요청에서는 `isPending`과 `isFetching`이 같이 true일 수 있고, 기존 데이터를 이미 화면에 보여주면서 백그라운드로 새 데이터를 요청하는 경우에는 `isPending`은 false이고 `isFetching`은 true일 수 있다고 이해한다.

7. 서버 데이터를 읽는 것은 Query, 서버 데이터를 생성·수정·삭제하는 것은 Mutation으로 생각한다.

```text
Query
→ 서버 데이터 읽기

Mutation
→ POST / PATCH / DELETE처럼 서버 데이터 변경
```

8. Mutation이 성공하면 서버 원본은 바뀌었는데 브라우저의 기존 Query Cache는 과거 데이터일 수 있다. 그래서 Mutation 후에는 서버와 캐시를 다시 맞춰야 한다.

### `invalidateQueries`

```text
Mutation 성공
→ 관련 Query invalidate
→ 해당 Query를 stale 처리
→ 현재 사용 중인 Query라면 다시 서버 데이터를 확인
```

단순히 "POST 성공하면 무조건 GET 자동 실행"이라고 보기보다, 관련 캐시를 무효화해서 stale로 만들고 active Query가 다시 최신 데이터를 가져오게 하는 흐름으로 이해한다.

`invalidateQueries`는 Query Key의 부분 일치로 넓게 잡을 수 있어서 범위를 크게 잡으면 관련된 다른 캐시까지 같이 무효화될 수 있다.

```text
['transactions']
→ transactions로 시작하는 관련 Query들을 넓게 다룰 수 있음
```

그러므로 무효화 범위를 너무 크게 잡으면 불필요한 요청이 생길 수 있다.

9. Mutation 후 캐시를 맞추는 방법은 `invalidateQueries` 하나만 있는 것이 아니다.

```text
invalidateQueries
→ 서버에서 다시 조회

setQueryData
→ Mutation 응답 등 이미 알고 있는 데이터로 캐시를 직접 수정

Optimistic Update
→ 성공할 것이라 예상하고 화면/캐시를 먼저 변경
→ 실패하면 이전 상태로 되돌림
```

서버의 최종 계산 결과를 다시 받아야 하거나 구현을 단순하게 가져가고 싶으면 `invalidateQueries`, Mutation 응답만으로 캐시를 정확하게 갱신할 수 있다면 `setQueryData`, 즉각적인 반응이 중요하고 실패 처리까지 감당할 수 있다면 Optimistic Update를 고려한다고 이해한다.

10. Query Key 관리는 프로젝트가 커질수록 중요해진다. Query Key는 단순한 문자열이 아니라 캐시를 조회하고 공유하고 무효화하는 기준이므로, Key 구조 자체를 캐시 구조 설계라고 생각한다.

```text
transactions
├─ list
│  ├─ filter A
│  └─ filter B
└─ detail
   ├─ id 1
   └─ id 2
```

프로젝트가 커지면 Key를 여기저기 직접 쓰기보다 key factory처럼 한곳에서 관리해서 `list`, `detail`, `filter` 구조를 일관되게 유지하는 것이 좋다고 이해한다.

```tsx
const transactionKeys = {
  all: ['transactions'] as const,

  lists: () =>
    [...transactionKeys.all, 'list'] as const,

  list: (filters: TransactionFilters) =>
    [...transactionKeys.lists(), filters] as const,

  details: () =>
    [...transactionKeys.all, 'detail'] as const,

  detail: (id: number) =>
    [...transactionKeys.details(), id] as const,
};
```

이렇게 하면 조회할 때와 `invalidateQueries`할 때 같은 구조를 사용해서 오타나 잘못된 무효화 범위를 줄일 수 있다.

11. 서버 데이터를 `useEffect + useState`만으로 가져오는 코드도 동작할 수 있지만, 실제 서버 상태를 관리하려고 하면 직접 처리해야 할 문제가 계속 늘어난다고 이해한다.

```text
data
isLoading
error
캐시
lastUpdatedAt
isRefetching
retry
request cancellation
```

그리고 다음 같은 문제를 직접 해결해야 한다.

```text
다른 컴포넌트에서도 같은 데이터를 요청하면?
페이지를 나갔다 돌아오면?
이미 가져온 데이터를 재사용하려면?
서버 데이터가 바뀌었는지 어떻게 알까?
요청 실패 시 재시도하려면?
이전 요청과 새 요청이 겹치면?
Mutation 후 목록을 어떻게 갱신할까?
```

그래서 TanStack Query의 가치는 단순히 `fetch` 코드를 짧게 만드는 데 있는 것이 아니라, **서버 데이터의 캐시·최신성·재요청·로딩·에러·동기화 같은 문제를 서버 상태라는 하나의 모델로 관리하는 것**에 있다고 이해한다.

12. Query에서 받은 데이터를 같은 의미의 `useState`에 다시 복사하지 않는 것을 Single Source of Truth와 연결해서 이해한다.

```tsx
const { data } = useQuery(...);
const [transactions, setTransactions] = useState(data);
```

이렇게 하면 같은 의미의 데이터가 두 군데에 생길 수 있다.

```text
Query Cache의 transactions
+
Local State의 transactions
```

Query Cache가 refetch로 최신 데이터가 되어도 Local State는 예전 값을 들고 있을 수 있어서 다시 동기화 코드가 필요해질 수 있다.

보통은 Query 데이터를 그대로 사용한다.

```tsx
const { data: transactions = [] } = useQuery(...);
```

다만 서버 데이터를 **편집 폼의 초기값**으로 가져온 뒤 사용자가 별도의 임시 값을 수정하는 상황처럼 의미가 달라지는 경우는 구분한다.

```text
Query data
→ 서버 원본의 클라이언트 복사본

편집 중 form value
→ 사용자가 수정 중인 별도 임시 상태
```

즉 판단 기준은 다음과 같다.

```text
같은 의미의 데이터인가?
→ Query data를 다시 Local State에 복사하지 않음

별도의 편집/임시 의미가 생겼는가?
→ Local / Form State로 둘 수 있음
```

13. `staleTime`은 모든 Query에 똑같이 넣는 단순 성능 숫자가 아니라, **이 데이터를 얼마 동안 최신이라고 믿어도 되는지에 대한 freshness 정책**이라고 이해한다.

예를 들어 데이터마다 성격이 다르다.

```text
거의 변하지 않는 기준 목록
→ 오래 fresh로 볼 수 있음

거래내역
→ 새 거래가 생기면 빠르게 최신화할 필요가 있음

실시간성이 중요한 데이터
→ 최신성 요구가 훨씬 높음
```

그래서 staleTime은 다음을 보면서 정한다.

```text
데이터 변경 빈도
사용자가 최신성을 기대하는 정도
API 비용
화면 진입 빈도
수동 갱신 가능성
```

머릿속에서는 다음 질문으로 기억한다.

```text
이 데이터를 사용자가 얼마 동안
최신이라고 믿어도 괜찮은가?
```

즉 `staleTime`은 단순히 요청 수를 줄이기 위한 캐시 시간이 아니라 **데이터 특성과 제품 요구에 맞춘 최신성 판단 기준**이다.

## 한 문장 정리

> 서버가 원본인 데이터는 TanStack Query가 클라이언트 복사본과 요청 상태를 관리하게 하고, queryKey로 데이터를 식별하고 queryFn으로 가져온다. staleTime으로 그 데이터를 언제까지 최신이라고 믿을지 판단하고 gcTime으로 사용하지 않는 캐시의 보관 기간을 관리한다. Query 데이터를 같은 의미의 local state로 다시 복사하지 않으며, Mutation으로 서버 데이터가 바뀌면 invalidateQueries, setQueryData, Optimistic Update 등을 이용해 서버와 캐시를 다시 맞춘다.

## Quick Recall

```text
fetch / axios
→ HTTP 요청

TanStack Query
→ 서버 상태의 클라이언트 복사본 + 요청 상태 관리
```

```text
queryKey → 어떤 데이터인가?
queryFn  → 어떻게 가져오는가?
staleTime → 언제까지 최신이라고 믿는가?
gcTime → 아무도 쓰지 않을 때 언제 캐시에서 지우는가?
```

```text
Query data → 같은 의미의 useState로 재복사하지 않기
편집 중 임시 값 → 의미가 다르면 Form / Local State 가능
```

## Open Questions

- 실제 프로젝트에서 `invalidateQueries`, `setQueryData`, Optimistic Update 중 어떤 방식을 선택할지 더 많은 사례를 경험해볼 필요가 있다.
- 실제 프로젝트 규모에서 Query Key factory를 어느 단위까지 나누는 것이 적절한지 경험해볼 필요가 있다.

## Understanding Timeline

- 2026-09-09: TanStack Query를 API 요청 도구가 아니라 서버 데이터의 클라이언트 복사본을 캐싱하고 서버와 동기화하는 서버 상태 관리자로 이해했다.
- 2026-09-09: `staleTime`을 자동 요청 주기가 아니라 신선도 기준으로, `gcTime`을 inactive 캐시의 보관 기간으로 구분했다.
- 2026-09-09: Mutation 후 서버와 캐시를 맞추는 방법이 `invalidateQueries`뿐 아니라 `setQueryData`, Optimistic Update도 있다는 흐름을 연결했다.
- 2026-09-09: Query Key를 단순한 캐시 이름이 아니라 조회·공유·무효화의 기준이 되는 캐시 구조 설계로 이해하고, 프로젝트가 커지면 key factory 형태로 관리하는 관점을 추가했다.
- 2026-09-10: 서버 상태를 `useEffect + useState`로 직접 관리할 때 생기는 반복 문제를 TanStack Query의 역할과 연결하고, Query 데이터를 같은 의미의 Local State에 재복사하지 않는 Single Source 원칙과 staleTime을 데이터 최신성 정책으로 보는 관점을 보강했다.

## Connections

- [React 상태 저장 위치 결정 플로우](state-location-decision-flow.md) — 서버가 원본을 가지고 있는 값을 Server State로 판단한 다음, 실제 클라이언트에서 그 서버 상태를 관리하는 방법으로 TanStack Query가 이어진다.
- [React 렌더링과 상태 위치](rendering-and-state.md) — Query 데이터를 같은 의미의 Local State로 다시 복사하지 않는 판단은 계산 가능한 값이나 동일한 의미의 데이터를 중복 저장하지 않는 Single Source of Truth와 연결된다.
- [Debounce와 Throttle 이벤트 최적화](debounce-throttle-event-optimization.md) — 검색처럼 Query가 너무 자주 시작될 수 있는 상황에서 Debounce로 실행 빈도를 줄이고, `enabled`와 AbortSignal로 Query 실행 조건과 이미 시작된 요청을 제어하는 흐름으로 이어진다.

## Sources

- 2026-09-09 TanStack Query로 서버 상태 관리하기 3강 학습 자료
- 2026-09-09 TanStack Query 핵심 요약 및 Query Key 관리 정리 대화
- 2026-09-10 3강 누락 개념 복습 및 이해 확인 대화
