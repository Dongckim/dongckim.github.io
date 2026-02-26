---
title: "System Design - Scalable URL Shortener"
layout: single
Typora-root-url: ../
categories: SystemDesign
tag: [python, fastapi, mysql, redis, docker, system-design]
use_math: true
---

## Scalable URL Shortener System Design

![URL Shortening Concept]({{site.url}}/images/2026-02-26-systemdesign-urlshortener/url-concept.png){: .align-center}

Tech Stack: `FastAPI` `MySQL` `Redis` `Docker`

단순한 기능 구현을 넘어, 1억 명(100M DAU)의 사용자를 수용할 수 있는 아키텍처를 목표로 설계한 URL Shortener 시스템이다. bit.ly, TinyURL 등과 같은 서비스를 직접 설계하고 구현하며 얻은 인사이트를 정리한다.

---

## 1. Functional Requirements (기능적 요구사항)

문제 설명에서 **동사(Verb)**를 추출하여 핵심 기능을 정의한다.

| 동작 | 기능 | API |
|---|---|---|
| **생성 (Create)** | 긴 URL을 받아 고유한 단축 URL을 반환 | `POST /api/urls/shorten` |
| **리다이렉트 (Read)** | 단축 URL 접속 시 원본 URL로 즉시 이동 | `GET /api/urls/{shortUrl}` |
| **분석 (Track)** | 각 단축 URL의 클릭 수를 비동기 추적 | 내부 이벤트 처리 (비공개) |

---

## 2. Non-Functional Requirements (비기능적 요구사항)

문제 설명에서 **형용사(Adjective)**를 추출하여 품질 제약 조건을 정의한다.

| 제약 조건 | 설명 |
|---|---|
| **고가용성 (High Availability)** | 대규모 트래픽에도 24/7 중단 없는 서비스 |
| **저지연 (Low Latency)** | 10ms 이내의 빠른 리다이렉트 응답 |
| **유일성 (Uniqueness)** | 생성된 단축 URL 간의 충돌 원천 차단 |
| **내구성 (High Durability)** | 서버 장애에도 데이터 영속성 보장 |

### Scale Estimation

- **읽기/쓰기 비율**: 100:1 (Read-Heavy 시스템)
- **일일 쓰기 요청**: 약 100만 건
- **데이터 보관 기간**: 5년
- **항목당 크기**: 약 500 bytes

---

## 3. Data Model (데이터 모델)

문제 설명에서 **명사(Noun)**를 추출하여 엔티티를 정의한다. 성능 최적화를 위해 URL 매핑과 분석(Analytics) 데이터를 물리적으로 분리하는 **Decoupled 아키텍처**를 채택했다.

| 테이블명 | 역할 | 주요 속성 |
|---|---|---|
| **URLMapping** | 원본-단축 URL 매핑 정보 | `id`(PK), `short_url`(Unique Index), `original_url`, `created_at` |
| **Analytics** | 클릭 수 및 통계 관리 | `id`(PK), `short_url_id`(FK), `click_count` |

두 테이블은 **1:1 관계**이며, 독립적인 서비스로 관리하여 분석 트래픽이 리다이렉트 성능에 영향을 주지 않도록 격리했다.

![URL Shortener Data Model]({{site.url}}/images/2026-02-26-systemdesign-urlshortener/data-model.png){: .align-center}

---

## 4. 핵심 알고리즘: ID 생성과 Base62 인코딩

### 왜 해싱이 아닌가?

ID 생성 전략을 선택할 때, 여러 후보를 검토했다.

| 방식 | 장점 | 단점 |
|---|---|---|
| **Hash Function** | 구현 간단 | 충돌 가능성, 충돌 해소 로직 필요 |
| **UUID** | 전역 유일성 보장 | 128bit로 길이가 너무 길어 "단축" 목적에 부적합 |
| **Snowflake ID** | 시간 순서 보장, 분산 환경 지원 | 구현 복잡도 높음 |
| **Machine ID + Sequence** | 간단하고 충돌 없음, 샤딩과 자연스럽게 연동 | 머신 수 제한 필요 |

최종적으로 **Machine ID + Sequence Number** 방식을 채택하고, 생성된 정수를 Base62로 인코딩한다.

### Base62 인코딩

- **메커니즘**: Integer ID → Base62 Encoding → Short String (예: `123456` → `FXsk`)
- **문자 집합**: `[a-z, A-Z, 0-9]` = 62개 문자
- **수용량**: $62^7 \approx 3.5 \times 10^{12}$ (약 3.5조 개)로 5년 이상의 데이터를 충분히 수용

Base64 대신 Base62를 선택한 이유는, URL에서 `+`와 `/`가 특수 문자로 해석될 수 있어 안전한 문자만 사용하기 위함이다.

---

## 5. High-Level Architecture (고수준 아키텍처)

Redis 캐싱과 Docker 오케스트레이션이 포함된 전체 흐름이다.

### 쓰기 경로 (Write Path)

![URL Shortener Write Path]({{site.url}}/images/2026-02-26-systemdesign-urlshortener/write-path.png){: .align-center}

