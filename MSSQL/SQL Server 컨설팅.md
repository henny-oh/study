추적 플래그
기술 지원 대상이 아니므로 부작용 발생가능
->사용에 주의하고 학술, 테스트 용도로만 제한

data platform virtual summit 2022
->sql server 2017&2019 intelligent query processing 참가

-db 관리 부문
1. sql server 2019 설치할 때 memory 변경 시 경고 메시지
   메모리 탭에서 min/max 지정 
   기본값? 권장?

   권장값 사용하려고 하면 에러메시지 : 최소 서버 메모리를 최대 서버 메모리와 동일한 값으로 수정해서는 안됩니다. 계속하려면 sql server 데이터 베이스 엔진에 대해 권장되는 메모리 구성에 동의해야 합니다.

    ->t신경쓰지말고 필요한대로 setup해서 사용하기

    값 수정(custom memory) 고객도 많은데 max/min 설정 오류나 부적절한 설정으로 인한 오류가 많아서 띄우는 오류
    ->multi instance 환경이나 다른 메모리 경합 이슈 등도 많다고 함

    ->주의 차원에서의 에러 메시지

    기본방식 : sql server, 장치에 대해 모를 때 사용하기
    
2. 고민 - 대량 데이터 delete 방어/ 로깅 / 모니터링 방안
    -접근 제어 / 감사 솔루션 사용
    : where절 없는 delete나 update 등 사전에 차단
    :고비용 -ssms 용 저렴한 서드 파티 툴에서 지원

    -dml instead of 트리거 사용
    :특정 테이블 단위 방어/로깅에 사용

    -테이블 단위 행 수 변경 이력추적
    :sys.partitions.rows

    -확장 이벤트 사용 (가능성 제시. 실제 미 활용)
    :일반 sql 이벤트에서 delete 필터링
    :sql 실행 관련 부가 정보도 추적 가능

    -기타
    : alter, drop 등의 작업은 ddl trigger로 방어/추적 가능

    -권한 제한

    ?코오롱베니트에서는 어떻게 처리하는가

3. 사례 mssql_dq 대량 대기 & i/o,cpu,duration 발생
   ->linked server에서 발생할 수 있음

   -mssql_dq
   :원격 쿼리로 다른 서버에서 한 서버로 쿼리 수행 후 결과를 기다리는 대기.
   :multiple active result set(mars)에서 데드락 인지할 때 사용(흔치 않음)

   -발생
   :exec 형태로 원격 프로시저 호출 시 호출 서버 쪽에서 대기 발생
   ->기본적으로 oleddb, async network io 대기도 호출 쪽에서 발생
   테스트 결과 : open (quert, rowset) 에서는 미발생

4. cmemthread 대기 대량 발생으로 서비스 문제 지속
    긴급 지원- 진단 분석
    memoryclerk sqlgeneral  메모리 65gb 점유 발견
    ->비동기 통계 업데이트 이슈(과거 알려진 메모리 누수 버그)는 고갯사 비 해당

    momory object 상세 확인
    ->memobj msxml 대량 메모리 점유 확인
    ->메모리 누수 이슈로 판단

    원인 추정
    sp_xml_preparedocument
    연계된 관계ㅖ사 db 서비스 장애로 대량 호출 중 작업 실패
    remove document가 일관 미완료된 것으로 추정 - 동일 증상 재현 성공

    조치
    해당 memory clear를 위한 다양한 시도 실패
    결국 서비스 재시작 후 해결

5. 사례-RESOURCE semaphore 대기, 쿼리 실행 메모리 조정
   한 쿼리가 사용할 수 있는 최대 실행 메모리 = max memory *(70~90%)*(20~25%)
   (max memory : resource pool 의 max memory 설정)

   쿼리 실행 메모리가 너무 크거나/작거나/parameter sniffing/etc 문제가 된다면
   -쿼리 튜닝
   -쿼리 실행 메모리 조정(특히 대용량 메모리)
    :resource pool의 디폴트 값 조정
    :쿼리 힌트 MIN_GRANT_PERCENT, MAX_GRANT_PERCENT(2012+)

6. SSMS에서 데이터베이스 분리(온라인 상태) 중 서비스 문제 발생
   데이터베이스 분리 -> '연결 삭제' 체크 옵션
   -실제론 SINGLE_USER모드 전환 시도 : 실행중인 트랜잭션 모두 ROLLBACK (ALTER DATABASE [DB] SET SINGLE_USER WITH ROLLBACK IMMEDIATE 후 sp_detach작업 실행)
   -대량 / 큰 트랜잭션 수행 시 문제 발생
    :단순 SELECT 만 수행 중이면 시도 가능

    주의 사항
    사전에 DB 연결/ 사용 세션 확인
    SSMS GUI 사용은 항상 주의 필요
    -가능하면 SCRIPT로 처리
    PLAN CACHE 삭제 등도 주의

