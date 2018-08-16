---
title: 인증 | Microsoft Docs
description: 직접 회선 API v1.1에서 API 요청을 인증하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 6e83cc45e925f5d94b70260de6ac54e6f4052ca4
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301906"
---
# <a name="authentication"></a>인증

> [!IMPORTANT]
> 이 문서에서는 직접 회선 API 1.1의 인증에 대해 설명합니다. 클라이언트 응용 프로그램과 봇 간에 새 연결을 만들 경우에는 [직접 회선 API 3.0](bot-framework-rest-direct-line-3-0-authentication.md)을 대신 사용합니다.

클라이언트는 [Bot Framework 포털](../bot-service-channel-connect-directline.md)의 직접 회선 채널 구성 페이지에서 가져오는 **비밀**을 사용하거나 런타임에 가져오는 **토큰**을 사용하여 직접 회선 API 1.1에 대한 요청을 인증할 수 있습니다.

비밀 또는 토큰은 "Bearer" 체계 또는 "BotConnector" 체계를 사용하여 각 요청의 `Authorization` 헤더에 지정해야 합니다. 

**Bearer 체계**:
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**BotConnector 체계**:
```http
Authorization: BotConnector SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>비밀 및 토큰

직접 회선 **비밀**은 연결된 봇에 속하는 모든 대화에 액세스하는 데 사용할 수 있는 마스터 키입니다. **토큰**을 가져오는 데도 **비밀**을 사용할 수 있습니다. 비밀은 만료되지 않습니다. 

직접 회선 **토큰**은 단일 대화에 액세스하는 데 사용할 수 있는 키입니다. 토큰은 만료되지만 새로 고칠 수 있습니다. 

서비스 간 응용 프로그램을 만드는 경우 직접 회선 API 요청의 `Authorization` 헤더에 **비밀**을 지정하는 것이 가장 간단한 방법일 수 있습니다. 클라이언트가 웹 브라우저 또는 모바일 앱에서 실행되는 응용 프로그램을 작성하는 경우 비밀을 토큰(단일 대화에서만 가능하고 새로 고치지 않으면 만료됨)으로 교환하고 직접 회선 API 요청의 `Authorization` 헤더에서 **토큰**을 지정할 수 있습니다. 본인에게 가장 적합한 보안 모델을 선택합니다.

> [!NOTE]
> 직접 회선 클라이언트 자격 증명은 봇의 자격 증명과 다릅니다. 이 자격 증명을 사용하면 키를 별도로 수정할 수 있으며, 봇 암호를 공개하지 않고도 클라이언트 토큰을 공유할 수 있습니다. 

## <a name="get-a-direct-line-secret"></a>직접 회선 비밀 가져오기

<a href="https://dev.botframework.com/" target="_blank">Bot Framework 포털</a>에서 봇에 대한 직접 회선 채널 구성 페이지를 통해 [직접 회선 비밀을 가져올 수 있습니다](../bot-service-channel-connect-directline.md).

![직접 회선 구성](../media/direct-line-configure.png)

## <a id="generate-token"></a> 직접 회선 토큰 생성

단일 대화에 액세스하는 데 사용할 수 있는 직접 회선 토큰을 생성하려면 먼저 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 포털</a>의 직접 회선 채널 구성 페이지에서 직접 회선 비밀을 획득합니다. 그런 다음, 이 요청을 실행하여 직접 회선 비밀을 직접 회선 토큰과 교환합니다.

```http
POST https://directline.botframework.com/api/tokens/conversation
Authorization: Bearer SECRET
```

이 요청의 `Authorization` 헤더에서 **SECRET**을 직접 회선 비밀 값으로 바꿉니다.

다음 코드 조각은 토큰 생성 요청 및 응답의 예제를 제공합니다.

### <a name="request"></a>요청

```http
POST https://directline.botframework.com/api/tokens/conversation
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>response

요청이 성공적이면 응답에는 하나의 대화에 유효한 토큰이 포함됩니다. 토큰은 30분 후에 만료됩니다. 토큰이 유효한 상태를 유지하려면 만료되기 전에 [토큰을 새로 고쳐야 합니다](#refresh-token).

```http
HTTP/1.1 200 OK
[other headers]

"RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn"
```

### <a name="generate-token-versus-start-conversation"></a>토큰 생성 및 대화 시작

토큰 생성 작업(`POST /api/tokens/conversation`)은 단일 대화에 액세스하는 데 사용할 수 있는 `token`을 반환한다는 점에서 [대화 시작](bot-framework-rest-direct-line-1-1-start-conversation.md) 작업(`POST /api/conversations`)과 비슷합니다. 그러나 대화 시작 작업과 달리, 토큰 생성 작업은 대화를 시작하거나 봇에 연결하지 않습니다. 

토큰을 클라이언트에 배포하고 대화를 시작하려는 경우 토큰 생성 작업을 사용합니다. 대화를 즉시 시작하려는 경우 대신 [대화 시작](bot-framework-rest-direct-line-1-1-start-conversation.md) 작업을 사용합니다.

## <a id="refresh-token"></a> 직접 회선 토큰 새로 고침

직접 회선 토큰은 생성된 시간부터 30분 동안 유효하며, 만료되지 않는 한, 횟수에 제한없이 새로 고칠 수 있습니다. 만료된 토큰은 새로 고칠 수 없습니다. 직접 회선 토큰을 새로 고치려면 다음 요청을 실행합니다.

```http
POST https://directline.botframework.com/api/tokens/{conversationId}/renew
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

이 요청의 URI에서 **{conversationId}** 를 토큰이 유효한 대화의 ID로 바꾸고, 이 요청의 `Authorization` 헤더에서 **TOKEN_TO_BE_REFRESHED**를 새로 고치려는 직접 회선 토큰으로 바꿉니다.

다음 코드 조각은 토큰 새로 고침 요청 및 응답의 예제를 제공합니다.

### <a name="request"></a>요청

```http
POST https://directline.botframework.com/api/tokens/abc123/renew
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>response

요청이 성공적이면 응답에는 이전 토큰과 동일한 대화에 대해 유효한 새 토큰이 포함됩니다. 새 토큰은 30분 후에 만료됩니다. 새 토큰이 유효한 상태를 유지하려면 만료되기 전에 [토큰을 새로 고쳐야 합니다](#refresh-token).

```http
HTTP/1.1 200 OK
[other headers]

"RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0"
```

## <a name="additional-resources"></a>추가 리소스

- [주요 개념](bot-framework-rest-direct-line-1-1-concepts.md)
- [직접 회선에 봇 연결](../bot-service-channel-connect-directline.md)