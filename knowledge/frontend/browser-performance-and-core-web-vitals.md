# 브라우저 성능 측정과 Core Web Vitals

## Current Understanding

1. 성능 문제는 코드를 보고 "리렌더링이 많은 것 같다", "계산이 무거워 보인다"고 바로 최적화하는 것이 아니라 다음 흐름으로 본다.

```text
진짜 느린가?
↓
어디서 시간이 걸렸는가?
↓
왜 걸렸는가?
↓
무엇을 수정할 것인가?
↓
수정 후 실제로 좋아졌는가?
```

2. 사용자의 행동이 실제 화면 변화로 이어지는 흐름을 다음처럼 연결해서 이해한다.

```text
사용자 Click
    ↓
JavaScript 이벤트 처리
    ↓
React State 변경
    ↓
React Render / Commit
    ↓
브라우저 Style 계산
    ↓
Layout
    ↓
Paint
    ↓
Composite
    ↓
사용자 눈에 화면 변화
```

3. 성능 도구의 역할을 다음처럼 구분한다.

```text
React Profiler
→ 어떤 컴포넌트가 렌더링되고 계산되고 있지?

Chrome Performance
→ 브라우저가 어디서 시간을 썼나?

Lighthouse
→ 페이지 로딩 품질이 전반적으로 괜찮나?

Core Web Vitals
→ 사용자가 느끼는 주요 성능을 어떤 지표로 볼 것인가?
```

React Profiler와 Chrome Performance는 경쟁하는 도구가 아니라 보는 범위가 다르기 때문에 필요하면 같이 사용한다.

```text
React Profiler
→ React가 왜 많이 렌더링했지?

Chrome Performance
→ 그래서 실제 브라우저는 어디에서 오래 일했지?
```

4. 브라우저 렌더링 흐름은 다음처럼 기억한다.

```text
JavaScript
↓
Style
↓
Layout
↓
Paint
↓
Composite
```

변경 내용에 따라 이 단계가 항상 전부 발생하는 것은 아니지만, 성능을 볼 때 `Main Thread`, `Long Task`, `Style Calculation`, `Layout`, `Paint`, `Composite`를 주요 포인트로 본다.

### Main Thread

브라우저의 많은 UI 관련 작업은 Main Thread에서 실행된다고 이해한다.

```text
JavaScript 실행
이벤트 처리
Style 계산
Layout
Paint 관련 작업
```

Main Thread를 너무 오래 점유하면 클릭과 입력 처리가 늦어지고, 렌더링이나 애니메이션도 지연될 수 있다.

### Long Task

Main Thread의 하나의 작업이 오래 지속되면 다른 사용자 입력이나 Paint가 기다려야 할 수 있다. Chrome Performance에서는 보통 **50ms를 넘는 Main Thread 작업**을 Long Task로 보는 기준을 사용한다고 이해한다.

```text
입력
→ 무거운 계산
→ Main Thread 오래 점유
→ 다음 입력 / Paint 지연
```

### Style Calculation

DOM이나 CSS 조건이 달라졌을 때 어떤 CSS가 각 요소에 적용되는지 계산하는 과정이다. DOM이 매우 크거나 Style 재계산이 지나치게 자주 발생하면 비용이 커질 수 있다.

### Layout

요소의 너비, 높이, 위치 같은 geometry를 계산하는 과정이다. 한 요소의 크기 변화가 자식이나 주변 요소 위치에 영향을 주면 Layout 범위가 커질 수 있다.

### Paint

Layout 결과를 바탕으로 텍스트, 배경, border, shadow, 이미지 같은 실제 픽셀을 그리는 과정이다.

### Composite

여러 Layer를 적절한 순서로 합쳐 최종 화면을 만드는 과정이다. `transform` 기반 변화는 상황에 따라 Layout과 Paint를 피하고 Composite 중심으로 처리될 수 있고, `width`, `left`, `top`처럼 geometry를 바꾸는 속성은 Layout을 유발할 가능성이 크다고 이해한다.

5. `Layout Thrashing`은 DOM의 크기나 위치를 읽고 쓰는 작업을 반복하면서 강제로 Layout 계산이 계속 발생하는 현상으로 이해한다.

```text
읽기
→ 쓰기
→ 다시 읽기
→ 다시 쓰기
→ Layout 반복
```

가능하면 DOM 읽기와 쓰기를 섞어 반복하기보다 다음처럼 묶는 방향을 고려한다.

```text
읽기
읽기
읽기
↓
쓰기
쓰기
쓰기
```

6. Chrome Performance는 막연하게 오래 녹화하기보다 실제 문제 행동을 정확하게 지정해서 측정해야 한다고 이해한다.

