---
title: Bot Framework Emulator를 사용한 봇 테스트 및 디버그 | Microsoft Docs
description: Bot Framework Emulator 데스크톱 응용 프로그램을 사용하여 봇을 검사, 테스트 및 디버그하는 방법을 살펴봅니다.
keywords: 기록, msbot 도구, 언어 서비스, 음성 인식
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/20/2018
ms.openlocfilehash: 9fb42795d789f89d0bdc74d348d50dbc0ccaa7cb
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389526"
---
# <a name="debug-with-the-emulator"></a>에뮬레이터를 사용하여 디버그

Bot Framework Emulator는 봇 개발자가 로컬 또는 원격으로 봇을 테스트 및 디버그할 수 있는 데스크톱 응용 프로그램입니다. 에뮬레이터를 사용하면 봇과 채팅하고 봇이 보내고 받는 메시지를 검사할 수 있습니다. 에뮬레이터는 봇과 메시지를 교환할 때 메시지가 웹 채팅 UI에 표시되는 대로 메시지를 표시하며 JSON 요청과 응답을 기록합니다. 봇을 클라우드에 배포하기 전에 에뮬레이터를 사용하여 로컬로 실행하고 테스트합니다. 아직 Azure Bot Service를 사용하여 봇을 [만들지](./bot-service-quickstart.md) 않았거나 채널에서 실행되도록 구성하지 않은 경우에도 에뮬레이터를 사용하여 봇을 테스트할 수 있습니다.

## <a name="prerequisites"></a>필수 조건
- [에뮬레이터](https://github.com/Microsoft/BotFramework-Emulator/releases) 설치
- [ngrok][ngrokDownload] 터널링 소프트웨어 설치

## <a name="connect-to-a-bot-running-on-localhost"></a>로컬 호스트에서 실행되는 봇에 연결

로컬로 실행되는 봇에 연결하려면 **봇 열기**를 클릭하고 .bot 파일을 선택합니다. 

![에뮬레이터 UI](media/emulator-v4/emulator-welcome.png)

## <a name="view-detailed-message-activity-with-the-inspector"></a>검사기를 사용 하여 자세한 메시지 활동 보기

봇에 메시지를 보내면 봇이 응답합니다. 대화 창에서 메시지 풍선을 클릭하고 창의 오른쪽에 있는 **검사기** 기능을 사용하여 원시 JSON 활동을 검사할 수 있습니다. 메시지 풍선을 선택하면 해당 풍선이 노란색으로 바뀌며 활동 JSO 개체가 채팅 창 왼쪽에 표시됩니다. JSON 정보에는 채널 ID, 활동 유형, 대화 ID, 텍스트 메시지, 엔드포인트 URL 등과 같은 주요 메타데이터가 포함됩니다. 사용자가 보낸 활동과 봇이 응답한 활동을 모두 살펴볼 수 있습니다. 

![에뮬레이터 메시지 활동](media/emulator-v4/emulator-view-message-activity-02.png)

## <a name="save-and-load-conversations-with-bot-transcripts"></a>봇 기록으로 대화 저장 및 로드

에뮬레이터의 활동은 기록으로 저장할 수 있습니다. 열려 있는 라이브 채팅 창에서 기록 파일에 **기록으로 저장**을 선택합니다. 언제든 **다시 시작** 단추를 사용하여 대화를 지우고 봇 연결을 다시 시작할 수 있습니다.  

![에뮬레이터 - 기록 저장 ](media/emulator-v4/emulator-live-chat.png)

기록을 로드하려면 **파일 > 기록 파일 열기**를 선택하고 기록을 선택하면 됩니다. 새 기록 창이 열리고 메시지 활동을 출력 창에 렌더링합니다. 

![에뮬레이터 - 기록 로드](media/emulator-v4/emulator-load-transcript.png)

## <a name="add-services"></a>서비스 추가 

에뮬레이터에서 직접 LUIS 앱, QnA 기술 자료 또는 디스패치 모델을 .bot 파일에 등록할 수 있습니다. .bot 파일이 로드되면 에뮬레이터 창 맨 왼쪽의 서비스 단추를 선택합니다. **서비스** 메뉴 아래에 LUIS, QnA Maker, 디스패치를 추가하는 옵션이 표시됩니다. 

서비스 앱을 추가하려면 **+** 단추를 클릭하고 추가하려는 서비스를 선택합니다. 봇 파일에 서비스를 추가하려면 Azure Portal에 로그인하라는 메시지가 표시되면 서비스를 봇 응용 프로그램에 연결합니다. 

![LUIS 연결](media/emulator-v4/emulator-connect-luis-btn.png)

서비스가 연결되면 라이브 채팅 창으로 돌아가 서비스가 연결되어 작동 중임을 확인할 수 있습니다. 

![QnA 연결됨](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-services"></a>서비스 검사

새 v4 에뮬레이터를 사용하여 LUIS 및 QnA에서 JSON 응답도 검사할 수 있습니다. 연결된 언어 서비스가 있는 봇을 사용하여 LOG 창의 오른쪽 아래에서 **추적**을 선택할 수 있습니다. 이 새로운 도구는 에뮬레이터에서 직접 언어 서비스를 업데이트하는 기능도 제공합니다. 

![LUIS 검사기](media/emulator-v4/emulator-luis-inspector.png)

연결된 LUIS 서비스를 통해 추적 링크가 **Luis 추적**을 지정하는 것을 볼 수 있습니다. 선택하면 의도, 엔터티, 지정된 점수를 포함하는 LUIS 서비스로부터의 원시 응답이 표시됩니다. 사용자 표현에 대한 의도를 다시 할당하는 옵션도 있습니다. 

![QnA 검사기](media/emulator-v4/emulator-qna-inspector.png)

연결된 QnA 서비스를 통해 로그는 **QnA 추적**을 표시하며 선택한 경우 해당 활동과 관련한 질문 및 답변 쌍을 신뢰도 점수와 함께 미리 볼 수 있습니다. 여기에서 답변에 대한 대체 질문 문구를 추가할 수 있습니다.

## <a name="configure-ngrok"></a>ngrok 구성

Windows를 사용하고 있고 방화벽 또는 다른 네트워크 경계 뒤에서 Bot Framework Emulator를 실행 중이며 원격 호스팅되는 봇에 연결하려는 경우, **ngrok** 터널링 소프트웨어를 설치하여 구성해야 합니다. Bot Framework Emulator는 ngrok 터널링 소프트웨어([inconshreveable][inconshreveable] 개발)와 긴밀히 통합되며 필요하면 자동으로 실행할 수 있습니다.

**에뮬레이터 설정**을 열고, ngrok 경로를 입력하고, 로컬 주소에 대한 ngrok 무시 여부를 선택하고, **저장**을 클릭합니다.

![ngrok 경로](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>추가 리소스

Bot Framework Emulator는 오픈 소스입니다. [버그 및 제안을 제출][EmulatorGithubBugs]하여 개발에 [기여][EmulatorGithubContribute]할 수 있습니다.



[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
