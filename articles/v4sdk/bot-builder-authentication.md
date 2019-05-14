---
title: Azure Bot Service를 통해 봇에 인증 추가 | Microsoft Docs
description: Azure Bot Service 인증 기능을 사용하여 봇에 SSO를 추가하는 방법을 알아봅니다.
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 04/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4fd61d5d68b5b7b3a535afdd47d635eef3820622
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033638"
---
<!-- Related TODO:
- Check code in [Web Chat channel](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0)
- Check guidance in [DirectLine authentication](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0)
-->

<!-- General TODO: (Feedback from CSE (Nafis))
- Add note that: token management is based on user ID
- Explain why/how to share existing website authentication with a bot.
- Risk: Even people who use a DirectLine token can be vulnerable to user ID impersonation.
    Docs/samples that show exchange of secret for token don't specify a user ID, so an attacker can impersonate a different user by modifying the ID client side. There's a [blog post](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fblog.botframework.com%2F2018%2F09%2F01%2Fusing-webchat-with-azure-bot-services-authentication%2F&data=02%7C01%7Cv-jofing%40microsoft.com%7Cee005e1c9d2c4f4e7ea508d6b231b422%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C636892323874079713&sdata=b0DWMxHzmwQvg5EJtlqKFDzR7fYKmg10fXma%2B8zGqEI%3D&reserved=0) that shows how to do this properly.
"Major issues":
- This doc is a sample walkthrough, but there's no deeper documentation explaining how the Azure Bot Service is handling tokens. How does the OAuth flow work? Where is it storing my users' access tokens? What's the security and best practices around using it?

"Minor issues":
- AAD v2 steps tell you to add delegated permission scopes during registration, but this shouldn't be necessary in AAD v2 due to dynamic scopes. (Ming, "This is currently necessary because scopes are not exposed through our runtime API. We don’t currently have a way for the developer to specify which scope he wants at runtime.")

- "The scope of the connection setting needs to have both openid and a resource in the Azure AD graph, such as Mail.Read." Unclear if I need to take some action at this point to make happen. Kind of out of context. I'm registering an AAD application in the portal, there's no connection setting
- Does the bot need all of these scopes for the samples? (e.g. "Read all users' basic profiles")
-->

# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Azure Bot Service를 통해 봇에 인증 추가

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

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

## <a name="prerequisites"></a>필수 조건

