---

title: 'Mysticeti: 수이의 합의 알고리즘'

date: '2025-09-05'

category: 'blockchain'

---

![image](https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti.png)

[그림1] Mysticeti 개요

### 목차

- 개요
- Mysticeti의 특징
- Mysticeti의 작동 방식
- Mysticeti V2에서의 변화
- 마무리

## 개요

최근 두바이에서 열린 Sui Basecamp 2025에서 미스틴 랩스(Mysten Labs)는 합의 알고리즘 **Mysticeti의 두 번째 버전(V2)** 을 공식 발표했다. Mysticeti V2는 Mysticeti V1으로 전환할 때와 같은 단발적 교체 작업이 아니라, 이미 안정적으로 운영 중인 Mysticeti를 기반으로 점진적인 최적화 시리즈를 도입하는 형태다.

특히 주목할 점은, Narwhal & Tusk, Bullshark, 그리고 Mysticeti V1까지 이어지는 짧은 시간 동안의 급격한 혁신적 개선 이후, Mysticeti V2는 V1 공개 약 1년 반 만에 등장했다는 것이다. 이는 곧 Mysticeti V1이 충분히 성숙하고 안정적인 합의 프로토콜로 자리 잡았음을 보여준다. 즉, 더 이상 프로토콜 전체를 갈아엎는 “파괴적 혁신(breaking change)”이 아닌, 세밀한 성능·효율성 개선에 집중하는 단계로 넘어왔다는 방증이다.

이러한 맥락에서 Mysticeti V2의 발표는 Mysticeti가 이미 **실제 프로덕션 환경에서 검증된 고성능 BFT 합의 알고리즘**임을 드러내며, 앞으로의 개선이 안정성과 최적화에 초점을 맞출 것임을 시사한다.

이번 글에서는 Mysticeti의 특징과 작동 방식에 대해 자세히 살펴보고, V2에서 발표된 변화와 앞으로 기대할 수 있는 최적화 방향까지 정리해본다.

![image](https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-1.png)

## Mysticeti 특징

Mysticeti 이전에도 Sui는 Bullshark라는 DAG 기반 BFT 합의 알고리즘을 사용하고 있었다. 그러나 Bullshark는 DAG 구조임에도 불구하고 기존 선형 블록체인과 유사한 **certified(명시적 인증) 방식**을 채택했다. 즉, 블록을 확정하기 위해 검증자 서명을 수집하고 이를 다시 인증하는 절차를 거쳤다.

이 방식은 **처리량(throughput)** 측면에서는 성과가 있었지만, **지연 시간(latency)** 을 낮추는 데에는 한계가 있었다. 예컨대, 여러 블록이 동시에 한꺼번에 커밋되는 상황이 발생하거나, 실제 트랜잭션 실행보다 서명 수집과 인증 과정에서 더 많은 리소스가 소모되는 비효율이 있었다.

이러한 문제를 해결하기 위해 Mysticeti는 다음과 같은 **세 가지 핵심 설계 특징**을 도입했다.

### 1. Uncertified DAG

Mysticeti는 기존의 명시적 인증(certification) 절차를 제거하고, 대신 **지지(support)** 와 **투표(vote)** 라는 개념을 활용하는 **uncertified DAG** 방식을 채택했다.

기존 방식에서는 블록을 커밋하기 위해 서명 → 서명 검증 → 최종 인증이라는 복잡한 3단계 메시지 교환이 필요했다. 반면 Mysticeti에서는 단순히 블록을 전파하고 DAG에 연결하는 것만으로 합의에 참여할 수 있다.

검증자는 문제가 없는 블록을 받으면 별도의 서명을 보내지 않고, 자신의 블록 안에서 과거 블록을 지지(support)하거나 특정 트랜잭션에 투표(vote)함으로써 간접적으로, 그러나 결정론적으로 인증을 수행한다.

#### 지지(Support) vs. 투표(Vote)

- **지지(Support)**
	블록 B′가 과거의 블록 B ≡ (A, r, h)를 "지지"한다고 할 때, 이는 B′에서 해시를 따라 **깊이 우선 탐색**했을 때 검증자 A가 라운드 r에서 생성한 최초의 블록이 B인 경우를 의미한다.
	
	이는 비잔틴 노드가 여러 개의 블록을 만들었더라도, 모든 정직한 노드가 동일한 블록을 선택하게 하는 결정론적인(deterministic) 역할을 한다.

- **투표(Vote)**
	과거 트랜잭션에 대한 찬반의 의사를 블록에 기록하는 것을 의미한다.
	
	이는 블록 공간을 차지하는 트랜잭션 종류의 한 가지이며, 암묵적 투표와 명시적 투표로 나뉜다. 자세한 내용은 Fast-path 트랜잭션 커밋 방식에서 다루자.

### 2. Independent Commit per Block

Bullshark에서는 리더가 제안한 블록을 다른 검증자들이 승인해야만 커밋이 가능했는데, 이는 리더가 병목이 되거나 실패할 경우 전체 합의가 지연되는 구조였다.

