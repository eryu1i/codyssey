# AWS 웹 서비스 배포 실습

## 1. 개요
Amazon Web Service(AWS)의 서울 리전(ap-northeast-2)에 VPC 기반 네트워크를 구성하고,
Public Subnet의 EC2 인스턴스에 Nginx 웹 서버를 배포하여 외부에서 접속 가능한 상태로 운영한 실습입니다.

## 2. 아키텍처
전체 구성도는 [`docs/architecture.png`](./docs/architecture.png) 참고.

- VPC: `10.0.0.0/16`
- Public Subnet: `10.0.1.0/24`
- Internet Gateway → Route Table(`0.0.0.0/0 → IGW`) 연결
- EC2: t2.micro / t3.micro (Ubuntu LTS), Nginx 설치
- Security Group: HTTP(80) 전체 허용, SSH(22) 내 IP만 허용

## 3. 외부 접속 검증

- **선택한 검증 방식**: (A) 브라우저 접속
- **접속 정보**: `http://43.203.196.210`
- **접속 결과**: 정상적으로 "Welcome to nginx!" 페이지 확인 (200 OK)

| 검증일시 | 방식 | URL | 응답 |
|---|---|---|---|
| 2026-09-17 18:48 | A | http://43.203.196.210 | 200 OK |

스크린샷: `docs/screenshots/access-proof.png`

## 4. 네트워크 구성 상세

| 항목 | 값 |
|---|---|
| Region | ap-northeast-2 (서울) |
| VPC CIDR | 10.0.0.0/16 |
| Public Subnet CIDR | 10.0.1.0/24 |
| Route Table | 0.0.0.0/0 → Internet Gateway |
| 인스턴스 타입 | t2.micro / t3.micro |
| OS | Ubuntu LTS |
| EBS 크기 | 8~10GiB |

## 5. 보안 그룹 규칙

| 방향 | 프로토콜 | 포트 | 소스 | 설명 |
|---|---|---|---|---|
| Inbound | TCP | 80 | 0.0.0.0/0 | HTTP (웹 서비스) |
| Inbound | TCP | 22 | 학습자 개인 IP/32 | SSH (관리자 전용) |
| Outbound | ALL | ALL | 0.0.0.0/0 | 기본 아웃바운드 |

> 0.0.0.0/0에 대한 전체 포트(0-65535) 허용 규칙은 생성하지 않았습니다.

## 6. IAM 최소 권한
- 사용 계정: IAM 사용자 (루트 계정 미사용)
- 부여 권한: AmazonEC2FullAccess, AmazonVPCFullAccess (EC2/VPC/Security Group 관련 작업 중심)
- AdministratorAccess 및 실습과 무관한 서비스(S3, RDS 등) 권한 미부여

## 7. 폴더 구조
```
.
├── README.md
└── docs/
    ├── architecture.png
    ├── troubleshooting.md
    ├── cleanup-checklist.md
    └── screenshots/
        ├── access-proof.png
        ├── ec2-terminated.png
        └── ebs-cleaned.png
```

## 8. 리소스 정리
실습 종료 후 EC2, EBS, Elastic IP, Internet Gateway, VPC까지 모두 정리 완료.
상세 내역은 [`docs/cleanup-checklist.md`](./docs/cleanup-checklist.md) 참고.
