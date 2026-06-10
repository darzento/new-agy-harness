---
name: oma-team
description: 다수의 에이전트(Planner -> Executor -> Verifier)가 참여하는 표준 파이프라인 협업 워크플로우를 가동합니다.
allowed-tools:
  - invoke_subagent
  - send_message
---

# OMA Team 워크플로우

이 스킬은 Antigravity CLI 환경에서 여러 서브 에이전트들이 단일 태스크를 달성하기 위해 `기획 -> 실행 -> 검증 -> 수정(피드백)`의 파이프라인 형태의 팀 협업을 시작하도록 트리거합니다.

## When to use this skill
- 사용자가 "oma-team", "팀", "협업" 등의 키워드로 작업을 지시할 때.
- 복잡한 구현이나, 확실한 테스트/검증 단계를 거쳐야 하는 메인 피쳐(Feature) 개발 작업을 시작할 때.

## Instructions
1. 오케스트레이터는 사용자의 목표를 바탕으로 팀 파이프라인을 준비합니다.
2. 각 서브 에이전트는 독립된 고유의 역할을 수행하며, JSON 통신 포맷을 준수해 상태를 보고해야 합니다.
3. 작업 중 발생하는 중간 문서와 코멘트는 반드시 프로젝트 루트의 `_workspace/` 디렉토리에 저장합니다.

## Workflow
### 1단계: Plan (기획)
오케스트레이터가 `omc-planner`를 기동(`invoke_subagent`)하여 사용자의 요구사항을 넘깁니다. 플래너가 기획안 작성을 마치고 `TASK_COMPLETE`를 반환할 때까지 대기합니다.

### 2단계: Execute (실행)
플래너의 산출물(PRD) 경로를 `omc-executor` 에이전트에게 전달하여 기동합니다. 코드 구현이 완료되어 `TASK_COMPLETE`를 반환하면 다음 단계로 이동합니다. (구현이 막히면 피드백 요청을 처리합니다)

### 3단계: Verify (검증)
`omc-verifier` 에이전트를 기동하여 수정한 코드가 기획안을 충족하는지 검증합니다.
- 만약 검증을 통과하여 `TASK_COMPLETE`가 반환되면 워크플로우를 최종 종료하고 사용자에게 완료를 보고합니다.
- 만약 검증에 실패하여 `REFACTOR_REQUEST`가 반환되면, 해당 피드백 리포트와 함께 다시 **2단계: Execute**로 되돌아가는 루프(Loop)를 가동합니다. (최대 재시도 횟수는 3회로 제한)
