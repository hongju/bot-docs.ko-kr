---
title: 동일한 .NET Framework 프로젝트 내에서 기존 봇 마이그레이션 | Microsoft Docs
description: 기존 v3 봇을 가져와서 동일한 프로젝트를 사용하여 v4 SDK로 마이그레이션합니다.
keywords: 봇 마이그레이션, Formflow, 대화, v3 봇
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c486970a3880c95ff10e9d4bb59b50a5600c7343
ms.sourcegitcommit: 178140eb060d71803f1c6357dd5afebd7f44fe1d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/17/2019
ms.locfileid: "65855884"
---
# <a name="migrate-a-net-sdk-v3-bot-to-v4"></a>.NET SDK v3 봇을 v4로 마이그레이션

이 문서에서는 _프로젝트 형식을 변환하지 않고_ v3 [ContosoHelpdeskChatBot](https://github.com/Microsoft/intelligent-apps/tree/master/ContosoHelpdeskChatBot/ContosoHelpdeskChatBot)을 v4 봇으로 변환합니다. 이것은 .NET Framework 프로젝트로 그대로 유지됩니다.
이 변환은 다음 단계로 구분됩니다.

1. NuGet 패키지 업데이트 및 설치
1. Global.asax.cs 파일 업데이트
1. MessagesController 클래스 업데이트
1. 대화 변환

<!--TODO: Link to the converted bot...[ContosoHelpdeskChatBot](https://github.com/EricDahlvang/intelligent-apps/tree/v4netframework/ContosoHelpdeskChatBot).-->

Bot Framework SDK v4는 SDK v3과 동일한 기본 REST API를 기반으로 합니다. 그러나 SDK v4는 이전 버전의 SDK를 리팩터링하여 개발자가 자신의 봇을 더 유연하게 제어할 수 있도록 합니다. SDK의 주요 변경 내용은 다음과 같습니다.

- 상태가 상태 관리 개체 및 속성 접근자를 통해 관리됩니다.
- 턴 처리기 설정 및 활동 전달이 변경되었습니다.
- Scorable 클래스가 더 이상 존재하지 않습니다. 대화에 제어를 전달하기 전에 턴 처리기에서 "global" 명령을 확인할 수 있습니다.
- 이전 버전과 매우 다른 새 Dialogs 라이브러리입니다. 구성 요소, 폭포 대화 및 v4용 Formflow 대화의 커뮤니티 구현을 사용하여 이전 대화를 새 대화 시스템으로 변환해야 합니다.

특정 변경에 대한 자세한 내용은 [v3 및 v4 .NET SDK 간의 차이점](migration-about.md)을 참조하세요.

## <a name="update-and-install-nuget-packages"></a>NuGet 패키지 업데이트 및 설치

1. **Microsoft.Bot.Builder.Azure**를 안정적인 최신 버전으로 업데이트합니다.

    그러면 **Microsoft.Bot.Builder** 및 **Microsoft.Bot.Connector** 패키지도 종속성이므로 업데이트됩니다.

1. **Microsoft.Bot.Builder.History** 패키지를 삭제합니다. 이 패키지는 v4 SDK의 일부가 아닙니다.
1. **Autofac.WebApi2**를 추가합니다.

    이 패키지는 ASP.NET에 종속성 주입을 지원하기 위해 사용합니다.

1. **Bot.Builder.Community.Dialogs.Formflow**를 추가합니다.

    이 패키지는 v3 Formflow 정의 파일에서 v4 대화를 빌드하기 위한 커뮤니티 라이브러리입니다. **Microsoft.Bot.Builder.Dialogs**도 종속성 중 하나이므로 설치됩니다.

이 시점에서 빌드하면 컴파일러 오류가 발생하지만, 무시할 수 있습니다. 변환이 완료되면 작업 코드가 준비됩니다.

<!--
## Add a BotDataBag class

This file will contain wrappers to add a v3-style **IBotDataBag** to make dialog conversion simpler.

```csharp
using System.Collections.Generic;

namespace ContosoHelpdeskChatBot
{
    public class BotDataBag : Dictionary<string, object>, IBotDataBag
    {
        public bool RemoveValue(string key)
        {
            return base.Remove(key);
        }

        public void SetValue<T>(string key, T value)
        {
            this[key] = value;
        }

        public bool TryGetValue<T>(string key, out T value)
        {
            if (!ContainsKey(key))
            {
                value = default(T);
                return false;
            }

            value = (T)this[key];

            return true;
        }
    }

    public interface IBotDataBag
    {
        int Count { get; }

        void Clear();

        bool ContainsKey(string key);

        bool RemoveValue(string key);

        void SetValue<T>(string key, T value);

        bool TryGetValue<T>(string key, out T value);
    }
}
```
-->

## <a name="update-your-globalasaxcs-file"></a>Global.asax.cs 파일 업데이트

스캐폴딩 중 일부가 변경되었으며 v4에서 [상태 관리](../bot-builder-concept-state.md) 인프라의 일부를 직접 설정해야 합니다. 예를 들어 v4에서는 봇 어댑터를 사용하여 인증을 처리하고, 활동을 봇 코드로 전달하며, 상태 속성을 미리 선언합니다.

이제 v4에서 대화 지원에 필요한 `DialogState`에 대한 상태 속성을 만듭니다. 종속성 주입을 사용하여 컨트롤러와 봇 코드에 필요한 정보를 가져옵니다.

**Global.asax.cs**에서,

1. `using` 문을 업데이트합니다.
    ```csharp
    using System.Configuration;
    using System.Reflection;
    using System.Web.Http;
    using Autofac;
    using Autofac.Integration.WebApi;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    ```
1. **Application_Start** 메서드에서 다음 줄을 제거합니다
    ```csharp
    BotConfig.UpdateConversationContainer();
    this.RegisterBotModules();
    ```
    그리고 다음 줄을 삽입합니다.
    ```csharp
    GlobalConfiguration.Configure(BotConfig.Register);
    ```
1. 더 이상 참조되지 않는 **RegisterBotModules** 메서드를 제거합니다.
1. **BotConfig.UpdateConversationContainer** 메서드를 다음 **BotConfig.Register** 메서드로 바꿉니다. 이 메서드는 종속성 주입을 지원하는 데 필요한 개체를 등록합니다.
    > [!NOTE]
    > 이 봇은 _사용자_ 또는 _개인 대화_ 상태를 사용하지 않습니다. 여기서는 이러한 상태가 포함된 줄이 주석 처리됩니다.
    ```csharp
    public static void Register(HttpConfiguration config)
    {
        ContainerBuilder builder = new ContainerBuilder();
        builder.RegisterApiControllers(Assembly.GetExecutingAssembly());

        SimpleCredentialProvider credentialProvider = new SimpleCredentialProvider(
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppIdKey],
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppPasswordKey]);

        builder.RegisterInstance(credentialProvider).As<ICredentialProvider>();

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create Conversation State object.
        // The Conversation State object is where we persist anything at the conversation-scope.
        ConversationState conversationState = new ConversationState(dataStore);
        builder.RegisterInstance(conversationState).As<ConversationState>();

        //var userState = new UserState(dataStore);
        //var privateConversationState = new PrivateConversationState(dataStore);

        // Create the dialog state property acccessor.
        IStatePropertyAccessor<DialogState> dialogStateAccessor
            = conversationState.CreateProperty<DialogState>(nameof(DialogState));
        builder.RegisterInstance(dialogStateAccessor).As<IStatePropertyAccessor<DialogState>>();

        IContainer container = builder.Build();
        AutofacWebApiDependencyResolver resolver = new AutofacWebApiDependencyResolver(container);
        config.DependencyResolver = resolver;
    }
    ```

## <a name="update-your-messagescontroller-class"></a>MessagesController 클래스 업데이트

여기서는 v4에서 턴 처리기가 수행되므로 많이 변경해야 합니다. 턴 처리기 자체를 제외한 대부분은 상용구로 생각할 수 있습니다. **Controllers\MessagesController.cs** 파일에서,

1. `using` 문을 업데이트합니다.
    ```csharp
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Web.Http;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Dialogs;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    ```
1. 클래스에서 `[BotAuthentication]` 특성을 제거합니다. v4에서 봇의 어댑터는 인증을 처리합니다.
1. 다음 필드를 추가합니다. **ConversationState**는 대화의 범위가 지정된 상태를 관리하며, v4에서 대화를 지원하려면 **IStatePropertyAccessor\<DialogState>** 가 필요합니다.
    ```csharp
    private readonly ConversationState _conversationState;
    private readonly ICredentialProvider _credentialProvider;
    private readonly IStatePropertyAccessor<DialogState> _dialogData;

    private readonly DialogSet _dialogs;
    ```
1. 다음과 같은 생성자를 추가합니다.
    - 인스턴스 필드를 초기화합니다.
    - ASP.NET 종속성 주입을 사용하여 매개 변수 값을 가져옵니다. (이를 지원하기 위해 **Global.asax.cs**에 이러한 클래스의 인스턴스를 등록했습니다.)
    - 만들고 있는 대화 컨텍스트의 원본이 되는 대화 집합을 만들고 초기화합니다. (이 작업은 v4에서 명시적으로 수행해야 합니다.)
    ```csharp
    public MessagesController(
        ConversationState conversationState,
        ICredentialProvider credentialProvider,
        IStatePropertyAccessor<DialogState> dialogData)
    {
        _conversationState = conversationState;
        _dialogData = dialogData;
        _credentialProvider = credentialProvider;

        _dialogs = new DialogSet(dialogData);
        _dialogs.Add(new RootDialog(nameof(RootDialog)));
    }
    ```
1. **Post** 메서드의 본문을 바꿉니다. 여기서는 어댑터를 만들고 이를 사용하여 메시지 루프(턴 처리기)를 호출합니다. 봇의 상태 변경 내용을 저장하기 위해 각 턴의 끝에 `SaveChangesAsync`를 사용합니다.

    ```csharp
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {

        var botFrameworkAdapter = new BotFrameworkAdapter(_credentialProvider);

        var invokeResponse = await botFrameworkAdapter.ProcessActivityAsync(
            Request.Headers.Authorization?.ToString(),
            activity,
            OnTurnAsync,
            default(CancellationToken));

        if (invokeResponse == null)
        {
            return Request.CreateResponse(HttpStatusCode.OK);
        }
        else
        {
            return Request.CreateResponse(invokeResponse.Status);
        }
    }
    ```
1. 봇의 [턴 처리기](../bot-builder-basics.md#the-activity-processing-stack) 코드가 포함된 **OnTurnAsync** 메서드를 추가합니다.
    > [!NOTE]
    > v4에는 Scorable이 없습니다. 활성 대화를 계속하기 전에 봇의 턴 처리기에서 사용자로부터 받은 `cancel` 메시지를 확인합니다.
    ```csharp
    protected async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    {
        // We're only handling message activities in this bot.
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            // Create the dialog context for our dialog set.
            DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

            // Globally interrupt the dialog stack if the user sent 'cancel'.
            if (turnContext.Activity.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
            {
                Activity reply = turnContext.Activity.CreateReply($"Ok restarting conversation.");
                await turnContext.SendActivityAsync(reply);
                await dc.CancelAllDialogsAsync();
            }

            try
            {
                // Continue the active dialog, if any. If we just cancelled all dialog, the
                // dialog stack will be empty, and this will return DialogTurnResult.Empty.
                DialogTurnResult dialogResult = await dc.ContinueDialogAsync();
                switch (dialogResult.Status)
                {
                    case DialogTurnStatus.Empty:
                        // There was no active dialog in the dialog stack; start the root dialog.
                        await dc.BeginDialogAsync(nameof(RootDialog));
                        break;

                    case DialogTurnStatus.Complete:
                        // The last dialog on the stack completed and the stack is empty.
                        await dc.EndDialogAsync();
                        break;

                    case DialogTurnStatus.Waiting:
                    case DialogTurnStatus.Cancelled:
                        // The active dialog is waiting for a response from the user, or all
                        // dialogs were cancelled and the stack is empty. In either case, we
                        // don't need to do anything here.
                        break;
                }
            }
            catch (FormCanceledException)
            {
                // One of the dialogs threw an exception to clear the dialog stack.
                await turnContext.SendActivityAsync("Cancelled.");
                await dc.CancelAllDialogsAsync();
                await dc.BeginDialogAsync(nameof(RootDialog));
            }
        }
    }
    ```
1. _메시지_ 활동만 처리하므로 **HandleSystemMessage** 메서드는 삭제할 수 있습니다.

### <a name="delete-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>CancelScorable 및 GlobalMessageHandlersBotModule 클래스 삭제

v4에 Scorable이 존재하지 않고 `cancel` 메시지에 대응하도록 턴 처리기를 업데이트했으므로 **CancelScorable**(**Dialogs\CancelScorable.cs**) 및 **GlobalMessageHandlersBotModule** 클래스를 삭제할 수 있습니다.

## <a name="convert-your-dialogs"></a>대화 변환

원래 대화를 많이 변경하여 v4 SDK로 마이그레이션합니다. 지금은 컴파일러 오류에 대해 걱정하지 마세요. 변환이 완료되면 이러한 문제가 해결됩니다.
필요 이상으로 원래 코드를 수정하지 않기 위해 마이그레이션을 완료한 후에도 컴파일러 경고가 남아 있습니다.

모든 대화는 v3의 `IDialog<object>` 인터페이스를 구현하는 대신 `ComponentDialog`에서 파생됩니다.

이 봇에는 변환해야 하는 다음 네 개의 대화가 있습니다.

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | 옵션을 표시하고 다른 대화를 시작합니다. |
| [InstallAppDialog](#update-the-install-app-dialog) | 머신에 앱을 설치하라는 요청을 처리합니다. |
| [LocalAdminDialog](#update-the-local-admin-dialog) | 로컬 머신 관리자 권한에 대한 요청을 처리합니다. |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | 암호 재설정에 대한 요청을 처리합니다. |

이러한 대화는 입력을 수집하지만, 머신에서는 이러한 작업을 수행하지 않습니다.

### <a name="make-solution-wide-dialog-changes"></a>솔루션 수준 대화 변경

1. 전체 솔루션에서 나타나는 모든 `IDialog<object>` 항목을 `ComponentDialog`로 바꿉니다.
1. 전체 솔루션에서 나타나는 모든 `IDialogContext` 항목을 `DialogContext`로 바꿉니다.
1. 각 대화 클래스에서 `[Serializable]` 특성을 제거합니다.

대화 내의 제어 흐름과 메시징은 더 이상 동일한 방식으로 처리되지 않으므로 각 대화를 변환할 때 이를 수정해야 합니다.

| 작업(Operation) | v3 코드 | v4 코드 |
| :--- | :--- | :--- |
| 대화의 시작 처리 | `IDialog.StartAsync` 구현 | 폭포 대화의 첫 번째 단계로 설정 또는 `Dialog.BeginDialogAsync` 구현 |
| 대화의 연속 처리 | `IDialogContext.Wait` 호출 | 폭포 대화에 추가 단계 추가 또는 `Dialog.ContinueDialogAsync` 구현 |
| 사용자에게 메시지 보내기 | `IDialogContext.PostAsync` 호출 | `ITurnContext.SendActivityAsync` 호출 |
| 자식 대화 시작 | `IDialogContext.Call` 호출 | `DialogContext.BeginDialogAsync` 호출 |
| 현재 대화에 대한 완료 신호 | `IDialogContext.Done` 호출 | `DialogContext.EndDialogAsync` 호출 |
| 사용자의 입력 가져오기 | `IAwaitable<IMessageActivity>` 매개 변수 사용 | 폭포 내에서 프롬프트 사용 또는 `ITurnContext.Activity` 사용 |

v4 코드에 대한 참고 사항:

- `DialogContext.Context` 속성을 사용하여 현재 턴 컨텍스트를 가져옵니다.
- 폭포 단계에는 `DialogContext`에서 파생된 `WaterfallStepContext` 매개 변수가 있습니다.
- 모든 구체적인 대화 및 프롬프트 클래스는 `Dialog` 추상 클래스에서 파생됩니다.
- 구성 요소 대화를 만들 때 ID를 할당합니다. 대화 집합의 각 대화에는 해당 집합 내에서 고유한 ID가 할당되어야 합니다.

### <a name="update-the-root-dialog"></a>루트 대화 업데이트

이 봇에서 루트 대화는 사용자에게 옵션 집합에서 선택하도록 요청하는 메시지를 표시한 다음, 해당 선택 항목에 따라 자식 대화를 시작합니다. 그런 다음, 대화의 수명 동안 반복됩니다.

- 기본 흐름을 v4 SDK의 새로운 개념인 폭포 대화로 설정할 수 있습니다. 고정된 일단의 단계를 순서대로 실행합니다. 자세한 내용은 [순차적 대화 흐름 구현](~/v4sdk/bot-builder-dialog-manage-conversation-flow.md)을 참조하세요.
- 프롬프트는 이제 프롬프트 클래스를 통해 처리됩니다. 이 클래스는 입력을 요청하고, 최소한의 일부 처리 및 유효성 검사를 수행하며, 값을 반환하는 짧은 자식 대화입니다. 자세한 내용은 [대화 프롬프트를 사용하여 사용자 입력 수집](~/v4sdk/bot-builder-prompts.md)을 참조하세요.

**Dialogs/RootDialog.cs** 파일에서,

1. `using` 문을 업데이트합니다.
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. `HelpdeskOptions` 옵션을 문자열 목록에서 선택 목록으로 변환해야 합니다. 여기서는 선택 번호(목록에서), 선택 값 또는 선택 항목의 동의어를 유효한 입력으로 수락하는 선택 프롬프트가 사용됩니다.
    ```csharp
    private static List<Choice> HelpdeskOptions = new List<Choice>()
    {
        new Choice(InstallAppOption) { Synonyms = new List<string>(){ "install" } },
        new Choice(ResetPasswordOption) { Synonyms = new List<string>(){ "password" } },
        new Choice(LocalAdminOption)  { Synonyms = new List<string>(){ "admin" } }
    };
    ```
1. 생성자를 추가합니다. 이 코드는 다음을 수행합니다.
   - 만들어지는 ID가 대화의 각 인스턴스에 할당됩니다. 대화 ID는 대화가 추가되는 대화 집합의 일부입니다. **MessageController** 클래스에서 초기화된 대화 집합이 봇에 있음을 상기하세요. 각 `ComponentDialog`에는 고유한 대화 ID 집합이 있는 자체의 내부 대화 집합이 있습니다.
   - 선택 프롬프트를 포함한 다른 대화를 자식 대화로 추가합니다. 여기서는 각 대화 ID에 대한 클래스 이름만 사용하고 있습니다.
   - 3단계 폭포 대화를 정의합니다. 이 대화는 잠시 후에 구현하겠습니다.
     - 대화는 먼저 사용자에게 수행할 작업을 선택하도록 요청하는 메시지를 표시합니다.
     - 그런 다음, 해당 선택 항목과 연결된 자식 대화를 시작합니다.
     - 그리고, 마지막으로 다시 시작합니다.
   - 폭포의 각 단계는 대리자이며, 가능한 한 원래 대화의 기존 코드를 사용하여 폭포의 다음 단계를 구현합니다.
   - 구성 요소 대화 상자를 시작하면 해당 _초기 대화 상자_가 시작됩니다. 기본적으로 이 대화는 구성 요소 대화에 추가된 첫 번째 자식 대화입니다. 다른 자식 대화를 초기 대화로 할당하려면 구성 요소의 `InitialDialogId` 속성을 수동으로 설정해야 합니다.
    ```csharp
    public RootDialog(string id)
        : base(id)
    {
        AddDialog(new WaterfallDialog("choiceswaterfall", new WaterfallStep[]
            {
                PromptForOptionsAsync,
                ShowChildDialogAsync,
                ResumeAfterAsync,
            }));
        AddDialog(new InstallAppDialog(nameof(InstallAppDialog)));
        AddDialog(new LocalAdminDialog(nameof(LocalAdminDialog)));
        AddDialog(new ResetPasswordDialog(nameof(ResetPasswordDialog)));
        AddDialog(new ChoicePrompt("options"));
    }
    ```
1. **StartAsync** 메서드는 삭제할 수 있습니다. 구성 요소 대화가 시작되면 해당 _초기_ 대화가 자동으로 시작됩니다. 이 경우 이 대화는 생성자에서 정의한 폭포 대화입니다. 또한 첫 번째 단계에서 자동으로 시작됩니다.
1. **MessageReceivedAsync** 및 **ShowOptions** 메서드를 삭제하고 폭포의 첫 번째 단계로 바꿉니다. 이 두 가지 메서드는 사용자에게 인사하고 사용 가능한 옵션 중 하나를 선택하도록 요청했습니다.
   - 여기서는 선택 목록을 볼 수 있으며, 인사말 및 오류 메시지가 선택 프롬프트에 대한 호출에서 옵션으로 제공됩니다.
   - 선택 프롬프트가 완료되면 폭포 대화에서 다음 단계로 계속 진행되므로 대화에서 호출할 다음 메서드를 지정할 필요가 없습니다.
   - 선택 프롬프트는 유효한 입력을 받거나 전체 대화 스택이 취소될 때까지 반복됩니다.
    ```csharp
    private async Task<DialogTurnResult> PromptForOptionsAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Prompt the user for a response using our choice prompt.
        return await stepContext.PromptAsync(
            "options",
            new PromptOptions()
            {
                Choices = HelpdeskOptions,
                Prompt = MessageFactory.Text(GreetMessage),
                RetryPrompt = MessageFactory.Text(ErrorMessage)
            },
            cancellationToken);
    }
    ```
1. **OnOptionSelected**를 폭포의 두 번째 단계로 대체할 수 있습니다. 여전히 사용자의 입력에 따라 자식 대화를 시작합니다.
   - 선택 프롬프트에서 `FoundChoice` 값을 반환합니다. 이 값은 단계 컨텍스트의 `Result` 속성에 표시됩니다. 대화 스택에서는 모든 반환 값을 개체로 처리합니다. 반환 값이 대화 중 하나에서 가져온 것이면 개체의 값 형식을 알 수 있습니다. 각 프롬프트 유형에서 반환하는 목록은 [프롬프트 유형](../bot-builder-concept-dialog.md#prompt-types)을 참조하세요.
   - 선택 프롬프트에서 예외를 throw하지 않으므로 try-catch 블록을 제거할 수 있습니다.
   - 이 메서드는 항상 적절한 값을 반환하도록 fall을 반복적으로 추가해야 합니다. 이 코드는 결코 적중되지 않아야 하지만, 해당 코드가 적중되면 대화가 "정상적으로 실패"할 수 있습니다.
    ```csharp
    private async Task<DialogTurnResult> ShowChildDialogAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // string optionSelected = await userReply;
        string optionSelected = (stepContext.Result as FoundChoice).Value;

        switch (optionSelected)
        {
            case InstallAppOption:
                //context.Call(new InstallAppDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(InstallAppDialog),
                    cancellationToken);
            case ResetPasswordOption:
                //context.Call(new ResetPasswordDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(ResetPasswordDialog),
                    cancellationToken);
            case LocalAdminOption:
                //context.Call(new LocalAdminDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(LocalAdminDialog),
                    cancellationToken);
        }

        // We shouldn't get here, but fail gracefully if we do.
        await stepContext.Context.SendActivityAsync(
            "I don't recognize that option.",
            cancellationToken: cancellationToken);
        // Continue through to the next step without starting a child dialog.
        return await stepContext.NextAsync(cancellationToken: cancellationToken);
    }
    ```
1. 마지막으로, 이전의 **ResumeAfterOptionDialog** 메서드를 폭포의 마지막 단계로 바꿉니다.
    - 원래 대화에서 수행한 대로 대화를 종료하고 티켓 번호를 반환하는 대신, 스택에서 원래 인스턴스를 자체의 새 인스턴스로 바꿔 폭포를 다시 시작합니다. 원래 앱에서는 항상 반환 값(티켓 번호)을 무시하고 루트 대화를 다시 시작하므로 이 작업을 수행할 수 있습니다.
    ```csharp
    private async Task<DialogTurnResult> ResumeAfterAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        try
        {
            //var message = await userReply;
            var message = stepContext.Context.Activity;

            int ticketNumber = new Random().Next(0, 20000);
            //await context.PostAsync($"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.");
            await stepContext.Context.SendActivityAsync(
                $"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.",
                cancellationToken: cancellationToken);

            //context.Done(ticketNumber);
        }
        catch (Exception ex)
        {
            // await context.PostAsync($"Failed with message: {ex.Message}");
            await stepContext.Context.SendActivityAsync(
                $"Failed with message: {ex.Message}",
                cancellationToken: cancellationToken);

            // In general resume from task after calling a child dialog is a good place to handle exceptions
            // try catch will capture exceptions from the bot framework awaitable object which is essentially "userReply"
            logger.Error(ex);
        }

        // Replace on the stack the current instance of the waterfall with a new instance,
        // and start from the top.
        return await stepContext.ReplaceDialogAsync(
            "choiceswaterfall",
            cancellationToken: cancellationToken);
    }
    ```

### <a name="update-the-install-app-dialog"></a>설치 앱 대화 업데이트

설치 앱 대화에서는 4단계 폭포 대화로 설정된 몇 가지 논리적 작업을 수행합니다. 기존 코드를 폭포 단계로 팩터링하는 방법은 각 대화에 대한 논리적 연습입니다. 각 단계마다 코드의 원본이 되는 원래 메서드가 표시됩니다.

1. 사용자에게 검색 문자열을 요청합니다.
1. 데이터베이스에서 잠재적인 일치 항목을 쿼리합니다.
   - 하나의 적중 항목이 있는 경우 이 항목을 선택하고 계속합니다.
   - 여러 개의 적중 항목이 있는 경우 사용자에게 이러한 항목 중 하나를 선택하도록 요청합니다.
   - 적중 항목이 없는 경우 대화가 종료됩니다.
1. 머신에 앱을 설치하도록 사용자에게 요청합니다.
1. 정보를 데이터베이스에 기록하고 확인 메시지를 보냅니다.

**Dialogs/InstallAppDialog.cs** 파일에서,

1. `using` 문을 업데이트합니다.
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. 프롬프트 및 대화에 사용할 ID 상수를 정의합니다. 이렇게 하면 사용할 문자열이 한 곳에 정의되므로 대화 코드를 더 쉽게 유지 관리할 수 있습니다.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainId = "mainDialog";
    private const string TextId = "textPrompt";
    private const string ChoiceId = "choicePrompt";
    ```
1. 대화 상태를 추적하는 데 사용할 키에 대한 상수를 정의합니다.
    ```csharp
    // Set up keys for managing collected information.
    private const string InstallInfo = "installInfo";
    ```
1. 생성자를 추가하고 구성 요소의 대화 집합을 초기화합니다. 이번에는 `InitialDialogId` 속성을 명시적으로 설정합니다. 즉, 기본 폭포 대화가 집합에 추가되는 첫 번째 항목이 될 필요가 없습니다. 예를 들어 프롬프트를 먼저 추가하려면 런타임 문제를 일으키지 않고 해당 프롬프트를 추가할 수 있습니다.
    ```csharp
    public InstallAppDialog(string id)
        : base(id)
    {
        // Initialize our dialogs and prompts.
        InitialDialogId = MainId;
        AddDialog(new WaterfallDialog(MainId, new WaterfallStep[] {
            GetSearchTermAsync,
            ResolveAppNameAsync,
            GetMachineNameAsync,
            SubmitRequestAsync,
        }));
        AddDialog(new TextPrompt(TextId));
        AddDialog(new ChoicePrompt(ChoiceId));
    }
    ```
1. **StartAsync**를 폭포의 첫 번째 단계로 바꿀 수 있습니다.
    - 상태를 직접 관리해야 하므로 대화 상태에서 설치 앱 개체를 추적할 수 있습니다.
    - 사용자에게 입력을 요청하는 메시지는 프롬프트에 대한 호출에서 옵션이 됩니다.
    ```csharp
    private async Task<DialogTurnResult> GetSearchTermAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Create an object in dialog state in which to track our collected information.
        stepContext.Values[InstallInfo] = new InstallApp();

        // Ask for the search term.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("Ok let's get started. What is the name of the application? "),
            },
            cancellationToken);
    }
    ```
1. **appNameAsync** 및 **multipleAppsAsync**를 폭포의 두 번째 단계로 바꿀 수 있습니다.
    - 이제 사용자의 마지막 메시지를 살펴보는 대신 프롬프트 결과를 얻을 수 있습니다.
    - 데이터베이스 쿼리 및 if 문이 **appNameAsync**에서와 동일하게 구성됩니다. v4 대화에서 작동하도록 if 문에 있는 각 블록의 코드가 업데이트되었습니다.
        - 하나의 적중 항목이 있으면 대화 상태를 업데이트하고 다음 단계로 계속 진행합니다.
        - 여러 개의 적중 항목이 있으면 선택 프롬프트를 사용하여 옵션 목록에서 선택하도록 사용자에게 요청합니다. 즉 **multipleAppsAsync**만 삭제할 수 있습니다.
        - 적중 항목이 없으면 이 대화를 종료하고 루트 대화에 null을 반환합니다.
    ```csharp
    private async Task<DialogTurnResult> ResolveAppNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the text prompt.
        var appname = stepContext.Result as string;

        // Query the database for matches.
        var names = await this.getAppsAsync(appname);

        if (names.Count == 1)
        {
            // Get our tracking information from dialog state and add the app name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.AppName = names.First();

            return await stepContext.NextAsync();
        }
        else if (names.Count > 1)
        {
            // Ask the user to choose from the list of matches.
            return await stepContext.PromptAsync(
                ChoiceId,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("I found the following applications. Please choose one:"),
                    Choices = ChoiceFactory.ToChoices(names),
                },
                cancellationToken);
        }
        else
        {
            // If no matches, exit this dialog.
            await stepContext.Context.SendActivityAsync(
                $"Sorry, I did not find any application with the name '{appname}'.",
                cancellationToken: cancellationToken);

            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }
    ```
1. 쿼리가 해결되면 **appNameAsync**도 사용자에게 머신 이름을 요청했습니다. 폭포의 다음 단계에서 논리의 해당 부분을 캡처합니다.
    - 마찬가지로 v4에서 상태를 직접 관리해야 합니다. 여기서 유일하게 까다로운 점은 이전 단계에서 서로 다른 두 개의 논리 분기를 통해 이 단계에 도달할 수 있다는 것입니다.
    - 이번에는 다른 옵션을 제공하기만 하면 이전과 동일한 텍스트 프롬프트를 사용하여 사용자에게 머신 이름을 요청합니다.
    ```csharp
    private async Task<DialogTurnResult> GetMachineNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the tracking info. If we don't already have an app name,
        // Then we used the choice prompt to get it in the previous step.
        var install = stepContext.Values[InstallInfo] as InstallApp;
        if (install.AppName is null)
        {
            install.AppName = (stepContext.Result as FoundChoice).Value;
        }

        // We now need the machine name, so prompt for it.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(
                    $"Found {install.AppName}. What is the name of the machine to install application?"),
            },
            cancellationToken);
    }
    ```
1. **machineNameAsync**의 논리가 폭포의 마지막 단계에서 래핑됩니다.
    - 텍스트 프롬프트 결과에서 머신 이름을 검색하고 대화 상태를 업데이트합니다.
    - 지원 코드가 다른 프로젝트에 있으므로 데이터베이스에 대한 업데이트 호출을 제거합니다.
    - 그런 다음, 사용자에게 성공 메시지를 보내고 대화를 종료합니다.
    ```csharp
    private async Task<DialogTurnResult> SubmitRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            // Get the tracking info and add the machine name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.MachineName = stepContext.Context.Activity.Text;

            //TODO: Save to this information to the database.
        }

        await stepContext.Context.SendActivityAsync(
            $"Great, your request to install {install.AppName} on {install.MachineName} has been scheduled.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. 데이터베이스를 시뮬레이션하기 위해 데이터베이스 대신 정적 목록을 쿼리하도록 **getAppsAsync**를 업데이트했습니다.
    ```csharp
    private async Task<List<string>> getAppsAsync(string Name)
    {
        List<string> names = new List<string>();

        // Simulate querying the database for applications that match.
        return (from app in AppMsis
            where app.ToLower().Contains(Name.ToLower())
            select app).ToList();
    }

    // Example list of app names in the database.
    private static readonly List<string> AppMsis = new List<string>
    {
        "µTorrent 3.5.0.44178",
        "7-Zip 17.1",
        "Ad-Aware 9.0",
        "Adobe AIR 2.5.1.17730",
        "Adobe Flash Player (IE) 28.0.0.105",
        "Adobe Flash Player (Non-IE) 27.0.0.130",
        "Adobe Reader 11.0.14",
        "Adobe Shockwave Player 12.3.1.201",
        "Advanced SystemCare Personal 11.0.3",
        "Auslogics Disk Defrag 3.6",
        "avast! 4 Home Edition 4.8.1351",
        "AVG Anti-Virus Free Edition 9.0.0.698",
        "Bonjour 3.1.0.1",
        "CCleaner 5.24.5839",
        "Chmod Calculator 20132.4",
        "CyberLink PowerDVD 17.0.2101.62",
        "DAEMON Tools Lite 4.46.1.328",
        "FileZilla Client 3.5",
        "Firefox 57.0",
        "Foxit Reader 4.1.1.805",
        "Google Chrome 66.143.49260",
        "Google Earth 7.3.0.3832",
        "Google Toolbar (IE) 7.5.8231.2252",
        "GSpot 2701.0",
        "Internet Explorer 903235.0",
        "iTunes 12.7.0.166",
        "Java Runtime Environment 6 Update 17",
        "K-Lite Codec Pack 12.1",
        "Malwarebytes Anti-Malware 2.2.1.1043",
        "Media Player Classic 6.4.9.0",
        "Microsoft Silverlight 5.1.50907",
        "Mozilla Thunderbird 57.0",
        "Nero Burning ROM 19.1.1005",
        "OpenOffice.org 3.1.1 Build 9420",
        "Opera 12.18.1873",
        "Paint.NET 4.0.19",
        "Picasa 3.9.141.259",
        "QuickTime 7.79.80.95",
        "RealPlayer SP 12.0.0.319",
        "Revo Uninstaller 1.95",
        "Skype 7.40.151",
        "Spybot - Search & Destroy 1.6.2.46",
        "SpywareBlaster 4.6",
        "TuneUp Utilities 2009 14.0.1000.353",
        "Unlocker 1.9.2",
        "VLC media player 1.1.6",
        "Winamp 5.56 Build 2512",
        "Windows Live Messenger 2009 16.4.3528.331",
        "WinPatrol 2010 31.0.2014",
        "WinRAR 5.0",
    };
    ```

### <a name="update-the-local-admin-dialog"></a>로컬 관리자 대화 업데이트

v3에서 이 대화는 사용자에게 인사하고, Formflow 대화를 시작한 다음, 결과를 데이터베이스에 저장합니다. 여기서는 2단계 폭포로 쉽게 변환됩니다.

1. `using` 문을 업데이트합니다. 이 대화에는 v3 Formflow 대화가 포함되어 있습니다. v4에서는 Formflow 커뮤니티 라이브러리를 사용할 수 있습니다.
    ```csharp
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. 대화 상태에서 결과를 사용할 수 있으므로 `LocalAdmin`에 대한 인스턴스 속성을 제거할 수 있습니다.
1. 대화에 사용할 ID 상수를 정의합니다. 커뮤니티 라이브러리에서 생성된 대화 ID는 항상 대화에서 생성하는 클래스의 이름으로 설정됩니다.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string AdminDialog { get; } = nameof(LocalAdminPrompt);
    ```
1. 생성자를 추가하고 구성 요소의 대화 집합을 초기화합니다. Formflow 대화도 동일한 방식으로 만들어집니다. 생성자에 있는 구성 요소의 대화 집합에 추가하기만 하면 됩니다.
    ```csharp
    public LocalAdminDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            SaveResultAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildLocalAdminForm, FormOptions.PromptInStart));
    }
    ```
1. **StartAsync**를 폭포의 첫 번째 단계로 바꿀 수 있습니다. Formflow는 생성자에 이미 만들었고, 나머지 두 명령문은 이를 해석합니다.
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Great I will help you request local machine admin.");

        // Begin the Formflow dialog.
        return await stepContext.BeginDialogAsync(AdminDialog, cancellationToken: cancellationToken);
    }
    ```
1. **ResumeAfterLocalAdminFormDialog**를 폭포의 두 번째 단계로 바꿀 수 있습니다. 인스턴스 속성 대신 단계 컨텍스트에서 반환 값을 가져와야 합니다.
    ```csharp
    private async Task<DialogTurnResult> SaveResultAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var admin = stepContext.Result as LocalAdminPrompt;

            //TODO: Save to this information to the database.
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. **BuildLocalAdminForm**은 Formflow에서 인스턴스 속성을 업데이트하지 않는다는 것을 제외하고는 거의 동일하게 유지됩니다.
    ```csharp
    // Nearly the same as before.
    private IForm<LocalAdminPrompt> BuildLocalAdminForm()
    {
        //here's an example of how validation can be used in form builder
        return new FormBuilder<LocalAdminPrompt>()
            .Field(nameof(LocalAdminPrompt.MachineName),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.MachineName = (string)value;
                return result;
            })
            .Field(nameof(LocalAdminPrompt.AdminDuration),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.AdminDuration = Convert.ToInt32((long)value) as int?;
                return result;
            })
            .Build();
    }
    ```

