# 通过 Github Actions 部署

::: warning 注意
目前只支持 worker 和 pages 的部署。
有问题请通过 `Github Issues` 反馈，感谢。

`worker.dev` 域名在中国无法访问，请自定义域名
:::

## 部署步骤

### Fork 仓库并启用 Actions

- 在 GitHub fork 本仓库
- 打开仓库的 `Actions` 页面
- 找到 `Deploy Backend` 点击 `enable workflow` 启用 `workflow`
- 如果需要前后端分离并直连 Worker, 找到 `Deploy Frontend` 点击 `enable workflow` 启用 `workflow`
- 如果需要通过 Page Functions 转发后端请求的 Pages 部署, 找到 `Deploy Frontend with page function` 点击 `enable workflow` 启用 `workflow`

### 配置 Secrets

然后在仓库页面 `Settings` -> `Secrets and variables` -> `Actions` -> `Repository secrets`, 添加以下 `secrets`:

- 公共 `secrets`

   | 名称                    | 说明                                                                                                            |
   | ----------------------- | --------------------------------------------------------------------------------------------------------------- |
   | `CLOUDFLARE_ACCOUNT_ID` | Cloudflare 账户 ID, [参考文档](https://developers.cloudflare.com/workers/wrangler/ci-cd/#cloudflare-account-id) |
   | `CLOUDFLARE_API_TOKEN`  | Cloudflare API Token, [参考文档](https://developers.cloudflare.com/workers/wrangler/ci-cd/#api-token)           |

- worker 后端 `secrets`

   | 名称                           | 说明                                                                                                                                    |
   | ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
   | `BACKEND_TOML`                 | 后端配置文件，[参考此处](/zh/guide/cli/worker.html#修改-wrangler-toml-配置文件)                                                         |
   | `ADMIN_PASSWORDS_JSON`         | (可选) Admin 控制台密码的 JSON 数组，例如 `["请替换为随机密码"]`。配置后，部署流程会将其作为 Cloudflare 加密 Secret 覆盖 `BACKEND_TOML` 中的 `ADMIN_PASSWORDS`，可用于轮换无法读取的旧密码 |
   | `DEBUG_MODE`                   | (可选) 是否开启调试模式，配置为 `true` 开启, 默认 worker 部署日志不会输出到 Github Actions 页面，开启后会输出                           |
   | `BACKEND_USE_MAIL_WASM_PARSER` | (可选) 是否使用 wasm 解析邮件，配置为 `true` 开启, 功能参考 [配置 worker 使用 wasm 解析邮件](/zh/guide/feature/mail_parser_wasm_worker) |
   | `USE_WORKER_ASSETS`            | (可选) 部署带有前端资源的 Worker, 配置为 `true` 开启                                                                                    |

- pages 前端 `secrets`

   > [!warning] 注意
   > 如果选择部署带有前端资源的 Worker, 则无须配置这些 `secrets`

   | 名称               | 说明                                                                                                                                                                                      |
   | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
   | `FRONTEND_ENV`     | `Deploy Frontend` workflow 使用的前端配置文件，请复制 `frontend/.env.example` 的内容，[并参考此处修改](/zh/guide/cli/pages.html)。如果是前后端分离直连 Worker，`VITE_API_BASE` 应填写后端 Worker API 根地址，并且以 `https://` 开头、末尾不要带 `/`。地址配置错误时，常见现象是前端报 `map` 错误或接口返回 `405` |
   | `FRONTEND_NAME`    | 你在 Cloudflare Pages 创建的项目名称，可通过 [用户界面](https://temp-mail-docs.awsl.uk/zh/guide/ui/pages.html) 或者 [命令行](https://temp-mail-docs.awsl.uk/zh/guide/cli/pages.html) 创建 |
   | `FRONTEND_BRANCH`  | (可选) pages 部署的分支，可不配置，默认 `production`                                                                                                                                      |
   | `PAGE_TOML`        | (可选) 仅供 `Deploy Frontend with page function` workflow 使用。通过 page functions 转发后端请求时需要配置，请复制 `pages/wrangler.toml` 的内容，并根据实际情况修改 `service` 字段为你的 worker 后端名称。这个 workflow 会以 Pages 模式构建前端并走同域请求，因此不会读取 `FRONTEND_ENV` |
   | `TG_FRONTEND_NAME` | (可选) 你在 Cloudflare Pages 创建的项目名称，同 `FRONTEND_NAME`，如果需要 Telegram Mini App 功能，请填写                                                                                  |

- 自动更新 `secrets`

   | 名称         | 说明 |
   | ------------ | ---- |
   | `SYNC_TOKEN` | 启用 `Upstream Sync` 时必填。创建仅授权当前 fork 的 fine-grained personal access token，并授予 `Contents: Read and write` 与 `Workflows: Read and write`。GitHub 内置的 `GITHUB_TOKEN` 无法合并包含 workflow 文件的上游提交 |

### 部署

- 打开仓库的 `Actions` 页面
- 找到 `Deploy Backend` 点击 `Run workflow` 选择分支手动部署
- 如果需要前后端分离并直连 Worker, 找到 `Deploy Frontend`，点击 `Run workflow` 选择分支手动部署
- 如果需要通过 Page Functions 转发后端请求的 Pages 部署, 找到 `Deploy Frontend with page function`，点击 `Run workflow` 手动部署

如果需要轮换 Admin 控制台密码，请生成至少 32 位的随机密码，将 `["随机密码"]` 保存为 `ADMIN_PASSWORDS_JSON`，然后手动运行 `Deploy Backend`。部署成功后，新密码立即生效。请持续保留该 Secret；如果删除它，后续部署会重新使用 `BACKEND_TOML` 中的 `ADMIN_PASSWORDS`。

### 自动更新与 Page Functions 转发

如果你既想通过 `Upstream Sync` 自动更新，又想让 Pages 通过 Page Functions 转发后端请求，请使用 `Deploy Frontend with page function` workflow，而不是 `Deploy Frontend`。

- 先启用 `Upstream Sync`、`Deploy Backend` 和 `Deploy Frontend with page function`
- 在仓库 `Secrets` 中配置 `PAGE_TOML`，内容复制 `pages/wrangler.toml`
- 将 `PAGE_TOML` 里的 `service` 改成你的 Worker 后端名称
- 这个 workflow 会执行 `pnpm build:pages`，前端走同域请求，不读取 `FRONTEND_ENV`
- 每次 `Upstream Sync` 成功完成后，如果 `PAGE_TOML` 已配置，`Deploy Frontend with page function` 会自动部署前端；同步失败不会触发部署

## 如何配置自动更新

1. 创建仅授权当前 fork 的 fine-grained personal access token，并授予 `Contents: Read and write` 与 `Workflows: Read and write`
2. 在仓库 `Settings` -> `Secrets and variables` -> `Actions` 中将 token 保存为 `SYNC_TOKEN`
3. 打开仓库的 `Actions` 页面，找到 `Upstream Sync`，点击 `enable workflow` 启用 `workflow`
4. 如果不希望保存长期 token，可以不启用定时同步，并在仓库主页使用 `Sync fork` 手动同步
