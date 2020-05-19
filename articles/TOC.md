# [Azure Bot Service のドキュメント](index.yml)
# 概要
## [Azure Bot Service について](bot-service-overview-introduction.md)
## [新機能](what-is-new.md)
# クイック スタート
## [.NET を使用してボットを作成する](dotnet/bot-builder-dotnet-sdk-quickstart.md)
## [JavaScript を使用してボットを作成する](javascript/bot-builder-javascript-quickstart.md)
## [Python を使用してボットを作成する](python/bot-builder-python-quickstart.md)
## [Azure Bot Service を使用してボットを作成する](v4sdk/abs-quickstart.md)
# チュートリアル
## [1.基本的なボットを作成してデプロイする](v4sdk/bot-builder-tutorial-basic-deploy.md)
## [2.QnA Maker を追加して、ボットを再デプロイする](v4sdk/bot-builder-tutorial-add-qna.md)
## [ボットへの認証の追加](bot-builder-tutorial-authentication.md)
# サンプル
## [GitHub の Bot Framework サンプル リポジトリ](https://github.com/Microsoft/BotBuilder-Samples/blob/master/README.md)
# 概念
## [ボットのしくみ](v4sdk/bot-builder-basics.md)
## [状態の管理](v4sdk/bot-builder-concept-state.md)
## [ダイアログ ライブラリ](v4sdk/bot-builder-concept-dialog.md)
## [ミドルウェア](v4sdk/bot-builder-concept-middleware.md)
## アダプティブ ダイアログ
### [アダプティブ ダイアログの概要](v4sdk/bot-builder-adaptive-dialog-Introduction.md)
### [アダプティブ ダイアログのイベントとトリガー](v4sdk/bot-builder-concept-adaptive-dialog-Triggers.md)
### [アダプティブ ダイアログのアクション](v4sdk/bot-builder-concept-adaptive-dialog-Actions.md)
### [アダプティブ ダイアログを使用したユーザー入力の要求](v4sdk/bot-builder-concept-adaptive-dialog-Inputs.md)
### [アダプティブ ダイアログの認識エンジン](v4sdk/bot-builder-concept-adaptive-dialog-recognizers.md)
### [アダプティブ ダイアログでの言語の生成](v4sdk/bot-builder-concept-adaptive-dialog-generators.md)
### [アダプティブ ダイアログでのメモリ スコープと状態の管理](v4sdk/bot-builder-concept-adaptive-dialog-memory-states.md)
## 認証
### [ボット認証](v4sdk/bot-builder-concept-authentication.md)
### [ID プロバイダー](v4sdk/bot-builder-concept-identity-providers.md)
### [シングル サインオン](v4sdk/bot-builder-concept-sso.md)
## [ボット リソースの管理](v4sdk/bot-file-basics.md)
## [Microsoft Teams のボットのしくみ](v4sdk/bot-builder-basics-teams.md)
## スキル
### [スキルについて](v4sdk/skills-conceptual.md)
### [スキル ボットについて](v4sdk/skills-about-skill-bots.md)
### [スキル コンシューマーについて](v4sdk/skills-about-skill-consumers.md)
## [アダプティブ式](v4sdk/bot-builder-concept-adaptive-expressions.md)
## [言語の生成](v4sdk/bot-builder-concept-language-generation.md)
<!-- [Language understanding](v4sdk/bot-builder-concept-luis.md) -->
## [Bot Service テンプレート](bot-service-concept-templates.md)
## [Cognitive Services](bot-service-concept-intelligence.md)
## [ボットの主要シナリオ](bot-service-scenario-overview.md)
### [コマース ボット](bot-service-scenario-commerce.md)
### [Cortana の Skill ボット](bot-service-scenario-cortana-skill.md)
### [Enterprise Productivity ボット](bot-service-scenario-enterprise-productivity.md)
### [Information ボット](bot-service-scenario-informational.md)
### [モノのインターネット ボット](bot-service-scenario-internet-things.md)
# 操作方法
## [Design (デザイン)](design/TOC.md)
## 開発
<!-- ## [Best practice for welcoming the user](v4sdk/bot-builder-welcome-user.md) -->
### [テキスト メッセージを送受信する](v4sdk/bot-builder-howto-send-messages.md)
### [メッセージにメディアを追加する](v4sdk/bot-builder-howto-add-media-attachments.md)
### [ユーザー アクションをガイドするボタンを追加する](v4sdk/bot-builder-howto-add-suggested-actions.md)
### [ユーザーと会話データを保存する](v4sdk/bot-builder-howto-v4-state.md)
### [ユーザーに入力を求める](v4sdk/bot-builder-primitive-prompts.md)
### [ユーザーへのウェルカム メッセージの送信](v4sdk/bot-builder-send-welcome-message.md)
### [ユーザーへのプロアクティブな通知の送信](v4sdk/bot-builder-howto-proactive-message.md)
### [連続して行われる会話フローの実装](v4sdk/bot-builder-dialog-manage-conversation-flow.md)
### [ボットに自然言語の理解を追加する](v4sdk/bot-builder-howto-v4-luis.md)
### [QnA Maker を使用してユーザーの質問に回答する](v4sdk/bot-builder-howto-qna.md)
### [複数の LUIS と QnA モデルを使用する](v4sdk/bot-builder-tutorial-dispatch.md)
### [ダイアログの複雑さを管理する](v4sdk/bot-builder-compositcontrol.md)
### [ブランチとループを使用して高度な会話フローを作成する](v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)
<!--#### [Implement a greeting dialog](v4sdk/bot-builder-dialogs-greeting.md)--TODO: Add once there's a sample.-->
### [ユーザーによる割り込みの処理](v4sdk/bot-builder-howto-handle-user-interrupt.md)
### アダプティブ ダイアログ
#### [アダプティブ ダイアログを使用してボットを作成する](v4sdk\bot-builder-dialogs-adaptive.md)
#### [アダプティブ、コンポーネント、ウォーターフォール、およびカスタム ダイアログを使用してボットを作成する](v4sdk/bot-builder-mixed-dialogs.md)
### [ストレージに直接書き込む](v4sdk/bot-builder-howto-v4-storage.md)
### 認証
#### [ボットに認証を追加する](v4sdk/bot-builder-authentication.md)
#### [ボットにシングル サインオンを追加する](v4sdk/bot-builder-authentication-sso.md)
### [ボットのカスタム ストレージの実装](v4sdk/bot-builder-custom-storage.md)
### 言語の生成
#### [ボットでの言語生成テンプレートの使用](language-generation/bot-builder-howto-use-lg-templates.md)
### スキル
#### [スキルを実装する](v4sdk/skill-implement-skill.md)
#### [v2.1 スキル マニフェストを作成する](v4sdk/skills-write-manifest-2-1.md)
#### [v2.0 スキル マニフェストを作成する](v4sdk/skills-write-manifest-2-0.md)
#### [スキル内でダイアログを使用する](v4sdk/skill-actions-in-dialogs.md)
#### [スキル コンシューマーを実装する](v4sdk/skill-implement-consumer.md)
#### [ダイアログを使用してスキルを使用する](v4sdk/skill-use-skilldialog.md)
<!--
#### [Use middleware in a skill consumer](v4sdk/skill-middleware-in-consumer.md)
#### [Add claims validation](v4sdk/skill-add-claims-validation.md)
-->
### [ボットへのテレメトリの追加](v4sdk/bot-builder-telemetry.md)
### [QnA ボットへのテレメトリの追加](v4sdk/bot-builder-telemetry-QnAMaker.md)
### [ボットのテレメトリ データを分析する](v4sdk/bot-builder-telemetry-analytics-queries.md)
### [ボットで Direct Line Speech を使用する](directline-speech-bot.md)
### [.NET](dotnet/TOC.md)
### [Node.js](nodejs/TOC.md)
## テスト
### [ボットの単体テスト](v4sdk/unit-test-bots.md)
### [ボットへのトレース アクティビティの追加](v4sdk/using-trace-activities.md)
## [Debug](debug/TOC.md)
## 配置
### [ボットを Azure にデプロイする](bot-builder-deploy-az-cli.md)
### [継続的デプロイを設定する](bot-service-build-continuous-deployment.md)
## [管理](manage/TOC.md)
## [移行](v4sdk/migration/TOC.md)
# リファレンス
## [.NET SDK v4](https://aka.ms/botframework-v4-cs-sdk)
## [JavaScript SDK v4](https://aka.ms/bot-jssdk-v4)
## [Python SDK v4](https://aka.ms/botframework-v4-python-sdk)
## [REST](rest-api/TOC.md)
## [.NET SDK v3](https://aka.ms/botframework-v3-cs-sdk)
## [Node.js SDK v3](https://aka.ms/bot-jssdk-v3)
## BF CLI ツールのリファレンス
### [BF CLI の概要](v4sdk/bf-cli-overview.md)
### [BF CLI のリファレンス](v4sdk/bf-cli-reference.md)
## アダプティブ式
### [構築済みの関数](adaptive-expressions/adaptive-expressions-prebuilt-functions.md)
### [API リファレンス](adaptive-expressions/adaptive-expressions-API-reference.md)
## LG
### [API リファレンス](language-generation/language-generation-API-reference.md)
### [.lg ファイル形式](file-format/bot-builder-lg-file-format.md)
### [構造化された応答テンプレート](language-generation/language-generation-structured-response-template.md)
### [LG から挿入した関数](language-generation/functions-injected-from-language-generation.md)
## ファイル形式
### [.lu ファイル形式](file-format/bot-builder-lu-file-format.md)
### [.qna ファイル形式](file-format/bot-builder-qna-file-format.md)
## [エンティティとアクティビティの種類](bot-service-activities-entities.md)
# [リソース](resources/TOC.md)
