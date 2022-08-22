# PYTHON과 MSSQL 연동하기
PYTHON으로 MSSQL DB를 연결하는 코드를 확인한다.

## 1.MSSQL DB 모듈 설치
pymssql라는 모듈을 이용해 MSSQL을 연결하고 데이터를 확인한다.

<code>
$ pip install pymssql

</code>

## 2.MSSQL 연결 쿼리

### (1) pymssql 패키지 import
<code>
import pymssql
 </code>
<br>
<br>

### (2)MSSQL 접속
connect 메소드를 사용해 DB를 연결한다. 호스트명, 로그인 등 접속정보를 파라미터로 지정해 통신한다.

#### -WINDOW 인증
<code>
conn = pymssql.connect(host=r"(local)", database='DB', charset='utf8')
 </code>
<br>
<br>

#### -SQL 인증
<code>
conn = pymssql.connect(host=r"(local)", user='sa', password='pwd', database='DB')
 </code>
<br>
<br>

### (3)Connection 으로부터 Cursor 생성
커서 객체를 가져온다.
<code>
cursor = conn.cursor()
 </code>
<br>
<br>

### (4)SQL문 실행
SQL문을 DB로 보내 해당하는 데이터를 가져온다.
<code>
cursor.execute('SELECT * FROM Customer;')
 </code>
<br>
<br>

### (5)데이타 하나씩 Fetch하여 출력
데이터를 한 행씩 출력한다.

<code>
row = cursor.fetchone()
while row:
    print(row[0], row[1])
    row = cursor.fetchone()
</code>
<br>
<br>

### (6) 연결 끊기
DB연결을 종료한다.

<code>
conn.close()            
</code>
<br>
<br>

##### 참고 자료
-Link: [MSSQL 사용 파이썬 예제][SQLlink]

[SQLlink]: http://pythonstudy.xyz/python/article/208-MSSQL-%EC%82%AC%EC%9A%A9 "web"
http://pythonstudy.xyz/python/article/208-MSSQL-%EC%82%AC%EC%9A%A9

-Link: [pymssql 사용법][SQLlink]

[SQLlink]: https://ponyozzang.tistory.com/630 "web"
https://ponyozzang.tistory.com/630