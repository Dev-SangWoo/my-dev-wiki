# 프론트엔드 번들·자원 로딩 최적화

## Current Understanding

1. 브라우저가 화면을 보여주는 흐름을 대략 다음처럼 연결해서 이해한다.

```text
HTML 다운로드
↓
CSS 다운로드
↓
JavaScript 다운로드
↓
JavaScript 파싱
↓
JavaScript 실행
↓
React 실행
↓
화면 렌더링
```

JavaScript가 크면 다운로드만 오래 걸리는 것이 아니라 압축 해제, 파싱, 실행 비용도 커질 수 있다.

```text
큰 JavaScript
→ 다운로드 비용
→ 압축 해제 비용
→ 파싱 비용
→ 실행 비용
→ 첫 화면이 늦어질 수 있음
```

그래서 번들 크기는 단순히 네트워크 용량만의 문제가 아니라고 이해한다.

2. Bundle은 브라우저에 전달하기 위해 묶인 **내 코드 + 라이브러리 코드들의 묶음**으로 이해한다.

초기 번들에 첫 화면에서 사용하지 않는 차트, 에디터, 지도, 관리자 기능 같은 코드까지 모두 들어 있으면 사용자는 필요하지 않은 코드까지 처음부터 다운로드하고 처리해야 한다.

```text
초기 번들이 큼
+
첫 화면에서 안 쓰는 코드도 포함
→ 초기 로딩 비용 증가
```

3. Bundle Analyzer는 최종 번들에 무엇이 얼마나 들어 있는지 확인해서 **왜 번들이 큰지**와 **초반에 필요 없는 코드가 들어 있는지** 찾는 도구로 이해한다.

큰 라이브러리를 발견하면 바로 삭제하는 것이 아니라 다음 기준으로 본다.

```text
1. 정말 많이 쓰는 라이브러리인가?
2. 첫 화면에 필요한가?
3. 특정 페이지에서만 쓰는가?
4. 더 작은 대체제가 있는가?
5. 개별 import가 가능한가?
6. dynamic import로 늦게 불러와도 되는가?
```

즉 크기만 보는 것이 아니라 **왜 초기 번들에 있어야 하는지**를 판단한다.

4. `Code Splitting`, `Lazy Loading`, `Dynamic Import`는 연결되어 있지만 역할은 다르게 본다.

```text
Code Splitting
→ 큰 JavaScript 번들을 여러 chunk로 나누는 것

Lazy Loading
→ 나눠진 코드를 필요한 순간까지 불러오지 않는 것

Dynamic Import
→ 런타임에 필요한 모듈을 불러올 수 있게 하는 방법
```

짧게 기억하면:

```text
Code Splitting → 나누기
Lazy Loading   → 늦게 가져오기
import()       → 필요할 때 동적으로 불러오기
```

5. React에서는 `React.lazy`와 `Suspense`를 이용해서 컴포넌트를 필요한 순간에 불러올 수 있다고 이해한다.

```tsx
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(
  () => import('./HeavyChart'),
);

function Dashboard() {
  return (
    <Suspense fallback={<p>차트 로딩 중...</p>}>
      <HeavyChart />
    </Suspense>
  );
}
```

머릿속 흐름은 다음과 같다.

```text
Dashboard 렌더
↓
HeavyChart가 실제 렌더링에 필요해짐
↓
HeavyChart chunk 요청
↓
다운로드 중 Suspense fallback 표시
↓
완료 후 HeavyChart 표시
```

`React.lazy`를 "다른 작업이 전부 끝난 뒤 실행"하는 것으로 보기보다 **해당 컴포넌트가 실제 렌더링에 필요해지는 순간 코드가 요청된다**고 이해한다.

`Suspense`의 `fallback`은 그 코드를 기다리는 동안 보여줄 대체 UI다.

6. Next.js에서는 `next/dynamic`으로 동적 import를 사용할 수 있다고 이해한다.

```tsx
import dynamic from 'next/dynamic';

const HeavyEditor = dynamic(
  () => import('./HeavyEditor'),
  {
    loading: () => <EditorSkeleton />,
  },
);
```

`ssr: false`는 브라우저 전용 코드처럼 서버에서 렌더링할 수 없는 기능을 클라이언트에서만 렌더링하도록 할 때 사용할 수 있다.

```text
ssr: false
→ 서버에서는 렌더링하지 않음
→ 클라이언트 JavaScript가 실행된 뒤 렌더링
```

따라서 첫 화면 핵심 콘텐츠에 무조건 적용하는 옵션으로 보지 않는다.

7. Lazy Loading 후보는 **첫 화면에 꼭 필요한가?**를 기준으로 본다.