Mysticeti에서는 모든 블록이 잠재적으로 리더 역할을 수행할 수 있으며, 리더가 되면 자신이 보는 DAG 뷰에 따라 독립적인 커밋 결정을 내릴 수 있다.

그 결과 합의가 더 분산적이고 유연해지며, 특정 노드 장애에도 지연 없이 합의를 이어갈 수 있어 지연 시간 단축과 장애 내성(fault tolerance)이 크게 향상된다.

### 3. Single Signature per Block

기존 인증 방식에서는 블록을 커밋하기 위해 적어도 세 차례의 서명(블록 서명 → 검증 서명 → 최종 인증)이 필요했다. 이는 곧 네트워크 트래픽 증가와 지연 시간 확대를 불러왔다.

Mysticeti는 각 블록에 대해 **한 번의 서명만 요구**함으로써, 네트워크 메시지를 대폭 줄이고 시스템 효율을 크게 개선했다. 이 단순화 덕분에 합의 속도는 더 빨라지고, 처리량도 안정적으로 유지된다.


## Mysticeti의 작동 방식

Mysticeti에서 모든 트랜잭션은 블록에 포함되어 처리된다. 여기에는 **합의가 꼭 필요하지 않은 트랜잭션**도 포함되는데, Mysticeti는 이런 트랜잭션이 뒤에 오는 블록들로부터 **직접 투표**를 받아, 자신이 속한 블록이 확정되기 전이라도 먼저 커밋될 수 있도록 설계되어 있다. 덕분에 전체 지연 시간을 크게 줄일 수 있다.

### 블록 커밋 방식

먼저, 합의의 기본 단위인 **블록이 어떻게 커밋되는지**부터 살펴보자.

Mysticeti는 블록(정확히는 proposer slot)을 다음 세 가지 상태 중 하나로 표시한다.

- **to-commit**: 확정(commit) 상태
- **to-skip**: 건너뛰기(skip) 상태
- **undecided**: 아직 미정 상태

목표는 결국 모든 블록을 `to-commit` 혹은 `to-skip`으로 분류하는 것이다.

이를 위해 Mysticeti는 DAG 안에서 두 가지 핵심 패턴을 식별한다.

![image](https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-2.png)

[그림1] Certificate Pattern, Skip Pattern


1. **Direct Decision Rule**

가장 먼저 적용되는 규칙은 **직접 결정 규칙(Direct Decision Rule)** 이다. 합의 과정에서 어떤 블록이 특정한 패턴을 만족하는지를 확인하는데, 만약 그 블록이 Certificate Pattern을 보이면 곧바로 to-commit 상태가 된다. 반대로 Skip Pattern을 보이면 to-skip 상태가 된다. 전체 합의의 대부분은 이 단계에서 빠르게 결정되므로, 정상적인 상황에서는 블록이 즉시 commit 혹은 skip 상태로 정리된다.

<div style="display: flex; justify-content: space-around; align-items: flex-start;">
  <div style="flex: 1; text-align: center; margin: 0 10px;">
    <img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-3.png" alt="Direct decision rule: to-commit" style="max-width: 100%; height: auto;">
    <p>[그림2] Direct decision rule: to-commit</p>
  </div>
  <div style="flex: 1; text-align: center; margin: 0 10px;">
    <img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-4.png" alt="Direct decision rule: to-skip" style="max-width: 100%; height: auto;">
    <p>[그림3] Direct decision rule: to-skip</p>
  </div>
</div>

2. **Indirect Decision Rule**

모든 블록이 Direct Rule에서 명확히 결정되는 것은 아니다. 어떤 블록은 여전히 undecided 상태로 남는데, 이때 적용되는 것이 **간접 결정 규칙(Indirect Decision Rule)** 이다. 규칙은 단순하다. undecided 블록이 있으면 3라운드 뒤에 처음 등장하는 블록을 기준점, 즉 앵커(anchor)로 삼는다. 앵커가 to-commit 상태라면, undecided 블록도 조건에 따라 함께 to-commit 되거나 to-skip 으로 정리된다. 반대로 앵커가 여전히 undecided 상태라면 기존 undecided 블록도 그대로 유지된다. 이렇게 해서 Direct Rule에서 남은 블록들도 점차 결정된다.

![image](https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-5.png)
[그림4] Indirect decision rule: anchor

r라운드의 블록 $L_r$이 undecided 상태라고 할 때, 3라운드 뒤인 r+3 라운드에서 처음으로 나오는 to-commit 또는 undecided 상태의 블록을 블록 $L_r$의 앵커(anchor)로 정한다. 이 앵커의 상태에 따라 블록 $L_r$의 상태가 결정된다.

<div style="display: flex; justify-content: space-around; align-items: flex-start;">
  <div style="flex: 1; text-align: center; margin: 0 10px;">
    <img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-6.png" alt="Indirect decision rule: to-commit" style="max-width: 100%; height: auto;">
    <p>[그림5] Indirect decision rule: to-commit</p>
  </div>
  <div style="flex: 1; text-align: center; margin: 0 10px;">
    <img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-7.png" alt="Indirect decision rule: to-skip" style="max-width: 100%; height: auto;">
    <p>[그림6] Indirect decision rule: to-skip</p>
  </div>
