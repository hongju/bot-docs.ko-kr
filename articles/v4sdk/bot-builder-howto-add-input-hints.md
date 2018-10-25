---
title: 메시지에 입력 힌트 추가 | Microsoft Docs
description: Bot Builder SDK를 사용하여 메시지에 입력 힌트를 추가하는 방법에 대해 알아봅니다.
keywords: 입력 힌트, 입력 허용, 입력 필요, 입력 무시, 음성
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 08/24/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d55accd5ad9ad7db12d0b0e6865e04dcf7718110
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996741"
---
# <a name="add-input-hints-to-messages"></a>메시지에 입력 힌트 추가

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

메시지에 대한 입력 힌트를 지정하여 메시지를 클라이언트에 전달한 후 봇이 사용자 입력을 허용, 필요 또는 무시하는지 여부를 나타낼 수 있습니다. 많은 채널에서 이 기능을 사용하여 클라이언트가 사용자 입력 컨트롤 상태를 적절히 설정할 수 있습니다. 예를 들어 메시지의 입력 힌트가 봇이 사용자 입력을 무시하는 것을 표시하는 경우 클라이언트는 사용자가 입력을 제공하지 못하게 막으려면 마이크를 종료하고 입력란을 사용하지 않도록 설정할 수 있습니다.

필요한 라이브러리가 입력 힌트에 포함되게 해야 합니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
using Microsoft.Bot.Schema;
```

<!--TODO: Remove the following remark after the next release of the NuGet packages.-->

이러한 예제에 사용된 **MessageFactory** 클래스는 다음 네임스페이스에서 정의됩니다.

```cs
using Microsoft.Bot.Builder.Core.Extensions;
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const {InputHints, MessageFactory} = require('botbuilder');
```

---

## <a name="accepting-input"></a>입력 허용

봇이 입력에 대해 수동적으로 준비하지만 사용자의 응답을 기다리지 않고 있음을 나타내려면 메시지의 입력 힌트를 _입력 허용_으로 설정합니다. 많은 채널에서 이렇게 하면 클라이언트의 입력 상자를 사용하고 마이크를 종료하게 되지만 여전히 사용자에게 액세스할 수 있습니다. 예를 들어 사용자가 마이크 단추를 누르고 있는 경우 Cortana는 마이크를 열고 사용자로부터 입력을 허용합니다. 다음 코드는 봇이 사용자 입력을 허용하고 있음을 나타내는 메시지를 만듭니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.AcceptingInput);
await context.SendActivityAsync(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.AcceptingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="expecting-input"></a>입력 필요

봇이 사용자의 응답을 기다리고 있음을 나타내려면 메시지의 입력 힌트를 _입력 필요_로 설정합니다. 많은 채널에서 이렇게 하면 클라이언트의 입력 상자를 사용하고 마이크를 시작하게 됩니다. 다음 예제 코드는 봇이 사용자 입력을 기다리고 있음을 나타내는 메시지를 만듭니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.ExpectingInput);
await context.SendActivityAsync(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.ExpectingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="ignoring-input"></a>입력 무시

봇이 사용자의 입력을 받을 준비가 되어 있지 않음을 나타내려면 메시지의 입력 힌트를 _입력 무시_로 설정합니다. 많은 채널에서 이렇게 하면 클라이언트의 입력 상자를 사용하지 않고 마이크를 종료하게 됩니다. 다음 예제 코드는 봇이 사용자 입력을 무시하고 있는 것을 나타내는 메시지를 만듭니다.

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.IgnoringInput);
await context.SendActivityAsync(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.IgnoringInput);
await context.sendActivity(basicMessage);
```

---

## <a name="default-values-for-input-hint"></a>입력 힌트에 대한 기본값

메시지에 대한 입력 힌트를 설정하지 않은 경우 Bot Builder SDK가 이 논리를 사용하여 자동으로 입력 힌트를 설정해줍니다.

- 봇이 프롬프트를 보내는 경우 메시지에 대한 입력 힌트는 봇이 **입력이 필요**함을 지정합니다.</li>
- 봇이 단일 메시지를 보내는 경우 메시지에 대한 입력 힌트는 봇이 **입력을 허용**하고 있음을 지정합니다.</li>
- 봇이 일련의 연속 메시지를 보내는 경우 그 중 최종 메시지를 제외한 모든 메시지에 대한 입력 힌트는 봇이 **입력을 무시**하고 있음을 지정하고 최종 메시지에 대한 입력 힌트는 **입력을 허용**하고 있음을 지정합니다.

