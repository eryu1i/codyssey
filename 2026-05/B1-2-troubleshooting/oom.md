[Bug] OOM - MemoryGuard 메모리 초과로 인한 프로세스 강제 종료

## 1. Description
`agent-app-leak` 실행 후 약 11초(MEMORY_LIMIT=100) 경과 시 
`SELF-TERMINATED` 메시지와 함께 프로세스가 예고 없이 종료됨.
애플리케이션 내부 메모리 보호 정책(MemoryGuard)에 의한 강제 종료가 반복적으로 발생함.

## 2. Evidence & Logs

[ monitor.sh 관제 데이터 ]
[2026-05-21 20:32:14] PID:270 271  CPU:0.9% MEM:8.6% DISK_USED:1%
[2026-05-21 20:32:18] PID:270 271  CPU:1.7% MEM:8.9% DISK_USED:1%
[2026-05-21 20:32:21] PID:270 271  CPU:0.9% MEM:9.3% DISK_USED:1%
[2026-05-21 20:32:25] PID:270 271  CPU:0%   MEM:9.7% DISK_USED:1%
[2026-05-21 20:32:28] PID:270 271  CPU:0.9% MEM:9.9% DISK_USED:1%
[2026-05-21 20:32:32] PID:270 271  CPU:0.9% MEM:10.6% DISK_USED:1%
[2026-05-21 20:32:35] PID:270 271  CPU:0.9% MEM:11.0% DISK_USED:1%
[2026-05-21 20:32:39] PID:270 271  CPU:0.9% MEM:11.3% DISK_USED:1%
→ MEM% 8.6% → 11.3% 지속 상승 패턴 확인

[ 앱 실행 로그 — Before (MEMORY_LIMIT=100) ]
2026-05-21 19:51:05 [INFO] Agent listening at port 15034
2026-05-21 19:51:07 [INFO] [MemoryWorker] Current Heap: 25MB
2026-05-21 19:51:10 [INFO] [MemoryWorker] Current Heap: 50MB
2026-05-21 19:51:13 [INFO] [MemoryWorker] Current Heap: 75MB
2026-05-21 19:51:16 [INFO] [MemoryWorker] Current Heap: 100MB
2026-05-21 19:51:16 [CRITICAL] [MemoryGuard] Memory limit exceeded (100MB >= 100MB)
2026-05-21 19:51:16 [CRITICAL] [MemoryGuard] Self-terminating process 225
>>> [SYSTEM] SELF-TERMINATED (Memory Limit Exceeded) <
→ 실행 후 약 11초 만에 강제 종료

[ 앱 실행 로그 — After (MEMORY_LIMIT=256) ]
2026-05-21 20:00:05 [INFO] Agent listening at port 15034
2026-05-21 20:00:07 [INFO] [MemoryWorker] Current Heap: 25MB
...
2026-05-21 20:00:38 [INFO] [MemoryWorker] Current Heap: 275MB
2026-05-21 20:00:38 [CRITICAL] [MemoryGuard] Memory limit exceeded (275MB >= 256MB)
→ 실행 후 약 33초 만에 강제 종료

## 3. Root Cause Analysis
- `agent-app-leak` 내부 MemoryWorker가 3초마다 25MB씩 힙 메모리를 할당하고
  해제하지 않는 메모리 누수(Memory Leak) 결함이 존재함.
- 물리 메모리 사용량이 `MEMORY_LIMIT`에 도달하면 MemoryGuard 정책이
  시스템 전체 불안정 방지를 위해 해당 프로세스를 강제 종료함.
- 메모리 누수는 힙(Heap) 영역에서 할당된 데이터가 GC(가비지 컬렉션) 또는
  명시적 해제 없이 누적되어 발생하는 전형적인 패턴임.

## 4. Workaround & Verification
- 조치: `MEMORY_LIMIT` 환경변수를 100MB → 256MB로 상향 조정
- Before: MEMORY_LIMIT=100, 약 11초 생존 후 종료
- After:  MEMORY_LIMIT=256, 약 33초 생존 후 종료
- 실제 운영 환경에서 이와 같은 메모리 누수가 발생한다면, 힙 메모리 할당 후 명시적 해제 또는 주기적 GC 호출 로직 추가가 필요함.