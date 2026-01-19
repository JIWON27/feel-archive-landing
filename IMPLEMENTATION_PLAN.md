# Feel-Archive Implementation Plan

> **문서 버전**: 1.0
> **작성일**: 2026-01-19
> **프로젝트**: Feel-Archive (공간 기반 감정 아카이빙 플랫폼)

---

## 1. 프로젝트 개요

### 1.1 현재 상태
| 항목 | 상태 |
|------|------|
| 프론트엔드 | 정적 HTML 랜딩 페이지만 존재 (Next.js 미구축) |
| 백엔드 | 미구축 (Spring Boot 프로젝트 없음) |
| 데이터베이스 | 미구축 (스키마 없음) |
| 인프라 | 미구축 (CI/CD, AWS 없음) |

### 1.2 목표
- **기간**: 4주 (평일 1-2시간, 주말 3-5시간)
- **대상 사용자**: 20-30대, 10-100명 소규모
- **핵심 어필**: GIS 공간 쿼리, DDD, 이벤트 기반 시스템

### 1.3 기술 스택
| 영역 | 기술 |
|------|------|
| Frontend | Next.js 14, React Query, Tailwind CSS, Kakao Maps SDK |
| Backend | Java 17, Spring Boot 3.x, JPA + QueryDSL, Spring Batch |
| Database | MySQL 8 (Spatial Extension) |
| Cache | Redis |
| Infra | AWS (EC2, RDS), Vercel, GitHub Actions |

---

## 2. 기능 우선순위

### 2.1 필수 (MVP Core)
| 기능 | 사용자 가치 | 기술 하이라이트 |
|------|------------|----------------|
| 이메일 인증 | 개인화된 경험 | JWT, Spring Security |
| Archive CRUD | 감정 기록의 핵심 | JPA, 도메인 모델 |
| GIS 공간 쿼리 | 위치 기반 발견 | MySQL Spatial, ST_Distance_Sphere |
| 지도 뷰 | 시각적 탐색 | Kakao Maps SDK |
| 피드 뷰 | 콘텐츠 소비 | 무한 스크롤, 필터링 |

### 2.2 선택 (Complete Experience)
| 기능 | 사용자 가치 | 기술 하이라이트 |
|------|------------|----------------|
| 좋아요/스크랩 | 참여 및 저장 | 낙관적 UI 업데이트 |
| 타임캡슐 | 미래의 나에게 | 이벤트 기반 시스템 |
| 필터/정렬 | 원하는 콘텐츠 찾기 | QueryDSL 동적 쿼리 |

### 2.3 후순위 (Polish)
| 기능 | 사용자 가치 | 기술 하이라이트 |
|------|------------|----------------|
| 월간 리포트 | 감정 회고 | 집계 쿼리, 차트 시각화 |
| Spring Batch | - | 배치 처리 데모 |
| Redis 캐싱 | 빠른 응답 | 캐시 전략 |

### 2.4 연기 (Post-MVP)
- 이미지 업로드 (S3 연동 복잡성)
- 위치 정밀도 상세 설정
- 이메일 알림 발송

---

## 3. Phase별 상세 계획

### Phase 0: Project Setup (Day 1-2)

#### 목표
개발 환경 구축 및 스켈레톤 앱 실행

#### 백엔드 구조
```
backend/
├── build.gradle
├── src/main/java/com/feelarchive/
│   ├── FeelArchiveApplication.java
│   ├── global/
│   │   ├── config/          # Security, JPA, QueryDSL 설정
│   │   ├── exception/       # 전역 예외 처리
│   │   └── common/          # 공통 DTO, 유틸
│   └── domain/
│       ├── member/          # 회원 도메인
│       ├── archive/         # 아카이브 도메인
│       ├── timecapsule/     # 타임캡슐 도메인
│       ├── report/          # 리포트 도메인
│       └── notification/    # 알림 도메인
└── src/main/resources/
    └── application.yml
```

