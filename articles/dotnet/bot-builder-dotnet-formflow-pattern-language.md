---
title: 패턴 언어를 사용하여 사용자 환경 사용자 지정 | Microsoft Docs
description: .NET용 Bot Builder SDK로 패턴 언어를 사용하여 FormFlow 프롬프트를 사용자 지정하고 FormFlow 템플릿을 재정의하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a192b69b2ffbac428d80b2fe7c3fd9180caacd4f
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905585"
---
# <a name="customize-user-experience-with-pattern-language"></a>패턴 언어를 사용하여 사용자 환경 사용자 지정

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

프롬프트를 사용자 지정 하거나 기본 템플릿을 재정의하는 경우 패턴 언어를 사용하여 프롬프트의 콘텐츠 및/또는 형식을 지정할 수 있습니다. 

## <a name="prompts-and-templates"></a>프롬프트 및 템플릿

[프롬프트][promptAttribute]는 정보나 확인을 요청하기 위해 사용자에게 전송되는 메시지를 나타냅니다. [Prompt 특성](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-prompt-attribute)을 사용하거나 [IFormBuilder<T>.Field][field]를 통해 암시적으로 프롬프트를 사용자 지정할 수 있습니다. 

양식은 템플릿을 사용하여 프롬프트 및 기타 항목(예: 도움말)을 자동으로 구성합니다. [Template attribute](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-template-attribute)를 사용하여 클래스 또는 필드의 기본 템플릿을 재정의 할 수 있습니다. 

> [!TIP]
> [FormConfiguration.Templates][formConfiguration] 클래스는 패턴 언어를 사용하는 방법에 대한 좋은 예를 제공하는 내장형 템플릿 집합을 정의합니다.

## <a name="elements-of-pattern-language"></a>패턴 언어 요소

패턴 언어는 중괄호(`{}`)를 사용하여 런타임 시 실제 값으로 대체될 요소를 식별합니다. 이 표에는 패턴 언어 요소가 나열되어 있습니다.

| 요소 | 설명 |
|----|----|
| `{<format>}` | 현재 필드(특성이 적용되는 필드)의 값을 보여줍니다. |
| `{&}` | 현재 필드의 설명을 표시합니다(달리 지정하지 않는 한 필드의 이름입니다). |
| `{<field><format>}` | 명명된 필드의 값을 표시합니다. | 
| `{&<field>}` | 명명된 필드의 설명을 표시합니다. |
| <code>{&#124;&#124;}</code> | 현재 선택 항목(필드의 현재 값이 될 수 있음), "기본 설정 없음" 또는 열거형의 값을 표시합니다. |
| `{[{<field><format>} ...]}` | [Separator][separator] 및 [LastSeparator][lastSeparator]를 사용하여 명명된 필드에서 값의 목록을 표시하여 목록의 개별 값을 구분합니다. |
| `{*}` | 각 활성 필드에 대해 한 줄을 표시합니다. 각 줄에는 필드 설명과 현재 값이 포함됩니다. | 
| `{*filled}` | 실제 값을 포함하는 각 활성 필드에 대해 한 줄을 표시합니다. 각 줄에는 필드 설명과 현재 값이 포함됩니다. |
| `{<nth><format>}` | 템플릿의 n 번째 인수에 적용되는 일반 C# 형식 지정자입니다. 사용 가능한 인수 목록은 [TemplateUsage][templateUsage]를 참조하세요. |
| `{?<textOrPatternElement>...}` | 조건부 대체 문자입니다. 패턴 요소로 참조된 모든 요소에 값이 있으면 값이 대체되고 전체 식이 사용됩니다. |

위에 나열된 요소에 대해:

- `<field>` 자리 표시자는 양식 클래스에 있는 필드의 이름입니다. 예를 들어, 클래스에 `Size`라는 이름의 필드가 포함되어있는 경우 `{Size}`를 패턴 요소로 지정할 수 있습니다. 

- 패턴 요소 내의 줄임표(`"..."`)는 요소가 여러 값을 포함할 수 있음을 나타냅니다.

- `<format>` 자리 표시자는 C# 형식 지정자입니다. 예를 들어 클래스에 이름이 `Rating`이고 형식이 `double`인 필드가 들어있는 경우 패턴 요소로 `{Rating:F2}`를 지정하여 두 자리의 전체 자릿수를 표시할 수 있습니다.

- `<nth>` 자리 표시자는 템플릿의 n 번째 인수를 참조합니다.

### <a name="pattern-language-within-a-prompt-attribute"></a>프롬프트 특성 내의 패턴 언어

이 예에서는 `{&}` 요소를 사용하여 `Sandwich` 필드에 대한 설명을 표시하고 `{||}` 요소를 사용하여 이 필드에 대한 선택 목록을 표시합니다.

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns1)]