</div>

앵커가 undecided라면 블록 $L_r$도 여전히 undecided로 남게 된다. 하지만 앵커가 to-commit이라면 앵커가 블록 $L_r$의 certificate pattern을 참조하고 있다면 to-commit 상태가 되고, 그렇지 않다면 to-skip 상태가 된다.

3. **Commit Sequence**

마지막 단계는 **커밋 시퀀스(Commit Sequence)** 이다. 이 단계에서는 블록들을 순서대로 보면서 _to-commit_ 상태는 실제로 커밋하고, to-skip 상태는 건너뛴다. 만약 undecided 블록이 나오면 거기서 멈추고, 새로운 블록이 도착했을 때 다시 확인한다. 이렇게 순차적으로 commit과 skip을 처리하면서 DAG의 블록들이 일관된 순서대로 정리된다.

### Fast-path 트랜잭션 커밋 방식

앞서 fast-path 트랜잭션도 블록에 포함되어 합의가 진행되며, 자신이 포함된 블록이 커밋되기 전이라도 충분한 투표를 받으면 트랜잭션이 먼저 커밋하여 지연시간을 줄일 수 있다고 하였습니다.

투표는 암묵적 투표와 명시적 투표 2가지로 나뉩니다.

- 검증자가 트랜잭션을 수신한 후 **자신의 블록에 그것을 포함시키는 행위**는 암묵적 투표
- 명시적으로 특정 트랜잭션에 대한 투표 메시지를 넣는 경우는 명시적 투표

![image](https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-8.png)
[그림7] fast-path 트랜잭션 실행

그림7과 같이 암묵적 투표와 명시적 투표를 종합하여 2f+1이상의 검증자로부터 충분한 투표를 받고 certificate pattern이 관측되면 트랜잭션이 실행할 수 있습니다.

<div style="display: flex; justify-content: space-around; align-items: flex-start;">
  <div style="flex: 1; text-align: center; margin: 0 10px;">
    <img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-9.png" alt="fast-path 트랜잭션 커밋 방법1" style="max-width: 100%; height: auto;">
    <p>[그림8] fast-path 트랜잭션 커밋 방법1</p>
  </div>
  <div style="flex: 1; text-align: center; margin: 0 10px;">
    <img src="https://raw.githubusercontent.com/tain030/blog-post/main/images/mysticeti-10.png" alt="fast-path 트랜잭션 커밋 방법2" style="max-width: 100%; height: auto;">
    <p>[그림9] fast-path 트랜잭션 커밋 방법2</p>
  </div>
</div>

이렇게 실행된 트랜잭션은 그림8,9와 같이 둘 중 하나의 조건을 만족하게 되면 커밋됩니다.

(1) 2f + 1 개의 certificate pattern

(2) 1 개의 certificate라도 합의에 의해 커밋된 블록의 인과적 히스토리에 포함

### Fast-path 트랜잭션 커밋

Mysticeti의 재미있는 특징은 트랜잭션이 블록보다 먼저 커밋될 수 있다는 점이다.

앞서 말했듯이 모든 트랜잭션은 블록에 포함되지만, 후속 블록들로부터 충분한 투표를 받으면 블록이 확정되기 전에 트랜잭션이 먼저 커밋된다. 이를 **Fast-path 커밋**이라고 한다.

투표 방식은 두 가지다.

- **암묵적 투표(Implicit Vote)**: 검증자가 트랜잭션을 자기 블록에 포함시키는 것 자체가 투표가 된다.
    
- **명시적 투표(Explicit Vote)**: 특정 트랜잭션을 빠르게 확정하기 위해 별도의 투표 메시지를 블록에 기록한다.
    

트랜잭션은 2f+1 이상의 검증자로부터 투표를 받고, 그 결과 Certificate Pattern이 관찰되면 실행할 수 있게 된다.

이때 커밋 조건은 두 가지가 있다. 첫째, 트랜잭션이 포함된 블록이 2f+1개의 certificate pattern을 확보하면 커밋된다. 둘째, 단 하나의 certificate만 있더라도, 그 certificate가 이미 커밋된 블록의 인과적 히스토리에 포함되어 있으면 커밋된다.

이 과정을 통해 Mysticeti는 블록 확정과 별도로 트랜잭션 단위에서 초저지연 커밋을 실현한다.


## 참고문헌

- [Safety and Liveness](https://medium.com/dsrv/safety-and-liveness-2c0f0f87aead)
- [MYSTICETI: Reaching the Latency Limits with Uncertified DAGs](https://arxiv.org/pdf/2310.14821)
- [Mysticeti v2: Breaking the Surface for Faster Sui Consensus](https://youtu.be/uS6rbdt_064?list=TLGGhzkrg_QVqu0wNDA5MjAyNQ)
