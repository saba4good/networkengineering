---
title: 라우팅 경로의 이원화로 인한 문제
date: 2022-11-26
tags: [asymmetric routing, unknown unicast flooding]
---
알 수 없는 유니캐스트로 트래픽이 잠깐씩 올라가는 것을 볼 때가 있다. 이 현상의 원인 중 하나는 트래픽 양방향의 경로가 다를 때, mac address table의 특성 때문에 생기게 된다.[^1]

MAC address table은 장비 내로 들어오는 frame의 source address를 보고 업데이트를 한다. MAC 테이블을 사용하는 것은 frame을 내보낼 때 사용하지만, MAC 테이블에 주소를 등록하는 건 frame이 들어올 때 한다는 뜻이다. 즉, 2개의 호스트가 서로 통신을 할 때, 중간의 장비가 양쪽이 하는 말을 다 들을 수 있다는 가정하에 만들어진 디자인이다.

그렇기 때문에 경로가 이원화되면 이 디자인은 활용될 수 없어지고, 백업 플랜을 사용해야 하는데, 백업 플랜은 들어온 포트를 제외한 모든 포트로 frame을 내보내는 것이다!(굉장하지!@.@)

만약 경로가 이원화된 통신이 자주 일어나고, 경로에 있는 L2 장비에 동일 vlan이 많아지면, 부하도 커질 수 있겠다.

그런데, 이 L2 장비가 알 수 없는 유니캐스트를 모든 포트로 내보내는 기능을 못쓰게 해뒀다면, 혹은 그게 default 값이라면 어떻게 될까?

ARP 테이블에는 MAC 주소가 있고, MAC 테이블에 주소가 없는 동안은 frame이 버려진다.(통신 불가 상태) 그러다가 타이머가 울리고 ARP 테이블에서 MAC 주소가 사라지면, ARP 통신으로 인해 ARP 테이블에도 MAC 주소가 올라가고, MAC 테이블에도 주소가 생기게 된다.(ARP response 패킷이나 Gratuitous ARP 패킷으로 인해 말을 안하던 호스트의 말을 들을 수 있는 기회가 생긴다!) 보통 이럴 때 Gratuitous ARP 타이머(60, 90, 180 seconds)동안 통신 두절 상태가 이어지는 것을 볼 수 있다.

경로가 이원화되는 이유는 여러가지가 있는데, 서버의 이중화 포트가 서로 다른 네트웍 스위치에 Active-Active로 연결되어 있는 경우, 혹은 서로 다른 기관끼리 통신할 때, 커뮤니케이션 문제로 한 쪽은 인터넷으로, 다른 한 쪽은 전용회선으로 통신하게 되거나, 혹은 장비를 재기동했는데 기존 설정이 저장되지 않아서 세팅이 변경된 경우 등 다양하게 있을 수 있다.

경로가 이원화되면 방향에 따라 hops 수가 달라지게 되어 TTL에서 차이가 나게 될 수 있다.

[^1]: [Unicast Flooding in Switched Campus Networks](https://www.cisco.com/c/en/us/support/docs/switches/catalyst-6000-series-switches/23563-143.html){:target="_blank"}{:rel="noopener noreferrer"}