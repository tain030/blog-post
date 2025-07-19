# Key Takeaways

- Uncertified DAG
- 클라이언트 브로드캐스팅
- 암묵적 인증

# 용어 설명

## 분산시스템의 딜레마 (FLP Impossibility)

완벽한 비동기 네트워크 내에서는 Safety와 Liveness를 모두 완벽히 만족하는 합의 알고리즘을 설계하는 것이 불가능하다는 것이 증명되었습니다. 이 증명을 **“FLP Impossibility”** 라고 합니다.

## Safety, Liveness

- **Safety**
    
    만약 합의가 이루어졌다면, 모든 정상적인 노드는 **동일한 값**에 접근 가능합니다. 즉, 철수와 영희가 정직하게 노드를 운영한다면, 서로 다른 블록체인이 아닌 하나의 같은 블록체인을 각각 유지할 수 있습니다. 우리가 아는 블록체인을 기준으로 말한다면 Safety가 보장되지 않을 경우, 같은 블록 높이에 두 개의 블록이 생성될 수 있습니다. 즉, 포크가 발생할 수 있습니다.
    
    블록체인에서 Safety는 반드시 보장되어야 합니다. 
    
- **Liveness**
    
    모든 정상적인 노드는 결국 합의에 도달합니다. 만약 Liveness가 보장되지 않을 경우 분산된 시스템을 구성하는 노드들이 무한히 합의에 도달하지 못하는, 즉 새로운 블록이 계속해서 만들어지지 않는 상태가 유지될 수 있습니다.
    

## Reliable Broadcast

합의 알고리즘은 내부적으로 Reliable Broadcast를 사용(하거나 그와 동등한 속성을 구현)합니다.

"한 명의 정직한 노드(sender)가 보낸 메시지를, 시스템의 모든 정직한 노드가 똑같이 수신하도록 보장하는 것" 이 유일한 목표입니다.

### 특징 및 보장 속성 (Properties)

1. 무결성 (Integrity): 정직한 노드는 최대 한 번만 메시지를 전달(deliver)받는다. 또한, 전달받은 메시지는 반드시 최초에 sender가 보낸 원본 메시지여야 한다. (악의적인 노드가 메시지를
위조하거나 변조할 수 없다.)
2. 전체성 (Totality): 만약 어떤 정직한 노드 하나라도 메시지 M을 전달받았다면, 결국 모든 다른 정직한 노드들도 반드시 똑같은 메시지 M을 전달받는다. (일부만 받고 일부는 못 받는 "분열" 상태가
발생하지 않는다.)
3. 유효성 (Validity): 만약 메시지를 보낸 sender가 정직하다면, 그가 보낸 메시지는 결국 모든 정직한 노드에게 전달된다. (악의적인 노드가 메시지 전파를 막을 수 없다.)

### 장점

- 상대적으로 가벼움: 합의에 비해 통신 복잡도나 요구 조건이 낮습니다.
- 명확한 책임: "하나의 메시지를 모두에게 똑같이"라는 명확한 역할만 수행합니다.
- 모듈성: 합의 알고리즘을 만들 때, "메시지 전파는 RBC가 알아서 해주겠지"라고 가정하고 더 상위 레벨의 로직에 집중할 수 있게 해주는 강력한 부품이 됩니다.

### 단점 및 한계

- 순서를 보장하지 않음 (No Ordering): RBC는 메시지의 전달 순서에 대해서는 아무것도 보장하지 않습니다. 노드 A가 보낸 메시지 M1과 노드 B가 보낸 메시지 M2가 있을 때, 어떤 노드는 M1을 먼저
받고, 다른 노드는 M2를 먼저 받을 수 있습니다. 이것이 합의와의 가장 결정적인 차이점입니다.

## 비잔틴 장군 문제

![image.png](attachment:5b93285a-9ee1-4ce9-bfb8-1bfbf386bb42:image.png)

스파이(악의적인 노드)가 f명 있을 때 안전하려면 최소 3f+1명의 장군(노드)가 있어야 한다는 것이 BFT(Byzantine Fault Tolerance)이며, PBFT는 이 조건에 기반한 **실용적 합의 알고리즘**입니다.

$$
n = 3f + 1
$$

<aside>
💡

**$2f+1$**: 다수의 동의를 얻는 데 필요한 최소 숫자

</aside>

