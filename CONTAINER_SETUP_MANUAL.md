# コンテナイメージ作成：手動ビルド版ガイド（教員向け）

## 📋 概要

このガイドでは、教員が**学期初めに1回だけ**コンテナイメージを手動でビルド・公開し、学生は**事前ビルド済みイメージ**を使用する方法を説明します。

**所要時間**: 約20分（学期初めに1回のみ）  
**前提条件**: Podman がインストール済み

**使用ツール**: Podman のみ（Dockerは不要）

**セキュリティ**: ビルド時に最新のセキュリティアップデートが適用され、学期を通じて同じイメージを使用します。

---

## 🎯 この方法の利点

- ✅ **学生の初回起動が1-2分**（5-10分ではない）
- ✅ GitHub Actions不要（手動ビルド）
- ✅ 学期中のメンテナンスほぼゼロ
- ✅ 教員が完全にコントロール

---

## 🚀 セットアップ手順（学期初めに1回）

### ステップ1: GitHub Personal Access Token の作成（5分）

#### 1-1. GitHubにアクセス

1. https://github.com/settings/tokens にアクセス
2. 「**Generate new token**」→「**Generate new token (classic)**」をクリック

#### 1-2. トークンの設定

1. **Note**: `Codespaces Container Registry`
2. **Expiration**: `No expiration`（または学期終了日）
3. **Scopes**: 以下にチェック
   - ✅ `write:packages` - パッケージの書き込み
   - ✅ `read:packages` - パッケージの読み取り
   - ✅ `delete:packages` - パッケージの削除（オプション）

4. 「**Generate token**」をクリック

5. 表示されたトークンを**すぐにコピー**（一度しか表示されません）
   ```
   例: ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   ```

6. 安全な場所に保存（パスワードマネージャー推奨）

---

### ステップ2: ローカルでイメージをビルド（5分）

#### 2-1. リポジトリをクローン

```bash
# ローカルにクローン
git clone https://github.com/your-username/your-repository.git
cd your-repository
```

#### 2-2. イメージをビルド

**Docker使用の場合**:

```bash
# .devcontainerディレクトリに移動
cd .devcontainer

# イメージをビルド
docker build \
  --tag ghcr.io/your-username/go-devcontainer:latest \
  --tag ghcr.io/your-username/go-devcontainer:2025spring \
  .

# ビルド成功の確認
docker images | grep go-devcontainer
```

**Podman使用の場合**:

```bash
# .devcontainerディレクトリに移動
cd .devcontainer

# イメージをビルド
podman build \
  --tag ghcr.io/your-username/go-devcontainer:latest \
  --tag ghcr.io/your-username/go-devcontainer:2025spring \
  .

# ビルド成功の確認
podman images | grep go-devcontainer
```

**ビルド時間**: 約5分

**重要**: `your-username` を実際のGitHubユーザー名に置き換えてください。

---

### ステップ3: GitHub Container Registry にログイン（1分）

#### Docker使用の場合:

```bash
# 環境変数にトークンを設定
export GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ログイン
echo $GITHUB_TOKEN | docker login ghcr.io -u your-username --password-stdin
```

#### Podman使用の場合:

```bash
# 環境変数にトークンを設定
export GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ログイン
echo $GITHUB_TOKEN | podman login ghcr.io -u your-username --password-stdin
```

**ログイン成功**: `Login Succeeded!` と表示される

---

### ステップ4: イメージをプッシュ（5分）

#### Docker使用の場合:

```bash
# latestタグをプッシュ
docker push ghcr.io/your-username/go-devcontainer:latest

# 学期タグもプッシュ
docker push ghcr.io/your-username/go-devcontainer:2025spring
```

#### Podman使用の場合:

```bash
# latestタグをプッシュ
podman push ghcr.io/your-username/go-devcontainer:latest

# 学期タグもプッシュ
podman push ghcr.io/your-username/go-devcontainer:2025spring
```

**プッシュ時間**: 約3-5分

**進捗表示例**:
```
Pushing layer sha256:abc123...
Pushing layer sha256:def456...
...
Successfully pushed
```

---

### ステップ5: パッケージを公開設定（2分）

#### 5-1. GitHubでパッケージを確認

1. GitHubプロフィール → 「**Packages**」タブ
2. 「**go-devcontainer**」パッケージが表示される

#### 5-2. 公開設定

1. パッケージをクリック
2. 右側の「**Package settings**」
3. 下にスクロール → 「**Danger Zone**」
4. 「**Change visibility**」をクリック
5. 「**Public**」を選択
6. パッケージ名 `go-devcontainer` を入力
7. 「**I understand, change package visibility**」をクリック

#### 5-3. リポジトリと連携（推奨）

1. Package settings で「**Connect repository**」
2. テンプレートリポジトリを選択
3. 「**Connect repository**」をクリック

---

### ステップ6: devcontainer.json を更新（1分）

#### 6-1. イメージ参照に変更

`.devcontainer/devcontainer.json` の先頭部分を更新:

**変更前**:
```json
{
  "name": "Golang Development Environment (Debian)",
  "build": {
    "dockerfile": "Dockerfile",
    "context": "."
  },
  ...
}
```

