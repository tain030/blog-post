---
title: 'Test Pagenation3'
date: '2025-01-30'
category: 'blockchain'
---

## **목차**

1. Kademlia란?

2. Kademlia 핵심 개념
   1) XOR Metric: Kademlia의 거리 개념
   2) Node State: K-bucket과 라우팅 테이블 구조
   3) Kademlia Protocol: RPC를 통한 노드 탐색 및 데이터 저장
   4) Routing Table: 효율적인 검색을 위한 라우팅 테이블 최적화
   5) Efficient Key Re-Publishing: 키-값 저장의 지속성과 네트워크 부하 관리

3. 요약 및 결론

---

## Kademlia란?

Kademlia는 **분산 해시 테이블(Distributed Hash Table, DHT) 기반의 P2P 프로토콜**이다.
📌 **XOR 거리(XOR Metric)를 활용한 효율적인 노드 탐색과 데이터 저장**이 특징이며,
📌 **빠른 검색 속도, 적은 네트워크 트래픽, 높은 내결함성(fault tolerance)을 제공**한다.

💡 **주요 사용 사례:**

- **BitTorrent의 Magnet 링크 검색**
- **Ethereum DevP2P의 노드 탐색**
- **IPFS의 콘텐츠 검색**

이제 Kademlia의 핵심 개념을 하나씩 살펴보자.

---

## **2. XOR Metric: Kademlia의 거리 개념**

Kademlia는 **노드 간의 거리를 XOR 연산을 이용해 정의**한다.

- **각 노드는 고유한 160비트 ID를 가짐**
- 두 노드 간 거리는 다음과 같이 계산됨: distance(A,B)=A⊕B\text{distance}(A, B) = A \oplus Bdistance(A,B)=A⊕B
- **XOR 거리 특징:**
    - 값이 작을수록 가까운 노드
    - **대칭적** (distance(A, B) = distance(B, A))
    - 트리 구조로 노드를 그룹화하여 탐색 효율을 높일 수 있음

💡 **예제:**

- `Node A = 0011`, `Node B = 0100`, `Node C = 0101`이라고 하자.
- `A ⊕ B = 0111` → **거리는 3비트 차이지만 XOR 거리상 `0xxx` 서브트리라서 실제 XOR 거리는 1**
- `B ⊕ C = 0001` → **거리는 1비트 차이지만 `010x` 서브트리라서 실제 XOR 거리는 3**

📌 **즉, 단순히 비트 차이 개수가 아니라, "최소 공통 서브트리의 높이"가 XOR 거리!** 🚀

---

## **3. Node State: Kademlia의 라우팅 테이블과 K-bucket**

Kademlia의 **각 노드는 자신과 XOR 거리별로 다른 노드들의 정보를 저장**한다.

- 라우팅 테이블은 여러 개의 **k-bucket(최대 `k=20`개의 노드 저장)**으로 구성됨.
- **가까운 노드는 자주 교류하며**, 먼 노드는 덜 교류함.
- 새로운 노드가 추가될 때:
    1. k-bucket이 비어 있으면 추가
    2. k-bucket이 가득 찼다면, 기존 노드가 오래된(stale) 경우 교체
    3. 그렇지 않으면 새로운 노드를 **Replacement Cache**에 저장

📌 **Replacement Cache란?**

- k-bucket이 가득 찼을 때 **새로운 노드를 버리지 않고 후보군으로 저장**
- 기존 노드가 응답하지 않으면 **Replacement Cache에서 대체 노드를 선택하여 교체**

---

## **4. Kademlia Protocol: RPC 기반 노드 탐색 및 데이터 저장**

Kademlia의 핵심 기능은 **RPC(Remote Procedure Call)를 이용한 노드 탐색과 데이터 저장**이다.

📌 **주요 RPC 종류:**

|RPC|역할|
|---|---|
|`PING`|노드가 살아 있는지 확인|
|`STORE`|특정 `key-value`를 저장|
|`FIND_NODE`|특정 `NodeID`에 가장 가까운 노드 찾기|
|`FIND_VALUE`|특정 `key`의 value 찾기|

💡 **📌 Node Lookup 과정:**

1. XOR 거리가 가까운 `k`개의 노드를 선택하여 `FIND_NODE` RPC 전송
2. 응답한 노드 중에서 **더 가까운 노드를 발견하면** 다시 요청
3. 더 이상 가까운 노드가 없으면 검색 종료

📌 **Accelerated Lookups (빠른 검색 최적화)**

- 기존 방식: 한 번에 **1비트**씩 XOR 거리 기반 탐색
- 최적화 방식: 한 번에 **`b`비트**씩 XOR 거리 기반 탐색 → **hop 수 감소** 🚀

---

## **5. Routing Table: Kademlia의 효율적인 라우팅 테이블 관리**

📌 **Kademlia의 라우팅 테이블 크기가 커질수록 hop 수가 줄어듦!**

- 라우팅 테이블을 크기를 증가시키면, **한 번에 더 먼 거리로 이동 가능**
- 이를 통해 **검색 속도가 향상됨**

📌 **Unresponsive 노드 관리 (노드가 응답하지 않을 때)**

- 패킷 손실이 발생하면, **해당 노드를 일정 시간 동안 "lock"**
- **Exponential Backoff 적용:**
    - 첫 번째 실패 → 1초 후 재시도
    - 두 번째 실패 → 2초 후 재시도
    - 세 번째 실패 → 4초 후 재시도 …

---

## **6. Efficient Key Re-Publishing: 지속적인 데이터 유지 및 부하 관리**

📌 **Kademlia에서 key-value가 사라지지 않도록 하기 위한 전략:**

1. `STORE`를 받은 노드는 **즉시 재게시하지 않음**
    - 왜? `k`개의 노드가 이미 저장했을 것이라고 가정하기 때문!
    - 따라서 **불필요한 중복 저장을 방지**하여 네트워크 부하를 줄임
2. 일정 시간이 지나면, 노드들은 key-value를 다시 게시하여 **데이터 지속성을 보장**

💡 **즉, "1시간 동안 재게시하지 않는 전략"을 통해 네트워크 트래픽을 최적화할 수 있음!** 🚀

---

## **7. 결론 및 다음 포스팅 안내**

Kademlia는 **XOR 거리 기반 탐색, 효율적인 라우팅 테이블 관리, 최적화된 노드 검색 및 데이터 저장 방식**을 통해
분산 P2P 네트워크에서 **빠르고 안정적인 DHT 시스템**을 제공한다.

💡 **다음 포스팅에서는 Kademlia의 실제 구현을 다룰 예정!**
**예제 코드와 함께 Kademlia를 Rust/Python으로 직접 구현해보자.** 🚀

---

### **🎯 최종 정리**

|개념|설명|
|---|---|
|**XOR 거리**|`distance(A, B) = A ⊕ B`, 최소 공통 서브트리 기준|
|**k-buckets**|XOR 거리 기반으로 노드 저장, Replacement Cache 유지|
|**노드 탐색**|`FIND_NODE`, `FIND_VALUE`를 이용한 효율적 검색|
|**Routing Table 최적화**|`b`비트씩 고려하여 hop 수 감소|
|**Key Re-Publishing**|1시간 동안 재게시 방지하여 트래픽 감소|

👉 **Kademlia는 효율적이고 확장성이 뛰어난 DHT 시스템이다!** 🚀