## 지연시간(Latency) vs. 처리량(Throughput)

지연 시간은 단일 트랜잭션 처리에 소요되는 시간을, 처리량은 단위 시간당 처리 가능한 트랜잭션 수를 의미

- **Latency (지연 시간)**:
    
    트랜잭션이 제출된 후 블록체인에 **최종 확정**되기까지 걸리는 시간 (즉, finality 시간)
    
- **Throughput (처리량)**:
    
    시스템이 **단위 시간당 처리 가능한 트랜잭션 수** (보통 초당 트랜잭션 수, TPS)
    
- DAG 기반 합의는 **병렬성**을 통해 throughput을 크게 높일 수 있어요.
    
    👉 동시에 여러 블록을 생성하고, 여러 트랜잭션을 병렬로 포함 가능 → TPS 상승
    
- 하지만 이로 인해 **"언제 커밋할 수 있는가?"를 결정하는 룰이 복잡**해지고,
    
    👉 여러 wave를 기다려야 하는 구조가 되어 **latency는 높아짐**
    

## DAG (Directed Acyclic Graph)

## PBFT

![image.png](attachment:fcb6d7f7-a241-4311-ac87-3ad50e9cdddb:image.png)

```jsx
pre-prepare -> prepare -> commit
```

1. **request :** 클라이언트(C)로부터 primary node(0)으로 블록 생성에 대한 합의 요청 메세지가 들어온다.
2. **pre-prepare :** primary는 해당 메세지를 검증하고, 복사하여 replica node(1,2,3)에 브로드캐스팅한다.
3. **prepare :** 각각의 replica node들은 전달받은 메세지를 확인하고 올바른지 검증한 후 본인을 제외한 다른 노드들에게 브로드캐스팅한다.
4. **commit :** prepare의 과정을 반복 수행한다.
5. **reply :** 각각의 replica node들은 일정 수 이상의 메세지를 받으면 클라이언트의 요청을 수용하는 것으로 합의하고, 클라이언트에게 합의 완료 메세지를 보낸다.

PBFT에서는 각 단계마다 어떤 정보를 담은 메시지를 누구에게, 어떠한 방식으로 전송하고 수신할지가 정해져 있습니다. 그러나 각 단계별로 제한 시간이 정해져 있지 않기 때문에 **비동기 네트워크 환경**이라고 할 수 있습니다. 그렇기에 중간에 메시지가 무한히 지연되면서 특정 단계에서 합의가 멈추는 경우가 있는데 이러한 경우 Liveness를 보장하지 못하게 됩니다. 예를 들어, 블록을 제안하는 단 하나의 노드가 멈추게 된다면 다음 단계로 나아가지 못하는 경우가 발생합니다. 즉, PBFT에서는 합의가 이루어진다면 모든 정상적인 노드가 단 하나의 블록에 합의할 수 있기 때문에 Safety는 보장되지만, 합의가 기약 없이 지연될 수도 있기 때문에 Liveness는 보장되지 않을 수 있습니다.

따라서 PBFT는 Liveness를 어느 정도 보장하기 위해서 **동기 네트워크를 일부 도입**합니다. 즉, 최초에 블록을 제안하는 단 하나의 노드가 멈추어 제한 시간 안에 합의의 과정이 진행되지 못하는 경우, ‘view-change’라는 기능을 통해 다른 블록 제안자를 선정, 네트워크에 참여하는 멤버를 추가 및 탈퇴함으로써 변경하여 강제로 다음 블록 생성 단계로 넘어가게 됩니다. 바로 여기서 ‘제한 시간을 둔다’라는 부분이 동기 네트워크를 도입하는 부분입니다.

## Tendermint (DPoS + PBFT)

```jsx
propose -> pre-vote -> pre-commit
```

PBFT에는 가장 큰 걸림돌이 있는데, 바로 많은 메시지 교환으로 인한 **네트워크 지연 및 과부화**입니다. 이러한 문제 때문에 PBFT 합의 방식은 노드 수가 많지 않은 **프라이빗 블록체인에서만** 쓸 수 있었습니다.

하지만 이 pBFT가 DPoS와 만나 퍼블릭 블록체인에 사용되기 시작합니다. DPoS는 지분을 대리인에게 위임하여 합의에 간접적으로 참여하는 PoS로, 미국의 간접 민주주의와 비슷하다고 보면 됩니다.

### 특징

