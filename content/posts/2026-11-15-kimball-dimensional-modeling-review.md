---
title: "[Kimball] 차원 모델링 복습: 설계 절차, 팩트, 차원, 통합"
date: 2026-11-15T22:52:58+09:00
categories: ["데이터 모델링"]
tags: ["Kimball", "차원 모델링", "SCD"]
summary: "Kimball 차원 모델링을 설계 절차, 팩트, 차원, 통합의 네 영역으로 나눠 복습하고 Kimball Group 기법 문서와 대조했다."
draft: false
---

## 들어가기

![Kimball 차원 모델링 학습 범위: 설계 절차, 팩트, 차원, 통합](/images/2026-11-15-kimball/overview.png)

지표를 만들다 보면 "이 테이블을 어떤 기준으로 나눠야 하지?"에서 자주 막힌다. 얼마 전에는 제품 이벤트 로그로 서비스별 지표를 만들려다가 팩트 테이블을 서비스별로 쪼갤지 고민했다. "팩트는 프로세스 기준으로 나누고 서비스는 차원으로 둔다"가 답인데, 이 답이 바로 나오지 않았다. 그래서 Kimball의 차원 모델링을 다시 공부하고 정리했다.

차원 모델링(dimensional modeling)은 데이터를 두 종류의 테이블로 나누는 설계 방법이다. 측정값은 팩트 테이블에 두고, 그 측정이 일어난 맥락은 차원 테이블에 둔다.

이 글은 학습 내용을 설계 절차, 팩트, 차원, 통합의 네 영역으로 나눠 복습한다. 영역마다 용어의 정의, 판단 기준, 예시를 적었다. 설명은 Kimball Group의 기법 문서와 대조했고, 그 문서에 없는 내용은 본문에 따로 표시했다. 실제 테이블에 적용한 결과도 다음에 공유해볼 예정이다.

공부하면서 헷갈렸던 질문 세 가지

- 차원은 왜 카디널리티가 낮은 게 좋다고 하는가?
- "보고서에 그대로 찍어도 읽히는 값"은 무슨 뜻인가?
- 변화 추적 유형은 실무에서 몇 번까지 쓰는가?

예시는 온라인 쇼핑몰의 주문으로 통일했다.

{{< figure src="/images/2026-11-15-kimball/fig1-study-map.png" alt="차원 모델링 학습 지도" caption="그림 1. 차원 모델링 학습 지도" >}}

설계 절차 네 단계 가운데 그레인 선언이 가장 중요하다. 그레인을 정해야 차원과 팩트를 고를 수 있다. 통합은 여러 팩트가 같은 차원을 쓰도록 맞추는 단계다.

## 기반: 설계 절차 4단계

설계할 때 정하는 것은 네 가지다. 비즈니스 프로세스를 고르고, 그레인을 선언하고, 차원을 찾고, 팩트를 찾는다. 그레인은 차원과 팩트보다 먼저 정한다. 네 가지를 정하기 전에 요구사항부터 파악한다. 비즈니스 사용자가 보려는 성과와 원천 시스템에 실제로 있는 데이터를 함께 확인한다.

**1. 비즈니스 프로세스 선택.** 비즈니스 프로세스(business process)는 조직이 수행하는 운영 활동이다. 주문 접수나 배송처럼 성과 지표가 나오는 활동을 말한다. "주문을 받는다"처럼 동사로 쓸 수 있으면 프로세스다. 부서, 보고서, 대시보드는 프로세스가 아니다. 팩트 테이블 하나는 대부분 프로세스 하나의 결과를 담는다.

**2. 그레인 선언.** 그레인(grain)은 팩트 테이블의 한 행이 나타내는 단위다. "한 행 = 주문서의 상품 한 줄"처럼 문장으로 적는다. 차원과 팩트를 고르기 전에 선언한다. 뒤에서 고르는 모든 차원과 팩트가 이 그레인과 맞아야 하기 때문이다. 가장 원자적인 수준에서 시작하는 것이 원칙이다. 원자 데이터는 나중에 어떤 기준으로도 묶을 수 있지만, 집계해 둔 데이터는 다시 쪼갤 수 없다. 그레인이 다르면 팩트 테이블을 따로 만들고, 한 테이블에 그레인을 섞지 않는다.

**3. 차원 식별.** 차원(dimension)은 측정이 일어난 맥락을 설명하는 테이블이다. 육하원칙으로 찾는다.

