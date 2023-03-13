# CWAgent Windows Hostname

- 페이지 생성일: 2023.03.13

<br>

## CWAgent 를 활용하여 hostname 전달 (테스트)

- 사전에 CWAgent 설치 및 기본 설정 파일(config.json)을 통해, CWAgent는 **Running** 상태임을 가정
- Windows **hostname** 도 사전에 원하는 값으로 변경했음을 가정


### Test code

<br>

```json
{
    "metrics": {
        "append_dimensions": {
            "InstanceId": "${aws:InstanceId}"
        },
        "metrics_collected": {
            "Windows": {
                "metric_sets": [
                    {
                        "name": "Hostname",
                        "dimensions": [
                            {
                                "Name": "ServerName",
                                "Value": "${env:COMPUTERNAME}"
                            }
                        ],
                        "metrics": [
                            {
                                "name": "HostName",
                                "rename": "MyHostName",
                                "unit": "None",
                                "expression": "HOSTNAME",
                                "namespace": "CustomMetric/Windows"
                            }
                        ]
                    }
                ],
                "metrics_collection_interval": 300
            }
        }
    }
}
```

<br>

#### 가이드 상(ChatGPT) 가능하다고 하는 것

- AWS 인스턴스 ID를 차원으로 추가합니다.
- Windows에서 호스트 이름을 수집합니다.
- 수집 된 호스트 이름을 "MyHostName"이라는 이름으로 CloudWatch Metrics에 게시합니다.



<br>