- Gossip 프로토콜
- 리더 1명
- `2f+1`개의 메시지
- 타임아웃 → View Change 메시지
- 비파이프라이닝

### 핵심 메커니즘

리더가 제안한 블록 A가 `2f+1`개의 pre-vote를 받으면 해당 블록을 Lock한 후 pre-commit을 진행하며, `2f+1`개의 pre-commit을 받으면 commit합니다. Timeout이 지나면 다음 라운드로 진행되고 새로운 리더가 새로운 블록을 제안하게 됩니다. 이 과정에서 Timeout까지 블록 A를 받지 못한 노드가 리더가 된다면 블록 A가 합의된 것을 모른채 블록 A와 전혀 관계없는 블록 B를 제안할 가능성이 있습니다.

이전 라운드에서 블록 A를 Lock했던 노드들은 이 블록 B를 거부하게 되고 블록 B는 합의되지 못한 채 다음 라운드로 넘어가게 됩니다. 이렇게 블록 A와 관계없는 블록이 commit되어 체인이 포크되는 것을 방지하고 Safety를 가지게 됩니다.

### 문제점

Hidden Lock Problem

## HotStuff

![image.png](attachment:7431d42b-2e6e-4f58-b23e-20cab7d2373e:image.png)

```jsx
prepare -> pre-commit -> commit
```

1. **Prepare**
    1. 리더
    새로운 블록 B를 제안하고, 이전 라운드의 가장 높은 QC(HighQC)를 포함하여 노드들에게 `PREPARE` 메시지를 보냅니다.
    2. 노드
    `PREPARE` 메시지를 받은 노드는 제안된 블록 B가 자신의 현재 상태와 충돌하지 않는지, 그리고 HighQC가 유효한지 검증합니다. 만약 유효하면 블록 B에 대한 `PREPARE_VOTE`를 리더에게 보냅니다.
2. **Pre-commit**
    1. 리더
        
        블록 B에 대한 2f+1개 이상의 `PREPARE_VOTE`를 수집하여 `Prepare_QC`를 생성합니다. 이 `Prepare_QC`를 포함하여 노드들에게 `PRECOMMIT` 메시지를 보냅니다.
        
    2. 노드
        
        `PRECOMMIT` 메시지를 받은 노드는 `Prepare_QC`가 유효한지 검증하고, 만약 유효하면 블록 B에 대한 `PRECOMMIT_VOTE`를 리더에게 보냅니다.
        
3. **Commit**
    1. 리더
        
        블록 B에 대한 2f+1개 이상의 `PRECOMMIT_VOTE`를 수집하여 `Precommit_QC`를 생성합니다. 이 `Precommit_QC`를 포함하여 노드들에게 `COMMIT` 메시지를 보냅니다.
        
    2. 노드
        
        `OMMIT` 메시지를 받은 노드는 `Precommit_QC`가 유효한지 검증하고, 만약 유효하면 블록 B에 대한 `COMMIT_VOTE`를 리더에게 보냅니다. 이 시점에서 노드는 블록 B를 **최종적으로 확정(commit)**합니다.
        

### 특징

- 리더가 Propose 및 Broadcast, 다른 노드들은 응답만
- 리더 1명
- 2f+1의 **서명**
- 타임아웃 → 자동으로 다음 View 진행
- 파이프라이닝

### 핵심 메커니즘

HotStuff의 핵심적인 안전성 보장은 바로 **"세 체인(Three-Chain)" 규칙**에 있습니다. 이는 블록이 최종적으로 `commit` 되기 위해서는 적어도 세 개의 연속적인 QC 체인이 형성되어야 한다는 원리입니다.

HotStuff는 **리더가 하나의 블록에만 QC를 만들도록 강제하지 않습니다.** Byzantine 리더는 여러 블록을 제안하거나 포크를 만들 수도 있습니다. 때문에 다음 리더가 그 체인을 **계속 이어간다는 증거**가 있어야 합니다.

## Bullshark

Certified DAG

# 개요

블록체인과 같은 분산 시스템은 비잔틴 장군 문제에 대한 내성을 가져야 합니다. 이를 BFT(비잔틴 장애 허용)라 하며, 초기 퍼블릭 블록체인인 비트코인과 이더리움은 이를 **확률적으로** 달성합니다.

