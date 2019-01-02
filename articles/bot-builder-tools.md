---
title: CLI 도구를 사용하여 봇 관리
description: Bot Builder 도구를 사용하면 명령줄에서 봇 리소스를 직접 관리할 수 있습니다.
keywords: BotBuilder 템플릿, LUDown, QnA, LUIS, MSBot, 관리, CLI, .bot, 봇
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 11/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5ffaf9a946e1a540b82819b7f745200f47384819
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645663"
---
# <a name="manage-bots-using-cli-tools"></a>CLI 도구를 사용하여 봇 관리

Bot Builder 도구는 계획, 빌드, 테스트, 게시, 연결 및 평가 단계를 포함하는 종단 간 봇 개발 워크플로를 다룹니다. 이러한 도구가 개발 주기의 각 단계에서 지원하는 방식을 살펴보겠습니다.

## <a name="plan"></a>계획

### <a name="create-mock-conversations-using-chatdown"></a>Chatdown을 사용하여 모의 대화 만들기

Chatdown은 .chat 파일을 사용하여 모의 전사를 생성하는 전사 생성기입니다. 생성된 모의 전사 파일은 stdout으로 출력됩니다.

성공적인 애플리케이션 또는 웹 사이트와 마찬가지로 좋은 봇은 지원되는 시나리오에 대한 명확성으로 시작합니다. 다음과 같은 경우 봇과 사용자 간의 대화 모형을 만드는 것이 유용합니다.