#### 프론트엔드 구조
```
frontend/
├── package.json
├── src/
│   ├── app/                 # Next.js App Router
│   │   ├── (auth)/          # 인증 관련 페이지
│   │   ├── (main)/          # 메인 서비스 페이지
│   │   └── layout.tsx
│   ├── components/          # 재사용 컴포넌트
│   ├── hooks/               # 커스텀 훅
│   ├── lib/                 # API 클라이언트, 유틸
│   └── types/               # TypeScript 타입
└── tailwind.config.js
```

#### 완료 조건 (DoD)
- [ ] `./gradlew bootRun` 성공
- [ ] `npm run dev` 성공
- [ ] `docker-compose up` 으로 MySQL, Redis 실행
- [ ] Kakao Maps 테스트 페이지에서 지도 렌더링

---

### Phase 1: Authentication & Member Domain (Week 1, Days 1-3)

#### 목표
회원가입/로그인 기능 구현

#### 데이터 모델
```
Member
├── id (PK, BIGINT)
├── email (UNIQUE, VARCHAR)
├── password (VARCHAR, BCrypt)
├── nickname (VARCHAR)
├── profileImageUrl (VARCHAR, nullable)
├── createdAt (DATETIME)
└── updatedAt (DATETIME)
```

#### API 명세
| Method | Endpoint | Request | Response |
|--------|----------|---------|----------|
| POST | /api/auth/signup | { email, password, nickname } | 201 Created |
| POST | /api/auth/login | { email, password } | { accessToken } |
| POST | /api/auth/logout | - | 200 OK |
| GET | /api/members/me | - | { id, email, nickname, ... } |
| PATCH | /api/members/me | { nickname?, profileImageUrl? } | 200 OK |

#### 사용자 플로우
```
[회원가입]
이메일 입력 → 비밀번호 입력 → 닉네임 입력 → 가입 완료 → 로그인 페이지

[로그인]
이메일 입력 → 비밀번호 입력 → JWT 발급 → 홈으로 이동
```

#### 완료 조건 (DoD)
- [ ] 이메일/비밀번호로 회원가입 가능
- [ ] 로그인 시 JWT 토큰 발급
- [ ] 토큰으로 인증된 API 호출 가능
- [ ] 로그아웃 시 클라이언트 토큰 삭제
- [ ] 프론트엔드 인증 상태 유지 (새로고침 후에도)

---

### Phase 2: Archive Core + GIS (Week 1, Days 4-7)

#### 목표
위치 기반 아카이브 생성 및 지도 표시 (핵심 기능)

#### 데이터 모델
```
Archive
├── id (PK, BIGINT)
├── memberId (FK → Member)
├── content (TEXT)
├── latitude (DOUBLE)
├── longitude (DOUBLE)
├── point (POINT, SRID 4326, SPATIAL INDEX)
├── locationName (VARCHAR)
├── locationPrecision (ENUM: EXACT, AREA, CITY)
├── isPublic (BOOLEAN)
├── imageUrl (VARCHAR, nullable)
├── likeCount (INT, default 0)
├── createdAt (DATETIME)
├── updatedAt (DATETIME)
└── deletedAt (DATETIME, nullable, Soft Delete)

ArchiveEmotion
├── id (PK, BIGINT)
├── archiveId (FK → Archive)
└── emotionType (ENUM: HAPPY, SAD, ANXIOUS, ANGRY, CALM, EXCITED, LONELY, GRATEFUL, PEACEFUL, THRILLED, NOSTALGIC)
```

#### GIS 쿼리 (핵심)
```sql
-- 반경 N km 내 아카이브 조회
SELECT a.*,
       ST_Distance_Sphere(a.point, ST_SRID(POINT(:lng, :lat), 4326)) as distance
FROM archive a
WHERE ST_Distance_Sphere(a.point, ST_SRID(POINT(:lng, :lat), 4326)) <= :radiusMeters
  AND a.deleted_at IS NULL
  AND a.is_public = true
ORDER BY distance;

-- 바운딩 박스 내 마커 조회 (지도 뷰)
SELECT a.id, a.latitude, a.longitude, ae.emotion_type
FROM archive a
JOIN archive_emotion ae ON a.id = ae.archive_id
WHERE MBRContains(
    ST_GeomFromText('POLYGON((:swLng :swLat, :neLng :swLat, :neLng :neLat, :swLng :neLat, :swLng :swLat))'),
    a.point
)
AND a.deleted_at IS NULL
AND a.is_public = true;
```

