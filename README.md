# elastic-edu



### < 환경별 Elastic Stack 설치하기 >

1) [GCP에서 Single Node로 Elastic 설치하기](./Install-guide/01.GCP-SingleNode-Elastic-Install.md)

2) [WSL2에서 Elastic 설치하기](./Install-guide/02.WSL2-Elastic-Install.md)

## 에러 메시지

{"log.level":"warn","@timestamp":"2023-10-24T09:11:19.180Z","log.origin":{"file.name":"fleet/fleet_gateway.go","file.line":194},"message":"Possible transient error during checkin with fleet-server, retrying","log":{"source":"elastic-agent"},"error":{"message":"fail to checkin to fleet-server: all hosts failed: 1 error occurred:\n\t* requester 0/1 to host https://10.109.210.170:8220/ errored: Post \"https://10.109.210.170:8220/api/fleet/agents/ea9abeb2-e162-4839-8d25-fee4456ce2b1/checkin?\": dial tcp 10.109.210.170:8220: connect: connection refused\n\n"},"request_duration_ns":75002574534,"failed_checkins":1,"retry_after_ns":77367008597,"ecs.version":"1.6.0"}
