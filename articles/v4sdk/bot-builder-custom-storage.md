---
title: 봇에 사용자 지정 스토리지 구현 | Microsoft Docs
description: Bot Framework SDK v4.0에서 사용자 지정 스토리지를 빌드하는 방법입니다.
keywords: 사용자 지정, 스토리지, 상태, 대화
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 41a33c20148e128efa1d10b72410eb06a6a94982
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905006"
---
# <a name="implement-custom-storage-for-your-bot"></a>봇에 사용자 지정 스토리지 구현

[!INCLUDE[applies-to](../includes/applies-to.md)]

한 개의 상호작용은 세 가지 영역으로 나뉩니다. 첫째, Azure Bot Service와 활동을 교환하는 것, 둘째, Store(저장소)를 통해 대화 상태를 로드하고 저장하는 것, 그리고 마지막으로 봇에서 작업을 수행하는 데 필요한 다른 백 엔드 서비스입니다.

![규모 확장 다이어그램](../media/scale-out/scale-out-interaction.png)

이 문서에서는 Azure Bot Service 및 Store와의 상호작용에 관한 의미 체계를 살펴봅니다.

Bot Framework에는 기본 구현이 포함되어 있습니다. 이 구현은 많은 애플리케이션의 요구 사항에 매우 적합하며, 이를 사용하기 위해 수행해야 하는 모든 작업은 몇 줄의 초기화 코드를 통해 해당 요소를 모두 연결하는 것입니다. 많은 샘플에서 이를 잘 보여 줍니다.

그러나 여기서의 목표는 기본 구현의 의미 체계가 애플리케이션에서 원하는 대로 제대로 작동하지 않을 때 수행할 수 있는 작업을 설명하는 것입니다. 기본 요점은 이 의미 체계가 프레임워크이며, 확정된 동작으로 미리 준비된 애플리케이션이 아니라는 것입니다. 즉 프레임워크의 많은 메커니즘을 구현하는 것이 기본 구현일 뿐이며 유일한 구현은 아니다.

특히 프레임워크는 Azure Bot Service를 통한 활동 교환과 모든 Bot 상태의 로드 및 저장 사이의 관계를 지시하지 않습니다, 단순히 기본값을 제공합니다. 이 점을 더 자세히 설명하기 위해 다른 의미 체계가 있는 대체 구현을 개발할 것입니다. 이 대체 솔루션은 프레임워크에서 똑같이 적합하며, 개발 중인 애플리케이션에 더 적합할 수도 있습니다. 이는 모두 시나리오에 따라 달라집니다.

## <a name="behavior-of-the-default-botframeworkadapter-and-storage-providers"></a>기본 BotFrameworkAdapter 및 스토리지 공급자의 동작

먼저, 다음 시퀀스 다이어그램과 같이 프레임워크 패키지의 일부로 제공되는 기본 구현을 살펴보겠습니다.

![규모 확장 다이어그램](../media/scale-out/scale-out-default.png)

활동을 받으면 봇에서 이 대화에 해당하는 상태를 로드합니다. 그런 다음, 이 상태와 방금 받은 활동을 사용하여 대화 논리를 실행합니다. 대화를 실행하는 과정에서 하나 이상의 아웃바운드 활동이 만들어 즉시 보냅니다. 대화 처리가 완료되면 봇에서 업데이트된 상태를 저장하여 이전 상태를 새 상태로 덮어씁니다.

이 동작에 문제가 될 수 있는 몇 가지 사항을 고려해 볼 가치가 있습니다.

첫째, 어떤 이유로 Save(저장) 작업이 실패하는 경우 응답을 본 사용자가 상태가 앞으로 이동했다고 느끼므로 상태는 채널에서 보이는 것과 암시적으로 동기화되지 않습니다. 이는 일반적으로 상태와 응답 메시지가 성공적인 경우보다 더 나쁩니다. 이로 인해 대화 설계에 영향을 미칠 수 있습니다. 예를 들어 대화에는 추가적이거나 그렇지 않으면 중복된 확인 교환이 포함될 수 있습니다. 