1. 클라이언트가 `POST /api/urls/shorten`으로 긴 URL을 전송
2. URL Shortening Service가 ID Generator에게 고유 ID 요청
3. ID를 Base62로 인코딩하여 단축 URL 생성
4. URLMapping 테이블에 매핑 정보 저장 후 단축 URL 반환

### 읽기 경로 (Read Path)

![URL Shortener High-Level Architecture]({{site.url}}/images/2026-02-26-systemdesign-urlshortener/high-level-architecture.png){: .align-center}

1. 단축 URL로 `GET` 요청 발생
2. **Cache First**: Redis에서 매핑 조회 (Hit 시 즉시 응답)
3. **Cache Miss**: DB에서 원본 URL 조회 → Redis에 캐시 업데이트
4. `302 Found` 응답으로 원본 URL로 리다이렉트

### 분석 경로 (Analytics Path)

```
Redirect 발생 → Message Queue (Kafka) → Analytics Service → In-Memory Counter (Redis) → DB Flush
```

리다이렉트 시점에 메인 로직을 블로킹하지 않고, Message Queue를 통해 **비동기적으로** 클릭 수를 업데이트한다. Redis의 인메모리 카운터를 활용해 실시간 집계 후, 주기적으로 DB에 Flush하는 구조다.

---

## 6. Scaling Strategy (확장 전략)

### Machine ID를 Shard Key로 활용

단축 URL의 첫 번째 문자를 Machine ID(Prefix)로 사용하면, ID Generator와 DB 샤드가 **1:1로 매핑**된다.

- **쓰기**: 각 머신이 독립적으로 ID를 생성하고 자신의 샤드에만 저장 → 완전한 병렬 처리
- **읽기**: 단축 URL의 prefix로 어떤 샤드를 조회할지 즉시 결정 (예: `a82c7w` → 샤드 `a`)
- **확장**: 새 머신 추가 시 고유 prefix를 부여하면 기존 시스템에 영향 없이 수평 확장 가능

이 구조의 핵심 이점:

| 이점 | 설명 |
|---|---|
| **Scalability** | 머신 추가만으로 수평 확장 |
| **Concurrency** | 독립적 쓰기 경로로 동시 처리량 극대화 |
| **Isolation** | 한 샤드의 장애가 전체 시스템에 영향을 주지 않음 |
| **Simplicity** | 백업, 인덱싱 등 유지보수를 샤드 단위로 수행 |

---

## 7. Deep Dive: 실전 구현의 교훈 (Engineering Insights)

### 301 vs 302 Redirect 전략

| 응답 코드 | 동작 | 특성 |
|---|---|---|
| **301 Moved Permanently** | 브라우저가 결과를 캐싱 | 서버 부하 감소, 통계 수집 불가 |
| **302 Found** | 매 요청마다 서버 경유 | 통계 수집 가능, 서버 부하 증가 |

서버가 매번 요청을 받아 통계를 수집할 수 있도록 **302 Found**를 선택했다. Analytics 기능이 핵심 요구사항이므로, 매 클릭을 서버가 인지해야 한다.

### 캐싱 최적화

**80/20 법칙(파레토 법칙)**에 따라 전체 트래픽의 80%가 상위 20%의 인기 URL에 집중된다. Read-through 캐싱 패턴을 적용하여:

- Redis가 요청을 먼저 받고, 캐시 미스 시 자동으로 DB 조회 후 캐시 저장
- Request Handler가 캐시 미스 로직을 직접 관리할 필요 없음
- 메모리 효율을 극대화하면서 $O(1)$ 조회 성능 확보

### 인프라 자동화

Docker Compose를 통해 MySQL과 Redis를 컨테이너로 격리 운영하며, 환경 일관성과 배포 재현성을 확보했다.

---

## 8. 요약 및 회고

### Level별 기대치

| 차원 | Mid-Level | Senior | Staff |
|---|---|---|---|
| **ID 생성** | 유일성과 짧음의 요구사항 설명 | Hash/UUID/Snowflake 비교 분석 | 멀티 리전 ID 조율, 클럭 스큐 처리 |
| **인코딩** | Base62 설명 및 ID 공간 계산 | Base16/62/64 트레이드오프 논의 | 분석/디버깅에 미치는 인코딩 영향 |
| **스케일링** | 샤딩 개념 이해 | 샤드 키 전략 설계, 읽기 라우팅 | 핫 샤드 처리, Consistent Hashing |
| **캐싱** | 캐시 레이어 포함 설계 | 캐시 적중률 계산, 무효화 전략 | 멀티 티어 캐싱, Cache Stampede 방지 |

### 핵심 교훈

System Design은 단순히 "동작하는 코드"를 넘어 **"왜 이 구조를 선택했는가"**를 설명할 수 있어야 한다. URL Shortener는 간단해 보이지만, ID 생성 전략, 캐싱 계층, 샤딩 전략, 리다이렉트 방식 등 모든 결정에 **트레이드오프**가 존재한다. 이 트레이드오프를 인지하고 근거를 가지고 선택하는 것이 시스템 디자인의 본질이다.