#### API 명세
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/archives | 아카이브 작성 |
| GET | /api/archives | 목록 조회 (페이징, 필터, 정렬) |
| GET | /api/archives/{id} | 상세 조회 |
| PATCH | /api/archives/{id} | 수정 |
| DELETE | /api/archives/{id} | 삭제 (Soft Delete) |
| GET | /api/archives/map | 지도 마커 조회 (bounds 파라미터) |
| GET | /api/archives/nearby | 반경 내 조회 (lat, lng, radius 파라미터) |

#### 화면 구성
1. **아카이브 작성 페이지**
   - 위치 선택 (현재 위치 GPS / 지도에서 선택)
   - 감정 태그 복수 선택
   - 텍스트 입력 (2000자 이상)
   - 공개/비공개 선택

2. **탐색 - 지도 뷰**
   - Kakao Maps 전체 화면
   - 마커 클릭 시 미리보기 카드
   - 감정별 마커 색상 구분

3. **탐색 - 피드 뷰**
   - 카드 형태 리스트
   - 무한 스크롤
   - 필터 (감정, 거리, 시간)

#### 완료 조건 (DoD)
- [ ] 아카이브 생성 시 위도/경도 저장 (POINT 컬럼)
- [ ] Spatial Index 생성 확인
- [ ] `/api/archives/nearby?lat=37.5&lng=127&radius=5000` 정상 동작
- [ ] Kakao Maps에 마커 표시
- [ ] 마커 클릭 시 미리보기 표시
- [ ] 피드 뷰 무한 스크롤 동작

---

### Phase 3: Interactions + TimeCapsule (Week 2)

#### 목표
좋아요/스크랩 기능 및 타임캡슐 (이벤트 기반 시스템)

#### 데이터 모델
```
Like
├── id (PK)
├── memberId (FK)
├── archiveId (FK)
├── createdAt
└── UNIQUE(memberId, archiveId)

Scrap
├── id (PK)
├── memberId (FK)
├── archiveId (FK)
├── createdAt
└── UNIQUE(memberId, archiveId)

TimeCapsule
├── id (PK)
├── memberId (FK)
├── content (TEXT)
├── latitude (DOUBLE)
├── longitude (DOUBLE)
├── locationName (VARCHAR)
├── imageUrl (VARCHAR, nullable)
├── scheduledAt (DATETIME) -- 공개 예정 시간
├── isOpened (BOOLEAN, default false)
├── isLocked (BOOLEAN, default false) -- 30분 후 true
├── createdAt (DATETIME)
└── updatedAt (DATETIME)

TimeCapsuleEmotion
├── id (PK)
├── timeCapsuleId (FK)
└── emotionType (ENUM)

Notification
├── id (PK)
├── memberId (FK)
├── type (ENUM: TIME_CAPSULE_OPENED)
├── targetId (BIGINT)
├── isRead (BOOLEAN, default false)
└── createdAt (DATETIME)
```

#### 이벤트 기반 시스템
```java
// 1. 도메인 이벤트 정의
public record TimeCapsuleOpenedEvent(
    Long timeCapsuleId,
    Long memberId,
    LocalDateTime openedAt
) {}

// 2. 서비스에서 이벤트 발행
@Transactional
public void checkAndOpenTimeCapsules() {
    List<TimeCapsule> capsules = repository.findReadyToOpen();
    for (TimeCapsule capsule : capsules) {
        capsule.open();
        eventPublisher.publishEvent(
            new TimeCapsuleOpenedEvent(capsule.getId(), capsule.getMemberId(), now())
        );
    }
}

// 3. 리스너에서 알림 생성
@EventListener
@Async
public void handleTimeCapsuleOpened(TimeCapsuleOpenedEvent event) {
    notificationService.create(
        event.memberId(),
        NotificationType.TIME_CAPSULE_OPENED,
        event.timeCapsuleId()
    );
}
```

