# Golang開発環境用Dockerfile（最適化版・Podman対応）
# BuildKit/Buildah機能を活用した高速ビルド
#
# Podmanでビルドする場合:
#   podman build --format docker -t go-devcontainer:latest .
# または環境変数を設定:
#   export BUILDAH_FORMAT=docker
#   podman build -t go-devcontainer:latest .
FROM mcr.microsoft.com/devcontainers/base:debian AS builder

# 環境変数の設定
ENV GO_VERSION=1.25.3 \
    DEBIAN_FRONTEND=noninteractive

# システムパッケージの更新とインストール
RUN --mount=type=cache,target=/var/cache/apt \
    --mount=type=cache,target=/var/lib/apt \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get dist-upgrade -y && \
    apt-get install -y --no-install-recommends \
        sudo curl wget ca-certificates \
        git make build-essential gcc g++ \
        tzdata locales \
    && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Goのダウンロードとインストール（キャッシュ活用）
RUN --mount=type=cache,target=/tmp/go-cache \
    cd /tmp/go-cache && \
    if [ ! -f "go${GO_VERSION}.linux-amd64.tar.gz" ]; then \
        wget -q -O "go${GO_VERSION}.linux-amd64.tar.gz" \
            "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz"; \
    fi && \
    tar -C /usr/local -xzf "go${GO_VERSION}.linux-amd64.tar.gz"

# 最終ステージ
FROM mcr.microsoft.com/devcontainers/base:debian

# メタデータ
LABEL maintainer="onisi@omu.ac.jp" \
      description="Optimized Golang development environment for education" \
      version="25.10.1" \
      base-os="debian-bookworm" \
      build-date="" \
      org.opencontainers.image.source="https://github.com/OMU-WSYSCB-ONISI/golang" \
      org.opencontainers.image.licenses="MIT"　

# ARG for build-time configuration
ARG GO_VERSION=1.25.3
ARG TARGETARCH=amd64

# 環境変数
ENV GOPATH=/go \
    GOBIN=/go/bin \
    PATH=/usr/local/go/bin:/go/bin:$PATH \
    TZ=Asia/Tokyo \
    LANG=ja_JP.UTF-8 \
    LC_ALL=ja_JP.UTF-8 \
    GOCACHE=/home/vscode/.cache/go-build

# Goをコピー
COPY --from=builder /usr/local/go /usr/local/go

# システムパッケージのインストールとクリーンアップ（1レイヤー）
#RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
#    --mount=type=cache,target=/var/lib/apt,sharing=locked \
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        # 必須パッケージ
        sudo git make gcc ca-certificates \
        tzdata locales \
        # 便利なツール
        curl wget vim nano less \
    && \
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

# ディレクトリ作成と所有権設定（1レイヤー）
RUN mkdir -p /go/src /go/bin /go/pkg /workspaces /home/vscode/.cache/go-build && \
    chown -R vscode:vscode /go /workspaces /home/vscode/.cache && \
    chmod -R 755 /go /workspaces

# /workspacesディレクトリの作成と所有権設定
RUN mkdir -p /workspaces && \
    chown -R vscode:vscode /workspaces && \
    chmod -R 755 /workspaces

# vscodeユーザーに切り替え
USER vscode

# Go開発ツール（最小限・並列インストール）
#RUN --mount=type=cache,target=/go/pkg/mod,uid=1000,gid=1000 \
RUN go install golang.org/x/tools/gopls@latest & \
    go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest & \
    go install golang.org/x/tools/cmd/goimports@v0.14.0 & \
    go install github.com/go-delve/delve/cmd/dlv@latest & \
    wait && \
    go clean -modcache && \
    rm -rf /go/pkg/mod/cache

WORKDIR /workspaces

# ヘルスチェック（改善版）
# 注意: Podmanでビルドする場合は --format docker オプションが必要
# HEALTHCHECKはDocker image formatでのみサポートされます
# 例: podman build --format docker -t myimage .
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD go version && \
        gopls version && \
        [ -d "/go" ] && \
        [ -w "/workspaces" ] || exit 1

# デフォルトコマンド
CMD ["/bin/bash"]

# セキュリティ情報（コメント）
# - 非rootユーザー（vscode: UID/GID 1000）で実行
# - 最小限のパッケージのみインストール
# - 定期的なセキュリティアップデートを推奨










