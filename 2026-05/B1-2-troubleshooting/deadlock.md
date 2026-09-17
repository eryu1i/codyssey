[Bug] Deadlock - 멀티스레드 환경에서 교착상태 발생으로 프로세스 무응답

## 1. Description
`MULTI_THREAD_ENABLE=true` 설정 시 `agent-app-leak` 실행 후
프로세스가 종료되지 않고 PID가 유지되나, CPU/메모리 변화가 없고
로그 출력도 완전히 멈춘 무응답 상태가 지속됨.

## 2. Evidence & Logs

[ PID 존재 확인 — 프로세스 살아있음 ]
agent-a+  5428  ... /home/agent-admin/agent-app/agent-app-leak
agent-a+  5429  5428 ... /home/agent-admin/agent-app/agent-app-leak
→ 두 프로세스 모두 종료되지 않고 존재

[ top -H -p 5428,5429 — 스레드 CPU/MEM 정체 ]
PID   USER     %CPU  %MEM  COMMAND
5428  agent-a+  0.0   0.1  agent-app-leak
→ CPU 0.0%, MEM 변화 없음. 아무런 작업도 수행하지 않음

[ 앱 실행 로그 마지막 기록 ]
2026-05-21 21:24:18 [WARNING] [AgentWorker] Initializing concurrent transaction processors...
2026-05-21 21:24:18 [WARNING] [System] CAUTION: Strict resource locking is enabled.
2026-05-21 21:24:23 [INFO] [Worker-Thread-1] LOCK ACQUIRED: [Shared_Memory_A]. (Holding...)
2026-05-21 21:24:23 [INFO] [Worker-Thread-2] LOCK ACQUIRED: [Socket_Pool_B]. (Holding...)
2026-05-21 21:24:25 [INFO] [Worker-Thread-1] WAITING for [Socket_Pool_B]... (Status: BLOCKED)
2026-05-21 21:24:25 [INFO] [Worker-Thread-2] WAITING for [Shared_Memory_A]... (Status: BLOCKED)
→ 이후 로그 출력 완전히 멈춤

## 3. Root Cause Analysis
수집된 로그를 근거로 Deadlock 4대 조건이 모두 성립함을 확인:

| 조건 | 근거 |
|------|------|
| 상호 배제 | Shared_Memory_A, Socket_Pool_B 각각 하나의 스레드만 점유 가능 |
| 점유 대기 | Thread-1이 A 점유한 채 B 대기, Thread-2가 B 점유한 채 A 대기 |
| 비선점 | 상대방 자원을 강제로 가져오지 못하고 무한 대기 |
| 순환 대기 | Thread-1 → B 대기 → Thread-2 점유 → A 대기 → Thread-1 점유 → 순환 |

결론: Worker-Thread-1과 Worker-Thread-2가 서로의 자원을 무한히
기다리는 순환 대기 구조로 인해 교착상태 발생.

## 4. Workaround & Verification
- 조치: MULTI_THREAD_ENABLE 환경변수를 true → false로 변경
- Before (true):  Worker-Thread-1, 2 BLOCKED → 무응답 지속
- After  (false): Thread-A, B, C 순차 완료 → 정상 동작

[ After 정상 실행 로그 ]
2026-05-21 21:33:15 [INFO] [Thread-A] Task Started. Calculating... (20%)
2026-05-21 21:33:15 [INFO] [Thread-A] Task Completed. (100%)
2026-05-21 21:33:15 [INFO] [Thread-B] Task Started. Calculating... (20%)
2026-05-21 21:33:15 [INFO] [Thread-B] Task Completed. (100%)
2026-05-21 21:33:15 [INFO] [Thread-C] Task Started. Calculating... (20%)
2026-05-21 21:33:15 [INFO] [Thread-C] Task Completed. (100%)
2026-05-21 21:33:16 [INFO] [Scheduler] All tasks completed.
→ 멀티스레드 비활성화 시 교착상태 없이 정상 완료

- 실제 운영 환경에서 이와 같은 교착상태가 발생한다면, 모든 스레드에서 락 획득 순서를 동일하게 통일하거나(Lock Ordering), 일정 시간 후 락을 자동 해제하는 타임아웃 로직 추가가 필요함.