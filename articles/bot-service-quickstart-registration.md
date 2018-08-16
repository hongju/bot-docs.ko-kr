---
title: Bot Service를 사용하여 봇 채널 등록 만들기 | Microsoft Docs
description: Bot Service를 사용하여 기존 bot을 등록하는 방법을 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6a32bc5712937c615962e4f6edfc7ea691d3ec39
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574979"
---
# <a name="register-a-bot-with-bot-service"></a>Bot Service를 사용하여 봇 등록

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

이미 다른 곳에서 봇을 호스트했으며 Bot Service를 사용하여 다른 채널에 연결하려는 경우, Bot Service에 봇을 등록해야 합니다. 이 항목에서는 **봇 채널 등록** Bot Service를 사용하여 Bot Service에 봇을 등록하는 방법을 알아봅니다.

> [!IMPORTANT] 
> Azure에서 호스트되지 않은 경우에만 봇을 등록할 필요가 있습니다. Azure Portal을 통해 [봇을 만든 경우](bot-service-quickstart.md) 봇이 Bot Service에 이미 등록되어 있을 것입니다.

## <a name="log-in-to-azure"></a>Azure에 로그인
[Azure Portal](http://portal.azure.com)에 로그인합니다.

> [!TIP]
> 아직 구독이 없는 경우 <a href="https://azure.microsoft.com/en-us/free/" target="_blank">체험 계정</a>으로 등록할 수 있습니다.

## <a name="create-a-bot-channels-registration"></a>봇 채널 등록 만들기
Bot Service 기능을 사용할 수 있으려면 **봇 채널 등록** Bot Service가 필요합니다. 등록 봇ㅇ르 사용하여 채널에 봇을 연결할 수 있습니다.

**봇 채널 등록**을 만들려면 다음을 수행합니다.

1. Azure Portal의 왼쪽 위 모서리에 있는 **새로 만들기** 단추를 클릭한 다음, **AI + Cognitive Services > 봇 채널 등록**을 선택합니다. 

2. **봇 채널 등록**에 대한 정보가 포함된 새 블레이드가 열립니다. **만들기** 단추를 클릭하여 만들기 프로세스를 시작합니다. 

3. **Bot Service** 블레이드에서 이미지 아래 표에 지정된 대로 요청된 봇 정보를 제공합니다.  <br/>
   ![등록 봇 만들기 블레이드](~/media/azure-bot-quickstarts/registration-create-bot-service-blade.png)


   |                    설정                     |         제안 값         |                                                                                                  설명                                                                                                  |
   |------------------------------------------------|---------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |           <strong>봇 이름</strong>            |     봇의 표시 이름     |                                                  채널 및 디렉터리에 표시되는 봇의 표시 이름입니다. 이 이름은 언제든지 변경할 수 있습니다.                                                  |
   |         <strong>구독</strong>          |        사용자의 구독        |                                                                                사용할 Azure 구독을 선택합니다.                                                                                 |
   |        <strong>리소스 그룹</strong>         |         myResourceGroup         |                                 새 [리소스 그룹](/azure/azure-resource-manager/resource-group-overview#resource-groups)을 만들거나 기존 리소스 그룹을 선택할 수 있습니다.                                  |
   |                    위치                    |             미국 서부             |                                                        봇이 배포된 위치에 가까운 위치나 봇이 액세스할 기타 서비스에 가까운 위치를 선택합니다.                                                         |
   |         <strong>가격 책정 계층</strong>          |               F0                |             가격 책정 계층을 선택합니다. 언제든지 가격 책정 계층을 업데이트할 수 있습니다. 자세한 내용은 [Bot Service 가격](https://azure.microsoft.com/en-us/pricing/details/bot-service/)을 참조하세요.              |
   |      <strong>메시징 끝점</strong>       |               URL               |                                                                               봇 메시징 끝점의 URL을 입력합니다.                                                                                |
   |     <strong>Application Insights</strong>      |               다른                | [Application Insights](bot-service-manage-analytics.md)를 <strong>켜기</strong> 또는 <strong>끄기</strong>로 설정할지 결정합니다. <strong>켜기</strong>를 선택하면 지역 위치를 지정해야 합니다. |
   | <strong>Microsoft 앱 ID 및 암호</strong> | 자동 앱 ID 및 암호 만들기 |              Microsoft 앱 ID 및 암호를 수동으로 입력해야 하는 경우 이 옵션을 사용합니다. 그렇지 않으면 봇 만들기 프로세스에서 새 Microsoft 앱 ID 및 암호가 만들어집니다.               |


4. **만들기**를 클릭하여 서비스를 만들고 봇의 메시징 끝점을 등록합니다.

**알림**을 선택하여 등록이 만들어졌는지 확인합니다. 알림이 **배포 진행 중...** 에서 **배포 성공**으로 변경됩니다. **리소스로 이동** 단추를 클릭하여 봇의 리소스 블레이드를 엽니다. 

## <a name="bot-channels-registration-password"></a>봇 채널 등록 암호

**봇 채널 등록** Bot Service에는 연결된 App Service가 없습니다. 따라서 이 Bot Service에는 *MicrosoftAppID*만 있습니다. 암호를 수동으로 생성하고 직접 저장해야 합니다. [에뮬레이터](bot-service-debug-emulator.md)를 사용하여 봇을 테스트하려는 경우 이 암호가 필요합니다.

MicrosoftAppPassword를 생성하려면 다음을 수행합니다.

1. **설정** 블레이드에서 **관리**를 클릭합니다. 이것은 **Microsoft 앱 ID**로 표시되는 링크입니다. 이 링크를 따라가면 새 암호를 생성할 수 있는 창이 열립니다. <br/>
  ![설정 블레이드에서 링크 관리](~/media/azure-bot-quickstarts/registration-settings-manage-link.png)

2. **새 암호 생성**을 클릭합니다. 이렇게 하면 봇에 대한 새 암호가 생성됩니다. 이 암호를 복사하고 파일로 저장합니다. 이 암호는 이때만 볼 수 있습니다. 전체 암호를 저장하지는 않은 경우, 나중에 암호가 필요할 것으로 예상되면 이 프로세스를 반복하여 새 암호를 만들어야 합니다. <br/>
  ![Microsoft 앱 암호 생성](~/media/azure-bot-quickstarts/registration-generate-app-password.png)

## <a name="update-the-bot"></a>봇 업데이트

Node.js용 Bot Builder SDK를 사용하는 경우 다음과 같은 환경 변수를 설정합니다.

* MICROSOFT_APP_ID
* MICROSOFT_APP_PASSWORD

.NET용 Bot Builder SDK를 사용 중인 경우 web.config 파일에 다음 키 값을 설정합니다.

* MicrosoftAppId
* MicrosoftAppPassword

## <a name="test-the-bot"></a>봇 테스트

이제 Bot Service를 만들었으므로 [웹 채팅](bot-service-manage-test-webchat.md)에서 테스트합니다. 메시지를 입력하면 봇이 응답합니다.

## <a name="next-steps"></a>다음 단계

이 항목에서는 호스트된 봇을 Bot Service에 등록하는 방법을 알아보았습니다. 다음 단계로, Bot Service를 관리하는 방법을 알아봅니다.

> [!div class="nextstepaction"]
> [봇 관리](bot-service-manage-overview.md)

