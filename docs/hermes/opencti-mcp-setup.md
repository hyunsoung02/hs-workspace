# OpenCTI와 Hermes MCP 시연 환경 구축 기록

- 작업일 2026년 11월 8일
- 환경 연구실 시연 서버 `worker03`, Orca의 터미널과 브라우저, Hermes Agent
- 목표 공개 CTI를 OpenCTI에 적재하고, Hermes에서 기존 `opencti-mcp`를 통해 위협 객체와 관계를 읽기 전용으로 조회한다.
- 현재 상태 OpenCTI 설치·기동, 웹 로그인, 대시보드의 위협 데이터 표시까지 확인했다. Hermes의 MCP 조회와 APT29의 상세 관계는 검증 대기다.

## 1. 시연 구성과 선택 이유

~~~text
공개 MITRE ATT&CK 데이터 → OpenCTI
                             ↑ GraphQL 조회
Orca의 Hermes ── MCP ──→ opencti-mcp
~~~

OpenCTI는 위협 객체와 관계를 저장한다. 커뮤니티 `opencti-mcp`는 Hermes에 OpenCTI 조회 도구를 제공한다. Orca에서 사용하는 Superpowers는 설계·작업 순서·검증을 위한 개발 스킬이다. 첫 시연은 기존 MCP 서버로 실제 조회를 확인하고, 자체 CTI 조회 스킬은 필요할 때 같은 질문으로 적용 전후를 비교하는 범위로 잡았다. [OpenCTI MCP 패키지](https://pypi.org/project/opencti-mcp/) · [Superpowers 사용 환경](skills/superpowers.md)

## 2. 설치와 기동

1. 공식 [OpenCTI Docker 배포 저장소](https://github.com/OpenCTI-Platform/docker)를 `opencti-demo` 폴더로 내려받았다. 이 폴더명은 시연 환경을 구분하기 위해 붙였다.
2. `.env.sample`에서 `.env`를 만들고 OpenCTI 설정을 적용했다. `docker-compose.yml`은 본체, 검색 엔진, 메시지 큐, 저장소, Worker와 커넥터를 함께 실행하도록 정의한다. 컨테이너 이미지는 `docker compose up -d` 단계에서 내려받았다.
3. 아래 오류를 해결한 뒤 OpenCTI 대시보드까지 로그인했다. OpenCTI 로그에 표시된 버전은 `7.261106.0`이다.

공식 문서에서 확인할 수 있는 설치 절차와 변수는 [OpenCTI 설치 문서](https://docs.opencti.io/latest/deployment/installation/#using-docker)에 정리돼 있다.

## 3. 시행착오와 해결

| 증상 | 확인한 원인 | 조치와 확인 결과 |
|---|---|---|
| `xtm-rabbitmq-1 is unhealthy`, OpenCTI는 `Created` | RabbitMQ 로그의 `eacces`로 연결된 `rabbitmq.conf`를 읽지 못한 것을 확인했다. | 공식 기본 설정 파일 하나에 `chmod a+r rabbitmq.conf`로 읽기 권한을 추가했다. 즉시 실행한 대기 명령은 여전히 `unhealthy`였으나, 다음 기동 출력에서 RabbitMQ가 `Healthy`가 됐다. |
| `Bind for 0.0.0.0:8080 failed: port is already allocated` | 서버의 `8080`을 다른 서비스가 이미 사용 중이었다. 이후 점검에서 `.env`와 Compose의 유효 설정도 여전히 `8080`이었다. | `ss -ltn 'sport = :8081'`로 빈 포트를 확인한 뒤 `OPENCTI_PORT=8081`을 저장하고, Compose의 유효 설정과 새 컨테이너의 포트 매핑을 확인했다. |
| `getaddrinfo EAI_AGAIN rabbitmq` | 당시 OpenCTI가 Compose 내부에서 `rabbitmq` 이름을 조회하지 못했다. | RabbitMQ의 정상 상태를 확인하고, 같은 Compose 환경의 임시 OpenCTI 컨테이너에서 이름 조회가 성공하는지 검사했다. OpenCTI를 다시 띄운 뒤 앱 로그에서 `RabbitMq is alive`를 확인했다. 일시적인 이름 조회 실패의 원인까지는 확인하지 못했다. |
| `Token must be a valid UUID` | OpenCTI의 기본 관리자 초기화에 사용하는 `OPENCTI_ADMIN_TOKEN`이 UUID 형식에 맞지 않았다. | UUIDv4를 생성해 `.env`의 해당 항목만 갱신했다. 다음 기동에서는 UUID 오류를 지나 암호화 키 검사 단계까지 진행했다. |
| `app:encryption_key configuration is missing or invalid` | 로그에 나온 키 길이는 Base64 디코딩 후 22바이트였다. 요구 길이는 최소 32바이트다. | 암호학적으로 안전한 난수 32바이트를 Base64로 인코딩해 `OPENCTI_ENCRYPTION_KEY`를 갱신했다. 이후 OpenCTI 기동과 대시보드 로그인을 확인했다. |
| `.env`에서 관리자 정보를 바꾼 직후 로그인 실패 | `.env` 변경값은 실행 중인 컨테이너에 자동으로 들어가지 않는다. | Compose로 OpenCTI 설정을 다시 반영하도록 안내했고, 이후 대시보드에 로그인한 화면을 확인했다. 당시 로그인 실패의 세부 원인은 따로 기록되지 않았다. |
| Orca 브라우저에서는 열리는데 Mac Chrome의 `127.0.0.1:8081`은 `ERR_CONNECTION_REFUSED` | Mac에서 `127.0.0.1`은 Mac 자신을 가리킨다. Mac의 `8081`에 연결을 받는 프로그램이 없었다. SSH 예시의 자리표시자를 그대로 쓰거나 옵션만 따로 입력한 시도도 연결로 이어지지 않았다. | 시연 서버 IP의 `8081`로 접속하도록 안내한 뒤, Mac Chrome에서 접속 성공을 확인했다. |

포트는 `서버 8081 → 컨테이너 8080`으로 연결된다. `8081`은 기존 서비스와의 충돌을 피하려고 고른 값이다. Docker가 모든 호스트 주소에 포트를 연결한 상태에서 Mac의 접속은 확인했다. 방화벽·VPN을 포함한 실제 접근 범위와 랩의 포트 사용 정책은 별도 확인 항목이다. [Docker 포트 공개 문서](https://docs.docker.com/engine/network/port-publishing/) · [Compose 내부 서비스 이름 조회](https://docs.docker.com/compose/how-tos/networking/)

### 같은 증상을 다시 확인할 때

Compose 파일이 있는 폴더에서 서비스 상태, 실패한 서비스의 로그, 유효한 포트 설정 순서로 확인한다.

~~~bash
docker compose ps -a rabbitmq elasticsearch opencti
docker compose logs --no-color --tail=100 rabbitmq
docker compose logs --no-color --tail=100 opencti
docker compose config --environment | grep '^OPENCTI_PORT='
~~~

서비스 상태가 `unhealthy`로 끝나면 가장 최근 기동 시도의 첫 오류를 확인한다. `.env` 값은 Compose로 컨테이너에 반영한 뒤 확인하며, 공유하는 로그에는 토큰·비밀번호·내부 접속 정보가 포함되지 않도록 한다. [Docker Compose 로그](https://docs.docker.com/reference/cli/docker/compose/logs/) · [Compose 환경 변수 우선순위](https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation/)

## 4. 현재 확인된 결과

- Orca 브라우저와 Mac Chrome에서 OpenCTI 웹 화면에 접속했다.
- 로그인 뒤 대시보드에 `Intrusion Set` 185건과 `Malware` 746건이 표시됐고, 위협 목록에 APT29가 보였다. 이 숫자는 확인 당시의 화면 기준이다.
- APT29 상세 페이지의 관계, MITRE 데이터 전체 적재 여부, Hermes MCP 등록 및 실제 호출은 아직 확인하지 않았다.

## 5. 다음 검증

1. **OpenCTI에서 기준 결과 확보.** 상단 검색창에서 `APT29`를 찾아 `Intrusion Set` 상세 화면을 연다. 그룹 ID와 실제로 직접 연결된 공격 기법·악성코드의 ID, 관계 유형과 방향을 기록한다. [OpenCTI 검색 안내](https://docs.opencti.io/latest/usage/search/#global-search) · [MITRE ATT&CK APT29](https://attack.mitre.org/groups/G0016/)
2. **Hermes에 읽기 전용 MCP 등록.** 웹 Hermes의 MCP 설정에서 기존 `opencti-mcp`를 등록한다. 실행 환경에 `uvx`가 있다면 실행 명령은 `uvx`, 인자는 `--from opencti-mcp opencti-mcp`다. MCP 실행 환경에서 접근 가능한 OpenCTI 주소, 조회용 계정의 API 토큰, `OPENCTI_READ_ONLY=true`를 설정한다. [Hermes 웹 대시보드](https://hermes-agent.nousresearch.com/docs/user-guide/features/web-dashboard#mcp) · [opencti-mcp 설정](https://pypi.org/project/opencti-mcp/)
3. **실제 조회 비교.** Hermes에 “OpenCTI에서 APT29를 찾고, 직접 연결된 공격 기법과 악성코드를 확인된 ID 및 관계 유형과 함께 보여줘”라고 요청한다. 실제 MCP 호출 기록과 답변의 객체 ID·관계가 1단계 결과와 맞으면 통과다. 존재하지 않는 이름을 조회했을 때 결과 없음으로 답하는지도 확인한다.
4. **후속 구축 준비.** 시연에 사용한 버전과 설정 항목, 입력 질문, 기대 결과를 기록한 뒤 본 구축 환경에서 같은 시험을 다시 수행한다. 시연 서버에서 한 번 성공한 결과만으로 후속 환경의 안정성을 판단하지 않는다.
