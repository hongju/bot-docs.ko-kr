---
title: Azure Bot Service를 통해 봇에 인증 추가 | Microsoft Docs
description: Azure Bot Service 인증 기능을 사용하여 봇에 SSO를 추가하는 방법을 알아봅니다.
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 10/30/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 27e0b54b4e790e76c55deb858e50d8c81507443a
ms.sourcegitcommit: 91156d0866316eda8d68454a0c4cd74be5060144
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/07/2018
ms.locfileid: "53010598"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Azure Bot Service를 통해 봇에 인증 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

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
- 토큰을 검색하고, OAuthCards를 만들고 TokenResponse 이벤트를 처리할 수 있도록 C# 및 Node.js Bot Builder SDK에 대한 업데이트
- Azure AD에 인증하는 봇을 만드는 방법에 대한 샘플입니다.

기존 봇에 이러한 기능을 추가하기 위해 이 문서의 단계에서 추정할 수 있습니다. 다음은 새로운 인증 기능을 보여주는 샘플 봇입니다.

| 샘플 | BotBuilder 버전 | 설명 |
|:---|:---:|:---|
| **봇 인증**([C#](https://aka.ms/v4cs-bot-auth-sample) / [JS](https://aka.ms/v4js-bot-auth-sample)) | v4 | OAuthCard 지원을 보여 줍니다. |
| **봇 인증 MSGraph**([C#](https://aka.ms/v4cs-auth-msgraph-sample) / [JS](https://aka.ms/v4js-auth-msgraph-sample)) | v4 |  OAuth 2를 통한 Microsoft Graph API 지원을 보여 줍니다. |

> [!NOTE]
> 인증 기능은 BotBuilder v3에서도 작동합니다. 그러나 이 문서에서는 샘플 v4 코드만을 설명합니다.

추가 정보 및 지원은 [Bot Framework 추가 리소스](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)를 참조하세요.

## <a name="overview"></a>개요

이 자습서에서는 Azure AD v1 또는 v2 토큰을 사용하여 Microsoft Graph에 연결하는 샘플 봇을 만듭니다. 이 프로세스의 일환으로 [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub 리포지토리의 코드를 사용하고, 이 자습서에서는 봇 애플리케이션을 포함하여 설정하는 방법에 대해 설명합니다.

- **봇 및 인증 응용 프로그램 만들기**
- **봇 샘플 코드 준비**
- **에뮬레이터를 사용하여 봇 테스트**

이러한 단계를 완료하려면 Visual Studio 2017, npm, 노드 및 git를 설치해야 합니다. 또한 Azure, OAuth 2.0 및 봇 개발에 대해 알고 있어야 합니다.

완료하면 이메일 확인 및 보내기 또는 사용자 및 관리자 표시와 같은 Azure AD 애플리케이션에 대한 몇 가지 간단한 작업에 응답할 수 있는 봇을 갖습니다. 이렇게 하기 위해 봇에서 Microsoft.Graph 라이브러리에 대해 Azure AD 애플리케이션의 토큰을 사용합니다.

마지막 섹션에서는 일부 봇 코드를 세분화합니다.

- **토큰 검색 흐름에 대한 참고 사항**

## <a name="create-your-bot-and-an-authentication-application"></a>봇 및 인증 애플리케이션 만들기

봇 코드를 게시하는 등록 봇을 만들고, 봇에서 Office 365에 액세스하도록 허용하는 Azure AD(v1 또는 v2) 애플리케이션을 만들어야 합니다.

> [!NOTE]
> 이러한 인증 기능은 다른 유형의 봇과 작동합니다. 그러나 이 자습서에는 등록 전용 봇을 사용합니다.

### <a name="register-an-application-in-azure-ad"></a>Azure AD에서 애플리케이션 등록

봇에서 Microsoft Graph API, 사용자 고유의 Azure AD로 보호된 리소스 등에 연결하는 데 사용할 수 있는 Azure AD 애플리케이션이 필요합니다.

이 봇의 경우 Azure AD v1 또는 v2 엔드포인트를 사용할 수 있습니다.
v1 및 v2 엔드포인트 간의 차이점에 대한 정보는 [v1-v2 비교](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare) 및 [Azure AD v2.0 엔드포인트 개요](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)를 참조하세요.

#### <a name="to-create-an-azure-ad-v1-application"></a>Azure AD v1 애플리케이션을 만들려면

1. [Azure Portal에서 Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview)로 이동합니다.
1. **앱 등록**을 클릭합니다.
1. **앱 등록** 패널에서 **새 응용 프로그램 등록**을 클릭합니다.
1. 필수 필드를 입력하고 앱 등록을 만듭니다.
   1. 애플리케이션의 이름을 지정합니다.
   1. **응용 프로그램 유형**을 **웹앱/API**로 설정합니다.
   1. **로그온 URL**을 `https://token.botframework.com/.auth/web/redirect`로 설정합니다.
   1. **만들기**를 클릭합니다.
      - 만들어지면 **등록된 앱** 창에 표시됩니다.
      - **응용 프로그램 ID** 값을 기록합니다. 나중에 이를 _클라이언트 ID_로 제공합니다.
1. **설정**을 클릭하여 응용 프로그램을 구성합니다.
1. **키**를 클릭하여 **키** 패널을 엽니다.
   1. **암호** 아래에서 `BotLogin` 키를 만듭니다.
   1. 해당 **기간**을 **무기한**으로 설정합니다.
   1. **저장**을 클릭하고 키 값을 기록합니다. 나중에 이를 _애플리케이션 비밀_로 제공합니다.
   1. **키** 패널을 닫습니다.
1. **필요한 사용 권한**을 클릭하여 **필요한 사용 권한** 패널을 엽니다.
   1. **추가**를 클릭합니다.
   1. **API 선택**을 클릭한 다음, **Microsoft Graph**를 선택하고 **선택**을 클릭합니다.
   1. **사용 권한 선택**을 클릭합니다. 애플리케이션에서 사용할 애플리케이션 사용 권한을 선택합니다.

      > [!NOTE]
      > **관리자 필요**로 표시된 모든 사용 권한에는 사용자 및 테넌트 관리자가 모두 로그인해야 합니다. 따라서 봇은 이를 사용하지 않는 경향이 있습니다.

      다음 Microsoft Graph 위임된 사용 권한을 선택합니다.
      - 모든 사용자의 기본 프로필 읽기
      - 사용자 메일 읽기
      - 로그인 및 사용자 프로필 읽기
      - 사용자로 메일 보내기
      - 사용자의 기본 프로필 보기기
      - 사용자의 이메일 주소 보기

   1. **선택**을 클릭한 다음, **완료**를 클릭합니다.
   1. **필요한 사용 권한** 패널을 닫습니다.

이제 Azure AD v1 애플리케이션이 구성되었습니다.

#### <a name="to-create-an-azure-ad-v2-application"></a>Azure AD v2 애플리케이션을 만들려면

1. [Microsoft 응용 프로그램 등록 포털](https://apps.dev.microsoft.com)로 이동합니다.
1. **앱 추가**를 클릭합니다.
1. Azure AD 앱에 이름을 지정하고, **만들기**를 클릭합니다.

    **응용 프로그램 ID** GUID를 기록합니다. 나중에 이를 연결 설정에 대한 클라이언트 ID로 제공합니다.

1. **응용 프로그램 비밀** 아래에서 **새 암호 생성**을 클릭합니다.

    팝업에서 암호를 기록합니다. 나중에 이를 연결 설정에 대한 클라이언트 비밀로 제공합니다.

1. **플랫폼** 아래에서 **플랫폼 추가**를 클릭합니다.
1. **플랫폼 추가** 팝업에서 **웹**을 클릭합니다.
    1. **암시적 흐름 허용**을 표시한 채로 둡니다.
    1. **리디렉션 URL**의 경우 `https://token.botframework.com/.auth/web/redirect`를 입력합니다.
    1. **로그아웃 URL** 비어 둡니다.
1. **Microsoft Graph 권한** 아래에서 추가 위임된 권한을 추가할 수 있습니다.
    - 이 자습서의 경우 **Mail.Read**, **Mail.Send**, **openid**, **프로필**, **User.Read** 및 **User.ReadBasic.All** 권한을 추가합니다.
      연결 설정의 범위는 **openid** 및 **Mail.Read**와 같은 Azure AD 그래프의 리소스를 모두 포함해야 합니다.
    - 선택한 사용 권한을 기록합니다. 나중에 이를 연결 설정에 대한 범위로 제공합니다.

1. 페이지 맨 아래에서 **저장**을 클릭합니다.

### <a name="create-your-bot-on-azure"></a>Azure에서 봇 만들기

[Azure Portal](https://portal.azure.com/)을 사용하여 **봇 채널 등록**을 만듭니다.

### <a name="register-your-azure-ad-application-with-your-bot"></a>봇에 Azure AD 애플리케이션 등록

다음 단계는 방금 만든 Azure AD 애플리케이션을 봇에 등록하는 것입니다.

#### <a name="to-register-an-azure-ad-v1-application"></a>Azure AD v1 애플리케이션을 등록하려면

1. [Azure Portal](http://portal.azure.com/)에서 봇의 리소스 페이지로 이동합니다.
1. **설정**을 클릭합니다.
1. 페이지 아래쪽의 **OAuth 연결 설정** 아래에서 **설정 추가**를 클릭합니다.
1. 다음과 같이 양식을 채웁니다.
    1. **이름**에 대해 연결의 이름을 입력합니다. 봇 코드에서 사용합니다.
    1. **서비스 공급자**에 대해 **Azure Active Directory**를 선택합니다. 이를 선택하면 Azure AD 관련 필드가 표시됩니다.
    1. **클라이언트 ID**에 대해 Azure AD v1 응용 프로그램에 대해 기록한 응용 프로그램 ID를 입력합니다.
    1. **클라이언트 비밀**에 대해 응용 프로그램의 `BotLogin` 키에 대해 기록한 키를 입력합니다.
    1. **권한 부여 유형**에 대해 `authorization_code`를 입력합니다.
    1. **로그인 URL**에 대해 `https://login.microsoftonline.com`을 입력합니다.
    1. **테넌트 ID**에 대해 Azure Active Directory에 대한 테넌트 ID를 입력합니다(예: `microsoft.com` 또는 `common`).

       인증될 수 있는 사용자와 연결된 테넌트가 됩니다. 누구든지 봇을 통해 자신을 인증하도록 허용하려면 `common` 테넌트를 사용합니다.

    1. **리소스 URL**에 `https://graph.microsoft.com/`을 입력합니다.
    1. **범위**를 비워 둡니다.
1. **저장**을 클릭합니다.

> [!NOTE]
> 이러한 값을 사용하면 애플리케이션은 Microsoft Graph API를 통해 Office 365 데이터에 액세스할 수 있습니다.

이제 봇 코드에서 이 연결 이름을 사용하여 사용자 토큰을 검색할 수 있습니다.

#### <a name="to-register-an-azure-ad-v2-application"></a>Azure AD v2 애플리케이션을 등록하려면

1. [Azure Portal](http://portal.azure.com/)에서 봇의 봇 채널 등록 페이지로 이동합니다.
1. **설정**을 클릭합니다.
1. 페이지 아래쪽의 **OAuth 연결 설정** 아래에서 **설정 추가**를 클릭합니다.
1. 다음과 같이 양식을 채웁니다.
    1. **이름**에 대해 연결의 이름을 입력합니다. 봇 코드에서 사용합니다.
    1. **서비스 공급자**에 대해 **Azure Active Directory v2**를 선택합니다. 이를 선택하면 Azure AD 관련 필드가 표시됩니다.
    1. **클라이언트 ID**에 대해 응용 프로그램 등록의 Azure AD v2 응용 프로그램 ID를 입력합니다.
    1. **클라이언트 비밀**에 대해 응용 프로그램 등록의 Azure AD v2 응용 프로그램 암호를 입력합니다.
    1. **테넌트 ID**에 대해 Azure Active Directory에 대한 테넌트 ID를 입력합니다(예: `microsoft.com` 또는 `common`).

        인증될 수 있는 사용자와 연결된 테넌트가 됩니다. 누구든지 봇을 통해 자신을 인증하도록 허용하려면 `common` 테넌트를 사용합니다.

    1. **범위**에 대해 응용 프로그램 등록에서 선택한 사용 권한의 이름을 입력합니다. `Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`

        > [!NOTE]
        > Azure AD v2의 경우 **범위**는 대/소문자 구분, 공백으로 구분된 값의 목록을 사용합니다.

1. **저장**을 클릭합니다.

> [!NOTE]
> 이러한 값을 사용하면 애플리케이션은 Microsoft Graph API를 통해 Office 365 데이터에 액세스할 수 있습니다.

이제 봇 코드에서 이 연결 이름을 사용하여 사용자 토큰을 검색할 수 있습니다.

#### <a name="to-test-your-connection"></a>연결을 테스트하려면

1. 방금 만든 연결을 엽니다.
1. **서비스 공급자 연결 설정** 창의 맨 위에 있는 **연결 테스트**를 클릭합니다.
1. 처음에 앱에서 요청하는 사용 권한을 나열하는 새 브라우저 탭을 열고 수락하라는 메시지를 표시해야 합니다.
1. **Accept**를 클릭합니다.
1. 이렇게 하면 **<your-connection-name>에 대한 연결 테스트가 성공했습니다** 페이지로 리디렉션됩니다.

## <a name="prepare-the-bot-sample-code"></a>봇 샘플 코드 준비

선택한 샘플에 따라 C# 또는 Node로 작업하게 됩니다.

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

사용자가 봇에 사용자 로그인이 필요한 작업 수행을 요청하는 경우 봇은 `OAuthPrompt`를 사용하여 지정된 연결에 대한 토큰 검색을 시작할 수 있습니다. `OAuthPrompt`는 다음과 같이 진행되는 토큰 검색 흐름을 만듭니다.

1. Azure Bot Service에 현재 사용자 및 연결에 대한 토큰이 이미 있는지 확인합니다. 토큰이 있을 경우 토큰이 반환됩니다.
1. Azure Bot Service에 캐시된 토큰이 없을 경우 사용자가 클릭할 수 있는 로그인 단추인 `OAuthCard`가 생성됩니다.
1. 사용자가 `OAuthCard` 로그인 단추를 클릭하면 Azure Bot Service가 사용자의 토큰을 직접 전송하거나 사용자가 6자리 인증 코드를 제공하여 채팅 창에 입력하도록 합니다.
1. 사용자에게 인증 코드가 제공되면 봇이 이 인증 코드를 사용자의 토큰으로 교환합니다.

다음의 두 코드 조각은 `OAuthPrompt`에서 가져온 것으로, 프롬프트에서 이러한 단계가 작동하는 방식을 보여줍니다.

### <a name="check-for-a-cached-token"></a>캐시된 토큰에 대한 확인

이 코드에서 봇은 먼저 Azure Bot Service에 사용자에 대한 토큰(현재 작업 전송자에 의해 식별됨) 및 지정된 ConnectionName(구성에서 사용되는 연결 이름)이 이미 있는지 빠른 검사를 수행합니다. Azure Bot Service는 캐시된 토큰이 이미 있거나 없습니다. GetUserTokenAsync에 대한 호출은 이 빠른 검사를 수행합니다. Azure Bot Service에 토큰이 있고 반환하는 경우 토큰을 즉시 사용할 수 있습니다. Azure Bot Service에 토큰이 없는 경우 이 메서드는 Null을 반환합니다. 이 경우 봇은 로그인할 사용자에 대한 사용자 지정된 OAuthCard를 보낼 수 있습니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

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

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

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

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

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

## <a name="additional-resources"></a>추가 리소스
[Bot Builder SDK](https://github.com/microsoft/botbuilder)
