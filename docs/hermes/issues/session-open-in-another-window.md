# Hermes 세션 중복 오류 해결

`/resume`으로 기존 대화를 열 때 다음 메시지가 나타나면, 같은 대화가 다른 Hermes 창이나 터미널에서 사용 중일 수 있습니다.

```text
This chat is open in another Hermes window/terminal.
Use it there, or start a new chat here.
```

Orca 연결이 끊기거나 창이 비정상 종료돼도 서버에 Hermes 터미널 UI(TUI) 프로세스가 남을 수 있습니다.

## 1. 기존 창 확인

기존 창에 접근할 수 있으면 그곳에서 대화를 이어가거나 `/quit`으로 종료한 뒤 원하는 창에서 `/resume`을 실행합니다.

## 2. 서버에 남은 프로세스 확인

기존 창을 찾을 수 없다면 Orca에서 **+ → Terminal**을 선택합니다. 아래 명령은 Hermes의 채팅 입력창 대신 **Hermes가 실행 중인 Linux 서버의 일반 터미널**에서 실행합니다.

```bash
ps -u "$(whoami)" -o pid=,etime=,tty=,comm= | grep -i hermes | head -20
ps -u "$(whoami)" -o pid=,ppid=,etime=,tty=,args= | grep '[u]i-tui/dist/entry.js'
```

후보 PID를 골랐다면 명령줄, 실행 시간, 터미널과 작업 폴더를 확인합니다. 아래 명령을 실행하면 **현재 출력에서 확인한 PID**를 입력할 수 있습니다.

```bash
printf '확인할 PID: '
IFS= read -r pid
ps -p "$pid" -o pid=,ppid=,etime=,tty=,args=
readlink "/proc/$pid/cwd"
```

## 3. 확인된 세션 종료 후 복구

Hermes 프로필, 작업 폴더, 터미널이 모두 본인의 중복 세션과 일치할 때만 해당 프로세스를 종료합니다. 여러 사람이 같은 서버 계정을 쓰면 다른 사람의 작업이 같은 목록에 나타날 수 있습니다. PID가 확실하지 않으면 서버 관리자에게 확인합니다.

```bash
kill "$pid"
ps -p "$pid"
```

해당 PID가 더 이상 표시되지 않으면 Hermes에서 `/resume`을 다시 실행합니다.

## 재발 방지

- 같은 대화 세션은 한 Orca 작업창에서 사용합니다.
- 작업을 마칠 때 Hermes에서 `/quit`으로 종료합니다.
- 연결이 끊겼다면 `/resume` 전에 기존 TUI가 남아 있는지 확인합니다.