**変更後**:
```json
{
  "name": "Golang Development Environment (Debian)",
  
  // 事前ビルド済みイメージを使用
  "image": "ghcr.io/your-username/go-devcontainer:latest",
  
  // buildセクションは削除またはコメントアウト
  // "build": {
  //   "dockerfile": "Dockerfile",
  //   "context": "."
  // },
  
  ...
}
```

**重要**: `your-username` を実際のGitHubユーザー名に置き換えてください。

#### 6-2. コミット・プッシュ

```bash
git add .devcontainer/devcontainer.json
git commit -m "Use pre-built container image"
git push origin main
```

---

## ✅ 動作確認（2分）

### Codespacesでのテスト

1. リポジトリページで「**Code**」→「**Codespaces**」
2. 「**Create codespace on main**」をクリック
3. **約1-2分で起動**（事前ビルド済みのため高速）
4. VS Codeのような画面が表示されたら成功 ✅

### 環境の確認

Codespaces内のターミナルで:

```bash
go version
gopls version
golangci-lint version
dlv version
air -v
```

すべて正常に表示されれば完了です。

---

## 🔄 学期中に更新が必要になった場合（稀）

### 緊急更新の手順

1. **Dockerfileを編集**
   ```bash
   vim .devcontainer/Dockerfile
   # Go版を更新など
   ```

2. **再ビルド**
   ```bash
   cd .devcontainer
   docker build --tag ghcr.io/your-username/go-devcontainer:latest .
   ```

3. **再プッシュ**
   ```bash
   docker push ghcr.io/your-username/go-devcontainer:latest
   ```

4. **学生に通知**
   ```
   【重要】開発環境の更新
   
   Codespace内で:
   Ctrl+Shift+P → "Codespaces: Rebuild Container"
   約1-2分で最新環境に更新されます
   ```

---

## 📋 完了チェックリスト

### 学期初めの作業（1回のみ）

- [ ] Personal Access Token作成
- [ ] ローカルでイメージをビルド（5分）
- [ ] GitHub Container Registryにログイン
- [ ] イメージをプッシュ（5分）
- [ ] パッケージを公開設定
- [ ] リポジトリと連携
- [ ] devcontainer.jsonをimage参照に変更
- [ ] コミット・プッシュ
- [ ] Codespacesで動作確認（1-2分起動）

**合計所要時間: 約20分**

---

## 🛠️ トラブルシューティング

### Q1: ビルドが失敗する

**A**: Dockerfileの構文を確認

```bash
# エラーメッセージを確認
docker build --tag test .

# よくある原因:
# - Go版の指定ミス
# - パッケージ名の間違い
# - ネットワークエラー
```

### Q2: プッシュが失敗する

**A**: ログイン状態を確認

```bash
# 再ログイン
docker logout ghcr.io
echo $GITHUB_TOKEN | docker login ghcr.io -u your-username --password-stdin

# 再プッシュ
docker push ghcr.io/your-username/go-devcontainer:latest
```

### Q3: 学生がイメージをプルできない

**A**: パッケージの公開設定を確認

```
GitHub → Packages → go-devcontainer
→ Settings → Visibility が "Public" になっているか確認
```

### Q4: Codespacesが古いイメージを使っている

**A**: Rebuild Containerを実行

```
Codespace内で:
Ctrl+Shift+P → "Codespaces: Rebuild Container"
```

---

## 💡 タグ戦略（推奨）

### 複数のタグを使用

```bash
# ビルド時に複数のタグを付ける
docker build \
  --tag ghcr.io/your-username/go-devcontainer:latest \
  --tag ghcr.io/your-username/go-devcontainer:2025spring \
  --tag ghcr.io/your-username/go-devcontainer:go1.23 \
  .

# すべてプッシュ
docker push ghcr.io/your-username/go-devcontainer:latest
docker push ghcr.io/your-username/go-devcontainer:2025spring
docker push ghcr.io/your-username/go-devcontainer:go1.23
```

### タグの使い分け

| タグ | 用途 | 推奨 |
|------|------|------|
| `latest` | 常に最新版 | ✅ 推奨 |
| `2025spring` | 学期固定版 | ⭐ 推奨（ロールバック用） |
| `go1.23` | Go版固定 | オプション |

**devcontainer.jsonでは `latest` を使用**

---

## 📊 この方法の利点まとめ

### 学生側

- ✅ **初回起動が1-2分**（事前ビルド済み）
- ✅ 2回目以降も1-2分
- ✅ 環境構築の待ち時間なし
- ✅ トラブルが少ない

### 教員側

- ✅ **学期初めに20分の作業のみ**
- ✅ GitHub Actions不要
- ✅ イメージを完全にコントロール
- ✅ 学期中のメンテナンスほぼゼロ

### システム全体

- ✅ シンプルな構成
- ✅ 信頼性が高い
- ✅ トラブルが少ない
- ✅ コスト: ほぼ無料

---

## 🎉 まとめ

### 実現できること

```
初回起動: 1-2分（事前ビルド済み）
2回目以降: 1-2分
教員作業: 学期初めに20分（1回のみ）
学期中: メンテナンスほぼゼロ
```

### 次のステップ

1. Personal Access Token作成
2. ローカルでイメージをビルド（5分）
3. GitHub Container Registryにプッシュ（5分）
4. パッケージを公開設定
5. devcontainer.jsonを更新
6. 動作確認

**この手順で、高速起動かつメンテナンスフリーの環境が実現します！** 🚀
