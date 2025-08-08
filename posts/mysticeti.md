---
title: 'Mysticeti: 수이의 합의 알고리즘'
date: '2025-07-25'
category: 'blockchain'
---

<figure>
  <img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti.png" alt="Mysticeti Overview">
  <figcaption>[그림1] Mysticeti 개요</figcaption>
</figure>

### 목차

- 개요
- Mysticeti의 특징
- Mysticeti의 작동 방식
- 요약

## 개요

블록체인과 같은 분산 시스템은 비잔틴 장군 문제에 대한 내성을 가져야 합니다. 이를 BFT(비잔틴 장애 허용)라 하며, 초기 퍼블릭 블록체인인 비트코인과 이더리움은 이를 **확률적으로** 달성합니다.

예를 들어 비트코인은 블록을 생성하는 데 막대한 연산 자원이 필요하도록 설계되어, 악의적인 공격자가 블록체인을 조작하려면 천문학적인 비용을 감수해야 합니다. 이더리움 또한 초기에는 비슷한 PoW(작업증명)를 사용했으며, 이후 PoS(지분증명) 체제로 전환하면서 공격 성공 시 금전적 손해를 입고, 재공격이 어렵도록 설계되어 있습니다.

그러나 이러한 확률적 BFT 방식은 일반 사용자에게도 **높은 수수료 부담**과 **거래 확정까지의 긴 대기 시간**이라는 단점을 안겨주었습니다. 이러한 한계로 인해 수이, 세이, 하이퍼리퀴드 같은 비교적 최신 블록체인들은 정족수(quorum) 기반으로 명확한 합의가 이루어지는 구조인 **결정적 BFT(Deterministic BFT)** 방식을 사용하고 있습니다.

결정적 BFT는 PBFT → Tendermint → HotStuff로 발전하며 **선형 블록체인 구조**에서 효율적인 합의를 추구해왔습니다. 그럼에도 불구하고 선형 구조로 인한 확장성의 한계가 존재하였고, 최근에는 블록의 병렬처리를 가능하게하는 **DAG(Directed Acyclic Graph)** 구조를 도입한 Bullshark, Mysticeti, Autobahn BFT 등으로 발전하고 있습니다.

이번 글에서는 이 중 현재 Sui 블록체인의 합의 알고리즘으로 사용되고 있는 Mysticeti에 대해 알아보도록 하겠습니다. 그 전에 Sui에서 사용자가 보낸 트랜잭션이 어떻게 처리되는지 간단하게 살펴보고 넘어가겠습니다.

![image](https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-1.png)

그림과 같이 사용자가 보낸 트랜잭션은 여타 블록체인과 마찬가지로 유효성 검증을 거친 후 이상이 없다면 합의로 넘어가게 됩니다. 다만 Sui는 Object-based 모델로, 소유객체만을 포함하고 있는 트랜잭션은 합의를 거치지 않고 fast path를 통해 실행됩니다. 엄밀히 말하면 합의를 거치지 않는 것은 아니고 블록 단위로 합의를 진행하는 동시에 트랜잭션 단위로 빠르게 처리될 수 있는 것인데 이 글을 끝까지 읽으면 무슨 말인지 알 수 있습니다. 앞으로 이런 트랜잭션을 **fast-path 트랜잭션**이라고 하겠습니다.

이 글에서는 먼저 Mysticeti의 중요한 3가지 특징을 알아보고, 기본적인 **블록간 합의**가 어떻게 이뤄지는지, **fast-path 트랜잭션**은 어떻게 더 빨리 커밋될 수 있는지에 대해 파헤쳐보겠습니다.

## Mysticeti 특징

Mysticeti 이전에도 Sui는 Bullshark라는 DAG 기반 BFT 합의 알고리즘을 사용하고 있었습니다. 그러나 Bullshark는 DAG 구조임에도 기존 블록체인처럼 **certified 방식**, 즉 명시적인 서명 수집을 통해 합의를 진행했으며, 이로 인해 **처리량(throughput)**은 증가했지만 **지연 시간(latency)**은 줄이기 어려웠습니다.

