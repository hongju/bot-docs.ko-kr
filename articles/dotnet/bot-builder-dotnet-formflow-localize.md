---
title: 폼 콘텐츠 지역화 | Microsoft Docs
description: FormFlow 및 .NET용 Bot Builder SDK를 사용하여 폼 콘텐츠를 지역화하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: bb0ac4b8e3fa34ec8863bb323ae968db37972a6f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301875"
---
# <a name="localize-form-content"></a>폼 콘텐츠 지역화

폼의 지역화 언어는 현재 스레드의 [CurrentUICulture](https://msdn.microsoft.com/en-us/library/system.threading.thread.currentuiculture(v=vs.110).aspx) 및 [CurrentCulture](https://msdn.microsoft.com/en-us/library/system.threading.thread.currentculture(v=vs.110).aspx)에 따라 결정됩니다. 기본적으로 문화권은 현재 메시지의 **로캘** 필드에서 파생되지만, 해당 기본 동작을 재정의할 수 있습니다. 봇이 생성되는 방식에 따라, 지역화된 정보를 다음과 같은 최대 3개의 원본에서 가져올 수 있습니다.

- **PromptDialog** 및 **FormFlow**에 대한 기본 제공 지역화
- 폼에서 정적 문자열을 생성하는 리소스 파일
- 동적 계산 필드, 메시지 또는 확인을 위해 문자열을 사용하여 만든 리소스 파일

## <a name="generate-a-resource-file-for-the-static-strings-in-your-form"></a>폼에서 정적 문자열에 대한 리소스 파일 생성

폼의 정적 문자열에는 폼이 C# 클래스의 정보에서 생성하는 문자열과 사용자가 프롬프트, 템플릿, 메시지 또는 확인으로 지정하는 문자열이 포함됩니다. 기본 제공 템플릿에서 생성되는 문자열은 이미 지역화되어 있으므로 정적 문자열로 간주되지 않습니다. 폼의 다양한 문자열은 자동으로 생성되므로 일반 C# 리소스 문자열을 직접 사용하는 것은 가능하지 않습니다. 대신, `IFormBuilder.SaveResources`를 호출하거나 .NET용 BotBuilder SDK에 포함된 **RView** 도구를 사용하여 폼의 정적 문자열에 대한 리소스 파일을 생성할 수 있습니다.

### <a name="use-iformbuildersaveresources"></a>IFormBuilder.SaveResources 사용

폼에서 [IFormBuilder.SaveResources][saveResources]를 호출하여 리소스 파일을 생성하고 문자열을 .resx 파일에 저장할 수 있습니다.

### <a name="use-rview"></a>RView 사용

또는 NET용 BotBuilder SDK에 포함된 <a href="https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Tools/RView" target="_blank">RView</a> 도구를 사용하여 .dll 또는 .exe 기준의 리소스 파일을 생성할 수 있습니다. .resx 파일을 생성하려면 **rview**를 실행하고 정적 폼 빌드 메서드 및 해당 메서드의 경로를 포함하는 어셈블리를 지정합니다. 이 코드 조각은 **RView**를 사용하여 `Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.resx` 리소스 파일을 생성하는 방법을 보여 줍니다. 

```csharp
rview -g Microsoft.Bot.Sample.AnnotatedSandwichBot.dll Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.BuildForm
```

이 코드 조각은 이 **rview** 명령을 실행하여 생성되는 .resx 파일를 보여 줍니다.

```xml
<data name="Specials_description;VALUE" xml:space="preserve">
<value>Specials</value>
</data>
<data name="DeliveryAddress_description;VALUE" xml:space="preserve">
<value>Delivery Address</value>
</data>
<data name="DeliveryTime_description;VALUE" xml:space="preserve">
<value>Delivery Time</value>
</data>
<data name="PhoneNumber_description;VALUE" xml:space="preserve">
<value>Phone Number</value>
</data>
<data name="Rating_description;VALUE" xml:space="preserve">
<value>your experience today</value>
</data>
<data name="message0;LIST" xml:space="preserve">
<value>Welcome to the sandwich order bot!</value>
</data>
<data name="Sandwich_terms;LIST" xml:space="preserve">
<value>sandwichs?</value>
</data>
```

## <a name="configure-your-project"></a>프로젝트 구성

리소스 파일을 생성한 후 프로젝트에 추가하고 다음 단계를 완료하여 중립 언어를 설정합니다. 

1. 프로젝트를 마우스 오른쪽 단추로 클릭하고 **응용 프로그램**을 선택합니다.
2. **어셈블리 정보**를 클릭합니다.
3. 봇을 개발할 때 사용한 언어에 해당하는 **중립 언어** 값을 선택합니다.

폼이 만들어지면 [IFormBuilder.Build][build] 메서드는 폼 형식 이름을 포함하는 리소스를 자동으로 찾은 후, 해당 리소스를 사용하여 폼의 정적 문자열을 지역화합니다. 

> [!NOTE]
> [Advanced.Field.SetDefine][setDefine]을 사용하여 정의된 동적 계산 필드([동적 필드 사용](bot-builder-dotnet-formflow-formbuilder.md#dynamically-define-field-values-confirmations-and-messages)에 설명됨)는 해당 문자열이 폼이 채워질 때 생성되므로 정적 필드와 같은 방식으로 지역화할 수 없습니다. 그러나 일반 C# 지역화 메커니즘을 사용하여 동적 계산 필드를 지역화할 수 있습니다.

### <a name="localize-resource-files"></a>리소스 파일 지역화 

프로젝트에 리소스 파일을 추가한 후 <a href="https://developer.microsoft.com/en-us/windows/develop/multilingual-app-toolkit" target="_blank">MAT(다국어 앱 도구 키트)</a>를 사용하여 지역화할 수 있습니다. **MAT**를 설치한 후 다음 단계를 완료하여 프로젝트에서 사용할 수 있도록 설정합니다.

1. Visual Studio 솔루션 탐색기에서 프로젝트를 선택합니다.
2. **도구**, **다국어 앱 도구 키트** 및 **사용**을 차례로 클릭합니다.
3. 프로젝트를 마우스 오른쪽 단추로 클릭하고고 **다국어 앱 도구 키트**, **번역 추가**를 선택하여 번역을 선택합니다. 이렇게 하면 자동 또는 수동으로 번역할 수 있는 업계 표준 <a href="https://en.wikipedia.org/wiki/XLIFF" target="_blank">XLF</a> 파일이 생성됩니다.

> [!NOTE]
> 이 문서에서는 다국어 앱 도구 키트를 사용하여 콘텐츠를 지역화하는 방법을 설명하지만 다른 다양한 지역화 방법을 구현할 수도 있습니다.

## <a name="see-it-in-action"></a>실제 동작 확인

이 코드에서는 위에 설명된 대로 [FormBuilder를 사용하여 폼 사용자 지정](bot-builder-dotnet-formflow-formbuilder.md)에 따라 지역화를 구현합니다. 이 예제에서 `DynamicSandwich` 클래스(여기에는 나오지 않음)에는 동적 로 계산 필드, 메시지 및 확인에 대한 지역화 정보가 포함되어 있습니다.

[!code-csharp[Build localized form](../includes/code/dotnet-formflow-localize.cs#buildLocalizedForm)]

이 코드 조각은 `CurrentUICulture`가 **French**일 때 봇과 사용자 간에 생성되는 상호 작용을 보여 줍니다.

```console
Bienvenue sur le bot d'ordre "sandwich" !
Quel genre de "sandwich" vous souhaitez sur votre "sandwich"?
 1. BLT
 2. Jambon Forêt Noire
 3. Poulet Buffalo
 4. Faire fondre le poulet et Bacon Ranch
 5. Combo de coupe à froid
 6. Boulette de viande Marinara
 7. Poulet rôti au four
 8. Rôti de boeuf
 9. Rotisserie poulet
 10. Italienne piquante
 11. Bifteck et fromage
 12. Oignon doux Teriyaki
 13. Thon
 14. Poitrine de dinde
 15. Veggie
> 2

Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> ?
* Vous renseignez le champ longueur.Réponses possibles:
* Vous pouvez saisir un numéro 1-2 ou des mots de la description. (Six pouces, ou Pied Long)
* Retourner à la question précédente.
* Assistance: Montrez les réponses possibles.
* Abandonner: Abandonner sans finir
* Recommencer remplir le formulaire. (Vos réponses précédentes sont enregistrées.)
* Statut: Montrer le progrès en remplissant le formulaire jusqu'à présent.
* Vous pouvez passer à un autre champ en entrant son nom. ("Sandwich", Longueur, Pain, Fromage, Nappages, Sauces, Adresse de remise, Délai de livraison, ou votre expérience aujourd'hui).
Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> 1

Quel genre de pain vous souhaitez sur votre "sandwich"?
 1. Neuf grains de blé
 2. Neuf grains miel avoine
 3. Italien
 4. Fromage et herbes italiennes
 5. Pain plat
> neuf
Par pain "neuf" vouliez-vous dire (1. Neuf grains miel avoine, ou 2. Neuf grains de blé)
```

## <a name="sample-code"></a>샘플 코드

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>추가 리소스

- [FormFlow의 기본 기능](bot-builder-dotnet-formflow.md)
- [FormFlow의 고급 기능](bot-builder-dotnet-formflow-advanced.md)
- [FormBuilder를 사용하여 폼 사용자 지정](bot-builder-dotnet-formflow-formbuilder.md)
- [JSON 스키마를 사용하여 폼 정의](bot-builder-dotnet-formflow-json-schema.md)
- [패턴 언어를 사용하여 사용자 환경 사용자 지정](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Builder SDK 참조</a>

[build]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1.build 

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[saveResources]: /dotnet/api/microsoft.bot.builder.formflow.iform-1.saveresources