예를 들어 비트코인은 블록을 생성하는 데 막대한 연산 자원이 필요하도록 설계되어, 악의적인 공격자가 블록체인을 조작하려면 천문학적인 비용을 감수해야 합니다. 이더리움 또한 초기에는 비슷한 PoW(작업증명)를 사용했으며, 이후 PoS(지분증명) 체제로 전환하면서 공격 성공 시 금전적 손해를 입고, 재공격이 어렵도록 설계되어 있습니다.

그러나 이러한 확률적 BFT 방식은 일반 사용자에게도 **높은 수수료 부담**과 **거래 확정까지의 긴 대기 시간**이라는 단점을 안겨주었습니다. 이러한 한계로 인해 수이, 세이, 하이퍼리퀴드 같은 비교적 최신 블록체인들은 정족수(quorum) 기반으로 명확한 합의가 이루어지는 구조인 **결정적 BFT(Deterministic BFT)** 방식을 사용하고 있습니다.

결정적 BFT는 PBFT → Tendermint → HotStuff로 발전하며 **선형 블록체인 구조**에서 효율적인 합의를 추구해왔습니다. 그럼에도 불구하고 선형 구조로 인한 확장성의 한계가 존재하였고, 최근에는 블록의 병렬처리를 가능하게하는 **DAG(Directed Acyclic Graph)** 구조를 도입한 Bullshark, Mysticeti, Autobahn BFT 등으로 발전하고 있습니다.

이번 글에서는 이 중 현재 Sui 블록체인의 합의 알고리즘으로 사용되고 있는 Mysticeti에 대해 알아보도록 하겠습니다. 그 전에 Sui에서 사용자가 보낸 트랜잭션이 어떻게 처리되는지 간단하게 살펴보고 넘어가겠습니다.

![image.png](attachment:d775a34f-b471-45c9-8ee1-00b9c5bea620:image.png)

그림과 같이 사용자가 보낸 트랜잭션은 여타 블록체인과 마찬가지로 유효성 검증을 거친 후 이상이 없다면 합의로 넘어가게 됩니다. 다만 Sui는 Object-based 모델로, 소유객체만을 포함하고 있는 트랜잭션은 합의를 거치지 않고 fast path를 통해 실행됩니다. 엄밀히 말하면 합의를 거치지 않는 것은 아니고 블록 단위로 합의를 진행하는 동시에 트랜잭션 단위로 빠르게 처리될 수 있는 것인데 이 글을 끝까지 읽으면 무슨 말인지 알 수 있습니다. 앞으로 이런 트랜잭션을 **fast-path 트랜잭션**이라고 하겠습니다.

이 글에서는 먼저 Mysticeti의 중요한 3가지 특징을 알아보고, 기본적인 **블록간 합의**가 어떻게 이뤄지는지, **fast-path 트랜잭션**은 어떻게 더 빨리 커밋될 수 있는지에 대해 파헤쳐보겠습니다.

# Mysticeti 특징

Mysticeti 이전에도 Sui는 Bullshark라는 DAG 기반 BFT 합의 알고리즘을 사용하고 있었습니다. 그러나 Bullshark는 DAG 구조임에도 기존 블록체인처럼 **certified 방식**, 즉 명시적인 서명 수집을 통해 합의를 진행했으며, 이로 인해 **처리량(throughput)**은 증가했지만 **지연 시간(latency)**은 줄이기 어려웠습니다.

예컨대, 동시에 여러 블록이 한꺼번에 커밋되는 상황이 발생하거나, 실제 블록 실행보다 서명 수집 및 인증에 더 많은 리소스가 소모되는 비효율 문제가 있었습니다.

<aside>
💡

### 처리량(Throughput) vs. 지연시간(Latency)

---

지연 시간은 단일 트랜잭션 처리에 소요되는 시간을, 처리량은 단위 시간당 처리 가능한 트랜잭션 수를 의미

- **Throughput (처리량)**:
    
    시스템이 **단위 시간당 처리 가능한 트랜잭션 수** (보통 초당 트랜잭션 수, TPS)
    
- **Latency (지연 시간)**:
    
    트랜잭션이 제출된 후 블록체인에 **최종 확정**되기까지 걸리는 시간 (즉, finality 시간)
    

- DAG 기반 합의는 **병렬성**을 통해 throughput을 크게 높일 수 있어요.
    
    👉 동시에 여러 블록을 생성하고, 여러 트랜잭션을 병렬로 포함 가능 → TPS 상승
    