### <a name="update-the-reset-password-dialog"></a>재설정 암호 대화 업데이트

v3에서 이 대화는 사용자에게 인사하고, 전달 코드를 사용하여 사용자에게 권한을 부여하고, 실패하거나 Formflow 대화를 시작한 다음, 암호를 다시 설정합니다. 이 경우에도 여전히 폭포로 잘 변환됩니다.

1. `using` 문을 업데이트합니다. 이 대화에는 v3 Formflow 대화가 포함되어 있습니다. v4에서는 Formflow 커뮤니티 라이브러리를 사용할 수 있습니다.
    ```csharp
    using System;
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. 대화에 사용할 ID 상수를 정의합니다. 커뮤니티 라이브러리에서 생성된 대화 ID는 항상 대화에서 생성하는 클래스의 이름으로 설정됩니다.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string ResetDialog { get; } = nameof(ResetPasswordPrompt);
    ```
1. 생성자를 추가하고 구성 요소의 대화 집합을 초기화합니다. Formflow 대화도 동일한 방식으로 만들어집니다. 생성자에 있는 구성 요소의 대화 집합에 추가하기만 하면 됩니다.
    ```csharp
    public ResetPasswordDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            ProcessRequestAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildResetPasswordForm, FormOptions.PromptInStart));
    }
    ```
