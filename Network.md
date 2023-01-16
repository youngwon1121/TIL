



## TCP/IP  4계층

- Application
- Transport (TCP/UDP)
- Internet (IP)
- Network 



## Application 계층

port번호를 통해 해당 PC의 어떤 어플리케이션인지 찾아갈 수 있다.

HTTP, DNS

#### HTTP

- Stateless
  - 장점 : LB등의 장비로 서버 scaling 하기에 용이함.
  - 단점 : 보내야하는 데이터가 늘어남
- Connectionless
  - 장점 : 서버에서 유지해야하는 connection이 줄어 자원을 아낄 수 있음.
  - 단점 : 매 요청마다 TCP 3-way handshake를 새로해야함

## Transport 계층

#### TCP 

- 연결지향 - TCP 3 way handshake
- IP계층이 하지 못하는<b> 데이터 전달보증</b>
- IP계층이 하지 못하는 <b>패킷 순서보증</b>



#### UDP

​	TCP가 해주는것 보장X

## IP계층

Src IP, Dst IP를 통해 목적지를 찾아갈 수 있다.

- 패킷 소실 가능성
- 패킷 전달 순서 보장 X