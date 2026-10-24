# Orca와 worker03 원격 서버 연결

## 연결 구조

```text
내 Mac의 Orca → OpenVPN → 연구실 worker03 → 원격 Orca/Hermes 실행
```

Orca 로그인 계정, Linux 서버 계정, Hermes 프로필은 서로 다른 개념입니다.

## 필요한 정보

- 연구실 VPN 설정 파일
- 서버의 SSH 설정: `HostName`, `User`, `IdentityFile`
- 서버에서 생성한 Orca 액세스 링크: `orca://pair?code=...`

서버 IP와 Hermes 아이디·비밀번호만으로는 액세스 링크를 만들 수 없습니다. 액세스 링크는 서버에서 Orca 원격 서비스를 실행한 뒤 생성해야 합니다.

## 기본 연결 순서

1. Mac에서 OpenVPN을 연결합니다.
2. 터미널에서 교수님이 준 SSH 별칭으로 서버 접속을 확인합니다.

   ```bash
   ssh worker03
   ```

3. 서버에서 안내받은 방식으로 Orca 원격 서비스를 실행하고 액세스 링크를 생성합니다.
4. Mac의 Orca에서 **서버 추가**를 열고 `orca://pair?code=...` 전체를 붙여 넣습니다.
5. 서버 이름을 알아보기 쉽게 지정하고 연결합니다.

액세스 링크는 비밀번호처럼 취급하고 GitHub, 메신저, 화면 캡처에 공개하지 않습니다.

## SSH 설정 예시

`~/.ssh/config` 예시입니다. 실제 값은 교수님이 준 값으로 바꿉니다.

```sshconfig
Host worker03
  HostName <WORKER03_IP>
  User <SSH_USER>
  IdentityFile ~/.ssh/<PRIVATE_KEY_FILE>
```

`IdentityFile~/...`처럼 붙여 쓰지 말고 `IdentityFile` 뒤에 공백을 넣어야 합니다.

## SSH 터널이 필요한 경우

연구실 안내가 로컬 포트 전달 방식일 때만 사용합니다.

```bash
ssh -N -L <LOCAL_PORT>:127.0.0.1:<REMOTE_PORT> worker03
```

포트 번호는 임의로 정하지 말고 연구실에서 지정한 값을 사용합니다. 이 터미널을 닫으면 터널도 종료됩니다.

## 연결 실패 점검

1. OpenVPN이 연결되어 있는지 확인합니다.
2. `ssh worker03`이 되는지 확인합니다.
3. 서버의 Orca 원격 서비스가 실행 중인지 확인합니다.
4. 오래된 페어링 링크라면 서버에서 새 링크를 발급합니다.
5. Mac 재시작 뒤에는 VPN과 SSH 터널을 다시 연결합니다.
