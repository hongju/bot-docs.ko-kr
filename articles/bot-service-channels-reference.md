---
title: 채널 참조
description: Bot Framework 채널 참조
keywords: 채널 참조, Bot Builder 채널, Bot Framework 채널
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 03/01/2019
ms.openlocfilehash: 0fb0f650b44d320d78a0ada5d46105048019964c
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224961"
---
# <a name="categorized-activities-by-channel"></a>분류된 채널별 활동

다음 표에는 채널별로 제공할 수 있는 이벤트(네트워크 활동)가 나와 있습니다.

표에 사용되는 기호는 다음과 같습니다.

기호              | 의미
:------------------:|:------------------------------------------------
:white_check_mark:  |봇에서 해당 활동(Activity)을 받아야 합니다.
:x:                 |봇에서 해당 활동을 받지 **않아야 합니다**.
:white_large_square:|현재 봇에서 해당 활동을 받을 수 있는지의 여부가 확정되어 있지 않습니다.

활동은 의미에 따라 별도의 범주로 분류할 수 있습니다. 각 범주별로 가능한 활동 표가 마련되어 있습니다.

<a name="conversational"></a>대화형
--------------

 \                      | Cortana            | 직접 회선        | Direct Line(웹 채팅) | Email              | Facebook           | GroupMe            | Kik                | Teams              | Slack              | Skype   | Skype(비즈니스용) | Telegram | Twilio  
:---------------------- | :-----:            | :----------------: | :--------------------: |:----:              | :------:           | :-----:            | :-----:            | :---:              | :---:              | :---:   | :------------: | :------: | :----:  
Message                 | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction         | :x:                | :x:                | :x:                    | :x:                | :x:                | :x:                | :x:                | :white_check_mark: | :x:                | :x:      | :x:             | :x:       | :x:      

- 모든 채널에서 메시지 활동을 보냅니다.
- Dialog를 사용하는 경우 일반적으로 메시지 활동은 항상 Dialog에 전달해야 합니다.
- MessageReaction은 대화의 매우 많은 부분을 차지하지만 실제로는 그렇지 않습니다.
- MessageReaction에는 논리적으로 Added 및 Removed의 두 가지 유형이 있습니다.


> [!TIP]
> "메시지 반응"은 이전 주석의 "엄지 손가락 위로"(thumps up)와 같은 것입니다. 그것들은 불규칙하게 발생할 수 있습니다. 그래서 그것들은 단추와 비슷하다고 생각할 수 있습니다. 이 활동은 현재 Teams 채널에서 보냅니다.  


<a name="welcome"></a>시작
-------

 \                         | Cortana            | 직접 회선        | Direct Line(웹 채팅) | Email   | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype(비즈니스용) | Telegram | Twilio  
:----------------------    | :-----:            | :---------:        | :--------------------: |:----:   | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
ConversationUpdate         | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :x:     | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                | :x:                | :x:                    | :x:     | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     

- 일반적으로 채널에서 ConversationUpdate 활동을 보냅니다.
- MessageReaction에는 논리적으로 Added 및 Removed의 두 가지 유형이 있습니다.
- "Welcome"(환영) 봇 동작은 단순히 ConversationUpdate.Added를 연결하여 구현할 수 있다고 가정하기 쉽지만, 가끔씩 작동하는 것입니다.
- 그러나 이 동작은 간소화된 것이므로 신뢰할 수 있는 "Welcome" 동작을 생성하려면 봇 구현에서 상태를 사용해야 할 수도 있습니다.


<a name="application-extensibility"></a>애플리케이션 확장성
-------------------------

 \                      | Cortana            | 직접 회선        | Direct Line(웹 채팅) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype(비즈니스용) | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.*                    | :white_large_square: | :white_check_mark: | :white_check_mark:    | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  

- Event 활동은 Direct Line(_aka Direct Line_)의 확장 메커니즘입니다.
- 클라이언트와 서버를 모두 소유한 애플리케이션에서 이 Event 활동을 사용하여 서비스를 통해 자체 이벤트를 터널링하도록 선택할 수 있습니다.


<a name="microsoft-teams"></a>Microsoft 팀
------------------

 \                      | Cortana            | 직접 회선        | Direct Line(웹 채팅) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype(비즈니스용) | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Invoke.TeamsVerification   | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     

- Microsoft Teams는 형식화된 다른 여러 활동과 함께 몇 가지 Teams 특정 Invoke 활동을 정의합니다.
- Invoke 활동은 애플리케이션에 관련되며 클라이언트에서 정의하는 것이 아닙니다.
- 활동의 특정 하위 형식만 호출한다는 일반적인 개념은 없습니다.
- Invoke는 현재 봇에서 요청-응답 동작을 트리거하는 유일한 활동입니다.

매우 중요한 사항으로, 대화(Dialogs)를 사용하여 OAuth 프롬프트에서 작동하려면 Invoke.TeamsVerification 활동을 Dialog에 전달해야 합니다.


<a name="message-update"></a>메시지 업데이트
--------------

 \                      | Cortana            | 직접 회선        | Direct Line(웹 채팅) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype(비즈니스용) | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
