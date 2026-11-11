# SHIP-SAY | AI 스마트 조선소 안전관리 플랫폼

KT AIVLE School 빅프로젝트 (AIVLE Team 25, 6인), **우수상**

작업 허가서, 안전 신고, PPE 점검, CCTV 감시, 디지털 트윈을 하나로 묶은 조선소 안전관리 웹 서비스입니다.

> **이 저장소는 팀 프로젝트의 코드 사본입니다.**
>
> 원본 저장소: [DDyodong/Ship-Say](https://github.com/DDyodong/Ship-Say)
>
> 작업 담당과 커밋 계정이 다른 부분이 있습니다. 클라우드와 배포는 제가 담당했고, 일부 커밋은 팀원 계정으로 올라갔습니다.

## 내 역할: Cloud, 디지털 트윈 화면

| 영역 | 내가 한 일 | 위치 |
|---|---|---|
| 클라우드 인프라 | 영상 처리 환경(VPC, 보안 그룹, 원본 영상 S3 접근 제한, CloudTrail)과 웹 서비스 환경(ECS Fargate, ALB, RDS, CloudFront) 구성, IAM 그룹 6개로 권한 분리 | [`infra/`](infra/) |
| 배포 자동화 | GitHub Actions + OIDC로 프론트(S3, CloudFront), 백엔드(ECR, ECS) 배포 | [`deploy-backend.yml`](.github/workflows/deploy-backend.yml), [`deploy-frontend.yml`](.github/workflows/deploy-frontend.yml) |
| 디지털 트윈 2D 지도 | Kakao Map API로 야드 지도 화면 구현 | [`KakaoYardMap.jsx`](frontend/src/components/digitalTwin/KakaoYardMap.jsx) |
| 3D 설비 상세 화면 | 설비 상세 3D 화면 구현 | [`FactoryDetailTwin.jsx`](frontend/src/pages/admin/FactoryDetailTwin.jsx) |

팀원이 만든 부분: 위험도 예측 모델, 백엔드(Spring Boot), 작업 허가서 RAG, 작업자 안전 상태 감지 등

## 알려진 한계
- 실시간 카메라는 연결하지 않고, 저장된 CCTV 데이터로 시뮬레이션했습니다.
- 한화오션이 보안구역이라 지도 좌표는 검증용 값입니다.
- 프로젝트 종료 후 AWS 계정을 닫아 라이브 데모는 없습니다.

---
# Safety AI Control

조선소 작업허가, 안전 신고, PPE 점검, 알림과 디지털 트윈 시나리오를 하나의 작업자와 관리자 웹에서 관리하는 PoC입니다.

## 현재 데이터 범위

| 구분 | 현재 상태 |
| --- | --- |
| 회원과 권한, 작업허가, 안전 신고, PPE 점검 | 백엔드와 DB API 연결 |
| 작업허가서 PDF와 SIMOPS 분석 | FastAPI 분석 서비스 연결 |
| 용접 로봇과 품질 데모 | 보유 CSV 데이터 재생 |
| 기타 공장 설비, 작업자 위치, 위험 예측 | 검증 데이터 연결, 현장 입력 스키마로 교체 가능 |

검증 화면은 `VALIDATION
