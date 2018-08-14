---
title: Azure Search를 사용하여 데이터 기반 환경 만들기 | Microsoft Docs
description: Azure Search를 사용하여 데이터 기반 환경을 만들고 Node.js용 Bot Builder SDK 및 Azure Search를 사용하여 사용자가 봇에 있는 많은 양의 콘텐츠를 탐색하도록 도와주는 방법을 알아봅니다.
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1a50bb8af6556830ee9f9b047d7c5a2d3399a6b9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302442"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>Azure Search를 사용하여 데이터 기반 환경 만들기 
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.js](../nodejs/bot-builder-nodejs-search-azure.md)

봇에 [Azure Search][search]를 추가하여 사용자가 많은 양의 콘텐츠를 탐색하도록 도와주고, 봇의 사용자를 위한 데이터 기반 탐색 환경을 만들 수 있습니다.

Azure Search는 키워드 검색, 기본 제공 언어학, 사용자 지정 채점, 패싯 탐색 등을 제공하는 Azure 서비스입니다. Azure Search는 Azure SQL DB, DocumentDB, Blob Storage 및 Table Storage를 비롯한 다양한 소스의 콘텐츠를 인덱싱할 수도 있습니다. 다른 데이터 원본에 대한 "푸시" 인덱싱을 지원하며 PDF, Office 문서 및 구조화되지 않은 데이터가 포함된 기타 형식을 열 수 있습니다. 수집된 콘텐츠는 Azure Search 인덱스로 이동한 후 봇에서 쿼리할 수 있습니다.

## <a name="install-dependencies"></a>종속성 설치

명령 프롬프트에서 봇의 프로젝트 디렉터리로 이동하고 NPM(노드 패키지 관리자)을 사용하여 다음 모듈을 설치합니다.

