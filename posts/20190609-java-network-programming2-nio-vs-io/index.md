# Java Network Programming(2) - NIO vs IO

### 설명

- IO

    Java IO는 데이터를 파일, 네트워크, 메모리 등의 data input, output를 stream를 통해 도와주는 API이다.

- NIO

    New IO의 약자로 IO와 비슷하지만 stream이 아닌 buffer를 통해 input, output를 도와주는 API이다.

|IO      |NIO         |
|--------|------------|
|Stream  |Buffer      |
|Blocking|Non-blocking|
|        |Selectors   |


### Stream vs Buffer

- Stream

    캐시를 거치지 않고 한번에 하나 이상의 byte를 읽을 수 있다. 즉각적으로 읽고 쓰기 때문에 에러 출력이나 반응 속도를 중요시하는 키보드 입력, 화면 출력 프로세스에서 사용하기 좋다.

- Buffer

    캐시 역할을 하기 때문에 읽는 개체와 쓰는 개체를 비동기로 운영할 수 있다. 하지만 Buffer에 들어간 데이터를 즉각적으로 읽고, 쓰지 않기 때문에 프로세스가 비정상적으로 종료된다면 Buffer안의 데이터는 무효화된다.

### Blocking vs Non-blocking

- Java IO

    Stream 읽거나 쓸때 blocking된다. 즉, 읽을 때 읽을 데이터가 있을 때까지 멈춰있는다. 쓸때는 모든 데이터를 기록할 때까지 멈춰있는다. 그렇기 때문에 blocking된 Stream를 사용하면 병목현상이 일어날 수 있다.

- Java NIO

    Buffer Non-blocking를 설정할 수 있기 때문에 읽거나 쓰면 현재 상태를 그대로 반환한다.

### Selectors

Java NIO의 selectors는 한 개의 thread로 여러 개의 channel을 다룰 수 있다. 여러개의 channel를 selectors에 등록해놓고 readable, writeable 상태의 channel를 반납하므로써 여러 channel를 다룰 수 있다.

### 성능

[참고](http://eincs.com/2009/08/java-nio-bytebuffer-channel-file/)