| 질문 | 차원 |
| --- | --- |
| 언제 | 주문일, 배송일 |
| 누가 | 고객 |
| 무엇을 | 상품 |
| 어디서 | 판매 채널 |
| 왜 | 쿠폰, 프로모션 |

그레인이 원자적일수록 붙일 수 있는 차원이 많아진다. 주문 한 줄에는 상품을 붙일 수 있지만, 일별 매출 합계에는 붙일 수 없다.

**4. 팩트 식별.** 팩트(fact)는 그 프로세스에서 측정된 숫자다. 수량, 금액 같은 값이다. 선언한 그레인과 맞는 값만 넣는다. 주문 한 줄이 그레인인 테이블에 주문 전체의 배송비를 줄마다 넣으면, 합계를 낼 때 배송비가 줄 수만큼 곱해진다.

그레인이 맞는지는 아래 질문으로 확인한다. Kimball 문서의 표현이 아니라 공부하면서 정리한 기준이다. **"이 컬럼을 테이블의 모든 행에 대해 더해도 의미가 같은가?"** 아니라면 그 컬럼은 다른 그레인에 속한다.

## 팩트

팩트 테이블(fact table)은 프로세스 하나에서 나온 측정값을 모아 둔 테이블이다. 설계할 때는 두 가지를 정한다. 어떤 값을 더해도 되는지(가산성)와, 행이 언제 생기고 언제 바뀌는지(유형)다.

### 구조와 키

팩트 테이블에는 세 종류의 컬럼이 있다.

- **차원 외래키**: 차원 테이블과 조인할 때 쓰는 키다.
- **측정값**: 수량, 금액 같은 숫자다.
- **퇴화 차원(degenerate dimension)**: 주문번호처럼 따로 차원 테이블을 만들지 않고 팩트에 그대로 두는 식별자다.

행은 그레인에 해당하는 외래키와 퇴화 차원을 조합해서 식별한다. 측정값은 null이어도 된다. 합계나 평균 같은 집계 함수가 null을 빼고 계산하기 때문이다. 외래키에는 null을 넣지 않는다. 차원에 "해당 없음" 행을 만들어 두고 그 키를 넣는다.

### 가산성 3종

가산성(additivity)은 측정값을 어떤 기준으로 합산해도 되는지를 나타내는 성질이다.

| 종류 | 뜻 | 예 | 다루는 법 |
| --- | --- | --- | --- |
| 가산(additive) | 어느 차원으로 묶어도 합산 가능 | 수량, 판매 금액 | 그대로 저장하고 합산 |
| 반가산(semi-additive) | 일부 차원으로만 합산 가능. 보통 시간 축으로는 불가 | 재고, 잔액 | 시간 축은 평균이나 기간 말 값으로 집계 |
| 비가산(non-additive) | 합산하면 의미가 없음 | 비율, 단가 | 분자와 분모를 저장하고, 집계한 뒤에 계산 |

비가산 값은 계산한 결과를 팩트에 저장하지 않는다. 행마다 저장해 둔 단가를 평균 내면 틀린 값이 나온다. 금액 합계를 수량 합계로 나눠야 맞다. 그래서 팩트에는 분자와 분모를 따로 저장하고, 비율은 지표를 뽑을 때 계산한다.

### 유형 4종

유형은 한 행이 나타내는 대상과 행이 바뀌는 시점으로 나눈다. Kimball Group은 트랜잭션, 주기 스냅샷, 누적 스냅샷을 기본 유형으로 설명한다. 팩트리스는 측정값이 없는 팩트 테이블이고 별도 기법으로 다룬다.

| 유형 | 한 행 | 판단 질문 | 행이 생기고 바뀌는 시점 |
| --- | --- | --- | --- |
| 트랜잭션(transaction) | 한 시점에 일어난 사건 1건 | 한 번에 일어난 사건인가? | 사건이 생기면 추가. 이후 바뀌지 않음 |
| 주기 스냅샷(periodic snapshot) | 정해진 주기(일, 주, 월)의 대상별 상태 | 특정 시점의 상태를 보려는가? | 주기마다 대상 × 날짜 행을 추가. 활동이 없어도 행을 만듦 |
| 누적 스냅샷(accumulating snapshot) | 시작과 끝이 있는 프로세스 1건 | 단계 사이에 걸린 시간을 보려는가? | 시작할 때 추가하고, 단계가 진행될 때마다 같은 행을 갱신 |
| 팩트리스(factless) | 측정값 없이 사건 발생이나 관계만 기록 | 건수만 세면 되는가? | 사건 기록은 사건이 생기면 추가. 가능한 조합을 미리 담는 커버리지 테이블도 있음 |

