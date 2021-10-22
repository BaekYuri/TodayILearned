#### ROS

- 메타 운영체제
- ROS1에서 ROS2로 바뀌면서, 
  - TCPROS/UDPROS -> DDS로 바뀜
    - 다수의 로봇을 다루기 쉬움
  - Master -> Node로 바뀜
    - 마스터가 죽어서 본 시스템이 죽을 걱정을 안해도 됨



- 용어

  - 노드
    - ROS에서 최소 단위의 실행 프로세스를 가리키는 용어. 하나의 파이썬 스크립트 = 하나의 노드
  - 메시지
    - 노드에서 다른 노드로 정보를 전달하는 단방향,  비동기식, 연속성 통신
    - 마스터를 통해서 각 노드의 Publisher와 Subscriber의 정보를 공유해서 메시지를 주고받음
    - 메시지를 주고받으려면 토픽(메시지의 이름), 메시지 타입을 맞춰주어야 함
  - 패키지
    - ROS 소프트웨어의 기본 단위
  - rviz2
    - 라이다, tf, image 등의 메시지를 3D로 시각화 해줌
  - rqt
    - 토픽으로 나오는 메시지를 모니터 할 수 있고, 노드와 메시지의 관계를 보기 쉽게 그래프로 그려줌

