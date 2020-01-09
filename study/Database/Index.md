# Index

- 특정 컬럼의 조회를 위해 index가 없다면 DB는 테이블 전체를 스캔해야 하는 문제가 있다.

> #### Full table scan
> - row의 값을 순차적으로 scan하며 값을 비교한다.
> - Full table scan은 가장 느린 scanning방법이며 많은 자료가 담긴 Disk를 읽기 위한 I/O를 사용하며 자원을 잡아먹는다.
> - 결국엔 속도도 느리고 자원도 많이 사용하는 좋지 않은 방법이다.
> - 이러한 문제를 해결하기 위해 Database에서 Index기능을 제공한다.

## DB Index

- 읽기 성능을 향상시키기 위한 일종의 자료구조
- Index는 관련된 Table과 별도로 저장되며 Index로 설정한 컬럼값이 변경되거나 추가되면 Index도 동시에 업데이트
- 주로 B Tree 혹은 B+Tree 알고리즘으로 이루어져 있다.
- Primary Key에는 Database가 자동으로 Index기능을 설정하여 관리한다.

## Index 유형

### Cluster Index

> row가 디스크에 물리적으로 index order와 같은 순서로 저장된다.
> 그러므로 단 하나의 클러스터 인덱스만이 존재

- 해당 키 값을 기반으로 테이블이나 뷰의 데이터 행을 정렬하고 저장
- 인덱스 정의에 여러 열이 포함된다. 
- 데이터 행 자체는 한 가지 순서로만 저장될 수 있으므로 테이블당 클러스터형 인덱스는 하나만 있을 수 있습니다.
- 테이블의 데이터 행이 정렬된 순서로 저장될 때만 테이블에 클러스터형 인덱스가 포함된다.
- 클러스터형 인덱스가 포함된 테이블을 클러스터형 테이블이라고 한다. 
- 테이블에 클러스터형 인덱스가 없으면 해당 데이터 행은 힙이라는 정렬되지 않은 구조로 저장

> 힙(Heap)
> 힙으로 저장된 테이블에서 하나 이상의 비클러스터형 인덱스를 만들 수 있다. 
> 데이터는 순서를 지정하지 않고 힙에 저장
> 일반적으로 데이터는 처음에 행이 테이블에 삽입되는 순서대로 저장되지만 데이터베이스 엔진 에서 힙 안의 데이터를 이동하여 행을 효율적으로 저장할 수 있다. 따라서 데이터 순서는 예측할 수 없다. 
> 힙에서 반환되는 행의 순서를 지정하려면 ORDER BY 절을 사용해야 한다. 
> 행의 스토리지 순서를 지정하려면 테이블에서 클러스터형 인덱스를 만들어 테이블이 힙이 되지 않도록 합니다.

### Non-Cluster Index

- 비클러스터형 인덱스의 구조는 데이터 행으로부터 독립적
- 비클러스터형 인덱스에는 비클러스터형 인덱스 키 값이 있으며 각 키 값 항목에는 해당 키 값이 포함된 데이터 행에 대한 포인터가 있다.
- 비클러스터형 인덱스의 인덱스 행에서 데이터 행으로의 포인터를 행 로케이터라 함 
	- 행 로케이터의 구조는 데이터 페이지가 힙에 저장되는지 아니면 클러스터형 테이블에 저장되는지에 따라 다르다. 
		- 힙의 경우 행 로케이터는 행에 대한 포인터. 
		- 클러스터형 테이블의 경우 행 로케이터는 클러스터형 인덱스 키
- 클러스터형 인덱스의 리프 수준에 키가 아닌 열을 추가하여 기존 키 제한을 무시하고 완전히 포괄되는 인덱싱된 쿼리를 실행할 수 있다.
- primary key로 할당되지 않은 키에 대한 쿼리 성능을 높여준다.
- unique 키를 추가할 수 있게 해준다.

> #### Reference
> [DB의 Index 기능에 대해](https://nesoy.github.io/articles/2017-07/DBIndex)
> [클러스터형 및 비클러스터형 인덱스 소개](https://docs.microsoft.com/ko-kr/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described?view=sql-server-ver15)
> [힙(클러스터형 인덱스가 없는 테이블)](https://docs.microsoft.com/ko-kr/sql/relational-databases/indexes/heaps-tables-without-clustered-indexes?view=sql-server-ver15)
> [What do Clustered and Non clustered index actually mean?](https://stackoverflow.com/questions/1251636/what-do-clustered-and-non-clustered-index-actually-mean)
> [Clustered vs Non-clustered Index](https://www.guru99.com/clustered-vs-non-clustered-index.html)