트랜잭션 팩트는 측정이 일어났을 때만 행이 생긴다. 새 행만 추가하면 되므로 증분 적재로 처리하기 쉽다. 늦게 들어오거나 정정되는 데이터가 있으면 키 기준으로 병합(merge)한다. 적재 방식은 Kimball 문서의 내용이 아니라 구현 관행이다.

주기 스냅샷은 그레인이 개별 사건이 아니라 주기다. 그 주기에 활동이 없어도 행을 넣고 값은 0이나 null로 둔다. 재고나 잔액 같은 반가산 값은 대부분 여기에 들어간다. 구현할 때는 날짜 목록과 대상 목록을 곱해서 행을 먼저 만들고 값을 채우는 방식을 많이 쓴다(구현 관행).

누적 스냅샷은 주문 접수, 결제, 출고, 배송 완료처럼 단계가 정해진 프로세스에 쓴다. 단계마다 날짜 키를 두고, 단계 사이에 걸린 시간을 측정값으로 둔다. 기본 유형 셋 중에 행을 갱신하는 것은 누적 스냅샷뿐이다.

### 대표 패턴

- **집계 팩트(aggregate fact)**: 원자 팩트를 미리 집계해 둔 테이블이다. 조회 성능을 높이려는 목적으로만 만들고, 원자 팩트와 함께 둔다.
- **통합 팩트(consolidated fact)**: 그레인이 같은 여러 프로세스의 값을 한 테이블에 나란히 둔 것이다. 계획 대비 실적 비교에 많이 쓴다.
- **헤더와 라인**: 주문 전체에 걸린 값(배송비)과 줄 단위 값(수량)은 그레인이 다르다. 주문 단위 값은 규칙을 정해 줄 단위로 배분해서 넣는다. 주문의 차원 키와 주문번호도 줄 단위 팩트에 넣는다.

## 차원

차원 테이블(dimension table)은 팩트를 걸러 보거나 묶어 볼 때 쓰는 속성을 모아 둔 테이블이다. 조회하는 사람이 바로 쓸 수 있게 만드는 것이 기준이다.

### 구조

차원은 컬럼이 많은 비정규화 테이블로 만든다. 상품, 카테고리, 대분류를 테이블 세 개로 나누지 않고 상품 차원 하나에 컬럼으로 넣는다. 조인이 줄고, 사용자는 테이블 하나만 보면 된다. 차원은 팩트보다 훨씬 작아서 값이 중복 저장되어도 부담이 작다.

**"보고서에 그대로 찍어도 읽히는 값"이란.** 차원 속성은 보고서의 행 머리글이나 필터 값으로 그대로 나간다. 그래서 코드가 아니라 사람이 읽을 수 있는 말로 저장하라는 뜻이다.

| 피할 값 | 권장 값 |
| --- | --- |
| 채널 코드 `A01` | `모바일 앱` |
| `Y` / `N` | `쿠폰 사용` / `쿠폰 미사용` |
| null | `해당 없음` |

코드로 조회하는 사람이 있으면 코드 컬럼과 설명 컬럼을 둘 다 둔다.

**"카디널리티가 낮은 게 좋다"란.** 카디널리티(cardinality)는 한 컬럼에 들어 있는 서로 다른 값의 개수다. Kimball Group은 차원 테이블을 카디널리티가 낮은 텍스트 속성이 많은 넓은 테이블로 설명한다. 차원 테이블의 행 수가 아니라 속성 하나하나에 대한 말이다. 카테고리처럼 값이 수십 개인 속성으로 묶으면 요약이 된다. 주문번호처럼 행마다 값이 다른 속성으로 묶으면 요약이 안 된다. 고객 차원처럼 행이 많은 차원 자체는 문제가 아니다. 행마다 값이 다른 식별자는 퇴화 차원으로 팩트에 두고, 연속된 숫자는 구간으로 나눠서 속성으로 만든다.

### 대리키

