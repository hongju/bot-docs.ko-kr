---
title: 인증 | Microsoft Docs
description: 직접 회선 API v3.0에서 API 요청을 인증하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/10/2019
ms.openlocfilehash: 717a95d580bad218ade9a884522724f1c6b96ad7
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032644"
---
# <a name="authentication"></a>인증

클라이언트는 [Bot Framework 포털](../bot-service-channel-connect-directline.md)의 직접 회선 채널 구성 페이지에서 가져오는 **비밀**을 사용하거나 런타임에 가져오는 **토큰**을 사용하여 직접 회선 API 3.0에 대한 요청을 인증할 수 있습니다. 비밀 또는 토큰은 다음 형식을 사용하여 각 요청의 `Authorization` 헤더에 지정해야 합니다. 

```http
Authorization: Bearer SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>비밀 및 토큰

직접 회선 **비밀**은 연결된 봇에 속하는 모든 대화에 액세스하는 데 사용할 수 있는 마스터 키입니다. **토큰**을 가져오는 데도 **비밀**을 사용할 수 있습니다. 비밀은 만료되지 않습니다. 

직접 회선 **토큰**은 단일 대화에 액세스하는 데 사용할 수 있는 키입니다. 토큰은 만료되지만 새로 고칠 수 있습니다. 

서비스 간 애플리케이션을 만드는 경우 직접 회선 API 요청의 `Authorization` 헤더에 **비밀**을 지정하는 것이 가장 간단한 방법일 수 있습니다. 클라이언트가 웹 브라우저 또는 모바일 앱에서 실행되는 애플리케이션을 작성하는 경우 비밀을 토큰(단일 대화에서만 가능하고 새로 고치지 않으면 만료됨)으로 교환하고 직접 회선 API 요청의 `Authorization` 헤더에서 **토큰**을 지정할 수 있습니다. 본인에게 가장 적합한 보안 모델을 선택합니다.

> [!NOTE]
> 직접 회선 클라이언트 자격 증명은 봇의 자격 증명과 다릅니다. 이 자격 증명을 사용하면 키를 별도로 수정할 수 있으며, 봇 암호를 공개하지 않고도 클라이언트 토큰을 공유할 수 있습니다. 

## <a name="get-a-direct-line-secret"></a>직접 회선 비밀 가져오기

<a href="https://dev.botframework.com/" target="_blank">Bot Framework 포털</a>에서 봇에 대한 직접 회선 채널 구성 페이지를 통해 [직접 회선 비밀을 가져올 수 있습니다](../bot-service-channel-connect-directline.md).

![직접 회선 구성](../media/direct-line-configure.png)

## <a id="generate-token"></a> 직접 회선 토큰 생성

단일 대화에 액세스하는 데 사용할 수 있는 직접 회선 토큰을 생성하려면 먼저 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 포털</a>의 직접 회선 채널 구성 페이지에서 직접 회선 비밀을 획득합니다. 그런 다음, 이 요청을 실행하여 직접 회선 비밀을 직접 회선 토큰과 교환합니다.

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer SECRET
```

이 요청의 `Authorization` 헤더에서 **SECRET**을 직접 회선 비밀 값으로 바꿉니다.

다음 코드 조각은 토큰 생성 요청 및 응답의 예제를 제공합니다.

### <a name="request"></a>요청

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

토큰 매개 변수를 포함하는 요청 페이로드는 선택 사항이지만 권장됩니다. Direct Line 서비스에 돌려 보낼 수 있는 토큰을 생성할 때는 더 안전한 연결을 위해 다음 페이로드를 제공합니다. 이 값을 포함하면 Direct Line이 사용자 ID와 이름의 추가적인 보안 유효성 검사를 수행하여 악의적 클라이언트에 의한 값 변조를 방지할 수 있습니다. 또한 이 값을 통해 Direct Line의 _대화 업데이트_ 작업 보내기 기능을 개선하여 사용자가 대화에 참가하는 즉시 대화 업데이트를 생성하도록 할 수 있습니다. 이 정보를 제공하지 않은 경우 사용자가 콘텐츠를 보내야 Direct Line이 대화 업데이트를 보낼 수 있습니다.