- 봇에서 지원되는 시나리오를 프레이밍합니다.
- 비즈니스 의사 결정자가 검토하고 피드백을 제공합니다.
- 사용자와 .chat 봇 파일 형식 간의 대화 흐름을 통해 "정상 경로"(다른 경로와 마찬가지로)를 정의하면 사용자와 봇 간의 대화 모형을 만들 수 있습니다. Chatdown CLI 도구는 .chat 파일을 [Bot Framework Emulator V4](https://github.com/microsoft/botframework-emulator)에서 볼 수 있는 대화 전사 파일(.transcript)로 변환합니다.

`.chat` 파일의 예는 다음과 같습니다.

```markdown
user=Joe
bot=LulaBot

bot: Hi!
user: yo!
bot: [Typing][Delay=3000]
Greetings!
What would you like to do?
* update - You can update your account
* List - You can list your data
* help - you can get help

user: I need the bot framework logo.

bot:
Here you go.
[Attachment=bot-framework.png]
[Attachment=http://yahoo.com/bot-framework.png]
[AttachmentLayout=carousel]

user: thanks
bot:
Here's a form for you
[Attachment=card.json adaptivecard]

```

### <a name="create-a-transcript-file-from-chat-file"></a>.chat 파일에서 전사 파일 만들기
Chatdown 명령은 다음과 같습니다.

```bash
chatdown sample.chat > sample.transcript
```

그러면 `sample.chat`을 사용하여 `sample.transcript`를 출력합니다. 자세한 내용은 [Chatdown CLI][chatdown]를 참조하세요.

## <a name="build"></a>빌드
### <a name="create-a-luis-application-with-ludown"></a>LUDown을 사용하여 LUIS 애플리케이션 만들기
LUDown 도구를 사용하여 LUIS와 QnA 둘 다에 대한 새 .json 모델을 만들 수 있습니다.  
LUIS 포털에서와 마찬가지로, LUIS 애플리케이션에 대한 [의도](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents) 및 [엔터티](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities)를 정의할 수 있습니다.

‘#\<intent-name\>’은 새 의도 정의 섹션을 설명합니다. 그 뒤의 각 줄에는 해당 의도를 설명하는 [발화](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-example-utterances)가 표시됩니다.

예를 들어, 다음과 같이 단일 .lu 파일에 여러 개의 LUIS 의도를 만들 수 있습니다. 

```LUDown
# Greeting
- Hi
- Hello
- Good morning
- Good evening

# Help
- help
- I need help
- please help
```

### <a name="create-qna-pairs-with-ludown"></a>LUDown을 사용하여 QnA 쌍 만들기

.lu 파일 형식은 다음 표기법을 사용하는 QnA 쌍도 지원합니다. 

```LUDown
> comment
### ? question ?
  ```markdown
    answer
  ```

LUDown 도구는 질문과 대답을 qnamaker JSON 파일로 자동으로 구분하며, 이 파일을 사용하여 새로운 [QnaMaker.ai](http://qnamaker.ai) 기술 자료를 만들 수 있습니다.

```LUDown
### ? How do I change the default message for QnA Maker?
  ```markdown
  You can change the default message if you use the QnAMakerDialog. 
  See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
  ```

단일 대답에 대해 새 변형된 질문 줄을 추가하여 동일한 대답에 여러 질문을 추가할 수도 있습니다.

```LUDown
### ? What is your name?
- What should I call you?
  ```markdown
    I'm the echoBot! Nice to meet you.
  ```

### <a name="generate-json-models-with-ludown"></a>LUDown을 사용하여 .json 모델 생성

.lu 형식으로 LUIS 또는 QnA 언어 구성 요소를 정의한 후 LUIS .json, QnA .json 또는 QnA .tsv 파일에 게시할 수 있습니다. LUDown 도구를 실행하면 동일한 작업 디렉터리에서 구문 분석할 .lu 파일을 찾습니다. LUDown 도구는 .lu 파일을 사용하여 LUIS 또는 QnA 둘 다를 대상으로 지정할 수 있으므로 일반 명령 **ludown parse <Service> -- in <luFile>** 을 사용하여 생성할 대상 언어 서비스를 지정하면 됩니다. 

샘플 작업 디렉터리에는 LUIS 모델을 만들기 위한 ‘1.lu’ 및 QnA 기술 자료를 만들기 위한 ‘qna1.lu’ 등 구문 분석할 두 개의 .lu 파일이 있습니다.

#### <a name="generate-luis-json-models"></a>LUIS .json 모델 생성

LUDown을 사용하여 현재 작업 디렉터리에 LUIS 모델을 생성하려면 다음을 입력하면 됩니다.

```shell
ludown parse ToLuis --in <luFile>
```

#### <a name="generate-qna-knowledge-base"></a>QnA 기술 자료 생성

마찬가지로, QnA 기술 자료를 만들려면 구문 분석 대상을 변경하면 됩니다.

```shell
ludown parse ToQna --in <luFile> 
```

생성된 JSON 파일은 LUIS 및 QnA가 해당 포털이나 새 CLI 도구를 통해 사용할 수 있습니다. 자세한 내용은 [LUdown CLI][ludown] GitHub 리포지토리를 참조하세요.

### <a name="track-service-references-using-bot-file"></a>.bot 파일을 사용하여 서비스 참조 추적

새 [MSBot][msbotCli] 도구를 사용하면 봇에서 사용하는 다양한 서비스에 대한 메타데이터를 모두 한곳에 저장하는 **.bot** 파일을 만들 수 있습니다. 이 파일을 사용하면 봇이 CLI에서 이러한 서비스에 연결할 수도 있습니다. 도구는 npm 모듈로 제공됩니다. 도구를 설치하려면 다음을 실행합니다.

```shell
npm install -g msbot
```

봇 파일을 만들려면 CLI에서 **msbot init**, 봇 이름, 대상 URL 엔드포인트를 차례로 입력합니다. 예를 들면 다음과 같습니다.

```shell
msbot init --name TestBot --endpoint http://localhost:9499/api/messages
```
봇을 서비스에 연결하려면 CLI에서 **msbot connect**와 적절한 서비스를 차례로 입력합니다.

```shell
msbot connect [Service]
```

지원되는 서비스 목록을 가져오려면 [추가 정보][msbotCli] 파일을 참조하세요.

### <a name="create-and-manage-luis-applications-using-luis-cli"></a>LUIS CLI를 사용하여 LUIS 애플리케이션 만들기 및 관리

새 도구 집합에는 LUIS 리소스를 독립적으로 관리할 수 있는 [LUIS 확장][luisCli]이 포함되어 있습니다. 다운로드할 수 있는 npm 모듈로 제공됩니다.

```shell
npm install -g luis-apis
```
CLI에서 LUIS 도구에 대한 기본 명령 사용법은 다음과 같습니다.

```shell
luis <action> <resource> <args...>
```
봇을 LUIS에 연결하려면 **.luisrc** 파일을 만들어야 합니다. 이는 애플리케이션이 아웃바운드 호출을 수행할 때 LUIS appID 및 암호를 서비스 엔드포인트에 프로비전하는 구성 파일입니다. 다음과 같이 **luis init**를 실행하면 이 파일을 만들 수 있습니다.

```shell
luis init
```
도구가 파일을 생성하기 전에 LUIS 작성 키, 지역 및 appID를 입력하라는 메시지가 터미널에 표시됩니다.  

이 파일이 생성되고 나면 애플리케이션이 CLI에서 다음 명령을 사용하여 LUDown에서 생성된 LUIS .json 파일을 사용할 수 있습니다.

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```
자세한 내용은 [LUIS CLI][luisCli] GitHub 리포지토리를 참조하세요.

### <a name="create-qna-maker-kb-using-qna-maker-cli"></a>QnA Maker CLI를 사용하여 QnA Maker KB 만들기

새 도구 집합에는 LUIS 리소스를 독립적으로 관리할 수 있는 [QnA 확장][qnaCli]이 포함되어 있습니다. 다운로드할 수 있는 npm 모듈로 제공됩니다.

```shell
npm install -g qnamaker
```
QnA Maker 도구를 사용하여 기술 자료를 만들고, 업데이트, 게시, 삭제 및 학습할 수 있습니다. [ludown parse toqna](#generate-qna-knowledge-base) 명령을 통해 생성된 파일을 사용하여 기술 자료를 생성/대체할 수 있습니다.

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

자세한 내용은 [QnA Maker CLI][qnaCli] GitHub 리포지토리를 참조하세요.

### <a name="create-dispatch-model-using-dispatch-cli"></a>디스패치 CLI를 사용하여 디스패치 모델 만들기

디스패치는 LUIS 모델, QnA 기술 자료 등(파일 형식으로 디스패치하는 데 추가되는 항목)과 같은 여러 봇 모듈 간에 의도를 디스패치하는 데 사용되는 LUIS 모델을 만들고 평가하는 도구입니다.

디스패치 모델을 사용하는 경우는 다음과 같습니다.

- 봇은 여러 모듈로 구성되며 사용자의 발화를 이러한 모듈로 라우팅하고 봇 통합을 평가하는 데 도움이 필요합니다.
- 단일 LUIS 모델의 의도 분류에 대한 품질을 평가합니다.
- 텍스트 파일에서 텍스트 분류 모델을 만듭니다.

봇에서 사용하는 [LUIS 애플리케이션][msbotCli-luis]과 [QnA Maker 기술 자료][msbotCli-qna]를 사용하여 .bot 파일을 어셈블한 후에는 간단히 다음을 사용하여 디스패치 모델을 빌드할 수 있습니다. 

```shell
dispatch create -b <YOUR-BOT-FILE> | msbot connect dispatch --stdin
```
자세한 내용은 [디스패치 CLI][dispatchCli]를 참조하세요.

## <a name="test"></a>테스트

Bot Framework [에뮬레이터](bot-service-debug-emulator.md)는 봇 개발자가 localhost를 사용하거나 터널을 통해 원격으로 실행하여 봇을 테스트하고 디버그할 수 있는 데스크톱 애플리케이션입니다.

## <a name="publish"></a>게시

Azure CLI를 사용하여 Azure Bot Service에 봇을 만들고, 다운로드하고, 게시할 수 있습니다. 다음을 통해 봇 확장을 설치합니다. 
```shell
az extension add -n botservice
```

### <a name="create-azure-bot-service-bot"></a>Azure Bot Service 봇 만들기

참고: 최신 버전의 `az cli`를 사용해야 합니다. MSBot 도구에서 작동할 수 있도록 az cli를 업그레이드하세요. 

다음을 통해 Azure 계정에 로그인합니다. 
```shell
az login
```

로그인하면 다음을 사용하여 새 Azure Bot Service 봇을 만들 수 있습니다. 
```shell
az bot create [options]
```

봇을 만들고 .bot 파일을 봇 구성으로 업데이트하려면 다음을 실행합니다.  
```shell
az bot create [options] --msbot | msbot connect bot --stdin
```

기존 봇이 있는 경우 다음을 실행합니다.  
```shell
az bot show [options] --msbot | msbot connect bot --stdin
```

| 옵션                            | 설명                                   |
|-----------------------------------|-----------------------------------------------|
| --kind -k [필수]              | 봇의 종류입니다.  허용되는 값: function, registration, webapp|
| --name -n [필수]              | 봇의 리소스 이름입니다. |
| --appid                           | 봇에서 사용할 MSA 계정 ID입니다.   |
| --location -l                     | 위치입니다. `az configure --defaults location=<location>`을 사용하여 기본 위치를 구성할 수 있습니다.  기본값: westus|
| --msbot                           | 출력을 .bot 파일과 호환되는 json으로 표시합니다.  허용되는 값: true, false|
| --password -p                     | 개발자 포털의 봇에 대한 MSA 암호입니다. |
| --resource-group -g               | 리소스 그룹의 이름입니다. `az configure --defaults group=<name>`을 사용하여 기본 그룹을 구성할 수 있습니다.  기본값: build2018 |
| --tags                            | 봇에 추가할 태그 집합입니다. |


### <a name="configure-channels"></a>채널 구성

Azure CLI를 사용하여 봇의 채널을 관리할 수 있습니다. 
```shell
>az bot -h
Group
   az bot: Manage Bot Services.
    Subgroups:
        directline: Manage Directline Channel on a Bot.
        email     : Manage Email Channel on a Bot.
        facebook  : Manage Facebook Channel on a Bot.
        kik       : Manage Kik Channel on a Bot.
        msteams   : Manage Msteams Channel on a Bot.
        skype     : Manage Skype Channel on a Bot.
        slack     : Manage Slack Channel on a Bot.
        sms       : Manage Sms Channel on a Bot.
        telegram  : Manage Telegram Channel on a Bot.
        webchat   : Manage Webchat Channel on a Bot.

    Commands:
        create    : Create a new Bot Service.
        delete    : Delete an existing Bot Service.
        download  : Download an existing Bot Service.
        publish   : Publish to an existing Bot Service.
        show      : Get an existing Bot Service.
        update    : Update an existing Bot Service.

```

## <a name="additional-information"></a>추가 정보
- [GitHub의 Bot Builder 도구][cliTools]

<!-- Footnote links -->

[cliTools]: https://aka.ms/botbuilder-tools-readme
[azureCli]: https://aka.ms/botbuilder-tools-azureCli
[msbotCli]: https://aka.ms/botbuilder-tools-msbot-readme
[msbotCli-luis]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-luis-application
[msbotCli-qna]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-qna-maker-knowledge-base
[chatdown]: https://aka.ms/botbuilder-tools-chatdown
[ludown]: https://aka.ms/botbuilder-ludown
[luisCli]: https://aka.ms/botbuilder-luis-cli
[qnaCli]: https://aka.ms/botbuilder-tools-qnaMaker
[dispatchCli]: https://aka.ms/botbuilder-tools-dispatch
