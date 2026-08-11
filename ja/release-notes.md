<!-- pre-align:aligned sig=79f355249cbd -->

<a id="storage-object-storage-release-notes"></a>
## Storage > Object Storage > release notes { #storage-object-storage-release-notes }

<a id="august-25-2026"></a>
## 2026. 08. 25. { #august-25-2026 }
<a id="august-25-2026-added-features"></a>
### 新規機能追加 { #august-25-2026-added-features }
* [コンソール] 作業履歴の照会、作業中断機能の追加
    * コンテナを空にする、オブジェクトのコピー/移動/削除の作業履歴を照会し、作業を中断する機能を追加しました。
* [API] コンテナポリシードキュメントを使用したコンテナ設定機能の追加
    * アクセスポリシー、CORS、オブジェクトロックコンテナの設定をサポートします。

<a id="august-25-2026-feature-updates"></a>
### 機能改善/変更 { #august-25-2026-feature-updates }
* [API] Amazon S3 API互換性の改善
    * ロックコンテナの作成および設定をサポートします。

<a id="may-27-2026"></a>
## 2026. 05. 27. { #may-27-2026 }
<a id="may-27-2026-added-features"></a>
### 新規機能追加 { #may-27-2026-added-features }
* [コンソール] ライフサイクルルールの設定および一括適用機能の追加
* [API] コンテナポリシードキュメントを使用したコンテナ設定機能の追加
    * ライフサイクルルールの設定をサポートします。

<a id="may-27-2026-feature-updates"></a>
### 機能改善/変更 { #may-27-2026-feature-updates }
* [API] Amazon S3 API互換性の改善
    * ドメインスタイル(Virtual Hosted Domain)エンドポイントをサポートします。
    * Trailing Checksumを使用したアップロード整合性検証をサポートします。
    * CORS Preflightリクエストをサポートします。

<a id="july-29-2025"></a>
## 2025. 07. 29. { #july-29-2025 }
<a id="july-29-2025-feature-updates"></a>
### 機能改善/変更 { #july-29-2025-feature-updates }
* [API] 書き込みリクエスト速度制限(rate limit)ポリシーの追加
    * ストレージアカウント単位で、1秒あたり500件を超えるリクエストに対して制限ポリシーを適用します。

<a id="may-27-2025"></a>
## 2025. 05. 27. { #may-27-2025 }
<a id="may-27-2025-added-features"></a>
### 新規機能追加 { #may-27-2025-added-features }
* [コンソール] コンテナ複製の一時停止/再開機能を追加

<a id="may-27-2025-feature-updates"></a>
### 機能改善/変更 { #may-27-2025-feature-updates }
* [API] Amazon S3 API互換性の改善
    * オブジェクトリストで各オブジェクトのLastModified値がミリ秒単位まで表示される問題を修正しました。

<a id="may-27-2025-bug-fixes"></a>
### バグ修正 { #may-27-2025-bug-fixes }
* [API] Amazon S3互換APIでオブジェクトロックが設定されたバケットにマルチパートアップロードすると、リクエストが失敗し、既存パートオブジェクトが削除されるバグを修正しました。

<a id="august-27-2024"></a>
## 2024. 08. 27. { #august-27-2024 }
<a id="august-27-2024-feature-updates"></a>
### 機能改善/変更 { #august-27-2024-feature-updates }
* [コンソール][API] サービスゲートウェイIPアクセス制御設定を追加
    * サービスゲートウェイを使用したリクエストに対するIP ACL例外処理が可能です。
* [コンソール] 同じ組織の他のプロジェクトへの複製機能追加
* [API] Amazon S3 API互換性の改善
    * 以下のエラー状況のレスポンス互換性を改善しました。
        * 無効な名前のバケット作成リクエスト
        * 無効なパスのコンテナリクエスト
        * 無効なパスのオブジェクトリクエスト

<a id="may-28-2024"></a>
## 2024. 05. 28. { #may-28-2024 }
<a id="may-28-2024-added-features"></a>
### 新規機能追加 { #may-28-2024-added-features }
* [コンソール][API] Economyストレージクラスの追加 - 韓国(板橋)リージョン
    * データへのアクセス頻度、コスト要件に応じてストレージクラスを選択できます。