1. **StartAsync**를 폭포의 첫 번째 단계로 바꿀 수 있습니다. Formflow는 이미 생성자에 만들었습니다. 그렇지 않으면 동일한 논리를 유지하면서 v3 호출을 이와 동등한 v4 호출로 변환할 수 있습니다.
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Alright I will help you create a temp password.");

        // Check the passcode and fail out or begin the Formflow dialog.
        if (sendPassCode(stepContext))
        {
            return await stepContext.BeginDialogAsync(ResetDialog, cancellationToken: cancellationToken);
        }
        else
        {
            //here we can simply fail the current dialog because we have root dialog handling all exceptions
            throw new Exception("Failed to send SMS. Make sure email & phone number has been added to database.");
        }
    }
    ```
1. **sendPassCode**는 주로 연습으로 남아 있습니다. 원래 코드는 주석 처리되고, 메서드는 true를 반환합니다. 또한 원래 봇에서 사용되지 않았으므로 이메일 주소를 다시 제거할 수 있습니다.
    ```csharp
    private bool sendPassCode(DialogContext context)
    {
        //bool result = false;

        //Recipient Id varies depending on channel
        //refer ChannelAccount class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dd/def/class_microsoft_1_1_bot_1_1_connector_1_1_channel_account.html#a0b89cf01fdd73cbc00a524dce9e2ad1a
        //as well as Activity class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
        //int passcode = new Random().Next(1000, 9999);
        //Int64? smsNumber = 0;
        //string smsMessage = "Your Contoso Pass Code is ";
        //string countryDialPrefix = "+1";

        // TODO: save PassCode to database
        //using (var db = new ContosoHelpdeskContext())
        //{
        //    var reset = db.ResetPasswords.Where(r => r.EmailAddress == email).ToList();
        //    if (reset.Count >= 1)
        //    {
        //        reset.First().PassCode = passcode;
        //        smsNumber = reset.First().MobileNumber;
        //        result = true;
        //    }

        //    db.SaveChanges();
        //}

        // TODO: send passcode to user via SMS.
        //if (result)
        //{
        //    result = Helper.SendSms($"{countryDialPrefix}{smsNumber.ToString()}", $"{smsMessage} {passcode}");
        //}

        //return result;
        return true;
    }
    ```
1. **BuildResetPasswordForm**는 변경되지 않습니다.
1. **ResumeAfterLocalAdminFormDialog**를 폭포의 두 번째 단계로 바꿀 수 있습니다. 그러면 단계 컨텍스트에서 반환 값을 가져올 수 있습니다. 원래 대화에서 아무 작업도 수행하지 않은 이메일 주소를 제거하고, 데이터베이스를 쿼리하는 대신 더미 결과를 제공했습니다. 동일한 논리를 유지하면서 v3 호출을 이와 동등한 v4 호출로 변환할 수 있습니다.
    ```csharp
    private async Task<DialogTurnResult> ProcessRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var prompt = stepContext.Result as ResetPasswordPrompt;
            int? passcode;

            // TODO: Retrieve the passcode from the database.
            passcode = 1111;

            if (prompt.PassCode == passcode)
            {
                string temppwd = "TempPwd" + new Random().Next(0, 5000);
                await stepContext.Context.SendActivityAsync(
                    $"Your temp password is {temppwd}",
                    cancellationToken: cancellationToken);
            }
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```

### <a name="update-models-as-necessary"></a>필요에 따라 모델 업데이트

Formflow 라이브러리를 참조하는 일부 모델에서 `using` 문을 업데이트해야 합니다.

1. `LocalAdminPrompt`에서 다음으로 변경합니다.
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    ```
1. `ResetPasswordPrompt`에서 다음으로 변경합니다.
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    using System;
    ```

## <a name="update-webconfig"></a>Web.config 업데이트

**MicrosoftAppId** 및 **MicrosoftAppPassword**에 대한 구성 키를 주석 처리합니다. 이렇게 하면 에뮬레이터에 이러한 값을 제공하지 않고도 봇을 로컬로 디버그할 수 있습니다.

## <a name="run-and-test-your-bot-in-the-emulator"></a>에뮬레이터에서 봇 실행 및 테스트

이 시점에서 봇을 IIS에서 로컬로 실행하고 에뮬레이터를 사용하여 연결할 수 있어야 합니다.

1. IIS에서 봇을 실행합니다.
1. 에뮬레이터를 시작하고, 봇의 엔드포인트(예: **http://localhost:3978/api/messages**)에 연결합니다.
    - 처음으로 봇을 실행하는 경우라면 **파일 > 새 봇**을 클릭하고 화면에 표시된 지시를 따릅니다. 그렇지 않은 경우 **파일 > 봇 열기**를 클릭하여 기존 봇을 엽니다.
    - 구성에서 포트 설정을 다시 확인합니다. 예를 들어 봇이 브라우저에서 `http://localhost:3979/`로 열렸으면 에뮬레이터에서 봇의 엔드포인트를 `http://localhost:3979/api/messages`로 설정합니다.
1. 네 개의 대화가 모두 작동해야 하고, 폭포 단계에서 중단점을 설정하여 대화 컨텍스트 및 대화 상태가 이러한 지점에 있는지 확인할 수 있습니다.

## <a name="additional-resources"></a>추가 리소스

v4 개념 항목:

- [봇 작동 방식](../bot-builder-basics.md)
- [상태 관리](../bot-builder-concept-state.md)
- [다이얼로그 라이브러리](../bot-builder-concept-dialog.md)

v4 방법 항목:

- [문자 메시지 보내기 및 받기](../bot-builder-howto-send-messages.md)
- [사용자 및 대화 데이터 저장](../bot-builder-howto-v4-state.md)
- [순차적 대화 흐름 구현](../bot-builder-dialog-manage-conversation-flow.md)
