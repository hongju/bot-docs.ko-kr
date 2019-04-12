---
title: Azure Bot Service를 통해 봇에 인증 추가 | Microsoft Docs
description: Azure Bot Service 인증 기능을 사용하여 봇에 SSO를 추가하는 방법을 알아봅니다.
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 04/09/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 27c97d257261a6f3b9d867503aee40382b685e20
ms.sourcegitcommit: 562dd44e38abacaa31427da5675da556a970cf11
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/10/2019
ms.locfileid: "59477106"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Azure Bot Service를 통해 봇에 인증 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Azure Bot Service 및 v4 SDK는 새로운 봇 인증 기능을 포함하고 있으며 사용자를 Azure AD(Azure Active Directory), GitHub, Uber 등의 다양한 ID 공급자에 인증하는 봇을 쉽게 개발할 수 있는 기능을 제공합니다. 이러한 기능 덕분에 일부 클라이언트의 _매직 코드를 확인_할 필요가 없으므로 사용자 환경이 개선됩니다.

이전에 봇은 OAuth 컨트롤러 및 로그인 링크를 포함하고, 대상 클라이언트 ID 및 비밀을 저장하고, 사용자 토큰 관리를 수행해야 했습니다. 봇은 사용자에게 웹 사이트에 로그인할 것을 요청하는데, 이때 사용자가 ID 확인에 사용할 수 있는 _매직 코드_가 생성됩니다.

이제 봇 개발자는 더 이상 OAuth 컨트롤러를 호스트하거나 토큰 수명 주기를 관리할 필요가 없습니다. 이는 모두 Azure Bot Service에서 수행될 수 있습니다.

기능은 다음과 같습니다.

- 6자리 매직 코드 확인에 대한 필요를 제거하는 새 WebChat 및 DirectLineJS 라이브러리와 같은 새로운 인증 기능을 지원하는 채널에 대한 향상된 기능
- 다양한 OAuth ID 공급자에 대한 연결 설정을 추가, 삭제 및 구성하는 Azure Portal에 대한 향상된 기능
- Azure AD(v1 및 v2 엔드포인트 모두), GitHub 및 기타를 포함하는 다양한 기본 제공 ID 공급자에 대한 지원
- 토큰을 검색하고, OAuthCards를 만들고 TokenResponse 이벤트를 처리할 수 있도록 C# 및 Node.js Bot Framework SDK에 대한 업데이트
- Azure AD에 인증하는 봇을 만드는 방법에 대한 샘플입니다.

기존 봇에 이러한 기능을 추가하기 위해 이 문서의 단계에서 추정할 수 있습니다. 다음 샘플 봇은 새로운 인증 기능을 보여줍니다.

> [!NOTE]
> 인증 기능은 BotBuilder v3에서도 작동합니다. 그러나 이 문서에서는 v4 샘플 코드만 다룹니다.

### <a name="about-this-sample"></a>이 샘플 정보

Azure 봇 리소스를 만들고, 봇이 Office 365에 액세스하도록 허용하는 새 Azure AD(v1 또는 v2) 애플리케이션을 만들어야 합니다. 봇 리소스는 봇의 자격 증명을 등록합니다. 인증 기능을 테스트하려면 이 자격 증명이 필요하며, 심지어 봇 코드를 로컬로 실행하는 경우에도 마찬가지입니다.

> [!IMPORTANT]
> Azure에서 봇을 등록할 때마다 봇에 Azure AD 앱이 할당됩니다. 그러나 이 앱은 채널-봇 액세스를 보호합니다.
애플리케이션마다 봇이 사용자 대신 인증할 수 있는 추가 AAD 앱이 필요합니다.

