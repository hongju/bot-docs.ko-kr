---
title: Botbuilder 템플릿을 사용하여 봇 만들기
description: Botbuilder 도구를 사용하면 명령줄에서 바로 봇 리소스를 관리할 수 있습니다.
keywords: botbuilder 템플릿, node.js, python, java, .net, ludown, yeoman
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
ms.openlocfilehash: 60cdc3de200336b00173749a553205a47a32457e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300701"
---
# <a name="create-bots-with-botbuilder-templates"></a>Botbuilder 템플릿을 사용하여 봇 만들기

이제 템플릿을 사용하여 다음 Botbuilder SDK 플랫폼에서 각각 봇을 만들 수 있습니다. 

- Node.js
- 파이썬
- 자바
- .NET

## <a name="prerequisites"></a>필수 조건

Node.js, Java 및 Python 템플릿은 모두 간단한 에코 봇을 만들며, 모두 [Yeoman](http://yeoman.io/)을 통해 제공됩니다.

- [Node.js](https://nodejs.org/en/) v 8.5 이상
- [Yeoman](http://yeoman.io/)

아직 Yeoman을 설치하지 않은 경우 전역으로 설치해야 합니다.

```shell
npm install -g yo
```

## <a name="nodejs-python-java-templates"></a>Node.js, Python, Java 템플릿
명령줄에서 선택한 새 폴더로 cd합니다. 각각 SDK **v3** 및 **v4** 버전을 대상으로 하는 두 가지 Botbuilder Node.js 템플릿 버전을 사용할 수 있습니다. Python 및 Java 템플릿은 v4에서만 사용할 수 있습니다. 

**Node.js v3 Botbuilder** 템플릿 생성기를 설치하려면

```shell
npm install generator-botbuilder
```

**Node.js v4 Botbuilder** 템플릿 생성기를 설치하려면
```shell
npm install generator-botbuilder@preview
```

Python 및 Java 템플릿은 **v4에서만 사용할 수 있습니다**. 

**Python**:
```shell
npm install generator-botbuilder-python
```

**Java**의 경우
```shell
npm install generator-botbuilder-java
```

생성기가 설치된 후 CLI에서 **yo** 명령을 실행하면 Yeoman에서 사용 가능한 생성기 목록을 볼 수 있습니다.

![Yeoman 인터페이스](media/botbuilder-templates/yeoman-generator-botbuilder.png)

선택한 작업 디렉터리로 전환하고 사용할 생성기를 선택합니다. 이름, 설명 등 봇을 만들기 위한 다양한 옵션을 확인하는 메시지가 표시됩니다. 모든 프롬프트를 완료하면 동일한 작업 폴더에 에코 봇 템플릿이 만들어집니다.

![Node.js Yeoman 템플릿](media/botbuilder-templates/new-template-node.png)


## <a name="net"></a>.NET

각각 SDK **v3** 및 **v4** 버전을 대상으로 하는 두 가지 템플릿을 .NET에 사용할 수 있습니다. 둘 다 [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package) 패키지로 제공되며, 다운로드하려면 [Visual Studio Marketplace](https://marketplace.visualstudio.com/)에서 다음 링크 중 하나를 클릭합니다.

- [BotBuilder V3 템플릿](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3)
- [BotBuilder V4 템플릿](https://aka.ms/Ylcwxk)

#### <a name="prerequisites"></a>필수 조건

- [Visual Studio 2015 이상](https://www.visualstudio.com/downloads/)
- [Azure 계정](https://azure.microsoft.com/en-us/free/)

### <a name="install-the-templates"></a>템플릿 설치

저장된 디렉터리에서 VSIX 패키지를 열면 봇 작성기 템플릿이 Visual Studio에 설치되며, 다음에 템플릿을 열어 사용할 수 있습니다.

![VSIX 패키지](media/botbuilder-templates/botbuilder-vsix-template.png)

템플릿을 사용하여 새로운 봇 프로젝트를 만들려면 Visual Studio를 열고 **파일** > **새로 만들기** > **프로젝트**를 선택한 다음, Visual C#에서 **Bot Framework** > Simple Echo Bot Application(간단한 에코 봇 응용 프로그램)을 선택합니다. 이렇게 하면 원하는 대로 편집할 수 있는 새 에코 봇 프로젝트가 로컬에 만들어집니다. 

![.NET 봇 템플릿](media/botbuilder-templates/new-template-dotnet.png)

## <a name="store-your-bot-information-with-msbot"></a>MSBot을 사용하여 봇 정보 저장

새로운 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) 도구를 사용하면 봇이 사용하는 다양한 서비스에 대한 메타데이터를 모두 한곳에 저장하는 **.bot** 파일을 만들 수 있습니다. 이 파일을 사용하면 봇이 CLI에서 이러한 서비스에 연결할 수도 있습니다. 이 도구는 npm 모듈로 제공됩니다. 도구를 설치하려면 다음을 실행합니다.

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

| 서비스 | 설명 |
| ------ | ----------- |
| azure  |봇을 Azure Bot Service 등록에 연결합니다.|
|localhost| 봇을 localhost 엔드포인트에 연결합니다.|
|luis     | 봇을 LUIS 응용 프로그램에 연결합니다. |
| qna     |봇을 QnA 기술 자료에 연결합니다.|
|help [cmd]  |[cmd]에 대한 도움말을 표시합니다.|

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>.bot 파일을 사용하여 봇을 ABS에 연결

MSBot 도구가 설치되어 있으면 az bot **show** 명령을 실행하여 Azure Bot Service의 기존 리소스 그룹에 봇을 쉽게 연결할 수 있습니다. 

```azurecli
az bot show -n <botname> -g <resourcegroup> --msbot | msbot connect azure --stdin
```

이렇게 하면 대상 리소스 그룹에서 현재 엔드포인트, MSA appID 및 암호를 가져와 .bot 파일의 정보를 적절하게 업데이트합니다. 


## <a name="manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>새로운 botbuilder 도구를 사용하여 LUIS 및 QnA 서비스 관리, 업데이트 또는 만들기

[봇 작성기 도구](https://github.com/microsoft/botbuilder-tools)는 명령줄에서 바로 봇 리소스를 관리하고 조작할 수 있게 해주는 새 도구 집합입니다. 

>[!TIP]
> 모든 봇 작성기 도구에는 **-h** 또는 **-help**를 입력하여 명령줄에서 액세스할 수 있는 전역 도움말 명령이 포함되어 있습니다. 이 명령은 언제든지 모든 작업에서 사용할 수 있으며, 사용 가능한 옵션의 유용한 표시를 설명과 함께 제공합니다. 

### <a name="ludown"></a>LUDown
[LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown)을 사용하면 **.lu** 파일을 통해 봇의 강력한 언어 구성 요소를 설명하고 만들 수 있습니다. 새 .lu 파일은 LUDown 도구가 대상 서비스와 관련된 .json 파일을 사용하고 출력하는 markdown 형식 유형입니다. 이제 .lu 파일을 사용하여 새로운 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-get-started-create-app) 응용 프로그램 또는 [QnA](https://qnamaker.ai/Documentation/CreateKb) 기술 자료를 각각 다른 형식으로 만들 수 있습니다. LUDown은 npm 모듈로 제공되며, 머신에 전역으로 설치하여 사용할 수 있습니다.

```shell
npm install -g ludown
```
LUDown 도구를 사용하여 LUIS와 QnA 둘 다에 대한 새 .json 모델을 만들 수 있습니다.  


### <a name="creating-a-luis-application-with-ludown"></a>LUDown를 사용하여 LUIS 응용 프로그램 만들기

LUIS 포털에서와 마찬가지로, LUIS 응용 프로그램에 대한 [의도](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents) 및 [엔터티](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities)를 정의할 수 있습니다. 

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

### <a name="qna-pairs-with-ludown"></a>LUDown을 사용하는 QnA 쌍

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

### <a name="generating-json-models-with-ludown"></a>LUDown을 사용하여 .json 모델 생성

.lu 형식으로 LUIS 또는 QnA 언어 구성 요소를 정의한 후 LUIS .json, QnA .json 또는 QnA .tsv 파일에 게시할 수 있습니다. LUDown 도구를 실행하면 동일한 작업 디렉터리에서 구문 분석할 .lu 파일을 찾습니다. LUDown 도구는 .lu 파일을 사용하여 LUIS 또는 QnA 둘 다를 대상으로 지정할 수 있으므로 일반 명령 **ludown parse <Service> -- in <luFile>** 을 사용하여 생성할 대상 언어 서비스를 지정하면 됩니다. 

샘플 작업 디렉터리에는 LUIS 모델을 만들기 위한 ‘1.lu’ 및 QnA 기술 자료를 만들기 위한 ‘qna1.lu’ 등 구문 분석할 두 개의 .lu 파일이 있습니다.

#### <a name="generate-luis-json-models"></a>LUIS .json 모델 생성

LUDown을 사용하여 현재 작업 디렉터리에 LUIS 모델을 생성하려면 다음을 입력하면 됩니다.

```shell
ludown parse ToLuis --in <luFile> 
```

#### <a name="generate-qna-knowledge-base-json"></a>QnA 기술 자료 .json 생성

마찬가지로, QnA 기술 자료를 만들려면 구문 분석 대상을 변경하면 됩니다. 

```shell
ludown parse ToQna --in <luFile> 
```

생성된 JSON 파일은 LUIS 및 QnA가 해당 포털이나 새 CLI 도구를 통해 사용할 수 있습니다. 

## <a name="connect-to-luis-an-qna-maker-services-from-the-cli"></a>CLI에서 LUIS 및 QnA Maker 서비스에 연결

### <a name="connect-to-luis-from-the-cli"></a>CLI에서 LUIS에 연결 

새 도구 집합에는 LUIS 리소스를 독립적으로 관리할 수 있게 해주는 [LUIS 확장](https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS)이 포함되어 있습니다. 다운로드할 수 있는 npm 모듈로 제공됩니다.

```shell
npm install -g luis-apis
```
CLI에서 LUIS 도구에 대한 기본 명령 사용법은 다음과 같습니다.

```shell
luis <action> <resource> <args...>
```
봇을 LUIS에 연결하려면 **.luisrc** 파일을 만들어야 합니다. 이는 응용 프로그램이 아웃바운드 호출을 수행할 때 LUIS appID 및 암호를 서비스 엔드포인트에 프로비전하는 구성 파일입니다. 다음과 같이 **luis init**를 실행하면 이 파일을 만들 수 있습니다.

```shell
luis init
```
도구가 파일을 생성하기 전에 LUIS 작성 키, 지역 및 appID를 입력하라는 메시지가 터미널에 표시됩니다.  

![LUIS init](media/bot-builder-tools/luis-init.png) 


이 파일이 생성되고 나면 응용 프로그램이 CLI에서 다음 명령을 사용하여 LUDown에서 생성된 LUIS .json 파일을 사용할 수 있습니다.

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```

### <a name="connect-to-qna-from-the-cli"></a>CLI에서 QnA에 연결

새 도구 집합에는 LUIS 리소스를 독립적으로 관리할 수 있게 해주는 [QnA 확장](https://github.com/Microsoft/botbuilder-tools/tree/master/QnAMaker)이 포함되어 있습니다. 다운로드할 수 있는 npm 모듈로 제공됩니다.

```shell
npm install -g qnamaker
```
QnA Maker 도구를 사용하여 기술 자료를 만들고, 업데이트, 게시, 삭제 및 학습할 수 있습니다. 시작하려면 서비스에 대한 엔드포인트를 사용하는 데 필요한 **.qnamakerrc** 파일을 만들어야 합니다. **qnamaker init**를 실행하고 프롬프트에 따라 이 파일을 쉽게 만들고 QnA Maker 기술 자료 ID를 프로비전할 수 있습니다. 

```shell
qnamaker init 
```
![QnaMaker init](media/bot-builder-tools/qnamaker-init.png)

.qnamakerrc 파일이 생성되고 나면 다음 명령을 사용하여 QnA 기술 자료에 연결하고 LUDown에서 생성된 기술 자료 .json/.tsv 파일을 사용할 수 있습니다.

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="references"></a>참조
- [BotBuilder 도구 소스 코드](https://github.com/Microsoft/botbuilder-tools)
- [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot)
- [ChatDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown)
- [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
