---
title: LINE에 봇 연결 | Microsoft Docs
description: LINE에 대한 봇 연결을 구성하는 방법을 알아봅니다.
keywords: 봇 연결, 봇 채널, LINE 봇, 자격 증명, 구성, 휴대폰
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/7/2019
ms.openlocfilehash: f6ed40c5ad3999ea5a123bf051de849917f22b42
ms.sourcegitcommit: e41dabe407fdd7e6b1d6b6bf19bef5f7aae36e61
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/27/2019
ms.locfileid: "56893513"
---
# <a name="connect-a-bot-to-line"></a>LINE에 봇 연결

LINE 앱을 통해 사용자와 통신하도록 봇을 구성할 수 있습니다.

## <a name="log-into-the-line-console"></a>LINE 콘솔에 로그인

*Line에 로그인*을 사용하여 LINE 계정의 [LINE 개발자 콘솔](https://developers.line.biz/console/register/messaging-api/provider/)에 로그인합니다. 

> [!NOTE]
> 아직 다운로드하지 않았다면 [LINE을 다운로드](https://line.me/)한 후 설정으로 가서 이메일 주소를 등록합니다.

### <a name="register-as-a-developer"></a>개발자로 등록

LINE 개발자 콘솔에 처음으로 로그인하는 경우 이름과 이메일 주소를 입력하여 개발자 계정을 만드세요.

![LINE 스크린샷 개발자 등록](./media/channels/LINE-screenshot-1.png)

## <a name="create-a-new-provider"></a>새 공급자 만들기

봇의 공급자를 아직 설정하지 않았다면 우선 공급자를 만드세요. 공급자는 앱을 제공하는 독립체(개인 또는 회사)입니다.

![LINE 스크린샷 공급자 만들기](./media/channels/LINE-screenshot-2.png)

## <a name="create-a-messaging-api-channel"></a>메시징 API 채널 만들기

다음으로, 새로운 메시징 API 채널을 만듭니다. 

![LINE 스크린샷 채널 유형](./media/channels/LINE-channel-type-selection.png)

녹색 사각형을 클릭하여 새로운 메시징 API 채널을 만듭니다.

![LINE 스크린샷 채널 만들기](./media/channels/LINE-create-channel.png)

이름에는 "LINE" 또는 이와 유사한 문자열을 포함할 수 없습니다. 필수 필드를 채우고 채널 설정을 확인합니다.

![LINE 스크린샷 채널 설정](./media/channels/LINE-screenshot-4.png)

## <a name="get-necessary-values-from-your-channel-settings"></a>채널 설정에서 필요한 값 가져오기

채널 설정을 확인하면 다음과 유사한 페이지로 이동합니다.

![LINE 스크린샷 채널 페이지](./media/channels/LINE-screenshot-5.png)

생성된 채널을 클릭하여 채널 설정에 액세스하고 아래로 스크롤하여 **기본 정보 > 채널 비밀**을 찾습니다. 이 암호를 임시로 저장합니다. 사용 가능한 기능에 `PUSH_MESSAGE`가 포함되어 있는지 확인합니다.

![LINE 스크린샷 채널 비밀](./media/channels/LINE-screenshot-6.png)

그런 다음, **메시징 설정**까지 더 스크롤합니다. 여기서 **채널 액세스 토큰** 필드와 *실행* 단추를 볼 수 있습니다. 해당 단추를 클릭하여 액세스 토큰을 가져오고, 마찬가지로 임시로 저장합니다.

![LINE 스크린샷 채널 토큰](./media/channels/LINE-screenshot-8.png)

## <a name="connect-your-line-channel-to-your-azure-bot"></a>Azure 봇에 LINE 채널 연결

[Azure Portal](https://portal.azure.com/)에 로그인하고 봇을 찾은 후 **채널**을 클릭합니다. 

![LINE 스크린샷 azure 설정](./media/channels/LINE-channel-setting-2.png)

여기서 LINE 채널을 선택하고 위의 채널 비밀 및 액세스 토큰을 적절한 필드에 붙여넣습니다. 변경 내용을 저장합니다.

Azure에서 제공하는 사용자 지정 웹후크 URL을 복사합니다.

![LINE 스크린샷 azure 설정](./media/channels/LINE-channel-setting-1.png)

## <a name="configure-line-webhook-settings"></a>LINE 웹후크 설정 구성

다음으로, LINE 개발자 콘솔로 돌아가서 Azure의 웹후크 URL을 **메시지 설정 > 웹후크 URL**에 붙여넣고 **확인**을 클릭하여 연결을 확인합니다. Azure에서 채널을 방금 만든 경우 적용되는 데 몇 분 정도 걸릴 수 있습니다.

그런 다음, **메시지 설정 > 웹후크 사용**을 설정합니다.

> [!IMPORTANT]
> LINE 개발자 콘솔에서 웹후크 URL을 먼저 설정한 후에만 **웹후크 사용 = 사용**을 설정합니다. 빈 URL로 웹후크를 사용하도록 설정하면 UI에는 사용 상태로 표시되더라도 사용 상태가 설정되지 않습니다.

웹후크 URL을 추가한 후 웹후크를 사용하도록 설정했으면 이 페이지를 다시 로드하고 이러한 변경 내용이 올바르게 설정되었는지 확인하세요.

![LINE 스크린샷 웹후크](./media/channels/LINE-screenshot-9.png)

## <a name="test-your-bot"></a>봇 테스트

이러한 단계를 완료한 경우, 봇은 LINE에서 사용자와 통신하도록 성공적으로 구성되며 테스트할 준비가 된 것입니다.

### <a name="add-your-bot-to-your-line-mobile-app"></a>LINE 모바일 앱에 봇 추가

LINE 개발자 콘솔에서 설정 페이지로 이동하면 봇의 QR 코드가 표시됩니다. 

모바일 LINE 앱에서 맨 오른쪽의 점 세 개([**...**])가 있는 탐색 탭으로 이동하고 QR 코드 아이콘을 탭합니다. 

![LINE 스크린샷 모바일 앱](./media/channels/LINE-screenshot-12.jpg)

QR 코드 판독기로 개발자 콘솔의 QR 코드를 가리킵니다. 이제 모바일 LINE 앱에서 봇과 상호 작용하고 봇을 테스트할 수 있을 것입니다.

### <a name="automatic-messages"></a>자동 메시지

봇 테스트를 시작한 후 봇이 `conversationUpdate` 활동에 지정된 것과 다른 예상치 못한 메시지를 보내는 경우가 있습니다.  다음과 같은 대화가 표시될 수 있습니다.

![LINE 스크린샷 대화](./media/channels/LINE-screenshot-conversation.jpg)

이러한 메시지를 보내지 않도록 하려면 자동 응답 메시지를 해제해야 합니다.

![LINE 스크린샷 자동 응답](./media/channels/LINE-screenshot-10.png)

또는 이러한 메시지를 유지하도록 선택할 수 있습니다. 이 경우 "메시지 설정"을 클릭하고 편집하는 것이 좋습니다.

![LINE 스크린샷 자동 응답 설정](./media/channels/LINE-screenshot-11.png)

## <a name="troubleshooting"></a>문제 해결

* 봇이 메시지에 전혀 응답하지 않는 경우 Azure Portal에서 봇으로 이동하고 웹 채팅으로 테스트를 선택합니다.  
    * 봇이 웹 채팅에서는 작동하지만 LINE에서는 응답하지 않을 경우 LINE 개발자 콘솔 페이지를 다시 로드하고 위의 웹후크 지침을 반복하세요. 웹후크를 사용하도록 설정하기 전에 **웹후크 URL**을 설정하도록 하세요.
    * 웹 채팅에서 봇이 작동하지 않을 경우 봇의 문제를 디버그한 후 다시 돌아와서 LINE 채널 구성을 완료하세요.