```text
1. DevTools → Performance
2. Record
3. 문제 행동 한 번 수행
4. Stop
5. 해당 구간 확대
6. Main Thread 확인
7. Long Task 확인
8. Call Stack 확인
```

### Flame Chart 읽기

Flame Chart에서는 가로 길이와 세로 구조를 다르게 본다.

```text
가로 길이
→ 얼마나 오래 실행됐는가?

세로 구조
→ 누가 누구를 호출했는가?
```

그래서 단순히 "클릭이 느리다"에서 끝나는 것이 아니라 다음처럼 실제 병목 함수까지 내려가는 것이 목표라고 이해한다.

```text
click
└─ handleSearch       240ms
   └─ filterData      180ms
      └─ expensiveFn  150ms
```

7. Core Web Vitals는 다음 표로 구분해서 기억한다.

| 지표 | 전체 이름 | 무엇을 보는가 | 좋은 기준 | 머릿속 질문 |
| --- | --- | --- | --- | --- |
| **LCP** | Largest Contentful Paint | 첫 화면의 주요 콘텐츠가 보이는 로딩 속도 | **2.5초 이하** | 핵심 콘텐츠가 빨리 보이는가? |
| **INP** | Interaction to Next Paint | 클릭·탭·입력 이후 다음 시각적 피드백까지의 상호작용 응답성 | **200ms 이하** | 사용자의 행동에 빨리 반응하는가? |
| **CLS** | Cumulative Layout Shift | 예상하지 못한 화면 이동이 얼마나 발생하는지에 대한 시각적 안정성 | **0.1 이하** | 화면이 갑자기 밀리거나 흔들리지 않는가? |

이 기준은 실제 사용자 경험을 볼 때 모바일과 데스크톱을 나누고 사용자 방문의 **75번째 백분위수**에서 평가하는 관점도 함께 기억한다.

짧게 외우면:

```text
LCP → 로딩
INP → 상호작용
CLS → 시각적 안정성
```

8. LCP는 첫 화면의 중요한 콘텐츠가 얼마나 빨리 보이는지를 본다고 이해한다. 중요한 Hero 이미지가 JavaScript 실행이나 API 요청 뒤에야 발견되면 이미지 요청 자체가 늦어져 LCP가 나빠질 수 있다.

```text
HTML 요청
↓
JavaScript 다운로드
↓
JavaScript 실행
↓
API 요청
↓
이미지 URL 확인
↓
이미지 다운로드
↓
화면 표시
```

LCP 최적화에서는 중요한 리소스를 빨리 발견하고 요청하게 만들고, 리소스 로드 시간을 줄이고, Main Thread 작업 때문에 렌더링이 늦어지지 않게 하는 것을 중요하게 본다.

```text
첫 화면 핵심 이미지
→ 빨리 로딩

화면 아래 이미지
→ lazy loading 고려
```

첫 화면에서 반드시 필요한 Hero 이미지에 lazy loading을 적용하면 오히려 LCP가 늦어질 수 있다고 이해한다.

9. INP는 사용자가 클릭이나 입력 같은 상호작용을 했을 때 다음 시각적 피드백이 언제 나타나는지를 본다고 이해한다.

Main Thread가 큰 JavaScript 작업으로 오래 점유되어 있으면 클릭 이벤트가 들어와도 기다려야 한다.

```text
거대한 JS 작업 실행 중
        ↑
      Click

Click
↓
기존 작업 종료 기다림
↓
Event Handler 실행
↓
React 업데이트
↓
Paint
```

### INP가 좋지 않을 때 어떻게 해결할까?

중요한 것은 `INP가 나쁘다 → useMemo`처럼 바로 해결책을 정하지 않는 것이다.

먼저 Chrome Performance에서 어디에서 시간이 길어졌는지 확인한다.

```text
Input Delay가 긴가?
Event Handler가 긴가?
Render가 긴가?
Paint가 긴가?
```

원인을 확인한 뒤 상황에 따라 다음 해결 후보를 고려한다.

| 상황 / 문제 | 고려할 해결 후보 |
| --- | --- |
| 같은 비싼 계산이 불필요하게 반복됨 | `useMemo` 고려 |
| 클라이언트에서 하기 너무 큰 계산 | 서버에서 계산하는 방법 고려 |
| Main Thread를 오래 점유하는 무거운 계산 | Web Worker 활용 고려 |
| 하나의 작업이 너무 길게 Main Thread를 점유함 | 작업 분할 고려 |
| 한 번에 너무 많은 리스트를 렌더링함 | virtualization 고려 |
| 관계없는 컴포넌트까지 넓게 렌더링됨 | 렌더링 범위 감소, State Colocation / 컴포넌트 분리 고려 |

머릿속에서는 다음처럼 기억한다.

