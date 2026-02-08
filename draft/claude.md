#ai 
## 一、基础操作与快捷键

### 核心快捷键

| 快捷键 | 功能 | 说明 |
|--------|------|------|
| `Ctrl+C` | 取消/中断 | 取消当前生成或退出 Claude Code |
| `Ctrl+D` | 退出 | 完全退出 Claude Code |
| `Ctrl+L` | 清屏 | 清空终端屏幕 |
| `Tab` | 自动补全 | 补全命令或文件路径 |
| `↑/↓` | 历史记录 | 浏览之前的命令 |
| `Shift+Enter` | 换行 | 多行输入（需先运行 `/terminal-setup` 配置） |
| `Esc` | 打断 | 立即停止当前工作 |
| `Esc + Esc` | 回退 | 撤销上一步操作（可选择仅回退对话或代码） |
| `Ctrl+R` | 搜索历史 | 反向搜索之前的提示词 |
| `Ctrl+S` | 暂存 | 暂存当前提示词，稍后恢复 |
| `Shift+Tab` (按两次) | Plan 模式 | 进入"先规划后执行"模式 |

### macOS 特有
- `Opt+Enter`：换行
- `Cmd+K`：清空对话
- `Ctrl+V`：粘贴图片（注意：不是 Cmd+V）

### Windows/Linux 特有
- `Alt+P`：切换模型
- `Ctrl+K`：清空对话

---

## 二、斜杠命令大全

在 Claude Code 会话中输入 `/` 可查看所有可用命令。

### 常用命令

| 命令         | 功能                   | 使用频率  |
| ---------- | -------------------- | ----- |
| `/clear`   | 清空对话历史               | ⭐⭐⭐⭐⭐ |
| `/compact` | 压缩对话（保留摘要，减少 Token）  | ⭐⭐⭐⭐⭐ |
| `/context` | 可视化当前上下文使用情况         | ⭐⭐⭐⭐⭐ |
| `/cost`    | 查看费用统计               | ⭐⭐⭐⭐  |
| `/model`   | 切换 AI 模型             | ⭐⭐⭐⭐  |
| `/init`    | 生成 CLAUDE. md 项目记忆文件 | ⭐⭐⭐⭐  |
| `/add`     | 添加文件到上下文             | ⭐⭐⭐⭐  |
| `/add-dir` | 添加目录到上下文             | ⭐⭐⭐⭐  |
| `/skills`  | 查看可用技能包              | ⭐⭐⭐   |
| `/mcp`     | 管理 MCP 服务器           | ⭐⭐⭐   |
| `/hooks`   | 管理自动化钩子              | ⭐⭐    |
| `/agents`  | 管理子代理                | ⭐⭐    |
| `/doctor`  | 诊断环境配置               | ⭐⭐⭐⭐  |
| `/vim`     | 切换 Vim 模式            | ⭐⭐    |
| `/export`  | 导出对话记录               | ⭐⭐⭐   |
| `/status`  | 查看系统状态               | ⭐⭐⭐   |

---

## 三、代码分析与阅读技巧

### 1. 文件引用 (@ 提及)

这是最核心的代码分析功能，可以精确控制 Claude 的上下文。

```bash
# 引用单个文件
@src/auth.ts 请分析这个认证模块的安全性

# 引用整个目录（递归分析）
@src/components/ 检查所有组件的性能问题

# 引用多个文件对比
@src/old-implementation.ts @src/new-implementation.ts 比较这两个实现的优劣

# 使用通配符
@src/**/*.test.ts 检查所有测试文件的覆盖率
```

**技巧**：Claude 会自动模糊匹配，输入 `@auth` 可能匹配到 `auth.ts`、`auth.controller.ts` 等。

### 2. 使用 Plan 模式进行深度分析

**Plan 模式**（按两次 `Shift+Tab` 进入）是分析复杂代码库的最佳实践：

