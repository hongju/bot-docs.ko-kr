---
title: Azure Search를 사용하여 데이터 기반 환경 만들기 | Microsoft Docs
description: Azure Search를 사용하여 데이터 기반 환경을 만들고 .NET 및 Azure Search용 Bot Framework SDK를 사용하여 사용자가 봇에 있는 많은 양의 콘텐츠를 이동하는 데 도움이 되는 방법에 대해 알아봅니다.
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/28/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e0ffb9c24b5e85b0eb1afdd885654e4864e65939
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032931"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>Azure Search를 사용하여 데이터 기반 환경 만들기 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-search-azure.md)

봇에 [Azure Search](https://azure.microsoft.com/en-us/services/search/)를 추가하여 사용자가 많은 양의 콘텐츠를 이동하고 데이터 기반 탐색 환경을 만들 수 있습니다.

Azure Search는 키워드 검색, 기본 제공 언어학, 사용자 지정 스코어링, 패싯 탐색 등을 제공하는 Azure 서비스입니다. Azure Search는 또한 Azure SQL DB, DocumentDB, Blob Storage 및 Table Storage를 비롯한 다양한 소스의 콘텐츠를 인덱싱할 수 있습니다. 다른 데이터 원본에 대한 "푸시" 인덱싱을 지원하며 PDF, Office 문서 및 구조화되지 않은 데이터가 포함된 기타 형식을 열 수 있습니다. 수집된 콘텐츠는 Azure Search 인덱스로 이동한 후 봇에서 쿼리할 수 있습니다.

## <a name="prerequisites"></a>필수 조건

[Microsoft.Azure.Search](https://www.nuget.org/packages/Microsoft.Azure.Search/4.0.0-preview) Nuget 패키지를 봇 프로젝트에 설치합니다.

봇의 솔루션에는 다음 세 가지 C# 프로젝트가 필요합니다. 이러한 프로젝트는 봇과 Azure Search에 대한 추가 기능을 제공합니다. [GitHub](https://aka.ms/v3-cs-search-demo)에서 프로젝트를 포크하거나 소스 코드를 직접 다운로드합니다.

- **Search.Azure** 프로젝트는 Azure Service 호출을 정의합니다.
- **Search.Contracts** 프로젝트는 데이터를 처리하는 제네릭 인터페이스 및 데이터 모델을 정의합니다.
- **Search.Dialogs** 프로젝트에는 Azure Search를 쿼리하는 데 사용되는 다양한 제네릭 Bot Builder 대화 상자가 포함됩니다.

## <a name="configure-azure-search-settings"></a>Azure Search 설정 구성

값 필드에 있는 사용자 고유의 Azure Search 자격 증명을 사용하여 프로젝트의 **Web.config** 파일에서 Azure Search 설정을 구성합니다. `AzureSearchClient` 클래스의 생성자는 이러한 설정을 사용하여 봇을 Azure 서비스에 등록하고 바인딩하게 됩니다.

```xml
<appSettings>
    <add key="SearchDialogsServiceName" value="Azure-Search-Service-Name" /> <!-- replace value field with Azure Service Name --> 
    <add key="SearchDialogsServiceKey" value="Azure-Search-Service-Primary-Key" /> <!-- replace value field with Azure Service Key --> 
    <add key="SearchDialogsIndexName" value="Azure-Search-Service-Index" /> <!-- replace value field with your Azure Search Index --> 
</appSettings>
```

## <a name="create-a-search-dialog"></a>검색 대화 상자 만들기

봇의 프로젝트에서 새 `AzureSearchDialog` 클래스를 만들어 봇에서 Azure 서비스를 호출합니다. 이 새 클래스는 **Search.Dialogs** 프로젝트에서 `SearchDialog` 클래스를 상속해야 합니다. 이는 어려운 과제 중 대부분을 처리합니다. `GetTopRefiners()` 재정의를 사용하면 검색을 처음부터 시작하지 않고도 검색 개체의 상태를 유지하면서 검색 결과의 범위를 좁히거나 필터링할 수 있습니다. `TopRefiners` 배열에서 사용자 고유의 사용자 지정 구체화를 추가하면 사용자가 검색 결과를 필터링하거나 범위를 좁힐 수 있습니다. 

```cs
[Serializable]
public class AzureSearchDialog : SearchDialog
{
    private static readonly string[] TopRefiners = { "refiner1", "refiner2", "refiner3" }; // define your own custom refiners 

    public AzureSearchDialog(ISearchClient searchClient) : base(searchClient, multipleSelection: true)
    {
    }

    protected override string[] GetTopRefiners()
    {
        return TopRefiners;
    }
}
```

## <a name="define-the-response-data-model"></a>응답 데이터 모델 정의

`Search.Contracts` 프로젝트 내 **SearchHit.cs** 클래스는 Azure Search 응답에서 구문 분석하는 관련 데이터를 정의합니다. 봇의 경우 유일한 필수 포함 항목은 생성자에서 `PropertyBag` IDictionary를 선언 및 생성하는 것입니다. 봇의 요구 사항에 관련된 이 클래스의 다른 모든 속성을 정의할 수 있습니다. 

```cs
[Serializable]
public class SearchHit
{
    public SearchHit()
    {
        this.PropertyBag = new Dictionary<string, object>();
    }

    public IDictionary<string, object> PropertyBag { get; set; }

    // customize the fields below as needed 
    public string Key { get; set; }

    public string Title { get; set; }

    public string PictureUrl { get; set; }

    public string Description { get; set; }
}
```

## <a name="after-azure-search-responds"></a>Azure Search 응답 후 

Azure 서비스에 성공적으로 쿼리한 후, 봇이 사용자에게 표시하기 위해 관련 데이터를 검색하도록 검색 결과는 구문 분석되어야 합니다. 이를 사용하도록 설정하려면 `SearchResultMapper` 클래스를 만들어야 합니다. 생성자에서 만들어진 `GenericSearchResult` 개체는 각 결과가 봇의 데이터 모델에 각각 구문 분석된 후 결과 및 패싯을 각각 저장하는 목록 및 사전을 정의합니다. 

**SearchHit.cs**의 데이터 모델과 일치하도록 `ToSearchHit` 메서드의 속성을 동기화합니다. `ToSearchHit` 메서드가 실행되고, 응답의 모든 결과에 대해 새 `SearchHit`를 생성합니다.  

```cs
public class SearchResultMapper : IMapper<DocumentSearchResult, GenericSearchResult>
{
    public GenericSearchResult Map(DocumentSearchResult documentSearchResult)
    {
        var searchResult = new GenericSearchResult();

        searchResult.Results = documentSearchResult.Results.Select(r => ToSearchHit(r)).ToList();
        searchResult.Facets = documentSearchResult.Facets?.ToDictionary(kv => kv.Key, kv => kv.Value.Select(f => ToFacet(f)));

        return searchResult;
    }

    private static GenericFacet ToFacet(FacetResult facetResult)
    {
        return new GenericFacet
        {
            Value = facetResult.Value,
            Count = facetResult.Count.Value
        };
    }

    private static SearchHit ToSearchHit(SearchResult hit)
    {
        return new SearchHit
        {
            // custom properties defined in SearchHit.cs 
            Key = (string)hit.Document["id"],
            Title = (string)hit.Document["title"],
            PictureUrl = (string)hit.Document["thumbnail"],
            Description = (string)hit.Document["description"]
        };
    }
}
```
결과가 구문 분석되고 저장된 후에도 정보는 계속 사용자에게 표시되어야 합니다. `SearchHitStyler` 클래스는 `SearchHit` 클래스에서 데이터 모델을 수용하도록 관리되어야 합니다. 예를 들어 **SearchHit.cs** 클래스의 `Title`, `PictureUrl` 및 `Description` 속성은 새 카드 첨부파일을 만들기 위해 샘플 코드에서 사용됩니다. 다음 코드는 사용자에게 표시할 `GenericSearchResult` 결과 목록에서 모든 `SearchHit` 개체에 대해 새 카드 첨부 파일을 만듭니다.   

```cs
[Serializable]
public class SearchHitStyler : PromptStyler
{
    public override void Apply<T>(ref IMessageActivity message, string prompt, IReadOnlyList<T> options, IReadOnlyList<string> descriptions = null)
    {
        var hits = options as IList<SearchHit>;
        if (hits != null)
        {
            var cards = hits.Select(h => new ThumbnailCard
            {
                Title = h.Title,
                Images = new[] { new CardImage(h.PictureUrl) },
                Buttons = new[] { new CardAction(ActionTypes.ImBack, "Pick this one", value: h.Key) },
                Text = h.Description
            });

            message.AttachmentLayout = AttachmentLayoutTypes.Carousel;
            message.Attachments = cards.Select(c => c.ToAttachment()).ToList();
            message.Text = prompt;
        }
        else
        {
            base.Apply<T>(ref message, prompt, options, descriptions);
        }
    }
}
```
검색 결과가 사용자에게 표시되며 Azure Search를 성공적으로 봇에 추가되었습니다.

## <a name="samples"></a>샘플

.NET용 Bot Framework SDK를 사용하여 봇을 통해 Azure Search를 지원하는 방법을 보여주는 두 개의 전체 샘플은 GitHub에서 [부동산 봇 샘플](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-Search/RealEstateBot) 또는 [작업 나열 봇 샘플](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-Search/JobListingBot)을 참조하세요. 

## <a name="additional-resources"></a>추가 리소스

- [Azure Search][search]
- [대화 상자 개요](bot-builder-dotnet-dialogs.md)

[search]: /azure/search/search-what-is-azure-search