```text
INP 문제
↓
어디가 느린지 측정
↓
원인 확인
↓
그 원인에 맞는 해결책 선택
```

즉 `useMemo`, 서버 계산, Web Worker, 작업 분할, virtualization, 렌더링 범위 감소는 모두 해결 **후보**이고, 어떤 것을 사용할지는 측정한 병목에 따라 결정한다.

10. CLS는 화면이 갑자기 밀리거나 크기가 변하는 시각적 불안정성을 본다고 이해한다. 이미지 크기를 미리 알려주면 이미지가 로드되기 전에 공간을 확보해서 주변 콘텐츠가 덜 밀릴 수 있다.

```text
이미지가 아직 없어도
↓
브라우저가 공간을 미리 확보
↓
이미지 로드
↓
주변 콘텐츠가 덜 밀림
```

`width`와 `height`를 알려주거나 `aspect-ratio`로 비율을 확보하는 방식으로 로드 전 공간을 잡을 수 있다.

스켈레톤도 실제 콘텐츠와 크기 차이가 크면 CLS를 만들 수 있어서 다음처럼 기억한다.

```text
Skeleton의 형태
≈ 실제 Content의 형태
```

11. Lighthouse는 페이지를 자동으로 검사해서 Performance, Accessibility, SEO 같은 여러 품질 영역에 대한 보고서를 만드는 도구로 이해한다.

```text
Lighthouse 100점
≠ 실제 서비스가 모든 환경에서 무조건 빠름
```

Lighthouse는 특정 조건의 Lab Measurement이기 때문에 점수 자체를 목표로 두기보다 문제를 발견하고 원인을 좁히는 데 사용한다.

12. 성능 문제를 증상과 지표로도 연결해서 본다.

| 증상 | 우선 볼 지표 | 대표적으로 의심할 부분 |
| --- | --- | --- |
| 첫 화면 핵심 이미지가 늦게 보임 | LCP | 이미지 요청 시점, 서버, JavaScript |
| 버튼을 눌렀는데 반응이 늦음 | INP | Long Task, 무거운 이벤트 처리나 렌더링 |
| 검색 입력이 버벅임 | INP | 계산, 리렌더링 |
| 이미지가 나타나면서 화면이 밀림 | CLS | 이미지 크기 미지정, 공간 미확보 |
| Hero 영역이 늦게 나타남 | LCP | lazy loading, JavaScript 이후 늦은 리소스 발견 |
| 로딩 후 카드 위치가 바뀜 | CLS | Skeleton과 실제 콘텐츠 크기 차이 |

13. `Lab Data`와 `Field Data`는 역할이 다르다고 이해한다.

| 구분 | 의미 | 예시 | 어디에 좋은가 |
| --- | --- | --- | --- |
| **Lab Data** | 개발자가 통제된 환경에서 측정 | Lighthouse, Chrome DevTools, 로컬 테스트 | 재현, 디버깅, 수정 전후 비교 |
| **Field Data** | 실제 사용자가 사용하는 환경에서 수집 | CrUX, RUM, 실사용자 Web Vitals | 실제 기기·네트워크·사용자 행동에서 문제가 있는지 확인 |

머릿속에서는 다음처럼 구분한다.

```text
Lab
→ 원인을 찾는 데 좋음

Field
→ 실제 문제가 있는지 확인하는 데 좋음
```

즉 Lighthouse 점수만 좋다고 실제 모든 사용자의 경험도 좋다고 단정하지 않는다.

14. Lighthouse 결과는 점수만 보고 끝내지 않고, 문제가 되는 지표에서 실제 원인까지 내려가서 본다.

```text
Performance 72점
↓
왜 72점이지?
↓
LCP가 4.1초네
↓
LCP Element가 Hero Image네
↓
Network에서 이미지 요청 시작이 늦네
↓
왜 늦지?
↓
JavaScript 실행 후 이미지가 만들어지네
```

전체 흐름은 다음처럼 기억한다.

```text
Lighthouse
→ 문제 발견

Performance / Network
→ 원인 분석

코드
→ 해결

다시 Lighthouse
→ 결과 검증
```

15. 성능 문제를 만나면 먼저 어떤 사용자 행동에서 느린지 정하고, React 문제인지 브라우저 문제인지 네트워크 문제인지 범위를 좁힌 뒤 적절한 도구를 사용한다.

```text
느리다
↓
어떤 사용자 행동에서?
↓
React 문제인가?
브라우저 문제인가?
네트워크 문제인가?
↓
Profiler / Performance / Network
↓
가장 큰 병목 확인
↓
수정
↓
다시 측정
```

핵심은 **코드부터 고치지 않는 것**이다.

