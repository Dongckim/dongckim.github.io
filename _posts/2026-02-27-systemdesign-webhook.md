---
title: "System Design - Scalable & Reliable Webhook System"
layout: single
Typora-root-url: ../
categories: SystemDesign
tag: [python, fastapi, mysql, redis, celery, docker, system-design]
use_math: true
---

## Scalable & Reliable Webhook System Design

Tech Stack: `FastAPI` `MySQL` `Redis` `Celery` `Docker`

단순한 POST API 수신을 넘어, **실시간성(Real-time)**과 **신뢰성(Reliability)**을 동시에 보장하는 웹후크 시스템을 설계하고 구현했다. 초당 수만 건의 이벤트 스파이크를 견디고, 네트워크 오류 시에도 데이터 유실 없이 재시도하는 아키텍처를 목표로 한다.

---

## Background

Webhooks는 특정 이벤트에 의해 트리거되는 실시간 알림을 외부 시스템으로 전송하는 방식이다. 전통적인 API가 **Polling**에 의존하는 것과 달리, 웹후크는 이벤트 발생 즉시 데이터를 **Push**하여 효율성과 실시간성을 모두 확보한다.

![Traditional API vs Webhook]({{site.url}}/images/2026-02-27-systemdesign-webhook/traditional-vs-webhook.png){: .align-center}

| 방식 | 동작 원리 | 특성 |
|---|---|---|
| **Polling** | "음식 나왔나요?"를 계속 반복 | 비효율적, 불필요한 네트워크 자원 낭비 |
| **Webhook** | "음식 나왔습니다!" 벨을 울림 | 이벤트 기반, 실시간, 리소스 최대 90% 절약 |

실제 서비스 사례:
- **Stripe**: 결제 완료, 환불 등 금융 이벤트 전달
- **Shopify**: 주문 생성, 배송 업데이트 알림
- **GitHub**: PR 머지, 이슈 생성 등 개발 이벤트 트리거

---

## 1. Functional Requirements (기능적 요구사항)

문제 설명에서 **동사(Verb)**를 추출하여 핵심 기능을 정의한다.

| 동작 | 기능 | API |
|---|---|---|
| **수신 (Ingestion)** | 외부 서비스의 이벤트 알림(JSON)을 수신 | `POST /webhook` |
| **검증 (Verify)** | HMAC 서명을 통해 신뢰할 수 있는 요청인지 확인 | 미들웨어 로직 |
| **처리 (Process)** | 비즈니스 로직(결제 승인, 권한 부여 등) 수행 | 비동기 워커 |
| **조회 (Monitor)** | 처리된 이벤트의 상태 및 로그 확인 | `GET /api/events` |

### Out of Scope

- 웹후크 구독 등록/관리 (Provider 측 구현)
- 이벤트 타입별 세부 비즈니스 로직
- UI 대시보드

---

## 2. Non-Functional Requirements (비기능적 요구사항)

문제 설명에서 **형용사(Adjective)**를 추출하여 품질 제약 조건을 정의한다.

| 제약 조건 | 설명 |
|---|---|
| **낮은 지연 시간 (Low Latency)** | 요청 수신 후 200ms 이내에 200 OK 응답 반환 |
| **멱등성 (Idempotency)** | 동일한 이벤트가 여러 번 전송되어도 중복 처리 방지 |
| **최소 한 번 전달 (At-least-once)** | 시스템 장애 시에도 이벤트 유실 없이 최소 1회 처리 보장 |
| **고가용성 (High Availability)** | 시스템 컴포넌트 장애 시에도 서비스 지속 |
| **부하 분산 (Load Buffering)** | 트래픽 폭주 시 메시지 큐를 통한 압력 조절 (Back-pressure) |

### Scale Estimation

- **일일 이벤트 처리량**: 100만 건 (Peak 시 5배 증가)
- **성능 목표**: End-to-End 지연 시간 200ms 미만
- **데이터 보관**: 감사(Audit)를 위해 원본 페이로드 30일 보관
- **항목당 크기**: 약 5KB → 30일 총 저장량 약 150GB

---

## 3. Data Model (데이터 모델)

문제 설명에서 **명사(Noun)**를 추출하여 엔티티를 정의한다. 중복 처리를 막기 위해 `event_id`를 기반으로 한 상태 관리가 필수적이다.

