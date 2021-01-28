## 1. ROS 용어 정리

#### 1. 기본 용어

* **Node** - 최소 단위의 실행 가능한 프로세서(프로그램). ROS에서는 프로그램을 노드 단위로 나누어 작업하고(별개의 프로그램이 됨), 노드와 노드는 **메시지 통신**으로 데이터를 주고 받는다. 노드가 open source로 존재하면 필요한 노드만 가져와서 사용이 가능하다. 

* **Package** - 하나 이상의 노드와 노드 실행을 위한 정보를 묶어 놓은 것.

* **Message** - 노드 간의 데이터를 주고 받는 방법. 



#### 2. Message 통신 방법

* **Topic** - 단방향 연속성 메시지 통신 방법. 메시지를 받는 노드를 Subscriber, 메시지를 주는 노드를 Publisher라고 한다. 연속성이 있으므로 통신이 끊어질 때까지 데이터를 계속 단방향(Publisher -> Subscriber)으로 전달한다. 
* **Service** - 양방향 일회성 메시지 통신 방법. Service client가 특정 서비스 요청을 하고 Service server가 요청에 대한 응답을 한다. 일회성이므로 단 한 번 요청, 응답 후 통신이 끊어진다.
* **Action** - 양방향 메시지 통신 방법. Service처럼 Action client가 특정 액션 목표를 요청하고 Action server가 액션 결과를 전달한다는 점은 동일하지만, Action의 경우 server가 중간중간 피드백을 전달한다는 차이점이 있다. 

> **대부분의 통신은 Topic으로 이루어지고 나머지는 Service가 차지한다(Action은 잘 안쓰임).**



#### 3. Message 통신 과정

![roscore](https://user-images.githubusercontent.com/48755185/106152038-573a8480-61c0-11eb-84db-1ef864be4c92.JPG)

* roscore로 마스터를 구동 

  > 데이터에 직접적인 관여는 하지 않고, 노드 간의 통신을 연결시켜주는 매개체로 작동한다.

* subscriber 노드 구동(turtlesim_node) 

  > 노드 이름, 토픽 이름, 받고자 하는 메시지 타입, IP와 Port 번호 정보를 마스터에게 전달한다.

* publisher 노드 구동(turtle_teleop_key)

  > 마찬가지로 노드에 대한 정보를 마스터에게 전달한다.

* 마스터가 subscriber 노드에게 publisher 노드의 정보를 전달

  > 받기를 원하는 정보를 publisher가 가지고 있기 때문에 마스터가 서로를 연결 시킨다. 

* 노드 정보가 전달되면 subscriber가 publisher에게 TCPROS 통신으로 연결하기를 요청하고, publisher가 통신 연결에 응답하면 두 노드는 TCPROS 통신으로 연결되어 메시지 통신이 가능한 상태가 됨

* 전달 받기를 원했던 /turtle1/cmd_vel 메시지를 TCPROS 통신으로 전달 받는다. 