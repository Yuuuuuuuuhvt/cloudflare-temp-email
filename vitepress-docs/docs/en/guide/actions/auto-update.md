# How to Configure Auto-Update for GitHub Actions Deployment

::: warning Notice
If you encounter any issues, please report them via `GitHub Issues`. Thank you.
Auto-update will not execute SQL files for the D1 database. When the database schema changes, you need to execute them manually.
:::

1. Create a fine-grained personal access token scoped only to the fork and grant `Contents: Read and write` plus `Workflows: Read and write`. The built-in `GITHUB_TOKEN` cannot merge upstream commits that modify workflow files
2. Save the token as `SYNC_TOKEN` under repository `Settings` -> `Secrets and variables` -> `Actions`
3. Open the repository `Actions` page, find `Upstream Sync`, and click `enable workflow`
4. Customize the update interval by modifying the `schedule` configuration in `Upstream Sync`; refer to [cron expressions](https://crontab.guru/)
5. If you do not want to store a long-lived token, leave scheduled sync disabled and use `Sync fork` on the repository homepage instead

Only a successful `Upstream Sync` triggers downstream deployment workflows. When synchronization fails, deployments are skipped so that an old commit is not published again.
