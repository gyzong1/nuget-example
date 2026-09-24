# NuGet + JFrog CLI GitHub Actions 示例

基于测试项目 [gyzong1/nuget-example](https://github.com/gyzong1/nuget-example)，用 JFrog CLI 完成：

1. **构建并上传**至 Artifactory
2. **搜集并发布** Build Info
3. **扫描** Build（Xray）

示例工作流文件：`.github/workflows/nuget.yml`

## 使用方法

将 `nuget.yml` 复制到目标仓库：

```text
.github/workflows/nuget.yml
```

### Github 仓库配置


| 类型       | 名称                | 说明                                           |
| -------- | ----------------- | -------------------------------------------- |
| Variable | `JF_URL`          | JFrog Platform URL，如 `https://acme.jfrog.io` |
| Secret   | `JF_ACCESS_TOKEN` | 需具备 Deploy / Build Info / Xray Scan 权限       |


### Artifactory 仓库配置

流水线通过 `NUGET_REPO_RESOLVE` / `NUGET_REPO_DEPLOY` 指向 NuGet 仓库。请先在 Artifactory 中创建以下 **NuGet** 类型仓库：


| 类型      | 示例名称                         | 说明                                                               |
| ------- | ---------------------------- | ---------------------------------------------------------------- |
| Local   | `guoyz-github-nuget-local`   | 存放本流水线发布的 `.nupkg`                                                |
| Remote  | `guoyz-github-nuget-remote`  | 代理 nuget.org，URL 为 `https://www.nuget.org/`                      |
| Virtual | `guoyz-github-nuget-virtual` | 聚合上述 local + remote；**Default Deployment Repository** 指向对应 local |


说明：

流水线中使用了 build-scan, 需将相关仓库和 build 加入 **Xray Indexed Resources**，以便 `jf build-scan` 可扫描依赖与制品。

### 可调环境变量


| 变量                       | 默认值                          | 说明       |
| ------------------------ | ---------------------------- | -------- |
| `JFROG_CLI_BUILD_NAME`   | `guoyz-github-nuget-example` | Build 名称 |
| `JFROG_CLI_BUILD_NUMBER` | `${{ github.run_number }}`   | Build 编号 |
| `NUGET_REPO_RESOLVE`     | `guoyz-github-nuget-virtual` | 解析仓库     |
| `NUGET_REPO_DEPLOY`      | `guoyz-github-nuget-virtual` | 部署仓库     |


## 核心 JFrog CLI 步骤


| 步骤            | 命令                                                              |
| ------------- | --------------------------------------------------------------- |
| 配置 NuGet 仓库   | `jf nuget-config`                                               |
| 还原依赖并记录       | `jf nuget restore --build-name/--build-number`                  |
| 编译 / 打包       | `msbuild` + `nuget pack`                                        |
| 上传包并记录        | `jf rt upload "*.nupkg" <repo>/ --build-name/--build-number`    |
| 搜集环境信息        | `jf rt build-collect-env`                                       |
| 搜集 Git 信息     | `jf rt build-add-git`                                           |
| 发布 Build Info | `jf rt build-publish`                                           |
| 扫描 Build      | `jf build-scan`                                                 |

## 参考链接

- [安装 JFrog CLI](https://docs.jfrog.com/integrations/docs/download-and-install-the-jfrog-cli)
- [JFrog CLI 快速开始](https://docs.jfrog.com/integrations/docs/jfrog-cli-quick-start)
- [JFrog CLI 文档总览](https://docs.jfrog.com/integrations/docs/jfrog-cli)
- [jf nuget 命令说明](https://docs.jfrog.com/artifactory/docs/jf-nuget)
- [jf dotnet 命令说明](https://docs.jfrog.com/artifactory/docs/jf-dotnet)
