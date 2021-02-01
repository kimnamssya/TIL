## ROS 명령어와 도구

##### ROS 명령어 (중요 명령어)

|      명령어       |                          설명                          |
| :---------------: | :----------------------------------------------------: |
|       roscd       |           지정한 ros 패키지 디렉토리로 이동            |
|      roscore      |                      master 실행                       |
|      rosrun       |                       node 실행                        |
|     roslaunch     |                     node 다중 실행                     |
|     rostopic      |                  ros topic 정보 확인                   |
|    rosservice     |                 ros service 정보 확인                  |
|      rosnode      |                   ros node 정보 확인                   |
|     rosparam      |                ros parameter 정보 확인                 |
|      rosbag       |            통신 중인 메시지 기록 또는 재생             |
| catkin_create_pkg | 패키지 생성(catkin은 ros의 컴파일, 빌드를 위한 시스템) |
|    catkin_make    |                      패키지 빌드                       |
|      rospack      |                  ros 패키지 정보 확인                  |



##### ROS 개발 도구

##### 1. RViz

* ROS의 3D 시각화 툴(센서 데이터, 영상 데이터, 관성 데이터 등)

<img src="https://user-images.githubusercontent.com/48755185/106488552-2c6d6a80-64f7-11eb-9dab-b381f1b0e007.png" alt="스크린샷, 2021-02-02 01-36-35" style="zoom:80%;" />



##### 2. RQT

* 플러그인 방식의 종합 GUI 툴

<img src="https://user-images.githubusercontent.com/48755185/106489179-d51bca00-64f7-11eb-90e8-2abd9e326c45.png" alt="스크린샷, 2021-02-02 01-41-41" style="zoom:70%;" />

uvc_camera_node를 통해 웹캠 raw 데이터 전송 후 rqt_image_view 플러그인을 통해 시각화

> uvc_camera_node가 rqt_image_view node에게 raw 데이터 메시지를 계속해서 보내고 rqt는 raw 데이터 정보를 시각화 해서 보여준다(즉, 두 노드가 통신 중).

---

<img src="https://user-images.githubusercontent.com/48755185/106489825-86bafb00-64f8-11eb-9178-ec4d519822a8.png" alt="스크린샷, 2021-02-02 00-21-07" style="zoom:80%;" />

rostopic echo /image_raw 를 통해 전송 중인 raw 데이터를 확인해본 결과

> rostopic echo는 현재 전송 중인 메시지를 확인하는 명령어

----

<img src="https://user-images.githubusercontent.com/48755185/106490021-bc5fe400-64f8-11eb-91bc-473cda65d679.png" alt="스크린샷, 2021-02-02 00-21-39" style="zoom:80%;" />

rostopic info /image_raw 를 통해 /image_raw topic의 정보 확인

> uvc_camera가 계속해서 raw 데이터를 보내고 있으므로 publisher이고, 현재 rqt_image_view를 꺼놓은 상태이므로 subscriber는 none으로 표시 됨.

---

<img src="https://user-images.githubusercontent.com/48755185/106490296-05179d00-64f9-11eb-8c98-0925ddaaaebb.png" alt="스크린샷, 2021-02-02 00-26-51" style="zoom:80%;" />

rqt_graph 플러그인을 통해 현재 실행되고 있는 노드들의 통신을 시각화해서 볼 수 있다.

> teleop_turtle 노드가 turtlesim 노드에게 /turtle1/cmd_vel 라는 메시지(topic)를 보내는 중이고, 
>
> uvc_camera가 image_view에게 image_raw 라는 메시지(topic)를 보내는 중임.

---

<img src="https://user-images.githubusercontent.com/48755185/106490626-5aec4500-64f9-11eb-9dfa-aee87dad7f1a.png" alt="스크린샷, 2021-02-02 00-34-25" style="zoom:80%;" />

rqt_plot 플러그인을 통해 turtle1의 pose를 그래프로 확인할 수 있다(linear velocity, angular velocity 등)

---

<img src="https://user-images.githubusercontent.com/48755185/106490891-a272d100-64f9-11eb-9214-1d7d85e68c1b.png" alt="스크린샷, 2021-02-02 01-54-39" style="zoom:80%;" />

rqt_bag 플러그인을 통해 현재 전송중인 메시지의 기록, 재생이 가능하다.



##### 3. Gazebo

* 물리 엔진을 탑재한 3차원 시뮬레이터(엔진마다 정역학, 동역학 등 강점이 다르고 사용할 엔진을 선택 가능)
* ROS와의 호환성이 좋음

<img src="https://user-images.githubusercontent.com/48755185/106491488-42305f00-64fa-11eb-8df8-c3494c32a50d.png" alt="스크린샷, 2021-02-02 01-59-08" style="zoom:80%;" />

gazebo 실행 화면