둘째, 구현이 여러 노드에 걸쳐 확장되어 배포되는 경우 실수로 상태를 덮어쓸 수 있습니다. 이는 대화에서 확인 메시지를 전송하는 채널에 활동을 보냈을 가능성이 있으므로 특히 혼란스러울 수 있습니다. 토핑에 대해 물어볼 때 사용자가 버섯을 추가하고 지체 없이 치즈를 추가하는 경우 후속 작업을 실행하는 여러 인스턴스가 있는 확장된 시나리오에서 봇을 실행하는 여러 머신에 동시에 보낼 수 있는 피자 주문 봇의 예를 고려해 보세요. 이 경우 한 머신이 다른 머신에서 기록한 상태를 덮어쓸 수 있는 "경합 상태"라고 하는 것이 있습니다. 하지만 시나리오에서는 응답이 이미 전송되었으므로 사용자는 버섯과 치즈가 모두 추가되었다는 확인 메시지를 받았습니다. 아쉽게도 피자가 도착하면 버섯 또는 치즈만 포함되고 둘 모두는 포함되지 않았을 것입니다.

## <a name="optimistic-locking"></a>낙관적 잠금

해결 방법은 상태 주위에 일부 잠금을 도입하는 것입니다. 여기서 사용할 특정 잠금 방식은 '낙관적 잠금'이라고 합니다. 왜냐하면 모든 것이 마치 각각 실행되는 것처럼 실행되게 하고 처리가 완료되면 동시성 위반을 감지하게 되기 때문입니다. 이는 복잡하게 들릴 수 있지만, 봇 프레임워크에서 클라우드 스토리지 기술과 올바른 확장 지점을 사용하여 구축하기가 매우 쉽습니다.

여기서는 ETag(엔터티 태그) 헤더에 기반한 표준 HTTP 메커니즘을 사용합니다. 이 메커니즘을 이해하는 것은 다음 코드를 이해하는 데 중요합니다. 다음 다이어그램에서는 시퀀스를 보여 줍니다.

![규모 확장 다이어그램](../media/scale-out/scale-out-precondition-failed.png)

이 다이어그램에서는 일부 리소스에 대한 업데이트를 수행하는 두 클라이언트의 경우를 보여 줍니다. 클라이언트에서 GET 요청을 발급하고 서버에서 리소스가 반환되는 경우 ETag 헤더가 수반됩니다. ETag 헤더는 리소스의 상태를 나타내는 불투명 값입니다. 리소스가 변경되면 ETag가 업데이트됩니다. 클라이언트에서 상태를 업데이트하면 서버에 이를 다시 게시하므로 이 요청을 통해 클라이언트에서 이전에 받은 ETag 값을 사전 조건 If-Match 헤더에 연결합니다. 이 ETag가 값과 일치하지 않으면 412 사전 조건 실패로 인해 서버에서 마지막으로 반환(모든 클라이언트에 대한 응답에서)한 사전 조건 확인이 실패합니다. 이 오류는 리소스가 업데이트되었다는 POST 요청을 수행하는 클라이언트에 대한 표시기입니다. 이 오류가 표시되면 클라이언트에 대한 일반적인 동작은 리소스를 다시 가져오고(GET), 원하는 업데이트를 적용하고, 리소스를 다시 게시(POST)하는 것입니다. 물론, 이 두 번째 POST는 다른 클라이언트에서 리소스를 제공하고 업데이트하지 않았다고 가정하고 클라이언트에서 다시 시도하는 경우에만 성공합니다.

이 프로세스는 리소스를 보유한 클라이언트에서 계속 처리하지만 리소스 자체가 다른 클라이언트에서 제한 없이 액세스 할 수 있다는 의미에서 "잠겨" 있지 않으므로 "낙관적"이라고 합니다. 처리가 완료될 때까지 리소스 상태에 대한 클라이언트 간의 경합은 확인되지 않습니다. 일반적으로 이 전략은 분산형 시스템에서 반대되는 "비관적" 접근 방식보다 더 적합합니다.

이제까지 살펴본 낙관적 잠금 메커니즘은 프로그램 논리를 안전하게 다시 시도할 수 있다고 가정하고 있습니다. 물론 여기서 고려해야 할 중요 사항은 외부 서비스 호출에 발생하는 상황입니다. 가장 좋은 해결 방법은 이러한 서비스가 멱등원(idempotent)이 될 수 있는 경우입니다. 컴퓨터 과학에서, idempotent 작업은 동일한 입력 매개 변수를 사용하여 두 번 이상 호출되는 경우 추가적인 효과가 없습니다. GET, PUT 및 DELETE를 구현하는 순수 HTTP REST 서비스가 이 설명에 적합합니다. 여기서 추론은 직관적입니다. 즉 처리를 다시 시도할 수 있으므로 다시 시도의 일환으로 다시 실행될 때 추가 효과가 없는 호출을 수행하는 것이 좋습니다. 이 토론을 위해 여기서는 이상적인 세계에 살고 있고, 이 문서의 시작 부분에 있는 시스템 그림의 오른쪽에 표시된 백 엔드 서비스가 모든 idempotent HTTP REST 서비스라고 가정하여 활동 교환만 집중적으로 살펴보겠습니다.

