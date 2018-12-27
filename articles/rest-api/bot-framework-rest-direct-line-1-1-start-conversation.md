---
title: 대화 시작 | Microsoft Docs
description: 직접 회선 API v1.1을 사용하여 대화를 시작하는 방법에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: da81182d80ebac0d5aaba5a2660899d87c7e2b40
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000220"
---
# <a name="start-a-conversation"></a>대화 시작

> [!IMPORTANT]
> 이 문서에서는 직접 회선 API 1.1을 사용하여 대화를 시작하는 방법을 설명합니다. 클라이언트 애플리케이션과 봇 간에 새 연결을 만들 경우에는 [직접 회선 API 3.0](bot-framework-rest-direct-line-3-0-start-conversation.md)을 대신 사용합니다.

직접 회선 대화는 클라이언트에서 명시적으로 열고, 봇과 클라이언트가 참여하고 유효한 자격 증명이 있는 경우에만 실행될 수 있습니다. 대화가 열려 있는 동안 봇과 클라이언트 모두 메시지를 보낼 수 있습니다. 둘 이상의 클라이언트가 지정된 대화에 연결할 수 있으며 각 클라이언트는 여러 사용자를 대신하여 참여할 수 있습니다.

## <a name="open-a-new-conversation"></a>새 대화 열기

봇과 새 대화를 시작하려면 다음 요청을 실행합니다.

```http
POST https://directline.botframework.com/api/conversations
Authorization: Bearer SECRET_OR_TOKEN
```

다음 코드 조각은 대화 시작 요청 및 응답의 예제를 제공합니다.

### <a name="request"></a>요청

```http
POST https://directline.botframework.com/api/conversations
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>response

요청이 성공하면 응답에는 대화 ID, 토큰 및 토큰이 만료될 때까지의 시간(초)을 나타내는 값이 포함됩니다.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800
}
```

## <a name="start-conversation-versus-generate-token"></a>대화 시작과 토큰 생성 비교

대화 시작 작업(`POST /api/conversations`)은 단일 대화에 액세스하는 데 사용할 수 있는 `token`을 반환한다는 점에서 [토큰 생성](bot-framework-rest-direct-line-1-1-authentication.md#generate-token) 작업(`POST /api/tokens/conversation`)과 비슷합니다. 그러나 대화 시작 작업은 대화를 시작하고 봇에 연결하는 반면 토큰 생성 작업은 이러한 작업을 수행하지 않습니다. 

대화를 즉시 시작하려는 경우 대화 시작 작업을 사용합니다. 토큰을 클라이언트에 배포하고 대화를 시작하려는 경우 [토큰 생성](bot-framework-rest-direct-line-1-1-authentication.md#generate-token) 작업을 대신 사용합니다. 

## <a name="additional-resources"></a>추가 리소스

- [주요 개념](bot-framework-rest-direct-line-1-1-concepts.md)
- [인증](bot-framework-rest-direct-line-1-1-authentication.md)