[English](README.md) | [中文](README.zh.md)

# Ikunism

<p align="center"><a href="https://github.com/cuihairu/ikunism/tree/main/docs"><img src="https://img.shields.io/badge/%E6%96%87%E6%A1%A3-blue" alt="docs"></a>&nbsp;&nbsp;<a href="https://github.com/cuihairu/ikunism/releases"><img src="https://img.shields.io/badge/status-in%20development-orange" alt="status"></a></p> <a href="https://cuihairu.github.io/ikunism/zh/"><img src="https://img.shields.io/badge/%E6%96%87%E6%A1%A3-blue" alt="文档"></a>

一个以篮球和小鸡梗图为核心视觉的游戏 monorepo。当前方向:主线章节制叙事游戏《爱坤宗》(宗主被镇压、弟子化鸡、闯关收碎片、唤醒同门),平台策略为 Steam 买断制先行,微信小游戏和抖音小游戏后行(降敏 F2P 版,变现只考虑广告,不接支付)。

## 当前建议

- 游戏客户端引擎定稿 **Godot 4.x**（Steam 买断制先行；2026-10-04 引擎重估拍板，见 `docs/engine-selection.md` §0.1，Cocos 退出候选、Unity 留 3D 升级路径）。
- 后端零自建（2026-10-04 拍板）：玩家侧全走 Steamworks（榜单/成就/云存档）、内容侧全走 Cloudflare 免费层（R2/Workers/Pages），管理面=git+wrangler 发布配置；`apps/backend` 停建归档，见 `docs/gameplay-v3.md`「平台与商业化」。
- 美术资源先用 AI 生成概念图，再统一做裁切、压缩、尺寸适配，不要一开始就手工堆素材。

## 目录结构

```text
.
|-- apps
|   |-- backend          # Go 后端(排行榜/配置接口)— 2026-10-04 停建归档,仅参考
|   `-- game             # 游戏客户端(TS demo 脚手架 + Cocos 模板;引擎方向 Godot 4.x,尚未初始化)
|-- docs
|   `-- *.md             # 设计与审计文档 31 份(引擎/玩法/关卡/故事/叙事/资源/…)
|-- packages
|   |-- game-config      # 共享配置、数值、关卡、文案
|   `-- platform-sdk     # 微信/抖音平台能力封装
|-- tools
|   `-- art              # 提示词、批处理、压缩裁切脚本
|-- package.json
`-- pnpm-workspace.yaml
```

## 为什么这样拆

- `apps/game` 和 `apps/backend` 分开，避免你后面把游戏逻辑和服务端逻辑混在一起。
- `packages/platform-sdk` 负责屏蔽微信和抖音小游戏 API 差异，广告、登录、分享都从这里走。
- `packages/game-config` 放共享 JSON/TS 配置，数值和活动更容易维护。
- `tools/art` 单独管理提示词和图片处理脚本，后续换模型或批量出图不会污染业务代码。

## 下一步

1. 按 **Godot 4.x** 在 `apps/game` 初始化引擎工程(见 `docs/engine-selection.md` §0.1;Cocos 脚手架仅作参考)。
2. 先做一个最小玩法原型,例如“跑跳教学关 → 节奏模块”(见 `docs/gameplay-v3.md`)。
3. 确定一套小鸡视觉基准,再批量生成表情、皮肤、梗图素材。

## 开发文档

- 架构和选型见 `docs/architecture.md`
- 三引擎选型深研(Godot/Cocos/Unity,Steam 先行)见 `docs/engine-selection.md`
- 游戏工作室技能包分析见 `docs/game-studio-analysis.md`
- 图片工具和出图流程见 `docs/art-tools.md`
- 占位资源清单见 `docs/placeholder-assets-checklist.md`
- 资源命名规范见 `docs/asset-naming-conventions.md`
- AI 资源导入规则见 `docs/ai-asset-import-rules.md`
- Cocos 工程规划见 `docs/cocos-project-plan.md`
- Cocos 组件映射见 `docs/cocos-component-mapping.md`
- Gameplay 节点绑定见 `docs/gameplay-node-binding.md`
- Cocos 字段挂载清单见 `docs/cocos-field-checklist.md`
- Creator 组件壳见 `docs/creator-component-shells.md`
- Creator 接入手册见 `docs/creator-integration-playbook.md`
- Creator 文件落位图见 `docs/creator-file-map.md`
- Gameplay 反馈钩子见 `docs/gameplay-feedback-hooks.md`
- Home/Result 页面钩子见 `docs/home-result-hooks.md`
- 第一版玩法设计见 `docs/gameplay-v1.md`
- 第二版玩法设计(爱坤宗·四艺修行)见 `docs/gameplay-v2.md`
- 第三版玩法设计(主线章节制)见 `docs/gameplay-v3.md`
- 第一版(V1)范围清单(Steam Demo)见 `docs/v1-scope.md`
- 故事圣经(世界观/主线/七罪心魔/宽恕结局)见 `docs/story.md`
- 《爱坤宗:鸡年飞升》故事圣经见 `docs/story-bible.md`
- 《爱坤宗:碎梦重圆》叙事设计圣经见 `docs/narrative-bible.md`
- 《爱坤宗:鸡鸣破晓》故事圣经与叙事设计见 `docs/narrative-design.md`
- 坤拳武学设定集(爱坤宗/坤门宗)见 `docs/kunquan-system.md`
- 关卡与玩法设计(章节/曲目/结局挂点)见 `docs/levels.md`
- 《爱坤宗:鸡年飞升》关卡与玩法设计文档见 `docs/level-design.md`
- Demo 关卡细案(序章+第一章+第二章,配表)见 `docs/levels-demo-spec.md`
- 热梗语料收集与分析见 `docs/meme-collection.md`
- 平台合规分析见 `docs/compliance.md`

## 参考实现(可运行)

2026-10-04 决策后(引擎 = Godot 4.x;零自建服务器),以下均为参考级:它们跑的是 Cocos 脚手架 demo 与已归档的 Go 后端。Godot 引擎工程尚未初始化。

- 已归档 Go 后端:`go -C apps/backend run ./cmd/server`(`/health`、`/config/game`、`/leaderboard`、`POST /leaderboard/submit`)
- 游戏侧离线 demo:`pnpm --dir apps/game demo`
- 游戏侧在线 demo:`pnpm --dir apps/game demo:online`
- 游戏侧内存场景流 demo:`pnpm --dir apps/game demo:cocos-flow`
