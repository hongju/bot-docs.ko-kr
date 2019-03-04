일부 채널은 메시지 텍스트 및 첨부 파일만으로는 구현할 수 없는 기능을 제공합니다. 채널 관련 기능을 구현하려면 활동 개체의 _채널 데이터_ 속성에서 채널에 네이티브 메타데이터를 전달할 수 있습니다. 예를 들어, 봇은 채널 데이터 속성을 사용하여 Telegram에 스티커를 전송하도록 지시하거나 Office365에 이메일을 전송하도록 지시할 수 있습니다.

이 문서에서는 메시지 작업의 채널 데이터 속성을 사용하여 다음과 같은 채널 관련 기능을 구현하는 방법을 설명합니다.

| 채널 | 기능 |
|----|----|
| Email | 본문, 제목 및 중요 메타데이터를 포함하는 이메일 보내기 및 받기 |
| Slack | 완전히 신뢰할 수 있는 Slack 메시지 보내기 |
| Facebook | 고유하게 Facebook 알림 보내기 |
| Telegram | 음성 메모 또는 스티커 공유와 같은 Telegram 관련 작업 수행 |
| Kik | 네이티브 Kik 메시지 보내기 및 받기 |

> [!NOTE]
> 활동 개체의 채널 데이터 속성 값은 JSON 개체입니다.
> 따라서 이 문서의 예제에서는 다양한 시나리오에서 예상되는 `channelData` JSON 속성의 형식을 보여줍니다.
> .NET을 사용하여 JSON 개체를 만들려면 `JObject`(.NET) 클래스를 사용합니다.

## <a name="create-a-custom-email-message"></a>사용자 지정 이메일 메시지 만들기

이메일 메시지를 만들려면 활동 개체의 채널 데이터 속성을 이러한 속성을 포함하는 JSON 개체로 설정합니다.

| 자산 | 설명 |
|----|----|
| bccRecipients | 메시지의 Bcc(숨은 참조) 필드에 추가할 세미콜론(;)으로 구분된 이메일 주소 문자열입니다. |
| ccRecipients | 메시지의 Cc(참조) 필드에 추가할 세미콜론(;)으로 구분된 이메일 주소 문자열입니다. |
| htmlBody | 이메일 메시지의 본문을 지정하는 HTML 문서입니다. 지원되는 HTML 요소 및 특성에 대한 정보는 채널의 설명서를 참조하세요. |
| 중요도 | 이메일의 중요도 수준입니다. 유효한 값은 **높음**, **기본** 및 **낮음**입니다. 기본값은 **기본**입니다. |
| 제목 | 이메일의 제목입니다. 필드 요구 사항에 대한 정보는 채널의 설명서를 참조하세요. |
| toRecipients | 메시지의 받는 사람 필드에 추가할 세미콜론(;)으로 구분된 이메일 주소 문자열입니다. |

> [!NOTE]
> 봇이 이메일 채널을 통해 사용자로부터 받는 메시지는 위에서 설명한 것과 같은 JSON 개체로 채워지는 채널 데이터 속성을 포함할 수 있습니다.

이 코드 조각은 사용자 지정 이메일 메시지에 대한 `channelData` 속성의 예제를 보여 줍니다.

