# Blockchain

## Block

블록이란 **유효한 정보의 묶음**이다.

비트코인의 경우, 블록은 유효한 거래 정보의 묶음이며, 약 2300개의 거래 정보가 포함되어 있다.

블록은 **블록 헤더**, **정보**, **기타 정보**로 구성된다.

### Block Header

블록 헤더는 version, previous block hash, merkle hash, timestamp, bits, nonce로 구성된다.

1. version : 소프트웨어/프로토콜 버전
2. previous block hash : 블록 체인에서 이전 블록의 해쉬
3. merkle hash : 개별 정보의 해쉬를 2진 트리 형태로 구성할 때, 트리 루트의 해쉬
4. timestamp : 블록이 생성된 시간
5. bits : 블록 생성 시 난이도 조절 수치
6. nonce : 0부터 1씩 증가시켜 난이도를 만족하는 해쉬값을 찾기 위한 수치

### Block Hash

블록 해쉬는 **블록의 유효성을 검증**하는 값이다.
블록 헤더의 6 가지 정보를 입력 값으로 구해진다.
블록 전체 값의 해쉬가 아니라 **블록 헤더의 값만** 입력 값으로 사용한다.

## Blockchain

블록 체인은 블록이 이어져 만들어진 **블록의 집합체**이다.
previous block hash를 통해 블록들은 체인을 이루게된다.

## Proof of Work

작업 증명은 새로운 블록이 생성되어 블록 체인에 추가되기위한 작업이다.
새로운 블록이 블록 체인에 추가되기 위해서는 난이도 조절 수치인 bits를 만족시키는 nonce를 찾는 작업이 필요하다.
bits는 block header의 해쉬인 block hash의 값이 일정 수치 이하의 값을 가지도록하는 기준이되고, block header의 6가지 값 중 5가지는 고정된 값을 가지며 nonce만 0부터 1씩 증가시켜 이를 만족시키는 block hash를 찾는 작업을 진행하게된다.

## 참고자료 
  1. https://homoefficio.github.io/2017/11/19/%EB%B8%94%EB%A1%9D%EC%B2%B4%EC%9D%B8-%ED%95%9C-%EB%B2%88%EC%97%90-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0/
  2. https://www.youtube.com/watch?time_continue=2&v=bBC-nXj3Ng4
  3. https://www.blockchain.com/charts
