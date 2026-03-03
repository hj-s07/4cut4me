# 4cut4me

클라우드를 활용한 네컷 사진·동영상 자동 저장 서비스입니다.  
QR 업로드만으로 사진/영상이 S3에 저장되며, 고가용성과 보안을 고려해 인프라를 구성했습니다.

<img src="https://github.com/user-attachments/assets/6f505b4f-144f-449c-8ffa-c31539536c5a" width="900" />

## 서비스 개요

- QR 업로드 기반 자동 저장(사진/영상)
- Load Balancer + Auto Scaling, RDS Multi-AZ 기반 고가용성 구성
- HTTPS 적용
- S3 VPC Endpoint 기반 내부 통신 경로 구성 및 접근 정책 설계
- CloudTrail 기반 S3 접근 로그 수집

<img src="https://github.com/user-attachments/assets/838f3669-c0db-49cb-ae73-b51021c57f90" width="900" />

## 아키텍처

- Route 53 + ACM(HTTPS) + Load Balancer
- EC2(Web/App) + Auto Scaling Group
- RDS(MySQL) Multi-AZ
- S3(이미지/영상 저장)
- S3 VPC Endpoint + Bucket Policy 조건 기반 접근 제어
- CloudTrail로 S3 접근 로그 수집

## 저장소 구성

- `app.py` : Flask 엔트리, Blueprint 등록 및 기본 라우팅/헬스체크
- `gallery.py` : QR 처리 → 파일 수집 → S3 업로드/삭제, 갤러리/검색/페이지네이션
- `photo_detail.py` : 상세 화면 라우팅
- `admin_user.py` : 사용자/관리 기능 관련 라우팅
- `templates/` : 화면 템플릿
- `DB/` : RDS 접근 및 메타데이터 처리
- `Dockerfile` : 컨테이너 실행 환경(Flask + headless Chrome 포함)

## 동작 흐름(요약)

1) 사용자가 QR 이미지를 업로드  
2) 서버에서 QR을 디코딩해 URL 추출  
3) headless Chrome으로 페이지 접근/다운로드 처리  
4) 파일을 S3에 업로드하고, 메타데이터는 RDS에 저장  
5) 갤러리에서 저장된 사진/영상 조회 및 삭제

## 로컬 실행

사전 준비: Docker Desktop(권장), Docker

```bash
# build
docker build -t 4cut4me .

# run (컨테이너 포트 80)
docker run --rm -p 8080:80 4cut4me

# health check
curl http://localhost:8080/health
```

## 설정/주의사항

S3 버킷/리전, RDS 접속정보 등은 실행 환경에 맞게 설정이 필요합니다.

운영에서는 환경변수(.env) 또는 Secret Manager/Parameter Store 등으로 민감정보를 분리하는 방식을 권장합니다.

## 사용 기술

Backend: Flask(Python)

Storage: S3 (boto3)

QR/Automation: OpenCV + Selenium(headless Chrome)

DB: RDS(MySQL, Multi-AZ)

Infra: VPC / ALB / Auto Scaling / HTTPS / VPC Endpoint / CloudTrail
