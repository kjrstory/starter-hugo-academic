---
title : "자동화 솔루션 n8n: 유튜브에서 슬랙으로 알람 "
date : "2022-02-21T14:55:49+09:00"
draft: false
featured: false
authors:
  - admin
tags:
  - n8n
  - youtube
  - slack
---


[n8n](https://n8n.io)은 IFTTT, Zapier와 유사한 자동화 솔루션이다.
가장 큰 차이점은 n8n은 오픈소스여서 사용자가 직접 서버를 올릴 수 있다. 
IFTTT, Zapier도 좋은 서비스이지만 유료화를 하면서 제대로 사용을 하려면 돈을 주고 사용해야 한다.
n8n은 별도의 서버가 필요하고 사용법이 다소 까다롭지만 직접 나만의 자동화 솔루션을 사용하고 싶은 사용자에게 좋은 대안 SW이다.
지금은 n8n 데스크탑을 설치하여 데스크탑을 서버로 사용하고 있다. 

이 포스트에서는 youtube의 원하는 채널에서 업데이트 되면 slack으로 알람을 주는 자동화 workflow를 소개한다. 
youtube뿐만 아니라 다양한 소스에서 오는 정보들을 slack으로 합치려고 한다.
slack도 나중에는 온프레미스에 설치할 수 있는 mattermost로 바꿀려고 한다.

![Fig0](/img/YoutubeToSlack_Fig0.png)

전체 Flow는 다섯단계로 나눈다. 각각의 단계를 n8n에서는 노드라고 부른다. 위의 workflow는 5개의 노드로 이루어져있다.
각 노드를 더블클릭하면 상세 설정 화면으로 이동한다. 

### 노드 1
![Fig1](/img/YoutubeToSlack_Fig1.png)

리눅스에서 주로 쓰이는 cron 작업으로 일정 주기로 반복하는 설정을 한다.
여기서는 30분마다 명령을 주게 설정하였다.

### 노드 2

![Fig2_1](/img/YoutubeToSlack_Fig2_1.png)
youtube의 채널 설정을 한다.
그전에 youtubue 인증을 해야되는데 이 부분이 다소 까다롭다. 이것은 별도의 포스트에 올려보려고 한다.
Limit은 5개의 소스만 받는다는것이다. 
Channel ID는 내가 원하는 채널의 채널명을 입력하면 된다.
여기서는 [슈카월드](https://www.youtube.com/channel/UCsJ6RuBiTVWRX156FVbeaGg)의 채널을 예시로 들었다. 


GetResults를 누르면 임시 결과를 아래와 같이 볼 수 있다.
![Fig2_3](/img/YoutubeToSlack_Fig2_3.png)


### 노드 3 

![Fig3](/img/YoutubeToSlack_Fig3.png)
2 단계에서 보여지는 데이터를 단순화하기 위해 Json파일을 Flat하게 바꾸는 역할을 한다.
여러 변수 중 id 배열만 가져오기로 한다.

### 노드 4

4단계에서는 3단계에서 받은 item중 새로운 아이템만 필터링한다. 
여기서 쓰이는 자바크립트 코드는 아래와 같다.
``` javascript
const staticData = getWorkflowStaticData('global');
const newIds = items.map(item => item.json["videoId"]);
const oldIds = staticData.oldIds; 

if (!oldIds) {
  staticData.oldIds = newIds;
  return items;
}


const actualNewIds = newIds.filter((id) => !oldIds.includes(id));
const actualNew = items.filter((data) => actualNewIds.includes(data.json["videoId"]));
staticData.oldIds = [...actualNewIds, ...oldIds];

return actualNew;
```

### 노드 5

![Fig5_1](/img/YoutubeToSlack_Fig5_1.png)
5단계에서는 Slack에 대해 설정하는 부분이다. youtube인증과 마찬가지로 slack의 인증을 거쳐야하는데 역시 이부분도 다른 포스트에 작성하기로 한다. 
slack에 알람을 줄 채널 이름을 n8n으로 정하고 Text부분은 아래와 같이 만들었다.

![Fig5_2](/img/YoutubeToSlack_Fig5_2.png)

n8n데스크탑을 켜놓은 상태에서 등록한 유튜브 채널에 업데이트가 되면 30분에 한번씩 모니터링을 해서 업데이트를 하게 된다.

![YoutubeToSlackResult](/img/YoutubeToSlack_Result.png)



또한 n8n에서는 이 workflow를 json파일 형식으로 Import/Export 가 가능하다.

```
{
  "name": "Youtube2Slack",
  "nodes": [
    {
      "parameters": {
        "resource": "video",
        "limit": 5,
        "filters": {
          "channelId": "UCsJ6RuBiTVWRX156FVbeaGg"
        },
        "options": {
          "order": "date"
        }
      },
      "name": "YouTube",
      "type": "n8n-nodes-base.youTube",
      "position": [
        460,
        -120
      ],
      "typeVersion": 1,
      "credentials": {
        "youTubeOAuth2Api": {
          "id": "4",
          "name": "YouTube account"
        }
      }
    },
    {
      "parameters": {
        "functionCode": "const staticData = getWorkflowStaticData('global');\nconst newIds = items.map(item => item.json[\"videoId\"]);\nconst oldIds = staticData.oldIds; \n\nif (!oldIds) {\n  staticData.oldIds = newIds;\n  return items;\n}\n\n\nconst actualNewIds = newIds.filter((id) => !oldIds.includes(id));\nconst actualNew = items.filter((data) => actualNewIds.includes(data.json[\"videoId\"]));\nstaticData.oldIds = [...actualNewIds, ...oldIds];\n\nreturn actualNew;\n"
      },
      "name": "Filter new items",
      "type": "n8n-nodes-base.function",
      "position": [
        780,
        -120
      ],
      "typeVersion": 1
    },
    {
      "parameters": {
        "functionCode": "item = item[\"id\"]\n\nreturn item;"
      },
      "name": "Flatten JSON",
      "type": "n8n-nodes-base.functionItem",
      "position": [
        620,
        -120
      ],
      "typeVersion": 1
    },
    {
      "parameters": {
        "triggerTimes": {
          "item": [
            {
              "mode": "everyX",
              "value": 30,
              "unit": "minutes"
            }
          ]
        }
      },
      "name": "Every 30 mins",
      "type": "n8n-nodes-base.cron",
      "position": [
        300,
        -120
      ],
      "typeVersion": 1
    },
    {
      "parameters": {
        "authentication": "oAuth2",
        "channel": "n8n",
        "text": "=새로운 동영상이 도착했습니다.\nhttps://www.youtube.com/watch?v={{$node[\"Filter new items\"].json[\"videoId\"]}}",
        "attachments": [],
        "otherOptions": {}
      },
      "name": "Slack",
      "type": "n8n-nodes-base.slack",
      "typeVersion": 1,
      "position": [
        940,
        -120
      ],
      "credentials": {
        "slackOAuth2Api": {
          "id": "1",
          "name": "Slack account"
        }
      }
    }
  ],
  "connections": {
    "YouTube": {
      "main": [
        [
          {
            "node": "Flatten JSON",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Flatten JSON": {
      "main": [
        [
          {
            "node": "Filter new items",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Every 30 mins": {
      "main": [
        [
          {
            "node": "YouTube",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Filter new items": {
      "main": [
        [
          {
            "node": "Slack",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": true,
  "settings": {},
  "id": 1
}
```
