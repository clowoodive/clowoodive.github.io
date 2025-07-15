---
title: "[MySQL] - Windows에서 MySQL General Log 백업 방법"
excerpt: ""

categories:
  - MySQL
tags:
  - MySQL
  - Database
last_modified_at: 2025-07-14T00:00:00
---

{% capture notice-env %}
#### Environment
 - MySQL 8.0
 - Windows Server 2022
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


Windows Server 2022 인스턴스에 설치해서 사용중인 MySQL의 디버깅 용도로 general log를 켜놓았었다. 

파일이 점점 커져서 가끔 로그를 확인할 때 시간도 걸리고 디스크 용량 관리도 할겸 지워야 하는데, 하나의 파일에 쌓인 로그를 지우면 최근 로그도 지워져서 로깅을 하는 의미가 퇴색되는 문제가 있다.

로그 파일 롤링을 하면 좋겠지만 MySQL 자체에 general log 롤링 기능이 없으며, 윈도우에서는 리눅스에서 처럼 logrotate 같은 유틸리티도 없기 때문에 다른 방법을 써야한다.

MySQL 공식 문서를 보면 아래와 같은 두 가지 방법이 있다.

### 방법 1

```
// Windows 에서는 rm 대신 rename
$> mv host_name.log host_name-old.log
$> mysqladmin flush-logs general
$> mv host_name-old.log backup-directory
```

첫 번째 방법은 파일명을 먼저 변경하고 flush를 통해 파일 핸들을 닫고 다시 여는 방법이다. 

이 방법 관련 내용들을 인터넷에서 검색해 보면 파일에 대한 lock으로 인해 파일명 변경이 불가능할 수도 있다는 얘기도 있는데, 공식 문서에서는 그런 내용은 없다.

### 방법 2

```sql
SET GLOBAL general_log = 'OFF';
-- 파일명 변경
SET GLOBAL general_log = 'ON';
```

두 번째 방법은 로그 자체를 끄고 파일명을 변경하는 방법인데, 이 방법대로 테스트 해보면 로그를 일시적으로 끄기 때문에 그 사이에 발생한 로그가 유실될 수 있다는 것을 알수있다.

따라서 첫 번째 방법을 시도해 보고 안되면 그때 두 번째 방법을 쓰기로 했다.

전체 과정은 로그를 백업하는 커맨드를 배치파일로 작성하고, 스케쥴러에 등록하는 두가지 단계로 구성했다.

### 롤링 커맨드 배치파일

```tsx
@echo off
setlocal

:: 설정
set LOG_DIR=C:\ProgramData\MySQL\MySQL Server 8.0\Data
set FILE_NAME=general
set LOG_FILE=%FILE_NAME%.log
set SIZE_LIMIT_MB=5

:: 파일 크기 확인
for %%A in ("%LOG_DIR%\%LOG_FILE%") do set /A FILE_SIZE_MB=%%~zA / 1048576

:: echo %FILE_SIZE_MB%

if %FILE_SIZE_MB% LSS %SIZE_LIMIT_MB% (goto break)

for /f "skip=1" %%l in ('wmic os get locale') do (
    set LOC=%%l
    goto done
)
:done

:: 한국어 (0x0412 = 1042), 영어(미국: 0x0409 = 1033)
if "%LOC%"=="0412" (
	:: yyyy-mm-dd
	set DATESTAMP=%DATE%_%TIME:~0,2%%TIME:~3,2%%TIME:~6,2%
) else if "%LOC%"=="0409" (
	:: 영문 로캘(Fri mm/dd/yyyy) 인 경우
	set DATESTAMP=%DATE:~10,4%-%DATE:~4,2%-%DATE:~7,2%_%TIME:~0,2%%TIME:~3,2%%TIME:~6,2%
) else (
	goto break
)

CD /d %LOG_DIR%
set BACKUP_FILE=%FILE_NAME%_%DATESTAMP%.log

rename "%LOG_FILE%" "%BACKUP_FILE%"
"C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqladmin.exe" -u root --password=pass1! flush-logs general

:: 오래된 로그 삭제 (30일 초과)
:: forfiles /p "%LOG_DIR%" /m %FILE_NAME%_*.log /d -30 /c "cmd /c del @path"

:break
endlocal

```

### 스케쥴러에 등록

Task Scheduler UI로 구성해도 되지만 효율성을 위해 커맨드로 작성했다.

```
SchTasks /Create /TN "MysqlGeneralLogBackupTask" /SC Daily /ST 05:00 /TR "C:\Server\MysqlGeneralLogBackup.bat"
```

해당 서버는 테스트 용이라 하루동안 쌓이는 로그양도 크지 않으며 백업 파일의 크기를 맞출 필요도 없기에 하루 한번 수행하도록 커맨드를 작성했다. 따라서 하루만에 로그가 많이 쌓인다면 백업 로그 사이즈가 커질수도 있다.

### References

- [https://dev.mysql.com/doc/refman/8.0/en/query-log.html](https://dev.mysql.com/doc/refman/8.0/en/query-log.html)