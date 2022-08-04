 # 특정 명령어가 권한 문제로 실행 되지 않을 때


실행 권한이 없어 TRUNCATE 명령어가 실행되지 않는다는 메시지를 확인했다.

이 문제를 해결하기 위해서는 방법이 두가지이다.

    1)  ALTER 권한 이나 DDLadmin db role 부여
    2)  프로시져를 생성해 EXECUTE 권한 부여

(1)의 문제는 해당 권한을 부여하게 되면 다른 테이블에도 해당 명령어를 사용할 수 있어 문제가 생길 수 있다.
 특정 TABLE을 TRUNCATE하는 프로시저를 작성하는 (2) 방법을 사용한다면 특정 테이블만 TRUNCATE할 수 있다.

<hr/>

## 1)  ALTER 권한 이나 DDLadmin db role 부여
ALTER 권한이나 DDLadmin 역할 멤버를 추가하면 truncate명령어를 실행 할 수 있다.

### -ALTER 권한
<pre>
<code>
USE TEST
GO
GRANT ALTER TO [user_id]
GO
</code>
</pre>
USE [DB명]으로 ALTER 실행권한을 줄 DB를 지정하고 
GRANT EXECUTE TO [계정명]으로 권한을 줄 계정을 지정한다.


### -db_ddlamdin 역할 멤버 추가
<pre>
<code>
USE TEST
GO
ALTER ROLE [db_ddladmin] ADD MEMBER [user_id]
GO
</code>
</pre>
ALTER ROLE [역할멤버자격] ADD MEMBER [계정명]으로 해당 역할에 계정을 추가한다.


<hr/>

## 2)  프로시져를 생성해 EXECUTE 권한 부여
아래 SQL문으로 OFFICE라는 TABLE을 TRUNCATE하는 프로시저를 작성한다.

<pre>
<code>
CREATE PROCEDURE dbo.TRUNCATE_OFFICE
AS
BEGIN
	TRUNCATE TABLE OFFICE
END
GO
</code>
</pre>
CREATE PROCEDURE [스키마명].[프로시저명]의 형태로 프로시저 명을 지정하며 TRUNCATE TABLE [테이블명]으로 TRUNCATE 할 테이블을 지정한다.

<pre>
<code>
USE TEST
GO
GRANT EXECUTE TO [user_id]
GO
</code>
</pre>

USE [DB명]으로 EXECUTE 실행권한을 줄 DB를 지정하고 
GRANT EXECUTE TO [계정명]으로 권한을 줄 계정을 지정한다.

<pre>
<code>
EXEC dbo.TRUNCATE_OFFICE
</code>
</pre>
위 SQL문을 통해 프로시저를 수행하면 지정한 테이블만 TRUNCATE할 수 있다.

<hr/>

## GUI로 권한 확인하기
보안 > 로그인 > 계정 > 속성 > 사용자 매핑에서 접근할 DB를 매핑 및 역할 멤버 자격을 부여한다.
  
  -사용자 매핑
![Alt text](/MSSQL/IMAGE/2.PNG)
    
-데이터베이스 역할 멤버 지정
![Alt text](/MSSQL/IMAGE/1.PNG)


데이터베이스 > 속성 > 사용 권한에서 계정의 사용권한을 확인 및 설정 할 수 있다.

![Alt text](/MSSQL/IMAGE/3.PNG)

<명시적> 탭에서 허용을 한 사용권한과 연관된 명령어는 <유효> 탭에서 확인할 수 있다. 확인할 수 있는 명령어는 모두 사용할 수 있다.
![Alt text](/MSSQL/IMAGE/4.PNG)

**참고
 TRUNCATE 명령어는 ALTER 권한이 있으면 수행 가능하다.