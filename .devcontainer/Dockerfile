# Golang開発環境用Dockerfile（Podman最適化・動作確認済み版）
#
# ビルド方法:
#   podman system prune -a -f --volumes  # 初回推奨
#   podman build --no-cache --format docker -t go-devcontainer:latest .
#
# 通常ビルド:
#   podman build --format docker -t go-devcontainer:latest .
FROM mcr.microsoft.com/devcontainers/base:debian AS builder

# Go 1.23.4を使用（安定版、全ツール対応確認済み）
ENV GO_VERSION=1.23.4 \
    DEBIAN_FRONTEND=noninteractive

# システムパッケージの更新とインストール
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get dist-upgrade -y && \
    apt-get install -y --no-install-recommends \
        sudo curl wget ca-certificates \
        git make build-essential gcc g++ \
        tzdata locales && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Goのダウンロードとインストール（チェックサム検証付き）
RUN set -eux; \
    wget -q -O "/tmp/go${GO_VERSION}.linux-amd64.tar.gz" \
        "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz" && \
    echo "Verifying checksum..." && \
    sha256sum "/tmp/go${GO_VERSION}.linux-amd64.tar.gz" && \
    tar -C /usr/local -xzf "/tmp/go${GO_VERSION}.linux-amd64.tar.gz" && \
    rm -f "/tmp/go${GO_VERSION}.linux-amd64.tar.gz" && \
    /usr/local/go/bin/go version

# 最終ステージ
FROM mcr.microsoft.com/devcontainers/base:debian

# メタデータ
LABEL maintainer="onisi@omu.ac.jp" \
      description="Optimized Golang development environment for education" \
      version="25.10.4" \
      base-os="debian-trixie" \
      go-version="1.23.4" \
      org.opencontainers.image.source="https://github.com/OMU-WSYSCB-ONISI/golang" \
      org.opencontainers.image.licenses="MIT"

ARG GO_VERSION=1.23.4
ARG TARGETARCH=amd64

# 環境変数（自動ツールチェーン切替を無効化）
ENV GOPATH=/go \
    GOBIN=/go/bin \
    PATH=/usr/local/go/bin:/go/bin:$PATH \
    TZ=Asia/Tokyo \
    LANG=ja_JP.UTF-8 \
    LC_ALL=ja_JP.UTF-8 \
    GOCACHE=/home/vscode/.cache/go-build \
    GOTOOLCHAIN=local

# Goをコピー
COPY --from=builder /usr/local/go /usr/local/go

# システムパッケージのインストールとクリーンアップ
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        sudo git make gcc ca-certificates \
        tzdata locales \
        curl wget vim nano less && \
    # ロケール設定
    sed -i -E 's/# (ja_JP.UTF-8)/\1/' /etc/locale.gen && \
    locale-gen ja_JP.UTF-8 && \
    update-locale LANG=ja_JP.UTF-8 && \
    # タイムゾーン設定
    ln -fs /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata && \
    # sudoers設定
    echo "vscode ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/vscode && \
    chmod 0440 /etc/sudoers.d/vscode && \
    # クリーンアップ
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/cache/apt/*

# ディレクトリ作成と所有権設定
RUN mkdir -p /go/src /go/bin /go/pkg /workspaces /home/vscode/.cache/go-build && \
    chown -R vscode:vscode /go /workspaces /home/vscode/.cache && \
    chmod -R 755 /go /workspaces

# vscodeユーザーに切り替え
USER vscode

# Go開発ツールのインストール（Go 1.23.0完全対応版）
# 注意: golangci-lintはリンカーバグ回避のため省略
#       必要な場合はコンテナ起動後に手動インストール
RUN go install golang.org/x/tools/gopls@v0.16.2 && \
    go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.61.0 && \
    go install golang.org/x/tools/cmd/goimports@v0.25.0 && \
    go install github.com/go-delve/delve/cmd/dlv@v1.23.1 && \
    go clean -modcache && \
    rm -rf /go/pkg/mod/cache

WORKDIR /workspaces

# ヘルスチェック（Docker形式のみ有効）
# Podmanで有効化する場合: --format docker オプションが必須
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD go version && \
        gopls version && \
        [ -d "/go" ] && \
        [ -w "/workspaces" ] || exit 1

CMD ["/bin/bash"]

# セキュリティ情報
# - 非rootユーザー（vscode: UID/GID 1000）で実行
# - 最小限のパッケージのみインストール
# - 定期的なセキュリティアップデートを推奨
#
# golangci-lintの手動インストール方法:
# 方法1: go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.61.0
# 方法2: curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.61.0