대리키(surrogate key)는 원천 시스템의 키와 상관없이 웨어하우스에서 새로 매기는 일련번호 키다. 차원의 기본키로 쓴다. 쓰는 이유는 네 가지다.

- 원천 키가 바뀌거나 재사용되어도 영향을 받지 않는다.
- 여러 원천에 흩어진 같은 대상을 하나로 합칠 수 있다.
- 같은 고객의 과거 버전과 현재 버전을 다른 행으로 둘 수 있다.
- "해당 없음", "미상" 같은 행을 만들 수 있다.

원천 키는 버리지 않고 속성으로 남긴다. 날짜 차원은 예외다. 값이 예측 가능하고 바뀌지 않아서 YYYYMMDD 같은 날짜 형태의 정수를 키로 쓸 수 있다.

### 변화 추적

변화 추적은 차원 속성 값이 바뀌었을 때 이전 값을 어떻게 남길지 정하는 규칙이다. 보통 SCD(Slowly Changing Dimension, 천천히 변하는 차원) 유형이라고 부른다.

| 유형 | 방식 | 과거 이력 |
| --- | --- | --- |
| 0 | 원래 값을 유지하고 바꾸지 않음 | 처음 값만 남음 |
| 1 | 덮어쓰기 | 없음 |
| 2 | 새 행 추가 (유효 기간과 현재 여부 표시) | 전부 남음 |
| 3 | 이전 값을 담는 컬럼 추가 | 직전 값만 남음 |
| 4 | 자주 바뀌는 속성을 미니 차원으로 분리 | 미니 차원에 남음 |
| 5 | 4에 더해, 기본 차원에 현재 미니 차원 키를 덮어쓰기로 둠 | 4와 같고 현재 값 조회가 쉬움 |
| 6 | 2의 행에 현재 값 컬럼을 추가하고 덮어씀 | 전부 남고 현재 값도 같이 봄 |
| 7 | 팩트에 키를 두 개 둠 (당시 버전용, 현재 버전용) | 6과 같은 결과를 키로 구현 |

**실무에서 몇 번까지 쓰는가.** Kimball Group은 1, 2, 3을 핵심 기법으로 부르고, 0과 4\~7은 이전부터 설명하던 고급·혼합 기법에 2013년 The Data Warehouse Toolkit 3판에서 번호를 붙인 것이다. 그래서 "3까지가 기본"이라는 말은 맞다. 다만 얼마나 자주 쓰는지는 다르다. Kimball Group도 유형 3은 비교적 드물게 쓴다고 설명한다. 조직 개편처럼 이전 기준과 현재 기준을 나란히 비교해야 할 때 쓴다. 어느 유형을 가장 많이 쓰는지에 대한 통계는 원문에서 찾지 못했다. 4\~7은 정의만 읽었고 직접 만들어 보지 않았다.

유형은 테이블 단위가 아니라 속성 단위로 정한다. 고객 이름 오타 수정은 1, 고객 등급 변경은 2처럼 한 차원 안에 여러 유형이 같이 있다.

### 계층

계층(hierarchy)은 일, 월, 분기, 연처럼 값을 단계적으로 묶어 올라가는 관계다. 깊이가 고정된 계층은 차원 하나에 단계별 컬럼으로 넣는다. 조직도처럼 깊이가 일정하지 않은 계층은 컬럼으로 넣을 수 없어서, 계층 안의 모든 경로를 한 행씩 담는 브리지 테이블(bridge table)을 따로 둔다.

### 대표 패턴

- **역할 수행 차원(role-playing dimension)**: 날짜 차원 하나를 주문일, 배송일처럼 여러 용도로 조인해서 쓰는 것이다. 용도마다 뷰를 만들어 이름을 구분한다.
- **정크 차원(junk dimension)**: 값이 몇 개 안 되는 플래그와 상태 값을 한 차원에 모은 것이다. 가능한 모든 조합이 아니라 원천 데이터에 실제로 나타난 조합만 담는다. 팩트에 플래그 컬럼이 계속 늘어나는 것을 막는다.
- **날짜 차원(date dimension)**: 요일, 주차, 공휴일 여부를 미리 계산해 둔 달력 테이블이다. 거의 모든 팩트가 조인한다.

## 실행과 검증: 팩트 유형 고르기

