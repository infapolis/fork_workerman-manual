# 바쁜 프로세스 디버깅
가끔 ```php start.php status``` 명령을 사용하여 ```busy``` 상태의 프로세스를 볼 수 있습니다. 이는 해당 프로세스가 작업을 처리하고 있음을 나타내며, 보통 작업이 완료되면 프로세스가 정상적인 ```idle``` 상태로 복구됩니다. 보통은 큰 문제가 없습니다. 그러나 계속해서 ```busy``` 상태임을 유지한다면, 프로세스 내의 작업이 블록되거나 무한 루프에 머물고 있다는 것을 의미하며 다음 방법을 사용하여 이를 식별할 수 있습니다.

## strace + lsof 명령어 사용하여 식별

**1. busy 프로세스의 pid 찾기**
```php start.php status```를 실행한 뒤 다음과 같이 표시될 때
![](../images/d1903ed65ef2f3b0850e84ccbedc52aa.png)
그림에서 ```busy``` 프로세스의 ```pid```는 ```11725``` 및 ```11748```입니다.

**2. 프로세스 추적(strace)**
하나의 프로세스 ```pid```(여기서 ```11725```를 선택)를 선택하고, ```strace -ttp 11725```를 실행하면 다음과 같이 나타납니다.
![](../images/7ce9f36da926f670949609dcdc593ab4.png)
작업이 fd가 16인 설명자의 읽기 이벤트를 기다리는 끊임없는 루프인 것을 볼 수 있습니다. 이는 이 설명자가 데이터를 반환할 때까지 기다리는 것입니다.

시스템 호출이 표시되지 않는 경우 현재 터미널을 유지한 채로 다른 터미널을 열어 ```kill -SIGALRM 11725```(프로세스에 대해 알람 시그널을 보냄)를 실행하고 strace 터미널에서 응답이 있는지, 특정 시스템 호출에 블로킹되어 있는지 확인할 수 있습니다. 여전히 시스템 호출이 표시되지 않으면 프로그램은 아마도 비즈니스 루프에 머물고 있을 가능성이 높으며 페이지 하단의 "프로세스를 오랫동안 busy 상태로 만드는 다른 이유" 항목의 2번을 참조하여 해결할 수 있습니다.

epoll_wait 또는 select 시스템 호출에서 블록되어 있는 경우, 이는 프로세스가 이미 ```idle``` 상태임을 나타냅니다.

**3. 프로세스 설명자(lsof) 확인**
```lsof -nPp 11725```를 실행하면 다음과 같이 나타납니다.
![](../images/27bd629c3a1ac93f9f4b535d01df2ac1.png)
설명자 16에 해당하는 것은 16u 레코드(마지막 행)로, ```fd=16``` 설명자가 원격 주소가 ```101.37.136.135:80```인 TCP 연결임을 알 수 있습니다. 따라서 이 프로세스는 HTTP 리소스에 액세스하는 것으로 보이며, ```poll([{fd=16, events=...``` 루프는 이 설명자가 데이터를 반환할 때까지 기다리는 것으로 설명합니다. 이것이 이 프로세스가 ```busy``` 상태에 있는 이유를 설명합니다.

**해결:**
프로세스가 어디에 블록되어 있는지를 파악했으므로, 이제 문제를 해결하기 쉽습니다. 예를 들어 위에서 식별한 것에 따르면, 비즈니스가 curl을 호출하고 해당 URL이 오랫동안 데이터를 반환하지 않아 프로세스가 계속해서 기다리고 있는 것으로 보입니다. 이 경우 URL 제공자에게 URL 반환이 느린 이유를 확인하고, 동시에 curl 호출할 때 타임아웃 매개변수를 추가하여 2초 이상 반환되지 않으면 타임아웃되게끔 설정하여 긴 시간 동안 블로킹되지 않도록 합니다(이 경우 프로세스는 약 2초간 ```busy``` 상태가 될 수 있습니다).

## 프로세스를 긴 시간 busy 상태로 만드는 다른 이유
프로세스가 블록되거나 ```busy``` 상태로 만드는 것 외에도, 다음과 같은 이유로 ```busy``` 상태에 있을 수 있습니다.

**1. 비즈니스에 치명적인 오류가 발생하여 프로세스가 계속해서 종료되는 경우**
**현상:** 이러한 경우 시스템 부하가 상당히 높으면 ```status```의 `load average`가 1 또는 그 이상일 것입니다. 프로세스의 `exit_count` 숫자가 상당히 높아져 지속적으로 증가합니다.
**해결:** (```-d``` 없이) workerman을 디버그 방식(```php start.php start```)으로 실행하여 비즈니스 오류를 확인하고 수정하면 됩니다.

**2. 코드에서 무한 루프**
**현상:** top에서 busy 프로세스가 높은 CPU를 사용한다는 것을 확인할 수 있으며, ```strace -ttp pid``` 명령어로 어떠한 시스템 호출 정보도 출력되지 않습니다.
**해결:** [gdb 및 PHP 소스 코드로 식별](https://www.laruence.com/2011/12/06/2381.html)하는 방법을 참조하고 다음 단계를 거쳐 진행합니다.
1. ```php -v```로 버전을 확인합니다.
2. [해당하는 PHP 버전의 소스 코드를 다운로드](https://www.php.net/releases/)합니다.
3. ```gdb --pid=busy 프로세스의 pid```를 실행합니다.
4. ```source php 소스 경로/.gdbinit```를 실행합니다.
5. ```zbacktrace```를 사용하여 호출 스택을 출력합니다.
마지막 단계에서는 현재 실행 중인 PHP 코드의 호출 스택을 볼 수 있으며, 이는 PHP 코드의 무한 루프 위치를 나타냅니다.
참고: ```zbacktrace```가 호출 스택을 출력하지 않는 경우에는 당신이 사용한 PHP 컴파일에 ```-g``` 매개변수가 추가되지 않았을 수 있으므로 PHP를 다시 컴파일하고 workerman을 다시 시작하여 식별할 수 있습니다.

**3. 타이머를 무한히 추가하는 경우**
비즈니스 코드가 계속해서 타이머를 추가하는 데 이를 삭제하지 않아 프로세스 안에 타이머가 계속해서 늘어나게 되어 프로세스가 무한한 타이머를 실행하도록 만드는 것입니다. 아래 코드는 이러한 예시입니다.
```php
$worker = new Worker;
$worker->onConnect = function($con){
    Timer::add(10, function(){});
};
Worker::runAll();
```
위의 코드는 클라이언트가 접속할 때마다 타이머를 추가하지만, 전체 비즈니스 코드에서 타이머를 삭제하는 로직이 없어 시간이 지남에 따라 프로세스 내에서 계속해서 타이머가 증가하고 마침내 프로세스가 무한 타이머를 실행하게 됩니다.
올바른 코드는 다음과 같습니다.
```php
$worker = new Worker;
$worker->onConnect = function($con){
    $con->timer_id = Timer::add(10, function(){});
};
$worker->onClose = function($con){
    Timer::del($con->timer_id);
};
Worker::runAll();
```