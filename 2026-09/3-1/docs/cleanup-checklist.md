# 리소스 정리 체크리스트

실습 종료 후 과금 방지를 위해 아래 항목을 순서대로 확인하고 체크했습니다.

## 정리 순서 및 체크리스트

- [x] **1. EC2 인스턴스 종료(Terminate)**
  - 콘솔: EC2 → 인스턴스 → 인스턴스 상태 → 인스턴스 종료
  - 확인: 인스턴스 상태가 `Terminated`로 표시됨
  - 스크린샷/근거: `docs/screenshots/ec2-terminated.png`

- [x] **2. EBS 볼륨 삭제 확인 (미사용 볼륨 포함)**
  - 콘솔: EC2 → 볼륨(Volumes)
  - 확인: "You currently have no volumes in this region" 확인 (남은 볼륨 없음)
  - 스크린샷/근거: `docs/screenshots/ebs-cleaned.png`

- [x] **3. Elastic IP 릴리스 확인**
  - 콘솔: EC2 → 탄력적 IP
  - 확인: 실습 시작부터 Elastic IP를 별도로 할당하지 않았으며, 목록에도 아무 항목이 없음(과금 대상 없음)

- [x] **4. Internet Gateway 분리(Detach) 및 삭제**
  - 콘솔: VPC → 인터넷 게이트웨이
  - 확인: VPC에서 분리(Detach) 후 삭제 완료

- [x] **5. Route Table / Subnet 삭제**
  - 콘솔: VPC → 라우팅 테이블 / 서브넷
  - 확인: Public Subnet 삭제 완료. (Main Route Table은 AWS 정책상 개별 삭제 불가하며, VPC 삭제 시 자동 정리됨을 확인)

- [x] **6. Security Group 삭제**
  - 콘솔: VPC → 보안 그룹
  - 확인: 커스텀 보안 그룹(web-sg) 삭제 완료

- [x] **7. VPC 삭제**
  - 콘솔: VPC → VPC
  - 확인: VPC 및 하위 종속 리소스(Main Route Table 포함) 전체 삭제 완료

- [x] **8. Billing Dashboard 최종 확인**
  - 콘솔: Billing → 비용 관리 대시보드
  - 확인: 실행 중인 과금 리소스 없음, 프리티어 범위 내에서 정상 종료됨을 확인

---

## 최종 확인
- [x] 위 1~7번(필수 항목)까지 모두 완료되어 과금 위험 리소스가 남아있지 않음을 확인했습니다.
- [x] 확인 일시: `2026-09-17 18:48` 이후 정리 완료