16. 개발 환경에서 잘 동작한다고 실제 사용자 환경에서도 빠르다고 판단하지 않는다. 고성능 CPU와 빠른 네트워크에서는 문제가 가려질 수 있기 때문에 Chrome Performance의 CPU / Network throttling으로 느린 환경을 모의해서 볼 수 있다고 이해한다.

```text
내 PC
→ 검색 부드러움

CPU slowdown / 느린 Network
→ 검색 버벅임 확인 가능
```

따라서 성능 확인을 `내 컴퓨터에서 잘 됨`으로 끝내지 않는다.

17. 지금까지 배운 내용을 성능 흐름 하나로 연결하면 다음처럼 볼 수 있다.

```text
사용자 검색 입력
    ↓
Debounce
    ↓
TanStack Query
    ↓
Server State
    ↓
React Render
    ↓
State 범위 판단
    ↓
필요한 memoization
    ↓
React Commit
    ↓
Browser
JavaScript → Style → Layout → Paint → Composite
    ↓
사용자에게 화면 표시
    ↓
LCP / INP / CLS
```

즉 앞에서 배운 상태 위치, Debounce, 서버 상태, memoization이 결국 브라우저가 실제 화면을 만드는 비용과 사용자 경험 지표로 연결된다고 이해한다.

18. 성능 최적화의 전체 원칙은 다음처럼 기억한다.

```text
느리다
↓
코드부터 수정하지 않는다
↓
문제 행동 지정
↓
측정
↓
병목 확인
↓
원인 분석
↓
수정
↓
재측정
```

성능 개선을 설명할 때도 다음 순서를 유지하면 무엇을 왜 바꿨는지가 명확해진다.

```text
측정
→ 지표
→ 원인 추적
→ 해결
→ 재측정
```

## Quick Recall

| 도구 / 지표 | 핵심 질문 |
| --- | --- |
| React Profiler | React가 왜 많이 렌더링했지? |
| Chrome Performance | 브라우저가 어디서 오래 일했지? |
| Lighthouse | 페이지 품질에 어떤 문제가 있지? |
| LCP | 핵심 콘텐츠가 빨리 보이는가? |
| INP | 사용자 행동에 빨리 반응하는가? |
| CLS | 화면이 안정적인가? |

```text
Main Thread 오래 점유
→ Long Task
→ 입력 / 렌더링 / Paint 지연 가능
```

```text
Flame Chart
가로 → 실행 시간
세로 → 호출 관계
```

```text
INP가 나쁨
≠ 무조건 useMemo

측정
→ Input Delay / Event Handler / Render / Paint 확인
→ 원인에 맞는 해결책 선택
```

```text
Lab
→ 원인 찾기

Field
→ 실제 사용자 문제 확인
```

```text
Lighthouse
→ 문제 발견
→ Performance / Network로 원인 분석
→ 코드 수정
→ 다시 측정
```

## Open Questions

없음.

## Understanding Timeline

- 2026-09-10: 성능 문제를 코드만 보고 추측해서 고치기보다 문제 행동을 지정하고 측정 → 병목 확인 → 원인 분석 → 수정 → 재측정하는 흐름으로 확장했다.
- 2026-09-10: React Render/Commit 뒤에도 브라우저의 JavaScript → Style → Layout → Paint → Composite 과정이 이어진다는 흐름을 연결했다.
- 2026-09-10: React Profiler, Chrome Performance, Lighthouse, Core Web Vitals의 역할을 서로 다른 관점의 성능 도구와 지표로 구분했다.
- 2026-09-10: LCP는 로딩, INP는 상호작용 응답성, CLS는 시각적 안정성으로 연결하고 각 문제의 원인을 측정해서 해결책을 선택한다는 관점을 형성했다.
- 2026-09-10: Lab Data와 Field Data의 역할을 구분하고, Lighthouse 점수에서 실제 원인까지 내려가 분석한 뒤 느린 기기와 네트워크 조건에서도 다시 검증하는 흐름을 추가했다.

## Connections

- [React 메모이제이션과 참조 동일성](react/memoization-and-reference-equality.md) — React 내부의 렌더링과 계산 최적화를 실제 브라우저 Main Thread와 렌더링 비용 측정으로 확장하는 흐름으로 연결된다.
- [프론트엔드 번들·자원 로딩 최적화](bundle-and-resource-loading-optimization.md) — LCP·CLS·초기 JavaScript와 이미지 병목을 Bundle, Lazy Loading, 이미지 최적화, Prefetch/Preload 같은 실제 자원 로딩 전략으로 해결하는 흐름으로 이어진다.

## Sources

- 2026-09-10 Chrome Performance + Lighthouse + Core Web Vitals 6강 학습 자료
- 2026-09-10 사용자의 6강 학습 요약