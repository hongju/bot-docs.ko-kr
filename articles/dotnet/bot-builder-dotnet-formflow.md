---
title: FormFlow의 기본 기능 | Microsoft Docs
description: .NET용 Bot Builder SDK 내에서 FormFlow를 사용하여 대화 흐름을 안내하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f5b36e1f916539b78f9bdcdd0970317db723f408
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000380"
---
# <a name="basic-features-of-formflow"></a>FormFlow의 기본 기능

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[다이얼로그](bot-builder-dotnet-dialogs.md)는 매우 강력하고 유연하지만 샌드위치 주문과 같은 안내형 대화를 처리하려면 많은 노력이 필요할 수 있습니다. 대화의 각 지점에서는 진행될 다음 과정에 대한 여러 가능성이 있습니다. 예를 들어, 모호성을 없애거나, 도움말을 제공하거나, 뒤로 이동하거나, 진행 상태를 표시해야 할 수 있습니다. .NET용 Bot Builder SDK 내에서 **FormFlow**를 사용하여 이와 같은 안내형 대화를 관리하는 프로세스를 크게 간소화할 수 있습니다. 

FormFlow는 지정된 지침에 따라, 안내형 대화를 관리하는 데 필요한 다이얼로그를 자동으로 생성합니다. FormFlow를 사용하면 직접 다이얼로그를 만들고 관리할 때 얻을 수 있는 유연성이 다소 손실되지만 FormFlow를 사용하여 안내형 대화를 디자인하면 봇 개발에 소요되는 시간을 크게 단축할 수 있습니다. 또한 FormFlow 생성 다이얼로그 및 다른 유형의 다이얼로그 조합을 사용하여 봇을 구성할 수도 있습니다. 예를 들어 [LuisDialog][LuisDialog]는 사용자 입력을 평가하여 의도를 파악하지만 FormFlow 다이얼로그는 폼을 완성하는 프로세스를 안내할 수 있습니다.

이 문서에서는 FormFlow의 기본 기능을 사용하여 사용자로부터 정보를 수집하는 봇을 만드는 방법을 설명합니다.

## <a id="forms-and-fields"></a>폼 및 필드

FormFlow를 사용하여 봇을 만들려면 봇이 사용자로부터 수집해야 하는 정보를 지정해야 합니다. 예를 들어, 봇의 목표가 사용자의 샌드위치 주문을 받는 것인 경우 봇이 주문을 이행하는 데 필요한 데이터를 위한 필드를 포함하는 폼을 정의해야 합니다. 봇이 사용자로부터 수집할 데이터를 나타내는 하나 이상의 공용 속성이 포함된 C# 클래스를 만들어 폼을 정의할 수 있습니다. 각 속성은 다음 데이터 형식 중 하나여야 합니다.

- 정수(sbyte, byte, short, ushort, int, uint, long, ulong)
- 부동 소수점(float, double)
- 문자열
- Datetime
- 열거형
- 열거형 목록

데이터 형식은 null일 수 있습니다. 이 경우 필드에 값이 없도록 모델링할 수 있습니다. 폼 필드가 null을 허용하지 않는 열거형 속성을 기준으로 할 경우 열거형의 값 **0**은 **null**을 나타내고(즉, 필드에 없이 없음을 나타냄) **1**에서 열거형 값을 시작해야 합니다. FormFlow는 다른 모든 속성 형식 및 메서드를 무시합니다.

복합 개체의 경우 최상위 C# 클래스에 대한 폼과 복합 개체에 대한 다른 폼을 만들어야 합니다. 일반적인 [다이얼로그](bot-builder-dotnet-dialogs.md)를 사용하여 폼을 함께 구성할 수 있습니다. [Advanced.IField][iField]를 구현하거나 [Advanced.Field][field]를 사용하고 내부에 사전을 채워서 폼을 직접 정의할 수도 있습니다. 

> [!NOTE]
> C# 클래스 또는 JSON 스키마를 사용하여 폼을 정의할 수 있습니다. 이 문서에서는 C# 클래스를 사용하여 폼을 정의하는 방법을 설명합니다. JSON 스키마를 사용하는 방법에 대한 자세한 내용은 [JSON 스키마를 사용하여 폼 정의](bot-builder-dotnet-formflow-json-schema.md)를 참조하세요.

## <a name="simple-sandwich-bot"></a>간단한 샌드위치 봇

샌드위치 주문을 받도록 디자인된 간단한 샌드위치 봇의 다음 예제를 살펴보세요. 

### <a id="create-class"></a> 폼 만들기