```json
"channelData": {
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

## <a name="create-a-full-fidelity-slack-message"></a>완전히 신뢰할 수 있는 Slack 메시지 만들기

완전히 신뢰할 수 있는 Slack 메시지를 만들려면 활동 개체의 채널 데이터 속성을 <a href="https://api.slack.com/docs/messages" target="_blank">Slack 메시지</a>, <a href="https://api.slack.com/docs/message-attachments" target="_blank">Slack 첨부 파일</a> 및/또는 <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack 단추</a>를 지정하는 JSON 개체로 설정합니다.

> [!NOTE]
> 설정 단추에서 Slack 메시지를 지원 해야 합니다 **대화형 메시지** 때 있습니다 [봇을 연결](../bot-service-manage-channels.md) Slack 채널에 있습니다.

이 코드 조각은 사용자 지정 Slack 메시지에 대한 `channelData` 속성의 예제를 보여 줍니다.

```json
"channelData": {
   "text": "Now back in stock! :tada:",
   "attachments": [
        {
            "title": "The Further Adventures of Slackbot",
            "author_name": "Stanford S. Strickland",
            "author_icon": "https://api.slack.com/img/api/homepage_custom_integrations-2x.png",
            "image_url": "http://i.imgur.com/OJkaVOI.jpg?1"
        },
        {
            "fields": [
                {
                    "title": "Volume",
                    "value": "1",
                    "short": true
                },
                {
                    "title": "Issue",
                    "value": "3",
                    "short": true
                }
            ]
        },
        {
            "title": "Synopsis",
            "text": "After @episod pushed exciting changes to a devious new branch back in Issue 1, Slackbot notifies @don about an unexpected deploy..."
        },
        {
            "fallback": "Would you recommend it to customers?",
            "title": "Would you recommend it to customers?",
            "callback_id": "comic_1234_xyz",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "recommend",
                    "text": "Recommend",
                    "type": "button",
                    "value": "recommend"
                },
                {
                    "name": "no",
                    "text": "No",
                    "type": "button",
                    "value": "bad"
                }
            ]
        }
    ]
}
```

사용자가 Slack 메시지 내의 단추를 클릭하면 봇은 채널 데이터 속성이 `payload` JSON 개체로 채워진 응답 메시지를 받게 됩니다.
`payload` 개체는 원본 메시지의 콘텐츠를 지정하고, 클릭된 단추를 식별하고, 단추를 클릭한 사용자를 식별합니다.

이 코드 조각은 사용자가 Slack 메시지에서 단추를 클릭할 때 봇에서 수신하는 메시지에서 `channelData` 속성의 예제를 보여줍니다.

```json
"channelData": {
    "payload": {
        "actions": [
            {
                "name": "recommend",
                "value": "yes"
            }
        ],
        . . .
        "original_message": "{…}",
        "response_url": "https://hooks.slack.com/actions/..."
    }
}
```

봇은 일반적인 방식으로 이 메시지에 응답하거나 `payload` 개체의 `response_url` 속성으로 지정된 엔드포인트에 해당 응답을 직접 게시할 수 있습니다.
`response_url`에 응답을 게시하는 시기 및 방법에 대한 정보는 <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack 단추</a>를 참조하세요.

다음 JSON을 사용하여 동적 단추를 만들 수 있습니다.

```json
{
    "text": "Would you like to play a game ? ",
    "attachments": [
        {
            "text": "Choose a game to play!",
            "fallback": "You are unable to choose a game",
            "callback_id": "wopr_game",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "game",
                    "text": "Chess",
                    "type": "button",
                    "value": "chess"
                },
                {
                    "name": "game",
                    "text": "Falken's Maze",
                    "type": "button",
                    "value": "maze"
                },
                {
                    "name": "game",
                    "text": "Thermonuclear War",
                    "style": "danger",
                    "type": "button",
                    "value": "war",
                    "confirm": {
                        "title": "Are you sure?",
                        "text": "Wouldn't you prefer a good game of chess?",
                        "ok_text": "Yes",
                        "dismiss_text": "No"
                    }
                }
            ]
        }
    ]
}
```

대화형 메뉴를 만들려면 다음 JSON을 사용합니다.

```json
{
    "text": "Would you like to play a game ? ",
    "response_type": "in_channel",
    "attachments": [
        {
            "text": "Choose a game to play",
            "fallback": "If you could read this message, you'd be choosing something fun to do right now.",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "callback_id": "game_selection",
            "actions": [
                {
                    "name": "games_list",
                    "text": "Pick a game...",
                    "type": "select",
                    "options": [
                        {
                            "text": "Hearts",
                            "value": "menu_id_hearts"
                        },
                        {
                            "text": "Bridge",
                            "value": "menu_id_bridge"
                        },
                        {
                            "text": "Checkers",
                            "value": "menu_id_checkers"
                        },
                        {
                            "text": "Chess",
                            "value": "menu_id_chess"
                        },
                        {
                            "text": "Poker",
                            "value": "menu_id_poker"
                        },
                        {
                            "text": "Falken's Maze",
                            "value": "menu_id_maze"
                        },
                        {
                            "text": "Global Thermonuclear War",
                            "value": "menu_id_war"
                        }
                    ]
                }
            ]
        }
    ]
}
```

## <a name="create-a-facebook-notification"></a>Facebook 알림 만들기

Facebook 알림을 만들려면 활동 개체의 채널 데이터 속성을 이러한 속성을 지정하는 JSON 개체로 설정합니다.

| 자산 | 설명 |
|----|----|
| notification_type | 알림 유형(예: **REGULAR**, **SILENT_PUSH**, **NO_PUSH**)
| 첨부 파일 | 이미지, 비디오 또는 다른 멀티미디어 형식을 지정하는 첨부 파일 또는 영수증과 같은 템플릿 기반 첨부 파일입니다. |

> [!NOTE]
> `notification_type` 속성 및 `attachment` 속성의 형식 및 콘텐츠에 대한 자세한 내용은 <a href="https://developers.facebook.com/docs/messenger-platform/send-api-reference#guidelines" target="_blank">Facebook API 설명서</a>를 참조하세요. 

이 코드 조각은 Facebook 영수증 첨부 파일에 대한 `channelData` 속성의 예제를 보여 줍니다.

```json
"channelData": {
    "notification_type": "NO_PUSH",
    "attachment": {
        "type": "template"
        "payload": {
            "template_type": "receipt",
            . . .
        }
    }
}
```

## <a name="create-a-telegram-message"></a>Telegram 메시지 만들기

음성 메모 또는 스티커 공유와 같은 Telegram 관련 작업을 구현하는 메시지를 만들려면 활동 개체의 채널 데이터 속성을 이러한 속성을 지정하는 JSON 개체로 설정합니다. 

| 자산 | 설명 |
|----|----|
| 메서드 | 호출할 Telegram Bot API 메서드입니다. |
| 매개 변수 | 지정된 메서드의 매개 변수입니다. |

다음과 같은 Telegram 메서드가 지원됩니다. 

- answerInlineQuery
- editMessageCaption
- editMessageReplyMarkup
- editMessageText
- forwardMessage
- kickChatMember
- sendAudio
- sendChatAction
- sendContact
- sendDocument
- sendLocation
- sendMessage
- sendPhoto
- sendSticker
- sendVenue
- sendVideo
- sendVoice
- unbanChateMember

이러한 Telegram 메서드 및 해당 매개 변수에 대한 자세한 내용은 <a href="https://core.telegram.org/bots/api#available-methods" target="_blank">Telegram Bot API 설명서</a>를 참조하세요.

> [!NOTE]
> <ul><li><code>chat_id</code> 매개 변수는 모든 Telegram 메서드에 공통적입니다. 매개 변수로 <code>chat_id</code>를 지정하지 않는 경우 프레임워크는 ID를 제공합니다.</li>
> <li>파일 콘텐츠 인라인을 전달하는 대신 아래 예제에서와 같이 URL 및 미디어 형식을 사용하여 파일을 지정합니다.</li>
> <li>봇이 Telegram 채널에서 받는 각 메시지 내에서 <code>ChannelData</code> 속성은 봇이 이전에 전송한 메시지를 포함합니다.</li></ul>

이 코드 조각은 단일 Telegram 메서드를 지정하는 `channelData` 속성의 예제를 보여 줍니다.

```json
"channelData": {
    "method": "sendSticker",
    "parameters": {
        "sticker": {
            "url": "https://domain.com/path/gif",
            "mediaType": "image/gif",
        }
    }
}
```

이 코드 조각은 Telegram 메서드의 배열을 지정하는 `channelData` 속성의 예제를 보여줍니다.

```json
"channelData": [
    {
        "method": "sendSticker",
        "parameters": {
            "sticker": {
                "url": "https://domain.com/path/gif",
                "mediaType": "image/gif",
            }
        }
    },
    {
        "method": "sendMessage",
        "parameters": {
            "text": "<b>This message is HTML formatted.</b>",
            "parse_mode": "HTML"
        }
    }
]
```

## <a name="create-a-native-kik-message"></a>네이티브 Kik 메시지 만들기

네이티브 Kik 메시지를 만들려면 활동 개체의 채널 데이터 속성을 이 속성을 지정하는 JSON 개체로 설정합니다.

| 자산 | 설명 |
|----|----|
| 메시지의 최대 전달 수 | Kik 메시지의 배열입니다. Kik 메시지 형식에 대한 자세한 내용은 <a href="https://dev.kik.com/#/docs/messaging#message-formats" target="_blank">Kik 메시지 형식</a>을 참조하세요. |

이 코드 조각은 네이티브 Kik 메시지에 대한 `channelData` 속성의 예제를 보여 줍니다.

```json
"channelData": {
    "messages": [
        {
            "chatId": "c6dd8165…",
            "type": "link",
            "to": "kikhandle",
            "title": "My Webpage",
            "text": "Some text to display",
            "url": "http://botframework.com",
            "picUrl": "http://lorempixel.com/400/200/",
            "attribution": {
                "name": "My App",
                "iconUrl": "http://lorempixel.com/50/50/"
            },
            "noForward": true,
            "kikJsData": {
                    "key": "value"
                }
        }
    ]
}
```

## <a name="create-a-line-message"></a>LINE 메시지 만들기

LINE 관련 메시지 유형(예: 스티커, 템플릿 또는 휴대폰 카메라 열기 같은 LINE 관련 작업 유형)을 구현하는 메시지를 만들려면 활동 개체의 채널 데이터 속성을 LINE 메시지 유형 및 작업 유형을 지정하는 JSON 개체로 설정하세요. 

| 자산 | 설명 |
|----|----|
| 형식 | LINE 작업/메시지 유형 이름 |

다음과 같은 LINE 메시지 유형이 지원됩니다.
* 스티커
* 이미지맵 
* 템플릿(단추, 확인, 회전식) 
* Flex 

이러한 LINE 작업은 메시지 유형 JSON 개체의 작업 필드에 지정할 수 있습니다. 
* 포스트백 
* Message 
* URI 
* Datetimerpicker 
* Camera 
* 카메라 앨범 
* 위치 

이러한 LINE 메서드 및 해당 매개 변수에 대한 자세한 내용은 [LINE Bot API 설명서](https://developers.line.biz/en/docs/messaging-api/)를 참조하세요. 

이 코드 조각은 채널 메시지 유형 `ButtonTemplate` 및 camera, cameraRoll, Datetimepicker와 같은 3개의 작업 유형을 지정하는 `channelData` 속성의 예를 보여줍니다. 

```json
"channelData": { 
    "type": "ButtonsTemplate", 
    "altText": "This is a buttons template", 
    "template": { 
        "type": "buttons", 
        "thumbnailImageUrl": "https://example.com/bot/images/image.jpg", 
        "imageAspectRatio": "rectangle", 
        "imageSize": "cover", 
        "imageBackgroundColor": "#FFFFFF", 
        "title": "Menu", 
        "text": "Please select", 
        "defaultAction": { 
            "type": "uri", 
            "label": "View detail", 
            "uri": "http://example.com/page/123" 
        }, 
        "actions": [{ 
                "type": "cameraRoll", 
                "label": "Camera roll" 
            }, 
            { 
                "type": "camera", 
                "label": "Camera" 
            }, 
            { 
                "type": "datetimepicker", 
                "label": "Select date", 
                "data": "storeId=12345", 
                "mode": "datetime", 
                "initial": "2017-12-25t00:00", 
                "max": "2018-01-24t23:59", 
                "min": "2017-12-25t00:00" 
            } 
        ] 
    } 
}
```

## <a name="additional-resources"></a>추가 리소스

- [엔터티 및 작업 형식](../bot-service-activities-entities.md)
- [Bot Framework 작업 스키마](https://aka.ms/botSpecs-activitySchema)
