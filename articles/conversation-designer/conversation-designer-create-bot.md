---
title: 대화 디자이너 봇 만들기 | Microsoft Docs
description: 대화 디자이너를 사용하여 새 봇을 만드는 방법을 알아봅니다.
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 1b33d08e56bf8ae473b28cf1cf3a26d8138ce861
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301502"
---
# <a name="create-a-new-conversation-designer-bot"></a>새 대화 디자이너 봇 만들기
> [!IMPORTANT]
> 대화 디자이너는 일부 고객에게 제공되지 않습니다. 대화 디자이너의 가용성에 대한 자세한 내용은 올해 하반기에 제공됩니다.

이 자습서에서는 새 대화 디자이너 봇을 만들기 위한 단계별 지침을 안내합니다. 

## <a name="prerequisites"></a>필수 조건

- 대화 디자이너에는 Azure 구독이 필요합니다. <a href="https://azure.microsoft.com/en-us/" target="_blank">여기</a>에서 시작하면 됩니다.
- 아직 수행하지 않았다면 이 기능을 사용하여 계정을 만든 후에 적어도 한 번 [LUIS 포털](https://luis.ai)에 로그인해야 합니다.
- JavaScript 프로그래밍에 익숙해야 합니다. 사용자 지정 스크립트 함수는 JavaScript로 작성됩니다.
- Microsoft Edge 또는 Google Chrome

## <a name="create-a-conversation-designer-bot"></a>디자이너 대화 봇 만들기

디자이너 대화 봇을 만들려면 다음 단계를 수행합니다.
1. https://dev.botframework.com/으로 가서 로그인합니다. Microsoft Corporation에 제공한 이메일 주소를 사용하여 이 비공개 미리 보기 릴리스에서 참여합니다.
2. 오른쪽 상단 탐색 패널에서 **봇 만들기**를 클릭합니다. 
3. **만들기**를 클릭하여 *대화 디자이너를 사용하여 봇을 만듭니다*.
4. 여러 [샘플 봇](conversation-designer-sample-bots.md) 중에서 시작할 봇을 하나 선택합니다. **다음**을 클릭합니다. 사용할 **샘플 봇**을 잘 모를 경우 빌드하려는 봇에 가장 가깝다고 생각되는 봇을 선택합니다. 나중에 다른 **샘플 봇**으로 전환할 수 있습니다.
5. 모든 필드를 완료하고 **봇 만들기**를 클릭합니다. 봇 프로비전이 완료되는 데 약 2분이 소요됩니다. 

## <a name="bot-provisioning"></a>봇 프로비전

다음 Azure 기능은 대화 디자이너 봇을 만들 때 자동으로 프로비전됩니다. 

1. 지정한 봇 이름을 갖는 Azure 리소스 그룹
2. Azure App Service
3. Azure App Service 계획 
4. Azure Storage 계정
5. Application Insights 
6. [LUIS.ai](https://luis.ai)에 대한 Cognitive Services 구독입니다. 앱 이름으로 **Bot handle**(및 임의로 생성된 문자열)을 사용하여 LUIS 앱이 만들어집니다.
7. Microsoft 계정 단일 앱입니다. [자세히 알아보기](https://apps.dev.microsoft.com/#/appList)

## <a name="welcome-message"></a>환영 메시지

봇이 프로비전되면 대화 디자이너에서 봇의 **빌드** 페이지가 열립니다. 시작하는 데 도움이 되는 정보를 포함하는 환영 메시지가 표시됩니다. 이러한 옵션을 탐색하거나 메시지를 닫고 봇 작동을 시작합니다. 다시 가져올 수 있습니다 시작 메시지에 있는 왼쪽 상단 탐색 패널에서 줄임표(**...**)를 클릭한 후 **시작..**  옵션을 선택하여 환영 메시지로 돌아갈 수 있습니다.

## <a name="next-step"></a>다음 단계
> [!div class="nextstepaction"]
> [봇 저장](conversation-designer-save-bot.md)

## <a name="additional-resources"></a>추가 리소스
* [태스크](conversation-designer-tasks.md)에 대해 자세히 알아보기
* [Node용 Bot Builder SDK](../nodejs/index.md)에 대해 자세히 알아보기 
