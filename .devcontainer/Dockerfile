# Golang開発環境用Dockerfile（最終最適化版）
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
RUN wget -q -O go.tar.gz https://go.dev/dl/go1.25.3.linux-amd64.tar.gz && \
    tar -C /usr/local -xzf go.tar.gz && \
    rm go.tar.gz

# 最終ステージ
FROM mcr.microsoft.com/devcontainers/base:debian

# メタデータ
LABEL maintainer="your-email@example.com"
LABEL description="Optimized Golang development environment"
LABEL version="2.2"

# 環境変数
ENV GOPATH=/go \
    PATH=/usr/local/go/bin:/go/bin:$PATH \
    TZ=Asia/Tokyo \
    LANG=ja_JP.UTF-8 \
    LC_ALL=ja_JP.UTF-8

# Goをコピー
COPY --from=builder /usr/local/go /usr/local/go

# 最小限のパッケージのみインストール
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        sudo git make gcc ca-certificates \
        tzdata locales \
    && \
    sed -i -E 's/# (ja_JP.UTF-8)/\1/' /etc/locale.gen && \
    locale-gen ja_JP.UTF-8 && \
    ln -fs /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && \
    echo "vscode ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/vscode && \
    chmod 0440 /etc/sudoers.d/vscode && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

USER vscode

# Go作業ディレクトリ
RUN mkdir -p /go/src /go/bin /go/pkg

# Go開発ツール（最小限・並列インストール）
RUN go install golang.org/x/tools/gopls@v0.14.2 & \
    go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.62.2 & \
    go install golang.org/x/tools/cmd/goimports@v0.14.0 & \
    wait && \
    go clean -modcache && \
    rm -rf /go/pkg/mod/cache

WORKDIR /workspaces

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD go version || exit 1

CMD ["/bin/bash"]