| 테이블명 | 역할 | 주요 속성 |
|---|---|---|
| **ProcessedEvents** | 멱등성 체크 및 결과 저장 | `event_id`(PK), `payload`, `status`(SUCCESS/FAIL), `created_at` |
| **RetryLogs** | 실패한 이벤트의 재시도 이력 | `id`(PK), `event_id`(FK), `retry_count`, `last_error`, `next_retry_at` |

`event_id`에 **Unique Index**를 설정하여 DB 레벨에서 중복 삽입을 원천 차단한다.

---

## 4. 핵심 아키텍처: 수신과 처리의 분리 (Decoupling)

웹후크 설계의 핵심은 **"일단 빨리 받고, 일은 나중에 하는 것"**이다.

### Basic Design의 한계

초기 설계는 Request Handler가 HTTP 수신과 비즈니스 로직을 동시에 처리하는 구조다.

![Webhook Basic Design]({{site.url}}/images/2026-02-27-systemdesign-webhook/basic-design.png){: .align-center}

**문제점**: Request Handler가 이벤트를 처리한 후 DB 저장 전에 실패하면, 이벤트가 유실된다. 또한 트래픽 스파이크 시 DB 커넥션 풀이 고갈되어 서버가 다운될 수 있다.

### Message Queue를 활용한 개선 설계

![Webhook High-Level Design]({{site.url}}/images/2026-02-27-systemdesign-webhook/high-level-design.png){: .align-center}

**Ingestion Path (수신 경로)**:
1. 클라이언트(Provider)가 `POST /webhook` 요청
2. FastAPI: HMAC 서명 검증 후 즉시 **Redis(Message Queue)**에 작업 전달
3. 클라이언트에게 즉시 **200 OK** 반환 (지연 시간 최소화)

**Processing Path (처리 경로)**:
1. Celery Worker: 큐에서 이벤트를 하나씩 꺼냄
2. MySQL Check: `event_id`를 조회하여 이미 처리된 이벤트인지 확인 (멱등성 보장)
3. Business Logic: 실제 작업 수행 및 결과 DB 기록
4. Retry Logic: 실패 시 Exponential Backoff 전략으로 재시도 예약

![Webhook Sequence Diagram]({{site.url}}/images/2026-02-27-systemdesign-webhook/sequence-diagram.png){: .align-center}

이 설계는 **실패 복구(Failure Recovery)**, **부하 완충(Load Buffering)**, **수평 확장(Scalability)** 세 가지 이점을 동시에 제공한다.

---

## 5. Scaling & Reliability Strategy (확장 및 신뢰성 전략)

### 컴포넌트별 장애 처리

**1. Request Handler 장애**

![Request Handler Failure]({{site.url}}/images/2026-02-27-systemdesign-webhook/request-handler-failure.png){: .align-center}

| 시나리오 | 대응 전략 |
|---|---|
| 큐 적재 전 실패 | 클라이언트가 200 OK를 받지 못하므로 재전송 → 자동 복구 |
| 타임아웃 | Circuit Breaker로 연쇄 장애 차단, 클라이언트에 즉시 실패 신호 |

**2. Message Queue 장애**

![Message Queue Failure]({{site.url}}/images/2026-02-27-systemdesign-webhook/message-queue-failure.png){: .align-center}

| 전략 | 설명 |
|---|---|
| **Durable Queue** | 메시지를 디스크에 영속화하여 서버 재시작 후에도 유지 |
| **Multi-Node Replication** | Kafka/RabbitMQ 클러스터로 메시지 복제, 단일 노드 장애 허용 |

**3. Queue Consumer 장애**

![Consumer Failure]({{site.url}}/images/2026-02-27-systemdesign-webhook/consumer-failure.png){: .align-center}

| 전략 | 설명 |
|---|---|
| **Multiple Instances** | 컨슈머를 다중 배포하여 하나 실패 시 다른 인스턴스가 이어받음 |
| **Message Acknowledgment** | DB 저장 완료 후에만 메시지 Dequeue → 미완료 이벤트는 큐에 잔류 |
| **Auto-Restart (Kubernetes)** | 실패한 컨슈머 자동 재시작, 큐 길이에 따른 수평 확장 |

**4. Database 장애**

| 전략 | 설명 |
|---|---|
| **Exponential Backoff** | 쓰기 실패 시 $2^n$ 초 간격으로 재시도 (DB 과부하 방지) |
| **Replication & Failover** | Primary 장애 시 Secondary로 자동 전환 |

### 지수 백오프 (Exponential Backoff)

