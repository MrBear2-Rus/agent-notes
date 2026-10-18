# DeepSeek Harness 使用笔记

## 任务 1 · 我用 Agent 做了什么

**1. 使用了什么 Agent**

DeepSeek Harness 

**2. 安装和配置过程**

- 安装：Windows 桌面版，配置Deepseek的api

**3. 我交给了它什么任务**

比如辅助我写代码，看论文和资料等等。比如我交给它了一些在Linux(WSL)上的一些指令的查询工作，然后它帮我查到了。再有就是一些检查代码的工作，比如检查为什么我的指令运行不了，检查判定程序为什么没有给我过之类的。（点名Linux和CMake题的md判断，一定要删了题目才能判对）

**4. Agent 做了哪些修改**

做了很多，但目前他还没有做什么代码性的工作，我只是问了一些技术性问题

**5. git diff 里我看到了什么**

看到了AI试图改判题的程序，然后被我制止了（

**6. 最终结果是否符合预期**

- 基本符合：审计结论可直接用；脚本我在 WSL 的临时目录里自检后来通过了。

## 任务 2 · 五个问题

1. **为什么 Agent 能够读取文件、修改代码，而普通聊天 AI 通常不能？**

   因为 Agent = 模型 + 运行时：模型可以输出「工具调用」，由宿主在真实文件系统上执行、再把结果回灌进下一轮。聊天 AI 只有文本进出，没有能落到磁盘的执行通道。

2. **Tool 在 Agent 中起到了什么作用？**
   把模型的意图翻译成对真实世界的确定动作（读 / 写 / 执行 / 检索）。它同时是权限与审计的落点：Agent 能做什么，取决于工具白名单，而不是模型想做什么。

3. **为什么项目需要给 Agent 配置一份类似"员工手册"的规则？**
   划定边界（哪些文件能改、哪些命令禁跑）和流程（先读再改、改完自检），避免误删误改。

4. **Agent 为什么可能"忘记"之前说过的内容？Agent 的额度怎么样计算？**
   上下文窗口有限，超出后早期对话会被截断或摘要。Agent的额度在服务器端就会统计，dsh也有统计的

5. **如果一个 Agent 可以随便执行任何终端命令，会有什么风险？**
   不可逆破坏（删库、覆盖、以及经典的sudo rm -rf）、数据外泄、装上恶意依赖。

## 任务 3（可选）· MCP / Skill / Tools 的查找与配置记录（我发现Deepseek可以自己总结，所以这一章是它自己找完的，我没亲自找:)）

- **rea（MCP，本地逆向分析）**：`cordis.patch.yml` 里一条 `mcp-client` 条目，`transport: stdio`，用 `node` 直接跑 `rea.mjs mcp`（配置注释写明：Windows 上 `child_process.spawn` 执行不了 `.cmd`，所以不用 npx）。环境变量只作用于这个子进程：`GHIDRA_INSTALL_DIR`（Ghidra 12.1.4）、`JAVA_HOME`（JDK 25）、`REA_GHIDRA_STARTUP_TIMEOUT_MS=3600000`（大体积静态链接程序会超过默认 5.5 分钟的分析 deadline）。工具以 `mcp__rea__<工具名>` 暴露给模型。
- **Kling（MCP，图像 / 视频生成）**：实测调用 `who_am_i` 成功 —— OAuth 已授权，`mcpVersion 1.3.3`，可用模型含 `kling-image-v3_0_omni`、`kling-video-v4_0`、`motion_control` 等；返回值同时提示「工具列表已更新」，需要重启宿主刷新连接器（已转告我）。
- **Skill / 预设**：本机装了 `dsh-code-review` 插件，它在宿主启动时声明一个 agent preset「**代码审查模式**」（只读审查，输出 critical / high / medium / low / nit 分级 + 文件行号），文件在 `C:\Users\99228\.dsh\agent-presets\code-review\`；新建会话时在预设选择器里选即可。
- **给自己加的用法**：写文档要配图 → Kling；想搞清某个二进制 / 网页的内部行为 → REA；提交前自查 → 开「代码审查模式」会话做只读 review。
- **本次没有新增 MCP**（不想动这份正在用的配置），但把配方记下来了：在 `cordis.patch.yml` 末尾追加一条 `- insert:` 的 `@deepseek-ai/dsh-mcp-client` 条目（`serverName / transport / command / args / env`），重启宿主生效。

