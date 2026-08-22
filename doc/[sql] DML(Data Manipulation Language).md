# MySQLv8 - DML
Google Cloud Standard SQL - DML
IBM - DML
MySQL 데이터베이스 한번에 끝내기 (SQL Full Tutorial Course using MySQL Database)

DML (Data Manipulation Language) ?
DATA 값을 조작하는 명령어 - INSERT, UPDATE, DELETE



## DML: INSERT, UPDATE, DELETE
- SELECT
- VALUES
- INSERT
- INSERT(MySQL Workbench)
- INSERT INTO SELECT
- UPDATE
- DELETE


## VALUES
VALUES는 괄호 (values...) 단위로 ROW를 구성하는 가상 TABLE 생성 명령이다.

VALUES (1, 2);
column1	column2
1	2

VALUES (1), (2);
column1
1
2

VALUES (1, 2), (3, 4);
column1	column2
1	2
3	4



## INSERT
테이블 이름 다음에 나오는 열 생략 가능
생략할 경우에 VALUE 다음에 나오는 값들의 순서 및 개수가 테이블이 정의된 열 순서 및 개수와 동일해야 함

INSERT INTO <table>
VALUE(val1, val2, ...);


INSERT INTO test
VALUE(1, 123, 1.1, "TEST");
SELECT * FROM test;



## INSERT INTO SELECT
test 테이블에 있는 내용을 test2 테이블에 삽입

INSERT INTO test2 SELECT * FROM test;
SELECT * FROM test2;


[MYSQL] SELECT 한 내용 INSERT 시키는 방법

1. select 한 내용의 전체 컬럼 Insert

INSERT INTO [table] SELECT * FROM [table] WHERE [조건];

당연한 이야기지만 select하는 테이블과 insert할 테이블의 컬럼은 일치해야 합니다.


2. 원하는 컬럼만 select 해서 Insert

INSERT INTO [table] (column1, colum2, colum3) SELECT column1, colum2, colum3 FROM [table] WHERE [조건];

PRIMARY키가 있어 1번의 방법으로 INSERT가 안되는 경우 PRIMARY키를 제외한 컬럼을 직접 선택해서 INSERT하는 방법입니다.



++
위의 글을 참조한 이유는 잘 동작하던 쿼리문이

UID를 추가한 이후 
Column count doesn't match value count at row 1 의 에러 문구가 나왔었다.
실제 소스를 보면서 설명하겠습니다.

insert into HPG
select system, cbroff, consl from CNS
where system = system ; 

일 경우 에러가 없던 문구에서
Column (UID)를 추가하였습니다. 했더니 에러 문구가 위의 msgbox로 출력되었습니다.
즉 From table과 Insert Table의 컬럼 수가 일치하지 않다는 것으로 판단됩니다.
따라서, UID는 자동 증가이기 때문에

insert into HPG(system, cbroff, consl)
select system, cbroff, consl from CNS
where system = system ;

를 추가 함으로 써 에러를 수정할 수 있었습니다.


## INSERT INTO <columns...> ON CONFLICT (<key>) DO UPDATE SET VALUES ();
UPDATE SET에서
EXCLUDED.<column> 표현은 CONFLICT시 VALUE에 지정한 신규 column 값을 의미한다.
기존 값의 경우 <table>.<column> 으로 표현한다.

예제) https://www.postgresql.org/docs/current/sql-insert.html
DROP TABLE IF EXISTS test;
TRUNCATE TABLE test;

CREATE table IF NOT EXISTS test (
  id INT PRIMARY KEY,
  a INT NOT NULL,
  b INT NOT NULL,
  c INT NOT NULL
);

INSERT INTO test(id, a, b, c) VALUES(1, 1, 1, 1);

SELECT * FROM test;

INSERT INTO test AS t (id, a, b, c)
VALUES (1, 2, 2, 2)
ON CONFLICT (id) 
DO UPDATE SET
  a=EXCLUDED.a,
  b=t.b;

SELECT * FROM test;

INSERT INTO test (id, a, b, c)
VALUES (1, 3, 3, 3)
ON CONFLICT (id) 
DO UPDATE SET 
(a, b, c) = (EXCLUDED.a, EXCLUDED.b, EXCLUDED.c);

SELECT * FROM test;
# EXCLUDED의 역할
Table의 기존 값이 아닌 INSERT를 통해 새로 지정한 값
EXCLUDED는 충돌로 인해 삽입되지 않은 값들을 참조합니다.
즉, 삽입하려고 시도했으나 충돌한 데이터가 EXCLUDED를 통해 참조됩니다.
이를 사용하여 기존의 데이터를 업데이트할 수 있습니다.



## UPDATE <dst.tb> SET <dst.k>=<val>,… FROM <src.tb> WHERE <dst.k>=<src.v>

