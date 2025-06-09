# 바글바글 실시간 알람 서버
바글바글 서비스의 실시간 알람을 전달하는 서버입니다.
[(바글바글 메인 서버 레포지토리 링크)](https://github.com/BaGulBaGul/BaGulBaGul_Backend_Main)
<br>
작성한 게시물에 댓글이 달리거나, 댓글에 답글이 달리거나, 좋아요 표시를 받는 등의 다양한 상황에서 메인서버에서 알람을 발행하고 이것을 실시간 알람 서버에서 받아 즉시 사용자에게 전달합니다.
<br>

# 동작 흐름
![bagulbagul_alarm drawio](https://github.com/user-attachments/assets/9153e704-b7bf-4048-a863-0f80b2e0cc46)
사용자는 실시간 알람 서버에 구독을 요청하고 jwt토큰을 이용해 인증하며 SSE 연결을 맺습니다.<br>
실시간 알람 서버는 redis pub/sub에 userId를 토픽으로 구독합니다.<br>
메인 서버가 redis pub/sub에 알람 정보를 전달하면 실시간 알람 서버는 SSE로 사용자에게 알람을 전송합니다.

# 사용 기술
- spring webflux
- redis pubsub
- jwt

# 특징
![alarm_inner drawio](https://github.com/user-attachments/assets/7a674fc2-a293-46bd-a10e-33f88894840b)
- spring webflux를 사용해 비동기 논블록 구조로 동작합니다. 알람이 발행되는 빈도가 낮다는 점을 고려할 때 각 sse연결마다 하나의 스레드를 점유하는 전통적인 spring mvc방식에 비해 높은 효율로 sse연결 관리가 가능합니다.
- redis pub/sub에 구독을 하는 등의 블록 가능성이 있는 연산은 작업 스레드에서 처리함으로써 이벤트 루프가 블록되지 않도록 했습니다.
- 같은 사용자의 모든 sse 연결이 하나의 RedisMessageListener와 Sink를 공유해서 더 효율적인 메세지 수신과 송신이 가능하도록 했습니다.
  - ConcurrentMap의 compute 등을 이용해 RedisUserAlarmSubscribeInfo에서 자체적인 sse 연결 구독자 정보를 관리하고 동시성 문제를 해결했습니다.
- Redis Pub/Sub에서 메세지는 즉시 사용되고 구독자가 없으면 버려집니다. 이는 실시간 알람에 잘 맞는 효율적인 구조입니다.