* [コンソール][API] ライフサイクル満了動作の追加
    * ライフサイクルが満了したオブジェクトを他のコンテナに移動できます。

<a id="may-28-2024-feature-updates"></a>
### 機能改善/変更 { #may-28-2024-feature-updates }
* [コンソール] オブジェクトコピー機能の改善
    * 複数のオブジェクトを選択してコピー/移動できます。
* [コンソール] コンテナ名のルール変更
    * 最小3文字から最大63文字まで許可されます。
    * 英字小文字、数字、-、. を使用できます。
    * IP形式の名前は使用できません。

<a id="february-27-2024"></a>
## 2024. 02. 27. { #february-27-2024 }
<a id="february-27-2024-added-features"></a>
### 新規機能追加 { #february-27-2024-added-features }
* [コンソール][API] オブジェクトアップロードポリシー設定機能を追加
* [コンソール] コンテナを空にする機能を追加

<a id="february-27-2024-feature-updates"></a>
### 機能改善/変更 { #february-27-2024-feature-updates }
* [コンソール] オブジェクト削除機能の改善

<a id="november-28-2023"></a>
## 2023. 11. 28. { #november-28-2023 }
<a id="november-28-2023-added-features"></a>
### 新規機能追加 { #november-28-2023-added-features }
* [API] Amazon S3互換API、AWS Signature V4 Chunked Uploadをサポート
* [コンソール] 署名付きURLの作成機能を追加

<a id="may-30-2023"></a>
## 2023. 05. 30. { #may-30-2023 }
<a id="may-30-2023-added-features"></a>
### 新規機能追加 { #may-30-2023-added-features }
* [コンソール][API] IP ACL機能を追加

<a id="march-28-2023"></a>
## 2023. 03. 28. { #march-28-2023 }
<a id="march-28-2023-feature-updates"></a>
### 機能改善/変更 { #march-28-2023-feature-updates }
* [API] APIエンドポイントの変更

<a id="march-28-2023-bug-fixes"></a>
### バグ修正 { #march-28-2023-bug-fixes }
* [API] パブリックコンテナにアップロードされたマルチパートオブジェクトのセグメントオブジェクトが他のコンテナにある場合、トークンなしでダウンロードできない問題を修正しました。
* [API] Amazon S3互換APIを利用して同じ名前のマルチパートオブジェクトを更新した時に、以前のパートオブジェクトが削除されない問題を修正しました。

<a id="november-29-2022"></a>
## 2022. 11. 29. { #november-29-2022 }
<a id="november-29-2022-added-features"></a>
### 新規機能追加 { #november-29-2022-added-features }
* [コンソール][API] オブジェクトロックコンテナ設定機能を追加

<a id="october-13-2022"></a>
## 2022. 10. 13. { #october-13-2022 }
<a id="october-13-2022-added-features"></a>
### 新規機能追加 { #october-13-2022-added-features }
* [コンソール] コンテナACL設定機能を追加

<a id="september-27-2022"></a>
## 2022. 09. 27. { #september-27-2022 }
<a id="september-27-2022-added-features"></a>
### 新規機能追加 { #september-27-2022-added-features }
* [コンソール] 暗号化コンテナ設定機能を追加
* [コンソール] オリジン間リソース共有(CORS)設定機能を追加

<a id="september-27-2022-feature-updates"></a>
### 機能改善/変更 { #september-27-2022-feature-updates }
* [コンソール] コンテナ情報照会、設定UIを改善

<a id="august-23-2022"></a>
## 2022. 08. 23. { #august-23-2022 }
<a id="august-23-2022-added-features"></a>
### 新規機能追加 { #august-23-2022-added-features }
* [API] RFCを遵守するETag形式の使用設定を追加

<a id="march-29-2022"></a>
## 2022. 03. 29. { #march-29-2022 }
<a id="march-29-2022-added-features"></a>
### 新規機能追加 { #march-29-2022-added-features }
* [コンソール] リージョン間のコンテナ複製機能を追加

<a id="january-25-2022"></a>
## 2022. 01. 25. { #january-25-2022 }
<a id="january-25-2022-added-features"></a>
### 新規機能追加 { #january-25-2022-added-features }
* [コンソール] S3 API認証情報発行機能の追加

