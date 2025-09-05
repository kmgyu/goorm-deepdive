# 자바 가비지 컬렉터 (GC)
## 가비지 컬렉터란?

**가비지 컬렉터(Garbage Collector, GC)**는 프로그래머 대신 사용이 끝난 객체의 **메모리를 자동으로 회수**해주는 메커니즘입니다. 프로그램이 실행되면서 더 이상 필요 없게 된 객체(garbage, 쓰레기 객체)가 힙(heap) 메모리에 쌓이는데, 가비지 컬렉터는 주기적으로 힙을 검사해 이러한 쓰레기 객체를 정리합니다.

모든 언어가 가비지 컬렉터를 사용하는 것은 아닙니다. **C/C++ 같은 언매니지드 언어(unmanaged language)**에서는 개발자가 `malloc`/`free` 또는 `new`/`delete` 등을 통해 **직접 메모리를 할당/해제**해야 합니다. 이러한 수동 관리 방식은 잘만 사용하면 오버헤드가 적지만, 실수하면 _메모리 누수_나 _이중 해제_ 같은 문제가 발생할 수 있습니다. 반면 **자바, C# 등의 매니지드 언어(managed language)**는 가비지 컬렉터와 같은 자동 메모리 관리 장치를 통해 개발자가 명시적으로 메모리 해제를 하지 않아도 됩니다. 예를 들어 자바 개발자는 JVM의 GC 덕분에 메모리 해제 코드를 직접 작성하지 않고도, 메모리 누수나 오버플로우 문제를 크게 줄일 수 있습니다.

## 왜 가비지 컬렉터를 알아야 할까?

가비지 컬렉터는 **자동으로 메모리 관리**를 해주기 때문에, 평소에는 개발자가 GC의 동작을 깊이 의식하지 않아도 됩니다. 그러나 **GC 동작을 알아두면 좋은 이유**가 몇 가지 있습니다. 첫째, 메모리 누수(memory leak)나 OutOfMemory 오류 등이 발생했을 때 내부 메커니즘을 이해하지 못하면 문제 원인을 파악하고 해결하기 어렵습니다. 둘째, GC로 인해 **예기치 않은 성능 저하**가 발생할 수 있기 때문입니다. 예를 들어 GC가 실행되는 동안 애플리케이션이 **일시 정지(STW, Stop-The-World)**되는데, 이 **GC 일시 정지(pause)** 시간이 길어지면 응답 속도에 영향을 줄 수 있습니다. 동시성 요구사항이 높은 시스템에서는 GC가 병목이 될 수 있으므로, GC 튜닝이나 모니터링을 통해 **일시 정지 시간을 줄이고** 애플리케이션 성능을 향상시킬 수 있어야 합니다.

 

응답 속도가 중요한 애플리케이션(예: GUI 프로그램이나 웹서비스)은 **짧은 GC 일시정지**가 필수적이며, GC로 인해 긴 멈춤이 생기면 곧바로 체감 성능 저하로 이어집니다. 반면 배치 처리나 데이터 분석 같이 **처리량(throughput)**이 더 중요한 작업은 **일시정지 시간이 길어져도** 상관없는 대신 **GC가 전체적으로 빠르게 많은 객체를 처리**하는 능력이 중요합니다. 이러한 **응답성 vs 처리량**의 트레이드오프 때문에 자바에는 다양한 GC 알고리즘이 존재하며, 각 GC마다 **일시정지 시간을 줄이는 정도와 처리 효율**이 다릅니다. 다음 섹션부터 GC의 기본 원리와 여러 GC 알고리즘의 특성을 알아보겠습니다.

## 객체의 생존 여부 판단: Reachability vs Reference Counting

가비지 컬렉터는 _어떤 객체가 메모리에서 해제되어도 되는지_를 판단해야 합니다. **객체의 생존 여부를 판단하는 대표적인 두 가지 방법**이 있습니다: **참조 카운팅(reference counting)**과 **도달 가능성 분석(reachability analysis)**입니다.

