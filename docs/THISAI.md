# ThisAI 定制版本维护

## 仓库与分支

- `origin`：`lqy007700/sub2api-KlN`，自己的代码发布仓库。
- `community`：`KlN-4096/sub2api`，社区二开更新来源。
- `upstream`：`Wei-Shaw/sub2api`，保留官方源用于比较。
- `klno`：社区二开基线，跟踪 `community/klno`，不加入 ThisAI 定制。
- `codex/thisai-branding`：在社区基线上加入 ThisAI 图标、蓝色主题、登录页面和品牌文案，服务器部署使用这个分支。

当前定制基于社区 `v0.2.14-klno.3`，社区提交为
`de08df02ae1d81668a22f798b398aa0438ac1276`。

数据库中已有的站点名称、副标题、Logo 和首页内容会优先于代码默认值。
更新代码不会自动覆盖这些设置。既有部署如果仍显示旧副标题，应通过站点设置修改。

原 `sync-upstream.yml` 是社区维护者直接同步官方、重建分支和发布版本的工作流。
本仓库默认分支和定制分支已限制它只在原社区仓库运行。
不要在纯社区镜像 `klno` 分支上手动运行这个工作流。
这里配置的是明确的 Git 更新来源，没有启用自动同步或自动生产部署。

## 获取社区更新并保留 UI 定制

先提交或暂存自己的改动，下面的操作要求工作区干净。
社区维护者可能重写历史，所以必须先保存旧基线，再只重放自己的提交。

```bash
set -e
test -z "$(git status --porcelain)"

thisai_old_base=$(git rev-parse klno)
thisai_backup_id=$(date -u +%Y%m%dT%H%M%SZ)
git branch "codex/backup-thisai-${thisai_backup_id}" codex/thisai-branding
git branch "codex/backup-klno-${thisai_backup_id}" klno

git fetch community klno --tags
git rebase --onto community/klno "$thisai_old_base" codex/thisai-branding
```

有冲突时检查冲突文件、同时保留社区逻辑与 ThisAI 定制，再执行
`git rebase --continue`；需要撤回这次重放时执行 `git rebase --abort`。
不要把发生历史重写的整个社区分支直接合并进定制分支。

验证更新后的代码：

```bash
(cd frontend && pnpm install --frozen-lockfile && pnpm run build && pnpm exec vitest run src/router/__tests__/title.spec.ts)
(cd backend && GOSUMDB=sum.golang.org go test -tags embed ./internal/web -count=1)
(cd backend && GOSUMDB=sum.golang.org go test -tags unit ./internal/service -run 'Setting|Email|BalanceNotify|UserService|TurnState|OutboundUserAgent' -count=1)
git diff --check
```

测试通过后再更新自己的 GitHub 分支。推送前先备份远端原有提交，
并用明确的 `--force-with-lease` 防止覆盖其他人刚刚推送的内容。

```bash
git fetch origin
thisai_remote_klno=$(git rev-parse refs/remotes/origin/klno)
thisai_remote_ui=$(git rev-parse refs/remotes/origin/codex/thisai-branding)

git push origin \
  "${thisai_remote_klno}:refs/tags/archive/klno-${thisai_backup_id}" \
  "${thisai_remote_ui}:refs/tags/archive/thisai-${thisai_backup_id}"

git branch -f klno community/klno
git push --atomic \
  "--force-with-lease=refs/heads/klno:${thisai_remote_klno}" \
  "--force-with-lease=refs/heads/codex/thisai-branding:${thisai_remote_ui}" \
  origin \
  klno:refs/heads/klno \
  codex/thisai-branding:refs/heads/codex/thisai-branding
```

本地 `klno` 继续跟踪 `community/klno`；定制分支跟踪
`origin/codex/thisai-branding`。`remote.pushDefault=origin` 将默认推送目标设为自己的仓库。

## 在服务器构建改版 UI 镜像

在独立源码目录取自己的定制分支：

```bash
git clone --branch codex/thisai-branding \
  https://github.com/lqy007700/sub2api-KlN.git sub2api-thisai-src
cd sub2api-thisai-src

thisai_build_sha=$(git rev-parse HEAD)
thisai_base_version=$(git describe --tags --always origin/klno)
docker build \
  --build-arg "VERSION=${thisai_base_version}-thisai.${thisai_build_sha:0:12}" \
  --build-arg "COMMIT=${thisai_build_sha}" \
  --build-arg GOPROXY=https://proxy.golang.org,direct \
  --build-arg GOSUMDB=sum.golang.org \
  --label org.opencontainers.image.source=https://github.com/lqy007700/sub2api-KlN \
  -t "sub2api-thisai:${thisai_build_sha:0:12}" .
```

需要 Bash 运行上述构建命令。仓库根目录的 Dockerfile 会构建 Vue 前端，
并通过 Go 的 `embed` 把改版 UI 编入应用；仅修改后台站点设置无法替代这一步。

构建镜像不会更换正在运行的容器。上线时复用生产上的
`/opt/sub2api-klno/docker-compose.local.yml`，只把应用服务 `sub2api-klno`
的镜像切到本次构建的固定提交标签；保留数据库、Redis、端口、卷和网络配置。
目标应用更新应使用 `--no-deps --pull never`，不要用源码仓库中的整套 Compose
替换现有生产编排。先保存应用原镜像和编排配置，再验证健康检查、页面资源及其他容器状态。
