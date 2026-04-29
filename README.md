# nanobot（a tiny change）
---

## 一、原项目的 Memory 写入机制

以下内容描述上游仓库的设计（未使用本仓库补丁时的典型行为），便于与第二节区分。

### 1. 分层结构

1. **会话层（短期）**  
   当前对话保存在会话数据（`session.messages`）中，随上下文窗口滚动。

2. **Consolidator → `memory/history.jsonl`**  
   对话过长时，`Consolidator` 将较早片段**摘要后追加**到 `memory/history.jsonl`。  
   - **JSONL**，每行一条，含 **cursor**，便于增量读取与 Dream 分批处理。  
   - 这是压缩后的「原始材料」，不等于最终整理好的长期档案。

3. **Dream → 三个长期 Markdown 文件（上游默认）**  
   **Dream**（定时 `/dream` 或手动触发）会读取：
   - `history.jsonl` 中尚未被 Dream 处理过的条目；
   - 当前的 **`SOUL.md`**（助手人设与表述）、**`USER.md`**（用户画像与偏好）、**`memory/MEMORY.md`**（项目事实与上下文）。

   分两阶段：**Phase 1** 分析沉淀与去重；**Phase 2** 通过工具对上述文件做**增量编辑**，也可按需新建 `skills/` 下的 SKILL。Dream 的 Prompt 设计中将三类文件并列作为可持续写入的长期知识载体。

4. **GitStore（上游默认）**  
   若在工作区初始化 nanobot 自带的 git，通常会把 **`SOUL.md`、`USER.md`、`memory/MEMORY.md`** 一并纳入跟踪与自动提交，便于 `/dream-log`、`/dream-restore` 比对与回滚。

5. **运行时进入模型上下文**  
   **`ContextBuilder`** 组装系统提示：身份与路径、`AGENTS.md` / **`SOUL.md` / `USER.md`** / `TOOLS.md` 等 **bootstrap**（文件存在则加载），以及非模板占位时的 **`memory/MEMORY.md`**、`always` skill、未消费的近期 history 片段等。

上游更完整的英文说明见 [`docs/memory.md`](docs/memory.md)（若与本仓库描述冲突，以第二节「修改后」为准）。

---

## 二、本仓库的修改：修改前后对比

本节说明：**在同一套 Consolidator / Dream / GitStore / ContextBuilder 骨架下**，本仓库相对上游做了哪些收敛与约束。

### 2.1 对照总表

| 维度 | 修改前（上游默认） | 修改后（本仓库） |
|------|-------------------|------------------|
| **Dream 维护的长期文件** | `SOUL.md`、`USER.md`、`memory/MEMORY.md` 均可被 Dream 更新 | 仅 **`memory/MEMORY.md`**；Dream 不再写入 **`SOUL.md` / `USER.md`** |
| **Dream Phase 1 上下文** | 同时预览三类文件内容（及对 MEMORY 的行龄标注策略） | 仅预览 **`memory/MEMORY.md`**（行龄标注仍仅针对该文件） |
| **GitStore 跟踪文件** | `SOUL.md`、`USER.md`、`memory/MEMORY.md` | 仅 **`memory/MEMORY.md`** |
| **主对话 / 子代理的文件工具** | `edit_file` / `write_file` 可改写根目录 `SOUL.md`、`USER.md` | 对 **`SOUL.md`、`USER.md`** 使用 **拒绝列表**，引导写入 **`memory/MEMORY.md`** |
| **系统提示 bootstrap** | 默认注入 **`USER.md`**（若存在） | **不再默认注入 `USER.md`**；仍可选择加载 **`SOUL.md`**（若存在） |
| **模板与内置说明** | Dream 模板区分 USER / SOUL / MEMORY 职责 | `dream_phase1` / `dream_phase2`、skill、`templates/USER.md` 等与「长期记忆仅 MEMORY.md」对齐 |

### 2.2 截图示例（修改前 → 修改后）

**修改前（上游行为）：** 助手仍可能将「保存到记忆」落到 **`USER.md`**，并提及由 Dream 管理——与本仓库「长期事实统一进 **`memory/MEMORY.md`**」不一致：

![修改前：助手回复指向 USER.md](./images/截屏2026-04-29%2021.19.41.png)

对应工作区中也会出现写入 **`USER.md`** 的条目：

![修改前：工作区中的 USER.md](./images/截屏2026-04-29%2021.20.13.png)

**修改后（本仓库）：** 结构化长期事实落在 **`memory/MEMORY.md`**（Dream / git 亦仅围绕该文件）；交互中可将多条用户事实并入「长期记忆」并最终反映在该文件中：

![修改后：会话中确认写入长期记忆](./images/截屏2026-04-29%2021.44.40.png)

![修改后：memory/MEMORY.md 内容示例](./images/截屏2026-04-29%2021.44.25.png)