7. sql server 2016 parallel redo 성능 저하(부작용)
   이슈 : standby 서버에서 병렬 redo 시 대량 대기 발생
   조치 : trace flag 3459(병렬 redo disable)
   다른 버그 : 이후 tf3459 해제 후 parallel 동작 안함
   추가 권장 사항 : 관련 fetch (or cu) 업데이트 할 석 ( no "cu" 상태는 비 권장)

8. xtp_thread_pool 명령 세션
   -in-memory OLTP thread pool
   ![Alt text](/MSSQL/IMAGE/7.PNG)

9. delayed durability(2014+) 성능 테스트-지연된 내구성
    특수한 시나리오 또는 환경에서 의미있는 기능
    
    acid
    -full vs delayed durability
    insert/delete/update ->commit : transaction log에 기록
    해당옵션을 설정하면 log write가 비동기 ( log 끝날때까지 기다릴 필요 없음)

     sys.sp_flush_log로 보완 :수동으로 중간중간 log write

10. standard edition, soft luma, cpu masking 그리고  maxdop
    standard edition은 24코어, 48cpu 사용(window 80cpu)

11. hammerdb로 sql server 성능 테스트
    before after 비교 가능 -테스트 동안 모니터링 및 성능 정보 수집 필요

12. 사례 -db옵션 "대상 복구 시간(checkpoint)" 설정 값 조정 부작용
    불규칙하게 서비스 성능/안정성 문제 발생
    600초(10분)으로 설정
    ->10분 단위 대량 io로 서버 병목 발생

13. tempdb 성능 문제
    증상 
    pagelatch_대기 대량 발생
    ->pfs(page free space) 페이지 병목
    objlock 대량 대기, i/o 대량 대기 etc

14. hypothetical index
    껍데기 인덱스
    데이터는 없이 인덱스 메타 정보와 통계만 생성 (비공식 옵션 사용)
    autopilot 모드에서 query optimizer가 사용
        -일종의 인덱스 성능 시뮬레이션
        -실제 인덱스 유효성 사전 평가 활용
        -테스트 or 개발 장비 필요

         with statistics_only 0: 비공계옵션 1: 껍데기 인덱스, -1:인덱스+ 통계정보


15. bitlocker와 sql 823,17053,9001
    디스크 관리자 ->bitlocker로 암호화됨->윈도우즈 장치 암호화옵션 설정되어 있었음

16. 해킹, 랜섬웨어
    웹 서버의 게시판 파일 업로드 등의 취약점 이용
    윈도우 bitlocker로 디스크 암호화 후 금전 요구

    window 뿐만 아니라 sql server도 보안 경계
    -관리자 권한 가진 서비스 계정, sa, xp_cmdshell

17. sql injection, time attck(time based blind sql injection)
    고부하 쿼리

    방어조치
    ->app에서 command timeout지정 패턴 사용
        오류 로그 기록 후 동일 오류 반복 또는 대량 발생 시 알림
    장시간 수행되는 쿼리 모니터링
    보안 솔루션 사용

18. query store off 또는 clear 시 disk 대기 이슈
    fix : transactions and log truncation may be blocked when you use query store in sql server 2016 and 2017
    해결 : 2017cu11+, 2016 sp2 cu5+
            db 사용량이 작거나 점검 때 off 또는 clear
            모니터링 필수
    2019+에서 force 옵션 지원
    alter database {0} set query_store =off(forced) ->2019 cu6+,2017 cu 21+에서만 지원하는 것으로 문서에 언급

19. msdtc 실패
    포트 차단으로 인한 오류
    DTCPing 테스트

20. preemptive_com_querynterface 대기로 compile 대기/차단 발생
    원격쿼리, 컴파일 차단, 후속 호출 모두 차단, 로그인 쿼리로 호출빈도 최다

    ->원인 결과
    open tran=1,serializabvble 사용으로 단순 원격 select도 dtc 승격
    dtc에서 필요한 포트 막혀서 drc  사용 실패
    원격 서버 쿼리 추적 시 쿼리 자체가 들어오지 않음
    확장 이벤트 wait 분석 시 dtc 호출/ 비호출에 따라서 수집된 정보가 달랐음(dtc  사용 시 문제 발생)
    ->preemptive_os_reverttoself 대기 발생
    결국 dtc 포트가 막혀 있어 원격 쿼리/ dtx 호출 자체가 장시간 대기/차단 발생

21. d-raid 6 vs raid 10
    고객 시나리오
    신규 hw 장비 도입 +sql server upgrade
    신규 san storage(full flash) 포함
    
    hw 관련 정보 리뷰 및 제안
    벤더사에서 d-raid6 추천
    읽기 대비 쓰기가 4~5배 많은 고객 db 고려
    검증을 우해 raid10와의 비교 테스트 제안
    disk BMT 도구로 고객사에서 간단한 테스트 수행->raid 10



쿼리 부문

1. 2022 Intelligent query processing(iqp) 기능 (새로운 차원) 확장/진보

2. option(use hing(force_legacy_cardinality_estimation))은 재컴파일 유발하는가
해당 힌트는 2017/2019 등 업그레이드 시 신규 ce로 인한 부작용 시 사용 가능
실제 적용 시 재컴파일 발새 여부가 궁금
->유발하지 않음!!


