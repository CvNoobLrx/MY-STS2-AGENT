# STS2 AI Agent — 当前开发路线图

更新时间：`2026-03-10`

---

## 总体进度

| 阶段 | 描述 | 状态 |
|------|------|------|
| Phase 0A | 环境搭建 | ✅ 完成 |
| Phase 0B | 逆向侦察 | ✅ 完成 |
| Phase 1A | 协议冻结 | ✅ 完成 |
| Phase 1B | Mod 骨架 + `/health` | ✅ 完成 |
| Phase 1C | 最小纵切 | ✅ 完成 |
| Phase 2 | 战斗状态提取 | ✅ 完成 |
| Phase 3 | 战斗动作执行 | ✅ 完成 |
| Phase 4A | 地图 / 奖励 / 宝箱 | ⚡ 代码完成，部分待实机验证 |
| Phase 4B | 事件 / 休息点 | ❌ 未开始 |
| Phase 4C | 商店 | ❌ 未开始 |
| Phase 5 | MCP 完整化 | ⚡ 基础已就绪，随 4B/4C 同步扩展 |
| Phase 6 | 集成与回归 | ❌ 未开始 |

---

## 已实现功能清单

### C# Mod HTTP API

| 端点 | 状态 |
|------|------|
| `GET /health` | ✅ 已验证 |
| `GET /state` | ✅ 已验证 |
| `GET /actions/available` | ✅ 已验证 |
| `POST /action` | ✅ 已验证 |

### 动作

| 动作 | 实机验证 |
|------|---------|
| `end_turn` | ✅ |
| `play_card` | ✅ |
| `choose_map_node` | ✅ |
| `proceed` | ✅ |
| `collect_rewards_and_proceed` | ❓ 未验证 |
| `claim_reward` | ❓ 未验证 |
| `choose_reward_card` | ❓ 未验证 |
| `skip_reward_cards` | ❓ 未验证 |
| `select_deck_card` | ❓ 未验证 |

### 状态 Payload

| 字段 | 状态 |
|------|------|
| `combat.player` / `combat.hand` / `combat.enemies` | ✅ 已实现 |
| `run.deck` / `run.relics` / `run.potions` / `run.gold` | ✅ 已实现，❓ 未验证 |
| `map.current_node` / `map.available_nodes` | ✅ 已验证 |
| `reward.*` / `selection.*` | ✅ 已实现，❓ 未验证 |
| `event` | ❌ null 桩 |
| `shop` | ❌ null 桩 |
| `rest` | ❌ null 桩 |
| `game_over` | ❌ null 桩 |

### MCP 工具

13 个工具已注册，与 HTTP 端点 1:1 映射。随 Mod 侧新增动作同步扩展。

---

## 待完成工作与分工

### 分工原则

| 角色 | 定位 | 适合的任务类型 |
|------|------|----------------|
| **Codex** | 自主批量代码生成 | C# Mod 端的新功能实现（状态提取 + 动作执行） |
| **Claude Code** | 交互式协作 | Python MCP 端扩展、API 文档更新、代码审查、架构决策 |

**关键约定：双方不同时修改同一个文件。** Codex 负责 C# Mod 代码，Claude Code 负责 Python MCP + 文档。如需交叉操作，在本文档中标注并确认后再执行。

---

### 🔵 Codex 负责（C# Mod 端）

#### T-C1: 实机验证已实现功能

- 验证 `run.deck`、`run.relics`、`run.potions` 状态输出
- 验证 `claim_reward`、`choose_reward_card`、`skip_reward_cards`
- 验证 `select_deck_card` + `selection.cards`
- 验证 `collect_rewards_and_proceed`
- 输出验证结果，更新 `docs/phase-1c-status.md`

#### T-C2: Phase 4B — 事件系统（C# 侧）

- 实现 `EventPayload` 状态提取（事件 ID、标题、描述、选项列表）
- 实现 `choose_event_option` 动作
- 在 `GameStateService.BuildStatePayload()` 中接入，替换 `@event = null`
- 在 `GameActionService.ExecuteAsync()` 中注册新动作
- 在 `GameStateService.BuildAvailableActionNames()` 中注册可用性判断
- 涉及文件：`GameStateService.cs`、`GameActionService.cs`

#### T-C3: Phase 4B — 休息点系统（C# 侧）

- 实现 `RestPayload` 状态提取（可用选项：休息/升级/其他）
- 实现 `rest_site_action` 动作（休息回血、升级卡牌）
- 升级卡牌场景需要复用 `select_deck_card` 或新增子流程
- 在 `GameStateService.BuildStatePayload()` 中接入，替换 `rest = null`
- 涉及文件：`GameStateService.cs`、`GameActionService.cs`

