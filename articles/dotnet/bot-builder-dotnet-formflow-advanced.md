---
title: FormFlow의 고급 기능 | Microsoft Docs
description: FormFlow 및 .NET용 Bot Builder SDK를 사용하여 사용자 환경을 사용자 지정하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: fbdaa845c86150a572772c4aae8239880f42dd9e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997780"
---
# <a name="advanced-features-of-formflow"></a>FormFlow의 고급 기능

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[FormFlow의 기본 기능](bot-builder-dotnet-formflow.md)에서는 일반적인 사용자 환경을 제공하는 기본 FormFlow 구현에 대해 설명합니다. FormFlow를 사용하여 보다 사용자 지정된 사용자 환경을 제공하려면 초기 양식 상태를 지정하고, 비즈니스 논리를 추가하여 필드 간 상호 종속성을 관리하고 사용자 입력을 처리한 다음, 특성을 사용하여 프롬프트 사용자 지정, 템플릿 재정의, 선택적 필드 지정, 사용자 입력 일치 및 사용자 입력의 유효성 검사를 수행합니다. 

## <a name="specify-initial-form-state-and-entities"></a>초기 양식 상태 및 엔터티 지정

[FormDialog][formDialog]를 시작할 때, 필요에 따라 상태 인스턴스를 전달할 수 있습니다. 상태 인스턴스를 전달하면 FormFlow는 기본적으로 이미 값이 포함된 모든 필드에 대해 단계를 건너뜁니다. 해당 필드에 대해서는 사용자에게 확인 메시지가 표시되지 않습니다. 강제로 양식에서 모든 필드에 대해 사용자에게 확인 메시지를 표시하려면 `FormDialog`를 시작할 때 [FormOptions.PromptFieldsWithValues][promptFieldsWithValues]를 전달합니다. 필드에 초기 값이 포함된 경우 프롬프트는 해당 값을 기본값으로 사용합니다.

