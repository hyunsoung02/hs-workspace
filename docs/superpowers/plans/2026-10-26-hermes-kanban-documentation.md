# Hermes Kanban Documentation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 연구실 구성원이 기존 Delegate Task 방식과 Hermes Kanban 방식의 차이와 작업 흐름을 쉽게 이해할 수 있는 문서를 제공한다.

**Architecture:** 하나의 독립적인 Markdown 문서에 설명, 비교표, Mermaid 순서도 2개를 함께 둔다. README에는 기존 플러그인 표와 분리된 `작업 방식` 표를 추가해 문서로 연결한다.

**Tech Stack:** Markdown, GitHub Mermaid

**Spec:** 2026-10-26 Codex 대화에서 승인된 짧은 문서 설계

## Global Constraints

- 새 문서는 `docs/hermes/skills/kanban-board.md`에 작성한다.
- 기존 Delegate Task와 Hermes Kanban 흐름을 각각 Mermaid 순서도로 표현한다.
- Hermes Kanban과 GitHub Projects를 명확히 구분한다.
- `ains-lab/cag` 저장소 사용 범위는 확정 사실로 단정하지 않고 교수님 확인사항으로 표시한다.
- README 기존 표와 링크는 유지한다.

## Review Focus

- Mermaid 코드 블록의 문법이 GitHub에서 렌더링 가능한지 확인한다.
- 비교표가 정상적인 Markdown 표 형식인지 확인한다.
- Hermes Kanban과 GitHub Projects의 역할이 뒤섞이지 않았는지 확인한다.
- Dispatcher를 AI 에이전트로 잘못 설명하지 않았는지 확인한다.
- 확정되지 않은 `ains-lab/cag` 운영 방식을 사실처럼 표현하지 않았는지 확인한다.

---

### Task 1: Hermes Kanban 설명 문서 작성

**Files:**
- Create: `docs/hermes/skills/kanban-board.md`

**Interfaces:**
- Consumes: 사용자가 제공한 Hermes Kanban 구성요소와 작업 흐름
- Produces: 연구실 구성원이 읽을 독립 문서와 README 연결 대상

- [ ] **Step 1:** 기존 Delegate Task 방식, Hermes Kanban 방식, 구성요소, 역할, 비교표와 연구실 활용 구조를 작성한다.
- [ ] **Step 2:** Delegate Task와 Hermes Kanban 흐름을 Mermaid `flowchart TD` 문법으로 각각 작성한다.
- [ ] **Step 3:** GitHub Projects와 Hermes Kanban의 목적 차이와 함께 사용하는 방식을 명시한다.
- [ ] **Step 4:** Markdown 표와 Mermaid 블록의 시작·종료 구조를 확인한다.

### Task 2: README 문서 링크 추가

**Files:**
- Modify: `README.md`

**Interfaces:**
- Consumes: Task 1의 `docs/hermes/skills/kanban-board.md`
- Produces: README의 `작업 방식` 문서 링크

- [ ] **Step 1:** 기존 `플러그인` 표 아래에 `작업 방식` 표를 추가한다.
- [ ] **Step 2:** 링크가 `docs/hermes/skills/kanban-board.md`를 가리키는지 확인한다.
- [ ] **Step 3:** `git diff --check`와 링크 대상 파일 존재 여부를 확인한다.