예컨대, 동시에 여러 블록이 한꺼번에 커밋되는 상황이 발생하거나, 실제 블록 실행보다 서명 수집 및 인증에 더 많은 리소스가 소모되는 비효율 문제가 있었습니다.

<aside>
💡처리량(Throughput) vs. 지연시간(Latency)

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

### 1. Uncertified DAG

Mysticeti는 기존의 명시적인 인증(certification) 과정을 제거하고, 대신 **지지(support)**와 **투표(vote)**를 통해 합의를 달성하는 **uncertified DAG 방식**을 도입했습니다.

기존 방식은 블록을 커밋하기 위해 서명을 수집하고 그 서명을 다시 인증하는 등 **복잡한 3단계 메시지 교환**이 필요했지만, Mysticeti는 단순히 블록이나 트랜잭션을 전파하는 것만으로도 합의에 참여할 수 있습니다.

검증자는 이상 없는 블록을 받으면, 별도 서명을 보내는 대신 자신의 블록에서 과거 블록을 지지(support)하거나 트랜잭션에 투표(vote)함으로써 **간접적이고 결정론적으로 인증**합니다.

<aside>
💡

#### 지지(Support) vs. 투표(Vote)

- **지지(Support)**
블록 B′가 과거의 블록 B ≡ (A, r, h)를 "지지"한다고 할 때, 이는 B′에서 해시를 따라 **깊이 우선 탐색**했을 때 검증자 A가 라운드 r에서 생성한 최초의 블록이 B인 경우를 의미합니다.
  - 이는 비잔틴 노드가 여러 개의 블록을 만들었더라도, 모든 정직한 노드가 동일한 블록을 선택하게 하는 결정론적인(deterministic) 역할을 합니다.

- **투표(Vote)**
**과거 트랜잭션에 대한 찬반의 의사를 블록에 기록하는 것**을 의미합니다.
  - 이는 블록 공간을 차지하는 트랜잭션 종류의 한 가지이며, 암묵적 투표와 명시적 투표로 나뉩니다. 자세한 내용은 Fast-path 트랜잭션 커밋 방식에서 다루겠습니다.

</aside>

### 2. Independent Commit per Block

기존에는 한 명의 리더가 블록을 제안하고, 다른 검증자들이 그 블록을 서명으로 승인해야 커밋되는 방식이었습니다. 이는 리더가 병목이 되거나 실패할 경우 전체 합의가 지연되는 구조였습니다.

반면 Mysticeti에서는 모든 블록이 리더가 될 잠재력을 가지고 있으며, 리더로 선정되면 자신이 보는 DAG 뷰를 기반으로 독립적인 커밋 결정을 내릴 수 있습니다.

이로 인해 합의가 더 **분산되고 유연**해져, 지연 시간을 줄이고 장애에 강해집니다.

### 3. Single signature per block

또한 기존의 인증 방식은 블록 또는 트랜잭션을 커밋하기 위해 메시지를 서명해서 보내고, 이 메시지를 검증했다고 서명한 메시지를 받아서, 인증이 되었다고 서명한 블록 또는 트랜잭션을 보내야합니다. 이렇듯 적어도 3번 메시지를 주고받아야하기 때문에 어느 정도 지연시간이 필요합니다.

이로 인해 네트워크 트래픽과 지연 시간을 최소화하고, 전체 시스템 효율성을 크게 향상시킬 수 있습니다.

## Mysticeti 작동 방식

Mysticeti에서 모든 트랜잭션은 블록에 포함되어 처리됩니다. 합의가 필요 없는 트랜잭션도 합의가 필요한 트랜잭션과 마찬가지로 블록에 포함되지만, 특별히 후속 블록들로부터 개별 트랜잭션에 대한 투표를 받을 수 있도록 하여, 자신이 포함된 블록이 커밋되기 전이라도 충분한 투표를 받으면 트랜잭션이 먼저 커밋될 수 있도록 한 것입니다.

### 블록 커밋 방식

먼저 공통적으로 합의의 기반이 되는 블록의 커밋 방식에 대해 알아보겠습니다. 블록에는 to-commit, to-skip, undecided 3가지의 상태가 있습니다. Mysticeti는 커밋하기 위해 모든 블록을 to-commit, to-skip으로 표시하는 것이 목표입니다. 엄밀히 말하면 proposer slot이라는 논리적 블록 개념의 상태지만 슬롯은 블록과 같다고 생각하시면 됩니다.

