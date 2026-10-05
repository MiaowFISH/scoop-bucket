# MiaowFISH/scoop-bucket

个人 [Scoop](https://scoop.sh) bucket。收录我自己常用、但官方 [Extras][extras] 等 bucket 没收录的软件，开源的 GitHub Releases 和闭源官网直链都有。

[![Excavator](https://github.com/MiaowFISH/scoop-bucket/actions/workflows/excavator.yml/badge.svg)](https://github.com/MiaowFISH/scoop-bucket/actions/workflows/excavator.yml)
[![Pull Requests](https://github.com/MiaowFISH/scoop-bucket/actions/workflows/pull_request.yml/badge.svg)](https://github.com/MiaowFISH/scoop-bucket/actions/workflows/pull_request.yml)

## 安装

```pwsh
scoop bucket add miaow https://github.com/MiaowFISH/scoop-bucket
scoop install miaow/<manifest-name>
```

删除 bucket：`scoop bucket rm miaow`

## 收录内容

| manifest | 说明 |
| --- | --- |
| [`folia`](bucket/folia.json) | 全屏沉浸式歌词播放的在线音乐播放器，Electron 桌面端 |
| [`qq-chat-exporter`](bucket/qq-chat-exporter.json) | QQ 聊天记录与表情包导出，需要本机装有 QQNT |
| [`qq-chat-exporter-framework`](bucket/qq-chat-exporter-framework.json) | 同一项目的 Framework 模式，与桌面端 QQ 共用登录状态 |

## 更新是怎么发生的

仓库里没有任何本地脚本，更新完全交给 GitHub Actions：

| 工作流 | 触发时机 | 做什么 |
| --- | --- | --- |
| `excavator.yml` | 每 4 小时 | 对每个 manifest 跑 `scoop checkver`，发现新版本就开一个 PR |
| `issues.yml` | 新 issue，或给 issue 打 `verify` 标签 | 跑一遍 checkver，必要时直接修好 hash 并提交回仓库 |
| `pull_request.yml` | 新 PR，或 PR 下发 `/verify` 评论 | 校验 JSON 格式、必填字段、hash、checkver、autoupdate |
| `lint-pr-title.yml` | PR 打开 / 改名 / 推新提交 | 校验 PR 标题符合 scoop 命名约定 |

**Excavator 开的是 PR，不是直接改仓库。** 新版本要等你 review 完 diff（确认 `url` 和 `hash` 合理）再合并，才会进 bucket。这就是人工确认的那一步。

如果 GitHub 把仓库的 workflow 权限设成了只读，Excavator 会因为拿不到写权限而失败——去 `Settings` → `Actions` → `General` → `Workflow permissions` 确认没有锁死成只读。

## 添加软件

1. 复制模板：`bucket/app-name.json.template` → `bucket/<app-name>.json`
2. 填 `version` / `description` / `homepage` / `license`，以及 `architecture` 里对应架构的 `url`
3. 算 hash：

   ```powershell
   $url = 'https://example.com/app-1.2.3-x64.zip'
   Invoke-WebRequest $url -OutFile "$env:TEMP\dl"
   (Get-FileHash "$env:TEMP\dl" -Algorithm SHA256).Hash.ToLower()
   Remove-Item "$env:TEMP\dl"
   ```

4. 写 `checkver`，否则 Excavator 不知道去哪儿看新版本：

   ```json
   "checkver": "https://github.com/OWNER/REPO/releases/latest"
   ```

   闭源官网直链则指到页面并用正则抠版本号：

   ```json
   "checkver": {
     "url": "https://example.com/download",
     "regex": "v?(\\d+\\.\\d+(\\.\\d+)?)"
   }
   ```

   上游换下载链接导致 hash 对不上时，用 `.github/ISSUE_TEMPLATE/hash-error.yml` 开一个 issue，`issues.yml` 会自动把新 hash 补上。

manifest 的字段含义见 [App Manifests][manifests]，版本检测规则见 [App Manifest Autoupdate][autoupdate]。

[extras]: https://github.com/ScoopInstaller/Extras
[manifests]: https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests
[autoupdate]: https://github.com/ScoopInstaller/Scoop/wiki/App-Manifest-Autoupdate
