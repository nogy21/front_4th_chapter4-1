# 프론트엔드 배포 파이프라인

## 1. 개요

다음은 GitHub Actions을 활용한 S3 + CloudFront 배포 파이프라인을 나타낸 다이어그램입니다.

![스크린샷 2025-02-19 오전 12 30 49](https://github.com/user-attachments/assets/1990f22a-6438-4d4e-af04-3953c38a74ff)

이 배포 파이프라인은 다음과 같은 방식으로 동작합니다.

1. **개발자가 코드를 GitHub 저장소에 푸시**하면, GitHub Actions가 자동으로 실행됩니다.
2. GitHub Actions에서 **Next.js 프로젝트를 빌드**하고 정적 파일을 생성합니다.
3. 빌드된 파일을 **S3에 업로드**하고, CloudFront의 캐시를 무효화하여 최신 버전을 제공할 수 있도록 합니다.
4. 사용자는 CloudFront를 통해 **가장 가까운 엣지 서버에서 정적 콘텐츠를 빠르게 로드**할 수 있습니다.

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

### 배포 과정 (Step-by-Step)

GitHub Actions을 활용하여 `.github/workflows/deployment.yml`을 작성하고, 배포는 다음 단계로 진행됩니다.

#### **1. GitHub Actions (CI/CD 자동화)**

1. **저장소에서 최신 코드 체크아웃** (`Checkout step`)
2. **pnpm과 Node.js 18.x 버전 설치** (`Install pnpm, Setup Node.js step`)
3. **프로젝트 의존성 설치** (`Install dependencies step`)
4. **Next.js 프로젝트 빌드 (정적 파일 생성)** (`Build step`)

#### **2. AWS 서비스 (S3 + CloudFront 배포)**

5. **AWS 자격 증명 구성** (`Configure AWS credentials step`)
6. **S3에 빌드된 정적 파일 업로드** (`Deploy to S3 step`)
7. **CloudFront 캐시 무효화 (최신 파일 제공)** (`Invalidate CloudFront cache step`)

<br>

## 2. 주요 링크

- [S3 버킷 웹사이트 엔드포인트](http://h99-nogy21.s3-website.ap-northeast-2.amazonaws.com)
- [CloudFront 배포 도메인 이름](https://d29duocsuqn1b7.cloudfront.net)

<br>

## 3. 주요 개념

### GitHub Actions과 배포 자동화 (CD)

- **GitHub Actions**을 활용하여 코드가 `main` 브랜치에 푸시될 때 **자동으로 빌드 및 배포**가 실행됩니다.
- CI(Continuous Integration, 지속적 통합)는 포함되지 않으며, **CD(Continuous Deployment, 지속적 배포)만 수행**합니다.
- 개발자가 직접 서버에 배포할 필요 없이, **S3에 정적 파일이 업로드되고 CloudFront 캐시가 갱신됩니다.**

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

| 성능 지표                      | CDN 미적용 (S3 직접 제공) | CDN 적용 후 (CloudFront) | 개선율   | 설명                                       |
| ------------------------------ | ------------------------- | ------------------------ | -------- | ------------------------------------------ |
| TTFB (Time to First Byte)      | 98ms                      | 35.08ms                  | 🔽 64.2% | 첫 바이트 도달 시간                        |
| FCP (First Contentful Paint)   | 0.2s                      | 0.2s                     | 🚫 0%    | 첫 콘텐츠 렌더링 시간                      |
| LCP (Largest Contentful Paint) | 0.6s                      | 0.3s                     | 🔽 50%   | 최대 콘텐츠 렌더링 시간                    |
| TBT (Total Blocking Time)      | 480ms                     | 480ms                    | 🚫 0%    | 브라우저가 렌더링을 차단한 시간            |
| CLS (Cumulative Layout Shift)  | 0.005                     | 0                        | 🔽 100%  | 레이아웃 이동 지표 (0에 가까울수록 안정적) |
| SI (Speed Index)               | 0.8s                      | 0.4s                     | 🔽 50%   | 시각적 콘텐츠가 렌더링되는 속도            |

### 🚀 **주요 개선 사항**

- **TTFB(64.2% 개선)** → CloudFront의 캐싱으로 서버 응답 속도가 대폭 향상됨.
- **LCP(50% 개선)** → 주요 콘텐츠가 더 빨리 로드되어 사용자 경험이 향상됨.
- **CLS(100% 개선)** → 페이지 로딩 중 레이아웃 이동이 사라져 UX가 안정적임.