## <a name="buffering-outbound-activities"></a>아웃바운드 활동 버퍼링

활동을 보내는 것은 idempotent 작업이 아니며, 종단 간 시나리오에서 많은 의미가 있는 것도 명확하지 않습니다. 모든 활동이 완료되면 보기에 추가되거나 텍스트 음성 변환 에이전트에 말하는 메시지를 전달하는 경우가 많습니다.

이러한 활동을 보낼 때 방지해야 할 주요 사항은 여러 번 보내는 것입니다. 당면한 문제는 낙관적 잠금 메커니즘에서는 논리를 여러 번 다시 실행해야 한다는 것입니다. 해결 방법은 간단합니다. 즉 논리를 다시 실행하지 않는 것을 확신할 수 있을 때까지 대화의 아웃바운드 활동을 버퍼링해야 합니다. 이는 저장 작업이 성공적으로 완료될 때까지입니다. 다음과 같은 흐름을 살펴보겠습니다.

![규모 확장 다이어그램](../media/scale-out/scale-out-buffer.png)

대화 실행을 중심으로 다시 시도 루프를 작성할 수 있다고 가정할 때 Save 작업에서 사전 조건 실패가 발생하면 다음 동작이 발생합니다.

![규모 확장 다이어그램](../media/scale-out/scale-out-save.png)

이 메커니즘을 적용하고 이전의 예를 다시 살펴보면 주문에 피자 토핑이 추가된다는 잘못된 긍정 승인은 결코 표시되지 않습니다. 실제로 여러 머신에 걸쳐 배포를 확장했을 수 있지만 낙관적 잠금 체계를 사용하여 상태 업데이트를 효과적으로 직렬화했습니다. 그러나 피자 주문에서 항목 추가에 따른 승인은 이제 전체 상태를 정확히 반영하도록 작성할 수도 있습니다. 예를 들어 사용자가 "치즈"를 바로 입력하면 봇에서 "버섯"을 응답하기 전에 나오는 두 개의 응답은 "치즈로 토핑된 피자"와 "치즈와 버섯으로 토핑된 피자"가 될 수 있습니다.

시퀀스 다이어그램을 살펴보면 성공적인 Save 작업 후에 응답이 손실될 수 있다는 것을 알 수 있지만, 종단 간 통신에서는 어디서나 응답이 손실될 수 있습니다. 요점은 상태 관리 인프라에서 해결할 수 있는 문제가 아니라는 것입니다. 여기에는 더 높은 수준의 프로토콜과 아마도 채널 사용자와 관련된 프로토콜이 필요합니다. 예를 들어 봇에서 응답하지 않았다고 사용자에게 표시되는 경우 사용자가 최종적으로 다시 시도하거나 이러한 동작을 시도할 것으로 예상하는 것이 합리적입니다. 따라서 시나리오에서 이와 같이 일시적으로 중단되는 경우도 합리적이지만, 사용자가 잘못된 긍정 승인 또는 기타 의도하지 않은 메시지를 필터링할 수 있도록 예상하는 것은 매우 합리적이지 않습니다. 

새 사용자 지정 스토리지 솔루션에 이 모든 것을 통합하여 프레임워크의 기본 구현에서 수행하지 않는 세 가지 작업을 수행할 것입니다. 첫째, ETag를 사용하여 경합을 검색하고, 둘째, ETag 오류가 검색되면 처리를 다시 시도하며, 마지막으로 저장이 성공할 때까지 아웃바운드 활동을 버퍼링합니다. 이 문서의 나머지 부분에서는 이러한 세 부분의 구현에 대해 설명합니다.

## <a name="implementing-etag-support"></a>ETag 지원 구현

단위 테스트를 지원하기 위해 ETag 지원을 통해 새 저장소에 대한 인터페이스를 정의하는 것으로 시작합니다. 인터페이스가 있으면 하나는 네트워크에 연결할 필요 없이 메모리에서 실행되는 단위 테스트, 다른 하나는 프로덕션에 사용되는 두 가지 버전을 작성할 수 있습니다. 이 인터페이스를 사용하면 ASP.NET에서 제공하는 종속성 주입 메커니즘을 매우 쉽게 활용할 수 있습니다.

