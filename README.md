# pi-isolated-subagent-plugin

这是一个本地 Pi package/plugin，用来把任务委派给运行在**独立 `pi` 子进程**里的子 agent。

本插件基于 Pi 官方示例 `examples/extensions/subagent` 改造，核心目标是验证：

- 子 agent 有自己的系统提示词。
- 子 agent 不承接主 agent 的系统提示词。
- 子 agent 不共享主会话历史、上下文文件、skills、extensions、prompt templates。

## 关键改动

相比官方 subagent 示例，本插件做了两个关键改动：

1. 子进程使用 `--system-prompt <file>`，而不是 `--append-system-prompt <file>`。

   `--system-prompt` 会替换子进程的系统提示词；如果使用 `--append-system-prompt`，则会把子 agent 提示词追加到 Pi 默认系统提示词后面，不符合隔离要求。

2. 子进程启动时额外加入隔离参数：

   ```text
   --no-extensions
   --no-context-files
   --no-skills
   --no-prompt-templates
   ```

   这些参数用于阻止子进程加载主环境中的扩展、上下文文件、skills 和 prompt templates。

## 目录结构

```text
pi-isolated-subagent-plugin/
├── package.json
├── README.md
├── agents/
│   └── isolated-test.md
├── extensions/
│   └── subagent/
│       ├── agents.ts
│       └── index.ts
└── prompts/
    └── isolated-test.md
```

说明：

- `package.json`：声明这是一个 Pi package，并通过 `pi` 字段暴露 extension 和 prompt template。
- `extensions/subagent/index.ts`：注册 `subagent` 工具，并负责启动隔离的 `pi` 子进程。
- `extensions/subagent/agents.ts`：发现并读取 agent 定义。
- `agents/isolated-test.md`：内置测试 agent，要求最终回答以 `ISOLATED:` 开头。
- `prompts/isolated-test.md`：用于快速测试的 prompt template。

## 安装

如果插件目录在本机：

```cmd
pi install D:\Tools\pi-isolated-subagent-plugin
```

如果从 GitHub 克隆后安装：

```cmd
pi install 路径\到\skill-0923
```

安装后启动 Pi：

```cmd
pi
```

## 测试

进入 Pi 后运行：

```text
/isolated-test
```

也可以直接要求主 agent 调用工具：

```text
请使用 subagent 工具，调用 agent "isolated-test"，任务是：证明你是隔离子 agent。
```

预期结果：子 agent 的最终回答必须以 `ISOLATED:` 开头，并说明自己运行在隔离的 `pi` 子进程里。

## 子进程启动方式

插件中的子 agent 会以类似下面的方式启动：

```text
pi --mode json -p --no-session ^
   --no-extensions --no-context-files --no-skills --no-prompt-templates ^
   --system-prompt <temp-agent-prompt> ^
   --model deepseek/deepseek-flash ^
   --tools read,grep,find,ls ^
   Task: ...
```

其中最重要的是：

- `--system-prompt`：替换子 agent 的系统提示词。
- `--no-session`：不继承主会话历史。
- `--no-extensions`：不加载扩展。
- `--no-context-files`：不加载 `AGENTS.md`、`CLAUDE.md` 等上下文文件。
- `--no-skills`：不加载 skills。
- `--no-prompt-templates`：不加载 prompt templates。

## 注意事项

这是一种“提示词与上下文隔离”，不是操作系统级安全沙箱。

子 agent 仍然运行在同一台机器上，并使用当前 Pi 可用的模型凭据。若需要更强隔离，应额外使用独立配置目录、独立 API key、容器或虚拟机。
