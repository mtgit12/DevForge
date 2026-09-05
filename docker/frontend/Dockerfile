# syntax=docker/dockerfile:1

# =====================================================================
# Bun + Next.js (output: "standalone")
#
# ステージ構成:
#   base    : 共通のベース設定
#   deps    : 依存関係のインストール（キャッシュ層）
#   dev     : 開発用（HMR / bun run dev）      -> compose の target: dev
#   builder : 本番ビルド（bun run build）
#   runner  : 本番実行（standalone server.js） -> compose の target: runner
#
# build context は ./frontend（Next.js アプリのルート）を想定
# =====================================================================

ARG BUN_VERSION=1

# ---------------------------------------------------------------------
# Stage: base
# ---------------------------------------------------------------------
FROM oven/bun:${BUN_VERSION}-alpine AS base

WORKDIR /app

# Next.js の匿名テレメトリを無効化（送信したい場合はこの行を削除）
ENV NEXT_TELEMETRY_DISABLED=1

# ---------------------------------------------------------------------
# Stage: deps  — 依存関係のみをインストールしてレイヤキャッシュを効かせる
# ---------------------------------------------------------------------
FROM base AS deps

# lockfile 系を先にコピーすることで、ソース変更時に再インストールが走らないようにする
# bun.lock  : Bun 1.2 以降のテキスト形式
# bun.lockb : Bun 1.1 以前のバイナリ形式
COPY package.json bun.lock* bun.lockb* ./

RUN --mount=type=cache,target=/root/.bun/install/cache \
    bun install --frozen-lockfile

# ---------------------------------------------------------------------
# Stage: dev  — 開発サーバー（Compose からバインドマウントして使う）
# ---------------------------------------------------------------------
FROM base AS dev

ENV NODE_ENV=development
ENV HOSTNAME=0.0.0.0
ENV PORT=3000

# Docker（特に macOS / Windows）内ではファイル変更イベントが届かないことがあるため
# ポーリングでの監視を有効にする
ENV WATCHPACK_POLLING=true
ENV CHOKIDAR_USEPOLLING=true

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# 開発ステージはあえて root のまま動かす。
# ホスト側ソースをバインドマウントするため、UID 不一致による書き込み権限エラーを避ける狙い。
# （本番ステージ runner は非 root で実行する）

EXPOSE 3000

CMD ["bun", "run", "dev"]

# ---------------------------------------------------------------------
# Stage: builder  — 本番ビルド
# ---------------------------------------------------------------------
FROM base AS builder

ENV NODE_ENV=production

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# public/ が存在しないプロジェクトでも runner の COPY が失敗しないようにする
RUN mkdir -p public

# next.config.ts に output: "standalone" が必須
#
# ビルドキャッシュを効かせたい場合は下記のように cache mount を足せる。
# ただし .next/cache がイメージに含まれなくなるため、
# ビルド時の fetch キャッシュは実行時に利用できなくなる点に注意。
#   RUN --mount=type=cache,target=/app/.next/cache bun run build
RUN bun run build

# ---------------------------------------------------------------------
# Stage: runner  — 本番実行（最小構成）
# ---------------------------------------------------------------------
FROM oven/bun:${BUN_VERSION}-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1
ENV PORT=3000
ENV HOSTNAME=0.0.0.0

# プリレンダーキャッシュ用のディレクトリを先に用意し、所有者を bun にしておく
RUN mkdir -p .next && chown bun:bun .next

COPY --from=builder --chown=bun:bun /app/public ./public

# Output File Tracing により、実行に必要なファイルだけがコピーされる
# https://nextjs.org/docs/app/api-reference/config/next-config-js/output
COPY --from=builder --chown=bun:bun /app/.next/standalone ./
COPY --from=builder --chown=bun:bun /app/.next/static ./.next/static

# ビルド時の fetch キャッシュを起動直後から使いたい場合は以下を有効化
# COPY --from=builder --chown=bun:bun /app/.next/cache ./.next/cache

# セキュリティのため非 root ユーザーに切り替える
USER bun

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://127.0.0.1:3000/ || exit 1

# standalone 出力の server.js を Bun で起動する
CMD ["bun", "server.js"]