* [bluebird](https://www.npmjs.com/package/bluebird)
* [lodash](https://www.npmjs.com/package/lodash)
* [요청](https://www.npmjs.com/package/request)

## <a name="prerequisites"></a>필수 조건

다음은 **필수**입니다. 
- Azure 구독 및 Azure Search 기본 키가 있습니다. Azure Portal에서 이러한 값을 확인할 수 있습니다.
- [SearchDialogLibrary](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchDialogLibrary) 라이브러리를 봇의 프로젝트 디렉터리에 복사합니다. 이 라이브러리에는 사용자가 검색할 수 있는 일반 대화 상자가 포함되어 있지만 필요에 따라 봇에 맞게 사용자 지정할 수 있습니다. 

- [SearchProviders](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchProviders) 라이브러리를 봇의 프로젝트 디렉터리에 복사합니다. 이 라이브러리에는 요청을 만들어 Azure Search에 제출하는 데 필요한 모든 구성 요소가 포함되어 있습니다.

## <a name="connect-to-the-azure-service"></a>Azure Service에 연결 

봇의 주 프로그램 파일(예: app.js)에서 이전에 설치한 두 라이브러리에 대한 참조 경로를 만듭니다. 

```javascript
var SearchLibrary = require('./SearchDialogLibrary');
var AzureSearch = require('./SearchProviders/azure-search');
```

봇에 다음 샘플 코드를 추가합니다. `AzureSearch` 개체에서 고유의 Azure Search 설정을 `.create` 메서드에 전달합니다. 런타임에 봇을 Azure Search 서비스에 바인딩하고 `Promise` 형태로 완료된 사용자 쿼리를 기다립니다.  

```javascript
// Azure Search
var azureSearchClient = AzureSearch.create('Your-Azure-Search-Service-Name', 'Your-Azure-Search-Primary-Key', 'Your-Azure-Search-Service-Index');
var ResultsMapper = SearchLibrary.defaultResultsMapper(ToSearchHit);
```

 `azureSearchClient` 참조는 봇의 `.env` 설정에서 Azure Service의 권한 부여 설정을 전달하는 Azure Search 모델을 생성합니다. 
 `ResultsMapper`는 Azure 응답 개체를 구문 분석하고 `ToSearchHit` 메서드에 정의된 대로 데이터를 매핑합니다. 이 메서드의 구현 예제는 [Azure Search 응답 후](#after-azure-search-responds)를 참조하세요.

## <a name="register-the-search-library"></a>검색 라이브러리 등록
`SearchLibrary` 모듈 자체에서 직접 검색 대화 상자를 사용자 지정할 수 있습니다. `SearchLibrary`는 Azure Search를 호출하는 것을 비롯하여 대부분의 주요 작업을 수행합니다. 

봇의 주 프로그램 파일에 다음 코드를 추가하여 검색 대화 상자 라이브러리를 봇에 등록합니다. 

```javascript
bot.library(SearchLibrary.create({
    multipleSelection: true,
    search: function (query) { return azureSearchClient.search(query).then(ResultsMapper); },
    refiners: ['refiner1', 'refiner2', 'refiner3'], // customize your own refiners 
    refineFormatter: function (refiners) {
        return _.zipObject(
            refiners.map(function (r) { return 'By ' + _.capitalize(r); }),
            refiners);
    }
}));
```
`SearchLibrary`는 모든 검색 관련 대화 상자를 저장할 뿐만 아니라 Azure Search에 제출할 사용자 쿼리도 가져옵니다. 사용자가 검색 결과의 범위를 좁히거나 필터링할 수 있게 엔터티를 지정하려면 `refiners` 배열에 고유한 구체화를 정의해야 합니다.  

## <a name="create-a-search-dialog"></a>검색 대화 상자 만들기

원하는 대로 대화 상자를 구조화하도록 선택할 수 있습니다. Azure Search 대화 상자를 설정하려면 `SearchLibrary` 개체에서 `.begin` 메서드를 호출하여 Bot Builder SDK에서 생성한 `session` 개체를 제출하기만 하면 됩니다. 

```javascript
function (session) {
        // Trigger Azure Search dialogs 
        SearchLibrary.begin(session);
    },
    function (session, args) {
        // Process selected search results
        session.send(
            'Search Completed!',
            args.selection.map(  ); // format your response 
    }
```
대화 상자에 대한 자세한 내용은 [대화 상자로 대화 관리](bot-builder-nodejs-dialog-manage-conversation.md)를 참조하세요.

## <a name="after-azure-search-responds"></a>Azure Search 응답 후 

Azure Search가 성공적으로 확인되면 이제 응답 개체에서 원하는 데이터를 저장하고 사용자에게 의미 있는 방식으로 표시해야 합니다.

> [!TIP]
> [util 모듈][NodeUtil]을 포함하는 것을 고려해 보세요. Azure Search의 응답 형식을 지정하고 매핑하는 데 도움이 됩니다.

봇의 주 프로그램 파일에서 `ToSearchHit` 메서드를 만듭니다. 이 메서드는 Azure 응답에서 필요한 관련 데이터의 형식을 지정하는 개체를 반환합니다. 다음 코드는 `ToSearchHit` 메서드에서 고유한 매개 변수를 정의하는 방법을 보여 줍니다. 
 
 ```javascript
 function ToSearchHit(azureResponse) {
     return {
         // define your own parameters 
         key: azureResponse.id,
         title: azureResponse.title,
         description: azureResponse.description,
         imageUrl: azureResponse.thumbnail
     };
 }
```
이 작업이 완료되면 데이터를 사용자에게 표시하기만 하면 됩니다. 

 **SearchDialogLibrary** 프로젝트의 **index.js** 파일에서 `searchHitAsCard` 메서드는 Azure Search의 각 응답을 구문 분석하고 사용자에게 표시할 새 카드 개체를 만듭니다. 봇의 주 프로그램 파일에 있는 `ToSearchHit` 메서드에 정의한 필드를 `searchHitAsCard` 메서드의 속성과 동기화해야 합니다. 

다음은 `ToSearchHit` 메서드의 정의된 매개 변수를 사용하여 사용자에게 렌더링할 카드 첨부 파일 UI를 빌드하는 방법과 위치를 보여 줍니다. 

```javascript
function searchHitAsCard(showSave, searchHit) {
    var buttons = showSave
        ? [new builder.CardAction().type('imBack').title('Save').value(searchHit.key)]
        : [];

    var card = new builder.HeroCard()
        .title(searchHit.title) 
        .buttons(buttons);

    if (searchHit.description) {
        card.subtitle(searchHit.description);
    }

    if (searchHit.imageUrl) {
        card.images([new builder.CardImage().url(searchHit.imageUrl)]);
    }

    return card;
}
```

## <a name="sample-code"></a>샘플 코드

Node.js용 Bot Builder SDK를 사용하여 봇과 함께 Azure Search를 지원하는 방법을 보여 주는 두 개의 완전한 샘플은 GitHub의 [부동산 봇 샘플](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search/RealEstateBot) 또는 [작업 목록 봇 샘플](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search/JobListingBot)을 참조하세요. 

## <a name="additional-resources"></a>추가 리소스

* [Azure Search][search]
* [Node Util][NodeUtil]
* [대화 상자](bot-builder-nodejs-dialog-manage-conversation.md)

[NodeUtil]: https://nodejs.org/api/util.html
[search]: /azure/search/search-what-is-azure-search