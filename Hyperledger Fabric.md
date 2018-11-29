# Hyperledger Fabric

## Transaction

![](https://cdn-images-1.medium.com/max/1600/0*Blz6yYjORY-E1LVB.png)

### 1. Client A가 트랜잭션 요청(Transaction Proposal)

Client A가 블록체인 네트워크에 트랜잭션을 요청한다.
txPayload에 내용이 들어가고 clientSig에 서명이 들어가 신원 정보를 확인한다.
트랜잭션 요청(Transaction proposal)은 클라이언트의 SDK에서 gRPC 프로토콜로 전송할 수 있는 프로토콜 버퍼 형태로 변형한다.
클라이언트가 이벤트를 발생시키면, 피어들은 리슨하고 있다가 그 다음 액션을 취한다.
이벤트는 비즈니스 네트워크 모델 파일(.cto)에 사전에 정의되어야 하며, 특정된 트랜잭션만 발생할 수 있다.

```
// .cto  비즈니스 모델 파일
event Basic Event {}

// Event publisher
async function basicEventTransaction(basicEventTransaction) {
  let factory = getFactory();
  
  let basicEvent = factory.newEvent('org.namespace', 'BasicEvent');
  emit(basicEvent);
}

// Event Subscriber
businessNetworkConnection.on('event', (event) => {
  // event: { "$class": "org.namespace.BasicEvent", "eventId": "0000-0000-0000-00000#0" }
  console.log(event);
}
```

### 2. Endorsing Peer가 Client A의 서명을 확인하고 트랜잭션 실행

Endorsing peer가 트랜잭션을 전달받으면 다음을 확인한다.
  1. 트랜잭션이 형식에 맞게 내용이 잘 채워져있는지
  2. 이전에 제출된 적이 있었던 트랜잭션은 아닌지
  3. 서명이 유효한지(MSP를 통함)
  4. 트랜잭션을 제출한 클라이언트가 권한이 있는지
  
모두 확인될 경우 Endorsing peer는 Transaction proposal을 인자로 받아 체인코드를 실행한다. (Simulate/Execute tx)
State DB의 값에 체인코드가 실제로 실행되며 결과값, read/write set을 반환한다.
**이 시점에는 원장에 업데이트하지 않고, 반환된 결과값을 Endorsing peer의 서명을 포함한 Proposal response로 클라이언트 SDK로 전송한다.**

### 2.5 Client A가 Proposal responses 검토

클라이언트 SDK는 Proposal response의 Endorsing peer의 서명을 확인한 뒤 각 피어로부터의 Proposal resopnse를 비교한다.
단순한 쿼리 같이 Ordering service가 필요 없는 경우에는 쿼리의 결과 값을 얻고 프로세스를 종료한다. (READ)
원장 업데이트를 해야할 경우, 클라이언트 차원에서 Endorsement policy에 전하는 Proposal response가 왔는지 검토한다.
이 단계에서 클라이언트가 endorsement policy를 검토하지 않는다해도, committing 단계에서 각 피어가 별도로 검토한다.

### 3 Client A가 Transaction 전달

검토가 완료되고 나면 클라이언트가 Transaction message에 proposal과 proposal response를 담아서 Ordering service에 보낸다.
트랜잭션에는 read/write set이 들어가 있고, Endorsing peer의 서명과 채널 ID가 포함된다.
Orderer는 트랜잭션 내용에 관계 없이, 모든 채널에서 발생하는 트랜잭션을 받아서 시간 순서대로 정렬하여 블럭을 생성한다.
Orderer가 합의하는 방식은 기본적으로 pluggable하나, SOLO와 Kafka 두 가지를 제공한다.

### 4. Committing Peer에서 Transaction 검증 및 커밋

Orderer가 생선한 트랙잭션 블럭이 해당 채널의 모든 Committing peer에게 전달된다.
각각의 Committing peer는 블럭 내의 모든 트랜잭션이 각각의 Endorsement policy를 준수하는지, world state 값의 read-set 버전이 맞는지 확인한다.
검사 과정이 끝나면 해당 블럭 내의 트랜잭션에는 valid/invalid 값이 태그된다.

### 5. Ledger Update

각 피어는 최종 검증을 마친 블럭을 채널 내 체인에 붙인다.
그리고 유효한 트랜잭션의 경우 write-set이 state DB에 입력된다.
일련의 과정이 마무리되면 이벤트를 발생시켜 클라이언트에게 작업 결과를 알린다.

## Network

### 네트워크 생성

![](https://cdn-images-1.medium.com/max/1600/0*5wRfq8qqgOs1Gw1K.png)

Ordering Service(O), Network Policy(NP), Client Application(CA)로 구성된다.
그리고 그 Application을 소유하는 네트워크 참가 조직(RD)가 필요하다.
Ordering Service가 채널에 대한 정책과 멤버쉽 정책을 소유하고, 이를 기반으로 관리자 역할을 한다.

### 컨소시움 정의

![](https://cdn-images-1.medium.com/max/1600/0*6-TS3Wj0MX4FDcFc.png)

CA1, CA2 조직이 추가되고, RA, RB 조직이 추가되었다. 이들은 X509 root certificate에 따라 서로를 확인한다.

### 채널 생성

![](https://cdn-images-1.medium.com/max/1600/0*PbcWrHSB6VotAR4f.png)

Ordering Service 내에서 Config block을 생성하는 것으로 채널을 구현된다.
채널에 가입하고자 하는 조직은 이 채널의 config에 따라 추가 인증을 거쳐야 한다.
위 다이어그램에서 채널 C1이 추가되고 이에 대한 Channel Policy(CP)가 더해지고, 참여조직 RA, RB가 추가되었다.

### Peer

![](https://cdn-images-1.medium.com/max/1600/0*EcQlzDnzjjfvqFSz.png)

위 다이어그램에서 C1의 Peer(P1)이 추가되었고, P1은 Ledger(L1)을 가지고있다.
Peer의 역할은 다음과 같고, 하나의 Peer가 여러 역할을 가질 수 있다.

- Endorsing Peer : 스마트 컨트랙트에 수행될 트랜잭션을 시뮬레이션해보고 그 결과를 클라이언트 어플리케이션에 리턴해주는 피어이다.
                   트랜잭션을 검증하는 역할이다.
- Committing Peer : 오더링까지 되어서 전달된 트랜잭션 블럭을 검증하고, 검증이 완료되면 자기가 갖고있는 원장에 해당 블럭을 추가하고 관리한다.
                    모든 피어는 기본적으로 Committing Peer이다.
- Anchor Peer : 채널 내의 대표 피어이다. 채널에 다른 조직이 가입하면 가장 먼저 발견하는 피어이다.
- Leading Peer : 여러 피어를 가지고 있는 조직의 경우, 모든 피어를 대표하여 네트워크와 커뮤니케이션 하는 피어이다.

### Application, Smart Contract

![](https://cdn-images-1.medium.com/max/1600/0*Ht7rU0I8z3DQhqr0.png)

Smart Contract(Chaincode)가 Peer에 반드시 설치되고 실행되어야만 클라이언트 어플리케이션이 정상 작동하고, Transaction Proposal이 들어올 수 있다.
위 다이어그램에서 S4가 추가되었다.
Transaction Proposal이 클라이언트 어플리케이션에서 발생하면 스마트 컨트랙트가 endorsing peer에 시뮬레이션을 돌리고
그 결과(endorsement)를 다시 어플리케이션에 리턴한다.
그 결과가 endorsement policy에 부합하면 그 결과를 Ordering service(O1)에 보내서 committing peer에 보낸다.

### 네트워크 완성

![](https://cdn-images-1.medium.com/max/1600/0*sMFzaEYWzeFmc07P.png)

위 다이어그램은 네트워크가 확장된 모습이다.
이 네트워크에는 하나의 Ordering Service(O)가 있다.
네트워크를 정의하는 NP가 있고 RD라는 조직이 Ordering Service를 하고있다.
조직은 총 RA, RB, RC, RD 네 개의 조직이 참여하고 있고, 각각 CA1, CA2, CA3, CA4라는 인증 정책을 가지고 있다.
채널은 C1, C2 두 개가 있고 각각 CP1, CP2 라는 정책을 가지고 있다.
C1에는 RA, RB가 참여하고 있고, C2에는 RB, RC가 참여하고있다.
피어는 P1, P2, P3 세 개가 있고, 각각은 RA, RB, RC 조직에 속한다.
RD는 Ordering Serivce를 하고 있고 피어는 따로 가지고 있지 않다.
각각의 피어는 속한 채널의 Ledger와 Smart Contract를 갖고있다.
P2의 경우 두 채널에 동시에 속하고 있어 두 개의 Ledger와 Smart Contract를 동시에 들고있다.

Reference

* https://developer.ibm.com/kr/cloud/blockchain/blockchain-special-series/2018/11/11/hyperledger-fabric-architecture-3-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EA%B0%80-%EB%A7%8C%EB%93%9C%EB%8A%94-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/
* https://developer.ibm.com/kr/cloud/blockchain/blockchain-special-series/2018/11/11/hyperledger-fabric-architecture-4-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98/
* https://www.youtube.com/watch?v=smytS8dQCtk
