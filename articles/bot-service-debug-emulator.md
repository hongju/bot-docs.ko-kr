---
title: Bot Framework Emulator를 사용한 봇 테스트 및 디버그 | Microsoft Docs
description: Bot Framework Emulator 데스크톱 응용 프로그램을 사용하여 봇을 검사, 테스트 및 디버그하는 방법을 살펴봅니다.
keywords: 기록, msbot 도구, 언어 서비스, 음성 인식
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/30/2018
ms.openlocfilehash: 2a92281e9bbee09d8dfb00fbbc67fe6ac6b6c69b
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756517"
---
# <a name="debug-with-the-bot-framework-emulator"></a>Bot Framework Emulator를 사용하여 디버그

Bot Framework Emulator는 봇 개발자가 로컬 또는 원격으로 봇을 테스트 및 디버그할 수 있는 데스크톱 응용 프로그램입니다. 에뮬레이터를 사용하면 봇과 채팅하고 봇이 보내고 받는 메시지를 검사할 수 있습니다. 에뮬레이터는 봇과 메시지를 교환할 때 메시지가 웹 채팅 UI에 표시되는 대로 메시지를 표시하며 JSON 요청과 응답을 기록합니다. 

> [!TIP] 
> 봇을 클라우드에 배포하기 전에 에뮬레이터를 사용하여 로컬로 실행하고 테스트합니다. 아직 Bot Framework에 [등록](~/bot-service-quickstart-registration.md)하거나 채널에서 실행하도록 구성하지 않은 경우에도 에뮬레이터로 봇을 테스트할 수 있습니다.

![에뮬레이터 UI](media/emulator-v4/emulator-welcome.png)

## <a name="download-the-bot-framework-emulator"></a>Bot Framework Emulator 다운로드

