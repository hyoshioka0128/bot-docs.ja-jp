---
title: フォームの内容をローカライズする - Bot Service
description: FormFlow と Bot Framework SDK for .NET を使用してフォームの内容をローカライズする方法を説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/02/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: accffd89fa6bba8e8bb43aad573cf8df4b3723b0
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75796847"
---
# <a name="localize-form-content"></a>フォームの内容をローカライズする

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

フォームのローカライズ言語は、現在のスレッドの [CurrentUICulture](https://msdn.microsoft.com/library/system.threading.thread.currentuiculture(v=vs.110).aspx) と [CurrentCulture](https://msdn.microsoft.com/library/system.threading.thread.currentculture(v=vs.110).aspx) によって決まります。
既定では、カルチャは現在のメッセージの **Locale** フィールドから取得されますが、その既定動作をオーバーライドできます。
ボットの作成方法に応じて、最大 3 つの異なるソースからローカライズされた情報を取得できます。

- **PromptDialog** および **FormFlow** に組み込まれているローカライズ
- フォームの静的な文字列に対してユーザーが生成するリソース ファイル
- 動的計算フィールド、メッセージ、または確認の文字列を使用してユーザーが作成するリソース ファイル

## <a name="generate-a-resource-file-for-the-static-strings-in-your-form"></a>フォーム内の静的な文字列のリソース ファイルを生成する

フォームの静的な文字列には、フォームが C# クラスの情報から生成する文字列と、ユーザーがプロンプト、テンプレート、メッセージ、または確認として指定する文字列が含まれます。
組み込まれているテンプレートから生成される文字列は、既にローカライズされているため、静的な文字列とは見なされません。
フォーム内の文字列の多くは自動的に生成されるので、通常の C# リソース文字列を直接使用することはできません。
代わりに、`IFormBuilder.SaveResources` を呼び出すか、または BotBuilder SDK for .NET に含まれる **RView** ツールを使用して、フォーム内の静的文字列のリソース ファイルを生成できます。

### <a name="use-iformbuildersaveresources"></a>IFormBuilder.SaveResources を使用する

フォームで [IFormBuilder.SaveResources][saveResources] を呼び出して文字列を .resx ファイルに保存することにより、リソース ファイルを生成できます。

### <a name="use-rview"></a>RView を使用する

もう 1 つの方法として、BotBuilder SDK for .NET に含まれる <a href="https://aka.ms/v3-cs-RView-library" target="_blank">RView</a> ツールを使用することで、.dll または .exe に基づくリソース ファイルを生成できます。
.resx ファイルを生成するには、**rview** を実行して、静的フォーム構築メソッドを含むアセンブリとそのメソッドへのパスを指定します。
次のスニペットでは、**RView** を使用して `Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.resx` リソース ファイルを生成する方法を示します。

```csharp
rview -g Microsoft.Bot.Sample.AnnotatedSandwichBot.dll Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.BuildForm
```

次の抜粋では、この **rview** コマンドを実行することによって生成される .resx ファイルの部分を示します。

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

## <a name="configure-your-project"></a>プロジェクトを構成する

リソース ファイルを生成した後、それをプロジェクトに追加し、以下の手順に従ってニュートラル言語を設定します。 

1. プロジェクトを右クリックし、 **[アプリケーション]** を選択します。
2. **[アセンブリ情報]** をクリックします。
3. ボットを開発した言語に対応する **[ニュートラル言語]** の値を選択します。

フォームが作成されるとき、[IFormBuilder.Build][build] メソッドはフォームの種類の名前を含むリソースを自動的に検索し、それらを使ってフォームの静的な文字列をローカライズします。 

> [!NOTE]
> [Advanced.Field.SetDefine][setDefine] を使って定義されている動的計算フィールドの場合は ([動的フィールドの使用](bot-builder-dotnet-formflow-formbuilder.md#dynamically-define-field-values-confirmations-and-messages)に関するページの説明を参照)、動的計算フィールドの文字列はフォームの設定時に作成されるため、静的フィールドと同じ方法ではローカライズできません。 ただし、C# の通常のローカライズ メカニズムを使って、動的計算フィールドをローカライズできます。

### <a name="localize-resource-files"></a>リソース ファイルをローカライズする 

リソース ファイルをプロジェクトに追加した後、<a href="https://developer.microsoft.com/windows/develop/multilingual-app-toolkit" target="_blank">多言語アプリ ツールキット (MAT)</a> を使用してリソース ファイルをローカライズできます。 **MAT** をインストールした後、以下の手順に従ってプロジェクトで MAT を有効にします。

1. Visual Studio のソリューション エクスプローラーでプロジェクトを選択します。
2. **[ツール]** 、 **[Multilingual App Toolkit]\(多言語アプリ ツールキット\)** 、 **[有効にする]** の順にクリックします。
3. プロジェクトを右クリックし、 **[Multilingual App Toolkit]\(多言語アプリ ツールキット\)** 、 **[Add Translations]\(翻訳の追加\)** の順に選択して、翻訳を選択します。 これにより、自動または手動で翻訳できる業界標準の <a href="https://en.wikipedia.org/wiki/XLIFF" target="_blank">XLF</a> ファイルが作成されます。

> [!NOTE]
> この記事では多言語アプリ ツールキットを使って内容をローカライズする方法を説明しましたが、他のさまざまな手段を使ってローカライズを実装することができます。

## <a name="see-it-in-action"></a>実際の動作を確認する

このコード例では、「[FormBuilder によるフォームのカスタマイズ](bot-builder-dotnet-formflow-formbuilder.md)」の例を基にして前で説明したようにローカライズを実装します。 この例では、`DynamicSandwich` クラス (ここでは示されていません) に、動的計算フィールド、メッセージ、確認のローカライズ情報が含まれています。

[!code-csharp[Build localized form](../includes/code/dotnet-formflow-localize.cs#buildLocalizedForm)]

このスニペットでは、`CurrentUICulture` が **French** の場合のボットとユーザー間の対話の結果を示しています。

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

## <a name="sample-code"></a>サンプル コード

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>その他のリソース

- [FormFlow の基本的機能](bot-builder-dotnet-formflow.md)
- [FormFlow の高度な機能](bot-builder-dotnet-formflow-advanced.md)
- [FormBuilder によるフォームのカスタマイズ](bot-builder-dotnet-formflow-formbuilder.md)
- [JSON スキーマによるフォームの定義](bot-builder-dotnet-formflow-json-schema.md)
- [パターン言語によるユーザー エクスペリエンスのカスタマイズ](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET リファレンス</a>

[build]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1.build 

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[saveResources]: /dotnet/api/microsoft.bot.builder.formflow.iform-1.saveresources
