# Superpowers 스킬 실행하기

## 설치와 활성화

개인 Hermes 프로필에 설치합니다.

```bash
hermes --profile hyunsoung plugins install obra/superpowers --enable
```

설치 후 실행 중인 Hermes를 종료하고 같은 프로필로 새로 엽니다.

```bash
hermes --profile hyunsoung
```

## 스킬 이름 확인

Hermes에서 스킬 목록을 확인합니다.

```text
/help skills
```

또는 자연어로 `사용 가능한 superpowers 스킬 목록을 보여줘`라고 요청합니다.

## 가장 확실한 실행 방법

Superpowers 스킬은 보통 `superpowers:스킬-이름` 형식입니다.

```text
/skill superpowers:writing-plans 이 작업의 구현 계획을 작성해줘
```

설치된 Hermes 버전이 네임스페이스 슬래시 명령을 지원한다면 다음도 사용할 수 있습니다.

```text
/superpowers:writing-plans 이 작업의 구현 계획을 작성해줘
```

`/superpowers-writting plan`은 등록된 명령 형식이 아니며 `writing` 철자도 다릅니다.

## `/superpower...` 자동완성이 안 뜰 때

1. 현재 프로필이 `hyunsoung`인지 `/status`로 확인합니다.
2. 플러그인이 설치·활성화됐는지 플러그인 목록에서 확인합니다.
3. Hermes를 완전히 종료한 뒤 같은 프로필로 다시 엽니다.
4. 정확한 스킬 이름을 `/help skills`에서 확인합니다.
5. 슬래시 자동완성이 없어도 `/skill superpowers:writing-plans ...`로 직접 실행합니다.

`muse plugins ...`는 다른 CLI의 명령입니다. `zsh: command not found: muse`가 나오면 Hermes 플러그인 관리에는 사용하지 않습니다.
