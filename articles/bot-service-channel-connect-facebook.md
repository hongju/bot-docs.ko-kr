---
title: Facebook Messenger에 봇 연결 | Microsoft Docs
description: Facebook Messenger에 대한 봇 연결을 구성하는 방법을 알아봅니다.
keywords: Facebook Messenger, 봇 채널, Facebook 앱, 앱 ID, 앱 암호, Facebook 봇, 자격 증명
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 0a9ad7d51234b417d5d0f27dbcffe4ce839ba94a
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301205"
---
# <a name="connect-a-bot-to-facebook-messenger"></a>Facebook Messenger에 봇 연결

Facebook Messenger 개발에 대한 자세한 내용은 [Messenger 플랫폼 설명서](https://developers.facebook.com/docs/messenger-platform)를 참조하세요. Facebook의 [사전 실행 지침](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public), [빠른 시작](https://developers.facebook.com/docs/messenger-platform/guides/quick-start) 및 [설치 가이드](https://developers.facebook.com/docs/messenger-platform/guides/setup)를 검토할 수 있습니다.

Facebook Messenger를 사용하여 통신하도록 봇을 구성하려면 Facebook 페이지에서 Facebook Messenger를 사용하도록 설정하고 앱에 봇을 연결합니다.

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

> [!NOTE]
> Facebook UI는 사용 중인 버전에 따라 약간 다를 수 있습니다.

## <a name="copy-the-page-id"></a>페이지 ID 복사

봇은 Facebook 페이지를 통해 액세스합니다. [새 Facebook 페이지를 만들거나](https://www.facebook.com/bookmarks/pages) 기존 페이지로 이동합니다.

* Facebook 페이지의 **About**(정보) 페이지를 열고 **페이지 ID**를 복사한 후 저장합니다.

## <a name="create-a-facebook-app"></a>Facebook 앱 만들기

페이지에서 [새 Facebook 앱을 만들고](https://developers.facebook.com/quickstarts/?platform=web) 앱 ID 및 앱 암호를 생성합니다.

![앱 ID 만들기](~/media/channels/FB-CreateAppId.png)

* **앱 ID** 및 **앱 암호**복사한 후 저장합니다.

![앱 ID 및 암호 저장](~/media/channels/FB-get-appid.png)

"Allow API Access to App Settings"(앱 설정에 대한 API 액세스 허용) 슬라이더를 "Yes"(예)로 설정합니다.

![앱 설정](~/media/bot-service-channel-connect-facebook/api_settings.png)

## <a name="enable-messenger"></a>Messenger 사용


새 Facebook 앱에서 Facebook Messenger를 사용하도록 설정합니다.

* 앱의 **Product Setup**(제품 설치) 페이지에서 **Get Started**(시작)를 클릭한 후 **Get Started**(시작)를 다시 클릭합니다.


![Messenger 사용](~/media/channels/FB-AddMessaging1.png)

## <a name="generate-a-page-access-token"></a>페이지 액세스 토큰 생성

**Token Generation**(토큰 생성) 패널에서 대상 페이지를 선택합니다. 페이지 액세스 토큰이 생성됩니다.

* **페이지 액세스 토큰**을 복사한 후 저장합니다.

![토큰 생성](~/media/channels/FB-generateToken.png)

## <a name="enable-webhooks"></a>웹후크 사용

**Set up Webhooks**(웹 후크 설정)를 클릭하여 Facebook Messenger에서 봇으로 메시징 이벤트를 전달합니다.

![웹후크 사용](~/media/channels/FB-webhook.png)

## <a name="provide-webhook-callback-url-and-verify-token"></a>웹후크 콜백 URL 제공 및 토큰 확인

[Bot Framework 포털](https://dev.botframework.com/)로 돌아갑니다. 봇을 열고 **채널** 탭을 클릭한 후 **Facebook Messenger**를 클릭합니다.

* 포털에서 **콜백 URL** 및 **토큰 확인** 값을 복사합니다.

![값 복사](~/media/channels/fb-callbackVerify.png)

1. Facebook Messenger로 돌아가서 **콜백 URL** 및 **토큰 확인** 값을 붙여넣습니다.

2. **Subscription Fields**(구독 필드) 아래에서 *message\_deliveries*, *messages*, *messaging\_options* 및 *messaging\_postbacks*를 선택합니다.

3. **Verify and Save**(확인 및 저장)를 클릭합니다.

![웹후크 구성](~/media/channels/FB-webhookConfig.png)

4. Facebook 페이지에 웹후크를 가입합니다.

![웹후크 가입](~/media/bot-service-channel-connect-facebook/subscribe-webhook.png)


## <a name="provide-facebook-credentials"></a>Facebook 자격 증명 제공

Bot Framework 포털에서 이전에 Facebook Messenger에서 복사한 **페이지 ID**, **앱 ID**, **앱 암호** 및 **페이지 액세스 토큰** 값을 붙여넣습니다.

![자격 증명 입력](~/media/channels/fb-credentials2.png)

## <a name="submit-for-review"></a>검토를 위해 제출

Facebook의 기본 앱 설정 페이지에서 개인 정보 취급 방침 URL 및 서비스 약관 URL을 제공해야 합니다. [Code of Conduct](https://aka.ms/bf-conduct)(준수 사항) 페이지에는 개인 정보 취급 방침을 만드는 데 도움이 되는 타사 리소스 링크가 포함되어 있습니다. [Terms of Use](https://aka.ms/bf-terms)(사용 약관) 페이지에는 해당 서비스 약관 문서를 만드는 데 도움이 되는 샘플 약관이 포함되어 있습니다.

봇이 완료되면 Facebook에서는 Messenger에 게시된 앱에 대한 자체 [검토 프로세스](https://developers.facebook.com/docs/messenger-platform/app-review)가 진행됩니다. 봇이 Facebook의 [플랫폼 정책](https://developers.facebook.com/docs/messenger-platform/policy-overview)을 준수하는지 테스트됩니다.

## <a name="make-the-app-public-and-publish-the-page"></a>앱을 공개로 설정하고 페이지 게시

> [!NOTE]
> 앱은 게시될 때까지 [개발 모드](https://developers.facebook.com/docs/apps/managing-development-cycle)입니다. 플러그 인 및 API 기능은 관리자, 개발자 및 테스터만 사용할 수 있습니다.

검토가 성공적으로 완료되면 앱 검토(App Review) 아래의 App Dashboard(앱 대시보드)에서 앱을 Public(공개)으로 설정합니다.
이 봇과 연결된 Facebook 페이지가 게시되는지 확인합니다. 페이지 설정에 상태가 표시됩니다.
