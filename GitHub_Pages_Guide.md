# GitHub Pages 運用ガイド (gametlet-gh-pages リポジトリ用)

このドキュメントは、Gametletアプリのお知らせ、アップデート通知、およびプライバシーポリシーなどの静的コンテンツをGitHub Pagesで公開するための手順を説明します。これらのコンテンツは、別途作成したパブリックリポジトリ `gametlet-gh-pages` を利用して配信されます。

---

## 1. GitHub Pagesリポジトリ (`gametlet-gh-pages`) のセットアップ

### 1.1. リポジトリの作成 (初回のみ)

まだ作成していない場合は、GitHub上で `gametlet-gh-pages` という名前の**パブリック**リポジトリを作成してください。READMEファイルを追加して初期化することをお勧めします。

### 1.2. GitHub Pagesの有効化

`gametlet-gh-pages` リポジトリでGitHub Pagesを有効化します。

1.  `gametlet-gh-pages` リポジトリの **Settings** タブに移動します。
2.  左側のサイドバーメニューから **Pages** を選択します。
3.  **Build and deployment** の下にある **Source** の項目で、**Deploy from a branch** を選択します。
4.  **Branch** で `main` ブランチを選択し、**Folder** で `/` (root) を選択します。
5.  **Save** ボタンをクリックします。

公開処理には1〜2分かかることがあります。設定が完了すると、`https://<あなたのGitHubユーザー名>.github.io/gametlet-gh-pages/` のようなURLでサイトが公開されます。

---

## 2. コンテンツの配置と更新

お知らせ機能で使用する設定ファイル (`app-config.json`, `notices.json`) や、プライバシーポリシー (`privacy_policy.md`)、ルートページ (`index.html`) などのコンテンツは、`gametlet-gh-pages` リポジトリの適切なディレクトリに配置します。

### 推奨ディレクトリ構造

```
gametlet-gh-pages/
├── announcements/
│   ├── app-config.json
│   └── notices.json
├── index.html
└── privacy_policy.md
```

### 2.1. 設定ファイルの配置

*   **`announcements/app-config.json`**: アプリの全体設定、アップデート通知、お知らせの有効/無効などを管理します。
*   **`announcements/notices.json`**: アプリ内で表示される個別のお知らせコンテンツを管理します。

### 2.2. 静的コンテンツの配置

*   **`index.html`**: GitHub PagesのルートURL (`https://<あなたのGitHubユーザー名>.github.io/gametlet-gh-pages/`) で表示されるページです。
*   **`privacy_policy.md`**: プライバシーポリシーのコンテンツです。GitHub PagesはMarkdownファイルを自動的にHTMLに変換して表示します。

### 2.3. コンテンツの更新手順

`gametlet-gh-pages` リポジトリ内のファイルを変更し、`main` ブランチにプッシュするたびに、GitHub Pagesは自動的に更新されます。

1.  ローカルで `gametlet-gh-pages` リポジトリをクローンまたは更新します。
2.  必要なファイルを編集または追加します。
3.  変更をコミットし、`main` ブランチにプッシュします。

    ```bash
    git add .
    git commit -m "Update GitHub Pages content"
    git push origin main
    ```

---

## 3. アプリケーション側の設定

GametletアプリからGitHub Pages上の設定ファイルを参照するようにURLを設定します。

### 3.1. URL設定の確認

`lib/core/services/app_config_service.dart` のURL設定を更新します。

```dart
static const String _productionConfigUrl = 
    'https://<あなたのGitHubユーザー名>.github.io/gametlet-gh-pages/announcements/app-config.json';
```

### 3.2. 動作モードの切り替え

開発環境での確認方法：

```dart
// ローカル設定でテスト
AppConfigService.fetchConfig(useLocalConfig: true)

// GitHub Pagesでテスト  
AppConfigService.fetchConfig(useLocalConfig: false)
```

---

## 4. 動作確認手順

Gametletアプリのお知らせとアップデート通知機能のGitHub Pages連携を動作確認する手順です。

### 4.1. 基本接続テスト

1.  **ブラウザでの直接確認**:
    ```
    https://<あなたのGitHubユーザー名>.github.io/gametlet-gh-pages/announcements/app-config.json
    ```
    → JSONファイルが正しく表示されることを確認します。

2.  **アプリでの接続確認**:
    *   アプリを起動します。
    *   タイトル画面で、`app-config.json` に設定したテスト用のお知らせ（例: "GitHub Pages接続テスト"）が表示されることを確認します。

### 4.2. アップデート通知テスト

`gametlet-gh-pages` リポジトリの `announcements/app-config.json` を更新します。

```json
{
  "latestVersion": "1.1.0",
  "recommendedVersion": "1.0.0", 
  "showUpdateNotification": true,
  "updateNotification": {
    "title": "新しいバージョンが利用可能",
    "message": "バージョン1.1.0がリリースされました。新機能とバグ修正が含まれています。",
    "updateButtonText": "今すぐアップデート",
    "laterButtonText": "後で"
  }
}
```

