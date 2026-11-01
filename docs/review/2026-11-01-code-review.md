# 代码深审报告(ikunism,2026-11-01)

> 方法:按 `code-review` 技能流程执行——每条发现先自行取证(读源码逐行核对)再入表,禁止未验证主张;每条带 file:line 证据;按严重度排序。
> 范围:全仓(Go 后端 `apps/backend`、TS 前端 `apps/game`、`packages/game-config`、`packages/platform-sdk`、`tools/art`)。测试与 `dist/` 产物不属审查对象(仅作基线)。
> 结论:**无 P0/P1(无安全漏洞、无崩溃/数据丢失类缺陷),故当场零修复;P2×2、P3×5 记 `docs/todo.md`。**

## 一、安全静态扫描(①)

| 类别 | 命中 | 证据/说明 |
|------|------|-----------|
| 硬编码密钥 | 0 | 全仓 grep 无 API key / token / 私钥 / 口令字面量;仅 `platform-sdk/src/index.ts:18` `token?: string` 为接口字段类型,非值 |
| shell 注入 | 0 | 生产代码无 `os/exec`、`child_process`、`spawn`、`system`;`cmd/server/main_test.go:87` 的 `exec.Command` 为自测模式(测试代码),`grep` 命中的其余为模板字符串显示(`Score: ${...}`)非执行 |
| SQL 拼接 | 0 | `leaderboard/store.go` 全部走 gorm 参数化(`Order`/`Limit`/`Create`/`Find`),无 `Raw`/`Exec`/`fmt.Sprintf` 拼 SQL;`Order("score DESC")` 为固定字面量 |
| eval / Function / 动态 require | 0 | 全仓无 `eval`、`new Function`、`vm.runIn`、动态 import |
| 反序列化 | 安全 | `config/gameplay.go:81`、`server.go:84`(`ShouldBindJSON`)均绑定到类型化结构体,无 `interface{}`/任意实例化;TS 侧 `JSON.parse` 无 |
| 路径穿越 | 0 | 后端无文件服务端点、无用户入参路径;`sqlite.go:31-48` 路径来自 env/`runtime.Caller` 固定推导;`gameplay.go:63-73` 为仓库内固定相对路径。`tools/art` 见 B7 |
| 认证/限流 | 按设计 | `/leaderboard/submit` 无鉴权——`docs/gameplay-v3.md`「平台与商业化」已明确「零自建服务器、客户端上报可容忍」;后端 2026-10-04 已停建归档 |

安全面**零漏洞**。唯一值得记录的是 `leaderboard/submit` 无长度上限(见 B2,P2)。

## 二、隐藏 bug 猎捕(②)

### P2(记 todo)

**B1 — 回合结束后分数仍可被改动并回流到提交分**
- 证据:`apps/game/assets/scripts/gameplay/gameplay-session.ts:24-33`——`addHit`/`addMiss` 不检查 `state.finished`,计时归零后仍累加分数/重置连击;`apps/game/assets/scripts/ui/gameplay-scene-controller.ts:23-31`——`tick` 在 `snapshot.finished` 时**每帧**重复调用 `gameManager.finishRound(snapshot.score)`(`game-manager.ts:23-26`→`applyScore`)。
- 触发:倒计时结束(剩余 0s)后、场景切走前,玩家再按一次投篮键(`gameplay-scene.ts:41-56` 仍在绑定)→ `onHit`→`addHit` 加分 → 下一帧 `finishRound(新分)` 覆盖 `currentScore`/`bestScore` → Result 页提交的 `currentScore`(result-scene-controller.ts:15-20)被抬高。
- 影响:提交分可含超时后分数(玩法完整性);demo/参考级代码,需超时后操作才触发。
- 修法:在 `addHit`/`addMiss` 开头 `if (this.state.finished) return this.snapshot;`;`GameplaySceneController.tick` 加 once 守卫(`finishRound` 只触发一次)。

