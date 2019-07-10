---
title: Cortana 스킬 테스트 | Microsoft Docs
description: Cortana 스킬을 호출하여 Cortana 봇을 테스트하는 방법을 알아봅니다.
keywords: Bot Framework SDK, 봇 등록, cortana
author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/01/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7d07317afb6d89c2d22d6f4983f7b21c3a1a053c
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496672"
---
# <a name="test-a-cortana-skill"></a>Cortana 스킬 테스트

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]
 
Bot Framework SDK를 사용하여 Cortana 스킬을 빌드한 경우 Cortana에서 호출하여 테스트할 수 있습니다. 다음 지침에서는 Cortana 스킬을 사용해보는 데 필요한 단계를 안내합니다.

## <a name="register-your-bot"></a>봇 등록
Azure에서 Bot Service를 사용하여 [봇을 만든 경우](~/bot-service-quickstart.md) 봇을 이미 등록했을 것이므로 이 단계를 건너뛰어도 됩니다.

다른 곳에서 봇을 배포했거나 봇을 로컬로 테스트하려는 경우 Cortana에 연결할 수 있도록 봇을 [등록](bot-service-quickstart-registration.md)해야 합니다. 등록 프로세스 동안 봇의 **메시징 끝점**을 제공해야 합니다. 봇을 로컬로 테스트하려는 경우 [ngrok](http://ngrok.com) 같은 터널링 소프트웨어를 실행하여 로컬 봇의 끝점을 가져와야 합니다.

## <a name="get-messaging-endpoint-using-ngrok"></a>ngrok을 사용하여 메시징 끝점 가져오기

봇을 로컬로 실행 중인 경우 [ngrok](https://ngrok.com) 같은 터널링 소프트웨어를 실행하여 테스트에 사용할 끝점을 가져올 수 있습니다. ngrok을 사용하여 끝점을 가져오려면 콘솔 창에서 다음을 입력합니다. 

```cmd
 ngrok.exe http 3979 -host-header="localhost:3979"
``` 

이렇게 하면 포트 3978에서 실행되는 봇으로 요청을 전달하는 ngrok 전달 링크가 구성 및 표시됩니다. 전달 링크의 URL은 `https://0d6c4024.ngrok.io`와 유사합니다.  링크에 `/api/messages`를 추가하여 메시징 끝점 URL을 `https://0d6c4024.ngrok.io/api/messages` 형식으로 만듭니다. 

봇의 [설정](~/bot-service-manage-settings.md) 블레이드에 있는 **구성** 섹션에 이 끝점 URL을 입력합니다.

## <a name="enable-speech-recognition-priming"></a>음성 인식 초기화 사용
봇이 LUIS(Language Understanding) 앱을 사용하는 경우 LUIS 애플리케이션 ID를 등록된 Bot Service에 연결해야 합니다. 이렇게 하면 봇이 LUIS 모델에 정의된 음성 발언을 이해하는 데 도움이 됩니다. 자세한 내용은 [음성 초기화](~/bot-service-manage-speech-priming.md)를 참조하세요.

## <a name="add-the-cortana-channel"></a>Cortana 채널 추가
Cortana를 채널로 추가하려면 [Cortana에 봇 연결](bot-service-channel-connect-cortana.md)에 나열된 단계를 따르세요.

## <a name="test-using-web-chat-control"></a>웹 채팅 컨트롤을 사용하여 테스트

Bot Service에서 통합된 웹 채팅 컨트롤을 사용하여 봇을 테스트하려면 **웹 채팅에서 테스트**를 클릭하고 메시지를 입력하여 봇이 작동하는지 확인합니다.

## <a name="test-using-emulator"></a>에뮬레이터를 사용하여 테스트

[에뮬레이터](~/bot-service-debug-emulator.md)를 사용하여 봇을 테스트하려면 다음을 수행합니다.

1. 봇을 실행합니다.
2. 에뮬레이터를 열고 필요한 정보를 입력합니다. 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요. 
3. **연결**을 클릭하여 봇에 에뮬레이터를 연결합니다.
4. 메시지를 입력하여 봇이 작동하는지 확인합니다.

## <a name="test-using-cortana"></a>Cortana를 사용하여 테스트
Cortana에 호출 구를 말하여 Cortana 스킬을 호출할 수 있습니다. 
1. Cortana를 엽니다.
2. Cortana 내에서 노트를 열고 **내 정보**를 클릭하여 Cortana에서 사용 중인 계정을 확인합니다. 봇을 등록하는 데 사용한 동일한 Microsoft 계정으로 로그인했는지 확인합니다. 
   ![Cortana의 노트에 로그인](~/media/cortana/cortana-notebook.png)
2. Cortana 앱의 마이크 단추 또는 Windows의 “무엇이든지 물어보세요” 검색 상자를 클릭하고 봇의 [호출 구][InvocationNameGuidelines]를 말합니다. 호출 구에는 호출할 스킬을 고유하게 식별하는 *호출 이름*이 포함됩니다. 예를 들어, 스킬의 호출 이름이 "Northwind Photo"인 경우 적절한 호출 구에는 "Ask Northwind Photo to..." 또는 "Tell Northwind Photo that..."이 포함될 수 있습니다.

   Cortana용으로 구성한 경우 봇의 *호출 이름*을 지정합니다.
   ![Cortana 채널을 구성할 때 호출 이름 입력](~/media/cortana/cortana-invocation-name-callout.png)

3. Cortana가 호출 구를 인식하는 경우 Cortana의 캔버스에서 봇이 시작됩니다. 

## <a name="troubleshoot"></a>문제 해결

Cortana 스킬이 시작되지 않으면 다음을 확인합니다.
* Bot Framework 포털에 봇을 등록하는 데 사용한 동일한 Microsoft 계정을 사용하여 Cortana에 로그인되어 있는지 확인합니다.
* **웹 채팅에서 테스트**를 클릭하여 **채팅** 창을 열고 메시지를 입력하여 봇이 작동하는지 확인합니다.
* 호출 이름이 [지침][InvocationNameGuidelines]을 충족하는지 확인합니다. 호출 이름이 세 단어보다 길거나, 발음하기 어렵거나, 다른 단어처럼 들리면 Cortana가 인식하는 데 문제가 있을 수 있습니다.
* 스킬이 LUIS 모델을 사용하는 경우 [음성 인식 초기화를 사용하도록 설정](~/bot-service-manage-speech-priming.md)해야 합니다.

추가 문제 해결 팁과 Cortana 대시보드에서 기술 디버깅을 사용하도록 설정하는 방법에 대한 자세한 내용은 [Cortana 기술 디버깅 사용][Cortana-TestBestPractice]을 참조하세요. 


## <a name="next-steps"></a>다음 단계

Cortana 스킬을 테스트했으며 원하는 방식대로 작동하는지 확인했으면 베타 테스터 그룹에 배포하거나 공개적으로 릴리스할 수 있습니다. 자세한 내용은 [Cortana 기술 게시][Cortana-Publish]를 참조하세요.

## <a name="additional-resources"></a>추가 리소스
* [Cortana 기술 키트][CortanaGetStarted]

[CortanaGetStarted]: /cortana/getstarted

[BFPortal]: https://dev.botframework.com/
[CortanaDevCenter]: https://developer.microsoft.com/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines 


[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice
[Cortana-Publish]: /cortana/skills/publish-skill
