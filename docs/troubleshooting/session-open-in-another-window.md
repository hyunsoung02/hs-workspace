# 중복 새션 해결 방법

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
1.명령을 Hermes 입력창(hyunsoung >)에 입력한 것 같습니다. 일반 터미널에서 실행해야 합니다.
   1. Orca 상단의 +를 누릅니다.
   2. Terminal을 선택합니다.
   3. hyunsoung >가 아닌 일반 터미널에서 아래 짧은 명령을 실행하세요.

ps -u "$(whoami)" -o pid=,etime=,tty=,comm= | grep -i hermes | head -20

2. 정확히 확인하려면 일반 터미널에서 실행하세요.

ps -p 2428475 -o pid=,etime=,tty=,args=

readlink /proc/2428475/cwd
 
   결과가 다음 조건을 모두 만족하면 해당 프로세스가 맞을 가능성이 높습니다.
- Hermes TUI 명령
- hyunsoung 프로필
- test-orca-server 작업 폴더

3. Hermes TUI만 정확히 찾기 위해 일반 터미널에서 다음을 실행하세요.

ps -u "$(whoami)" -o pid=,ppid=,etime=,tty=,args= | grep '[u]i-tui/dist/entry.js'
   
   출력에서 실행 시간이 세션을 실행 시간을 찾으면 됩니다.
찾았습니다. 기존 채팅을 점유한 TUI는 다음 항목입니다.

PID: 2428530

실행 시간: 02:22:25

터미널: pts/12

부모 PID: 2428475

--삭제

kill 2428530

--종료 확인

ps -p 2428530

다른 연구원의 작업까지 종료될 수 있습니다. PID가 확실하지 않다면 종료하지 말고 서버 관리자에게 확인합니다.

## 재발 방지

- 같은 세션을 여러 Orca 작업창에서 동시에 열지 않습니다.
- 작업을 마칠 때는 창만 닫지 말고 Hermes에서 `/quit`으로 종료합니다.
- 연결이 끊긴 뒤 바로 `/resume`하지 말고 남은 TUI가 있는지 먼저 확인합니다.
