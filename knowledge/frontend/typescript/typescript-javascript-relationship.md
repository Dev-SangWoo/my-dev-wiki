# TypeScript와 JavaScript의 관계

## Current Understanding

1. TypeScript는 JavaScript를 기반으로 한 언어이고, 대부분의 유효한 JavaScript 코드는 TypeScript 코드로도 취급할 수 있다고 이해한다. 다만 TypeScript 자체가 그대로 JavaScript 런타임에서 실행되는 것이라기보다, 일반적으로 JavaScript로 변환된 뒤 실행된다고 보는 것이 정확하다.

2. TypeScript는 JavaScript의 런타임 동작을 모델링하는 타입 시스템을 가지고 있고, 런타임에서 발생할 수 있는 오류를 가능한 한 코드 실행 전에 정적 타입 검사 단계에서 찾으려고 하는 언어라고 이해한다.

3. TypeScript의 타입 체커를 통과했다고 해서 런타임 오류가 절대 발생하지 않는 것은 아니다. 타입 시스템이 모든 런타임 상황을 완전히 보장하는 것은 아니다.

4. 타입 어노테이션은 TypeScript에게 개발자의 의도를 전달해 줄 수 있는 수단이라고 이해한다.

## Quick Recall

```text
JavaScript
→ TypeScript의 기반
→ 대부분의 유효한 JS 코드는 TS 코드로도 취급 가능

TypeScript
→ JS의 런타임 동작을 타입 시스템으로 모델링
→ 실행 전에 오류를 찾는 것을 목표로 함
→ 하지만 타입 체크 통과 = 런타임 오류 0 보장은 아님

타입 어노테이션
→ 개발자의 의도를 타입 체커에 전달하는 수단
```

## Open Questions

없음.

## Understanding Timeline

- 2026-09-16: TypeScript를 JavaScript와 완전히 별개의 실행 언어로 보기보다 JavaScript를 기반으로 정적 타입 검사를 더하는 언어로 이해했다.
- 2026-09-16: "JavaScript는 TypeScript로 구동된다"는 표현을, 대부분의 JavaScript 코드를 TypeScript 코드로 취급할 수 있고 TypeScript는 일반적으로 JavaScript로 변환되어 실행된다는 관점으로 교정했다.
- 2026-09-16: 타입 체커가 많은 오류를 실행 전에 찾을 수 있지만 런타임 오류 전체를 보장하지는 않는다는 한계를 함께 이해했다.

## Connections

없음.

## Sources

- 2026-09-16 사용자가 읽고 있는 TypeScript 관련 책의 아이템 1 요약
