---
title: Azure Bot Service를 통해 봇에 인증 추가 | Microsoft Docs
description: Azure Bot Service 인증 기능을 사용하여 봇에 SSO를 추가하는 방법을 알아봅니다.
author: JonathanFingold
ms.author: JonathanFingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ROBOTS: NOINDEX
ms.date: 10/04/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d1b14d405b4df19db81269fc1f588305840485bd
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405966"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Azure Bot Service를 통해 봇에 인증 추가

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]  

이 자습서에서는 사용자를 Azure AD(Azure Active Directory), GitHub 및 Uber 등과 같은 다양한 ID 공급자에 인증하는 봇을 쉽게 개발하는 기능을 제공하는 Azure Bot Service에서 새 봇 인증 기능을 사용합니다. 이러한 업데이트는 또한 일부 클라이언트에 대한 _매직 코드 확인_을 제거하여 향상된 사용자 환경을 위한 단계를 수행합니다.

이전에 봇은 OAuth 컨트롤러 및 로그인 링크를 포함하고, 대상 클라이언트 ID 및 비밀을 저장하고, 사용자 토큰 관리를 수행해야 했습니다.
<!--
These capabilities were bundled in the BotAuth and AuthBot samples that are on GitHub.
-->

이제 봇 개발자는 더 이상 OAuth 컨트롤러를 호스트하거나 토큰 수명 주기를 관리할 필요가 없습니다. 이는 모두 Azure Bot Service에서 수행될 수 있습니다.

기능은 다음과 같습니다.

- 6자리 매직 코드 확인에 대한 필요를 제거하는 새 WebChat 및 DirectLineJS 라이브러리와 같은 새로운 인증 기능을 지원하는 채널에 대한 향상된 기능
- 다양한 OAuth ID 공급자에 대한 연결 설정을 추가, 삭제 및 구성하는 Azure Portal에 대한 향상된 기능
- Azure AD(v1 및 v2 엔드포인트 모두), GitHub 및 기타를 포함하는 다양한 기본 제공 ID 공급자에 대한 지원
- 토큰을 검색하고, OAuthCards를 만들고 TokenResponse 이벤트를 처리할 수 있도록 C# 및 Node.js Bot Framework SDK에 대한 업데이트
- Azure AD(v1 및 v2 엔드포인트) 및 GitHub에 인증하는 봇을 만드는 방법에 대한 샘플

기존 봇에 이러한 기능을 추가하기 위해 이 문서의 단계에서 추정할 수 있습니다. 다음은 새로운 인증 기능을 보여주는 샘플 봇입니다.