**B2 — `/leaderboard/submit` 入参无长度/上限校验**
- 证据:`apps/backend/internal/server/server.go:82-108`——仅校验非空与 `Score < 0`,无 `playerName`/`playerId` 长度上限、无 `Score` 上限;未鉴权端点。
- 影响:超大字符串入库/回显,数据库与响应膨胀(轻度 DoS 面);分数可写 `MaxInt`。
- 修法:`playerName`/`playerId` 限长(如 64/64)、`Score` 加上限;或标注"后端已归档,仅在复用前补"。
- 注:后端为已归档参考实现,文档已声明客户端上报可容忍——若属生产代码应升 P1。

### P3(记 todo)

**B3 — Creator 模板 `update` 每帧调用 `finishRound`(无 once 守卫)**
- 证据:`apps/game/creator-templates/GameplaySceneComponent.ts:63-65`(`if (this.remainingSeconds <= 0) this.finishRound();`)+ `:157-161`(`finishRound` 为 TODO 桩)。
- 影响:桩实现后(场景跳转/保存)将每帧重复触发。模板级潜在缺陷。

**B4 — perfect/hit 判定窗口硬编码,与 `ShotInputController`/配表重复**
- 证据:`apps/game/creator-templates/GameplaySceneComponent.ts:87-93`(`0.45~0.65`/`0.2~0.85` 硬编码),而 `shot-input.ts:18-22` 默认值与此重复、`gameplay.v1.json` 另有配表口径。
- 影响:改配表/默认窗口时模板漂移。改为引用统一常量或配置。

**B5 — `HomeSceneComponent.loadHomeData` 的 `catch` 不可达(死代码)**
- 证据:`apps/game/creator-templates/HomeSceneComponent.ts:38-49`——`try` 块全同步(无 `await`),无法抛错,`catch`/`renderErrorState` 永不可达。
- 影响:模板误导;实际加载失败不会走错误分支。加真实 `await` 或删 `try/catch`。

**B6 — `SceneFlow`、`PlatformBridge` 孤儿类(导出但零引用)**
- 证据:`apps/game/assets/scripts/core/scene-flow.ts`、`apps/game/assets/scripts/platform/platform-bridge.ts`——全仓仅 `dist/` 声明文件提及,无源码 import。
- 影响:死代码。删除或纳入公共导出。

**B7 — 素材清单脚本未将目标路径约束在素材根内**
- 证据:`tools/art/scripts/rename-from-manifest.ps1:16-35`——`Join-Path $artRoot $item.target` 后直接 `Copy-Item`,无 `Resolve-Path`/根内校验;清单条目含 `..` 可写出素材树。
- 影响:本地一次性脚本,清单为可信仓库文件,风险低;作加固项。

## 三、质量门基线(③)

| 检查 | 结果 |
|------|------|
| `go -C apps/backend test -race ./...` | PASS(5 包,无数据竞态) |
| `pnpm -r test`(vitest) | PASS(3 包:game-config 88、platform-sdk 20、apps/game 74 = **182 测试**) |
| `pnpm typecheck` | PASS(3 包) |
| `pnpm lint`(game 包=tsc --noEmit) | PASS |
| `pnpm build`(tsc 全仓) | PASS |

基线全绿。测试覆盖率:Go 5 包测试、TS 182 用例均覆盖核心逻辑(shot-input/gameplay-session/cultivation/medals/http-api/platform)。

## 四、发现汇总(⑥)

| 严重度 | 数量 | 明细 | 处置 |
|--------|------|------|------|
| P0 | 0 | — | — |
| P1 | 0 | — | — |
| P2 | 2 | B1、B2 | 记 `docs/todo.md` |
| P3 | 5 | B3、B4、B5、B6、B7 | 记 `docs/todo.md` |

- **发现数 7 / 已修数 0 / 剩余数 7**(无 P0/P1,故当场零修复;P2/P3 全部转 todo)。
- **commit 清单**:本会话相关提交为 `1bd45ec docs: add incremental recheck to doc audit`(审计复核);**本审无修复 commit**(无 P0/P1 可修)。`docs/audit` 与 `docs/review` 提交见本报告落库 commit。
- 复核:每条发现均经源码逐行核对(V 已确认),未采纳任何未验证主张。
