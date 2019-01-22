---
title: JSON 스키마 및 FormFlow를 사용하여 양식 정의 | Microsoft Docs
description: .NET용 Bot Framework SDK에서 JSON 스키마 및 FormFlow를 사용하여 양식을 정의하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d84252281baa57a15b093cfd0ba92fe5fe422027
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225858"
---
# <a name="define-a-form-using-json-schema"></a>JSON 스키마를 사용하여 양식 정의

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

FormFlow로 봇을 만들 때 [C# 클래스](bot-builder-dotnet-formflow.md#create-class)를 사용하여 양식을 정의하면 C# 형식의 정적 정의에서 양식이 파생됩니다. 아니면 <a href="http://json-schema.org/documentation.html" target="_blank">JSON 스키마</a>를 대신 사용하여 양식을 정의할 수 있습니다. JSON 스키마를 사용하여 정의된 양식은 순전히 데이터 기반입니다. 스키마를 업데이트하여 양식(따라서, 봇의 동작)을 간단히 변경할 수 있습니다. 

JSON 스키마는 <a href="http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JObject.htm" target="_blank">JObject</a> 내의 필드를 설명하고 프롬프트, 템플릿 및 용어를 제어하는 주석을 포함합니다. FormFlow에서 JSON 스키마를 사용하려면 `Microsoft.Bot.Builder.FormFlow.Json` NuGet 패키지를 프로젝트에 추가하고 `Microsoft.Bot.Builder.FormFlow.Json` 네임스페이스를 가져와야 합니다.

## <a name="standard-keywords"></a>표준 키워드 

FormFlow는 다음과 같은 표준 <a href="http://json-schema.org/documentation.html" target="_blank">JSON 스키마</a> 키워드를 지원합니다.

| 키워드 | 설명 | 
|----|----|
| 형식 | 필드에 포함된 데이터 형식을 정의합니다. |
| enum | 필드에 유효한 값을 정의합니다. |
| minimum | 필드에 허용되는 최소 숫자 값을 정의합니다([NumericAttribute][numericAttribute] 참조). |
| maximum | 필드에 허용되는 최대 숫자 값을 정의합니다([NumericAttribute][numericAttribute] 참조). |
| 필수 | 필수 필드를 정의합니다. |
| 패턴 | 문자열 값의 유효성 검사합니다([PatternAttribute][patternAttribute] 참조). |

## <a name="extensions-to-json-schema"></a>JSON 스키마에 대한 확장

FormFlow는 표준 <a href="http://json-schema.org/documentation.html" target="_blank">JSON 스키마</a>를 확장하여 몇 가지 추가 속성을 지원합니다.

### <a name="additional-properties-at-the-root-of-the-schema"></a>스키마 루트의 추가 특성

| 자산 | 값 |
|----|----|
| OnCompletion | 양식을 완료하기 위한 인수 `(IDialogContext context, JObject state)`가 있는 C# 스크립트입니다. |
| 참조 | 스크립트에 포함할 참조입니다. 예: `[assemblyReference, ...]` 경로는 절대 경로 또는 현재 디렉터리의 상대 경로여야 합니다. 기본적으로 스크립트에 `Microsoft.Bot.Builder.dll`이 포함됩니다. |
| 가져오기 | 스크립트에 포함할 가져오기입니다. 예: `[import, ...]` 기본적으로 스크립트에 `Microsoft.Bot.Builder`, `Microsoft.Bot.Builder.Dialogs`, `Microsoft.Bot.Builder.FormFlow`, `Microsoft.Bot.Builder.FormFlow.Advanced`, `System.Collections.Generic` 및 `System.Linq` 네임스페이스가 포함됩니다. |

### <a name="additional-properties-at-the-root-of-the-schema-or-as-peers-of-the-type-property"></a>스키마 루트의 추가 특성 또는 type 속성의 피어인 추가 속성

| 자산 | 값 |
|----|----|
| 템플릿 | `{ TemplateUsage: { Patterns: [string, ...], <args> }, ...}` |
| prompt | `{ Patterns:[string, ...] <args>}` |

JSON 스키마로 템플릿과 프롬프트를 지정하려면, [TemplateAttribute][templateAttribute] 및 [PromptAttribute][promptAttribute]에 정의된 어휘와 동일한 어휘를 사용합니다. 스키마의 속성 이름과 값은 기본 C# 열거형의 속성 이름 및 값과 일치해야 합니다. 예를 들어 이 스키마 코드 조각은 `TemplateUsage.NotUnderstood` 템플릿을 재정의하고 `TemplateBaseAttribute.ChoiceStyle`을 지정하는 템플릿을 정의합니다. 

```json
"Templates":{ "NotUnderstood": { "Patterns": ["I don't get it"], "ChoiceStyle":"Auto"}}
```

### <a name="additional-properties-as-peers-of-the-type-property"></a>type 속성의 피어인 추가 속성

|   자산   |          목차           |                                                   설명                                                    |
|--------------|-----------------------------|------------------------------------------------------------------------------------------------------------------|
|   Datetime   |            bool             |                                  필드가 `DateTime` 필드인지 나타냅니다.                                  |
|   Describe   |      문자열 또는 개체       |                  [DescribeAttribute][describeAttribute]에 설명된 대로 필드의 설명입니다.                  |
|    용어     |       `[string,...]`        |                  TermsAttribute에 설명된 대로 필드 값을 일치시키는 정규식입니다.                  |
|  MaxPhrase   |             int             |                  `Language.GenerateTerms(string, int)`를 통해 용어를 실행하여 확장합니다.                   |
|    값    | `{ string: {Describe:string |                                  object, Terms:[string, ...], MaxPhrase}, ...}`                                  |
|    Active    |           script            | `(JObject state)->bool` 인수로 필드, 메시지 또는 확인이 활성 상태인지 여부를 테스트하는 C# 스크립트입니다.  |
|   유효성 검사   |           script            |      `(JObject state, object value)->ValidateResult` 인수로 필드 값의 유효성을 검사하는 C# 스크립트입니다.      |
|    Define    |           script            |        `(JObject state, Field<JObject> field)` 인수를 사용하여 필드를 동적으로 정의하는 C# 스크립트입니다.        |
|     다음     |           script            | `(object value, JObject state)`로 필드를 채운 후 다음 단계를 결정하는 C# 스크립트입니다. |
|    이전    |          `[confirm          |                                                  message, ...]`                                                  |
|    이후     |          `[confirm          |                                                  message, ...]`                                                  |
| 종속성 |        [string, ...]        |                           이 필드, 메시지 또는 확인이 사용되는 필드입니다.                           |

**Before** 속성이나 **After** 속성의 값 내에 `{Confirm:script|[string, ...], ...templateArgs}` 를 사용하여 `(JObject state)` 인수가 있는 C# 스크립트나 선택적 템플릿 인수로 무작위로 선택되는 패턴 집합을 사용한 확인을 정의합니다.

**Before** 속성이나 **After** 속성의 값 내에 `{Message:script|[string, ...] ...templateArgs}` 를 사용하여 `(JObject state)` 인수가 있는 C# 스크립트나 선택적 템플릿 인수로 무작위로 선택되는 패턴 집합을 사용한 메시지를 정의합니다.

## <a name="scripts"></a>스크립트

위에 설명된 속성 중 일부는 스크립트를 속성 값으로 포함합니다. 스크립트는 대개 메서드 본문에서 찾을 수 있는 C# 코드 조각일 수 있습니다. **References** 속성 및/또는 **Imports** 속성을 사용하여 참조를 추가할 수 있습니다. 특수 글로벌 변수는 다음과 같습니다.

| 변수 | 설명 |
|----|----|
| choice | 스크립트를 실행하기 위한 내부 디스패치입니다. |
| state | 모든 스크립트에 대한 `JObject` 양식 상태입니다. |
| ifield | Message/Confirm 프롬프트 빌더를 제외한 모든 스크립트에 대해 현재 필드에 대한 추론을 허용하는 `IField<JObject>`입니다. |
| 값 | **Validate**을 위해 유효성이 검사될 개체 값입니다. |
| 필드 | **Define**의 필드를 동적으로 업데이트하도록 허용하는 `Field<JObject>`입니다. |
| context | **OnCompletion**에 게시 결과를 허용하는 `IDialogContext` 컨텍스트입니다. |

JSON 스키마를 통해 정의된 필드는 다른 필드와 마찬가지로 프로그래밍 방식으로 정의를 확장하거나 재정의하는 기능을 갖습니다. 또한 같은 방식으로 지역화할 수 있습니다.

## <a name="json-schema-example"></a>JSON 스키마 예제

양식을 정의하는 가장 간단한 방법은 C# 코드를 비롯한 모든 코드를 JSON 스키마에 직접 정의하는 것입니다. 다음 예제는 [FormBuilder를 사용하여 양식 사용자 지정](bot-builder-dotnet-formflow-formbuilder.md)에 설명된 주석이 지정된 샌드위치 봇에 대한 JSON 스키마를 보여줍니다.

```json
{
  "References": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.dll" ],
  "Imports": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.Resource" ],
  "type": "object",
  "required": [
    "Sandwich",
    "Length",
    "Ingredients",
    "DeliveryAddress"
  ],
  "Templates": {
    "NotUnderstood": {
      "Patterns": [ "I do not understand \"{0}\".", "Try again, I don't get \"{0}\"." ]
    },
    "EnumSelectOne": {
      "Patterns": [ "What kind of {&} would you like on your sandwich? {||}" ],
      "ChoiceStyle": "Auto"
    }
  },
  "properties": {
    "Sandwich": {
      "Prompt": { "Patterns": [ "What kind of {&} would you like? {||}" ] },
      "Before": [ { "Message": [ "Welcome to the sandwich order bot!" ] } ],
      "Describe": { "Image": "https://placeholdit.imgix.net/~text?txtsize=16&txt=Sandwich&w=125&h=40&txttrack=0&txtclr=000&txtfont=bold" },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "BLT",
        "BlackForestHam",
        "BuffaloChicken",
        "ChickenAndBaconRanchMelt",
        "ColdCutCombo",
        "MeatballMarinara",
        "OvenRoastedChicken",
        "RoastBeef",
        "RotisserieStyleChicken",
        "SpicyItalian",
        "SteakAndCheese",
        "SweetOnionTeriyaki",
        "Tuna",
        "TurkeyBreast",
        "Veggie"
      ],
      "Values": {
        "RotisserieStyleChicken": {
          "Terms": [ "rotis\\w* style chicken" ],
          "MaxPhrase": 3
        }
      }
    },
    "Length": {
      "Prompt": {
        "Patterns": [ "What size of sandwich do you want? {||}" ]
      },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "SixInch",
        "FootLong"
      ]
    },
    "Ingredients": {
      "type": "object",
      "required": [ "Bread" ],
      "properties": {
        "Bread": {
          "type": [
            "string",
            "null"
          ],
          "Describe": {
            "Title": "Sandwich Bot",
            "SubTitle": "Bread Picker"
          },
          "enum": [
            "NineGrainWheat",
            "NineGrainHoneyOat",
            "Italian",
            "ItalianHerbsAndCheese",
            "Flatbread"
          ]
        },
        "Cheese": {
          "type": [
            "string",
            "null"
          ],
          "enum": [
            "American",
            "MontereyCheddar",
            "Pepperjack"
          ]
        },
        "Toppings": {
          "type": "array",
          "items": {
            "type": "integer",
            "enum": [
              "Everything",
              "Avocado",
              "BananaPeppers",
              "Cucumbers",
              "GreenBellPeppers",
              "Jalapenos",
              "Lettuce",
              "Olives",
              "Pickles",
              "RedOnion",
              "Spinach",
              "Tomatoes"
            ],
            "Values": {
              "Everything": { "Terms": [ "except", "but", "not", "no", "all", "everything" ] }
            }
          },
          "Validate": "var values = ((List<object>) value).OfType<string>(); var result = new ValidateResult {IsValid = true, Value = values} ; if (values != null && values.Contains(\"Everything\")) { result.Value = (from topping in new string[] {  \"Avocado\", \"BananaPeppers\", \"Cucumbers\", \"GreenBellPeppers\", \"Jalapenos\", \"Lettuce\", \"Olives\", \"Pickles\", \"RedOnion\", \"Spinach\", \"Tomatoes\"} where !values.Contains(topping) select topping).ToList();} return result;",
          "After": [ { "Message": [ "For sandwich toppings you have selected {Ingredients.Toppings}." ] } ]
        },
        "Sauces": {
          "type": [
            "array",
            "null"
          ],
          "items": {
            "type": "string",
            "enum": [
              "ChipotleSouthwest",
              "HoneyMustard",
              "LightMayonnaise",
              "RegularMayonnaise",
              "Mustard",
              "Oil",
              "Pepper",
              "Ranch",
              "SweetOnion",
              "Vinegar"
            ]
          }
        }
      }
    },
    "Specials": {
      "Templates": {
        "NoPreference": { "Patterns": [ "None" ] }
      },
      "type": [
        "string",
        "null"
      ],
      "Active": "return (string) state[\"Length\"] == \"FootLong\";",
      "Define": "field.SetType(null).AddDescription(\"cookie\", DynamicSandwich.FreeCookie).AddTerms(\"cookie\", Language.GenerateTerms(DynamicSandwich.FreeCookie, 2)).AddDescription(\"drink\", DynamicSandwich.FreeDrink).AddTerms(\"drink\", Language.GenerateTerms(DynamicSandwich.FreeDrink, 2)); return true;",
      "After": [ { "Confirm": "var cost = 0.0; switch ((string) state[\"Length\"]) { case \"SixInch\": cost = 5.0; break; case \"FootLong\": cost=6.50; break;} return new PromptAttribute($\"Total for your sandwich is {cost:C2} is that ok?\");" } ]
    },
    "DeliveryAddress": {
      "type": [
        "string",
        "null"
      ],
      "Validate": "var result = new ValidateResult{ IsValid = true, Value = value}; var address = (value as string).Trim(); if (address.Length > 0 && (address[0] < '0' || address[0] > '9')) {result.Feedback = DynamicSandwich.BadAddress; result.IsValid = false; } return result;"
    },
    "PhoneNumber": {
      "type": [ "string", "null" ],
      "pattern": "(\\(\\d{3}\\))?\\s*\\d{3}(-|\\s*)\\d{4}"
    },
    "DeliveryTime": {
      "Templates": {
        "StatusFormat": {
          "Patterns": [ "{&}: {:t}" ],
          "FieldCase": "None"
        }
      },
      "DateTime": true,
      "type": [
        "string",
        "null"
      ],
      "After": [ { "Confirm": [ "Do you want to order your {Length} {Sandwich} on {Ingredients.Bread} {&Ingredients.Bread} with {[{Ingredients.Cheese} {Ingredients.Toppings} {Ingredients.Sauces} to be sent to {DeliveryAddress} {?at {DeliveryTime}}?" ] } ]
    },
    "Rating": {
      "Describe": "your experience today",
      "type": [
        "number",
        "null"
      ],
      "minimum": 1,
      "maximum": 5,
      "After": [ { "Message": [ "Thanks for ordering your sandwich!" ] } ]
    }
  },
  "OnCompletion": "await context.PostAsync(\"We are currently processing your sandwich. We will message you the status.\");"
}
```

## <a name="implement-formflow-with-json-schema"></a>JSON 스키마로 FormFlow 구현

JSON 스키마로 FormFlow를 구현하려면 `FormBuilder`와 동일한 흐름 인터페이스를 지원하는 `FormBuilderJson`을 사용합니다. 다음 코드 예제는 [FormBuilder를 사용하여 양식 사용자 지정](bot-builder-dotnet-formflow-formbuilder.md)에 설명된 주석이 지정된 샌드위치 봇에 대한 JSON 스키마를 구현하는 방법을 보여줍니다.

[!code-csharp[Use JSON schema](../includes/code/dotnet-formflow-json-schema.cs#useSchema)]

## <a name="sample-code"></a>샘플 코드

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>추가 리소스

- [FormFlow의 기본 기능](bot-builder-dotnet-formflow.md)
- [FormFlow의 고급 기능](bot-builder-dotnet-formflow-advanced.md)
- [FormBuilder를 사용하여 양식 사용자 지정](bot-builder-dotnet-formflow-formbuilder.md)
- [양식 콘텐츠 지역화](bot-builder-dotnet-formflow-localize.md)
- [패턴 언어를 사용하여 사용자 환경 사용자 지정](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Framework SDK 참조</a>

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute
