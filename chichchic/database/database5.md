# 클라우드 컴퓨팅
정보 분석 및 처리 저장 관리 유통이 클라우드 공간에서 이루어지는 컴퓨팅 시스템

## 장점
초기 구입 비용이 낮다.
휴대성이 높다.
가용율이 향상된다.
다양한 기기를 단말기로 사용할 수 있다.
안전하게 데이터를 보관할 수 있다.
서버에 사용되는 인프라 구축 비용이 절감된다.(전문 지식이 필요 없어짐)

## 단점
공격 당하면 개인정보 유출 가능성이 있다.
재해에 의해 서버가 손상되면 복구가 불가능하다.
APP 설치에 있어 제약이 있다.(새로운 APP 미지원 등)
열악한 통신 환경에서는 서비스가 어렵다.
개인정보가 어디있는지 물리적으로는 알기 어렵다.

# 빅데이터
IT의 발전으로 다룰 수 있게된 방대한 양의 데이터

## 특징
양이 방대하고 생성 속도가 빠르다.
정형화 되지 않은 다양한 형태의 데이터로 이루어져있다.

## 기존 데이터와 차이점
데이터는 정형화 되어있지만, 빅 데이터는 비정형화 되어있는 다양한 데이터(문자, 영상, 위치 등)를 다룬다.
하드웨어 고가의 저장 장치가 기존에는 data warehouse였으면 빅데이터는 주로 클라우드 컴퓨팅을 사용한다.
소프트 웨어를 분석하는데 있어 RDBMS는 SAS, SPSS, data minig등을 사용했으나, 빅데이터는 R, text mining, opinion mining등을 사용한다.

# Hashing

bucket: 1개 이상의 data를 저장하는 단위, 여러 record를 저장하는 단위이다.
hashing 함수: 바로 찾을 수 있게 해주는 함수로 search key값을 통해 address를 매칭시켜준다.
search key값이 다르지만 address가 같은 경우가 있다.
static hasing: 미리 bucket의 수를 고정시켜 둔 hasing

## hash function
최악의 경우 모든 search key값이 같은 bucket을 이용하게 된다.
uniform(일정)하고 random하게 들어가는게 이상적이다.

**overflow bucket**
skewed(ununiform)된 경우 bucket이 부족해서 bucket을 생성한 후 연결하는 방식
아무리 좋은 hash function이라고 해도 완전이 overflow bucket을 제거할 수 없다.

## static hasing의 비효율성
DB의 크기는 grow and shrink한다.
이로 인해 너무 많은 overflow가 발생하거나 너무 많은 공간이 낭비될 수 있다.
따라서, 주기적으로 re-organization하여 예방하는 dynamic hasing을 사용하는것이 좋다.

## dynamic hasing(extendable hasing)
최대 2^32까지 가능
처음에는 prefix만 사용한다.
실제 bucket의 수는 < 2^i
**![hash prefix](https://raw.githubusercontent.com/chichchic/GAZA_COMMERCE/main/chichchic/images/hash_prefix.png)

### insertion
1. h(k_i) = x 게산
2. i개의 상위 비트값만 이용해 bucket address table에서 bucket 주소를 반환
3. bucket에 공간이 있을 경우 데이터를 삽입
4. bucket에 공간이 없을 경우 split

**split 하는 방법**
- if, i(add table의 hash_prefix) > i_j(bucket의 hash prefix)
bucket을 새로 생성한 후 i_j값을 1 증가한다. 이전 bucket에 있던 값을 전부 reinsert한다.
- if, i == i_j
i값을 증가시키거나 overflow bucket(사용하고 있는 bucket 수를 확인해서 결정한다.)
이후 다시 값들을 reinsert한다.

### deletion
insertion의 반대 과정
1. h(k_i) = x 계산
2. bucket에 data 삭제
3. bucket에 아무런 data가 없을 경우 bucket을 merge(split의 반대)

# bitmap indices
multikey를 사용하여 검색할 때 사용한다.
record가 fixed될 때 효과적이다.
record number에 따라 값이 존재(1 아니면0으로 만든다. record수가 bit수와 일치)
연산을 사용해서 동시에 만족하는 record를 쉽게 찾을 수 있다.
ans, or, not 연산을 매우 빠르게 수행할 수 있는 장점이 있다.
sql에 이러한 index를 만들 수 있는 문법을 각각 제공한다.

**제한 조건**
record가 많지 않다.
속성이 가질 수 있는 범위가 작다.
