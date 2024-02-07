# DB-day13
/*
    계층형 쿼리
    오라클에서 지원하고 있는 기능 
    관계형 데이터베이스의 데이터는 수평적인 데이터로 구성되어있는데
    계층형 쿼리로 수직적 구조로 표현할 수 있음
    메뉴, 부서, 권한 등을 계층형쿼리로 만들 수 있음
*/
SELECT department_id
        ,LPAD(' ',3 * (LEVEL-1)) || department_name as 부서명
        ,LEVEL   --(계층) 트리 내에서 어떤 단계에 있는지 나타내는 정수값
        ,parent_id
        
FROM departments
START WITH parent_id is null --시작
CONNECT BY PRIOR department_id = parent_id; --구조가 어떻게 연결되는지
--departments 테이블에 데이터를 삽입하시오
--it헬프데스크 하위 부서로
--department_id: 280
--department_name : CHATBOT팀
SELECT a.employee_id
    , a.manager_id
    , LPAD(' ',3*(LEVEL -1)) || a.emp_name
    ,b.department_name
    ,a.department_id
FROM employees a, departments b
WHERE a.department_id = b.department_id
START WITH a.manager_id IS NULL
CONNECT BY PRIOR a.employee_id = a.manager_id
AND a.department_id =30;

INSERT INTO departments(department_name, department_id,parent_id)
VALUES('CHATBOT팀',280,230);

SELECT a.employee_id
    , a.manager_id
    , LPAD(' ',3*(LEVEL -1)) || a.emp_name
    ,b.department_name
    ,a.department_id
FROM employees a, departments b
WHERE a.department_id = b.department_id
START WITH a.manager_id IS NULL
CONNECT BY PRIOR a.employee_id = a.manager_id
--ORDER BY b.department_name; --계층형 트리가 깨짐
ORDER SIBLINGS BY b.department_name;--트리를 유지하고 동일 level 에서만


SELECT department_id
        ,LPAD(' ',3 * (LEVEL-1)) || department_name as 부서명
        ,parent_id
        ,CONNECT_BY_ROOT department_name as rootNm -- root row에 접근
        ,SYS_CONNECT_BY_PATH(department_name, '>') as pethNm
        ,CONNECT_BY_ISLEAF as leafNm --마지막노드1,자식있으면0 
FROM departments
START WITH parent_id is null --시작
CONNECT BY PRIOR department_id = parent_id;

CREATE table tb_level(
    이름 VARCHAR2(50)
    ,직책 VARCHAR2(50)
    ,레벨 number
    ,상위레벨 number
    );

INSERT INTO tb_level VALUES('이사장','사장',1,null);
INSERT INTO tb_level VALUES('김부장','부장',2,1);
INSERT INTO tb_level VALUES('서차장','차장',3,2);
INSERT INTO tb_level VALUES('장과장','과장',4,3);
INSERT INTO tb_level VALUES('박과장','과장',4,3);
INSERT INTO tb_level VALUES('이대리','대리',5,4);
INSERT INTO tb_level VALUES('김대리','대리',5,4);
INSERT INTO tb_level VALUES('최사원','사원',6,5);
INSERT INTO tb_level VALUES('강사원','사원',6,5);
INSERT INTO tb_level VALUES('주사원','사원',6,5);

SELECT*
FROM tb_level;

SELECT 이름
        ,LPAD(' ',3 * (LEVEL-1)) || 직책 as 직책단계
        ,레벨
        ,상위레벨--(계층) 트리 내에서 어떤 단계에 있는지 나타내는 정수값
FROM tb_level
START WITH 상위레벨 is null --시작
CONNECT BY PRIOR 레벨= 상위레벨;
/*
    계층형 쿼리 응용(샘플 데이터 생성)
*/
SELECT '2013'||LPAD(LEVEL,2,'0') as 년월
FROM dual
CONNECT BY LEVEL <=12;

SELECT period as 년월
    ,SUM (loan_jan_amt) as 대출합계
FROM kor_loan_status
WHERE period LIKE '2013%'
GROUP BY period;

SELECT a.년월
    ,NVL(b.대출합계,0) as 대출합계
FROM (SELECT '2013'||LPAD(LEVEL,2,'0') as 년월 -- 1워루터 12월 달력!!!
      FROM dual
      CONNECT BY LEVEL <=12) a

    ,(
    SELECT period as 년월
    ,SUM (loan_jan_amt) as 대출합계
     FROM kor_loan_status
     WHERE period LIKE '2013%'
     GROUP BY period) b
WHERE a.년월 = b.년월(+)
ORDER BY 1;

--202401~202412 sysdate를 이용하여 출력하시오
--connect by level 사용
SELECT TO_CHAR(SYSDATE, 'YYYY') || LPAD(LEVEL,2,'0') AS mm
FROM dual
CONNECT BY LEVEL <=12;
-- 이번달 1일부터 ~ 마지막날까지 출력하시오
-- 20240201
-- 20240202 ...

SELECT TO_CHAR(LAST_DAY(sysdate),'DD')
FROM dual
CONNECT BY LEVEL <= (SELECT TO_CHAR(LAST_DAY(sysdate), 'DD')
                    FROM dual);
-- member 회원의 생일 (mem_bir)를 이용하여
-- 월별 회원수를 출력하시오(모든월이 나오도록)
SELECT b.월
      ,NVL(a.수,0)
FROM(
SELECT TO_CHAR(mem_bir,'MM')as 월
      ,count(*) as 수
FROM member
GROUP BY TO_CHAR(mem_bir,'MM')
order by 1) a
,
(SELECT LEVEL as 월
FROM dual
CONNECT BY LEVEL <=12) b

WHERE a.월(+) = b.월
ORDER BY 1;




