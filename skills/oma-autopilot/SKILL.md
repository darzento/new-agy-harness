---
name: oma-autopilot
description: 최소한의 사용자 개입으로 기획부터 개발, 검증까지 완전 자율 주행(End-to-End) 방식으로 작업을 완료합니다.
allowed-tools:
  - invoke_subagent
  - send_message
---

# OMA Autopilot

Autopilot은 OMA Team 워크플로우를 극대화하여 사용자에게 중간 확인을 묻지 않고 끝까지 자율적으로 작업을 완수하는 원버튼(One-button) 스킬입니다.

## When to use this skill
- 사용자가 "autopilot", "자동", "알아서 다 해줘"와 같은 키워드로 완전 자율 실행을 원할 때.
- 목표는 명확하고 중간 피드백이 불필요한 단기적인 개발 과제(MVP 개발 등)를 수행할 때.

## Instructions
1. 이 모드에서는 Planner, Executor, Verifier 간의 통신에서 오케스트레이터가 사용자에게 승인(Approval)을 요청하지 않습니다.
2. 모든 의사 결정(예외 처리, 재시도, 리팩토링)은 에이전트들의 자율적인 피드백 루프를 통해 결정됩니다.

## Workflow
1. **Initialize**: 오케스트레이터가 태스크 아이디(Task ID)를 발급하고 `_workspace/` 하위에 자율 주행 상태 추적 로그를 생성합니다.
2. **Execute Pipeline**: `oma-team`의 Plan -> Execute -> Verify 단계를 자동으로 호출하되, 에러 발생 시 서브 에이전트의 피드백을 사용자 개입 없이 그대로 다음 에이전트에게 라우팅합니다.
3. **Auto-Correction**: Verifier가 실패를 선언하면 최대 5번까지 Executor에게 자동으로 버그 수정 루프를 강제 가동합니다.
4. **Final Reporting**: 모든 단계가 성공적으로 끝나거나, 최대 재시도 횟수를 초과한 경우에만 결과를 요약(Summary Artifact)하여 사용자에게 최종 보고합니다.