외부 서버 일시 장애 시 즉시 재시도는 오히려 부하를 가중시킨다. 재시도 간격을 $2^n$ 초로 증가시켜 시스템 회복 시간을 확보한다.

$$\text{Retry Interval} = 2^n \text{ seconds}, \quad n = 1, 2, 3, ...$$

---

## 6. Deep Dive: Engineering Insights

### 보안: HMAC 서명 검증

웹후크는 외부에서 오는 요청이므로, **출처 인증**이 필수적이다.

**HMAC 서명 흐름**:
1. Provider(예: Stripe)와 서비스가 **Secret Key**를 사전 공유
2. Provider는 payload의 HMAC 해시를 생성하여 요청 헤더에 포함
3. 서비스는 동일한 Secret으로 해시를 재계산하여 일치 여부 검증

```python
import hmac
import hashlib

def verify_signature(payload: bytes, signature: str, secret: str) -> bool:
    expected = hmac.new(
        secret.encode(), payload, hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

추가 보안 레이어:

| 기법 | 설명 | 효과 |
|---|---|---|
| **IP Whitelisting** | 알려진 Provider IP만 허용 | 무단 IP 차단 |
| **Rate Limiting** | 분당 최대 요청 수 제한 | DoS 공격 방어 |
| **HTTPS Only** | 전송 계층 암호화 | 패킷 탈취 방지 |

### 중복 요청 처리 (Idempotency)

네트워크 재시도로 인해 동일 이벤트가 여러 번 도달할 수 있다.

**Idempotency Key 패턴**:

```sql
-- event_id Unique Index로 중복 삽입 원천 차단
INSERT INTO ProcessedEvents (event_id, payload, status)
VALUES (:event_id, :payload, 'PROCESSING')
ON DUPLICATE KEY IGNORE;
```

처리 전 `event_id` 존재 여부를 확인하고, 이미 처리된 이벤트는 즉시 스킵하여 비즈니스 로직의 중복 실행을 차단한다.

### 순서 보장 없는 이벤트 처리 (Out-of-Order)

네트워크 지연으로 `invoice.paid`가 `invoice.created`보다 먼저 도착할 수 있다.

**핵심 원칙**: 웹후크 처리 로직은 이벤트 순서를 **가정하지 않아야 한다**.

```
invoice.paid 수신 시:
  1. Stripe API에서 최신 invoice 데이터를 직접 조회 (Source of Truth)
  2. 로컬 DB 업데이트 → 상태를 PAID로 변경

invoice.created 수신 시 (뒤늦게 도착):
  1. event의 created_time과 DB의 updated_time 비교
  2. 이미 최신 상태이면 → 스킵 (중복 처리 방지)
```

**설계 원칙 정리**:

| 원칙 | 구현 방법 |
|---|---|
| **Stateless 처리** | 로컬 DB 상태가 아닌 Source of Truth(Provider API) 기준으로 처리 |
| **Idempotent 설계** | 동일 이벤트를 여러 번 처리해도 결과가 동일해야 함 |
| **Timestamp 활용** | `event_id`와 `timestamp`로 Outdated 이벤트 감지 후 스킵 |

---

## 7. 요약 및 회고

### Level별 기대치

| 차원 | Junior | Senior | Staff |
|---|---|---|---|
| **신뢰성** | 단순 API 수신 구현 | 재시도 및 멱등성 설계 | DLQ(Dead Letter Queue) 및 장애 전파 차단 |
| **성능** | 동기식 처리 | 메시지 큐 기반 비동기 설계 | 분산 큐(Kafka) 파티셔닝 전략 |
| **보안** | 인증 없음 | HMAC 서명 및 IP 화이트리스트 검증 | mTLS 및 페이로드 암호화 |
| **순서 처리** | 순서 가정 로직 | Timestamp 기반 스킵 | 이벤트 소싱 및 CQRS 패턴 적용 |

### 핵심 교훈

System Design은 단순히 "동작하는 코드"가 아니라 **"실패할 수 있는 모든 상황을 가정한 설계"**다.

웹후크 시스템의 핵심은 세 가지로 요약된다:

1. **빠르게 받고, 나중에 처리하라** — Decoupling을 통해 응답 지연 최소화
2. **실패를 가정하라** — 모든 컴포넌트는 장애가 날 수 있다. 재시도와 멱등성으로 보완
3. **순서를 믿지 마라** — 네트워크는 순서를 보장하지 않는다. Source of Truth에서 직접 확인
