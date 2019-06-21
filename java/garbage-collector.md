# Garbage Collector

#ProgrammingLanguage/Java

출처: [NAVER D2](https://d2.naver.com/helloworld/1329)
[성능튜닝 가비지 컬렉터(GC) 이해하기](https://12bme.tistory.com/57)


## Garbage
> 주소를 잃어버려서 사용할 수 없는 메모리  
> **Dangling Object**  
![](garbage-collector/bear_sketch@2x.png)

- - - -

## Garbage Collector
> 쓰레기를 정리해주는 프로그램  

* JVM 메모리가 부족한 상황에서 OS에 추가 메모리를 요청할 떄, GC를 작동
* 서버환경에서는 Idel time에 실행됨

- - - -

## 과정
### Stop-the-world
> GC를 실행하기 위해, JVM이 App 실행을 멈추는 것  

* GC실행하는 Thread 제외하고 나머지 Thread 모두 멈춤
* 작업을 끝낸후 재개
* 대개, **GC 튜닝**은 **stop-the-world** 시간을 줄이는 것
* ![](garbage-collector/bear_sketch@2x%202.png)

* 명시적 메모리 해제를 위해 `null` 사용하거나, `System.gc()` 를 부를떄, `null`은 괜찮으나, 후자는 절대 사용하면 안됨
- - - -

### 가정 / 전제조건
#### Weak Generational Hypothesis
* 대부분의 객체는 금방 접근 불가능한 상태(unreachable) 
* 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재

- - - -

### HotSpot JVM
크게 2개의 물리 공간으로 나눔

![](garbage-collector/22296F3C58FF0CA120.jpeg)

![](garbage-collector/bear_sketch@2x%203.png)

#### Young 영역(Young Generation 영역)
* 새롭게 생성된 객체 대부분 여기
* 대부분의 객체가 **금방 접근 불가능한 생태**가 되기 때문에 생성되었다가 사라짐 -> 사라질 때 **Minor GC 발생**한다고 함
* 총 3개의 영역으로 나누어짐
> **Minor GC**  
> Young 영역에서 일어나는 GC  
> **Major GC**  
> Old 영역이나 Perm 영역에서 일어나는 GC  


##### Eden 영역
* **새로 생성한 대부분의 객체**는 여기에 위치
* GC 한 번 발생 후 살아남으면 Survivor 영역으로 옮겨짐
* -> Survivor 영역으로 객체가 계속 쌓임 
Survivor 영역 중 하나는 항상 **비워진** 상태여야하기 때문에, 비워져있지 않은 Survivor영역으로 Minor GC에서 살아남은 객체가 이동한다. 




##### Survivor 영역(2개)(to, from)
* 하나가 가득차면 살아남은 객체를 다른 영역(다른 survivor 혹은 old)으로 이동 -> 가득찬 영역은 비워짐

위의 과정을 반복하다가 살아남은 객체는 Old로 이동



##### 특징
* Survivor 영역 중 하나는 반드시 비어있는 상태로 남아짐

![](garbage-collector/E526B6C6-F650-47B2-A1A8-3A1662D876B7.png)

**이러한 이동과정은 병목이 발생하기 떄문에 성능에 영향을 준다**

#### HotSpot JVM 의 특징
빠른 메모리 할당을 위해 -> ~bump-the-pointer~ & ~TLABs(Thread-Level Allocation Buffers)~


#### bump-the-pointer
* Eden 영역에 할당된 마지막 객체 추적
* 마지막 객체는 맨 위(top)에 있음
* 다음 생성시 이 할당공간에 넣기 적당한지 판단
* 적당할시 넣고, 새로 생성된 객체는 Top에 존재
* -> 새로운 객체 생성시 마지막에 추가된 객체만 점검 -> 빠른 메모리 할당
![](garbage-collector/bear_sketch@2x%204.png)

##### 문제점
* Multi Thread 환경에서는 lock이 발생할 수 밖에 없음
* -> Lock-contention으로 인한 큰 성능 하락
* -> **TLABs**로 해결

#### TLABs(Thread-Level Allocation Buffers)
* Eden 에 쓰레드마다 적절히 영역을 나누어 줌
* 쓰레드는 자기 영역만 접근 가능
* 자기 영역에 대한 **bump-the-pointer** 적용

![](garbage-collector/bear_sketch@2x%205.png)

#### Old 영역(Old Generation 영역)
* **접근 불가능한 생태**가 되지 않아 Young 영역에서 살아남은 객체들이 복사되는 곳
* 대부분 Young 영역보다 크게 할당
* 크기가 커 GC적게 발생
* -> 사라질때 **Major GC(Full GC)** 발생한다고 말함

![](garbage-collector/A715BDE9-A0F1-4640-843B-C98E4DDEF131.png)

* 데이터가 가득 차면 GC 실행
* GC 방식에 따라 처리 절차가 다름

##### GC 방식 (JDK 7)
* Serial GC
* Parallel GC
* Parallel Old GC(Parallel Compacting GC)
* Concurrent Mark & Sweep GC(CMS)
* G1(Garbage First) GC

##### Serial GC(-XX:+UseSerialGC)
> 운영 서버에서 절대 사용하면 안되는 방식  
* CPU 코어가 **하나** 만 있을때 사용하기 위함
* 처리하는 쓰레드가 **1개**

![](garbage-collector/2703E43958FF0CD22F.jpeg)
1. 살아있는 객체는 Eden에 존재
2. Eden 꽉 차면 To Survivor 영역으로 이동한다. 너무 크면 바로 Old로 이동한다. From Survivor에 살아있는 객체를 To Survivor로 이동시킨다.
3. To Survivor 꽉 찼을 경우, Eden 영역이나 From Survivor 영엑에 남아있는 객체들을 Old로 옮긴다.
4. 이후 Old 영역과 Perm영역은 **Mark-Sweep-Compact** 콜렉션 알고리즘에 따라 진행한다.

**mark-sweep-compact** 알고리즘 
1. Old 영역에 살아있는 객체 식별(**Mark**)
2. Heap 앞 부분 부터 확인하여 살아 있는 것만 남김(**Sweep**)
3. 객체들을 연속적으로 쌓이게 해, Heap의 가장 앞 부분 부터 채움
-> 객체 존재하는 부분 / 아닌 부분 나눔(**Compaction**)

![](garbage-collector/bear_sketch@2x%206.png)

> 적은 메모리와 CPU 코어 개수가 적을 때 적합  

##### Parallel GC(-XX:+UseParallelGC) == Throughput GC
- Serial GC와 기본 알고리즘 동일
- 처리하는 쓰레드가 **여러개** -> 빠른 처리
- 메모리 충분 / CPU 코어 개수 많을때 유용

##### Parallel Old GC(-XX:+UseParallelOldGC)
* JDK 5 update 6 부터 제공한 방식

**Mark-Summary-Compaction** 알고리즘
* **Summary** 단계에서 **Sweep** 단계와 달리 별도 살아있는 객체를 식별

##### CMS GC(-XX:+UseConcMarkSweepGC) == Low Latency GC
1. **Initial Mark** 
	* 클래스 로더에서 가장 가까운 객체 중 살아있는 객체만 찾음
	* 멈추는 시간이 매우 짧음
2. **Concurrent Mark**
	* 확인한 살아있는 객체를 참조하고 있는 객체들을 따라가면서 확인
	* **다른 쓰레드가 실행 중인 상태에서 동시에 진행**
3. **Remark** 
	* Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체 확인
4. **Concurrent Sweep** 
	* 쓰레기를 정리
	* **다른 쓰레드가 실행되고 있는 상황에서 진행**

**Stop-the-world**시간이 매우 짧음
-> 응답 속도가 매우 중요할 때 사용

![](garbage-collector/AB1A0D76-65E1-4766-8E2A-280B32F11D41.png)
	

###### 단점
* 다른 방식에 비해 **메모리와 CPU 많이 사용**
* Compaction 단계가 기본적으로 제공되지 않음
-> 조각난 메모리 많을 시, Compaction 하면 다른 방식보다 Stop-the-world 시간이 길어짐

-> 신중히 검토후 사용해야함

##### G1 GC
> 위의 방식과는 완전히 다른 방식  

![](garbage-collector/1F6E6E53-5A92-4240-9E8C-5C230B0ED087.png)

* 바둑판 영역에 객체 할당하고 GC 
* 해당 영역이 가득차면 다른 영역에 객체 할당하고 GC 실행(Young 영역의 세가지 영역에서 Old 영역으로 이동하는 단계가 없어진 것으로 보면된다)
* CMS GC대체하기 위해 만듦
* 성능이 우수!
* JDK7에서 정식으로 제공



## 주의!
> 환경에 따라 GC 옵션을 잘 선택해야한다  

* WAS 쓰레드 수
* 장비당 WAS 인스턴스 수
등등 영향을 주는 외부 요인들이 많다!

> 지속적인 튜닝 / 모니터링 통해 가장 적합한 옵션을 찾아야할 것  