**確認項目：**
- [ ] タイトル画面でアップデート通知ダイアログが表示される
- [ ] 「今すぐアップデート」ボタンがApp Storeを開く
- [ ] 「後で」ボタンでダイアログが閉じる
- [ ] 通知回数制限（3回）が機能する

### 4.3. お知らせ機能テスト

#### 4.3.1. 緊急お知らせ

`announcements/notices.json` に緊急のお知らせを追加します。

```json
{
  "notices": [
    {
      "id": "urgent_test",
      "title": "緊急メンテナンスのお知らせ",
      "message": "システムメンテナンスのため、一時的にサービスを停止いたします。\n\n日時：2025年2月1日 2:00-4:00\nご不便をおかけして申し訳ございません。",
      "priority": "urgent",
      "created_at": "2025-01-28T15:00:00Z", 
      "valid_until": "2025-02-01T06:00:00Z",
      "show_once": true
    }
  ]
}
```

**確認項目：**
- [ ] 赤いアイコンで表示される
- [ ] 緊急度の高い見た目（赤色テーマ）
- [ ] お知らせ一覧で「緊急」ラベルが表示される

#### 4.3.2. 複数お知らせテスト

`announcements/notices.json` に複数のお知らせを追加します。

```json
{
  "notices": [
    {
      "id": "notice_001",
      "title": "新機能のご紹介",
      "message": "バージョン1.1では以下の新機能が追加されました：\n\n• 新しいパズルモード\n• UI改善\n• パフォーマンス向上",
      "priority": "normal",
      "created_at": "2025-01-28T10:00:00Z",
      "valid_until": "2025-03-31T23:59:59Z", 
      "show_once": true
    },
    {
      "id": "notice_002", 
      "title": "イベント開催中",
      "message": "期間限定のスペシャルパズルイベントを開催中です！",
      "priority": "high",
      "created_at": "2025-01-28T14:00:00Z",
      "valid_until": "2025-02-28T23:59:59Z",
      "show_once": true
    }
  ]
}
```

**確認項目：**
- [ ] 複数お知らせが順番に表示される
- [ ] プログレスインジケーターが表示される
- [ ] 「確認して次へ」ボタンで次のお知らせに進む
- [ ] 最後のお知らせで自動的にダイアログが閉じる

### 4.4. メンテナンスモードテスト

`announcements/app-config.json` を更新します。

```json
{
  "maintenanceMode": true,
  "maintenanceMessage": "現在システムメンテナンス中です。\n\n終了予定時刻：2025年1月28日 18:00\nご不便をおかけして申し訳ございません。"
}
```

**確認項目：**
- [ ] タイトル画面でメンテナンス画面が表示される
- [ ] ホーム画面に進めない
- [ ] 「再確認」ボタンで設定を再読み込み

### 4.5. エラーハンドリングテスト

#### 4.5.1. ネットワークエラー

```bash
# WiFiを無効にしてアプリを起動
# または不正なURLを設定
```

**確認項目：**
- [ ] エラー画面が表示される
- [ ] ローカル設定にフォールバック
- [ ] 「再試行」ボタンが機能する

#### 4.5.2. JSON形式エラー

`announcements/app-config.json` または `announcements/notices.json` に意図的にJSON形式エラーを発生させます。

```json
{
  "invalid": "json format"
  // カンマ抜けなど意図的なエラー
}
```

**確認項目：**
- [ ] 適切なエラーメッセージが表示される
- [ ] アプリがクラッシュしない
- [ ] デフォルト設定で動作継続

---

## 5. 運用のベストプラクティス

1.  **段階的リリース**: 少数のお知らせから開始し、効果を確認してから本格運用します。
2.  **定期的なメンテナンス**: 期限切れお知らせの削除や、設定ファイルの最適化を定期的に行います。
3.  **ユーザー体験の配慮**: お知らせの頻度制限、緊急度の適切な使用、メッセージの簡潔性を心がけます。

---

## 6. トラブルシューティング

### よくある問題

1.  **設定が反映されない**
    *   GitHub Pagesの更新待ち（最大10分程度かかる場合があります）
    *   ブラウザキャッシュをクリア
    *   JSON形式エラーの確認
2.  **お知らせが表示されない**
    *   有効期限の確認
    *   表示履歴のリセット（デバッグボタン使用）
    *   `showOnce` 設定の確認
3.  **アプリがクラッシュする**
    *   JSON形式の検証
    *   必須フィールドの確認
    *   エラーログの確認

### デバッグ方法

```dart
// デバッグ用ログ確認
print('AppConfigService: 設定取得開始');
print('AppConfigService: URL = $_productionConfigUrl');

// テスト用履歴クリア（開発モードのみ）
// タイトル画面の赤いボタンを使用
```