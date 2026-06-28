# Cursor Cloud Agent 环境镜像
#
# 说明：
# - 这个 Dockerfile 只负责把"运行时工具链"装好（Node 20 + Python 3.12 + uv）。
# - 不要 COPY 项目源码：Cursor 会自行管理 workspace 并 checkout 正确的 commit，
#   项目依赖由 environment.json 里的 install 命令（npm run setup:all）安装。
# - MiroFish 后端要求 Python >=3.11,<3.13，这里固定 3.12；前端要求 Node >=18，这里用 20。

FROM python:3.12-slim-bookworm

# 系统依赖 + Node.js 20（NodeSource）
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
       curl ca-certificates git build-essential \
  && curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
  && apt-get install -y --no-install-recommends nodejs \
  && rm -rf /var/lib/apt/lists/*

# 从 uv 官方镜像复制 uv（Python 包管理器，后端用它 sync 依赖）
COPY --from=ghcr.io/astral-sh/uv:0.9.26 /uv /uvx /bin/

# 自检（构建日志里能看到版本，方便排查）
RUN node -v && npm -v && python --version && uv --version