좋은 후보:

```text
차트
지도
Rich Text Editor
PDF Viewer
결제 모듈
관리자 전용 기능
큰 Modal 내부 콘텐츠
사용자가 클릭해야 보이는 영역
```

좋지 않은 후보:

```text
Header
첫 화면 주요 제목
Hero 영역
핵심 버튼
LCP 이미지
항상 필요한 작은 컴포넌트
```

머릿속에서는 다음 질문으로 시작한다.

```text
첫 화면에 꼭 필요한가?
```

8. Code Splitting은 많을수록 좋은 것이 아니다. 너무 잘게 쪼개면 chunk와 네트워크 요청이 지나치게 많아지고 관리도 복잡해질 수 있다.

```text
1개 너무 큰 번들
→ 문제일 수 있음

너무 많은 작은 chunk
→ 이것도 문제일 수 있음
```

그래서 **의미 있는 기능 단위**로 나누는 것을 우선한다.

라우트 단위로 나누는 `Route-based Splitting`도 자연스러운 기준이라고 이해한다.

```text
/          → Home 코드
/dashboard → Dashboard 코드
/admin     → Admin 코드
/editor    → Editor 코드
```

사용자가 방문하지 않는 라우트의 코드를 처음부터 받을 필요가 없다는 생각이다.

9. `Tree Shaking`은 사용하지 않는 코드를 빌드 결과에서 제거하는 최적화로 이해한다.

다만 라이브러리 구조나 import 방식에 따라 잘 되지 않을 수 있어서 import 방식도 본다.

특히 아이콘 몇 개만 필요한데 라이브러리 전체를 가져오는 식의 import는 번들 크기를 키울 수 있다.

```text
필요한 코드만 import 가능한가?
→ Bundle Analyzer와 함께 확인
```

10. 이미지도 초기 성능에서 매우 큰 비중을 차지할 수 있기 때문에 JavaScript만 줄이고 끝내지 않는다.

이미지 최적화는 다음 기준으로 본다.

### 실제 크기

```text
실제 표시 크기
≈ 전달할 이미지 크기
```

작게 표시할 이미지를 지나치게 큰 원본 크기로 전달하지 않는다.

### 반응형 이미지

모바일과 데스크톱에서 필요한 이미지 크기가 다르면 화면 크기에 맞는 이미지를 제공하는 것을 고려한다.

```text
모바일
→ 더 작은 이미지

데스크톱
→ 필요한 만큼 큰 이미지
```

`srcset`, `sizes` 같은 방식을 통해 브라우저가 적절한 이미지를 고를 수 있다.

### 이미지 포맷

용도에 따라 포맷을 다르게 본다.

```text
사진
→ WebP / AVIF 고려

투명도가 필요
→ PNG / WebP 등 고려

로고 / 아이콘
→ SVG 고려
```

특정 포맷 하나가 모든 상황의 정답이라고 보지는 않는다.

### 이미지 Lazy Loading

첫 화면 아래쪽처럼 당장 필요하지 않은 이미지는 lazy loading 후보다.

```text
첫 화면 아래 상품 목록
블로그 하단 이미지
갤러리 뒷부분
→ lazy loading 고려
```

반대로 첫 화면 핵심 LCP 이미지는 lazy loading을 주의한다.

```text
첫 화면 핵심 이미지
→ 빨리 로딩

화면 아래 이미지
→ lazy loading 고려
```

또 `width`, `height`, `aspect-ratio` 등을 이용해 이미지가 로드되기 전 공간을 미리 확보해서 CLS를 줄이는 것도 같이 본다.

11. `Prefetch`와 `Preload`는 가져오는 시점과 목적이 다르다고 이해한다.

```text
Preload
→ 현재 페이지에 꼭 필요한 리소스
→ 높은 우선순위로 미리 가져오기

Prefetch
→ 나중에 필요할 가능성이 있는 리소스
→ 여유가 있을 때 미리 준비하기
```

예:

```text
Preload 후보
→ 중요 폰트
→ LCP 이미지
→ 현재 페이지 핵심 CSS

Prefetch 후보
→ 다음 페이지
→ 다음 탭
→ 사용자가 곧 들어갈 가능성이 높은 상품 상세 코드나 데이터
```

Prefetch도 많이 할수록 좋은 것이 아니다.

```text
사용 가능성이 높은가?
리소스가 큰가?
정말 곧 필요할까?
현재 중요한 요청을 방해하지 않을까?
```

를 보고 판단한다.

12. Lazy Loading과 Prefetch는 동작은 반대처럼 보이지만 목적은 같다고 이해한다.

