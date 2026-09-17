# 트러블슈팅 보고서

실습 중 발생한 오류를 증상 → 가설 → 검증 → 조치 → 결과 → 재발방지 순서로 기록합니다.

---

## 사례 1: `sudo apt update` 실행 실패

| 항목 | 내용 |
|---|---|
| **증상(문제 상황)** | EC2 인스턴스에 SSH로 접속한 뒤 `sudo apt update`를 실행했을 때 정상적으로 진행되지 않고 실패함(패키지 목록을 가져오지 못함). 이후 `sudo apt install -y nginx`를 실행하자 `nginx.service does not exist` 에러로 이어져, nginx 자체가 설치되지 않았음을 확인함 |
| **원인 가설** | (1) 네트워크(Route Table의 IGW 경로 또는 Security Group 아웃바운드 설정) 문제로 외부 패키지 저장소에 접근하지 못했을 가능성, (2) 또는 APT 저장소 서버 응답 지연이나 일시적인 네트워크 지연으로 인한 타임아웃 가능성 |
| **검증 방법** | Public Subnet의 Route Table에 `0.0.0.0/0 → Internet Gateway` 경로가 정상적으로 설정되어 있는지 재확인. 이후 `sudo apt update` 명령을 다시 한 번 실행해봄 |
| **조치 내용** | 네트워크 설정 자체는 정상임을 확인한 상태에서, `sudo apt update` 명령을 재실행함 |
| **결과** | 재실행 시 패키지 목록을 정상적으로 가져왔고, 이어서 `sudo apt install -y nginx` 및 `sudo systemctl enable --now nginx` 명령이 정상적으로 동작하여 nginx가 설치·실행됨. `curl http://localhost` 요청에서 200 응답(Nginx 기본 페이지) 확인 |
| **재발 방지** | 첫 명령 실행 시 즉시 실패로 단정하지 않고, 네트워크 연결 상태(Route Table, IGW 연결)를 먼저 확인한 뒤 동일 명령을 재시도하는 절차를 표준화. 인스턴스 생성 직후에는 `sudo apt update`가 일시적으로 지연되거나 실패할 수 있으므로, 실패 시 바로 재시도해보는 것을 체크리스트에 추가 |

---

> 📌 이후 실습 과정(외부 접속 검증, 리소스 정리)에서는 추가적인 오류 없이 정상 진행되었습니다.
