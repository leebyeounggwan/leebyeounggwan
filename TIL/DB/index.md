인덱스 컬럼 선정 시
1. Where절에서 like, =, between 등의 조건과 함께 사용되는 컬럼(컬럼 변형있는 경우 제외)
2. Where절에서 부정형(not, <>), null 비교 사용하는 컬럼은 제외 (부정형이나 null비교를 제거할 수 있으면 가능)
3. 분포도가 나쁠 경우 제외

결합 인덱스 사용 시
결합인덱스에 구성된 컬럼들 중 반드시 첫 번째 조건에 지정되어야 해당 인덱스가 사용됨

인덱스 설계 절차
접근 경로 수집 -> 분포도 조사에 의한 후보컬럼 선정 -> 접근 경로 결정 -> 컬럼 조합 및 순서 결정

접근 경로  ' col1 like 'a100%' and col2 = '20041010' '

컬럼조합 및 순서 결정 방법
- 단일 컬럼의 분포도가 양호하면 단일 컬럼 인덱스로 확정
- 항상 사용되는 컬럼을 선두 컬럼으로 (선행컬럼 미존재시 결합인덱스 이용 x)
- 등치 조건으로 사용되는 컬럼을 선행 컬럼으로
	1) <, >, <=,>=,between like 등의 비교연산시 해당 컬럼 이후 range Scan
	2) Like , between <, > 보다 등치(=)조건 컬럼 먼저 수행
- Order by, group by 순서 적용 
	-인덱스는 컬럼 값이 정렬되어 있으므로, 오더바이나 그룹바이절에 사용되는 컬럼순으로 인덱스를 생성하면 정렬 부하를 줄일 수 있다.

인덱스가 사용되지 않는 경우
1. 인덱스 컬럼에 외부적 변형이 가해질 때
2. 인덱스 컬럼에 내부적 변환(type conversion) 이 발생할 때
3. 부정형 조건 사용시
4. Is null, is not null 사용시 (인덱스에는 null에 대한 정보가 없어서 풀스캔 발생) 
	-> char형의 경우 ' '로 바꿔 인덱스 사용되도록 가능
5. Date, number 형 컬럼에 like 사용시
	-> like 연산자는 연산 대상 컬럼이 character형이 아닐때 해당 컬럼을 변환
6. Like 조건에 %로 시작하는 비교값이 들어오면 인덱스 사용 x
7.  Having절의 컬럼에 대한 인덱스는 사용 x where절에서 인덱스를 사용해 데이터를 가져 온 후 사용
8. In 연산자는 인덱스를 이용하지만 or로 연결되는 구문이므로 개수가 많아지면 성능 저하 우려가 있다.
9. 결합 인덱스에서 like, between 사용된 컬럼 까지만 인덱스가 사용되므로 뒤로 미룬다.


```
Select dept, ename, sal
From emp
Where 

1. sal * 12 < 5000;
-> sal < 5000/12;
2. Dept = 7788; (dept가 char type일때)
-> Dept = to_char(7788);
3. Dept != '005'
-> dept > '005'
Or dept < '005'
4. Empno <> 1122;
-> not exists (select 'x' from emp where empno = 1122);
5. Date is not null
-> date > 0;
6. Job like '%BCD';
-> job like 'ABCD%';
7. Group by deptid
   Having deptid = 211;
-> where deptid = 211
   Group by deptid;

```


IN / EXISTS
- 데이터 존재여부 판단을 위한 in 과 exist는 결과는 같으나 내부로직은 다름.
- 데이터 집합의 처리 범위가 넓고 부분 범위 처리가 필요한 경우는 EXISTS가 유리, 데이터 집합의 처리 범위가 좁고 결과 데이터 집합이 작다면 IN이 유리

```
Select empno, dept
From emp e
Where deptno in (select deptno from dept d);
```
:  IN 절은 일반적으로 전체 범위 처리를 통해 서브 쿼리의 조건을 만족하는 모든 데이터 집합을 구한 다음, 메인쿼리의 조건을 처리(제공 역할), 서브쿼리 수행 후 dept의 범위를 줄여 성능 보장

```
Select empno, dept
From emp e
Where exists (select deptno from dept d where e.deptno = d.deptno);
```
: exist의 서브 쿼리는 메인 쿼리가 찾은 데이터에 대해 Loop 형식으로 동작 --> 메인 쿼리가 먼저 수행, 서브 쿼리는 존재 여부만 판단(확인 역할)
Emp 테이블의 레코드 개수에 따라 전체 성능이 영향을 받으므로 emp의 범위를 줄이는 것이좋다.
* deptno에 null이 있을 경우 null은 비교할 수 있는 데이터가 아니므로 최종 결과에 나타나지 않는다.


그룹함수
1. Sum 함수는 null 값을 자동 제외하고 합을 구하므로 nvl을 쓸 필요 없다.
2. Avg 함수도 null을 제외하고 평균을 구하므로 
`avg(col) != avg(nvl(col,0)`
3. 그룹함수(count제외) 사용 시 처리 대상 집합이 공집합이어도 논리적인 결과는 존재한다고 본다.
```
Select col, nvl(col, 0)
From tb
Where 1=2; 
-> No Rows Selected.

Select MAX(col), COUNT(col)
From tb
Where 1 = 2;
-> NULL NULL 0 0
   1 row selected.
```

OR
- AND 연산자는 많이 사용할수록 처리범위 줄어듦
- OR는 가급적 간단하게 사용 union all, decode, in 등으로 대체