#### API 명세
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/archives/{id}/like | 좋아요 토글 |
| POST | /api/archives/{id}/scrap | 스크랩 토글 |
| GET | /api/members/me/scraps | 내 스크랩 목록 |
| POST | /api/timecapsules | 타임캡슐 작성 |
| GET | /api/timecapsules | 내 타임캡슐 목록 |
| GET | /api/timecapsules/{id} | 타임캡슐 상세 |
| PATCH | /api/timecapsules/{id} | 타임캡슐 수정 (30분 내) |
| GET | /api/notifications | 알림 목록 |
| PATCH | /api/notifications/{id}/read | 알림 읽음 처리 |

#### 비즈니스 규칙
1. **타임캡슐 30분 규칙**
   - 작성 후 30분 이내만 수정 가능
   - 30분 후 `isLocked = true` 설정
   - 잠금 후 수정/삭제 불가

2. **타임캡슐 공개**
   - `scheduledAt` 시간 도달 시 자동 공개
   - 공개 시 알림 생성
   - 스케줄러로 주기적 확인 (@Scheduled)

#### 완료 조건 (DoD)
- [ ] 좋아요 토글 시 `likeCount` 업데이트
- [ ] 스크랩 목록 조회 가능
- [ ] 타임캡슐 작성 후 30분 경과 시 수정 불가
- [ ] 타임캡슐 예정 시간 도달 시 알림 생성
- [ ] 알림 목록 조회 및 읽음 처리

---

### Phase 4: Report + Polish (Week 3)

#### 목표
월간 리포트 및 UI/UX 개선

#### 리포트 데이터
```java
public record MonthlyReportDto(
    YearMonth month,
    int totalArchives,
    int totalTimeCapsules,
    Map<EmotionType, Long> emotionDistribution,
    List<DailyEmotionDto> dailyEmotions
) {}

public record DailyEmotionDto(
    LocalDate date,
    EmotionType primaryEmotion,
    int archiveCount
) {}
```

#### API 명세
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/reports/monthly | 월간 리포트 (year, month 파라미터) |
| GET | /api/reports/emotions | 감정 분포 통계 |

#### UI 개선 항목
- [ ] 로딩 상태 (Skeleton UI)
- [ ] 에러 상태 (Toast 알림)
- [ ] 빈 상태 ("기록이 없습니다" 메시지)
- [ ] 모바일 반응형
- [ ] 접근성 (키보드 네비게이션, ARIA)

#### 완료 조건 (DoD)
- [ ] 월간 감정 분포 바 차트 표시
- [ ] 일별 감정 흐름 라인 차트 표시
- [ ] 모든 페이지 로딩/에러/빈 상태 처리
- [ ] 모바일 화면에서 사용 가능

---

### Phase 5: Infrastructure + Deployment (Week 4)

#### 목표
CI/CD 파이프라인 구축 및 프로덕션 배포

#### 인프라 아키텍처
```
┌─────────────────────────────────────────────────────────┐
│                        Production                        │
│                                                          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐          │
│  │  Vercel  │───▶│   EC2    │───▶│   RDS    │          │
│  │ (Next.js)│    │(Spring)  │    │ (MySQL)  │          │
│  └──────────┘    └──────────┘    └──────────┘          │
│       │               │                                  │
│       └───────────────┼──────────────────┐              │
│                       │                  │              │
│                  ┌────▼────┐      ┌──────▼─────┐       │
│                  │ ElastiC │      │ CloudWatch │       │
│                  │  ache   │      │   Logs     │       │
│                  └─────────┘      └────────────┘       │
└─────────────────────────────────────────────────────────┘
```

#### CI/CD 파이프라인
```yaml
# Backend: .github/workflows/backend.yml
on:
  push:
    branches: [main]
    paths: ['backend/**']

jobs:
  test:
    - Checkout
    - Setup JDK 17
    - Run tests (./gradlew test)

  build:
    needs: test
    - Build Docker image
    - Push to ECR

  deploy:
    needs: build
    - Deploy to EC2 (docker pull & run)
```