Mac, Windows 및 Linux에 대한 다운로드 패키지는 [GitHub 릴리스 페이지](https://github.com/Microsoft/BotFramework-Emulator/releases)를 통해 사용할 수 있습니다.

## <a name="connect-to-a-bot-running-on-local-host"></a>로컬 호스트에서 실행되는 봇에 연결

![에뮬레이터 엔드포인트](media/emulator-v4/emulator-endpoint.png)

로컬로 실행되는 봇에 연결하려면 왼쪽 위의 봇 탐색기 탭을 선택합니다. **엔드포인트** 탭에서 **+** 아이콘을 클릭합니다. 여기서 엔드포인트를 봇이 연결을 위해 로컬로 실행되는 것과 같은 포트로 지정할 수 있습니다. **제출**을 클릭하면 봇과 상호 작용할 수 있는 라이브 채팅 창으로 이동합니다.

## <a name="view-detailed-message-activity-with-the-inspector"></a>검사기를 사용 하여 자세한 메시지 활동 보기

![에뮬레이터 메시지 활동](media/emulator-v4/emulator-view-message-activity-02.png)

대화 창에서 아무 메시지 풍선이나 클릭하고 창의 오른쪽에 있는 **검사기** 기능을 사용하여 원시 JSON 활동을 검사할 수 있습니다. 메시지 풍선을 선택하면 해당 풍선이 노란색으로 바뀌며 활동 JSO 개체가 채팅 창 왼쪽에 표시됩니다. 이 JSON 정보에는 채널 ID, 활동 유형, 대화 ID, 텍스트 메시지, 엔드포인트 URL 등과 같은 주요 메타데이터가 포함됩니다. 사용자가 보낸 활동과 봇이 응답한 활동을 모두 검사할 수 있습니다. 

[채널 검사기](bot-service-channel-inspector.md)를 사용하여 특정 채널의 지원되는 기능을 미리 볼 수도 있습니다.


## <a name="save-and-load-conversations-with-bot-transcripts"></a>봇 기록으로 대화 저장 및 로드

에뮬레이터의 메시지 활동은 기록으로 저장할 수 있습니다. 열려 있는 라이브 채팅 창에서 **기록으로 저장**을 선택하여 출력 기록 파일의 이름을 지정하고 위치를 선택합니다. 

>[!TIP]
> 언제든 **다시 시작** 단추를 사용하여 대화를 지우고 봇 연결을 다시 시작할 수 있습니다.  

![에뮬레이터 - 기록 저장 ](media/emulator-v4/emulator-live-chat.png)

기록을 로드하려면 **파일** --> **기록 파일 열기**, 기록을 선택하기만 하면 됩니다. 새 기록 창이 열리고 메시지 활동을 출력 창에 렌더링합니다. 

![에뮬레이터 - 기록 로드](media/emulator-v4/emulator-load-transcript.png)

## <a name="author-transcripts-with-chatdown"></a>Chatdown으로 기록 작성

[Chatdown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown) 도구는 [markdown](https://daringfireball.net/projects/markdown/syntax) 파일을 사용하여 활동 기록을 생성하는 기록 생성기입니다. Markdown 형식으로 자체 기록을 완전히 작성하고 **.chat** 파일로 저장하여 기록을 생성할 수 있습니다. 이것은 봇 개발 중에 모의 대화 시나리오를 신속하게 생성할 때 유용합니다.  

### <a name="prerequisites"></a>필수 조건

- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) v.4 이상 
- [Node.js](https://nodejs.org/en/)
 
Chatdown은 Node.js가 필요한 npm 모듈로 제공됩니다. Chatdown을 설치하려면 머신에 글로벌로 설치합니다. 

```
npm install -g chatdown
```
### <a name="create-and-load-transcript-transcript-files"></a>기록 Transcript 파일 만들기 및 로드 ###

다음은 **.chat** 파일을 작성하는 방법의 예제입니다. 이 파일은 두 부분을 포함하는 Markdown입니다.
- 대화 참가자(사용자, 봇)를 정의하는 헤더
- 참가자 간의 전후 대화

```
user=John Doe
bot=Bot

bot: Hello!
user: hey
bot: [Typing][Delay=3000]
What can I do for you?
user: Actually nevermind, goodbye.
bot: bye!
```
[여기를 클릭](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown/Examples)하면 .chat 파일의 더 많은 샘플을 볼 수 있습니다. 

기록을 생성하려면 CLI에서 **chatdown** 명령을 사용하고 .chat 파일의 이름과 '>', 출력 기록 파일의 이름을 입력합니다. 

```
chatdown sample.chat > sample.transcript
```
## <a name="manage-bot-resources-with-the-msbot-tool"></a>MSBot 도구를 사용한 봇 리소스 관리

새로운 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) 도구를 사용하면 봇이 사용하는 다양한 서비스에 대한 메타데이터를 모두 한곳에 저장하는 **.bot** 파일을 만들 수 있습니다. 이 파일을 사용하면 봇이 CLI로부터 이러한 서비스에 연결할 수도 있습니다. 이 도구는 npm 모듈로 제공되며 설치하려면 다음을 실행합니다.

```
npm install -g msbot 
```
![MSBot CLI 창](media/emulator-v4/msbot-cli-window.png)


봇 파일을 만들려면 CLI에서 **msbot init**, 봇 이름, 대상 URL 엔드포인트를 차례로 입력합니다. 예를 들면 다음과 같습니다.

```shell
msbot init --name az-cli-bot --endpoint http://localhost:3984/api/messages
```
![Botfile](media/emulator-v4/botfile-generated.png)

>**참고:** 이 가이드에 사용되는 봇은 간단한 에코 봇으로, Azure CLI 봇 확장을 통해 생성되었습니다. [여기를 클릭](https://github.com/Microsoft/botbuilder-tools/tree/master/AzureCli)하면 Azure CLI를 통한 봇 빌드에 대해 자세히 볼 수 있습니다. 

이제 .bot 파일을 통해 에뮬레이터에 봇을 간편하게 로드할 수 있습니다. .bot 파일은 봇에 다양한 엔드포인트와 언어 구성 요소를 등록하는 데도 필요합니다. 

![봇-파일-드롭다운](media/emulator-v4/bot-file-dropdown.png)

## <a name="add-language-services"></a>언어 서비스 추가 

에뮬레이터에서 직접 .bot 파일에 LUIS 앱이나 OnA 기술 자료를 간편하게 등록할 수 있습니다. .bot 파일이 로드되면 에뮬레이터 창 맨 왼쪽의 서비스 단추를 선택합니다. **서비스** 메뉴 아래에 LUIS, QnA, Maker, Dispatch, 엔드포인트 및 Azure Bot Service 추가를 위한 옵션이 표시됩니다. 

LUIS 앱을 추가하려면 간단히 LUIS 메뉴에서 **+** 단추를 클릭하고 LUIS 앱 자격 증명을 입력한 다음, **제출**을 클릭합니다. 그러면 .bot 파일에 LUIS 응용 프로그램이 등록되고 서비스를 봇 응용 프로그램에 연결합니다. 

![LUIS 연결](media/emulator-v4/emulator-connect-luis-btn.png)

마찬가지로 QnA 기술 자료를 추가하려면 간단히 QnA 메뉴에서 **+** 단추를 클릭하고 QnA Maker 기술 자료 자격 증명을 입력한 다음, **제출**을 클릭합니다. 이제 기술 자료가 .bot 파일에 등록되어 사용할 수 있습니다. 

![QnA 연결](media/emulator-v4/emulator-connect-qna-btn.png)

서비스가 연결되면 라이브 채팅 창으로 돌아가 서비스가 연결되어 작동 중임을 확인할 수 있습니다. 

![QnA 연결됨](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-language-services"></a>언어 서비스 검사

새 v4 에뮬레이터를 사용하여 LUIS 및 QnA에서 JSON 응답도 검사할 수 있습니다. 연결된 언어 서비스가 있는 봇을 사용하여 LOG 창의 오른쪽 아래에서 **추적**을 선택할 수 있습니다. 이 새로운 도구는 에뮬레이터에서 직접 언어 서비스를 업데이트하는 기능도 제공합니다. 

![LUIS 검사기](media/emulator-v4/emulator-luis-inspector.png)

연결된 LUIS 서비스를 통해 추적 링크가 **Luis 추적**을 지정하는 것을 볼 수 있습니다. 선택하면 의도, 엔터티, 지정된 점수를 포함하는 LUIS 서비스로부터의 원시 응답이 표시됩니다. 사용자 표현에 대한 의도를 다시 할당하는 옵션도 있습니다. 

![QnA 검사기](media/emulator-v4/emulator-qna-inspector.png)

연결된 QnA 서비스를 통해 로그는 **QnA 추적**을 표시하며 선택한 경우 해당 활동과 관련한 질문 및 답변 쌍을 신뢰도 점수와 함께 미리 볼 수 있습니다. 여기에서 답변에 대한 대체 질문 문구를 추가할 수 있습니다.

[!TIP]
> 이러한 기능은 v4 SDK 봇에서만 제공됩니다.  


## <a name="speech-recognition"></a>음성 인식
The Bot Framework Emulator는 [Cognitive Services Speech API](/azure/cognitive-services/Speech/home)를 통한 음성 인식을 지원합니다. 따라서 개발하는 동안 에뮬레이터에서 음성을 통해 Cortana 기술 또는 음성 지원 봇을 연습할 수 있습니다. Bot Framework Emulator는 하루에 봇당 최대 3시간의 무료 음성 인식을 제공합니다. 

## <a id="ngrok"></a> ngrok 설치 및 구성

Windows를 사용하고 있고 방화벽 또는 다른 네트워크 경계 뒤에서 Bot Framework Emulator를 실행 중이며 원격 호스팅되는 봇에 연결하려는 경우, **ngrok** 터널링 소프트웨어를 설치하여 구성해야 합니다. Bot Framework Emulator는 [ngrok][ngrokDownload] 터널링 소프트웨어([inconshreveable][inconshreveable] 개발)와 긴밀히 통합되며 필요하면 자동으로 실행할 수 있습니다.

**ngrok**를 Windows에 설치하고 에뮬레이터가 이를 사용하게 하려면 다음 단계를 완료합니다. 

1. [ngrok][ngrokDownload] 실행 파일을 로컬 머신에 다운로드합니다.

2. 에뮬레이터 앱 설정 대화 상자를 열고 ngrok 경로를 입력한 다음, 로컬 주소에 대한 ngrok 무시 여부를 선택하고 **저장**을 클릭합니다.

![ngrok 경로](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>추가 리소스

Bot Framework Emulator는 오픈 소스입니다. [버그 및 제안을 제출][EmulatorGithubBugs]하여 개발에 [기여][EmulatorGithubContribute]할 수 있습니다.


[EmulatorGithub]: https://github.com/Microsoft/BotFramework-Emulator
[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
[BotFrameworkDevPortal]: https://dev.botframework.com/


[EmulatorConnectPicture]: ~/media/emulator/emulator-connect_localhost_credentials.png
[EmulatorNgrokPath]: ~/media/emulator/emulator-configure_ngrok_path.png
[EmulatorNgrokMonitor]: ~/media/emulator/emulator-testbot-ngrok-monitoring.png
[EmulatorUI]: ~/media/emulator/emulator-ui-new.png

[TroubleshootingGuide]: ~/bot-service-troubleshoot-general-problems.md
[TroubleshootingAuth]: ~/bot-service-troubleshoot-authentication-problems.md
[NodeGetStarted]: ~/nodejs/bot-builder-nodejs-quickstart.md
[CSGetStarted]: ~/dotnet/bot-builder-dotnet-quickstart.md