#### T-C4: Phase 4C — 商店系统（C# 侧）

- 实现 `ShopPayload` 状态提取（在售卡牌、遗物、药水、删牌费用、玩家金币）
- 实现商店动作：`buy_card`、`buy_relic`、`buy_potion`、`remove_card_at_shop`
- 在 `GameStateService.BuildStatePayload()` 中接入，替换 `shop = null`
- 涉及文件：`GameStateService.cs`、`GameActionService.cs`

#### T-C5: GameOver 状态提取

- 实现 `GameOverPayload`（胜负、到达层数、角色、得分等）
- 在 `GameStateService.BuildStatePayload()` 中接入，替换 `game_over = null`
- 涉及文件：`GameStateService.cs`

#### T-C6: 重构 — Payload 类拆分（可选，低优先级）

- 将 `GameStateService.cs` 中 20+ 个 Payload 类拆分到独立文件
- 建议目录：`STS2AIAgent/Models/` 或 `STS2AIAgent/Game/Payloads/`
- 不改变业务逻辑，纯文件组织优化

---

### 🟢 Claude Code 负责（Python MCP 端 + 文档 + 审查）

#### T-M1: MCP 工具扩展 — 事件系统

- 前置：T-C2 完成后
- 在 `client.py` 中新增 `choose_event_option(option_index)` 方法
- 在 `server.py` 中注册 `choose_event_option` MCP 工具
- 补充 docstring：说明何时可用、参数含义

#### T-M2: MCP 工具扩展 — 休息点系统

- 前置：T-C3 完成后
- 在 `client.py` 中新增 `rest_site_action(action, option_index?)` 方法
- 在 `server.py` 中注册对应 MCP 工具
- 补充 docstring

#### T-M3: MCP 工具扩展 — 商店系统

- 前置：T-C4 完成后
- 在 `client.py` 中新增商店相关方法
- 在 `server.py` 中注册对应 MCP 工具
- 补充 docstring

#### T-M4: API 文档更新

- 随 T-C2/C3/C4/C5 完成后同步更新 `docs/api.md`
- 新增 Payload 结构示例
- 新增动作说明
- 保持协议文档与实现一致

#### T-M5: 代码审查

- 每当 Codex 完成一个 T-C 任务，审查代码质量
- 检查项：错误处理、线程安全、命名规范、与现有模式一致性
- 发现问题则反馈给用户决定是否修复

---

## 执行顺序

```
Phase 4A 验证
  T-C1 (Codex: 实机验证)
      │
      ▼
Phase 4B 事件 + 休息点
  T-C2 (Codex: 事件 C#) ──完成──▶ T-M1 (Claude Code: 事件 MCP)
  T-C3 (Codex: 休息点 C#) ──完成──▶ T-M2 (Claude Code: 休息点 MCP)
      │
      ▼
Phase 4C 商店
  T-C4 (Codex: 商店 C#) ──完成──▶ T-M3 (Claude Code: 商店 MCP)
      │
      ▼
补充
  T-C5 (Codex: GameOver)
  T-C6 (Codex: 重构，可选)
  T-M4 (Claude Code: 文档更新，持续)
  T-M5 (Claude Code: 代码审查，持续)
      │
      ▼
Phase 6 集成与回归
```

**并行规则：**
- T-C2 和 T-C3 可以并行（互不依赖）
- T-M1/M2/M3 各自依赖对应的 T-C 任务完成
- T-M4 和 T-M5 持续进行，不阻塞主线
- T-C4 建议在 T-C2/C3 之后开始（减少同时修改 GameStateService.cs 的风险）

---

## 文件修改权归属

| 文件 / 目录 | 归属 | 说明 |
|-------------|------|------|
| `STS2AIAgent/**/*.cs` | Codex | C# Mod 全部代码 |
| `mcp_server/src/sts2_mcp/*.py` | Claude Code | Python MCP 代码 |
| `docs/api.md` | Claude Code | 协议文档 |
| `docs/phase-1c-status.md` | Codex | 阶段验证记录 |
| `docs/roadmap-current.md` | 双方可更新 | 本文档，进度同步用 |
| `AGENTS.md` | 双方可更新 | 仓库指南 |

---

## 已知风险

1. **GameStateService.cs 并发修改** — T-C2/C3/C4 都需要修改此文件，顺序执行可避免冲突
2. **实机验证依赖** — 新功能需要游戏运行到对应界面才能测试，无法自动化
3. **STS2 版本更新** — 游戏更新可能破坏逆向依赖，需重新检查类型访问路径
4. **Payload 类膨胀** — 继续在 GameStateService.cs 中堆积类定义会加剧可读性问题
