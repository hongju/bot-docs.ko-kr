---
title: Azure CLI를 사용하여 봇 만들기
description: Botbuilder 도구를 사용하면 명령줄에서 바로 봇 리소스를 관리할 수 있음
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 20258949cd8ea403e5cc9bf774d6a3b7c1e86e7e
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352902"
---
# <a name="create-bots-with-azure-cli"></a>Azure CLI를 사용하여 봇 만들기

[봇 작성기 도구](https://github.com/microsoft/botbuilder-tools)는 명령줄에서 바로 봇 리소스를 관리하고 조작할 수 있게 해주는 새 도구 집합입니다. 

이 자습서에서는 다음에 대한 방법을 보여줍니다.

- Azure CLI 봇 확장을 사용하도록 설정
- Azure CLI를 사용하여 새 봇 만들기 
- 개발에 대한 로컬 복사본 다운로드
- 새 MSBot 도구를 사용하여 모든 봇 리소스 정보 저장
- LUDown를 사용하여 LUIS 및 QnA 모델을 관리하고 만들거나 업데이트
- CLI에서 LUIS 및 QnA Maker 서비스에 연결
- CLI에서 Azure에 봇 배포

## <a name="prerequisites"></a>필수 조건

명령줄에서 이러한 도구를 사용하도록 설정하려면 Node.js를 머신에 설치해야 합니다. 

- [Node.js(v 8.5 이상)](https://nodejs.org/en/)

## <a name="1-enable-azure-cli"></a>1. Azure CLI를 사용하도록 설정

이제 다른 모든 Azure 리소스처럼 [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)를 사용하여 봇을 관리할 수 있습니다. Azure CLI를 사용하려면 다음 단계를 수행합니다.

1. Azure CLI가 아직 없는 경우 [다운로드](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)합니다. 

2. Azure Bot 확장 배포 패키지를 다운로드하려면 다음 명령을 입력합니다.

```azurecli
az extension add -n botservice
```

>[!TIP]
> Azure Bot 확장은 현재 v3 봇만 지원합니다.
  
3. 다음 명령을 실행하여 Azure CLI에 [로그인](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest)합니다.

```azurecli
az login
```
고유한 임시 인증 코드를 묻는 메시지가 나타납니다. 로그인하려면 웹 브라우저를 사용하여 Microsoft [장치 로그인](https://microsoft.com/devicelogin)을 방문하고 계속하려면 CLI에서 제공하는 코드를 붙여넣습니다. 

![MS 장치 로그인](media/bot-builder-tools/ms-device-login.png)

로그인에 성공하면 계정 및 리소스를 관리하는 사용할 수 있는 옵션 목록과 함께 Azure CLI 시작 화면이 표시됩니다.

![Azure Bot CLI](media/bot-builder-tools/az-cli-bot.png)


 Azure CLI 명령의 전체 목록은 [여기를 클릭](https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest)합니다.


## <a name="2-create-a-new-bot-from-azure-cli"></a>2. Azure CLI에서 새 봇 만들기

새 봇 확장 및 Azure CLI를 사용하여 명령줄에서 완전히 새로운 봇을 만들 수 있습니다. 

```azurecli
az bot [command]
```
|명령|  |
|----|----|
| create      |리소스 추가|
| delete     |리소스 복제하기|
| 다운로드   | 봇 소스 코드 다운로드|
| 게시   |기존 봇 서비스에 게시|
| 표시 |기존 봇 리소스를 표시합니다.|
| update| 기존 봇 서비스 업데이트|

CLI에서 새 봇을 만들려면 기존 [리소스 그룹](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview)을 선택하거나 새로 만들어야 합니다. 

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot"
```
요청이 성공한 후 확인 메시지가 표시됩니다.
```
obtained msa app id and password. Provisioning bot now.
```

> [!TIP]
> **리소스 그룹을 찾을 수 없습니다**라는 오류 메시지가 표시되면 Azure CLI에서 [구독](https://docs.microsoft.com/en-us/azure/architecture/cloud-adoption-guide/adoption-intro/subscription-explainer)을 설정해야 할 수 있습니다. Azure 구독은 리소스 그룹을 만들 때 입력한 구독과 일치해야 합니다. 설정하려면 다음을 입력합니다.
> ```azurecli
> az account set --subscription "your-subscription-name"
> ```
> 계정에 대한 구독 목록을 보려면 다음 명령을 입력합니다.
> ```azurecli
> az account list
> ```

기본적으로 새 .NET 봇이 만들어집니다. **-- lang** 인수를 사용하여 언어를 지정함으로써 플랫폼 SDK를 지정할 수 있습니다. 현재 봇 확장 패키지는 C# 및 Node.js 봇 SDK를 지원합니다. 예를 들어 **Node.js 봇을 만들**려면

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot" --lang Node 
```
새 echo 봇이 Azure에서 리소스 그룹에 프로비전됩니다. 이를 테스트하려면 Web App 봇 보기의 봇 관리 헤더 아래에서 **Webchat 테스트**를 선택하면 됩니다. 

![Azure Echo 봇](media/bot-builder-tools/az-echo-bot.png) 

## <a name="3-download-the-bot-locally"></a>3. 로컬로 봇 다운로드

두 가지 방법으로 새 봇의 소스 코드를 다운로드할 수 있습니다.
- Azure Portal에서 다운로드합니다.
- 새 Azure CLI를 사용하여 다운로드합니다.

포털에서 봇 소스 코드를 다운로드하려면 봇 관리 아래에서 봇 리소스 및 **빌드**를 차례로 선택합니다. 로컬로 봇의 소스 코드를 관리 또는 검색에 사용할 수 있는 여러 가지 옵션이 있습니다. 

![Azure Portal 봇 다운로드](media/bot-builder-tools/az-portal-manage-code.png)

CLI를 사용하여 봇 소스를 다운로드하려면 다음 명령을 입력합니다. 봇은 하위 디렉터리에 다운로드됩니다. 하위 디렉터리가 아직 존재하지 않는 경우 명령을 입력하여 만듭니다.

```azurecli
az bot download --name "my-bot-name" --resource-group "my-resource-group"
```
그러나 봇을 다운로드할 디렉터리를 지정할 수도 있습니다.
예: 

![CLI 다운로드 명령](media/bot-builder-tools/cli-bot-download-command.png)

![CLI 봇 다운로드](media/bot-builder-tools/cli-bot-download.png)

위의 명령을 사용하면 로컬로 봇을 개발할 수 있도록 지정된 위치에 직접 봇의 소스 코드를 다운로드할 수 있습니다.


## <a name="4-store-your-bot-information-with-msbot"></a>4. MSBot을 사용하여 봇 정보 저장

새로운 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) 도구를 사용하면 봇이 사용하는 다양한 서비스에 대한 메타데이터를 모두 한곳에 저장하는 **.bot** 파일을 만들 수 있습니다. 이 파일을 사용하면 봇이 CLI에서 이러한 서비스에 연결할 수도 있습니다. 도구는 npm 모듈로 제공됩니다. 도구를 설치하려면 다음을 실행합니다.

```shell
npm install -g msbot 
```

봇 파일을 만들려면 CLI에서 **msbot init**, 봇 이름, 대상 URL 엔드포인트를 차례로 입력합니다. 예를 들면 다음과 같습니다.

```shell
msbot init --name name-of-my-bot --endpoint http://localhost:bot-port-number/api/messages
```
봇을 서비스에 연결하려면 CLI에서 **msbot connect**와 적절한 서비스를 차례로 입력합니다.

```shell
msbot connect service-type
```

| 서비스 유형 | 설명 |
| ------ | ----------- |
| azure  |봇을 Azure Bot Service 등록에 연결|
|endpoint| 봇을 localhost 같은 엔드포인트에 연결|
|luis     | 봇을 LUIS 응용 프로그램에 연결 |
| qna     |봇을 QnA 기술 자료에 연결|
|도움말[cmd]  |[cmd]에 대한 도움말을 표시|

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>.bot 파일을 사용하여 봇을 ABS에 연결

MSBot 도구가 설치되어 있으면 az bot **show** 명령을 실행하여 Azure Bot Service의 기존 리소스 그룹에 봇을 쉽게 연결할 수 있습니다. 

```azurecli
az bot show -n my-bot-name -g my-resource-group --msbot | msbot connect azure --stdin
```

이렇게 하면 대상 리소스 그룹에서 현재 엔드포인트, MSA appID 및 암호를 가져와 .bot 파일의 정보를 적절하게 업데이트합니다. 


## <a name="5-manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>5. 새 botbuilder 도구를 사용하여 LUIS 및 QnA 서비스 관리, 업데이트 또는 만들기

[봇 작성기 도구](https://github.com/microsoft/botbuilder-tools)는 명령줄에서 바로 봇 리소스를 관리하고 조작할 수 있게 해주는 새 도구 집합입니다. 

>[!TIP]
> 모든 봇 작성기 도구에는 **-h** 또는 **-help**를 입력하여 명령줄에서 액세스할 수 있는 글로벌 도움말 명령이 포함되어 있습니다. 이 명령은 언제든지 모든 작업에서 사용할 수 있으며, 사용 가능한 옵션의 유용한 표시를 설명과 함께 제공합니다.

### <a name="ludown"></a>LUDown
[LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown)을 사용하면 **.lu** 파일을 통해 봇의 강력한 언어 구성 요소를 설명하고 만들 수 있습니다. 새 .lu 파일은 LUDown 도구가 대상 서비스와 관련된 .json 파일을 사용하고 출력하는 markdown 형식 유형입니다. 이제 .lu 파일을 사용하여 새로운 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-get-started-create-app) 응용 프로그램 또는 [QnA](https://qnamaker.ai/Documentation/CreateKb) 기술 자료를 각각 다른 형식으로 만들 수 있습니다. LUDown은 npm 모듈로 제공되며, 머신에 글로벌로 설치하여 사용할 수 있습니다.

```shell
npm install -g ludown
```
LUDown 도구를 사용하여 LUIS와 QnA 둘 다에 대한 새 .json 모델을 만들 수 있습니다.  


### <a name="creating-a-luis-application-with-ludown"></a>LUDown를 사용하여 LUIS 응용 프로그램 만들기

LUIS 포털에서와 마찬가지로, LUIS 응용 프로그램에 대한 [의도](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents) 및 [엔터티](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities)를 정의할 수 있습니다. 

`# \<intent-name\>`은 새 의도 정의 섹션을 설명합니다. 이후 줄에는 의도를 설명하는 [발언](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-example-utterances)이 포함됩니다.

예를 들어 다음과 같이 단일 .lu 파일에 여러 개의 LUIS 의도를 만들 수 있습니다. 

```ludown
# Greeting
Hi
Hello
Good morning
Good evening

# Help
help
I need help
please help
```

### <a name="qna-pairs-with-ludown"></a>LUDown을 사용하는 QnA 쌍

.lu 파일 형식은 다음 표기법을 사용하는 QnA 쌍도 지원합니다. 

  ```ludown
  > This is a comment. QnA definitions have the general format:
  ### ? this-is-the-question-string
  - this-is-an-alternate-form-of-the-same-question
  - this-is-another-one
    ```markdown
    this-is-the-answer
    ```
  ```
LUDown 도구는 질문과 대답을 qnamaker JSON 파일로 자동으로 구분하며, 이 파일을 사용하여 새로운 [QnaMaker.ai](http://qnamaker.ai) 기술 자료를 만들 수 있습니다.

  ```ludown
  ### ? How do I change the default message for QnA Maker?
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

단일 대답에 대해 새 변형된 질문 줄을 추가하여 동일한 대답에 여러 질문을 추가할 수도 있습니다. 

  ```ludown
  ### ? What is your name?
  - What should I call you?
    ```markdown
    I'm the echoBot! Nice to meet you.
    ```
  ```

### <a name="generating-json-models-with-ludown"></a>LUDown을 사용하여 .json 모델 생성

.lu 형식으로 LUIS 또는 QnA 언어 구성 요소를 정의한 후 LUIS .json, QnA .json 또는 QnA .tsv 파일에 게시할 수 있습니다. LUDown 도구를 실행하면 동일한 작업 디렉터리에서 구문 분석할 .lu 파일을 찾습니다. LUDown 도구는 .lu 파일을 사용하여 LUIS 또는 QnA 둘 다를 대상으로 지정할 수 있으므로 일반 명령 **ludown parse <Service> -- in <luFile>** 을 사용하여 생성할 대상 언어 서비스를 지정하면 됩니다. 

샘플 작업 디렉터리에는 LUIS 모델을 만들기 위한 'luis-sample.lu' 및 QnA 기술 자료를 만들기 위한 'qna-sample.lu' 등 구문 분석할 두 개의 .lu 파일이 있습니다.


#### <a name="generate-luis-json-models"></a>LUIS .json 모델 생성

**luis-sample.lu** 
```ludown
# Greeting
- Hi
- Hello
- Good morning
- Good evening
```

LUDown을 사용하여 현재 작업 디렉터리에 LUIS 모델을 생성하려면 다음을 입력하면 됩니다.

```shell
ludown parse ToLuis --in ludown-file-name.lu
```

#### <a name="generate-qna-knowledge-base-json"></a>QnA 기술 자료 .json 생성

**qna-sample.lu**
  ```ludown
  > This is a sample ludown file for QnA Maker.

  ### ? How do I change the default message
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

마찬가지로, QnA 기술 자료를 만들려면 구문 분석 대상을 변경하면 됩니다. 

```shell
ludown parse ToQna --in ludown-file-name.lu
```

생성된 JSON 파일은 LUIS 및 QnA가 해당 포털이나 새 CLI 도구를 통해 사용할 수 있습니다. 

## <a name="6-connect-to-luis-an-qna-maker-services-from-the-cli"></a>6. CLI에서 LUIS 및 QnA Maker 서비스에 연결

### <a name="connect-to-luis-from-the-cli"></a>CLI에서 LUIS에 연결 

새 도구 집합에는 LUIS 리소스를 독립적으로 관리할 수 있게 해주는 [LUIS 확장](https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS)이 포함되어 있습니다. 다운로드할 수 있는 npm 모듈로 제공됩니다.

```shell
npm install -g luis-apis
```
CLI에서 LUIS 도구에 대한 기본 명령 사용법은 다음과 같습니다.

```shell
luis action-name resource-name arguments-list
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
QnA Maker 도구를 사용하여 기술 자료를 만들고, 업데이트, 게시, 삭제 및 학습할 수 있습니다. 시작하려면 서비스에 엔드포인트를 사용하는 데 필요한 **.qnamakerrc** 파일을 만들어야 합니다. **qnamaker init**를 실행하고 QnA Maker 기술 자료 ID와 함께 프롬프트를 따르면 이 파일을 쉽게 만들 수 있습니다. 

```shell
qnamaker init 
```
![QnaMaker init](media/bot-builder-tools/qnamaker-init.png)

.qnamakerrc 파일이 생성되고 나면 다음 명령을 사용하여 QnA 기술 자료에 연결하고 LUDown에서 생성된 기술 자료 .json/.tsv 파일을 사용할 수 있습니다.

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="7-publish-to-azure-from-the-cli"></a>7. CLI에서 Azure에 게시

봇의 소스 코드를 변경한 후 다음을 실행하여 변경 내용을 원활히 게시할 수 있습니다.

```azurecli
az bot publish --name "my-bot-name" --resource-group "my-resource-group"
```

## <a name="references"></a>참조
- [BotBuilder 도구 소스 코드](https://github.com/Microsoft/botbuilder-tools)
- [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot)
- [ChatDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown)
- [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/ludown)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)


