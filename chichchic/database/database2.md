# CMD
DDL: 데이터 정의어
colum 생성 제거 등. 테이블 생성, 변경, 제거하는데 사용
DML: 데이터 조작어
데이터를 삽입, 수정, 삭제, 검색하는 기능
DCL: 데이터 제어어(운영자들이 사용한다.)
보안을 위해 데이터 접근 및 사용권한을 사용자별로 부여, 취소하는 기능

# 테이블 속성 정의
1. 이름(필수)
2. 데이터 타입(필수)
3. NULL 허용 여부
4. 기본값

**primary key**
기본키를 지정하는 키워드
Tuple을 확실히 구분해 줄 수 있는 속성. 대표성을 가진다.
NULL을 허용하지 않는다.

**unique**
속성에 대한 tuple값들이 중복되지 않는다.
NULL을 허용한다.
여러개 지정이 가능하다.

**foreign key**
외래키가 참조를 어디서 하는지 선언
참조 무결성 제약 조건: 동기화를 위해 옵션 지정

**constraint**
데이터 무결성 제약 조건
속성값에 제약을 두어 유효 데이터 값을 지정한다.

데이터 베이스 - 테이블 - 속성 순으로 작은 단위

## DDL
CREATE
ALERT
DROP
RENAME
TRUNCA
TE

## DCL
GRANT
REVOKE

## DML
SELECT
INSERT
UPDATE
DELETE