| 샘플 | BotBuilder 버전 | 설명 |
|:---|:---:|:---|
| [AadV1Bot](https://aka.ms/AadV1Bot) | v3 | Azure AD v1 엔드포인트를 사용하여 v3 C# SDK에서 OAuthCard 지원을 보여줍니다. |
| [AadV2Bot](https://aka.ms/AadV2Bot) | v3 |  Azure AD v2 엔드포인트를 사용하여 v3 C# SDK에서 OAuthCard 지원을 보여줍니다. |
| [GitHubBot](https://aka.ms/GitHubBot) | v3 |  GitHub를 사용하여 v3 C# SDK에서 OAuthCard 지원을 보여줍니다. |
| [BasicOAuth](https://aka.ms/BasicOAuth) | v3 |  v3 C# SDK에서 OAuth 2.0 지원을 보여줍니다. |

> [!NOTE]
> 인증 기능은 또한 BotBuilder v3을 사용하여 Node.js와 작동합니다. 그러나 이 문서에서는 샘플 C# 코드만을 설명합니다.

추가 정보 및 지원은 [Bot Framework 추가 리소스](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)를 참조하세요.

## <a name="overview"></a>개요

이 자습서에서는 Azure AD v1 또는 v2 토큰을 사용하여 Microsoft Graph에 연결하는 샘플 봇을 만듭니다. <!--verify this info and fix wording--> 이 프로세스의 일부로, GitHub 리포지토리에서 코드를 사용하고, 이 자습서에서는 봇 애플리케이션을 포함하여 설정하는 방법을 설명합니다.

- [봇 및 인증 애플리케이션 만들기](#create-your-bot-and-an-authentication-application)
- [봇 샘플 코드 준비](#prepare-the-bot-sample-code)
- [에뮬레이터를 사용하여 봇 테스트](#use-the-emulator-to-test-your-bot)

이러한 단계를 완료하려면 Visual Studio 2017, npm, 노드 및 git를 설치해야 합니다. 또한 Azure, OAuth 2.0 및 봇 개발에 대해 알고 있어야 합니다.

완료하면 이메일 확인 및 보내기 또는 사용자 및 관리자 표시와 같은 Azure AD 애플리케이션에 대한 몇 가지 간단한 작업에 응답할 수 있는 봇을 갖습니다. 이렇게 하기 위해 봇에서 Microsoft.Graph 라이브러리에 대해 Azure AD 애플리케이션의 토큰을 사용합니다.

마지막 섹션에서는 일부 봇 코드를 세분화합니다.

- [토큰 검색 흐름에 대한 참고 사항](#notes-on-the-token-retrieval-flow)

## <a name="create-your-bot-and-an-authentication-application"></a>봇 및 인증 애플리케이션 만들기

배포된 봇의 코드에 메시징 엔드포인트를 설정하는 등록 봇을 만들고, 봇에서 Office 365에 액세스하도록 허용하는 Azure AD(v1 또는 v2) 애플리케이션을 만들어야 합니다.

> [!NOTE]
> 이러한 인증 기능은 다른 유형의 봇과 작동합니다. 그러나 이 자습서에는 등록 전용 봇을 사용합니다.

### <a name="register-an-application-in-azure-ad"></a>Azure AD에서 애플리케이션 등록

봇이 Microsoft Graph API에 연결하는 데 사용할 수 있는 Azure AD 애플리케이션이 필요합니다.

이 봇의 경우 Azure AD v1 또는 v2 엔드포인트를 사용할 수 있습니다.
v1 및 v2 엔드포인트 간의 차이점에 대한 정보는 [v1-v2 비교](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare) 및 [Azure AD v2.0 엔드포인트 개요](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)를 참조하세요.

#### <a name="to-create-an-azure-ad-application"></a>Azure AD 애플리케이션을 만들려면 다음을 수행합니다.

다음 단계에 따라 새 Azure AD 애플리케이션을 만듭니다. 만드는 앱에 v1 또는 v2 엔드포인트를 사용할 수 있습니다.

> [!TIP]
> 애플리케이션이 요청한 대리자 권한에 동의할 수 있는 테넌트에 Azure AD 애플리케이션을 만들고 등록해야 합니다.

1. Azure Portal에서 [Azure Active Directory][azure-aad-blade] 패널을 엽니다.
    올바른 테넌트에 있지 않은 경우 **디렉터리 전환**을 클릭하여 올바른 테넌트로 전환합니다. (테넌트를 만드는 방법에 대한 지침은 [포털에 액세스 및 테넌트 만들기](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)를 참조하세요.)
1. **앱 등록** 패널을 엽니다.
1. **앱 등록** 패널에서 **새 등록**을 클릭합니다.
1. 필수 필드를 입력하고 앱 등록을 만듭니다.

   1. 애플리케이션의 이름을 지정합니다.
   1. 애플리케이션에 **지원되는 계정 유형**을 선택합니다. (옵션 중 하나는 이 샘플에서 작동합니다.)
   1. **리디렉션 URI**에 대해
       1. **웹**을 선택합니다.
       1. `https://token.botframework.com/.auth/web/redirect` URL을 선택합니다.
   1. **등록**을 클릭합니다.

      - 생성되면 Azure에 앱에 대한 **개요** 페이지가 표시됩니다.
      - **애플리케이션(클라이언트) ID** 값을 기록해 둡니다. 나중에 Azure AD 애플리케이션을 봇에 등록할 때 이 값을 _클라이언트 ID_로 사용할 것입니다.
      - **디렉터리(테넌트) ID** 값도 기록해 둡니다. 애플리케이션을 봇에 등록할 때 이 값도 사용됩니다.

1. 탐색 창에서 **인증서 및 비밀**을 클릭하여 애플리케이션 비밀을 만듭니다.

   1. **클라이언트 비밀** 아래에서 **새 클라이언트 비밀**을 클릭합니다.
   1. 이 앱을 만드는 데 필요할 수도 있는 비밀을 식별하는 설명(예: `bot login`)을 추가합니다.
   1. **만료**를 **안 함**으로 선택합니다.
   1. **추가**를 클릭합니다.
   1. 페이지를 나가기 전에 비밀을 기록해 둡니다. 나중에 Azure AD 애플리케이션을 봇에 등록할 때 이 값을 _클라이언트 암호_로 사용할 것입니다.

1. 탐색 창에서 **API 사용 권한**을 클릭하여 **API 사용 권한** 패널을 엽니다. 앱의 API 사용 권한을 명시적으로 설정하는 것이 좋습니다.

   1. **사용 권한 추가**을 클릭하여 **API 사용 권한 요청** 창을 표시합니다.
   1. 샘플에서 **Microsoft API** 및 **Microsoft Graph**를 선택합니다.
   1. **위임된 권한**을 선택하고 필요한 사용 권한이 선택되어 있는지 확인합니다. 이 샘플에는 다음과 같은 사용 권한이 필요합니다.

      > [!NOTE]
      > **관리자 동의가 필요함**으로 표시된 모든 사용 권한에는 사용자와 테넌트 관리자가 모두 로그인되어 있어야 합니다. 그래야 봇이 거리를 둘 수 있습니다.

      - **openid**
      - **profile**
      - **Mail.Read**
      - **Mail.Send**
      - **User.Read**
      - **User.ReadBasic.All**

   1. **권한 추가**를 클릭합니다. (봇을 통해 사용자가 이 앱에 처음 액세스하면 동의를 허용해야 합니다.)

이제 Azure AD 애플리케이션이 구성되었습니다.

### <a name="create-your-bot-on-azure"></a>Azure에서 봇 만들기

[Azure Portal](https://portal.azure.com/)을 사용하여 **봇 채널 등록**을 만듭니다.

### <a name="register-your-azure-ad-application-with-your-bot"></a>봇에 Azure AD 애플리케이션 등록

다음 단계는 방금 만든 Azure AD 애플리케이션을 봇에 등록하는 것입니다.

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD v1](#tab/aadv1)

1. [Azure Portal](http://portal.azure.com/)에서 봇의 리소스 페이지로 이동합니다.
1. **설정**을 클릭합니다.
1. 페이지 아래쪽의 **OAuth 연결 설정** 아래에서 **설정 추가**를 클릭합니다.
1. 다음과 같이 양식을 채웁니다.

    1. **이름**에 대해 연결의 이름을 입력합니다. 이 이름을 봇 코드에 사용할 것입니다.
    1. **서비스 공급자**에 대해 **Azure Active Directory**를 선택합니다. 이를 선택하면 Azure AD 관련 필드가 표시됩니다.
    1. **클라이언트 ID**에 대해 Azure AD v1 애플리케이션에 대해 기록한 애플리케이션(클라이언트) ID를 입력합니다.
    1. **클라이언트 비밀**에 Azure AD 앱에 봇이 액세스하는 것을 허용하기 위해 만든 비밀을 입력합니다.
    1. **권한 부여 유형**에 대해 `authorization_code`를 입력합니다.
    1. **로그인 URL**에 대해 `https://login.microsoftonline.com`을 입력합니다.
    1. **Tenant ID**에 Azure AD 앱에 대해 앞서 기록해 둔 디렉터리(테넌트) ID를 입력합니다.

       인증될 수 있는 사용자와 연결된 테넌트가 됩니다.

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
    1. **클라이언트 ID**에 대해 Azure AD v1 애플리케이션에 대해 기록한 애플리케이션(클라이언트) ID를 입력합니다.
    1. **클라이언트 비밀**에 Azure AD 앱에 봇이 액세스하는 것을 허용하기 위해 만든 비밀을 입력합니다.
    1. **Tenant ID**에 Azure AD 앱에 대해 앞서 기록해 둔 디렉터리(테넌트) ID를 입력합니다.

       인증될 수 있는 사용자와 연결된 테넌트가 됩니다.

    1. **범위**에 대해 애플리케이션 등록에서 선택한 사용 권한의 이름을 입력합니다. `Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`

        > [!NOTE]
        > Azure AD v2의 경우 **범위**는 대/소문자 구분, 공백으로 구분된 값의 목록을 사용합니다.

1. **저장**을 클릭합니다.

> [!NOTE]
> 이러한 값을 사용하면 애플리케이션은 Microsoft Graph API를 통해 Office 365 데이터에 액세스할 수 있습니다.

---

#### <a name="to-test-your-connection"></a>연결을 테스트하려면

1. 방금 만든 연결을 엽니다.
1. **서비스 공급자 연결 설정** 창의 맨 위에 있는 **연결 테스트**를 클릭합니다.
1. 처음에 앱에서 요청하는 사용 권한을 나열하는 새 브라우저 탭을 열고 수락하라는 메시지를 표시해야 합니다.
1. **Accept**를 클릭합니다.
1. 그런 다음, **`<your-connection-name>' Succeeded에 대한 연결 테스트** 페이지로 리디렉션해야 합니다.

## <a name="prepare-the-bot-sample-code"></a>봇 샘플 코드 준비

1. [https://github.com/Microsoft/BotBuilder](https://github.com/Microsoft/BotBuilder ) 에서 github 리포지토리를 복제합니다.
1. 솔루션 `BotBuilder\CSharp\Microsoft.Bot.Builder.sln`을 열고 빌드합니다.
1. 해당 솔루션을 닫고 `BotBuilder\CSharp\Samples\Microsoft.Bot.Builder.Samples.sln`을 엽니다.
1. 시작 프로젝트를 설정합니다.
    - v1 Azure AD 애플리케이션을 사용하는 봇의 경우 `Microsoft.Bot.Sample.AadV1Bot` 프로젝트를 사용합니다.
    - v2 Azure AD 애플리케이션을 사용하는 봇의 경우 `Microsoft.Bot.Sample.AadV2Bot` 프로젝트를 사용합니다.
1. `Web.config` 파일을 열고, 다음과 같이 앱 설정을 수정합니다.
    1. `ConnectionName`을 봇의 OAuth 2.0 연결 설정을 구성할 때 사용한 값으로 설정합니다.
    1. `MicrosoftAppId` 값을 봇의 앱 ID로 설정합니다.
    1. `MicosoftAppPassword` 값을 봇의 비밀로 설정합니다.

    > [!IMPORTANT]
    > 비밀의 문자에 따라 암호를 XML 이스케이프해야 할 수 있습니다. 예를 들어 모든 앰퍼샌드(&)는 `&amp;`로 인코딩되어야 합니다.

    ```xml
    <appSettings>
        <add key="ConnectionName" value="<your-AAD-connection-name>"/>
        <add key="MicrosoftAppId" value="<your-bot-appId>" />
        <add key="MicrosoftAppPassword" value="<your-bot-password>" />
    </appSettings>
    ```

    **Microsoft 앱 ID** 및 **Microsoft 앱 암호** 값을 가져오는 방법을 알지 못하는 경우 [bot-channels-registration-password](bot-service-quickstart-registration.md#bot-channels-registration-password)에 설명된 대로 새 암호를 만들거나 [find-your-azure-bots-appid-and-appsecret](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret)에 설명된 배포에서 **봇 채널 등록**을 사용하여 프로비저닝된 **Microsoft 앱 ID** 및 **Microsoft 앱 암호**를 검색합니다.

    > [!NOTE]
    > 이제 이 봇 코드를 Azure 구독에 게시할 수 있지만(프로젝트를 마우스 오른쪽 단추로 클릭하고 **게시** 선택) 이 자습서에서는 필요하지 않습니다. Azure Portal에서 봇을 구성할 때 사용한 애플리케이션 및 호스팅 계획을 사용하는 게시 구성을 설정해야 합니다.

## <a name="use-the-emulator-to-test-your-bot"></a>에뮬레이터를 사용하여 봇 테스트

봇을 로컬로 테스트하려면 [봇 에뮬레이터](https://github.com/Microsoft/BotFramework-Emulator)를 설치해야 합니다. v3 또는 v4 에뮬레이터를 사용할 수 있습니다.

1. 봇을 시작합니다(디버깅을 사용하거나 사용하지 않고).
1. 페이지에 대한 localhost 포트 번호를 적어 둡니다. 이 정보는 봇과 상호 작용하기 위해 필요합니다.
1. 에뮬레이터를 시작합니다.
1. 봇에 연결합니다.

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

사용자가 봇에 사용자 로그인이 필요한 작업 수행을 요청하는 경우 봇은 `Microsoft.Bot.Builder.Dialogs.GetTokenDialog`를 사용하여 지정된 연결에 대한 토큰 검색을 시작할 수 있습니다. 다음 두 코드 조각은 `GetTokenDialog` 클래스에서 제공됩니다.

### <a name="check-for-a-cached-token"></a>캐시된 토큰에 대한 확인

이 코드에서 봇은 먼저 Azure Bot Service에 사용자에 대한 토큰(현재 작업 전송자에 의해 식별됨) 및 지정된 ConnectionName(구성에서 사용되는 연결 이름)이 이미 있는지 빠른 검사를 수행합니다. Azure Bot Service는 캐시된 토큰이 이미 있거나 없습니다. GetUserTokenAsync에 대한 호출은 이 '빠른 검사'를 수행합니다. Azure Bot Service에 토큰이 있고 반환하는 경우 토큰을 즉시 사용할 수 있습니다. Azure Bot Service에 토큰이 없는 경우 이 메서드는 Null을 반환합니다. 이 경우 봇은 로그인할 사용자에 대한 사용자 지정된 OAuthCard를 보낼 수 있습니다.

```csharp
// First ask Bot Service if it already has a token for this user
var token = await context.GetUserTokenAsync(ConnectionName).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
    await SendOAuthCardAsync(context, (Activity)context.Activity);
}
```

### <a name="send-an-oauthcard-to-the-user"></a>사용자에게 OAuthCard 보내기

원하는 모든 텍스트 및 단추 텍스트를 사용하여 OAuthCard를 사용자 지정할 수 있습니다. 중요한 부분은 다음과 같습니다.

- `ContentType`을 `OAuthCard.ContentType`으로 설정합니다.
- `ConnectionName` 속성을 사용하려는 연결의 이름으로 설정합니다.
- `Type` `ActionTypes.Signin`의 `CardAction`이 있는 단추 하나를 포함합니다. 로그인 링크에 대한 값을 지정할 필요가 없습니다.

이 호출의 끝에서 봇은 "토큰이 다시 돌아오기를 기다려야" 합니다. 로그인을 위해 수행해야 하는 많은 사용자가 있을 수 있으므로 이 대기는 주 작업 스트림에서 수행됩니다.

```csharp
private async Task SendOAuthCardAsync(IDialogContext context, Activity activity)
{
    await context.PostAsync($"To do this, you'll first need to sign in.");

    var reply = await context.Activity.CreateOAuthReplyAsync(_connectionName, _signInMessage, _buttonLabel).ConfigureAwait(false);
    await context.PostAsync(reply);

    context.Wait(WaitForToken);
}
```

### <a name="wait-for-a-tokenresponseevent"></a>TokenResponseEvent에 대한 대기

이 코드에서 봇의 대화 상자 클래스는 `TokenResponseEvent`에 대해 대기합니다(대화 상자 스택에 라우팅되는 방법에 대한 정보는 아래와 같음). `WaitForToken` 메서드는 먼저 이 이벤트가 전송되었는지 확인합니다. 전송된 경우 봇에서 사용할 수 있습니다. 그렇지 않은 경우 `WaitForToken` 메서드는 봇에 전송된 모든 텍스트를 사용하고 `GetUserTokenAsync`에 전달합니다. 이에 대한 이유는 일부 클라이언트(예: WebChat)에서 매직 코드 확인 코드가 필요하지 않고 `TokenResponseEvent`에서 토큰을 직접 보낼 수 있기 때문입니다. 다른 클라이언트는 여전히 매직 코드가 필요합니다(예: Facebook 또는 Slack). Azure Bot Service는 이러한 클라이언트에게 6자리 매직 코드를 제공하고 사용자에게 채팅 창에 이를 입력하도록 요청합니다. 적합하지 않은 경우 이는 '대체' 동작이므로 `WaitForToken`가 코드를 받는 경우 봇은 이 코드를 Azure Bot Service에 보내고 토큰을 다시 가져올 수 있습니다. 이 호출 또한 실패하는 경우 오류를 보고하거나 다른 작업을 수행할 수 있습니다. 그러나 대부분의 경우에서 봇은 이제 사용자 토큰을 갖습니다.

**MessageController.cs** 파일을 살펴보면 이 유형의 `Event` 작업 또한 대화 상자 스택으로 라우팅되는 것을 확인할 수 있습니다.

```csharp
private async Task WaitForToken(IDialogContext context, IAwaitable<object> result)
{
    var activity = await result as Activity;

    var tokenResponse = activity.ReadTokenResponseContent();
    if (tokenResponse != null)
    {
        // Use the token to do exciting things!
    }
    else
    {
        if (!string.IsNullOrEmpty(activity.Text))
        {
            tokenResponse = await context.GetUserTokenAsync(ConnectionName,
                                                               activity.Text);
            if (tokenResponse != null)
            {
                // Use the token to do exciting things!
                return;
            }
        }
        await context.PostAsync($"Hmm. Something went wrong. Let's try again.");
        await SendOAuthCardAsync(context, activity);
    }
}
```

### <a name="message-controller"></a>메시지 컨트롤러

봇에 대한 후속 호출에서 토큰은 이 샘플 봇에서 캐시되지 않습니다. 봇은 Azure Bot Service에 토큰을 항상 요청할 수 있기 때문입니다. 이렇게 하면 봇은 토큰 수명 주기를 관리하고, 토큰을 새로 고치는 등의 작업을 수행할 필요가 없습니다. Azure Bot Service에서 이 모든 작업을 수행합니다.

```csharp
else if(message.Type == ActivityTypes.Event)
{
    if(message.IsTokenResponseEvent())
    {
        await Conversation.SendAsync(message, () => new Dialogs.RootDialog());
    }
}
```
## <a name="additional-resources"></a>추가 리소스
[Bot Framework SDK](https://github.com/microsoft/botbuilder)