- **참조 카운팅 알고리즘**: 각 객체에 **참조 횟수 값을 저장**해두고, 객체를 참조하는 변수가 하나 늘어날 때마다 카운트를 +1, 참조가 줄어들 때마다 -1 합니다[happybplus.tistory.com](https://happybplus.tistory.com/1043#:~:text=%EA%B0%9D%EC%B2%B4%EB%A5%BC%20%EA%B0%80%EB%A6%AC%ED%82%A4%EB%8A%94%20%EC%B0%B8%EC%A1%B0%20%EC%B9%B4%EC%9A%B4%ED%84%B0%20%EB%A5%BC,%EC%B6%94%EA%B0%80%ED%95%98%EA%B3%A0). 이 카운트가 0이 되면 더 이상 어떤 곳에서도 참조되지 않는 객체이므로 가비지로 간주하고 메모리를 회수합니다. 참조 카운팅은 구현이 간단하고 실시간으로 객체의 유효 여부를 알 수 있다는 장점이 있어 **Python의 CPython**이나 **Objective-C, Swift의 ARC** 등에서 사용됩니다. 하지만 **자바의 JVM에서는 참조 카운팅을 사용하지 않습니다】. 그 이유는 순환 참조(Circular Reference) 문제 등 특수한 상황을 처리하기 어렵기 때문입니다. 예를 들어 객체 A와 B가 서로를 참조하는 경우(순환 구조) 둘 다 다른 곳에서 참조되지 않아도 **참조 카운트는 1로 남아** 있기 때문에, 참조 카운팅만으로는 둘을 가비지로 인식하지 못합니다. (Python은 이런 순환 참조를 해결하기 위해 **주기적인 사이클 탐지 GC**를 별도로 수행합니다.)
    
- **도달 가능성 분석 알고리즘**: 현대의 Java, C# 등 대부분의 가비지 컬렉터는 **도달 가능성(reachability)**을 기반으로 객체 생존을 판정합니다. 도달 가능성 분석은 **GC 루트(GC Root)**라 불리는 특별한 객체 집합을 시작점으로 삼아, 그 루트에서 **참조(chain)를 따라 도달 가능한 모든 객체를 생존 객체로 간주**합니다. GC 루트란 **스택 변수, 전역 및 정적 변수, 레지스터, JNI 레퍼런스** 등 애플리케이션에서 바로 접근할 수 있는 루트 객체들을 말합니다. GC는 이 루트 객체들이 참조하고 있는 객체들, 또 그 객체들이 참조하는 객체들을 **그래프 탐색**하듯이 따라가서 표시(marking)합니다. 이때 **어느 GC 루트에서도 도달할 수 없는 객체**는 더 이상 프로그램에서 사용할 수 없는 **가비지 객체**로 판단하여 수거 대상으로 표시합니다. 자바 JVM뿐만 아니라 대부분의 모던 런타임(JVM, CLR 등)이 이 **Tracing GC(추적 가비지 컬렉션)** 방식을 택하고 있습니다. 도달 가능성 기반 방식은 참조 카운팅의 순환 참조 문제를 해결하고, 보통 **고성능(throughput)** 측면에서도 유리하지만, 단점으로 한 번 GC를 돌 때 애플리케이션을 잠시 멈추고 힙 전체를 훑어야 하므로 **일시 정지 시간**이 발생한다는 점이 있습니다.
    

요약하면, 자바 GC는 **Reachability 기반**으로 동작하기 때문에 개발자가 객체 참조를 `null`로 지우는 등의 직접적인 메모리 해제는 필요 없지만, 반대로 어떤 객체가 메모리 해제될 시점을 정확히 예측하기는 어렵습니다. (반면 Python의 경우 참조 카운팅 덕분에 **대부분 객체가 즉시 해제**되므로, `__del__`을 통한 파괴자 호출 타이밍이 비교적 예측 가능합니다.)

## 가비지 컬렉션의 기본 알고리즘들

객체의 생존 판정 이후, **GC는 가비지 객체들을 실제로 회수(수집)하는 단계**를 수행합니다. 가장 기본적인 GC 수집 기법 몇 가지를 알아보겠습니다:

- **Mark-Sweep (마크-스윕)**: 이름 그대로 **"표시하고 쓸어담기"** 두 단계로 이루어진 알고리즘입니다. 먼저 GC는 앞서 설명한 도달 가능성 분석 등을 통해 **쓰레기 객체들을 모두 표시(mark)**합니다[happybplus.tistory.com](https://happybplus.tistory.com/1043#:~:text=%EB%A8%BC%EC%A0%80%20%ED%9A%8C%EC%88%98%ED%95%A0%20%EA%B0%9D%EC%B2%B4%EB%93%A4%EC%9D%84%20%EB%AA%A8%EB%91%90%20%ED%91%9C%EC%8B%9C%ED%95%9C,%EB%8B%A4%EC%9D%8C). 그 다음 **표시된 객체들을 한꺼번에 메모리에서 제거(sweep)**합니다[happybplus.tistory.com](https://happybplus.tistory.com/1043#:~:text=%EB%A8%BC%EC%A0%80%20%ED%9A%8C%EC%88%98%ED%95%A0%20%EA%B0%9D%EC%B2%B4%EB%93%A4%EC%9D%84%20%EB%AA%A8%EB%91%90%20%ED%91%9C%EC%8B%9C%ED%95%9C,%EB%8B%A4%EC%9D%8C). 마크-스윕은 구현이 간단하고 현재까지도 기본으로 쓰이는 기법이지만, 단점으로 **GC 실행 시간이 힙 내 객체 수에 비례해 오래 걸릴 수 있고(pauses 시간의 불규칙성)**, **메모리 단편화(fragmentation)** 문제가 발생합니다. 많은 객체를 한 번에 표시하고 삭제해야 하므로 **힙에 객체가 많을수록 GC 시간이 길어**지고, 또 가비지들을 치운 자리에 **빈 공간들이 듬성듬성 남아** 메모리가 조각날 수 있습니다. 단편화가 심해지면 큰 객체 할당 시 연속된 큰 빈 공간을 찾기 어려워져, JVM이 **다시 GC를 잦게 일으키는 악순환**이 생길 수도 있습니다.
    
- **Mark-Copy (마크-복사)**: 마크-스윕의 첫 번째 단점(많은 가비지 처리 시 효율 저하)을 개선한 알고리즘입니다. 이 방식에서는 가용 메모리를 **동일 크기의 두 영역으로 반씩 나누어** 한쪽 영역만 사용합니다. 한쪽이 꽉 차면 **살아남은 객체들만 반대쪽 영역으로 복사**하고, 이전 영역을 통째로 비우는 식입니다. 한 번의 GC에서 대부분 객체가 죽는다면 **소수의 생존 객체만 복사**하면 되므로 효율적이고, 덤으로 **메모리도 연속적으로 정리**되기 때문에 단편화 문제가 아예 없습니다. 객체들을 복사하면서 힙 한쪽 끝부터 차곡차곡 채워넣으므로 **할당도 빠른 bump-pointer 방식**을 쓸 수 있습니다. Mark-Copy는 구현도 상대적으로 간단하고 빨라서, **젊은 객체들이 주로 죽는 환경에서 뛰어난 성능**을 보입니다. 단점은 말 그대로 **메모리의 절반을 복사용으로 비워둬야** 해서 공간 낭비가 있다는 것입니다.
    
- **Mark-Compact (마크-압축)**: 마크-스윕의 두 번째 단점(메모리 단편화)을 해결한 알고리즘입니다. 먼저 **표시(mark)** 단계는 기존 마크-스윕과 동일하게 수행합니다. 이후 **압축(compact)** 단계에서 **모든 생존 객체들을 힙의 한쪽으로 몰아서** 복사 배치하고, 나머지 빈 공간은 한 번에 정리합니다. 이렇게 하면 메모리 공간이 연속적으로 확보되어 단편화 문제가 해소됩니다. Mark-Compact의 단점은 **객체를 이동시키는 추가 작업**이 필요하기 때문에 GC 수행 동안 약간의 오버헤드가 있다는 것입니다. (대신 압축을 통해 메모리가 정돈되므로, 이후 **객체 할당이 빨라지고** 대규모 힙에서도 **메모리 활용 효율이 높아지는** 이점이 있습니다.)
    

위 알고리즘들은 GC의 **기본적인 동작 원리**를 나타내지만, 현대 JVM의 GC들은 이들을 단독으로 쓰기보다는 **각 장단점을 조합**하여 사용합니다. 예컨대, 자바의 기본 GC는 **어린 객체 영역(Young Generation)**에는 Mark-Copy 알고리즘을, **오래된 객체 영역(Old Generation)**에는 Mark-Sweep 또는 필요시 Mark-Compact 알고리즘을 적용합니다. 이러한 전략을 이해하려면 다음에 설명할 **세대별 수집 기법**을 알아야 합니다.

## 세대별 GC와 자바 힙 구조

현대 JVM 가비지 컬렉터는 **"Generational Garbage Collection(세대별 수집)"** 개념을 기반으로 설계됩니다. **세대별 수집 이론**은 다음 두 가지 경험적 사실에 근거합니다:

1. 대부분의 객체는 **금방 참조되지 않게 되어** "일찍 죽는다".
    
2. 한 번 GC에서 살아남은 객체는 **오랫동안 살아남을 가능성이 커진다**.
    

이 가설에 따라, JVM은 힙 메모리를 **최소 두 개 이상의 영역(Generations)**으로 나누고 객체를 **생존 기간(나이)**에 따라 다른 영역에 분리하여 관리합니다. 새로운 객체들은 우선 **Young Generation(젊은 세대)**에 할당되고, 그 중 여러 번의 GC를 견뎌낸 **장수 객체**들만 **Old Generation(노년 세대)**으로 승격(promotion)됩니다[happybplus.tistory.com](https://happybplus.tistory.com/1043#:~:text=Image). (자바 8까지는 여기에 **Permanent Generation(영구 영역)** 혹은 Metaspace도 존재했지만, 이는 클래스 메타데이터 용도로 GC와는 별개이므로 여기서는 생략합니다.)

 

**Young Generation** 내부는 다시 세분화되어 **Eden(에덴)** 영역과 두 개의 **Survivor(서바이버) 영역**으로 구성됩니다. 객체 생성 시마다 **새 객체는 Eden에 할당**되며, Eden이 가득 차면 **Minor GC(마이너 GC)**가 일어납니다. Minor GC에서는 Reachability 분석으로 Eden의 쓰레기 객체를 제거하고, **Eden에서 살아남은 객체들을 Survivor 영역 중 하나로 이동**시킵니다. 이후 Eden을 깨끗이 비운 다음 애플리케이션은 다시 실행되며, 나중에 Eden이 다시 채워지면 또 Minor GC가 발생합니다. 이때 지난번에 사용했던 Survivor로 **객체들을 교차 복사**하여 생존 객체를 점진적으로 추려내는 **복사 수집**을 수행합니다. 자바에서는 Survivor 두 곳(S0, S1)을 번갈아 가며 사용하여, Minor GC 때마다 Eden+한쪽 Survivor의 살아남은 객체를 다른 Survivor로 복사하고 나머지는 청소하는 **semi-space 복사 알고리즘**을 구현하고 있습니다. 객체마다 **Age(age) 값**을 두어 몇 번 GC에서 살아남았는지 기록하고, **일정 age 이상 누적된 객체는 Old 세대로 승격**시킵니다.

 

Young Generation에서 발생하는 Minor GC는 **짧은 주기로 매우 자주** 일어나지만, 대부분 **소량의 객체만 생존**하고 대다수는 금방 회수되므로, Mark-Copy 복사 방식으로도 큰 부하 없이 빠르게 처리됩니다. 반면 **Old Generation**은 비교적 **드물게 GC**가 일어나는데, 이때 일어나는 GC를 **Major GC(메이저 GC)** 또는 **Full GC**라고 부릅니다. Old Gen에는 Young Gen에서 오래 살아남은 객체들이 모여 있기 때문에 **객체의 생존률이 높고** 힙 공간도 크습니다. 따라서 Major GC는 Minor GC에 비해 **훨씬 시간도 오래 걸리고** 애플리케이션 **일시 정지 시간도 길어질 수** 있습니다. 자바 GC는 이런 특성을 이용해, Old 영역에서는 **매번 압축까지 수행하지 않고** 우선 Mark-Sweep으로 빠르게 수거만 합니다가, **단편화가 누적되어 메모리 할당에 지장이 생길 때에만** Mark-Compact으로 **압축을 수행**합니다. 이렇게 하면 불필요한 객체 이동을 줄여 성능을 높이면서도, 가끔씩 압축을 통해 단편화 문제도 해결할 수 있습니다. (Major GC가 발생하면 Minor GC도 함께 수행되므로, Major GC를 Full GC라고 부르기도 합니다.)

> **용어 설명: Stop-The-World (STW)** – **Minor GC든 Major GC든, GC를 수행하는 동안에는 애플리케이션 스레드들이 멈춰야(STW)** 합니다. Minor GC는 짧게 끝나지만 Major GC는 길면 수백 ms~수 초까지 걸리기도 합니다. 이런 **GC 일시 정지 시간**은 응답성에 치명적일 수 있으므로, 최신 GC 알고리즘은 가능한 애플리케이션을 멈추지 않거나(STW 최소화) **동시에(concurrent)** 수집을 수행하는 방향으로 발전해왔습니다.

## 자바의 주요 가비지 컬렉터들

자바 HotSpot JVM에는 다양한 GC 알고리즘(Collector) 구현이 있으며, 각각 **특정 환경에 최적화**되어 있습니다. 아래에서는 자바의 주요 GC 유형들의 **동작 방식과 특징**을 살펴보고, **장단점과 사용 사례**를 정리합니다.

### 1. Serial GC (직렬 GC)

**Serial GC**는 가장 단순한 형태의 가비지 컬렉터로, **모든 GC 작업을 한 개의 쓰레드로 수행**합니다. Minor GC와 Major GC 모두 단일 스레드로 실행되고, GC 중에는 항상 **애플리케이션이 멈춥니다(STW)**. Serial GC는 **멀티코어를 활용하지 못하기 때문에** 현대 서버 환경에서는 잘 쓰이지 않지만, **JVM의 Client 모드 기본값**으로 작은 애플리케이션이나 임베디드/단일 코어 환경에서는 여전히 적합할 수 있습니다. 구현적으로 Serial GC는 **Young 영역은 복사 수집**, **Old 영역은 마크-컴팩트 수집**을 수행하여 단순하고 안전하게 메모리를 관리합니다. 메모리를 정리할 때 객체를 압축하여 **힙 최적화**를 하기 때문에, 적은 메모리에서도 잘 동작하는 편입니다.

- **장점**: GC 구현이 단순하고, **메모리 사용량이 작음**(GC용 추가 구조체나 멀티스레드 동기화 비용이 적음). 소규모 힙에서는 GC 일시 정지도 수 초 이내로 비교적 짧은 편.
    
- **단점**: GC 시 **애플리케이션 정지 시간이 김**. 멀티코어 환경에서도 1개의 GC 쓰레드만 사용하므로 현대 서버의 성능을 활용 못함. 대용량 힙이나 고성능 요구 상황에서는 부적합.
    
- **사용 사례**: **단일 CPU**나 **소용량 힙(몇백 MB 이하)**을 사용하는 애플리케이션. GC pause 시간이 크게 문제되지 않는 간단한 데스크톱 애플리케이션 등. 또한 임베디드 장치나 Android의 ART 같은 한정된 리소스 환경에서 사용되기도 함.
    

### 2. Parallel GC (병렬 GC)

**Parallel GC**는 일명 **Throughput GC(처리량 지향 GC)**라고도 불립니다. Serial GC와 알고리즘 구조는 비슷하지만, **Young Generation 수집을 여러 쓰레드로 병렬 수행**하여 GC 속도를 높입니다. 기본적으로 CPU 코어 수만큼 GC 스레드를 활용하며 (`-XX:ParallelGCThreads` 옵션으로 조절 가능), 자바 8까지 **Server 모드의 기본 GC**로 사용되었습니다 (JDK8까지 서버 JVM 기본값). Parallel GC는 **멀티코어에서 GC 처리량을 극대화**하는 데 유리하나, GC 실행 중에는 여전히 **STW**입니다. 자바 8 기준으로 Parallel GC에도 Old 영역을 병렬로 처리하는 옵션(`-XX:+UseParallelOldGC`)이 있어 **Young와 Old 모두 병렬처리** 및 **Old 영역 압축(compaction)**까지 수행할 수 있습니다. (기본 `UseParallelGC`인 경우 Young는 병렬이지만 Old는 Serial 방식 마크-컴팩트입니다.)

- **장점**: **멀티코어를 활용**하여 GC 처리 속도가 빠름. 동일한 STW라도 Serial보다 짧게 끝날 수 있고, **전체 애플리케이션 처리량을 높이는 데** 유리합니다. 튜닝 파라미터가 비교적 단순해서 기본 설정으로도 무난한 성능을 냅니다.
    
- **단점**: **Pause Time(정지 시간) 자체는 여전히 김** – GC 동안 모든 애플리케이션 쓰레드가 멈추므로, 응답 지연에 민감한 작업에는 부적합합니다. 또한 동시 수행 GC에 비하면 메모리 및 CPU 자원 공유 효율은 낮습니다.
    
- **사용 사례**: **처리량이 최우선**이고 0.51초 이상의 정지 시간도 용인될 수 있는 배치 작업, 오프라인 계산 작업 등에 적합. 예를 들어 데이터 마이그레이션, 대용량 리포트 생성, 일괄 청구서 처리 등 **일시 정지가 조금 길어도** 상관없는 백엔드 작업에 많이 사용됩니다. 힙 크기가 중간큰 수준이고 CPU 코어가 많은 환경에서 좋은 성능을 냅니다.
    

### 3. CMS (Concurrent Mark-Sweep) GC

**CMS(Concurrent Mark-Sweep) GC**는 자바 5~8 시대에 주로 쓰인 **낮은 지연(low pause) GC**입니다. 이름처럼 **Old Generation을 대상**으로 Mark-Sweep 알고리즘을 **애플리케이션 스레드와 동시(concurrently)** 수행하여, **장애가 적은 짧은 일시정지 GC**를 구현합니다. Young Generation 수집은 기본적으로 Parallel GC와 동일하게 처리하되, Old 영역 수집 시에 **가능한 애플리케이션을 멈추지 않고** 백그라운드에서 Mark 작업과 Sweep 작업을 진행합니다. CMS의 내부 단계는 **Initial Mark (초기표시)** -> **Concurrent Mark (동시표시)** -> **Remark (재표시)** -> **Concurrent Sweep (동시삭제)** 순으로 이루어집니다. Initial Mark 단계와 Remark 단계에서는 **짧은 STW 일시정지**가 발생하지만 (초기에 루트 객체들만 표시, 마지막에 동시 마크 중 놓친 변경분을 반영), 대부분의 마킹 작업과 삭제 작업은 애플리케이션 스레드와 병행하여 수행됩니다. 이로써 **GC Pause 시간을 짧게** 유지하는 대신, 애플리케이션이 실행되는 동안에도 일부 CPU를 GC가 차지하여 **전체 CPU 사용량은 증가**합니다. CMS는 **메모리 압축을 하지 않는(non-compacting)** **비이동(non-moving)** 수집기라서, 시간이 지날수록 Old 영역에 **단편화**가 누적될 수 있습니다. 단편화로 인해 큰 객체 할당이 어려워지거나 Old 영역이 가득 차면, 어쩔 수 없이 **Fallback으로 Stop-The-World Full GC**(Serial Mark-Compact)를 수행하게 되는데 이를 **Concurrent Mode Failure**라고 부릅니다. 이 문제가 자주 발생하면 힙을 넉넉히 늘리거나 CMS 메모리 기준 파라미터를 조정해야 합니다.

- **장점**: **짧은 GC 일시정지 시간** – 대부분의 작업을 동시에 처리하므로 애플리케이션 응답성을 크게 해치지 않음. 자바 8까지는 실무에서 **가장 널리 쓰인 저지연 GC**로 안정성과 검증이 많이 되었음.
    
- **단점**: **메모리 단편화** 문제가 있음 – 압축을 안 하므로 힙이 조각날 수 있고, 가끔 긴 Full GC를 야기. 또한 **추가 GC 스레드**가 돌아가므로 **CPU 부하**가 증가하고, 힙이 클 경우 동시 마크에도 시간이 오래 걸릴 수 있음. CMS는 자바 9부터 **Deprecated**되어 이후 JVM에서는 사용할 수 없으며(GC 교체를 위해 코드베이스 유지 중단), 최신 JDK에서는 제거되었습니다.
    
- **사용 사례**: (과거 시점) **UI, 대화형 서버, 저지연 서비스** 등 **일시정지 시간 최소화**가 중요한 애플리케이션. 예를 들어 웹 서버, 거래 시스템 등에서 한때 표준처럼 사용되었습니다. 현재는 CMS의 설계 철학을 계승한 G1 GC가 기본으로 대체되었습니다.
    

### 4. G1 (Garbage-First) GC

**G1 GC**는 Java 7에서 도입되고 Java 9부터 **JVM 기본 GC**가 된 최신 수집기입니다. G1의 목표는 **병렬적이고 점진적인 수집을 통해 GC 일시정지 시간을 예측 가능하고 짧게 유지**하는 것입니다. G1은 기존의 Young/Old 구분을 논리적으로 유지하면서, 힙 메모리를 **고정 크기의 Region(지역)** 단위로 세분화하여 관리합니다. 각 Region은 용도에 따라 Eden, Survivor, Old 등으로 **유연하게 할당**되며, 필요에 따라 일부 Old 영역 Region도 Young 수집 시 함께 회수하는 **Mixed GC**를 수행합니다. 이는 G1이 **heap 전체를 한 번에 비우지 않고**, **여러 작은 영역들 중 우선 순위가 높은(garbage-first)** 영역부터 회수하는 전략을 취하기 때문입니다. G1에서는 각 Region의 **잔여 가비지 양과 수집 비용을 모델링**하여, 사용자가 원하는 **목표 최대 pause 시간**(목표 STW 시간)에 맞춰 매번 몇 개의 Region을 회수할지 결정합니다. 예를 들어 `-XX:MaxGCPauseMillis` 옵션으로 200ms를 목표로 주면, G1은 과거 통계에 기반해 이 시간 내에 처리가능한 영역들만 부분 수거하여 일시정지를 짧게 끝냅니다.

 

G1의 내부 알고리즘은 **복사 수집 + 마크-컴팩트**를 조합한 형태입니다. Young 영역 수집시에는 역시 **객체를 새로운 Region으로 복사**하여 단편화를 방지하고, Old 영역도 일정 주기마다 **Concurrent Mark**를 통해 전체 힙의 live 객체를 표시한 뒤, 필요에 따라 **부분적으로 Region 단위 압축**을 시행합니다. G1은 **SATB (Snapshot-At-The-Beginning)**라는 쓰레기 객체 포착 기법을 사용하여, Concurrent Mark 중 애플리케이션이 객체 참조를 바꿔도 안전하게 수집을 진행합니다. Young 수집(복사)은 여전히 STW로 진행되지만, G1은 **여러 GC 쓰레드가 병렬로 객체 복사 및 Region 정리를 수행**하므로 멀티코어 효율을 살립니다. 또한 Old 영역의 마킹과 정리 대부분은 **동시 수행**되어, CMS처럼 긴 멈춤 없이도 힙 전반의 가비지를 식별합니다. G1은 **컴팩팅 수집기**이므로 CMS와 달리 **메모리 단편화가 누적되지 않고** 필요 시 Region 이동으로 해소됩니다. 다만 G1도 완전히 실시간(Real-time) GC는 아니어서, 목표한 일시정지 시간을 100% 보장하지는 못하며 힙이 매우 부족한 상황 등에서는 예상보다 긴 Stop-The-World Full GC가 발생할 수 있습니다.

- **장점**: **일정한 짧은 일시정지 시간** – 대용량 힙(수 GB 이상)에서도 짧게는 수십 ms 단위의 멈춤으로 GC를 완료할 확률이 높습니다. **메모리 압축 수행** – Region 단위로 항상 복사/압축하므로 힙이 단편화되지 않아 메모리 효율이 좋습니다. 또한 **GC 튜닝 옵션이 적어** 기본 설정으로도 양호한 성능을 내고, 별다른 조정 없이 대부분의 서버 애플리케이션에 **무난한 선택**입니다 (Java 9+ 기본값).
    
- **단점**: **처리량(Throughput)**은 동일 조건에서 Parallel GC보다 약간 낮을 수 있습니다. 짧은 pause를 위해 애플리케이션과 병행 처리 및 Region 관리 등 **추가 부하**가 있기 때문입니다. 또한 매우 짧은 응답 지연이 요구되는(예: 수 ms 이하) 시스템에서는 G1의 pause도 길게 느껴질 수 있습니다.
    
- **사용 사례**: **대부분의 자바 애플리케이션의 기본 GC**로 권장됩니다. 특히 **큰 힙 메모리(몇 GB~数十GB)**를 사용하고, **일정 수준 이하의 응답 지연**을 유지해야 하는 서버에 적합합니다. 이전에는 CMS 쓰던 곳은 대부분 G1으로 마이그레이션되었으며, GC 튜닝 경험이 없어도 무난한 선택지입니다. 반면, 일괄 처리 배치처럼 응답시간 상관없이 **최대 처리량을 쥐어짜는 상황**에서는 Parallel GC가 나은 경우도 있습니다.
    

> **심화:** _G1 vs CMS_ – G1은 CMS의 단점을 개선하기 위해 만들어졌습니다. 두 GC 모두 **낮은 일시정지**를 목표로 하지만, **G1은 힙을 region으로 분할**하고 **객체 이동(복사/압축)**을 통해 CMS의 단점인 **단편화 문제를 해결**했습니다. 또한 G1은 **pause 시간을 사용자가 제어**할 수 있고 예측 가능한 반면, CMS는 힙 상황에 따라 pause 변동이 컸습니다. 결과적으로 G1은 **CMS를 대체**하여 JDK9부터 기본 GC가 되었고, Java 14에서 CMS는 완전히 제거되었습니다.

### 5. ZGC (Z Garbage Collector)

**ZGC**는 JDK 11에서 실험적으로 도입되어 JDK 15부터 정식 지원된 **초저지연(ultra-low latency) GC**입니다. ZGC의 가장 큰 특징은 **어떤 힙 크기에서도 GC 일시정지 시간을 몇 밀리초 이내**로 유지하는 것입니다. 심지어 수백 GB~수 TB 규모의 힙에서도 **GC로 인한 stop-the-world를 10ms 이하**로 유지하는 것을 목표로 합니다. ZGC는 이를 위해 **모든 비용이 큰 작업을 애플리케이션 스레드와 동시에(concurrently) 수행**하고, 참조 색인 업데이트 등 최소한의 작업만 STW 구간에 처리합니다[docs.oracle.com](https://docs.oracle.com/en/java/javase/17/gctuning/z-garbage-collector.html#:~:text=The%20Z%20Garbage%20Collector%20,sizes%20from%208MB%20to%2016TB). 구체적으로, ZGC는 **Mark, Sweep, Compaction 모든 단계를 concurrent**하게 실행하며, 그 과정에서 객체 참조를 추적하기 위해 **Read Barrier(읽기 장벽)** 기술 등을 활용합니다. 또한 **Region 단위의 컬렉션**을 사용하나 G1과 달리 **Generational (세대별)** 구분이 없었는데, JDK 21에서 젊은 세대 개념이 추가된 **Generational ZGC**가 등장하여 성능을 더욱 향상시켰습니다.

 

ZGC는 기본적으로 **moving, compacting collector**이므로 메모리 단편화 문제가 없고, 커다란 객체(험ongous object)도 효율적으로 관리합니다. 힙이 성장하고 축소하는 것을 동적으로 감지하여 **미사용 메모리를 OS에 반환**하는 기능도 있어, 클라우드 환경 등에서 메모리 효율성을 높일 수 있습니다. ZGC의 내부 구현은 매우 복잡하지만, 요약하면 **GC 작업을 잘게 쪼개어 애플리케이션 실행과 섞어 수행**함으로써 **"GC로 인한 눈에 띄는 지연을 없애는"** 데 초점을 맞춥니다. 단, 이렇게 동시성 GC를 구현하려면 애플리케이션의 모든 객체 참조(load/store)에 추가 배리어 비용이 따르고, 또 GC 자체가 많은 스레드로 백그라운드 작업을 하므로, **CPU 사용률 증가** 및 **일부 throughput 저하**가 있을 수 있습니다. 그럼에도 특정 애플리케이션 분야(예: 초저지연 트레이딩 시스템 등)에서는 이 미미한 오버헤드보다 일시정지 시간 제거의 이점이 훨씬 큽니다.

- **장점**: **거의 실시간에 가까운 짧은 일시정지** – 수십 GB~TB급 메모리에서도 GC로 인한 멈춤이 몇 ms 수준으로 무시할만한 정도입니다. 힙 크기에 **pause time이 비례하지 않아** 메모리를 크게 늘려도 지연이 증가하지 않습니다[docs.oracle.com](https://docs.oracle.com/en/java/javase/17/gctuning/z-garbage-collector.html#:~:text=The%20Z%20Garbage%20Collector%20,sizes%20from%208MB%20to%2016TB). 메모리 단편화도 효과적으로 해결하고, 필요 시 미사용 힙을 운영체제에 반환하여 **메모리 효율**도 높입니다[docs.oracle.com](https://docs.oracle.com/en/java/javase/17/gctuning/z-garbage-collector.html#:~:text=The%20Z%20Garbage%20Collector%20,sizes%20from%208MB%20to%2016TB).
    
- **단점**: **상대적으로 새로운 기술**로서, 아주 넓은 범위에서 안정성이 검증된 것은 아닙니다(JDK 17 이상 LTS 버전에서 많이 개선됨). 또한 **CPU와 메모리 오버헤드**가 다소 있고, 절대 처리량은 G1 등에 비해 약간 떨어질 수 있습니다.
    
- **사용 사례**: **일시정지 시간에 극도로 민감한 서비스**에 적합합니다. 예를 들어, 수밀리초의 지연도 치명적인 고주파 매매 시스템, 실시간 이벤트 처리 시스템, 대용량 메모리 캐시 서버 등이 고려 가능합니다. 또한 **수백 GB 이상의 초대형 힙**을 다루는 Java 애플리케이션(예: 빅데이터 처리)에서도 ZGC는 관리 효율과 짧은 pause로 유용합니다. 아직까지는 G1이 기본인 경우가 많지만, 최신 JDK를 도입할 수 있고 저지연 요구가 있다면 ZGC를 시도해볼 만합니다.
    

> **심화:** _Shenandoah GC_ – ZGC와 유사하게 **초저지연**을 목표로 한 GC로 **Shenandoah (셔넌도어) GC**도 있습니다. OpenJDK의 RedHat 팀이 주도하여 개발한 Shenandoah는 JDK 12부터 옵션으로 포함되었으며, **클래스 로드 배리어 방식**으로 STW 시간을 최소화합니다. ZGC와 큰 목표는 같지만 내부 구현에 차이가 있으며, Shenandoah 역시 **heap 크기에 상관없이 (pause time이 거의 고정)**라는 슬로건을 가집니다. 둘 다 현재 활발히 발전 중인 최신 GC이며, JDK 17 기준으로 ZGC가 통합되어 일반적으로 Shenandoah보다 범용적으로 쓰이고 있습니다.

## 다양한 GC의 비교: 장단점과 활용 요약

위에서 설명한 자바 GC들을 **요약 비교**하면 다음과 같습니다 (JDK 17 기준):

|GC 유형|수집 방식|STW 일시정지|장점|단점|적합한 용도|
|---|---|---|---|---|---|
|**Serial GC**|Mark-Copy (Young), Mark-Compact (Old)|**모든 GC STW**, 단일 스레드 처리|구현 단순, 메모리 소비 적음. 소형 앱에서 충분히 빠름.|일시정지 길고 멀티코어 활용 못함.|단일 코어 or **소규모 힙** 앱.|
|**Parallel GC**|Mark-Copy (Young), Mark-Compact (Old, 병렬)|**모든 GC STW**, 멀티스레드 병렬 처리|멀티코어 활용으로 **처리량 최고**. 대용량 힙도 짧은 시간에 수집.|일시정지 동안 응답 지연 큼. 실시간성 부족.|**처리량 중시** 배치, 오프라인 작업.|
|**CMS (Conc. Mark-Sweep)**|Mark-Sweep (Old, 동시), Copy (Young)|**Young GC STW**; **Old GC 대부분 동시**, 초기/최종 마크만 짧은 STW|**짧은 pause**로 응답성 우수. 검증된 안정성 (JDK8까지 사용).|메모리 **단편화** 발생. Concurrent mode failure 시 Full GC 위험. 추가 CPU 부담.|(과거) **저지연 서비스**(웹/GUI). JDK9부터 G1로 대체.|
|**G1 (Garbage-First)**|Region 단위 Mark-Copy + Compact (병렬/동시)|**Young GC STW (병렬)**; **Old GC 동시 + 부분 STW**|**Pause 시간 짧고 예측 가능**. 메모리 압축, 단편화 적음. 기본 GC로 범용적 성능.|Parallel 대비 약간 **처리량 저하** 가능. 매우 짧은 pause 요구에는 부족할 수 있음.|**대부분 애플리케이션의 기본 선택**, 대용량 힙 서버.|
|**ZGC**|Region 기반 Conc. Mark-Compact (최소 STW)[docs.oracle.com](https://docs.oracle.com/en/java/javase/17/gctuning/z-garbage-collector.html#:~:text=The%20Z%20Garbage%20Collector%20,sizes%20from%208MB%20to%2016TB)|**거의 전체 동시**; <1~10ms의 매우 짧은 STW|**초저지연**(pause 수 ms). 힙 크기 영향 거의 없음[docs.oracle.com](https://docs.oracle.com/en/java/javase/17/gctuning/z-garbage-collector.html#:~:text=The%20Z%20Garbage%20Collector%20,sizes%20from%208MB%20to%2016TB). 메모리 효율 좋음.|최신 기법으로 **검증 적음**. 약간의 CPU/메모리 오버헤드.|**실시간 응답 시스템**, 초대형 힙 활용 앱. (JDK17+에서 사용 가능)|

## 다른 언어의 메모리 관리 방식과의 비교

자바의 GC와 타 언어들의 메모리 관리 전략을 비교해보겠습니다. 아래 표는 몇 가지 대표 언어와의 차이를 요약한 것입니다:

|언어|메모리 관리 방식|특징 요약|
|---|---|---|
|**C/C++**|**수동 관리 (Unmanaged)**|개발자가 `malloc/free` 혹은 RAII 패턴으로 직접 관리. 가비지 컬렉터가 없어 오버헤드 거의 없지만, 잘못 사용할 경우 **메모리 누수** 등의 위험. C++ 스마트 포인터(`std::shared_ptr`)는 **참조 카운팅**으로 일부 자동 해제 보조.|
|**Java (자바)**|**자동 GC (Managed)**|JVM이 **세대별 추적 GC**로 자동 메모리 관리. 개발 편의성이 높고 **메모리 누수 방지**에 유리. 단, **GC 일시정지**로 인한 지연과 **GC 튜닝**이 필요할 수 있음.|
|**Python**|**자동 관리 (참조 카운팅 + GC)**|**참조 카운트** 기반으로 객체를 즉시 해제하고, 순환 참조 객체는 **별도 주기적 GC**로 수집. 참조 변경 시마다 카운트 갱신하므로 오버헤드 약간 있고, 성능은 tracing GC보다 낮을 수 있으나 **일시정지 분산**(한 번에 몰아서 안 함)이라는 장점. 또 참조 카운팅으로 인해 **객체 소멸 시점이 결정적**이라 파일/리소스 해제 등의 부가 효과가 즉시 발생하는 점이 있음.|
|**Go (Golang)**|**자동 GC (비세대 병행 GC)**|**Concurrent Mark & Sweep** GC 사용. **Generational (세대별)** 구조가 없고, **비이동(non-compacting)** 수집기라 단편화는 메모리 할당기 설계로 완화. 쓰레기 수집을 애플리케이션과 동시 실행하여 **짧은 지연 시간**을 목표로 함. **write barrier** 기반으로 객체 변경을 추적하며, 세대 구분이 없는 대신 Go 런타임의 **escape analysis**가 단기 객체를 스택에 할당해 GC 부담을 줄임. 전반적으로 **낮은 지연, 적당한 처리량**으로, 개발자가 GC 튜닝보다는 Go 특유의 메모리 관리 패턴(예: object pool 지양, slice/shared 메모리 활용 등)에 익숙해지면 됨.|
|**C# / .NET**|**자동 GC (Managed)**|**자바와 유사하게 세대별 추적 GC**를 사용. 0,1,2 총 **3세대**로 나누어 짧은 수명 객체는 자주 수집하고, 장수 객체는 2세대에 남겨 가끔만 수집[dileepsreepathi.medium.com](https://dileepsreepathi.medium.com/senior-net-dev-interview-q-a-5ec71e03ced1#:~:text=The%20C,based%20on%20their%20lifespan). **Mark-Sweep-Compact** 단계를 거치며 힙 단편화 방지를 위해 **압축 이동** 수행[dileepsreepathi.medium.com](https://dileepsreepathi.medium.com/senior-net-dev-interview-q-a-5ec71e03ced1#:~:text=,memory%20allocation%20for%20future%20objects). Large Object Heap 등 특수 영역 관리, Span<T> 같은 값 타입을 활용한 할당 줄이기 등 .NET 최적화 기법이 있음. 전반적인 GC 동작 원리는 Java와 매우 비슷하며, 튜닝 옵션도 비슷하게 제공됨.|

_비고_: 이 밖에 **JavaScript**(V8 등)도 세대별 mark-sweep/compact GC를 사용하고, **Rust**는 GC 없이 철저한 컴파일타임 소유권 검사로 메모리를 관리합니다. 현대 언어들은 각기 다른 방식으로 **메모리 안전성과 성능의 균형**을 추구하고 있으며, **가비지 컬렉션**은 그중 자동 메모리 관리를 구현하는 주요한 방법입니다.

## 결론

이상으로 **자바 가비지 컬렉터의 전반적인 동작 원리와 다양한 구현들의 특성**을 살펴보았습니다. GC는 개발자의 부담을 덜어주는 고마운 존재이지만, 때때로 내부 동작을 이해하고 조율해야 하는 복잡한 주제이기도 합니다. **GC 튜닝**은 애플리케이션의 성격에 맞는 적절한 수집기를 선택하고, 힙 크기나 pause time 목표 등을 조정하여 **응답 시간과 처리량의 균형점을 찾는 과정**입니다. 예컨대 실시간성이 중요하면 **G1 또는 ZGC**를, 처리량 극대화가 목표면 **Parallel GC**를, 메모리가 작으면 **Serial GC**를 고려하는 식입니다. 또한 GC 로그와 모니터링 도구(예: VisualVM, JFR 등)를 통해 **메모리 사용 패턴과 GC 빈도**를 분석함으로써, 메모리 누수를 발견하거나 GC 동작을 최적화할 수 있습니다.

 

마지막으로, 자동 메모리 관리의 이점과 함께 **한 번 쯤은 메모리를 수동으로 관리해야 했던 시대**를 떠올려 보면, 현재의 GC 기술이 얼마나 발전했는지 실감할 수 있습니다. 자바를 비롯한 현대 언어들의 가비지 컬렉터는 수십 년 간의 연구를 통해 **점점 똑똑하고 빠르게** 진화해왔습니다. 개발자는 GC를 완전히 통제할 수는 없지만, **GC 동작 원리와 튜닝 기법을 이해**함으로써 메모리 관리라는 난제를 더욱 효과적으로 다룰 수 있을 것입니다.

 

**참고한 자료:** 자바 공식 튜닝 가이드, 오라클 GC 기초 튜토리얼, Line Engineering 블로그 (Go GC 분석), Stack Overflow Q&A 등.

인용

[

![](https://www.google.com/s2/favicons?domain=https://happybplus.tistory.com&sz=32)

[백엔드 기술 세미나] Garbage Collector — 행복한 B+

https://happybplus.tistory.com/1043

](https://happybplus.tistory.com/1043#:~:text=%EA%B0%9D%EC%B2%B4%EB%A5%BC%20%EA%B0%80%EB%A6%AC%ED%82%A4%EB%8A%94%20%EC%B0%B8%EC%A1%B0%20%EC%B9%B4%EC%9A%B4%ED%84%B0%20%EB%A5%BC,%EC%B6%94%EA%B0%80%ED%95%98%EA%B3%A0)[

![](https://www.google.com/s2/favicons?domain=https://happybplus.tistory.com&sz=32)

[백엔드 기술 세미나] Garbage Collector — 행복한 B+

https://happybplus.tistory.com/1043

](https://happybplus.tistory.com/1043#:~:text=%EB%A8%BC%EC%A0%80%20%ED%9A%8C%EC%88%98%ED%95%A0%20%EA%B0%9D%EC%B2%B4%EB%93%A4%EC%9D%84%20%EB%AA%A8%EB%91%90%20%ED%91%9C%EC%8B%9C%ED%95%9C,%EB%8B%A4%EC%9D%8C)[

![](https://www.google.com/s2/favicons?domain=https://happybplus.tistory.com&sz=32)

[백엔드 기술 세미나] Garbage Collector — 행복한 B+

https://happybplus.tistory.com/1043

](https://happybplus.tistory.com/1043#:~:text=Image)[

![](https://www.google.com/s2/favicons?domain=https://docs.oracle.com&sz=32)

The Z Garbage Collector

https://docs.oracle.com/en/java/javase/17/gctuning/z-garbage-collector.html

](https://docs.oracle.com/en/java/javase/17/gctuning/z-garbage-collector.html#:~:text=The%20Z%20Garbage%20Collector%20,sizes%20from%208MB%20to%2016TB)[

![](https://www.google.com/s2/favicons?domain=https://dileepsreepathi.medium.com&sz=32)

Senior .NET Dev Interview Q/A: I. C# Garbage Collector: How is it… | by Dileep Sreepathi | Medium

https://dileepsreepathi.medium.com/senior-net-dev-interview-q-a-5ec71e03ced1

](https://dileepsreepathi.medium.com/senior-net-dev-interview-q-a-5ec71e03ced1#:~:text=The%20C,based%20on%20their%20lifespan)[

![](https://www.google.com/s2/favicons?domain=https://dileepsreepathi.medium.com&sz=32)

Senior .NET Dev Interview Q/A: I. C# Garbage Collector: How is it… | by Dileep Sreepathi | Medium

https://dileepsreepathi.medium.com/senior-net-dev-interview-q-a-5ec71e03ced1

](https://dileepsreepathi.medium.com/senior-net-dev-interview-q-a-5ec71e03ced1#:~:text=,memory%20allocation%20for%20future%20objects)