이 문서에서는 Azure AD v1 또는 v2 토큰을 사용하여 Microsoft Graph에 연결하는 샘플 봇에 대해 설명합니다. 연결된 Azure AD 앱을 만들고 등록하는 방법도 알아봅니다. 이 프로세스에서 [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub 리포지토리의 코드를 사용할 것입니다. 이 문서에서는 다음과 같은 프로세스를 다룹니다.

- **봇 리소스 만들기**
- **Azure AD 애플리케이션 만들기**
- **봇에 Azure AD 애플리케이션 등록**
- **봇 샘플 코드 준비**

작업을 완료하면 로컬로 실행되면서 이메일 확인 및 전송, 사용자 및 관리자 표시 등 Azure AD 애플리케이션에 대한 간단한 작업에 응답할 수 있는 봇이 하나 생깁니다. 이렇게 하기 위해 봇에서 Microsoft.Graph 라이브러리에 대해 Azure AD 애플리케이션의 토큰을 사용합니다. OAuth 로그인 기능을 테스트하기 위해 봇을 게시할 필요는 없지만, 봇에 유효한 Azure 앱 ID 및 암호가 필요합니다.

이러한 인증 기능은 다른 유형의 봇에서도 작동합니다. 하지만 이 문서에서는 등록 전용 봇을 사용합니다.

### <a name="web-chat-and-direct-line-considerations"></a>웹 채팅 및 Direct Line 고려 사항

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

웹 채팅에 Azure Bot Service 인증을 사용할 때 고려해야 하는 몇 가지 중요한 보안 문제가 있습니다.

1. 봇이 공격자를 다른 사람으로 착각하게 만드는 가장을 차단해야 합니다. 웹 채팅에서 공격자는 웹 채팅 인스턴스의 사용자 ID를 변경하여 다른 사람으로 가장할 수 있습니다.

    이를 방지하려면 사용자 ID를 추측할 수 없게 만들어야 합니다. Direct Line 채널에서 향상된 인증 옵션을 사용하도록 설정하면 Azure Bot Service가 사용자 ID 변경을 탐지하고 거부할 수 있습니다. Direct Line에서 봇으로 전송되는 메시지의 사용자 ID는 사용자가 웹 채팅을 초기화할 때 사용한 것과 항상 동일합니다. 이 기능을 사용하려면 사용자 ID가 `dl_`로 시작해야 합니다.

1. 올바른 사용자로 로그인해야 합니다. 사용자에게는 두 개의 ID가 있는데, 하나는 채널 ID이고 다른 하나는 ID 공급자와 작업할 때 사용하는 ID입니다. 웹 채팅에서, Azure Bot Service는 로그인 프로세스가 웹 채팅 자체와 동일한 브라우저 세션에서 완료되도록 보장할 수 있습니다.

    이 보호를 사용하려면 봇의 웹 채팅 클라이언트를 호스팅할 수 있는 트러스트된 도메인 목록을 포함하고 있는 Direct Line 토큰을 사용하여 웹 채팅을 시작합니다. 그런 다음, Direct Line 구성 페이지에서 트러스트된 도메인(원본) 목록을 정적으로 지정합니다.

Direct Line의 `/v3/directline/tokens/generate` REST 엔드포인트를 사용하여 대화용 토큰을 생성하고, 요청 페이로드의 사용자 ID를 지정합니다. 코드 샘플은 [향상된 Direct Line 인증 기능](https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/) 블로그 게시물을 참조하세요.

<!-- The eventual article about this should talk about the tokens/generate endpoint and its parameters: user, trustedOrigins, and [maybe] eTag.
Sample payload
{
  "user": {
    "id": "string",
    "name": "string",
    "aadObjectId": "string",
    "role": "string"
  },
  "trustedOrigins": [
    "string"
  ],
  "eTag": "string"
}
 -->

## <a name="prerequisites"></a>필수 조건

- [봇 기본 사항][concept-basics] 및 [상태 관리][concept-state]에 대한 지식
- Azure 및 OAuth 2.0 개발에 대한 지식
- Visual Studio 2017 이상, Node.js, npm 및 git
- 다음 샘플 중 하나

| 샘플 | BotBuilder 버전 | 데모 |
|:---|:---:|:---|
| [**CSharp**][cs-auth-sample] 또는 [**JavaScript**][js-auth-sample]로 **봇 인증** | v4 | OAuthCard 지원 |
| [**CSharp**][cs-msgraph-sample] 또는 [**JavaScript**][js-msgraph-sample]로 작성된 **봇 인증 MSGraph** | v4 |  Microsoft Graph API 지원과 OAuth 2 |

## <a name="create-your-bot-resource-on-azure"></a>Azure에서 봇 리소스 만들기

[Azure Portal](https://portal.azure.com/)을 사용하여 **봇 채널 등록**을 만듭니다.

## <a name="create-and-register-an-azure-ad-application"></a>Azure AD 애플리케이션 만들기 및 등록

봇이 Microsoft Graph API에 연결하는 데 사용할 수 있는 Azure AD 애플리케이션이 필요합니다.

이 봇의 경우 Azure AD v1 또는 v2 엔드포인트를 사용할 수 있습니다.
v1 및 v2 엔드포인트 간의 차이점에 대한 정보는 [v1-v2 비교](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare) 및 [Azure AD v2.0 엔드포인트 개요](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)를 참조하세요.

### <a name="create-your-azure-ad-application"></a>Azure AD 애플리케이션 만들기

다음 단계에 따라 새 Azure AD 애플리케이션을 만듭니다. 만드는 앱에 v1 또는 v2 엔드포인트를 사용할 수 있습니다.

> [!TIP]
> Azure AD 애플리케이션을 만든 후 관리자 권한을 갖고 있는 테넌트에 등록해야 합니다.

1. Azure Portal에서 [Azure Active Directory][azure-aad-blade] 채널을 엽니다.
    올바른 테넌트에 있지 않은 경우 **디렉터리 전환**을 클릭하여 올바른 테넌트로 전환합니다. (테넌트를 만드는 방법에 대한 지침은 [포털에 액세스 및 테넌트 만들기](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)를 참조하세요.)
1. **앱 등록** 패널을 엽니다.
1. **앱 등록** 패널에서 **새 애플리케이션 등록**을 클릭합니다.
1. 필수 필드를 입력하고 앱 등록을 만듭니다.

   1. 애플리케이션의 이름을 지정합니다.
   1. **애플리케이션 유형**을 **웹앱/API**로 설정합니다.
   1. **로그온 URL**을 `https://token.botframework.com/.auth/web/redirect`로 설정합니다.
   1. **만들기**를 클릭합니다.

      - 만들어지면 **등록된 앱** 창에 표시됩니다.
      - **애플리케이션 ID** 값을 기록합니다. 나중에 Azure AD 애플리케이션을 봇에 등록할 때 이 값을 _클라이언트 ID_로 사용할 것입니다.

1. **설정**을 클릭하여 애플리케이션을 구성합니다.
1. **키**를 클릭하여 **키** 패널을 엽니다.

   1. **암호** 아래에서 `BotLogin` 키를 만듭니다.
   1. 해당 **기간**을 **무기한**으로 설정합니다.
   1. **저장**을 클릭하고 키 값을 기록합니다. 나중에 Azure AD 애플리케이션을 봇에 등록할 때 이 값을 _클라이언트 암호_로 사용할 것입니다.
   1. **키** 패널을 닫습니다.

1. **필요한 사용 권한**을 클릭하여 **필요한 사용 권한** 패널을 엽니다.

   1. **추가**를 클릭합니다.
   1. **API 선택**을 클릭한 다음, **Microsoft Graph**를 선택하고 **선택**을 클릭합니다.
   1. **사용 권한 선택**을 클릭합니다. 애플리케이션에서 사용할 위임된 권한을 선택합니다.

      > [!NOTE]
      > **관리자 필요**로 표시된 모든 사용 권한에는 사용자 및 테넌트 관리자가 모두 로그인해야 합니다. 따라서 봇은 이를 사용하지 않는 경향이 있습니다.

      다음 Microsoft Graph 위임된 사용 권한을 선택합니다.
      - 모든 사용자의 기본 프로필 읽기
      - 사용자 메일 읽기
      - 사용자로 메일 보내기
      - 로그인 및 사용자 프로필 읽기
      - 사용자의 기본 프로필 보기기
      - 사용자의 이메일 주소 보기

   1. **선택**을 클릭한 다음, **완료**를 클릭합니다.
   1. **필요한 사용 권한** 패널을 닫습니다.

이제 Azure AD v1 애플리케이션이 구성되었습니다.

### <a name="register-your-azure-ad-application-with-your-bot"></a>봇에 Azure AD 애플리케이션 등록

다음 단계는 앞에서 만든 Azure AD 애플리케이션을 봇에 등록하는 것입니다.

# [<a name="azure-ad-v1"></a>Azure AD v1](#tab/aadv1)

1. [Azure Portal](http://portal.azure.com/)에서 봇의 리소스 페이지로 이동합니다.
1. **설정**을 클릭합니다.
1. 페이지 아래쪽의 **OAuth 연결 설정** 아래에서 **설정 추가**를 클릭합니다.
1. 다음과 같이 양식을 채웁니다.

    1. **이름**에 대해 연결의 이름을 입력합니다. 이 이름을 봇 코드에 사용할 것입니다.
    1. **서비스 공급자**에 대해 **Azure Active Directory**를 선택합니다. 이를 선택하면 Azure AD 관련 필드가 표시됩니다.
    1. **클라이언트 ID**에 대해 Azure AD v1 애플리케이션에 대해 기록한 애플리케이션 ID를 입력합니다.
    1. **클라이언트 비밀**에 대해 애플리케이션의 `BotLogin` 키에 대해 기록한 키를 입력합니다.
    1. **권한 부여 유형**에 대해 `authorization_code`를 입력합니다.
    1. **로그인 URL**에 대해 `https://login.microsoftonline.com`을 입력합니다.
    1. **테넌트 ID**에 대해 Azure Active Directory에 대한 테넌트 ID를 입력합니다(예: `microsoft.com` 또는 `common`).

       인증될 수 있는 사용자와 연결된 테넌트가 됩니다. 누구든지 봇을 통해 자신을 인증하도록 허용하려면 `common` 테넌트를 사용합니다.

    1. **리소스 URL**에 `https://graph.microsoft.com/`을 입력합니다.
    1. **범위**를 비워 둡니다.

1. **저장**을 클릭합니다.

> [!NOTE]
> 이러한 값을 사용하면 애플리케이션은 Microsoft Graph API를 통해 Office 365 데이터에 액세스할 수 있습니다.

# [<a name="azure-ad-v2"></a>Azure AD v2](#tab/aadv2)

1. [Azure Portal](http://portal.azure.com/)에서 봇의 봇 채널 등록 페이지로 이동합니다.
1. **설정**을 클릭합니다.
1. 페이지 아래쪽의 **OAuth 연결 설정** 아래에서 **설정 추가**를 클릭합니다.
1. 다음과 같이 양식을 채웁니다.

    1. **이름**에 대해 연결의 이름을 입력합니다. 봇 코드에서 사용합니다.
    1. **서비스 공급자**에 대해 **Azure Active Directory v2**를 선택합니다. 이를 선택하면 Azure AD 관련 필드가 표시됩니다.
    1. **클라이언트 ID**에 대해 애플리케이션 등록의 Azure AD v2 애플리케이션 ID를 입력합니다.
    1. **클라이언트 비밀**에 대해 애플리케이션 등록의 Azure AD v2 애플리케이션 암호를 입력합니다.
    1. **테넌트 ID**에 대해 Azure Active Directory에 대한 테넌트 ID를 입력합니다(예: `microsoft.com` 또는 `common`).

        인증될 수 있는 사용자와 연결된 테넌트가 됩니다. 누구든지 봇을 통해 자신을 인증하도록 허용하려면 `common` 테넌트를 사용합니다.

    1. **범위**에 대해 애플리케이션 등록에서 선택한 사용 권한의 이름을 입력합니다. `Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`

        > [!NOTE]
        > Azure AD v2의 경우 **범위**는 대/소문자 구분, 공백으로 구분된 값의 목록을 사용합니다.

1. **저장**을 클릭합니다.

> [!NOTE]
> 이러한 값을 사용하면 애플리케이션은 Microsoft Graph API를 통해 Office 365 데이터에 액세스할 수 있습니다.

---

### <a name="test-your-connection"></a>연결 테스트

1. 연결 항목을 클릭하여 방금 만든 연결을 엽니다.
1. **서비스 공급자 연결 설정** 창의 맨 위에 있는 **연결 테스트**를 클릭합니다.
1. 처음에 앱에서 요청하는 사용 권한을 나열하는 새 브라우저 탭을 열고 수락하라는 메시지를 표시해야 합니다.
1. **Accept**를 클릭합니다.
1. 그러면 **\<your-connection-name>에 대한 연결 테스트가 성공했습니다** 페이지로 리디렉션됩니다.

이제 봇 코드에서 이 연결 이름을 사용하여 사용자 토큰을 검색할 수 있습니다.

## <a name="prepare-the-bot-sample-code"></a>봇 샘플 코드 준비

선택한 샘플에 따라 C# 또는 Node로 작업하게 됩니다.

| 샘플 | BotBuilder 버전 | 데모 |
|:---|:---:|:---|
| [**CSharp**][cs-auth-sample] 또는 [**JavaScript**][js-auth-sample]로 **봇 인증** | v4 | OAuthCard 지원 |
| [**CSharp**][cs-msgraph-sample] 또는 [**JavaScript**][js-msgraph-sample]로 작성된 **봇 인증 MSGraph** | v4 |  Microsoft Graph API 지원과 OAuth 2 |

1. 위의 샘플 링크 중 하나를 클릭하고 github 리포지토리를 복제합니다.
1. GitHub readme 페이지에서 해당 봇(C# 또는 Node)을 실행하는 방법에 대한 지침을 따르세요.
1. C# 봇 인증 샘플을 사용 중인 경우

    1. `AuthenticationBot.cs` 파일의 `ConnectionName` 변수를 봇의 OAuth 2.0 연결 설정을 구성할 때 사용한 값으로 설정합니다.
    1. `BotConfiguration.bot` 파일의 `appId` 값을 봇의 앱 ID로 설정합니다.
    1. `BotConfiguration.bot` 파일의 `appPassword` 값을 봇의 앱 비밀로 설정합니다.

1. Node/JS 봇 인증 샘플을 사용 중인 경우

    1. `bot.js` 파일의 `CONNECTION_NAME` 변수를 봇의 OAuth 2.0 연결 설정을 구성할 때 사용한 값으로 설정합니다.
    1. `bot-authentication.bot` 파일의 `appId` 값을 봇의 앱 ID로 설정합니다.
    1. `bot-authentication.bot` 파일의 `appPassword` 값을 봇의 앱 비밀로 설정합니다.

    > [!IMPORTANT]
    > 비밀의 문자에 따라 암호를 XML 이스케이프해야 할 수 있습니다. 예를 들어 모든 앰퍼샌드(&)는 `&amp;`로 인코딩되어야 합니다.

    ```json
    {
        "name": "BotAuthentication",
        "secretKey": "",
        "services": [
            {
            "appId": "",
            "id": "http://localhost:3978/api/messages",
            "type": "endpoint",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages",
            "name": "BotAuthentication"
            }
        ]
    }
    ```

    **Microsoft 앱 ID** 및 **Microsoft 앱 암호** 값을 가져오는 방법을 알지 못하는 경우 Azure Portal에서 봇에 대해 프로비전된 Azure 앱 서비스의 **ApplicationSettings**를 확인합니다.

    > [!NOTE]
    > 이제 이 봇 코드를 Azure 구독에 다시 게시할 수 있지만(프로젝트를 마우스 오른쪽 단추로 클릭하고 **게시** 선택) 이 자습서에서는 필요하지 않습니다. Azure Portal에서 봇을 구성할 때 사용한 애플리케이션 및 호스팅 계획을 사용하는 게시 구성을 설정해야 합니다.

## <a name="use-the-emulator-to-test-your-bot"></a>에뮬레이터를 사용하여 봇 테스트

봇을 로컬로 테스트하려면 [봇 에뮬레이터](https://github.com/Microsoft/BotFramework-Emulator)를 설치해야 합니다. v3 또는 v4 에뮬레이터를 사용할 수 있습니다.

1. 봇을 시작합니다(디버깅을 사용하거나 사용하지 않고).
1. 페이지에 대한 localhost 포트 번호를 적어 둡니다. 이 정보는 봇과 상호 작용하기 위해 필요합니다.
1. 에뮬레이터를 시작합니다.
1. 봇에 연결합니다. 인증을 사용할 때 봇 구성에서 **Microsoft 앱 ID** 및 **Microsoft 앱 암호**를 사용해야 합니다.
1. 토큰을 사용할 수 있게 되면 Azure Bot Service가 에뮬레이터에 토큰을 반환하도록, 에뮬레이터 설정에서 **OAuthCards에 로그인 확인 코드 사용**을 선택하고 **ngrok**를 사용하도록 설정합니다.

   연결을 구성하지 않은 경우 주소 및 봇의 Microsoft 앱 ID와 암호를 제공합니다. 봇의 URL에 `/api/messages`를 추가합니다. URL은 `http://localhost:portNumber/api/messages`와 비슷합니다.

1. `help`를 입력하여 봇에 사용 가능한 명령 목록을 보고 인증 기능을 테스트합니다.
1. 로그인하면 로그아웃할 때까지 자격 증명을 다시 제공할 필요가 없습니다.
1. 로그아웃하고, 인증을 취소하려면 `signout`을 입력합니다.

<!--To restart completely from scratch you also need to:
1. Navigate to the **AppData** folder for your account.
1. Go to the **Roaming/botframework-emulator** subfolder.
1. Delete the **Cookies** and **Coolies-journal** files.
-->

> [!NOTE]
> 봇 인증은 Bot Connector Service를 사용해야 합니다. 서비스는 봇에 대한 봇 채널 등록 정보에 액세스합니다. 포털에서 봇의 메시징 엔드포인트를 설정해야 하는 이유입니다. 인증은 또한 HTTPS를 사용해야 합니다. 로컬로 실행하는 봇에 대해 HTTPS 전달 주소를 만들어야 했던 이유입니다.

<!--The following is necessary for WebChat:
1. Use the **ngrok** command-line tool to get a forwarding HTTPS address for your bot.
   - For information on how to do this, see [Debug any Channel locally using ngrok](https://blog.botframework.com/2017/10/19/debug-channel-locally-using-ngrok/).
   - Any time you exit **ngrok**, you will need to redo this and the following step before starting the Emulator.
1. On the Azure Portal, go to the **Settings** blade for your bot.
   1. In the **Configuration** section, change the **Messaging endpoint** to the HTTPS forwarding address generated by **ngrok**.
   1. Click **Save** to save your change.
-->

## <a name="notes-on-the-token-retrieval-flow"></a>토큰 검색 흐름에 대한 참고 사항

사용자가 봇에 사용자 로그인이 필요한 작업 수행을 요청하는 경우 봇은 `OAuthPrompt`를 사용하여 지정된 연결에 대한 토큰 검색을 시작할 수 있습니다. `OAuthPrompt`는 다음과 같이 진행되는 토큰 검색 흐름을 만듭니다.

1. Azure Bot Service에 현재 사용자 및 연결에 대한 토큰이 이미 있는지 확인합니다. 토큰이 있을 경우 토큰이 반환됩니다.
1. Azure Bot Service에 캐시된 토큰이 없을 경우 사용자가 클릭할 수 있는 로그인 단추인 `OAuthCard`가 생성됩니다.
1. 사용자가 `OAuthCard` 로그인 단추를 클릭하면 Azure Bot Service가 사용자의 토큰을 직접 전송하거나 사용자가 6자리 인증 코드를 제공하여 채팅 창에 입력하도록 합니다.
1. 사용자에게 인증 코드가 제공되면 봇이 이 인증 코드를 사용자의 토큰으로 교환합니다.

다음의 두 코드 조각은 `OAuthPrompt`에서 가져온 것으로, 프롬프트에서 이러한 단계가 작동하는 방식을 보여줍니다.

### <a name="check-for-a-cached-token"></a>캐시된 토큰에 대한 확인

이 코드에서 봇은 먼저 Azure Bot Service에 사용자에 대한 토큰(현재 작업 전송자에 의해 식별됨) 및 지정된 ConnectionName(구성에서 사용되는 연결 이름)이 이미 있는지 빠른 검사를 수행합니다. Azure Bot Service는 캐시된 토큰이 이미 있거나 없습니다. GetUserTokenAsync에 대한 호출은 이 빠른 검사를 수행합니다. Azure Bot Service에 토큰이 있고 반환하는 경우 토큰을 즉시 사용할 수 있습니다. Azure Bot Service에 토큰이 없는 경우 이 메서드는 Null을 반환합니다. 이 경우 봇은 로그인할 사용자에 대한 사용자 지정된 OAuthCard를 보낼 수 있습니다.

# [<a name="c"></a>C#](#tab/csharp)

```csharp
// First ask Bot Service if it already has a token for this user
var token = await adapter.GetUserTokenAsync(turnContext, connectionName, null, cancellationToken).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
}
```

# [<a name="javascript"></a>JavaScript](#tab/javascript)

```javascript
public async getUserToken(context: TurnContext, code?: string): Promise<TokenResponse|undefined> {
    // Get the token and call validator
    const adapter: any = context.adapter as any; // cast to BotFrameworkAdapter
    return await adapter.getUserToken(context, this.settings.connectionName, code);
}
```

---

### <a name="send-an-oauthcard-to-the-user"></a>사용자에게 OAuthCard 보내기

원하는 모든 텍스트 및 단추 텍스트를 사용하여 OAuthCard를 사용자 지정할 수 있습니다. 중요한 부분은 다음과 같습니다.

- `ContentType`을 `OAuthCard.ContentType`으로 설정합니다.
- `ConnectionName` 속성을 사용하려는 연결의 이름으로 설정합니다.
- `Type` `ActionTypes.Signin`의 `CardAction`이 있는 단추 하나를 포함합니다. 로그인 링크에 대한 값을 지정할 필요가 없습니다.

이 호출의 끝에서 봇은 "토큰이 다시 돌아오기를 기다려야" 합니다. 로그인을 위해 수행해야 하는 많은 사용자가 있을 수 있으므로 이 대기는 주 작업 스트림에서 수행됩니다.

# [<a name="c"></a>C#](#tab/csharp)

```csharp
private async Task SendOAuthCardAsync(ITurnContext turnContext, IMessageActivity message, CancellationToken cancellationToken = default(CancellationToken))
{
    if (message.Attachments == null)
    {
        message.Attachments = new List<Attachment>();
    }

    message.Attachments.Add(new Attachment
    {
        ContentType = OAuthCard.ContentType,
        Content = new OAuthCard
        {
            Text = "Please sign in",
            ConnectionName = connectionName,
            Buttons = new[]
            {
                new CardAction
                {
                    Title = "Sign In",
                    Text = "Sign In",
                    Type = ActionTypes.Signin,
                },
            },
        },
    });

    await turnContext.SendActivityAsync(message, cancellationToken).ConfigureAwait(false);
}
```

# [<a name="javascript"></a>JavaScript](#tab/javascript)

```javascript
private async sendOAuthCardAsync(context: TurnContext, prompt?: string|Partial<Activity>): Promise<void> {
    // Initialize outgoing message
    const msg: Partial<Activity> =
        typeof prompt === 'object' ? {...prompt} : MessageFactory.text(prompt, undefined, InputHints.ExpectingInput);
    if (!Array.isArray(msg.attachments)) { msg.attachments = []; }

    const cards: Attachment[] = msg.attachments.filter((a: Attachment) => a.contentType === CardFactory.contentTypes.oauthCard);
    if (cards.length === 0) {
        // Append oauth card
        msg.attachments.push(CardFactory.oauthCard(
            this.settings.connectionName,
            this.settings.title,
            this.settings.text
        ));
    }

    // Send prompt
    await context.sendActivity(msg);
}
```

---

### <a name="wait-for-a-tokenresponseevent"></a>TokenResponseEvent에 대한 대기

이 코드에서 봇은 `TokenResponseEvent`에 대해 대기합니다(대화 상자 스택에 라우팅되는 방법에 대한 정보는 아래와 같음). `WaitForToken` 메서드는 먼저 이 이벤트가 전송되었는지 확인합니다. 전송된 경우 봇에서 사용할 수 있습니다. 그렇지 않은 경우 `RecognizeTokenAsync` 메서드는 봇에 전송된 모든 텍스트를 사용하고 `GetUserTokenAsync`에 전달합니다. 이에 대한 이유는 일부 클라이언트(예: WebChat)에서 매직 코드 확인 코드가 필요하지 않고 `TokenResponseEvent`에서 토큰을 직접 보낼 수 있기 때문입니다. 다른 클라이언트는 여전히 매직 코드가 필요합니다(예: Facebook 또는 Slack). Azure Bot Service는 이러한 클라이언트에게 6자리 매직 코드를 제공하고 사용자에게 채팅 창에 이를 입력하도록 요청합니다. 적합하지 않은 경우 이는 '대체' 동작이므로 `RecognizeTokenAsync`가 코드를 받는 경우 봇은 이 코드를 Azure Bot Service에 보내고 토큰을 다시 가져올 수 있습니다. 이 호출 또한 실패하는 경우 오류를 보고하거나 다른 작업을 수행할 수 있습니다. 그러나 대부분의 경우에서 봇은 이제 사용자 토큰을 갖습니다.

각 샘플의 봇 코드를 살펴보면 `Event` 및 `Invoke` 활동도 대화 상자 스택으로 라우팅되는 것을 알 수 있습니다.

# [<a name="c"></a>C#](#tab/csharp)

```csharp
// This can be called when the bot receives an Activity after sending an OAuthCard
private async Task<TokenResponse> RecognizeTokenAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (IsTokenResponseEvent(turnContext))
    {
        // The bot received the token directly
        var tokenResponseObject = turnContext.Activity.Value as JObject;
        var token = tokenResponseObject?.ToObject<TokenResponse>();
        return token;
    }
    else if (IsTeamsVerificationInvoke(turnContext))
    {
        var magicCodeObject = turnContext.Activity.Value as JObject;
        var magicCode = magicCodeObject.GetValue("state")?.ToString();

        var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, magicCode, cancellationToken).ConfigureAwait(false);
        return token;
    }
    else if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // make sure it's a 6-digit code
        var matched = _magicCodeRegex.Match(turnContext.Activity.Text);
        if (matched.Success)
        {
            var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, matched.Value, cancellationToken).ConfigureAwait(false);
            return token;
        }
    }

    return null;
}

private bool IsTokenResponseEvent(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Event && activity.Name == "tokens/response";
}

private bool IsTeamsVerificationInvoke(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Invoke && activity.Name == "signin/verifyState";
}
```

# [<a name="javascript"></a>JavaScript](#tab/javascript)

```javascript
private async recognizeToken(context: TurnContext): Promise<PromptRecognizerResult<TokenResponse>> {
    let token: TokenResponse|undefined;
    if (this.isTokenResponseEvent(context)) {
        token = context.activity.value as TokenResponse;
    } else if (this.isTeamsVerificationInvoke(context)) {
        const code: any = context.activity.value.state;
        await context.sendActivity({ type: 'invokeResponse', value: { status: 200 }});
        token = await this.getUserToken(context, code);
    } else if (context.activity.type === ActivityTypes.Message) {
        const matched: RegExpExecArray = /(\d{6})/.exec(context.activity.text);
        if (matched && matched.length > 1) {
            token = await this.getUserToken(context, matched[1]);
        }
    }

    return token !== undefined ? { succeeded: true, value: token } : { succeeded: false };
}

private isTokenResponseEvent(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Event && activity.name === 'tokens/response';
}

private isTeamsVerificationInvoke(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Invoke && activity.name === 'signin/verifyState';
}
```

---

### <a name="message-controller"></a>메시지 컨트롤러

봇에 대한 후속 호출에서 토큰은 이 샘플 봇에서 캐시되지 않습니다. 봇은 Azure Bot Service에 토큰을 항상 요청할 수 있기 때문입니다. 이렇게 하면 봇은 토큰 수명 주기를 관리하고, 토큰을 새로 고치는 등의 작업을 수행할 필요가 없습니다. Azure Bot Service에서 이 모든 작업을 수행합니다.

### <a name="further-reading"></a>추가 참고 자료

- [Bot Framework 추가 리소스](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)에는 추가 지원의 링크가 포함되어 있습니다.
- [Bot Framework SDK](https://github.com/microsoft/botbuilder) 리포지토리에는 Bot Builder SDK와 관련된 리포지토리, 샘플, 도구 및 사양 정보가 들어 있습니다.

<!-- Footnote-style links -->

[Azure portal]: https://ms.portal.azure.com
[azure-aad-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview
[aad-registration-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-auth-sample]: https://aka.ms/v4cs-bot-auth-sample
[js-auth-sample]: https://aka.ms/v4js-bot-auth-sample
[cs-msgraph-sample]: https://aka.ms/v4cs-auth-msgraph-sample
[js-msgraph-sample]: https://aka.ms/v4js-auth-msgraph-sample