`SandwichOrder` 클래스는 폼을 정의하고, 열거형은 샌드위치를 작성하기 위한 옵션을 정의합니다. 또한 이 클래스에는 [FormBuilder][formBuilder]를 사용하여 폼을 만들고 간단한 환영 메시지를 정의하는 정적 `BuildForm` 메서드도 포함되어 있습니다. 

FormFlow를 사용하려면 먼저 `Microsoft.Bot.Builder.FormFlow` 네임스페이스를 가져와야 합니다.

[!code-csharp[Define form](../includes/code/dotnet-formflow.cs#defineForm)]

### <a name="connect-the-form-to-the-framework"></a>폼을 프레임워크에 연결 

폼을 프레임워크에 연결하려면 컨트롤러에 추가해야 합니다. 이 예제에서 `Conversation.SendAsync` 메서드는 정적 `MakeRootDialog` 메서드를 호출한 후 `FormDialog.FromForm` 메서드를 호출하여 `SandwichOrder` 폼을 만듭니다. 

[!code-csharp[Connect form to framework](../includes/code/dotnet-formflow.cs#connectToFramework)]

### <a name="see-it-in-action"></a>실제 동작 확인

간단히 C# 클래스를 사용하여 폼을 정의하고 프레임워크에 연결하여 봇과 사용자 간의 대화를 자동으로 관리하도록 FormFlow를 설정했습니다. 아래에 나오는 예제 상호 작용은 FormFlow의 기본 기능을 사용하여 만든 봇 기능을 보여 줍니다. 각 상호 작용에서 **>** 기호는 사용자가 응답을 입력하는 시점을 나타냅니다. 

#### <a name="display-the-first-prompt"></a>첫 번째 프롬프트 표시 

이 폼은 `SandwichOrder.Sandwich` 속성을 채웁니다. 이 폼은 "Please select a sandwich" 프롬프트를 자동으로 생성합니다. 이 프롬프트의 단어 "sandwich"는 속성 이름 `Sandwich`에서 파생됩니다. `SandwichOptions` 열거형은 사용자에게 제공되는 선택 옵션을 정의하며, 각 열거형 값은 대/소문자 및 밑줄 변경에 따라 자동으로 단어로 구분됩니다.

```console
Please select a sandwich
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

#### <a name="provide-guidance"></a>지침 제공

사용자는 대화 중 어떤 지점에서든지 “help”를 입력하여 폼을 채우기 위한 지침을 얻을 수 있습니다. 예를 들어, 샌드위치 프롬프트에서 "help"를 입력하면 봇은 이 지침에 따라 응답합니다. 

```console
> help
* You are filling in the sandwich field. Possible responses:
* You can enter a number 1-15 or words from the descriptions. (BLT, Black Forest Ham, Buffalo Chicken, Chicken And Bacon Ranch Melt, Cold Cut Combo, Meatball Marinara, Oven Roasted Chicken, Roast Beef, Rotisserie Style Chicken, Spicy Italian, Steak And Cheese, Sweet Onion Teriyaki, Tuna, Turkey Breast, and Veggie)
* Back: Go back to the previous question.
* Help: Show the kinds of responses you can enter.
* Quit: Quit the form without completing it.
* Reset: Start over filling in the form. (With defaults from your previous entries.)
* Status: Show your progress in filling in the form so far.
* You can switch to another field by entering its name. (Sandwich, Length, Bread, Cheese, Toppings, and Sauce).
```

#### <a name="advance-to-the-next-prompt"></a>다음 프롬프트로 진행

사용자가 초기 샌드위치 프롬프트에 대한 응답으로 "2"를 입력하면 봇은 폼으로 정의되는 `SandwichOrder.Length` 속성에 대한 프롬프트를 표시합니다.

```console
Please select a sandwich
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
> 2
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="return-to-the-previous-prompt"></a>이전 페이지로 돌아가기 

사용자가 대화의 이 시점에서 "back"을 입력하면 봇은 이전 프롬프트를 반환합니다. 프롬프트는 사용자의 현재 선택 항목("Black Forest Ham")을 표시합니다. 사용자는 다른 숫자를 입력하여 해당 선택을 변경하거나 “c”를 입력하여 선택을 확인할 수 있습니다.

```console
> back
Please select a sandwich(current choice: Black Forest Ham)
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
> c
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="clarify-user-input"></a>사용자 입력 확인

사용자가 선택 항목을 나타내기 위해 텍스트(숫자 대신)로 응답했으나 사용자 입력이 둘 이상의 선택 항목과 일치할 경우 봇은 자동으로 확인을 요청합니다. 

```console
Please select a bread
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> nine grain
By "nine grain" bread did you mean (1. Nine Grain Honey Oat, 2. Nine Grain Wheat)
> 1
```

사용자 입력이 유효한 선택 항목과 일치하지 않으면 봇은 사용자에게 확인용 메시지를 자동으로 표시합니다.

```console
Please select a cheese (1. American, 2. Monterey Cheddar, 3. Pepperjack)
> amercan
"amercan" is not a cheese option.
> american smoked
For cheese I understood American. "smoked" is not an option.
```

사용자 입력이 하나의 속성에 대해 여러 선택 항목을 지정하고 봇이 지정된 선택 항목을 이해하지 못하면 봇은 사용자에게 확인용 메시지를 자동으로 표시합니다.

```console
Please select one or more toppings
 1. Banana Peppers
 2. Cucumbers
 3. Green Bell Peppers
 4. Jalapenos
 5. Lettuce
 6. Olives
 7. Pickles
 8. Red Onion
 9. Spinach
 10. Tomatoes
> peppers, lettuce and tomato
By "peppers" toppings did you mean (1. Green Bell Peppers, 2. Banana Peppers)
> 1
```

#### <a name="show-current-status"></a>현재 상태 표시

사용자가 주문 중 언제든지 “status”를 입력하면 봇의 응답은 이미 지정된 값과 지정될 값을 나타냅니다. 

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> status
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Unspecified  
```

#### <a name="confirm-selections"></a>선택 확인

사용자가 폼을 완료하면 봇은 선택을 확인하라는 메시지를 표시합니다.

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> 1
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
>
```

사용자는 "no"를 입력한 경우 봇에서 이전 선택을 업데이트할 수 있습니다. “yes”를 입력한 경우 폼이 완료되며 제어 권한이 호출 다이얼로그로 반환됩니다. 

```console
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> no
What do you want to change?
 1. Sandwich(Black Forest Ham)
 2. Length(Six Inch)
 3. Bread(Nine Grain Honey Oat)
 4. Cheese(American)
 5. Toppings(Lettuce, Tomatoes, and Green Bell Peppers)
 6. Sauce(Honey Mustard)
> 2
Please select a length (current choice: Six Inch) (1. Six Inch, 2. Foot Long)
> 2
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Foot Long
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> y
```

## <a name="handling-quit-and-exceptions"></a>처리 취소 및 예외

사용자가 폼에 “quit”을 입력하거나 대화 중 특정 시점에 예외가 발생하면 봇은 이벤트가 발생한 단계, 이벤트가 발생했을 때의 폼 상태 및 이벤트 전에 성공적으로 완료된 폼 단계를 알아야 합니다. 폼은 `FormCanceledException<T>` 클래스를 통해 이 정보를 반환합니다. 

이 코드 예제에서는 예외를 catch하고 발생한 이벤트에 따라 메시지를 표시하는 방법을 보여 줍니다. 

[!code-csharp[Handle exception or quit](../includes/code/dotnet-formflow.cs#handleExceptionOrQuit)]

## <a name="summary"></a>요약

이 문서에서는 FormFlow의 기본 기능을 사용하여 다음을 할 수 있는 봇을 만드는 방법을 설명했습니다.

- 대화를 자동으로 생성하고 관리
- 명확한 지침 및 도움말 제공
- 숫자와 텍스트 항목 이해
- 이해된 내용과 이해되지 않은 내용에 대해 사용자에게 피드백 제공 
- 필요한 경우 확인용 질문 제시 
- 단계 간 이동

경우에 따라 기본 FormFlow 기능만으로 충분하지만, FormFlow의 일부 고급 기능을 봇에 통합할 때 얻을 수 있는 이점도 고려해야 합니다. 자세한 내용은 [FormFlow의 고급 기능](bot-builder-dotnet-formflow-advanced.md) 및 [FormBuilder를 사용하여 양식 사용자 지정](bot-builder-dotnet-formflow-formbuilder.md)을 참조하세요.

## <a name="sample-code"></a>샘플 코드

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="next-steps"></a>다음 단계

FormFlow는 다이얼로그 개발을 간소화합니다. FormFlow의 고급 기능을 통해 FormFlow 개체의 동작 방식을 사용자 지정할 수 있습니다.

> [!div class="nextstepaction"]
> [FormFlow의 고급 기능](bot-builder-dotnet-formflow-advanced.md)

## <a name="additional-resources"></a>추가 리소스

- [FormBuilder를 사용하여 양식 사용자 지정](bot-builder-dotnet-formflow-formbuilder.md)
- [양식 콘텐츠 지역화](bot-builder-dotnet-formflow-localize.md)
- [JSON 스키마를 사용하여 양식 정의](bot-builder-dotnet-formflow-json-schema.md)
- [패턴 언어를 사용하여 사용자 환경 사용자 지정](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Builder SDK 참조</a>

[LuisDialog]: /dotnet/api/microsoft.bot.builder.dialogs.luisdialog-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1
