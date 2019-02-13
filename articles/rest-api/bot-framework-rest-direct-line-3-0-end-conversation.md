---
title: 대화 종료 | Microsoft Docs
description: Direct Line API v3.0을 사용하여 대화를 종료하는 방법에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 438558995f83ade38404856d61ba66ee77480a27
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711937"
---
# <a name="end-a-conversation"></a>대화 종료

**endOfConversation** [활동](bot-framework-rest-connector-activities.md)은 채널 또는 봇이 대화를 종료했음을 의미합니다. 

> [!NOTE] 
> **endOfConversation** 이벤트는 매우 적은 채널에서 전송되지만 Cortana 채널은 이를 수용하는 유일한 채널입니다. Direct Line을 포함한 다른 채널은 이 기능을 구현하지 않는 대신 해당 활동을 드롭 또는 전달합니다. 각 채널은 endOfConversation 활동에 대응하는 방법을 결정합니다. DirectLine 클라이언트를 설계할 때 봇이 이미 종료된 대화에 작업을 전송하는 경우에 오류를 생성하는 등 적절하게 동작하도록 클라이언트를 업데이트합니다.

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
