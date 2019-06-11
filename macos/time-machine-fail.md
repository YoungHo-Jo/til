# Time Machine Backup Failure

## 상태

라즈베리파이 3 B에 라즈비안을 올려서 사용중이다. 현재 PHP, Docker, MySQL 등 많이 설치되어 좀 버거운게 현실이지만 주로 AFP를 올려서 타임머신 백업에 사용한다.
HDD는 두 개가 연결되어있고 라즈베리파이 공급되는 2.5A로는 전원이 부족하기에때문에 외부 USB 전원 공급이 가능한 허브로 두 HDD를 연결하였다.
각, 1TB, 500GB이며 후자의 HDD는 400G 부근에 물리적 베드 섹터가 존재한다. 
현재 타임머신 용으로는 500GB의 베드 섹터 이전부분만 할당하여 사용중이다.

## 증상

정상적으로 작동하다가 가끔식 `Backup Failed`가 일어난다.

> Time Machine couldn't complete the backup to "raspberrypi.local".
> The backup disk image "/Volumes/com.apple.TimeMachine.Time Machine-..." could not be accessed (error 30).


## 해결

먼저 Raspberrypi에서 `fsck.hfsplus [disk]`를 실행한다.

그래도 안되면 `umount` 하였다가 다시 `mount -a`를 해보고

다시 안될시 reboot을 한다. 

## 원인?

베드 섹터로 인한 문제로 짐작하고있으나, 베드 섹터가 없는 환경에서도 비슷한 증상이 일어나는 포스트를 본적이 있어서 확신할 수는 없다.