- [봇 기본 사항][concept-basics], [상태 관리][concept-state], [대화 상자 라이브러리][concept-dialogs], [순차적 대화 흐름을 구현하는 방법][simple-dialog], [대화 상자 프롬프트를 사용하여 사용자 입력을 수집하는 방법][dialog-prompts] 및 [대화 상자 재사용 방법][component-dialogs]에 대한 지식
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

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD v1](#tab/aadv1)

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

# <a name="azure-ad-v2tabaadv2"></a>[Azure AD v2](#tab/aadv2)

1. [Microsoft 애플리케이션 등록 포털](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)로 이동합니다.
1. **앱 추가**를 클릭합니다.
1. Azure AD 앱에 이름을 지정하고, **만들기**를 클릭합니다.

    **애플리케이션 ID** GUID를 기록합니다. 나중에 이를 연결 설정에 대한 클라이언트 ID로 제공합니다.

1. **애플리케이션 비밀** 아래에서 **새 암호 생성**을 클릭합니다.

    팝업에서 암호를 기록합니다. 나중에 이를 연결 설정에 대한 클라이언트 비밀로 제공합니다.

1. **플랫폼** 아래에서 **플랫폼 추가**를 클릭합니다.
1. **플랫폼 추가** 팝업에서 **웹**을 클릭합니다.

    1. **암시적 흐름 허용**을 표시한 채로 둡니다.
    1. **리디렉션 URL**의 경우 `https://token.botframework.com/.auth/web/redirect`를 입력합니다.
    1. **로그아웃 URL** 비어 둡니다.

1. **Microsoft Graph 권한** 아래에서 추가 위임된 권한을 추가할 수 있습니다.

    - 이 문서의 경우 **Mail.Read**, **Mail.Send**, **openid**, **profile**, **User.Read** 및 **User.ReadBasic.All** 권한을 추가합니다.
      연결 설정의 범위는 **openid** 및 **Mail.Read**와 같은 Azure AD 그래프의 리소스를 모두 포함해야 합니다.
    - 선택한 사용 권한을 기록합니다. 나중에 이를 연결 설정에 대한 범위로 제공합니다.

1. 페이지 맨 아래에서 **저장**을 클릭합니다.

이제 Azure AD v2 애플리케이션이 구성되었습니다.

---

### <a name="register-your-azure-ad-application-with-your-bot"></a>봇에 Azure AD 애플리케이션 등록

다음 단계는 앞에서 만든 Azure AD 애플리케이션을 봇에 등록하는 것입니다.

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD v1](#tab/aadv1)

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

# <a name="azure-ad-v2tabaadv2"></a>[Azure AD v2](#tab/aadv2)

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

## <a name="prepare-the-bot-code"></a>봇 코드 준비

이 프로세스를 완료하려면 봇의 앱 ID 및 암호가 필요합니다. 봇의 앱 ID 및 암호를 검색하려면 다음을 수행합니다.

1. [Azure portal][]에서 채널 등록 봇을 만든 리소스 그룹으로 이동합니다.
1. **배포** 창을 열고 봇에 대한 배포를 엽니다.
1. **입력** 창에서 봇의 **appId** 및 **appSecret**에 대한 값을 복사합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- TODO: Add guidance (once we have it) on how not to hard-code IDs and ABS auth. -->

1. Github에서 작업할 리포지토리: [**봇 인증**][cs-auth-sample] 또는 [**봇 인증 MSGraph**][cs-msgraph-sample]를 복제합니다.
1. 해당 특정 봇을 실행하는 방법은 GitHub 추가 정보 페이지에 대한 지침을 따릅니다. <!--TODO: Can we remove this step and still have the instructions make sense? What is the minimum we need to say in its place? -->
1. **appsettings.json** 업데이트:

    - `ConnectionName`을 봇에 추가한 OAuth 연결 설정의 이름으로 설정합니다.
    - `MicrosoftAppId` 및 `MicrosoftAppPassword`를 봇의 앱 ID 및 앱 비밀로 설정합니다.

      봇 비밀의 문자에 따라 암호를 XML로 이스케이프해야 할 수 있습니다. 예를 들어 모든 앰퍼샌드(&)는 `&amp;`로 인코딩되어야 합니다.

[!code-json[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/appsettings.json)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. Github에서 작업할 리포지토리: [**봇 인증**][js-auth-sample] 또는 [**봇 인증 MSGraph**][js-msgraph-sample]를 복제합니다.
1. 해당 특정 봇을 실행하는 방법은 GitHub 추가 정보 페이지에 대한 지침을 따릅니다. <!--TODO: Can we remove this step and still have the instructions make sense? What is the minimum we need to say in its place? -->
1. **.env** 업데이트:

    - `connectionName`을 봇에 추가한 OAuth 연결 설정의 이름으로 설정합니다.
    - `MicrosoftAppId` 및 `MicrosoftAppPassword` 값을 봇의 앱 ID 및 앱 암호로 설정합니다.

      봇 비밀의 문자에 따라 암호를 XML로 이스케이프해야 할 수 있습니다. 예를 들어 모든 앰퍼샌드(&)는 `&amp;`로 인코딩되어야 합니다.

    [!code-txt[.env](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/.env)]

---

**Microsoft 앱 ID** 및 **Microsoft 앱 암호** 값을 가져오는 방법을 모르는 경우 다음과 같이 수행할 수 있습니다.

- [여기서 설명한 대로](../bot-service-quickstart-registration.md#bot-channels-registration-password) 새 암호를 만듭니다.
- [여기서 설명한 대로](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret) 배포에서 **봇 채널 등록**을 사용하여 프로비저닝한 **Microsoft 앱 ID** 및 **Microsoft 앱 암호**를 검색합니다.

> [!NOTE]
> 이제 이 봇 코드를 Azure 구독에 게시할 수 있지만(프로젝트를 마우스 오른쪽 단추로 클릭하고 **게시** 선택) 이 문서에서는 필요하지 않습니다. Azure Portal에서 봇을 구성할 때 사용한 애플리케이션 및 호스팅 계획을 사용하는 게시 구성을 설정해야 합니다.

## <a name="test-the-bot"></a>봇 테스트

1. 아직 설치하지 않은 경우 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)를 설치합니다.
1. 샘플을 머신에서 로컬로 실행합니다.
1. 에뮬레이터를 시작하고 봇에 연결한 다음, 메시지를 보냅니다.

    - 봇에 연결할 때 봇의 앱 ID 및 암호를 제공해야 합니다.
    - `help`를 입력하여 봇에 사용 가능한 명령 목록을 보고 인증 기능을 테스트합니다.
    - 로그인하면 로그아웃할 때까지 자격 증명을 다시 제공할 필요가 없습니다.
    - 로그아웃하고, 인증을 취소하려면 `logout`을 입력합니다.

> [!NOTE]
> 봇 인증은 Bot Connector Service를 사용해야 합니다. 이 서비스는 봇에 대한 봇 채널 등록 정보에 액세스합니다.

# <a name="bot-authenticationtabbot-oauth"></a>[봇 인증](#tab/bot-oauth)

**봇 인증** 샘플에서 대화 상자는 사용자가 로그인한 후 사용자 토큰을 검색하도록 디자인됩니다.

![샘플 출력](media/how-to-auth/auth-bot-test.png)

# <a name="bot-authentication-msgraphtabbot-msgraph-auth"></a>[봇 인증 MSGraph](#tab/bot-msgraph-auth)

**봇 인증 MSGraph** 샘플에서 대화 상자는 사용자가 로그인한 후 제한된 명령 세트를 허용하도록 디자인됩니다.

![샘플 출력](media/how-to-auth/msgraph-bot-test.png)

---

## <a name="additional-information"></a>추가 정보

사용자가 봇에 사용자 로그인이 필요한 작업 수행을 요청하는 경우 봇은 `OAuthPrompt`를 사용하여 지정된 연결에 대한 토큰 검색을 시작할 수 있습니다. `OAuthPrompt`는 다음과 같이 진행되는 토큰 검색 흐름을 만듭니다.

1. Azure Bot Service에 현재 사용자 및 연결에 대한 토큰이 이미 있는지 확인합니다. 토큰이 있을 경우 토큰이 반환됩니다.
1. Azure Bot Service에 캐시된 토큰이 없을 경우 사용자가 클릭할 수 있는 로그인 단추인 `OAuthCard`가 생성됩니다.
1. 사용자가 `OAuthCard` 로그인 단추를 클릭하면 Azure Bot Service가 사용자의 토큰을 직접 전송하거나 사용자가 6자리 인증 코드를 제공하여 채팅 창에 입력하도록 합니다.
1. 사용자에게 인증 코드가 제공되면 봇이 이 인증 코드를 사용자의 토큰으로 교환합니다.

다음 섹션에서는 샘플이 몇 가지의 일반적인 인증 작업을 구현하는 방법을 설명합니다.

### <a name="use-an-oauth-prompt-to-sign-the-user-in-and-get-a-token"></a>OAuth 프롬프트를 사용하여 사용자를 로그인하고 토큰을 가져옵니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![봇 아키텍처](media/how-to-auth/architecture.png)

<!-- The two authentication samples have nearly identical architecture. Using 18.bot-authentication for the sample code. -->

**Dialogs\MainDialog.cs**

해당 생성자의 **MainDialog**에 OAuth 프롬프트를 추가합니다. 여기서 연결 이름에 대한 값은 **appsettings.json** 파일에서 검색했습니다.

[!code-csharp[Add OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=23-31)]

대화 상자 단계 내에서 `BeginDialogAsync`를 사용하여 사용자에게 로그인하라고 요청하는 OAuth 프롬프트를 시작합니다.

- 사용자가 이미 로그인한 경우 사용자에게 프롬프트를 표시하지 않고 토큰 응답 이벤트를 생성합니다.
- 사용자가 아직 로그인하지 않은 경우 사용자에게 로그인하라는 프롬프트를 표시합니다. Azure Bot Service는 사용자가 로그인을 시도한 후 토큰 응답 이벤트를 보냅니다.

[!code-csharp[Use the OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=49)]

다음 대화 상자 단계 내에서 이전 단계의 결과에 토큰이 존재하는지 확인합니다. Null이 아닌 경우 사용자가 성공적으로 로그인합니다.

[!code-csharp[Get the OAuthPrompt result](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=54-58)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![봇 아키텍처](media/how-to-auth/architecture-js.png)

**dialogs/mainDialog.js**

해당 생성자의 **MainDialog**에 OAuth 프롬프트를 추가합니다. 여기서 연결 이름에 대한 값은 **.env** 파일에서 검색했습니다.

[!code-javascript[Add OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=23-28)]

대화 상자 단계 내에서 `beginDialog`를 사용하여 사용자에게 로그인하라고 요청하는 OAuth 프롬프트를 시작합니다.

- 사용자가 이미 로그인한 경우 사용자에게 프롬프트를 표시하지 않고 토큰 응답 이벤트를 생성합니다.
- 사용자가 아직 로그인하지 않은 경우 사용자에게 로그인하라는 프롬프트를 표시합니다. Azure Bot Service는 사용자가 로그인을 시도한 후 토큰 응답 이벤트를 보냅니다.

[!code-javascript[Use OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=57)]

다음 대화 상자 단계 내에서 이전 단계의 결과에 토큰이 존재하는지 확인합니다. Null이 아닌 경우 사용자가 성공적으로 로그인합니다.

[!code-javascript[Get OAuthPrompt result](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=61-64)]

---

### <a name="wait-for-a-tokenresponseevent"></a>TokenResponseEvent에 대한 대기

OAuth 프롬프트를 시작할 때 사용자의 토큰을 검색하는 토큰 응답 이벤트를 기다립니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\AuthBot.cs**

**AuthBot**은 `ActivityHandler`에서 파생되며 토큰 응답 이벤트 작업을 명시적으로 처리합니다. 여기서는 OAuth 프롬프트에서 이벤트를 처리하고 토큰을 검색할 수 있는 활성 대화 상자를 계속합니다.

[!code-csharp[OnTokenResponseEventAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Bots/AuthBot.cs?range=32-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/authBot.js**

**AuthBot**은 `ActivityHandler`에서 파생되며 토큰 응답 이벤트 작업을 명시적으로 처리합니다. 여기서는 OAuth 프롬프트에서 이벤트를 처리하고 토큰을 검색할 수 있는 활성 대화 상자를 계속합니다.

[!code-javascript[onTokenResponseEvent](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/bots/authBot.js?range=28-33)]

---

### <a name="log-the-user-out"></a>사용자 로그아웃

연결이 시간 초과할 때까지 기다리지 않고 사용자가 명시적으로 로그아웃하도록 하는 것이 더 좋습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\LogoutDialog.cs**

[!code-csharp[Allow logout](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/LogoutDialog.cs?range=20-61&highlight=35)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/logoutDialog.js**

[!code-javascript[Allow logout](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/logoutDialog.js?range=13-42&highlight=25)]

---

### <a name="further-reading"></a>추가 참고 자료

- [Bot Framework 추가 리소스](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)에는 추가 지원의 링크가 포함되어 있습니다.
- [Bot Framework SDK](https://github.com/microsoft/botbuilder) 리포지토리에는 Bot Builder SDK와 관련된 리포지토리, 샘플, 도구 및 사양 정보가 들어 있습니다.

<!-- Footnote-style links -->

[Azure Portal]: https://ms.portal.azure.com
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