```json
{
  "user": {
    "id": "string",
    "name": "string"
  },
  "trustedOrigins": [
    "string"
  ]
}
```

| 매개 변수 | Type | 설명 |
| :--- | :--- | :--- |
| `user.id` | string | 선택 사항입니다. 토큰 안에 인코딩할 사용자의 채널 특정 ID입니다. Direct Line 사용자의 경우 이 값이 `dl_`로 시작되어야 합니다. 각 대화마다 고유한 사용자 ID를 만들 수 있고 보안을 강화하기 위해 이 ID는 추측할 수 없어야 합니다. |
| `user.name` | string | 선택 사항입니다. 토큰 안에 인코딩할 사용자의 친숙한 표시 이름입니다. |
| `trustedOrigins` | 문자열 배열 | 선택 사항입니다. 토큰 안에 포함할 신뢰할 수 있는 도메인 목록입니다. 봇의 웹 채팅 클라이언트를 호스팅할 수 있는 도메인입니다. 봇에 대한 Direct Line 구성 페이지의 목록과 일치해야 합니다. |

### <a name="response"></a>response

요청이 성공하면 응답에는 하나의 대화에 유효한 `token`과 토큰이 만료될 때까지의 시간(초)을 나타내는 `expires_in` 값이 포함됩니다. 토큰이 유효한 상태를 유지하려면 만료되기 전에 [토큰을 새로 고쳐야 합니다](#refresh-token).

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

### <a name="generate-token-versus-start-conversation"></a>토큰 생성 및 대화 시작

토큰 생성 작업(`POST /v3/directline/tokens/generate`)은 단일 대화에 액세스하는 데 사용할 수 있는 `token`을 반환한다는 점에서 [대화 시작](bot-framework-rest-direct-line-3-0-start-conversation.md) 작업(`POST /v3/directline/conversations`)과 비슷합니다. 그러나 대화 시작 작업과 달리, 토큰 생성 작업은 대화를 시작하지 않고, 봇에 연결하지 않고, 스트리밍 WebSocket URL을 만들지 않습니다. 

토큰을 클라이언트에 배포하고 대화를 시작하려는 경우 토큰 생성 작업을 사용합니다. 대화를 즉시 시작하려는 경우 대신 [대화 시작](bot-framework-rest-direct-line-3-0-start-conversation.md) 작업을 사용합니다.

## <a id="refresh-token"></a> 직접 회선 토큰 새로 고침

직접 회선 토큰은 만료되지만 않으면 횟수에 제한없이 새로 고칠 수 있습니다. 만료된 토큰은 새로 고칠 수 없습니다. 직접 회선 토큰을 새로 고치려면 다음 요청을 실행합니다. 

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

이 요청의 `Authorization` 헤더에서 **TOKEN_TO_BE_REFRESHED**를 새로 고칠 직접 회선 토큰과 바꿉니다.

다음 코드 조각은 토큰 새로 고침 요청 및 응답의 예제를 제공합니다.

### <a name="request"></a>요청

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>response

요청이 성공하면 응답에는 이전 토큰과 동일한 대화에 대해 유효한 새로운 `token`과 토큰이 만료될 때까지의 시간(초)을 나타내는 `expires_in` 값이 포함됩니다. 새 토큰이 유효한 상태를 유지하려면 만료되기 전에 [토큰을 새로 고쳐야 합니다](#refresh-token).

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0",
  "expires_in": 1800
}
```

## <a name="additional-resources"></a>추가 리소스

- [주요 개념](bot-framework-rest-direct-line-3-0-concepts.md)
- [직접 회선에 봇 연결](../bot-service-channel-connect-directline.md)