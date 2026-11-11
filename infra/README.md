# infra | 클라우드 인프라 설계 기록

AWS 인프라를 설계하고 설정한 내용을 정리했습니다.
계정 번호, 엔드포인트, 비밀값은 올리지 않았습니다.

## 설계 원칙
- 외부에 여는 것은 API뿐이고, GPU 서버와 DB는 내부에서만 접근합니다.
- 원본 영상은 사람이 직접 열 수 없고, 처리 파이프라인만 접근합니다.
- 비밀번호는 코드나 설정 파일에 넣지 않고 Secrets Manager에 둡니다.
- 비용은 월 $10 GPU 예산 알림으로 관리합니다.

## 1. 영상 처리 환경 (us-east-1)

**네트워크**: VPC `shipyard-safety-vpc` (10.0.0.0/16)

| 구분 | CIDR | 가용 영역 |
|---|---|---|
| Public 서브넷 1 | 10.0.0.0/20 | us-east-1a |
| Public 서브넷 2 | 10.0.16.0/20 | us-east-1c |
| Private 서브넷 1 | 10.0.128.0/20 | us-east-1a |
| Private 서브넷 2 | 10.0.144.0/20 | us-east-1c |

- Public 라우팅 테이블은 인터넷 게이트웨이로 연결했습니다.
- Private 서브넷은 NAT 게이트웨이 없이 S3 엔드포인트로만 S3에 접근합니다.

**보안 그룹**

| 이름 | 인바운드 | 소스 | 용도 |
|---|---|---|---|
| api-sg | 80, 443 | 0.0.0.0/0 | 외부에서 API 접근 허용 |
| gpu-sg | 8000 | api-sg | API 서버에서만 GPU 서버 접근 |
| db-sg | DB 포트 | api-sg, gpu-sg | 애플리케이션 서버에서만 DB 접근 |

**원본 영상 보호 (S3)**
- 버킷을 `shipyard-safety-raw-video`(원본)와 `shipyard-safety-processed-video`(처리 후)로 분리했습니다.
- 버킷 정책으로 `video-processing-pipeline` 역할과 root 계정 외의 접근을 Deny합니다.
- 기본 암호화(SSE-S3)를 적용했습니다.
- **검증**: root는 접근되고 IAM 사용자는 거부되는 것을 확인했습니다.

**감사**: CloudTrail 추적 `shipyard-safety-trail`을 켰습니다.

## 2. 웹 서비스 환경 (서울)
- 프론트: CloudFront → S3(정적 파일)
- 백엔드: ALB → ECS Fargate(0.5 vCPU, 1GB, 포트 8080) → RDS MySQL
- 이미지는 ECR에 두고, 업로드 파일은 S3를 사용합니다.
- DB 비밀번호는 Secrets Manager에서 태스크 시작 시 주입합니다.
- 배포 절차는 [deployment.md](deployment.md)를 참고하세요.

## 3. 권한 관리 (IAM)
팀 역할별로 IAM 그룹 6개를 두고 사용자를 나눠 배치했습니다.

| 그룹 | 대상 |
|---|---|
| app-dev | 앱 개발 |
| be-group | 백엔드 |
| cloud | 클라우드 |
| data-modeling | 데이터 모델링 |
| fe-group | 프론트엔드 |
| pm-admin | PM, 관리자 |

루트 계정에는 MFA(패스키)를 적용했습니다.

## 알려진 한계
- IAM 사용자 계정에는 MFA를 적용하지 않았습니다.
- 프로젝트 종료 후 AWS 계정을 닫아 라이브 환경은 없습니다.
