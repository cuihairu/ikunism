# Creator 组件壳说明

## 目标

提供一组可以直接复制进 `Cocos Creator` 的真实组件壳代码，减少你从抽象模板切到正式组件时的改写量。

## 位置

组件壳放在：

- [README.md](../apps/game/creator-templates/README.md)
- [BootSceneComponent.ts](../apps/game/creator-templates/BootSceneComponent.ts)
- [HomeSceneComponent.ts](../apps/game/creator-templates/HomeSceneComponent.ts)
- [GameplaySceneComponent.ts](../apps/game/creator-templates/GameplaySceneComponent.ts)
- [ResultSceneComponent.ts](../apps/game/creator-templates/ResultSceneComponent.ts)

## 为什么单独放目录

这些文件依赖 `cc` 模块。

当前 monorepo 还没有真正初始化 Creator 工程，所以它们不能直接参与现有 TypeScript 构建。单独放在 `creator-templates` 下，后续复制到 Creator 管理的 `assets/scripts` 最稳。

## 使用顺序

1. 在 Creator 里建四个场景。
2. 把对应组件壳复制到 Creator 项目里。
3. 按 [docs/cocos-field-checklist.md](cocos-field-checklist.md) 挂好字段。
4. 再逐步把仓库里的业务层接进来：
   - `assets/scripts/core`
   - `assets/scripts/gameplay`
   - `assets/scripts/ui`
   - `src/services`

## 当前状态

这批组件壳已经包含：

- `@property` 字段
- 生命周期位置
- 按钮与触摸事件注册位置
- TODO 标记，指向下一步要接的业务逻辑

它们的作用是“正式接入壳”，不是最终完成态。
