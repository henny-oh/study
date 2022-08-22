# SYNONYM 데이터 컬럼 조회하기
ORACLE SYNONYM(시노님)의 칼럼과 데이터 타입을 조회하는 쿼리문이다.

<pre>
<code>
SELECT A.COLUMN_NAME, A.DATA_TYPE 
FROM all_tab_columns A
JOIN ALL_SYNONYMS B
        ON A.TABLE_NAME =B.TABLE_NAME
WHERE B.SYNONYM_NAME='<찾고싶은 SYNONYM 명>'
</code>
</pre>
ALL_SYNONYMS 테이블은 SYNONYM과 연결된 테이블명을 함께 보여준다.
여기서 매칭되는 테이블을 all_tab_columns 테이블에서 찾아 해당하는 칼럼명과 데이터타입을 확인한다.