#### Spring Batch 설정
```java
// Soft Delete 정리 Job (매일 새벽 3시)
@Scheduled(cron = "0 0 3 * * *")
public void cleanupDeletedArchives() {
    // 30일 이상 지난 soft deleted 아카이브 물리적 삭제
    archiveRepository.deletePhysicallyOlderThan(LocalDateTime.now().minusDays(30));
}
```

#### 완료 조건 (DoD)
- [ ] PR 생성 시 자동 테스트 실행
- [ ] main 브랜치 push 시 자동 배포
- [ ] 백엔드: EC2에서 실행 중
- [ ] 프론트엔드: Vercel에서 실행 중
- [ ] 데이터베이스: RDS MySQL 연결
- [ ] HTTPS 적용
- [ ] Swagger UI 접근 가능
- [ ] Spring Batch 정리 작업 스케줄링

---

## 4. 리스크 및 대응

| 리스크 | 영향도 | 대응 전략 |
|--------|--------|----------|
| 4주 내 미완성 | 높음 | Phase 2 (GIS) = 최소 MVP, 이후는 선택적 |
| 오버엔지니어링 | 중간 | 단순 구현 먼저, 필요 시 리팩토링 |
| GIS 쿼리 성능 | 중간 | Spatial Index 필수, 테스트 데이터로 검증 |
| Kakao Maps 제한 | 낮음 | Week 1 프로토타입으로 조기 검증 |
| AWS 비용 | 낮음 | Free Tier 활용, 불필요한 리소스 정리 |

---

## 5. 복구 정보 (Recovery Info)

### 현재 기준점 (Baseline)
- **마지막 정상 상태**: 정적 랜딩 페이지 (index.html)
- **Git commit**: 254e6a2 (Add files via upload)
- **확인 방법**: index.html 브라우저에서 열기

### 되돌림 전략
1. **Phase별 독립 배포**: 각 Phase 완료 시 배포 가능 상태 유지
2. **DB 마이그레이션 버전 관리**: Flyway 사용
3. **Feature Flag**: 미완성 기능 비활성화 가능
4. **Git 브랜치 전략**: feature 브랜치에서 개발, main은 배포 가능 상태 유지

### 롤백 포인트
| Checkpoint | 설명 | 롤백 방법 |
|------------|------|----------|
| Phase 1 완료 | 인증 기능 동작 | 해당 커밋으로 revert |
| Phase 2 완료 | GIS 핵심 기능 동작 | 해당 커밋으로 revert |
| Phase 3 완료 | 상호작용 기능 동작 | 해당 커밋으로 revert |
| Phase 4 완료 | 리포트 기능 동작 | 해당 커밋으로 revert |

---

## 6. 검증 계획

### 단위 테스트
- Member 도메인: 회원가입 검증, 비밀번호 암호화
- Archive 도메인: CRUD, Soft Delete, 감정 태그
- TimeCapsule 도메인: 30분 잠금 규칙, 공개 조건

### 통합 테스트
- 인증 플로우: 가입 → 로그인 → 인증된 API 호출
- GIS 쿼리: 실제 좌표로 반경 검색

### E2E 검증
| 시나리오 | 검증 방법 |
|----------|----------|
| 회원가입/로그인 | 브라우저에서 전체 플로우 수행 |
| 아카이브 작성 | 위치 선택 → 감정 태그 → 저장 → 지도에 표시 확인 |
| GIS 검색 | Postman으로 `/api/archives/nearby` 호출 |
| 타임캡슐 | 작성 → 30분 대기 → 수정 불가 확인 |
| 배포 | CI/CD 파이프라인 전체 실행 |

---

## 부록: 참고 문서

- **SPEC.md**: 전체 요구사항 정의서
- **TASK_LIST.json**: 상세 작업 목록
- **index.html**: 디자인 참조 (색상: #D19D5E, 폰트, 레이아웃)
