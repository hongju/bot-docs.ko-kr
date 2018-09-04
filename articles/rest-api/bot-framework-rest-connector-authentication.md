---
title: 요청 인증 | Microsoft Docs
description: Bot Connector API 및 Bot State API에서 API 인증을 요청하는 방법을 살펴봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 998586820a0489bc4cca1d25b53cb6ac8162c452
ms.sourcegitcommit: 0b2be801e55f6baa048b49c7211944480e83ba95
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/28/2018
ms.locfileid: "43115048"
---
# <a name="authentication"></a>인증

봇은 보안된 채널(SSL/TLS)의 HTTP를 통해 Bot Connector 서비스와 통신합니다. 봇이 Connector 서비스에 요청을 보낼 때는 Connector 서비스가 ID 확인에 사용할 수 있는 정보를 포함해야 합니다. 마찬가지로 Connector 서비스가 봇에 요청을 보낼 때는 봇이 ID 확인에 사용할 수 있는 정보를 포함해야 합니다. 이 문서에서는 봇과 Bot Connector 서비스 간에 발생하는 서비스 수준 인증을 위한 인증 기술과 요구 사항에 대해 설명합니다. 자체 인증 코드를 작성한다면 이 문서에서 설명하는 보안 절차를 구현하여 봇이 Bot Connector 서비스와의 메시지 교환을 사용하도록 설정해야 합니다.

> [!IMPORTANT]
> 자체 인증 코드를 작성할 경우 모든 보안 절차를 정확하게 구현하는 것이 중요합니다. 이 문서의 모든 단계를 구현하면 공격자가 봇에게 전송된 메시지를 읽고 봇을 가장하는 메시지를 보내고 비밀 키를 도용하는 위험을 줄일 수 있습니다. 

[.NET용 Bot Builder SDK](../dotnet/bot-builder-dotnet-overview.md) 또는 [Node.js용 Bot Builder SDK](../nodejs/index.md)를 사용할 경우 SDK가 자동으로 작업을 수행하므로 이 문서의 보안 절차를 실행할 필요가 없습니다. [등록](../bot-service-quickstart-registration.md) 중에 봇에 대해 얻은 앱 ID와 암호로 프로젝트를 구성하기만 하면 SDK가 나머지를 처리합니다.

