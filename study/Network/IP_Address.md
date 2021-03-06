# IP Address

## IPv4

- 일반적으로 사용되는 IP 주소
- 32비트 길이 xxx.xxx.xxx.xxx
- .은 우리가 보기 쉽게 하기 위해 찍은 것 원래는 그냥 32비트 2진수
- 갯수는 총 43억개로 이제 거의 포화상태

## IPv6

- IPv4의 단점을 해결
- 128비트 길이의 주소
- 효율적인 네트워크 환경

## IPv4 vs IPv6

### 표기법
- IPv4
	- 32비트의 주소를 10진수로 표기
- IPv6
	- 128비트의 주소를 16진수로 표기

### 통신방식

- IPv4
	- 유니캐스트
	- 멀티캐스트
	- 브로드캐스트
- IPv6
	- 유니캐스트
	- 멀티캐스트
	- 애니캐스트

> 유니캐스트 : 1대1통신
> 멀티캐스트 : 1대 특정 다수 통신
> 브로드캐스트 : 1대 불특정 다수 통신
> 애니캐스트 : 1대 특정 다수 통신이지만 제일 가까운 목적지를 찾아 간다.

### 패킷 헤더

- IPv4
	- 가변적
	- 네트워크 통신의 모든 계층에서 오류 체크가 이루어진다.
- IPv6
	- 고정적
	- 플러그 앤 플레이도 지원된다.

## IP 주소의 이해

- IP 주소는 네트워크 부분과 호스트 부분으로 나눠진다.
	- 네트워크 부분은 교실, 호스트부분은 학생

## IP 주소는 언제 사용되는가?

### IP 주소

- 컴퓨터 네트워크에서 장치들이 서로를 인식하고 통신을 하기 위해서 사용하는 특수한 번호
- 네트워크에 연결된 장치가 라우터이든 일반 서버이든, 모든 기계는 이 특수한 번호를 가지고 있어야 한다. 

### 언제 사용되는가

- 이 번호를 이용하여 발신자를 대신하여 메시지가 전송되고 수신자를 향하여 예정된 목적지로 전달된다.

[IP에 대해서 조금 더 알아보기](https://grapherstory.tistory.com/82?category=713235)
[IP 주소](https://ko.wikipedia.org/wiki/IP_%EC%A3%BC%EC%86%8C)