인터페이스는 Load(로드) 및 Save(저장) 메서드로 구성됩니다. 이 두 가지에는 모두 상태에 사용할 키가 있습니다. Load 작업은 데이터 및 연결된 ETag를 반환합니다. 그리고 Save 작업에서 이러한 항목을 가져옵니다. 또한 Save 작업은 부울을 반환합니다. 이 부울은 ETag가 일치하고 Save 작업이 성공했는지 여부를 나타냅니다. 이는 일반적인 오류 표시기가 아니라 사전 조건 실패의 특정 표시기를 위한 것이며, 다시 시도 루프의 형태로 표시기 주위에 제어 흐름 논리를 작성하게 되므로 이를 예외가 아닌 반환 코드로 모델링합니다.

이 최하위 수준의 스토리지 부분을 연결할 수 있도록 하려면 직렬화 요구 사항을 배치하지 않아야 하지만, 콘텐츠 저장이 JSON 형식이 되도록 지정하는 것이 좋습니다. 저장소 구현에서는 이러한 방식으로 content(콘텐츠) 형식을 설정할 수 있습니다. .NET에서 이 작업을 수행하는 가장 쉽고 자연스러운 방법은 인수 유형을 사용하는 것입니다. 특히 content 인수를 JObject로 입력합니다. JavaScript 또는 TypeScript에서 이는 일반 네이티브 개체일 뿐입니다.  

결과 인터페이스는 다음과 같습니다.

```csharp
public interface IStore
{
  Task<(JObject content, string eTag)> LoadAsync(string key);
  Task<bool> SaveAsync(string key, JObject content, string eTag);
}
```
Azure Blob Storage에 대해 이를 구현하는 것은 간단합니다.
```csharp
public class BlobStore : IStore
{
  private CloudBlobContainer _container;

  public BlobStore(string myAccountName, string myAccountKey, string containerName)
  {
    var storageCredentials = new StorageCredentials(myAccountName, myAccountKey);
    var cloudStorageAccount = new CloudStorageAccount(storageCredentials, useHttps: true);
    var client = cloudStorageAccount.CreateCloudBlobClient();
    _container = client.GetContainerReference(containerName);
  }

  public async Task<(JObject content, string eTag)> LoadAsync(string key)
  {
    var blob = _container.GetBlockBlobReference(key);
    try
    {
      var content = await blob.DownloadTextAsync();
      var obj = JObject.Parse(content);
      var eTag = blob.Properties.ETag;
      return (obj, eTag);
    }
    catch (StorageException e)
      when (e.RequestInformation.HttpStatusCode ==
        (int)HttpStatusCode.NotFound)
    {
      return (new JObject(), null);
    }
  }

  public async Task<bool> SaveAsync(string key, JObject obj, string eTag)
  {
    var blob = _container.GetBlockBlobReference(key);
    blob.Properties.ContentType = "application/json";
    var content = obj.ToString();
    if (eTag != null)
    {
      try
      {
        await blob.UploadTextAsync(content,
          new AccessCondition { IfMatchETag = eTag },
          new BlobRequestOptions(),
          new OperationContext());
      }
      catch (StorageException e)
        when (e.RequestInformation.HttpStatusCode ==
          (int)HttpStatusCode.PreconditionFailed)
      {
        return false;
      }
    }
    else
    {
      await blob.UploadTextAsync(content);
    }
    return true;
  }
}
```

알 수 있듯이 Azure Blob Storage는 여기서 실제 작업을 수행합니다. 특정 예외를 catch하고 호출 코드의 기대치를 충족하기 위해 해당 예외가 변환되는 방법에 주의합니다. 즉, Load의 '찾을 수 없음' 예외에서 null을 반환하고, Save의 '사전 조건 실패' 예외에서 부울을 반환해야 합니다.