- 하지만 이로 인해 **"언제 커밋할 수 있는가?"를 결정하는 룰이 복잡**해지고,
    
    👉 여러 wave를 기다려야 하는 구조가 되어 **latency는 높아짐**
    
</aside>

이러한 문제를 해결하기 위해 Mysticeti는 다음과 같은 **3가지 핵심 설계 특징**을 채택하고 있습니다.

## 1. Uncertified DAG

Mysticeti는 기존의 명시적인 인증(certification) 과정을 제거하고, 대신 **지지(support)**와 **투표(vote)**를 통해 합의를 달성하는 **uncertified DAG 방식**을 도입했습니다.

기존 방식은 블록을 커밋하기 위해 서명을 수집하고 그 서명을 다시 인증하는 등 **복잡한 3단계 메시지 교환**이 필요했지만, Mysticeti는 단순히 블록이나 트랜잭션을 전파하는 것만으로도 합의에 참여할 수 있습니다.

검증자는 이상 없는 블록을 받으면, 별도 서명을 보내는 대신 자신의 블록에서 과거 블록을 지지(support)하거나 트랜잭션에 투표(vote)함으로써 **간접적이고 결정론적으로 인증**합니다.

<aside>
💡

### 지지(Support) vs. 투표(Vote)

---

- **지지(Support)**
블록 B′가 과거의 블록 B ≡ (A, r, h)를 "지지"한다고 할 때, 이는 B′에서 해시를 따라 **깊이 우선 탐색**했을 때 검증자 A가 라운드 r에서 생성한 최초의 블록이 B인 경우를 의미합니다.
    - 이는 비잔틴 노드가 여러 개의 블록을 만들었더라도, 모든 정직한 노드가 동일한 블록을 선택하게 하는 결정론적인(deterministic) 역할을 합니다.

- **투표(Vote)**
**과거 트랜잭션에 대한 찬반의 의사를 블록에 기록하는 것**을 의미합니다.
    - 이는 블록 공간을 차지하는 트랜잭션 종류의 한 가지이며, 암묵적 투표와 명시적 투표로 나뉩니다. 자세한 내용은 Fast-path 트랜잭션 커밋 방식에서 다루겠습니다.
</aside>

## 2. Independent Commit per Block

기존에는 한 명의 리더가 블록을 제안하고, 다른 검증자들이 그 블록을 서명으로 승인해야 커밋되는 방식이었습니다. 이는 리더가 병목이 되거나 실패할 경우 전체 합의가 지연되는 구조였습니다.

반면 Mysticeti에서는 모든 블록이 리더가 될 잠재력을 가지고 있으며, 리더로 선정되면 자신이 보는 DAG 뷰를 기반으로 독립적인 커밋 결정을 내릴 수 있습니다.

이로 인해 합의가 더 **분산되고 유연**해져, 지연 시간을 줄이고 장애에 강해집니다.

## 3. Single signature per block

또한 기존의 인증 방식은 블록 또는 트랜잭션을 커밋하기 위해 메시지를 서명해서 보내고, 이 메시지를 검증했다고 서명한 메시지를 받아서, 인증이 되었다고 서명한 블록 또는 트랜잭션을 보내야합니다. 이렇듯 적어도 3번 메시지를 주고받아야하기 때문에 어느 정도 지연시간이 필요합니다.

이로 인해 네트워크 트래픽과 지연 시간을 최소화하고, 전체 시스템 효율성을 크게 향상시킬 수 있습니다.

# Mysticeti 작동 방식

Mysticeti에서 모든 트랜잭션은 블록에 포함되어 처리됩니다. 합의가 필요 없는 트랜잭션도 합의가 필요한 트랜잭션과 마찬가지로 블록에 포함되지만, 특별히 후속 블록들로부터 개별 트랜잭션에 대한 투표를 받을 수 있도록 하여, 자신이 포함된 블록이 커밋되기 전이라도 충분한 투표를 받으면 트랜잭션이 먼저 커밋될 수 있도록 한 것입니다.

## 블록 커밋 방식

먼저 공통적으로 합의의 기반이 되는 블록의 커밋 방식에 대해 알아보겠습니다. 블록에는 to-commit, to-skip, undecided 3가지의 상태가 있습니다. Mysticeti는 커밋하기 위해 모든 블록을 to-commit, to-skip으로 표시하는 것이 목표입니다. 엄밀히 말하면 proposer slot이라는 논리적 블록 개념의 상태지만 슬롯은 블록과 같다고 생각하시면 됩니다.

