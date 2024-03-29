---
title: 서버를 한 대씩 작업할 때, L4 로드발란서의 세션 테이블은 어떻게 해야 할까?
date: 2022-12-29
tags: [l4, slb, failover, session table]
---
서버를 한 대씩 작업을 하기 위해 로드발란서에서 작업하는 서버로 세션을 만들어주지 않기로 했다. 이때 기존에 있던 세션은 어떻게 처리해야 할까?

![그림 1.0](/networkengineering/docs/assets/images/l4_sessions-1.png)

그림 1.0처럼 L4가 기존에 각 서버에 세션을 분배해주고 있다고 하자. 서버를 하나씩 작업하기 전에 L4에서 작업하는 real 서버를 내려주었다. 이때 L4의 세션 테이블에서 작업하는 real 서버에 물린 세션 항목을 삭제해버리면, 기존에 있던 세션인 2.2.2.2:25002에서 서버#1로 가는 세션은 사라진다. 이때 삭제하는 항목에 대해 L4가 reset 패킷을 날려주면 좋긴 하지만, L4가 세션을 삭제하면서 reset 패킷을 날려주진 않는다.

![그림 1.1](/networkengineering/docs/assets/images/l4_sessions-2.png)

그러면 client 입장에서는 세션이 살아있기 때문에, 계속 통신을 하려는 시도를 할 것이고, 이러한 패킷은 그림 1.1처럼 L4에 새로운 세션으로 생성되어 작업을 하지 않고 있는 서버#2로 보내진다. 이 패킷을 받은 서버#2는 "어? 나한텐 이 패킷의 세션이 없는데, 이건 syn 패킷도 아니네? 버려!"하고 패킷 드랍을 하게 될 것이고, client는 계속 응답을 기다리는 상황이 발생하게 된다.

그럼 그동안 서버#1은 뭘하고 있느냐? 서버#1도 client 2.2.2.2:25002에 패킷을 보내고 있었겠지만, L4는 이미 세션테이블을 삭제해버려서 이 패킷의 Source IP를 변경하지 않고 그대로 client에게 보냈을 것이다. client는 VIP로 패킷을 보냈었지, real 서버 IP인 10.1.1.1에게 패킷을 보낸적이 없으니 역시 모르는 세션으로 간주하고 패킷 drop을 했을 것이다.[^1]

이러한 상황에 이르지 않으려면, L4가 세션테이블의 항목을 삭제하기 전에, 서버#1이 fin/reset 패킷을 보내주는 것이 필요하다. 만약 서버를 바로 다운시킨다면, L4에서 real을 disable시킬 때 세션 테이블을 강제로 삭제시키지 말도록 하고, 서버를 다운시키면 된다. 만약 작업 내용 중 서버를 다운시키지 않는다면, application이라도 종료를 시켜주어야 한다. 그렇게 해야 L4에 세션 테이블이 살아 있을 때, 서버에서 fin 혹은 reset 패킷을 보내줄테니까 말이다.

[^1]: 물론 패킷이 client까지 갔다면 말이다. IP가 다르므로 방화벽도 통과하지 못했을 확률이 더 높다.