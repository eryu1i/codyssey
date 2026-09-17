[Analysis] 로그 패턴 분석을 통한 스케줄링 알고리즘 추론

## 1. 로그 관찰 개요
`agent-app-leak`의 정상 실행 상태(MULTI_THREAD_ENABLE=false)에서
Scheduler가 Thread-A, B, C를 처리하는 방식을 분석하여
적용된 스케줄링 기법을 역추론함.

## 2. 증거 자료
[ 앱 실행 로그 ]
2026-05-21 21:33:15,210 [INFO] [Scheduler] Registered Tasks: ['Thread-A', 'Thread-B', 'Thread-C']
2026-05-21 21:33:15,210 [INFO] [Thread-A] Task Started. Calculating... (20%)
2026-05-21 21:33:15,266 [INFO] [Thread-A] Calculating... (40%)
2026-05-21 21:33:15,323 [INFO] [Thread-A] Calculating... (60%)
2026-05-21 21:33:15,379 [INFO] [Thread-A] Calculating... (80%)
2026-05-21 21:33:15,436 [INFO] [Thread-A] Task Completed. (100%)
2026-05-21 21:33:15,490 [INFO] [Thread-B] Task Started. Calculating... (20%)
...
2026-05-21 21:33:15,711 [INFO] [Thread-B] Task Completed. (100%)
2026-05-21 21:33:15,767 [INFO] [Thread-C] Task Started. Calculating... (20%)
...
2026-05-21 21:33:15,989 [INFO] [Thread-C] Task Completed. (100%)

## 3. 패턴 분석 및 결론

**Round Robin 아님**
Thread-A가 100% 완료되기 전에 B나 C가 끼어든 적이 없음.
Round Robin이었다면 A 진행 중에 B, C가 번갈아 실행됐을 것.

**비선점(Non-preemptive) 방식 확실**
한 스레드가 완료되어야 다음 스레드가 시작됨.

**FCFS 또는 비선점 Priority 중 하나로 추론**
- Registered Tasks 로그가 우선순위 없이 단순 리스트 구조
  ['Thread-A', 'Thread-B', 'Thread-C']
- 우선순위 할당 로그(예: priority=1, 2, 3)가 존재하지 않음
- 등록된 순서대로 실행되는 FCFS에 더 가깝다고 추론하나,
  비선점 Priority(A>B>C)와 로그상 결과가 동일하여 완전한 확신은 불가

**최종 결론: FCFS(First Come First Served)로 추론**
바이너리 디컴파일이 금지된 환경에서 로그만으로 판단할 수 있는
최선의 근거로 FCFS를 채택함.

## 4. 장단점 및 적합한 아키텍처 분석

### FCFS (First Come First Served) — 추론 결과
| 구분 | 내용 |
|------|------|
| 장점 | 구현이 단순함, 기아(Starvation) 현상 없음 |
| 단점 | 앞 작업이 오래 걸리면 뒤 작업이 오래 대기(Convoy Effect) |

적합한 서비스: 처리량(Throughput)이 중요하고 응답 시간이 크게 중요하지 않은
배치 처리 서버(Batch Server)에 적합.

부적합한 서비스: 실시간 응답이 중요한 웹 서버나 게임 서버에는 부적합.
앞 요청이 오래 걸리면 뒤 요청이 그만큼 대기해야 하므로
사용자 경험(UX)이 크게 저하될 수 있음.

---

### Round Robin
| 구분 | 내용 |
|------|------|
| 장점 | 모든 작업이 공평하게 CPU를 나눠 가짐, 응답 시간이 균등함 |
| 단점 | Context Switching 비용 발생, Time Quantum 설정에 따라 성능 차이 큼 |

적합한 서비스: 실시간 응답이 중요한 웹 서버, 대화형 시스템.
여러 사용자 요청이 동시에 들어와도 골고루 처리되어
특정 요청이 오래 기다리지 않음.

부적합한 서비스: 작업 단위가 크고 처리량이 중요한 배치 서버.
Context Switching 오버헤드가 누적되어 전체 처리량이 떨어질 수 있음.

---

### Priority (우선순위)
| 구분 | 내용 |
|------|------|
| 장점 | 중요한 작업을 먼저 처리할 수 있음, 긴급 요청 즉시 대응 가능 |
| 단점 | 우선순위가 낮은 작업이 계속 밀려 영원히 실행 안 될 수 있음(Starvation) |

적합한 서비스: 긴급도가 다른 작업이 혼재하는 시스템.
예를 들어 OS 커널의 인터럽트 처리, 실시간 제어 시스템처럼
반드시 먼저 처리해야 하는 작업이 있는 경우에 적합.

부적합한 서비스: 모든 요청이 동등하게 처리되어야 하는 서비스.
우선순위가 낮은 사용자 요청이 계속 밀릴 수 있어
공평성이 중요한 환경에는 부적합.