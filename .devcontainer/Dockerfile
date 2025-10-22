# Golang開発環境用Dockerfile（修正版）
# マルチステージビルドでイメージサイズを削減
FROM mcr.microsoft.com/devcontainers/base:debian AS builder

# 環境変数の設定
ENV GO_VERSION=1.25.3 \
    DEBIAN_FRONTEND=noninteractive

# システムパッケージの更新とインストール
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get dist-upgrade -y && \
    apt-get install -y --no-install-recommends \
        sudo curl wget ca-certificates \
        git make build-essential gcc g++ \
        tzdata locales \
    && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Goのインストール
RUN wget -q -O go.tar.gz https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz && \
    tar -C /usr/local -xzf go.tar.gz && \
    rm go.tar.gz

# 最終ステージ
FROM mcr.microsoft.com/devcontainers/base:debian

# メタデータ
LABEL maintainer="onisi@omu.ac.jp"
LABEL description="Optimized Golang development environment"
LABEL version="25.10.1"
LABEL base-os="debian-bookworm"
LABEL build-date=""

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

# 最小限のパッケージのみインストール
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
RUN go install golang.org/x/tools/gopls@v0.14.2 & \
    go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.62.2 & \
    go install golang.org/x/tools/cmd/goimports@v0.14.0 & \
    go install github.com/go-delve/delve/cmd/dlv@latest & \
    wait && \
    go clean -modcache && \
    rm -rf /go/pkg/mod/cache

WORKDIR /workspaces

# ヘルスチェック（改善版）
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD go version && \
        gopls version && \
        [ -d "/go" ] && \
        [ -w "/workspaces" ] || exit 1

# デフォルトコマンド
CMD ["/bin/bash"]



