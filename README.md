# 프론트엔드 배포 파이프라인

## 1. 개요

![스크린샷 2025-02-19 오전 12 30 49](https://github.com/user-attachments/assets/1990f22a-6438-4d4e-af04-3953c38a74ff)

### 사전 작업

1. AWS 인프라 세팅

   - S3 버킷 생성 및 정책 설정
   - CloudFront 생성 및 도메인 설정
   - IAM 설정

2. 저장소 세팅

   - Github 저장소와 Next.js 프로젝트 연동
   - 저장소 Secrets 설정
   - GitHub Actions 환경 설정
   - 저장소 워크플로우 설정

### 배포 과정

GitHub Actions에 워크플로우(`.github/workflows/deployment.yml`)를 작성해 다음과 같이 배포가 진행되도록 합니다.

1.  저장소를 체크아웃합니다. (`Checkout step`)
2.  pnpm과 Node.js 18.x 버전을 구성합니다. (`Install pnpm, Setup Node.js step`)
3.  프로젝트 의존성을 설치합니다. (`Install dependencies step`)
4.  Next.js 프로젝트를 빌드합니다. (`Build step`)
5.  AWS 자격 증명을 구성합니다. (`Configure AWS credentials step`)
6.  빌드된 정적 파일을 S3에 업로드합니다. (`Deploy to S3 step`)
7.  CloudFront 캐시를 무효화하여 최신 파일을 제공할 수 있도록 합니다. (`Invalidate CloudFront cache step`)

<br>

## 2. 주요 링크

- [S3 버킷 웹사이트 엔드포인트](http://h99-nogy21.s3-website.ap-northeast-2.amazonaws.com)
- [CloudFront 배포 도메인 이름](https://d29duocsuqn1b7.cloudfront.net)

<br>

## 3. 주요 개념

### GitHub Actions과 배포 자동화 (CD)

- **GitHub Actions**을 활용해 코드가 `main` 브랜치에 푸시될 때 자동으로 배포가 실행되도록 설정.
- **CD(Continuous Deployment, 지속적 배포) 파이프라인**을 통해 변경 사항이 감지되면 자동으로 S3에 업로드되고, CloudFront 캐시가 무효화됨.
- 빌드된 정적 파일이 자동으로 배포되므로, **개발자가 배포 과정을 직접 수행할 필요가 없음**.

### S3와 정적 사이트 배포

- **Amazon S3**(Simple Storage Service)는 정적 파일을 저장할 수 있는 객체 스토리지 서비스.
- Next.js에서 `output: "export"` 설정을 사용하면 **완전히 정적인 HTML, CSS, JavaScript 파일로 변환되며**, 이를 S3에 업로드하여 정적 웹사이트처럼 배포 가능.
- 그러나 **S3는 CDN 기능이 없기 때문에, 여러 지역에서 접근 시 속도가 느려질 수 있음**.

### CloudFront와 CDN (Content Delivery Network)

- **Amazon CloudFront**는 글로벌 엣지 로케이션(Edge Location)을 활용하여 사용자와 가까운 서버에서 캐시된 콘텐츠를 제공하는 CDN 서비스.
- **S3와 CloudFront의 차이점**:
  - S3는 **원본(origin) 서버**로, 정적 파일을 저장하지만 배포 최적화 기능이 없음.
  - CloudFront는 **CDN 역할**을 하며, 여러 위치에서 데이터를 캐싱하여 페이지 로딩 속도를 향상시킴.
- **주요 장점**:
  - 원격 사용자에게 **더 빠른 응답 시간** 제공 (지연 시간 단축).
  - 트래픽이 증가해도 부하를 분산하여 **서버 성능 최적화**.
  - 보안 기능 강화 (DDoS 방어, HTTPS 지원).

### 캐시 무효화(Cache Invalidation)

- CloudFront는 기본적으로 **정적 파일을 캐싱**하여 빠르게 제공하지만, 배포 이후 변경된 파일이 즉시 반영되지 않을 수 있음.
- 이를 해결하기 위해 **캐시 무효화(Cache Invalidation)를 수행하여 최신 파일을 제공할 수 있도록 함**.

### Repository Secret과 환경 변수

- AWS 액세스 키, S3 버킷, CloudFront ID 등의 보안 정보를 GitHub Secrets에 저장하여 보안성을 유지.

<br>

## 4. CDN 도입 전후 성능 비교

CDN을 도입한 후 페이지 로딩 속도 및 주요 성능 지표를 측정하여 비교했습니다.

| 성능 지표                      | CDN 미적용 (S3 직접 제공) | CDN 적용 후 (CloudFront) | 개선율 |
| ------------------------------ | ------------------------- | ------------------------ | ------ |
| TTFB (Time to First Byte)      |                           |                          |        |
| FCP (First Contentful Paint)   |                           |                          |        |
| LCP (Largest Contentful Paint) |                           |                          |        |
| Total Load Time                |                           |                          |        |
