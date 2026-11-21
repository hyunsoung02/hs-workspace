# Orca SSH 공개키 자동 인증

macOS의 SSH 에이전트와 키체인을 사용해 Orca에서 원격 서버에 접속합니다. 모든 터미널 명령은 **로컬 맥**에서 실행합니다. 예시의 `your-user@your-ssh-host`를 실제 계정과 호스트로 바꿉니다. 예시는 기본 SSH 포트 `22`를 사용합니다.

## 1. SSH 키 확인 또는 생성

```bash
ls -l ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub
```

두 파일이 있으면 기존 키를 사용합니다. 두 파일이 모두 없으면 다음 명령을 실행합니다. 저장 위치에서 Enter를 누르고 키 암호를 두 번 입력합니다. 기존 키를 덮어쓸지 묻는다면 `n`을 입력합니다.

개인키만 남아 있다면 `ssh-keygen -y -f ~/.ssh/id_ed25519 > ~/.ssh/id_ed25519.pub`로 공개키를 복구할 수 있습니다.

```bash
ssh-keygen -t ed25519
```

`id_ed25519`는 로컬에 보관하는 개인키입니다. 서버의 `~/.ssh/authorized_keys`에는 `.pub`으로 끝나는 공개키만 등록합니다.

## 2. 서버에 공개키 등록

```bash
ORCA_SSH_TARGET='your-user@your-ssh-host'
ssh "$ORCA_SSH_TARGET" '
  umask 077
  mkdir -p ~/.ssh &&
    chmod 700 ~/.ssh &&
    touch ~/.ssh/authorized_keys &&
    chmod 600 ~/.ssh/authorized_keys &&
    IFS= read -r public_key &&
    {
      grep -Fqx -- "$public_key" ~/.ssh/authorized_keys ||
        printf "\n%s\n" "$public_key" >> ~/.ssh/authorized_keys
    }
' < ~/.ssh/id_ed25519.pub
```

등록 중 암호를 묻는다면 서버 계정 비밀번호를 입력합니다. 위 명령은 같은 공개키 줄이 이미 있으면 다시 추가하지 않습니다.

## 3. 맥의 SSH 에이전트에 키 추가

```bash
/usr/bin/ssh-add --apple-use-keychain ~/.ssh/id_ed25519
/usr/bin/ssh-add -l
```

이 단계에서 묻는 암호는 키를 생성할 때 설정한 키 암호입니다. `Identity added`가 나오면 에이전트에 키가 추가된 상태입니다. `--apple-use-keychain`은 키 암호를 macOS 키체인에도 저장합니다. `ssh-add -l`은 에이전트에 올라간 키의 지문을 보여줍니다.

같은 터미널에서 아래 명령을 실행합니다. `SSH_KEY_OK`가 나오면 암호 입력이나 기존 SSH 연결 재사용 없이 지정한 키로 새 접속을 열 수 있습니다.

```bash
ssh -i ~/.ssh/id_ed25519 \
  -o IdentitiesOnly=yes -o PreferredAuthentications=publickey \
  -o BatchMode=yes -o ControlPath=none \
  "$ORCA_SSH_TARGET" 'echo SSH_KEY_OK'
```

## 4. Orca에서 저장하고 연결 테스트

1. **설정 → SSH 호스트**에서 대상을 추가하거나 기존 호스트를 편집합니다.
2. 호스트, 사용자 이름, 포트를 입력합니다. 로컬 SSH 에이전트를 사용하려면 **개인 키 파일**은 비워 둔다.
3. **변경 사항 저장**을 누른다.
4. 호스트 목록에서 **연결 테스트**를 실행합니다. `SSH 비밀번호` 창 없이 테스트가 성공하면 설정이 완료된 것이다.
5. Orca를 완전히 종료한 뒤 다시 열고 연결을 확인합니다. 앱을 열 때 바로 접속하려면 해당 호스트에서 **시작 시 다시 연결**을 켠다.

## 5. 비밀번호 창이 다시 뜰 때

| 증상 | 확인할 항목 |
| --- | --- |
| `SSH 비밀번호` 또는 호스트의 비밀번호를 묻는다 | Orca의 호스트·사용자·포트가 터미널에서 성공한 대상과 같은지 확인합니다. 개인 키 파일 칸을 비운 상태로 저장했는지 확인하고, `ssh-add -l`에 키가 표시되는지 확인한 뒤 연결 테스트를 다시 실행합니다. |
| `Passphrase` 또는 개인 키 암호를 묻는다 | 키를 생성할 때 정한 암호다. Orca가 암호로 보호된 키 파일을 직접 읽는 경우, 기본적으로 앱을 종료하면 기억한 암호가 지워진다. 에이전트에 키를 추가하고 개인 키 파일 칸을 비운 상태에서 다시 테스트합니다. |
| 맥에 다시 로그인한 뒤 `ssh-add -l`에 키가 없다 | 3단계의 `ssh-add` 명령을 다시 실행한 뒤 Orca에서 연결 테스트합니다. |
| 다른 PC에서도 같은 서버에 키로 접속하려 한다 | 그 PC에서 별도 키를 만들고 해당 공개키를 서버에 추가합니다. |

## 참고

- [Orca Docs — SSH worktrees](https://orca.dev/docs/ssh)
- [GitHub Docs — macOS에서 SSH 키를 에이전트에 추가하기](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=mac)
- [OpenSSH — sshd와 authorized_keys](https://man.openbsd.org/sshd.8)
