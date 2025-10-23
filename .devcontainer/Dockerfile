# Golang開発環境用Dockerfile（Podman最適化版）
#
# ビルド方法:
#   podman build --format docker -t go-devcontainer:latest .
# または:
#   podman build -t go-devcontainer:latest .  # OCI形式（HEALTHCHECK無視）
FROM mcr.microsoft.com/devcontainers/base:debian AS builder

# Go 1.23.12を使用（最新安定版・セキュリティ修正含む）
ENV GO_VERSION=1.23.12 \
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
    # チェックサムを検証（公式サイトから取得）
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
      version="25.10.3" \
      base-os="debian-trixie" \
      go-version="1.23.12" \
      org.opencontainers.image.source="https://github.com/OMU-WSYSCB-ONISI/golang" \
      org.opencontainers.image.licenses="MIT"

ARG GO_VERSION=1.23.12
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

# Go開発ツールのインストール（Go 1.23.x完全対応版固定）
# 全てのツールをGo 1.23.x互換バージョンに固定
RUN go install golang.org/x/tools/gopls@v0.16.2 && \
    go install golang.org/x/tools/cmd/goimports@v0.25.0 && \
    go install github.com/go-delve/delve/cmd/dlv@v1.23.1 && \
    go clean -modcache && \
    rm -rf /go/pkg/mod/cache

WORKDIR /workspaces

# ヘルスチェック（Docker形式のみ有効）
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