> [!WARNING]
> 2016년 12월에 Bot Framework 보안 프레임워크 v3.1이 도입되어 토큰 생성 및 유효성 검사 중에 사용되는 몇 가지 값에 대한 변경이 있었습니다. 2017년 늦가을에 토큰 생성 및 유효성 검사 중에 사용되는 값에 대한 변경 내용을 포함하는 Bot Framework 보안 프로토콜 v3.2가 도입되었습니다.
> 자세한 내용은 [보안 프로토콜 변경 내용](#security-protocol-changes)을 참조하세요.

## <a name="authentication-technologies"></a>인증 기술

봇과 Bot Connector 간에 신뢰를 설정하는 데는 4가지 인증 기술이 사용됩니다. 

| 기술 | 설명 |
|----|----|
| **SSL/TLS** | SSL/TLS는 모든 서비스 간 연결에 사용됩니다. `X.509v3` 인증서는 모든 HTTPS 서비스의 ID를 설정하는 데 사용됩니다. **클라이언트는 항상 서비스 인증서가 신뢰할 수 있고 유효한지 검사해야 합니다.** 클라이언트 인증서는 이 스키마의 일부로 사용되지 않습니다. |
| **OAuth 2.0** | MSA(Microsoft 계정)에 대한 OAuth 2.0 로그인/AAD v2 로그인 서비스는 봇이 메시지를 보내는 데 사용할 수 있는 보안 토큰을 생성하는 데 사용됩니다. 이 토큰은 서비스 간 토큰으로, 사용자 로그인이 필요하지 않습니다. |
| **JWT(JSON 웹 토큰)** | JSON 웹 토큰은 봇으로 오고 간 토큰을 인코딩하는 데 사용됩니다. 이 문서에서 설명한 요구 사항에 따라 **클라이언트는 수신한 모든 JWT 토큰**을 완전히 확인해야 합니다. |
| **OpenID 메타데이터** | Bot Connector 서비스는 잘 알려진 정적 엔드포인트에서 OpenID 메타데이터에 대한 자체 JWT 토큰을 서명하는 데 사용하는 유효한 토큰 목록을 게시합니다. |

이 문서에서는 표준 HTTPS 및 JSON을 통해 이러한 기술을 사용하는 방법을 설명합니다. OpenID 도우미 등이 유용할 수는 있지만 특수한 SDK가 필요하지 않습니다.

## <a id="bot-to-connector"></a> 봇에서 Bot Connector 서비스에 대한 요청 인증

Bot Connector Service와 통신하려면 다음 형식을 사용하여 각 API 요청의 `Authorization` 헤더에 액세스 토큰을 지정해야 합니다. 

```http
Authorization: Bearer ACCESS_TOKEN
```

이 다이어그램에서는 봇-커넥터 인증의 단계를 보여 줍니다.

![MSA 로그인 서비스와 봇에 대해 차례로 인증](../media/connector/auth_bot_to_bot_connector.png)

> [!IMPORTANT]
> 아직 수행하지 않은 경우 Bot Framework에 [봇을 등록](../bot-service-quickstart-registration.md)하여 해당 AppID 및 암호를 가져와야 합니다. 액세스 토큰을 요청하려면 봇의 AppID 및 암호가 필요합니다.

### <a name="step-1-request-an-access-token-from-the-msaaad-v2-login-service"></a>1단계: MSA/AAD v2 로그인 서비스에서 액세스 토큰 요청

MSA/AAD v2 로그인 서비스에서 액세스 토큰을 요청하려면 **MICROSOFT-APP-ID** 및 **MICROSOFT-APP-PASSWORD**를 Bot Framework에 봇을 [등록](../bot-service-quickstart-registration.md)할 때 획득한 앱 ID 및 암호로 바꾸어 다음 요청을 실행합니다.

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="step-2-obtain-the-jwt-token-from-the-msaaad-v2-login-service-response"></a>2단계: MSA/AAD v2 로그인 서비스 응답에서 JWT 토큰 가져오기

MSA/AAD v2 로그인 서비스에서 응용 프로그램이 인증되면 JSON 응답 본문은 액세스 토큰, 형식 및 만료(초)를 지정합니다. 

토큰을 요청의 `Authorization` 헤더에 추가할 때는 이 응답에 지정된 정확한 값을 사용해야 합니다(즉 이스케이프하지 않거나 토큰 값을 인코딩하지 않음). 액세스 토큰은 만료 시점까지 유효합니다. 봇의 성능에 영향을 주는 토큰 만료를 방지하기 위해 캐시하도록 선택하고 미리 토큰을 새로 고칠 수 있습니다.

이 예제에는 MSA/AAD v2 로그인 서비스로부터의 응답을 보여 줍니다.

```http
HTTP/1.1 200 OK
... (other headers) 
```

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

### <a name="step-3-specify-the-jwt-token-in-the-authorization-header-of-requests"></a>3단계: 요청의 인증 헤더에 JWT 토큰 지정

Bot Connector 서비스에 API 요청을 보낼 때 다음 형식을 사용하여 요청의 `Authorization` 헤더에 액세스 토큰을 지정합니다.

```http
Authorization: Bearer ACCESS_TOKEN
```

Bot Connector 서비스에 보내는 모든 요청은 `Authorization` 헤더에 액세스 토큰을 포함해야 합니다. 토큰이 올바른 형식이며 만료되지 않았고 MSA/AAD v2 로그인 서비스에서 생성되었다면 Bot Connector 서비스가 요청을 인증합니다. 추가 검사를 수행하여 토큰이 요청을 보낸 봇에 속해 있는지 확인합니다.

다음 예제에서는 요청의 `Authorization` 헤더에서 액세스 토큰을 지정하는 방법을 보여 줍니다. 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/12345/activities 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
    
(JSON-serialized Activity message goes here)
```

> [!IMPORTANT]
> 사용자가 Bot Connector 서비스에 보낸 요청의 `Authorization` 헤더에만 JWT 토큰을 지정합니다. 보안되지 않은 채널을 통해 토큰을 보내거나 다른 서비스로 보낸 HTTP 요청에 토큰을 포함하지 않아야 합니다. MSA/AAD v2 로그인 서비스로부터 구한 JWT 토큰은 암호와 유사하며 세심한 주의가 필요합니다. 이 토큰을 소유한 모든 사람은 봇을 대시하여 작업을 수행하는 데 사용할 수 있습니다. 

#### <a name="bot-to-connector-example-jwt-components"></a>커넥터에 대한 봇: JWT 구성 요소 예제

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "https://api.botframework.com",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  appid: "<YOUR MICROSOFT APP ID>",
  ... other fields follow
}
```

> [!NOTE]
> 현실에서는 실제 필드가 다를 수 있습니다. 위에서 지정한 대로 모든 JWT 토큰을 만들고 그 유효성을 검사합니다.

## <a id="connector-to-bot"></a> Bot Connector 서비스에서 봇에 대한 요청 인증

Bot Connector 서비스가 봇에 요청을 보낼 때는 요청의 `Authorization` 헤더에 서명된 JWT 토큰을 지정합니다. 봇은 서명된 JWT 토큰의 인증을 확인하여 Bot Connector 서비스로부터의 호출을 인증할 수 있습니다. 

이 다이어그램에서는 커넥터-봇 인증의 단계를 보여 줍니다.

![Bot Connector에서 봇에 대한 호출 인증](../media/connector/auth_bot_connector_to_bot.png)

### <a id="openid-metadata-document"></a> 2단계: OpenID 메타 데이터 문서 가져오기

OpenID 메타데이터 문서는 Bot Connector 서비스의 유효한 서명 키를 나열하는 두 번째 문서의 위치를 지정합니다. OpenID 메타데이터 문서를 가져오려면 HTTPS를 통해 이 요청을 실행합니다.

```http
GET https://login.botframework.com/v1/.well-known/openidconfiguration
```

> [!TIP]
> 이것은 응용 프로그램에 하드 코딩할 수 있는 정적 URL입니다. 

다음 예제에서는 `GET` 요청에 대한 응답으로 반환되는 OpenID 메타데이터 문서를 보여 줍니다. `jwks_uri` 속성은 Bot Connector 서비스의 유효한 서명 키가 포함된 문서의 위치를 지정합니다.

```json
{
    "issuer": "https://api.botframework.com",
    "authorization_endpoint": "https://invalid.botframework.com",
    "jwks_uri": "https://login.botframework.com/v1/.well-known/keys",
    "id_token_signing_alg_values_supported": [
      "RS256"
    ],
    "token_endpoint_auth_methods_supported": [
      "private_key_jwt"
    ]
}
```

### <a id="connector-to-bot-step-3"></a> 3단계: 유효한 서명 키 목록 가져오기

유효한 서명 키의 목록을 가져오려면 OpenID 메타데이터 문서에서 `jwks_uri` 속성이 지정한 URL에 대한 HTTPS를 통해 `GET` 요청을 실행합니다. 예: 

```http
GET https://login.botframework.com/v1/.well-known/keys
```

응답 본문이 [JWK 형식](https://tools.ietf.org/html/rfc7517)으로 문서를 지정하지만 각 키에 대한 추가 속성 `endorsements`도 포함합니다. 키 목록은 비교적 안정적이며 장시간 캐시될 수 있습니다(기본적으로 Bot Builder SDK 안에서 5일).

각 키 안의 `endorsements` 속성은 들어오는 요청의 [Activity][Activity] 개체 내 `channelId` 속성에서 지정한 채널 ID가 진짜인지 확인하는 데 사용할 수 있는 하나 이상의 인증 문자열을 포함합니다. 인증이 필요한 채널 ID 목록은 각 봇 안에서 구성할 수 있습니다. 봇 개발자가 선택한 채널 ID 값을 어떤 식으로든 재정의할 수는 있지만 기본적으로 모든 게시된 채널 ID의 목록입니다. 채널 ID에 대한 인증 필요한 경우

- 이 채널 ID로 봇에 전송된 모든 [Activity][Activity] 개체에는 해당 채널에 대한 인증을 통해 서명된 JWT 토큰이 함께 있도록 요구해야 합니다. 
- 인증이 없으면 봇은 **HTTP 403(금지 됨)** 상태 코드를 반환하여 요청을 거부해야 합니다.

### <a name="step-4-verify-the-jwt-token"></a>4단계: JWT 토큰 확인

Bot Connector 서비스에서 보낸 토큰의 신뢰성을 확인하려면 요청의 `Authorization` 헤더에서 토큰을 추출하고 구문 분석하여 그 콘텐츠와 서명을 확인해야 합니다. 

JWT 구문 분석 라이브러리는 일반적으로 특정 토큰 문자(발급자, 대상 등)가 정확한 값을 포함하도록 요구하기 위해 이 라이브러리를 구성해야 하기는 하지만, 여러 폴랫폼에 사용할 수 있고 대부분은 JWT 토큰을 위한 안전하고 믿을 수 있는 구문 분석을 구현합니다.  토큰을 구문 분석할 때는 구문 분석 라이브러리를 구성하거나 자체 유효성 검사를 작성하여 토큰이 이러한 요구 사항에 부합하도록 해야 합니다.

1. 토큰이 HTTP `Authorization` 헤더에서 "Bearer" 스키마로 전송되었습니다.
2. 토큰이 [JWT 표준](http://openid.net/specs/draft-jones-json-web-token-07.html)에 부합하는 유효한 JSON입니다.
3. 토큰이 값이 `https://api.botframework.com`인 "issuer" 클레임을 포함합니다.
4. 토큰이 봇의 Microsoft 앱 ID와 동일한 값을 갖는 "audience" 클레임을 포함합니다.
5. 토큰이 유효 기간 범위 안에 있습니다. 업계 표준 클록 오차는 5분입니다.
6. 토큰이 [2단계](#openid-metadata-document)에서 검색한 Open ID 메타데이터의 `id_token_signing_alg_values_supported` 속성에서 지정한 서명 알고리즘을 사용하여 [3단계](#connector-to-bot-step-3)에서 검색한 OpenID 키 문서에 나열된 키와 유효한 암호화 서명을 갖습니다.
7. 토큰이 들어오는 요청의 [Activity][Activity] 개체 루트에서 `servieUrl` 속성과 일치하는 값과 "serviceUrl" 클레임을 포함합니다. 

토큰이 이 요구 사항에 일부라도 부합하지 않으면 봇은 **HTTP 403(금지 됨)** 상태 코드를 반환하여 요청을 거부해야 합니다.

> [!IMPORTANT]
> 이 요구 사항은 모두 중요하며 특히 4와 6번 요구 사항이 중요합니다. 이러한 인증 요구 사항을 하나라도 구현하지 못하면 봇이 JWT 토큰을 공개할 수 있는 공격에 노출됩니다.

구현자는 봇으로 전송되는 JWT 토큰의 유효성 검사를 비활성화하는 방법을 노출해서는 안 됩니다.

#### <a name="connector-to-bot-example-jwt-components"></a>봇에 대한 커넥터: JWT 구성 요소 예제

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOU MICROSOFT APP ID>",
  iss: "https://api.botframework.com",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> 현실에서는 실제 필드가 다를 수 있습니다. 위에서 지정한 대로 모든 JWT 토큰을 만들고 그 유효성을 검사합니다.

## <a id="emulator-to-bot"></a> Bot Framework Emulator에서 봇에 대한 요청 인증

> [!WARNING]
> 2017년 늦가을에 Bot Framework 보안 프로토콜 v3.2가 발표되었습니다. 이 새 버전에는 Bot Framework Eumaltor와 봇 간에 교환되는 토큰 내에 새 "issuer" 값이 포함됩니다. 이러한 변경에 대비하여 아래 단계에서는 v3.1 및 v3.2 발급자 값을 모두 확인하는 방법을 간략하게 설명합니다. 

[Bot Framework Emulator](../bot-service-debug-emulator.md)는 봇 기능 테스트에 사용할 수 있는 데스크톱 도구입니다. Bot Framework Emulator가 앞에서 설명한 것과 같은 [인증 기술](#authentication-technologies)을 사용하더라도 실제 Bot Connector 서비스를 가장하는 것은 불가능합니다. 대신 봇이 만든 것과 동일한 토큰을 만들기 위해 사용자가 에뮬레이터를 봇에 연결할 때 지정한 Microsoft 앱 ID 및 Microsoft 앱 암호를 사용합니다. 에뮬레이터가 봇에 요청을 보내면 요청의 `Authorization` 헤더에서 JWT 토큰을 지정합니다. 기본적으로 봇의 자체 자격 증명을 사용하여 요청을 인증하게 됩니다. 

인증 라이브러리를 구현하고 Bot Framework Emulator의 요청을 수락하려면 이 추가적인 인증 경로를 추가해야 합니다. 이 경로는 구조적으로 [커넥터 -> 봇](#connector-to-bot) 확인 경로와 유사하지만 봇 커넥터의 OpenID 문서가 아닌 MSA의 OpenID 문서를 사용합니다.

이 다이어그램에서는 에뮬레이터-봇 인증의 단계를 보여 줍니다.

![Bot Framework Emulator에서 봇에 대한 호출 인증](../media/connector/auth_bot_framework_emulator_to_bot.png)

---
### <a name="step-2-get-the-msa-openid-metadata-document"></a>2단계: MSA OpenID 메타 데이터 문서 가져오기

OpenID 메타데이터 문서는 유효한 서명 키를 나열하는 두 번째 문서의 위치를 지정합니다. MSA OpenID 메타데이터 문서를 가져오려면 HTTPS를 통해 이 요청을 실행합니다.

```http
GET https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration
```

다음 예제에서는 `GET` 요청에 대한 응답으로 반환되는 OpenID 메타데이터 문서를 보여 줍니다. `jwks_uri` 속성은 유효한 서명 키가 포함된 문서의 위치를 지정합니다.

```json
{
    "authorization_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/authorize",
    "token_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/token",
    "token_endpoint_auth_methods_supported":["client_secret_post","private_key_jwt"],
    "jwks_uri":"https://login.microsoftonline.com/common/discovery/v2.0/keys",
    ...
}
```

### <a id="emulator-to-bot-step-3"></a> 3단계: 유효한 서명 키 목록 가져오기

유효한 서명 키의 목록을 가져오려면 OpenID 메타데이터 문서에서 `jwks_uri` 속성이 지정한 URL에 대한 HTTPS를 통해 `GET` 요청을 실행합니다. 예: 

```http
GET https://login.microsoftonline.com/common/discovery/v2.0/keys 
Host: login.microsoftonline.com
```

응답 본문은 [JWK 형식](https://tools.ietf.org/html/rfc7517)으로 문서를 지정합니다. 

### <a name="step-4-verify-the-jwt-token"></a>4단계: JWT 토큰 확인

에뮬레이터에서 보낸 토큰의 신뢰성을 확인하려면 요청의 `Authorization` 헤더에서 토큰을 추출하고 구문 분석하여 그 콘텐츠와 서명을 확인해야 합니다. 

JWT 구문 분석 라이브러리는 일반적으로 특정 토큰 문자(발급자, 대상 등)가 정확한 값을 포함하도록 요구하기 위해 이 라이브러리를 구성해야 하기는 하지만, 여러 폴랫폼에 사용할 수 있고 대부분은 JWT 토큰을 위한 안전하고 믿을 수 있는 구문 분석을 구현합니다.  토큰을 구문 분석할 때는 구문 분석 라이브러리를 구성하거나 자체 유효성 검사를 작성하여 토큰이 이러한 요구 사항에 부합하도록 해야 합니다.

1. 토큰이 HTTP `Authorization` 헤더에서 "Bearer" 스키마로 전송되었습니다.
2. 토큰이 [JWT 표준](http://openid.net/specs/draft-jones-json-web-token-07.html)에 부합하는 유효한 JSON입니다.
3. 토큰이 값이 `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` 또는 `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/`인 "issuer" 클레임을 포함합니다. 두 발급자 값에 대한 확인을 통해 보안 프로토콜 v3.1 및 v3.2 발급자 값을 모두 확인하도록 합니다.
4. 토큰이 봇의 Microsoft 앱 ID와 동일한 값을 갖는 "audience" 클레임을 포함합니다.
5. 토큰이 봇의 Microsoft 앱 ID와 동일한 값을 갖는 "appid" 클레임을 포함합니다.
6. 토큰이 유효 기간 범위 안에 있습니다. 업계 표준 클록 오차는 5분입니다.
7. 토큰이 [3단계](#emulator-to-bot-step-3)에서 검색한 OpenID 키 문서에 나열된 키와 유효한 암호화 키를 갖습니다.

> [!NOTE]
> 요구 사항 5는 에뮬레이터 인증 경로에 특정됩니다. 

토큰이 이 요구 사항에 일부라도 부합하지 않으면 봇은 **HTTP 403(금지 됨)** 상태 코드를 반환하여 요청을 종료해야 합니다.

> [!IMPORTANT]
> 이 요구 사항은 모두 중요하며 특히 4와 7번 요구 사항이 중요합니다. 이러한 인증 요구 사항을 하나라도 구현하지 못하면 봇이 JWT 토큰을 공개할 수 있는 공격에 노출됩니다.

#### <a name="emulator-to-bot-example-jwt-components"></a>봇에 대한 에뮬레이터: JWT 구성 요소 예제

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOUR MICROSOFT APP ID>",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> 현실에서는 실제 필드가 다를 수 있습니다. 위에서 지정한 대로 모든 JWT 토큰을 만들고 그 유효성을 검사합니다.

## <a name="security-protocol-changes"></a>보안 프로토콜 변경 내용

> [!WARNING]
> 보안 프로토콜 v3.0 지원이 **2017년 7월 31일** 중단되었습니다. 자체 인증 코드를 작성한 경우(즉 Bot Builder SDK를 사용하여 봇을 만들지 않음) 아래 목록의 v3.1 값을 사용하도록 응용 프로그램을 업데이트하여 보안 프로토콜 v3.1로 업그레이드해야 합니다. 

### <a name="bot-to-connector-authenticationbot-to-connector"></a>[커넥터에 대한 봇 인증](#bot-to-connector)

#### <a name="oauth-login-url"></a>OAuth 로그인 URL

| 프로토콜 버전 | 유효한 값 |
|----|----|
| v3.1 & v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>OAuth 범위

| 프로토콜 버전 | 유효한 값 |
|----|----|
| v3.1 & v3.2 |  `https://api.botframework.com/.default` |

### <a name="connector-to-bot-authenticationconnector-to-bot"></a>[봇에 대한 커넥터 인증](#connector-to-bot)

#### <a name="openid-metadata-document"></a>OpenID 메타데이터 문서

| 프로토콜 버전 | 유효한 값 |
|----|----|
| v3.1 & v3.2 | `https://login.botframework.com/v1/.well-known/openidconfiguration` |

#### <a name="jwt-issuer"></a>JWT 발급자

| 프로토콜 버전 | 유효한 값 |
|----|----|
| v3.1 & v3.2 | `https://api.botframework.com` |

### <a name="emulator-to-bot-authenticationemulator-to-bot"></a>[봇에 대한 에뮬레이터 인증](#emulator-to-bot)

#### <a name="oauth-login-url"></a>OAuth 로그인 URL

| 프로토콜 버전 | 유효한 값 |
|----|----|
| v3.1 & v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>OAuth 범위

| 프로토콜 버전 | 유효한 값 |
|----|----|
| v3.1 & v3.2 |  봇의 Microsoft 앱 ID + `/.default` |

#### <a name="jwt-audience"></a>JWT 대상

| 프로토콜 버전 | 유효한 값 |
|----|----|
| v3.1 & v3.2 | 봇의 Microsoft 앱 ID  |

#### <a name="jwt-issuer"></a>JWT 발급자

| 프로토콜 버전 | 유효한 값 |
|----|----|
| v3.1 | `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` |
| v3.2 | `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/` |

#### <a name="openid-metadata-document"></a>OpenID 메타데이터 문서

| 프로토콜 버전 | 유효한 값 |
|----|----|
| v3.1 & v3.2 | `https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration` |

## <a name="additional-resources"></a>추가 리소스

- [Bot Framework 인증 문제 해결](../bot-service-troubleshoot-authentication-problems.md)
- [JWT(JSON Web Token) draft-jones-json-web-token-07](http://openid.net/specs/draft-jones-json-web-token-07.html)
- [JWS(JSON Web Signature) draft-jones-json-web-signature-04](https://tools.ietf.org/html/draft-jones-json-web-signature-04)
- [JWK(JSON Web Key) RFC 7517](https://tools.ietf.org/html/rfc7517)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