상태를 표시하기 위해서는 다음과 같은 DAG 패턴을 알아야 합니다. 

- *Certificate Pattern*
- *Skip Pattern*

![[그림1] Certificate Pattern, Skip Pattern](attachment:7f78b490-2708-454c-ad7e-3576cd7af4db:Pasted_image_20250608144407.png)

[그림1] Certificate Pattern, Skip Pattern

먼저 *Certificate Pattern*([그림1]의 오른쪽)은 자식 라운드(r+1)의 `2f+1`개 이상의 블록이 r라운드의 블록($L_r$)을 부모 블록으로 지지하는 패턴입니다. Mysticeti에서는 이 경우 블록 $L_r$이 *certified* 되었다고 하며, 블록에 포함된 트랜잭션들이 Safety를 만족하여 실행할 수 있게 됩니다. 즉, 그림과 같이 N = 4일 때 3개 이상의 지지를 받게 되는 경우입니다. 참고로 꼭 r+2 라운드가 아니더라도 $L_r$을 지지했던 r+1 라운드의 블록들을 모두 자신의 경로에 포함하고 있는 블록([그림1 (b)] r+2 라운드의 녹색 블록)을 블록 $L_r$의 **certificate**라고 합니다.

반대로 *Skip Pattern*([그림1]의 왼쪽)은 자식 라운드(r+1)의 `2f+1`개 이상의 블록이 r라운드의 블록($L_r$)을 지지하지 않는 패턴입니다. 여기서 주의할 점은 부모 블록으로 받은 지지가 `2f+1`개에 미치지 못한 것이 아니라, `2f+1`개 이상의 블록이 부모 블록으로 지지하지 않을 때라는 것입니다. 즉, 이 경우 2개의 r+1 블록으로부터 지지를 받았을 경우에는 Skip Pattern이 아니고 0개 혹은 1개의 블록에게만 지지받았을 경우가 됩니다.

이제 이 패턴을 식별해서 상태를 결정하고 커밋해야 하는데 이를 위해 다음 3단계를 거칩니다. 

1. Direct decision rule
2. Indirect decision rule
3. Commit sequence

**Direct Decision Rule**

![[그림2] Direct decision rule: to-commit](attachment:4636e58d-0129-4010-aed2-b575fa4a0da1:image.png)

[그림2] Direct decision rule: to-commit

![[그림3] Direct decision rule: to-skip](attachment:83fb1540-318b-4915-9370-8cec14ba2d6a:image.png)

[그림3] Direct decision rule: to-skip

그림2와 같이 Certificate Pattern이 관찰되어 *certified*되면 **블록 $L_r$은 to-commit 상태로 결정되며, 그림3과 같이Skip Pattern이 관찰되면 to-skip 상태가 됩니다. 이것을 Direct decision rule이라 하며 대부분의 정상적인 상황에서 이것으로 충분합니다.

하지만 앞선 예시와 같이 2개의 r+1 라운드 블록으로부터 지지를 받은 경우 *Certificate Pattern*도 아니고 *Skip Pattern*도 아니게 됩니다. 이 경우 블록은 undecided 상태가 되며, 비잔틴 노드가 많거나 네트워크가 안 좋은 환경에서 많이 발생합니다. 이런 경우 undecided 상태를 to-commit, to-skip으로 만들기 위한 indirect decision rule이 있습니다.

**Indirect Decision Rule**

![[그림4] Indirect decision rule: anchor](attachment:1e1fcbdf-ddb9-4ed2-a2ef-61474a32f3d8:image.png)

[그림4] Indirect decision rule: anchor

r라운드의 블록 $L_r$이 undecided 상태라고 할 때, 3라운드 뒤인 r+3 라운드에서 처음으로 나오는 to-commit 또는 undecided 상태의 블록을 블록 $L_r$의 **앵커(anchor)**로 정합니다. 이 앵커의 상태에 따라 블록 $L_r$의 상태가 결정됩니다.

![[그림5] Indirect decision rule: to-commit](attachment:9225efb2-93dd-407c-9c98-cda19ffdd346:image.png)

[그림5] Indirect decision rule: to-commit