이 소스 코드는 모두 해당 [샘플](https://aka.ms/scale-out)에서 사용할 수 있으며, 이 샘플에는 메모리 저장소 구현이 포함되어 있습니다.

## <a name="implementing-the-retry-loop"></a>다시 시도 루프 구현
루프의 기본 모양은 시퀀스 다이어그램에 표시된 동작에서 직접 파생됩니다.

Activity(활동)를 받으면 해당 대화에 해당하는 상태에 대한 키를 만듭니다. Activity 및 대화 상태 간의 관계를 변경하지 않으므로 기본 상태 구현과 동일한 방식으로 키를 만듭니다.

적절한 키가 만들어지면 해당 상태를 로드하려고 합니다(Load). 그런 다음, 봇의 대화를 실행하고 저장 작업을 시도합니다(Save). Save가 성공하면 대화 실행에 따른 아웃바운드 활동을 보내고 이 작업이 완료됩니다. 그렇지 않으면 다시 돌아가서 Load 이전의 전체 프로세스를 반복합니다. Load를 다시 수행하면 새 ETag가 제공되므로 다음에 Save 작업을 수행하면 성공하게 됩니다.

결과적으로 OnTurn 구현은 다음과 같습니다.
```csharp
public async Task OnTurnAsync(ITurnContext turnContext,
  CancellationToken cancellationToken = default(CancellationToken))
{
  // Create the storage key for this conversation.
  string key = $"{turnContext.Activity.ChannelId}/conversations/{turnContext.Activity.Conversation?.Id}";

  // The execution sits in a loop because there might be a retry if the save operation fails.
  while (true)
  {
    // Load any existing state associated with this key
    var (oldState, etag) = await _store.LoadAsync(key);

    // Run the dialog system with the old state and inbound activity,
    // resulting in a new state and outbound activities.
    var (activities, newState) = await DialogHost.RunAsync(_rootDialog, turnContext.Activity, oldState);

    // Save the updated state associated with this key.
    bool success = await _store.SaveAsync(key, newState, etag);

    // Following a successful save, send any outbound Activities, otherwise retry everything.
    if (success)
    {
      if (activities.Any())
      {
        // This is an actual send on the TurnContext we were given and so will actual do a send this time.
        await turnContext.SendActivitiesAsync(activities);
      }
      break;
    }
  }
}
```
대화 실행을 함수 호출로 모델링했습니다. 아마도 더 정교한 구현은 인터페이스를 정의하고 이 종속성을 주입할 수 있게 만들었을 것입니다. 그러나 여기서의 목적을 위해 대화를 모두 정적 함수 뒤에 배치하면 이 접근 방식의 기능적 특성을 강조합니다. 일반적으로 중요한 부분이 기능하게 되도록 구현을 구성하면 네트워크에서 성공적으로 작동하는 데 매우 적합합니다.


## <a name="implementing-outbound-activity-buffering"></a>아웃바운드 활동 버퍼링 구현 

다음 요구 사항은 Save 작업이 성공적으로 수행될 때까지 아웃바운드 활동을 버퍼링한다는 것입니다. 이렇게 하려면 사용자 지정 BotAdapter 구현이 필요합니다. 다음 코드에서는 Activity를 보내는 대신 목록에 Activity를 추가하는 추상 SendActivity 함수를 구현합니다. 호스팅할 대화는 더 현명하지 않습니다.
이 특정 시나리오에서는 UpdateActivity 및 DeleteActivity 작업이 지원되지 않으므로 이러한 메서드에서 '구현되지 않음'만 throw합니다. 또한 SendActivity에서 반환되는 값에는 관심이 없습니다. 이는 활동에 대한 업데이트를 보내야 하는 시나리오(예: 채널에 표시된 카드의 단추를 사용하지 않도록 설정하는 경우)의 일부 채널에서 사용합니다. 이러한 메시지 교환은 특히 상태가 필요할 때 복잡해질 수 있으며, 이는 이 문서의 범위를 벗어납니다. 사용자 지정 BotAdapter의 전체 구현은 다음과 같습니다.

```csharp
public class DialogHostAdapter : BotAdapter
{
  private List<Activity> _response = new List<Activity>();

  public IEnumerable<Activity> Activities => _response;

  public override Task<ResourceResponse[]> SendActivitiesAsync(ITurnContext turnContext,
    Activity[] activities, CancellationToken cancellationToken)
  {
    foreach (var activity in activities)
    {
      _response.Add(activity);
    }
    return Task.FromResult(new ResourceResponse[0]);
  }

  public override Task DeleteActivityAsync(ITurnContext turnContext,
    ConversationReference reference, CancellationToken cancellationToken)
  {
    throw new NotImplementedException();
  }

  public override Task<ResourceResponse> UpdateActivityAsync(ITurnContext turnContext,
    Activity activity, CancellationToken cancellationToken)
  {
    throw new NotImplementedException();
  }
}
```
이제 수행해야 할 나머지 작업인 통합은 이러한 다양한 새 조각을 모두 결합하여 기존 프레임워크에 연결하기만 하면 됩니다. 기본 다시 시도 루프는 IBot OnTurn 기능에만 있습니다. 여기에는 테스트를 위해 종속성을 주입할 수 있는 사용자 지정 IStore 구현이 포함되어 있습니다. 모든 대화 호스팅 코드를 하나의 public static 함수를 공개하는 DialogHost라는 클래스에 배치했습니다. 이 함수는 인바운드 활동과 이전 상태를 가져온 다음, 결과 활동과 새 상태를 반환하도록 정의되어 있습니다.

이 함수에서 가장 먼저 수행해야 하는 작업은 앞에서 소개한 사용자 지정 BotAdapter를 만드는 것입니다. 그런 다음, DialogSet 및 DialogContext를 만들고, 일반적인 Continue 또는 Begin 흐름을 수행하는 것과 똑같은 방식으로 대화를 실행합니다. 다루지 않은 유일한 부분은 사용자 지정 접근자가 필요하다는 것입니다. 이는 대화 상태를 대화 시스템에 쉽게 전달할 수 있게 하는 매우 단순한 쐐기입니다. 접근자는 대화 시스템에서 작업할 때 참조 의미 체계를 사용하므로 핸들을 전달하기만 하면 됩니다. 더 명확히 하기 위해 참조 의미 체계에 사용하는 클래스 템플릿을 제한했습니다.

계층화에 매우 주의하고, 다양한 구현에서 서로 다르게 직렬화할 수 있는 경우 플러그형 스토리지 계층 내에 JsonSerialization이 필요하지 않았으므로 호스팅 코드에 JsonSerialization 인라인 방식으로 배치합니다.

드라이버 코드는 다음과 같습니다.
```csharp
public class DialogHost
{
  private static readonly JsonSerializer StateJsonSerializer = new JsonSerializer()
    { TypeNameHandling = TypeNameHandling.All };

  public static async Task<Tuple<Activity[], JObject>> RunAsync(Dialog rootDialog,
    Activity activity, JObject oldState)
  {
    // A custom adapter and corresponding TurnContext that buffers any messages sent.
    var adapter = new DialogHostAdapter();
    var turnContext = new TurnContext(adapter, activity);

    // Run the dialog using this TurnContext with the existing state.
    JObject newState = await RunTurnAsync(rootDialog, turnContext, oldState);

    // The result is a set of activities to send and a replacement state.
    return Tuple.Create(adapter.Activities.ToArray(), newState);
  }

  private static async Task<JObject> RunTurnAsync(Dialog rootDialog,
    TurnContext turnContext, JObject state)
  {
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
      // If we have some state, deserialize it. (This mimics the shape produced by BotState.cs.)
      var dialogState = state?[nameof(DialogState)]?.ToObject<DialogState>(StateJsonSerializer);

      // A custom accessor is used to pass a handle on the state to the dialog system.
      var accessor = new RefAccessor<DialogState>(dialogState);

      // The following is regular dialog driver code.
      var dialogs = new DialogSet(accessor);
      dialogs.Add(rootDialog);

      var dialogContext = await dialogs.CreateContextAsync(turnContext);
      var results = await dialogContext.ContinueDialogAsync();

      if (results.Status == DialogTurnStatus.Empty)
      {
        await dialogContext.BeginDialogAsync("root");
      }

      // Serialize the result, and put its value back into a new JObject.
      return new JObject
      {
        { nameof(DialogState), JObject.FromObject(accessor.Value, StateJsonSerializer) }
      };
    }

    return state;
  }
}
```
마지막으로, 상태가 참조로 사용되므로 사용자 지정 접근자인 Set을 구현하기만 하면 됩니다.
```csharp
public class RefAccessor<T> : IStatePropertyAccessor<T> where T : class
{
  public RefAccessor(T value)
  {
    Value = value;
  }

  public T Value { get; private set; }

  public string Name => nameof(T);

  public Task<T> GetAsync(ITurnContext turnContext, Func<T> defaultValueFactory = null,
    CancellationToken cancellationToken = default(CancellationToken))
  {
    if (Value == null)
    {
      if (defaultValueFactory == null)
      {
        throw new KeyNotFoundException();
      }
      else
      {
        Value = defaultValueFactory();
      }
    }
    return Task.FromResult(Value);
  }

  public Task DeleteAsync(ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken))
  {
    throw new NotImplementedException();
  }

  public Task SetAsync(ITurnContext turnContext, T value,
    CancellationToken cancellationToken = default(CancellationToken))
  {
    throw new NotImplementedException();
  }
}
```

## <a name="additional-resources"></a>추가 리소스
이 문서에서 사용된 [C#](http://aka.ms/scale-out) 샘플 코드는 GitHub에서 사용할 수 있습니다.

