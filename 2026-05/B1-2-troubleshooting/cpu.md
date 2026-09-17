[Bug] CPU 과점유 - Watchdog 보호 정책에 의한 프로세스 강제 종료

## 1. Description
`agent-app-leak` 실행 후 CpuWorker가 CPU 사용률을 지속적으로 높이다가
내부 Watchdog 임계치(50%)를 초과하는 순간 SIGTERM 신호와 함께
프로세스가 강제 종료됨.

## 2. Evidence & Logs

[ monitor.sh 관제 데이터 ]
[2026-05-21 21:06:05] PID:4449 4453  CPU:0.9% MEM:8.9% DISK_USED:1%
[2026-05-21 21:06:09] PID:4449 4453  CPU:0.9% MEM:8.8% DISK_USED:1%
[2026-05-21 21:06:12] PID:4449 4453  CPU:2.5% MEM:8.6% DISK_USED:1%
[2026-05-21 21:06:16] PID:4449 4453  CPU:1.7% MEM:8.8% DISK_USED:1%
[2026-05-21 21:06:19] PID:4449 4453  CPU:4.5% MEM:8.9% DISK_USED:1%
→ 시스템 전체 CPU 4.5% 상승 확인

[ 앱 실행 로그 — Before (CPU_MAX_OCCUPY=100) ]
2026-05-21 21:14:35 [INFO] [CpuWorker] Started. Maximum CPU Limit: 100%
2026-05-21 21:14:35 [INFO] [CpuWorker] Current Load: 5.00%
2026-05-21 21:14:38 [INFO] [CpuWorker] Current Load: 13.67%
2026-05-21 21:14:45 [INFO] [CpuWorker] Current Load: 18.66%
2026-05-21 21:14:48 [INFO] [CpuWorker] Current Load: 26.18%
2026-05-21 21:14:57 [INFO] [CpuWorker] Current Load: 35.09%
2026-05-21 21:15:00 [INFO] [CpuWorker] Current Load: 44.70%
2026-05-21 21:15:03 [INFO] [CpuWorker] Current Load: 49.83%
2026-05-21 21:15:06 [INFO] [CpuWorker] Current Load: 50.21%
2026-05-21 21:15:06 [CRITICAL] [CpuWorker] CPU Threshold Violated! (50.21%)
>>> [SYSTEM] WATCHDOG: INITIATING EMERGENCY ABORT (SIGTERM) <
Terminated

[ 특이사항 ]
시스템 전체 CPU(monitor.sh 기준)는 최대 4.5%로 낮게 측정됨.
이는 앱 내부 CpuWorker가 자체적으로 부하를 시뮬레이션하는 방식으로 동작하며,
실제 OS 레벨 CPU 점유와 차이가 있음. 앱 내부 로그가 해당 프로세스의
CPU 사용률을 더 정확하게 반영함.

[ 앱 실행 로그 — After (CPU_MAX_OCCUPY=20) ]
2026-05-21 21:19:55 [INFO] [CpuWorker] Started. Maximum CPU Limit: 20%
2026-05-21 21:19:55 [INFO] [CpuWorker] Current Load: 5.00%
2026-05-21 21:20:05 [INFO] [CpuWorker] Current Load: 16.29%
2026-05-21 21:20:07 [INFO] [CpuWorker] Peak reached (20.00%). Starting cooldown...
2026-05-21 21:20:11 [INFO] [CpuWorker] Current Load: 19.42%
2026-05-21 21:20:14 [INFO] [CpuWorker] Current Load: 12.73%
2026-05-21 21:20:16 [INFO] [CpuWorker] Cooldown complete. Resuming...
→ Watchdog 임계치(50%) 미달로 SIGTERM 없이 정상 운영

## 3. Root Cause Analysis
- CpuWorker는 CPU_MAX_OCCUPY 값을 목표치로 삼아 부하를 점진적으로 높임.
- CPU_MAX_OCCUPY=100일 때 CpuWorker가 50%를 초과하면서
  앱 내부 Watchdog 임계치(고정값 50%)를 위반함.
- Watchdog은 시스템 전체 안정성 보호를 위해 SIGTERM을 발생시켜 프로세스를 종료함.
- CPU_MAX_OCCUPY=20일 때는 목표치가 낮아 Watchdog 임계치에 도달하지 않으므로
  프로세스가 정상 운영됨.

## 4. Workaround & Verification
- 조치: CPU_MAX_OCCUPY 환경변수를 100 → 20으로 하향 조정
- Before: CPU_MAX_OCCUPY=100, CpuWorker 50.21% 도달 → Watchdog SIGTERM 종료
- After:  CPU_MAX_OCCUPY=20, CpuWorker 20%에서 cooldown 반복 → 정상 운영
- 실제 운영 환경에서 이와 같은 CPU 과점유가 발생한다면, CpuWorker의 부하 상승 로직에 상한선을 두거나 작업 단위를 분산 처리하는 방식으로 개선이 필요함.