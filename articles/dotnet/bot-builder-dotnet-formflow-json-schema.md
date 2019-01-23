---
title: JSON スキーマと FormFlow を使用したフォームの定義 | Microsoft Docs
description: Bot Framework SDK for .NET で JSON スキーマと FormFlow を使用してフォームを定義する方法を説明します。
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
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225857"
---
# <a name="define-a-form-using-json-schema"></a>JSON スキーマによるフォームの定義

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

FormFlow を使用してボットを作成するときに [C# クラス](bot-builder-dotnet-formflow.md#create-class)を使ってフォームを定義する場合、フォームは、C# の型の静的定義から派生します。 代わりに、<a href="http://json-schema.org/documentation.html" target="_blank">JSON スキーマ</a>を使用してフォームを定義することもできます。 JSON スキーマを使用して定義したフォームは純粋にデータ ドリブンです。スキーマを更新するだけで、フォームを (したがってボットの動作も) 変更できます。 

JSON スキーマは <a href="http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JObject.htm" target="_blank">JObject</a> 内でフィールドを記述します。JSON スキーマには、プロンプト、テンプレート、用語を制御する注釈が含まれています。 JSON スキーマと FormFlow を併用するには、`Microsoft.Bot.Builder.FormFlow.Json` NuGet パッケージをプロジェクトに追加し、`Microsoft.Bot.Builder.FormFlow.Json` 名前空間をインポートする必要があります。

## <a name="standard-keywords"></a>標準キーワード 

FormFlow は、以下の標準 <a href="http://json-schema.org/documentation.html" target="_blank">JSON スキーマ</a> キーワードをサポートしています。

| Keyword | 説明 | 
|----|----|
| type | フィールドに入るデータの型を定義します。 |
| enum | フィールドの有効な値を定義します。 |
| minimum | フィールドで許可される最小の数値を定義します ([NumericAttribute][numericAttribute] で説明)。 |
| maximum | フィールドで許可される最大の数値を定義します ([NumericAttribute][numericAttribute] で説明)。 |
| 必須 | どれが必須フィールドであるかを定義します。 |
| pattern | 文字列値を検証します ([PatternAttribute][patternAttribute] で説明)。 |

## <a name="extensions-to-json-schema"></a>JSON スキーマの拡張

FormFlow では、いくつかの追加プロパティをサポートするよう、標準 <a href="http://json-schema.org/documentation.html" target="_blank">JSON スキーマ</a>が拡張されています。

### <a name="additional-properties-at-the-root-of-the-schema"></a>スキーマのルートにある追加プロパティ

| プロパティ | 値 |
|----|----|
| OnCompletion | フォームを完了処理するための C# スクリプト (引数 `(IDialogContext context, JObject state)` を使用)。 |
| 参照 | スクリプトに含める参照。 たとえば、「 `[assemblyReference, ...]` 」のように入力します。 パスは、絶対パスか、現在のディレクトリから見た相対パスにする必要があります。 既定では、スクリプトには `Microsoft.Bot.Builder.dll` が含まれます。 |
| インポートする | スクリプトに含めるインポート。 たとえば、「 `[import, ...]` 」のように入力します。 既定では、スクリプトには `Microsoft.Bot.Builder`、`Microsoft.Bot.Builder.Dialogs`、`Microsoft.Bot.Builder.FormFlow`、`Microsoft.Bot.Builder.FormFlow.Advanced`、`System.Collections.Generic`、`System.Linq` の名前空間が含まれます。 |

### <a name="additional-properties-at-the-root-of-the-schema-or-as-peers-of-the-type-property"></a>スキーマのルートに置かれる、または type プロパティのピアとなる追加プロパティ

| プロパティ | 値 |
|----|----|
| テンプレート | `{ TemplateUsage: { Patterns: [string, ...], <args> }, ...}` |
| Prompt | `{ Patterns:[string, ...] <args>}` |

JSON スキーマでテンプレートおよびプロンプトを指定するには、[TemplateAttribute][templateAttribute] と [PromptAttribute][promptAttribute] で定義されているのと同じボキャブラリを使用します。 スキーマのプロパティの名前と値は、基礎となる C# 列挙型のプロパティの名前と値と同じでなければなりません。 たとえば、次のスキーマ スニペットは、`TemplateUsage.NotUnderstood` テンプレートをオーバーライドし、`TemplateBaseAttribute.ChoiceStyle` を指定するテンプレートを定義します。 

```json
"Templates":{ "NotUnderstood": { "Patterns": ["I don't get it"], "ChoiceStyle":"Auto"}}
```

### <a name="additional-properties-as-peers-of-the-type-property"></a>type プロパティのピアとなる追加プロパティ

|   プロパティ   |          目次           |                                                   説明                                                    |
|--------------|-----------------------------|------------------------------------------------------------------------------------------------------------------|
|   Datetime   |            bool             |                                  フィールドが `DateTime` フィールドかどうかを示します。                                  |
|   Describe   |      文字列またはオブジェクト       |                  フィールドの説明 ([DescribeAttribute][describeAttribute] で説明)。                  |
|    Terms     |       `[string,...]`        |                  フィールド値を照合するための正規表現 (TermsAttribute で説明)。                  |
|  MaxPhrase   |             int             |                  `Language.GenerateTerms(string, int)` で用語を処理して、展開します。                   |
|    値    | `{ string: {Describe:string |                                  object, Terms:[string, ...], MaxPhrase}, ...}`                                  |
|    アクティブ    |           script            | フィールド、メッセージ、確認がアクティブかどうかをテストするための C# スクリプト (引数 `(JObject state)->bool` を使用)。  |
|   検証   |           script            |      フィールド値を検証するための C# スクリプト (引数 `(JObject state, object value)->ValidateResult` を使用)。      |
|    Define    |           script            |        フィールドを動的に定義するための C# スクリプト (引数 `(JObject state, Field<JObject> field)` を使用)。        |
|     次へ     |           script            | フィールドに入力した後の次のステップを判別するための C# スクリプト (引数 `(object value, JObject state)` を使用)。 |
|    実装する前    |          `[confirm          |                                                  message, ...]`                                                  |
|    実装した後     |          `[confirm          |                                                  message, ...]`                                                  |
| 依存関係 |        [string, ...]        |                           このフィールド、メッセージ、確認が依存するフィールド。                           |

**Before** プロパティか **After** プロパティの値の中で `{Confirm:script|[string, ...], ...templateArgs}` を使用すると、引数 `(JObject state)` を使用する C# スクリプトか、オプションのテンプレート引数でランダムに選択される一連のパターンのどちらかを使用して、確認を定義できます。

**Before** プロパティか **After** プロパティの値の中で `{Message:script|[string, ...] ...templateArgs}` を使用すると、引数 `(JObject state)` を使用する C# スクリプトか、オプションのテンプレート引数でランダムに選択される一連のパターンのどちらかを使用して、メッセージを定義できます。

## <a name="scripts"></a>スクリプト

前述のプロパティの中には、プロパティ値としてスクリプトが含まれるものもあります。 スクリプトは、メソッドの本体に普通に見出せるような、C# コードの任意のスニペットとすることができます。 **References** プロパティまたは **Imports** プロパティ (あるいはその両方) を使用して、参照を追加できます。 特別なグローバル変数には以下が含まれます。

| 可変 | 説明 |
|----|----|
| choice | スクリプトが実行する内部ディスパッチ。 |
| state | すべてのスクリプトにバインドされる `JObject` フォーム状態。 |
| ifield | メッセージ/確認プロンプト ビルダー以外のすべてのスクリプトで、現在のフィールドに関する論理的な判断ができるようにする `IField<JObject>`。 |
| value | **Validate** で検証するオブジェクト値。 |
| フィールド | **Define** でフィールドを動的に更新できるようにするための.`Field<JObject>`。 |
| context | **OnCompletion** で結果をポスティングできるようにするための `IDialogContext` コンテキスト。 |

JSON スキーマを使用して定義されているフィールドには、プログラムを使用して定義を拡張したりオーバーライドしたりする点で他のフィールドと同じ機能があります。 同じ方法でローカライズすることもできます。

## <a name="json-schema-example"></a>JSON スキーマの例

フォームを定義する最も簡単な方法は、C# コードを含め、すべてを JSON スキーマで直接定義することです。 次の例は、「[FormBuilder によるフォームのカスタマイズ](bot-builder-dotnet-formflow-formbuilder.md)」で説明されている注釈付きサンドウィッチ ボットの JSON スキーマを示しています。

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

## <a name="implement-formflow-with-json-schema"></a>JSON スキーマを使用した FormFlow の実装

JSON スキーマを使用した FormFlow を実装するには、`FormBuilderJson` を使用します。これは、`FormBuilder` と同じ fluent インターフェイスをサポートしています。 次のコード例は、「[FormBuilder によるフォームのカスタマイズ](bot-builder-dotnet-formflow-formbuilder.md)」で説明されている注釈付きサンドウィッチ ボットの JSON スキーマを実装する方法を示しています。

[!code-csharp[Use JSON schema](../includes/code/dotnet-formflow-json-schema.cs#useSchema)]

## <a name="sample-code"></a>サンプル コード

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>その他のリソース

- [FormFlow の基本的機能](bot-builder-dotnet-formflow.md)
- [FormFlow の高度な機能](bot-builder-dotnet-formflow-advanced.md)
- [FormBuilder によるフォームのカスタマイズ](bot-builder-dotnet-formflow-formbuilder.md)
- [フォーム コンテンツのローカライズ](bot-builder-dotnet-formflow-localize.md)
- [パターン言語によるユーザー エクスペリエンスのカスタマイズ](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET リファレンス</a>

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute
