# Recall Knowledge Workflow

사용자가 "이거 어디서 봤는데", "예전에 내가 뭐라고 이해했지?"처럼 과거 지식을 다시 찾으려 할 때 사용한다.

항상 루트 `AGENTS.md`의 Core Rules를 우선한다.

## Procedure

1. 사용자의 표현에서 기억 단서를 추출한다. 정확한 기술 용어를 기억하지 못해도 의미를 기준으로 탐색한다.
2. `INDEX.md`에서 관련 domain과 후보 문서를 좁힌다.
3. 파일명, `Current Understanding`, `Connections`, `Open Questions`, `Understanding Timeline`을 검색한다.
4. 가장 관련 있는 소수의 지식 파일을 읽는다. 처음부터 저장소 전체를 무분별하게 읽지 않는다.
5. 답변에서는 일반 지식보다 **사용자가 과거에 어떻게 이해했는지**를 우선해서 복원한다.
6. 필요한 경우 현재 질문과 연결되는 다른 지식 파일을 `Connections`를 따라 함께 읽는다.
7. 찾지 못했다면 존재하지 않는 기억을 만들어내지 말고 저장된 기록에서 찾지 못했다고 명확히 말한다.

## Retrieval priorities

1. 현재 사용자의 이해 (`Current Understanding`)
2. 관련 개념과의 연결 (`Connections`)
3. 아직 남아 있는 의문 (`Open Questions`)
4. 과거 이해 변화 (`Understanding Timeline`)
5. 학습 맥락 (`Sources`)

목표는 문서를 보여주는 것이 아니라, 사용자가 다시 생각을 이어갈 수 있도록 필요한 기억을 되돌려주는 것이다.
