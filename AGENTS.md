# AGENTS.md — MiroFish 在 Cursor Cloud Agent 中运行指南

本文件给 Cursor Cloud Agent（以及任何在云端环境里操作本仓库的人）阅读，说明如何把
MiroFish 跑起来并自测。

## 项目是什么

MiroFish 是一个多智能体"群体智能预测引擎"：从真实世界抽取种子信息，构建一个并行
数字世界，让上千个有独立人格/记忆的智能体互动演化，从而推演未来走向。

- 后端：Python (Flask)，端口 `5001`，入口 `backend/run.py`
- 前端：Vue + Vite，端口 `3000`，`/api` 反向代理到后端 `5001`
- 仿真引擎：基于 CAMEL-AI 的 OASIS（`camel-oasis` / `camel-ai`）
- 记忆：Zep Cloud

## 运行环境（已由 .cursor 配置自动处理）

- `.cursor/Dockerfile`：装好 Node 20 + Python 3.12 + uv
- `.cursor/environment.json`：
  - `install`: `npm run setup:all`（装 root + frontend 的 npm 依赖，并 `uv sync` 后端依赖）
  - `terminals`: 分别长跑 `npm run backend` 和 `npm run frontend`

## 必需的 Secrets（务必在 Cursor 云端环境的 Secrets 里配置）

后端 `backend/app/config.py` 直接从环境变量读取，**无需创建 .env 文件**，把下面这些配成
Runtime Secret 即可（敏感 Key 用 Runtime Secret，不会泄露到对话/commit）：

| 变量名 | 必需 | 说明 |
|---|---|---|
| `LLM_API_KEY` | ✅ | LLM 密钥。推荐阿里百炼 qwen-plus：https://bailian.console.aliyun.com/ |
| `LLM_BASE_URL` | ✅ | 例如 `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| `LLM_MODEL_NAME` | ✅ | 例如 `qwen-plus` |
| `ZEP_API_KEY` | ✅ | Zep Cloud 记忆，免费额度即可：https://app.getzep.com/ |
| `OASIS_DEFAULT_MAX_ROUNDS` | 可选 | 模拟轮数，默认 10。**先用 < 40 试，消耗很大** |

> ⚠️ 成本提示：LLM 消耗很大，每次模拟会产生大量 token 调用。先用少量轮数验证流程，
> 确认通了再加大规模。

## 如何验证已经跑起来

1. 等 `install` 完成、两个 terminal（backend / frontend）都起来。
2. 后端启动时会调用 `Config.validate()`，若缺少 `LLM_API_KEY` / `ZEP_API_KEY` 会直接报错退出
   —— 看到后端正常监听 `0.0.0.0:5001` 说明 Key 配好了。
3. 通过远程桌面接管 VM，在 VM 内浏览器打开 `http://localhost:3000`，应能看到 MiroFish 前端。
4. 在界面上传一份种子材料（数据分析报告或小说文本）并用自然语言描述预测需求，跑一次小规模模拟验证链路。

## 注意

- Cloud Agent 环境是临时的、随任务销毁，**不适合作为长期对外托管的线上服务**。
  它适合：开发、改代码、自测、出 PR。
- 若 Docker 构建因 `dockerfile` 路径报错，把 `.cursor/environment.json` 里的
  `build.dockerfile` 改成 `.cursor/Dockerfile` 再试（不同 Cursor 版本解析基准可能不同）。