MessageUpdate | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     

- 메시지 업데이트는 현재 Teams에서 지원됩니다.


<a name="oauth"></a>OAuth
-------

 \                      | Cortana            | 직접 회선        | Direct Line(웹 채팅) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype(비즈니스용) | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.TokenResponse| :white_large_square:  | :white_check_mark:   | :white_check_mark:    | :x:    | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 

매우 중요한 사항으로, 대화(Dialogs)를 사용하여 OAuth 프롬프트에서 작동하려면 Event.TokenResponse 활동을 Dialog에 전달해야 합니다.


<a name="uncategorized"></a>범주화되지 않음 
-------------

 \                      | Cortana  | 직접 회선        | Direct Line(웹 채팅) | Email | Facebook | GroupMe | Kik     | Teams | Slack | Skype | Skype(비즈니스용) | Telegram | Twilio  
:---------------------- | :-----:  | :---------:        | :--------------------: |:----: | :------: | :-----: | :-----: | :---: | :---: | :---: | :------------: | :------: | :----:  
EndOfConversation       | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate      | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Typing                  | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                 | :x:      | :x:                | :x:                    | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     


<a name="out-of-use-includes-payment-specific-invoke"></a>사용 중지(Payment 특정 Invoke 포함)
---------------------------------------------
- DeleteUserData 
- Invoke.PaymentRequest  
- Invoke.Address
- Ping

---

## <a name="summary-of-activities-supported-per-channel"></a>지원되는 채널별 활동 요약

<a name="cortana"></a>Cortana
-------
- Message
- ConversationUpdate
- _Event.TokenResponse_
- _EndOfConversation(창을 닫는 경우?)_

<a name="direct-line"></a>직접 회선
--------
- Message
- ConversationUpdate
- Event.TokenResponse
- Event.*
- _Event.CreateConversation_
- _Event.ContinueConversation_

<a name="email"></a>Email
-----
- Message

<a name="facebook"></a>Facebook
--------
- Message
- _Event.TokenResponse_

<a name="groupme"></a>GroupMe
-------
- Message
- ConversationUpdate
- _Event.TokenResponse_

<a name="kik"></a>Kik
---
- Message
- ConversationUpdate
- _Event.TokenResponse_

<a name="teams"></a>Teams
-----
- Message
- ConversationUpdate
- MessageReaction
- MessageUpdate
- MessageDelete
- Invoke.TeamsVerification
- Invoke.ComposeResponse


<a name="slack"></a>Slack
-----
- Message
- ConversationUpdate
- _Event.TokenResponse_


<a name="skype"></a>Skype
-----
- Message
- ContactRelationUpdate
- _Event.TokenResponse_


<a name="skype-business"></a>Skype(비즈니스용)
--------------
- Message
- ContactRelationUpdate 
- _Event.TokenResponse_


<a name="telegram"></a>Telegram
--------
- Message
- ConversationUpdate
- _Event.TokenResponse_


<a name="twilio"></a>Twilio
------
- Message

## <a name="summary-table-all-activities-to-all-channels"></a>모든 채널의 모든 활동에 대한 요약 표

 \                         | Cortana              | 직접 회선          | Direct Line(웹 채팅) | Email                | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype(비즈니스용) | Telegram | Twilio  
:----------------------    | :-----:              | :---------:          | :--------------------: |:----:                | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Message                    | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :white_check_mark:   | :white_check_mark:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction            | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:      | :x:      | :x:             | :x:       | :x:      
ConversationUpdate         | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     
Event.*                    | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Invoke.TeamsVerification   | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
MessageUpdate              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
Event.TokenResponse        | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 
EndOfConversation          | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate         | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Typing                     | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                    | :x:                  | :x:                  | :x:                    | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     

## <a name="web-chat"></a>웹 채팅 
웹 채팅에서 보내는 항목은 다음과 같습니다.
- "message": "text" 및/또는 "attachments" 포함
- "event": "name" 및 "value"(JSON/문자열로) 포함
- "typing": 사용자가 "sendTypingIndicator" 옵션을 설정하면 웹 채팅에서 "contactRelationUpdate"를 보내지 않습니다. 웹 채팅은 "messageReaction"을 지원하지 않습니다. 이 기능을 지원하도록 명시적으로 요청하는 사용자가 없습니다.

웹 채팅에서 기본적으로 렌더링하는 항목은 다음과 같습니다.
- "message": 캐러셀 또는 스택 방식으로 렌더링되며, 활동의 옵션에 따라 달라집니다.
- "typing": 5초 동안 렌더링하여 숨기거나 다음 활동이 시작될 때까지 숨깁니다.
- "conversationUpdate": 숨깁니다
- "event": 숨깁니다
- 기타: 경고 상자를 표시합니다(프로덕션 환경에는 표시되지 않음). 이 렌더링 파이프라인을 수정하여 사용자 지정 렌더링을 추가, 제거 또는 대체할 수 있습니다.

웹 채팅을 사용하여 활동 유형과 페이로드를 보낼 수 있습니다. 이 기능은 문서화하거나 추천하지 않습니다. 대신 "이벤트" 활동을 사용해야 합니다.