```
# 进入 Plan 模式后输入：
请分析这个项目的架构设计，包括：
1. 模块依赖关系
2. 数据流设计
3. 潜在的性能瓶颈
4. 安全漏洞风险
```

Claude 会先**只读不写**，探索代码库结构，生成详细分析报告，等你确认后再执行具体修改。

### 3. 深度思考模式

在提示词中加入特定关键词激活不同级别的思考：

- `think` - 基础思考
- `think hard` - 深入思考
- `think harder` - 深度分析
- `ultrathink` - 极限思考（最耗 Token，适合复杂架构分析）

示例：
```bash
请 ultrathink 分析这段代码的并发安全问题
```

### 4. 结合 Shell 命令 (! 前缀)

```bash
# 在对话中直接执行命令获取上下文
!git log --oneline -20  查看最近提交历史
!find . -name "*.ts" | head -20  列出所有 TypeScript 文件
!npm test 2>&1 | head -50  查看测试错误输出

# 组合使用
分析 @src/ 目录，先执行 !cloc src/ 统计代码行数，然后找出最复杂的模块
```

---

## 四、项目初始化与记忆管理

### 1. 生成 CLAUDE. md (`/init`)

```bash
# 自动生成项目记忆文件
/init

# 或带描述生成
claude /init "这是一个 Node.js + React 的微服务项目，使用 TypeScript 和 PostgreSQL"
```

CLAUDE. md 文件会记录项目架构、编码规范、常用命令等，Claude 会自动读取并遵循。

### 2. 快速添加记忆 (# 前缀)

在对话中输入以 `#` 开头的句子，会自动追加到 CLAUDE. md：

```bash
# 使用 Redis 缓存用户会话数据
```

这行会被自动记录到记忆文件中。

### 3. 查看和编辑记忆

```bash
/memory  # 查看当前记忆状态
```

---

## 五、高级工作流技巧

### 1. 免授权模式（狂飙模式）

如果你厌倦了频繁的权限确认：

```bash
claude --dangerously-skip-permissions
```

启动后底部会显示黄色 `Bypassing Permissions` 提示，所有操作无需确认直接执行。**谨慎使用！**

建议设置别名：
```bash
alias cc='claude --dangerously-skip-permissions'
```

### 2. 会话管理

```bash
# 意外关闭后恢复
claude --continue

# 查看所有会话并选择恢复
claude --resume

# 给会话命名（在会话内）
/rename feature-auth-impl

# 按名称恢复
claude --resume feature-auth-impl

# 跨设备同步（网页版与终端同步）
claude --teleport <session_id>
```

### 3. 子代理 (Subagents)

对于大型项目，可以创建专门的子代理并行处理：

```bash
/agents  # 管理子代理

# 示例：创建专门处理测试的代理
# 然后可以并行执行：
@src/auth.ts 修复登录bug（主会话）
@tests/auth.test.ts 同步更新测试（子代理）
```

### 4. 自定义斜杠命令

创建个人或项目级命令：

```bash
# 项目级命令（仅当前项目可用）
mkdir -p .claude/commands
echo "检查代码中的内存泄漏风险：" > .claude/commands/memory-check.md

# 个人级命令（所有项目可用）
mkdir -p ~/.claude/commands
echo "生成详细的代码审查报告：" > ~/.claude/commands/cr.md

# 使用
/memory-check
/cr
```

带参数的命令：
```bash
echo '修复 Issue #$ARGUMENTS，遵循项目代码规范' > .claude/commands/fix.md

# 使用
/fix 123
```

---

## 六、Skills 技能包系统

Skills 是预封装的工作流，可以扩展 Claude Code 的能力。

### 安装官方 Skills

```bash
# 前端设计
npx skills-installer install @anthropics/claude-code/frontend-design --client claude-code

# PDF 处理
npx skills-installer install @anthropics/claude-code/pdf --client claude-code

# 文档协同
npx skills-installer install @anthropics/claude-code/doc-coauthoring --client claude-code
```

