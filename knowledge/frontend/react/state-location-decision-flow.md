# React 상태 저장 위치 결정 플로우

## Current Understanding

## 2강 핵심: 상태를 결정하는 사고 순서

어떤 값을 보면 바로 `useState`부터 떠올리지 않고, **그 값의 원본이 어디에 있고 어디까지 유지·공유되어야 하는지**부터 판단한다.

기본적인 사고 순서는 다음처럼 기억한다.

```text
1. 다른 값으로 계산할 수 있는가?
   → Derived Value

2. 실제 원본을 서버가 가지고 있는가?
   → Server State

3. 주소로 공유하거나 유지해야 하는가?
   → URL State

4. 여러 영역에서 사용하지만 서버 데이터는 아닌가?
   → Global State 후보

5. 특정 컴포넌트나 작은 하위 트리에서만 사용하는가?
   → Local State
```

짧게 외우면:

```text
계산 가능? → Derived
서버 소유? → Server
URL 필요? → URL
전역 공유? → Global
나머지 → Local
```

실제 구조를 잡을 때는 여기서 `Form State`와 `Lifting State Up`까지 같이 본다.

```text
계산 가능?
→ Derived

서버 소유?
→ Server

URL에 남아야 해?
→ URL

사용자가 입력하거나 수정 중인 폼 값?
→ Form

한 컴포넌트 / 작은 하위 트리에서만 사용?
→ Local

가까운 형제 컴포넌트가 같이 사용?
→ 가장 가까운 공통 부모로 Lift Up

멀리 떨어진 여러 영역에서 공유하는 클라이언트 소유 상태?
→ Global 후보
```

핵심은 **여러 컴포넌트가 쓴다고 바로 Global로 보내지 않는 것**이다. 사용하는 범위가 가까우면 먼저 공통 부모로 올리는 방법을 보고, 앱 전반이나 멀리 떨어진 영역이 공유하는 클라이언트 소유 상태일 때 Global State를 후보로 본다.

## Local State

특정 컴포넌트 또는 작은 하위 트리 안에서만 필요한 상태로 이해한다.

대표적인 예:

```text
모달 열림 여부
드롭다운 열림 여부
현재 선택된 탭
입력창의 임시 값
hover 상태
아코디언 열림 여부
```

Local State는 사용하는 곳 가까이에 있으므로 변경 범위가 작고 코드 추적이 쉽다. 자주 바뀌는 상태를 불필요하게 상위나 전역으로 올리지 않으면 관련 없는 영역까지 영향을 주는 범위도 줄일 수 있다.

또 해당 컴포넌트가 실제로 unmount되면 그 컴포넌트가 소유하던 Local State도 함께 사라진다고 이해한다.

```text
상태는 가능한 한
실제 사용하는 곳 가까이에 둔다.
```

## Global State

여러 페이지나 멀리 떨어진 컴포넌트가 공유하는 **클라이언트 소유 상태**로 이해한다.

대표적인 예:

```text
다크 모드
언어 설정
사이드바 접힘 상태
전역 토스트
다단계 작성 과정의 임시 데이터
```

반대로 다음처럼 원본의 성격이 다른 값을 `여러 곳에서 쓴다`는 이유만으로 Global Store에 넣지 않는다.

```text
서버 상품 목록 / 거래내역
→ Server State

검색 페이지 번호 / 필터 / 정렬 조건
→ URL로 유지·공유해야 한다면 URL State

총금액 / 완료 개수
→ Derived Value

모달 하나의 열림 상태
→ 보통 Local State
```

따라서 Global 여부는 단순히 사용 컴포넌트 수가 아니라 다음처럼 판단한다.

```text
클라이언트가 소유하는 값인가?
+
멀리 떨어진 여러 영역이 공유해야 하는가?
→ Global State 후보
```

## Server State

서버가 실제 원본을 가지고 있고, 클라이언트는 그 데이터를 가져와 복사본을 보여주는 상태로 이해한다.

서버 상태는 데이터 하나만 있는 것이 아니라 요청 과정의 상태도 함께 존재한다.

```text
data
isPending
isError
error
isFetching
```

그래서 서버 데이터를 일반 `useState` 몇 개로 직접 관리하면 `data`, 로딩, 에러뿐 아니라 캐시, 재요청, 최신성, 요청 취소 같은 문제까지 직접 관리해야 할 수 있다.

```text
서버 데이터라고 해서
일반 useState로 관리하는 것이 최선은 아니다.
```

이 판단 이후 실제 서버 상태 관리 방법은 TanStack Query 지식으로 이어진다.

## URL State