상태에 바인딩할 [LUIS](https://luis.ai/) 엔터티를 전달할 수도 있습니다. `EntityRecommendation.Type`이 C# 클래스의 필드 경로인 경우 `EntityRecommendation.Entity`는 인식기를 통해 전달되어 필드에 바인딩됩니다. FormFlow는 엔터티에 바인딩된 모든 필드에 대해 단계는 건너뜁니다. 해당 필드에 대해서는 사용자에게 확인 메시지가 표시되지 않습니다. 

## <a name="add-business-logic"></a>비즈니스 논리 추가 

양식 필드 간의 상호 종속성을 처리하거나, 필드 값을 가져오거나 설정하는 동안 특정 논리를 적용하려면 유효성 검사 함수 내에서 비즈니스 논리를 지정할 수 있습니다. 유효성 검사 함수를 사용하면 상태를 조작하고, 다음이 포함될 수 있는 [ValidateResult][validateResult] 개체를 반환할 수 있습니다. 

- 값이 잘못된 이유를 설명하는 피드백 문자열
- 변환된 값
- 값을 명확하게 설명하는 일련의 선택 항목

이 코드 예제에서는 `Toppings` 필드에 대한 유효성 검사 함수를 보여 줍니다. 필드에 대한 입력이 `ToppingOptions.Everything` 열거형 값을 포함하는 경우, 함수는 `Toppings` 필드 값에 전체 토핑 목록이 포함되어 있는지 확인합니다.

[!code-csharp[Validation function](../includes/code/dotnet-formflow-advanced.cs#validationFunction)]

유효성 검사 함수 외에도 [Term](#match-user-input-using-the-terms-attribute) 특성을 추가하여 “everything” 또는 “not”과 같은 사용자 식을 일치시킬 수 있습니다.

[!code-csharp[Terms for Toppings](../includes/code/dotnet-formflow-advanced.cs#toppingsTerms)]

이 코드 조각은 위에 표시된 유효성 검사 함수를 사용하여, 사용자가 “everything but Jalapenos”를 요청하는 경우 봇과 사용자 간의 상호 작용을 보여 줍니다. 

```console
Please select one or more toppings (current choice: No Preference)
 1. Everything
 2. Avocado
 3. Banana Peppers
 4. Cucumbers
 5. Green Bell Peppers
 6. Jalapenos
 7. Lettuce
 8. Olives
 9. Pickles
 10. Red Onion
 11. Spinach
 12. Tomatoes
> everything but jalapenos
For sandwich toppings you have selected Avocado, Banana Peppers, Cucumbers, Green Bell Peppers, Lettuce, Olives, Pickles, Red Onion, Spinach, and Tomatoes.
```

## <a name="formflow-attributes"></a>FormFlow 특성

이러한 C# 특성을 클래스에 추가하여 FormFlow 대화 상자의 동작을 사용자 지정할 수 있습니다.

| 특성 | 목적 |
|----|----| 
| [Describe][describeAttribute] | 템플릿이나 카드에 필드 또는 값을 표시하는 방법 변경 |
| [Numeric][numericAttribute] | 숫자 필드에 허용되는 값 제한 |
| [Optional][optionalAttribute] | 필드를 선택 사항으로 표시 |
| [Pattern][patternAttribute] | 문자열 필드의 유효성을 검사할 정규식 정의 |
| [Prompt][promptAttribute] | 필드에 대한 프롬프트 정의 |
| [Template][templateAttribute] | 프롬프트 또는 프롬프트의 값을 생성하는 데 사용할 템플릿 정의 |
| [Terms][termsAttribute] | 필드 또는 값과 일치되는 입력 용어 정의 |

## <a name="customize-prompts-using-the-prompt-attribute"></a>Prompt 특성을 사용하여 프롬프트 사용자 지정

양식의 각 필드에 대해 기본 프롬프트가 자동으로 생성되지만, `Prompt` 특성을 사용하여 필드에 대한 사용자 지정 프롬프트를 지정할 수 있습니다. 예를 들어, `SandwichOrder.Sandwich` 필드에 대한 기본 프롬프트가 “Please select a sandwich”이면 `Prompt` 특성을 추가하여 해당 필드에 대한 사용자 지정 프롬프트를 지정할 수 있습니다.

[!code-csharp[Prompt attribute](../includes/code/dotnet-formflow-advanced.cs#promptAttribute)]

이 예제에서는 [패턴 언어](bot-builder-dotnet-formflow-pattern-language.md)를 사용하여 런타임에 양식 데이터로 프롬프트를 동적으로 채웁니다. `{&}`는 필드에 대한 설명으로 바뀌고, `{||}`는 열거형의 선택 항목 목록으로 바뀝니다. 

> [!NOTE]
> 기본적으로 필드에 대한 설명은 필드 이름에서 생성됩니다. 필드에 대해 사용자 지정 설명을 지정하려면 `Describe` 특성을 추가합니다.

이 코드 조각은 위의 예제에서 지정된 사용자 지정 프롬프트를 보여 줍니다.

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

`Prompt` 특성은 양식에서 프롬프트 표시 방법에 영향을 주는 매개 변수를 지정할 수도 있습니다. 예를 들어, `ChoiceFormat` 매개 변수는 양식에서 선택 항목 목록을 렌더링하는 방법을 결정합니다.

[!code-csharp[Prompt attribute ChoiceFormat parameter](../includes/code/dotnet-formflow-advanced.cs#promptChoice)]

이 예제에서 `ChoiceFormat` 매개 변수의 값은 선택 항목을 번호 매기기 목록 대신 글머리 기호 목록으로 표시해야 함을 나타냅니다.

```console
What kind of sandwich would you like?
- BLT
- Black Forest Ham
- Buffalo Chicken
- Chicken And Bacon Ranch Melt
- Cold Cut Combo
- Meatball Marinara
- Oven Roasted Chicken
- Roast Beef
- Rotisserie Style Chicken
- Spicy Italian
- Steak And Cheese
- Sweet Onion Teriyaki
- Tuna
- Turkey Breast
- Veggie
>
```

## <a name="customize-prompts-using-the-template-attribute"></a>Template 특성을 사용하여 프롬프트 사용자 지정

`Prompt` 특성을 사용하면 단일 필드에 대한 프롬프트를 사용자 지정할 수 있는 반면, `Template` 특성을 사용하면 FormFlow에서 프롬프트를 자동으로 생성하는 데 사용하는 기본 템플릿을 바꿀 수 있습니다. 이 코드 예제에서는 `Template` 특성을 사용하여 양식에서 모든 열거형 필드를 처리하는 방법을 다시 정의합니다. 이 특성은 사용자가 하나의 항목만 선택할 수 있음을 나타내고, [패턴 언어](bot-builder-dotnet-formflow-pattern-language.md)를 사용하여 프롬프트 텍스트를 설정한 다음, 양식에서 줄당 하나의 항목만 표시하도록 지정합니다. 

[!code-csharp[Template attribute](../includes/code/dotnet-formflow-advanced.cs#templateAttribute)]

이 코드 조각은 `Bread` 필드 및 `Cheese` 필드에 대해 생성된 프롬프트를 보여 줍니다.

```console
What kind of bread would you like on your sandwich?
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> 

What kind of cheese would you like on your sandwich? 
 1. American
 2. Monterey Cheddar
 3. Pepperjack
> 
```

`Template` 특성을 사용하여 FormFlow에서 프롬프트를 생성하는 데 사용하는 기본 템플릿을 바꾸는 경우 양식에서 생성되는 프롬프트 및 메시지에 일부 변형을 삽입하는 것이 좋습니다. 이 작업을 위해 [패턴 언어](bot-builder-dotnet-formflow-pattern-language.md)를 사용하여 여러 텍스트 문자열을 정의할 수 있으며, 양식에서 프롬프트 또는 메시지를 표시해야 할 때마다 사용 가능한 옵션 중에서 임의로 선택됩니다.

이 코드 예제에서는 [TemplateUsage.NotUnderstood][notUnderstood] 템플릿을 다시 정의하여 메시지의 두 가지 변형을 지정합니다. 봇이 사용자 입력을 해석할 수 없음을 전달해야 하는 경우 두 텍스트 문자열 중 하나를 임의로 선택하여 메시지 내용을 결정합니다. 

[!code-csharp[Template variations of message](../includes/code/dotnet-formflow-advanced.cs#templateMessages)]

이 코드 조각은 봇과 사용자 간에 생성된 상호 작용의 예를 보여 줍니다. 

```console
What size of sandwich do you want? (1. Six Inch, 2. Foot Long)
> two feet
I do not understand "two feet".
> two feet
Try again, I don't get "two feet"
> 
```

## <a name="designate-a-field-as-optional-using-the-optional-attribute"></a>Optional 특성을 사용하여 필드를 선택 사항으로 지정

필드를 선택 사항으로 지정하려면 `Optional` 특성을 사용합니다. 이 코드 예제에서는 `Cheese` 필드를 선택 사항으로 지정합니다.

[!code-csharp[Optional attribute](../includes/code/dotnet-formflow-advanced.cs#optionalAttribute)]

필드가 선택 사항이고 값이 지정되지 않은 경우에는 현재 선택 항목이 “기본 설정 없음”으로 표시됩니다.

```console
What kind of cheese would you like on your sandwich? (current choice: No Preference)
  1. American
  2. Monterey Cheddar
  3. Pepperjack
 >
```

필드가 선택 사항이고 사용자가 값을 지정한 경우에는 “기본 설정 없음”이 목록의 마지막 선택 항목으로 표시됩니다.

```console
What kind of cheese would you like on your sandwich? (current choice: American)
 1. American
 2. Monterey Cheddar
 3. Pepperjack
 4. No Preference
>
```

## <a name="match-user-input-using-the-terms-attribute"></a>Terms 특성을 사용하여 사용자 입력 일치

사용자가 FormFlow를 사용하여 빌드된 봇에 메시지를 보내면 봇은 입력을 용어 목록과 일치시켜 사용자 입력의 의미를 파악하려고 합니다. 기본적으로 용어 목록은 필드 또는 값에 다음 단계를 적용하여 생성합니다. 

1. 대/소문자 변경 및 밑줄(_)에서 나눕니다.
2. 각 <a href="https://en.wikipedia.org/wiki/N-gram" target="_blank">n-그램</a>을 최대 길이까지 생성합니다.
3. 각 단어의 끝에 “s?”를 추가합니다(복수 지원). 

예를 들어, “AngusBeefAndGarlicPizza” 값은 다음 용어를 생성합니다. 

- ‘angus?’
- ‘beefs?’
- ‘garlics?’
- ‘pizzas?’
- ‘angus? beefs?’
- ‘garlics? pizzas?’
- ‘angus beef and garlic pizza’

이 기본 동작을 재정의하고 사용자 입력을 필드 또는 필드 값과 일치시키는 데 사용되는 용어 목록을 정의하려면 `Terms` 특성을 사용합니다. 예를 들어, `Terms` 특성(정규식 포함)을 사용하여 사용자가 “rotisserie” 단어의 철자를 잘못 입력할 수 있다는 사실을 고려할 수 있습니다. 

[!code-csharp[Terms attribute](../includes/code/dotnet-formflow-advanced.cs#termsAttribute)]

`Terms` 특성을 사용하여 사용자 입력을 유효한 선택 항목 중 하나와 일치시킬 수 있는 가능성을 높입니다. 이 예제의 `Terms.MaxPhrase` 매개 변수는 `Language.GenerateTerms`에서 용어의 추가 변형을 생성하도록 합니다. 

이 코드 조각은 사용자가 “Rotisserie”의 철자를 잘못 입력할 때 봇과 사용자 간에 생성되는 상호 작용을 보여 줍니다.

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
> rotissary checkin
For sandwich I understood Rotisserie Style Chicken. "checkin" is not an option.
```

## <a name="validate-user-input-using-the-numeric-attribute-or-pattern-attribute"></a>Numeric 특성 또는 Pattern 특성을 사용하여 사용자 입력의 유효성 검사

숫자 필드에 허용되는 값의 범위를 제한하려면 `Numeric` 특성을 사용합니다. 이 코드 예제에서는 `Numeric` 특성을 사용하여 `Rating` 필드에 대한 입력이 1에서 5 사이의 숫자여야 하도록 지정합니다. 

[!code-csharp[Numeric attribute](../includes/code/dotnet-formflow-advanced.cs#numericAttribute)]

특정 필드 값에 필요한 형식을 지정하려면 `Pattern` 특성을 사용합니다. 이 코드 예제에서는 `Pattern` 특성을 사용하여 `PhoneNumber` 필드 값에 필요한 형식을 지정합니다.

[!code-csharp[Pattern attribute](../includes/code/dotnet-formflow-advanced.cs#patternAttribute)]

## <a name="summary"></a>요약

이 문서에서는 FormFlow를 사용하여 초기 양식 상태를 지정하고, 비즈니스 논리를 추가하여 필드 간 상호 종속성을 관리하고 사용자 입력을 처리한 다음, 특성을 사용하여 프롬프트 사용자 지정, 템플릿 재정의, 선택적 필드 지정, 사용자 입력 일치 및 사용자 입력의 유효성 검사를 수행하여 사용자 지정된 사용자 환경을 제공하는 방법을 설명했습니다. FormFlow를 사용하여 사용자 환경을 사용자 지정하는 다른 방법에 대한 자세한 내용은 [FormBuilder를 사용하여 양식 사용자 지정](bot-builder-dotnet-formflow-formbuilder.md)을 참조하세요.

## <a name="sample-code"></a>샘플 코드

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>추가 리소스

- [FormFlow의 기본 기능](bot-builder-dotnet-formflow.md)
- [FormBuilder를 사용하여 양식 사용자 지정](bot-builder-dotnet-formflow-formbuilder.md)
- [양식 콘텐츠 지역화](bot-builder-dotnet-formflow-localize.md)
- [JSON 스키마를 사용하여 양식 정의](bot-builder-dotnet-formflow-json-schema.md)
- [패턴 언어를 사용하여 사용자 환경 사용자 지정](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Builder SDK 참조</a>

[formDialog]: /dotnet/api/microsoft.bot.builder.formflow.formdialog

[promptFieldsWithValues]: /dotnet/api/microsoft.bot.builder.formflow.formoptions.promptfieldswithvalues

[validateResult]: /dotnet/api/microsoft.bot.builder.formflow.validateresult

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[optionalAttribute]: /dotnet/api/microsoft.bot.builder.formflow.optionalattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[termsAttribute]: /dotnet/api/microsoft.bot.builder.formflow.termsattribute

[notUnderstood]: /dotnet/api/microsoft.bot.builder.formflow.templateusage.notunderstood