팩트 유형은 무엇을 보려는지에 따라 고르고, 고른 뒤에는 그레인이 지켜지는지 검사한다. 한 시점의 사건이면 트랜잭션 팩트부터 검토한다. 세 질문 어디에도 해당하지 않으면 그레인 문장을 다시 쓴다.

{{< figure src="/images/2026-11-15-kimball/fig2-fact-type-flow.png" alt="팩트 유형 선택 흐름" caption="그림 2. 팩트 유형 선택 흐름" >}}

유형을 정한 뒤에는 세 가지를 확인한다.

| 확인 | 방법 | 막는 문제 |
| --- | --- | --- |
| 그레인 유일성 | 그레인에 해당하는 키 조합으로 묶어 2건 이상인 행이 없는지 검사 | 중복 행 때문에 생기는 이중 집계 |
| 혼합 그레인 | 컬럼마다 "모든 행을 더해도 의미가 같은가" 확인 | 주문 전체 값이 줄 수만큼 곱해지는 문제 |
| 가산성 표시 | 컬럼마다 가산, 반가산, 비가산을 문서에 기록 | 잔액을 기간으로 합산하거나 비율을 더하는 실수 |

유일성 검사는 아래처럼 쓸 수 있다. 결과가 0행이면 통과다.

```sql
-- 예시: 한 행 = 주문서의 상품 한 줄
select order_number, line_number, count(*) as row_count
from fact_order_line
group by order_number, line_number
having count(*) > 1;
```

이 검사 목록은 일반적인 관행을 정리한 것이고, 아직 직접 적용해 보지는 않았다.

## 통합

통합은 서로 다른 프로세스의 팩트를 같은 차원으로 나란히 볼 수 있게 맞추는 작업이다. 적합 차원이 핵심이다.

**적합 차원.** 적합 차원(conformed dimension)은 여러 팩트 테이블이 같이 쓰는 차원이다. 컬럼 이름과 값 목록이 같아야 한다. 주문 팩트와 배송 팩트가 같은 고객 차원을 쓰면 두 결과를 고객 기준으로 나란히 놓을 수 있다. 한 번 정의해서 여러 팩트에서 재사용하고, 정의할 때는 그 데이터를 관리하는 담당자들과 합의해야 한다. 월 차원처럼 날짜 차원을 한 단계 묶어 올린 차원도 적합한 것으로 본다.

**버스 매트릭스.** 버스 매트릭스(bus matrix)는 행에 비즈니스 프로세스, 열에 적합 차원을 놓고 어느 프로세스가 어느 차원을 쓰는지 표시한 표다. 행을 따라 읽으면 그 프로세스에 붙는 차원이 보이고, 열을 따라 읽으면 여러 프로세스가 같이 써야 하는 차원이 보인다. 구현은 한 번에 한 행씩 한다. 프로세스를 하나씩 구현하더라도 나중에 서로 연결되도록 잡아 주는 설계도로 쓴다. 적합 차원을 공유하면서 프로세스 단위로 나눠 구축하는 이 방식을 버스 아키텍처(bus architecture)라고 부른다.

| 프로세스 | 날짜 | 고객 | 상품 | 판매 채널 | 프로모션 |
| --- | --- | --- | --- | --- | --- |
| 주문을 받는다 | ● | ● | ● | ● | ● |
| 배송한다 | ● | ● | ● |  |  |
| 재고를 보관한다 | ● |  | ● |  |  |
| 반품을 처리한다 | ● | ● | ● | ● |  |

**드릴 어크로스.** 드릴 어크로스(drill-across)는 여러 팩트의 값을 한 보고서에 나란히 놓는 조회 방식이다. 팩트끼리 직접 조인하지 않는다. 팩트마다 같은 적합 차원 속성으로 따로 집계한 뒤, 그 속성 값을 기준으로 결과를 정렬해서 병합한다. 그레인이 다른 팩트를 직접 조인하면 행이 늘어나서 합계가 틀어진다.

**적합 팩트.** 적합 팩트(conformed fact)는 여러 팩트 테이블에서 정의와 단위가 같은 측정값이다. 정의가 다르면 이름도 다르게 붙인다.

들어가기에서 한 고민도 여기서 정리됐다. 서비스는 버스 매트릭스의 행이 아니라 열에 들어간다. 팩트를 서비스별로 나누지 않고, 서비스를 적합 차원으로 만들어 모든 팩트가 참조하게 할 생각이다. 이 부분은 설계만 했고 아직 구현하지 않았다.

