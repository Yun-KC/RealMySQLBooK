## 클러스터링 인덱스

> **클러스터링이란?**  
> 여러 개를 하나로 묵는다는 의미로 사용되는데, 주어진 데이터들이 얼마나, 어떻게 유사한 지에 따라 데이터셋을 요약, 정하는 것이다.

### **클러스터링 인덱스**

MySQL 서버에서 클러스터링은 테이블의 레코드를 비슷한 것(프라이머리 키를 기준으로)들 끼리 묶어서 저장하는 형태로 구현한다.  
이는 주로 비슷한 값들이 동시에 조회하는 경우가 많다는 점에서 착안한 것이다.
MySQL 에서 클러스터링 인덱스는 InnoDB 스토리지 엔진에서만 지원한다.

클러스터링 인덱스는 테이블의 프라이머리 키에 대해서만 적용되는 내용이다. 즉 프라이머리 키 값이 비슷한 레코드끼리 묶어서 저장하는 것을 **클러스터링 인덱스**라고 표현한다.

- 프라이머리 키 값에 의해 레코드의 저장 위치가 결정된다.

  - 클러스터링 인덱스는 인덱스 알고리즘이라기보다 테이블 레코드의 저장 방식이라 볼 수 있다.

- 프라이머리 카 값이 변경된다면 레코드의 물리적인 저장 위치가 바뀌어야 한다.
- 프라이머리 키 값으로 클러스터링된 테이블은 프라이머리 키 값 자체에 대한 의존도가 상당히 크기 때문에 신중히 프라이머리 키를 결정해야한다.
- 프라이머리 키 기반의 검색이 매우 빠르며, 대신 레코드의 저장이나 키의 변경이 상대적으로 느리다.

> _주의_  
> 일반적으로 B-Tree 인덱스도 인덱스 키 값으로 이미 정렬되어 저장된다. 하지만 B-Tree를 클러스터링 인덱스라고 부르지 않는다.  
> 테이블의 레코드가 프라이머리 키 값으로 정렬되어 저장된 경우만 "클러스터링 인덱스" 또는 "클러스터링 테이블"이라고 한다.

**프라이머리 키가 없다면?**  
프라이머리 키가 없는 경우 InnoDB 스토리지 엔진은 프라이머리 키를 대체할 칼럼을 선택한다.

1. NOT NULL 옵션의 유니크 인덱스(UNIQUE INDEX) 중에서 첫 번째 인덱스
2. 자동으로 유니크한 값을 가지도록 증가하는 칼럼을 내부적으로 추가
   - 이 경우 아무 의미 없는 숫자 값으로 크럴스터링 되는 것이며, 우리에게 아무런 혜택을 주지 않는다.
   - 클러스터링 인덱스는 테이블당 단 하나만 가질 수 있는 엄청난 혜택이므로 가능하다면 프라이머리 키를 명시적으로 생성해야 한다.

### **세컨더리 인덱스에 미치는 영향**

InnoDB에서 세컨더리 인덱스가 실제 레코드 주소를 가지고 있다면, 클러스터링 키 값이 변경될 때마다 레코드의 주소가 변경되는 특성 때문에 테이블의 모든 인덱스에 저장된 주솟값을 변경해야 할 것이다. 때문에 InnoDB 테이블(클러스터링 테이블)의 모든 세컨더리 인덱스는 해당 레코드의 주소가 아니라 프라이머리 키 값을 저장하도록 구현되어 있다.

### **클러스터링 인덱스의 장점과 단점**

- 장점
  - 프라이머리 키(클러스터링 키)의 검색 처리 성능이 매우 빠름(특히, 프라이머리 키를 범위 검색하는 경우)
  - 테이블의 모든 세컨더리 인덱스가 프라이머리 키를 가지고 있기 때문에 인덱스만으로 처리될 수 있는 경우가 많음
- 단점
  - 테이블의 모든 세컨더리 인덱스가 클러스터링 키를 갖기 때문에 클러스터링의 키 값의 크기가 클 경우 전체적으로 인덱스의 크기가 커짐
  - 세컨터리 인덱스를 통해 검색할 때 프라이머리 키로 다시 한번 검색해야 하므로 처리 성능이 느림
  - INSERT할 때 프라이머리 키에 의해 레코드의 저장 위치가 결정되기 때문에 처리 성능이 느림
  - 프라이머리 키를 변경할 때 레코드를 DELETE하고 INSERT하는 작업이 필요하기 때문에 처리 성능이 느림
- 요약
  - 클러스터링 인덱스의 장점은 빠른 읽기, 단점은 느림 쓰기
  - 일반적인 웹 서비스와 같은 온라인 트랜잭션 환경에서는 쓰기와 읽기의 비율이 2:8 또는 1:9 정도이기 때문에 조금 느린 쓰기를 감수하고 읽기를 빠르게 유지하는 것이 매우 중요하다.

