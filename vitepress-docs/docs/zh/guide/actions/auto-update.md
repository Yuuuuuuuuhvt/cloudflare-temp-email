# Github Actions 部署如何配置自动更新

::: warning 注意
有问题请通过 `Github Issues` 反馈，感谢。
自动更新不会执行 D1 数据库的 sql 文件，当数据库 schema 变动时，需要手动执行。
:::

1. 创建仅授权当前 fork 的 fine-grained personal access token，并授予 `Contents: Read and write` 与 `Workflows: Read and write`。内置 `GITHUB_TOKEN` 无法合并包含 workflow 文件的上游提交
2. 在仓库 `Settings` -> `Secrets and variables` -> `Actions` 中将 token 保存为 `SYNC_TOKEN`
3. 打开仓库的 `Actions` 页面，找到 `Upstream Sync`，点击 `enable workflow` 启用 `workflow`
4. 修改 `Upstream Sync` 的 `schedule` 配置可自定义更新间隔，参考 [cron 表达式](https://crontab.guru/)
5. 如果不希望保存长期 token，可以不启用定时同步，并在仓库主页使用 `Sync fork` 手动同步

只有成功完成的 `Upstream Sync` 会触发后续部署。同步失败时，部署 workflow 会跳过执行，避免重复发布旧提交。