## 마치며

**배운 점.** 네 영역 모두 그레인에서 출발한다. 팩트 유형은 한 행이 무엇인지에 따라 정해지고, 가산성은 그 행을 더해도 되는지를 따지며, 차원은 그 행에 붙일 수 있는 맥락이다. 막힐 때마다 "한 행이 무엇인가"부터 다시 따져 보면 답이 나왔다. "팩트는 프로세스로 나누고 나머지는 차원으로 둔다"는 원칙도 여기서 나온다.

**아직 안 한 것.** 이 글은 학습 정리다. 실제 테이블에는 아직 적용하지 않았다. 변화 추적 유형 4\~7, 깊이가 일정하지 않은 계층, 브리지 테이블은 정의만 읽었고 직접 만들어 보지 않았다.

**다음 단계.** 제품 이벤트 로그에 이 절차를 적용해 볼 생각이다. 프로세스를 고르고 그레인을 문장으로 선언한 뒤, 유일성 검사까지 붙이는 것이 목표다. 적용하다가 이 글의 설명과 다른 점이 나오면 고쳐 적겠다.

**대조 범위.** 설명은 Kimball Group의 기법 문서를 문서별로 요약해서 대조했다. 다음 네 가지는 그 문서에서 해당 설명을 찾지 못했고, 책(The Data Warehouse Toolkit)과는 아직 대조하지 않았다. 프로세스를 동사로 쓴다는 기준, 부서와 보고서는 프로세스가 아니라는 설명, 팩트 행을 외래키와 퇴화 차원의 조합으로 식별한다는 설명, 차원이 팩트보다 작아서 중복 저장 부담이 작다는 설명이다.

### 참고 자료

- [Kimball Group, Dimensional Modeling Techniques](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/) (기법 목록. 아래는 이 목록에서 열어 본 문서다)
- 설계 절차: [Four-Step Dimensional Design Process](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/four-4-step-design-process/), [Business Processes](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/business-process/), [Grain](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/grain/)
- 팩트: [Fact Table Structure](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/fact-table-structure/), [Nulls in Fact Tables](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/fact-table-null/), [Additive, Semi-Additive, Non-Additive Facts](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/additive-semi-additive-non-additive-fact/), [Transaction Fact Tables](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/transaction-fact-table/), [Periodic Snapshot Fact Tables](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/periodic-snapshot-fact-table/), [Accumulating Snapshot Fact Tables](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/accumulating-snapshot-fact-table/), [Factless Fact Tables](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/factless-fact-table/), [Aggregated Fact Tables](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/aggregate-fact-table-cube/), [Consolidated Fact Tables](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/consolidated-fact-table/), [Header/Line Fact Tables](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/header-line-fact-table/), [Allocated Facts](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/allocated-fact/)
- 차원: [Dimension Table Structure](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dimension-table-structure/), [Dimension Surrogate Keys](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dimension-surrogate-key/), [Degenerate Dimensions](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/degenerate-dimension/), [Denormalized Flattened Dimensions](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/denormalized-flattened-dimension/), [Flags and Indicators](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/flag-indicator-attribute/), [Null Attributes in Dimensions](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/null-dimension-attribute/), [Calendar Date Dimension](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/calendar-date-dimension/), [Role-Playing Dimensions](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/role-playing-dimension/), [Junk Dimensions](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/junk-dimension/), [Fixed Depth Hierarchies](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/fixed-depth-hierarchy/), [Ragged Hierarchies](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/ragged-variable-depth-hierarchy/)
- 변화 추적: [Type 2: Add New Row](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/type-2/), [Type 3: Add New Attribute](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/type-3/), [Design Tip #152: Slowly Changing Dimension Types 0, 4, 5, 6 and 7](https://www.kimballgroup.com/2013/02/design-tip-152-slowly-changing-dimension-types-0-4-5-6-7/)
- 통합: [Conformed Dimensions](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/conformed-dimension/), [Shrunken Rollup Dimensions](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/shrunken-rollup-dimension/), [Drilling Across](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/drilling-across/), [Multipass SQL](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/multipass-sql/), [Bus Architecture](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/enterprise-data-warehouse-bus-architecture/), [Bus Matrix](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/enterprise-data-warehouse-bus-matrix/), [Conformed Facts](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/conformed-fact/)
