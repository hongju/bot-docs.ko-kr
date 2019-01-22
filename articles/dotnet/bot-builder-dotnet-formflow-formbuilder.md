---
title: FormBuilder를 사용하여 양식 사용자 지정 | Microsoft Docs
description: .NET용 Bot Framework SDK용 FormBuilder를 사용하여 대화 흐름 및 콘텐츠를 동적으로 변경하고 사용자 지정하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8a13358afca190606e235475a58f6aedd146fee5
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225408"
---
# <a name="customize-a-form-using-formbuilder"></a>FormBuilder를 사용하여 양식 사용자 지정

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[FormFlow의 기본 기능](bot-builder-dotnet-formflow.md)은 상당히 일반적인 사용자 환경을 제공하는 기본 FormFlow 구현을 설명하고 [FormFlow의 고급 기능](bot-builder-dotnet-formflow-advanced.md)은 비즈니스 논리 및 특성을 사용하여 사용자 환경을 사용자 지정하는 방법을 설명합니다. 이 문서에서는 [FormBuilder][formBuilder]를 사용하여 양식이 단계를 실행하는 순서를 지정하고 필드 값, 확인 및 메시지를 동적으로 정의하여 사용자 환경을 보다 많이 사용자 지정하는 방법에 대해 설명합니다. 

## <a name="dynamically-define-field-values-confirmations-and-messages"></a>필드 값, 확인 및 메시지를 동적으로 정의

FormBuilder를 사용하면 필드 값, 확인 및 메시지를 동적으로 정의할 수 있습니다.

### <a name="dynamically-define-field-values"></a>필드 값을 동적으로 정의 

foot-long(약 30cm) 샌드위치를 명시하는 모든 주문에 무료 음료나 쿠키를 추가하도록 설계된 샌드위치 봇은 `Sandwich.Specials` 필드를 사용하여 무료 항목에 대한 데이터를 저장합니다. 이 경우 `Sandwich.Specials` 필드의 값은 주문에 foot-long(약 30cm) 샌드위치 포함 여부에 따라 각 주문에 대해 동적으로 설정되어야 합니다. 

`Specials` 필드는 선택 사항으로 지정되며 "None"은 선호 항목이 없음을 나타내는 선택 항목의 텍스트로 지정됩니다.

[!code-csharp[Field definition](../includes/code/dotnet-formflow-formbuilder.cs#fieldDefinition)]

이 코드 예제는 `Specials` 필드의 값을 동적으로 설정하는 방법을 보여줍니다. 

[!code-csharp[Define value](../includes/code/dotnet-formflow-formbuilder.cs#defineValue)]

이 예제에서 [Advanced.Field.SetType][setType] 메서드는 필드 형식(`null`은 열거형 필드를 나타냄)을 지정합니다. [Advanced.Field.SetActive][setActive] 메서드는 샌드위치의 길이가 `Length.FootLong`인 경우에만 활성화되는 것으로 지정합니다. 마지막으로 [Advanced.Field.SetDefine][setDefine] 메서드는 필드를 정의하는 비동기 대리자를 지정합니다. 대리자에게는 현재 상태 개체와 동적으로 정의되는 [Advanced.Field][field]가 전달됩니다. 대리자는 필드의 유창한(fluent) 메서드를 사용하여 값을 동적으로 정의합니다. 이 예제에서 값은 문자열이고 `AddDescription` 및 `AddTerms` 메서드는 각 값에 대한 설명과 용어를 지정합니다.

> [!NOTE]
> 필드 값을 동적으로 정의하려면 [Advanced.IField][iField]를 직접 구현하거나 위의 예제와 같이 [Advanced.FieldReflector][FieldReflector]클래스를 사용하여 프로세스를 간소화할 수 있습니다. 

### <a name="dynamically-define-messages-and-confirmations"></a>동적으로 메시지 및 확인 정의

FormBuilder를 사용하면 메시지와 확인을 동적으로 정의할 수 있습니다. 각 메시지 및 확인은 양식의 이전 단계가 비활성이거나 완료된 경우에만 실행됩니다. 

이 코드 예제는 샌드위치의 비용을 계산하는 동적으로 생성된 확인을 보여줍니다. 

[!code-csharp[Define confirmation](../includes/code/dotnet-formflow-formbuilder.cs#defineConfirmation)]

## <a name="customize-a-form-using-formbuilder"></a>FormBuilder를 사용하여 양식 사용자 지정

이 코드 예제에는 FormBuilder를 사용하여 양식의 단계를 정의하고 [선택 항목의 유효성을 검사](bot-builder-dotnet-formflow-advanced.md#add-business-logic)하며 [필드 값과 확인을 동적으로 정의](#dynamically-define-field-values-confirmations-and-messages)합니다. 기본적으로 양식의 단계는 나열된 순서로 실행됩니다. 단, 이미 값이 포함된 필드 또는 명시적 탐색이 지정된 경우 단계가 생략될 수 있습니다. 

[!code-csharp[FormBuilder form](../includes/code/dotnet-formflow-formbuilder.cs#formBuilderForm)]

이 예제에서 양식은 다음 단계를 실행합니다.

- 환영 메시지를 표시합니다. 
- `SandwichOrder.Sandwich`를 채웁니다. 
- `SandwichOrder.Length`를 채웁니다. 
- `SandwichOrder.Bread`를 채웁니다. 
- `SandwichOrder.Cheese`를 채웁니다. 
- `SandwichOrder.Toppings`를 채우고 사용자가 `ToppingOptions.Everything`을 선택한 경우 없는 값을 추가합니다. -. 선택한 토핑을 확인하는 메시지를 표시합니다. 
- `SandwichOrder.Sauces`를 채웁니다. 
- `SandwichOrder.Specials`의 필드 값을 [동적으로 정의](#dynamically-define-field-values)합니다. 
- 샌드위치의 비용에 대한 확인을 [동적으로 정의](#dynamically-define-messages-and-confirmations)합니다. 
- `SandwichOrder.DeliveryAddress`를 채우고 결과 문자열을 [확인](bot-builder-dotnet-formflow-advanced.md#add-business-logic)합니다. 주소가 숫자로 시작하지 않으면 양식에서 메시지를 반환합니다. 
- 사용자 지정 프롬프트로 `SandwichOrder.DeliveryTime`을 채웁니다. 
- 주문을 확인합니다. 
- 클래스에 정의되었지만 `Field`에 명시적으로 참조되지 않은 나머지 필드를 추가합니다. (예제에서 `AddRemainingFields` 메서드를 호출하지 않으면 양식은 명시적으로 참조되지 않은 필드를 포함하지 않습니다.) 
- 감사 메시지를 표시합니다. 
- 주문을 처리할 `OnCompletionAsync` 처리기를 정의합니다. 

## <a name="sample-code"></a>샘플 코드

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>추가 리소스

- [FormFlow의 기본 기능](bot-builder-dotnet-formflow.md)
- [FormFlow의 고급 기능](bot-builder-dotnet-formflow-advanced.md)
- [양식 콘텐츠 지역화](bot-builder-dotnet-formflow-localize.md)
- [JSON 스키마를 사용하여 양식 정의](bot-builder-dotnet-formflow-json-schema.md)
- [패턴 언어를 사용하여 사용자 환경 사용자 지정](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Framework SDK 참조</a>

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1

[setType]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.settype

[setActive]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setactive

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[FieldReflector]: /dotnet/api/microsoft.bot.builder.formflow.advanced.fieldreflector-1
