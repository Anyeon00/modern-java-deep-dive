

# Stream
>데이터를 담고 있지 않다.



스트림은 데이터 처리 작업이 주요 목적이다.

데이터를 저장하지 않고 그저 어떻게 다룰지를 논하는 것이다.



스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다.



스트림 파이프라인은 이 원소들로 수행하는 **연산 단계**를 표현하는 개념이다.





Stream 의 단순한 문법보단 중요한 개념과 주의점에 대해 살펴보자





## 스트림과 스트림 파이프라인
스트림 파이프라인은 소스 스트림에서 시작해 종단 연산으로 끝나며, 그 사이에 하나 이상의 중간 연산이 있을 수 있다.



단어에 대해 간단히 표현하자면

중간 연산 : 스트림을 반환하는 연산

종단 연산 : 스트림을 반환하지 않는 연산 이다.



예를 들어, filter, map 등등은 중간 연산이고 collect, toArray, forEach 등등은 종단 연산이다.

(forEach 연산은 스트림 계산 결과를 보고할 때만 사용하자)





>스트림 파이프 라인은 lazy 하다.

=> 종단 연산이 호출 될 때 중간 연산이 이루어지며, 종단 연산에 사용되는 데이터만 연산이 일어난다.

중간 연산마다 연산의 파이프라인을 리턴한다



많은 사람들이 스트림으로 병렬처리를 한다고 얘기를 하지만, 기본적으로 스트림 파이프라인은 순차적이다.

병렬로 실행하기 위해서는 종단 연산 이전에 parallel 메서드를 호출하여야 한다.





### 내부 반복자
Stream 은 내부 반복자이므로 처리 속도가 빠르고 병렬 처리에 효율적이다.



for문과 iterator 는 컬렉션의 요소를 컬렉션 바깥쪽으로 가져와서 반복한다 -> 외부 반복자



하지만 스트림은 처리 방법을 컬렉션 내부로 주입시켜서 처리하기에 컬렉션에서 데이터를 꺼내는 비용이 절감된다.





**라고는 하지만...**





50만개의 int 를 저장하는 배열을 만들고 가장 큰 원소를 찾는 시간을 측정하였을 시,

>for-loop 이 순차 스트림의 15배 가량 빠른 연산이 이루어졌다.

(스트림이 비교적 최신 기술이라 컴파일러 최적화의 문제라고 한다..)

(연산이 크거나 복잡하면 그 간격이 점점 좁혀지긴 한다)



## 주의 사항
### 가독성 측면
람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.



연산에 적절한 이름을 지어주고 세부 구현을 주 프로그램 로직 밖으로 빼내 전체적인 가독성을 높이는 방법은 스트림 파이프라인에서 특히 중요하다!!



또한, 스트림을 정말 다양하게 적용할 수 있지만 그것이 필수는 아니다.

스트림을 과하게 적용하는 경우 가독성과 유지보수성은 처참해진다. 물론 적당히 사용하면 가독성이 좋아진다.



즉, 스트림과 반복 중 무엇을 사용해야 하는지 고민이 든다면, 둘 다 해보고 더 읽기 좋은 것을 선택하는게 최선이다.





#### 람다

스트림을 이용하는 경우 람다를 자주 사용하게 된다. 따라서 람다를 이상하게 쓰면 스트림과 함께 가독성이 처참해진다.



람다가 가질 수 있는 문제들을 살펴보자.



익명 클래스를 사용하면 코드가 너무 길어지기 때문에 람다를 사용하여 간결하게 표현하는게 더 낫지만 항상 그렇지만은 않다.



람다는 이름이 없고 문서화도 못하기에 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다 자체를 쓰지 말아야 한다.

람다는 세줄이 넘어가면 가독성이 심하게 나빠진다. 이러한 경우 차라리 람다를 쓰지 않는 것이 더 나을 수 있다.



### 스트림 병렬화 측면
데이터 소스가 Stream.iterate 거나 중간 연산으로 limit 을 사용하면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.



대체로 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap 인스턴스거나 배열, int, long 범위 일 때 병렬화의 효과가 가장 좋다.



이 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 일을 다수의 스레드에 분배하기 좋다.

또한, 원소들을 순차적으로 실행할 때의 참조 지역성이 뛰어나다.

-> 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 것이다.



참조 지역성이 낮으면 스레드는 데이터가 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리는데 시간을 허비한다.

=> 다량의 데이터 처리하는 연산을 병렬화할 때 아주 중요한 요소.

=> 기본 타입의 배열이 참조 지역성이 가장 뛰어난 자료구조 (데이터 자체가 메모리에 연속되어 저장)





>성능 향상 추정 방법

(스트림 안의 원소 수 x 원소당 수행되는 코드 줄 수) 가 최소 수십만은 되어야 성능 향상을 기대할 수 있다.



스레드 풀 생성, 컨텍스트 스위칭등을 고려해보면 당연히 데이터 양이 방대해야 어느정도 효과를 볼 것임을 예상할 수 있다.



스트림 병렬화는 오직 성능 최적화 수단임을 기억해야 한다. 변경 전후로 반드시 성능을 테스트 하고 적용해야 한다.

계산이 정확하게 수행되고 성능이 빨라질 것이라는 확신이 없다면 시도하지 않는 것이 더 나은 판단이다.





안타깝게도 생각보다 우리가 스트림으로 병렬화를 쓸 일이 거의 없다





출처

Effecttive Java 3/E

이것이 자바다

https://sigridjin.medium.com/java-stream-api%EB%8A%94-%EC%99%9C-for-loop%EB%B3%B4%EB%8B%A4-%EB%8A%90%EB%A6%B4%EA%B9%8C-50dec4b9974b