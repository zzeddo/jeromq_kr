# Artimesia(Sook) 예제

jeroMQ은 ØMQ 라이브러리 기반으로 java 개발 언어를 사용하여 서비스를 구축할 수 있는 프레임워크를 제공합니다. 본 장에서 ØMQ 가이드의 예제를 구현하도록 하겠습니다.

# 1장-기본

프로그래밍은 예술로 치장된 과학이지만 우리 대부분은 소프트웨어의 물리학을 거의 이해하지 못하고 있으며 과학으로 가르치는 곳이 거의 없습니다. 소프트웨어의 물리학은 알고리즘이나, 자료 구조, 프로그래밍 언어나 추상화 등이 아닙니다. 이런 것들은 다만 우리가 그냥 만들어내고 사용하고 버리는 도구들일 뿐입니다. 소프트웨어의 진정한 물리학은 인간의 물리학입니다. 특히 복잡성 앞에서의 인간의 한계와 우리의 협력을 통하여 거대한 문제를 작은 조각들로 쪼개서 해결하려고 하는 욕구가 있습니다. 이것이 프로그래밍 과학입니다 : 사람들이 쉽게 이해하고 사용할 수 있는 구성요소들을 만들어냄으로써, 사람들이 그걸 이용하여 거대한 문제들을 해결하게 도와줍니다.

우리는 연결된 세상 속에 살고 있습니다. 그리고 현대의 소프트웨어들은 이런 세상 속을 항해할 수 있습니다. 따라서 미래의 거대한 솔루션들을 위한 구성요소들은 반드시 서로 연결되어야 하며 대규모로 병렬적이어야 합니다. 더 이상 프로그램이 그냥 “강력하고 침묵” 하기만 하면 되던 그런 시대는 지났습니다. 코드는 이젠 코드와 대화해야 합니다. 코드는 수다스러워야 하고 사교적이어야 하며 연결되어야 합니다. 코드는 반드시 인간의 두뇌처럼 작동해야 하고, 수조 개의 뉴런들이 서로에게 메시지를 던지듯이, 중앙 통제가 없는 대규모 병렬 네트워크로 단일 장애점 없어야 합니다. 그러면서도 극도로 어려운 문제들을 해결할 수 있어야 합니다. 그리고 코드의 미래가 인간의 두뇌처럼 보인다는 것은 우연이 아닙니다. 왜냐면 결국 네트워크도 단말에서 인간의 두뇌에 연결되며, 그 어떤 네트워크의 진화의 종착점도 인간의 두뇌이기 때문입니다.

## 물으면 얻을 것이다.

Hello World 예제로 시작하며, 클라이언트와 서버 프로그램을 만들도록 하겠습니다. 클라이언트에서 서버로 “Hello”을 보내면 서버에서 “World”를 클라이언트에게 응답합니다. 예제에서 ØMQ 소켓을 5555 포트로 오픈하여 요청에 대응합니다.

### hwserver.java: Hello World 서버

```java
package guide;

//
//  Hello World server in Java
//  Binds REP socket to tcp://*:5555
//  Expects "Hello" from client, replies with "World"
//

import org.zeromq.SocketType;
import org.zeromq.ZMQ;
import org.zeromq.ZContext;

public class hwserver
{
    public static void main(String[] args) throws Exception
    {
        try (ZContext context = new ZContext()) {
            // Socket to talk to clients
            ZMQ.Socket socket = context.createSocket(SocketType.REP);
            socket.bind("tcp://*:5555");

            while (!Thread.currentThread().isInterrupted()) {
                byte[] reply = socket.recv(0);
                System.out.println(
                    "Received " + ": [" + new String(reply, ZMQ.CHARSET) + "]"
                );

                String response = "world";
                socket.send(response.getBytes(ZMQ.CHARSET), 0);

                Thread.sleep(1000); //  Do some 'work'
            }
        }
    }
}
```

### hwclient.java: Hello World 클라이언트

```java
package guide;

//
//  Hello World client in Java
//  Connects REQ socket to tcp://localhost:5555
//  Sends "Hello" to server, expects "World" back
//

import org.zeromq.SocketType;
import org.zeromq.ZMQ;
import org.zeromq.ZContext;

public class hwclient
{
    public static void main(String[] args)
    {
        try (ZContext context = new ZContext()) {
            //  Socket to talk to server
            System.out.println("Connecting to hello world server");

            ZMQ.Socket socket = context.createSocket(SocketType.REQ);
            socket.connect("tcp://localhost:5555");

            for (int requestNbr = 0; requestNbr != 10; requestNbr++) {
                String request = "Hello";
                System.out.println("Sending Hello " + requestNbr);
                socket.send(request.getBytes(ZMQ.CHARSET), 0);

                byte[] reply = socket.recv(0);
                System.out.println(
                    "Received " + new String(reply, ZMQ.CHARSET) 
                );
            }
        }
    }
}
```

### 빌드 및 테스트

~~~{.bash}
PS C:\artemisia\jeromq> javac guide/hwserver.java        
PS C:\artemisia\jeromq> javac guide/hwclient.java
PS C:\artemisia\jeromq> java guide.hwserver
Received : [Hello]
Received : [Hello]
Received : [Hello]
Received : [Hello]
Received : [Hello]
Received : [Hello]
Received : [Hello]
Received : [Hello]
Received : [Hello]
Received : [Hello]

C:\artemisia\jeromq>java guide.hwclient
Connecting to hello world server
Sending Hello 0
Received world
Sending Hello 1
Received world
Sending Hello 2
Received world
Sending Hello 3
Received world
Sending Hello 4
Received world
Sending Hello 5
Received world
Sending Hello 6
Received world
Sending Hello 7
Received world
Sending Hello 8
Received world
Sending Hello 9
Received world
~~~

## ØMQ 버전 확인하기

ØMQ는 자주 버전이 변경되며 만약 문제가 있다면 다음 버전에서 해결될 수도 있습니다. ØMQ 버전 확인하는 방법은 다음과 같습니다.

### version.java : ØMQ 버전 확인하기

```java
package guide;

import org.zeromq.ZMQ;

//  Report 0MQ version
public class version
{
    public static void main(String[] args)
    {
        String version = ZMQ.getVersionString();
        int fullVersion = ZMQ.getFullVersion();

        System.out.println(
            String.format(
                "Version string: %s, Version int: %d", version, fullVersion
            )
        );
    }
}
```

### 빌드 및 테스트

~~~{.bash}
PS C:\artemisia\jeromq> javac guide/version.java 
PS C:\artemisia\jeromq> java guide.version      
Version string: 4.1.7, Version int: 40107
~~~

## 메시지 송신하기

두 번째 고전적인 패턴으로 서버의 단방향 데이터 전송으로, 서버가 일련의 클라이언트들에게 변경정보들을 배포합니다. 예를 들면 우편번호, 온도, 상대습도 등으로 구성된 기상 변경정보들이 배포하여 실제 기상관측소처럼 임의의 값을 생성하도록 하였습니다.

변경정보의 전송은 시작도 끝이 없이 반복되며, 마치 끝도 없는 방송(Broadcast) 같습니다. 클라이언트 응용프로그램은 서버에서 발행되는 데이터를 들으며 특정 우편번호에 대한 것만 수신합니다. 기본적으로 뉴욕시의 우편번호(10001)가 설정되었지만 매개변수로 변경이 가능합니다.

### wuserver.java: 기상 정보 변경 서버

```java
//  Weather update server
//  Binds PUB socket to tcp://*:5556
//  Publishes random weather updates

#include <szmq/szmq.h>
#include <cstdlib>
#include <boost/format.hpp>
#include <czmq.h>
using namespace std;

int main (void)
{
    //  Prepare our context and publisher
    szmq::Context context;
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(context);
    publisher.bind(szmq::SocketUrl("tcp://*:5556"));

    //  Initialize random number generator
    //std::srand (static_cast<unsigned int>(std::time(0)));
    srand((unsigned) time(NULL));
    while (1) {
        //  Get values that will fool the boss
        int zipcode, temperature, relhumidity;
        zipcode     = rand() % 100000;
        temperature = rand() % 215 - 80;
        relhumidity = rand() %50 + 10;
        //  Send message to all subscribers
        char update[20];
        snprintf (update, 20 ,	"%05d %d %d", zipcode, temperature, relhumidity);
        publisher.send(szmq::Message::from(update));
    }
    publisher.close();
    return 0;
}
```

이론적으로는 클라이언트, 서버에서 누가 bind()를 하던 connect()를 하는지는 문제가 되지 않지만 실제로는 PUB 소켓일 경우 bind(), SUB 소켓일 경우 connect()를 수행합니다.

### wuclient.java: 기상 변경 클라이언트

```java
//  Weather update client
//  Connects SUB socket to tcp://localhost:5556
//  Collects weather updates and finds avg temp in zipcode

#include <szmq/szmq.h>
#include <sstream>
#include <boost/format.hpp>
using namespace std;

int main (int argc, char *argv [])
{
    //  Socket to talk to server
    std::cout << "Collecting updates from weather server...\n";
    szmq::Context context;
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(context);

    subscriber.connect(szmq::SocketUrl("tcp://localhost:5556"));
    
    //  Subscribe to zipcode, default is NYC, 10001
    const char *filter = (argc > 1)? argv [1]: "10001 ";
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, filter, strlen(filter));

    //  Process 100 updates
    int update_nbr;
    long total_temp = 0;
    for (update_nbr = 0; update_nbr < 100; update_nbr++) {
        auto buffer = subscriber.recv1().read<std::string>();
        int zipcode, temperature, relhumidity;
        std::istringstream iss(buffer);
		iss >> zipcode >> temperature >> relhumidity ;
        total_temp += temperature;
    }
    std::cout << boost::format("Average temperature for zipcode %1% was %2%F\n") % filter % (int)(total_temp/update_nbr);
    subscriber.close();
    return 0;
}
```

PUB-SUB 소켓에 대한 한 가지 중요한 사항으로 구독자가 메시지를 받기 시작하는 시기를 정확히 알 수 없습니다. 구독자를 기동하고 기다리는 동안 발행자를 기동 하면, 구독자는 항상 발행자가 보내는 첫 번째 메시지를 유실힙니다. 사유는 구독자가 발행자에게 연결할 때(작지면 0이 아닌 시간 소요) 발행자가 이미 메시지를 전송했기 때문입니다.

TCP 연결 생성은 네트워크와 연결 대상들 간의 경유 단계(hops)에 따라 몇 밀리초(milliseconds) 지연을 발생시키며, 그 시간 동안 ØMQ는 많은 메시지를 보낼 수 있습니다. 편의상 연결 설정에 5밀리초가 소요되고, 발행자가 초당 100만 메시지를 송신할 수 있다면 발행자와 연결에 필요한 5밀리초 사이에 5000개 메시지를 전송할 수 있습니다.

정리하면, 클라이언트는 선택한 우편번호를 구독하고 해당 우편번호에 대한 100개 변경정보들을 수집합니다. 우편번호가 무작위로 분포하는 경우에는 약 1천만 변경정보가 발생합니다. 클라이언트를 시작한 후 서버를 시작하면 클라이언트는 문제없이 작동합니다. 서버를 중지하고 재시작해도 클라이언트는 계속 작동합니다. 클라이언트가 100의 변경정보들을 수집하면 평균을 계산하여 화면에 출력하고 종료합니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples>  cl -EHsc wuserver.java szmq.lib
PS D:\work\sook\src\szmq\examples>  cl -EHsc wuclient.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./wuserver

PS D:\work\sook\src\szmq\examples> ./wuclient
Collecting updates from weather server...
Average temperature for zipcode 10001  was 26F
~~~

## 나누어서 정복하라

마지막 예는 작은 슈퍼 컴퓨터를 만들어 보겠으며, 슈퍼 컴퓨팅 응용프로그램은 전형적인 병렬 처리 모델입니다.

* 호흡기(ventilator)는 병렬 처리 가능한 작업들을 생성합니다.
* 일련의 작업자(Worker)들이 작업들을 처리합니다.
* 수집기(Sink)는 작업자 프로세스들로부터 작업 결과를 수집합니다.

실제로 작업자는 초고속 컴퓨터들에서 병렬 처리로 실행되며 GPU(그래픽 처리 장치)를 사용하여 어려운 연산을 수행합니다. 호흡기 소스는 100개의 작업할 메시지들을 만들며 각 메시지에는 작업자의 부하를 시간으로 전달(<100 msec)하여 해당 시간 동안 대기하게 합니다.

### taskvent.java: 호흡기

```java
//  Task ventilator
//  Binds PUSH socket to tcp://localhost:5557
//  Sends batch of tasks to workers via that socket

#include <szmq/szmq.h>
#include <cstdlib>
#include <iostream>
#include <chrono>
#include <boost/format.hpp>
using namespace std;
int main (void) 
{
    szmq::Context context;
    szmq::Socket<ZMQ_PUSH, szmq::ZMQ_SERVER> sender(context);
    sender.bind(szmq::SocketUrl("tcp://*:5557"));

    //  Socket to send start of batch message on
    szmq::Socket<ZMQ_PUSH, szmq::ZMQ_CLIENT> sink(context);
    sink.connect(szmq::SocketUrl("tcp://localhost:5558"));

    cout << "Press Enter when the workers are ready: ";
    getchar ();
    cout <<"Sending tasks to workers...\n";

    //  The first message is "0" and signals start of batch
    sink.sendOne(szmq::Message::from("0"));

    //  Initialize random number generator
    std::srand (static_cast<unsigned int>(std::time(0)));

    //  Send 100 tasks
    int task_nbr;
    int total_msec = 0;     //  Total expected cost in msecs
    for (task_nbr = 0; task_nbr < 100; task_nbr++) {
        int workload;
        //  Random workload from 1 to 100msecs
        workload = std::rand() % 100 + 1;
        total_msec += workload;
        sender.sendOne(szmq::Message::from(workload));
    }
    cout << boost::format("Total expected cost: %1% msec\n") % total_msec;
    sender.close();
    sink.close();
    return 0;
}
```

작업자는 호흡기로부터 받은 메시지에 지정된 시간(밀리초)을 받아 시간만큼 대기하고 수집기에 메시지를 전달합니다.

### taskwork.java 작업자

```java
//  Task worker
//  Connects PULL socket to tcp://localhost:5557
//  Collects workloads from ventilator via that socket
//  Connects PUSH socket to tcp://localhost:5558
//  Sends results to sink via that socket

#include <szmq/szmq.h>
#include <iostream>
#include <thread>
#include <string>
using namespace std;

int main (void) 
{
    //  Socket to receive messages on
    szmq::Context context;
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_CLIENT> receiver(context);
    receiver.connect(szmq::SocketUrl("tcp://localhost:5557"));

    //  Socket to send messages to
    szmq::Socket<ZMQ_PUSH, szmq::ZMQ_CLIENT> sender(context);
    sender.connect(szmq::SocketUrl("tcp://localhost:5558"));

    //  Process tasks forever
    while (1) {
        auto workload = receiver.recvOne().read<int>();
        cout << workload << ".";
        this_thread::sleep_for(chrono::milliseconds(workload));  //  Do the work
        sender.sendOne(szmq::Message::from(""));  //  Send results to sink
    }
    sender.close();
    receiver.close();
    return 0;
}
```

수집기는 100개의 작업들의 결과를 수집하고 전체 경과 시간을 계산하여 작업자들의 개수에 따른 병렬 처리로 작업 시간 개선을 확인합니다(예 : 작업자가 1개일 때 3초이면, 작업자가 3개이면 1초).

### tasksink.java: sinker

```java
//  Task sink
//  Binds PULL socket to tcp://localhost:5558
//  Collects results from workers via that socket

#include <szmq/szmq.h>
#include <boost/format.hpp>
#include <chrono>

using namespace std;

int main (void) 
{
    //  Prepare our context and socket
    szmq::Context context;
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_SERVER> receiver(context);
    receiver.bind(szmq::SocketUrl("tcp://*:5558"));

    //  Wait for start of batch
    auto buffer = receiver.recvOne();

    //  Start our clock now
    chrono::steady_clock::time_point begin = chrono::steady_clock::now();

    //  Process 100 confirmations
    int task_nbr;
    for (task_nbr = 0; task_nbr < 100; task_nbr++) {
        auto buffer = receiver.recvOne();
        if (task_nbr % 10 == 0)
            cout << ":";
        else
            cout << ".";
    }
    //  Calculate and report duration of batch
    chrono::steady_clock::time_point end = chrono::steady_clock::now();
    auto duration = chrono::duration_cast<chrono::milliseconds>(end - begin).count();
    cout <<  boost::format("Total elapsed time: %1% msec\n") % duration;

    receiver.close();
    return 0;
}
```

평균 실행 시간은 대략 5초 정도입니다. 작업자의 개수를 1, 2, 4개로 하면 작업자의 수에 반비례로 실행 시간이 단축됩니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples>  cl -EHsc taskvent.java szmq.lib
PS D:\work\sook\src\szmq\examples>  cl -EHsc taskwork.java szmq.lib
PS D:\work\sook\src\szmq\examples>  cl -EHsc tasksink.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./taskvent
Press Enter when the workers are ready:
Sending tasks to workers...
Total expected cost: 5208 msec

PS D:\work\sook\src\szmq\examples> ./taskwork
70.40.35.37.80.36.6.93.68.12.93.50.44.48.38.22.95.93.23.53.69.97.46.87.96.65.49.21.18.3.10.44.60.95.68.2.38.83.38.70.76.97.3.66.89.24.62.59.37.79.91.95.88.53.30.19.97.90.4.6.73.57.35.30.61.30.89.78.96.45.92.24.61.28.80.63.15.38.22.72.50.65.36.72.79.9.36.25.67.17.1.10.45.9.64.23.48.41.76.86.

PS D:\work\sook\src\szmq\examples> ./tasksink
:.........:.........:.........:.........:.........:.........:.........:.........:.........:.........Total elapsed time: 5379 msec
~~~

* 작업자는 상류의 호흡기와 하류의 수집기를 연결(connect)하며 자유롭게 작업자를 추가할 수 있습니다. 만약 작업자가 호흡기와 수집기에 바인딩(bind)를 수행할 경우 호흡기와 싱크 변경(및 추가) 시마다 작업자를 추가해야 합니다. 작업자에서 동적 요소로 연결(connect)을 수행하고 호흡기와 싱크는 정적 요소로 바인딩(bind)를 수행합니다.
* 모든 작업자가 시작할 때까지 호흡기의 작업 시작과 동기화해야 합니다. 이것은 ØMQ의 일반적인 원칙이며 간단한 해결 방법은 없습니다. connect()함수 수행 시 일정 정도의 시간이 소요되며, 여러 작업자가 호흡기에 연결할 때 첫 번째 작업자가 성공적으로 연결하여 메시지를 수신하는 동안 다른 작업자는 연결을 수행합니다. 동기화되어야만 시스템은 병렬로 동작합니다. 호흡기에서 작업자들이 동기화를 위하여 사용한 getchar()를 제거할 경우 어떤 일이 발생하는지 확인해 보시기 바랍니다.
* 호흡기의 PUSH 소켓은 작업자들에게 작업들을 분배하고 이것을 “부하 분산”이라고 부릅니다(호흡기에 작업자들을 연결하여 동기화되었다는 가정).
* 수집기의 PULL 소켓은 작업자들의 결과를 균등하게 수집하여 이를 “공정-대기열(fair-queuing)”이라고 합니다.

세련된 프로그래머는 세련된 암살자와 같은 모토를 공유합니다. : 항상 일이 끝나면 깨끗하게 정리하기. ØMQ를 사용할 경우, Python과 같은 개발언어는 자동으로 메모리 해제를 수행하지만, C/C++ 언어의 경우 종료 시 각 객체들에 대한 메모리 해제를 수행하지 않을 경우 메모리 누수 현상이나 불안정된 상태를 가질 수 있습니다.

## 왜 ØMQ가 필요한가

오늘날 많은 응용프로그램들이 일종의 네트워크상(랜 혹은 인터넷)에서 각종 구성 요소들로 확장되고 있으며 많은 개발자들이 TCP와 UDP 통신규약을 사용하고 있습니다. 이러한 통신규약을 사용하지 어렵지는 않지만 메시지를 전송(A->B)하는 것과 신뢰성 있게 메시지를 전송하는 것에는 큰 차이가 있습니다.

대부분의 메세징 프로젝트와 같이 AMQP에서도 재사용에 대한 오랫동안 지속된 문제를 해결하려 했으며, 새로운 개념으로 “브로커(Broker)”를 발명하였으며, 브로커는 주소 지정, 라우팅, 대기열 관리를 수행하였습니다. 그 결과 클라이언트/서버 통신규약과 일련의 API들은 응용프로그램들이 브로커를 통하여 소통할 수 있게 되었습니다. 브로커는 거대한 네트워크상에서 복잡성을 제거하는 데는 뛰어났지만, 주키퍼처럼 브로커 기반 메세징은 상황을 개선하기는 보다 더 나쁘게 만들었습니다. 이것은 브로커라는 부가적인 거대한 요소를 추가함으로 새로운 단일 장애점(SPOF, Single Point Of Failure)이 되었습니다. 브로커는 빠르게 병목 지점이 되어감에 따라 새로운 위험을 관리해야 했습니다. 소프트웨어적으로 이러한 이슈(장애조치)를 해결하기 위하여 브로커를 2개, 3개, 4개 추기 해야 했습니다. 이것을 결국 더욱 많은 조각으로 나누어지며, 복잡성을 증가하고, 중단 지점을 늘어가게 했습니다.

브로커 중심 설정은 이것을 위한 별도의 운영 조직을 필요하게 되었으며, 브로커가 매일 잘 동작하는지 모니터링하고 비정상적인 동작을 보일 때는 조정해야 했습니다. 더 많은 머신과 백업을 위한 추가적인 머신 그리고 이것을 관리하기 위한 사람 등, 브로커는 많은 데이터가 움직이는 거대한 응용프로그램에 대하여 오랫동안 여러 개의 팀을 통해 운영이 가능할 경우 적절합니다.

중소규모의 응용프로그램 개발자들이 네트워크 프로그래밍을 회피하거나 규모를 조정할 수 없는 모노리식(monolithic) 응용프로그램을 만들면서 덧에 빠지거나 잘못된 네트워크 프로그래밍으로 인하여 불안정하고 복잡한 응용프로그램을 만들어 유지 보수가 어렵게 됩니다. 또는 상용 메시징 제품을 의지하고 확장 가능하지만 투자 비용이 크거나 도태되는 기술일 경우도 있습니다. 지난 세기에서 메시징이 거대한 화두였지만 탁월한 선택은 없었으며 상용 메세징 제품을 지원하거나 라이선스를 판매하는 사람들에게는 축복이었지만, 사용자에게 비극이었기 때문입니다.

우리가 필요한 것은 메세징에 대한 작업은 단순하고 저비용이어야 하고 어떤 응용프로그램이나 운영체제에서도 동작해야 합니다. 이것은 어떤 의존성도 없는 라이브러리가 되어야 하며, 어떤 프로그램 언어에서도 사용 가능해야 합니다.

이것이 ØMQ입니다 : 내장된 라이브러리를 통하여 적은 비용과 효율적으로 네트워크상에서 대부분의 문제를 해결할 수 있습니다.

ØMQ의 특이점은 다음과 같습니다 :

* 백그라운드 스레드들에서 비동기 I/O 처리합니다.
* 백그라운드 스레드들은 응용프로그램 스레드들 간에 통신을 하며, 잠금 없는 자료 구조를 사용하여 동시성 ØMQ 응용프로그램은 잠금, 세마포어, 대기 상태와 같은 것들을 필요로 하지 않습니다.
* 서비스 구성요소는 동적 요구에 따라 들어오고 나갈 수 있으며 ØMQ는 자동으로 재연결할 수 있습니다.
서비스 구성요소를 어떤 순서로도 시작할 수 있으며 마치 서비스 지향 아키텍처처럼 네트워크상에서 언제든지 합류하거나 떠날 수 있습니다.
* 필요시 자동으로 메시지들을 대기열에 넣습니다.
* 지능적으로 수행하여 메시지를 대기열에 추가하기 전에 가능한 수신자에게 전송합니다.
* 가득 찬 대기열(Over-full Queue(HWM, 최고 수위 표시))을 다룰 수 있습니다.
* 대기열이 가득 차게 되면 ØMQ는 특정 메세징 종류(소위 “패턴”)에 따라 자동적으로 송신자의 메시지를 막거나 혹은 버릴 수 있습니다.
* 응용프로그램들을 임의의 전송계층상에서 상호 통신하게 합니다.
* TCP, PGM(multicast), inproc(in-process), ipc(inter-process) 등 다른 전송계층들을 사용하더라도 소스코드를 수정하지 않고 통신할 수 있습니다.
* 지연/차단된 수신자들을 메세징 패턴에 따라 다른 전략을 사용하여 안전하게 처리합니다.
* 요청-응답, 발행-구독과 같은 다양한 패턴을 통하여 메시지를 전송할 수 있습니다. 이러한 패턴은 네트워크의 구조와 위상을 생성하는 방법입니다.
* 단일 호출(zmq_proxy())로 메시지를 대기열에 추가, 전달 또는 캡처하는 프록시를 만들 수 있습니다. 프록시는 네트워크의 상호 연결 복잡성을 줄일 수 있습니다.
* 네트워크 상의 단순한 프레이밍(Framing)을 사용하여 메시지를 전송한 상태 그대로 전달합니다. 10,000개 메시지를 전송하면 10,000개 메시지를 받게 됩니다.
* 메시지의 어떤 포맷도 강요하지 않습니다.
* 메시지들은 0에서 기가바이트로 거대할 수 있으며 데이터를 표현하기 위해서는 상위에 별도 제품을 사용하면 됩니다.
* 필요할 경우 자동 재시도를 통해 네트워크 장애를 지능적으로 처리합니다.
* 탄소 배출량을 줄입니다.
* CPU 자원을 덜 사용하여 전력을 덜 소모하게 하며, 오래된 컴퓨터도 사용할 수 있게 합니다. 엘 고어(Al Gore)는 ØMQ를 사랑할 겁니다.

사실 ØMQ는 나열한 것 이상의 것을 하며 네트워크 지원 응용프로그램 개발에 파괴적인 영향을 줄 수 있습니다. 표면적으로 소켓과 같은 API이지만 메시지 처리 절차는 내부적으로 일련의 메시지 처리 작업들로 쪼개져서 처리됩니다. 이것은 우아하고 자연스럽고 규모를 조정할 수 있으며 각각의 작업들은 임의의 전송계층에서 하나의 노드, 여러 개의 노드들과 매핑되어 처리됩니다. 2개의 노드들이 하나의 프로세스(노드는 스레드)에서, 2개의 노드들이 하나의 머신(노드는 프로세스)에서, 2개의 노드들이 하나의 네트워크(노드는 머신)에서 처리되며 응용프로그램 소스는 모두 동일합니다.

ØMQ를 사용하면서 결국 진리와 깨달음을 경험하는 순간, 당신은 번쩍-우르릉-쾅쾅 깨달음(zap-pow-kaboom satori) 패러다임 전환을 하게 될 것입니다.

# 2장-소켓 및 패턴

1장 - 기본에서 몇 가지 ØMQ 패턴의 기본 예제로 ØMQ의 세계로 뛰어들었습니다 : 요청-응답, 발행-구독, 파이프라인. 이장에서는 우리의 손을 더럽히면서 실제 프로그램에서 이러한 도구들을 어떻게 사용하는지 배우게 될 것입니다.

## 다중 소켓 다루기

한 번에 여러 소켓에서 읽으려면 szmq::poll()을 사용해야 합니다. 더 좋은 방법은 szmq::poll()을 프레임워크로 감싸서 이벤트 중심으로 반응하도록 변형하는 것이지만, 여기서 다루고 싶은 것보다 훨씬 더 많은 작업이 필요합니다.

그렇게 하지 말라는 예로, 비차단(nonblocking) 소켓 읽기를 수행하는 방법을 보여 드리겠습니다. 
다음 예제는 2개의 소켓으로 메시지를 비차단 읽기로 수신하고 있으며 프로그램은 날씨 변경정보의 구독자와 병렬 작업의 작업자 역할을 수행하여 다소 혼란스럽습니다.

### msreader.java: 다중 소켓 읽기

```java
//  Reading from multiple sockets
//  This version uses a simple recv loop

#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <thread>
using namespace std;

int main (void) 
{
    //  Connect to task ventilator
    szmq::Context context;
    // make a non blocking socket
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_CLIENT> receiver(context);
    receiver.connect(szmq::SocketUrl("tcp://localhost:5557"));

    //  Connect to weather server
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(context);
    subscriber.connect(szmq::SocketUrl("tcp://localhost:5556"));
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, "10001", 5);

    //  Process messages from both sockets
    //  We prioritize traffic from the task ventilator
    while (1) {
        while (1) {
            auto msg = receiver.recvOne(0);
            if (msg.size() != 0) {
                //  Process task
                cout << "P";
            }
            else
                break;
        }
        while (1) {
            auto msg = subscriber.recvOne(0);
            if (msg.size() != 0) {
                //  Process weather update
                cout << "S";
            }
            else
                break;
        }
        //  No activity, so sleep for 1 msec
        std::this_thread::sleep_for(std::chrono::milliseconds(1));      //  Do some 'work'
    }
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc msreader.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./taskvent
Press Enter when the workers are ready:
Sending tasks to workers...
Total expected cost: 5207 msec
PS D:\work\sook\src\szmq\examples> ./taskwork
74.98.24.45.45.82.85.63.8.75.41.95.53.76.63.19.40.36.59.77.44.47.
94.60.10.96.46.52.11.39.79.63.18.44.45.99.10.71.75.29.9.77.59.96.41.
15.52.91.70.78.88.55.36.59.97.77.82.96.98.14.32.98.37.83.5.20.88.43.
8.24.92.23.19.24.54.25.24.16.32.29.75.54.70.52.21.54.50.53.52.75.25.
32.75.26.7.37.71.14.48.78.
PS D:\work\sook\src\szmq\examples> ./msreader
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPSSSSSSSSSSSSSSSSSSSSS
~~~

이런 접근에 대한 비용은 첫 번째 메시지에 대한 추가적인 지연이 발생합니다(메시지 처리하기에도 바쁜데 루푸의 마지막에 sleep()). 이러한 접근은 고속 처리가 필요한 응용프로그램에서 치명적인 지연을 발생합니다. nanosleep()를 사용할 경우 바쁘게 반복(busy-loop)되지 않는지 확인해야 합니다.

예제에서 2개의 루프에서 첫 번쨰 소켓(ZMQ_PULL)이 두 번쨰 소켓(ZMQ_SUB)보다 먼저 처리하게 하였습니다.

대안으로 이제 zmq_poll()을 사용하는 응용프로그램을 보도록 하겠습니다.

### mspoller.java: 다중 소켓 폴러

```java
//  Reading from multiple sockets
//  This version uses zmq_poll()

#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <thread>
using namespace std;

int main (void) 
{

    //  Connect to task ventilator
    szmq::Context context;
    // make a non blocking socket
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_CLIENT> receiver(context, szmq::NonblockingFlag(false));
    receiver.connect(szmq::SocketUrl("tcp://localhost:5557"));

    //  Connect to weather server
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(context, szmq::NonblockingFlag(false));
    subscriber.connect(szmq::SocketUrl("tcp://localhost:5556"));
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, "10001", 5);

    //  Process messages from both sockets
    while (1) {
        std::vector<szmq::PollItem> pollItems = {
			{reinterpret_cast<void*>(*receiver), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*subscriber), 0, ZMQ_POLLIN, 0}};
        szmq::poll(pollItems, 2, -1);
        if (pollItems [0].revents & ZMQ_POLLIN) {
            auto msg = receiver.recvOne();
            //  Process task
            cout << "P";

        }
        if (pollItems [1].revents & ZMQ_POLLIN) {
            auto msg = subscriber.recvOne();
            //  Process weather update
            cout << "S";
        }
    }
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc mspoller.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./taskvent
Press Enter when the workers are ready:
Sending tasks to workers...
Total expected cost: 5207 msec
PS D:\work\sook\src\szmq\examples> ./taskwork
74.98.24.45.45.82.85.63.8.75.41.95.53.76.63.19.40.36.59.77.44.47.
94.60.10.96.46.52.11.39.79.63.18.44.45.99.10.71.75.29.9.77.59.96.41.
15.52.91.70.78.88.55.36.59.97.77.82.96.98.14.32.98.37.83.5.20.88.43.
8.24.92.23.19.24.54.25.24.16.32.29.75.54.70.52.21.54.50.53.52.75.25.
32.75.26.7.37.71.14.48.78.
PS D:\work\sook\src\szmq\examples> ./mspoller
SSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPPP
PPPPPPPPPPPPPPPPSSSSSSSSSSSSSSSSSSSS
...
~~~

"szmq::poll(pollItems, -1)"에서 3번째 인수가 “-1”일 경우, 이벤트가 발생할때까지 기다리게 됩니다.

## 멀티파트 메시지들

ØMQ는 여러 개의 프레임들로 하나의 메시지를 구성하게 하며 멀티파트 메시지를 제공합니다. 현실에서는 응용프로그램은 멀티파트 메시지를 많이 사용하며 IP 주소 정보를 메시지에 내포하여 직열화 용도로 사용합니다. 우리는 이것을 응답 봉투에서 나중에 다루겠습니다.

멀티파트 메시지 작업 시, 개별 파트는 Message 객체이며, 메시지를 5개의 파트들로 전송한다면 반드시 5개의 Message 객체들을 생성, 전송, 파괴를 해야 합니다. 이것을 미리 하거나(Message 배열이나 구조체에 저장하여) 혹은 전송 이후 차례로 할 수 있습니다.

```java
  socket.sendMore(szmq::Message::from(std::string("message")));
  ...
  socket.sendMore(szmq::Message::from(std::string("message")));
  ...
  socket.sendOne(szmq::Message::from(std::string("message")));
```

메세지를 수신시에 메시지에 있는 모든 파트들을 수신하고 처리하는 예제입니다. 파트는 1개이거나 멀티파트로 가능합니다.

```java
while (1) {
    auto message socket.recvOne().read<std::string>()
    // Process the message frame
    ...
    if (!socket.hasMore())
        break; // Last message frame
}
```

### rrboker.java 단순한 요청/응답 브로커

```java
//  Simple request-reply broker

#include <szmq/szmq.h>
#include <iostream>
using namespace std;

int main (void) 
{
    //  Prepare our context and sockets
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);    
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> backend(context);
    int rc = frontend.bind(szmq::SocketUrl("tcp://*:5559"));
    assert (rc == 0);
    rc = backend.bind(szmq::SocketUrl("tcp://*:5560"));
    assert (rc == 0);

    //  Initialize poll set
    std::vector<szmq::PollItem> pollItems = {
		{reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN, 0},
        {reinterpret_cast<void*>(*backend), 0, ZMQ_POLLIN, 0}};

    //  Switch messages between sockets
    while (1) {
        szmq::poll(pollItems, 2, -1);
        if (pollItems [0].revents & ZMQ_POLLIN) {
            while (1) {
                //  Process all parts of the message
                auto message = frontend.recvOne();
                bool more = frontend.hasMore();
                if(more)
                    backend.sendMore(message);
                else{
                    backend.sendOne(message);
                    break;       //  Last message part
                }
            }
        }
        if (pollItems [1].revents & ZMQ_POLLIN) {
            while (1) {
                //  Process all parts of the message
                auto message = backend.recvOne();
                bool more = backend.hasMore();
                if(more)
                    frontend.sendMore(message);
                else{
                    frontend.sendOne(message);
                    break;       //  Last message part
                }
            }
        }
    }
    //  We never get here, but clean up anyhow
    frontend.close();
    backend.close();
    return 0;
}
```

### rrhwserver.java : 브로커를 통한 Hello World 서버

```java
//  Hello World server

#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <thread>

using namespace std;

int main (void)
{
    //  Socket to talk to clients
    szmq::Context context;
    szmq::Socket<ZMQ_REP, szmq::ZMQ_CLIENT> responder(context);
    responder.connect(szmq::SocketUrl("tcp://localhost:5560"));

    while (1) {
        auto buffer = responder.recvOne();
        cout << "received request: " << buffer.read<std::string>() << endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));      //  Do some 'work'
        responder.sendOne(szmq::Message::from("World"));
    }
    responder.close();
    return 0;
}
```

### rrhwclient.java : 브로커를 통한 Hello World 클라이언트

```java
//  Hello World client
#include <szmq/szmq.h>
#include <assert.h>
#include <iostream>

using namespace std;

int main (void)
{
    cout << "Connecting to hello world server...\n";
    szmq::Context context;
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> requester(context);
    requester.connect(szmq::SocketUrl("tcp://localhost:5559"));

    int request_nbr;
    for (request_nbr = 0; request_nbr != 10; request_nbr++) {
        cout << "Sending Hello : " << request_nbr << endl;
        requester.sendOne(szmq::Message::from("Hello"));
        auto buffer = requester.recvOne();
        cout << "Received : " << buffer.read<std::string>() << endl;
    }
    requester.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc rrbroker.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc rrhwserver.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc rrhwclient.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./rrbroker

PS D:\work\sook\src\szmq\examples> ./rrhwserver
received request: Hello
received request: Hello
...

PS D:\work\sook\src\szmq\examples> ./rrhwclient
Connecting to hello world server...
Sending Hello : 0
Received : World
Sending Hello : 1
Received : World
Sending Hello : 2
Received : World
...
~~~

REQ - ROUTER - DEALER - REP 패턴으로 테스트가 가능하였습니다.
이런 구조에서는 작업자에서 제공하는 서비스를 구분해야 할 경우(설비 연계 서비스, 사용자 연계 서비스, 트랜젝션 처리 서비스 등) 개별 서비스에 대하여 요청/응답 브로커에서 분배합니다. 클라이언트에서 각 서비스를 제공하는 작업자들의 정보를 등록하여 찾아가게 하거나, 브로커에 서비스 추가/삭제하여 클라이언트가 브로커에 서비스를 요청하고 제공받게 할 수 있습니다. 작업자의 서비스들을 클라이언트에 정적으로 보관하거나, 브로커에서 서비스 관리하지 않기 위해서는 클라이언트에서 필요한 서비스를 찾을 수 있도록 서비스를 식별하기 위한 중앙의 저장소를 두어 운영할 수도 있습니다.

## ØMQ의 내장된 프록시 함수
이전 섹션의 rrbroker의 코어 루프는 매우 유용하고 재사용 가능합니다. 적은 노력으로 PUB-SUB 포워더와 공유 대기열과 같은 작은 중개자를 구축할 수 있었습니다. ØMQ는 이 같은 기능을 감싼 하나의 함수 szmq::proxy()를 준비하고 있습니다.

2개의 소켓(또는 데이터를 캡처하려는 경우 3개 소켓)이 연결, 바인딩 및 구성되어야 합니다. szmq::proxy() 함수를 호출하면 rrbroker의 메인 루프를 시작하는 것과 같습니다. szmq::proxy()를 호출하도록 요청-응답 브로커를 다시 작성하겠습니다.

### msgqueue.java: 메세지 대기열 브로커

```java
//  Simple message queuing broker
//  Same as request-reply broker but using shared queue proxy

#include <szmq/szmq.h>
using namespace std;

int main (void) 
{
    szmq::Context context;

    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);      
    int rc = frontend.bind(szmq::SocketUrl("tcp://*:5559"));
    assert (rc == 0);

    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> backend(context);
    rc = backend.bind(szmq::SocketUrl("tcp://*:5560"));
    assert (rc == 0);

    //  Start the proxy
    szmq::proxy(reinterpret_cast<void*>(*frontend), reinterpret_cast<void*>(*backend), NULL);
    //  We never get here...
    frontend.close();
    backend.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc msgqueue.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc rrhwserver.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc rrhwclient.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./msgqueue

PS D:\work\sook\src\szmq\examples> ./rrhwserver
received request: Hello
received request: Hello
...

PS D:\work\sook\src\szmq\examples> ./rrhwclient
Connecting to hello world server...
Sending Hello : 0
Received : World
Sending Hello : 1
Received : World
Sending Hello : 2
Received : World
...
~~~

## 전송계층 간의 연결
ØMQ 사용자의 계속되는 요청은 “기술 X와 ØMQ 네트워크를 어떻게 연결합니까?”입니다. 여기서 X는 다른 네트워킹 또는 메시징 기술입니다.

간단한 대답은 브리지(Bridge)를 만드는 것입니다. 브리지는 한 소켓에서 하나의 통신규약을 말하고, 다른 소켓에서 두 번째 통신규약으로 변환하는 작은 응용프로그램입니다. 통신규약 번역기라고 할 수 있으며, ØMQ는 2개의 서로 다른 전송방식과 네트워크를 연결할 수 있습니다.

예제로, 발행자와 일련의 구독자들 간에 2개의 네트워크를 연결하는 작은 프록시를 작성하도록 하겠습니다. 프론트엔드 소켓은 날씨 서버가 있는 내부 네트워크를 향하며, 백엔드 소켓은 외부 네트워크의 일련의 구독자들을 향하게 합니다. 프록시의 프론트엔드 소켓에서 날씨 서비스에 구독하고 백엔드 소켓으로 데이터를 재배포하게 합니다.

### wuproxy.java: 기상정보 변경 프록시

```java
//  Weather proxy device

#include <szmq/szmq.h>
using namespace std;

int main (void)
{
    szmq::Context context;

    szmq::Socket<ZMQ_XSUB, szmq::ZMQ_CLIENT> frontend(context);      
    frontend.connect(szmq::SocketUrl("tcp://192.168.55.210:5556"));

    szmq::Socket<ZMQ_XPUB, szmq::ZMQ_SERVER> backend(context);
    backend.bind(szmq::SocketUrl("tcp://10.1.1.0:8100"));


    //  Run the proxy until the user interrupts us
    szmq::proxy(reinterpret_cast<void*>(*frontend), reinterpret_cast<void*>(*backend), NULL);
    
    frontend.close();
    backend.close();
    return 0;
}
```

이전 프록시 예제와 매우 유사하지만, 중요한 부분은 프론트엔드와 백엔드 소켓이 서로 다른 2개의 네트워크에 있다는 것입니다. 예를 들어 이 모델을 사용하여 멀티캐스트 네트워크(pgm 전송방식)를 TCP 발행자에 연결할 수도 있습니다.

## 오류 처리와 ETERM
ØMQ의 오류 처리 철학은 빠른 실패와 회복의 조합입니다. 프로세스는 내부 오류에 대해 가능한 취약하고 외부 공격 및 오류에 대해 가능한 강력해야 대응해야 합니다. 유사한 예로 살아있는 세포는 내부 오류가 하나만 감지되면 스스로 죽지만, 외부로부터의 공격에는 모든 가능한 수단을 통하여 저항합니다.

어설션(assertion)은 ØMQ의 안정적인 코드에 필수적인 조미료입니다. 그들은 세포막에 있어야 하며 그런 막을 통하여 결함이 내부 또는 외부인지 구분하고, 불확실한 경우 설계 결함으로 수정해야 합니다.. C/C++에서 어설션은 오류 발생 시 응용프로그램을 즉시 중지합니다. 다른 언어에서는 예외 혹은 중지가 발생할 수 있습니다.

hwserver.c를 수정(hwserver1.c)하여 “strerror(errno)” 확인하겠습니다.

### hwserver1.java : Hellow World 서버 수정본

```java
//  Hello World server

#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <thread>

using namespace std;

int main (void)
{
    //  Socket to talk to clients
    szmq::Context context;
    szmq::Socket<ZMQ_REP, szmq::ZMQ_SERVER> responder(context);
    int rc = responder.bind(szmq::SocketUrl("tcp://*:5555"));
    if (rc == -1){
        printf("E: bind failed: %s\n", zmq_strerror(zmq_errno()));
        return -1;
    }
    
    while (1) {
        auto buffer = responder.recvOne();
        cout << "received request: " << buffer.read<std::string>() << endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));      //  Do some 'work'
        responder.sendOne(szmq::Message::from("World"));
    }
    responder.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc hwserver1.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./hwserver1

PS D:\work\sook\src\szmq\examples> ./hwserver1
E: bind failed: Address in use
~~~

C/C++의 assert()는 최적화에 의해 완전히 제거되므로 assert()내에서 ØMQ API를 호출해서는 안됩니다. 최적화를 통해 모든assert()는 제거된 응용프로그램에서 개발자의 의도에 따라 ØMQ API 호출을 작성하여 오류가 발생할 경우 중단하게 해야 합니다.

프로세스를 깨끗하게 종료하는 방법을 알아보겠습니다. 이전 섹션의 병렬 파이프라인 예제를 보면, 백그라운드에서 많은 수의 작업자를 시작한 후 작업이 완료되면 수동으로 해당 작업자를 종료해야 했습니다. 이것을 작업이 완료되면 작업자에서 종료 신호를 보내 자동으로 종료하도록 하겠습니다. 이 작업을 수행하기 가장 좋은 대상은 작업이 완료된 시점을 알고 있는 수집기입니다.

작업자와 수집기를 연결하기 위하여 사용한 방법은 단방향 PUSH/PULL 소켓입니다. 다른 소켓 유형으로 전환하거나 다중 소켓을 혼합할 수 있습니다. 후자를 선택하여 : PUB-SUB 모델로 수집기에서 종료 메시지를 작업자에게 보냅니다.

* 수집기는 새로운 단말에 PUB 소켓을 생성합니다.
* 작업자들은 SUB 소켓을 수집기 단말에 연결합니다.
* 수집기가 일괄 작업의 종료 감지하면 PUB 소켓에 종료 신호를 보냅니다.
* 작업자들이 종료 메시지(kill message)를 감지하면 종료됩니다.

### taskwork2.java: 종료 신호를 처리하는 작업자

```java
//  Task worker - design 2
//  Adds pub-sub flow to receive and respond to kill signal

#include <szmq/szmq.h>
#include <iostream>
#include <thread>
#include <string>
using namespace std;

int main (void) 
{
    //  Socket to receive messages on
    szmq::Context context;
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_CLIENT> receiver(context);
    receiver.connect(szmq::SocketUrl("tcp://localhost:5557"));

    //  Socket to send messages to
    szmq::Socket<ZMQ_PUSH, szmq::ZMQ_CLIENT> sender(context);
    sender.connect(szmq::SocketUrl("tcp://localhost:5558"));

    //  Socket for control input
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> controller(context);
    controller.connect(szmq::SocketUrl("tcp://localhost:5559"));
    controller.setSockOpt(ZMQ_SUBSCRIBE, "", 0);

    //  Process messages from either socket
    while (1) {
        std::vector<szmq::PollItem> pollItems = {
			{reinterpret_cast<void*>(*receiver), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*controller), 0, ZMQ_POLLIN, 0}};
        szmq::poll(pollItems, 2, -1);

        if (pollItems [0].revents & ZMQ_POLLIN) {
            auto string = receiver.recvOne().read<std::string>();
            cout << string << ".";
            this_thread::sleep_for(chrono::milliseconds(stol(string)));
            sender.sendOne(szmq::Message::from(""));  //  Send results to sink
        }
        //  Any waiting controller command acts as 'KILL'
        if (pollItems [1].revents & ZMQ_POLLIN)
            break;                      //  Exit loop
    }
    sender.close();
    receiver.close();
    controller.close();
    return 0;
}
```

### tasksink2.java: 강제 종료(kill) 신호를 가진 병렬 작업 수집기(sink)

```java
//  Task sink - design 2
//  Adds pub-sub flow to send kill signal to workers

#include <szmq/szmq.h>
#include <boost/format.hpp>
#include <chrono>

using namespace std;

int main (void) 
{
    //  Socket to receive messages on
    szmq::Context context;
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_SERVER> receiver(context);
    receiver.bind(szmq::SocketUrl("tcp://*:5558"));

    //  Socket for worker control
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> controller(context);
    controller.bind(szmq::SocketUrl("tcp://*:5559"));

    //  Wait for start of batch
    auto buffer = receiver.recvOne();
    
    //  Start our clock now
    chrono::steady_clock::time_point begin = chrono::steady_clock::now();

    //  Process 100 confirmations
    int task_nbr;
    for (task_nbr = 0; task_nbr < 100; task_nbr++) {
        auto buffer = receiver.recvOne();
        if (task_nbr % 10 == 0)
            cout << ":";
        else
            cout << ".";
    }
    //  Calculate and report duration of batch
    chrono::steady_clock::time_point end = chrono::steady_clock::now();
    auto duration = chrono::duration_cast<chrono::milliseconds>(end - begin).count();
    cout <<  boost::format("Total elapsed time: %1% msec\n") % duration;

    //  Send kill signal to workers
    controller.sendOne(szmq::Message::from(std::string("KILL")));
    receiver.close();
    controller.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc taskwork2.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc tasksink2.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./taskvent
Press Enter when the workers are ready:
Sending tasks to workers...
Total expected cost: 4924 msec

PS D:\work\sook\src\szmq\examples> ./taskwork2
79.92.61.63.19.64.79.75.99.77.20.26.9.30.41.25.13.65.37.5.79.62.90.
23.24.54.69.19.16.39.48.10.59.2.85.27.18.12.60.12.79.5.73.70.95.42.
67.97.74.30.32.49.48.32.7.73.55.23.30.56.21.33.11.66.4.96.53.47.75.
98.94.67.94.92.96.1.61.9.17.81.68.7.63.72.55.27.60.93.13.1.21.87.94.
79.13.40.30.50.53.20.

PS D:\work\sook\src\szmq\examples> ./tasksink2
:.........:.........:.........:.........:.........:...
......:.........:.........:.........:.........
Total elapsed time: 5103 msec
~~~

## ØMQ에서 멀티스레드

ØMQ는 멀티스레드 응용프로그램의 작성에 최적화되어 있습니다. 전통적인 소켓을 사용하셨다면, ØMQ 소켓 사용 시 약간의 재조정이 필요하지만, ØMQ 멀티스레딩은 기존에 알고 계셨던 멀티스레드 응용프로그램 작성 경험이 거의 필요하지 않습니다. 기존의 지식을 정원에 던져버리고 기름을 부어 태워 버리십시오. 책(지식)을 불태우는 경우는 드물지만, 동시성 프로그래밍의 경우 필요합니다.

ØMQ에서는 완벽한 멀티스레드 응용프로그램을 만들기 위해(그리고 문자 그대로), 뮤텍스, 잠금이 불필요하며 ØMQ 소켓을 통해 전송되는 메시지를 제외하고는 어떤 다른 형태의 스레드 간 통신이 필요하지 않습니다.

“완벽한 멀티스레드 프로그램”은 작성하기 쉽고 이해하기 쉬운 코드로 어떤 프로그래밍 언어와 운영체제에서도 동일한 설계 방식으로 동작하며, CPU 수에 비례하여 자원의 낭비 없이 성능이 확장되어야 합니다.

멀티스레드 코드 작성하기 위한 기술(잠금 및 세마포어, 크리티컬 섹션 등)을 배우기 위해 수년의 시간을 보냈다면, 그것이 아무것도 아니라는 것을 깨닫는 순간 어처구니가 없을 것입니다. 30년 이상의 동시성 프로그래밍에서 배운 교훈이 있다면, 그것은 단지 “상태(& 자원)를 공유하지 마십시오”입니다. 이것은 한잔의 맥주를 같이 마시려고 하는 두 명의 술꾼들과 같습니다. 그들이 좋은 친구인지는 중요하지 않습니다. 조만간 그들은 싸우게 될 것입니다. 그리고 술꾼들과 테이블이 많을수록 맥주를 놓고 서로 더 많이 싸우게 됩니다. 멀티스레드 응용프로그램의 비극의 대다수는 술집에서 술꾼들의 싸움처럼 보입니다.

ØMQ로 즐거운 멀티스레드 코드를 작성하려면 몇 가지 규칙을 따라야 합니다.

* 스레드 내에 데이터를 개별적으로 고립시키고 스레드들 간에 공유하지 않습니다. 예외는 ØMQ 컨텍스트로 스레드 안전합니다.
* 전통적인 병렬 처리인 뮤텍스, 크리티컬 섹션, 세마포어 등을 멀리 하며, 이것은 ØMQ 응용프로그램에 부적절합니다.
* 프로세스 시작 시 단 하나의 ØMQ 컨텍스트를 생성하여 inproc(스레드 간 통신) 소켓을 통해 연결하려는 모든 스레드들에 전달합니다.
* 응용프로그램에서 구조체를 생성하기 위하여 할당된 스레드들(attached threads)을 사용하며, inproc상에서 페어 소켓을 사용하여 부모 스레드에 연결합니다. 이러한 패턴은 부모 소켓을 바인딩 한 다음에 부모 소켓을 연결하는 자식 스레드를 만듭니다.
* 독립적인 작업을 위하여 자체 컨텍스트를 가진 해제된 스레드(detached threads) 사용합니다. TCP상에서 연결하며 나중에 소스 코드의 큰 변경 없이 독립적인 프로세스들로 전환 가능합니다.
* 모든 스레드들 간의 정보 교환은 ØMQ 메시지로 이루어지며, 어느 정도 공식적으로 정의할 수 있습니다.
* 스레드들 간에 ØMQ 소켓을 공유하지 말아야 하며, ØMQ 소켓은 스레드 안전하지 않습니다. 소켓을 다른 스레드로 전환하는 것은 기술적으로는 가능하지만 숙련된 기술이 필요합니다. 여러 스레드에서 하나의 소켓을 취급하는 유일한 방법은 마법 같은 가비지 수집을 가진 언어 바인딩 정도입니다.

실제 동작 방법으로 이전 Hello World 서버(hwserver)를 보다 유능한 것으로 변환하겠습니다. 원래 서버는 단일 스레드로 구동되었습니다. 요청당 작업이 적을 경우 문제 되지 않지만 : 하나의 ØMQ 스레드가 CPU 코어에서 최고 속도로 실행되며 대기 없이 많은 작업을 수행할 수 있습니다. 그러나 현실적인 서버는 요청마다 중요한 작업을 수행해야 합니다. 단일 CPU 코어로는 10,000개의 클라이언트의 요청을 하나의 스레드인 서버에서 처리하기에는 충분하지 않습니다. 그래서 현실적인 서버는 여러 작업자 스레드들을 구동시켜 10,000개의 클라이언트의 요청을 최대한 빨리 받아 작업자 스레드에 배포합니다. 작업자 스레드들은 작업을 분산 처리하여 결국 응답을 보냅니다.

물론 브로커와 외부 작업자 프로세스를 사용하여 모든 작업을 수행할 수 있지만, 16개 프로세스로 16 코어 CPU를 소모하기보다는 하나의 프로세스에서 멀티스레드를 구동하는 것이 더욱 쉽습니다. 추가로 하나의 프로세스에서 작업자 스레드들을 실행하면 네트워크 홉, 지연시간, 트래픽을 줄일 수 있습니다.

멀티스레드 버전의 Hello World 서비스는 기본적으로 하나의 프로세스 내에서 브로커와 작업자로 구성하게 합니다.



1. pthread 라이브러를 사용할 경우 

### mtserver.java: 멀티스레드 Hello World 서버

```java
//  Multithreaded Hello World server

#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <thread>
#include <pthread.h>
using namespace std;
static void *
worker_routine (void *arg) {
    //  Socket to talk to dispatcher
    szmq::Context *context = (szmq::Context *) arg; 
    szmq::Socket<ZMQ_REP, szmq::ZMQ_CLIENT> receiver(*context);
    receiver.connect(szmq::SocketUrl("inproc://workers"));

    while (1) {
        auto buffer = receiver.recvOne();
        cout << "received request: " << buffer.read<std::string>() << endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));      //  Do some 'work'
        receiver.sendOne(szmq::Message::from("World"));
    }
    receiver.close();
    return NULL;
}

int main (void)
{
    szmq::Context context;

    //  Socket to talk to clients
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> clients(context);   
    clients.bind(szmq::SocketUrl("tcp://*:5555")); 
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> workers(context);
    workers.bind(szmq::SocketUrl("inproc://workers"));

    //  Launch pool of worker threads
    int thread_nbr;
    for (thread_nbr = 0; thread_nbr < 5; thread_nbr++) {
        pthread_t worker;
        pthread_create (&worker, NULL, worker_routine, (void *)&context);
    }
    //  Connect work threads to client threads via a queue proxy
    szmq::proxy(reinterpret_cast<void*>(*clients), reinterpret_cast<void*>(*workers), NULL);

    //  We never get here, but clean up anyhow
    clients.close();
    workers.close();
    return 0;
}
```

2. std::thread를 사용할 경우

### mtserver1.java: 멀티스레드 Hello World 서버

```java
//  Multithreaded Hello World server

#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <thread>
using namespace std;
static void *
worker_routine (void *arg) {
    //  Socket to talk to dispatcher
    szmq::Context *context = (szmq::Context *) arg; 
    szmq::Socket<ZMQ_REP, szmq::ZMQ_CLIENT> receiver(*context);
    receiver.connect(szmq::SocketUrl("inproc://workers"));

    while (1) {
        auto buffer = receiver.recvOne();
        cout << "received request: " << buffer.read<std::string>() << endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));      //  Do some 'work'
        receiver.sendOne(szmq::Message::from("World"));
    }
    receiver.close();
    return NULL;
}

int main (void)
{
    szmq::Context context;

    //  Socket to talk to clients
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> clients(context);   
    clients.bind(szmq::SocketUrl("tcp://*:5555")); 
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> workers(context);
    workers.bind(szmq::SocketUrl("inproc://workers"));


    //  Launch pool of worker threads
    int thread_nbr;
    for (thread_nbr = 0; thread_nbr < 5; thread_nbr++) {
        thread worker(&worker_routine, (void *)&context);
        worker.detach();
    }
    //  Connect work threads to client threads via a queue proxy
    szmq::proxy(reinterpret_cast<void*>(*clients), reinterpret_cast<void*>(*workers), NULL);

    //  We never get here, but clean up anyhow
    clients.close();
    workers.close();
    return 0;
}
```

3. czmq의 zthread를 사용할 경우

### mtserver2.java: 멀티스레드 Hello World 서버

```java
//  Multithreaded Hello World server

#include <szmq/szmq.h>
#include <iostream>
#include <czmq.h>
using namespace std;
static void *
worker_routine (void *arg) {
    //  Socket to talk to dispatcher
    szmq::Context *context = (szmq::Context *) arg; 
    szmq::Socket<ZMQ_REP, szmq::ZMQ_CLIENT> receiver(*context);
    receiver.connect(szmq::SocketUrl("inproc://workers"));

    while (1) {
        auto buffer = receiver.recvOne();
        cout << "received request: " << buffer.read<std::string>() << endl;
        zclock_sleep(1000);    //  Do some 'work'
        receiver.sendOne(szmq::Message::from("World"));
    }
    receiver.close();
    return NULL;
}

int main (void)
{
    szmq::Context context;

    //  Socket to talk to clients
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> clients(context);   
    clients.bind(szmq::SocketUrl("tcp://*:5555")); 
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> workers(context);
    workers.bind(szmq::SocketUrl("inproc://workers"));

    //  Launch pool of worker threads
    int thread_nbr;
    for (thread_nbr = 0; thread_nbr < 5; thread_nbr++) {
        zthread_new(worker_routine, (void *)&context);
    }
    //  Connect work threads to client threads via a queue proxy
    szmq::proxy(reinterpret_cast<void*>(*clients), reinterpret_cast<void*>(*workers), NULL);

    //  We never get here, but clean up anyhow
    clients.close();
    workers.close();
    return 0;
}
```


이제 코드를 보면 무엇을 하는지 아실 수 있으실 겁니다. 작동 방법 :

* 서버(mtserver)는 일련의 작업자 스레드들(worker threads)를 시작합니다. 각 작업자 스레드는 REP 소켓을 만들어 연결하여 소켓의 요청을 처리합니다. 작업자 스레드는 단일 스레드 서버와 같습니다. 유일한 차이점은 전송방식(tcp 대신 inproc)과 바인드-연결(bind-connect) 방향입니다.
* 서버는 클라이언트(hwclient)와 통신하기 위해 ROUTER 소켓을 생성하고 외부 인터페이스(tcp를 통해)에 바인딩(bind)합니다.
* 서버는 작업자들과 통신하기 위해 DEALER 소켓을 생성하고 내부 인터페이스 (inproc 통해)에 바인딩(bind)합니다.
* 서버는 두 소켓(ROUTER-DEALER)을 연결하는 프록시(zmq_proxy())를 시작합니다. 프록시는 모든 클라이언트에서 들어오는 요청을 작업자들에게 배포하며 작업자들의 응답을 클라이언트에 전달합니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc  -MTd mtserver.java szmq.lib pthreadVC2.lib

PS D:\work\sook\src\szmq\examples> ./hwclient
Connecting to hello world server...
Sending Hello : 0
Received : World
Sending Hello : 1
Received : World
Sending Hello : 2
Received : World
..

PS D:\work\sook\src\szmq\examples> ./mtserver
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
~~~

스레드들의 생성은 대부분 프로그래밍 언어에서 이식성이 없다는 점에 유의하십시오. POSIX 라이브러리 pthreads이지만 원도우에서는 다른 API(예 : std::thread 등)를 사용해야 됩니다.

## 스레드들간의 신호(PAIR 소캣)

ØMQ로 멀티스레드 응용프로그램을 만들 때 스레드들 간 협업하는 방법에 대하여 궁금하실 겁니다. sleep() 함수를 삽입하거나 멀티스레딩 기술인 세마포어 혹은 뮤텍스와 같이 사용하고 싶더라도 ØMQ 메시지만 사용해야 합니다. 술꾼과 맥주병 이야기에서 언급한 것처럼 스레드들 간에 메시지 전달만 수행하고 공유하는 상태(& 자원)가 없어야 합니다.

준비되셨다면 서로 신호를 보내는 3개의 스레드를 만들어 봅시다. 예제에서 inproc 전송방식로 PAIR 소켓을 사용합니다.

### mtrelay.java: 멀티 스레드(MT) relay

```java
//  Multithreaded relay

#include <szmq/szmq.h>
#include <iostream>
#include <thread>
using namespace std;

static void *
step1 (void *arg) {
    szmq::Context *context = (szmq::Context *) arg; 
    //  Connect to step2 and tell it we're ready
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> xmitter(*context);
    xmitter.connect(szmq::SocketUrl("inproc://step2"));
    cout << "Step 1 ready, signaling step 2\n";
    xmitter.sendOne(szmq::Message::from("READY"));
    xmitter.close();
    return NULL;
}

static void *
step2 (void *arg) {
    szmq::Context *context = (szmq::Context *) arg; 
    //  Bind inproc socket before starting step1
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> receiver(*context);
    receiver.bind(szmq::SocketUrl("inproc://step2"));
    thread pair(&step1, (void *)context);
    pair.join();
    //  Wait for signal and pass it on
    auto string = receiver.recvOne();
    receiver.close();
    //  Connect to step3 and tell it we're ready
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> xmitter(*context);
    xmitter.connect(szmq::SocketUrl("inproc://step3"));
    cout << "Step 2 ready, signaling step 3\n";
    xmitter.sendOne(szmq::Message::from("READY"));
    xmitter.close();

    return NULL;
}

int main (void)
{
    szmq::Context context;
    //  Bind inproc socket before starting step2
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> receiver(context);   
    receiver.bind(szmq::SocketUrl("inproc://step3")); 
    thread pair(&step2, (void *)&context);
    pair.join();
    //  Wait for signal
    auto string = receiver.recvOne();
    receiver.close();
    cout << "Test successful!\n";
    return 0;
}

```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc  -MTd mtrelay.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./mtrelay
Step 1 ready, signaling step 2
Step 2 ready, signaling step 3
Test successful!
~~~

std::thread에서 join()을 사용한 것은 스레드의 순차적인 처리가 가능하게 하기 위함입니다. 이전 예제에서 detech()는 부모와 자신간의 독자적으로 구동하기 위한 목적으로 자식 스레드에서 "while - loop"가 수행되 경우 적절합니다.

이것은 ØMQ 멀티스레드을 위한 고전적인 패턴입니다.
* 2개의 스레드는 공유 컨텍스트를 통하여 inproc로 통신합니다.
* 부모 스레드는 하나의 소켓을 만들어 inproc://step3(혹은 step2)에 바인딩한 다음, 자식 스레드를 시작하여 컨텍스트를 전달합니다.
* 자식 스레드는 두 번째 소켓을 만들어서 inproc://step3(혹은 step2)에 연결한 다음, 준비된 부모 스레드에 신호를 보냅니다.

이 패턴(ZMQ_PAIR)을 사용하는 멀티스레드 코드는 프로세스에서는 사용할 수 없습니다. inproc 소켓 쌍(PAIR)을 사용하여 강하게 연결된 응용프로그램(스레드가 구조적으로 상호 의존적)을 작성할 수 있습니다. 낮은 지연 시간이 핵심적일 경우 사용하시기 바랍니다. 다른 디자인 패턴으로 느슨하게 연결된 응용프로그램의 경우, 스레드들이 자체 컨텍스트가 있고 ipc 또는 tcp를 통해 통신합니다. 느슨하게 연결된 스레드들은 독립적인 프로세스로 쉽게 분리 할 수 있습니다.

PAIR 소켓을 사용한 예제를 처음으로 보여 주었습니다. PAIR를 사용하는 이유는 다른 소켓 조합으로도 동작할 것 같지만 이러한 신호 인터페이스로 사용하면 부작용이 있습니다.

## 노드 간의 협업

네트워크상에서 노드들 간의 협업하려 하면, PAIR 소켓을 적용할 수 없습니다. 이것은 스레드들과 노드들 간에 전략이 다른 몇 가지 영역 중 하나입니다. 주로 노드들은 왔다 갔다 하고 스레드들은 보통 정적입니다. PAIR 소켓은 원격 노드가 가고 다시 돌아오면 자동으로 재연결하지 않습니다.

스레드들과 노드들 간의 두 번째 중요한 차이점은 일반적으로 스레드들의 수는 고정되어 있지만 노드들의 수는 가변적입니다. 노드 협업을 위해 이전 시나리오(날씨 서버 및 클라이언트)를 가져와서 구독자 시작 시 발행자에 대한 연결에 소요되는 시간으로 인하여 데이터가 유실하지 않도록 하겠습니다.

다음은 응용프로그램 동작 방식입니다.

* 발행자는 미리 예상되는 구독자들의 숫자를 알고 있으며 이것은 어딘가에서 얻을 수 있는 마법 숫자입니다.
* 발행자가 시작하면서 모든 구독자들이 연결될 때까지 기다립니다. 이것이 노드 협업 부분입니다. 각 구독자들이 구독하기 시작하면 다른 소켓으로 발행자에게 준비가 되었음을 알립니다.
* 발행자는 모든 구독자들이 연결되면 데이터 전송을 시작됩니다.

[옮긴이] 마법 숫자(magic number)는 이름 없는 숫자 상수(unnamed numerical constants)로 상황에 따라 상수값이 변경(예 : 하드웨어 아키텍처에 따란 INT는 16 비트 혹은 32 비트로 표현)될 수 있을 때 지정합니다. 현실에서는 구독자들의 숫자가 실제 얼마인지 정확히 알기 어렵기 때문에 예제에서는 정해진 구독자의 숫자를 마법 숫자(magic number)로 지칭합니다.

이 경우에 REP-REQ 소켓을 통하여 구독자들과 발행자 간에 동기화하도록 하겠습니다. 아래는 발행자의 소스 코드입니다.

### syncpub.java: 동기화된 발행자

```java
//  Synchronized publisher

#include <szmq/szmq.h>
#include <iostream>
using namespace std;
#define SUBSCRIBERS_EXPECTED  2  //  We wait for 2 subscribers 

int main (void)
{
    szmq::Context context;

    //  Socket to talk to clients
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(context);
    int sndhwm = 1100000;
    publisher.setSockOpt(ZMQ_SNDHWM, &sndhwm, sizeof (int));
    publisher.bind(szmq::SocketUrl("tcp://*:5561"));

    //  Socket to receive signals
    szmq::Socket<ZMQ_REP, szmq::ZMQ_SERVER> syncservice(context);
    syncservice.bind(szmq::SocketUrl("tcp://*:5562"));

    //  Get synchronization from subscribers
    cout << "Waiting for subscribers\n";
    int subscribers = 0;
    while (subscribers < SUBSCRIBERS_EXPECTED) {
        //  - wait for synchronization request
        auto string = syncservice.recvOne();
        //  - send synchronization reply
        syncservice.sendOne(szmq::Message::from(""));
        subscribers++;
    }
    //  Now broadcast exactly 1M updates followed by END
    cout << "Broadcasting messages\n";
    int update_nbr;
    for (update_nbr = 0; update_nbr < 1000000; update_nbr++)
        publisher.sendOne(szmq::Message::from("Rhubarb"));
    
    publisher.sendOne(szmq::Message::from("END"));

    publisher.close();
    syncservice.close();
    return 0;
}
```

아래는 구독자 소스 코드입니다.

### syncsub.java: 동기화된 구독자

```java
//  Synchronized subscriber

#include <szmq/szmq.h>
#include <iostream>
#include <czmq.h>
using namespace std;

int main (void)
{
    szmq::Context context;

    //  First, connect our subscriber socket
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(context);
    subscriber.connect(szmq::SocketUrl("tcp://localhost:5561"));
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, "", 0);

    //  0MQ is so fast, we need to wait a while...
    zclock_sleep(1000);     //  Do some 

    //  Second, synchronize with publisher
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> syncclient(context);
    syncclient.connect(szmq::SocketUrl("tcp://localhost:5562"));

    //  - send a synchronization request
    syncclient.sendOne(szmq::Message::from(""));

    //  - wait for synchronization reply
    auto string = syncclient.recvOne().read<std::string>();

    //  Third, get our updates and report how many we got
    int update_nbr = 0;
    cout << "Ready to subscribe\n";
    while (1) {
        auto msg = subscriber.recv1().read<std::string>();
        if ((update_nbr % 10000) == 0) cout << ".";
        if (msg.compare("END") == 0) {
            break;
        }
        update_nbr++ ;
    }
    cout << "Received " << update_nbr << " updates\n";
    subscriber.close();
    syncclient.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd syncpub.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd syncsub.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./syncpub
Waiting for subscribers
Broadcasting messages

PS D:\work\sook\src\szmq\examples> ./syncsub
Received 1000000 updates
~~~

* 예제 수행시 syncpub에서 1백만개의 메세지를 전송하면 syncsub에서  0.3백만개의 메세지만 받는 경우가 발생하는 경우, 원인은 syncsub가 syncpub의 메세지 전송 속도를 따라가지 못해 메세지 유실이 발생하기 때문입니다. 해결 방안으로 syncpub에서 메세지 전송 지연을 두거나 syncsub의 처리 속도를 개선할 수 있습니다.

‘syncsub’에서 REQ/REP 통신가 완료될 때까지 SUB 연결이 완료되지 않는다고 하면 inproc 이외의 전송을 사용하는 경우 아웃바운드 연결이 어떤 순서로든 완료된다는 보장은 없습니다. 따라서 예제에서는 `syncsub’에서 SUB 소켓 연결하고 REQ/REP 동기화 전송 사이에 1초의 대기를 수행하였습니다.

## 발행-구독 메시지 봉투
발행-구독 패턴에서, 키를 개별 메시지 프레임으로 분할하고 봉투라고 했습니다. 발행-구독 봉투를 사용하기 위해서는 직접 만들어야 합니다. 선택 사항으로 이전 발행-구독 예제에서는 이런 작업을 수행하지 않았습니다. 발행-구독 봉투를 이용하려면 약간의 코드를 추가해야 합니다. 키와 데이터를 별도의 메시지 프레임에 나누어서 있어 실제 코드를 보면 바로 이해할 수 있습니다.

구독자에서 구독시 필터를 사용하여 접두사 일치 수행하는 것을 기억하십시오. 즉, “XYZ로 시작하는 모든 메시지”를 찾아야 합니다. 분명한 질문으로 실수로 접두사 일치가 데이터와 일치하지 않도록 데이터에서 키를 구분하는 방법입니다. 가장 좋은 방법은 봉투를 사용하여 프레임 경계를 넘지 않으므로 일치 여부 확인하는 것입니다. 최소 예제로 발행-구독 봉투가 코드로 구현하였으며 발행자는 A와 B의 두 가지 유형의 메시지를 보냅니다.

봉투에는  A와 B의 두 가지 메시지 유형이 있습니다.

### psenvpub.java: Pub-Sub 봉투 발행자(enveloper publisher)

```java
//  Pubsub envelope publisher
//  Note that the zhelpers.h file also provides s_sendmore

#include <szmq/szmq.h>
#include <chrono>
#include <thread>
using namespace std;

int main (void)
{
    //  Prepare our context and publisher
    szmq::Context context;
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(context);
    publisher.bind(szmq::SocketUrl("tcp://*:5563"));

    while (1) {
        //  Write two messages, each with an envelope and content
        publisher.sendMore(szmq::Message::from("A"));
        publisher.sendOne(szmq::Message::from("We don't want to see this"));
        publisher.sendMore(szmq::Message::from("B"));
        publisher.sendOne(szmq::Message::from("We would like to see this"));
        this_thread::sleep_for(chrono::milliseconds(1000));  //  Do the work
    }
    //  We never get here, but clean up anyhow
    publisher.close();
    return 0;
}
```

구독자(subscriber)는 B 유형의 메시지만 원합니다.

### psenvsub.java : Pub-Sub 봉투 구독자(enveloper subscriber)

```java
//  Pubsub envelope subscriber

#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
using namespace std;

int main (void)
{
    //  Prepare our context and subscriber
    szmq::Context context;
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(context);
    subscriber.connect(szmq::SocketUrl("tcp://localhost:5563"));
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, "B", 1);

    while (1) {
        //  Read envelope with address
        auto address = subscriber.recvOne().read<std::string>();
        auto contents = subscriber.recvOne().read<std::string>();
        cout << boost::format("[%1%] %2% \n") % address % contents;
    } 
    //  We never get here, but clean up anyhow
    subscriber.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd psenvpub.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd psenvsub.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./psenvpub

PS D:\work\sook\src\szmq\examples> ./psenvsub
[B] We would like to see this
[B] We would like to see this
[B] We would like to see this
...
~~~

예제에서 구독 필터가 전체 멀티 파트 메시지(키와 데이터)를 거부하거나 수락함을 보여줍니다. 멀티파트 메시지의 일부만 받지 않습니다. 

# 3장-고급 요청-응답 패턴

2장-소켓 및 패턴에서 우리는 ØMQ의 새로운 측면에 대하여 일련의 작은 응용프로그램을 개발하여 ØMQ 기본적인 사용 방법을 살펴보았습니다. 3장에서는 ØMQ의 핵심 요청-응답 패턴 위에 만들어진 고급 패턴을 알아보며 작은 응용프로그램을 개발해 보겠습니다.

## 요청-응답 메커니즘

이미 멀티파트 메시지를 간략히 알아보았습니다. 이제 주요 사용 사례인 응답 메시지 봉투를 알아보겠습니다. 봉투는 데이터 자체를 건드리지 않고 하나의 주소로 데이터를 안전하게 포장하는 방법입니다. 봉투에서 응답 주소를 분리하여 메시지 내용 또는 구조에 관계없이 주소를 생성, 읽기 및 제거하는 API 및 프록시와 같은 범용 중개자를 작성할 수 있습니다.

요청-응답 패턴에서 봉투는 응답을 위한 반송 주소를 가지고 있습니다. 이것은 상태가 없는 ØMQ 네트워크가 어떻게 왕복 요청-응답 대화를 수행하는 방법입니다.

REQ 및 REP 소켓을 사용하면 봉투들을 볼 수 조차 없습니다. REQ/REP 소켓들은 자동으로 봉투를 처리하지만 대부분의 흥미로운 요청-응답 패턴의 경우 특히 ROUTER 소켓에서 봉투를 이해하고 싶을 것입니다. 우리는 단계적으로 작업해 나가겠습니다.

### 간단한 응답 봉투

요청-응답 교환은 요청 메시지와 이에 따른 응답 메시지로 구성됩니다. 간단한 요청-응답 패턴에는 각 요청에 대해 하나의 응답이 있습니다. 고급 패턴에서는 요청과 응답이 비동기적으로 동작할 수 있지만 응답 봉투는 항상 동일한 방식으로 동작합니다.

ØMQ 응답 봉투는 공식적으로 0개 이상의 회신 주소, 공백 구분자, 메시지 본문(0개 이상의 프레임들)으로 구성됩니다. 봉투는 순차적으로 여러 소켓에 의해 생성되어 함께 동작합니다. 우리는 이것을 구체적으로 살펴보겠습니다.

REQ 소켓을 통해 “Hello”를 보내는 것으로 시작하겠습니다. REQ 소켓은 주소가 없고 공백 구분자 프레임과 “Hello” 문자열이 포함된 메시지 프레임만 있는 가장 간단한 응답 봉투를 생성합니다. 이것은 2개의 프레임으로 구성된 메시지입니다.

REP 소켓은 일치하는 작업을 수행합니다. 봉투에서 공백 구분자 프레임 제거하고 전체 봉투를 저장한 후 “Hello” 문자열을 응용프로그램에 전달합니다. 따라서 우리의 원래 Hello World 예제는 내부적으로 요청-응답 봉투를 처리하여 응용프로그램에서는 이를 보지 못했습니다.

hwclient와 hwserver 사이에 흐르는 네트워크 데이터를 감시한다면, 보게 될 것은 다음과 같습니다. : 모든 요청과 모든 응답은 사실 두 개의 프레임으로 “하나의 공백 프레임”과 “본문”입니다. 간단한 REQ-REP 대화에는 별 의미가 없는 것 같지만 ROUTER와 DEALER가 봉투를 다루는 방법을 살펴보면 그 이유를 알 수 있습니다.

* 요청시 : hwclient-[“Hello”]->REQ->[””+”Hello”]->REP->[“Hello”]->hwserver
+ 응답시 : hwserver-[“World”]->REP-[””+”World”]->REQ-[“World”]->hwclient

ROUTER 소켓은 다른 소켓과 달리 모든 연결을 추적하고 호출자에게 이에 대해 알려줍니다. 호출자에게 알리는 방법은 수신된 각 메시지 선두에 연결 식별자(ID)를 붙이는 것입니다. 때때로 주소라고도 할 수 있는 식별자(ID)는 “이것은 연결에 대한 고유한 핸들”이란 의미를 가진 이진 문자열(binary string)입니다. 그래서 ROUTER 소켓을 통해 메시지를 보낼 때 먼저 식별자(ID) 프레임을 보냅니다.

메시지를 수신할 때 ZMQ_ROUTER 소켓은 메시지를 응용프로그램에 전달하기 전에 발신 상대의 식별자(ID)를 포함하는 메시지 부분을 메시지 선두에 추가해야 합니다. 수신된 메시지는 연결된 모든 상대 사이의 공정 대기열에 놓입니다. ZMQ_ROUTER 소켓에서 메시지를 보낼 때 메시지의 첫 번째 부분을 제거하고 이를 사용하여 메시지가 라우팅 될 상대의 식별자(ID)가 되게 합니다.

식별자들(IDs)은 이해하기 어려운 개념이지만 ØMQ 전문가가 되고 싶다면 필수입니다. ROUTER 소켓은 작동하는 각 연결에 대해 임의의 식별자(ID)를 만듭니다. ROUTER 소켓에 3개의 REQ 소켓이 연결되어 있는 경우 각각의 REQ 소켓에 대해 하나씩 3개의 임의의 ID를 생성합니다.

동작 가능한 예제로 계속하면 REQ 소켓에 3 바이트 식별자(ID) ABC가 있다고 하면 내부적으로 ROUTER 소켓이 ABC를 검색할 수 있는 해쉬 테이블 가지고 REQ 소켓에 대한 TCP 연결을 찾을 수 있게 합니다.

송/수신시에 프레임 구성
* 송신 : APP -[“Hello”]-> REQ -[””+”Hello”]-> ROUTER -[ID+””+”Hello”]-> DEALER -[ID+””+”Hello”]-> REP -[“Hello”]-> APP
* 수신 : APP -[“World”]-> REP -[ID+””+”World”]-> DEALER -[ID+””+”World”]-> ROUTER -[””+”World”]-> REQ -[“World”]-> APP

### test_frame.java Hello World 예제 프로그램의 프레임 흐름

```java
//  Multithreaded Hello World server
#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <thread>
#include <boost/format.hpp>
using namespace std;

#define NBR_THREADS 1
static void *
worker_routine (void *arg) {
    //  Socket to talk to dispatcher
    szmq::Context *context = (szmq::Context *) arg; 
    szmq::Socket<ZMQ_REP, szmq::ZMQ_CLIENT> receiver(*context);
    receiver.connect(szmq::SocketUrl("inproc://workers"));
    while (1) {
        while (1) {
            //  Process all parts of the message
            auto msg = receiver.recvOne();
            cout << boost::format("[DEALER->REP][%1%] %2% \n") % msg.size() % szmq::strhex(msg.read<std::string>());   
            if(!receiver.hasMore())
                break;
        }
        //  Do some 'work'
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));    //  Do some 'work'
        //  Send reply back to client
        receiver.sendOne(szmq::Message::from("World"));
    }
    receiver.close();
    return NULL;
}

static void *
client_routine (void *arg) {
    //  Socket to talk to dispatcher   
    szmq::Context *context = (szmq::Context *) arg; 
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> sender(*context);
    sender.connect(szmq::SocketUrl("inproc://clients"));
    while (1) {        
        //  Send request to worker
        sender.sendOne(szmq::Message::from("Hello"));
        while (1) {
            //  Process all parts of the message
            auto msg = sender.recvOne();
            cout << boost::format("[ROUTER->REQ][%1%] %2% \n") % msg.size() % szmq::strhex(msg.read<std::string>());   
            if(!sender.hasMore())
                break;
        }
    }
    sender.close();
    return NULL;
}

int main (void)
{
    szmq::Context context;
    //  Socket to talk to clients
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);    
    frontend.bind(szmq::SocketUrl("inproc://clients"));
    //  Socket to talk to workers
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> backend(context);
    backend.bind(szmq::SocketUrl("inproc://workers"));

    int thread_nbr;
    //  Launch pool of client threads
    for (thread_nbr = 0; thread_nbr < NBR_THREADS; thread_nbr++) {     
        thread client(&client_routine, (void *)&context);   
        client.detach();
    }
   //  Launch pool of worker threads
    for (thread_nbr = 0; thread_nbr < NBR_THREADS; thread_nbr++) {        
        thread worker(&worker_routine, (void *)&context);   
        worker.detach();
    }

    //  Initialize poll set
    std::vector<szmq::PollItem> pollItems = {
		{reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN, 0},
        {reinterpret_cast<void*>(*backend), 0, ZMQ_POLLIN, 0}};

    //  Connect work threads to client threads via a queue proxy
    while (1) {        
        szmq::poll(pollItems, 2, -1);
        if (pollItems [0].revents & ZMQ_POLLIN) {
            //  Process all parts of the message
            while (1) {
                //  Process all parts of the message
                auto msg = frontend.recvOne();
                cout << boost::format("[ROUTER->DEALER][%1%] %2% \n") % msg.size() % szmq::strhex(msg.read<std::string>());    
                bool more = frontend.hasMore();
                if(more)
                    backend.sendMore(msg);
                else{
                    backend.sendOne(msg);
                    break;       //  Last message part
                }
            }
        }
        if (pollItems [1].revents & ZMQ_POLLIN) {
            while (1) {
                //  Process all parts of the message
                auto msg = backend.recvOne();
                cout << boost::format("[DEALER->ROUTER][%1%] %2% \n") % msg.size() % szmq::strhex(msg.read<std::string>());    
                bool more = backend.hasMore();
                if(more)
                    frontend.sendMore(msg);
                else{
                    frontend.sendOne(msg);
                    break;       //  Last message part
                }
            } 
        }
    }
    frontend.close();
    backend.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd test_frame.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./test_frame
[ROUTER->DEALER][5]    )
[ROUTER->DEALER][0]
[ROUTER->DEALER][5] Hello
[DEALER->REP][5] Hello
[DEALER->ROUTER][5]    )
[DEALER->ROUTER][0]
[DEALER->ROUTER][5] World
[ROUTER->REQ][5] World
~~~

다음 예제는 멀티파트 메세지를 처리할 수 있도록 수정한 예제입니다.

### test_frame1.java Hello World 멀티 파트 메세지 처리 예제

```java
//  Multithreaded Hello World server
#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <thread>
#include <boost/format.hpp>
using namespace std;

#define NBR_THREADS 1
static void *
worker_routine (void *arg) {
    //  Socket to talk to dispatcher
    szmq::Context *context = (szmq::Context *) arg; 
    szmq::Socket<ZMQ_REP, szmq::ZMQ_CLIENT> receiver(*context);
    receiver.connect(szmq::SocketUrl("inproc://workers"));
    while (1) {
        //  Process all parts of the message
        auto msg = receiver.recvOne();
        cout << boost::format("[REP->SERVER][%1%] %2% \n") % msg.size() % msg.read<std::string>(); 
        //  Do some 'work'
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));    //  Do some 'work'
        //  Send reply back to client
        receiver.sendOne(szmq::Message::from("World"));
    }
    receiver.close();
    return NULL;
}

static void *
client_routine (void *arg) {
    //  Socket to talk to dispatcher   
    szmq::Context *context = (szmq::Context *) arg; 
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> sender(*context);
    sender.connect(szmq::SocketUrl("inproc://clients"));
    while (1) {        
        //  Send request to worker
        sender.sendOne(szmq::Message::from("Hello"));
        //  Process all parts of the message
        auto msg = sender.recvOne();
        cout << boost::format("[REQ->CLIENT][%1%] %2% \n") % msg.size() % msg.read<std::string>();  
    }
    sender.close();
    return NULL;
}

int main (void)
{
    szmq::Context context;
    //  Socket to talk to clients
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);    
    frontend.bind(szmq::SocketUrl("inproc://clients"));
    //  Socket to talk to workers
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> backend(context);
    backend.bind(szmq::SocketUrl("inproc://workers"));

    int thread_nbr;
    //  Launch pool of client threads
    for (thread_nbr = 0; thread_nbr < NBR_THREADS; thread_nbr++) {     
        thread client(&client_routine, (void *)&context);   
        client.detach();
    }
   //  Launch pool of worker threads
    for (thread_nbr = 0; thread_nbr < NBR_THREADS; thread_nbr++) {        
        thread worker(&worker_routine, (void *)&context);   
        worker.detach();
    }

    //  Initialize poll set
    std::vector<szmq::PollItem> pollItems = {
		{reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN, 0},
        {reinterpret_cast<void*>(*backend), 0, ZMQ_POLLIN, 0}};

    //  Connect work threads to client threads via a queue proxy
    while (1) {        
        szmq::poll(pollItems, 2, -1);
        if (pollItems [0].revents & ZMQ_POLLIN) {
            //  Process all parts of the message
            auto msgs = frontend.recvMultiple();
            for (auto it = begin (msgs); it != end (msgs); ++it) 
                cout << boost::format("[ROUTER->DEALER][%1%] %2% \n") % it->size() % it->read<std::string>();    
            backend.sendMultiple(msgs);
        }
        if (pollItems [1].revents & ZMQ_POLLIN) {
            //  Process all parts of the message
            auto msgs = backend.recvMultiple();
            for (auto it = begin (msgs); it != end (msgs); ++it) 
                cout << boost::format("[DEALER->ROUTER][%1%] %2% \n") % it->size() % it->read<std::string>();
            frontend.sendMultiple(msgs);
        }
    }
    frontend.close();
    backend.close();
    return 0;

```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd test_frame1.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./test_frame1
[ROUTER->DEALER][5]    )
[ROUTER->DEALER][0]
[ROUTER->DEALER][5] Hello
[DEALER->REP][5] Hello
[DEALER->ROUTER][5]    )
[DEALER->ROUTER][0]
[DEALER->ROUTER][5] World
[ROUTER->REQ][5] World
~~~

특수 문자를 HEX 코드로 변환하여 출력할 수 있는 메세지 객체의 dump() 함수를 사용하도록 변경합니다.

### test_frame2.java Hello World 특수 문자를 HEX로 변경하는 dump() 사용

```java
//  Multithreaded Hello World server
#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <thread>
#include <boost/format.hpp>
using namespace std;

#define NBR_THREADS 1
static void *
worker_routine (void *arg) {
    //  Socket to talk to dispatcher
    szmq::Context *context = (szmq::Context *) arg; 
    szmq::Socket<ZMQ_REP, szmq::ZMQ_CLIENT> receiver(*context);
    receiver.connect(szmq::SocketUrl("inproc://workers"));
    while (1) {
        //  Process all parts of the message
        auto msg = receiver.recvOne();
        msg.dump();
        //  Do some 'work'
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));    //  Do some 'work'
        //  Send reply back to client
        receiver.sendOne(szmq::Message::from("World"));
    }
    receiver.close();
    return NULL;
}

static void *
client_routine (void *arg) {
    //  Socket to talk to dispatcher   
    szmq::Context *context = (szmq::Context *) arg; 
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> sender(*context);
    sender.connect(szmq::SocketUrl("inproc://clients"));
    while (1) {        
        //  Send request to worker
        sender.sendOne(szmq::Message::from("Hello"));
        //  Process all parts of the message
        auto msg = sender.recvOne();
        msg.dump();
    }
    sender.close();
    return NULL;
}

int main (void)
{
    szmq::Context context;
    //  Socket to talk to clients
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);    
    frontend.bind(szmq::SocketUrl("inproc://clients"));
    //  Socket to talk to workers
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> backend(context);
    backend.bind(szmq::SocketUrl("inproc://workers"));

    int thread_nbr;
    //  Launch pool of client threads
    for (thread_nbr = 0; thread_nbr < NBR_THREADS; thread_nbr++) {     
        thread client(&client_routine, (void *)&context);   
        client.detach();
    }
   //  Launch pool of worker threads
    for (thread_nbr = 0; thread_nbr < NBR_THREADS; thread_nbr++) {        
        thread worker(&worker_routine, (void *)&context);   
        worker.detach();
    }

    //  Initialize poll set
    std::vector<szmq::PollItem> pollItems = {
		{reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN, 0},
        {reinterpret_cast<void*>(*backend), 0, ZMQ_POLLIN, 0}};

    //  Connect work threads to client threads via a queue proxy
    while (1) {        
        szmq::poll(pollItems, 2, -1);
        if (pollItems [0].revents & ZMQ_POLLIN) {
            //  Process all parts of the message
            auto msgs = frontend.recvMultiple();
            for (auto it = begin (msgs); it != end (msgs); ++it) 
                it->dump();
            backend.sendMultiple(msgs);
        }
        if (pollItems [1].revents & ZMQ_POLLIN) {
            //  Process all parts of the message
            auto msgs = backend.recvMultiple();
            for (auto it = begin (msgs); it != end (msgs); ++it) 
                  it->dump();
            frontend.sendMultiple(msgs);
        }
    }
    frontend.close();
    backend.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd test_frame2.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./test_frame2
PS D:\work\sook\src\szmq\examples> ./test_frame2
[005]0080000029
[000]
[005]Hello
[005]Hello
[005]0080000029
[000]
[005]World
[005]World
...
~~~

* REQ 소켓은 메시지 데이터 앞에 공백 구분자 프레임을 넣어 네트워크로 보냅니다. REQ 소켓은 동기식으로 REQ 소켓은 항상 하나의 요청을 보내고 하나의 응답을 기다립니다. REQ 소켓은 한 번에 하나의 상대와 통신합니다. REQ 소켓을 여러 상대에 연결하려면, 요청들은 한 번에 하나씩 각 상대에 분산되고 응답이 예상됩니다.
* REP 소켓은 모든 식별자(ID) 프레임과 공백 구분자를 읽고 저장한 다음 프레임을 호출자에게 전달합니다. REP 소켓은 동기식이며 한 번에 하나의 상대와 통신합니다. REP 소켓을 여러 개의 단말들에 연결하면 요청들은 상대로부터 공정한 형태로 읽히고, 응답은 항상 마지막 요청을 한 동일한 상대로 전송됩니다.
* DEALER 소켓은 응답 봉투를 인식하지 못하며 응답 봉투를 멀티파트 메시지처럼 처리합니다. DEALER 소켓은 비동기식이며 PUSH와 PULL이 결합된 것과 같으며 모든 연결 간에 보내진 메시지(ROUTER->DEALER)를 배포하고 모든 연결에서 받은 메시지(REP->DEALER)를 순서대로 공정 대기열에 보관합니다.
* ROUTER 소켓은 DEALER처럼 응답 봉투를 인식하지 못합니다. ROUTER는 연결에 대한 식별자(ID)를 만들고 수신된 메시지의 첫 번째 프레임으로 생성한 식별자(ID)를 호출자에게 전달합니다. 반대로 호출자가 메시지를 보낼 때 첫 번째 메시지 프레임을 식별자(ID)로 사용하여 보낼 연결을 찾습니다. ROUTER는 비동기로 동작합니다.

### mtserver_client.java : 다중 서버와 클라이언트 스레드를 통한 테스트

mtserver 예제는 여러 개(예 : 10개)의 hwclient 요청을 ROUTER 소켓(asynchronous server)에서 받아 프록시(zmq_proxy())로 DEALER 소켓으로 전달하면 5개의 worker 스레드들이 받아 응답하는 형태였으며 worker에 대하여서만 스레드 구성하였으나, 다음 예제에서는 client도 스레드로 구성하여 테스트 하였습니다.

```java
//  Multithreaded Hello World server

#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <thread>
using namespace std;

static void *
client_routine (void *arg) {
    //  Socket to talk to dispatcher   
    szmq::Context *context = (szmq::Context *) arg; 
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> sender(*context);
    sender.connect(szmq::SocketUrl("inproc://clients"));
    while (1) {        
        //  Send request to worker
        sender.sendOne(szmq::Message::from("Hello"));
        //  Process all parts of the message
        auto buffer = sender.recvOne();
        cout << "received reply: " << buffer.read<std::string>() << endl;
    }
    sender.close();
    return NULL;
}
static void *
worker_routine (void *arg) {
    //  Socket to talk to dispatcher
    szmq::Context *context = (szmq::Context *) arg; 
    szmq::Socket<ZMQ_REP, szmq::ZMQ_CLIENT> receiver(*context);
    receiver.connect(szmq::SocketUrl("inproc://workers"));

    while (1) {
        auto buffer = receiver.recvOne();
        cout << "received request: " << buffer.read<std::string>() << endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));      //  Do some 'work'
        receiver.sendOne(szmq::Message::from("World"));
    }
    receiver.close();
    return NULL;
}

int main (void)
{
    szmq::Context context;

    //  Socket to talk to clients
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> clients(context);   
    clients.bind(szmq::SocketUrl("inproc://clients")); 
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> workers(context);
    workers.bind(szmq::SocketUrl("inproc://workers"));

    //  Launch pool of worker threads
    int thread_nbr;
    for (thread_nbr = 0; thread_nbr < 5; thread_nbr++) {
        thread worker(&worker_routine, (void *)&context);
        worker.detach();
    }
    //  Launch pool of clients threads
    for (thread_nbr = 0; thread_nbr < 5; thread_nbr++) {
        thread client(&client_routine, (void *)&context);
        client.detach();
    }

    //  Connect work threads to client threads via a queue proxy
    szmq::proxy(reinterpret_cast<void*>(*clients), reinterpret_cast<void*>(*workers), NULL);

    //  We never get here, but clean up anyhow
    clients.close();
    workers.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc  mtserver_client.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./mtserver_client
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received reply: World
received reply: World
received reply: World
received reply: World
received request: Helloreceived request: Hello
received request: Hello
...
~~~

## 식별자와 주소

ØMQ에서 식별자(ID)의 개념은 ROUTER 소켓이 다른 소켓에 대한 연결을 식별하는 방법입니다. 더 광범위하게, 식별자들(IDs)은 응답 봉투의 주소로 사용됩니다. 대부분의 경우, 식별자(ID)는 로컬에서 ROUTER 소켓에 대한 해쉬 테이블의 색인 키로 사용됩니다. 독자적으로 상대는 물리적인 주소(네트워크의 단말 “tcp : //192.168.55.117 : 5670”)와 논리적인 주소(UUID 혹은 이메일 주소, 다른 고유한 키)를 가질 수 있습니다.

이는 규칙을 뒤집고 상대가 ROUTER에 연결될 때까지 기다리지 않고 ROUTER를 상대에 연결하는 경우에도 마찬가지입니다. 그러나 ROUTER 소켓이 식별자(ID) 대신 논리 주소를 사용하도록 강제할 수 있습니다. zmq_setsockopt() 참조 페이지는 소켓 식별자(ID) 설정하는 방법이 설명되어 있으며 다음과 같이 작동합니다.

* ROUTER에 바인딩 혹은 연결하려는 상대 응용프로그램 소켓(DEALER 또는 REQ)에서 대한 옵션 ZMQ_IDENTITY 옵션을 설정합니다.
* 일반적으로 상대는 이미 바인딩된 ROUTER 소켓에 연결합니다. 그러나 ROUTER도 상대에 연결할 수도 있습니다.
* 연결 수행 시, 상대 소켓(REQ 혹은 DEALER)은 ROUTER 소켓에 “연결에 대한 식별자(ID)로 사용하십시오”라고 알려줍니다.
* 상대 소켓(REQ 혹은 DEALER)이 그렇게 하지 않으면, ROUTER는 연결에 대해 일반적인 임의의 식별자(ID)를 생성합니다.
* 이제 ROUTER 소켓은 해당 상대에서 들어오는 모든 메시지에 대하여 접두사 식별자 프레임으로 논리 주소를 응용프로그램에 제공합니다.
* ROUTER는 모든 보내는 메시지에 대하여서도 접두사 식별자 프레임을 논리 주소로 사용됩니다.

사용 예 : client.setSockOpt (ZMQ_IDENTITY, "PEER1", 5);

다음은 ROUTER 소켓에 연결하는 2개의 상대의 간단한 예입니다. 하나는 논리적 주소 “PEER2”를 부과합니다.

### identity.java: 식별자 검사

```java
//  Demonstrate request-reply identities

#include <szmq/szmq.h>
#include <iostream>
using namespace std;

int main (void) 
{
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> sink(context);
    sink.bind(szmq::SocketUrl("inproc://example"));

    //  First allow 0MQ to set the identity
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> anonymous(context);
    anonymous.connect(szmq::SocketUrl("inproc://example"));
    anonymous.sendOne(szmq::Message::from("ROUTER uses a generated 5 byte identity"));
    auto msgs = sink.recvMultiple();
    for (auto it = begin (msgs); it != end (msgs); ++it) 
        it->dump();

    //  Then set the identity ourselves
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> identified(context);
    identified.setSockOpt(ZMQ_IDENTITY, "PEER2", 5);
    identified.connect(szmq::SocketUrl("inproc://example"));
    identified.sendOne(szmq::Message::from("ROUTER socket uses REQ's socket identity"));
    msgs = sink.recvMultiple();
    for (auto it = begin (msgs); it != end (msgs); ++it) 
        it->dump();

    sink.close();
    anonymous.close();
    identified.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd identity.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./identity
[005]0080000029
[000]
[027]ROUTER uses a generated 5 byte identity
[005]PEER2
[000]
[028]ROUTER socket uses REQ's socket identity
~~~

## 부하 분산 패턴

이제 몇 가지 코드를 살펴보겠습니다. ROUTER 소켓에 REQ 소켓을 연결 한 다음 DEALER 소켓에 연결하는 방법을 살펴보겠습니다(REQ-ROUTER-DEALER). 2개의 예제는 부하 분산 패턴과 동일한 처리 방법을 따릅니다. 이 패턴은 단순히 응답 채널 역할을 하는 것보다 계획적인 라우팅을 위해 ROUTER 소켓을 사용하는 첫 번째 사례입니다.

부하 분산 패턴은 매우 일반적이며 이 책에서 여러 번 보았습니다. 간단한 라운드 로빈 라우팅(PUSH 및 DEALER 제공하는 것처럼)으로 주요 문제를 해결하였지만 라운드 로빈은 작업의 처리 시간이 고르지 않은 경우 비효율적이 될 수 있습니다.

우체국에 비유하면, 카운터 당 대기열이 하나 있고 우표를 사는 사람(빠르고 간단한 거래)이 있고 새 계정을 여는 사람(매우 느린 거래)이 있다면 우표 구매자가 대기열에 부당하게 기다리는 것을 발견하게 될 것입니다. 우체국에서와 마찬가지로 메시징 아키텍처가 불공평하면 사람들은 짜증을 낼 것입니다.

우체국에 대한 솔루션은 업무 처리 특성에 따라 느린 작업에 하나 또는 두 개의 카운터을 생성하고, 그 외의 작업은 다른 카운터가 선착순으로 클라이언트에게 서비스를 제공하도록 단일 대기열을 만드는 것입니다.

단순한 접근 방식의 PUSH와 DEALER를 사용하는 한 가지 이유는 순수한 성능 때문입니다. 미국의 주요 공항에 도착하면 이민국에서 사람들이 줄을 서서 기다리는 것을 볼 수 있습니다. 국경 순찰대원은 단일 대기열을 사용하는 대신 각 카운터로 미리 사람들을 보내 대기하게 합니다. 사람들이 미리 50야드를 걷게 하여 승객당 1~2분 시간을 절약 가능한 것은 모든 여권 검사는 거의 같은 시간이 걸리기에 다소 공정합니다. 이것이 PUSH 및 DEALER의 전략입니다. 이동 거리가 줄어들도록 미리 작업 부하를 보냅니다.

이것은 ØMQ에서 되풀이되는 주제입니다 : 세계의 문제는 다양하며 올바른 방법으로 각각 다른 문제를 해결함으로써 이익을 얻을 수 있습니다. 공항은 우체국이 아니며 문제의 규모가 다릅니다.

브로커(ROUTER)에 연결된 작업자(DEALER 또는 REQ) 시나리오로 돌아가 보겠습니다. 브로커는 작업자가 언제 준비되었는지 알고 있어야 하며, 매번 최저사용빈도(LRU) 작업자를 선택할 수 있도록 작업자 목록을 유지해야 합니다.

솔루션은 실제로 간단합니다. 작업자가 시작할 때와 각 작업을 완료한 후에 “ready” 메시지를 보냅니다. 브로커는 이러한 메시지를 하나씩 읽습니다. 메시지를 읽을 때마다 마지막으로 사용한 작업자가 보낸 것입니다. 그리고 우리는 ROUTER 소켓을 사용하기 때문에 작업자에게 작업을 회신할 수 있는 ID가 있습니다.

작업에 대한 결과는 응답이 보내지고, 새로운 작업 요청에 따라 작업에 대한 응답은 보내지기 때문에 요청-응답에 대한 응용입니다. 이들을 이해하기 위하여 예제 코드로 구현하겠습니다.

### rtreq.java: ROUTER 브로커와 REQ 작업자

일련의 REQ 작업자들과 통신하는 ROUTER 브로커를 사용하는 부하 분산 패턴의 예입니다.

```java
// 2015-01-16T09:56+08:00
//  ROUTER-to-REQ example

#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <cstdlib>
#include <thread>
#include <boost/format.hpp>
using namespace std;

#define NBR_WORKERS 10

static void *
worker_task(void *ctx, void *args)
{
    szmq::Context *context = (szmq::Context *) ctx; 
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> worker(*context);

#ifdef _WIN32
    worker.setId((intptr_t)args);
#else
    worker.setId();          //  Set a printable identity.
#endif

    worker.connect(szmq::SocketUrl("inproc://workers"));

    int total = 0;
    while (1) {
        //  Tell the broker we're ready for work
        worker.sendOne(szmq::Message::from("Hi Boss"));

        //  Get workload from broker, until finished
        auto workload = worker.recvOne().read<std::string>();
        int finished = (workload.compare("Fired!") == 0);

        if (finished) {
            cout << boost::format("[%1%] Completed: %2% tasks\n") %(intptr_t)args % total; 
            break;
        }
        total++;

        //  Do some random work
        this_thread::sleep_for(chrono::milliseconds(std::rand() % 500 + 1)); 
    }
    worker.close();
    return NULL;
}

//  .split main task
//  While this example runs in a single process, that is only to make
//  it easier to start and stop the example. Each thread has its own
//  context and conceptually acts as a separate process.

int main(void)
{
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> broker(context); 
    broker.bind(szmq::SocketUrl("inproc://workers"));
    
    std::srand (static_cast<unsigned int>(std::time(0)));

    int worker_nbr;
    for (worker_nbr = 0; worker_nbr < NBR_WORKERS; worker_nbr++) {
        thread worker(&worker_task, (void *)&context, (void *)(intptr_t)worker_nbr);
        worker.detach();
    }
    //  Run for five seconds and then tell workers to end
    chrono::steady_clock::time_point begin = chrono::steady_clock::now();
    int workers_fired = 0;
    while (1) {
        //  Next message gives us least recently used worker
        auto identity = broker.recvOne().read<std::string>();
        broker.sendMore(szmq::Message::from(std::string(identity)));
        broker.recvOne();     //  Envelope delimiter
        broker.recvOne();     //  Response from worker
        broker.sendMore(szmq::Message::from(std::string("")));

        //  Encourage workers until it's time to fire them
        chrono::steady_clock::time_point end = chrono::steady_clock::now();
        auto duration = chrono::duration_cast<chrono::milliseconds>(end - begin).count();
        if (duration < 5000)
            broker.sendOne(szmq::Message::from(std::string("Work harder")));
        else {
            broker.sendOne(szmq::Message::from(std::string("Fired!")));
            if (++workers_fired == NBR_WORKERS)
                break;
        }
    }
    this_thread::sleep_for(chrono::milliseconds(1000)); 
    broker.close();
    return 0;
}
```

예제는 5초 동안 실행된 다음 각 작업자가 처리한 작업의 개수를 출력합니다. ROUTER 소켓을 통한 부하 분산으로 REQ 요청을 처리하면 작업의 공정한 분배를 기대할 수 있습니다.

### 빌드 및 테스트
1. 원도우 환경의 경우

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd rtreq.java szmq.lib
PS D:\work\sook\src\szmq\examples> ./rtreq
[1] Completed: 17 tasks
[0] Completed: 17 tasks
[4] Completed: 17 tasks
[3] Completed: 17 tasks
[2] Completed: 17 tasks
[8] Completed: 17 tasks
[5] Completed: 17 tasks
[6] Completed: 17 tasks
[9] Completed: 17 tasks
[7] Completed: 17 tasks
~~~

2. 리눅스 환경의 경우

~~~{.bash}
zedo@sook:/work/sook/src/szmq/examples$ g++ -o rtreq rtreq.java  -lsook-szmq -lzmq  -lpthread
zedo@sook:/work/sook/src/szmq/examples$ ./rtreq
[5] Completed: 22 tasks
[4] Completed: 22 tasks
[9] Completed: 18 tasks
[2] Completed: 19 tasks
[6] Completed: 17 tasks
[7] Completed: 19 tasks
[1] Completed: 22 tasks
[3] Completed: 24 tasks
[8] Completed: 21 tasks
[0] Completed: 23 tasks
~~~

mtserver_client 예제에서는 REQ 클라이언트 스레드들과 REP 작업자 스레드들과 ROUTER-DEALER 간에 zmq_proxy()을 통해 그대로 전달하여, 식별자(ID) 및 공백 구분자 프레임 없이 데이터만으로 통신이 가능했습니다. 위의 예제는 ROUTER 소켓에서 받은 데이터를 직접 다루기 때문에 3개의 프레임들(ID + empty delimiter + body) 처리가 필요합니다.

## ROUTER 브로커와 DEALER 작업자
어디서든 REQ를 사용할 수 있는 곳이면 DEALER를 사용할 수 있습니다. 2개의 구체적인 차이점이 있습니다.
* REQ 소켓은 모든 데이터 프레임 앞에 공백 구분자 프레임을 보냅니다. DEALER는 그렇지 않습니다.
* REQ 소켓은 응답을 받기 전에 하나의 요청 메시지만 보냅니다. DEALER는 완전히 비동기적입니다.

동기와 비동기 동작은 엄격한 요청-응답을 수행하기 때문에 예제에 영향을 미치지 않습니다. 이는 “4장-신뢰할 수 있는 요청-응답 패턴”에서 다루며 오류 복구를 처리할 때 더 관련이 있습니다.

이제 똑같은 예제지만 REQ 소켓이 DEALER 소켓으로 대체되었습니다.

### rtdealer.java: ROUTER에서 DEALER

```java
// 2015-01-16T09:56+08:00
//  ROUTER-to-DEALER  example

#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <cstdlib>
#include <thread>
#include <boost/format.hpp>
using namespace std;

#define NBR_WORKERS 10

static void *
worker_task(void *ctx, void *args)
{
    szmq::Context *context = (szmq::Context *) ctx; 
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> worker(*context);

#ifdef _WIN32
    worker.setId((intptr_t)args);
#else
    worker.setId();          //  Set a printable identity.
#endif
    worker.connect(szmq::SocketUrl("inproc://workers"));
    int total = 0;
    while (1) {
        //  Tell the broker we're ready for work
        worker.sendMore(szmq::Message::from(""));
        worker.sendOne(szmq::Message::from("Hi Boss"));
        //  Get workload from broker, until finished
        worker.recvOne();   //  Envelope delimiter
        auto workload = worker.recvOne().read<std::string>();
        int finished = (workload.compare("Fired!") == 0);
        if (finished) {
            cout << boost::format("[%1%] Completed: %2% tasks\n") %(intptr_t)args % total; 
            break;
        }
        total++;
        //  Do some random work
        this_thread::sleep_for(chrono::milliseconds(std::rand() % 500 + 1)); 
    }
    worker.close();
    return NULL;
}

//  .split main task
//  While this example runs in a single process, that is only to make
//  it easier to start and stop the example. Each thread has its own
//  context and conceptually acts as a separate process.

int main(void)
{
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> broker(context); 
    broker.bind(szmq::SocketUrl("inproc://workers"));
    std::srand (static_cast<unsigned int>(std::time(0)));

    int worker_nbr;
    for (worker_nbr = 0; worker_nbr < NBR_WORKERS; worker_nbr++) {
        thread worker(&worker_task, (void *)&context, (void *)(intptr_t)worker_nbr);
        worker.detach();
    }
    //  Run for five seconds and then tell workers to end
    chrono::steady_clock::time_point begin = chrono::steady_clock::now();
    int workers_fired = 0;
    while (1) {
        //  Next message gives us least recently used worker
        auto identity = broker.recvOne().read<std::string>();
        broker.sendMore(szmq::Message::from(std::string(identity)));
        broker.recvOne();     //  Envelope delimiter
        broker.recvOne();     //  Response from worker
        broker.sendMore(szmq::Message::from(std::string("")));

        //  Encourage workers until it's time to fire them
        chrono::steady_clock::time_point end = chrono::steady_clock::now();
        auto duration = chrono::duration_cast<chrono::milliseconds>(end - begin).count();
        if (duration < 5000)
            broker.sendOne(szmq::Message::from(std::string("Work harder")));
        else {
            broker.sendOne(szmq::Message::from(std::string("Fired!")));
            if (++workers_fired == NBR_WORKERS)
                break;
        }
    }
    this_thread::sleep_for(chrono::milliseconds(1000)); 
    broker.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd rtdealer.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./rtdealer
[0] Completed: 17 tasks
[3] Completed: 17 tasks
[5] Completed: 17 tasks
[7] Completed: 17 tasks
[6] Completed: 17 tasks
[1] Completed: 17 tasks
[9] Completed: 17 tasks
[8] Completed: 17 tasks
[2] Completed: 17 tasks
[4] Completed: 17 tasks
~~~

rtreq 예제에서 REQ 소켓을 사용한 작업자에서는 공백 구분자 없이 데이터만 송/수신하였지만, DEALER 사용한 작업자에서는 공백 구분자와 데이터 프레임을 송/수신하였습니다. main() 함수의 소스는 변경이 없지만 작업자 스레드에서 공백 구분자를 송/수신하도록 변경되었습니다.

DEALER-ROUTER 송/수신시에 프레임 구성은 다음과 같습니다.
* 송신 : APP-[””+”Hello”]->DEALER-[””+”Hello”]->ROUTER-[ID+””+”Hello”]->APP
* 수신 : APP-[ID+””+”World”]-ROUTER->[ID+””+”World”]->DEALER-[””+”World”]->APP

작업자가 DEALER 소켓을 사용하고 데이터 프레임 이전에 공백 구분자를 읽고 쓴다는 점을 제외하면 코드는 거의 동일합니다. 이것은 REQ 작업자와의 호환성을 유지하기 위한 접근 방식입니다.

DEALER에서 공백 구분자 프레임을 넣은 이유를 기억하십시오. REP 소켓에서 종료되는 다중도약 네트워크 확장 요청을 허용하여 응답 봉투에서 공백 구분자를 분리하여 데이터 프레임을 응용프로그램에 전달할 수 있습니다.

메시지가 REP 소켓을 경유하지 않는다면 양쪽에 공백 구분자 프레임을 생략할 수 있으며 이렇게 함으로 간단합니다. 이것은 순수한 DEALER와 ROUTER 프로토콜을 이용하고 싶은 경우에 일반적인 설계 방법입니다.

DEALER-ROUTER 송/수신시에서 공백을 제거할 경우 프레임 구성은 다음과 같습니다.
* 송신 : APP-[“Hello”]->DEALER-[“Hello”]->ROUTER-[ID+”Hello”]->APP
* 수신 : APP-[ID+”World”]-ROUTER->[ID+”World”]->DEALER-[World”]->APP

### rtdealer1.java : DEALER-ROUTER 송/수신에서 공백을 제거

```java
// 2015-01-16T09:56+08:00
//  ROUTER-to-DEALER  example

#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <cstdlib>
#include <thread>
#include <boost/format.hpp>
using namespace std;

#define NBR_WORKERS 10

static void *
worker_task(void *ctx, void *args)
{
    szmq::Context *context = (szmq::Context *) ctx; 
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> worker(*context);
#ifdef _WIN32
    worker.setId((intptr_t)args);
#else
    worker.setId();          //  Set a printable identity.
#endif
    worker.connect(szmq::SocketUrl("inproc://workers"));
    int total = 0;
    while (1) {
        //  Tell the broker we're ready for work
        worker.sendOne(szmq::Message::from("Hi Boss"));
        //  Get workload from broker, until finished
        auto workload = worker.recvOne().read<std::string>();
        int finished = (workload.compare("Fired!") == 0);
        if (finished) {
            cout << boost::format("[%1%] Completed: %2% tasks\n") %(intptr_t)args % total; 
            break;
        }
        total++;
        //  Do some random work
        this_thread::sleep_for(chrono::milliseconds(std::rand() % 500 + 1)); 
    }
    worker.close();
    return NULL;
}

//  .split main task
//  While this example runs in a single process, that is only to make
//  it easier to start and stop the example. Each thread has its own
//  context and conceptually acts as a separate process.

int main(void)
{
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> broker(context); 
    broker.bind(szmq::SocketUrl("inproc://workers"));
    
    std::srand (static_cast<unsigned int>(std::time(0)));

    int worker_nbr;
    for (worker_nbr = 0; worker_nbr < NBR_WORKERS; worker_nbr++) {
        thread worker(&worker_task, (void *)&context, (void *)(intptr_t)worker_nbr);
        worker.detach();
    }
    //  Run for five seconds and then tell workers to end
    chrono::steady_clock::time_point begin = chrono::steady_clock::now();
    int workers_fired = 0;
    while (1) {
        //  Next message gives us least recently used worker
        auto identity = broker.recvOne().read<std::string>();
        broker.sendMore(szmq::Message::from(std::string(identity)));
        broker.recvOne();     //  Response from worker

        //  Encourage workers until it's time to fire them
        chrono::steady_clock::time_point end = chrono::steady_clock::now();
        auto duration = chrono::duration_cast<chrono::milliseconds>(end - begin).count();
        if (duration < 5000)
            broker.sendOne(szmq::Message::from(std::string("Work harder")));
        else {
            broker.sendOne(szmq::Message::from(std::string("Fired!")));
            if (++workers_fired == NBR_WORKERS)
                break;
        }
    }
    this_thread::sleep_for(chrono::milliseconds(1000)); 
    broker.close();
    return 0;
}
```

### 빌드 및 테스트
~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd rtdealer1.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./rtdealer1
[0] Completed: 17 tasks
[9] Completed: 17 tasks
[2] Completed: 17 tasks
[8] Completed: 17 tasks
[5] Completed: 17 tasks
[3] Completed: 17 tasks
[6] Completed: 17 tasks
[4] Completed: 17 tasks
[7] Completed: 17 tasks
[1] Completed: 17 tasks
~~~

## 부하 분산 메시지 브로커

이전 예제까지 절반 정도 완성되었습니다. 일련의 작업자들을 더미 요청 및 응답으로 관리할 수 있지만 클라이언트와 통신할 방법은 없습니다. 클라이언트 요청을 수락하는 두 번째 프론트엔드 ROUTER 소켓을 추가하고 이전 예제를 프론트엔드에서 백엔드로 메시지를 전환할 수 있는 프록시로 바꾸면, 유용하고 재사용 가능한 작은 부하 분산 메시지 브로커을 가지게 됩니다.

이 브로커는 다음과 같은 작업을 수행합니다.

* 일련의 클라이언트들로부터의 연결을 받습니다.
* 일련의 작업자들로부터의 연결을 받습니다.
* 클라이언트의 요청을 받고 단일 대기열에 보관합니다.
* 부하 분산 패턴을 사용하여 이러한 요청들을 작업자에게 보냅니다.
* 작업자들로부터 응답을 받습니다.
* 이러한 응답을 원래 요청한 클라이언트로 다시 보냅니다.

브로커 코드는 상당히 길지만 이해할 가치가 있습니다.

### lbbroker.java: 부하 분산 브로커

```java
//  Load-balancing broker
//  Clients and workers are shown here in-process

#include <szmq/szmq.h>
#include <iostream>
#include <chrono>
#include <cstdlib>
#include <thread>
#include <queue>
#include <boost/format.hpp>
using namespace std;

#define NBR_CLIENTS 10
#define NBR_WORKERS 3
//  Basic request-reply client using REQ socket
//  Because s_send and s_recv can't handle 0MQ binary identities, we
//  set a printable text identity to allow routing.
//
static void *
client_task(void *args)
{
    szmq::Context context; 
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> client(context);
#ifdef _WIN32
    client.setId((intptr_t)args);
#else
    client.setId();          //  Set a printable identity.
#endif
    client.connect(szmq::SocketUrl("tcp://localhost:5672"));
    client.sendOne(szmq::Message::from("HELLO"));
    auto reply = client.recvOne().read<std::string>();
    cout << boost::format("[%1%] Client: %2% \n") %(intptr_t)args % reply;
    client.close();
    return NULL;
}


//  .split worker task
//  While this example runs in a single process, that is just to make
//  it easier to start and stop the example. Each thread has its own
//  context and conceptually acts as a separate process.
//  This is the worker task, using a REQ socket to do load-balancing.
//  Because s_send and s_recv can't handle 0MQ binary identities, we
//  set a printable text identity to allow routing.

static void *
worker_task(void *args)
{
    szmq::Context context;
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> worker(context);
#ifdef _WIN32
    worker.setId((intptr_t)args);
#else
    worker.setId();          //  Set a printable identity.
#endif
    worker.connect(szmq::SocketUrl("tcp://localhost:5673"));
    //  Tell broker we're ready for work
    worker.sendOne(szmq::Message::from("READY"));
    while (1) {     
        //  Read and save all frames until we get an empty frame
        //  In this example there is only 1, but there could be more
        auto client_id = worker.recvOne();
        {
            auto empty = worker.recvOne();
        }
        //  Get request, send reply
        auto request = worker.recvOne().read<std::string>();
        cout << boost::format("[%1%] Worker: %2% \n") %(intptr_t)args % request;
        auto empty = szmq::Message::from(std::string(""));
        auto reply = szmq::Message::from(std::string("OK"));
        worker.sendMultiple(client_id, empty, reply);
    }
    worker.close();
    return NULL;
}

//  .split main task
//  This is the main task. It starts the clients and workers, and then
//  routes requests between the two layers. Workers signal READY when
//  they start; after that we treat them as ready when they reply with
//  a response back to a client. The load-balancing data structure is 
//  just a queue of next available workers.

int main(void)
{
    //  Prepare our context and sockets
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);    
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> backend(context);
    frontend.bind(szmq::SocketUrl("tcp://*:5672"));
    backend.bind(szmq::SocketUrl("tcp://*:5673"));

    int client_nbr;
    for (client_nbr = 0; client_nbr < NBR_CLIENTS; client_nbr++) {
        thread client(&client_task, (void *)(intptr_t)client_nbr);
        client.detach();
    }
    int worker_nbr;
    for (worker_nbr = 0; worker_nbr < NBR_WORKERS; worker_nbr++) {
        thread worker(&worker_task, (void *)(intptr_t)worker_nbr);
        worker.detach();
    }
    //  Logic of LRU loop
    //  - Poll backend always, frontend only if 1+ worker ready
    //  - If worker replies, queue worker as ready and forward reply
    //    to client if necessary
    //  - If client requests, pop next worker and send request to it
    //
    //  A very simple queue structure with known max size
    std::queue<szmq::Message> worker_queue;
    //  Initialize poll set
    std::vector<szmq::PollItem> pollItems = {
        {reinterpret_cast<void*>(*backend), 0, ZMQ_POLLIN, 0},
        {reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN, 0}}; 
    while (1) {
        //  Poll frontend only if we have available workers
        szmq::poll(pollItems, worker_queue.size() ? 2 : 1, -1);
        //  Handle worker activity on backend
        if (pollItems [0].revents & ZMQ_POLLIN) {
            //  Queue worker identity for load-balancing
            auto worker_id = backend.recvOne();
            worker_queue.push(worker_id);
            this_thread::sleep_for(chrono::milliseconds(10));
            //  Second frame is empty
            {
                auto empty = backend.recvOne();
            }
            //  Third frame is READY or else a client reply identity
            auto client_id = backend.recvOne();
              //  If client reply, send rest back to frontend            
            if ((client_id.read<std::string>().compare("READY")) != 0) {                
                {
                    auto empty = backend.recvOne();
                }
                auto reply = backend.recvOne();
                auto empty = szmq::Message::from(std::string(""));
                frontend.sendMultiple(client_id, empty, reply);
                if (--client_nbr == 0)
                    break;      //  Exit after N messages
                    
            }
        }
        //  .split handling a client request
        //  Here is how we handle a client request:
        if (pollItems [1].revents & ZMQ_POLLIN) {
            //  Now get next client request, route to last-used worker
            //  Client request is [identity][empty][request]
            auto client_id = frontend.recvOne();
            {
                auto empty = frontend.recvOne();
            }
            auto request = frontend.recvOne();
            auto worker_id = worker_queue.front(); //worker_queue [0];
            worker_queue.pop();
            auto empty = szmq::Message::from(std::string(""));
            backend.sendMultiple(worker_id, empty, client_id, empty, request);
        }
    }
    frontend.close();
    backend.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc  lbbroker.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./lbbroker
[0] Worker: HELLO
[2] Worker: HELLO
[3[1] Worker: HELLO
] Client: OK
[0] Worker: HELLO
[0] Client: OK
[8] Client: OK
[2] Worker: HELLO
[1] Worker: HELLO
[5] Client: OK
[0] Worker: HELLO
[2] Client: OK
[[2] Worker: HELLO
7] Client: OK
[9] Client: OK
[1] Worker: HELLO
[0] Worker: HELLO
[6] Client: OK
[4] Client: OK
[1] Client: OK
~~~

* 송/수신 시 구조
  - 송신 : APP(client)->REQ->ROUTER(frontend)->ROUTER(backend)->REQ->APP(worker)
  - 수신 : APP(worker)->REQ->ROUTER(backend)->ROUTER(frontend)->REQ->APP(client)
* 송/수신 시에 멀티파트 메시지 구성
  - 송신 : CLIENT -[“HELLO”]-> REQ -[””+”HELLO”]-> ROUTER -[CID+””+”HELLO”]-> logic -[WID+”“+CID+””+”HELLO”]-> ROUTER -[”“+CID+””+”HELLO”]-> REQ -[CID+””+”HELLO”]-> WORKER
  - 수신 : WORKER -[CID+””+”OK”]-> REQ -[”“+CID+””+”OK”]-> ROUTER ->[WID+”“CID+””+”OK”]-> logic -[CID+””+”OK”]-> ROUTER -[+””+”OK”]-> REQ -[“OK”]-> CLIENT

## ØMQ 고급 API

요청-응답을 패턴에 대한 화제를 벗어나 ØMQ API 자신에 대하여 보도록 하겠습니다. 이러한 우회하는 이유가 있습니다. 우리가 더 복잡한 예제를 작성함에 따라 저수준 ØMQ API가 점점 다루기 힘들기 시작합니다.

jeroMQ는 CZMQ 라이브러리에서 다음 기능을 사용합니다.

* 이식 가능한 시간.
  시간을 밀리초 단위의 해상도나 수 밀리초 간 대기 설정하는 것은 이식성이 없습니다. 현실적인 ØMQ 응용프로그램은 이식 가능한 시간이 필요하며, 고급 API에서 제공해야 합니다.
* zmq_poll()을 대체할 리엑터.
  폴링 루프는 간단하지만 어색합니다. 이것들을 많이 사용하면 동일한 작업을 반복해서 수행합니다 : 타이머를 계산, 소켓이 준비되면 코드를 호출. 소켓 리더와 타이머가 있는 소켓 리엑터로 반복 작업을 줄일 수 있습니다.


### lbbroker2.java: sendMultiple(), recvMultiple() 사용한 부하 분산 브로커

앞의 예제(lbbroker.java)에서는 단일 메세지 전송 형태로 진행했다면, 이번에는 다중 메세지 송/수신을 하도록 수정하였습니다.
czmp의 zlist는 szmq 메세지 객체 처리에 적절하지 않아 제외하고,  zthread는 더 이상 지원하지 않은 관계로 std::thread를 사용하였습니다.
main()에서 backend를 poll 수행시 시간 지연을 두고 있으며, client_task() 스레드가 정상적인 메세지를 수신하고 main()이 종료하게 합니다.
- this_thread::sleep_for(chrono::milliseconds(10));

```java
//  Load-balancing broker
//  Clients and workers are shown here in-process

#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>
#include <thread>
#include <czmq.h>
#include <list>
#include <boost/format.hpp>
using namespace std;

#define NBR_CLIENTS 10
#define NBR_WORKERS 3
#define WORKER_READY   "READY"      //  Signals worker is ready

//  Basic request-reply client using REQ socket
static void *
client_task(void *args)
{
    szmq::Context context; 
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> client(context);
    client.setId((intptr_t)args);
    client.connect(szmq::SocketUrl("tcp://localhost:5672"));
    client.sendOne(szmq::Message::from("HELLO"));
    auto reply = client.recv1().read<std::string>();
    cout << "[" << (intptr_t)args << "]" << " Client : " << reply << endl;
    client.close();
    return NULL;
}

// Worker using REQ socket to do load-balancing
static void *
worker_task(void *args)
{
    szmq::Context context;
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> worker(context);
    worker.setId((intptr_t)args);
    worker.connect(szmq::SocketUrl("tcp://localhost:5673"));
    //  Tell broker we're ready for work
    worker.sendOne(szmq::Message::from(WORKER_READY));
    //  Process messages as they arrive(ClientID + "" + "Hello")
    while (1) {     
        //  Read and save all frames until we get an empty frame
        //  In this example there is only 1, but there could be more
        auto msgs = worker.recvN();
        cout << "[" << (intptr_t)args << "]" << " Worker : " << msgs.back().read<std::string>() << endl; //request("HELLO")
		msgs.pop_back();
		msgs.emplace_back(szmq::Message::from(std::string("OK")));
        worker.sendMultiple(msgs);
    }
    worker.close();
    return NULL;
}

//  .split main task
//  This is the main task. It starts the clients and workers, and then
//  routes requests between the two layers. Workers signal READY when
//  they start; after that we treat them as ready when they reply with
//  a response back to a client. The load-balancing data structure is 
//  just a queue of next available workers.

int main(void)
{
    //  Prepare our context and sockets
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);    
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> backend(context);
    frontend.bind(szmq::SocketUrl("tcp://*:5672"));
    backend.bind(szmq::SocketUrl("tcp://*:5673"));

    std::vector < std::thread > threadPool;
    int client_nbr;
    for (client_nbr = 0; client_nbr < NBR_CLIENTS; client_nbr++) {
		thread client(&client_task, (void *)(intptr_t)client_nbr);
        client.detach();
    }
    int worker_nbr;
    for (worker_nbr = 0; worker_nbr < NBR_WORKERS; worker_nbr++) {
        thread worker(&worker_task, (void *)(intptr_t)worker_nbr);
        worker.detach();
    }
    std::list<szmq::Message> workers;
    //  Initialize poll set
    std::vector<szmq::PollItem> items = {
        {reinterpret_cast<void*>(*backend), 0, ZMQ_POLLIN, 0},
        {reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN, 0}}; 
    while (1) {
        //  Poll frontend only if we have available workers
        szmq::poll(items, workers.size() ? 2 : 1, -1);
        //  Handle worker activity on backend
        if (items [0].revents & ZMQ_POLLIN) {
            //  Queue worker identity for load-balancing
            auto msgs = backend.recvN();
			auto worker_id = msgs.front();
            workers.emplace_back(worker_id);
			msgs.erase(msgs.begin()); // delete the worker_id
			msgs.erase(msgs.begin()); // delete the empty delimiter	
            //  Third frame is READY or else a client reply identity
            auto client_id = msgs.front();
              //  If client reply, send rest back to frontend            
            if ((client_id.read<std::string>().compare(WORKER_READY)) != 0) {  
                frontend.sendMultiple(msgs);	// clientID + "" + reply
                if (--client_nbr == 0)
                    break;      //  Exit after N messages
                cout << "client_nbr : " << client_nbr << endl; 
            }
            zclock_sleep(10);  // need to sync with workers thread
        }
        //  .split handling a client request
        //  Here is how we handle a client request:
        if (items [1].revents & ZMQ_POLLIN) {
            //  Now get next client request, route to last-used worker
            //  Client request is [identity][empty][request]
            auto msgs = frontend.recvN();
            msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
			msgs.insert(msgs.begin(), workers.front());			
            workers.pop_front();
            backend.sendMultiple(msgs);  // WorkerID + "" + ClientID + "" + request 
        }
    }
	//  When we're done, clean up properly
	while (!workers.empty()) {
		workers.pop_back();
	}	
    frontend.close();
    backend.close();
    return 0;
}
```

### 빌드 및 테스트 결과
1. 원도우

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd lbbroker2.java szmq.lib czmq.lib

PS D:\work\sook\src\szmq\examples> ./lbbroker2
Worker: HELLO
Worker: HELLO
Client : Worker: HELLO
OK
Client : OK
Worker: HELLO
Client : OK
Worker: HELLO
Client : OK
Worker: HELLO
Client : OK
Worker: HELLO
Client : OK
Worker: HELLO
Client : OK
Worker: HELLO
Client : Worker: HELLO
OK
Client : OK
Client : OK
~~~

2. 리눅스(Ubuntu)
~~~{.bash}
zedo@sook:/work/sook/src/szmq/examples$ g++ -o lbbroker2 lbbroker2.java -lsook-szmq -lzmq -lpthread
zedo@sook:/work/sook/src/szmq/examples$ ./lbbroker
[1] Worker: HELLO 
[2] Worker: HELLO 
[0] Client: OK 
[0] Worker: HELLO 
[6] Client: OK 
[1] Worker: HELLO 
[9] Client: OK 
[2] Worker: HELLO 
[7] Client: OK 
[0] Worker: HELLO 
[3] Client: OK 
[1] Worker: HELLO 
[1] Client: OK 
[2] Worker: HELLO 
[0] Worker: HELLO 
[5] Client: OK 
[4] Client: OK 
[1] Worker: HELLO 
[2] Client: OK 
[8] Client: OK 
~~~

이전 예제는 여전히 szmq::poll()을 사용합니다. 그렇다면 리엑터는 어떻게 된 것일까요? CZMQ zloop 리엑터는 간단하지만 기능적이며 다음의 것을 수행할 수 있습니다.
* 소켓에 처리 함수(zloop_reader())를 설정합니다. 즉, 소켓에 입력이 있을 때마다 호출되는 코드입니다.
* 소켓에서 처리 함수(zloop_reader())를 취소합니다.
* 특정 간격으로 한번 또는 여러 번 꺼지는 타이머를 설정합니다.
* 타이머를 취소합니다.

zloop는 내부적으로 zmq_poll()을 사용합니다. 처리 함수(zloop_reader())를 추가하거나 제거할 때마다 폴링 세트를 재구축되고, 다음 타이머와 일치하도록 폴링 제한시간을 계산합니다. 그리고 주의가 필요한 각 소켓 및 타이머에 대한 처리 함수(zloop_reader())와 타이머 함수(zloop_timer())를 호출합니다.

리액터 패턴을 사용하면 기존 코드가 변경되며 루프(while(true))가 제거됩니다.. 주요 논리는 다음과 같습니다.

```java
zloop_t *reactor = zloop_new ();
zloop_reader (reactor, self->backend, s_handle_backend, self);
zloop_start (reactor);
zloop_destroy (&reactor);
```

메시지의 실제 처리는 전용 함수 또는 메서드 내에 있으며(s_handle_frontend(), s_handle_backend()) 이런 형태로 마음에 들지 않을 수 있습니다 : 그것은 취향의 문제입니다. 이 패턴은 타이머 처리와 소켓의 처리가 섞여 있는 경우에 도움이 됩니다. 이 책의 예제에서 간단한 경우 zmq_poll()을 사용하고 복잡한 경우 zloop를 사용하겠습니다.

### lbbroker3.java : zloop을 사용한 부하 분산 브로커

s_backend_handle()과 s_frontend_handle()에 전달되는 구조체는 다음과 같습니다.

```java
typedef struct {
    szmq::detail::SocketImpl& frontend;    //  Listen to clients
    szmq::detail::SocketImpl& backend;     //  Listen to workers
    queue<szmq::Message> *workers;  // List of ready workers
} lbbroker_t;
```

### lbbroker3.java "handler"에 전달되는 포인터를 변경

```java
//  Load-balancing broker
//  Demonstrates use of the CZMQ API and reactor style
//
//  The client and worker tasks are identical from the previous example.
//  .skip

#include <szmq/szmq.h>
#include <czmq.h>
#include <iostream>
#include <cstdlib>
#include <thread>
#include <queue>
#include <boost/format.hpp>
using namespace std;

#define NBR_CLIENTS 10
#define NBR_WORKERS 3
#define WORKER_READY   "\001"      //  Signals worker is ready

//  Basic request-reply client using REQ socket
//
static void *
client_task(void *args)
{
    szmq::Context context; 
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> client(context);
    client.connect(szmq::SocketUrl("tcp://localhost:5672"));
    while(true){
        client.sendOne(szmq::Message::from("HELLO"));
        auto reply = client.recvOne().read<std::string>();
        cout << "Client : " << reply << endl;
        this_thread::sleep_for(chrono::milliseconds(1000));
    }
    client.close();
    return NULL;
}

//  Worker using REQ socket to do load-balancing
//
static void *
worker_task(void *args)
{
    szmq::Context context;
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> worker(context);
    worker.connect(szmq::SocketUrl("tcp://localhost:5673"));
    //  Tell broker we're ready for work
    worker.sendOne(szmq::Message::from(WORKER_READY));
    //  Process messages as they arrive(ClientID +  "Hello")
    while (true) {     
        auto msgs = worker.recvMultiple();
		cout << "Worker: " << msgs.back().read<std::string>() << endl; //request("HELLO")
		msgs.pop_back();
		msgs.emplace_back(szmq::Message::from(std::string("OK")));
        worker.sendMultiple(msgs);  // ClietID +  "OK"
    }
    worker.close();
    return NULL;
}

//  .until
//  Our load-balancer structure, passed to reactor handlers
typedef struct {
    szmq::detail::SocketImpl& frontend;             //  Listen to clients
    szmq::detail::SocketImpl& backend;              //  Listen to workers
    queue<szmq::Message> *workers;  // List of ready workers
} lbbroker_t;

//  .split reactor design
//  In the reactor design, each time a message arrives on a socket, the
//  reactor passes it to a handler function. We have two handlers; one
//  for the frontend, one for the backend:

//  Handle input from client, on frontend
int s_handle_frontend (zloop_t *loop, szmq::PollItem *poller, void *arg)
{
    lbbroker_t *self = (lbbroker_t *) arg;
    auto msgs = self->frontend.recvMultiple();
    msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
    msgs.insert(msgs.begin(), self->workers->front());			
    self->workers->pop();
    self->backend.sendMultiple(msgs);  //WorkerID + "" + ClientID + "" + request     
    //  Cancel reader on frontend if we went from 1 to 0 workers
    if (self->workers->size() == 0) {
        szmq::PollItem poller = {reinterpret_cast<void*>(*self->frontend), 0, ZMQ_POLLIN };
        zloop_poller_end (loop, &poller);
    }
    return 0;
}

//  Handle input from worker, on backend
int s_handle_backend (zloop_t *loop, szmq::PollItem *poller, void *arg)
{
    //  Use worker identity for load-balancing
    lbbroker_t *self = (lbbroker_t *) arg;
    auto msgs = self->backend.recvMultiple();
    auto worker_id = msgs.front();    
    self->workers->push(worker_id);
    msgs.erase(msgs.begin()); // delete the worker_id
    msgs.erase(msgs.begin()); // delete the empty delimiter	  
    //  Enable reader on frontend if we went from 0 to 1 workers
    if (self->workers->size() == 1) {
        szmq::PollItem poller = {reinterpret_cast<void*>(*self->frontend), 0, ZMQ_POLLIN};
        zloop_poller (loop, &poller, s_handle_frontend, self);
    }
    auto client_id = msgs.front();
    if ((client_id.read<std::string>().compare(WORKER_READY)) != 0) { 
        self->frontend.sendMultiple(msgs);	// clientID + "" + reply
    }
    return 0;
}

//  .split main task
//  And the main task now sets up child tasks, then starts its reactor.
//  If you press Ctrl-C, the reactor exits and the main task shuts down.
//  Because the reactor is a CZMQ class, this example may not translate
//  into all languages equally well.

int main (void)
{
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> backend(context);
    backend.bind(szmq::SocketUrl("tcp://*:5673"));
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);
    frontend.bind(szmq::SocketUrl("tcp://*:5672"));
    // make lbbroker structure
    lbbroker_t self = {frontend, backend, new queue<szmq::Message>};

    int client_nbr;
    for (client_nbr = 0; client_nbr < NBR_CLIENTS; client_nbr++) {
		thread client(&client_task, (void *)(intptr_t)client_nbr);
        client.detach();
    }
    int worker_nbr;
    for (worker_nbr = 0; worker_nbr < NBR_WORKERS; worker_nbr++) {
        thread worker(&worker_task, (void *)(intptr_t)worker_nbr);
        worker.detach();
    }   

    //  Prepare reactor and fire it up
    zloop_t *reactor = zloop_new ();
    szmq::PollItem poller = {reinterpret_cast<void *>(*backend), 0, ZMQ_POLLIN };
    zloop_poller (reactor, &poller, s_handle_backend, &self);
    zloop_start  (reactor);
    zloop_destroy (&reactor);

    //  When we're done, clean up properly    
    while (self.workers->size()) {
        self.workers->pop();
    }
    frontend.close();
    backend.close();
    delete (self.workers);
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd lbbroker3.java szmq.lib czmq.lib
PS D:\work\sook\src\szmq\examples> ./lbbroker3
Worker: HELLO
Worker: HELLO
Client : OK
Worker: HELLO
Client : OK
Worker: HELLO
Worker: HELLO
Client : OK
Worker: HELLO
Client : OK
Client : OK
Worker: HELLO
Client : OK
Worker: HELLO
Worker: HELLO
Client : OK
Client : OK
Client : OK
Worker: HELLO
Client : OK
...
~~~

## 비동기 클라이언트/서버 패턴

ROUTER에서 DEALER 예제에서 하나의 서버가 여러 작업자들과 비동기적으로 통신하는 1 대 N 사용 사례(mtserver와 hwclient(REQ) 사례)를 보았습니다. 역으로 다양한 클라이언트들이 단일 서버와 비동기식으로 통신하고 매우 유용한 N-to-1 아키텍처를 사용할 수 있습니다.

동작 방식은 다음과 같습니다.

* 클라이언트는 서버에 연결하여 요청을 보냅니다.
* 각 요청에 대해 서버는 0개 이상의 응답을 보냅니다.
* 클라이언트는 응답을 기다리지 않고 여러 요청들을 보낼 수 있습니다.
* 서버는 새로운 요청을 기다리지 않고 여러 응답들을 보낼 수 있습니다.

### asyncsrv.java : 비동기 클라이언트(N)/서버(1)

```java
//  Asynchronous client-to-server (DEALER to ROUTER)
//
//  While this example runs in a single process, that is to make
//  it easier to start and stop the example. Each task has its own
//  context and conceptually acts as a separate process.
#include <szmq/szmq.h>
#include <iostream>
#include <czmq.h>
#include <cstdlib>
#include <thread>
using namespace std;

#define NBR_THREADS 3

//  This is our client task
//  It connects to the server, and then sends a request once per second
//  It collects responses as they arrive, and it prints them out. We will
//  run several client tasks in parallel, each with a different random ID.

static void *
client_task (void *args)
{
    szmq::Context context; 
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> client(context);
#ifdef _WIN32
    client.setId((intptr_t)args);
#else
    client.setId();          //  Set a printable identity.
#endif
    client.connect(szmq::SocketUrl("tcp://localhost:5570"));
    std::vector<szmq::PollItem> pollItems = {
        {reinterpret_cast<void*>(*client), 0, ZMQ_POLLIN, 0}};
    int request_nbr = 0;
    while (true) {
        //  Tick once per second, pulling in arriving messages
        int centitick;
        for (centitick = 0; centitick < 100; centitick++) {
            szmq::poll(pollItems, 1, 10);
            if (pollItems [0].revents & ZMQ_POLLIN) {
                auto msgs = client.recvN();
                cout << "client task id :(" << (intptr_t)args << ") received" << endl;
                for (auto it = begin (msgs); it != end (msgs); ++it) 
                    it->dump();
            }
        }
        char request[20];
        snprintf (request, 20 ,	"request #%d ",++request_nbr);
        client.sendOne(szmq::Message::from(request));
    }
    client.close();
    return NULL;
}

//  .split server task
//  This is our server task.
//  It uses the multithreaded server model to deal requests out to a pool
//  of workers and route replies back to clients. One worker can handle
//  one request at a time but one client can talk to multiple workers at
//  once.

static void * server_worker (void *ctx);

static void *
server_task (void *args)
{
    //  Frontend socket talks to clients over TCP
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);      
    frontend.bind(szmq::SocketUrl("tcp://*:5570"));
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> backend(context);
    backend.bind(szmq::SocketUrl("inproc://backend"));

    std::vector <std::thread> threadPool;
    for (int thread_nbr = 0; thread_nbr < 5; ++thread_nbr) {
        threadPool.push_back(std::thread([&]() {
            server_worker((void *)&context); // will connect with inproc://workers
        }));
    }
    //  Connect backend to frontend via a proxy
    szmq::proxy(reinterpret_cast<void*>(*frontend), reinterpret_cast<void*>(*backend), NULL);
    frontend.close();
    backend.close();
    return NULL;
}

//  .split worker task
//  Each worker task works on one request at a time and sends a random number
//  of replies back, with random delays between replies:

static void *
server_worker (void *ctx)
{
    szmq::Context *context = (szmq::Context *)ctx;
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> worker(*context);
    worker.connect(szmq::SocketUrl("inproc://backend"));

    while (true) {
        //  The DEALER socket gives us the reply envelope and message
        auto msgs = worker.recvN();
        auto identity = msgs.front();
        msgs.erase(msgs.begin());
        auto content = msgs.front();
        msgs.erase(msgs.begin());
        
        //  Send 0..4 replies back
        int reply, replies = std::rand() % 5;
        for (reply = 0; reply < replies; reply++) {
            //  Sleep for some fraction of a second
            zclock_sleep(std::rand() % 1000 + 1);
            worker.sendMultiple(identity, content);
        }
    }
}

//  The main thread simply starts several clients and a server, and then
//  waits for the server to finish.

int main (void)
{
    std::vector <std::thread> threadPool;
    for (int thread_nbr = 0; thread_nbr < NBR_THREADS; ++thread_nbr) {
        threadPool.push_back(std::thread([&]() {
            client_task(((void *)(intptr_t)thread_nbr)); 
        }));
    }
    thread(&server_task, nullptr).detach();
    zclock_sleep(5000); //  Run for 5 seconds then quit
    return 0;
}
```

### 빌드 및 실행

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc asyncsrv.java szmq.lib czmq.lib

PS D:\work\sook\src\szmq\examples> ./asyncsrv
client task id :(0) received
[010]request #1
client task id :(1) received
[010]request #1
client task id :(2) received
[010]request #1
client task id :(1) received
[010]request #2
client task id :(0) received
[010]request #2
client task id :(2) received
[010]request #2
client task id :(2) received
[010]request #2
client task id :(2) received
[010]request #2
~~~

이 예제는 다중 프로세스 아키텍처를 시뮬레이션하여 멀티스레드를 사용하여 하나의 프로세스에서 실행됩니다. 예제를 실행하면 서버에서 받은 응답을 출력하는 3개의 클라이언트 (각각 임의의 식별자(ID)가 있음)가 표시됩니다. 주의 깊게 살펴보면 각 클라이언트 작업이 요청당 0개 이상의 응답을 받는 것을 볼 수 있습니다.

이 코드에 대한 설명입니다.

* 클라이언트는 초당 한 번씩 요청을 보내고 0개 이상의 응답을받습니다. zmq_poll()을 사용하여 작업을 수행하기 위해, 단순히 1초 제한시간으로 폴링할 수 없으며 마지막 응답을 받은 후 1초 후에 새 요청을 보내게 됩니다. 그래서 우리는 높은 빈도(1초당 100회 폴링(1/100초(10밀리초) 간격))로 폴링합니다. 이는 거의 정확합니다.
* 서버는 작업자 스레드 풀(pool)을 사용하여 각 스레드가 하나의 요청을 동기적으로 처리합니다. 클라이언트(DEALER)는 내부 대기열을 사용하여 프론트엔드 소켓(ROUTER)에 연결하고 작업자(DEALER)도 내부 대기열을 사용하여 백엔드 소켓(DEALER)에 연결합니다. zmq_proxy() 호출하여 프런트 엔드와 백엔드 소켓 간에 통신하도록 합니다.

클라이언트와 서버간에 DEALER와 ROUTER 통신을 수행하고 있지만, 서버 메인 스레드와 작업자 스레드들 간에는 내부적으로 DEALER와 DEALER를 수행하고 있습니다. REP 소켓을 사용했듯이 작업자들은 엄격하게 동기식입니다. 하지만 여러 회신을 보내려고 하기 때문에 비동기 소켓이 필요합니다. 회신들을 각 응답에 대하여 직접 라우팅하지 않기 위해서, 항상 요청을 보낸 단일 서버 스레드로 가게 합니다.

라우팅 봉투에 대해 생각해 봅시다. 클라이언트는 단일 프레임으로 구성된 메시지를 보냅니다. 서버 스레드는 2개 프레임 메시지(CLINT ID + DATA)를 받습니다. 2개 프레임을 작업자에게 보내면 일반 응답 봉투로 취급하고 2개 프레임 메시지(CLINT ID + DATA)로 반환합니다. 그러면 작업자는 첫 번째 프레임을 라우팅할 ID(CLIENT ID)로 두 번째 프레임을 클라이언트에 대한 응답으로 사용합니다.

이제 소켓들에 대해 : 부하 분산 ROUTER와 DEALER 패턴을 작업자들 간의 통신에 사용할 수 있었지만 추가 작업이 있습니다. 이 경우 DEALER와 DEALER 패턴은 괜찮겠지만 단점은 각 요청에 대한 지연시간가 짧지만 작업 분산이 평준화되지 않을 위험이 있습니다. 이 경우 단순화시켜 대응합니다.

클라이언트와 통신 상태를 유지하는 서버를 구축할 때 고전적인 문제가 발생합니다. 서버가 클라이언트에 대한 통신 상태를 유지하지만, 클라이언트는 ‘희미(동적)한 존재(comes and goes)’지만 연결을 유지할 경우 결국 서버 자원은 부족하게 됩니다. 기본 식별자(ID)를 사용하여 동일한 클라이언트들이 계속 연결을 유지하더라도 각 연결은 새로운 연결처럼 보입니다.

위의 예제를 매우 짧은 시간(작업자가 요청을 처리하는 데 걸리는 시간) 동안만 통신 상태를 유지 한 다음 통신 상태를 버리는 방식으로 문제를 해결할 수 있습니다. 그러나 해결안은 많은 경우에 실용적이지 않습니다. 상태 기반 비동기 서버에서 클라이언트 상태를 적절하게 관리하려면 다음의 작업을 수행해야 합니다.

* 일정 시간 간격으로 클라이언트에서 서버로 심박을 보냅니다. 위의 예제에서는 클라이언트에서 1초당 한번 요청을 보냈으며, 심박으로 신뢰할 수 있게 사용할 수 있습니다.
* 클라이언트 식별자(ID)를 키로 사용하여 상태를 저장합니다.
* 클라이언트로부터 중단된 심박을 감지합니다. 클라이언트로부터 일정 시간(예 : 2초) 동안 요청이 없으면 서버는 이를 감지하고 해당 클라이언트에 대해 보유하고 있는 모든 상태를 폐기할 수 있습니다.

### asyncsrv1.java : 타이머를 사용히지 않고 N(작업자)-N(클라이언트)간의 통신

```java
//  Asynchronous client-to-server (DEALER to ROUTER)
//
//  While this example runs in a single process, that is to make
//  it easier to start and stop the example. Each task has its own
//  context and conceptually acts as a separate process.
#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
#include <cstdlib>
#include <thread>
using namespace std;

#define NBR_WORKERS 5
#define NBR_CLIENTS 3

//  This is our client task
//  It connects to the server, and then sends a request once per second
//  It collects responses as they arrive, and it prints them out. We will
//  run several client tasks in parallel, each with a different random ID.

static void *
client_task (void *args)
{
    szmq::Context context; 
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> client(context);
#ifdef _WIN32
    client.setId((intptr_t)args);
#else
    client.setId();          //  Set a printable identity.
#endif
    client.connect(szmq::SocketUrl("tcp://localhost:5570"));

    std::vector<szmq::PollItem> pollItems = {
        {reinterpret_cast<void*>(*client), 0, ZMQ_POLLIN, 0}};
    int request_nbr = 0;
    while (true) {
        //  Tick once per second, pulling in arriving messages
        int centitick;
        for (centitick = 0; centitick < 100; centitick++) {
            szmq::poll(pollItems, 1, 10);
            if (pollItems [0].revents & ZMQ_POLLIN) {
                auto msgs = client.recvMultiple();
                cout << boost::str(boost::format("[%1%] client task received : ") % (intptr_t)args);
                for (auto it = begin (msgs); it != end (msgs); ++it) 
                    it->dump();
            }
        }
        client.sendOne(szmq::Message::from(boost::str(boost::format("request #%1%") % ++request_nbr)));
    }
    client.close();
    return NULL;
}

//  .split worker task
//  Each worker task works on one request at a time and sends a random number
//  of replies back, with random delays between replies:

static void *
server_worker (void *ctx)
{
    szmq::Context *context = (szmq::Context *)ctx;
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> worker(*context);
    worker.connect(szmq::SocketUrl("inproc://backend"));

    while (true) {
        //  The DEALER socket gives us the reply envelope and message
        auto msgs = worker.recvMultiple();
        auto identity = msgs.front();
        msgs.erase(msgs.begin());
        auto content = msgs.front();
        msgs.erase(msgs.begin());
        
        //  Send 0..4 replies back
        int reply, replies = std::rand() % 5;
        for (reply = 0; reply < replies; reply++) {
            //  Sleep for some fraction of a second
            this_thread::sleep_for(chrono::milliseconds(std::rand() % 1000 + 1));
            worker.sendMultiple(identity, content);
        }
    }
}

//  The main thread simply starts several clients and a server, and then
//  waits for the server to finish.

int main (void)
{
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);      
    frontend.bind(szmq::SocketUrl("tcp://*:5570"));
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> backend(context);
    backend.bind(szmq::SocketUrl("inproc://backend"));
    
    //  Launch pool of worker threads, precise number is not critical
    int thread_nbr;
    for (thread_nbr = 0; thread_nbr < NBR_WORKERS; thread_nbr++) {
        thread worker(&server_worker, (void *)&context);
        worker.detach();
    }

    for (thread_nbr = 0; thread_nbr < NBR_CLIENTS; thread_nbr++) {     
        thread client(&client_task, (void *)(intptr_t)thread_nbr);
        client.detach();
    } 
    //  Connect backend to frontend via a proxy
    szmq::proxy(reinterpret_cast<void*>(*frontend), reinterpret_cast<void*>(*backend), NULL);
    frontend.close();
    backend.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
// 원도우
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd asyncsrv1.java szmq.lib
PS D:\work\sook\src\szmq\examples> ./asyncsrv1
[0] client task received : [010]request #1
[2] client task received : [010]request #1
[1] client task received : [010]request #1
[2] client task received : [010]request #2
[0] client task received : [010]request #2
[1] client task received : [010]request #2
[1] client task received : [010]request #2
[1] client task received : [010]request #2
[1] client task received : [010]request #2
...

// 리눅스
zedo@sook:/work/sook/src/szmq/examples$ g++ -o asyncsrv1 asyncsrv1.java -lsook-szmq -lzmq -lpthread 
zedo@sook:/work/sook/src/szmq/examples$ ./asyncsrv1
[2] client task received : [010]request #1
[2] client task received : [010]request #1
[0] client task received : [010]request #1
[0] client task received : [010]request #1
[2] client task received : [010]request #2
[0] client task received : [010]request #2
...
~~~

## 동작 예제 : 브로커 간 라우팅

지금까지 본 모든 것을 가져와 실제 응용프로그램으로 확장해 보겠습니다. 여러 번의 반복을 거처 단계별로 구축하겠습니다. 우량(VIP) 고객이 긴급하게 전화를 걸어 대규모 클라우드 컴퓨팅 시설의 설계를 요청합니다. 고객의 클라우드에 대한 비전은 많은 데이터 센터에 퍼져 있는 각 클라이언트들과 작업자들이 클러스터를 통해 하나로 동작하는 있습니다. 우리는 실전이 항상 이론을 능가한다는 것을 알만큼 똑똑하기 때문에 ØMQ를 사용하여 동작하는 시뮬레이션을 만들겠습니다. 우리의 고객은 자신의 상사가 마음을 바꾸기 전에 예산을 확정하기 바라며, 트위터에서 ØMQ에 대한 훌륭한 정보를 읽은 것 같습니다.

### 상세한 요구 사항

몇 잔의 에스프레소를 마시고 코드 작성에 뛰어들고 싶지만, 전체적으로 잘못된 문제에 대한 놀라운 해결책을 제공하기 전에, 자세한 사항을 확인하라고 마음속에서 무언가의 속삭임 있습니다. 그래서 고객에서 “클라우드로 어떤 일을 하고 싶으시나요?”라고 묻습니다.

* 작업자는 다양한 종류의 하드웨어에서 실행되지만, 어떤 작업도 처리할 수 있어야 합니다. 클러스터당 수백 개의 작업자가 있고 대략 12개의 클러스터가 있습니다.
* 클라이언트는 작업자에게 작업을 요청합니다. 각 작업은 독립적인 작업 단위로 클라이언트는 가용한 작업자를 찾아 가능한 한 빨리 작업을 보내려고 합니다. 많은 클라이언트가 존재하며 임의적으로 왔다 갔다 합니다.
* 클라우드에서 진짜 어려운 것은 클러스터를 언제든지 추가하고 제거할 수 있어야 하는 것입니다. 클러스터는 속해 있는 모든 작업자와 클라이언트들을 함께 즉시 클라우드를 떠나거나 합류할 수 있습니다.
* 자체 클러스터에 작업자가 없는 경우, 클라이언트의 작업은 클라우드에서 가용한 다른 클러스터의 작업자에게 전달됩니다.
* 클라이언트는 한 번에 하나의 작업을 보내 응답을 기다립니다. 제한시간(X초) 내에 응답을 받지 못하면 작업을 다시 전송합니다. 이것은 우리의 고려사항은 아니며 클라이언트 API가 이미 수행하고 있습니다.
* 작업자들은 한 번에 하나의 작업을 처리합니다. 그들은 매우 단순하게 작업을 처리합니다. 작업자들의 수행이 중단되면 그들을 기동한 스크립트에 의해 재시작됩니다.

위에서 설명한 것을 제대로 이해했는지 다시 확인합니다.

* “클러스터들 간에 일종의 초고속 네트워크 상호 연결이 있을 것입니다. 맞습니까?” 고객은 “예, 물론 우리는 바보가 아닙니다.”라고 말합니다.
* “통신 규모는 어느 정도입니까?”라고 묻습니다. 고객은 “클러스터 당 최대 1,000개의 클라이언트들, 각 클라이언트는 초당 최대 10 개의 요청을 수행합니다. 요청은 작고 응답도 각각 1KB 이하로 작습니다.”라고 응답합니다(20 Mbytes=20,000,000 bytes=1(Cluster) * 1,000(clients) * 10(requests) * 1,000(bytes) * 2(send/recv)).

그러면 고객의 요구 사항에 대하여 약간의 계산을 하고 이것이 일반 TCP상에서 작동할지 확인합니다. 클라이언트들 2,500 개 x 10/초(request) x 1,000바이트(date) x 2방향(send/recv) = 50 MBytes/초 또는 400 Mbit/초, 1Gb 대역폭의 TCP 네트워크에는 문제가 되지 않습니다.

이것은 간단한 문제로 특별한 하드웨어나 통신규약들이 필요하지 않고 다소 영리한 라우팅 알고리즘과 신중한 설계만 필요로 합니다. 먼저 하나의 클러스터(하나의 데이터 센터)를 설계하고 클러스터를 함께 연결하는 방법을 알아보겠습니다.

### 단일 클러스터 아키텍처

작업자들과 클라이언트들은 동기적으로 동작합니다. 부하 분산 패턴을 사용하여 작업들을 작업자들에게 전달하기 원하며 작업자들은 모두 동일한 기능을 수행합니다. 데이터센터에는 작업자는 익명이며 특정 서비스에 대한 개념이 없습니다. 클라이언트들은 직접 주소를 지정하지 않습니다. 재시도(제한시간(X초) 내에 응답을 받지 못하면 클라이언트에서 작업을 다시 전송)가 자동으로 이루어지므로 통신에 대한 보증은 언급하지 않아도 좋을 것입니다.

우리가 이미 보았듯이 클라이언트들과 작업자들은 서로 직접 통신하지 않습니다. 동적으로 노드들을 추가하거나 제거하는 것이 불가능합니다. 따라서 우리의 기본 모델은 이전에 살펴본 요청-응답 메시지 브로커로 구성됩니다.

### 다중 클러스터로 확장

이제 하나 이상의 클러스터로 확장합니다. 각 클러스터에는 일련의 클라이언트들 및 작업자들이 있으며 이들을 함께 결합하는 브로커(broker)가 있습니다.

여기서 질문입니다 : 각 클러스터에 속해 있는 클라이언트들이 다른 클러스터의 작업자들과 통신하는 방법은 어떻게 될까요? 여기 몇 가지 방법이 있으며 각각 장단점이 있습니다.

*  클라이언트들은 직접 양쪽 브로커에 연결할 수 있습니다. 장점은 브로커들과 작업자들을 수정할 필요가 없습니다. 그러나 클라이언트들은 더 복잡해지고 전체 토폴로지를 알아야 합니다. 예를 들어 세 번째 또는 네 번째 클러스터를 추가하려는 경우 모든 클라이언트들이 영향을 받습니다. 영향으로 클라이언트의 라우팅 및 장애조치 로직을 변경해야 하며 좋지 않습니다.
* 작업자들은 직접 양쪽 브로커에 연결하려 하려 하지만 REQ 작업자는 하나의 브로커에만 응답할 수 있어 그렇게 할 수 없습니다. REP를 사용하려 하지만 REP는 부하 분산처럼 사용자 지정 가능한 브로커와 작업자 간 라우팅을 제공하지 않고 내장된 부하 분산만 제공하여 잘못된 것입니다. 유휴 작업자에게 작업을 분배하려면 정확하게 부하 분산이 필요합니다. 유일한 해결책은 작업자 노드들에 대해 ROUTER 소켓을 사용하는 것입니다. 이것을 “아이디어# 1”로 명명하겠습니다.
* 브로커들은 상호 간에 연결할 수 있습니다. 가장 적은 추가 연결을 생성하여 가장 깔끔해 보입니다. 동적으로 클러스터를 추가하기 어려지만 설계 범위를 벗어난 것 같습니다. 이제 클라이언트들과 작업자들은 실제 네트워크 토폴로지를 모르게 하고 브로커들 간에는 여유 용량이 있을 때 서로에게 알립니다. 이것을 “아이디어 #2”로 명명하겠습니다.

아이디어 #1을 분석해 보겠습니다. 이 모델에서는 작업자들이 양쪽 브로커들에 연결하고 하나의 브로커에서 작업을 수락합니다.

좋은 방법으로 보이지만 우리가 원하는 것을 제공하지 않습니다. 클라이언트들은 가능하면 로컬 작업자들을 얻고 기다리는 것보다 더 나은 경우에만 원격 작업자를 얻습니다. 또한 작업자들은 양쪽 브로커들에게 “준비(READY)”신호를 보내고 다른 작업자는 유휴 상태로 있는 동안 한 번에 두 개의 작업들을 받을 수 있습니다. 이 디자인은 잘못된 것 같으며 원인은 다시 우리가 양쪽 말단에 라우팅 로직 넣어야 하기 때문입니다.

그럼, 아이디어# 2에서 우리는 브로커들을 상호 연결하고 우리가 익숙한 REQ 소켓을 사용하는 클라이언트들과 또는 작업자들을 손대지는 않습니다.

이 디자인은 문제가 한 곳에서 해결되고 나머지 것들은 보이지 않기 때문에 매력적입니다. 기본적으로 브로커들은 서로에게 비밀 채널들을 열고 낙타 상인처럼 속삭입니다. “이봐, 나는 여유가 좀 있는데 클라이언트들이 너무 많을 경우 알려 주시면 우리가 대응하겠습니다.”

사실 이것은 더 복잡한 라우팅 알고리즘 일뿐입니다 : 브로커들은 서로를 위해 하청업체가 됩니다. 실제 코드를 작성하기 전에 이와 같은 설계를 좋아할 점들이 있습니다.

* 일반적인 경우(동일한 클러스터의 클라이언트들과 작업자들)를 기본으로 하고 예외적인 경우(클러스터들 간 작업을 섞음)에 대한 추가 작업을 수행합니다.
* 다른 유형의 작업에 대해 다른 메시지 흐름을 사용하게 합니다. 다르게 처리하게 하기 위함으로 예를 들면 서로 다른 유형의 네트워크 연결을 사용합니다.
* 부드럽게 확장할 수 있습니다. 3개 이상의 브로커들간의 상호 연결하는 것은 다소 복잡하게 되지만, 이것이 문제라고 판단되면 하나의 슈퍼 브로커를 추가하여 쉽게 해결할 수 있습니다.

이제 동작하는 예제를 만들겠습니다. 전체 클러스터를 하나의 프로세스로 압축합니다. 분명히 현실적이지는 않지만 시뮬레이션하기에는 단순하게 만들어 시뮬레이션을 정확하게 실제 프로세스들로 확장 할 수 있습니다. 이것이 ØMQ의 아름다움입니다 - 미시 수준에서 설계하고 거시 수준까지 확장 할 수 있습니다. 스레드들은 프로세스들이 된 다음 하드웨어 머신들로 점차 거시적으로 확장 가능 하지만, 패턴들과 논리는 동일하게 유지됩니다. 각 “클러스터” 프로세스들에는 클라이언트 스레드들, 작업자 스레드들 및 브로커 스레드가 포함됩니다.

이제 기본 모델을 잘 알게 되었습니다.

* REQ 클라이언트 스레드들은 작업부하들을 생성하여 브로커(ROUTER)로 전달합니다.
* REQ 작업자 스레드들은 작업부하들을 처리하고 결과들을 브로커(ROUTER)로 반환합니다.
* 브로커는 부하 분산 패턴을 사용하여 작업부하들을 대기열에 넣고 분배합니다.

### 페더레이션 및 상대 연결

브로커들을 상호 연결하는 방법에는 여러 가지가 있습니다. 우리가 원하는 것은 다른 브로커들에게 “우리는 여유 용량이 있어”라고 말한 다음 여러 작업들을 받는 것입니다. 또한 우리는 다른 브로커들에게 “그만, 우리는 여유 용량이 없어”라고 말할 수 있어야 합니다. 완벽할 필요는 없으며, 때로는 즉시 처리할 수 없는 작업들을 받은 다음 가능한 한 빨리 처리합니다.

가장 쉬운 상호 연결은 연합(Federation)이며, 브로커들이 클라이언트들과 작업자들을 서로 시뮬레이션하는 것입니다. 이를 수행하기 위해 클러스터의 백엔드를 다른 브로커의 프론트엔드 소켓에 연결합니다. 한 소켓을 단말에 바인딩하고 다른 단말에 연결이 모두 가능한지 확인하십시오.

연합은 브로커들과 타당하고 좋은 처리 방식을 가진 단순한 로직을 제공합니다. 클라이언트들이 없을 때 다른 브로커에게 “준비(READY)”라고 알리고 하나의 작업을 받아들입니다. 유일한 문제는 이것이 너무 단순하다는 것입니다. 연합된 브로커는 한 번에 하나의 작업만 처리할 수 있습니다. 브로커가 잠금 단계 클라이언트와 작업자로 하게 되면 정의상 잠금 단계가 되며, 비록 많은 작업자들이 있어도 동시에 사용할 수 없습니다. 우리의 브로커들은 완전히 비동기적으로 연결되어야 합니다.

페더레이션 모델은 다른 종류의 라우팅, 특히 부하 분산이나 라운드 로빈보다는 서비스 이름 및 근접성에 따라 라우팅하는 서비스 지향 아키텍처들(SOAs)에 적합합니다. 모든 용도에 적응하는 가능한 것은 아니지만 용도에 따라 사용할 수 있습니다.

### peering1_tcp.java : state flow에 대한 기본작업(tcp)

tcp를 사용하여 스레드간 통신이 가능하도록 변경하여 테스트를 수행합니다. tcp 소켓을 사용하도록 변경합니다.

```java
//  Broker peering simulation (part 1)
//  Prototypes the state flow
#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>
#include <thread>
using namespace std;

int main (int argc, char *argv [])
{
    //  First argument is this broker's name
    //  Other arguments are our peers' names
    //
    if (argc < 2) {
        cout << "syntax: peering1 me {you}...\n";
        return 0;
    }    
    srand (static_cast<unsigned int>(std::time(0)));
    char *self = argv[1];
    cout << "I: preparing broker at "<< self <<"...\n";

    //  Bind state backend to endpoint
    szmq::Context context;
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> statebe(context);
    char fmt[255];
    snprintf(fmt, 255, "tcp://*:%s", self);
    cout << fmt << endl;
    statebe.bind(szmq::SocketUrl(fmt));
    
    //  Connect statefe to all peers
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> statefe(context);
    statefe.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    for (int argn = 2; argn < argc; argn++) {
        char *peer = argv[argn];
        cout <<"I: connecting to state backend at "<< peer << endl;
        snprintf(fmt, 255, "tcp://localhost:%s", peer);
        cout << fmt << endl;
        statefe.connect(szmq::SocketUrl(fmt));
    }
    //  .split main loop
    //  The main loop sends out status messages to peers, and collects
    //  status messages back from peers. The zmq_poll timeout defines
    //  our own heartbeat:
    while (true) {
        //  Poll for activity, or 1 second timeout
        std::vector<szmq::PollItem> items = {
			{reinterpret_cast<void*>(*statefe), 0, ZMQ_POLLIN, 0}};
        szmq::poll(items, 1, 1000);    // timeout 1sec
        //  Handle incoming status messages
        if (items [0].revents & ZMQ_POLLIN) {
            auto peer_name = statefe.recv1().read<std::string>();
            auto available = statefe.recv1().read<int>();
            snprintf(fmt, 255, "%s - %d workers received", peer_name.c_str(), available);
            cout << fmt << endl;
        }
        else {
            //  Send random values for worker availability
            statebe.sendMultiple(szmq::Message::from(self), szmq::Message::from(rand()%10));
        }
    }
    return EXIT_SUCCESS;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd peering1_tcp.java szmq.lib
PS D:\work\sook\src\szmq\examples> ./peering1_tcp 5555 5556
I: preparing broker at 5555...
I: connecting to state backend at 5556
5556 - 5 workers received
5556 - 6 workers received
5556 - 9 workers received
...

PS D:\work\sook\src\szmq\examples> ./peering1_tcp 5556 5555
I: preparing broker at 5556...
I: connecting to state backend at 5555
5555 - 0 workers received
5555 - 8 workers received
5555 - 8 workers received
~~~

### peering1_inproc.java : state flow에 대한 기본작업(inproc)
inproc를 사용하여 스레드간 통신이 가능하도록 변경하여 테스트를 수행합니다. inproc 소켓을 사용하도록 변경합니다.

```java
//  Broker peering simulation (part 1)
//  Prototypes the state flow
#include <szmq/szmq.h>
#include <czmq.h>
#include <iostream>
#include <thread>
#include <cstdlib>
using namespace std;

static void *
broker(szmq::Context *ctx, char *argv [])
{
    //  First argument is this broker's name
    //  Other arguments are our peers' names
    //
    char fmt[255];
    char *self = argv[1];
    cout << "I: preparing broker at "<< self <<"...\n";

    //  Bind state backend to endpoint
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> statebe(*ctx);
    snprintf(fmt, 255, "inproc://%s-state", self);
    statebe.bind(szmq::SocketUrl(fmt));
    
    //  Connect statefe to all peers
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> statefe(*ctx);
    statefe.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    int argc = 0;
    while(argv[++argc]);
    for (int argn = 2; argn < argc; argn++) {
        char *peer = argv[argn];
        cout <<"I: connecting to state backend at "<< peer << endl;
        snprintf(fmt, 255, "inproc://%s-state", peer);
        statefe.connect(szmq::SocketUrl(fmt));
    }
    //  .split main loop
    //  The main loop sends out status messages to peers, and collects
    //  status messages back from peers. The zmq_poll timeout defines
    //  our own heartbeat:

    while (true) {
        //  Poll for activity, or 1 second timeout
        std::vector<szmq::PollItem> pollItems = {
			{reinterpret_cast<void*>(*statefe), 0, ZMQ_POLLIN, 0}};
        szmq::poll(pollItems, 1, 1000);    // timeout 1sec
        //  Handle incoming status messages
        if (pollItems [0].revents & ZMQ_POLLIN) {
            auto peer_name = statefe.recvOne().read<std::string>();
            auto available = statefe.recvOne().read<int>();
            snprintf(fmt, 255, "%s - %d workers received", peer_name.c_str(), available);
            cout << fmt << endl;
        }
        else {
            //  Send random values for worker availability
            statebe.sendMultiple(szmq::Message::from(self), szmq::Message::from(rand()%10));
        }
    }
}

int main (int argc, char *argv [])
{
    //  First argument is this broker's name
    //  Other arguments are our peers' names
    //
    if (argc < 2) {
        cout << "syntax: peering1 me {you}...\n";
        return 0;
    }    
    srand (static_cast<unsigned int>(std::time(0)));
    szmq::Context context;
    thread(&broker, &context, argv).detach();
    zclock_sleep(100);

    // Change the order
    int m,n;
    for(n=2; n < argc; n++){
        char *temp = strdup(argv[1]);
        for(m = 2; m < argc; m++)     
        {            
            argv[m-1] = argv[m];
        }
        argv[m-1] = temp;
        thread(&broker, &context, argv).detach();
        zclock_sleep(100);
    }
    while(true){}
    return EXIT_SUCCESS;
}
```

### 빌드 및 테스트

~~~{.bash}
//원도우의 경우
PS D:\work\sook\src\szmq\examples> ./peering_inproc dog cat fish bird
I: preparing broker at dog...
I: connecting to state backend at cat
I: connecting to state backend at fish
I: connecting to state backend at bird
I: preparing broker at cat...
I: connecting to state backend at fish
I: connecting to state backend at bird
I: connecting to state backend at dog
I: preparing broker at fish...
I: connecting to state backend at bird
I: connecting to state backend at dog
I: connecting to state backend at cat
I: preparing broker at bird...
I: connecting to state backend at dog
I: connecting to state backend at cat
I: connecting to state backend at fish
dog - 1 workers received
dog - 1 workers received
dog - 1 workers received
dog - 7 workers received
dog - 7 workers received
fish - 1 workers received
...

//리눅스의 경우
[zedo@jeroMQ examples]$ g++ -o peering_inproc peering_inproc.java  -lsook-szmq -lzmq  -lpthread
[zedo@jeroMQ examples]$ ./peering_inproc dag cat bird fish
I: preparing broker at dag...
I: connecting to state backend at cat
I: connecting to state backend at bird
I: connecting to state backend at fish
I: preparing broker at cat...
I: connecting to state backend at bird
I: connecting to state backend at fish
I: connecting to state backend at dag
I: preparing broker at bird...
I: connecting to state backend at fish
I: connecting to state backend at dag
I: connecting to state backend at cat
I: preparing broker at fish...
I: connecting to state backend at dag
I: connecting to state backend at cat
I: connecting to state backend at bird
dag - 2 workers received
dag - 2 workers received
dag - 2 workers received
...
~~~

### 로컬 및 클라우드 흐름에 대한 기본 작업

2개의 대기열이 필요합니다. 하나는 로컬 클라이언트들의 요청을 위한 것이고 다른 하나는 클라우드 클라이언트들의 요청을 위한 것입니다. 한 가지 옵션은 로컬 및 클라우드 프론트엔드에서 메시지를 가져와서 각각의 대기열들에서 퍼내는 것이지만 ØMQ 소켓은 이미 대기열이 존재하여 무의미합니다. 따라서 ØMQ 소켓 버퍼들을 대기열들로 사용합니다.

특정 작업을 수행하기 위해 백엔드를 폴링하면 수신한 메시지는 작업자의 “준비(READY)” 상태이거나 응답일 수 있습니다. 응답인 경우 로컬 클라이언트 프론트엔드(localfe) 혹은 클라우드 프론트엔드(cloudfe)를 통해 반환합니다.
작업자가 응답하면 사용할 수 있게 되었으므로 대기열에 넣고 큐의 크기를 하나 증가 시킵니다.
작업자들이 가용할 동안 프론트엔드에서 요청을 받아 로컬 작업자(localbe)에게 또는 무작위로 클라우드 상대(cloudbe)로 라우팅합니다.

### peering2_tcp.java: local과 cloud 간 흐름의 기본 구현

대응하는 TCP 포트를 지정하며 브로커의 peer 지정시에 주의가 필요합니다.
localfe(bind) : self
localbe(bind) : std::stoi(self) + 1
couldfe(bind) : std::stoi(self) + 2
couldbe(connect) : std::stoi(peer) + 2

```java
//  Broker peering simulation (part 2)
//  Prototypes the request-reply flow
#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
#include <cstdlib>
#include <thread>
#include <queue>
#define NBR_CLIENTS 10
#define NBR_WORKERS 3
#define WORKER_READY   "\001"      //  Signals worker is ready

using namespace std;

//  Our own name; in practice this would be configured per node
static char *self;

//  .split client task
//  The client task does a request-reply dialog using a standard
//  synchronous REQ socket:

static void *
client_task (void *args)
{
    szmq::Context context;
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> client(context);
    client.connect(szmq::SocketUrl(boost::str(boost::format("tcp://localhost:%1%") % self)));
    cout << boost::format("client : tcp://localhost:%1%\n") % self;
    while (true) {
        //  Send request, get reply
        client.sendOne(szmq::Message::from("HELLO"));
        auto reply = client.recvOne();
        cout << "Client: " << reply.read<std::string>() << endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));  
    }
    client.close();
    return NULL;
}

//  .split worker task
//  The worker task plugs into the load-balancer using a REQ
//  socket:

static void *
worker_task (void *args)
{
    szmq::Context context;
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> worker(context);
    worker.connect(szmq::SocketUrl(boost::str(boost::format("tcp://localhost:%1%") % (stoi(self)+1))));
    cout << boost::format("worker : tcp://localhost:%1%\n") % (stoi(self)+1);
    //  Tell broker we're ready for work
    worker.sendOne(szmq::Message::from(WORKER_READY));

    //  Process messages as they arrive
    while (true) {
        auto msgs = worker.recvMultiple();
        cout << "Worker: " << msgs.back().read<std::string>() << endl; //request("HELLO")
        msgs.pop_back();
		msgs.emplace_back(szmq::Message::from(std::string("OK")));
        worker.sendMultiple(msgs);
    }
    worker.close();
    return NULL;
}

//  .split main task
//  The main task begins by setting-up its frontend and backend sockets
//  and then starting its client and worker tasks:

int main (int argc, char *argv [])
{
    //  First argument is this broker's name
    //  Other arguments are our peers' names
    //
    if (argc < 2) {
        cout << "syntax: peering2 me {you}...\n";
        return 0;
    }
    self = argv [1];
    cout <<"I: preparing broker at " << self << "...\n";
    srand (static_cast<unsigned int>(std::time(0)));

    szmq::Context context;
    //  Bind cloud frontend to endpoint
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> cloudfe(context); 
    cloudfe.setId(string(self));
    cloudfe.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % (stoi(self)+2))));

    //  Connect cloud backend to all peers
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_CLIENT> cloudbe(context); 
    cloudbe.setId(string(self));
    int argn;
    for (argn = 2; argn < argc; argn++) {
        char *peer = argv [argn];
        cout <<"I: connecting to cloud frontend at " << stoi(peer)+2 << endl;
        cloudbe.connect(szmq::SocketUrl(boost::str(boost::format("tcp://localhost:%1%") % (stoi(peer)+2))));
    }
    //  Prepare local frontend and backend
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> localfe(context);    
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> localbe(context);
    localfe.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % self)));
    localbe.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % (stoi(self)+1))));

    //  Get user to tell us when we can start...
    cout << "Press Enter when all brokers are started: ";
    getchar ();

    //  Start local workers
  int worker_nbr;
    for (worker_nbr = 0; worker_nbr < NBR_WORKERS; worker_nbr++){
        thread worker(&worker_task, nullptr);
        worker.detach();
    }
    //  Start local clients
    int client_nbr;
    for (client_nbr = 0; client_nbr < NBR_CLIENTS; client_nbr++){
        thread client(&client_task, nullptr);
        client.detach();
    }

    // Interesting part
    //  .split request-reply handling
    //  Here, we handle the request-reply flow. We're using load-balancing
    //  to poll workers at all times, and clients only when there are one 
    //  or more workers available.

    //  Least recently used queue of available workers
    std::queue<szmq::Message> worker_queue;

    while (true) {
        //  First, route any waiting replies from workers
        std::vector<szmq::PollItem> backends = {
            {reinterpret_cast<void*>(*localbe), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*cloudbe), 0, ZMQ_POLLIN, 0}};        

        //  If we have no workers, wait indefinitely
        szmq::poll(backends, 2, worker_queue.size() ? 1000 : -1);

        //  Handle reply from local worker
        std::vector<szmq::Message> msgs;
        if (backends [0].revents & ZMQ_POLLIN) {
            msgs = localbe.recvMultiple();
            auto identity = msgs.front();
            worker_queue.push(identity);
            msgs.erase(msgs.begin()); // delete the worker_id
            msgs.erase(msgs.begin()); // delete the empty delimiter	
            //  If it's READY, don't route the message any further
            auto client_id = msgs.front();
            if((client_id.read<std::string>().compare(WORKER_READY)) == 0)
                msgs.clear();
        }
        //  Or handle reply from peer broker
        else
        if (backends [1].revents & ZMQ_POLLIN) {
            msgs = cloudbe.recvMultiple();
            //  We don't use peer broker identity for anything
            auto identity = msgs.front();
            msgs.erase(msgs.begin()); // delete the cloudbe id
            msgs.erase(msgs.begin()); // delete the empty delimiter	
        }
        //  Route reply to cloud if it's addressed to a broker
        for (argn = 2; msgs.size() && argn < argc; argn++) {
            auto client_id = msgs.front();
            if((client_id.read<std::string>().compare(argv [argn])) == 0){
                cloudfe.sendMultiple(msgs); msgs.clear();
            }
        }
        //  Route reply to client if we still need to
        if (msgs.size()){
            localfe.sendMultiple(msgs); msgs.clear();
        }
        //  .split route client requests
        //  Now we route as many client requests as we have worker capacity
        //  for. We may reroute requests from our local frontend, but not from 
        //  the cloud frontend. We reroute randomly now, just to test things
        //  out. In the next version, we'll do this properly by calculating
        //  cloud capacity:

        while (worker_queue.size()) {
            std::vector<szmq::PollItem> frontends = {
                {reinterpret_cast<void*>(*localfe), 0, ZMQ_POLLIN, 0},
                {reinterpret_cast<void*>(*cloudfe), 0, ZMQ_POLLIN, 0}};   
            szmq::poll(frontends, 2, 0);
            int reroutable = 0;
            //  We'll do peer brokers first, to prevent starvation
            if (frontends [1].revents & ZMQ_POLLIN) {
                msgs = cloudfe.recvMultiple();
                reroutable = 0;
            }
            else
            if (frontends [0].revents & ZMQ_POLLIN) {
                msgs = localfe.recvMultiple();
                reroutable = 1;
            }
            else
                break;      //  No work, go back to backends

            //  If reroutable, send to cloud 20% of the time
            //  Here we'd normally use cloud status information
            //
            if (reroutable && argc > 2 && (rand() % 5) == 0) {
                //  Route to random broker peer
                int peer = rand() % (argc - 2) + 2;
                msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
                msgs.insert(msgs.begin(), szmq::Message::from(string(argv[peer])));	
                cloudbe.sendMultiple(msgs); msgs.clear();
            }
            else {
                msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
                msgs.insert(msgs.begin(), worker_queue.front()); //worker_queue [0];
                worker_queue.pop();
                localbe.sendMultiple(msgs); msgs.clear();
            }
        }
    }
    //  When we're done, clean up properly
	while (worker_queue.size()) {
		worker_queue.pop();
	}	
    cloudfe.close();
    cloudbe.close();
    localfe.close();
    localbe.close();
    return EXIT_SUCCESS;
}
```

### 빌드 및 테스트

~~~{.bash}
// 원도우
PS D:\work\sook\src\szmq\examples> cl -EHsc peering2_tcp.java szmq.lib


PS D:\work\sook\src\szmq\examples> ./peering2_tcp 6000 5000
PS D:\work\sook\src\szmq\examples> ./peering2_tcp 5000 6000
I: preparing broker at 5000...
I: connecting to cloud frontend at 6002
Press Enter when all brokers are started:
worker : tcp://localhost:5001
worker : tcp://localhost:5001
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
worker : tcp://localhost:5001
Worker: HELLO
Worker: HELLO
Worker: HELLO
Client: OK
...

// 리눅스
zedo@sook:/work/sook/src/szmq/examples$ g++ -o peering2_tcp peering2_tcp.java -lsook-szmq -lzmq -lpthread
zedo@sook:/work/sook/src/szmq/examples$ ./peering2_tcp 5000 6000
I: preparing broker at 5000...
I: connecting to cloud frontend at 6002
Press Enter when all brokers are started: 
worker : tcp://localhost:5001
worker : tcp://localhost:5001
worker : tcp://localhost:5001
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
client : tcp://localhost:5000
Worker: HELLO
client : tcp://localhost:5000
client : tcp://localhost:
client : tcp://localhost:50005000
client : tcp://localhost:5000
Worker: HELLO
Client: OK
...
~~~

### 결합하기

위의 예제들을 하나의 패키지로 합치겠습니다. 이전에는 전체 클러스터를 하나의 프로세스로 실행합니다. 2가지 예제를 가져와서 하나로 병합하여 원하는 수의 클러스터들을 시뮬레이션할 수 있도록 하겠습니다.

이 코드는 270줄로 이전 2개의 예제를 합친 크기입니다. 이는 클라이언트들과 작업자들 및 클라우드 작업부하를 분산시키는 클러스터 시뮬레이션에 좋습니다. 다음은 코드입니다.

### peering3.java : 클로스터 전체 시뮬레이션

peering3.c는 ipc 전송 방식을 사용하여 원도우에서 사용 가능한 tcp 전송 방식으로 변경하겠습니다. 변경을 위하여 바인드와 연결에 사용되는 포트 번호를 지정합니다.

localfe : self
localbe : atoi(self)+1
cloudfe : atoi(self)+2
cloudbe : atoi(peer)+2
statefe : atoi(peer)+4
statebe : atoi(self)+4
monitor : atoi(self)+3  --> 모니터링을 위하여 신규 포트 추가됨

```java
//  Broker peering simulation (part 2)
//  Prototypes the request-reply flow
#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
#include <cstdlib>
#include <thread>
#include <queue>
#define NBR_CLIENTS 10
#define NBR_WORKERS 5
#define WORKER_READY   "\001"      //  Signals worker is ready

using namespace std;

//  Our own name; in practice this would be configured per node
static char *self;

//  .split client task
//  The client task does a request-reply dialog using a standard
//  synchronous REQ socket:

static void *
client_task (void *args)
{
    szmq::Context context;
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> client(context);
    client.connect(szmq::SocketUrl(boost::str(boost::format("tcp://localhost:%1%") % self)));
    cout << boost::format("[d]client : tcp://localhost:%1%\n") % self;
    szmq::Socket<ZMQ_PUSH, szmq::ZMQ_CLIENT> monitor(context);
    monitor.connect(szmq::SocketUrl(boost::str(boost::format("tcp://localhost:%1%") % (stoi(self)+3))));
    cout << boost::format("[d]monitor : tcp://localhost:%1%\n") % (stoi(self)+3);
    while (true) {
        std::this_thread::sleep_for(std::chrono::milliseconds((rand() % 5)* 100)); 
        int burst = rand() % 15;
        while (burst--) {
            auto task_id =  boost::str(boost::format("%1$06X") % (rand() % 0x1000000)); 
            //  Send request, get reply
            client.sendOne(szmq::Message::from(task_id));
            std::vector<szmq::PollItem> pollset = {
			    {reinterpret_cast<void*>(*client), 0, ZMQ_POLLIN, 0}};
            szmq::poll(pollset, 1, 10*1000);    // 10 seconds timeout
            if (pollset [0].revents & ZMQ_POLLIN) {
                auto reply = client.recvOne();               
                monitor.sendOne(szmq::Message::from(boost::str(boost::format("I: CLIENT recived reply %1%") % reply.read<std::string>())));
            } else {
                monitor.sendOne(szmq::Message::from(boost::str(boost::format("E: CLIENT EXIT - lost task %1%") % task_id)));
                return NULL;
            }
        } 
    }
    client.close();
    monitor.close();
    return NULL;
}

//  .split worker task
//  The worker task plugs into the load-balancer using a REQ
//  socket:

static void *
worker_task (void *args)
{
    szmq::Context context;
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> worker(context);
    worker.connect(szmq::SocketUrl(boost::str(boost::format("tcp://localhost:%1%") % (stoi(self)+1))));
    cout << boost::format("[d]worker : tcp://localhost:%1%\n") % (stoi(self)+1);
    //  Tell broker we're ready for work
    worker.sendOne(szmq::Message::from(WORKER_READY));

    //  Process messages as they arrive
    while (true) {
        auto msgs = worker.recvMultiple();
        std::this_thread::sleep_for(std::chrono::milliseconds((rand() % 2)* 1000)); 
        worker.sendMultiple(msgs);
    }
    worker.close();
    return NULL;
}

//  .split main task
//  The main task begins by setting-up its frontend and backend sockets
//  and then starting its client and worker tasks:

int main (int argc, char *argv [])
{
    //  First argument is this broker's name
    //  Other arguments are our peers' names
    //
    if (argc < 2) {
        cout << "syntax: peering3 me {you}...\n";
        return 0;
    }
    self = argv [1];
    cout <<"[d]I: preparing broker at " << self << "...\n";
    srand (static_cast<unsigned int>(std::time(0)));

    szmq::Context context;
    //  Prepare local frontend and backend
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> localfe(context);    
    localfe.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % self)));
    cout << "[d]localfe : tcp::/*:" << self << endl;

    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> localbe(context);
    localbe.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % (stoi(self)+1))));
    cout << "[d]localbe : tcp::/*:" << stoi(self)+1 << endl;

    //  Bind cloud frontend to endpoint
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> cloudfe(context); 
    cloudfe.setId(string(self));
    cloudfe.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % (stoi(self)+2))));

    //  Connect cloud backend to all peers
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_CLIENT> cloudbe(context); 
    cloudbe.setId(string(self));
    int argn;
    for (argn = 2; argn < argc; argn++) {
        char *peer = argv [argn];
        cout <<"I: connecting to cloud frontend at " << stoi(peer)+2 << endl;
        cloudbe.connect(szmq::SocketUrl(boost::str(boost::format("tcp://localhost:%1%") % (stoi(peer)+2))));
    }

    //  Bind state backend to endpoint
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> statebe (context); 
    statebe.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % (stoi(self)+4))));
    cout << "[d]statebe : tcp::/*:" << stoi(self)+4 << endl;

    //  Connect state frontend to all peers
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> statefe(context); 
    statefe.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    for (argn = 2; argn < argc; argn++) {
        char *peer = argv [argn];
        cout <<"I: connecting to state backend at " << stoi(peer)+4<< endl;
        statefe.connect(szmq::SocketUrl(boost::str(boost::format("tcp://localhost:%1%") % (stoi(peer)+4))));
    }
    //  Prepare monitor socket
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_SERVER> monitor(context); 
    monitor.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % (stoi(self)+3))));
    cout << "[d]monitor : tcp::/*:" << stoi(self)+3 << endl;

    //  Get user to tell us when we can start...
    cout << "Press Enter when all brokers are started: ";
    getchar ();

    //  Start local workers
  int worker_nbr;
    for (worker_nbr = 0; worker_nbr < NBR_WORKERS; worker_nbr++){
        thread worker(&worker_task, nullptr);
        worker.detach();
    }
    //  Start local clients
    int client_nbr;
    for (client_nbr = 0; client_nbr < NBR_CLIENTS; client_nbr++){
        thread client(&client_task, nullptr);
        client.detach();
    }

    //  .split main loop
    //  The main loop has two parts. First, we poll workers and our two service
    //  sockets (statefe and monitor), in any case. If we have no ready workers,
    //  then there's no point in looking at incoming requests. These can remain 
    //  on their internal ØMQ queues:

    //  Least recently used queue of available workers
    int local_capacity = 0;
    int cloud_capacity = 0;
    std::queue<szmq::Message> worker_queue;

    while (true) {
        //  First, route any waiting replies from workers
        std::vector<szmq::PollItem> primary = {
            {reinterpret_cast<void*>(*localbe), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*cloudbe), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*statefe), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*monitor), 0, ZMQ_POLLIN, 0}};        
        //  If we have no workers, wait indefinitely
        szmq::poll(primary, 4, local_capacity? 1000 : -1);
        //  Track if capacity changes during this iteration
        int previous = local_capacity;
        //  Handle reply from local worker
        std::vector<szmq::Message> msgs;
        if (primary [0].revents & ZMQ_POLLIN) {
            msgs = localbe.recvMultiple();
            auto identity = msgs.front();
            worker_queue.push(identity);
            local_capacity++;
            msgs.erase(msgs.begin()); // delete the worker_id
            msgs.erase(msgs.begin()); // delete the empty delimiter	
            //  If it's READY, don't route the message any further
            auto client_id = msgs.front();
            if((client_id.read<std::string>().compare(WORKER_READY)) == 0)
                msgs.clear();
        }
        //  Or handle reply from peer broker
        else
        if (primary [1].revents & ZMQ_POLLIN) {
            msgs = cloudbe.recvMultiple();
            //  We don't use peer broker identity for anything
            auto identity = msgs.front();
            msgs.erase(msgs.begin()); // delete the cloudbe id
            msgs.erase(msgs.begin()); // delete the empty delimiter	
        }
        //  Route reply to cloud if it's addressed to a broker
        for (argn = 2; msgs.size() && argn < argc; argn++) {
            auto client_id = msgs.front();
            if((client_id.read<std::string>().compare(argv [argn])) == 0){
                cloudfe.sendMultiple(msgs); msgs.clear();
            }
        }
        //  Route reply to client if we still need to
        if (msgs.size()){
            localfe.sendMultiple(msgs); msgs.clear();
        }
        //  .split handle state messages
        //  If we have input messages on our statefe or monitor sockets, we
        //  can process these immediately:

        if (primary [2].revents & ZMQ_POLLIN) {
            auto peer = statefe.recvOne();
            auto status = statefe.recvOne().read<int>();
            cloud_capacity = status;
        }
        if (primary [3].revents & ZMQ_POLLIN) {
            auto status = monitor.recvOne().read<std::string>();
            cout << status << endl;
        }
        //  .split route client requests
        //  Now we route as many client requests as we have worker capacity
        //  for. We may reroute requests from our local frontend, but not from 
        //  the cloud frontend. We reroute randomly now, just to test things
        //  out. In the next version, we'll do this properly by calculating
        //  cloud capacity:
        while (local_capacity + cloud_capacity) {
            std::vector<szmq::PollItem> secondary  = {
                {reinterpret_cast<void*>(*localfe), 0, ZMQ_POLLIN, 0},
                {reinterpret_cast<void*>(*cloudfe), 0, ZMQ_POLLIN, 0}};   
            szmq::poll(secondary, local_capacity ? 2 : 1, 0);
            //  We'll do peer brokers first, to prevent starvation
            if (secondary [0].revents & ZMQ_POLLIN)
                msgs = localfe.recvMultiple();
            else
            if (secondary [1].revents & ZMQ_POLLIN) 
                msgs = cloudfe.recvMultiple();
            else
                break;      //  No work, go back to backends

            if (local_capacity) {
                msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
                msgs.insert(msgs.begin(), worker_queue.front()); //worker_queue [0];
                worker_queue.pop();
                localbe.sendMultiple(msgs); msgs.clear();
                local_capacity--;
            }
            else {
                //  Route to random broker peer
                int peer = rand() % (argc - 2) + 2;
                msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
                msgs.insert(msgs.begin(), szmq::Message::from(string(argv[peer])));	
                for (auto it = begin (msgs); it != end (msgs); ++it) 
                    it->dump();
                cloudbe.sendMultiple(msgs); msgs.clear();
            }
        }
        //  .split broadcast capacity
        //  We broadcast capacity messages to other peers; to reduce chatter,
        //  we do this only if our capacity changed.

        if (local_capacity != previous) {
            //  We stick our own identity onto the envelope
            statebe.sendOne(szmq::Message::from(self));
            //  Broadcast new capacity
            statebe.sendOne(szmq::Message::from(local_capacity));
        }
    }
    //  When we're done, clean up properly
	while (worker_queue.size()) {
		worker_queue.pop();
	}	
    localfe.close();
    localbe.close();    
    cloudfe.close();
    cloudbe.close();
    return EXIT_SUCCESS;
}
```

### 빌드 및 테스트

~~~{.bash}
// 원도우
D:\work\sook\src\szmq\examples> cl -EHsc -MTd peering3_tcp.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./peering3_tcp 5000 6000
[d]I: preparing broker at 5000...
[d]localfe : tcp::/*:5000
[d]localbe : tcp::/*:5001
I: connecting to cloud frontend at 6002
[d]statebe : tcp::/*:5004
I: connecting to state backend at 6004
[d]monitor : tcp::/*:5003
Press Enter when all brokers are started:
[d]monitor : tcp://localhost:5003
[d]monitor : tcp://localhost:5003
I: CLIENT recived reply 0018BE
I: CLIENT recived reply 0018BE
I: CLIENT recived reply 0018BE
I: CLIENT recived reply 0018BE
I: CLIENT recived reply 0018BE
...

PS D:\work\sook\src\szmq\examples> ./peering3_tcp 6000 5000
[d]I: preparing broker at 6000...
[d]localfe : tcp::/*:6000
[d]localbe : tcp::/*:6001
I: connecting to cloud frontend at 5002
[d]statebe : tcp::/*:6004
I: connecting to state backend at 5004
[d]monitor : tcp::/*:6003
Press Enter when all brokers are started:
...

//리눅스
zedo@sook:/work/sook/src/szmq/examples$ g++ -o peering3_tcp peering3_tcp.java -lsook-szmq -lzmq -lpthread 
zedo@sook:/work/sook/src/szmq/examples$ ./peering3_tcp 5000 6000
[d]I: preparing broker at 5000...
[d]localfe : tcp::/*:5000
[d]localbe : tcp::/*:5001
I: connecting to cloud frontend at 6002
[d]statebe : tcp::/*:5004
I: connecting to state backend at 6004
[d]monitor : tcp::/*:5003
Press Enter when all brokers are started: 
[d]worker : tcp://localhost:[d]worker : tcp://localhost:50015001
...
[d]monitor : tcp://localhost:5003
I: CLIENT recived reply 529353
I: CLIENT recived reply 90CA99
...
zedo@sook:/work/sook/src/szmq/examples$ ./peering3_tcp 6000 5000
[d]I: preparing broker at 6000...
[d]localfe : tcp::/*:6000
[d]localbe : tcp::/*:6001
...
~~~

* peering3_inproc.java : 정상적으로 동작합니다.

```java
//  Broker peering simulation (part 2)
//  Prototypes the request-reply flow
#include <szmq/szmq.h>
#include <iostream>
#include <czmq.h>
#include <cstdlib>
#include <thread>
#include <queue>
#define NBR_CLIENTS 10
#define NBR_WORKERS 5
#define WORKER_READY   "\001"      //  Signals worker is ready

using namespace std;


//  .split client task
//  The client task does a request-reply dialog using a standard
//  synchronous REQ socket:

static void 
client_task (void *context, char* self)
{   
    int rc;
    szmq::Context *ctx = (szmq::Context *) context;
    char fmt[255];
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> client(*ctx);    
    snprintf(fmt, 255, "inproc://%s-localfe", self);
    client.connect(szmq::SocketUrl(fmt)); 
    szmq::Socket<ZMQ_PUSH, szmq::ZMQ_CLIENT> monitor(*ctx);
    snprintf(fmt, 255, "inproc://%s-monitor", self);
    monitor.connect(szmq::SocketUrl(fmt));
    while (true) {
        zclock_sleep((rand() % 5));
        int burst = rand() % 15;
        while (burst--) {
            char task_id[5];
            snprintf(task_id, 5, "%04X", rand() % 0x10000);
            //  Send request, get reply
            rc = client.sendOne(szmq::Message::from(string(task_id)));
            if (rc == -1) printf("E: client.sendOne() : %s\n",zmq_strerror(zmq_errno()));
            std::vector<szmq::PollItem> pollset = {
			    {reinterpret_cast<void*>(*client), 0, ZMQ_POLLIN, 0}};
            rc = szmq::poll(pollset, 1, 10*1000);    // 10 seconds timeout
            if (rc == -1) printf("E: [client]szmq::poll() : %s\n",zmq_strerror(zmq_errno()));
            if (pollset [0].revents & ZMQ_POLLIN) {
                auto reply = client.recvOne().read<std::string>();
                snprintf(fmt, 255, "[%s]I: CLIENT recived reply %s", self, reply.c_str());               
                rc = monitor.sendOne(szmq::Message::from(fmt));
                if (rc == -1) printf("E: [client]monitor.sendOne() : %s\n",zmq_strerror(zmq_errno()));
            } else {
                snprintf(fmt, 255, "[%s]E: CLIENT EXIT - lost task %s", self, task_id);   
                rc = monitor.sendOne(szmq::Message::from(fmt));
                if (rc == -1) printf("E: [client]monitor.sendOne() : %s\n",zmq_strerror(zmq_errno()));
                client.close();
                monitor.close();
                return;
            }
        } 
    }
    client.close();
    monitor.close();
}

//  .split worker task
//  The worker task plugs into the load-balancer using a REQ
//  socket:

static void 
worker_task (void *context, char* self)
{
    int rc;
    szmq::Context *ctx = (szmq::Context *) context;
    char fmt[255];
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> worker(*ctx);
    snprintf(fmt, 255, "inproc://%s-localbe", self);
    worker.connect(szmq::SocketUrl(fmt));
    szmq::Socket<ZMQ_PUSH, szmq::ZMQ_CLIENT> monitor(*ctx);
    snprintf(fmt, 255, "inproc://%s-monitor", self);
    monitor.connect(szmq::SocketUrl(fmt));
    //  Tell broker we're ready for work
    rc = worker.sendOne(szmq::Message::from(WORKER_READY));
    if (rc == 0) printf("E: worker.sendOne() : %s\n",zmq_strerror(zmq_errno()));
    //  Process messages as they arrive
    while (true) {
        auto msgs = worker.recvN();
        auto msgbuffer = msgs.back().read<std::string>();
        snprintf(fmt, 255, "[%s] Worker received :  %s", self, msgbuffer.c_str());   
        rc = monitor.sendOne(szmq::Message::from(fmt));     
        if (rc == -1) printf("E: monitor.sendOne() : %s\n",zmq_strerror(zmq_errno()));   
        zclock_sleep((rand() % 2));
        rc = worker.sendMultiple(msgs); msgs.clear();
        if (rc == 0) printf("E: worker.sendMultiple() : %s\n",zmq_strerror(zmq_errno()));
    }
    worker.close();
}

//  .split main task
//  The main task begins by setting-up its frontend and backend sockets
//  and then starting its client and worker tasks:
static void 
broker (void *args, char* argv[])
{
    int rc;
    char fmt[255];
    char* self = argv [1];
    cout <<"I: preparing broker at " << self << "...\n";    

    szmq::Context *ctx = (szmq::Context *) args;
    //  Prepare local frontend and backend
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> localfe(*ctx);   
    snprintf(fmt, 255, "inproc://%s-localfe", self); 
    localfe.bind(szmq::SocketUrl(fmt));
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> localbe(*ctx);
    snprintf(fmt, 255, "inproc://%s-localbe", self); 
    localbe.bind(szmq::SocketUrl(fmt));

    //  Bind cloud frontend to endpoint
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> cloudfe(*ctx); 
    cloudfe.setId(string(self));
    //cloudfe.setSockOpt(ZMQ_IDENTITY, self, strlen(self));
    snprintf(fmt, 255, "inproc://%s-cloudfe", self); 
    cloudfe.bind(szmq::SocketUrl(fmt));

    //  Connect cloud backend to all peers
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_CLIENT> cloudbe(*ctx); 
    cloudbe.setId(string(self));
    // cloudbe.setSockOpt(ZMQ_IDENTITY, self, strlen(self));
    int argc = 0;
    while(argv[++argc]);
    for (int argn = 2; argn < argc; argn++) {
        char *peer = argv [argn];
        snprintf(fmt, 255, "[%s] I: connecting to cloud frontend at '%s'", self, peer); 
        cout << fmt << endl;
        snprintf(fmt, 255, "inproc://%s-cloudbe", peer); 
        cloudbe.connect(szmq::SocketUrl(fmt));
    }

    //  Bind state backend to endpoint
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> statebe(*ctx);
    snprintf(fmt, 255, "inproc://%s-state", self); 
    statebe.bind(szmq::SocketUrl(fmt));

    //  Connect state frontend to all peers
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> statefe(*ctx); 
    statefe.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    for (int argn = 2; argn < argc; argn++) {
        char *peer = argv [argn];
        snprintf(fmt, 255, "[%s] I: connecting to state backend at '%s'", self, peer); 
        cout << fmt << endl;
        snprintf(fmt, 255, "inproc://%s-statebe", peer); 
        statefe.connect(szmq::SocketUrl(fmt));
    }
    //  Prepare monitor socket
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_SERVER> monitor(*ctx);
    snprintf(fmt, 255, "inproc://%s-monitor", self);
    monitor.bind(szmq::SocketUrl(fmt));

    std::vector < std::thread > threadPool;
    //  Start local workers
    for (int worker_nbr = 0; worker_nbr < NBR_WORKERS; ++worker_nbr) {
        threadPool.push_back(std::thread([&]() {
            worker_task((void *)ctx, self); // will connect with inproc(localbe)
        }));
    }
    for (int client_nbr = 0; client_nbr < NBR_CLIENTS; ++client_nbr) {
        threadPool.push_back(std::thread([&]() {
            client_task((void *)ctx, self); // will connect with inproc(localbe)
        }));
    }

    //  .split main loop
    //  The main loop has two parts. First, we poll workers and our two service
    //  sockets (statefe and monitor), in any case. If we have no ready workers,
    //  then there's no point in looking at incoming requests. These can remain 
    //  on their internal ØMQ queues:

    //  Least recently used queue of available workers
    int local_capacity = 0;
    int cloud_capacity = 0;
    std::string peer_name;
    std::queue<szmq::Message> worker_queue;

    while (true) {
        //  First, route any waiting replies from workers
        std::vector<szmq::PollItem> primary = {
            {reinterpret_cast<void*>(*localbe), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*cloudbe), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*statefe), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*monitor), 0, ZMQ_POLLIN, 0}};        
        //  If we have no workers, wait indefinitely
        szmq::poll(primary, 4, local_capacity ? 1000 : -1);
        //  Track if capacity changes during this iteration
        int previous = local_capacity;
        std::vector<szmq::Message> msgs;
        //  Handle reply from local worker        
        if (primary [0].revents & ZMQ_POLLIN) {
            msgs = localbe.recvN();
            auto identity = msgs.front();
            worker_queue.push(identity);
            local_capacity++;
            msgs.erase(msgs.begin()); // delete the worker_id
            msgs.erase(msgs.begin()); // delete the empty delimiter	
            //  If it's READY, don't route the message any further
            auto client_id = msgs.front().read<std::string>();
            if((client_id.compare(WORKER_READY)) == 0)
                msgs.clear();
        }
        //  Or handle reply from peer broker
        else
        if (primary [1].revents & ZMQ_POLLIN) {
            msgs = cloudbe.recvN();
            //  We don't use peer broker identity for anything
            auto identity = msgs.front();
            msgs.erase(msgs.begin()); // delete the cloudbe id
            msgs.erase(msgs.begin()); // delete the empty delimiter	
            //  Route reply to cloud if it's addressed to a broker
            for (int argn = 2; msgs.size() && argn < argc; argn++) {
                auto client_id = msgs.front();
                if((client_id.read<std::string>().compare(string(argv [argn]))) == 0){
                    rc = cloudfe.sendMultiple(msgs); msgs.clear();
                    if (rc == 0) printf("E: cloudfe.sendMultiple() : %s\n",zmq_strerror(zmq_errno()));
                }
            }
        }
        //  Route reply to client if we still need to
        if (msgs.size()){
            for (auto it = begin (msgs); it != end (msgs); ++it) 
                it->dump();
            rc = localfe.sendMultiple(msgs); msgs.clear();
            if (rc == 0) printf("E: localfe.sendMultiple() : %s\n",zmq_strerror(zmq_errno()));
        }
        //  .split handle state messages
        //  If we have input messages on our statefe or monitor sockets, we
        //  can process these immediately:

        if (primary [2].revents & ZMQ_POLLIN) {
            peer_name = statefe.recv1().read<std::string>();
            auto statue = statefe.recv1().read<int>();
            cloud_capacity = statue;
        }
        if (primary [3].revents & ZMQ_POLLIN) {
            auto status = monitor.recv1().read<std::string>();
            cout << status << endl;
        }
        //  .split route client requests
        //  Now we route as many client requests as we have worker capacity
        //  for. We may reroute requests from our local frontend, but not from 
        //  the cloud frontend. We reroute randomly now, just to test things
        //  out. In the next version, we'll do this properly by calculating
        //  cloud capacity:
        while (local_capacity + cloud_capacity) {
            std::vector<szmq::PollItem> secondary  = {
                {reinterpret_cast<void*>(*localfe), 0, ZMQ_POLLIN, 0},
                {reinterpret_cast<void*>(*cloudfe), 0, ZMQ_POLLIN, 0}};   
            szmq::poll(secondary, local_capacity ? 2 : 1, 0);
            //  We'll do peer brokers first, to prevent starvation
            if (secondary [0].revents & ZMQ_POLLIN)
                msgs = localfe.recvN();
            else
            if (secondary [1].revents & ZMQ_POLLIN) 
                msgs = cloudfe.recvN();
            else
                break;      //  No work, go back to backends

            if (local_capacity) {
                msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
                msgs.insert(msgs.begin(), worker_queue.front()); //worker_queue [0];
                worker_queue.pop();
                rc = localbe.sendMultiple(msgs); msgs.clear();
                if (rc == 0) printf("E: localbe.sendMultiple() : %s\n",zmq_strerror(zmq_errno()));
                local_capacity--;
            }
            else 
            if (cloud_capacity){
                //  Route to random broker peer
                //int peer = rand() % (argc - 2) + 2;
                msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
                //msgs.insert(msgs.begin(), szmq::Message::from(string(argv[peer])));	
                msgs.insert(msgs.begin(), szmq::Message::from(peer_name));	
                rc = cloudbe.sendMultiple(msgs); msgs.clear();
                if (rc == 0) printf("E: cloudbe.sendMultiple() : %s\n",zmq_strerror(zmq_errno()));
            }
        }
        //  .split broadcast capacity
        //  We broadcast capacity messages to other peers; to reduce chatter,
        //  we do this only if our capacity changed.
        if (local_capacity != previous) {
            //  We stick our own identity onto the envelope
            rc = statebe.sendOne(szmq::Message::from(self));
            if (rc == -1) printf("E: statebe.sendOne() : %s\n",zmq_strerror(zmq_errno()));
            //  Broadcast new capacity
            rc = statebe.sendOne(szmq::Message::from(local_capacity));
            if (rc == -1) printf("E: statebe.sendOne() : %s\n",zmq_strerror(zmq_errno()));
        }
        msgs.clear();
    }
    //  When we're done, clean up properly
	while (worker_queue.size()) {
		worker_queue.pop();
	}	
    localfe.close();
    localbe.close();    
    cloudfe.close();
    cloudbe.close();
    statefe.close();
    statebe.close();
    monitor.close();
}

int main (int argc, char *argv [])
{
    //  First argument is this broker's name
    //  Other arguments are our peers' names
    if (argc < 2) {
        printf ("syntax: peering3_inproc me {you}...\n");
        return 0;
    }    
    szmq::Context ctx;
    srand (static_cast<unsigned int>(std::time(0)));
    std::vector < std::thread > threadPool;
    //  Start broker
    threadPool.push_back(std::thread([&]() {
         broker((void *)&ctx, argv); 
    }));
    zclock_sleep(100);
    // Change the order
    int m,n;
    for(n=2; n < argc; n++){
        char *temp = strdup(argv[1]);
        for(m = 2; m < argc; m++)     
        {            
            argv[m-1] = argv[m];
        }
        argv[m-1] = temp;
        threadPool.push_back(std::thread([&]() {
            broker((void *)&ctx, argv); 
        }));
        zclock_sleep(100);
    }
    while(1);  
    return EXIT_SUCCESS;
}
```

* 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc peering3_inproc.java szmq.lib czmq.lib
PS D:\work\sook\src\szmq\examples> ./peering3_inproc1 dog cat fish bird
I: preparing broker at dog...
[dog] I: connecting to cloud frontend at 'cat'
[dog] I: connecting to cloud frontend at 'fish'
[dog] I: connecting to cloud frontend at 'bird'
[dog] I: connecting to state backend at 'cat'
[dog] I: connecting to state backend at 'fish'
[dog] I: connecting to state backend at 'bird'
I: preparing broker at cat...
[cat] I: connecting to cloud frontend at 'fish'
[cat] I: connecting to cloud frontend at 'bird'
[cat] I: connecting to cloud frontend at 'dog'
[cat] I: connecting to state backend at 'fish'
[cat] I: connecting to state backend at 'bird'
[cat] I: connecting to state backend at 'dog'
....
~~~

# 4장-신뢰할 수 있는 요청-응답 패턴

"3장-고급 요청-응답 패턴"은 ØMQ의 요청-응답 패턴의 고급 사용 방법을 예제와 함께 다루었습니다. 이 장에서는 신뢰성에 대한 일반적인 요구 사항을 보고 ØMQ의 핵심 요청-응답 패턴상에서 일련의 신뢰할 수 있는 메시징 패턴을 구축합니다.

이 장에서는 사용자관점의 요청-응답 패턴에 중점을 두고 있으며, 자체 ØMQ 아키텍처를 설계하는 데 도움이 되는 재사용 가능한 모델입니다.

* 게으른 해적 패턴(LPP) : 클라이언트 측의 신뢰할 수 있는 요청-응답
* 단순한 해적 패턴(SPP) : 부하 분산을 사용한 신뢰할 수 있는 요청-응답
* 편집증 해적 패턴(PPP) : 심박을 통한 신뢰할 수 있는 요청-응답
* 집사(majordomo) 패턴(MDP) : 서비스 지향 신뢰할 수 있는 메세지 대기열
* 타이타닉 패턴 : 디스크 기반 / 연결 해제된 신뢰할 수 있는 메세지 대기열
* 바이너리 스타 패턴 : 기본-백업 서버 장애조치
* 프리랜서 패턴 : 브로커 없는 신뢰할 수 있는 요청 응답

## 클라이언트 측면의 신뢰성(게으른 해적 패턴)

클라이언트 코드의 일부 변경하여 매우 간단하고 신뢰성 있는 요청-응답을 얻을 수 있습니다. 우리는 이것을 게으른 해적 패턴이라고 부릅니다. 수신 차단을 수행하는 대신 다음을 수행합니다. : 

* REQ 소켓을 폴링(poll)하고 응답이 도착했을 때만 수신합니다.
* 제한시간 내에 응답이 도착하지 않으면 요청을 재전송합니다.
* 여러 번의 요청들 후에도 여전히 응답이 없으면 트랜잭션을 포기합니다.

### lpclient.java: 게으른 해적 클라이언트

```java
#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
#define REQUEST_TIMEOUT 2500 // msecs, (>1000!)
#define REQUEST_RETRIES 3 // Before we abandon
#define SERVER_ENDPOINT "tcp://localhost:5555"

using namespace std;

szmq::detail::ClientSocketImpl& s_client_socket(szmq::Context *context)
{
    szmq::detail::ClientSocketImpl *client = new szmq::detail::ClientSocketImpl(ZMQ_REQ, szmq::ZMQ_CLIENT, *context);
    client->connect(szmq::SocketUrl(SERVER_ENDPOINT));
    return *client;
}

int main()
{
    szmq::Context context;
    szmq::detail::ClientSocketImpl& client = s_client_socket(&context);
    cout << "I: Connecting to server...\n";

    int sequence = 0;
    int retries_left = REQUEST_RETRIES;
    cout << "Entering while loop...\n";
    while(retries_left) // interrupt needs to be handled
    {
        // We send a request, then we get a reply
        auto request = boost::str(boost::format("%1%") % ++sequence);
        client.sendOne(szmq::Message::from(request));
        int expect_reply = 1;
        while(expect_reply)
        {
            cout << "Expecting reply....\n";
            std::vector<szmq::PollItem> items = {
			    {reinterpret_cast<void*>(*client), 0, ZMQ_POLLIN, 0}};
            szmq::poll(items, 1, REQUEST_TIMEOUT);    // 2.5 seconds timeout
            // Here we process a server reply and exit our loop if the
            // reply is valid. If we didn't get a reply we close the
            // client socket, open it again and resend the request. We
            // try a number times before finally abandoning:
            if (items[0].revents & ZMQ_POLLIN)
            {
                // We got a reply from the server, must match sequence
                auto reply = client.recvOne().read<std::string>();
                cout << "reply : " << reply << endl;
                if (stoi(reply) == sequence)
                {
                    cout << "I: server replied OK (" << reply << ")\n";
                    retries_left=REQUEST_RETRIES;
                    expect_reply = 0;
                }
                else
                {
                    cout << "E: malformed reply from server: " << reply << endl;
                }
            }
            else 
            {
                if(--retries_left == 0)
                {
                    cout << "E: Server seems to be offline, abandoning\n";
                    break;
                }
                else
                {
                    cout << "W: no response from server, retrying...\n";
                    cout << "I: reconnecting to server...\n";
                    client = std::move(s_client_socket(&context));
                    client.sendOne(szmq::Message::from(request));
                }
            }
        }
    }
    client.close();
    return 0;
}
```

대응하는 서버를 함께 구동합니다.

### lpserver.java : 게으른 해적 서버

```java
//  Lazy Pirate server
//  Binds REQ socket to tcp://*:5555
//  Like hwserver except:
//   - echoes request as-is
//   - randomly runs slowly, or exits to simulate a crash.

#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
#include <cstdlib>
#include <thread>

using namespace std;
int main (void)
{
    srand (static_cast<unsigned int>(std::time(0)));

    szmq::Context context;
    szmq::Socket<ZMQ_REP, szmq::ZMQ_SERVER> server(context);
    server.bind(szmq::SocketUrl("tcp://*:5555"));

    int cycles = 0;
    while (1) {
        auto request = server.recvOne().read<std::string>();
        cycles++;
        //  Simulate various problems, after a few cycles
        if (cycles > 3 && (rand() % 3 == 0)) {
            cout << "I: simulating a crash\n";
            break;
        }
        else
        if (cycles > 3 &&  (rand() % 3 == 0)) {
            cout << "I: simulating CPU overload\n";
            std::this_thread::sleep_for(std::chrono::milliseconds(2000)); 
        }
        cout << "I: normal request (" << request << ")\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));     //  Do some heavy work
        server.sendOne(szmq::Message::from(request));
    }
    server.close();
    return 0;
}
```

이 테스트 케이스를 실행하려면 2개의 콘솔 창에서 클라이언트와 서버를 시작하십시오. 서버는 몇 개의 메시지들 받은 이후 무작위로 오동작합니다. 클라이언트의 응답을 확인할 수 있습니다. 다음은 서버부터의 전형적인 출력 예시입니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd lpserver.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd lpclient.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./lpserver
I: normal request (1)
I: normal request (1)
I: normal request (1)
I: simulating CPU overload
I: normal request (1)

PS D:\work\sook\src\szmq\examples> ./lpclient
I: Connecting to server...
Entering while loop...
Expecting reply....
After polling
Polling Done..
W: no response from server, retrying...
I: reconnecting to server...
Expecting reply....
After polling
Polling Done..
reply : 1
I: server replied OK (1)
~~~

클라이언트는 REQ 소켓의 엄격한 송/수신 주기를 지키기 위하여 무식한 강제 종료/재오픈을 수행합니다. REQ 소켓(sync) 대신에 DEALER 소켓(async)을 사용하고 싶겠지만 좋은 결정은 아닙니다. 첫째, 그것은 REQ 소켓 봉투들 모방하는 것이 복잡하고(그것이 무엇인지 잊었다면, 그것을 하고 싶지 않은 좋은 징조입니다), 둘째, 예상하지 못한 응답들을 받을 가능성이 있습니다.

DEALER의 경우 멀티파트 메시지(multipart message) 형태로 첫 번째 파트는 공백 구분자(empty delimter), 두 번째 파트는 데이터(body)로 REP 소켓에 데이터 전송 필요합니다.

### hwclient2.java : REP 대신에 DEALER 소켓을 사용

```java
/**
**Hello World client
DEALER socket connect to tcp://localhost:5555
send "Hello" to server，expect receive "World"
*/
#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
using namespace std;

int main (void)
{
	cout << "Connecting to hello world server...\n";
	szmq::Context context;
	szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> requester(context);
    requester.connect(szmq::SocketUrl("tcp://localhost:5555"));
	int request_nbr;	//request number
	int reply_nbr = 0;  //receive respond number

	for (request_nbr = 0; request_nbr < 10; request_nbr++)
	{
		cout << "Sending request msg: Hello NO=" << request_nbr+1 << endl;
		//send request msg to server
		requester.sendMore(szmq::Message::from(""));  //send multi part msg,the first part is empty part
		requester.sendOne(szmq::Message::from("Hello")); //the second part is your request msg
		//receive reply msg
		if (requester.recvOne().read<std::string>().compare("") == 0){
			auto reply = requester.recvOne().read<std::string>();
			cout << boost::format("Received respond msg: %1% NO=%2%\n\n") % reply % ++reply_nbr;
		}
		//if the first part you received is not empty part,discard the whole ZMQ msg
		else{
			cout << "Discard the ZMQ message!\n";
		}
	}
	requester.close();
	return 0;
}

```

hwserver는 REP소켓을 사용하며 수정 없이 사용 가능합니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc  hwclient2.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./hwserver
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello
received request: Hello

PS D:\work\sook\src\szmq\examples> ./hwclient2
Connecting to hello world server...
Sending request msg: Hello NO=1
Received respond msg: World NO=1
Sending request msg: Hello NO=2
Received respond msg: World NO=2
Sending request msg: Hello NO=3
Received respond msg: World NO=3
Sending request msg: Hello NO=4
Received respond msg: World NO=4
Sending request msg: Hello NO=5
Received respond msg: World NO=5
Sending request msg: Hello NO=6
Received respond msg: World NO=6
Sending request msg: Hello NO=7
Received respond msg: World NO=
Sending request msg: Hello NO=8
Received respond msg: World NO=8
Sending request msg: Hello NO=9
Received respond msg: World NO=9
Sending request msg: Hello NO=10
Received respond msg: World NO=10
~~~

## 신뢰할 수 있는 큐잉(단순한 해적 패턴)

두 번째 접근 방식은 대기열 프록시를 사용하여 게으른 해적 패턴을 확장하여 투명하게 여러 대의 서버들과 통신하게 하며, 서버들은 좀 더 정확하게 "작업자들"라고 부를 수 있습니다. 최소한의 구동 모델인 단순한 해적 패턴부터 시작하여 단계별로 개발할 것입니다.

단순한 해적 패턴에서 작업자들은 상태 비저장입니다. 응용프로그램은 데이터베이스와 같이 공유 상태가 필요하지만 메시징 프레임워크를 설계할 당시에는 인지하지 못할 수도 있습니다. 대기열 프록시가 있다는 것은 클라이언트가 작업자들이 오가는 것을 인지하지 못하는 것을 의미합니다. 한 작업자가 죽을 경우, 다른 작업자가 인계받습니다. 이것은 멋지고 간단한 전송 방식이지만 하나의 실제 약점으로, 중앙 대기열 자체가 단일 실패 지점으로 관리해야 할 문제가 됩니다.

대기열 프록시의 기본은 “3장-고급 요청-응답 패턴”의 부하 분산 브로커입니다. “죽거나 차단된 작업자들을 처리하기 위해 최소한 해야 할 것은 무엇입니까?” 실제 해야 할 것은 의외로 적습니다. 우리는 이미 클라이언트들에서 재시도 기능을 부가한 적 있으며 따라서 부하 분산 패턴을 사용하면 꽤 잘 작동할 것입니다. 이것은 QMQ의 철학에 부합하며 P2P(peer-to-peer)와 같은 요청-응답 패턴 사이에 프록시를 추가하여 확장할 수 있습니다.

여기에는 특별한 클라이언트가 필요하지 않습니다. 이전의 게으른 해적 클라이언트(lpclient)를 사용할 수 있으며, 대기열은 부하 분산 브로커의 기본 작업과 동일합니다 

### spqueue.java : 단순한 해적 브로커

```java
//  Simple Pirate broker
//  This is identical to load-balancing pattern, with no reliability
//  mechanisms. It depends on the client for recovery. Runs forever.
#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
#include <queue>
#define WORKER_READY   "\001"      //  Signals worker is ready

using namespace std;

int main (void)
{
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);    
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> backend(context);
    frontend.bind(szmq::SocketUrl("tcp://*:5555"));  //  For clients
    backend.bind(szmq::SocketUrl("tcp://*:5556"));   //  For workers

    //  Queue of available workers
    std::queue<szmq::Message> worker_queue;
    
    //  The body of this example is exactly the same as lbbroker2.
    //  .skip
    while (true) {
        std::vector<szmq::PollItem> items = {
            {reinterpret_cast<void*>(*backend), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN, 0}};
        //  Poll frontend only if we have available workers
        szmq::poll(items, worker_queue.size() ? 2 : 1, -1);
        //  Handle worker activity on backend
        if (items [0].revents & ZMQ_POLLIN) {
            //  Use worker identity for load-balancing
            auto msgs = backend.recvMultiple();
            auto identity = msgs.front();   //worker ID
            worker_queue.push(identity);
			msgs.erase(msgs.begin()); // delete the worker_id
			msgs.erase(msgs.begin()); // delete the empty delimiter	

            //  Forward message to client if it's not a READY
            auto client_id = msgs.front();
              //  If client reply, send rest back to frontend            
            if ((client_id.read<std::string>().compare(WORKER_READY)) == 0)    
                msgs.clear();
            else
                frontend.sendMultiple(msgs);	// clientID + "" + reply
        }
        if (items [1].revents & ZMQ_POLLIN) {
            //  Get client request, route to first available worker
            auto msgs = frontend.recvMultiple();
            msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
			msgs.insert(msgs.begin(), worker_queue.front());			
            worker_queue.pop();
            backend.sendMultiple(msgs);  // WorkerID + "" + ClientID + "" + 
        }
    }
    //  When we're done, clean up properly
	while (worker_queue.size()) {
		worker_queue.pop();
	}	
    frontend.close();
    backend.close();
    return 0;
    //  .until
}
```

다음은 작업자 코드로, 게으른 해적 서버(lpserver)를 가져 와서 부하분산 패턴에 맞게 적용하였습니다.(REQ 소켓 및 “준비(ready)” 신호 사용).

### spworker.java: 단순한 해적 작업자

```java
//  Simple Pirate worker
//  Connects REQ socket to tcp://*:5556
//  Implements worker part of load-balancing
#include <szmq/szmq.h>
#include <iostream>
#include <iomanip>
#include <sstream>
#include <boost/format.hpp>
#include <cstdlib>
#include <thread>
#define WORKER_READY   "\001"      //  Signals worker is ready

using namespace std;
int main (void)
{
    srand (static_cast<unsigned int>(std::time(0)));
    szmq::Context context;
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> worker(context);
    std::stringstream identity;
    identity << std::hex << std::uppercase
        << std::setw(4) << std::setfill('0') << rand() % (0x10000) << "-"
        << std::setw(4) << std::setfill('0') << rand() % (0x10000);
    worker.setId(identity.str());
    worker.connect(szmq::SocketUrl("tcp://localhost:5556"));

    cout << boost::format("I: (%1%) worker ready\n") % identity.str();
    worker.sendOne(szmq::Message::from(WORKER_READY));

    int cycles = 0;
    while (true) {
        auto msgs = worker.recvMultiple();
        cycles++;
        //  Simulate various problems, after a few cycles
        if (cycles > 3 && (rand() % 5 == 0)) {
            cout << boost::format("I: (%1%) simulating a crash\n") % identity.str();
            break;
        }
        else
        if (cycles > 3 &&  (rand() % 5 == 0)) {
            cout << boost::format("I: (%1%) simulating CPU overload\n") % identity.str();
            std::this_thread::sleep_for(std::chrono::milliseconds(3000)); 
        }
        cout << boost::format("I: (%1%) normal reply\n") % identity.str();
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));     //  Do some heavy work
        worker.sendMultiple(msgs);
    }
    worker.close();
    return 0;
}
```

이를 테스트하려면 순서에 관계없이 여러 개의 작업자들, 게으른 해적 클라이언트(lpclient) 및 대기열(spqueue)을 시작하십시오. 그러면 결국 작업자들이 모두 중단되는 것을 볼 수 있으며 클라이언트는 재시도(3회)를 수행한 후 포기합니다. 대기열은 멈추지 않으며 작업자들과 클라이언트들을 계속해서 재시작할 수 있습니다. 이 모델은 어떤 수량의 클라이언트들과 작업자라도 함께 작동합니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd spqueue.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd spworker.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./spqueue

PS D:\work\sook\src\szmq\examples> ./spworker
I: (3CE5-138F) worker ready
I: (3CE5-138F) normal reply
I: (3CE5-138F) normal reply
I: (3CE5-138F) normal reply
I: (3CE5-138F) simulating a crash

PS D:\work\sook\src\szmq\examples> ./lpclient
I: Connecting to server...
Entering while loop...
Expecting reply....
After polling
Polling Done..
W: no response from server, retrying...
I: reconnecting to server...
Expecting reply....
After polling
Polling Done..
W: no response from server, retrying...
I: reconnecting to server...
Expecting reply....
After polling
Polling Done..
E: Server seems to be offline, abandoning
~~~

spqueue와 spworker, lpclient를 하나의 프로세스로 구동할 수 있도록 변경한 프로그램은 다음과 같습니다.

sppattern.java : inproc를 통한 spqueue와 spworker, lpclient 함께 구동

```java
//  Simple Pirate broker
//  This is identical to load-balancing pattern, with no reliability
//  mechanisms. It depends on the client for recovery. Runs forever.
#include <szmq/szmq.h>
#include <iostream>
#include <iomanip>
#include <sstream>
#include <thread>
#include <boost/format.hpp>
#include <queue>
#define NBR_CLIENTS 3
#define NBR_WORKERS 3
#define REQUEST_TIMEOUT 2500 // msecs, (>1000!)
#define REQUEST_RETRIES 3 // Before we abandon
#define FRONTEND "inproc://frontend"
#define BACKEND "inproc://backend"
#define WORKER_READY   "\001"      //  Signals worker is ready

using namespace std;

szmq::detail::ClientSocketImpl& s_client_socket(szmq::Context& context)
{
    szmq::detail::ClientSocketImpl *client = new szmq::detail::ClientSocketImpl(ZMQ_REQ, szmq::ZMQ_CLIENT, context);
    client->connect(szmq::SocketUrl(FRONTEND));
    return *client;
}

//  Lazy Pirate client using REQ socket
//
static void *
client_task (void *ctx)
{
    szmq::Context *context = (szmq::Context *)ctx;
    szmq::detail::ClientSocketImpl& client = s_client_socket(*context);
    client.connect(szmq::SocketUrl(FRONTEND));
    cout << "I: Connecting to server...\n";

    int sequence = 0;
    int retries_left = REQUEST_RETRIES;
    cout << "Entering while loop...\n";
    while(retries_left) // interrupt needs to be handled
    {
        // We send a request, then we get a reply
        auto request = boost::str(boost::format("%1%") % ++sequence);
        client.sendOne(szmq::Message::from(request));
        int expect_reply = 1;
        while(expect_reply)
        {
            cout << "Expecting reply....\n";
            std::vector<szmq::PollItem> items = {
			    {reinterpret_cast<void*>(*client), 0, ZMQ_POLLIN, 0}};
            cout << "After polling\n";
            szmq::poll(items, 1, REQUEST_TIMEOUT);    // 2.5 seconds timeout
            cout << "Polling Done.. \n";
            // Here we process a server reply and exit our loop if the
            // reply is valid. If we didn't get a reply we close the
            // client socket, open it again and resend the request. We
            // try a number times before finally abandoning:
            if (items[0].revents & ZMQ_POLLIN)
            {
                // We got a reply from the server, must match sequence
                auto reply = client.recvOne().read<std::string>();
                cout << "reply : " << reply << endl;
                if (stoi(reply) == sequence)
                {
                    cout << "I: server replied OK (" << reply << ")\n";
                    retries_left=REQUEST_RETRIES;
                    expect_reply = 0;
                }
                else
                {
                    cout << "E: malformed reply from server: " << reply << endl;
                }
            }
            else 
            {
                if(--retries_left == 0)
                {
                    cout << "E: Server seems to be offline, abandoning\n";
                    break;
                }
                else
                {
                    cout << "W: no response from server, retrying...\n";
                    cout << "I: reconnecting to server...\n";
                    client = std::move(s_client_socket(*context));
                    client.sendOne(szmq::Message::from(request));
                }
            }
        }
    }
    client.close();
    return NULL;
}

//  Lazy Pirate server using REQ socket
//
static void *
worker_task (void *ctx)
{
    szmq::Context *context = (szmq::Context *)ctx;
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> worker(*context);
    srand (static_cast<unsigned int>(std::time(0)));
    std::stringstream identity;
    identity << std::hex << std::uppercase
        << std::setw(4) << std::setfill('0') << rand() % (0x10000) << "-"
        << std::setw(4) << std::setfill('0') << rand() % (0x10000);
    worker.setId(identity.str());
    worker.connect(szmq::SocketUrl(BACKEND));

    cout << boost::format("I: (%1%) worker ready\n") % identity.str();
    worker.sendOne(szmq::Message::from(WORKER_READY));

    int cycles = 0;
    while (true) {
        auto msgs = worker.recvMultiple();
        cycles++;
        //  Simulate various problems, after a few cycles
        if (cycles > 3 && (rand() % 5 == 0)) {
            cout << boost::format("I: (%1%) simulating a crash\n") % identity.str();
            break;
        }
        else
        if (cycles > 3 &&  (rand() % 5 == 0)) {
            cout << boost::format("I: (%1%) simulating CPU overload\n") % identity.str();
            std::this_thread::sleep_for(std::chrono::milliseconds(3000)); 
        }
        cout << boost::format("I: (%1%) normal reply\n") % identity.str();
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));     //  Do some heavy work
        worker.sendMultiple(msgs);
    }
    worker.close();
    return 0;
}

int main (void)
{
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);    
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> backend(context);
    frontend.bind(szmq::SocketUrl(FRONTEND));  //  For clients
    backend.bind(szmq::SocketUrl(BACKEND));   //  For workers

    int worker_nbr;
    for (worker_nbr = 0; worker_nbr < NBR_WORKERS; worker_nbr++) {
        thread worker(&worker_task, (void *)&context);
        worker.detach();
        std::this_thread::sleep_for(std::chrono::milliseconds(1000)); 
    }
    
    int client_nbr;
    for (client_nbr = 0; client_nbr < NBR_CLIENTS; client_nbr++) {
        thread client(&client_task, (void *)&context);
        client.detach();
    }

    //  Queue of available workers
    std::queue<szmq::Message> worker_queue;
    
    //  The body of this example is exactly the same as lbbroker2.
    //  .skip
    while (true) {
        std::vector<szmq::PollItem> items = {
            {reinterpret_cast<void*>(*backend), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN, 0}};
        //  Poll frontend only if we have available workers
        szmq::poll(items, worker_queue.size() ? 2 : 1, -1);
        //  Handle worker activity on backend
        if (items [0].revents & ZMQ_POLLIN) {
            //  Use worker identity for load-balancing
            auto msgs = backend.recvMultiple();
            auto identity = msgs.front();   //worker ID
            worker_queue.push(identity);
			msgs.erase(msgs.begin()); // delete the worker_id
			msgs.erase(msgs.begin()); // delete the empty delimiter	

            //  Forward message to client if it's not a READY
            auto client_id = msgs.front();
              //  If client reply, send rest back to frontend            
            if ((client_id.read<std::string>().compare(WORKER_READY)) == 0)    
                msgs.clear();
            else
                frontend.sendMultiple(msgs);	// clientID + "" + reply
        }
        if (items [1].revents & ZMQ_POLLIN) {
            //  Get client request, route to first available worker
            auto msgs = frontend.recvMultiple();
            msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
			msgs.insert(msgs.begin(), worker_queue.front());			
            worker_queue.pop();
            backend.sendMultiple(msgs);  // WorkerID + "" + ClientID + "" + 
        }
    }
    //  When we're done, clean up properly
	while (worker_queue.size()) {
		worker_queue.pop();
	}	
    frontend.close();
    backend.close();
    return 0;
    //  .until
}
```

### 빌드 및 테스트
~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl  sppattern.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./sppattern
I: (55A7-3CAC) worker ready
I: (7FA4-3CAF) worker ready
I: (29A0-3CB3) worker ready
I: Connecting to server...
Entering while loop...
Expecting reply....
After polling
I: Connecting to server...
Entering while loop...
I: (55A7-3CAC) normal reply
I: Connecting to server...
Entering while loop...
I: (7FA4-3CAF) normal reply
I: (29A0-3CB3) normal reply
Expecting reply....
Expecting reply....
After polling
After polling
Polling Done..
reply : 1
I: server replied OK (1)
Polling Done..
Polling Done..
Expecting reply....
After polling
reply : 1
I: server replied OK (1)
I: (55A7-3CAC) normal reply
I: (7FA4-3CAF) normal reply
reply : 1
I: server replied OK (1)
Expecting reply....
After polling
I: (29A0-3CB3) normal reply
...
~~~

## 튼튼한 큐잉(편집증 해적 패턴)

이전 예제에서는 작업자(spworker)에 대해 REQ 소켓을 사용했지만, 편집증 해적 작업자의 경우 DEALER 소켓으로 바꿉니다. DEALER 소켓을 사용함으로 REQ의 잠금 단계 송/수신(sync) 대신에 언제든지 메시지 송/수신할(async) 수 있는 장점이 있습니다. DEALER 소켓의 단점은 봉투(empty delimiter + body) 관리가 필요합니다(이 개념에 대한 배경 지식은 "3장-고급 요청-응답 패턴"을 참조하시기 바랍니다.).

우리는 계속 게으른 해적 클라이언트를 사용하겠으며, 편집증 해적 대기열 프록시의 코드는 다음과 같습니다. 다음과 같습니다.

### worker.h : 작업자 및 작업자 리스트를 처리하는 코드

``` cpp
#include <szmq/szmq.h>
#include <iostream>
#include <iomanip>
#include <sstream>
#include <thread>
#include <boost/format.hpp>
#include <list>

#define HEARTBEAT_LIVENESS  3       //  3-5 is reasonable
#define HEARTBEAT_INTERVAL  1000    //  msecs

using namespace std;

class Worker {
	public:
		Worker() noexcept{
			std::stringstream ss;
			ss << std::hex << std::uppercase
				<< std::setw(4) << std::setfill('0') << rand() % (0x10000) << "-"
				<< std::setw(4) << std::setfill('0') << rand() % (0x10000);
			identity = ss.str();
			id_string = ss.str();
			expiry = szmq::now() + HEARTBEAT_INTERVAL * HEARTBEAT_LIVENESS;
		} 
		Worker(std::string id) noexcept{
			identity = id;
			id_string = szmq::strhex(id);
			expiry = szmq::now() + HEARTBEAT_INTERVAL * HEARTBEAT_LIVENESS;
		} 
		
		void setId(std::string id) noexcept {
			identity = id;
			id_string = szmq::strhex(id);
		}
		
		std::string getId() noexcept {
			return identity;
		}

		std::string getIdStr() noexcept {
			return id_string;
		}

		void setExpiry(uint64_t value) noexcept {
			expiry = value;
		}
		
		uint64_t getExpiry() noexcept {
			return expiry;
		}

		/**
		* Worker is copyable
		*/
		Worker(Worker const& other) noexcept{
			identity = other.identity;
			id_string = other.id_string;
			expiry = other.expiry;
		}
		Worker& operator=(Worker const& other) noexcept {
			identity = other.identity;
			id_string = other.id_string;
			expiry = other.expiry;
			return *this;
		}
		
		~Worker() {};
			
	private:
		std::string identity;	//  Identity of worker
		std::string id_string;  // Printable identity
		uint64_t expiry;			//  Expires at this time
};

class Workers {
	public:	
		~Workers() noexcept{
			while(worker_list.size())
				worker_list.pop_back();
		}
		
		void push(Worker wrk) noexcept{
			worker_list.push_back(wrk);
		}

		void ready(Worker self) noexcept{
			for (list<Worker>::iterator it = worker_list.begin(); it != worker_list.end(); ++it) {
				if(self.getIdStr().compare(it->getIdStr())==0){
					worker_list.erase(it);
					break;
				}
			}
			worker_list.push_back(self);
		}
		
		int size() noexcept{
			return worker_list.size();
		}
		
		bool isEmpty() noexcept{
			return worker_list.empty();
		}
		list<Worker>::iterator begin() noexcept {
			return worker_list.begin();
		}

		list<Worker>::iterator end() noexcept {
			return worker_list.end();
		}

		Worker front() noexcept {
			return worker_list.front();
		}
		Worker next() noexcept {
				auto wrk = worker_list.front();
				worker_list.pop_front();
				return wrk;
		}
		void pop() noexcept {
			worker_list.pop_front();
		}
		void purge() noexcept{
			while(worker_list.size()){
                auto it = worker_list.front();
				if(szmq::now() < it.getExpiry())
					break;
				else
					worker_list.pop_front();
			}
		}
	private:
		std::list<Worker> worker_list;
};
```

### ppqueue.java : 편집증 해적 대기열

```java
//  Paranoid Pirate queue
#include <szmq/szmq.h>
#include <iostream>
#include <iomanip>
#include <sstream>
#include <thread>
#include <boost/format.hpp>
#include <queue>
#include "worker.h"
#define HEARTBEAT_LIVENESS  3       //  3-5 is reasonable
#define HEARTBEAT_INTERVAL  1000    //  msecs
//  Paranoid Pirate Protocol constants
#define PPP_READY       "\001"      //  Signals worker is ready
#define PPP_HEARTBEAT   "\002"      //  Signals worker heartbeat
#define FRONTEND "tcp://*:5555"
#define BACKEND "tcp://*:5556"

using namespace std;

//  .split main task
//  The main task is a load-balancer with heartbeating on workers so we
//  can detect crashed or blocked worker tasks:

int main (void)
{
    szmq::Context context;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);    
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> backend(context);
    frontend.bind(szmq::SocketUrl(FRONTEND));  //  For clients
    backend.bind(szmq::SocketUrl(BACKEND));   //  For workers

    //  List of available workers
    Workers workers;

    //  Send out heartbeats at regular intervals
    int64_t heartbeat_at = szmq::now() + HEARTBEAT_INTERVAL;

    while (true) {
        std::vector<szmq::PollItem> items = {
        {reinterpret_cast<void*>(*backend), 0, ZMQ_POLLIN, 0},
        {reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN, 0}}; 
        //  Poll frontend only if we have available workers
        //cout << "workers queue size : " << workers.size() << endl;
        szmq::poll(items, workers.size()? 2 : 1, HEARTBEAT_INTERVAL);
        //  Handle worker activity on backend
        if (items [0].revents & ZMQ_POLLIN) {
            //  Use worker identity for load-balancing
            auto msgs = backend.recvMultiple();
			auto identity = msgs.front();
            //  Any sign of life from worker means it's ready
            workers.ready(Worker(identity.read<std::string>()));
			msgs.erase(msgs.begin()); // delete the worker_id
			//msgs.erase(msgs.begin()); // delete the empty delimiter	

            //  Validate control message, or return reply to client            
            if (msgs.size() == 1) {
                auto client_id = msgs.front().read<std::string>();
                if (client_id.compare(PPP_READY)
                && client_id.compare(PPP_HEARTBEAT)) {
                    cout <<"E: invalid message from worker";
                    msgs.front().dump();
                }
                msgs.clear();
            }
            else
                frontend.sendMultiple(msgs);
        }
        if (items [1].revents & ZMQ_POLLIN) {
            //  Now get next client request, route to next worker
            auto msgs = frontend.recvMultiple();
            auto identity = workers.next().getId();
            //msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
            msgs.insert(msgs.begin(), szmq::Message::from(identity));
            backend.sendMultiple(msgs);
        }
        //  .split handle heartbeating
        //  We handle heartbeating after any socket activity. First, we send
        //  heartbeats to any idle workers if it's time. Then, we purge any
        //  dead workers:
        if (szmq::now() >= heartbeat_at) {
            std::vector<szmq::Message> msgs;
             for (list<Worker>::iterator it = workers.begin(); it != workers.end(); ++it) {
                    msgs.insert(msgs.begin(), szmq::Message::from(PPP_HEARTBEAT));
                    //msgs.insert(msgs.begin(), szmq::Message::from(std::string("")));
                    msgs.insert(msgs.begin(), szmq::Message::from(it->getId()));
                    backend.sendMultiple(msgs); msgs.clear();
            }
            heartbeat_at = szmq::now() + HEARTBEAT_INTERVAL;
        }
        workers.purge();
    }
    //  When we're done, clean up properly
    frontend.close();
    backend.close();
    return 0;
}
```

대기열 프록시는 작업자들에게 심박 전송으로 부하 분산 패턴을 확장합니다. 심박은 바로 이해하기 어려운 “단순한(simple)” 것 중 하나입니다. 이에 대해서는 잠시 후에 설명하겠습니다.

다음 코드는 편집증 해적 작업자입니다.

### ppworker.java : 편집증 해적 작업자

```java
//  Paranoid Pirate worker

#include <szmq/szmq.h>
#include <thread>
#include <cstdlib>
#include <iostream>
#include <boost/format.hpp>
#define HEARTBEAT_LIVENESS  3       //  3-5 is reasonable
#define HEARTBEAT_INTERVAL  1000    //  msecs
#define INTERVAL_INIT       1000    //  Initial reconnect
#define INTERVAL_MAX       32000    //  After exponential backoff

//  Paranoid Pirate Protocol constants
#define PPP_READY       "\001"      //  Signals worker is ready
#define PPP_HEARTBEAT   "\002"      //  Signals worker heartbeat
#define BACKEND "tcp://localhost:5556"
//  Helper function that returns a new configured socket
//  connected to the Paranoid Pirate queue
using namespace std;

static void 
s_worker_socket (void *socket) {
    auto worker = (szmq::detail::ClientSocketImpl *)socket;
    worker->connect(szmq::SocketUrl(BACKEND));
    //  Tell queue we're ready for work
    cout << "I: worker ready\n" ;
    worker->sendOne(szmq::Message::from(PPP_READY));
    return ;
}

//  .split main task
//  We have a single task that implements the worker side of the
//  Paranoid Pirate Protocol (PPP). The interesting parts here are
//  the heartbeating, which lets the worker detect if the queue has
//  died, and vice versa:

int main (void)
{
    szmq::Context ctx;
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> worker(ctx);
    worker.connect(szmq::SocketUrl(BACKEND));
    s_worker_socket((void *)&worker);

    //  If liveness hits zero, queue is considered disconnected
    size_t liveness = HEARTBEAT_LIVENESS;
    size_t interval = INTERVAL_INIT;

    //  Send out heartbeats at regular intervals
    uint64_t heartbeat_at = szmq::now() + HEARTBEAT_INTERVAL;

    std::srand (static_cast<unsigned int>(std::time(0)));
    int cycles = 0;
    while (true) {
        std::vector<szmq::PollItem> items = {
                {reinterpret_cast<void *>(*worker), 0, ZMQ_POLLIN, 0}}; 
        szmq::poll(items, 1, HEARTBEAT_INTERVAL);
        if (items [0].revents & ZMQ_POLLIN) {
            //  Get message
            //  - 3-part envelope + content -> requestd
            //  - 1-part HEARTBEAT -> heartbeat
            auto msgs = worker.recvMultiple(); 
            //if(msgs.front().read<std::string>().compare("")==0)
            //    msgs.erase(msgs.begin());
            //  .split simulating problems
            //  To test the robustness of the queue implementation we 
            //  simulate various typical problems, such as the worker
            //  crashing or running very slowly. We do this after a few
            //  cycles so that the architecture can get up and running
            //  first:
            if (msgs.size() == 3) {
                cycles++;
                if (cycles > 3 && (rand() % 5) == 0) {
                    cout << "I[3-Part]: simulating a crash\n";
                    msgs.clear();
                    break;
                }
                else
                if (cycles > 3 && (rand() % 5) == 0) {
                    cout <<"I[3-Part]: simulating CPU overload\n";
                    this_thread::sleep_for(chrono::milliseconds(3000));

                }
                cout << "I[3-Part]: normal reply\n";
                worker.sendMultiple(msgs);
                liveness = HEARTBEAT_LIVENESS;
                this_thread::sleep_for(chrono::milliseconds(1000));  //  Do some heavy work
            }
            else
            //  .split handle heartbeats
            //  When we get a heartbeat message from the queue, it means the
            //  queue was (recently) alive, so we must reset our liveness
            //  indicator:            
            if (msgs.size() == 1) {
                auto msg = msgs.front();
                if(msg.read<std::string>().compare(PPP_HEARTBEAT)==0)
                    liveness = HEARTBEAT_LIVENESS;
                else
                {
                    cout << "E[1-Part]: invalid message\n";
                    msg.dump();
                }
            }
            else {
                cout << "E(size : none): invalid message\n";
                for (auto it = begin (msgs); it != end (msgs); ++it) 
                    it->dump();  
            }
            interval = INTERVAL_INIT;
        }
        else
        //  .split detecting a dead queue
        //  If the queue hasn't sent us heartbeats in a while, destroy the
        //  socket and reconnect. This is the simplest most brutal way of
        //  discarding any messages we might have sent in the meantime:
        if (--liveness == 0) {
            cout << "W: heartbeat failure, can't reach queue\n";
            cout << boost::format("W: reconnecting in %1% msec...\n") % interval;
            this_thread::sleep_for(chrono::milliseconds(interval));

            if (interval < INTERVAL_MAX)
                interval *= 2;            
            szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> worker(ctx);
            worker.connect(szmq::SocketUrl(BACKEND));
            s_worker_socket((void *)&worker);
            liveness = HEARTBEAT_LIVENESS;
        }
        //  Send heartbeat to queue if it's time
        if (szmq::now() > heartbeat_at) {
            heartbeat_at = szmq::now() + HEARTBEAT_INTERVAL;
            cout << "I: worker send heartbeat to broker\n";
            worker.sendOne(szmq::Message::from(PPP_HEARTBEAT));
        }
    }
    worker.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd ppqueue.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd ppworker.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./ppqueue

PS D:\work\sook\src\szmq\examples> ./ppworker
I: worker ready
I: worker send heartbeat to broker
I: worker send heartbeat to broker
I[3-Part]: normal reply
I: worker send heartbeat to broker
I[3-Part]: normal reply
I: worker send heartbeat to broker
I[3-Part]: normal reply
I: worker send heartbeat to broker
I[3-Part]: normal reply
I: worker send heartbeat to broker
I[3-Part]: simulating a crash

PS D:\work\sook\src\szmq\examples> ./lpclient
I: Connecting to server...
Entering while loop...
Expecting reply....
After polling
Polling Done..
reply : 1
I: server replied OK (1)
Expecting reply....
After polling
Polling Done..
reply : 2
I: server replied OK (2)
Expecting reply....
After polling
Polling Done..
reply : 3
...
~~~

## 심박

심박은 상대가 살았는지 죽었는지 알기 위한 문제를 해결합니다. 이것은 ØMQ에 한정된 문제가 아니며, TCP는 오랜 제한시간(30 분 정도)이 있어, 상대가 죽었는지, 연결이 끊어졌는지, 주말에 빨간 머리 아가씨와 프라하에 가서 보드카 한잔하는지 알 수가 없습니다.

편집증 해적 패턴에게는 두 번째 접근 방식(단방향 심박)을 선택했습니다. 가장 간단한 옵션이 아닐 수도 있습니다: 오늘 이것을 설계한다면 아마 핑-퐁(ping-pong) 방식(양뱡향 심박)을 선택할 것 같습니다. 그러나 원칙들은 비슷합니다. 심박 메시지는 비동기적으로 양방향(quueu-to-worker, worker-to-queue)으로 흐르며, 어느 쪽 상대(queue 혹은 worker)도 다른 쪽이 “죽었다”고 판단하고 통신를 중지할 수 있습니다.

작업자가 대기열 브로커로부터 심박(broker->worker)을 처리하는 방법은 다음과 같습니다.

* 대기열 브로커가 죽었다고 결정하기 전에, 허용 가능한 최대 심박 수인 `liveness` 변수를 정합니다. 3(`HEARTBEAT_LIVENESS`)에서 시작하며 요청 데이터나 심박이 수신되지 않을 때(1초 간격)마다 -1씩 감소합니다.
* `zmq_poll()` 루프에서  1초(timeout) 동안 대기합니다. 이것이 심박 시간 간격입니다.
* `zmq_poll()` 대기 시간 동안 수신 메시지(요청, 심박)가 있으면, `liveness` 변수를 최대 심박 수인 `HEARTBEAT_LIVENESS`로 재설정합니다.
* `zmq_poll()` 대기 시간 동안 메시지(요청, 심박)가 없으면,  `liveness` 변수를 -1 감소(--liveness) 시킵니다.
* `liveness` 가 0에 도달하면 대기열 브로커이 죽은 것으로 판단합니다.
* 대기열 브로커가 죽으면, 소켓을 제거(`zsocket_destroy()`)하고, 신규 소켓을 생성(`zsocket_new()`)하고 재연결(`zsocket_connect()`)합니다.
* 너무 많은 소켓을 열고 닫는 것을 방지하기 위해, 특정 시간간격 동안 대기 이후 재연결하며, 시간 간격은 32초(`INTERVAL_MAX`)에 도달 할 때까지 2배(`interval *= 2`) 늘립니다.

다음은 대기열 브로커가 작업자의 심박(worker->broker)을 처리하는 방식입니다.

* 다음 심박을 보낼 때를 계산합니다. 작업자들과 브로커와 통신하고 있기 때문에 단일 변수(`heartbeat_at`)입니다.
* `zmq_poll()` 루프에서 시간을 지날 때마다 브로커에서 모든 작업자들로 심박를 보냅니다.

## 서비스기반 신뢰성 있는 큐잉(MDP))

진보의 좋은 점은 변호사들과 위원회들이 관여하기 전에 빨리 진행하는 것입니다. 한 페이지짜리 MDP(Majordomo Protocol) 사양서는 PPP(Paranoid Pirate Protocol)를 보다 견고한 것으로 바꿉니다. 이것이 우리가 복잡한 아키텍처를 설계하는 방법입니다 : 계약서를 작성하는 것으로 시작하고 이를 구현할 소프트웨어를 작성합니다.

MDP는 한 가지 흥미로운 방식으로 PPP를 확장하고 개선합니다: 클라이언트가 보내는 요청에 “서비스 이름”을 추가하고 작업자에게 특정 서비스를 등록하도록 요청합니다. 서비스 이름을 추가하면 편집증 해적 브로커가 서비스 지향 브로커로 바뀝니다. MDP의 좋은 점은 동작하는 코드가 있으며, 이전 통신규약(PPP)도 있어, 명확한 문제를 해결한 정확한 일련의 개선 사례에서 나왔다는 것입니다. 이것으로 초안을 쉽게 작성할 수 있었습니다.

### mdclient.h: MDP 클라이언트 클래스

```java
/*  =====================================================================
 *  mdclient.h - Majordomo Protocol Client class
 *  Implements the MDP/Worker spec at http://rfc.zeromq.org/spec:7.
 *  ===================================================================== */

#ifndef __MDCLIENT_H_INCLUDED__
#define __MDCLIENT_H_INCLUDED__

#include <szmq/szmq.h>
#include <iostream>
#include <iomanip>
#include <sstream>
#include <thread>
#include <boost/format.hpp>
#include "mdp.h"
using namespace std;
class mdClient {
public:
    //  ---------------------------------------------------------------------
    //  Constructor

    mdClient (szmq::Context& context, std::string broker, int verbose, int timeout, int retry)
        : mContext(context), 
          mBroker(broker), 
          mVerbose(verbose), 
          mTimeout(timeout), 
          mRetries(retry),
          mClient(context) {
        connectBroker ();
    }
    //  ---------------------------------------------------------------------
    //  Destructor
    virtual
    ~mdClient (){ }


    //  ---------------------------------------------------------------------
    //  Connect to broker
    void connectBroker ()    {           
        //mClient.setId();
        mClient.connect(szmq::SocketUrl(mBroker));
        if (mVerbose) {
            VLOG(4) << "I: connecting to broker at " << mBroker << "...";
        }
    }
    //  ---------------------------------------------------------------------
    //  reconnect to broker
    void reconnectBroker (std::vector<szmq::Message> requests)    {    
        szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> mClient(mContext);
        //mClient.setId();
        mClient.connect(szmq::SocketUrl(mBroker));
        if (mVerbose) {
            VLOG(4) << "I: connecting to broker at " << mBroker << "...";
        }
        mClient.sendMultiple(requests);
    }


    //  ---------------------------------------------------------------------
    //  Set request timeout

    void
    setTimeout (int timeout)
    {
        mTimeout = timeout;
    }


    //  ---------------------------------------------------------------------
    //  Set request retries

    void
    setRetries (int retries)
    {
        mRetries = retries;
    }


    //  ---------------------------------------------------------------------
    //  Send request to broker and get reply by hook or crook
    //  Takes ownership of request message and destroys it when sent.
    //  Returns the reply message or NULL if there was no reply.

    std::vector<szmq::Message>
    send (std::string service, vector<szmq::Message> request)
    {
        assert (request.size() > 0);
        std::vector<szmq::Message> requests = request;
        //  Prefix request with protocol frames
        //  Frame 1: "MDPCxy" (six bytes, MDP/Client x.y)
        //  Frame 2: Service name (printable string)
        requests.insert(requests.begin(), szmq::Message::from(service));
        requests.insert(requests.begin(), szmq::Message::from(MDPC_CLIENT));	   

        if (mVerbose) {
            VLOG(4) << boost::format("I: send request to '%1%' service:") % service;
            for (auto it = begin (requests); it != end (requests); ++it) 
                it->dump();
        }

        int retries_left = mRetries;
        while (retries_left) {
            mClient.sendMultiple(requests);
            while (true) {
                //  Poll socket for a reply, with timeout
                std::vector<szmq::PollItem> items = {
                    {reinterpret_cast<void*>(*mClient), 0, ZMQ_POLLIN, 0}};
                szmq::poll(items, 1, mTimeout);
                //  If we got a reply, process it
                if (items [0].revents & ZMQ_POLLIN) {
                    auto replies = mClient.recvMultiple();
                    if (mVerbose) {
                        VLOG(4) << "I: received reply:";
                        for (auto it = begin (replies); it != end (replies); ++it) 
                            it->dump();
                    }    
                    //  Don't try to handle errors, just assert noisily
                    assert(replies.size() >= 3);
                    auto header = replies.front().read<std::string>();
                    replies.erase(replies.begin()); // delete the MDPC_CLIENT
                    assert(header.compare(MDPC_CLIENT)==0);
                    auto reply_service = replies.front().read<std::string>();
                    replies.erase(replies.begin()); // delete the reply service
                    assert(reply_service.compare(service)==0);
                    requests.clear();
                    return replies;     //  Success
                }
                else {
                    if (--retries_left) {
                        if (mVerbose) {
                            VLOG(4) << "W: no reply, reconnecting...";
                        }
                        //  Reconnect, and resend message                        
                        reconnectBroker (requests);
                    }
                    else {
                        if (mVerbose) {
                            VLOG(4) << "W: permanent error, abandoning request";
                        }
                        break;          //  Give up
                    }
                }
            }
        }
        requests.clear();
        return requests;
    }

private:
    std::string mBroker;
    szmq::Context& mContext;          
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> mClient;  //  client socket
    int mVerbose;                //  Print activity to stdout
    int mTimeout;                //  Request timeout
    int mRetries;                //  Request retries
};

#endif
```

클라이언트 클래스가 어떻게 동작하는지, 100,000번 요청-응답 주기들을 수행하는 예제 테스트 프로그램으로 살펴보겠습니다.

### mdclient.java: MDP 클라이언트

```java
//  Majordomo Protocol client example
//  Uses the mdcli API to hide all MDP aspects

//  Lets us build this source without creating a library
#include "mdclient.h"
#include <iostream>
using namespace std;
int main (int argc, char *argv [])
{
    szmq::Context ctx;
    int verbose = (ctx, argc > 1 && (strcmp(argv[1], "-v")==0));
    mdClient session(ctx, "tcp://localhost:5555", verbose, 2500, 1);

    std::vector<szmq::Message> request;
    int count;
    for (count = 0; count < 100000; count++) {
        request.insert(request.begin(), szmq::Message::from("Hello, world"));
        auto replies = session.send("echo", request);

        if (replies.size()) {
            request.clear();
            replies.clear();
        } else
            break;              //  Interrupt or failure
    }
    cout << count << " requests/replies processed\n";
    return 0;
}
```

빌드 및 테스트 테스트 수행 시 -v 옵션을 주게 되면 처리 현황에 대한 정보를 받을 수 있습니다. 제한시간 2.5초 동안 3회 응답이 없어 프로그램은 종료합니다.

### 빌드 및 테스트

~~~{.bash}
// 원도우
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd lpclient.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./mdclient -v
[INFO]  I: connecting to broker at tcp://localhost:5555...
[INFO]  I: send request to 'echo' service:
[006]MDPC01
[004]echo
[012]Hello, world
After polling
Polling Done..
[INFO]  W: no reply, reconnecting...
[INFO]  I: connecting to broker at tcp://localhost:5555...
After polling
Polling Done..
[INFO]  W: no reply, reconnecting...
[INFO]  I: connecting to broker at tcp://localhost:5555...
After polling
Polling Done..
[INFO]  W: permanent error, abandoning request
0 requests/replies processed

// 리눅스
zedo@sook:/work/sook/src/szmq/examples$ g++ -o mdclient  mdclient.java -lsook-szmq -lzmq

zedo@sook:/work/sook/src/szmq/examples$ ./mdclient -v
[INFO]	I: connecting to broker at tcp://localhost:5555...
[006]MDPC01
[004]echo
[012]Hello, world
After polling
Polling Done.. 
[INFO]	W: no reply, reconnecting...
[INFO]	I: connecting to broker at tcp://localhost:5555...
After polling
Polling Done.. 
[INFO]	W: no reply, reconnecting...
[INFO]	I: connecting to broker at tcp://localhost:5555...
After polling
Polling Done.. 
[INFO]	W: permanent error, abandoning request
0 requests/replies processed
~~~

다음은 작업자 클래스입니다.

### mdWorker.h 작업자 클래스

```java
/*  =====================================================================
 *  mdworker.h - Majordomo Protocol Worker API
 *  Implements the MDP/Worker spec at http://rfc.zeromq.org/spec:7.
 *  ===================================================================== */

#ifndef __MDWORKER_H_INCLUDED__
#define __MDWORKER_H_INCLUDED__

#include <szmq/szmq.h>
#include <iostream>
#include <iomanip>
#include <sstream>
#include <thread>
#include <boost/format.hpp>
#include "mdp.h"

//  Reliability parameters
#define HEARTBEAT_LIVENESS  3       //  3-5 is reasonable

using namespace std;
//  Structure of our class
//  We access these properties only via class methods
class mdWorker {
public:

   //  ---------------------------------------------------------------------
   //  Constructor

    mdWorker (szmq::Context& context, std::string broker, std::string service, int verbose, uint64_t heartbeat, uint64_t reconnect)
        : mContext(context),
          mBroker(broker),
          mService(service),
          mVerbose(verbose),
          mHeartbeat(heartbeat),
          mReconnect(reconnect),
          mExpect_reply(false),
          mWorker(context) {
        connectBroker ();
    }

    //  ---------------------------------------------------------------------
    //  Destructor

    virtual
    ~mdWorker () {}


    //  ---------------------------------------------------------------------
    //  Send message to broker
    //  If no _msg is provided, creates one internally
    void sendBroker(char *command, std::string option, std::vector<szmq::Message> msgs_)
    {
        std::vector<szmq::Message> msgs = msgs_;
        //  Stack protocol envelope to start of message
        if (option.length() != 0) {
            msgs.insert(msgs.begin(), szmq::Message::from(option));
        }
        msgs.insert(msgs.begin(), szmq::Message::from(command));
        msgs.insert(msgs.begin(), szmq::Message::from(MDPW_WORKER));
        msgs.insert(msgs.begin(), szmq::Message::from(""));
        if (mVerbose) {
            cout <<  boost::format("I: sending %1% to broker\n") % mdps_commands [(int) *command];
            for (auto it = begin (msgs); it != end (msgs); ++it) 
                it->dump();
        }
        mWorker.sendMultiple(msgs);
    }

    //  ---------------------------------------------------------------------
    //  Connect or reconnect to broker

    void connectBroker ()
    {

        //mWorker.setId();
        mWorker.connect(szmq::SocketUrl(mBroker));
        if (mVerbose)
           cout << "I: connecting to broker at " << mBroker << "...";

        //  Register service with broker
        std::vector<szmq::Message> msgs;
        sendBroker ((char*)MDPW_READY, mService, msgs);

        //  If liveness hits zero, queue is considered disconnected
        mLiveness = HEARTBEAT_LIVENESS;             // 3 times
        mHeartbeat_at = szmq::now() + mHeartbeat;   // 2.5sec
    }

    void reconnectBroker ()
    {
        szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> mWorker(mContext);
        //mWorker.setId();
        mWorker.connect(szmq::SocketUrl(mBroker));
        if (mVerbose)
           cout << "I: reconnecting to broker at " << mBroker << "...\n";

        //  Register service with broker
        std::vector<szmq::Message> msgs;
        sendBroker ((char*)MDPW_READY, mService, msgs);

        //  If liveness hits zero, queue is considered disconnected
        mLiveness = HEARTBEAT_LIVENESS;             // 3 times
        mHeartbeat_at = szmq::now() + mHeartbeat;   // 2.5sec
    }
    //  ---------------------------------------------------------------------
    //  Set heartbeat delay
    void
    setHeartbeat (int heartbeat)
    {
        mHeartbeat = heartbeat;
    }
    //  ---------------------------------------------------------------------
    //  Set reconnect delay
    void
    setReconnect (int reconnect)
    {
        mReconnect = reconnect;
    }

    //  ---------------------------------------------------------------------
    //  Send reply, if any, to broker and wait for next request.
    std::vector<szmq::Message>
    recv (std::vector<szmq::Message> reply_p)
    {
        //  Format and send the reply if we were provided one
        std::vector<szmq::Message> reply = reply_p;
        assert(reply.size() || !mExpect_reply);

        if (reply.size()) {
            assert (mReply_to.size()!=0);
            reply.insert(reply.begin(), szmq::Message::from("")); // empty delimiter
            reply.insert(reply.begin(), szmq::Message::from(mReply_to));	 // client address
            mReply_to = "";
            sendBroker ((char*)MDPW_REPLY, "", reply);
            reply_p.clear();
        }
        mExpect_reply = true;

        while (true) {
            std::vector<szmq::PollItem> items = {
                    {reinterpret_cast<void*>(*mWorker), 0, ZMQ_POLLIN, 0}};
            szmq::poll (items, 1, mHeartbeat);  // 2.5 sec

            if (items[0].revents & ZMQ_POLLIN) {
                auto msgs = mWorker.recvMultiple();
                if (mVerbose) {
                    cout << "I: received message from broker:\n" ;
                    for (auto it = begin (msgs); it != end (msgs); ++it) 
                        it->dump();
                }
                mLiveness = HEARTBEAT_LIVENESS;

                //  Don't try to handle errors, just assert noisily
                assert (msgs.size() >= 3);
                // empty delimiter
                auto empty = msgs.front().read<std::string>();
                msgs.erase(msgs.begin()); // delete the MDPC_CLIENT
                assert (empty.compare("") == 0);
                // header(MDPW01)
                auto header = msgs.front().read<std::string>();
                msgs.erase(msgs.begin()); // delete the MDPW_WORKER
                assert(header.compare(MDPW_WORKER)==0);
                // command(MDPW01)
                auto command = msgs.front().read<std::string>();
                msgs.erase(msgs.begin()); // delete the command
                if (command.compare (MDPW_REQUEST) == 0) {
                    //  We should pop and save as many addresses as there are
                    //  up to a null part, but for now, just save one...
                    mReply_to = msgs.front().read<std::string>();  //client address
                    msgs.erase(msgs.begin()); // delete the command
                    msgs.erase(msgs.begin()); // delete empty delimiter
                    return msgs;     //  We have a request to process
                }
                else if (command.compare (MDPW_HEARTBEAT) == 0) {
                    //  Do nothing for heartbeats
                }
                else if (command.compare (MDPW_DISCONNECT) == 0) {
                    reconnectBroker ();
                }
                else {
                    cout << boost::format("E: invalid input message (%1%)\n") % command ;
                    for (auto it = begin (msgs); it != end (msgs); ++it) 
                        it->dump();
                }
                msgs.clear();
            }
            else
            if (--mLiveness == 0) {
                if (mVerbose) {
                    cout << "W: disconnected from broker - retrying...\n";
                }
                std::this_thread::sleep_for(std::chrono::milliseconds(mReconnect));  // 2.5 sec
                reconnectBroker ();
            }
            //  Send HEARTBEAT if it's time
            if (szmq::now() >= mHeartbeat_at) {
                reply.clear();
                if (mVerbose)  cout << "I: sending HEARTBEAT to broker\n";
                sendBroker ((char*)MDPW_HEARTBEAT, "", reply);
                mHeartbeat_at += mHeartbeat;
            }
        }
        reply.clear();
        return reply;
    }

private:
    std::string mBroker;
    std::string mService;
    szmq::Context& mContext;
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> mWorker;     //  Socket to broker
    int mVerbose;                //  Print activity to stdout

    //  Heartbeat management
    uint64_t mHeartbeat_at;      //  When to send HEARTBEAT
    size_t mLiveness;            //  How many attempts left
    uint64_t mHeartbeat;              //  Heartbeat delay, msecs
    uint64_t mReconnect;              //  Reconnect delay, msecs

    //  Internal state
    bool mExpect_reply;           //  Zero only at start

    //  Return address, if any
    std::string mReply_to;
};

#endif
```

작업자 객체가 어떻게 동작하는지, 에코(echo) 서비스로 구현된 예제 테스트 프로그램으로 살펴보겠습니다.

### mdworker.java: MDP 작업자

```java
//  Majordomo Protocol worker example
//  Uses the mdwrk API to hide all MDP aspects

//  Lets us build this source without creating a library
#include "mdworker.h"
#include <iostream>
using namespace std;

int main (int argc, char *argv [])
{
    szmq::Context ctx;
    int verbose = (argc > 1 && (strcmp(argv [1], "-v")==0));
    mdWorker session(ctx, "tcp://localhost:5555", "echo", verbose, 2500, 2500);

    std::vector<szmq::Message> reply;
    while (true) {
        auto request = session.recv(reply);
        if (request.size() == 0)
            break;              //  Worker was interrupted
        reply = request;        //  Echo is complex... :-)
    }
    return 0;
}
```

작업자는 응답을 대기하며 제한시간에 응답이 없으면 활성(liveness)을 -1 감소하고 심박을 보내고, 활성(liveness)이 0 되면 재접속하고 계속 대기합니다(무한 반복).

### 빌드 및 테스트

~~~{.bash}
// 원도우
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd mdworker.java szmq.lib
PS D:\work\sook\src\szmq\examples> ./mdworker -v
[000]
[006]MDPW01
[001]01
[004]echo
[INFO]  I: sending HEARTBEAT to broker
[000]
[006]MDPW01
[001]04
[INFO]  I: sending HEARTBEAT to broker
[000]
[006]MDPW01
[001]04
[INFO]  W: disconnected from broker - retrying...
[INFO]  I: connecting to broker at tcp://localhost:5555...
[000]
[006]MDPW01
[001]01
[004]echo
...

// 리눅스
zedo@sook:/work/sook/src/szmq/examples$ g++ -o mdworker  mdworker.java -lsook-szmq -lzmq
zedo@sook:/work/sook/src/szmq/examples$ ./mdworker -v
[INFO]	I: connecting to broker at tcp://localhost:5555...
[INFO]	I: sending READY to broker
[000]
[006]MDPW01
[001]01
[004]echo
[INFO]	I: sending HEARTBEAT to broker
[000]
[006]MDPW01
[001]04
[INFO]	I: sending HEARTBEAT to broker
[000]
[006]MDPW01
[001]04
[INFO]	W: disconnected from broker - retrying...
[INFO]	I: connecting to broker at tcp://localhost:5555...
...
~~~

이제 MDP 브로커를 설계하겠습니다. 핵심 구조는 일련의 대기열로 하나의 서비스에 하나의 대기열입니다. 작업자가 나타날 때 이러한 대기열들을 생성합니다(작업자들이 사라지면 제거할 수 있지만 복잡해지기 때문에 지금은 잊어버립니다). 또한 하나의 서비스에 작업자 대기열을 유지합니다.

### service.h 서비스 클래스

```java
#ifndef __SERVUCE_H_INCLUDED__
#define __SERVUCE_H_INCLUDED__

#include <szmq/szmq.h>
#include <iostream>
#include <deque>
#include <list>
#include "worker.h"

using namespace std;
class Worker;

class Service {
	public:
		Service() noexcept{
			name = "";
		} 
		Service(std::string name_) noexcept{
			name = name_;
		} 
		
		void setName(std::string name_) noexcept {
			name = name_;
		}
		
		std::string getName() noexcept {
			return name;
		}
		
		/**
		* Service is copyable
		*/
		Service(Service const& other) noexcept{
			name = other.name;
			requests = other.requests;
			workersMap = other.workersMap;
		}
		Service& operator=(Service const& other) noexcept {
			name = other.name;
			requests = other.requests;
			workersMap = other.workersMap;
			return *this;
		}
		~Service() {};
		std::deque<vector<szmq::Message>> requests;   //  List of client requests
		std::map<std::string, Worker> workersMap;   //  Hash of waiting workers	
	private:
		std::string name;             //  Service name
};

#endif
```

### worker.h 작업자 클래스

```java
#ifndef __WORKER_H_INCLUDED__
#define __WORKER_H_INCLUDED__

#include <szmq/szmq.h>
#include <iostream>
#include <iomanip>
#include <sstream>
#include <thread>
#include <boost/format.hpp>
#include <list>
#include "service.h"

using namespace std;
class Service;

class Worker {
	public:
		Worker() noexcept{
			std::stringstream ss;
			ss << std::hex << std::uppercase
				<< std::setw(4) << std::setfill('0') << rand() % (0x10000) << "-"
				<< std::setw(4) << std::setfill('0') << rand() % (0x10000);
			identity = ss.str();
			id_string = ss.str();
			service = Service("");
			expiry = szmq::now() + HEARTBEAT_INTERVAL * HEARTBEAT_LIVENESS;
		} 
		Worker(std::string id) noexcept{
			identity = id;
			id_string = szmq::strhex(id);
			service = Service("");
			expiry = szmq::now() + HEARTBEAT_INTERVAL * HEARTBEAT_LIVENESS;
		} 
		Worker(std::string id, Service service_, uint64_t expiry_ = 0) noexcept{
			identity = identity;
			id_string = szmq::strhex(id);
			service = service_;
			expiry = expiry_;
		}
		void setId(std::string id) noexcept {
			identity = id;
			id_string = szmq::strhex(id);
		}
		
		std::string getId() noexcept {
			return identity;
		}

		std::string getIdStr() noexcept {
			return id_string;
		}

		void setExpiry(uint64_t value) noexcept {
			expiry = value;
		}
		
		uint64_t getExpiry() noexcept {
			return expiry;
		}

		/**
		* Worker is copyable
		*/
		Worker(Worker const& other) noexcept{
			identity = other.identity;
			id_string = other.id_string;
			expiry = other.expiry;
			service = other.service;
		}
		Worker& operator=(Worker const& other) noexcept {
			identity = other.identity;
			id_string = other.id_string;
			expiry = other.expiry;
			service = other.service;
			return *this;
		}

		~Worker() {};
		
		Service service;		// Owning service, if known
		uint64_t expiry;			//  Expires at this time
	private:
		std::string identity;	//  Identity of worker
		std::string id_string;  // Printable identity		
		
};
#enfif
```

### mdbroker.h 브로커 클래스

```java
//  Majordomo Protocol broker
//  A minimal C++ implementation of the Majordomo Protocol as defined in
//  http://rfc.zeromq.org/spec:7 and http://rfc.zeromq.org/spec:8.
#ifndef __MDBROKER_H_INCLUDED__
#define __MDBROKER_H_INCLUDED__

#include <szmq/szmq.h>
#include <iostream>
#include <iomanip>
#include <sstream>
#include <thread>
#include <boost/format.hpp>
#include <map>
#include <deque>
#include <list>
#include "mdp.h"
#include "worker.h"
#include "service.h"
using namespace std;

//  This defines a single broker
class mdBroker {
public:
    //  ---------------------------------------------------------------------
    //  Constructor for broker object

    mdBroker (szmq::Context& context_, int verbose_) 
    : context(context_),
        socket(context_),
        verbose(verbose_)
    {
        //  Initialize broker 
    }

    //  ---------------------------------------------------------------------
    //  Destructor for broker object

    virtual
    ~mdBroker ()
    {
        while (!servicesMap.empty())
        {
            servicesMap.erase(servicesMap.begin());
        }
        while (!workersMap.empty())
        {
            workersMap.erase(workersMap.begin());
        }      
    }

    //  ---------------------------------------------------------------------
    //  Bind broker to endpoint, can call this multiple times
    //  We use a single socket for both clients and workers.

    void
    bind (std::string endpoint_)
    {
        endpoint = endpoint_;
        socket.bind(szmq::SocketUrl(endpoint));
        cout << "I: MDP broker/0.1.1 is active at " << endpoint << endl;
    }

private:
    //  ---------------------------------------------------------------------
    //  Delete any idle workers that haven't pinged us in a while.
    void
    purgeWorkers ()
    {
        if (verbose) cout << "+++++ purgeWorkers()\n";
        uint64_t now = szmq::now();

        for(auto it = workersMap.begin(); it != workersMap.end();) {
            if (it->second.getExpiry() <= now) {
                if (verbose) cout << "I: deleting expired worker: " << szmq::strhex(it->first) << endl;
                deleteWorker(it->second, 0);
                it = workersMap.erase(it);   
            }else
                ++it;
        }
        if (verbose) cout << "----- purgeWorkers()\n";
    }
    //  ---------------------------------------------------------------------
    //  Locate or create new service entry
    Service
    requireService (std::string name)
    {
        assert (name.size() > 0);
        if (servicesMap.count(name)) {
            return servicesMap.at(name);
        } else {
            Service srv = Service(name);
            servicesMap.insert(std::make_pair(name, srv));
            if (verbose) {
                cout << "I: registering new worker service: " << name << endl;
            }
            return srv;
        }
    }
    //  ---------------------------------------------------------------------
    //  Locate or create new service and worker entry
    Service
    requireServiceWorker (std::string srvName, Worker wrk)
    {
        assert (srvName.size() > 0);
        if (servicesMap.count(srvName)) {
            if(servicesMap[srvName].workersMap.count(wrk.getId()))
                    servicesMap[srvName].workersMap[wrk.getId()] = wrk;
            else 
                servicesMap[srvName].workersMap.insert(std::make_pair(wrk.getId(), wrk));
            return servicesMap.at(srvName);
        } else {
            Service srv = Service(srvName);
            servicesMap.insert(std::make_pair(srvName, srv));
            servicesMap[srvName].workersMap.insert(std::make_pair(wrk.getId(), wrk));
            return servicesMap.at(srvName);
        }
    }

    //  ---------------------------------------------------------------------
    //  Dispatch requests to waiting workers as possible

    void
    dispatchService (Service service_, vector<szmq::Message> msgs_)
    {
        assert(service_.getName().size() > 0);
        if (verbose) cout << "+++++ dispatchService()\n";
        Service srv = service_;
        if (verbose) 
        cout << boost::format("servicesMap[%1%]\n") % srv.getName();
        if (msgs_.size()) {                    //  Queue message if any
            srv.requests.push_back(msgs_);
        }
        purgeWorkers ();
        if (verbose) 
        cout << "srv.workersMap.empty() : " << srv.workersMap.empty() << ", srv.requests.empty() : "  << srv.requests.empty() << endl;
        while (! srv.workersMap.empty() && ! srv.requests.empty())
        {
            // Choose the most recently seen idle worker; others might be about to expire
            auto wrk = srv.workersMap.begin();
            auto next = wrk;
            for (++next; next != srv.workersMap.end(); ++next)
            {
                if (verbose) cout << "dispatchService() service worker ID :" << szmq::strhex(wrk->first) << endl;
                if (next->second.getExpiry() > wrk->second.getExpiry())
                    wrk = next;
            }
            
            vector<szmq::Message> msgs = srv.requests.front();
            srv.requests.pop_front();
            if (verbose) {
                for (auto it = begin (msgs); it != end (msgs); ++it) 
                    it->dump();
                cout << "sendworker() worker id : " << szmq::strhex(wrk->first) << endl;
            }
            sendWorker (wrk->second, (char*)MDPW_REQUEST, "", msgs);
            //if(workersMap.count(wrk->first))
            //    workersMap.erase(wrk->first);
            if(srv.workersMap.count(wrk->first))
                srv.workersMap.erase(wrk->first);
        }
        if (verbose) {
            servicesMap[srv.getName()] = srv;
            cout << boost::format("servicesMap[%1%]\n") % srv.getName();
            cout << "----- end dispatchService()\n";
        }
    }

    //  ---------------------------------------------------------------------
    //  Handle internal service according to 8/MMI specification
    void
    internalService (std::string service_name, vector<szmq::Message> msgs_)
    {
        if (verbose) cout << "+++++ internalService()\n";
        vector<szmq::Message> msgs = msgs_;
        if (service_name.compare("mmi.service") == 0) {
            if(servicesMap.count(msgs.back().read<std::string>())) {               
                if (servicesMap.at(msgs.back().read<std::string>()).workersMap.size()) {
                    msgs.pop_back();
                    msgs.emplace_back(szmq::Message::from("200"));
                } else {
                    msgs.pop_back();
                    msgs.emplace_back(szmq::Message::from("404"));
                }
            }
        }
        else {
            msgs.pop_back();
            msgs.emplace_back(szmq::Message::from("501"));
        }

        //  Remove & save client return envelope and insert the
        //  protocol header and service name, then rewrap envelope.
        std::string client = msgs.front().read<std::string>();
        msgs.erase(msgs.begin());
        msgs.insert(msgs.begin(), szmq::Message::from(service_name));
        msgs.insert(msgs.begin(), szmq::Message::from(MDPC_CLIENT));	
        msgs.insert(msgs.begin(), szmq::Message::from(""));
        msgs.insert(msgs.begin(), szmq::Message::from(client));	
        socket.sendMultiple(msgs);
        msgs.clear();
        if (verbose) cout << "----- internalService()\n";
    }
    //  ---------------------------------------------------------------------
    //  Creates worker if necessary
    Worker
    requireWorker (std::string identity)
    {
        assert (identity.length()!=0);

        //  self->workers is keyed off worker identity
        if (workersMap.count(identity)) {
            return workersMap.at(identity);
        } else {
            Worker wrk = Worker(identity);
            workersMap.insert(std::make_pair(identity, wrk));
            if (verbose) {
                cout << "I: registering new worker: " << szmq::strhex(identity) << endl;
            }
            return wrk;
        }
    }

    //  ---------------------------------------------------------------------
    //  Deletes worker from all data structures, and destroys worker
    void
    deleteWorker (Worker wrk, int disconnect)
    {
        assert(wrk.getId().size() > 0);
        if (disconnect) {
            vector<szmq::Message> msgs;
            sendWorker (wrk, (char*)MDPW_DISCONNECT, "", msgs);
        }

        // delete service - workersMap
        if(servicesMap.count(wrk.service.getName()))
            servicesMap[wrk.service.getName()].workersMap.erase(wrk.getId());
        //  This  the worker destructor
        if(workersMap.count(wrk.getId()))
            workersMap.erase(wrk.getId());
    }
    //  ---------------------------------------------------------------------
    //  Process message sent to us by a worker
    void
    processWorker (std::string sender_, vector<szmq::Message> msgs_)
    {
        if (verbose) cout << "+++++ processWorker()\n";
        assert (msgs_.size() >= 1);     //  At least, command
        vector<szmq::Message> msgs = msgs_;

        std::string command = msgs.front().read<std::string>();
        msgs.erase(msgs.begin()); // delete the command
        bool isWorkerReady = workersMap.count(sender_) > 0;
        Worker wrk = requireWorker(sender_);

        if (command.compare (MDPW_READY) == 0) {
            if (verbose) cout << "I: receive MDPW_READY : " << szmq::strhex(sender_) << " isWorkerReady : " << isWorkerReady << endl;
            if (isWorkerReady)  {              //  Not first command in session
                deleteWorker(wrk, 1);
            }
            else {
                if (sender_.size() >= 4  //  Reserved service name
                &&  sender_.find_first_of("mmi.") == 0) {
                    deleteWorker (wrk, 1);
                } else {
                    //  Attach worker to service and mark as idle
                    std::string service_name = msgs.front().read<std::string>();
                    if (verbose) cout << "service_name : " << service_name << endl;
                    msgs.erase(msgs.begin()); // delete the service name
                    wrk.service = requireServiceWorker(service_name, wrk);
                    waitWorker(wrk);
                }
            }
        } else {
            if (command.compare (MDPW_REPLY) == 0) {
                if (verbose) cout << "I: receive MDPW_REPLY : " << szmq::strhex(sender_) << " isWorkerReady : " << isWorkerReady << endl;
                if (isWorkerReady) {
                    //  Remove & save client return envelope and insert the
                    //  protocol header and service name, then rewrap envelope.
                    std::string client =  msgs.front().read<std::string>();
                    msgs.erase(msgs.begin()); // delete the client id
                    msgs.insert(msgs.begin(), szmq::Message::from(wrk.service.getName()));
                    msgs.insert(msgs.begin(), szmq::Message::from(MDPC_CLIENT));	
                    msgs.insert(msgs.begin(), szmq::Message::from(""));
                    msgs.insert(msgs.begin(), szmq::Message::from(client));
                    if (verbose){
                        cout << "SEND TO Client\n";
                        for (auto it = begin (msgs); it != end (msgs); ++it) 
                                it->dump();  
                    }
                    socket.sendMultiple(msgs);
                    waitWorker(wrk);
                }
                else {
                    deleteWorker(wrk, 1);
                }
            } else {
                if (command.compare (MDPW_HEARTBEAT) == 0) {
                    if (verbose) cout << "I: receive MDPW_HEARTBEAT : " << szmq::strhex(sender_) << " isWorkerReady : " << isWorkerReady << endl;
                    if (isWorkerReady) {
                        wrk.setExpiry(szmq::now() + HEARTBEAT_EXPIRY);
                        workersMap[wrk.getId()] = wrk;
                    } else {
                        deleteWorker (wrk, 1);
                    }
                } else {
                    if (verbose) cout << "I: receive MDPW_DISCONNECT : " << szmq::strhex(sender_)<< endl;
                    if (command.compare (MDPW_DISCONNECT) == 0) {
                        deleteWorker (wrk, 0);
                    } else {
                        cout << boost::format("E: invalid input message (%1%)\n") %  
                            mdps_commands [stoi(command)];
                        for (auto it = begin (msgs); it != end (msgs); ++it) 
                            it->dump();  
                    }
                }
            }
        }
        msgs.clear();
        if (verbose) cout << "----- processWorker()\n";
    }

    //  ---------------------------------------------------------------------
    //  Send message to worker
    //  If pointer to message is provided, sends that message
    void
    sendWorker (Worker worker_,
        char *command_, std::string option_, vector<szmq::Message> msgs_)
    {
        if (verbose) cout << "+++++ sendWorker()\n";
        vector<szmq::Message> msgs = msgs_;

        //  Stack protocol envelope to start of message
        if (option_.size() > 0)                 //  Optional frame after command
            msgs.insert(msgs.begin(), szmq::Message::from(option_));
        msgs.insert(msgs.begin(), szmq::Message::from(command_)); 
        msgs.insert(msgs.begin(), szmq::Message::from(MDPW_WORKER)); 
        //  Stack routing envelope to start of message
        msgs.insert(msgs.begin(), szmq::Message::from("")); 
        msgs.insert(msgs.begin(), szmq::Message::from(worker_.getId())); 
        if (verbose) {
            cout << boost::format("I: sending %1% to worker\n") % 
                mdps_commands [(int) *command_];
            for (auto it = begin(msgs); it != end(msgs); ++it) 
                it->dump();
        }
        socket.sendMultiple(msgs);
        //msgs.clear();
        if (verbose) cout << "----- sendWorker()\n";
    }

    //  ---------------------------------------------------------------------
    //  This worker is now waiting for work
    void   
    waitWorker(Worker wrk_)
    {
        if (verbose) cout << "+++++ waitWorker()\n";
        assert(wrk_.getId().size() > 0);
        Worker wrk = wrk_;        
        //  Queue to broker and service waiting lists
        wrk.setExpiry(szmq::now() + HEARTBEAT_EXPIRY);
        servicesMap[wrk.service.getName()] = wrk.service;
        servicesMap[wrk.service.getName()].workersMap[wrk.getId()] = wrk;
        // Attempt to process outstanding requests
        vector<szmq::Message> msgs;
        dispatchService (wrk.service, msgs);
        workersMap[wrk.getId()] = wrk;
    }
    //  ---------------------------------------------------------------------
    //  Process a request coming from a client
    void
    processClient (std::string sender_, vector<szmq::Message> msgs_)
    {
        if (verbose) cout << "+++++ processClient()\n";
        assert (msgs_.size() >= 2);     //  Service name + body
        vector<szmq::Message> msgs = msgs_;
        std::string service_name = msgs.front().read<std::string>();
        if (verbose) cout << "service_name : " << service_name << endl;
        msgs.erase(msgs.begin()); // delete the service name
        Service srv = requireService(service_name);
        //  Set reply return address to client sender
        msgs.insert(msgs.begin(), szmq::Message::from("")); 
        msgs.insert(msgs.begin(), szmq::Message::from(sender_)); 
        if (service_name.length() >= 4
        &&  service_name.find_first_of("mmi.") == 0) {
            internalService (service_name, msgs);
        } else {
            dispatchService(srv, msgs);
        }
        msgs.clear();
        if (verbose) cout << "----- processClient()\n";
    }

public:
    //  Get and process messages forever or until interrupted
    void
    startBrokering() {
        uint64_t heartbeat_at = szmq::now() + HEARTBEAT_INTERVAL;
        while (true) {
            std::vector<szmq::PollItem> items = {
                {reinterpret_cast<void*>(*socket), 0, ZMQ_POLLIN, 0}};
            uint64_t timeout = heartbeat_at - szmq::now();
            if (timeout < 0)
                timeout = 0;
            if (verbose) cout << "poll timeout : " << (long)timeout << endl;
            szmq::poll (items, 1, (long)timeout);
            //  Process next input message, if any
            if (items [0].revents & ZMQ_POLLIN) {
                auto msgs = socket.recvMultiple();
                if (verbose) {
                    cout << "I: received message:" << endl;
                    for (auto it = begin(msgs); it != end(msgs); ++it) 
                        it->dump();
                }
                std::string sender =  msgs.front().read<std::string>();
                msgs.erase(msgs.begin()); // delete the sender id
                msgs.erase(msgs.begin()); // empty delimiter the sender id
                std::string header =  msgs.front().read<std::string>();
                msgs.erase(msgs.begin()); // delete the header
                if (verbose) {
                    cout << "========================\n";
                    cout << "sbrok, sender: "<< szmq::strhex(sender) << endl;
                    cout << "sbrok, header: "<< header << endl;
                    cout << "msg size: " << msgs.size() << endl;
                    for (auto it = begin(msgs); it != end(msgs); ++it) 
                        it->dump();
                }
                if (header.compare(MDPC_CLIENT) == 0) {
                    if (verbose) cout << "I: receive MDPC_CLIENT : " << szmq::strhex(sender) << endl;
                    processClient(sender, msgs);
                }
                else if (header.compare(MDPW_WORKER) == 0) {
                    if (verbose) cout << "I: receive MDPW_WORKER : " << szmq::strhex(sender) << endl;
                    processWorker(sender, msgs);
                }
                else {
                    cout <<"E: invalid message:" << endl;
                    for (auto it = begin(msgs); it != end(msgs); ++it) 
                        it->dump();
                    msgs.clear();
                }
            }
            //  Disconnect and delete any expired workers
            //  Send heartbeats to idle workers if needed
            if (szmq::now() >= heartbeat_at) {
                purgeWorkers ();
                vector<szmq::Message> tmp;
                for (auto it = workersMap.begin(); it != workersMap.end(); ++it)                     
                    sendWorker(it->second, (char*)MDPW_HEARTBEAT, "", tmp);
                heartbeat_at += HEARTBEAT_INTERVAL;
            }
        }
    }

private:
    szmq::Context& context;                  //  0MQ context
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> socket;                    //  Socket for clients & workers
    int verbose;                               //  Print activity to stdout
    std::string endpoint;                      //  Broker binds to this endpoint
    std::map<std::string, Service> servicesMap;  //  Hash of known services
    std::map<std::string, Worker> workersMap;    //  Hash of known workers
    //std::list<Worker> workers;              //  List of waiting workers
};

#endif
```

### mdbroker.java 브로커 프로그램

```java
//  Majordomo Protocol broker
//  A minimal C implementation of the Majordomo Protocol as defined in
//  http://rfc.zeromq.org/spec:7 and http://rfc.zeromq.org/spec:8.

#include "mdbroker.h"
#include <szmq/szmq.h>

//  ---------------------------------------------------------------------
//  Main broker work happens here

int main (int argc, char *argv [])
{
    szmq::Context ctx;
    int verbose = (argc > 1 && strcmp (argv [1], "-v") == 0);
    mdBroker brk(ctx, verbose);
    brk.bind ("tcp://*:5555");
    brk.startBrokering();

    return 0;
}
```

이것은 우리가 본 것 중 가장 복잡한 예제입니다. 거의 500라인의 코드이지만 완전한 서비스 지향 브로커로는 짧은 편입니다.

브로커 코드에서 주의할 몇 가지 사항은 다음과 같습니다.

* MDP는 단일 ROUTER 소켓에서 클라이언트들과 작업자들을 모두 처리할 수 ​​있습니다. 이는 브로커를 배포하고 관리하는 데는 좋습니다: 대부분의 프록시가 필요로 하는 두 개가 아닌 하나의 ØMQ 단말에 있습니다.
* 브로커는 모든 MDP/v0.1 사양을 구현하였으며,  브로커가 잘못된 명령을 보내는 경우에 연결 해제, 심박 및 나머지들을 포함합니다.
* 멀티스레드로 실행하기 위해 확장될 수 있으며, 각각의 스레드는 하나의 소켓과 하나의 클라이언트 및 작업자 집합을 관리합니다. 이것은 대규모 아키텍처로 분할하기 위한 흥미로운 주제가 될 수 있습니다. C 코드는 작업을 쉽게 하기 위해 이미 브로커 클래스를 중심으로 구성되어 있습니다.
* 기본(primary)/장애조치(failover) 또는 라이브(live)/라이브(live) 브로커 신뢰성 모델은 쉽지만, 브로커는 기본적으로 서비스 존재를 제외하고는 상태가 없기 때문에, 클라이언트들과 작업자들이 처음 선택한 브로커가 죽었을 경우 다른 브로커를 선택하는 것은 클라이언트들과 작업자들의 책임입니다.
* 예제는 2.5초간격 심박을 사용하며, 주로 추적을 활성화할 때 출력량을 줄이기 위해서입니다.  실제 값은 대부분의 LAN 응용프로그램에서 더 작을 수 있으나 모든 재시도는 혹시 서비스가 재시작되는 것을 고려해 충분히 늦추어져야 합니다(최소 10초).

현재 MDP 구현과 통신규약은 개선하고 확장하였으며, 현재 자체 [Github 프로젝트](https://github.com/zeromq/majordomo)로 자리 잡았습니다. 적절하게 사용 가능한 MDP 스택을 원한다면 GitHub 프로젝트를 사용하십시오.

### 빌드 및 테스트
- 브로커(mdbroker)와 작업자(mdworker)를 실행하고 나서 클라이언트(mdclient)를 수행합니다.
- 클라이언트(mdclient)에서 루프를 100,000번 수행합니다.
- 작업자(mdworker)에 등록된 서비스 이름이 "echo"와 클라이언트에서 요청하는 서비스 이름인 "echo"가 일치해야만 정상 구동됩니다.

~~~{.bash}
// 원도우
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd mdworker.java szmq.lib  
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd mdclient.java szmq.lib  
PS D:\work\sook\src\szmq\examples> cl -EHsc -MTd mdbroker.java szmq.lib  

PS D:\work\sook\src\szmq\examples> ./mdbroker
I: MDP broker/0.1.1 is active at tcp://*:5555

PS D:\work\sook\src\szmq\examples> ./mdworker

PS D:\git_store\zguide\examples\C++> ./mdclient
100000 requests/replies processed

// 리눅스
zedo@sook:/work/sook/src/szmq/examples$ g++ -o mdworker  mdworker.java -lsook-szmq -lzmq
zedo@sook:/work/sook/src/szmq/examples$ g++ -o mdclient  mdclient.java -lsook-szmq -lzmq
zedo@sook:/work/sook/src/szmq/examples$ g++ -o mdbroker  mdbroker.java -lsook-szmq -lzmq

zedo@sook:/work/sook/src/szmq/examples$ ./mdbroker
I: MDP broker/0.1.1 is active at tcp://*:5555

zedo@sook:/work/sook/src/szmq/examples$ ./mdworker

zedo@sook:/work/sook/src/szmq/examples$ ./mdclient
100000 requests/replies processed
~~~

* MDP는 단일 ROUTER 소켓에서 클라이언트들과 작업자들을 모두 처리할 수 ​​있습니다. 이는 브로커를 배포하고 관리하는 데는 좋습니다: 대부분의 프록시가 필요로 하는 두 개가 아닌 하나의 ØMQ 단말에 있습니다.
* 브로커는 모든 MDP/v0.1 사양을 구현하였으며,  브로커가 잘못된 명령을 보내는 경우에 연결 해제, 심박 및 나머지들을 포함합니다.
* 멀티스레드로 실행하기 위해 확장될 수 있으며, 각각의 스레드는 하나의 소켓과 하나의 클라이언트 및 작업자 집합을 관리합니다. 이것은 대규모 아키텍처로 분할하기 위한 흥미로운 주제가 될 수 있습니다. C 코드는 작업을 쉽게 하기 위해 이미 브로커 클래스를 중심으로 구성되어 있습니다.
* 기본(primary)/장애조치(failover) 또는 라이브(live)/라이브(live) 브로커 신뢰성 모델은 쉽지만, 브로커는 기본적으로 서비스 존재를 제외하고는 상태가 없기 때문에, 클라이언트들과 작업자들이 처음 선택한 브로커가 죽었을 경우 다른 브로커를 선택하는 것은 클라이언트들과 작업자들의 책임입니다.
* 예제는 2.5초간격 심박을 사용하며, 주로 추적을 활성화할 때 출력량을 줄이기 위해서입니다.  실제 값은 대부분의 LAN 응용프로그램에서 더 작을 수 있으나 모든 재시도는 혹시 서비스가 재시작되는 것을 고려해 충분히 늦추어져야 합니다(최소 10초).

현재 MDP 구현과 통신규약은 개선하고 확장하였으며, 현재 자체 [Github 프로젝트](https://github.com/zeromq/majordomo)로 자리 잡았습니다. 적절하게 사용 가능한 MDP 스택을 원한다면 GitHub 프로젝트를 사용하십시오.

## 비동기 MDP 패턴

이전의 MDP 구현은 간단하지만 멍청합니다. 클라이언트는 섹시한 API로 감싼 단순한 해적 패턴입니다. 명령어창에서 클라이언트, 브로커 및 작업자를 실행하면 약 14초 내에 100,000개의 요청을 처리(`-v` 옵션 제거)할 수 있으며, 이는 CPU 리소스 있는 한도에서 메시지 프레임들을 주변으로 복사 가능하기 때문입니다. 그러나 진짜 문제는 우리가 네트워크 왕복(round-trips)입니다. ØMQ는 네이글 알고리즘을 비활성화하지만 왕복은 여전히 느립니다.

네이글(Nagle) 알고리즘은 TCP/IP 기반의 네트워크에서 특정 작은 인터넷 패킷 전송을 억제하는 알고리즘으로 작은 패킷을 가능한 모아서 큰 패킷으로 모아서 한 번에 전송합니다. 네트워크 전송의 효율을 높여주지만 실시간으로 운용해야 하는 응용프로그램에서는 오히려 지연을 발생시키게 됩니다.

이론은 이론적으로는 훌륭하지만 실제 해보는 것이 좋습니다. 간단한 테스트 프로그램으로 실제 왕복 비용을 측정해 봅시다. 
프로그램에서 대량의 메시지들을 보내고, 
* 첫째 각 메시지에 대한 응답을 하나씩 기다리고, 
* 두 번째는 일괄 처리로, 모든 응답을 한꺼번에 읽게 합니다. 

두 접근 방식 모두 동일한 작업을 수행하지만 결과는 매우 다릅니다. 클라이언트, 중개인 및 작업자를 동작하는 예제를 작성하겠습니다.

### tripping.java : 왕복 데모(Round-trip demonstrator)

```java
//  Round-trip demonstrator
//  While this example runs in a single process, that is just to make
//  it easier to start and stop the example. The client task signals to
//  main when it's ready.

#include <szmq/szmq.h>
#include <czmq.h>
#include <iostream>
#include <chrono>
#include <cstdlib>
#include <thread>
#include <boost/format.hpp>
using namespace std;

static void *
client_task (szmq::Context *ctx)
{
    char fmt[255];
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> pipe(*ctx);
    pipe.connect(szmq::SocketUrl("inproc://internal"));

    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> client(*ctx);
    client.setId();
    client.connect(szmq::SocketUrl("tcp://localhost:5555"));

    cout << "Setting up test...\n";
    zclock_sleep(100);

    int requests;
    uint64_t start;

    cout << "Synchronous round-trip test...\n";
    start = zclock_time();
    for (requests = 0; requests < 10000; requests++) {
        client.sendOne(szmq::Message::from("hello"));
        auto reply = client.recv1();
    }
    snprintf(fmt, 255, " %d calls/second", 
        (1000 * 10000) / (int) (zclock_time () - start));
    cout << fmt << endl;

    cout << "Asynchronous round-trip test...\n";
    start = zclock_time();
    for (requests = 0; requests < 100000; requests++)
        client.sendOne(szmq::Message::from("hello"));
    for (requests = 0; requests < 100000; requests++) 
        auto reply = client.recv1();
    snprintf(fmt, 255, " %d calls/second", 
        (1000 * 10000) / (int) (zclock_time () - start));
    cout << fmt << endl;
    pipe.sendOne(szmq::Message::from("done"));
    client.close();
    pipe.close();
    return NULL;
}

//  .split worker task
//  Here is the worker task. All it does is receive a message, and
//  bounce it back the way it came:

static void *
worker_task (void *args)
{
    szmq::Context context;
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> worker(context);
    worker.connect(szmq::SocketUrl("tcp://localhost:5556"));

    while (true) {
        auto msg = worker.recv1();
        worker.sendOne(msg);
    }
    worker.close();
    return NULL;
}

//  .split broker task
//  Here is the broker task. It uses the {{zmq_proxy}} function to switch
//  messages between frontend and backend:

static void *
broker_task (void *args)
{
    //  Prepare our context and sockets
    szmq::Context context;
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> frontend(context);
    frontend.bind(szmq::SocketUrl("tcp://*:5555"));
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> backend(context);
    backend.bind(szmq::SocketUrl("tcp://*:5556"));
    szmq::proxy (reinterpret_cast<void*>(*frontend), reinterpret_cast<void*>(*backend), NULL);
    frontend.close();
    backend.close();
    return NULL;
}

//  .split main task
//  Finally, here's the main task, which starts the client, worker, and
//  broker, and then runs until the client signals it to stop:

int main (void)
{
    //  Create threads
    szmq::Context context;
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> pipe(context);
    pipe.bind(szmq::SocketUrl("inproc://internal"));

    thread(&client_task, &context).detach();
    thread(&worker_task, nullptr).detach();
    thread(&broker_task, nullptr).detach();

    //  Wait for signal on client pipe
    auto signal = pipe.recvOne();
    pipe.close();
    return 0;
}

```

클라이언트 스레드는 시작하기 전에 잠시 대기를 수행(`zclock_sleep(100)`)합니다. 이것은 라우터 소켓의 "기능들"중 하나는 아직 연결되지 않은 상대의 주소로 메시지를 보내면 메시지가 유실됩니다. 예제에서는 부하 분산 메커니즘을 사용하지 않아, 일정 시간 동안 대기하지 않으면, 작업자 스레드의 연결이 지연되어 메시지가 유실되면 테스트가 엉망이 됩니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc  tripping.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./tripping1
Setting up test...
Synchronous round-trip test...
 1737 calls/second
Asynchronous round-trip test...
 135135 calls/second
PS D:\work\sook\src\szmq\examples> ./tripping1
Setting up test...
Synchronous round-trip test...
 1986 calls/second
Asynchronous round-trip test...
 138888 calls/second
~~~

테스트 결과를 보았듯이, 간단한 예제의 경우에서 동기식으로 진행되는 왕복은 비동기식보다 약 10배 더 느립니다. "물 들어올 때 노 저어라" 말처럼, 비동기식을 MDP에 적용하여 더 빠르게 만들 수 있는지 보겠습니다.

먼저 클라이언트 객체의 `send()`을 수정하여 송신(`send()`)과 수신(`recv()`)으로 분리합니다.

### mdclient2.h: 비동기 MDP 클라이언트 객체

```java
/*  =====================================================================
 *  mdclient.h - Majordomo Protocol Client class
 *  Implements the MDP/Worker spec at http://rfc.zeromq.org/spec:7.
 *  ===================================================================== */

#ifndef __MDCLIENT_H_INCLUDED__
#define __MDCLIENT_H_INCLUDED__

#include <szmq/szmq.h>
#include <iostream>
#include <iomanip>
#include <sstream>
#include <thread>
#include <boost/format.hpp>
#include "mdp.h"
using namespace std;
class mdClient {
public:
    //  ---------------------------------------------------------------------
    //  Constructor

    mdClient (szmq::Context& context, std::string broker, int verbose, int timeout, int retry)
        : mContext(context), 
          mBroker(broker), 
          mVerbose(verbose), 
          mTimeout(timeout), 
          mRetries(retry),
          mClient(context) {
        connectBroker ();
    }
    //  ---------------------------------------------------------------------
    //  Destructor
    virtual
    ~mdClient (){ }


    //  ---------------------------------------------------------------------
    //  Connect to broker
    void connectBroker ()    {           
        //mClient.setId();
        mClient.connect(szmq::SocketUrl(mBroker));
        if (mVerbose) {
            VLOG(4) << "I: connecting to broker at " << mBroker << "...";
        }
    }
    //  ---------------------------------------------------------------------
    //  reconnect to broker
    void reconnectBroker (std::vector<szmq::Message> requests)    {    
        szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> mClient(mContext);
        //mClient.setId();
        mClient.connect(szmq::SocketUrl(mBroker));
        if (mVerbose) {
            VLOG(4) << "I: connecting to broker at " << mBroker << "...";
        }
        mClient.sendMultiple(requests);
    }


    //  ---------------------------------------------------------------------
    //  Set request timeout

    void
    setTimeout (int timeout)
    {
        mTimeout = timeout;
    }


    //  ---------------------------------------------------------------------
    //  Set request retries

    void
    setRetries (int retries)
    {
        mRetries = retries;
    }


    //  The send method now just sends one message, without waiting for a
    //  reply. Since we're using a DEALER socket we have to send an empty
    //  frame at the start, to create the same envelope that the REQ socket
    //  would normally make for us:

    int
    send (std::string service, szmq::Message request)
    {
        assert (request.size() > 0);
        std::vector<szmq::Message> requests;
        requests.insert(requests.begin(), request);
        //  Prefix request with protocol frames
        //  Frame 0: empty (REQ emulation)
        //  Frame 1: "MDPCxy" (six bytes, MDP/Client x.y)
        //  Frame 2: Service name (printable string)
        requests.insert(requests.begin(), szmq::Message::from(service));
        requests.insert(requests.begin(), szmq::Message::from(MDPC_CLIENT));	   
        requests.insert(requests.begin(), szmq::Message::from(""));	  
        if (mVerbose) {
            VLOG(4) << boost::format("I: send request to '%1%' service:") % service;
            for (auto it = begin (requests); it != end (requests); ++it) 
                it->dump();
        }
        mClient.sendMultiple(requests);
        return 0;
    }
    //  The recv method waits for a reply message and returns that to the 
    //  caller.
    //  ---------------------------------------------------------------------
    //  Returns the reply message or NULL if there was no reply. Does not
    //  attempt to recover from a broker failure, this is not possible
    //  without storing all unanswered requests and resending them all...

    std::vector<szmq::Message>
    recv (std::string service)
    {
        std::vector<szmq::PollItem> items = {
            {reinterpret_cast<void*>(*mClient), 0, ZMQ_POLLIN, 0}};
        szmq::poll(items, 1, mTimeout);
        //  If we got a reply, process it
        if (items [0].revents & ZMQ_POLLIN) {
            auto replies = mClient.recvMultiple();
            if (mVerbose) {
                VLOG(4) << "I: received reply:";
                for (auto it = begin (replies); it != end (replies); ++it) 
                    it->dump();
            }    
            //  Don't try to handle errors, just assert noisily
            assert(replies.size() >= 4);
            replies.erase(replies.begin()); // empty maeesage
            auto header = replies.front().read<std::string>();
            replies.erase(replies.begin()); // delete the MDPC_CLIENT
            assert(header.compare(MDPC_CLIENT)==0);
            auto reply_service = replies.front().read<std::string>();
            replies.erase(replies.begin()); // delete the reply service
            assert(reply_service.compare(service)==0);
            return replies;     //  Success
        }
        if (mVerbose) {
            VLOG(4) << "W: permanent error, abandoning request";
        }
        std::vector<szmq::Message> tmp;
        return tmp;
    }
private:
    std::string mBroker;
    szmq::Context& mContext;          
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> mClient;  //  client socket
    int mVerbose;                //  Print activity to stdout
    int mTimeout;                //  Request timeout
    int mRetries;                //  Request retries
};

#endif
```

차이점은 다음과 같습니다.

* 클라이언트에서 REQ 대신 DEALER 소켓을 사용하여, 각 요청과 각 응답 전에 공백 구분자(empty delimiter) 프레임을 넣어 봉투(envelope)를 구성합니다.
* 요청을 재시도하지 않습니다. 응용프로그램을 재시도 필요한 경우 자체적으로 수행할 수 있습니다.
* 동기식 send(`mdcli_send()`) 메서드를 비동기식 send(`mdcli_send()`) 및 recv(`mdcli_recv()`) 메서드로 분리합니다.
* send 메서드는 비동기식이며 전송 후 즉시 결과를 반환하기 때문에 발신자는 응답을 받기 전에 많은 메시지들을 보낼 수 있습니다.
* recv 메서드는 하나의 응답을 기다렸다가(제한시간(2.5초) 내) 호출자에게 응답 메시지를 반환합니다.

다음은 대응하는 클라이언트 테스트 프로그램으로,
100,000개의 메시지들를 보낸 다음 100,000개의 메시지를 받습니다.

### mdclient2.java : 비동기 MDP 클라이언트

```java
//  Majordomo Protocol client example - asynchronous
//  Uses the mdcli API to hide all MDP aspects

//  Lets us build this source without creating a library
#include "mdclient2.h"
#include <iostream>
using namespace std;
int main (int argc, char *argv [])
{
    szmq::Context ctx;
    int verbose = (ctx, argc > 1 && (strcmp(argv[1], "-v")==0));
    mdClient session(ctx, "tcp://localhost:5555", verbose, 2500, 1);

    int count;
    for (count = 0; count < 100000; count++) {
        auto request = szmq::Message::from("Hello, world");
        session.send("echo", request);
    }

    for (count = 0; count < 100000; count++) {
        auto replies = session.recv("echo");

        if (replies.size())
            replies.clear();
        else
            break;              //  Interrupt or failure
    }
    cout << count << " replies received\n";
    return 0;
}
```

프로토콜을 전혀 수정하지 않았기 때문에 브로커와 작업자의 코드는 변경되지 않았습니다. 즉각적인 성능 향상이 있었습니다. 

핵심 구현 기준은 아니지만 성능의 대가는 복잡성이란 것을 의미합니다. MDP를 위해 구현할 가치가 있을까요? 수천 명의 클라이언트들을 지원하는 웹 프론트엔드의 경우 필요하겠지만, DNS와 같은 이름 조회 서비스의 경우 하나의에서 세션이 완료되면 더 이상 필요하지 않습니다.

## 서비스 검색

우리는 훌륭한 서비스 지향 브로커를 가지고 있지만, 특정 서비스를 사용할 수 있는지 여부를 알 수는 없습니다. 클라이언트에서 요청이 실패했는지 여부(제한시간 내에 응답이 오지 않음)는 알지만 이유는 알 수 없습니다. 브로커에게 "에코 서비스가 실행 중입니까?"라고 물을 수 있으면 유용합니다. 가장 확실한 방법은 MDP/클라이언트 통신규약을 수정하여 브로커에 서비스 실행 여부를 묻는 명령을 추가하는 것입니다. 그러나 MDP/클라이언트는 단순하다는 매력이 있지만 서비스 검색 기능을 추가하면 MDP/작업자 통신규약만큼 복잡해집니다.

RFC로 [MMI: Majordomo Management Interface](http://rfc.zeromq.org/spec:8)는 MDP 통신규약 위에 구축된 작은 RFC입니다. 이미 브로커에서 구현(`internalService()`)했지만, 코드 전부를 읽지 않는다면 아마 놓쳤을 것입니다. 브로커에서 어떻게 작동하는지 설명하겠습니다.

* 클라이언트가 `mmi.`로 시작하는 서비스를 요청하면, 브로커는 작업자에게 전달하는 대신 내부적으로 처리합니다.
* 브로커에서는 서비스 검색 서비스로 `mmi.service` 처리합니다.
* 요청에 대한 반환값은 외부 서비스의 이름(작업자가 제공 한 실제 서비스)입니다.
* 브로커는 해당 서비스로 등록된 작업자가 존재 여부에 따라 "200"(OK) 또는 "404"(Not found)를 반환합니다.

### mmiecho.java: MDP상 서비스 검색

```java
//  MMI echo query example

//  Lets us build this source without creating a library
#include "mdclient.h"
#include <szmq/szmq.h>
using namespace std;
int main (int argc, char *argv [])
{
    szmq::Context ctx;
    int verbose = (ctx, argc > 1 && (strcmp(argv[1], "-v")==0));
    mdClient session(ctx, "tcp://localhost:5555", verbose, 2500, 1);

    //  This is the service we want to look up
    std::vector<szmq::Message> request;
    request.insert(request.begin(), szmq::Message::from("echo"));
    //  This is the service we send our request to
    auto replies = session.send("mmi.service", request);

    if (replies.size()) {
        replies.erase(replies.begin()); // delete empty delimiter
        std::string reply_code = replies.front().read<std::string>();
        cout << "Lookup echo service: " << reply_code << endl;
    }
    else
        cout << "E: no response from broker, make sure it's running\n";

    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc  mmiecho.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./mmiecho -v
[INFO]  I: send request to 'mmi.service' service:
[006]MDPC01
[011]mmi.service
[004]echo
[INFO]  I: received reply:
[006]MDPC01
[011]mmi.service
[000]
[003]200
Lookup echo service: 200
~~~

"echo" 서비스로 등록된 작업자가 실행 혹은 실행되지 않는 상황에서 테스트하면  "200(OK)" 혹은 "404(Not found)"가 표시됩니다.
예제에서 브로커에서 MMI를 구현한 것은 조잡합니다. 예를 들어, 작업자가 사라지더라도 서비스는 "현재"상태로 유지됩니다. 사실 브로커는 제한시간 후에 작업자가 없는 서비스를 제거해야합니다.

"echo" 서비스로 브로커에 등록된 작업자를 중지시키고 "mmiecho"을 두 차례 수행하면 "200(OK)"가 나오나 3번째 부터는 "404(Not found)"가 출력되는 것은 브로커에서 심박 수행시 "s_broker_purge()"을 통하여 대기중인 작업자들의 제한시간 초과(7.5초=2.5초(HEARTBEAT_INTERVAL) * 3(HEARTBEAT_LIVENESS))된 경우 삭제하기 때문입니다.

## 멱등성 서비스

멱등성은 약으로 복용하는 것이 아닙니다. 이것이 의미하는 바는 작업을 반복해도 안전하다는 것입니다. 시계의 시간을 확인하기는 멱등성입니다. 아이들에게 신용 카드를 빌려주기는 아닙니다(아이에 따라 결과가 달라짐). 많은 클라이언트-서버 사용 사례들은 멱등성이지만 일부는 그렇지 않습니다. 멱등성 사례의 다음과 같습니다.
* 멱등성(冪等性)은 연산을 여러 번 적용하더라도 결과가 달라지지 않는 성질입니다.
  - 예) 절댓값 함수 - abs(abs(x)) = abs(x)

멱등성이 아닌 작업을 처리하려면 중복 요청들을 감지하고 거부하는 표준적인 솔루션을 사용해야 하며, 다음을 의미합니다.

* 클라이언트는 모든 요청에 고유한 클라이언트 식별자(ID)와 고유한 메시지 번호로 넣어야 합니다.(client ID + Message No + request)
* 서버는 응답을 보내기 전에, 응답 메시지에 클라이언트 식별자(ID)와 메시지 번호의 조합하여 키로 저장합니다.(client ID + Message No + reply)
* 서버는 클라이언트로부터 요청을 받으면, 먼저 해당 클라이언트 ID와 메시지 번호에 대한 응답이 있는지 확인합니다(client ID + Message No + reply). 서버에 이미 응답이 있다면 요청을 처리하지 않고 기존 응답만 다시 보냅니다.

## 비연결 신뢰성(타이타닉 패턴)

MDP가 "신뢰할 수 있는" 메시지 브로커라는 사실을 알게 되면 하드디스크를 추가 할 수 있습니다. 결국 이것은 대부분의 기업 메시징 시스템에서 동작합니다. 이와 같이 유혹적인 생각은 다소 부정적일 수밖에 없는 것에 유감입니다. 하지만 냉소주의는 내 전문 분야 중 하나입니다. 그래서 아키텍처의 중심에 하드디스크 기반 브로커를 배치하여 지속성을 유지하지 않는 이유는 다음과 같습니다.

* 게으른 해적 클라이언트(LPP)는 잘 작동하였습니다. 클라이언트-서버 직접 연결(LPP)에서 부하 분산 브로커(SPP)까지 전체 범위의 아키텍처에서 동작합니다. 게으른 해적 패턴의 작업자는 상태 비저장이며 멱등성이라고 가정하는 경향이 있지만, 지속성이 필요한 경우 한계가 있습니다.
* 하드디스크는 느린 성능에서 부가적인 관리(새벽 6시 장애, 일과 시작 시 필연적인 고장 등), 수리 등 많은 문제를 가져옵니다. 일반적으로 해적 패턴의 아름다움은 단순성입니다. 해적 패턴의 요소들(클라이언트, 브로커, 작업자)은 충돌하지 않지만, 여전히 하드웨어가 걱정된다면 브로커가 없는 P2P(Peer-To-Peer) 패턴으로 이동할 수 있습니다. 이장의 뒷부분에서 설명하겠습니다.

그러나 이렇게 말했지만 , 하드디스크 기반 안정성이 필연적인 사용 사례는 비연결 비동기 네트워크입니다. 해적의 주요 문제, 즉 클라이언트가 실시간으로 응답을 기다려야 하는 문제를 해결합니다. 클라이언트와 작업자가 가끔씩만 연결되어 있는 경우(이메일과 유사하게 생각) 클라이언트들과 작업자들 간에 상태 비저장 네트워크를 사용할 수 없습니다. 상태를 중간에 두어야 합니다.

여기 타이타닉 패턴이 있으며, 우리는 메시지를 디스크에 기록하여 클라이언트들과 작업자들이 산발적으로 연결되어 있어도 메시지가 유실되지 않도록 합니다. MDP에서 브로커에 서비스 검색 기능(internalService())을 추가한 것처럼 타이타닉을 별도의 통신규약으로 확장하는 대신 MDP 위에 계층화할 것입니다. 이것은 놀랍게도 게으르게 보이는 것은 발사 후 망각형(fire-and-forget) 신뢰성을 브로커가 아닌 전문화된 작업자에 구현하기 때문입니다. 작업자에 구현하기 좋은 이유는 다음과 같습니다.

* 우리가 나누고 정복하기 때문에 훨씬 쉽습니다. 브로커가 메시지 라우팅을 처리하고, 작업자는 신뢰성을 처리합니다.
* 특정 개발 언어로 작성된 브로커와 다른 개발 언어로 작성된 작업자를 혼합할 수 있습니다.
* 작업자를 통해 발사 후 망각형 기술로 독자적으로 진화시킵니다.
단 하나의 단점은 브로커와 하드디스크 간에 부가적인 네트워크 홉이지만 장점들은 단점을 능가하는 가치가 있습니다.

지속적인 요청-응답 아키텍처를 만드는 방법들은 많이 있지만, 우리는 간단하고 고통 없는 것을 지향하겠습니다. 몇 시간 동안 사용해 본 후 가장 간단한 설계는 "프록시 서비스"입니다. 즉, 타이타닉은 작업자들에게 영향을 주지 않습니다. 클라이언트가 즉시 응답을 원하면, 서비스에 직접 말하고 서비스가 가용하기를 기대합니다. 클라이언트가 기꺼이 기다리며 타이타닉과 대화하면서 "이봐, 친구, 내가 식료품을 사는 동안 이걸 처리해 주시겠어요?"라고 묻습니다.

타이타닉은 작업자이자 클라이언트이며, 클라이언트와 타이타닉 간의 대화는 다음과 같습니다.

* 클라이언트 : 나의 요청을 수락해 주세요. - 타이타닉 : 네, 수락했습니다.
* 클라이언트 : 저에게 응답이 있나요? - 타이타닉 : 네, 여기 있습니다. 혹은 아니요, 아직 없습니다.
* 클라이언트 : 네. 요청을 지워주신다면 감사하겠습니다. - 타이타닉 : 네, 지웠습니다.

한편 타이타닉과 브로커, 작업자 간의 대화는 다음과 같습니다.

* 타이타닉 : 안녕, 브로커, 커피 서비스가 있나요? - 브로커 : 음, 예, 있는 것 같아요
* 타이타닉 : 안녕, 커피 서비스(작업자), 이거 처리해 주세요.
* 커피(작업자) : 네, 여기 있습니다.
* 타이타닉 : 달콤해!

타이타닉에 대한 공식 사양서를 작성하기 전에, 클라이언트가 타이타닉과 통신하는 방법을 고려하겠습니다. 하나는 단일 서비스를 사용하여 3개 요청 유형을 보내는 것입니다. 
두 번째는 더 단순하게 그냥 3개의 서비스를 사용하는 것입니다.

* titanic.request : [titanic-client] 요청 메시지를 저장하고, 요청에 대한 UUID를 반환합니다.
* titanic.reply : [titanic-worker] 주어진 요청 UUID에 대해 가능한 응답을 가져옵니다.
* titanic.close : [titanic-client] 응답이 저장되고 처리되었는지 확인합니다.

ØMQ에 대한 멀티스레딩 경험을 통해 간단한 멀티스레드 작업자를 작성해 3개의 서비스를 제공합니다.
타이타닉에서 사용할 ØMQ 메시지와 프레임 설계를 하겠습니다. 이를 타이타닉 서비스 통신규약(TSP)으로 제공됩니다.

TSP를 사용하면 MDP를 통해 직접 서비스에 접근하는 것보다 클라이언트 응용프로그램에 더 많은 작업이 수행됩니다. 다음은 짧지만 강력한 "echo"클라이언트 예제입니다.

### ticlient.java: 타이타닉 클라이언트

```java
//  Titanic client example
//  Implements client side of http://rfc.zeromq.org/spec:9

//  Lets build this source without creating a library
#include "mdclient.h"
#include <szmq/szmq.h>
using namespace std;

//  Calls a TSP service
//  Returns response if successful (status code 200 OK), else NULL
//
static vector<szmq::Message>
s_service_call (mdClient *session_, string service_, std::vector<szmq::Message> request_)
{
    auto reply = session_->send(service_, request_);
    if(reply.size()){
        auto status =  reply.back().read<std::string>();
        if(status.compare("200")==0)
            return reply;
        else 
        if(status.compare("400")==0) {
            cout << "E: client fatal error, aborting\n";
            exit(EXIT_FAILURE);
        }
        else 
        if(status.compare("500")==0) {
            cout << "E: server fatal error, aborting\n";
            exit(EXIT_FAILURE);
        }
    } else 
        exit(EXIT_SUCCESS);   //  Interrupted or failed
    reply.clear();
    return reply; //  Didn't succeed; don't care why not
}
    
//  .split main task
//  The main task tests our service call by sending an echo request:

int main (int argc, char *argv [])
{
    szmq::Context ctx;
    int verbose = (ctx, argc > 1 && (strcmp(argv[1], "-v")==0));
    mdClient session(ctx, "tcp://localhost:5555", verbose, 2500, 1);


    //  1. Send 'echo' request to Titanic
    std::vector<szmq::Message> request;
    request.insert(request.begin(), szmq::Message::from("echo"));
    request.insert(request.begin(), szmq::Message::from("Hello world"));
    auto reply = s_service_call(&session, "titanic.request", request);

    std::string uuid;
    if (reply.size()) {
        reply.erase(reply.begin()); // delete return value
        uuid = reply.front().read<std::string>();
        cout << boost::format("I: request UUID : %1%\n") % uuid;
    }
    //  2. Wait until we get a reply
    while (true) {
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        request.clear();
        request.insert(request.begin(), szmq::Message::from(uuid));
        reply = s_service_call(&session, "titanic.reply", request);
        if (reply.size()) {
            auto reply_string = reply.back().read<std::string>();
            cout << "Reply : " << reply_string << endl;

            //  3. Close request
            request.clear();
            request.insert(request.begin(), szmq::Message::from(uuid));
            reply = s_service_call(&session, "titanic.close", request);
            break;
        }
        else {
            cout << "I: no reply yet, trying again...\n";
            std::this_thread::sleep_for(std::chrono::milliseconds(5000));    //  Try again in 5 seconds
        }
    }
    return 0;
}
```

### 빌드 및 테스트

타아타닉 작업자가 구동중이 아닌 상태에서 클라이언트 구동시 아래와 같은 결과가 발생합니다.

~~~{.bash}
PS D:\work\sook\src\szmq\examples> ./ticlient -v
[006]MDPC01
[015]titanic.request
[011]Hello world
[004]echo
[INFO]  W: permanent error, abandoning request
~~~

물론 이러한 기능은 어떤 종류의 프레임워크나 API로 감싸질 수 있습니다. 일반 응용프로그램 개발자에게 전체 메시징의 세부 사항을 배우도록 요청하는 것은 좋지 않습니다 : 이는 개발자들의 머리를 아프게 하고, 시간이 소요되며, 코드가 복잡하고 버그가 많게 되며 지능을 부가하기 어렵습니다.

### uuid_test.java 타이타닉에 사용되는 UUID 생성을 위한 테스트 프로그램

```java
#include <iostream>
#include <boost/algorithm/string.hpp>
#include <boost/uuid/uuid.hpp>            // uuid class
#include <boost/uuid/uuid_generators.hpp> // generators
#include <boost/uuid/uuid_io.hpp>         // streaming operators etc.
using namespace std;

void main()
{
	std::string tmp = boost::uuids::to_string(boost::uuids::random_generator()());
	std::string tmp1 = boost::algorithm::replace_all_copy(tmp, "-", "");
    cout << "uuidstr : " << tmp1 << endl;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc uuid_test.java

PS D:\work\sook\src\szmq\examples> ./uuid_test
uuidstr : 52da7b17d13845bd8fdc4dd7754b68ed
PS D:\work\sook\src\szmq\examples> ./uuid_test
uuidstr : 88695b8590474056a8970a72cb8e04d3
PS D:\work\sook\src\szmq\examples> ./uuid_test
uuidstr : d1cc01f25f9044f3adf33f51168ce248
...
~~~

예를 들어, 실제 응용프로그램에서 클라이언트는 각 요청을 차단할 수 있지만, 작업이 실행되는 동안 유용한 작업을 수행하고 싶어 합니다. 이를 위하여 백그라운드 스레드를 구축하고 스레드 간 통신하기 위해 주요한 연결 작업이 필요합니다. 이런 요구사항에 대하여 일반 개발자가 잘못 사용하지 않도록 단순한 API로 감싸 내부를 추상화하는 방법을 제공하는 것이, MDP에서 서비스 확인에서 사용한 동일한 방식입니다.

다음은 타이타닉 서버의 구현입니다. 타이타닉은 클라이언트와의 대화를 처리하기 위해 3개의 스레드들을 사용하여 3개의 서비스를 처리하며, 가장 확실한 접근 방식(메시지 당 하나의 파일)을 사용하여 디스크를 통한 완전한 지속성을 수행합니다. 너무 놀라울 만큼 단순합니다. 유일한 복잡한 부분은 디렉토리를 계속해서 읽는 것을 피하기 위해 모든 요청들에 대한 개별 대기열을 보유하게 하는 부분입니다.

### titanic.java: 타이타닉 서버

```java
//  Titanic service
//  Implements server side of http://rfc.zeromq.org/spec:9

//  Lets us build this source without creating a library
#include "mdworker.h"
#include "mdclient.h"
#include <szmq/szmq.h>
#include <iostream>
#include <fstream>
#include <sstream>
#include <boost/format.hpp>
#include <boost/filesystem.hpp>           // file operation
#include <boost/uuid/uuid.hpp>            // uuid class
#include <boost/uuid/uuid_generators.hpp> // generators
#include <boost/uuid/uuid_io.hpp>         // streaming operators etc.
#include <boost/algorithm/string.hpp>
using namespace std;

//  Return a new UUID as a printable character string
//  Caller must free returned string when finished with it

static std::string
s_generate_uuid (void)
{
    std::string tmp = boost::uuids::to_string(boost::uuids::random_generator()());
    std::string uuidstr =  boost::algorithm::replace_all_copy(tmp, "-", "");
    return uuidstr;
}

//  Returns freshly allocated request filename for given UUID

#define TITANIC_DIR ".titanic"

static std::string
s_request_filename (std::string uuid) {
    std::string filename = boost::str(boost::format("%1%/%2%.req") % TITANIC_DIR % uuid);
    return filename;
}

//  Returns freshly allocated reply filename for given UUID

static std::string
s_reply_filename (std::string uuid) {
    std::string filename = boost::str(boost::format("%1%/%2%.rep") % TITANIC_DIR % uuid);
    return filename;
}

//  .split Titanic request service
//  The {{titanic.request}} task waits for requests to this service. It writes
//  each request to disk and returns a UUID to the client. The client picks
//  up the reply asynchronously using the {{titanic.reply}} service:

static void *
titanic_request (void *ctx, int verbose_)
{
    szmq::Context *context = (szmq::Context *)ctx;
    mdWorker worker(*context, "tcp://localhost:5555", "titanic.request", verbose_, 2500, 2500);
    // internal processing
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> pipe(*context);
    pipe.connect(szmq::SocketUrl("inproc://internal"));

    std::vector<szmq::Message> reply;

    while (true) {
        //  Send reply if it's not null
        //  And then get next request from broker
        auto request = worker.recv(reply);
        if (verbose_) {
            cout << "titanic_request() received request " << endl;
            for (auto it = begin (request); it != end (request); ++it) 
                    it->dump();
        }
        if (request.size()==0)
            break;      //  Interrupted, exit

        //  Ensure message directory exists
        boost::filesystem::path dir(TITANIC_DIR);
        boost::filesystem::create_directory(dir);

        //  Generate UUID and save message to disk
        auto uuid = s_generate_uuid ();
        if (verbose_) cout << "titanic_request()  uuid : " << uuid << endl;
        auto filename = s_request_filename (uuid);
        boost::filesystem::ofstream file(filename);
        for (auto it = begin (request); it != end (request); ++it) 
                file << it->read<std::string>() << endl;
        file.close();
        request.clear();

        //  Send UUID through to message queue
        pipe.sendOne(szmq::Message::from(uuid));

        //  Now send UUID back to client
        //  Done by the mdwrk_recv() at the top of the loop
        reply.clear();
        reply.insert(reply.begin(), szmq::Message::from("200"));
        reply.insert(reply.begin(), szmq::Message::from(uuid));
    }
    return NULL;
}

//  .split Titanic reply service
//  The {{titanic.reply}} task checks if there's a reply for the specified
//  request (by UUID), and returns a 200 (OK), 300 (Pending), or 400
//  (Unknown) accordingly:

static void *
titanic_reply (void *ctx, int verbose_)
{
    szmq::Context *context = (szmq::Context *)ctx;
    mdWorker worker(*context, "tcp://localhost:5555", "titanic.reply", verbose_, 2500, 2500);

    std::vector<szmq::Message> reply;
    while (true) {
        //  Send reply if it's not null
        //  And then get next request from broker
        auto request = worker.recv(reply);
        if (verbose_) {
            cout << "titanic_reply() received request " << endl;
            for (auto it = begin (request); it != end (request); ++it) 
                    it->dump();       
        }
        if (request.size()==0)
            break;      //  Interrupted, exit
        std::string uuid = request.front().read<std::string>();
        request.erase(request.begin()); // delete uuid
        auto req_filename =  s_request_filename (uuid);
        auto rep_filename =  s_reply_filename (uuid);
        if (boost::filesystem::exists(rep_filename)){
            boost::filesystem::ifstream file(rep_filename);
            std::string line;
            while (std::getline(file, line)) {
                reply.insert(reply.begin(), szmq::Message::from(line));
            }
            reply.insert(reply.end(), szmq::Message::from("200"));
            file.close();
        }
        else {
            reply.clear();
            if(boost::filesystem::exists(req_filename))
                reply.insert(reply.end(), szmq::Message::from("300"));   //Pending
            else
                reply.insert(reply.end(), szmq::Message::from("400"));   //Unknown
        }
        request.clear();
    }
    return NULL;
}

//  .split Titanic close task
//  The {{titanic.close}} task removes any waiting replies for the request
//  (specified by UUID). It's idempotent, so it is safe to call more than
//  once in a row:

static void *
titanic_close (void *ctx, int verbose_)
{
    szmq::Context *context = (szmq::Context *)ctx;
    mdWorker worker(*context, "tcp://localhost:5555", "titanic.close", verbose_, 2500, 2500);

    std::vector<szmq::Message> reply;

    while (true) {
        //  Send reply if it's not null
        //  And then get next request from broker
        auto request = worker.recv(reply);
        if (verbose_) {
            cout << "titanic_reply() received request " << endl;
            for (auto it = begin (request); it != end (request); ++it) 
                    it->dump();   
        }
        if (request.size()==0)
            break;      //  Interrupted, exit
        auto uuid =  request.front().read<std::string>();
        request.erase(request.begin()); // delete the uuid
        std::string req_filename = s_request_filename (uuid);
        std::string rep_filename = s_reply_filename (uuid);

        boost::filesystem::remove(req_filename);
        boost::filesystem::remove(rep_filename);

        reply.clear();
        reply.insert(reply.begin(), szmq::Message::from("200"));
    }
    return NULL;
}

//  .split try to call a service
//  Here, we first check if the requested MDP service is defined or not,
//  using a MMI lookup to the Majordomo broker. If the service exists,
//  we send a request and wait for a reply using the conventional MDP
//  client API. This is not meant to be fast, just very simple:

static int s_service_success (void *ctx, std::string uuid, int verbose_)
{
    //  Load request message, service will be first frame
    std::string filename = s_request_filename (uuid);
    //  If the client already closed request, treat as successful
    if (!boost::filesystem::exists(filename))
        return 1;

    vector<szmq::Message> request;
    boost::filesystem::ifstream file(filename);
    std::string line;
    while (std::getline(file, line)) {
        request.insert(request.begin(), szmq::Message::from(line));
    }
    file.close();
    auto service = request.front();
    auto service_name = service.read<std::string>();

    //  Create MDP client session with short timeout
    szmq::Context *context = (szmq::Context *)ctx;
    mdClient client(*context, "tcp://localhost:5555",  verbose_, 1000, 1);     // 1sec timeout, 1 retry

    //  Use MMI protocol to check if service is available
    std::vector<szmq::Message> mmi_request;
    mmi_request.insert(mmi_request.begin(), service);
    auto reply = client.send("mmi.service", mmi_request);
    reply.erase(reply.begin()); // delete empty delimiter
    auto service_ok = (reply.size() > 0 && reply.front().read<std::string>().compare("200")==0);
    if (verbose_) cout << "service_ok : " << service_ok << endl;
    reply.clear();

    int result = 0;
    if (service_ok) {
        auto reply = client.send(service_name, request);
        if (reply.size() > 0) {
            filename = s_reply_filename (uuid);
            boost::filesystem::ofstream file(filename);
            assert(file);
            for (auto it = begin (reply); it != end (reply); ++it) 
                    file << it->read<std::string>() << endl;
            file.close();
            reply.clear();
            result = 1;
        }
    }
    return result;
}

//  .split worker task
//  This is the main thread for the Titanic worker. It starts three child
//  threads; for the request, reply, and close services. It then dispatches
//  requests to workers using a simple brute force disk queue. It receives
//  request UUIDs from the {{titanic.request}} service, saves these to a disk
//  file, and then throws each request at MDP workers until it gets a
//  response.

int main (int argc, char *argv [])
{
    szmq::Context context;
    int verbose = (argc > 1 && (strcmp(argv[1], "-v")==0));
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> request_pipe(context);
    request_pipe.bind(szmq::SocketUrl("inproc://internal"));
    {
        thread ti_request(&titanic_request, (void *)&context, verbose);
        ti_request.detach();
    }
    {
        thread ti_reply(&titanic_reply, (void *)&context,verbose);
        ti_reply.detach();
    }
    {
        thread ti_close(&titanic_close, (void *)&context,verbose);
        ti_close.detach();
    }
    //  Main dispatcher loop
    std::vector<szmq::PollItem> items ={
        {reinterpret_cast<void*>(*request_pipe), 0, ZMQ_POLLIN, 0}};
    while (true) {
        //  We'll dispatch once per second, if there's no activity
        szmq::poll(items, 1, 1000);
        if (items[0].revents & ZMQ_POLLIN) {
            //  Ensure message directory exists
            boost::filesystem::path dir(TITANIC_DIR);
            boost::filesystem::create_directory(dir);
            //  Append UUID to queue, prefixed with '-' for pending
            auto msg = request_pipe.recvOne();
            msg.dump();
            if (msg.size()==0)
                break;          //  Interrupted
            auto uuid = msg.read<std::string>();
            std::string filename = boost::str(boost::format("%1%/queue") % TITANIC_DIR);
            boost::filesystem::ofstream file(filename);
            file << "-" << uuid << endl;
            file.close();
        }
        //  Brute force dispatcher
        char entry [] = "?.......:.......:.......:.......:";
        FILE *file = fopen (TITANIC_DIR "/queue", "r+");
        while (file && fread (entry, 33, 1, file) == 1) {
            //  UUID is prefixed with '-' if still waiting
            if (entry [0] == '-') {
                if (verbose) cout <<"I: processing request" << string(entry+1) << endl;
                if (s_service_success ((void *)&context, string(entry+1), verbose)) {
                    if (verbose) 
                        cout << "I: s_service_success() OK, change : " << "+" <<  string(entry+1) << endl;
                    //  Mark queue entry as processed
                    fseek (file, -33, SEEK_CUR);
                    fwrite ("+", 1, 1, file);
                    fseek (file, 32, SEEK_CUR);
                }
            }
            //  Skip end of line, LF or CRLF
            if (fgetc (file) == '\r')
                fgetc (file);
        }
        if (file)
            fclose (file);
    }
    return 0;
}
```

### 빌드 및 테스트

테스트를 위해서는 mdbroker, mdworker, titanic, ticlient 순서로 실행이 필요합니다.

~~~{.bash}
// 원도우
PS D:\work\sook\src\szmq\examples> cl -EHsc  mdbroker.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc  mdworker.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc  titanic.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc  ticlient.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./mdbroker
I: MDP broker/0.1.1 is active at tcp://*:5555
...
srv.workersMap.empty() : 0, srv.requests.empty() : 1
servicesMap[titanic.request]
srv.workersMap.empty() : 0, srv.requests.empty() : 0
servicesMap[titanic.request]
srv.workersMap.empty() : 0, srv.requests.empty() : 1
servicesMap[echo]
srv.workersMap.empty() : 0, srv.requests.empty() : 0
servicesMap[echo]
srv.workersMap.empty() : 0, srv.requests.empty() : 1
servicesMap[titanic.reply]
srv.workersMap.empty() : 0, srv.requests.empty() : 0
servicesMap[titanic.reply]
srv.workersMap.empty() : 0, srv.requests.empty() : 1
servicesMap[titanic.close]
srv.workersMap.empty() : 0, srv.requests.empty() : 0
servicesMap[titanic.close]
...

PS D:\work\sook\src\szmq\examples> ./titanic
[032]5215fb2588ec45c8bed1f063e9f86534
[032]c41d1b6af55540b2a007ac313643a52f
...

PS D:\work\sook\src\szmq\examples> ./mdworker

PS D:\work\sook\src\szmq\examples> ./ticlient -v
I: connecting to broker at tcp://localhost:5555...
I: send request to 'titanic.request' service:
[006]MDPC01
[015]titanic.request
[011]Hello world
[004]echo
I: received reply:
[006]MDPC01
[015]titanic.request
[000]
[032]5215fb2588ec45c8bed1f063e9f86534
[003]200
I: request UUID : 5215fb2588ec45c8bed1f063e9f86534
I: send request to 'titanic.reply' service:
[006]MDPC01
[013]titanic.reply
[032]5215fb2588ec45c8bed1f063e9f86534
I: received reply:
[006]MDPC01
[013]titanic.reply
[000]
[011]Hello world
[004]echo
[000]
[003]200
Reply : 20
I: send request to 'titanic.close' service:
[006]MDPC01
[013]titanic.close
[032]5215fb2588ec45c8bed1f063e9f86534
I: received reply:
[006]MDPC01
[013]titanic.close
[000]
[003]200

// 리눅스
zedo@sook:/work/sook/src/szmq/examples$ g++ -o titanic  titanic.java -lsook-szmq -lzmq -lpthread -lboost_system -lboost_filesystem
zedo@sook:/work/sook/src/szmq/examples$ g++ -o ticlient  ticlient.java -lsook-szmq -lzmq

zedo@sook:/work/sook/src/szmq/examples$ ./mdbroker
I: MDP broker/0.1.1 is active at tcp://*:5555

zedo@sook:/work/sook/src/szmq/examples$ ./mdworker

zedo@sook:/work/sook/src/szmq/examples$ ./titanic
[032]01a3352c54734a44a77627e4b96f6aad

zedo@sook:/work/sook/src/szmq/examples$ ./ticlient -v
I: connecting to broker at tcp://localhost:5555...
I: send request to 'titanic.request' service:
[006]MDPC01
[015]titanic.request
[011]Hello world
[004]echo
I: received reply:
[006]MDPC01
[015]titanic.request
[000]
[032]01a3352c54734a44a77627e4b96f6aad
[003]200
I: request UUID : 01a3352c54734a44a77627e4b96f6aad
I: send request to 'titanic.reply' service:
[006]MDPC01
[013]titanic.reply
[032]01a3352c54734a44a77627e4b96f6aad
I: received reply:
[006]MDPC01
[013]titanic.reply
[000]
[011]Hello world
[004]echo
[000]
[003]200
Reply : 200
I: send request to 'titanic.close' service:
[006]MDPC01
[013]titanic.close
[032]01a3352c54734a44a77627e4b96f6aad
I: received reply:
[006]MDPC01
[013]titanic.close
[000]
[003]200
~~~

리눅스의 경우 메이크 파일(makefile)을 통하여 빌드를 수행하면 간단하게 진행할 수 있습니다.
(titnaic.mk)

~~~{.bash}
TARGET = titanic 
OBJECTS = $(TARGET).o 
SOURCES = $(TARGET).java
BOOST = -lboost_system -lboost_filesystem
ZMQ = -lsook-szmq -lzmq
PTHREAD = -lpthread
GCC = g++ -std=c++17
FLAGS = -Wall -pedantic -Wextra 

build: $(OBJECTS)
        $(GCC) $(FLAGS) $(OBJECTS) -o $(TARGET) $(ZMQ) $(PTHREAD) $(BOOST)

$(OBJECTS): $(SOURCE)
        $(GCC) $(FLAGS) -c $(SOURCES)

clean:
        rm -rf *.o $(TARGET)
~~~

테스트를 위해 mdbroker 및 titanic을 시작한 다음, ticlient를 실행합니다. 그리고 mdworker를 시작하면 클라이언트(ticlient)가 응답을 받고 행복하게 종료되는 것을 볼 수 있습니다.

* 일부 루프는 전송으로 시작되고, 다른 루프는 메시지 수신으로 시작됩니다. 이는 타이타닉이 서로 다른 역할들로 클라이언트와 작업자 역할을 수행하기 때문입니다.
* 타이타닉 브로커는 MMI 서비스 검색 통신규약을 사용하여 실행 중인 서비스들에게만 요청을 보냅니다. 우리의 작은 MDP 브로커의 MMI 구현은 매우 간소화되어 항상 작동하지는 않을 겁니다.
* inproc 연결(request_pipe)을 통해 titanic.request 서비스로부터 메인 디스패처로 새로운 요청 데이터를 보냅니다. 이렇게 하면 디스패처가 디스크 디렉터리를 읽고, 모든 요청 파일을 메모리에 적제하고, 파일을 날짜/시간별로 정렬할 필요가 없습니다.

## 고가용성 쌍 (바이너리 스타 패턴)

바이너리 스타 패턴은 2개의 서버를 기본-백업으로 고가용성 쌍으로 배치합니다. 주어진 시간에 이들 중 하나(활성)는 클라이언트 응용프로그램의 연결을 수락합니다. 다른 하나(비활성)는 아무것도 하지 않지만, 2대의 서버들은 서로를 모니터링합니다. 활성 서버가 네트워크에서 사라지고 일정 시간이 지나면 비활성 서버가 활성 서버의 역할을 수행하며 요구 사항은 다음과 같습니다.

* 똑바른 고가용성 솔루션을 제공합니다.
* 충분히 이해하고 사용할 만큼 단순해야 합니다.
* 필요할 때만 안정적으로 장애조치를 수행합니다.

구현시 사용되는 용어는 다음과 같습니다.

바이너리 스타에서 사용하는 주요 용어입니다.
* 기본(Primary) : 일반적으로 또는 처음에 활성화된 서버.
* 백업(Backup) : 일반적으로 비활성 상태의 서버. 기본(Primary) 서버가 네트워크에서 사라지고 클라이언트 응용프로그램이 백업 서버에 연결을 요청할 때 활성화됩니다.
* 활성(Active) : 클라이언트 연결을 수락하는 서버. 활성 서버는 최대 하나입니다.
* 수동(Passive) : 활성화된 서버가 사라지면 인계받는 서버. 바이너리 스타 쌍이 정상적으로 실행 중이면, 기본 서버가 활성 상태이고 백업 서버가 비활성 상태입니다. 장애조치가 발생하면 역할이 바뀝니다.

### bstarsrv.java: 바이너리 스타 서버

```java
//  Binary Star server proof-of-concept implementation. This server does no
//  real work; it just demonstrates the Binary Star failover model.
#include <szmq/szmq.h>
#include <iostream>
using namespace std;

//  States we can be in at any point in time
typedef enum {
    STATE_PRIMARY = 1,          //  Primary, waiting for peer to connect
    STATE_BACKUP = 2,           //  Backup, waiting for peer to connect
    STATE_ACTIVE = 3,           //  Active - accepting connections
    STATE_PASSIVE = 4           //  Passive - not accepting connections
} state_t;

//  Events, which start with the states our peer can be in
typedef enum {
    PEER_PRIMARY = 1,           //  HA peer is pending primary
    PEER_BACKUP = 2,            //  HA peer is pending backup
    PEER_ACTIVE = 3,            //  HA peer is active
    PEER_PASSIVE = 4,           //  HA peer is passive
    CLIENT_REQUEST = 5          //  Client makes request
} event_t;

//  Our finite state machine
typedef struct {
    state_t state;              //  Current state
    event_t event;              //  Current event
    uint64_t peer_expiry;        //  When peer is considered 'dead'
} bstar_t;

//  We send state information this often
//  If peer doesn't respond in two heartbeats, it is 'dead'
#define HEARTBEAT 1000          //  In msecs(1sec)

//  .split Binary Star state machine
//  The heart of the Binary Star design is its finite-state machine (FSM).
//  The FSM runs one event at a time. We apply an event to the current state,
//  which checks if the event is accepted, and if so, sets a new state:

static bool
s_state_machine (bstar_t *fsm)
{
    bool exception = false;
    
    //  These are the PRIMARY and BACKUP states; we're waiting to become
    //  ACTIVE or PASSIVE depending on events we get from our peer:
    if (fsm->state == STATE_PRIMARY) {
        if (fsm->event == PEER_BACKUP) {
            cout << "I: connected to backup (passive), ready active\n";
            fsm->state = STATE_ACTIVE;
        }
        else
        if (fsm->event == PEER_ACTIVE) {
            cout << "I: connected to backup (active), ready passive\n";
            fsm->state = STATE_PASSIVE;
        }
        //  Accept client connections
    }
    else
    if (fsm->state == STATE_BACKUP) {
        if (fsm->event == PEER_ACTIVE) {
            cout << "I: connected to primary (active), ready passive\n";
            fsm->state = STATE_PASSIVE;
        }
        else
        //  Reject client connections when acting as backup
        if (fsm->event == CLIENT_REQUEST)
            exception = true;
    }
    else
    //  .split active and passive states
    //  These are the ACTIVE and PASSIVE states:

    if (fsm->state == STATE_ACTIVE) {
        if (fsm->event == PEER_ACTIVE) {
            //  Two actives would mean split-brain
            cout << "E: fatal error - dual actives, aborting\n";
            exception = true;
        }
    }
    else
    //  Server is passive
    //  CLIENT_REQUEST events can trigger failover if peer looks dead
    if (fsm->state == STATE_PASSIVE) {
        if (fsm->event == PEER_PRIMARY) {
            //  Peer is restarting - become active, peer will go passive
            cout << "I: primary (passive) is restarting, ready active\n";
            fsm->state = STATE_ACTIVE;
        }
        else
        if (fsm->event == PEER_BACKUP) {
            //  Peer is restarting - become active, peer will go passive
            cout << "I: backup (passive) is restarting, ready active\n";
            fsm->state = STATE_ACTIVE;
        }
        else
        if (fsm->event == PEER_PASSIVE) {
            //  Two passives would mean cluster would be non-responsive
            cout << "E: fatal error - dual passives, aborting\n";
            exception = true;
        }
        else
        if (fsm->event == CLIENT_REQUEST) {
            //  Peer becomes active if timeout has passed
            //  It's the client request that triggers the failover
            assert (fsm->peer_expiry > 0);
            if (szmq::now() >= fsm->peer_expiry) {
                //  If peer is dead, switch to the active state
                cout << "I: failover successful, ready active\n";
                fsm->state = STATE_ACTIVE;
            }
            else
                //  If peer is alive, reject connections
                exception = true;
        }
    }
    return exception;
}

//  .split main task
//  This is our main task. First we bind/connect our sockets with our
//  peer and make sure we will get state messages correctly. We use
//  three sockets; one to publish state, one to subscribe to state, and
//  one for client requests/replies:

int main (int argc, char *argv [])
{
    //  Arguments can be either of:
    //      -p  primary server, at tcp://localhost:5001
    //      -b  backup server, at tcp://localhost:5002
    szmq::Context context;
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> statepub(context);
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> statesub(context);
    statesub.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend(context);

    bstar_t fsm;

    if (argc == 2 && (strcmp(argv[1], "-p")==0)) {
        cout << "I: Primary active, waiting for backup (passive)\n";
        frontend.bind(szmq::SocketUrl("tcp://*:5001"));
        statepub.bind(szmq::SocketUrl("tcp://*:5003"));
        statesub.connect(szmq::SocketUrl("tcp://localhost:5004"));
        fsm.state = STATE_PRIMARY;
    }
    else
    if (argc == 2 && (strcmp(argv[1], "-b")==0)) {
        cout << "I: Backup passive, waiting for primary (active)\n";
        frontend.bind(szmq::SocketUrl("tcp://*:5002"));
        statepub.bind(szmq::SocketUrl("tcp://*:5004"));
        statesub.connect(szmq::SocketUrl("tcp://localhost:5003"));
        fsm.state = STATE_BACKUP;
    }
    else {
        cout << "Usage: bstarsrv { -p | -b }\n";
        frontend.close();
        statepub.close();
        statesub.close();
        exit (0);
    }
    //  .split handling socket input
    //  We now process events on our two input sockets, and process these
    //  events one at a time via our finite-state machine. Our "work" for
    //  a client request is simply to echo it back:

    //  Set timer for next outgoing state message
    uint64_t send_state_at = szmq::now() + HEARTBEAT;
    while (true) {
        std::vector<szmq::PollItem> items = {
            {reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*statesub), 0, ZMQ_POLLIN, 0}};

        uint64_t time_left = send_state_at - szmq::now();
        if (time_left < 0)
            time_left = 0;
        szmq::poll (items, 2, time_left);
        if (items [0].revents & ZMQ_POLLIN) {
            //  Have a client request
            auto msgs = frontend.recvMultiple();
            for (auto it = begin (msgs); it != end (msgs); ++it) 
                    it->dump();  
            fsm.event = CLIENT_REQUEST;
            if (s_state_machine (&fsm) == true){
                cout << "State Transition Error : " << fsm.event << endl;
            } else         
                frontend.sendMultiple(msgs);  //  Answer client by echoing request back
        }
        if (items [1].revents & ZMQ_POLLIN) {
            //  Have state from our peer, execute as event
            auto message = statesub.recvOne().read<int>();                    
            fsm.event = (event_t)(message);
            cout << "Event from Peer : " << fsm.event << endl;    
            if (s_state_machine (&fsm)) {
                cout << "State Transition Error : " << fsm.event << endl;
                break; //  Error, so exit
            }
            fsm.peer_expiry = szmq::now() + 2 * HEARTBEAT;
        }
        //  If we timed out, send state to peer
        if (szmq::now() >= send_state_at) {
            statepub.sendOne(szmq::Message::from((int)fsm.state));
            send_state_at = szmq::now() + HEARTBEAT;
        }
    }
    //  Shutdown sockets and context
    frontend.close();
    statepub.close();
    statesub.close();
    return 0;
}
```

클라이언트 코드는 다음과 같습니다.

### bstarcli.java: 바이너리 스타 클라이언트

```java
//  Binary Star client proof-of-concept implementation. This client does no
//  real work; it just demonstrates the Binary Star failover model.

#include <szmq/szmq.h>
#include <iostream>
#include <thread>
#include <boost/format.hpp>
using namespace std;
#define REQUEST_TIMEOUT     1000    //  msecs
#define SETTLE_DELAY        2000    //  Before failing over

szmq::detail::ClientSocketImpl& s_client_socket(szmq::Context *context, std::string address)
{
    szmq::detail::ClientSocketImpl *client = new szmq::detail::ClientSocketImpl(ZMQ_REQ, szmq::ZMQ_CLIENT, *context);
    client->connect(szmq::SocketUrl(address));
    return *client;
}

int main (void)
{
    szmq::Context context;

    std::string server [] = { "tcp://localhost:5001", "tcp://localhost:5002" };
    int server_nbr = 0;

    cout << "I: connecting to server at " << server[server_nbr] <<"...\n";
    szmq::detail::ClientSocketImpl& client = s_client_socket(&context, server[server_nbr]);

    int sequence = 0;
    while (true) {
        //  We send a request, then we work to get a reply
        auto request = boost::str(boost::format("%1%") % ++sequence); 
        client.sendOne(szmq::Message::from(request));

        int expect_reply = 1;
        while (expect_reply) {
            //  Poll socket for a reply, with timeout
            std::vector<szmq::PollItem> items = {
                {reinterpret_cast<void*>(*client), 0, ZMQ_POLLIN, 0}};
            szmq::poll (items,1, REQUEST_TIMEOUT);
            //  .split main body of client
            //  We use a Lazy Pirate strategy in the client. If there's no
            //  reply within our timeout, we close the socket and try again.
            //  In Binary Star, it's the client vote that decides which
            //  server is primary; the client must therefore try to connect
            //  to each server in turn:
            
            if (items [0].revents & ZMQ_POLLIN) {
                //  We got a reply from the server, must match sequence
                auto reply = client.recvOne().read<std::string>();
                if (stoi(reply) == sequence) {
                    cout  << "I: server replied OK (" << reply << ")\n";
                    expect_reply = 0;
                    std::this_thread::sleep_for(std::chrono::milliseconds(1000));   //  One request per second
                }
                else
                    cout  << "E: bad reply from server: (" << reply << ")\n";
            } else {
                cout << "W: no response from server, failing over\n";
                //  Old socket is confused; close it and open a new one			    
                server_nbr = (server_nbr + 1) % 2; 
                std::this_thread::sleep_for(std::chrono::milliseconds(SETTLE_DELAY)); 
                cout << "I: connecting to server at " << server[server_nbr] << "...\n";
                client = std::move(s_client_socket(&context, server[server_nbr]));
                client.sendOne(szmq::Message::from(request));
            }
        }
    }
    client.close();
    return 0;
}
```

바이너리 스타의 테스트를 위하여 서버들과 클라이언트를 시작합니다. 시작하는 순서는 어느 쪽이 먼저라도 상관없습니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc bstarsrv.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc bstarcli.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./bstarsrv -p
I: Primary active, waiting for backup (passive)
Event from Peer : 2
I: connected to backup (passive), ready active
Event from Peer : 4
Event from Peer : 4
[005]0080000029
[000]
[001]1
Event from Peer : 4
[005]0080000029
[000]
[001]2
Event from Peer : 4
[005]0080000029
[000]
[001]3
...

PS D:\work\sook\src\szmq\examples> ./bstarsrv -b
I: Backup passive, waiting for primary (active)
Event from Peer : 1
Event from Peer : 3
I: connected to primary (active), ready passive
Event from Peer : 3
[005]0080000029
[000]
[001]6
I: failover successful, ready active
[005]0080000029
[000]
[001]7
[005]0080000029
[000]
[001]8
[005]0080000029
...

PS D:\work\sook\src\szmq\examples> ./bstarcli
I: connecting to server at tcp://localhost:5001...
I: server replied OK (1)
I: server replied OK (2)
I: server replied OK (3)
I: server replied OK (4)
I: server replied OK (5)
W: no response from server, failing over
[ERROR] zmq_disconnect() : No such file or directory
I: connecting to server at tcp://localhost:5002...
I: server replied OK (6)
I: server replied OK (7)
I: server replied OK (8)
I: server replied OK (9)
...
~~~

바이너리 스타는 유한 상태 머신에 의해 구동됩니다. 발생하는 이벤트들은 상대 상태이며 “Peer Active”은 다른 서버가 활성 상태라는 것을 알려 줍니다. “Client Request”은 서버가 클라이언트 요청을 받았음을 의미합니다. “Client Vote”는 비활성 상태의 서버가 클라이언트 요청을 받아 활성화 상태로 전환되거나, 상대가 2개의 심박을 받는 동안 비활성 상태임을 의미합니다.

서버는 상태 변경을 위해 PUB-SUB 소켓을 사용합니다. 다른 소켓 조합은 여기서 동작하지 않습니다. PUSH 및 DEALER 소켓은 메시지 수신 준비가 된 상대가 없는 경우 차단해 버립니다. PAIR 소켓은 상대가 사라지고 복귀하면 다시 연결되지 않습니다. ROUTER 소켓은 메시지를 보내기 전에 상대의 주소가 필요합니다.

### 바이너리 스타 리엑터

바이너리 스타는 재사용 가능한 리엑터(reactor) 클래스로 만들면 유용하고 범용적입니다. 리엑터는 처리할 메시지가 있을 때마다 코드를 실행하고 호출합니다. 각 서버에 기능이 필요할 때마다 바이너리 스타 코드를 복사/붙여 넣기 하는 것보다 리엑터 API를 호출하는 것이 쉽습니다.

C 개발언어에서는 CZMQ 라이브러리의 zloop 클래스를 사용합니다. zloop는 소켓 및 타이머 이벤트에 반응하는 핸들러를 등록할 수 있습니다. 바이너리 스타 리엑터에서 클라이언트 요청(CLIENT_REQUEST)과 이벤트에 따른 상태 변경(ACTIVE -> PASSIVE 등)을 위한 핸들러를 제공합니다. 다음은 바이너리 스타 리엑터를 위한 API(bstar API)입니다.

### bstar.h: 바이너리 스타 핵심 클래스

```java
/*  =====================================================================
 *  bstar - Binary Star reactor
 *  ===================================================================== */

#ifndef __BSTAR_H_INCLUDED__
#define __BSTAR_H_INCLUDED__

#include <szmq/szmq.h>
#include <czmq.h>
#include <iostream>
using namespace std;

//  Arguments for constructor
#define BSTAR_PRIMARY   1
#define BSTAR_BACKUP    0

//  We send state information every this often
//  If peer doesn't respond in two heartbeats, it is 'dead'
#define BSTAR_HEARTBEAT     1000        //  In msecs

//  States we can be in at any point in time
typedef enum {
    STATE_PRIMARY = 1,          //  Primary, waiting for peer to connect
    STATE_BACKUP = 2,           //  Backup, waiting for peer to connect
    STATE_ACTIVE = 3,           //  Active - accepting connections
    STATE_PASSIVE = 4           //  Passive - not accepting connections
} state_t;

//  Events, which start with the states our peer can be in
typedef enum {
    PEER_PRIMARY = 1,           //  HA peer is pending primary
    PEER_BACKUP = 2,            //  HA peer is pending backup
    PEER_ACTIVE = 3,            //  HA peer is active
    PEER_PASSIVE = 4,           //  HA peer is passive
    CLIENT_REQUEST = 5          //  Client makes request
} event_t;

class bstar;

//  Binary Star finite state machine (applies event to state)
static int s_execute_fsm (bstar *self);
// Update peer expiry time
static void s_update_peer_expiry (bstar *self);
//  Publish state to peer
int s_send_state (zloop_t *loop, int timer_id, void *arg);
//  Receive state from peer, execute finite state machine
int s_recv_state (zloop_t *loop, zmq_pollitem_t *poller, void *arg);
//  Application wants to speak to us, see if it's possible
int s_voter_ready (zloop_t *loop, zmq_pollitem_t *poller, void *arg);

class bstar {
public :
    //  Create a new Binary Star instance, using local (bind) and
    //  remote (connect) endpoints to set-up the server peering.
    bstar(szmq::Context& context, int primary, std::string local, std::string remote, bool verbose)
        : ctx(context),
          statepub(context),
          statesub(context),
          frontend(context),
          verbose(verbose){
        //  Initialize the Binary Star
        active_fn = NULL;
        voter_fn = NULL;
        passive_fn= NULL;
        loop = zloop_new();
        state = primary? STATE_PRIMARY : STATE_BACKUP;
        //  Create publisher for state going to peer
        statepub.bind(szmq::SocketUrl(local));
        //  Create subscriber for state coming from peer
        statesub.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
        statesub.connect(szmq::SocketUrl(remote));
        //  Set-up basic reactor events
        zloop_timer (loop, BSTAR_HEARTBEAT, 0, s_send_state, this);
        zmq_pollitem_t poller = {reinterpret_cast<void*>(*statesub), 0, ZMQ_POLLIN };
        zloop_poller (loop, &poller, s_recv_state, this);
    }
    //  Return underlying zloop reactor, for timer and reader
    //  registration and cancelation.
    zloop_t *zloop (){
        return loop;
    }

    //  This method registers a client voter socket. Messages received
    //  on this socket provide the CLIENT_REQUEST events for the Binary Star
    //  FSM and are passed to the provided application handler. We require
    //  exactly one voter per {{bstar}} instance:
    int voter (std::string endpoint, zloop_fn handler, void *arg){
        if (verbose) zclock_log("+++++ voter()");
        //  Hold actual handler+arg so we can call this later
        frontend.bind(szmq::SocketUrl(endpoint));
        //assert (!voter_fn);
        voter_fn = handler;
        voter_arg = arg;
        zmq_pollitem_t poller = { reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN };
        if (verbose) zclock_log("----- voter()");
        return zloop_poller (loop, &poller, s_voter_ready, this);
    }

    //  Register handlers to be called each time there's a state change:
    void newActive (zloop_fn handler, void *arg){
        if (verbose) zclock_log("+++++ newActive()");
        assert (!active_fn);
        active_fn = handler;
        active_arg = arg;
        if (verbose) zclock_log("----- newActive()");
    }
    void newPassive (zloop_fn handler, void *arg){
        if (verbose) zclock_log("+++++ newPassive()");
        assert (!passive_fn);
        passive_fn = handler;
        passive_arg = arg;
        if (verbose) zclock_log("----- newPassive()");
    }

    //  Register main state change handlers
    void newMaster (zloop_fn handler, void *arg){
        if (verbose) zclock_log("+++++ newMaster()");
        assert (!active_fn);
        active_fn = handler;
        active_arg = arg;
        if (verbose) zclock_log("----- newMaster()");
    }
    void newSlave (zloop_fn handler, void *arg){
        if (verbose) zclock_log("+++++ newSlave()");
        assert (!passive_fn);
        passive_fn = handler;
        passive_arg = arg;
        if (verbose) zclock_log("----- newSlave()");
    }

    //  Enable/disable verbose tracing
    void setVerbose (){
        zloop_set_verbose (loop, verbose);
    }

    //  Start the reactor, ends if a callback function returns -1, or the
    //  process received SIGINT or SIGTERM.
    int start (){
        if (verbose) zclock_log("+++++ start()");
        assert (voter_fn);
        s_update_peer_expiry (this);
        if (verbose) zclock_log("----- start()");
        return zloop_start (loop);
    }
    //  Destroy a Binary Star instance
    ~bstar(){
        zloop_destroy (&loop);
        statepub.close();
        statesub.close();
    };

    zloop_t *loop;              //  Reactor loop
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> statepub;             //  State publisher
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> statesub;             //  State subscriber
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> frontend;             //  State subscriber
    state_t state;              //  Current state
    event_t event;              //  Current event
    uint64_t peer_expiry;        //  When peer is considered 'dead'
    zloop_fn *voter_fn;         //  Voting socket handler
    void *voter_arg;            //  Arguments for voting handler
    zloop_fn *active_fn;        //  Call when become active
    void *active_arg;           //  Arguments for handler
    zloop_fn *passive_fn;         //  Call when become passive
    void *passive_arg;            //  Arguments for handler
    bool verbose;
private :     
    szmq::Context& ctx;                //  Our private context

};


//  Binary Star finite state machine (applies event to state)
//  Returns -1 if there was an exception, 0 if event was valid.
static int
s_execute_fsm (bstar *self)
{
    int rc = 0;
    //  Primary server is waiting for peer to connect
    //  Accepts CLIENT_REQUEST events in this state
    if (self->state == STATE_PRIMARY) {
        if (self->event == PEER_BACKUP) {
            zclock_log ("I: connected to backup (passive), ready as active");
            self->state = STATE_ACTIVE;
            if (self->active_fn)
                (self->active_fn) (self->loop, NULL, self->active_arg);
        }
        else
        if (self->event == PEER_ACTIVE) {
            zclock_log ("I: connected to backup (active), ready as passive");
            self->state = STATE_PASSIVE;
            if (self->passive_fn)
                (self->passive_fn) (self->loop, NULL, self->passive_arg);
        }
        else
        if (self->event == CLIENT_REQUEST) {
            // Allow client requests to turn us into the active if we've
            // waited sufficiently long to believe the backup is not
            // currently acting as active (i.e., after a failover)
            assert (self->peer_expiry > 0);
            if (zclock_time () >= self->peer_expiry) {
                zclock_log ("I: request from client, ready as active");
                self->state = STATE_ACTIVE;
                if (self->active_fn)
                    (self->active_fn) (self->loop, NULL, self->active_arg);
            } else
                // Don't respond to clients yet - it's possible we're
                // performing a failback and the backup is currently active
                rc = -1;
        }
    }
    else
    //  Backup server is waiting for peer to connect
    //  Rejects CLIENT_REQUEST events in this state
    if (self->state == STATE_BACKUP) {
        if (self->event == PEER_ACTIVE) {
            zclock_log ("I: connected to primary (active), ready as passive");
            self->state = STATE_PASSIVE;
            if (self->passive_fn)
                (self->passive_fn) (self->loop, NULL, self->passive_arg);
        }
        else
        if (self->event == CLIENT_REQUEST)
            rc = -1;
    }
    else
    //  Server is active
    //  Accepts CLIENT_REQUEST events in this state
    //  The only way out of ACTIVE is death
    if (self->state == STATE_ACTIVE) {
        if (self->event == PEER_ACTIVE) {
            //  Two actives would mean split-brain
            zclock_log ("E: fatal error - dual actives, aborting");
            rc = -1;
        }
    }
    else
    //  Server is passive
    //  CLIENT_REQUEST events can trigger failover if peer looks dead
    if (self->state == STATE_PASSIVE) {
        if (self->event == PEER_PRIMARY) {
            //  Peer is restarting - become active, peer will go passive
            zclock_log ("I: primary (passive) is restarting, ready as active");
            self->state = STATE_ACTIVE;
        }
        else
        if (self->event == PEER_BACKUP) {
            //  Peer is restarting - become active, peer will go passive
            zclock_log ("I: backup (passive) is restarting, ready as active");
            self->state = STATE_ACTIVE;
        }
        else
        if (self->event == PEER_PASSIVE) {
            //  Two passives would mean cluster would be non-responsive
            zclock_log ("E: fatal error - dual passives, aborting");
            rc = -1;
        }
        else
        if (self->event == CLIENT_REQUEST) {
            //  Peer becomes active if timeout has passed
            //  It's the client request that triggers the failover
            assert (self->peer_expiry > 0);
            if (zclock_time () >= self->peer_expiry) {
                //  If peer is dead, switch to the active state
                zclock_log ("I: failover successful, ready as active");
                self->state = STATE_ACTIVE;
            }
            else
                //  If peer is alive, reject connections
                rc = -1;
        }
        //  Call state change handler if necessary
        if (self->state == STATE_ACTIVE && self->active_fn)
            (self->active_fn) (self->loop, NULL, self->active_arg);
    }
    return rc;
}

static void
s_update_peer_expiry (bstar *self)
{
    self->peer_expiry = zclock_time () + 2 * BSTAR_HEARTBEAT;
}

//  Reactor event handlers...

//  Publish our state to peer
int s_send_state (zloop_t *loop, int timer_id, void *arg)
{
    bstar *self = (bstar *) arg;
    self->statepub.sendOne(szmq::Message::from(to_string(self->state)));
    return 0;
}

//  Receive state from peer, execute finite state machine
int s_recv_state (zloop_t *loop, zmq_pollitem_t *poller, void *arg)
{
    bstar *self = (bstar *) arg;
    std::string state = self->statesub.recvOne().read<std::string>();
    if (state.length() > 0) {
        self->event = (event_t)stoi(state);
        s_update_peer_expiry (self);
    }
    return s_execute_fsm (self);
}

//  Application wants to speak to us, see if it's possible
int s_voter_ready (zloop_t *loop, zmq_pollitem_t *poller, void *arg)
{
    bstar *self = (bstar *)arg;
    //  If server can accept input now, call appl handler
    self->event = CLIENT_REQUEST;
    if (s_execute_fsm (self) == 0)
        (self->voter_fn) (self->loop, poller, self->voter_arg);
    else {
        //  Destroy waiting message, no-one to read it
        auto msg = self->statesub.recvOne();
    }
    return 0;
}

#endif
```

추상화된 클래스 통하여 바이너리 스타 서버는 짦은 코드로 작성 가능합니다.

### bstarsrv2.java : 바이너리 스타 서버(핵심 클래스(bstar) 사용)
```java
//  Binary Star server, using bstar reactor

//  Lets us build this source without creating a library
#include "bstar.h"
#include <szmq/szmq.h>
#include <czmq.h>
#include <iostream>
using namespace std;

//  Echo service to client
int s_echo (zloop_t *loop, zmq_pollitem_t *poller, void *arg)
{
    bstar *self = (bstar *)arg;
    auto msgs = self->frontend.recvMultiple();
    for (auto it = begin (msgs); it != end (msgs); ++it) 
        it->dump(); 
    self->frontend.sendMultiple(msgs);
    return 0;
}
int main (int argc, char *argv [])
{
    bool verbose = true;
    szmq::Context context;
    //  Arguments can be either of:
    //      -p  primary server, at tcp://localhost:5001
    //      -b  backup server, at tcp://localhost:5002
    bstar *self;
    if (argc > 1 && strcmp (argv [1], "-p") == 0) {
        cout << "I: Primary active, waiting for backup (passive)\n";
        self = new bstar(context, BSTAR_PRIMARY,
            "tcp://*:5003", "tcp://localhost:5004", verbose);
        self->voter ("tcp://*:5001", s_echo, (void *)self);
    }
    else
    if (argc > 1 && streq (argv [1], "-b")) {
        cout <<"I: Backup passive, waiting for primary (active)\n";
        self = new bstar(context, BSTAR_BACKUP,
            "tcp://*:5004", "tcp://localhost:5003", verbose);
        self->voter ("tcp://*:5002", s_echo, (void *)self);
    }
    else {
        cout << "Usage: bstarsrvs { -p | -b }\n";
        exit (0);
    }
    self->start();
    delete(self);
    return 0;
}
```

* 클라이언트는 이전에 작성한 bstarcli를 변경 없이 사용합니다.

### 빌드 및 테스트(원도우 및 리눅스에서 정상 동작)

PS D:\work\sook\src\szmq\examples> cl -EHsc bstarsrv2.java szmq.lib czmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc bstarcli.java szmq.lib

~~~{.bash}
PS D:\work\sook\src\szmq\examples> ./bstarsrv2 -p
I: Primary active, waiting for backup (passive)
20-11-17 13:32:21 +++++ voter()
20-11-17 13:32:21 ----- voter()
20-11-17 13:32:21 +++++ start()
20-11-17 13:32:21 ----- start()
20-11-17 13:32:24 I: request from client, ready as active
[005]0080000029
[000]
[001]1
[005]0080000029
[000]
[001]2
...
PS D:\work\sook\src\szmq\examples> ./bstarcli -v
I: connecting to server at tcp://localhost:5001...
I: server replied OK (1)
I: server replied OK (2)
...

~~~

## 브로커 없는 신뢰성(프리랜서 패턴)

ØMQ를 "브로커 없는 메시징"이라고 설명하면서, 브로커 기반 신뢰성에 너무 집중하는 것은 모순처럼 보입니다. 메시징에서 실생활에서와 같이 중개인(middleman)은 부담이자 장점입니다. 실제로 대부분의 메시징 아키텍처는 분산과 중개 메시징을 혼합하여 각 메시징의 장점을 얻을 수 있습니다. 각 메시징의 장단점을 자유롭게 선택할 수 있을 때 최상의 결과를 얻을 수 있습니다. 
마치 혼자서 저녁 식사를 위한 와인 1병을 사기 위해 10분 걸어서 편의점에 가는 것과, 친구들과의 파티를 위해 5종류의 와인을 사려 20분 정도 운전해서 홈플러스까지 가는 이유와 같습니다.  
현실 세계 경제에 기본은 소요되는 시간, 에너지 및 비용에 민감하며 상대적 평가를 통해 선택되며, 최적의 메시지 기반 아키텍처에도 마찬가지입니다.

이것이 ØMQ가 브로커 중심 아키텍처를 강요하지 않는 이유입니다. ØMQ는 브로커(일명 프록시)를 구축할 수 있는 도구를 주어 지금까지 12개 정도 여러 패턴들을 구축했습니다.
  - REQ-REP
  - PUB-SUB
  - REQ-ROUTER
  - DEALER-REP
  - DEALER-ROUTER
  - DEALER-DEALER
  - ROUTER-ROUTER
  - PUSH-PULL
  - PAIR-PAIR
  - LPP : Lazy Pirate Pattern
  - SPP : Simple Pirate Pattern
  - PPP : Paranoid Pirate Pattern
  - MDP : Majordomo Pattern

따라서 이장을 마무리하면서 우리는 지금까지 만들어온 브로커 기반 신뢰성을 해체하고 프리랜스 패턴이라고 불리는 분산형 P2P(peer-to-peer) 아키텍처로 돌아가겠습니다.
P2P의 핵심 사용 사례는 이름 확인 서비스(name resolution service)입니다. 이것은 ØMQ 아키텍처의 공통적인 문제입니다 : 연결할 단말을 어떻게 아시나요? 코드상 하드 코딩된 TCP/IP 주소는 매우 취약하기 때문에 주소를 구성 파일에 넣게 되면 관리가 끔찍해집니다. 모든 PC 또는 휴대폰의 웹브라우저에서 "google.com"이 "74.125.230.82"라는 것을 알기 위해 구성 파일을 직접 만들어야 한다고 생각해 보십시오.

여기서 구현하는 ØMQ 이름 서비스는 다음의 기능을 수행해야 합니다 :

* 논리적 이름으로 바인딩 단말과 연결 단말로 확인합니다. 현실적인 이름 서비스는 다중 바인딩 단말들을 제공하고 가능하면 다중 연결 단말들도 제공합니다.
* 다중 병렬 환경들(예 : "테스트", "생산")을 코드 수정 없이 관리할 수 있습니다.
* 신뢰성이 보장돼야 합니다. 이름 서비스를 사용할 수 없으면 응용프로그램이 네트워크에 연결할 수 없기 때문입니다.

서비스 지향 MDP 브로커 뒤에 이름 서비스를 배치하는 것은 현명한 생각이지만, 클라이언트가 직접 연결 가능한 이름 서비스 서버로 노출하는 것이 훨씬 간단합니다. 올바른 작업을 위하여 이름 서비스가 글로벌 네트워크 단말이 되기 위해서 하드 코딩된 소스코드 또는 구성 파일이 필요합니다.

처리하려는 장애 유형은 서버 충돌 및 재시작, 서버의 과도한 루프, 서버 과부하, 네트워크 문제입니다. 안정성을 확보하기 위해 이름 서버들의 저장소를 생성하여 서버 하나가 충돌하거나 사라지면, 클라이언트가 다른 서버에 연결하게 합니다. 실제로는 저장소은 2개면 충분하지만 예제에서는 저장소의 크기는 제한이 없다고 가정합니다.

이 아키텍처에서는 대규모 집합의 클라이언트들이 소규모 서버들에 직접 연결됩니다. 서버들은 각각의 주소로 바인딩합니다. 작업자가 브로커에 연결하는 MDP와 같은 브로커 기반 접근 방식과 근본적으로 다릅니다. 클라이언트에는 선택 가능한 몇 가지 옵션들이 있습니다.

* REQ 소켓과 게으른 해적 패턴(응답이 없을 경우 재시도)을 사용하십시오. 쉽지만 추가 기능으로 클라이언트들이 죽은 서버들에 계속해서 재연결을 하지 않게 하는 기능입니다.
* DEALER 소켓을 사용하고 응답을 받을 때까지 여러 개의 요청들을 비동기로 수행합니다(연결된 모든 서버들로 부하 분산됨). 효과적이지만 우아하지는 않습니다.
* 클라이언트가 특정 서버에 주소를 지정할 수 있도록 ROUTER 소켓을 사용합니다. 그러나 어떻게 클라이언트는 서버 소켓의 식별자(ID)를 인식할까요? 서버가 먼저 클라이언트를 핑(ping)하거나(복잡함), 서버가 하드 코딩된 고정 식별자(ID)를 가지고, 사전에 클라이언트가 인지하게 합니다.(서버들 구성이 변경(추가/삭제)될 때마다 클라이언트는 구성 정보 변경 필요하여 사용이 불편합니다.)

다음 절부터는 구현을 위한 개별 방법 설명하겠습니다.

### 모델 1: 간단한 재시도와 장애조치

클라이언트가 선택 가능한 3가지 방법들은 단순하고 무식하고 복잡하고 귀찮게 나타납니다. 우선 간단한 것부터 시작하여 꼬임을 해결해 봅시다. 
게으른 해적 가져와서 다중 서버 단말들에 동작하도록 재작성합니다.

### flserver1.java : 프리랜서 서버, 모델 1

```java
//  Freelance server - Model 1
//  Trivial echo service

#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
using namespace std;

int main (int argc, char *argv [])
{
    if (argc < 2) {
        cout << boost::format("I: syntax: %1% <endpoint>\n") % argv [0];
        return 0;
    }
    szmq::Context ctx;
    szmq::Socket<ZMQ_REP, szmq::ZMQ_SERVER> server(ctx);
    server.bind(szmq::SocketUrl(argv[1]));

    cout << "I: echo service is ready at " << argv [1] << endl;
    while (true) {
        auto msg = server.recvOne();
        if(msg.size()==0)
            break;          //  Interrupted
        server.sendOne(msg);
    }
    server.close();
    return 0;
}
```

그런 다음 하나 이상의 단말을 인수로 지정하여 클라이언트를 시작합니다.

### flclient1.java : 프리랜서 클라이언트, 모델 1

```java
//  Freelance client - Model 1
//  Uses REQ socket to query one or more services

#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
using namespace std;
#define REQUEST_TIMEOUT     1000
#define MAX_RETRIES         3       //  Before we abandon

static szmq::Message 
s_try_request (szmq::Context *ctx, std::string endpoint, szmq::Message request)
{
    cout << "I: trying echo service at "<< endpoint << endl;
    szmq::Context *context = ctx;
    szmq::Socket<ZMQ_REQ, szmq::ZMQ_CLIENT> client(*context);
    client.connect(szmq::SocketUrl(endpoint));

    //  Send request, wait safely for reply
    client.sendOne(request);
    std::vector<szmq::PollItem> items = {
		{reinterpret_cast<void*>(*client), 0, ZMQ_POLLIN, 0}};
    szmq::Message reply;
    szmq::poll(items, 1, REQUEST_TIMEOUT);
    if (items [0].revents & ZMQ_POLLIN)
        reply = client.recvOne();

    //  Close socket in any case, we're done with it now
    client.close();
    return reply;
}

//  .split client task
//  The client uses a Lazy Pirate strategy if it only has one server to talk
//  to. If it has two or more servers to talk to, it will try each server just
//  once:

int main (int argc, char *argv [])
{
    szmq::Context ctx;

    szmq::Message request(szmq::Message::from("Hello world"));
    szmq::Message reply;

    int endpoints = argc - 1;
    if (endpoints == 0)
        cout << boost::format("I: syntax: %1% <endpoint> ...\n") % argv [0];
    else
    if (endpoints == 1) {
        //  For one endpoint, we retry N times
        int retries;
        for (retries = 0; retries < MAX_RETRIES; retries++) {
            char *endpoint = argv [1];
            reply = s_try_request (&ctx, endpoint, request);
            if (reply.size() > 0)
                break;          //  Successful
            cout << "W: no response from " << endpoint << ", retrying...\n";
        }
    }
    else {
        //  For multiple endpoints, try each at most once
        int endpoint_nbr;
        for (endpoint_nbr = 0; endpoint_nbr < endpoints; endpoint_nbr++) {
            char *endpoint = argv [endpoint_nbr + 1];
            reply = s_try_request (&ctx, endpoint, request);
            if (reply.size() > 0)
                break;          //  Successful
            cout << "W: no response from " << endpoint << endl;
        }
    }
    if (reply.size() > 0)
        printf ("Service is running OK\n");
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc flserver1.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc flclient1.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./flserver1 tcp://*:5555
I: echo service is ready at tcp://*:5555

PS D:\work\sook\src\szmq\examples> ./flserver1 tcp://*:5556
I: echo service is ready at tcp://*:5556

I: trying echo service at tcp://localhost:5555...
Service is running OK

PS D:\work\sook\src\szmq\examples> ./flclient1 tcp://localhost:5555 tcp://localhost:5556
I: trying echo service at tcp://localhost:5555...
Service is running OK

PS D:\work\sook\src\szmq\examples> ./flclient1 tcp://localhost:5557
I: trying echo service at tcp://localhost:5557...
W: no response from tcp://localhost:5557, retrying...
I: trying echo service at tcp://localhost:5557...
W: no response from tcp://localhost:5557, retrying...
I: trying echo service at tcp://localhost:5557...
W: no response from tcp://localhost:5557, retrying...
~~~

기본 접근 방식은 게으른 해적이지만, 클라이언트는 하나의 성공적인 응답만 받는 것을 목표로 합니다. 클라이언트 프로그램은 단일 서버 혹은 여러 서버들을 실행하는지에 따라 2가지 처리 방식이 있습니다.

* 단일 서버에서 클라이언트는 게으른 해적과 동일하게 수차례(3번) 재시도합니다.
* 여러 서버에서 클라이언트는 서버들의 응답을 받기 위해 최대 1번 시도합니다.  

그러나 설계에는 단점이 있습니다. 클라이언트에서 많은 소켓을 연결하고 기본 이름 서버가 죽으면 각 클라이언트는 고통스러운 제한시간으로 인한 지연을 경험합니다.

### 모델 2: 잔인한 엽총 학살

두 번째 옵션으로 클라이언트를 DEALER 소켓을 사용하도록 전환하겠습니다. 
여기서 우리의 목표는 특정 서버의 상태(죽고, 살고)에 관계없이 가능한 한 빨리 응답을 받는 것입니다.
클라이언트는 다음과 같은 접근 방식을 가집니다.

* 클라이언트가 준비가 되면, 모든 서버들에 연결합니다.
* 요청이 있을 때, 서버들의 대수만큼 요청을 보냅니다.
* 클라이언트는 첫 번째 응답을 기다렸다가 받아들입니다.
* 다른 서버들의 응답들은 무시합니다.

서버가 기동 중에 실제로 일어날 일은 ØMQ가 클라이언트의 요청들을 분배하여 각 서버들이 하나의 요청을 받고 하나의 응답을 보내도록 합니다. 서버가 오프라인이고 연결이 끊어지면, ØMQ는 다른 나머지 서버로 요청을 분배합니다. 따라서 서버는 경우에 따라 동일한 요청을 두 번 이상 받을 수 있습니다.

클라이언트는 여러 번의 응답들을 받지만 정확한 횟수로 응답되었는지를 보장할 수 없습니다. 요청들 및 응답들은 유실될 수 있습니다(예 : 요청을 처리하는 동안 서버가 죽는 경우).

따라서 요청들에 번호를 매겨서 요청 번호와 일치하지 않는 응답은 무시합니다. 모델 1(간단한 재시도와 장애조치) 서버는 에코 서버라서 작동하지만 우연은 필연이 아니기에 이해를 위한 좋은 기초가 아닙니다. 따라서 모델 2 서버를 만들어 요청에 대한 정확한 응답 번호와 “OK”로 반환하게 합니다. 메시지들은 2개의 부분으로 구성되어 있습니다 : 시퀀스 번호(응답 번호)와 본문(OK)

하나 이상의 서버들을 시작할 때, 바인딩할 단말을 지정합니다.

### flserver2.java : 프리랜서 서버, 모델2

```java
//  Freelance server - Model 2
//  Does some work, replies OK, with message sequencing

#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
using namespace std;

int main (int argc, char *argv [])
{
    if (argc < 2) {
        cout << boost::format("I: syntax: %1% <endpoint>\n") % argv [0];
        return 0;
    }
    szmq::Context ctx;
    szmq::Socket<ZMQ_REP, szmq::ZMQ_SERVER> server(ctx);
    server.bind(szmq::SocketUrl(argv[1]));


    cout << "I: echo service is ready at " << argv [1] << endl;
    while (true) {
        auto request = server.recvMultiple();
        if(request.size()==0)
            break;          //  Interrupted
        //  Fail nastily if run against wrong client
        assert (request.size() == 2);
        auto sequence = request.front();
        request.clear();

        vector <szmq::Message> reply;
        reply.emplace_back(sequence);
        reply.emplace_back(szmq::Message::from("OK"));
        server.sendMultiple(reply);
    }
    server.close();
    return 0;
}
```

### flclient.h: 프리랜서 클라이언트 클래스, 모델2

```java
//  Freelance client - Model 2
//  Uses DEALER socket to blast one or more services
#ifndef __FLCLIENT_H_INCLUDED__
#define __FLCLIENT_H_INCLUDED__
#include <szmq/szmq.h>
#include <czmq.h>
#include <iostream>
#include <boost/format.hpp>
using namespace std;
//  If not a single service replies within this time, give up
#define GLOBAL_TIMEOUT 2500

//  .split class implementation
//  Here is the {{flclient}} class implementation. Each instance has a 
//  context, a DEALER socket it uses to talk to the servers, a counter 
//  of how many servers it's connected to, and a request sequence number:
class flclient {
public :
    flclient (szmq::Context& context) noexcept
        : ctx(context),
          socket(context){
              servers = 0;
              sequence= 0;
    }
    ~flclient(){};
    void connect(std::string endpoint ) noexcept {
        cout << boost::format("I: connecting %1% ...\n") % endpoint;
        socket.connect(szmq::SocketUrl(endpoint));
        servers++;
    }
    
    vector<szmq::Message> 
    request (vector<szmq::Message> request_)
    {
        vector<szmq::Message> request = request_;

        //  Prefix request with sequence number and empty envelope
        std::string sequence_text = boost::str(boost::format("%1%") % ++sequence);
        request.insert(request.begin(), szmq::Message::from(sequence_text));
        request.insert(request.begin(), szmq::Message::from(std::string("")));

        //  Blast the request to all connected servers
        int server;
        for (server = 0; server < servers; server++) {
            socket.sendMultiple(request);
        }
        //  Wait for a matching reply to arrive from anywhere
        //  Since we can poll several times, calculate each one
        vector<szmq::Message> reply;
        uint64_t endtime = szmq::now() + GLOBAL_TIMEOUT;
        while (szmq::now() < endtime) {
            std::vector<szmq::PollItem> items = {
                {reinterpret_cast<void*>(*socket), 0, ZMQ_POLLIN, 0}};
            szmq::poll(items, 1, (endtime - szmq::now()));
            if (items [0].revents & ZMQ_POLLIN) {
                //  Reply is [empty][sequence][OK]
                reply = socket.recvMultiple();
                assert (reply.size() == 3);
                reply.erase(reply.begin()); // delete the empty delimiter	
                auto seq = reply.front().read<std::string>();
                reply.erase(reply.begin()); // delete the sequence	
                int sequence_nbr = stoi(seq);
                if (sequence_nbr == sequence)
                    break;
                reply.clear();
            }
        }
        return reply;
    }

private :
    szmq::Context& ctx;        //  Our context wrapper
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> socket;     //  DEALER socket talking to servers
    size_t servers;     //  How many servers we have connected to
    uint sequence;      //  Number of requests ever sent
};

#endif
```

클라이언트를 시작하며, 연결한 단말들을 인수로 지정합니다.

### flclient2.java : 프리랜서 클라이언트, 모델2

```java
//  Freelance client - Model 2
//  Uses DEALER socket to blast one or more services

#include "flclient.h"
#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>


int main (int argc, char *argv [])
{
    if (argc == 1) {
        cout << boost::format("I: syntax: %1% <endpoint> ...\n") % argv [0];
        return 0;
    }
    szmq::Context context;
    //  Create new freelance client object
    flclient client(context);

    //  Connect to each endpoint
    int argn;
    for (argn = 1; argn < argc; argn++)
        client.connect (argv [argn]);

    //  Send a bunch of name resolution 'requests', measure time
    int requests = 10000;
    uint64_t start = szmq::now();
    while (requests--) {
        vector<szmq::Message> request;
        request.insert(request.begin(), szmq::Message::from("random name"));
        auto reply = client.request(request);
        if (reply.size()==0) {
            cout << "E: name service not available, aborting\n";
            break;
        }
        reply.clear();
    }
    cout << boost::format("Average round trip cost: %1% usec\n") %
        (int) ((szmq::now() - start) / 10);
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}

*** 1대의 서버를 대상으로 한 경우
PS D:\work\sook\src\szmq\examples> ./flserver2 tcp://*:5560
I: echo service is ready at tcp://*:5560

PS D:\work\sook\src\szmq\examples> ./flclient2 tcp://localhost:5560
I: connecting tcp://localhost:5560 ...
Average round trip cost: 169 usec

*** 2대의 서버를 대상으로 한 경우
PS D:\work\sook\src\szmq\examples> ./flserver2 tcp://*:5560
I: echo service is ready at tcp://*:5560
PS D:\work\sook\src\szmq\examples> ./flserver2 tcp://*:5560
I: echo service is ready at tcp://*:5561

PS D:\work\sook\src\szmq\examples> ./flclient2 tcp://localhost:5560 tcp://localhost:5561
I: connecting tcp://localhost:5560 ...
I: connecting tcp://localhost:5561 ...
Average round trip cost: 171 usec

*** 모든 서버가 죽은 경우
PS D:\work\sook\src\szmq\examples> ./flclient2 tcp://localhost:5562 tcp://localhost:5563
I: connecting tcp://localhost:5562 ...
I: connecting tcp://localhost:5563 ...
E: name service not available, aborting
Average round trip cost: 250 usec
~~~

클라이언트에서 1만 번의 요청-응답 처리 시간(round trip cost)에 대하여 모든 서버가 죽은 경우, 1만 번이 아닌 1회에 대한 요청-응답 처리 시간(round trip cost)이 산정됩니다. 즉 제한시간인 2.5초(2.5초/10,000=0.25 msec=250 usec)가 산정됩니다.

클라이언트 구현에서 주의할 사항들은 다음과 같습니다.

* 클라이언트는 작은 클래스 기반 API로 구성되어 ØMQ 컨텍스트와 소켓을 생성하고 서버와 통신하는 어려운 작업을 은폐하였습니다. 즉, 산탄총이 목표로 발사되어 무수한 산탄이 나가는 것을 클라이언트에서 서버로 "말하기"라고 할 수 있습니다.
* 클라이언트는 몇 초(2.5초) 내에 응답하는 서버를 찾지 못하면 추적을 포기합니다.
* 클라이언트는 DELAER 소켓을 사용하므로, 유효한 REP 봉투(empty delimiter + body)를 만들어야 합니다. 즉, 메시지 앞에 빈 메시지 프레임을 추가해야 합니다.


클라이언트는 10,000개의 이름 확인 요청들(서버가 딱히 아무것도 하지 않는 가짜 요청)을 수행하고 평균 비용(시간)을 측정합니다. 내 테스트 컴퓨터에서 한 서버와 통신하는 데 약 60 마이크로초(usec)가 소요되었습니다. 3대의 서버와 통신하면 약 80 마이크로초(usec)가 소요됩니다.

산탄총 접근에 대한 장단점은 다음과 같습니다.

* 장점 : 단순하고 만들기 쉬우며 이해하기 쉽습니다.
* 장점 : 최소한 하나의 서버가 실행되면 장애조치 작업을 수행하고 빠르게 작동합니다.
* 단점 : 불필요한 네트워크 트래픽을 생성합니다.
* 단점 : 서버들에 대한 우선순위를 지정할 수 없습니다 (예 : Primary, Secondary).
* 단점 : 서버는 한 번에 최대 하나의 요청을 수행할 수 있습니다.

### 모델 3: 복잡하고 불쾌한 방법

샷건 접근 방식이 실현하기에 너무 좋은 것처럼 보입니다. 과학적으로 가능한 대안들을 보도록 하겠습니다. 모델 3(복잡하고 불쾌한 옵션)을 탐구하며, 마침내 실제 적용하기 위해서는 잔인하더라도 복잡하고 불쾌한 방법이 선호되는 것을 깨닫게 됩니다. 이것은 삶의 이야기와 같이 단순하거나 무식한 방법으로 살고 싶지만, 인생에서는 복잡하고 불쾌한 방법으로 해결해야 하는 경우가 많기 때문입니다.

클라이언트의 주요 문제를 ROUTER 소켓으로 전환하여 해결할 수 있습니다. 특정 서버가 죽은 경우 요청 피하는 것이 일반적으로 똑똑한 방법입니다. ROUTER 소켓으로 전환하여 서버가 단일 스레드 같아지는 문제를 해결합니다.

2개의 익명 소켓들(식별자(ID)를 설정하지 않은) 사이에서 ROUTER와 ROUTER 통신을 수행하는 것은 불가능하며, 관리할 또 다른 개념을 만드는 대신 연결 단말을 식별자(ID)로 사용합니다. 이것은 클라이언트/서버 모두에게 고유한 식별자이며 샷건 모델에서 사용한 이력을 통해 양측이 동의할 수 있습니다. 두 개의 ROUTER 소켓을 연결하는 교활하고 효과적인 방법입니다.

ØMQ 식별자(ID)의 동작 방식은 서버 ROUTER 소켓은 식별자(ID)를 ROUTER 소켓에 바인딩하기 전에 설정합니다. 클라이언트가 연결되면 양쪽이 실제 메시지를 보내기 전에 식별자(ID) 교환을 위한 통신을 수행합니다.
* 클라이언트에서 `zmq_setsockopt()`, `zsocket_set_identity()`를 통해 식별자 지정 가능합니다.

식별자(ID)를 설정하지 않은 클라이언트 ROUTER 소켓은 서버에 null 식별자를 보내면 서버는 임의의 UUID를 생성하여 자체적으로 사용할 클라이언트를 지정합니다. 
서버는 자신의 식별자(우리가 사용하기로 한 단말 문자열(예 : tcp://localhost:5556))를 클라이언트에 보냅니다.

클라이언트에서 서버로 연결되면 클라이언트가 메시지를 서버로 전송 가능함을 의미합니다 (클라이언트에서 서버 단말(예 : tcp://localhost:5556)을 식별자(ID)로 지정하여 ROUTER 소켓에서 전송). 그것은 `zmq_connect()`를 수행하고 잠시 시간이 지난 후에 연결됩니다.
여기에 한 가지 문제가 있습니다: 우리는 서버가 실제로 가용하여 연결을 완료할 수 있는 시점을 모릅니다. 서버가 온라인 상태이면 몇 밀리초(msec)가 걸릴 수 있지만 서버가 다운되고 시스템 관리자가 점심을 먹으러 가면 지금부터 한 시간이 걸릴 수도 있습니다.

여기에 작은 역설이 있습니다. 서버가 언제 연결되고 가용한 시점을 알아야 합니다. 프리랜서 패턴에서는 앞부분에서 본 브로커 기반 패턴과 달리 서버가 말을 할 때까지 침묵합니다. 
따라서 서버가 온라인이라고 말할 때까지 서버와 통신할 수 없으며, 우리가 요청하기 전까지는 통신할 수 없습니다.

해결책은 모델 2의 샷건 접근을 약간 혼합하는 것이며, 방법은 인지 가능한 모든 것에 대하여 산탄총을 발사(무해하게)하여 움직이는 것이 있다면 살아 있다는 것을 아는 것입니다. 
우리는 실제 요청을 실행하는 것이 아니라 일종의 핑-퐁(ping-pong) 심박입니다.

이것은 우리를 다시 통신규약 영역으로 가져와서, 프리랜서 클라이언트와 서버가 핑-퐁(ping-pong) 명령과 요청-응답을 수행하는 방법을 정의하는 사양서로 만들었습니다.

- [Freelance Protocol](https://rfc.zeromq.org/spec/10)

서버로 구현하는 것은 짧고 달콤합니다. 다음은 에코 서버입니다.

### flserver3.java: 프리랜서 서버, 모델3

```java
//  Freelance server - Model 3
//  Uses an ROUTER/ROUTER socket but just one thread

#include <szmq/szmq.h>
#include <iostream>
#include <boost/format.hpp>
using namespace std;

int main (int argc, char *argv [])
{
    int verbose = (argc > 1 && (strcmp(argv[1], "-v")==0));
    szmq::Context ctx;
    //  Prepare server socket with predictable identity
    std::string bind_endpoint = "tcp://*:5555";
    std::string connect_endpoint = "tcp://localhost:5555";
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> server(ctx);     
    server.setId(connect_endpoint);
    server.bind(szmq::SocketUrl(bind_endpoint));
    cout << "I: service is ready at " << bind_endpoint << endl;

    while (true) {
        auto request = server.recvMultiple();
        if (verbose) cout << "request\n";
        if(verbose && request.size() > 0){
            for (auto it = begin (request); it != end (request); ++it) 
                it->dump();
        }
        if (request.size()==0)
            break;          //  Interrupted

        //  Frame 0: identity of client
        //  Frame 1: PING, or client control frame
        //  Frame 2: request body
        auto identity = request.front();
        request.erase(request.begin()); // delete the identity
        auto control = request.front();
        request.erase(request.begin()); // delete the control
        vector<szmq::Message> reply;
        if(control.read<std::string>().compare("PING")==0){   
            reply.emplace_back(szmq::Message::from("PONG"));
        }
        else {
            reply.emplace_back(control);
            reply.emplace_back(szmq::Message::from("OK"));
        }
        request.clear();
        reply.insert(reply.begin(), identity);
        if (verbose) cout << "reply\n";
        if (verbose && reply.size() > 0){
            for (auto it = begin (reply); it != end (reply); ++it) 
                it->dump();
        }
        server.sendMultiple(reply);
    }

    server.close();
    return 0;
}
```

그러나 프리랜서 클라이언트는 거대해졌습니다. 명확하게 하기 위해 예제 응용프로그램과 어려운 작업을 수행하는 클래스로 나뉩니다. 다음은 메인 응용프로그램입니다.

### flclient3.java : 프리랜서 클라이언트, 모델 3

```java
//  Freelance client - Model 3
//  Uses flcliapi class to encapsulate Freelance pattern

//  Lets us build this source without creating a library
#include <szmq/szmq.h>
#include <iostream>
#include <thread>
#include <boost/format.hpp>
#include "flclient3.h"
using namespace std;

int main (int argc, char *argv [])
{
    int verbose = (argc > 1 && (strcmp(argv[1], "-v")==0));
    szmq::Context context;
    //  Create new freelance agent object
    flagent agent(context, verbose);
    thread([&](){agent.run();}).detach();

    //  Create new freelance client object
    flclient client(context, verbose);

    //  Connect to several endpoints
    client.connect ("tcp://localhost:5555");
    client.connect ("tcp://localhost:5556");
    client.connect ("tcp://localhost:5557");

    //  Send a bunch of name resolution 'requests', measure time
    int requests = 1000;
    uint64_t start = szmq::now();
    while (requests--) {
        vector<szmq::Message> request;
        request.emplace_back(szmq::Message::from("random name"));
        auto reply = client.request(request);
        if (reply.size() == 0) {
            cout << "E: name service not available, aborting\n";
            break;
        }
        request.clear();
        reply.clear();
    }
    cout << boost::format("Average round trip cost: %1% usec\n") %
        (int)((szmq::now() - start) / 10);
    return 0;
}
```

그리고 여기에 MDP 브로커만큼 복잡하고 거대한 클라이언트 API 클래스가 있습니다.

### flclient3.h: 프리랜서 클라이언트 객체

```java
//  Freelance client - Model 2
//  Uses DEALER socket to blast one or more services
#ifndef __FLCLIENT_H_INCLUDED__
#define __FLCLIENT_H_INCLUDED__
#include <szmq/szmq.h>
#include <czmq.h>
#include <iostream>
#include <thread>
#include <map>
#include <list>
#include <boost/format.hpp>

using namespace std;
//  If no server replies within this time, abandon request
#define GLOBAL_TIMEOUT  2500    //  msecs
//  PING interval for servers we think are alive
#define PING_INTERVAL   2000    //  msecs
//  Server considered dead if silent for this long
#define SERVER_TTL      6000    //  msecs

//  .split API structure
//  This API works in two halves, a common pattern for APIs that need to
//  run in the background. One half is an frontend object our application
//  creates and works with; the other half is a backend "agent" that runs
//  in a background thread. The frontend talks to the backend over an
//  inproc pipe socket:
class flagent;

class server {
public :
    server(szmq::Context& context, szmq::detail::ClientSocketImpl& socket_, string endpoint, int verbose) noexcept
        : ctx(context),
          endpoint(endpoint),
          socket(socket_),
          verbose(verbose){
              alive = 0;
              ping_at = szmq::now() + PING_INTERVAL;
              expires = szmq::now() + SERVER_TTL;
          }

    ~server(){};

    int ping(std::string key) noexcept {
        if (verbose) cout << "+++++ server.ping()\n";
        if (szmq::now() >= ping_at) {
            vector <szmq::Message> ping;
            ping.emplace_back(szmq::Message::from(endpoint));
            ping.emplace_back(szmq::Message::from("PING"));
            if (verbose){
                for (auto it = begin (ping); it != end (ping); ++it) 
                    it->dump();
            }
            socket.sendMultiple(ping);
            ping.clear();
            ping_at = szmq::now() + PING_INTERVAL;
        }
        if (verbose) cout << "---- server.ping()\n";
        return 0;
    }

    uint64_t
    tickless (std::string key,  uint64_t tickless_)
    {
        uint64_t tickless = tickless_;
        if (tickless > ping_at)
            tickless = ping_at;
        return tickless;
    }

    /**
    * server is moveable
    server(server const& other) noexcept {
        ctx = std::move(other.ctx);
        socket = std::move(other.socket);
        endpoint = other.endpoint;
        ping_at = other.ping_at;
        expires = other.ping_at;
        alive = other.alive;
    }
   */
    server& operator=(server const& other) noexcept {
        ctx = std::move(other.ctx);
        socket = std::move(other.socket);
        endpoint = other.endpoint;
        ping_at = other.ping_at;
        expires = other.expires;
        alive = other.alive;
        verbose = verbose;
        return *this;
    }

    std::string endpoint;       // endpoint 
    uint64_t ping_at;            //  Next ping at this time
    uint64_t expires;            //  Expires at this time     
    uint alive;                 //  1 if known to be alive
private :
    int verbose;
    szmq::Context& ctx;        //  Our context wrapper
    //szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> socket;
    szmq::detail::ClientSocketImpl& socket; //  agent router
};


class flclient {
public :
    flclient (szmq::Context& context, int verbose) noexcept
        : ctx(context),
          pipe(context),
          verbose(verbose){
            pipe.bind(szmq::SocketUrl("inproc://internal"));
    }
    ~flclient(){
        pipe.close();
    };

    //  .split connect method
    //  To implement the connect method, the frontend object sends a multipart
    //  message to the backend agent. The first part is a string "CONNECT", and
    //  the second part is the endpoint. It waits 100msec for the connection to
    //  come up, which isn't pretty, but saves us from sending all requests to a
    //  single server, at startup time:
    void connect(std::string endpoint ) noexcept {
        if (verbose) cout << "+++++ flclient.connect()\n";
        if (verbose) cout << boost::format("I: connecting %1% ...\n") % endpoint;        
        vector<szmq::Message> msgs;
        msgs.insert(msgs.begin(), szmq::Message::from("CONNECT"));
        msgs.insert(msgs.begin(), szmq::Message::from(endpoint));
        if (verbose) {
            for (auto it = begin (msgs); it != end (msgs); ++it) 
                it->dump();
        }
        pipe.sendMultiple(msgs);
        msgs.clear();
        std::this_thread::sleep_for(std::chrono::milliseconds(100));  //  Allow connection to come up
        if (verbose) cout << "----- flclient.connect()\n";
    }

    //  To implement the request method, the frontend object sends a message
    //  to the backend, specifying a command "REQUEST" and the request message:
    vector<szmq::Message> 
    request (vector<szmq::Message> request_) noexcept {
        if (verbose) cout << "+++++ flclient.request()\n";
        vector<szmq::Message> request = request_;
        request.emplace_back(szmq::Message::from(std::string("REQUEST")));
        if (verbose) {
            for (auto it = begin (request); it != end (request); ++it) 
                it->dump();
        }
        pipe.sendMultiple(request);
        request.clear();
        auto reply = pipe.recvMultiple();
        if (verbose) {
        for (auto it = begin (reply); it != end (reply); ++it) 
            it->dump();
        }
        if(reply.size() > 0){
            auto status = reply.front().read<std::string>();
            if(status.compare("FAILED")==0) {
                cout << "flclient.request() : " << status << endl;
                reply.clear();
            }
        }
        if (verbose) cout << "----- flclient.request()\n";
        return reply;
    }
private :
    int verbose;
    szmq::Context& ctx;        //  Our context wrapper
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> pipe; //  Pipe through to flcliapi agent
};


class flagent {
public :
    flagent(szmq::Context& context, int verbose) noexcept
        : ctx(context),
          pipe(context),
          router(context),
          verbose(verbose){
              sequence = 0;
              expires = szmq::now() + SERVER_TTL;
              pipe.connect(szmq::SocketUrl("inproc://internal"));
          }

    ~flagent(){
        while (!serversMap.empty())
            serversMap.erase(serversMap.begin());
        while(activesList.size())
				activesList.pop_back();
        pipe.close();
        router.close();
    };

    void controlMessage() noexcept{
        if (verbose) cout << "+++++ flagent.controlMessage()\n";
        auto msg = pipe.recvMultiple();
        if (verbose) {
            for (auto it = begin (msg); it != end (msg); ++it) 
                it->dump();
        }
        auto command = msg.back().read<std::string>(); msg.pop_back();
        if (verbose) cout << "command : " << command << endl;
        if (command.compare("CONNECT") == 0){
            auto endpoint = msg.front().read<std::string>(); msg.erase(msg.begin());
            if (verbose) cout << boost::format("I: connecting to %1%...\n") % endpoint;
            router.connect(szmq::SocketUrl(endpoint));
            server serverIns = server(ctx, router, endpoint, verbose);
            serverIns.ping_at = szmq::now() + PING_INTERVAL;
            serverIns.expires = szmq::now() + SERVER_TTL;
            serversMap.insert(std::make_pair(endpoint, serverIns));
            if (verbose) {
                for (auto it = begin (serversMap); it != end (serversMap); ++it) 
                    cout << "serversMap : " << it->first << endl;  
            }
            activesList.push_back(serverIns);
        } else
        if (command.compare("REQUEST") == 0){
            //assert (request.size() > 0);    //  Strict request-reply cycle
            //  Prefix request with sequence number and empty envelope
            auto sequence_text = boost::str(boost::format("%1%") % ++sequence);
            msg.insert(msg.begin(), szmq::Message::from(sequence_text));
            //  Take ownership of request message
            request = msg;
            msg.clear();
            //  Request expires after global timeout
            expires = szmq::now() + GLOBAL_TIMEOUT;
        }
        if (verbose) cout << "----- flagent.controlMessage()\n";
    }

    //  This method processes one message from a connected
    //  server:
    void routerMessage() noexcept{
        if (verbose) cout << "+++++ flagent.routerMessage() \n";
        auto reply = router.recvMultiple();
        if (verbose) {
            for (auto it = begin (reply); it != end (reply); ++it) 
                it->dump();
        }
        //  Frame 0 is server that replied
        auto endpoint = reply.front().read<std::string>(); reply.erase(reply.begin());
        server serverIns = serversMap.at(endpoint);              
        serverIns.ping_at = szmq::now() + PING_INTERVAL;
        serverIns.expires = szmq::now() + SERVER_TTL;      
        if(!serverIns.alive){
            activesList.push_back(serverIns);
            serverIns.alive = 1;
        } 
        //  Frame 1 may be sequence number for reply or PONG message
        auto control = reply.front().read<std::string>(); reply.erase(reply.begin());
        if(control.compare("PONG")==0){     
            cout << "PONG" << endl;          
        }
        else 
        if (stoi (control) == sequence) {
            reply.emplace_back(szmq::Message::from("OK"));
            if (verbose) {
                for (auto it = begin (reply); it != end (reply); ++it) 
                    it->dump();
            }
            pipe.sendMultiple(reply);
            reply.clear();
            request.clear();
        }
        else
            reply.clear();
        if (verbose) cout << "----- flagent.routerMessage() \n";
    }

    void* run(){
        if (verbose) cout << "+++++ flagent.run()\n";
        std::vector<szmq::PollItem> items = {
            {reinterpret_cast<void*>(*pipe), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*router), 0, ZMQ_POLLIN, 0}};
        while (true) {
            //  Calculate tickless timer, up to 1 hour
            uint64_t tickless = szmq::now() + 1000 * 3600;
            if (request.size() > 0 &&  tickless > expires)
                tickless = expires;
            for (auto it = begin (serversMap); it != end (serversMap); ++it) 
                it->second.tickless(it->first, tickless);
            // Polling 
            szmq::poll (items, 2, (tickless - szmq::now()));
            if (items [0].revents & ZMQ_POLLIN) //pipe
                controlMessage();
            if (items [1].revents & ZMQ_POLLIN) //router
                routerMessage();
            //  If we're processing a request, dispatch to next server
            if (request.size() > 0) {
                if (szmq::now() >= expires) {
                    //  Request expired, kill it
                    cout << "flagent.run() send FAILED\n";
                    pipe.sendOne(szmq::Message::from("FAILED"));
                    request.clear();
                }
                else {
                    //  Find server to talk to, remove any expired ones
                    while (activesList.size() > 0) {
                        server serverIns = activesList.front();                        
                        if (szmq::now() >= serverIns.expires) {
                            activesList.pop_front();
                            serverIns.alive = 0;
                        }
                        else {
                            if (verbose) cout << "Making request Message \n";
                            request.insert(request.begin(), szmq::Message::from(serverIns.endpoint));
                            if (verbose) {
                            for (auto it = begin (request); it != end (request); ++it) 
                                it->dump();
                            }
                            router.sendMultiple(request);
                            request.clear();
                            break;
                        }
                    }
                }
            }
            //  Disconnect and delete any expired servers
            //  Send heartbeats to idle servers if needed
            for (auto it = begin (serversMap); it != end (serversMap); ++it) {
                if (verbose)  cout << "sending ping to server : " << it->first << endl;
                it->second.ping(it->first);
            }
        }
        if (verbose) cout << "----- flagent.run()\n";
        return nullptr;
    }

private :
    int verbose;
    szmq::Context& ctx;        //  Our context wrapper
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_CLIENT> router; //  Socket to talk to servers
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> pipe; //  Socket to talk back to application
    std::map<std::string, server> serversMap;      //Servers we've connected to
    std::list<server> activesList;         //  Servers we know are alive
    uint sequence;              //  Number of requests ever sent
    vector<szmq::Message> request;            //  Current request if any
    vector<szmq::Message> reply;              //  Current reply if any
    uint64_t expires;            //  Timeout for request/reply
}; 


#endif
```

### 빌드 및 테스트

~~~{.bash}
// 원도우
PS D:\work\sook\src\szmq\examples> cl -EHsc flserver3.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc flclient3.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./flserver3 -v
request
[005]006b8b4567
[001]1
[00b]random name
reply
[005]006b8b4567
[001]1
[002]OK
...
[005]006b8b4567
[004]1000
[00b]random name
reply
[005]006b8b4567
[004]1000
[002]OK

PS D:\work\sook\src\szmq\examples> ./flclient3
[020]tcp://localhost:5555
[001]1
[011]random name
[001]1
[002]OK
...
[020]tcp://localhost:5555
[004]1000
[011]random name
...
[004]1000
[002]OK

// 리눅스
zedo@sook:/work/sook/src/szmq/examples$ g++ -o flserver3 flserver3.java -lsook-szmq -lzmq
zedo@sook:/work/sook/src/szmq/examples$ g++ -o flclient3 flclient3.java -lsook-szmq -lzmq -lpthread

zedo@sook:/work/sook/src/szmq/examples$ ./flserver3 -v
request
[005]006b8b4567
[001]1
[00b]random name
reply
[005]006b8b4567
[001]1
[002]OK
...
[005]006b8b4567
[004]1000
[00b]random name
reply
[005]006b8b4567
[004]1000
[002]OK

zedo@sook:/work/sook/src/szmq/examples$ ./flclient3
[020]tcp://localhost:5555
[001]1
[011]random name
[001]1
[002]OK
...
[020]tcp://localhost:5555
[004]1000
[011]random name
...
[004]1000
[002]OK
~~~

프리랜서 API 구현은 상당히 정교하며 이전에 보지 못한 몇 가지 기술을 사용합니다.


 * 멀티스레드 API : 클라이언트 API는 두 부분으로 구성되며, 응용프로그램 스레드에서 실행되는 동기식 flcliapi 클래스와 백그라운드 스레드로 실행되는 비동기 agent 클래스입니다.
 ØMQ가 멀티스레드 응용프로그램들을 쉽게 만드는 방법을 생각하면 flcliapi(동기식) 및 agent(비동기식) 클래스는 inproc 소켓을 통해 메시지들로 통신합니다. 
 모든 ØMQ 측면(예 : 컨텍스트 생성 및 삭제)은 API에 은폐되어 있습니다. 실제로 agent는 미니 브로커처럼 동작하여 백그라운드에서 서버와 통신하면서, 요청 시 가용한 서버을 찾기 위해 최선을 다할 수 있습니다.
* 무지연 폴링 타이머(Tickless poll timer) : 이전 폴링 루프에서 우리는 항상 일정한 시간 간격(예 : 1초)을 사용했는데, 이는 단순하지만 저전력 클라이언트(노트북 또는 휴대폰 등)에 적합하지 않습니다. 
재미와 지구를 구하기 위해 에이전트가 무지연 타이머 사용하면, 무지연 타이머는 예상되는 다음 제한시간을 기준으로 폴링 지연을 계산합니다. 
적절한 구현은 순서가 지정된 제한시간 목록을 유지하며, 모든 제한시간을 확인하고 다음 제한시간까지 폴링 지연을 수행합니다.

## 결론

이 장에서는 다양한 신뢰성 있는 요청-응답 메커니즘을 보았으며, 각각의 특정 비용과 장점이 있었습니다.
예제 코드는 최적화되어 있지는 않지만 대부분 실제 사용이 가능합니다. 모든 다른 패턴 중에서 두 개의 패턴이 양산용으로 사용하기에 돋보였으며, 
첫째는 브로커 기반 안정성을 위한 집사(Majordomo) 패턴과, 
둘째는 브로커 없는 안정성을 위한 프리랜서(Freelance) 패턴입니다.

# 5장 - 고급 발행-구독 패턴

"3장 - 고급 요청-응답 패턴" 및 "4장 - 신뢰할 수 있는 요청-응답 패턴"에서 ØMQ의 요청-응답 패턴의 고급 사용을 보았습니다. 
그 모든 것을 이해하셨다면 축하드리며, 
이 장에서는 발행-구독(publish-subscribe)에 중점을 두고, ØMQ의 핵심인 발행-구독 패턴을 성능, 안정성, 변경정보 배포 및 모니터링을 위한 상위 수준 패턴으로 확장합니다.

다루는 내용은 다음과 같습니다.

* 발행-구독을 사용해야 할 때
* 너무 느린 구독자를 처리하는 방법(자살하는 달팽이 패턴)
* 고속 구독자 설계 방법(블랙박스 패턴)
* 발행-구독 네트워크를 모니터링하는 방법(에스프레소(Espresso) 패턴)
* 공유 카-값 저장소를 만드는 방법(복제 패턴)
* 리엑터를 사용하여 복잡한 서버를 단순화하는 방법
* 바이너리 스타 패턴을 사용하여 서버에 장애조치를 추가하는 방법

PUB 소켓은 각 메시지를 “all of many”로 보내는 반면 PUSH 및 DEALER 소켓은 메시지를 “one of many”로 수신자들에게 순차적으로 전달합니다. 단순히 PUSH를 PUB로 바꾸거나 역으로 하더라도 모든 것이 똑같이 동작하기를 바랄 수는 없습니다. 사람들이 바꾸어 사용(PUSH, PUB)하는 것을 자주 제안하기 때문에 문제는 반복됩니다.

확장성을 얻기 위해 발행-구독(pub-sub)은 푸시-풀(push-pull)과 동일하게 백-채터(back-chatter)를 제거합니다. 백-채터는 수신자가 발신자에게 응답하지 않음을 의미합니다. 일부 예외적인 경우로, SUB 소켓은 PUB 소켓에 구독을 보내지만 익명이며 드물게 일어납니다.

요청-응답과 같이 무엇이 잘못될 수 있는지의 관점에서 신뢰성을 정의하겠습니다. 다음은 발행-구독의 전형적인 장애 사례입니다.

* 구독자가 늦게 가입하여 발행자가 이미 보낸 메시지들을 유실합니다.
* 구독자가 메시지들를 너무 느리게 처리하여, 대기열이 가득 차면 유실됩니다.
* 구독자가 자리를 비운 동안 메시지들를 유실할 수 있습니다.
* 구독자가 장애 후 재시작되면 이미 받은 데이터들을 유실할 수 있습니다.
* 네트워크가 과부하가 되어 데이터들을 유실할 수 있습니다(특히 PGM 통신의 경우).
* 네트워크의 처리 속도가 너무 느려 발행자의 대기열이 가득 차고 발행자에게 장애가 발생할 수 있습니다(ØMQ v3.2 이전).

항상 단순한 것은 아니지만 장애 사례들에는 해결책이 있습니다. 신뢰성은 대부분의 경우 필요하지 않는 복잡한 기능을 요구하지만, ØMQ는 기본적으로 제품 형태로 제공하려고 시도하진 않습니다(신뢰성을 적용하기 위한 범용적인 설계 방안은 존재하지 않습니다).

## 발행-구독 추적 (에스프레소 패턴)

발행-구독 네트워크를 추적하는 방법을 보면서 이장을 시작하겠습니다. “2장 - 소켓 및 패턴”에서 전송 계층 간의 다리 역할을 수행하는 간단한 프록시를 보았습니다. zmq_proxy() 메서드에는 3개 인수는 소켓들로 전송 계층 간 연결할 프론트앤드 소켓과 백엔드 소켓, 모든 메시지를 송신할 캡처 소켓입니다.

### espresso.java : 에스프레소 패턴

```java
//  Espresso Pattern
//  This shows how to capture data using a pub-sub proxy

#include <szmq/szmq.h>
#include <iostream>
#include <thread>
#include <cstdlib>
#include <boost/format.hpp>
using namespace std;

//  The subscriber thread requests messages starting with
//  A and B, then reads and counts incoming messages.

static void*
subscriber_thread (szmq::Context *ctx)
{
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(*ctx);
    //  Subscribe to "A" and "B"
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, "A", 1);
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, "B", 1);   
    subscriber.connect(szmq::SocketUrl("tcp://localhost:6001"));
    int count = 0;
    while (count < 5) {
        auto string = subscriber.recvOne().read<std::string>();
        if (string.length() == 0)
            break;              //  Interrupted
        count++;
    }
    subscriber.close();
    return NULL;
}

//  .split publisher thread
//  The publisher sends random messages starting with A-J:

static void*
publisher_thread (szmq::Context *ctx)
{
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(*ctx);
    publisher.bind(szmq::SocketUrl("tcp://*:6000"));
    while (true) {
        auto string = boost::str(boost::format("%1%-%2%") % char(rand() % 10 + 'A') % (rand() % 100000));
        publisher.sendOne(szmq::Message::from(string));
        std::this_thread::sleep_for(std::chrono::milliseconds(100));     //  Wait for 1/10th second
    }
    return NULL;
}

//  .split listener thread
//  The listener receives all messages flowing through the proxy, on its
//  pipe. In CZMQ, the pipe is a pair of ZMQ_PAIR sockets that connect
//  attached child threads. In other languages your mileage may vary:

static void*
listener_thread (szmq::Context *ctx)
{
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> pipe(*ctx);
    pipe.bind(szmq::SocketUrl("inproc://internal"));
    //  Print everything that arrives on pipe
    while (true) {
        auto frame = pipe.recvOne();
        if (frame.size()==0)
            break;              //  Interrupted
        frame.dump();
    }
    pipe.close();
    return NULL;
}

//  .split main thread
//  The main task starts the subscriber and publisher, and then sets
//  itself up as a listening proxy. The listener runs as a child thread:

int main (void)
{
    srand (static_cast<unsigned int>(std::time(0)));
    //  Start child threads
    szmq::Context ctx;
    szmq::Socket<ZMQ_XPUB, szmq::ZMQ_SERVER> xpublisher(ctx);
    xpublisher.bind(szmq::SocketUrl("tcp://*:6001"));
    szmq::Socket<ZMQ_XSUB, szmq::ZMQ_CLIENT> xsubscriber(ctx);
    xsubscriber.connect(szmq::SocketUrl("tcp://localhost:6000"));  
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> pipe(ctx);
    pipe.connect(szmq::SocketUrl("inproc://internal"));
    {
        thread(&publisher_thread, &ctx).detach();
    }
    {
        thread(&subscriber_thread, &ctx).detach();
    }
    {
        thread(&listener_thread, &ctx).detach();
    }    
    szmq::proxy(reinterpret_cast<void*>(*xsubscriber), reinterpret_cast<void*>(*xpublisher), reinterpret_cast<void*>(*pipe));

    cout <<" interrupted\n";
    //  Tell attached threads to exit
    xsubscriber.close();
    xpublisher.close();
    pipe.close();
    return 0;
}
```

에스프레소는 생성한 리스너 스레드는 PAIR 소켓을 읽고 출력을 합니다. PAIR 소켓은 파이프(pipe)의 한쪽 끝이며 다른 쪽 끝(추가 PAIR)은 zmq_proxy()에 전달하는 소켓입니다. 실제로 관심 있는 메시지를 필터링하여 추적하려는 대상(에스프레소 패턴의 이름처럼)의 본질을 얻습니다.

구독자 스레드는 “A”및 “B”를 포함한 메시지 구독하며, 수신된 메시지가 5 개째 소켓을 제거하고 종료합니다. 예제를 실행하면 리스너는 2개의 구독 메시지(/01)들, 5개의 데이터 메시지들(“A”, “B” 포함)과 2개의 구독 취소 메시지(/00)를 출력하고 조용해집니다.

구독자는 발행자에게 구독 시와 구독 취소 시 이벤트(event)를 보내며 보내는 메시지는 바이트(byte) 형태로 첫 번째 바이트(HEX 코드)는 “00”은 구독, “01”은 구독 취소이며 나머지 바이트들은 토픽(sizeof(event)-1)으로 구성됩니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc espresso.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./espresso
[002] 0141       --> "01" 구독, 토픽 : "41" A
[002] 0142       --> "01" 구독, 토픽 : "41" B
[007] B-91164
[007] B-12979
[007] A-52599
[007] A-06417
[007] A-45770
[002] 0041       --> "00" 구독 취소, 토픽 : "41" A
[002] 0042       --> "00" 구독 취소, 토픽 : "41" B
~~~

## 마지막 값 캐싱

상용 발행-구독 시스템을 사용했다면 빠르고 즐거운 ØMQ 발행-구독 모델에서 누락된 일부 익숙한 기능들을 알 수 있습니다. 이 중 하나는 마지막 값 캐싱(LVC : Last Value Caching)입니다. 이것은 새로운 구독자가 네트워크에 참여할 때 누락한 메시지 받을 수 있는 방식으로 문제를 해결합니다. 이론은 발행자들에게 새로운 구독자가 참여했고 특정 토픽에 구독하려는 시점을 알려주면, 발행자는 해당 토픽에 대한 마지막 메시지를 재송신할 수 있습니다.

“ØMQ를 사용하여 LVC(last value cache)를 만들 수 있을까요?”에 대한 대답은 “예”이며 발행자와 구독자들 사이에 프록시를 만들면 됩니다. PGM 네트워크 스위치와 유사하지만 ØMQ는 우리가 직접 프로그래밍할 수 있습니다.

최악의 시나리오를 가정한 발행자와 구독자를 만드는 것부터 시작하겠습니다. 이 발행자는 병리학적입니다. 발행자는 각각 1000개의 토픽들에 즉시 메시지를 보내고 1초마다 임의의 토픽에 하나의 변경정보를 전송합니다. 구독자는 연결하고 토픽을 구독합니다. 마지막 값 캐핑(LVC)이 없으면 구독자는 데이터를 얻기 위해 평균 500초를 기다려야 합니다. TV 드라마에서 그레고르라는 탈옥한 죄수가 8.3분 내에 구독자의 데이터를 얻게 하지 못한다면 장난감 토끼 로저에게서 머리를 날려버리겠다고 위협하는 것처럼 말입니다.

다음은 발행자 코드입니다. 일부 주소에 연결하는 명령 줄 옵션이 없으면 단말에 바인딩되며, 나중에 마지막 값 캐시(LVC)에 연결에 사용합니다.

### pathopub.java : 병리학적인 발행자

```java
//  Pathological publisher
//  Sends out 1,000 topics and then one random update per second

#include <szmq/szmq.h>
#include <iostream>
#include <thread>
#include <cstdlib>
#include <string>
using namespace std;

int main (int argc, char *argv [])
{
    szmq::Context context;
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(context);

    if (argc == 2)
        publisher.bind(szmq::SocketUrl(argv [1]));
    else
        publisher.bind(szmq::SocketUrl("tcp://*:5556"));

    //  Ensure subscriber connection has time to complete
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));   

    //  Send out all 1,000 topic messages
    int topic_nbr;
    for (topic_nbr = 0; topic_nbr < 1000; topic_nbr++) {
        publisher.sendMore(szmq::Message::from(to_string(topic_nbr)));
        publisher.sendOne(szmq::Message::from("Save Roger"));
    }
    //  Send one random update per second
    srand (static_cast<unsigned int>(std::time(0)));
    while (true) {
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));  
        publisher.sendMore(szmq::Message::from(to_string(rand() % 1000)));
        publisher.sendOne(szmq::Message::from("Off with his head!"));
    }
    publisher.close();
    return 0;
}
```

구독자 코드는 다음과 같습니다.

### pathosub.java : 병리학적인 구독자

```java
//  Pathological subscriber
//  Subscribes to one random topic and prints received messages

#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>
#include <string>
using namespace std;

int main (int argc, char *argv [])
{
    szmq::Context context;
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(context);

    if (argc == 2)
        subscriber.connect(szmq::SocketUrl(argv [1]));
    else
        subscriber.connect(szmq::SocketUrl("tcp://localhost:5556"));

    srand (static_cast<unsigned int>(std::time(0)));
    std::string subscription = to_string(rand() % 1000);
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, subscription.c_str(), subscription.length());
    
    while (true) {
        auto topic = subscriber.recvOne().read<std::string>();
        auto data = subscriber.recvOne().read<std::string>();
        if(topic.compare(subscription) != 0)
            break;
        cout << data << endl;
    }
    subscriber.close();
    return 0;
}
```

먼저 구독자를 실행하고 다음 발행자를 실행하면 구독자가 기대한 대로 “Save Roger”를 출력합니다.
“pathsub” 실행하면 생성된 토픽에 해당되는 “Sava Roger”을 수신하고 임의의 긴 시간(1초~16.7분) 후에 “Off with his head!”을 수신하게 됩니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc pathopub.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc pathosub.java szmq.lib
PS D:\work\sook\src\szmq\examples> ./pathopub
PS D:\work\sook\src\szmq\examples> ./pathosub
Save Roger
Off with his head!

// 주소를 지정한 경우
PS D:\work\sook\src\szmq\examples> ./pathopub tcp://*:5555
PS D:\work\sook\src\szmq\examples> ./pathosub tcp://localhost:5555
Save Roger
Off with his head!
~~~

두 번째 구독자를 실행하면 Roger의 곤경(“Save Roger”을 수신하지 못하고 오랜 시간 후에 “Off with his head!” 수신)을 알게 되며 데이터 수신하기까지 오랜 시간을 기다려야 합니다. 여기에 마지막 값 캐시(LVC)가 있습니다. 프록시가 2개의 소켓에 바인딩하고 양쪽의 메시지를 처리합니다.

### lvcach.java : 마지막 값 저장 프록시

```java
//  Last value cache
//  Uses XPUB subscription messages to re-send data
#include <szmq/szmq.h>
#include <iostream>
#include <map>
using namespace std;

int main (void)
{
    szmq::Context context;
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> frontend(context);
    frontend.connect(szmq::SocketUrl("tcp://localhost:5557"));
    szmq::Socket<ZMQ_XPUB, szmq::ZMQ_SERVER> backend(context);
    backend.bind(szmq::SocketUrl("tcp://*:5558"));

    //  Subscribe to every single topic from publisher
    frontend.setSockOpt(ZMQ_SUBSCRIBE, "", 0);

    //  Store last instance of each topic in a cache
    std::map<std::string, std::string> cache;

    //  .split main poll loop
    //  We route topic updates from frontend to backend, and
    //  we handle subscriptions by sending whatever we cached,
    //  if anything:
    while (true) {
        std::vector<szmq::PollItem> items = {
			{reinterpret_cast<void*>(*frontend), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*backend), 0, ZMQ_POLLIN, 0}};

        szmq::poll(items, 2, 1000);
        //  Any new topic data we cache and then forward
        if (items [0].revents & ZMQ_POLLIN) {
            auto topic = frontend.recvOne().read<std::string>();
            auto current = frontend.recvOne().read<std::string>();
            if (topic.length()==0)
                break;
            if(cache.count(topic)==0)
                cache.insert(std::make_pair(topic, current));   //insert
            else
                cache[topic] = current;  //update
            backend.sendMore(szmq::Message::from(topic));
            backend.sendOne(szmq::Message::from(current));
        }
        //  .split handle subscriptions
        //  When we get a new subscription, we pull data from the cache:
        if (items [1].revents & ZMQ_POLLIN) {
            auto frame = backend.recvOne().read<std::string>();            
            if (frame.length() == 0)
                break;
            //  Event is one byte 0=unsub or 1=sub, followed by topic
            if (frame[0] == 1) {
                auto topic = frame.substr(1, frame.length());
                cout << "Sending cached topic " << topic << endl;
                if(cache.count(topic)) {
                    auto previous = cache.at(topic);
                    backend.sendMore(szmq::Message::from(topic));
                    backend.sendOne(szmq::Message::from(previous));
                }
            }
        }
    }
    
    while (!cache.empty())
        cache.erase(cache.begin());
    frontend.close();
    backend.close();
    return 0;
}
```

#### 빌드 및 테스트


프록시와 발행자를 순서대로 실행합니다.
그리고 원하는 수의 구독자들을 실행 시 프록시의 5558 포트에 연결합니다.

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc lvcache.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./lvcache
Sending cached topic 985
Sending cached topic 158

PS D:\work\sook\src\szmq\examples> ./pathopub tcp://*:5557

PS D:\work\sook\src\szmq\examples> ./pathosub tcp://localhost:5558
Save Roger
Off with his head!
PS D:\work\sook\src\szmq\examples> ./pathosub tcp://localhost:5558
Save Roger
~~~

각 구독자는 “Save Roger”를 보고 받으며, 탈옥한 죄수 그레고르는 다시 감옥으로 돌아가 따뜻한 우유 한잔과 저녁 식사를 하게 되었습니다. 그는 진정 범죄를 저지르기보다는 관심받기를 원한 것이었습니다.

## 느린 구독자 감지(자살하는 달팽이 패턴)

실제 생활에서 발행-구독 패턴을 사용할 때 발생하는 일반적인 문제는 느린 구독자입니다. 이상적인 세상에서 우리는 발행자에서 구독자로 데이터를 전속력으로 전송합니다. 실제로 구독자 응용프로그램은 종종 인터프리터 개발 언어로 작성되었거나 혹은 많은 작업을 처리하거나 잘못된 코드로 인한 오류로 발행자가 전송하는 데이터를 처리하지 못하는 상황이 발생할 수 있습니다.

자살하는 달팽이 패턴은 특히 구독자가 자신의 클라이언트들과 서비스 수준 계약(SLA, Service Level Agreements)을 맺고 있고 특정 최대 지연 시간을 보장해야 하는 경우에 효과적입니다. 최대 지연 시간을 보장하기 위해 구독자를 중단하는 것은 건설적인 방법은 아니지만, 가정 설정문(assertion) 모델입니다. 오늘 중단하면 문제는 해결되지만 지연되는 데이터가 하류(구독자들)로 흐르도록 허용하면 문제가 더 넓게 퍼져 많은 손상을 발생하지만 레이더(문제를 감지하는 수단)로 감지하는데 오래 걸릴 수 있습니다.

자살하는 달팽이에 대한 작은 예제입니다.

### suisnail.java : 자살하는 달팽이

```java
//  Suicidal Snail

#include <szmq/szmq.h>
#include <iostream>
#include <thread>
#include <cstdlib>
using namespace std;

//  This is our subscriber. It connects to the publisher and subscribes
//  to everything. It sleeps for a short time between messages to
//  simulate doing too much work. If a message is more than one second
//  late, it croaks.

#define MAX_ALLOWED_DELAY   1000    //  msecs

static void*
subscriber (szmq::Context *ctx)
{
    // Interanl pipe
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> pipe(*ctx);
    pipe.connect(szmq::SocketUrl("inproc://sub"));
    //  Subscribe to everything
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(*ctx);
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    subscriber.connect(szmq::SocketUrl("tcp://localhost:5556"));

    //  Get and process messages
    while (true) {
        auto msg = subscriber.recvOne().read<uint64_t>();;
         //  Suicide snail logic
        if (szmq::now() - msg > MAX_ALLOWED_DELAY) {
            cout << "E: subscriber cannot keep up, aborting\n";
            pipe.sendOne(szmq::Message::from("gone and died"));
            break;
        }
        //  Work for 1 msec plus some random additional time
        std::this_thread::sleep_for(std::chrono::milliseconds(1 + rand() % 2)); 
    }
    pipe.close();
    subscriber.close();
    return NULL;
}

//  .split publisher task
//  This is our publisher task. It publishes a time-stamped message to its
//  PUB socket every millisecond:

static void*
publisher (szmq::Context *ctx)
{
    // Interanl pipe
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> pipe(*ctx);
    pipe.connect(szmq::SocketUrl("inproc://pub"));
    //  Prepare publisher
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(*ctx, szmq::NonblockingFlag{true});
    publisher.bind(szmq::SocketUrl("tcp://*:5556"));

    while (true) {
        //  Send current clock (msecs) to subscribers
        uint64_t now = szmq::now();
        publisher.sendOne(szmq::Message::from(now));
        auto signal = pipe.recvOne(0).read<std::string>();
        if (signal.length() > 0) {
            cout << "Message from main() : " << signal << endl;
            break;
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(1));   //  1msec wait
    }
    pipe.close();
    publisher.close();
    return NULL;
}

//  .split main task
//  The main task simply starts a client and a server, and then
//  waits for the client to signal that it has died:

int main (void)
{
    szmq::Context ctx;
    // Interanl pipe
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> subpipe(ctx);
    subpipe.bind(szmq::SocketUrl("inproc://sub"));
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> pubpipe(ctx);
    pubpipe.bind(szmq::SocketUrl("inproc://pub"));
    // thread   
    thread(&publisher, &ctx).detach();
    thread(&subscriber, &ctx).detach();
    auto fromSub = subpipe.recvOne().read<std::string>();
    cout << "Message from subscriber thread : " << fromSub << endl;
    pubpipe.sendOne(szmq::Message::from("break"));
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    subpipe.close();
    pubpipe.close();
    return 0;
}
```

pipe 통신은 PAIR 진행되며, 2개의 스레드에 대하여 pubpipe, subpipe가 필요합니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc suisnail.java szmq.lib
PS D:\work\sook\src\szmq\examples> ./suisnail
1605306034227
1605306034229
...
1605306035476
1605306035478
E: subscriber cannot keep up, aborting
Message from subscriber thread : gone and died
Message from main() : break
~~~

자살하는 달팽이 예제에서 몇 가지 주목할 사항은 다음과 같습니다.

* 여기에 있는 메시지는 밀리초(msec) 단위의 현재 시스템 시각으로만 구성됩니다. 현실적인 응용프로그램에서는 적어도 메시지는 "타임스탬프가 있는 메시지 헤더"와 "데이터가 있는 메시지 본문"으로 구성되어야 합니다.
* 예제에서는 단일 프로세스에서 2개의 스레드인 발행자와 구독자가 있습니다. 실제로는 별도의 프로세스들입니다. 스레드는 편의상 데모를 위하여 사용합니다.

## 고속 구독자 (블랙 박스 패턴)

이제 구독자를 더 빠르게 만드는 한 가지 방법을 보겠습니다. 발행-구독의 일반적인 사용 사례는 증권거래소에서 발생하는 시장 데이터와 같은 대용량 데이터 스트림을 배포하는 것입니다. 
일반적인 설정에는 발행자가 증권거래소에 연결되어 주가를 여러 구독자들에게 보냅니다. 구독자가 적으면 TCP를 사용할 수 있습니다. 많은 구독자의 경우 안정적인 멀티캐스트, 즉 PGM을 사용합니다.

 샤딩을 사용하여 작업을 병렬 및 독립 스트림으로 분할합니다(한 스트림에서 토픽 키의 절반, 다른 스트림에서 토픽 키의 절반). 많은 스트림을 사용할 수 있지만 유휴 CPU 코어가 없으면 성능이 확장되지 않습니다. 이제 하나의 메시지 스트림을 2개의 메시지 스트림으로 분할하는 방법을 보겠습니다.

 병렬 샤딩(Parallel Sharding)은 데이터가 서로 공유되지 않도록 완전히 분리하여 독립적으로 동작하는 샤딩(Sharding)을 여러 개로 묶어서 병렬 처리를 통해 성능 및 확장성을 확보하는 방식입니다.

 이상적으로는 아키텍처에서 완전히 부하 처리가 가능한 스레드 수를 코어 수와 일치시키려고 했습니다. 스레드가 코어 및 CPU 주기를 놓고 싸우기(Time sharing) 시작하면 스레드를 추가하는 비용이 메시지 처리량보다 큽니다(더 많은 메시지 처리하기 위해 더 많은 CPU 코어 구매). 더 많은 I/O 스레드를 생성하는 것에는 이점이 없습니다.

 ### hssub.java : 고속 구독자

 ```java
//  Hello World client
#include <szmq/szmq.h>
#include <assert.h>
#include <iostream>
#include <thread>
#include <boost/format.hpp>
#define NBR_WORKERS 3
using namespace std;

//  This is our subscriber. It connects to the publisher and subscribes
//  to everything. It sleeps for a short time between messages to
//  simulate doing too much work. If a message is more than one second
//  late, it croaks.
#define MAX_ALLOWED_DELAY   1000    //  msecs

//  .split publisher task
static void*
publisher (szmq::Context *ctx)
{
    // Interanl pipe
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> pipe(*ctx);
    pipe.connect(szmq::SocketUrl("inproc://pub"));
    //  Prepare publisher
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(*ctx);
    publisher.bind(szmq::SocketUrl("tcp://*:5556"));

    while (true) {
        //  Send current clock (msecs) to subscribers
        uint64_t now = szmq::now();
        publisher.sendOne(szmq::Message::from(now));
        auto signal = pipe.recvOne(0).read<std::string>();
        if (signal.length() > 0) {
            cout << "Publisher Message from main() : " << signal << endl;
            break;
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(1));   //  1msec wait
    }
    pipe.close();
    publisher.close();
    return NULL;
}

// subscriber task
static void*
subscriber (szmq::Context *ctx)
{
    // Interanl pipe
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> pipe(*ctx);
    pipe.connect(szmq::SocketUrl("inproc://sub"));
    //  Subscribe to everything
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(*ctx);
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    subscriber.connect(szmq::SocketUrl("tcp://localhost:5556"));
    
    //  Socket to send messages to
    szmq::Socket<ZMQ_PUSH, szmq::ZMQ_SERVER> sender(*ctx);
    sender.bind(szmq::SocketUrl("tcp://*:5557"));
    //  Get and send messages
    while (true) {
        auto msg = subscriber.recvOne();
        sender.sendOne(msg);
        //  Suicide snail logic
        if (szmq::now() - msg.read<uint64_t>() > MAX_ALLOWED_DELAY) {
            cout << "E: subscriber cannot keep up, aborting\n";
            pipe.sendOne(szmq::Message::from("gone and died"));
            break;
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(1 + rand() % 2)); 
    }
    pipe.close();
    subscriber.close();
    return NULL;
}
// worker task
static void*
worker(void *args, szmq::Context *ctx)
{
    // Interanl pipe
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> pipe(*ctx);
    pipe.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    pipe.connect(szmq::SocketUrl("inproc://work"));
    //  Subscribe to everything
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_CLIENT> worker(*ctx);
    worker.connect(szmq::SocketUrl("tcp://localhost:5557"));
    // poll items
    std::vector<szmq::PollItem> items = {
			{reinterpret_cast<void*>(*worker), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*pipe), 0, ZMQ_POLLIN, 0}};
    //  Get and send messages
    while (true) {
        szmq::poll(items, 2, 10);
        if (items [0].revents & ZMQ_POLLIN) {
            auto msg = worker.recvOne().read<uint64_t>();
            cout << boost::format("[%1%]Receive: [%2%]\n") % (intptr_t)args % msg;
        }
        if (items [1].revents & ZMQ_POLLIN) {
            auto signal = pipe.recvOne(0).read<std::string>();
            if (signal.length() > 0) {
                cout << "Worker Message from main() : " << signal << endl;
                break;
            }
        
        }
    }
    pipe.close();
    worker.close();
    return NULL;
}

int main (void)
{
    szmq::Context ctx;
    // Interanl pipe
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> subpipe(ctx);
    subpipe.bind(szmq::SocketUrl("inproc://sub"));
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> pubpipe(ctx);
    pubpipe.bind(szmq::SocketUrl("inproc://pub"));
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> workpipe(ctx);
    workpipe.bind(szmq::SocketUrl("inproc://work"));
    // thread  
    thread(&publisher, &ctx).detach();
    thread(&subscriber, &ctx).detach();
    for (int worker_nbr = 0; worker_nbr < NBR_WORKERS; worker_nbr++)
        thread(&worker, (void *)(intptr_t)worker_nbr, &ctx).detach();
    // receive signal from subpipe
    auto fromSub = subpipe.recvOne().read<std::string>();
    cout << "Message from subscriber thread : " << fromSub << endl;
    pubpipe.sendOne(szmq::Message::from("break"));
    workpipe.sendOne(szmq::Message::from("break"));
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    subpipe.close();
    pubpipe.close();
    workpipe.close();
    return 0;
}
 ```

 3개의 작업자들에게 라운드로빈 형태로 메시지가 전달되는 것을 확인 가능합니다.

 ### 빌드 및 테스트

 ~~~{.bash}
 PS D:\work\sook\src\szmq\examples> cl -EHsc hssub.java szmq.lib
PS D:\work\sook\src\szmq\examples> ./hssub
[2]Receive: [1605313432325]
[1]Receive: [1605313432327]
[0]Receive: [1605313432329]
...
[1]Receive: [1605313436314]
[0]Receive: [1605313436316]
[2]Receive: [1605313436318]
E: subscriber cannot keep up, aborting
[1]Receive: [1605313436320]
Message from subscriber thread : gone and died
Worker Message from main() : break
Worker Message from main() : break
Worker Message from main() : break
Publisher Message from main() : break
 ~~~

 ## 신뢰할 수 있는 발행-구독 (복제 패턴)

 좀 더 큰 작업 예제로, 신뢰할 수 있는 발행-구독 아키텍처를 만드는 문제를 보겠습니다. 
우리는 이것을 단계적으로 개발하며 목표는 일련의 응용프로그램들이 일부 공통 상태를 공유하게 합니다. 기술적 도전 과제들은 다음과 같습니다.

* 수천 또는 수만 개의 클라이언트 응용프로그램들이 있습니다.
* 임의로 네트워크에 가입하고 탈퇴합니다.
* 이러한 응용프로그램들은 하나의 최종 일관성 상태를 공유해야 합니다.
* 모든 응용프로그램은 언제든지 상태를 변경할 수 있습니다.

우리는 한 번에 하나씩 문제를 해결하면서 단계적으로 복제(Clone)를 개발할 것입니다. 먼저 일련의 클라이언트들에서 공유 상태를 변경하는 방법을 보겠습니다. 우리는 상태를 표현하고 변경한 방법을 결정해야 합니다. 가장 단순한 형식은 키-값 저장소(해시 테이블)로 하나의 키-값 쌍은 공유 상태를 변경하는 기본 단위가 됩니다.

변경정보는 신규 키-값 쌍이거나 기존 키의 수정된 값 혹은 삭제된 키입니다. 지금은 전체 저장소가 메모리에 저장할 수 있고 응용프로그램들이 해시 테이블이나 사전을 사용하는 것처럼 키로 접근한다고 가정합니다. 더 큰 저장소와 일종의 지속성이 필요한 경우 상태를 데이터베이스에 저장할 수 있지만 여기서는 관련이 없습니다.

다음은 서버 코드입니다.

### clonesrv1.java : 복제 서버, 모델 1

```java
//  Clone server Model One
#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>
#include <thread>
#include <string>
#include <map>
#include <boost/format.hpp>
using namespace std;

int main (void)
{
    //  Prepare our context and publisher socket
    szmq::Context ctx;
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(ctx);
    publisher.bind(szmq::SocketUrl("tcp://*:5556"));
    std::this_thread::sleep_for(std::chrono::milliseconds(200)); 
    // making key-value map
    std::map<uint64_t, uint64_t> kvmap;
    int64_t sequence = 0;
    std::srand (static_cast<unsigned int>(std::time(0)));
    vector <szmq::Message> kvmsg;
    while (true) {
        //  Distribute as key-value message
        uint64_t key = rand() % 10000;
        uint64_t value = rand() % 1000000;
        kvmsg.emplace_back(szmq::Message::from(key));  //  frame 0: key (0MQ string)
        kvmsg.emplace_back(szmq::Message::from(++sequence)); //  frame 1: sequence (8 bytes, network order)
        kvmsg.emplace_back(szmq::Message::from(value));  //  frame 2: body (blob)
        //cout << sequence << ", " << key << ", " << value << endl;
        publisher.sendMultiple(kvmsg);
        if(kvmap.count(key))
            kvmap[key] = value;
        else
            kvmap.insert(std::make_pair(key, value));
        kvmsg.clear();
    }
    cout << boost::format(" Interrupted\n%1% messages out\n") % (int)sequence;
    publisher.close();
    while (!kvmap.empty())
        kvmap.erase(kvmap.begin());
    return 0;
}
```

다음은 클라이언트 코드입니다.

### clonecli1.java : 복제 클라이언트, 모델 1

```java
//  Clone client Model One
#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>
#include <thread>
#include <map>
#include <boost/format.hpp>
using namespace std;

int main (void)
{
    //  Prepare our context and updates socket
    szmq::Context ctx;
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> updates(ctx);
    updates.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    updates.connect(szmq::SocketUrl("tcp://localhost:5556"));

    // making key-value map
    std::map<uint64_t, uint64_t> kvmap;
    int64_t sequence = 0;

    while (true) {
        auto kvmsg = updates.recvMultiple();
        if (kvmsg.size()==0)
            break;          //  Interrupted
        //  frame 0: key (0MQ string)
        //  frame 1: sequence (8 bytes, network order)
        //  frame 2: body (blob)
        auto key = kvmsg.front().read<uint64_t>(); kvmsg.erase(kvmsg.begin()); //delete key
        sequence = kvmsg.front().read<int64_t>(); kvmsg.erase(kvmsg.begin()); //delete sequence
        auto value = kvmsg.front().read<uint64_t>();
        //cout << sequence << ", " << key << ", " << value << endl;
        if(kvmap.count(key))
            kvmap[key] = value;
        else
            kvmap.insert(std::make_pair(key, value));
        if((sequence % 10000) == 0)
            cout << boost::format("kvmap key : %1%, value %2%\n") % key % kvmap.at(key);
        kvmsg.clear();
    }
    cout << boost::format(" Interrupted\n%1% messages out\n") % (int)sequence;
    updates.close();
    while (!kvmap.empty())
        kvmap.erase(kvmap.begin());
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc clonesrv1.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc clonecli1.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./clonesrv1

PS D:\work\sook\src\szmq\examples> ./clonecli1
kvmap key : 1182, value 7692    --> 10,000번에 한번씩 출력
kvmap key : 3804, value 16830
kvmap key : 8983, value 21542
kvmap key : 265, value 12
kvmap key : 1481, value 1079
~~~

* 멀티파트(multipart) ØMQ 메시지는 3개의 프레임으로 구성됩니다 : 키(ØMQ 문자열), 순서 번호(네트워크 바이트 순서의 64 비트 값(int64_t)), 바이너리 본문(모든 항목을 포함)
* 서버는 임의의 4자리 키(0~9999)로 메시지를 생성하므로, 크지만 방대하지는 않은 해시 테이블(1만 개 항목들)을 시뮬레이션할 수 있습니다.
* 현재 버전에서는 삭제를 구현하지 않습니다. 모든 메시지는 삽입 또는 변경입니다.
* 서버는 소켓을 바인딩하고 200 밀리초 동안 일시 중지합니다. 이는 구독자가 서버의 소켓에 연결할 때 메시지를 유실하는 느린 참여 증후군(slow joiner syndrome)을 방지합니다. 이후 버전의 복제 코드에서는 제거하겠습니다.
* 소켓에 대하여 코드상에서 발행자와 구독자라는 용어를 사용할 것입니다. 이것은 나중에 다른 일을 하는 다중 소켓들로 작업할 때 도움이 됩니다.

다음은 현재 동작하는 가장 간단한 형식의 kvmsg 클래스입니다.

### kvmsg_simple.h : 키-값 메시지 클래스

```java
/*  =====================================================================
 *  kvmsg_simple - simple key-value message class for example applications
 *  ===================================================================== */
        
#ifndef __KVSIMPLE_H_INCLUDED__
#define __KVSIMPLE_H_INCLUDED__

#include <szmq/szmq.h>
#include <iostream>
#include <algorithm>
#include <thread>
#include <map>
#include <boost/format.hpp>
using namespace std;

//  Message is formatted on wire as 3 frames:
//  frame 0: key (string)
//  frame 1: sequence (unit64_t)
//  frame 2: body (string)
#define FRAME_KEY       0
#define FRAME_SEQ       1
#define FRAME_BODY      2
#define KVMSG_FRAMES    3

class kvmsg {
public :
    //  Constructor, sets sequence as provided
    kvmsg(uint64_t sequence, int verbose=0) noexcept
        : mverbose(verbose){
            setSequence(sequence);
          }
    kvmsg recv(szmq::detail::SocketImpl& socket) noexcept {
        kvmsg me(0, mverbose);
        for (int frame_nbr = 0; frame_nbr < KVMSG_FRAMES; frame_nbr++) {
            me.mpresent [frame_nbr] = 1;
            me.mframe[frame_nbr] = socket.recvOne();
            //  Verify multipart framing
            bool rcvmore = (frame_nbr < KVMSG_FRAMES - 1)? true: false;
            if (socket.hasMore() != rcvmore) 
                break;
        }
        return me;
    }
    void send(szmq::detail::SocketImpl& socket) noexcept {
        for (int frame_nbr = 0; frame_nbr < KVMSG_FRAMES; frame_nbr++) {
            if (frame_nbr < KVMSG_FRAMES - 1)
                socket.sendMore(mframe[frame_nbr]);
            else
                socket.sendOne(mframe[frame_nbr]);
        }
    }
    std::string key(){
        if (mpresent [FRAME_KEY] == 1) {
            mkey = mframe[FRAME_KEY].read<std::string>();
            return mkey;
        }
        else
            return NULL;
    }
    uint64_t sequence(){
        return mframe [FRAME_SEQ].read<uint64_t>();
    }
    std::string body(){
        if (mpresent [FRAME_BODY]==1)
            return mframe[FRAME_BODY].read<std::string>();
        else
            return NULL;
    }
    size_t size() {
        if (mpresent [FRAME_BODY] == 1)
            return mframe [FRAME_BODY].size();
        else
            return 0;
    }
    //  Set message key as provided
    void setKey(std::string key){
        mframe[FRAME_KEY] = szmq::Message::from(key);
        mpresent [FRAME_KEY] = 1;
    }
    //  Set message sequence number
    void setSequence (uint64_t sequence){
        mframe [FRAME_SEQ] = szmq::Message::from(sequence);
        mpresent [FRAME_SEQ] = 1;
    }
    //  Set message body
    void setBody (std::string body){
        mframe [FRAME_BODY] = szmq::Message::from(body);
        mpresent [FRAME_BODY] = 1;
    }

    //  Set message key using printf format
    void fmtKey (char *format, ...){
        char value[KVMSG_KEY_MAX + 1];
        va_list args;
        va_start (args, format);
        vsnprintf (value, KVMSG_KEY_MAX, format, args);
        va_end (args);
        setKey (value);
    }
    //  Set message body using printf format
    void fmtBody (char *format, ...){
        char value[KVMSG_KEY_MAX + 1];
        va_list args;
        va_start (args, format);
        vsnprintf (value, KVMSG_KEY_MAX, format, args);
        va_end (args);
        setBody (value);
    }

    //  Store entire kvmsg into hash map, if key/value are set
    //  Nullifies kvmsg reference, and destroys automatically when no longer
    //  needed.
    void store (std::map<std::string, std::string> *kvmap){
        if (mpresent [FRAME_KEY] == 1 &&  mpresent [FRAME_BODY] == 1) {
            if(kvmap->count(key()))
                kvmap->at(key()) = body();
            else
                kvmap->insert(std::make_pair(key(), body()));
        }
    }
    //  Dump message to stderr, for debugging and tracing
    void dump (){
        cout << boost::format("[seq:%1%]\n") % sequence();
        cout << boost::format("[key:%1%]\n") % key();
        cout << boost::format("[size:%1%]\n") % size();
        cout << boost::format("[body:%1%]\n") % body();
    }

    //  Runs self test of class
    int test(){
        cout << " * kvmsg: \n";

        //  Prepare our context and sockets
        szmq::Context ctx;
        szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> output(ctx);
        output.bind(szmq::SocketUrl("inproc://kvmsg_selftest"));
        szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> input(ctx);
        input.connect(szmq::SocketUrl("inproc://kvmsg_selftest"));

        std::map<std::string, std::string> kvmap;

        //  Test send and receive of simple message
        kvmsg kvmsg(1);
        kvmsg.setKey("key");
        kvmsg.setBody("body");
        if (mverbose)
            kvmsg.dump();
        kvmsg.send(output);
        kvmsg.store(&kvmap);
        auto kvmsg1 = kvmsg.recv(input);;
        if (mverbose)
            kvmsg1.dump();
        assert (kvmsg1.key().compare("key") == 0);
        kvmsg1.store (&kvmap);
        for (auto it = begin (kvmap); it != end (kvmap); ++it) 
            cout << "kvmap(key) : " << it->first <<", kvmap(body) : " << it->second << endl;
        //  Shutdown and destroy all objects
        while(!kvmap.empty())
            kvmap.erase(kvmap.begin());
        output.close();
        input.close();
        cout << "OK\n";
        return 0;
    }
   // Copyable 
    kvmsg(kvmsg const& other) noexcept {
        mverbose = other.mverbose;
        msequence = other.msequence;
        std::copy(other.mframe, other.mframe+KVMSG_FRAMES, mframe);
        std::copy(other.mpresent, other.mpresent+KVMSG_FRAMES, mpresent);
        mkey = other.mkey;
    }

    kvmsg& operator=(kvmsg const& other) noexcept {
        mverbose = other.mverbose;
        msequence = other.msequence;
        std::copy(other.mframe, other.mframe+KVMSG_FRAMES, mframe);
        std::copy(other.mpresent, other.mpresent+KVMSG_FRAMES, mpresent);
        mkey = other.mkey;
        return *this;
    }
    //  Destructor
     ~kvmsg(){};
private :
    int mverbose;
    uint64_t msequence;
    //  Presence indicators for each frame
    int mpresent [KVMSG_FRAMES];
    //  Corresponding 0MQ message frames, if any
    szmq::Message mframe [KVMSG_FRAMES];
    //  Key, copied into safe C string
    std::string mkey;
};

#endif     
```

 “kvmsg” 클래스에 대한 테스트를 수행하기 위한 “kvsimtest.c”는 다음과 같습니다.

 ### kvsimtest.java : “kvmsg” 클래스에 대한 테스트

 ```java
#include <szmq/szmq.h>
#include <iostream>
#include "kvmsg_simple.h"

using namespace std;

int main(){
	kvmsg kvmsg(1,1);
	kvmsg.test();
	return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc kvsimtest.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./t11
 * kvmsg:
[seq:1]
[key:key]
[size:4]
[body:body]
[seq:1]
[key:key]
[size:4]
[body:body]
kvmap(key) : key, kvmap(body) : body
OK
~~~

다음은 kvmsg 클래스 사용한 clone 서버와 클라이언트 예제입니다.

### clonesrv_kvm.java

```java
//  Clone server Model One
#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>
#include <thread>
#include <string>
#include <map>
#include <boost/format.hpp>
#include "kvmsg_simple.h"
using namespace std;

int main (void)
{
    //  Prepare our context and publisher socket
    szmq::Context ctx;
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(ctx);
    publisher.bind(szmq::SocketUrl("tcp://*:5556"));
    std::this_thread::sleep_for(std::chrono::milliseconds(200)); 

    std::map<std::string, std::string> kvmap;
    uint64_t sequence = 0;
    std::srand (static_cast<unsigned int>(std::time(0)));
    while (true) {
        //  Distribute as key-value message
        kvmsg kvmsg(++sequence);
        kvmsg.fmtKey("%d", rand() % 10000);
        kvmsg.fmtBody("%d", rand() % 1000000);
        cout << kvmsg.sequence() << ", " << kvmsg.key() << ", " << kvmsg.body() << endl;
        kvmsg.send(publisher);
        kvmsg.store(&kvmap);
    }
    cout << boost::format(" Interrupted\n%1% messages out\n") % (int) sequence;
    while(!kvmap.empty())
        kvmap.erase(kvmap.begin());
    publisher.close();
    return 0;
}
```

### clonecli_kvm.java

```java
//  Clone client Model One
#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>
#include <thread>
#include <map>
#include <boost/format.hpp>
#include "kvmsg_simple.h"
using namespace std;

int main (void)
{
    //  Prepare our context and updates socket
    szmq::Context ctx;
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> updates(ctx);
    updates.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    updates.connect(szmq::SocketUrl("tcp://localhost:5556"));

    std::map<std::string, std::string> kvmap;
    uint64_t sequence = 0;

    while (true) {
        kvmsg kvmsg = kvmsg.recv (updates);
        kvmsg.store (&kvmap);
        cout << kvmsg.sequence() << ", " << kvmsg.key() << ", " << kvmsg.body() << endl;
        sequence++;
    }
    printf (" Interrupted\n%d messages in\n", (int) sequence);
    while(!kvmap.empty())
        kvmap.erase(kvmap.begin());
    updates.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc clonesrv_kvm.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc clonecli_kvm.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./clonesrv_kvm
83168, 5588, 7219
83169, 9508, 20825
83170, 6126, 20640
83171, 923, 25304
83172, 8279, 23030
83173, 6969, 3773
83174, 2134, 28908
...
PS D:\work\sook\src\szmq\examples> ./clonecli_kvm
83168, 5588, 7219
83169, 9508, 20825
83170, 6126, 20640
83171, 923, 25304
83172, 8279, 23030
83173, 6969, 3773
83174, 2134, 28908
...
~~~

서버와 클라이언트 모두 해시 테이블을 유지하지만, 첫 번째 모델은 서버를 시작하기 전에 모든 클라이언트들을 구동하고 클라이언트들이 충돌하지 않는 경우에만 제대로 작동하여 인위적입니다.

### 대역 외 스냅샷 얻기

이제 두 번째 문제가 있습니다. 늦게 가입하는 클라이언트들과 혹은 장애조치 이후 재시작한 클라이언트들을 처리하는 방법입니다.

늦게 가입(또는 재시작)한 클라이언트들에 대하여 서버가 이미 발행한 메시지들을 받을 수 있게 하려면 서버 상태의 스냅샷을 얻어야 합니다. “메시지”의 의미를 “순서화된 키-값 쌍”으로 줄인 것처럼 “상태 정보”의 의미도 “해시 테이블” 줄일 수 있습니다. 서버 상태 정보를 얻기 위해 클라이언트는 DEALER 소켓을 열고 명시적으로 요청합니다.

이 작업을 수행하기 위해 타이밍 문제를 해결해야 합니다. 상태 스냅샷을 얻기는 일정 시간이 걸리며 스냅샷이 크면 상당히 오래 걸릴 수 있습니다. 
스냅샷에 올바르게 변경정보를 적용해야 하지만 서버는 변경정보 전송을 언제 시작할지 모릅니다.
한 가지 방법은 클라이언트가 구독을 시작하고 첫 번째 변경정보를 받은 다음 "변경정보 N에 대한 상태(state for update N)"를 요청하는 것입니다. 이렇게 하려면 서버가 각 변경정보에 대해 하나의 스냅샷을 저장해야 하므로 실용적이지 않습니다.

따라서 클라이언트의 동기화는 다음과 같이 수행합니다.

* 클라이언트는 먼저 변경정보들을 구독한 다음 상태 요청을 하면, 상태가 이전 오래된 변경정보보다 최신이 됩니다. 
* 클라이언트는 서버가 상태로 응답할 때까지 대기하고 모든 변경정보를 대기열에 넣습니다. 
변경정보를 대기열에 넣기만 하고 처리하지 않도록 하는 것은 대기열을 읽지 않음으로써 가능합니다 : ØMQ는 변경정보들을 소켓 대기열에 넣어 보관합니다.
* 클라이언트가 상태 변경을 받으면 대기열에 넣어 두었던 변경정보 읽기를 다시 시작합니다. 그러나 상태 변경 시점보다 오래된 변경정보들은 모두 삭제됩니다. 따라서 상태 변경 전에 대기열에 최대 200 개의 변경정보가 포함된 경우 클라이언트는 최대 201개의 변경정보를 삭제합니다.
* 그런 다음 클라이언트는 자체 상태 스냅샷에 변경정보를 적용합니다.

다음의 서버 예제는 ØMQ의 자체 내부 대기열을 이용하는 단순한 모델입니다. 

### clonesrv2.java : 복제 서버, 모델 2

```java
//  Clone server - Model Two

//  Lets us build this source without creating a library
#include "kvmsg_simple.h"
#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>
#include <thread>
using namespace std;

//  Routing information for a key-value snapshot
typedef struct {
    szmq::detail::SocketImpl& socket;           //  ROUTER socket to send to
    std::string identity;     //  Identity of peer who requested state
} kvroute_t;

static int s_send_single (std::string key, std::string body, kvroute_t kvroute)
static void state_manager (szmq::Context *ctx);

int main (void)
{
    //  Prepare our context and sockets
    szmq::Context ctx;
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(ctx);
    publisher.bind(szmq::SocketUrl("tcp://*:5557"));
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> pipe(ctx);
    pipe.bind(szmq::SocketUrl("inproc://internal"));

    uint64_t sequence = 0;
    srand (static_cast<unsigned int>(std::time(0)));

    //  Start state manager and wait for synchronization signal
    thread(&state_manager, &ctx).detach();
    auto msg = pipe.recvOne();
    cout << "I: state_manager is " << msg.read<std::string>() << endl;
    
    while (true) {
        //  Distribute as key-value message
        kvmsg kvmsg(++sequence);
        kvmsg.fmtKey("%d", rand() % 10000);
        kvmsg.fmtBody("%d", rand() % 1000000);
        kvmsg.send(publisher);
        kvmsg.send(pipe);
    }
    cout << boost::format(" Interrupted\n%1% messages out\n") % (int) sequence;
    publisher.close();
    pipe.close();
    return 0;
}

//  Send one state snapshot key-value pair to a socket
//  Hash item data is our kvmsg object, ready to send
static int
s_send_single (std::string key, std::string body, kvroute_t kvroute)
{
    //  Send identity of recipient first
    kvroute.socket.sendMore(szmq::Message::from(kvroute.identity));
    kvmsg kvmsg(0);
    kvmsg.setKey(key);
    kvmsg.setBody(body);
    kvmsg.send(kvroute.socket);
    return 0;
}

//  .split state manager
//  The state manager task maintains the state and handles requests from
//  clients for snapshots:

static void
state_manager (szmq::Context *ctx)
{
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> pipe(*ctx);
    pipe.connect(szmq::SocketUrl("inproc://internal"));

    std::map<std::string, std::string> kvmap;
    pipe.sendOne(szmq::Message::from("READY"));

    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> snapshot(*ctx);
    snapshot.bind(szmq::SocketUrl("tcp://*:5556"));
    std::vector<szmq::PollItem> items = {
			{reinterpret_cast<void*>(*pipe), 0, ZMQ_POLLIN, 0},
            {reinterpret_cast<void*>(*snapshot), 0, ZMQ_POLLIN, 0}};
    uint64_t sequence = 0;       //  Current snapshot version number
    while (true) {
        szmq::poll (items, 2, -1);
        //  Apply state update from main thread
        if (items [0].revents & ZMQ_POLLIN) {
            kvmsg kvmsg = kvmsg.recv(pipe);
            sequence = kvmsg.sequence ();
            kvmsg.store (&kvmap);
        }
        //  Execute state snapshot request
        if (items [1].revents & ZMQ_POLLIN) {
            auto identity = snapshot.recvOne().read<std::string>();
            if (identity.size() == 0)
                break;          //  Interrupted

            //  Request is in second frame of message
            auto request = snapshot.recvOne().read<std::string>();
            if (request.compare("ICANHAZ?") != 0){
                cout << boost::format("E: bad request : (%1%), aborting\n") % request;
                break;
            }
            //  Send state snapshot to client
            kvroute_t routing = { snapshot, identity };
            //  For each entry in kvmap, send kvmsg to client
            for (auto it = begin (kvmap); it != end (kvmap); ++it) 
                s_send_single(it->first, it->second, routing);

            //  Now send END message with sequence number
            cout << "Sending state shapshot= " << (int) sequence << endl;
            snapshot.sendMore(szmq::Message::from(identity));
            kvmsg kvmsg(sequence);
            kvmsg.setKey("KTHXBAI");
            kvmsg.setBody("");
            kvmsg.send(snapshot);
        }
    }
    while(!kvmap.empty())
        kvmap.erase(kvmap.begin());
    pipe.close();
    snapshot.close();
}
```

### clonecli2.java : 복제 클라이언트, 모델 2
```java
//  Clone client - Model Two

//  Lets us build this source without creating a library
#include "kvmsg_simple.h"

int main (void)
{
    //  Prepare our context and subscriber
    szmq::Context ctx;
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> snapshot(ctx);
    snapshot.connect(szmq::SocketUrl("tcp://localhost:5556"));
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(ctx);
    subscriber.connect(szmq::SocketUrl("tcp://localhost:5557"));

    std::map<std::string, std::string> kvmap;

    //  Get state snapshot
    uint64_t sequence = 0;
    snapshot.sendOne(szmq::Message::from("ICANHAZ?"));
    while(true){
        kvmsg kvmsg = kvmsg.recv(snapshot);
        if(kvmsg.key().compare("KTHXBAI") == 0) {
            sequence = kvmsg.sequence();
            cout << "Received snapshot=" << (int)sequence << endl;
            break;
        }
        kvmsg.store(&kvmap);
    }
    //  Now apply pending updates, discard out-of-sequence messages
    while (true) {
        kvmsg kvmsg = kvmsg.recv(subscriber);
        if(kvmsg.sequence() > sequence){
            sequence = kvmsg.sequence();
            kvmsg.store (&kvmap);
        }
    }
    while (!kvmap.empty())
        kvmap.erase(kvmap.begin());
    snapshot.close();
    subscriber.close();
    return 0;
}
```

clonecli2에서 "ICANHAZ?" 메시지를 clonesrv2로 보내면 서버는 해시 테이블에 저장한 변경정보들을 `s_send_single()`통하여 모두 전송하고 "KTHXBAI" 메시지를 전송합니다.
clonecli2에서 "KTHXBAI"을 받으면 해당 sequence를 기준으로 clonesrv2에서 발행된 변경정보의 sequence와 비교하여 이후의 것들만 받아 해시 테이블에 보관합니다.(이전 정보는 폐기)
* "ICANHAZ?"는 "I Can has?"(가져도 될까요?)이며 "KTHXBAI"는 "Ok, Thank you, goodbye"(예, 고마워요, 잘 있어요)를 의미합니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc clonesrv2.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc clonecli2.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./clonesrv2
I: state_manager is READY
Sending state shapshot= 20483
Sending state shapshot= 3397381
Sending state shapshot= 7798297

PS D:\work\sook\src\szmq\examples> ./clonecli2
Received snapshot=20483
PS D:\work\sook\src\szmq\examples> ./clonecli2
Received snapshot=3397381
PS D:\work\sook\src\szmq\examples> ./clonecli2
Received snapshot=7798297

~~~

2개의 프로그램에서 몇 가지 주목할 사항은 다음과 같습니다.
* 서버는 2가지 작업을 수행합니다. 하나의 스레드(clonesrv2)는 변경정보(무작위로)를 생성하여 메인 PUB 소켓으로 보내고, 다른 스레드(state_manager)는 파이프(PAIR)에 변경정보를 받아 해시 테이블에 보관하고 ROUTER 소켓에서 클라이언트의 상태 요청들을 처리합니다. 메인 스레드(clonesrv2)와 자식 스레드(state_manager)는 `inproc://` 연결을 통해 PAIR 소켓을 통해 통신합니다.
* 클라이언트는 정말 간단합니다. C 언어에서는 약 50 줄의 코드로 구성됩니다. 많은 무거운 작업이 kvmsg 클래스에서 수행됩니다. 그럼에도 불구하고 기본 복제 패턴은 처음에 보였던 것보다 구현하기가 쉽습니다.
* 우리는 상태를 직렬화하기 위해 화려한 것을 사용하지 않습니다. 해시 테이블은 일련의 kvmsg 객체를 보유하고 서버는 이러한 객체를 일련의 메시지로 클라이언트 요청(ICANHAZ) 시에 보냅니다. 여러 클라이언트들이 한 번에 상태를 요청하면, 각각 다른 스냅샷을 얻습니다.
* 우리는 클라이언트에 정확히 하나의 서버가 있다고 가정하며 서버가 실행 중이어야 합니다. 서버에 장애가 발생하면 어떻게 되는지는 여기서 다루지 않습니다.

### 클라이언트들로부터 변경정보 재발행

두 번째 모델에서 키-값 저장소에 대한 변경 사항은 서버 자체적으로 나왔습니다. 중앙집중식 모델에서는 유용하며, 예를 들어 서버에 중앙 구성 파일을 두고 각 노드들의 로컬 저장소에 사용하기 위해 배포합니다. 더 흥미로운 모델은 서버가 아닌 클라이언트로부터 변경정보들을 받는 것이며, 이럴 경우 서버는 상태 비저장(stateless) 브로커가 되며, 몇 가지 이점을 제공합니다.

* 우리는 서버의 안정성에 대해 덜 걱정합니다. 서버에 충돌이 발생하면 재시작하여 클라이언트에 신규 값을 제공할 수 있습니다.
* 키-값 저장소를 사용하여 활성 동료 간에 지식(저장소)을 공유할 수 있습니다.

클라이언트에서 다시 서버로 변경정보를 보내기 위해, 다양한 소켓 패턴을 사용할 수 있지만 가장 간단한 솔루션은 PUSH-PULL 조합입니다.

클라이언트들이 서로에게 직접 변경정보를 발행하지 않는 이유는 지연 시간(latency)이 줄어들지만 일관성(consistency)을 보장할 수 없기 때문입니다. 
변경정보를 받는 사람에 따라 변경정보 순서를 변경한다면 일관된 공유 상태를 얻을 수 없습니다. 
서로 다른 키들을 변경하는 두 개의 클라이언트들이 있다고 가정하면 잘 작동하지만 두 개의 클라이언트들이 거의 동시에 동일한 키를 변경하려고 하면 동일한 키에 다른 값으로 변경되어 일관성이 없어집니다.

모든 변경 사항을 조정하기 위해, 서버는 모든 변경정보에 고유한 순서 번호를 추가할 수 있습니다. 고유한 순서 번호를 통해 클라이언트는 네트워크 정체(cogestion) 및 대기열 오버플로우(overflow)를 포함하여 심각한 장애를 감지할 수 있습니다. 클라이언트는 수신 메시지 스트림에 구멍(순서 번호가 연결되지 않는 부분)이 있음을 발견하면 조치를 취할 수 있습니다. 클라이언트가 서버에 접속하여 누락된 메시지를 요청하는 것이 합리적으로 보이지만 실제로는 유용하지 않습니다. 만약 구멍 네트워크 처리 부하에 따른 스트레스로 인한 것이라면, 네트워크에 더 많은 스트레스를 가하면 상황이 악화됩니다. 클라이언트가 할 수 있는 일은 사용자에게 “계속할 수 없음”이라고 경고하고 중지하고, 누군가가 문제의 원인을 수동으로 확인할 때까지 재시작하지 않는 것입니다.

클라이언트에서 상태 변경들을 생성하겠습니다. 다음은 서버의 코드입니다.

### clonesrv3.java : 복제 서버, 모델 3

```java
//  Clone server - Model Three

//  Lets us build this source without creating a library
#include "kvmsg_simple.h"
#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>

//  Routing information for a key-value snapshot
typedef struct {
    szmq::detail::SocketImpl& socket;           //  ROUTER socket to send to
    std::string identity;     //  Identity of peer who requested state
} kvroute_t;

//  Send one state snapshot key-value pair to a socket
//  Hash item data is our kvmsg object, ready to send
static int
s_send_single (std::string key, std::string body, kvroute_t kvroute)
{
    //  Send identity of recipient first
    kvroute.socket.sendMore(szmq::Message::from(kvroute.identity));
    kvmsg kvmsg(0);
    kvmsg.setKey(key);
    kvmsg.setBody(body);
    kvmsg.send(kvroute.socket);
    return 0;
}

int main (void)
{
    //  Prepare our context and sockets
    szmq::Context ctx;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> snapshot(ctx);
    snapshot.bind(szmq::SocketUrl("tcp://*:5556"));
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(ctx);
    publisher.bind(szmq::SocketUrl("tcp://*:5557"));
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_SERVER> collector(ctx);
    collector.bind(szmq::SocketUrl("tcp://*:5558"));    

    //  .split body of main task
    //  The body of the main task collects updates from clients and
    //  publishes them back out to clients:
    uint64_t sequence = 0;
    std::map<std::string, std::string> kvmap;
    std::vector<szmq::PollItem> items = {
        {reinterpret_cast<void*>(*collector), 0, ZMQ_POLLIN, 0},
        {reinterpret_cast<void*>(*snapshot), 0, ZMQ_POLLIN, 0}};
    while (true) {
        szmq::poll (items, 2, 1000);

        //  Apply state update sent from client
        if (items [0].revents & ZMQ_POLLIN) {
            kvmsg kvmsg = kvmsg.recv(collector);
            kvmsg.setSequence(++sequence);
            kvmsg.send(publisher);
            kvmsg.store (&kvmap);
            cout << boost::format("I: publishing update %1%\n") % (int) sequence;
        }
        //  Execute state snapshot request
        if (items [1].revents & ZMQ_POLLIN) {
            auto identity = snapshot.recvOne().read<std::string>();
            //  Request is in second frame of message
            auto request = snapshot.recvOne().read<std::string>();
            if (request.compare("ICANHAZ?")!=0){
                cout << boost::format("E: bad request : (%1%), aborting\n") % request;
                break;
            }
            //  Send state snapshot to client
            kvroute_t routing = { snapshot, identity };
            //  For each entry in kvmap, send kvmsg to client
           for (auto it = begin (kvmap); it != end (kvmap); ++it) 
                s_send_single(it->first, it->second, routing);

            //  Now send END message with sequence number
            cout << "Sending state shapshot= " << (int) sequence << endl;
            snapshot.sendMore(szmq::Message::from(identity));
            kvmsg kvmsg(sequence);
            kvmsg.setKey("KTHXBAI");
            kvmsg.setBody("");
            kvmsg.send(snapshot);
        }
    }
    cout << boost::format(" Interrupted\n%1% messages out\n") % (int) sequence;
    while(!kvmap.empty())
        kvmap.erase(kvmap.begin());
    collector.close();
    snapshot.close();
    publisher.close();
    return 0;
}
```

다음은 클라이언트의 코드입니다.

### clonecli3.java : 복제 클라이언트, 모델 3

```java
//  Clone client - Model Three

//  Lets us build this source without creating a library
#include "kvmsg_simple.h"

int main (void)
{
    //  Prepare our context and subscriber
    szmq::Context ctx;
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> snapshot(ctx);
    snapshot.connect(szmq::SocketUrl("tcp://localhost:5556"));
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(ctx);
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    subscriber.connect(szmq::SocketUrl("tcp://localhost:5557"));
    szmq::Socket<ZMQ_PUSH, szmq::ZMQ_CLIENT> publisher(ctx);
    publisher.connect(szmq::SocketUrl("tcp://localhost:5558"));

    std::map<std::string, std::string> kvmap;
    srand (static_cast<unsigned int>(std::time(0)));

    //  .split getting a state snapshot
    //  We first request a state snapshot:
    uint64_t sequence = 0;
    snapshot.sendOne(szmq::Message::from("ICANHAZ?"));
    while (true) {
        kvmsg kvmsg = kvmsg.recv(snapshot);
        if(kvmsg.key().compare("KTHXBAI") == 0) {
            sequence = kvmsg.sequence();
            cout << "Received snapshot=" << (int)sequence << endl;
            break;
        }
        kvmsg.store(&kvmap);
    }
    //  .split processing state updates
    //  Now we wait for updates from the server and every so often, we
    //  send a random key-value update to the server:
    uint64_t alarm = szmq::now() + 1000;
    while (true) {
        std::vector<szmq::PollItem> items = {
            {reinterpret_cast<void*>(*subscriber), 0, ZMQ_POLLIN, 0}};
        uint64_t tickless = (int) ((alarm - szmq::now()));
        if (tickless < 0)
            tickless = 0;
        szmq::poll (items, 1, tickless);
        if (items [0].revents & ZMQ_POLLIN) {
            kvmsg kvmsg = kvmsg.recv (subscriber);
            //  Discard out-of-sequence kvmsgs, incl. heartbeats
            if (kvmsg.sequence () > sequence) {
                sequence = kvmsg.sequence ();
                kvmsg.store (&kvmap);
                cout << boost::format("I: received update=%1%\n") % (int) sequence;
            }
        }
        //  If we timed out, generate a random kvmsg
        if (szmq::now() >= alarm) {
            kvmsg kvmsg(0);
            kvmsg.fmtKey("%d", rand() % 10000);
            kvmsg.fmtBody("%d", rand() % 1000000);
            kvmsg.send(publisher);
            alarm = szmq::now() + 1000;
        }
    }
    cout << boost::format(" Interrupted\n%1% messages out\n") % (int) sequence;
    while(!kvmap.empty())
        kvmap.erase(kvmap.begin());
    subscriber.close();
    snapshot.close();
    publisher.close();
    return 0;
}
```

clonecli3에서 1초마다 보내는 변경정보를 clonesrv3은 클라이언트들에 발행하며 clonecli3 중지하면 clonesrv3도 더 이상 변경정보를 발행하지 않습니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc clonesrv3.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc clonecli3.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./clonesrv3
Sending state shapshot= 0
I: publishing update 1
I: publishing update 2
...
I: publishing update 13
Sending state shapshot= 13
I: publishing update 14
...
Sending state shapshot= 16
I: publishing update 17

PS D:\work\sook\src\szmq\examples> ./clonecli3
Received snapshot=0
I: received update=1
I: received update=2
...
I: received update=13
PS D:\work\sook\src\szmq\examples> ./clonecli3
Received snapshot=13
I: received update=14
I: received update=15
I: received update=16
...
PS D:\work\sook\src\szmq\examples> ./clonecli3
Received snapshot=22
...
~~~

세 번째 설계에서 몇 가지 주목할 점은 다음과 같습니다.
* 서버가 단일 작업(clonesrv3)으로 축소되었으며 클라이언트로부터 수신되는 변경정보를 위한 PULL 소켓, 상태 요청 및 스냅샷 전달을 위한 위한 ROUTER 소켓, 클라이언트에게 변경정보 송신을 위한 PUB 소켓을 관리합니다.
* 클라이언트는 간단한 무지연(tickless) 타이머를 사용하여 1초에 한 번씩 서버에 무작위 업데이트를 보냅니다. 실제 구현에서는 응용프로그램 코드에서 업데이트를 유도합니다.

### 하위트리와 작업

클라이언트들의 수가 증가하면 공유 스토어의 규모도 커질 것입니다. 모든 클라이언트에게 모든 스냅샷을 보내는 것은 합리적이지 않습니다. 이것은 발행-구독의 고전적인 이야기입니다 : 클라이언트들의 수가 매우 적으면 모든 메시지들을 모든 클라이언트들에게 보낼 수 있습니다. 아키텍처를 커지면 비효율적이 되며 클라이언트는 다양한 영역에 전문화되어 있습니다.

따라서 서버의 공유 저장소로 작업할 때에도 일부 클라이언트들은 해당 저장소의 일부에서만 작업하기를 원하여, 일부 저장소를 하위트리(subtree)라고 합니다. 클라이언트는 상태 요청 시 하위 트리를 요청해야 하며 변경정보들을 구독할 때 동일한 하위트리를 지정해야 합니다.

트리에 대한 몇 가지 공통적인 문법들이 있으며, 하나는 경로 계층(path hierarchy)이고 다른 하나는 토픽 트리(topic tree)입니다. 다음과 같이 보입니다.

* 경로 계층 : /some/list/of/paths
* 토픽 트리 : some.list.of.topics

예제에서는 경로 계층을 사용하고, 클라이언트와 서버를 확장하여 클라이언트가 단일 하위트리로 작업하게 합니다. 단일 하위트리로 작업하는 방법을 알게 되면 용도에 따라 다중 하위트리들을 처리하도록 확장할 수 있습니다.

다음은 모델 3의 작은 변형으로 하위트리를 구현하는 서버의 코드입니다.

### clonesrv4.java : 복제 서버, 모델 4

```java
//  Clone server - Model Three

//  Lets us build this source without creating a library
#include "kvmsg_simple.h"
#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>
#include <boost/format.hpp>
using namespace std;

//  Routing information for a key-value snapshot
typedef struct {
    szmq::detail::SocketImpl& socket;           //  ROUTER socket to send to
    std::string identity;     //  Identity of peer who requested state
    std::string subtree;          //  Client subtree specification
} kvroute_t;

//  Send one state snapshot key-value pair to a socket
//  Hash item data is our kvmsg object, ready to send
static int
s_send_single (std::string key, std::string body, kvroute_t kvroute)
{
    kvmsg kvmsg(0);
    kvmsg.setKey(key);
    kvmsg.setBody(body);
    if(kvroute.subtree.length() <= kvmsg.key().length() && 
     kvroute.subtree.compare(kvmsg.key().substr(0, kvroute.subtree.length()))==0){
         //  Send identity of recipient first
        kvroute.socket.sendMore(szmq::Message::from(kvroute.identity));
        kvmsg.send(kvroute.socket);
     }
    return 0;
}

int main (int argc, char *argv [])
{
    int verbose = (argc > 1 && strcmp (argv [1], "-v") == 0);
    //  Prepare our context and sockets
    szmq::Context ctx;
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> snapshot(ctx);
    snapshot.bind(szmq::SocketUrl("tcp://*:5556"));
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher(ctx);
    publisher.bind(szmq::SocketUrl("tcp://*:5557"));
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_SERVER> collector(ctx);
    collector.bind(szmq::SocketUrl("tcp://*:5558"));    

    //  .split body of main task
    //  The body of the main task collects updates from clients and
    //  publishes them back out to clients:
    uint64_t sequence = 0;
    std::map<std::string, std::string> kvmap;
    std::vector<szmq::PollItem> items = {
        {reinterpret_cast<void*>(*collector), 0, ZMQ_POLLIN, 0},
        {reinterpret_cast<void*>(*snapshot), 0, ZMQ_POLLIN, 0}};
    while (true) {
        szmq::poll (items, 2, 1000);

        //  Apply state update sent from client
        if (items [0].revents & ZMQ_POLLIN) {
            kvmsg kvmsg = kvmsg.recv(collector);
            kvmsg.setSequence(++sequence);
            kvmsg.send(publisher);
            kvmsg.store (&kvmap);
            if (verbose) kvmsg.dump();
            cout << boost::format("I: publishing update %1%\n") % (int) sequence;
        }
        //  Execute state snapshot request
        if (items [1].revents & ZMQ_POLLIN) {
            auto identity = snapshot.recvOne().read<std::string>();
            //  Request is in second frame of message
            auto request = snapshot.recvOne().read<std::string>();
            std::string subtree;
            if (request.compare("ICANHAZ?")==0){
                subtree = snapshot.recvOne().read<std::string>();
            }
            else{
                std::cout << boost::format("E: bad request : (%1%), aborting\n") % request;
                break;
            }
            //  Send state snapshot to client
            kvroute_t routing = { snapshot, identity, subtree };
            //  For each entry in kvmap, send kvmsg to client
           for (auto it = begin (kvmap); it != end (kvmap); ++it) 
                s_send_single(it->first, it->second, routing);

            //  Now send END message with sequence number
            cout << "Sending state shapshot= " << (int) sequence << endl;
            snapshot.sendMore(szmq::Message::from(identity));
            kvmsg kvmsg(sequence);
            kvmsg.setKey("KTHXBAI");
            kvmsg.setBody("");
            kvmsg.send(snapshot);
        }
    }
    cout << boost::format(" Interrupted\n%1% messages out\n") % (int) sequence;
    while(!kvmap.empty())
        kvmap.erase(kvmap.begin());
    collector.close();
    snapshot.close();
    publisher.close();
    return 0;
}
```

하위트리의 저장소의 내용을 구독하는 클라이언트의 코드입니다.

### clonecli4.java : 복제 클라이언트, 모델 4

```java
//  Clone client - Model Three

//  Lets us build this source without creating a library
#include "kvmsg_simple.h"

#define SUBTREE "/client/"

int main (int argc, char *argv [])
{
    int verbose = (argc > 1 && strcmp (argv [1], "-v") == 0);
    //  Prepare our context and subscriber
    szmq::Context ctx;
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> snapshot(ctx);
    snapshot.connect(szmq::SocketUrl("tcp://localhost:5556"));
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(ctx);
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    subscriber.connect(szmq::SocketUrl("tcp://localhost:5557"));
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, SUBTREE, 8);
    szmq::Socket<ZMQ_PUSH, szmq::ZMQ_CLIENT> publisher(ctx);
    publisher.connect(szmq::SocketUrl("tcp://localhost:5558"));

    std::map<std::string, std::string> kvmap;
    srand (static_cast<unsigned int>(std::time(0)));

    //  .split getting a state snapshot
    //  We first request a state snapshot:
    uint64_t sequence = 0;
    snapshot.sendMore(szmq::Message::from("ICANHAZ?"));
    snapshot.sendOne(szmq::Message::from(SUBTREE));
    while (true) {
        kvmsg kvmsg = kvmsg.recv(snapshot);
        if(kvmsg.key().compare("KTHXBAI") == 0) {
            sequence = kvmsg.sequence();
            cout << "Received snapshot=" << (int)sequence << endl;
            break;
        }
        kvmsg.store(&kvmap);
    }
    //  .split processing state updates
    //  Now we wait for updates from the server and every so often, we
    //  send a random key-value update to the server:
    uint64_t alarm = szmq::now() + 1000;
    while (true) {
        std::vector<szmq::PollItem> items = {
            {reinterpret_cast<void*>(*subscriber), 0, ZMQ_POLLIN, 0}};
        uint64_t tickless = (int) ((alarm - szmq::now()));
        if (tickless < 0)
            tickless = 0;
        szmq::poll (items, 1, tickless);
        if (items [0].revents & ZMQ_POLLIN) {
            kvmsg kvmsg = kvmsg.recv (subscriber);
            if (verbose) kvmsg.dump();
            //  Discard out-of-sequence kvmsgs, incl. heartbeats
            if (kvmsg.sequence () > sequence) {
                sequence = kvmsg.sequence ();
                kvmsg.store (&kvmap);
                cout << boost::format("I: received update=%1%\n") % (int) sequence;
            }
        }
        //  If we timed out, generate a random kvmsg
        if (szmq::now() >= alarm) {
            kvmsg kvmsg(0);
            kvmsg.fmtKey("%s%d", SUBTREE, rand() % 10000);
            kvmsg.fmtBody("%d", rand() % 1000000);
            kvmsg.send(publisher);
            alarm = szmq::now() + 1000;
        }
    }
    cout << boost::format(" Interrupted\n%1% messages out\n") % (int) sequence;
    while(!kvmap.empty())
        kvmap.erase(kvmap.begin());
    subscriber.close();
    snapshot.close();
    publisher.close();
    return 0;
}
```

### 빌드 및 테스트

clonecli4에서 필터링을 통해 "SUBTREE"가 포함되어 발행(publish)된 kvmsg 객체를 받아 해시 테이블에 저정하며, 1초 간격으로 상태 요청에 사용될 kvmsg의 key에 "SUBTREE"을 포함하여 보냅니다.

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc clonesrv4.java szmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc clonecli4.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./clonesrv4 -v
Sending state shapshot= 0
[seq:1], [key:/client/1066], [size:4], [body:9960]
I: publishing update 1
[seq:2], [key:/client/702], [size:4], [body:1374]
I: publishing update 2
[seq:3], [key:/client/4484], [size:5], [body:28077]
I: publishing update 3
...

PS D:\work\sook\src\szmq\examples> ./clonecli4 -v
Received snapshot=0
[seq:1], [key:/client/1066], [size:4], [body:9960]
I: received update=1
[seq:2], [key:/client/702], [size:4], [body:1374]
I: received update=2
[seq:3], [key:/client/4484], [size:5], [body:28077]
I: received update=3
~~~

## 임시값들

임시값은 정기적으로 갱신(refresh)되지 없으면 자동으로 만료(expire)되는 값입니다. 등록 서비스에 복제가 사용된다고 생각하면, 임시값을 동적 값으로 사용할 수 있습니다. 노드는 네트워크에 가입하고 주소를 게시하고 이를 정기적으로 갱신합니다. 노드가 죽으면 그 주소는 결국 제거됩니다.

임시값들에 대한 일반적인 추상화는 이를 세션에 연결하고 세션이 종료될 때 삭제하는 것입니다. 
복제에서 세션은 클라이언트들에 의해 정의되며 클라이언트가 죽으면 종료됩니다. 
더 간단한 대안은 유효시간(TTL(Time to Live))을 임시값에 포함시켜 서버에서 TTL에 정해진 시간 동안 값이 갱신되지 않을 경우 만료하는 데 사용합니다.

이제 임시값을 구현합니다. 먼저 키-값 메시지에서 TTL을 인코딩하는 방법이 필요하며 프레임으로 추가할 수 있습니다.  속성으로 ØMQ 프레임을 사용할 때의 문제점은 매번 신규 속성을 추가할 때마다 메시지 구조를 변경할 경우 ØMQ 버전 간의 호환성을 깨뜨립니다. 따라서 메시지에 속성 프레임을 추가하고 속성값을 가져오고 입력할 수 있는 코드를 작성해 보겠습니다.

다음으로, "이 값을 삭제하십시오"라고 말하는 방법이 필요합니다. 지금까지는 서버와 클라이언트는 항상 맹목적으로 해시 테이블에 신규 값을 넣거나 기존 값을 변경했습니다. 
앞으로는 해시 테이블에서 값이 비어 있으면 "키 삭제"를 의미하게 하겠습니다.

다음은 기존 `kvmsg_simple` 보다 좀 더 완벽한 버전의 `kvmsg` 클래스이며, 속성들 프레임(나중에 필요할 UUID 프레임을 추가)을 구현하였습니다. 또한 필요한 경우 해시 테이블에서 키(key)를 삭제함으로 빈 값(empty value)을 처리합니다.

### kvmsg.h : 키-값 메시지 클래스

```java
/*  =====================================================================
 *  kvmsg_simple - simple key-value message class for example applications
 *  ===================================================================== */
        
#ifndef __KVMSG_H_INCLUDED__
#define __KVMSG_H_INCLUDED__

#include <szmq/szmq.h>
#include <cstdarg>
#include <string>
#include <sstream>
#include <iostream>
#include <algorithm>
#include <thread>
#include <map>
#include <boost/format.hpp>
#include <boost/uuid/uuid.hpp>            // uuid class
#include <boost/uuid/uuid_generators.hpp> // generators
#include <boost/uuid/uuid_io.hpp>         // streaming operators etc.
#include <boost/algorithm/string.hpp>
using namespace std;
//  Keys are short strings
#define KVMSG_KEY_MAX   255

//  Message is formatted on wire as 5 frames:
//  frame 0: key (string)
//  frame 1: sequence (uint64_t(8 byte))
//  frame 2: uuid (sting, 16 bytes)
//  frame 3: properties (string)
//  frame 4: body (string)
#define FRAME_KEY       0
#define FRAME_SEQ       1
#define FRAME_UUID      2
#define FRAME_PROPS     3
#define FRAME_BODY      4
#define KVMSG_FRAMES    5

class kvmsg {
public :
    //  Constructor, sets sequence as provided
    kvmsg(uint64_t sequence, int verbose=0) noexcept
        : mverbose(verbose){
            setSequence(sequence);
          }
    //  .split property encoding
    //  These two helpers serialize a list of properties to and from a
    //  message frame:   
    void encodeProps() noexcept{  // map to string
        std::string props;
        for (auto it = begin (mprops); it != end (mprops); ++it){
            if (props.length() == 0)
                props = boost::str(boost::format("%1%=%2%") % it->first % it->second);
            else
                props = boost::str(boost::format("%1%=%2%\n%3%") % it->first % it->second % props);
	    }
        mframe[FRAME_PROPS] = szmq::Message::from(props);
        mpresent [FRAME_PROPS] = 1;
        // return props;
    }
    void decodeProps() noexcept{   // string to map
        std::istringstream iss(mframe[FRAME_PROPS].read<std::string>());
        while(!mprops.empty())
            mprops.erase(mprops.begin());
        std::string key, val;
        while(std::getline(std::getline(iss, key, '=') >> std::ws, val))
            mprops.insert(std::make_pair(key, val));
        //return mprops;
    }
    //  This method reads a key-value message from the socket and returns a 
    //  new {{kvmsg}} instance:
    kvmsg recv(szmq::detail::SocketImpl& socket) noexcept {
        kvmsg me(0, mverbose);
        for (int frame_nbr = 0; frame_nbr < KVMSG_FRAMES; frame_nbr++) {
            me.mpresent [frame_nbr] = 1;
            me.mframe[frame_nbr] = socket.recvOne();
            //  Verify multipart framing
            bool rcvmore = (frame_nbr < KVMSG_FRAMES - 1)? true: false;
            if (socket.hasMore() != rcvmore) 
                break;
        }
        me.decodeProps();
        return me;
    }
    //  Send key-value message to socket; any empty frames are sent as such.
    void send(szmq::detail::SocketImpl& socket) noexcept {
        encodeProps();  // make map to string
        for (int frame_nbr = 0; frame_nbr < KVMSG_FRAMES; frame_nbr++) {
            if (frame_nbr < KVMSG_FRAMES - 1)
                socket.sendMore(mframe[frame_nbr]);
            else
                socket.sendOne(mframe[frame_nbr]);
        }
    }
    std::string key(){
        if (mpresent [FRAME_KEY] == 1) {
            mkey = mframe[FRAME_KEY].read<std::string>();
            return mkey;
        }
        else
            return "";
    }
    uint64_t sequence(){
        if (mpresent [FRAME_SEQ] == 1) 
            return mframe [FRAME_SEQ].read<uint64_t>();
        else
            return 0;
    }
    //  Return body size from last read message, if any, else zero
    std::string body(){
        if (mpresent [FRAME_BODY]==1)
            return mframe[FRAME_BODY].read<std::string>();
        else
            return "";
    }
    //  Return uuid
    std::string uuid(){
        if (mpresent [FRAME_UUID]==1)
            return mframe[FRAME_UUID].read<std::string>();
        else
            return "";
    }    
    size_t size() {
        if (mpresent [FRAME_BODY] == 1)
            return mframe [FRAME_BODY].size();
        else
            return 0;
    }
    //  Set message key as provided
    void setKey(std::string key){
        mframe[FRAME_KEY] = szmq::Message::from(key);
        mpresent [FRAME_KEY] = 1;
    }
    //  Set message sequence number
    void setSequence (uint64_t sequence){
        mframe [FRAME_SEQ] = szmq::Message::from(sequence);
        mpresent [FRAME_SEQ] = 1;
    }
    //  Set message body
    void setBody (std::string body){
        mframe [FRAME_BODY] = szmq::Message::from(body);
        mpresent [FRAME_BODY] = 1;
    }

    //  Set message key using printf format
    void fmtKey (char *format, ...){
        char value[KVMSG_KEY_MAX + 1];
        va_list args;
        va_start (args, format);
        vsnprintf (value, KVMSG_KEY_MAX, format, args);
        va_end (args);
        setKey (value);
    }
    //  Set message body using printf format
    void fmtBody (char *format, ...){
        char value[KVMSG_KEY_MAX + 1];
        va_list args;
        va_start (args, format);
        vsnprintf (value, KVMSG_KEY_MAX, format, args);
        va_end (args);
        setBody (value);
    }

    void setUUID(){
        std::string tmp = boost::uuids::to_string(boost::uuids::random_generator()());
        std::string uuidstr =  boost::algorithm::replace_all_copy(tmp, "-", "");
        mframe [FRAME_UUID] = szmq::Message::from(uuidstr);
        mpresent [FRAME_UUID] = 1;
    }
    //  These methods get and set a specified message property:
    //  Get message property, return "" if no such property is defined.
    std::string getProp(std::string name){
        if(mprops.count(name))
            return mprops[name];
        else
            return "";        
    }
    //  Set message property
    void setProp(std::string name, std::string value){
        if(mprops.count(name))
            mprops[name] = value;
        else
            mprops.insert(std::make_pair(name, value));
    }
    //  Store entire kvmsg into hash map, if key/value are set
    //  Nullifies kvmsg reference, and destroys automatically when no longer
    //  needed.
    void store (std::map<std::string, kvmsg> *kvmap){
        kvmsg *self = this;
        if (mpresent [FRAME_KEY] == 1 &&  mpresent [FRAME_BODY] == 1) {
            if(kvmap->count(key()))
                kvmap->at(key()) = *self;
            else
                kvmap->insert(std::make_pair(key(), *self));
        }
    }
    void erase (std::map<std::string, kvmsg> *kvmap){
        kvmsg *self = this;
        if (mpresent [FRAME_KEY] == 1 &&  mpresent [FRAME_BODY] == 1) {
            if(kvmap->count(key()))
                kvmap->erase(self->key());
        }
    }   
    //  Dump message to stderr, for debugging and tracing
    void dump (){
        cout << boost::format("[seq:%1%], ") % sequence();
        cout << boost::format("[key:%1%], ") % key();
        cout << boost::format("[uuid:%1%], [props: ") % uuid();        
        for (auto it = begin (mprops); it != end (mprops); ++it) 
            cout << it->first << "=" << it->second << ";";
        cout << boost::format("], [size:%1%], ") % size();
        cout << boost::format("[body:%1%]\n") % body();
    }


    // Copyable 
    kvmsg(kvmsg const& other) noexcept {
        mverbose = other.mverbose;
        std::copy(other.mframe, other.mframe+KVMSG_FRAMES, mframe);
        std::copy(other.mpresent, other.mpresent+KVMSG_FRAMES, mpresent);
        mkey = other.mkey;
        mprops = other.mprops;
    }

    kvmsg& operator=(kvmsg const& other) noexcept {
        mverbose = other.mverbose;
        std::copy(other.mframe, other.mframe+KVMSG_FRAMES, mframe);
        std::copy(other.mpresent, other.mpresent+KVMSG_FRAMES, mpresent);
        mkey = other.mkey;
        mprops = other.mprops;
        return *this;
    }
    //  Destructor
     ~kvmsg(){
         while(!mprops.empty())
            mprops.erase(mprops.begin());
     };
    //  Runs self test of class
    int test(){
        cout << " * kvmsg: \n";

        //  Prepare our context and sockets
        szmq::Context ctx;
        szmq::Socket<ZMQ_DEALER, szmq::ZMQ_SERVER> output(ctx);
        output.bind(szmq::SocketUrl("inproc://kvmsg_selftest"));
        szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> input(ctx);
        input.connect(szmq::SocketUrl("inproc://kvmsg_selftest"));

        std::map<std::string, kvmsg> kvmap;

        //  Test send and receive of simple message
        kvmsg kvmsg(1);
        kvmsg.setKey("key");
        kvmsg.setUUID();
        kvmsg.setBody("body");
        if (mverbose)
            kvmsg.dump();
        kvmsg.send(output);
        kvmsg.store(&kvmap);
        auto kvmsg1 = kvmsg.recv(input);
        if (mverbose)
            kvmsg1.dump();
        assert (kvmsg1.key().compare("key") == 0);
        kvmsg1.store (&kvmap);
        for (auto it = begin (kvmap); it != end (kvmap); ++it) 
            cout << "kvmap(key) : " << it->first <<", kvmap(body) : " << it->second.body() << endl;
        
        // Test send and receive of message with properties
        kvmsg.setProp ("prop1", "value1");
        kvmsg.setProp ("prop2", "value1");
        kvmsg.setProp ("prop2", "value2");
        kvmsg.setKey  ("key");
        kvmsg.setUUID ();
        kvmsg.setBody ("body");
        assert (kvmsg.getProp ("prop2").compare("value2")==0);
        if (mverbose)
            kvmsg.dump();
        kvmsg.send (output);
        auto kvmsg2 = kvmsg.recv(input);
        if (mverbose)
            kvmsg2.dump ();
        assert (kvmsg2.key().compare("key")==0);
        assert (kvmsg2.getProp("prop2").compare("value2")==0);
        //  Shutdown and destroy all objects
        while(!kvmap.empty())
            kvmap.erase(kvmap.begin());
        output.close();
        input.close();
        cout << "OK\n";
        return 0;
    }     
private :
    int mverbose;
    //  Presence indicators for each frame
    int mpresent [KVMSG_FRAMES];
    //  Corresponding 0MQ message frames, if any
    szmq::Message mframe [KVMSG_FRAMES];
    //  Key, copied into safe C string
    std::string mkey;
    //  map of properties, as name, value strings
    std::map<std::string, std::string> mprops;
};

#endif     
```

* 테스트를 위하여 "kvmsgtest.java" 코드는 다음과 같습니다.

```java
#include <szmq/szmq.h>
#include <iostream>
#include "kvmsg.h"

using namespace std;

int main(){
	kvmsg kvmsg(1,1);
	kvmsg.test();
	return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc kvmsgtest.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./kvmsgtest
 * kvmsg:
[seq:1], [key:key], [uuid:9eb01b7a82c44fc7a39df099a8729df5], [props: ], [size:4], [body:body]
[seq:1], [key:key], [uuid:9eb01b7a82c44fc7a39df099a8729df5], [props: ], [size:4], [body:body]
kvmap(key) : key, kvmap(body) : body
[seq:1], [key:key], [uuid:91241b1e6b3d48bab8fbd23dcae7b0fd], [props: prop1=value1;prop2=value2;], [size:4], [body:body]
[seq:1], [key:key], [uuid:91241b1e6b3d48bab8fbd23dcae7b0fd], [props: prop1=value1;prop2=value2;], [size:4], [body:body]
OK
~~~

모델 5 클라이언트는 모델 4와 거의 동일합니다. 이제 전체 kvmsg 클래스를 사용하고 각 메시지에 임의의 ttl 속성(초 단위로 측정)을 설정합니다.

### 리엑터 사용

지금까지 우리는 서버에서 폴(poll) 루프를 사용했습니다. 다음 서버 모델에서는 리엑터로 전환하여 사용합니다. CZMQ의 zloop 클래스를 사용합니다. 리액터를 사용하면 코드가 더 장황해지지만 서버의 각 부분이 개별 리엑터 핸들러에 의해 처리되기 때문에 쉽게 이해할 수 있습니다.

단일 스레드를 사용하고 리엑터 핸들러에 서버 객체를 전달합니다. 서버를 여러 개의 스레드들로 구성하여 각 스레드는 하나의 소켓 또는 타이머를 처리하지만 스레드가 데이터를 공유하지 않을 경우 더 잘 작동합니다. 이 경우 모든 작업은 서버의 해시 맵을 중심으로 이루어지며 하나의 스레드가 더 간단합니다.

3개의 리엑터 핸들러는 다음과 같습니다.

* 하나는 ROUTER 소켓을 통해 오는 스냅샷 요청들을 처리합니다.
* 하나는 PULL 소켓을 통해 오는 클라이언트들의 변경정보를 처리합니다.
* 하나는 TTL이 경과된 임시값을 만료 처리(값을 공백으로 처리)합니다.

### clonesrv5.java : 복제 서버, 모델 5

```java
//  Clone server - Model Five

//  Lets us build this source without creating a library
#include "kvmsg.h"
#include <czmq.h>
#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>
#include <boost/format.hpp>
using namespace std;

//  Our server is defined by these properties
static int s_snapshots (zloop_t *loop, zmq_pollitem_t *poller, void *args);
static int s_collector (zloop_t *loop, zmq_pollitem_t *poller, void *args);
static int s_flush_ttl (zloop_t *loop, int timer_id, void *args);

//  Routing information for a key-value snapshot
class clonesrv {
public :
    clonesrv(szmq::Context& context, int port, int verbose) 
        : ctx(context),
          port(port),
          snapshot(context),
          publisher(context),
          collector(context),
          mverbose(verbose){
        sequence = 0;
		loop = zloop_new ();
        zloop_set_verbose (loop, mverbose);
	} 
    ~clonesrv() {
        while(!kvmap.empty())
            kvmap.erase(kvmap.begin());
        zloop_destroy(&loop);
        collector.close();
        snapshot.close();
        publisher.close();
    };
    int mverbose;
    szmq::Context& ctx;                //  Context wrapper
    std::map<std::string, kvmsg> kvmap;             //  Key-value store
    zloop_t *loop;              //  zloop reactor
    int port;                   //  Main port we're working on
    uint64_t sequence;           //  How many updates we're at
    szmq::Socket<ZMQ_ROUTER, szmq::ZMQ_SERVER> snapshot;             //  Handle snapshot requests
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher;            //  Publish updates to clients
    szmq::Socket<ZMQ_PULL, szmq::ZMQ_SERVER> collector;            //  Collect updates from clients  
};

//  We handle ICANHAZ? requests by sending snapshot data to the
//  client that requested it:
//  Routing information for a key-value snapshot
typedef struct {
    szmq::detail::SocketImpl& socket;           //  ROUTER socket to send to
    std::string identity;     //  Identity of peer who requested state
    std::string subtree;          //  Client subtree specification
} kvroute_t;

//  Send one state snapshot key-value pair to a socket
//  Hash item data is our kvmsg object, ready to send
static int
s_send_single (std::string key, kvmsg data, kvroute_t kvroute)
{
    if(kvroute.subtree.length() <= data.key().length() && 
     kvroute.subtree.compare(data.key().substr(0, kvroute.subtree.length()))==0){
         //  Send identity of recipient first
        kvroute.socket.sendMore(szmq::Message::from(kvroute.identity));
        data.send(kvroute.socket);
     }
    return 0;
}

int main (int argc, char *argv [])
{
    int verbose = (argc > 1 && strcmp (argv [1], "-v") == 0);
    szmq::Context context;
    clonesrv self(context, 5556, verbose);    
    
    //  Set up our clone server sockets
    self.snapshot.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % self.port)));
    self.publisher.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % (self.port + 1))));
    self.collector.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % (self.port + 2))));
    //  Register our handlers with reactor
    zmq_pollitem_t poller = { 0, 0, ZMQ_POLLIN };
    poller.socket = reinterpret_cast<void*>(*self.snapshot);
    zloop_poller (self.loop, &poller, s_snapshots, (void *)&self);
    poller.socket = reinterpret_cast<void*>(*self.collector);
    zloop_poller (self.loop, &poller, s_collector, (void *)&self);
    zloop_timer (self.loop, 1000, 0, s_flush_ttl, (void *)&self);

    //  Run reactor until process interrupted
    zloop_start (self.loop);
    return 0;
}

//  .split snapshot handler
//  This is the reactor handler for the snapshot socket; it accepts
//  just the ICANHAZ? request and replies with a state snapshot ending
//  with a KTHXBAI message:

static int
s_snapshots (zloop_t *loop, zmq_pollitem_t *poller, void *args)
{
    clonesrv *self = (clonesrv *)args;

    auto identity = self->snapshot.recvOne().read<std::string>();
    if (identity.length() > 0) {
        //  Request is in second frame of message
        auto request = self->snapshot.recvOne().read<std::string>();
        std::string subtree = "";
        if (request.compare("ICANHAZ?")==0) {
            subtree = self->snapshot.recvOne().read<std::string>();
        }
        else
            cout << boost::format("E: bad request (%1%), aborting\n") % request;

        if (subtree.length() > 0) {
            //  Send state socket to client
            kvroute_t routing = { self->snapshot, identity, subtree };
            for (auto it = begin (self->kvmap); it != end (self->kvmap); ++it) 
                s_send_single(it->first, it->second, routing);

            //  Now send END message with sequence number
            cout << boost::format("I: sending shapshot=%1%\n") % (int) self->sequence;
            self->snapshot.sendMore(szmq::Message::from(identity));
            kvmsg kvmsg(self->sequence);
            kvmsg.setKey("KTHXBAI");
            kvmsg.setBody("");
            kvmsg.send(self->snapshot);
        }
    }
    return 0;
}

//  .split collect updates
//  We store each update with a new sequence number, and if necessary, a
//  time-to-live. We publish updates immediately on our publisher socket:

static int
s_collector (zloop_t *loop, zmq_pollitem_t *poller, void *args)
{
    clonesrv *self = (clonesrv *)args;

    kvmsg kvmsg = kvmsg.recv(self->collector);
    kvmsg.setSequence (++self->sequence);
    kvmsg.send (self->publisher);
    // cout << "s_collector() : " << kvmsg.getProp("ttl") << endl;
    kvmsg.dump();
    uint64_t ttl = stoll (kvmsg.getProp ("ttl"));
    if (ttl)
        kvmsg.setProp ("ttl", boost::str(boost::format("%1%") % (szmq::now()+ttl*1000)));
    kvmsg.store (&self->kvmap);
    cout << boost::format("I: publishing update=%1%\n") % (int) self->sequence;
    return 0;
}

//  .split flush ephemeral values
//  At regular intervals, we flush ephemeral values that have expired. This
//  could be slow on very large data sets:

//  If key-value pair has expired, delete it and publish the
//  fact to listening clients.
static int
s_flush_single (std::string key, kvmsg data, void *args)
{
    clonesrv *self = (clonesrv *)args;
    //cout << "s_flush_single() : " << data.getProp("ttl") << endl;
    //data.dump();
    uint64_t ttl = stoll(data.getProp("ttl"));
    if (ttl && szmq::now () >= ttl) {
        data.setSequence (++self->sequence);
        data.setBody ("");
        data.send (self->publisher);
        data.store(&self->kvmap);
        //data.erase (&self->kvmap);
        cout << boost::format("I: publishing delete=%1%\n") % (int) self->sequence;
    }
    return 0;
}

static int
s_flush_ttl (zloop_t *loop, int timer_id, void *args)
{
    clonesrv *self = (clonesrv *)args;
    if (&self->kvmap)
        for (auto it = begin (self->kvmap); it != end (self->kvmap); ++it) {
            s_flush_single(it->first, it->second, args );
        }
    return 0;
}
```

kvmsg클래스에 TTL 속성을 부가한 클라이언트 소스는 다음과 같습니다.

### clonecli5.java : 복제 클라이언트, 모델 5

```java
//  Clone client - Model Three

//  Lets us build this source without creating a library
#include "kvmsg.h"

#define SUBTREE "/client/"

int main (int argc, char *argv [])
{
    int verbose = (argc > 1 && strcmp (argv [1], "-v") == 0);
    //  Prepare our context and subscriber
    szmq::Context ctx;
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> snapshot(ctx);
    snapshot.connect(szmq::SocketUrl("tcp://localhost:5556"));
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber(ctx);
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    subscriber.connect(szmq::SocketUrl("tcp://localhost:5557"));
    subscriber.setSockOpt(ZMQ_SUBSCRIBE, SUBTREE, 8);
    szmq::Socket<ZMQ_PUSH, szmq::ZMQ_CLIENT> publisher(ctx);
    publisher.connect(szmq::SocketUrl("tcp://localhost:5558"));

    std::map<std::string, kvmsg> kvmap;
    srand (static_cast<unsigned int>(std::time(0)));

    //  .split getting a state snapshot
    //  We first request a state snapshot:
    uint64_t sequence = 0;
    snapshot.sendMore(szmq::Message::from("ICANHAZ?"));
    snapshot.sendOne(szmq::Message::from(SUBTREE));
    while (true) {
        kvmsg kvmsg = kvmsg.recv(snapshot);
        if(kvmsg.key().compare("KTHXBAI") == 0) {
            sequence = kvmsg.sequence();
            cout << "Received snapshot=" << (int)sequence << endl;
            break;
        }
        kvmsg.store(&kvmap);
    }
    //  .split processing state updates
    //  Now we wait for updates from the server and every so often, we
    //  send a random key-value update to the server:
    uint64_t alarm = szmq::now() + 1000;
    while (true) {
        std::vector<szmq::PollItem> items = {
            {reinterpret_cast<void*>(*subscriber), 0, ZMQ_POLLIN, 0}};
        uint64_t tickless = (int) ((alarm - szmq::now()));
        if (tickless < 0)
            tickless = 0;
        szmq::poll (items, 1, tickless);
        if (items [0].revents & ZMQ_POLLIN) {
            kvmsg kvmsg = kvmsg.recv (subscriber);
            if (verbose) kvmsg.dump();
            //  Discard out-of-sequence kvmsgs, incl. heartbeats
            if (kvmsg.sequence () > sequence) {
                sequence = kvmsg.sequence ();
                kvmsg.store (&kvmap);
                cout << boost::format("I: received update=%1%\n") % (int) sequence;
            }
        }
        //  If we timed out, generate a random kvmsg
        if (szmq::now() >= alarm) {
            kvmsg kvmsg(0);
            kvmsg.setKey(boost::str(boost::format("%1%%2%") % SUBTREE % (rand() % 10000)));
            kvmsg.setBody(boost::str(boost::format("%1%") %(rand() % 1000000)));
            kvmsg.setProp("ttl", boost::str(boost::format("%1%") % (rand() % 30)));
            kvmsg.send(publisher);
            alarm = szmq::now() + 1000;
        }
    }
    cout << boost::format(" Interrupted\n%1% messages out\n") % (int) sequence;
    while(!kvmap.empty())
        kvmap.erase(kvmap.begin());
    subscriber.close();
    snapshot.close();
    publisher.close();
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
// 원도우
PS D:\work\sook\src\szmq\examples> cl -EHsc clonesrv5.java szmq.lib czmq.lib
PS D:\work\sook\src\szmq\examples> cl -EHsc clonecli5.java szmq.lib

PS D:\work\sook\src\szmq\examples> ./clonesrv5
[seq:1], [key:/client/7454], [uuid:], [props: ttl=10;], [size:5], [body:20506]
I: publishing update=1
[seq:2], [key:/client/3304], [uuid:], [props: ttl=23;], [size:5], [body:31294]
I: publishing update=2
[seq:3], [key:/client/6705], [uuid:], [props: ttl=15;], [size:5], [body:24539]
I: publishing update=3

PS D:\work\sook\src\szmq\examples> ./clonecli5 -v
Received snapshot=0
[seq:1], [key:/client/833], [uuid:], [props: ttl=16;], [size:5], [body:24623]
I: received update=1
[seq:1], [key:/client/8028], [uuid:], [props: ttl=20;], [size:5], [body:24017]
[seq:2], [key:/client/1569], [uuid:], [props: ttl=0;], [size:5], [body:24831]
I: received update=2
[seq:3], [key:/client/4847], [uuid:], [props: ttl=11;], [size:4], [body:4950]
I: received update=3

// 리눅스
zedo@sook:/work/sook/src/szmq/examples$ g++ -o clonesrv5 clonesrv5.java -lsook-szmq -lzmq -lczmq
zedo@sook:/work/sook/src/szmq/examples$ g++ -o clonecli5 clonecli5.java -lsook-szmq -lzmq

zedo@sook:/work/sook/src/szmq/examples$ ./clonesrv5 -v
I: sending shapshot=0
I: publishing update=1
I: publishing update=2
I: publishing update=3

zedo@sook:/work/sook/src/szmq/examples$ ./clonecli5 -v
Received snapshot=0
[seq:1], [key:/client/596], [uuid:], [props: ttl=12;], [size:6], [body:438481]
I: received update=1
[seq:2], [key:/client/6536], [uuid:], [props: ttl=12;], [size:6], [body:286243]
I: received update=2
[seq:3], [key:/client/4332], [uuid:], [props: ttl=26;], [size:6], [body:266366]
I: received update=3
[seq:4], [key:/client/3293], [uuid:], [props: ttl=27;], [size:6], [body:916440]
~~~


### 신뢰성을 위한 바이너리 스타 패턴 추가

지금까지 살펴본 복제 모델은 비교적 간단했습니다. 이제 우리는 불쾌하고 복잡한 영역에 들어가서 다른 에스프레소를 마시게 될 것입니다. “신뢰성 있는” 메시징을 만드는 것이 복잡한 만큼 “실제로 이것이 필요한가?”라는 의문이 있어야 합니다. “신뢰성이 없는” 혹은 “충분히 좋은 신뢰성을 가진” 상황에서 벗어날 수 있다면 비용과 복잡성 측면에서 큰 승리를 한 것입니다. 물론 때때로 일부 데이터가 유실할 수 있지만 좋은 절충안입니다. 하지만 한 모금의 에스프레소는 정말 좋습니다. 복잡성의 세계로 뛰어들어 가겠습니다.

이전 모델(리엑터와 TTL)로 작업할 때 서버를 중지하고 재시작하면 복구된 것처럼 보이지만 물론 적절한 현재 상태 대신 빈 상태에 변경정보들을 반영합니다. 네트워크에 가입하는 모든 신규 클라이언트는 전체 이력 데이터 대신 최신 변경정보만 받습니다.

우리가 서버가 죽거나 충돌하면 복구하는 방법이 필요합니다. 또한 서버가 일정 시간 동안 작동하지 않는 경우 백업을 제공해야 합니다. 누군가 “신뢰성” 구현이 필요하면 우선 처리해야 하는 장애들에 대한 목록 요구해야 하며, 우리의 경우 다음과 같습니다.

* 서버 프로세스가 충돌하여 자동 또는 수동으로 재시작됩니다. 프로세스는 상태를 잃고 어딘가에서 다시 가져와야 합니다.
* 서버 머신(하드웨어)이 죽고 상당한 시간 동안 오프라인 상태입니다. 클라이언트는 어딘가의 대체 서버로 전환해야 합니다.
* 서버 프로세스 또는 머신이 네트워크에서 연결 해제됩니다(예 : 스위치가 죽거나 데이터센터가 무응답). 언제든지 정상화될 수 있지만 그동안 클라이언트들은 대체 서버가 필요합니다.

첫 번째 단계는 대체 서버를 추가하는 것입니다."4장 - 신뢰할 수 있는 요청-응답 패턴"의 바이너리 스타 패턴을 사용하여 기본 및 백업으로 구성할 수 있습니다. 바이너리 스타는 리엑터를 사용하므로 이전 서버 모델(리엑터+TTL)을 재구성하기에는 유용합니다.

기본 서버가 충돌하더라도 변경정보들이 손실되지 않도록 해야 합니다. 가장 간단한 방법은 클라이언트의 상태 변경정보를 2개 서버(기본-백업)에 모두 보내는 것입니다. 그런 다음 백업 서버는 클라이언트 역할을 수행하며, 모든 클라이언트가 수행하는 것처럼 서버로부터 변경정보들을 수신하여 상태를 동기화합니다. 또한 백업 서버는 클라이언트로부터 새로운 변경정보를 받지만 해시테 이블에 저장하지 않고 잠시 동안 가지고 있습니다.

따라서 모델 6은 모델 5에 비해 다음과 같은 변경 사항이 적용되었습니다.

* 클라이언트가 서버로 전송하는 상태 변경정보에 PUSH-PULL 패턴 대신 PUB-SUB 패턴을 사용합니다. 이렇게 하면 2대의 서버로 변경정보가 동일하게 펼쳐져 갈 수 있습니다. 그렇지 않으면 2개의 DEALER 소켓들을 사용해야 합니다.
* 서버 변경정보 메시지(클라이언트로 가는)에 심박을 추가하여, 클라이언트가 기본 서버의 죽음을 감지하게 하여, 클라이언트가 백업 서버로 전환하게 합니다.
* 2대의 서버를 바이너리 스타 bstar 리엑터 클래스를 사용하여 연결합니다. 바이너리 스타는 클라이언트들이 활성화 상태로 여기는 서버에 명시적인 요청하는 투표에 의존합니다. 투표 메커니즘으로 스냅샷 요청들을 사용할 것입니다.
* 모든 변경정보 메시지에 고유의 식별자로 UUID 필드를 추가하였습니다. 클라이언트는 상태 변경정보에서 생성하고, 서버는 다시 클라이언트에서 변경정보를 받아하여 전파합니다.
* 비활성 서버는 "보류 목록(Pending list)"에 
  - 클라이언트에서 수신되었지만 아직 활성 서버에서는 수신되지 않은 변경정보들을 저장하거나, 
  - 활성 서버에서는 수신되었지만 클라이언트에서는 아직 수신되지 않은 변경정보들을 저장합니다.
  - 보류 목록은 오래된 것부터 최신 순서로 정렬되어 있으므로 최신 것부터 변경정보들을 쉽게 제거할 수 있습니다.

  클라이언트 로직을 유한 상태 머신으로 설계하는 것이 유용합니다. 클라이언트는 다음 3가지 상태(INITIAL, SYNCING, ACTIVE)를 순환합니다.

* [INITIAL] 클라이언트는 소켓(SUB, DEALER, PUB)을 열고 연결하고, 첫 번째 서버에서 스냅샷을 요청합니다. 클라이언트들의 요청 폭주를 방지하기 위해 주어진 서버에 2번만 요청합니다. 하나의 요청이 유실되는 것은 불행이지만, 두 번째까지 유실되는 것은 부주의입니다.
* [SYNCING] 클라이언트는 서버로부터 응답(스냅샷 데이터)을 기다렸다가 수신하여 저장합니다. 정해진 제한 시간 내에 서버로부터 응답이 없으면 다음 서버로 장애조치합니다.
* [ACTIVE] 클라이언트가 스냅샷을 받고, 서버의 변경정보들을 기다렸다가 처리합니다. 
다시 정해진 제한 시간(timeout)에 서버로부터 응답이 없으면 다음 서버로 장애조치합니다.

클라이언트는 영원히 반복됩니다. 시작 또는 장애조치 중에 일부 클라이언트들은 기본 서버와 통신을 시도하고 다른 클라이언트들은 백업 서버와 통신을 시도할 것입니다. 바이너리 스타 상태 머신은 이것을 정확하게 처리합니다. 소프트웨어가 정확하다는 것을 증명하는 것은 어렵지만 대신에 우리는 소프트웨어가 틀렸다는 것을 증명할 수 없을 때까지 두드려 봅니다.

장애조치는 다음과 같이 이루어집니다. 

* 클라이언트는 기본 서버가 더 이상 심박를 보내지 않음을 감지하고 죽었다는 결론을 내립니다. 클라이언트는 백업 서버에 연결하고 신규 상태 스냅샷을 요청합니다.
* 백업 서버는 클라이언트로부터 스냅샷 요청을 받기 시작하고 기본 서버의 죽음을 감지하여 기본 서버로 인계합니다.
* 백업 서버는 "보류 목록"을 자신의 해시 테이블에 적용한 다음, 클라이언트의 상태 스냅샷 요청을 처리하기 시작합니다.

기본 서버가 다시 온라인 상태가 되면 다음을 수행합니다.

* 비활성 상태로 서버를 시작하고, 복제 클라이언트로 백업 서버에 연결합니다.
* SUB 소켓을 통해 클라이언트들로부터 변경정보들을 수신하기 시작합니다.

몇 가지 가정들은 다음과 같습니다.

* 하나 이상의 서버가 계속 실행됩니다. 2대의 서버들이 충돌하면 모든 서버 상태가 유실되고 복구할 방법이 없습니다.
* 여러 클라이언트들이 동일한 해시 테이블의 키를 동시에 변경하지 않습니다. 클라이언트들의 변경정보들은 다른 순서로 2대 서버들에 도달합니다. 따라서 백업 서버는 기본 서버와 다른 순서의 변경정보들을 보류 목록(Pending List)의 반영할 수 있습니다. 한 클라이언트의 변경정보들은 항상 동일한 순서로 2대 서버들에 도달하므로 안전합니다.

따라서 바이너리 스티 패턴을 사용하는 고가용성 서버 쌍의 아키텍처에는 2대의 서버와 서버들과 통신하는 일련의 클라이언트들이 있습니다.

여기에 6번째로 복제 서버의 마지막 모델(모델 6)의 코드입니다.

### clonesrv6.java : 복제 서버, 모델 6

```java
//  Clone server - Model Five

//  Lets us build this source without creating a library
#include "bstar.h"
#include "kvmsg.h"
#include <czmq.h>
#include <szmq/szmq.h>
#include <iostream>
#include <cstdlib>
#include <list>
#include <boost/format.hpp>
using namespace std;

//  Our server is defined by these properties
// Bstar reactor handlers
static int s_snapshots (zloop_t *loop, zmq_pollitem_t *poller, void *args);
static int s_collector (zloop_t *loop, zmq_pollitem_t *poller, void *args);
static int s_flush_ttl (zloop_t *loop, int timer_id, void *args);
static int s_send_hugz (zloop_t *loop, int timer_id, void *args);
static int s_new_active(zloop_t *loop, zmq_pollitem_t *poller, void *args);
static int s_new_passive(zloop_t *loop, zmq_pollitem_t *poller, void *args);
static int s_subscriber(zloop_t *loop, zmq_pollitem_t *poller, void *args);

//  Routing information for a key-value snapshot
class clonesrv {
public :
    clonesrv(szmq::Context& context, int verbose) 
        : ctx(context),
          port(port),
          publisher(context),
          collector(context),
          subscriber(context),
          snapshot(context),
          mverbose(verbose){
        sequence = 0;
		loop = zloop_new ();
        zloop_set_verbose (loop, mverbose);
	} 
    ~clonesrv() {
        while(!kvmap.empty())
            kvmap.erase(kvmap.begin());
        zloop_destroy(&loop);
        publisher.close();      
        collector.close();
        subscriber.close();

    };
    int mverbose;
    szmq::Context& ctx;                //  Context wrapper
    std::map<std::string, kvmsg> kvmap;             //  Key-value store
    bstar *bstr;        //binary star server class(Bstar reactor core)
    zloop_t *loop;              //  zloop reactor
    int port;                   //  Main port we're working on
    int peer;                   //  Main port of our peer
    uint64_t sequence;           //  How many updates we're at
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_SERVER> publisher;    //  Publish updates to clients
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_SERVER> collector;   //  Collect updates from clients  
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber;   //  Handle snapshot requests
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> snapshot; 
    std::list<kvmsg> pending;                             //  Pending updates from clients
    bool primary;               //  true if we're primary
    bool active;                //  true if we're active
    bool passive;               //  true if we're passive
};


int main (int argc, char *argv [])
{
    int verbose = (argc > 1 && (strcmp (argv [1], "-v") == 0 || strcmp (argv [2], "-v") == 0));
    szmq::Context context;
    clonesrv self(context, verbose);    
    if (argc > 1 && strcmp (argv [1], "-p") == 0) {
        zclock_log ("I: primary active, waiting for backup (passive)");
        self.bstr = new bstar(context, BSTAR_PRIMARY, "tcp://*:5003",
                                 "tcp://localhost:5004", verbose);
        self.bstr->voter("tcp://*:5556", s_snapshots, (void *)&self);
        self.port = 5556;
        self.peer = 5566;
        self.primary = true;
    }
    else
    if (argc > 1 && strcmp (argv [1], "-b") == 0) {
        zclock_log ("I: backup passive, waiting for primary (active)");
        self.bstr = new bstar(context, BSTAR_BACKUP, "tcp://*:5004",
                                 "tcp://localhost:5003", verbose);
        self.bstr->voter("tcp://*:5566", s_snapshots, (void *)&self);
        self.port = 5566;
        self.peer = 5556;
        self.primary = false;
    }
    else {
        zclock_log("Usage: clonesrv6 { -p | -b }");
        exit (0);
    }
    self.bstr->setVerbose(true);

    //  Set up our clone server sockets
    self.collector.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    self.publisher.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % (self.port + 1))));
    self.collector.bind(szmq::SocketUrl(boost::str(boost::format("tcp://*:%1%") % (self.port + 2))));

    //  Set up our own clone client interface to peer
    self.subscriber.setSockOpt(ZMQ_SUBSCRIBE, "", 0);
    self.subscriber.connect(szmq::SocketUrl(boost::str(boost::format("tcp://localhost:%1%") % (self.peer + 1))));

    //  .split main task body
    //  After we've setup our sockets, we register our binary star
    //  event handlers, and then start the bstar reactor. This finishes
    //  when the user presses Ctrl-C or when the process receives a SIGINT
    //  interrupt:

    //  Register state change handlers
    self.bstr->newActive(s_new_active, (void *)&self);
    self.bstr->newPassive (s_new_passive, (void *)&self);


    //  Register our other handlers with the bstar reactor
    zmq_pollitem_t poller = { reinterpret_cast<void*>(*self.collector), 0, ZMQ_POLLIN };
    // poller.socket = reinterpret_cast<void*>(*self.collector);
    zloop_poller (self.bstr->loop, &poller, s_collector, (void *)&self);
    zloop_timer (self.bstr->loop, 1000, 0, s_flush_ttl, (void *)&self);
    zloop_timer (self.bstr->loop, 1000, 0, s_send_hugz, (void *)&self);

    //  Start the bstar reactor
    self.bstr->start();
    //  Interrupted, so shut down
    while(!self.pending.empty())
        self.pending.pop_back();
    delete (self.bstr);
    return 0;
}

//  We handle ICANHAZ? requests by sending snapshot data to the
//  client that requested it:
//  Routing information for a key-value snapshot
typedef struct {
    szmq::detail::SocketImpl& socket;           //  ROUTER socket to send to
    std::string identity;     //  Identity of peer who requested state
    std::string subtree;          //  Client subtree specification
} kvroute_t;

//  Send one state snapshot key-value pair to a socket
//  Hash item data is our kvmsg object, ready to send
static int
s_send_single (std::string key, kvmsg data, kvroute_t kvroute)
{
    if(kvroute.subtree.length() <= data.key().length() && 
     kvroute.subtree.compare(data.key().substr(0, kvroute.subtree.length()))==0){
         //  Send identity of recipient first
        kvroute.socket.sendMore(szmq::Message::from(kvroute.identity));
        data.send(kvroute.socket);
     }
    return 0;
}

//  .split snapshot handler
//  This is the reactor handler for the snapshot socket; it accepts
//  just the ICANHAZ? request and replies with a state snapshot ending
//  with a KTHXBAI message:

static int
s_snapshots (zloop_t *loop, zmq_pollitem_t *poller, void *args)
{
    clonesrv *self = (clonesrv *)args;

    auto identity = self->bstr->frontend.recvOne().read<std::string>();
    if (identity.length() > 0) {
        //  Request is in second frame of message
        auto request = self->bstr->frontend.recvOne().read<std::string>();
        std::string subtree = "";
        if (request.compare("ICANHAZ?")==0) {
            subtree = self->bstr->frontend.recvOne().read<std::string>();
        }
        else
            cout << boost::format("E: bad request (%1%), aborting\n") % request;

        if (subtree.length() > 0) {
            //  Send state socket to client
            kvroute_t routing = { self->bstr->frontend, identity, subtree };
            for (auto it = begin (self->kvmap); it != end (self->kvmap); ++it) 
                s_send_single(it->first, it->second, routing);

            //  Now send END message with sequence number
            cout << boost::format("I: sending shapshot=%1%\n") % (int) self->sequence;
            self->bstr->frontend.sendMore(szmq::Message::from(identity));
            kvmsg kvmsg(self->sequence);
            kvmsg.setKey("KTHXBAI");
            kvmsg.setBody("");
            kvmsg.send(self->bstr->frontend);
        }
    }
    return 0;
}

//  .split collect updates
//  The collector is more complex than in the clonesrv5 example because the 
//  way it processes updates depends on whether we're active or passive. 
//  The active applies them immediately to its kvmap, whereas the passive 
//  queues them as pending:

//  If message was already on pending list, remove it and return true,
//  else return false.
static bool
s_was_pending (clonesrv *self, kvmsg msg)
{
    kvmsg held = self->pending.front();
    for (auto it = begin (self->pending); it != end (self->pending); ++it){
        if(msg.uuid().compare(it->uuid()) == 0) {
            self->pending.erase(it);
            return true;
        }
    }
    return false;
}

static int
s_collector (zloop_t *loop, zmq_pollitem_t *poller, void *args)
{
    clonesrv *self = (clonesrv *)args;

    kvmsg kvmsg = kvmsg.recv(self->collector);
    if (self->active) {
        kvmsg.setSequence (++self->sequence);
        kvmsg.send (self->publisher);
        uint64_t ttl = stoll (kvmsg.getProp ("ttl"));
        if (ttl)
            kvmsg.setProp ("ttl", boost::str(boost::format("%1%") % (szmq::now()+ttl*1000)));
        kvmsg.store (&self->kvmap);
        zclock_log ("I: publishing update=%d", (int) self->sequence);
    }
    else{
        //  If we already got message from active, drop it, else
        //  hold on pending list
        if (!s_was_pending (self, kvmsg))
            self->pending.emplace_back(kvmsg);
    }
    return 0;
}

//  .split flush ephemeral values
//  At regular intervals, we flush ephemeral values that have expired. This
//  could be slow on very large data sets:

//  If key-value pair has expired, delete it and publish the
//  fact to listening clients.
static int
s_flush_single (std::string key, kvmsg data, void *args)
{
    clonesrv *self = (clonesrv *)args;
    //cout << "s_flush_single() : " << data.getProp("ttl") << endl;
    //data.dump();
    uint64_t ttl = stoll(data.getProp("ttl"));
    if (ttl && szmq::now () >= ttl) {
        data.setSequence (++self->sequence);
        data.setBody ("");
        data.send (self->publisher);
        data.store(&self->kvmap);
        //data.erase (&self->kvmap);
        zclock_log ("I: publishing delete=%d", (int) self->sequence);
    }
    return 0;
}

static int
s_flush_ttl (zloop_t *loop, int timer_id, void *args)
{
    clonesrv *self = (clonesrv *)args;
    if (&self->kvmap)
        for (auto it = begin (self->kvmap); it != end (self->kvmap); ++it) {
            s_flush_single(it->first, it->second, args );
        }
    return 0;
}

//  .split heartbeating
//  We send a HUGZ message once a second to all subscribers so that they
//  can detect if our server dies. They'll then switch over to the backup
//  server, which will become active:
static int
s_send_hugz (zloop_t *loop, int timer_id, void *args)
{
    clonesrv *self = (clonesrv *) args;

    kvmsg msg(self->sequence);
    msg.setKey ("HUGZ");
    msg.setBody("");
    msg.send(self->publisher);
    return 0;
}

//  .split handling state changes
//  When we switch from passive to active, we apply our pending list so that
//  our kvmap is up-to-date. When we switch to passive, we wipe our kvmap
//  and grab a new snapshot from the active server:

static int
s_new_active (zloop_t *loop, zmq_pollitem_t *unused, void *args)
{
    clonesrv *self = (clonesrv *) args;

    self->active = true;
    self->passive = false;

    //  Stop subscribing to updates
    zmq_pollitem_t poller = { reinterpret_cast<void*>(*self->subscriber), 0, ZMQ_POLLIN };
    zloop_poller_end (self->bstr->loop, &poller);

    //  Apply pending list to own hash table
    while(self->pending.size()){
        kvmsg msg = self->pending.front(); self->pending.pop_front();
        msg.setSequence(++self->sequence);
        msg.send(self->publisher);
        msg.store(&self->kvmap);
        zclock_log ("I: publishing pending=%d", (int) self->sequence);
    }
    return 0;
}

static int
s_new_passive (zloop_t *loop, zmq_pollitem_t *unused, void *args)
{
    clonesrv *self = (clonesrv *) args;

    while(!self->kvmap.empty())
        self->kvmap.erase(self->kvmap.begin());
    self->active = false;
    self->passive = true;

    //  Start subscribing to updates
    zmq_pollitem_t poller = { reinterpret_cast<void*>(*self->subscriber), 0, ZMQ_POLLIN };
    zloop_poller (self->bstr->loop, &poller, s_subscriber, self);

    return 0;
}

//  .split subscriber handler
//  When we get an update, we create a new kvmap if necessary, and then
//  add our update to our kvmap. We're always passive in this case:

static int
s_subscriber (zloop_t *loop, zmq_pollitem_t *poller, void *args)
{
    clonesrv *self = (clonesrv *) args;
    //  Get state snapshot if necessary
    if (self->kvmap.size() == 0) {
        self->snapshot.connect(szmq::SocketUrl(boost::str(boost::format("tcp://localhost:%1%") % self->peer)));
        zclock_log ("I: asking for snapshot from: tcp://localhost:%d",self->peer);
        self->snapshot.sendMore(szmq::Message::from("ICANHAZ?"));
        self->snapshot.sendOne(szmq::Message::from("")); // blank subtree to get all
        while (true) {
            kvmsg msg = msg.recv(self->snapshot);
            if (msg.key().compare("KTHXBAI") == 0) {
                self->sequence = msg.sequence();
                break;          //  Done
            }
            msg.store(&self->kvmap);
        }
        zclock_log ("I: received snapshot=%d", (int) self->sequence);
    }
    //  Find and remove update off pending list
    kvmsg msg = msg.recv(self->subscriber);
    if (msg.key().compare("HUGZ") != 0) {
        if (!s_was_pending (self, msg)) {
            //  If active update came before client update, flip it
            //  around, store active update (with sequence) on pending
            //  list and use to clear client update when it comes later
            self->pending.emplace_back(msg);
        }
        //  If update is more recent than our kvmap, apply it
        if (msg.sequence() > self->sequence) {
            self->sequence = msg.sequence();
            msg.store (&self->kvmap);
            zclock_log ("I: received update=%d", (int) self->sequence);
        }
    }

    return 0;
}
```

이 모델은 수백 줄의 코드에 불과하지만 작동하기 위해 많은 시간이 소요되었습니다. 정확히 말하면 모델 6을 구현하는데 대략 1주일이 걸렸으며 "신이시여, 이건 예제로 하기에는 너무 복잡합니다." 하며 삽질하였습니다. 우리는 작은 응용프로그램에 지금까지 적용한 거의 개념들을 모아 조립하였습니다. 바이너리 스타, TTL, 유한 상태 머신, 리엑터, 장애조치, 임시값, 하위트리 등이 있습니다. 나를 놀라게 한 것은 앞선 설계가 꽤 정확하다는 것입니다. 여전히 많은 소켓 흐름을 작성하고 디버깅하는 것은 매우 어렵습니다.

리엑터 기반 설계는 코드에서 많은 지저분한 작업을 제거하여, 남은 것을 더 간단하고 이해하기 쉽게 합니다. "4장 - 신뢰할 수 있는 요청-응답 패턴"의 bstar 리액터를 재사용하였습니다. 전체 서버가 하나의 스레드로 실행되므로 스레드 간 이상함이 일어나지 않습니다 - 모든 핸들러들에 전달된 구조체 포인터(self)만으로 즐거운 작업을 수행합니다. 리엑터 사용 시 하나의 좋은 효과는 폴링 루프와 약연결(loosely coupled)된 코드는 재사용하기 훨씬 쉽다는 것입니다. 모델 6의 큰 덩어리는 모델 5에서 가져왔습니다.

* "clonecli6.c" 상태(INITIAL, SYNCING, ACTIVE)를 PUSH-PULL을 PUB-SUB로 변경한 클라이언트의 마지막 모델(모델 6)로 다음 주제에서 설명합니다.

### 빌드 및 테스트

~~~{.bash}
PS D:\work\sook\src\szmq\examples> cl -EHsc clonesrv6.java libzmq.lib czmq.lib

PS D:\work\sook\src\szmq\examples> ./clonesrv6 -p -v
20-11-19 07:53:02 I: primary active, waiting for backup (passive)
20-11-19 07:53:02 +++++ voter()
20-11-19 07:53:02 ----- voter()
...
D: 20-11-19 07:53:12 zloop: call SUB socket handler (000001EECE69B6C0, 0)
20-11-19 07:53:12 I: connected to backup (passive), ready as active
D: 20-11-19 07:53:12 zloop: cancel SUB poller (000001EECE7F0730, 0)
D: 20-11-19 07:53:12 zloop polling for 742 msec
...

PS D:\work\sook\src\szmq\examples> ./clonesrv6 -b -v
20-11-19 07:53:11 I: backup passive, waiting for primary (active)
20-11-19 07:53:11 +++++ voter()
20-11-19 07:53:11 ----- voter()
...
D: 20-11-19 07:53:12 zloop: call SUB socket handler (000001CD6DCEAB00, 0)
20-11-19 07:53:12 I: connected to primary (active), ready as passive
D: 20-11-19 07:53:12 zloop: register SUB poller (000001CD6DEC1A40, 0)
D: 20-11-19 07:53:12 zloop polling for 222 msec
...
~~~

### 클러스터된 해시맵 통신규약

모델 6 서버는 이전 모델(모델 5(임시값 + TTL))에 바이너리 스타 패턴을 매우 많이 혼합시켰지만, 모델 6 클라이언트는 좀 더 많이 복잡합니다. 클라이언트에 대해 알아보기 전에 최종 통신규약을 보면, 클러스터된 해시맵 통신규약으로 ØMQ RFC 사이트에 사양서로 작성해 두었습니다.

대략적으로 이와 같은 복잡한 통신규약을 설계하는 방법에는 두 가지가 있습니다. 첫 번째 방법은 각 흐름을 자체 소켓 집합으로 분리하는 것입니다. 이것이 우리가 여기서 사용한 접근 방식입니다. 장점은 각 흐름이 단순하고 깔끔하다는 것입니다. 단점은 한 번에 다중 소켓 흐름을 관리하는 것이 매우 복잡할 수 있습니다. 리엑터를 사용하면 더 단순해지지만 그래도 정상적으로 동작하게 하기 위해 함께 조정할 부분들이 많습니다.

두 번째 방법은 모든 것에 단일 소켓 쌍을 사용하는 것입니다. 이 경우 서버에는 ROUTER를, 클라이언트들에는 DEALER를 사용하여 해당 연결상에서 모든 작업을 수행했습니다. 이러한 방법은 더 복잡한 통신규약을 만들지만 적어도 복잡성은 모두 한곳에 집중되어 있습니다. “7장 - ØMQ 활용한 고급 아키텍처”에서 ROUTER-DEALER 조합을 통해 수행되는 통신규약의 예를 살펴보겠습니다.

클라이언트는 스냅샷 연결(포트 번호 P)에 “ICANHAZ?” 명령을 전송하여 시작해야 합니다. 이 명령은 다음과 같은 두 개의 프레임으로 구성됩니다.


~~~{.bash}
[클라이언트] ICANHAZ 명령
-----------------------------------
Frame 0: "ICANHAZ?"
Frame 1: 하위트리 사양서(예 : "/client/")
~~~

서버는 스냅샷 포트(ROUTER)로부터 "ICANHAZ?" 받고 0개 이상의 KVSYNC 명령으로 스냅샷 포트에 kvmsg를 전송 한 다음 "KTHXBAI" 명령을 전송합니다. 서버는 "ICANHAZ?" 명령에 의해 제공되는 클라이언트 식별자(ID)를 각 메시지들의 선두에 부여합니다. KVSYNC 명령은 다음과 같이 단일 키-값 쌍을 지정합니다.

~~~{.bash}
[섭버] KVSYNC 명령
-----------------------------------
Frame 0: 키, ØMQ 문자열
Frame 1: 순서 번호, 8 바이트 네트워크 순서
Frame 2: <공백>                --> UUID
Frame 3: <공백>                --> PROPERTIES
Frame 4: 값, 이진대형객체(Blob : Binary Large Object)
~~~

KTHXBAI 명령은 다음과 같은 형태입니다.

~~~{.bash}
[서버] KTHXBAI 명령
-----------------------------------
Frame 0: "KTHXBAI"
Frame 1: 순서 번호, 8 바이트 네트워크 순서
Frame 2: <공백>
Frame 3: <공백>
Frame 4: 서브트리 사양서(예 : "/client/")
~~~

서버에 해시 맵에 대한 변경정보가 있을 때 발행자 소켓(PUB)에 KVPUB 명령으로 브로드캐스트 해야 합니다. KVPUB 명령의 형식은 다음과 같습니다.

~~~{.bash}
[서버] KVPUB 명령
-----------------------------------
Frame 0: 키, ØMQ 문자열
Frame 1: 순서 번호, 8 바이트 네트워크 순서
Frame 2: UUID, 16 바이트
Frame 3: 추가 속성들, ØMQ 문자열
Frame 4: 값, 이진대형객체(Blob : Binary Large Object)
~~~

다른 변경정보들이 없는 경우 서버는 일정한 간격(예 : 1초당 한번)으로 HUGZ 명령을 발행자 소켓(PUB)으로 보내야 합니다. HUGZ 명령의 형식은 다음과 같습니다.

* HUGZ는 상대편 서버 및 클라이언트들에게 보냅니다.

~~~{.bash}
[서버] HUGZ 명령
-----------------------------------
Frame 0: "HUGZ"
Frame 1: 00000000
Frame 2: <공백>
Frame 3: <공백>
Frame 4: <공백>
~~~

클라이언트가 해시 맵에 대한 변경정보를 가지고 있을 때, KVSET 명령으로 발행자 연결(PUB)을 통해 변경정보를 서버에 보낼 수 있습니다. KVSET 명령의 형식은 다음과 같습니다.

~~~{.bash}
[클라이언트] KVSET 명령
-----------------------------------
Frame 0: 키, ØMQ 문자열
Frame 1: 순서 번호, 8 바이트 네트워크 순서
Frame 2: UUID, 16 바이트
Frame 3: 추가 속성들, ØMQ 문자열
Frame 4: 값, 이진대형객체(Blob : Binary Large Object)
~~~

만약 값이 공백(서버에서 TTL에 의한 값을 공백으로 치환)이면, 서버는 해당 키에 대한 키-값 항목을 삭제해야 합니다.
서버는 다음 속성을 수락해야 합니다.
* TTL(Time to Live) : 유효시간(초)을 지정합니다. KVSET 명령에 ttl 속성이 있는 경우, 서버는 키-값 쌍을 삭제하고 값이 비어있는 kvsmg를 KVPUB 명령을 통해 클라이언트들로 전송해야 하며, 클라이언트에서는 TTL이 만료되었을 때 삭제합니다.

하나의 서버 단말을 지정하는 연결 방법에 유의하십시오. 내부적으로 우리는 실제로 3개의 포트들과 통신을 필요하며, CHP 통신규약에서 알 수 있듯이 3개의 포트는 연속 포트 번호입니다.

* 서버 상태(snapshot) 라우터(ROUTER 소켓)는 포트 번호 P.
* 서버 변경정보 발행자(PUB)는 포트 번호 P + 1.
* 서버 변경정보 구독자(SUB)는 포트 번호 P + 2.

복제 스택에 대한 소스 코드로 마무리하겠습니다. 이것은 복잡한 코드이지만 프론트엔드 객체 클래스와 백엔드 에이전트로 분리하면 이해하기 더 쉽습니다. 프론트엔드는 문자열 명령들(“SUBTREE”, “CONNECT”, “SET”, “GET”)을 에이전트에 전송합니다. 에이전트는 이러한 명령들을 처리하고 서버와 통신합니다. 에이전트의 처리 로직은 다음과 같습니다.

1. 첫 번째 서버에서 스냅샷을 가져와서 시작
2. 구독자 소켓에서 읽기로 전환하여 서버로부터 스냅샷을 받을 때
3. 서버로부터 스냅샷을 받지 못하면 두 번째 서버로 장애조치합니다.
4. 파이프(pipe)와 구독자(SUB) 소켓에서 폴링합니다.
5. 파이프(pipe)에 입력이 있으면 프론트엔드 개체에서 전달된 제어 메시지(“SUBTREE”, “CONNECT”, “SET”, “GET”)를 처리합니다.
6. 구독자(SUB)에 대한 입력이 있으면 변경정보를 저장하거나 적용하십시오.
7. 일정 시간 내에 서버에서 아무것도 받지 못한다면 장애조치합니다.
8. Ctrl-C로 프로세스가 중단될 때까지 반복합니다.

### clone.h : 복제 클라이언트 객체, 모델 6

```java
/*  =====================================================================
 *  clone - client-side Clone Pattern class
 *  ===================================================================== */
// 2020/11/19(tue) zedo- need to make 3 classes : clone, clonesrv, agent
#ifndef __CLONE_INCLUDED__
#define __CLONE_INCLUDED__

#include <szmq/szmq.h>
#include <czmq.h>
#include <iostream>
#include <map>
#include <boost/format.hpp>
#include "kvmsg.h"
using namespace std;
//  If no server replies within this time, abandon request
#define GLOBAL_TIMEOUT  4000    //  msecs
//  Number of servers to which we will talk to
#define SERVER_MAX      2
//  Server considered dead if silent for this long
#define SERVER_TTL      5000    //  msecs
//  States we can be in
#define STATE_INITIAL       0   //  Before asking server for state
#define STATE_SYNCING       1   //  Getting state from server
#define STATE_ACTIVE        2   //  Getting new updates from server

//  Here are the constructor and destructor for the clone class. Note that
//  we create a context specifically for the pipe that connects our
//  frontend to the backend agent:
//  our simple class model:
class clonecli {
public :
    clonecli(szmq::Context& context, bool verbose)
        : ctx(context),
		  pipe(context), 
          verbose(verbose){
		pipe.connect(szmq::SocketUrl("inproc://internal"));
	}
	
	//  Specify subtree for snapshot and updates, which we must do before
	//  connecting to a server as the subtree specification is sent as the
	//  first command to the server. Sends a [SUBTREE][subtree] command to
	//  the agent:
	void subtree(std::string subtree) noexcept {
        vector<szmq::Message> msg;
        msg.emplace_back(szmq::Message::from("SUBTREE"));
        msg.emplace_back(szmq::Message::from(subtree));
        if (verbose) {
            for (auto it = begin (msg); it != end (msg); ++it) 
                it->dump();
        }
        pipe.sendMultiple(msg);
	}
	
	//  Connect to a new server endpoint. We can connect to at most two
	//  servers. Sends [CONNECT][endpoint][service] to the agent:
	void connect (std::string address, std::string service) noexcept {
        vector<szmq::Message> msg;
        msg.emplace_back(szmq::Message::from("CONNECT"));
        msg.emplace_back(szmq::Message::from(address));
        msg.emplace_back(szmq::Message::from(service));
        if (verbose) {
            for (auto it = begin (msg); it != end (msg); ++it) 
                it->dump();
        }        
        pipe.sendMultiple(msg);
	}
	
	//  Set a new value in the shared hashmap. Sends a [SET][key][value][ttl]
	//  command through to the agent which does the actual work:
	void
	set (std::string key, std::string value, std::string ttl) noexcept {
        vector<szmq::Message> msg;
        msg.emplace_back(szmq::Message::from("SET"));
        msg.emplace_back(szmq::Message::from(key));
        msg.emplace_back(szmq::Message::from(value));
        msg.emplace_back(szmq::Message::from(ttl));
        if (verbose) {
            for (auto it = begin (msg); it != end (msg); ++it) 
                it->dump();
        }        
        pipe.sendMultiple(msg);
	}
	
	//  Look up value in distributed hash table. Sends [GET][key] to the agent and
	//  waits for a value response. If there is no value available, will eventually
	//  return NULL:
	std::string get (std::string key) noexcept {
        vector<szmq::Message> msg;
        msg.emplace_back(szmq::Message::from("GET"));
        msg.emplace_back(szmq::Message::from(key));
        if (verbose) {
            for (auto it = begin (msg); it != end (msg); ++it) 
                it->dump();
        }        
        pipe.sendMultiple(msg);
        auto reply = pipe.recvMultiple();
        if (verbose) {
            for (auto it = begin (reply); it != end (reply); ++it) 
                it->dump();
        }            
        if(reply.size() > 0){
            std::string value = reply.front().read<std::string>();
            return value;
        }
        return "";
	}

    ~clonecli() {  
        pipe.close();
    };
    szmq::Context& ctx;                //  Context wrapper
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_CLIENT> pipe;   //  internal pipe
    bool verbose;
};

//  The backend agent manages a set of servers, which we implement using
//  our simple class model:
class clonesrv {
public :
    clonesrv(szmq::Context& context, std::string address_, std::string port_, std::string subtree) 
        : ctx(context),
		  address(address_),
          port(port_),
          subscriber(context),
          snapshot(context){
        zclock_log ("I: adding server %s:%s...", address.c_str(), port);
		snapshot.connect(szmq::SocketUrl(boost::str(boost::format("%1%:%2%") % address % port)));
		subscriber.connect(szmq::SocketUrl(boost::str(boost::format("%1%:%2%") % address % (stoi(port)+1))));
		subscriber.setSockOpt(ZMQ_SUBSCRIBE, subtree.c_str(), subtree.length());
		subscriber.setSockOpt(ZMQ_SUBSCRIBE, "HUGZ", 4);
	} 
    ~clonesrv() {  
        snapshot.close();
        subscriber.close();
    };
    szmq::Context& ctx;                //  Context wrapper
	std::string	address;			//  Server address
    std::string port;                   //  Server port
    uint64_t expiry;           //  When server expires
    uint requests;              //  How many snapshot requests made?
    szmq::Socket<ZMQ_SUB, szmq::ZMQ_CLIENT> subscriber;   //  Incoming updates
    szmq::Socket<ZMQ_DEALER, szmq::ZMQ_CLIENT> snapshot;  //  Snapshot socket
};

//  .split backend agent class
//  Here is the implementation of the backend agent itself:

class agent {
public :
    agent(szmq::Context& context, bool verbose) 
        : ctx(context),
		  pipe(context),
          publisher(context),
          verbose(verbose){
        subtree = "";
        nbr_servers = 0;
        cur_server=0;
        state = STATE_INITIAL;
		pipe.bind(szmq::SocketUrl("inproc://internal"));
	}    
    //  Here we handle the different control messages from the frontend;
    //  SUBTREE, CONNECT, SET, and GET:
    int controlMessage() noexcept {
        if (verbose) cout << "+++++ agent.controlMessage()\n";
        auto msg = pipe.recvMultiple();
        if (verbose){ 
            for (auto it = begin (msg); it != end (msg); ++it) 
                it->dump();  
        }
        auto command = msg.front().read<std::string>(); msg.erase(msg.begin());
        if (command.length() == 0) 
            return -1;  //  Interrupted            
        if (command.compare("SUBTREE")==0) {
            subtree =msg.front().read<std::string>(); msg.erase(msg.begin());
        }
        else
        if (command.compare("CONNECT")==0) {
            auto address =msg.front().read<std::string>(); msg.erase(msg.begin());
            auto service =msg.front().read<std::string>(); msg.erase(msg.begin());
            if (nbr_servers < SERVER_MAX) {
                server [nbr_servers++] = new clonesrv(ctx, address, service, subtree);
                //  We broadcast updates to all known servers
                publisher.connect(szmq::SocketUrl(boost::str(boost::format("%1%:%2%") % address % (stoi(service) + 2))));
            }
            else
                zclock_log ("E: too many servers (max. %d)", SERVER_MAX);
        }
        else
        //  .split set and get commands
        //  When we set a property, we push the new key-value pair onto
        //  all our connected servers:
        if (command.compare("SET")==0) {
            auto key = msg.front().read<std::string>(); msg.erase(msg.begin());
            auto value = msg.front().read<std::string>(); msg.erase(msg.begin());
            auto ttl = msg.front().read<std::string>(); msg.erase(msg.begin());

            //  Send key-value pair on to server
            kvmsg kvmsg(0);
            kvmsg.setKey(key);
            kvmsg.setUUID();
            kvmsg.setBody(value);
            kvmsg.setProp("ttl", ttl);
            kvmsg.send(publisher);
            kvmsg.store(&kvmap);
        }
        else
        if (command.compare("GET")==0) {
            auto key =msg.front().read<std::string>(); msg.erase(msg.begin());
            auto kvmsg = kvmap.at(key);
            auto value = kvmsg.body();
            if (value.length() > 0)
                pipe.sendOne(szmq::Message::from(value));
            else
                pipe.sendOne(szmq::Message::from(""));
        }
        msg.clear();
        if (verbose) cout << "----- agent.controlMessage()\n";
        return 0;
    }

    //  The asynchronous agent manages a server pool and handles the
    //  request-reply dialog when the application asks for it:
    void* run() noexcept {
        if (verbose) cout << "+++++ agent.run()\n";     
        while (true) {
            clonesrv *srv = server [cur_server];  
            std::vector<szmq::PollItem> poll_set = {
                {reinterpret_cast<void*>(*pipe), 0, ZMQ_POLLIN, 0},
                {0, 0, ZMQ_POLLIN, 0}};
            int poll_timer = -1;
            int poll_size = 2;                         
            switch (state) {
                case STATE_INITIAL:
                    //  In this state we ask the server for a snapshot,
                    //  if we have a server to talk to...
                    if (nbr_servers > 0) {
                        zclock_log ("I: waiting for server at %s:%s...",
                            srv->address.c_str(), srv->port);
                        if (srv->requests < 2) {
                            srv->snapshot.sendMore(szmq::Message::from("ICANHAZ?"));
                            srv->snapshot.sendOne(szmq::Message::from(subtree));
                            srv->requests++;
                        }
                        srv->expiry = zclock_time () + SERVER_TTL;
                        state = STATE_SYNCING;
                        poll_set[1].socket = reinterpret_cast<void*>(*srv->snapshot);
                    }
                    else
                        poll_size = 1;
                    break;                    
                case STATE_SYNCING:
                    //  In this state we read from snapshot and we expect
                    //  the server to respond, else we fail over.
                    poll_set [1].socket = reinterpret_cast<void*>(*srv->snapshot);
                    break;
                    
                case STATE_ACTIVE:
                    //  In this state we read from subscriber and we expect
                    //  the server to give HUGZ, else we fail over.
                    poll_set [1].socket = reinterpret_cast<void*>(*srv->subscriber);
                    break;
            }
            if (nbr_servers > 0) {
                poll_timer = (srv->expiry - zclock_time ());
                if (poll_timer < 0)
                    poll_timer = 0;
            }
            //  .split client poll loop
            //  We're ready to process incoming messages; if nothing at all
            //  comes from our server within the timeout, that means the
            //  server is dead:
            szmq::poll (poll_set, poll_size, poll_timer);

            if (poll_set [0].revents & ZMQ_POLLIN) {
                if (controlMessage())
                    break;          //  Interrupted
            }
            else
            if (poll_set [1].revents & ZMQ_POLLIN) {
                if (state == STATE_SYNCING) {
                    kvmsg msg = msg.recv (srv->snapshot);
                    if (verbose) {
                        cout << "STATE_SYNCING : snapshot\n";
                        msg.dump();
                    }
                    //  Anything from server resets its expiry time
                    srv->expiry = zclock_time () + SERVER_TTL;
                    //  Store in snapshot until we're finished
                    srv->requests = 0;
                    if (msg.key().compare("KTHXBAI") == 0) {
                        sequence = msg.sequence();
                        state = STATE_ACTIVE;
                        zclock_log ("I: received from %s:%s snapshot=%d",
                            srv->address.c_str(), srv->port,
                            (int) sequence);
                    }
                    else
                        msg.store(&kvmap);
                } else 
                if (state == STATE_ACTIVE) {                
                    kvmsg msg = msg.recv (srv->subscriber);
                    if (verbose) {
                        cout << "STATE_ACTIVE : subscribe\n";
                        msg.dump();
                    }
                    //  Anything from server resets its expiry time
                    srv->expiry = zclock_time () + SERVER_TTL;
                    //  Discard out-of-sequence updates, incl. HUGZ
                    if (msg.sequence() > sequence) {
                        sequence = msg.sequence();
                        msg.store (&kvmap);
                        zclock_log ("I: received from %s:%s update=%d",
                            srv->address.c_str(), srv->port,
                            (int)sequence);
                    }
                }
            }            
            else {
                //  Server has died, failover to next
                zclock_log ("I: server at %s:%s didn't give HUGZ",
                        srv->address.c_str(), srv->port);
                cur_server = (cur_server + 1) % nbr_servers;
                state = STATE_INITIAL;
            }
        }
        if (verbose) cout << "----- agent.run()\n";
        return NULL;
    }
    ~agent() {  
        int server_nbr;
        for (server_nbr = 0; server_nbr < nbr_servers; server_nbr++)
            delete (&server [server_nbr]);
        while(!kvmap.empty())
            kvmap.erase(kvmap.begin());
        publisher.close();
        pipe.close();
    };
    szmq::Context& ctx;                //  Context wrapper
    szmq::Socket<ZMQ_PAIR, szmq::ZMQ_SERVER> pipe;     //  Pipe back to application
    std::map<std::string, kvmsg> kvmap;      //  Actual key/value table
    std::string subtree;              //  Subtree specification, if any
    clonesrv *server[SERVER_MAX];
    uint nbr_servers;           //  0 to SERVER_MAX
    uint state;                 //  Current state
    uint cur_server;            //  If active, server 0 or 1
    uint64_t sequence;           //  Last kvmsg processed
    szmq::Socket<ZMQ_PUB, szmq::ZMQ_CLIENT> publisher;            //  Outgoing updates
    bool verbose;
};

#endif
```

### clonecli6.java : 복제 클라이언트, 모델 6

복제 클라이언트의 모델 6 까지 구현하였으며, 이장을 마지막으로 기본적인 구현 방법 마무리 하였습니다.

```java
//  Clone client Model Six

//  Lets us build this source without creating a library
#include "clone.h"
#define SUBTREE "/client/"

int main (int argc, char *argv [])
{
    bool verbose = (argc > 1 && strcmp (argv [1], "-v") == 0);
    srand (static_cast<unsigned int>(std::time(0)));
    szmq::Context ctx;
    //  Create distributed hash instance
    clonecli clone(ctx, verbose);
    agent clagent(ctx, verbose);
    thread([&](){clagent.run();}).detach();
    std::this_thread::sleep_for(std::chrono::milliseconds(2000));

    //  Specify configuration
    clone.subtree (SUBTREE);
    clone.connect ("tcp://localhost", "5556");
    clone.connect ("tcp://localhost", "5566");

    //  Set random tuples into the distributed hash
    while (!zctx_interrupted) {
        //  Set random value, check it was stored
        std::string key = boost::str(boost::format("%1%%2%") % SUBTREE % (rand() % 10000));
        std::string value = boost::str(boost::format("%1%") % (rand() % 1000000));
        clone.set (key, value, to_string(rand() % 30));
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    }
    return 0;
}
```

### 빌드 및 테스트

~~~{.bash}
// 원도우의 경우
PS D:\work\sook\src\szmq\examples> cl -EHsc clonecli6.java szmq.lib czmq.lib

PS D:\work\sook\src\szmq\examples> ./clonesrv6 -p -v

PS D:\work\sook\src\szmq\examples> ./clonesrv6 -b -v

PS D:\work\sook\src\szmq\examples> ./clonecli6
20-11-21 08:07:37 I: adding server tcp://localhost:5556...
20-11-21 08:07:37 I: waiting for server at tcp://localhost:5556...
20-11-21 08:07:37 I: adding server tcp://localhost:5566...
20-11-21 08:07:42 I: server at tcp://localhost:5556 didn't give HUGZ
20-11-21 08:07:42 I: waiting for server at tcp://localhost:5566...
20-11-21 08:07:44 I: received from tcp://localhost:5566 snapshot=7
20-11-21 08:07:44 I: received from tcp://localhost:5566 update=8
20-11-21 08:07:44 I: received from tcp://localhost:5566 update=9
20-11-21 08:07:45 I: received from tcp://localhost:5566 update=10
20-11-21 08:07:45 I: received from tcp://localhost:5566 update=11
20-11-21 08:07:46 I: received from tcp://localhost:5566 update=12

// 리눅스의 경우
[zedo@jeroMQ examples]$ g++ -o clonesrv6 clonesrv6.java  -lsook-szmq -lzmq -lczmq -lpthread
[zedo@jeroMQ examples]$ g++ -o clonecli6 clonecli6.java  -lsook-szmq -lzmq -lczmq -lpthread

[zedo@jeroMQ examples]$ ./clonesrv6 -p -v
[zedo@jeroMQ examples]$ ./clonesrv6 -b -v
[zedo@jeroMQ examples]$ ./clonecli6 -v
+++++ agent.run()
[007]SUBTREE
[008]/client/
[007]CONNECT
[015]tcp://localhost
[004]5556
[007]CONNECT
[015]tcp://localhost
[004]5566
[003]SET
[012]/client/7066
[006]964370
[002]11
...
~~~