기존에 입력되어 있는 값 변경하는 구문
WHERE절 항상 사용 할 것, WHERE절 생략 가능하나 생략시 테이블 전체 행이 변경 대상이 됨. 

# simple exam
UPDATE test
SET col1=1, col2=1.0, col3='test'
WHERE id=1;

SELECT * FROM test;


# 기본 UPDATE 구문
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;

### 예시 1: UPDATE 기본 사용
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    salary NUMERIC,
    position VARCHAR(100)
);

INSERT INTO employees (name, salary, position)
VALUES
    ('John Doe', 50000, 'Engineer'),
    ('Jane Smith', 60000, 'Manager'),
    ('Tom Jones', 45000, 'Intern');

UPDATE employees
SET salary = 55000, position = 'Senior Engineer'
WHERE id = 1;


### 예시 2: 조건(WHERE) 없이 전체 업데이트
특정 조건 없이 모든 행을 업데이트할 수도 있습니다. 다만, 이는 주의해서 사용해야 하며, 조건 없이 실행할 경우 테이블의 모든 행이 업데이트됩니다.

UPDATE employees
SET salary = salary + 5000;


### 예시 3: 여러 조건을 사용한 업데이트
UPDATE employees
SET salary = 65000
WHERE position = 'Manager' AND salary < 65000;


### 예시 4: 하위 쿼리와 함께 사용한 업데이트
UPDATE employees
SET salary = salary + 10000
WHERE id IN (SELECT id FROM employees WHERE position = 'Engineer');


### 예시 5: 여러 행을 한꺼번에 업데이트 (UPDATE ... FROM)
CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(100)
);

INSERT INTO departments (department_name)
VALUES ('Engineering'), ('Marketing'), ('HR');

UPDATE employees
SET department_id = d.department_id
FROM departments d
WHERE employees.position = 'Engineer' AND d.department_name = 'Engineering';


### 예시 6: VALUES 구문을 사용한 다중 업데이트
UPDATE employees AS e
SET salary = v.new_salary
FROM (VALUES 
    (1, 60000),  -- id가 1인 직원의 급여를 60000으로 업데이트
    (2, 65000)   -- id가 2인 직원의 급여를 65000으로 업데이트
) AS v(id, new_salary)
WHERE e.id = v.id;


## INSERT and UPDATE

SELECT e
FROM pg_type t 
JOIN pg_enum e ON t.oid = e.enumtypid  
JOIN pg_catalog.pg_namespace n ON n.oid = t.typnamespace
WHERE t.typname = 'en_clean_table_type';

SELECT * FROM tb_table_clean_config;
SELECT * FROM tb_table_clean_config WHERE clean_table = 'TB_SERVER_ALERT_LOG';
INSERT INTO tb_table_clean_config AS ('TB_SERVER_ALERT_LOG', TRUE, 'EMS', 1, 1007) ON CONFLICT (clean_table) DO NOTHING;
UPDATE tb_table_clean_config SET storage_limit_date=180 WHERE clean_table = 'TB_SERVER_ALERT_LOG';


## DELETE
행 단위로 데이터 삭제하는 구문
DELTE FROM <table> WHERE <condition-expr>;

데이터는 지워지지만 테이블 용량은 줄어들지 않음
실제로는 데이터공간에 지워진것으로 마크만 될뿐 실제 데이터 공간 삭제X, 그러므로 데이터 값 복구가능
원하는 레코드만 골라 지울 수 있음
삭제 후 잘못 삭제한 것을 되돌릴 수 있음

DELETE FROM test
WHERE id=1;

SELECT * FROM test;


## DELETE(DML) vs TRUNCATE(DDL)
DELETE 명령으로 삭제한 데이터는 COMMIT 전이라면 복구가 가능 
DELETE는 DML이며  여러개의 DML이 하나의 transaction이므로 DELETE 사용시 COMMIT 명령을 따로 실행해야 변경사항이 바로 테이블에 적용됨
TRUNCATE는 DDL이며  하나의 DDL이 하나의 transaction이므로 TRUNCATE 사용시 AUTO COMMIT이 적용되어 변경사항이 즉시 테이블에 적용됨

DELETE ?
테이블의 특정행 삭제. 단, WHERE 절을 생략하면 모든행이 삭제.
삭제된 데이터의 공간 반납 X, TABLE 용량은 줄어들지 않음.
COMMIT 하지 않았다면 ROLLBACK 명령을 통해 취소 가능.

TRUNCATE?
테이블의 구조는 남기고(DROP과 차이점) TABLE의 보유 데이터 전체 삭제.
삭제된 데이터의 공간 반납 X, TABLE 용량은 줄어들지 않음.
자동 COMMIT 되므로 명령 실행 취소 불가능.

ex) 다음 두 가지 명령 실행 후 SELECT 결과는 같지만 DELETE로 삭제한 데이터는 COMMIT 이전에 복구 가능
DELETE FROM <table>;
TRUNCATE TABLE <table>;
