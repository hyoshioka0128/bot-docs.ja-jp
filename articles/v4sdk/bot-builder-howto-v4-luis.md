---
title: ボットに自然言語の理解を追加する | Microsoft Docs
description: 自然言語理解のための LUIS を Bot Framework SDK と共に使用する方法について説明します。
keywords: Language Understanding, LUIS, 意図, 認識エンジン, エンティティ, ミドルウェア
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: cedd6e57ec0490a18b23f22c7895e4ee52dc8509
ms.sourcegitcommit: e9cd857ee11945ef0b98a1ffb4792494dfaeb126
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/01/2019
ms.locfileid: "71694534"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>ボットに自然言語の理解を追加する

[!INCLUDE[applies-to](../includes/applies-to.md)]
ユーザーの真意を会話と文脈から理解する能力は難しいタスクかもしれませんが、より自然な会話の印象をボットに与えることができます。 そうした能力は LUIS と呼ばれる Language Understanding によって実現され、ボットがユーザーのメッセージの意図を認識できるように、ユーザーがより自然な言葉を使用できるように、また、会話フローがより適切に管理されるようになります。 このトピックでは、LUIS を航空券予約アプリケーションに追加して、ユーザー入力に含まれるさまざまな意図やエンティティを認識する手順について説明します。 

