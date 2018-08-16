---
title: 대화 종료 | Microsoft Docs
description: Direct Line API v3.0을 사용하여 대화를 종료하는 방법에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ac984609acfdd8f85088bd47ccded1f45e953b2c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302826"
---
# <a name="end-a-conversation"></a>대화 종료

클라이언트나 봇이 **endOfConversation** [활동](bot-framework-rest-connector-activities.md)을 보내 Direct Line 대화의 종료를 표시할 수 있습니다.  

## <a name="send-an-endofconversation-activity"></a>endOfConversation 활동 보내기

**endOfConversation** 활동은 봇과 클라이언트 간 통신을 종료합니다. **endOfConversation** 활동을 보낸 후에도 클라이언트가 `HTTP GET`을 사용하여 [메시지를 검색](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get)하려 할 수 있으나 클라이언트나 봇 모두 대화에 추가 메시지를 보낼 수 없습니다. 

대화를 종료하려면 간단히 POST 요청을 실행하여 **endOfConversation** 활동을 보냅니다.

### <a name="request"></a>요청

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
    "type": "endOfConversation",
    "from": {
        "id": "user1"
    }
}
```

### <a name="response"></a>response

요청에 성공하면 응답에 보낸 활동에 대한 ID가 포함됩니다.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0004"
}
```

## <a name="additional-resources"></a>추가 리소스

- [주요 개념](bot-framework-rest-direct-line-3-0-concepts.md)
- [인증](bot-framework-rest-direct-line-3-0-authentication.md)
- [봇에 활동 보내기](bot-framework-rest-direct-line-3-0-send-activity.md)