### **클러스터링 테이블 사용 시 주의사항**

- 클러스터링 인덱스 키의 크기에 따라 인덱스 크기가 달라진다.
- 프라이머리 키에 따라 레코드의 위치가 결정된다. 따라서 AUTO-INCREMENT 보다 업무적인 칼럼(해당 레코드를 대표할 수 있는 칼럼)으로 생성해야 한다.
- 프라이머리 키는 반드시 명시해야 함.
- 여러 개의 칼럼이 복합으로 프라이머리 키가 만들어지는 경우 프라이머리 키의 크기가 길어질 때 AUTO-INCREMENT 칼럼을 인조 식별자로 사용할 수 있다.
  - 로그 테이블과 같이 조회보다는 INSERT 위주의 테이블들은 AUTO-INCREMENT를 이용한 인조 식별자를 프라이머리 키로 설정하는 것이 성능 향상에 도움이 된다.

## 유니크 인덱스

유니크는 테이블이나 인덱스에 같은 값이 2개 이상 저장될 수 없음을 의미한다. (인덱스라기보다는 제약 조건에 가깝다고 볼 수 있다.)

MyISAM이나 MEMORY 테이블에서 프라이머리 키는 NULL이 허용되지 않는 유니크 인덱스와 같지만, InnoDB 테이블의 프라이머리 키는 클러스터링 키의 역할도 하므로 유니크 인덱스와는 근복적으로 다르다.

### **유니크 인덱스와 일반 세컨더리 인덱스의 비교**

유니크 인덱스와 유니크하지 않은 일반 세컨더리 인덱스는 사실 인덱스의 구조상 아무런 차이점이 없다.

- 인덱스 읽기
  - 유니크 인덱스와 세컨더리 인덱스는 성능상 거의 차이가 없다.
  - 유니크 하지 않은 세컨더리 인덱스는 중복된 값이 허용되므로 읽어야 할 레코드가 많아서 느린 것이지, 인덱스 자체의 특성 떄문에 느린 것이 아니다.
  - 읽어야 할 레코드 건수가 같다면 성능상의 차이는 미미하다.
- 인덱스 쓰기
  - 유니크 인덱스의 키 값을 쓸 때는 중복된 값이 있는지 없는지 체크하는 과정이 한 단계 더 필요하다. 그래서 유니크하지 않은 세컨더리 인덱스의 쓰기보다 느리다.
  - MySQL에서는 유니크 인덱스의 중복된 값을 체크할 때는 읽기 잠금을 사용하고, 쓰기를 할 때는 쓰기 잠금을 사용하는데 이 과정에서 데드락이 아주 빈번히 발생한다.
  - InnoDB 스토리지 엔진에서는 인덱스 키의 저장을 버퍼링하기 위해 체인지 버퍼(Change Buffer)가 사용된다. 그래서 인덱스의 저장이나 변경 작업이 상당히 빨리 처리되지만, 유니크 인덱스는 중복 체크를 해야 함으로 작업 자체를 버퍼링하지 못한다. 이 때문에 유니크 인덱스는 일반 세컨더리 인덱스보다 변경 작업이 더 느리게 작동한다.

### **외래키**

MySQL에서 외래키는 InnoDB 스토리지 엔진에서만 생성할 수 있으며, 외래키 제약이 설정되면 자동으로 연관되는 테이블의 칼럼에 인덱스까지 생성된다. 외래키가 제거되지 않은 상태에서는 자동으로 생성된 인덱스를 삭제할 수 없다.

InnoDB의 외래키 관리에 중요한 특징

- 테이블의 변경(쓰기 잠금)이 발생하는 경우에만 잠금 경합(잠금 대기)이 발생한다.
  - 자식 테이블의 외래 키 칼럼의 변경은 부모 테이블의 확인이 필요한데, 이 상태에서 부모 테이블의 해당 레코드가 쓰기 잠금이 걸려 있으면 해당 쓰기 잠금이 해제될 때까지 기다리게 되는 것이다.
- 외래키와 연관되지 않은 칼럼의 변경은 최대한 잠금 경합(잠금 대기)을 발생시키지 않는다.
  - 자식 테이블의 외래키가 아닌 칼럼의 변경은 외래키로 인한 잠금 확장이 발생하지 않는다.

물리적으로 외래키를 생성하면 자식 테이블에 레코드가 추가되는 경우 해당 참조키가 부모 테이블에 있는지 확인한다. 외래 키를 생성할 때 고려 사항은 이를 체크하기 위해 연관 테이블에 읽기 잠금을 걸어야 한다는 것이다. 이렇게 잠금이 다른 테이블로 확장되면 그만큼 전체적으로 쿼리의 동시 처리에 영향을 미치기 때문이다.