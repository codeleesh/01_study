# 개요

- 매일 고민되는 점심시간 메뉴를 고민하지 않고 선택해보려고 만들게 되었다.
- 카카오 알림, 텔레그램 Chatbot 등등이 있으나 Slack을 자주 써보고 싶었다.
- 정기적으로(오전 11시) 알람을 주기 위해서 Spirng-boot2 Batch를 공부하면서  진행할 예정이다.
- 여기서는 SlackChatBot 만드는 방법을 소개한다.



## 진행 순서

- Chatbot 사전 준비
- Chatbot 생성
- Slack 채널 생성 및 채널정보 얻기
- Chat API 호출 및 소스코드 구현



### Chatbot 사전 준비

- Slack 계정 생성

  - https://slack.com/intl/ko-kr/

- Slack App 생성

  - https://api.slack.com/apps

- Creating, managing, and building apps

  https://api.slack.com/start/overview#creating

  Bot User OAuth Token
  xoxb-1651296225187-2266599551777-iuzgIzhhNxSV9aH6HKqPHRHo

  

### Chatbot 생성





conversations.list

https://api.slack.com/methods/conversations.list/test



Scheduling messages

https://api.slack.com/messaging/scheduling



## API response

URL

`https://slack.com/api/conversations.list?limit=3&pretty=1`

`Authorization: Bearer xoxb-1651296225187-2266599551777-iuzgIzhhNxSV9aH6HKqPHRHo`

```
{
    "ok": true,
    "channels": [
        {
            "id": "C027FP81Z0A",
            "name": "점심해결하자",
            "is_channel": true,
            "is_group": false,
            "is_im": false,
            "created": 1625749394,
            "is_archived": false,
            "is_general": false,
            "unlinked": 0,
            "name_normalized": "점심해결하자",
            "is_shared": false,
            "parent_conversation": null,
            "creator": "U01JQ9B0RF1",
            "is_ext_shared": false,
            "is_org_shared": false,
            "shared_team_ids": [
                "T01K58Q6M5H"
            ],
            "pending_shared": [],
            "pending_connected_team_ids": [],
            "is_pending_ext_shared": false,
            "is_member": true,
            "is_private": false,
            "is_mpim": false,
            "topic": {
                "value": "",
                "creator": "",
                "last_set": 0
            },
            "purpose": {
                "value": "점심식사",
                "creator": "U01JQ9B0RF1",
                "last_set": 1625749395
            },
            "previous_names": [],
            "num_members": 3
        }
    ],
    "response_metadata": {
        "next_cursor": ""
    }
}
```