현재 화면의 조건을 URL로 표현하고, 새로고침·뒤로가기·앞으로가기·링크 공유와 연결해야 하는 값은 URL State 후보로 본다.

대표적인 예:

```text
검색어
필터
정렬 조건
페이지 번호
선택된 탭
조회 기간
```

예:

```text
/products?category=shoes&page=2&sort=price
```

URL State의 중요한 성질은 단순 저장 위치가 아니라 **현재 화면 상태를 주소로 복원하고 공유할 수 있다는 것**이다.

```text
새로고침해도 조건 유지
현재 화면 링크 공유 가능
뒤로가기 / 앞으로가기와 연결
브라우저 히스토리와 연결
```

또 필터 조건이 바뀌면 기존 페이지 번호가 더 이상 자연스럽지 않을 수 있으므로, 카테고리 변경 시 `page=1`로 되돌리는 것처럼 **서로 의존하는 URL 상태를 함께 조정하는 UI 규칙**도 고려한다.

## Form State

사용자가 입력하거나 수정 중인 과정의 값은 Form State로 구분해서 볼 수 있다.

```text
이메일
비밀번호
목표 이름
목표 금액
편집 중인 임시 값
```

서버에서 받아온 값을 편집 폼의 초기값으로 사용할 수는 있지만, 이후 사용자가 수정 중인 값은 서버 원본과 의미가 다른 **편집 중 임시 상태**가 될 수 있다.

## Derived Value

기존 데이터로 계산할 수 있는 값은 별도 state로 중복 저장하지 않는다.

예:

```text
goals
→ 원본

completedGoals
totalSavedAmount
completionRate
→ 계산 결과
```

또는:

```text
products
→ Server State

totalProductCount = products.length
→ Derived Value
```

이런 값을 다시 state로 저장하면 원본과 계산 결과를 따로 동기화해야 해서 Single Source of Truth가 깨질 수 있다.

```text
계산할 수 있는 값은
상태로 중복 저장하지 않는다.
```

## 한 화면에 여러 종류의 상태가 함께 존재할 수 있다

상태 종류는 페이지마다 하나를 고르는 것이 아니다. 한 화면에서도 값의 성격에 따라 서로 다른 상태 종류가 동시에 존재할 수 있다.

```text
ProductPage

category
→ URL State

products
→ Server State

isFilterOpen
→ Local State

totalProductCount
→ Derived Value
```

즉 컴포넌트 단위로 `이 화면은 Local State 화면`이라고 정하는 것이 아니라 **값 하나하나의 소유자와 역할을 보고 분류한다.**

## Quick Recall

```text
계산 가능? → Derived
서버 소유? → Server
URL 필요? → URL
폼 입력/수정 중? → Form
나만 써? → Local
가까운 형제가 같이 써? → Lift Up
멀리 떨어진 여러 영역이 공유하는 클라이언트 상태? → Global 후보
```

반드시 기억할 기준:

```text
서버 데이터라고 해서 일반 useState가 최선은 아니다.
여러 컴포넌트가 쓴다고 무조건 Global State는 아니다.
계산 가능한 값은 state로 중복 저장하지 않는다.
URL로 표현해야 하는 화면 조건은 URL State로 관리한다.
상태는 가능한 한 실제 사용하는 곳 가까이에 둔다.
```

## Open Questions

없음.

## Understanding Timeline

- 2026-09-09: 값을 보면 바로 `useState`를 떠올리기보다 계산 가능 여부, 서버 소유 여부, URL 필요 여부, 사용하는 범위를 순서대로 판단해 상태의 위치를 결정하는 흐름을 정리했다.
- 2026-09-10: 2강 내용을 다시 대조하면서 Local/Global/Server/URL/Derived의 역할을 값의 소유자와 유지·공유 범위 기준으로 연결하고, 한 화면 안에서도 여러 종류의 상태가 함께 존재할 수 있다는 구조로 이해를 보강했다.

## Connections

- [React 렌더링과 상태 위치](rendering-and-state.md) — 상태를 사용하는 곳 가까이에 둘수록 불필요한 리렌더링 범위를 줄일 수 있다는 이해와 연결된다.
- [TanStack Query 서버 상태와 캐시 관리](tanstack-query-server-state-cache.md) — 서버가 원본인 데이터를 Server State로 판단한 뒤, 실제 클라이언트에서 그 서버 데이터의 복사본과 요청 상태를 관리하는 방법으로 이어진다.

## Sources

- 2026-09-09 React 상태 종류와 저장 위치를 구분하는 2강 학습 및 정리 대화
- 2026-09-10 사용자가 다시 제공한 2강 상태 분류 학습 내용