## <a name="prerequisites"></a>前提条件
- [LUIS](https://www.luis.ai) アカウント
- この記事のコードは、**コア ボット** サンプルをベースにしています。 サンプルのコピー ( **[C#](https://aka.ms/cs-core-sample) または [JavaScript](https://aka.ms/js-core-sample)** ) が必要になります。 
- [ボットの基本](bot-builder-basics.md)、[自然言語処理](https://docs.microsoft.com/azure/cognitive-services/luis/what-is-luis)、および[ボット リソースの管理](bot-file-basics.md)に関する知識。

## <a name="about-this-sample"></a>このサンプルについて

このコア ボットのコード サンプルは、空港の航空券予約アプリケーションの例を示しています。 これは LUIS サービスを使用してユーザー入力を認識し、認識した最上位の LUIS の意図を返します。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
ユーザー入力の処理が完了するたびに、`DialogBot` では `UserState` と `ConversationState` の両方の現在の状態が保存されます。 必要な情報がすべて収集されると、コード サンプルによりデモの航空券予約が作成されます。 この記事では、このサンプルの LUIS 部分について説明します。 ただし、サンプルの一般的なフローは次のようになります。

- 新しいユーザーが接続され、ようこそカードを表示すると、`OnMembersAddedAsync` が呼び出されます。 
- ユーザー入力を受け取るたびに、`OnMessageActivityAsync` が呼び出されます。

![LUIS サンプル ロジック フロー](./media/how-to-luis/luis-logic-flow.png)

`OnMessageActivityAsync` モジュールは、`Run` ダイアログ拡張メソッドによって適切なダイアログを実行します。 そして、メイン ダイアログは LUIS ヘルパーを呼び出して、最もスコアが高いユーザーの意図を見つけます。 ユーザー入力の最上位の意図によって "BookFlight" が返されると、ヘルパーは LUIS によって返されたユーザー情報を入力します。 その後、メイン ダイアログは `BookingDialog` を開始し、これにより、次のような追加情報を必要に応じてユーザーから取得します。

- `Origin` 出発地
- `TravelDate` 航空券の予約日
- `Destination` 到着地

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
ユーザー入力の処理が完了するたびに、`dialogBot` では `userState` と `conversationState` の両方の現在の状態が保存されます。 必要な情報がすべて収集されると、コード サンプルによりデモの航空券予約が作成されます。 この記事では、このサンプルの LUIS 部分について説明します。 ただし、サンプルの一般的なフローは次のようになります。

- 新しいユーザーが接続され、ようこそカードを表示すると、`onMembersAdded` が呼び出されます。 
- ユーザー入力を受け取るたびに、`OnMessage` が呼び出されます。

![LUIS サンプルの javascript ロジック フロー](./media/how-to-luis/luis-logic-flow-js.png)

`onMessage` モジュールは、ユーザー入力を収集する `mainDialog` を実行します。
そして、メイン ダイアログは、LUIS ヘルパー `FlightBookingRecognizer` を呼び出して、最もスコアが高いユーザーの意図を見つけます。 ユーザー入力の最上位の意図によって "BookFlight" が返されると、ヘルパーは LUIS によって返されたユーザー情報を入力します。
この応答が返されると、`mainDialog` は LUIS から返されたユーザー情報を保持して、`bookingDialog` を開始します。 `bookingDialog` は、次のような追加情報を必要に応じてユーザーから取得します

- `destination` 到着地。
- `origin` 出発地。
- `travelDate` 航空券の予約日。

---

ダイアログ、状態などのサンプルの他の側面については、[ダイアログ プロンプトを使用したユーザー入力の収集](bot-builder-prompts.md)に関するページ、「[ユーザーおよび会話データを保存する](bot-builder-howto-v4-state.md)」を参照してください。

## <a name="create-a-luis-app-in-the-luis-portal"></a>LUIS ポータルでの LUIS アプリの作成
LUIS ポータルにサインインして、ご自身のバージョンのサンプル LUIS アプリを作成します。 アプリケーションは、 **[マイ アプリ]** で作成および管理できます。

1. **[Import new app]\(新しいアプリのインポート\)** を選択します。 
1. **[Choose App file (JSON format)...]\(アプリ ファイル (JSON 形式) を選択...\)** をクリックします。 
1. サンプルの `CognitiveModels` フォルダーにある `FlightBooking.json` ファイルを選択します。 **[オプション名]** に、「**FlightBooking**」と入力します。 このファイルには、次の 3 つの意図が含まれています:"Book Flight"、"Cancel"、および "None"。 これらの意図を使って、ユーザーがどのようなつもりでボットに送メッセージを信しているのかを把握します。
1. アプリを[トレーニング](https://docs.microsoft.com/azure/cognitive-services/LUIS/luis-how-to-train)します。
1. アプリを*運用*環境に[発行](https://docs.microsoft.com/azure/cognitive-services/LUIS/publishapp)します。

### <a name="why-use-entities"></a>エンティティを使用する理由
LUIS エンティティを使うと、標準の意図とは異なる物事やイベントを、お使いのボットがインテリジェントに解釈できるようになります。 これにより、ユーザーから追加情報を収集できるようになり、お使いのボットが、よりインテリジェントに応答したり、ユーザーにその追加情報をたずねる特定の質問をスキップしたりできます。 FlightBooking.json ファイルには、3 つの LUIS の意図である "Book Flight"、"Cancel"、および "None" の定義と共に、"From.Airport" や "To.Airport" などのエンティティのセットも含まれています。 これらのエンティティにより、ユーザーが新しい旅行の予約を要求したときに、LUIS は、そのユーザーの元の入力に含まれる追加情報を検出し、返すことができます。

エンティティ情報が LUIS の結果にどのように表示されるかについては、「[意図とエンティティが含まれる発話テキストからデータを抽出する](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-data-extraction)」を参照してください。

## <a name="obtain-values-to-connect-to-your-luis-app"></a>LUIS アプリに接続するための値を取得する
LUIS アプリには、その発行後、ボットからアクセスできるようになります。 ボット内から LUIS アプリにアクセスするためには、いくつかの値を記録する必要があります。 その情報は、LUIS ポータルを使用して取得できます。

### <a name="retrieve-application-information-from-the-luisai-portal"></a>LUIS.ai ポータルからアプリケーション情報を取得する
設定ファイル (`appsettings.json` または `.env`) は、すべてのサービス参照を 1 か所にまとめる場所として機能します。 取得した情報は、次のセクションでこのファイルに追加されます。 
1. 発行済みの LUIS アプリを [luis.ai](https://www.luis.ai) から選択します。
1. 発行済み LUIS アプリを開いた状態で **[管理]** タブを選択します。![LUIS アプリの管理](./media/how-to-luis/manage-luis-app.png)
1. 左側の **[アプリケーション情報]** タブで、 _[アプリケーション ID]_ に表示される値を <YOUR_APP_ID> として記録します。
1. 左側の **[Keys and Endpoints]\(キーとエンドポイント\)** タブを選択し、 _[オーサリング キー]_ に表示される値を <YOUR_AUTHORING_KEY> として記録します。
1. 下へスクロールしてページの最後まで移動し、"_リージョン_" に表示される値を <YOUR_REGION> として記録します。

### <a name="update-the-settings-file"></a>設定ファイルを更新する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

LUIS アプリにアクセスするために必要な情報 (アプリケーション ID、オーサリング キー、リージョンなど) を `appsettings.json` ファイルに追加します。 これらは、発行済みの LUIS アプリから先ほど保存した値です。 API ホスト名は `<your region>.api.cognitive.microsoft.com` 形式にする必要があることに注意してください。

**appsetting.json**  
[!code-json[appsettings](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/appsettings.json?range=1-7)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

LUIS アプリにアクセスするために必要な情報 (アプリケーション ID、オーサリング キー、リージョンなど) を `.env` ファイルに追加します。 これらは、発行済みの LUIS アプリから先ほど保存した値です。 API ホスト名は `<your region>.api.cognitive.microsoft.com` 形式にする必要があることに注意してください。

**.env**  
[!code[env](~/../BotBuilder-Samples/samples/javascript_nodejs/13.core-bot/.env?range=1-5)]

---

## <a name="configure-your-bot-to-use-your-luis-app"></a>LUIS アプリを使用するためのボットの構成

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

NuGet パッケージ **Microsoft.Bot.Builder.AI.Luis** がプロジェクトにインストールされていることを確認します。

LUIS サービスに接続するために、ボットは、上記で追加した情報を appsetting.json ファイルからプルします。 `FlightBookingRecognizer` クラスには、appsetting.json ファイルからのユーザーの設定に関するコードが含まれています。このクラスは、`RecognizeAsync` メソッドを呼び出すことで LUIS サービスにクエリを実行します。

**FlightBookingRecognizer.cs**  

[!code-csharp[luisHelper](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/FlightBookingRecognizer.cs?range=12-39)]

`FlightBookingEx.cs` には、*From*、*To*、および *TravelDate* を抽出するロジックが含まれています。これは、`MainDialog.cs` から `FlightBookingRecognizer.RecognizeAsync<FlightBooking>` を呼び出したときに LUIS の結果を格納するために使用される部分クラス `FlightBooking.cs` を拡張します。

**CognitiveModels\FlightBookingEx.cs**  

[!code-csharp[luis helper](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/CognitiveModels/FlightBookingEx.cs?range=8-35)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

LUIS を使用するには、ご自身のプロジェクトでは、**botbuilder ai** npm パッケージをインストールする必要があります。

LUIS サービスに接続するために、ボットは、上記で追加した `.env` ファイルの情報を使用します。 `flightBookingRecognizer.js` クラスには、`.env` ファイルからユーザーの設定をインポートするコードが含まれています。また、`recognize()` メソッドを呼び出すことで LUIS サービスにクエリを実行します。

**dialogs/flightBookingRecognizer.js**

[!code-javascript[luis helper](~/../BotBuilder-Samples/samples/javascript_nodejs/13.core-bot/dialogs/flightBookingRecognizer.js?range=6-64)]

From、To、および TravelDate を抽出するロジックは、`flightBookingRecognizer.js` 内のヘルパー メソッドとして実装されています。 これらのメソッドは、`mainDialog.js` から `flightBookingRecognizer.executeLuisQuery()` を呼び出した後に使用されます

---

お客様のボットで使用するための LUIS の構成と接続が完了しました。

## <a name="test-the-bot"></a>ボットのテスト

最新の [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をダウンロードしてインストールします

1. ご自身のマシンを使ってローカルでサンプルを実行します。 手順については、readme ファイルで [C# サンプル](https://aka.ms/cs-core-sample)または [JS サンプル](https://aka.ms/js-core-sample)のいずれかを参照してください。

1. エミュレーターで、"パリの旅行する"、"パリからベルリンに移動する" などのメッセージを入力します。 FlightBooking.json ファイルの任意の発話を、意図 "Book flight" のトレーニングに使用します。

![LUIS の予約の入力](./media/how-to-luis/luis-user-travel-input.png)

LUIS から返された最上位の意図が "Book flight" に解決されると、お使いのボットは、旅行の予約を作成するための情報が十分に確保されるまで質問を続けます。 予約を作成できたら、その予約情報をユーザーに返します。 

![LUIS の予約の結果](./media/how-to-luis/luis-travel-result.png)

この時点で、コードのボット ロジックはリセットされ、引き続き追加の予約を作成できます。 

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [QnA Maker を使用して質問に回答する](./bot-builder-howto-qna.md)