상태를 표시하기 위해서는 다음과 같은 DAG 패턴을 알아야 합니다.

- *Certificate Pattern*
- *Skip Pattern*

|![image](https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-2.png)|
|:--:|
|[그림1] Certificate Pattern, Skip Pattern|

먼저 *Certificate Pattern*([그림1]의 오른쪽)은 자식 라운드(r+1)의 `2f+1`개 이상의 블록이 r라운드의 블록($L_r$)을 부모 블록으로 지지하는 패턴입니다. Mysticeti에서는 이 경우 블록 $L_r$이 *certified* 되었다고 하며, 블록에 포함된 트랜잭션들이 Safety를 만족하여 실행할 수 있게 됩니다. 즉, 그림과 같이 N = 4일 때 3개 이상의 지지를 받게 되는 경우입니다. 참고로 꼭 r+2 라운드가 아니더라도 $L_r$을 지지했던 r+1 라운드의 블록들을 모두 자신의 경로에 포함하고 있는 블록([그림1 (b)] r+2 라운드의 녹색 블록)을 블록 $L_r$의 **certificate**라고 합니다.

반대로 *Skip Pattern*([그림1]의 왼쪽)은 자식 라운드(r+1)의 `2f+1`개 이상의 블록이 r라운드의 블록($L_r$)을 지지하지 않는 패턴입니다. 여기서 주의할 점은 부모 블록으로 받은 지지가 `2f+1`개에 미치지 못한 것이 아니라, `2f+1`개 이상의 블록이 부모 블록으로 지지하지 않을 때라는 것입니다. 즉, 이 경우 2개의 r+1 블록으로부터 지지를 받았을 경우에는 Skip Pattern이 아니고 0개 혹은 1개의 블록에게만 지지받았을 경우가 됩니다.

이제 이 패턴을 식별해서 상태를 결정하고 커밋해야 하는데 이를 위해 다음 3단계를 거칩니다.

1. Direct decision rule
2. Indirect decision rule
3. Commit sequence

**Direct Decision Rule**

<div class="image-row">
  <figure>
    <img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-3.png" alt="Direct decision rule: to-commit" />
    <figcaption>[그림2] Direct decision rule: to-commit</figcaption>
  </figure>
  <figure>
    <img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-4.png" alt="Direct decision rule: to-skip" />
    <figcaption>[그림3] Direct decision rule: to-skip</figcaption>
  </figure>
</div>

그림2와 같이 Certificate Pattern이 관찰되어 *certified*되면 **블록 $L_r$은 to-commit 상태로 결정되며, 그림3과 같이Skip Pattern이 관찰되면 to-skip 상태가 됩니다. 이것을 Direct decision rule이라 하며 대부분의 정상적인 상황에서 이것으로 충분합니다.

하지만 앞선 예시와 같이 2개의 r+1 라운드 블록으로부터 지지를 받은 경우 *Certificate Pattern*도 아니고 *Skip Pattern*도 아니게 됩니다. 이 경우 블록은 undecided 상태가 되며, 비잔틴 노드가 많거나 네트워크가 안 좋은 환경에서 많이 발생합니다. 이런 경우 undecided 상태를 to-commit, to-skip으로 만들기 위한 indirect decision rule이 있습니다.

**Indirect Decision Rule**

<figure>
  <img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-5.png" alt="Indirect decision rule: anchor">
  <figcaption>[그림4] Indirect decision rule: anchor</figcaption>
</figure>

r라운드의 블록 $L_r$이 undecided 상태라고 할 때, 3라운드 뒤인 r+3 라운드에서 처음으로 나오는 to-commit 또는 undecided 상태의 블록을 블록 $L_r$의 **앵커(anchor)**로 정합니다. 이 앵커의 상태에 따라 블록 $L_r$의 상태가 결정됩니다.

<table>
<tr>
<td><img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-6.png" alt="Indirect decision rule: to-commit" /></td>
<td><img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-7.png" alt="Indirect decision rule: to-skip" /></td>
</tr>
<tr>
<td>[그림5] Indirect decision rule: to-commit</td>
<td>[그림6] Indirect decision rule: to-skip</td>
</tr>
</table>

