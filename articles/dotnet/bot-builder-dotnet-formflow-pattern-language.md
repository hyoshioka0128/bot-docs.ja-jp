---
title: パターン言語によるユーザー エクスペリエンスのカスタマイズ | Microsoft Docs
description: パターン言語と Bot Builder SDK for .NET を使用した FormFlow プロンプトのカスタマイズ方法や FormFlow テンプレートのオーバーライド方法を学習します。
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
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905584"
---
# <a name="customize-user-experience-with-pattern-language"></a>パターン言語によるユーザー エクスペリエンスのカスタマイズ

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

プロンプトをカスタマイズまたは既定のテンプレートをオーバーライドする場合、パターン言語を使用してプロンプトの内容や書式を指定することができます。 

## <a name="prompts-and-templates"></a>プロンプトとテンプレート

[プロンプト][promptAttribute]は、情報を要求したり、確認を求めたりするためにユーザーに送信するメッセージを定義します。 プロンプトのカスタマイズは、[Prompt 属性](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-prompt-attribute)を使用して行うか、[IFormBuilder<T>.Field][field] によって暗黙的に行います。 

フォームは、テンプレートを使用してプロンプトやヘルプなどのその他の要素を自動的に作成します。 [Template 属性](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-template-attribute)を使用すると、クラスまたはフィールドの既定のテンプレートをオーバーライドすることができます。 

> [!TIP]
> [FormConfiguration.Templates][formConfiguration] クラスでは、パターン言語の使用方法の良い例となる一連の組み込みテンプレートが定義されています。

## <a name="elements-of-pattern-language"></a>パターン言語の要素

パターン言語では、実行時に実際の値に置き換えられる要素を中かっこ (`{}`) を使用して識別します。 次の表は、パターン言語の要素をまとめたものです。

| 要素 | 説明 |
|----|----|
| `{<format>}` | 現在のフィールド (属性が適用されるフィールド) の値を表示します。 |
| `{&}` | 現在のフィールドの説明 (指定されてない場合はフィールドの名前) を表示します。 |
| `{<field><format>}` | 名前付きフィールドの値を表示します。 | 
| `{&<field>}` | 名前付きフィールドの説明を表示します。 |
| <code>{&#124;&#124;}</code> | 現在の選択肢を表示します。これは、フィールドの現在の値、"no preference"、列挙型の値のいずれかです。 |
| `{[{<field><format>} ...]}` | 名前付きフィールドの値の一覧を表示します。リスト内の個々の値は、[Separator][separator] および [LastSeparator][lastSeparator] を使用して分割されます。 |
| `{*}` | アクティブなフィールドそれぞれについて 1 行を表示します。各行には、フィールドの説明と現在の値が含まれます。 | 
| `{*filled}` | 実際に値を含むアクティブなフィールドそれぞれについて 1 行を表示します。各行には、フィールドの説明と現在の値が含まれます。 |
| `{<nth><format>}` | テンプレートの n 番目の引数に適用される通常の C# の書式指定子です。 利用可能な引数の一覧は、「[TemplateUsage][templateUsage]」を参照してください。 |
| `{?<textOrPatternElement>...}` | 条件付き置換です。 参照されるすべてのパターン要素に値がある場合、値が置換されて式全体が使用されます。 |

上記の要素について

- `<field>` プレース ホルダーは、フォーム クラス内のフィールドの名前です。 たとえば、クラスに `Size` という名前のフィールドが含まれている場合、パターン要素として `{Size}` を指定することができます。 

- パターン要素内の省略記号 (`"..."`) は、要素に複数の値が存在してもよいことを示しています。

- `<format>` プレース ホルダーは、C# の書式指定子です。 たとえば、クラスに `Rating` という名前の `double` 型のフィールドが含まれている場合、有効桁数 2 桁の数字を表示するパターン要素として `{Rating:F2}` を指定することができます。

- `<nth>` プレース ホルダーは、テンプレートの n 番目の引数を参照します。

### <a name="pattern-language-within-a-prompt-attribute"></a>Prompt 属性内のパターン言語

この例では、`Sandwich` フィールドの説明を表示する `{&}` 要素と、そのフィールドの選択肢の一覧を表示する `{||}` 要素を使用しています。

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns1)]

結果のプロンプトは次のようになります。

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

## <a name="formatting-parameters"></a>書式パラメーター

プロンプトとテンプレートは、以下の書式パラメーターをサポートします。