### 使用 Skills

```bash
/skills  # 查看已安装的 Skills

# 在对话中调用
使用 pdf skill 提取 report.pdf 中的表格数据
使用 frontend-design skill 优化这个页面的响应式布局
```

---

## 七、Hooks 自动化钩子

Hooks 可以在特定事件后自动执行命令。

### 常用 Hook 配置（在 `~/.claude/settings.json` 中）

```json
{
  "hooks": {
    "after-tool-use-hook": {
      "command": "bun run format || true",
      "enabled": true,
      "blocking": false
    },
    "before-tool-use-hook": {
      "command": "if [[ {{toolName}} == 'rm' ]]; then echo '危险操作！'; exit 1; fi",
      "enabled": true
    }
  }
}
```

**场景示例**：
- 自动格式化代码（保存后运行 Prettier）
- 危险操作拦截（阻止 `rm -rf` 等命令）
- 自动提交（代码修改后自动 git commit）

---

## 八、实战：代码分析工作流

### 场景 1：接手新项目

```bash
# 1. 启动并初始化
claude
/init "这是一个 Python FastAPI 项目"

# 2. 整体架构分析（Plan 模式）
Shift+Tab × 2
请分析整个项目的架构：
- 目录结构是否合理
- 依赖关系图
- 核心业务逻辑位置
- 潜在的技术债务

# 3. 深入关键模块
@src/core/ ultrathink 分析这个核心模块的设计模式

# 4. 检查测试覆盖
@tests/ 检查测试覆盖率，找出未覆盖的关键路径
```

### 场景 2：Code Review

```bash
# 1. 查看变更
!git diff HEAD~5 > /tmp/changes.patch
@/tmp/changes.patch 请 review 这些变更

# 2. 检查特定文件
@src/payment/ 重点检查支付模块的安全性和异常处理

# 3. 生成审查报告
创建详细的代码审查报告，包括：
- 代码质量问题（按严重程度分级）
- 性能优化建议
- 安全风险提醒
- 重构建议
```

### 场景 3：Bug 分析

```bash
# 1. 粘贴错误截图（Ctrl+V）
这是生产环境的错误截图，请分析原因

# 2. 定位相关代码
@src/utils/error-handler.ts 检查错误处理逻辑

# 3. 追踪调用链
!grep -r "functionName" src/ --include="*.ts"
分析这些调用点，找出 null pointer 的根源

# 4. 提供修复方案
think harder 提供三种修复方案，并说明各自的优缺点
```

---

## 九、性能优化建议

### Token 管理

```bash
# 定期清理上下文
/compact "保留项目架构和当前任务要点"

# 新任务开始时
/clear

# 监控使用情况
/context  # 查看上下文占用百分比
```

### 多实例并行

```bash
# 终端 1：处理前端
claude
分析 @frontend/src/components/

# 终端 2：处理后端（同时运行）
claude
分析 @backend/src/api/

# 终端 3：处理测试（同时运行）
claude
生成 @backend/tests/ 的单元测试
```

---

## 十、快速参考卡

```bash
# 启动
claude                    # 交互模式
claude "prompt"           # 直接执行
claude -p "prompt"        # 打印模式（非交互）
claude --continue         # 恢复上次会话

# 文件操作
@file.ts                  # 引用文件
@src/                     # 引用目录
!command                  # 执行 Shell

# 模式切换
Shift+Tab                 # 自动编辑模式（无需确认）
Shift+Tab × 2             # Plan 模式（先规划后执行）

# 实用命令
/clear                    # 清空对话
/compact                  # 压缩对话
/context                  # 查看上下文
/cost                     # 查看费用
/model                    # 切换模型
/doctor                   # 诊断环境
```

Claude Code 的设计理念是**大道至简**——你不需要记住复杂的指令，像和同事交流一样自然地描述需求即可。掌握这些技巧后，你的编程效率会有质的飞跃！