앵커가 undecided라면 블록 $L_r$도 여전히 undecided로 남게 됩니다. 하지만 앵커가 to-commit이라면 앵커가 블록 $L_r$의 certificate pattern을 참조하고 있다면 to-commit 상태가 되고, 그렇지 않다면 to-skip 상태가 됩니다.

**Commit Sequence**

마지막으로, 이렇게 상태 분류가 끝난 블록들을 순회하며 undecided 블록이 나올 때까지 to-commit 상태의 블록들을 커밋하고 to-skip 상태의 블록들은 건너뜁니다.

이후 유효한 블록이 수신될 때마다 이 과정을 반복해, 새로운 블록의 상태를 결정하고 이전의 undecided 블록의 상태를 결정할 수 있는지 확인합니다.

### Fast-path 트랜잭션 커밋 방식

앞서 fast-path 트랜잭션도 블록에 포함되어 합의가 진행되며, 자신이 포함된 블록이 커밋되기 전이라도 충분한 투표를 받으면 트랜잭션이 먼저 커밋하여 지연시간을 줄일 수 있다고 하였습니다.

투표는 암묵적 투표와 명시적 투표 2가지로 나뉩니다.

- 검증자가 트랜잭션을 수신한 후 **자신의 블록에 그것을 포함시키는 행위**는 암묵적 투표
- 명시적으로 특정 트랜잭션에 대한 투표 메시지를 넣는 경우는 명시적 투표

![image](https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-8.png)
[그림7] fast-path 트랜잭션 실행

그림7과 같이 암묵적 투표와 명시적 투표를 종합하여 2f+1이상의 검증자로부터 충분한 투표를 받고 certificate pattern이 관측되면 트랜잭션을 실행할 수 있습니다.

<table>
<tr>
<td><img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-9.png" alt="fast-path 트랜잭션 커밋 방법1" /></td>
<td><img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-10.png" alt="fast-path 트랜잭션 커밋 방법2" /></td>
</tr>
<tr>
<td>[그림8] fast-path 트랜잭션 커밋 방법1</td>
<td>[그림9] fast-path 트랜잭션 커밋 방법2</td>
</tr>
</table>

이렇게 실행된 트랜잭션은 그림8,9와 같이 둘 중 하나의 조건을 만족하게 되면 커밋됩니다.

(1) 2f + 1 개의 certificate pattern

(2) 1 개의 certificate라도 합의에 의해 **커밋된 블록의 인과적 히스토리에 포함**

[Platform Evolution: Mysticeti v2: Breaking the Surface for Faster Sui Consensus](https://www.notion.so/Platform-Evolution-Mysticeti-v2-Breaking-the-Surface-for-Faster-Sui-Consensus-1f741fda1ff080619e7fd18fdf3f65f8?pvs=21)

## 핵심 정리

- Uncertified DAG

## 참고문헌

- [Safety and Liveness](https://medium.com/dsrv/safety-and-liveness-2c0f0f87aead)
- [합의 알고리즘 이해하기 - PBFT Consensus Algorithm](https://steemit.com/consensus/@kblock/48-pbft-consensus-algorithm)
- [코스모스의 Tendermint — 블록체인 합의 알고리즘 분석(3)](https://medium.com/@organmo/%EB%B8%94%EB%A1%9D%EC%B2%B4%EC%9D%B8-%ED%95%A9%EC%9D%98-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EB%B6%84%EC%84%9D-%EC%BD%94%EC%8A%A4%EB%AA%A8%EC%8A%A4%EC%9D%98-tendermint-3-64f5c74b7e8c)
- [Aptos의 DiemBFT — 블록체인 합의알고리즘 분석 (4)](https://medium.com/@organmo/aptos%EC%9D%98-diembft-%EB%B8%94%EB%A1%9D%EC%B2%B4%EC%9D%B8-%ED%95%A9%EC%9D%98%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EB%B6%84%EC%84%9D-4-ec10ab7509f0)
- [What is the difference between PBFT, Tendermint, HotStuff, and HotStuff-2?](https://decentralizedthoughts.github.io/2023-04-01-hotstuff-2/)