| 使用法 | 説明 |
|----|----|
| `AllowDefault` | <code>{&#124;&#124;}</code> パターン要素に適用されます。 フォームがフィールドの現在の値を使用可能な選択肢として表示するかどうかを決定します。 `true` の場合、現在の値が使用可能な値として表示されます。 既定では、 `true`です。 |
| `ChoiceCase` | <code>{&#124;&#124;}</code> パターン要素に適用されます。 各選択肢のテキストを正規化 (例: 各単語の最初の文字を大文字にするかどうか) するかどうかを決定します。 既定では、 `CaseNormalization.None`です。 使用できる値については、「[CaseNormalization][caseNormalization]」をご覧ください。 |
| `ChoiceFormat` | <code>{&#124;&#124;}</code> パターン要素に適用されます。 選択肢の一覧を番号付きリストと箇条書きリストのどちらで表示するかを決定します。 番号付きリストの場合、`ChoiceFormat` を `{0}` (既定値) に設定します。 箇条書きリストの場合、`ChoiceFormat` を `{1}` に設定します。 |
| `ChoiceLastSeparator` | <code>{&#124;&#124;}</code> パターン要素に適用されます。 選択肢のインライン一覧で、最後の選択肢の前に区切り記号を含めるかどうかを決定します。 |
| `ChoiceParens` | <code>{&#124;&#124;}</code> パターン要素に適用されます。 選択肢のインライン一覧をかっこ内に表示するかどうかを決定します。 `true` の場合、選択肢の一覧はかっこ内に表示されます。 既定では、 `true`です。 |
| `ChoiceSeparator` | <code>{&#124;&#124;}</code> パターン要素に適用されます。 選択肢のインライン一覧で、最後の選択肢を除くすべての選択肢の前に区切り記号を含めるかどうかを決定します。 | 
| `ChoiceStyle` | <code>{&#124;&#124;}</code> パターン要素に適用されます。 選択肢の一覧をインラインで表示するか、1 行ごとに表示するかを決定します。 既定では、`ChoiceStyleOptions.Auto` です。この場合、選択肢をインライン表示するかリスト表示するかは、実行時に決定されます。 使用できる値については、「[ChoiceStyleOptions][choiceStyleOptions]」をご覧ください。 |
| `Feedback` | プロンプトのみに適用されます。 フォームがユーザーの選択を理解したことを示すために、選択内容をエコーするかどうかを決定します。 既定では、`FeedbackOptions.Auto` です。この場合、ユーザーの入力の一部が理解できない場合のみエコーを返します。 使用できる値については、「[FeedbackOptions][feedbackOptions]」をご覧ください。 |
| `FieldCase` | フィールドの説明のテキストを正規化 (例: 各単語の最初の文字を大文字にするかどうか) するかどうかを決定します。 既定では、`CaseNormalization.Lower` です。この場合、説明を小文字に変換します。 使用できる値については、「[CaseNormalization][caseNormalization]」をご覧ください。 |
| `LastSeparator` | `{[]}` パターン要素に適用されます。 項目の配列で、最後の項目の前に区切り記号を含めるかどうかを決定します。 |
| `Separator` | `{[]}` パターン要素に適用されます。 項目の配列で、最後の項目を除くすべての項目の前に区切り記号を含めるかどうかを決定します。 |
| `ValueCase` | フィールドの値のテキストを正規化 (例: 各単語の最初の文字を大文字にするかどうか) するかどうかを決定します。 既定では、`CaseNormalization.InitialUpper` です。この場合、各単語の最初の文字を大文字に変換します。 使用できる値については、「[CaseNormalization][caseNormalization]」をご覧ください。 |

### <a name="prompt-attribute-with-formatting-parameter"></a>書式パラメーターを含む Prompt 属性 

次の例では、`ChoiceFormat` パラメーターを使用して選択肢の一覧が、箇条書きリストとして表示されるように指定しています。

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns2)]

結果のプロンプトは次のようになります。

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

## <a name="sample-code"></a>サンプル コード

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>その他のリソース

- [FormFlow の基本的機能](bot-builder-dotnet-formflow.md)
- [FormFlow の高度な機能](bot-builder-dotnet-formflow-advanced.md)
- [FormBuilder によるフォームのカスタマイズ](bot-builder-dotnet-formflow-formbuilder.md)
- [フォーム コンテンツのローカライズ](bot-builder-dotnet-formflow-localize.md)
- [JSON スキーマによるフォームの定義](bot-builder-dotnet-formflow-json-schema.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET リファレンス</a>

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[field]: /dotnet/api/microsoft.bot.builder.formflow.iformbuilder-1.field

[formConfiguration]: /dotnet/api/microsoft.bot.builder.formflow.formconfiguration

[separator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.separator

[lastSeparator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.lastseparator

[templateUsage]: /dotnet/api/microsoft.bot.builder.formflow.templateusage

[caseNormalization]: /dotnet/api/microsoft.bot.builder.formflow.casenormalization

[choiceStyleOptions]: /dotnet/api/microsoft.bot.builder.formflow.choicestyleoptions

[feedbackOptions]: /dotnet/api/microsoft.bot.builder.formflow.feedbackoptions
