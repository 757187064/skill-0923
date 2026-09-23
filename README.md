# pi-isolated-subagent-plugin

A local Pi package that delegates work to subagents running in **fully isolated
`pi` child processes**.

It is based on the official
`examples/extensions/subagent` extension, with two deliberate changes:

1. The child is started with `--system-prompt <file>` instead of
   `--append-system-prompt <file>`, so the child's system prompt **replaces**
   Pi's default system prompt rather than appending to it.
2. The child is started with additional isolation flags:
   `--no-extensions --no-context-files --no-skills --no-prompt-templates`.

The package ships its own `isolated-test` agent and an `isolated-test` prompt
template, so it is self-contained and never writes into the Pi agent directory.

## Layout

```
pi-isolated-subagent-plugin/
├── package.json                  # "pi" manifest declares extension + prompts
├── extensions/subagent/
│   ├── index.ts                  # subagent tool (spawns isolated pi child)
│   └── agents.ts                 # discovers bundled + user/project agents
├── agents/
│   └── isolated-test.md          # bundled agent definition
├── prompts/
│   └── isolated-test.md          # /isolated-test prompt template
└── README.md
```

## Install

```bash
pi install D:\Tools\pi-isolated-subagent-plugin
```

Then start Pi and run:

```
/isolated-test
```

or invoke the tool directly:

```
Use the subagent tool with agent "isolated-test" and task "Prove you are isolated."
```

The final answer must start with `ISOLATED:`.

## Isolated child invocation

```
pi --mode json -p --no-session \
   --no-extensions --no-context-files --no-skills --no-prompt-templates \
   --system-prompt <temp-agent-prompt> \
   --model deepseek/deepseek-flash \
   --tools read,grep,find,ls \
   Task: ...
```