![[그림6] Indirect decision rule: to-skip](attachment:02fc1f12-9d2c-402f-921c-c9e38c4c0685:image.png)

[그림6] Indirect decision rule: to-skip

앵커가 undecided라면 블록 $L_r$도 여전히 undecided로 남게 됩니다. 하지만 앵커가 to-commit이라면 앵커가 블록 $L_r$의 certificate pattern을 참조하고 있다면 to-commit 상태가 되고, 그렇지 않다면 to-skip 상태가 됩니다.

**Commit Sequence**

마지막으로, 이렇게 상태 분류가 끝난 블록들을 순회하며 undecided 블록이 나올 때까지 to-commit 상태의 블록들을 커밋하고 to-skip 상태의 블록들은 건너뜁니다.

이후 유효한 블록이 수신될 때마다 이 과정을 반복해, 새로운 블록의 상태를 결정하고 이전의 undecided 블록의 상태를 결정할 수 있는지 확인합니다.

## Fast-path 트랜잭션 커밋 방식

앞서 fast-path 트랜잭션도 블록에 포함되어 합의가 진행되며, 자신이 포함된 블록이 커밋되기 전이라도 충분한 투표를 받으면 트랜잭션이 먼저 커밋하여 지연시간을 줄일 수 있다고 하였습니다.

투표는 암묵적 투표와 명시적 투표 2가지로 나뉩니다.

- 검증자가 트랜잭션을 수신한 후 **자신의 블록에 그것을 포함시키는 행위**는 암묵적 투표
- 명시적으로 특정 트랜잭션에 대한 투표 메시지를 넣는 경우는 명시적 투표

![[그림7] fast-path 트랜잭션 실행 ](attachment:8fb4d25f-b56e-498d-9526-02445877b8fc:image.png)

[그림7] fast-path 트랜잭션 실행 

그림7과 같이 암묵적 투표와 명시적 투표를 종합하여 2f+1이상의 검증자로부터 충분한 투표를 받고 certificate pattern이 관측되면 트랜잭션을 실행할 수 있습니다.

![[그림8] fast-path 트랜잭션 커밋 방법1](attachment:50804ee4-8564-48e1-a531-07d83a396ebf:image.png)

[그림8] fast-path 트랜잭션 커밋 방법1

![[그림9] fast-path 트랜잭션 커밋 방법2](attachment:12bbd89e-656e-40b8-98c4-4e7ffdf6f675:image.png)

[그림9] fast-path 트랜잭션 커밋 방법2

이렇게 실행된 트랜잭션은 그림8,9와 같이 둘 중 하나의 조건을 만족하게 되면 커밋됩니다.

(1) 2f + 1 개의 certificate pattern

(2) 1 개의 certificate라도 합의에 의해 **커밋된 블록의 인과적 히스토리에 포함**

[Platform Evolution: Mysticeti v2: Breaking the Surface for Faster Sui Consensus](https://www.notion.so/Platform-Evolution-Mysticeti-v2-Breaking-the-Surface-for-Faster-Sui-Consensus-1f741fda1ff080619e7fd18fdf3f65f8?pvs=21) 

# 참고문헌

- [Safety and Liveness](https://medium.com/dsrv/safety-and-liveness-2c0f0f87aead)
- [합의 알고리즘 이해하기 - PBFT Consensus Algorithm](https://steemit.com/consensus/@kblock/48-pbft-consensus-algorithm)
- [코스모스의 Tendermint — 블록체인 합의 알고리즘 분석(3)](https://medium.com/@organmo/%EB%B8%94%EB%A1%9D%EC%B2%B4%EC%9D%B8-%ED%95%A9%EC%9D%98-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EB%B6%84%EC%84%9D-%EC%BD%94%EC%8A%A4%EB%AA%A8%EC%8A%A4%EC%9D%98-tendermint-3-64f5c74b7e8c)
- [Aptos의 DiemBFT — 블록체인 합의알고리즘 분석 (4)](https://medium.com/@organmo/aptos%EC%9D%98-diembft-%EB%B8%94%EB%A1%9D%EC%B2%B4%EC%9D%B8-%ED%95%A9%EC%9D%98%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EB%B6%84%EC%84%9D-4-ec10ab7509f0)
- [What is the difference between PBFT, Tendermint, HotStuff, and HotStuff-2?](https://decentralizedthoughts.github.io/2023-04-01-hotstuff-2/)
