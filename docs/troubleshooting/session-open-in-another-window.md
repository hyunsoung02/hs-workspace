# `This chat is open in another Hermes window/terminal` 해결

## 증상

`/resume`으로 기존 대화를 열 때 다음 오류가 나타납니다.

```text
This chat is open in another Hermes window/terminal.
Use it there, or start a new chat here.
```

## 원인

같은 대화 세션을 잡고 있는 Hermes TUI 프로세스가 서버에 남아 있기 때문입니다. Orca 연결이 끊기거나 창이 비정상 종료되면 화면은 사라져도 서버 프로세스는 계속 살아 있을 수 있습니다.

토큰이 0인 것, VPN을 다시 연결한 것, Hermes 프로필 이름은 이 오류의 직접 원인이 아닙니다.

## 가장 안전한 해결 순서

1. 예전 Hermes 창에 접근할 수 있으면 그 창에서 `/quit`으로 종료합니다.
2. 접근할 수 없으면 worker03 터미널에서 내 계정의 Hermes TUI 후보만 확인합니다.

   ```bash
   ps -u "$(whoami)" -o pid=,ppid=,etime=,tty=,args= | grep '[u]i-tui/dist/entry.js'
   ```

3. 오류에 나온 세션 시작 시각과 후보의 실행 시간·터미널(`TTY`)을 비교합니다.
4. 정확히 일치하는 프로세스 하나를 확인한 뒤 종료합니다.

   ```bash
   kill <CONFIRMED_TUI_PID>
   ```

5. 종료 여부를 확인합니다.

   ```bash
   ps -p <CONFIRMED_TUI_PID>
   ```

6. Hermes를 다시 열고 기존 세션을 선택합니다.

   ```text
   /resume
   ```

## 하지 말아야 할 것

공용 계정에서는 아래와 같이 모든 Hermes 프로세스를 한꺼번에 종료하지 않습니다.

```bash
killall hermes
pkill -f hermes
```

다른 연구원의 작업까지 종료될 수 있습니다. PID가 확실하지 않다면 종료하지 말고 서버 관리자에게 확인합니다.

## 재발 방지

- 같은 세션을 여러 Orca 작업창에서 동시에 열지 않습니다.
- 작업을 마칠 때는 창만 닫지 말고 Hermes에서 `/quit`으로 종료합니다.
- 연결이 끊긴 뒤 바로 `/resume`하지 말고 남은 TUI가 있는지 먼저 확인합니다.