결과 프롬프트는 다음과 같습니다.

```console
What kind of sandwich would you like?
1. BLT
2. Black Forest Ham
3. Buffalo Chicken
4. Chicken And Bacon Ranch Melt
5. Cold Cut Combo
6. Meatball Marinara
7. Oven Roasted Chicken
8. Roast Beef
9. Rotisserie Style Chicken
10. Spicy Italian
11. Steak And Cheese
12. Sweet Onion Teriyaki
13. Tuna
14. Turkey Breast
15. Veggie
>
```

## <a name="formatting-parameters"></a>형식 지정 매개 변수

프롬프트 및 템플릿은 다음과 같은 형식 지정 매개 변수를 지원합니다.

| 사용 현황 | 설명 |
|----|----|
| `AllowDefault` | <code>{&#124;&#124;}</code> 패턴 요소에 적용됩니다. 양식에 현재 필드 값을 선택 가능한 값으로 표시할지 여부를 결정합니다. `true`이면 현재 값이 가능한 값으로 표시됩니다. 기본값은 `true`입니다. |
| `ChoiceCase` | <code>{&#124;&#124;}</code> 패턴 요소에 적용됩니다. 각 선택 항목의 텍스트를 정규화할지 여부를 결정합니다(예: 각 단어의 첫 글자를 대문자로 표시할지 여부). 기본값은 `CaseNormalization.None`입니다. 가능한 값은 [CaseNormalization][caseNormalization]을 참조하세요. |
| `ChoiceFormat` | <code>{&#124;&#124;}</code> 패턴 요소에 적용됩니다. 선택 목록을 번호 매기기 목록 또는 글머리 기호 목록으로 표시할지 결정합니다. 번호 매기기 목록은 `ChoiceFormat`을 `{0}`(기본값)으로 설정합니다. 글머리 기호 목록은 `ChoiceFormat`을 `{1}`로 설정합니다. |
| `ChoiceLastSeparator` | <code>{&#124;&#124;}</code> 패턴 요소에 적용됩니다. 선택한 항목의 인라인 목록에서 마지막 선택 항목 앞에 구분 기호를 포함할지 결정합니다. |
| `ChoiceParens` | <code>{&#124;&#124;}</code> 패턴 요소에 적용됩니다. 선택 항목의 인라인 목록을 괄호 안에 표시할지 여부를 결정합니다. `true`이면 선택 목록이 괄호 안에 표시됩니다. 기본값은 `true`입니다. |
| `ChoiceSeparator` | <code>{&#124;&#124;}</code> 패턴 요소에 적용됩니다. 선택 항목의 인라인 목록에서 마지막 선택 항목을 제외한 모든 선택 항목 앞에 구분 기호를 포함할지 결정합니다. | 
| `ChoiceStyle` | <code>{&#124;&#124;}</code> 패턴 요소에 적용됩니다. 선택 목록을 인라인으로 표시할지 또는 한 줄에 표시할지 결정합니다. 기본은 `ChoiceStyleOptions.Auto`이며 이 값은 런타임 시 선택 항목을 인라인으로 표시할지 또는 목록으로 표시할지 결정합니다. 가능한 값은 [ChoiceStyleOptions][choiceStyleOptions]를 참조하세요. |
| `Feedback` | 프롬프트에만 적용됩니다. 양식이 사용자의 선택을 다시 표시하여 양식이 선택을 이해했음을 나타낼지 여부를 결정합니다. 기본값은 `FeedbackOptions.Auto`이며 이 값은 일부는 이해할 수 없는 경우에만 사용자 입력을 다시 표시합니다. 가능한 값은 [FeedbackOptions][feedbackOptions]를 참조하세요. |
| `FieldCase` | 필드 설명의 텍스트를 정규화할지 여부를 결정합니다(예: 각 단어의 첫 글자를 대문자로 표시할지 여부). 기본값은 `CaseNormalization.Lower`이며 이 값은 설명을 소문자로 변환합니다. 가능한 값은 [CaseNormalization][caseNormalization]을 참조하세요. |
| `LastSeparator` | `{[]}` 패턴 요소에 적용됩니다. 항목의 배열에서 마지막 항목 앞에 구분 기호를 포함할지 결정합니다. |
| `Separator` | `{[]}` 패턴 요소에 적용됩니다. 항목의 배열에서 마지막 항목을 제외한 배열의 모든 항목 앞에 구분 기호를 포함할지 결정합니다. |
| `ValueCase` | 필드 값의 텍스트를 정규화할지 여부를 결정합니다(예: 각 단어의 첫 글자를 대문자로 표시할지 여부). 기본값은 `CaseNormalization.InitialUpper`이며 이 값은 각 단어의 첫 글자를 대문자로 변환합니다. 가능한 값은 [CaseNormalization][caseNormalization]을 참조하세요. |

### <a name="prompt-attribute-with-formatting-parameter"></a>서식 지정 매개 변수가 있는 프롬프트 특성 

이 예제에서는 `ChoiceFormat` 매개 변수를 사용하여 선택 목록을 글머리 기호 목록으로 표시하도록 지정합니다.

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns2)]

결과 프롬프트는 다음과 같습니다.

```console
What kind of sandwich would you like?
* BLT
* Black Forest Ham
* Buffalo Chicken
* Chicken And Bacon Ranch Melt
* Cold Cut Combo
* Meatball Marinara
* Oven Roasted Chicken
* Roast Beef
* Rotisserie Style Chicken
* Spicy Italian
* Steak And Cheese
* Sweet Onion Teriyaki
* Tuna
* Turkey Breast
* Veggie
>
```

## <a name="sample-code"></a>샘플 코드

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>추가 리소스

- [FormFlow의 기본 기능](bot-builder-dotnet-formflow.md)
- [FormFlow의 고급 기능](bot-builder-dotnet-formflow-advanced.md)
- [FormBuilder를 사용하여 양식 사용자 지정](bot-builder-dotnet-formflow-formbuilder.md)
- [양식 콘텐츠 지역화](bot-builder-dotnet-formflow-localize.md)
- [JSON 스키마를 사용하여 양식 정의](bot-builder-dotnet-formflow-json-schema.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Builder SDK 참조</a>

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[field]: /dotnet/api/microsoft.bot.builder.formflow.iformbuilder-1.field

[formConfiguration]: /dotnet/api/microsoft.bot.builder.formflow.formconfiguration

[separator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.separator

[lastSeparator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.lastseparator

[templateUsage]: /dotnet/api/microsoft.bot.builder.formflow.templateusage

[caseNormalization]: /dotnet/api/microsoft.bot.builder.formflow.casenormalization

[choiceStyleOptions]: /dotnet/api/microsoft.bot.builder.formflow.choicestyleoptions

[feedbackOptions]: /dotnet/api/microsoft.bot.builder.formflow.feedbackoptions
