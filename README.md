# Orca + Hermes 사용 기록

연구실의 공용 Linux 서버에 Orca로 접속하고, 개인 Hermes 프로필을 사용하는 방법과 문제 해결 기록입니다.

> 보안: 실제 서버 IP, 계정 비밀번호, SSH 개인 키, `orca://pair?code=...` 링크는 저장하지 않습니다.

## 빠른 문제 찾기

| 증상 | 확인할 문서 |
|---|---|
| Orca에서 worker03에 처음 연결하고 싶다 | [Orca와 원격 서버 연결](docs/setup/orca-worker03-connection.md) |
| `hyunsoung`이 서버 이름인지 프로필인지 헷갈린다 | [Hermes 프로필 이해하기](docs/setup/hermes-profiles.md) |
| `This chat is open in another Hermes window/terminal` 오류가 뜬다 | [중복 세션 오류 해결](docs/troubleshooting/session-open-in-another-window.md) |
| Superpowers를 설치했는데 `/superpower...`가 안 뜬다 | [Superpowers 스킬 실행](docs/troubleshooting/superpowers-slash-command.md) |
| 플러그인 설치가 보안 검사에서 차단된다 | [플러그인 보안 검사](docs/troubleshooting/plugin-security-scan.md) |

## 카테고리

### 환경 구축

- [Orca와 원격 서버 연결](docs/setup/orca-worker03-connection.md)
- [Hermes 프로필 이해하기](docs/setup/hermes-profiles.md)

### 문제 해결

- [중복 세션 오류 해결](docs/troubleshooting/session-open-in-another-window.md)
- [Superpowers 스킬 실행](docs/troubleshooting/superpowers-slash-command.md)
- [플러그인 보안 검사](docs/troubleshooting/plugin-security-scan.md)

### 보안

- [공용 서버 안전 수칙](docs/security/shared-server-safety.md)

## 새 문제 추가 방법

1. [문제 기록 템플릿](ISSUE_TEMPLATE.md)을 복사합니다.
2. 알맞은 `docs/` 하위 폴더에 새 Markdown 파일을 만듭니다.
3. 이 README의 표와 카테고리에 링크를 추가합니다.
4. 비밀번호, 토큰, IP, 개인 키 경로, 페어링 링크가 없는지 확인합니다.
