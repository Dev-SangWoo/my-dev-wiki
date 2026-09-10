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

성능을 볼 때 `Main Thread`, `Long Task`, `Style Calculation`, `Layout`, `Paint`, `Composite`를 주요 포인트로 본다.

5. `Layout Thrashing`은 DOM의 크기나 위치를 읽고 쓰는 작업을 반복하면서 강제로 Layout 계산이 계속 발생하는 현상으로 이해한다.

```text
읽기
→ 쓰기
→ 다시 읽기
→ 다시 쓰기
→ Layout 반복
```

6. Chrome Performance는 막연하게 오래 녹화하기보다 실제 문제 행동을 정확하게 지정해서 측정해야 한다고 이해한다.

```text
문제 행동 지정
→ 측정
→ 시간이 많이 걸린 구간 확인
→ 원인 확인
→ 수정
→ 다시 측정
```

7. Core Web Vitals의 핵심은 다음 세 가지로 기억한다.

```text
LCP
→ 로딩
→ Largest Contentful Paint
→ 주요 콘텐츠가 보일 때까지
→ 좋은 기준 2.5초 이하

INP
→ 상호작용
→ Interaction to Next Paint
→ 사용자 행동 뒤 다음 시각적 피드백까지
→ 좋은 기준 200ms 이하

CLS
→ 시각적 안정성
→ Cumulative Layout Shift
→ 화면이 갑자기 밀리거나 흔들리는 정도
→ 좋은 기준 0.1 이하
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

그래서 INP가 좋지 않다고 무조건 `useMemo`를 붙이는 것이 아니라 먼저 Performance에서 어디가 느린지 확인해야 한다.

```text
Input Delay가 긴가?
Event Handler가 긴가?
Render가 긴가?
Paint가 긴가?
```

원인에 따라 `useMemo`, 서버 계산, Web Worker, 작업 분할, virtualization, 렌더링 범위 감소 등을 고려할 수 있다고 이해한다.

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

스켈레톤도 실제 콘텐츠와 크기 차이가 크면 CLS를 만들 수 있어서 다음처럼 기억한다.

```text
Skeleton의 형태
≈ 실제 Content의 형태
```

11. Lighthouse는 페이지를 자동으로 검사해서 Performance, Accessibility, SEO 같은 여러 품질 영역에 대한 보고서를 만드는 도구로 이해한다.

12. 성능 최적화의 전체 원칙은 다음처럼 기억한다.

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

## Quick Recall

```text
React Profiler
→ React가 왜 많이 렌더링했지?

Chrome Performance
→ 브라우저가 어디서 오래 일했지?

Lighthouse
→ 페이지 품질에 어떤 문제가 있지?

Core Web Vitals
→ LCP / INP / CLS로 사용자 경험 성능을 본다
```

```text
LCP → 로딩
INP → 상호작용
CLS → 시각적 안정성
```

## Open Questions

없음.

## Understanding Timeline

- 2026-09-10: 성능 문제를 코드만 보고 추측해서 고치기보다 문제 행동을 지정하고 측정 → 병목 확인 → 원인 분석 → 수정 → 재측정하는 흐름으로 확장했다.
- 2026-09-10: React Render/Commit 뒤에도 브라우저의 JavaScript → Style → Layout → Paint → Composite 과정이 이어진다는 흐름을 연결했다.
- 2026-09-10: React Profiler, Chrome Performance, Lighthouse, Core Web Vitals의 역할을 서로 다른 관점의 성능 도구와 지표로 구분했다.
- 2026-09-10: LCP는 로딩, INP는 상호작용 응답성, CLS는 시각적 안정성으로 연결하고 각 문제의 원인을 측정해서 해결책을 선택한다는 관점을 형성했다.

## Connections

- [React 메모이제이션과 참조 동일성](react/memoization-and-reference-equality.md) — React 내부의 렌더링과 계산 최적화를 실제 브라우저 Main Thread와 렌더링 비용 측정으로 확장하는 흐름으로 연결된다.

## Sources

- 2026-09-10 Chrome Performance + Lighthouse + Core Web Vitals 6강 학습 자료
- 2026-09-10 사용자의 6강 학습 요약
