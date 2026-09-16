# TypeScript tsconfig와 엄격한 타입 검사

## Current Understanding

1. TypeScript 설정은 `tsconfig.json`으로 관리할 수 있다고 이해한다.

2. `noImplicitAny`는 TypeScript가 타입을 추론하지 못해서 암묵적으로 `any`가 되는 경우를 허용하지 않는 옵션이라고 이해한다. 이 옵션이 `true`라고 해서 모든 변수에 타입 어노테이션을 직접 작성해야 하는 것은 아니고, TypeScript가 타입을 정상적으로 추론할 수 있다면 타입을 직접 적지 않아도 된다. 핵심은 묵시적인 `any`를 막는 것이다.

3. `strictNullChecks`는 `null`과 `undefined`를 다른 타입에 그냥 포함되는 값처럼 다루지 않고, 각각 하나의 타입으로 구분해서 확인하는 옵션이라고 이해한다. `true`일 때 값에 `null`이 들어올 수 있다면 `number | null` 같은 식으로 타입에 명시해야 한다.

4. 값이 `null`이나 `undefined`가 아니라고 확실히 알고 있다면 `!`로 타입 체커에게 확언할 수도 있다고 이해한다. 다만 `!`는 런타임에서 값을 검사해 주는 기능이 아니라 개발자가 타입 체커에게 안전하다고 보장하는 표현이다.

5. `noImplicitAny`와 `strictNullChecks`를 포함한 여러 엄격한 타입 검사를 한꺼번에 적용하려면 보통 `strict: true`를 사용한다고 이해한다.

6. `noUncheckedIndexedAccess`는 객체나 배열을 인덱스로 접근할 때 실제로 그 값이 존재한다고 보장할 수 없는 경우를 더 엄격하게 확인하는 옵션이라고 이해한다. 이런 접근 결과에 `undefined` 가능성을 포함시켜 접근 오류를 놓치지 않도록 도와준다.

## Quick Recall

```text
tsconfig.json
→ TypeScript 컴파일러/타입 검사 설정

noImplicitAny
→ 타입을 직접 안 썼다고 무조건 오류가 아님
→ 추론 실패로 암묵적 any가 되는 경우를 오류로 처리

strictNullChecks
→ null / undefined를 독립적인 타입으로 다룸
→ 필요하면 number | null처럼 명시

!
→ null/undefined가 아니라고 타입 체커에게 확언
→ 런타임 검사는 아님

strict
→ noImplicitAny, strictNullChecks 등을 포함한 strict 계열 검사 활성화

noUncheckedIndexedAccess
→ 인덱스로 접근한 값이 없을 가능성까지 고려
→ 필요한 경우 undefined 가능성을 타입에 포함
```

## Open Questions

없음.

## Understanding Timeline

- 2026-09-16: `noImplicitAny`를 "모든 변수에 타입을 직접 작성해야 하는 옵션"으로 보던 표현을, 타입 추론이 실패해 암묵적으로 `any`가 되는 경우를 금지하는 옵션으로 교정했다.
- 2026-09-16: `strictNullChecks`를 통해 `null`과 `undefined`가 별도의 타입으로 취급되고, 필요한 경우 유니온 타입으로 명시해야 한다는 흐름을 이해했다.
- 2026-09-16: non-null assertion `!`가 런타임 검사가 아니라 타입 체커에 대한 개발자의 확언이라는 점을 구분했다.
- 2026-09-16: 개별 엄격 옵션과 `strict`의 관계, 그리고 인덱스 접근을 더 안전하게 검사하는 `noUncheckedIndexedAccess`를 함께 연결했다.

## Connections

- [TypeScript와 JavaScript의 관계](typescript-javascript-relationship.md) — TypeScript가 실행 전에 오류를 찾으려는 타입 시스템이라는 관점이 실제로 어떤 엄격도 설정을 통해 적용되는지 연결된다.

## Sources

- 2026-09-16 사용자가 읽고 있는 TypeScript 관련 책의 아이템 2 요약
- TypeScript 공식 TSConfig Reference (`noImplicitAny`, `strictNullChecks`, `strict`, `noUncheckedIndexedAccess`)로 사실 관계 확인