<a id="january-25-2022-feature-updates"></a>
### 機能改善/変更 { #january-25-2022-feature-updates }
* [コンソール] マルチパートオブジェクトを削除する際、セグメントオブジェクトを一括削除
* [API] コンテナ名に入力可能な文字制限
    * 一部の特殊文字(' " < > ;)とスペース、相対パス文字(. ..)を制限します。

<a id="october-26-2021"></a>
## 2021. 10. 26. { #october-26-2021 }
<a id="october-26-2021-feature-updates"></a>
### 機能改善/変更 { #october-26-2021-feature-updates }
* [コンソール] 静的Webサイト設定時の入力可能文字制限ルールを変更
    * 最大256バイト、英数字、一部特殊文字(-, _, ., /)のみ入力できます。
* [コンソール] オブジェクトをコピーする際の入力制限ルールを変更
    * サブフォルダを含むパスを入力できます。
    * {パスの最大長さ} = 1024 - {オブジェクト名の長さ} - 1

<a id="february-23-2021"></a>
## 2021. 02. 23. { #february-23-2021 }
<a id="february-23-2021-feature-updates"></a>
### 機能改善/変更 { #february-23-2021-feature-updates }
* [コンソール] コンテナ設定機能の改善
    * コンテナ単位でアクセスポリシー、オブジェクトのライフサイクル、バージョン管理ポリシー、静的Webサイトなどを設定できるように機能を改善しました。

<a id="november-24-2020"></a>
## 2020. 11. 24. { #november-24-2020 }
<a id="november-24-2020-feature-updates"></a>
### 機能改善/変更 { #november-24-2020-feature-updates }
* [コンソール] 前方一致検索
    * 入力した文字列で始まるコンテナ、フォルダ、オブジェクトを検索できるように検索機能を改善しました。

<a id="february-25-2020"></a>
## 2020. 02. 25. { #february-25-2020 }
<a id="february-25-2020-added-features"></a>
### 新規機能追加 { #february-25-2020-added-features }
* [API] Amazon S3互換APIを追加

<a id="april-24-2018"></a>
## 2018. 04. 24. { #april-24-2018 }
<a id="april-24-2018-feature-updates"></a>
### 機能改善/変更 { #april-24-2018-feature-updates }
* [コンソール] フォルダ名の特殊文字制限
    * コンテナおよびフォルダ名に使用できない特殊文字リストにスラッシュ(/)を追加しました。

<a id="october-26-2017"></a>
## 2017. 10. 26. { #october-26-2017 }
<a id="october-26-2017-feature-updates"></a>
### 機能改善/変更 { #october-26-2017-feature-updates }
* [コンソール] フォルダ名の特殊文字制限
    * コンテナおよびフォルダ名に一部の特殊文字(., .., &, <, >, ", ', ;)と空白を使用できないように修正しました。

<a id="march-23-2017"></a>
## 2017. 03. 23. { #march-23-2017 }
<a id="march-23-2017-feature-updates"></a>
### 機能改善/変更 { #march-23-2017-feature-updates }
* [コンソール] アップロード可能なファイルサイズの明示
    * 最大5GBまでアップロードできます。

<a id="december-22-2016"></a>
## 2016. 12. 22. { #december-22-2016 }
<a id="december-22-2016-bug-fixes"></a>
### バグ修正 { #december-22-2016-bug-fixes }
* [コンソール] フォルダ名が「/」で始まるフォルダを作成した際、フォルダが表示されない問題を修正
* [コンソール] フォルダ名とファイル名が同じ場合、ファイルの削除ができない問題を修正
* [コンソール] タイトルに「#」が含まれるファイルをダウンロードできない問題を修正
* [コンソール] APIエンドポイント設定ポップアップのコピーボタンが動作しない問題を修正
* [コンソール] 0Byteのファイルがダウンロードできない現象を修正

<a id="december-8-2016"></a>
## 2016. 12. 08. { #december-8-2016 }
<a id="december-8-2016-bug-fixes"></a>
### バグ修正 { #december-8-2016-bug-fixes }
* [コンソール] リソースが残っている状態でサービスの利用を終了する際の案内文言を修正
