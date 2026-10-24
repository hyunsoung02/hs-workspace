# Hermes에 Superpowers 설치 및 슬래시 명령 설정

이 문서는 공용 `worker03` 서버에서 `hyunsoung` Hermes 프로필에 Superpowers를 설치하면서 발생한 문제와 해결 과정을 정리한 기록입니다.

## 1. 플러그인 설치

서버 터미널에서 사용할 프로필을 명시해 설치합니다.

```bash
hermes --profile hyunsoung plugins install obra/superpowers --enable
```

설치 후 실행 중인 Hermes를 종료하고 같은 프로필로 다시 실행합니다.

```bash
hermes --profile hyunsoung
```

## 2. 문제: Superpowers 스킬이 목록에 나타나지 않음

### 증상

플러그인은 설치됐지만 `/brainstorming`, `/writing-plans` 같은 스킬이 Hermes의 슬래시 명령 목록에 나타나지 않았습니다.

### worker03에서 확인한 원인

당시 `hyunsoung` 프로필의 `skills.external_dirs`에는 OMH 스킬 폴더만 등록돼 있었고, Superpowers 스킬 폴더는 포함돼 있지 않았습니다.

> 최신 Superpowers 플러그인은 Hermes의 네이티브 스킬 로더에 스킬을 직접 등록합니다. 따라서 아래 설정은 모든 환경에 필요한 기본 설치 과정이 아니라, 스킬이 검색되지 않았던 worker03 환경에서 사용한 해결 방법입니다.

### 해결

기존 OMH 경로를 유지하면서 Superpowers 경로를 함께 등록했습니다.

```bash
hermes --profile hyunsoung config set skills.external_dirs '["/home/ainsdev/.hermes/profiles/hyunsoung/omh/skills", "/home/ainsdev/.hermes/profiles/hyunsoung/plugins/superpowers/skills"]'
```

설정 확인:

```bash
hermes --profile hyunsoung config get skills.external_dirs
```

Hermes를 다시 실행하거나 현재 대화에서 스킬 목록을 새로 불러옵니다.

```text
/reload-skills
```

`/reload`는 `.env` 환경변수만 다시 읽는 명령이므로 스킬 갱신에는 `/reload-skills`를 사용합니다.

## 3. 문제: 원하는 슬래시 명령 형식과 다름

Hermes의 기본 스킬 명령은 스킬 이름을 그대로 사용합니다.

```text
/brainstorming
/writing-plans
/systematic-debugging
```

원하는 형식은 다음과 같았습니다.

```text
/superpowers-brainstorming
/superpowers-writing-plans
/superpowers-systematic-debugging
```

원본 스킬의 이름이나 파일을 수정하지 않고, Hermes의 스킬 번들 기능으로 새로운 명령을 추가했습니다.

```bash
hermes --profile hyunsoung bundles create superpowers-brainstorming \
  --skill superpowers:brainstorming \
  --description '구현 전 아이디어·요구사항을 질문으로 정리하고 설계에 합의합니다.'
```

연결 구조는 다음과 같습니다.

```text
/superpowers-brainstorming
        ↓
Hermes 스킬 번들
        ↓
superpowers:brainstorming 원본 스킬
```

따라서 `/brainstorming`의 이름을 바꾼 것이 아닙니다. 같은 원본 스킬을 불러오는 `/superpowers-brainstorming` 명령을 추가한 것입니다.

## 4. 적용 범위

현재 설치와 설정은 다음 범위에만 적용돼 있습니다.

```text
worker03 서버 → hyunsoung Hermes 프로필
```

같은 서버를 사용해도 다른 Hermes 프로필에 자동으로 적용되지는 않습니다. 다른 연구원이 사용하려면 각자의 프로필에서 다음 작업을 별도로 진행해야 합니다.

1. Superpowers 설치 및 활성화
2. Hermes 재실행
3. 슬래시 스킬 목록 확인
4. 필요한 경우에만 해당 프로필의 스킬 검색 경로 설정
5. `/superpowers-*` 형식을 원하면 해당 프로필에 번들 생성

플러그인 설치나 업데이트가 보안 검사에서 차단되면 경고 내용을 먼저 검토해야 합니다. 출처와 변경 내용을 확인하지 않은 상태에서 보안 검사를 우회하지 않습니다.

## 5. 최종 확인

Hermes에서 `/superpowers-`를 입력했을 때 생성한 명령과 한국어 설명이 자동완성 목록에 나타나는지 확인합니다.

예:

```text
/superpowers-brainstorming 로그인 기능의 요구사항을 정리해줘.
```
