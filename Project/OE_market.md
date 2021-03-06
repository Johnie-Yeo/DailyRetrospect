# 오이마켓

> https://github.com/connect-foundation/2019-20/

## 프로젝트 전반적 개요

### Auth server

- github oauth & jwt
- mysql
- 유저 정보, 데이터 정합성 및 트랜잭션 고려

### Chat server

- socket io
- mongo

### Product server

- 상품 CRUD & 검색
- MongoDB & Elastic search 사용
- 상품에 대한 디테일 밖에 없으므로 정규화를 하지 않고 싶다

### Client server

- react build -> obj storage -> cdn
- 앞에 nginx를 통해 entrance 관리

## 나의 역할

### FrontEnd

#### 상품 조회

- 상품 이미지를 보여주기 위해 캐로셀 적용
	- 캐로셀은 nuka carousel 사용
	- vanilla로 구현한 경험이 있기 때문에 라이브러리 사용 시도
	- 내 입맛에 딱 맞는 라이브러리를 찾기 굉장히 힘들었고 빠른 시일 내에 React기반 나만의 라이브러리 제작 예정
- Geolocation API를 이용하여 현 위치로부터 판매자의 위치까지 거리 제공

#### 상품 등록

- 이미지 Resize
	- Canvas API를 이용하여 구현
	- 트래픽 부하를 줄이기 위한 mobile/desktop 버전을 각각 업로드
	- 이후 user agent 정보를 기반으로 적합한 크기의 이미지 제공
- 이미지 업로드
	- 사진을 선택함과 동시에 resize가 이뤄지고 이후 즉시 업로드 진행
	- 업로드 이후 CDN의 URI를 가져온다.
	- 미리 사진부터 업로드함으로써 submit에 소요되는 시간을 크게 줄임
	- 업로드된 이미지의 URI는 react의 context api와 local storage에서 항상 같이 유지함으로써 페이지를 나갔다가 다시 들어와도 살아있도록 보장
- History 관리
	- 현재 페이지를 히스토리로 저장하지 않음으로써 상품 등록 이후 다시 돌아오지 못하게 막았다.

#### 인증

- 인증되지 않거나 중복 인증을 막기위한 접근 제어

### BackEnd

#### 인증

- Github OAuth 적용
	- 회원관리 용이
- jwt를 이용한 인증으로 서버의 부담을 완화
	- 모든 api서버가 토큰 키를 공유함으로써 인증서버의 부담을 더 줄임
	- HTTP Only 옵션을 통한 XSS공격에 대응
	- CSRF 공격을 방지하기 위한 request header의 referer을 참조
- 에러 처리
	- 의미에 맞는 에러처리
- MySQL 사용
	- 정형화된 데이터 관리
	- 정합성 보장
	- 트랜잭션에 대한 고려

#### 이미지 업로드

- NCP Object Storage와 CDN을 연동하여 활용
	- 이미지 리소스 관리에 대한 부분을 클라우드에게 전가
	- multerS3 모듈 활용

#### 상품 CRUD

- MongoDB 사용
	- 상품을 관리하기 위한 테이블만이 필요
	- 관계가 생기지 않고 정규화를 하지 않는 편이 더 효율적
	- 속도에 더 유리
	- 검색을 위한 Elasticsearch와 더 잘 맞다.

#### 검색

- full text query
	- nori 형태소 분석기가 베이스
	- search analyzer은 standard를 사용하고 색인은 nori로 하였다.
		- search analyzer을 standard로 설정한 이유는 정확한 값을 찾고 싶었기 때문에
	- standard 검색을 위해 decompound_mode는 mixed로, 또 filter에 shingle을 사용함으로써 단어의 조합이더라도 검색이 가능하도록 하였다.
	- 검색은 문장이 올 수 있고 단어가 올 수 있으므로 match와 term에 대해 검색하되 bool.should로 묶어 처리하였다.
- term verctor api 활용

## 힘들었던 점

- 협업의 방향성
	- 처음에는 스프린트 범위를 좁게 잡아서 하나의 목표를 3명이 빠르게 해결하고자 시도
	- 각자의 작업에 의존성이 커지면서 다른사람이 작업을 끝내야만 내 작업을 시도할 수 있는 일이 생겨남
	- 현재 진행 상황에 대해 모두가 100% 이해 할 수 있는 장점은 있지만 작업속도가 굉장히 느렸다
	- 마스터님의 조언도 많이 얻었고 최종적으로는 각자 독립된 작업을 하되 코드리뷰를 통해 전체 진행을 파악하는 방향으로 진행했다.
	- 확실히 작업 속도는 눈에 띄게 빨라졌는데 우리가 처음에 원했던 진정한 협업에서는 조금 거리가 멀어진 느낌이기도 했다.
	- 현업에서 협업이 어떻게 이뤄질까에 대해 더 궁금해졌다.