```text
Lazy Loading
→ 당장 필요 없는 것은 늦게 가져오기

Prefetch
→ 곧 필요할 가능성이 높은 것은 미리 가져오기
```

둘 다 결국:

```text
사용자에게 필요한 순간에
필요한 자원이 준비되어 있게 만들기
```

라는 목표로 연결된다.

13. Lazy Loading으로 기다리는 시간이 생기는 경우 `Suspense`의 Skeleton이나 fallback UI도 성능 UX의 일부로 본다.

좋은 Skeleton은:

```text
1. 실제 콘텐츠 크기와 비슷함
2. 사용자가 무엇이 로딩 중인지 이해할 수 있음
3. CLS를 크게 만들지 않음
```

즉 숫자만 줄이는 것이 아니라 **핵심 콘텐츠를 먼저 보여주고 부가 콘텐츠는 나중에 보여주는 방식**으로 사용자가 느끼는 속도도 본다.

14. 성능 최적화는 항상 Trade-off가 있다고 이해한다.

| 최적화 | 장점 | 단점 / 비용 |
| --- | --- | --- |
| Lazy Loading | 초기 번들 감소 | 나중에 기다림 발생 가능 |
| Prefetch | 다음 화면이 빨라질 수 있음 | 사용하지 않을 리소스 다운로드 가능 |
| Image Compression | 이미지 용량 감소 | 화질 저하 가능 |
| Code Splitting | 초기 JavaScript 감소 | chunk 관리 복잡, 추가 네트워크 요청 |

그래서 성능 최적화는 **최대한 많이 적용하는 것이 아니라 상황에 맞게 균형을 잡는 것**이라고 이해한다.

15. 전체 자원 로딩 구조를 다음처럼 연결해서 본다.

```text
사용자 진입
│
├─ HTML
├─ CSS
├─ JavaScript
│   ├─ Bundle
│   ├─ Code Splitting
│   └─ Dynamic Import
│
├─ Image
│   ├─ Size
│   ├─ Format
│   ├─ Responsive
│   └─ Lazy Loading
│
└─ 미래 자원
    ├─ Prefetch
    └─ Preload
```

그리고 자원을 언제 가져올지 판단할 때는 다음처럼 생각한다.

```text
첫 화면에 반드시 필요한가?
│
├─ YES
│   → 빠르게 로딩
│   → preload / 즉시 로딩 / 리소스 최적화 고려
│
└─ NO
    ↓
    곧 사용할 가능성이 높은가?
    │
    ├─ YES
    │   → prefetch 고려
    │
    └─ 확실하지 않음
        → lazy loading 고려
```

## Quick Recall

```text
Bundle Analyzer
→ 왜 초기 번들이 큰지 확인

Code Splitting
→ 나누기

Lazy Loading
→ 필요할 때까지 늦추기

Dynamic Import
→ 런타임에 코드 불러오기

Tree Shaking
→ 안 쓰는 코드 제거
```

```text
preload
→ 지금 꼭 필요

prefetch
→ 곧 필요할 가능성

lazy
→ 지금은 필요하지 않음
```

```text
이미지
→ 실제 표시 크기
→ 반응형 크기
→ 적절한 포맷
→ 아래쪽은 lazy 고려
→ LCP 이미지는 빠르게
→ 공간 미리 확보해서 CLS 주의
```

```text
최적화
≠ 최대한 많이 적용

= 초기 비용과 나중 비용 사이에서
  사용자에게 필요한 순간을 기준으로 균형 잡기
```

## Open Questions

없음.

## Understanding Timeline

- 2026-09-11: 초기 로딩 성능을 JavaScript 번들, 이미지, 앞으로 필요할 자원의 로딩 시점 문제로 나눠 보고, Code Splitting·Lazy Loading·Prefetch·Preload를 `언제 자원을 가져올 것인가`라는 하나의 판단 흐름으로 연결했다.
- 2026-09-11: 성능 최적화를 많이 적용하는 것이 목적이 아니라 초기 로딩 비용과 이후 대기·네트워크 비용 사이의 Trade-off를 상황에 맞게 조절하는 것이라고 이해했다.

## Connections

- [브라우저 성능 측정과 Core Web Vitals](browser-performance-and-core-web-vitals.md) — 6강에서 측정한 LCP·CLS·초기 JavaScript/이미지 병목을 실제 Bundle, Lazy Loading, 이미지 최적화와 자원 로딩 전략으로 해결하는 흐름으로 이어진다.

## Sources

- 2026-09-11 Bundle 최적화 + Lazy Loading + Dynamic Import + 이미지 최적화 + Prefetch 7강 학습 자료
- 2026-09-11 사용자의 7강 학습 요약
